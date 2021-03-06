<properties
   pageTitle="Backup e restauração do Service Fabric | Microsoft Azure"
   description="Documentação conceitual de backup e restauração do Service Fabric"
   services="service-fabric"
   documentationCenter=".net"
   authors="mcoskun"
   manager="timlt"
   editor="subramar,jessebenson"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/13/2016"
   ms.author="mcoskun"/>

# Fazer backup e restaurar Reliable Services e Reliable Actors

O Azure Service Fabric é uma plataforma de alta disponibilidade que replica o estado em vários nós a fim de manter essa alta disponibilidade. Portanto, mesmo se um nó do cluster falhar, os serviços continuarão disponíveis. Embora essa redundância interna fornecida pela plataforma possa ser suficiente para algumas pessoas, em certos casos é recomendável fazer o backup dos dados do serviço (em um repositório externo).

>[AZURE.NOTE] É essencial fazer backup e restaurar seus dados (e testar se funcionam conforme esperado) para que você possa se recuperar de cenários de perda de dados.

Por exemplo, convém fazer o backup dos dados de um serviço nos seguintes cenários:

* No caso de perda permanente de um cluster inteiro do Service Fabric ou de todos os nós que estão em execução em uma determinada partição.

* Erros administrativos nos quais o estado é acidentalmente excluído ou corrompido. Por exemplo, isso pode ocorrer quando um administrador com privilégios suficientes exclui o serviço por engano.

* Bugs no serviço que venham a corromper os dados. Por exemplo, isso pode ocorrer quando uma atualização de código de serviço inicia a gravação de dados com falha em uma coleção confiável. Nesse caso, convém reverter o código e os dados para um estado anterior.

* Processamento de dados offline. Talvez seja conveniente processar no modo offline os dados de business intelligence que ocorrem separadamente do serviço que gera os dados.

O recurso de Backup/Restauração permite que os serviços criados na API de Reliable Services criem e restaurem os backups. As APIs de backup fornecidas pela plataforma permitem fazer backups do estado de uma partição de serviço, sem bloquear operações de leitura ou gravação. As APIs de restauração permitem que o estado de uma partição de serviço seja restaurado de um backup escolhido.

## Tipos de Backup

Há duas opções de backup: Completo e Incremental. Um backup completo é um backup que contém todos os dados necessários para recriar o estado da réplica: pontos de verificação e todos os registros de log. Como ele tem os pontos de verificação e o log, um backup completo pode ser restaurado por si só.

O problema com backups completos surge quando os pontos de verificação são grandes. Por exemplo, uma réplica que tem 16 GB de estado terá pontos de verificação que podem chegar a aproximadamente 16 GB. Caso tenhamos um Objetivo de Ponto de Recuperação de 5 minutos, deve-se fazer backup da máquina a cada 5 minutos. Cada vez que ele fizer o backup, será preciso copiar 16 GB de pontos de verificação além de 50 MB (configurável usando **CheckpointThresholdInMB**) em logs.

![Exemplo de Backup Completo.](media/service-fabric-reliable-services-backup-restore/FullBackupExample.PNG)

A solução para esse problema é o backup incremental, em que apenas os registros de log desde o último backup têm backup realizado.

![Exemplo de Backup Incremental.](media/service-fabric-reliable-services-backup-restore/IncrementalBackupExample.PNG)

Como os backups incrementais são apenas as alterações desde o último backup (não incluem os pontos de verificação), eles tendem a ser mais rápidos, mas não podem ser restaurados por conta própria. Para restaurar um backup incremental, a cadeia de backup inteira será necessária. Uma cadeia de backup é uma cadeia de backups, começando com um backup completo e seguido por um número de backups incrementais contíguos.

## Fazer backup de Reliable Services

O autor do serviço tem controle total sobre quando realizar backups e onde os backups serão armazenados.

Para iniciar um backup, o serviço precisará invocar a função de membro herdado **BackupAsync**. Os backups somente podem ser realizados a partir de réplicas primárias e exigem a concessão do status de gravação.

Conforme mostrado abaixo, o **BackupAsync** captura um objeto **BackupDescription**, em que é possível especificar um backup completo ou incremental, bem como uma função de retorno de chamada, **Func<< BackupInfo, CancellationToken, Task<bool>>>** que será invocada quando a pasta de backup tiver sido criada localmente e estiver pronta para ser movida para algum armazenamento externo.

```C#

BackupDescription myBackupDescription = new BackupDescription(backupOption.Incremental,this.BackupCallbackAsync);

await this.BackupAsync(myBackupDescription);

```

A solicitação para fazer um backup incremental pode falhar com **FabricFullBackupMissingException** que indica que a réplica nunca teve um backup completo ou alguns dos registros de log desde que esse último backup foi truncado. Os usuários podem modificar a taxa de truncamento ao modificar o **CheckpointThresholdInMB**.

O **BackupInfo** fornece informações sobre o backup, incluindo o local da pasta na qual o tempo de execução salvou o backup (**BackupInfo.Directory**). A função de retorno de chamada pode mover o **BackupInfo.Directory** para um repositório externo ou outro local. Além disso, esta função retorna um booliano que indica se ele foi capaz de mover a pasta de backup para seu local de destino.

O código a seguir demonstra como o método **BackupCallbackAsync** pode ser usado para carregar o backup no Armazenamento do Azure:

```C#
private async Task<bool> BackupCallbackAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
{
    var backupId = Guid.NewGuid();

    await externalBackupStore.UploadBackupFolderAsync(backupInfo.Directory, backupId, cancellationToken);

    return true;
}
```

No exemplo acima, **ExternalBackupStore** é a classe de exemplo usada para realizar a interface com o Armazenamento de Blobs do Azure e **UploadBackupFolderAsync** é o método que compacta a pasta e a coloca no armazenamento de Blobs do Azure.

Observe que:

- Pode haver apenas uma operação de backup em execução por réplica em um determinado momento. Mais de uma chamada de **BackupAsync** por vez lançará **FabricBackupInProgressException** para limitar a execução de backups para apenas uma.

- Se uma réplica passar por failover enquanto um backup estiver em andamento, talvez o backup não tenha sido concluído. Portanto, após a conclusão do failover, é responsabilidade do serviço reiniciar o backup invocando **BackupAsync**, conforme necessário.

## Restaurar Reliable Services

Em geral, os casos em que você talvez precise executar uma operação de restauração se enquadram em uma destas categorias:

- A partição do serviço perdeu os dados. Por exemplo, o disco de duas entre três réplicas de uma partição (incluindo a réplica primária) é corrompido ou apagado. Pode ser necessário que a nova réplica primária restaure os dados a partir de um backup.

- O serviço inteiro é perdido. Por exemplo, um administrador remove todo o serviço e, portanto, o serviço e os dados precisam ser restaurados.

- O serviço replica dados corrompidos de aplicativo (por exemplo, devido a um bug de aplicativo). Nesse caso, o serviço precisa ser atualizado ou revertido para remover a causa do dano, e os dados não corrompidos precisam ser restaurados.

Embora muitas abordagens sejam possíveis, oferecemos alguns exemplos sobre como usar **RestoreAsync** para se recuperar dos cenários acima.

## Perda de dados de partição em Reliable Services

Nesse caso, o tempo de execução detectaria automaticamente a perda de dados e chamaria a API **OnDataLossAsync**.

O autor do serviço precisa executar o seguinte para fazer a recuperação:

- Substitua o método da classe base virtual **OnDataLossAsync**.

- Localize o último backup no local externo que contém backups do serviço.

- Baixe o backup mais recente (e descompacte o backup na pasta de backup se ele tiver sido compactado).

- O método **OnDataLossAsync** fornece um **RestoreContext**. Chame a API **RestoreAsync** no **RestoreContext** fornecido.

- Retorna verdadeiro se a restauração foi um sucesso.

Veja a seguir um exemplo de implementação do método **OnDataLossAsync**:

```C#

protected override async Task<bool> OnDataLossAsync(RestoreContext restoreCtx, CancellationToken cancellationToken)
{
    var backupFolder = await this.externalBackupStore.DownloadLastBackupAsync(cancellationToken);

    var restoreDescription = new RestoreDescription(backupFolder);

    await restoreCtx.RestoreAsync(restoreDescription);

    return true;
}
```

**RestoreDescription** passados para a chamada **RestoreContext.RestoreAsync** contém um membro chamado **BackupFolderPath**. Ao restaurar um único backup completo, o **BackupFolderPath** deve ser definido como o caminho local da pasta que contém o backup completo. Ao restaurar um backup completo e um número de backups incrementais, **BackupFolderPath** deve ser definido como o caminho local da pasta que contém não apenas o backup completo, mas também todos os backups incrementais. A chamada **RestoreAsync** pode lançar **FabricFullBackupMissingException** se o **BackupFolderPath** fornecido não contém um backup completo. Ele também pode lançar **ArgumentException** se **BackupFolderPath** tem uma cadeia quebrada de backups incrementais. Por exemplo, se ele contiver o backup completo, o primeiro e o terceiro backup incremental, mas não o segundo backup incremental.

>[AZURE.NOTE] O RestorePolicy é definido como Seguro por padrão. Isso significa que a API **RestoreAsync** falhará com ArgumentException se detectar que a pasta de backups contém um estado mais antigo ou igual ao estado contido nesta réplica. **RestorePolicy.Force** pode ser usado para ignorar a verificação de segurança. Isso é especificado como parte de **RestoreDescription**.

## Serviço perdido ou excluído

Se um serviço for removido, primeiro recrie o serviço antes de restaurar os dados. É importante criar o serviço com a mesma configuração, por exemplo, esquema de particionamento, para que os dados possam ser restaurados perfeitamente. Quando o serviço estiver funcionando, a API para restauração dos dados (**OnDataLossAsync** acima) precisará ser chamada em todas as partições desse serviço. Uma maneira de conseguir isso é usando **[FabricClient.TestManagementClient.StartPartitionDataLossAsync](https://msdn.microsoft.com/library/mt693569.aspx)** em cada partição.

Neste ponto, a implementação é igual à do cenário anterior. Cada partição deve restaurar o backup mais recente relevante do armazenamento externo. Uma limitação é que a ID de partição pode ter sido alterada, uma vez que o tempo de execução cria IDs de partição dinamicamente. Portanto, o serviço precisa armazenar o nome do serviço e as informações de partição apropriadas para identificar o backup correto mais recente para restauração em cada partição.

>[AZURE.NOTE] Não é recomendável usar **FabricClient.ServiceManager.InvokeDataLossAsync** em cada partição para restaurar o serviço inteiro, desde que pode corromper o estado do cluster.

## Replicação de dados de aplicativo corrompidos

Se a atualização de aplicativo implantado recentemente tiver um bug, poderá causar corrupção de dados. Por exemplo, uma atualização de aplicativo pode começar a atualizar todos os registros de número de telefone em um dicionário confiável com um código de área inválido. Nesse caso, os números de telefone inválidos serão replicados, uma vez que o Service Fabric não tem conhecimento da natureza dos dados que estão sendo armazenados.

A primeira coisa a ser feita depois de detectar um bug tão sério e que causa a corrupção de dados é congelar o serviço no nível do aplicativo e, se possível, atualizar para a versão do código do aplicativo que não tenha o bug. No entanto, mesmo depois que o código do serviço é corrigido, os dados ainda podem estar corrompidos e, portanto, talvez seja necessário restaurá-los. Nesses casos, talvez não seja suficiente restaurar o backup mais recente, uma vez que os backups mais recentes também podem estar corrompidos. Assim, você precisa localizar o backup mais recente realizado antes de os dados serem corrompidos.

Caso não saiba ao certo quais backups estão corrompidos, implemente um novo cluster do Service Fabric e restaure os backups das partições afetadas, assim como o cenário descrito anteriormente para "Serviço excluído ou interrompido". Para cada partição, comece a restaurar os backups a partir do mais recente. Caso encontre um backup sem danos, mova ou exclua todos os outros backups mais recentes dessa partição. Repita esse processo para cada partição. Agora, quando **OnDataLossAsync** for chamado na partição do cluster de produção, o último backup localizado no armazenamento externo será aquele escolhido de acordo com o processo acima.

Agora, as etapas na seção "Serviço perdido ou excluído" podem ser usadas para restaurar o estado do serviço para o estado anterior à corrupção do estado pelo código com bug.

Observe que:

- Sempre que você restaura, há uma chance do backup restaurado ser mais antigo do que o estado da partição antes de os dados serem perdidos. Por isso, essa restauração deve ser usada apenas como último recurso para recuperar o máximo possível de dados.

- A cadeia de caracteres que representa o caminho da pasta de backup e os caminhos dos arquivos dentro da pasta de backup podem ser maiores do que 255 caracteres, dependendo do caminho de FabricDataRoot e do comprimento do nome do Tipo de Aplicativo. Isso pode fazer com que alguns métodos .NET, como **Directory.Move**, lancem a exceção **PathTooLongException**. Uma solução alternativa é chamar diretamente as APIs do kernel32 como **CopyFile**.

## Fazer backup e restaurar Reliable Actors

Faça backup e restaure Reliable Actors criados na funcionalidade de backup e restauração fornecida pelos serviços confiáveis. O proprietário do serviço deve criar um serviço de Ator personalizado que derive de **ActorService** (que é um serviço confiável do Service Fabric que hospeda atores) e, em seguida, fazer backup/restauração semelhante para os Reliable Services, como descrito acima nas seções anteriores. Uma vez que os backups serão usados por partição, será feito backup dos estados de todos os atores nessa partição específica (e a restauração é semelhante e acontecerá de acordo com a partição).


- Quando você cria um serviço de ator personalizado, é preciso registrar o serviço de ator personalizado também enquanto registra o ator. Confira **ActorRuntime.RegistorActorAsync**.
- Atualmente, **KvsActorStateProvider** dá suporte apenas para backups completos. A opção **RestorePolicy.Safe** também é ignorada por **KvsActorStateProvider**.

>[AZURE.NOTE] O padrão ActorStateProvider (ou seja, **KvsActorStateProvider**) **não** limpa as pastas de backup por si só (sob a pasta de trabalho de aplicativo obtido por meio de ICodePackageActivationContext.WorkDirectory). Isso pode fazer com que a pasta de trabalho seja preenchida. Você deve limpar explicitamente a pasta de backup no retorno de chamada de backup depois que você mover o backup para um armazenamento externo de limpeza.


## Testando o backup e a restauração

É importante garantir que o backup dos dados críticos esteja sendo feito e que os dados possam ser restaurados desse backup. Isso pode ser feito invocando o cmdlet **Invoke-ServiceFabricPartitionDataLoss** no PowerShell, que pode induzir a perda de dados em uma determinada partição para testar se a funcionalidade de backup e restauração dos dados para seu serviço está funcionando conforme o esperado. Também é possível invocar de modo programático a perda de dados e fazer a restauração a partir desse evento.

>[AZURE.NOTE] Você pode encontrar um exemplo de implementação da funcionalidade de backup e restauração no Aplicativo de Referência da Web no Github. Examine o serviço Inventory.Service para obter mais detalhes.

## Nos bastidores: mais detalhes sobre backup e restauração

Veja mais alguns detalhes sobre backup e restauração.

### Backup
O Gerenciador de Estado Confiável permite a criação de backups consistentes sem bloquear as operações de leitura e gravação. Para fazer isso, ele utiliza um mecanismo de ponto de verificação e persistência de log. O Gerenciador de Estado Confiável usa pontos de verificação (leves) difusos em determinados pontos para aliviar a pressão do log transacional e melhorar os tempos de recuperação. Quando **BackupAsync** é chamado, o Gerenciador de Estado Confiável instrui todos os objetos Reliable a copiar seus arquivos de ponto de verificação mais recentes em uma pasta de backup local. Em seguida, o Gerenciador de Estado Confiável copia todos os registros de log, desde o "ponteiro inicial" até o registro de log mais recente, na pasta de backup. Como todos os registros de log, até o último, são incluídos no backup, e o Gerenciador de Estado Confiável preserva o registro em log write-ahead, o Gerenciador de Estado Confiável garante que todas as transações confirmadas (**CommitAsync** retornou com êxito) sejam incluídas no backup.

As transações confirmadas após o **BackupAsync** ser chamado podem ou não estar no backup. Após o preenchimento da pasta de backup local pela plataforma (ou seja, o backup local é concluído pelo tempo de execução), o retorno de chamada de backup do serviço é invocado. Esse retorno de chamada é responsável por mover a pasta de backups para um local externo, por exemplo, o Armazenamento do Azure.

### Restaurar

O Gerenciador de Estado Confiável permite a restauração de um backup usando a API **RestoreAsync**. O método **RestoreAsync** em **RestoreContext** só pode ser chamado no método **OnDataLossAsync**. O bool retornado por **OnDataLossAsync** indica se o serviço restaurou seu estado a partir de uma fonte externa. Se **OnDataLossAsync** retornar true, o Service Fabric recompilará todas as outras réplicas por meio da primária. O Service Fabric garante que as réplicas que receberão **OnDataLossAsync** chamem a primeira transição para a função primária, embora não recebam status de leitura ou de gravação. Isso significa que para os implementadores de StatefulService, o **RunAsync** não será chamado até que **OnDataLossAsync** seja concluído com êxito. Em seguida, **OnDataLossAsync** será invocado na nova primária. A API continua sendo chamada, até que um serviço conclua essa API com êxito, retornando verdadeiro ou falso, e conclua a reconfiguração relevante.

Primeiro, **RestoreAsync** descarta todos os estados existentes na réplica primária na qual foi chamado. Depois, o Gerenciador de Estado Confiável cria todos os objetos Reliable que existem na pasta de backup. Em seguida, os objetos Reliable são instruídos a restaurar a partir dos pontos de verificação na pasta de backup. Finalmente, o Gerenciador de Reliable State recupera seu próprio estado a partir dos registros de log na pasta de backup e executa a recuperação. Como parte do processo de recuperação, as operações que começaram do "ponto de partida" e confirmaram os registros de log na pasta de backup são reproduzidas aos objetos Reliable. Essa etapa garante que o estado recuperado seja consistente.

<!---HONumber=AcomDC_0518_2016-->