---
title: 更新并重新启动 Linux 节点使用 kured 中 Azure Kubernetes 服务 (AKS)
description: 了解如何更新 Linux 节点和自动重新引导这些与 kured 中 Azure Kubernetes 服务 (AKS)
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 02/28/2019
ms.author: iainfou
ms.openlocfilehash: b426399f73375618a2084eff82abba5d4934b914
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074208"
---
# <a name="apply-security-and-kernel-updates-to-linux-nodes-in-azure-kubernetes-service-aks"></a>将安全更新和内核更新应用到 Linux 节点在 Azure Kubernetes 服务 (AKS)

若要保护你的群集，安全更新自动应用于 AKS 的 Linux 节点。 这些更新包括 OS 安全修复项或内核更新。 其中的部分更新需要重启节点才能完成更新进程。 AKS 不会自动重新启动这些 Linux 节点，以完成更新过程。

以保持 （目前以预览版在 AKS 中） 的 Windows 服务器节点的最新的过程是稍有不同。 Windows Server 节点不会接收每日更新。 相反，您执行 AKS 升级部署与最新的基本 Windows Server 映像和修补程序的新节点。 对于使用 Windows Server 节点的 AKS 群集，请参阅[升级在 AKS 中的节点池][nodepool-upgrade]。

本文介绍如何使用开放源代码[kured （KUbernetes 重新启动守护程序）] [ kured]要监视的 Linux 节点需要重新启动，然后自动处理正在运行的 pod 和节点的重新安排重新启动进程。

> [!NOTE]
> `Kured` 是 Weaveworks 提供的一个开源项目。 我们尽可能地在 AKS 中提供对该项目的支持。 可以 #weave 社区 slack 通道中找到其他支持。

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 门户][aks-quickstart-portal]。

还需安装并配置 Azure CLI 2.0.59 或更高版本。 运行  `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅 [安装 Azure CLI][install-azure-cli]。

## <a name="understand-the-aks-node-update-experience"></a>了解 AKS 节点更新体验

在 AKS 群集中，Kubernetes 节点作为 Azure 虚拟机 (VM) 运行。 这些基于 Linux 的虚拟机使用 Ubuntu 映像，其 OS 配置为每晚自动检查更新。 如果有可用的安全更新或内核更新，则会自动下载并进行安装。

![使用 kured 进行的 AKS 节点更新和重启进程](media/node-updates-kured/node-reboot-process.png)

部分安全更新（如内核更新）需要重启节点才能完成更新进程。 需要重新启动在 Linux 节点创建名为的文件 */var/run/reboot-required*。 此重启进程不会自动进行。

你可以使用自己的工作流和进程来重启节点，或使用 `kured` 安排该进程。 与`kured`、 一个[DaemonSet] [ DaemonSet]部署，在群集中每个 Linux 节点上运行 pod。 DaemonSet 中的这些 pod 可监视是否存在 /var/run/reboot-required 文件，然后启动重启节点的进程。

### <a name="node-upgrades"></a>节点升级

AKS 中还有额外的进程，可通过该进程升级群集。 升级通常是指移动到 Kubernetes 的较新版本，而不仅是应用节点安全更新。 AKS 升级执行以下操作：

* 部署新节点，应用最新的安全更新和 Kubernetes 版本。
* 封锁并排除旧节点。
* 将 Pod 调度到新节点上。
* 删除旧节点。

在升级事件期间，无法继续使用同一 Kubernetes 版本。 必须指定 Kubernetes 的较新版本。 若要升级到 Kubernetes 的最新版本，可以[升级 AKS 群集][aks-upgrade]。

## <a name="deploy-kured-in-an-aks-cluster"></a>在 AKS 群集中部署 kured

要部署 `kured` DaemonSet，请从以下示例 YAML 清单的 GitHub 项目页应用它们。 此清单创建角色、群集角色、绑定和服务帐户，然后使用支持 AKS 群集 1.9 或更高版本的 `kured` 1.1.0 版部署 DaemonSet。

```console
kubectl apply -f https://github.com/weaveworks/kured/releases/download/1.1.0/kured-1.1.0.yaml
```

也可以为 `kured` 配置其他参数，例如与 Prometheus 或 Slack 集成。 有关其他配置参数的详细信息，请参阅 [kured 安装文档][kured-install]。

## <a name="update-cluster-nodes"></a>更新群集节点

默认情况下，在 AKS 中的 Linux 节点检查更新每天晚上。 如果不想等待，可以手动执行更新以检查 `kured` 是否正常运行。 首先，按照步骤[与任意 AKS 节点建立 SSH 连接][aks-ssh]。 与 Linux 节点建立 SSH 连接后，检查有更新，并将其应用，如下所示：

```console
sudo apt-get update && sudo apt-get upgrade -y
```

如果所应用的更新需要重启节点，则会将文件写入 /var/run/reboot-required。 默认情况下，`Kured` 每 60 分钟检查一次需要重启的节点。

## <a name="monitor-and-review-reboot-process"></a>监视和查看重启进程

如果 DaemonSet 中的某个副本检测到某个节点需要重启，系统将通过 Kubernetes API 对该节点进行锁定。 该锁定可防止将其他 pod 调度到此节点上。 该锁定还指示一次只应重启一个节点。 封锁节点后，将从节点中排除运行中的 pod，并重启节点。

可使用 [kubectl get nodes][kubectl-get-nodes] 命令监视节点的状态。 以下示例输出显示了一个在节点准备进行重启时状态为 SchedulingDisabled 的节点：

```
NAME                       STATUS                     ROLES     AGE       VERSION
aks-nodepool1-28993262-0   Ready,SchedulingDisabled   agent     1h        v1.11.7
```

更新过程完成后，可使用带有 `--output wide` 参数的 [kubectl get nodes][kubectl-get-nodes] 命令查看节点的状态。 通过此附加输出，可发现基础节点的 KERNEL-VERSION 会有所差异，如以下示例输出所示。 *Aks nodepool1 28993262 0*已在上一步并显示了内核版本中更新*4.15.0-1039-azure*。 在节点*aks nodepool1 28993262 1*的尚未更新的显示的内核版本*4.15.0-1037-azure*。

```
NAME                       STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-28993262-0   Ready     agent     1h        v1.11.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS   4.15.0-1039-azure   docker://3.0.4
aks-nodepool1-28993262-1   Ready     agent     1h        v1.11.7   10.240.0.5    <none>        Ubuntu 16.04.6 LTS   4.15.0-1037-azure   docker://3.0.4
```

## <a name="next-steps"></a>后续步骤

本文详细介绍如何使用`kured`作为安全更新过程的一部分自动重新启动 Linux 节点。 若要升级到 Kubernetes 的最新版本，可以[升级 AKS 群集][aks-upgrade]。

对于使用 Windows Server 节点的 AKS 群集，请参阅[升级在 AKS 中的节点池][nodepool-upgrade]。

<!-- LINKS - external -->
[kured]: https://github.com/weaveworks/kured
[kured-install]: https://github.com/weaveworks/kured#installation
[kubectl-get-nodes]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[DaemonSet]: concepts-clusters-workloads.md#statefulsets-and-daemonsets
[aks-ssh]: ssh.md
[aks-upgrade]: upgrade-cluster.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
