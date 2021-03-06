<properties
	pageTitle="Criar uma VM do Linux usando um modelo do Azure | Microsoft Azure"
	description="Crie uma VM do Linux no Azure usando um modelo do Azure Resource Manager."
	services="virtual-machines-linux"
	documentationCenter=""
	authors="vlivech"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="04/29/2016"
	ms.author="v-livech"/>

# Criar uma VM do Linux usando um modelo do Azure

Este artigo mostra como implantar rapidamente uma nova Máquina Virtual do Linux no Azure usando um Modelo do Azure. O artigo requer uma conta do Azure ([obtenha uma avaliação gratuita](https://azure.microsoft.com/pricing/free-trial/)) e a [CLI do Azure](../xplat-cli-install.md) conectada (`azure login`) e no modo do gerenciador de recursos (`azure config mode arm`). Você também pode implantar rapidamente uma VM do Linux usando o [Portal do Azure](virtual-machines-linux-quick-create-portal.md) ou a [CLI do Azure](virtual-machines-linux-quick-create-cli.md).


## Comando rápido

Nos exemplos de comandos a seguir, substitua os valores entre &lt; e &gt; pelos valores de seu próprio ambiente.

```bash
azure group create \
-n quicksecuretemplate \
-l eastus \
--template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json
```

## Passo a passo detalhado

Os modelos permitem criar VMs no Azure com as configurações que você deseja personalizar durante a inicialização, como nomes de usuário e nomes de host. Para este artigo, nos concentraremos em Iniciar uma VM do Ubuntu usando um modelo do Azure que cria um grupo de segurança de rede (NSG) com apenas uma porta aberta (22 para SSH) e que requer chaves SSH para o logon.

Os modelos do Azure Resource Manager são arquivos JSON que podem ser usados para tarefas simples e únicas – como iniciar uma VM Ubuntu, como feito neste artigo – ou para construir as configurações do Azure complexas de ambientes inteiros como um teste, implantação de produção ou desenvolvimento de rede para o sistema operacional para implantação de pilha do aplicativo.

## Criar a VM Linux

O exemplo de código a seguir mostra como chamar o `azure group create` para criar um grupo de recursos e implantar uma VM do Linux protegida por SSH ao mesmo tempo usando [este modelo do Azure Resource Manager](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json). Lembre-se de que, em seu exemplo, você precisará usar nomes que sejam exclusivos para seu ambiente. Este exemplo usa `quicksecuretemplate` como o nome do grupo de recursos, `securelinux` como o nome da VM e `quicksecurelinux` como um nome de subdomínio.

```bash
azure group create \
-n quicksecuretemplate \
-l eastus \
--template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json
```

Saída

```bash
info:    Executing command group create
+ Getting resource group quicksecuretemplate
+ Creating resource group quicksecuretemplate
info:    Created resource group quicksecuretemplate
info:    Supply values for the following parameters
sshKeyData: ssh-rsa AAAAB3Nza<..ssh public key text..>VQgwjNjQ== vlivech@azure
+ Initializing template configurations and parameters
+ Creating a deployment
info:    Created template deployment "azuredeploy"
data:    Id:                  /subscriptions/<..subid text..>/resourceGroups/quicksecuretemplate
data:    Name:                quicksecuretemplate
data:    Location:            eastus
data:    Provisioning State:  Succeeded
data:    Tags: null
data:
info:    group create command OK
```

Você pode criar um novo grupo de recursos e implantar uma VM usando o parâmetro `--template-uri` ou pode baixar ou criar um modelo localmente e passar o modelo usando o parâmetro `--template-file` com um caminho para o arquivo de modelo como um argumento. A CLI do Azure solicita os parâmetros necessários ao modelo.

## Próximas etapas

Depois de criar VMs Linux com modelos, você desejará ver que outras estruturas de aplicativos estão disponíveis para implantar com modelos. Pesquise a [galeria de modelos](https://azure.microsoft.com/documentation/templates/) para descobrir quais estruturas de aplicativos implantar em seguida.

<!---HONumber=AcomDC_0504_2016-->