---
title: Azure Cosmos DB：SQL .NET Core API、SDK 和资源
description: 了解有关 SQL .NET Core API 和 SDK 的所有信息，包括发布日期、停用日期和 Azure Cosmos DB .NET Core SDK 各版本之间所做的更改。
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/22/2018
ms.author: sngun
ms.openlocfilehash: 4ec9d5e605a2319a04dac4cb52dbe49c77354479
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510637"
---
# <a name="azure-cosmos-db-net-core-sdk-for-sql-api-release-notes-and-resources"></a>适用于 SQL API 的 Azure Cosmos DB .NET Core SDK：发行说明和资源
> [!div class="op_single_selector"]
> * [.NET](sql-api-sdk-dotnet.md)
> * [.NET 更改源](sql-api-sdk-dotnet-changefeed.md)
> * [.NET Core](sql-api-sdk-dotnet-core.md)
> * [Node.js](sql-api-sdk-node.md)
> * [异步 Java](sql-api-sdk-async-java.md)
> * [Java](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](https://docs.microsoft.com/rest/api/cosmos-db/)
> * [REST 资源提供程序](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [BulkExecutor - .NET](sql-api-sdk-bulk-executor-dot-net.md)
> * [BulkExecutor - Java](sql-api-sdk-bulk-executor-java.md)

| |  |
|---|---|
|**SDK 下载**| [NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/)|
|**API 文档**|[ 参考文档](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet)|
|**示例**|[.NET代码示例](sql-api-dotnet-samples.md)|
|**入门**|[Azure Cosmos DB .NET Core SDK 入门](sql-api-dotnet-core-get-started-preview.md)|
|**Web 应用教程**|[使用 Azure Cosmos DB 进行 Web 应用程序开发](sql-api-dotnet-application.md)|
|**当前受支持的框架**|[.NET Standard 1.6 和 .NET Standard 1.5](https://www.nuget.org/packages/NETStandard.Library)|

## <a name="release-notes"></a>发行说明

Azure Cosmos DB .NET Core SDK 具有与最新版 [Azure Cosmos DB .NET SDK](sql-api-sdk-dotnet.md) 相同的功能。

### <a name="a-name3001-preview3001-preview"></a><a name="3.0.0.1-preview"/>3.0.0.1-preview
* .NET SDK [版本 3.0.0](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) 的预览版 1（公共预览版）。
* 适用于 .NET Standard，.NET Standard 支持 .NET Framework 4.6.1+ 和 .NET Core 2.0+
* 新对象模型，使用顶级 CosmosClient 和方法拆分成相关的 CosmosDatabases、CosmosContainers 和 CosmosItems 类。
* 支持流。
* 对来自服务器的 CosmosResponseMessage 进行了更新以返回状态代码，并且仅在未返回响应时引发异常。

### <a name="a-name240240"></a><a name="2.4.0"/>2.4.0

* SDK 的 System.Net.Http 版本匹配的 NuGet 包中定义的内容
* 增加小数精度为 LINQ 查询的大小。
* 添加新的类 CompositePath、 CompositePathSortOrder、 SpatialSpec、 SpatialType 和 PartitionKeyDefinitionVersion
* 已添加到 DocumentCollection TimeToLivePropertyPath
* 添加了的 CompositeIndexes 和到 IndexPolicy SpatialIndexes
* 添加了的版本为 PartitionKeyDefinition
* 添加到 PartitionKey None

### <a name="a-name230230"></a><a name="2.3.0"/>2.3.0

 * 添加了的 IdleTcpConnectionTimeout、 OpenTcpConnectionTimeout、 MaxRequestsPerTcpConnection 和到 ConnectionPolicy MaxTcpConnectionsPerEndpoint。
 
### <a name="a-name223223"></a><a name="2.2.3"/>2.2.3

* 诊断改进

### <a name="a-name222222"></a><a name="2.2.2"/>2.2.2

* 添加了环境变量设置“POCOSerializationOnly”。

* 删除了 DocumentDB.Spatial.Sql.dll，现在在 Microsoft.Azure.Documents.ServiceInterop.dll 中包含

### <a name="a-name221221"></a><a name="2.2.1"/>2.2.1

* 对 StoredProcedure 执行调用故障转移期间的重试逻辑进行了改进。

* 将 DocumentClientEventSource 设为单一实例。 

* 修复了 GatewayAddressCache 超时不遵守 ConnectionPolicy RequestTimeout 的问题。

### <a name="a-name220220"></a><a name="2.2.0"/>2.2.0

* 对于直接/TCP 传输诊断，添加了 TransportException，这是该 SDK 的一个内部异常类型。 出现在异常消息中时，此类型会输出附加信息，以便排查客户端连接问题。

* 添加了新的构造函数重载，它采用 HttpMessageHandler，后者是用于发送 HttpClient 请求的 HTTP 处理程序堆栈（例如，HttpClientHandler）。

* 修复了无法正确处理带有空值的标头的 bug。

* 改进了集合缓存验证。

### <a name="a-name213213"></a><a name="2.1.3"/>2.1.3

* 更新到 4.3.2 版 System.Net.Security。

### <a name="a-name212212"></a><a name="2.1.2"/>2.1.2

* 诊断跟踪改进。

### <a name="a-name211211"></a><a name="2.1.1"/>2.1.1

* 为多区域请求暂时性故障添加了更多复原能力。

### <a name="a-name210210"></a><a name="2.1.0"/>2.1.0

* 添加了多区域写入支持。
* 使用 TOP 和 MaxBufferedItemCount 改进跨分区查询性能。

### <a name="a-name200200"></a><a name="2.0.0"/>2.0.0

* 添加了请求取消支持。
* 将 SetCurrentLocation 添加到 ConnectionPolicy，它会根据区域自动填充首选位置。
* 修复了具有 Min/Max 以及在单个分区上没有文档匹配的筛选的跨分区查询中的 Bug。
* DocumentClient 方法现在与 IDocumentClient 具有奇偶校验。
* 更新了直接 TCP 传输堆栈以减少建立的连接数。
* 为非 Windows 客户端添加了对 Direct Mode TCP 的支持。

### <a name="a-name200-preview2200-preview2"></a><a name="2.0.0-preview2"/>2.0.0-preview2

* 添加了请求取消支持。
* 将 SetCurrentLocation 添加到 ConnectionPolicy，它会根据区域自动填充首选位置。
* 修复了具有 Min/Max 以及在单个分区上没有文档匹配的筛选的跨分区查询中的 Bug。

### <a name="a-name200-preview200-preview"></a><a name="2.0.0-preview"/>2.0.0-preview

* DocumentClient 方法现在与 IDocumentClient 具有奇偶校验。
* 更新了直接 TCP 传输堆栈以减少建立的连接数。
* 为非 Windows 客户端添加了对 Direct Mode TCP 的支持。

### <a name="a-name11001100"></a><a name="1.10.0"/>1.10.0

* 将 ConsistencyLevel 属性添加到了 FeedOptions。
* 将 JsonSerializerSettings 添加到了 RequestOptions 和 FeedOptions。
* 将 EnableReadRequestsFallback 添加到了 ConnectionPolicy。

### <a name="a-name191191"></a><a name="1.9.1"/>1.9.1

* 修复了临界情况下跨分区 order by 查询出现的 KeyNotFoundException。
* 修复了在 LINQ 查询的 select 子句中不接受 JsonProperty 属性的 bug。

### <a name="a-name182182"></a><a name="1.8.2"/>1.8.2

* 修复了在某些争用情况下出现的 bug，该 bug 在使用会话一致性级别时导致间歇性错误“Microsoft.Azure.Documents.NotFoundException: 读取会话不可用于输入会话令牌”。

### <a name="a-name181181"></a><a name="1.8.1"/>1.8.1

* 修复了其中 FeedOptions.MaxItemCount = -1 引发 System.ArithmeticException: 页大小为负的回归。
* 向 QueryMetrics 添加了新的 ToString() 函数。
* 公开了有关读取集合的分区统计信息。
* 向 ChangeFeedOptions 添加了 PartitionKey 属性。
* 小 bug 修复。

### <a name="a-name171171"></a><a name="1.7.1"/>1.7.1
 
 * 添加了通过使用 DocumentCollection 上的 UniqueKeyPolicy 属性来指定文档唯一索引的功能。
 * 修复了自定义 JsonSerializer 设置无法用于某些查询和存储过程执行的 bug。

### <a name="a-name170170"></a><a name="1.7.0"/>1.7.0
 
 * 在 API 参考文档、程序集中的元数据信息和 NuGet 包中品牌从 Azure DocumentDB 更改为 Azure Cosmos DB。
 * 通过以直接连接模式发送的请求的响应公开诊断信息和延迟。 在 ResourceResponse 类中属性名称是 RequestDiagnosticsString 和 RequestLatency。
 * 此 SDK 版本需要最新版本的 Azure Cosmos DB 模拟器（可从 https://aka.ms/cosmosdb-emulator 下载）。
 
### <a name="a-name160160"></a><a name="1.6.0"/>1.6.0

* 添加了多项可靠性修复和改进。

### <a name="a-name151151"></a><a name="1.5.1"/>1.5.1 

* 内部更改与支持 [Microsoft.Azure.Graphs](https://docs.microsoft.com/azure/cosmos-db/graph-sdk-dotnet) 相关

### <a name="a-name150150"></a><a name="1.5.0"/>1.5.0 

* 添加了 PartitionKeyRangeId 的支持作为 FeedOption，将查询结果限定在某个特定分区键范围值。
* 添加了 StartTime 的支持作为 ChangeFeedOption，以开始查找该时间后的更改。

### <a name="a-name141141"></a><a name="1.4.1"/>1.4.1

*   修复了 JsonSerializable 类中可能引起堆栈溢出异常的问题。

### <a name="a-name140140"></a><a name="1.4.0"/>1.4.0

*   现已开始支持在实例化 [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient?view=azure-dotnet) 实例时指定自定义 JsonSerializerSettings。

### <a name="a-name132132"></a><a name="1.3.2"/>1.3.2

*   支持 .NET Standard 1.5 作为目标框架之一。

### <a name="a-name131131"></a><a name="1.3.1"/>1.3.1

*   修复了影响 x64 计算机的一个问题，该计算机不支持 SSE4 指令，并在运行 Azure Cosmos DB 查询时引发 SEHException。

### <a name="a-name130130"></a><a name="1.3.0"/>1.3.0

*   添加了对名为 ConsistentPrefix 的新一致性级别的支持。
*   添加了对单独分区的查询指标的支持。
*   添加了对限制查询的继续令牌大小的支持。
*   添加了对失败请求的详情跟踪的支持。
*   改进了 SDK 中的一些性能。

### <a name="a-name122122"></a><a name="1.2.2"/>1.2.2

* 修复了忽略 FeedOptions 中为聚合查询提供的 PartitionKey 值的问题。
* 修复了在中途跨分区执行 OrderBy 查询期间透明处理分区管理的问题。

### <a name="a-name121121"></a><a name="1.2.1"/>1.2.1

* 修复了在 ASP.NET 上下文内使用时，在某些异步 API 中导致死锁的问题。

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0

* 修复程序，用于使 SDK 更具弹性，以便在某些情况下自动故障转移。

### <a name="a-name112112"></a><a name="1.1.2"/>1.1.2

* 修复程序，用于解决偶尔导致“WebException: 无法解析远程名称”的问题。
* 通过向 ReadDocumentAsync API 添加新重载，添加了对直接读取类型化文档的支持。

### <a name="a-name111111"></a><a name="1.1.1"/>1.1.1

* 添加了对聚合查询（COUNT、MIN、MAX、SUM 和 AVG）的 LINQ 支持。
* 修复了由于使用了事件处理程序而导致的 ConnectionPolicy 对象的内存泄漏问题。
* 修复了使用 ETag 时 UpsertAttachmentAsync 不正常工作的问题。
* 修复了对字符串字段进行排序时跨分区按查询条件排序不正常工作的问题。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0

* 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。 请参阅[聚合支持](how-to-sql-query.md#Aggregates)。
* 将分区集合上的最小吞吐量从 10,100 RU/s 降低到 2500 RU/s。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0

通过 Azure Cosmos DB .NET Core SDK，可构建能在 Windows、Mac 和 Linux 上快速运行的跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。 Azure Cosmos DB .NET Core SDK 的最新版本完全兼容 [Xamarin](https://www.xamarin.com)，可用于生成面向 iOS、Android 和 Mono (Linux) 的应用程序。  

### <a name="a-name010-preview010-preview"></a><a name="0.1.0-preview"/>0.1.0-preview

通过 Azure Cosmos DB .NET Core Preview SDK，可构建能在 Windows、Mac 和 Linux 上快速运行的跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。

Azure Cosmos DB .NET Core 预览版 SDK 与最新版 [Azure Cosmos DB .NET SDK](sql-api-sdk-dotnet.md) 功能相同，并支持以下内容：
* 所有[连接模式](performance-tips.md#networking)：网关模式、Direct TCP 和 Direct HTTP。
* 所有[一致性级别](consistency-levels.md)：强一致性、会话一致性、有限过期一致性和最终一致性。
* [已分区集合](partition-data.md)。
* [多区域数据库帐户和异地复制](distribute-data-globally.md)。

若有与此 SDK 相关的问题，请发布到 [StackOverflow](https://stackoverflow.com/questions/tagged/azure-documentdb)，或在 [GitHub 存储库](https://github.com/Azure/azure-documentdb-dotnet/issues)中提出问题。

## <a name="release--retirement-dates"></a>发布和停用日期

| Version | 发布日期 | 停用日期 |
| --- | --- | --- |
| [2.4.0](#2.4.0) |2019 年 5 月 5， |--- |
| [2.3.0](#2.3.0) |2019 年 4 月 4日日 |--- |
| [2.2.3](#2.2.3) |2019 年 3 月 11日日 |--- |
| [2.2.2](#2.2.2) |2019 年 2 月 6 日， |--- |
| [2.2.1](#2.2.1) |2018 年 12 月 24 日 |--- |
| [2.2.0](#2.2.0) |2018 年 12 月 7 日 |--- |
| [2.1.3](#2.1.3) |2018 年 10 月 15 日 |--- |
| [2.1.2](#2.1.2) |2018 年 10 月 4 日 |--- |
| [2.1.1](#2.1.1) |2018 年 9 月 27 日 |--- |
| [2.1.0](#2.1.0) |2018 年 9 月 21 日 |--- |
| [2.0.0](#2.0.0) |2018 年 9 月 7 日 |--- |
| [1.9.1](#1.9.1) |2018 年 3 月 9 日 |--- |
| [1.8.2](#1.8.2) |2018 年 2 月 21 日 |--- |
| [1.8.1](#1.8.1) |2018 年 2 月 5 日 |--- |
| [1.7.1](#1.7.1) |2017 年 11 月 16 日 |--- |
| [1.7.0](#1.7.0) |2017 年 11 月 10 日 |--- |
| [1.6.0](#1.6.0) |2017 年 10 月 17 日 |--- |
| [1.5.1](#1.5.1) |2017 年 10 月 2日 |--- |
| [1.5.0](#1.5.0) |2017 年 8 月 10 日 |--- | 
| [1.4.1](#1.4.1) |2017 年 8 月 7 日 |--- |
| [1.4.0](#1.4.0) |2017 年 8 月 2 日 |--- |
| [1.3.2](#1.3.2) |2017 年 6 月 12 日 |--- |
| [1.3.1](#1.3.1) |2017 年 5 月 23 日 |--- |
| [1.3.0](#1.3.0) |2017 年 5 月 10 日 |--- |
| [1.2.2](#1.2.2) |2017 年 4 月 19 日 |--- |
| [1.2.1](#1.2.1) |2017 年 3 月 29 日 |--- |
| [1.2.0](#1.2.0) |2017 年 3 月 25 日 |--- |
| [1.1.2](#1.1.2) |2017 年 3 月 20 日 |--- |
| [1.1.1](#1.1.1) |2017 年 3 月 14 日 |--- |
| [1.1.0](#1.1.0) |2017 年 2 月 16 日 |--- |
| [1.0.0](#1.0.0) |2016 年 12 月 21 日 |--- |
| [0.1.0-preview](#0.1.0-preview) |2016 年 11 月 15 日 |2016 年 12 月 31 日 |

## <a name="see-also"></a>另请参阅
若要了解有关 Cosmos DB 的详细信息，请参阅 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服务页。

