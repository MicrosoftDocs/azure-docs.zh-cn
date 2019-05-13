---
title: Azure 时序见解预览版用例 | Microsoft Docs
description: 了解 Azure 时序见解预览版用例。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: 27c79155a4630f49faf49f30b2d46bf1e0dffd90
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508310"
---
# <a name="azure-time-series-insights-preview-use-cases"></a>Azure 时序见解预览版用例

本文汇总了 Azure 时间系列 Insights 预览版的几个常见用例。 这篇文章中的建议作为起点来开发应用程序和使用时序见解的解决方案。

具体而言，本文回答了以下问题：

* 时序见解的常见用例有哪些？
* 使用的时序见解的优点是什么[进行数据探索和可视化的异常情况检测](#data-exploration-and-visual-anomaly-detection)？
* 使用的时序见解的优点是什么[操作分析和流程效率](#operational-analysis-and-driving-process-efficiency)？
* 使用的时序见解的优点是什么[高级分析](#advanced-analytics)？

其中概述使用方案如下所述。

## <a name="introduction"></a>简介

Azure 时序见解是一种端到端的平台即服务产品。 用于收集、 处理、 存储、 分析和查询高度符合，时间系列优化 IoT 规模的数据。 时序见解是即席数据浏览和运营分析的理想选择。 时序见解是唯一可扩展且自定义的服务产品/服务，满足行业 IoT 部署的广泛需求。

## <a name="data-exploration-and-visual-anomaly-detection"></a>数据浏览和直观异常检测

即时浏览和分析数十亿事件，确定其中的异常，发现数据的隐藏趋势。 时序见解可为 IoT 和 DevOps 分析工作负荷提供近实时的性能。

[![数据资源管理器](media/v2-update-use-cases/data-explorer.svg)](media/v2-update-use-cases/data-explorer.svg#lightbox)

大多数客户都认为，获取见解的效率是时序见解最强大的资产之一。 时序见解无需前期数据准备。 时序见解可在几分钟内快速将你连接到 Azure IoT 中心或 Azure 事件中心中的数十亿个事件。 连接后，可直观显示和分析数十亿事件，发现异常，发现数据的隐藏趋势。

时序见解直观且易于使用。 连一行代码也无需编写即可与数据交互。 此外，无需学习新语言。 时序见解为熟悉 SQL 的高级用户提供基于文本的精细查询。 为初学者提供“选择+单击”式探索模式。

客户可以利用这种高效率，快速诊断与资产相关的问题。 时序见解可以执行 DevOps 来找出 IoT 解决方案中 bug 的根本原因。 它们还可以针对数据科学方面的问题，标识出要调查的区域。  

与时序见解中存储的数据进行交互主要有三种方式：

- 第一个也是最简单的方法是使用时序见解预览版资源管理器。 可以使用它在一个位置快速显示所有 IoT 数据。 它提供热度地图等工具，可帮助发现数据中的异常情况。 它还提供透视视图。 使用它可以在一个仪表板中比较一个或多个时序见解环境中的多达四个视图。 通过仪表板，可以查看所有位置的时序数据。 了解[时序见解预览版资源管理器](./time-series-insights-update-explorer.md)的详细信息。 要规划时序见解环境，请阅读[时序见解规划](./time-series-insights-update-plan.md)。

- 第二种方法是使用 JavaScript SDK 在 Web 应用程序中快速嵌入强大的图表和图形。 只需几行代码，即可编写功能强大的查询。 使用它们填充折线图、饼图、条形图、热度地图和数据网格等。 通过使用 SDK，所有这些元素都是现成可用的。 SDK 还提取时序见解查询 API。 可以使用它们来创建类似 SQL 的谓词，用于查询要在仪表板上显示的数据。 对于混合表示层解决方案，时序见解提供参数化 URL。 它们提供与时序见解预览版资源管理器之间的无缝连接点，可帮助深入分析数据。

    * 读取[时间系列 Insights JS 客户端库](tutorial-explore-js-client-lib.md)并[时序见解客户端](https://github.com/Microsoft/tsiclient)文档，了解有关 JavaScript SDK 的详细信息。

    * 了解有关通过查看共享 Url 和新的用户界面的详细信息[资源管理器预览版中的将数据可视化](time-series-insights-update-explorer.md)。

- 第三种方法是使用功能强大的 API 查询存储在时序见解中的数据。 时序见解有临时运算符，如`from`， `to`， `first`，和`last`。 它具有聚合和转换等`average`， `min`， `max`， `split by`， `order by`，并`DateHistogram`。 它还具有如筛选运算符`has`， `in`， `and`， `or`， `greater than`，并`REGEX`。 所有这些运算符使下游应用程序能够快速找到数据中的相关趋势和模式。 可使用它们来填充自生成的可视化效果以发现异常。

## <a name="operational-analysis-and-driving-process-efficiency"></a>操作性分析和提高处理效率

使用时序见解可大规模监视设备的运行状况、使用情况和性能。 时序见解提供了一种衡量操作效率的简便方法。 时序见解有助于管理多种不可预测的 IoT 工作负荷，且不影响引入或查询性能。

[![概述](media/v2-update-use-cases/overview.svg)](media/v2-update-use-cases/overview.svg#lightbox)

如果与正确的技术或解决方案相结合，来自操作过程的数据的流式传输和连续处理可成功地转换任何业务。 这些解决方案通常是多个系统的组合。 它们可以探索和分析不断变化的数据，特别是在 IoT 领域，并且采用相同的模式。

这些模式通常始于支持 IoT 的平台，这类平台从各种语言环境的设备和传感器中引入数十亿个事件。 这些系统可以处理和分析流数据，获取实时见解和操作。 通常将数据存档到热存储和冷存储，以实现近实时和批量分析。

收集的数据经过一系列处理，以便对数据进行净化并使其适合特定上下文，从而使其可用于下游查询和分析方案。 Azure 提供丰富的服务，可应用于资产维护和制造等 IoT 方案。 这些服务包括时序见解、IoT 中心、事件中心、Azure 流分析、Azure Functions、Azure 逻辑应用、Azure Databricks、Azure 机器学习和 Power BI。

解决方案体系结构可以通过以下方式实现：

- 通过 IoT 中心或事件中心来引入数据，以获得最佳的安全性、吞吐量和延迟。
- 执行数据处理和计算。 通过流分析、逻辑应用和 Azure Functions 等服务传送引入的数据。 使用的服务取决于特定的数据处理需求。
- 来自处理管道的计算信号被推送到时序见解进行存储和分析。

时序见解提供近实时数据浏览和基于资产的历史数据见解。 根据业务需求，可通过将时序见解连接到 Azure HDInsight，基于存储在时序见解中的数据来运行 MapReduce 和 Hive 作业。 可通过时序见解公共图面查询 API 向 Power BI 和其他客户应用程序提供存储在时序见解中的数据。 此数据可用于深层业务和操作智能方案。

## <a name="advanced-analytics"></a>高级分析

与机器学习和 Azure Databricks 等高级分析服务集成。 时序见解从数百万台设备中引入原始数据。 它添加了可由 Azure 分析服务套件无缝使用的上下文数据。

[![Analytics](media/v2-update-use-cases/advanced-analytics.svg)](media/v2-update-use-cases/advanced-analytics.svg#lightbox)

高级分析和机器学习会使用和处理大量数据。 该数据用于制定数据驱动的决策并执行预测分析。 在 IoT 用例中，高级分析算法可以从数百万台设备中收集数据。 这些设备每秒内会多次传输数据。 从 IoT 设备收集的数据是原始数据。 它缺少上下文信息，例如设备的位置和传感器读数的单位。 因此，原始数据很难直接用于高级分析。

时序见解以两种简单且经济高效的方式在 IoT 数据与高级分析之间搭建了桥梁：

- 首先，时序见解通过使用 IoT 中心从数百万台设备收集原始遥测数据。 它通过上下文信息来丰富数据，并将数据转换为 parquet 格式。 此格式可以轻松与其他高级分析服务集成，例如机器学习、Azure Databricks 和第三方应用程序。

    时序见解可以作为整个组织中所有数据的真实来源。 它创建了一个供下游分析工作负荷使用的中央存储库。 由于时序见解是一种近实时存储服务，因此高级分析模型可以利用传入的 IoT 遥测数据不断学习。 这样一来，模型可以更准确地进行预测。

- 其次，可将机器学习和预测模型的输出馈送到时序见解，以直观显示和存储其结果。 此过程可帮助组织优化和调整其模型。 使用时序见解，可轻松直观显示与训练模型输出相同平面上的流式遥测数据。 通过这种方式，它可以帮助数据科学团队发现异常并识别模式。  

## <a name="next-steps"></a>后续步骤

- 了解[时序见解预览版资源管理器](./time-series-insights-update-explorer.md)的详细信息。

- 要规划环境，请阅读[时序见解预览版规划](./time-series-insights-update-plan.md)。

- 阅读[时序见解客户端](https://github.com/Microsoft/tsiclient)文档。
