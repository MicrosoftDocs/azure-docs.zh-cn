---
title: 响应 Azure Blob 存储事件 | Microsoft Docs
description: 使用 Azure 事件网格订阅 Blob 存储事件。
services: storage,event-grid
author: normesta
ms.author: normesta
ms.reviewer: cbrooks
ms.date: 01/30/2018
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.openlocfilehash: b03d7d98fe43eacab63f45ccacd7d8dea9598c8e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142163"
---
# <a name="reacting-to-blob-storage-events"></a>响应 Blob 存储事件

Azure 存储事件允许应用程序使用新式无服务器体系结构响应 blob 的创建和删除。 为此，它无需复杂的代码或高价低效的轮询服务。  相反，可以通过 [Azure 事件网格](https://azure.microsoft.com/services/event-grid/)向订阅者（如 [Azure Functions](https://azure.microsoft.com/services/functions/) 或 [Azure 逻辑应用](https://azure.microsoft.com/services/logic-apps/)，甚至是你的自定义 HTTP 侦听器）推送事件，且仅需为已使用的内容付费。

Blob 存储事件会可靠地发送到事件网格服务，该服务通过丰富的重试策略和死信传递向应用程序提供可靠的传递服务。

常见的 Blob 存储事件方案包括图像或视频处理、搜索索引或任何面向文件的工作流。  异步文件上传十分适合事件。  基于事件的体系结构对于鲜少更改，但要求立即响应的情况尤为有效。

查看[将 Blob 存储事件路由到自定义 Web 终结点 - CLI](storage-blob-event-quickstart.md) 或[将 Blob 存储事件路由到自定义 Web 终结点 - PowerShell](storage-blob-event-quickstart-powershell.md)，获取简单示例。 

![事件网格模型](./media/storage-blob-event-overview/event-grid-functional-model.png)

## <a name="blob-storage-accounts"></a>Blob 存储帐户
可在常规用途 v2 存储帐户和 Blob 存储帐户中使用 Blob 存储事件。 常规用途 v2 存储帐户支持所有存储服务（包括 Blob、文件、队列和表）的所有功能。 Blob 存储帐户是一个专用存储帐户，用于将非结构化数据作为 Blob（对象）存储到 Azure 存储中。 Blob 存储帐户类似于常规用途存储帐户，并且具有现在使用的所有卓越的耐用性、可用性、伸缩性和性能功能，包括用于块 blob 和追加 blob 的 100% API 一致性。 有关详细信息，请参阅 [Azure 存储帐户概述](../common/storage-account-overview.md)。

## <a name="available-blob-storage-events"></a>可用的 Blob 存储事件
事件网格使用[事件订阅](../../event-grid/concepts.md#event-subscriptions)将事件消息路由到订阅方。  Blob 存储事件订阅包括两种类型的事件：  

> |事件名称|描述|
> |----------|-----------|
> |`Microsoft.Storage.BlobCreated`|当通过 `PutBlob`、`PutBlockList` 或 `CopyBlob` 操作创建或替换 blob 时触发|
> |`Microsoft.Storage.BlobDeleted`|当通过 `DeleteBlob` 操作删除 blob 时触发|

## <a name="event-schema"></a>事件架构
Blob 存储事件包含响应数据更改所需的所有信息。  可以识别 Blob 存储事件，因为 eventType 属性以“Microsoft.Storage”开头。 关于事件网格事件属性使用情况的其他信息，请参阅文档[事件网格事件架构](../../event-grid/event-schema.md)。  

> |属性|Type|描述|
> |-------------------|------------------------|-----------------------------------------------------------------------|
> |主题|string|发出该事件的存储帐户的完整 Azure 资源管理器 ID。|
> |subject|string|作为事件使用者的对象的相对资源路径，使用的扩展 Azure 资源管理器格式与用于描述存储帐户、服务和适用于 Azure RBAC 容器的格式相同。  此格式包含保留大小写的 blob 名称。|
> |EventTime|string|事件生成的日期/时间，采用 ISO 8601 格式|
> |eventType|string|“Microsoft.Storage.BlobCreated”或“Microsoft.Storage.BlobDeleted”|
> |ID|string|此事件的唯一标识符|
> |dataVersion|string|数据对象的架构版本。|
> |metadataVersion|string|顶级属性的架构版本。|
> |数据|对象|blob 存储特定事件数据的集合|
> |data.contentType|string|blob 的内容类型，由 blob 的 Content-Type 标头返回|
> |data.contentLength|数字|blob 的大小，以整数表示字节数，由 blob 的 Content-Type 标头返回。  随 BlobCreated 事件一起发送，但不包括 BlobDeleted。|
> |data.url|string|作为事件使用者的对象的 URL|
> |data.eTag|string|此事件触发时，对象的 etag。  不适用于 BlobDeleted 事件。|
> |data.api|string|触发此事件的 API 操作的名称。 对于 BlobCreated 事件，其值为“PutBlob”、“PutBlockList”或“CopyBlob”。 对于 BlobDeleted 事件，其值为“DeleteBlob”。 这些值与出现在 Azure 存储诊断日志中的 API 名称相同。 请参阅[记录的操作和状态消息](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)。|
> |data.sequencer|string|一个不透明的字符串值，表示任何特定 blob 名称的事件的逻辑顺序。  用户可以使用标准字符串比较，了解同一个 blob 名称上两个事件的相对序列。|
> |data.requestId|string|用于存储 API 操作的服务生成的请求 ID。 可用于通过 Azure 存储诊断日志中的“request-id-header”字段关联到这些日志，并且由“x-ms-request-id”标头中的初始化 API 调用返回。 请参阅[日志格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。|
> |data.clientRequestId|string|用于存储 API 操作的客户端提供的请求 ID。 可用于通过 Azure 存储诊断日志中的“client-request-id”字段关联到这些日志，并且可以通过“x-ms-client-request-id”标头提供到客户端请求中。 请参阅[日志格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。 |
> |data.storageDiagnostics|对象|Azure 存储服务中偶尔附带的诊断数据。 如果存在，事件使用者应忽略它。|
|data.blobType|string|Blob 的类型。 有效值为“BlockBlob”或“PageBlob”。| 

下面是关于 BlobCreated 事件的示例：
```json
[{
  "topic": "/subscriptions/319a9601-1ec0-0000-aebc-8fe82724c81e/resourceGroups/testrg/providers/Microsoft.Storage/storageAccounts/myaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file1.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T01:57:26.005121Z",
  "id": "602a88ef-0001-00e6-1233-1646070610ea",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "799304a4-bbc5-45b6-9849-ec2c66be800a",
    "requestId": "602a88ef-0001-00e6-1233-164607000000",
    "eTag": "0x8D4E44A24ABE7F1",
    "contentType": "text/plain",
    "contentLength": 447,
    "blobType": "BlockBlob",
    "url": "https://myaccount.blob.core.windows.net/testcontainer/file1.txt",
    "sequencer": "00000000000000EB000000000000C65A",
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

有关详细信息，请参阅 [Blob 存储事件架构](../../event-grid/event-schema-blob-storage.md)。

## <a name="filtering-events"></a>筛选事件
可按事件类型以及已创建或已删除对象的容器名称和 blob 名称来筛选 blob 事件订阅。  可在[创建](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest)事件订阅期间或[以后](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest)将筛选器应用于事件订阅。 事件网格中的使用者筛选器基于“开始时间”和“结束时间”的匹配进行筛选，将含有匹配使用者的事件传送给订阅方。 

Blob 存储事件使用者使用的格式：

```
/blobServices/default/containers/<containername>/blobs/<blobname>
```

要匹配存储帐户的所有事件，可将主题筛选器留空。

要匹配在一组共享前缀的容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containerprefix
```

要匹配在特定容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containername/
```

要匹配在共享 blob 名称前缀的特定容器中创建的 blob 的事件，请使用 `subjectBeginsWith` 筛选器，如下所示：

```
/blobServices/default/containers/containername/blobs/blobprefix
```

要匹配在共享 blob 后缀的特定容器中创建的 blob 事件，请使用 `subjectEndsWith` 筛选器，例如“.log”或“.jpg”。 有关详细信息，请参阅 [事件网格概念](../../event-grid/concepts.md#event-subscriptions)。

## <a name="practices-for-consuming-events"></a>使用事件的做法
处理 Blob 存储事件的应用程序应遵循以下建议的做法：
> [!div class="checklist"]
> * 由于可将多个订阅配置为将事件路由至相同的事件处理程序，因此请勿假定事件来自特定的源，而是应检查消息的主题，确保它来自所期望的存储帐户。
> * 同样，检查 eventType 是否为准备处理的项，并且不假定所接收的全部事件都是期望的类型。
> * 消息在一段延迟时间后会无序到达，请使用 etag 字段来了解对象的相关信息是否是最新的。  此外，还可使用 sequencer 字段来了解任何特定对象的事件顺序。
> * 使用 blobType 字段可了解 blob 中允许何种类型的操作，以及应当使用哪种客户端库类型来访问该 blob。 有效值为 `BlockBlob` 或 `PageBlob`。 
> * 将 URL 字段与 `CloudBlockBlob` 和 `CloudAppendBlob` 构造函数配合使用，以访问 blob。
> * 忽略不了解的字段。 此做法有助于适应将来可能添加的新功能。


## <a name="next-steps"></a>后续步骤

了解关于事件网格的详细信息，尝试一下 Blob 存储事件：

- [关于事件网格](../../event-grid/overview.md)
- [将 Blob 存储事件路由到自定义 Web 终结点](storage-blob-event-quickstart.md)
