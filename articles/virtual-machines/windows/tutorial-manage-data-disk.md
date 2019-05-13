---
title: 教程 - 使用 Azure PowerShell 管理 Azure 磁盘 | Microsoft Docs
description: 本教程介绍如何使用 Azure PowerShel 为虚拟机创建和管理 Azure 磁盘
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/29/2018
ms.author: cynthn
ms.custom: mvc
ms.subservice: disks
ms.openlocfilehash: 4a60d3d77408e7c05311a2bd6bcceeb9331bd1af
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64924680"
---
# <a name="tutorial---manage-azure-disks-with-azure-powershell"></a>教程 - 使用 Azure PowerShell 管理 Azure 磁盘

Azure 虚拟机使用磁盘来存储 VM 操作系统、应用程序和数据。 创建 VM 时，请务必选择适用于所需工作负荷的磁盘大小和配置。 本教程介绍如何部署和管理 VM 磁盘。 学习内容：

> [!div class="checklist"]
> * OS 磁盘和临时磁盘
> * 数据磁盘数
> * 标准磁盘和高级磁盘
> * 磁盘性能
> * 附加和准备数据磁盘

## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。 

若要打开 Cloud Shell，只需要从代码块的右上角选择“试一试”。 也可以通过转到 [https://shell.azure.com/powershell](https://shell.azure.com/powershell) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

## <a name="default-azure-disks"></a>默认 Azure 磁盘

创建 Azure 虚拟机后，会自动向此虚拟机附加两个磁盘。 

**操作系统磁盘** - 操作系统磁盘大小最大可达 4 TB，并可托管 VM 操作系统。  OS 磁盘默认分配有一个 C: 驱动器号。 已针对 OS 性能优化了 OS 磁盘的磁盘缓存配置。 OS 磁盘不得承载应用程序或数据。 对于应用程序和数据，请使用数据磁盘，详情请参见本文稍后部分。

临时磁盘- 临时磁盘使用 VM 所在的 Azure 主机上的固态驱动器。 临时磁盘具有高性能，可用于临时数据处理等操作。 但是，如果将 VM 移动到新的主机，临时磁盘上存储的数据都会被删除。 临时磁盘的大小由 [VM 大小](sizes.md)决定。 临时磁盘默认分配有一个 D: 驱动器号。

## <a name="azure-data-disks"></a>Azure 数据磁盘

可添加额外的数据磁盘，用于安装应用程序和存储数据。 在任何需要持久和灵敏数据存储的情况下，都应使用数据磁盘。 虚拟机的大小决定可附加到 VM 的数据磁盘数。 对于每个 VM vCPU，都可以附加四个数据磁盘。

## <a name="vm-disk-types"></a>VM 磁盘类型

Azure 提供两种类型的磁盘。

**标准磁盘** - 受 HDD 支持，可以在确保性能的同时提供经济高效的存储。 标准磁盘适用于经济高效的开发和测试工作负荷。

**高级磁盘** - 由基于 SSD 的高性能、低延迟磁盘提供支持。 完美适用于运行生产工作负荷的 VM。 高级存储支持 DS 系列、DSv2 系列、GS 系列和 FS 系列 VM。 高级磁盘分为 5 种类型（P10、P20、P30、P40、P50），磁盘大小决定磁盘类型。 选择时，磁盘大小值舍入为下一类型。 例如，如果大小不到 128 GB，则磁盘类型为 P10；如果大小在 129 GB 到 512 GB 之间，则磁盘类型为 P20。

### <a name="premium-disk-performance"></a>高级磁盘性能
[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

尽管上表确定了每个磁盘的最大 IOPS，但还可通过条带化多个数据磁盘实现更高级别的性能。 例如，可向 Standard_GS5 VM 附加 64 个数据磁盘。 如果这些磁盘的大小都为 P30，则最大可实现 80,000 IOPS。 若要详细了解每个 VM 的最大 IOPS，请参阅 [VM 类型和大小](./sizes.md)。

## <a name="create-and-attach-disks"></a>创建并附加磁盘

若要完成本教程中的示例，必须具备现有虚拟机。 需要时，使用以下命令创建虚拟机。

使用 [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置虚拟机上管理员帐户所需的用户名和密码：


使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建虚拟机。 系统将提示你输入 VM 的管理员帐户的用户名和密码。

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupDisk" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" 
```


使用 [New-AzDiskConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azdiskconfig) 创建初始配置。 以下示例配置大小为 128 GB 的磁盘。

```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
    -Location "EastUS" `
    -CreateOption Empty `
    -DiskSizeGB 128
```

使用 [New-AzDisk](https://docs.microsoft.com/powershell/module/az.compute/new-Azdisk) 命令创建数据磁盘。

```azurepowershell-interactive
$dataDisk = New-AzDisk `
    -ResourceGroupName "myResourceGroupDisk" `
    -DiskName "myDataDisk" `
    -Disk $diskConfig
```

使用 [Get-AzVM](https://docs.microsoft.com/powershell/module/az.compute/get-azvm) 命令获取要向其添加数据磁盘的虚拟机。

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "myResourceGroupDisk" -Name "myVM"
```

使用 [Add-AzVMDataDisk](https://docs.microsoft.com/powershell/module/az.compute/add-azvmdatadisk) 命令向虚拟机配置添加数据磁盘。

```azurepowershell-interactive
$vm = Add-AzVMDataDisk `
    -VM $vm `
    -Name "myDataDisk" `
    -CreateOption Attach `
    -ManagedDiskId $dataDisk.Id `
    -Lun 1
```

使用 [Update-AzVM](https://docs.microsoft.com/powershell/module/az.compute/add-azvmdatadisk) 命令更新虚拟机。

```azurepowershell-interactive
Update-AzVM -ResourceGroupName "myResourceGroupDisk" -VM $vm
```

## <a name="prepare-data-disks"></a>准备数据磁盘

将磁盘附加到虚拟机后，需要将操作系统配置为使用该磁盘。 以下示例演示如何手动配置添加到 VM 的第一个磁盘。 还可使用[自定义脚本扩展](./tutorial-automate-vm-deployment.md)自动执行此过程。

### <a name="manual-configuration"></a>手动配置

创建与虚拟机的 RDP 连接。 打开 PowerShell 并运行此脚本。

```azurepowershell
Get-Disk | Where partitionstyle -eq 'raw' |
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -AssignDriveLetter -UseMaximumSize |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false
```

## <a name="verify-the-data-disk"></a>验证数据磁盘

若要验证是否已附加数据磁盘，请查看附加 `DataDisks` 的 `StorageProfile`。

```azurepowershell-interactive
$vm.StorageProfile.DataDisks
```

输出应该类似于以下示例：

```
Name            : myDataDisk
DiskSizeGB      : 128
Lun             : 1
Caching         : None
CreateOption    : Attach
SourceImage     :
VirtualHardDisk :
```


## <a name="next-steps"></a>后续步骤

本教程中介绍了以下 VM 磁盘主题：

> [!div class="checklist"]
> * OS 磁盘和临时磁盘
> * 数据磁盘数
> * 标准磁盘和高级磁盘
> * 磁盘性能
> * 附加和准备数据磁盘

转到下一教程，了解如何自动配置 VM。

> [!div class="nextstepaction"]
> [自动配置 VM](./tutorial-automate-vm-deployment.md)
