---
title: Azure 诊断扩展故障排除
description: 排查在 Azure 虚拟机、Service Fabric 或云服务中使用 Azure 诊断时遇到的问题。
services: azure-monitor
author: rboucher
ms.service: azure-monitor
ms.subservice: diagnostic-extension
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: robb
ms.openlocfilehash: 99ac4ffc288773e52183d371ef2c20f6153bc0f3
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471789"
---
# <a name="azure-diagnostics-troubleshooting"></a>Azure 诊断故障排除
本文介绍有关使用 Azure 诊断的故障排除信息。 有关 Azure 诊断的详细信息，请参阅 [Azure 诊断概述](diagnostics-extension-overview.md)。

## <a name="logical-components"></a>逻辑组件
**诊断插件启动器 (DiagnosticsPluginLauncher.exe)**：启动 Azure 诊断扩展。 用作入口点进程。

**诊断插件 (DiagnosticsPlugin.exe)**：配置、启动和管理监视代理的生存期。 这是由启动器启动的主要进程。

**监视代理（MonAgent\*.exe 进程）**：监视、收集和传输诊断数据。  

## <a name="logartifact-paths"></a>日志/项目路径
以下是一些重要日志和项目的路径。 文档剩余部分将始终引用此信息。

### <a name="azure-cloud-services"></a>Azure 云服务
| 项目 | 路径 |
| --- | --- |
| **Azure 诊断配置文件** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\Config.txt |
| **日志文件** | C:\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\ |
| **诊断数据的本地存储** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Tables |
| **监视代理配置文件** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MaConfig.xml |
| **Azure 诊断扩展包** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version> |
| **日志收集实用工具路径** | %SystemDrive%\Packages\GuestAgent\ |
| **MonAgentHost 日志文件** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

### <a name="virtual-machines"></a>虚拟机
| 项目 | 路径 |
| --- | --- |
| **Azure 诊断配置文件** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\RuntimeSettings |
| **日志文件** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\ |
| **诊断数据的本地存储** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Tables |
| **监视代理配置文件** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MaConfig.xml |
| **状态文件** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\Status |
| **Azure 诊断扩展包** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>|
| **日志收集实用工具路径** | C:\WindowsAzure\Logs\WaAppAgent.log |
| **MonAgentHost 日志文件** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

## <a name="metric-data-doesnt-appear-in-the-azure-portal"></a>指标数据不显示在 Azure 门户中
Azure 诊断提供可在 Azure 门户中显示的指标数据。 如果无法查看门户中的这些数据，请检查 Azure 诊断存储帐户中的 WADMetrics\* 表，以查看是否存在相应的指标记录。

此处，表的 PartitionKey 是资源 ID、虚拟机或虚拟机规模集。 RowKey 是指标名称（也称为性能计数器名称）。

如果资源 ID 不正确，请检查“诊断配置” > “指标” > “ResourceId”，以查看是否已正确设置资源 ID。

如果没有特定指标数据，请检查“诊断配置” > “PerformanceCounter”，以查看是否包含指标（性能计数器）。 默认启用以下计数器：
- \Processor(_Total)\% 处理器时间
- \Memory\Available Bytes
- \ASP.NET Applications(__Total__)\Requests/Sec
- \ASP.NET Applications(__Total__)\Errors Total/Sec
- \ASP.NET\Requests Queued
- \ASP.NET\Requests Rejected
- \Processor(w3wp)\% Processor Time
- \Process(w3wp)\Private Bytes
- \Process(WaIISHost)\% Processor Time
- \Process(WaIISHost)\Private Bytes
- \Process(WaWorkerHost)\% Processor Time
- \Process(WaWorkerHost)\Private Bytes
- \Memory\Page Faults/sec
- \.NET CLR Memory(_Global_)\% Time in GC
- \LogicalDisk(C:)\Disk Write Bytes/sec
- \LogicalDisk(C:)\Disk Read Bytes/sec
- \LogicalDisk(D:)\Disk Write Bytes/sec
- \LogicalDisk(D:)\Disk Read Bytes/sec

如果配置设置正确，但仍看不到指标数据，请按照以下指南进行故障排除。


## <a name="azure-diagnostics-is-not-starting"></a>Azure 诊断不启动
有关为和 Azure 诊断无法启动的信息，请参阅之前提供的日志文件位置中的 DiagnosticsPluginLauncher.log 和 DiagnosticsPlugin.log 文件。

如果这些日志指示 `Monitoring Agent not reporting success after launch`，则表示启动 MonAgentHost.exe 失败。 在之前部分中指示 `MonAgentHost log file` 的位置查看日志。

日志文件的最后一行包含退出代码。  

```
DiagnosticsPluginLauncher.exe Information: 0 : [4/16/2016 6:24:15 AM] DiagnosticPlugin exited with code 0
```
如果发现退出代码为负数，请参阅[参考部分](#references)中的[退出代码表](#azure-diagnostics-plugin-exit-codes)。

## <a name="diagnostics-data-is-not-logged-to-azure-storage"></a>未将诊断数据记录到 Azure 存储
确定是未显示数据还是仅显示部分数据。

### <a name="diagnostics-infrastructure-logs"></a>诊断基础结构日志
诊断将在诊断基础结构日志中记录所有错误。 请确保已[在配置中允许捕获诊断基础结构日志](#how-to-check-diagnostics-extension-configuration)。 然后，可在配置的存储帐户中快速查找 `DiagnosticInfrastructureLogsTable` 表中显示的任何相关错误。

### <a name="no-data-is-appearing"></a>未显示数据
完全不显示事件数据的最常见原因，是存储帐户信息定义不正确。

解决方案：更正诊断配置，然后重新安装诊断。

如果存储帐户是配置正确，远程访问计算机并确认*DiagnosticsPlugin.exe*并*MonAgentCore.exe*正在运行。 如果未运行，请按照 [Azure 诊断不启动](#azure-diagnostics-is-not-starting)中的步骤进行操作。

如果进程正在运行，请转到[数据是否是本地捕获的？](#is-data-getting-captured-locally)并按此处的介绍进行操作。

如果这不能解决问题，请尝试为：

1. 卸载代理
2. Remove directory C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics
3. 重新安装代理


### <a name="part-of-the-data-is-missing"></a>部分数据丢失
如果获得了部分数据，则表示数据收集/传输管道设置正确。 请按照此处的子节说明，缩小问题范围。

#### <a name="is-the-collection-configured"></a>是否配置了收集？
诊断配置包含要收集的特定类型数据的介绍。 [查看配置](#how-to-check-diagnostics-extension-configuration)以验证是否仅查找已配置进行收集的数据。

#### <a name="is-the-host-generating-data"></a>主机是否生成数据？
- **性能计数器**：打开 perfmon 并检查计数器。

- **跟踪日志**：远程访问 VM 并向应用的配置文件添加 TextWriterTraceListener。  请参阅 https://msdn.microsoft.com/library/sk36c28t.aspx 设置文本侦听器。  确保 `<trace>` 元素具有 `<trace autoflush="true">`。<br />
如果没有看到生成跟踪日志，请查看“关于跟踪日志丢失的更多信息”。

- **ETW 跟踪**：远程访问 VM 并安装 PerfView。  在 PerfView 中运行“文件” > “用户命令” > “侦听 etwprovder1” > “etwprovider2”等。 侦听命令区分大小写，ETW 提供程序的逗号分隔列表之间不能有空格。 如果命令未能运行，可选择 Perfview 工具右下角的“日志”按钮，查看尝试运行的内容以及结果。  假设输入正确，则会弹出一个新窗口。 几秒钟后，即可看到 ETW 跟踪信息。

- **事件日志**：远程访问 VM。 打开 `Event Viewer`确保事件存在。

#### <a name="is-data-getting-captured-locally"></a>数据是否是本地捕获的？
接下来，请确保本地捕获到了数据。
数据本地存储在诊断数据本地存储中的 `*.tsf` 文件中。 不同类型的日志被收集在不同的 `.tsf` 文件中。 文件名称与 Azure 存储中的表名相似。

例如，`Performance Counters` 收集在 `PerformanceCountersTable.tsf` 中。 事件日志收集在 `WindowsEventLogsTable.tsf` 中。 使用[本地日志提取](#local-log-extraction)部分中的说明打开本地收集文件，验证它们是否是在磁盘上收集的。

如果没有看到在本地收集日志，并且已确认主机正在生成数据，则可能是有配置问题。 仔细查看配置。

还要查看为 MonitoringAgent MaConfig.xml 生成的配置。 验证是否存在描述相关日志源的部分。 然后验证该部分是否在诊断配置和监视代理配置间的转换中丢失。

#### <a name="is-data-getting-transferred"></a>是否传输了数据？
如果已验证数据是本地捕获的，但仍未在存储帐户中看到它，请按以下步骤操作：

- 验证提供存储帐户是否正确，且是否有为给定的存储帐户滚动更新过密钥。 对于 Azure 云服务，有时我们发现人们不更新其 `useDevelopmentStorage=true`。

- 验证提供的存储帐户是否正确。 确保未受到一些禁止组件访问公共存储终结点的网络限制。 执行此操作的一种方法是远程访问计算机，自行尝试向相同的存储帐户写入一些内容。

- 最后，可并查看监视代理报告的故障。 监视代理将其日志写入位于诊断数据本地存储中的 `maeventtable.tsf`。 按照[本地日志提取](#local-log-extraction)部分中的说明打开此文件。 然后尝试确定是否有 `errors` 指示读取本地文件或写入存储失败。

### <a name="capturing-and-archiving-logs"></a>捕获并存档日志
若打算联系支持人员，他们可能会要求你做的第一件事是从计算机收集日志。 可以通过自行收集节省时间。 在日志收集实用程序路径上运行 `CollectGuestLogs.exe` 实用程序。 它会在同一文件夹中生成一个 .zip 文件，其中包含所有相关 Azure 日志。

## <a name="diagnostics-data-tables-not-found"></a>找不到诊断数据表
Azure 存储中保存 ETW 事件的表是使用以下代码命名的：

```csharp
        if (String.IsNullOrEmpty(eventDestination)) {
            if (e == "DefaultEvents")
                tableName = "WADDefault" + MD5(provider);
            else
                tableName = "WADEvent" + MD5(provider) + eventId;
        }
        else
            tableName = "WAD" + eventDestination;
```

下面是一个示例：

```XML
        <EtwEventSourceProviderConfiguration provider="prov1">
          <Event id="1" />
          <Event id="2" eventDestination="dest1" />
          <DefaultEvents />
        </EtwEventSourceProviderConfiguration>
        <EtwEventSourceProviderConfiguration provider="prov2">
          <DefaultEvents eventDestination="dest2" />
        </EtwEventSourceProviderConfiguration>
```
```JSON
"EtwEventSourceProviderConfiguration": [
    {
        "provider": "prov1",
        "Event": [
            {
                "id": 1
            },
            {
                "id": 2,
                "eventDestination": "dest1"
            }
        ],
        "DefaultEvents": {
            "eventDestination": "DefaultEventDestination",
            "sinks": ""
        }
    },
    {
        "provider": "prov2",
        "DefaultEvents": {
            "eventDestination": "dest2"
        }
    }
]
```
此代码生成四个表：

| 事件 | 表名称 |
| --- | --- |
| provider=”prov1” &lt;Event id=”1” /&gt; |WADEvent+MD5(“prov1”)+”1” |
| provider=”prov1” &lt;Event id=”2” eventDestination=”dest1” /&gt; |WADdest1 |
| provider=”prov1” &lt;DefaultEvents /&gt; |WADDefault+MD5(“prov1”) |
| provider=”prov2” &lt;DefaultEvents eventDestination=”dest2” /&gt; |WADdest2 |

## <a name="references"></a>参考

### <a name="how-to-check-diagnostics-extension-configuration"></a>如何检查诊断扩展配置
检查扩展配置的最简方法是转到 [Azure 资源浏览器](http://resources.azure.com)、再转到 Azure 诊断扩展 (IaaSDiagnostics/PaaDiagnostics) 所在的虚拟机或云服务。

或者，通过远程桌面连接到计算机并查看“日志项目路径部分”中所述的 Azure 诊断配置文件。

在任何一种情况下，都请先搜索“Microsoft.Azure.Diagnostics”，再搜索“xmlCfg”或“WadCfg”字段。

如果在虚拟机上进行搜索，且存在 WadCfg 字段，则表示配置为 JSON 格式。 如果存在 xmlCfg 字段，则表示配置在 XML 中，且已进行 base64 编码。 你需要[将其解码](https://www.bing.com/search?q=base64+decoder)才能查看诊断加载的 XML。

对于云服务角色，如果从磁盘选择配置，数据采用 base64 编码，则需要[将其解码](https://www.bing.com/search?q=base64+decoder)才能查看诊断加载的 XML。

### <a name="azure-diagnostics-plugin-exit-codes"></a>Azure 诊断插件退出代码
该插件返回以下退出代码：

| 退出代码 | 描述 |
| --- | --- |
| 0 |成功。 |
| -1 |常规错误。 |
| -2 |无法加载 rcf 文件。<p>仅当在 VM 上不正确地手动调用了来宾代理插件启动器时，才会发生此内部错误。 |
| -3 |无法加载诊断配置文件。<p><p>解决方案：这是配置文件未通过架构验证的结果。 解决方案是提供符合架构的配置文件。 |
| -4 |监视代理诊断的另一个实例已在使用本地资源目录。<p><p>解决方案：为“LocalResourceDirectory”指定不同的值。 |
| -6 |来宾代理插件启动器尝试使用无效的命令行启动诊断。<p><p>仅当在 VM 上不正确地手动调用了来宾代理插件启动器时，才会发生此内部错误。 |
| -10 |Diagnostics 插件退出并返回未处理的异常。 |
| -11 |来宾代理程序无法创建负责启动和监视监视代理的进程。<p><p>解决方案：验证是否有足够的系统资源可用于启动新进程。<p> |
| -101 |调用诊断插件时参数无效。<p><p>仅当在 VM 上不正确地手动调用了来宾代理插件启动器时，才会发生此内部错误。 |
| -102 |插件进程将无法初始化自身。<p><p>解决方案：验证是否有足够的系统资源可用于启动新进程。 |
| -103 |插件进程将无法初始化自身。 具体而言，它无法创建记录器对象。<p><p>解决方案：验证是否有足够的系统资源可用于启动新进程。 |
| -104 |无法加载来宾代理提供的 rcf 文件。<p><p>仅当在 VM 上不正确地手动调用了来宾代理插件启动器时，才会发生此内部错误。 |
| -105 |诊断插件无法打开诊断配置文件。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了诊断插件时才会发生。 |
| -106 |无法读取诊断配置文件。<p><p>这是配置文件未通过架构验证的结果。 <br><br>解决方案：提供符合架构的配置文件。 有关详细信息，请参阅[如何检查诊断扩展配置](#how-to-check-diagnostics-extension-configuration)。 |
| -107 |传递给监视代理的资源目录无效。<p><p>此内部错误应仅当在 VM 上不正确地手动调用了监视代理时才会发生。</p> |
| -108 |无法将诊断配置文件转换为监视代理配置文件。<p><p>此内部错误应仅当使用无效的配置文件手动调用了诊断插件时才会发生。 |
| -110 |常规诊断配置错误。<p><p>此内部错误应仅当使用无效的配置文件手动调用了诊断插件时才会发生。 |
| -111 |无法启动监视代理。<p><p>解决方案：验证是否有足够的系统资源可用。 |
| -112 |常见错误 |

### <a name="local-log-extraction"></a>本地日志提取
监视代理将日志和项目收集为 `.tsf` 文件。 `.tsf` 文件不可读，但可以将其转换为 `.csv`，如下所示：

```
<Azure diagnostics extension package>\Monitor\x64\table2csv.exe <relevantLogFile>.tsf
```
将在与相应 `.tsf` 文件相同的路径中创建一个名为 `<relevantLogFile>.csv` 的新文件。

>[!NOTE]
> 只需对主 tsf 文件（例如 PerformanceCountersTable.tsf）运行此实用工具。 将自动处理随附的文件（例如 PerformanceCountersTables_\*\*001.tsf、PerformanceCountersTables_\*\*002.tsf 等）。

### <a name="more-about-missing-trace-logs"></a>关于跟踪日志丢失的更多信息

>[!NOTE]
> 以下信息主要适用于 Azure 云服务，除非已在于 IaaS VM 上运行的应用程序上配置了 DiagnosticsMonitorTraceListener。

- 确保在 web.config 或 app.config 中配置了 DiagnosticMonitorTraceListener。这是云服务项目中默认配置的。 然而，某些客户将其注释掉了，导致诊断不收集相关跟踪语句。

- 如果没有从 OnStart 或 Run 方法写入日志，请确保 DiagnosticMonitorTraceListener 位于 app.config 中。默认情况下，它位于 web.config 中，但这仅适用于在 w3wp.exe 中运行的代码。 因此需将其置于 app.config 中，以捕获在 WaIISHost.exe 中运行的跟踪。

- 确保使用的是 Diagnostics.Trace.TraceXXX，而不是 Diagnostics.Debug.WriteXXX 从发布版本中删除了 Debug 语句。

- 确保已编译的代码实际上具有 Diagnostics.Trace 行（使用反射器、ildasm 或 ILSpy 验证）。 从已编译的二进制文件中删除了 Diagnostics.Trace 命令，除非使用 TRACE 条件编译符号。 这是使用 msbuild 构建项目时常会发生的问题。   

## <a name="known-issues-and-mitigations"></a>已知问题和缓解措施
下面是已知问题和已知缓解措施的列表：

**1. .NET 4.5 依赖项**

Microsoft Azure 诊断扩展对于 .NET 4.5 框架或更高版本存在运行时依赖关系。 在撰写时，为 Azure 云服务配置的所有计算机以及基于 Azure 虚拟机的所有官方映像都安装了 .NET 4.5 或更高版本。

尝试在未安装 .NET 4.5 或更高版本的计算机上运行 Microsoft Azure 诊断扩展时，仍可能遇到问题。 从旧映像或快照创建计算机或者自带自定义磁盘时，会发生这种情况。

这在运行 DiagnosticsPluginLauncher.exe 时通常显示为退出代码 255。 由以下未经处理的异常引发的故障：
```
System.IO.FileLoadException: Could not load file or assembly 'System.Threading.Tasks, Version=1.5.11.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies
```

**缓解：** 在计算机上安装 .NET 4.5 或更高版本。

**2.性能计数器数据在存储中可用，但不显示在门户中**

默认情况下，虚拟机中的门户体验会显示某些性能计数器。 如果未看到性能计数器，且知道正在生成数据，因为数据在存储中可用，此时进行以下检查：

- 存储中的数据是否有英文计数器名称。 如果计数器名称不是英文，门户指标图表将无法识别它。 **缓解措施**：将系统帐户的计算机语言更改为英语。 要执行此操作，请选择“控制面板” > “区域” > “管理” > “复制设置”。 接下来，取消选择“欢迎界面和系统帐户”，以免将自定义语言应用到系统帐户。

- 如果在性能计数器名称中使用通配符 (\*)，则在将性能计数器发送到 Azure 存储接收器时，门户将无法关联已配置和已收集的计数器。 **缓解措施**：要确保可以使用通配符并让门户展开 (\*)，请将性能计数器路由到[“Azure Monitor”接收器](diagnostics-extension-schema.md#diagnostics-extension-111)。

