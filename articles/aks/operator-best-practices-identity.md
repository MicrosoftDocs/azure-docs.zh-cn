---
title: 操作员最佳做法 - Azure Kubernetes 服务 (AKS) 中的标识
description: 了解有关为 Azure Kubernetes 服务 (AKS) 中的群集管理身份验证和授权的群集操作员最佳做法
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: conceptual
ms.date: 04/24/2019
ms.author: iainfou
ms.openlocfilehash: f98e38556458b8d8a675d1e3f985aacfca022082
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074165"
---
# <a name="best-practices-for-authentication-and-authorization-in-azure-kubernetes-service-aks"></a>有关 Azure Kubernetes 服务 (AKS) 中的身份验证和授权的最佳做法

在 Azure Kubernetes 服务 (AKS) 中部署和维护群集时，需要实施相应的方法来管理对资源和服务的访问。 如果不采用这些控制措施，帐户可能会有权访问它们不需要访问的资源和服务。 此外，可能难以跟踪用于做出更改的凭据集。

本最佳做法文章重点介绍群集操作员如何管理 AKS 群集的访问和标识。 在本文中，学习如何：

> [!div class="checklist"]
> * 使用 Azure Active Directory 对 AKS 群集用户进行身份验证
> * 使用基于角色的访问控制 (RBAC) 来控制对资源的访问
> * 使用托管标识在其他服务中进行自我身份验证

## <a name="use-azure-active-directory"></a>使用 Azure Active Directory

**最佳做法指导** - 使用 Azure AD 集成部署 AKS 群集。 使用 Azure AD 可以集中化标识管理组件。 访问 AKS 群集时，用户帐户或组状态的任何更改会自动更新。 根据下一部分所述，使用角色或群集角色和绑定，将用户或组的权限范围限定为所需的最少量权限。

Kubernetes 群集的开发人员和应用程序所有者需要访问不同的资源。 Kubernetes 不提供标识管理解决方案来控制哪些用户可与哪些资源交互。 通常，你会将群集与现有的标识解决方案相集成。 Azure Active Directory (AD) 提供企业就绪的标识管理解决方案，并可与 AKS 群集相集成。

使用 AKS 中与 Azure AD 集成的群集，创建角色或群集角色用于定义对资源的访问权限。 然后，从 Azure AD 将角色绑定到用户或组。 下一部分将介绍这些 Kubernetes 基于角色的访问控制 (RBAC)。 下图显示了 Azure AD 集成，以及如何控制对资源的访问：

![与 AKS 集成的 Azure Active Directory 的群集级身份验证](media/operator-best-practices-identity/cluster-level-authentication-flow.png)

1. 开发人员在 Azure AD 中进行验证身份。
1. Azure AD 令牌颁发终结点颁发访问令牌。
1. 开发人员使用 Azure AD 令牌执行操作，例如 `kubectl create pod`
1. Kubernetes 使用 Azure Active Directory 验证令牌，并提取开发人员的组成员身份。
1. 应用 Kubernetes 基于角色的访问控制 (RBAC) 和群集策略。
1. 开发人员的请求成功或失败，具体取决于前面的 Azure AD 组成员身份和 Kubernetes RBAC 验证以及策略。

若要创建使用 Azure AD 的 AKS 群集，请参阅[将 Azure Active Directory 与 AKS 集成][aks-aad]。

## <a name="use-role-based-access-controls-rbac"></a>使用基于角色的访问控制 (RBAC)

**最佳做法指导** - 使用 Kubernetes RBAC 定义用户或组对群集中的资源拥有的权限。 创建角色和绑定，用于分配所需的最少量权限。 与 Azure AD 集成，使用户状态或组成员身份的任何更改可自动更新，并使群集资源访问权限保持最新状态。

在 Kubernetes 中，可以针对群集中的资源提供精细访问控制。 可在群集级别或者针对特定的命名空间定义权限。 可以定义能够使用哪些权限管理哪些资源。 这些角色将结合绑定应用到用户或组。 有关角色、群集角色和绑定的详细信息，请参阅 [Azure Kubernetes 服务 (AKS) 的访问和标识选项][aks-concepts-identity]。

例如，可以创建一个角色，用于授予对名为 *finance-app* 的命名空间中的资源的完全访问权限，如以下示例 YAML 清单中所示：

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role
  namespace: finance-app
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
```

绑定的 Azure AD 用户，然后会创建 RoleBinding *developer1\@contoso.com*到 RoleBinding，如下面的 YAML 清单中所示：

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role-binding
  namespace: finance-app
subjects:
- kind: User
  name: developer1@contoso.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: finance-app-full-access-role
  apiGroup: rbac.authorization.k8s.io
```

当*developer1\@contoso.com*进行身份验证必须针对 AKS 群集中的资源的完全权限*财务应用*命名空间。 这样，即可以逻辑方式隔离和控制对资源的访问权限。 应根据上一部分中所述，将 Kubernetes RBAC 与 Azure AD 集成结合使用。

若要了解如何使用 Azure AD 组来控制使用 RBAC 的 Kubernetes 资源的访问权限，请参阅[控制在 AKS 中使用基于角色的访问控制和 Azure Active Directory 标识的群集资源的访问权限][ azure-ad-rbac].

## <a name="use-pod-identities"></a>使用 pod 标识

**最佳做法指导** - 不要在 pod 或容器映像中使用固定的凭据，因为它们存在泄漏或滥用的风险。 应该使用 pod 标识通过中心 Azure AD 标识解决方案来自动请求访问权限。 Pod 标识适用于与 Linux pod 和仅容器映像配合使用。

当 pod 需要访问其他 Azure 服务（例如 Cosmos DB、Key Vault 或 Blob 存储）时，pod 需要访问凭据。 可以使用容器映像定义这些访问凭据或将其注入为 Kubernetes 机密，但需要手动创建并分配这些凭据。 通常，凭据会在不同的 pod 之间重复使用，并且不会定期轮换。

使用 Azure 资源的托管标识（目前作为关联的 AKS 开源项目来实现）可以通过 Azure AD 自动请求服务访问权限。 不要手动定义 pod 的凭据。pod 会实时请求访问令牌，并可以使用该令牌来访问仅为它们分配的服务。 在 AKS 中，群集操作员会部署两个组件，以允许 pod 使用托管标识：

* **节点管理标识 (NMI) 服务器**是在 AKS 群集中每个节点上作为守护程序集运行的 pod。 NMI 服务器侦听发送到 Azure 服务的 pod 请求。
* **托管标识控制器 (MIC)** 是一个中心 pod，它有权查询 Kubernetes API 服务器，并检查对应于某个 pod 的 Azure 标识映射。

当 pod 请求 Azure 服务访问权限时，网络规则会将流量重定向到节点管理标识 (NMI) 服务器。 NMI 服务器根据远程地址识别请求 Azure 服务访问权限的 pod，并查询托管标识控制器 (MIC)。 MIC 检查 AKS 群集中的 Azure 标识映射，然后，NMI 服务器根据 pod 的标识映射从 Azure Active Directory (AD) 请求访问令牌。 Azure AD 提供对 NMI 服务器的访问权限，该访问权限将返回给 pod。 然后，pod 可以使用此访问令牌请求对 Azure 中服务的访问权限。

在以下示例中，开发人员创建了一个使用托管标识请求 Azure SQL Server 实例访问权限的 pod：

![pod 标识可让 pod 自动请求对其他服务的访问权限](media/operator-best-practices-identity/pod-identities.png)

1. 群集操作员首先创建一个服务帐户，当 pod 请求服务访问权限时，可以使用该帐户来映射标识。
1. 部署 NMI 服务器和 MIC，以便将访问令牌的任何 pod 请求中继到 Azure AD。
1. 开发人员使用托管标识部署一个 pod，该 pod 可通过 NMI 服务器请求访问令牌。
1. 该令牌将返回给 pod，并用于访问 Azure SQL Server 实例。

> [!NOTE]
> 托管的 pod 标识是一个开放源代码项目，并不受 Azure 技术支持。

若要使用 pod 标识，请参阅 [Kubernetes 应用程序的 Azure Active Directory 标识][aad-pod-identity]。

## <a name="next-steps"></a>后续步骤

本最佳做法文章重点介绍了群集和资源的身份验证与授权。 若要实施其中的某些最佳做法，请参阅以下文章：

* [将 Azure Active Directory 与 AKS 集成][aks-aad]
* [将 Azure 资源的托管标识与 AKS 配合使用][aad-pod-identity]

有关 AKS 中的群集操作的详细信息，请参阅以下最佳做法：

* [多租户和群集隔离][aks-best-practices-scheduler]
* [基本 Kubernetes 计划程序功能][aks-best-practices-scheduler]
* [高级 Kubernetes 计划程序功能][aks-best-practices-advanced-scheduler]

<!-- EXTERNAL LINKS -->
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-aad]: azure-ad-integration-cli.md
[managed-identities:]: ../active-directory/managed-identities-azure-resources/overview.md
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[azure-ad-rbac]: azure-ad-rbac.md
