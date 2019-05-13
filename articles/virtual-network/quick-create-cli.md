---
title: 创建虚拟网络 - 快速入门 - Azure CLI
titlesuffix: Azure Virtual Network
description: 本快速入门介绍如何使用 Azure CLI 创建虚拟网络。 虚拟网络能让 Azure 资源（例如虚拟机）彼此之间私下通信以及与 Internet 进行通信。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/12/2018
ms.author: kumud
ms.openlocfilehash: 6306d893f491f93cc31b7e478afe5632e997285c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64692642"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-cli"></a>快速入门：使用 Azure CLI 创建虚拟网络

虚拟网络能让 Azure 资源（例如虚拟机 (VM)）彼此之间私下通信以及与 Internet 进行通信。 本快速入门介绍如何创建虚拟网络。 创建虚拟网络后，将两个 VM 部署到该虚拟网络中。 然后可以从 Internet 连接到 VM，并通过新的虚拟网络进行私下通信。

如果还没有 Azure 订阅，请现在就创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果决定在本地安装并使用 Azure CLI，本快速入门要求使用 Azure CLI 2.0.28 或更高版本。 若要查找已安装的版本，请运行 `az --version`。 有关安装或升级信息，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-resource-group-and-a-virtual-network"></a>创建资源组和虚拟网络

在创建虚拟网络之前，必须创建一个资源组用于托管该虚拟网络。 使用 [az group create](/cli/azure/group) 创建资源组。 此示例在 eastus 位置中创建一个名为 myResourceGroup 的资源组：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet) 创建虚拟网络。 此示例创建名为 myVirtualNetwork 的默认虚拟网络，它具有一个名为 default 的子网：

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --subnet-name default
```

## <a name="create-virtual-machines"></a>创建虚拟机

在虚拟网络中创建两个 VM。

### <a name="create-the-first-vm"></a>创建第一个 VM

使用 [az vm create](/cli/azure/vm) 创建 VM。 如果默认密钥位置中尚不存在 SSH 密钥，该命令会创建它们。 若要使用特定的一组密钥，请使用 `--ssh-key-value` 选项。 `--no-wait` 选项会在后台创建 VM，因此可继续执行下一步。 此示例创建名为 myVm1 的 VM：

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>创建第二个 VM

由于已在上一步骤中使用了 `--no-wait` 选项，因此可以继续并创建名为 myVm2 的第二个 VM。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --generate-ssh-keys
```

### <a name="azure-cli-output-message"></a>Azure CLI 输出消息

创建 VM 可能需要数分钟的时间。 Azure 创建 VM 后，Azure CLI 会返回如下输出：

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
  "zones": ""
}
```

记下 publicIpAddress。 在下一步中，此地址将用于从 Internet 连接到 VM。

## <a name="connect-to-a-vm-from-the-internet"></a>从 Internet 连接到 VM

在此命令中，将 `<publicIpAddress>` 替换为 myVm2 VM 的公用 IP 地址：

```bash
ssh <publicIpAddress>
```

## <a name="communicate-between-vms"></a>VM 之间进行通信

若要确认 myVm2 VM 和 myVm1 VM 之间的私下通信，请输入此命令：

```bash
ping myVm1 -c 4
```

将从 10.0.0.4 收到四条回复。

退出与 *myVm2* VM 的 SSH 会话。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组及其所有资源，可以使用 [az group delete](/cli/azure/group) 将其删除：

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了默认的虚拟网络和两个 VM。 从 Internet 连接到了其中一个 VM，并在两个 VM 之间进行了私下通信。 若要了解有关虚拟网络设置的详细信息，请参阅[管理虚拟网络](manage-virtual-network.md)。

Azure 可让 VM 之间进行不受限制的私下通信。 默认情况下，Azure 仅允许从 Internet 到 Windows VM 的入站远程桌面连接。 要了解有关配置不同类型的 VM 网络通信的详细信息，请转到[筛选网络流量](tutorial-filter-network-traffic.md)教程。
