<properties 
	pageTitle="Monitorar a integridade e o uso do aplicativo com o Application Insights" 
	description="Introdução ao Application Insights. Analise o uso, disponibilidade e desempenho de seu local ou aplicativos do Microsoft Azure." 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/25/2015" 
	ms.author="awills"/>
 
# Monitore o desempenho em aplicativos da web

*O Application Insights está em modo de visualização.*


Certifique-se de que seu aplicativo está sendo bem executado, e saiba rapidamente sobre quaisquer falhas. O [Application Insights][start] vai lhe dizer sobre quaisquer problemas de desempenho e exceções, e o ajudará a localizar e diagnosticar as causas raízes.

O Application Insights pode monitorar serviços e aplicativos Web Java e ASP.NET e serviços WCF. Eles podem ser hospedados no local, em máquinas virtuais ou como sites do Microsoft Azure.

No lado do cliente, o Application Insights pode realizar a telemetria de páginas da Web e de uma grande variedade de dispositivos, incluindo iOS, Android e aplicativos da Windows Store.


## <a name="setup"></a>Configurar o monitoramento de desempenho

Se você ainda não tem o Application Insights adicionado ao seu projeto (ou seja, não tem o ApplicationInsights.config), escolha uma destas formas para começar:

* [Aplicativos Web ASP.NET](app-insights-asp-net.md)
 * [Adicionar monitoramento de exceção](app-insights-asp-net-exceptions.md)
 * [Adicionar monitoramento de dependência](app-insights-monitor-performance-live-website-now.md)
* [Aplicativos Web J2EE](app-insights-java-get-started.md)
 * [Adicionar monitoramento de dependência](app-insights-java-agent.md)


## <a name="view"></a>Explorando métricas de desempenho

No [portal do Azure](https://portal.azure.com), navegue até o recurso do Application Insights que você configurou para seu aplicativo. A folha de visão geral mostra os dados de desempenho básicos:



Clique em qualquer gráfico para ver mais detalhes e para ver os resultados por um período mais longo. Por exemplo, clique no bloco Solicitações e, em seguida, selecione um intervalo de tempo:


![Clique por mais dados e selecione um intervalo de tempo](./media/app-insights-web-monitor-performance/appinsights-48metrics.png)

Clique em um gráfico para selecionar outras medidas que são exibidas, ou adicionar um novo gráfico e selecionar a métrica:

![Clique em um gráfico para escolher métricas](./media/app-insights-web-monitor-performance/appinsights-61perfchoices.png)

> [AZURE.NOTE] **Desmarque todas as métricas** para ver a seleção completa que está disponível. As métricas se enquadram em grupos; quando qualquer membro de um grupo é selecionado, somente os outros membros do grupo aparecem.


## <a name="metrics"></a>O que significa tudo isso? Blocos e relatórios de desempenho

Existe uma variedade de métricas de desempenho que você pode obter. Vamos começar com estas que aparecem por padrão na folha do aplicativo.


### Solicitações

O número de solicitações de HTTP receberam em um período especifico. Compare isso com os resultados em outros reatórios para ver como seu aplicativo se comporta conforme a carga varia.

As solicitações HTTP incuem todas as solicitações GET ou POST para páginas, dados e imagens.

Clique no mosaico para obter contagens para URLs específicas.

### Tempo de resposta média

Mede o tempo entre uma solicitação da web inserindo seu aplicativo e a resposta que está sendo devolvida.

Os pontos mostram uma média da movimentação. Se existe várias solicitações, pode haver agumas que desviam da média sem um pico óbvio ou profundo no gráfico.

Procure por picos incomuns. Em geral, espere o tempo de resposta para eevar com uma elevação nas solicitações. Se a elevação é desproporcional, seu aplicativo pode ser atingido por um limite de recurso como CPU ou a capacidade de um serviço que usa.

Clique no bloco para obter tempos para URLs específicas.

![](./media/app-insights-web-monitor-performance/appinsights-42reqs.png)


### Solicitações mais lentas

![](./media/app-insights-web-monitor-performance/appinsights-44slowest.png)

Mostra quais solicitações podem precisar de sintonização de desempenho.


### Solicitações falhas

![](./media/app-insights-web-monitor-performance/appinsights-46failed.png)

Uma contagem de solicitações que gerou exceções não capturadas.

Clique no mosaico para ver os detalhes de falhas específicas, e selecione uma solicitação individual para ver seus detalhes.

Somente uma amostra representativa de falhas é retida para inspeção individual.

### Outras métricas

Para ver o que outras métricas que você pode exibir, clique em um gráfico e, em seguida, desmarque a seleção de todas as métricas para ver o conjunto disponível completo. Clique em (i) para ver cada definição da métrica.

![Desmarque a seleção de todas as métricas para ver o conjunto completo](./media/app-insights-web-monitor-performance/appinsights-62allchoices.png)


Ao selecionar qualquer métrica, desabilitará as outras que não podem aparecer no mesmo gráfico.

## Contadores de desempenho do sistema


O Windows fornece uma ampla variedade de contadores de desempenho, mas você também pode definir seus próprios.

(Para aplicativos hospedados no Azure, [envie o Diagnóstico do Azure para o Application Insights](app-insights-azure-diagnostics.md).)

Para ver um conjunto de [contadores de desempenho](http://www.codeproject.com/Articles/8590/An-Introduction-To-Performance-Counters) comuns, abra a folha **Servidores**. Você também pode escolher contadores ao editar um gráfico e selecionar uma métrica da seção Contadores de desempenho:

![](./media/app-insights-web-monitor-performance/sys-perf.png)

O conjunto completo de métricas disponíveis no seu sistema pode ser determinado em sistemas Windows usando o comando PowerShell [`Get-Counter -ListSet *`](https://technet.microsoft.com/library/hh849685.aspx).

Se os contadores que deseja não estiverem na lista de métricas, você poderá adicioná-los ao conjunto coletado pelo SDK. Abra ApplicationInsights.config e edite a diretiva do coletor de desempenho:

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\Objects\Processes"/>
        <Add PerformanceCounter="\Sales(electronics)# Items Sold" ReportAs="Item sales"/>
      </Counters>
    </Add>

Você pode capturar os contadores padrão e aqueles implementados por conta própria. `\Objects\Processes` está disponível em todos os sistemas Windows; `\Sales...` é um exemplo de um contador personalizado que pode ser implementado em um servidor Web.

O formato é `\Category(instance)\Counter"`, ou apenas `\Category\Counter` para categorias que não têm instâncias.


`ReportAs` é necessário para os nomes de contadores que contêm caracteres além dos seguintes: letras, colchetes arredondados, barras "/", hifens, sublinhados, espaços e pontos.

Se você especificar uma instância, ela será coletada como uma dimensão "CounterInstanceName" da métrica reportada.

### Coletando contadores de desempenho no código

Para coletar contadores de desempenho do sistema e enviá-los por push ao Application Insights, você pode usar o trecho a seguir:

    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\.NET CLR Memory([replace-with-application-process-name])# GC Handles", "GC Handles")));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);

Ou você pode fazer a mesma coisa com métricas personalizadas que você criou:

    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Sales(electronics)# Items Sold", "Items sold"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);

Além disso, se você quiser

### Contagens de exceção

*Qual é a diferença entre a Taxa de exceções e as Métricas de exceções?*

* *Taxa de exceções* é um contador de desempenho do sistema. O CLR conta todas as exceções tratadas e sem tratamento que são lançadas, e divide o total em um intervalo de amostragem pela duração do intervalo. O SDK do Application Insights coleta esse resultado e o envia para o portal.
* *Exceções* é uma contagem dos relatórios TrackException recebida pelo portal no intervalo de amostragem do gráfico. Ele inclui apenas as exceções tratadas em que você tenha gravado chamadas TrackException em seu código e não inclui todas as [exceções sem tratamento](app-insights-asp-net-exceptions.md). 

## Definir alertas

Para ser notificado por email sobre valores incomuns de qualquer métrica, adicione um alerta. Você pode escolher para enviar o email para os administradores de conta ou para endereços de email específicos.

![](./media/app-insights-web-monitor-performance/appinsights-413setMetricAlert.png)

Defina o recurso antes de outras propriedades. Não escolha os recursos webtest se você quer definir alertas em métricas de desempenho ou de uso.

Observe as unidades quando você for solicitado para inserir o valor de limite.

*Não vejo o botão Adicionar Alerta.* - Esta é uma conta de grupo para a qual você tem acesso somente leitura? Verifique com o administrador da conta.

## <a name="diagnosis"></a>Diagnosticando problemas

Aqui estão algumas dicas para localizar e diagnosticar problemas de desempenho:

* Configure os [testes de web][availability] para ser alertado se seu site cair ou responder de forma incorreta ou lenta. 
* Compare a contagem de Solicitação com outras métricas para ver se falhas ou resposta lenta são relatadas ao carregar.
* [Inserir e pesquisar instruções de rastreamento][diagnostic] em seu código para ajudar a detectar problemas.

## <a name="next"></a>Próximas etapas

[Testes da web][availability] - Tem solicitações da web enviadas ao seu aplicativo em intervalos regulares em todo o mundo.

[Capture e pesquise rastreamento de diagnóstico][diagnostic] - Inserir chamadas de rastreamento e separar através dos resultados para problemas de pinpoint.

[Rastreamento de uso][usage] - Saiba como as pessoas usam se aplicativo.

[Solução de problemas][qna] - e Perguntas e respostas

## Vídeo

[AZURE.VIDEO performance-monitoring-application-insights]

<!--Link references-->

[availability]: app-insights-monitor-web-app-availability.md
[diagnostic]: app-insights-diagnostic-search.md
[greenbrown]: app-insights-asp-net.md
[qna]: app-insights-troubleshoot-faq.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[start]: app-insights-overview.md
[usage]: app-insights-web-track-usage.md

 

<!-----------HONumber=AcomDC_0330_2016-->