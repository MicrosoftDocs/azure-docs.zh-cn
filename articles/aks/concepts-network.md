---
title: 概念 - Azure Kubernetes 服务 (AKS) 中的网络
description: 了解 Azure Kubernetes 服务 (AKS) 中的网络，包括 kubenet 和 Azure CNI、入口控制器、负载均衡器和静态 IP 地址。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 02/28/2019
ms.author: iainfou
ms.openlocfilehash: 2d51699138914e4a8ad5d2a133161fcfce71e9fe
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074048"
---
# <a name="network-concepts-for-applications-in-azure-kubernetes-service-aks"></a>Azure Kubernetes 服务 (AKS) 中应用程序的网络概念

在基于容器的微服务应用程序开发方法中，应用程序组件必须协同工作才能处理其任务。 Kubernetes 提供支持此应用程序通信的各种资源。 可以在内部或外部连接应用程序并将其公开。 要生成高可用性应用程序，可以对应用程序进行负载均衡。 更复杂的应用程序可能需要配置 SSL/TLS 终止的入口流量或多个组件的路由。 出于安全考虑，可能还需要限制流量流入或 Pod 和节点之间的流量。

本文介绍了为 AKS 中的应用程序提供网络的核心概念：

- [服务](#services)
- [Azure 虚拟网络](#azure-virtual-networks)
- [入口控制器](#ingress-controllers)
- [网络策略](#network-policies)

## <a name="kubernetes-basics"></a>Kubernetes 基础知识

为允许访问应用程序或让应用程序组件相互通信，Kubernetes 为虚拟网络提供了抽象层。 Kubernetes 节点连接到虚拟网络，可为 Pod 提供入站和出站连接。 kube-proxy 组件在每个节点上运行，以提供这些网络功能。

在 Kubernetes 中，服务以逻辑方式对 Pod 进行分组，以允许通过 IP 地址或 DNS 名称以及特定端口进行直接访问。 此外，还可使用负载均衡器分发流量。 使用入口控制器也可实现更复杂的应用程序流量路由。 使用 Kubernetes 网络策略可提供安全性，还可筛选 Pod 网络流量（在 AKS 中，为预览版）。

Azure 平台还有助于简化 AKS 群集的虚拟网络。 创建 Kubernetes 负载均衡器时，将创建和配置基础 Azure 负载均衡器资源。 打开 Pod 的网络端口时，会配置相应的 Azure 网络安全组规则。 对于 HTTP 应用程序路由，Azure 还可以在配置新的入口路由时配置外部 DNS。

## <a name="services"></a>服务

为简化应用程序工作负载的网络配置，Kubernetes 使用服务以逻辑方式对一组 Pod 进行分组并提供网络连接。 可用的服务类型如下：

- **群集 IP** - 创建在 AKS 群集中使用的内部 IP 地址。 适用于支持群集中其他工作负载的仅限内部使用的应用程序。

    ![显示 AKS 群集中群集 IP 流量的示意图][aks-clusterip]

- **NodePort** - 在基础节点上创建端口映射，该映射允许使用节点 IP 地址和端口直接访问应用程序。

    ![显示 AKS 群集中 NodePort 流量的示意图][aks-nodeport]

- **LoadBalancer** - 创建 Azure 负载均衡器资源、配置外部 IP 地址并将请求的 Pod 连接到负载均衡器后端池。 为允许客户流量发送到应用程序，要在所需端口上创建负载均衡规则。 

    ![显示 AKS 群集中负载均衡器流量的示意图][aks-loadbalancer]

    针对入站流量的其他控制和路由，可能要改用[入口控制器](#ingress-controllers)。

- **ExternalName** - 创建特定的 DNS 条目，便于访问应用程序。

可以动态分配负载均衡器和服务的 IP 地址，也可以指定要使用的现有静态 IP 地址。 可以分配内部和外部静态 IP 地址。 这个现有静态 IP 地址通常与 DNS 条目绑定。

可以创建内部和外部负载均衡器。 内部负载均衡器仅分配一个专用 IP 地址，因此无法从 Internet 访问。

## <a name="azure-virtual-networks"></a>Azure 虚拟网络

在 AKS 中，可以部署使用以下两种网络模型之一的群集：

- Kubenet 网络 - 通常在部署 AKS 群集时创建和配置网络资源。
- Azure 容器网络接口 (CNI) 网络 - AKS 群集连接到现有的虚拟网络资源和配置。

### <a name="kubenet-basic-networking"></a>Kubenet（基本）网络

kubenet 网络选项是用于创建 AKS 群集的默认配置。 使用 kubenet，节点从 Azure 虚拟网络子网获取 IP 地址。 Pod 接收从逻辑上不同的地址空间到节点的 Azure 虚拟网络子网的 IP 地址。 然后配置网络地址转换 (NAT)，以便 Pod 可以访问 Azure 虚拟网络上的资源。 流量的源 IP 地址是 NAT 到节点的主 IP 地址。

节点使用 [kubenet][kubenet] Kubernetes 插件。 可以让 Azure 平台创建和配置虚拟网络，或选择将 AKS 群集部署到现有虚拟网络子网中。 同样，只有 Pod 和接收可路由 IP 地址的节点才能使用 NAT 与 AKS 群集外的其他资源进行通信。 这种方法大大减少了需要在网络空间中保留供 Pod 使用的 IP 地址数量。

有关详细信息，请参阅[为 AKS 群集配置 kubenet 网络][aks-configure-kubenet-networking]。

### <a name="azure-cni-advanced-networking"></a>Azure CNI（高级）网络

借助 Azure CNI，每个 pod 都可以从子网获取 IP 地址，并且可以直接访问。 这些 IP 地址在网络空间必须是唯一的，并且必须事先计划。 每个节点都有一个配置参数来表示它支持的最大 Pod 数。 这样，就会为每个节点预留相应的 IP 地址数。 此方法需要更多的计划，并且经常会耗尽 IP 地址或者在应用程序需求增长时需要在更大的子网中重新生成群集。

节点使用 [Azure 容器网络接口 (CNI)][cni-networking] Kubernetes 插件。

![显示两个节点的示意图，其中的网桥将每个节点连接到单个 Azure VNet][advanced-networking-diagram]

Azure CNI 通过 kubenet 网络提供以下功能：

- 在虚拟网络中，群集中的每个 Pod 都分配了一个 IP 地址。 Pod 可以直接与群集中的其他 Pod 以及虚拟网络中的其他节点通信。
- 启用了服务终结点的子网中的 Pod 可以安全连接到 Azure 服务（例如 Azure 存储和 SQL DB）。
- 可以创建用户定义的路由 (UDR) 将来自 Pod 的流量路由到网络虚拟设备。

有关详细信息，请参阅[为 AKS 群集配置 Azure CNI][aks-configure-advanced-networking]。

## <a name="ingress-controllers"></a>入口控制器

创建 LoadBalancer 类型服务时，系统会创建基础 Azure 负载均衡器资源。 负载均衡器配置为在给定端口上将流量分发到服务中的 Pod。 LoadBalancer 仅在第 4 层工作 - 服务不知道实际的应用程序，不会考虑任何其他路由。

入口控制器在第 7 层工作，可使用更智能的规则来分发应用程序流量。 入口控制器的一个常见用途是基于入站 URL 将 HTTP 流量路由到不同的应用程序。

![显示 AKS 群集中入口流量的示意图][aks-ingress]

在 AKS 中，可以使用 NGINX 之类的服务器创建入口资源，或使用 AKS HTTP 应用程序路由功能。 为 AKS 群集启用 HTTP 应用程序路由时，Azure 平台会创建入口控制器和 External-DNS 控制器。 在 Kubernetes 中创建新的入口资源时，系统会在特定于群集的 DNS 区域中创建所需的 DNS A 记录。 有关详细信息，请参阅[部署 HTTP 应用程序路由][aks-http-routing]。

入口的另一个常见功能是 SSL/TLS 终止。 在通过 HTTPS 访问的大型 Web 应用程序上，TLS 终止可以由入口资源处理，而不是在应用程序自身内部处理。 要提供自动 TLS 认证生成和配置，可以将入口资源配置为使用 Let's Encrypt 之类的提供程序。 有关使用 Let's Encrypt 配置 NGINX 入口控制器的详细信息，请参阅 [Ingress 和 TLS ][aks-ingress-tls]。

## <a name="network-security-groups"></a>网络安全组

网络安全组筛选 VM（例如 AKS 节点）的流量。 创建服务（如 LoadBalancer）时，Azure 平台会自动配置所需的任何网络安全组规则。 请勿手动配置网络安全组规则，以筛选 AKS 群集中 Pod 的流量。 定义任何所需端口并作为 Kubernetes 服务清单的一部分转发，让 Azure 平台创建或更新相应的规则。 还可以使用网络策略（如下一部分中所述）来自动向 Pod 应用流量筛选器规则。

## <a name="network-policies"></a>网络策略

默认情况下，AKS 群集中的所有 Pod 都可以无限制地发送和接收流量。 为了提高安全性，你可能想要定义用来控制流量流的规则。 后端应用程序通常只向所需的前端服务公开，或者数据库组件仅可由连接到它们的应用程序层访问。

网络策略是提供可用于控制 pod 之间的流量流的 AKS 中的 Kubernetes 功能。 可选择基于分配的标签、命名空间或流量端口等设置来允许或拒绝流量。 网络安全组更多是针对 AKS 节点，而不是针对 Pod。 使用网络策略是一种更合适的用来控制流量流的云本机方式。 因为 Pod 是在 AKS 群集中动态创建的，则可以动态应用所需的网络策略。

有关详细信息，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用网络策略保护 Pod 之间的流量][use-network-policies]。

## <a name="next-steps"></a>后续步骤

要开始使用 AKS 网络，请使用 [kubenet][aks-configure-kubenet-networking] 或 [Azure CNI][aks-configure-advanced-networking] 创建和配置具有自己的 IP 地址范围的 AKS 群集。

如需相关的最佳做法，请参阅 [AKS 中的网络连接和安全性的最佳做法][operator-best-practices-network]。

有关核心 Kubernetes 和 AKS 概念的详细信息，请参阅以下文章：

- [Kubernetes/AKS 群集和工作负荷][aks-concepts-clusters-workloads]
- [Kubernetes/AKS 访问和标识][aks-concepts-identity]
- [Kubernetes/AKS 安全性][aks-concepts-security]
- [Kubernetes/AKS 存储][aks-concepts-storage]
- [Kubernetes/AKS 规模][aks-concepts-scale]

<!-- IMAGES -->
[aks-clusterip]: ./media/concepts-network/aks-clusterip.png
[aks-nodeport]: ./media/concepts-network/aks-nodeport.png
[aks-loadbalancer]: ./media/concepts-network/aks-loadbalancer.png
[advanced-networking-diagram]: ./media/concepts-network/advanced-networking-diagram.png
[aks-ingress]: ./media/concepts-network/aks-ingress.png

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[aks-http-routing]: http-application-routing.md
[aks-ingress-tls]: ingress.md
[aks-configure-kubenet-networking]: configure-kubenet.md
[aks-configure-advanced-networking]: configure-azure-cni.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-identity]: concepts-identity.md
[use-network-policies]: use-network-policies.md
[operator-best-practices-network]: operator-best-practices-network.md
