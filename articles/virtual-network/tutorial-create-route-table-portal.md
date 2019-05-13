---
title: 路由网络流量 - 教程 - Azure 门户
titlesuffix: Azure Virtual Network
description: 本教程介绍如何使用 Azure 门户通过路由表路由网络流量。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/12/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 153c692a8fb0fa538ec49c6eafa11815dd794b5d
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64681538"
---
# <a name="tutorial-route-network-traffic-with-a-route-table-using-the-azure-portal"></a>教程：使用 Azure 门户通过路由表路由网络流量

默认情况下，Azure 会在虚拟网络中的所有子网之间路由流量。 可以创建自己的路由来覆盖 Azure 的默认路由。 创建自定义路由的功能非常有用，例如，可以通过网络虚拟设备 (NVA) 在子网之间路由流量。 本教程介绍如何执行以下操作：

> [!div class="checklist"]
> * 创建路由表
> * 创建路由
> * 创建包含多个子网的虚拟网络
> * 将路由表关联到子网
> * 创建用于流量路由的 NVA
> * 将虚拟机 (VM) 部署到不同子网
> * 通过 NVA 将从一个子网的流量路由到另一个子网

如果需要，可以使用 [Azure CLI](tutorial-create-route-table-cli.md) 或 [Azure PowerShell](tutorial-create-route-table-powershell.md) 完成本教程。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="sign-in-to-azure"></a>登录 Azure

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-a-route-table"></a>创建路由表

1. 在屏幕的左上方，选择“创建资源” > “网络” > “路由表”。

1. 在“创建路由表”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入 *myRouteTablePublic*。 |
    | 订阅 | 选择订阅。 |
    | 资源组 | 选择“新建”，输入 *myResourceGroup*，然后选择“确定”。 |
    | 位置 | 保留默认值“美国东部”。
    | BGP 路由传播 | 保留默认值“已启用”。 |

1. 选择“创建”。

## <a name="create-a-route"></a>创建路由

1. 在门户的搜索栏中，输入 *myRouteTablePublic*。

1. 当“myRouteTablePublic”出现在搜索结果中时，请选择它。

1. 在“myRouteTablePublic”中的“设置”下，选择“路由” > “+ 添加”。

    ![添加路由](./media/tutorial-create-route-table-portal/add-route.png)

1. 在“添加路由”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 路由名称 | 输入 *ToPrivateSubnet*。 |
    | 地址前缀 | 输入 *10.0.1.0/24*。 |
    | 下一跃点类型 | 选择“虚拟设备”。 |
    | 下一跃点地址 | 输入 *10.0.2.4*。 |

1. 选择“确定”。

## <a name="associate-a-route-table-to-a-subnet"></a>将路由表关联到子网

将路由表关联到子网之前，必须先创建虚拟网络和子网。

### <a name="create-a-virtual-network"></a>创建虚拟网络

1. 在屏幕的左上方，选择“创建资源” > “网络” > “虚拟网络”。

1. 在“创建虚拟网络”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入 myVirtualNetwork。 |
    | 地址空间 | 输入 *10.0.0.0/16*。 |
    | 订阅 | 选择订阅。 |
    | 资源组 | 选择“选择现有项” > “myResourceGroup”。 |
    | 位置 | 保留默认值“美国东部”。 |
    | 子网 - 名称 | 输入“公共”。 |
    | 子网 - 地址范围 | 输入 *10.0.0.0/24*。 |

1. 将剩余的字段保留默认值，然后选择“创建”。

### <a name="add-subnets-to-the-virtual-network"></a>将子网添加到虚拟网络

1. 在门户的搜索栏中，输入 *myVirtualNetwork*。

1. 当“myVirtualNetwork”出现在搜索结果中时，将其选中。

1. 在“myVirtualNetwork”中的“设置”下，选择“子网” > “+ 子网”。

    ![添加子网](./media/tutorial-create-route-table-portal/add-subnet.png)

1. 在“添加子网”中输入以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入“专用”。 |
    | 地址空间 | 输入 *10.0.1.0/24*。 |

1. 将剩余的字段保留默认设置，然后选择“确定”。

1. 再次选择“+ 子网”。 这一次请输入以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入“外围网络”。 |
    | 地址空间 | 输入 *10.0.2.0/24*。 |

1. 与上次一样，将剩余的字段保留默认值，然后选择“确定”。

    Azure 显示三个子网：“公共”、“专用”和“外围网络”。

### <a name="associate-myroutetablepublic-to-your-public-subnet"></a>将 myRouteTablePublic 关联到“公共”子网

1. 选择“公共”。

1. 在“公共”中，选择“路由表” > “MyRouteTablePublic” > “保存”。

    ![关联路由表](./media/tutorial-create-route-table-portal/associate-route-table.png)

## <a name="create-an-nva"></a>创建 NVA

NVA 是帮助实现网络功能（例如路由和防火墙优化）的 VM。 如果需要，可以选择不同的操作系统。 本教程假设使用 **Windows Server 2016 Datacenter**。

1. 在屏幕的左上方，选择“创建资源” > “计算” > “Windows Server 2016 Datacenter”。

1. 在“创建虚拟机 - 基本信息”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | **项目详细信息** | |
    | 订阅 | 选择订阅。 |
    | 资源组 | 选择“myResourceGroup”。 |
    | **实例详细信息** |  |
    | 虚拟机名称 | 输入 *myVmNva*。 |
    | 区域 | 选择“美国东部”。 |
    | 可用性选项 | 保留默认值“不需要基础结构冗余”。 |
    | 映像 | 保留默认值“Microsoft Windows Server 2016 Datacenter”。 |
    | 大小 | 保留默认值“标准 DS1 v2”。 |
    | **管理员帐户** |  |
    | 用户名 | 输入所选用户名。 |
    | 密码 | 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    | 确认密码 | 重新输入密码。 |
    | **入站端口规则** |  |
    | 公共入站端口 | 保留默认值“无”。
    | **节省资金** |  |
    | 已有 Windows 许可证？ | 保留默认值“否”。 |

1. 选择“下一步:磁盘”。

1. 在“创建虚拟机 - 磁盘”中，选择符合需求的设置。

1. 选择“下一步:网络”。

1. 在“创建虚拟机 - 网络”中选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 虚拟网络 | 保留默认值“myVirtualNetwork”。 |
    | 子网 | 选择“外围网络(10.0.2.0/24)”。 |
    | 公共 IP | 选择“无”。 不需要公共 IP 地址。 VM 不会通过 Internet 进行连接。|

1. 将剩余的字段保留默认值，然后选择“下一步:管理”。

1. 在“创建虚拟机 - 管理”中，为“诊断存储帐户”选择“新建”。

1. 在“创建存储帐户”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入 *mynvastorageaccount*。 |
    | 帐户类型 | 保留默认值“存储(常规用途 v1)”。 |
    | 性能 | 保留默认值“标准”。 |
    | 复制 | 保留默认值“本地冗余存储(LRS)”。

1. 选择“确定”

1. 选择“查看 + 创建”。 随后你会转到“查看 + 创建”页，Azure 将验证配置。

1. 显示“验证通过”时，请选择“创建”。

    创建 VM 需要几分钟时间。 请等到 Azure 创建完 VM 为止。 “部署正在进行”页会显示部署详细信息。

1. VM 准备就绪后，选择“转到资源”。

## <a name="turn-on-ip-forwarding"></a>启用 IP 转发

为 *myVmNva* 启用 IP 转发。 当 Azure 向 *myVmNva* 发送网络流量时，如果流量发往不同的 IP 地址，则 IP 转发会将流量发送到正确的位置。

1. 在“myVmNva”中的“设置”下，选择“网络”。

1. 选择“myvmnva123”。 这是 Azure 为 VM 创建的网络接口。 该接口带有一个数字字符串，使其名称保持唯一。

    ![VM 网络](./media/tutorial-create-route-table-portal/virtual-machine-networking.png)

1. 在“设置”下选择“IP 配置”。

1. 在“myvmnva123 - IP 配置”中，对“IP 转发”选择“已启用”，然后选择“保存”。

    ![启用 IP 转发](./media/tutorial-create-route-table-portal/enable-ip-forwarding.png)

## <a name="create-public-and-private-virtual-machines"></a>创建公共和专用虚拟机

在虚拟网络中创建公共 VM 和专用 VM。 稍后，我们将使用这些 VM 来查看 Azure 如何通过 NVA 将“公共”子网流量路由到“专用”子网。

完成[创建 NVA](#create-an-nva) 的步骤 1-12。 为大多数字段使用相同的设置。 但以下值必须不同：

| 设置 | 值 |
| ------- | ----- |
| **公共 VM** | |
| 基础 |  |
| 虚拟机名称 | 输入 *myVmPublic*。 |
| 网络 | |
| 子网 | 选择“公共(10.0.0.0/24)”。 |
| 公共 IP 地址 | 接受默认值。 |
| 公共入站端口 | 选择“允许所选端口”。 |
| 选择入站端口 | 选择“HTTP”和“RDP”。 |
| 管理 | |
| 诊断存储帐户 | 保留默认值“mynvastorageaccount”。 |
| **专用 VM** | |
| 基础 |  |
| 虚拟机名称 | 输入 *myVmPrivate*。 |
| 网络 | |
| 子网 | 选择“专用(10.0.1.0/24)”。 |
| 公共 IP 地址 | 接受默认值。 |
| 公共入站端口 | 选择“允许所选端口”。 |
| 选择入站端口 | 选择“HTTP”和“RDP”。 |
| 管理 | |
| 诊断存储帐户 | 保留默认值“mynvastorageaccount”。 |

在 Azure 创建 *myVmPublic* VM 时，你可以创建 *myVmPrivate* VM。 在 Azure 创建完两个 VM 之前，请不要继续执行剩余的步骤。

## <a name="route-traffic-through-an-nva"></a>通过 NVA 路由流量

### <a name="sign-in-to-myvmprivate-over-remote-desktop"></a>通过远程桌面登录到 myVmPrivate

1. 在门户的搜索栏中，输入 *myVmPrivate*。

1. 当“myVmPrivate”VM 出现在搜索结果中时，请选择它。

1. 选择“连接”，以便与 *myVmPrivate* VM 建立远程桌面连接。

1. 在“连接到虚拟机”中，选择“下载 RDP 文件”。 Azure 会创建远程桌面协议 (*.rdp*) 文件，并将其下载到计算机。

1. 打开下载的 *.rdp* 文件。

    1. 出现提示时，选择“连接”。

    1. 输入创建专用 VM 时指定的用户名和密码。

    1. 可能需要选择“更多选项” > “使用其他帐户”才能使用专用 VM 的凭据。

1. 选择“确定”。

    在登录过程中可能会收到证书警告。

1. 请选择“是”以连接到 VM。

### <a name="enable-icmp-through-the-windows-firewall"></a>允许 ICMP 通过 Windows 防火墙

在稍后的步骤中，我们将使用跟踪路由工具来测试路由。 跟踪路由使用 Internet 控制消息协议 (ICMP)，而 Windows 防火墙默认拒绝该协议。 启用通过 Windows 防火墙的 ICMP。

1. 在 *myVmPrivate* 的远程桌面中，打开 PowerShell。

1. 输入以下命令：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

    在本教程中，我们将使用跟踪路由来测试路由。 对于生产环境，我们不建议允许通过 Windows 防火墙的 ICMP。

### <a name="turn-on-ip-forwarding-within-myvmnva"></a>在 myVmNva 中启用 IP 转发

我们已使用 Azure 为 VM 的网络接口[启用了 IP 转发](#turn-on-ip-forwarding)。 VM 的操作系统也必须转发网络流量。 使用以下命令为 *myVmNva* VM 的操作系统启用 IP 转发。

1. 在 *myVmPrivate* VM 中的命令提示符下，打开远程桌面并连接到 *myVmNva* VM：

    ```cmd
    mstsc /v:myvmnva
    ```

1. 从 *myVmNva* 上的 PowerShell 中输入以下命令，以启用 IP 转发：

    ```powershell
    Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
    ```

1. 重启 *myVmNva* VM。 在任务栏中，选择“开始”按钮  > “电源”按钮 >“其他(已计划)” > “继续”。

    这也会断开远程桌面会话的连接。

1. 重启 *myVmNva* VM 后，与 *myVmPublic* VM 建立远程桌面会话。 在仍与 *myVmPrivate* VM 保持连接的情况下，打开命令提示符并运行以下命令：

    ```cmd
    mstsc /v:myVmPublic
    ```
1. 在 *myVmPublic* 的远程桌面中，打开 PowerShell。

1. 输入以下命令，启用通过 Windows 防火墙的 ICMP：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

## <a name="test-the-routing-of-network-traffic"></a>测试网络流量的路由

首先，让我们测试从 *myVmPublic* VM 到 *myVmPrivate* VM 的网络流量路由。

1. 在 *myVmPublic* VM 上的 PowerShell 中，输入以下命令：

    ```powershell
    tracert myVmPrivate
    ```

    响应类似于以下示例：

    ```powershell
    Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
    over a maximum of 30 hops:

    1    <1 ms     *        1 ms  10.0.2.4
    2     1 ms     1 ms     1 ms  10.0.1.4

    Trace complete.
    ```

    可以看到，第一个跃点为路由到 10.0.2.4。 此地址是 NVA 的专用 IP 地址。 第二个跃点为路由到 *myVmPrivate* VM 的专用 IP 地址：10.0.1.4。 前面我们已将路由添加到 *myRouteTablePublic* 路由表，并已将它关联到“公共”子网。 因此，Azure 通过 NVA 发送了流量，而不是直接将流量发送到“专用”子网。

1. 关闭与 *myVmPublic* VM 建立的远程桌面会话，这样，就会与 *myVmPrivate* VM 保持连接。

1. 在 *myVmPrivate* VM 中的命令提示符下，输入以下命令：

    ```cmd
    tracert myVmPublic
    ```

    该命令测试从 *myVmPrivate* VM 到 *myVmPublic* VM 的网络流量路由。 响应类似于以下示例：

    ```cmd
    Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
    over a maximum of 30 hops:

    1     1 ms     1 ms     1 ms  10.0.0.4

    Trace complete.
    ```

    可以看到，Azure 直接将流量从 *myVmPrivate* VM 路由到了 *myVmPublic* VM。 默认情况下，Azure 直接在子网之间路由流量。

1. 关闭与 *myVmPrivate* VM 建立的远程桌面会话。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，可将资源组及其包含的所有资源一并删除：

1. 在门户的搜索栏中，输入 *myResourceGroup*。

1. 当在搜索结果中看到“myResourceGroup”时，将其选中。

1. 选择“删除资源组”。

1. 对于“键入资源组名称:”，输入“myResourceGroup”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

在本教程中，你创建了一个路由表并将其关联到了某个子网。 还创建了一个简单 NVA，用于将流量从公共子网路由到专用子网。 了解如何执行该操作后，可以部署 [Azure 市场](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking)中提供的其他预配置 NVA。 这些 NVA 可以执行许多有用的网络功能。 若要了解有关路由的详细信息，请参阅[路由概述](virtual-networks-udr-overview.md)和[管理路由表](manage-route-table.md)。

尽管可以在一个虚拟网络中部署多个 Azure 资源，但 Azure 无法将某些 PaaS 服务的资源部署到虚拟网络。 可以限制访问某些 Azure PaaS 服务的资源。 不过，只能对来自虚拟网络子网的流量进行这种限制。 若要了解如何限制 Azure PaaS 资源的网络访问，请继续学习下一篇教程。

> [!div class="nextstepaction"]
> [限制 PaaS 资源的网络访问](tutorial-restrict-network-access-to-resources.md)
