---
title: 使用用于 C++ 的存储客户端库列出 Azure 存储资源 | Microsoft Docs
description: 了解如何在用于 C++ 的 Microsoft Azure 存储客户端库中使用列表 API 来枚举容器、blob、队列、表和实体。
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: mhopkins
ms.reviewer: dineshm
ms.subservice: common
ms.openlocfilehash: edf50b97ff25a67b41bad266df9236145f288409
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146881"
---
# <a name="list-azure-storage-resources-in-c"></a>使用 C++ 列出 Azure 存储资源
使用 Azure 存储进行开发时，很多情况下列表操作很重要。 本文介绍如何使用用于 C++ 的 Microsoft Azure 存储客户端库中提供的列表 API 最有效率地枚举 Azure 存储中的对象。

> [!NOTE]
> 本指南主要面向适用于 C++ 版本 2.x 的 Azure 存储客户端库，该库可通过 [NuGet](https://www.nuget.org/packages/wastorage) 或 [GitHub](https://github.com/Azure/azure-storage-cpp) 获取。
> 
> 

存储客户端库提供了多种方法，用于列出或查询 Azure 存储中的对象。 本文将探讨以下方案：

* 列出帐户中的容器
* 列出容器或虚拟 blob 目录中的 blob
* 列出帐户中的队列
* 列出帐户中的表
* 查询表中的实体

使用不同的重载针对不同的方案演示上述每种方法。

## <a name="asynchronous-versus-synchronous"></a>异步与同步
由于 C++ 的存储客户端库是在 [C++ REST 库](https://github.com/Microsoft/cpprestsdk)基础上构建的，因此我们实际上也支持使用 [pplx::task](https://microsoft.github.io/cpprestsdk/classpplx_1_1task.html) 进行异步操作。 例如：

```cpp
pplx::task<list_blob_item_segment> list_blobs_segmented_async(continuation_token& token) const;
```

同步操作包装了相应的异步操作：

```cpp
list_blob_item_segment list_blobs_segmented(const continuation_token& token) const
{
    return list_blobs_segmented_async(token).get();
}
```

如果要使用多个线程应用程序或服务，我们建议直接使用异步 API，不必创建线程来调用同步 API，那样会严重影响性能。

## <a name="segmented-listing"></a>分段列表
云存储的规模决定了要使用分段列表。 例如，你可能在 Azure blob 容器中有超过一百万个 blob，或者在 Azure 表中有十亿个以上的实体。 这些不是理论上的数字，而是实际的客户使用情况。

因此，要在单个响应中列出所有对象是不实际的。 与之相反，可以使用分页来列出对象。 每个列表 API 都有*分段*重载。

分段列表操作的响应包括：

* <i>_segment</i>，其中包含针对列表 API 进行单个调用时返回的结果集。
* continuation_token，将传递给下一个调用，以获取下一页结果。 当不再有需要返回的结果时，继续标记为 null。

例如，进行典型调用以列出容器中的所有 blob 时，该调用的代码段可能如下所示。 我们的[示例](https://github.com/Azure/azure-storage-cpp/blob/master/Microsoft.WindowsAzure.Storage/samples/BlobsGettingStarted/Application.cpp)中提供了该代码：

```cpp
// List blobs in the blob container
azure::storage::continuation_token token;
do
{
    azure::storage::list_blob_item_segment segment = container.list_blobs_segmented(token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}

    token = segment.continuation_token();
}
while (!token.empty());
```

请注意，一页中返回的结果数可以通过每个 API 的重载中的参数 *max_results* 进行控制，例如：

```cpp
list_blob_item_segment list_blobs_segmented(const utility::string_t& prefix, bool use_flat_blob_listing,
    blob_listing_details::values includes, int max_results, const continuation_token& token,
    const blob_request_options& options, operation_context context)
```

如果未指定 *max_results* 参数，则会在单个页面中返回默认的最大值（最多 5000 个结果）。

另请注意，针对 Azure 表存储进行查询时，可能不会返回任何记录，或者返回的记录数小于所指定的 *max_results* 参数的值，即使继续标记不为空。 可能的一个原因是，查询可能无法在 5 秒钟内完成。 只要继续标记不为空，查询就会继续，代码不应假定分段结果的大小。

大多数情况下，建议采用分段列表编码模式，因为这样可以明确地了解列表或查询的进度，以及服务对每个请求是如何响应的。 具体说来，对于 C++ 应用程序或服务来说，对列表进程进行低级别的控制可以更好地控制内存和性能。

## <a name="greedy-listing"></a>贪婪列表
早期版本的用于 C++ 的存储客户端库（0.5.0 预览版以及更低版本）包括适用于表和查询的不分段列表 API，如以下示例所示：

```cpp
std::vector<cloud_table> list_tables(const utility::string_t& prefix) const;
std::vector<table_entity> execute_query(const table_query& query) const;
std::vector<cloud_queue> list_queues() const;
```

这些方法在实现时，以分段 API 包装器的方式进行。 每次对分段列表进行响应时，代码会将结果附加到一个矢量，并在对完整的容器进行扫描后返回所有结果。

当存储帐户或表所包含的对象数量较少时，此方法可以使用。 但是，随着对象数目的增加，所需的内存可能会增加且没有限制，因为所有结果都保留在内存中。 一个列表操作可能需要很长时间，调用方在此期间无法获得进度方面的信息。

SDK 中的此类贪婪列表 API 在 C#、Java 或 JavaScript Node.js 环境中不存在。 若要避免使用这些贪婪 API 带来的潜在问题，我们在 0.6.0 预览版中删除了它们。

如果代码调用这些贪婪 API：

```cpp
std::vector<azure::storage::table_entity> entities = table.execute_query(query);
for (auto it = entities.cbegin(); it != entities.cend(); ++it)
{
    process_entity(*it);
}
```

应该修改代码，改用分段列表 API：

```cpp
azure::storage::continuation_token token;
do
{
    azure::storage::table_query_segment segment = table.execute_query_segmented(query, token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
    {
        process_entity(*it);
    }

    token = segment.continuation_token();
} while (!token.empty());
```

可以指定该段的 *max_results* 参数，在请求数和内存使用量之间进行平衡，满足应用程序的性能要求。

此外，如果使用了分段列表 API，但采用“贪婪”方式将数据存储在本地集合中，则我们也强烈建议对代码进行重构，以便谨慎地处理在规模较大的本地集合中存储数据的问题。

## <a name="lazy-listing"></a>懒惰列表
虽然贪婪列表带来了各种潜在的问题，但如果容器中的对象不是很多，则使用起来还是很方便的。

如果还使用 C# 或 Oracle Java SDK，则应熟悉枚举型编程模式，该模式提供懒惰形式的列表，仅在需要时才提取具有特定偏移量的数据。 在 C++ 中，基于迭代器的模板也提供了类似方法。

典型的懒惰列表 API（使用 **list_blobs** 作为示例）如下所示：

```cpp
list_blob_item_iterator list_blobs() const;
```

使用懒惰列表模式的典型代码段可能会如下所示：

```cpp
// List blobs in the blob container
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}
```

请注意，懒惰列表仅在同步模式下可用。

与贪婪列表相比，懒惰列表仅在必要时提取数据。 实际上，它仅在下一个迭代器进入下一段的情况下，才从 Azure 存储提取数据。 因此，内存使用量被控制为界定的大小，而且运行速度也快。

延迟列表 API 包括在用于 C++ 的存储客户端库的 2.2.0 版中。

## <a name="conclusion"></a>结论
在本文中，我们针对用于 C++ 的存储客户端库中的各种对象，对列表 API 的不同重载进行了讨论。 总结：

* 在出现多个线程的情况下，强烈建议使用异步 API。
* 大多数情况下，建议使用分段的列表。
* 在库中提供懒惰列表是将其作为包装器，适合在同步方案中使用。
* 不建议使用贪婪列表，因此已将其从库中删除。

## <a name="next-steps"></a>后续步骤
有关 Azure 存储以及用于 C++ 的客户端库的更多信息，请参阅以下资源。

* [如何通过 C++ 使用 Blob 存储](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [如何通过 C++ 使用表存储](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [如何通过 C++ 使用队列存储](../storage-c-plus-plus-how-to-use-queues.md)
* [适用于 C++ 的 Azure 存储客户端库 API 文档。](https://azure.github.io/azure-storage-cpp/)
* [Azure 存储团队博客](https://blogs.msdn.com/b/windowsazurestorage/)
* [Azure 存储文档](https://azure.microsoft.com/documentation/services/storage/)

