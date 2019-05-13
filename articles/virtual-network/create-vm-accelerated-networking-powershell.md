---
title: 创建具有加速网络的 Azure 虚拟机 | Microsoft Docs
description: 了解如何创建具有加速网络的 Linux 虚拟机。
services: virtual-network
documentationcenter: ''
author: gsilva5
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/04/2018
ms.author: gsilva
ms.openlocfilehash: ef6086afa17f1ab864d70678a6da6df2a78e0c16
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190263"
---
# <a name="create-a-windows-virtual-machine-with-accelerated-networking"></a>创建具有加速网络的 Windows 虚拟机

本教程介绍如何创建具有加速网络的 Windows 虚拟机 (VM)。 若要使用加速网络创建 Linux VM，请参阅[使用加速网络创建 Linux VM](create-vm-accelerated-networking-cli.md)。 使用加速网络可以实现对 VM 的单根 I/O 虚拟化 (SR-IOV)，大幅提升其网络性能。 这种高性能路径会绕过数据路径中的主机，降低延迟、抖动，以及受支持 VM 类型上的最苛刻网络工作负荷的 CPU 利用率。 下图显示了具有和没有加速网络的两个 VM 之间的通信：

![比较](./media/create-vm-accelerated-networking/accelerated-networking.png)

在不使用加速网络的情况下，传入和传出 VM 的所有网络流量必须遍历主机和虚拟交换机。 虚拟交换机针对网络流量实施所有策略，例如网络安全组、访问控制列表、隔离和其他网络虚拟化服务。 若要了解有关虚拟交换机的详细信息，请参阅[HYPER-V 网络虚拟化和虚拟交换机](https://technet.microsoft.com/library/jj945275.aspx)。

在使用加速网络的情况下，网络流量将抵达 VM 的网络接口 (NIC)，然后转发到 VM。 由虚拟交换机应用的所有网络策略现在都会卸载，并在硬件中应用。 由于在硬件中应用策略，NIC 可以绕过主机和虚拟交换机将网络流量直接转发到 VM，同时保留它在主机中应用的所有策略。

加速网络的优势仅适用于已启用该功能的 VM。 为获得最佳效果，最好是在连接到同一个 Azure 虚拟网络 (VNet) 的最少两个 VM 上启用此功能。 跨 VNet 通信或者在本地连接时，此功能对总体延迟的影响极小。

## <a name="benefits"></a>优点
* **更低的延迟/更高的每秒数据包数 (pps)：** 从数据路径中去除虚拟交换机可以消除数据包在主机中进行策略处理所花费的时间，同时增大了 VM 中可处理的数据包数。
* **减少抖动：** 虚拟交换机处理取决于需要应用的策略数量，以及正在执行处理的 CPU 工作负荷。 将策略实施卸载到硬件消除了这种可变性，因为可以将数据包直接传送到 VM，省去了主机与 VM 之间的通信，以及所有的软件中断和上下文切换。
* **降低了 CPU 利用率：** 绕过主机中的虚拟交换机可以减少用于处理流量的 CPU 资源。

## <a name="limitations-and-constraints"></a>限制和约束

### <a name="supported-operating-systems"></a>支持的操作系统
从 Azure 库即可支持以下分发：
* **Windows Server 2016 Datacenter** 
* **Windows Server 2012 R2 Datacenter**

### <a name="supported-vm-instances"></a>支持的 VM 实例
大多数常规用途实例以及具有 2 个或更多 vCPU 的计算优化实例都支持加速网络。  这些受支持的系列包括：D/DSv2 和 F/Fs

在支持超线程的实例上，具有 4 个或更多 vCPU 的 VM 实例支持加速网络。 受支持的系列包括：D/Dsv3、 E/Esv3、 Fsv2、 Lsv2、 Ms/Mms 和 Ms/Mmsv2。

有关 VM 实例的详细信息，请参阅[Windows VM 大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="regions"></a>区域
在所有公共 Azure 区域和 Azure 政府云中均可用。

### <a name="enabling-accelerated-networking-on-a-running-vm"></a>在正在运行的 VM 上启用加速网络
未启用加速网络的受支持 VM 大小只有在停止和解除分配时才能启用该功能。

### <a name="deployment-through-azure-resource-manager"></a>通过 Azure 资源管理器部署
虚拟机（经典）无法部署加速网络。

## <a name="create-a-windows-vm-with-azure-accelerated-networking"></a>创建具有 Azure 加速网络的 Windows VM
## <a name="portal-creation"></a>在门户中创建
尽管本文提供了具有加速网络使用 Azure Powershell 创建虚拟机的步骤，但也可以[具有加速网络使用 Azure 门户创建虚拟机](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 在创建虚拟机在门户中，当**创建虚拟机**边栏选项卡中，选择**网络**选项卡。在此选项卡上，没有适合**加速网络**。  如果已选择[受支持的操作系统](#supported-operating-systems)并[的 VM 大小](#supported-vm-instances)，此选项将自动填充为"开启"。  如果没有，它将填充加速网络的"关闭"选项，并为用户提供的原因未启用它的原因。   
* *注意：* 可以通过门户启用仅支持的操作系统。  如果正在使用自定义映像，并且你的映像支持加速网络，请使用创建 VM CLI 或 Powershell。 

创建虚拟机后，您可以确认以下已启用加速网络的确认中的说明启用了加速网络。

## <a name="powershell-creation"></a>Powershell 创建
## <a name="create-a-virtual-network"></a>创建虚拟网络

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

安装[Azure PowerShell](/powershell/azure/install-az-ps)版本 1.0.0 或更高版本。 要查找当前安装的版本，请运行 `Get-Module -ListAvailable Az`。 如果你需要安装或升级，安装最新版本中的 Az 模块[PowerShell 库](https://www.powershellgallery.com/packages/Az)。 在 PowerShell 会话中，登录到 Azure 帐户使用[Connect AzAccount](/powershell/module/az.accounts/connect-azaccount)。

在以下示例中，请将示例参数名称替换为自己的值。 参数名称示例包括 myResourceGroup、myNic 和 myVM。

使用 [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup) 创建资源组。 以下示例在 centralus 位置创建名为 myResourceGroup 的资源组：

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "centralus"
```

首先，创建的子网配置[新建 AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/New-azVirtualNetworkSubnetConfig)。 以下示例创建名为 mySubnet 的子网：

```powershell
$subnet = New-AzVirtualNetworkSubnetConfig `
    -Name "mySubnet" `
    -AddressPrefix "192.168.1.0/24"
```

创建具有虚拟网络[新建 AzVirtualNetwork](/powershell/module/az.Network/New-azVirtualNetwork)，使用*mySubnet*子网。

```powershell
$vnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
    -Location "centralus" `
    -Name "myVnet" `
    -AddressPrefix "192.168.0.0/16" `
    -Subnet $Subnet
```

## <a name="create-a-network-security-group"></a>创建网络安全组

首先，创建网络安全组规则[新建 AzNetworkSecurityRuleConfig](/powershell/module/az.Network/New-azNetworkSecurityRuleConfig)。

```powershell
$rdp = New-AzNetworkSecurityRuleConfig `
    -Name 'Allow-RDP-All' `
    -Description 'Allow RDP' `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389
```

创建网络安全组与[新建 AzNetworkSecurityGroup](/powershell/module/az.Network/New-azNetworkSecurityGroup) ，并将分配*允许 RDP 全部*安全规则应用于它。 除 Allow-RDP-All 规则外，该网络安全组还包含多个默认规则。 其中一个默认规则会禁用所有来自 Internet 的入站访问，这正是向网络安全组分配 Allow-RDP-All 规则的原因，为的是在虚拟机创建后能够远程连接到虚拟机。

```powershell
$nsg = New-AzNetworkSecurityGroup `
    -ResourceGroupName myResourceGroup `
    -Location centralus `
    -Name "myNsg" `
    -SecurityRules $rdp
```

将关联到的网络安全组*mySubnet*子网[集 AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/Set-azVirtualNetworkSubnetConfig)。 网络安全组中的规则对子网中部署的所有资源都是有效的。

```powershell
Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name 'mySubnet' `
    -AddressPrefix "192.168.1.0/24" `
    -NetworkSecurityGroup $nsg
```

## <a name="create-a-network-interface-with-accelerated-networking"></a>创建具有加速网络的网络接口
使用 [New-AzPublicIpAddress](/powershell/module/az.Network/New-azPublicIpAddress) 创建一个公共 IP 地址。 如果不打算从 Internet 访问虚拟机，则不需要公共 IP 地址，但必须完成本文中的步骤。

```powershell
$publicIp = New-AzPublicIpAddress `
    -ResourceGroupName myResourceGroup `
    -Name 'myPublicIp' `
    -location centralus `
    -AllocationMethod Dynamic
```

创建与网络接口[新建 AzNetworkInterface](/powershell/module/az.Network/New-azNetworkInterface)启用了加速网络，并将公共 IP 地址分配给网络接口。 下面的示例在 myVnet 虚拟网络的 mySubnet 子网中创建名为 myNic 的网络接口，并向其分配 myPublicIp 公共 IP 地址：

```powershell
$nic = New-AzNetworkInterface `
    -ResourceGroupName "myResourceGroup" `
    -Name "myNic" `
    -Location "centralus" `
    -SubnetId $vnet.Subnets[0].Id `
    -PublicIpAddressId $publicIp.Id `
    -EnableAcceleratedNetworking
```

## <a name="create-the-virtual-machine"></a>创建虚拟机

使用 [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential) 将 VM 凭据设置为 `$cred` 变量：

```powershell
$cred = Get-Credential
```

首先，定义使用 VM[新建 AzVMConfig](/powershell/module/az.compute/new-azvmconfig)。 以下示例定义名为 myVM 的 VM，其大小支持加速网络 (*Standard_DS4_v2*)：

```powershell
$vmConfig = New-AzVMConfig -VMName "myVm" -VMSize "Standard_DS4_v2"
```

若要获取所有 VM 大小和特性列表，请参阅 [Windows VM 大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

通过 [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) 和 [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) 创建 VM 配置的其余部分。 以下示例创建一个 Windows Server 2016 VM：

```powershell
$vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
    -Windows `
    -ComputerName "myVM" `
    -Credential $cred `
    -ProvisionVMAgent `
    -EnableAutoUpdate
$vmConfig = Set-AzVMSourceImage -VM $vmConfig `
    -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" `
    -Skus "2016-Datacenter" `
    -Version "latest"
```

将先前创建的网络接口附加[添加 AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface):

```powershell
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
```

最后，创建 VM [New-azvm](/powershell/module/az.compute/new-azvm):

```powershell
New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "centralus"
```

## <a name="confirm-the-driver-is-installed-in-the-operating-system"></a>确认操作系统中安装了驱动程序

在 Azure 中创建 VM 后，立即连接到 VM 并确认 Windows 中安装了驱动程序。

1. 在 Internet 浏览器中打开 Azure [门户](https://portal.azure.com)并使用 Azure 帐户登录。
2. 在 Azure 门户顶部包含“搜索资源”文本的框中，键入 myVm。 当“myVm”出现在搜索结果中时，请单击它。 如果“连接”按钮下方显示“正在创建”，则表示 Azure 尚未完成创建 VM。 仅在“连接”按钮下方不再显示“正在创建”时，才在概述的左上角单击“连接”。
3. 输入在[创建虚拟机](#create-the-virtual-machine)中输入的用户名和密码。 如果从未连接到 Azure 中的 Windows VM，请参阅[连接到虚拟机](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#connect-to-virtual-machine)。
4. 右键单击 Windows“开始”按钮，并单击“设备管理器”。 展开“网络适配器”节点。 确认已显示“Mellanox ConnectX-3 Virtual Function Ethernet Adapter”，如下图所示：

    ![设备管理器](./media/create-vm-accelerated-networking/device-manager.png)

现在已为 VM 启用加速网络。

## <a name="enable-accelerated-networking-on-existing-vms"></a>在现有 VM 上启用加速网络
如果创建的 VM 没有加速网络，则可在现有 VM 上启用此功能。  VM 必须支持加速网络，前提是满足以下先决条件（上文亦有列出）：

* VM 必须是加速网络支持的大小
* VM 必须是受支持的 Azure 库映像（以及适用于 Linux 的内核版本）
* 在任何 NIC 上启用加速网络前，必须停止/解除分配可用性集或 VMSS 中的所有 VM

### <a name="individual-vms--vms-in-an-availability-set"></a>单个 VM 与可用性集中的 VM
首先停止/解除分配 VM，或集合中的所有 VM（如果是可用性集）：

```azurepowershell
Stop-AzureRmVM -ResourceGroup "myResourceGroup" `
    -Name "myVM"
```

请务必注意，如果 VM 是单独创建的并且没有可用性集，则只需停止/解除分配单个 VM 即可启用加速网络。  如果 VM 是随可用性集创建的，则在任何 NIC 上启用加速网络前，必须停止/解除分配可用性集中包含的所有 VM。 

一旦停止，即可在 VM 的 NIC 上启用加速网络：

```azurepowershell
$nic = Get-AzureRMNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic"

$nic.EnableAcceleratedNetworking = $true

$nic | Set-AzureRMNetworkInterface
```

重新启动 VM，则，如果在可用性集中，在组中，所有 Vm，并确认已启用加速网络：

```azurepowershell
Start-AzureRmVM -ResourceGroup "myResourceGroup" `
    -Name "myVM"
```

### <a name="vmss"></a>VMSS
VMSS 略有不同，但遵循相同的工作流。  首先，停止 VM：

```azurepowershell
Stop-AzVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet"
```

VM 停止后，更新网络接口下的加速网络属性：

```azurepowershell
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet"

$vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].EnableAcceleratedNetworking = $true

Update-AzVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -VirtualMachineScaleSet $vmss
```

请注意，VMSS 的 VM 升级功能可使用三种不同的设置（自动、滚动和手动）应用更新。  在这些说明中，策略设置为自动，以便 VMSS 在可重启后立即收到更改。  若要将其设置为自动以便立即收到更改，请执行以下操作：

```azurepowershell
$vmss.UpgradePolicy.AutomaticOSUpgrade = $true

Update-AzVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -VirtualMachineScaleSet $vmss
```

最后，重启 VMSS：

```azurepowershell
Start-AzVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet"
```

请在重启后等待升级完成，但一旦完成，VF 将在 VM 内部出现。  （请确保使用的是支持的操作系统和 VM 大小）

### <a name="resizing-existing-vms-with-accelerated-networking"></a>调整具有加速网络的现有 VM 的大小

启用加速网络的 VM 只能调整为支持加速网络的 VM 的大小。  

启用加速网络的 VM 不能使用调整大小操作调整为不支持加速网络的 VM 实例的大小。  相反，若要调整其中一个 VM 的大小，请执行以下操作：

* 停止/解除分配 VM，或如果在可用性集/VMSS 中，则停止/解除分配集合/VMSS 中的所有 VM。
* 必须在 VM 的 NIC 上禁用加速网络，或者如果在可用性集/VMSS 中，则必须在集合/​​VMSS 中的所有 VM 上禁用。
* 一旦加速网络被禁用，VM/可用性集/VMSS 即可移至不支持加速网络的新大小并重启。
