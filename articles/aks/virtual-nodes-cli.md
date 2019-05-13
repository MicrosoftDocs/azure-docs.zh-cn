---
title: 在 Azure Kubernetes 服务 (AKS) 中使用 Azure CLI 创建虚拟节点
description: 了解如何通过 Azure CLI 创建使用虚拟节点运行 Pod 的 Azure Kubernetes 服务 (AKS) 群集。
services: container-service
author: iainfoulds
ms.topic: conceptual
ms.service: container-service
ms.date: 05/06/2019
ms.author: iainfou
ms.openlocfilehash: 7631a2d6aef2efedf30c0b9015913c89949d4c29
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506963"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-using-the-azure-cli"></a>创建 Azure Kubernetes 服务 (AKS) 群集并将其配置为通过 Azure CLI 使用虚拟节点

若要快速缩放 Azure Kubernetes 服务 (AKS) 群集中的应用程序工作负载，可以使用虚拟节点。 使用虚拟节点可快速预配 Pod，并且只需对其执行时间按秒付费。 无需等待 Kubernetes 群集自动缩放程序部署 VM 计算节点来运行其他 Pod。 仅在使用 Linux pod 和节点支持的虚拟节点。

本文介绍如何创建和配置虚拟网络资源和 AKS 群集，然后启用虚拟节点。

## <a name="before-you-begin"></a>开始之前

ACI 和 AKS 群集中运行的 Pod 可以借助虚拟节点进行网络通信。 若要提供此通信，应创建虚拟网络子网并分配委派的权限。 虚拟节点仅适用于使用高级网络创建的 AKS 群集。 默认情况下，AKS 群集是使用基本网络创建的。 本文介绍如何创建虚拟网络和子网，然后部署使用高级网络的 AKS 群集。

如果以前没有使用过 ACI，请在订阅中注册服务提供程序。 可以使用 [az provider list][az-provider-list] 命令检查 ACI 提供程序注册的状态，如下面的示例所示：

```azurecli-interactive
az provider list --query "[?contains(namespace,'Microsoft.ContainerInstance')]" -o table
```

Microsoft.ContainerInstance 提供程序应报告为“已注册”，如下面的示例输出所示：

```
Namespace                    RegistrationState
---------------------------  -------------------
Microsoft.ContainerInstance  Registered
```

如果提供程序显示为“未注册”，请使用 [az provider register][az-provider-register] 注册提供程序，如下面的示例所示：

```azurecli-interactive
az provider register --namespace Microsoft.ContainerInstance
```

## <a name="regional-availability"></a>区域可用性

虚拟节点部署支持以下区域：

* 澳大利亚东部 (australiaeast)
* 美国中部 (centralus)
* 美国东部 (eastus)
* 美国东部 2 (eastus2)
* 日本东部 (japaneast)
* 北欧 (northeurope)
* 东南亚 (southeastasia)
* 美国中西部 (westcentralus)
* 西欧 (westeurope)
* 美国西部 (westus)
* 美国西部 2 (westus2)

## <a name="known-limitations"></a>已知限制
虚拟节点功能是很大程度取决于 ACI 的功能集。 尚不支持以下方案的虚拟节点

* 使用服务主体来请求 ACR 映像。 [解决方法](https://github.com/virtual-kubelet/virtual-kubelet/blob/master/providers/azure/README.md#Private-registry)是使用[Kubernetes 机密](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)
* [虚拟网络限制](../container-instances/container-instances-vnet.md)包括 VNet 对等互连、 Kubernetes 网络策略和网络安全组与 internet 的出站流量。
* Init 容器
* [主机别名](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)
* [参数](../container-instances/container-instances-exec.md#restrictions)exec 在 ACI 中的
* [Daemonset](concepts-clusters-workloads.md#statefulsets-and-daemonsets)将不将 pod 部署到虚拟节点

## <a name="launch-azure-cloud-shell"></a>启动 Azure Cloud Shell

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。

若要打开 Cloud Shell，请从代码块的右上角选择“试一试”。 也可以通过转到 [https://shell.azure.com/bash](https://shell.azure.com/bash) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

如果希望在本地安装并使用 CLI，则本文需要 Azure CLI 版本 2.0.49 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑组。 使用 [az group create][az-group-create] 命令创建资源组。 以下示例在 westus 位置创建名为 myResourceGroup 的资源组。

```azurecli-interactive
az group create --name myResourceGroup --location westus
```

## <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [az network vnet create][az-network-vnet-create] 命令创建虚拟网络。 以下示例使用 10.0.0.0/8 地址前缀创建一个名为 myVnet 的虚拟网络和一个名为 myAKSSubnet 的子网。 此子网的地址前缀默认为 10.240.0.0/16：

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16
```

现在，使用 [az network vnet subnet create][az-network-vnet-subnet-create] 命令为虚拟节点创建其他子网。 以下示例使用 10.241.0.0/16 地址前缀创建一个名为 myVirtualNodeSubnet 的子网。

```azurecli-interactive
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name myVirtualNodeSubnet \
    --address-prefixes 10.241.0.0/16
```

## <a name="create-a-service-principal"></a>创建服务主体

若要允许 AKS 群集与其他 Azure 资源交互，请使用 Azure Active Directory 服务主体。 可以通过 Azure CLI 或门户自动创建此服务主体，也可以预先创建一个服务主体并分配其他权限。

使用 [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] 命令创建服务主体。 `--skip-assignment` 参数限制分配任何其他权限。

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

输出类似于以下示例：

```
{
  "appId": "bef76eb3-d743-4a97-9534-03e9388811fc",
  "displayName": "azure-cli-2018-11-21-18-42-00",
  "name": "http://azure-cli-2018-11-21-18-42-00",
  "password": "1d257915-8714-4ce7-a7fb-0e5a5411df7f",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

记下 *appId* 和 *password*。 后续步骤会用到这些值。

## <a name="assign-permissions-to-the-virtual-network"></a>分配对虚拟网络的权限

若要允许群集使用和管理虚拟网络，必须授予 AKS 服务主体使用网络资源的适当权限。

首先，使用 [az network vnet show][az-network-vnet-show] 获取虚拟网络资源 ID：

```azurecli-interactive
az network vnet show --resource-group myResourceGroup --name myVnet --query id -o tsv
```

若要授予 AKS 群集使用虚拟网络的适当访问权限，请使用 [az role assignment create][az-role-assignment-create] 命令创建角色分配。 将 `<appId`> 和 `<vnetId>` 替换为在前两个步骤中收集的值。

```azurecli-interactive
az role assignment create --assignee <appId> --scope <vnetId> --role Contributor
```

## <a name="create-an-aks-cluster"></a>创建 AKS 群集

将 AKS 群集部署到上一步中创建的 AKS 子网。 使用 [az network vnet subnet show][az-network-vnet-subnet-show] 获取此子网的 ID：

```azurecli-interactive
az network vnet subnet show --resource-group myResourceGroup --vnet-name myVnet --name myAKSSubnet --query id -o tsv
```

使用 [az aks create][az-aks-create] 命令创建 AKS 群集。 以下示例创建一个具有一个节点的名为 myAKSCluster 的群集。 将 `<subnetId>` 替换为上一步中获取的 ID，然后将 `<appId>` 和 `<password>` 替换为 

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id <subnetId> \
    --service-principal <appId> \
    --client-secret <password>
```

几分钟后，该命令完成并返回有关群集的 JSON 格式信息。

## <a name="enable-virtual-nodes-addon"></a>启用虚拟节点外接程序

若要启用虚拟节点，现在使用 [az aks enable-addons][az-aks-enable-addons] 命令。 以下示例使用上一步中创建的名为 myVirtualNodeSubnet  的子网：

```azurecli-interactive
az aks enable-addons \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --addons virtual-node \
    --subnet-name myVirtualNodeSubnet
```

## <a name="connect-to-the-cluster"></a>连接至群集

若要将 `kubectl` 配置为连接到 Kubernetes 群集，请使用 [az aks get-credentials][az-aks-get-credentials] 命令。 此步骤下载凭据，并将 Kubernetes CLI 配置为使用这些凭据。

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

若要验证到群集的连接，请使用 [kubectl get][kubectl-get] 命令返回群集节点的列表。

```console
kubectl get nodes
```

以下示例输出依次显示了所创建的单个 VM 节点以及用于 Linux 的虚拟节点 virtual-node-aci-linux：

```
$ kubectl get nodes

NAME                          STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux        Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0      Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>部署示例应用

创建名为 `virtual-node.yaml` 的文件，并将其复制到以下 YAML 中。 若要在节点上计划容器，需定义 [nodeSelector][node-selector] 和 [toleration][toleration]。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aci-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aci-helloworld
  template:
    metadata:
      labels:
        app: aci-helloworld
    spec:
      containers:
      - name: aci-helloworld
        image: microsoft/aci-helloworld
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
```

使用 [kubectl apply][kubectl-apply] 命令运行该应用程序。

```console
kubectl apply -f virtual-node.yaml
```

使用带有 `-o wide` 参数的 [kubectl get pods][kubectl-get] 命令输出 Pod 和计划节点的列表。 请注意，已在 `virtual-node-aci-linux` 节点上计划 `aci-helloworld` pod。

```
$ kubectl get pods -o wide

NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
aci-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

系统从被委派用于虚拟节点的 Azure 虚拟网络子网中为该 Pod 分配了一个内部 IP 地址。

> [!NOTE]
> 如果使用存储在 Azure 容器注册表中的映像，请[配置并使用 Kubernetes 机密][acr-aks-secrets]。 虚拟节点的当前限制是，不能使用集成的 Azure AD 服务主体身份验证。 如果不使用机密，则在虚拟节点上计划的 Pod 将无法启动并报告错误 `HTTP response status code 400 error code "InaccessibleImage"`。

## <a name="test-the-virtual-node-pod"></a>测试虚拟节点 Pod

若要测试虚拟节点上运行的 Pod，请使用 Web 客户端浏览到演示应用程序。 由于为该 Pod 分配了一个内部 IP 地址，因此，可以从 AKS 群集上的另一个 Pod 快速测试此连接。 创建一个测试 Pod，并在其上附加一个终端会话：

```console
kubectl run -it --rm virtual-node-test --image=debian
```

使用 `apt-get` 在 Pod 中安装 `curl`：

```console
apt-get update && apt-get install -y curl
```

现在使用 `curl` 访问 Pod 的地址，例如 *http://10.241.0.4*。 提供上一个 `kubectl get pods` 命令中所示的你自己的内部 IP 地址：

```console
curl -L http://10.241.0.4
```

将显示演示应用程序，如以下精简版示例输出中所示：

```
$ curl -L 10.241.0.4

<html>
<head>
  <title>Welcome to Azure Container Instances!</title>
</head>
[...]
```

使用 `exit` 关闭与测试 Pod 之间的终端会话。 会话结束时便会删除该 Pod。

## <a name="remove-virtual-nodes"></a>删除虚拟节点

如果不再想要使用虚拟节点，则可以使用 [az aks disable-addons][az aks disable-addons] 命令禁用它们。 

首先，删除虚拟节点上运行的 helloworld pod:

```azurecli-interactive
kubectl delete -f virtual-node.yaml
```

下面的示例命令禁用 Linux 的虚拟节点：

```azurecli-interactive
az aks disable-addons --resource-group myResourceGroup --name myAKSCluster --addons virtual-node
```

现在，删除虚拟网络资源和资源组：

```azurecli-interactive
# Change the name of your resource group, cluster and network resources as needed
RES_GROUP=myResourceGroup
AKS_CLUSTER=myAKScluster
AKS_VNET=myVnet
AKS_SUBNET=myVirtualNodeSubnet

# Get AKS node resource group
NODE_RES_GROUP=$(az aks show --resource-group $RES_GROUP --name $AKS_CLUSTER --query nodeResourceGroup --output tsv)

# Get network profile ID
NETWORK_PROFILE_ID=$(az network profile list --resource-group $NODE_RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Get the service association link (SAL) ID
SAL_ID=$(az network vnet subnet show --resource-group $RES_GROUP --vnet-name $AKS_VNET --name $AKS_SUBNET --query id --output tsv)/providers/Microsoft.ContainerInstance/serviceAssociationLinks/default

# Delete the default SAL ID for the subnet
az resource delete --ids $SAL_ID --api-version 2018-07-01

# Delete the subnet delegation to Azure Container Instances
az network vnet subnet update --resource-group $RES_GROUP --vnet-name $AKS_VNET --name $AKS_SUBNET --remove delegations 0
```

## <a name="next-steps"></a>后续步骤

在本文中，在虚拟节点上计划了一个 Pod，并为其分配了一个专用的内部 IP 地址。 也可以改为创建服务部署，并通过负载均衡器或入口控制器将流量路由到 Pod。 有关详细信息，请参阅[在 AKS 中创建基本入口控制器][aks-basic-ingress]。

虚拟节点通常是 AKS 中缩放解决方案的一个组件。 有关缩放解决方案的详细信息，请参阅以下文章：

- [使用 Kubernetes 水平 Pod 自动缩放程序][aks-hpa]
- [使用 Kubernetes 群集自动缩放程序][aks-cluster-autoscaler]
- [查看虚拟节点的自动缩放示例][virtual-node-autoscale]
- [了解有关在 Virtual Kubelet 开放源代码库的更多信息][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[aks-github]: https://github.com/azure/aks/issues
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-create]: /cli/azure/group#az-group-create
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-network-vnet-show]: /cli/azure/network/vnet#az-network-vnet-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-show
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks disable-addons]: /cli/azure/aks#az-aks-disable-addons
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider#az-provider-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[acr-aks-secrets]: ../container-registry/container-registry-auth-aks.md#access-with-kubernetes-secret
