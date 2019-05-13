---
title: 诊断虚拟机网络流量筛选器问题 - 快速入门 - Azure CLI | Microsoft Docs
description: 本快速入门介绍了如何使用 Azure 网络观察程序的 IP 流验证功能来诊断虚拟机网络流量筛选器问题。
services: network-watcher
documentationcenter: network-watcher
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 460513e4818cbef8fca0cd1b84d69b3021afaab7
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64690431"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-cli"></a>快速入门：诊断虚拟机网络流量筛选器问题 - Azure CLI

在本快速入门中，请先部署虚拟机 (VM)，然后检查到某个 IP 地址和 URL 的通信以及来自某个 IP 地址的通信。 确定通信失败的原因以及解决方法。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本快速入门要求运行 Azure CLI 2.0.28 或更高版本。 要查找已安装的版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。 验证 CLI 版本以后，请运行 `az login`，以便创建与 Azure 的连接。 本快速入门中的 CLI 命令已格式化，适合在 Bash Shell 中运行。

## <a name="create-a-vm"></a>创建 VM

在创建 VM 之前，必须创建该 VM 所属的资源组。 使用 [az group create](/cli/azure/group) 创建资源组。 以下示例在 eastus 位置创建名为 myResourceGroup 的资源组：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az vm create](/cli/azure/vm) 创建 VM。 如果默认密钥位置中尚不存在 SSH 密钥，该命令会创建它们。 若要使用特定的一组密钥，请使用 `--ssh-key-value` 选项。 以下示例创建名为 myVm 的 VM：

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

创建 VM 需要几分钟时间。 在创建好 VM 且 CLI 返回输出之前，请勿继续执行剩余的步骤。

## <a name="test-network-communication"></a>测试网络通信

若要通过网络观察程序测试网络通信，必须先在要测试的 VM 所在的区域中启用网络观察程序，然后使用网络观察程序的 IP 流验证功能来测试通信。

### <a name="enable-network-watcher"></a>启用网络观察程序

如果已在“美国东部”区域中启用网络观察程序，请跳到[使用 IP 流验证](#use-ip-flow-verify)。 使用 [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) 命令在“美国东部”区域中创建网络观察程序：

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-ip-flow-verify"></a>使用 IP 流验证

创建 VM 时，Azure 在默认情况下会允许或拒绝出入 VM 的网络流量。 可以在以后覆盖 Azure 的默认设置，允许或拒绝其他类型的流量。 若要测试来自一个源 IP 地址但发往不同目标的流量是获得允许还是被拒绝，请使用 [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow) 命令。

测试从 VM 发往 www.bing.com 的某个 IP 地址的出站通信：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

数秒钟后返回结果，指示名为 **AllowInternetOutbound** 的安全规则已允许访问。

测试从 VM 发往 172.31.0.100 的出站通信：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

返回的结果指示名为 **DefaultOutboundDenyAll** 的安全规则已拒绝了访问。

测试从 172.31.0.100 发往 VM 的入站通信：

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

返回的结果指示名为 **DefaultInboundDenyAll** 的安全规则已拒绝了访问。 了解哪些安全规则允许或拒绝出入 VM 的流量以后，即可确定问题解决方法。

## <a name="view-details-of-a-security-rule"></a>查看安全规则的详细信息

若要确定[使用 IP 流验证](#use-ip-flow-verify)中的规则允许或阻止通信的原因，请使用 [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) 命令查看网络接口的有效安全规则：

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

返回的输出包含 **AllowInternetOutbound** 规则的以下文本，该规则在[使用 IP 流验证](#use-ip-flow-verify)下的前述步骤中允许对 www.bing.com 进行出站访问：

```azurecli
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

可以在上述输出中看到 **destinationAddressPrefix** 为 **Internet**。 尚不清楚 13.107.21.200 与 **Internet** 的关系如何。 可以看到多个地址前缀列在 **expandedDestinationAddressPrefix** 下。 列表中的一个前缀是 **12.0.0.0/6**，涵盖了 IP 地址范围 12.0.0.1-15.255.255.254。 由于 13.107.21.200 在该地址范围内，因此 **AllowInternetOutBound** 规则允许此出站流量。 另外，在上述输出中没有显示优先级更高（数字更小）的可以覆盖此规则的规则。 若要拒绝到某个 IP 地址的出站通信，可以添加一项优先级更高的安全规则，拒绝通过端口 80 向该 IP 地址发送出站流量。

在[使用 IP 流验证](#use-ip-flow-verify)中运行 `az network watcher test-ip-flow` 命令以测试发往 172.131.0.100 的出站通信时，输出指示 **DefaultOutboundDenyAll** 规则拒绝了该通信。 **DefaultOutboundDenyAll** 规则相当于在 `az network nic list-effective-nsg` 命令的以下输出中列出的 **DenyAllOutBound** 规则：

```azurecli
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

该规则将 **0.0.0.0/0** 列为 **destinationAddressPrefix**。 此规则拒绝到 172.131.0.100 的出站通信，因为此地址不在 `az network nic list-effective-nsg` 命令输出中的任何其他出站规则的 **destinationAddressPrefix** 范围内。 若要允许出站通信，可以添加一项优先级更高的安全规则，允许出站流量到达 172.131.0.100 的端口 80。

在[使用 IP 流验证](#use-ip-flow-verify)中运行 `az network watcher test-ip-flow` 命令以测试来自 172.131.0.100 的入站通信时，输出指示 **DefaultInboundDenyAll** 规则拒绝了该通信。 **DefaultInboundDenyAll** 规则相当于在 `az network nic list-effective-nsg` 命令的以下输出中列出的 **DenyAllInBound** 规则：

```azurecli
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

**DenyAllInBound** 规则会应用，因为如 `az network nic list-effective-nsg` 命令的输出所示，没有任何其他允许端口 80 将入站流量从 172.131.0.100 发往 VM 的规则有更高的优先级。 若要允许入站通信，可以添加一项优先级更高的安全规则，允许通过端口 80 从 172.131.0.100 发送入站流量。

本快速入门中的检查测试了 Azure 配置。 如果检查返回预期的结果，而网络问题仍然存在，请确保在 VM 和要与之通信的终结点之间没有防火墙，且 VM 中的操作系统没有防火墙来允许或拒绝通信。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组及其包含的所有资源，可以使用 [az group delete](/cli/azure/group) 将其删除：

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已创建 VM 并对入站和出站网络流量筛选器进行诊断。 你已了解了如何通过网络安全组规则来允许或拒绝出入 VM 的流量。 请详细了解[安全规则](../virtual-network/security-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)以及如何[创建安全规则](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule)。

即使相应的网络流量筛选器已就位，与 VM 的通信仍可能因路由配置问题而失败。 若要了解如何诊断 VM 网络路由问题，请参阅[诊断 VM 路由问题](diagnose-vm-network-routing-problem-cli.md)；若要使用某个工具诊断出站路由、延迟和流量筛选问题，请参阅[排查连接问题](network-watcher-connectivity-cli.md)。
