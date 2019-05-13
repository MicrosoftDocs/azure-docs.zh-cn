---
title: 排查无数据问题 - 用于 .NET 的 Application Insights
description: 在 Azure Application Insights 中看不到数据？ 试试这里。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: e231569f-1b38-48f8-a744-6329f41d91d3
ms.service: application-insights
ms.workload: mobile
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/23/2018
ms.author: mbullwin
ms.openlocfilehash: 467586fd23332469338dabd2feb6a42ce4b17af5
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471850"
---
# <a name="troubleshooting-no-data---application-insights-for-net"></a>排查无数据问题 - 用于 .NET 的 Application Insights
## <a name="some-of-my-telemetry-is-missing"></a>缺少一些遥测数据
*在 Application Insights 中，我只看到应用生成的一部分事件。*

* 如果持续看到同一个部分，可能是由于自适应[采样](../../azure-monitor/app/sampling.md)所导致。 要确认这一点，请打开“搜索”（通过“概述”边栏选项卡），并查看请求或其他事件的实例。 单击“属性”部分底部的“...”，获取完整的属性详细信息。 如果请求计数 > 1，则表示采样正在进行。
* 否则，有可能是已达到定价计划的[数据率限制](../../azure-monitor/app/pricing.md#limits-summary)。 系统每隔一分钟应用这些限制。

## <a name="no-data-from-my-server"></a>服务器未提供数据
*我已在 Web 服务器上安装应用，但未看到服务器提供任何遥测数据。服务器在开发计算机上正常运行。*

* 可能是防火墙有问题。 [为 Application Insights 设置防火墙例外即可发送数据](../../azure-monitor/app/ip-addresses.md)。
* IIS 服务器可能缺少某些必备组件：.NET Extensibility 4.5 和 ASP.NET 4.5。

*我已在 Web 服务器上[安装状态监视器](../../azure-monitor/app/monitor-performance-live-website-now.md)来监视现有应用，但未看到任何结果。*

* 请参阅[排查状态监视器问题](../../azure-monitor/app/monitor-performance-live-website-now.md#troubleshoot)。

## <a name="q01"></a>Visual Studio 中没有“添加 Application Insights”选项
*在解决方案资源管理器中右键单击现有项目时，未看到任何 Application Insights 选项。*

* 工具并非支持所有类型的 .NET 项目。 支持 Web 和 WCF 项目。 对于其他项目类型，例如桌面或服务应用程序，仍可以[手动将 Application Insights SDK 添加到项目](../../azure-monitor/app/windows-desktop.md)。
* 请务必使用 [Visual Studio 2013 Update 3 或更高版本](https://docs.microsoft.com/visualstudio/releasenotes/vs2013-update3-rtm-vs)。 该软件预装了开发人员分析工具，其中提供了 Application Insights SDK。
* 选择“工具”、“扩展和更新”，检查“开发人员分析工具”是否已安装并启用。 如果是，请单击“更新”查看是否有可用的更新。
* 打开“新建项目”对话框，选择“ASP.NET Web 应用程序”。 如果看到了 Application Insights 选项，则表示工具已安装。 如果没有，请尝试卸载然后重新安装 Developer Analytics Tools。

## <a name="q02"></a>添加 Application Insights 失败
*尝试将 Application Insights 添加到现有项目时看到错误消息。*

可能的原因：

* 与 Application Insights 门户通信失败；或者
* Azure 帐户存在某种问题；
* 对于在其中创建新资源的订阅或组，只拥有[读取访问权限](../../azure-monitor/app/resources-roles-access-control.md)。

解决方法：

* 检查是否为适当的 Azure 帐户提供了登录凭据。
* 在浏览器中，检查是否可以访问 [Azure 门户](https://portal.azure.com)。 打开“设置”并查看是否有任何限制。
* [将 Application Insights 添加到现有项目](../../azure-monitor/app/asp-net.md)：在解决方案资源管理器中，右键单击项目并选择“添加 Application Insights”。
* 如果仍不起作用，请执行[手动过程](../../azure-monitor/app/windows-services.md)在门户中添加资源，然后将 SDK 添加到项目。

## <a name="emptykey"></a>遇到错误“检测密钥不能为空”
可能是在安装 Application Insights 或日志记录适配器时发生了问题。

在解决方案资源管理器中右键单击项目，并选择“Application Insights”>“配置 Application Insights”。 将得到一个对话框，邀请你登录 Azure 并创建 Application Insights 资源或重复使用现有资源。

## <a name="NuGetBuild"></a> 生成服务器上出现“缺少 NuGet 包”
*在开发计算机上调试时所有生成都没有问题，但生成服务器上出现 NuGet 错误。*

请参阅 [NuGet Package Restore](https://docs.nuget.org/Consume/Package-Restore)（NuGet 包还原）和 [Automatic Package Restore](https://docs.nuget.org/Consume/package-restore/migrating-to-automatic-package-restore)（自动包还原）。

## <a name="missing-menu-command-to-open-application-insights-from-visual-studio"></a>缺少用于通过 Visual Studio 打开 Application Insights 的菜单命令
*在解决方案资源管理器中右键单击我的项目时，未看到任何 Application Insights 命令，或者未看到“打开 Application Insights”命令。*

可能的原因：

* Application Insights 资源是手动创建的，或者项目是 Application Insights 工具不支持的类型。
* 开发人员分析工具已在 Visual Studio 中禁用。
* Visual Studio 版本低于 2013 Update 3。

解决方法：

* 确保 Visual Studio 版本为 2013 Update 3 或更高。
* 选择“工具”、“扩展和更新”，检查“开发人员分析工具”是否已安装并启用。 如果是，请单击“更新”查看是否有可用的更新。
* 在解决方案资源管理器中右键单击项目。 如果看到命令“Application Insights”>“配置 Application Insights”，请使用它将项目连接到 Application Insights 服务中的资源。

否则，开发人员分析工具不直接支持项目类型。 要查看遥测数据，请登录到 [Azure 门户](https://portal.azure.com)，在左侧导航栏中选择“Application Insights”，然后选择应用程序。

## <a name="access-denied-on-opening-application-insights-from-visual-studio"></a>从 Visual Studio 打开 Application Insights 时出现“拒绝访问”
*使用“打开 Application Insights”菜单命令时转到了 Azure 门户，但同时出现“拒绝访问”错误。*

上一次在默认浏览器中使用的 Microsoft 登录名无权访问[将 Application Insights 添加到此应用时所创建的资源](../../azure-monitor/app/asp-net.md)。 有两个可能的原因：

* 有多个的 Microsoft 帐户 - 也许同时拥有工作帐户和个人 Microsoft 帐户。 上一次在默认浏览器中使用的登录名的关联帐户不是有权[将 Application Insights 添加到项目](../../azure-monitor/app/asp-net.md)的帐户。
  * 解决方法：单击浏览器窗口右上角的用户名并注销。使用拥有访问权限的帐户登录。 然后，在左侧导航栏中单击“Application Insights”并选择应用。
* 其他人将 Application Insights 添加到了项目，但他们忘记向你提供创建项目的[资源组的访问权限](../../azure-monitor/app/resources-roles-access-control.md)。
  * 解决方法：如果他们使用的是组织帐户，便可以将你添加到团队；或者，他们可以单独向你授予对资源组的访问权限。

## <a name="asset-not-found-on-opening-application-insights-from-visual-studio"></a>从 Visual Studio 打开 Application Insights 时出现“找不到资产”
*使用“打开 Application Insights”菜单命令时转到了 Azure 门户，但同时出现“找不到资产”错误。*

可能的原因：

* 应用程序的 Application Insights 资源已删除；或
* 检测密钥是通过直接编辑 ApplicationInsights.config 设置或更改的，但未更新项目文件。

ApplicationInsights.config 中的检测密钥控制遥测数据发送到的位置。 项目文件中的某个行控制在 Visual Studio 中使用命令时打开哪个资源。

解决方法：

* 在解决方案资源管理器中右键单击项目，并依次选择“Application Insights”、“配置 Application Insights”。 在对话框中，可以选择将遥测数据发送到现有资源，或创建新资源。 或：
* 直接打开资源。 登录到 [Azure 门户](https://portal.azure.com)，在左侧导航栏中单击“Application Insights”，然后选择应用。

## <a name="where-do-i-find-my-telemetry"></a>在哪里可以找到我的遥测数据？
*我已登录到 [Microsoft Azure 门户](https://portal.azure.com)，并且看到了 Azure 主页仪表板。但是，可以在哪里找到我的 Application Insights 数据？*

* 在左侧导航栏中单击“Application Insights”，并选择应用名称。 如果未看到任何项目，则需要[在 Web 项目中添加或配置 Application Insights](../../azure-monitor/app/asp-net.md)。  
  然后，会看到一些摘要图表。 可以逐个单击这些图表查看详细信息。
* 在 Visual Studio 中调试应用时，请单击“Application Insights”按钮。

## <a name="q03"></a>没有服务器数据（或根本没有数据）
*我运行了应用，并在 Microsoft Azure 中打开 Application Insights 服务，但所有图表都显示“了解如何收集...”或“未配置”。* 或者，*只有页面视图和用户数据，但没有任何服务器数据。*

* 在 Visual Studio 中以调试模式运行应用程序 (F5)。 使用应用程序，以便生成一些遥测。 检查是否可以在 Visual Studio 的“输出”窗口中看到记录的事件。  
  ![](./media/asp-net-troubleshoot-no-data/output-window.png)
* 在 Application Insights 门户中，打开[诊断搜索](../../azure-monitor/app/diagnostic-search.md)。 数据通常会先显示在此处。
* 单击“刷新”按钮。 边栏选项卡会定期自行刷新，但你也可以手动刷新。 时间范围越大，刷新间隔就越长。
* 检查检测密钥是否匹配。 在 Application Insights 门户的应用主边栏选项卡中，查看“概要”下拉列表中的“检测密钥”。 然后，在 Visual Studio 的项目中，打开 ApplicationInsights.config 并找到 `<instrumentationkey>`。 检查两个密钥是否相同。 如果不同：  
  * 在门户中单击“Application Insights”，并找到具有正确密钥的应用资源；或
  * 在 Visual Studio 解决方案资源管理器中右键单击项目，并依次选择“Application Insights”、“配置”。 重置应用，将遥测数据发送到正确的资源。
  * 如果找不到匹配的密钥，请检查在 Visual Studio 中使用的登录凭据是否与门户中使用的相同。
* 在 [Microsoft Azure 主页仪表板](https://portal.azure.com)中，查看服务运行状况地图。 如果看到警报指示，请等待它们恢复“正常”，关闭再重新打开 Application Insights 应用程序边栏选项卡。
* 另请查看[我们的状态博客](https://blogs.msdn.microsoft.com/servicemap-status/)。
* 是否针对[服务器端 SDK](../../azure-monitor/app/api-custom-events-metrics.md) 编写了可能更改 `TelemetryClient` 实例或 `TelemetryContext` 中的检测密钥的任何代码？ 或者，是否编写了可能筛选掉过多内容的[筛选或采样配置](../../azure-monitor/app/api-filtering-sampling.md)？
* 如果编辑了 ApplicationInsights.config，请仔细检查 [TelemetryInitializers 和 TelemetryProcessors](../../azure-monitor/app/api-filtering-sampling.md) 的配置。 命名不当的类型或参数可能导致 SDK 不发送任何数据。

## <a name="q04"></a>未提供有关页面视图、浏览器和使用情况的任何数据
*在“服务器响应时间”和“服务器请求”图表中看到了数据，但“页面视图加载时间”或者“浏览器”或“使用情况”边栏选项卡中没有任何数据。*

这些数据来自网页中的脚本。 

* 如果将 Application Insights 添加到现有 Web 项目，则[必须手动添加脚本](../../azure-monitor/app/javascript.md)。
* 确保 Internet Explorer 不是以兼容模式显示站点。
* 使用浏览器的调试功能（在某些浏览器中请按 F12，并选择“网络”）验证数据是否发送到了 `dc.services.visualstudio.com`。

## <a name="no-dependency-or-exception-data"></a>没有依赖项或异常数据
请参阅[依赖项遥测](../../azure-monitor/app/asp-net-dependencies.md)和[异常遥测](asp-net-exceptions.md)。

## <a name="no-performance-data"></a>没有性能数据
性能数据（CPU、IO 速率等）适用于 [Java Web 服务](../../azure-monitor/app/java-collectd.md)、[Windows 桌面应用](../../azure-monitor/app/windows-desktop.md)、[IIS Web 应用和服务（如果安装了状态监视器）](../../azure-monitor/app/monitor-performance-live-website-now.md)以及 [Azure 云服务](../../azure-monitor/app/app-insights-overview.md)。 可在“设置”、“服务器”下面看到这些数据。

## <a name="no-server-data-since-i-published-the-app-to-my-server"></a>将应用发布到服务器后未看到（服务器）数据
* 请检查是否确实将 Microsoft. ApplicationInsights DLL 连同 Microsoft.Diagnostics.Instrumentation.Extensions.Intercept.dll 一起复制到了服务器。
* 在防火墙中，可能需要[打开某些 TCP 端口](../../azure-monitor/app/ip-addresses.md)。
* 如果必须使用代理在企业网络外部发送数据，请在 Web.config 中设置 [defaultProxy](https://msdn.microsoft.com/library/aa903360.aspx)
* Windows Server 2008：请确保已安装以下更新：[KB2468871](https://support.microsoft.com/kb/2468871)、[KB2533523](https://support.microsoft.com/kb/2533523)、[KB2600217](https://support.microsoft.com/kb/2600217)。

## <a name="i-used-to-see-data-but-it-has-stopped"></a>我以前看到了数据，但现在看不到
* 请查看[状态博客](https://blogs.msdn.com/b/applicationinsights-status/)。
* 是否达到了数据点的每月配额？ 打开“设置/配额和定价”即可检查。如果达到了配额，可以升级计划，或付费购买更多的容量。 请参阅[定价方案](https://azure.microsoft.com/pricing/details/application-insights/)。

## <a name="i-dont-see-all-the-data-im-expecting"></a>未按预期看到所有数据
如果应用程序发送大量数据，并且使用的是用于 ASP.NET 的 Application Insights SDK 2.0.0-beta3 或更高版本，则[自适应采样](../../azure-monitor/app/sampling.md)功能可以正常运行，只发送一部分遥测数据。

可以禁用该功能，但不建议这样做。 采样旨在正确传输相关遥测数据，以便进行诊断。

## <a name="client-ip-address-is-0000"></a>客户端 IP 地址为 0.0.0.0

在 2018 年 2 月 5 日，我们宣布我们删除了客户端 IP 地址的日志记录。 这不会影响地理位置。

> [!NOTE]
> 如果需要 IP 地址的前 3 个八位字节，则可以使用[遥测初始化程序](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling#add-properties-itelemetryinitializer)添加自定义属性。
> 这不会影响 2018 年 2 月 5 日之前收集的数据。

## <a name="wrong-geographical-data-in-user-telemetry"></a>用户遥测数据包含错误的地理数据
城市、区域和国家/地区维度派生自 IP 地址，因此不一定始终都是准确的。 首先针对位置处理这些 IP 地址，然后将其更改为 0.0.0.0 以进行存储。

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>在 Azure 云服务中运行时发生“找不到方法”异常
生成的项目是否面向 .NET 4.6？ Azure 云服务角色不能现成地支持 4.6。 请先[在每个角色上安装 4.6](../../cloud-services/cloud-services-dotnet-install-dotnet.md)，再运行应用。

## <a name="troubleshooting-logs"></a>故障排除日志

按照这些说明来捕获框架的故障排除日志。

### <a name="net-framework"></a>.NET Framework

1. 从 NuGet 安装 [Microsoft.AspNet.ApplicationInsights.HostingStartup](https://www.nuget.org/packages/Microsoft.AspNet.ApplicationInsights.HostingStartup) 包。 安装的版本必须与当前安装的 `Microsoft.ApplicationInsighs` 版本匹配

2. 修改 applicationinsights.config 文件以包括以下内容：

    ```xml
    <TelemetryModules>
      <Add Type="Microsoft.ApplicationInsights.Extensibility.HostingStartup.FileDiagnosticsTelemetryModule, Microsoft.AspNet.ApplicationInsights.HostingStartup">
        <Severity>Verbose</Severity>
        <LogFileName>mylog.txt</LogFileName>
        <LogFilePath>C:\\SDKLOGS</LogFilePath>
      </Add>
    </TelemetryModules>
    ```
    应用程序必须对配置的位置具有写入权限

3. 重新启动进程，以便 SDK 获取这些新设置

4. 完成后还原这些更改。

### <a name="net-core"></a>.NET Core

1. 安装[Microsoft.AspNetCore.ApplicationInsights.HostingStartup](https://www.nuget.org/packages/Microsoft.AspNetCore.ApplicationInsights.HostingStartup)从 NuGet 包。 安装的版本必须与当前安装的 `Microsoft.ApplicationInsights` 版本匹配

2. 修改 `Startup.cs` 类中的 `ConfigureServices` 方法：

    ```csharp
    services.AddSingleton<ITelemetryModule, FileDiagnosticsTelemetryModule>();
    services.ConfigureTelemetryModule<FileDiagnosticsTelemetryModule>( (module, options) => {
        module.LogFilePath = "C:\\SDKLOGS";
        module.LogFileName = "mylog.txt";
        module.Severity = "Verbose";
    } );
    ```
    应用程序必须对配置的位置具有写入权限

3. 重新启动进程，以便 SDK 获取这些新设置

4. 完成后还原这些更改。

## <a name="still-not-working"></a>仍然无法解决问题...
* [Application Insights 论坛](https://social.msdn.microsoft.com/Forums/vstudio/en-US/home?forum=ApplicationInsights)
