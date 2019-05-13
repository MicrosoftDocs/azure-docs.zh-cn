---
title: 在 Azure Kubernetes 服务 (AKS) 中使用门户创建虚拟节点
description: 了解如何通过 Azure 门户创建使用虚拟节点运行 Pod 的 Azure Kubernetes 服务 (AKS) 群集。
services: container-service
author: iainfoulds
ms.topic: conceptual
ms.service: container-service
ms.date: 05/06/2019
ms.author: iainfou
ms.openlocfilehash: 4376db8cdfa90b8d29ecd9b210e683848b4c94b4
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072588"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-in-the-azure-portal"></a>创建 Azure Kubernetes 服务 (AKS) 群集并将其配置为使用 Azure 门户中的虚拟节点

若要在 Azure Kubernetes 服务 (AKS) 群集中快速部署工作负荷，可以使用虚拟节点。 使用虚拟节点可快速预配 Pod，并且只需对其执行时间按秒付费。 在缩放方案中，无需等待 Kubernetes 群集自动缩放程序部署 VM 计算节点来运行其他 Pod。 仅在使用 Linux pod 和节点支持的虚拟节点。

本文介绍如何创建和配置虚拟网络资源以及启用了虚拟节点的 AKS 群集。

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

如果提供程序显示为*NotRegistered*，注册提供程序使用 [az 提供程序注册] [az 提供程序注册] 在下面的示例所示：

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

## <a name="sign-in-to-azure"></a>登录 Azure

通过 https://portal.azure.com 登录到 Azure 门户。

## <a name="create-an-aks-cluster"></a>创建 AKS 群集

在 Azure 门户的左上角，选择“创建资源” > “Kubernetes 服务”。

在“基本信息”页面上，配置以下选项：

- *项目详细信息*：选择 Azure 订阅，然后选择或创建 Azure 资源组，例如 *myResourceGroup*。 输入 **Kubernetes 群集名称**，例如 *myAKSCluster*。
- *群集详细信息*：选择 AKS 群集的区域、Kubernetes 版本和 DNS 名称前缀。
- *主节点池*:选择 AKS 节点的 VM 大小。 一旦部署 AKS 群集，不能更改 VM 大小。
     - 选择要部署到群集中的节点数。 在本文中，将“节点计数”设置为 *1*。 部署群集后，可以调整节点计数。

单击“下一步:**缩放**。

上**规模**页上，选择*已启用*下**虚拟节点**。

![创建 AKS 群集并启用虚拟节点](media/virtual-nodes-portal/enable-virtual-nodes.png)

默认情况下，将创建一个 Azure Active Directory 服务主体。 此服务主体用于群集通信以及与其他 Azure 服务集成。

群集还配置有高级网络。 虚拟节点配置为使用自己的 Azure 虚拟网络子网。 此子网具有委托的权限，可连接 AKS 群集之间的 Azure 资源。 如果还没有委托的子网，Azure 门户将创建并配置 Azure 虚拟网络和子网，以便与虚拟节点配合使用。

选择“查看 + 创建”。 完成验证后，选择“创建”。

创建 AKS 群集并让其可供使用需要几分钟的时间。

## <a name="connect-to-the-cluster"></a>连接至群集

Azure Cloud Shell 是免费的交互式 shell，可以使用它运行本文中的步骤。 它预安装有常用 Azure 工具并将其配置与帐户一起使用。 若要管理 Kubernetes 群集，请使用 Kubernetes 命令行客户端 [kubectl][kubectl]。 `kubectl` 客户端已预装在 Azure Cloud Shell 中。

若要打开 Cloud Shell，请从代码块的右上角选择“试一试”。 也可以通过转到 [https://shell.azure.com/bash](https://shell.azure.com/bash) 在单独的浏览器标签页中启动 Cloud Shell。 选择“复制”以复制代码块，将其粘贴到 Cloud Shell 中，然后按 Enter 来运行它。

使用 [az aks get-credentials][az-aks-get-credentials] 命令将 `kubectl` 配置为连接到 Kubernetes 群集。 以下示例获取名为 *myResourceGroup* 的资源组中群集名称 *myAKSCluster* 的凭据：

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

若要验证到群集的连接，请使用 [kubectl get][kubectl-get] 命令返回群集节点的列表。

```azurecli-interactive
kubectl get nodes
```

以下示例输出依次显示了所创建的单个 VM 节点以及用于 Linux 的虚拟节点 virtual-node-aci-linux：

```
$ kubectl get nodes

NAME                           STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux         Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0       Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>部署示例应用

在 Azure Cloud Shell 中，创建一个名为 `virtual-node.yaml` 的文件并复制到以下 YAML 中。 若要在节点上计划容器，需定义 [nodeSelector][node-selector] 和 [toleration][toleration]。 这些设置允许在虚拟节点上计划 Pod，并确认已成功启用该功能。

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

```azurecli-interactive
kubectl apply -f virtual-node.yaml
```

使用带有 `-o wide` 参数的 [kubectl get pods][kubectl-get] 命令输出 Pod 和计划节点的列表。 请注意，已在 `virtual-node-linux` 节点上计划 `virtual-node-helloworld` pod。

```
$ kubectl get pods -o wide

NAME                                     READY     STATUS    RESTARTS   AGE       IP           NODE
virtual-node-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

系统从被委派用于虚拟节点的 Azure 虚拟网络子网中为该 Pod 分配了一个内部 IP 地址。

> [!NOTE]
> 如果使用存储在 Azure 容器注册表中的映像，请[配置并使用 Kubernetes 机密][acr-aks-secrets]。 虚拟节点的当前限制是，不能使用集成的 Azure AD 服务主体身份验证。 如果不使用机密，则在虚拟节点上计划的 Pod 将无法启动并报告错误 `HTTP response status code 400 error code "InaccessibleImage"`。

## <a name="test-the-virtual-node-pod"></a>测试虚拟节点 Pod

若要测试虚拟节点上运行的 Pod，请使用 Web 客户端浏览到演示应用程序。 由于为该 Pod 分配了一个内部 IP 地址，因此，可以从 AKS 群集上的另一个 Pod 快速测试此连接。 创建一个测试 Pod，并在其上附加一个终端会话：

```azurecli-interactive
kubectl run -it --rm virtual-node-test --image=debian
```

使用 `apt-get` 在 Pod 中安装 `curl`：

```azurecli-interactive
apt-get update && apt-get install -y curl
```

现在使用 `curl` 访问 Pod 的地址，例如 *http://10.241.0.4*。 提供上一个 `kubectl get pods` 命令中所示的你自己的内部 IP 地址：

```azurecli-interactive
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

## <a name="next-steps"></a>后续步骤

在本文中，在虚拟节点上计划了一个 Pod，并为其分配了一个专用的内部 IP 地址。 也可以改为创建服务部署，并通过负载均衡器或入口控制器将流量路由到 Pod。 有关详细信息，请参阅[在 AKS 中创建基本入口控制器][aks-basic-ingress]。

虚拟节点是 AKS 中缩放解决方案的一个组件。 有关缩放解决方案的详细信息，请参阅以下文章：

- [使用 Kubernetes 水平 Pod 自动缩放程序][aks-hpa]
- [使用 Kubernetes 群集自动缩放程序][aks-cluster-autoscaler]
- [查看虚拟节点的自动缩放示例][virtual-node-autoscale]
- [了解有关在 Virtual Kubelet 开放源代码库的更多信息][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[aks-github]: https://github.com/azure/aks/issues]
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet

<!-- LINKS - internal -->
[aks-network]: ./networking-overview.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[acr-aks-secrets]: ../container-registry/container-registry-auth-aks.md#access-with-kubernetes-secret
[az-provider-list]: /cli/azure/provider#az-provider-list