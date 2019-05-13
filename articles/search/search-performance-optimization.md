---
title: 部署策略和有关优化性能-Azure 搜索最佳实践
description: 了解优化 Azure 搜索性能和配置最佳规模的技术与最佳做法。
author: LiamCavanagh
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 03/02/2019
ms.author: liamca
ms.custom: seodec2018
ms.openlocfilehash: 32352a857f0a74dc008dc1ad76b4a5951a36b956
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024550"
---
# <a name="deployment-strategies-and-best-practices-for-optimizing-performance-on-azure-search"></a>部署策略和有关优化 Azure 搜索的性能最佳实践

本文介绍高级方案的可伸缩性和可用性的复杂要求的最佳实践。 

## <a name="develop-baseline-numbers"></a>制定基准数字
针对搜索性能进行优化，应重点降低查询执行时间。 若要执行此操作，需要了解典型的查询负载如下所示。 以下准则可以帮助您实现此基准查询数目。

1. 选取完成典型搜索请求应该花费的目标延迟（或最大时间量）。
2. 针对搜索服务，使用现实数据集来创建和测试工作负载，以测量这些延迟速率。
3. 每秒 (QPS) 查询以较低数字开头和逐渐增加在测试中执行，直到查询延迟降至低于定义的目标延迟数。 这是一个重要的基准，可帮助你计划应用程序在使用量增长方面的规模。
4. 只要有可能，请重用 HTTP 连接。 如果使用 Azure 搜索 .NET SDK，这意味着你应该重用某个实例或 [SearchIndexClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient) 实例，并且如果使用 REST API，就应该重用单个 HttpClient。
5. 不同查询请求的物质，因此该搜索时使用索引的不同部分。 变体很重要，因为如果不断执行相同的搜索请求时，缓存的数据将开始使性能看起来会超过它可能会使用更多不同查询设置好。
6. 改变的查询请求的结构，以便获取不同类型的查询。 不是每个搜索查询执行相同的级别。 例如，文档查找或搜索建议为具有大量平面和筛选器的查询比通常更快。 正如您期望在生产环境中测试组合应大致相同的比例中包括各种查询。  

在创建这些测试工作负荷时，需要记住 Azure 搜索的下面这些特征︰

+ 可以通过将过多的搜索查询推送一次重载你的服务。 发生这种情况时，会看到 HTTP 503 响应代码。 若要在测试期间避免 503，启动不同范围的搜索请求以添加更多的搜索请求时，请参阅延迟速率中的差异。

+ Azure 搜索不在后台运行索引编制任务。 如果你的服务处理查询和索引工作负荷，同时，考虑到这一点通过任一引入到你的查询测试，索引作业或通过浏览运行索引作业在非高峰时间的选项。

> [!NOTE]
> [Visual Studio Load Testing](https://www.visualstudio.com/docs/test/performance-testing/run-performance-tests-app-before-release) 是真正适合执行基准测试的好方法，因为在需要针对 Azure 搜索执行查询并启用请求的并行化时，它允许执行 HTTP 请求。
> 
> 

## <a name="scaling-for-high-query-volume-and-throttled-requests"></a>查询量很大的缩放和限制的请求数
在收到太多限制的请求数，或超过目标延迟速率增加的查询负载时，您可以查找来降低延迟速率中两种方式之一：

1. **增加副本数：** 副本就像数据副本，它允许 Azure 搜索对面向多个副本的请求进行负载均衡。  在副本之间的所有负载均衡和数据的复制都是由 Azure 搜索管理的，可以随时更改为服务分配的副本数量。  最大可在一个标准搜索服务中分配 12 个副本，并在一个基本搜索服务中分配 3 个副本。 可以从 [Azure 门户](search-create-service-portal.md)或 [PowerShell](search-manage-powershell.md) 调整副本。
2. **增加搜索层：** Azure 搜索来自[许多层](https://azure.microsoft.com/pricing/details/search/)，其中每个层都提供不同级别的性能。  在某些情况下，可能有太多查询，即使在副本数超出限制时，所在的层仍无法提供足够的低延迟速率。在这种情况下，你可能想要考虑利用更高的搜索层，比如 Azure 搜索 S3 层，非常适合于使用大量的文档和极高查询工作负荷方案之一。

## <a name="scaling-for-slow-individual-queries"></a>缓慢的各个查询的扩展
高延迟速率的另一个原因是单个查询花费太长时间才能完成。 在这种情况下，将不帮助添加副本。 两个选项可能有助于包括以下各项的选项：

1. **增加分区数**：分区是一种机制，可在额外资源之间拆分数据。 添加第二个分区将数据拆分为两个、 第三个分区拆分为三种类型，等等。 一个正的副作用是，速度较慢的查询有时会执行速度更快由于并行计算。 我们已记下在低选择性查询，如匹配许多文档或通过大量的文档进行计数的方面的查询的并行化。 由于需要大量计算的文档的相关性进行评分或计算文档，添加额外的分区的数量可帮助更快地完成查询。  
   
   在标准搜索服务中最多可以有 12 个分区，在基本搜索服务中最多可以有 1 个分区。  可以从 [Azure 门户](search-create-service-portal.md)或 [PowerShell](search-manage-powershell.md) 调整分区。

2. **限制高基数字段：** 高基数字段包含的可查找或具有大量唯一值，并且结果是，当计算结果会消耗大量资源的可筛选字段。 例如，产品 ID 或描述字段设置为可查找/可筛选将计为高基数因为值从文档到文档的大部分都是唯一。 只要有可能，请限制高基数字段的数量。

3. **增加搜索层：** 另一种方法是，向上移动到更高的 Azure 搜索层，可以为较慢的查询改进性能。 每个更高的层提供更快的 Cpu 和更多内存，这两种对查询性能产生积极的影响。

## <a name="scaling-for-availability"></a>调整可用性的规模
副本不仅可以帮助缩短查询延迟，而且还可以允许高可用性。 借助单个副本，应该可以预期周期性的停机时间，因为在软件更新之后，或针对其他将执行的维护活动后，服务器会周期性停机。  因此，请务必考虑应用程序是否需要搜索（查询）以及写入（编制索引事件）的高可用性。 Azure 搜索在具有以下属性的所有付费搜索产品/服务上提供 SLA 选项：

* 针对只读工作负荷（查询），需要有 2 个副本才可达到高可用性
* 针对读写工作负荷（查询和索引），需要有 3 个或更多副本才可达到高可用性

有关这方面的更多详细信息，请访问 [Azure 搜索服务级别协议](https://azure.microsoft.com/support/legal/sla/search/v1_0/)。

由于副本是个数据副本，拥有多个副本允许 Azure 搜索执行不仅允许将继续在其他副本上的查询执行计算机重新启动和维护针对一个副本。 相反，如果要立即复制副本，应，则会导致查询性能下降，假定这些副本已充分利用的资源。

## <a name="scaling-for-geo-distributed-workloads-and-geo-redundancy"></a>缩放地理分散的工作负荷和异地冗余
对于地理上分散的工作负荷，都远承载 Azure 搜索的数据中心的用户将具有较高延迟速率。 一种解决方法是预配具有更接近于这些用户的区域中的多个搜索服务。 Azure 搜索当前不提供自动化方法来跨区域异地复制 Azure 搜索索引，但有一些技巧，可以用来使此过程很容易实现和管理。 我们会在下面几节介绍这些技巧。

地理上分散的一组搜索服务的目标是在其中将用户路由到 Azure 搜索服务提供最低的延迟，在此示例中所示的两个或多个区域中有可用的两个或多个索引：

   ![按区域的服务的交叉表][1]

### <a name="keeping-data-in-sync-across-multiple-azure-search-services"></a>让数据在多个 Azure 搜索服务之间保持同步
有两个选项可让分布式搜索服务保持同步，包括使用 [Azure 搜索索引器](search-indexer-overview.md)或推送 API（也称为 [Azure 搜索 REST API](https://docs.microsoft.com/rest/api/searchservice/)。  

### <a name="use-indexers-for-updating-content-on-multiple-services"></a>使用索引器更新多个服务上的内容

如果已在一个服务上使用索引器，您可以配置第二个索引器上要使用相同的数据源对象的第二个服务将数据提取从相同的位置。 每个区域中的每个服务都有其自己的索引器和目标索引 （搜索索引未共享，这意味着重复数据），但每个索引器引用的是相同的数据源。

下面是该体系结构将如下所示的高级视觉对象。

   ![具有分布式索引器和服务组合的单一数据源][2]

### <a name="use-rest-apis-for-pushing-content-updates-on-multiple-services"></a>使用 REST Api，可用于按多个服务上的内容更新
如果使用 Azure 搜索 REST API [Azure 搜索索引中的推送内容](https://docs.microsoft.com/rest/api/searchservice/update-index)，你可以保持各种搜索服务同步通过将更改推送到所有搜索服务时需要进行更新时。 在代码中，请确保处理情况下，一个搜索服务更新失败，但失败的其他搜索服务。

## <a name="leverage-azure-traffic-manager"></a>利用 Azure 流量管理器
通过使用 [Azure 流量管理器](../traffic-manager/traffic-manager-overview.md)，可以将请求路由到多个地理定位的网站，而这些网站由多个 Azure 搜索服务支持。 流量管理器的一个优点是，它可以探测 Azure 搜索以确保其可用，并在发生停机时会用户路由到备用搜索服务。 此外，如果通过 Azure 网站路由搜索请求，Azure 流量管理器允许在网站启动但没有 Azure 搜索的情形下进行负载均衡。 下面是利用流量管理器的体系结构的示例。

   ![按区域服务的交叉表（带有中央流量管理器）][3]

## <a name="monitor-performance"></a>监视性能
Azure 搜索提供能够分析和监视通过服务的性能[搜索流量分析](search-traffic-analytics.md)。 启用此功能，并将检测添加到客户端应用程序，可以 （可选） 到 Azure 存储帐户，然后可以处理以便分析或在 Power BI 中直观显示记录的单独的搜索操作以及聚合度量值。 指标捕获这种方式提供性能统计信息，如查询或查询响应时间的平均数。 此外，通过操作日志记录，可以向下钻取到解特定搜索操作的详细信息。

流量分析是可用于了解延迟速率从该 Azure 搜索的角度。 由于记录的查询性能指标基于在 Azure 搜索中完全处理一个查询所花费的时间（从请求它的时间到发送时间），因此，能够使用此工具来确定延迟问题是来自 Azure 搜索服务端还是来自服务外部，如来自网络延迟。  

## <a name="next-steps"></a>后续步骤
若要深入了解有关定价层和每层的服务限制，请参阅 [Azure 搜索中的服务限制](search-limits-quotas-capacity.md)。

请访问[容量规划](search-capacity-planning.md)，了解有关分区和副本组合的详细信息。

要获得有关性能的更多深入分析，或者要查看如何实现在这篇文章中讨论的优化演示，请观看以下视频︰

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 
> 

<!--Image references-->
[1]: ./media/search-performance-optimization/geo-redundancy.png
[2]: ./media/search-performance-optimization/scale-indexers.png
[3]: ./media/search-performance-optimization/geo-search-traffic-mgr.png
