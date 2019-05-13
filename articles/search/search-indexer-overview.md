---
title: 用于在编制索引期间抓取数据源的索引器 - Azure 搜索
description: 对 Azure SQL 数据库、Azure Cosmos DB 或 Azure 存储爬网，提取可搜索的数据并填充 Azure 搜索索引。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 87e35573eea836fc8a88c7515409c070ec63aa3b
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024896"
---
# <a name="indexers-in-azure-search"></a>Azure 搜索中的索引器

Azure 搜索中的*索引器*是一种爬网程序，它从外部 Azure 数据源提取可搜索的数据和元数据，并根据索引与数据源之间字段到字段映射填充索引。 这种方法有时称为拉取模式因为服务而无需编写任何代码，将数据添加到索引提取数据。

索引器为基础数据源类型或平台，Azure，Cosmos DB、 Azure 表存储和 Blob 存储上的 SQL Server 的单个索引器。 Blob 存储索引器具有其他属性特定于 blob 的内容类型。

可以单独使用索引器来引入数据，也可以结合索引器使用多种技术来加载索引中的部分字段。

可以按需运行索引器，也可以采用每 15 分钟运行一次的定期数据刷新计划来运行索引器。 要进行更频繁的更新，则需要采用“推送模式”，便于同时更新 Azure 搜索和外部数据源中的数据。

## <a name="approaches-for-creating-and-managing-indexers"></a>创建及管理索引器的方法

可以使用以下方法创建和管理索引器：

* [门户 > 导入数据向导](search-import-data-portal.md)
* [服务 REST API](https://docs.microsoft.com/rest/api/searchservice/Indexer-operations)
* [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.iindexersoperations)

一开始会将新的索引器宣布为预览版功能。 预览版功能首先在 API（REST 和 .NET）中引入，然在逐渐公开发行以后再集成到门户中。 如果评估的是新索引器，则应做好编写代码的计划。

## <a name="permissions"></a>权限

与索引器，其中包括 GET 请求的状态或定义，相关的所有操作都需要[管理员 api 密钥](search-security-api-keys.md)。 

<a name="supported-data-sources"></a>

## <a name="supported-data-sources"></a>支持的数据源

索引器在 Azure 上抓取数据存储。

* [Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Azure Cosmos DB](search-howto-index-cosmosdb.md)
* [Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)
* [Azure 表存储](search-howto-indexing-azure-tables.md) 

> [!Note]
> 不支持 azure 表存储[认知搜索](cognitive-search-concept-intro.md)。
>

## <a name="basic-configuration-steps"></a>基本配置步骤
索引器可提供数据源独有的功能。 因此，索引器或数据源配置的某些方面会因索引器类型而不同。 但是，所有索引器的基本构成元素和要求都相同。 下面介绍所有索引器都适用的共同步骤。

### <a name="step-1-create-a-data-source"></a>步骤 1：创建数据源
索引器获取从数据源连接*数据源*对象。 数据源定义提供的连接字符串以及可能的凭据。 调用[创建数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source) REST API 或 [DataSource 类](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasource)以创建资源。

数据源的配置和管理独立于使用数据源的索引器，这意味着多个索引器可使用一个数据源，同时加载多个索引。

### <a name="step-2-create-an-index"></a>步骤 2：创建索引
索引器会自动执行某些与数据引入相关的任务，但通常不会自动创建索引。 先决条件是必须具有预定义的索引，且索引的字段必须与外部数据源中的字段匹配。 字段需要匹配的名称和数据类型。 有关构建索引的详细信息，请参阅 [创建索引（Azure 搜索 REST API）](https://docs.microsoft.com/rest/api/searchservice/Create-Index)或[索引类](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index)。 如需字段关联方面的帮助，请参阅 [Azure 搜索索引器中的字段映射](search-indexer-field-mappings.md)。

> [!Tip]
> 虽然不能使用索引器来生成索引，但可以使用门户中的**导入数据**向导。 大多数情况下，该向导可以根据源中现有的元数据推断索引架构，提供一个初级索引架构，该架构在向导处于活动状态时可以进行内联编辑。 在服务上创建索引以后，若要在门户中进一步进行编辑，多数情况下只能添加新字段。 可以将向导视为索引的创建工具而非修订工具。 如需手动方式的学习，请一步步完成[门户演练](search-get-started-portal.md)。

### <a name="step-3-create-and-schedule-the-indexer"></a>步骤 3：创建和计划索引器
索引器定义是一个构造，它将结合在一起的所有元素与数据引入相关。 所需的元素包括一个数据源和索引。 可选元素包括计划和字段映射。 字段映射只是如果源字段和索引字段清楚地对应可选的。 索引器可从另一个服务引用数据源，前提是该数据源来自同一个订阅。 有关构建索引器的详细信息，请参阅 [创建索引器（Azure 搜索 REST API）](https://docs.microsoft.com/rest/api/searchservice/Create-Indexer)。

<a id="RunIndexer"></a>

## <a name="run-indexers-on-demand"></a>索引器按需运行

通常会计划编制索引时，在索引器，也可以根据需要使用调用[运行命令](https://docs.microsoft.com/rest/api/searchservice/run-indexer):

    POST https://[service name].search.windows.net/indexers/[indexer name]/run?api-version=2019-05-06
    api-key: [Search service admin key]

> [!NOTE]
> “运行 API”成功返回时，已计划索引器调用，但实际处理过程以异步方式发生。 

你可以监视门户中或通过获取索引器状态 API 的索引器状态。 

<a name="GetIndexerStatus"></a>

## <a name="get-indexer-status"></a>获取索引器状态

您可以检索通过索引器的状态和执行历史记录[获取索引器状态命令](https://docs.microsoft.com/rest/api/searchservice/get-indexer-status):


    GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2019-05-06
    api-key: [Search service admin key]

响应包含总体索引器状态、最后一次（或正在进行的）索引器调用以及最近索引器调用的历史记录。

    {
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2018-11-26T03:37:18.853Z",
            "endTime":"2018-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
         },
        "executionHistory":[ {
            "status":"success",
             "errorMessage":null,
            "startTime":"2018-11-26T03:37:18.853Z",
            "endTime":"2018-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        }]
    }

执行历史记录包含最多 50 个最近完成的执行，它们被按反向时间顺序排序（因此，最新执行出现在响应中的第一个）。

## <a name="next-steps"></a>后续步骤
了解基本概念后，下一步是查看每种数据源特定的要求和任务。

* [Azure SQL 数据库或 Azure 虚拟机上的 SQL Server](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Azure Cosmos DB](search-howto-index-cosmosdb.md)
* [Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)
* [Azure 表存储](search-howto-indexing-azure-tables.md)
* [使用 Azure 搜索 Blob 索引器为 CSV blob 编制索引](search-howto-index-csv-blobs.md)
* [使用 Azure 搜索 Blob 索引器为 JSON blob 编制索引](search-howto-index-json-blobs.md)
