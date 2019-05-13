---
title: 扩展 Azure Linux VM 上的虚拟硬盘 | Microsoft Docs
description: 了解如何使用 Azure CLI 在 Linux VM 上扩展虚拟硬盘
services: virtual-machines-linux
documentationcenter: ''
author: roygara
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/15/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 4113d582647b5bea86980824714936d24dafc870
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65511146"
---
# <a name="expand-virtual-hard-disks-on-a-linux-vm-with-the-azure-cli"></a>使用 Azure CLI 扩展 Linux VM 上的虚拟硬盘

本文介绍了如何使用 Azure CLI 扩展 Linux 虚拟机 (VM) 的托管磁盘。 可通过[添加数据磁盘](add-disk.md)来扩充存储空间，也可扩展现有的数据磁盘。 在 Azure 中的 Linux VM 上，操作系统 (OS) 的默认虚拟硬盘大小通常为 30 GB。 

> [!WARNING]
> 请始终确保在文件系统处于正常状态，磁盘分区表类型将支持新的大小，并确保在执行磁盘重设大小操作之前备份了你的数据。 有关详细信息，请参阅[在 Azure 中备份 Linux VM](tutorial-backup-vms.md)。 

## <a name="expand-an-azure-managed-disk"></a>扩展 Azure 托管磁盘
确保已安装了最新的 [Azure CLI](/cli/azure/install-az-cli2) 并已使用 [az login](/cli/azure/reference-index#az-login) 登录到 Azure 帐户。

本文需要 Azure 中的现有 VM 已附加至少一个数据磁盘并且该磁盘已准备就绪。 如果尚无可用的 VM，请参阅[使用数据磁盘创建和准备 VM](tutorial-manage-disks.md#create-and-attach-disks)。

在以下示例中，请将示例参数名称（例如 *myResourceGroup* 和 *myVM*）替换成自己的值。

1. 当 VM 正在运行时，无法在虚拟硬盘上执行操作。 使用 [az vm deallocate](/cli/azure/vm#az-vm-deallocate) 解除分配 VM。 以下示例在名为 myResourceGroup 的资源组中释放名为 myVM 的 VM：

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

    > [!NOTE]
    > 只有释放 VM 才能扩展虚拟硬盘。 使用 `az vm stop` 停止 VM 不会释放计算资源。 若要释放计算资源，请使用 `az vm deallocate`。

1. 使用 [az disk list](/cli/azure/disk#az-disk-list) 查看资源组中的托管磁盘列表。 以下示例显示 myResourceGroup 资源组中托管磁盘的列表：

    ```azurecli
    az disk list \
        --resource-group myResourceGroup \
        --query '[*].{Name:name,Gb:diskSizeGb,Tier:accountType}' \
        --output table
    ```

    使用 [az disk update](/cli/azure/disk#az-disk-update) 扩展所需磁盘。 以下示例将名为 *myDataDisk* 的托管磁盘扩展为 *200* GB：

    ```azurecli
    az disk update \
        --resource-group myResourceGroup \
        --name myDataDisk \
        --size-gb 200
    ```

    > [!NOTE]
    > 扩展托管磁盘时，更新的大小将向上舍入到最接近的托管磁盘大小。 有关可用托管磁盘大小和层的表，请参阅 [Azure 托管磁盘概述 - 定价和计费](../windows/managed-disks-overview.md)。

1. 使用 [az vm start](/cli/azure/vm#az-vm-start) 启动 VM。 以下示例在名为 myResourceGroup 的资源组中启动名为 myVM 的 VM：

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```


## <a name="expand-a-disk-partition-and-filesystem"></a>扩展磁盘分区和文件系统
若要使用扩展的磁盘，请扩展基础分区和文件系统。

1. 使用适当的凭据，通过 SSH 登录到 VM。 可以使用 [az vm show](/cli/azure/vm#az-vm-show) 查看 VM 的 公共 IP 地址：

    ```azurecli
    az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
    ```

1. 扩展基础分区和文件系统。

    a. 如果磁盘已装载，请将其卸载：

    ```bash
    sudo umount /dev/sdc1
    ```

    b. 使用 `parted` 查看磁盘信息并重设分区大小：

    ```bash
    sudo parted /dev/sdc
    ```

    使用 `print` 查看有关现有分区布局的信息。 其输出类似于以下示例，该示例显示基础磁盘大小为 215 GB：

    ```bash
    GNU Parted 3.2
    Using /dev/sdc1
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) print
    Model: Unknown Msft Virtual Disk (scsi)
    Disk /dev/sdc1: 215GB
    Sector size (logical/physical): 512B/4096B
    Partition Table: loop
    Disk Flags:
    
    Number  Start  End    Size   File system  Flags
        1      0.00B  107GB  107GB  ext4
    ```

    c. 使用 `resizepart` 展开分区。 输入分区号 1以及新分区的大小：

    ```bash
    (parted) resizepart
    Partition number? 1
    End?  [107GB]? 215GB
    ```

    d. 若要退出，请输入 `quit`。

1. 重设分区大小后，请使用 `e2fsck` 验证分区一致性：

    ```bash
    sudo e2fsck -f /dev/sdc1
    ```

1. 使用 `resize2fs` 重设文件系统大小：

    ```bash
    sudo resize2fs /dev/sdc1
    ```

1. 将分区安装到目标位置，例如 `/datadrive`：

    ```bash
    sudo mount /dev/sdc1 /datadrive
    ```

1. 若要验证是否已调整数据磁盘的大小，请使用 `df -h`。 以下示例输出显示数据驱动器 */dev/sdc1* 现在为 200 GB：

    ```bash
    Filesystem      Size   Used  Avail Use% Mounted on
    /dev/sdc1        197G   60M   187G   1% /datadrive
    ```

## <a name="next-steps"></a>后续步骤
* 如需更多存储，还可以[向 Linux VM 添加数据磁盘](add-disk.md)。 
* 有关磁盘加密的详细信息，请参阅[使用 Azure CLI 加密 Linux VM 上的磁盘](encrypt-disks.md)。
