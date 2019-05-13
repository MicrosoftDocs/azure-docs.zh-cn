---
title: Azure 中的更新管理解决方案
description: 本文旨在帮助你了解如何使用 Azure 更新管理解决方案来管理 Windows 和 Linux 计算机的更新。
services: automation
ms.service: automation
ms.subservice: update-management
author: georgewallace
ms.author: gwallace
ms.date: 04/29/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 43ca5bbfd789c1c41826c9a13a0030b80c72d4bc
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510656"
---
# <a name="update-management-solution-in-azure"></a>Azure 中的更新管理解决方案

可以使用在 Azure 自动化中的更新管理解决方案中的 Windows 和 Linux 计算机在 Azure 中、 在本地环境或其他云提供商管理操作系统更新。 可以快速评估所有代理计算机上可用更新的状态，并管理为服务器安装所需更新的过程。

可以直接通过 Azure 自动化帐户为虚拟机启用更新管理。 若要了解如何通过自动化帐户为虚拟机启用更新管理，请参阅[管理多个虚拟机的更新](manage-update-multi.md)。 还可以通过 Azure 门户中的虚拟机页面为虚拟机启用更新管理。 此方案适用于 [Linux](../virtual-machines/linux/tutorial-monitoring.md#enable-update-management) 和 [Windows](../virtual-machines/windows/tutorial-monitoring.md#enable-update-management) 虚拟机。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="solution-overview"></a>解决方案概述

由更新管理托管的计算机使用以下配置执行评估和更新部署：

* 用于 Windows 或 Linux 的 Microsoft 监视代理 (MMA)
* 用于 Linux 的 PowerShell 所需状态配置 (DSC)
* 自动化混合 Runbook 辅助角色
* 适用于 Windows 计算机的 Microsoft 更新或 Windows Server 更新服务 (WSUS)

下图是针对行为和数据流的概念性视图，说明了解决方案如何评估安全更新并将其应用到工作区中所有连接的 Windows Server 和 Linux 计算机：

![更新管理流程](./media/automation-update-management/update-mgmt-updateworkflow.png)

更新管理可用于在同一租户的多个订阅中本机登记计算机。

发布 CVE 后，Linux 计算机需要 2-3 小时才会显示修补程序以供评估。  对于 Windows 计算机，发布后，需要 12-15 小时才会显示修补程序以供评估。

计算机完成更新符合性扫描后，代理将转发到 Azure Monitor 日志的大容量中的信息。 在 Windows 计算机上，符合性扫描默认情况下每 12 小时运行一次。

如果 MMA 重启，除了按扫描计划扫描，更新符合性扫描还会在更新安装前和更新安装后的 15 分钟内启动。

对于 Linux 计算机，符合性扫描默认情况下每 3 小时执行一次。 如果 MMA 代理重启，则会在 15 分钟内启动符合性扫描。

解决方案根据所配置的与之进行同步的源来报告计算机的更新情况。 如果将 Windows 计算机配置为向 WSUS 报告，则结果可能不同于 Microsoft 更新所显示的内容，具体取决于 WSUS 上次通过 Microsoft 更新进行同步的时间。 对于配置为向本地存储库（而非公共存储库）报告的 Linux 计算机来说，行为也是如此。

> [!NOTE]
> 若要正确地向服务进行报告，更新管理要求启用某些 URL 和端口。 若要了解有关这些要求的详细信息，请参阅[混合辅助角色的网络规划](automation-hybrid-runbook-worker.md#network-planning)。

可以通过创建计划的部署，在需要更新的计算机上部署和安装软件更新。 归类为“可选”的更新不包括在 Windows 计算机的部署范围内。 只有必需的更新会包括在部署范围内。

在计划的部署定义通过显式指定计算机或通过选择接收适用更新目标计算机[计算机组](../azure-monitor/platform/computer-groups.md)基于日志搜索的一组特定的计算机，或[Azure 查询](#azure-machines)，它动态地选择基于指定条件的 Azure Vm。 这些组是从不同[作用域配置](../azure-monitor/insights/solution-targeting.md)，仅用于确定哪些计算机获取启用此解决方案的管理包。 

也可以指定一个计划来批准并设置可以安装更新的一个时段。 这段时间称为维护时段。 如果需要重新启动，并且选择了适当的重启的选项，将为重新启动后保留十分钟的维护时段。 如果修补所花时间超过预期并且维护时段中有十分钟内，不会发生重新启动。

通过 Azure 自动化中的 runbook 安装更新。 你无法查看这些 runbook，它们不需要任何配置。 创建更新部署时，更新部署会创建一个计划，该计划在指定的时间为所包括的计算机启动主更新 Runbook。 此主 Runbook 会在每个代理上启动一个子 Runbook 来安装必需的更新。

目标计算机会按更新部署中指定的日期和时间，以并行方式执行部署。 在安装之前，会运行扫描来验证更新是否仍然是必需的。 对于 WSUS 客户端计算机，如果更新未在 WSUS 中获得批准，则更新部署会失败。

不支持具有注册多个 Log Analytics 工作区 （多宿主） 中的更新管理的计算机。

## <a name="clients"></a>客户端

### <a name="supported-client-types"></a>支持的客户端类型

下表显示了支持的操作系统列表：

|操作系统  |说明  |
|---------|---------|
|Windows Server 2008、Windows Server 2008 R2 RTM    | 仅支持更新评估。         |
|Windows Server 2008 R2 SP1 及更高版本（包括 Windows Server 2012 和 Windows Server 2016）    |需要 .NET Framework 4.5.1 或更高版本。 （[下载 .NET 框架](/dotnet/framework/install/guide-for-developers)）<br/> 需要 Windows PowerShell 4.0 或更高版本。 （[下载 WMF 4.0](https://www.microsoft.com/download/details.aspx?id=40855)）<br/> 为提高可靠性，建议使用 Windows PowerShell 5.1。  （[下载 WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616)）        |
|CentOS 6 (x86/x64) 和 7 (x64)      | Linux 代理必须具有访问更新存储库的权限。 基于分类的修补需要借助“yum”来返回 CentOS 当前没有的安全数据。 在 CentOS 上基于分类的修补的详细信息，请参阅[更新 Linux 上的分类](#linux-2)          |
|Red Hat Enterprise 6 (x86/x64) 和 7 (x64)     | Linux 代理必须具有访问更新存储库的权限。        |
|SUSE Linux Enterprise Server 11 (x86/x64) 和 12 (x64)     | Linux 代理必须具有访问更新存储库的权限。        |
|Ubuntu 14.04 LTS、16.04 LTS 和 18.04 (x86/x64)      |Linux 代理必须具有访问更新存储库的权限。         |

> [!NOTE]
> 使用更新管理可以管理 azure 虚拟机规模集。 更新管理实例本身并不在基本映像上的工作原理。 你将需要以增量方式，并同时更新所有 VM 实例计划更新。

### <a name="unsupported-client-types"></a>不支持的客户端类型

下表列出了不受支持的操作系统：

|操作系统  |说明  |
|---------|---------|
|Windows 客户端     | 不支持客户端操作系统（例如 Windows 7 和 Windows 10）。        |
|Windows Server 2016 Nano Server     | 不支持。       |

### <a name="client-requirements"></a>客户端要求

#### <a name="windows"></a>Windows

Windows 代理必须配置为与 WSUS 服务器通信或必须有权访问 Microsoft 更新。 可以将更新管理与 System Center Configuration Manager 配合使用。 若要了解有关集成方案的详细信息，请参阅[将 System Center Configuration Manager 与更新管理集成](oms-solution-updatemgmt-sccmintegration.md#configuration)。 需要 [Windows 代理](../azure-monitor/platform/agent-windows.md)。 如果加入 Azure 虚拟机，则会自动安装此代理。

#### <a name="linux"></a>Linux

对于 Linux，计算机必须能够访问更新存储库。 更新存储库可以是专用的，也可以是公共的。 需要 TLS 1.1 或 TLS 1.2 才能与更新管理进行交互。 此解决方案不支持配置为向多个 Azure Log Analytics 工作区报告的 Log Analytics Linux 代理。

有关如何安装适用于 Linux 的 Log Analytics 代理并下载最新版本的信息，请参阅[适用于 Linux 的 Log Analytics 代理](https://github.com/microsoft/oms-agent-for-linux)。 有关如何安装 Log Analytics 代理的 Windows 的信息，请参阅[将监视代理 Microsoft 为 Windows](../log-analytics/log-analytics-windows-agent.md)。

## <a name="permissions"></a>权限

若要创建和管理更新部署，需要特定的权限。 若要了解这些权限，请参阅[基于角色的访问 - 更新管理](automation-role-based-access-control.md#update-management)。

## <a name="solution-components"></a>解决方案组件

该解决方案包括以下资源。 资源将添加到你的自动化帐户。 它们是直接连接的代理，在 Operations Manager 中，则是连接的管理组。

### <a name="hybrid-worker-groups"></a>混合辅助角色组

启用此解决方案以后，任何直接连接到 Log Analytics 工作区的 Windows 计算机都会自动配置为混合 Runbook 辅助角色，为此解决方案中包括的 Runbook 提供支持。

此解决方案管理的每台 Windows 计算机都会作为自动化帐户的一个“系统混合辅助角色组”列在“混合辅助角色组”窗格中。 解决方案使用命名约定 *Hostname FQDN_GUID*。 不能在帐户中通过 Runbook 将这些组作为目标进行操作。 如果尝试使用它们，它们将失败。 这些组仅用于为管理解决方案提供支持。

如果为解决方案和混合 Runbook 辅助角色组成员身份使用同一帐户，则可以将 Windows 计算机添加到自动化帐户中的混合 Runbook 辅助角色组来为自动化 Runbook 提供支持。 此功能是在 7.2.12024.0 版本的混合 Runbook 辅助角色中添加的。

### <a name="management-packs"></a>管理包

如果 System Center Operations Manager 管理组已连接到 Log Analytics 工作区，则会在 Operations Manager 中安装以下管理包。 在添加此解决方案后，还会在直接连接的 Windows 计算机上安装这些管理包。 你不需要对这些管理包进行配置或管理。

* Microsoft System Center Advisor Update Assessment Intelligence Pack (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* 更新部署 MP

> [!NOTE]
> 如果您在管理组级别配置将与工作区相关联的代理与 Operations Manager 1807 管理组，若要获取它们才会显示当前的解决方法是重写**IsAutoRegistrationEnabled**到 **，则返回 true**中**Microsoft.IntelligencePacks.AzureAutomation.HybridAgent.Init**规则。

有关如何更新解决方案管理包的详细信息，请参阅[将 Operations Manager 连接到 Azure Monitor 日志](../azure-monitor/platform/om-agents.md)。

> [!NOTE]
> 对于使用 Operations Manger 代理的系统，若要能够由更新管理完全托管，需要将代理更新为 Microsoft Monitoring Agent。 若要了解如何更新代理，请参阅[如何升级 Operations Manager 代理](https://docs.microsoft.com/system-center/scom/deploy-upgrade-agents)。 使用 Operations Manager 环境，需要它的 14 或更高版本运行 System Center Operations Manager 2012 R2 UR。

## <a name="onboard"></a>启用更新管理

要开始修补系统，需要启用更新管理解决方案。 可使用多种方法将计算机加入到更新管理。 以下是推荐和支持的方法，可用于加入此解决方案：

* [通过虚拟机](automation-onboard-solutions-from-vm.md)
* [通过浏览多个计算机](automation-onboard-solutions-from-browse.md)
* [通过自动化帐户](automation-onboard-solutions-from-automation-account.md)
* [使用 Azure 自动化 runbook](automation-onboard-solutions.md)
  
### <a name="confirm-that-non-azure-machines-are-onboarded"></a>确认非 Azure 计算机已加入

若要确认直接连接的计算机进行通信的 Azure Monitor 日志，几分钟后，你可以运行一个以下日志搜索。

#### <a name="linux"></a>Linux

```loganalytics
Heartbeat
| where OSType == "Linux" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

#### <a name="windows"></a>Windows

```loganalytics
Heartbeat
| where OSType == "Windows" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

在 Windows 计算机上，可以查看以下信息来验证代理与 Azure Monitor 日志连接：

1. 在控制面板中，打开 **Microsoft Monitoring Agent**。 在“Azure Log Analytics”选项卡上，代理会显示以下消息：“Microsoft Monitoring Agent 已成功连接到 Log Analytics。”
2. 打开“Windows 事件日志”。 转到“应用程序和服务日志\Operations Manager”，搜索来自“服务连接器”源的事件 ID 3000 和事件 ID 5002。 这些事件指示计算机已注册到 Log Analytics 工作区并且正在接收配置。

如果代理无法与通信 Azure Monitor 日志和代理配置为通过防火墙或代理服务器与 internet 进行通信，请确认防火墙或代理服务器已正确配置。 若要了解如何验证防火墙或代理服务器是否已正确配置，请参阅 [Windows 代理的网络配置](../azure-monitor/platform/agent-windows.md)或 [Linux 代理的网络配置](../log-analytics/log-analytics-agent-linux.md)。

> [!NOTE]
> 如果 Linux 系统配置为与代理或 Log Analytics 网关通信，并且你将载入此解决方案，请使用以下命令更新 proxy.conf 权限来向 omiuser 组授予对文件的读取权限：
>
> `sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf`
> `sudo chmod 644 /etc/opt/microsoft/omsagent/proxy.conf`

执行评估后，新添加的 Linux 代理会显示状态“已更新”。 此过程可能需要长达 6 小时的时间。

若要确认 Operations Manager 管理组是否正在与 Azure Monitor 日志通信，请参阅[验证 Operations Manager 与 Azure Monitor 日志集成](../azure-monitor/platform/om-agents.md#validate-operations-manager-integration-with-azure-monitor)。

## <a name="data-collection"></a>数据收集

### <a name="supported-agents"></a>支持的代理

下表介绍了该解决方案支持的连接的源：

| 连接的源 | 支持 | 描述 |
| --- | --- | --- |
| Windows 代理 |“是” |该解决方案从 Windows 代理收集有关系统更新的信息，然后开始安装必需的更新。 |
| Linux 代理 |“是” |该解决方案从 Linux 代理收集有关系统更新的信息，然后开始在受支持的发行版上安装必需的更新。 |
| Operations Manager 管理组 |“是” |该解决方案从已连接的管理组中的代理收集有关系统更新的信息。<br/>从 Operations Manager 代理直接连接到 Azure Monitor 日志不是必需的。 数据将从管理组转发到 Log Analytics 工作区。 |

### <a name="collection-frequency"></a>收集频率

对于每台托管的 Windows 计算机，每天执行两次扫描。 每隔 15 分钟就会调用一次 Windows API 来查询上次更新时间，以确定状态是否已更改。 如果状态已更改，则会启动符合性扫描。

对于每台托管的 Linux 计算机，每 3 小时执行一次扫描。

可能需要 30 分钟到 6 小时，仪表板才会显示受托管计算机提供的已更新数据。

使用更新管理的计算机的 Azure Monitor 日志数据使用情况的平均值为每月大约 25 MB。 此值仅为近似值，且随时可能基于环境而更改。 建议监视环境以查看具体使用情况。

## <a name="viewing-update-assessments"></a>查看更新评估

在自动化帐户中，选择“更新管理”来查看计算机的状态。

此视图提供有关计算机、缺少的更新、更新部署和计划的更新部署的信息。 在“符合性”列中，可以看到计算机的最后评估时间。 在“更新代理准备”列中，可以看到更新代理的运行状况。 如果有问题，请选择相应链接来转到疑难解答文档，以帮助你了解可以采用什么步骤来更正问题。

若要运行日志搜索来返回有关计算机、更新或部署的信息，请在列表中选择相应的项。 此时将打开“日志搜索”窗格，其中显示了针对所选项的查询。

![更新管理默认视图](media/automation-update-management/update-management-view.png)

## <a name="install-updates"></a>安装更新

对工作区中的所有 Linux 和 Windows 计算机进行更新评估后，可以通过创建“更新部署”安装必需的更新。 若要创建更新部署，必须具有到自动化帐户的写访问权限和写入访问权限，并针对部署中的任何 Azure Vm。 更新部署是为一台或多台计算机计划的必需更新安装。 应当指定部署日期和时间，以及要在部署范围中包括的计算机或计算机组。 若要了解有关计算机组的详细信息，请参阅[Azure Monitor 日志中的计算机组](../azure-monitor/platform/computer-groups.md)。

在更新部署中包括计算机组时，只会在创建计划时对组成员身份评估一次。 不会反映对组所做的后续更改。 若要解决这种用法[动态组](#using-dynamic-groups)，这些组在部署时解析和为 Azure Vm 或非 Azure Vm 的已保存的搜索查询定义。

> [!NOTE]
> 默认情况下，从 Azure 市场部署的 Windows 虚拟机设置为从 Windows 更新服务接收自动更新。 添加此解决方案或者将 Windows 虚拟机添加到工作区时，此行为不会更改。 如果不主动通过此解决方案管理更新，则会应用默认行为（即自动应用更新）。

若要避免在 Ubuntu 上的维护时段外应用更新，请重新配置无人参与升级包，禁用自动更新。 有关如何配置此包的信息，请参阅 [Ubuntu Server 指南中的自动更新主题](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)。

基于 Azure 市场中提供的按需 Red Hat Enterprise Linux (RHEL) 映像创建的虚拟机注册为访问 Azure 中部署的 [Red Hat 更新基础结构 (RHUI)](../virtual-machines/virtual-machines-linux-update-infrastructure-redhat.md)。 对于任何其他 Linux 发行版，必须按照其支持的方法从发行版联机文件存储库对其进行更新。

若要创建新的更新部署，请选择“计划更新部署”。 **新的更新部署**页随即打开。 为下表中介绍的属性输入值，然后单击“创建”：

| 属性 | 描述 |
| --- | --- |
| 名称 |用于标识更新部署的唯一名称。 |
|操作系统| Linux 或 Windows|
| 若要更新的组 |适用于 Azure 机定义的订阅、 资源组、 位置和标记来生成要在部署中包含的 Azure Vm 的动态组组合所基于的查询。 </br></br>对于非 Azure 计算机，选择现有的已保存搜索以选择要包括在部署中的非 Azure 计算机组。 </br></br>有关详细信息，请参阅[动态组](automation-update-management.md#using-dynamic-groups)|
| 要更新的计算机 |选择已保存的搜索、已导入的组或者从下拉列表中选择“计算机”并选择单个计算机。 如果选择“计算机”，则计算机的就绪状态将在“更新代理商准备情况”列中显示。</br> 要了解在 Azure Monitor 日志中创建计算机组的不同方法，请参阅 [Azure Monitor 日志中的计算机组](../azure-monitor/platform/computer-groups.md) |
|更新分类|选择所需的所有更新分类|
|包括/排除更新|这将打开“包括/排除”页。 要包含或排除的更新位于单独的选项卡上。 有关如何处理包含的详细信息，请参阅[包含行为](automation-update-management.md#inclusion-behavior) |
|计划设置|选择启动时间，然后选择任“一次”或“重复”|
| 前脚本 + 后脚本|选择要在部署前和部署后运行的脚本|
| 维护时段 |为更新设置的分钟数。 该值不能小于 30 分钟，且不能大于 6 小时 |
| 重新启动控制| 确定应如何处理重新启动。 可用选项包括：</br>需要时重新启动(默认)</br>始终重新启动</br>从不重新启动</br>仅重启 - 不安装更新|

此外，能够以编程方式创建更新部署。 若要了解如何使用 REST API 创建更新部署，请参阅[软件更新配置 - 创建](/rest/api/automation/softwareupdateconfigurations/create)。 此外，还有一个示例 Runbook，可用于创建每周更新部署。 若要了解有关此 Runbook 的详细信息，请参阅[为资源组中的一个或多个 VM 创建每周更新部署](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1)。

### <a name="multi-tenant"></a>跨租户更新部署

如果你在向“更新管理”进行报告的另一个 Azure 租户中存在需要修补的计算机，则需要使用以下解决方法来计划它们。 你可以使用 [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) cmdlet 和开关 `-ForUpdate` 来创建计划，然后使用 [New-AzureRmAutomationSoftwareUpdateConfiguration](/powershell/module/azurerm.automation/new-azurermautomationsoftwareupdateconfiguration
) cmdlet 并将另一个租户中的计算机传递给 `-NonAzureComputer` 参数。 以下示例展示了如何执行此操作：

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$sched = New-AzureRmAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdate

New-AzureRmAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName <automationAccountName> -Schedule $sched -Windows -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

## <a name="view-missing-updates"></a>查看缺少的更新

选择“缺少的更新”可查看计算机中缺少的更新列表。 每个更新都会列出，并且可供选择。 将显示有关需要更新的计算机数和操作系统的信息，以及指向更多信息的链接。 “日志搜索”窗格显示有关更新的更多详细信息。

## <a name="view-update-deployments"></a>查看更新部署

选择“更新部署”选项卡可查看现有更新部署的列表。 选择表中的任一更新部署会打开该更新部署的“更新部署运行”窗格。

![更新部署结果的概述](./media/automation-update-management/update-deployment-run.png)

若要从 REST API 查看更新部署，请参阅[软件更新配置运行](/rest/api/automation/softwareupdateconfigurationruns)。

## <a name="update-classifications"></a>更新分类

下表列出了更新管理中的更新分类，以及每个分类的定义。

### <a name="windows"></a>Windows

|分类  |描述  |
|---------|---------|
|关键更新     | 解决关键、非安全相关错误的特定问题的更新。        |
|安全更新     | 产品特定、安全相关问题的更新。        |
|更新汇总     | 一起打包以便于部署的一组累积修补程序。        |
|功能包     | 在产品版本以外发布的新产品功能。        |
|Service Pack     | 应用于应用程序的一组累积修补程序。        |
|定义更新     | 对病毒或其他定义文件的更新。        |
|工具     | 可帮助完成一个或多个任务的实用工具或功能。        |
|更新     | 对当前已安装的应用程序或文件的更新。        |

### <a name="linux"></a>Linux

|分类  |描述  |
|---------|---------|
|关键和安全更新     | 特定问题或产品特定、安全相关问题的更新。         |
|其他更新     | 本质上不是关键更新或不是安全更新的所有其他更新。        |

对于 Linux，由于云中的数据扩充，更新管理可以区分云中的关键更新和安全更新，同时显示评估数据。 为了进行修补，更新管理依赖于计算机上提供的分类数据。 与其他发行版不同，CentOS 没有现成的此信息。 如果你已将 CentOS 计算机配置为以某种方式返回以下命令的安全数据，则更新管理将能够基于分类进行修补。

```bash
sudo yum -q --security check-update
```

当前没有受支持的方法可用来在 CentOS 上提供原生分类数据。 目前，只能尽力为可能自己实现了此功能的客户提供支持。

## <a name="firstparty-predownload"></a>高级的设置

更新管理依赖于 Windows 更新来下载和安装 Windows 更新。 因此，我们会遵循 Windows 更新使用的许多设置。 如果使用设置来启用非 Windows 更新，更新管理也将管理这些更新。 如果你想要在更新部署发生之前启用下载更新，更新部署会更快且不太可能会超过维护时段。

### <a name="pre-download-updates"></a>预下载更新

若要在组策略中自动配置下载更新，可以将“[配置自动更新设置](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates#BKMK_comp5)”设置为 **3**。 将在后台下载所需的更新，但不会安装它们。 这让更新管理可以始终控制计划，但允许在更新管理维护时段外下载更新。 这可阻止更新管理中出现“已超过维护时段”错误。

此外，还可以使用 PowerShell 进行相关设置，在要自动下载更新的系统上运行 PowerShell。

```powershell
$WUSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
$WUSettings.NotificationLevel = 3
$WUSettings.Save()
```

### <a name="disable-automatic-installation"></a>禁用自动安装

Azure 虚拟机具有更新默认情况下启用的自动安装。 这会导致在安装之前它们要安装的更新管理计划的更新。 可以通过设置禁用此行为`NoAutoUpdate`注册表项`1`。 以下 PowerShell 代码片段演示了一种方法执行此操作。

```powershell
$AutoUpdatePath = "HKLM:SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
Set-ItemProperty -Path $AutoUpdatePath -Name NoAutoUpdate -Value 1
```

### <a name="enable-updates-for-other-microsoft-products"></a>启用其他 Microsoft 产品的更新

默认情况下，Windows 更新仅为 Windows 提供更新。 如果启用**让我的其他 Microsoft 产品更新时我更新 Windows**，你要更新提供的其他产品，包括为 SQL Server 或其他第一方软件的安全修补程序。 不能通过组策略配置此选项。 在你想要启用其他第一方修补程序的系统上运行以下 PowerShell，更新管理将支持此设置。

```powershell
$ServiceManager = (New-Object -com "Microsoft.Update.ServiceManager")
$ServiceManager.Services
$ServiceID = "7971f918-a847-4430-9279-4a52d1efe18d"
$ServiceManager.AddService2($ServiceId,7,"")
```

## <a name="third-party"></a> Windows 上的第三方修补程序

更新管理依赖于要修补程序支持的 Windows 系统的本地配置的更新存储库。 这是 WSUS 或 Windows 更新。 可以使用 [System Center Updates Publisher](/sccm/sum/tools/updates-publisher
) (Updates Publisher) 之类的工具将自定义更新发布到 WSUS 中。 这种情况下允许使用 System Center Configuration Manager 作为其更新存储库与第三方软件的修补程序计算机更新管理。 若要了解如何配置 Updates Publisher，请参阅[安装 Updates Publisher](/sccm/sum/tools/install-updates-publisher)。

## <a name="ports"></a>网络规划

更新管理特别需要以下地址。 与以下地址的通信通过端口 443 进行。

|Azure Public  |Azure 政府  |
|---------|---------|
|*.ods.opinsights.azure.com     |*.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|* .blob.core.windows.net|*.blob.core.usgovcloudapi.net|
|* .azure-automation.net|*.azure-automation.us|

有关混合 Runbook 辅助角色所需端口的详细信息，请参阅[混合辅助角色端口](automation-hybrid-runbook-worker.md#hybrid-worker-role)。

建议在定义异常时使用列出的地址。 对于 IP 地址，可以下载 [Microsoft Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。 此文件每周更新一次，反映当前已部署的范围和任何即将对 IP 范围进行的更改。

## <a name="search-logs"></a>搜索日志

除了 Azure 门户中提供的详细信息以外，还可以针对日志执行搜索。 在解决方案页上，选择**Log Analytics**。 此时将打开**日志搜索**窗格。

还可访问 Log Analytics 搜索 API 文档，[了解如何自定义查询或从不同客户端使用查询等。](
https://dev.loganalytics.io/).

### <a name="sample-queries"></a>示例查询

以下各部分提供了此解决方案收集的更新记录的示例日志查询：

#### <a name="single-azure-vm-assessment-queries-windows"></a>单个 Azure VM 评估查询 (Windows)

将 VMUUID 值替换为要查询的虚拟机的 VM GUID。 您可以找到 VMUUID 应使用通过 Azure Monitor 日志中运行以下查询： `Update | where Computer == "<machine name>" | summarize by Computer, VMUUID`

##### <a name="missing-updates-summary"></a>缺少更新摘要

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"b08d5afa-1471-4b52-bd95-a44fea6e4ca8"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security"
```

##### <a name="missing-updates-list"></a>缺少更新列表

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"8bf1ccc6-b6d3-4a0b-a643-23f346dfdf82"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

#### <a name="single-azure-vm-assessment-queries-linux"></a>单个 Azure VM 评估查询 (Linux)

对于某些 Linux 发行版中，没有[字节顺序](https://en.wikipedia.org/wiki/Endianness)与来自 Azure 资源管理器和 Azure Monitor 日志中存储的内容的 VMUUID 值不匹配。 以下查询可检查任一字节序的匹配情况。 使用 GUID 的 big-endian 和 little-endian 格式替换 VMUUID 值可正常地返回结果。 您可以找到 VMUUID 应使用通过 Azure Monitor 日志中运行以下查询： `Update | where Computer == "<machine name>"
| summarize by Computer, VMUUID`

##### <a name="missing-updates-summary"></a>缺少更新摘要

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

##### <a name="missing-updates-list"></a>缺少更新列表

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl

```

#### <a name="multi-vm-assessment-queries"></a>多 VM 评估查询

##### <a name="computers-summary"></a>计算机摘要

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Approved, Optional, Classification) by SourceComputerId, UpdateID
    | distinct SourceComputerId, Classification, UpdateState, Approved, Optional
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed" and (Optional==false or Classification has "Critical" or Classification has "Security") and Approved!=false, iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity
| union (Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by SourceComputerId, Product, ProductArch
    | distinct SourceComputerId, Classification, UpdateState
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed", iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity)
| summarize assessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity>-1), notAssessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==-1), computersNeedCriticalUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==4), computersNeedSecurityUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==2), computersNeedOtherUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==1), upToDateComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==0)
| summarize assessedComputersCount=sum(assessedComputersCount), computersNeedCriticalUpdatesCount=sum(computersNeedCriticalUpdatesCount),  computersNeedSecurityUpdatesCount=sum(computersNeedSecurityUpdatesCount), computersNeedOtherUpdatesCount=sum(computersNeedOtherUpdatesCount), upToDateComputersCount=sum(upToDateComputersCount), notAssessedComputersCount=sum(notAssessedComputersCount)
| extend allComputersCount=assessedComputersCount+notAssessedComputersCount


```

##### <a name="missing-updates-summary"></a>缺少更新摘要

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| union (Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification )
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security"
```

##### <a name="computers-list"></a>计算机列表

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Product, Computer, ComputerEnvironment) by SourceComputerId, Product, ProductArch
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed"), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed"), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed"), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2)
| union(Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, Optional, Approved, Computer, ComputerEnvironment) by Computer, SourceComputerId, UpdateID
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed" and Approved!=false), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed" and Approved!=false), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed" and Optional==false and Approved!=false), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2) )
| order by ComplianceOrder asc, missingCriticalUpdatesCount desc, missingSecurityUpdatesCount desc, missingOtherUpdatesCount desc, displayName asc
| project-away ComplianceOrder
```

##### <a name="missing-updates-list"></a>缺少更新列表

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| union(Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2)
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

## <a name="using-dynamic-groups"></a>使用动态组

更新管理提供面向更新部署的 Azure 或非 Azure Vm 的动态组的功能。 因此，不需要编辑你要添加的计算机的部署，会在部署时评估这些组。

> [!NOTE]
> 创建更新部署时，您必须具有适当权限。 若要了解详细信息，请参阅[安装更新](#install-updates)。

### <a name="azure-machines"></a>Azure 计算机

这些组由查询进行定义，当更新部署开始时，将评估该组的成员。 动态组不适用于经典 Vm。 定义查询时，结合使用以下各项填充动态组

* 订阅
* 资源组
* 位置
* 标记

![选择组](./media/automation-update-management/select-groups.png)

若要预览动态组的结果，请单击“预览”按钮。 那时此预览将显示组成员身份，在此示例中，我们要搜索标记“Role”等于“BackendServer”的计算机。 如果多台计算机已添加此标记，会针对该组将它们添加到任何未来部署中。

![预览组](./media/automation-update-management/preview-groups.png)

### <a name="non-azure-machines"></a>非 Azure 计算机

适用于非 Azure 计算机，已保存的搜索也称为计算机组用于创建动态组。 若要了解如何创建已保存的搜索，请参阅[创建计算机组](../azure-monitor/platform/computer-groups.md#creating-a-computer-group)。 创建你的组后你可以从保存的搜索结果列表中选择它。 单击**预览版**预览此时已保存的搜索中的计算机。

![选择组](./media/automation-update-management/select-groups-2.png)

## <a name="integrate-with-system-center-configuration-manager"></a>集成 System Center Configuration Manager

已经投资购买了 System Center Configuration Manager 来管理电脑、服务器和移动设备的客户还依赖 Configuration Manager 的优势和成熟度来帮助他们管理软件更新。 Configuration Manager 是其软件更新管理 (SUM) 周期的一部分。

若要了解如何将管理解决方案与 System Center Configuration Manager 集成，请参阅[将 System Center Configuration Manager 与更新管理集成](oms-solution-updatemgmt-sccmintegration.md)。

## <a name="inclusion-behavior"></a>包含行为

更新包含可用于指定要应用的特定更新。 安装已包含的修补程序或包。 如果包含修补程序或包且选中了一个分类，将同时安装包含的项和满足分类的项。

请务必注意，排除项会替代包含项。 例如，如果定义 `*` 的排除规则，全部排除后将不会安装任何修补程序或包。 已排除的修补程序仍显示为在计算机中缺少。 对于 Linux 计算机，如果包含包且已排除相关包，将不会安装此包。

## <a name="patch-linux-machines"></a>修补 Linux 计算机

下列部分说明了 Linux 修补的潜在问题。

### <a name="unexpected-os-level-upgrades"></a>意外的 OS 级别升级

在某些 Linux 版本（例如 Red Hat Enterprise Linux）中，可以通过包执行 OS 级别的升级。 这可能会导致运行更新管理并更改 OS 版本号。 由于更新管理使用相同的方法来更新管理员将在 Linux 计算机本地使用的包，因此，此行为是有意实施的。

若要避免通过更新管理运行来更新 OS 版本，可以使用“排除”功能。

在 Red Hat Enterprise Linux 中，要排除的包名称为 redhat-release-server.x86_64。

![适用于 Linux 的要排除的包](./media/automation-update-management/linuxpatches.png)

### <a name="critical--security-patches-arent-applied"></a>不会应用关键 / 安全修补程序

将更新部署到 Linux 计算机时，可以选择更新分类。 这会筛选已应用到满足指定条件的计算机的更新。 部署更新时，会在计算机本地应用此筛选器。

由于更新管理在云中执行更新扩充，因此某些更新可以标记更新管理中为具有安全影响，即使在本地计算机不具有该信息。 如果向 Linux 计算机应用关键更新，可能有些更新不会标记为对该计算机具有安全影响，因此不会应用这些更新。

但是，更新管理仍可能报告该计算机不合规，因为其中包含有关更新的其他信息。

按更新分类部署更新在 CentOS 上无法现成地运行。 若要为 CentOS 正确部署更新，请选择所有分类以确保应用更新。 对于 SUSE，如果首先需要与 zypper（包管理器）或其依赖项相关的安全更新，则仅选择“其他更新”作为分类可能会导致同时安装某些安全更新。 此行为是 zypper 的一项限制。 某些情况下，可能需要重新运行更新部署。 若要验证，请检查更新日志。

## <a name="remove-a-vm-from-update-management"></a>从更新管理中删除 VM

从更新管理中删除 VM：

* 在 Log Analytics 工作区中，从范围配置 `MicrosoftDefaultScopeConfig-Updates` 的已保存的搜索中删除 VM。 已保存的搜索位于工作区的“常规”下。
* 删除 [Microsoft Monitoring Agent](../azure-monitor/learn/quick-collect-windows-computer.md#clean-up-resources) 或 [适用于 Linux 的 Log Analytics 代理](../azure-monitor/learn/quick-collect-linux-computer.md#clean-up-resources)。

## <a name="next-steps"></a>后续步骤

继续学习教程来了解如何管理 Windows 虚拟机的更新。

> [!div class="nextstepaction"]
> [管理 Azure Windows VM 的更新和修补程序](automation-tutorial-update-management.md)

* 使用中的日志搜索[Azure Monitor 日志](../log-analytics/log-analytics-log-searches.md)若要查看详细的更新数据。
* 针对更新部署状态[创建警报](automation-tutorial-update-management.md#configure-alerts)。

* 若要了解如何通过 REST API 与更新部署交互，请参阅[软件更新配置](/rest/api/automation/softwareupdateconfigurations)
* 要了解如何对更新管理进行故障排除，请参阅[更新管理故障排除](troubleshoot/update-management.md)
