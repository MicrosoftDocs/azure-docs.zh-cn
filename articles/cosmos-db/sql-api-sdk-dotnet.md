---
title: Azure Cosmos DB：SQL .NET API、SDK 和资源
description: 了解有关 SQL .NET API 和 SDK 的所有信息，包括发布日期、停用日期和 Azure Cosmos DB .NET SDK 各版本之间所做的更改。
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 03/09/2018
ms.author: sngun
ms.openlocfilehash: 3c420882b734883039ec95d609c155617359fa25
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510724"
---
# <a name="azure-cosmos-db-net-sdk-for-sql-api-download-and-release-notes"></a>适用于 SQL API 的 Azure Cosmos DB .NET SDK：下载和发行说明
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
|**SDK 下载**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)|
|**API 文档**|[ 参考文档](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet)|
|**示例**|[.NET代码示例](sql-api-dotnet-samples.md)|
|**入门**|[Azure Cosmos DB .NET SDK 入门](sql-api-get-started.md)|
|**Web 应用教程**|[使用 Azure Cosmos DB 进行 Web 应用程序开发](sql-api-dotnet-application.md)|
|**当前受支持的框架**|[Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)|

## <a name="release-notes"></a>发行说明

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

* 诊断跟踪改进

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

### <a name="a-name12201220"></a><a name="1.22.0"/>1.22.0

* 将 ConsistencyLevel 属性添加到了 FeedOptions。
* 将 JsonSerializerSettings 添加到了 RequestOptions 和 FeedOptions。
* 将 EnableReadRequestsFallback 添加到了 ConnectionPolicy。

### <a name="a-name12111211"></a><a name="1.21.1"/>1.21.1

* 修复了临界情况下跨分区 order by 查询出现的 KeyNotFoundException。
* 修复了在 LINQ 查询的 select 子句中不接受 JsonProperty 属性的 bug。

### <a name="a-name12021202"></a><a name="1.20.2"/>1.20.2

* 修复了在某些争用情况下出现的 bug，该 bug 在使用会话一致性级别时导致间歇性错误“Microsoft.Azure.Documents.NotFoundException:读取会话不可用于输入会话令牌”。

### <a name="a-name12011201"></a><a name="1.20.1"/>1.20.1

* 修复了其中 FeedOptions.MaxItemCount = -1 引发 System.ArithmeticException: 页大小为负的回归。
* 向 QueryMetrics 添加了新的 ToString() 函数。
* 公开了有关读取集合的分区统计信息。
* 向 ChangeFeedOptions 添加了 PartitionKey 属性。
* 小 bug 修复。

### <a name="a-name11911191"></a><a name="1.19.1"/>1.19.1

* 添加了通过使用 DocumentCollection 上的 UniqueKeyPolicy 属性来指定文档唯一索引的功能。
* 修复了自定义 JsonSerializer 设置无法用于某些查询和存储过程执行的 bug。

### <a name="a-name11901190"></a><a name="1.19.0"/>1.19.0

* 在 API 参考文档、程序集中的元数据信息和 NuGet 包中品牌从 Azure DocumentDB 更改为 Azure Cosmos DB。 
* 通过以直接连接模式发送的请求的响应公开诊断信息和延迟。 在 ResourceResponse 类中属性名称是 RequestDiagnosticsString 和 RequestLatency。
* 此 SDK 版本需要最新版本的 Azure Cosmos DB 模拟器（可从 https://aka.ms/cosmosdb-emulator 下载）。 

### <a name="a-name11811181"></a><a name="1.18.1"/>1.18.1 

* 对 Microsoft 友元程序集进行了内部更改。

### <a name="a-name11801180"></a><a name="1.18.0"/>1.18.0 

* 添加了多项可靠性修复和改进。

### <a name="a-name11701170"></a><a name="1.17.0"/>1.17.0 

* 添加了 PartitionKeyRangeId 的支持作为 FeedOption，将查询结果限定在某个特定分区键范围值。 
* 添加了 StartTime 的支持作为 ChangeFeedOption，以开始查找该时间后的更改。

### <a name="a-name11611161"></a><a name="1.16.1"/>1.16.1
* 修复了 JsonSerializable 类中可能引起堆栈溢出异常的问题。

### <a name="a-name11601160"></a><a name="1.16.0"/>1.16.0
*   修复了在因在 DocumentClient 构造函数中引入 JsonSerializerSettings 作为可选参数而需要重新编译的问题。
* 将 DocumentClient 构造函数标记为已过时，需要 JsonSerializerSettings 作为最后一个参数，以在传入 JsonSerializerSettings 参数时允许 ConnectionPolicy 和 ConsistencyLevel 参数的默认值。

### <a name="a-name11501150"></a><a name="1.15.0"/>1.15.0
*   现已开始支持在实例化 [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient?view=azure-dotnet) 时指定自定义 JsonSerializerSettings。

### <a name="a-name11411141"></a><a name="1.14.1"/>1.14.1
*   修复了影响 x64 计算机的一个问题，这些计算机不支持 SSE4 指令，并在运行 Azure Cosmos DB SQL 查询时引发 SEHException。

### <a name="a-name11401140"></a><a name="1.14.0"/>1.14.0
*   添加了对名为 ConsistentPrefix 的新一致性级别的支持。
*   添加了对单独分区的查询指标的支持。
*   添加了对限制查询的继续令牌大小的支持。
*   添加了对失败请求的详情跟踪的支持。
*   改进了 SDK 中的一些性能。

### <a name="a-name11341134"></a><a name="1.13.4"/>1.13.4
* 与 1.13.3 功能相同。 进行了一些内部更改。

### <a name="a-name11331133"></a><a name="1.13.3"/>1.13.3
* 与 1.13.2 功能相同。 进行了一些内部更改。

### <a name="a-name11321132"></a><a name="1.13.2"/>1.13.2
* 修复了忽略 FeedOptions 中为聚合查询提供的 PartitionKey 值的问题。
* 修复了在中途跨分区执行 OrderBy 查询期间透明处理分区管理的问题。

### <a name="a-name11311131"></a><a name="1.13.1"/>1.13.1
* 修复了在 ASP.NET 上下文内使用时，在某些异步 API 中导致死锁的问题。

### <a name="a-name11301130"></a><a name="1.13.0"/>1.13.0
* 修复程序，用于使 SDK 更具弹性，以便在某些情况下自动故障转移。

### <a name="a-name11221122"></a><a name="1.12.2"/>1.12.2
* 修复了偶尔会导致“WebException:无法解析远程名称”的问题。
* 通过向 ReadDocumentAsync API 添加新重载，添加了对直接读取类型化文档的支持。

### <a name="a-name11211121"></a><a name="1.12.1"/>1.12.1
* 添加了对聚合查询（COUNT、MIN、MAX、SUM 和 AVG）的 LINQ 支持。
* 修复了由于使用了事件处理程序而导致的 ConnectionPolicy 对象的内存泄漏问题。
* 修复了使用 ETag 时 UpsertAttachmentAsync 不正常工作的问题。
* 修复了对字符串字段进行排序时跨分区按查询条件排序不正常工作的问题。

### <a name="a-name11201120"></a><a name="1.12.0"/>1.12.0
* 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。 请参阅[聚合支持](how-to-sql-query.md#Aggregates)。
* 将分区集合上的最小吞吐量从 10,100 RU/s 降低到 2500 RU/s。

### <a name="a-name11141114"></a><a name="1.11.4"/>1.11.4
* 针对 32 位主机进程中某些分区查询的失败问题进行了修复。
* 针对网关模式中使用令牌更新会话容器的请求失败问题进行了修复。
* 修复了以下问题：投影中调用 UDF 的查询在某些情况下失败。
* 用于提高请求的读取和写入吞吐量的客户端性能修复。

### <a name="a-name11131113"></a><a name="1.11.3"/>1.11.3
* 针对使用令牌更新会话容器的请求失败问题进行了修复。
* 添加了对 SDK 在 32 位主机进程中工作的支持。 请注意，如果使用跨分区查询，建议使用 64 位主机进程以提高性能。
* 针对在 IN 表达式中使用大量分区键值调用查询的方案改进了性能。
* 在设置 PopulateQuotaInfo 请求时针对文档集合读取请求的 ResourceResponse 中填充了各种资源配额统计信息。

### <a name="a-name11111111"></a><a name="1.11.1"/>1.11.1
* 对 1.11.0 版本中引入的 CreateDocumentCollectionIfNotExistsAsync API 的轻微性能修复。
* 针对 SDK 中涉及高度并发请求的方案的性能修复。

### <a name="a-name11101110"></a><a name="1.11.0"/>1.11.0
* 支持新类和方法，可处理集合内的文档[更改源](change-feed.md)。
* 支持跨分区查询延续和跨分区查询的一些性能改进。
* 添加 CreateDatabaseIfNotExistsAsync 和 CreateDocumentCollectionIfNotExistsAsync 方法。
* 针对以下系统函数提供了 LINQ 支持：IsDefined、IsNull 和 IsPrimitive。
* 修复了将 Nuget 包与具有 project.json 工具的项目搭配使用时，自动将 Microsoft.Azure.Documents.ServiceInterop.dll 和 DocumentDB.Spatial.Sql.dll 程序集自动 binplace 到应用程序的 bin 文件夹中的问题。
* 支持发出客户端侧 ETW 跟踪，这对调试方案很有用。

### <a name="a-name11001100"></a><a name="1.10.0"/>1.10.0
* 添加已分区集合得直接连接支持。
* 改进了 Bounded Staleness 一致性级别的性能。
* 为地域隔离的空间查询指定集合索引策略的同时，已添加多边形和 LineString 数据类型。
* 转换谓词时，添加 StringEnumConverter、IsoDateTimeConverter 和 UnixDateTimeConverter的 LINQ 支持。
* 各种 SDK bug 修复。

### <a name="a-name195195"></a><a name="1.9.5"/>1.9.5
* 修复了导致以下 NotFoundException 的问题：读取会话不可用于输入会话令牌。 在地理分散的帐户的读取区域查询时，在某些情况下会发生此异常。
* 已公开 ResourceResponse 类中的 ResponseStream 属性，使用该属性可直接访问响应中的基础流。

### <a name="a-name194194"></a><a name="1.9.4"/>1.9.4
* 修改了 ResourceResponse、FeedResponse、StoredProcedureResponse 和 MediaResponse 类，以实现相应的公共接口，以便可以模拟它们进行测试驱动的部署 (TDD)。
* 解决了使用自定义 JsonSerializerSettings 对象序列化数据时导致格式错误的分区键标头的问题。

### <a name="a-name193193"></a><a name="1.9.3"/>1.9.3
* 修复了一个问题，该问题导致长时间运行的查询失败并显示以下错误：授权令牌当前无效。
* 解决了从跨分区 top/order-by 查询中删除了原始 SqlParameterCollection 的问题。

### <a name="a-name192192"></a><a name="1.9.2"/>1.9.2
* 已对分区集合添加并行查询支持。
* 已对分区集合添加跨分区 ORDER BY 和 TOP 查询支持。
* 修复了当使用对 Azure Cosmos DB Nuget 包的引用来引用 Azure Cosmos DB 项目时缺少所需的 DocumentDB.Spatial.Sql.dll 和 Microsoft.Azure.Documents.ServiceInterop.dll 引用的错误。
* 已修复在 LINQ 中使用用户定义的函数时无法使用不同类型的参数的错误。 
* 已修复全局复制帐户中的 Bug，此 Bug 会使 Upsert 调用定向至读取位置而非写入位置。
* 已将缺少的方法添加到了 IDocumentClient 接口： 
  * 采用 mediaStream 和选项作为参数的 UpsertAttachmentAsync 方法
  * 采用选项作为参数的 CreateAttachmentAsync 方法
  * 采用 querySpec 作为参数的 CreateOfferQuery 方法。
* 已解封 IDocumentClient 接口中公开的公共类。

### <a name="a-name180180"></a><a name="1.8.0"/>1.8.0
* 添加了对多区域数据库帐户的支持。
* 添加了对重试限制请求的支持。  用户可以通过配置 ConnectionPolicy.RetryOptions 属性来自定义重试次数和最长等待时间。
* 添加了新的 IDocumentClient 接口，用于定义所有 DocumentClient 属性和方法的签名。  在做出此项更改的同时，已用于创建 IQueryable 和 IOrderedQueryable 的扩展方法更改为 DocumentClient 类本身的方法。
* 添加了配置选项，以便设置给定 Azure Cosmos DB 终结点 URI 的 ServicePoint.ConnectionLimit。  使用 ConnectionPolicy.MaxConnectionLimit 可以更改默认值（设置为 50）。
* 已弃用 IPartitionResolver 及其实现。  对 IPartitionResolver 的支持现已过时。 建议使用分区集合来提高存储和吞吐量。

### <a name="a-name171171"></a><a name="1.7.1"/>1.7.1
* 向采用 RequestOptions 作为参数的基于 Uri 的 ExecuteStoredProcedureAsync 方法添加了重载。

### <a name="a-name170170"></a><a name="1.7.0"/>1.7.0
* 对文档添加了生存时间 (TTL) 支持。

### <a name="a-name163163"></a><a name="1.6.3"/>1.6.3
* 修复了用于将其打包为 Azure 云服务解决方案的一部分的 .NET SDK 的 Nuget 包中的 Bug。

### <a name="a-name162162"></a><a name="1.6.2"/>1.6.2
* 实现了[分区集合](partition-data.md)和[用户定义的性能级别](performance-levels.md)。 

### <a name="a-name153153"></a><a name="1.5.3"/>1.5.3
* **[已修复]** 查询 Azure Cosmos DB 终结点引发以下异常：“System.Net.Http.HttpRequestException:将内容复制到流时出错”。

### <a name="a-name152152"></a><a name="1.5.2"/>1.5.2
* 扩展了 LINQ 支持，包括用于分页、条件表达式和范围比较的新运算符。
  * Take 运算符在 LINQ 中启用 SELECT TOP 行为
  * CompareTo 运算符使能够进行字符串范围比较
  * Conditional (?) 和 coalesce 运算符 (??)
* **[已修复]** 合并模型投影与 LINQ 查询中的 Where-In 时引发 ArgumentOutOfRangeException。 [#81](https://github.com/Azure/azure-documentdb-dotnet/issues/81)

### <a name="a-name151151"></a><a name="1.5.1"/>1.5.1
* **[已修复]** 如果 Select 不是最后一个表达式，则 LINQ 提供程序假定没有投影，并且错误地生成 SELECT *。  [#58](https://github.com/Azure/azure-documentdb-dotnet/issues/58)

### <a name="a-name150150"></a><a name="1.5.0"/>1.5.0
* 实现的 Upsert，已添加 UpsertXXXAsync 方法
* 所有请求的性能改进
* 对于字符串，LINQ 提供程序支持条件表达式、联合表达式和 CompareTo 方法
* **[已修复]** LINQ 提供程序 --> 在列表上实现 Contains 方法以生成与 IEnumerable 和 Array 上相同的 SQL
* **[已修复]** BackoffRetryUtility 再次使用相同的 HttpRequestMessage 而不是在重试时创建一个新的
* **[已过时]** UriFactory.CreateCollection --> 现在应使用 UriFactory.CreateDocumentCollection

### <a name="a-name141141"></a><a name="1.4.1"/>1.4.1
* **[已修复]** 使用非英语区域性信息（如 nl-NL 等）时出现的本地化问题。 

### <a name="a-name140140"></a><a name="1.4.0"/>1.4.0
* 添加了基于 ID 的路由
  * 新增用于协助生成基于 ID 的资源链接的 UriFactory 帮助程序
  * DocumentClient 上的新重载以采用 URI
* 已在用于地理空间的 LINQ 中添加了 IsValid() 和 IsValidDetailed()
* 增强了 LINQ 提供程序支持：
  * **数学** - Abs、Acos、Asin、Atan、Ceiling、Cos、Exp、Floor、Log、Log10、Pow、Round、Sign、Sin、Sqrt、Tan、Truncate
  * **字符串** - Concat、Contains、EndsWith、IndexOf、Count、ToLower、TrimStart、Replace、Reverse、TrimEnd、StartsWith、SubString、ToUpper
  * **数组** - Concat、Contains、Count
  * **IN** 运算符

### <a name="a-name130130"></a><a name="1.3.0"/>1.3.0
* 现已开始支持修改索引策略。
  * DocumentClient 中的新 ReplaceDocumentCollectionAsync 方法
  * ResourceResponse 中的新 IndexTransformationProgress 属性<T>，用于跟踪索引策略更改的百分比进度
  * DocumentCollection.IndexingPolicy 现在是可变的
* 现已开始支持空间索引和查询。
  * 用于序列化/反序列化空间类型（如点和多边形）的新 Microsoft.Azure.Documents.Spatial 命名空间
  * 用于索引存储在 Cosmos DB 中 GeoJSON 数据的新 SpatialIndex 类
* **[已修复]** 通过 LINQ 表达式 [#38](https://github.com/Azure/azure-documentdb-net/issues/38) 生成的 SQL 查询不正确。

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0
* 添加了对 Newtonsoft.Json v5.0.7 的依赖关系。
* 更改为支持 OrderBy：
  
  * LINQ 提供程序支持 OrderBy() 或 OrderByDescending() 
  * 支持 Order By 的 IndexingPolicy 
    
    **可能推出的重大更改** 
    
    如果现有代码预配包含自定义索引策略的集合，需要将现有代码更新为支持新的 IndexingPolicy 类。 如果没有任何自定义索引策略，此更改不会影响你。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0
* 现已开始支持使用新的 HashPartitionResolver 和 RangePartitionResolver 类以及 IPartitionResolver 执行数据分区。
* 添加了 DataContract 序列化。
* 在 LINQ 提供程序中添加了 GUID 支持。
* 在 LINQ 中添加了 UDF 支持。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* GA SDK

## <a name="release--retirement-dates"></a>发布和停用日期
Microsoft 至少会在停用 SDK 前提前 12 个月发出通知，以便顺利转换为更高版本/受支持版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议始终尽早升级到最新的 SDK 版本。 

使用已停用的 SDK 对 Azure Cosmos DB 发出的任何请求都会遭服务拒绝。

<br/>

| Version | 发布日期 | 停用日期 |
| --- | --- | --- |
| [2.4.0](#2.4.0) |2019 年 5 月 5， |--- |
| [2.3.0](#2.3.0) |2019 年 4 月 4日日 |--- |
| [2.2.3](#2.2.3) |2019 年 2 月 11日日 |--- |
| [2.2.2](#2.2.2) |2019 年 2 月 6 日 |--- |
| [2.2.1](#2.2.1) |2018 年 12 月 24 日 |--- |
| [2.2.0](#2.2.0) |2018 年 12 月 7 日 |--- |
| [2.1.3](#2.1.3) |2018 年 10 月 15 日 |--- |
| [2.1.2](#2.1.2) |2018 年 10 月 4 日 |--- |
| [2.1.1](#2.1.1) |2018 年 9 月 27 日 |--- |
| [2.1.0](#2.1.0) |2018 年 9 月 21 日 |--- |
| [2.0.0](#2.0.0) |2018 年 9 月 7 日 |--- |
| [1.22.0](#1.22.0) |2018 年 4 月 19 日 |--- |
| [1.21.1](#1.20.1) |2018 年 3 月 9 日 |--- |
| [1.20.2](#1.20.1) |2018 年 2 月 21 日 |--- |
| [1.20.1](#1.20.1) |2018 年 2 月 5 日 |--- |
| [1.19.1](#1.19.1) |2017 年 11 月 16 日 |--- |
| [1.19.0](#1.19.0) |2017 年 11 月 10 日 |--- |
| [1.18.1](#1.18.1) |2017 年 11 月 7 日 |--- |
| [1.18.0](#1.18.0) |2017 年 10 月 17 日 |--- |
| [1.17.0](#1.17.0) |2017 年 8 月 10 日 |--- |
| [1.16.1](#1.16.1) |2017 年 8 月 7 日 |--- |
| [1.16.0](#1.16.0) |2017 年 8 月 2 日 |--- |
| [1.15.0](#1.15.0) |2017 年 6 月 30 日 |--- |
| [1.14.1](#1.14.1) |2017 年 5 月 23 日 |--- |
| [1.14.0](#1.14.0) |2017 年 5 月 10 日 |--- |
| [1.13.4](#1.13.4) |2017 年 5 月 9 日 |--- |
| [1.13.3](#1.13.3) |2017 年 5 月 6 日 |--- |
| [1.13.2](#1.13.2) |2017 年 4 月 19 日 |--- |
| [1.13.1](#1.13.1) |2017 年 3 月 29 日 |--- |
| [1.13.0](#1.13.0) |2017 年 3 月 24 日 |--- |
| [1.12.2](#1.12.2) |2017 年 3 月 20 日 |--- |
| [1.12.1](#1.12.1) |2017 年 3 月 14 日 |--- |
| [1.12.0](#1.12.0) |2017 年 2 月 15 日 |--- |
| [1.11.4](#1.11.4) |2017 年 2 月 6 日 |--- |
| [1.11.3](#1.11.3) |2017 年 1 月 26 日 |--- |
| [1.11.1](#1.11.1) |2016 年 12 月 21 日 |--- |
| [1.11.0](#1.11.0) |2016 年 12 月 08 日 |--- |
| [1.10.0](#1.10.0) |2016 年 9 月 27 日 |--- |
| [1.9.5](#1.9.5) |2016 年 9 月 1 日 |--- |
| [1.9.4](#1.9.4) |2016 年 8 月 24 日 |--- |
| [1.9.3](#1.9.3) |2016 年 8 月 15 日 |--- |
| [1.9.2](#1.9.2) |2016 年 7 月 23 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.1](#1.7.1) |2016 年 5 月 6 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 26 日 |--- |
| [1.6.3](#1.6.3) |2016 年 4 月 8 日 |--- |
| [1.6.2](#1.6.2) |2016 年 3 月 29 日 |--- |
| [1.5.3](#1.5.3) |2016 年 2 月 19 日 |--- |
| [1.5.2](#1.5.2) |2015 年 12 月 14 日 |--- |
| [1.5.1](#1.5.1) |2015 年 11 月 23 日 |--- |
| [1.5.0](#1.5.0) |2015 年 10 月 5 日 |--- |
| [1.4.1](#1.4.1) |2015 年 8 月 25 日 |--- |
| [1.4.0](#1.4.0) |2015 年 8 月 13 日 |--- |
| [1.3.0](#1.3.0) |2015 年 8 月 5 日 |--- |
| [1.2.0](#1.2.0) |2015 年 7 月 6 日 |--- |
| [1.1.0](#1.1.0) |2015 年 4 月 30 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 8 日 |--- |


## <a name="faq"></a>常见问题解答
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另请参阅
若要了解有关 Cosmos DB 的详细信息，请参阅 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服务页。 

