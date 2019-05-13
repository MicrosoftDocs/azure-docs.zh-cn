---
title: 使用 Azure 时序见解预览版塑造事件 | Microsoft Docs
description: 了解如何塑造 Azure 时序见解预览版的事件。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: 38072d111d51cc2d2c6265643b69a870a679a454
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466145"
---
# <a name="shape-events-with-azure-time-series-insights-preview"></a>使用 Azure 时序见解预览版塑造事件

本文介绍如何塑造 JSON 文件，以最大化 Azure 时序见解预览版的查询效率。

## <a name="best-practices"></a>最佳做法

必须考虑如何将事件发送到时序见解预览版。 也就是说，始终应该：

* 尽量高效地通过网络发送数据。
* 以特定的方式存储数据，以便更好地根据方案来聚合数据。

若要尽可能提高查询性能，请执行以下操作：

* 不要发送不必要的属性。 时序见解预览版按使用情况收费。 最好是存储后再处理要查询的数据。
* 对于静态数据，请使用实例字段。 此做法有助于避免通过网络发送静态数据。 实例字段是时序模型的组件，其工作方式类似于时序见解 GA 服务中的引用数据。 若要详细了解实例字段，请参阅[时序模型](./time-series-insights-update-tsm.md)。
* 在两个或多个事件中共享维度属性。 此做法可以更有效地通过网络发送数据。
* 不要使用深层数组嵌套。 时序见解预览版最多支持两个级别的包含对象的嵌套数组。 时序见解预览版会将消息中的数组平展成包含属性值对的多个事件。
* 如果所有或大多数事件只存在几个度量，最好是在同一个对象中将这些度量作为单独的属性发送。 单独发送度量可以减少事件数目，并可能提高查询的性能，因为需处理的事件更少。

## <a name="example"></a>示例

以下示例基于至少两个设备发送度量值或信号的场景。 度量值或信号可能包括“流速”、“引擎油压”、“温度”和“湿度”。

以下示例提到单个 IoT 中心消息，其中的外部数组包含通用维度值的共享节。 该外部数组使用时序实例数据来提高消息的效率。 时序实例包含设备元数据，这些数据不会根据每个事件变化，但提供适用于数据分析的属性。 若要减少通过网络发送的字节数并提高消息的效率，可以考虑批处理通用维度值并采用时序实例元数据。

### <a name="example-json-payload"></a>示例 JSON 有效负载

```JSON
[
    {
        "deviceId": "FXXX",
        "timestamp": "2018-01-17T01:17:00Z",
        "series": [
            {
                "Flow Rate ft3/s": 1.0172575712203979,
                "Engine Oil Pressure psi ": 34.7
            },
            {
                "Flow Rate ft3/s": 2.445906400680542,
                "Engine Oil Pressure psi ": 49.2
            }
        ]
    },
    {
        "deviceId": "FYYY",
        "timestamp": "2018-01-17T01:18:00Z",
        "series": [
            {
                "Flow Rate ft3/s": 0.58015072345733643,
                "Engine Oil Pressure psi ": 22.2
            }
        ]
    }
]
```

### <a name="time-series-instance"></a>时序实例 
> [!NOTE]
> 时序 ID 为 *deviceId*。

```JSON
{
    "timeSeriesId": [
      "FXXX"
    ],
    "typeId": "17150182-daf3-449d-adaf-69c5a7517546",
    "hierarchyIds": [
      "b888bb7f-06f0-4bfd-95c3-fac6032fa4da"
    ],
    "description": null,
    "instanceFields": {
      "L1": "REVOLT SIMULATOR",
      "L2": "Battery System",
    }
  },
  {
    "timeSeriesId": [
      "FYYY"
    ],
    "typeId": "17150182-daf3-449d-adaf-69c5a7517546",
    "hierarchyIds": [
      "b888bb7f-06f0-4bfd-95c3-fac6032fa4da"
    ],
    "description": null,
    "instanceFields": {
      "L1": "COMMON SIMULATOR",
      "L2": "Battery System",
    }
  },
```

时序见解预览版会在查询时联接一个表（平展后）。 此表包含其他列，例如“类型”。 以下示例演示如何才能[塑造](./time-series-insights-send-events.md#json)遥测数据：

| deviceId  | Type | L1 | L2 | timestamp | series.Flow Rate ft3/s | series.Engine Oil Pressure psi |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| `FXXX` | Default_Type | SIMULATOR | 电池系统 | 2018-01-17T01:17:00Z |   1.0172575712203979 |    34.7 |
| `FXXX` | Default_Type | SIMULATOR |   电池系统 |    2018-01-17T01:17:00Z | 2.445906400680542 |  49.2 |
| `FYYY` | LINE_DATA    COMMON | SIMULATOR |    电池系统 |    2018-01-17T01:18:00Z | 0.58015072345733643 |    22.2 |

在上一示例中，请注意以下几点：

* 静态属性存储在时序见解预览版中，这样可以优化通过网络发送的数据。
* 时序见解预览版数据在查询时通过在实例中定义的 *timeSeriesId* 进行联接。
* 使用了两个嵌套层，这是时序见解预览版支持的最大嵌套数量。 必须避免深层嵌套的数组。
* 由于度量很少，因此将其作为单独的属性在同一对象中发送。 在此示例中，**series.Flow Rate psi**、**series.Engine Oil Pressure psi** 和 **series.Flow Rate ft3/s** 是唯一的列。

>[!IMPORTANT]
> 实例字段不与遥测数据一起存储。 它们与元数据一起存储在**时序模型**中。
> 上表表示查询视图。

## <a name="next-steps"></a>后续步骤

- 若要将这些指导原则付诸实践，请参阅 [Azure 时序见解预览版查询语法](./time-series-insights-query-data-csharp.md)。 可以详细了解时序见解预览版数据访问 REST API 的查询语法。

- 若要了解支持的 JSON 形状，请参与[支持的 JSON 形状](./time-series-insights-send-events.md#json)。
