---
title: 教程 - 使用 Azure CLI 管理 Azure 磁盘 | Microsoft Docs
description: 本教程介绍如何使用 Azure CLI 为虚拟机创建和管理 Azure 磁盘
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/14/2018
ms.author: cynthn
ms.custom: mvc
ms.subservice: disks
ms.openlocfilehash: aff00978178f63b7324168e9aca765b77decf2b4
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64922915"
---
# <a name="tutorial---manage-azure-disks-with-the-azure-cli"></a>教程 - 使用 Azure CLI 管理 Azure 磁盘

Azure 虚拟机 (VM) 使用磁盘来存储操作系统、应用程序和数据。 创建 VM 时，请务必选择适用于所需工作负荷的磁盘大小和配置。 本教程演示如何部署和管理 VM 磁盘。 学习内容：

> [!div class="checklist"]
> * OS 磁盘和临时磁盘
> * 数据磁盘数
> * 标准磁盘和高级磁盘
> * 磁盘性能
> * 附加和准备数据磁盘
> * 调整磁盘大小
> * 磁盘快照


## <a name="default-azure-disks"></a>默认 Azure 磁盘

创建 Azure 虚拟机后，会自动向此虚拟机附加两个磁盘。

**操作系统磁盘** - 操作系统磁盘大小可达 2 TB，并可托管 VM 操作系统。 默认情况下，OS 磁盘标记为“/dev/sda”。 已针对 OS 性能优化了 OS 磁盘的磁盘缓存配置。 由于此配置，OS 磁盘不应该用于应用程序或数据。 对于应用程序和数据，请使用数据磁盘，本教程后面会对其进行详细介绍。

临时磁盘- 临时磁盘使用 VM 所在的 Azure 主机上的固态驱动器。 临时磁盘具有高性能，可用于临时数据处理等操作。 但是，如果将 VM 移动到新的主机，临时磁盘上存储的数据都会被删除。 临时磁盘的大小由 VM 大小决定。 临时磁盘标记为“/dev/sdb”，且装载点为 /mnt。

## <a name="azure-data-disks"></a>Azure 数据磁盘

若要安装应用程序和存储数据，可添加额外的数据磁盘。 在任何需要持久和响应性数据存储的情况下，都应使用数据磁盘。 虚拟机的大小决定可附加到 VM 的数据磁盘数。 对于每个 VM vCPU，都可以附加四个数据磁盘。

## <a name="vm-disk-types"></a>VM 磁盘类型

Azure 提供两种类型的磁盘：标准磁盘和高级磁盘。

### <a name="standard-disk"></a>标准磁盘

标准存储受 HDD 支持，可以在确保性能的同时提供经济高效的存储。 标准磁盘适用于经济高效的开发和测试工作负荷。

### <a name="premium-disk"></a>高级磁盘

高级磁盘由基于 SSD 的高性能、低延迟磁盘提供支持。 完美适用于运行生产工作负荷的 VM。 高级存储支持 DS 系列、DSv2 系列、GS 系列和 FS 系列 VM。 选择磁盘大小时，大小值将舍入为下一类型。 例如，如果磁盘大小小于 128 GB，则磁盘类型为 P10。 如果磁盘大小介于 129 GB 和 512 GB 之间，则大小为 P20。 如果超过 512 GB，则大小为 P30。

### <a name="premium-disk-performance"></a>高级磁盘性能
[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

尽管上表确定了每个磁盘的最大 IOPS，但还可通过条带化多个数据磁盘实现更高级别的性能。 例如，Standard_GS5 VM 最多可实现 80,000 IOPS。 若要详细了解每个 VM 的最大 IOPS，请参阅 [Linux VM 大小](sizes.md)。

## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的操作步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。

若要打开 Cloud Shell，请从代码块的右上角选择“试一试”。 也可以通过转到 [https://shell.azure.com/powershell](https://shell.azure.com/bash) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

## <a name="create-and-attach-disks"></a>创建并附加磁盘

可创建磁盘，并将其附加到新建 VM 或现有 VM。

### <a name="attach-disk-at-vm-creation"></a>在 VM 创建时附加磁盘

使用 [az group create](/cli/azure/group#az-group-create) 命令创建资源组。

```azurecli-interactive
az group create --name myResourceGroupDisk --location eastus
```

使用 [az vm create](/cli/azure/vm#az-vm-create) 命令创建 VM。 下面的示例创建名为 *myVM* 的 VM，添加名为 *azureuser* 的用户帐户，并生成 SSH 密钥（如果这些密钥不存在）。 `--datadisk-sizes-gb` 参数用于指定应创建并附加到虚拟机的附加磁盘。 若要创建并附加多个磁盘，请使用空格分隔的磁盘大小值列表。 在以下示例中，创建的 VM 具有两个均为 128 GB 的数据磁盘。 因为磁盘大小为 128 GB，所以这两个磁盘都配置为 P10，每个磁盘最多提供 500 IOPS。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupDisk \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 128
```

### <a name="attach-disk-to-existing-vm"></a>将磁盘附加到现有 VM

要创建新磁盘并将其附加到现有虚拟机，请使用 [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) 命令。 以下示例创建大小为 128 GB 的高级磁盘，并将其附加到上一步创建的 VM 中。

```azurecli-interactive
az vm disk attach \
    --resource-group myResourceGroupDisk \
    --vm-name myVM \
    --disk myDataDisk \
    --size-gb 128 \
    --sku Premium_LRS \
    --new
```

## <a name="prepare-data-disks"></a>准备数据磁盘

将磁盘附加到虚拟机后，需要将操作系统配置为使用该磁盘。 以下示例演示如何手动配置磁盘。 还可使用 cloud-init 自动执行此过程，[后面的教程](./tutorial-automate-vm-deployment.md)对此进行了介绍。


创建与虚拟机的 SSH 连接。 将示例 IP 地址替换为虚拟机的公共 IP 地址。

```azurecli-interactive
ssh 10.101.10.10
```

使用 `fdisk` 对磁盘进行分区。

```bash
(echo n; echo p; echo 1; echo ; echo ; echo w) | sudo fdisk /dev/sdc
```

使用 `mkfs` 命令将文件系统写入分区。

```bash
sudo mkfs -t ext4 /dev/sdc1
```

装载新磁盘使其在操作系统中可访问。

```bash
sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive
```

现在可以通过 datadrive 装入点访问磁盘，可运行 `df -h` 命令对此进行验证。

```bash
df -h
```

输出显示新驱动器装载在 /datadrive 上。

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  1.4G   28G   5% /
/dev/sdb1       6.8G   16M  6.4G   1% /mnt
/dev/sdc1        50G   52M   47G   1% /datadrive
```

要确保在重启后重新装载驱动器，必须将其添加到 /etc/fstab 文件。 为此，请使用 `blkid` 实用工具获取磁盘的 UUID。

```bash
sudo -i blkid
```

输出显示驱动器的 UUID，在本例中为 `/dev/sdc1`。

```bash
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
```

在 /etc/fstab 文件中添加类似于以下内容的行。

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive  ext4    defaults,nofail   1  2
```

配置磁盘后，请关闭 SSH 会话。

```bash
exit
```

## <a name="take-a-disk-snapshot"></a>创建磁盘快照

创建磁盘快照时，Azure 会创建磁盘的只读时间点副本。 Azure VM 快照可用于快速保存配置更改前 VM 所处的状态。 如果出现问题或错误，则可使用快照还原 VM。 VM 具有多个磁盘时，将分别对每个磁盘进行快照。 若要执行应用程序一致性备份，请考虑在创建磁盘快照之前停止 VM。 或者，使用 [Azure 备份服务](/azure/backup/)，以便在 VM 运行时执行自动备份。

### <a name="create-snapshot"></a>创建快照

创建虚拟机磁盘快照前，需要磁盘 ID 或名称。 使用 [az vm show](/cli/azure/vm#az-vm-show) 命令返回磁盘 ID。 在此示例中，磁盘 ID 存储在变量中，以便能够在稍后的步骤中使用。

```azurecli-interactive
osdiskid=$(az vm show \
   -g myResourceGroupDisk \
   -n myVM \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)
```

获取虚拟机磁盘 ID 后，使用以下命令可创建磁盘快照。

```azurecli-interactive
az snapshot create \
    --resource-group myResourceGroupDisk \
    --source "$osdiskid" \
    --name osDisk-backup
```

### <a name="create-disk-from-snapshot"></a>从快照创建磁盘

然后，可将此快照转换为可用于重新创建虚拟机的磁盘。

```azurecli-interactive
az disk create \
   --resource-group myResourceGroupDisk \
   --name mySnapshotDisk \
   --source osDisk-backup
```

### <a name="restore-virtual-machine-from-snapshot"></a>从快照还原虚拟机

若要演示如何还原虚拟机，请删除现有虚拟机。

```azurecli-interactive
az vm delete \
--resource-group myResourceGroupDisk \
--name myVM
```

从快照磁盘创建新虚拟机。

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupDisk \
    --name myVM \
    --attach-os-disk mySnapshotDisk \
    --os-type linux
```

### <a name="reattach-data-disk"></a>重新附加数据磁盘

需要将所有数据磁盘重新附加到虚拟机。

先使用 [az disk list](/cli/azure/disk#az-disk-list) 命令找到数据磁盘名称。 此示例将磁盘名称放在名为“datadisk”的变量中，会在下一步中使用该变量。

```azurecli-interactive
datadisk=$(az disk list \
   -g myResourceGroupDisk \
   --query "[?contains(name,'myVM')].[id]" \
   -o tsv)
```

使用 [ az vm disk attach ](/cli/azure/vm/disk#az-vm-disk-attach) 命令附加磁盘。

```azurecli-interactive
az vm disk attach \
   –g myResourceGroupDisk \
   --vm-name myVM \
   --disk $datadisk
```

## <a name="next-steps"></a>后续步骤

本教程中介绍了以下 VM 磁盘主题：

> [!div class="checklist"]
> * OS 磁盘和临时磁盘
> * 数据磁盘数
> * 标准磁盘和高级磁盘
> * 磁盘性能
> * 附加和准备数据磁盘
> * 调整磁盘大小
> * 磁盘快照

转到下一教程，了解如何自动配置 VM。

> [!div class="nextstepaction"]
> [自动配置 VM](./tutorial-automate-vm-deployment.md)
