---
title: 使用"简单"的搜索语法的 Azure 搜索的查询示例
description: 用于查询 Azure 搜索索引的全文搜索、筛选搜索、地理搜索、分面搜索和其他搜索字符串的简单查询示例。
author: HeidiSteen
manager: cgronlun
tags: Simple query analyzer syntax
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 0c47212e51725e7d4a173c441709dca739d4e357
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024528"
---
# <a name="query-examples-using-the-simple-search-syntax-in-azure-search"></a>在 Azure 搜索中使用"简单"的搜索语法的查询示例

[简单查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)调用默认查询分析器，用于对 Azure 搜索索引执行全文搜索查询。 简单查询分析器速度很快，处理对象是 Azure 搜索中的全文搜索、筛选及分面搜索和地理搜索等常见方案。 本文逐步展示了一些示例，它们显示在使用简单语法时可用的查询操作。

备选的查询语法是[完整的 Lucene](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)，它支持模糊搜索和通配符搜索等更复杂的查询结构，而这可能需要额外的处理时间。 要获取完整语法的详细信息和演示示例，请参阅 [Lucene 语法查询示例](search-query-lucene-examples.md)。

## <a name="formulate-requests-in-postman"></a>在 Postman 中创建请求

下面的示例使用“纽约工作岗位”搜索索引，它包含基于[纽约市开放数据](https://nycopendata.socrata.com/)计划提供的数据集得出的岗位。 此数据不应认为是最新或完整数据。 该索引位于 Microsoft 提供的一项沙盒服务上，也就是说无需 Azure 订阅或 Azure 搜索即可试用这些查询。

要在 GET 上发出 HTTP 请求，需具备 Postman 或其等效工具。 有关详细信息，请参阅[使用 REST 客户端进行浏览](search-fiddler.md)。

### <a name="set-the-request-header"></a>设置请求标头

1. 在请求标头中，将“Content-Type”设为 `application/json`。

2. 添加 api-key，并将其设为此字符串：`252044BE3886FE4A8E3BAA4F595114BB`。 它是托管“纽约工作岗位”索引的沙盒搜索服务的查询密钥。

指定请求标头后，只需更改“search=”字符串即可在本文中的各项查询中重复使用。 

  ![Postman 请求标头](media/search-query-lucene-examples/postman-header.png)

### <a name="set-the-request-url"></a>设置请求 URL

请求是一个与包含 Azure 搜索终结点和搜索字符串的 URL 配对的 GET 命令。

  ![Postman 请求标头](media/search-query-lucene-examples/postman-basic-url-request-elements.png)

URL 组合具备以下元素：

+ `https://azs-playground.search.windows.net/` 是由 Azure 搜索开发团队维护的沙盒搜索服务。 
+ `indexes/nycjobs/` 是该服务的索引集合中的“纽约工作岗位”索引。 请求中需同时具备服务名称和索引。
+ `docs` 是包含所有可搜索内容的文档集合。 请求标头中提供的查询 api-key 仅适用于针对文档集合的读取操作。
+ `api-version=2019-05-06` 设置了 api-version（每个请求都需具备此参数）。
+ `search=*` 是查询字符串，此元素在初始查询中为 NULL，返回前 50 个结果（此为默认情况）。

## <a name="send-your-first-query"></a>发送自己的第一个查询

进行验证，将以下请求粘贴至 GET 并单击“发送”。 结果以详细的 JSON 文档形式返回。 返回整个文档，其中可以查看所有字段并将所有值。

作为验证步骤并查看文档结构，请将此 URL 粘贴到 REST 客户端。

  ```http
  https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search=*
  ```

查询字符串 `search=*` 是一个未指定的搜索，它与 NULL 或空搜索等效。 它的用处不大，但却是你能执行的最简单的搜索。

可选择将 `$count=true` 添加到 URL，以便返回一个符合搜索条件的文档的计数。 在空搜索字符串上，这就是索引中的所有文档（在“纽约工作岗位”例子中，数量约为 2800）。

## <a name="how-to-invoke-simple-query-parsing"></a>如何调用简单查询分析

如果是交互式查询，无需指定任何内容：默认为简单查询。 在代码中，如果之前调用过 **queryType=full** 以选择完整查询语法，可以使用 **queryType=simple** 重置默认值。

## <a name="example-1-field-scoped-query"></a>示例 1：字段范围查询

第一个示例并未特定于分析器，但我们将先使用它来介绍第一个基本查询概念，即“包含”。 本示例显示查询执行情况以及对几个特定字段的响应。 当你的工具是 Postman 或搜索资源管理器时，了解如何构建可读的 JSON 响应非常重要。 

出于简洁目的，该查询仅针对 business_title 字段并指定仅返回职位。 语法是 searchFields 和 select，前者将查询执行限制为只执行 business_title 字段，后者指定响应中包含哪些字段。

### <a name="partial-query-string"></a>部分查询字符串

```http
searchFields=business_title&$select=business_title&search=*
```

下面是使用以逗号分隔列表中的多个字段相同的查询。

```http
search=*&searchFields=business_title, posting_type&$select=business_title, posting_type
```

### <a name="full-url"></a>完整的 URL

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&searchFields=business_title&$select=business_title&search=*
```

此查询的响应应与以下屏幕截图类似。

  ![Postman 示例响应](media/search-query-lucene-examples/postman-sample-results.png)

你可能已经注意到响应中的搜索分数。 由于搜索不是全文搜索或者没有应用条件，因此不存在排名时评分统统为 1。 对于不带条件的空搜索，按任意顺序返回行。 包括实际条件时，能看到搜索分数变成有意义的值。

## <a name="example-2-look-up-by-id"></a>示例 2：按 ID 查找

本示例不太典型，但在评估搜索行为时，可以检查特定文档的整个内容，以理解它为何包含或不包含在结果中。 若要返回整个文档，请使用[查找操作](https://docs.microsoft.com/rest/api/searchservice/lookup-document)传入文档 ID。

所有文档都有一个唯一标识符。 要在查找查询中试用此语法，请先返回一个文档 ID 列表，以便找到要使用的文档。 对于纽约工作岗位，标识符存储在 `id` 字段中。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&searchFields=id&$select=id&search=*
```

下面的示例是基于 `id`“9E1E3AF9-0660-4E00-AF51-9B654925A2D5”返回特定文档的查找查询，最早出现在前面的响应中。 以下查询返回整个文档，而不仅是所选字段。 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs/9E1E3AF9-0660-4E00-AF51-9B654925A2D5?api-version=2019-05-06&$count=true&search=*
```

## <a name="example-3-filter-queries"></a>示例 3：筛选器查询

[筛选器语法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)是可以配合 **search** 使用或单独使用的 OData 表达式。 如果筛选表达式能够完全限定所需的文档，则不带 search 参数的单独筛选器很有用。 不使用查询字符串也就不会执行词法或语言分析、评分（所有评分为 1）和排名。 请注意，搜索字符串为空。

```http
POST /indexes/nycjobs/docs/search?api-version=2019-05-06
    {
      "search": "",
      "filter": "salary_frequency eq 'Annual' and salary_range_from gt 90000",
      "select": "select=job_id, business_title, agency, salary_range_from",
      "count": "true"
    }
```

如果一起使用，则会先对整个索引应用 filter，再对筛选结果执行 search。 filter 可减少 search 查询需要处理的文档集，因此是一种非常有用的技术，可用于提高查询性能。

  ![筛选器查询响应](media/search-query-simple-examples/filtered-query.png)

若要使用 GET 在 Postman 中尝试此查询，可以粘贴以下字符串：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,business_title,agency,salary_range_from&search=&$filter=salary_frequency eq 'Annual' and salary_range_from gt 90000
```

另一种合并筛选器和搜索的有效方法是通过筛选表达式中的 **`search.ismatch*()`**，在其中可以使用筛选器中的搜索查询。 此筛选表达式使用计划中的通配符来选择包含字词 plan、planner、planning 等的 business_title。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,business_title,agency&search=&$filter=search.ismatch('plan*', 'business_title', 'full', 'any')
```

有关该函数的详细信息，请参阅[“筛选器示例”中的 search.ismatch](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)。

## <a name="example-4-range-filters"></a>示例 4：范围筛选器

通过任何数据类型的 **`$filter`** 表达式支持范围筛选。 以下示例搜索数字和字符串字段。 

数据类型在范围筛选器中很重要，当数字数据位于数字字段且字符串数据位于字符串字段中时效果最佳。 由于 Azure 搜索中的数字字符串不可比较，因此字符串字段中的数字数据不适用于范围。 

为方便阅读，以下示例采用 POST 格式（数字范围后接文本范围）：

```http
POST /indexes/nycjobs/docs/search?api-version=2019-05-06
    {
      "search": "",
      "filter": "num_of_positions ge 5 and num_of_positions lt 10",
      "select": "job_id, business_title, num_of_positions, agency",
      "orderby": "agency",
      "count": "true"
    }
```
  ![数字范围的范围筛选器](media/search-query-simple-examples/rangefilternumeric.png)


```http
POST /indexes/nycjobs/docs/search?api-version=2019-05-06
    {
      "search": "",
      "filter": "business_title ge 'A*' and business_title lt 'C*'",
      "select": "job_id, business_title, agency",
      "orderby": "business_title",
      "count": "true"
    }
```

  ![文本范围的范围筛选器](media/search-query-simple-examples/rangefiltertext.png)

也可以使用 GET 在 Postman 中尝试这些查询：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&search=&$filter=num_of_positions ge 5 and num_of_positions lt 10&$select=job_id, business_title, num_of_positions, agency&$orderby=agency&$count=true
```

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&search=&$filter=business_title ge 'A*' and business_title lt 'C*'&$select=job_id, business_title, agency&$orderby=business_title&$count=true
```

> [!NOTE]
> 基于值的范围分面是常见的搜索应用程序要求。 有关为分面导航结构生成筛选器的详细信息和示例，请参阅[*如何实现分面导航*中的“基于范围的筛选器”](search-faceted-navigation.md#filter-based-on-a-range)。

## <a name="example-5-geo-search"></a>示例 5：地理搜索

示例索引包含带有纬度和经度坐标的 geo_location 字段。 此示例使用 [geo.distance 函数](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)来筛选从起点开始，直到所提供的任意距离（以公里为单位）圆周范围内的文档。 可以调整查询 (4) 中的最后一个值，以缩小或放大查询的表面积。

为方便阅读，以下示例采用 POST 格式：

```http
POST /indexes/nycjobs/docs/search?api-version=2019-05-06
    {
      "search": "",
      "filter": "geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4",
      "select": "job_id, business_title, work_location",
      "count": "true"
    }
```
为更具可读性的结果，搜索结果中剪裁掉，以包括作业 ID、 职务和工作位置。 起始坐标是从索引中的随机文档（在本例中，为斯塔顿岛上的某个工位）获取的。

也可以使用 GET 在 Postman 中尝试此查询：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search=&$select=job_id, business_title, work_location&$filter=geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4
```

## <a name="example-6-search-precision"></a>示例 6：搜索精度

字词查询是独立评估的单个字词（可能很多是这样）。 短语查询用引号括起来，作为逐字字符串进行评估。 匹配精度由运算符和 searchMode 控制。

示例 1：`&search=fire` 返回 150 个结果，即整个文档中包含“fire”一词的所有匹配项。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search=fire
```

示例 2：`&search=fire department` 返回 2002 个结果。 针对此文档返回了包含“fire”或“department”的匹配项。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search=fire department
```

示例 3：`&search="fire department"` 返回 82 个结果。 将该字符串用引号引起来，构成对这两个词的逐字搜索，在索引中包含该组合词的已标记化的字词中查找匹配项。 这就解释了为何诸如 `search=+fire +department` 之类的搜索是不等效的。 需具备两个字词，但独立扫描它们。 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&search="fire department"
```

## <a name="example-7-booleans-with-searchmode"></a>示例 7：使用 searchMode 的布尔值

简单语法支持字符形式的布尔运算符 (`+, -, |`) 。 searchMode 参数用于在精准率和召回率之间做出权衡，其中 `searchMode=any` 倾向于召回率（符合任何条件的文档都能进入结果集），而 `searchMode=all` 倾向于精准率（符合所有条件的文档才能进入结果集）。 默认为 `searchMode=any`；在使用多个运算符堆叠查询并获取更广泛而不是更窄的结果时，这可能会产生混淆。 在使用 NOT 时尤为如此，该运算符导致结果包括所有“不含”特定字词的文档。

使用默认的 searchMode (any) 时，返回了 2800 个文档：其中有包含多部分字词“fire department”的文档，以及所有不带有字词“Metrotech Center”的文档。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&searchMode=any&search="fire department"  -"Metrotech Center"
```

  ![搜索模式 - any](media/search-query-simple-examples/searchmodeany.png)

若将 searchMode 更改为 `all`，会强制累积条件并返回一个较小的结果集，它只有 21 个文档，其文档数是包含完整短语“fire department”的文档减去 Metrotech Center 工作岗位数之差。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&searchMode=all&search="fire department"  -"Metrotech Center"
```
  ![搜索模式 - all](media/search-query-simple-examples/searchmodeall.png)

## <a name="example-8-structuring-results"></a>示例 8：构建结果

有多个参数控制着搜索结果中包括哪些字段、每批返回多少文档以及排列顺序。 此示例重新设置了上述几个示例，使用 $select 语句和逐字搜索条件将结果限制为仅包含特定字段，返回了 82 个匹配项 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"
```
将此改动追加到上一示例，即可按职位排序。 这种排序是可行的，因为在该索引中 civil_service_title 是可排序的。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title
```

使用 $top 参数可实现结果分页，本例中返回了前 5 个文档：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=0
```

要获取后续 5 个文档，请跳过第一批：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2019-05-06&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=5
```

## <a name="next-steps"></a>后续步骤
尝试在代码中指定查询。 以下链接介绍如何使用默认的简单语法为 .NET 和 REST API 设置搜索查询。

* [使用 .NET SDK 查询 Azure 搜索索引](search-query-dotnet.md)
* [使用 REST API 查询 Azure 搜索索引](search-create-index-rest-api.md)

可在以下链接找到其他语法参考、查询体系结构和示例：

+ [生成高级查询的 Lucene 语法查询示例](search-query-lucene-examples.md)
+ [Azure 搜索中全文搜索的工作原理](search-lucene-query-architecture.md)
+ [简单的查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [完整 Lucene 查询](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [筛选器和 Orderby 语法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)
