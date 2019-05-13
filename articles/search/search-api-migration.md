---
title: 升级到最新的 Azure 搜索服务 REST API 版本 - Azure 搜索
description: 查看 API 版本的差异，了解将现有代码迁移到最新 Azure 搜索服务 REST API 版本所需的操作。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.openlocfilehash: 286d8bbc01b5916e842c196aed5a49ef1c76bc3c
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025201"
---
# <a name="upgrade-to-the-latest-azure-search-service-rest-api-version"></a>升级到最新的 Azure 搜索服务 REST API 版本
如果使用旧版[Azure 搜索服务 REST API](https://docs.microsoft.com/rest/api/searchservice/)，本文有助于升级应用程序以使用最新公开发布的 API 版本 2019年-05-06。

版本 2019年-05-06 的 REST API 包含某些早期版本的更改。 这些更改主要涉及向后兼容性，因此更改代码只需最小的工作量，具体取决于之前使用的是哪个版本。 [升级步骤](#UpgradeSteps)概述了使用新功能所需的代码更改。

> [!NOTE]
> Azure 搜索服务实例支持范围的 REST API 版本，包括之前的。 你可以继续使用这些 API 版本，但我们建议将代码迁移到最新版本，以便可以访问新功能。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-2019-05-06"></a>什么是版本 2019年-05-06 中的新增功能
版本 2019年-05-06 是最新正式发布的 Azure 搜索服务 REST API 版本。 已转换到此 API 版本中已公开发布状态的功能包括：

* [记忆式键入功能](index-add-suggesters.md)是完成部分指定的术语输入 typeahead 功能。

* [复杂类型](search-howto-complex-data-types.md)的 Azure 搜索索引中的结构化的对象数据提供本机支持。

* [解析模式 JsonLines](search-howto-index-json-blobs.md)、 部分的 Azure Blob 编制索引，创建一个搜索文档，每个由换行符分隔的 JSON 实体。

* [认知搜索](cognitive-search-concept-intro.md)提供索引，可利用 AI 扩充的引擎由各种认知服务。

与此正式更新同时发生多个预览功能发布。 若要查看的新预览功能列表，请参阅[搜索 REST api 版本 2019年-05-06-预览](search-api-preview.md)。

## <a name="breaking-changes"></a>重大变化

包含以下功能的现有代码将中断 api-version = 2019年-05-06。

### <a name="indexer-for-azure-cosmos-db---datasource-is-now-type-cosmosdb"></a>适用于 Azure Cosmos DB 的索引器数据源是现在"type":"cosmosdb"

如果使用的[Cosmos DB 索引器](search-howto-index-cosmosdb.md )，则必须更改`"type": "documentdb"`到`"type": "cosmosdb"`。

### <a name="indexer-execution-result-errors-no-longer-have-status"></a>索引器执行结果错误不再有状态

索引器执行的错误结构以前`status`元素。 此元素已删除，因为它不提供有用的信息。

### <a name="indexer-data-source-api-no-longer-returns-connection-strings"></a>索引器数据源 API 不会再返回连接字符串

从 API 版本数据的源 API 2019-05-06 和 2019年-05-06-预览及更高版本，无法再连接字符串在响应中返回的任何 REST 操作。 在以前的 API 版本，使用 POST，创建数据源返回 Azure 搜索**201**跟 OData 响应中，包含纯文本中的连接字符串。

### <a name="named-entity-recognition-cognitive-skill-is-now-discontinued"></a>名为认知技能现已终止的实体识别

如果您调用[名称实体识别](cognitive-search-skill-named-entity-recognition.md)技能在代码中的，调用将失败。 替代功能是[实体识别](cognitive-search-skill-entity-recognition.md)。 您应能够技能引用替换为任何其他更改。 API 签名是相同的两个版本。 

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升级步骤
如果要从以前的 GA 版本进行升级，2017年-11-11 或 2016年-09-01，您可能无需进行任何更改到代码中，而不更改版本号。 仅对于以下情况，可能需要更改代码：

* 当 API 响应中返回无法识别的属性时，代码失效。 默认情况下，应用程序应忽略无法理解的属性。

* 代码仍坚持 API 请求，并尝试将其重新发送到新 API 版本。 例如，如果应用程序仍存留从搜索 API 返回的延续标记（有关详细信息，请查找[搜索 API 参考](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)中的 `@search.nextPageParameters`）。

如果任一种情况适用，都可能需要更改相应代码。 否则，无需更改应该是必要除非你想要开始使用[的新功能](#WhatsNew)的版本 2019年-05-06。

如果要从预览 API 版本进行升级，上述内容同样适用，但还必须注意某些预览功能将不可用版本 2019年-05-06 中：

* ["更喜欢这项"查询](search-more-like-this.md)仍是仅限预览版的功能。

如果你的代码使用这些功能，您将不能升级到的 API 版本 2019年-05-06，而不删除它们的使用。

> [!IMPORTANT]
> 预览 API 用于测试和评估，不应在生产环境中使用。
> 

### <a name="upgrading-complex-types"></a>升级的复杂类型

如果你的代码使用较早的预览版 API 版本 2017年-11-11-Preview 或 2016年-09-01-预览的复杂类型，有一些新功能和更改限制版本中需要注意其中的 2019年-05-06:

+ 在子字段和每个索引的复杂集合数的深度限制已降低。 如果创建索引超过这些限制使用预览 api 版本，任何尝试更新或重新创建它们使用 API 版本 2019年-05-06 将失败。 如果这适用于你，你将需要重新设计架构，以适应新的限制，然后重新生成索引。

+ 2019-05-06 上每个文档的复杂集合的元素数量的 api 版本中没有新限制。 如果超过这些限制使用预览 api 版本的文档创建索引，重新建立索引使用 api 版本 2019年-05-06 该数据的任何尝试将失败。 如果这适用于你，你需要重建索引数据之前减少每个文档的复杂集合元素的数量。

有关详细信息，请参阅[的 Azure 搜索服务限制](search-limits-quotas-capacity.md)。

### <a name="how-to-upgrade-an-old-complex-type-structure"></a>如何升级旧的复杂类型结构

如果你的代码使用较早的预览版 API 版本之一使用复杂类型，则可能使用的索引定义格式，如下所示：

```json
{
  "name": "hotels",  
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.microsoft" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.microsoft" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true, "analyzer": "tagsAnalyzer" },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true },
    { "name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address", "type": "Edm.ComplexType" },
    { "name": "Address/StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
    { "name": "Address/City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/PostalCode", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/Country", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)" }, 
    { "name": "Rooms/Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene" },
    { "name": "Rooms/Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene" },
    { "name": "Rooms/Type", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
    { "name": "Rooms/BedOptions", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
    { "name": "Rooms/SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
    { "name": "Rooms/Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "facetable": true, "analyzer": "tagsAnalyzer" }
  ]
}  
```

在 API 版本 2017年-11-11-Preview 中引入了用于定义索引字段的较新类似于树中的格式。 在新的格式，每个复杂字段有一个字段集合定义了其子字段。 在 API 版本 2019年-05-06，以独占方式使用此新格式，并尝试创建或更新使用旧格式的索引将失败。 如果必须创建使用旧格式的索引，您将需要使用 API 版本 2017年-11-11-预览将其更新到新的格式之前可以使用 API 版本 2019年-05-06 管理它们。

可以使用以下步骤使用 API 版本 2017年-11-11-预览为新的格式更新"平面"索引：

1. 执行 GET 请求以检索你的索引。 如果它已在新的格式，即已完成。

2. 转换到新的格式从"平面"格式的索引。 您必须为此编写代码，因为没有任何示例代码可在本文撰写之时。

3. 执行 PUT 请求来更新为新格式的索引。 请确保未更改的字段，可搜索性/filterability 如索引的任何其他详细信息，因为这不允许更新索引 api。

> [!NOTE]
> 不能使用旧的"平面"格式从 Azure 门户创建的索引进行管理。 请尽早与"树"升级"平面"表示形式的索引。

## <a name="next-steps"></a>后续步骤

查看 Azure 搜索服务 REST API 参考文档。 如果遇到问题，我们寻求帮助上[堆栈溢出](https://stackoverflow.com/)或[请联系支持人员](https://azure.microsoft.com/support/community/?product=search)。

> [!div class="nextstepaction"]
> [搜索服务 REST API 参考](https://docs.microsoft.com/rest/api/searchservice/)

