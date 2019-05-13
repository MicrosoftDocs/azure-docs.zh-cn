---
author: cynthn
ms.author: cynthn
ms.date: 04/30/2019
ms.topic: include
ms.service: virtual-machines-linux
manager: jeconnoc
ms.openlocfilehash: e1b3b5fe603072069cb3a19c7597fcc1872fefd7
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416257"
---
标准化的虚拟机 (VM) 映像，组织可迁移到云，并确保在部署中的一致性。 映像通常包括预定义的安全和配置设置和必要的软件。 设置映像管道需要时间、 基础结构和安装过程中，但使用 Azure VM 映像生成器，只需提供简单的配置描述你的映像、 将其提交到服务，和生成，并分发映像。
 
Azure VM 映像生成器 （Azure 映像生成器），可以使用 Windows 或基于 Linux 的 Azure Marketplace 映像，现有的自定义映像或 Red Hat Enterprise Linux (RHEL) ISO 启动并开始添加自己的自定义。 由于基于映像生成器[HashiCorp Packer](https://packer.io/)，还可以导入现有的 Packer shell 预配程序脚本。 此外可以指定你希望你托管在 Azure 共享映像库 (virtual-machines-common-shared-image-galleries.md) 作为托管映像或 VHD 的映像。

> [!IMPORTANT]
> Azure 映像生成器目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="preview-features"></a>预览版功能

对于预览版，支持这些功能：

- 黄金基准映像，创建包括您的最小安全和企业配置，并留出部门来进一步对其进行自定义以满足其需求。
- 修补的现有映像，映像生成器将允许您不断修补现有的自定义映像。
- 与 Azure 共享映像库中的集成，可分发版本，和缩放映像全局，并为您提供了映像管理系统。
- 与现有的映像的集成生成管道，只需从你的管道，调用图像生成器或使用简单的预览图像生成器 Azure DevOps 任务。
- 迁移到 Azure 的现有映像自定义管道。 使用现有的脚本、 命令和进程自定义映像。
- 使用 Red Hat 使订阅支持。 创建具有符合条件的、 未使用 Red Hat 订阅使用的 Red Hat Enterprise 映像。
- 创建的 VHD 格式的图像。
 

## <a name="regions"></a>区域
Azure 映像生成器服务将可供在这些区域中预览。 映像可以分发这些区域之外。
- 美国东部
- 美国东部 2
- 美国中西部
- 美国西部
- 美国西部 2

## <a name="os-support"></a>OS 支持
AIB 将支持 Azure Marketplace 基础操作系统映像：
- Ubuntu 18.04
- Ubuntu 16.04
- RHEL 7.6
- CentOS 7.6
- Windows 2016
- Windows 2019


## <a name="how-it-works"></a>工作原理


![Azure 映像生成器的概念图](./media/virtual-machines-image-builder-overview/image-builder.png)

Azure 映像生成器是完全托管的 Azure 服务访问的 Azure 资源提供程序。 Azure 映像生成器进程具有三个主要部分： 源、 自定义和分发，它们都表示在模板中。 下图显示了带有它们的某些属性的组件。 
 


**图像生成器进程** 

![Azure 映像生成器进程的概念图](./media/virtual-machines-image-builder-overview/image-builder-process.png)

1. 创建映像模板为.json 文件。 此.json 文件包含有关图像源、 自定义和分发的信息。 有多个示例中的[Azure 映像生成器 GitHub 存储库](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts)。
1. 将其提交到服务，这将指定的资源组中创建的映像模板项目。 在后台，映像生成器将下载源映像或 ISO，并根据需要的脚本。 它们存储在你的订阅，采用格式中自动创建单独的资源组：数据<DestinationResourceGroup>_<TemplateName>。 
1. 创建映像模板后，你可以然后生成映像。 在后台图像生成器使用的模板和源代码文件以在数据中创建 VM、 网络和存储<DestinationResourceGroup>_<TemplateName>资源组。
1. 映像生成器将映像分发作为映像创建的一部分，它根据模板，然后删除数据中的其他资源<DestinationResourceGroup>_<TemplateName>针对过程创建的资源组。


## <a name="permissions"></a>权限

若要允许 Azure VM 映像生成器分发到托管映像或共享映像库的映像，你将需要提供服务"Azure 虚拟机映像生成器"的参与者权限 (应用 ID: cf32a0cc-373c-47c9-9156-0db11f6a6dfc) 上的资源组。 

如果使用现有的自定义托管的映像或映像版本，Azure 映像生成器将需要向这些资源组的读取者访问权限的最小值。

可以分配访问权限使用 Azure CLI:

```azurecli-interactive
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/<distributeResoureGroupName>
```

如果找不到的服务帐户，这可能意味着，正在添加的角色分配的订阅尚未注册资源提供程序。


## <a name="costs"></a>成本
将会产生一些计算、 网络和存储成本时创建、 生成和存储使用 Azure 映像生成器的映像。 这些成本是类似于手动创建自定义映像所产生的成本。 对于资源，你将在 Azure 费率收费。 

在映像创建过程中，文件将下载并存储在`IT_<DestinationResourceGroup>_<TemplateName>`资源组，这将会导致较小的存储成本。 f 不希望保留这些图像生成后删除映像模板。
 
映像生成器创建所需的 vm 使用 D1v2 VM 大小和存储和网络的 VM。 这些资源将持续的生成过程的持续时间和映像生成器已完成创建映像会被删除。 
 
Azure 映像生成器将映像分发到你所选区域，这可能会导致网络出口费用。
 
## <a name="next-steps"></a>后续步骤 
 
若要试用 Azure 映像生成器，请参阅构建的文章[Linux](../articles/virtual-machines/linux/image-builder.md)或[Windows](../articles/virtual-machines/windows/image-builder.md)映像。
 
 
