---
title: 介绍如何将 Azure SignalR 服务与 ASP.NET 配合使用的快速入门
description: 有关如何使用 Azure SignalR 服务通过 ASP.NET 框架创建聊天室的快速入门。
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 04/20/2019
ms.author: zhshang
ms.openlocfilehash: cda1eddc5fa40e97038274cf9b4c3c2cb9552871
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154503"
---
# <a name="quickstart-create-a-chat-room-with-aspnet-and-signalr-service"></a>快速入门：使用 ASP.NET 和 SignalR 服务创建聊天室

Azure SignalR 服务基于[适用于 ASP.NET Core 2.0 的 SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction)，后者**并非**与 ASP.NET SignalR 100% 兼容。 Azure SignalR 服务基于最新的 ASP.NET Core 技术重新实现了 ASP.NET SignalR 数据协议。 使用用于 ASP.NET SignalR 的 Azure SignalR 服务时，某些 ASP.NET SignalR 功能不再受支持，例如 Azure SignalR 在客户端重新连接时不重播消息。 另外，Forever Frame 传输和 JSONP 也不受支持。 若要使 ASP.NET SignalR 应用程序兼容 SignalR 服务，必须进行一些代码更改并确保所依赖库的版本正确。 

请参阅[版本差异文档](https://docs.microsoft.com/aspnet/core/signalr/version-differences?view=aspnetcore-2.2)，获取在 ASP.NET SignalR 和 ASP.NET Core SignalR 之间进行的功能比较的完整列表。

本快速入门介绍如何从 ASP.NET 和 Azure SignalR 服务着手来创建类似的[聊天室应用程序](./signalr-quickstart-dotnet-core.md)。


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
## <a name="prerequisites"></a>先决条件

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
* [.NET 4.6.1](https://www.microsoft.com/net/download/windows)
* [ASP.NET SignalR 2.4.1](https://www.nuget.org/packages/Microsoft.AspNet.SignalR/)

## <a name="sign-in-to-azure"></a>登录 Azure

使用 Azure 帐户登录到 [Azure 门户](https://portal.azure.com/)。

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

ASP.NET SignalR 应用程序不支持无服务器模式。 对于 Azure SignalR 服务实例，请始终使用“默认”或“经典”。

也可根据[创建 SignalR 服务脚本](scripts/signalr-cli-create-service.md)中的说明，创建在本快速入门中使用的 Azure 资源。

## <a name="clone-the-sample-application"></a>克隆示例应用程序

在部署该服务时，让我们切换到使用代码。 克隆[来自 GitHub 的示例应用](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom)，设置 SignalR 服务连接字符串，并在本地运行该应用程序。

1. 打开 git 终端窗口。 切换到要克隆示例项目的文件夹。

1. 运行下列命令以克隆示例存储库。 此命令在计算机上创建示例应用程序的副本。

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```

## <a name="configure-and-run-chat-room-web-app"></a>配置并运行聊天室 Web 应用

1. 启动 Visual Studio，并打开所克隆存储库的 *aspnet-samples/ChatRoom/* 文件夹中的解决方案。

1. 在打开了 Azure 门户的浏览器中，查找并选择所创建的实例。

1. 选择“密钥”以查看 SignalR 服务实例的连接字符串。

1. 选择并复制主连接字符串。

1. 现在请在 web.config 文件中设置连接字符串。

    ```xml
    <configuration>
    <connectionStrings>
        <add name="Azure:SignalR:ConnectionString" connectionString="<Replace By Your Connection String>"/>
    </connectionStrings>
    ...
    </configuration>
    ```

1. 需在 *Startup.cs* 中调用 `MapAzureSignalR({your_applicationName})` 而不是 `MapSignalR()`，传入连接字符串，使应用程序连接到服务，而不是自行托管 SignalR。 将 `{YourApplicationName}` 替换为应用程序的名称。 此名称是独一无二的名称，可以将此应用程序与其他应用程序区别开来。 可以使用 `this.GetType().FullName` 作为值。

    ```cs
    public void Configuration(IAppBuilder app)
    {
        // Any connection or hub wire up and configuration should go here
        app.MapAzureSignalR(this.GetType().FullName);
    }
    ```

    此外还需在使用这些 API 之前参考服务 SDK。 打开“工具”|“NuGet 包管理器”|“包管理器控制台”，然后运行以下命令：

    ```powershell
    Install-Package Microsoft.Azure.SignalR.AspNet
    ```

    除了这些更改，所有其他功能保持不变，你仍然可以使用已经熟悉的中心界面来编写业务逻辑。

    > [!NOTE]
    > 在实现时，Azure SignalR 服务 SDK 会公开用于协商的终结点 `/signalr/negotiate`。 它会在客户端尝试连接时返回特殊的协商响应，将客户端重定向到连接字符串中定义的服务终结点。

1. 按 **F5** 以调试模式运行项目。 可以看到应用程序在本地运行。 它现在会连接到 Azure SignalR 服务，而不是由应用程序自身来托管 SignalR 运行时。

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]



> [!IMPORTANT]
> 删除资源组的操作不可逆，资源组以及其中的所有资源将被永久删除。 请确保不会意外删除错误的资源组或资源。 如果在现有资源组（其中包含要保留的资源）中为托管此示例而创建了相关资源，可从各自的边栏选项卡逐个删除这些资源，而不要删除资源组。
> 
> 

登录到 [Azure 门户](https://portal.azure.com)，并单击“资源组”。

在“按名称筛选...”文本框中键入资源组的名称。 本快速入门的说明使用了名为“SignalRTestResources”的资源组。 在结果列表中的资源组上，单击“...”，然后单击“删除资源组”。

   
![删除](./media/signalr-quickstart-dotnet-core/signalr-delete-resource-group.png)

片刻之后，将会删除该资源组及其包含的所有资源。

## <a name="next-steps"></a>后续步骤

在本快速入门中，我们创建了一个新的 Azure SignalR 服务资源，并将其与 ASP.NET Web 应用配合使用。 接下来，需了解如何将 Azure SignalR 服务与 ASP.NET Core 配合使用，以便开发实时应用程序。

> [!div class="nextstepaction"]
> [将 Azure SignalR 服务与 ASP.NET Core 配合使用](./signalr-quickstart-dotnet-core.md)
