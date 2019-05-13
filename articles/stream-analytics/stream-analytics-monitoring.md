---
title: 了解 Azure 流分析中的作业监视
description: 本文介绍如何在 Azure 门户中监视 Azure 流分析作业。
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 43dd8be998e0f8f3b5a2b783c6a01d5b5ef3da12
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506928"
---
# <a name="understand-stream-analytics-job-monitoring-and-how-to-monitor-queries"></a>了解流分析作业监视以及如何监视查询

## <a name="introduction-the-monitor-page"></a>简介：监视页
Azure 门户提供了可用于监视和排查查询和作业性能问题的关键性能指标。 要查看这些指标，请浏览到感兴趣的想要查看其指标的流分析作业，并查看“概览”页面上的“监视”部分。  

![流分析作业监视链接](./media/stream-analytics-monitoring/02-stream-analytics-monitoring-block.png)

此窗口如下所示：

![流分析作业监视仪表板](./media/stream-analytics-monitoring/01-stream-analytics-monitoring.png)  

## <a name="metrics-available-for-stream-analytics"></a>可用于流分析的指标
| 指标                 | 定义                               |
| ---------------------- | ---------------------------------------- |
| 囤积的输入事件       | 积压的输入事件的数量。 此指标的非零值意味着作业无法跟上传入事件的数量。 如果此值缓慢增长或始终为非零，则应横向扩展作业。 可以访问[了解和调整流单元](stream-analytics-streaming-unit-consumption.md)了解详细信息。 |
| 数据转换错误 | 无法转换为预期输出架构的输出事件的数量。 可以将错误策略更改为“删除”，以删除遇到此情况的事件。 |
| 早期输入事件       | 应用程序时间戳早于其到达时间超过 5 分钟的事件。 |
| 失败的函数请求 | 失败的 Azure 机器学习函数（如果存在）调用数。 |
| 函数事件数        | 发送到 Azure 机器学习函数（如果存在）的事件数。 |
| 函数请求      | 对 Azure 机器学习函数（如果存在）的调用数。 |
| 输入反序列化错误       | 不可反序列化的输入事件数。  |
| 输入事件字节      | 流分析作业收到的数据量（以字节为单位）。 这可以用于验证正在发送到输入源的事件。 |
| 输入事件           | 从输入事件反序列化的记录数。 此计数不包括导致反序列化错误的传入事件。 |
| 接收到的输入源       | 作业收到的消息数。 事件中心的消息是单个 EventData。 对于 Blob，消息是单个 blob。 请注意，在反序列化之前被计输入源。 如果反序列化错误，则输入的源可以是大于输入事件。 否则，它可以是小于或等于输入事件，因为每条消息可以包含多个事件。 |
| 延迟输入事件      | 到达时间晚于已配置的延迟到达容错时段的事件。 详细了解 [Azure 流分析事件顺序注意事项](stream-analytics-out-of-order-and-late-events.md)。 |
| 无序事件数    | 收到的无序事件的数目，系统根据事件排序策略来删除这些事件，或者为其提供一个经过调整的时间戳。 这可能会受“无序容错时段”设置的影响。 |
| 输出事件          | 流分析作业发送到输出目标的数据量，以事件数来衡量。 |
| 运行时错误         | 与查询处理相关的错误总数（不包括引入事件或输出结果时发现的错误） |
| SU % 利用率       | 从作业的“比例”选项卡向一个作业分配的流单元利用率。 如果此指标达到 80% 或以上，则很可能会出现事件处理延迟或停止处理的情况。 |
| 水印延迟       | 作业中所有输出的所有分区之间的最大水印延迟。 |

可以使用这些指标来[监视流分析作业的性能](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-set-up-alerts#scenarios-to-monitor)。 

## <a name="customizing-monitoring-in-the-azure-portal"></a>在 Azure 门户中自定义监视
可以在“编辑图表”设置中调整图表类型、显示的指标和时间范围。 有关详细信息，请参阅[如何自定义监视](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md)。

  ![流分析查询监视器时间图](./media/stream-analytics-monitoring/08-stream-analytics-monitoring.png)  


## <a name="latest-output"></a>最新的输出
用于监视作业的另一个有趣的数据点是最后一个输出的时间，如“概述”页所示。
此时间是作业的最新输出的应用程序时间（即使用事件数据的时间戳的时间）。

## <a name="get-help"></a>获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>后续步骤
* [Azure 流分析简介](stream-analytics-introduction.md)
* [Azure 流分析入门](stream-analytics-real-time-fraud-detection.md)
* [缩放 Azure 流分析作业](stream-analytics-scale-jobs.md)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

