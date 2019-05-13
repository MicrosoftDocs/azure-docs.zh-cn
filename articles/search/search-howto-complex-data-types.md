---
title: 如何为复杂数据类型建模 - Azure 搜索
description: 可以在使用复杂类型和集合数据类型的 Azure 搜索索引中建模嵌套或分层数据结构。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
tags: complex data types; compound data types; aggregate data types
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: seodec2018
ms.openlocfilehash: 00606ed5cbcd8681748241e9404c6e6e5aa95021
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147315"
---
# <a name="how-to-model-complex-data-types-in-azure-search"></a>如何在 Azure 搜索中为复杂数据类型建模

外部数据集用于填充 Azure 搜索索引有时包括分层或嵌套子结构。 示例可能包括多个位置和电话号码的单个客户、 多个颜色和大小的单一 SKU，单个书籍的多位作者等等。 在建模术语中，可能会看到这些结构称为*复杂数据类型*，*复合数据类型*，*复合数据类型*，或*聚合数据类型*。 在 Azure 搜索术语中，复杂类型是包含子级 （子字段） 的字段本身可以是简单或复杂。 这是类似于编程语言中的结构化的数据类型。 复杂的字段可以是单个字段，表示文档中的单个对象或一个集合，它表示对象的数组

Azure 搜索以本机方式支持复杂类型和集合。 在一起，这些类型，可在 Azure 搜索索引中几乎所有嵌套的 JSON 结构的模型。 在以前版本的 Azure 搜索 Api，仅平展的行集无法导入。 在最新版本中，索引可以现在更紧密地对应于源数据。 换而言之，如果您的源数据具有复杂类型，索引可以还具有复杂类型。

若要开始，我们建议[酒店的数据集](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md)，以便在加载**导入数据**向导在 Azure 门户中。 向导在源中检测到复杂类型，并提供建议索引架构基于检测到的结构。

> [!Note]
> 对复杂类型的支持已在公开发布`api-version=2019-05-06`。 
>
> 如果你搜索的解决方案基于集合中的平展数据集的早期解决方法，应更改索引以包含复杂类型，如最新的 API 版本中受支持。 有关升级的 API 版本的详细信息，请参阅[升级到最新的 REST API 版本](search-api-migration.md)或[升级到最新的.NET SDK 版本](search-dotnet-sdk-migration-version-9.md)。

## <a name="example-of-a-complex-structure"></a>复杂结构示例

下面的 JSON 文档是简单的字段和复杂的字段组成。 复杂字段，如`Address`和`Rooms`，具有子字段。 `Address` 具有一组这些子字段的值，因为它是单个对象在文档中。 与此相反，`Rooms`集合中有多个组及其子字段的值，一个用于每个对象。

```json
{
    "HotelId": "1",
    "HotelName": "Secret Point Motel",
    "Description": "Ideally located on the main commercial artery of the city in the heart of New York.",
    "Address": {
        "StreetAddress": "677 5th Ave",
        "City": "New York",
        "StateProvince": "NY"
    },
    "Rooms": [
        {
            "Description": "Budget Room, 1 Queen Bed (Cityside)",
            "Type": "Budget Room",
            "BaseRate": 96.99,
        },
        {
            "Description": "Deluxe Room, 2 Double Beds (City View)",
            "Type": "Deluxe Room",
            "BaseRate": 150.99,
        },
    ]
}
```

## <a name="creating-complex-fields"></a>创建复杂的字段

与任何索引定义中，可以使用门户中，如[REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)，或[.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) ，创建包含复杂类型的架构。 

下面的示例演示具有简单的字段、 集合和复杂类型的 JSON 索引架构。 请注意，在复杂类型中，每个子字段都有一种类型，可能有属性，只需为顶级字段执行操作。 架构对应于上面的示例数据。 `Address` 是一个复杂的字段，不是的集合 （一家酒店都有一个地址）。 `Rooms` 为复杂集合字段 （一家酒店具有许多聊天室）。

<!---
For indexes used in a [push-model data import](search-what-is-data-import.md) strategy, where you are pushing a JSON data set to an Azure Search index, you can only have the basic syntax shown here: single complex types like `Address`, or a `Collection(Edm.ComplexType)` like `Rooms`. You cannot have complex types nested inside other complex types in an index used for push-model data ingestion.

Indexers are a different story. When defining an indexer, in particular one used to build a knowledge store, your index can have nested complex types. An indexer is able to hold a chain of complex data structures in-memory, and when it includes a skillset, it can support highly complex data forms. For more information and an example, see [How to get started with Knowledge Store](knowledge-store-howto.md).
-->

```json
{
    "name": "hotels",
    "fields": [
        {   "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true    },
        {   "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
        { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
        {   "name": "Address", "type": "Edm.ComplexType",
            "fields": [{
                    "name": "StreetAddress",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true  },
                {
                    "name": "City",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "StateProvince",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                }
            ]
        },
        {
            "name": "Rooms",
            "type": "Collection(Edm.ComplexType)",
            "fields": [{
                    "name": "Description",
                    "type": "Edm.String",
                    "searchable": true,
                    "analyzer": "en.lucene"
                },
                {
                    "name": "Type",
                    "type": "Edm.String",
                    "searchable": true
                },
                {
                    "name": "BaseRate",
                    "type": "Edm.Double",
                    "filterable": true,
                    "facetable": true
                },
            ]
        }
    ]
}
```
## <a name="updating-complex-fields"></a>正在更新复杂域

所有[重建索引规则](search-howto-reindex.md)适用于字段一般情况下仍将应用于复杂的字段。 在这里，主要的规则的重述一些添加字段不需要重新生成索引，但大多数修改执行操作。

### <a name="structural-updates-to-the-definition"></a>对定义结构更新

可以在任何时间无需重新生成索引到复杂的字段添加新的子字段。 例如，将"ZipCode"添加到`Address`或"便利设施"到`Rooms`允许，就像将顶级字段添加到索引。 现有文档具有新的字段的 null 值，直到显式通过更新你的数据填充这些字段。

请注意，在复杂类型中，每个子字段都有一种类型，可能有属性，只需为顶级字段执行操作

### <a name="data-updates"></a>数据更新

与上传操作更新现有文档索引中的运行方式相同的方式对复杂和简单字段--替换所有字段。 但是，合并 （或 mergeOrUpload 时应用于现有文档） 不起作用相同的所有字段。 具体而言，合并没有合并集合中的元素的功能。 这适用于基元类型的集合以及复杂的集合。 若要更新集合，您将需要检索完整的集合值进行的更改，并将包含新集合在索引 API 请求。


## <a name="searching-complex-fields"></a>搜索复杂字段

自由的搜索表达式正常工作，具有复杂类型。 如果任何可搜索字段或子字段任意位置在文档中的与匹配，文档本身是一个匹配项。 

微妙的时你具有多个词语或运算符，且一些术语具有字段名称指定，按原样使用的查询更多[Lucene 语法](query-lucene-syntax.md)。 例如，此查询将尝试匹配这两个术语，"Portland"和"OR"针对两个子字段的地址字段：

```json
search=Address/City:Portland AND Address/State:OR
```

此类查询都是无关联的全文搜索 (与不同的筛选器，可以使用任何关联的一系列复杂的子字段的查询或所有，如 SQL 中的相关子查询)。 这意味着上面的 Lucene 查询将返回包含"波特兰，Maine"的文档，以及"波特兰，俄勒冈州"和其他俄勒冈州的城市。 这是因为针对整个文档中的指定字段的所有值计算每个子句，因此没有概念的"当前子文档"。 

 

## <a name="selecting-complex-fields"></a>选择复杂的字段

`$select`参数用于选择在搜索结果中返回哪些字段。 若要使用此参数选择复杂的字段的特定子字段，包含父字段和由斜杠分隔的子字段 (`/`)。

```json
$select=HotelName, Address/City, Rooms/BaseRate
```

必须将字段标记为可检索索引中如果你希望它们在搜索结果中。 只有标记为可检索的字段可在`$select`语句。 


## <a name="filter-facet-and-sort-complex-fields"></a>筛选器、 分面和排序复杂字段

相同[OData 路径语法](query-odata-filter-orderby-syntax.md)用作筛选器和审定搜索还可用于分面、 排序和搜索请求中选择字段。 对于复杂类型，规则将应用用于控制哪些子字段可以标记为可排序或可查找。 

### <a name="faceting-sub-fields"></a>分面子字段 

可以将标记为可查找任何子字段，除非它是类型`Edm.GeographyPoint`或`Collection(Edm.GeographyPoint)`。 

当分面导航结构返回文档计数时的计数是相对于父文档 （酒店） 不到复杂集合 （聊天室） 中的嵌套文档。 例如，假设一家酒店有 20 聊天室的类型"套件"。 给定此分面参数`facet=Rooms/Type`，分面计数将是一个父文档 （酒店） 和不中间子文档 （聊天室）。 

### <a name="sorting-complex-fields"></a>对复杂字段进行排序

排序操作应用于文档 （酒店） 和不子文档 （聊天室）。 在必须将复杂类型集合，例如聊天室，请务必认识到，不能在对排序聊天室根本。 事实上，不能对任何集合进行排序。 

排序操作的工作单值字段时是否为简单字段，或为复杂类型中的子字段。 例如，`$orderby=Address/ZipCode`复杂类型是否可排序，因为每个酒店的只有一个邮政编码。 

摆有关排序规则，在索引字段中必须标记为可筛选和可排序，以用于`$orderby`语句。 

## <a name="next-steps"></a>后续步骤

 请尝试[酒店的数据集](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md)中**导入数据**向导。 你将需要提供用于访问数据的自述文件中的 Cosmos DB 连接信息。 
 
 使用该信息后，该向导的第一步是创建新的 Azure Cosmos DB 数据源。 进一步上在向导中，转到目标索引页中，你将看到索引具有复杂类型。 创建和加载此索引，并执行查询，以了解新的结构。

> [!div class="nextstepaction"]
> [导入、 索引和查询快速入门： 门户向导](search-get-started-portal.md)