<properties
   pageTitle="Introdução ao Microsoft Power BI Embedded Preview"
   description="Power BI Embedded, adicione relatórios interativos do Power BI a seu aplicativo de business intelligence"
   services="power-bi-embedded"
   documentationCenter=""
   authors="minewiskan"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="powerbi"
   ms.date="06/28/2016"
   ms.author="owend"/>

# Introdução ao Microsoft Power BI Embedded Preview

O **Power BI Embedded** é um serviço do Azure que permite que os desenvolvedores de aplicativos adicionem relatórios interativos do Power BI a seus próprios aplicativos. O **Power BI Embedded** funciona com aplicativos existentes sem a necessidade de recriá-los ou alterar o modo de entrada de seus usuários.

Para saber mais sobre o Power BI Embedded, confira [O que é o Power BI Embedded](power-bi-embedded-what-is-power-bi-embedded.md).

Conforme descrito em [O que é o Power BI Embedded](power-bi-embedded-what-is-power-bi-embedded.md), os recursos para **Microsoft Power BI Embedded** são provisionados pelas [ APIs do Azure ARM](https://msdn.microsoft.com/library/mt712306.aspx). Nesse caso, o recurso que você provisiona é uma **Coleção de Espaços de Trabalho do Power BI**. A próxima seção mostra como criar uma coleção de espaços de trabalho.

![](media\power-bi-embedded-get-started\introduction.png)

## Criar uma coleção de espaços de trabalho
Uma **coleção de espaços de trabalho** é o recurso mais avançado do Azure e um contêiner para o conteúdo que será inserido em seu aplicativo. Uma **Coleção de espaços de trabalho** pode ser criada de duas maneiras:

   -	Usando o Portal do Azure manualmente
   -	Programaticamente, usando as APIs do ARM (Azure Resource Manager)

Vamos percorrer as etapas para criar uma **Coleção de espaços de trabalho** usando o Portal do Azure.

   1.	Abra e entre no **Portal do Azure**: [http://portal.azure.com](http://portal.azure.com).

   2.	Clique em **+ Novo** no painel superior.

       ![](media\power-bi-embedded-get-started\create-workspace-1.png)

   3.	Em **Dados + Análise**, clique em **Power BI Embedded**.
   4.	Na **Folha de Criação**, insira as informações necessárias. Para ver **Preços**, confira [Preço do Power BI Embedded](http://go.microsoft.com/fwlink/?LinkID=760527).

       ![](media\power-bi-embedded-get-started\create-workspace-2.png)

   5. Clique em **Criar**.

A **Coleção de espaços de trabalho** levará alguns minutos para provisionar. Quando ela for concluída, você será levado para a **Folha de Coleção de espaços de trabalho**.

   ![](media\power-bi-embedded-get-started\create-workspace-3.png)

Essa **Folha de Criação** contém as informações necessárias para chamar as APIs que criam espaços de trabalho e implanta conteúdo nelas.

A próxima seção descreve como as **Chaves de acesso** são usadas para gerar **tokens do aplicativo** que são usados na autenticação de solicitações de API.

<a name="view-access-keys"/>
## Exibir chaves de acesso da API do Power BI

Uma das informações mais importantes necessárias para chamar as APIs REST do Power BI são as **Chaves de Acesso**. Elas são usadas para gerar os **tokens do aplicativo** que são usados na autenticação das solicitações de API. Para exibir suas **Chaves de Acesso**, clique em **Chaves de Acesso** na **Folha de Configurações**. Para saber mais sobre **tokens do aplicativo**, confira [Como funciona o fluxo de tokens de aplicativo](power-bi-embedded-app-token-flow.md).

   ![](media\power-bi-embedded-get-started\access-keys.png)

Você notará que tem duas chaves.

   ![](media\power-bi-embedded-get-started\access-keys-2.png)

Copie essas chaves e armazene-as com segurança em seu aplicativo. É muito importante tratar essas chaves como faria com uma senha, pois elas dão acesso a todo o conteúdo em sua **Coleção de espaços de trabalho**.

Embora duas chaves estejam listadas, somente uma chave é usada de cada vez. A segunda chave é fornecida para regenerar as chaves periodicamente sem interromper o acesso ao serviço.

Agora que você tem uma instância do Power BI para seu aplicativo e as **Chaves de Acesso**, pode importar um relatório em seu próprio aplicativo. Antes de aprender como importar um relatório, a próxima seção descreve a criação de relatórios e conjuntos de dados do Power BI para inserir em um aplicativo.

## Criar relatórios e conjuntos de dados do Power BI para inserir em um aplicativo

Agora que você criou uma instância do Power BI para seu aplicativo e tem **Chaves de Acesso**, precisa criar os relatórios e conjuntos de dados do Power BI que deseja inserir. Conjuntos de dados e relatórios podem ser criados usando o **Power BI Desktop**. Você pode baixar o [Power BI Desktop gratuitamente](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/). Ou, para começar rapidamente, você pode baixar o [Retail Analysis Sample PBIX](http://go.microsoft.com/fwlink/?LinkID=780547) (PBIX de exemplo de análise de varejo). Para saber mais sobre como usar o **Power BI Desktop**, confira [Introdução ao Power BI Desktop](https://powerbi.microsoft.com/pt-BR/guided-learning/powerbi-learning-0-2-get-started-power-bi-desktop).

Com o **Power BI Desktop**, você se conecta à fonte de dados importando uma cópia dos dados para o **Power BI Desktop** ou se conectando diretamente à fonte de dados usando o **DirectQuery**.

Estas são as diferenças entre o uso de **Importar** e **DirectQuery**.

|Importar | DirectQuery
|---|---
|Tabelas, colunas, *e dados* são importados ou copiados para o **Power BI Desktop**. Ao trabalhar com visualizações, o **Power BI Desktop** consulta uma cópia dos dados. Para ver as alterações ocorridas nos dados subjacentes, você deve atualizar ou importar novamente um conjunto de dados completo e atual.|Somente *tabelas e colunas* são importadas ou copiadas para o **Power BI Desktop**. Ao trabalhar com as visualizações, o **Power BI Desktop** consulta a fonte de dados subjacente, o que significa que você sempre estará exibindo os dados atuais.

Para saber mais sobre como se conectar a uma fonte de dados, confira [Conectar-se a uma fonte de dados](power-bi-embedded-connect-datasource.md).

Depois de salvar seu trabalho no **Power BI Desktop**, um arquivo PBIX será criado. Esse arquivo contém o relatório. Além disso, se você importar dados, o PBIX conterá o conjunto de dados completo, mas se usar o **DirectQuery**, o PBIX conterá apenas um esquema de conjunto de dados. Implante o PBIX em seu espaço de trabalho programaticamente usando a [API de importação do Power BI](https://msdn.microsoft.com/library/mt711504.aspx).

> [AZURE.NOTE] O **Power BI Embedded** tem APIs adicionais para alterar o servidor e o banco de dados para onde seu conjunto de dados está apontando e para definir uma credencial da conta de serviço que o conjunto de dados usará a fim de se conectar ao banco de dados. Confira [Post SetAllConnections](https://msdn.microsoft.com/library/mt711505.aspx) e [Patch Gateway Datasource](https://msdn.microsoft.com/library/mt711498.aspx) (Correção de fonte de dados do gateway).

## Próximas etapas
Nas etapas anteriores, você criou uma coleção de espaços de trabalho, o primeiro relatório e o primeiro conjunto de dados. Agora é hora de aprender a escrever código para o **Power BI Embedded**. Para ajudar você a começar, criamos um aplicativo Web de exemplo: [Introdução a exemplos](power-bi-embedded-get-started-sample.md). O exemplo mostra a você como:

  -	Provisionar conteúdo
      - Criar um espaço de trabalho
      - Importar um arquivo PBIX
      - Atualizar cadeias de conexão e definir suas credenciais para os conjuntos de dados.

  -	Inserir um relatório com segurança

## Consulte também
- [Introdução a exemplos](power-bi-embedded-get-started-sample.md)
- [O que é o Power BI Embedded?](power-bi-embedded-what-is-power-bi-embedded.md)
- [Introdução ao Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-getting-started)
- [Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)
- [Preço do Power BI Embedded](http://go.microsoft.com/fwlink/?LinkID=760527)

<!---HONumber=AcomDC_0629_2016-->