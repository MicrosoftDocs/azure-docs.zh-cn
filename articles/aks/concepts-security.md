---
title: 概念 - Azure Kubernetes 服务 (AKS) 安全性
description: 了解 Azure Kubernetes 服务 (AKS) 安全性，包括 master 和节点通信、网络策略和 Kubernetes 机密。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: iainfou
ms.openlocfilehash: 2e655627267546d88f76a2487817bca3153ee91d
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074022"
---
# <a name="security-concepts-for-applications-and-clusters-in-azure-kubernetes-service-aks"></a>Azure Kubernetes 服务 (AKS) 中应用程序和群集的安全性相关概念

在 Azure Kubernetes 服务 (AKS) 中运行应用程序工作负荷的过程中，若要保护客户数据，关键是要确保群集的安全性。 Kubernetes 包括安全组件，如网络策略和机密。 Azure 会添加组件，例如网络安全组和协调群集升级。 这些安全组件共同确保 AKS 群集运行最新的 OS 安全更新和 Kubernetes 版本，并确保安全的 pod 流量和对敏感凭据的安全访问。

本文介绍用于保护 AKS 中应用程序的核心概念：

- [主组件安全性](#master-security)
- [节点安全性](#node-security)
- [群集升级](#cluster-upgrades)
- [网络安全](#network-security)
- [Kubernetes 机密](#kubernetes-secrets)

## <a name="master-security"></a>主组件安全

在 AKS 中，Kubernetes 主组件是 Microsoft 提供的托管服务的一部分。 每个 AKS 群集都有其自己的专用单租户 Kubernetes 主组件，用于提供 API 服务器、计划程序等。此主管理并由 Microsoft 维护。

默认情况下，Kubernetes API 服务器使用公共 IP 地址，并具有完全限定的域名 (FQDN)。 可使用 Kubernetes 基于角色的访问控制和 Azure Active Directory 控制对 API 服务器的访问。 有关详细信息，请参阅 [Azure AD 与 AKS 集成][aks-aad]。

## <a name="node-security"></a>节点安全性

AKS 节点是由你管理和维护的 Azure 虚拟机。 运行使用小鲸鱼容器运行时优化的 Ubuntu 分发版的 Linux 节点。 Windows 服务器节点 （目前以预览版在 AKS 中） 运行优化的 Windows Server 2019 发布，也使用小鲸鱼容器运行时。 创建或纵向扩展了 AKS 群集时，会自动使用最新的 OS 安全更新和配置来部署节点。

Azure 平台会自动适用于 Linux 节点上每晚的 OS 安全修补程序。 如果 Linux OS 安全更新要求重新启动主机，不会自动执行了重新启动。 可以手动重新启动 Linux 节点，或一种常见方法是使用[Kured][kured]，一个适用于 Kubernetes 的开放源代码重启守护程序。 Kured 作为 [DaemonSet][aks-daemonsets] 运行并监视每个节点，用于确定指示需要重启的文件是否存在。 通过使用相同的 [cordon 和 drain 进程](#cordon-and-drain)作为群集升级，来跨群集管理重启。

Windows Server 中的节点 （当前在 AKS 中的预览版），Windows 更新不会不会自动运行，并应用最新的更新。 有关在 Windows 更新发布周期和验证过程按定期计划，应在 AKS 群集中执行 Windows Server 节点池上的升级。 此升级过程创建节点运行的最新的 Windows Server 映像和修补程序，然后删除旧节点。 此过程的详细信息，请参阅[升级在 AKS 中的节点池][nodepool-upgrade]。

系统将节点部署到专用虚拟网络子网中，且不分配公共 IP 地址。 出于故障排除和管理的目的，会默认启用 SSH。 只能使用内部 IP 地址访问此 SSH。

为提供存储，节点使用 Azure 托管磁盘。 这些是由高性能固态硬盘支持的高级磁盘，适用于大多数规模的 VM 节点。 托管磁盘上存储的数据在 Azure 平台内会自动静态加密。 为提高冗余，还会在 Azure 数据中心内安全复制这些磁盘。

目前，在恶意的多租户使用情况下，AKS 或其他位置中的 Kubernetes 环境并不完全安全。 用于节点的其他安全功能（例如 *Pod 安全策略*或更细粒度的基于角色的访问控制 (RBAC)）可增加攻击的难度。 但是，为了在运行恶意多租户工作负荷时获得真正的安全性，虚拟机监控程序应是你唯一信任的安全级别。 Kubernetes 的安全域成为整个群集，而不是单个节点。 对于这些类型的恶意多租户工作负荷，应使用物理隔离的群集。 有关如何隔离工作负荷的详细信息，请参阅 [AKS 中的群集隔离最佳做法][cluster-isolation]，

## <a name="cluster-upgrades"></a>群集升级

为实现安全性和符合性及使用最新功能，Azure 提供相应工具来协调 AKS 群集和组件的升级。 此升级业务流程同时包括 Kubernetes 主组件和代理组件。 可查看适用于 AKS 群集的[可用 Kubernetes 版本的列表](supported-kubernetes-versions.md)。 若要开始进行升级，先指定一个可用版本（在上述版本中）。 接着 Azure 会安全隔离和排空每个 AKS 节点并执行升级。

### <a name="cordon-and-drain"></a>隔离和排空

在升级过程中，AKS 节点是单独封锁从群集，因此它们不会计划新的 pod。 然后将节点排空并进行升级，操作如下：

- 一个新的节点会部署到的节点池。 此节点都运行最新 OS 映像和修补程序。
- 升级已标识一个现有节点。 此节点上 pod 是正常终止并安排在节点池中的其他节点上。
- 从 AKS 群集中删除此现有节点。
- 在群集中的下一个节点封锁和排除使用相同的过程，直到所有节点成功都替换作为升级过程的一部分。

有关详细信息，请参阅[升级 AKS 群集][aks-upgrade-cluster]。

## <a name="network-security"></a>网络安全

如需实现本地网络的连接和安全性，可将 AKS 群集部署到现有 Azure 虚拟网络子网。 这些虚拟网络可具有指向本地网络的 Azure 站点到站点 VPN 或 Express Route 连接。 可以使用专用的内部 IP 地址定义 Kubernetes 入口控制器，以便只能通过此内部网络连接访问服务。

### <a name="azure-network-security-groups"></a>Azure 网络安全组

为筛选虚拟网络中的通信流量，Azure 使用网络安全组规则。 这些规则定义要允许或拒绝哪些源和目标 IP 范围、端口和协议访问资源。 会创建默认规则以允许 TLS 流量流向 Kubernetes API 服务器。 在使用负载均衡器、端口映射或入口路由创建服务时，AKS 会自动修改网络安全组，以便流量流向正确的方向。

## <a name="kubernetes-secrets"></a>Kubernetes 机密

Kubernetes *机密*用于将敏感数据注入到 pod，例如访问凭据或密钥。 首先使用 Kubernetes API 创建机密。 在定义 pod 或部署时，可以请求特定机密。 机密仅提供给所计划的 pod 需要该机密的节点，且机密存储在 *tmpfs* 中，不写入磁盘。 当节点上最后一个需要该机密的 pod 被删除后，将从该节点的 tmpfs 中删除该机密。 机密存储在给定的命名空间中，只有同一命名空间中的 pod 能访问该机密。

使用机密会减少 pod 或服务 YAML 清单中定义的敏感信息。 可以请求存储在 Kubernetes API 服务器中的机密，作为 YAML 清单的一部分。 此方法仅为 pod 提供特定的机密访问权限。

## <a name="next-steps"></a>后续步骤

若要开始为 AKS 群集提供保护，请参阅[升级 AKS 群集][aks-upgrade-cluster]。

如需相关的最佳做法，请参阅 [AKS 中群集安全性和升级的最佳做法][operator-best-practices-cluster-security]。

有关核心 Kubernetes 和 AKS 概念的详细信息，请参阅以下文章：

- [Kubernetes/AKS 群集和工作负荷][aks-concepts-clusters-workloads]
- [Kubernetes/AKS 标识][aks-concepts-identity]
- [Kubernetes/AKS 虚拟网络][aks-concepts-network]
- [Kubernetes/AKS 存储][aks-concepts-storage]
- [Kubernetes/AKS 规模][aks-concepts-scale]

<!-- LINKS - External -->
[kured]: https://github.com/weaveworks/kured
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/

<!-- LINKS - Internal -->
[aks-daemonsets]: concepts-clusters-workloads.md#daemonsets
[aks-upgrade-cluster]: upgrade-cluster.md
[aks-aad]: azure-ad-integration.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[cluster-isolation]: operator-best-practices-cluster-isolation.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
