---
title: WebJobs SDK 入门 - Azure
description: 用于事件驱动的后台处理的 WebJobs SDK 简介。 了解如何访问 Azure 服务和第三方服务中的数据。
services: app-service\web, storage
documentationcenter: .net
author: ggailey777
manager: jeconnoc
editor: ''
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: a9e902a13071abefff71dc4db089380979c8d459
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198619"
---
# <a name="get-started-with-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>用于事件驱动的后台处理的 Azure WebJobs SDK 入门

本文介绍如何使用 Visual Studio 2017 创建 Azure WebJobs SDK 项目、在本地运行它，然后将其部署到 [Azure 应用服务](overview.md)。 创建的项目是使用 WebJobs SDK 版本 3.x 的 .NET Core 控制台应用。 如果你对使用 .NET Framework 的版本 2.x 感兴趣，请参阅[使用 Visual Studio 开发和部署 WebJob - Azure 应用服务](webjobs-dotnet-deploy-vs.md)。

若要详细了解如何使用 WebJobs SDK，请参阅[如何使用 Azure WebJobs SDK 进行事件驱动的后台处理](webjobs-sdk-how-to.md)。

## <a name="prerequisites"></a>必备组件

* [安装](/visualstudio/install/)包含 **Azure 开发**工作负荷的 Visual Studio 2017。 如果已安装 Visual Studio，但未配置该工作负荷，请选择“工具”>“获取工具和功能”添加该工作负荷。

* 必须有一个 [Azure 帐户](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)才能将 WebJobs SDK 项目发布到 Azure。

## <a name="create-a-project"></a>创建一个项目

1. 在 Visual Studio 中，选择“文件”>“新建”>“项目”。

2. 选择“.NET Core”>“控制台应用(.NET Core)”。

3. 将项目命名为 *WebJobsSDKSample*，然后选择“确定”。

   ![“新建项目”对话框](./media/webjobs-sdk-get-started/new-project.png)

## <a name="webjobs-nuget-packages"></a>WebJobs NuGet 包

1. 安装以下 NuGet 包的最新稳定版本 3.x 版：

   * `Microsoft.Azure.WebJobs`
   * `Microsoft.Azure.WebJobs.Extensions`

     下面是适用于版本 3.0.4 的**包管理器控制台**命令：

     ```powershell
     Install-Package Microsoft.Azure.WebJobs -version 3.0.4
     Install-Package Microsoft.Azure.WebJobs.Extensions -version 3.0.1
     ```

## <a name="create-the-host"></a>创建主机

主机是函数的运行时容器，它侦听触发器并调用函数。 以下步骤创建一个实现 [`IHost`](/dotnet/api/microsoft.extensions.hosting.ihost) 的主机，它是 ASP.NET Core 中的通用主机。

1. 在 *Program.cs* 中，添加 `using` 语句：

    ```cs
    using Microsoft.Extensions.Hosting;
    ```

1. 将 `Main`方法替换为以下代码：

    ```cs
    static void Main(string[] args)
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        var host = builder.Build();
        using (host)
        {
            host.Run();
        }
    }
    ```

在 ASP.NET Core 中，通过调用 [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder) 实例上的方法来设置主机配置。 有关详细信息，请参阅 [.NET 通用主机](/aspnet/core/fundamentals/host/generic-host)。 `ConfigureWebJobs` 扩展方法初始化 WebJobs 主机。 在 `ConfigureWebJobs` 中，初始化特定的 WebJobs 扩展并设置这些扩展的属性。  

## <a name="enable-console-logging"></a>启用控制台日志记录

在本部分，设置使用 [ASP.NET Core 日志记录框架](/aspnet/core/fundamentals/logging)的控制台日志记录。

1. 安装以下 NuGet 包的最新稳定版本：

   * `Microsoft.Extensions.Logging` - 日志记录框架。
   * `Microsoft.Extensions.Logging.Console` - 用于将日志发送到控制台的控制台提供程序。

   下面是 2.2.0 版的“包管理器控制台”命令：

   ```powershell
   Install-Package Microsoft.Extensions.Logging -version 2.2.0
   ```

   ```powershell
   Install-Package Microsoft.Extensions.Logging.Console -version 2.2.0
   ```

1. 在 *Program.cs* 中，添加 `using` 语句：

   ```cs
   using Microsoft.Extensions.Logging;
   ```

1. 在 [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder) 上调用 [`ConfigureLogging`](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) 方法。 [`AddConsole`](/dotnet/api/microsoft.extensions.logging.consoleloggerextensions.addconsole) 方法将控制台日志记录添加到配置中。

    ```cs
    builder.ConfigureLogging((context, b) =>
    {
        b.AddConsole();
    });
    ```

    现在，`Main` 方法如下所示：

    ```cs
    static void Main(string[] args)
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();
                });
        var host = builder.Build();
        using (host)
        {
            host.Run();
        }
    }
    ```

    此项更新执行以下操作：

    * 禁用[仪表板日志记录](https://github.com/Azure/azure-webjobs-sdk/wiki/Queues#logs)。 仪表板是一个旧式监视工具，不建议对高吞吐量生产方案使用仪表板日志记录。
    * 使用默认[筛选](webjobs-sdk-how-to.md#log-filtering)添加控制台提供程序。

现在即可以添加由到达 [Azure 存储队列](../azure-functions/functions-bindings-storage-queue.md)的消息触发的函数。

## <a name="install-the-storage-binding-extension"></a>安装存储绑定扩展

从版本 3.x 开始，必须显式安装 WebJobs SDK 所需的存储绑定扩展。 在以前版本中，存储绑定已包含在 SDK 中。

1. 安装 [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 包的最新稳定版本，即 3.x 版。 

    下面是适用于版本 3.0.3 的**包管理器控制台**命令：

    ```powershell
    Install-Package Microsoft.Azure.WebJobs.Extensions.Storage -Version 3.0.3
    ```

2. 在 `ConfigureWebJobs` 扩展方法中，调用 [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder) 实例上的 `AddAzureStorage` 方法来初始化存储扩展。 此时，`ConfigureWebJobs` 方法如下例所示：

    ```cs
    builder.ConfigureWebJobs(b =>
                    {
                        b.AddAzureStorageCoreServices();
                        b.AddAzureStorage();
                    });
    ```

## <a name="create-a-function"></a>创建函数

1. 右键单击项目，选择“添加” > “新建项...”，选择“类”，将新的 C# 类文件命名为 *Functions.cs*，然后选择“添加”。

1. 在 Functions.cs 中，使用以下代码替换生成的模板：

   ```cs
   using Microsoft.Azure.WebJobs;
   using Microsoft.Extensions.Logging;

   namespace WebJobsSDKSample
   {
       public class Functions
       {
           public static void ProcessQueueMessage([QueueTrigger("queue")] string message, ILogger logger)
           {
               logger.LogInformation(message);
           }
       }
   }
   ```

   `QueueTrigger` 特性告知运行时，在名为 `queue` 的 Azure 存储队列中写入新消息时，应调用此函数。 队列消息的内容将提供给 `message` 参数中的方法代码。 在方法的正文中处理触发器数据。 在此示例中，代码只是记录消息。

   `message` 参数不一定要是字符串。 也可以绑定到 JSON 对象、字节数组或 [CloudQueueMessage](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage) 对象。 [参阅队列触发器用法](../azure-functions/functions-bindings-storage-queue.md#trigger---usage)。 每个绑定类型（例如队列、Blob 或表）具有一组可以绑定到的不同参数类型。

## <a name="create-a-storage-account"></a>创建存储帐户

在本地运行的 Azure 存储模拟器不具备 WebJobs SDK 所需的全部功能。 因此，在本部分，我们应在 Azure 中创建一个存储帐户，并将项目配置为使用该帐户。 如果已有一个存储帐户，请跳到步骤 6。

1. 在 Visual Studio 中打开“服务器资源管理器”并登录 Azure。 右键单击“Azure”节点，选择“连接到 Microsoft Azure 订阅”。

   ![登录 Azure](./media/webjobs-sdk-get-started/sign-in.png)

1. 在“服务器资源管理器”中的“Azure”节点下，右键单击“存储”并选择“创建存储帐户”。

   ![“创建存储帐户”菜单](./media/webjobs-sdk-get-started/create-storage-account-menu.png)

1. 在“创建存储帐户”对话框中，输入存储帐户的唯一名称。

1. 选择在其中创建了应用服务应用的同一**区域**或者靠近的区域。

1. 选择“创建”。

   ![创建存储帐户](./media/webjobs-sdk-get-started/create-storage-account.png)

1. 在“服务器资源管理器”中的“存储”节点下，选择新存储帐户。 在“属性”窗口中，选择“连接字符串”值字段右侧的省略号 (**...**)。

   ![“连接字符串”右侧的省略号](./media/webjobs-sdk-get-started/conn-string-ellipsis.png)

1. 复制连接字符串并将此值保存在某处，以方便再次复制。

   ![复制连接字符串](./media/webjobs-sdk-get-started/copy-key.png)

## <a name="configure-storage-to-run-locally"></a>将存储配置为在本地运行

WebJobs SDK 在 Azure 的“应用程序设置”中查找存储连接字符串。 在本地运行时，它会在本地配置文件或环境变量中查找此值。

1. 右键单击项目，选择“添加” > “新建项...”，选择“JavaScript JSON 配置文件”，将新文件命名为 *appsettings.json*，然后选择“添加”。 

1. 在新文件中添加 `AzureWebJobsStorage` 字段，如以下示例所示：

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}"
    }
    ```

1. 将 *{storage connection string}* 替换为先前复制的连接字符串。

1. 在解决方案资源管理器中选择“appsettings.json”文件，在“属性”窗口中，将“复制到输出目录”设置为“如果较新则复制”。

稍后，你将在 Azure 应用服务中的应用中添加相同的连接字符串应用设置。

## <a name="test-locally"></a>本地测试

在本部分，我们将生成并在本地运行项目，然后通过创建队列消息来触发函数。

1. 按 **Ctrl+F5** 运行项目。

   控制台显示运行时已找到函数，并等待队列消息触发该函数。 v3.x 主机生成以下输出：

   ```console
    info: Microsoft.Azure.WebJobs.Hosting.JobHostService[0]
          Starting JobHost
    info: Host.Startup[0]
          Found the following functions:
          WebJobsSDKSample.Functions.ProcessQueueMessage

    info: Host.Startup[0]
          Job host started
    Application started. Press Ctrl+C to shut down.
    Hosting environment: Development
    Content root path: C:\WebJobsSDKSample\WebJobsSDKSample\bin\Debug\netcoreapp2.1\
   ```

1. 关闭控制台窗口。

1. 在 Visual Studio 的“服务器资源管理器”中，展开新存储帐户所在的节点，然后右键单击“队列”。

1. 选择“创建队列”。

1. 输入 *queue* 作为队列名称，然后选择“确定”。

   ![创建队列](./media/webjobs-sdk-get-started/create-queue.png)

1. 右键单击新队列所在的节点，然后选择“查看队列”。

1. 选择“添加消息”图标。

   ![创建队列](./media/webjobs-sdk-get-started/create-queue-message.png)

1. 在“添加消息”对话框中，输入 *Hello World!* 作为**消息正文**，然后选择“确定”。 现在，队列中会出现一条消息。

   ![创建队列](./media/webjobs-sdk-get-started/hello-world-text.png)

1. 再次运行该项目。

   由于在 `ProcessQueueMessage` 函数中使用了 `QueueTrigger` 特性，因此 WeJobs SDK 运行时会在启动时侦听队列消息。 它会在名为 *queue* 的队列中查找新队列消息，并调用函数。

   由于[队列轮询指数退让](../azure-functions/functions-bindings-storage-queue.md#trigger---polling-algorithm)，运行时最长可能需要花费 2 分钟才能找到消息并调用函数。 以[开发模式](webjobs-sdk-how-to.md#host-development-settings)运行可以缩减此等待时间。

   控制台输出如下所示：

   ```console
    info: Function.ProcessQueueMessage[0]
          Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=2c319369-d381-43f3-aedf-ff538a4209b8)
    info: Function.ProcessQueueMessage[0]
          Trigger Details: MessageId: b00a86dc-298d-4cd2-811f-98ec39545539, DequeueCount: 1, InsertionTime: 1/18/2019 3:28:51 AM +00:00
    info: Function.ProcessQueueMessage.User[0]
          Hello World!
    info: Function.ProcessQueueMessage[0]
          Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=2c319369-d381-43f3-aedf-ff538a4209b8)
   ```

1. 关闭控制台窗口。 

1. 返回“队列”窗口并刷新。 该消息已消失，因为本地运行的函数已对其进行处理。 

## <a name="add-application-insights-logging"></a>添加 Application Insights 日志记录

在 Azure 中运行项目时，无法通过查看控制台输出来监视函数执行。 我们建议的监视解决方案是 [Application Insights](../azure-monitor/app/app-insights-overview.md)。 有关详细信息，请参阅[监视 Azure Functions](../azure-functions/functions-monitoring.md)。

在本部分，我们将执行以下任务来设置 Application Insights 日志记录，然后部署到 Azure：

* 请确保已准备好要使用的应用服务应用和 Application Insights 实例。
* 将应用服务应用配置为使用前面创建的 Application Insights 实例和存储帐户。
* 设置项目，以便将日志记录到 Application Insights。

### <a name="create-app-service-app-and-application-insights-instance"></a>创建应用服务应用和 Application Insights 实例

1. 如果没有可用的应用服务应用，请[创建一个](app-service-web-get-started-dotnet-framework.md)。 创建应用时，还可以创建已连接的 Application Insights 资源。 执行此操作时，系统会在应用中为你设置 `APPINSIGHTS_INSTRUMENTATIONKEY`。

1. 如果没有可用的 Application Insights 资源，请[创建一个](../azure-monitor/app/create-new-resource.md )。 将“应用程序类型”设置为“常规”，并跳过“复制检测密钥”后面的部分。

1. 如果已有可用的 Application Insights 资源，请[复制检测密钥](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key)。

### <a name="configure-app-settings"></a>配置应用设置 

1. 在 Visual Studio 的“服务器资源管理器”中，展开“Azure”下面的“应用服务”节点。

1. 展开应用服务应用所在的资源组，然后右键单击应用服务应用。

1. 选择“查看设置”。

1. 在“连接字符串”框中添加以下条目。

   |名称  |连接字符串  |数据库类型|
   |---------|---------|------|
   |AzureWebJobsStorage | {前面复制的存储连接字符串}|“自定义”|

1. 如果“应用程序设置”框中没有 Application Insights 检测密钥，请添加前面复制的检测密钥。 （根据应用服务应用的创建方式，该框中可能已包含检测密钥。）

   |名称  |值  |
   |---------|---------|
   |APPINSIGHTS_INSTRUMENTATIONKEY | {instrumentation key} |

1. 将 *{instrumentation key}* 替换为所用 Application Insights 资源中的检测密钥。

1. 选择“保存”。

1. 向项目添加 Application Insights 连接，以便在本地运行。 在“appsettings.json”文件中，添加 `APPINSIGHTS_INSTRUMENTATIONKEY` 字段，如下例所示：

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}",
        "APPINSIGHTS_INSTRUMENTATIONKEY": "{instrumentation key}"
    }
    ```

    将 *{instrumentation key}* 替换为所用 Application Insights 资源中的检测密钥。

1. 保存所做更改。

### <a name="add-application-insights-logging-provider"></a>添加 Application Insights 日志记录提供程序

若要利用 [Application Insights](../azure-monitor/app/app-insights-overview.md) 日志记录，请更新日志记录代码以执行以下操作：

* 使用默认[筛选](webjobs-sdk-how-to.md#log-filtering)添加 Application Insights 日志记录提供程序；在本地运行时，所有“信息”和更高级别的日志将转到控制台和 Application Insights。
* 将 `LoggerFactory` 对象放在 `using` 块中，以确保在主机退出时刷新日志输出。

1. 为 Application Insights 日志记录提供程序安装 NuGet 包的最新稳定版本 3.x 版：`Microsoft.Azure.WebJobs.Logging.ApplicationInsights`。

   下面是 3.0.2 版的“包管理器控制台”命令：

   ```powershell
   Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version 3.0.2
   ```

1. 打开“Program.cs”并使用以下代码替换 `Main` 方法中的代码：

    ```cs
    static void Main(string[] args)
    {
        var builder = new HostBuilder();
        builder.UseEnvironment(EnvironmentName.Development);
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                    b.AddAzureStorage();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();

                    // If the key exists in settings, use it to enable Application Insights.
                    string instrumentationKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
                    if (!string.IsNullOrEmpty(instrumentationKey))
                    {
                        b.AddApplicationInsights(o => o.InstrumentationKey = instrumentationKey);
                    }
                });
        var host = builder.Build();
        using (host)
        {
            host.Run();
        }
    }
    ```

    这将 Application Insights 提供程序添加到日志记录，使用先前添加到你的应用设置的密钥。

## <a name="test-application-insights-logging"></a>测试 Application Insights 日志记录

在本部分，我们将再次在本地运行，以验证日志记录数据现在是否转到 Application Insights 和控制台。

1. 像[前面](#trigger-the-function-in-azure)所做的那样，在 Visual Studio 中使用“服务器资源管理器”创建队列消息，不过这次要输入 Hello App Insights! 作为消息文本。

1. 运行该项目。

   WebJobs SDK 将处理队列消息，控制台窗口中会显示日志。

1. 关闭控制台窗口。

1. 打开 [Azure 门户](https://portal.azure.com/)并转到 Application Insights 资源。

1. 选择“搜索”。

   ![选择“搜索”](./media/webjobs-sdk-get-started/select-search.png)

1. 如果未看到“Hello App Insights!” 消息，请定期选择“刷新”几分钟。 （日志不会立即显示，因为 Application Insights 客户端需要花费片刻时间来刷新它处理的日志。）

   ![Application Insights 中的日志](./media/webjobs-sdk-get-started/logs-in-ai.png)

1. 关闭控制台窗口。

## <a name="deploy-as-a-webjob"></a>部署到 Azure

在部署期间，可以创建一个要在其中运行函数的应用服务实例。 将 .NET Core 控制台应用发布到 Azure 中的应用服务时，该应用会自动以 WebJob 的形式运行。 若要详细了解发布过程，请参阅[使用 Visual Studio 开发和部署 WebJob](webjobs-dotnet-deploy-vs.md)。

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)]

## <a name="trigger-the-function-in-azure"></a>触发 Azure 中的函数

1. 确保不是在本地运行（如果控制台窗口仍旧打开，请将其关闭）。 否则，本地实例可能是处理所创建的任何队列消息的第一个实例。

1. 在 Visual Studio 的“队列”页中，像以前一样向队列添加消息。

1. 刷新“队列”页后新消息消失，因为它已由 Azure 中运行的函数处理。

   > [!TIP]
   > 若要在 Azure 中进行测试，请使用[开发模式](webjobs-sdk-how-to.md#host-development-settings)来确保立即调用队列触发函数，并避免[队列轮询指数退让](../azure-functions/functions-bindings-storage-queue.md#trigger---polling-algorithm)导致的延迟。

### <a name="view-logs-in-application-insights"></a>在 Application Insights 中查看日志

1. 打开 [Azure 门户](https://portal.azure.com/)并转到 Application Insights 资源。

1. 选择“搜索”。

1. 如果未看到“Hello Azure!” 消息，请定期选择“刷新”几分钟。

   查看 WebJob 中运行的函数发出的日志，包括在上一部分中输入的 *Hello Azure!* 文本。

## <a name="add-an-input-binding"></a>添加输入绑定

输入绑定可以简化读取数据的代码。 在本示例中，队列消息是 Blob 名称，我们将使用该 Blob 名称来查找和读取 Azure 存储中的 Blob。

1. 在 *Functions.cs* 中，将 `ProcessQueueMessage` 方法替换为以下代码：

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
   }
   ```

   在此代码中，`queueTrigger` 是[绑定表达式](../azure-functions/functions-bindings-expressions-patterns.md)，意味着它将在运行时解析为不同的值。  在运行时，它具有队列消息的内容。

1. 添加 `using`：

   ```cs
   using System.IO;
   ```

1. 在存储帐户中创建 Blob 容器。

   a. 在 Visual Studio 的“服务器资源管理器”中，展开你的存储帐户所在的节点，右键单击“Blob”，并选择“创建 Blob 容器”。

   b. 在“创建 Blob 容器”对话框中，输入 *container* 作为容器名称，然后单击“确定”。

1. 将 *Program.cs* 文件上传到 Blob 容器。 （此处使用的文件用作示例；可以上传任何文本文件，并使用该文件的名称创建队列消息。）

   a. 在“服务器资源管理器”中，双击创建的容器所在的节点。

   b. 在“容器”窗口中，选择“上传”按钮。

   ![Blob 上传按钮](./media/webjobs-sdk-get-started/blob-upload-button.png)

   c. 找到并选择“Program.cs”，然后选择“确定”。

1. 在前面创建的队列中创建队列消息，并使用 *Program.cs* 作为消息的文本。

   ![队列消息 Program.cs](./media/webjobs-sdk-get-started/queue-msg-program-cs.png)

1. 在本地运行项目

   该队列消息会触发函数，而该函数又会读取 Blob 并记录其长度。 控制台输出如下所示：

   ```console
   Found the following functions:
   ConsoleApp1.Functions.ProcessQueueMessage
   Job host started
   Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   Blob name:Program.cs
   Size: 532 bytes
   Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   ```

## <a name="add-an-output-binding"></a>添加输出绑定

输出绑定可以简化写入数据的代码。 本示例在前一个示例的基础上做了修改，它会写入 Blob 的副本，而不是记录其大小。 Blob 存储绑定包含在我们之前安装的 Azure 存储扩展包中。

1. 将 `ProcessQueueMessage`方法替换为以下代码：

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       [Blob("container/copy-{queueTrigger}", FileAccess.Write)] Stream outputBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
       myBlob.CopyTo(outputBlob);
   }
   ```

1. 创建另一个队列消息，并使用 *Program.cs* 作为消息的文本。

1. 在本地运行项目

   该队列消息会触发函数，而该函数又会读取 Blob、记录其长度并创建新 Blob。 控制台输出相同，但在转到 Blob 容器窗口并选择“刷新”时，会看到名为 *copy-Program.cs* 的新 Blob。

## <a name="republish-the-updates-to-azure"></a>将更新重新发布到 Azure

1. **在“解决方案资源管理器”** 中，右键单击该项目并选择“发布”。

1. 在“发布”对话框中，确保当前配置文件已选中，然后选择“发布”。 “输出”窗口中会详细显示发布结果。
 
1. 通过再次将某个文件上传到 Blob 容器，并将一条消息添加到与所上传文件同名的队列，来验证 Azure 中的函数。 将会看到，该消息已从队列中删除，并且 Blob 容器中创建了该文件的副本。 

## <a name="next-steps"></a>后续步骤

本文介绍了如何创建、运行和部署 WebJobs SDK 3.x 项目。

> [!div class="nextstepaction"]
> [详细了解 WebJobs SDK](webjobs-sdk-how-to.md)
