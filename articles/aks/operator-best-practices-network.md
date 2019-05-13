---
title: 操作员最佳做法 - Azure Kubernetes 服务 (AKS) 中的网络连接
description: 了解 Azure Kubernetes 服务 (AKS) 中虚拟网络资源和连接的群集运算符的最佳实践
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 12/10/2018
ms.author: iainfou
ms.openlocfilehash: b26af87de8a09f987d69f0441a817638e626b4af
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192235"
---
# <a name="best-practices-for-network-connectivity-and-security-in-azure-kubernetes-service-aks"></a>Azure Kubernetes 服务 (AKS) 中的网络连接和安全的最佳做法

在创建和管理 Azure Kubernetes 服务 (AKS) 中的群集时，可以为节点和应用程序提供网络连接。 这些网络资源包括 IP 地址范围、负载均衡器和入口控制器。 若要保持高质量的应用程序服务，则需要计划和配置这些资源。

这篇最佳做法文章重点介绍群集运算符的网络连接和安全。 在本文中，学习如何：

> [!div class="checklist"]
> * 比较 AKS 中的 kubenet 和 Azure CNI 网络模式
> * 计划所需的 IP 地址和连接
> * 使用负载均衡器、入口控制器或 Web 应用程序防火墙 (WAF) 分配流量
> * 安全地连接到群集节点

## <a name="choose-the-appropriate-network-model"></a>选择合适的网络模型

**最佳做法指南** - 为了与现有的虚拟网络或本地网络集成，请在 AKS 中使用 Azure CNI 网络。 利用此网络模型，还可以更大程度地将企业环境中的资源和控制相分离。

虚拟网络为 AKS 节点和客户提供了用于访问应用程序的基本链接。 将 AKS 群集部署到虚拟网络有两种不同的方法：

* **Kubenet 网络** - Azure 在部署群集时管理虚拟网络资源，并使用 [kubenet][kubenet] Kubernetes 插件。
* **Azure CNI 网络** - 部署到现有的虚拟网络，并使用 [Azure 容器网络接口 (CNI)][cni-networking] Kubernetes 插件。 Pod 接收可以路由到其他网络服务或本地资源的各个 Ip。

容器网络接口 (CNI) 是与供应商无关的协议，允许容器运行时将向网络提供程序发出请求。 Azure CNI 将 IP 地址分配给 Pod 和节点，并在接到现有的 Azure 虚拟网络时提供 IP 地址管理 (IPAM) 功能。 每个节点和 Pod 资源接收 Azure 虚拟网络中的 IP 地址，与其他资源或服务通信不需要其他路由。

![显示两个节点的示意图，其中的网桥将每个节点连接到单个 Azure VNet](media/operator-best-practices-network/advanced-networking-diagram.png)

对于大多数生产部署，应使用 Azure CNI 网络。 使用此网络模型可以将资源的控制和管理相分离。 从安全角度看，通常希望不同的团队来管理和保护这些资源。 使用 Azure CNI 网络，可以通过分配到每个 Pod 的 IP 地址直接连接到现有 Azure 资源、本地资源或其他服务。

使用 Azure CNI 网络时，虚拟网络资源位于 AKS 群集外单独存在的资源组中。 委托 AKS 服务主体的权限以访问和管理这些资源。 AKS 群集使用的服务主体在虚拟网络中的子网上必须至少具有[网络参与者](../role-based-access-control/built-in-roles.md#network-contributor)权限。 如果希望定义[自定义角色](../role-based-access-control/custom-roles.md)而不是使用内置的网络参与者角色，则需要以下权限：
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

有关 AKS 服务主体委托的详细信息，请参阅[委托对其他 Azure 资源的访问权限][sp-delegation]。

每个节点和 Pod 在接收自己的 IP 地址时，请规划 AKS 子网的地址范围。 子网必须大到足以为每个部署的节点、Pod 和网络资源提供 IP 地址。 每个 AKS 群集必须位于自己的子网中。 要允许连接到 Azure 中的本地网络或对等互连网络，请勿使用与现有网络资源重叠的 IP 地址范围。 每个节点使用 kubenet 和 Azure CNI 网络运行的 Pod 数量存在默认限制。 若要处理纵向扩展事件或群集升级，还需要可在分配的子网中使用的其他 IP 地址。

若要计算所需的 IP 地址，请参阅[在 AKS 中配置 Azure CNI 网络][advanced-networking]。

### <a name="kubenet-networking"></a>Kubenet 网络

尽管 kubenet 不需要在部署群集之前设置虚拟网络，但也有一些缺点：

* 节点和 Pod 位于不同的 IP 子网中。 用户定义的路由 (UDR) 和 IP 转发用于 Pod 和节点之间的路由流量。 这个额外的路由可能会降低网络性能。
* 连接到现有本地网络或与其他 Azure 虚拟网络对等互连可能很复杂。

Kubenet 适用于小型开发或测试工作负荷，因为无需从 AKS 群集单独创建虚拟网络和子网。 流量较低或者将工作负荷直接迁移到容器中的简单网站，也可以受益于使用 kubenet 网络部署的 AKS 群集的简单性。 对于大多数生产部署，应计划和使用 Azure CNI 网络。 还可以[使用 kubenet 配置自己的 IP 地址范围和虚拟网络][aks-configure-kubenet-networking]。

## <a name="distribute-ingress-traffic"></a>分配入口流量

**最佳实践指南** - 要将 HTTP 或 HTTPS 流量分配到应用程序，请使用入口资源和控制器。 入口控制器通过常规 Azure 负载均衡器提供附加功能，可作为本机 Kubernetes 资源进行管理。

Azure 负载均衡器可以将客户流量分配到 AKS 群集中的各个应用程序，但是对这些流量的了解有限。 负载均衡器资源在第 4 层工作，并根据协议或端口分配流量。 大多数使用 HTTP 或 HTTPS 的 Web 应用程序应使用在第 7 层工作的 Kuberenetes 入口资源和控制器。 入口可以根据应用程序的 URL 分配流量并处理 TLS/SSL 终端。 此功能还可以减少公开和映射的 IP 地址数。 使用负载平衡器，每个应用程序通常需要分配一个公共 IP 地址并映射到 AKS 群集中的服务。 使用入口资源，单个 IP 地址可以将流量分配给多个应用程序。

![显示 AKS 群集中入口流量的示意图](media/operator-best-practices-network/aks-ingress.png)

 入口有两个组件：

 * 入口资源，和
 * 入口控制器

入口资源是 `kind: Ingress` 的 YAML 清单，它定义了将流量路由到 AKS 群集中运行的服务的主机、证书和规则。 以下示例 YAML 清单会将 myapp.com 的流量分配到 blogservice 或 storeservice 两个服务中的一个。 客户根据他们访问的 URL，被定向到一个或另一个服务。

```yaml
kind: Ingress
metadata:
 name: myapp-ingress
   annotations: kubernetes.io/ingress.class: "PublicIngress"
spec:
 tls:
 - hosts:
   - myapp.com
   secretName: myapp-secret
 rules:
   - host: myapp.com
     http:
      paths:
      - path: /blog
        backend:
         serviceName: blogservice
         servicePort: 80
      - path: /store
        backend:
         serviceName: storeservice
         servicePort: 80
```

入口控制器是在 AKS 节点上运行的守护程序并监视传入请求。 然后根据入口资源中定义的规则分配流量。 最佳常见的入口控制器基于 [NGINX]。 AKS 不会限制于特定的控制器，因此可以使用其他控制器，例如 [Contour][contour]、[HAProxy][haproxy] 或 [Traefik][traefik]。

入口有许多方案，包括以下操作指南：

* [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
* [创建使用内部、专用网络和 IP 地址的入口控制器][aks-ingress-internal]
* [创建使用你自己的 TLS 证书的入口控制器][aks-ingress-own-tls]
* 创建一个使用 Let's Encrypt 的入口控制器，以自动生成[具有动态公共 IP 地址][aks-ingress-tls]或[具有静态公共 IP 地址][aks-ingress-static-tls]的 TLS 证书

## <a name="secure-traffic-with-a-web-application-firewall-waf"></a>使用 Web 应用程序防火墙 (WAF) 保护流量

**最佳做法指南** - 要扫描传入流量是否存在潜在攻击，请使用 Web 应用程序防火墙 (WAF)，例如 [Barracuda WAF for Azure][barracuda-waf] 或 Azure 应用程序网关。 这些更高级的网络资源还可以将流量路由到 HTTP 和 HTTPS 连接或基本 SSL 终端之外。

将流量分配到服务和应用程序的入口控制器通常是 AKS 群集中的 Kubernetes 资源。 控制器作为守护程序在 AKS 节点上运行，并使用一些节点资源（例如 CPU、内存和网络带宽）。 在较大的环境中，通常需要将部分流量路由或 TLS 终端卸载到 AKS 群集之外的网络资源。 还需要扫描传入流量是否存在潜在攻击。

![Azure 应用程序网关等 Web 应用程序防火墙 (WAF) 可以保护和分配 AKS 群集的流量](media/operator-best-practices-network/web-application-firewall-app-gateway.png)

Web 应用程序防火墙 (WAF) 通过筛选传入流量提供额外的安全层。 开放式 Web 应用程序安全项目 (OWASP) 提供了一套规则来监视跨网站脚本或 cookie 中毒之类的攻击。 [Azure 应用程序网关][app-gateway]（目前在 AKS 中预览）是一种 WAF，可在流量到达 AKS 群集和应用程序之前与 AKS 群集集成以提供这些安全功能。 其他第三方解决方案也可以执行这些功能，因此可以在给定的产品中继续使用现有的资源和专业知识。

负载均衡器或入口资源继续在 AKS 群集中运行以进一步优化流量分配。 通过资源定义，可以将应用程序网关可以作为入口控制器进行集中管理。 要快速入门，请[创建应用程序网关入口控制器][app-gateway-ingress]。

## <a name="control-traffic-flow-with-network-policies"></a>使用网络策略控制流量流

**最佳做法指南** - 使用网络策略允许或拒绝到 Pod 的流量。 默认情况下，将允许群集中 Pod 之间的所有流量。 为了提高安全性，请定义对 Pod 通信进行限制的规则。

网络策略是一项 Kubernetes 功能，可用于控制 Pod 之间的流量流。 你可以选择基于分配的标签、命名空间或流量端口等设置来允许或拒绝流量。 使用网络策略提供了一种云本机方式来控制流量流。 因为 Pod 是在 AKS 群集中动态创建的，则可以动态应用所需的网络策略。 不要使用 Azure 网络安全组来控制 Pod 到 Pod 流量，请使用网络策略。

若要使用网络策略，必须在创建 AKS 群集时启用此功能。 无法在现有 AKS 群集上启用网络策略。 请提前进行规划以确保在群集上启用网络策略并根据需要使用它们。

网络策略是使用 YAML 清单作为 Kubernetes 资源创建的。 策略应用于所定义的 Pod，然后，入口或出口规则定义流量可以如何流动。 以下示例将向应用了 *app: backend* 标签的 Pod 应用网络策略。 然后，入口规则仅允许来自具有 *app: frontend* 标签的 Pod 的流量：

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

若要开始使用策略，请参阅[在 Azure Kubernetes 服务 (AKS) 中使用网络策略保护 Pod 之间的流量][use-network-policies]。

## <a name="securely-connect-to-nodes-through-a-bastion-host"></a>通过堡垒主机安全地连接到节点

**最佳做法指南** - 不公开到 AKS 节点的远程连接。 在管理虚拟网络中创建堡垒主机或跳转盒。 使用堡垒主机将流量安全地路由到 AKS 群集以远程管理任务。

AKS 中的大多数操作都可以使用 Azure 管理工具或通过 Kubernetes API 服务器来完成。 AKS 节点不会连接到公共 Internet，并且仅在专用网络上可用。 要连接到节点并执行维护或排查问题，请通过堡垒主机或跳转盒路由连接。 此主机应位于与 AKS 群集虚拟网络安全对等互连的单独的管理虚拟网络中。

![使用堡垒主机或跳转盒连接到 AKS 节点](media/operator-best-practices-network/connect-using-bastion-host-simplified.png)

堡垒主机的管理网络也应受到保护。 使用 [Azure ExpressRoute][expressroute] 或 [VPN 网关][ vpn-gateway]连接到本地网络，并使用网络安全组控制访问。

## <a name="next-steps"></a>后续步骤

本文重点介绍网络连接性和安全性。 有关 Kubernetes 中的网络基础知识的详细信息，请参阅 [Azure Kubernetes 服务 (AKS) 中应用程序的网络概念][aks-concepts-network]

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[app-gateway-ingress]: https://github.com/Azure/application-gateway-kubernetes-ingress
[nginx]: https://www.nginx.com/products/nginx/kubernetes-ingress-controller
[contour]: https://github.com/heptio/contour
[haproxy]: https://www.haproxy.org
[traefik]: https://github.com/containous/traefik
[barracuda-waf]: https://www.barracuda.com/products/webapplicationfirewall/models/5

<!-- INTERNAL LINKS -->
[aks-concepts-network]: concepts-network.md
[sp-delegation]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-own-tls]: ingress-own-tls.md
[app-gateway]: ../application-gateway/overview.md
[use-network-policies]: use-network-policies.md
[advanced-networking]: configure-azure-cni.md
[aks-configure-kubenet-networking]: configure-kubenet.md
