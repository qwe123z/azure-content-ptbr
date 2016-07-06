<properties
	pageTitle="Conectar o Operations Manager ao Log Analytics | Microsoft Azure"
	description="Para manter seu investimento existente no System Center Operations Manager e usar funcionalidades estendidas com o Log Analytics, você pode integrar o Operations Manager ao seu espaço de trabalho do OMS."
	services="log-analytics"
	documentationCenter=""
	authors="MGoedtel"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="log-analytics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/11/2016"
	ms.author="magoedte"/>

# Conectar o Operations Manager ao Log Analytics

Para manter seu investimento existente no System Center Operations Manager e usar funcionalidades estendidas com o Log Analytics, você pode integrar o Operations Manager ao seu espaço de trabalho do OMS. Isso permite aproveitar as oportunidades do OMS e continuar a usar o Operations Manager para:

- Continuar a monitorar a integridade dos seus serviços de TI com o Operations Manager
- Manter a integração com suas soluções de ITSM com suporte ao gerenciamento de incidentes e problemas
- Gerenciar o ciclo de vida dos agentes implantados em máquinas virtuais locais e em IaaS em nuvem pública que podem ser monitorados com o Operations Manager

A integração com o System Center Operations Manager agrega valor à sua estratégia de operações de serviço, aproveitando a velocidade e eficiência do OMS ao coletar, armazenar e analisar dados do Operations Manager. O OMS ajuda a correlacionar e busca identificar as falhas dos problemas e identificar recorrências para dar suporte ao seu processo de gerenciamento de problemas existente. A flexibilidade do mecanismo de pesquisa para examinar dados de desempenho, de eventos e de alertas com painéis avançados e recursos de relatórios para expor tais dados de maneiras significativas demonstra a potência do que OMS traz para complementar o Operations Manager.

Os agentes de relatórios para o grupo de gerenciamento do Operations Manager coletam dados dos seus servidores com base em como as fontes de dados e soluções do Log Analytics habilitados na sua assinatura do OMS. Dependendo da solução que você habilitou, dados dessas soluções são enviados diretamente de um servidor de gerenciamento do Operations Manager para o serviço Web do OMS ou por causa do volume de dados coletados no sistema gerenciado por agente, são enviados diretamente do agente para o serviço Web do OMS. O servidor de gerenciamento encaminha diretamente os dados do OMS para o serviço Web do OMS e eles nunca são gravados no banco de dados do OperationsManager ou OperationsManagerDW. Quando um servidor de gerenciamento perde a conectividade com o serviço Web do OMS, ele armazena os dados em cache localmente até a comunicação ser restabelecida com o OMS. Se o servidor de gerenciamento estiver offline devido a uma manutenção planejada ou paralisação inesperada, outro servidor de gerenciamento no grupo de gerenciamento continuará a conectividade com o OMS.

O diagrama a seguir mostra a relação entre o System Center Operations Manager e o OMS.

![oms-operations-manager-integration-diagram](./media/log-analytics-om-agents/oms-operations-manager-connection.png)

## Requisitos do sistema
Antes de começar, examine os detalhes a seguir para verificar se você atende aos pré-requisitos necessários.

- O OMS dá suporte apenas ao Operations Manager 2012 SP1 UR6 e superior e ao Operations Manager 2012 R2 UR2 e superior. Foi adicionado suporte a proxy ao Operations Manager 2012 SP1 UR7 e Operations Manager 2012 R2 UR3.
- Todos os agentes do Operations Manager devem atender aos requisitos de suporte mínimos. Verifique se os agentes estão em dia pelo menos até a atualização mínima, caso contrário o tráfego de agente do Windows falhará e muitos erros poderão lotar o log de eventos do Operations Manager.
- Uma assinatura do OMS. Para obter informações adicionais, leia a [Introdução ao Log Analytics](log-analytics-get-started.md).

## Conectando o Operations Manager ao OMS
Realize a série de etapas a seguir para configurar o grupo de gerenciamento do Operations Manager para se conectar a um dos seus espaços de trabalho do OMS.

1. Abra o console do Operations Manager e selecione o espaço de trabalho **Administração**.
2. Expanda o nó do Operations Management Suite e clique em **Conexão**.
3. Clique no link **Registrar-se no Operations Management Suite**.
4. Na página **Assistente de Integração do Operations Management Suite: autenticação**, insira o endereço de email ou número de telefone e a senha da conta de administrador associada à sua assinatura do OMS e clique em **Entrar**.
5. Depois de ser autenticado com êxito, na página **Assistente de Integração do Operations Management Suite: selecione o espaço de trabalho**, será solicitado que você selecione seu espaço de trabalho do OMS. Se você tiver mais de um espaço de trabalho, selecione aquele em que você deseja registrar-se com o grupo de gerenciamento do Operations Manager da lista suspensa e clique em **Avançar**.

    >[AZURE.NOTE] O Operations Manager dá suporte a apenas um espaço de trabalho do OMS por vez. A conexão e os computadores que foram registrados ao OMS com o espaço de trabalho anterior são removidos do OMS.

6. Na página **Assistente de Integração do Operations Manager Suite: resumo**, confirme suas configurações e, se elas estiverem corretas, clique em **Criar**.
7. Na página **Assistente de Integração do Operations Management Suite: conclusão** clique em **Fechar**.

### Adicionar computadores gerenciados por agente
Depois de configurar a integração com o seu espaço de trabalho do OMS, apenas uma conexão com o OMS é estabelecida, nenhum dado é coletado dos agentes de relatórios para o grupo de gerenciamento. Isso só acontecerá depois de configurar quais computadores gerenciados por agente específicos coletarão dados para o Log Analytics. Você pode selecionar os objetos de computador individualmente ou um grupo que contém objetos de computador do Windows. Não é possível selecionar um grupo que contém instâncias de outra classe, como discos lógicos ou Bancos de Dados SQL.

1. Abra o console do Operations Manager e selecione o espaço de trabalho **Administração**.
2. Expanda o nó do Operations Management Suite e clique em **Conexão**.
3. Clique no link **Adicionar um Computador/Grupo** no cabeçalho Ações no lado direito do painel.
4. Na caixa de diálogo **Pesquisa de Computador**, é possível pesquisar computadores ou grupos monitorados pelo Operations Manager. Selecione os computadores ou grupos a serem carregado para o OMS, clique em **Adicionar** e em **OK**.

Você pode exibir computadores e grupos configurados para coletar dados do nó Computadores Gerenciados no Operations Management Suite no espaço de trabalho **Administração** do Console de operações. Aqui, é possível adicionar ou remover computadores e grupos conforme necessário.

### Definir configurações de proxy do OMS no Console de operações
Execute as etapas a seguir se um servidor proxy interno estiver entre o grupo de gerenciamento e o serviço Web do OMS. Essas configurações são gerenciadas centralmente do grupo de gerenciamento e distribuídas para sistemas gerenciados por agentes que estão incluídos no escopo para coletar dados para OMS. Isso é útil para quando determinadas soluções ignorarem o servidor de gerenciamento e enviarem dados diretamente para o serviço Web do OMS.

1. Abra o console do Operations Manager e selecione o espaço de trabalho **Administração**.
2. Expanda Operations Management Suite e clique em **Conexões**.
3. Na exibição Conexão do OMS, clique em **Configurar Servidor Proxy**.
4. Na página **Assistente do Operations Management Suite: servidor proxy**, selecione **Usar um servidor proxy para acessar o Operations Management Suite** e digite a URL com o número da porta, como http://corpproxy:80 por exemplo, e clique em **Concluir**.

Se o servidor proxy exigir autenticação, execute as etapas a seguir para configurar as credenciais e as configurações que precisam ser propagadas para computadores gerenciados que relatarão para o OMS no grupo de gerenciamento.

1. Abra o console do Operations Manager e selecione o espaço de trabalho **Administração**.
2. Em **Configuração RunAs**, selecione **Perfis**.
3. Abra o perfil **Executar System Center Advisor como proxy de perfil**.
4. No Assistente de perfil Executar como, clique em Adicionar para usar uma conta Executar como. Você pode criar uma nova conta [Executar Como](https://technet.microsoft.com/library/hh321655.aspx) ou usar uma conta existente. Essa conta deve ter permissões suficientes para passar pelo servidor proxy.
5. Para definir a conta a ser gerenciada, escolha **Uma classe, grupo ou objeto selecionado**, clique em **Selecionar...** e em **Grupo...** para abrir a caixa de **Pesquisa de Objetos**.
6. Procure e selecione o **Grupo de Servidores de Monitoramento do Microsoft System Center Advisor**. Clique em **OK** depois de selecionar o grupo para fechar a caixa **Pesquisa de Grupo**.
7.	Clique em **OK** para fechar a caixa **Adicionar uma conta Executar como**.
8.	Clique em **Salvar** para concluir o assistente e salvar suas alterações.

Depois de a conexão ser criada e você configurar quais agentes coletarão e relatarão dados para o OMS, a seguinte configuração é aplicada ao grupo de gerenciamento, não necessariamente nesta ordem:

- A conta Executar como **Microsoft.SystemCenter.Advisor.RunAsAccount.Certificate** é criada. Ela está associada ao perfil Executar como **Blob de perfil Executar como do Microsoft System Center Advisor** e está voltada para duas classes: **Servidor de Coleta** e **Grupo de Gerenciamento do Operations Manager**.
- Dois conectores são criados. O primeiro é denominado **Microsoft.SystemCenter.Advisor.DataConnector** e é automaticamente configurado com uma assinatura para encaminhar todos os alertas gerados de instâncias de todas as classes no grupo de gerenciamento para o Log Analytics do OMS. O segundo conector é o **Conector do Advisor**, que é responsável pela comunicação com o serviço Web do OMS e o compartilhamento de dados.
- Agentes e grupos que você selecionou para coletar dados do grupo de gerenciamento serão adicionados ao **Grupo de Servidores de Monitoramento do Microsoft System Center Advisor**.

## Atualizações do pacote de gerenciamento
Após o assistente de configuração ser concluído, cada grupo de gerenciamento do Operations Manager estabelecerá uma conexão com o serviço do OMS. O servidor de gerenciamento será sincronizado com o serviço Web e receberá informações de configuração atualizadas na forma de pacotes de gerenciamento para as soluções que você habilitou integradas ao Operations Manager. O Operations Manager verificará atualizações para esses pacotes de gerenciamento, baixando-as e importando-as imediatamente quando estiverem disponíveis. Há duas regras específicas que controlam esse comportamento:

- **Microsoft.SystemCenter.Advisor.MPUpdate** - Atualiza os pacotes de gerenciamento base do OMS. Executada a cada doze (12) horas por padrão.
- **Microsoft.SystemCenter.Advisor.Core.GetIntelligencePacksRule** - Atualiza os pacotes de gerenciamento de solução habilitados no seu espaço de trabalho. Executada a cada cinco (5) minutos por padrão.

Você pode substituir essas duas regras para impedir o download automático desabilitando-as ou modificando a frequência com que o servidor de gerenciamento é sincronizado com o OMS para determinar se um novo pacote de gerenciamento está disponível e deve ser baixado. Siga as etapas em [Como substituir uma regra ou monitor](https://technet.microsoft.com/library/hh212869.aspx) para modificar o parâmetro **Frequency** com um valor em segundos para alterar o agendamento de sincronização ou modificar o parâmetro **Enabled** para desabilitar as regras. Direcionar as substituições a todos os objetos da classe Grupo de Gerenciamento do Operations Manager.

Se você quiser continuar seguindo o processo de controle de alterações existente para controlar as versões do pacote de gerenciamento no grupo de gerenciamento de produção, desabilite as regras e habilite-as durante horários específicos quando as atualizações forem permitidas. Se você tiver um desenvolvimento ou um grupo de gerenciamento de garantia de qualidade em seu ambiente e ele tiver conectividade com a Internet, configure esse grupo de gerenciamento com um espaço de trabalho do OMS para dar suporte a esse cenário. Isso permitirá analisar e avaliar as versões iterativas dos pacotes de gerenciamento do OMS antes de liberá-las para seu grupo de gerenciamento de produção.

## Alternar um grupo do Operations Manager para um novo espaço de trabalho do OMS
1. Faça logon na sua assinatura do OMS e crie um novo espaço de trabalho no [Microsoft Operations Management Suite](http://oms.microsoft.com/).
2. Abra o console do Operations Manager com uma conta que seja membro da função Administradores do Operations Manager e selecione o espaço de trabalho **Administração**.
3. Expanda o Operations Management Suite e selecione **Conexões**.
4. Selecione o link **Reconfigurar o Operation Management Suite** no meio do painel.
5. Siga o **Assistente de Integração do Operations Management Suite** e insira o endereço de email ou número de telefone e a senha da conta de administrador associada ao seu espaço de trabalho do OMS.

    > [AZURE.NOTE] A página **Assistente de Integração do Operations Management: selecione o espaço de trabalho** apresentará o espaço de trabalho existente que está em uso.


## Validar a Integração do Operations Manager com o OMS
Existem algumas maneiras diferentes de verificar se a integração do OMS ao Operations Manager foi bem-sucedida.

### Para confirmar a integração do portal do OMS

1.	No portal do OMS, clique no bloco **Configurações**
2.  Selecione **Fontes Conectadas**.
3.	Na tabela da seção System Center Operations Manager, você deve ver o nome do grupo de gerenciamento listado com o número de agentes e status de quando os dados foram recebidos pela última vez.

    ![oms-settings-connectedsources](./media/log-analytics-om-agents/oms-settings-connectedsources.png)

4.  Anote o valor da **ID do Espaço de Trabalho** no lado esquerdo da página Configurações. Você vai validá-la junto ao seu grupo de gerenciamento do Operations Manager abaixo.

### Para confirmar a integração do console de Operações

1.	Abra o console do Operations Manager e selecione o espaço de trabalho **Administração**.
2.	Clique em **Pacotes de Gerenciamento** e, na caixa de texto **Procurar por:**, digite **Advisor** ou **Intelligence**.
3.	Dependendo das soluções que tiver habilitado, você verá um pacote de gerenciamento correspondente listado nos resultados da pesquisa. Por exemplo, se tiver habilitado a solução de Alert Management, o pacote de gerenciamento do Microsoft System Center Advisor Alert Management constará na lista.
4.  Da exibição **Monitoramento**, navegue até a exibição **Operations Management Suite\\Estado de Integridade**. Selecione um servidor de Gerenciamento no painel **Estado do Servidor de Gerenciamento** e, no painel **Exibição de Detalhes**, confirme se o valor da propriedade **URI do serviço de autenticação** coincide com a ID do espaço de trabalho do OMS.

    ![oms-opsmgr-mg-authsvcuri-property-ms](./media/log-analytics-om-agents/oms-opsmgr-mg-authsvcuri-property-ms.png)


## Remover a Integração com o OMS
Quando você não precisar mais da integração entre o grupo de gerenciamento do Operations Manager e o espaço de trabalho do OMS, há várias etapas necessárias para remover corretamente a conexão e a configuração do grupo de gerenciamento. O procedimento a seguir orientará você a atualizar seu espaço de trabalho do OMS, excluindo a referência do grupo de gerenciamento, excluindo os conectores do OMS e, em seguida, excluindo os pacotes de gerenciamento com suporte do OMS.

1.  No portal do OMS, clique no bloco **Configurações**.
2.	Selecione **Fontes Conectadas**.
3.	Na tabela na seção System Center Operations Manager, você verá o nome do grupo de gerenciamento que deseja remover do espaço de trabalho. Na coluna **Últimos Dados**, clique em **Remover**.  
4.	Uma janela será exibida solicitando que você confirme se deseja continuar com a remoção. Clique em **Sim** para confirmar.  
5.	Abra o Shell de Comando do Operations Manager com uma conta que seja membro da função Administradores do Operations Manager.

    >[AZURE.WARNING] Verifique se você não tem pacotes de gerenciamento personalizados com a palavra Advisor ou IntelligencePack no nome antes de prosseguir, caso contrário, as etapas a seguir vão excluí-los do grupo de gerenciamento.

6.	No prompt de shell de comando, digite `Get-SCOMManagementPack -name "*advisor*" | Remove-SCOMManagementPack`

7.	Em seguida, digite `Get-SCOMManagementPack -name “*IntelligencePack*” | Remove-SCOMManagementPack`

8.	Abra o Console de operações do Operations Manager com uma conta que seja membro da função Administradores do Operations Manager.
9.	Em **Administração**, selecione o nó **Pacotes de Gerenciamento** e, na caixa **Procurar por:**, digite **Advisor** e verifique se os seguintes pacotes de gerenciamento ainda foram importados no grupo de gerenciamento:

    - Microsoft System Center Advisor
    - Microsoft System Center Advisor Interno

Para excluir os dois conectores, Microsoft.SystemCenter.Advisor.DataConnector e o Conector do Advisor, salve o script do PowerShell abaixo em seu computador e execute-o usando os exemplos a seguir.

```
    .\OM2012_DeleteConnector.ps1 “Advisor Connector” <ManagementGroupName>
    .\OM2012_DeleteConnectors.ps1 “Microsoft.SytemCenter.Advisor.DataConnector” <ManagementGroupName>
```

>[AZURE.NOTE] O computador no qual você executará esse script, se não for um servidor de gerenciamento, deverá ter o shell de comando do Operations Manager 2012 SP1 ou R2 instalado, dependendo da versão do seu grupo de gerenciamento.

```
    `param(
    [String] $connectorName,
    [String] $mgName="localhost"
    )
    $mg = new-object Microsoft.EnterpriseManagement.ManagementGroup $mgName
    $admin = $mg.GetConnectorFrameworkAdministration()
    ##########################################################################################
    # Configures a connector with the specified name.
    ##########################################################################################
    function New-Connector([String] $name)
    {
         $connectorForTest = $null;
         foreach($connector in $admin.GetMonitoringConnectors())
    {
    if($connectorName.Name -eq ${name})
    {
         $connectorForTest = Get-SCOMConnector -id $connector.id
    }
    }
    if ($connectorForTest -eq $null)
    {
         $testConnector = New-Object Microsoft.EnterpriseManagement.ConnectorFramework.ConnectorInfo
         $testConnector.Name = $name
         $testConnector.Description = "${name} Description"
         $testConnector.DiscoveryDataIsManaged = $false
         $connectorForTest = $admin.Setup($testConnector)
         $connectorForTest.Initialize();
    }
    return $connectorForTest
    }
    ##########################################################################################
    # Removes a connector with the specified name.
    ##########################################################################################
    function Remove-Connector([String] $name)
    {
        $testConnector = $null
        foreach($connector in $admin.GetMonitoringConnectors())
       {
        if($connector.Name -eq ${name})
       {
         $testConnector = Get-SCOMConnector -id $connector.id
       }
      }
     if ($testConnector -ne $null)
     {
        if($testConnector.Initialized)
     {
     foreach($alert in $testConnector.GetMonitoringAlerts())
     {
       $alert.ConnectorId = $null;
       $alert.Update("Delete Connector");
     }
     $testConnector.Uninitialize()
     }
     $connectorIdForTest = $admin.Cleanup($testConnector)
     }
    }
    ##########################################################################################
    # Delete a connector's Subscription
    ##########################################################################################
    function Delete-Subscription([String] $name)
    {
      foreach($testconnector in $admin.GetMonitoringConnectors())
      {
      if($testconnector.Name -eq $name)
      {
        $connector = Get-SCOMConnector -id $testconnector.id
      }
    }
    $subs = $admin.GetConnectorSubscriptions()
    foreach($sub in $subs)
    {
      if($sub.MonitoringConnectorId -eq $connector.id)
      {
        $admin.DeleteConnectorSubscription($admin.GetConnectorSubscription($sub.Id))
      }
     }
    }
    #New-Connector $connectorName
    write-host "Delete-Subscription"
    Delete-Subscription $connectorName
    write-host "Remove-Connector"
    Remove-Connector $connectorName
```

No futuro, se você planejar reconectar o grupo de gerenciamento para um espaço de trabalho do OMS, será necessário importar novamente o arquivo de pacote de gerenciamento `Microsoft.SystemCenter.Advisor.Resources.<Language>\.mpb` do pacote cumulativo de atualizações mais recente aplicado ao grupo de gerenciamento. Você pode encontrar esse arquivo no `%ProgramFiles%\Microsoft System Center 2012` ou na pasta `System Center 2012 R2\Operations Manager\Server\Management Packs for Update Rollups`.

## Próximas etapas

- [Adicionar soluções do Log Analytics da Galeria de Soluções](log-analytics-add-solutions.md) para adicionar funcionalidade e obter dados.
- [Definir configurações de proxy e firewall no Log Analytics](log-analytics-proxy-firewall.md) se sua organização usar um servidor proxy ou firewall para que os agentes possam se comunicar com o serviço do Log Analytics.

<!---HONumber=AcomDC_0518_2016-->