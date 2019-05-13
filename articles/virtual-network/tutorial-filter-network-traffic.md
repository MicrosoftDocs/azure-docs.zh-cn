---
title: 筛选网络流量 - 教程 - Azure 门户
titlesuffix: Azure Virtual Network
description: 本教程介绍如何在 Azure 门户中使用网络安全组筛选发往子网的网络流量。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.service: virtual-network
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/13/2018
ms.author: kumud
ms.openlocfilehash: ad34c6a876ca21bc7ef32cce638240e0d23b3177
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64723907"
---
# <a name="tutorial-filter-network-traffic-with-a-network-security-group-using-the-azure-portal"></a>教程：使用 Azure 门户通过网络安全组筛选网络流量

可以使用网络安全组来筛选虚拟网络子网的入站和出站网络流量。 网络安全组包含安全规则，这些规则可按 IP 地址、端口和协议筛选网络流量。 安全规则应用到子网中部署的资源。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建网络安全组和安全规则
> * 创建虚拟网络并将网络安全组关联到子网
> * 将虚拟机 (VM) 部署到子网中
> * 测试流量筛选器

可以根据需要使用 [Azure CLI](tutorial-filter-network-traffic-cli.md) 或 [PowerShell](tutorial-filter-network-traffic-powershell.md) 完成本教程中的步骤。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="sign-in-to-azure"></a>登录 Azure

通过 https://portal.azure.com 登录到 Azure 门户。

## <a name="create-a-virtual-network"></a>创建虚拟网络

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“网络”，然后选择“虚拟网络”。
3. 输入或选择以下信息，接受剩下的默认设置，然后选择“创建”：

    | 设置                 | 值                                              |
    | ---                     | ---                                                |
    | 名称                    | myVirtualNetwork                                   |
    | 地址空间           | 10.0.0.0/16                                        |
    | 订阅            | 选择订阅。                          |
    | 资源组          | 选择“新建”，并输入 myResourceGroup |
    | 位置                | 选择“美国东部”。                                |
    | 子网 - 名称            | mySubnet                                           |
    | 子网 - 地址范围  | 10.0.0.0/24                                        |

## <a name="create-application-security-groups"></a>创建应用程序安全组

使用应用程序安全组可以将功能类似的服务器（例如 Web 服务器）组合在一起。

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 在“在市场中搜索”框中输入“应用程序安全组”。 当“应用程序安全组”显示在搜索结果中时，将其选中，再次在“所有项”下选择“应用程序安全组”，然后选择“创建”。
3. 输入或选择以下信息，然后选择“创建”：

    | 设置        | 值                                                         |
    | ---            | ---                                                           |
    | 名称           | myAsgWebServers                                               |
    | 订阅   | 选择订阅。                                     |
    | 资源组 | 选择“使用现有”，然后选择“myResourceGroup”。 |
    | 位置       | 美国东部                                                       |

4. 再次完成步骤 3 并指定以下值：

    | 设置        | 值                                                         |
    | ---            | ---                                                           |
    | 名称           | myAsgMgmtServers                                              |
    | 订阅   | 选择订阅。                                     |
    | 资源组 | 选择“使用现有”，然后选择“myResourceGroup”。 |
    | 位置       | 美国东部                                                       |

## <a name="create-a-network-security-group"></a>创建网络安全组

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 依次选择“网络”、“网络安全组”。
3. 输入或选择以下信息，然后选择“创建”：

    |设置|值|
    |---|---|
    |名称|myNsg|
    |订阅| 选择订阅。|
    |资源组 | 选择“使用现有”，然后选择“myResourceGroup”。|
    |位置|美国东部|

## <a name="associate-network-security-group-to-subnet"></a>将网络安全组关联到子网

1. 在门户顶部的“搜索资源、服务和文档”框中，开始键入“myNsg”。 当“myNsg”出现在搜索结果中时，将其选中。
2. 在“设置”下选择“子网”，然后选择“+ 关联”，如下图所示：

    ![将 NSG 关联到子网](./media/tutorial-filter-network-traffic/associate-nsg-subnet.png)

3. 在“关联子网”下选择“虚拟网络”，然后选择“myVirtualNetwork”。 依次选择“子网”、“mySubnet”、“确定”。

## <a name="create-security-rules"></a>创建安全规则

1. 在“设置”下选择“入站安全规则”，然后选择“+ 添加”，如下图所示：

    ![添加入站安全规则](./media/tutorial-filter-network-traffic/add-inbound-rule.png)

2. 创建一项允许端口 80 和 443 与 **myAsgWebServers** 应用程序安全组通信的安全规则。 在“添加入站安全规则”下输入或选择以下值，接受其余默认值，然后选择“添加”：

    | 设置                 | 值                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | 目标             | 选择“应用程序安全组”，然后选择 **myAsgWebServers** 作为**应用程序安全组**。  |
    | 目标端口范围 | 输入 80,443                                                                                                    |
    | 协议                | 选择 TCP                                                                                                      |
    | 名称                    | Allow-Web-All                                                                                                   |

3. 使用以下值再次完成步骤 2：

    | 设置                 | 值                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | 目标             | 选择“应用程序安全组”，然后选择 **myAsgMgmtServers** 作为**应用程序安全组**。 |
    | 目标端口范围 | 输入 3389                                                                                                      |
    | 协议                | 选择 TCP                                                                                                      |
    | 优先度                | 输入 110                                                                                                       |
    | 名称                    | Allow-RDP-All                                                                                                   |

    在本教程中，将在 Internet 上为分配给 *myAsgMgmtServers* 应用程序安全组的 VM 公开 RDP（端口 3389）。 在生产环境中，建议使用 VPN 或专用网络连接来连接到要管理的 Azure 资源，而不要向 Internet 公开端口 3389。

完成步骤 1-3 以后，请复查所创建的规则。 你的列表应如下图中的列表所示：

![安全规则](./media/tutorial-filter-network-traffic/security-rules.png)

## <a name="create-virtual-machines"></a>创建虚拟机

在虚拟网络中创建两个 VM。

### <a name="create-the-first-vm"></a>创建第一个 VM

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“计算”，然后选择“Windows Server 2016 Datacenter”。
3. 输入或选择以下信息，保留剩下的默认设置，然后选择“确定”：

    |设置|值|
    |---|---|
    |名称|myVmWeb|
    |用户名| 输入所选用户名。|
    |密码| 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |订阅| 选择订阅。|
    |资源组| 选择“使用现有资源组”，再选择“myResourceGroup”。|
    |位置| 选择“美国东部”|

4. 选择 VM 的大小，然后选择“选择”。
5. 在“设置”下选择以下值，接受剩下的默认设置，然后选择“确定”：

    |设置|值|
    |---|---|
    |虚拟网络 |选择“myVirtualNetwork”|
    |网络安全组 | 选择“高级”。|
    |网络安全组（防火墙）| 选择“(新) myVmWeb-nsg”，然后在“选择网络安全组”下选择“无”。 |

6. 在“摘要”中的“创建”下，选择“创建”以启动 VM 部署。

### <a name="create-the-second-vm"></a>创建第二个 VM

再次完成步骤 1-6，但在步骤 3 中，将 VM 命名为“myVmMgmt”。 部署 VM 需要几分钟时间。 在 VM 部署完以前，请勿转到下一步。

## <a name="associate-network-interfaces-to-an-asg"></a>将网络接口关联到 ASG

门户在创建 VM 时，已为每个 VM 创建一个网络接口，并将该网络接口附加到 VM。 请将每个 VM 的网络接口添加到以前创建的应用程序安全组：

1. 在门户顶部的“搜索资源、服务和文档”框中，开始键入“myVmWeb”。 当“myVmWeb”VM 出现在搜索结果中时，请将其选中。
2. 在“设置”下选择“网络”。  选择“配置应用程序安全组”，然后选择 **myAsgWebServers** 作为**应用程序安全组**，最后选择“保存”，如下图所示：

    ![关联到 ASG](./media/tutorial-filter-network-traffic/associate-to-asg.png)

3. 再次完成步骤 1 和 2，搜索 **myVmMgmt** VM 并选择 **myAsgMgmtServers** ASG。

## <a name="test-traffic-filters"></a>测试流量筛选器

1. 连接到 *myVmMgmt* VM。 在门户顶部的“搜索”框中输入“myVmMgmt”。 当“myVmMgmt”出现在搜索结果中时，将其选中。 选择“连接”按钮。
2. 选择“下载 RDP 文件”。
3. 打开下载的 rdp 文件，然后选择“连接”。 输入在创建 VM 时指定的用户名和密码。 可能需要选择“更多选择”，然后选择“使用其他帐户”，以指定在创建 VM 时输入的凭据。
4. 选择“确定”。
5. 你可能会在登录过程中收到证书警告。 如果收到警告，请选择“是”或“继续”以继续连接。

    连接将会成功，因为允许通过端口 3389 将入站流量从 Internet 发往已附加到 *myVmMgmt* VM 的网络接口所在的 *myAsgMgmtServers* 应用程序安全组。

6. 在 PowerShell 会话中输入以下命令，从 *myVmMgmt* VM 连接到 *myVmWeb* VM：

    ``` 
    mstsc /v:myVmWeb
    ```

    可以从 myVmMgmt VM 连接到 myVmWeb VM，因为默认情况下，同一虚拟网络中的 VM 可以彼此通过任何端口通信。 但是，无法创建一个从 Internet 连接到 *myVmWeb* VM 的远程桌面连接，因为默认情况下，*myAsgWebServers* 的安全规则不允许通过端口 3389 发送来自 Internet 的入站流量，并且系统会拒绝从 Internet 到所有资源的入站流量。

7. 若要在 *myVmWeb* VM 上安装 Microsoft IIS，请在 *myVmWeb* VM 上通过 PowerShell 会话输入以下命令：

    ```powershell
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    ```

8. 完成 IIS 安装后，从 *myVmWeb* VM 断开连接，从而保留 *myVmMgmt* 远程桌面连接。
9. 从 *myVmMgmt* VM 断开连接。
10. 在 Azure 门户顶部的“搜索资源、服务和文档”框中，开始在计算机中键入“myVmWeb”。 当“myVmWeb”出现在搜索结果中时，请选择它。 记下 VM 的**公共 IP 地址**。 下图所示地址为 137.135.84.74，但你的地址不同于此：

    ![公共 IP 地址](./media/tutorial-filter-network-traffic/public-ip-address.png)
  
11. 若要确认能否从 Internet 访问 *myVmWeb* Web 服务器，请在计算机上打开 Internet 浏览器并浏览到 `http://<public-ip-address-from-previous-step>`。 此时会看到 IIS 欢迎屏幕，因为允许通过端口 80 将入站流量从 Internet 发往已附加到 *myVmWeb* VM 的网络接口所在的 *myAsgWebServers* 应用程序安全组。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，可将资源组及其包含的所有资源一并删除：

1. 在门户顶部的“搜索”框中输入“myResourceGroup”。 当在搜索结果中看到“myResourceGroup”时，将其选中。
2. 选择“删除资源组”。
3. 对于“键入资源组名称:”，输入“myResourceGroup”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

在本教程中，你已创建一个网络安全组并将其关联到虚拟网络子网。 若要详细了解网络安全组，请参阅[网络安全组概述](security-overview.md)和[管理网络安全组](manage-network-security-group.md)。

默认情况下，Azure 在子网之间路由流量。 你也可以选择通过某个 VM（例如，充当防火墙的 VM）在子网之间路由流量。 若要了解如何创建路由表，请继续学习下一教程。

> [!div class="nextstepaction"]
> [创建路由表](./tutorial-create-route-table-portal.md)