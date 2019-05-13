---
title: 在 Azure 中的租户之间共享库映像 |Microsoft Docs
description: 了解如何在使用共享图像库的 Azure 租户间共享的 VM 映像。
services: virtual-machines-linux
author: cynthn
manager: jeconnoc
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 04/05/2019
ms.author: cynthn
ms.openlocfilehash: 1578ba840c6dca93feb68754863439811d7ef099
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65158725"
---
# <a name="share-gallery-vm-images-across-azure-tenants"></a>在 Azure 租户之间共享库的 VM 映像

[!INCLUDE [virtual-machines-share-images-across-tenants](../../../includes/virtual-machines-share-images-across-tenants.md)]


## <a name="create-a-vm-using-azure-cli"></a>使用 Azure CLI 创建 VM

在租户 1 使用 appID、 应用密钥和租户 1 的 ID 的服务主体登录。 可以使用`az account show --query "tenantId"`以根据需要获取租户 Id。

```azurecli-interactive
az account clear
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 1 ID>'
az account get-access-token 
```
 
在租户 2 使用 appID、 应用密钥和租户 2 的 ID 的服务主体登录：

```azurecli-interactive
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 2 ID>'
az account get-access-token
```

创建 VM。 在示例中的信息将替换为自己。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image "/subscriptions/<Tenant 1 subscription>/resourceGroups/<Resource group>/providers/Microsoft.Compute/galleries/<Gallery>/images/<Image definition>/versions/<version>" \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="next-steps"></a>后续步骤

如果遇到任何问题，可以[对共享映像库进行故障排除](troubleshooting-shared-images.md)。