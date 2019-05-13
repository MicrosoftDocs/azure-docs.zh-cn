---
title: 诊断 Azure 通知中心的已删除的通知
description: 了解如何诊断 Azure 通知中心的已删除通知的常见问题。
services: notification-hubs
documentationcenter: Mobile
author: jwargo
manager: patniko
editor: spelluru
ms.assetid: b5c89a2a-63b8-46d2-bbed-924f5a4cce61
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: NA
ms.devlang: multiple
ms.topic: article
ms.date: 04/04/2019
ms.author: jowargo
ms.openlocfilehash: eebf9ef63a8622c4cc431322b786fdf30f6352fe
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64925823"
---
# <a name="diagnose-dropped-notifications-in-azure-notification-hubs"></a>诊断 Azure 通知中心的已删除的通知

有关 Azure 通知中心的一个常见问题是如何进行故障排除时从应用程序的通知不会显示在客户端设备上。 客户想要知道删除通知的位置和原因，以及如何解决此问题。 本文标识了通知被删除或设备收不到通知的各种原因。 它还介绍了如何确定根本原因。

首先务必了解通知中心如何将通知推送到设备。

![通知中心体系结构][0]

在典型的发送通知流中，消息从*应用程序后端*发送到通知中心。 通知中心处理的所有注册。 它会考虑配置的标记和标记表达式来确定目标。 目标是需要接收推送通知注册。 这些注册可以横跨任何受支持的平台：Android、 百度 （中国的 Android 设备），即发即 OS (Amazon) iOS、 Windows、 和 Windows Phone。

确定目标之后，通知中心将通知推送到设备平台的*推送通知服务*。 示例包括适用于 Apple 的 Apple Push Notification 服务 (APNs) 和适用于 Google 的 Firebase Cloud Messaging (FCM)。 通知中心推送跨多批注册拆分的通知。 它使用各自的推送通知服务，基于你在设置在 Azure 门户中的凭据进行身份验证**配置通知中心**。 然后，推送通知服务将通知转发到各自的*客户端设备*。

通知传递的最后一步是平台的推送通知服务和设备之间。 在任何 （客户端、 应用程序后端，通知中心和平台的推送通知服务） 的推送通知过程中的四个阶段，通知传递可能会失败。 有关通知中心体系结构的详细信息，请参阅[通知中心概述]。

未能传递通知可能会出现在初始测试/暂存阶段。 于此阶段删除的通知可能表示存在配置问题。 在生产环境中将通知传递在失败时，可能会删除部分或全部通知。 在这种情况下指示更深层次的应用程序或消息模式问题。

下一节会查看在其中可能删除通知，范围从常见到少见的方案。

## <a name="notification-hubs-misconfiguration"></a>通知中心配置错误 ##

若要将通知发送到各自的推送通知服务，通知中心必须验证自身身份的应用程序上下文中。 必须使用目标平台通知服务 （Microsoft、 Apple、 Google 等） 来创建开发人员帐户。 然后，必须注册您的应用程序从哪里获得令牌或密钥，用于处理目标 PNS 的操作系统。

你必须将平台凭据添加到 Azure 门户中。 如果设备未不收到任何通知，第一步是确保在通知中心配置正确的凭据。 凭据必须与匹配特定于平台的开发人员帐户下创建的应用程序。

有关如何完成此过程的分步说明，请参阅 [Azure 通知中心入门]。

下面是一些需要检查的常见错误配置：

### <a name="notification-hub-name-location"></a>通知中心名称位置

确保以下各个位置中的通知中心名称（不含错字）均相同：
   * 你注册了一个从客户端
   * 从后端发送通知的位置
   * 在配置推送通知服务凭据

请确保在客户端上使用正确的共享的访问签名配置字符串和应用程序后端。 通常情况下，必须使用**DefaultListenSharedAccessSignature**客户端上并**DefaultFullSharedAccessSignature**上应用程序后端。 这将授予权限将通知发送到通知中心。

### <a name="apn-configuration"></a>APN 配置 ###

必须维护两个不同的中心： 一个用于生产，另一个用于测试。 必须上传到一个单独的中心比证书/中心，将在生产环境中使用的沙盒环境中使用的证书。 请勿尝试将不同类型的证书上传到相同的中心。 它将导致通知失败。

如果您无意中将不同类型的证书上载到相同的中心，应删除该中心并重新开始使用新的中心。 如果出于某种原因无法删除该中心，您必须至少从中心删除所有现有注册。

### <a name="fcm-configuration"></a>FCM 配置 ###

1. 絋粄*服务器密钥*获取 Firebase 匹配项从 Azure 门户中注册的服务器密钥。

   ![Firebase 服务器密钥][3]

2. 确保在客户端上配置“项目 ID”。 可以从 Firebase 仪表板获取“项目 ID”的值。

   ![Firebase 项目 ID][1]

## <a name="application-issues"></a>应用程序问题 ##

### <a name="tags-and-tag-expressions"></a>标记和标记表达式 ###

如果使用标记或标记表达式来细分受众，就可以发送通知时，找到任何目标。 此错误基于指定的标记或标记表达式在发送调用中。

查看注册，请确保标记匹配时发送通知。 然后，验证从含有这些注册的客户端收到通知。

例如，假设使用通知中心注册所有使用标记"政治"。 如果您然后发送带标记的通知"体育"，就不会通知发送到任何设备。 复杂的用例可能涉及标记表达式通过使用"标记 A"注册所在*或*"标记 B"，但您针对"标记 A & & 标记 B" 本文后面的自我诊断提示部分演示如何查看注册及其标记。

### <a name="template-issues"></a>模板问题 ###

如果使用模板，请确保遵循[模板]中所述的准则。

### <a name="invalid-registrations"></a>注册无效 ###

如果通知中心配置正确并且标记或标记表达式的使用也正确，找到的有效目标。 应将通知发送到这些目标。 接着，通知中心会并行启动几个处理批次。 每个批次将消息发送到一组注册。

> [!NOTE]
> 由于通知中心处理并行批处理，因此不保证传递通知的顺序。

通知中心已针对"大多数一次"消息传递模型优化。 我们尝试执行重复数据消除，从而不向设备传递一次以上的通知。 注册会检查以确保发送到推送通知服务之前，每个设备标识符发送该只有一个消息。

每个批次发送到推送通知服务，后者会接受并验证注册。 在此过程中，就可以推送通知服务将在批处理中检测具有一个或多个注册的错误。 推送通知服务向通知中心，然后返回一个错误并停止该过程。 推送通知服务会完全删除该批次。 这是如此使用 APNs，它使用 TCP 流协议。

在这种情况下，从数据库中删除出错的注册。 然后，我们针对该批次中的其他设备重试通知传递。

若要获取有关针对注册的失败传递尝试的更多错误信息，可以使用通知中心 REST API [按消息遥测：获取通知消息遥测数据](https://msdn.microsoft.com/library/azure/mt608135.aspx)和 [PNS 反馈](https://msdn.microsoft.com/library/azure/mt705560.aspx)。 有关示例代码，请参阅[发送 REST 示例](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/SendRestExample/)。

## <a name="push-notification-service-issues"></a>推送通知服务问题

推送通知服务收到通知后，它将通知传递到设备。 此时，通知中心有无法控制传递到设备的通知。

平台通知服务非常强大，因为这些通知往往在几秒内到达设备。 如果推送通知服务进行限制，通知中心会应用指数回退策略。 如果推送通知服务在 30 分钟内都无法访问，则策略在过期并永久删除消息。

如果推送通知服务尝试传递通知，但设备处于脱机状态，推送通知服务存储通知。 仅在有限的一段时间存储。 等设备可用时再将通知传递到设备。

每个应用程序将存储只有一个最近的通知。 如果设备处于脱机状态时发送了多个通知，每个新通知将导致被放弃的最后一个。 只保留最新通知调用*合并*在 APNs 并*折叠*FCM 中。 （FCM 使用折叠密钥）。当设备在很长时间处于脱机状态时，则放弃存储设备的通知。 有关详细信息，请参阅[APNs overview]并[About FCM messages]。

与通知中心，您可以在使用泛型 SendNotification API 通过 HTTP 标头中传递合并密钥。 例如，对于 .NET SDK，你会使用 `SendNotificationAsync`。 SendNotification API 还会按原样向各自的推送通知服务传递的 HTTP 标头。

## <a name="self-diagnosis-tips"></a>自我诊断提示

下面介绍了诊断通知中心已删除通知的根本原因的各种途径。

### <a name="verify-credentials"></a>验证凭据 ###

#### <a name="push-notification-service-developer-portal"></a>推送通知服务开发人员门户 ####

在各自的推送通知服务开发人员门户（APNs、FCM、Windows 通知服务等）中验证凭据。 有关详细信息，请参阅[教程：使用 Azure 通知中心向通用 Windows 平台应用发送通知](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification)创建的解决方案。

#### <a name="azure-portal"></a>Azure 门户 ####

若要查看和匹配与从推送通知服务开发人员门户获取的凭据，请转到**访问策略**在 Azure 门户中的选项卡。

![Azure 门户访问策略][4]

### <a name="verify-registrations"></a>验证注册

#### <a name="visual-studio"></a>Visual Studio ####

在 Visual Studio 中，您可以通过连接到 Azure 服务器资源管理器来查看和管理多个 Azure 服务，包括通知中心。 此快捷方式是主要用于开发/测试环境。

![Visual Studio 服务器资源管理器][9]

可以查看和管理你的中心中的所有注册。 注册可按平台、 本机或模板注册、 标记、 推送通知服务标识符、 注册 ID 和到期日期进行分类。 还可以在此页面中编辑注册。 它对于编辑标记特别有用。

右键单击通知中心**服务器资源管理器**，然后选择**诊断**。 

![Visual Studio 服务器资源管理器：诊断菜单](./media/notification-hubs-diagnosing/diagnose-menu.png)

会看到以下页面：

![Visual Studio：诊断页](./media/notification-hubs-diagnosing/diagnose-page.png)

切换到“设备注册”页：

![Visual Studio：设备注册](./media/notification-hubs-diagnosing/VSRegistrations.png)

若要发送测试通知消息，可以使用“测试性发送”页：

![Visual Studio：测试发送](./media/notification-hubs-diagnosing/test-send-vs.png)

> [!NOTE]
> 使用 Visual Studio 编辑注册，仅在开发/测试，并使用有限的注册。 如果你需要编辑批量注册，请考虑使用导出和导入注册功能中所述[How To:导出和修改批量注册](https://msdn.microsoft.com/library/dn790624.aspx)。

#### <a name="service-bus-explorer"></a>服务总线资源管理器 ####

很多客户使用[服务总线资源管理器](https://github.com/paolosalvatori/ServiceBusExplorer)来查看和管理其通知中心。 服务总线资源管理器是一个开源项目。 

### <a name="verify-message-notifications"></a>验证消息通知

#### <a name="azure-portal"></a>Azure 门户 ####

若要向客户端发送测试通知，而不启动和运行服务后端，请在“支持 + 故障排除”下选择“测试发送”。

![Azure 中的测试发送功能][7]

#### <a name="visual-studio"></a>Visual Studio ####

也可以从 Visual Studio 发送测试通知。

![Visual Studio 中的测试发送功能][10]

有关将通知中心与 Visual Studio 服务器资源管理器搭配使用的详细信息，请参阅以下文章：

* [如何查看通知中心的设备注册](https://docs.microsoft.com/previous-versions/windows/apps/dn792122(v=win.10))
* [深入探讨：Visual Studio 2013 Update 2 RC 和 Azure SDK 2.3]
* [宣布推出 Visual Studio 2013 Update 3 和 Azure SDK 2.4]

### <a name="debug-failed-notifications-and-review-notification-outcome"></a>调试失败的通知和查看通知结果

#### <a name="enabletestsend-property"></a>EnableTestSend 属性 ####

当发送通过通知中心通知时，通知最初排入队列。 通知中心确定正确的目标后，便将通知发送到推送通知服务。 如果你使用 REST API 或任一客户端 Sdk，在发送调用的返回只表示消息已排队与通知中心。 它不提供深入了解通知中心最终发送到推送通知服务通知发生了什么情况。

如果通知没有到达客户端设备，可能已出错时通知中心尝试将它传递到推送通知服务。 例如，有效负载大小可能超出了推送通知服务允许的上限，或者在通知中心配置的凭据可能无效。

若要深入了解推送通知服务错误，可以使用 [EnableTestSend] 属性。 从门户或 Visual Studio 客户端发送测试消息时，系统会自动启用此属性。 可以使用此属性以查看详细的调试信息并且也是通过 Api。 目前可在 .NET SDK 中使用此属性。 它将添加到所有客户端 Sdk 最终。

若要结合使用 `EnableTestSend` 属性和 REST 调用，可在发送调用的末尾追加名为 test 的查询字符串参数。 例如：

```text
https://mynamespace.servicebus.windows.net/mynotificationhub/messages?api-version=2013-10&test
```

#### <a name="net-sdk-example"></a>.NET SDK 示例 ####

下面是使用 .NET SDK 发送本机弹出 (toast) 通知的示例：

```csharp
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName);
var result = await hub.SendWindowsNativeNotificationAsync(toast);
Console.WriteLine(result.State);
```

在执行结束时，`result.State` 只表示 `Enqueued`。 结果不提供任何见解推送通知发生了什么情况。

接下来，可以使用 `EnableTestSend` 布尔属性。 初始化 `NotificationHubClient` 时，可使用 `EnableTestSend` 属性获取有关发送通知时出现的推送通知服务错误的详细状态。 发送调用需要更长时间才能返回，因为它首先需要通知中心将通知传递到推送通知服务。

```csharp
    bool enableTestSend = true;
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName, enableTestSend);

    var outcome = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(outcome.State);

    foreach (RegistrationResult result in outcome.Results)
    {
        Console.WriteLine(result.ApplicationPlatform + "\n" + result.RegistrationId + "\n" + result.Outcome);
    }
```

#### <a name="sample-output"></a>示例输出 ####

```text
DetailedStateAvailable
windows
7619785862101227384-7840974832647865618-3
The Token obtained from the Token Provider is wrong
```

此消息表示在通知中心配置的凭据是无效或没有中心中的注册问题。 删除此注册，让客户端将消息发送之前重新创建注册。

> [!NOTE]
> 使用 `EnableTestSend` 属性受到严重限制。 只能在开发/测试环境中和一组有限的注册，请使用此选项。 调试通知发送到只有 10 个设备。 在处理调试都会发送，每分钟 10 也没有限制。

### <a name="review-telemetry"></a>查看遥测 ###

#### <a name="azure-portal"></a>Azure 门户 ####

在该门户中，可以快速了解通知中心的所有活动。

1. 在“概述”选项卡上，可以查看每个平台的注册、通知和错误的汇总视图。

   ![通知中心概述仪表板][5]

2. 在“监视器”选项卡上，可以添加许多其他平台特定指标，以便进行深入了解。 你可以专门介绍通知中心尝试将通知发送到推送通知服务时返回的错误。

   ![Azure 门户活动日志][6]

3. 首先查看“传入消息”、“注册操作”和“成功通知”。 然后转到每个平台选项卡查看特定于推送通知服务的错误。

4. 如果通知中心的身份验证设置不正确，则出现“PNS 身份验证错误”消息。 它表示要检查推送通知服务凭据。

#### <a name="programmatic-access"></a>以编程方式访问 ####

有关以编程方式访问的详细信息，请参阅[以编程方式访问](https://docs.microsoft.com/previous-versions/azure/azure-services/dn458823(v=azure.100))。

> [!NOTE]
> 与遥测相关的多项功能（例如，导出和导入注册、通过 API 进行遥测访问）只能在“标准”服务层级使用。 如果您尝试使用这些功能从免费或基本服务层，将获得一条异常消息，如果使用 SDK。 如果使用直接通过 REST Api 的功能，您将收到 HTTP 403 （禁止） 错误。
>
> 若要使用与遥测相关的功能，首先确保在 Azure 门户中使用标准服务层。  

<!-- IMAGES -->
[0]: ./media/notification-hubs-diagnosing/Architecture.png
[1]: ./media/notification-hubs-diagnosing/FCMConfigure.png
[3]: ./media/notification-hubs-diagnosing/FCMServerKey.png
[4]: ../../includes/media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png
[5]: ./media/notification-hubs-diagnosing/PortalDashboard.png
[6]: ./media/notification-hubs-diagnosing/PortalAnalytics.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[8]: ./media/notification-hubs-diagnosing/VSRegistrations.png
[9]: ./media/notification-hubs-diagnosing/VSServerExplorer.png
[10]: ./media/notification-hubs-diagnosing/VSTestNotification.png

<!-- LINKS -->
[通知中心概述]: notification-hubs-push-notification-overview.md
[Azure 通知中心入门]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[模板]: https://msdn.microsoft.com/library/dn530748.aspx
[APNs overview]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html
[About FCM messages]: https://firebase.google.com/docs/cloud-messaging/concept-options
[Export and modify registrations in bulk]: https://msdn.microsoft.com/library/dn790624.aspx
[Service Bus Explorer code]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a
[View device registrations for notification hubs]: https://msdn.microsoft.com/library/windows/apps/xaml/dn792122.aspx
[深入探讨：Visual Studio 2013 Update 2 RC 和 Azure SDK 2.3]: https://azure.microsoft.com/blog/2014/04/09/deep-dive-visual-studio-2013-update-2-rc-and-azure-sdk-2-3/#NotificationHubs
[宣布推出 Visual Studio 2013 Update 3 和 Azure SDK 2.4]: https://azure.microsoft.com/blog/2014/08/04/announcing-release-of-visual-studio-2013-update-3-and-azure-sdk-2-4/
[EnableTestSend]: https://docs.microsoft.com/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.enabletestsend?view=azure-dotnet
[Programmatic telemetry access]: https://msdn.microsoft.com/library/azure/dn458823.aspx
