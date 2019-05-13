---
title: 创建虚拟网络 - 快速入门 - Azure PowerShell
titlesuffix: Azure Virtual Network
description: 本快速入门介绍如何使用 Azure 门户创建虚拟网络。 虚拟网络能让 Azure 资源（例如虚拟机）彼此之间私下通信以及与 Internet 进行通信。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.service: virtual-network
ms.devlang: ''
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/04/2018
ms.author: kumud
ms.openlocfilehash: 30b1b08e27eca826a9f882475723913810b3feb4
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64721852"
---
# <a name="quickstart-create-a-virtual-network-using-powershell"></a>快速入门：使用 PowerShell 创建虚拟网络

虚拟网络能让 Azure 资源（例如虚拟机 (VM)）彼此之间以及与 Internet 进行私下通信。 本快速入门介绍如何创建虚拟网络。 创建虚拟网络后，将两个 VM 部署到该虚拟网络中。 然后可以从 Internet 连接到 VM，并通过虚拟网络进行私下通信。

如果还没有 Azure 订阅，请现在就创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-powershell.md)]

如果决定在本地安装并使用 PowerShell，则本快速入门需要使用 Azure PowerShell 模块 1.0.0 版本或更高版本。 要查找已安装的版本，请运行 `Get-Module -ListAvailable Az`。 请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-az-ps)，获取安装和升级信息。

最后，如果在本地运行 PowerShell，则还将需运行 `Connect-AzAccount`。 该命令创建与 Azure 的连接。

## <a name="create-a-resource-group-and-a-virtual-network"></a>创建资源组和虚拟网络

要配置资源组和虚拟网络，必须完成几个步骤。

### <a name="create-the-resource-group"></a>创建资源组

在创建虚拟网络之前，必须创建一个资源组用于托管该虚拟网络。 使用 [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup) 创建资源组。 此示例在 eastus 位置中创建一个名为 myResourceGroup 的资源组：

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

### <a name="create-the-virtual-network"></a>创建虚拟网络

使用 [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络。 此示例在 EastUS  位置创建名为 myVirtualNetwork 的默认虚拟网络：

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

### <a name="add-a-subnet"></a>添加子网

Azure 将资源部署到虚拟网络中的子网，因此需要创建子网。 使用 [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) 创建名为“default”的子网配置：

```azurepowershell-interactive
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name default `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

### <a name="associate-the-subnet-to-the-virtual-network"></a>将子网关联到虚拟网络

可以使用 [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork) 将子网配置写入虚拟网络。 此命令创建子网：

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>创建虚拟机

在虚拟网络中创建两个 VM。

### <a name="create-the-first-vm"></a>创建第一个 VM

使用 [New-AzVM](/powershell/module/az.compute/new-azvm) 创建第一个 VM。 运行下一个命令时，系统会提示输入凭据。 为 VM 输入用户名和密码：

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "default" `
    -Name "myVm1" `
    -AsJob
```

`-AsJob` 选项在后台创建 VM。 可以继续执行下一步。

Azure 开始在后台创建 VM 时，将得到如下结果：

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```

### <a name="create-the-second-vm"></a>创建第二个 VM

使用此命令创建第二个 VM：

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "default" `
  -Name "myVm2"
```

必须创建另一个用户和密码。 Azure 创建 VM 需要几分钟时间。

> [!IMPORTANT]
> 请在 Azure 创建完 VM 之后再继续下一步骤。  Azure 将输出返回到 PowerShell 时，即完成创建。

## <a name="connect-to-a-vm-from-the-internet"></a>从 Internet 连接到 VM

使用 [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) 返回 VM 的公共 IP 地址。 此示例返回 myVm1 VM 的公共 IP 地址：

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

在本地计算机上打开命令提示符。 运行 `mstsc` 命令。 将 `<publicIpAddress>` 替换为上一步骤中返回的公共 IP 地址：

> [!NOTE]
> 如果在本地计算机上已从 PowerShell 提示符运行了这些命令，并且使用的是 Az PowerShell 模块 1.0 版本或更高版本，那么可以继续使用该接口。

```cmd
mstsc /v:<publicIpAddress>
```

将远程桌面协议 (.rdp) 文件下载到计算机后，随即会打开一个远程桌面。

1. 出现提示时，选择“连接”。

1. 输入在创建 VM 时指定的用户名和密码。

    > [!NOTE]
    > 可能需要选择“更多选择” > “使用其他帐户”，以指定在创建 VM 时输入的凭据。

1. 选择“确定”。

1. 可能会收到证书警告。 如果收到证书警告，选择“确定”或“继续”。

## <a name="communicate-between-vms"></a>VM 之间进行通信

1. 在 myVm1 远程桌面中，打开 PowerShell。

1. 输入 `ping myVm2` 。

    将得到这样的信息：

    ```powershell
    PS C:\Users\myVm1> ping myVm2

    Pinging myVm2.ovvzzdcazhbu5iczfvonhg2zrb.bx.internal.cloudapp.net
    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 10.0.0.5:
        Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
    ```

    由于使用 Internet 控制消息协议 (ICMP)，执行 ping 操作失败。 默认情况下，不允许 ICMP 通过 Windows 防火墙。

1. 要允许 myVm2 在后面的步骤中对 myVm1 执行 ping 操作，请输入以下命令：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

    该命令允许 ICMP 通过 Windows 防火墙入站。

1. 关闭与 *myVm1* 的远程桌面连接。

1. 重复[从 Internet 连接到 VM](#connect-to-a-vm-from-the-internet) 中的步骤。 这一次，连接到 myVm2。

1. 在 *myVm2* VM上的命令提示符处，输入 `ping myvm1`。

    将得到这样的信息：

    ```cmd
    C:\windows\system32>ping myVm1

    Pinging myVm1.e5p2dibbrqtejhq04lqrusvd4g.bx.internal.cloudapp.net [10.0.0.4] with 32 bytes of data:
    Reply from 10.0.0.4: bytes=32 time=2ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
    Reply from 10.0.0.4: bytes=32 time<1ms TTL=128

    Ping statistics for 10.0.0.4:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 2ms, Average = 0ms
    ```

    将从 *myVm1* 收到答复，因为在上一步中已经允许 ICMP 通过 *myVm1* VM 上的 Windows 防火墙。

1. 关闭与 *myVm2* 的远程桌面连接。

## <a name="clean-up-resources"></a>清理资源

使用虚拟网络和 VM 后，请使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) 删除资源组和组内所有资源：

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了默认的虚拟网络和两个 VM。 你从 Internet 连接到了其中一个 VM，然后该 VM 与另一个 VM 进行了私下通信。 若要了解有关虚拟网络设置的详细信息，请参阅[管理虚拟网络](manage-virtual-network.md)。

Azure 可让虚拟机彼此之间进行不受限制的私下通信。 默认情况下，Azure 仅允许从 Internet 到 Windows VM 的入站远程桌面连接。 要了解有关配置不同类型的 VM 网络通信的详细信息，请转到[筛选网络流量](tutorial-filter-network-traffic.md)教程。
