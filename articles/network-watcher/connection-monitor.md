---
title: 监视网络通信 - 教程 - Azure 门户 | Microsoft Docs
description: 了解如何使用 Azure 网络观察程序的连接监视器功能监视两个虚拟机之间的网络通信。
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to monitor communication between a VM and another VM. If the communication fails, I need to know why, so that I can resolve the problem.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/25/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: aa62c06d0c12a42d34ef9b13b8b4533d197d8d19
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64715815"
---
# <a name="tutorial-monitor-network-communication-between-two-virtual-machines-using-the-azure-portal"></a>教程：使用 Azure 门户监视两个虚拟机之间的网络通信

在虚拟机 (VM) 和终结点（例如另一 VM）之间成功通信对于组织来说可能很重要。 有时候，引入配置更改可能会导致通信中断。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建两个 VM
> * 使用网络观察程序的连接监视器功能监视 VM 之间的通信
> * 根据连接监视器指标生成警报
> * 诊断两个 VM 之间的通信问题，并了解如何解决该问题

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="sign-in-to-azure"></a>登录 Azure

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-vms"></a>创建 VM

创建两个 VM。

### <a name="create-the-first-vm"></a>创建第一个 VM

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“计算”，然后选择操作系统。 在本教程中，使用的是 **Windows Server 2016 Datacenter**。
3. 输入或选择以下信息，保留剩下的默认设置，然后选择“确定”：

    |设置|值|
    |---|---|
    |名称|myVM1|
    |用户名| 输入所选用户名。|
    |密码| 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |订阅| 选择订阅。|
    |资源组| 选择“新建”，并输入 myResourceGroup|
    |位置| 选择“美国东部”|

4. 选择 VM 的大小，然后选择“选择”。
5. 在“设置”下选择“扩展”。 选择“添加扩展”，然后选择“用于 Windows 的网络观察程序代理”，如下图所示：

    ![网络观察程序代理扩展](./media/connection-monitor/nw-agent-extension.png)

6. 在“用于 Windows 的网络观察程序代理”下选择“创建”，在“安装扩展”下选择“确定”，，然后在“扩展”下选择“确定”。
7. 接受其余“设置”的默认值，然后选择“确定”。
8. 在“摘要”中的“创建”下，选择“创建”以启动 VM 部署。

### <a name="create-the-second-vm"></a>创建第二个 VM

再次完成[创建第一个 VM](#create-the-first-vm) 中的步骤，并做出以下更改：

|步骤|设置|值|
|---|---|---|
| 1 | 选择某一版本的 **Ubuntu Server** |                                                                         |
| 3 | 名称                                  | myVm2                                                                   |
| 3 | 身份验证类型                   | 粘贴 SSH 公钥，或者在选择“密码”后输入密码。 |
| 3 | 资源组                        | 选择“使用现有资源组”，再选择“myResourceGroup”。                 |
| 6 | 扩展                            | **适用于 Linux 的网络代理**                                             |

部署 VM 需要几分钟时间。 在继续余下的步骤之前，请等待 VM 完成部署。

## <a name="create-a-connection-monitor"></a>创建连接监视器

创建一个连接监视器，监视通过 TCP 端口 22 进行的从 *myVm1* 到 *myVm2* 的通信。

1. 在门户左侧选择“所有服务”。
2. 首先在“筛选”框中键入“网络观察程序”。 搜索结果中出现“网络观察程序”后，将其选中。
3. 在“监视”下选择“连接监视器”。
4. 选择“+ 添加”。
5. 输入或选择要监视的连接信息，然后选择“添加”。 在下图所示的示例中，将通过端口 22 监视从 *myVm1* VM 到 *myVm2* VM 的连接：

    | 设置                  | 值               |
    | ---------                | ---------           |
    | 名称                     | myVm1-myVm2(22)     |
    | 源                   |                     |
    | 虚拟机          | myVM1               |
    | 目标              |                     |
    | 选择一个虚拟机 |                     |
    | 虚拟机          | myVm2               |
    | 端口                     | 22                  |

    ![添加连接监视器](./media/connection-monitor/add-connection-monitor.png)

## <a name="view-a-connection-monitor"></a>查看连接监视器

1. 完成[创建连接监视器](#create-a-connection-monitor)中的步骤 1-3 以查看连接监视。 可以看到现有连接监视器的列表，如下图所示：

    ![连接监视器](./media/connection-monitor/connection-monitors.png)

2. 选择名为 **myVm1-myVm2(22)** 的监视器（如上图所示），以便查看监视器的详细信息（如下图所示）：

    ![监视器详细信息](./media/connection-monitor/vm-monitor.png)

    请注意以下信息：

    | Item                     | 值                      | 详细信息                                                     |
    | ---------                | ---------                  |--------                                                     |
    | 状态                   | 可访问                  | 指示终结点是否可以访问。|
    | 平均 往返时间          | 指示进行连接所需的往返时间，以毫秒为单位。 连接监视器每 60 秒探测一次连接，因此可以监视一段时间的延迟情况。                                         |
    | Hops                     | 连接监视器指示两个终结点之间的跃点数。 在此示例中，连接是在同一虚拟网络中的两个 VM 之间进行的，因此只有一个到 IP 地址 10.0.0.5 的跃点。 如果在 VM 之间存在通过其他方式（例如 VPN 网关或网络虚拟设备）完成的系统的或自定义的路由、路由流量，则会列出其他跃点。                                                                                                                         |
    | 状态                   | 终结点出现绿色复选标记指示该终结点是正常的。    ||

## <a name="generate-alerts"></a>生成警报

警报通过警报规则在 Azure Monitor 中创建，可以按固定的时间间隔自动运行保存的查询或自定义日志搜索。 生成的警报可以自动运行一项或多项操作，例如通知某人或启动另一进程。 设置警报规则时，目标资源决定了可以用于生成警报的一系列指标。

1. 在 Azure 门户中选择“监视器”服务，然后选择“警报 > “新建警报规则”。
2. 单击“选择目标”，然后选择要作为目标的资源。 选择“订阅”，然后设置“资源类型”，以便筛选出要使用的连接监视器。

    ![目标为选中状态的警报屏幕](./media/connection-monitor/set-alert-rule.png)
1. 选中目标资源以后，请选择“添加条件”。网络观察程序有[创建警报时基于的指标](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-near-real-time-metric-alerts#metrics-and-dimensions-supported)。 将“可用信号”设置为指标 ProbesFailedPercent 和 AverageRoundtripMs：

    ![信号处于选中状态的警报页](./media/connection-monitor/set-alert-signals.png)
1. 填写警报详细信息，例如警报规则名称、说明和严重性。 也可向警报添加操作组，以便自动完成和自定义警报响应。

## <a name="view-a-problem"></a>查看问题

默认情况下，Azure 允许在同一虚拟网络中的 VM 之间通过所有端口进行通信。 一段时间之后，你或者组织中的其他人可能会覆盖 Azure 的默认规则，无意中引发通信故障。 完成下述用于制造通信问题的步骤，然后再次查看连接监视器：

1. 在门户顶部的搜索框中输入“myResourceGroup”。 当“myResourceGroup”资源组出现在搜索结果中时，将其选中。
2. 选择 **myVm2-nsg** 网络安全组。
3. 选择“入站安全规则”，然后选择“添加”，如下图所示：

    ![入站安全规则](./media/connection-monitor/inbound-security-rules.png)

4. 允许在一个虚拟网络的所有 VM 之间通信的默认规则是名为 **AllowVnetInBound** 的规则。 创建一项优先级高于（数字较小）**AllowVnetInBound** 规则（拒绝通过端口 22 进行的入站通信）的规则。 选择或输入以下信息，接受剩下的默认设置，然后选择“添加”：

    | 设置                 | 值          |
    | ---                     | ---            |
    | 目标端口范围 | 22             |
    | 操作                  | 拒绝           |
    | 优先度                | 100            |
    | 名称                    | DenySshInbound |

5. 由于连接监视器按 60 秒的时间间隔进行探测，因此请等待数分钟，然后在门户左侧选择“网络观察程序”、“连接监视器”，并再次选择“myVm1-myVm2(22)”监视器。 如下图所示，现在的结果有所不同：

    ![监视器详细信息错误](./media/connection-monitor/vm-monitor-fault.png)

    可以看到在 **myvm2529** 网络接口的状态列中有一个红色感叹号。

6. 若要了解状态变化的原因，请选择上图中的“10.0.0.5”。 连接监视器通知你通信失败的原因是：“由于以下网络安全组规则，通信被阻止：UserRule_DenySshInbound”。

    如果你并不知道某人已实施你在步骤 4 中创建的安全规则，则可以从连接监视器中了解到，该规则是引发通信问题的原因。 然后，你就可以更改、覆盖或删除该规则，以便还原 VM 之间的通信。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，可将资源组及其包含的所有资源一并删除：

1. 在门户顶部的“搜索”框中输入“myResourceGroup”。 当在搜索结果中看到“myResourceGroup”时，将其选中。
2. 选择“删除资源组”。
3. 对于“键入资源组名称:”，输入“myResourceGroup”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何监视两个 VM 之间的连接， 同时还介绍了如何通过网络安全组规则阻止到 VM 的通信。 若要了解连接监视器可能返回的所有不同的响应，请参阅[响应类型](network-watcher-connectivity-overview.md#response)。 还可以监视 VM、完全限定的域名、统一资源标识符或 IP 地址之间的连接。

有时候，你可能会发现某个虚拟网络中的资源无法与通过 Azure 虚拟网关连接的其他网络中的资源通信。 请转到下一教程，了解如何诊断虚拟网关的问题。

> [!div class="nextstepaction"]
> [诊断网络之间的通信问题](diagnose-communication-problem-between-networks.md)
