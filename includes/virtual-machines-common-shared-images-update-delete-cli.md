---
title: include 文件
description: include 文件
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: b34d37fa79ccb8344fdacd99877403d61ba5f5c2
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138907"
---
## <a name="update-resources"></a>更新资源

有一些限制可以更新的内容。 可以更新以下项目： 

共享映像库：
- 描述

映像定义：
- 建议的 vCPU 数
- 建议的内存
- 描述
- 生命周期终结日期

映像版本：
- 区域副本计数
- 目标区域数
- 从最新项中排除
- 生命周期终结日期

更新的库使用说明 ([az sig 更新](https://docs.microsoft.com/cli/azure/sig?view=azure-cli-latest#az-sig-update)。 

```azurecli-interactive
az sig update \
   --gallery-name myGallery \
   --resource-group myGalleryRG \
   --set description="My updated description."
```


更新映像定义使用的说明[az sig 图像定义更新](https://docs.microsoft.com/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-update)。

```azurecli-interactive
az sig image-definition update \
   --gallery-name myGallery\
   --resource-group myGalleryRG \
   --gallery-image-definition myImageDefinition \
   --set description="My updated description."
```

更新要添加的区域复制到使用的映像版本[az sig 映像版本更新](https://docs.microsoft.com/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-update)。 此更改将需要一段时间，如图像将复制到新的区域。

```azurecli-interactive
az sig image-version update \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --add publishingProfile.targetRegions  name=eastus
```

## <a name="delete-resources"></a>删除资源

您必须首先删除映像版本的相反顺序删除资源。 删除的所有映像的版本后，您可以删除图像定义。 删除所有图像定义后，可以都删除库。 

删除映像版本使用[az sig 映像版本删除](https://docs.microsoft.com/cli/azure/sig/image-version?view=azure-cli-latest#az-sig-image-version-delete)。

```azurecli-interactive
az sig image-version delete \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 
```

删除映像定义使用[az sig 图像定义删除](https://docs.microsoft.com/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-delete)。

```azurecli-interactive
az sig image-definition delete \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition
```


删除映像库使用[az sig 删除](https://docs.microsoft.com/cli/azure/sig?view=azure-cli-latest#az-sig-delete)。

```azurecli-interactive
az sig delete \
   --resource-group myGalleryRG \
   --gallery-name myGallery
```