---
title: Linux 救援 VM 中的 Chroot 环境。
description: 本文介绍如何排查 Linux 中救援虚拟机 (VM) 上的 chroot 环境问题。
mservices: virtual-machines-linux
documentationcenter: ''
author: kailashmsft
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.date: 05/05/2020
ms.author: kaib
ms.openlocfilehash: 0d8a030061ef6aa848344152edaa3267ad916e2a
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/05/2020
ms.locfileid: "93377932"
---
# <a name="chroot-environment-in-a-linux-rescue-vm"></a>Linux 救援 VM 中的 Chroot 环境

本文介绍如何排查 Linux 中救援虚拟机 (VM) 上的 chroot 环境问题。

## <a name="ubuntu-16x--ubuntu-18x"></a>Ubuntu 16.x 和 Ubuntu 18.x

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount /dev/sdc1 /rescue
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="rhelcentosoracle-6x--oracle-8x--rhelcentos-7x-with-raw-partitions"></a>RHEL/Centos/Oracle 3.x && Oracle 8.x && RHEL/Centos 7、windows，其中包含原始分区

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc2 /rescue
      mount -o nouuid /dev/sdc1 /rescue/boot/
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="rhelcentos-7x-with-lvm"></a>包含 LVM 的 RHEL/Centos 7.x

   > [!NOTE]
   > 如果原始 VM 的 OS 磁盘包含逻辑卷管理器 (LVM)，请在 OS 磁盘上使用包含 Raw 分区的映像创建救援 VM。

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令激活逻辑卷组：

      ```
      vgscan --mknodes
      vgchange -ay
      lvscan
      ```

   1. 使用 `lsblk` 命令检索 LVM 名称：

      ```
      [user@myvm ~]$ lsblk
      NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
      sda                 8:0    0   64G  0 disk
      ├─sda1              8:1    0  500M  0 part /boot
      ├─sda2              8:2    0   63G  0 part /
      sdb                 8:16   0    4G  0 disk
      └─sdb1              8:17   0    4G  0 part /mnt/resource
      sdc                 8:0    0   64G  0 disk
      ├─sdc1              8:1    0  500M  0 part
      ├─sdc2              8:2    0   63G  0 part
      ├─sdc3              8:3    0    2M  0 part
      ├─sdc4              8:4    0   63G  0 part
        ├─rootvg-tmplv  253:0    0    2G  0 lvm  
        ├─rootvg-usrlv  253:1    0   10G  0 lvm  
        ├─rootvg-optlv  253:2    0    2G  0 lvm  
        ├─rootvg-homelv 253:3    0    1G  0 lvm  
        ├─rootvg-varlv  253:4    0    8G  0 lvm  
        └─rootvg-rootlv 253:5    0    2G  0 lvm
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount /dev/mapper/rootvg-rootlv /rescue
      mount /dev/mapper/rootvg-varlv /rescue/var
      mount /dev/mapper/rootvg-homelv /rescue/home
      mount /dev/mapper/rootvg-usrlv /rescue/usr
      mount /dev/mapper/rootvg-tmplv /rescue/tmp
      mount /dev/mapper/rootvg-optlv /rescue/opt
      mount /dev/sdc2 /rescue/boot/
      mount /dev/sdc1 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue/home
      umount /rescue/var
      umount /rescue/usr
      umount /rescue/tmp
      umount /rescue/opt
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="rhel-8x-with-lvm"></a>RHEL 8.x 与 LVM

   > [!NOTE]
   > 如果原始 VM 的 OS 磁盘包含逻辑卷管理器 (LVM)，请在 OS 磁盘上使用包含 Raw 分区的映像创建救援 VM。

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令激活逻辑卷组：

      ```
      vgscan --mknodes
      vgchange -ay
      lvscan
      ```

   1. 使用 `lsblk` 命令检索 LVM 名称：

      ```
      [user@myvm ~]$ lsblk
      NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
      sda                 8:0    0   64G  0 disk
      ├─sda1              8:1    0  500M  0 part /boot
      ├─sda2              8:2    0   63G  0 part /
      sdb                 8:16   0    4G  0 disk
      └─sdb1              8:17   0    4G  0 part /mnt/resource
      sdc                 8:0    0   64G  0 disk
      ├─sdc1              8:1    0  500M  0 part
      ├─sdc2              8:2    0   63G  0 part
      │ ├─rootvg-tmplv  253:0    0    2G  0 lvm  
      │ ├─rootvg-usrlv  253:1    0   10G  0 lvm  
      │ ├─rootvg-homelv 253:2    0    1G  0 lvm  
      │ ├─rootvg-varlv  253:3    0    8G  0 lvm  
      │ └─rootvg-rootlv 253:4    0    2G  0 lvm  
      ├─sdc14             8:14   0    4M  0 part
      └─sdc15             8:15   0  495M  0 part
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount /dev/mapper/rootvg-rootlv /rescue
      mount /dev/mapper/rootvg-varlv /rescue/var
      mount /dev/mapper/rootvg-homelv /rescue/home
      mount /dev/mapper/rootvg-usrlv /rescue/usr
      mount /dev/mapper/rootvg-tmplv /rescue/tmp
      mount /dev/sdc1 /rescue/boot/
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue/home
      umount /rescue/var
      umount /rescue/usr
      umount /rescue/tmp
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="oracle-7x"></a>Oracle 7、windows

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc2 /rescue
      mount -o nouuid /dev/sdc1 /rescue/boot/
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="suse-sles-12-sp4-suse-sles-12-sp4-for-sap---suse-sles-15-sp1-suse-sles-15-sp1-for-sap"></a>SUSE-SLES 12 SP4、SUSE-SLES 12 SP4 For SAP 以及 ## SUSE-SLES 15 SP1、SUSE-SLES 15 SP1 For SAP

1. 停止或解除分配受影响的 VM。
1. 使用托管磁盘在同一资源组 (RSG) 和位置中创建 OS 版本相同的救援 VM 映像。
1. 使用 Azure 门户创建受影响虚拟机的 OS 磁盘的快照。
1. 基于该 OS 磁盘快照创建一个磁盘，并将该磁盘附加到救援 VM。
1. 创建磁盘后，排查救援 VM 中的 chroot 环境问题。

   1. 使用以下命令以 root 用户身份访问 VM：

      `sudo su -`

   1. 使用 `dmesg` 查找磁盘（用于发现新磁盘的方法可能各不相同）。 以下示例使用 **dmesg** 根据 **SCSI** 磁盘进行筛选：

      `dmesg | grep SCSI`

      输出类似于以下示例。 在此示例中，我们想要查找 **sdc** 磁盘：

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. 使用以下命令访问 chroot 环境：

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc4 /rescue
      mount -o nouuid /dev/sdc3 /rescue/boot/
      mount /dev/sdc2 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. 排查 chroot 环境问题。

   1. 使用以下命令退出 chroot 环境：

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue
      ```

      > [!NOTE]
      > 如果收到错误 `unable to unmount /rescue`，请将 -l 选项添加到 umount 命令。
      >
      > 示例： `umount -l /rescue`

1. 从救援 VM 中分离磁盘，并执行与原始 VM 之间的磁盘交换。
1. 启动原始 VM 并检查其连接状态。

## <a name="next-steps"></a>后续步骤

- [排查 SSH 连接问题](troubleshoot-ssh-connection.md)
