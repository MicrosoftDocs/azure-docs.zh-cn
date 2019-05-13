---
title: Kubernetes on Azure 教程 - 缩放应用程序
description: 此 Azure Kubernetes 服务 (AKS) 教程介绍如何缩放 Kubernetes 中的节点和 Pod，以及如何实施水平 Pod 自动缩放。
services: container-service
author: zr-msft
ms.service: container-service
ms.topic: tutorial
ms.date: 12/19/2018
ms.author: zarhoads
ms.custom: mvc
ms.openlocfilehash: 74cd73cd7f9d5be9da552fb22ef2a2750ddc8321
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61029850"
---
# <a name="tutorial-scale-applications-in-azure-kubernetes-service-aks"></a>教程：在 Azure Kubernetes 服务 (AKS) 中缩放应用程序

如果已按照教程执行，则在 AKS 中已有可正常工作的 Kubernetes 群集，并且已部署了示例 Azure 投票应用。 在本教程第 5 部分（共 7 部分）中，会在应用中扩大 Pod 并尝试 Pod 自动缩放。 还会了解如何缩放 Azure VM 节点数以更改群集用于托管工作负荷的容量。 学习如何：

> [!div class="checklist"]
> * 缩放 Kubernetes 节点
> * 手动缩放运行应用程序的 Kubernetes Pod
> * 配置运行应用前端的自动缩放 Pod

在另外的教程中，Azure 投票应用程序将更新为新版本。

## <a name="before-you-begin"></a>开始之前

上一教程中，应用程序已打包到容器映像中。 该映像已上传到 Azure容器注册表，同时，你创建了 AKS 群集。 然后，将应用程序部署到了 AKS 群集。 如果尚未完成这些步骤，并且想要逐一完成，请开始参阅[教程 1 – 创建容器映像][aks-tutorial-prepare-app]。

此教程需要运行 Azure CLI 2.0.53 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI][azure-cli-install]。

## <a name="manually-scale-pods"></a>手动缩放 Pod

在前述教程中部署 Azure 投票前端和 Redis 实例时，创建了单个副本。 若要查看群集中 Pod 的数目和状态，请使用 [kubectl get][kubectl-get] 命令，如下所示：

```console
kubectl get pods
```

以下示例输出显示一个前端 Pod 和一个后端 Pod：

```
NAME                               READY     STATUS    RESTARTS   AGE
azure-vote-back-2549686872-4d2r5   1/1       Running   0          31m
azure-vote-front-848767080-tf34m   1/1       Running   0          31m
```

若要手动更改 *azure-vote-front* 部署中的 Pod 数，请使用 [kubectl scale][kubectl-scale] 命令。 以下示例将前端 Pod 数增加到 *5*：

```console
kubectl scale --replicas=5 deployment/azure-vote-front
```

再次运行 [kubectl get pods][kubectl-get]，验证 AKS 是否创建其他 Pod。 一分钟左右之后，其他 Pod 会在群集中提供：

```console
$ kubectl get pods

                                    READY     STATUS    RESTARTS   AGE
azure-vote-back-2606967446-nmpcf    1/1       Running   0          15m
azure-vote-front-3309479140-2hfh0   1/1       Running   0          3m
azure-vote-front-3309479140-bzt05   1/1       Running   0          3m
azure-vote-front-3309479140-fvcvm   1/1       Running   0          3m
azure-vote-front-3309479140-hrbf2   1/1       Running   0          15m
azure-vote-front-3309479140-qphz8   1/1       Running   0          3m
```

## <a name="autoscale-pods"></a>自动缩放 Pod

Kubernetes 支持[水平 Pod 自动缩放][kubernetes-hpa]以根据 CPU 利用率或其他选择指标调整部署中的 Pod 数。 [指标服务器][metrics-server]用来将资源利用率提供给 Kubernetes，可自动部署在 AKS 群集 1.10 及更高版本中。 若要查看 AKS 群集的版本，请使用 [az aks show][az-aks-show] 命令，如以下示例所示：

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query kubernetesVersion
```

如果 AKS 群集的版本低于 *1.10*，请安装指标服务器，否则请跳过此步骤。 若要进行安装，请克隆 `metrics-server` GitHub 存储库并安装示例资源定义。 若要查看这些 YAML 定义的内容，请参阅[适用于 Kuberenetes 1.8+ 的指标服务器][metrics-server-github]。

```console
git clone https://github.com/kubernetes-incubator/metrics-server.git
kubectl create -f metrics-server/deploy/1.8+/
```

若要使用自动缩放程序，Pod 必须定义了 CPU 请求和限制。 在 `azure-vote-front` 部署中，前端容器已请求了 0.25 个 CPU，限制为 0.5 个 CPU。 这些资源请求和限制的定义方式如以下示例代码片段所示：

```yaml
resources:
  requests:
     cpu: 250m
  limits:
     cpu: 500m
```

下面的示例使用 [kubectl autoscale][kubectl-autoscale] 命令自动缩放 *azure-vote-front* 部署中的 Pod 数。 如果 CPU 利用率超过 50%，则自动缩放程序会将 Pod 增加到最多 *10* 个实例。 为部署定义的最小实例数为 *3*：

```console
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
```

若要查看自动缩放程序的状态，请使用 `kubectl get hpa` 命令，如下所示：

```
$ kubectl get hpa

NAME               REFERENCE                     TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
azure-vote-front   Deployment/azure-vote-front   0% / 50%   3         10        3          2m
```

在 Azure 投票应用处于最小负荷状态几分钟之后，Pod 副本数会自动减少到 3 个。 也可再次使用 `kubectl get pods` 来查看不需要的 Pod 是否已删除。

## <a name="manually-scale-aks-nodes"></a>手动缩放 AKS 节点

如果在前面的教程中使用命令创建了 Kubernetes 群集，则它具有一个节点。 如果计划在群集上有更多或更少的容器工作负荷，则可以手动调整节点数。

下面的示例将名为 myAKSCluster 的 Kubernetes 群集中的节点数增加到 3 个。 该命令需要几分钟时间完成。

```azurecli
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 3
```

当群集成功缩放时，输出类似于以下示例：

```
"agentPoolProfiles": [
  {
    "count": 3,
    "dnsPrefix": null,
    "fqdn": null,
    "name": "myAKSCluster",
    "osDiskSizeGb": null,
    "osType": "Linux",
    "ports": null,
    "storageProfile": "ManagedDisks",
    "vmSize": "Standard_D2_v2",
    "vnetSubnetId": null
  }
```

## <a name="next-steps"></a>后续步骤

在本教程中，在 Kubernetes 群集中使用了不同的缩放功能。 你已了解如何：

> [!div class="checklist"]
> * 手动缩放运行应用程序的 Kubernetes Pod
> * 配置运行应用前端的自动缩放 Pod
> * 手动缩放 Kubernetes 节点

继续学习下一教程，了解如何在 Kubernetes 中更新应用程序。

> [!div class="nextstepaction"]
> [在 Kubernetes 中更新应用程序][aks-tutorial-update-app]

<!-- LINKS - external -->
[kubectl-autoscale]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-scale]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale
[kubernetes-hpa]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[metrics-server-github]: https://github.com/kubernetes-incubator/metrics-server/tree/master/deploy/1.8%2B
[metrics-server]: https://v1-12.docs.kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-update-app]: ./tutorial-kubernetes-app-update.md
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
