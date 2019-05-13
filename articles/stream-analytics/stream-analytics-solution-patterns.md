---
title: Azure Stream Analytics 解决方案模式
description: 了解 Azure Stream Analytics 的解决方案模式。
author: zhongc
ms.author: zhongc
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 4f4f22628d2c2a6beb7974aa9b776a2148a3fee0
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65238057"
---
# <a name="azure-stream-analytics-solution-patterns"></a>Azure Stream Analytics 解决方案模式

类似于在 Azure 中的许多其他服务，Stream Analytics 最适合用于与其他服务创建较大的端到端解决方案。 本文介绍 Azure Stream Analytics 的简单解决方案和各种体系结构模式。 您可以基于这些模式来开发更复杂的解决方案。 在本文中所述的模式可在各种方案。 特定于方案的模式的示例位于[Azure 解决方案体系结构](https://azure.microsoft.com/solutions/architecture/?product=stream-analytics)。

## <a name="create-a-stream-analytics-job-with-a-real-time-dashboard"></a>使用实时仪表板创建 Stream Analytics 作业

使用 Azure Stream Analytics 方便使用，你可以快速构建实时仪表板和警报。 一个简单的解决方案引入事件从事件中心或 IoT 中心，并[馈送使用的流式处理数据集的 Power BI 仪表板](/power-bi/service-real-time-streaming)。 有关详细信息，请参阅详细的教程[分析使用 Stream Analytics 的电话呼叫数据并在 Power BI 仪表板中的将结果可视化](stream-analytics-manage-job.md)。

![ASA Power BI 仪表板](media/stream-analytics-solution-patterns/pbidashboard.png)

可以在从 Azure 门户在几分钟内构建此解决方案。 没有涉及任何大量的编码，并且使用 SQL 语言来表达业务逻辑。

此实时仪表板解决方案模式从事件源到浏览器中的 Power BI 仪表板提供了最低的延迟。 Azure Stream Analytics 是唯一的 Azure 服务使用此内置功能。

## <a name="use-sql-for-dashboard"></a>使用 SQL 的仪表板

Power BI 仪表板提供低延迟，但它不能用于生成完全成熟的 Power BI 报表。 常见的报告模式是首先输出到 SQL 数据库数据。 最新数据，然后使用 Power BI 的 SQL 查询 SQL 连接器。

![ASA SQL 仪表板](media/stream-analytics-solution-patterns/sqldashboard.png)

使用 SQL 数据库为您提供更多的灵活性，代价是增加延迟时间。 此解决方案是最适合于具有大于一秒的延迟要求的作业。 使用此方法，可以最大限度地进行进一步的切片的 Power BI 的实用程序和报表的切割数据。 您还可以使用其他仪表板解决方案，例如 Tableau 的灵活性。

SQL 不是高吞吐量数据存储，并从 Azure Stream Analytics 到 SQL 数据库的最大吞吐量为 24 MB/秒。 如果你的解决方案中的事件源生成数据的更高版本的增长，需要使用 Stream Analytics 中的处理逻辑来减少到 SQL 输出率。 技术，如筛选、 开窗聚合模式匹配与临时联接，可以使用分析函数。 到 SQL 输出率可以进一步优化使用中所述技术[到 Azure SQL 数据库的 Azure Stream Analytics 输出](stream-analytics-sql-output-perf.md)。

## <a name="incorporate-real-time-insights-into-your-application-with-event-messaging"></a>将合并到使用事件消息传送应用程序的实时见解

第二个最受欢迎使用 Stream Analytics 是生成实时警报。 在此解决方案模式下，可以使用 Stream Analytics 中的业务逻辑来检测[临时和空间模式](stream-analytics-geospatial-functions.md)或[异常](stream-analytics-machine-learning-anomaly-detection.md)，则生成警报信号。 但是，不同于 Stream Analytics 其中作为首选的终结点使用 Power BI 仪表板解决方案，可以使用大量的中间数据接收器。 这些接收器包括事件中心、 服务总线和 Azure Functions。 作为应用程序生成器中，需要决定哪些数据接收器最适合于你的方案。

必须实现下游事件使用者逻辑以在现有业务工作流中生成警报。 可以在 Azure Functions 中实现自定义逻辑，因为函数是可以执行此集成的最快方法。 Stream Analytics 作业的输出可在用于 Azure 函数的教程[从 Azure Stream Analytics 作业运行 Azure Functions](stream-analytics-with-azure-functions.md)。 Azure Functions 还支持各种类型的包括文本和电子邮件通知。 逻辑应用还可能用于与事件中心之间 Stream Analytics 和逻辑应用的此类集成。

![ASA 事件消息传送应用程序](media/stream-analytics-solution-patterns/eventmessagingapp.png)

事件中心，但是，提供最灵活的集成点。 很多其他服务，例如 Azure 数据资源管理器和时序见解，可以使用事件中心的事件。 从 Azure Stream Analytics，以完整的解决方案，可以直接向事件中心接收器连接服务。 事件中心也是最高吞吐量消息传递代理可在 Azure 上针对此类集成方案。

## <a name="dynamic-applications-and-websites"></a>动态应用程序和网站

可以创建自定义实时可视化效果，例如仪表板，或将可视化效果，使用 Azure Stream Analytics 和 Azure SignalR 服务的映射。 使用 SignalR，web 客户端可以更新，并且在实时中显示动态内容。

![ASA 动态应用程序](media/stream-analytics-solution-patterns/dynamicapp.png)

## <a name="incorporate-real-time-insights-into-your-application-through-data-stores"></a>通过数据存储合并你的应用程序的实时见解

大多数 web 服务和 web 应用程序现在使用请求/响应模式为表示层提供服务。 请求/响应模式是从简单到生成，并且可以轻松地扩展与使用无状态前端和可缩放存储，Cosmos DB 等低响应时间。

通常，较高的数据卷的基于 CRUD 的系统中创建性能瓶颈。 [事件溯源解决方案模式](/azure/architecture/patterns/event-sourcing.md)用于解决性能瓶颈。 临时模式和见解，也比较困难并且效率低下，若要从传统的数据存储中提取。 新式大批量的数据驱动的应用程序通常采用基于数据流的体系结构。 作为动态数据计算引擎的 azure Stream Analytics 是该体系结构中的关键。

![ASA 事件溯源应用](media/stream-analytics-solution-patterns/eventsourcingapp.png)

在此解决方案模式下，事件处理，然后通过 Azure Stream Analytics 聚合到数据存储区。 与使用传统的请求/响应模式的数据存储进行交互的应用程序层。 由于 Stream Analytics 能够处理大量的事件中实时应用程序是高度可缩放，而无需增加数据存储层。 数据存储层是系统实质上是具体化的视图。 [到 Azure Cosmos DB 的 azure Stream Analytics 输出](stream-analytics-documentdb-output.md)描述如何使用 Cosmos DB 作为 Stream Analytics 输出。

在其中处理逻辑是复杂且真实应用程序中需要单独升级逻辑的某些部分，可与事件中心作为中间事件代理一起组合多个 Stream Analytics 作业。

![ASA 复杂事件溯源应用](media/stream-analytics-solution-patterns/eventsourcingapp2.png)

此模式可以提高复原能力和系统的可管理性。 但是，即使 Stream Analytics 保证一次性处理，是在极少数的重复事件可能会进入中间的事件中心。 务必下游 Stream Analytics 作业的计划重复数据删除使用逻辑键 lookback 窗口中的事件。 有关事件传送的详细信息，请参阅[事件的传递保证](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics)引用。

## <a name="use-reference-data-for-application-customization"></a>应用程序自定义使用引用数据

Azure Stream Analytics 引用数据功能专为最终用户自定义警报阈值，处理规则，如并[地域隔离区](geospatial-scenarios.md)。 应用程序层可以接受参数的更改，并将其存储在 SQL 数据库中。 Stream Analytics 作业定期查询数据库中的更改，并使可访问的自定义参数，通过引用数据联接。 有关如何使用引用数据的应用程序自定义的详细信息，请参阅[SQL 引用数据](sql-reference-data.md)并[引用数据联接](/stream-analytics-query/reference-data-join-azure-stream-analytics)。

此模式还可用来实现规则引擎，从引用数据定义的规则的阈值。 有关规则的详细信息，请参阅[处理 Azure Stream Analytics 中的可配置基于阈值的规则](stream-analytics-threshold-based-rules.md)。

![ASA 引用数据应用](media/stream-analytics-solution-patterns/refdataapp.png)

## <a name="add-machine-learning-to-your-real-time-insights"></a>将机器学习添加到你的实时见解

Azure Stream Analytics 的内置[异常情况检测模型](stream-analytics-machine-learning-anomaly-detection.md)是引入到实时应用程序的机器学习的简便方法。 机器学习更广范围需求，请参阅[与 Azure 机器学习评分服务集成，Azure Stream Analytics](stream-analytics-machine-learning-integration-tutorial.md)。

有关想要将在线培训和评分合并到相同的 Stream Analytics 管道，高级的用户，请参阅此示例说明如何执行，其[线性回归](stream-analytics-high-frequency-trading.md)。

![ASA 机器学习应用](media/stream-analytics-solution-patterns/mlapp.png)

## <a name="near-real-time-data-warehousing"></a>准实时数据仓库

另一种常见模式是实时数据仓库，也称为流式处理数据仓库。 事件到达事件中心和 IoT 中心从应用程序中，除了[IoT Edge 上运行的 Azure Stream Analytics](stream-analytics-edge.md)可用于满足数据清理、 数据精简和数据存储和转发的需求。 IoT Edge 上运行的 Stream Analytics 可以正常处理带宽限制和连接问题系统中。 可以使用 SQL 输出适配器输出到 SQL 数据仓库;但是，最大吞吐量被限制为 10 MB/秒。

![ASA 数据仓库](media/stream-analytics-solution-patterns/datawarehousing.png)

提高具有一些滞后时间权衡吞吐量的一种方法是将事件存档到 Azure Blob 存储，然后[将其导入 SQL 数据仓库和 Polybase](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md)。 您必须手动将拼结在一起从 Stream Analytics 到 blob 存储输出和输入从 blob 存储到的 SQL 数据仓库[将数据存档按时间戳](stream-analytics-custom-path-patterns-blob-storage-output.md)和定期导入。

在此使用情况模式下，Azure Stream Analytics 用作近乎实时的 ETL 引擎。 新到达的事件持续转换和下游分析服务消耗的存储。

![ASA 高吞吐量数据仓库](media/stream-analytics-solution-patterns/datawarehousing2.png)

## <a name="archiving-real-time-data-for-analytics"></a>存档实时分析数据

大多数的数据科学和分析活动仍会脱机。 通过 Azure 数据湖存储第 2 代输出和 Parquet 输出格式，可以通过 Azure Stream Analytics 存档数据。 此功能中删除以数据直接传输到 Azure Data Lake Analytics、 Azure Databricks 和 Azure HDInsight 的摩擦。 Azure Stream Analytics 用作此解决方案中近实时 ETL 引擎。 您可以浏览 Data Lake 中的已存档的数据使用各种计算引擎。

![ASA 离线分析](media/stream-analytics-solution-patterns/offlineanalytics.png)

## <a name="use-reference-data-for-enrichment"></a>使用引用数据来扩充

数据扩充通常是必需的 ETL 引擎。 Azure Stream Analytics 支持通过数据充实[引用数据](stream-analytics-use-reference-data.md)从 SQL 数据库和 Azure Blob 存储。 登录 Azure Data Lake 和 SQL 数据仓库中的数据可以进行数据扩充。

![使用数据扩充 ASA 离线分析](media/stream-analytics-solution-patterns/offlineanalytics.png)

## <a name="operationalize-insights-from-archived-data"></a>操作已存档的数据的见解

如果您将脱机分析模式与近实时应用程序模式中，您可以创建反馈循环。 自动调整用于更改数据中的模式应用程序可通过反馈循环。 此反馈循环可以是只需更改警报的阈值或很复杂，比如重新训练机器学习模型。 相同的解决方案体系结构可以应用于这两个运行在云中和 IoT Edge 上的 ASA 作业。

![ASA insights 操作化](media/stream-analytics-solution-patterns/insightsoperationalization.png)

## <a name="how-to-monitor-asa-jobs"></a>如何监视 ASA 作业

Azure Stream Analytics 作业可以全天候可用性，以处理传入事件连续进行实时运行。 其运行时间保证对整个应用程序的运行状况至关重要。 虽然 Stream Analytics 是提供行业中唯一的流式处理分析服务[99.9%的可用性保证](https://azure.microsoft.com/support/legal/sla/stream-analytics/v1_0/)，仍可能会产生一定的停机时间。 多年来，Stream Analytics 已引入了指标、 日志和作业状态，以反映作业的运行状况。 所有网卡都通过 Azure Monitor 服务显示，并且可以进一步导出到 OMS。 有关详细信息，请参阅[了解 Stream Analytics 作业监视以及如何监视查询](stream-analytics-monitoring.md)。

![ASA 监视](media/stream-analytics-solution-patterns/monitoring.png)

有两个监视的重要事项：

- [作业失败状态](job-states.md)

    首先，这需要确保在作业运行。 而处于运行状态的作业，不生成任何新指标或日志。 作业可以更改为失败状态的原因多种多样，包括具有高的 SU 利用率级别 （即，用完资源）。

- [水印延迟指标](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/)

    此指标反映多久后您的处理管道处于时钟时间 （秒）。 某些延迟归功于固有的处理逻辑。 因此，监视上升趋势是更为重要比监视绝对值的数值。 稳定状态下延迟应通过应用程序设计，不是由监视或警报进行寻址。

如果失败，活动日志和[诊断日志](stream-analytics-job-diagnostic-logs.md)是开始查找的错误的最佳位置。

## <a name="build-resilient-and-mission-critical-applications"></a>生成可复原和关键的应用程序

无论 Azure Stream Analytics 的 SLA 保证和如何小心您运行的端到端应用程序中，会发生服务中断。 如果应用程序是关键任务应用程序，则需要要进行恢复，适当地准备用于服务中断。

用于触发的警报的应用程序，最重要的事情是检测下一步的警报。 您可以选择重启该作业从当前时间恢复时，忽略以往警报。 作业开始时间语义是第一个输出时，不是第一个输入时间。 输入将向后倒适量的时间，以保证在指定的时间的第一个输出为完整且正确。 不会获取部分聚合，并因此意外触发警报。

您还可以选择从一定量的过去的时间启动输出。 事件中心和 IoT 中心的保留策略保留合理的数据，以允许从过去的处理。 弊端是速度有多快可以赶上为当前时间和开始生成及时的新警报。 数据将失去其值快速随着时间推移，因此，必须快速赶上为当前时间。 有两种方法来保持同步，快速：

- 正在保持同步时，预配更多资源 (SU)。
- 从当前时间重新启动。

从当前重启的时间是轻松地处理在处理期间保留间隙的权衡。 重新启动这种方式也许能够正常警报方案，但可能会产生问题的仪表板方案和是是非初学者，以存档和数据仓库方案。

预配更多资源可以加快完成此过程，但具有处理速率激增的效果很复杂。

- 测试你的作业可以扩展到更多的 Su。 并非所有查询都可缩放。 您需要确保您的查询很[并行化](stream-analytics-parallelization.md)。

- 请确保有足够多的分区中的上游事件中心或 IoT 中心，您可以添加更多吞吐量单位 (Tu)，从而缩放输入的吞吐量。 请记住，每个事件中心 TU maxes out 输出 2 MB/秒的速率。

- 请确保你已预配足够的资源中输出接收器 （即，SQL 数据库、 Cosmos DB），因此它们不限制在输出中，这有时可能会激增会导致系统死锁。

最重要的事情是预期的处理速率更改、 测试这些方案之前投入生产，并准备好调整正确在故障恢复期间处理的规模。

在极端的情形下，传入事件都是所有延迟，[很可能所有延迟的事件将被删除](stream-analytics-time-handling.md)如果延迟到达时段应用到你的作业。 丢弃的事件可能似乎是一个令人费解的开头; 行为但是，假设 Stream Analytics 是一种实时处理引擎，它预期传入事件接近挂钟时间。 它具有要删除违反这些约束的事件。

### <a name="backfilling-process"></a>回填进程

幸运的是，可以使用以前的数据存档模式来适当地处理这些延迟事件。 思路是存档作业在到达时间处理传入事件，并将事件存档到 Azure Blob 或 Azure Data Lake Store 中的适当的时间存储桶具有其事件时间。 它并不重要事件的到达方式后期，将不会被删除。 它始终会移在正确的时间存储桶中。 在恢复期间，就可以重新处理该存档的事件以及回填到所选的存储结果。

![ASA 回填](media/stream-analytics-solution-patterns/backfill.png)

回填过程必须与处理系统，这很可能具有不同的编程模型比 Azure Stream Analytics 脱机批完成。 这意味着您必须重新实现整个处理逻辑。

有关回填，仍然很重要，至少暂时设置多个资源到输出接收器来处理更高的吞吐量比处理需求的稳定状态。

|方案  |现在仅从重新启动  |重启从上次停止时间 |从现在 + 回填存档事件重新启动|
|---------|---------|---------|---------|
|**仪表板**   |创建差异    |确定获取短时间的停机    |用于长时间的中断 |
|**警报**   |可以接受 |确定获取短时间的停机    |不需要 |
|**事件溯源应用** |可以接受 |确定获取短时间的停机    |用于长时间的中断 |
|**数据仓库**   |数据丢失  |可以接受 |不需要 |
|**离线分析**  |数据丢失  |可以接受 |不需要|

## <a name="putting-it-all-together"></a>汇总

不难想象，上面提到的所有解决方案模式可以都组合在一起在复杂的端到端系统。 组合的系统可以包括仪表板、 警报、 事件来源的应用程序、 数据仓库和脱机分析功能。

关键是设计您的系统的可组合模式，以便可以生成每个子系统，测试升级，并独立地恢复。

## <a name="next-steps"></a>后续步骤

现在已了解多种使用 Azure Stream Analytics 的解决方案模式。 接下来，你可以进行深入了解并创建第一个流分析作业：

* [使用 Azure 门户创建流分析作业](stream-analytics-quick-create-portal.md)。
* [使用 Azure PowerShell 创建流分析作业](stream-analytics-quick-create-powershell.md)。
* [使用 Visual Studio 创建流分析作业](stream-analytics-quick-create-vs.md)。
