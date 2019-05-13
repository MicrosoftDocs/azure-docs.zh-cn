---
title: 在 Azure Kubernetes 服务 (AKS) 上使用 GPU
description: 了解如何在 Azure Kubernetes 服务 (AKS) 上将 GPU 用于高性能计算或图形密集型工作负荷
services: container-service
author: zr-msft
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 02/28/2019
ms.author: zarhoads
ms.openlocfilehash: 150eaa6a4df558ed0c737d99cbcc8010baf63e96
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073926"
---
# <a name="use-gpus-for-compute-intensive-workloads-on-azure-kubernetes-service-aks"></a>在 Azure Kubernetes 服务 (AKS) 上将 GPU 用于计算密集型工作负荷

图形处理单元 (GPU) 通常用于计算密集型工作负荷，例如图形和可视化工作负荷。 AKS 支持创建启用 GPU 的节点池，以在 Kubernetes 中运行这些计算密集型工作负荷。 有关可用的启用了 GPU 的 VM 的详细信息，请参阅 [Azure 中 GPU 优化 VM 的大小][gpu-skus]。 对于 AKS 节点，我们建议最小大小为“Standard_NC6”。

> [!NOTE]
> 启用 GPU 的 VM 包含专用硬件，这些硬件定价较高，其可用性受区域限制。 有关详细信息，请参阅[定价][azure-pricing]工具和[区域可用性] [azure-availability]。

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集，其中包含支持 GPU 的节点。 AKS 群集须运行 Kubernetes 1.10 或更高版本。 如果需要满足这些要求的 AKS 群集，请参阅本文第一部分来[创建 AKS 群集](#create-an-aks-cluster)。

还需安装并配置 Azure CLI 2.0.59 或更高版本。 运行  `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅 [安装 Azure CLI][install-azure-cli]。

## <a name="create-an-aks-cluster"></a>创建 AKS 群集

如果需要可满足最低要求（启用了 GPU 的节点和 Kubernetes 版本 1.10 或更高版本）的 AKS 群集，请完成以下步骤。 如果已拥有满足这些要求的 AKS 群集，请[跳至下一部分](#confirm-that-gpus-are-schedulable)。

首先，使用 [az group create][az-group-create] 命令为群集创建资源组。 以下示例在“Eastus”区域创建名为“myResourceGroup”的资源组：

```azurecli
az group create --name myResourceGroup --location eastus
```

现在，使用 [az aks create][az-aks-create] 命令创建 AKS 群集。 以下示例会创建具有一个节点（大小为 `Standard_NC6`）的群集，并运行 Kubernetes 版本 1.11.7：

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-vm-size Standard_NC6 \
    --node-count 1 \
    --kubernetes-version 1.11.8
```

使用 [az aks get-credentials][az-aks-get-credentials] 命令获取 AKS 群集的凭据：

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="confirm-that-gpus-are-schedulable"></a>确认 GPU 是可计划的

创建 AKS 群集后，确认 GPU 在 Kubernetes 中是可计划的。 首先，使用 [kubectl get nodes][kubectl-get] 命令列出群集中的节点：

```
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-28993262-0   Ready    agent   6m    v1.11.7
```

现在，使用 [kubectl describe node][kubectl-describe] 命令确认 GPU 是可计划的。 在“容量”部分下，GPU 应列为 `nvidia.com/gpu:  1`。 如果未看到 GPU，请参阅[对 GPU 可用性进行故障排除](#troubleshoot-gpu-availability)部分。

以下精简示例显示了 GPU 在名为“aks-nodepool1-18821093-0”的节点上可用：

```
$ kubectl describe node aks-nodepool1-28993262-0

Name:               aks-nodepool1-28993262-0
Roles:              agent
Labels:             accelerator=nvidia

[...]

Capacity:
 cpu:                6
 ephemeral-storage:  30428648Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             57713780Ki
 nvidia.com/gpu:     1
 pods:               110
Allocatable:
 cpu:                5916m
 ephemeral-storage:  28043041951
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             52368500Ki
 nvidia.com/gpu:     1
 pods:               110
System Info:
 Machine ID:                 9148b74152374d049a68436ac59ee7c7
 System UUID:                D599728C-96F3-B941-BC79-E0B70453609C
 Boot ID:                    a2a6dbc3-6090-4f54-a2b7-7b4a209dffaf
 Kernel Version:             4.15.0-1037-azure
 OS Image:                   Ubuntu 16.04.5 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://1.13.1
 Kubelet Version:            v1.11.7
 Kube-Proxy Version:         v1.11.7
PodCIDR:                     10.244.0.0/24
ProviderID:                  azure:///subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Compute/virtualMachines/aks-nodepool1-28993262-0
Non-terminated Pods:         (9 in total)
  Namespace                  Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                     ------------  ----------  ---------------  -------------  ---
  gpu-resources              nvidia-device-plugin-97zfc               0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m4s

[...]
```

## <a name="run-a-gpu-enabled-workload"></a>运行启用了 GPU 的工作负荷

若要查看 GPU 的运行情况，请通过相应的资源请求计划启用了 GPU 的工作负荷。 在此示例中，我们针对 [MNIST 数据集](http://yann.lecun.com/exdb/mnist/)运行一个 [Tensorflow](https://www.tensorflow.org/versions/r1.1/get_started/mnist/beginners) 作业。

创建名为“samples-tf-mnist-demo.yaml”的文件并粘贴以下 YAML 清单。 以下作业清单包括资源限制 `nvidia.com/gpu: 1`：

> [!NOTE]
> 如果在调用驱动程序时收到版本不匹配错误，例如，CUDA 驱动程序版本不足以支持 CUDA 运行时版本，请查看 nVidia 驱动程序矩阵兼容性图表 - [https://docs.nvidia.com/deploy/cuda-compatibility/index.html](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: samples-tf-mnist-demo
  name: samples-tf-mnist-demo
spec:
  template:
    metadata:
      labels:
        app: samples-tf-mnist-demo
    spec:
      containers:
      - name: samples-tf-mnist-demo
        image: microsoft/samples-tf-mnist-demo:gpu
        args: ["--max_steps", "500"]
        imagePullPolicy: IfNotPresent
        resources:
          limits:
           nvidia.com/gpu: 1
      restartPolicy: OnFailure
```

使用 [kubectl apply][kubectl-apply] 命令运行该作业。 此命令分析清单文件并创建定义的 Kubernetes 对象：

```console
kubectl apply -f samples-tf-mnist-demo.yaml
```

## <a name="view-the-status-and-output-of-the-gpu-enabled-workload"></a>查看启用了 GPU 的工作负荷的状态和输出

将 [kubectl get jobs][kubectl-get] 命令与 `--watch` 参数配合使用，监视作业的进度。 先拉取映像并处理数据集可能需要几分钟时间。 当“COMPLETIONS”列显示“1/1”时，作业便已成功完成：

```
$ kubectl get jobs samples-tf-mnist-demo --watch

NAME                    COMPLETIONS   DURATION   AGE

samples-tf-mnist-demo   0/1           3m29s      3m29s
samples-tf-mnist-demo   1/1   3m10s   3m36s
```

若要查看启用了 GPU 的工作负荷的输出，首先请使用 [kubectl get pods][kubectl-get] 命令获取 pod 名称：

```
$ kubectl get pods --selector app=samples-tf-mnist-demo

NAME                          READY   STATUS      RESTARTS   AGE
samples-tf-mnist-demo-smnr6   0/1     Completed   0          3m
```

现在，使用 [kubectl logs][kubectl-logs] 命令查看 pod 日志。 以下示例 pod 日志确认已发现适当的 GPU 设备，即 `Tesla K80`。 为自己的 pod 提供名称：

```
$ kubectl logs samples-tf-mnist-demo-smnr6

2019-02-28 23:47:34.749013: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2019-02-28 23:47:34.879877: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties:
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: 3130:00:00.0
totalMemory: 11.92GiB freeMemory: 11.85GiB
2019-02-28 23:47:34.879915: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: 3130:00:00.0, compute capability: 3.7)
2019-02-28 23:47:39.492532: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.097
Accuracy at step 10: 0.6993
Accuracy at step 20: 0.8208
Accuracy at step 30: 0.8594
Accuracy at step 40: 0.8685
Accuracy at step 50: 0.8864
Accuracy at step 60: 0.901
Accuracy at step 70: 0.905
Accuracy at step 80: 0.9103
Accuracy at step 90: 0.9126
Adding run metadata for 99
Accuracy at step 100: 0.9176
Accuracy at step 110: 0.9149
Accuracy at step 120: 0.9187
Accuracy at step 130: 0.9253
Accuracy at step 140: 0.9252
Accuracy at step 150: 0.9266
Accuracy at step 160: 0.9255
Accuracy at step 170: 0.9267
Accuracy at step 180: 0.9257
Accuracy at step 190: 0.9309
Adding run metadata for 199
Accuracy at step 200: 0.9272
Accuracy at step 210: 0.9321
Accuracy at step 220: 0.9343
Accuracy at step 230: 0.9388
Accuracy at step 240: 0.9408
Accuracy at step 250: 0.9394
Accuracy at step 260: 0.9412
Accuracy at step 270: 0.9422
Accuracy at step 280: 0.9436
Accuracy at step 290: 0.9411
Adding run metadata for 299
Accuracy at step 300: 0.9426
Accuracy at step 310: 0.9466
Accuracy at step 320: 0.9458
Accuracy at step 330: 0.9407
Accuracy at step 340: 0.9445
Accuracy at step 350: 0.9486
Accuracy at step 360: 0.9475
Accuracy at step 370: 0.948
Accuracy at step 380: 0.9516
Accuracy at step 390: 0.9534
Adding run metadata for 399
Accuracy at step 400: 0.9501
Accuracy at step 410: 0.9552
Accuracy at step 420: 0.9535
Accuracy at step 430: 0.9545
Accuracy at step 440: 0.9533
Accuracy at step 450: 0.9526
Accuracy at step 460: 0.9566
Accuracy at step 470: 0.9547
Accuracy at step 480: 0.9548
Accuracy at step 490: 0.9545
Adding run metadata for 499
```

## <a name="clean-up-resources"></a>清理资源

若要删除本文中创建的相关 Kubernetes 对象，请使用 [kubectl delete job][kubectl delete] 命令，如下所示：

```console
kubectl delete jobs samples-tf-mnist-demo
```

## <a name="troubleshoot-gpu-availability"></a>对 GPU 可用性进行故障排除

如果看到节点上的 GPU 不可用，则可能需要为 nVidia 设备插件部署 DaemonSet。 此 DaemonSet 在会每个节点上运行 pod，以便为 GPU 提供所需驱动程序。

首先，使用 [kubectl create namespace][kubectl-create] 命令创建命名空间，例如“gpu-resources”：

```console
kubectl create namespace gpu-resources
```

创建名为“nvidia-device-plugin-ds.yam”的文件并粘贴以下 YAML 清单。 更新清单中间部分的 `image: nvidia/k8s-device-plugin:1.11`，以便与 Kubernetes 版本匹配。 例如，如果 AKS 群集运行的是 Kubernetes 版本 1.12，请将标记更新为 `image: nvidia/k8s-device-plugin:1.12`。

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    kubernetes.io/cluster-service: "true"
  name: nvidia-device-plugin
  namespace: gpu-resources
spec:
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      containers:
      - image: nvidia/k8s-device-plugin:1.11 # Update this tag to match your Kubernetes version
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
      nodeSelector:
        beta.kubernetes.io/os: linux
        accelerator: nvidia
```

现在，使用 [kubectl apply][kubectl-apply] 命令创建 DaemonSet：

```
$ kubectl apply -f nvidia-device-plugin-ds.yaml

daemonset "nvidia-device-plugin" created
```

再次运行 [kubectl describe node][kubectl-describe] 命令，验证 GPU 现在是否在该节点上可用。

## <a name="next-steps"></a>后续步骤

若要运行 Apache Spark 作业，请参阅[在 AKS 上运行 Apache Spark 作业][aks-spark]。

有关在 Kubernetes 上运行机器学习 (ML) 工作负荷的更多信息，请参阅 [Kubeflow 实验室][kubeflow-labs]。

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeflow-labs]: https://github.com/Azure/kubeflow-labs
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[azure-pricing]: https://azure.microsoft.com/pricing/
[azure-availability]: https://azure.microsoft.com/global-infrastructure/services/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-spark]: spark-job.md
[gpu-skus]: ../virtual-machines/linux/sizes-gpu.md
[install-azure-cli]: /cli/azure/install-azure-cli
