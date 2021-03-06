<properties
	pageTitle="Práticas Recomendadas para o Serviço de Aplicativo do Azure"
	description="Aprenda as práticas recomendadas e solução de problemas do Serviço de Aplicativo do Azure."
	services="app-service"
	documentationCenter=""
	authors="dariagrigoriu"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/19/2016"
	ms.author="dariagrigoriu"/>
    
# Práticas Recomendadas para o Serviço de Aplicativo do Azure

Este artigo resume as práticas recomendadas para usar o [Serviço de Aplicativo do Azure](http://go.microsoft.com/fwlink/?LinkId=529714).

## <a name="colocation"></a>Colocação
Quando recursos do Azure compondo uma solução, como um aplicativo Web e um banco de dados de recursos estão localizados em regiões diferentes, os efeitos podem incluir o seguinte:

*  Maior latência na comunicação entre recursos
*  Encargos monetários para transferência de dados de saída entre regiões, como observado na [página de preços do Azure](https://azure.microsoft.com/pricing/details/data-transfers)

A colocação na mesma região é melhor para que os recursos do Azure que compõem uma solução como um aplicativo Web e um banco de dados ou uma conta de armazenamento usada para armazenar conteúdo ou dados. Durante a criação de recursos, você deve verificar se eles estão na mesma região do Azure, a menos que você tenha motivos de design ou de negócios específicos para que eles não estejam. Você pode mover um aplicativo do Serviço de Aplicativo para a mesma região do banco de dados utilizando o [recurso de clonagem de Serviço de Aplicativo](app-service-web-app-cloning-portal.md) atualmente disponível para aplicativos do Plano do Serviço de Aplicativo Premium.

## <a name="memoryresources"></a>Quando aplicativos consomem mais memória do que o esperado
Quando você observar que um aplicativo consome mais memória do que o esperado, conforme indicado no monitoramento ou nas recomendações de serviço, considere o [recurso de Reparo Automático de Serviço de Aplicativo](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites). Uma das opções para o recurso de Reparo Automático é tomar ações personalizadas com base em um limite de memória. Ações abrangem o espectro de notificações de email a investigação por meio de despejo de memória no ponto de atenuação ao reciclar o processo de trabalho. A Recuperação Automática pode ser configurada por meio de web.config e por meio de uma interface do usuário amigável, conforme descrito nesta postagem de blog para a [Extensão de Site de Suporte de Serviço de Aplicativo](https://azure.microsoft.com/blog/additional-updates-to-support-site-extension-for-azure-app-service-web-apps).

## <a name="CPUresources"></a>Quando aplicativos consomem mais CPU do que o esperado
Quando você observar que um aplicativo consome mais CPU que o esperado ou enfrenta picos de CPU repetidos, conforme indicado pelo monitoramento ou pelas recomendações serviço, considere escalar verticalmente ou horizontalmente o plano de Serviço de Aplicativo. Se seu aplicativo estiver com estado, escalar verticalmente será a única opção, enquanto que, se seu aplicativo estiver sem estado, escalar horizontalmente dará mais flexibilidade e maior potencial de escala.

Para saber mais sobre aplicativos "com estado" versus "sem estado", assista a este vídeo: [Planejando um aplicativo de várias camadas ponta a ponta escalonável no aplicativo Web do Microsoft Azure](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid). Para saber mais sobre como as opções de colocação em escala e dimensionamento automático, leia: [Dimensionar um aplicativo Web no Serviço de Aplicativo do Azure](web-sites-scale.md).

## <a name="socketresources"></a>Quando os recursos de soquete são exauridos
Uma razão comum para esgotar as conexões TCP de saída é o uso de bibliotecas de cliente que não são implementadas para reutilizar conexões TCP ou, no caso de um protocolo de nível superior, como HTTP - Keep-Alive não sendo utilizado. Consulte a documentação para cada uma das bibliotecas referenciadas pelos aplicativos no seu Plano de Serviço de Aplicativo para garantir que eles são configurados ou acessados em seu código para reutilização eficiente de conexões de saída. Além disso, siga as diretrizes de documentação biblioteca de criação correta e versão ou de limpeza para evitar vazamento de conexões. Embora essas investigações de bibliotecas de cliente estejam em andamento, o impacto pode ser reduzido escalando horizontalmente para várias instâncias.

<!---HONumber=AcomDC_0525_2016-->