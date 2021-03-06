<properties
   pageTitle="Criar um cluster do Azure Service Fabric local ou em qualquer nuvem | Microsoft Azure"
   description="Saiba como criar um cluster do Azure Service Fabric em qualquer computador (físico ou virtual) que executar o Windows Server, seja localmente ou em qualquer nuvem."
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="06/21/2016"
   ms.author="chackdan"/>


# Criar um cluster do Azure Service Fabric localmente ou na nuvem

O Azure Service Fabric permite a criação de clusters do Service Fabric em quaisquer máquinas virtuais ou computadores que estiverem executando o Windows Server. Isso significa que você pode implantar e executar aplicativos do Service Fabric em qualquer ambiente em que tenha um conjunto de computadores com Windows Server interconectados, seja ele local ou em qualquer provedor de nuvem. O Service Fabric fornece um pacote de instalação para a criação de clusters do Service Fabric.

Este artigo explica as etapas necessárias para criar um cluster usando o pacote autônomo do Service Fabric local, mas ele pode ser adaptada facilmente a qualquer outro ambiente, como na nuvem.

>[AZURE.NOTE] Atualmente, essa Oferta Autônoma está em versão de visualização. [Clique aqui](http://go.microsoft.com/fwlink/?LinkID=733084) se quiser baixar uma cópia do EULA agora.

<a id="downloadpackage"></a>
## Baixar o pacote autônomo do Service Fabric


[Baixe o pacote autônomo do Service Fabric para Windows Server 2012 R2](http://go.microsoft.com/fwlink/?LinkId=730690), que se chama *Microsoft.Azure.ServiceFabric.WindowsServer.&lt;version&gt;.zip*.

No pacote de download, você encontrará os seguintes arquivos:

|**Nome do arquivo**|**Descrição breve**|
|-----------------------|--------------------------|
|MicrosoftAzureServiceFabric.cab|O arquivo CAB que contém os binários que serão implantados em cada máquina no cluster.|
|ClusterConfig.Unsecure.DevCluster.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para um cluster de desenvolvimento de uma única VM/Máquina, de três nós, não protegido, incluindo as informações para cada nó que faz parte do cluster. |
|ClusterConfig.Unsecure.MultiMachine.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para o cluster, incluindo as informações para cada computador que faz parte de um cluster de várias VMs/máquinas não protegido.|
|ClusterConfig.Windows.DevCluster.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para um cluster de desenvolvimento de uma única VM/Máquina, de três nós, protegido, incluindo as informações para cada nó que faz parte do cluster. O cluster é protegido usando [identidades do Windows](https://msdn.microsoft.com/library/ff649396.aspx).|
|ClusterConfig.Windows.MultiMachine.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para o cluster protegido, incluindo as informações para cada computador que faz parte de um cluster protegido de várias VMs/máquinas. O cluster é protegido usando [identidades do Windows](https://msdn.microsoft.com/library/ff649396.aspx).|
|ClusterConfig.x509.DevCluster.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para um cluster de desenvolvimento de uma única VM/Máquina, de três nós, protegido, incluindo as informações para cada nó que faz parte do cluster. O cluster é protegido usando certificados x509 do Windows.|
|ClusterConfig.x509.MultiMachine.JSON|Arquivo de exemplo de configuração de cluster que contém todas as configurações para o cluster protegido, incluindo as informações para cada computador que faz parte de um cluster protegido de várias VMs/máquinas. O cluster é protegido usando certificados x509.|
|EULA.txt|Os termos de licença para uso do pacote autônomo Microsoft Azure Service Fabric. [Clique aqui](http://go.microsoft.com/fwlink/?LinkID=733084) se quiser baixar uma cópia do EULA agora.|
|Readme. txt|Link para as notas de versão e instruções básicas de instalação. Trata-se de um pequeno subconjunto das instruções que você encontrará nesta página.|
|CreateServiceFabricCluster.ps1|Script do PowerShell que cria o cluster usando as configurações no arquivo ClusterConfig.JSON.|
|RemoveServiceFabricCluster.ps1|Script do PowerShell para remoção do cluster usando as configurações no arquivo ClusterConfig.JSON.|
|AddNode.ps1|Script do PowerShell para cluster adicionando um nó a um cluster existente|
|RemoveNode.ps1|Script do PowerShell para cluster removendo um nó de um cluster|


## Planejar e preparar para a implantação do cluster
As etapas a seguir precisam ser executadas antes da criação do cluster.

### Etapa 1: Planejar a infraestrutura de cluster
Você está prestes a criar um cluster do Service Fabric nas máquinas que possui. Sendo assim, você decide a quais tipos de falhas deseja que o cluster sobreviva. Por exemplo, você precisa de linhas de energia ou conexões com a Internet separadas que alimentam esses computadores? Além disso, você também deve levar em consideração a segurança física dos computadores. Qual é sua localização física e quem precisa ter acesso a eles? Depois de tomar essas decisões, você mapeia logicamente os computadores para vários domínios de falha (consulte abaixo para obter mais informações). O planejamento da infra-estrutura para os clusters de produção será envolverá muito mais aspectos do que para os clusters de teste.

### Etapa 2: Preparar os computadores para atender os pré-requisitos
Pré-requisitos para cada computador que você deseja adicionar ao cluster:

- Um mínimo de 2 GB de memória é recomendado
- Conectividade de rede - certifique-se de que os computadores estejam em uma rede ou redes seguras
- Windows Server 2012 R2 ou Windows Server 2012 (para isso, você precisa ter o KB2858668 instalado).
- .NET Framework 4.5.1 ou superior, instalação completa
- Windows PowerShell 3.0
- O administrador do cluster que estiver implantando e configurando o cluster precisa ter privilégios de administrador em cada um dos computadores.
- O serviço RemoteRegistry deve ser executado em todos os computadores.

### Etapa 3: Determinar o tamanho inicial do cluster
Cada nó consiste em uma pilha completa do Service Fabric e é um membro individual do cluster do Service Fabric. Em uma implantação típica do Service Fabric, há um nó por instância de SO (física ou virtual). O tamanho do cluster é determinado por suas necessidades de negócios. No entanto, você precisa ter um tamanho mínimo de cluster de três nós (máquinas/VMs). Observe que, para fins de desenvolvimento, você pode ter mais de um nó em um determinado computador. Em um ambiente de produção, o Service Fabric dá suporte apenas a um nó por máquina virtual ou física.

### Etapa 4: Determinar o número de domínios de falha e domínios de atualização
Um **FD (domínio de falha)** é uma unidade física de falha e está diretamente relacionada à infraestrutura física dos datacenters. Um domínio de falha consiste em componentes de hardware (computadores, comutadores, entre outros) que compartilham um ponto único de falha. Embora não haja um mapeamento individual entre domínios de falha e racks, de maneira geral, cada rack pode ser considerado um domínio de falha. Ao considerar os nós no cluster, é altamente recomendável que os nós sejam distribuídos entre pelo menos três domínios de falha.

Ao especificar FDs em *ClusterConfig.JSON*, você pode escolher somente o nome do FD. O Service Fabric dá suporte a FDs hierárquicos, de modo que você pode refletir a topologia da sua infraestrutura neles. Por exemplo, é permitido o seguinte:

- "faultDomain": "fd:/Room1/Rack1/Machine1"
- "faultDomain": "fd:/FD1"
- "faultDomain": "fd:/Room1/Rack1/PDU1/M1"


Um **UD (domínio de atualização)** é uma unidade lógica de nós. Durante uma atualização orquestrada do Service Fabric (atualização de aplicativo ou atualização de cluster), todos os nós em um UD são desativados para realizar a atualização, enquanto os nós em outros UDs permanecem disponíveis para atender às solicitações. As atualizações de firmware que você fizer em seus computadores não se aplicarão a UDs, portanto você deve fazê-las um computador por vez.

A maneira mais simples de pensar nesses conceitos é considerar os FDs como a unidade da falha não planejada e as UDs como a unidade da manutenção planejada.

Ao especificar UDs em *ClusterConfig*.JSON*, você pode escolher o nome do UD. Por exemplo, é permitido o seguinte:

- "upgradeDomain": "UD0"
- "upgradeDomain": "UD1A"
- "upgradeDomain": "DomainRed"
- "upgradeDomain": "Blue"

### Etapa 5: Baixar o pacote autônomo do Service Fabric para Windows Server
[Baixe o pacote autônomo do Service Fabric para Windows Server](http://go.microsoft.com/fwlink/?LinkId=730690) e descompacte o pacote em um computador de implantação que não faz parte do cluster ou em um dos computadores que farão parte do cluster.

<a id="createcluster"></a>
## Criar o cluster

Após ter concluído as etapas descritas na seção de planejamento e preparação acima, você está pronto para criar o cluster.

### Etapa 1: Modificar a configuração do cluster
Abra *ClusterConfig.JSON* do pacote baixado. Você pode usar qualquer editor de sua escolha para modificar as seguintes configurações:

|**Parâmetro de configuração**|**Descrição**|
|-----------------------|--------------------------|
|NodeTypes|Os tipos de nós permitem separar os nós do cluster em vários grupos. Um cluster precisa ter pelo menos um NodeType. Todos os nós em um grupo têm as seguintes características em comum. <br> *Name* - é o nome do tipo de nó. <br>*EndPoints* - são vários pontos de extremidade (portas) nomeados que estão associados a esse tipo de nó. Você pode usar qualquer número da porta que desejar, desde que ele não entre em conflito com qualquer outra coisa no manifesto e não esteja sendo usado por outro programa no computador/VM <br> *PlacementProperties* - descrevem propriedades para esse tipo de nó, que você usará como restrições de posicionamento para serviços do sistema ou seus serviços. Essas propriedades de nó são pares de chave/valor definidos pelo usuário que fornecem metadados extras para um determinado nó. Exemplos de propriedades de nó incluem se o nó tem ou não um disco rígido ou placa de vídeo, o número de eixos no seu disco rígido, núcleos e outras propriedades físicas. <br> *Capacities* - as capacidades do nó definem o nome e a quantidade de um recurso específico que um determinado nó tem disponível para consumo. Por exemplo, um nó pode definir que tem capacidade para uma métrica chamada "MemoryInMb" e que tem 2048 MB de memória disponível por padrão. Essas capacidades são usadas no tempo de execução para garantir que serviços que necessitam de uma quantidade específica de recursos sejam colocados em nós que tenham os recursos restantes disponíveis.|
|Nós|Os detalhes de cada um de nós que farão parte do cluster (tipo de nó, nome do nó, endereço IP, domínio de falha e domínio de atualização do nó). Os computadores nos quais você deseja que o cluster seja criado precisam ser listados aqui com o endereço IP. <br> Se você usar os mesmos endereços IP para todos os nós, um cluster one-box será criado e será possível usá-lo para fins de teste. Clusters one-box não devem ser usados para a implantação de cargas de trabalho de produção.|

### Etapa 2: Executar o script de criação de cluster
Depois de ter modificado a configuração do cluster no documento JSON e adicionado todas as informações do nó a ele, execute o script do PowerShell de criação de cluster na pasta do pacote e transmita para ele o caminho para o arquivo de configuração e o local da raiz do pacote.

Esse script pode ser executado em qualquer computador que tenha acesso de administrador a todos os computadores listados como nós no arquivo de configuração do cluster. O computador no qual esse script é executado pode ou não fazer parte do cluster.

```
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.JSON -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab
```

>[AZURE.NOTE] Os logs de implantação estão disponíveis localmente na VM/máquina que você executou o powershell CreateServiceFabricCluster. Você os encontrará em uma subpasta chamada "DeploymentTraces" na pasta da qual você executou o comando PS.

## Adicionar nós ao cluster do Service Fabric 

1. Prepare a VM/Máquina a ser adicionada ao cluster (confira a etapa 2 na seção acima Planejar e preparar para implantação de cluster). 
2. Planeje a qual Domínio de falha e Domínio de atualização você vai adicionar essa VM/Máquina.
3. [Baixe o pacote autônomo do Service Fabric para Windows Server](http://go.microsoft.com/fwlink/?LinkId=730690) e descompacte o pacote na VM/Máquina que está planejando adicionar ao cluster. 
4. Abra um prompt de administração do PowerShell e navegue até o local do pacote descompactado
5. Execute AddNode.PS1

```
.\AddNode.ps1 -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClusterConnectionEndPoint 182.17.34.52:19000 -UpgradeDomain UD1 -FaultDomain FD1
```

## Remover nós do cluster do Service Fabric 

1. Usando o TS, acesse a VM/máquina que você deseja remover do cluster
2. Abra um prompt de administração do PowerShell e navegue até o local do pacote descompactado
5. Executar RemoveNode.PS1

```
.\RemoveNode.ps1 -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -ExistingClusterConnectionEndPoint 182.17.34.52:19000
```

## Excluir o cluster do Service Fabric 
1. Usando o TS, acesse uma das VMs/máquinas que faz parte do cluster
2. Abra um prompt de administração do PowerShell e navegue até o local do pacote descompactado
5. Executar RemoveNode.PS1

```
.\RemoveNode.ps1 -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -ExistingClusterConnectionEndPoint 182.17.34.52:19000
```


## Próximas etapas
- [Noções básicas sobre segurança de cluster](service-fabric-cluster-security.md)
- [ SDK do Service Fabric e introdução](service-fabric-get-started.md)
- [Gerenciando seu aplicativo da Malha do Serviço no Visual Studio](service-fabric-manage-application-in-visual-studio.md).
- [Visão geral do recurso de criação de cluster autônomo e uma comparação com clusters gerenciados pelo Azure](service-fabric-deploy-anywhere.md)

<!---HONumber=AcomDC_0622_2016-->