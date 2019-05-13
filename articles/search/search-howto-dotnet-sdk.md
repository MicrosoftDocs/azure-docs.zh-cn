---
title: 如何在 .NET 应用程序中使用 Azure 搜索 - Azure 搜索
description: 了解如何使用 C# 和 .NET SDK 在 .NET 应用程序中使用 Azure 搜索。 基于代码的任务包括连接到服务，为内容编制索引，以及查询索引。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.openlocfilehash: 25a156c4403b7a89f7a7bf7f6acf22fa34216791
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025133"
---
# <a name="how-to-use-azure-search-from-a-net-application"></a>如何使用 .NET 应用程序中的 Azure 搜索

> [!Important]
> 此内容是仍在构建。 NuGet 上提供了 Azure 搜索.NET SDK 的 9.0 版。 我们正在努力更新本迁移指南说明如何升级到最新版本。 请继续关注。
>

文本介绍了如何使用 [Azure 搜索 .NET SDK](https://aka.ms/search-sdk)。 可以使用 .NET SDK，在应用程序中使用 Azure 搜索实现丰富的搜索体验。

## <a name="whats-in-the-azure-search-sdk"></a>什么是 Azure 搜索 SDK
SDK 包括一些客户端库。借助它，不仅可以管理索引、数据源、索引器和同义词映射，还能上传和管理文档并执行查询，所有这些操作都无需处理 HTTP 和 JSON 的详细信息。 这些客户端库全部作为 NuGet 包进行分发。

主 NuGet 包是 `Microsoft.Azure.Search`，它是一个元包，包括所有作为依赖关系的其他程序包。 如果你刚入门，或者如果你知道应用程序将需要 Azure 搜索的所有功能，请使用此程序包。

SDK 中的其他 NuGet 程序包有：
 
  - `Microsoft.Azure.Search.Data`：如果使用 Azure 搜索开发 .NET 应用程序，则使用此包，并且只需查询或更新索引中的文档。 如果还需要创建或更新索引、同义词映射或其他服务级资源，请改用 `Microsoft.Azure.Search` 包。
  - `Microsoft.Azure.Search.Service`：如果在 .NET 中开发自动化以管理 Azure 搜索索引、同义词映射、索引器、数据源或其他服务级资源，请使用此包。 如果只需要查询或更新索引中的文档，请改用 `Microsoft.Azure.Search.Data` 包。 如果需要 Azure 搜索的所有功能，请改用 `Microsoft.Azure.Search` 包。
  - `Microsoft.Azure.Search.Common`：Azure 搜索 .NET 库需要的常见类型。 不应在应用程序中直接使用此包；它只是用作依赖项。

各种客户端库定义 `Index`、`Field` 和 `Document` 等类，以及 `SearchServiceClient` 和 `SearchIndexClient` 类中的 `Indexes.Create` 和 `Documents.Search` 等操作。 这些类已组织成以下命名空间：

* [Microsoft.Azure.Search](https://docs.microsoft.com/dotnet/api/microsoft.azure.search)
* [Microsoft.Azure.Search.Models](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models)

Azure 搜索 .NET SDK 的当前版本现已正式发布。 如果想要提供反馈，供我们包含到下一个版本中，请访问我们的[反馈页](https://feedback.azure.com/forums/263029-azure-search/)。

.NET SDK 支持版本 `2017-11-11` 的 [Azure 搜索 REST API](https://docs.microsoft.com/rest/api/searchservice/)。 该版本现在包括对同义词的支持，以及对索引器的逐步改进。 

此 SDK 不支持[管理操作](https://docs.microsoft.com/rest/api/searchmanagement/)（如创建和缩放搜索服务以及管理 API 密钥）。 如果需要从 .NET 应用程序管理搜索资源，可以使用 [Azure 搜索 .NET 管理 SDK](https://aka.ms/search-mgmt-sdk)。

## <a name="upgrading-to-the-latest-version-of-the-sdk"></a>升级到最新版本的 SDK
如果已在使用较旧版本的 Azure 搜索 .NET SDK，并且想要升级到最新的正式发布版本，[文本](search-dotnet-sdk-migration-version-5.md)介绍了操作方法。

## <a name="requirements-for-the-sdk"></a>SDK 的要求
1. Visual Studio 2017。
2. 自己的 Azure 搜索服务。 要使用 SDK，需要服务的名称以及一个或多个 API 密钥。 [在门户中创建服务](search-create-service-portal.md)将帮助你完成这些步骤。
3. 在 Visual Studio 中，通过使用“管理 NuGet 程序包”来下载 Azure 搜索 .NET SDK [NuGet 程序包](https://www.nuget.org/packages/Microsoft.Azure.Search)。 只需在 NuGet.org 上搜索程序包名称 `Microsoft.Azure.Search`（或者如果你只需要其中一部分功能，则可以搜索上述其中一个其他程序包名称）。

Azure 搜索 .NET SDK 支持面向 .NET Framework 4.5.2 及更高版本，以及 .NET Core 的应用程序。

## <a name="core-scenarios"></a>核心方案
需要在搜索应用程序中完成几项操作。 在本教程中，我们介绍以下核心方案：

* 创建索引
* 使用文档填充索引
* 使用全文搜索和筛选器搜索文档

下面的示例代码说明了其中每项功能。 可随意在自己的应用程序中使用代码片段。

### <a name="overview"></a>概述
我们将要探索的示例应用程序会创建一个名为“hotels”的新索引、用几个文档对该索引进行填充，并执行一些搜索查询。 以下是主程序，演示总体流程：

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteHotelsIndexIfExists(serviceClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateHotelsIndex(serviceClient);

    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);

    ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

> [!NOTE]
> 可以在 [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) 上找到本演练中所用示例应用程序的完整源代码。
> 
>

我们将逐步进行介绍。 首先，我们需要创建一个新的 `SearchServiceClient`。 使用此对象，可以管理索引。 要构建一个，需要提供 Azure 搜索服务名称以及管理 API 密钥。 可以在[示例应用程序](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)的 `appsettings.json` 文件中输入此信息。

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

> [!NOTE]
> 如果提供了不正确的密钥（例如，需要管理密钥的查询密钥），首次调用 `Indexes.Create` 之类上的操作方法时，`SearchServiceClient` 会引发 `CloudException`（错误消息为“已禁用”）。 如果遇到此情况，请仔细检查我们的 API 密钥。
> 
> 

以下几行调用方法来创建名为“hotels”的索引，如果该索引已存在，请事先删除它。 我们稍后会介绍这些方法。

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteHotelsIndexIfExists(serviceClient);

Console.WriteLine("{0}", "Creating index...\n");
CreateHotelsIndex(serviceClient);
```

接下来，需要填充索引。 为此，我们需要 `SearchIndexClient`。 有两种方法可获取一个：构建它，或调用 `SearchServiceClient` 上的 `Indexes.GetClient`。 为方便起见，我们使用后者。

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

> [!NOTE]
> 在典型的搜索应用程序中，索引管理和填充由搜索查询中的一个单独的组件处理。 `Indexes.GetClient` 对于填充索引很方便，因为使用它则无需提供另一个 `SearchCredentials`。 它通过向新 `SearchIndexClient` 传递用于创建 `SearchServiceClient` 的管理密钥来实现此目的。 但是，在执行查询的应用程序中，最好直接创建 `SearchIndexClient` ，这样可以传入查询密钥而不是管理密钥。 这与最小特权原则一致，可帮助使应用程序更安全。 可在[此处](https://docs.microsoft.com/rest/api/searchservice/#authentication-and-authorization)了解有关管理密钥和查询密钥的详细信息。
> 
> 

现在，我们已有 `SearchIndexClient`，可填充索引。 执行此操作使用的是我们稍后会介绍的另一种方法。

```csharp
Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(indexClient);
```

最后，我们执行一些搜索查询，并显示结果。 这次我们使用不同的 `SearchIndexClient`：

```csharp
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

RunQueries(indexClientForQueries);
```

我们会在以后详细地查看 `RunQueries` 方法。 下面是用于创建新 `SearchIndexClient` 的代码：

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "hotels", new SearchCredentials(queryApiKey));
    return indexClient;
}
```

这次我们使用查询密钥，因为我们不需要对索引进行写访问。 可以在[示例应用程序](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)的 `appsettings.json` 文件中输入此信息。

如果使用有效服务名称和 API 密钥运行此应用程序，输出应如下所示：

    Deleting index...
    
    Creating index...
    
    Uploading documents...
    
    Waiting for documents to be indexed...
    
    Search the entire index for the term 'budget' and return only the hotelName field:
    
    Name: Roach Motel
    
    Apply a filter to the index to find hotels cheaper than $150 per night, and return the hotelId and description:
    
    ID: 2   Description: Cheapest hotel in town
    ID: 3   Description: Close to town hall and the river
    
    Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:
    
    Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
    Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00
    
    Search the entire index for the term 'motel':
    
    ID: 2   Base rate: 79.99        Description: Cheapest hotel in town     Description (French): Hôtel le moins cher en ville      Name: Roach Motel       Category: Budget        Tags: [motel, budget]   Parking included: yes   Smoking allowed: yes    Last renovated on: 4/28/1982 12:00:00 AM +00:00 Rating: 1/5     Location: Latitude 49.678581, longitude -122.131577
    
    Complete.  Press any key to end application...

本文末尾处提供了应用程序的完整源代码。

接下来，我们会详细介绍 `Main` 调用的每个方法。

### <a name="creating-an-index"></a>创建索引
在创建后`SearchServiceClient`，`Main`删除"hotels"索引，如果它已存在。 使用以下方法完成此操作：

```csharp
private static void DeleteHotelsIndexIfExists(SearchServiceClient serviceClient)
{
    if (serviceClient.Indexes.Exists("hotels"))
    {
        serviceClient.Indexes.Delete("hotels");
    }
}
```

此方法使用给定的 `SearchServiceClient` 来检查索引是否存在，如果存在，则删除该索引。

> [!NOTE]
> 为简单起见，本文中的示例代码使用 Azure 搜索 .NET SDK 的同步方法。 建议用户在自己的应用程序中使用异步方法，使应用程序保持可缩放且响应迅速。 例如，在上述方法中，可以使用 `ExistsAsync` 和 `DeleteAsync`，而不是 `Exists` 和 `Delete`。
> 
> 

接下来，`Main` 通过调用如下方法创建新的“hotels”索引：

```csharp
private static void CreateHotelsIndex(SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = "hotels",
        Fields = FieldBuilder.BuildForType<Hotel>()
    };

    serviceClient.Indexes.Create(definition);
}
```

此方法使用用于定义新索引架构的一列 `Field` 对象创建新的 `Index` 对象。 每个字段都有名称、数据类型和数个属性（定义其搜索行为）。 `FieldBuilder` 类通过检查给定 `Hotel` 模型类的公共属性和特性，使用反射来为索引创建 `Field` 对象的列表。 我们会在以后详细地查看 `Hotel` 类。

> [!NOTE]
> 如果需要，始终可以直接创建 `Field` 对象的列表，而不是使用 `FieldBuilder`。 例如，你可能不想使用模型类，或者可能需要使用不希望通过添加属性来进行修改的现有模型类。
>
> 

除了字段，还可以向索引添加计分配置文件、建议器或 CORS 选项（为简洁起见，示例中省略了这些项）。 可在 [SDK 参考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index)以及 [Azure 搜索 REST API 参考](https://docs.microsoft.com/rest/api/searchservice/)中，找到有关 Index 对象及其组成部分的详细信息。

### <a name="populating-the-index"></a>填充索引
`Main` 中的下一步是填充新创建的索引。 在以下方法中完成此操作：

```csharp
private static void UploadDocuments(ISearchIndexClient indexClient)
{
    var hotels = new Hotel[]
    {
        new Hotel()
        { 
            HotelId = "1", 
            BaseRate = 199.0, 
            Description = "Best hotel in town",
            DescriptionFr = "Meilleur hôtel en ville",
            HotelName = "Fancy Stay",
            Category = "Luxury", 
            Tags = new[] { "pool", "view", "wifi", "concierge" },
            ParkingIncluded = false, 
            SmokingAllowed = false,
            LastRenovationDate = new DateTimeOffset(2010, 6, 27, 0, 0, 0, TimeSpan.Zero), 
            Rating = 5, 
            Location = GeographyPoint.Create(47.678581, -122.131577)
        },
        new Hotel()
        { 
            HotelId = "2", 
            BaseRate = 79.99,
            Description = "Cheapest hotel in town",
            DescriptionFr = "Hôtel le moins cher en ville",
            HotelName = "Roach Motel",
            Category = "Budget",
            Tags = new[] { "motel", "budget" },
            ParkingIncluded = true,
            SmokingAllowed = true,
            LastRenovationDate = new DateTimeOffset(1982, 4, 28, 0, 0, 0, TimeSpan.Zero),
            Rating = 1,
            Location = GeographyPoint.Create(49.678581, -122.131577)
        },
        new Hotel() 
        { 
            HotelId = "3", 
            BaseRate = 129.99,
            Description = "Close to town hall and the river"
        }
    };

    var batch = IndexBatch.Upload(hotels);

    try
    {
        indexClient.Documents.Index(batch);
    }
    catch (IndexBatchException e)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
}
```

此方法有四个部分。 第一部分创建的一个 `Hotel` 对象数组，将用作我们上传到索引的输入数据。 为简单起见，此数据是硬编码的。 在自己的应用程序中，数据可能来自外部数据源（如 SQL 数据库）。

第二部分创建包含文档的 `IndexBatch`。 创建 Batch 时，指定要应用到 Batch 的操作，在这种情况下应调用 `IndexBatch.Upload`。 然后，使用 `Documents.Index` 方法将 Batch 上传到 Azure 搜索索引。

> [!NOTE]
> 在本示例中，我们只需上传文档。 如果想要将更改合并到现有文档或删除文档，可以改为调用 `IndexBatch.Merge`、`IndexBatch.MergeOrUpload` 或 `IndexBatch.Delete` 创建 Batch。 还可以通过调用 `IndexBatch.New` 在单个 Batch 中混用不同操作，这会用到一个 `IndexAction` 对象的集合，其中的每个对象都会指示 Azure 搜索对文档执行特定操作。 可以通过调用 `IndexAction.Merge`、`IndexAction.Upload` 之类的相应方法，创建自带操作的每个 `IndexAction`。
> 
> 

此方法的第三部分是处理索引重要错误情况的 catch 块。 如果 Azure 搜索服务无法为 Batch 中的某些文档编制索引，`Documents.Index` 将引发 `IndexBatchException`。 如果在服务负载过大时为文档编制索引，可能会发生这种情况。 **强烈建议在代码中显式处理这种情况。**  可以延迟为失败的文档编制索引，并重试，也可以像此示例一样记录并继续执行，还可以执行其他操作，具体取决于应用程序对数据一致性的要求。

> [!NOTE]
> 可以使用 [`FindFailedActionsToRetry`](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.indexbatchexception.findfailedactionstoretry) 方法来构造一个新的批处理，其中仅包含上次调用 `Index` 时失败的操作。 在 [StackOverflow](https://stackoverflow.com/questions/40012885/azure-search-net-sdk-how-to-use-findfailedactionstoretry) 上有如何正确使用该方法的讨论。
>
>

最后，`UploadDocuments` 方法延迟了两秒钟。 编制索引在 Azure 搜索服务中异步进行，因此，示例应用程序需要等待很短时间，以确保文档可用于搜索。 此类延迟通常仅在演示、测试和示例应用程序中是必需的。

<a name="how-dotnet-handles-documents"></a>

#### <a name="how-the-net-sdk-handles-documents"></a>.NET SDK 如何处理文档
用户可能想知道 Azure 搜索 .NET SDK 如何将用户定义的类（如 `Hotel` ）的实例上传到索引。 为了帮助回答这个问题，请看 `Hotel` 类：

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;
using Newtonsoft.Json;

// The SerializePropertyNamesAsCamelCase attribute is defined in the Azure Search .NET SDK.
// It ensures that Pascal-case property names in the model class are mapped to camel-case
// field names in the index.
[SerializePropertyNamesAsCamelCase]
public partial class Hotel
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsFilterable]
    public string HotelId { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public double? BaseRate { get; set; }

    [IsSearchable]
    public string Description { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.FrLucene)]
    [JsonProperty("description_fr")]
    public string DescriptionFr { get; set; }

    [IsSearchable, IsFilterable, IsSortable]
    public string HotelName { get; set; }

    [IsSearchable, IsFilterable, IsSortable, IsFacetable]
    public string Category { get; set; }

    [IsSearchable, IsFilterable, IsFacetable]
    public string[] Tags { get; set; }

    [IsFilterable, IsFacetable]
    public bool? ParkingIncluded { get; set; }

    [IsFilterable, IsFacetable]
    public bool? SmokingAllowed { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public DateTimeOffset? LastRenovationDate { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public int? Rating { get; set; }

    [IsFilterable, IsSortable]
    public GeographyPoint Location { get; set; }
}
```

首先要注意的是 `Hotel` 的每个公共属性都对应于索引定义中的一个字段，但有一个重要差异：每个字段的名称以小写字母开头（“骆驼拼写法”），而 `Hotel` 的每个公共属性的名称以大写字母开头（“帕斯卡拼写法”）。 在执行目标架构不受应用程序开发人员控制的数据绑定的 .NET 应用程序中，这种情况很常见。 不必违反 .NET 命名准则将属性名设为 camel 大小写，而可以使用 `[SerializePropertyNamesAsCamelCase]` 属性指示 SDK 将属性名自动映射到 camel 大小写。

> [!NOTE]
> Azure 搜索 .NET SDK 使用 [NewtonSoft JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm) 库将自定义模型对象序列化为 JSON 和从 JSON 反序列化。 如果需要，可以自定义此序列化。 有关详细信息，请参阅[使用 JSON.NET 的自定义序列化](#JsonDotNet)。
> 
> 

要注意的第二个问题是修饰每个公共属性的属性 (如`IsFilterable`， `IsSearchable`， `Key`，和`Analyzer`)。 这些属性直接映射到 [Azure 搜索索引的相应属性](https://docs.microsoft.com/rest/api/searchservice/create-index#request)。 `FieldBuilder` 类使用这些属性构造索引的字段定义。

有关第三个重要的事情`Hotel`类是公共属性的数据类型。 这些属性的 .NET 类型映射到它们在索引定义中的等效字段类型。 例如，`Category` 字符串属性映射到 `Edm.String` 类型的 `category` 字段。 `bool?` 和 `Edm.Boolean`、 `DateTimeOffset?`和 `Edm.DateTimeOffset` 等之间存在相似的类型映射。[Azure 搜索 .NET SDK 参考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions.get)中的 `Documents.Get` 方法记录了类型映射的具体规则。 `FieldBuilder` 类会处理此映射，但你最好还是了解此映射，以便在需要排查任何序列化问题时可以下手。

使用自己的类作为文档的这种功能可以在这两个方向上正常工作；此外，还可以检索搜索结果，并使用 SDK 自动将结果反序列化为所选类型，我们会在下一节中对此进行介绍。

> [!NOTE]
> Azure 搜索 .NET SDK 还使用 `Document` 类支持动态类型化文档，该类是字段名称到字段值的键/值映射。 如果在设计时不知道索引架构，或者绑定到特定模型类不太方便，这很有用。 该 SDK 中处理文档的所有方法都有使用 `Document` 类的重载，以及采用泛型类型参数的强类型重载。 本教程中的示例代码仅使用后者。 [`Document` 类](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.document)继承自 `Dictionary<string, object>`。
> 
>

**为何应使用可为 null 的数据类型**

设计自己的模型类以映射到 Azure 搜索索引时，建议将值类型的属性（如 `bool` 和 `int`）声明为可以为 null（例如，`bool?` 而不是 `bool`）。 如果使用不可为 null 属性，必须 **保证** 索引中的所有文档的对应字段都不包含 null 值。 该 SDK 和 Azure 搜索服务都不会帮助强制实施此检查。

这不只是假想的问题：假设将新字段添加到 `Edm.Int32` 类型的现有索引。 更新索引定义后，所有文档的该新字段都具有 null 值（因为 Azure 搜索中的所有类型都可以为 null）。 如果随后使用该字段具有不可为 null `int` 属性的模型类，则在尝试检索文档时会获得如下所示的 `JsonSerializationException`：

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

由于此原因，最佳做法是建议在模型类中使用可以为 null 的类型。

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-jsonnet"></a>使用 JSON.NET 的自定义序列
SDK 使用 JSON.NET 对文档进行序列化和反序列化。 必要时可以通过定义自己的 `JsonConverter` 或 `IContractResolver` 来自定义序列化和反序列化（有关详细信息，请参阅 [JSON.NET 文档](https://www.newtonsoft.com/json/help/html/Introduction.htm)）。 想要使应用程序中的现有模型类适用于 Azure 搜索和其他更高级的方案时，这可能非常有用。 例如，使用自定义序列，可以：

* 包含或排除模型类的某些属性作为文档字段存储。
* 在代码中的属性名称与索引中的字段名称之间进行映射。
* 创建可用于将属性映射到文档字段的自定义属性。

可以在 GitHub 上的 Azure 搜索 .NET SDK 的单元测试中找到实现自定义序列化的示例。 一个好的起点是[此文件夹](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Search/Search.Tests/Tests/Models)。 它包含了自定义序列化测试所用的类。

### <a name="searching-for-documents-in-the-index"></a>在索引中搜索文档
示例应用程序中的最后一步是在索引中搜索一些文档。 执行此操作的方法如下所示：

```csharp
private static void RunQueries(ISearchIndexClient indexClient)
{
    SearchParameters parameters;
    DocumentSearchResult<Hotel> results;

    Console.WriteLine("Search the entire index for the term 'budget' and return only the hotelName field:\n");

    parameters =
        new SearchParameters()
        {
            Select = new[] { "hotelName" }
        };

    results = indexClient.Documents.Search<Hotel>("budget", parameters);

    WriteDocuments(results);

    Console.Write("Apply a filter to the index to find hotels cheaper than $150 per night, ");
    Console.WriteLine("and return the hotelId and description:\n");

    parameters =
        new SearchParameters()
        {
            Filter = "baseRate lt 150",
            Select = new[] { "hotelId", "description" }
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    parameters =
        new SearchParameters()
        {
            OrderBy = new[] { "lastRenovationDate desc" },
            Select = new[] { "hotelName", "lastRenovationDate" },
            Top = 2
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.WriteLine("Search the entire index for the term 'motel':\n");

    parameters = new SearchParameters();
    results = indexClient.Documents.Search<Hotel>("motel", parameters);

    WriteDocuments(results);
}
```

每次执行查询时，此方法首先创建一个新的 `SearchParameters` 对象。 这用于为查询指定其他选项，如排序、筛选、分页和查找。 在此方法中，我们要为不同查询设置 `Filter`、`Select`、`OrderBy` 和 `Top` 属性。 所有 `SearchParameters` 属性在[此处](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.searchparameters)进行了说明。

下一步是实际执行搜索查询。 使用 `Documents.Search` 方法完成此操作。 对于每个查询，我们将要使用的搜索文本（或 `"*"`，如果没有搜索文本）作为字符串传递，还会传递先前创建的搜索参数。 此外，我们指定 `Hotel` 作为 `Documents.Search` 的类型参数，这指示 SDK 将搜索结果中的文档反序列化为类型为 `Hotel` 的对象。

> [!NOTE]
> 可在[此处](https://docs.microsoft.com/rest/api/searchservice/Simple-query-syntax-in-Azure-Search)找到有关搜索查询表达式语法的详细信息。
> 
> 

最后，在每个查询后，该方法循环访问搜索结果中的所有匹配项，将每个文档打印到控制台：

```csharp
private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.Results)
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

让我们依次仔细查看每个查询。 下面是用于执行第一个查询的代码：

```csharp
parameters =
    new SearchParameters()
    {
        Select = new[] { "hotelName" }
    };

results = indexClient.Documents.Search<Hotel>("budget", parameters);

WriteDocuments(results);
```

在此示例中，我们要搜索与单词“budget”匹配的酒店，我们只想取回通过 `Select` 参数指定的酒店名称。 下面是结果：

    Name: Roach Motel

接下来，我们想要查找一夜的费用不超过 150 美元的酒店，并仅返回酒店 ID 和说明：

```csharp
parameters =
    new SearchParameters()
    {
        Filter = "baseRate lt 150",
        Select = new[] { "hotelId", "description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

此查询使用 OData `$filter` 表达式 `baseRate lt 150` 来筛选索引中的文档。 可在[此处](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search)找到有关 Azure 搜索支持的 OData 语法的详细信息。

下面是查询的结果：

    ID: 2   Description: Cheapest hotel in town
    ID: 3   Description: Close to town hall and the river

接下来，我们想要查找最近翻修的前两个酒店，并显示酒店名称和上次翻修日期。 代码如下： 

```csharp
parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "lastRenovationDate desc" },
        Select = new[] { "hotelName", "lastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

在此示例中，我们再次使用 OData 语法来指定 `OrderBy` 参数作为 `lastRenovationDate desc`。 我们还将 `Top` 设置为 2 以确保仅获取前两个文档。 与以前一样，我们设置 `Select` 以指定应返回的字段。

下面是结果：

    Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
    Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00

最后，我们想要查找与单词“motel”匹配的所有酒店：

```csharp
parameters = new SearchParameters();
results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

以下是结果，它包含所有字段，因为我们未指定 `Select` 属性：

    ID: 2   Base rate: 79.99        Description: Cheapest hotel in town     Description (French): Hôtel le moins cher en ville      Name: Roach Motel       Category: Budget        Tags: [motel, budget]   Parking included: yes   Smoking allowed: yes    Last renovated on: 4/28/1982 12:00:00 AM +00:00 Rating: 1/5     Location: Latitude 49.678581, longitude -122.131577

本教程到此步骤结束，但不要就此打住。 * * 后续步骤用于详细了解 Azure 搜索提供其他资源。

## <a name="next-steps"></a>后续步骤
* 浏览 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search) 和 [REST API](https://docs.microsoft.com/rest/api/searchservice/) 的参考资料。
* 查看[命名约定](https://docs.microsoft.com/rest/api/searchservice/Naming-rules)，了解命名各种对象的规则。
* 查看 Azure 搜索中[受支持的数据类型](https://docs.microsoft.com/rest/api/searchservice/Supported-data-types)。
