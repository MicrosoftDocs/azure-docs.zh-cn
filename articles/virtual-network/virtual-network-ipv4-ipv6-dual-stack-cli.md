---
title: 部署 IPv6 双堆栈应用程序 - 基本负载均衡器 - CLI
titlesuffix: Azure Virtual Network
description: 本文介绍如何使用 Azure CLI 在 Azure 虚拟网络中部署 IPv6 双堆栈应用程序。
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: 396c37d4c8de6a890102e435c5ec6cc70b598638
ms.sourcegitcommit: 7581df526837b1484de136cf6ae1560c21bf7e73
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/31/2020
ms.locfileid: "80421022"
---
# <a name="deploy-an-ipv6-dual-stack-application-using-basic-load-balancer---cli"></a>部署使用基本负载均衡器的 IPv6 双堆栈应用程序 - CLI

本文介绍如何使用 Azure CLI 部署一个具有基本负载均衡器的双堆栈 (IPv4 + IPv6) 应用程序，其中包含具有双堆栈子网的双堆栈虚拟网络、采用双重 (IPv4 + IPv6) 前端配置的基本负载均衡器、具有采用双重 IP 配置的 NIC 的 VM、双重网络安全组规则，以及双重公共 IP。

若要部署使用标准负载均衡器的双堆栈 (IPV4 + IPv6) 应用程序，请参阅[使用 Azure CLI 部署具有标准负载均衡器的 IPv6 双堆栈应用程序](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-cli.md)。


如果还没有 Azure 订阅，请现在就创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果你决定在本地安装并使用 Azure CLI，本快速入门要求使用 Azure CLI 2.0.49 或更高版本。 若要查找已安装的版本，请运行 `az --version`。 有关安装或升级信息，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

在创建双堆栈虚拟网络之前，必须先使用 [az group create](/cli/azure/group) 创建一个资源组。 下面的示例*在东部*位置创建名为*DsResourceGroup01*的资源组：

```azurecli
az group create \
--name DsResourceGroup01 \
--location eastus
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses-for-load-balancer"></a>为负载均衡器创建 IPv4 和 IPv6 公共 IP 地址
若要在 Internet 上访问 IPv4 和 IPv6 终结点，需要为负载均衡器创建 IPv4 和 IPv6 公共 IP 地址。 使用 [az network public-ip create](/cli/azure/network/public-ip) 创建公共 IP 地址。 以下示例在 *DsResourceGroup01* 资源组中创建名为 *dsPublicIP_v4* 与 *dsPublicIP_v6* 的 IPv4 和 IPv6 公共 IP 地址：

```azurecli
# Create an IPV4 IP address
az network public-ip create \
--name dsPublicIP_v4  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku BASIC  \
--allocation-method dynamic  \
--version IPv4

# Create an IPV6 IP address
az network public-ip create \
--name dsPublicIP_v6  \
--resource-group DsResourceGroup01  \
--location eastus \
--sku BASIC  \
--allocation-method dynamic  \
--version IPv6

```

## <a name="create-public-ip-addresses-for-vms"></a>为 VM 创建公共 IP 地址

若要在 Internet 上远程访问 VM，需要为 VM 创建 IPv4 公共 IP 地址。 使用 [az network public-ip create](/cli/azure/network/public-ip) 创建公共 IP 地址。

```azurecli
az network public-ip create \
--name dsVM0_remote_access  \
--resource-group DsResourceGroup01 \
--location eastus  \
--sku BASIC  \
--allocation-method dynamic  \
--version IPv4

az network public-ip create \
--name dsVM1_remote_access  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku BASIC  \
--allocation-method dynamic  \
--version IPv4
```

## <a name="create-basic-load-balancer"></a>创建基本负载均衡器

在本部分，你将为负载均衡器配置双重前端 IP（IPv4 和 IPv6）与后端地址池，然后创建基本负载均衡器。

### <a name="create-load-balancer"></a>创建负载均衡器

使用 [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest) 创建名为 **dsLB** 的基本负载均衡器，其中包含名为 **dsLbFrontEnd_v4** 的前端池以及名为 **dsLbBackEndPool_v4** 的后端池（与在上一步骤中创建的 IPv4 公共 IP 地址 **dsPublicIP_v4** 相关联）。 

```azurecli
az network lb create \
--name dsLB  \
--resource-group DsResourceGroup01 \
--sku Basic \
--location eastus \
--frontend-ip-name dsLbFrontEnd_v4  \
--public-ip-address dsPublicIP_v4  \
--backend-pool-name dsLbBackEndPool_v4
```

### <a name="create-ipv6-frontend"></a>创建 IPv6 前端

使用 [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az-network-lb-frontend-ip-create) 创建 IPv6 前端 IP。 以下示例创建名为 *dsLbFrontEnd_v6* 的前端 IP 配置，并附加 *dsPublicIP_v6* 地址：

```azurecli
az network lb frontend-ip create \
--lb-name dsLB  \
--name dsLbFrontEnd_v6  \
--resource-group DsResourceGroup01  \
--public-ip-address dsPublicIP_v6

```

### <a name="configure-ipv6-back-end-address-pool"></a>配置 IPv6 后端地址池

使用 [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool?view=azure-cli-latest#az-network-lb-address-pool-create) 创建 IPv6 后端地址池。 以下示例创建名为*dsLbBackEndPool_v6*的后端地址池，以包括具有 IPv6 NIC 配置的 VM：

```azurecli
az network lb address-pool create \
--lb-name dsLB  \
--name dsLbBackEndPool_v6  \
--resource-group DsResourceGroup01
```

### <a name="create-a-health-probe"></a>创建运行状况探测器
使用 [az network lb probe create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create) 创建运行状况探测，以监视虚拟机的运行状况。 

```azurecli
az network lb probe create -g DsResourceGroup01  --lb-name dsLB -n dsProbe --protocol tcp --port 3389
```

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则用于定义将流量分配给 VM 的方式。 定义传入流量的前端 IP 配置和后端 IP 池以接收流量，同时定义所需的源端口和目标端口。 

使用 [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) 创建负载均衡器规则。 以下示例创建名为 *dsLBrule_v4* 和 *dsLBrule_v6* 的负载均衡器规则，并通过 IPv4 和 IPv6 前端 IP 配置均衡 *TCP* 端口 *80* 上的流量：

```azurecli
az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v4  \
--resource-group DsResourceGroup01  \
--frontend-ip-name dsLbFrontEnd_v4  \
--protocol Tcp  \
--frontend-port 80  \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v4


az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v6  \
--resource-group DsResourceGroup01 \
--frontend-ip-name dsLbFrontEnd_v6  \
--protocol Tcp  \
--frontend-port 80 \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v6

```

## <a name="create-network-resources"></a>创建网络资源
在部署某些 VM 之前，必须创建支持性的网络资源 - 可用性集、网络安全组、虚拟网络和虚拟 NIC。 
### <a name="create-an-availability-set"></a>创建可用性集
若要提高应用的可用性，请将 VM 放到可用性集中。

使用 [az vm availability-set create](https://docs.microsoft.com/zh-cn/cli/azure/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-create) 创建可用性集。 以下示例创建名为 *dsAVset* 的可用性集：

```azurecli
az vm availability-set create \
--name dsAVset  \
--resource-group DsResourceGroup01  \
--location eastus \
--platform-fault-domain-count 2  \
--platform-update-domain-count 2  
```

### <a name="create-network-security-group"></a>创建网络安全组

创建一个网络安全组，以通过其中的规则控制 VNET 中的入站和出站通信。

#### <a name="create-a-network-security-group"></a>创建网络安全组

使用 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) 创建网络安全组


```azurecli
az network nsg create \
--name dsNSG1  \
--resource-group DsResourceGroup01  \
--location eastus

```

#### <a name="create-a-network-security-group-rule-for-inbound-and-outbound-connections"></a>针对入站和出站连接创建网络安全组规则

使用 [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) 创建网络安全组规则，以允许通过端口 3389 进行 RDP 连接、通过端口 80 进行 Internet 连接，并允许出站连接。

```azurecli
# Create inbound rule for port 3389
az network nsg rule create \
--name allowRdpIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 100  \
--description "Allow Remote Desktop In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges 3389

# Create inbound rule for port 80
az network nsg rule create \
--name allowHTTPIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 200  \
--description "Allow HTTP In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges 80  \
--destination-address-prefixes "*"  \
--destination-port-ranges 80

# Create outbound rule

az network nsg rule create \
--name allowAllOut  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 300  \
--description "Allow All Out"  \
--access Allow  \
--protocol "*"  \
--direction Outbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges "*"
```


### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-create) 创建虚拟网络。 以下示例创建名为 *dsVNET* 的虚拟网络，其中包含子网 *dsSubNET_v4* 和 *dsSubNET_v6*：

```azurecli
# Create the virtual network
az network vnet create \
--name dsVNET \
--resource-group DsResourceGroup01 \
--location eastus  \
--address-prefixes "10.0.0.0/16" "ace:cab:deca::/48"

# Create a single dual stack subnet

az network vnet subnet create \
--name dsSubNET \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--address-prefixes "10.0.0.0/24" "ace:cab:deca:deed::/64" \
--network-security-group dsNSG1
```

### <a name="create-nics"></a>创建 NIC

使用 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) 为每个 VM 创建虚拟 NIC。 以下示例为每个 VM 创建一个虚拟 NIC。 每个 NIC 采用两个 IP 配置（1 个 IPv4 配置，1 个 IPv6 配置）。 使用 [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-create) 创建 IPv6 配置。

```azurecli
# Create NICs
az network nic create \
--name dsNIC0  \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1  \
--vnet-name dsVNET  \
--subnet dsSubNet  \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4  \
--lb-name dsLB  \
--public-ip-address dsVM0_remote_access

az network nic create \
--name dsNIC1 \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4 \
--lb-name dsLB \
--public-ip-address dsVM1_remote_access

# Create IPV6 configurations for each NIC

az network nic ip-config create \
--name dsIp6Config_NIC0  \
--nic-name dsNIC0  \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name dsNIC1 \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB
```

### <a name="create-virtual-machines"></a>创建虚拟机

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create) 创建 VM。 以下示例创建两台 VM 和所需的虚拟网络组件（如果它们尚不存在）。 

按如下所示创建虚拟机 *dsVM0*：

```azurecli
 az vm create \
--name dsVM0 \
--resource-group DsResourceGroup01 \
--nics dsNIC0 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest  
```

按如下所示创建虚拟机 *dsVM1*：

```azurecli
az vm create \
--name dsVM1 \
--resource-group DsResourceGroup01 \
--nics dsNIC1 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest 
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>在 Azure 门户中查看 IPv6 双堆栈虚拟网络
可以在 Azure 门户中查看 IPv6 双堆栈虚拟网络，如下所示：
1. 在门户的搜索栏中输入 *dsVnet*。
2. 当“myVirtualNetwork”出现在搜索结果中时，将其选中。**** 此时会启动名为 *dsVnet* 的双堆栈虚拟网络的“概述”页。**** 该双堆栈虚拟网络显示了位于 *dsSubnet* 双堆栈子网中的两个 NIC，这些 NIC 采用 IPv4 和 IPv6 配置。

  ![Azure 中的 IPv6 双堆栈虚拟网络](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-vnet.png)



## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、VM 和所有相关的资源，可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令将其删除。

```azurecli
 az group delete --name DsResourceGroup01
```

## <a name="next-steps"></a>后续步骤

在本文中，你已使用双重前端 IP 配置（IPv4 和 IPv6）创建了一个基本负载均衡器。 你还创建了两个虚拟机，它们包含采用双重 IP 配置（IPV4 + IPv6）的 NIC，并已添加到负载均衡器的后端池。 要了解有关 Azure 虚拟网络中 IPv6 支持的更多信息，请参阅[什么是 Azure 虚拟网络的 IPv6？](ipv6-overview.md)
