---
title: Azure 自动化混合 Runbook 辅助角色
description: 本文介绍如何安装和使用混合 Runbook 辅助角色，该角色是 Azure 自动化的一项功能，可用于在本地数据中心或云提供商的计算机上运行 Runbook。
services: automation
ms.service: automation
ms.subservice: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 04/05/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 785cf5159615b4a81740e853f2b513f0e6d74aec
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198459"
---
# <a name="automate-resources-in-your-datacenter-or-cloud-by-using-hybrid-runbook-worker"></a>使用混合 Runbook 辅助角色使数据中心或云端的资源实现自动化

Azure 自动化中的 Runbook 可能无权访问其他云或本地环境中的资源，因为它们在 Azure 云平台中运行。 利用 Azure 自动化的混合 Runbook 辅助角色功能，既可以直接在托管角色的计算机上运行 Runbook，也可以对环境中的资源运行 Runbook，从而管理这些本地资源。 Runbook 在 Azure 自动化中进行存储和管理，然后发送到一台或多台指定的计算机。

下图说明了此功能：

![混合 Runbook 辅助角色概述](media/automation-hybrid-runbook-worker/automation.png)

每个混合 Runbook 辅助角色都是你在安装代理时指定的混合 Runbook 辅助角色组的成员。 一个组可以包含一个代理，但是可以在一个组中安装多个代理，以实现高可用性。

在混合 Runbook 辅助角色中启动 Runbook 时，可以指定该辅助角色会在其中运行的组。 组中的每个辅助角色都会轮询 Azure 自动化以查看是否有可用作业。 如果作业可用，获取作业的第一个辅助角色将执行该作业。 作业队列的处理时间取决于混合辅助角色硬件配置文件和负载。 不能指定特定的辅助角色。 混合 Runbook 辅助角色不受 Azure 沙盒所具有的诸多限制。 它们没有磁盘空间、内存或网络套接字方面的相同限制。 混合 Runbook 辅助角色仅受混合 Runbook 辅助角色本身所拥有的资源的限制。 此外，混合 Runbook 辅助角色不受 Azure 沙盒所具有的 180 分钟[公平共享](automation-runbook-execution.md#fair-share)时间限制。 若要了解有关 Azure 沙盒和混合 Runbook 辅助角色的服务限制的详细信息，请参阅作业[限制](../azure-subscription-service-limits.md#automation-limits)页。

## <a name="install-a-hybrid-runbook-worker"></a>安装混合 Runbook 辅助角色

安装混合 Runbook 辅助角色的过程取决于 OS。 下表包含指向可用于安装的方法的链接。

若要安装和配置 Windows 混合 Runbook 辅助角色，可使用两种方法。 建议的方法是使用自动化 Runbook 来彻底实现配置 Windows 计算机过程的自动化。 第二种方法使用分步过程来手动安装和配置角色。 对于 Linux 计算机，运行 Python 脚本，在计算机上安装代理。

|操作系统  |部署类型  |
|---------|---------|
|Windows     | [PowerShell](automation-windows-hrw-install.md#automated-deployment)<br>[手动](automation-windows-hrw-install.md#manual-deployment)        |
|Linux     | [Python](automation-linux-hrw-install.md#installing-a-linux-hybrid-runbook-worker)        |

> [!NOTE]
> 为了使用所需状态配置 (DSC) 管理支持混合 Runbook 辅助角色的服务器配置，需将其添加为 DSC 节点。 若要进一步了解如何载入它们以供 DSC 管理，请参阅[载入由 Azure 自动化 DSC 管理的计算机](automation-dsc-onboarding.md)。
>
>如果启用[更新管理解决方案](automation-update-management.md)，任何连接到 Azure Log Analytics 工作区的计算机将自动配置为混合 Runbook 辅助角色，以支持此解决方案中包括的 Runbook。 但是，该计算机未注册到任何已在自动化帐户中定义的混合辅助角色组。 只要将同一个帐户同时用于解决方案和混合 Runbook 辅助角色组成员身份，即可将该计算机添加到自动化帐户的混合 Runbook 辅助角色组，以支持自动化 Runbook。 此功能已添加到 7.2.12024.0 版本的混合 Runbook 辅助角色。

开始部署混合 Runbook 辅助角色之前，请查看[有关规划网络的信息](#network-planning)。 成功部署辅助角色后，请查看[在混合 Runbook 辅助角色上运行 Runbook](automation-hrw-run-runbooks.md)，了解如何配置 Runbook，使本地数据中心或其他云环境中的过程实现自动化。

## <a name="remove-a-hybrid-runbook-worker"></a>删除混合 Runbook 辅助角色

可以从组中删除一个或多个混合 Runbook 辅助角色，或者根据要求删除该组。 若要从本地计算机中删除混合 Runbook 辅助角色，请使用以下步骤：

1. 在 Azure 门户中，转到自动化帐户。
2. 在“帐户设置”下，选择“密钥”并记下“URL”和“主访问密钥”的值。 下一步需要用到此信息。

### <a name="windows"></a>Windows

在管理员模式下打开 PowerShell 会话，并运行以下命令。 可使用 **-Verbose** 开关获取删除过程的详细日志。

```powershell-interactive
Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey>
```

若要从混合辅助角色组中删除过时的计算机，请使用可选的 `machineName` 参数。

```powershell-interactive
Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey> -machineName <ComputerName>
```

### <a name="linux"></a>Linux

可在混合 Runbook 辅助角色上使用命令 `ls /var/opt/microsoft/omsagent` 获取工作区 ID。 目录中有一个名为工作区 ID 的文件夹。

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> 此代码不会从计算机中删除 Microsoft Monitoring Agent，而只会删除混合 Runbook 辅助角色的功能和配置。

## <a name="remove-a-hybrid-worker-group"></a>删除混合辅助角色组

要删除某个组，首先需要使用前面所示的过程，从每台计算机中删除属于该组的混合 Runbook 辅助角色。 然后，使用以下步骤删除该组：

1. 在 Azure 门户中打开自动化帐户。
2. 在“流程自动化”下选择“混合辅助角色组”。 选择要删除的组。 将显示该组的属性页。

   ![“属性”页](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)

3. 在所选组的属性页中，选择“删除”。 系统会显示一条消息，要求确认此操作。 如果确定要继续，请选择“是”。

   ![确认消息](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)

   完成此过程可能需要数秒钟的时间。 可以在菜单中的“通知”下面跟踪操作进度。

## <a name="network-planning"></a>配置网络

### <a name="hybrid-worker-role"></a>混合辅助角色

若要连接到并将注册到 Azure 自动化混合 Runbook 辅助角色，它必须有权访问的端口号和此部分所述的 Url。 此访问权限是到顶部[端口和 Url 所需的 Microsoft Monitoring Agent](../azure-monitor/platform/agent-windows.md)以连接到 Azure Monitor 日志。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

如果代理与 Azure 自动化服务之间的通信使用代理服务器，请确保相应的资源是否可访问。 混合 Runbook 辅助角色和自动化服务中请求的超时时间为 30 秒。 在 3 次尝试后，请求将失败。 如果使用防火墙来限制对 Internet 的访问，则必须将防火墙配置为允许访问。 如果将 Log Analytics 网关用作代理，请确保为混合辅助角色配置 Log Analytics 网关。 有关如何执行此操作的说明，请参阅[为自动化混合辅助角色配置 Log Analytics 网关](https://docs.microsoft.com/azure/log-analytics/log-analytics-oms-gateway)。

混合 Runbook 辅助角色与自动化通信时需要以下端口和 URL：

* 端口：只需使用 TCP 443 即可进行出站 Internet 访问。
* 全局 URL：*.azure-automation.net
* 美国弗吉尼亚州政府的全局 URL：*.azure-automation.us
* 代理服务： https://\<workspaceId\>.agentsvc.azure-automation.net

建议在定义例外时使用列出的地址。 对于 IP 地址，可以下载 [Microsoft Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。 此文件每周更新，包含当前部署的范围以及即将对 IP 范围进行的更新。

如果为特定的区域定义了自动化帐户，则可以限制与该区域数据中心之间的通信。 下表提供了每个区域的 DNS 记录：

| **区域** | **DNS 记录** |
| --- | --- |
| 美国中西部 | wcus-jobruntimedata-prod-su1.azure-automation.net</br>wcus-agentservice-prod-1.azure-automation.net |
| 美国中南部 |scus-jobruntimedata-prod-su1.azure-automation.net</br>scus-agentservice-prod-1.azure-automation.net |
| 美国东部 2 |eus2-jobruntimedata-prod-su1.azure-automation.net</br>eus2-agentservice-prod-1.azure-automation.net |
| 美国西部 2 |wus2-jobruntimedata-prod-su1.azure-automation.net</br>wus2-agentservice-prod-1.azure-automation.net |
| 加拿大中部 |cc-jobruntimedata-prod-su1.azure-automation.net</br>cc-agentservice-prod-1.azure-automation.net |
| 西欧 |we-jobruntimedata-prod-su1.azure-automation.net</br>we-agentservice-prod-1.azure-automation.net |
| 北欧 |ne-jobruntimedata-prod-su1.azure-automation.net</br>ne-agentservice-prod-1.azure-automation.net |
| 东南亚 |sea-jobruntimedata-prod-su1.azure-automation.net</br>sea-agentservice-prod-1.azure-automation.net|
| 印度中部 |cid-jobruntimedata-prod-su1.azure-automation.net</br>cid-agentservice-prod-1.azure-automation.net |
| 日本东部 |jpe-jobruntimedata-prod-su1.azure-automation.net</br>jpe-agentservice-prod-1.azure-automation.net |
| 澳大利亚东部 |ae-jobruntimedata-prod-su1.azure-automation.net</br>ae-agentservice-prod-1.azure-automation.net |
| 澳大利亚东南部 |ase-jobruntimedata-prod-su1.azure-automation.net</br>ase-agentservice-prod-1.azure-automation.net |
| 英国南部 | uks-jobruntimedata-prod-su1.azure-automation.net</br>uks-agentservice-prod-1.azure-automation.net |
| 美国政府弗吉尼亚州 | usge-jobruntimedata-prod-su1.azure-automation.us<br>usge-agentservice-prod-1.azure-automation.us |

有关区域 IP 地址列表（非区域名称列表），请从 Microsoft 下载中心下载 [Azure 数据中心 IP 地址](https://www.microsoft.com/download/details.aspx?id=41653) XML 文件。

> [!NOTE]
> Azure 数据中心 IP 地址 XML 文件列出了 Microsoft Azure 数据中心使用的 IP 地址范围。 文件中包含计算、SQL 和存储范围。
>
>每周都将发布更新的文件。 该文件反映当前已部署的范围和任何即将对 IP 范围进行的更改。 数据中心至少在一周后才会使用文件中显示的新范围。
>
> 建议每周下载新的 XML 文件。 然后，更新网站以正确地标识 Azure 中运行的服务。 Azure ExpressRoute 用户应注意，此文件过去经常在每个月的第一周更新 Azure 空间的边界网关协议 (BGP) 播发。

### <a name="update-management"></a>更新管理

除了混合 Runbook 辅助角色所需的标准地址和端口以外，更新管理还特别需要以下地址。 与这些地址的通信通过端口 443 完成。

|Azure Public  |Azure Government   |
|---------|---------|
|*.ods.opinsights.azure.com     |*.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|* .blob.core.windows.net|*.blob.core.usgovcloudapi.net|

## <a name="next-steps"></a>后续步骤

* 若要了解如何配置 Runbook，使本地数据中心或其他云环境中的过程自动化，请参阅[在混合 Runbook 辅助角色上运行 Runbook](automation-hrw-run-runbooks.md)。
* 若要了解如何对混合 Runbook 辅助角色进行故障排除，请参阅[混合 Runbook 辅助角色的故障排除](troubleshoot/hybrid-runbook-worker.md#general)

