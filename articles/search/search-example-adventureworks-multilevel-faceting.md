---
title: 示例：多层分面 - Azure 搜索
description: 了解如何为多层分类生成分面结构，创建可以包含在应用程序页面中的嵌套式导航结构。
author: cstone
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: chstone
ms.openlocfilehash: e17a91a35b69102e4e0ac6025559bbc32e71d8fb
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024125"
---
# <a name="example-multi-level-facets-in-azure-search"></a>示例：Azure 搜索中的多层分面

Azure 搜索架构不显式支持多层分类类别，但你可以对其进行近似处理，方法是：在进行索引编制之前先处理内容，然后对结果应用某些特殊的处理。 

## <a name="start-with-the-data"></a>从数据开始

本文中的示例基于先前的示例 - [为 AdventureWorks Inventory 数据库建模](search-example-adventureworks-modeling.md)，目的是演示 Azure 搜索中的多层分面。

AdventureWorks 有一个简单的双级分类，其中的关系为父子关系。 若要获取此结构的固定长度分类深度，可以使用简单的 SQL 联接查询将分类分组：

```T-SQL
SELECT 
  parent.Name Parent, category.Name Category, category.ProductCategoryId
FROM 
  SalesLT.ProductCategory category
LEFT JOIN
  SalesLT.ProductCategory parent
  ON category.ParentProductCategoryId=parent.ProductCategoryId
```

  ![查询结果](./media/search-example-adventureworks/prod-query-results.png "查询结果")

## <a name="indexing-to-a-collection-field"></a>为 Collection 字段编制索引

在包含此结构的索引中，在 Azure 搜索架构中创建 **Collection(Edm.String)** 字段来存储此数据，确保字段属性包含 searchable、filterable、facetable 和 retrievable。

现在，在为引用特定分类类别的内容编制索引时，请将分类作为数组提交，该数组包含来自各级别分类的文本。 例如，对于 `ProductCategoryId = 5 (Mountain Bikes)` 的实体，请将字段以 `[ "Bikes", "Bikes|Mountain Bikes"]` 形式提交

请注意，子类别值“Mountain Bikes”中包含父类别“Bikes”。 每个子类别应该嵌入整个路径（相对于根元素而言）。 竖线字符分隔符是任意的，但必须一致且不应显示在源文本中。 分隔符将用在应用程序代码中，目的是根据分面结果重新构造分类树。

## <a name="construct-the-query"></a>构造查询

在发出查询时包括以下分面规范（其中的分类是可分面的分类字段）：`facet = taxonomy,count:50,sort:value`

计数值必须足够高，否则无法返回所有可能的分类值。 AdventureWorks 数据包含 41 个不同的分类值，因此 `count:50` 已足够。

  ![分面的筛选器](./media/search-example-adventureworks/facet-filter.png "分面的筛选器")

## <a name="build-the-structure-in-client-code"></a>在客户端代码中生成结构

在客户端应用程序代码中重新构造分类树，方法是：在竖线字符上拆分每个分面值。

```javascript
var sum = 0
  , categories = {children:{},length:0}
  , results = getSearchResults();
separator = separator || '|';
field = field || 'taxonomy';
results['@search.facets'][field].forEach(function(d) {
  var node = categories;
  var parts = d.value.split(separator);
  sum += d.count;
  parts.forEach(function(c, i) {
    if (!_(node.children).has(c)) {
      node.children[c] = {};
      node.children[c].count = d.count;
      node.children[c].children = {};
      node.children[c].length = 0;
      node.children[c].filter = parts.slice(0,i+1).join(separator);
      node.length++;
    }
    node = node.children[c];
  });
});
categories.count = sum;
```

**categories** 对象现在可以用来呈现计数准确的可折叠分类树：

  ![多层分面筛选器](./media/search-example-adventureworks/multi-level-facet.png "多层分面筛选器")

 
树中的每个链接应该应用相关的筛选器。 例如：

+ **taxonomy/any**`(x:x eq 'Accessories')` 返回 Accessories 分支中的所有文档
+ **taxonomy/any**`(x:x eq 'Accessories|Bike Racks')` 仅返回 Accessories 分支下子类别为“Bike Racks”的文档。

此方法在升级后可以涵盖更复杂的场景，例如在不同的父类别下出现的更深的分类树和重复的子类别（例如，`Bike Components|Forks` 和 `Camping Equipment|Forks`）。

> [!TIP]
> 查询速度受返回的分面数的影响。 若要支持极大的分类集，考虑添加可分面的 **Edm.String** 字段，用于保存每个文档的顶级分类值。 然后应用如上所述的相同方法，但在用户展开顶级节点时仅执行集合-分面查询（在根分类字段上进行筛选）。 或者，在不需要 100% 召回的情况下，直接将分面计数降至一个合理的数字，确保分面条目按计数排序。

## <a name="see-also"></a>另请参阅

示例：[为 Azure 搜索的 AdventureWorks Inventory 数据库建模](search-example-adventureworks-modeling.md)