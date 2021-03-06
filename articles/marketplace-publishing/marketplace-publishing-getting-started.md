<properties
   pageTitle="Visão geral de como criar e implantar uma oferta no Marketplace | Microsoft Azure"
   description="Entender as etapas necessárias para se tornar um Desenvolvedor da Microsoft aprovado e criar e implantar uma imagem de máquina virtual, o modelo, serviço de dados ou o serviço de desenvolvedor no Azure Marketplace"
   services="marketplace-publishing"
   documentationCenter=""
   authors="HannibalSII"
   manager=""
   editor=""/>

<tags
   ms.service="marketplace"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/01/2016"
   ms.author="hascipio" />

# Como publicar uma oferta no Azure Marketplace
Este artigo é fornecido para ajudar um desenvolvedor a criar e implantar uma solução no Azure Marketplace a ser adquirida e utilizada por outros parceiros e clientes do Azure.

A primeira coisa que você gostaria de fazer como um publicador é definir que tipo de solução sua empresa está oferecendo. O Azure Marketplace oferece suporte a várias soluções, e cada uma delas requer um conjunto ligeiramente diferente de trabalho para publicar com êxito no Marketplace.

Tipos de soluções:

- Imagem de máquina virtual
- Serviço do desenvolvedor
- Serviço de dados
- Modelo de solução

Algumas etapas são compartilhadas entre os diferentes tipos de soluções. Este artigo fornece uma breve visão geral das etapas que você precisará concluir para qualquer tipo de solução.

> [AZURE.NOTE] Antes de começar qualquer trabalho no Azure Marketplace, você deve ser pré-aprovado. Isso não é aplicável a editores de serviço de dados.

||Imagem de máquina virtual |Serviço do desenvolvedor | Serviço de dados | Modelo de solução |
|----|----|----|----|----|
| **Obter a aprovação prévia** | [Microsoft Azure Certified][link-certification] | Visualização particular | n/d | [Microsoft Azure Certified][link-certification] |
| **Etapa 1: registrar sua conta de desenvolvedor** | [Conta do Microsoft Developer: criação e registro][link-accts] | Visualização particular | [Conta do Microsoft Developer: criação e registro][link-accts] | [Conta do Microsoft Developer: criação e registro][link-accts] |
|**Etapa 2: criar sua oferta**| [Pré-requisitos gerais não técnicos](marketplace-publishing-pre-requisites.md)| Visualização particular | [Pré-requisitos gerais não técnicos](marketplace-publishing-pre-requisites.md)| [Pré-requisitos gerais não técnicos](marketplace-publishing-pre-requisites.md)|
||[Pré-requisitos técnicos de VM][link-single-vm-prereq] | Visualização particular | [Pré-requisitos técnicos de serviço de dados](marketplace-publishing-data-service-creation-prerequisites.md) | [Pré-requisitos técnicos de modelo de solução](marketplace-publishing-solution-template-creation-prerequisites.md) |
||[Guia de publicação de imagem de VM][link-single-vm] | Visualização particular | [Guia de publicação do serviço de dados](marketplace-publishing-data-service-creation.md) | [Guia de publicação de modelo de solução](marketplace-publishing-solution-template-creation.md) |
||[Guia de conteúdo de marketing do Azure Marketplace][link-pushstaging] | Visualização particular | [Guia de conteúdo de marketing do Azure Marketplace][link-pushstaging] | [Guia de conteúdo de marketing do Azure Marketplace][link-pushstaging] |
|**Etapa 3: enviar por push sua oferta para preparo** | [Testar sua oferta de VM em preparo](marketplace-publishing-vm-image-test-in-staging.md) | Visualização particular | [Testar sua oferta de serviço de dados em preparo](marketplace-publishing-data-service-test-in-staging.md) | [Testar o modelo de solução em preparo](marketplace-publishing-solution-template-test-in-staging.md) |
|**Etapa 4: implantar a oferta no Marketplace** | [Implantar a oferta no Marketplace][link-pushprod] | Visualização particular | [Implantar a oferta no Marketplace][link-pushprod] | [Implantar a oferta no Marketplace][link-pushprod] |

## Suporte
- [Guia de pós-produção para ofertas de máquina virtual](marketplace-publishing-vm-image-post-publishing.md)
- [Entendendo o relatório de percepções do vendedor][suppt-rpt-insights]
- [Entendendo os relatórios de pagamento][suppt-rpt-payouts]
- [Como alterar seu incentivo ao revendedor de Provedor de Soluções na Nuvem](marketplace-publishing-csp-incentive.md)
- [Solução de problemas comuns de publicação no Marketplace][suppt-common]
- [Obtenha suporte como um editor][suppt-general]

## Recursos adicionais
- Para saber mais sobre os portais usados, veja [Portais necessários](marketplace-publishing-portals.md).

**Máquinas virtuais**

- [Configurando o PowerShell do Azure](marketplace-publishing-powershell-setup.md)
- [Criando uma imagem de máquina virtual no local](marketplace-publishing-vm-image-creation-on-premise.md)
- [Criar uma máquina virtual que executa o Windows no Portal de Visualização do Azure](../virtual-machines/virtual-machines-windows-hero-tutorial.md)

**Serviços de dados**

- [Mapeamento de OData de serviço de dados](marketplace-publishing-data-service-creation-odata-mapping.md)
- [Nós de mapeamento de OData de serviço de dados](marketplace-publishing-data-service-creation-odata-mapping-nodes.md)
- [Exemplos de mapeamento de OData de serviço de dados](marketplace-publishing-data-service-creation-odata-mapping-examples.md)

[suppt-general]: marketplace-publishing-get-publisher-support.md
[suppt-rpt-insights]: marketplace-publishing-report-seller-insights.md
[suppt-rpt-payouts]: marketplace-publishing-report-payout.md
[suppt-common]: marketplace-publishing-support-common-issues.md
[link-certification]: marketplace-publishing-azure-certification.md
[link-accts]: marketplace-publishing-accounts-creation-registration.md
[link-single-vm]: marketplace-publishing-vm-image-creation.md
[link-single-vm-prereq]: marketplace-publishing-vm-image-creation-prerequisites.md
[link-multi-vm]: marketplace-publishing-solution-template-creation.md
[link-multi-vm-prereq]: marketplace-publishing-solution-template-creation-prerequisites.md
[link-datasvc]: marketplace-publishing-data-service-creation.md
[link-datasvc-prereq]: marketplace-publishing-data-service-creation-prerequisites.md
[link-devsvc]: marketplace-publishing-dev-service-creation.md
[link-devsvc-prereq]: marketplace-publishing-dev-service-creation-prerequisites.md
[link-pushstaging]: marketplace-publishing-push-to-staging.md
[link-pushprod]: marketplace-publishing-push-to-production.md

<!---HONumber=AcomDC_0608_2016-->