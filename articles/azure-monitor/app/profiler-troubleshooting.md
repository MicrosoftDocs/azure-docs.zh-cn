---
title: 排查 Azure Application Insights Profiler 的问题
description: 本文介绍了疑难解答步骤和信息，以帮助开发人员启用和使用 Application Insights Profiler。
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 05a2eaeb3b716988a8ae1eddcaa5a5a58cc3776a
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/22/2021
ms.locfileid: "98675690"
---
# <a name="troubleshoot-problems-enabling-or-viewing-application-insights-profiler"></a>排查启用或查看 Application Insights Profiler 时遇到的问题

> [!CAUTION]
> 在 Azure 应用服务中对 ASP.NET Core 应用运行探查器时有一个 bug。 我们有一个修补程序，但需要花费几周的时间来部署全球。 可以按照[此处](./asp-net-core.md#enable-application-insights-server-side-telemetry-visual-studio)的说明将 Application Insights SDK 添加到应用程序来解决此 bug。

## <a name="general-troubleshooting"></a><a id="troubleshooting"></a>常规故障排除

### <a name="profiles-are-uploaded-only-if-there-are-requests-to-your-application-while-profiler-is-running"></a>仅当在运行 Profiler 期间对应用程序发出了请求时，才上传配置文件

Azure 应用程序 Insights 探查器每小时收集一次数据两分钟。 当你在 "**配置 Application Insights Profiler** " 窗格中选择 "**立即配置文件**" 按钮时，它还可以收集数据。

> [!NOTE]
> 仅当分析数据可以附加到运行探查器时发生的请求时，才会上载分析数据。 

Profiler 将跟踪消息和自定义事件写入到 Application Insights 资源。 您可以使用这些事件来查看探查器的运行方式：

1. 搜索由 Profiler 发送到 Application Insights 资源的跟踪消息和自定义事件。 可以使用此搜索字符串查找相关的数据：

    ```
    stopprofiler OR startprofiler OR upload OR ServiceProfilerSample
    ```
    下图显示了两个从两个 AI 资源中进行搜索的示例： 
    
   * 在左侧，应用程序在 Profiler 运行时不接收请求。 此消息说明，由于没有发生活动，上传操作已取消。 

   * 在右侧，Profiler 已启动，并在检测到它运行期间发生的请求时发送了自定义事件。 如果 `ServiceProfilerSample` 显示自定义事件，则表示已捕获某个配置文件，并且该配置文件在 " **Application Insights 性能** " 窗格中可用。

     如果未显示任何记录，则探查器未运行。 要进行故障排除，请参阅本文后面的特定应用类型的故障排除部分。  

     ![搜索 Profiler 遥测数据][profiler-search-telemetry]

### <a name="other-things-to-check"></a>要检查的其他事项
* 确保应用在 .NET Framework 4.6 上运行。
* 如果 Web 应用是 ASP.NET Core 应用程序，则必须至少运行 ASP.NET Core 2.0。
* 如果要查看的数据的期限超过了好几周，请尝试限制时间筛选器并重试。 七天后将删除跟踪。
* 请确保代理或防火墙未阻止访问 https://gateway.azureserviceprofiler.net 。
* 免费或共享应用服务计划不支持 Profiler。 如果你正在使用这些计划之一，请尝试扩展至其中一个基本计划，并且 Profiler 应开始工作。

### <a name="double-counting-in-parallel-threads"></a><a id="double-counting"></a>并行线程的重复计算

在某些情况下，堆栈查看器中的总时间指标大于请求的持续时间。

当两个或多个并行线程与请求关联时，可能会出现这种情况。 在这种情况下，总线程时间就会超过已用时间。

一个线程可能会等待另一个线程完成。 查看器会尝试检测此情况并省略不相关的等待时间。 这样，它会倾向于显示过多信息，而不是省略关键信息。

当你在跟踪中看到并行线程时，确定正在等待的线程，以便你能够确定请求的热路径。

通常情况下，快速进入等待状态的线程只是在等待其他线程完成。 请专注于其他线程，忽略等待中线程花费的时间。

### <a name="error-report-in-the-profile-viewer"></a>配置文件查看器中的错误报告
在门户中提交支持票证。 请务必包含错误消息中的相关性 ID。

## <a name="troubleshoot-profiler-on-azure-app-service"></a>排查 Azure 应用服务中的 Profiler 问题
要使 Profiler 正常工作：
* Web 应用服务计划必须属于“基本”层或更高层。
* Web 应用必须已启用 Application Insights。
* Web 应用必须具有以下应用设置：

    |应用设置    | Value    |
    |---------------|----------|
    |APPINSIGHTS_INSTRUMENTATIONKEY         | Application Insights 资源的 iKey    |
    |APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
    |DiagnosticServices_EXTENSION_VERSION | ~3 |


* ApplicationInsightsProfiler3 webjob 必须正在运行。 若要检查 webjob：
   1. 转到 [Kudu](/archive/blogs/cdndevs/the-kudu-debug-console-azure-websites-best-kept-secret)。
   1. 在“工具”菜单中，选择“WebJobs 仪表板” 。  
      “WebJobs”窗格随即打开。 
   
      ![屏幕截图显示了 WebJobs 窗格，其中显示了作业的名称、状态和上次运行时间。][profiler-webjob]   
   
   1. 若要查看 webjob 的详细信息（包括日志），请选择“ApplicationInsightsProfiler3”链接。  
     “连续 WebJob 详细信息”窗格随即打开。

      ![屏幕截图显示了“连续 WebJob 详细信息”窗格。][profiler-webjob-log]

如果探查器不能正常工作，可以下载日志并将其发送给我们的团队以获得帮助 serviceprofilerhelp@microsoft.com 。

### <a name="check-the-diagnostic-services-site-extension-status-page"></a>检查 "诊断服务站点扩展" 状态页
如果通过门户中的 " [Application Insights" 窗格](profiler.md) 启用了探查器，则它是由 "诊断服务" 站点扩展启用的。

可以通过转到以下 url 来查看此扩展的 "状态" 页： `https://{site-name}.scm.azurewebsites.net/DiagnosticServices`

> [!NOTE]
> "状态页" 链接的域将随云的不同而异。
此域将与应用服务的 Kudu 管理站点相同。

此状态页显示探查器的安装状态和 Snapshot Collector 代理。 如果出现意外错误，则会显示该错误，并显示如何修复此错误。

你可以使用应用服务的 Kudu 管理站点获取此状态页的基 url：
1. 在 Azure 门户中，打开应用服务应用程序。
2. 选择 " **高级工具**"，或搜索 **Kudu**。
3. 选择“转到”。
4. 进入 Kudu 管理站点后，请在 URL 中 **追加以下项， `/DiagnosticServices` 然后按 enter**。
 它将如下所示： `https://<kudu-url>/DiagnosticServices`

它将显示类似于以下内容的状态页： " ![ 诊断服务状态" 页](./media/diagnostic-services-site-extension/status-page.png)
    
### <a name="manual-installation"></a>手动安装

配置 Profiler 时，将对 Web 应用的设置进行更新。 如果你的环境有此要求，则可以手动应用更新。 例如，应用程序在适用于 PowerApps 的 Web 应用环境中运行。 若要手动应用更新，请执行以下操作：

1. 在“Web 应用控制”窗格中，打开“设置” 。

1. 将“.NET Framework 版本”设置为“v4.6” 。

1. 将“Always On”设置为“打开” 。
1. 创建以下应用设置：

    |应用设置    | Value    |
    |---------------|----------|
    |APPINSIGHTS_INSTRUMENTATIONKEY         | Application Insights 资源的 iKey    |
    |APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
    |DiagnosticServices_EXTENSION_VERSION | ~3 |

### <a name="too-many-active-profiling-sessions"></a>活动分析会话太多

最多可对在同一服务计划中运行的四个 Web 应用启用探查器。 如果有四个以上，则探查器可能会引发 *ServiceProfiler。 TooManyETWSessionException*。 若要解决此情况，请将一些 web 应用移到不同的服务计划。

### <a name="deployment-error-directory-not-empty-dhomesitewwwrootapp_datajobs"></a>部署错误：目录不为空“D:\\home\\site\\wwwroot\\App_Data\\jobs”

如果在已启用 Profiler 的情况下将 Web 应用重新部署到 Web 应用资源，可能会看到如下消息：

*目录不为空“D:\\home\\site\\wwwroot\\App_Data\\jobs”*

如果从脚本或 Azure Pipelines 运行 Web 部署，则会发生此错误。 解决方法是将以下附加部署参数添加到 Web 部署任务：

```
-skip:Directory='.*\\App_Data\\jobs\\continuous\\ApplicationInsightsProfiler.*' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs\\continuous$' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs$'  -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data$'
```

这些参数将删除 Application Insights Profiler 所用的文件夹，并取消阻止重新部署进程。 它们不影响当前运行的 Profiler 实例。

### <a name="how-do-i-determine-whether-application-insights-profiler-is-running"></a>如何确定 Application Insights Profiler 是否正在运行？

Profiler 在 Web 应用中以连续 Web 作业的形式运行。 可以在 [Azure 门户](https://portal.azure.com)中打开 Web 应用资源。 在“WebJobs”窗格中，查看 ApplicationInsightsProfiler 的状态 。 如果探查器未运行，请打开“日志”获取详细信息。

## <a name="troubleshoot-vms-and-cloud-services"></a>对 VM 和云服务进行故障排除

>**云服务 WAD 中附带的探查器中的 bug 已修复。** 用于云服务的最新版本的 WAD (1.12.2.0) 适用于所有最新版本的 App Insights SDK。 云服务主机将自动升级 WAD，但不会立即升级。 若要强制升级，可以重新部署服务或重新启动节点。

若要查看 Azure 诊断是否正确配置了探查器，请执行以下步骤： 
1. 验证部署的 Azure 诊断配置的内容是否符合你的预期。 

1. 其次，确保 Azure 诊断在 Profiler 命令行上传递正确的 iKey。 

1. 最后，检查 Profiler 日志文件，以查看 Profiler 是否已运行但返回了错误。 

若要检查用于配置 Azure 诊断的设置：

1. 登录到虚拟机 (VM)，然后打开位于此位置的日志文件。 此插件版本在你的计算机上可能会更高。
    
    对于 VM：
    ```
    c:\WindowsAzure\logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.11.3.12\DiagnosticsPlugin.log
    ```
    
    对于云服务：
    ```
    c:\logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.11.3.12\DiagnosticsPlugin.log  
    ```

1. 在该文件中，可以搜索字符串“WadCfg”，找到传递给 VM 用于配置 Azure 诊断的设置。 可以检查 Profiler 接收器使用的 iKey 是否正确。

1. 检查用于启动 Profiler 的命令行。 用于启动 Profiler 的参数位于以下文件中。 （驱动器可能是 C: 或 D: 并且可能会隐藏目录。）

    对于 VM：
    ```
    C:\ProgramData\ApplicationInsightsProfiler\config.json
    ```
    
    对于云服务：
    ```
    D:\ProgramData\ApplicationInsightsProfiler\config.json
    ```

1. 确保 Profiler 命令行中的 iKey 是正确的。 

1. 使用上述 config.json 文件中的路径检查 Profiler 日志文件 BootstrapN.log。 它将显示表示 Profiler 正在使用的设置的调试信息。 此外，还将显示来自 Profiler 的状态和错误消息。  

    对于 Vm，文件如下所示：
    ```
    C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.17.0.6\ApplicationInsightsProfiler
    ```

    对于云服务：
    ```
    C:\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.17.0.6\ApplicationInsightsProfiler
    ```

    如果当应用程序接收请求时 Profiler 正在运行，则会显示以下消息：“检测到来自 iKey 的活动”。 

    上传跟踪时，会显示以下消息：“开始上传跟踪”。 


## <a name="edit-network-proxy-or-firewall-rules"></a>编辑网络代理或防火墙规则

如果你的应用程序通过代理或防火墙连接到 Internet，则可能需要更新规则，以便与探查器服务进行通信。

Application Insights Profiler 使用的 IP 包含在 Azure Monitor 服务标记中。 有关详细信息，请参阅 [服务标记文档](../../virtual-network/service-tags-overview.md)。


[profiler-search-telemetry]:./media/profiler-troubleshooting/Profiler-Search-Telemetry.png
[profiler-webjob]:./media/profiler-troubleshooting/Profiler-webjob.png
[profiler-webjob-log]:./media/profiler-troubleshooting/Profiler-webjob-log.png