---
title: 将 Azure Active Directory 与 Azure Kubernetes Service 集成
description: 如何创建支持 Azure Active Directory 的 Azure Kubernetes Service (AKS) 群集。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 04/26/2019
ms.author: iainfou
ms.openlocfilehash: 026c0eefc0c4fe31e72ecad91a4a7b558f367487
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192105"
---
# <a name="integrate-azure-active-directory-with-azure-kubernetes-service"></a>将 Azure Active Directory 与 Azure Kubernetes Service 集成

可将 Azure Kubernetes Service (AKS) 配置为使用 Azure Active Directory (AD) 进行用户身份验证。 在此配置中，你可以登录到 AKS 群集使用 Azure Active Directory 身份验证令牌。 此外，群集管理员是可以配置基于用户的标识或目录的组成员身份的 Kubernetes 基于角色的访问控制 (RBAC)。

本文介绍如何部署的先决条件 AKS 和 Azure AD 中，然后介绍如何部署 Azure AD 启用群集，并使用 Azure 门户在 AKS 群集中创建基本的 RBAC 角色。 此外可以[完成这些步骤使用 Azure CLI][azure-ad-cli]。

以下限制适用：

- 只有在创建新的启用 RBAC 的群集时，才能启用 Azure AD。 不能在现有 AKS 群集上启用 Azure AD。
- *来宾*用户在 Azure AD 中，此类是使用联合的登录中从另一个目录，不支持。

## <a name="authentication-details"></a>身份验证详细信息

使用 OpenID Connect 向 AKS 群集提供 Azure AD 身份验证。 OpenID Connect 是构建在 OAuth 2.0 协议顶层的标识层。 有关 OpenID Connect 的详细信息，请参阅 [Open ID Connect 文档][open-id-connect]。

在 Kubernetes 群集内部，使用 Webhook 令牌身份验证来验证身份验证令牌。 Webhook 令牌身份验证作为 AKS 群集的一部分进行配置和管理。 有关 Webhook 令牌身份验证的详细信息，请参阅 [Webhook 身份验证文档][kubernetes-webhook]。

若要提供 AKS 群集的 Azure AD 身份验证，请创建两个 Azure AD 应用程序。 第一个应用程序是一个提供用户身份验证的服务器组件。 第二个应用程序是适用于身份验证的 CLI 提示你时，将使用的客户端组件。 此客户端应用程序使用实际客户端提供的凭据进行身份验证的服务器应用程序。

> [!NOTE]
> 将 Azure AD 配置用于 AKS 身份验证时，会配置两个 Azure AD 应用程序。 若要为每个应用程序委派权限的步骤必须由 Azure 租户管理员完成。

## <a name="create-server-application"></a>创建服务器应用程序

第一个 Azure AD 应用程序用于获取用户的 Azure AD 组成员身份。 在 Azure 门户中创建此应用程序。

1. 选择**Azure Active Directory** > **应用注册** > **新注册**。

    * 为应用程序名称，如*AKSAzureADServer*。
    * 有关**支持的帐户类型**，选择*此组织目录中的帐户*。
    * 选择*Web*有关**重定向 URI**键入，然后输入任何格式的 URI 值，例如*https://aksazureadserver*。
    * 选择**注册**完成。

1. 选择“清单”，将 `groupMembershipClaims` 值编辑为 `"All"`。

    ![将组成员身份更新为“所有”](media/aad-integration/edit-manifest.png)

    **保存**更新完成后。

1. 在 Azure AD 应用程序的左侧导航窗格中，选择**证书和机密**。

    * 选择 **+ 新客户端机密**。
    * 添加密钥的说明，例如*AKS Azure AD 服务器*。 选择到期时间，然后选择**添加**。
    * 记下密钥值。 它仅显示此初始时间。 在部署 Azure AD 启用 AKS 群集时，此值称为`Server application secret`。

1. 在 Azure AD 应用程序的左侧导航窗格中，选择**API 的权限**，然后选择 **+ 添加权限**。

    * 下**Microsoft Api**，选择*Microsoft Graph*。
    * 选择**委托的权限**，然后勾选旁边**Directory > Directory.Read.All （读取目录数据）**。
        * 如果默认委派权限**用户 > User.Read (登录并读取用户配置文件)** 不存在，请将此权限的检查。
    * 选择**应用程序权限**，然后勾选旁边**Directory > Directory.Read.All （读取目录数据）**。

        ![设置 graph 权限](media/aad-integration/graph-permissions.png)

    * 选择**添加权限**以保存的更新。

    * 下**授予许可**部分中，选择**授予管理员同意**。 此按钮将灰显并且不可用，如果当前帐户不是租户管理员联系。

        成功授予权限后，门户中会显示以下通知：

        ![权限授予成功的通知](media/aad-integration/permissions-granted.png)

1. 在 Azure AD 应用程序的左侧导航窗格中，选择**公开一个 API**，然后选择 **+ 添加作用域**。
    
    * 设置*作用域名称*，*管理员许可显示名称*，并*管理员许可说明*，如*AKSAzureADServer*。
    * 请确保**状态**设置为*已启用*。

        ![将服务器应用程序公开为与其他服务一起使用的 API](media/aad-integration/expose-api.png)

    * 选择**添加作用域**。

1. 返回到应用程序**概述**页上，并记下**应用程序 （客户端） ID**。 在部署 Azure AD 启用 AKS 群集时，此值称为`Server application ID`。

   ![获取应用程序 ID](media/aad-integration/application-id.png)

## <a name="create-client-application"></a>创建客户端应用程序

使用 Kubernetes CLI 进行日志记录时使用第二个 Azure AD 应用程序 (`kubectl`)。

1. 选择**Azure Active Directory** > **应用注册** > **新注册**。

    * 为应用程序名称，如*AKSAzureADClient*。
    * 有关**支持的帐户类型**，选择*此组织目录中的帐户*。
    * 选择*Web*有关**重定向 URI**键入，然后输入任何格式的 URI 值，例如*https://aksazureadclient*。
    * 选择**注册**完成。

1. 在 Azure AD 应用程序的左侧导航窗格中，选择**API 的权限**，然后选择 **+ 添加权限**。

    * 选择**我的 Api**，然后选择 Azure AD 服务器应用程序创建在上一步骤中，例如*AKSAzureADServer*。
    * 选择**委派权限**，然后将放在 Azure AD 服务器应用程序旁的检查。

        ![配置应用程序权限](media/aad-integration/select-api.png)

    * 选择**添加权限**。

    * 下**授予许可**部分中，选择**授予管理员同意**。 此按钮将灰显并且不可用，如果当前帐户不是租户管理员联系。

        成功授予权限后，门户中会显示以下通知：

        ![权限授予成功的通知](media/aad-integration/permissions-granted.png)

1. 在 Azure AD 应用程序的左侧导航窗格中，记**应用程序 ID**。 部署支持 Azure AD 的 AKS 群集时，此值称为 `Client application ID`。

   ![获取应用程序 ID](media/aad-integration/application-id-client.png)

## <a name="get-tenant-id"></a>获取租户 ID

最后，获取 Azure 租户的 ID。 创建 AKS 群集时，使用此值。

在 Azure 门户中，选择“Azure Active Directory” > “属性”并记下“目录 ID”。 创建 Azure AD 启用 AKS 群集时，此值称为`Tenant ID`。

![获取 Azure 租户 ID](media/aad-integration/tenant-id.png)

## <a name="deploy-cluster"></a>部署群集

使用 [az group create][az-group-create] 命令为 AKS 群集创建资源组。

```azurecli
az group create --name myResourceGroup --location eastus
```

使用 [az aks create][az-aks-create] 命令部署群集。 将收集时创建 Azure AD 应用程序的服务器应用程序 ID 和机密、 客户端应用 ID 和租户 ID 的值替换为下面的示例命令中的值：

```azurecli
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --generate-ssh-keys \
  --aad-server-app-id b1536b67-29ab-4b63-b60f-9444d0c15df1 \
  --aad-server-app-secret wHYomLe2i1mHR2B3/d4sFrooHwADZccKwfoQwK2QHg= \
  --aad-client-app-id 8aaf8bd5-1bdd-4822-99ad-02bfaa63eea7 \
  --aad-tenant-id 72f988bf-0000-0000-0000-2d7cd011db47
```

花费几分钟才能创建 AKS 群集。

## <a name="create-rbac-binding"></a>创建 RBAC 绑定

在对 AKS 群集使用 Azure Active Directory 帐户之前，需要创建角色绑定或群集角色绑定。 “角色”定义要授予的权限，“绑定”将这些权限应用于目标用户。 这些分配可应用于特定命名空间或整个群集。 有关详细信息，请参阅[使用 RBAC 授权][rbac-authorization]。

首先，使用[az aks get-credentials 来获取凭据][ az-aks-get-credentials]命令`--admin`参数，以在登录到具有管理员访问权限的群集。

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

接下来，创建你想要授予 AKS 群集的访问权限的 Azure AD 帐户 ClusterRoleBinding。 下面的示例会对群集中的所有命名空间的帐户完全访问权限。

- 如果授予用户的 RBAC 绑定是相同的 Azure AD 租户中，基于分配权限的用户主体名称 (UPN)。 继续到步骤创建 ClusterRuleBinding YAML 清单。

- 如果用户是在不同的 Azure AD 中租户、 查询和使用*objectId*属性改为。 如果需要请获取*objectId*的所需的用户帐户使用[az ad 用户显示][ az-ad-user-show]命令。 提供所需的帐户的用户主体名称 (UPN):

    ```azurecli-interactive
    az ad user show --upn-or-object-id user@contoso.com --query objectId -o tsv
    ```

创建一个文件（例如 *rbac-aad-user.yaml*），然后粘贴以下内容。 在最后一行中，替换*userPrincipalName_or_objectId* UPN 或对象 id 为具体取决于用户是相同的 Azure AD 租户，还是不。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: userPrincipalName_or_objectId
```

使用 [kubectl apply][kubectl-apply] 命令应用绑定，如以下示例所示：

```console
kubectl apply -f rbac-aad-user.yaml
```

此外，可为 Azure AD 组的所有成员创建角色绑定。 使用组对象 ID 指定 Azure AD 组，如以下示例所示。 创建一个文件（例如 *rbac-aad-group.yaml*），然后粘贴以下内容。 将组对象 ID 更新为 Azure AD 租户中的某个组对象 ID：

 ```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
   kind: Group
   name: "894656e1-39f8-4bfe-b16a-510f61af6f41"
```

使用 [kubectl apply][kubectl-apply] 命令应用绑定，如以下示例所示：

```console
kubectl apply -f rbac-aad-group.yaml
```

有关使用 RBAC 保护 Kubernetes 群集的详细信息，请参阅[使用 RBAC 授权][rbac-authorization]。

## <a name="access-cluster-with-azure-ad"></a>使用 Azure AD 访问群集

接下来，使用 [az aks get-credentials][az-aks-get-credentials] 命令提取非管理员用户的上下文。

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

在您运行`kubectl`命令时，系统会提示使用 Azure 进行身份验证。 请按照屏幕说明完成该过程中，如下面的示例中所示：

```console
$ kubectl get nodes

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BUJHWDGNL to authenticate.

NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-79590246-0   Ready     agent     1h        v1.13.5
aks-nodepool1-79590246-1   Ready     agent     1h        v1.13.5
aks-nodepool1-79590246-2   Ready     agent     1h        v1.13.5
```

完成后，缓存身份验证令牌。 您仅再次提示进行签名的令牌已过期时或重新创建的 Kubernetes 配置文件中。

如果在成功登录后看到授权错误消息，请检查是否存在以下问题：
1. 用户在登录原样不 （这种情况下通常是这种情况时使用不同的目录中的联合的帐户） 在 Azure AD 实例中的来宾。
2. 用户不是 200 多个组的成员。
3. 在服务器的应用程序注册中定义的机密使用-aad 服务器应用程序机密配置的值不匹配

```console
error: You must be logged in to the server (Unauthorized)
```

## <a name="next-steps"></a>后续步骤

若要使用 Azure AD 用户和组来控制对群集资源的访问，请参阅[控制在 AKS 中使用基于角色的访问控制和 Azure AD 标识的群集资源的访问权限][azure-ad-rbac]。

有关如何保护 Kubernetes 群集的详细信息，请参阅[适用于 AKS 的访问和标识选项)][rbac-authorization]。

标识和资源控制的最佳实践，请参阅[身份验证和授权在 AKS 中的最佳做法][operator-best-practices-identity]。

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v1-protocols-openid-connect-code.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
