---
title: 排查加入更新管理、更改跟踪和库存时发生的错误
description: 了解如何排查更新管理、更改跟踪和库存解决方案的加入错误
services: automation
author: georgewallace
ms.author: gwallace
ms.date: 03/20/2019
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: 16a03840f6bbf44853cf01e50189a194672d153e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145150"
---
# <a name="troubleshoot-errors-when-onboarding-solutions"></a>排查加入解决方案时发生的错误

在加入更新管理、更改跟踪或库存等解决方案时，可能会遇到错误。 本文描述可能会发生的各种错误及其解决方法。

## <a name="general-errors"></a>常规错误

### <a name="missing-write-permissions"></a>场景：加入失败并显示消息 - 无法启用解决方案

#### <a name="issue"></a>问题

您尝试载入虚拟机到解决方案时收到以下消息之一：

```error
The solution cannot be enabled due to missing permissions for the virtual machine or deployments
```

```error
The solution cannot be enabled on this VM because the permission to read the workspace is missing
```

#### <a name="cause"></a>原因

在虚拟机，工作区中，或用户的不正确或缺少权限导致此错误。

#### <a name="resolution"></a>解决方法

请确保你有加入虚拟机所需的权限。 查看[加入计算机所需的权限](../automation-role-based-access-control.md#onboarding)并尝试重新加入解决方案。 如果收到错误`The solution cannot be enabled on this VM because the permission to read the workspace is missing`，确保您具有`Microsoft.OperationalInsights/workspaces/read`能够找到 VM 是否加入工作区的权限。

### <a name="computer-group-query-format-error"></a>场景：ComputerGroupQueryFormatError

#### <a name="issue"></a>问题

此错误代码表示，用来将此解决方案作为目标的已保存搜索计算机组查询没有正确设置格式。 

#### <a name="cause"></a>原因

你可能已更改了该查询，或者系统可能已更改了该查询。

#### <a name="resolution"></a>解决方法

可以删除对此解决方案的查询，重新载入解决方案，这会重新创建查询。 可以在你的工作区内找到此查询，它位于“保存的搜索”下。 查询名称是 **MicrosoftDefaultComputerGroup**，查询类别是与此查询关联的解决方案的名称。 如果启用了多个解决方案，则 **MicrosoftDefaultComputerGroup** 会在“保存的搜索”下显示多次。

### <a name="policy-violation"></a>场景：PolicyViolation

#### <a name="issue"></a>问题

此错误代码表示部署由于违反一个或多个策略而失败。

#### <a name="cause"></a>原因 

实施的某个策略正在阻止操作完成。

#### <a name="resolution"></a>解决方法

为了成功部署解决方案，需要考虑更改指示的策略。 由于可以定义许多不同类型的策略，因此所需的特定更改取决于所违反的策略。 例如，如果在某个资源组上定义了一个拒绝更改该资源组中某些类型的资源内容的权限的策略，则可以（例如）执行以下任一操作：

* 将该策略完全删除。
* 尝试加入到不同的资源组。
* 通过以下方式修改策略，例如：
  * 将策略重定位到特定资源（例如，特定自动化帐户）。
  * 修改该策略已配置为拒绝的资源集。

检查 Azure 门户的右上角的通知或导航到包含你的自动化帐户和选择的资源组**部署**下**设置**若要查看失败部署。 若要了解有关 Azure Policy 的详细信息，请访问：[Azure Policy 概述](../../governance/policy/overview.md?toc=%2fazure%2fautomation%2ftoc.json)。

### <a name="unlink"></a>场景：尝试取消链接工作区的错误

#### <a name="issue"></a>问题

尝试取消链接工作区时收到以下错误：

```error
The link cannot be updated or deleted because it is linked to Update Management and/or ChangeTracking Solutions.
```

#### <a name="cause"></a>原因

您仍将活动 Log Analytics 工作区中的解决方案取决于要链接的自动化帐户和日志分析工作区时发生此错误。

### <a name="resolution"></a>解决方法

若要解决此问题将需要从你的工作区中删除以下解决方案，如果正在使用它们：

* 更新管理
* 更改跟踪
* 在非工作时间启动/停止 VM

一旦删除的解决方案可以取消链接工作区。 请务必清理工作区和自动化帐户从这些解决方案还从任何现有项目。  

* 更新管理
  * 从自动化帐户中删除更新部署 （计划）
* 在非工作时间启动/停止 VM
  * 在自动化帐户下删除解决方案组件上的所有锁**设置** > **锁**。
  * 需要删除在非工作时间启动/停止 Vm 的额外步骤查看，请[删除在非工作时间启动/停止 VM](../automation-solution-vm-management.md##remove-the-solution)。

## <a name="mma-extension-failures"></a>MMA 扩展失败

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)] 

部署解决方案时，会部署各种相关资源。 其中一个资源是 Microsoft Monitoring Agent 扩展或 Log Analytics Linux 代理。 这些是由负责与配置的 Log Analytics 工作区，用于更高版本协调的二进制文件的下载进行通信的虚拟机的来宾代理已安装的虚拟机扩展和其他文件，你是解决方案载入依赖于后开始执行。
通常，通知中心显示的通知首先会指出发生了 MMA 或 Log Analytics Linux 代理安装失败。 单击该通知可以获取有关特定失败问题的更多信息。 依次导航到“资源组”资源和“部署”元素，其中也提供了发生的部署失败问题的详细信息。
MMA 或 Log Analytics Linux 代理安装可能会出于各种原因而失败，这些失败问题的解决步骤根据问题的不同而异。 具体的故障排除步骤如下。

以下部分介绍当载入的 MMA 扩展部署中导致失败时可以遇到的各种问题。

### <a name="webclient-exception"></a>场景：在 WebClient 请求期间发生异常

虚拟机上的 MMA 扩展无法与外部资源通信，并且部署失败。

#### <a name="issue"></a>问题

下面是返回的错误消息示例：

```error
Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage.
```

```error
'Manifest download error from https://<endpoint>/<endpointId>/Microsoft.EnterpriseCloud.Monitoring_MicrosoftMonitoringAgent_australiaeast_manifest.xml. Error: UnknownError. An exception occurred during a WebClient request.
```

#### <a name="cause"></a>原因

此错误的部分潜在原因包括：

* 没有配置中的 VM，只允许特定端口的代理。

* 某个防火墙设置已阻止访问所需的端口和地址。

#### <a name="resolution"></a>解决方法

确保已打开正确的端口和地址用于通信。 有关端口和地址的列表，请参阅[规划网络](../automation-hybrid-runbook-worker.md#network-planning)。

### <a name="transient-environment-issue"></a>场景：安装失败，因为临时环境问题

安装 Microsoft Monitoring Agent 扩展在由于另一安装或阻止安装操作在部署过程失败

#### <a name="issue"></a>问题

下面是可能返回的错误消息示例：

```error
The Microsoft Monitoring Agent failed to install on this machine. Please try to uninstall and reinstall the extension. If the issue persists, please contact support.
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1618'
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.2) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.2\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1601'
```

#### <a name="cause"></a>原因

此错误的部分潜在原因包括：

* 另一项安装正在进行
* 系统会触发将模板部署期间重新启动

#### <a name="resolution"></a>解决方法

此错误是暂时性的。 请重试部署，以安装该扩展。

### <a name="installation-timeout"></a>场景：安装超时

安装了 MMA 扩展因超时而未完成。

#### <a name="issue"></a>问题

下面的示例是可能返回的错误消息：

```error
Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 15614
```

#### <a name="cause"></a>原因

此错误是因为在安装过程中要在高负载下的虚拟机。

### <a name="resolution"></a>解决方法

请尝试在 VM 的负载降低时安装 MMA 扩展。

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者无法解决问题，请访问以下渠道之一获取更多支持：

* 通过 [Azure 论坛](https://azure.microsoft.com/support/forums/)获取 Azure 专家的解答
* 与 [@AzureSupport](https://twitter.com/azuresupport)（Microsoft Azure 官方帐户）联系，它可以将 Azure 社区引导至适当的资源来改进客户体验：提供解答、支持和专业化服务。
* 如需更多帮助，可以提交 Azure 支持事件。 请转到 [Azure 支持站点](https://azure.microsoft.com/support/options/)并选择“获取支持”。
