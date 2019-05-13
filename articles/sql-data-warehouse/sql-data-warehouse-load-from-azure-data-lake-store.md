---
title: 教程：从 Azure Data Lake 存储加载到 Azure SQL 数据仓库 |Microsoft Docs
description: 使用 PolyBase 外部表将数据从 Azure Data Lake 存储加载到 Azure SQL 数据仓库。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: implement
ms.date: 04/26/2019
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 6b5083d6b4cf6758997e4e0551e5f3c2968a31c1
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145972"
---
# <a name="load-data-from-azure-data-lake-storage-to-sql-data-warehouse"></a>Azure Data Lake 存储的数据加载到 SQL 数据仓库
使用 PolyBase 外部表将数据从 Azure Data Lake 存储加载到 Azure SQL 数据仓库。 尽管可以在 Data Lake 存储中存储的数据运行即席查询，我们建议将数据导入 SQL 数据仓库以获得最佳性能。

> [!div class="checklist"]
> * 创建从 Data Lake 存储中加载所需的数据库对象。
> * 连接到 Data Lake 存储目录。
> * 将数据载入 Azure SQL 数据仓库。

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="before-you-begin"></a>开始之前
开始本教程之前，请下载并安装最新版 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。

若要运行本教程，需要：

* Azure Active Directory 应用程序要用于服务到服务身份验证，如果要从 Gen1 加载。 若要创建，请遵循 [Active directory 身份验证](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)

>[!NOTE] 
> 如果要从 Azure 数据湖存储 Gen1 加载，则需要客户端 ID、 密钥和 OAuth2.0 令牌终结点值的 Active Directory 应用程序从 SQL 数据仓库连接到你的存储帐户。 有关如何获取这些值的详细信息可在上面的链接中找到。 对于 Azure Active Directory 应用注册，请使用“应用程序 ID”作为客户端 ID。
> 

* Azure SQL 数据仓库。 请参阅[创建和查询 Azure SQL 数据仓库](create-data-warehouse-portal.md)。

* Data Lake 存储帐户。 请参阅[开始使用 Azure Data Lake 存储](../data-lake-store/data-lake-store-get-started-portal.md)。 

##  <a name="create-a-credential"></a>创建凭据
若要访问你的 Data Lake 存储帐户，你将需要创建数据库主密钥进行加密下一步中使用的凭据密码。 然后创建数据库范围的凭据。 Gen1，对于数据库作用域凭据存储在 AAD 中设置的服务主体凭据。 对于第 2 代，必须在数据库作用域凭据中使用的存储帐户密钥。 

若要连接到 Data Lake Storage Gen1，必须先创建 Azure Active Directory 应用程序，创建访问密钥，并授予应用程序访问 Data Lake Storage Gen1 资源的权限。 有关说明，请参阅[使用 Active Directory 验证 Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。

```sql
-- A: Create a Database Master Key.
-- Only necessary if one does not already exist.
-- Required to encrypt the credential secret in the next step.
-- For more information on Master Key: https://msdn.microsoft.com/library/ms174382.aspx?f=255&MSPPError=-2147217396

CREATE MASTER KEY;


-- B (for Gen1): Create a database scoped credential
-- IDENTITY: Pass the client id and OAuth 2.0 Token Endpoint taken from your Azure Active Directory Application
-- SECRET: Provide your AAD Application Service Principal key.
-- For more information on Create Database Scoped Credential: https://msdn.microsoft.com/library/mt270260.aspx

CREATE DATABASE SCOPED CREDENTIAL ADLSCredential
WITH
    IDENTITY = '<client_id>@<OAuth_2.0_Token_EndPoint>',
    SECRET = '<key>'
;

-- B (for Gen2): Create a database scoped credential
-- IDENTITY: Provide any string, it is not used for authentication to Azure storage.
-- SECRET: Provide your Azure storage account key.

CREATE DATABASE SCOPED CREDENTIAL ADLSCredential
WITH
    IDENTITY = 'user',
    SECRET = '<azure_storage_account_key>'
;

-- It should look something like this for Gen1:
CREATE DATABASE SCOPED CREDENTIAL ADLSCredential
WITH
    IDENTITY = '536540b4-4239-45fe-b9a3-629f97591c0c@https://login.microsoftonline.com/42f988bf-85f1-41af-91ab-2d2cd011da47/oauth2/token',
    SECRET = 'BjdIlmtKp4Fpyh9hIvr8HJlUida/seM5kQ3EpLAmeDI='
;
```

## <a name="create-the-external-data-source"></a>创建外部数据源
使用此 [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql) 命令存储数据的位置。 

```sql
-- C (for Gen1): Create an external data source
-- TYPE: HADOOP - PolyBase uses Hadoop APIs to access data in Azure Data Lake Storage.
-- LOCATION: Provide Data Lake Storage Gen1 account name and URI
-- CREDENTIAL: Provide the credential created in the previous step.

CREATE EXTERNAL DATA SOURCE AzureDataLakeStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'adl://<datalakestoregen1accountname>.azuredatalakestore.net',
    CREDENTIAL = ADLSCredential
);

-- C (for Gen2): Create an external data source
-- TYPE: HADOOP - PolyBase uses Hadoop APIs to access data in Azure Data Lake Storage.
-- LOCATION: Provide Data Lake Storage Gen2 account name and URI
-- CREDENTIAL: Provide the credential created in the previous step.

CREATE EXTERNAL DATA SOURCE AzureDataLakeStorage
WITH (
    TYPE = HADOOP,
    LOCATION='abfs://<container>@<AzureDataLake account_name>.dfs.core.windows.net', -- Please note the abfs endpoint
    CREDENTIAL = ADLSCredential
);
```

## <a name="configure-data-format"></a>配置数据格式
若要从 Data Lake 存储导入数据，需要指定外部文件格式。 此对象用于定义如何将这些文件写入 Data Lake 存储中。
有关完整列表，请查看我们的 T-SQL 文档 [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql)（创建外部文件格式）

```sql
-- D: Create an external file format
-- FIELD_TERMINATOR: Marks the end of each field (column) in a delimited text file
-- STRING_DELIMITER: Specifies the field terminator for data of type string in the text-delimited file.
-- DATE_FORMAT: Specifies a custom format for all date and time data that might appear in a delimited text file.
-- Use_Type_Default: Store missing values as default for datatype.

CREATE EXTERNAL FILE FORMAT TextFileFormat
WITH
(   FORMAT_TYPE = DELIMITEDTEXT
,    FORMAT_OPTIONS    (   FIELD_TERMINATOR = '|'
                    ,    STRING_DELIMITER = ''
                    ,    DATE_FORMAT         = 'yyyy-MM-dd HH:mm:ss.fff'
                    ,    USE_TYPE_DEFAULT = FALSE
                    )
);
```

## <a name="create-the-external-tables"></a>创建外部表
指定数据源和文件格式后，可以开始创建外部表。 外部表是你与外部数据交互的方式。 位置参数可以指定文件或目录。 如果指定目录，将会加载该目录中的所有文件。

```sql
-- D: Create an External Table
-- LOCATION: Folder under the Data Lake Storage root folder.
-- DATA_SOURCE: Specifies which Data Source Object to use.
-- FILE_FORMAT: Specifies which File Format Object to use
-- REJECT_TYPE: Specifies how you want to deal with rejected rows. Either Value or percentage of the total
-- REJECT_VALUE: Sets the Reject value based on the reject type.

-- DimProduct
CREATE EXTERNAL TABLE [dbo].[DimProduct_external] (
    [ProductKey] [int] NOT NULL,
    [ProductLabel] [nvarchar](255) NULL,
    [ProductName] [nvarchar](500) NULL
)
WITH
(
    LOCATION='/DimProduct/'
,   DATA_SOURCE = AzureDataLakeStorage
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = VALUE
,   REJECT_VALUE = 0
)
;

```

## <a name="external-table-considerations"></a>外部表注意事项
创建外部表很容易，但有一些细微差别，需要进行讨论。

外部表经过强类型化。 这意味着所引入的每行数据必须满足表架构定义。
如果某个行不符合架构定义，则该行会被拒绝加载。

使用 REJECT_TYPE 和 REJECT_VALUE 可以定义在最终表中必须存在的数据行数或数据的百分比。 在加载过程中，如果达到拒绝值，则加载会失败。 行被拒绝的最常见原因是架构定义不匹配。 例如，当文件中的数据是字符串时，如果错误地为列指定了 int 的架构，则每一行都将无法加载。

Data Lake Storage Gen1 使用基于角色的访问控制 (RBAC) 控制对数据的访问。 也就是说，服务主体必须拥有对位置参数中定义的目录以及最终目录和文件的子项的读取权限。 这样一来，PolyBase 就可以进行身份验证，并加载该数据。 

## <a name="load-the-data"></a>加载数据
将数据从 Data Lake 存储使用加载[CREATE TABLE AS SELECT (TRANSACT-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse)语句。 

CTAS 将创建新表，并在该表中填充 select 语句的结果。 CTAS 将新表定义为包含与 select 语句结果相同的列和数据类型。 如果选择了外部表中的所有列，则新表将是外部表中的列和数据类型的副本。

在此示例中，我们将从外部表 DimProduct_external 创建名为 DimProduct 的哈希分布表。

```sql

CREATE TABLE [dbo].[DimProduct]
WITH (DISTRIBUTION = HASH([ProductKey]  ) )
AS
SELECT * FROM [dbo].[DimProduct_external]
OPTION (LABEL = 'CTAS : Load [dbo].[DimProduct]');
```


## <a name="optimize-columnstore-compression"></a>优化列存储压缩
默认情况下，SQL 数据仓库将表存储为聚集列存储索引。 加载完成后，某些数据行可能未压缩到列存储中。  发生这种情况的原因多种多样。 若要了解详细信息，请参阅[管理列存储索引](sql-data-warehouse-tables-index.md)。

若要在加载后优化查询性能和列存储压缩，请重新生成表，以强制列存储索引压缩所有行。

```sql

ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD;

```

## <a name="optimize-statistics"></a>优化统计信息
最好是在加载之后马上创建单列统计信息。 对于统计信息，可以使用多个选项。 例如，如果针对每个列创建单列统计信息，则重新生成所有统计信息可能需要花费很长时间。 如果知道某些列不会在查询谓词中使用，可以不创建有关这些列的统计信息。

如果决定针对每个表的每个列创建单列统计信息，可以使用 [statistics](sql-data-warehouse-tables-statistics.md)（统计信息）一文中的存储过程代码示例 `prc_sqldw_create_stats`。

以下示例是创建统计信息的不错起点。 它会针对维度表中的每个列以及事实表中的每个联接列创建单列统计信息。 以后，随时可以将单列或多列统计信息添加到其他事实表列。

## <a name="achievement-unlocked"></a>大功告成！
已成功地将数据载入 Azure SQL 数据仓库。 干得不错！

## <a name="next-steps"></a>后续步骤 
在本教程中，创建了外部表以定义 Data Lake Storage Gen1 中存储的数据的结构，然后使用了 PolyBase CREATE TABLE AS SELECT 语句将数据加载到数据仓库。 

完成了以下操作：
> [!div class="checklist"]
> * 创建需要从 Data Lake Storage Gen1 加载的数据库对象。
> * 连接到 Data Lake Storage Gen1 目录。
> * 将数据加载到了 Azure SQL 数据仓库。
> 

加载数据是使用 SQL 数据仓库开发数据仓库解决方案的第一步。 请查看我们的开发资源。

> [!div class="nextstepaction"]
>[了解如何在 SQL 数据仓库中开发表](sql-data-warehouse-tables-overview.md)




