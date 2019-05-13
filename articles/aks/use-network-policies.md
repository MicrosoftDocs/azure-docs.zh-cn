---
title: 在 Azure Kubernetes 服务 (AKS) 中使用网络策略保护 Pod
description: 了解如何保护流量流入和 pod 流通过使用 Azure Kubernetes 服务 (AKS) 中的 Kubernetes 网络策略
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 05/06/2019
ms.author: iainfou
ms.openlocfilehash: a0512806ec797f43fc54d8a28a7cbadf86faf1d9
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230018"
---
# <a name="secure-traffic-between-pods-using-network-policies-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中使用网络策略保护 Pod 之间的流量

在 Kubernetes 中运行最新的基于微服务的应用程序时，通常想要控制哪些组件可以相互通信。 应为流量可以在 Azure Kubernetes 服务 (AKS) 群集中的 pod 之间流动，如何应用最小特权原则。 让我们假设你可能想要阻止直接到后端应用程序的流量。 *网络策略*在 Kubernetes 中的功能允许您定义的群集中的 pod 之间入口和出口流量的规则。

本文介绍如何安装网络策略引擎和创建 Kubernetes 网络策略来控制在 AKS 中的 pod 之间的流量流。 网络策略应仅用于基于 Linux 的节点和 AKS 中的 pod。

## <a name="before-you-begin"></a>开始之前

你需要 Azure CLI 版本 2.0.61 或更高版本安装和配置。 运行  `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅 [安装 Azure CLI][install-azure-cli]。

> [!TIP]
> 如果在预览期间使用的网络策略功能，我们建议您[创建新的群集](#create-an-aks-cluster-and-enable-network-policy)。
> 
> 如果你想要继续使用在预览期间使用网络策略的现有测试群集，将群集升级到最新 GA 版本新 Kubernetes 版本，然后将部署下面的 YAML 清单，若要修复崩溃的指标服务器和 Kubernetes仪表板。 此修补程序才需要使用的是 Calico 网络策略引擎的群集。
>
> 作为安全性最佳实践，[查看此 YAML 清单的内容][ calico-aks-cleanup]若要了解什么部署到 AKS 群集。
>
> `kubectl delete -f https://raw.githubusercontent.com/Azure/aks-engine/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml`

## <a name="overview-of-network-policy"></a>网络策略概述

AKS 群集中的所有 pod 可以发送和接收流量，而不受限制，默认情况下。 为了提高安全性，可定义用来控制流量流的规则。 后端应用程序通常仅公开所需的前端服务，例如。 或者，数据库组件仅可以访问连接到它们的应用程序层。

网络策略是一种 Kubernetes 规范，用于定义访问策略的 Pod 之间的通信。 使用网络策略，定义一组有序的规则以发送和接收流量，将它们应用于一系列匹配一个或多个标签选择器的 pod。

按 YAML 清单定义这些网络策略规则。 网络策略可以是清单的作为更广泛，还会创建部署或服务的一部分。

### <a name="network-policy-options-in-aks"></a>在 AKS 中的网络策略选项

Azure 提供两种方法来实现网络策略。 创建 AKS 群集时选择的网络策略选项。 创建群集后，无法更改策略选项：

* Azure 的实现，调用*Azure 网络策略*。
* *网络策略 calico*，打开源网络和网络安全解决方案由成立[Tigera][tigera]。

这两个实现使用 Linux *IPTables*强制执行指定的策略。 策略将转换为组允许的和不允许 IP 对。 然后，这些对进行编程作为 IPTable 筛选器规则。

网络策略仅适用于 Azure CNI （高级） 选项。 实现方式不同，两个选项：

* *Azure 网络策略*-Azure CNI 设置在节点内部网络的 VM 主机的桥梁。 将数据包传递桥时应用筛选规则。
* *Calico 网络策略*-Azure CNI 设置了本地内核在节点内部流量的路由。 将策略应用 pod 的网络接口上。

### <a name="differences-between-azure-and-calico-policies-and-their-capabilities"></a>Azure 和 Calico 策略和及其功能之间的差异

| 功能                               | Azure                      | Calico                      |
|------------------------------------------|----------------------------|-----------------------------|
| 支持的平台                      | Linux                      | Linux                       |
| 支持的网络选项             | Azure CNI                  | Azure CNI                   |
| 与 Kubernetes 规范的符合性 | 支持的所有策略类型 |  支持的所有策略类型 |
| 其他功能                      | 无                       | 扩展包括全局网络策略、 全局网络设置和主机终结点的策略模型。 有关使用的详细信息`calicoctl`CLI 来管理这些扩展功能，请参阅[calicoctl 用户参考][calicoctl]。 |
| 支持                                  | 受 Azure 支持和工程团队 | Calico 社区支持。 有关其他付费支持的详细信息，请参阅[项目 Calico 支持选项][calico-support]。 |
| 日志记录                                  | 规则已添加 / 删除 IPTables 中记录下每个主机上 */var/log/azure-npm.log* | 有关详细信息，请参阅[Calico 组件日志][calico-logs] |

## <a name="create-an-aks-cluster-and-enable-network-policy"></a>创建 AKS 群集并启用网络策略

若要查看在操作中，网络策略，让我们来创建，然后展开上一个策略，定义通信流：

* 拒绝流向 Pod 的所有流量。
* 允许基于 Pod 标签的流量。
* 允许基于命名空间的流量。

首先，让我们创建的 AKS 群集，支持网络策略。 创建群集时，可以仅启用的网络策略功能。 无法在现有 AKS 群集上启用网络策略。

若要使用网络策略和 AKS 群集，必须使用[Azure CNI 插件][ azure-cni]并定义自己的虚拟网络和子网。 如需详细了解如何规划所需的子网范围，请参阅[配置高级网络][use-advanced-networking]。

以下示例脚本：

* 创建虚拟网络和子网。
* 创建 Azure Active Directory (Azure AD) 与 AKS 群集配合使用的服务主体。
* 对虚拟网络的 AKS 服务主体授予“参与者”权限。
* 在定义虚拟网络中创建的 AKS 群集并启用网络策略。
    * *Azure*使用网络策略选项。 若要转而使用网络策略选项 Calico，使用`--network-policy calico`参数。

提供自己的安全 SP_PASSWORD。 您可以替换*RESOURCE_GROUP_NAME*并*CLUSTER_NAME*变量：

```azurecli-interactive
SP_PASSWORD=mySecurePassword
RESOURCE_GROUP_NAME=myResourceGroup-NP
CLUSTER_NAME=myAKSCluster
LOCATION=canadaeast

# Create a resource group
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

# Create a virtual network and subnet
az network vnet create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16

# Create a service principal and read in the application ID
SP_ID=$(az ad sp create-for-rbac --password $SP_PASSWORD --skip-assignment --query [appId] -o tsv)

# Wait 15 seconds to make sure that service principal has propagated
echo "Waiting for service principal to propagate..."
sleep 15

# Get the virtual network resource ID
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)

# Assign the service principal Contributor permissions to the virtual network resource
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor

# Get the virtual network subnet resource ID
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)

# Create the AKS cluster and specify the virtual network and service principal information
# Enable network policy by using the `--network-policy` parameter
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --network-policy azure
```

创建群集需要几分钟时间。 群集准备就绪后，配置`kubectl`通过使用连接到 Kubernetes 群集[az aks get-credentials 来获取凭据][ az-aks-get-credentials]命令。 此命令将下载凭据，并将 Kubernetes CLI 配置为使用这些凭据：

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

## <a name="deny-all-inbound-traffic-to-a-pod"></a>拒绝流向 Pod 的所有入站流量

定义规则以允许特定网络流量之前，请首先创建用于拒绝所有流量的网络策略。 此策略，可仅所需的流量开始列入允许列表的起始点。 此外，还可清楚看到，应用网络策略后，相关流量被丢弃。

对于示例应用程序环境和通信规则，让我们首先创建名为命名空间*开发*运行示例 pod:

```console
kubectl create namespace development
kubectl label namespace/development purpose=development
```

创建运行 NGINX 示例后端 pod。 此后端 pod 可以用于模拟的示例后端的基于 web 的应用程序。 在 development 命名空间中创建此 Pod，并且打开端口 80，以提供 Web 流量。 将 Pod 贴上标签：app=webapp,role=backend，以便我们可在下一节中使用网络策略定向到它：

```console
kubectl run backend --image=nginx --labels app=webapp,role=backend --namespace development --expose --port 80 --generator=run-pod/v1
```

创建另一个 pod，并附加一个终端会话，以便测试可以成功访问默认 NGINX 网页：

```console
kubectl run --rm -it --image=alpine network-policy --namespace development --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`以确认你是否可以访问默认 NGINX 网页：

```console
wget -qO- http://backend
```

以下示例输出显示默认 NGINX 网页，返回：

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

退出附加的终端会话。 自动删除测试 pod。

```console
exit
```

### <a name="create-and-apply-a-network-policy"></a>创建并应用网络策略

现在，您可以使用基本的 NGINX 网页上的示例后端 pod 确认后，创建了网络策略来拒绝所有流量。 创建名为 `backend-policy.yaml` 的文件并粘贴以下 YAML 清单。 使用此清单*podSelector*若要将策略附加到具有 pod *app:webapp，角色： 后端*标签，如示例 NGINX pod。 入口下未定义任何规则，因此将拒绝流向 Pod 的所有入站流量：

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress: []
```

使用网络策略应用所[kubectl 适用][ kubectl-apply]命令并指定 YAML 清单的名称：

```azurecli-interactive
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-network-policy"></a>测试网络策略


让我们看您是否可以使用 NGINX 网页上的后端 pod 试。 创建另一个测试 Pod，并附加一个终端会话：

```console
kubectl run --rm -it --image=alpine network-policy --namespace development --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`以查看是否可以访问默认 NGINX 网页。 这一次，将超时值设为 2 秒。 网络策略现在阻止所有入站的流量，因此无法加载页面，如下面的示例中所示：

```console
$ wget -qO- --timeout=2 http://backend

wget: download timed out
```

退出附加的终端会话。 自动删除测试 pod。

```console
exit
```

## <a name="allow-inbound-traffic-based-on-a-pod-label"></a>允许基于 Pod 标签的入站流量

在上一节中安排后端 NGINX pod 和创建了网络策略来拒绝所有流量。 让我们创建前端 pod 并更新网络策略以允许来自前端 pod 的流量。

更新网络策略，以允许来自具有标签 app:webapp,role:frontend 的 Pod 和任何命名空间的流量。 编辑以前*后端 policy.yaml*文件，并添加*matchLabels*入口规则，以便你清单看起来如下例所示：

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

> [!NOTE]
> 此网络策略使用 namespaceSelector 和 podSelector 元素作为入口规则。 YAML 语法非常重要的入口规则是累加性。 在此示例中，两个元素必须匹配要应用的入口规则。 以前的 Kubernetes 版本*1.12*可能不正确解释这些元素和限制网络流量，正如您期望。 有关此行为的详细信息，请参阅[的行为与选择器][policy-rules]。

使用更新的网络策略应用所[kubectl 适用][ kubectl-apply]命令并指定 YAML 清单的名称：

```azurecli-interactive
kubectl apply -f backend-policy.yaml
```

计划标记为一个 pod*应用 = webapp，角色 = 前端*和附加终端会话：

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace development --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`以查看是否可以访问默认 NGINX 网页：

```console
wget -qO- http://backend
```

因为入口规则允许流量与该标签的 pod*应用程序： web 应用，角色： 前端*，允许来自前端 pod 的流量。 以下示例输出显示返回的默认 NGINX 网页：

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

退出附加的终端会话。 自动删除 pod。

```console
exit
```

### <a name="test-a-pod-without-a-matching-label"></a>测试没有匹配标签的 Pod

网络策略允许来自标记为 app: webapp,role: frontend 的 Pod 的流量，但应拒绝其他所有流量。 让我们测试以查看其他 pod 而无需这些标签是否可以访问后端 NGINX pod。 创建另一个测试 Pod，并附加一个终端会话：

```console
kubectl run --rm -it --image=alpine network-policy --namespace development --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`以查看是否可以访问默认 NGINX 网页。 网络策略阻止的入站的流量，因此无法加载页面，如下面的示例中所示：

```console
$ wget -qO- --timeout=2 http://backend

wget: download timed out
```

退出附加的终端会话。 自动删除测试 pod。

```console
exit
```

## <a name="allow-traffic-only-from-within-a-defined-namespace"></a>仅允许来自定义命名空间的流量

在上述示例中，您创建的网络策略拒绝所有流量，然后再更新策略以允许来自特定标签的 pod 的流量。 另一个常见需求是将通信限制为仅在给定的命名空间内。 如果前面的示例中的流量*开发*命名空间中，创建如阻止来自另一个命名空间的流量的网络策略*生产*，到达 pod。

首先，创建新的命名空间，模拟生产命名空间：

```console
kubectl create namespace production
kubectl label namespace/production purpose=production
```

在具有标签 app=webapp,role=frontend 的 production 命名空间中计划测试 Pod。 附加终端会话：

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace production --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`以确认你是否可以访问默认 NGINX 网页：

```console
wget -qO- http://backend.development
```

由于在 pod 的标签匹配网络策略中当前允许范围，允许流量。 网络策略不会查看命名空间，只有 Pod 标签会查看。 以下示例输出显示返回的默认 NGINX 网页：

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

退出附加的终端会话。 自动删除测试 pod。

```console
exit
```

### <a name="update-the-network-policy"></a>更新网络策略

让我们更新入口规则*namespaceSelector*部分中，仅允许从流量*开发*命名空间。 编辑 backend-policy.yaml 清单文件，如以下示例所示：

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: development
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

在更复杂示例中，您可以定义多个入口规则，如*namespaceSelector* ，然后*podSelector*。

使用更新的网络策略应用所[kubectl 适用][ kubectl-apply]命令并指定 YAML 清单的名称：

```azurecli-interactive
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-updated-network-policy"></a>测试更新后的网络策略

计划中的另一个 pod*生产*命名空间和附加终端会话：

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace production --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`若要查看网络策略现在拒绝流量：

```console
$ wget -qO- --timeout=2 http://backend.development

wget: download timed out
```

退出测试 Pod：

```console
exit
```

从被拒绝的流量*生产*命名空间、 计划重新测试 pod*开发*命名空间和附加终端会话：

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace development --generator=run-pod/v1
```

在 shell 提示符下，使用`wget`若要查看网络策略允许的流量：

```console
wget -qO- http://backend
```

允许流量，因为在 pod 安排命名空间中的匹配项允许范围的网络策略中。 以下示例输出显示了返回的默认 NGINX 网页：

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

退出附加的终端会话。 自动删除测试 pod。

```console
exit
```

## <a name="clean-up-resources"></a>清理资源

在本文中，我们将创建两个命名空间，并将网络策略应用。 若要清理这些资源，请使用[kubectl 删除][ kubectl-delete]命令并指定资源名称：

```console
kubectl delete namespace production
kubectl delete namespace development
```

## <a name="next-steps"></a>后续步骤

有关网络资源的详细信息，请参阅[网络应用程序在 Azure Kubernetes 服务 (AKS) 的概念][concepts-network]。

若要了解有关策略的详细信息，请参阅[Kubernetes 网络策略][kubernetes-network-policies]。

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[policy-rules]: https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors
[aks-github]: https://github.com/azure/aks/issues
[tigera]: https://www.tigera.io/
[calicoctl]: https://docs.projectcalico.org/v3.6/reference/calicoctl/
[calico-support]: https://www.projectcalico.org/support
[calico-logs]: https://docs.projectcalico.org/v3.6/maintenance/component-logs
[calico-aks-cleanup]: https://github.com/Azure/aks-engine/blob/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[use-advanced-networking]: configure-advanced-networking.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[concepts-network]: concepts-network.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
