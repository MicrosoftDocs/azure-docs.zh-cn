---
title: 在 Azure Kubernetes 服务 (AKS) 中创建用于 Pod 的静态卷
description: 了解如何在 Azure Kubernetes 服务 (AKS) 中使用 Azure 磁盘手动创建卷
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/01/2019
ms.author: iainfou
ms.openlocfilehash: 02a863a4ddf59fb36c5f2ae7f3092896d2e1d860
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65072163"
---
# <a name="manually-create-and-use-a-volume-with-azure-disks-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 中通过 Azure 磁盘手动创建并使用卷

基于容器的应用程序通常需要访问数据并将数据保存在外部数据卷中。 如果单个 Pod 需要访问存储，则可以使用 Azure 磁盘来提供本机卷供应用程序使用。 本文介绍了如何手动创建 Azure 磁盘并将其附加到 AKS 中的 Pod。

> [!NOTE]
> Azure 磁盘一次只能装载到单个 Pod 中。 如果需要在多个 Pod 之间共享永久性卷，请使用 [Azure 文件][azure-files-volume]。

有关 Kubernetes 卷的详细信息，请参阅 [AKS 中应用程序的存储选项][concepts-storage]。

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 门户][aks-quickstart-portal]。

还需安装并配置 Azure CLI 2.0.59 或更高版本。 运行  `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅 [安装 Azure CLI][install-azure-cli]。

## <a name="create-an-azure-disk"></a>创建 Azure 磁盘

创建用于 AKS 的 Azure 磁盘时，可以在**节点**资源组中创建磁盘资源。 此方法允许 AKS 群集访问和管理磁盘资源。 如果想要在单独的资源组中创建磁盘，则必须向群集的 Azure Kubernetes 服务 (AKS) 服务主体授予磁盘的资源组的 `Contributor` 角色。

对于本文，请在节点资源组中创建磁盘。 首先，使用 [az aks show][az-aks-show] 命令获取资源组名称并添加 `--query nodeResourceGroup` 查询参数。 以下示例获取名为 myResourceGroup 的资源组中 AKS 群集名称 myAKSCluster 的节点资源组：

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

现在，使用 [az disk create][az-disk-create] 命令创建磁盘。 指定在上一命令中获取的节点资源组名称，然后指定磁盘资源的名称，例如 *myAKSDisk*。 以下示例创建一个 *20*GiB 的磁盘，并且在创建后输出磁盘的 ID：

```azurecli-interactive
az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name myAKSDisk  \
  --size-gb 20 \
  --query id --output tsv
```

> [!NOTE]
> Azure 磁盘依据特定大小的 SKU 收取费用。 这些 Sku 的范围从 S4 或 P4 磁盘的 32GiB 到 32TiB S80 或 P80 （处于预览状态） 的磁盘。 高级托管磁盘的吞吐量和 IOPS 性能取决于 SKU 和 AKS 群集中节点的实例大小。 请参阅[托管磁盘的定价和性能][managed-disk-pricing-performance]。

在命令成功完成后将显示磁盘资源 ID，如以下示例输出中所示。 在下一步骤中将使用此磁盘 ID 来装载磁盘。

```console
/subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

## <a name="mount-disk-as-volume"></a>装载磁盘作为卷

若要将 Azure 磁盘装载到 Pod 中，请在容器规范中配置卷。使用以下内容创建名为 `azure-disk-pod.yaml` 的新文件。 将 `diskName` 更新为在上一步骤中创建的磁盘的名称，将 `diskURI` 更新为在磁盘创建命令的输出中显示的磁盘 ID。 如果需要，请更新 `mountPath`，这是 Azure 磁盘在 Pod 中的装载路径。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.15.5
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: myAKSDisk
          diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

使用 `kubectl` 命令创建 Pod。

```console
kubectl apply -f azure-disk-pod.yaml
```

现在你有一个正在运行的 Pod，其中 Azure 磁盘被装载到 `/mnt/azure`。 可以使用 `kubectl describe pod mypod` 来验证磁盘是否已成功装载。 以下精简示例输出显示容器中装载的卷：

```
[...]
Volumes:
  azure:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     myAKSDisk
    DiskURI:      /subscriptions/<subscriptionID/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
    Kind:         Managed
    FSType:       ext4
    CachingMode:  ReadWrite
    ReadOnly:     false
  default-token-z5sd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-z5sd7
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              1m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-z5sd7"
  Normal  SuccessfulMountVolume  41s   kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "azure"
[...]
```

## <a name="next-steps"></a>后续步骤

如需相关的最佳做法，请参阅[在 AKS 中存储和备份的最佳做法][operator-best-practices-storage]。

有关 AKS 群集与 Azure 磁盘进行交互的详细信息，请参阅 [Azure 磁盘的 Kubernetes 插件][kubernetes-disks]。

<!-- LINKS - external -->
[kubernetes-disks]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_disk/README.md
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/resource#az-resource-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-files-volume]: azure-files-volume.md
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
