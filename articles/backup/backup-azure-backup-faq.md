<properties
   pageTitle="Perguntas frequentes do Backup do Azure | Microsoft Azure"
   description="Respostas para perguntas frequentes sobre o serviço de backup, o agente de backup, o backup e a retenção, a recuperação, a segurança e outras perguntas comuns sobre backup e recuperação de desastre."
   services="backup"
   documentationCenter=""
   authors="markgalioto"
   manager="jwhit"
   editor=""
   keywords="backup e recuperação de desastre; serviço de backup"/>

<tags
   ms.service="backup"
   ms.workload="storage-backup-recovery"
	 ms.tgt_pltfrm="na"
	 ms.devlang="na"
	 ms.topic="get-started-article"
	 ms.date="04/04/2016"
	 ms.author="trinadhk; giridham; arunak; markgal; jimpark;"/>

# Serviço de Backup do Azure - Perguntas frequentes

> [AZURE.SELECTOR]
- [Perguntas frequentes sobre backup para o modo Clássico](backup-azure-backup-faq.md)
- [Perguntas frequentes sobre backup para o modo ARM](backup-azure-backup-ibiza-faq.md)

Este artigo é uma lista de perguntas frequentes (e das respectivas respostas) sobre o serviço de Backup do Azure. Nossa comunidade responde rapidamente, e se uma pergunta for feita com frequência, nós a adicionaremos a este artigo. As respostas às perguntas normalmente fornecem referência ou informações de suporte. Você pode fazer perguntas sobre o Backup do Azure na seção Disqus deste artigo ou do artigo relacionado. Você também pode postar perguntas sobre o serviço de Backup do Azure no [fórum de discussão](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup).

## Instalação e configuração
**P1. Qual é a lista de sistemas operacionais com suporte por meio dos quais posso fazer backup no Azure usando o Backup do Azure?** <br/> R1. O Backup do Azure dá suporte aos sistemas operacionais da lista a seguir


| Sistema operacional | Plataforma | SKU |
| :------------- |-------------| :-----|
| Windows 8 e SPs mais recentes | 64 bits | Enterprise, Pro |
| Windows 7 e SPs mais recentes | 64 bits | Ultimate, Enterprise, Professional, Home Premium, Home Basic, Starter |
| Windows 8.1 e SPs mais recentes | 64 bits | Enterprise, Pro |
| Windows 10 | 64 bits | Enterprise, Pro, Home |
|Windows Server 2012 R2 e SPs mais recentes|	64 bits|	Standard, Datacenter, Foundation|
|Windows Server 2012 e SPs mais recentes|	64 bits|	Datacenter, Foundation, Standard|
|Windows Storage Server 2012 R2 e SPs mais recentes |64 bits|	Standard, Workgroup|
|Windows Storage Server 2012 e SPs mais recentes |64 bits |Standard, Workgroup
|Windows Server 2012 R2 e SPs mais recentes |64 bits|	Essential|
|Windows Server 2008 R2 SP1 |64 bits|	Standard, Enterprise, Datacenter, Foundation|
|Windows Server 2008 SP2 |64 bits|	Standard, Enterprise, Datacenter, Foundation|

**P2. Onde posso baixar o agente mais recente do Backup do Azure?** <br/> R2. Você pode baixar o agente mais recente para fazer backup do Windows Server, do System Center DPM ou do cliente Windows [daqui](http://aka.ms/azurebackup_agent). Se você quiser fazer backup de uma máquina virtual, use o Agente de VM (que instala automaticamente a extensão apropriada). O Agente de VM já está presente em máquinas virtuais criadas na galeria do Azure.

**P3. Qual versão do servidor SCDPM tem suporte?** <br/> R3. Recomendamos que você instale o agente de Backup do Azure [mais recente](http://aka.ms/azurebackup_agent) no pacote cumulativo de atualizações mais recente do SCDPM (UR6 de julho de 2015)

**P4. Ao configurar o agente do Backup do Azure, preciso inserir as **credenciais do cofre**. As credenciais do cofre expiram? R4. Sim, as credenciais do cofre expiram após 48 horas. Se o arquivo expirar, faça logon no Portal do Azure e baixe os arquivos de credenciais de cofre no seu cofre de backup.

**P5. Há algum limite para o número de cofres de backup que podem ser criados em cada assinatura do Azure?** <br/> R5. Sim. Desde julho de 2015, você pode criar 25 cofres por assinatura. Se você precisar de mais cofres, crie uma nova assinatura.

**P6. Devo considerar o cofre como uma entidade de cobrança?** <br/> R6. Embora seja possível obter uma fatura detalhada de cada cofre, é altamente recomendável que você considere uma assinatura do Azure como uma entidade de cobrança. Ela é consistente entre todos os serviços e é mais fácil de gerenciar.

**P7. Há um limite para o número de servidores/computadores que podem ser registrados em cada cofre?** <br/> R7. Sim, você pode registrar até 50 computadores por cofre. Para máquinas virtuais IaaS do Azure, o limite é 200 VMs por cofre. Se você precisar registrar mais computadores, crie um novo cofre.

**P8. Há um limite em relação à quantidade de dados que pode ser incluída no backup de um servidor/cliente Windows ou em um servidor SCDPM?** <br/> R8. Nº

**P9. Como registro meu servidor em outro datacenter?**<br/> R9. Os dados de backup são enviados para o datacenter do Serviço de Backup no qual estão registrados. A maneira mais fácil de alterar o datacenter é desinstalar e reinstalar o agente e registrar-se em um novo datacenter.

**P10. O que acontece se eu renomear um servidor Windows que está fazendo backup de dados no Azure?**<br/> R10. Ao renomear um servidor, todos os backups configurados atualmente serão interrompidos. Será necessário registrar o novo nome do servidor no Cofre de Backup. Quando você cria um novo registro, a primeira operação de backup é um backup completo, não um backup incremental. Se você precisar recuperar os dados incluídos em backups anteriores no cofre com o nome antigo do servidor, poderá recuperá-los usando a opção [**Outro servidor**](backup-azure-restore-windows-server.md#recover-to-an-alternate-machine) no assistente **Recuperar Dados**.


**P11. Em que tipos de unidades posso fazer backup de arquivos e pastas?** <br/> R11. Não é possível fazer backup do seguinte conjunto de unidades/volumes:

- Mídia removível: a unidade deve ser indicada como fixa para ser usada como uma origem de itens de backup.
- Volumes somente leitura: o volume deve ser gravável para que o VSS (Serviço de Cópias de Sombra de Volume) funcione.
- Volumes offline: o volume deve estar online para que o VSS funcione.
- Compartilhamento de rede: O volume deve ser local para o backup do servidor usando o backup online.
- Volumes protegidos pelo BitLocker: o volume deve ser desbloqueado antes de ser possível realizar o backup.
- Identificação do sistema de arquivos: NTFS é o único sistema de arquivos com suporte para esta versão do serviço de backup online.

**P12. De quais tipos de arquivo e pasta no servidor posso fazer backup?**<br/> R12. Os seguintes tipos têm suporte:

- Criptografado
- Compactado
- Esparso
- Compactado + esparso
- Links físicos: sem suporte, ignorado
- Ponto de nova análise: sem suporte, ignorado
- Criptografado + compactado: sem suporte, ignorado
- Criptografado + esparso: sem suporte, ignorado
- Fluxo compactado: sem suporte, ignorado
- Fluxo esparso: sem suporte, ignorado

**P13. Qual é o requisito de tamanho mínimo para a pasta de cache?** <br/> R13. O tamanho da pasta de cache determina a quantidade de dados submetida a backup. Sua pasta de cache deve ter 5% do espaço necessário para o armazenamento de dados.

**P14. Se a minha organização tiver um cofre de backup, como posso isolar dados de um servidor de outro servidor ao restaurar os dados?**<br/> R14. Todos os servidores registrados no mesmo cofre poderão recuperar os dados do backup feito por outros servidores *que usem a mesma senha*. Se houver servidores cujos dados de backup que você deseja isolar de outros servidores em sua organização, use uma senha designada para esses servidores. Por exemplo, os servidores de recursos humanos podem usar uma senha de criptografia, os servidores de contabilidade podem usar outra senha e os outros servidores de armazenamento podem usar uma terceira senha.

**P15. Posso "migrar" meus dados de backup entre assinaturas?** <br/> R15: Não

**P16. Posso "migrar" meu cofre de backup entre assinaturas?** <br/> R16: Não. O cofre é criado no nível da assinatura e não pode ser reatribuído a outra assinatura depois de criado.

**P17. O Agente de Backup do Azure funciona em um servidor que usa a eliminação de duplicação do Windows Server 2012?** <br/> R17: Sim. O serviço do agente converte os dados com eliminação de duplicação para dados normais quando prepara a operação de backup. Ele então otimiza os dados para backup, criptografa os dados e envia os dados criptografados para o serviço de backup online.

**P18. Se eu cancelar um trabalho de backup depois de iniciado, os dados de backup transferidos serão excluídos?** <br/> R18: Não. O cofre de backup armazena o backup dos dados transferidos até o ponto do cancelamento. O Backup do Azure usa um mecanismo de ponto de verificação para, ocasionalmente, adicionar pontos de verificação aos dados de backup durante o backup. Como há pontos de verificação nos dados de backup, o próximo processo de backup pode validar a integridade dos arquivos. O próximo backup acionado seria incremental em relação os dados cujo backup foi realizado anteriormente. Um backup incremental fornece uma melhor utilização da largura de banda, para que você não precise transferir os mesmos dados repetidamente.

**P19. Por que vejo o aviso "Não foram configurados Backups do Azure para esse servidor" embora tenha agendado backups regulares anteriormente?** <br/> R19: Esse aviso ocorre quando as configurações de agendamento de backup armazenadas no servidor local não são iguais às configurações armazenadas no cofre de backup. Quando o servidor ou as configurações tiverem sido recuperadas para um bom estado conhecido, os agendamentos de backup podem perder a sincronização. Se você receber esse aviso, [reconfigure a política de backup](backup-azure-manage-windows-server.md) e escolha **Executar o Backup Agora** para sincronizar novamente o servidor local com o Azure.

**P20. Quais regras de firewall devem ser configuradas para o Backup do Azure?** <br/> R20. Para obter proteção contínua de dados no local para o Azure e da carga de trabalho para o Azure, é recomendável permitir que seu firewall se comunique com as seguintes URLs:

- www.msftncsi.com
- *.Microsoft.com
- *.WindowsAzure.com
- *.microsoftonline.com
- *.windows.net

**P21. Posso instalar o agente de Backup do Azure em uma VM do Azure da qual o serviço de Backup do Azure já fez backup usando a extensão de VM?** <br/> R21. Com certeza. O Backup do Azure fornece backup no nível de VM para as máquinas virtuais do Azure usando a extensão de VM. Você pode instalar o agente de Backup do Azure em um sistema operacional Windows Convidado para proteger arquivos e pastas nesse sistema operacional convidado.

**P22. Posso instalar o agente de Backup do Azure em uma VM do Azure para fazer backup de arquivos e pastas presentes no armazenamento temporário fornecido pela VM do Azure?** <br/> R22. Você pode instalar o agente de Backup do Azure no SO convidado do Windows e fazer backup de arquivos e de pastas em um armazenamento temporário. No entanto, observe que os backups falharão assim que os dados do armazenamento temporário forem apagados. Além disso, se os dados de armazenamento temporário tiverem sido excluídos, você só poderá restaurar em um armazenamento não volátil.

**P23. Eu instalei o agente do Backup do Azure para proteger meus arquivos e minhas pastas. Agora posso instalar o SCDPM para trabalhar com o agente do Backup do Azure para proteger as cargas de trabalho do aplicativo/VM local no Azure?** <br/> R23. Para usar o Backup do Azure com o SCDPM, é aconselhável instalar o SCDPM primeiro e só depois instalar o agente do Backup do Azure. Isso garante a integração direta do agente do Backup do Azure com o SCDPM e permite proteger arquivos/pastas, cargas de trabalho de aplicativos e máquinas virtuais no Azure, diretamente do console de gerenciamento do SCDPM. A instalação do SCDPM após a instalação do agente do Backup do Azure para as finalidades mencionadas acima não é aconselhável nem tem suporte.

**P24. Qual é o comprimento do caminho do arquivo que pode ser especificado como parte da política de Backup do Azure usando o agente de Backup do Azure?** <br/> R24. O agente de Backup do Azure baseia-se em NTFS. A [especificação do comprimento do caminho de arquivo é limitada pela API do Windows](https://msdn.microsoft.com/library/aa365247.aspx#fully_qualified_vs._relative_paths). No caso de backup de arquivos com comprimento de caminho de arquivo maior do que aqueles especificados pela API do Windows, os clientes poderão optar por fazer backup da pasta pai ou da unidade de disco dos arquivos de backup.

**P25. Quais caracteres são permitidos no caminho de arquivo da política de Backup do Azure usando o agente de Backup do Azure?** <br> R25. O agente de Backup do Azure baseia-se em NTFS. Ele permite os [caracteres com suporte do NTFS](https://msdn.microsoft.com/library/aa365247.aspx#naming_conventions) como parte da especificação de arquivo.

**P26. Posso usar o servidor de Backup do Azure para criar um backup BMR (Recuperação Bare-Metal) para um servidor físico?** <br/> R26. Sim.

**P27. Posso configurar o serviço de Backup para enviar emails se um trabalho de backup falhar?** <br/> R27. Sim, o serviço de Backup tem vários alertas baseados em eventos que podem ser usados com um script do PowerShell. Para obter uma descrição completa, veja [Notificações de alerta](backup-azure-manage-vms.md#alert-notifications)



## Backup e retenção
**P1. Há um limite para o tamanho de cada fonte de dados submetida a backup?** <br/> R1. A partir de agosto de 2015, o tamanho máximo de fonte de dados para os sistemas operacionais com suporte é:

|S.Não |	Sistema operacional |	Tamanho máximo da fonte de dados |
| :-------------: |:-------------| :-----|
|1| Windows Server 2012 ou superior| 54400 GB|
|2| Windows 8 ou superior| 54400 GB|
|3| Windows Server 2008, Windows Server 2008 R2 | 1700 GB|
|4| Windows 7 | 1700 GB|

A tabela a seguir explica como cada tamanho de fonte de dados é determinado.

|	Fonte de dados |	Detalhes |
| :-------------: |:-------------|
|Volume |A quantidade de dados incluídos no backup do volume único de um servidor ou de uma máquina cliente|
|Máquina virtual do Hyper-V | Soma dos dados de todos os VHDs da máquina virtual da qual está sendo feito o backup|
|Banco de dados do Microsoft SQL Server | O tamanho de um banco de dados SQL único do qual está sendo feito o backup |
|Microsoft SharePoint |A soma dos bancos de dados de conteúdo e de configuração em um farm do SharePoint do qual está sendo feito o backup|
|Microsoft Exchange |Soma de todos os bancos de dados do Exchange em um servidor Exchange do qual está sendo feito o backup|
|Estado do Sistema/BMR |Cada cópia individual do BMR ou do estado do sistema da máquina da qual está sendo feito o backup|

**P2. Há limites para o número de vezes que um backup pode ser agendado por dia?**<br/> R2. Sim, você pode executar os trabalhos de backup no Windows Server ou no cliente Windows até três vezes por dia. Você pode executar trabalhos de backup no System Center DPM até duas vezes por dia. Você pode executar um trabalho de backup para VMs IaaS uma vez por dia.

**P3. Há uma diferença entre a política de agendamento de backup do DPM e do Windows Server (ou seja, no Windows Server sem o DPM)?** <br/> R3. Sim. Usando o DPM, você pode especificar o agendamento diário, semanal, mensal e anual. O Windows Server (sem o DPM) permite que você especifique apenas os agendamentos diários e semanais.

**P4. Há alguma diferença entre a política de retenção do DPM e do Windows Server/cliente Windows (ou seja, no Windows Server sem o DPM)?**<br/> R4. Não, o DPM e o Windows Server/cliente Windows têm políticas de retenção diárias, semanais, mensais e anuais.

**P5. Posso configurar minhas políticas de retenção de forma seletiva – ou seja, configurar semanal e diária, mas não anual e mensal?**<br/> R5. Sim, a estrutura de retenção de Backup do Azure permite que você tenha total flexibilidade na definição da política de retenção de acordo com suas necessidades.

**P6. Posso “agendar um backup” às 18h e especificar “políticas de retenção” em um momento diferente?**<br/> R6. Não. As políticas de retenção só podem ser aplicadas em pontos de backup. Na imagem a seguir, a política de retenção é especificada para backups realizados à meia-noite e às 18h. <br/>

![Retenção e agendamento de Backup](./media/backup-azure-backup-faq/Schedule.png) <br/>

**P7. Uma cópia incremental é transferida de acordo com as políticas de retenção agendadas?** <br/> R7. Não. A cópia incremental é enviada com base na hora mencionada na página de agendamento de backup. Os pontos que podem ser retidos são determinados com base na política de retenção.

**P8. Se um backup for mantido por um longo tempo, levará mais tempo para recuperar um ponto de dados mais antigo?** <br/> R8. Não. O tempo de recuperação do ponto mais antigo ou mais recente é o mesmo. Cada ponto de recuperação se comporta como um ponto completo.

**P9. Se cada ponto de recuperação é como um ponto completo, isso afeta o armazenamento de backup total cobrável?**<br/> R9. Os produtos típicos de ponto de retenção de longo prazo armazenam dados de backup como pontos completos. Os pontos completos *não oferecem economia* de armazenamento, mas são mais fáceis e rápidos de restaurar. As cópias incrementais *oferecem economia* de armazenamento, mas exigem que você restaure uma cadeia de dados, o que afeta o tempo de recuperação. A arquitetura de armazenamento do Backup do Azure oferece o melhor dos dois recursos, armazenando dados de forma otimizada para restaurações rápidas e incorrendo em baixos custos de armazenamento. Essa abordagem de armazenamento de dados garante que a largura de banda de entrada e saída seja usada com eficiência. A quantidade de armazenamento de dados e o tempo necessário para recuperar os dados são mantidos em um mínimo.

**P10. Há um limite para o número de pontos de recuperação que podem ser criados?**<br/> R10. Não. Eliminamos os limites nos pontos de recuperação. Você pode criar quantos pontos de recuperação desejar.

**P11. Por que a quantidade de dados transferida no backup não é igual à quantidade de dados da qual fiz backup?**<br/> A11. Todos os dados que passam por backup são compactados e criptografados antes de serem transferidos. Depois que a compactação e a criptografia forem aplicadas, os dados no cofre de backup serão de 30 a 40% menores.

**P12. Há uma maneira de ajustar a quantidade de largura de banda usada pelo serviço de Backup?**<br/> R12. Sim, use a opção **Alterar Propriedades** no Agente de Backup para ajustar a largura de banda. Ajuste a quantidade de largura de banda e os horários quando você usa essa largura de banda. Veja [Limitação de rede](../backup-configure-vault.md#enable-network-throttling) para saber mais.

**P13. Minha largura de banda de Internet é limitada para a quantidade de dados que preciso incluir no backup. Há uma maneira de mover dados para um determinado local com um pipe de rede grande e de enviar esses dados por push para o Azure?** <br/> R13. Você pode fazer backup de dados no Azure por meio do processo de backup online padrão ou pode usar o serviço Importção/Exportação do Azure para transferir dados para o armazenamento de blobs no Azure. Não há maneiras adicionais de enviar os dados de backup para o armazenamento do Azure. Para saber mais sobre como usar o serviço Importação/Exportação do Azure com o Backup do Azure, confira o artigo [Fluxo de trabalho do backup offline](backup-azure-backup-import-export.md).


## Recuperação
**P1. Quantas recuperações posso executar nos dados incluídos no backup no Azure?**<br/> R1. Não há limite para o número de recuperações do Backup do Azure.

**P2. É necessário pagar pelo tráfego de saída do datacenter do Azure durante as recuperações?**<br/> R2. Não. As suas recuperações são gratuitas e você não é cobrado pelo tráfego de saída.

## Segurança
**P1. Os dados são enviados para o Azure criptografados?** <br/> R1. Sim. Os dados são criptografados localmente no computador cliente/servidor/SCDPM usando AES256 e são enviados por um link HTTPS seguro.

**P2. Os dados de backup também são criptografados no Azure?**<br/> R2. Sim. Os dados enviados para o Azure permanecem criptografados (em repouso). A Microsoft não descriptografa os dados de backup em nenhum momento.

**P3. Qual é o comprimento mínimo da chave de criptografia usada para criptografar os dados de backup?** <br/> R3. A chave de criptografia deve ter pelo menos 16 caracteres.

**P4. O que acontecerá se eu inserir a chave de criptografia incorretamente? Posso recuperar os dados ou a Microsoft pode recuperar os dados?** <br/> R4. A chave usada para criptografar os dados de backup está presente apenas nas instalações do cliente. A Microsoft não mantém uma cópia no Azure e não tem qualquer acesso à chave. Se o cliente inserir a chave incorretamente, a Microsoft não poderá recuperar os dados de backup.

## Cache de backup

**P1. Como posso alterar o local de cache especificado para o agente de Backup do Azure?**<br/> R1. Percorra sequencialmente a lista com marcadores abaixo para alterar o local do cache.
- Pare o mecanismo do Backup ao executar o seguinte comando em um prompt de comando com privilégios elevados:

  ```PS C:\> Net stop obengine```

- Não mova os arquivos. Em vez disso, copie a pasta de espaço de cache para outra unidade com espaço suficiente. O espaço em cache original pode ser removido após a confirmação de que os backups estão funcionando com o novo espaço em cache.

- Atualize as entradas do registro a seguir com o caminho para a nova pasta de espaço em cache.<br/>

|Caminho do registro | Chave do Registro | Valor |
| ------ | ------- | ------|
| `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config` | ScratchLocation | *Novo local da pasta de cache* |
| `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider` | ScratchLocation | *Novo local da pasta de cache* |

- Reinicie o mecanismo do Backup ao executar o seguinte comando em um prompt de comando com privilégios elevados:

  ```PS C:\> Net start obengine```

  Assim que a criação do backup for concluída com êxito no novo local de cache, você poderá remover a pasta de cache original.

**P2. Onde posso colocar a pasta de cache para o agente do Backup do Azure para que ele funcione como esperado?**<br/> R2. Os locais a seguir para a pasta de cache não são recomendados:

- Compartilhamento de rede ou Mídia Removível: a pasta de cache deve ser local para o servidor que precisa de backup usando o backup online. Não há suporte para os locais de rede ou para a mídia removível como unidades USB.
- Volumes Offline: a pasta de cache deve estar online para o backup esperado com o agente do Backup do Azure.

**P3. Existe algum atributo da pasta de cache que não tenha suporte?**<br/> R3. Os atributos a seguir ou suas combinações não têm suporte para a pasta de cache:

- Criptografado
- Eliminação de duplicação
- Compactado
- Esparso
- Ponto de nova análise

É recomendável que nem a pasta de cache nem os metadados de VHD têm os atributos acima para o funcionamento esperado do agente do Backup do Azure.

<!---HONumber=AcomDC_0525_2016-->