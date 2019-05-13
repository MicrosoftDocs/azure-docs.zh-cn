---
title: 使用多个节点池在 Azure Kubernetes 服务 (AKS)
description: 了解如何创建和管理群集在 Azure Kubernetes 服务 (AKS) 的多个节点池
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/29/2019
ms.author: iainfou
ms.openlocfilehash: 1c24bbb9433e4164d4b2f6ce1ac7bd726cc36356
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506910"
---
# <a name="preview---create-and-manage-multiple-node-pools-for-a-cluster-in-azure-kubernetes-service-aks"></a>预览-创建和管理群集在 Azure Kubernetes 服务 (AKS) 的多个节点池

在 Azure Kubernetes 服务 (AKS) 节点具有相同配置的组合在一起成*节点池*。 这些节点池包含运行应用程序的基础虚拟机。 创建 AKS 群集，这会创建时定义节点和它们的大小 (SKU) 的初始数目*默认节点池*。 若要支持具有不同的计算或存储需求的应用程序，可以创建其他节点池。 例如，使用这些附加节点池提供计算密集型应用程序的 Gpu 或 SSD 的高性能存储访问。

本文介绍如何创建和管理 AKS 群集中的多个节点池。 此功能目前处于预览状态。

> [!IMPORTANT]
> AKS 预览功能是自助服务和可以选择加入的功能。 提供预览是为了从我们的社区收集反馈和 bug。 但是，Azure 技术支持部门不为其提供支持。 如果你创建一个群集，或者将这些功能添加到现有群集，则除非该功能不再为预览版并升级为公开发布版 (GA)，否则该群集不会获得支持。
>
> 如果遇到预览版功能的问题，请[在 AKS GitHub 存储库中提交问题][aks-github]，并在 Bug 标题中填写预览版功能的名称。

## <a name="before-you-begin"></a>开始之前

你需要 Azure CLI 版本 2.0.61 或更高版本安装和配置。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][install-azure-cli]。

### <a name="install-aks-preview-cli-extension"></a>安装 aks-preview CLI 扩展

CLI 命令来创建和管理多个节点池均位于*aks 预览版*CLI 扩展。 安装*aks 预览版*Azure CLI 扩展使用[az 扩展添加][ az-extension-add]命令，如下面的示例中所示：

```azurecli-interactive
az extension add --name aks-preview
```

> [!NOTE]
> 如果您先前安装了*aks 预览版*扩展，安装任何可用更新使用`az extension update --name aks-preview`命令。

### <a name="register-multiple-node-pool-feature-provider"></a>注册多个节点池功能提供程序

若要创建的 AKS 群集，可以使用多个节点池，请先启用你的订阅上的两个功能标志。 多节点池群集使用的虚拟机规模集 (VMSS) 来管理部署和 Kubernetes 节点的配置。 注册*MultiAgentpoolPreview*并*VMSSPreview*使用功能标志[az 功能注册][ az-feature-register]命令中所示下面的示例：

```azurecli-interactive
az feature register --name MultiAgentpoolPreview --namespace Microsoft.ContainerService
az feature register --name VMSSPreview --namespace Microsoft.ContainerService
```

> [!NOTE]
> 创建已成功注册后任何 AKS 群集*MultiAgentpoolPreview*使用此预览群集体验。 若要继续创建正则、 完全受支持的群集，不要启用预览功能在生产订阅。 使用单独的测试或开发的 Azure 订阅进行测试预览功能。

状态显示为“已注册”需要几分钟时间。 可以使用 [az feature list][az-feature-list] 命令检查注册状态：

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/MultiAgentpoolPreview')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/VMSSPreview')].{Name:name,State:properties.state}"
```

准备就绪后，使用 [az provider register][az-provider-register] 命令刷新 Microsoft.ContainerService 资源提供程序的注册状态：

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="limitations"></a>限制

创建和管理 AKS 群集中以支持多个节点池时，应考虑以下限制：

* 多个节点池才可用于创建已成功注册后的群集*MultiAgentpoolPreview*并*VMSSPreview*功能为你的订阅。 无法添加或管理现有之前已成功注册这些功能创建的 AKS 群集与节点池。
* 无法删除第一个节点池。
* 不能使用 HTTP 应用程序路由外接程序。

虽然此功能处于预览状态，下面的其他限制适用：

* AKS 群集可以具有最多八个节点池。
* AKS 群集可以在这些八个节点的池之间具有最多 400 节点。

## <a name="create-an-aks-cluster"></a>创建 AKS 群集

若要开始，请使用单个节点池创建 AKS 群集。 下面的示例使用[az 组创建][ az-group-create]命令以创建名为的资源组*myResourceGroup*中*eastus*区域。 名为 AKS 群集*myAKSCluster*然后，使用创建[az aks 创建][ az-aks-create]命令。 一个 *-kubernetes 版本*的*1.12.6*用于说明如何更新下一个步骤中的节点池。 你可以指定任何[受支持的 Kubernetes 版本][supported-versions]。

```azurecli-interactive
# Create a resource group in East US
az group create --name myResourceGroup --location eastus

# Create a basic single-node AKS cluster
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-vmss \
    --node-count 1 \
    --generate-ssh-keys \
    --kubernetes-version 1.12.6
```

创建群集需要几分钟时间。

群集准备就绪后，使用[az aks get-credentials 来获取凭据][ az-aks-get-credentials]命令，用于获取群集凭据`kubectl`:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="add-a-node-pool"></a>添加节点池

上一步中创建的群集有一个单个节点池。 让我们添加第二个节点池 using [az aks 节点池添加][ az-aks-nodepool-add]命令。 下面的示例创建一个名为的节点池*mynodepool*运行一次*3*节点：

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 3
```

若要查看的节点池的状态，请使用[az aks 节点池列表][ az-aks-nodepool-list]命令并指定资源组和群集名称：

```azurecli-interactive
az aks nodepool list --resource-group myResourceGroup --cluster-name myAKSCluster -o table
```

以下示例输出表明*mynodepool*已成功创建三个节点都在节点池。 在上一步骤中，默认值创建 AKS 群集时*nodepool1*创建时使用的节点计数*1*。

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster -o table

AgentPoolType            Count    MaxPods    Name        OrchestratorVersion    OsDiskSizeGb    OsType    ProvisioningState    ResourceGroup         VmSize
-----------------------  -------  ---------  ----------  ---------------------  --------------  --------  -------------------  --------------------  ---------------
VirtualMachineScaleSets  3        110        mynodepool  1.12.6                 100             Linux     Succeeded            myResourceGroupPools  Standard_DS2_v2
VirtualMachineScaleSets  1        110        nodepool1   1.12.6                 100             Linux     Succeeded            myResourceGroupPools  Standard_DS2_v2
```

> [!TIP]
> 如果没有*OrchestratorVersion*或*VmSize*添加节点池时指定了根据 AKS 群集的默认值创建节点。 在此示例中，这是 Kubernetes 版本*1.12.6*和节点大小*Standard_DS2_v2*。

## <a name="upgrade-a-node-pool"></a>升级的节点池

第一步中创建 AKS 群集时`--kubernetes-version`的*1.12.6*指定。 让我们来升级*mynodepool*到 Kubernetes *1.12.7*。 使用[az aks 节点的池升级][ az-aks-nodepool-upgrade]命令，将升级的节点池，如下面的示例中所示：

```azurecli-interactive
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --kubernetes-version 1.12.7 \
    --no-wait
```

列出使用重新工作节点池的状态[az aks 节点池列表][ az-aks-nodepool-list]命令。 下面的示例演示*mynodepool*处于*升级*状态变为*1.12.7*:

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster -o table

AgentPoolType            Count    MaxPods    Name        OrchestratorVersion    OsDiskSizeGb    OsType    ProvisioningState    ResourceGroup    VmSize
-----------------------  -------  ---------  ----------  ---------------------  --------------  --------  -------------------  ---------------  ---------------
VirtualMachineScaleSets  3        110        mynodepool  1.12.7                 100             Linux     Upgrading            myResourceGroup  Standard_DS2_v2
VirtualMachineScaleSets  1        110        nodepool1   1.12.6                 100             Linux     Succeeded            myResourceGroup  Standard_DS2_v2
```

花费几分钟才能将节点升级到指定的版本。

作为最佳做法，应为相同的 Kubernetes 版本升级 AKS 群集中的所有节点池。 升级单个节点池的功能，可以执行滚动升级，可以安排节点池来维护应用程序正常运行时间之间的 pod。

## <a name="scale-a-node-pool"></a>扩展节点池

为你的应用程序工作负荷需求变化，可能需要缩放池中节点的节点数。 可缩放的节点数，向上或向下。

<!--If you scale down, nodes are carefully [cordoned and drained][kubernetes-drain] to minimize disruption to running applications.-->

若要缩放的池中节点的节点数，请使用[az aks 节点池规模][ az-aks-nodepool-scale]命令。 下面的示例中的节点数量进行缩放*mynodepool*到*5*:

```azurecli-interactive
az aks nodepool scale \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 5 \
    --no-wait
```

列出使用重新工作节点池的状态[az aks 节点池列表][ az-aks-nodepool-list]命令。 下面的示例演示*mynodepool*处于*缩放*状态的新的计数*5*节点：

```console
$ az aks nodepool list -g myResourceGroupPools --cluster-name myAKSCluster -o table

AgentPoolType            Count    MaxPods    Name        OrchestratorVersion    OsDiskSizeGb    OsType    ProvisioningState    ResourceGroup         VmSize
-----------------------  -------  ---------  ----------  ---------------------  --------------  --------  -------------------  --------------------  ---------------
VirtualMachineScaleSets  5        110        mynodepool  1.12.7                 100             Linux     Scaling              myResourceGroupPools  Standard_DS2_v2
VirtualMachineScaleSets  1        110        nodepool1   1.12.6                 100             Linux     Succeeded            myResourceGroupPools  Standard_DS2_v2
```

花费几分钟时间才能完成缩放操作。

## <a name="delete-a-node-pool"></a>删除节点池

如果不再需要池时，可以将其删除并删除基础 VM 节点。 若要删除节点池，请使用[az aks 节点池删除][ az-aks-nodepool-delete]命令并指定节点的池名称。 以下示例将删除*mynoodepool*前面步骤中创建：

> [!CAUTION]
> 没有恢复选项时删除节点池可能会发生数据丢失。 如果不能在其他节点池上计划的 pod，这些应用程序将不可用。 请确保在使用的应用程序没有数据备份或能够在群集中其他节点的池上运行时，不要删除节点池。

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name mynodepool --no-wait
```

从下面的示例输出[az aks 节点池列表][ az-aks-nodepool-list]命令显示*mynodepool*处于*删除*状态：

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster -o table

AgentPoolType            Count    MaxPods    Name        OrchestratorVersion    OsDiskSizeGb    OsType    ProvisioningState    ResourceGroup         VmSize
-----------------------  -------  ---------  ----------  ---------------------  --------------  --------  -------------------  --------------------  ---------------
VirtualMachineScaleSets  5        110        mynodepool  1.12.7                 100             Linux     Deleting             myResourceGroupPools  Standard_DS2_v2
VirtualMachineScaleSets  1        110        nodepool1   1.12.6                 100             Linux     Succeeded            myResourceGroupPools  Standard_DS2_v2
```

需要几分钟，若要删除的节点和节点池。

## <a name="specify-a-vm-size-for-a-node-pool"></a>指定节点池的 VM 大小

在上一示例中创建节点池的默认 VM 大小使用在群集中创建的节点。 一种更常见方案是为你要使用不同的 VM 大小和功能创建节点池。 例如，可以创建包含具有大量的 CPU 或内存中，节点的节点池或提供 GPU 支持的节点池。 在下一步，您 [使用 taints 和 tolerations][#schedule-pods-using-taints-and-tolerations] 告知如何限制对 pod 访问的 Kubernetes 计划程序可在这些节点上运行。

在以下示例中，将创建使用基于 GPU 的节点池*Standard_NC6* VM 大小。 这些虚拟机采用 NVIDIA Tesla K80 卡。 有关可用的 VM 大小的信息，请参阅[在 Azure 中 Linux 虚拟机的大小][vm-sizes]。

创建节点池使用[az aks 节点池添加][ az-aks-nodepool-add]试命令。 这一次，指定的名称*gpunodepool*，并使用`--node-vm-size`参数来指定*Standard_NC6*大小：

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name gpunodepool \
    --node-count 1 \
    --node-vm-size Standard_NC6 \
    --no-wait
```

从下面的示例输出[az aks 节点池列表][ az-aks-nodepool-list]命令显示*gpunodepool*是*创建*节点指定*VmSize*:

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster -o table

AgentPoolType            Count    MaxPods    Name         OrchestratorVersion    OsDiskSizeGb    OsType    ProvisioningState    ResourceGroup         VmSize
-----------------------  -------  ---------  -----------  ---------------------  --------------  --------  -------------------  --------------------  ---------------
VirtualMachineScaleSets  1        110        gpunodepool  1.12.6                 100             Linux     Creating             myResourceGroupPools  Standard_NC6
VirtualMachineScaleSets  1        110        nodepool1    1.12.6                 100             Linux     Succeeded            myResourceGroupPools  Standard_DS2_v2
```

需要几分钟*gpunodepool*要成功创建。

## <a name="schedule-pods-using-taints-and-tolerations"></a>计划使用 taints 和 tolerations pod

现在将具有两个节点池在群集中最初创建的默认节点池和基于 GPU 的节点池。 使用[kubectl 获取节点][ kubectl-get]命令以查看你的群集中的节点。 下面的示例输出显示在每个节点池中的一个节点：

```console
$ kubectl get nodes

NAME                                 STATUS   ROLES   AGE     VERSION
aks-gpunodepool-28993262-vmss000000  Ready    agent   4m22s   v1.12.6
aks-nodepool1-28993262-vmss000000    Ready    agent   115m    v1.12.6
```

Kubernetes 计划程序能够使用排斥和容许来限制可在节点上运行的工作负荷。

* 将**排斥**应用到指明了只能计划特定 pod 的节点。
* 然后，将**容许**应用到可以*容许*节点排斥的 pod。

有关如何使用的详细信息高级 Kubernetes 计划功能，请参阅[在 AKS 中的高级计划程序功能的最佳做法][taints-tolerations]

在此示例中，将不应用到基于 GPU 的节点[kubectl 不节点][ kubectl-taint]命令。 指定的输出中的上一个基于 GPU 的节点的名称`kubectl get nodes`命令。 不作为应用*键： 值*，然后计划选项。 下面的示例使用*sku = gpu*配对，并定义 pod 否则必须*NoSchedule*功能：

```console
kubectl taint node aks-gpunodepool-28993262-vmss000000 sku=gpu:NoSchedule
```

下面的基本示例 YAML 清单使用允许以允许基于 GPU 的节点上运行 NGINX pod 的 Kubernetes 计划程序。 若要对 MNIST 数据集运行 Tensorflow 作业更为合适，但耗时比较长的示例，请参阅[适用于计算密集型工作负荷在 AKS 上使用 Gpu][gpu-cluster]。

创建名为 `gpu-toleration.yaml` 的文件，并将其复制到以下示例 YAML 中：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.15.9
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 1
        memory: 2G
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Pod 使用计划`kubectl apply -f gpu-toleration.yaml`命令：

```console
kubectl apply -f gpu-toleration.yaml
```

它需要几秒钟来计划 pod 和 NGINX 映像提取。 使用[kubectl 描述 pod] [ kubectl-describe]命令查看 pod 状态。 以下浓缩版的示例输出显示*sku = gpu:NoSchedule*允许应用。 在事件部分中，计划程序已分配到 pod *aks gpunodepool 28993262 vmss000000*基于 GPU 的节点：

```console
$ kubectl describe pod mypod

[...]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
                 sku=gpu:NoSchedule
Events:
  Type    Reason     Age    From                                          Message
  ----    ------     ----   ----                                          -------
  Normal  Scheduled  4m48s  default-scheduler                             Successfully assigned default/mypod to aks-gpunodepool-28993262-vmss000000
  Normal  Pulling    4m47s  kubelet, aks-gpunodepool-28993262-vmss000000  pulling image "nginx:1.15.9"
  Normal  Pulled     4m43s  kubelet, aks-gpunodepool-28993262-vmss000000  Successfully pulled image "nginx:1.15.9"
  Normal  Created    4m40s  kubelet, aks-gpunodepool-28993262-vmss000000  Created container
  Normal  Started    4m40s  kubelet, aks-gpunodepool-28993262-vmss000000  Started container
```

可以在节点上计划具有此应用不的 pod *gpunodepool*。 将计划中的任何其他 pod *nodepool1*节点池。 如果您创建其他节点的池，可以使用其他 taints 并 tolerations 来限制哪些 pod 可以安排在这些节点资源上。

## <a name="clean-up-resources"></a>清理资源

在本文中，您创建的 AKS 群集，包括基于 GPU 的节点。 若要减少不必要的成本，你可能想要删除*gpunodepool*，或整个 AKS 群集。

若要删除基于 GPU 的节点池，请使用[az aks 节点池删除][ az-aks-nodepool-delete]命令，在下面的示例中所示：

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name gpunodepool
```

若要删除群集本身，请使用[az 组删除][ az-group-delete]命令删除 AKS 资源组：

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>后续步骤

本文介绍了如何创建和管理 AKS 群集中的多个节点池。 有关如何跨节点池控制 pod 的详细信息，请参阅[在 AKS 中的高级计划程序功能的最佳做法][operator-best-practices-advanced-scheduler]。

<!-- EXTERNAL LINKS -->
[aks-github]: https://github.com/azure/aks/issues]
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe

<!-- INTERNAL LINKS -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-nodepool-add]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-nodepool-add
[az-aks-nodepool-list]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-nodepool-list
[az-aks-nodepool-upgrade]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-nodepool-upgrade
[az-aks-nodepool-scale]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-nodepool-scale
[az-aks-nodepool-delete]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-nodepool-delete
[vm-sizes]: ../virtual-machines/linux/sizes.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[gpu-cluster]: gpu-cluster.md
[az-group-delete]: /cli/azure/group#az-group-delete
[install-azure-cli]: /cli/azure/install-azure-cli
[supported-versions]: supported-kubernetes-versions.md
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
