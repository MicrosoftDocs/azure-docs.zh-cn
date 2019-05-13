---
title: Azure 状态监视器 v2 疑难解答和已知的问题 |Microsoft Docs
description: 状态监视器 v2 和故障排除示例的已知的问题。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
services: application-insights
documentationcenter: .net
author: MS-TimothyMothra
manager: alexklim
ms.assetid: 769a5ea4-a8c6-4c18-b46c-657e864e24de
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: tilee
ms.openlocfilehash: d640206fd72b4eb89afe5ed1750627823bca9637
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415877"
---
# <a name="troubleshooting-status-monitor-v2"></a>状态的故障排除监视 v2

启用监视时, 可能会遇到问题，数据收集。 本文档列出了所有已知的问题和故障排除示例。
如果你遇到此处未列出的问题，则可联系我们[此处](https://github.com/Microsoft/ApplicationInsights-Home/issues)。


> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="known-issues"></a>已知问题

### <a name="conflicting-dlls-in-an-applications-bin-directory"></a>应用程序的 bin 目录中的冲突 Dll

如果其中任何 Dll 的 bin 目录中存在，监视可能会失败。

- Microsoft.ApplicationInsights.dll
- Microsoft.AspNet.TelemetryCorrelation.dll
- System.Diagnostics.DiagnosticSource.dll

一些这些 Dll 包括在 Visual Studio 的默认应用程序模板，即使应用程序不使用它们。
可以使用故障排除工具查看症状行为：

- PerfView:
    ```
    ThreadID="7,500" 
    ProcessorNumber="0" 
    msg="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ExtVer="2.8.13.5972" 
    SubscriptionId="" 
    AppName="" 
    FormattedMessage="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ```

- iisreset + 应用程序负载 （无遥测）。 调查使用 Sysinternals （Handle.exe 和 ListDLLs.exe）
    ```
    .\handle64.exe -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
    E54: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

    .\Listdlls64.exe w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"
    0x0000000009be0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
    0x0000000009b90000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
    0x0000000004d20000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
    ```

### <a name="conflict-with-iis-shared-configuration"></a>使用 IIS 共享配置冲突

如果你拥有的 web 服务器群集，则可能使用[共享配置](https://docs.microsoft.com/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211)。 我们无法自动我们 HttpModule 注入此共享的配置。每个 web 服务器首先必须运行启用命令要将我们 DLL 安装到 GAC。

在您运行启用命令， 
1. 浏览到你共享的配置的目录，并找到你`applicationHost.config`文件。
2. 将此行添加到模块部分中的配置：
    ```
    <modules>
        <!-- Registered global managed http module handler. The 'Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll' must be installed in the GAC before this config is applied. -->
        <add name="ManagedHttpModuleHelper" type="Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" preCondition="managedHandler,runtimeVersionv4.0" />
    </modules>
    ```
    


## <a name="troubleshooting"></a>故障排除
    
### <a name="troubleshooting-powershell"></a>故障排除的 PowerShell

#### <a name="how-to-inspect-what-modules-are-available"></a>如何检查以及模块是否可用？
你可以审核已安装的模块使用命令： `Get-Module -ListAvailable`

#### <a name="how-to-import-a-module-into-the-current-session"></a>如何将模块导入到当前会话？
如果该模块尚未加载到 PowerShell 会话，可以手动加载使用命令： `Import-Module <path to psd1>`


### <a name="troubleshooting-the-status-monitor-v2-module"></a>故障排除状态监视器 v2 模块

#### <a name="how-to-review-what-commands-are-available-in-the-status-monitor-v2-module"></a>如何查看状态监视器 v2 模块中有哪些命令？
- 运行命令：`Get-Command -Module Az.ApplicationMonitor`若要获取可用命令：

    ```
    CommandType     Name                                               Version    Source
    -----------     ----                                               -------    ------
    Cmdlet          Disable-ApplicationInsightsMonitoring              0.2.1      Az.ApplicationMonitor
    Cmdlet          Disable-InstrumentationEngine                      0.2.1      Az.ApplicationMonitor
    Cmdlet          Enable-ApplicationInsightsMonitoring               0.2.1      Az.ApplicationMonitor
    Cmdlet          Enable-InstrumentationEngine                       0.2.1      Az.ApplicationMonitor
    Cmdlet          Get-ApplicationInsightsMonitoringConfig            0.2.1      Az.ApplicationMonitor
    Cmdlet          Get-ApplicationInsightsMonitoringStatus            0.2.1      Az.ApplicationMonitor
    Cmdlet          Set-ApplicationInsightsMonitoringConfig            0.2.1      Az.ApplicationMonitor
    ```

#### <a name="what-is-the-current-version-of-the-status-monitor-v2-module"></a>状态监视器 v2 模块的当前版本是什么？
- 运行命令：`Get-ApplicationInsightsMonitoringStatus`以获取有关此模块的信息的输出：
    - PowerShell 模块版本
    - Application Insights SDK 版本
    - PowerShell 模块的文件路径
    
查看我们[API 参考](status-monitor-v2-api-get-status.md)有关如何使用此 cmdlet 的详细说明。



### <a name="troubleshooting-running-processes"></a>故障排除正在运行的进程

您可以检查以查看是否已加载所有 Dll 检测计算机上的进程。
监视正常运行，如果应加载至少为 12 的 DLL。

- Cmd: `Get-ApplicationInsightsMonitoringStatus -InspectProcess`

查看我们[API 参考](status-monitor-v2-api-get-status.md)有关如何使用此 cmdlet 的详细说明。


### <a name="collect-etw-logs-with-perfview"></a>收集使用 PerfView ETW 日志

#### <a name="setup"></a>设置

1. 下载 PerfView.exe 和从 PerfView64.exe https://github.com/Microsoft/perfview/releases
2. 启动 PerfView64.exe
3. 展开"高级选项"
4. 取消选中：
    - Zip
    - 合并
    - .NET 符号集合
5. 设置其他提供程序： `61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,925fa42b-9ef6-5fa7-10b8-56449d7a2040,f7d60e07-e910-5aca-bdd2-9de45b46c560,7c739bb9-7861-412e-ba50-bf30d95eae36,61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,252e28f4-43f9-5771-197a-e8c7e750a984`


#### <a name="collecting-logs"></a>正在收集日志

1. 在具有管理员权限的 cmd 控制台中，执行`iisreset /stop`若要关闭 IIS 和所有 web 应用。
2. 在 PerfView 中，单击"启动集合"
3. 在具有管理员权限的 cmd 控制台中，执行`iisreset /start`启动 IIS。
4. 尝试浏览到您的应用程序。
5. 你的应用完成加载后，返回到 PerfView 并单击"停止收集"



## <a name="next-steps"></a>后续步骤

- 查看我们[API 参考](status-monitor-v2-overview.md#powershell-api-reference)若要查找你可能已经遗漏了一个参数。
- 如果你遇到此处未列出的问题，则可联系我们[此处](https://github.com/Microsoft/ApplicationInsights-Home/issues)。
