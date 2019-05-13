---
title: 使用共享 VM 映像在 Azure 中创建规模集 | Microsoft Docs
description: 了解如何使用 Azure CLI 在 Azure 中创建用于部署虚拟机规模集的共享 VM 映像。
services: virtual-machine-scale-sets
documentationcenter: ''
author: axayjo
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/25/2019
ms.author: akjosh; cynthn
ms.custom: ''
ms.openlocfilehash: 294a4afc138e25ddc70cd4fdec59310664671ae3
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149677"
---
# <a name="create-and-use-shared-images-for-virtual-machine-scale-sets-with-the-azure-cli-20"></a>通过 Azure CLI 2.0 创建和使用虚拟机规模集的共享映像

创建规模集时，需指定部署 VM 实例时要使用的映像。 [共享映像库](shared-image-galleries.md)大大简化了整个组织中的自定义映像共享。 自定义映像类似于市场映像，不同的是自定义映像的创建者是自己。 自定义映像可用于启动配置，例如预加载应用程序、应用程序配置和其他 OS 配置。 使用共享映像库，你可以在 AAD 租户内在同一区域或跨区域与组织中的其他用户共享自定义 VM 映像。 选择要共享哪些映像，要在哪些区域中共享，以及希望与谁共享它们。 你可以创建多个库，以便可以按逻辑方式对共享映像进行分组。 库是顶级资源，它提供完全基于角色的访问控制 (RBAC)。 你可以控制映像的版本，并且可以选择将每个映像版本复制到一组不同的 Azure 区域。 库仅适用于托管映像。 本文介绍如何执行以下操作：

> [!div class="checklist"]
> * 创建共享映像库
> * 创建共享映像定义
> * 创建共享映像版本
> * 基于共享映像创建 VM

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。


>[!NOTE]
> 本文介绍使用通用托管映像的过程。 不支持从专用 VM 映像创建规模集。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 2.0.46 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0]( /cli/azure/install-azure-cli)。


[!INCLUDE [virtual-machines-common-shared-images-cli](../../includes/virtual-machines-common-shared-images-cli.md)]

## <a name="create-a-scale-set-from-the-custom-vm-image"></a>从自定义 VM 映像创建规模集
创建规模集[ `az vmss create` ](/cli/azure/vmss#az-vmss-create)。 不要使用 *UbuntuLTS* 或 *CentOS* 之类的平台映像，而应指定自定义 VM 映像的名称。 以下示例创建名为 *myScaleSet* 的规模集，该规模集使用上一步骤中创建的名为 *myImage* 的自定义映像：

```azurecli-interactive
az vmss create \
  -g myGalleryRG \
  -n myScaleSet \
  --image "/subscriptions/<subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0" \
  --admin-username azureuser \
  --generate-ssh-keys
```

创建和配置所有的规模集资源和 VM 需要几分钟时间。


[!INCLUDE [virtual-machines-common-gallery-list-cli](../../includes/virtual-machines-common-gallery-list-cli.md)]


## <a name="clean-up-resources"></a>清理资源
若要删除规模集和其他资源，请使用 [az group delete](/cli/azure/group) 删除资源组及其所有资源。 `--no-wait` 参数会使光标返回提示符处，不会等待操作完成。 `--yes` 参数将确认是否希望删除资源，不会显示询问是否删除的额外提示。

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>后续步骤

此外可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共享的映像库中创建映像定义](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [根据映像版本创建 VM](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)


如果遇到任何问题，可以[对共享映像库进行故障排除](troubleshooting-shared-images.md)。
