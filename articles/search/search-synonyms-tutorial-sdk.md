---
title: 同义词C#示例-Azure 搜索
description: 在此C#的示例中，了解如何将同义词功能添加到 Azure 搜索中的索引。 同义词映射是等效术语的列表。 具有同义词支持的字段可扩展查询以包括用户提供的术语和所有相关的同义词。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 5b81e4b9a8773cc8e4cc76582ccf2df88565d3d8
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025173"
---
# <a name="example-add-synonyms-for-azure-search-in-c"></a>示例：使用 C# 为 Azure 搜索添加同义词

使用同义词进行搜索时，在语义上被视为与输入字词等效的字词也属于匹配项，从而扩展了查询的范围。 例如，使用“car”进行搜索时，也可将包含“automobile”或“vehicle”字词的文档视为匹配项。 

在 Azure 搜索中，同义词在“同义词映射”中通过可将等效字词关联在一起的“映射规则”进行定义。 此示例介绍了用于添加和使用现有索引中使用同义词的基本步骤。 学习如何：

> [!div class="checklist"]
> * 创建同义词映射使用[SynonymMap](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.synonymmap?view=azure-dotnet)类。 
> * 设置[SynonymMaps](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.field.synonymmaps?view=azure-dotnet)上应支持同义词通过查询扩展字段属性。

像往常一样，您可以查询启用了同义词的字段。 没有访问同义词所需的其他查询语法。

可以创建多个同义词映射，将其发布为在服务范围内可供任何索引使用的资源，然后引用要在字段级别使用的资源。 在查询时，除了搜索索引，Azure 搜索还会查看同义词映射（如果在查询所用的字段上指定了该映射）。

> [!NOTE]
> 可以以编程方式创建同义词，但不是在门户。 如果 Azure 门户支持同义词将对你很有用，请在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 上提供反馈

## <a name="prerequisites"></a>必备组件

教程要求如下：

* [Visual Studio](https://www.visualstudio.com/downloads/)
* [Azure 搜索服务](search-create-service-portal.md)
* [Microsoft.Azure.Search .NET 库](https://aka.ms/search-sdk)
* [如何使用 .NET 应用程序中的 Azure 搜索](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)

## <a name="overview"></a>概述

前后比较查询可演示同义词的值。 在此示例中，使用执行查询并针对示例索引返回结果的示例应用程序。 示例应用程序创建了一个名为“hotels”的小索引，其中填充了两个文档。 该应用程序使用未在索引中出现的字词和短语执行搜索查询，接着启用同义词功能，再次进行相同的搜索。 以下代码演示了整个流程。

```csharp
  static void Main(string[] args)
  {
      SearchServiceClient serviceClient = CreateSearchServiceClient();

      Console.WriteLine("{0}", "Cleaning up resources...\n");
      CleanupResources(serviceClient);

      Console.WriteLine("{0}", "Creating index...\n");
      CreateHotelsIndex(serviceClient);

      ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

      Console.WriteLine("{0}", "Uploading documents...\n");
      UploadDocuments(indexClient);

      ISearchIndexClient indexClientForQueries = CreateSearchIndexClient();

      RunQueriesWithNonExistentTermsInIndex(indexClientForQueries);

      Console.WriteLine("{0}", "Adding synonyms...\n");
      UploadSynonyms(serviceClient);
      EnableSynonymsInHotelsIndex(serviceClient);
      Thread.Sleep(10000); // Wait for the changes to propagate

      RunQueriesWithNonExistentTermsInIndex(indexClientForQueries);

      Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");

      Console.ReadKey();
  }
```
有关如何创建和填充示例索引的步骤，请参阅[如何使用 .NET 应用程序中的 Azure 搜索](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)。

## <a name="before-queries"></a>“启用前”查询

在 `RunQueriesWithNonExistentTermsInIndex` 中，使用“five star”、“internet”和“economy AND hotel”发出搜索查询。
```csharp
Console.WriteLine("Search the entire index for the phrase \"five star\":\n");
results = indexClient.Documents.Search<Hotel>("\"five star\"", parameters);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the term 'internet':\n");
results = indexClient.Documents.Search<Hotel>("internet", parameters);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the terms 'economy' AND 'hotel':\n");
results = indexClient.Documents.Search<Hotel>("economy AND hotel", parameters);
WriteDocuments(results);
```
两个已编入索引的文档都不包含这些字词，因此首次 `RunQueriesWithNonExistentTermsInIndex` 获得的输出如下。
~~~
Search the entire index for the phrase "five star":

no document matched

Search the entire index for the term 'internet':

no document matched

Search the entire index for the terms 'economy' AND 'hotel':

no document matched
~~~

## <a name="enable-synonyms"></a>启用同义词

启用同义词是一个两步过程。 首先定义并上传同义词规则，然后将字段配置为使用这些规则。 `UploadSynonyms` 和 `EnableSynonymsInHotelsIndex` 中概述了此过程。

1. 将同义词映射添加到搜索服务。 在 `UploadSynonyms` 中，我们在同义词映射“desc-synonymmap”中定义了四条规则，并将其上传到服务。
   ```csharp
    var synonymMap = new SynonymMap()
    {
        Name = "desc-synonymmap",
        Format = "solr",
        Synonyms = "hotel, motel\n
                    internet,wifi\n
                    five star=>luxury\n
                    economy,inexpensive=>budget"
    };

    serviceClient.SynonymMaps.CreateOrUpdate(synonymMap);
   ```
   同义词映射必须符合开源标准 `solr` 格式。 该格式在 [Azure 搜索中的同义词](search-synonyms.md)的`Apache Solr synonym format`部分进行了说明。

2. 将可搜索字段配置为允许在索引定义中使用同义词映射。 在 `EnableSynonymsInHotelsIndex` 中，我们对`category`和`tags`这两个字段启用了同义词功能，方法是将 `synonymMaps` 属性设置为新上传的同义词映射的名称。
   ```csharp
   Index index = serviceClient.Indexes.Get("hotels");
   index.Fields.First(f => f.Name == "category").SynonymMaps = new[] { "desc-synonymmap" };
   index.Fields.First(f => f.Name == "tags").SynonymMaps = new[] { "desc-synonymmap" };

   serviceClient.Indexes.CreateOrUpdate(index);
   ```
   添加同义词映射时，索引重建不是必需的。 可以向服务添加同义词映射，并对任意索引中的现有字段定义进行修正，使之使用新的同义词映射。 添加新属性不影响索引可用性。 这同样适用于禁用字段同义词功能的情况。 可以直接将 `synonymMaps` 属性设置为空列表。
   ```csharp
   index.Fields.First(f => f.Name == "category").SynonymMaps = new List<string>();
   ```

## <a name="after-queries"></a>“启用后”查询

上传同义词映射并对索引进行更新，允许其使用同义词映射以后，再次调用 `RunQueriesWithNonExistentTermsInIndex` 会获得如下输出：

~~~
Search the entire index for the phrase "five star":

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the term 'internet':

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the terms 'economy' AND 'hotel':

Name: Roach Motel       Category: Budget        Tags: [motel, budget]
~~~
第一个查询根据规则 `five star=>luxury` 查找文档。 第二个查询使用 `internet,wifi` 扩展了搜索，第三个查询在查找匹配的文档时同时使用 `hotel, motel` 和 `economy,inexpensive=>budget`。

添加同义词完全改观了搜索体验。 在此示例中，原始查询无法返回有意义的结果，即使索引中的文档密切相关。 启用同义词可以扩展索引，使之包括常用字词，但不更改索引中的基础数据。

## <a name="sample-application-source-code"></a>示例应用程序源代码
可以在 [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms) 上找到本演练中所用示例应用程序的完整源代码。

## <a name="clean-up-resources"></a>清理资源

在示例后清理的速度最快方法是通过删除包含 Azure 搜索服务的资源组。 现在，可以删除资源组以永久删除其中的所有内容。 在门户中，资源组名称显示在 Azure 搜索服务的“概述”页上。

## <a name="next-steps"></a>后续步骤

此示例演示了中的同义词功能C#代码来创建和发布映射规则，然后对查询中调用同义词映射。 可以在 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search) 和 [REST API](https://docs.microsoft.com/rest/api/searchservice/) 参考文档中找到更多信息。

> [!div class="nextstepaction"]
> [如何在 Azure 搜索中使用同义词](search-synonyms.md)
