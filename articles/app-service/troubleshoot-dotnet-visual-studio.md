---
title: 使用 Visual Studio 对应用进行故障排除 - Azure 应用服务
description: 了解如何通过内置于 Visual Studio 2013 的远程调试、跟踪和日志记录工具排除应用服务应用的故障。
services: app-service
documentationcenter: .net
author: cephalin
manager: cfowler
editor: ''
ms.assetid: def8e481-7803-4371-aa55-64025d116c97
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 08/29/2016
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: cbf6a44f1a3210906ec7ab0d04eecb997bc2c470
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412817"
---
# <a name="troubleshoot-an-app-in-azure-app-service-using-visual-studio"></a>使用 Visual Studio 对 Azure 应用服务中的应用进行故障排除
## <a name="overview"></a>概述
本教程介绍如何使用 Visual Studio 工具，通过远程运行[调试模式](https://docs.microsoft.com/visualstudio/debugger/)或查看应用程序日志和 Web 服务器日志，帮助调试[应用服务](https://go.microsoft.com/fwlink/?LinkId=529714)中的应用。

学习内容：

* Visual Studio 中提供的应用管理功能。
* 如何使用 Visual Studio 远程视图在远程应用中进行快速更改。
* 项目在 Azure 中运行时，如何对应用和 WebJob 远程运行调试模式。
* 如何创建应用程序跟踪日志并在该应用程序创建日志时对其进行查看。
* 如何查看 Web 服务器日志，包括详细的错误消息和失败请求跟踪。
* 如何将诊断日志发送至 Azure 存储帐户并在其中进行查看。

如果有 Visual Studio Ultimate，还可以使用 [IntelliTrace](/visualstudio/debugger/intellitrace) 进行调试。 本教程未介绍 IntelliTrace。

## <a name="prerequisites"></a>先决条件
本教程适用于在[在 Azure 应用服务中创建 ASP.NET 应用](app-service-web-get-started-dotnet-framework.md)中设置的开发环境、Web 项目和应用服务应用。 对于 WebJobs 部分，需要用到在 [Azure WebJobs SDK 入门][GetStartedWJ]中创建的应用程序。

在本教程中所示的代码示例适用于 C# MVC Web 应用程序，但对于 Visual Basic 和 Web 窗体应用程序，故障排除过程是一样的。

本教程假定你使用 Visual Studio 2019。 

流式日志功能仅适用于面向 .NET Framework 4 或更高版本的应用程序。

## <a name="sitemanagement"></a>应用配置和管理
通过 Visual Studio，用户可以访问 [Azure 门户](https://go.microsoft.com/fwlink/?LinkId=529715)中提供的一部分应用管理功能和配置设置。 本节介绍使用“服务器资源管理器”可以实现的功能。 若要了解最新的 Azure 集成功能，请同时试用**云资源管理器**。 可以从“视图”菜单打开这两个窗口。

1. 如果还没有登录 Visual Studio 中的 Azure ，右键单击“Azure”，然后选择连接到“服务器资源管理器”中的“Microsoft Azure 订阅”。

    另一种方法是安装可以访问帐户的管理证书。 如果选择安装证书，请右键单击“服务器资源管理器”中的“Azure”，然后选择上下文菜单中的“管理和筛选订阅”。 在“管理 Microsoft Azure 订阅”对话框中，单击“证书”选项卡，然后单击“导入”。 按照说明为 Azure 帐户下载并导入一个订阅文件（也称为 *.publishsettings* 文件）。

   > [!NOTE]
   > 将此订阅文件下载并保存到源代码目录之外的文件夹中（例如，在 Downloads 文件夹中），然后在导入完成后将其删除。 获得此订阅文件访问权的恶意用户可以编辑、创建和删除 Azure 服务。
   >
   >

    有关从 Visual Studio 连接至 Azure 资源的详细信息，请参阅[管理帐户、订阅和管理角色](https://go.microsoft.com/fwlink/?LinkId=324796#BKMK_AccountVCert)。
2. 在“服务资源管理器”中，展开“Azure”，并展开“应用服务”。
3. 展开包含在[在 Azure 应用服务中创建 ASP.NET 应用](app-service-web-get-started-dotnet-framework.md)中创建的应用的资源组，右键单击该应用节点，并单击“查看设置”。

    ![在服务器资源管理器中查看设置](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewsettings.png)

    此时会显示“Azure Web 应用”选项卡，可在此处查看 Visual Studio 中提供的应用管理和配置任务。

    ![“Azure Web 应用”窗口](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configtab.png)

    本教程介绍如何使用日志记录和跟踪下拉列表。 其中，还将用到远程调试，但会通过不同方式启用该调试。

    有关此窗口中“应用设置”框和“连接字符串”框的信息，请参见 [Azure 应用服务：应用程序字符串和连接字符串的工作原理](https://azure.microsoft.com/blog/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)。

    如果要执行的应用管理任务无法在此窗口进行，请单击“在管理门户中打开”，以便在浏览器窗口中打开 Azure 门户。

## <a name="remoteview"></a>在服务器资源管理器中访问应用文件
部署 Web 项目时，Web.config 文件中的 `customErrors` 标志通常设置为 `On` 或 `RemoteOnly`，这意味着如果出现问题你不会收到任何有帮助的错误消息。 无论发生何种错误，获得的都是类似如下所示的页面：

**'/' 应用程序中出现服务器错误：**

![无用的错误页](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror.png)

**发生错误：**

![无用的错误页](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror1.png)

**网站无法显示页面**

![无用的错误页](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror2.png)

找出错误原因的最简捷方法通常是启用详细错误消息，之前保留的屏幕截图中的第一个截图会告诉你如何做。 而实现此目的需要在部署的 Web.config 文件中进行更改。 可以编辑项目中的 *Web.config* 文件，并重新部署该项目，或创建 [Web.config 转换](https://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations)并部署调试版本，但还有一个更快捷的方式：在“解决方案资源管理器”中，可以通过使用*远程视图*功能直接查看并编辑远程应用中的文件。

1. 在“服务器资源管理器”中，依次展开“Azure”、“应用服务”、应用所在的资源组以及应用的节点。

    可看到允许访问应用的内容文件和日志文件的节点。
2. 展开“文件”节点，并双击 *Web.config* 文件。

    ![打开 Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfig.png)

    Visual Studio 将从远程应用打开 Web.config 文件，并在标题栏中的文件名旁显示 [Remote]。
3. 将以下行添加到 `system.web` 元素：

    `<customErrors mode="Off"></customErrors>`

    ![编辑 Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfigedit.png)
4. 刷新显示无用错误消息的浏览器，现在会获得详细的错误消息，如下所示：

    ![详细错误消息](./media/web-sites-dotnet-troubleshoot-visual-studio/detailederror.png)

    （通过以红色显示的行添加到 *Views\Home\Index.cshtml* 创建显示的错误。）

能够读取并编辑应用服务应用上的文件会使故障排除变得更简单，而编辑 Web.config 文件只是这类方案的示例之一。

## <a name="remotedebug"></a>远程调试应用
如果详细的错误消息提供的信息量不够，且无法本地重新创建该错误，则可以采用远程运行调试模式进行故障排除. 可以设置断点、直接操作内存、逐行执行代码，甚至更改代码路径。

远程调试不适用于 Visual Studio Express 版。

本部分介绍如何使用在[在 Azure 应用服务中创建 ASP.NET 应用](app-service-web-get-started-dotnet-framework.md)中创建的项目进行远程调试。

1. 打开在[在 Azure 应用服务中创建 ASP.NET 应用](app-service-web-get-started-dotnet-framework.md)中创建的 Web 项目。

2. 打开 *Controllers\HomeController.cs*。

3. 删除 `About()` 方法并在其位置插入以下代码。

``` c#
public ActionResult About()
{
    string currentTime = DateTime.Now.ToLongTimeString();
    ViewBag.Message = "The current time is " + currentTime;
    return View();
}
```

1. 在 `ViewBag.Message` 行上[设置一个断点](https://docs.microsoft.com/visualstudio/debugger/)。

1. 在“解决方案资源管理器”中，右键单击该项目并单击“发布”。

1. 在“配置文件”下拉列表中，选择在[在 Azure 应用服务中创建 ASP.NET 应用](app-service-web-get-started-dotnet-framework.md)中使用的相同配置文件。 然后，单击“设置”。

1. 在“发布”对话框中，单击“设置”选项卡，将“配置”更改为“调试”，然后单击“保存”。

    ![在调试模式下发布](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-publishdebug.png)

1. 单击“发布” 。 部署结束后，浏览器将打开应用的 Azure URL，此时关闭浏览器。

1. 在“服务器资源管理器”中，右键单击应用，并单击“附加调试器”。

    ![附加调试程序](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-attachdebugger.png)

    浏览器会自动打开运行在 Azure 中的主页。 可能需要等待大约 20 秒，以便 Azure 针对调试设置服务器。 此延迟只在 48 小时内首次于应用上运行调试模式时出现。 如果在同一时段再次开始调试，不会发生延迟。

    > [!NOTE] 
    > 如果启动调试器时出现问题，请尝试使用“云资源管理器”而不是“服务器资源管理器”执行此操作。
    >

1. 单击菜单中的“关于”。

    Visual Studio 会在断点处停止，代码在 Azure 中运行，而不是在本地计算机上。

1. 将鼠标悬停在 `currentTime` 变量上查看时间值。

    ![在运行于 Azure 中的调试模式下查看变量](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugviewinwa.png)

    看到的时间是 Azure 服务器时间，可能与本地计算机所处时区不同。

1. 为 `currentTime` 变量输入一个新值，如“Now running in Azure”。

1. 按 F5 继续运行。

     运行于 Azure 中的“关于”页面会显示在 currentTime 变量中输入的新值。

     ![显示新值的关于页面](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugchangeinwa.png)

## <a name="remotedebugwj"></a> 远程调试 WebJobs
本部分介绍如何使用在 [Azure WebJobs SDK 入门](https://github.com/Azure/azure-webjobs-sdk/wiki)中创建的项目和应用进行远程调试。

本部分所示的功能只能在 Visual Studio 2013 Update 4 或更高版本中使用。

远程调试仅适用于连续 WebJobs。 计划的和按需 WebJobs 不支持调试。

1. 打开在 [Azure WebJobs SDK 入门][GetStartedWJ]中创建的 Web 项目。

2. 在 ContosoAdsWebJob 项目中，打开 *Functions.cs*。

3. 在 `GnerateThumbnail` 方法的第一个语句中[设置一个断点](https://docs.microsoft.com/visualstudio/debugger/)。

    ![设置断点](./media/web-sites-dotnet-troubleshoot-visual-studio/wjbreakpoint.png)

4. 在“解决方案资源管理器”中，右键单击该 Web 项目（而非 WebJob 项目），并单击“发布”。

5. 在“配置文件”下拉列表中，选择与 [Azure WebJobs SDK 入门](https://github.com/Azure/azure-webjobs-sdk/wiki)中所用相同的配置文件。

6. 单击“设置”选项卡，将“配置”更改为“调试”，然后单击“发布”。

    Visual Studio 将部署 Web 和 WebJob 项目，浏览器将打开应用的 Azure URL。

7. 在“服务器资源管理器”中，展开“Azure”>“应用服务”> 你的资源组 > 你的应用 >“WebJobs”>“连续”，然后右键单击“ContosoAdsWebJob”。

8. 单击“附加调试器”。

    ![附加调试程序](./media/web-sites-dotnet-troubleshoot-visual-studio/wjattach.png)

    浏览器会自动打开运行在 Azure 中的主页。 可能需要等待大约 20 秒，以便 Azure 针对调试设置服务器。 此延迟只在 48 小时内首次于应用上运行调试模式时出现。 如果在同一时段再次开始调试，不会发生延迟。

9. 在打开 Contoso 广告主页的 Web 浏览器中，创建新的广告。

    创建广告会导致创建队列消息，WebJob 拾取并处理该消息。 当 WebJobs SDK 调用函数处理该队列消息时，代码命中断点。

10. 当调试器在断点处中断时，可以在程序运行云的同时，检查并更改变量值。 在下图中，调试器显示了传递给 `GenerateThumbnail` 方法的 blobInfo 对象的内容。

     ![调试器中的 blobInfo 对象](./media/web-sites-dotnet-troubleshoot-visual-studio/blobinfo.png)

11. 按 F5 继续运行。

     `GenerateThumbnail` 方法将完成创建缩略图。

12. 在浏览器中刷新“索引”页，会看到缩略图。

13. 在 Visual Studio 中，按 SHIFT+F5 停止调试。

14. 在“服务器资源管理器”中，右键单击 ContosoAdsWebJob 节点，并单击“查看仪表板”。

15. 使用 Azure 凭据登录，并单击 WebJob 名称转到 WebJob 的页面。

     ![单击 ContosoAdsWebJob](./media/web-sites-dotnet-troubleshoot-visual-studio/clickcaw.png)

     仪表板会显示最近执行的 `GenerateThumbnail` 函数。

     （下次单击“查看仪表板”时无需登录，浏览器会直接转到 WebJob 的页面。）

16. 单击函数名称可查看有关函数执行的详细信息。

     ![函数详细信息](./media/web-sites-dotnet-troubleshoot-visual-studio/funcdetails.png)

如果将函数[写入了日志](https://github.com/Azure/azure-webjobs-sdk/wiki)，可以单击“ToggleOutput”查看日志。

## <a name="notes-about-remote-debugging"></a>有关远程调试的说明

* 不建议在生产环境中以调试模式运行。 如果生产应用未进行横向扩展以容纳多个服务器实例，则调试会阻止 Web 服务器响应其他请求。 如果具有多个 Web 服务器实例，在附加至调试程序时会获得一个随机实例，但无法确保后续浏览器请求将前往相同的实例。 此外，调试版本一般不会部署到生产环境，针对版本生成的编译器优化可以逐行显示源代码中出现的情况。 至于如何解决生产环境中出现的问题，可利用的最佳资源是应用程序跟踪和 Web 服务器日志。
* 远程调试时避免长时间停止在断点处。 Azure 会将停止时间超过几分钟的进程视为无反应进程而将其关闭。
* 进行调试的时候，服务器会向 Visual Studio 发送数据，这可能会影响到带宽费用。 有关带宽费率的信息，请参阅 [Azure 定价](https://azure.microsoft.com/pricing/calculator/)。
* 请确保 *Web.config* 文件中 `compilation` 元素的 `debug` 属性设置为 true。 在发布调试版本配置时，默认设置为 true。

``` xml
<system.web>
  <compilation debug="true" targetFramework="4.5" />
  <httpRuntime targetFramework="4.5" />
</system.web>
```
* 如果发现调试程序没有针对希望调试的代码展开行动，可能需要更改“仅我的代码”设置。  有关详细信息，请参阅[指定是否仅使用 Visual Studio 中的“仅我的代码”调试用户代码](https://docs.microsoft.com/visualstudio/debugger/just-my-code)。
* 启用远程调试功能时，服务器上会出现一个计时器，48 小时后该功能会自动关闭。 这一 48 小时的限制是出于安全性与性能的考虑。 可以根据需要轻松地多次重启该功能。 我们建议，在不主动进行调试的时候，保持其处于禁用状态。
* 可以手动将调试器附加到任何进程，而不仅仅是应用进程 (w3wp.exe)。 有关如何在 Visual Studio 中使用调试模式的信息，请参阅[在 Visual Studio 中进行调试](/visualstudio/debugger/debugging-in-visual-studio)。

## <a name="logsoverview"></a>诊断日志概述
在应用服务应用中运行的 ASP.NET 应用程序可以创建以下几类日志：

* **应用程序跟踪日志**<br/>
  应用程序通过调用 [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) 类的方法创建这些日志。
* **Web 服务器日志**<br/>
  Web 服务器为向应用发出的每个 HTTP 请求创建一个日志条目。
* **详细错误消息日志**<br/>
  Web 服务器为失败的 HTTP 请求（导致状态代码 400 或更大数字的请求）创建带有一些额外信息的 HTML 页面。
* **失败请求跟踪日志**<br/>
  Web 服务器为失败的 HTTP 请求创建带有详细跟踪信息的 XML 文件。 Web 服务器还可提供 XSL 文件以格式化浏览器中的 XML。

日志记录会影响应用性能，因此 Azure 允许根据需要启用或禁用各种类型的日志。 对于应用程序日志，可以指定只写入特定严重级别以上的日志。 创建新的应用时，默认禁用所有日志记录。

日志将写入到应用文件系统中 *LogFiles* 文件夹内的文件，并可通过 FTP 访问。 Web 服务器日志和应用程序日志也可写入到 Azure 存储帐户。 可在存储帐户中为日志留出大于文件系统预留量的空间。 使用文件系统时，最多可存储 100 兆字节的日志。 （文件系统日志仅适合短期保留。 达到限制后，Azure 将删除旧日志文件以便为新日志腾出空间。）  

## <a name="apptracelogs"></a>创建并查看应用程序跟踪日志
在本部分中执行以下任务：

* 将跟踪语句添加到在 [Azure 和 ASP.NET 入门](app-service-web-get-started-dotnet-framework.md)中创建的 Web 项目。
* 在本地运行该项目时查看日志。
* 查看由运行于 Azure 中的应用程序生成的日志。

有关如何在 Web 作业中创建应用程序日志的信息，请参阅[如何使用 WebJobs SDK 处理 Azure 队列存储 - 如何写入日志](https://github.com/Azure/azure-webjobs-sdk/wiki)。 以下有关查看日志以及控制其在 Azure 中的存储方式的说明，同样适用于 WebJobs 创建的应用程序日志。

### <a name="add-tracing-statements-to-the-application"></a>向应用程序添加跟踪语句
1. 打开 *Controllers\HomeController.cs* 并将 `Index`、`About` 和 `Contact` 方法替换为以下代码，以便为 `System.Diagnostics` 添加 `Trace` 语句和 `using` 语句：

```c#
public ActionResult Index()
{
    Trace.WriteLine("Entering Index method");
    ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
    Trace.TraceInformation("Displaying the Index page at " + DateTime.Now.ToLongTimeString());
    Trace.WriteLine("Leaving Index method");
    return View();
}

public ActionResult About()
{
    Trace.WriteLine("Entering About method");
    ViewBag.Message = "Your app description page.";
    Trace.TraceWarning("Transient error on the About page at " + DateTime.Now.ToShortTimeString());
    Trace.WriteLine("Leaving About method");
    return View();
}

public ActionResult Contact()
{
    Trace.WriteLine("Entering Contact method");
    ViewBag.Message = "Your contact page.";
    Trace.TraceError("Fatal error on the Contact page at " + DateTime.Now.ToLongTimeString());
    Trace.WriteLine("Leaving Contact method");
    return View();
}        
```

1. 将 `using System.Diagnostics;` 语句添加到文件顶部。

### <a name="view-the-tracing-output-locally"></a>本地查看跟踪输出
1. 按 F5 以调试模式运行应用程序。

    默认跟踪侦听器将所有跟踪输出写入“输出”窗口，同时还有其他调试输出。 下图显示了来自添加到 `Index` 方法的跟踪语句的输出。

    ![在调试窗口中进行跟踪](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugtracing.png)

    以下步骤介绍如何在网页中查看跟踪输出而无需在调试模式下进行编译。
2. 打开应用程序 Web.config 文件（位于项目文件夹中），将 `<system.diagnostics>` 元素添加到文件末尾的关闭 `</configuration>` 元素之前：

``` xml
<system.diagnostics>
<trace>
  <listeners>
    <add name="WebPageTraceListener"
        type="System.Web.WebPageTraceListener,
        System.Web,
        Version=4.0.0.0,
        Culture=neutral,
        PublicKeyToken=b03f5f7f11d50a3a" />
  </listeners>
</trace>
</system.diagnostics>
```

`WebPageTraceListener` 允许通过浏览至 `/trace.axd` 查看跟踪输出。
1. 将<a href="https://msdn.microsoft.com/library/vstudio/6915t83k(v=vs.100).aspx">跟踪元素</a>添加到 Web.config file 文件中的 `<system.web>` 下面，如下所示：

``` xml
<trace enabled="true" writeToDiagnosticsTrace="true" mostRecent="true" pageOutput="false" />
```       

1. 按 Ctrl+F5 运行应用程序。
1. 在浏览器窗口的地址栏中，添加*trace.axd*到 URL，然后按 Enter (URL 类似于`http://localhost:53370/trace.axd`)。
1. 在“应用程序跟踪”页面上，单击第一行中（非浏览器链接中）的“查看详细信息”。

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd1.png)

    “请求细节”页面显示，在“跟踪信息”部分会看到添加到 `Index` 方法的跟踪语句的输出。

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd2.png)

    默认情况下，`trace.axd` 仅供本地使用。 如果希望可以从远程应用使用，可将 `localOnly="false"` 添加到 *Web.config* 文件中的 `trace` 元素，如以下示例所示：

        <trace enabled="true" writeToDiagnosticsTrace="true" localOnly="false" mostRecent="true" pageOutput="false" />

    但是，出于安全考虑，不推荐在生产应用中启用 `trace.axd`。 以下部分将介绍一种在应用服务应用中读取跟踪日志的更简单的方法。

### <a name="view-the-tracing-output-in-azure"></a>在 Azure 中查看跟踪输出
1. 在“解决方案资源管理器”中，右键单击该 Web 项目，并单击“发布”。
2. 在“发布 Web”对话框中，单击“发布”。

    在 Visual Studio 发布更新后，将打开一个主页的浏览器窗口（假设没有清除“连接”选项卡上的“目标 URL”）。
3. 在“服务器资源管理器”中，右键单击应用，并选择“查看流式处理日志”。

    ![上下文菜单中的查看流式处理日志](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewlogsmenu.png)

    “输出”窗口显示已连接至日志流式传输服务，每过一分钟没有要显示的日志，就添加一个通知行。

    ![上下文菜单中的查看流式处理日志](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-nologsyet.png)
4. 在显示应用程序主页的浏览器窗口，单击“联系人”。

    几秒钟内，添加到 `Contact` 方法的错误级跟踪的输出显示在“输出”窗口中。

    ![输出窗口中的错误跟踪](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-errortrace.png)

    Visual Studio 仅显示错误级跟踪，因为这是启用日志监视服务时的默认设置。 创建新的应用服务应用时，默认禁用所有日志记录，正如之前打开设置页面时所见：

    ![应用程序日志记录关闭](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-apploggingoff.png)

    然而，如果选择“查看流式处理日志”，Visual Studio 会自动将“应用程序日志记录(文件系统)”更改为“错误”，这意味着将报告错误级日志。 为了查看所有跟踪日志，可将该设置更改为“详细”。 如果选择的严重级别低于错误，也将报告所有更高严重级别的日志。 因此，如果选择“详细”，还可查看信息、警告以及错误日志。  

5. 在“服务器资源管理器”中，右键单击应用，并与之前一样单击“查看设置”。
6. 将“应用程序日志记录(文件系统)”更改为“详细”，然后单击“保存”。

    ![将跟踪级别设置为详细](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-applogverbose.png)
7. 现在，在显示“联系人”页面的浏览器窗口中，单击“主页”，并依次单击“关于”和“联系人”。

    几秒钟内，“输出”窗口会显示所有跟踪输出。

    ![详细的跟踪输出](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-verbosetraces.png)

    在本部分中，已通过使用应用设置启用和禁用日志记录。 此外，还可以通过修改 Web.config 文件启用和禁用跟踪侦听器。 然而，修改 Web.config 文件会导致应用域回收，而通过应用配置启用日志记录则不会出此情况。 如果问题重现需要花费较长时间或呈间歇性，回收应用域可能会“修复”该问题并强迫你一直等到其再次出现。 通过启用 Azure 中的诊断，可以立即开始捕获错误信息，并且不需要回收应用域。

### <a name="output-window-features"></a>输出窗口特性
“输出”窗口的“Microsoft Azure 日志”选项卡上有若干按钮和一个文本框：

![日志选项卡按钮](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-icons.png)

这些控件执行以下功能：

* 清理“输出”窗口。
* 启用或禁用自动换行。
* 启动或停止监视日志。
* 指定要监视的日志。
* 下载日志。
* 基于搜索字符串或正则表达式筛选日志。
* 关闭“输出”窗口。

如果输入搜索字符串或正则表达式，Visual Studio 会在客户端筛选日志记录信息。 这意味着可以在日志显示在“输出”窗口后输入条件，并可更改筛选条件而不必重新生成日志。

## <a name="webserverlogs"></a>查看 Web 服务器日志
Web 服务器日志将记录应用上所有的 HTTP 活动。 若要在“输出”窗口中查看这些日志，必须为应用启用日志并告知 Visual Studio 希望对其进行监视。

1. 在通过“服务器资源管理器”打开的“Azure Web 应用配置”选项卡上，将“Web 服务器日志记录”的状态更改为“开启”，然后单击“保存”。

    ![启用 Web 服务器日志记录](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-webserverloggingon.png)
2. 在“输出”窗口中，单击“指定要监视的 Microsoft Azure 日志”按钮。

    ![指定要监视的 Azure 日志。](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-specifylogs.png)
3. 在“Microsoft Azure 日志记录选项”对话框中，选择“Web 服务器日志”，然后单击“确定”。

    ![监视 Web 服务器日志](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorwslogson.png)
4. 在显示应用的浏览器窗口中，依次单击“主页”、“关于”、“联系人”。

    应用程序日志通常首先显示，其后是 Web 服务器日志。 可能需要等待一段时间才会显示日志。

    ![输出窗口中的 Web 服务器日志](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-wslogs.png)

默认情况下，通过使用 Visual Studio 第一次启用 Web 服务器日志时，Azure 会将日志写入文件系统。 或者，可以使用 Azure 门户指定将 Web 服务器日志写入存储帐户的 Blob 容器。

如果使用门户将 Web 服务器日志记录写入 Azure 存储帐户，之后在 Visual Studio 中禁用日志记录，则在 Visual Studio 中重新启用日志记录时，存储帐户设置将还原。

## <a name="detailederrorlogs"></a>查看详细的错误消息日志
详细的错误日志提供了有关导致错误响应代码（400 或更大）的 HTTP 请求的一些额外信息。 若要在“输出”窗口中查看这些日志，必须为应用启用日志并告知 Visual Studio 希望对其进行监视。

1. 在通过“服务器资源管理器”打开的“Azure Web 应用配置”选项卡上，将“详细的错误消息”的状态更改为“开启”，然后单击“保存”。

    ![启用详细的错误消息](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailedlogson.png)

2. 在“输出”窗口中，单击“指定要监视的 Microsoft Azure 日志”按钮。

3. 在“Microsoft Azure 日志记录选项”对话框中，单击“所有日志”，然后单击“确定”。

    ![监视所有日志](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorall.png)

4. 在浏览器的地址栏中，向 URL 添加一个额外字符以导致 404 错误（例如 `http://localhost:53370/Home/Contactx`），并按 Enter。

    几秒后，详细的错误日志显示在 Visual Studio 的“输出”窗口中。

    ![详细的错误日志 - 输出窗口](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorlog.png)

    Control + 单击该链接可在浏览器中看到排好格式的日志输出。

    ![详细的错误日志 - 浏览器窗口](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorloginbrowser.png)

## <a name="downloadlogs"></a>下载文件系统日志
任何可在“输出”窗口中监视的日志都可作为 *.zip* 文件进行下载。

1. 在“输出”窗口中单击“下载流式处理日志”。

    ![日志选项卡按钮](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadicon.png)

    “文件资源管理器”打开，其中显示 *Downloads* 文件夹中已下载文件处于选定状态。

    ![下载的文件](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadedfile.png)
2. 提取该 *.zip* 文件，会看到以下文件夹结构：

    ![下载的文件](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilefolders.png)

   * 应用程序跟踪日志位于 *LogFiles\Application* 文件夹的 *.txt* 文件中。
   * Web 服务器日志位于 *LogFiles\http\RawLogs* 文件夹的 *.log* 文件中。 可以使用诸如 [Log Parser](https://www.microsoft.com/download/details.aspx?displaylang=en&id=24659) 之类的工具查看并处理这些文件。
   * 详细的错误消息日志位于 *LogFiles\DetailedErrors* 文件夹的 *.html* 文件中。

     （*deployments* 文件夹内是由源代码管理发布创建的文件；其中没有任何有关 Visual Studio 发布的内容。 *Git* 文件夹内是与源代码管理发布以及日志文件流式传输服务相关的跟踪。）  

<!-- ## <a name="storagelogs"></a>View storage logs
Application tracing logs can also be sent to an Azure storage account, and you can view them in Visual Studio. To do that you'll create a storage account, enable storage logs in the Azure portal, and view them in the **Logs** tab of the **Azure Web App** window.

You can send logs to any or all of three destinations:

* The file system.
* Storage account tables.
* Storage account blobs.

You can specify a different severity level for each destination.

Tables make it easy to view details of logs online, and they support streaming; you can query logs in tables and see new logs as they are being created. Blobs make it easy to download logs in files and to analyze them using HDInsight, because HDInsight knows how to work with blob storage. For more information, see **Hadoop and MapReduce** in [Data Storage Options (Building Real-World Cloud Apps with Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options).

You currently have file system logs set to verbose level; the following steps walk you through setting up information level logs to go to storage account tables. Information level means all logs created by calling `Trace.TraceInformation`, `Trace.TraceWarning`, and `Trace.TraceError` will be displayed, but not logs created by calling `Trace.WriteLine`.

Storage accounts offer more storage and longer-lasting retention for logs compared to the file system. Another advantage of sending application tracing logs to storage is that you get some additional information with each log that you don't get from file system logs.

1. Right-click **Storage** under the Azure node, and then click **Create Storage Account**.

![Create Storage Account](./media/web-sites-dotnet-troubleshoot-visual-studio/createstor.png)

1. In the **Create Storage Account** dialog, enter a name for the storage account.

    The name must be must be unique (no other Azure storage account can have the same name). If the name you enter is already in use you'll get a chance to change it.

    The URL to access your storage account will be *{name}*.core.windows.net.
2. Set the **Region or Affinity Group** drop-down list to the region closest to you.

    This setting specifies which Azure datacenter will host your storage account. For this tutorial your choice won't make a noticeable difference, but for a production web app you want your web server and your storage account to be in the same region to minimize latency and data egress charges. The web app (which you'll create later) should run in a region as close as possible to the browsers accessing your web app in order to minimize latency.
3. Set the **Replication** drop-down list to **Locally redundant**.
   
    When geo-replication is enabled for a storage account, the stored content is replicated to a secondary datacenter to enable failover to that location in case of a major disaster in the primary location. Geo-replication can incur additional costs. For test and development accounts, you generally don't want to pay for geo-replication. For more information, see [Create, manage, or delete a storage account](../storage/common/storage-create-storage-account.md).
4. Click **Create**.

    ![New storage account](./media/web-sites-dotnet-troubleshoot-visual-studio/newstorage.png)    
5. In the Visual Studio **Azure Web App** window, click the **Logs** tab, and then click **Configure Logging in Management Portal**.

     <!-- todo:screenshot of new portal if the VS page link goes to new portal -- >
    ![Configure logging](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configlogging.png)

    This opens the **Configure** tab in the portal for your web app.
6. In the portal's **Configure** tab, scroll down to the application diagnostics section, and then change **Application Logging (Table Storage)** to **On**.
7. Change **Logging Level** to **Information**.
8. Click **Manage Table Storage**.

    ![Click Manage TableStorage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-stgsettingsmgmtportal.png)

    In the **Manage table storage for application diagnostics** box, you can choose your storage account if you have more than one. You can create a new table or use an existing one.

    ![Manage table storage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-choosestorageacct.png)
9. In the **Manage table storage for application diagnostics** box, click the check mark to close the box.
10. In the portal's **Configure** tab, click **Save**.
11. In the browser window that displays the application web app, click **Home**, then click **About**, and then click **Contact**.

     The logging information produced by browsing these web pages is written to the storage account.
12. In the **Logs** tab of the **Azure Web App** window in Visual Studio, click **Refresh** under **Diagnostic Summary**.

     ![Click Refresh](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-refreshstorage.png)

     The **Diagnostic Summary** section shows logs for the last 15 minutes by default. You can change the period to see more logs.

     (If you get a "table not found" error, verify that you browsed to the pages that do the tracing after you enabled **Application Logging (Storage)** and after you clicked **Save**.)

     ![Storage logs](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storagelogs.png)

     Notice that in this view you see **Process ID** and **Thread ID** for each log, which you don't get in the file system logs. You can see additional fields by viewing the Azure storage table directly.
13. Click **View all application logs**.

     The trace log table appears in the Azure storage table viewer.

     (If you get a "sequence contains no elements" error, open **Server Explorer**, expand the node for your storage account under the **Azure** node, and then right-click **Tables** and click **Refresh**.)

     ![Storage logs in table view](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracelogtableview.png)

     This view shows additional fields you don't see in any other views. This view also enables you to filter logs by using special Query Builder UI for constructing a query. For more information, see Working with Table Resources - Filtering Entities in [Browsing Storage Resources with Server Explorer](https://msdn.microsoft.com/library/ff683677.aspx).
14. To look at the details for a single row, double-click one of the rows.

     ![Trace table in Server Explorer](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetablerow.png)
 -->
## <a name="failedrequestlogs"></a>查看失败请求跟踪日志
如果需要详细了解 IIS 如何处理 HTTP 请求（在出现 URL 重写或身份验证问题等情况下），失败请求跟踪日志十分有用。

应用服务应用使用 IIS 7.0 及更高版本中提供的相同失败请求跟踪功能。 IIS 设置经过配置可记录指定错误，但无法访问该设置。 启用失败请求跟踪后，所有错误都将纳入捕获范围内。

使用 Visual Studio 可启用失败请求跟踪，但却无法在 Visual Studio 中对其进行查看。 这些日志是 XML 文件。 这些流式传输日志服务只监视认为在纯文本模式下可读的文件：*.txt*、*.html* 和 *.log* 文件。

可以通过 FTP 直接在浏览器中查看失败请求跟踪日志，或使用 FTP 工具将其下载到本地计算机后进行本地查看。 在本节中，将直接在浏览器中查看这些日志。

1. 在从“服务器资源管理器”打开的“Azure Web 应用”窗口的“配置”选项卡中，将“失败请求跟踪”的状态更改为“开启”，然后单击“保存”。

    ![启用失败请求跟踪](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequeston.png)
2. 在显示该应用的浏览器窗口的地址栏中，向 URL 添加一个额外字符，单击 Enter 将引发 404 错误。

    这会导致创建失败请求跟踪日志，以下步骤介绍如何查看或下载该日志。

3. 在 Visual Studio 中，在“Azure Web 应用”窗口的“配置”选项卡中，单击“在管理门户中打开”。

4. 在中[Azure 门户](https://portal.azure.com)**设置**为您的应用程序页上，单击**部署凭据**，然后输入新用户名和密码。

    ![新建 FTP 用户名和密码](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-enterftpcredentials.png)

    > [!NOTE]
    > 登录时，必须使用以应用名称为前缀的完整用户名。 例如，如果输入“myid”作为用户名，而站点为“myexample”，请使用“myexample\myid”登录。
    >

5. 在新的浏览器窗口中，转到显示在应用的“概述”页中“FTP 主机名”或“FTPS 主机名”下面的 URL。

6. 使用之前创建的 FTP 凭据（包括作为用户名前缀的应用名称）登录。

    浏览器将显示该应用的根文件夹。

7. 打开 *LogFiles* 文件夹。

    ![打开 LogFiles 文件夹](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilesfolder.png)

8. 打开名为 W3SVC 加数值的文件夹。

    ![打开 W3SVC 文件夹](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfolder.png)

    该文件夹包含启用失败请求跟踪之后记录在案的任何错误的 XML 文件，以及一个可供浏览器格式化 XML 的 XSL 文件。

    ![W3SVC 文件夹](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfoldercontents.png)

9. 单击希望查看跟踪信息的失败请求的 XML 文件。

    下图显示了示例错误的部分跟踪信息。

    ![浏览器中的失败请求跟踪](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequestinbrowser.png)

## <a name="nextsteps"></a>后续步骤
以上部分介绍了如何通过 Visual Studio 轻松查看由应用服务应用创建的日志。 以下部分提供了相关主题中其他资源的链接：

* 应用服务故障排除
* 在 Visual Studio 中进行调试
* 在 Azure 中进行远程调试
* 在 ASP.NET 应用程序中进行跟踪
* 分析 Web 服务器日志
* 分析失败请求跟踪日志
* 调试云服务

### <a name="app-service-troubleshooting"></a>应用服务故障排除
有关对 Azure 应用服务中的应用进行故障排除的详细信息，请参阅以下资源：

* [如何监视应用](web-sites-monitor.md)
* [使用 Visual Studio 2013 在 Azure 应用服务中调查内存泄漏](https://blogs.msdn.com/b/visualstudioalm/archive/2013/12/20/investigating-memory-leaks-in-azure-web-sites-with-visual-studio-2013.aspx)。 有关用于分析托管内存问题的 Visual Studio 功能的 Microsoft ALM 博客文章。
* [应该了解的 Azure 应用服务联机工具](https://azure.microsoft.com/blog/2014/03/28/windows-azure-websites-online-tools-you-should-know-about-2/)。 Amit Apple 发表的博客文章。

若要针对特定故障排除问题寻求帮助，可在以下论坛之一开启话题讨论：

* [ASP.NET 站点上的 Azure 论坛](https://forums.asp.net/1247.aspx/1?Azure+and+ASP+NET)。
* [MSDN 上的 Azure 论坛](https://social.msdn.microsoft.com/Forums/windowsazure/)。
* [StackOverflow.com](https://www.stackoverflow.com)。

### <a name="debugging-in-visual-studio"></a>在 Visual Studio 中进行调试
有关如何在 Visual Studio 中使用调试模式的详细信息，请参阅[在 Visual Studio 中进行调试](/visualstudio/debugger/debugging-in-visual-studio)和[使用 Visual Studio 2010 进行调试的提示](https://weblogs.asp.net/scottgu/archive/2010/08/18/debugging-tips-with-visual-studio-2010.aspx)。

### <a name="remote-debugging-in-azure"></a>在 Azure 中进行远程调试
有关应用服务应用和 WebJobs 远程调试的详细信息，请参阅以下资源：

* [Azure 应用服务远程调试简介](https://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/)。
* [Azure 应用服务远程调试简介第 2 部分 - 远程调试洞析](https://azure.microsoft.com/blog/2014/05/07/introduction-to-remote-debugging-azure-web-sites-part-2-inside-remote-debugging/)
* [Azure 应用服务远程调试简介第 3 部分 - 多实例环境和 GIT](https://azure.microsoft.com/blog/2014/05/08/introduction-to-remote-debugging-on-azure-web-sites-part-3-multi-instance-environment-and-git/)
* [WebJobs 调试（视频）](https://www.youtube.com/watch?v=ncQm9q5ZFZs&list=UU_SjTh-ZltPmTYzAybypB-g&index=1)

如果应用使用 Azure Web API 或移动服务后端且需要进行调试，请参阅[在 Visual Studio 中调试 .NET 后端](https://blogs.msdn.com/b/azuremobile/archive/2014/03/14/debugging-net-backend-in-visual-studio.aspx)。

### <a name="tracing-in-aspnet-applications"></a>在 ASP.NET 应用程序中进行跟踪
Internet 上对于 ASP.NET 跟踪没有全面且最新的介绍。 最佳做法是以针对 Web 窗体编写的老旧介绍性材料为主（因为 MVC 彼时并不存在），以专注于特定问题的新兴博客文章为辅。 以下资源可作为比较好的入门资源：

* [监视和遥测（使用 Azure 生成实际的云应用）](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry)。<br>
  电子书籍的章节就在 Azure 云应用程序中进行跟踪给出了建议。
* [ASP.NET 跟踪](/previous-versions/dotnet/articles/ms972204(v=msdn.10))<br/>
  内容过时，但不失为这一主题的优秀入门级资源。
* [跟踪侦听器](/dotnet/framework/debug-trace-profile/trace-listeners)<br/>
  有关跟踪侦听器的信息，但未提及 [WebPageTraceListener](/dotnet/api/system.web.webpagetracelistener)。
* [演练：将 ASP.NET 跟踪与 System.Diagnostics 跟踪集成](/previous-versions/b0ectfxd(v=vs.140))<br/>
  此文章已过时，但包括一些介绍性文章未涵盖的额外信息。
* [在 ASP.NET MVC Razor 视图中进行跟踪](https://blogs.msdn.com/b/webdev/archive/2013/07/16/tracing-in-asp-net-mvc-razor-views.aspx)<br/>
  除了在 Razor 视图中进行跟踪，文章还介绍了如何创建错误筛选器以便在 MVC 应用程序中记录所有未经处理的异常。 有关如何在 Web 窗体应用程序中记录所有未经处理的异常，请参阅 MSDN 上的[错误处理程序的完整示例](/previous-versions/bb397417(v=vs.140))中的 Global.asax 示例。 在 MVC 或 Web 窗体中，如果希望记录特定异常但想让默认框架对其进行处理，可捕获并重新引发异常，如下例所示：

``` c#
try
{
   // Your code that might cause an exception to be thrown.
}
catch (Exception ex)
{
    Trace.TraceError("Exception: " + ex.ToString());
    throw;
}
```

* [从 Azure 命令行流式传输诊断跟踪日志记录（加上 Glimpse！）](https://www.hanselman.com/blog/StreamingDiagnosticsTraceLoggingFromTheAzureCommandLinePlusGlimpse.aspx)<br/>
  如何使用命令行实现本教程中通过 Visual Studio 完成的任务。 [Glimpse](https://www.hanselman.com/blog/IfYoureNotUsingGlimpseWithASPNETForDebuggingAndProfilingYoureMissingOut.aspx) 是一个用于调试 ASP.NET 应用程序的工具。
* [使用 Web 应用日志记录和诊断 - 与 David Ebbo 协作完成](https://azure.microsoft.com/documentation/videos/azure-web-site-logging-and-diagnostics/)和[从 Azure Web 应用流式传输日志 - 与 David Ebbo 协作完成](https://azure.microsoft.com/documentation/videos/log-streaming-with-azure-web-sites/)<br>
  由 Scott Hanselman 和 David Ebbo 提供的视频。

对于错误日志记录，若不想编写自己的跟踪代码，可以使用开源日志记录框架，如 [ELMAH](https://nuget.org/packages/elmah/)。 有关详细信息，请参阅 [Scott Hanselman 有关 ELMAH 的博客文章](https://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx)。

此外，不需要使用 ASP.NET 或 `System.Diagnostics` 跟踪来从 Azure 获取流式传输日志。 应用服务应用流式处理日志服务会对其在 *LogFiles* 文件夹中找到的所有 *.txt*、*.html* 或 *.log* 文件进行流式处理。 因此，你可以创建自己的日志记录系统以写入应用的文件系统，文件会自动进行流式处理和下载。 所要做的就是编写在 *d:\home\logfiles* 文件夹中创建文件的应用程序代码。

### <a name="analyzing-web-server-logs"></a>分析 Web 服务器日志
有关分析 Web 服务器日志的详细信息，请参阅以下资源：

* [LogParser](https://www.microsoft.com/download/details.aspx?id=24659)<br/>
  用于查看 Web 服务器日志（*.log* 文件）中的数据的工具。
* [解决 IIS 性能问题或使用 logparser 解决应用程序错误](https://www.iis.net/learn/troubleshoot/performance-issues/troubleshooting-iis-performance-issues-or-application-errors-using-logparser)<br/>
  可用于分析 Web 服务器日志的 Log Parser 工具介绍。
* [Robert McMurray 有关 LogParser 使用的博客文章](https://blogs.msdn.com/b/robert_mcmurray/archive/tags/logparser/)<br/>
* [IIS 7.0、IIS 7.5 以及 IIS 8.0 中的 HTTP 状态代码](https://support.microsoft.com/kb/943891)

### <a name="analyzing-failed-request-tracing-logs"></a>分析失败请求跟踪日志
Microsoft TechNet 网站包含[使用失败请求跟踪](https://www.iis.net/learn/troubleshoot/using-failed-request-tracing)部分，有助于你了解如何使用这些日志。 然而，该文档主要着重于在 IIS 中配置失败请求跟踪，并不适用于 Azure 应用服务。

[GetStarted]: app-service-web-get-started-dotnet.md
[GetStartedWJ]: https://github.com/Azure/azure-webjobs-sdk/wiki
