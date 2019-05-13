---
title: 监视搜索服务的资源使用情况和查询指标 - Azure 搜索
description: 启用日志记录，从 Azure 搜索服务获取查询活动指标、资源使用情况以及其他系统数据。
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: bf78cd9b70aa4a82ef96fdd529d3ee5b1641038c
ms.sourcegitcommit: eea74d11a6d6ea6d187e90e368e70e46b76cd2aa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035359"
---
# <a name="monitor-resource-consumption-and-query-activity-in-azure-search"></a>在 Azure 搜索中监视资源使用情况和查询活动

可以在 Azure 搜索服务的“概览”页中查看系统数据，了解资源使用情况、查询指标，以及有多少配额可以用来创建更多的索引、索引器和数据源。 也可通过门户来配置 Log Analytics 或其他用于进行持久数据收集的资源。 

对于自我诊断和保留操作历史记录而言，设置日志十分有用。 在内部，日志会短时存在于后端，但足以进行调查和分析（如果你提交了支持票证）。 若要控制和访问日志信息，则应设置本文所述的某个解决方案。

本文介绍监视选项、如何启用日志记录和日志存储，以及如何查看日志内容。

## <a name="metrics-at-a-glance"></a>指标概览

内置到“概览”页中的“使用情况”和“监视”部分会针对资源使用情况和查询执行指标进行报告。 只要你开始使用此服务，就可以使用此信息，不需进行配置。 此页每隔几分钟就会刷新一次。 对于[将哪个层用于生产工作负荷](search-sku-tier.md)或是否要[调整活动副本和分区的数目](search-capacity-planning.md)这样的问题，可以根据这些指标进行最终决策，因为这些指标会显示资源的消耗速度，以及当前配置处理现有负载的有效程度。

“使用情况”选项卡显示资源可用性（相当于当前[限制](search-limits-quotas-capacity.md)而言）。 下图描述免费服务的情况，该服务的上限是每个类型 3 个对象，最大存储为 50 MB。 “基本”或“标准”服务的限制更高，在增加分区计数的情况下，最大存储会按比例增大。

![相对于有效限制的使用状态](./media/search-monitor-usage/usage-tab.png
 "相对于有效限制的使用状态")

## <a name="queries-per-second-qps-and-other-metrics"></a>每秒查询次数 (QPS) 和其他指标

“监视”选项卡显示每秒搜索查询数（简称 QPS，每分钟汇总一次）之类的指标的移动平均。 
搜索延迟是搜索服务处理搜索查询所需的时间（每分钟汇总一次）。 限制的搜索查询百分比（未显示）是受限制的搜索查询的百分比（也是每分钟汇总一次）。

这些数字是大概的数字，只是让你大致了解系统处理请求的情况。 实际的 QPS 可能高于或低于门户中报告的数字。

![每秒查询数活动](./media/search-monitor-usage/monitoring-tab.png "每秒查询数活动")

## <a name="activity-logs"></a>活动日志

**活动日志**从 Azure 资源管理器收集信息。 例如，在活动日志中发现的信息包括：创建或删除服务、更新资源组、查看名称可用性，或者获取处理请求所需的服务访问密钥。 

可以通过左侧导航窗格、顶部窗口命令栏中的“通知”或者“诊断并解决问题”页访问**活动日志**。

对于服务内任务（例如创建索引或删除数据源），你会看到针对每个请求的常规通知（例如“获取管理密钥”），但看不到具体操作本身。 对于这种级别的信息，必须启用附加监视解决方案。

## <a name="add-on-monitoring-solutions"></a>附加监视解决方案

Azure 搜索不在其管理的对象之外存储任何数据，这意味着日志数据必须在外部进行存储。 若要保留日志数据，可以配置下面的任何资源。 

下表比较了各种选项，这些选项用于存储日志以及添加深度监视指标，以便通过 Application Insights 监视服务操作和查询工作负荷。

| 资源 | 用途 |
|----------|----------|
| [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) | 记录的事件和查询指标，基于下面的架构上与应用程序中的用户事件关联起来。 这是唯一会考虑用户操作或信号的解决方案，它会映射用户发起的搜索中的事件，而不会筛选应用程序代码提交的请求。 若要使用此方法，请将检测代码复制并粘贴到源文件中，以便将请求信息路由到 Application Insights。 有关详细信息，请参阅[搜索流量分析](search-traffic-analytics.md)。 |
| [Azure Monitor 日志](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview) | 记录的事件和查询指标，基于下面的架构。 事件会记录到 Log Analytics 工作区。 可以针对工作区运行查询，以便从日志返回详细信息。 有关详细信息，请参阅[开始使用 Azure Monitor 日志](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-viewdata) |
| [Blob 存储](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview) | 记录的事件和查询指标，基于下面的架构。 事件记录到 Blob 容器并存储在 JSON 文件中。 使用 JSON 编辑器来查看文件内容。|
| [事件中心](https://docs.microsoft.com/azure/event-hubs/) | 记录的事件和查询指标，基于本文中记录的架构。 对于很大的日志，请选择此项作为备用数据收集服务。 |

Azure Monitor 日志和 Blob 存储都可用作一项免费共享服务，以便您可以尝试一下你的 Azure 订阅的生存期内无需任何费用。 Application Insights 可以免费注册和使用，前提是应用程序数据大小不超出特定限制（有关详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/monitor/)）。

下一部分详述如何通过多个步骤来启用和使用 Azure Blob 存储，以便收集和访问 Azure 搜索操作创建的日志数据。

## <a name="enable-logging"></a>启用日志记录

索引编制和查询工作负荷的日志记录默认处于关闭状态，并依赖于日志记录基础结构和长期外部存储的附加解决方案。 Azure 搜索本身保留的数据只有其创建和管理的对象，因此日志必须存储在别处。

本部分介绍如何使用 Blob 存储来存储记录的事件和指标数据。

1. [创建存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)（如果还没有）。 可以将它置于 Azure 搜索所在的资源组中，这样当你以后需要删除本练习中使用的所有资源时，就可以简化清理过程。

2. 打开搜索服务的“概览”页。 在左侧导航窗格中，向下滚动到“监视”，然后单击“启用监视”。

   ![启用监视](./media/search-monitor-usage/enable-monitoring.png "启用监视")

3. 选择要导出的数据：日志和/或指标。 可以将其复制到存储帐户，将其发送到事件中心或将其导出到 Azure Monitor 日志。

   若要存档到 Blob 存储，只有存储帐户必须存在。 容器和 Blob 会在导出日志数据时创建。

   ![配置 Blob 存储存档](./media/search-monitor-usage/configure-blob-storage-archive.png "配置 Blob 存储存档")

4. 保存配置文件。

5. 通过创建或删除对象（创建日志事件）以及通过提交查询（生成指标）来测试日志记录。 

保存配置文件后，日志记录就会启用。 仅当存在要记录或度量的活动时，才会创建容器。 将数据复制到存储帐户时，数据会被格式化为 JSON 并置于两个容器中：

* insights-logs-operationlogs：用于搜索流量日志
* insights-metrics-pt1m：用于指标

**需要一个小时之前容器会在 Blob 存储中。没有一个 blob，每小时，每个容器。**

可以使用 [Visual Studio Code](#download-and-open-in-visual-studio-code) 或其他 JSON 编辑器来查看文件。 

### <a name="example-path"></a>示例路径

```
resourceId=/subscriptions/<subscriptionID>/resourcegroups/<resourceGroupName>/providers/microsoft.search/searchservices/<searchServiceName>/y=2018/m=12/d=25/h=01/m=00/name=PT1H.json
```

## <a name="log-schema"></a>日志架构
包含搜索服务流量日志的 Blob 的结构如此部分所述。 每个 Blob 都有一个名为 **records** 的根对象，该对象包含一组日志对象。 每个 Blob 包含同一小时内发生的所有操作的记录。

| 名称 | Type | 示例 | 说明 |
| --- | --- | --- | --- |
| time |datetime |"2018-12-07T00:00:43.6872559Z" |操作的时间戳 |
| resourceId |字符串 |“/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/> MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE” |ResourceId |
| operationName |字符串 |“Query.Search” |操作的名称 |
| operationVersion |string |"2019-05-06" |使用的 api-version |
| category |字符串 |“OperationLogs” |常量 |
| resultType |字符串 |“Success” |可能的值：Success 或 Failure |
| resultSignature |int |200 |HTTP 结果代码 |
| durationMS |int |50 |操作持续时间，以毫秒为单位 |
| 属性 |对象 |请参阅下表 |包含特定于操作的数据的对象 |

**属性架构**

| 名称 | Type | 示例 | 说明 |
| --- | --- | --- | --- |
| 描述 |字符串 |“GET /indexes('content')/docs” |操作的终结点 |
| 查询 |string |"?search=AzureSearch&$count=true&api-version=2019-05-06" |查询参数 |
| 文档 |int |42 |处理的文档数目 |
| IndexName |字符串 |“testindex” |与操作关联的索引名称 |

## <a name="metrics-schema"></a>度量值架构

针对查询请求来捕获指标。

| 名称 | Type | 示例 | 说明 |
| --- | --- | --- | --- |
| resourceId |字符串 |“/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/>MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE” |资源 ID |
| metricName |字符串 |“Latency” |度量值名称 |
| time |datetime |"2018-12-07T00:00:43.6872559Z" |操作的时间戳 |
| average |int |64 |度量值时间间隔内原始样本的平均值 |
| minimum |int |37 |度量值时间间隔内原始样本的最小值 |
| maximum |int |78 |度量值时间间隔内原始样本的最大值 |
| total |int |258 |度量值时间间隔内原始样本的总计值 |
| 计数 |int |4 |用于生成度量值的原始样本数 |
| timegrain |字符串 |“PT1M” |采用 ISO 8601 的度量值时间粒度 |

所有度量值会按一分钟的时间间隔报告。 每个度量值都会显示每分钟的最小、最大和平均值。

对于 SearchQueriesPerSecond 度量值，最小值是该分钟内已注册的每秒搜索查询次数最低值。 最大值也是如此。 平均值是一分钟内的聚合值。
假设一分钟内出现以下情形：有 1 秒出现高负载（这是 SearchQueriesPerSecond 的最大值），紧接着有 58 秒的平均负载，最后 1 秒只有 1 个查询（这是最小值）。

对于 ThrottledSearchQueriesPercentage、minimum、maximum、average 和 total，全都具有相同的值：在一分钟内的搜索查询总数中，已限制搜索查询百分比。

## <a name="download-and-open-in-visual-studio-code"></a>下载文件并在 Visual Studio Code 中打开

可以使用任何 JSON 编辑器来查看日志文件。 如果没有编辑器，建议使用 [Visual Studio Code](https://code.visualstudio.com/download)。

1. 在 Azure 门户中打开存储帐户。 

2. 在左侧导航窗格中，单击“Blob”。 此时会看到 **insights-logs-operationlogs** 和 **insights-metrics-pt1m**。 这些容器是在将日志数据导出到 Blob 存储时由 Azure 搜索创建的。

3. 单击文件夹层次结构，直至找到 .json 文件。  通过上下文菜单来下载文件。

待文件下载完以后，在 JSON 编辑器中将其打开即可查看内容。

## <a name="use-system-apis"></a>使用系统 API
Azure 搜索 REST API 和 .NET SDK 支持采用编程方式访问服务指标、索引和索引器信息，以及文档计数。

* [获取服务统计信息](/rest/api/searchservice/get-service-statistics)
* [获取索引统计信息](/rest/api/searchservice/get-index-statistics)
* [计数文档](/rest/api/searchservice/count-documents)
* [获取索引器状态](/rest/api/searchservice/get-indexer-status)

若要使用 PowerShell 或 Azure CLI 来启用监视，请参阅[此处](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs#how-to-enable-collection-of-diagnostic-logs)的文档。

## <a name="next-steps"></a>后续步骤

[管理 Microsoft Azure 上的搜索服务](search-manage.md)一文提供有关服务管理的详细信息，[性能和优化](search-performance-optimization.md)一文提供优化指南。