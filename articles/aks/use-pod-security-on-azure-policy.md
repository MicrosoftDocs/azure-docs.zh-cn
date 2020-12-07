---
title: 'Azure Kubernetes Service 中的 Azure 策略的安全箱 (AKS) '
description: '了解如何在 Azure Kubernetes Service 上通过 Azure 策略保护 pod (AKS) '
services: container-service
ms.topic: article
ms.date: 09/22/2020
ms.openlocfilehash: 8e437095b3d527647a453ba89adaa2ab62672177
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348519"
---
# <a name="secure-pods-with-azure-policy"></a>使用 Azure Policy 保护 Pod

若要提高 AKS 群集的安全性，可以控制要授予哪些功能，以及根据公司策略运行的任何功能。 此访问通过 [用于 AKS 的 Azure 策略外接程序][kubernetes-policy-reference]提供的内置策略定义。 通过提供对 pod 规范安全方面（如 root 权限）的更多控制，可实现更严格的安全遵从性并了解群集中部署的内容。 如果 pod 不满足策略中指定的条件，Azure 策略可以禁止 pod 启动或标记冲突。 本文介绍如何使用 Azure 策略来限制 AKS 中 pod 的部署。

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 门户][aks-quickstart-portal]。

### <a name="install-the-azure-policy-add-on-for-aks"></a>安装适用于 AKS 的 Azure 策略外接程序

若要通过 Azure 策略保护 AKS pod，需要在 AKS 群集上安装用于 AKS 的 Azure 策略外接程序。 按照以下 [步骤安装 Azure 策略外接程序](../governance/policy/concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-aks)。

本文档假定你具有以下各项，这些部署在上面链接的演练中。

* 使用注册了 `Microsoft.ContainerService` 和 `Microsoft.PolicyInsights` 资源提供程序 `az provider register`
* Azure CLI 2.12 或更高版本
* 使用 Azure 策略外接程序安装的1.15 或更高版本上的 AKS 群集

## <a name="overview-of-securing-pods-with-azure-policy-for-aks"></a>用于 AKS 的 Azure 策略的安全箱概述

>[!NOTE]
> 本文档详细介绍了如何使用 Azure 策略来保护 pod，这是 [预览版中的 Kubernetes pod 安全策略功能](use-pod-security-policies.md)的后续内容。
> **Pod 安全策略 (预览) 和用于 AKS 的 Azure 策略外接程序不能同时启用。**
> 
> 如果在启用了 pod 安全策略的情况下将 Azure 策略外接程序安装到群集中，请 [按照以下步骤来禁用 pod 安全策略](use-pod-security-policies.md#enable-pod-security-policy-on-an-aks-cluster)。

在 AKS 群集中，在创建和更新资源时，会使用一个许可控制器来截获对 API 服务器的请求。 然后，该许可控制器可以根据一组规则 *验证* 资源请求是否应该创建。

以前，功能 [pod 安全策略 (预览) ](use-pod-security-policies.md) 通过 Kubernetes 项目启用，以限制可部署的内容。

使用 Azure 策略外接程序，AKS 群集可以使用内置的 Azure 策略，这些策略用于保护 pod 和其他类似于 pod 安全策略的 Kubernetes 资源。 适用于 AKS 的 Azure 策略外接程序安装一个验证程序 [的托管实例，即](https://github.com/open-policy-agent/gatekeeper)一个验证许可控制器。 适用于 Kubernetes 的 Azure 策略建立在开放源代码打开策略代理上，该代理依赖于 [Rego 策略语言](../governance/policy/concepts/policy-for-kubernetes.md#policy-language)。

本文档详细说明如何使用 Azure 策略来保护 AKS 群集中的 pod，并指导如何从 pod 安全策略迁移 (预览版) 。

## <a name="limitations"></a>限制

以下一般限制适用于适用于 Kubernetes 群集的 Azure 策略外接程序：

- Kubernetes 版本 **1.14** 或更高版本支持用于 Kubernetes 的 Azure 策略附加项。
- 适用于 Kubernetes 的 Azure 策略外接程序只能部署到 Linux 节点池
- 仅支持内置策略定义
- 每个群集每个策略的不符合记录的最大数目： **500**
- 每个订阅的不符合记录的最大数目： **1000000**
- 不支持 Azure 策略外接程序之外的网关安装程序。 在启用 Azure 策略外接程序之前，请卸载由以前的网关安装程序安装的所有组件。
- [不符合性的原因](../governance/policy/how-to/determine-non-compliance.md#compliance-reasons) 不可用于此 [资源提供程序模式](../governance/policy/concepts/definition-structure.md#resource-provider-modes)

以下限制仅适用于 AKS 的 Azure 策略外接程序：

- [AKS Pod 安全策略 (预览) ](use-pod-security-policies.md) 并且用于 AKS 的 Azure 策略外接程序不能同时启用。 
- 用于评估的 Azure 策略外接程序自动排除的命名空间： _kube_ 、 _gatekeeper-system_ _aks 和 periscope_ 。

### <a name="recommendations"></a>建议

下面是有关使用 Azure 策略外接程序的常规建议：

- Azure 策略外接程序需要3个要运行的网关守卫组件：1个审核 pod 和2个 webhook pod 副本。 在需要审核和强制操作的群集中，这些组件会消耗更多的资源，并在群集中增加 Kubernetes 资源和策略分配。

  - 对于在单个群集中最多具有20个约束的小于500个 pod：每个组件2个 vcpu 和 350 MB 内存。
  - 对于一个群集中超过500个 pod，最多有40个限制：每个组件3个 vcpu 和 600 MB 内存。

以下建议仅适用于 AKS 和 Azure 策略外接程序：

- 使用带有破坏的系统节点池 `CriticalAddonsOnly` 来计划网关守卫。 有关详细信息，请参阅 [使用系统节点池](use-system-pools.md#system-and-user-node-pools)。
- AKS 群集的安全出站流量。 有关详细信息，请参阅 [控制群集节点的出口流量](limit-egress-traffic.md)。
- 如果群集启用了 `aad-pod-identity`，节点托管标识 (NMI) pod 将修改节点的 iptable，以拦截对 Azure 实例元数据终结点的调用。 此配置意味着对元数据终结点发出的任何请求都将被 NMI 拦截，即使 pod 不使用 `aad-pod-identity`。 可以将 AzurePodIdentityException CRD 配置为通知 `aad-pod-identity` 应在不使用 NMI 进行出任何处理的情况下，代理与 CRD 中定义的标签匹配的 pod 所发起的对元数据终结点的任何请求。 应通过配置 AzurePodIdentityException CRD 在 `aad-pod-identity` 中排除在 _kube-system_ 命名空间中具有 `kubernetes.azure.com/managedby: aks` 标签的系统 pod。 有关详细信息，请参阅[禁用特定 pod 或应用程序的 aad-pod-identity](https://azure.github.io/aad-pod-identity/docs/configure/application_exception)。
  若要配置例外情况，请安装 [mic-exception YAML](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml)。

Azure 策略外接程序需要 CPU 和内存资源才能运行。 当群集的大小增加时，这些要求会增加。 有关使用 Azure 策略外接程序的一般指南，请参阅 [Azure 策略建议][policy-recommendations] 。

## <a name="azure-policies-to-secure-kubernetes-pods"></a>用于保护 Kubernetes pod 的 Azure 策略

安装 Azure 策略外接程序后，默认情况下不会应用任何策略。

有11个内置的单个 Azure 策略和两个内置的方案，它们专门保护 AKS 群集中的 pod。
每个策略都可以进行自定义。 [此处列出了 AKS 策略及其受支持效果][policy-samples]的完整列表。 阅读有关 [Azure 策略影响](../governance/policy/concepts/effects.md)的详细信息。

Azure 策略可以应用于管理组、订阅或资源组级别。 在资源组级别分配策略时，请确保在策略范围内选择目标 AKS 群集的资源组。 在策略范围内，已安装 Azure 策略外接程序的分配作用域中的每个群集。

如果使用 [pod 安全策略 (预览版) ](use-pod-security-policies.md)，请了解如何 [迁移到 Azure 策略以及其他行为差异](#migrate-from-kubernetes-pod-security-policy-to-azure-policy)。

### <a name="built-in-policy-initiatives"></a>内置策略计划

Azure 策略中的计划是一系列策略定义，旨在实现单一的总体目标。 使用计划可以简化跨 AKS 群集的策略的管理和分配。 作为单个对象存在的计划，详细了解 [Azure 策略计划](../governance/policy/overview.md#initiative-definition)。

适用于 Kubernetes 的 Azure 策略提供了两个内置方案，这些方案可保护 pod、 [基准](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) 和 [受限](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00)。

这两个内置方案都是从 [Kubernetes 的 pod 安全策略](https://github.com/kubernetes/website/blob/master/content/en/examples/policy/baseline-psp.yaml)中所使用的定义生成的。

|[Pod 安全策略控制](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Azure 策略定义链接| [基线计划](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) | [限制计划](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) |
|---|---|---|---|
|禁止运行特权容器|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95edb821-ddaf-4404-9732-666045e056b4)| 是 | 是
|不允许共享主机命名空间的使用|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a1ee2f-2a2a-4576-bf2a-e0e36709c2b8)| 是 | 是
|限制主机网络和端口的所有使用|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82985f06-dc18-4a48-bc1c-b9f4f0098cfe)| 是 | 是
|限制主机文件系统的任何使用|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F098fc59e-46c7-4d99-9b16-64990e543d75)| 是 | 是
|将 Linux 功能限制为 [默认集](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc26596ff-4d70-4e6a-9a30-c2506bd2f80c) | 是 | 是
|限制定义的卷类型的使用|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F16697877-1118-4fb1-9b65-9898ec2509ec)| - | 是-允许的卷类型为 `configMap` 、 `emptyDir` 、、 `projected` `downwardAPI` 、 `persistentVolumeClaim`|
|权限升级到根|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1c6e92c9-99f0-4e55-9cf2-0c234dc48f99) | - | 是 |
|限制容器的用户和组 Id|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | 是|
|限制分配拥有 pod 的卷的 FSGroup|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | 是-允许的规则为 `runAsUser: mustRunAsNonRoot` 、 `supplementalGroup: mustRunAs 1:65536` 、 `fsGroup: mustRunAs 1:65535` 、 `runAsGroup: mustRunAs 1:65535` 。  |
|需要 seccomp 配置文件|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F975ce327-682c-4f2e-aa46-b9598289b86c) | - | 是，allowedProfiles 是 * `docker/default` 或 `runtime/default` |

\* 自 v 1.11 后，Kubernetes 中已弃用 docker/默认值

### <a name="additional-optional-policies"></a>其他可选策略

还有其他 Azure 策略，可以在应用计划之外单独应用。 如果内置计划不满足你的要求，请考虑附加这些策略以及方案。

|[Pod 安全策略控制](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Azure 策略定义链接| 应用于基准计划（& a） | 除了限制计划之外还适用 |
|---|---|---|---|
|定义容器使用的 AppArmor 配置文件|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F511f5417-5d12-434d-ab2e-816901e72a5e) | 可选 | 可选 |
|允许非只读装载|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdf49d893-a74c-421d-bc95-c663042e5b80) | 可选 | 可选 |
|限制为特定 FlexVolume 驱动程序|[公有云](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4a8fce0-2dd5-4c21-9a36-8f0ec809d663) | 可选-如果只想限制 FlexVolume 驱动程序，而不是使用 "限制定义的卷类型的使用" 设置的其他驱动程序，请使用 | 不适用-受限制的计划包括 "限制使用定义的卷类型"，这不允许使用所有 FlexVolume 驱动程序 |

### <a name="unsupported-built-in-policies-for-managed-aks-clusters"></a>托管 AKS 群集不支持的内置策略

> [!NOTE]
> **AKS 中不支持** 以下3个策略，因为自定义由 AKS 作为托管服务管理和保护的方面。 这些策略专门针对具有非托管控制平面的 Azure Arc 连接群集构建。

|[Pod 安全策略控制](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)|
|---|
|定义容器的自定义 SELinux 上下文|
|限制容器使用的 sysctl 配置文件|
|定义了默认过程装载类型，以减少攻击面|

<!---
# Removing until custom initiatives are supported the week after preview

#### Custom initiative

If the built-in initiatives to address pod security do not match your requirements, you can choose your own policies to exist in a custom initiative. Read more about [building custom initatives in Azure Policy](../governance/policy/tutorials/create-and-manage#create-and-assign-an-initiative-definition).

--->

### <a name="namespace-exclusion"></a>命名空间排除

> [!WARNING]
> 管理员命名空间中的 pod （如 kube）必须运行才能使群集保持正常运行，从默认的已排除命名空间列表中删除所需的命名空间可能会因为所需的系统 pod 而触发策略冲突。

AKS 要求在群集上运行系统 pod 以提供关键服务，例如 DNS 解析。 限制 pod 功能的策略可能会影响系统 pod 的稳定性。 因此，在 **创建、更新和策略审核过程中，将在许可请求期间排除** 以下命名空间。 这会强制从 Azure 策略中排除这些命名空间的新部署。

1. kube-系统
1. 网关守卫-系统
1. azure-弧线
1. aks-periscope

在创建、更新和审核过程中，可以从评估中排除其他自定义命名空间。 如果有在批准命名空间中运行的专用 pod，并希望避免触发审核冲突，则应使用这些排除项。

## <a name="apply-the-baseline-initiative"></a>应用基线计划

> [!TIP]
> 所有策略都默认为审核效果。 可随时通过 Azure 策略将效果更新为 "拒绝"。

若要应用基线计划，可通过 Azure 门户进行分配。
<!--
1. Navigate to the Policy service in Azure portal
1. In the left pane of the Azure Policy page, select **Definitions**
1. Search for "Baseline Profile" on the search pane to the right of the page
1. Select `Kubernetes Pod Security Standards Baseline Profile for Linux-based workloads` from the `Kubernetes` category
-->
1. 单击 [此链接可访问 Azure 门户](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) 以查看 pod 安全基线计划
1. 将 **作用域** 设置为订阅级别，或仅设置为启用了 Azure 策略外接程序的包含 AKS 服务资源的资源组（不是Azure托管的“MC_<yourResourceGroup>_<region>”资源组）
1. 选择 " **参数** " 页面并将 **效果** 从更新 `audit` 为 " `deny` 阻止新的部署违反基线计划"
1. 添加其他命名空间，以便在创建、更新和审核期间从评估中排除[某些命名空间。](#namespace-exclusion) 
 ![更新效果](media/use-pod-security-on-azure-policy/update-effect.png)
1. 选择 " **查看 + 创建** " 以提交策略

通过运行确认策略已应用于群集 `kubectl get constrainttemplates` 。

> [!NOTE]
> 策略 [最多可能需要20分钟才能同步](../governance/policy/concepts/policy-for-kubernetes.md#assign-a-built-in-policy-definition) 到每个群集。

输出应如下所示：

```console
$ kubectl get constrainttemplate
NAME                                     AGE
k8sazureallowedcapabilities              30m
k8sazureblockhostnamespace               30m
k8sazurecontainernoprivilege             30m
k8sazurehostfilesystem                   30m
k8sazurehostnetworkingports              30m
```

## <a name="validate-rejection-of-a-privileged-pod"></a>验证特权 pod 的拒绝

首先测试在使用 `privileged: true` 安全性上下文计划 Pod 时所发生的情况。 此安全性上下文会提升 Pod 的特权。 基线计划不允许特权箱，因此，请求将被拒绝，导致部署被拒绝。

创建名为 `nginx-privileged.yaml` 的文件并粘贴以下 YAML 清单：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      securityContext:
        privileged: true
```

使用 [kubectl apply][kubectl-apply] 命令创建 pod，并指定 YAML 清单的名称：

```console
kubectl apply -f nginx-privileged.yaml
```

如预期那样，无法计划 pod，如以下示例输出所示：

```console
$ kubectl apply -f privileged.yaml

Error from server ([denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}): error when creating "privileged.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}
```

Pod 不会到达计划阶段，因此在继续之前，没有要删除的资源。

## <a name="test-creation-of-an-unprivileged-pod"></a>测试非特权 Pod 的创建

在上面的示例中，容器映像自动尝试使用根将 NGINX 绑定到端口 80。 基线策略计划拒绝了此请求，因此无法启动 pod。 现在，让我们尝试在无需访问权限的情况下运行相同的 NGINX pod。

创建名为 `nginx-unprivileged.yaml` 的文件并粘贴以下 YAML 清单：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

使用 [kubectl apply][kubectl-apply] 命令创建 Pod，并指定 YAML 清单的名称：

```console
kubectl apply -f nginx-unprivileged.yaml
```

Pod 已成功进行计划。 使用 [kubectl get pods][kubectl-get] 命令检查 Pod 的状态时，Pod 为 Running：

```console
$ kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          18s
```

此示例显示了仅影响违反集合中策略的部署的基准计划。 允许的部署将继续工作。

使用 [kubectl delete][kubectl-delete] 命令删除 NGINX 非特权 Pod，并指定 YAML 清单的名称：

```console
kubectl delete -f nginx-unprivileged.yaml
```

## <a name="disable-policies-and-the-azure-policy-add-on"></a>禁用策略和 Azure 策略外接程序

删除基准计划：

1. 导航到 Azure 门户上的 "策略" 窗格
1. 从左窗格中选择 **分配**
1. 单击 "..."基线配置文件旁边的按钮
1. 选择 "删除分配"

![删除分配](media/use-pod-security-on-azure-policy/delete-assignment.png)

若要禁用 Azure 策略外接程序，请使用 [az aks 加载项][az-aks-disable-addons] 命令。

```azurecli
az aks disable-addons --addons azure-policy --name MyAKSCluster --resource-group MyResourceGroup
```

了解如何 [从 Azure 门户删除 Azure 策略外接程序](../governance/policy/concepts/policy-for-kubernetes.md#remove-the-add-on-from-aks)。

## <a name="migrate-from-kubernetes-pod-security-policy-to-azure-policy"></a>从 Kubernetes pod 安全策略迁移到 Azure 策略

若要从 pod 安全策略迁移，需要在群集上执行以下操作。

1. 在群集上[禁用 pod 安全策略](use-pod-security-policies.md#clean-up-resources)
1. 启用 [Azure 策略外接程序][kubernetes-policy-reference]
1. 从[可用的内置策略][policy-samples]中启用所需的 Azure 策略

下面概述了 pod 安全策略与 Azure 策略之间的行为更改。

|场景| Pod 安全策略 | Azure Policy |
|---|---|---|
|安装|启用 pod 安全策略功能 |启用 Azure 策略外接程序
|部署策略| 部署 pod 安全策略资源| 将 Azure 策略分配到订阅或资源组作用域。 Azure 策略外接程序是 Kubernetes 资源应用程序所必需的。
| 默认策略 | 当在 AKS 中启用 pod 安全策略时，将应用默认的特权和无限制策略。 | 启用 Azure 策略外接程序不会应用默认策略。 必须在 Azure 策略中显式启用策略。
| 谁可以创建和分配策略 | 群集管理创建 pod 安全策略资源 | 对于 AKS 群集资源组，用户必须至少具有 "所有者" 角色或 "资源策略参与者" 权限。 -通过 API，用户可以在 AKS 群集资源范围内分配策略。 用户应在 AKS 群集资源上至少具有 "所有者" 或 "资源策略参与者" 权限。 -在 Azure 门户中，可以在管理组/订阅/资源组级别分配策略。
| 授权策略| 用户和服务帐户需要显式权限才能使用 pod 安全策略。 | 不需要对策略进行其他分配。 将策略分配到 Azure 后，所有群集用户都可以使用这些策略。
| 策略适用性 | 管理员用户绕过 pod 安全策略的实施。 |  (管理员 & 非管理员) 的所有用户都将看到相同的策略。 没有基于用户的特殊大小写。 可在命名空间级别排除策略应用程序。
| 策略作用域 | Pod 安全策略不带命名空间 | Azure 策略使用的约束模板不可带命名空间。
| 拒绝/审核/变化操作 | Pod 安全策略仅支持拒绝操作。 可以在 create 请求上对默认值进行转变。 可以在更新请求期间完成验证。| Azure 策略支持审核 & 拒绝操作。 目前尚不支持转变，但计划了。
| Pod 安全策略符合性 | 启用 pod 安全策略之前，不会看到已存在的有关 pod 的符合性。 启用 pod 安全策略后，创建不符合的 pod。 | 应用 Azure 策略之前存在的不兼容的 pod 将显示在 "策略冲突" 中。 如果策略设置为 "拒绝" 效果，则会拒绝启用 Azure 策略后创建的不符合的 pod。
| 如何查看群集上的策略 | `kubectl get psp` | `kubectl get constrainttemplate` -返回所有策略。
| Pod 安全策略标准版-特权 | 启用该功能时，默认情况下会创建特权 pod 安全策略资源。 | 特权模式意味着无限制，因此它等效于没有 Azure 策略分配。
| [Pod 安全策略标准-基线/默认值](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline-default) | 用户安装 pod 安全策略基线资源。 | Azure 策略提供了一个映射到基准箱安全策略的 [内置基线计划](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) 。
| [Pod 安全策略标准-受限](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) | 用户安装 pod 安全策略限制资源。 | Azure 策略提供了一种 [内置限制计划](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) ，该计划映射到受限箱安全策略。

## <a name="next-steps"></a>后续步骤

本文介绍如何应用 Azure 策略，该策略会限制无法部署特权箱以防止使用特权访问。 可以应用许多策略，例如限制卷使用的策略。 有关可用选项的详细信息，请参阅 [Azure Policy For Kubernetes 参考文档][kubernetes-policy-reference]。

有关限制 Pod 网络流量的详细信息，请参阅[在 AKS 中使用网络策略保护 Pod 之间的流量][network-policies]。

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity
[aad-pod-identity-exception]: https://azure.github.io/aad-pod-identity/docs/configure/application_exception

<!-- LINKS - internal -->
[policy-recommendations]: ../governance/policy/concepts/policy-for-kubernetes.md
[policy-limitations]: ../governance/policy/concepts/policy-for-kubernetes.md?#limitations
[kubernetes-policy-reference]: ../governance/policy/concepts/policy-for-kubernetes.md
[policy-samples]: ./policy-reference.md#microsoftcontainerservice
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policies]: use-network-policies.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-disable-addons]: /cli/azure/aks#az-aks-disable-addons
[az-policy-assignment-delete]: /cli/azure/policy/assignment#az-policy-assignment-delete
