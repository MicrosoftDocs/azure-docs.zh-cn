---
title: 使用 .NET 的 Azure 存储示例 | Microsoft Docs
description: 查看、下载和运行 Azure 存储的示例代码和应用程序 使用 .NET 存储客户端库发现 Blob、队列、表和文件的入门示例。
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 05/03/2019
ms.author: mhopkins
ms.subservice: common
ms.openlocfilehash: df7c14f1ee83015303657f9a0babde3d60c92292
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209694"
---
# <a name="azure-storage-samples-using-net"></a>使用 .NET 的 Azure 存储示例

## <a name="net-sample-index"></a>.NET 示例索引

下表概述了示例存储库和每个示例中涉及的场景。 单击链接可查看 GitHub 中相应的示例代码。

<table style="font-size:90%"><thead><tr><th style="font-size:110%">终结点</th><th style="font-size:110%">场景</th><th style="font-size:110%">代码示例</th></tr></thead><tbody> 
<tr> 
<td rowspan="16"><b>Blob</b></td>
<td>追加 blob</td> 
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs#L1144">Blob 入门</a></td> 
</tr> 
<tr> 
<td>块 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application</a>（Azure Blob 存储照片库 Web 应用程序）</td>
</tr> 
<tr> 
<td>客户端加密</td>
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/EncryptionSamples/BlobGettingStarted/Program.cs">Blob 加密示例</a></td>
</tr> 
<tr> 
<td>复制 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>创建容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application</a>（Azure Blob 存储照片库 Web 应用程序）</td>
</tr> 
<tr> 
<td>删除 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application</a>（Azure Blob 存储照片库 Web 应用程序）</td>
</tr> 
<tr> 
<td>删除容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>Blob 元数据/属性/统计信息</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>容器 ACL/元数据/属性</td>
<td><a href="https://github.com/Azure-Samples/storage-blobs-dotnet-webapp/blob/master/WebApp-Storage-DotNet/Controllers/HomeController.cs">Azure Blob Storage Photo Gallery Web Application</a>（Azure Blob 存储照片库 Web 应用程序）</td>
</tr> 
<tr> 
<td>获取页面范围</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>租赁 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>列出 Blob/容器</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/GettingStarted.cs">Blob 入门</a></td>
</tr> 
<tr> 
<td>页 blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/GettingStarted.cs">Blob 入门</a></td>
</tr>
<tr> 
<td>SAS</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr>   
<tr> 
<td>服务属性</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-getting-started/blob/master/BlobStorage/Advanced.cs">Blob 入门</a></td>
</tr>           
<tr> 
<td>快照 Blob</td>
<td><a href="https://github.com/Azure-Samples/storage-blob-dotnet-back-up-with-incremental-snapshots/blob/master/Program.cs">使用增量快照备份 Azure 虚拟机磁盘</a></td>
</tr> 
<tr> 
<td rowspan="9"><b>文件</b></td>
<td>创建共享/目录/文件</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr>
<tr> 
<td>删除共享/目录/文件</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/master/FileStorage/GettingStarted.cs">Getting Started with Azure File Service in .NET</a>（.NET 中 Azure 文件服务入门）</td> 
</tr> 
<tr> 
<td>目录属性/元数据</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr> 
<tr> 
<td>下载文件</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr> 
<tr> 
<td>文件属性/元数据/指标</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr> 
<tr> 
<td>文件服务属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr> 
<tr> 
<td>列出目录和文件</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/VisualStudioQuickStarts/DataFileStorage/Program.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr>
<tr> 
<td>列出共享</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr>
<tr> 
<td>共享属性/元数据/统计信息</td> 
<td><a href="https://github.com/Azure-Samples/storage-file-dotnet-getting-started/blob/9f12304b2f5f5472a1c87c1e21be4af5661ac043/FileStorage/Advanced.cs">Azure 存储 .NET 文件存储示例</a></td> 
</tr>
<tr> 
<td rowspan="8"><b>队列</b></td>
<td>添加消息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td>客户端加密</td> 
<td><a href="https://github.com/Azure/azure-storage-net/blob/master/Samples/GettingStarted/EncryptionSamples/QueueGettingStarted/Program.cs">Azure 存储 .NET 队列客户端加密</a></td> 
</tr> 
<tr> 
<td>创建队列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td>删除消息/队列</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td>扫视消息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET（.NET 中 Azure 队列服务入门）</a></td> 
</tr> 
<tr> 
<td>队列 ACL/元数据/统计信息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/Advanced.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td>队列服务属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/Advanced.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td>更新消息</td> 
<td><a href="https://github.com/Azure-Samples/storage-queue-dotnet-getting-started/blob/master/QueueStorage/GettingStarted.cs">Getting Started with Azure Queue Service in .NET</a>（.NET 中 Azure 队列服务入门）</td> 
</tr> 
<tr> 
<td rowspan="7"><b>表</b></td>
<td>创建表</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">Managing Concurrency using Azure Storage - Sample Application</a>（使用 Azure 存储管理并发 - 示例应用程序）</td> 
</tr> 
<tr> 
<td>删除实体/表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">.NET 中的 Azure 表存储入门</a></td> 
</tr> 
<tr> 
<td>插入/合并/替换实体</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">Managing Concurrency using Azure Storage - Sample Application</a>（使用 Azure 存储管理并发 - 示例应用程序）</td> 
</tr> 
<tr> 
<td>查询实体</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">.NET 中的 Azure 表存储入门</a></td> 
</tr> 
<tr> 
<td>查询表</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/BasicSamples.cs">.NET 中的 Azure 表存储入门</a></td> 
</tr> 
<tr> 
<td>表 ACL/属性</td> 
<td><a href="https://github.com/Azure-Samples/storage-table-dotnet-getting-started/blob/master/TableStorage/AdvancedSamples.cs">.NET 中的 Azure 表存储入门</a></td> 
</tr> 
<tr> 
<td>更新实体</td> 
<td><a href="https://code.msdn.microsoft.com/Managing-Concurrency-using-56018114/sourcecode?fileId=123913&pathId=50196262">Managing Concurrency using Azure Storage - Sample Application</a>（使用 Azure 存储管理并发 - 示例应用程序）</td> 
</tr> 
</tbody> 
</table>
<br/>

## <a name="azure-code-samples-library"></a>Azure 代码示例库

若要查看完整的示例库，请转到 [Azure 代码示例](https://azure.microsoft.com/resources/samples/?service=storage)库，其中提供了可下载和在本地运行的 Azure 存储示例。 代码示例库提供的示例代码格式为 .zip。 或者，可浏览和克隆其 GitHub 存储库来了解每个示例。

[!INCLUDE [storage-dotnet-samples-include](../../../includes/storage-dotnet-samples-include.md)]

## <a name="getting-started-guides"></a>入门指南

如果要查找有关如何安装和开始使用 Azure 存储客户端库的说明，请查看以下指南。

* [.NET 中的 Azure Blob 服务入门](../blobs/storage-dotnet-how-to-use-blobs.md)
* [.NET 中的 Azure 队列服务入门](../storage-dotnet-how-to-use-queues.md)
* [.NET 中 Azure 表服务入门](../../cosmos-db/table-storage-how-to-use-dotnet.md)
* [.NET 中的 Azure 文件服务入门](../storage-dotnet-how-to-use-files.md)

## <a name="next-steps"></a>后续步骤

了解有关其他语言的示例的信息：

* Java:[使用 Java 的 Azure 存储示例](storage-samples-java.md)
* 所有其他语言：[Azure 存储示例](../storage-samples.md)
