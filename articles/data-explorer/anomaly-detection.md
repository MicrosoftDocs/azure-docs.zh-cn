---
title: 时序异常检测和预测在 Azure 数据资源管理器
description: 了解如何分析时序数据异常情况检测和预测使用 Azure 数据资源管理器。
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/24/2019
ms.openlocfilehash: f40350129a12c7865051bcae80b74b6f9c069179
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233530"
---
# <a name="anomaly-detection-and-forecasting-in-azure-data-explorer"></a>异常情况检测和预测在 Azure 数据资源管理器

Azure 数据资源管理器执行正在收集的遥测数据从云服务或 IoT 设备。 此数据进行分析的各种见解，例如监视服务运行状况、 物理生产过程中，使用情况趋势和负载预测。 所选度量值来查找相对于典型的正常基线模式将度量值的方差模式的时间系列上执行分析。 Azure 数据资源管理器包含用于创建、 操作和多个时序分析的本机支持。 它可以创建和分析数千个时序，以秒为单位，启用接近实时监视解决方案和工作流。

本文详细介绍 Azure 数据资源管理器时时序异常情况检测和预测功能。 适用的时间系列函数基于可靠的已知分解模型，其中每个原始时序分解成季节性、 趋势，以及剩余组件。 异常检测到的离群值上的残留的组件，而预测通过推算出季节性和趋势组件。 Azure 数据资源管理器实现显著增强了通过自动季节性检测、 可靠的离群值分析和矢量化的实现处理数千个时序的以秒为单位的基本分解模型。

## <a name="prerequisites"></a>必备组件

读取[时间在 Azure 数据资源管理器中的时序分析](/azure/data-explorer/time-series-analysis)时间系列功能的概述。

## <a name="time-series-decomposition-model"></a>时序分解模型

时序预测和异常情况检测的 azure 数据资源管理器本机实现使用的已知分解模型。 此模型应用于度量值需要定期和趋势的行为，如服务流量、 组件的检测信号和 IoT 定期度量值来预测将来的指标值，并检测异常的清单的时间序列。 此回归过程假设是，非以前已知季节性和趋势行为，随机分布序列的时间。 然后可以预测季节性从将来指标值和趋势组件，统称为名为基线，并忽略剩余部分。 此外可以检测异常使用只有的剩余部分的离群值的值。
若要创建一个分解模型，使用函数[ `series_decompose()` ](/azure/kusto/query/series-decomposefunction)。 `series_decompose()`函数采用一系列时间序列，并自动分解每个到其季节性时序、 趋势、 剩余和基线组件。 

例如，可以通过使用以下查询分解为内部的 web 服务的流量：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (baseline, seasonal, trend, residual) = series_decompose(num, -1, 'linefit')  //  decomposition of a set of time series to seasonal, trend, residual, and baseline (seasonal+trend)
| render timechart with(title='Web app. traffic of a month, decomposition', ysplit=panels)
```

![时序分解](media/anomaly-detection/series-decompose-timechart.png)

* 标记为原始时序**num** （红色） 中。 
* 过程首先通过使用函数会自动检测季节性[ `series_periods_detect()` ](/azure/kusto/query/series-periods-detectfunction) ，并提取**季节性**模式 （以紫色）。
* 从原始时序中减去的季节性模式和使用该函数运行线性回归[ `series_fit_line()` ](/azure/kusto/query/series-fit-linefunction)若要查找**趋势**组件 （以浅蓝色表示）。
* 该函数中减去趋势并的其余部分是**残留**组件 （绿色）。
* 最后，该函数将添加季节性和趋势组件以生成**基线**（以蓝色表示）。

## <a name="time-series-anomaly-detection"></a>时序异常检测

该函数[ `series_decompose_anomalies()` ](/azure/kusto/query/series-decompose-anomaliesfunction)发现异常的点包含在一系列时间序列。 此函数将调用`series_decompose()`生成分解模型，然后运行[ `series_outliers()` ](/azure/kusto/query/series-outliersfunction)残差的组件。 `series_outliers()` 计算的剩余部分使用 Tukey 隔离测试每个点的异常分数。 异常情况分数 1.5 高于或低于-1.5 指示轻微的异常上升或分别拒绝。 异常情况分数 3.0 高于或低于-3.0 表示强的异常情况。 

下面的查询，可检测异常的内部 web 服务通信：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (anomalies, score, baseline) = series_decompose_anomalies(num, 1.5, -1, 'linefit')
| render anomalychart with(anomalycolumns=anomalies, title='Web app. traffic of a month, anomalies') //use "| render anomalychart with anomalycolumns=anomalies" to render the anomalies as bold points on the series charts.
```

![时序异常检测](media/anomaly-detection/series-anomaly-detection.png)

* 原始时序 （红色） 中。 
* 基线 (季节性 + 趋势) 组件 （以蓝色表示）。
* 基于原始时序异常点 （以紫色）。 异常点显著偏离预期的基准值。

## <a name="time-series-forecasting"></a>时间序列预测

该函数[ `series_decompose_forecast()` ](/azure/kusto/query/series-decompose-forecastfunction)预测将来值的一系列时间序列。 此函数将调用`series_decompose()`生成分解模型，然后为每个时序，由此在未来进行推算基线组件。

下面的查询可以预测下一步一周的 web 服务通信：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week by Time Series Decmposition')
```

![时间序列预测](media/anomaly-detection/series-forecasting.png)

* 原始的度量值 （以红色显示）。 未来的值缺失，并设置为 0，默认情况下。
* 进行推断 （以蓝色表示） 的基准组件来预测下一步一周的值。

## <a name="scalability"></a>可伸缩性

Azure 数据资源管理器查询语言语法，以处理多个时序的单一调用。 其唯一的优化的实现允许快的性能，这很关键的有效的异常情况检测和预测时监视数千个近实时方案中的计数器。

以下查询同时显示这三个时间序列的处理：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid
| extend offset=case(sid=='TS3', 4000000, sid=='TS2', 2000000, 0)   //  add artificial offset for easy visualization of multiple time series
| extend num=series_add(num, offset)
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week for 3 time series')
```

![时间序列可伸缩性](media/anomaly-detection/series-scalability.png)

## <a name="summary"></a>摘要

本文档详细介绍用于时序异常检测和预测的本机 Azure 数据资源管理器函数。 每个原始时序分解为检测异常和/或预测季节性、 趋势和剩余组件。 这些功能可用于接近实时的监视方案，例如故障检测、 预见性维护和需求和负载预测。

## <a name="next-steps"></a>后续步骤

了解如何[机器学习功能](/azure/data-explorer/machine-learning-clustering)在 Azure 数据资源管理器。
