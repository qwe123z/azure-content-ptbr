
<properties
		pageTitle="Monitorando uma VM do Linux com uma extensão de VM | Microsoft Azure"
		description="Saiba como usar a Extensão de Diagnóstico do Linux para monitorar os dados de desempenho e diagnóstico de uma VM do Linux no Azure."
		services="virtual-machines-linux"
		documentationCenter=""
  		authors="NingKuang"
		manager="timlt"
		editor=""
  		tags="azure-service-management"/>

<tags
		ms.service="virtual-machines-linux"
		ms.workload="infrastructure-services"
		ms.tgt_pltfrm="vm-linux"
		ms.devlang="na"
		ms.topic="article"
		ms.date="12/15/2015"
		ms.author="Ning"/>


# Usar a Extensão de Diagnóstico do Linux para monitorar os dados de desempenho e diagnóstico da VM do Linux

## Introdução

A Extensão de Diagnóstico do Linux ajuda um usuário a monitorar as VMs Linux em execução no Microsoft Azure. Ela oferece os seguintes recursos:

- Coleta e carregamento de informações de desempenho do sistema, da VM Linux para a tabela de armazenamento do usuário, incluindo informações de diagnóstico e syslog.
- Permite que os usuários personalizem as métricas de dados que serão coletadas e carregadas.
- Permite que os usuários carreguem arquivos de log especificados em uma tabela de armazenamento designada.

Na versão 2.0, os dados incluem:

- Todos os logs Rsyslog do Linux, incluindo logs de sistema, de segurança e de aplicativos.
- Todos os dados do sistema especificados [no site do System Center Cross Platform Solutions](https://scx.codeplex.com/wikipage?title=xplatproviders) (Soluções de Plataforma Cruzada do System Center).
- Arquivos de log especificados pelo usuário.

Essa extensão funciona com os modelos de implantação Clássico e do Resource Manager.


## Habilitar a extensão
Você pode habilitar essa extensão usando o [Portal do Azure](https://portal.azure.com/#), o Azure PowerShell ou os scripts da CLI do Azure.

Para exibir e configurar os dados do sistema e de desempenho diretamente do Portal do Azure, execute estas [etapas no blog do Azure](https://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/ "URL para o blog do Windows"/).


Este artigo se concentra em como habilitar e configurar a extensão usando os comandos da CLI do Azure. Isso permite que você leia e exiba os dados diretamente da tabela de armazenamento.

Observe que os métodos de configuração descritos abaixo não funcionarão para o Portal do Azure. Para exibir e configurar os dados de desempenho e do sistema diretamente do Portal do Azure, a extensão deve ser habilitada no Portal.


## Pré-requisitos
- **Agente Linux do Azure versão 2.0.6 ou posterior**. Observe que a maioria das imagens de galeria da VM Linux do Azure inclui a versão 2.0.6 ou posterior. É possível executar **WAAgent-version** para confirmar a versão que está instalada na VM. Se a VM estiver executando uma versão anterior à 2.0.6, execute [estas instruções no GitHub](https://github.com/Azure/WALinuxAgent "instruções") para atualizá-la.

- **CLI do Azure** Siga [esta orientação de instalação da CLI](../xplat-cli-install.md) para configurar o ambiente da CLI do Azure em seu computador. Depois de instalar a CLI do Azure, você poderá usar o comando **azure** em sua interface de linha de comando (Bash, Terminal ou prompt de comando) para acessar os comandos da CLI do Azure. Por exemplo:
	- Execute **azure vm extension set --help** para obter informações de ajuda detalhadas.
	- Execute **azure login** para entrar no Azure.
	- Execute **azure vm list** para listar todas as máquinas virtuais que você tem no Azure.
- Uma conta de armazenamento para armazenar os dados. Você precisará de um nome de conta de armazenamento criado anteriormente e de uma chave de acesso para carregar os dados no armazenamento.


## Usar o comando da CLI do Azure para habilitar a Extensão de Diagnóstico do Linux

### Cenário 1. Habilitar a extensão com o conjunto de dados padrão
Na versão 2.0 ou posterior, os dados padrão que serão coletados incluem:

- Todas as informações de Rsyslog (incluindo os logs de sistema, segurança e aplicativo).  
- Um conjunto principal de dados base do sistema. Observe que o conjunto de dados completo está descrito no [site do System Center Cross Platform Solutions](https://scx.codeplex.com/wikipage?title=xplatproviders). Se você quiser habilitar dados extras, continue com as etapas nos Cenários 2 e 3.

Etapa 1. Crie um arquivo chamado PrivateConfig.json com o conteúdo a seguir:

    {
        "storageAccountName" : "the storage account to receive data",
        "storageAccountKey" : "the key of the account"
    }

Etapa 2. Execute **azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions 2.* --private-config-path PrivateConfig.json**.


###   Cenário 2: Personalizar a métrica do monitor de desempenho  
Esta seção descreve como personalizar a tabela de dados de desempenho e diagnóstico.

Etapa 1. Crie um arquivo chamado PrivateConfig.json com o conteúdo descrito no Cenário 1. Além disso, crie um arquivo chamado PublicConfig.json. Especifique os dados específicos que deseja coletar.

Para conhecer todos os provedores e variáveis com suporte, consulte o [site do System Center Cross Platform Solutions](https://scx.codeplex.com/wikipage?title=xplatproviders). Você pode ter várias consultas e armazená-las em várias tabelas, acrescentando mais consultas ao script.

Por padrão, os dados de Rsyslog sempre são coletados.

    {
      	"perfCfg":
      	[
      	    {
      	        "query" : "SELECT PercentAvailableMemory, AvailableMemory, UsedMemory ,PercentUsedSwap FROM SCX_MemoryStatisticalInformation",
      	        "table" : "LinuxMemory"
      	    }
      	]
    }


Etapa 2. Execute **azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json**.


###   Cenário 3: Carregar os próprios arquivos de log
Esta seção descreve como coletar e carregar arquivos de log específicos em sua conta de armazenamento. Você precisa especificar o caminho até seu arquivo de log e o nome da tabela onde você deseja armazenar seu log. Você pode criar vários arquivos de log adicionando várias entradas de arquivo/tabela ao script.

Etapa 1. Crie um arquivo chamado PrivateConfig.json com o conteúdo descrito no Cenário 1. Em seguida, crie outro arquivo denominado PublicConfig.json com o conteúdo a seguir:

    {
        "fileCfg" :
        [
            {
                "file" : "/var/log/mysql.err",
                "table" : "mysqlerr"
             }
        ]
    }


Etapa 2. Execute **azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json**.


###   Cenário 4: Parar a coleta de todos os logs pela extensão
Esta seção descreve como parar a coleta de logs pela extensão. Observe que o processo do agente de monitoramento ainda estará em execução mesmo com essa reconfiguração. Se quiser parar o completamente processo do agente de monitoramento, você poderá fazer isso desabilitando a extensão. O comando para desabilitar a extensão é **azure vm extension set --disable <vm_name> LinuxDiagnostic Microsoft.OSTCExtensions '2.*'**.

Etapa 1. Crie um arquivo chamado PrivateConfig.json com o conteúdo descrito no Cenário 1. Crie outro arquivo denominado PublicConfig.json com o conteúdo a seguir:

    {
        "perfCfg" : [],
        "enableSyslog" : "false"
    }


Etapa 2. Execute **azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions '2.*' --private-config-path PrivateConfig.json --public-config-path PublicConfig.json**.


## Examinar os dados
Os dados de desempenho e diagnóstico são armazenados em uma tabela de Armazenamento do Azure. Revise [Como usar o Armazenamento de Tabelas do Azure no Ruby](../storage/storage-ruby-how-to-use-table-storage.md) para saber como acessar os dados na tabela de armazenamento por meio de scripts da CLI do Azure.

Além disso, você pode usar as ferramentas de interface do usuário a seguir para acessar os dados:

1. Gerenciador de Servidores do Visual Studio. Vá até sua conta de armazenamento. Depois que a VM for executada por aproximadamente cinco minutos, você deverá ver as quatro tabelas padrão: "LinuxCpu", "LinuxDisk", "LinuxMemory" e "Linuxsyslog". Clique duas vezes nos nomes de tabela para exibir os dados.

2. [Gerenciador de Armazenamento do Azure](https://azurestorageexplorer.codeplex.com/ "Gerenciador de Armazenamento do Azure").

![imagem](./media/virtual-machines-linux-classic-diagnostic-extension/no1.png)

Se você tiver habilitado fileCfg ou perfCfg (conforme descrito nos Cenários 2 e 3), você poderá usar o Gerenciador de Servidores do Visual Studio e o Gerenciador de Armazenamento do Azure para exibir dados não padrão.

## Problemas conhecidos
- Na versão 2.0 da Extensão de Diagnóstico do Linux, as informações do Rsyslog e o arquivo de log especificado pelo cliente só podem ser acessados por meio de scripts.

<!---HONumber=AcomDC_0615_2016-->