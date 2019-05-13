---
title: 在 Azure Kubernetes 服务 (AKS) 中配置 Azure CNI 网络
description: 了解如何在 Azure Kubernetes 服务 (AKS) 中配置 Azure CNI（高级）网络，包括将 AKS 群集部署到现有虚拟网络和子网中。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 10/11/2018
ms.author: iainfou
ms.openlocfilehash: 61968265670c53ebc4187c983996caa8c94a4cde
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508007"
---
# <a name="configure-azure-cni-networking-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中配置 Azure CNI 网络

默认情况下，AKS 群集使用 [kubenet][kubenet]，系统会为你创建 Azure 虚拟网络和子网。 使用 *kubenet*，节点从虚拟网络子网获取 IP 地址。 然后会在节点上配置网络地址转换 (NAT)，并且 Pod 将接收“隐藏”在节点 IP 背后的 IP 地址。 这种方法减少了需要在网络空间中保留供 Pod 使用的 IP 地址数量。

借助 [Azure 容器网络接口 (CNI)][cni-networking]，每个 Pod 都可以从子网获得 IP 地址，并且可供直接访问。 这些 IP 地址在网络空间中必须唯一，并且必须事先计划。 每个节点都有一个配置参数来表示它支持的最大 Pod 数。 这样，就会为每个节点预留相应的 IP 地址数。 使用此方法需要经过更详细的规划，并且经常会耗尽 IP 地址，或者在应用程序需求增长时需要在更大的子网中重建群集。

本文展示了如何使用 *Azure CNI* 网络来创建和使用 AKS 群集的虚拟网络子网。 有关网络选项和注意事项的详细信息，请参阅 [Kubernetes 和 AKS 的网络概念][aks-network-concepts]。

## <a name="prerequisites"></a>必备组件

* AKS 群集的虚拟网络必须允许出站 Internet 连接。
* 不要在同一子网中创建多个 AKS 群集。
* AKS 群集可能不会使用 Kubernetes 服务地址范围的 `169.254.0.0/16`、`172.30.0.0/16` 或 `172.31.0.0/16`。
* AKS 群集使用的服务主体在虚拟网络中的子网上必须至少具有[网络参与者](../role-based-access-control/built-in-roles.md#network-contributor)权限。 如果希望定义[自定义角色](../role-based-access-control/custom-roles.md)而不是使用内置的网络参与者角色，则需要以下权限：
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

## <a name="plan-ip-addressing-for-your-cluster"></a>规划群集的 IP 地址

使用 Azure CNI 网络配置的群集需要额外的规划。 虚拟网络及其子网的大小必须适应计划运行的 Pod 数以及群集的节点数。

Pod 和群集节点的 IP 地址是从虚拟网络中指定的子网分配的。 每个节点都配置了主 IP 地址。 默认情况下，Azure CNI 预先配置了 30 个额外的 IP 地址，这些地址被分配给安排在节点上的 Pod。 横向扩展群集时，将使用子网中的 IP 地址以类似方式配置每个节点。 还可以查看[每个节点的最大 Pod 数](#maximum-pods-per-node)。

> [!IMPORTANT]
> 应在考虑到升级和缩放操作的基础上确定所需的 IP 地址数。 如果设置的 IP 地址范围仅支持固定数量的节点，则无法升级或缩放群集。
>
> - **升级** AKS 群集时，会将一个新节点部署到该群集中。 服务和工作负荷开始在新节点上运行，旧节点将从群集中删除。 这种滚动升级过程要求至少有一个额外的 IP 地址块可用。 那么，节点计数是 `n + 1`。
>
> - **缩放** AKS 群集时，会将一个新节点部署到该群集中。 服务和工作负荷开始在新节点上运行。 确定 IP 地址范围时需要考虑到如何纵向扩展群集可以支持的节点和 Pod 数目。 此外，应该为升级操作包含一个额外的节点。 那么，节点计数是 `n + number-of-additional-scaled-nodes-you-anticipate + 1`。

如果预期节点将会运行最大数目的 Pod，并且会定期销毁和部署 Pod，则还应该考虑为每个节点分配一些额外的 IP 地址。 分配这些额外的 IP 地址是考虑到删除某个服务以及为了部署新服务并获取地址而释放 IP 地址可能需要几秒钟时间。

AKS 群集 IP 地址计划包括虚拟网络、至少一个节点和 Pod 子网以及 Kubernetes 服务地址范围。

| 地址范围 / Azure 资源 | 限制和调整大小 |
| --------- | ------------- |
| 虚拟网络 | Azure 虚拟网络的大小可以为 /8，但仅限于 65,536 个已配置的 IP 地址。 |
| 子网 | 大小必须足以容纳群集中可能预配的节点、Pod 以及所有 Kubernetes 和 Azure 资源。 例如，如果部署内部 Azure 负载均衡器，其前端 IP 分配自群集子网（而不是公共 IP）。 子网大小还应考虑到帐户升级操作或将来的缩放需求。<p />若要计算最小子网大小，包括用于升级操作的其他节点：`(number of nodes + 1) + ((number of nodes + 1) * maximum pods per node that you configure)`<p/>50 个节点群集的示例：`(51) + (51  * 30 (default)) = 1,581`（/21 或更大）<p/>50 节点群集的示例，其中还包括纵向扩展额外 10 个节点的预配：`(61) + (61 * 30 (default)) = 1,891`（/21 或更大）<p>如果在创建群集时没有指定每个节点的最大 Pod 数，则每个节点的最大 Pod 数将设置为 30。 所需的最小 IP 地址数取决于该值。 如果基于不同的最大值计算最小 IP 地址要求，请参阅[如何配置每个节点的最大 Pod 数](#configure-maximum---new-clusters)，以便在部署群集时设置此值。 |
| Kubernetes 服务地址范围 | 此范围不应由此虚拟网络上或连接到此虚拟网络的任何网络元素使用。 服务地址 CIDR 必须小于 /12。 |
| Kubernetes DNS 服务 IP 地址 | Kubernetes 服务地址范围内的 IP 地址将由群集服务发现 (kube-dns) 使用。 请勿使用地址范围内的第一个 IP 地址，例如 1。 子网范围内的第一个地址用于 kubernetes.default.svc.cluster.local 地址。 |
| Docker 桥地址 | IP 地址（采用 CIDR 表示法）用作节点上的 Docker 桥 IP 地址。 默认地址为 172.17.0.1/16。 |

## <a name="maximum-pods-per-node"></a>每个节点的最大 Pod 数

AKS 群集中每个节点的最大 Pod 数为 110。 每个节点的默认最大 Pod 数因 *kubenet* 和 *Azure CNI* 网络以及群集部署方法而异。

| 部署方法 | Kubenet 默认值 | Azure CNI 默认值 | 可在部署时配置 |
| -- | :--: | :--: | -- |
| Azure CLI | 110 | 30 | 是 （最多 250 个） |
| 资源管理器模板 | 110 | 30 | 是 （最多 250 个） |
| 门户 | 110 | 30 | “否” |

### <a name="configure-maximum---new-clusters"></a>配置最大值 - 新群集

只能在群集部署时配置每个节点的最大 Pod 数。 如果部署使用 Azure CLI 或使用资源管理器模板，则可以设置每个节点值的最大 pod，根据需要在以下`maxPods`指导原则：

| 网络 | 最小值 | 最大值 |
| -- | :--: | :--: |
| Azure CNI | 30 | 250 |
| Kubenet | 30 | 110 |

> [!NOTE]
> AKS 服务严格强制执行上表中的最小值。
您可以设置 maxPods 值低于最小值显示为这样做可以防止群集启动。

* **Azure CLI**：使用 [az aks create][az-aks-create] 命令部署群集时，指定 `--max-pods` 参数。 最大值为 250。
* **资源管理器模板**：使用资源管理器模板部署群集时，在 [ManagedClusterAgentPoolProfile] 对象中指定 `maxPods` 属性。 最大值为 250。
* **Azure 门户**：使用 Azure 门户部署群集时，不能更改每个节点的最大 Pod 数。 使用 Azure 门户部署时，Azure CNI 网络群集中每个节点的 Pod 数限制为 30 个。

### <a name="configure-maximum---existing-clusters"></a>配置最大值 - 现有群集

无法在现有 AKS 群集上更改每个节点的最大 Pod 数。 只能在最初部署群集时调整数量。

## <a name="deployment-parameters"></a>部署参数

创建 AKS 群集时，可为 Azure CNI 网络配置以下参数：

**虚拟网络**：要将 Kubernetes 群集部署到的虚拟网络。 要为群集创建新的虚拟网络，请选择“新建”，并按照“创建虚拟网络”部分中的步骤操作。 有关 Azure 虚拟网络的限制和配额的信息，请参阅 [Azure 订阅和服务限制、配额和约束](../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits)。

**子网**：要将群集部署到的虚拟网络中的子网。 若要在虚拟网络中为群集创建新的子网，请选择“新建”，并按照“创建子网”部分中的步骤操作。 对于混合连接，地址范围不应与环境中的其他任何虚拟网络重叠。

**Kubernetes 服务地址范围**：这是 Kubernetes 分配给群集中的内部[服务][services]的虚拟 IP 的集合。 可以使用任何专用地址范围，只要其符合以下要求即可：

* 不得在群集的虚拟网络 IP 地址范围内
* 不得与群集虚拟网络对等互连的任何其他虚拟网络重叠
* 不得与任何本地 IP 重叠
* 不得在范围 `169.254.0.0/16`、`172.30.0.0/16` 或 `172.31.0.0/16` 中

虽然从技术上来说可以在群集所在的虚拟网络中指定一个服务地址范围，但建议不要这样做。 如果使用重叠的 IP 范围，则可能导致不可预测的行为。 有关详细信息，请参阅本文中的[常见问题解答](#frequently-asked-questions)部分。 有关 Kubernetes 服务的详细信息，请参阅 Kubernetes 文档中的[服务][services]。

**Kubernetes DNS 服务 IP 地址**：群集的 DNS 服务的 IP 地址。 此地址必须在 Kubernetes 服务地址范围内。 请勿使用地址范围内的第一个 IP 地址，例如 1。 子网范围内的第一个地址用于 kubernetes.default.svc.cluster.local 地址。

**Docker 桥地址**：分配给 Docker 网桥的 IP 地址和子网掩码。 Docker 桥允许 AKS 节点与基础管理平台进行通信。 此 IP 地址不能在群集的虚拟网络 IP 地址范围内，并且不应当与网络上使用的其他地址范围重叠。

## <a name="configure-networking---cli"></a>配置网络 - CLI

使用 Azure CLI 创建 AKS 群集时，还可以配置 Azure CNI 网络。 可以使用以下命令在启用了 Azure CNI 网络的情况下创建新的 AKS 群集。

首先，将现有子网的子网资源 ID 加入到 AKS 群集将加入的子网资源 ID：

```console
$ az network vnet subnet list \
    --resource-group myVnet \
    --vnet-name myVnet \
    --query [].id --output tsv

/subscriptions/<guid>/resourceGroups/myVnet/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default
```

使用带有 `--network-plugin azure` 参数的 [az aks create][az-aks-create] 命令创建具有高级网络的群集。 使用上一步中收集的子网 ID 更新 `--vnet-subnet-id` 值：

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24
```

## <a name="configure-networking---portal"></a>配置网络 - 门户

以下 Azure 门户屏幕截图显示了在创建 AKS 群集过程中配置这些设置的示例：

![Azure 门户中的高级网络配置][portal-01-networking-advanced]

## <a name="frequently-asked-questions"></a>常见问题

以下问题和解答适用于 **Azure CNI** 网络配置。

* 是否可以在群集子网中部署 VM？

  不。 不支持在 Kubernetes 群集使用的子网中部署 VM。 可将 VM 部署在同一虚拟网络中，但必须部署在不同的子网中。

* *是否可以配置基于 Pod 的网络策略？*

  是的 Kubernetes 网络策略是在 AKS 中可用。 若要开始使用，请参阅[在 AKS 中使用网络策略保护 Pod 之间的流量][network-policy]。

* 可部署到节点的 Pod 数上限是否可配置？

  是的，使用 Azure CLI 或资源管理器模板部署群集时可配置。 请参阅[每个节点的最大 Pod 数](#maximum-pods-per-node)。

  无法在现有群集上更改每个节点的最大 Pod 数。

* 如何配置创建 AKS 群集期间创建的子网的其他属性？例如服务终结点。

  可以在 Azure 门户的标准虚拟网络配置页中，配置创建 AKS 群集期间创建的虚拟网络和子网的完整属性列表。

* 是否可以在我的群集虚拟网络中将另一子网用于 Kubernetes 服务地址范围？

  此配置是可以的，但建议不要这样做。 该服务地址范围是 Kubernetes 分配给群集中的内部服务的虚拟 IP (VIP) 的集合。 Azure 网络无法查看 Kubernetes 群集的服务 IP 范围。 由于无法查看群集的服务地址范围，因此有可能以后会在群集虚拟网络中创建新的子网，该子网与服务地址范围重叠。 如果出现这种形式的重叠，则 Kubernetes 为服务分配的 IP 可能是子网中另一资源正在使用的，导致不可预测的行为或故障。 如果能够确保所用地址范围不在群集的虚拟网络中，则可避免这种重叠风险。

## <a name="next-steps"></a>后续步骤

### <a name="networking-in-aks"></a>AKS 中的网络

通过以下文章详细了解 AKS 中的网络：

- [将静态 IP 地址用于 Azure Kubernetes 服务 (AKS) 负载均衡器](static-ip.md)
- [使用包含 Azure 容器服务 (AKS) 的内部负载均衡器](internal-lb.md)

- [使用外部网络连接创建基本入口控制器][aks-ingress-basic]
- [启用 HTTP 应用程序路由附加产品][aks-http-app-routing]
- [创建使用内部、专用网络和 IP 地址的入口控制器][aks-ingress-internal]
- [使用动态公共 IP 创建入口控制器并配置 Let 's Encrypt 以自动生成 TLS 证书][aks-ingress-tls]
- [使用静态公共 IP 创建入口控制器并配置 Let 's Encrypt 以自动生成 TLS 证书][aks-ingress-static-tls]

### <a name="aks-engine"></a>AKS 引擎

[Azure Kubernetes 服务引擎（AKS 引擎）][aks-engine]是一个开源项目，它能够生成 Azure 资源管理器模板用于在 Azure 上部署 Kubernetes 群集。

使用 AKS 引擎创建的 Kubernetes 群集支持 [kubenet][kubenet] 和 [Azure CNI][cni-networking] 插件。 因此，AKS 引擎同时支持这两种网络方案。

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[services]: https://kubernetes.io/docs/concepts/services-networking/service/
[portal]: https://portal.azure.com
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[aks-ssh]: ssh.md
[ManagedClusterAgentPoolProfile]: /azure/templates/microsoft.containerservice/managedclusters#managedclusteragentpoolprofile-object
[aks-network-concepts]: concepts-network.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-internal]: ingress-internal-ip.md
[network-policy]: use-network-policies.md
