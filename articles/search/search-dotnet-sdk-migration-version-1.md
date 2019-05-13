---
title: 升级到 Azure 搜索 .NET SDK 版本 1.1 - Azure 搜索
description: 从旧 API 版本将代码迁移到 Azure 搜索 .NET SDK 版本 1.1。 了解新增功能和所需的代码更改。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: 3f47656bb13d08ea56cf25a2a29897722abb1cdb
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024170"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-11"></a>升级到 Azure 搜索 .NET SDK 版本 1.1

> [!Important]
> 此内容是仍在构建。 NuGet 上提供了 Azure 搜索.NET SDK 的 9.0 版。 我们正在努力本文说明如何升级到 9.0。 
> 

如果使用的是版本 1.0.2-preview 或更早版本的 [Azure 搜索 .NET SDK](https://aka.ms/search-sdk)，本文有助于升级应用程序，以便使用版本 1.1。

有关包括示例的 SDK 的更多常规演练，请参阅[如何使用 .NET 应用程序中的 Azure 搜索](search-howto-dotnet-sdk.md)。

> [!NOTE]
> 升级到版本 1.1 后，或者已经在使用 1.1 和 2.0 之间的某个版本（含预览版），则应升级到版本 3。 有关说明，请参阅[升级到 Azure 搜索 .NET SDK 版本 3](search-dotnet-sdk-migration.md)。
>

首先，按以下方法操作更新 `Microsoft.Azure.Search` 的 NuGet 引用：使用 NuGet 包管理器控制台，或者在 Visual Studio 中右键单击项目引用，然后选择“管理 NuGet 程序包...”。

在 NuGet 已下载新的程序包及其依赖项后，请重新生成项目。

如果之前使用的是版本 1.0.0-preview、1.0.1-preview 或 1.0.2-preview，生成应该已成功，一切准备就绪！

如果之前使用的是版本 0.13.0-preview 或更早版本，应该会看到如下所示的生成错误：

    Program.cs(137,56,137,62): error CS0117: 'Microsoft.Azure.Search.Models.IndexBatch' does not contain a definition for 'Create'
    Program.cs(137,99,137,105): error CS0117: 'Microsoft.Azure.Search.Models.IndexAction' does not contain a definition for 'Create'
    Program.cs(146,41,146,54): error CS1061: 'Microsoft.Azure.Search.IndexBatchException' does not contain a definition for 'IndexResponse' and no extension method 'IndexResponse' accepting a first argument of type 'Microsoft.Azure.Search.IndexBatchException' could be found (are you missing a using directive or an assembly reference?)
    Program.cs(163,13,163,42): error CS0246: The type or namespace name 'DocumentSearchResponse' could not be found (are you missing a using directive or an assembly reference?)

下一步是逐个修复生成错误。 大多数修复需要更改 SDK 中已重命名的一些类和方法名称。 [版本 1.1 中的重大更改列表](#ListOfChangesV1)含有这些名称更改的列表。

如果使用自定义类对文档建模，并且这些类具有不可为 null 的基元类型的属性（例如，C# 中的 `int` 或 `bool`），应知道 1.1 版的 SDK 中会有 Bug 修复。 有关详细信息，请参阅[版本 1.1 中的 Bug 修复](#BugFixesV1)。

最后，在修复了任何生成错误后，可以对应用程序进行更改，以利用新功能（如果愿意）。

<a name="ListOfChangesV1"></a>

## <a name="list-of-breaking-changes-in-version-11"></a>版本 1.1 中的重大更改列表
以下列表按更改会影响应用程序代码的可能性排序。

### <a name="indexbatch-and-indexaction-changes"></a>IndexBatch 和 IndexAction 更改
`IndexBatch.Create` 已重命名为 `IndexBatch.New`，且不再有 `params` 参数。 可以将 `IndexBatch.New` 用于混用不同类型操作（合并、删除等）的 Batch。 此外，还有可用于创建 Batch（其中的所有操作都相同）的新静态方法：`Delete`、`Merge`、`MergeOrUpload` 和 `Upload`。

`IndexAction` 不再有公共构造函数，并且其属性现已不可变。 应该使用新的静态方法针对不同目的创建操作：`Delete`、`Merge`、`MergeOrUpload` 和 `Upload`。 `IndexAction.Create` 已删除。 如果使用了仅需要一个文档的重载，请务必改为使用 `Upload`。

#### <a name="example"></a>示例
如果代码如下所示：

    var batch = IndexBatch.Create(documents.Select(doc => IndexAction.Create(doc)));
    indexClient.Documents.Index(batch);

若要修复任何生成错误，可以更改为如下所示的代码：

    var batch = IndexBatch.New(documents.Select(doc => IndexAction.Upload(doc)));
    indexClient.Documents.Index(batch);

如有需要，可以进一步简化为如下所示的代码：

    var batch = IndexBatch.Upload(documents);
    indexClient.Documents.Index(batch);

### <a name="indexbatchexception-changes"></a>IndexBatchException 更改
`IndexBatchException.IndexResponse` 属性已重命名为 `IndexingResults`，且其类型现为 `IList<IndexingResult>`。

#### <a name="example"></a>示例
如果代码如下所示：

    catch (IndexBatchException e)
    {
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexResponse.Results.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

若要修复任何生成错误，可以更改为如下所示的代码：

    catch (IndexBatchException e)
    {
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

<a name="OperationMethodChanges"></a>

### <a name="operation-method-changes"></a>操作方法更改
Azure 搜索 .NET SDK 中的每个操作都公开为同步和异步调用方的一组方法重载。 这些方法重载的签名和分解在版本 1.1 中已更改。

例如，较早版本的 SDK 中的“获取索引统计信息”操作公开以下签名：

在 `IIndexOperations` 中：

    // Asynchronous operation with all parameters
    Task<IndexGetStatisticsResponse> GetStatisticsAsync(
        string indexName,
        CancellationToken cancellationToken);

在 `IndexOperationsExtensions` 中：

    // Asynchronous operation with only required parameters
    public static Task<IndexGetStatisticsResponse> GetStatisticsAsync(
        this IIndexOperations operations,
        string indexName);

    // Synchronous operation with only required parameters
    public static IndexGetStatisticsResponse GetStatistics(
        this IIndexOperations operations,
        string indexName);

版本 1.1 中同一操作的方法签名如下所示：

在 `IIndexesOperations` 中：

    // Asynchronous operation with lower-level HTTP features exposed
    Task<AzureOperationResponse<IndexGetStatisticsResult>> GetStatisticsWithHttpMessagesAsync(
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
        Dictionary<string, List<string>> customHeaders = null,
        CancellationToken cancellationToken = default(CancellationToken));

在 `IndexesOperationsExtensions` 中：

    // Simplified asynchronous operation
    public static Task<IndexGetStatisticsResult> GetStatisticsAsync(
        this IIndexesOperations operations,
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
        CancellationToken cancellationToken = default(CancellationToken));

    // Simplified synchronous operation
    public static IndexGetStatisticsResult GetStatistics(
        this IIndexesOperations operations,
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions));

从版本 1.1 开始，Azure 搜索 .NET SDK 以不同方式组织操作方法：

* 可选参数现已建模为默认参数，而不是附加的方法重载。 这有时可显著减少方法重载的数目。
* 扩展方法现在对调用方隐藏了许多与 HTTP 无关的细节。 例如，较早版本的 SDK 返回响应对象时带有 HTTP 状态代码，通常不需要检查这些状态代码，因为操作方法会为指示错误的任何状态代码引发 `CloudException`。 新的扩展方法只返回模型对象，使你无需在代码中对其进行解包。
* 核心接口现在反而公开了允许在 HTTP 级别进行更多控制的方法（如果需要）。 现在可以传入要包括在请求中的自定义 HTTP 标头，并且新的 `AzureOperationResponse<T>` 返回类型使你可以直接访问操作的 `HttpRequestMessage` 和 `HttpResponseMessage`。 `AzureOperationResponse` 在 `Microsoft.Rest.Azure` 命名空间中定义，替换 `Hyak.Common.OperationResponse`。

### <a name="scoringparameters-changes"></a>ScoringParameters 更改
名为 `ScoringParameter` 的新类已添加到最新的 SDK 中，使向搜索查询中的计分配置文件提供参数更为容易。 之前，`SearchParameters` 类的 `ScoringProfiles` 属性以 `IList<string>` 形式键入；现在它以 `IList<ScoringParameter>` 形式键入。

#### <a name="example"></a>示例
如果代码如下所示：

    var sp = new SearchParameters();
    sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
    sp.ScoringParameters = new[] { "featuredParam-featured", "mapCenterParam-" + lon + "," + lat };

若要修复任何生成错误，可以更改为如下所示的代码： 

    var sp = new SearchParameters();
    sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
    sp.ScoringParameters =
        new[]
        {
            new ScoringParameter("featuredParam", new[] { "featured" }),
            new ScoringParameter("mapCenterParam", GeographyPoint.Create(lat, lon))
        };

### <a name="model-class-changes"></a>模型类更改
由于[操作方法更改](#OperationMethodChanges)中描述的签名更改，`Microsoft.Azure.Search.Models` 命名空间中的许多类已重命名或已删除。 例如：

* `IndexDefinitionResponse` 已替换为 `AzureOperationResponse<Index>`
* `DocumentSearchResponse` 已重名为 `DocumentSearchResult`
* `IndexResult` 已重名为 `IndexingResult`
* `Documents.Count()` 现在返回带有文档计数的 `long`，而不是 `DocumentCountResponse`
* `IndexGetStatisticsResponse` 已重名为 `IndexGetStatisticsResult`
* `IndexListResponse` 已重名为 `IndexListResult`

总之，仅用于包裹模型对象的现有 `OperationResponse` 派生类已删除。 其余类已将其后缀从 `Response` 更改为 `Result`。

#### <a name="example"></a>示例
如果代码如下所示：

    IndexerGetStatusResponse statusResponse = null;

    try
    {
        statusResponse = _searchClient.Indexers.GetStatus(indexer.Name);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
        return;
    }

    IndexerExecutionResult lastResult = statusResponse.ExecutionInfo.LastResult;

若要修复任何生成错误，可以更改为如下所示的代码：

    IndexerExecutionInfo status = null;

    try
    {
        status = _searchClient.Indexers.GetStatus(indexer.Name);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
        return;
    }

    IndexerExecutionResult lastResult = status.LastResult;

#### <a name="response-classes-and-ienumerable"></a>响应类和 IEnumerable
可能影响代码的其他更改是：保留集合的响应类不再实现 `IEnumerable<T>`。 相反，可以直接访问集合属性。 例如，如果代码如下所示：

    DocumentSearchResponse<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
    foreach (SearchResult<Hotel> result in response)
    {
        Console.WriteLine(result.Document);
    }

若要修复任何生成错误，可以更改为如下所示的代码：

    DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
    foreach (SearchResult<Hotel> result in response.Results)
    {
        Console.WriteLine(result.Document);
    }

#### <a name="special-case-for-web-applications"></a>Web 应用程序的特例
如果有一个直接序列化 `DocumentSearchResponse` 以向浏览器发送搜索结果的 Web 应用程序，将需要更改代码，否则结果将不会正确序列化。 例如，如果代码如下所示：

    public ActionResult Search(string q = "")
    {
        // If blank search, assume they want to search everything
        if (string.IsNullOrWhiteSpace(q))
            q = "*";

        return new JsonResult
        {
            JsonRequestBehavior = JsonRequestBehavior.AllowGet,
            Data = _featuresSearch.Search(q)
        };
    }

可以通过获取搜索响应的 `.Results` 属性来更改代码，以修复结果呈现：

    public ActionResult Search(string q = "")
    {
        // If blank search, assume they want to search everything
        if (string.IsNullOrWhiteSpace(q))
            q = "*";

        return new JsonResult
        {
            JsonRequestBehavior = JsonRequestBehavior.AllowGet,
            Data = _featuresSearch.Search(q).Results
        };
    }

必须自己在代码中查找此类情况；**编译器不会警告你**，因为 `JsonResult.Data` 属于类型 `object`。

### <a name="cloudexception-changes"></a>CloudException 更改
`CloudException` 类已从 `Hyak.Common` 命名空间移动到 `Microsoft.Rest.Azure` 命名空间。 此外，其 `Error` 属性已重名为 `Body`。

### <a name="searchserviceclient-and-searchindexclient-changes"></a>SearchServiceClient 和 SearchIndexClient 更改
`Credentials` 属性的类型已从 `SearchCredentials` 更改为其基类（即 `ServiceClientCredentials`）。 如果需要访问 `SearchIndexClient` 或 `SearchServiceClient` 的 `SearchCredentials`，请使用新的 `SearchCredentials` 属性。

在早期版本的 SDK 中，`SearchServiceClient` 和 `SearchIndexClient` 具有需要 `HttpClient` 参数的构造函数。 这些已替换为需要 `HttpClientHandler` 和一个 `DelegatingHandler` 对象数组的构造函数。 这使得必要时安装用于预处理 HTTP 请求的自定义处理程序更为容易。

最后，需要 `Uri` 和 `SearchCredentials` 的构造函数已更改。 例如，如果代码如下所示：

    var client =
        new SearchServiceClient(
            new SearchCredentials("abc123"),
            new Uri("http://myservice.search.windows.net"));

若要修复任何生成错误，可以更改为如下所示的代码：

    var client =
        new SearchServiceClient(
            new Uri("http://myservice.search.windows.net"),
            new SearchCredentials("abc123"));

另请注意，凭据参数的类型已更改为 `ServiceClientCredentials`。 由于 `SearchCredentials` 派生自 `ServiceClientCredentials`，所以这不太可能影响代码。

### <a name="passing-a-request-id"></a>传递请求 ID
在早期版本的 SDK 中，可以设置 `SearchServiceClient` 或 `SearchIndexClient` 上的请求 ID，它将包含在对 REST API 的每个请求中。 如果需要与支持人员联系，这对于解决搜索服务的问题非常有用。 不过，为每个操作设置唯一请求 ID 更加有用，而不是将同一 ID 用于所有操作。 出于此原因，`SearchServiceClient` 和 `SearchIndexClient` 的 `SetClientRequestId` 方法已删除。 相反，可以通过可选参数 `SearchRequestOptions` 将请求 ID 传递给每个操作方法。

> [!NOTE]
> 在将来版本的 SDK 中，我们会添加一个新机制，用于在客户端对象上全局设置请求 ID，这与其他 Azure SDK 使用的方法一致。
> 
> 

### <a name="example"></a>示例
如果代码如下所示：

    client.SetClientRequestId(Guid.NewGuid());
    ...
    long count = client.Documents.Count();

若要修复任何生成错误，可以更改为如下所示的代码：

    long count = client.Documents.Count(new SearchRequestOptions(requestId: Guid.NewGuid()));

### <a name="interface-name-changes"></a>接口名称更改
操作组接口名称已全部更改，与其相应的属性名称保持一致：

* `ISearchServiceClient.Indexes` 的类型已从 `IIndexOperations` 更改为 `IIndexesOperations`。
* `ISearchServiceClient.Indexers` 的类型已从 `IIndexerOperations` 更改为 `IIndexersOperations`。
* `ISearchServiceClient.DataSources` 的类型已从 `IDataSourceOperations` 更改为 `IDataSourcesOperations`。
* `ISearchIndexClient.Documents` 的类型已从 `IDocumentOperations` 更改为 `IDocumentsOperations`。

此更改不太可能影响代码，除非出于测试目的创建了这些接口的模拟。

<a name="BugFixesV1"></a>

## <a name="bug-fixes-in-version-11"></a>版本 1.1 中的 Bug 修复
与自定义模型类的序列化有关的早期版本的 Azure 搜索 .NET SDK 中存在 Bug。 如果使用不可为 null 的值类型的属性创建了自定义模型类，可能会出现 Bug。

### <a name="steps-to-reproduce"></a>重现步骤
使用不可为 null 的值类型的属性创建自定义模型类。 例如，添加类型为 `int`（而不是 `int?`）的公共 `UnitCount` 属性。

如果使用该类型的默认值（例如，`int` 的 0）对文档编制索引，该字段在 Azure 搜索将为 null。 如果随后搜索该文档，`Search` 调用会引发 `JsonSerializationException`，声称无法将 `null` 转换为 `int`。

此外，筛选器可能不会按预期工作，因为 null 已写入索引，而不是预期值。

### <a name="fix-details"></a>修复详细信息
我们修复了版本 1.1 的 SDK 中存在的此问题。 现在，如果模型类如下所示：

    public class Model
    {
        public string Key { get; set; }

        public int IntValue { get; set; }
    }

将 `IntValue` 设置为 0，该值现在线路上正确序列化为 0，并在索引中存储为 0。 往返过程也按预期工作。

使用此方法时有一个潜在的问题需要注意：如果将模型类型与不可为 null 的属性一起使用，必须**保证**索引中的所有文档的对应字段都不包含 null 值。 该 SDK 和 Azure 搜索 REST API 都不会帮助强制实施此检查。

这不只是假想的问题：假设将新字段添加到 `Edm.Int32` 类型的现有索引。 更新索引定义后，所有文档的该新字段都具有 null 值（因为 Azure 搜索中的所有类型都可以为 null）。 如果随后使用该字段具有不可为 null `int` 属性的模型类，则在尝试检索文档时会获得如下所示的 `JsonSerializationException`：

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

由于此原因，最佳做法仍建议在模型类中使用可以为 null 的类型。

有关此 Bug 和修复的更多详细信息，请参阅 [GitHub 上的此问题](https://github.com/Azure/azure-sdk-for-net/issues/1063)。

