---
title: Azure CLI 脚本示例 - 筛选 VM 网络流量 | Microsoft Docs
description: Azure CLI 脚本示例 - 筛选入站和出站 VM 网络流量。
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.openlocfilehash: 549f966f86e9ddd447f24e21027504395981fd1e
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64729368"
---
# <a name="filter-inbound-and-outbound-vm-network-traffic-script-sample"></a>筛选入站和出站 VM 网络流量脚本示例

该脚本示例创建了包含前端和后端子网的虚拟网络。 前端子网的入站网络流量仅限于 HTTP、HTTPS 和 SSH，而从后端子网到 Internet 的出站流量则不受允许。 运行该脚本后，将具有一个包含两个 NIC 的虚拟机。 每个 NIC 连接到不同的子网。

可以通过 Azure [Cloud Shell](https://shell.azure.com/bash) 或本地 Azure CLI 安装来执行脚本。 如果在本地使用 CLI，此脚本要求运行版本 2.0.28 或更高版本。 要查找已安装的版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。 如果在本地运行 CLI，则还需运行 `az login` 以创建与 Azure 的连接。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/filter-network-traffic/filter-network-traffic.sh  "Filter VM network traffic")]

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源：

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟网络和网络安全组。 下表中的每条命令均链接到特定于命令的文档：

| 命令 | 说明 |
|---|---|
| [az group create](/cli/azure/group) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](/cli/azure/network/vnet) | 创建 Azure 虚拟网络和前端子网。 |
| [az network subnet create](/cli/azure/network/vnet/subnet) | 创建后端子网。 |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet) | 将 NSG 关联到子网。 |
| [az network public-ip create](/cli/azure/network/public-ip) | 创建用于从 Internet 访问 VM 的公共 IP 地址。 |
| [az network nic create](/cli/azure/network/nic) | 创建虚拟网络接口，并将其附加到虚拟网络的前端和后端子网。 |
| [az network nsg create](/cli/azure/network/nsg) | 创建关联到前端和后端子网的网络安全组 (NSG)。 |
| [az network nsg rule create](/cli/azure/network/nsg/rule) |创建 NSG 规则，允许或阻止特定子网的特定端口。 |
| [az vm create](/cli/azure/vm) | 创建虚拟机，并将 NIC 附加到每个 VM。 此命令还指定要使用的虚拟机映像和管理凭据。 |
| [az group delete](/cli/azure/group) | 删除资源组及其包含的所有资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](/cli/azure)。

可在[虚拟网络 CLI 示例](../cli-samples.md)中查找其他虚拟网络 CLI 脚本示例。
