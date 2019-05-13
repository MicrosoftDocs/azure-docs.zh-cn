---
title: 机器学习在 Azure 数据资源管理器的功能
description: 使用机器学习在 Azure 数据资源管理器中的根本原因分析的群集。
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/29/2019
ms.openlocfilehash: bc72cc21ab525ec82d9ce4b24e80ce82d92a5d21
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233489"
---
# <a name="machine-learning-capability-in-azure-data-explorer"></a>机器学习在 Azure 数据资源管理器的功能

Azure 数据资源管理器的大数据分析平台，用于监视服务运行状况、 QoS 或运行不正常设备使用内置的异常行为[异常情况检测和预测](/azure/data-explorer/anomaly-detection)函数。 一旦检测到异常模式，将执行根本原因分析 (RCA) 以缓解或解决异常情况。

诊断过程非常复杂和耗时较长且按照域专家已完成。 此过程包括提取和加入其他来自不同数据源相同的时间范围，查找图表其他变量的多个维度上值的分布中的更改和其他技术基于域知识和直觉。 由于这些诊断方案都是在 Azure 数据资源管理器中常见的机器学习插件是可用于轻松诊断阶段并缩短 RCA 的持续时间。

Azure 数据资源管理器有三个机器学习插件： [ `autocluster` ](/azure/kusto/query/autoclusterplugin)， [ `basket` ](/azure/kusto/query/basketplugin)，以及[ `diffpatterns` ](/azure/kusto/query/diffpatternsplugin)。 所有插件都实现聚类分析算法。 `autocluster`并`basket`插件群集单个记录集和`diffpatterns`插件群集两个记录集之间的差异。

## <a name="clustering-a-single-record-set"></a>聚类分析单个记录集

一种常见方案包括数据集选择的特定条件如表现出的异常行为、 高温度设备读数、 持续时间长命令和支出用户顶部的时间窗口。 我们想要一种简单且快速的方法来查找数据中的 （段） 的常见模式。 模式是通过多个维度 （分类列） 的记录共享相同的值的数据集的子集。 下面的查询生成，并在 10 分钟的箱中显示一周以上的时序的服务异常：

```kusto
let min_t = toscalar(demo_clustering1 | summarize min(PreciseTimeStamp));  
let max_t = toscalar(demo_clustering1 | summarize max(PreciseTimeStamp));  
demo_clustering1
| make-series num=count() on PreciseTimeStamp from min_t to max_t step 10m
| render timechart with(title="Service exceptions over a week, 10 minutes resolution")
```

![服务异常时间表](media/machine-learning-clustering/service-exceptions-timechart.png)

服务异常计数关联与总体服务流量。 工作日的星期一到星期五的可以清楚地看到的每日模式，与服务中出现增加，异常计数午间，并删除晚上计数中。 过了周末平面低计数是可见的。 可以使用检测到异常峰值[时间时序异常情况检测](/azure/data-explorer/anomaly-detection?#time-series-anomaly-detection)在 Azure 数据资源管理器。

在星期二下午上发生的数据中的第二个峰值。 使用以下查询来进一步诊断此峰值。 使用查询来重绘围绕中更高的分辨率 （在一分钟的箱中的八个小时） 的峰值图表以验证它是否清晰的峰值，并查看其边框。

```kusto
let min_t=datetime(2016-08-23 11:00);
demo_clustering1
| make-series num=count() on PreciseTimeStamp from min_t to min_t+8h step 1m
| render timechart with(title="Zoom on the 2nd spike, 1 minute resolution")
```

![专注于峰值时间表](media/machine-learning-clustering/focus-spike-timechart.png)

我们可以看到从 15:00 到 15:02 的窄两分钟的峰值。 在下面的查询，来计算此两分钟的窗口中的异常：

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| count
```

|Count |
|---------|
|972    |

在下面的查询示例超出 972 20 异常：

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| take 20
```

| PreciseTimeStamp            | 区域 | ScaleUnit | DeploymentId                     | 跟踪点 | ServiceHost                          |
|-----------------------------|--------|-----------|----------------------------------|------------|--------------------------------------|
| 2016-08-23 15:00:08.7302460 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 100005     | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:09.9496584 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 8d257da1-7a1c-44f5-9acd-f9e02ff507fd |
| 2016-08-23 15:00:10.5911748 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 100005     | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:12.2957912 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007007   | f855fcef-ebfe-405d-aaf8-9c5e2e43d862 |
| 2016-08-23 15:00:18.5955357 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 9d390e07-417d-42eb-bebd-793965189a28 |
| 2016-08-23 15:00:20.7444854 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 6e54c1c8-42d3-4e4e-8b79-9bb076ca71f1 |
| 2016-08-23 15:00:23.8694999 | eus2   | su2       | 89e2f62a73bb4efd8f545aeae40d7e51 | 36109      | 19422243-19b9-4d85-9ca6-bc961861d287 |
| 2016-08-23 15:00:26.4271786 | ncus   | su1       | e24ef436e02b4823ac5d5b1465a9401e | 36109      | 3271bae4-1c5b-4f73-98ef-cc117e9be914 |
| 2016-08-23 15:00:27.8958124 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 904498     | 8cf38575-fca9-48ca-bd7c-21196f6d6765 |
| 2016-08-23 15:00:32.9884969 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007007   | d5c7c825-9d46-4ab7-a0c1-8e2ac1d83ddb |
| 2016-08-23 15:00:34.5061623 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 1002110    | 55a71811-5ec4-497a-a058-140fb0d611ad |
| 2016-08-23 15:00:37.4490273 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007006   | f2ee8254-173c-477d-a1de-4902150ea50d |
| 2016-08-23 15:00:41.2431223 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 103200     | 8cf38575-fca9-48ca-bd7c-21196f6d6765 |
| 2016-08-23 15:00:47.2983975 | ncus   | su1       | e24ef436e02b4823ac5d5b1465a9401e | 423690590  | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:50.5932834 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 2a41b552-aa19-4987-8cdd-410a3af016ac |
| 2016-08-23 15:00:50.8259021 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 1002110    | 0d56b8e3-470d-4213-91da-97405f8d005e |
| 2016-08-23 15:00:53.2490731 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 36109      | 55a71811-5ec4-497a-a058-140fb0d611ad |
| 2016-08-23 15:00:57.0000946 | eus2   | su2       | 89e2f62a73bb4efd8f545aeae40d7e51 | 64038      | cb55739e-4afe-46a3-970f-1b49d8ee7564 |
| 2016-08-23 15:00:58.2222707 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007007   | 8215dcf6-2de0-42bd-9c90-181c70486c9c |
| 2016-08-23 15:00:59.9382620 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007006   | 451e3c4c-0808-4566-a64d-84d85cf30978 |

### <a name="use-autocluster-for-single-record-set-clustering"></a>用于设置群集的单个记录 autocluster()

即使有不超过 1000 个例外情况，它仍然难以发现的常见段，是因为每个列中有多个值。 可以使用[ `autocluster()` ](/azure/kusto/query/autoclusterplugin)插件以立即提取常见段的小列表并找到令人感兴趣群集的峰值两分钟内，如下面的查询中所示：

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| evaluate autocluster()
```

| 段 ID | Count | 百分比 | 区域 | ScaleUnit | DeploymentId | ServiceHost |
|-----------|-------|------------------|--------|-----------|----------------------------------|--------------------------------------|
| 0 | 639 | 65.7407407407407 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 第 | 94 | 9.67078189300411 | scus | su5 | 9dbd1b161d5b4779a73cf19a7836ebd6 |  |
| 2 | 82 | 8.43621399176955 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |
| 3 | 68 | 6.99588477366255 | scus | su3 | 90d3d2fc7ecc430c9621ece335651a01 |  |
| 4 | 55 | 5.65843621399177 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |

最主要的段包含 65.74%的总异常记录并共享四个维度，可以看到上述结果。 下一段不常见，包含仅 9.67%的记录以及共享三个维度。 其他段是甚至不太常见。 

Autocluster 使用专有算法来挖掘多个维度和提取感兴趣的段。 "有趣"意味着每个段的记录集和功能集的重要方面。 段也已分离，这意味着每个与其他大不相同。 一个或多个这些段可能相关的 RCA 进程。 为了尽量减少段检查和评估，autocluster 提取小部分列表。

### <a name="use-basket-for-single-record-set-clustering"></a>用于设置群集的单个记录 basket()

此外可以使用[ `basket()` ](/azure/kusto/query/basketplugin)插件，如下面的查询中所示：

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| evaluate basket()
```

| 段 ID | Count | 百分比 | 区域 | ScaleUnit | DeploymentId | 跟踪点 | ServiceHost |
|-----------|-------|------------------|--------|-----------|----------------------------------|------------|--------------------------------------|
| 0 | 639 | 65.7407407407407 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 1 | 642 | 66.0493827160494 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  |  |
| 2 | 324 | 33.3333333333333 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | 0 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 3 | 315 | 32.4074074074074 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | 16108 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 4 | 328 | 33.7448559670782 |  |  |  | 0 |  |
| 5 | 94 | 9.67078189300411 | scus | su5 | 9dbd1b161d5b4779a73cf19a7836ebd6 |  |  |
| 6 | 82 | 8.43621399176955 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |  |
| 7 | 68 | 6.99588477366255 | scus | su3 | 90d3d2fc7ecc430c9621ece335651a01 |  |  |
| 8 | 167 | 17.1810699588477 | scus |  |  |  |  |
| 9 | 55 | 5.65843621399177 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |  |
| 10 | 92 | 9.46502057613169 |  |  |  | 10007007 |  |
| 11 | 90 | 9.25925925925926 |  |  |  | 10007006 |  |
| 12 | 57 | 5.8641975308642 |  |  |  |  | 00000000-0000-0000-0000-000000000000 |

购物篮的项设置挖掘并提取的记录集的覆盖率高于阈值 （默认为 5%) 的所有段实现 Apriori 算法。 您可以看到多个段提取具有相似的 （适用于示例中，段 0、 1 或 2，3）。

有两个插件，功能强大且易于使用，但其重要限制是因为，在群集的单个记录集 （不带标签） 非监督式的方式。 因此，是不清楚提取的模式描述所选的记录集 （异常记录） 或全局记录集。

## <a name="clustering-the-difference-between-two-records-sets"></a>聚类分析两个记录集之间的差异

[ `diffpatterns()` ](/azure/kusto/query/diffpatternsplugin)插件从而克服了的限制`autocluster`和`basket`。 `Diffpatterns` 采用两个记录集，并提取不同它们之间的主要段。 一组通常包含设置正在调查的异常记录 (通过一个分析`autocluster`和`basket`)。 另一组包含引用记录集 （基准）。 

在下面的查询中，我们使用`diffpatterns`峰值的两分钟，不同于在基准范围内的群集内查找感兴趣的群集。 我们将定义为前 15:00 的八分钟基线窗口 （峰值启动时）。 我们还需要通过指定与基线或异常组是否属于特定记录的二进制列 (AB) 来扩展。 `Diffpatterns` 实现一种监督式的学习算法，其中两个类标签生成的异常与基线标志 (AB)。

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
let min_baseline_t=datetime(2016-08-23 14:50);
let max_baseline_t=datetime(2016-08-23 14:58); // Leave a gap between the baseline and the spike to avoid the transition zone.
let splitime=(max_baseline_t+min_peak_t)/2.0;
demo_clustering1
| where (PreciseTimeStamp between(min_baseline_t..max_baseline_t)) or
        (PreciseTimeStamp between(min_peak_t..max_peak_t))
| extend AB=iff(PreciseTimeStamp > splitime, 'Anomaly', 'Baseline')
| evaluate diffpatterns(AB, 'Anomaly', 'Baseline')
```

| 段 ID | 计数 A | 计数 B | 百分比 A | 百分比 B | PercentDiffAB | 区域 | ScaleUnit | DeploymentId | 跟踪点 |
|-----------|--------|--------|----------|----------|---------------|--------|-----------|----------------------------------|------------|
| 0 | 639 | 21 | 65.74 | 1.7 | 64.04 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  |
| 1 | 167 | 544 | 17.18 | 44.16 | 26.97 | scus |  |  |  |
| 2 | 92 | 356 | 9.47 | 28.9 | 19.43 |  |  |  | 10007007 |
| 3 | 90 | 336 | 9.26 | 27.27 | 18.01 |  |  |  | 10007006 |
| 4 | 82 | 318 | 8.44 | 25.81 | 17.38 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |
| 5 | 55 | 252 | 5.66 | 20.45 | 14.8 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |
| 6 | 57 | 204 | 5.86 | 16.56 | 10.69 |  |  |  |  |

最主要的段是被提取的同一段`autocluster`，它的覆盖两分钟异常窗口上也是 65.74%。 但其包括八分钟基线窗口上的方面是只有 1.7%。 不同之处是 64.04%。 这种差异看起来与相关联的异常峰值。 可以通过将原始图表拆分为属于此有问题的片段与其他段下面的查询中所示的记录来验证这种假设：

```kusto
let min_t = toscalar(demo_clustering1 | summarize min(PreciseTimeStamp));  
let max_t = toscalar(demo_clustering1 | summarize max(PreciseTimeStamp));  
demo_clustering1
| extend seg = iff(Region == "eau" and ScaleUnit == "su7" and DeploymentId == "b5d1d4df547d4a04ac15885617edba57"
and ServiceHost == "e7f60c5d-4944-42b3-922a-92e98a8e7dec", "Problem", "Normal")
| make-series num=count() on PreciseTimeStamp from min_t to max_t step 10m by seg
| render timechart
```

![验证 diffpattern 段时间表](media/machine-learning-clustering/validating-diffpattern-timechart.png)

此图表可以让我们查看在星期二下午上的峰值因使用发现的此特定段中的异常`diffpatterns`插件。

## <a name="summary"></a>摘要

Azure 数据资源管理器中机器学习插件是于很多情况。 `autocluster`和`basket`实现非监督式的学习算法，并且易于使用。 `Diffpatterns` 实现监督式学习算法和尽管比较复杂，它是更强大的 RCA 提取差异化段中。

和近乎实时的监视服务的自动临时方案中，以交互方式使用这些插件。 在 Azure 数据资源管理器，时序异常检测后跟一个诊断过程，它高度优化，可满足必要的性能标准。
