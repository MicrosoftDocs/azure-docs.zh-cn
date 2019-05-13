---
title: 在 Azure 中配置服务映射 | Microsoft Docs
description: 服务映射是 Azure 中的解决方案，可自动发现 Windows 和 Linux 系统上的应用程序组件并映射服务之间的通信。 本文提供了有关在环境中部署服务映射并在各种方案中使用它的详细信息。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: d3d66b45-9874-4aad-9c00-124734944b2e
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2019
ms.author: magoedte
ms.openlocfilehash: a50f080ee4d72ad098ca87f5b3a62b42f8cd5907
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142733"
---
# <a name="configure-service-map-in-azure"></a>在 Azure 中配置服务映射

服务映射自动发现 Windows 和 Linux 系统上的应用程序组件并映射服务之间的通信。 借助它，你可以按照自己的想法，将服务器作为提供重要服务的互连系统。 服务映射显示任何 TCP 连接的体系结构中服务器、进程和端口之间的连接，只需安装代理，无需任何其他配置。

本文介绍了配置服务映射和载入代理的详细信息。 有关使用服务映射的信息，请参阅[在 Azure 中使用服务映射解决方案]( service-map.md)。

## <a name="supported-azure-regions"></a>支持的 Azure 区域

服务映射当前在以下 Azure 区域中提供：
- 美国东部
- 美国中西部
- 加拿大中部
- 英国南部
- 西欧
- 东南亚

## <a name="supported-windows-operating-systems"></a>支持的 Windows 操作系统

以下部分列出了 Windows 上 Dependency Agent 支持的操作系统。 

>[!NOTE]
>服务映射仅支持 64 位平台。
>

### <a name="windows-server"></a>Windows Server

- Windows Server 2019
- Windows Server 2016 1803
- Windows Server 2016
- Windows Server 2012 R2
- Windows Server 2012
- Windows Server 2008 R2 SP1

### <a name="windows-desktop"></a>Windows 桌面

- Windows 10 1803
- Windows 10
- Windows 8.1
- Windows 8
- Windows 7

## <a name="supported-linux-operating-systems"></a>受支持的 Linux 操作系统

以下部分列出了 Linux 上 Dependency Agent 支持的操作系统。  

- 仅默认版本和 SMP Linux 内核版本受支持。
- 任何 Linux 分发版都不支持非标准内核版本（例如 PAE 和 Xen）。 例如，不支持版本字符串为“2.6.16.21-0.8-xen”的系统。
- 不支持自定义内核（包括标准内核的重新编译）。

### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| OS 版本 | 内核版本 |
|:--|:--|
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |
| 7.6 | 3.10.0-957 |

### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| OS 版本 | 内核版本 |
|:--|:--|
| 6.9 | 2.6.32-696 |
| 6.10 | 2.6.32-754 |

### <a name="centosplus"></a>CentOSPlus

| OS 版本 | 内核版本 |
|:--|:--|
| 6.9 | 2.6.32-696.18.7<br>2.6.32-696.30.1 |
| 6.10 | 2.6.32-696.30.1<br>2.6.32-754.3.5 |

### <a name="ubuntu-server"></a>Ubuntu Server

| OS 版本 | 内核版本 |
|:--|:--|
| Ubuntu 18.04 | 内核 4.15.\*<br>4.18* |
| Ubuntu 16.04.3 | 内核 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

### <a name="suse-linux-11-enterprise-server"></a>SUSE Linux 11 Enterprise Server

| OS 版本 | 内核版本
|:--|:--|
| 11 SP4 | 3.0.* |

### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| OS 版本 | 内核版本
|:--|:--|
| 12 SP2 | 4.4.* |
| 12 SP3 | 4.4.* |

## <a name="dependency-agent-downloads"></a>Dependency Agent 下载

| 文件 | 操作系统 | 版本 | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.8.1 | 622C99924385CBF539988D759BCFDC9146BB157E7D577C997CDD2674E27E08DD |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.8.1 | 3037934A5D3FB7911D5840A9744AE9F980F87F620A7F7B407F05E276FE7AE4A8 |

## <a name="connected-sources"></a>连接的源

服务映射从 Microsoft Dependency Agent 获取其数据。 Dependency Agent 依赖 Log Analytics 代理连接到 Log Analytics。 这意味着服务器必须首先安装和配置 Log Analytics 代理，然后再安装 Dependency Agent。  下表介绍了服务映射解决方案支持的连接的源。

| 连接的源 | 支持 | 描述 |
|:--|:--|:--|
| Windows 代理 | 是 | 服务映射从 Windows 计算机分析和收集数据。 <br><br>除[适用于 Windows 的 Log Analytics 代理](../../azure-monitor/platform/log-analytics-agent.md)外，Windows 代理还需要 Microsoft Dependency Agent。 查看支持的操作系统和完整的操作系统版本列表。 |
| Linux 代理 | 是 | 服务映射从 Linux 代理计算机分析和收集数据。 <br><br>除[适用于 Linux 的 Log Analytics 代理](../../azure-monitor/platform/log-analytics-agent.md)外，Linux 代理还需要 Microsoft Dependency Agent。 查看支持的操作系统和完整的操作系统版本列表。 |
| System Center Operations Manager 管理组 | 是 | 服务映射在连接的 [System Center Operations Manager 管理组](../../azure-monitor/platform/om-agents.md)中从 Windows 和 Linux 代理分析和收集数据。 <br><br>需要从 System Center Operations Manager 代理计算机直接连接到 Log Analytics。 |
| Azure 存储帐户 | 否 | 服务映射从代理计算机中收集数据，因此其中任何数据都不会从 Azure 存储中收集。 |

在 Windows 中，System Center Operations Manager 和 Log Analytics 都可使用 Microsoft Monitoring Agent (MMA) 收集和发送监视数据。 （根据上下文，可将此代理称为 System Center Operations Manager 代理、Log Analytics 代理、MMA 或直接代理。）System Center Operations Manager 和 Log Analytics 提供不同的 MMA 现成版本。 这些版本每个都可向 System Center Operations Manager 报告，或向 Log Analytics 报告，也可同时向两者报告。  

在 Linux 上，适用于 Linux 的 Log Analytics 代理收集监视数据并将其发送到 Log Analytics。 可以在服务器上配合使用服务映射和直接连接服务或向 Log Analytics 集成的 Operations Manager 管理组报告的 Log Analytics 代理。  

本文中将所有代理都称为“Log Analytics 代理”（不论是 Linux 还是 Windows，也不论是连接到 System Center Operations Manager 管理组还是直接连接到 Log Analytics）。 

服务映射代理本身不传输任何数据，它不需要对防火墙或端口做出任何更改。 服务映射中的数据始终由 Log Analytics 代理直接或通过 Log Analytics 网关传输到 Log Analytics 服务。

![服务映射代理](media/service-map-configure/agents.png)

如果你是一位 System Center Operations Manager 客户且具有连接到 Log Analytics 的管理组：

- 如果 System Center Operations Manager 代理可以访问 Internet 来连接到 Log Analytics，则无需进行额外配置。  
- 如果 System Center Operations Manager 代理无法通过 Internet 访问 Log Analytics，则需配置 Log Analytics 网关来与 System Center Operations Manager 配合使用。
  
如果 Windows 或 Linux 计算机无法直接连接到服务，则需要将 Log Analytics 代理配置为使用网关连接到 Log Analytics 工作区。 有关如何部署和配置 Log Analytics 网关的详细信息，请参阅[使用 Log Analytics 网关连接无法访问 Internet 的计算机](../../azure-monitor/platform/gateway.md)。  

### <a name="management-packs"></a>管理包

在 Log Analytics 工作区中激活服务映射时，将向该工作区中的所有 Windows 服务器转发到 300KB 的管理包。 若在[连接的管理组](../../azure-monitor/platform/om-agents.md)中使用 System Center Operations Manager 代理，则会从 System Center Operations Manager 部署服务映射管理包。 

管理包名为 Microsoft.IntelligencePacks.ApplicationDependencyMonitor。 它将写入 %Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs\。 管理包所使用的数据源是 %Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources\<AutoGeneratedID>\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll。

## <a name="data-collection"></a>数据收集

根据系统依赖关系的复杂性，可预计每个代理每天传输大约 25MB。 每个代理每 15 秒发送一次服务映射依赖关系数据。  

Dependency Agent 通常消耗 0.1% 的系统内存和 0.1% 的系统 CPU。

## <a name="diagnostic-and-usage-data"></a>诊断和使用情况数据

Microsoft 通过使用服务映射服务，自动收集使用情况和性能数据。 Microsoft 使用此数据提供和改进服务映射服务的质量、安全性和完整性。 数据包括有关软件配置的信息（如操作系统和版本）。 还包括 IP 地址、DNS 名称和工作站名称，能够准确高效地排除故障。 我们不收集姓名、地址或其他联系信息。

有关数据收集和使用的详细信息，请参阅 [Microsoft Online Services 隐私声明](https://go.microsoft.com/fwlink/?LinkId=512132)。

## <a name="installation"></a>安装

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

### <a name="azure-vm-extension"></a>Azure VM 扩展

Windows (DependencyAgentWindows) 和 Linux (DependencyAgentLinux) 都有一个扩展，你可以使用 [Azure VM 扩展](https://docs.microsoft.com/azure/virtual-machines/windows/extensions-features)轻松将 Dependency Agent 部署到 Azure VM。  借助 Azure VM 扩展，可以通过 PowerShell 脚本或直接在 VM 中使用 Azure 资源管理器模板将 Dependency Agent 部署到 Windows 和 Linux VM。  如果通过 Azure VM 扩展部署代理，则代理可以自动更新到最新版本。

若要通过 PowerShell 部署 Azure VM 扩展，可以使用以下示例：

```powershell
#
# Deploy the Dependency agent to every VM in a Resource Group
#

$version = "9.4"
$ExtPublisher = "Microsoft.Azure.Monitoring.DependencyAgent"
$OsExtensionMap = @{ "Windows" = "DependencyAgentWindows"; "Linux" = "DependencyAgentLinux" }
$rmgroup = "<Your Resource Group Here>"

Get-AzVM -ResourceGroupName $rmgroup |
ForEach-Object {
    ""
    $name = $_.Name
    $os = $_.StorageProfile.OsDisk.OsType
    $location = $_.Location
    $vmRmGroup = $_.ResourceGroupName
    "${name}: ${os} (${location})"
    Date -Format o
    $ext = $OsExtensionMap.($os.ToString())
    $result = Set-AzVMExtension -ResourceGroupName $vmRmGroup -VMName $name -Location $location `
    -Publisher $ExtPublisher -ExtensionType $ext -Name "DependencyAgent" -TypeHandlerVersion $version
    $result.IsSuccessStatusCode
}
```

可通过一种更为简单的方法来确保 Dependency Agent 安装在每个 VM 上，即在 Azure 资源管理器模板中包含此代理。  以下 JSON 代码示例可以添加到模板的“资源”节中。

```JSON
"type": "Microsoft.Compute/virtualMachines/extensions",
"name": "[concat(parameters('vmName'), '/DependencyAgent')]",
"apiVersion": "2017-03-30",
"location": "[resourceGroup().location]",
"dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
],
"properties": {
    "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
    "type": "DependencyAgentWindows",
    "typeHandlerVersion": "9.4",
    "autoUpgradeMinorVersion": true
}

```

### <a name="install-the-dependency-agent-on-microsoft-windows"></a>在 Microsoft Windows 上安装 Dependency Agent

可通过运行 `InstallDependencyAgent-Windows.exe` 在 Windows 计算机上手动安装 Dependency Agent。 如果在没有任何选项的情况下运行此可执行文件，它将启动一个安装向导，以交互方式指导用户安装。  

>[!NOTE]
>需要管理员特权才能安装或卸载代理。

使用以下步骤在每台 Windows 计算机上安装 Dependency Agent：

1.  按照 [Log Analytics 代理概述](../../azure-monitor/platform/log-analytics-agent.md)中所述的某种方法安装适用于 Windows 的 Log Analytics 代理。
2.  下载 Windows 代理，并使用以下命令运行： 
    
    `InstallDependencyAgent-Windows.exe`

3.  按照安装向导安装代理。
4.  如果 Dependency Agent 无法启动，请检查日志以获取详细的错误信息。 在 Windows 代理上，日志目录是 %Programfiles%\Microsoft Dependency Agent\logs。 

#### <a name="windows-command-line"></a>Windows 命令行

使用下表中的选项从命令行进行安装。 若要查看安装标志列表，请运行安装程序并使用 /? 标志运行安装程序，如下所示。

    InstallDependencyAgent-Windows.exe /?

| 标志 | 描述 |
|:--|:--|
| /? | 获取命令行选项列表。 |
| /S | 执行无提示安装，无用户提示。 |

默认情况下，Windows Dependency Agent的文件位于 C:\Program Files\Microsoft Dependency Agent 中。

### <a name="install-the-dependency-agent-on-linux"></a>在 Linux 上安装 Dependency Agent

通过 `InstallDependencyAgent-Linux64.bin`（包含自解压缩二进制文件的 Shell 脚本）在 Linux 计算机上安装 Dependency Agent。 可使用 `sh` 来运行该文件或将执行权限添加到文件本身。

>[!NOTE]
> 需要根目录访问才能安装或配置代理。

使用以下步骤在每台 Linux 计算机上安装 Dependency Agent：

1.  按照 [Log Analytics 代理概述](../../azure-monitor/platform/log-analytics-agent.md)中所述的某种方法安装 Log Analytics 代理。
2.  运行以下命令将 Linux Dependency Agent 安装为根：
    
    `sh InstallDependencyAgent-Linux64.bin`

3.  如果 Dependency Agent 无法启动，请检查日志以获取详细的错误信息。 在 Linux 代理上，日志目录是 /var/opt/microsoft/dependency-agent/log。

若要查看安装标志列表，请运行带有 -help 标志的安装程序，如下所示。

    InstallDependencyAgent-Linux64.bin -help

| 标志 | 描述 |
|:--|:--|
| -help | 获取命令行选项列表。 |
| -s | 执行无提示安装，无用户提示。 |
| --check | 检查权限和操作系统，但不安装代理。 |

Dependency Agent 的文件放置在以下目录中：

| 文件 | 位置 |
|:--|:--|
| 核心文件 | /opt/microsoft/dependency-agent |
| 日志文件 | /var/opt/microsoft/dependency-agent/log |
| 配置文件 | /etc/opt/microsoft/dependency-agent/config |
| 服务可执行文件 | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| 二进制存储文件 | /var/opt/microsoft/dependency-agent/storage |

## <a name="installation-script-examples"></a>安装脚本示例

要在多台服务器上同时轻松部署 Dependency Agent，可以使用以下脚本示例在 Windows 或 Linux 上下载和安装 Dependency Agent。

### <a name="powershell-script-for-windows"></a>适用于 Windows 的 PowerShell 脚本

```powershell
Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S
```

### <a name="shell-script-for-linux"></a>适用于 Linux 的 Shell 脚本

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
sudo sh InstallDependencyAgent-Linux64.bin -s
```

## <a name="desired-state-configuration"></a>Desired State Configuration

若通过期望状态配置 (DSC) 部署 Dependency Agent，可通过如下示例代码使用 xPSDesiredStateConfiguration 模块：

```powershell
configuration ServiceMap {

    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    $DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"

    Node localhost
    {
        # Download and install the Dependency agent
        xRemoteFile DAPackage 
        {
            Uri = "https://aka.ms/dependencyagentwindows"
            DestinationPath = $DAPackageLocalPath
        }

        xPackage DA
        {
            Ensure="Present"
            Name = "Dependency Agent"
            Path = $DAPackageLocalPath
            Arguments = '/S'
            ProductId = ""
            InstalledCheckRegKey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"
            InstalledCheckRegValueName = "DisplayName"
            InstalledCheckRegValueData = "Dependency Agent"
            DependsOn = "[xRemoteFile]DAPackage"
        }
    }
}
```

## <a name="remove-the-dependency-agent"></a>删除 Dependency Agent

### <a name="uninstall-agent-on-windows"></a>在 Windows 上卸载代理

管理员可通过“控制面板”卸载适用于 Windows 的 Dependency Agent。

管理员还可以运行 %Programfiles%\Microsoft Dependency Agent\Uninstall.exe 卸载 Dependency Agent。

### <a name="uninstall-agent-on-linux"></a>在 Linux 上卸载代理

通过以下命令，可从 Linux 卸载 Dependency Agent。

RHEL、CentOs 或 Oracle：

```
sudo rpm -e dependency-agent
```

Ubuntu：

```
sudo apt -y purge dependency-agent
```

## <a name="troubleshooting"></a>故障排除

如果安装或运行服务映射时遇到任何问题，可通过本部分内容获得帮助。 如果仍然无法解决问题，请联系 Microsoft 支持部门。

### <a name="dependency-agent-installation-problems"></a>Dependency Agent 安装问题

#### <a name="installer-prompts-for-a-reboot"></a>安装程序提示重新启动

安装或卸载 Dependency Agent 时，通常不需要重启。 在极少数的某些情况下，Windows Server 需要重启才能继续安装。 依赖关系（通常是 Microsoft Visual C++ 可再发行组件）因锁定的文件而需要重启时会发生这种情况。

#### <a name="message-unable-to-install-dependency-agent-visual-studio-runtime-libraries-failed-to-install-code--codenumber-appears"></a>出现消息“无法安装 Dependency Agent:Visual Studio 运行时库安装失败(code = [code_number])”

Microsoft Dependency Agent 基于 Microsoft Visual Studio 运行时库。 如果安装库时出现问题，将收到一条消息。 

运行时库安装程序在 %LOCALAPPDATA%\temp 文件夹中创建日志。 该文件为 dd_vcredist_arch_yyyymmddhhmmss.log，其中体系结构为“x86”或“amd64”，yyyymmddhhmmss 为创建该日志时的日期和时间（24 小时制）。 该日志提供有关阻止安装的问题的详细信息。

这可能有助于首次自行安装[最新运行时库](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)。

下表列出了代码号和建议的解决方法。

| 代码 | 描述 | 解决方法 |
|:--|:--|:--|
| 0x17 | 库安装程序需要尚未安装的 Windows 更新。 | 查看最新的库安装程序日志。<br><br>如果对“Windows8.1-KB2999226-x64.msu”的引用后跟一行“错误 0x80240017:无法执行 MSU 包”，则没有安装 KB2999226 的先决条件。 请遵循 [Windows 中的 Universal C Runtime](https://support.microsoft.com/kb/2999226) 中必备组件部分的说明。 可能需要运行 Windows 更新并重新启动多次，才能安装好必备组件。<br><br>再次运行 Microsoft Dependency Agent 安装程序。 |

### <a name="post-installation-issues"></a>安装后的问题

#### <a name="server-doesnt-appear-in-service-map"></a>服务映射中不显示服务器

如果已成功安装 Dependency Agent，但在服务映射解决方案中看不到服务器：
* Dependency Agent 是否已安装成功？ 可通过检查是否已安装并运行服务来验证这一点。<br><br>
Windows：查找名为“Microsoft Dependency Agent”的服务。<br>
Linux：查找正在运行的进程“microsoft-dependency-agent”。

* 是否属于 [Operations Management Suite/Log Analytics 的免费定价层](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions)？ 免费计划允许最多 5 个仅有的服务映射服务器。 服务映射中不再有任何其他的服务器，即使前 5 个服务器不再发送数据。

* 服务器是否正在向 Log Analytics 发送日志和性能数据？ 转到日志搜索，并为计算机运行以下查询： 

    Usage | where Computer == "computer-name" | summarize sum(Quantity), any(QuantityUnit) by DataType

结果中是否有多种不同的事件？ 是否为最新数据？ 如果是，则表示 Log Analytics 代理正常运行并正在与 Log Analytics 通信。 如果不是，请检查服务器上的代理：[适用于 Windows 的 Log Analytics 代理故障排除](https://support.microsoft.com/help/3126513/how-to-troubleshoot-monitoring-onboarding-issues)或[适用于 Linux 的 Log Analytics 代理故障排除](../../azure-monitor/platform/agent-linux-troubleshoot.md)。

#### <a name="server-appears-in-service-map-but-has-no-processes"></a>服务器会在服务映射中显示，但没有任何进程

如果在服务映射中看到了服务器，但没有任何进程或连接数据，则表明已安装并运行 Dependency Agent，但未加载内核驱动程序。 

请检查 C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log file（针对 Windows）或 /var/opt/microsoft/dependency-agent/log/service.log file（针对 Linux）。 文件的最后几行应指出为何未加载内核。 例如，如果更新内核，则内核在 Linux 上可能不受支持。

## <a name="next-steps"></a>后续步骤

- 部署和配置服务映射后，了解如何[使用服务映射]( service-map.md)。