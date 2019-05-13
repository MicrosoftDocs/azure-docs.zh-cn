---
title: 诊断网络之间的通信问题 - 教程 - Azure 门户 | Microsoft Docs
description: 了解如何使用网络观察程序的 VPN 诊断功能诊断通过 Azure 虚拟网关连接到本地虚拟网络或其他虚拟网络的 Azure 虚拟网络之间的通信问题。
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
Customer intent: I need to determine why resources in a virtual network can't communicate with resources in a different network.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: e4dc66a5ca6e18b12586b5b03742fba3d6c3d6c4
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64704221"
---
# <a name="tutorial-diagnose-a-communication-problem-between-networks-using-the-azure-portal"></a>教程：使用 Azure 门户诊断网络之间的通信问题

虚拟网关将 Azure 虚拟网络连接到本地虚拟网络或其他虚拟网络。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 使用网络观察程序的 VPN 诊断功能诊断虚拟网关的问题
> * 诊断网关连接问题
> * 解决网关问题

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>先决条件

若要使用 VPN 诊断，必须有一个现成的正在运行的 VPN 网关。 如果没有现成的需要诊断的 VPN 网关，可以使用 [PowerShell 脚本](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)部署一个。 可通过以下程序运行 PowerShell 脚本：
- **本地安装的 PowerShell**：此脚本需要 Azure PowerShell `Az` 模块。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell](/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 来创建与 Azure 的连接。
- **Azure Cloud Shell**：[Azure Cloud Shell](https://shell.azure.com/powershell) 安装并配置了最新版的 PowerShell，并允许登录到 Azure。

脚本需要大约一小时来创建 VPN 网关。 剩余步骤假定要诊断的网关是通过此脚本部署的。 如果改为诊断你自己的现有网关，结果会有所不同。

## <a name="sign-in-to-azure"></a>登录 Azure

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="enable-network-watcher"></a>启用网络观察程序

如果已在“美国东部”区域中启用网络观察程序，请跳到[诊断网关](#diagnose-a-gateway)。

1. 在门户中，选择“所有服务”。 在“筛选器”框中，输入“网络观察程序”。 结果中出现“网络观察程序”后，将其选中。
2. 选择“区域”，以便将其展开，然后选择“美国东部”右侧的“...”，如下图所示：

    ![启用网络观察程序](./media/diagnose-communication-problem-between-networks/enable-network-watcher.png)

3. 选择“启用网络观察程序”。

## <a name="diagnose-a-gateway"></a>诊断网关

1. 在门户左侧选择“所有服务”。
2. 首先在“筛选”框中键入“网络观察程序”。 搜索结果中出现“网络观察程序”后，将其选中。
3. 在“网络诊断工具”下选择“VPN 诊断”。
4. 选择“存储帐户”，然后选择要向其写入诊断信息的存储帐户。
5. 在“存储帐户”列表中，选择要使用的存储帐户。 如果没有现成的存储帐户，则请选择“+ 存储帐户”，输入或选择所需信息，然后选择“创建”，以便创建一个。 如果已使用[先决条件](#prerequisites)中的脚本创建了一个 VPN 网关，则可能需要在该网关所在的资源组 *TestRG1* 中创建存储帐户。
6. 从“容器”列表中选择要使用的容器，然后选择“选择”。 如果没有任何容器，请选择“+ 容器”，输入容器的名称，然后选择“确定”。
7. 选择一个网关，然后选择“开始故障排除”。 如下图所示，测试是针对名为 **Vnet1GW** 的网关运行的：

    ![VPN 诊断](./media/diagnose-communication-problem-between-networks/vpn-diagnostics.png)

8. 当测试正在运行时，“故障排除状态”列中会显示“正在运行”，而在上图中，该列显示“未启动”。 测试可能需要数分钟的运行时间。
9. 查看已完成测试的状态。 下图显示已完成的诊断测试的状态结果：

    ![状态](./media/diagnose-communication-problem-between-networks/status.png)

    可以看到“故障排除状态”为“不正常”，还可以在“状态”选项卡上看到问题的“摘要”和“详细信息”。
10. 选择“操作”选项卡时，VPN 诊断会提供其他信息。 在下图所示的示例中，VPN 诊断指示应检查每个连接的运行状况：

    ![操作](./media/diagnose-communication-problem-between-networks/action.png)

## <a name="diagnose-a-gateway-connection"></a>诊断网关连接

网关通过网关连接连接到其他网络。 网关和网关连接必须都正常才能在虚拟网络和连接的网络之间成功通信。

1. 再次完成[诊断网关](#diagnose-a-gateway)的步骤 7，这次选择一个连接。 在以下示例中，对名为 **VNet1toSite1** 的连接进行了测试：

    ![连接](./media/diagnose-communication-problem-between-networks/connection.png)

    测试运行时间为数分钟。
2. 完成对连接的测试以后，收到的结果类似于下图中显示在“状态”和“操作”选项卡上的结果：

    ![连接状态](./media/diagnose-communication-problem-between-networks/connection-status.png)

    ![连接操作](./media/diagnose-communication-problem-between-networks/connection-action.png)

    VPN 诊断会在“状态”选项卡上指示具体错误，并会在“操作”选项卡上根据问题原因提供多项建议。

    如果测试的网关是通过[先决条件](#prerequisites)中的[脚本](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)部署的，则“状态”选项卡上的问题和“操作”选项卡上的头两个项就是要找的问题。 脚本为本地 VPN 网关设备配置了占位符形式的 IP 地址 23.99.221.164。

    若要解决此问题，需确保本地 VPN 网关已[正确配置](../vpn-gateway/vpn-gateway-about-vpn-devices.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)，并将脚本为本地网关配置的 IP 地址更改为本地 VPN 网关的实际公共地址。

## <a name="clean-up-resources"></a>清理资源

如果你使用[先决条件](#prerequisites)中的脚本创建 VPN 网关只是为了完成本教程，完成后就不再需要它，请删除资源组及其包含的所有资源：

1. 在门户顶部的“搜索”框中输入“TestRG1”。 在搜索结果中看到 **TestRG1** 后，请将其选中。
2. 选择“删除资源组”。
3. 在“键入资源组名称:”字段中输入“TestRG1”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何诊断虚拟网关的问题。 可能需要记录出入 VM 的网络通信，以便通过查看日志来了解异常情况。 若要了解具体操作方法，请转到下一教程。

> [!div class="nextstepaction"]
> [记录传入和传出 VM 的网络流量](network-watcher-nsg-flow-logging-portal.md)
