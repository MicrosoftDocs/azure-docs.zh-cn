---
title: 适用于 Async Java 的 Azure Cosmos DB 性能提示
description: 了解用于提高 Azure Cosmos DB 数据库性能的客户端配置选项
author: SnehaGunda
ms.service: cosmos-db
ms.devlang: java
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: sngun
ms.openlocfilehash: 0da5f503c13ce03854964c0d18edd0031e25bb62
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235600"
---
# <a name="performance-tips-for-azure-cosmos-db-and-async-java"></a>适用于 Azure Cosmos DB 和 Async Java 的性能提示

> [!div class="op_single_selector"]
> * [异步 Java](performance-tips-async-java.md)
> * [Java](performance-tips-java.md)
> * [.NET](performance-tips.md)
> 

Azure Cosmos DB 是一个快速、弹性的分布式数据库，可以在提供延迟与吞吐量保证的情况下无缝缩放。 凭借 Azure Cosmos DB，无需对体系结构进行重大更改或编写复杂的代码即可缩放数据库。 扩展和缩减操作就像执行单个 API 调用或 SDK 方法调用一样简单。 但是，由于 Azure Cosmos DB 是通过网络调用访问的，因此，使用 [SQL Async Java SDK](sql-api-sdk-async-java.md) 时，可以通过客户端优化来获得最高性能。

如果有“如何改善数据库性能？”的疑问， 请考虑以下选项：

## <a name="networking"></a>网络
   <a id="same-region"></a>
1. **性能的（位于相同的 Azure 区域内）并置客户端**

    如果可能，请将任何调用 Azure Cosmos DB 的应用程序放在与 Azure Cosmos DB 数据库所在的相同区域中。 通过大致的比较发现，在同一区域中对 Azure Cosmos DB 的调用可在 1-2 毫秒内完成，而美国西海岸和美国东海岸之间的延迟则大于 50 毫秒。 根据请求采用的路由，各项请求从客户端传递到 Azure 数据中心边界时的此类延迟可能有所不同。 通过确保在与预配 Azure Cosmos DB 终结点所在的同一 Azure 区域中调用应用程序，可能会实现最低的延迟。 有关可用区域的列表，请参阅[ Azure Regions（Azure 区域）](https://azure.microsoft.com/regions/#services)。

    ![Azure Cosmos DB 连接策略演示](./media/performance-tips/same-region.png)

## <a name="sdk-usage"></a>SDK 用法
1. **安装最新的 SDK**

    Azure Cosmos DB SDK 正在不断改进以提供最佳性能。 请参阅 [Azure Cosmos DB SDK](sql-api-sdk-async-java.md) 页以了解最新的 SDK 并查看改进内容。
2. **在应用程序生存期内使用单一实例 Azure Cosmos DB 客户端**

    每个 AsyncDocumentClient 实例都是线程安全的，可执行高效的连接管理和地址缓存。 若要通过 AsyncDocumentClient 获得高效的连接管理和更好的性能，建议在应用程序生存期内对每个 AppDomain 使用单个 AsyncDocumentClient 实例。

   <a id="max-connection"></a>

3. **优化 ConnectionPolicy**

    使用 Async Java SDK 时，Azure Cosmos DB 请求是通过 HTTPS/REST 发出的，并且受制于默认的最大连接池大小 (1000)。 此默认值对于大多数用例是很理想的。 但是，如果你有一个包含许多分区的大型集合，则可以使用 setMaxPoolSize 将最大连接池大小设置为更大的数字（例如 1500）。

4. **优化分区集合的并行查询。**

    Azure Cosmos DB SQL 异步 Java SDK 支持并行查询，使你能够并行查询分区集合。 有关详细信息，请参阅与使用这些 SDK 相关的[代码示例](https://github.com/Azure/azure-cosmosdb-java/tree/master/examples/src/test/java/com/microsoft/azure/cosmosdb/rx/examples)。 并行查询旨改善查询延迟和串行配对物上的吞吐量。

    (a) ***优化 setMaxDegreeOfParallelism\:*** 并行查询的方式是并行查询多个分区。 但就查询本身而言，会按顺序提取单个已分区集合中的数据。 因此，通过使用 setMaxDegreeOfParallelism 设置分区数，最有可能实现查询的最高性能，但前提是所有其他系统条件仍保持不变。 如果不知道分区数，可使用 setMaxDegreeOfParallelism 设置一个较高的数值，系统会选择最小值（分区数、用户输入）作为最大并行度。

    请务必注意：如果数据能均匀地分散在与查询相关的所有分区上，并行查询就能带来最大的好处。 如果对分区集合进行分区，其中全部或大部分查询所返回的数据集中于几个分区（最坏的情况下为一个分区），则这些分区将遇到查询的性能瓶颈。

    (b) ***优化 setMaxBufferedItemCount\:*** 并行查询专用于在客户端处理当前一批结果时预提取结果。 预提取帮助改进查询中的的总体延迟。 setMaxBufferedItemCount 会限制预提取结果的数目。 通过将 setMaxBufferedItemCount 设置为预期返回的结果数（或较高的数值），可使查询从预提取获得最大的好处。

    预提取的工作方式不因 MaxDegreeOfParallelism 而异，并且有一个单独的缓冲区用来存储所有分区的数据。

5. **按 getRetryAfterInMilliseconds 间隔实现回退**

    在性能测试期间，应该增加负载，直到系统对小部分请求进行限制为止。 如果受到限制，客户端应用程序应按照服务器指定的重试间隔退让。 遵循退让可确保最大程度地减少等待重试的时间。
6. **增大客户端工作负荷**

    如果以高吞吐量级别（> 50,000 RU/s）进行测试，客户端应用程序可能成为瓶颈，因为计算机的 CPU 或网络利用率将达到上限。 如果达到此限制，可以跨多个服务器横向扩展客户端应用程序，以进一步推送 Azure Cosmos DB 帐户。

7. **使用基于名称的寻址**

    使用基于名称的寻址，其中的链接格式为 `dbs/MyDatabaseId/colls/MyCollectionId/docs/MyDocumentId`，而不是使用格式为 `dbs/<database_rid>/colls/<collection_rid>/docs/<document_rid>` 的 SelfLinks (\_self)（旨在避免检索用于构造链接的所有资源的 ResourceId）。 此外，由于会重新创建这些资源（名称可能相同），因此，缓存这些资源的用处不大。

   <a id="tune-page-size"></a>
8. **调整查询/读取源的页面大小以获得更好的性能**

    使用读取源功能（例如 readDocuments）执行批量文档读取时，或发出 SQL 查询时，如果结果集太大，则以分段方式返回结果。 默认情况下，以包括 100 个项的块或 1 MB 大小的块返回结果（以先达到的限制为准）。

    若要减少检索所有适用结果所需的网络往返次数，可以使用 [x-ms-max-item-count](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 请求标头将页面大小最大增加到 1000。 在只需要显示几个结果的情况下（例如，用户界面或应用程序 API 一次只返回 10 个结果），也可以将页面大小缩小为 10，以降低读取和查询所耗用的吞吐量。

    也可以使用 setMaxItemCount 方法设置页面大小。

9. **使用相应的计划程序（避免窃取事件循环 IO Netty 线程）**

    Async Java SDK 对非阻塞 IO使用 [netty](https://netty.io/)。 SDK 使用固定数量的 IO netty 事件循环线程（数量与计算机提供的 CPU 核心数相同）来执行 IO 操作。 API 返回的可观测对象针对某个共享的 IO 事件循环 netty 线程发出结果。 因此，切勿阻塞共享的 IO 事件循环 netty 线程。 针对 IO 事件循环 netty 线程执行 CPU 密集型工作或者阻塞操作可能导致死锁，或大大减少 SDK 吞吐量。

    例如，以下代码针对事件循环 IO netty 线程执行 CPU 密集型工作：

    ```java
    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribe(
      resourceResponse -> {
        //this is executed on eventloop IO netty thread.
        //the eventloop thread is shared and is meant to return back quickly.
        //
        // DON'T do this on eventloop IO netty thread.
        veryCpuIntensiveWork();
      });
    ```

    收到结果后，如果想要针对结果执行 CPU 密集型工作，应避免针对事件循环 IO netty 线程执行。 可以提供自己的计划程序，以提供自己的线程来运行工作。

    ```java
    import rx.schedulers;

    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribeOn(Schedulers.computation())
    subscribe(
      resourceResponse -> {
        // this is executed on threads provided by Scheduler.computation()
        // Schedulers.computation() should be used only when:
        //   1. The work is cpu intensive 
        //   2. You are not doing blocking IO, thread sleep, etc. in this thread against other resources.
        veryCpuIntensiveWork();
      });
    ```

    根据工作的类型，应该使用相应的现有 RxJava 计划程序来执行工作。 请阅读 [``Schedulers``](http://reactivex.io/RxJava/1.x/javadoc/rx/schedulers/Schedulers.html)。

    有关详细信息，请查看 Async Java SDK 的 [GitHub 页](https://github.com/Azure/azure-cosmosdb-java)。

10. **禁用 netty 日志记录** Netty 库日志记录非常琐碎，因此需要将其关闭（在配置中禁止登录可能并不足够），以避免产生额外的 CPU 开销。 如果不处于调试模式，请一起禁用 netty 日志记录。 因此，如果要使用 log4j 来消除 netty 中 ``org.apache.log4j.Category.callAppenders()`` 产生的额外 CPU 开销，请将以下行添加到基代码：

    ```java
    org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
    ```

11. **OS 打开文件资源限制**某些 Linux 系统（例如 Red Hat）对打开的文件数和连接总数施加上限。 运行以下命令以查看当前限制：

    ```bash
    ulimit -a
    ```

    打开的文件数 (nofile) 需要足够大，以便为配置的连接池大小和 OS 打开的其他文件留出足够的空间。 可以修改此参数，以增大连接池大小。

    打开 limits.conf 文件：

    ```bash
    vim /etc/security/limits.conf
    ```
    
    添加/修改以下行：

    ```
    * - nofile 100000
    ```

12. **对 netty 使用本机 SSL 实现**Netty 可以直接对 SSL 实现堆栈使用 OpenSSL，以实现更好的性能。 如果没有此配置，netty 将回退到 Java 的默认 SSL 实现。

    在 Ubuntu 上：
    ```bash
    sudo apt-get install openssl
    sudo apt-get install libapr1
    ```

    并将以下依赖项添加到项目的 maven 依赖项：
    ```xml
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-tcnative</artifactId>
      <version>2.0.20.Final</version>
      <classifier>linux-x86_64</classifier>
    </dependency>
    ```

对于其他平台（Red Hat、Windows、Mac 等），请参考 https://netty.io/wiki/forked-tomcat-native.html 中的说明

## <a name="indexing-policy"></a>索引策略
 
1. **从索引中排除未使用的路径以加快写入速度**

    Azure Cosmos DB 的索引策略允许使用索引路径（setIncludedPaths 和 setExcludedPaths）指定要在索引中包括或排除的文档路径。 在事先知道查询模式的方案中，使用索引路径可改善写入性能并降低索引存储空间，因为索引成本与索引的唯一路径数目直接相关。 例如，以下代码演示了如何使用“*”通配符 从索引中排除文档的整个部分（也称为子树）。

    ```Java
    Index numberIndex = Index.Range(DataType.Number);
    numberIndex.set("precision", -1);
    indexes.add(numberIndex);
    includedPath.setIndexes(indexes);
    includedPaths.add(includedPath);
    indexingPolicy.setIncludedPaths(includedPaths);
    collectionDefinition.setIndexingPolicy(indexingPolicy);
    ```

    有关详细信息，请参阅 [Azure Cosmos DB 索引策略](indexing-policies.md)。

## <a name="throughput"></a>吞吐量
<a id="measure-rus"></a>

1. **测量和优化较低的每秒请求单位使用量**

    Azure Cosmos DB 提供一组丰富的数据库操作，包括 UDF 的关系和层次查询、存储过程和触发 – 所有都在数据库集合的文档上操作。 与这些操作关联的成本取决于完成操作所需的 CPU、IO 和内存。 与考虑和管理硬件资源不同的是，可以考虑将请求单位 (RU) 作为所需资源的单个措施，以执行各种数据库操作和服务应用程序请求。

    吞吐量是基于为每个容器设置的[请求单位](request-units.md)数量预配的。 请求单位消耗以每秒速率评估。 如果应用程序的速率超过了为其容器预配的请求单位速率，则会受到限制，直到该速率降到容器的预配级别以下。 如果应用程序需要较高级别的吞吐量，可以通过预配更多请求单位来增加吞吐量。

    查询的复杂性会影响操作使用的请求单位数量。 谓词数、谓词性质、UDF 数目和源数据集的大小都会影响查询操作的成本。

    若要测量任何操作（创建、更新或删除）的开销，请检查 [x-ms-request-charge](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 标头来测量这些操作占用的请求单位数。 也可以在 ResourceResponse<T> 或 FeedResponse<T> 中找到等效的 RequestCharge 属性。

    ```Java
    ResourceResponse<Document> response = asyncClient.createDocument(collectionLink, documentDefinition, null,
                                                     false).toBlocking.single();
    response.getRequestCharge();
    ```

    在此标头中返回的请求费用是预配吞吐量的一小部分。 例如，如果预配了 2000 RU/s，上述查询返回 1000 个 1KB 文档，则操作成本为 1000。 因此在一秒内，服务器在对后续请求进行速率限制之前，只接受两个此类请求。 有关详细信息，请参阅[请求单位](request-units.md)和[请求单位计算器](https://www.documentdb.com/capacityplanner)。
<a id="429"></a>
2. **处理速率限制/请求速率太大**

    当客户端尝试超过帐户保留的吞吐量时，服务器的性能不会降低，并且不使用超过保留级别的吞吐量容量。 服务器将抢先结束 RequestRateTooLarge（HTTP 状态代码 429）的请求并返回 [x-ms-retry-after-ms](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 标头，该标头指示重新尝试请求前用户必须等待的时间量（以毫秒为单位）。

        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK 全部都会隐式捕获此响应，并遵循服务器指定的 retry-after 标头，并重试请求。 除非多个客户端同时访问帐户，否则下次重试就会成功。

    如果存在多个高于请求速率的请求操作，则客户端当前在内部设置为 9 的默认重试计数可能无法满足需要；在此情况下，客户端就会向应用程序引发 DocumentClientException，其状态代码为 429。 可以通过在 ConnectionPolicy 实例上使用 setRetryOptions 来更改默认重试计数。 默认情况下，如果请求继续以高于请求速率的方式运行，则在 30 秒的累积等待时间后将返回 DocumentClientException 和状态代码 429。 即使当前的重试计数小于最大重试计数（默认值 9 或用户定义的值），也会发生这种情况。

    尽管自动重试行为有助于改善大多数应用程序的复原能力和可用性，但是在执行性能基准测试时可能会造成冲突（尤其是在测量延迟时）。 如果实验达到服务器限制并导致客户端 SDK 静默重试，则客户端观测到的延迟会剧增。 若要避免性能实验期间出现延迟高峰，可以测量每个操作返回的费用，并确保请求以低于保留请求速率的方式运行。 有关详细信息，请参阅[请求单位](request-units.md)。
3. **针对小型文档进行设计以提高吞吐量**

    给定操作的请求费用（请求处理成本）与文档大小直接相关。 大型文档的操作成本高于小型文档的操作成本。

## <a name="next-steps"></a>后续步骤
若要深入了解如何设计应用程序以实现缩放和高性能，请参阅 [Azure Cosmos DB 中的分区和缩放](partition-data.md)。
