---
title: Azure CLI 脚本示例 - 通过网络虚拟设备路由流量 | Microsoft Docs
description: Azure CLI 脚本示例 - 通过防火墙网络虚拟设备路由流量。
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
ms.openlocfilehash: 2e5ec39e2d8fe1636018ac59dba372bd0faf31f4
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64716072"
---
# <a name="route-traffic-through-a-network-virtual-appliance-script-sample"></a>通过网络虚拟设备脚本示例路由流量

该脚本示例创建了包含前端和后端子网的虚拟网络。 它还会创建一个 VM，并启用 IP 转发，在两个子网之间路由流量。 运行脚本后，可将网络软件（例如防火墙应用程序）部署到 VM。

可以通过 Azure [Cloud Shell](https://shell.azure.com/bash) 或本地 Azure CLI 安装来执行脚本。 如果在本地使用 CLI，此脚本要求运行版本 2.0.28 或更高版本。 要查找已安装的版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。 如果在本地运行 CLI，则还需运行 `az login` 以创建与 Azure 的连接。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>示例脚本

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.sh "Route traffic through a network virtual appliance")]

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
| [az network subnet create](/cli/azure/network/vnet/subnet) | 创建后端子网和 DMZ 子网。 |
| [az network public-ip create](/cli/azure/network/public-ip) | 创建用于从 Internet 访问 VM 的公共 IP 地址。 |
| [az network nic create](/cli/azure/network/nic) | 创建虚拟网络接口，并对它启用 IP 转发。 |
| [az network nsg create](/cli/azure/network/nsg) | 创建网络安全组 (NSG)。 |
| [az network nsg rule create](/cli/azure/network/nsg/rule) | 创建允许 HTTP 和 HTTPS 端口入站到 VM 的 NSG 规则。 |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet)| 将 NSG 和路由表关联到子网。 |
| [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create)| 为所有路由创建路由表。 |
| [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create)| 创建路由，通过 VM 在子网和 Internet 之间路由流量。 |
| [az vm create](/cli/azure/vm) | 创建虚拟机并向其附加 NIC。 此命令还指定要使用的虚拟机映像和管理凭据。 |
| [az group delete](/cli/azure/group) | 删除资源组及其包含的所有资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](/cli/azure)。

可在[虚拟网络 CLI 示例](../cli-samples.md)中查找其他虚拟网络 CLI 脚本示例。
