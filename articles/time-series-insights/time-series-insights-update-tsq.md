---
title: Azure 时序见解预览版数据查询 | Microsoft Docs
description: Azure 时序见解预览版数据查询。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: seodec18
ms.openlocfilehash: d28cb35b1e8e4eaa86418da632c6fcabb636e643
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205021"
---
# <a name="data-querying"></a>数据查询

Azure 时序见解预览版允许通过公共 Surface API 对存储在环境中的事件和元数据进行数据查询。 这些 API 也用于[时序见解预览版资源管理器](./time-series-insights-update-explorer.md)。

时序见解中提供三个主要 API 类别：

* **环境 API**：允许查询时序见解环境本身。 查询示例包括调用方有权访问的环境和环境元数据的列表。

* **时序模型-查询 (TSM-Q) API**：允许对存储在时序模型的环境部分中的元数据执行创建、读取、更新和删除操作。 示例包括实例、类型和层次结构。

* **时序查询 (TSQ) API**：允许检索从源提供程序记录的事件数据。 这些 API 可通过执行操作，对时序数据进行转换、合并和执行计算。

第四个类别是功能强大的[时序表达式 (TSX) 语言](https://docs.microsoft.com/rest/api/time-series-insights/preview-tsx)。 它使用时序模型来实现高级计算的组合。

## <a name="azure-time-series-insights-preview-core-apis"></a>Azure 时序见解预览版核心 API

支持以下核心 API。

[![时间序列查询概述](media/v2-update-tsq/tsq.png)](media/v2-update-tsq/tsq.png#lightbox)

## <a name="environment-apis"></a>环境 API

提供以下环境 API：

* [获取环境 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-env#get-environments-api)：返回调用方有权访问的环境的列表。
* [获取环境可用性 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-env#get-environment-availability-api)：返回事件时间戳 `$ts` 中事件计数的分布。 此 API 通过返回事件计数（如果存在）来帮助确定时间戳中是否有任何事件。
* [获取事件架构 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-env#get-event-schema-api)：返回给定搜索范围的事件架构元数据。 此 API 可帮助检索给定搜索范围的架构中可用的所有元数据和属性。

## <a name="time-series-model-query-tsm-q-apis"></a>时序模型-查询 (TSM-Q) API

提供以下时序模型-查询 API：

* [模型设置 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#model-settings-api)：允许获取环境的默认类型和模型名称并对其进行修补。
* [类型 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#types-api)：允许对时序类型及其关联变量执行 CRUD。
* [层次结构 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#hierarchies-api)：允许对时序层次结构及其关联的字段路径执行 CRUD。
* [实例 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#instances-api)：允许对时序实例及其关联的实例字段执行 CRUD。

## <a name="time-series-query-tsq-apis"></a>时序查询 (TSQ) API

提供以下时序查询 API：

* [获取事件 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-query#get-events-api)：对于在时序见解中记录的来自源提供程序的事件，允许从其中查询和检索时序见解数据。

* [获取时序 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-query#get-series-api)：允许使用网络上记录的数据从捕获的事件中查询和检索时序见解数据。 返回的值基于模型中定义的变量或以内联方式提供的变量。

    >[!NOTE]
    > 即使在模型中指定或以内联方式提供 Aggregation 子句，也会忽略该子句。

  获取时序 API 为每个时间间隔的每个变量返回一个时序值。 时序值是时序见解对查询的输出 JSON 所用的格式。 返回的值基于所提供的时序 ID 和变量集。

* [聚合时序 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-query#aggregate-series-api)：允许通过对记录的数据进行采样和聚合，从捕获的事件中查询和检索时序见解数据。

  聚合时序 API 为每个时间间隔的每个变量返回一个时序值。 这些值基于所提供的时序 ID 和变量集。 聚合时序 API 使用存储在时序模型中或以内联方式提供的变量对数据进行聚合或采样，以实现缩减操作。

  支持的聚合类型：`Min`、`Max`、`Sum`、`Count`、`Average`

## <a name="next-steps"></a>后续步骤

- 详细了解如何[存储和入口](./time-series-insights-update-storage-ingress.md)Azure 时间系列 Insights 预览版中。

- 读取时间系列 Insights 预览版[数据建模](./time-series-insights-update-tsm.md)一文。

- 发现[选择时间序列 ID 时的最佳做法](./time-series-insights-update-how-to-id.md)。
