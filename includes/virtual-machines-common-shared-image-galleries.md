---
title: include 文件
description: include 文件
services: virtual-machines
author: axayjo
ms.service: virtual-machines
ms.topic: include
ms.date: 05/06/2019
ms.author: akjosh; cynthn
ms.custom: include file
ms.openlocfilehash: 0fe1de9bb674c66d1b665de25ee579bc86e42c75
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192366"
---
共享映像库是可以帮助你围绕自定义托管 VM 映像生成结构和组织的服务。 提供了共享的图像库：

- 管理全局复制的映像。
- 版本控制和映像以更轻松地管理进行分组。
- 在支持可用性区域的区域，使您的图像高度可用使用区域冗余存储 (ZRS) 帐户。 ZRS 提供了更好地防范区域性故障的复原能力。
- 跨订阅、 和租户，使用 RBAC 甚至之间共享。

使用共享映像库，可以将映像共享给组织内的不同用户、服务主体或 AD 组。 共享映像可以复制到多个区域，以便更快地扩展部署。

托管映像是完整 VM（包括任何附加的数据磁盘）的副本或者只是 OS 磁盘的副本，具体取决于映像的创建方式。 从映像创建 VM 时，将使用该映像中的 VHD 副本来为新 VM 创建磁盘。 托管映像保留在存储中，可反复用来创建新的 VM。

如果有大量的需要维护并想要使其整个公司内可用的托管映像，可以将共享映像库用作存储库，它可以更轻松地共享你的映像。 

共享映像库功能具有多种资源类型：

| 资源 | 描述|
|----------|------------|
| **托管映像** | 可以单独使用或用于创建的基本映像**映像版本**映像库中。 托管映像是从通用 VM 创建的。 托管映像是一种特殊的 VHD 类型，可用于生成多个 VM，并且现在可用于创建共享映像版本。 |
| **映像库** | 与 Azure 市场一样，**映像库**是用于管理和共享映像的存储库，但你可以控制谁有权访问这些映像。 |
| **映像定义** | 图像库中定义和携带有关映像和在组织内使用的要求的信息。 可以包括映像是 Windows 或 Linux，最小和最大内存要求等信息和发行说明。 它是某种映像类型的定义。 |
| **映像版本** | 使用库时，将使用**映像版本**来创建 VM。 可根据环境的需要创建多个映像版本。 与托管映像一样，在使用**映像版本**创建 VM 时，将使用映像版本来创建 VM 的新磁盘。 可以多次使用映像版本。 |

<br>


![演示如何在库中创建多个映像版本的示意图](./media/shared-image-galleries/shared-image-gallery.png)

## <a name="image-definitions"></a>映像定义

图像定义是映像的版本的逻辑分组。 映像定义包含有关为何创建映像的信息、 哪些 OS，它是和使用映像的信息。 图像定义就像所有有关创建特定映像的详细信息的计划。 你不需要部署的 VM 从图像定义，而从定义创建的映像版本。


有组合-中使用的三个参数为每个映像定义**发布服务器**，**提供**并**SKU**。 这些用于查找特定映像定义。 可以拥有共享一个或两个但不是全部三个值的映像版本。  例如，以下是三个映像定义及其值：

|映像定义|发布者|产品/服务|SKU|
|---|---|---|---|
|myImage1|Contoso|财务|后端|
|myImage2|Contoso|财务|前端|
|myImage3|测试|财务|前端|

所有这三个映像都有唯一的一组值。 格式是类似于如何目前指定发布者、 产品和 SKU [Azure Marketplace 映像](../articles/virtual-machines/windows/cli-ps-findimage.md)在 Azure PowerShell，若要获取 Marketplace 映像的最新版本。 每个映像定义需要有一组唯一的这些值。

以下是可以对图像定义设置，以便可以更轻松地跟踪自己的资源的其他参数：

* 操作系统状态-可以将操作系统状态设置为通用或专用化，但仅通用化目前支持。 必须从已使用 Sysprep 的 Windows 已通用化的 Vm 创建映像或`waagent -deprovision`适用于 Linux。
* 操作系统-可以是 Windows 或 Linux。
* 说明-要在图像定义存在的原因的更多详细信息的使用说明。 例如，您可能已预安装的应用程序的前端服务器有一个映像定义。
* 最终用户许可协议的可用于指向作为特定于图像定义的最终用户许可协议。
* 隐私声明和发行说明-在 Azure 存储中存储发行说明和隐私声明，并提供了一个 URI 访问其作为映像定义的一部分。
* 生命周期结束日期-若要使用自动化来删除旧映像定义将附加到图像定义的生命周期结束日期。
* 标记-创建图像定义时，可以添加标记。 有关标记的详细信息，请参阅[使用标记来组织资源](../articles/azure-resource-manager/resource-group-using-tags.md)
* 最小值和最大 vCPU 和内存建议-如果你的映像具有 vCPU 和内存推荐配置，则可以附加该信息向映像定义。
* 不允许使用的磁盘类型-您可以提供有关你的 VM 的存储需求的信息。 例如，如果映像不适合标准 HDD 磁盘，你将向不允许列表添加它们。


## <a name="regional-support"></a>区域支持

下表列出了源区域。 所有公共区域可以是目标区域，但若要将复制到澳大利亚中部和澳大利亚中部 2 您必须具备你的订阅已列入允许列表。 要请求允许列表，请转到： https://www.microsoft.com/en-au/central-regions-eligibility/


| 源区域 |
|---------------------|-----------------|------------------|-----------------|
| 澳大利亚中部   | 美国中部 EUAP | 韩国中部    | 英国南部 2      |
| 澳大利亚中部 2 | 东亚       | 韩国南部      | 英国西部         |
| 澳大利亚东部      | 美国东部         | 美国中北部 | 美国中西部 |
| 澳大利亚东南部 | 美国东部 2       | 北欧     | 西欧     |
| 巴西南部        | 美国东部 2 EUAP  | 美国中南部 | 印度西部      |
| 加拿大中部      | 法国中部  | 印度南部      | 美国西部         |
| 加拿大东部         | 法国南部    | 东南亚   | 美国西部         |
| 印度中部       | 日本东部      | 英国北部         | 美国西部 2       |
| 美国中部          | 日本西部      | 英国南部         |                 |



## <a name="limits"></a>限制 

有限制，对于每个订阅，使用共享映像库部署资源：
- 每个订阅，每个区域 100 共享的图像库
- 1,000 图像定义，每个订阅，每个区域
- 每个订阅，每个区域 10,000 映像版本

有关详细信息，请参阅[检查资源用量与限制](https://docs.microsoft.com/azure/networking/check-usage-against-limits)有关如何检查当前使用情况的示例。
 

## <a name="scaling"></a>扩展
使用共享映像库可以指定要让 Azure 保留的映像副本数。 这有助于实现多 VM 部署方案，因为可将 VM 部署分散到不同的副本，减少单个副本过载导致实例创建过程受到限制的可能性。

![演示如何缩放映像的示意图](./media/shared-image-galleries/scaling.png)


## <a name="replication"></a>复制
使用共享映像库还可以自动将映像复制到其他 Azure 区域。 可以根据组织的需要，将每个共享映像版本复制到不同的区域。 例如，始终在多个区域复制最新的映像，而只在 1 个区域提供所有旧版本。 这有助于节省共享映像版本的存储成本。 

创建共享映像版本后，可以更新该版本要复制到的区域。 复制到不同区域所需的时间取决于要复制的数据量，以及该版本要复制到的区域数。 在某些情况下，这可能需要几个小时。 在复制期间，可以查看每个区域的复制状态。 镜像复制完成后在区域中，你可以随后部署 VM 或规模集的区域中使用该映像版本。

![演示如何复制映像的示意图](./media/shared-image-galleries/replication.png)


## <a name="access"></a>访问

与共享映像库一样，共享映像和共享映像版本都是资源，可以使用内置的本机 Azure RBAC 控件来共享它们。 使用 RBAC 可以共享这些资源与其他用户、 服务主体和组。 您甚至可以共享给各位成员以外的租户中创建它们的访问权限。 一旦用户有权访问共享的映像版本，他们可以部署虚拟机或虚拟机规模集。  以下共享矩阵可以帮助你了解用户有权访问哪些资源：

| 与用户共享     | 共享的映像库 | 共享映像 | 共享映像版本 |
|----------------------|----------------------|--------------|----------------------|
| 共享的映像库 | 是                  | 是          | 是                  |
| 共享映像         | 否                   | 是          | 是                  |
| 共享映像版本 | 否                   | 否           | 是                  |

我们建议在库级别，为获得最佳体验共享。 有关 RBAC 的详细信息，请参阅[管理 Azure 资源使用 RBAC 访问权限](../articles/role-based-access-control/role-assignments-portal.md)。

映像可以还共享，在规模较大，在租户中使用多租户应用注册。 有关在租户间共享映像的详细信息，请参阅[在 Azure 租户之间共享虚拟机库映像](../articles/virtual-machines/linux/share-images-across-tenants.md)。

## <a name="billing"></a>计费
使用共享映像库服务不会产生额外的费用。 以下资源会产生费用：
- 存储共享映像版本的存储费用。 成本取决于映像版本的副本数和版本复制到的区域数量。 例如，如果 2 张图片，并且同时复制到 3 个区域，则您将更改为 6 的托管磁盘，根据其大小。 有关详细信息，请参阅[托管磁盘定价](https://azure.microsoft.com/pricing/details/managed-disks/)。
- 从该源区域保存到在复制区域的第一个映像版本的复制的网络出口费用。 后续副本处理在区域中，因此没有任何额外费用。 

## <a name="updating-resources"></a>正在更新资源

创建后，您可以对图像库资源进行一些更改。 以下是限制为：
 
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


## <a name="sdk-support"></a>SDK 支持

以下 SDK 支持创建共享映像库：

- [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/virtualmachines/management?view=azure-dotnet)
- [Java](https://docs.microsoft.com/java/azure/?view=azure-java-stable)
- [Node.js](https://docs.microsoft.com/javascript/api/azure-arm-compute/?view=azure-node-latest)
- [Python](https://docs.microsoft.com/python/api/overview/azure/virtualmachines?view=azure-python)
- [Go](https://docs.microsoft.com/go/azure/)

## <a name="templates"></a>模板

可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共享的映像库中创建映像定义](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [根据映像版本创建 VM](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

## <a name="frequently-asked-questions"></a>常见问题 

**问：** 如何列出不同订阅中的所有共享映像库资源？ 
 
 A. 若要在 Azure 门户上列出不同订阅中你有权访问的所有共享映像库资源，请执行以下步骤：

1. 打开 [Azure 门户](https://portal.azure.com)。
1. 转到“所有资源”。
1. 选择要列出其中的所有资源的所有订阅。
1. 查找类型为“专用库”的资源。
 
   若要查看映像定义和映像版本，还应选择“显示隐藏的类型”。
 
   若要列出不同订阅中你有权访问的所有共享映像库资源，请在 Azure CLI 中使用以下命令：

   ```bash
   az account list -otsv --query "[].id" | xargs -n 1 az sig list --subscription
   ```


**问：** 是否可将现有的映像移到共享映像库？
 
 A. 可以。 根据映像的类型，可能存在 3 种场景。

 方案 1：如果你有托管映像，则可以从该映像创建映像定义和映像版本。

 方案 2：如果你有非托管的通用化映像，可以从该映像创建托管映像，然后从该托管映像创建映像定义和映像版本。 

 方案 3：如果本地文件系统中包含 VHD，则需要上传 VHD、创建托管映像，然后可以从该映像创建映像定义和映像版本。
- 如果 VHD 适用于 Windows VM，请参阅[上传通用化 VHD](https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed)。
- 如果 VHD 适用于 Linux VM，请参阅[上传 VHD](https://docs.microsoft.com/azure/virtual-machines/linux/upload-vhd#option-1-upload-a-vhd)


**问：** 是否可以从专用化磁盘创建映像版本？

 A. 不可以，目前不支持将专用化磁盘用作映像。 如果你有专用化磁盘，需要通过将专用化磁盘附加到新 VM，[从 VHD 创建 VM](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-specialized-portal#create-a-vm-from-a-disk)。 运行 VM 后，需要遵照有关从 [Windows VM](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-custom-images) 或 [Linux VM](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-custom-images) 创建托管映像的说明操作。 创建通用化托管映像后，可以启动创建共享映像说明和映像版本的过程。

 
**问：** 创建后，是否可将共享映像库资源移到其他订阅？

 A. 不可以，无法将共享映像库资源移到其他订阅。 但是，可根据需要将库中的映像版本复制到其他区域。

**问：** 是否可以跨云（Azure 中国世纪互联、Azure 德国和 Azure 政府云）复制映像版本？ 

 A. 无法跨云复制映像版本。

**问：** 是否可以跨订阅复制映像版本？ 

 A. 不可以。但可以跨订阅中的区域复制映像版本，并通过 RBAC 在其他订阅中使用该版本。

**问：** 是否可以跨 Azure AD 租户共享映像版本？ 

 A. 是的您可以使用 RBAC 在租户间共享到个人。 不过，若要共享的规模，请参阅"映像跨 Azure 租户的共享库"使用[PowerShell](../articles/virtual-machines/windows/share-images-across-tenants.md)或[CLI](../articles/virtual-machines/linux/share-images-across-tenants.md)。


**问：** 跨目标区域复制映像版本需要多长时间？

 A. 映像版本复制时间完全取决于映像的大小，以及它要复制到的区域数。 但是，作为最佳做法，我们建议缩小映像，并在相互靠近的源与目标区域之间进行复制，以获得最佳效果。 可以使用 -ReplicationStatus 标志检查复制状态。


**问：** 源区域与目标区域之间的区别是什么？

 A. 源区域是创建映像版本的区域，而目标区域是存储映像版本副本的区域。 每个映像版本只能有一个源区域。 此外，在创建映像版本时，请确保将源区域位置传递为目标区域之一。  


**问：** 创建映像版本时如何指定源区域？

 A. 创建映像版本时，可以在 CLI 中使用 **-location** 标记或者在 PowerShell 中使用 **-Location** 标记，来指定源区域。 请确保要用作基本映像来创建映像版本的托管映像，位于创建映像版本的同一位置。 此外，在创建映像版本时，请确保将源区域位置传递为目标区域之一。  


**问：** 如何指定要在每个区域中创建的映像版本副本数？

 A. 可通过两种方式指定要在每个区域中创建的映像版本副本数：
 
1. 区域副本计数：指定要在每个区域创建的副本数。 
2. 通用副本计数：未指定区域副本计数时每个区域的默认计数。 

若要指定的区域副本计数，请将传递以及你想要在该区域中创建的副本数目的位置：“美国中南部=2”。 

如果未为每个位置指定区域副本计数，则默认副本数将是指定的通用副本计数。 

若要在 CLI 中指定通用副本计数，请在 `az sig image-version create` 命令中使用 **--replica-count** 参数。


**问：** 是否可以不在创建映像定义和映像版本的位置创建共享映像库？

 A. 是的，可以这样做。 但是，最佳做法是，我们建议您将资源组、 共享的映像库、 映像定义和映像版本保存在同一位置。


**问：** 使用共享映像库会产生哪些费用？

 A. 使用共享映像库服务不会产生费用，不过，存储映像版本会产生存储费用，将映像版本从源区域复制到目标区域会产生网络传出费用。

**问：** 应使用哪个 API 版本来创建共享映像库、映像定义和映像版本，并基于映像版本创建 VM/VMSS？

 A. 若要使用映像版本部署 VM 和虚拟机规模集，我们建议使用 API 2018-04-01 或更高版本。 若要处理共享映像库、映像定义和映像版本，我们建议使用 API 版本 2018-06-01。 
