---
title: 查询类型和组成部分 - Azure 搜索
description: 在 Azure 搜索中生成搜索查询并使用参数对结果进行筛选、选择和排序的基础知识。
author: HeidiSteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: seodec2018
ms.openlocfilehash: 7ed675e4c6988cf4c1340613323440de55a36843
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024464"
---
# <a name="how-to-compose-a-query-in-azure-search"></a>如何在 Azure 搜索中撰写查询

在 Azure 搜索中，查询是往返操作的完整规范。 请求中的参数提供匹配条件用于查找索引中的文档、引擎的执行指令，以及用于调整响应的指令。 

查询请求是一个丰富的构造，它指定哪些字段在搜索范围之内、如何搜索、要返回哪些字段、是否要排序或筛选，等等。 如果未指定这些内容，查询会作为全文搜索操作针对所有可搜索字段运行，返回一个任意排序的未评分结果集。

## <a name="apis-and-tools-for-testing"></a>用于测试的 API 和工具

下表列出用于提交查询的 API 和基于工具的方法。

| 方法 | 描述 |
|-------------|-------------|
| [搜索浏览器（门户）](search-explorer.md) | 提供搜索栏，以及索引和 API 版本选项。 结果会以 JSON 文档的形式返回。 <br/>[了解详细信息。](search-get-started-portal.md#query-index) | 
| [Postman 或 Fiddler](search-fiddler.md) | Web 测试工具是用公式表示 REST 调用的极佳选择。 REST API 支持 Azure 搜索中的每个可能操作。 在本文中，了解如何设置 HTTP 请求标头和正文，以便向 Azure 搜索发送请求。  |
| [SearchIndexClient (.NET)](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient?view=azure-dotnet) | 可用于查询 Azure 搜索索引的客户端。  <br/>[了解详细信息。](search-howto-dotnet-sdk.md#core-scenarios)  |
| [搜索文档 (REST API)](https://docs.microsoft.com/rest/api/searchservice/search-documents) | 索引上的 GET 或 POST 方法，使用查询参数进行其他输入。  |

## <a name="a-first-look-at-query-requests"></a>查询请求的初步认识

示例对于介绍新的概念非常有帮助。 作为在 [REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 中构造的一个代表性查询，本示例针对[房地产演示索引](search-get-started-portal.md)，并包括常用的参数。

```
{
    "queryType": "simple" 
    "search": "seattle townhouse* +\"lake\"",
    "searchFields": "description, city",
    "count": "true",
    "select": "listingId, street, status, daysOnMarket, description",
    "top": "10",
    "orderby": "daysOnMarket"
}
```

+ **`queryType`** 设置分析器，在 Azure 搜索中，该分析器可以是[默认的简单查询分析器](search-query-simple-examples.md)（最适合用于全文搜索），也可以是[完整的 Lucene 查询分析器](search-query-lucene-examples.md)（用于正则表达式、邻近搜索、模糊和通配符搜索等高级查询构造）。

+ **`search`** 提供匹配条件（通常是文本，但往往附带布尔运算符）。 包含单个独立字词的查询称为字词查询。 由括在引号中的多个部分组成的查询称为关键短语查询。 搜索可以是未定义的（例如 **`search=*`**），但搜索更有可能包含字词、短语和运算符，如以下示例中所示。

+ **`searchFields`** 是可选的，用于将查询执行限制为特定的字段。

还可以通过查询中包含的参数来调整响应。 在本示例中，结果集包含 **`select`** 语句中列出的字段。 此查询只返回前 10 个匹配项，但 **`count`** 会告知总共有多少个匹配的文档。 在此查询中，行已按 daysOnMarket 排序。

在 Azure 搜索中，查询执行始终针对一个使用请求中提供的 API 密钥进行身份验证的索引。 在 REST 中，两者均在请求标头中提供。

### <a name="how-to-run-this-query"></a>如何运行此查询

若要执行此查询，请使用[搜索浏览器和房地产演示索引](search-get-started-portal.md)。 

可将此查询字符串粘贴到浏览器的搜索栏中：`search=seattle townhouse +lake&searchFields=description, city&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket`

## <a name="how-query-operations-are-enabled-by-the-index"></a>索引如何启用查询操作

索引设计和查询设计在 Azure 搜索中紧密耦合。 需要提前知道的一个重要事实是，包含每个字段中属性的索引架构确定了可以生成的查询类型。 

字段中的索引属性设置允许的操作 - 字段在索引中是否可搜索、在结果中是否可检索、是否可排序、是否可筛选，等等。 在示例查询字符串中，只有 `"$orderby": "daysOnMarket"` 可以正常工作，因为 daysOnMarket 字段在索引架构中标记为可排序。 

![房地产示例的索引定义](./media/search-query-overview/realestate-sample-index-definition.png "房地产示例的索引定义")

上面的屏幕截图是房地产示例的索引属性的部分列表。 可在门户中查看整个索引架构。 有关索引属性的详细信息，请参阅[创建索引 REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)。

> [!Note]
> 某些查询功能在索引范围启用，而不是按字段启用。 这些功能包括：[同义词映射](search-synonyms.md)，[自定义分析器](index-add-custom-analyzers.md)，[建议器 （适用于记忆式键入功能和建议的查询） 构造](index-add-suggesters.md)，[评分逻辑排名结果](index-add-scoring-profiles.md)。

## <a name="elements-of-a-query-request"></a>查询请求的元素

查询始终指向单个索引。 不能联接索引或者创建自定义或临时数据结构作为查询目标。 

查询请求上的必需元素包括以下内容：

+ 以 URL 表示的、包含固定和用户定义组件的服务终结点与索引文档集合：**`https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`**
+ 之所以需要 **`api-version`**（仅适用于 REST），是因为始终有多个可用的 API 版本。 
+ **`api-key`**：查询或管理 API 密钥，用于对服务请求进行身份验证。
+ **`queryType`**：简单或完整类型，如果想使用内置的默认简单语法，则可以省略此元素。
+ **`search`** 或 **`filter`** 提供匹配条件，如果想要执行空搜索，则可以不指定此元素。 这两种查询类型都是作为简单分析器讨论的，但即使是高级查询，也需要通过搜索参数来传递复杂的查询表达式。

所有其他搜索参数都为可选参数。 有关属性的完整列表，请参阅[创建索引 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-index)。 有关如何在处理期间使用参数的详细信息，请参阅 [Azure 搜索中全文搜索的工作原理](search-lucene-query-architecture.md)。

## <a name="choose-a-parser-simple--full"></a>选择一个分析器：简单 | 完整

Azure 搜索基于 Apache Lucene ，提供两种查询分析器选择，分别用于处理典型查询和专用查询。 使用简单分析器的请求是通过[简单查询语法](query-simple-syntax.md)构建的。由于在自由格式文本查询中具有速度和效率优势，这种语法已选作默认语法。 此语法支持多种常用的搜索运算符，包括 AND、OR、NOT、短语、后缀和优先运算符。

在将 `queryType=full` 添加到请求时所启用的[完整 Lucene 查询语法](query-Lucene-syntax.md#bkmk_syntax)公开作为 [Apache Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) 的一部分开发的、已被广泛采用的且富有表达能力的查询语言。 完整语法扩展了简单语法。 为简单语法编写的任何查询在完整 Lucene 分析器下运行。 

以下示例演示了一个要点：采用不同 queryType 设置的同一个查询会产生不同的结果。 在第一个查询中，`^3` 被视为搜索字词的一部分。

```
queryType=simple&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

使用完整 Lucene 分析器的同一查询针对“ranch”解释字段内提升，从而大大提升了包含该特定字词的结果的搜索排名。

```
queryType=full&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

<a name="types-of-queries"></a>

## <a name="types-of-queries"></a>查询类型

Azure 搜索支持广泛的查询类型。 

| 查询类型 | 使用情况 | 示例和详细信息 |
|------------|--------|-------------------------------|
| 自由格式文本搜索 | 搜索参数和任一分析器| 全文搜索在索引中所有可搜索字段内扫描一个或多个字词，其工作原理与 Google 或必应等搜索引擎相同。 简介中的示例属于全文搜索。<br/><br/>全文搜索默认使用标准 Lucene 分析器来执行文本分析，以将字词设为小写，并删除“the”等干扰词。 可将默认设置替代为可以修改文本分析的[非英语分析器](index-add-language-analyzers.md#language-analyzer-list)或[专用的与语言无关的分析器](index-add-custom-analyzers.md#AnalyzerTable)。 例如，将整个字段内容视为单个标记的[关键字](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)。 此分析器可用于邮政编码、ID 和某些产品名称等数据。 | 
| 筛选的搜索 | [OData 筛选表达式](query-odata-filter-orderby-syntax.md)和任一分析器 | 筛选器查询对索引中的所有可筛选字段计算布尔表达式。 与搜索不同，筛选器查询与字段内容完全匹配，包括字符串字段的大小写区分。 另一项差别在于，筛选器查询以 OData 语法表示。 <br/>[筛选表达式示例](search-query-simple-examples.md#example-3-filter-queries) |
| 地理搜索 | 字段中的 [Edm.GeographyPoint 类型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)、筛选表达式和任一分析器 | 存储在字段中的具有 Edm.GeographyPoint 的坐标用于“附近查找”或基于地图的搜索控件。 <br/>[地理搜索示例](search-query-simple-examples.md#example-5-geo-search)|
| 范围搜索 | 筛选表达式和简单分析器 | 在 Azure 搜索中，范围查询是使用筛选器参数生成的。 <br/>[范围筛选器示例](search-query-simple-examples.md#example-4-range-filters) | 
| [字段内筛选](query-lucene-syntax.md#bkmk_fields) | 搜索参数和完整分析器 | 针对单个字段生成复合查询表达式。 <br/>[字段内筛选示例](search-query-lucene-examples.md#example-2-intra-field-filtering) |
| [模糊搜索](query-lucene-syntax.md#bkmk_fuzzy) | 搜索参数和完整分析器 | 匹配具有类似构造或拼写方式的字词。 <br/>[模糊搜索示例](search-query-lucene-examples.md#example-3-fuzzy-search) |
| [邻近搜索](query-lucene-syntax.md#bkmk_proximity) | 搜索参数和完整分析器 | 查找在文档中相互靠近的字词。 <br/>[邻近搜索示例](search-query-lucene-examples.md#example-4-proximity-search) |
| [术语提升](query-lucene-syntax.md#bkmk_termboost) | 搜索参数和完整分析器 | 如果某个文档包含提升的字词（相对于其他未提升的字词），则提高其排名。 <br/>[字词提升示例](search-query-lucene-examples.md#example-5-term-boosting) |
| [正则表达式搜索](query-lucene-syntax.md#bkmk_regex) | 搜索参数和完整分析器 | 基于正则表达式的内容进行匹配。 <br/>[正则表达式示例](search-query-lucene-examples.md#example-6-regex) |
|  [通配符或前缀搜索](query-lucene-syntax.md#bkmk_wildcard) | 搜索参数和完整分析器 | 基于前缀和波浪符 (`~`) 或单个字符 (`?`) 进行匹配。 <br/>[通配符搜索示例](search-query-lucene-examples.md#example-7-wildcard-search) |

## <a name="manage-search-results"></a>管理搜索结果 

查询结果会流式处理为 REST API 中的 JSON 文档，但如果使用 .NET API，则会内置序列化功能。 可通过在查询中设置参数并为响应选择特定字段来调整结果。

可通过以下方式使用查询上的参数来调整结果集的结构：

+ 对结果中的文档数量（默认为 50 个）进行限制或分批
+ 选择结果中要包含的字段
+ 设置排列顺序
+ 添加突出显示效果，以便在搜索结果正文中清楚看到匹配的搜索词

### <a name="tips-for-unexpected-results"></a>意外结果提示

有时可能会出现预料外的结果内容（而不是结构）。 如果查询结果并不是预期内容，可以尝试对查询进行以下修改，然后查看结果是否改进：

+ 将 **`searchMode=any`**（默认）更改为 **`searchMode=all`** 可获取符合所有条件而不是某个条件的匹配项。 在查询包含布尔运算符时更应如此。

+ 如果需要进行文本或词法分析但查询类型排除了语言处理环节，请更改查询方法。 在全文搜索、 文本或词法分析 autocorrects 拼写错误、 单一的复数形式 word 窗体和甚至不规则谓词或名词。 对于模糊搜索和通配符搜索等查询，其查询分析管道中不包含文本分析。 在某些情况下会采用正则表达式作为解决方法。 

### <a name="paging-results"></a>分页结果
Azure 搜索可轻松对搜索结果进行分页。 使用 **`top`** 和 **`skip`** 参数可顺利地发出搜索请求，接收搜索结果总集，并通过其中易于管理的有序子集轻松完成效果良好的搜索 UI 操作。 接收较小的结果子集时，还可以在搜索结果总集中获得文档计数。

有关对搜索结果分页的详细信息，请参阅 [如何在 Azure 搜索中对搜索结果分页](search-pagination-page-layout.md)一文。

### <a name="ordering-results"></a>对结果排序
接收搜索查询结果时，可请求 Azure 搜索按特定字段中的值对结果排序。 默认情况下，Azure 搜索根据每个文档的搜索评分（源自 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)）排名对结果排序。

如果希望 Azure 搜索返回按搜索评分以外的某个值排序的结果，可以使用 **`orderby`** 搜索参数。 对于地理空间值，可以指定 **`orderby`** 参数的值，使其包含字段名称及对 [**`geo.distance()` 函数**](query-odata-filter-orderby-syntax.md)的调用。 每个表达式可后接 `asc` 来指示按升序请求结果，或后接 **`desc`** 来指示按降序请求结果。 默认为升序。


### <a name="hit-highlighting"></a>突出显示
在 Azure 搜索中，使用 **`highlight`**、**`highlightPreTag`** 和 **`highlightPostTag`** 参数可轻松强调搜索结果中与搜索查询相匹配的确切部分。 可以指定应强调其匹配文本的可搜索字段，还可指定要附加到 Azure 搜索所返回匹配文本的开头和结尾的精确字符串标记。

## <a name="see-also"></a>另请参阅

+ [Azure 搜索中全文搜索的工作原理（查询分析体系结构）](search-lucene-query-architecture.md)
+ [搜索资源管理器](search-explorer.md)
+ [如何在 .NET 中进行查询](search-query-dotnet.md)
+ [如何在 REST 中进行查询](search-create-index-rest-api.md)
