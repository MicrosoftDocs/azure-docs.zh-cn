---
title: 附加到 Azure VM 的磁盘的故障排除 |Microsoft 文档
description: 提供指向 Azure 虚拟机虚拟硬盘 (VHD) 的故障排除资源的链接。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 10/31/2018
ms.author: rogarana
ms.reviewer: wmgries
ms.openlocfilehash: c00fdf3ad02edc1faf0d3257d3836a1c2f44d682
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150770"
---
# <a name="troubleshoot-disks-attached-to-azure-vms"></a>附加到 Azure VM 的磁盘的故障排除 

Azure 虚拟机 (VM) 依赖于 OS 磁盘和任何附加数据磁盘的虚拟硬盘 (VHD)。 VHD 作为页 Blob 存储在一个或多个 Azure 存储帐户中。 本文主要介绍如何对 VHD 可能会出现的常见问题进行排查。 

## <a name="troubleshoot-storage-deletion-errors-for-a-vm"></a>排查 VM 的存储删除错误

在某些情况下，当资源管理器部署中的 VM 包含附加的 VHD 时，在删除存储资源时可能会遇到错误。 请参阅以下文章之一，以帮助解决此问题： 

  * 在 Linux VM 上：[资源管理器部署中的存储删除错误](../../virtual-machines/linux/storage-resource-deletion-errors.md)  
  * 在 Windows VM 上：[资源管理器部署中的存储删除错误](../../virtual-machines/windows/storage-resource-deletion-errors.md)  

## <a name="troubleshoot-unexpected-reboots-of-vms-with-attached-vhds"></a>排查附加 VHD 的 VM 意外重启

如果遇到大量附加 VHD 的 VM 意外重启，请参阅以下文章之一：

  * 在 Linux VM 上：[附加 VHD 的 VM 意外重新启动](../../virtual-machines/linux/unexpected-reboots-attached-vhds.md)
  * 在 Windows VM 上：[附加 VHD 的 VM 意外重新启动](../../virtual-machines/linux/unexpected-reboots-attached-vhds.md)
