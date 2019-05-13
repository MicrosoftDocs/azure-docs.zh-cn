---
title: 如何在 Azure 虚拟机中启用嵌套虚拟化 | Microsoft 文档
description: 如何在 Azure 虚拟机中启用嵌套虚拟化
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
ms.author: cynthn
ms.date: 10/09/2017
ms.topic: conceptual
ms.service: virtual-machines-windows
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.openlocfilehash: acb44a34eae84d8a5718ebcc0003d3cf50b9d43a
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510044"
---
# <a name="how-to-enable-nested-virtualization-in-an-azure-vm"></a>如何在 Azure VM 中启用嵌套虚拟化

多个 Azure 虚拟机系列支持嵌套虚拟化。 此功能可在支持开发、测试、培训和演示环境等方面提供极大的灵活性。   

本文逐步介绍如何在 Azure VM 上启用 Hyper-V，并配置到该来宾虚拟机的 Internet 连接。

## <a name="create-a-nesting-capable-azure-vm"></a>创建一个支持嵌套的 Azure VM

创建新的 Windows Server 2016 Azure VM。  为了快速参考，所有 v3 虚拟机都支持嵌套虚拟化。 有关支持嵌套的虚拟机大小的完整列表，请参阅[Azure 计算单位](acu.md)一文。

请记得选择足以支持来宾虚拟机需求的 VM 大小。 在此示例中，我们将使用 D3_v3 大小的 Azure VM。 

可以在[此处](https://azure.microsoft.com/regions/services/)查看 Dv3 或 Ev3 系列虚拟机的区域可用性。

>[!NOTE]
>
>有关创建新虚拟机的详细说明，请参阅[使用 Azure PowerShell 模块创建和管理 Windows VM](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-manage-vm)
    
## <a name="connect-to-your-azure-vm"></a>连接到 Azure VM

创建到虚拟机的远程桌面连接。

1. 单击虚拟机属性上的“连接”按钮。 此时会创建和下载远程桌面协议文件（.rdp 文件）。

2. 若要连接到 VM，请打开下载的 RDP 文件。 出现提示时，请单击“连接”。 在 Mac 上，需要一个 RDP 客户端，例如 Mac 应用商店提供的这个[远程桌面客户端](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12)。

3. 输入在创建虚拟机时指定的用户名和密码，并单击“确定”。

4. 你可能会在登录过程中收到证书警告。 单击“是”或“继续”继续进行连接。

## <a name="enable-the-hyper-v-feature-on-the-azure-vm"></a>启用 Azure VM 上的 HYPER-V 功能
你可以手动配置这些设置，或者使用我们提供的 PowerShell 脚本来自动完成配置。

### <a name="option-1-use-a-powershell-script-to-configure-nested-virtualization"></a>选项 1：使用 PowerShell 脚本配置嵌套虚拟化
在 [GitHub](https://github.com/charlieding/Virtualization-Documentation/tree/live/hyperv-tools/Nested) 上提供了用于在 Windows Server 2016 主机上启用嵌套虚拟化的 PowerShell 脚本。 该脚本将首先检查先决条件，然后在 Azure VM 上配置嵌套虚拟化。 必须重启 Azure VM 才能完成配置。 此脚本在其他环境中也可以运行，但不能保证。 有关在 Azure 上运行嵌套虚拟化的现场视频演示，请查看 Azure 博客文章！ https://aka.ms/AzureNVblog。

### <a name="option-2-configure-nested-virtualization-manually"></a>选项 2：手动配置嵌套虚拟化

1. 在 Azure VM 上，以管理员身份打开 PowerShell。 

2. 启用 HYPER-V 功能和管理工具。

    ```powershell
    Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
    ```

    >[!WARNING] 
    >
    >此命令将重启 Azure VM。 在重启过程中将失去 RDP 连接。
    
3. Azure VM 重启后，请使用 RDP 重新连接 VM。

## <a name="set-up-internet-connectivity-for-the-guest-virtual-machine"></a>设置来宾虚拟机的 Internet 连接
为来宾虚拟机创建新虚拟网络适配器，并配置 NAT 网关以启用 Internet 连接。

### <a name="create-a-nat-virtual-network-switch"></a>创建 NAT 虚拟网络交换机

1. 在 Azure VM 上，以管理员身份打开 PowerShell。
   
2. 创建内部交换机。

    ```powershell
    New-VMSwitch -Name "InternalNAT" -SwitchType Internal
    ```

3. 查看交换机的属性，并记下新适配器的 ifIndex。

    ```powershell
    Get-NetAdapter
    ```

    ![NetAdapter](./media/virtual-machines-nested-virtualization/get-netadapter.png)

    >[!NOTE] 
    >
    >记下你刚创建的虚拟交换机的“ifIndex”。
    
4. 为 NAT 网关创建 IP 地址。
    
若要配置网关，需要一些有关你网络的信息：    
  * IPAddress - NAT 网关 IP 指定要用作虚拟网络子网的默认网关地址的 IPv4 或 IPv6 地址。 常规形式为 a.b.c.1（例如，“192.168.0.1”）。 尽管最后一个位置不一定是 .1，但通常是 1（基于前缀长度）。 通常情况下，应使用 RFC 1918 专用网络地址空间。 
  * PrefixLength - 子网前缀长度定义本地子网大小（子网掩码）。 子网前缀长度将介于 0 到 32 之间的一个整数值。 0 将映射整个 Internet，32 则只允许一个映射的 IP。 常用值范围从 24 到 12，具体要取决于多少 IP 需要附加到 NAT。 常用 PrefixLength 为 24 -- 这是子网掩码 255.255.255.0。
  * InterfaceIndex - ifIndex 是上一步中创建的虚拟交换机的接口索引。 

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 13
    ```

### <a name="create-the-nat-network"></a>创建 NAT 网络

若要配置网关，将需要提供有关网络和 NAT 网关的信息：
  * Name - 这是 NAT 网络的名称。 
  * InternalIPInterfaceAddressPrefix - NAT 子网前缀描述了上述 NAT 网关 IP 前缀，以及上述 NAT 子网前缀长度。 常规形式将为 a.b.c.0/NAT 子网前缀长度。 

在 PowerShell 中，创建一个新的 NAT 网络。
```powershell
New-NetNat -Name "InternalNat" -InternalIPInterfaceAddressPrefix 192.168.0.0/24
```


## <a name="create-the-guest-virtual-machine"></a>创建来宾虚拟机

1. 打开 Hyper-V 管理器并创建新的虚拟机。 配置虚拟机以使用你创建的新内部网络。
    
    ![NetworkConfig](./media/virtual-machines-nested-virtualization/configure-networking.png)
    
2. 在来宾虚拟机上安装操作系统。
    
    >[!NOTE] 
    >
    >你需要将操作系统的安装媒体安装到 VM 上。 在这种情况下，我们将使用 Windows 10 企业版。

## <a name="assign-an-ip-address-to-the-guest-virtual-machine"></a>向来宾虚拟机分配 IP 地址

你可以通过手动设置来宾虚拟机上的静态 IP 地址向来宾虚拟机分配 IP 地址，也可以在 Azure VM 上配置 DHCP 来动态分配 IP 地址。

###  <a name="option-1-configure-dhcp-to-dynamically-assign-an-ip-address-to-the-guest-virtual-machine"></a>选项 1：配置 DHCP 以将 IP 地址动态分配给来宾虚拟机
按照下面的步骤在主机虚拟机上配置 DHCP，以实现动态地址分配。

#### <a name="install-dchp-server-on-the-azure-vm"></a>在 Azure VM 上安装 DCHP 服务器

1. 打开服务器管理器。 在仪表板中，单击“添加角色和功能”。 随即会出现“添加角色和功能”向导。
  
2. 在向导中，单击“下一步”，直到出现“服务器角色”页。
  
3. 单击以选择“DHCP 服务器”复选框，然后依次单击“添加功能”和“下一步”，直至完成向导。
  
4. 单击“安装”。

#### <a name="configure-a-new-dhcp-scope"></a>配置新的 DHCP 作用域

1. 打开 DHCP 管理器。
  
2. 在导航窗格中，展开服务器名称，右键单击“IPv4”，然后单击“新作用域”。 “新作用域”向导出现后，单击“下一步”。
  
3. 输入作用域的名称和说明，然后单击“下一步”。
  
4. 为你的 DCHP 服务器定义 IP 范围（例如，192.168.0.100 到 192.168.0.200）。
  
5. 单击“下一步”直到出现“默认网关”页。 输入之前创建的 IP 地址（例如，192.168.0.1）作为默认网关，然后单击“添加”。
  
6. 单击“下一步”直到完成向导，保留所有默认值，然后单击“完成”。
    
### <a name="option-2-manually-set-a-static-ip-address-on-the-guest-virtual-machine"></a>选项 2：在来宾虚拟机上手动设置静态 IP 地址
如果未配置 DHCP 以向来宾虚拟机动态分配 IP 地址，请按照以下步骤设置静态 IP 地址。

1. 在 Azure VM 上，以管理员身份打开 PowerShell。

2. 右键单击来宾虚拟机，然后单击“连接”。

3. 登录到来宾虚拟机。

4. 在来宾虚拟机中，打开“网络和共享中心”。

5. 为上一节中创建的 NAT 网络范围内的地址配置网络适配器。

在此示例中，将使用 192.168.0.0/24 范围内的地址。

## <a name="test-connectivity-in-guest-virtual-machine"></a>在来宾虚拟机中测试连接

在来宾虚拟机中，打开浏览器并导航到网页。
    ![GuestVM](./media/virtual-machines-nested-virtualization/guest-virtual-machine.png)

## <a name="set-up-intranet-connectivity-for-the-guest-virtual-machine"></a>设置来宾虚拟机的 Intranet 连接

有关如何在来宾 VM 和 Azure VM 之间启用透明连接的说明，请参阅[此文档](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization-azure-virtual-network)。
