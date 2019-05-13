---
title: 开发人员最佳做法 - Azure Kubernetes 服务 (AKS) 中的资源管理
description: 了解有关 Azure Kubernetes 服务 (AKS) 中的资源管理的应用程序开发人员最佳做法
services: container-service
author: zr-msft
ms.service: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.author: zarhoads
ms.openlocfilehash: aebade14f3a8a1095925d17325ce99b78031dc32
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073956"
---
# <a name="best-practices-for-application-developers-to-manage-resources-in-azure-kubernetes-service-aks"></a>有关管理 Azure Kubernetes 服务 (AKS) 中的资源的应用程序开发人员最佳做法

在 Azure Kubernetes 服务 (AKS) 中开发和运行应用程序时，需要考虑到几个重要方面。 如何应对应用程序部署可能对所提供服务的最终用户体验造成的负面影响。 为帮助获得成功，在 AKS 中开发和运行应用程序时，请遵循一些最佳做法。

本最佳做法文章从应用程序开发人员的角度重点介绍如何运行群集和工作负荷。 有关管理最佳做法的信息，请参阅[有关 Azure Kubernetes 服务 (AKS) 中的隔离和资源管理的群集操作员最佳做法][operator-best-practices-isolation]。 本文内容：

> [!div class="checklist"]
> * pod 资源请求和限制是什么
> * 使用 Dev Spaces 与 Visual Studio Code 开发和部署应用程序的方法
> * 如何使用 `kube-advisor` 工具检查部署问题

## <a name="define-pod-resource-requests-and-limits"></a>定义 pod 资源请求和限制

**最佳做法指导** - 在 YAML 清单中针对所有 pod 设置 pod 请求和限制。 如果 AKS 群集使用资源配额，而你未定义这些值，则可能会拒绝你的部署。

管理 AKS 群集中的计算资源的主要方法是使用 pod 请求和限制。 这些请求和限制可让 Kubernetes 计划程序知道应该为 pod 分配哪些计算资源。

* **pod 请求**定义 pod 所需的 CPU 和内存量。 这些请求应该是 pod 提供可接受的性能级别所需的计算资源量。
    * 当 Kubernetes 计划程序尝试在节点上放置 pod 时，将使用 pod 请求来确定哪个节点具有足够的可用资源。
    * 监视应用程序的性能并调整这些请求，以确保定义的资源不会少于保持可接受性能级别所需的资源。
* **pod 限制**是 pod 可以使用的最大 CPU 和内存量。 这些限制有助于防止一个或两个失控 pod 在节点中占用过多的 CPU 和内存。 这种情况会降低节点的性能，以及其上运行的其他 pod 的性能。
    * 设置的 pod 限制不应超过节点可以支持的限制。 每个 AKS 节点将为核心 Kubernetes 组件保留一定的 CPU 和内存量。 应用程序可能会尝试消耗节点上的大量资源，使其他 pod 能够成功运行。
    * 同样，请在一天或一周的不同时间监视应用程序的性能。 确定峰值需求是什么，并根据满足应用程序需求所需的资源来调整 pod 限制。

最佳做法是在 pod 规范中定义这些请求和限制。 如果不包含这些值，Kubernetes 计划程序将不知道需要哪些资源。 计划程序可能会在资源不足的节点上计划 pod，从而无法提供可接受的应用程序性能。 群集管理员可以针对需要设置资源请求和限制的命名空间设置资源配额。 有关详细信息，请参阅[ AKS 群集中的资源配额][resource-quotas]。

定义 CPU 请求或限制时，值以 CPU 单位计量。 *1.0* CPU 相当于节点上的一个基础虚拟 CPU 核心。 GPU 使用与此相同的计量方法。 还可以定义小数请求或限制，通常以 millicpu 为单位。 例如，*100m* 表示 *0.1* 个基础虚拟 CPU 核心。

在以下单个 NGINX pod 的基本示例中，pod 请求 *100m* 的 CPU 时间和 *128 Mi* 的内存。 pod 的资源限制设置为 *250m* CPU 和 *256Mi* 内存：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

有关资源计量和分配的详细信息，请参阅[管理容器的计算资源][k8s-resource-limits]。

## <a name="develop-and-debug-applications-against-an-aks-cluster"></a>针对 AKS 群集开发和调试应用程序

**最佳做法指导** - 开发团队应该使用 Dev Spaces 针对 AKS 群集进行部署和调试。 此开发模型可确保在将应用部署到生产环境之前，实现基于角色的访问控制、网络或存储需求。

使用 Azure Dev Spaces 直接针对 AKS 群集开发、调试和测试应用程序。 在整个应用程序生命周期，团队中的开发人员共同协作进行生成和测试。 可以继续使用现有的工具，例如 Visual Studio 或 Visual Studio Code。 将为 Dev Spaces 安装一个扩展，以提供所需的选项用于在 AKS 群集中运行和调试应用程序：

![使用 Dev Spaces 在 AKS 群集中调试应用程序](media/developer-best-practices-resource-management/dev-spaces-debug.png)

这种使用 Dev Spaces 的集成式开发和测试过程减少了 [minikube][minikube] 等本地测试环境的需求。 可以针对 AKS 群集进行开发和测试。 可根据前面有关使用命名空间逻辑隔离群集的部分中所述保护和隔离此群集。 当准备好将应用部署到生产环境时，可以放心地进行部署，因为针对真正 AKS 群集的所有开发工作均已完成。

## <a name="use-the-visual-studio-code-extension-for-kubernetes"></a>使用适用于 Kubernetes 的 Visual Studio Code 扩展

**最佳做法指导** - 编写 YAML 清单时安装并使用适用于 Kubernetes 的 VS Code 扩展。 还可将该扩展用于集成式部署解决方案，不经常与 AKS 群集交互的应用程序所有者也许可从中获得帮助。

[适用于 Kubernetes 的 Visual Studio Code 扩展][vscode-kubernetes]可帮助你开发应用程序并将其部署到 AKS。 该扩展为 Kubernetes 资源以及 Helm 图表和模板提供 intellisense 功能。 还可以从 VS Code 内部浏览、部署和编辑 Kubernetes 资源。 该扩展还针对 pod 规范中设置的资源请求或限制提供 intellisense 检查：

![适用于 Kubernetes 的 VS Code 扩展中有关缺少内存限制的警告](media/developer-best-practices-resource-management/vs-code-kubernetes-extension.png)

## <a name="regularly-check-for-application-issues-with-kube-advisor"></a>定期使用 kube-advisor 检查应用程序问题

**最佳做法指导** - 定期运行最新版本的 `kube-advisor` 开放源代码工具，以检测群集中的问题。 如果针对现有 AKS 群集应用资源配额，请先运行 `kube-advisor`，以查找未定义资源请求和限制的 pod。

[kube-advisor][kube-advisor] 工具是一个关联的 AKS 开放源代码项目，它将扫描 Kubernetes 群集，并报告它找到的问题。 一项有用的检查是识别未应用资源请求和限制的 pod。

在托管许多开发团队和应用程序的 AKS 群集中，可能很难跟踪未设置这些资源请求和限制的 pod。 最佳做法是定期针对 AKS 群集运行 `kube-advisor`。

## <a name="next-steps"></a>后续步骤

本最佳做法文章从群集操作员的角度重点介绍了如何运行群集和工作负荷。 有关管理最佳做法的信息，请参阅[有关 Azure Kubernetes 服务 (AKS) 中的隔离和资源管理的群集操作员最佳做法][operator-best-practices-isolation]。

若要实施其中的某些最佳做法，请参阅以下文章：

* [使用 Dev Spaces 进行开发][dev-spaces]
* [使用 kube-advisor 检查问题][aks-kubeadvisor]

<!-- EXTERNAL LINKS -->
[k8s-resource-limits]: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
[vscode-kubernetes]: https://github.com/Azure/vscode-kubernetes-tools
[kube-advisor]: https://github.com/Azure/kube-advisor
[minikube]: https://kubernetes.io/docs/setup/minikube/

<!-- INTERNAL LINKS -->
[aks-kubeadvisor]: kube-advisor-tool.md
[dev-spaces]: ../dev-spaces/get-started-netcore.md
[operator-best-practices-isolation]: operator-best-practices-cluster-isolation.md
[resource-quotas]: operator-best-practices-scheduler.md#enforce-resource-quotas
