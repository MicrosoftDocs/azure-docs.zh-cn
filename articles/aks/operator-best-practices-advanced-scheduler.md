---
title: 有关调度器功能的最佳做法
titleSuffix: Azure Kubernetes Service
description: 了解有关使用 Azure Kubernetes 服务 (AKS) 中的高级调度器功能（例如污点 (taint) 和容忍度 (toleration)、节点选择器和亲和性，或 pod 间亲和性和反亲和性）的群集操作员最佳做法
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: 1a8138b4b2fdab2cdef8d2cb4c27de8d12ef38cd
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107340"
---
# <a name="best-practices-for-advanced-scheduler-features-in-azure-kubernetes-service-aks"></a>有关 Azure Kubernetes 服务 (AKS) 中的高级调度器功能的最佳做法

在 Azure Kubernetes 服务 (AKS) 中管理群集时，通常需要隔离团队和工作负荷。 Kubernetes 调度器提供高级功能，让你控制可在特定节点上调度哪些 Pod，或者如何在整个群集中适当地分配多 Pod 应用程序。 

本最佳做法文章重点介绍面向群集操作员的高级 Kubernetes 调度功能。 在本文中，学习如何：

> [!div class="checklist"]
> * 使用污点和容忍度来限制可在节点上调度的 pod
> * 使用节点选择器或节点亲和性为特定节点上运行的 pod 分配优先顺序
> * 使用 pod 间亲和性或反亲和性来拆离或组合 pod

## <a name="provide-dedicated-nodes-using-taints-and-tolerations"></a>使用污点和容忍度提供专用节点

**最佳做法指导** - 限制资源密集型应用程序（例如入口控制器）对特定节点的访问。 将节点资源保留给需要它们的工作负荷使用，但不允许在节点上调度其他工作负荷。

创建 AKS 群集时，可以部署支持 GPU 的节点或具有大量强大 CPU 的节点。 这些节点通常用于大数据处理工作负荷，例如机器学习 (ML) 或人工智能 (AI)。 由于此类硬件通常是需要部署的昂贵节点资源，因此需要限制可在这些节点上调度的工作负荷。 你可能想要专门使用群集中的某些节点来运行入口服务，并阻止其他工作负荷。

这种对不同节点的支持通过使用多个节点池来提供。 AKS 群集提供一个或多个节点池。

Kubernetes 调度器能够使用污点和容忍度来限制可在节点上运行的工作负荷。

* 将 **污点** 应用到指明了只能调度特定 pod 的节点。
* 然后，将 **容忍度** 应用到可以 *容忍* 节点污点的 pod。

将 pod 部署到 AKS 群集时，Kubernetes 只会在容忍度与污点相符的节点上调度 pod。 例如，假设你在 AKS 群集中为支持 GPU 的节点创建了一个节点池。 你定义了名称（例如 *gpu*），然后定义了调度值。 如果将此值设置为 *NoSchedule*，当 pod 未定义相应的容忍度时，Kubernetes 调度器无法在节点上调度 pod。

```console
kubectl taint node aks-nodepool1 sku=gpu:NoSchedule
```

将污点应用到节点后，在 pod 规范中定义容忍度，以允许在节点上进行调度。 以下示例定义 `sku: gpu` 和 `effect: NoSchedule`，以容忍度在上一步骤中应用到节点的污点：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

部署此 pod（例如，使用 `kubectl apply -f gpu-toleration.yaml`）后，Kubernetes 可以成功地在应用了污点的节点上调度 pod。 通过这种逻辑隔离，可以控制对群集中资源的访问。

应用污点时，请与应用程序开发人员和所有者协作，让他们在其部署中定义所需的容忍度。

若要详细了解如何在 AKS 中使用多个节点池，请参阅[为 AKS 中的群集创建和管理多个节点池][use-multiple-node-pools]。

### <a name="behavior-of-taints-and-tolerations-in-aks"></a>AKS 中的污点和容忍度的行为

升级 AKS 中的节点池时，污点和容忍度在应用于新节点时遵循一个设定的模式：

- **使用虚拟机规模集的默认群集**
  - 可以从 AKS API [污染节点池][taint-node-pool]，以使新横向扩展的节点接收 API 指定的节点污点。
  - 假设你的群集有两个节点 - *node1* 和 *node2*。 升级节点池。
  - 另外两个节点（node3 和 node4）将被创建，并且污点会被分别传递。
  - 原始 node1 和 node2 将被删除。

- **不支持虚拟机规模集的群集**
  - 同样，让我们假设你有一个双节点群集 - node1 和 node2。 在升级时，将创建另一个节点 (*node3*)。
  - *node1* 中的污点将应用于 *node3*，然后 *node1* 将被删除。
  - 将创建另一个新节点（名为 *node1*，因为以前的 *node1* 被删除），并且 *node2* 污点将应用于新的 *node1*。 然后，将删除 *node2*。
  - 实际上，*node1* 变成了 *node3*，*node2* 变成了 *node1*。

缩放 AKS 中的节点池时，污点和容忍度不会转移，这是设计使然。

## <a name="control-pod-scheduling-using-node-selectors-and-affinity"></a>使用节点选择器和亲和性控制 pod 调度

**最佳做法指导** - 使用节点选择器、节点亲和性或 pod 间亲和性来控制节点上的 pod 调度。 这些设置可让 Kubernetes 调度器以逻辑方式（例如，按节点中的硬件）隔离工作负荷。

使用污点和容忍度能够以硬分割的形式逻辑隔离资源 - 如果 pod 不容忍度某个节点的污点，则不会在该节点上调度该 pod。 另一种方法是使用节点选择器。 例如，可以标记节点来指示本地附加的 SSD 存储或大量内存，并在 pod 规范中定义节点选择器。 然后，Kubernetes 会在匹配的节点上调度这些 pod。 与容忍度不同，没有匹配的节点选择器的 pod 可在标记的节点上调度。 通过这种行为可以使用节点上未用的资源，但会将优先级分配给定义匹配的节点选择器的 pod。

让我们查看具有大量内存的节点示例。 这些节点可向请求大量内存的 pod 分配优先顺序。 为确保资源不会闲置，它们还允许运行其他 pod。

```console
kubectl label node aks-nodepool1 hardware=highmem
```

然后，pod 规范添加 `nodeSelector` 属性，以定义与节点上设置的标签匹配的节点选择器：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  nodeSelector:
      hardware: highmem
```

使用这些调度器选项时，请与应用程序开发人员和所有者协作，让他们正确定义其 pod 规范。

有关使用节点选择器的详细信息，请参阅[将 Pod 分配到节点][k8s-node-selector]。

### <a name="node-affinity"></a>节点亲和性

节点选择器是将 pod 分配到给定节点的基本方法。 使用节点亲和性可以获得更高的灵活性。 使用节点亲和性可以定义当 pod 无法与节点匹配时发生的情况。 可以要求 Kubernetes 调度器与包含标记主机的 pod 相匹配。 或者，如果“required”匹配项不可用， 则可以选择“*prefer*” 匹配项以允许在其他主机上调度 pod。

以下示例将节点亲和性设置为 *requiredDuringSchedulingIgnoredDuringExecution*。 这种亲和性要求 Kubernetes 调度使用具有匹配标签的节点。 如果没有可用的节点，则 pod 必须等待调度继续。 若要允许在其他节点上调度 pod，可改为将值设置为 preferredDuringSchedulingIgnoreDuringExecution：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values: highmem
```

该设置的 *IgnoredDuringExecution* 部分表示当节点标签更改时，不应从节点中逐出 pod。 Kubernetes 调度器仅对所要调度的新 pod 使用更新的节点标签，对于已在节点上调度的 pod 则不使用。

有关详细信息，请参阅[亲和性和反亲和性][k8s-affinity]。

### <a name="inter-pod-affinity-and-anti-affinity"></a>pod 间亲和性和反亲和性

Kubernetes 调度器逻辑隔离工作负荷的最终方法之一是使用 pod 间亲和性或反亲和性。 这些设置定义不应在具有现有匹配 pod 的节点上调度 pod，或者应该调度 pod。  默认情况下，Kubernetes 调度器会尝试在跨节点的副本集3中调度多个 pod。 可围绕此行为定义更具体的规则。

同时使用 Azure Redis 缓存的 Web 应用程序就是一个很好的例子。 可以使用 pod 反亲和性规则来请求 Kubernetes 调度器跨节点分配副本。 然后，可以使用亲和性规则来确保在相应缓存所在的同一主机上调度每个 Web 应用组件。 跨节点的 pod 分配如以下示例所示：

| **节点 1** | **节点 2** | **节点 3** |
|------------|------------|------------|
| webapp-1   | webapp-2   | webapp-3   |
| cache-1    | cache-2    | cache-3    |

与使用节点选择器或节点亲和性相比，此示例是一种更复杂的部署。 部署可让你控制 Kubernetes 如何在节点上调度 pod，并可以逻辑隔离资源。 有关这个使用 Azure Cache for Redis 的 Web 应用程序的完整示例，请参阅[在同一节点上共置 Pod][k8s-pod-affinity]。

## <a name="next-steps"></a>后续步骤

本文重点介绍了高级 Kubernetes 调度器功能。 有关 AKS 中的群集操作的详细信息，请参阅以下最佳做法：

* [多租户和群集隔离][aks-best-practices-scheduler]
* [基本 Kubernetes 调度器功能][aks-best-practices-scheduler]
* [身份验证和授权][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-taints-tolerations]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[k8s-node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[k8s-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
[k8s-pod-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#always-co-located-in-the-same-node

<!-- INTERNAL LINKS -->
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[taint-node-pool]: use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool
