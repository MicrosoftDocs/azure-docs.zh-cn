---
title: Azure PowerShell 脚本示例 - 管理 Web 流量 | Microsoft Docs
description: Azure PowerShell 脚本示例 - 使用应用程序网关和虚拟机规模集管理 Web 流量。
services: application-gateway
documentationcenter: networking
author: vhorne
manager: jpconnock
editor: tysonn
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: b9b34f124d1260fe17d1f43c6e97f20fd116c5f8
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65202844"
---
# <a name="manage-web-traffic-with-azure-powershell"></a>使用 Azure PowerShell 管理 Web 流量

此脚本创建使用虚拟机规模集作为后端服务器的应用程序网关。 然后，可以对应用程序网关进行配置来管理 Web 流量。 在运行脚本后，可以使用应用程序网关的公用 IP 地址测试该网关。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-powershell[main](../../../powershell_scripts/application-gateway/create-vmss/create-vmss.ps1 "Create application gateway")]

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、应用程序网关和所有相关资源。

```powershell
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建部署。 表中的每一项均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | 创建子网配置。 |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | 使用子网配置创建虚拟网络。 |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | 创建应用程序网关的公用 IP 地址。 |
| [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration) | 创建将子网与应用程序网关相关联的配置。 |
| [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig) | 创建为应用程序网关分配公用 IP 地址的配置。 |
| [New-AzApplicationGatewayFrontendPort](/powershell/module/az.network/new-azapplicationgatewayfrontendport) | 分配用于访问应用程序网关的端口。 |
| [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool) | 创建应用程序网关的后端池。 |
| [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) | 配置后端池的设置。 |
| [New-AzApplicationGatewayHttpListener](/powershell/module/az.network/new-azapplicationgatewayhttplistener) | 创建侦听器。 |
| [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule) | 创建路由规则。 |
| [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku) | 指定应用程序网关的层和容量。 |
| [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) | 创建应用程序网关。 |
| [Set-AzVmssStorageProfile](/powershell/module/az.compute/set-azvmssstorageprofile) | 创建规模集的存储配置文件。 |
| [Set-AzVmssOsProfile](/powershell/module/az.compute/set-azvmssosprofile) | 定义规模集的操作系统。 |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.compute/add-azvmssnetworkinterfaceconfiguration) | 定义规模集的网络接口。 |
| [New-AzVmss](/powershell/module/az.compute/new-azvm) | 创建虚拟机规模集。 |
| [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) | 获取应用程序网关的公用 IP 地址。 |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组及其中包含的所有资源。 | 

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](/powershell/azure/overview)。

可以在 [Azure 应用程序网关文档](../powershell-samples.md)中找到其他应用程序网关 PowerShell 脚本示例。
