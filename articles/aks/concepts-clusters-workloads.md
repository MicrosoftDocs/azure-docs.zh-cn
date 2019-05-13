---
title: 概念 - Azure Kubernetes Service (AKS) 中 Kubernetes 的核心概念
description: 了解 Kubernetes 的基本群集和工作负荷组件以及它们与 Azure Kubernetes 服务 (AKS) 中各个功能的关系
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 02/28/2019
ms.author: iainfou
ms.openlocfilehash: faac0f02d1a1b8927fa0c651f44f8b120a583d9a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230148"
---
# <a name="kubernetes-core-concepts-for-azure-kubernetes-service-aks"></a>Azure Kubernetes 服务 (AKS) 的 Kubernetes 核心概念

当应用程序开发向基于容器的开发转变时，资源的安排和管理就变得非常重要。 Kubernetes 是一个可用于可靠地规划和调度容错的应用程序工作负载的领先平台。 Azure Kubernetes 服务 (AKS) 是一种托管的 Kubernetes 服务，可进一步简化基于容器的应用程序的部署和管理。

本文介绍了核心 Kubernetes 基础结构组件，例如集群主机、节点和节点池。 还介绍了 Pod、部署和集等工作负荷资源，以及如何将资源分组到命名空间。

## <a name="what-is-kubernetes"></a>什么是 Kubernetes？

Kubernetes 是一个快速发展的平台，用于管理基于容器的应用程序及其相关网络和存储组件。 重点关注应用程序工作负荷，而不是底层基础结构组件。 Kubernetes 提供了一种声明性的部署方法，由一组针对管理操作的强大 API 提供支持。

你可以生成和运行可移植的、基于微服务的现代应用程序，这些应用程序可通过 Kubernetes 安排和管理这些应用程序组件的可用性而受益。 由于团队通过采用基于微服务的应用程序而取得进展，因此 Kubernetes 支持无状态和有状态应用程序。

作为开放平台，Kubernetes 可使用首选的编程语言、OS、库或消息总线生成应用程序。 现有的持续集成和持续交付 (CI/CD) 工具可以与 Kubernetes 集成，以计划和部署版本。

Azure Kubernetes 服务 (AKS) 提供托管 Kubernetes 服务，可简化部署和核心管理任务，包括协调升级。 Azure 平台可托管 AKS 群集主机，你只需为运行应用程序的 AKS 节点付费。 AKS 基于开源 Azure Kubernetes 服务引擎 ([aks 引擎][aks-engine])。

## <a name="kubernetes-cluster-architecture"></a>Kubernetes 群集体系结构

Kubernetes 群集分为两个组件：

- 群集主机节点提供核心 Kubernetes 服务和应用程序工作负荷的业务流程。
- 节点运行应用程序工作负荷。

![Kubernetes 群集主机和节点组件](media/concepts-clusters-workloads/cluster-master-and-nodes.png)

## <a name="cluster-master"></a>群集主机

创建 AKS 群集时，系统会自动创建和配置群集主机。 此群集主机作为从用户抽象出来的托管 Azure 资源提供。 没有任何群集主机费用，只有属于 AKS 群集的节点的费用。

群集主机包括以下核心 Kubernetes 组件：

- kube-apiserver - API 服务器，用于公开基础 Kubernetes API。 此组件为管理工具（如 `kubectl` 或 Kubernetes 仪表板）提供交互。
- etcd - 可维护 Kubernetes 群集和配置的状态，高可用性 etcd 是 Kubernetes 中的键值存储。
- kube-scheduler - 创建或缩放应用程序时，计划程序可确定哪些节点可以运行工作负荷并启动这些节点。
- kube-controller-manager - 控制器管理器可监视许多较小的控制器，这些控制器执行 Pod 复制和节点处理等操作。

AKS 提供单租户群集主和专用 API 服务器，计划程序等。由你来定义节点的数量和大小，Azure 平台配置群集主机与节点之间的安全通信。 通过 Kubernetes API（例如 `kubectl` 或 Kubernetes 仪表板）实现与群集主机之间的交互。

此托管的群集主机意味着无需配置某些组件，如高度可用的*etcd*存储区，但也意味着不能直接访问群集主机。 通过 Azure CLI 或 Azure 门户安排 Kubernetes 升级，先升级群集主机，然后升级节点。 要解决可能出现的问题，可以通过 Azure Monitor 日志查看群集主服务器日志。

如果需要以特定方式配置群集主机或直接对其进行访问，可以使用 [aks-engine][aks-engine] 部署自己的 Kubernetes 群集。

有关相关的最佳做法，请参阅[AKS 中的安全性和升级最佳做法][operator-best-practices-cluster-security]。

## <a name="nodes-and-node-pools"></a>节点和节点池

要运行应用程序和支持服务，需要 Kubernetes 节点。 一个 AKS 群集有一个或多个节点，节点是运行 Kubernetes 节点组件和容器运行时的 Azure 虚拟机 (VM)：

- `kubelet` 是 Kubernetes 代理，用于处理来自群集主机的业务流程请求并规划请求的容器的运行。
- 虚拟网络由每个节点上的 kube-proxy 处理。 代理路由流量并管理服务和 Pod 的 IP 地址。
- 容器运行时是允许容器化应用程序运行并与其他资源（如虚拟网络和存储）进行交互的组件。 在 AKS 中，Moby 用作容器运行时。

![Azure 虚拟机和 Kubernetes 节点的支持资源](media/concepts-clusters-workloads/aks-node-resource-interactions.png)

节点的 Azure VM 大小定义了 CPU 数量、内存大小以及可用存储的大小和类型（如高性能 SSD 或常规 HDD）。 如果预计需要使用大量 CPU 和内存或高性能存储的应用程序，则相应地规划节点大小。 还可以纵向扩展 AKS 群集中的节点数以满足需求。

在 AKS 中，群集中节点的 VM 映像当前基于 Ubuntu Linux。 创建 AKS 群集或纵向扩展节点数时，Azure 平台会创建所请求数量的 VM 并对其进行配置。 无需任何手动配置。

如果需要使用不同的主机 OS、容器运行时或包含自定义程序包，可以使用 [aks-engine][aks-engine] 部署自己的 Kubernetes 群集。 上游 `aks-engine` 会发布功能并提供配置选项，且这一操作会在 AKS 群集中支持这些操作和选项之前实施。 例如，如果想要使用 Windows 容器或容器运行时而不是 Moby，可以使用`aks-engine`配置和部署满足当前需求的 Kubernetes 群集。

### <a name="resource-reservations"></a>资源预留

你不需要在每个节点上管理核心 Kubernetes 组件（例如 *kubelet*、*kube-proxy* 和 *kube-dns*），但它们确实消耗某些可用的计算资源。 为保持节点性能和功能，每个节点上会预留以下计算资源：

- **CPU** - 60 ms
- **内存** - 20%，最多 4 GiB

这些预留意味着你的应用程序的可用 CPU 和内存量可能显示为少于节点本身包含的数量。 如果由于你运行的应用程序数太多而存在资源约束，则这些预留可以确保 CPU 和内存保持可供核心 Kubernetes 组件使用。 资源预留无法更改。

例如：

- **标准 DS2 v2** 节点大小包含 2 个 vCPU 和 7 GiB 内存
    - 7 GiB 内存的 20% = 1.4 GiB
    - 总共有 *(7 - 1.4) = 5.6 GiB* 内存可供节点使用
    
- **标准 E4s v3** 节点大小包含 4 个 vCPU 和 32 GiB 内存
    - 32 GiB 内存的 20% = 6.4 GiB，但 AKS 最多仅保留 4 GiB
    - 总共有 *(32 - 4) = 28 GiB* 内存可供节点使用
    
基础节点 OS 还需要一定量的 CPU 和内存资源来完成其自己的核心功能。

有关相关的最佳做法，请参阅[在 AKS 中的基本计划程序功能的最佳做法][operator-best-practices-scheduler]。

### <a name="node-pools"></a>节点池

具有相同配置的节点将统一合并成节点池。 Kubernetes 群集包含一个或多个节点池。 创建 AKS 群集时会定义初始节点数和大小，从而创建默认节点池。 AKS 中的此默认节点池包含运行代理节点的基础 VM。 多个节点池支持目前处于预览状态，在 AKS 中。

缩放或升级 AKS 群集时，将对默认节点池执行操作。 您还可以选择缩放或升级的特定节点池。 对于升级操作，将在节点池中的其他节点上计划运行的容器，直到成功升级所有节点。

有关如何使用在 AKS 中的多个节点池的详细信息，请参阅[创建和管理在 AKS 中为群集的多个节点池][use-multiple-node-pools]。

## <a name="pods"></a>Pod

Kubernetes 使用 Pod 来运行应用程序的实例。 Pod 表示应用程序的单个实例。 Pod 通常与容器存在 1 对 1 映射，但高级方案中一个 Pod 可能包含多个容器。 在同一个节点上共同计划这些多容器 Pod，并允许容器共享相关资源。

创建 Pod 时，可以定义资源限制以请求一定数量的 CPU 或内存资源。 Kubernetes 计划程序尝试计划在具有可用资源的节点上运行 Pod，以满足请求。 此外可以指定最大资源限制，防止给定的 Pod 从基础节点消耗过多计算资源。 最佳做法是包括所有 Pod 的资源限制，以帮助 Kubernetes 计划程序了解所需和允许的资源。

有关详细信息，请参阅 [Kubernetes Pod][kubernetes-pods] 和 [Kubernetes Pod 生命周期][kubernetes-pod-lifecycle]。

Pod 是逻辑资源，但容器是应用程序工作负荷的运行位置。 Pod 通常是短暂的可支配资源，单独计划的 Pod 会错过 Kubernetes 提供的一些高可用性和冗余功能。 相反，Pod 通常由 Kubernetes 控制器（例如 Deployment 控制器）进行部署和管理。

## <a name="deployments-and-yaml-manifests"></a>部署和 YAML 清单

部署表示由 Kubernetes Deployment 控制器管理的一个或多个相同 Pod。 部署定义要创建的副本 (Pod) 数量，Kubernetes 计划程序确保如果 Pod 或节点出现故障，则在正常节点上安排其他 Pod。

可以更新部署以更改 Pod 的配置、使用的容器映像或附加存储。 Deployment 控制器耗尽并终止给定数量的副本，从新部署定义创建副本，并继续该过程，直至部署中的所有副本都已更新。

AKS 中的大多数无状态应用程序应使用部署模型，而不是计划单个 Pod。 Kubernetes 可以监视部署的运行状况和状态，以确保在群集中运行所需数量的副本。 如果仅计划单独的 pod，如果他们遇到问题，则不会重新启动 pod，并且如果其当前节点遇到问题，则不会在正常的节点上重新计划 pod。

如果应用程序需要一定数量的实例才能做出管理决策，你不希望更新进程来中断该功能。 Pod 中断预算可用于定义在更新或节点升级期间部署中可以删除的副本数。 例如，如果部署中有 5 个副本，则可以定义 4 个 Pod 中断，以便一次只允许删除/重新计划一个副本。 与 Pod 资源限制一样，最佳做法是在需要始终存在最少数量副本的应用程序上定义 Pod 中断预算。

通常使用 `kubectl create` 或 `kubectl apply` 来创建和管理部署。 为创建部署，可使用 YAML（YAML 不标记语言）格式定义清单文件。 以下示例创建 NGINX Web 服务器的基本部署。 部署指定要创建的 3 个副本，并在容器上打开端口 80。 还为 CPU 和内存定义了资源请求和限制。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
            memory: 64Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

通过在 YAML 清单中包含负载均衡器等服务，还可以创建更复杂的应用程序。

有关详细信息，请参阅 [Kubernetes 部署][kubernetes-deployments]。

### <a name="package-management-with-helm"></a>使用 Helm 进行包管理

在 Kubernetes 中管理应用程序的常用方法是使用 [Helm][helm]。 可以生成和使用包含应用程序代码打包版本和 Kubernetes YAML 清单的现有公共 Helm chart 来部署资源。 这些 Helm chart 可以存储在本地，通常也可以存储在远程存储库中，例如 [Azure 容器注册表 Helm chart 存储库][acr-helm]。

为使用 Helm，Kubernetes 群集中会安装名为 Tiller 的服务器组件。 Tiller 管理群集中群集的安装。 Helm 客户端自身本地安装在计算机上，也可以在 [Azure Cloud Shell][azure-cloud-shell] 中使用。 可以使用客户端搜索或创建 Helm 图表，然后将其安装到 Kubernetes 群集。

![Helm 包括客户端组件和服务器端 Tiller 组件，用于在 Kubernetes 群集内创建资源](media/concepts-clusters-workloads/use-helm.png)

有关详细信息，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用 Helm 安装应用程序][aks-helm]。

## <a name="statefulsets-and-daemonsets"></a>StatefulSet 和 DaemonSet

部署控制器使用 Kubernetes 计划程序在具有可用资源的任何可用节点上运行给定数量的副本。 这种使用部署的方法对于无状态应用程序可能已足够，但对于需要持久命名约定或存储的应用程序则不行。 对于群集内需要在每个节点或选定节点上存在副本的应用程序，部署控制器不会查看副本在节点间的分布情况。

以下两种 Kubernetes 资源可以管理这类应用程序：

- StatefulSet - 维护超出单个 Pod 生命周期的应用程序的状态（如存储）。
- DaemonSet - 确保在 Kubernetes 启动进程早期每个节点上都有正在运行的实例。

### <a name="statefulsets"></a>StatefulSet

现代应用程序开发通常针对无状态应用程序，但 StatefulSet 可用于有状态应用程序（如包含数据库组件的应用程序）。 StatefulSet 是类似于创建和管理一个或多个相同 Pod 的部署。 StatefulSet 中的副本按照正常有序的方法来部署、缩放、升级和终止。 使用 StatefulSet，重新计划副本时，命名约定、网络名称和存储将保持不变。

使用 `kind: StatefulSet` 以 YAML 格式定义应用程序，然后 StatefulSet 控制器处理所需副本的部署和管理。 数据会写入到由 Azure 托管磁盘或 Azure 文件提供的永久性存储。 使用 StatefulSet，删除 StatefulSet 时，基础持久性存储仍然保持不变。

有关详细信息，请参阅 [Kubernetes StatefulSet][kubernetes-statefulsets]。

系统会规划 StatefulSet 中的副本，并在 AKS 群集中的任何可用节点上运行这些副本。 如需确保 Set 中至少有一个 Pod 在节点上运行，则可以改用 DaemonSet。

### <a name="daemonsets"></a>DaemonSet

对于特定的日志集合或监视需求，可能需要在所有或选定的节点上运行给定的 Pod。 DaemonSet 再次用于部署一个或多个相同的 Pod，但 DaemonSet 控制器会确保指定的每个节点都运行 Pod 实例。

在默认的 Kubernetes 计划程序启动之前，DaemonSet 控制器可以在群集启动进程的早期计划节点上的 Pod。 此功能可确保在计划 Deployment 或 StatefulSet 中的传统 Pod 之前启动 DaemonSet 中的 Pod。

与 StatefulSet 一样，系统使用 `kind: DaemonSet` 将 DaemonSet 定义为 YAML 定义的一部分。

有关详细信息，请参阅 [Kubernetes DaemonSet][kubernetes-daemonset]。

> [!NOTE]
> 如果使用[虚拟节点外接程序](virtual-nodes-cli.md#enable-virtual-nodes-addon)，Daemonset 不会在虚拟节点创建 pod。

## <a name="namespaces"></a>命名空间

Kubernetes 资源（如 Pod 和部署）以逻辑方式分组到命名空间中。 这些分组提供了一种以逻辑方式划分 AKS 群集并限制创建、查看或管理资源访问权限的方法。 例如，可以创建命名空间以分隔业务组。 用户只能与分配的命名空间内的资源进行交互。

![Kubernetes 命名空间以逻辑方式划分资源和应用程序](media/concepts-clusters-workloads/namespaces.png)

创建 AKS 群集时，可使用以下命名空间：

- default - 不提供任何命名空间时，默认情况下在此命名空间中创建 Pod 和部署。 在小型环境中，可以将应用程序直接部署到默认命名空间，而无需创建其他逻辑分隔。 与 Kubernetes API（例如 `kubectl get pods`）交互时，如果未指定命名空间，则使用默认值。
- kube-system - 此命名空间是核心资源的所在位置，例如 DNS 和代理等网络功能或 Kubernetes 仪表板。 通常不会将应用程序部署到此命名空间中。
- kube-public - 通常不使用此命名空间，但可以用于让资源在整个群集中可见，并可供任何用户查看。

有关详细信息，请参阅 [Kubernetes 命名空间][kubernetes-namespaces]。

## <a name="next-steps"></a>后续步骤

本文涵盖了一些核心 Kubernetes 组件以及如何将它们应用于 AKS 群集的内容。 有关核心 Kubernetes 和 AKS 概念的详细信息，请参阅以下文章：

- [Kubernetes/AKS 访问和标识][aks-concepts-identity]
- [Kubernetes/AKS 安全性][aks-concepts-security]
- [Kubernetes/AKS 虚拟网络][aks-concepts-network]
- [Kubernetes/AKS 存储][aks-concepts-storage]
- [Kubernetes/AKS 规模][aks-concepts-scale]

<!-- EXTERNAL LINKS -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubernetes-pods]: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
[kubernetes-pod-lifecycle]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
[kubernetes-deployments]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
[kubernetes-statefulsets]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[kubernetes-daemonset]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
[kubernetes-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[helm]: https://helm.sh/
[azure-cloud-shell]: https://shell.azure.com

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[acr-helm]: ../container-registry/container-registry-helm-repos.md
[aks-helm]: kubernetes-helm.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[operator-best-practices-scheduler]: operator-best-practices-scheduler.md
[use-multiple-node-pools]: use-multiple-node-pools.md
