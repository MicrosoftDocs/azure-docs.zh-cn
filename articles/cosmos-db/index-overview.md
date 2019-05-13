---
title: Azure Cosmos DB 中的索引
description: 了解 Azure Cosmos DB 中索引的工作原理。
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: thweiss
ms.openlocfilehash: 44706e5ebe2442dcb45dfc45e2c322938cf7dca9
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65068659"
---
# <a name="indexing-in-azure-cosmos-db---overview"></a>Azure Cosmos DB-概述中编制索引

Azure Cosmos DB 不限架构的数据库，可用于循环访问你的应用程序而无需处理架构或索引管理。 默认情况下，Azure Cosmos DB 自动索引中的所有项的每个属性您[容器](databases-containers-items.md#azure-cosmos-containers)而无需定义任何架构或配置辅助索引。

本文的目的是说明 Azure Cosmos DB 数据的索引以及是如何使用索引来提高查询性能。 建议为通过本部分探讨如何自定义之前[索引策略](index-policy.md)。

## <a name="from-items-to-trees"></a>从项到树

每次项存储在一个容器，其内容是投影为一个 JSON 文档，然后转换为树表示形式。 这意味着该项的每个属性获取表示为树中的一个节点。 伪根节点创建为项目的所有第一级别属性的父级。 叶节点包含的项执行的实际标量值。

作为示例，请考虑此项：

    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }

它将可以由以下树表示:

![上一项表示为一个树](./media/index-overview/item-as-tree.png)

请注意如何对数组进行编码在树中： 在数组中的每个条目获取带有该条目的数组中的索引的中间节点 (0、 1 等等。)。

## <a name="from-trees-to-property-paths"></a>从树属性路径

为什么 Azure Cosmos DB 会将项转换为树的原因是因为它允许通过这些树内的其路径中引用的属性。 若要获取的属性的路径，我们可以从根节点到该属性，遍历树，并将连接每个遍历节点的标签。

以下是上面所述的示例项目的每个属性的路径：

    /locations/0/country: "Germany"
    /locations/0/city: "Berlin"
    /locations/1/country: "France"
    /locations/1/city: "Paris"
    /headquarters/country: "Belgium"
    /headquarters/employees: 250
    /exports/0/city: "Moscow"
    /exports/1/city: "Athens"

时写入某个项，Azure Cosmos DB 有效地索引中每个属性的路径和及其对应的值。

## <a name="index-kinds"></a>索引种类

Azure Cosmos DB 目前支持两种类型的索引：

**范围**用于索引种类：

- 等式查询： 

   ```sql SELECT * FROM container c WHERE c.property = 'value'```

- 范围查询： 

   ```sql SELECT * FROM container c WHERE c.property > 'value'``` (适用于`>`， `<`， `>=`， `<=`， `!=`)

- `ORDER BY` 查询：

   ```sql SELECT * FROM container c ORDER BY c.property```

- `JOIN` 查询： 

   ```sql SELECT child FROM container c JOIN child IN c.properties WHERE child = 'value'```

标量值 （字符串或数字），可以使用范围索引。

**空间**用于索引种类：

- 地理空间距离查询： 

   ```sql SELECT * FROM container c WHERE ST_DISTANCE(c.property, { "type": "Point", "coordinates": [0.0, 10.0] }) < 40```

- 在查询中的地理空间信息： 

   ```sql SELECT * FROM container c WHERE ST_WITHIN(c.property, {"type": "Point", "coordinates": [0.0, 10.0] } })```

空间索引可应用于的格式正确[GeoJSON](geospatial.md)对象。 目前支持点、 Linestring 和多边形。

**复合**用于索引种类：

- `ORDER BY` 对多个属性的查询： 

   ```sql SELECT * FROM container c ORDER BY c.firstName, c.lastName```

## <a name="querying-with-indexes"></a>使用索引进行查询

提取数据编制索引时的路径使其易于查找索引，当处理查询时。 通过匹配`WHERE`建立索引的路径的列表查询的子句，则可以标识非常快速地匹配查询谓词的项。

例如，请考虑以下查询： `SELECT location FROM location IN company.locations WHERE location.country = 'France'`。 查询谓词 （筛选项，其中的任何位置及其国家/地区与具有"France"） 与中以红色突出显示的路径相匹配：

![匹配一个树内的特定路径](./media/index-overview/matching-path.png)

> [!NOTE]
> `ORDER BY`依据单一属性进行排序的子句*始终*需要范围索引，并且如果它所引用的路径没有任何一个将失败。 同样，多`ORDER BY`查询*始终*需要组合索引。

## <a name="next-steps"></a>后续步骤

阅读以下文章中有关索引的详细信息：

- [索引策略](index-policy.md)
- [如何管理索引策略](how-to-manage-indexing-policy.md)
