---
title: 通过 SSH 登录到 Azure Kubernetes 服务 (AKS) 群集节点
description: 了解如何与 Azure Kubernetes 服务 (AKS) 群集节点建立 SSH 连接，以完成故障排除和维护任务。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 03/05/2019
ms.author: iainfou
ms.openlocfilehash: d421fad5f574b0d10b24453aca01adf574f493e8
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407709"
---
# <a name="connect-with-ssh-to-azure-kubernetes-service-aks-cluster-nodes-for-maintenance-or-troubleshooting"></a>使用 SSH 连接到 Azure Kubernetes 服务 (AKS) 群集节点以进行维护或故障排除

在 Azure Kubernetes 服务 (AKS) 群集的整个生命周期内，可能需要访问 AKS 节点。 进行这种访问的原因包括维护、日志收集或其他故障排除操作。 AKS 节点是 Linux VM，因此可以使用 SSH 访问它们。 出于安全考虑，AKS 节点不会在 Internet 中公开。

本文介绍如何使用 AKS 节点的专用 IP 地址来与它们建立 SSH 连接。

## <a name="before-you-begin"></a>开始之前

本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 门户][aks-quickstart-portal]。

还需安装并配置 Azure CLI 2.0.59 或更高版本。 运行  `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅 [安装 Azure CLI][install-azure-cli]。

## <a name="add-your-public-ssh-key"></a>添加 SSH 公钥

默认情况下，在创建 AKS 群集时会生成 SSH 密钥。 如果在创建 AKS 群集时未指定自己的 SSH 密钥，请将 SSH 公钥添加到 AKS 节点。

若要将 SSH 密钥添加到 AKS 节点，请完成以下步骤：

1. 使用 [az aks show][az-aks-show] 获取 AKS 群集资源的资源组名称。 提供自己的核心资源组和 AKS 群集名称：

    ```azurecli-interactive
    az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
    ```

1. 使用 [az vm list][az-vm-list] 命令列出 AKS 群集资源组中的 VM。 这些 VM 是 AKS 节点：

    ```azurecli-interactive
    az vm list --resource-group MC_myResourceGroup_myAKSCluster_eastus -o table
    ```

    以下示例输出显示 AKS 节点：

    ```
    Name                      ResourceGroup                                  Location
    ------------------------  ---------------------------------------------  ----------
    aks-nodepool1-79590246-0  MC_myResourceGroupAKS_myAKSClusterRBAC_eastus  eastus
    ```

1. 若要将 SSH 密钥添加到节点，请使用 [az vm user update][az-vm-user-update] 命令。 提供资源组名称，然后提供在上一步骤中获取的 AKS 节点之一。 默认情况下，AKS 节点的用户名为 *azureuser*。 提供自己的 SSH 公钥位置（例如 *~/.ssh/id_rsa.pub*），或粘贴 SSH 公钥的内容：

    ```azurecli-interactive
    az vm user update \
      --resource-group MC_myResourceGroup_myAKSCluster_eastus \
      --name aks-nodepool1-79590246-0 \
      --username azureuser \
      --ssh-key-value ~/.ssh/id_rsa.pub
    ```

## <a name="get-the-aks-node-address"></a>获取 AKS 节点地址

AKS 节点不会在 Internet 中公开。 若要通过 SSH 连接到 AKS 节点，需使用专用 IP 地址。 下一步将在 AKS 群集中创建一个帮助器 Pod，以允许你通过 SSH 连接到节点的此专用 IP 地址。

使用 [az vm list-ip-addresses][az-vm-list-ip-addresses] 命令查看 AKS 群集节点的专用 IP 地址。 提供在前面 [az-aks-show][az-aks-show] 步骤中获取的自己的 AKS 群集资源组名称：

```azurecli-interactive
az vm list-ip-addresses --resource-group MC_myResourceGroup_myAKSCluster_eastus -o table
```

以下示例输出显示 AKS 节点的专用 IP 地址：

```
VirtualMachine            PrivateIPAddresses
------------------------  --------------------
aks-nodepool1-79590246-0  10.240.0.4
```

## <a name="create-the-ssh-connection"></a>建立 SSH 连接

若要与 AKS 节点建立 SSH 连接，请在 AKS 群集中运行帮助器 pod。 使用此帮助器 pod 可以通过 SSH 依次访问群集和其他 SSH 节点。 若要创建并使用此帮助器 pod，请完成以下步骤：

1. 运行 `debian` 容器映像，并在其上附加一个终端会话。 可以使用此容器来与 AKS 群集中的任何节点建立 SSH 会话：

    ```console
    kubectl run -it --rm aks-ssh --image=debian
    ```

1. 基础 Debian 映像不包含 SSH 组件。 将终端会话连接到该容器后，按如下所示使用 `apt-get` 安装 SSH 客户端：

    ```console
    apt-get update && apt-get install openssh-client -y
    ```

1. 在未连接到容器的新终端窗口中，使用 [kubectl get pod][kubectl-get] 命令列出 AKS 群集中的 pod。 在上一步骤中创建的 pod 以名称 *aks-ssh* 开头，如以下示例所示：

    ```
    $ kubectl get pods
    
    NAME                       READY     STATUS    RESTARTS   AGE
    aks-ssh-554b746bcf-kbwvf   1/1       Running   0          1m
    ```

1. 本文的第一个步骤已将 SSH 公钥添加到了 AKS 节点。 现在，请将 SSH 私钥复制到 pod 中。 此私钥用来与 AKS 节点建立 SSH 连接。

    提供在上一步骤中获取的自己的 *aks-ssh* pod 名称。 如果需要，请将 *~/.ssh/id_rsa* 更改为 SSH 私钥的位置：

    ```console
    kubectl cp ~/.ssh/id_rsa aks-ssh-554b746bcf-kbwvf:/id_rsa
    ```

1. 返回到与容器建立的终端会话，更新复制的 `id_rsa` SSH 私钥中的权限，使其成为用户只读的密钥：

    ```console
    chmod 0600 id_rsa
    ```

1. 现在，与 AKS 节点建立 SSH 连接。 同样，AKS 节点的默认用户名为 *azureuser*。 遵照提示继续建立连接，因为一开始就信任 SSH 密钥。 然后，系统会提供 AKS 节点的 bash 提示：

    ```console
    $ ssh -i id_rsa azureuser@10.240.0.4
    
    ECDSA key fingerprint is SHA256:A6rnRkfpG21TaZ8XmQCCgdi9G/MYIMc+gFAuY9RUY70.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.240.0.4' (ECDSA) to the list of known hosts.
    
    Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1018-azure x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      Get cloud support with Ubuntu Advantage Cloud Guest:
        https://www.ubuntu.com/business/services/cloud
    
    [...]
    
    azureuser@aks-nodepool1-79590246-0:~$
    ```

## <a name="remove-ssh-access"></a>删除 SSH 访问

完成后，运行 `exit` 退出 SSH 会话，然后运行 `exit` 退出交互式容器会话。 此容器会话关闭后，将删除用于从 AKS 群集进行 SSH 访问的 pod。

## <a name="next-steps"></a>后续步骤

如需其他故障排除数据，可以[查看 kubelet 日志][view-kubelet-logs]或[查看 Kubernetes 主节点日志][view-master-logs]。

<!-- EXTERNAL LINKS -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- INTERNAL LINKS -->
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-vm-list]: /cli/azure/vm#az-vm-list
[az-vm-user-update]: /cli/azure/vm/user#az-vm-user-update
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[view-kubelet-logs]: kubelet-logs.md
[view-master-logs]: view-master-logs.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
