<properties
   pageTitle="Comunicação de serviço WCF do Reliable services | Microsoft Azure"
   description="A comunicação de serviço WCF interna no Service Fabric fornece a comunicação WCF cliente-serviço para o Reliable Services."
   services="service-fabric"
   documentationCenter=".net"
   authors="BharatNarasimman"
   manager="timlt"
   editor="vturecek"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="required"
   ms.date="03/28/2016"
   ms.author="bharatn"/>

# Pilha de comunicação baseada no WCF para Reliable Services
A estrutura de Serviços confiáveis permite que os autores do serviço escolham qual pilha de comunicação desejam usar para seu serviço. Eles podem ligar a pilha de comunicação de sua escolha por meio do **ICommunicationListener** retornado dos métodos [CreateServiceReplicaListeners ou CreateServiceInstanceListeners](service-fabric-reliable-services-communication.md). A estrutura fornece uma implementação da pilha de comunicação baseada no WCF (Windows Communication Foundation) da pilha de comunicação para autores de serviço que desejam usar comunicação baseada no WCF.

## Ouvinte de comunicação do WCF
A implementação específica do WCF do **ICommunicationListener** é fornecida pela classe **Microsoft.ServiceFabric.Services.Communication.Wcf.Runtime.WcfCommunicationListener**.

Digamos que temos um contrato de serviço do tipo `ICalculator`

```csharp
[ServiceContract]
public interface ICalculator
{
    [OperationContract]
    Task<int> Add(int value1, int value2);
}
```

Podemos criar um ouvinte de comunicação do WCF no serviço da seguinte maneira.

```csharp

protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[] { new ServiceReplicaListener((context) =>
        new WcfCommunicationListener<ICalculator>(
            wcfServiceObject:this,
            serviceContext:context,
            //
            // The name of the endpoint configured in the ServiceManifest under the Endpoints section
            // that identifies the endpoint that the WCF ServiceHost should listen on.
            //
            endpointResourceName: "WcfServiceEndpoint",

            //
            // Populate the binding information that you want the service to use.
            //
            listenerBinding: WcfUtility.CreateTcpListenerBinding()
        )
    )};
}

```

## Desenvolvimento de clientes para a pilha de comunicação do WCF
Para o desenvolvimento de clientes que se comuniquem com serviços usando o WCF, a estrutura fornece **WcfClientCommunicationFactory**, que é a implementação específica do WCF do [ClientCommunicationFactoryBase](service-fabric-reliable-services-communication.md).

```csharp

public WcfCommunicationClientFactory(
    Binding clientBinding = null,
    IEnumerable<IExceptionHandler> exceptionHandlers = null,
    IServicePartitionResolver servicePartitionResolver = null,
    string traceId = null,
    object callback = null);
```

O canal de comunicação do WCF pode ser acessado do **WcfCommunicationClient** criado pelo **WcfCommunicationClientFactory**.

```csharp

public class WcfCommunicationClient : ServicePartitionClient<WcfCommunicationClient<ICalculator>>
   {
       public WcfCommunicationClient(ICommunicationClientFactory<WcfCommunicationClient<ICalculator>> communicationClientFactory, Uri serviceUri, ServicePartitionKey partitionKey = null, TargetReplicaSelector targetReplicaSelector = TargetReplicaSelector.Default, string listenerName = null, OperationRetrySettings retrySettings = null)
           : base(communicationClientFactory, serviceUri, partitionKey, targetReplicaSelector, listenerName, retrySettings)
       {
       }
   }

```

O código do cliente pode usar o **WcfCommunicationClientFactory** com o **WcfCommunicationClient** que implementa o **ServicePartitionClient** para determinar o ponto de extremidade de serviço e se comunicar com o serviço.

```csharp
// Create binding
Binding binding = WcfUtility.CreateTcpClientBinding();
// Create a partition resolver
IServicePartitionResolver partitionResolver = ServicePartitionResolver.GetDefault();
// create a  WcfCommunicationClientFactory object.
var wcfClientFactory = new WcfCommunicationClientFactory<ICalculator>
    (clientBinding: binding, servicePartitionResolver: partitionResolver);

//
// Create a client for communicating with the ICalculator service that has been created with the
// Singleton partition scheme.
//
var calculatorServiceCommunicationClient =  new WcfCommunicationClient(
                wcfClientFactory,
                ServiceUri,
                ServicePartitionKey.Singleton);

//
// Call the service to perform the operation.
//
var result = calculatorServiceCommunicationClient.InvokeWithRetryAsync(
                client => client.Channel.Add(2, 3)).Result;

```
>[AZURE.NOTE] O ServicePartitionResolver padrão supõe que o cliente está em execução no mesmo cluster que o serviço. Se este não for o caso, crie um objeto ServicePartitionResolver e passe os pontos de extremidade de conexão do cluster.

## Próximas etapas
* [Chamada de procedimento remoto com Reliable Services remoto](service-fabric-reliable-services-communication-remoting.md)

* [API Web com OWIN no Reliable Services](service-fabric-reliable-services-communication-webapi.md)

* [Securing communication for Reliable Services](service-fabric-reliable-services-secure-communication.md)

<!---HONumber=AcomDC_0420_2016-->