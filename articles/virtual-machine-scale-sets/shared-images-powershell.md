---
title: 使用共享 VM 映像在 Azure 中创建规模集 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 在 Azure 中创建用于部署虚拟机规模集的共享 VM 映像。
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
ms.openlocfilehash: ff8d94213e4e07b6597f6195126116a607c18bf7
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191714"
---
# <a name="create-and-use-shared-images-for-virtual-machine-scale-sets-with-the-azure-powershell"></a>通过 Azure PowerShell 创建和使用虚拟机规模集的共享映像

创建规模集时，需指定部署 VM 实例时要使用的映像。 共享映像库服务大大简化了整个组织中的自定义映像共享。 自定义映像类似于市场映像，不同的是自定义映像的创建者是自己。 自定义映像可用于启动配置，例如预加载应用程序、应用程序配置和其他 OS 配置。 

使用共享映像库，你可以在 AAD 租户内在同一区域或跨区域与组织中的其他用户共享自定义 VM 映像。 选择要共享哪些映像，要在哪些区域中共享，以及希望与谁共享它们。 你可以创建多个库，以便可以按逻辑方式对共享映像进行分组。 

库是顶级资源，它提供完全基于角色的访问控制 (RBAC)。 你可以控制映像的版本，并且可以选择将每个映像版本复制到一组不同的 Azure 区域。 库仅适用于托管映像。 

共享映像库功能具有多种资源类型。 我们将在本文中使用或生成这些资源类型：

| 资源 | 描述|
|----------|------------|
| **托管映像** | 这是基本映像，可以单独使用，也可用于在映像库中创建“映像版本”。 托管映像是从通用 VM 创建的。 托管映像是一种特殊的 VHD 类型，可用于生成多个 VM，并且现在可用于创建共享映像版本。 |
| **映像库** | 与 Azure 市场一样，**映像库**是用于管理和共享映像的存储库，但你可以控制谁有权访问这些映像。 |
| **映像定义** | 映像在库中定义，携带有关该映像及其在内部使用的要求的信息。 这包括了该映像是 Windows 还是 Linux 映像、发行说明以及最低和最高内存要求。 它是某种映像类型的定义。 |
| **映像版本** | 使用库时，将使用**映像版本**来创建 VM。 可根据环境的需要创建多个映像版本。 与托管映像一样，在使用**映像版本**创建 VM 时，将使用映像版本来创建 VM 的新磁盘。 可以多次使用映像版本。 |

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>开始之前

下列步骤详细说明如何将现有 VM 转换为可重用自定义映像，以便将其用于创建新 VM 实例。

完成本文中的示例，必须具有现有托管映像。 可以按照[教程：通过 Azure PowerShell 创建和使用虚拟机规模集的自定义映像](tutorial-use-custom-image-powershell.md)根据需要创建。 如果托管的映像中包含的数据磁盘，数据磁盘大小不能超过 1 TB。

通过本文进行操作时，请根据需要替换资源组和 VM 名称。


[!INCLUDE [virtual-machines-common-shared-images-ps](../../includes/virtual-machines-common-shared-images-powershell.md)]

## <a name="create-a-scale-set-from-the-shared-image-version"></a>从共享的映像版本创建规模集

使用 [New-AzVmss](/powershell/module/az.compute/new-azvmss) 创建虚拟机规模集。 以下示例从美国西部数据中心的新映像版本中创建一个规模集。 虚拟网络、公共 IP 地址和负载均衡器的 Azure 网络资源均会自动创建。 出现提示时，可针对规模集中的 VM 实例设置自己的管理凭据：

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName myVMSSRG `
  -Location 'South Central US' `
  -VMScaleSetName 'myScaleSet' `
  -VirtualNetworkName 'myVnet' `
  -SubnetName 'mySubnet'`
  -PublicIpAddressName 'myPublicIPAddress' `
  -LoadBalancerName 'myLoadBalancer' `
  -UpgradePolicyMode 'Automatic' `
  -ImageName $imageVersion.Id
```

创建和配置所有的规模集资源和 VM 需要几分钟时间。

[!INCLUDE [virtual-machines-common-gallery-list-ps](../../includes/virtual-machines-common-gallery-list-ps.md)]

[!INCLUDE [virtual-machines-common-shared-images-update-delete-ps](../../includes/virtual-machines-common-shared-images-update-delete-ps.md)]


## <a name="next-steps"></a>后续步骤

此外可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共享的映像库中创建映像定义](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [根据映像版本创建 VM](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

有关共享映像库的详细信息，请参阅[概述](shared-image-galleries.md)。 如果遇到问题，请参阅[排查共享映像库问题](troubleshooting-shared-images.md)。
