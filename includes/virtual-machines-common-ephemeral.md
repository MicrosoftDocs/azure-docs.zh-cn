---
title: include 文件
description: include 文件
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 05/02/2019
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: 47407df90a83501b8739a428789e20cddc59e83d
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65468332"
---
临时 OS 磁盘是本地的虚拟机 (VM) 存储上创建并不会保留到远程 Azure 存储。 临时 OS 磁盘适用于无状态工作负载，其中应用程序容错的单个 VM 失败，但更为关注大规模部署所需的时间或时间进行映像重置单个 VM 实例。 也很适合使用经典部署模型转移到 Resource Manager 部署模型部署的应用程序。 使用临时 OS 磁盘后，将观察到的 OS 磁盘和更快的 VM 重置映像较低的读取/写入延迟。 此外，临时 OS 磁盘是免费的则会产生的 OS 磁盘存储免费。 
 
临时磁盘的关键功能包括： 
- 它们可以用于 Marketplace 映像和自定义映像。
- 可以将 VM 映像部署到虚拟机缓存的大小。
- 能够快速重置或重置到原始的启动状态其虚拟机的映像。  
- 较低的运行时延迟类似于临时磁盘。 
- OS 磁盘不收费。 
 
 
持久性和临时 OS 磁盘之间的主要差异：

|                             | 持久性 OS 磁盘                          | 临时 OS 磁盘                              |    |
|-----------------------------|---------------------------------------------|------------------------------------------------|
| OS 磁盘的大小限制      | 2 个 TiB                                                                                        | 缓存大小的 VM 大小或 2TiB，两者中较小- [DS](../articles/virtual-machines/linux/sizes-general.md)， [ES](../articles/virtual-machines/linux/sizes-memory.md)， [M](../articles/virtual-machines/linux/sizes-memory.md)， [FS](../articles/virtual-machines/linux/sizes-compute.md)，和[GS](../articles/virtual-machines/linux/sizes-memory.md)              |
| 支持的 VM 大小          | 全部                                                                                          | DSv1、 DSv2、 DSv3、 esv3 系列、 Fs、 FsV2、 GS、 M                                               |
| 磁盘类型支持           | 托管和非托管 OS 磁盘                                                                | 托管的 OS 磁盘                                                               |
| 区域支持              | 所有区域                                                                                  | 所有区域                              |
| 数据暂留            | 写入到 OS 磁盘的 OS 磁盘数据存储在 Azure 存储                                  | 写入到 OS 磁盘的数据存储到的本地 VM 存储并不会保留到 Azure 存储。 |
| 停止解除分配状态      | 可以停止解除分配，并从已停止解除分配状态重新启动 Vm 和规模集实例 | Vm 和规模集实例不能为已停止解除分配                                  |
| 专用的 OS 磁盘支持 | “是”                                                                                          | “否”                                                                                 |
| OS 磁盘重设大小              | 支持的 VM 创建期间和之后 VM 处于停止解除分配                                | 仅在创建 VM 期间支持                                                  |
| 为新的 VM 大小重设大小   | OS 磁盘数据保留                                                                    | OS 磁盘上的数据之后，操作系统是重新预配                                      |

## <a name="register-for-the-preview"></a>注册预览版


自注册使用 Azure CLI 或 Azure PowerShell 的最新版本的临时 OS 磁盘的预览版。

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
Register-AzProviderFeature –FeatureName LocalDiffDiskPreview -ProviderNamespace Microsoft.Compute
```

若要检查你注册预览版：

```azurepowershell-interactive
Get-AzProviderFeature –FeatureName LocalDiffDiskPreview -ProviderNamespace Microsoft.Compute
```

### <a name="cli"></a>CLI

```azurecli-interactive
az provider register --namespace Microsoft.Compute
az feature register --namespace Microsoft.Compute --name LocalDiffDiskPreview
```

若要检查你注册预览版：
 
```azurecli-interactive
az provider show --namespace Microsoft.Compute
```


## <a name="scale-set-deployment"></a>规模集部署  
创建使用临时 OS 磁盘的规模集的过程是将添加`diffDiskSettings`属性设置为`Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile`模板中的资源类型。 此外，缓存策略必须设置为`ReadOnly`临时 OS 磁盘。 


```json
{ 
  "type": "Microsoft.Compute/virtualMachineScaleSets", 
  "name": "myScaleSet", 
  "location": "East US 2", 
  "apiVersion": "2018-06-01", 
  "sku": { 
    "name": "Standard_DS2_v2", 
    "capacity": "2" 
  }, 
  "properties": { 
    "upgradePolicy": { 
      "mode": "Automatic" 
    }, 
    "virtualMachineProfile": { 
       "storageProfile": { 
        "osDisk": { 
          "diffDiskSettings": { 
                "option": "Local" 
          }, 
          "caching": "ReadOnly", 
          "createOption": "FromImage" 
        }, 
        "imageReference":  { 
          "publisher": "Canonical", 
          "offer": "UbuntuServer", 
          "sku": "16.04-LTS", 
          "version": "latest" 
        } 
      }, 
      "osProfile": { 
        "computerNamePrefix": "myvmss", 
        "adminUsername": "azureuser", 
        "adminPassword": "P@ssw0rd!" 
      } 
    } 
  } 
}  
```

## <a name="vm-deployment"></a>VM 部署 
可以使用临时 OS 磁盘使用模板部署 VM。 若要创建使用临时 OS 磁盘的 VM 的过程是添加`diffDiskSettings`模板中的 microsoft.compute/virtualmachines 资源类型的属性。 此外，缓存策略必须设置为`ReadOnly`临时 OS 磁盘。 

```json
{ 
  "type": "Microsoft.Compute/virtualMachines", 
  "name": "myVirtualMachine", 
  "location": "East US 2", 
  "apiVersion": "2018-06-01", 
  "properties": { 
       "storageProfile": { 
            "osDisk": { 
              "diffDiskSettings": { 
                "option": "Local" 
              }, 
              "caching": "ReadOnly", 
              "createOption": "FromImage" 
            }, 
            "imageReference": { 
                "publisher": "MicrosoftWindowsServer", 
                "offer": "WindowsServer", 
                "sku": "2016-Datacenter-smalldisk", 
                "version": "latest" 
            }, 
            "hardwareProfile": { 
                 "vmSize": "Standard_DS2_v2" 
             } 
      }, 
      "osProfile": { 
        "computerNamePrefix": "myvirtualmachine", 
        "adminUsername": "azureuser", 
        "adminPassword": "P@ssw0rd!" 
      } 
    } 
 } 
```


## <a name="reimage-a-vm-using-rest"></a>重置映像虚拟机使用 REST
目前，若要重置映像使用临时 OS 磁盘的虚拟机实例的唯一方法是通过使用 REST API。 对于规模集，正在重置映像已可通过 Powershell、 CLI 和门户。

```
POST https://management.azure.com/subscriptions/{sub-
id}/resourceGroups/{rgName}/providers/Microsoft.Compute/VirtualMachines/{vmName}/reimage?a pi-version=2018-06-01" 
```
 
## <a name="frequently-asked-questions"></a>常见问题

**问：本地 OS 磁盘大小是多少？**

答：对于预览版，我们将支持的平台和/或最大 VM 缓存大小，将在其中放置虚拟机作为在同一节点上本地所有读/写到 OS 磁盘映像。 

**问：可以调整大小临时 OS 磁盘？**

答：否，临时 OS 磁盘预配后，不能调整 OS 磁盘。 

**问：可以将托管磁盘附加到临时 VM？**

答：是的可以将托管的数据磁盘附加到使用临时 OS 磁盘的 VM。 

**问：将所有 VM 大小都支持临时 OS 磁盘？**

答：不能，所有高级存储 VM 大小支持 （DS、 ES、 FS、 GS 和 M） 除外 B 系列、 N 系列和 H 系列大小。  
 
**问：可以临时 OS 磁盘应用到现有的 Vm 和规模集？**

答：不会，临时 OS 磁盘只能使用 vm 和规模集创建。 

**问：可以混合使用临时并处于正常状态在规模集中的 OS 磁盘？**

答：否，不能具有相同的规模集内的临时和永久 OS 磁盘实例的组合。 

**问：可以使用 Powershell 或 CLI 创建临时 OS 磁盘？**

答：是的您可以使用临时 OS 磁盘使用 REST、 模板、 PowerShell 和 CLI 创建 Vm。

**问：使用临时 OS 磁盘不支持哪些功能？**

答：不支持临时磁盘：
- 捕获 VM 映像
- 磁盘快照 
- Azure 磁盘加密 
- Azure 备份
- Azure Site Recovery  
- OS 磁盘交换 
