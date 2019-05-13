---
title: 数据导入以便将数据引入到搜索索引中 - Azure 搜索
description: 从外部数据源填充数据并将数据上传到 Azure 搜索中的索引。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 83ca0c11ab0065929d939b7345cbd15869740bb3
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024344"
---
# <a name="data-import-overview---azure-search"></a>数据导入概述-Azure 搜索

在 Azure 搜索中，将会针对已加载和已保存到[搜索索引](search-what-is-an-index.md)中的内容执行查询。 本文介绍填充索引的两种基本方法：一种是推送，即以编程方式将数据推送至索引；另一种是拉取，即将 [Azure 搜索索引器](search-indexer-overview.md)指向支持的数据源，以便拉取数据。

这两种方法，目的是向*将数据加载*从外部数据源到 Azure 搜索索引。 Azure 搜索将使你创建一个空的索引，但直到推送或拉取到其中的数据，不可查询。

## <a name="pushing-data-to-an-index"></a>将数据推送至索引
推送模式用于以编程方式将数据发送到 Azure 搜索，是最灵活的方法。 首先，它对数据源类型没有限制。 任何由 JSON 文档组成的数据集都可以推送至 Azure 搜索索引，前提是数据集中的每个文档的字段都映射到索引架构中定义的字段。 其次，它对执行频率没有限制。 可以根据需要选择相应的频率，将更改推送到索引。 对于具有极低延迟要求的应用程序（例如，如果需要搜索操作与动态库存数据库同步），只能选择推送模型。

此方法相比拉模型更加灵活，因为可以单个或批量上传文档（每批最多 1000 个或 16MB，以先达到为准）。 推送模型还允许将文档上传到 Azure 搜索，而不考虑数据的位置。

### <a name="how-to-push-data-to-an-azure-search-index"></a>如何将数据推送至 Azure 搜索索引

可以使用以下 API，将单个或多个文档加载到一个索引中：

+ [Add, Update, or Delete Documents (REST API)](https://docs.microsoft.com/rest/api/searchservice/AddUpdate-or-Delete-Documents)（添加、更新或删除文档 (REST API)）
+ [indexAction 类](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexaction?view=azure-dotnet)或 [indexBatch 类](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexbatch?view=azure-dotnet) 

目前尚没有支持通过门户推送数据的工具。

每个方法的介绍，请参阅[快速入门：创建使用 PowerShell 和 REST API 的 Azure 搜索索引](search-create-index-rest-api.md)或[快速入门：创建 Azure 搜索索引中的C# ](search-import-data-dotnet.md)。

<a name="indexing-actions"></a>

### <a name="indexing-actions-upload-merge-mergeorupload-delete"></a>索引操作： 上传、 合并、 mergeOrUpload，删除

可以控制在每个文档模式中，索引操作的类型指定文档是否应上传完整、 合并与现有的文档内容，或已删除。

在 REST API 发出 HTTP POST 请求具有 JSON 请求正文向 Azure 搜索索引的终结点 URL。 "Value"数组中的每个 JSON 对象包含文档密钥，并指定的索引操作添加的更新，或删除文档内容。 有关代码示例，请参阅[加载文档](search-create-index-rest-api.md#load-documents)。

在.NET SDK 中，将数据打包到`IndexBatch`对象。 `IndexBatch`封装的集合`IndexAction`对象，其中每个包含一个文档和一个属性，它指示要在该文档上执行的操作的 Azure 搜索。 有关代码示例，请参阅[构造 IndexBatch](search-import-data-dotnet.md#construct-indexbatch)。


| @search.action | 描述 | 每个文档必需的字段 | 说明 |
| -------------- | ----------- | ---------------------------------- | ----- |
| `upload` |`upload` 操作类似于“upsert”，如果文档是新文档，则插入；如果文档已经存在，则进行更新/替换。 |关键字段以及要定义的任何其他字段 |更新/替换现有文档时，会将请求中未指定的任何字段设置为 `null`。 即使该字段之前设置为了非 null 值也是如此。 |
| `merge` |使用指定的字段更新现有文档。 如果索引中不存在该文档，merge 会失败。 |关键字段以及要定义的任何其他字段 |merge 中指定的任何字段都将替换文档中的现有字段。 在.NET SDK 中，这包括类型的字段`DataType.Collection(DataType.String)`。 在 REST API 中，这包括类型的字段`Collection(Edm.String)`。 例如，如果文档包含值为 `["budget"]` 的字段 `tags`，并且已使用值 `["economy", "pool"]` 对 `tags` 执行合并，则 `tags` 字段的最终值将为 `["economy", "pool"]`。 而不会是 `["budget", "economy", "pool"]`。 |
| `mergeOrUpload` |如果索引中已存在具有给定关键字段的文档，则此操作的行为类似于 `merge`。 如果该文档不存在，则它的行为类似于对新文档进行 `upload` 。 |关键字段以及要定义的任何其他字段 |- |
| `delete` |从索引中删除指定文档。 |仅关键字段 |所指定关键字段以外的所有字段都会被忽略。 如果要从文档中删除单个字段，请改用 `merge`，只需将该字段显式设置为 null。 |

## <a name="decide-which-indexing-action-to-use"></a>确定要使用的索引操作
若要导入使用.NET SDK、 （上传、 合并、 删除和 mergeOrUpload） 的数据。 根据选择的以下操作，每个文档必须仅包含某些特定的字段：


### <a name="formulate-your-query"></a>表述查询
有两种方法可以 [使用 REST API 搜索索引](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。 一种方法是发出 HTTP POST 请求，这种请求的查询参数在请求主题的 JSON 对象中定义。 另一种方法是发出 HTTP GET 请求，这种请求的查询参数在请求 URL 中定义。 POST 的查询参数大小限制比 GET [宽松](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。 因此建议使用 POST，使用 GET 更方便的特殊情况除外。

POST 和 GET 都需要在请求 URL 中提供*服务名称*、*索引名称*和正确的 *API 版本*（发布本文档时的 API 版本为 `2019-05-06`）。 GET 的 URL 末尾为*查询字符串*，用于提供查询参数。 有关 URL 格式，请参见以下内容：

    https://[service name].search.windows.net/indexes/[index name]/docs?[query string]&api-version=2019-05-06

POST 的 URL 格式相同，只是查询字符串参数仅包含 API 版本。


## <a name="pulling-data-into-an-index"></a>将数据拉取到索引中
提取模型对支持的数据源进行爬网，将数据自动上传到索引中。 在 Azure 搜索中，此功能是通过索引器实现的，目前适用于以下平台：

+ [Blob 存储](search-howto-indexing-azure-blob-storage.md)
+ [表存储](search-howto-indexing-azure-tables.md)
+ [Azure Cosmos DB](https://aka.ms/documentdb-search-indexer)
+ [Azure VM 上的 Azure SQL 数据库和 SQL Server](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

索引器将索引连接到数据源（通常是表、视图或等效的结构），将源字段映射到索引中的等效字段。 在执行期间，行集会自动转换为 JSON 并载入指定的索引中。 所有索引器支持计划，使用户能够指定数据的刷新频率。 大多数索引器提供更改跟踪（如果受数据源的支持）。 除了识别新文档外，通过跟踪对现有文档的更改和删除外，索引器免除了主动管理索引中数据的必要。 


### <a name="how-to-pull-data-into-an-azure-search-index"></a>如何将数据拉取至 Azure 搜索索引

索引器功能已在 [Azure 门户](search-import-data-portal.md)、[REST API](/rest/api/searchservice/Indexer-operations) 和 [.NET SDK](/dotnet/api/microsoft.azure.search.indexersoperationsextensions) 中公开。 

使用门户的一个优势在于，Azure 搜索通常可以通过读取源数据集的元数据来生成默认的索引架构。 在处理生成的索引之前可对其进行修改，此后，只能编辑不需要重建索引的架构。 如果想要进行的更改会直接影响架构，则需要重建索引。 

## <a name="verify-data-import-with-search-explorer"></a>使用搜索浏览器验证数据导入

针对文档上传执行初步检查的捷径之一是在门户中使用**搜索浏览器**。 使用资源管理器可以直接查询索引，而无需编写任何代码。 搜索体验取决于默认设置，例如[简单语法](/rest/api/searchservice/simple-query-syntax-in-azure-search)和默认的 [searchMode 查询参数](/rest/api/searchservice/search-documents)。 结果以 JSON 格式返回，方便用户检查整个文档。

> [!TIP]
> 有大量的 [Azure 搜索代码示例](https://github.com/Azure-Samples/?utf8=%E2%9C%93&query=search)包含了嵌入的或随时可用的数据集，帮助用户轻松入门。 门户中还提供了一个示例索引器，以及一个由小型房地产数据集组成的数据源（名为“realestate-us-sample”）。 针对示例数据源运行预配置的索引器时，会创建索引并连同文档一起加载该索引，然后，可以使用搜索浏览器或编写的代码查询该索引。

## <a name="see-also"></a>另请参阅

+ [索引器概述](search-indexer-overview.md)
+ [门户演练：创建、加载和查询索引](search-get-started-portal.md)
