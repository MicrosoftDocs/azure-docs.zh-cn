---
title: 使用索引器连接 Azure SQL 数据库并为其内容编制索引 - Azure 搜索
description: 了解如何使用索引器抓取 Azure SQL 数据库中的数据，以便在 Azure 搜索中进行全文搜索。 本文介绍连接、索引器配置和数据引入。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: c23933e7f379a438d436fd99c5fea7899c5891ef
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025354"
---
# <a name="connect-to-and-index-azure-sql-database-content-using-azure-search-indexers"></a>使用 Azure 搜索索引器连接 Azure SQL 数据库并为其内容编制索引

必须先使用数据填充 [Azure 搜索索引](search-what-is-an-index.md)，然后才能对其进行查询。 如果数据驻留在 Azure SQL 数据库中，则 **Azure SQL 数据库的 Azure 搜索索引器**（或简称 **Azure SQL 索引器**）可自动执行索引编制过程，这意味着需要编写的代码更少且需要考虑的基础结构更少。

本文不但介绍了使用[索引器](search-indexer-overview.md)的机制，而且还介绍了仅适用于 Azure SQL 数据库的功能（如集成的更改跟踪）。 

除了 Azure SQL 数据库之外，Azure 搜索还针对 [Azure Cosmos DB](search-howto-index-cosmosdb.md)、[Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)和 [Azure 表存储](search-howto-indexing-azure-tables.md)提供了索引器。 若要请求对其他数据源的支持，请在 [Azure 搜索反馈论坛](https://feedback.azure.com/forums/263029-azure-search/)上提供反馈。

## <a name="indexers-and-data-sources"></a>索引器和数据源

**数据源**指定要编制索引的数据、用于访问数据的凭据和有效标识数据更改（新行、修改的行或删除的行）的策略。 它定义为独立的资源，以便可供多个索引器使用。

**索引器**是将单个数据源连接到目标搜索索引的资源。 可通过以下方式使用索引器：

* 执行数据的一次性复制以填充索引。
* 按计划使用数据源中的更改更新索引。
* 按需运行以根据需要更新索引。

单个索引器只能使用一个表或视图，但是，如果希望填充多个搜索索引，可以创建多个索引器。 有关概念的详细信息，请参阅[索引器操作：典型工作流](https://docs.microsoft.com/rest/api/searchservice/Indexer-operations#typical-workflow)。

可使用以下内容设置和配置 Azure SQL 索引器：

* [Azure 门户](https://portal.azure.com)中的导入数据向导
* Azure 搜索 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)
* Azure 搜索 [REST API](https://docs.microsoft.com/rest/api/searchservice/indexer-operations)

在本文中，我们将使用 REST API 创建**索引器**和**数据源**。

## <a name="when-to-use-azure-sql-indexer"></a>何时使用 Azure SQL 索引器
根据与数据相关的多个因素，可能适合也可能不适合使用 Azure SQL 索引器。 如果数据符合以下要求，可以使用 Azure SQL 索引器。

| 条件 | 详细信息 |
|----------|---------|
| 数据来自单个表或视图 | 如果数据分散在多个表中，可以创建数据的单一视图。 但是，如果使用视图，则无法使用 SQL Server 集成的更改检测来使用增量更改刷新索引。 有关详细信息，请参阅下文中的[捕获更改和删除的行](#CaptureChangedRows)。 |
| 数据类型是兼容的 | Azure 搜索索引中支持大多数但并非全部 SQL 类型。 有关列表，请参阅[映射数据类型](#TypeMapping)。 |
| 不需要进行实时数据同步 | 索引器最多每五分钟可以为表重新编制索引。 如果数据频繁更改并且所做更改需要在数秒或数分钟内反映在索引中，建议使用 [REST API](https://docs.microsoft.com/rest/api/searchservice/AddUpdate-or-Delete-Documents) 或 [.NET SDK](search-import-data-dotnet.md) 来直接推送更新的行。 |
| 可以进行增量索引编制 | 如果具有大型数据集并打算按计划运行索引器，则 Azure 搜索必须能够有效地标识新的、更改的或删除的行。 只有按需（而非按计划）编制索引时或者为少于 100,000 行的数据编制索引时，才允许非增量索引编制。 有关详细信息，请参阅下文中的[捕获更改和删除的行](#CaptureChangedRows)。 |

> [!NOTE] 
> Azure 搜索仅支持 SQL Server 身份验证。 如果需要支持 Azure Active Directory 密码身份验证，请为此 [UserVoice 建议](https://feedback.azure.com/forums/263029-azure-search/suggestions/33595465-support-azure-active-directory-password-authentica)投票。

## <a name="create-an-azure-sql-indexer"></a>创建 Azure SQL 索引器

1. 创建数据源：

   ```
    POST https://myservice.search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: admin-key

    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "Server=tcp:<your server>.database.windows.net,1433;Database=<your database>;User ID=<your user name>;Password=<your password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;" },
        "container" : { "name" : "name of the table or view that you want to index" }
    }
   ```

   可从 [Azure 门户](https://portal.azure.com)获取连接字符串；请使用 `ADO.NET connection string` 选项。

2. 创建目标 Azure 搜索索引（如果还没有）。 可以使用[门户](https://portal.azure.com)或[创建索引 API](https://docs.microsoft.com/rest/api/searchservice/Create-Index) 创建索引。 确保目标索引的架构与源表的架构兼容 - 请参阅 [SQL 和 Azure 搜索数据类型之间的映射](#TypeMapping)。

3. 通过为索引器命名并引用数据源和目标索引创建索引器：

    ```
    POST https://myservice.search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: admin-key

    {
        "name" : "myindexer",
        "dataSourceName" : "myazuresqldatasource",
        "targetIndexName" : "target index name"
    }
    ```

通过此方式创建的索引器不包含计划。 它会在创建后自动运行一次。 可使用**运行索引器**请求随时再次运行：

    POST https://myservice.search.windows.net/indexers/myindexer/run?api-version=2019-05-06
    api-key: admin-key

可自定义索引器行为的几个方面，例如批大小和可在索引器执行失败前跳过的文档数。 有关详细信息，请参阅[创建索引器 API](https://docs.microsoft.com/rest/api/searchservice/Create-Indexer)。

可能需要允许 Azure 服务连接到数据库。 有关如何执行该操作的说明，请参阅[从 Azure 连接](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)。

若要监视索引器状态和执行历史记录（已编制索引的项目数、失败数等），请使用**索引器状态**请求：

    GET https://myservice.search.windows.net/indexers/myindexer/status?api-version=2019-05-06
    api-key: admin-key

响应应类似于以下形式：

    {
        "\@odata.context":"https://myservice.search.windows.net/$metadata#Microsoft.Azure.Search.V2015_02_28.IndexerExecutionInfo",
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2015-02-21T00:23:24.957Z",
            "endTime":"2015-02-21T00:36:47.752Z",
            "errors":[],
            "itemsProcessed":1599501,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        },
        "executionHistory":
        [
            {
                "status":"success",
                "errorMessage":null,
                "startTime":"2015-02-21T00:23:24.957Z",
                "endTime":"2015-02-21T00:36:47.752Z",
                "errors":[],
                "itemsProcessed":1599501,
                "itemsFailed":0,
                "initialTrackingState":null,
                "finalTrackingState":null
            },
            ... earlier history items
        ]
    }

执行历史记录包含最多 50 个最近完成的执行，它们按反向时间顺序排序（以便最新执行出现在响应中的第一个）。
有关响应的其他信息可在[获取索引器状态](https://go.microsoft.com/fwlink/p/?LinkId=528198)中找到

## <a name="run-indexers-on-a-schedule"></a>按计划运行索引器
还可以排列索引器，以按计划定期运行。 若要执行此操作，在创建或更新索引器时添加**计划**属性。 下面的示例显示了用于更新索引器的 PUT 请求：

    PUT https://myservice.search.windows.net/indexers/myindexer?api-version=2019-05-06
    Content-Type: application/json
    api-key: admin-key

    {
        "dataSourceName" : "myazuresqldatasource",
        "targetIndexName" : "target index name",
        "schedule" : { "interval" : "PT10M", "startTime" : "2015-01-01T00:00:00Z" }
    }

**间隔**参数是必需的。 间隔是指开始两个连续的索引器执行之间的时间。 允许的最小间隔为 5 分钟；最长为一天。 必须将其格式化为 XSD“dayTimeDuration”值（[ISO 8601 持续时间](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration)值的受限子集）。 它的模式为：`P(nD)(T(nH)(nM))`。 示例：`PT15M` 为每隔 15 分钟，`PT2H` 为每隔 2 小时。

可选 **startTime** 指示应该开始计划执行的时间。 如果省略，则使用当前 UTC 时间。 此时间可以是过去时间，在此情况下计划第一个执行，就像索引器在 startTime 之后连续运行一样。  

一次只能运行一个索引器执行。 如果索引器在计划其执行时运行，该执行将推迟到下一个计划时间。

让我们考虑更具体的示例。 假设我们已配置以下每小时计划：

    "schedule" : { "interval" : "PT1H", "startTime" : "2015-03-01T00:00:00Z" }

下面是发生的具体情况：

1. 第一个索引器执行于 2015 年 3 月 1 日午夜 12:00 或大约时间开始。 UTC。
2. 假设此执行需要 20 分钟（或小于 1 小时的任何时间）。
3. 第二个执行于 2015 年 3 月 1 日凌晨 1:00 或大约时间开始。
4. 现在，假设此执行需要超过 1 小时（例如 70 分钟），以便在大约凌晨 2:10 完成。
5. 现在是凌晨 2:00，即将开始第三个执行。 但是，由于从凌晨 1:00 开始的第二个执行 仍在运行，将跳过第三个执行。 第三个执行于凌晨 3:00 开始。

可使用 **PUT 索引器请求**针对现有索引器添加、更改或删除计划。

<a name="CaptureChangedRows"></a>

## <a name="capture-new-changed-and-deleted-rows"></a>捕获新的、更改的和删除的行

Azure 搜索使用“增量索引编制”来避免索引器每次运行时都必须为整个表或视图重新编制索引。 Azure 搜索提供了两个更改检测策略来支持增量索引编制。 

### <a name="sql-integrated-change-tracking-policy"></a>SQL 集成的更改跟踪策略
如果 SQL 数据库支持[更改跟踪](https://docs.microsoft.com/sql/relational-databases/track-changes/about-change-tracking-sql-server)，我们建议使用 **SQL 集成的更改跟踪策略**。 这是最有效的策略。 此外，它允许 Azure 搜索标识删除的行，无需向表中添加显式“软删除”列。

#### <a name="requirements"></a>要求 

+ 数据库版本要求：
  * SQL Server 2012 SP3 及更高版本，如果使用的是 Azure VM 上的 SQL Server。
  * Azure SQL 数据库 V12，如果使用的是 Azure SQL 数据库。
+ 只有表（无视图）。 
+ 在数据库上，为表[启用更改跟踪](https://docs.microsoft.com/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server)。 
+ 表上没有组合主键（包含多个列的主键）。  

#### <a name="usage"></a>使用情况

若要使用此策略，按如下所示创建或更新数据源：

    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "connection string" },
        "container" : { "name" : "table or view name" },
        "dataChangeDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.SqlIntegratedChangeTrackingPolicy"
      }
    }

当使用 SQL 集成的更改跟踪策略时，不指定单独的数据删除检测策略 - 此策略具有对标识删除的行的内置支持。 但是，对于要“自动”检测的删除项，搜索索引中的文档键必须与 SQL 表中的主键相同。 

> [!NOTE]  
> 使用 [TRUNCATE TABLE](https://docs.microsoft.com/sql/t-sql/statements/truncate-table-transact-sql) 从 SQL 表中删除大量行时，需要[重置](https://docs.microsoft.com/rest/api/searchservice/reset-indexer)索引器才能重置更改跟踪状态，从而选取行删除项。

<a name="HighWaterMarkPolicy"></a>

### <a name="high-water-mark-change-detection-policy"></a>高使用标记更改检测策略

此更改检测策略依赖于对版本或行的上次更新时间进行捕获的一个“高使用标记”列。 如果在使用视图，则必须使用高使用标记策略。 高使用标记列必须满足以下要求。

#### <a name="requirements"></a>要求 

* 所有插入都为列指定一个值。
* 对某个项目的所有更新也会更改该列的值。
* 此列的值随每次插入或更新而增加。
* 具有以下 WHERE 和 ORDER BY 子句的查询可以高效执行：`WHERE [High Water Mark Column] > [Current High Water Mark Value] ORDER BY [High Water Mark Column]`

> [!IMPORTANT] 
> 强烈建议为高使用标记列使用 [rowversion](https://docs.microsoft.com/sql/t-sql/data-types/rowversion-transact-sql) 数据类型。 如果使用其他任何数据类型，则当存在与索引器查询并发执行的事务时，不能保证更改跟踪捕获所有更改。 在具有只读副本的配置中使用 **rowversion** 时，必须将索引器指向主副本。 只有主副本可以用于数据同步方案。

#### <a name="usage"></a>使用情况

若要使用高使用标记策略，请按如下所示创建或更新数据源：

    {
        "name" : "myazuresqldatasource",
        "type" : "azuresql",
        "credentials" : { "connectionString" : "connection string" },
        "container" : { "name" : "table or view name" },
        "dataChangeDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
           "highWaterMarkColumnName" : "[a rowversion or last_updated column name]"
      }
    }

> [!WARNING]
> 如果源表在高使用标记列上没有索引，SQL 索引器使用的查询可能会超时。特别是，当表中包含多个行时，`ORDER BY [High Water Mark Column]` 子句需要索引才能有效运行。
>
>

如果遇到超时错误，可以使用 `queryTimeout` 索引器配置设置，查询超时值设置为高于 5 分钟超时的默认值。 例如，要将超时设置为 10 分钟，请使用以下配置创建或更新索引器：

    {
      ... other indexer definition properties
     "parameters" : {
            "configuration" : { "queryTimeout" : "00:10:00" } }
    }

也可以禁用 `ORDER BY [High Water Mark Column]` 子句。 但是，不建议这样做，因为如果索引器执行由于错误而中断，索引器在将来运行时必须重新处理所有行，即使索引器在中断时就处理了几乎所有行也是如此。 若要禁用 `ORDER BY` 子句，使用索引器定义中的 `disableOrderByHighWaterMarkColumn` 设置：  

    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "disableOrderByHighWaterMarkColumn" : true } }
    }

### <a name="soft-delete-column-deletion-detection-policy"></a>软删除列删除检测策略
从源表中删除行时，可能还希望从搜索索引中删除这些行。 如果使用 SQL 集成的更改跟踪策略，此操作会自动完成。 但是，高使用标记更改跟踪策略不会帮助你处理删除的行。 怎么办？

如果以物理方式从表中删除行，Azure 搜索无法推断出不再存在的记录是否存在。  但是，可使用“软删除”技术以逻辑方式删除行，无需从表中删除它们。 将列添加到表或视图，并使用该列将行标记为已删除。

使用软删除技术时，可在创建或更新数据源时，按如下方式指定软删除策略：

    {
        …,
        "dataDeletionDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
           "softDeleteColumnName" : "[a column name]",
           "softDeleteMarkerValue" : "[the value that indicates that a row is deleted]"
        }
    }

**SoftDeleteMarkerValue** 必须是字符串 - 使用实际值的字符串表示形式。 例如，如果有一个整数列（使用值 1 标记删除的行），则使用 `"1"`。 如果有一个 BIT 列（使用布尔值 true 标记删除的行），请使用字符串文本 `True` 或 `true`（不区分大小写）。

<a name="TypeMapping"></a>

## <a name="mapping-between-sql-and-azure-search-data-types"></a>SQL 数据类型与 Azure 搜索数据类型之间的映射
| SQL 数据类型 | 允许的目标索引字段类型 | 说明 |
| --- | --- | --- |
| bit |Edm.Boolean、Edm.String | |
| int、smallint、tinyint |Edm.Int32、Edm.Int64、Edm.String | |
| bigint |Edm.Int64、Edm.String | |
| real、float |Edm.Double、Edm.String | |
| smallmoney、money decimal numeric |Edm.String |Azure 搜索不支持将十进制类型转换为 Edm.Double，因为这可能会损失精度 |
| char、nchar、varchar、nvarchar |Edm.String<br/>集合 (Edm.String) |如果 SQL 字符串表示 JSON 字符串数组，该字符串可用于填充 Collection(Edm.String) 字段：`["red", "white", "blue"]` |
| smalldatetime、datetime、datetime2、date、datetimeoffset |Edm.DateTimeOffset、Edm.String | |
| uniqueidentifer |Edm.String | |
| geography |Edm.GeographyPoint |仅支持具有 SRID 4326（这是默认值）的类型 POINT 的地理实例 |
| rowversion |不适用 |行版本列不能存储在搜索索引中，但可用于更改跟踪 |
| time、timespan、binary、varbinary、image、xml、geometry、CLR 类型 |不适用 |不支持 |

## <a name="configuration-settings"></a>配置设置
SQL 索引器公开多个配置设置：

| 设置 | 数据类型 | 目的 | 默认值 |
| --- | --- | --- | --- |
| queryTimeout |string |设置 SQL 查询执行的超时 |5 分钟（“00:05:00”） |
| disableOrderByHighWaterMarkColumn |bool |导致高使用标记策略使用的 SQL 查询省略 ORDER BY 子句。 请参阅[高使用标记策略](#HighWaterMarkPolicy) |false |

在索引器定义的 `parameters.configuration` 对象中使用这些设置。 例如，要将查询超时设置为 10 分钟，请使用以下配置创建或更新索引器：

    {
      ... other indexer definition properties
     "parameters" : {
            "configuration" : { "queryTimeout" : "00:10:00" } }
    }

## <a name="faq"></a>常见问题解答

**问：是否可以将 Azure SQL 索引器与在 Azure 中 IaaS VM 上运行的 SQL 数据库配合使用？**

是的。 但是，需要允许搜索服务连接到数据库。 有关详细信息，请参阅[配置从 Azure 搜索索引器到 Azure VM 上 SQL Server 的连接](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)。

**问：是否可以将 Azure SQL 索引器与本地运行的 SQL 数据库配合使用？**

无法直接配合使用。 我们不建议使用也不支持直接连接，因为这样做需要使用 Internet 流量打开数据库。 对于此方案，客户已使用诸如 Azure 数据工厂之类的桥技术取得了成功。 有关详细信息，请参阅[使用 Azure 数据工厂将数据推送到 Azure 搜索索引](https://docs.microsoft.com/azure/data-factory/data-factory-azure-search-connector)。

**问：是否可以将 Azure SQL 索引器与在 Azure 上 IaaS 中运行的非 SQL Server 数据库配合使用？**

不。 我们不支持此方案，因为我们尚未使用除 SQL Server 以外的任何数据库测试该索引器。  

**问：是否可以创建多个按计划运行的索引器？**

是的。 但是，一次只能在一个节点上运行一个索引器。 如果需要同时运行多个索引器，请考虑将搜索服务扩展到多个搜索单位。

**问：运行索引器是否会影响我的查询工作负荷？**

是的。 索引器在搜索服务中的一个节点上运行，该节点的资源在编制查询流量索引并进行处理和其他 API 请求之间共享。 如果运行密集型编制索引和查询工作负荷，并频繁遇到 503 错误或响应时间增加，请考虑[纵向扩展搜索服务](search-capacity-planning.md)。

**问：是否可以将[故障转移群集](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)中的次要副本用作数据源？**

视情况而定。 对于表或视图的完整索引编制，可以使用辅助副本。 

对于增量索引编制，Azure 搜索支持两个更改检测策略：SQL 集成的更改跟踪策略和高使用标记策略。

在只读副本上，SQL 数据库不支持集成的更改跟踪。 因此，必须使用高使用标记策略。 

我们的标准建议是为高使用标记列使用 rowversion 数据类型。 但是，使用 rowversion 依赖于 SQL 数据库的 `MIN_ACTIVE_ROWVERSION` 函数，该函数在只读副本上不受支持。 因此，如果使用 rowversion，必须将索引器指向主副本。

如果尝试在只读副本上使用 rowversion，则会看到以下错误： 

    "Using a rowversion column for change tracking is not supported on secondary (read-only) availability replicas. Please update the datasource and specify a connection to the primary availability replica.Current database 'Updateability' property is 'READ_ONLY'".

**问：是否可以使用替代的非 rowversion 列进行高使用标记更改跟踪？**

不建议这样做。 只有 **rowversion** 能够实现可靠的数据同步。 不过，取决于你的应用程序逻辑，如果满足以下条件，则可能也很可靠：

+ 你可以确保当索引器运行时在编制索引的表上没有未完成的事务（例如，所有表更新都按计划作为批处理进行，并且 Azure 搜索索引器计划设置为避免与表更新计划重叠）。  

+ 你定期执行完整重新索引来补充任何缺少的行。 