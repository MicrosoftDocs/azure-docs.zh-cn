---
title: Azure SignalR 服务常见问题解答
description: Azure SignalR 服务的常见问题解答。
author: sffamily
ms.service: signalr
ms.topic: overview
ms.date: 03/01/2019
ms.author: zhshang
ms.openlocfilehash: e9e41ffa335aa95b139a5d5658424c1c5915b569
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64914959"
---
# <a name="azure-signalr-service-faq"></a>Azure SignalR 服务常见问题解答

## <a name="is-azure-signalr-service-ready-for-production-use"></a>Azure SignalR 服务是否随时可用于生产？

是的。
有关正式版的通告，请参阅 [Azure SignalR 服务现已推出正式版](https://azure.microsoft.com/blog/azure-signalr-service-now-generally-available/)。 

完全支持 [ASP.NET Core SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction)。

对 ASP.NET SignalR 的支持目前仍以公共预览版提供。 此处提供了[代码示例](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom)。

## <a name="the-client-connection-closes-with-the-error-message-no-server-available-what-does-it-mean"></a>客户端连接关闭并显示错误消息“没有可用的服务器”。 它意味着什么？

仅当客户端正在向 SignalR 服务发送消息时，才会发生此错误。

如果你没有任何应用程序服务器并且只使用 SignalR 服务 REST API，则此行为是**设计使然**。
在无服务器体系结构中，客户端连接处于“侦听”模式，不会向 SignalR 服务发送任何消息。
详细了解 [REST API](./signalr-quickstart-rest-api.md)。

如果你有应用程序服务器，则此错误消息意味着没有任何应用程序服务器连接到 SignalR 服务实例。

可能的原因包括：
- 没有任何应用程序服务器连接到 SignalR 服务。 检查应用程序服务器日志，以查看可能的连接错误。 在使用多个应用程序服务器的高可用性设置中，这种情况非常罕见。
- SignalR 服务实例存在连接问题。 此问题是暂时性的，可自动恢复。
如果此问题持续一个小时以上，请[在 GitHub 上提出问题](https://github.com/Azure/azure-signalr/issues/new)，或者[在 Azure 中创建支持请求](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request)。

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>如果存在多个应用程序服务器，客户端消息是发送到所有服务器，还是只发送到其中的一个服务器？

客户端与应用程序服务器之间存在一对一映射。 来自一个客户端的消息始终会发送到同一个应用程序服务器。

在客户端或应用程序服务器断开连接之前，会一直保持客户端与应用程序服务器之间的映射。

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>如果我的某个应用程序服务器已关闭，如何发现这种情况并收到通知？

SignalR 服务将监视应用程序服务器的检测信号。
如果在指定的一段时间内未收到检测信号，则认为应用程序服务器处于脱机状态。 映射到此应用程序服务器的所有客户端连接将会断开。

## <a name="why-does-my-custom-iuseridprovider-throw-exception-when-switching-from-aspnet-core-signalr--sdk-to-azure-signalr-service-sdk"></a>从 ASP.NET Core SignalR SDK 切换到 Azure SignalR 服务 SDK 时，自定义 `IUserIdProvider` 为何引发异常？

调用 `IUserIdProvider` 时，ASP.NET Core SignalR SDK 与 Azure SignalR 服务 SDK 的 `HubConnectionContext context` 参数不相同。

在 ASP.NET Core SignalR 中，`HubConnectionContext context` 是包含所有属性有效值的物理客户端连接的上下文。

在 Azure SignalR 服务 SDK 中，`HubConnectionContext context` 是逻辑客户端连接的上下文。 物理客户端连接将连接到 SignalR 服务实例，因此，只提供有限数量的属性。

目前只能访问 `HubConnectionContext.GetHttpContext()` 和 `HubConnectionContext.User`。
可在[此处](https://github.com/Azure/azure-signalr/blob/kevinzha/faq/src/Microsoft.Azure.SignalR/ServiceHubConnectionContext.cs)查看源代码。

## <a name="can-i-configure-the-transports-available-in-signalr-service-as-configuring-it-on-server-side-with-aspnet-core-signalr-for-example-disable-websocket-transport"></a>是否可以像在服务器端使用 ASP.NET Core SignalR 配置传输一样，在 SignalR 服务中配置传输？ 例如，禁用 WebSocket 传输？

不是。

默认情况下，Azure SignalR 服务提供 ASP.NET Core SignalR 所支持的所有三种传输。 这些传输不可配置。 SignalR 服务将处理连接以及所有客户端连接的传输。

可根据[此文](https://docs.microsoft.com/aspnet/core/signalr/configuration?view=aspnetcore-2.1#configure-allowed-transports)中所述配置客户端传输。
