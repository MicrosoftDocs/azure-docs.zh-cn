---
title: 使用 Azure Kubernetes 服务 (AKS) 中的群集自动缩放程序
description: 了解如何使用群集自动缩放程序自动缩放群集以满足 Azure Kubernetes 服务 (AKS) 群集中的应用程序需求。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 01/29/2019
ms.author: iainfou
ms.openlocfilehash: d5a287a8da884290e94e9ac1c864abe28e47d53d
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508143"
---
# <a name="preview---automatically-scale-a-cluster-to-meet-application-demands-on-azure-kubernetes-service-aks"></a>预览-自动缩放以满足应用程序的需求在 Azure Kubernetes 服务 (AKS) 群集

若要满足 Azure Kubernetes 服务 (AKS) 中的应用程序需求，可能需要调整运行工作负载的节点数。 群集自动缩放程序组件可以监视群集中由于资源约束而无法进行计划的 Pod。 检测到问题时，节点数会增加，以满足应用程序需求。 还会定期检查节点是否缺少正在运行的 Pod，随后根据需要减少节点数。 这种自动增加或减少 AKS 群集中的节点数的功能使你可以运行具有成本效益的高效群集。

本文演示如何在 AKS 群集中启用和管理群集自动缩放程序。 群集自动缩放程序只应在单个节点池与 AKS 群集上的预览版中进行测试。

> [!IMPORTANT]
> AKS 预览功能是自助服务和可以选择加入的功能。 提供预览是为了从我们的社区收集反馈和 bug。 但是，Azure 技术支持部门不为其提供支持。 如果你创建一个群集，或者将这些功能添加到现有群集，则除非该功能不再为预览版并升级为公开发布版 (GA)，否则该群集不会获得支持。
>
> 如果遇到预览版功能的问题，请[在 AKS GitHub 存储库中提交问题][aks-github]，并在 Bug 标题中填写预览版功能的名称。

## <a name="before-you-begin"></a>开始之前

本文要求运行 Azure CLI 2.0.55 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

### <a name="install-aks-preview-cli-extension"></a>安装 aks-preview CLI 扩展

支持群集自动缩放程序的 AKS 群集必须使用虚拟机规模集并运行 Kubernetes 版本 1.12.4或更高版本。 此规模集支持处于预览状态。 若要选择加入并创建使用规模集的群集，请先使用 [az extension add][az-extension-add] 命令安装 aks-preview Azure CLI 扩展，如下面的示例中所示：

```azurecli-interactive
az extension add --name aks-preview
```

> [!NOTE]
> 如果你以前已安装*aks 预览版*扩展，安装任何可用更新使用`az extension update --name aks-preview`命令。

### <a name="register-scale-set-feature-provider"></a>注册规模集功能提供程序

要创建使用规模集的 AKS，还必须在订阅上启用功能标志。 若要注册 VMSSPreview 功能标志，请使用 [az feature register][az-feature-register] 命令，如以下示例所示：

```azurecli-interactive
az feature register --name VMSSPreview --namespace Microsoft.ContainerService
```

状态显示为“已注册”需要几分钟时间。 可以使用 [az feature list][az-feature-list] 命令检查注册状态：

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/VMSSPreview')].{Name:name,State:properties.state}"
```

准备就绪后，使用 [az provider register][az-provider-register] 命令刷新 Microsoft.ContainerService 资源提供程序的注册状态：

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="limitations"></a>限制

创建和管理 AKS 群集中使用虚拟机规模集时，应考虑以下限制：

* 不能使用 HTTP 应用程序路由外接程序。

## <a name="about-the-cluster-autoscaler"></a>关于群集自动缩放程序

若要进行调整以适应不断变化的应用程序需求（如工作日与夜间或周末之间），群集通常需要一种自动缩放方式。 AKS 群集可以采用两种方式之一进行缩放：

* 群集自动缩放程序会监视由于资源约束而无法在节点上计划的 Pod。 群集随后会自动增加节点数。
* 水平 Pod 自动缩放程序会在 Kubernetes 群集中使用指标服务器来监视 Pod 的资源需求。 如果服务需要更多资源，则会自动增加 Pod 数以满足需求。

![群集自动缩放程序和水平 Pod 自动缩放程序通常协同工作以支持所需的应用程序需求](media/autoscaler/cluster-autoscaler.png)

水平 Pod 自动缩放程序和群集自动缩放程序随后还可以根据需要减少 Pod 和节点数。 当已有一段时间存在未使用的容量时，群集自动缩放程序会减少节点数。 要由群集自动缩放程序删除的节点上的 Pod 会在群集中的其他位置安全地进行计划。 如果无法移动 Pod，则群集自动缩放程序可能无法减少，如以下情况：

* Pod 直接创建，不由控制器对象（如部署或副本集）提供支持。
* Pod 中断预算 (PDB) 限制太多，不允许 Pod 数低于特定阈值。
* Pod 使用在不同节点上进行计划时无法遵循的节点选择器或反相关性。

有关群集自动缩放程序如何可能无法减少的详细信息，请参阅[哪些类型的 Pod 可能会阻止群集自动缩放程序删除节点？][autoscaler-scaledown]

群集自动缩放程序对诸如缩放事件与资源阈值之间的时间间隔等内容使用启动参数。 这些参数由 Azure 平台订阅，当前并未公开以供你调整。 有关群集自动缩放程序使用的参数的详细信息，请参阅[群集自动缩放程序参数是什么？][autoscaler-parameters]。

两个自动缩放程序可以协同工作，通常部署在一个群集中。 结合使用时，水平 Pod 自动缩放程序侧重于运行满足应用程序需求所需的 Pod 数。 群集自动缩放程序侧重于运行支持计划 Pod 所需的节点数。

> [!NOTE]
> 使用群集自动缩放程序时，会禁用手动缩放。 请让群集自动缩放程序确定所需的节点数。 如果要手动缩放群集，则[禁用群集自动缩放程序](#disable-the-cluster-autoscaler)。

## <a name="create-an-aks-cluster-and-enable-the-cluster-autoscaler"></a>创建 AKS 群集并启用群集自动缩放程序

如果需要创建 AKS 群集，请使用 [az aks create][az-aks-create] 命令。 指定的 --kubernetes-version 应满足或超过在前面[开始之前](#before-you-begin)部分中概述最低版本号。 若要启用和配置群集自动缩放程序，请使用 --enable-cluster-autoscaler参数，并指定节点 --min-count 和 --max-count。

> [!IMPORTANT]
> 群集自动缩放程序是 Kubernetes 组件。 虽然 AKS 群集对节点使用虚拟机规模集，但请勿在 Azure 门户中或使用 Azure CLI 手动启用或编辑规模集自动缩放设置。 让 Kubernetes 群集自动缩放程序管理所需的规模设置。 有关详细信息，请参阅[我可以修改 MC_ 资源组中的 AKS 资源吗？](faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-mc_-resource-group)

以下示例创建启用了虚拟机规模集和群集自动缩放程序的 AKS 群集，并使用最少 1 个且最多 3 个节点：

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location canadaeast

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --enable-vmss \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

创建群集并配置群集自动缩放程序设置需要几分钟时间。

### <a name="enable-the-cluster-autoscaler-on-an-existing-aks-cluster"></a>在现有 AKS 群集上启用群集自动缩放程序

可以在满足前面[开始之前](#before-you-begin)部分中所概述要求的现有 AKS 群集上启用群集自动缩放程序。 使用 [az aks update][az-aks-update] 命令并选择 --enable-cluster-autoscaler，然后指定节点 --min-count 和 --max-count。 下面的示例在使用最少 1 个且最多 3 个节点的现有群集上启用群集自动缩放程序：

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

如果最小节点计数大于群集中的现有节点数，则创建附加节点需要花费几分钟时间。

## <a name="change-the-cluster-autoscaler-settings"></a>更改群集自动缩放程序设置

在创建或更新现有 AKS 群集的上一步中，群集自动缩放程序最小节点计数设置为 1，且最大节点计数设置为 3。 随着应用程序需求发生变化，可能需要调整群集自动缩放程序节点计数。

若要更改节点计数，请使用 [az aks update][az-aks-update] 命令并指定最小值和最大值。 下面的示例将 --min-count 设置为 1，并将 --max-count 设置为 5：

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --update-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

> [!NOTE]
> 在预览版期间，设置的最小节点计数不能高于当前为群集设置的计数。 例如，如果当前将最小计数设置为 1，则不能将最小计数更新为 3。

监视应用程序和服务的性能，并调整群集自动缩放程序节点计数以匹配所需性能。

## <a name="disable-the-cluster-autoscaler"></a>禁用群集自动缩放程序

如果不再希望使用群集自动缩放程序，则可以使用 [az aks update][az-aks-update] 命令禁用它。 群集自动缩放程序处于禁用状态时，不会删除节点。

若要删除群集自动缩放程序，请指定 --disable-cluster-autoscaler 参数，如下面的示例所示：

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --disable-cluster-autoscaler
```

可以使用 [az aks scale][az-aks-scale] 命令手动缩放群集。 如果使用水平 Pod 自动缩放程序，则该功能会在群集自动缩放程序禁用的情况下继续运行，但 Pod 可能会在节点资源全部已使用时最终无法进行计划。

## <a name="next-steps"></a>后续步骤

本文演示了如何自动缩放 AKS 节点数。 还可以使用水平 Pod 自动缩放程序自动调整运行应用程序的 Pod 数。 有关使用水平 Pod 自动缩放程序的步骤，请参阅[在 AKS 中缩放应用程序][aks-scale-apps]。

<!-- LINKS - internal -->
[aks-upgrade]: upgrade-cluster.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-scale-apps]: tutorial-kubernetes-scale.md
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[aks-github]: https://github.com/azure/aks/issues

<!-- LINKS - external -->
[az-aks-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
