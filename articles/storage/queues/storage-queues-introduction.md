---
title: Azure 队列简介 - Azure 存储
description: Azure 队列简介
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: overview
ms.date: 02/06/2019
ms.author: mhopkins
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: 544ff9d2c624ef62bf8041afd818153c1c4bfcc8
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142501"
---
# <a name="what-are-azure-queues"></a>什么是 Azure 队列？

Azure 队列存储是一项可存储大量消息的服务，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。 一条队列消息的大小最多可为 64 KB，一个队列中可以包含数百万条消息，直至达到存储帐户的总容量限值。

## <a name="common-uses"></a>常见用途

队列存储的常见用途包括：

* 创建积压工作以进行异步处理
* 将消息从 Azure Web 角色传递到 Azure 辅助角色

## <a name="queue-service-concepts"></a>队列服务概念

队列服务包含以下组件：

![队列概念](./media/storage-queues-introduction/queue1.png)

* **URL 格式：** 可使用以下 URL 格式访问队列：   
    https://`<storage account>`.queue.core.windows.net/`<queue>` 
  
    可使用以下 URL 访问示意图中的某个队列：  
  
    `https://myaccount.queue.core.windows.net/images-to-download`

* **存储帐户：** 对 Azure 存储进行的所有访问都要通过存储帐户完成。 有关存储帐户容量的详细信息，请参阅 [Azure 存储可伸缩性和性能目标](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) 。

* **队列：** 一个队列包含一组消息。 所有消息必须位于相应的队列中。 请注意，队列名称必须全部小写。 有关命名队列的信息，请参阅 [命名队列和元数据](https://msdn.microsoft.com/library/azure/dd179349.aspx)。

* **消息：** 一条消息（无论何种格式）的最大大小为 64 KB。 消息可以保留在队列中的最长时间为 7 天。

## <a name="next-steps"></a>后续步骤

* [创建存储帐户](../storage-create-storage-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
* [使用 .NET 的队列入门](storage-dotnet-how-to-use-queues.md)
