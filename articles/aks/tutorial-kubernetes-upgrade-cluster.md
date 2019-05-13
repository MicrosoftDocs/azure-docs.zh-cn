---
title: Azure 上的 Kubernetes 教程 - 升级群集
description: 此 Azure Kubernetes 服务 (AKS) 教程介绍如何将现有 AKS 群集升级到最新可用的 Kubernetes 版本。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: tutorial
ms.date: 12/19/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: b0d0d8326d6274252f4c4a865bc8f022daf9e199
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61029345"
---
# <a name="tutorial-upgrade-kubernetes-in-azure-kubernetes-service-aks"></a>教程：在 Azure Kubernetes 服务 (AKS) 中升级 Kubernetes

在应用程序和群集生命周期中，你可能希望升级到最新可用的 Kubernetes 版本并使用新功能。 可使用 Azure CLI 升级 Azure Kubernetes 服务 (AKS) 群集。

在本教程的第 7 部分中，升级了 Kubernetes 群集。 学习如何：

> [!div class="checklist"]
> * 确定 Kubernetes 的当前版本和可用版本
> * 升级 Kubernetes 节点
> * 验证升级是否成功

## <a name="before-you-begin"></a>开始之前

上一教程中，应用程序已打包到容器映像中。 该映像已上传到 Azure容器注册表，同时，你创建了 AKS 群集。 然后，应用程序部署到了 AKS 群集。 如果尚未完成这些步骤，并且想要逐一完成，请开始参阅[教程 1 – 创建容器映像][aks-tutorial-prepare-app]。

本教程需要运行 Azure CLI 2.0.53 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="get-available-cluster-versions"></a>获取可用的群集版本

在升级群集之前，请使用 [az aks get-upgrades][] 命令检查哪些 Kubernetes 版本可用于升级：

```azurecli
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

在以下示例中，当前版本为 1.9.11，可用的版本显示在“升级”列下。

```
Name     ResourceGroup    MasterVersion    NodePoolVersion    Upgrades
-------  ---------------  ---------------  -----------------  --------------
default  myResourceGroup  1.9.11           1.9.11             1.10.8, 1.10.9
```

## <a name="upgrade-a-cluster"></a>升级群集

为尽量减少对正在运行的应用程序造成中断，AKS 节点已进行仔细隔离和排空。 在此过程中，执行以下步骤：

1. Kubernetes 计划程序阻止在要升级的节点上计划其他 Pod。
1. 计划在群集中的其他节点上运行该节点上运行的 Pod。
1. 创建运行最新 Kubernetes 组件的节点。
1. 准备好新节点并将其加入群集后，Kubernetes 计划程序开始在该群集上运行 Pod。
1. 删除旧节点，群集的下一个节点随即开始隔离和排空进程。

使用 [az aks upgrade][] 命令升级 AKS 群集。 以下示例将群集升级到 Kubernetes 版本 1.10.9。

> [!NOTE]
> 一次只能升级一个次要版本。 例如，可以从 1.9.11 升级到 1.10.9，但不能从 1.9.6 直接升级到 1.11.x。 若要从 1.9.11 升级到 1.11.x，请先从 1.9.11 升级到 1.10.x，然后再执行一次升级从 1.10.x 升级到 1.11.x。

```azurecli
az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version 1.10.9
```

以下浓缩版的示例输出显示 kubernetesVersion 现在报告 1.10.9：

```json
{
  "agentPoolProfiles": [
    {
      "count": 3,
      "maxPods": 110,
      "name": "nodepool1",
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS1_v2",
    }
  ],
  "dnsPrefix": "myAKSClust-myResourceGroup-19da35",
  "enableRbac": false,
  "fqdn": "myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io",
  "id": "/subscriptions/<Subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "kubernetesVersion": "1.10.9",
  "location": "eastus",
  "name": "myAKSCluster",
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="validate-an-upgrade"></a>验证升级

使用 [az aks show][] 命令确认升级成功，如下所示：

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

以下示例输出显示 AKS 群集运行 KubernetesVersion 1.10.9：

```
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.10.9               Succeeded            myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io
```

## <a name="delete-the-cluster"></a>删除群集

由于此教程是该系列的最后一部分，因此可能需要删除 AKS 群集。 当 Kubernetes 节点在 Azure 虚拟机 (VM) 上运行时，即使你不使用群集，它们也会继续产生费用。 可以使用 [az group delete][az-group-delete] 命令删除资源组、容器服务及所有相关资源。

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> 删除群集时，AKS 群集使用的 Azure Active Directory 服务主体不会被删除。 有关如何删除服务主体的步骤，请参阅 [AKS 服务主体的注意事项和删除][sp-delete]。

## <a name="next-steps"></a>后续步骤

在本教程中，在 AKS 群集中升级了 Kubernetes。 你已了解如何：

> [!div class="checklist"]
> * 确定 Kubernetes 的当前版本和可用版本
> * 升级 Kubernetes 节点
> * 验证升级是否成功

请单击以下链接了解有关 AKS 的详细信息。

> [!div class="nextstepaction"]
> [AKS 概述][aks-intro]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-intro]: ./intro-kubernetes.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az aks show]: /cli/azure/aks#az-aks-show
[az aks get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az aks upgrade]: /cli/azure/aks#az-aks-upgrade
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-delete]: /cli/azure/group#az-group-delete
[sp-delete]: kubernetes-service-principal.md#additional-considerations
