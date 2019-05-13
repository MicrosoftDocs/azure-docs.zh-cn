---
title: 创建 NFS （网络文件系统） Ubuntu 服务器以供 pod 的 Azure Kubernetes 服务 (AKS)
description: 了解如何手动创建 NFS Ubuntu Linux 服务器卷，以便使用具有 pod 中 Azure Kubernetes 服务 (AKS)
services: container-service
author: ozboms
ms.service: container-service
ms.topic: article
ms.date: 4/25/2019
ms.author: obboms
ms.openlocfilehash: 55eb5b0b98a4097d2f300bacabbfef3b0a32b27b
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65468494"
---
# <a name="manually-create-and-use-an-nfs-network-file-system-linux-server-volume-with-azure-kubernetes-service-aks"></a>手动创建和使用 NFS （网络文件系统） Linux 服务器卷使用 Azure Kubernetes 服务 (AKS)
容器之间共享数据通常是基于容器的服务和应用程序的必需组件。 你通常具有需要外部的永久性卷上的相同信息的访问的各种 pod。    
尽管 Azure 文件是一个选项，Azure VM 上创建 NFS 服务器是另一种形式的永久共享存储。 

本文将演示如何创建 Ubuntu 虚拟机上的 NFS 服务器。 还将 AKS 容器访问权限提供给此共享的文件系统。

## <a name="before-you-begin"></a>开始之前
本文假定你拥有现有的 AKS 群集。 如果需要 AKS 群集，请参阅 AKS 快速入门[使用 Azure CLI] [ aks-quickstart-cli]或[使用 Azure 门户][aks-quickstart-portal]。

在 AKS 群集需要位于相同或对等互连虚拟网络与 NFS 服务器。 群集必须创建在现有的 VNET，可以是与 VM 相同的 VNET。

文档中介绍了使用现有的 VNET 配置的步骤：[在现有的 VNET 中创建 AKS 群集][ aks-virtual-network]和[使用 VNET 对等互连连接虚拟网络][peer-virtual-networks]

它还假定已创建的 Ubuntu Linux 虚拟机 (例如，18.04 LTS)。 设置和大小可以根据需要，并且可以通过 Azure 部署。 有关 Linux 快速入门中，请参阅[管理的 linux VM][linux-create]。

如果首次部署 AKS 群集，Azure 将自动填充虚拟网络字段时部署 Ubuntu 计算机，使其成为实时的同一 VNET 中。 但如果你想要改为使用对等网络，请参阅上面的文档。

## <a name="deploying-the-nfs-server-onto-a-virtual-machine"></a>部署到虚拟机上的 NFS 服务器
下面是用于设置 NFS 服务器在 Ubuntu 虚拟机中的脚本：
```bash
#!/bin/bash

# This script should be executed on Linux Ubuntu Virtual Machine

EXPORT_DIRECTORY=${1:-/export/data}
DATA_DIRECTORY=${2:-/data}
AKS_SUBNET=${3:-*}

echo "Updating packages"
apt-get -y update

echo "Installing NFS kernel server"

apt-get -y install nfs-kernel-server

echo "Making data directory ${DATA_DIRECTORY}"
mkdir -p ${DATA_DIRECTORY}

echo "Making new directory to be exported and linked to data directory: ${EXPORT_DIRECTORY}"
mkdir -p ${EXPORT_DIRECTORY}

echo "Mount binding ${DATA_DIRECTORY} to ${EXPORT_DIRECTORY}"
mount --bind ${DATA_DIRECTORY} ${EXPORT_DIRECTORY}

echo "Giving 777 permissions to ${EXPORT_DIRECTORY} directory"
chmod 777 ${EXPORT_DIRECTORY}

parentdir="$(dirname "$EXPORT_DIRECTORY")"
echo "Giving 777 permissions to parent: ${parentdir} directory"
chmod 777 $parentdir

echo "Appending bound directories into fstab"
echo "${DATA_DIRECTORY}    ${EXPORT_DIRECTORY}   none    bind  0  0" >> /etc/fstab

echo "Appending localhost and Kubernetes subnet address ${AKS_SUBNET} to exports configuration file"
echo "/export        ${AKS_SUBNET}(rw,async,insecure,fsid=0,crossmnt,no_subtree_check)" >> /etc/exports
echo "/export        localhost(rw,async,insecure,fsid=0,crossmnt,no_subtree_check)" >> /etc/exports

nohup service nfs-kernel-server restart
```
服务器将重启 （由于脚本），可以装载到 AKS NFS 服务器

>[!IMPORTANT]  
>请务必替换**AKS_SUBNET**与正确的订阅从你的群集或 else"*"将打开所有端口并连接到 NFS 服务器。

创建 VM 后，将上述脚本复制到一个文件。 然后，你可以将它从本地计算机，或脚本是任何位置，为 VM，并使用： 
```console
scp /path/to/script_file username@vm-ip-address:/home/{username}
```
后脚本在 VM 中，可以 ssh 到 VM 并通过命令执行：
```console
sudo ./nfs-server-setup.sh
```
如果由于权限被拒错误而失败的执行，通过命令的执行权限设置：
```console
chmod +x ~/nfs-server-setup.sh
```

## <a name="connecting-aks-cluster-to-nfs-server"></a>连接到 NFS 服务器的 AKS 群集
我们可以通过预配的永久性卷和永久性卷声明，用于指定如何访问该卷连接到群集的 NFS 服务器。  
连接两个服务在相同或对等互连虚拟网络中的是必需的。 此处的设置在同一 VNET 中群集的说明：[在现有的 VNET 中创建 AKS 群集][aks-virtual-network]

一旦它们位于同一虚拟网络中 （或对等互连），就必须预配永久性卷和永久性卷声明在 AKS 群集中。 然后，容器可以装载到其本地目录的 NFS 驱动器。

下面是永久性卷 （此定义假定你的群集和 VM 位于同一 VNET 中） 的示例 kubernetes 定义：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <NFS_NAME>
  labels:
    type: nfs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <NFS_INTERNAL_IP>
    path: <NFS_EXPORT_FILE_PATH>
```
替换**NFS_INTERNAL_IP**， **NFS_NAME**并**NFS_EXPORT_FILE_PATH**与 NFS 服务器的信息。

您还需要永久性卷声明文件。 下面是要包含的内容的示例：

>[!IMPORTANT]  
>**"storageClassName"** 需要保留为空字符串或声明不起作用。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <NFS_NAME>
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
  selector: 
    matchLabels:
      type: nfs
```

## <a name="troubleshooting"></a>故障排除
如果无法从群集连接到服务器，导出的目录或其父级可能出现问题，但没有足够的权限来访问服务器。

检查你导出的目录和其父目录都具有 777 权限。

可以通过运行以下命令检查权限和目录应有*drwxrwxrwx*权限：
```console
ls -l
```

## <a name="more-information"></a>详细信息
若要获取的完整演练或帮助您调试 NFS 服务器安装程序，以下是深入教程：
  - [NFS 教程][nfs-tutorial]

## <a name="next-steps"></a>后续步骤

如需相关的最佳做法，请参阅[在 AKS 中存储和备份的最佳做法][operator-best-practices-storage]。

<!-- LINKS - external -->
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[linux-create]: https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-vm
[nfs-tutorial]: https://help.ubuntu.com/community/SettingUpNFSHowTo#Pre-Installation_Setup
[aks-virtual-network]: https://docs.microsoft.com/azure/aks/configure-kubenet#create-an-aks-cluster-in-the-virtual-network
[peer-virtual-networks]: https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[operator-best-practices-storage]: operator-best-practices-storage.md
