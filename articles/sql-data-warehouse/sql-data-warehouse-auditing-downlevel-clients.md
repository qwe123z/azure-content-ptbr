<properties
   pageTitle="Suporte a clientes de versão anterior do SQL Data Warehouse para auditoria de dados | Microsoft Azure"
   description="Saiba mais sobre o suporte a clientes de versão anterior do SQL Data Warehouse para auditoria de dados"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.workload="data-management"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="05/31/2016" 
   ms.author="rortloff;barbkess;sonyama"/>

# SQL Data Warehouse - Suporte a clientes de versão anterior para auditoria e Mascaramento dinâmico de dados


A [Auditoria](sql-data-warehouse-auditing-overview.md) funciona com clientes SQL que oferecem suporte ao redirecionamento de TDS.

Qualquer cliente que implemente o protocolo TDS 7.4 também deve dar suporte a redirecionamento. As exceções incluem o JDBC 4.0, no qual o recurso de redirecionamento não tem suporte completo, e o Tedious para Node.JS, no qual o redirecionamento não foi implementado.

Para "clientes de versão anterior", ou seja, que oferecem suporte ao TDS versão 7.3 e inferior - o FQDN do servidor na cadeia de conexão deve ser modificado:

FQDN original do servidor na cadeia de conexão: <*nome do servidor*>.database.windows.net

FQDN do servidor modificado na cadeia de conexão: <*nome do servidor*>.database.**secure**.windows.net

Uma lista parcial de "Clientes de versão anterior" inclui:

- .NET 4.0 e inferior,
- ODBC 10.0 e inferior.
- JDBC (embora o JDBC dê suporte ao TDS 7.4, o recurso de redirecionamento de TDS não recebe suporte total)
- Tedious (para o Node.JS)

**Comentário:** a modificação do FQDN do servidor acima pode ser útil também para aplicar uma política de Auditoria no Nível do SQL Server sem a necessidade de uma etapa de configuração em cada banco de dados (redução temporária).

<!---HONumber=AcomDC_0601_2016-->