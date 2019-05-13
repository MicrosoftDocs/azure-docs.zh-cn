---
title: Azure Kubernetes 服务 (AKS) 的支持策略
description: 了解有关 Azure Kubernetes 服务 (AKS) 的支持策略、 共担责任和预览 （或 alpha 或 beta） 中的功能。
services: container-service
author: jnoller
ms.service: container-service
ms.topic: article
ms.date: 04/01/2019
ms.author: jenoller
ms.openlocfilehash: 9b779021eca11638e8ee52ec11d082e5b0e89cd4
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506680"
---
# <a name="support-policies-for-azure-kubernetes-service"></a>Azure Kubernetes 服务的支持策略

本文提供了有关技术支持策略和限制 Azure Kubernetes 服务 (AKS) 的详细信息。 文章还详细介绍了辅助角色节点管理、 托管的控制平面组件、 第三方开放源代码组件和安全或修补程序管理。

## <a name="service-updates-and-releases"></a>服务更新和版本

* 发布信息，请参阅[AKS 发行说明](https://github.com/Azure/AKS/releases)。
* 在预览版中功能的信息，请参阅[AKS 预览功能和相关的项目](https://github.com/Azure/AKS/blob/master/previews.md)。

## <a name="managed-features-in-aks"></a>托管在 AKS 中的功能

基本基础结构即服务 (IaaS) 云组件，如计算或网络组件以及与低级别的控件和自定义选项，让用户访问。 与此相反，AKS 提供统包的 Kubernetes 部署，为客户提供了一组常用的配置和所需的功能。 AKS 客户具有有限的自定义、 部署和其他选项。 这些客户不需要担心或直接管理 Kubernetes 群集。

通过 AKS，客户获取的完全托管型*控制平面*。 控制平面包含的所有组件和客户需要运行，并向最终用户提供 Kubernetes 群集的服务。 所有 Kubernetes 组件维护，由 Microsoft 运营。

Microsoft 管理和监视通过控制窗格的以下组件：

* Kubelet 或 Kubernetes API 服务器
* Etcd 或兼容的键-值存储，提供服务质量 (QoS)、 可伸缩性，以及运行时
* DNS 服务 （例如，kube dns 或 CoreDNS）
* Kubernetes 代理或网络

AKS 并不是完全托管的群集解决方案。 如辅助角色节点的一些组件具有*责任分担*，其中用户必须帮助维护 AKS 群集。 需用户输入，例如，若要应用辅助节点操作系统 (OS) 安全修补程序。

服务都*托管*意义上说，Microsoft 和 AKS 团队部署，在操作，并负责服务可用性和功能。 客户无法更改这些托管的组件。 Microsoft 限制用于确保一致且可缩放的用户体验的自定义。 完全可自定义解决方案，请参阅[AKS 引擎](https://github.com/Azure/aks-engine)。

> [!NOTE]
> AKS 辅助角色节点在 Azure 门户中显示为常规 Azure IaaS 资源。 但这些虚拟机部署到自定义 Azure 资源组 (前缀为 MC\\*)。 它是无法更改 AKS 辅助角色节点。 例如，可以使用安全外壳 (SSH) 更改的方式更改常规虚拟机的 AKS 辅助角色节点 （但是，不能更改基本的操作系统映像，并更改可能未在更新保持不变或重新启动），并可以将其他 Azure 资源附加到 AKS辅助角色节点。 进行更改时，但是*带外管理和自定义项，* AKS 群集可能会变得无法支持。 避免更改辅助角色节点，除非 Microsoft 支持部门要求你进行更改。

## <a name="shared-responsibility"></a>共担责任

创建群集时，客户定义 AKS 创建 Kubernetes 辅助角色节点。 在这些节点上执行客户工作负荷。 客户拥有并可以查看或修改辅助角色节点。

由于客户群集节点执行专用代码，并将敏感数据存储，Microsoft 支持部门可以仅以有限方式来访问它们。 Microsoft 支持部门不能登录、 执行命令中，或查看这些节点，而无需 express 客户权限或协助的日志。

由于辅助角色节点不敏感，Microsoft 将需要格外注意以限制其背景管理。 在许多情况下，你的工作负荷将继续运行，即使 Kubernetes 主节点、 etcd 和其他 Microsoft 托管的组件失败。 不小心修改后的辅助角色节点可能会导致丢失的数据和工作负荷，而会使群集不支持。

## <a name="aks-support-coverage"></a>AKS 支持覆盖范围

Microsoft 提供以下技术支持：

* Kubernetes 服务提供和支持，例如 API 服务器的所有 Kubernetes 组件建立的连接。
* 管理、 运行时间、 QoS 和操作 Kubernetes 控制平面服务 （Kubernetes 主节点、 API 服务器、 etcd 和 kube dns，例如）。
* Etcd。 支持包括 etcd 的所有数据的自动、 透明备份灾难规划和群集还原状态每隔 30 分钟。 这些备份不是直接提供给客户或用户。 它们确保数据可靠性和一致性。
* 适用于 Kubernetes 的 Azure 云提供程序驱动程序中的任何集成点。 其中包括到负载均衡器、 永久性卷或网络 （Kubernetes 和 Azure CNI） 等其他 Azure 服务集成。
* 问题或有关自定义的控制平面组件，如 Kubernetes API 服务器、 etcd 和 kube dns 问题。
* 有关网络问题，如 Azure CNI、 kubenet，或其他网络访问权限和功能的问题。 问题可能包括 DNS 解析、 数据包丢失、 路由和等等。 Microsoft 支持的各种网络方案：
  * Kubenet (basic) 和高级群集内部的网络 (Azure CNI) 和关联的组件
  * 连接到其他 Azure 服务和应用程序
  * 入口控制器和入口或负载均衡器配置
  * 网络性能和延迟

Microsoft 不提供对以下技术支持：

* 有关如何使用 Kubernetes 的问题。 例如，Microsoft 支持不提供有关如何创建自定义入口控制器，使用应用程序工作负荷或应用第三方或开放源代码软件程序包或工具的建议。
  > [!NOTE]
  > Microsoft 支持部门可以建议在 AKS 群集功能、 自定义和优化 （如 Kubernetes 操作问题和过程）。
* 第三方开放源代码项目未作为 Kubernetes 的一部分提供的控制平面，或使用 AKS 群集中部署。 Istio、 Helm、 Envoy，或其他人可能包括这些项目。
  > [!NOTE]
  > Microsoft 可以提供对 Helm 和 Kured 等第三方开放源代码项目的最大努力支持。 在第三方开源工具集成与 Kubernetes Azure 云提供商或其他特定于 AKS 的 bug，Microsoft 才支持示例和应用程序从 Microsoft 文档。
* 第三方关闭源软件。 此软件可以包括安全扫描工具和网络设备或软件。
* 多云或多供应商的生成版本有关的问题。 例如，Microsoft 不支持与运行联合 multipublic 云供应商解决方案相关的问题。
* 网络中未列出的其他自定义项[AKS 文档](https://docs.microsoft.com/azure/aks/)。
  > [!NOTE]
  > Microsoft 支持问题和与网络安全组 (Nsg) 相关的 bug。 例如，Microsoft 支持部门可以回答有关更新了 NSG 失败或意外的 NSG 或负载均衡器行为的问题。

## <a name="aks-support-coverage-for-worker-nodes"></a>AKS 支持覆盖范围的辅助角色节点

### <a name="microsoft-responsibilities-for-aks-worker-nodes"></a>AKS 辅助角色节点的 Microsoft 职责

Microsoft 和客户共同承担责任 Kubernetes 辅助角色节点位置：

* 基本 OS 映像已要求 （如监视和网络代理） 的新增功能。
* 辅助角色节点自动接收操作系统修补程序。
* 问题使用 Kubernetes 控制的平面辅助角色节点运行的组件自动进行修正。 组件包括：
  * Kube-proxy
  * 网络隧道提供到 Kubernetes 的通信路径的主机组件
  * kubelet
  * Docker 或小鲸鱼守护程序

> [!NOTE]
> 在辅助角色节点上，如果控件平面组件处于非正常运行，AKS 团队可能需要重启整个辅助角色节点。 由于他们对客户的活动的工作负荷和数据的受限访问，AKS 团队仅当客户他们会上报问题重启辅助角色节点。 如有可能，AKS 团队致力于防止影响应用程序需要重新启动。

### <a name="customer-responsibilities-for-aks-worker-nodes"></a>客户负责 AKS 辅助角色节点

Microsoft 不会自动重启辅助角色节点，以应用 OS 级修补程序。 OS 修补程序发送到辅助角色节点中，尽管*客户*负责重启辅助角色节点，以应用更改。 共享的库，如混合固态驱动器 (SSHD) 的守护程序和其他组件的系统或操作系统级别自动进行修补。

客户负责执行 Kubernetes 升级。 它们可以执行的 Azure 控制面板或 Azure CLI 通过升级。 这适用于包含安全或对 Kubernetes 的功能改进的更新。

> [!NOTE]
> 由于 AKS*托管服务*，其最终目标包括负责修补程序、 更新、 删除和日志收集，使服务管理更完整且无需干预。 随着端到端管理的服务的容量增加，未来版本可能会省略某些函数 （例如，节点重新启动和自动修补功能）。

### <a name="security-issues-and-patching"></a>安全问题和修补

如果一个或多个组件的 AKS 中找到安全漏洞，AKS 团队将修补所有受影响的群集来缓解此问题。 或者，团队将为用户提供的升级指南。

安全缺陷影响，如果零停机时间修补程序是可用的辅助角色节点，AKS 团队会应用该修补程序并通知用户的更改。

当安全修补程序需要辅助角色节点重新启动时，Microsoft 将通知此要求的客户。 客户负责重新启动或更新若要获取的群集修补程序。 如果用户没有应用根据 AKS 指南的修补程序，其群集将继续受到安全问题。

### <a name="node-maintenance-and-access"></a>节点维护和访问权限

辅助角色节点是共担责任，并归客户。 因此，客户可以登录到其辅助角色节点并进行潜在的有害更改，例如内核更新，然后安装或删除包。

如果客户进行破坏性更改，或导致平面组件处于脱机状态或无法正常工作的控件，AKS 将检测这种失败，并自动还原到以前的工作状态的辅助角色节点。

不过，客户可以登录到并更改辅助角色节点，因为更改可能会使群集不支持不建议为执行此操作。

## <a name="network-ports-access-and-nsgs"></a>网络端口、 访问和 Nsg

作为托管服务，AKS 有特定的网络和连接要求。 这些要求是比正常 IaaS 组件要求不太灵活。 在 AKS，自定义阻止特定端口 （例如，使用防火墙规则阻止出站端口 443），NSG 规则等操作和 Url 列入白名单可以使群集不支持。

> [!NOTE]
> 目前，AKS 不允许您完全锁定出你的群集 （例如，显式域或端口加入允许列表）。 Url 和端口的列表会发生变化而不发出警告。 可以通过创建 Azure 支持票证来获取更新的列表。 仅对于愿意接受其群集可用性可能会受到影响的客户列表是*在任何时间。*

## <a name="unsupported-alpha-and-beta-kubernetes-features"></a>不支持的 alpha 和 beta Kubernetes 的功能

AKS 支持仅稳定功能上游 Kubernetes 项目中。 除非有说明，否则，AKS 不支持 alpha 和 beta 上游 Kubernetes 项目中可用的功能。

在两个方案中，alpha 或 beta 功能可能会推出之前它们已公开发布：

* 客户与 AKS 产品、 支持或工程团队已满足，并且还需要尝试这些新功能。
* 这些功能已被[情况下一个功能标志启用](https://github.com/Azure/AKS/blob/master/previews.md)。 客户必须显式选择要使用这些功能。

## <a name="preview-features-or-feature-flags"></a>预览功能或功能标志

有关功能和要求扩展的测试的功能和用户反馈，Microsoft 发布新预览功能或功能在功能标志后面。 作为预发行版或 beta 功能，请考虑这些功能。

预览功能标志功能不适合在生产环境。 在 Api 和行为、 bug 修复和其他更改正在发生的更改可能导致不稳定的群集和停机时间。

公共预览版中的功能是属于最大努力支持，因为这些功能处于预览状态并不适合在生产环境，在营业时间内支持的 AKS 技术支持团队。 有关其他信息，请参阅：

* [Azure 支持常见问题](https://azure.microsoft.com/en-us/support/faq/)

> [!NOTE]
> 在 Azure 预览功能才会生效*订阅*级别。 请勿在生产订阅上安装预览版功能。 生产订阅后，预览功能可以更改默认 API 行为，并且会影响正常操作。

## <a name="upstream-bugs-and-issues"></a>上游 bug 和问题

给定的上游 Kubernetes 项目中的开发速度，总是出现 bug。 这些错误的一些无法修补或 AKS 系统中得到解决。 相反，bug 修补程序需要更大到上游的项目 （如 Kubernetes、 节点或辅助角色操作系统和内核） 的修补程序。 对于 Microsoft 拥有 （如 Azure 云提供程序） 的组件，AKS 和 Azure 的人员是致力于解决在社区中的上游问题。

通过一个或多个上游 bug 的根本原因是技术支持问题时，AKS 支持和工程团队将：

* 确定并链接任何支持的详细信息，以帮助解释为什么此问题会影响你的群集或工作负荷的上游 bug。 客户接收所需的存储库的链接，以便它们可以观看这些问题并查看新版本时提供的修补程序。
* 提供潜在解决方法或缓解措施。 如果可以缓解此问题，[已知问题](https://github.com/Azure/AKS/issues?q=is%3Aissue+is%3Aopen+label%3Aknown-issue)AKS 存储库中将字段。 已知问题备案说明：
  * 此问题，其中包括指向上游 bug。
  * 解决方法以及有关升级或其他详细信息的持久性的解决方案。
  * 大致时间线的问题包括，基于上游发布节奏。
