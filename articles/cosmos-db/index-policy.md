---
title: Azure Cosmos DB 索引编制策略
description: 了解如何配置和更改的默认索引策略的自动索引并提高性能，Azure Cosmos DB 中。
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: thweiss
ms.openlocfilehash: c7f2ccd2c074f2488c86b45a09859b308655df8d
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65068602"
---
# <a name="indexing-policies-in-azure-cosmos-db"></a>Azure Cosmos DB 中的索引策略

在 Azure Cosmos DB 中，每个容器具有决定了应该如何建立容器的项的索引的索引策略。 默认的新索引策略创建了容器索引强制实施的任何字符串或数字，范围索引的每个项的每个属性，对于任何 GeoJSON 对象的空间索引键入点。 这样，您无需考虑索引和索引管理提前获取高查询性能。

在某些情况下，你可能想要替代此自动行为，以更好地满足你的要求。 可以通过设置容器的索引策略来自定义其*索引模式*，并包括或排除*属性路径*。

## <a name="indexing-mode"></a>索引模式

Azure Cosmos DB 支持两种索引模式：

- **一致**：如果容器的索引策略设置为一致，如创建、 更新或删除项的索引时同步更新。 这意味着将读取查询的一致性[为帐户配置的一致性](consistency-levels.md)。

- **无**：如果容器的索引策略设置为 None，编制索引时有效地禁用了该容器。 这常用容器用作纯的键-值存储，而无需辅助索引时。 它还可以帮助加快大容量插入操作。

## <a name="including-and-excluding-property-paths"></a>包括和排除属性路径

自定义索引策略可以指定显式包括或从索引中排除的属性路径。 通过优化的路径中编制索引的数量，可以降低您的容器使用的存储量并提高写入操作的延迟。 这些路径定义以下[索引的概述部分中所述的方法](index-overview.md#from-trees-to-property-paths)增加了以下信息：

- 通向标量值 （字符串或数字） 的路径结尾 `/?`
- 通过一起讨论从数组的元素`/[]`表示法 (而不是`/0`，`/1`等。)
- `/*`可以使用通配符来匹配节点下的任何元素

再次执行相同的示例：

    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 }
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }

- `headquarters`的`employees`路径是 `/headquarters/employees/?`
- `locations``country`路径是 `/locations/[]/country/?`
- 到下的任何内容的路径`headquarters`是 `/headquarters/*`

中的索引策略明确包含路径，它也必须定义应应用于哪些索引类型，该路径，以及对于每个索引类型，此索引应用到的数据类型：

| 索引类型 | 允许的目标数据类型 |
| --- | --- |
| 范围 | 字符串或数字 |
| 空间 | 点、 LineString 或多边形 |

例如，我们可以包括`/headquarters/employees/?`路径以及指定那些`Range`索引应适用于在该路径上两`String`和`Number`值。

### <a name="includeexclude-strategy"></a>包括/排除策略

任何索引策略必须包含的根路径`/*`为包含或排除的路径。

- 包括要有选择性地排除不需要要编制索引的路径的根路径。 这是建议的方法，它可以让 Azure Cosmos DB 主动索引可能会添加到您的模型的任何新属性。
- 排除要有选择性地包含需要编制索引的路径的根路径。

- 有关路径替换为包含的常规字符： 字母数字字符和 _ （下划线），无需转义围绕双引号括起来 （例如，"/ 路径 /？"） 的路径字符串。 对于与其他特殊字符的路径，您需要进行转义围绕双引号括起来的路径字符串 (例如，"/\"路径 abc\"/？")。 如果希望在你的路径中特殊字符，可转义每个路径的安全性。 功能上，如果转义 Vs 不仅仅具有特殊字符的每个路径，它不起作用。

请参阅[本节](how-to-manage-indexing-policy.md#indexing-policy-examples)为索引策略示例。

## <a name="composite-indexes"></a>组合索引

查询`ORDER BY`两个或多个属性都需要一个组合索引。 目前，复合索引仅供多`ORDER BY`查询。 默认情况下，因此，应该定义没有复合索引[添加复合索引](how-to-manage-indexing-policy.md#composite-indexing-policy-examples)根据需要。

在定义了组合索引，需要指定：

- 两个或多个属性路径。 序列中的属性路径被定义相关问题。
- 顺序 （升序或降序）。

使用复合索引时使用的以下注意事项：

- 如果复合索引路径不匹配的 ORDER BY 子句中的属性序列，通过然后组合索引不能支持的查询

- 组合索引路径 （升序或降序） 的顺序也应匹配 ORDER BY 子句中的顺序。

- 组合索引还支持在所有路径上的 ORDER BY 子句具有相反的顺序。

请考虑下面的示例，其中属性定义组合索引 a、 b 和 c:

| **组合索引**     | **示例`ORDER BY`查询**      | **支持索引吗？** |
| ----------------------- | -------------------------------- | -------------- |
| ```(a asc, b asc)```         | ```ORDER BY  a asc, bcasc```        | ```Yes```            |
| ```(a asc, b asc)```          | ```ORDER BY  b asc, a asc```        | ```No```             |
| ```(a asc, b asc)```          | ```ORDER BY  a desc, b desc```      | ```Yes```            |
| ```(a asc, b asc)```          | ```ORDER BY  a asc, b desc```       | ```No```             |
| ```(a asc, b asc, c asc)``` | ```ORDER BY  a asc, b asc, c asc``` | ```Yes```            |
| ```(a asc, b asc, c asc)``` | ```ORDER BY  a asc, b asc```        | ```No```            |

应自定义索引策略，以便可以提供所有必要`ORDER BY`查询。

## <a name="modifying-the-indexing-policy"></a>修改索引策略

可以在任意时间更新容器的索引策略[通过使用 Azure 门户或某个受支持的 Sdk](how-to-manage-indexing-policy.md)。 对索引策略的更新触发从旧索引转换为新密码，而执行联机就地 （因此，在操作期间会占用更多存储空间）。 旧策略索引有效地转换到新的策略不会影响的写入可用性或容器上预配的吞吐量。 索引转换是一个异步操作，并完成所需的时间取决于预配的吞吐量、 项的数目和其大小。 

> [!NOTE]
> 正在重新编制索引时，查询可能不会返回所有匹配的结果，并会进行截断而不返回任何错误。 这意味着查询结果可能不是一致，直到完成索引转换。 可以跟踪索引转换进度[通过使用某个 Sdk](how-to-manage-indexing-policy.md)。

如果新索引策略模式设置为一致，索引转换正在进行时，可以不应用任何其他索引策略更改。 通过索引策略模式设置为 None （它将立即删除索引），可以取消正在运行的索引转换。

## <a name="indexing-policies-and-ttl"></a>索引策略和 TTL

[生存时间 (TTL) 功能](time-to-live.md)需要索引上开启的容器处于活动状态。 这意味着：

- 不能激活其中索引模式设置为无的容器上的 TTL
- 不能将索引模式设置为无在容器上其中激活 TTL。

对于在没有属性路径需要编制索引，但 TTL 是必需的方案，可以使用带有索引策略：

- 索引编制模式设置为一致，并
- 任何包含的路径，并
- `/*` 作为唯一排除路径。

## <a name="obsolete-attributes"></a>已过时的特性

使用索引策略时，可能会遇到现已过时的以下属性：

- `automatic` 一个布尔值定义索引策略的根目录。 它现在被忽略，并可以将设置为`true`，当你使用该工具需要它。
- `precision` 在包括的路径的索引级别定义一个数字。 它现在被忽略，并可以将设置为`-1`，当你使用该工具需要它。
- `hash` 是一种索引类型，将立即被替换为范围种类。

## <a name="next-steps"></a>后续步骤

阅读以下文章中有关索引的详细信息：

- [索引概述](index-overview.md)
- [如何管理索引策略](how-to-manage-indexing-policy.md)
