<properties
   pageTitle="Carregar dados do armazenamento de blobs do Azure no SQL Data Warehouse (PolyBase) | Microsoft Azure"
   description="Aprenda a usar o PolyBase para carregar dados do armazenamento de blobs do Azure no SQL Data Warehouse. Carregue algumas tabelas de dados públicos no esquema do Data Warehouse de Varejo da Contoso."
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="05/08/2016"
   ms.author="jrj;barbkess;sonyama"/>


# Carregar dados do armazenamento de blobs do Azure no SQL Data Warehouse (PolyBase)

> [AZURE.SELECTOR]
- [Fábrica de dados](sql-data-warehouse-load-from-azure-blob-storage-with-data-factory.md)
- [PolyBase](sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md)

Usar comandos do PolyBase e T-SQL para carregar dados do armazenamento de blobs do Azure no SQL Data Warehouse do Azure.

Para simplificar, este tutorial carrega duas tabelas em um Blob de Armazenamento do Azure público para o esquema do Data Warehouse de Varejo da Contoso. Para carregar o conjunto de dados completo, execute o exemplo [Carregar o Data Warehouse de Varejo completo da Contoso][] pelo repositório de Exemplos do Microsoft SQL Server.

Neste tutorial, você irá:

1. Configurar PolyBase para carregar do armazenamento de blobs do Azure
2. Carregar dados públicos em seu banco de dados
3. Execute otimizações após o carregamento ser concluído.


## Antes de começar
Para executar este tutorial, você precisa de uma conta do Azure que já tenha um banco de dados do SQL Data Warehouse. Se você ainda não tiver uma, consulte [Criar um SQL Data Warehouse][].

## 1\. Configurar a fonte de dados

O PolyBase usa objetos externos do T-SQL para definir o local e os atributos dos dados externos. As definições de objeto externo são armazenadas no SQL Data Warehouse. Os dados em si são armazenados externamente.

### 1\.1. Criar uma credencial

**Ignore esta etapa** se você estiver carregando os dados públicos da Contoso. Você não precisa proteger o acesso aos dados públicos, pois eles já estão acessíveis a qualquer pessoa.

**Não ignore esta etapa** se você estiver usando este tutorial como um modelo para carregar seus próprios dados. Para acessar dados por meio de uma credencial, use o seguinte script para criar uma credencial com escopo de banco de dados e, em seguida, usá-lo ao definir o local da fonte de dados.


```sql
-- A: Create a master key.
-- Only necessary if one does not already exist.
-- Required to encrypt the credential secret in the next step.

CREATE MASTER KEY;


-- B: Create a database scoped credential
-- IDENTITY: Provide any string, it is not used for authentication to Azure storage.
-- SECRET: Provide your Azure storage account key.


CREATE DATABASE SCOPED CREDENTIAL AzureStorageCredential
WITH
    IDENTITY = 'user',
    SECRET = '<azure_storage_account_key>'
;


-- C: Create an external data source
-- TYPE: HADOOP - PolyBase uses Hadoop APIs to access data in Azure blob storage.
-- LOCATION: Provide Azure storage account name and blob container name.
-- CREDENTIAL: Provide the credential created in the previous step.

CREATE EXTERNAL DATA SOURCE AzureStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://<blob_container_name>@<azure_storage_account_name>.blob.core.windows.net',
    CREDENTIAL = AzureStorageCredential
);
```

Pule para a etapa 2.

### 1\.2. Criar a fonte de dados externa

Use este comando [CREATE EXTERNAL DATA SOURCE][] para armazenar o local dos dados e o tipo de dados.

```sql
CREATE EXTERNAL DATA SOURCE AzureStorage_west_public
WITH 
(  
    TYPE = Hadoop 
,   LOCATION = 'wasbs://contosoretaildw-tables@contosoretaildw.blob.core.windows.net/'
); 
```

> [AZURE.IMPORTANT] Se você optar por tornar seus contêineres do armazenamento de blobs do Azure públicos, tenha em mente que, como proprietário dos dados, você será responsável por encargos de saída de dados quando os dados deixarem o datacenter.

## 2\. Configurar o formato de dados

Os dados são armazenados em arquivos de texto no armazenamento de blobs do Azure e cada campo é separado por um delimitador. Execute este comando [CREATE EXTERNAL FILE FORMAT][] para especificar o formato dos dados em arquivos de texto. Os dados da Contoso são descompactados e delimitados por barra vertical.

```sql
CREATE EXTERNAL FILE FORMAT TextFileFormat 
WITH 
(   FORMAT_TYPE = DELIMITEDTEXT
,	FORMAT_OPTIONS	(   FIELD_TERMINATOR = '|'
					,	STRING_DELIMITER = ''
					,	DATE_FORMAT		 = 'yyyy-MM-dd HH:mm:ss.fff'
					,	USE_TYPE_DEFAULT = FALSE 
					)
);
``` 

## 3\. Criar as tabelas externas

Agora que você especificou o formato do arquivo e da fonte de dados, você está pronto para criar as tabelas externas.

### 3\.1. Crie um esquema para os dados. 

Para criar um local para armazenar os dados da Contoso no banco de dados, crie um esquema.

```sql
CREATE SCHEMA [asb]
GO
```

### 3\.2. Crie as tabelas externas. 

Execute este script para criar as tabelas externas DimProduct e FactOnlineSales. Tudo o que estamos fazendo aqui é definir nomes de coluna e tipos de dados e associá-los ao local e ao formato dos arquivos de armazenamento de blobs do Azure. A definição é armazenada no SQL Data Warehouse e os dados ainda estão no Blob de Armazenamento do Azure.

O parâmetro **LOCATION** é a pasta sob a pasta raiz no Blob de Armazenamento do Azure. Cada tabela é em uma pasta diferente.


```sql

--DimProduct
CREATE EXTERNAL TABLE [asb].DimProduct (
	[ProductKey] [int] NOT NULL,
	[ProductLabel] [nvarchar](255) NULL,
	[ProductName] [nvarchar](500) NULL,
	[ProductDescription] [nvarchar](400) NULL,
	[ProductSubcategoryKey] [int] NULL,
	[Manufacturer] [nvarchar](50) NULL,
	[BrandName] [nvarchar](50) NULL,
	[ClassID] [nvarchar](10) NULL,
	[ClassName] [nvarchar](20) NULL,
	[StyleID] [nvarchar](10) NULL,
	[StyleName] [nvarchar](20) NULL,
	[ColorID] [nvarchar](10) NULL,
	[ColorName] [nvarchar](20) NOT NULL,
	[Size] [nvarchar](50) NULL,
	[SizeRange] [nvarchar](50) NULL,
	[SizeUnitMeasureID] [nvarchar](20) NULL,
	[Weight] [float] NULL,
	[WeightUnitMeasureID] [nvarchar](20) NULL,
	[UnitOfMeasureID] [nvarchar](10) NULL,
	[UnitOfMeasureName] [nvarchar](40) NULL,
	[StockTypeID] [nvarchar](10) NULL,
	[StockTypeName] [nvarchar](40) NULL,
	[UnitCost] [money] NULL,
	[UnitPrice] [money] NULL,
	[AvailableForSaleDate] [datetime] NULL,
	[StopSaleDate] [datetime] NULL,
	[Status] [nvarchar](7) NULL,
	[ImageURL] [nvarchar](150) NULL,
	[ProductURL] [nvarchar](150) NULL,
	[ETLLoadID] [int] NULL,
	[LoadDate] [datetime] NULL,
	[UpdateDate] [datetime] NULL
)
WITH
(
    LOCATION='/DimProduct/' 
,   DATA_SOURCE = AzureStorage_west_public
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = VALUE
,   REJECT_VALUE = 0
)
;
 
--FactOnlineSales
CREATE EXTERNAL TABLE [asb].FactOnlineSales 
(
	[OnlineSalesKey] [int]  NOT NULL,
	[DateKey] [datetime] NOT NULL,
	[StoreKey] [int] NOT NULL,
	[ProductKey] [int] NOT NULL,
	[PromotionKey] [int] NOT NULL,
	[CurrencyKey] [int] NOT NULL,
	[CustomerKey] [int] NOT NULL,
	[SalesOrderNumber] [nvarchar](20) NOT NULL,
	[SalesOrderLineNumber] [int] NULL,
	[SalesQuantity] [int] NOT NULL,
	[SalesAmount] [money] NOT NULL,
	[ReturnQuantity] [int] NOT NULL,
	[ReturnAmount] [money] NULL,
	[DiscountQuantity] [int] NULL,
	[DiscountAmount] [money] NULL,
	[TotalCost] [money] NOT NULL,
	[UnitCost] [money] NULL,
	[UnitPrice] [money] NULL,
	[ETLLoadID] [int] NULL,
	[LoadDate] [datetime] NULL,
	[UpdateDate] [datetime] NULL
)
WITH
(
    LOCATION='/FactOnlineSales/' 
,   DATA_SOURCE = AzureStorage_west_public
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = VALUE
,   REJECT_VALUE = 0
)
;
```

## 4\. Carregar os dados
Há diferentes maneiras de acessar dados externos. Você pode consultar dados diretamente da tabela externa, carregar os dados nas novas tabelas de banco de dados ou adicionar dados externos às tabelas de banco de dados existentes.


### 4\.1. Criar um novo esquema

O CTAS cria uma nova tabela que contém dados. Primeiro, crie um esquema para os dados da Contoso.

```sql
CREATE SCHEMA [cso]
GO
```

### 4\.2. Carregar os dados em novas tabelas

Para carregar dados de um armazenamento de blobs do Azure e salvá-lo em uma tabela no banco de dados, use a instrução de T-SQL (CTAS) [CREATE TABLE AS SELECT][]. Carregamento com CTAS utiliza as tabelas externas fortemente tipadas que você acabou de criar. Para carregar os dados em novas tabelas, use uma instrução [CTAS][] por tabela.

O CTAS cria uma nova tabela e a preenche com os resultados de uma instrução select. CTAS define a nova tabela para ter as mesmas colunas e tipos de dados como os resultados da instrução select. Se você selecionar todas as colunas de uma tabela externa, a nova tabela será uma réplica das colunas e dos tipos de dados na tabela externa.

Neste exemplo, criamos a dimensão e a tabela de fatos como tabela distribuídas de hash.


```sql
SELECT GETDATE();
GO

CREATE TABLE [cso].[DimProduct]            WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[DimProduct]             OPTION (LABEL = 'CTAS : Load [cso].[DimProduct]             ');
CREATE TABLE [cso].[FactOnlineSales]       WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[FactOnlineSales]        OPTION (LABEL = 'CTAS : Load [cso].[FactOnlineSales]        ');
```

### 4\.3 Acompanhar o progresso do carregamento

Você pode acompanhar o progresso do carregamento usando a DMV (exibição de gerenciamento dinâmico) `[sys].[dm_pdw_exec_requests]`.

```sql
-- To see all requests
SELECT * FROM sys.dm_pdw_exec_requests;

-- To see a particular request identified by its label
SELECT * FROM sys.dm_pdw_exec_requests as r;
WHERE r.label = 'CTAS : Load [cso].[DimProduct]             '
      OR r.label = 'CTAS : Load [cso].[FactOnlineSales]        '
;
```

## 5\. Otimizar a compactação columnstore

Por padrão, o SQL Data Warehouse armazena a tabela como um índice columnstore clusterizado. Após a conclusão do carregamento, algumas das linhas de dados não podem ser compactadas no columnstore. Há várias razões para isso acontecer. Para obter mais informações, consulte...

Para otimizar o desempenho da consulta e a compactação columnstore após um carregamento, recrie a tabela para forçar o índice columnstore a compactar todas as linhas.

```sql
SELECT GETDATE();
GO

ALTER INDEX ALL ON [cso].[DimProduct]               REBUILD;
ALTER INDEX ALL ON [cso].[FactOnlineSales]          REBUILD;
```

Para obter mais informações sobre como manter os índices columnstore, consulte o artigo [gerenciar índices columnstore][].

## 6\. Otimizar estatísticas

É melhor criar estatísticas de coluna única imediatamente após um carregamento. Há algumas opções de estatísticas. Por exemplo, se você criar estatísticas de coluna única em todas as colunas, poderá levar muito tempo para recompilar todas as estatísticas. Se você souber que determinadas colunas não estarão em predicados de consulta, você poderá ignorar a criação de estatísticas nessas colunas.

Se você optar por criar estatísticas de coluna única em cada coluna de todas as tabelas, poderá usar o exemplo de código do procedimento armazenado `prc_sqldw_create_stats` no artigo [estatísticas][].

O exemplo a seguir é um bom ponto de partida para a criação de estatísticas. Ele cria estatísticas de coluna única em cada coluna na tabela de dimensões e em cada coluna de junção das tabelas de fatos. Você poderá adicionar estatísticas de coluna única ou múltipla às colunas de tabelas de fatos posteriormente.


```sql
CREATE STATISTICS [stat_cso_DimProduct_AvailableForSaleDate] ON [cso].[DimProduct]([AvailableForSaleDate]);
CREATE STATISTICS [stat_cso_DimProduct_BrandName] ON [cso].[DimProduct]([BrandName]);
CREATE STATISTICS [stat_cso_DimProduct_ClassID] ON [cso].[DimProduct]([ClassID]);
CREATE STATISTICS [stat_cso_DimProduct_ClassName] ON [cso].[DimProduct]([ClassName]);
CREATE STATISTICS [stat_cso_DimProduct_ColorID] ON [cso].[DimProduct]([ColorID]);
CREATE STATISTICS [stat_cso_DimProduct_ColorName] ON [cso].[DimProduct]([ColorName]);
CREATE STATISTICS [stat_cso_DimProduct_ETLLoadID] ON [cso].[DimProduct]([ETLLoadID]);
CREATE STATISTICS [stat_cso_DimProduct_ImageURL] ON [cso].[DimProduct]([ImageURL]);
CREATE STATISTICS [stat_cso_DimProduct_LoadDate] ON [cso].[DimProduct]([LoadDate]);
CREATE STATISTICS [stat_cso_DimProduct_Manufacturer] ON [cso].[DimProduct]([Manufacturer]);
CREATE STATISTICS [stat_cso_DimProduct_ProductDescription] ON [cso].[DimProduct]([ProductDescription]);
CREATE STATISTICS [stat_cso_DimProduct_ProductKey] ON [cso].[DimProduct]([ProductKey]);
CREATE STATISTICS [stat_cso_DimProduct_ProductLabel] ON [cso].[DimProduct]([ProductLabel]);
CREATE STATISTICS [stat_cso_DimProduct_ProductName] ON [cso].[DimProduct]([ProductName]);
CREATE STATISTICS [stat_cso_DimProduct_ProductSubcategoryKey] ON [cso].[DimProduct]([ProductSubcategoryKey]);
CREATE STATISTICS [stat_cso_DimProduct_ProductURL] ON [cso].[DimProduct]([ProductURL]);
CREATE STATISTICS [stat_cso_DimProduct_Size] ON [cso].[DimProduct]([Size]);
CREATE STATISTICS [stat_cso_DimProduct_SizeRange] ON [cso].[DimProduct]([SizeRange]);
CREATE STATISTICS [stat_cso_DimProduct_SizeUnitMeasureID] ON [cso].[DimProduct]([SizeUnitMeasureID]);
CREATE STATISTICS [stat_cso_DimProduct_Status] ON [cso].[DimProduct]([Status]);
CREATE STATISTICS [stat_cso_DimProduct_StockTypeID] ON [cso].[DimProduct]([StockTypeID]);
CREATE STATISTICS [stat_cso_DimProduct_StockTypeName] ON [cso].[DimProduct]([StockTypeName]);
CREATE STATISTICS [stat_cso_DimProduct_StopSaleDate] ON [cso].[DimProduct]([StopSaleDate]);
CREATE STATISTICS [stat_cso_DimProduct_StyleID] ON [cso].[DimProduct]([StyleID]);
CREATE STATISTICS [stat_cso_DimProduct_StyleName] ON [cso].[DimProduct]([StyleName]);
CREATE STATISTICS [stat_cso_DimProduct_UnitCost] ON [cso].[DimProduct]([UnitCost]);
CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureID] ON [cso].[DimProduct]([UnitOfMeasureID]);
CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureName] ON [cso].[DimProduct]([UnitOfMeasureName]);
CREATE STATISTICS [stat_cso_DimProduct_UnitPrice] ON [cso].[DimProduct]([UnitPrice]);
CREATE STATISTICS [stat_cso_DimProduct_UpdateDate] ON [cso].[DimProduct]([UpdateDate]);
CREATE STATISTICS [stat_cso_DimProduct_Weight] ON [cso].[DimProduct]([Weight]);
CREATE STATISTICS [stat_cso_DimProduct_WeightUnitMeasureID] ON [cso].[DimProduct]([WeightUnitMeasureID]);
CREATE STATISTICS [stat_cso_FactOnlineSales_CurrencyKey] ON [cso].[FactOnlineSales]([CurrencyKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_CustomerKey] ON [cso].[FactOnlineSales]([CustomerKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_DateKey] ON [cso].[FactOnlineSales]([DateKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_OnlineSalesKey] ON [cso].[FactOnlineSales]([OnlineSalesKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_ProductKey] ON [cso].[FactOnlineSales]([ProductKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_PromotionKey] ON [cso].[FactOnlineSales]([PromotionKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_StoreKey] ON [cso].[FactOnlineSales]([StoreKey]);
```

## Missão cumprida!

Você carregou com êxito os dados públicos no SQL Data Warehouse do Azure. Bom trabalho!

Agora você pode começar a consultar as tabelas usando consultas, como mostrado a seguir:

```sql
SELECT  SUM(f.[SalesAmount]) AS [sales_by_brand_amount]
,       p.[BrandName]
FROM    [cso].[FactOnlineSales] AS f
JOIN    [cso].[DimProduct]      AS p ON f.[ProductKey] = p.[ProductKey]
GROUP BY p.[BrandName]
```

Continue explorando com o SQL Data Warehouse.

## Próximas etapas
Para carregar os dados Data Warehouse de Varejo da Contoso, use o script em Para obter mais dicas de desenvolvimento e consulte a [Visão geral de desenvolvimento do SQL Data Warehouse][].

<!--Image references-->

<!--Article references-->
[Criar um SQL Data Warehouse]: sql-data-warehouse-get-started-provision.md
[Load data into SQL Data Warehouse]: sql-data-warehouse-overview-load.md
[Visão geral de desenvolvimento do SQL Data Warehouse]: sql-data-warehouse-overview-develop.md
[gerenciar índices columnstore]:
[estatísticas]: sql-data-warehouse-develop-statistics.md
[CTAS]: sql-data-warehouse-develop-ctas.md
[label]: sql-data-warehouse-develop-label.md

<!--MSDN references-->
[CRIAR FONTE DE DADOS EXTERNA]: [CRIAR FORMATO DE ARQUIVO EXTERNO]:
[sys.dm_pdw_exec_requests]: https://msdn.microsoft.com/library/mt203887.aspx
[REBUILD]: https://msdn.microsoft.com/library/ms188388.aspx

<!--Other Web references-->
[Microsoft Download Center]: http://www.microsoft.com/download/details.aspx?id=36433
[Carregar o Data Warehouse de Varejo completo da Contoso]: https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/contoso-data-warehouse/readme.md

<!---HONumber=AcomDC_0518_2016-->