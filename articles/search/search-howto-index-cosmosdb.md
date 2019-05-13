---
title: 为 Azure Cosmos DB 数据源编制索引 - Azure 搜索
description: 在 Azure 搜索中抓取 Azure Cosmos DB 数据源并通过全文可搜索索引引入数据。 索引器可自动为所选数据源（如 Azure Cosmos DB）引入数据。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: d10a1df402fc4931c4d6cc513aa5e22cfe7ec2ba
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024716"
---
# <a name="how-to-index-cosmos-db-using-an-azure-search-indexer"></a>如何使用 Azure 搜索索引器的 Cosmos DB 索引

本文介绍如何配置 Azure Cosmos DB[索引器](search-indexer-overview.md)提取内容并使其可在 Azure 搜索中搜索。 此工作流创建 Azure 搜索索引并将其加载与从 Azure Cosmos DB 中提取的现有文本。 

术语可能会造成混淆，因为它是值得注意的是[Azure Cosmos DB 索引](https://docs.microsoft.com/azure/cosmos-db/index-overview)并[Azure 搜索索引](search-what-is-an-index.md)是不同的操作，唯一的每个服务。 在开始 Azure 搜索之前索引编制，Azure Cosmos DB 数据库必须已经存在并包含数据。

可以使用[门户](#cosmos-indexer-portal)，REST Api 或.NET SDK Cosmos 内容编制索引。 Cosmos DB 索引器在 Azure 搜索可爬网[Azure Cosmos 项](https://docs.microsoft.com/azure/cosmos-db/databases-containers-items#azure-cosmos-items)通过这些协议访问：

* [SQL API](https://docs.microsoft.com/azure/cosmos-db/sql-api-query-reference) 
* [MongoDB API](https://docs.microsoft.com/azure/cosmos-db/mongodb-introduction) （此 API 的 Azure 搜索支持为公共预览版）  

> [!Note]
> 用户之声有更多 API 支持的现有项。 可以强制转换为你想要查看 Azure 搜索中受支持 Cosmos Api 投票：[将表 API](https://feedback.azure.com/forums/263029-azure-search/suggestions/32759746-azure-search-should-be-able-to-index-cosmos-db-tab)，[图形 API](https://feedback.azure.com/forums/263029-azure-search/suggestions/13285011-add-graph-databases-to-your-data-sources-eg-neo4)， [Apache Cassandra API](https://feedback.azure.com/forums/263029-azure-search/suggestions/32857525-indexer-crawler-for-apache-cassandra-api-in-azu)。
>

<a name="cosmos-indexer-portal"></a>

## <a name="use-the-portal"></a>使用门户

Azure Cosmos 项的索引的最简单方法是使用中的向导[Azure 门户](https://portal.azure.com/)。 采样数据，并读取该容器中的元数据[**将数据导入**](search-import-data-portal.md) Azure 搜索中的向导可以创建一个默认索引、 源字段映射到目标索引字段，并加载在单个索引操作。 根据源数据的大小和复杂性，在数分钟内就能创建一个有效的全文搜索索引。

我们建议使用 Azure 搜索和 Azure Cosmos DB，最好是在同一区域中的同一 Azure 订阅。

### <a name="1---prepare-source-data"></a>1 - 准备源数据

您应具有 Cosmos 帐户，映射到 SQL API 或 MongoDB API 和 JSON 文档的容器的 Azure Cosmos 数据库。 

请确保你的 Cosmos DB 数据库包含数据。 [导入数据向导](search-import-data-portal.md)读取元数据并执行数据采样推断索引架构，但它还将数据加载从 Cosmos DB。 如果缺少数据，则该向导会停止出现以下错误"数据源检测索引架构时出现错误：无法生成原型索引因为数据源 emptycollection 未不返回任何数据"。

### <a name="2---start-import-data-wizard"></a>2 - 启动“导入数据”向导

你可以[启动向导](search-import-data-portal.md)从在 Azure 搜索服务页中，或通过单击命令栏**添加 Azure 搜索**中**设置**左边部分的存储帐户导航窗格。

   ![门户中的导入数据命令](./media/search-import-data-portal/import-data-cmd2.png "启动导入数据向导")

### <a name="3---set-the-data-source"></a>3 - 设置数据源

> [!NOTE] 
> 目前，无法创建或编辑**MongoDB**使用 Azure 门户或.NET SDK 的数据源。 但是，**可以**在门户中监视 MongoDB 索引器的执行历史记录。

在中**数据源**页上，源必须是**Cosmos DB**，使用以下规范：

+ **名称**是数据源对象的名称。 创建后，可以选择它的其他工作负荷。

+ **Cosmos DB 帐户**应使用 Cosmos DB 中的主密钥或辅助连接字符串`AccountEndpoint`和`AccountKey`。 帐户确定数据将转换为 SQL API 或 Mongo DB API

+ **数据库**帐户从现有的数据库。 

+ **集合**是文档的容器。 文档必须存在于为了成功导入的顺序。 

+ **查询**能为空如果所需的所有文档，否则可以输入选择一个文档子集的查询。 

   ![Cosmos DB 数据源定义](media/search-howto-index-cosmosdb/cosmosdb-datasource.png "Cosmos DB 数据源定义")

### <a name="4---skip-the-add-cognitive-search-page-in-the-wizard"></a>4 - 跳过向导中的“添加认知搜索”页

添加认知技能不是文档导入所需的。 除非需要特意在索引管道中[包含认知服务 API 和转换](cognitive-search-concept-intro.md)，否则应跳过此步骤。

若要跳过该步骤，请首先转到下一步的页。

   ![认知搜索的下一页按钮](media/search-get-started-portal/next-button-add-cog-search.png)

从该页面则可以跳过一些自定义索引。

   ![跳过认知技能步骤](media/search-get-started-portal/skip-cog-skill-step.png)

### <a name="5---set-index-attributes"></a>5 - 设置索引属性

在“索引”页中，应会看到带有数据类型的字段列表，以及一系列用于设置索引属性的复选框。 该向导可以生成基于元数据并通过抽样源数据的字段列表。 

您可以批量选择属性通过单击顶部的属性列的复选框。 选择**可检索**并**可搜索**应返回到客户端应用，可能会进行全文搜索处理每个字段。 您会注意到整数不是完整的文本或模糊搜索 （数字按原义计算和通常可在筛选器）。

查看内容的说明[属性编制索引](https://docs.microsoft.com/rest/api/searchservice/create-index#bkmk_indexAttrib)并[语言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)有关详细信息。 

花费片刻时间来检查所做的选择。 运行向导后，将创建物理数据结构，到时，除非删除再重新创建所有对象，否则无法编辑这些字段。

   ![Cosmos DB 索引定义](media/search-howto-index-cosmosdb/cosmosdb-index-schema.png "Cosmos DB 索引定义")

### <a name="6---create-indexer"></a>6 - 创建索引器

完全指定设置后，向导将在搜索服务中创建三个不同的对象。 数据源对象和索引对象作为命名的资源保存在 Azure 搜索服务中。 最后一个步骤创建索引器对象。 为索引器命名可让它作为独立的资源存在，无论在同一向导序列中创建了哪种索引和数据源对象，都可以计划和管理该索引器。

如果你不熟悉索引器，请记住，索引器是 Azure 搜索中的一个资源，它可以在外部数据源中爬网，以检索可搜索的内容。 输出**导入数据**向导是对 Cosmos DB 数据源进行爬网，提取可搜索内容，并将其导入到 Azure 搜索索引的索引器。

下面的屏幕截图显示了默认索引器配置。 您可以切换到**一次**如果你想要运行索引器一次。 单击**提交**运行该向导并创建所有对象。 随后会立即开始编制索引。

   ![Cosmos DB 索引器定义](media/search-howto-index-cosmosdb/cosmosdb-indexer.png "Cosmos DB 索引器定义")

可以在门户页监视数据导入。 进度通知指示索引状态以及已上传的文档数。 

索引编制完成后，可以使用[搜索浏览器](search-explorer.md)来查询索引。

> [!NOTE]
> 如果看不到所需的数据，您可能需要在多个字段上设置多个特性。 删除索引和索引器您刚刚创建，并逐步执行向导，修改您在步骤 5 中的索引属性的选择。 

<a name="cosmosdb-indexer-rest"></a>

## <a name="use-rest-apis"></a>使用 REST API

可以使用 REST API 对 Azure Cosmos DB 数据编制索引，按照常见的三部分组成的工作流到 Azure 搜索中的所有索引器： 创建数据源、 创建索引、 创建索引器。 当你提交创建索引器请求，从 Cosmos 存储提取数据时发生。 完成此请求后，将有一个可查询的索引。 

如果您正在评估 MongoDB，必须使用 REST API 创建数据源。

在 Cosmos DB 帐户中，可以选择是否让集合自动为所有文档编制索引。 默认情况下，为所有文档自动执行索引，但可关闭自动索引。 关闭索引功能后，只能通过本身的链接或通过使用文档 ID 进行查询的方法访问文档。 Azure 搜索要求在将由 Azure 搜索编制索引的集合中启用 Cosmos DB 自动索引。 

> [!WARNING]
> Azure Cosmos DB 是下一代 DocumentDB。 以前使用 api-version **2017年-11-11**您可以使用`documentdb`语法。 这意味着您可以指定作为数据源类型`cosmosdb`或`documentdb`。 从 API 版本开始**2019年-05-06** Azure 搜索 Api 和门户仅支持`cosmosdb`这篇文章中所述的语法。 这意味着，数据源类型必须`cosmosdb`如果想要连接到 Cosmos DB 终结点。

### <a name="1---assemble-inputs-for-the-request"></a>1-组装为请求的输入

对于每个请求，必须提供的服务名称和管理密钥 （在帖子标题中），Azure 搜索和存储帐户名称和 blob 存储的密钥。 可以使用[Postman](search-fiddler.md)将 HTTP 请求发送到 Azure 搜索。

将以下四个值复制到记事本，以便可以将其粘贴到一个请求：

+ Azure 搜索服务名称
+ Azure 搜索管理密钥
+ Cosmos DB 连接字符串

您可以在门户中找到这些值：

1. Azure 搜索在门户页面，从概述页中复制的搜索服务 URL。

2. 在左侧的导航窗格中，单击**密钥**然后将复制的主或辅助密钥 （它们是等效的）。

3. 切换到 Cosmos 存储帐户的门户页。 在左侧的导航窗格中下,**设置**，单击**密钥**。 此页提供了一个 URI，两个集的连接字符串和两个设置的键。 将一个连接字符串复制到记事本。

### <a name="2---create-a-data-source"></a>2-创建数据源

**数据源**指定要编制索引的数据、凭据和用于识别数据更改（如修改或删除了集合内的文档）的策略。 数据源定义为独立的资源，以便它可以被多个索引器使用。

若要创建数据源，公式化 POST 请求：

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCollection", "query": null },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        }
    }

请求正文包含数据源定义，其中应包括以下字段：

| 字段   | 描述 |
|---------|-------------|
| name | 必需。 选择任意名称来表示数据源对象。 |
|type| 必需。 必须是 `cosmosdb`。 |
|**凭据** | 必需。 必须是 Cosmos DB 连接字符串。<br/>对于 SQL 集合，连接字符串是按以下格式： `AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>`<br/>对于 MongoDB 集合，添加**ApiKind = MongoDb**到连接字符串：<br/>`AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=MongoDb`<br/>避免在终结点 URL 中包含端口号。 如果包含端口号，Azure 搜索将无法为 Azure Cosmos DB 数据库编制索引。|
| **容器** | 包含下列元素： <br/>**名称**：必需。 指定要编制索引的数据库集合的 ID。<br/>**查询**：可选。 可以指定一个查询来将一个任意 JSON 文档平整成 Azure 搜索可编制索引的平面架构。<br/>对于 MongoDB 集合，不支持查询。 |
| **dataChangeDetectionPolicy** | 推荐。 请参阅[为已更改的文档编制索引](#DataChangeDetectionPolicy)部分。|
|**dataDeletionDetectionPolicy** | 可选。 请参阅[为已删除的文档编制索引](#DataDeletionDetectionPolicy)部分。|

### <a name="using-queries-to-shape-indexed-data"></a>使用查询形成索引数据
可以指定一个 SQL 查询来平展嵌套的属性或数组、投影 JSON 属性并筛选要编制索引的数据。 

> [!WARNING]
> **MongoDB** 集合不支持自定义查询：`container.query` 参数必须设置为 null 或省略。 如果需要使用自定义查询，请在[用户之声](https://feedback.azure.com/forums/263029-azure-search)上告知我们。

示例文档：

    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "cosmosdb", "search"]
    }

筛选查询：

    SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark ORDER BY c._ts

平展查询：

    SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
    
    
投影查询：

    SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts


数组平展查询：

    SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark ORDER BY c._ts


### <a name="3---create-a-target-search-index"></a>3-创建目标搜索索引 

[创建目标 Azure 搜索索引](/rest/api/searchservice/create-index)如果你还没有。 下面的示例使用 ID 和说明字段创建索引：

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }

确保目标索引的架构与源 JSON 文档的架构或自定义查询投影的输出的架构兼容。

> [!NOTE]
> 对于已分区集合，默认文档键是 Azure Cosmos DB 的 `_rid` 属性，Azure 搜索会自动将其重命名为 `rid`，因为字段名称不能以下划线字符开头。 此外，Azure Cosmos DB 的 `_rid` 值包含了在 Azure 搜索键中无效的字符。 因此，`_rid` 值采用 Base64 编码。
> 
> 对于 MongoDB 集合，Azure 搜索会自动将 `_id` 属性重命名为 `doc_id`。  

### <a name="mapping-between-json-data-types-and-azure-search-data-types"></a>JSON 数据类型与 Azure 搜索数据类型之间的映射
| JSON 数据类型 | 兼容的目标索引字段类型 |
| --- | --- |
| Bool |Edm.Boolean、Edm.String |
| 类似于整数的数字 |Edm.Int32、Edm.Int64、Edm.String |
| 类似于浮点的数字 |Edm.Double、Edm.String |
| String |Edm.String |
| 基元类型的数组，如 ["a", "b", "c"] |集合 (Edm.String) |
| 类似于日期的字符串 |Edm.DateTimeOffset、Edm.String |
| GeoJSON 对象，例如 { "type":"Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| 其他 JSON 对象 |不适用 |

### <a name="4---configure-and-run-the-indexer"></a>4-配置和运行索引器

创建索引和数据源后，就可以准备创建索引器了：

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mycosmosdbindexer",
      "dataSourceName" : "mycosmosdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }

此索引器每两小时运行一次（已将计划间隔设置为“PT2H”）。 若要每隔 30 分钟运行一次索引器，可将间隔设置为“PT30M”。 支持的最短间隔为 5 分钟。 计划是可选的 - 如果省略，则索引器在创建后只运行一次。 但是，可以随时根据需要运行索引器。   

有关创建索引器 API 的更多详细信息，请参阅[创建索引器](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。

## <a name="use-net"></a>使用 .NET

.NET SDK 完全可与 REST API 搭配使用。 我们建议查看前面的 REST API 部分，以了解相关概念、工作流和要求。 然后，可以参阅以下 .NET API 参考文档，在托管代码中实现 JSON 索引器。

+ [microsoft.azure.search.models.datasource](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet)
+ [microsoft.azure.search.models.datasourcetype](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 
+ [microsoft.azure.search.models.index](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) 
+ [microsoft.azure.search.models.indexer](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)

<a name="DataChangeDetectionPolicy"></a>

## <a name="indexing-changed-documents"></a>为已更改的文档编制索引

数据更改检测策略旨在有效识别已更改的数据项。 目前，唯一支持的策略是 Azure Cosmos DB 提供的使用 `_ts`（时间戳）属性的 `High Water Mark` 策略，它按如下所示指定：

    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }

强烈建议使用此策略，以确保索引器性能良好。 

如果使用自定义查询，请确保查询投影 `_ts` 属性。

<a name="IncrementalProgress"></a>

### <a name="incremental-progress-and-custom-queries"></a>增量操作和自定义查询

索引编制过程中的增量操作可确保由于暂时性故障或执行时间限制而中断索引器执行时，索引器能够在下次运行时从中断位置运行，而不是从头开始重新为整个集合编制索引。 在为大型集合编制索引时，这一点尤其重要。 

要在使用自定义查询时启用增量操作，请确保查询按照 `_ts` 列对结果进行排序。 这会启用定期检查点，在出现故障时，Azure 搜索可以利用检查点提供增量操作。   

在某些情况下，即使查询包含 `ORDER BY [collection alias]._ts` 子句，Azure 搜索也可能不会推断出查询是按照 `_ts` 进行排序的。 可以告知 Azure 搜索，结果是通过使用 `assumeOrderByHighWaterMarkColumn` 配置属性进行排序的。 要指定此提示，请按如下所示创建或更新索引器： 

    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "assumeOrderByHighWaterMarkColumn" : true } }
    } 

<a name="DataDeletionDetectionPolicy"></a>

## <a name="indexing-deleted-documents"></a>为已删除的文档编制索引

从集合中删除行时，通常还需要从搜索索引中删除这些行。 数据删除检测策略旨在有效识别已删除的数据项。 目前，唯一支持的策略是 `Soft Delete` 策略（删除标有某种类型的标志），它按如下所示指定：

    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }

如果使用自定义查询，请确保查询投影由 `softDeleteColumnName` 引用的属性。

下面的示例创建具有软删除策略的数据源：

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCosmosDbCollectionId" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }

## <a name="watch-this-video"></a>观看此视频

在这个略显陈旧 7 分钟的视频，Azure Cosmos DB 计划经理 Andrew Liu 演示了如何将 Azure 搜索索引添加到 Azure Cosmos DB 容器。 视频中所示的门户页已过时，但信息仍适用。

>[!VIDEO https://www.youtube.com/embed/OyoYu1Wzk4w]

## <a name="NextSteps"></a>后续步骤

祝贺你！ 已学习了如何使用索引器将 Azure Cosmos DB 与 Azure 搜索集成。

* 若要详细了解 Azure Cosmos DB，请参阅 [Azure Cosmos DB 服务页](https://azure.microsoft.com/services/cosmos-db/)。
* 若要详细了解 Azure 搜索，请参阅[搜索服务页](https://azure.microsoft.com/services/search/)。
