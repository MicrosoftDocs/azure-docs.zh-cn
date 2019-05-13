---
title: 如何管理对资源的并发写入 - Azure 搜索
description: 使用乐观并发，避免在更新或删除 Azure 搜索索引、索引器及数据源的过程中出现冲突。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 07/21/2017
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 7e569fa30727f2df7411eee5fa6d48f9b9454460
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025341"
---
# <a name="how-to-manage-concurrency-in-azure-search"></a>如何管理 Azure 搜索中的并发

管理索引和数据源等 Azure 搜索资源时，务必安全地更新资源，尤其是应用程序的不同组件并发访问资源的情况下。 当两个客户端在没有协调的情况下并发更新资源时，可能出现争用条件。 为防止此情况，Azure 搜索提供“乐观并发模型”。 对资源没有任何锁定。 相反，每个资源均带有一个 ETag 用于标识资源版本，便于创建避免意外覆盖的请求。

> [!Tip]
> [示例 C# 解决方案](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer)中的概念代码阐释了并发控制如何在 Azure 搜索中工作。 该代码会创建调用并发控制的条件。 对大多数开发人员而言，读取[以下代码片段](#samplecode)可能就已足够，但若想运行它，请编辑 appsettings.json，以添加服务名称和管理员 API 密钥。 假设服务 URL 为 `http://myservice.search.windows.net`，服务名称是 `myservice`。

## <a name="how-it-works"></a>工作原理

乐观并发通过写入索引、索引器、数据源和 synonymMap 资源的 API 调用中的访问条件检查实现。

所有资源均有一个[实体标记 (ETag)](https://en.wikipedia.org/wiki/HTTP_ETag)，它提供对象版本信息。 通过先检查 ETag，确保资源的 ETag 与本地副本匹配，可避免典型工作流（获取、本地修改、更新）中的并发更新。

+ REST API 在请求头使用 [ETag](https://docs.microsoft.com/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search)。
+ .NET SDK 通过 accessCondition 对象，对资源设置 [If-Match | If-Match-None 标头](https://docs.microsoft.com/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search) 来设置 ETag。 从 [IResourceWithETag (.NET SDK)](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.iresourcewithetag) 继承的任何对象都具有 accessCondition 对象。

每次更新资源时，其 ETag 将自动更改。 实现并发管理时，只需对更新请求设置一个前提条件，要求远程资源的 ETag 与在客户端上修改的资源副本的 ETag 相同。 如果并发进程已更改远程资源，ETag 将不满足前提条件，请求将失败并出现 HTTP 412。 如果使用 .NET SDK，这表示为 `CloudException`，此时 `IsAccessConditionFailed()` 扩展方法返回 true。

> [!Note]
> 只存在一个并发机制。 无论对资源更新使用哪个 API，都始终使用该机制。

<a name="samplecode"></a>
## <a name="use-cases-and-sample-code"></a>用例和示例代码

以下代码演示对密钥更新操作的 accessCondition 检查：

+ 如果资源不复存在，则更新将失败
+ 如果资源版本更改，更新将失败

### <a name="sample-code-from-dotnetetagsexplainer-programhttpsgithubcomazure-samplessearch-dotnet-getting-startedtreemasterdotnetetagsexplainer"></a>[DotNetETagsExplainer 程序](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer)中的示例代码

```
    class Program
    {
        // This sample shows how ETags work by performing conditional updates and deletes
        // on an Azure Search index.
        static void Main(string[] args)
        {
            IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            IConfigurationRoot configuration = builder.Build();

            SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

            Console.WriteLine("Deleting index...\n");
            DeleteTestIndexIfExists(serviceClient);

            // Every top-level resource in Azure Search has an associated ETag that keeps track of which version
            // of the resource you're working on. When you first create a resource such as an index, its ETag is
            // empty.
            Index index = DefineTestIndex();
            Console.WriteLine(
                $"Test index hasn't been created yet, so its ETag should be blank. ETag: '{index.ETag}'");

            // Once the resource exists in Azure Search, its ETag will be populated. Make sure to use the object
            // returned by the SearchServiceClient! Otherwise, you will still have the old object with the
            // blank ETag.
            Console.WriteLine("Creating index...\n");
            index = serviceClient.Indexes.Create(index);
            
            Console.WriteLine($"Test index created; Its ETag should be populated. ETag: '{index.ETag}'");

            // ETags let you do some useful things you couldn't do otherwise. For example, by using an If-Match
            // condition, we can update an index using CreateOrUpdate and be guaranteed that the update will only
            // succeed if the index already exists.
            index.Fields.Add(new Field("name", AnalyzerName.EnMicrosoft));
            index =
                serviceClient.Indexes.CreateOrUpdate(
                    index,
                    accessCondition: AccessCondition.GenerateIfExistsCondition());

            Console.WriteLine(
                $"Test index updated; Its ETag should have changed since it was created. ETag: '{index.ETag}'");

            // More importantly, ETags protect you from concurrent updates to the same resource. If another
            // client tries to update the resource, it will fail as long as all clients are using the right
            // access conditions.
            Index indexForClient1 = index;
            Index indexForClient2 = serviceClient.Indexes.Get("test");

            Console.WriteLine("Simulating concurrent update. To start, both clients see the same ETag.");
            Console.WriteLine($"Client 1 ETag: '{indexForClient1.ETag}' Client 2 ETag: '{indexForClient2.ETag}'");

            // Client 1 successfully updates the index.
            indexForClient1.Fields.Add(new Field("a", DataType.Int32));
            indexForClient1 =
                serviceClient.Indexes.CreateOrUpdate(
                    indexForClient1,
                    accessCondition: AccessCondition.IfNotChanged(indexForClient1));

            Console.WriteLine($"Test index updated by client 1; ETag: '{indexForClient1.ETag}'");

            // Client 2 tries to update the index, but fails, thanks to the ETag check.
            try
            {
                indexForClient2.Fields.Add(new Field("b", DataType.Boolean));
                serviceClient.Indexes.CreateOrUpdate(
                    indexForClient2,
                    accessCondition: AccessCondition.IfNotChanged(indexForClient2));

                Console.WriteLine("Whoops; This shouldn't happen");
                Environment.Exit(1);
            }
            catch (CloudException e) when (e.IsAccessConditionFailed())
            {
                Console.WriteLine("Client 2 failed to update the index, as expected.");
            }

            // You can also use access conditions with Delete operations. For example, you can implement an
            // atomic version of the DeleteTestIndexIfExists method from this sample like this:
            Console.WriteLine("Deleting index...\n");
            serviceClient.Indexes.Delete("test", accessCondition: AccessCondition.GenerateIfExistsCondition());

            // This is slightly better than using the Exists method since it makes only one round trip to
            // Azure Search instead of potentially two. It also avoids an extra Delete request in cases where
            // the resource is deleted concurrently, but this doesn't matter much since resource deletion in
            // Azure Search is idempotent.

            // And we're done! Bye!
            Console.WriteLine("Complete.  Press any key to end application...\n");
            Console.ReadKey();
        }

        private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
        {
            string searchServiceName = configuration["SearchServiceName"];
            string adminApiKey = configuration["SearchServiceAdminApiKey"];

            SearchServiceClient serviceClient =
                new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
            return serviceClient;
        }

        private static void DeleteTestIndexIfExists(SearchServiceClient serviceClient)
        {
            if (serviceClient.Indexes.Exists("test"))
            {
                serviceClient.Indexes.Delete("test");
            }
        }

        private static Index DefineTestIndex() =>
            new Index()
            {
                Name = "test",
                Fields = new[] { new Field("id", DataType.String) { IsKey = true } }
            };
    }
}
```

## <a name="design-pattern"></a>设计模式

用于实现乐观并发的设计模式应包含一个循环用于重试访问条件检查，包含一个访问条件测试，并在尝试重新应用更改前选择性地检索更新后的资源。

此代码片段演示如何向已有的索引添加 synonymMap。 此代码摘自[同义词C#Azure 搜索的示例](search-synonyms-tutorial-sdk.md)。

代码片段获取“hotels”索引，检查更新操作上的对象版本，在条件失败时引发异常，然后重试该操作（最多三次），从服务器开始索引检索以获取最新版本。

        private static void EnableSynonymsInHotelsIndexSafely(SearchServiceClient serviceClient)
        {
            int MaxNumTries = 3;

            for (int i = 0; i < MaxNumTries; ++i)
            {
                try
                {
                    Index index = serviceClient.Indexes.Get("hotels");
                    index = AddSynonymMapsToFields(index);

                    // The IfNotChanged condition ensures that the index is updated only if the ETags match.
                    serviceClient.Indexes.CreateOrUpdate(index, accessCondition: AccessCondition.IfNotChanged(index));

                    Console.WriteLine("Updated the index successfully.\n");
                    break;
                }
                catch (CloudException e) when (e.IsAccessConditionFailed())
                {
                    Console.WriteLine($"Index update failed : {e.Message}. Attempt({i}/{MaxNumTries}).\n");
                }
            }
        }
        
        private static Index AddSynonymMapsToFields(Index index)
        {
            index.Fields.First(f => f.Name == "category").SynonymMaps = new[] { "desc-synonymmap" };
            index.Fields.First(f => f.Name == "tags").SynonymMaps = new[] { "desc-synonymmap" };
            return index;
        }


## <a name="next-steps"></a>后续步骤

有关如何安全更新现有索引的更多上下文，请查看[同义词 C# 示例](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms)。

尝试修改以下任一示例，将 ETag 或 AccessCondition 对象包含在内。

+ [GitHub 上的 REST API 示例](https://github.com/Azure-Samples/search-rest-api-getting-started)
+ [GitHub 上的 .NET SDK 示例](https://github.com/Azure-Samples/search-dotnet-getting-started)。 此解决方案包括“DotNetEtagsExplainer”项目，后者包含本文所示的代码。

## <a name="see-also"></a>另请参阅

[常见的 HTTP 请求和响应标头](https://docs.microsoft.com/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search)
[HTTP 状态代码](https://docs.microsoft.com/rest/api/searchservice/http-status-codes)
[索引操作 (REST API)](https://docs.microsoft.com/rest/api/searchservice/index-operations)
