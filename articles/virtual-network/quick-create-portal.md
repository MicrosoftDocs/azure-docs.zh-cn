---
title: 创建虚拟网络 - 快速入门 - Azure 门户
titlesuffix: Azure Virtual Network
description: 本快速入门介绍如何使用 Azure 门户创建虚拟网络。 虚拟网络能让 Azure 资源（例如虚拟机）彼此之间私下通信以及与 Internet 进行通信。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.service: virtual-network
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 11/30/2018
ms.author: kumud
ms.openlocfilehash: 995bc8e7b2eb4e9160b2b625067f20324df2cbfd
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64717014"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-portal"></a>快速入门：使用 Azure 门户创建虚拟网络

虚拟网络能让 Azure 资源（例如虚拟机 (VM)）彼此之间私下通信以及与 Internet 进行通信。 本快速入门介绍如何创建虚拟网络。 创建虚拟网络后，将两个 VM 部署到该虚拟网络中。 然后从 Internet 连接到 VM，并在两个 VM 之间进行私下通信。

如果还没有 Azure 订阅，请现在就创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="sign-in-to-azure"></a>登录 Azure

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-a-virtual-network"></a>创建虚拟网络

1. 在屏幕的左上方，选择“创建资源” > “网络” > “虚拟网络”。

1. 在“创建虚拟网络”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入 myVirtualNetwork。 |
    | 地址空间 | 输入 10.1.0.0/16。 |
    | 订阅 | 选择订阅。|
    | 资源组 | 选择“新建”，输入 myResourceGroup，然后选择“确定”。 |
    | 位置 | 选择“美国东部”。|
    | 子网 - 名称 | 输入 myVirtualSubnet。 |
    | 子网 - 地址范围 | 输入 10.1.0.0/24。 |

1. 将剩余的字段保留默认设置，然后选择 **“创建”** 。

## <a name="create-virtual-machines"></a>创建虚拟机

在虚拟网络中创建两个 VM：

### <a name="create-the-first-vm"></a>创建第一个 VM

1. 在屏幕左上角，选择“创建资源” > “计算” > “Windows Server 2016 Datacenter”。

1. 在“创建虚拟机 - 基本信息”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | **项目详细信息** | |
    | 订阅 | 选择订阅。 |
    | 资源组 | 选择 MyResourceGroup。 已在上一部分创建。 |
    | **实例详细信息** |  |
    | 虚拟机名称 | 输入 myVm1。 |
    | 区域 | 选择“美国东部”。 |
    | 可用性选项 | 保留默认值“不需要基础结构冗余”。 |
    | 映像 | 保留默认值“Microsoft Windows Server 2016 Datacenter”。 |
    | 大小 | 保留默认值“标准 DS1 v2”。 |
    | **管理员帐户** |  |
    | 用户名 | 输入所选用户名。 |
    | 密码 | 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    | 确认密码 | 重新输入密码。 |
    | **入站端口规则** |  |
    | 公共入站端口 | 保留默认值“无”。 |
    | **节省资金** |  |
    | 已有 Windows 许可证？ | 保留默认值“否”。 |

1. 选择“下一步:**磁盘”**。

1. 在“创建虚拟机 - 磁盘”中，保留默认设置，然后选择“下一步:网络”**。

1. 在“创建虚拟机 - 基本信息”中，选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 虚拟网络 | 保留默认值“myVirtualNetwork”。 |
    | 子网 | 保留默认值“myVirtualSubnet (10.1.0.0/24)”。 |
    | 公共 IP | 保留默认值“(new) myVm-ip”。 |
    | 网络安全端口 | 选择“允许所选端口”。 |
    | 选择入站端口 | 选择 HTTP 和 RDP。

1. **选择“下一步:管理”**。

1. 在“创建虚拟机 - 管理”中，为“诊断存储帐户”选择“新建”。

1. 在“创建存储帐户”中，输入或选择以下信息：

    | 设置 | 值 |
    | ------- | ----- |
    | 名称 | 输入 myvmstorageaccount。 |
    | 帐户类型 | 保留默认值“存储(常规用途 v1)”。 |
    | 性能 | 保留默认值“标准”。 |
    | 复制 | 保留默认值“本地冗余存储(LRS)”。 |

1. 选择“确定”

1. 选择“查看 + 创建”。 随后你会转到“查看 + 创建”页，Azure 将验证配置。

1. 看到“验证通过”时，选择“创建”。

### <a name="create-the-second-vm"></a>创建第二个 VM

1. 完成前面的步骤 1 和 9。

    > [!NOTE]
    > 在步骤 2 中，对于“虚拟机名称”，请输入 myVm2。
    >
    > 在步骤 7 中，对于“诊断存储帐户”，请确保选择 myvmstorageaccount。

1. 选择“查看 + 创建”。 随后你会转到“查看 + 创建”页，Azure 将验证配置。

1. 看到“验证通过”时，选择“创建”。

## <a name="connect-to-a-vm-from-the-internet"></a>从 Internet 连接到 VM

创建 myVm1 后，通过 Internet 连接到它。

1. 在门户的搜索栏中，输入 myVm1。

1. 选择“连接”按钮。

    ![连接到虚拟机](./media/quick-create-portal/connect-to-virtual-machine.png)

    选择“连接”按钮后，“连接到虚拟机”随即打开。

1. 选择“下载 RDP 文件”。 Azure 创建远程桌面协议 (.rdp) 文件，并下载到计算机。

1. 打开下载的 *.rdp* 文件。

    1. 出现提示时，选择“连接”。

    1. 输入在创建 VM 时指定的用户名和密码。

        > [!NOTE]
        > 可能需要选择“更多选择” > “使用其他帐户”，以指定在创建 VM 时输入的凭据。

1. 选择“确定”。

1. 在登录过程中可能会收到证书警告。 如果收到证书警告，请选择“确定”或“继续”。

1. VM 桌面出现后，将其最小化以返回到本地桌面。

## <a name="communicate-between-vms"></a>VM 之间进行通信

1. 在 myVm1 远程桌面中，打开 PowerShell。

1. 输入 `ping myVm2` 。

    你将看到类似于以下信息的内容：

    ```powershell
    Pinging myVm2.0v0zze1s0uiedpvtxz5z0r0cxg.bx.internal.clouda
    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 10.1.0.5:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
    ```

    由于 `ping` 使用 Internet 控制消息协议 (ICMP)，`ping` 失败。 默认情况下，不允许 ICMP 通过 Windows 防火墙。

1. 要允许 myVm2 在后面的步骤中对 myVm1 执行 ping 操作，请输入以下命令：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

    该命令允许 ICMP 通过 Windows 防火墙入站：

1. 关闭与 *myVm1* 的远程桌面连接。

1. 再次完成[从 Internet 连接到 VM](#connect-to-a-vm-from-the-internet) 中的步骤，但这次连接到 *myVm2*。

1. 从命令提示符输入 `ping myvm1`。

    你将看到类似于以下信息的内容：

    ```powershell
    Pinging myVm1.0v0zze1s0uiedpvtxz5z0r0cxg.bx.internal.cloudapp.net [10.1.0.4] with 32 bytes of data:
    Reply from 10.1.0.4: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time<1ms TTL=128

    Ping statistics for 10.1.0.4:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 1ms, Average = 0ms
    ```

    将从 *myVm1* 收到答复，因为在上一步中已经允许 ICMP 通过 *myVm1* VM 上的 Windows 防火墙。

1. 关闭与 *myVm2* 的远程桌面连接。

## <a name="clean-up-resources"></a>清理资源

使用虚拟网络和 VM 之后，请删除资源组和其包含的所有资源：

1. 在门户顶部的“搜索”框中输入“myResourceGroup”。

1. 当在搜索结果中看到“myResourceGroup”时，将其选中。

1. 选择“删除资源组”。

1. 对于“键入资源组名称”，请输入“myResourceGroup”，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了默认的虚拟网络和两个 VM。 从 Internet 连接到了其中一个 VM，并在两个 VM 之间进行了私下通信。 若要了解有关虚拟网络设置的详细信息，请参阅[管理虚拟网络](manage-virtual-network.md)。

默认情况下，Azure 可让 VM 彼此之间进行不受限制的私下通信。 相反，它只允许从 Internet 到 Windows VM 的入站远程桌面连接。 要了解有关配置不同类型的 VM 网络通信的详细信息，请转到[筛选网络流量](tutorial-filter-network-traffic.md)教程。