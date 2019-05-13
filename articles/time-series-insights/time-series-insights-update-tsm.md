---
title: Azure 时序见解预览中的时序模型 | Microsoft Docs
description: 了解 Azure 时序见解时序模型。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/29/2019
ms.custom: seodec18
ms.openlocfilehash: a9de28c96c2833033a3811835f57cffcccdf4619
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190334"
---
# <a name="time-series-model"></a>时序模型

本文介绍 Azure 时序见解预览版的时序模型部分。 其中介绍了该模型本身、其功能，以及如何开始生成和更新你自己的模型。

一直以来，从 IoT 设备收集的数据缺少上下文信息，以致难以快速查找和分析传感器。 时序模型的主要动机是简化 IoT 数据的查找和分析。 为了实现此目标，它会启用时序数据的策划、维护和扩充，以帮助准备随时可供使用的数据集。

时序模型在查询和导航中发挥着至关重要的作用，因为它们可将设备和非设备实体上下文化。 保存在时序模型中的数据利用其中存储的公式来推动时序查询计算。

[![时间时序模型概述](media/v2-update-tsm/tsm.png)](media/v2-update-tsm/tsm.png#lightbox)

## <a name="key-capabilities"></a>关键功能

时序模型的目标是让用户毫不费力地管理时序上下文化，它在时序见解预览版中可以实现以下功能。 它可以帮助你：

* 创作和管理计算或公式、利用标量函数转换数据、聚合操作，等等。
* 定义父子关系以实现导航和引用，并提供时序遥测的上下文。
* 定义与实例字段的实例部分相关联的属性，并使用这些属性来创建层次结构。

## <a name="entity-components"></a>实体组件

时序模型具有三个核心组件：

* <a href="#time-series-model-types">时间时序模型类型</a>
* <a href="#time-series-model-hierarchies">时间时序模型层次结构</a>
* <a href="#time-series-model-instances">时间时序模型实例</a>

若要指定时序模型，来组织 Azure 时序见解数据组合这些组件。

## <a name="time-series-model-types"></a>时序模型类型

时序模型类型可帮助你定义用于执行计算的变量或公式。 类型与特定的时序见解实例相关联。 一个类型可以包含一个或多个变量。 例如，某个时序见解实例的类型为“温度传感器”，其中包括变量“平均温度”、“最小温度”和“最大温度”。 当数据开始流入时序见解时，我们将创建默认类型。 可在模型设置中检索和更新默认类型。 默认类型只包含一个用于统计事件数目的变量。

### <a name="time-series-model-type-json-example"></a>时序模型类型 JSON 示例

示例：

```JSON
{
    "name": "SampleType",
    "description": "This is sample type",
    "variables": {
        "Avg Temperature": {
            "kind": "numeric",
            "filter": null,
            "value": { "tsx": "$event.temperature.Double" },
            "aggregation": {"tsx": "avg($value)"}
        },
        "Count Temperature": {
            "kind": "aggregate",
            "filter": null,
            "value": null,
            "aggregation": {"tsx": "count()"}
        }
    }
}
```

有关时序模型类型的详细信息，请参阅[参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#types-api)。

### <a name="variables"></a>变量

时序见解类型包含变量，这些变量是基于事件中的值执行的计算。 时序见解变量定义包含公式和计算规则。 变量定义包括种类、值、筛选器、化简和边界。 变量存储在时序模型的类型定义中，可以通过查询 API 以内联方式提供，以重写存储的定义。

以下矩阵充当变量定义的图例：

[![变量定义表类型](media/v2-update-tsm/table.png)](media/v2-update-tsm/table.png#lightbox)

| 定义 | 描述 |
| --- | ---|
| 变量种类 |  *数值*并*聚合*支持类型 |
| 变量筛选器 | 变量筛选器指定一个可选的筛选器子句，用于根据条件限制计算中考虑的行数。 |
| 变量值 | 变量值应在计算中使用。 有关的数据点有问题，请参阅相关字段。 |
| 变量聚合 | 变量的聚合函数可以实现一部分计算。 时序见解支持正则聚合（即 *min*、*max*、*avg*、*sum* 和 *count*）。 |

## <a name="time-series-model-hierarchies"></a>时序模型层次结构

层次结构通过指定属性名称及其关系来组织实例。 可以创建一个或多个层次结构。 它们不需要是当前数据部分，但每个实例应映射到一个层次结构。 一个时序模型实例可以映射到一个或多个层次结构。

层次结构按“层次结构 ID”、“名称”和“源”定义。 层次结构具有路径，即，用户想要创建的层次结构的自上而下的父子顺序。 父子属性映射实例字段。

### <a name="time-series-model-hierarchy-json-example"></a>时序模型层次结构 JSON 示例

示例：

```JSON
{
    "id": "4c6f1231-f632-4d6f-9b63-e366d04175e3",
    "name": "Location",
    "source": {
        "instanceFieldNames": [
                "state",
                "city"
            ]
    }
}
```

有关时序模型层次结构的详细信息，请参阅[参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#hierarchies-api)。

### <a name="hierarchy-definition-behavior"></a>层次结构定义行为

考虑下面的示例，其中，层次结构 H1 的定义包含 *building*、*floor* 和 *room*：

```plaintext
 H1 = [“building”, “floor”, “room”]
```

根据实例字段，层次结构属性和值如下表中所示：

| 时序 ID | 实例字段 |
| --- | --- |
| ID1 | “building” = “1000”, “floor” = “10”, “room” = “55”  |
| ID2 | “building” = “1000”, “room” = “55” |
| ID3 | “floor” = “10” |
| ID4 | “building” = “1000”, “floor” = “10”  |
| ID5 | “building”、“floor”和“room”均未设置 |

在前面的示例中， **ID1**并**ID4**显示为 H1 Azure 时序见解资源管理器中的层次结构的一部分，其余部分进行分类下*无父级实例*因为它们不符合指定的数据层次结构。

## <a name="time-series-model-instances"></a>时序模型实例

实例是时序本身。 在大多数情况下，*deviceId* 或 *assetId* 是环境中资产的唯一标识符。 实例具有关联的描述性信息，称为实例属性。 实例属性最起码包括层次结构信息。 实例属性还可以包括有用的描述性数据，例如制造商、操作员或上次检修日期。

实例按 *typeId*、*timeSeriesId*、*name*、*description*、*hierarchyIds* 和 *instanceFields* 定义。 每个实例只映射到一个类型以及一个或多个层次结构。 实例从层次结构继承所有属性，可以添加其他 *instanceFields* 以进一步定义实例属性。

*instanceFields* 是实例的属性，并且是定义实例的任何静态数据。 它们定义层次结构或非层次结构属性的值，同时还支持索引，以便能够执行搜索操作。

*名称*属性是可选的区分大小写。 如果 *name* 不可用，则默认为时序 ID。 如果提供了 *name*，仍将在 Well（资源管理器中图表下方的网格）中提供时序 ID。

### <a name="time-series-model-instance-json-example"></a>时序模型实例 JSON 示例

示例：

```JSON
{
    "typeId": "1be09af9-f089-4d6b-9f0b-48018b5f7393",
    "timeSeriesId": ["sampleTimeSeriesId"],
    "name": "sampleName",
    "description": "Sample Instance",
    "hierarchyIds": [
        "1643004c-0a84-48a5-80e5-7688c5ae9295"
    ],
    "instanceFields": {
        "state": "California",
        "city": "Los Angeles"
    }
}
```

有关时序模型实例的详细信息，请参阅[参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#instances-api)。

### <a name="time-series-model-settings-example"></a>时序模型设置示例

示例：

```JSON
{
    "modelSettings": {
        "name": "DefaultModel",
        "timeSeriesIdProperties": [
            {
                "name": "id",
                "type": "String"
            }
        ],
        "defaultTypeId": "1be09af9-f089-4d6b-9f0b-48018b5f7393"
    }
}
```

有关时序模型设置的详细信息，请参阅[参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#model-settings-api)。

## <a name="next-steps"></a>后续步骤

- 参阅 [Azure 时序见解预览存储和入口](./time-series-insights-update-storage-ingress.md)

- 了解新的[时序模型](https://docs.microsoft.com/rest/api/time-series-insights/preview-model)。