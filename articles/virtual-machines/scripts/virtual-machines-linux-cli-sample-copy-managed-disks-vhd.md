---
title: Azure CLI 示例 - 将托管磁盘复制到存储帐户
description: Azure CLI 示例 - 将托管磁盘导出或复制到存储帐户。
services: virtual-machines-linux
documentationcenter: storage
author: ramankumarlive
manager: kavithag
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/09/2019
ms.author: ramankum
ms.custom: mvc,seodec18
ms.openlocfilehash: c5055860df007d0f009bd08c2b211ffeeaad711f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510880"
---
# <a name="exportcopy-a-managed-disk-to-a-storage-account-using-the-azure-cli"></a>使用 Azure CLI 将托管磁盘导出/复制到存储帐户

此脚本将托管磁盘的基础 VHD 导出到相同或不同区域中的存储帐户。 它首先生成托管磁盘的 SAS URI，然后使用该 SAS URI 将 VHD 复制到存储帐户。 使用此脚本将托管磁盘复制到另一区域以进行区域扩展。 若要在 Azure 市场中发布托管磁盘的 VHD 文件，可以使用此脚本将 VHD 文件复制到存储帐户，然后生成已复制的 VHD 的 SAS URI，以便在市场中发布。   


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-azurecli[main](../../../cli_scripts/virtual-machine/copy-managed-disks-vhd-to-storage-account/copy-managed-disks-vhd-to-storage-account.sh "Copy the VHD of a managed disk")]


## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令生成托管磁盘的 SAS URI，然后使用该 SAS URI 将基础 VHD 复制到存储帐户。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az disk grant-access](https://docs.microsoft.com/cli/azure/disk?view=azure-cli-latest#az-disk-grant-access) | 生成只读 SAS，使用该 SAS 可以将基础 VHD 文件复制到存储帐户或将其下载到本地  |
| [az storage blob copy start](https://docs.microsoft.com/cli/azure/storage/blob/copy) | 将 blob 从一个存储帐户异步复制到另一个存储帐户 |

## <a name="next-steps"></a>后续步骤

[基于 VHD 创建托管磁盘](virtual-machines-linux-cli-sample-create-managed-disk-from-vhd.md?toc=%2fcli%2fmodule%2ftoc.json)

[基于托管磁盘创建虚拟机](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure)。

可以在 [Azure Linux VM 文档](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他虚拟机和托管磁盘 CLI 脚本示例。
