---
title: 通过参数化 URL 共享 Azure 时序见解自定义视图 | Microsoft Docs
description: 本文介绍如何在 Azure 时序见解中生成参数化 URL，以便轻松共享自定义视图。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.topic: conceptual
ms.workload: big-data
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: e70eb7ae73e88b37e649d519d0d0428554dd4ab3
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467516"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>通过参数化 URL 共享自定义视图

若要共享时间时序见解资源管理器中的自定义视图，可以以编程方式创建自定义视图的参数化的 URL。

时间时序见解资源管理器支持 URL 查询参数来指定视图中直接从该 URL 的体验。 例如，只使用 URL 就可以指定目标环境、搜索谓词以及所需的时间跨度。 当用户单击自定义 URL 时，界面会提供一个链接，直接指向时序见解门户中的相应资产。 数据访问策略适用。

> [!TIP]
> * 查看可用[时序见解演示](https://insights.timeseries.azure.com/samples)。
> * 阅读随附[时间时序见解资源管理器](./time-series-insights-explorer.md)文档。

## <a name="environment-id"></a>环境 ID

`environmentId=<guid>` 参数指定目标环境 ID。 它是一个组件的数据访问 FQDN，并可以在 Azure 门户中的环境概览的右上角找到它。 它是 `env.timeseries.azure.com` 之前的所有内容。

例如，`?environmentId=10000000-0000-0000-0000-100000000108` 是环境 ID 参数。

## <a name="time"></a>Time

可以通过参数化 URL 来指定绝对或相对时间值。

### <a name="absolute-time-values"></a>绝对时间值

对于绝对时间值，请使用 `from=<integer>` 和 `to=<integer>` 参数。

* `from=<integer>` 是以 JavaScript 毫秒为单位的一个值，代表搜索时间跨度的开始时间。
* `to=<integer>` 是以 JavaScript 毫秒为单位的一个值，代表搜索时间跨度的结束时间。

若要确定某个日期对应的 JavaScript 毫秒，请参阅[纪元和 Unix 时间戳转换器](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html)。

### <a name="relative-time-values"></a>相对时间值

对于相对时间值，请使用 `relativeMillis=<value>`，其中 value 为 JavaScript 毫秒，来自后端的最新数据。

例如，`&relativeMillis=3600000` 显示最新的 60 分钟的数据。

接受的值对应到时序见解资源管理器**快速时间**菜单中，并且包括：

* `1800000` （过去 30 分钟）
* `3600000` （过去 60 分钟）
* `10800000` （过去 3 小时）
* `21600000` （过去 6 小时）
* `43200000` （过去 12 小时）
* `86400000` （过去 24 小时）
* `604800000` （过去 7 天）
* `2592000000` （过去 30 小时）

### <a name="optional-parameters"></a>可选参数

`timeSeriesDefinitions=<collection of term objects>`参数指定时序见解视图的术语：

| 参数 | URL 项 | 描述 |
| --- | --- | --- |
| **name** | `\<string>` | 术语的名称。 |
| **splitBy** | `\<string>` | 要按其进行拆分的列名。 |
| **measureName** | `\<string>` | 度量值的列名。 |
| **predicate** | `\<string>` | 用于服务器端筛选的 where 子句。 |
| **useSum** | `true` | 一个可选参数，指定使用度量值的总和。 </br>  请注意，如果`Events`是所选度量值，默认情况下选择计数。  </br>  如果`Events`是未选中，默认情况下选择平均值。 |

* `multiChartStack=<true/false>`键-值对使图表中的堆叠。
* `multiChartSameScale=<true/false>`键-值对不同的条件在可选参数启用同一 y 轴比例。  
* `timeBucketUnit=<Unit>&timeBucketSize=<integer>`使您可以调整的时间间隔滑块，以提供更精细或流畅些，更聚合图表的视图。  
* `timezoneOffset=<integer>`参数使你可以在中查看为 UTC 的偏移量的图表的时区设置。

| 编号 | 描述 |
| --- | --- |
| `multiChartStack=false` | `true` 默认情况下启用因此传递`false`堆叠。 |
| `multiChartStack=false&multiChartSameScale=true` | 必须启用堆叠才能在不同的条件中使用同一 Y 轴比例。  它具有`false`默认情况下，因此传递 true 可启用此功能。 |
| `timeBucketUnit=<Unit>&timeBucketSize=<integer>` | 单位为天、小时、分钟、秒、毫秒。  请始终将单位大写。 </br> 为 timeBucketSize 传递所需的整数即可定义单位数。  注意，平滑度最多可以调整为 7 天。  |
| `timezoneOffset=-<integer>` | 整数的单位始终为毫秒。 </br> 注意，此功能与我们在 TSI 浏览器中启用的功能稍有差别，该浏览器允许选择本地浏览器时间或 UTC。 |

### <a name="examples"></a>示例

若要以 URL 参数的 TSI 环境添加时序定义，追加：

```plaintext
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

使用示例时序定义用于：

* 环境 ID
* 过去 60 分钟的数据
* 术语 （F1PressureID、 F2TempStation 和 F3VibrationPL） 组成的可选参数

您可以构造一个视图的以下参数化的 URL:

```plaintext
https://insights.timeseries.azure.com/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

> [!TIP]
> 请参阅实时的资源管理器[使用 URL](https://insights.timeseries.azure.com/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}])。

上述 URL 描述，并将生成 TSI 资源管理器视图：

[![时间时序见解资源管理器术语](media/parameterized-url/url1.png)](media/parameterized-url/url1.png#lightbox)

完整视图 （包括图表）：

[![图表视图](media/parameterized-url/url2.png)](media/parameterized-url/url2.png#lightbox)

## <a name="next-steps"></a>后续步骤

* 了解如何[查询的数据使用C# ](time-series-insights-query-data-csharp.md)。

* 了解如何[时间时序见解资源管理器](./time-series-insights-explorer.md)。
