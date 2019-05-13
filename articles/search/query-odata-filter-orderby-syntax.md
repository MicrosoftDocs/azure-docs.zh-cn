---
title: 适用于筛选器和 order-by 子句的 OData 表达式语法 - Azure 搜索
description: 适用于 Azure 搜索查询的筛选器和 order-by 表达式 OData 语法。
ms.date: 05/02/2019
services: search
ms.service: search
ms.topic: conceptual
author: Brjohnstmsft
ms.author: brjohnst
ms.manager: cgronlun
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: b1f77a9e0a3308098e5f6c699f2fc79e5c437f17
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024264"
---
# <a name="odata-expression-syntax-for-filters-and-order-by-clauses-in-azure-search"></a>Azure 搜索中适用于筛选器和 order-by 子句的 OData 表达式语法

Azure 搜索支持一组适用于 **$filter** 和 **$orderby** 表达式的 OData 表达式语法。 筛选表达式在查询分析期间进行求值，将搜索范围限制为特定字段或添加索引扫描期间使用的匹配条件。 Order-by 表达式作为后续处理步骤应用到结果集。 筛选器和 order-by 表达式都包含在查询请求中，遵循独立于**搜索**参数中使用的[简单](query-simple-syntax.md)或[完整](query-lucene-syntax.md)查询语法的 OData 语法。 本文提供筛选器和排序表达式中使用的 OData 表达式的参考文档。

## <a name="filter-syntax"></a>筛选器语法

**$filter** 表达式可以作为完全表达的查询独立执行，或用于优化具有其他参数的查询。 以下示例介绍了几个关键方案。 在第一个示例中，筛选器是查询的物质。


```POST
POST /indexes/hotels/docs/search?api-version=2019-05-06
    {
      "filter": "(baseRate ge 60 and baseRate lt 300) or hotelName eq 'Fancy Stay'"
    }
```

另一个常见用例是组合使用筛选器和 facet，其中筛选器根据用户驱动的 facet 导航选择减少查询外围应用：

```POST
POST /indexes/hotels/docs/search?api-version=2019-05-06
    {
      "search": "test",
      "facets": [ "tags", "baseRate,values:80|150|220" ],
      "filter": "rating eq 3 and category eq 'Motel'"
    }
```

### <a name="filter-operators"></a>筛选器运算符  

- 逻辑运算符（and、or、not）。  

- 比较表达式 (`eq, ne, gt, lt, ge, le`)。 字符串比较区分大小写。  

- 受支持的[实体数据模型](https://docs.microsoft.com/dotnet/framework/data/adonet/entity-data-model) (EDM) 类型的常量（有关受支持类型的列表，请参阅[支持的数据类型 [Azure 搜索]](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)）。 不支持集合类型的常量。  

- 对字段名称的引用。 仅 `filterable` 字段可用于筛选表达式。  

- 没有参数的 `any`。 这将测试 `Collection(Edm.String)` 类型的字段是否包含任何元素。  

- 具有有限 lambda 表达式支持的 `any` 和 `all`。 
    
  -   `Collection(Edm.String)` 类型的字段支持 `any/all`。 
    
  -   `any` 只能与简单的相等表达式或 `search.in` 函数配合使用。 简单表达式包含单一字段和文本值之间的比较，例如，`Title eq 'Magna Carta'`。
    
  -   `all` 只能与简单的不等表达式或 `not search.in` 配合使用。   

- 地理空间函数 `geo.distance` 和 `geo.intersects`。 `geo.distance` 函数以千米为单位返回两点之间的距离，一个点是字段，另一个点是作为筛选器一部分传递的常量。 如果给定的点位于给定的多边形范围内（其中该点是字段，多边形指定为作为筛选器一部分传递的常量），`geo.intersects` 函数会返回 true。  

  多边形是二维表面，作为定义边界环的一系列点存储（参见以下示例）。 多边形需要处于闭合状态，这意味着第一个点集和最后一个点集必须相同。 [多边形中的点必须采用逆时针顺序](https://docs.microsoft.com/rest/api/searchservice/supported-data-types#Anchor_1)。

  `geo.distance` 在 Azure 搜索中以千米为单位返回距离。 这与支持 OData 地理空间运算的其他服务不同，后者通常以米为单位返回距离。  

  > [!NOTE]  
  >  如果在筛选器中使用 geo.distance，则必须使用 `lt`、`le`、`gt` 或 `ge` 将函数返回的距离与常量进行比较。 比较距离时不支持使用 `eq` 和 `ne` 运算符。 例如，以下是 geo.distance 的正确用法：`$filter=geo.distance(location, geography'POINT(-122.131577 47.678581)') le 5`。  

- `search.in` 函数测试给定字符串字段是否等于给定的值列表之一。 它还可以用于 any 或 all，以便将字符串集合字段的单个值与给定的值列表进行比较。 字段与列表中每个值之间的相等性以区分大小写的方式进行确定，这与 `eq` 运算符的方式相同。 因此，`search.in(myfield, 'a, b, c')` 等表达式相当于 `myfield eq 'a' or myfield eq 'b' or myfield eq 'c'`，但 `search.in` 的表现会好得多。 

   `search.in` 函数的第一个参数是字符串字段引用（在 `any` 或 `all` 表达式中使用 `search.in` 的情况下，则为字符串集合字段中的范围变量）。 
  
   第二个参数是包含值列表的字符串，由空格和/或逗号分隔。 
  
   第三个参数是字符串的一个字符串，其中每个字符或此字符串的一部分被视为一个分隔符分析中的第二个参数的值列表时。 如果因为值包含空格和逗号而需要使用这些字符以外的分隔符，可以将可选的第三个参数指定为 `search.in`。 

  > [!NOTE]   
  > 某些方案需要将字段与大量常量值进行比较。 例如，使用筛选器实现安全剪裁可能需要将文档 ID 字段与请求用户被授予读取访问权限的 ID 列表进行比较。 在此类方案中，我们强烈建议使用 `search.in` 函数，而不要使用更复杂的相等表达式析取。 例如，使用 `search.in(Id, '123, 456, ...')` 而不是 `Id eq 123 or Id eq 456 or ....`。 
  >
  > 如果使用 `search.in`，则当第二个参数包含数百个或数千个值的列表时，可以获得次秒级响应时间。 请注意，尽管仍然受到最大请求大小的限制，但可以传递给 `search.in` 的项数没有明确限制。 但是，延迟会随着值数量的增长而增加。

- `search.ismatch` 函数将搜索查询作为筛选表达式的一部分进行求值。 与搜索查询匹配的文档将在结果集中返回。 此函数的以下重载可用：
  - `search.ismatch(search)`
  - `search.ismatch(search, searchFields)`
  - `search.ismatch(search, searchFields, queryType, searchMode)`

  其中： 
  
  - `search`：搜索查询（采用[简单](query-simple-syntax.md)或[完整](query-lucene-syntax.md)查询语法）。 
  - `queryType`：“简单”或“完整”，默认为“简单”。 指定 `search` 参数中使用的查询语言。
  - `searchFields`：要在其中进行搜索的可搜索字段的逗号分隔列表，默认为索引中的所有可搜索字段。    
  - `searchMode`：“any”或“all”，默认为“any”。 指示必须匹配任意搜索词还是全部搜索词才能将文档视为匹配。

  所有上述参数均相当于对应的[搜索请求参数](https://docs.microsoft.com/rest/api/searchservice/search-documents)。

- 与 `search.ismatch` 函数一样，对于与作为参数传递的搜索查询匹配的文档，`search.ismatchscoring` 函数会返回 true。 它们之间的区别在于，与 `search.ismatchscoring` 查询匹配的文档的相关性分数与文档整体分数有关，而对于 `search.ismatch`，文档分数不会发生更改。 此函数的以下重载可用于与 `search.ismatch` 的参数相同的参数：
  - `search.ismatchscoring(search)`
  - `search.ismatchscoring(search, searchFields)`
  - `search.ismatchscoring(search, searchFields, queryType, searchMode)`

  `search.ismatch` 和 `search.ismatchscoring` 函数互相完全正交，并与筛选器代数的其余部分完全正交。 这意味着两个函数可用于同一筛选表达式。 

### <a name="geospatial-queries-and-polygons-spanning-the-180th-meridian"></a>跨越 180 度经线的地理空间查询和多边形  
 对于许多地理空间查询库，构建包含 180 度经线（国际日期变更线线附近）的查询要么被禁止，要么需要变通方法，例如将多边形拆分为两个，子午线两侧各一个。  

 在 Azure 搜索中，如果查询形状为矩形并且坐标沿经度和纬度方向与网格布局对齐（例如，`geo.intersects(location, geography'POLYGON((179 65,179 66,-179 66,-179 65,179 65))'`），则包含 180 度经度的地理空间查询将按预期工作。 否则，对于非矩形的形状或未对齐的形状，请考虑使用拆分多边形方法。  

<a name="bkmk_limits"></a>

## <a name="filter-size-limitations"></a>筛选器大小限制 

 可以发送到 Azure 搜索的筛选表达式的大小和复杂性存在限制。 限制大致基于筛选器表达式中的子句数。 根据经验，如果有数百个子句，则就存在达到限制的风险。 我们建议正确设计应用程序，使之不会生成大小不受限制的筛选器。  


## <a name="filter-examples"></a>筛选器示例  

 查找小于 200 美元信用额度在 4 或更高分级的基本价格的所有酒店：  

```
$filter=baseRate lt 200.0 and rating ge 4
```

 查找除“Roach Motel”以外的自 2010 年以来经过翻修的所有酒店：  

```
$filter=hotelName ne 'Roach Motel' and lastRenovationDate ge 2010-01-01T00:00:00Z
```

 查找基本费率小于 200 美元自 2010 年的日期时间文字，其中包含太平洋标准时间的时区信息已翻新的所有酒店：  

```
$filter=baseRate lt 200 and lastRenovationDate ge 2010-01-01T00:00:00-08:00
```

 查找拥有停车场且不允许吸烟的所有酒店：  

```
$filter=parkingIncluded and not smokingAllowed
```

 \- 或 -  

```
$filter=parkingIncluded eq true and smokingAllowed eq false
```

 查找等级为豪华或包含停车场且评分为 5 分的所有酒店：  

```
$filter=(category eq 'Luxury' or parkingIncluded eq true) and rating eq 5
```

 查找带有“wifi”标签的所有酒店（其中每家酒店都将标签存储在 Collection(Edm.String) 字段中）：  

```
$filter=tags/any(t: t eq 'wifi')
```

 查找没有“motel”标签的所有酒店：  

```
$filter=tags/all(t: t ne 'motel')
```

 搜索带有任何标签的所有酒店：  

```
$filter=tags/any()
```

查找不具有标记的所有酒店：  

```
$filter=not tags/any()
```


 查找与给定参考点的距离在 10 千米范围内的所有酒店（其中 location 是 Edm.GeographyPoint 类型的字段）：  

```
$filter=geo.distance(location, geography'POINT(-122.131577 47.678581)') le 10
```

 查找描述为多边形的给定视区内的所有酒店（其中 location 是 Edm.GeographyPoint 类型的字段）。 请注意，多边形是闭合的（第一个点集和最后一个点集必须相同），且[这些点必须按逆时针顺序列出](https://docs.microsoft.com/rest/api/searchservice/supported-data-types#Anchor_1)。

```
$filter=geo.intersects(location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))')
```

 查找在“description”字段中没有值，或者将该值显式设置为 null 的所有酒店：  

```
$filter=description eq null
```

查找所有酒店名称等于 Roach motel 或预算酒店）。 短语包含空格，这是默认分隔符。 可以将 specicfy 在单引号中的一个替代分隔符作为第三个字符串参数：  

```
$filter=search.in(name, 'Roach motel,Budget hotel', ',')
```

查找名称等于“Roach motel”或“Budget hotel”并以“|”分隔的所有酒店：  

```
$filter=search.in(name, 'Roach motel|Budget hotel', '|')
```

查找带有“wifi”或“pool”标签的所有酒店：  

```
$filter=tags/any(t: search.in(t, 'wifi, pool'))
```

在标记中找到在集合中，如加热总是随身携带毛巾机架或 hairdryer 包含短语的匹配的项。 

```
$filter=tags/any(t: search.in(t, 'heated towel racks,hairdryer included', ','))
```

查找没有“motel”或“cabin”标签的所有酒店：  

```
$filter=tags/all(t: not search.in(t, 'motel, cabin'))
```

查找包含“waterfront”一词的文档。 此筛选器查询与包含 `search=waterfront` 的[搜索请求](https://docs.microsoft.com/rest/api/searchservice/search-documents)相同。

```
$filter=search.ismatchscoring('waterfront')
```

查找带有“hostel”一词且评分大于或等于 4 分的文档，或带有“motel”一词且评分等于 5 分的文档。 请注意，如果没有 `search.ismatchscoring` 函数，则无法表达此请求。

```
$filter=search.ismatchscoring('hostel') and rating ge 4 or search.ismatchscoring('motel') and rating eq 5
```

查找没有“luxury”一词的文档。

```
$filter=not search.ismatch('luxury') 
```

查找包含短语“ocean view”或评分等于 5 分的文档。 `search.ismatchscoring` 查询仅针对 `hotelName` 和 `description` 字段执行。
请注意，仅与析取的第二个子句匹配的文档也将被返回，即评分等于 5 分的酒店。 为了清楚地表明这些文档与表达式的任何得分部分都不匹配，它们返回的分数将为零。

```
$filter=search.ismatchscoring('"ocean view"', 'description,hotelName') or rating eq 5
```

在酒店描述中查找词条“hotel”和“airport”相距不超过 5 个字词且不允许吸烟的文档。 此查询使用[完整 Lucene 查询语言](query-lucene-syntax.md)。

```
$filter=search.ismatch('"hotel airport"~5', 'description', 'full', 'any') and not smokingAllowed 
```

## <a name="order-by-syntax"></a>Order-by 语法

**$orderby** 参数接受包含最多 32 个格式为 `sort-criteria [asc|desc]` 的表达式的逗号分隔列表。 排序条件可以是 `sortable` 字段的名称，也可以是对 `geo.distance` 或 `search.score` 函数的调用。 可以使用 `asc` 或 `desc` 显式指定排序顺序。 默认顺序为升序。

如果多个文档具有相同的排序条件且未使用 `search.score` 函数（例如，如果按 `rating` 数字字段排序，并且三个文档的评分均为 4 分），则会通过以降序按文档分数排序来打破平分的局面。 当文档分数相同时（例如，当请求中没有指定全文搜索查询时），平分文档的相对排序是不确定的。
 
可以指定多个排序条件。 表达式的顺序决定最终排序顺序。 例如，若要按分数降序排序，然后按评分排序，语法将为 `$orderby=search.score() desc,rating desc`。

**$orderby** 中 `geo.distance` 的语法与其在 **$filter** 中的语法相同。 如果在 **$orderby** 中使用 `geo.distance`，其应用到的字段必须为 `Edm.GeographyPoint` 类型，且还必须 `sortable`。  

**$orderby** 中 `search.score` 的语法为 `search.score()`。 `search.score` 函数不接受任何参数。  
 

## <a name="order-by-examples"></a>Order-by 示例

按基准费率对酒店进行升序排序：

```
$orderby=baseRate asc
```

按评分对酒店进行降序排序，然后按基准费率对酒店进行升序排序（请记住，升序是默认值）：

```
$orderby=rating desc,baseRate
```

按评分对酒店进行降序排序，然后按距给定坐标的距离进行升序排序：

```
$orderby=rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```

按 search.score 和评分对酒店进行降序排序，然后按距给定坐标的距离进行升序排序，以便在两个具有相同评分的酒店中，将距离最近的酒店列在前面：

```
$orderby=search.score() desc,rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```
<a name="bkmk_unsupported"></a>

## <a name="unsupported-odata-syntax"></a>不受支持的 OData 语法

-   算术表达式  

-   函数（距离和相交地理空间函数除外）  

-   包含任意 lambda 表达式的 `any/all`  

## <a name="see-also"></a>另请参阅  

+ [Azure 搜索中的分面导航](search-faceted-navigation.md) 
+ [Azure 搜索中的筛选器](search-filters.md) 
+ [搜索文档（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) 
+ [Lucene 查询语法](query-lucene-syntax.md)
+ [Azure 搜索中的简单查询语法](query-simple-syntax.md)   
