---
title: 部署和配置 Azure 防火墙使用 Azure PowerShell
description: 在本文中，您将了解如何部署和配置 Azure 防火墙使用 Azure PowerShell。
services: firewall
author: vhorne
ms.service: firewall
ms.date: 4/10/2019
ms.author: victorh
ms.openlocfilehash: 7c30e0aa0ae9735f5d08e1a2c4d6e6d36d778e27
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410229"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-powershell"></a>部署和配置 Azure 防火墙使用 Azure PowerShell

控制出站网络访问是整个网络安全计划的重要组成部分。 例如，你可能想要限制对网站的访问， 或者限制可以访问的出站 IP 地址和端口。

可以控制 Azure 子网的出站网络访问的一种方法是使用 Azure 防火墙。 使用 Azure 防火墙，可以配置：

* 应用程序规则，用于定义可从子网访问的完全限定域名 (FQDN)。
* 网络规则，用于定义源地址、协议、目标端口和目标地址。

将网络流量路由到用作子网默认网关的防火墙时，网络流量受到配置的防火墙规则的控制。

在本文中，创建具有三个子网，以便于部署的简化单个 VNet。 对于生产部署，我们建议使用[中心辐射模型](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)，其中，防火墙在其自身的 VNet 中。 工作负荷服务器在包含一个或多个子网的同一区域中的对等 VNet 内。

* **AzureFirewallSubnet** - 防火墙在此子网中。
* **Workload-SN** - 工作负荷服务器在此子网中。 此子网的网络流量通过防火墙。
* **Jump-SN** -“跳转”服务器在此子网中。 可以使用远程桌面连接到跳转服务器中的公共 IP 地址。 然后，可在跳转服务器中（使用另一个远程桌面）连接到工作负荷服务器。

![教程网络基础结构](media/tutorial-firewall-rules-portal/Tutorial_network.png)

在本文中，学习如何：

> [!div class="checklist"]
> * 设置测试网络环境
> * 部署防火墙
> * 创建默认路由
> * 配置一个应用程序规则以允许访问 www.google.com
> * 配置网络规则，以允许访问外部 DNS 服务器
> * 测试防火墙

如果您愿意，可以完成此过程使用[Azure 门户](tutorial-firewall-deploy-portal.md)。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>必备组件

此过程要求在本地运行 PowerShell。 必须安装 Azure PowerShell 模块。 运行 `Get-Module -ListAvailable Az` 即可查找版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 验证 PowerShell 版本以后，请运行 `Connect-AzAccount`，以便创建与 Azure 的连接。

## <a name="set-up-the-network"></a>设置网络

首先，创建一个资源组用于包含部署防火墙所需的资源。 然后创建 VNet、子网和测试服务器。

### <a name="create-a-resource-group"></a>创建资源组

资源组包含部署的所有资源。

```azurepowershell
New-AzResourceGroup -Name Test-FW-RG -Location "East US"
```

### <a name="create-a-vnet"></a>创建 VNet

此虚拟网络具有三个子网：

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name AzureFirewallSubnet -AddressPrefix 10.0.1.0/24
$Worksub = New-AzVirtualNetworkSubnetConfig -Name Workload-SN -AddressPrefix 10.0.2.0/24
$Jumpsub = New-AzVirtualNetworkSubnetConfig -Name Jump-SN -AddressPrefix 10.0.3.0/24
```

> [!NOTE]
> AzureFirewallSubnet 子网的最小大小为 /26。

现在，创建虚拟网络：

```azurepowershell
$testVnet = New-AzVirtualNetwork -Name Test-FW-VN -ResourceGroupName Test-FW-RG `
-Location "East US" -AddressPrefix 10.0.0.0/16 -Subnet $FWsub, $Worksub, $Jumpsub
```

### <a name="create-virtual-machines"></a>创建虚拟机

现在，创建跳转虚拟机和工作负荷虚拟机，并将其放入相应的子网。
出现提示时，请键入该虚拟机的用户名和密码。

创建 Srv 跳转虚拟机。

```azurepowershell
New-AzVm `
    -ResourceGroupName Test-FW-RG `
    -Name "Srv-Jump" `
    -Location "East US" `
    -VirtualNetworkName Test-FW-VN `
    -SubnetName Jump-SN `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

使用没有公共 IP 地址创建工作负荷的虚拟机。
出现提示时，请键入该虚拟机的用户名和密码。

```azurepowershell
#Create the NIC
$NIC = New-AzNetworkInterface -Name Srv-work -ResourceGroupName Test-FW-RG `
 -Location "East US" -Subnetid $testVnet.Subnets[1].Id 

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName Srv-Work -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Srv-Work -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName Test-FW-RG -Location "East US" -VM $VirtualMachine -Verbose
```

## <a name="deploy-the-firewall"></a>部署防火墙

现在部署到虚拟网络中防火墙。

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName Test-FW-RG `
  -Location "East US" -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name Test-FW01 -ResourceGroupName Test-FW-RG -Location "East US" -VirtualNetworkName Test-FW-VN -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP
```

记下专用 IP 地址。 稍后在创建默认路由时需要用到此地址。

## <a name="create-a-default-route"></a>创建默认路由

创建表，请禁用 BGP 路由传播

```azurepowershell
$routeTableDG = New-AzRouteTable `
  -Name Firewall-rt-table `
  -ResourceGroupName Test-FW-RG `
  -location "East US" `
  -DisableBgpRoutePropagation

#Create a route
 Add-AzRouteConfig `
  -Name "DG-Route" `
  -RouteTable $routeTableDG `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $testVnet `
  -Name Workload-SN `
  -AddressPrefix 10.0.2.0/24 `
  -RouteTable $routeTableDG | Set-AzVirtualNetwork
```

## <a name="configure-an-application-rule"></a>配置应用程序规则

应用程序规则允许对 www.google.com 出站访问。

```azurepowershell
$AppRule1 = New-AzFirewallApplicationRule -Name Allow-Google -SourceAddress 10.0.2.0/24 `
  -Protocol http, https -TargetFqdn www.google.com

$AppRuleCollection = New-AzFirewallApplicationRuleCollection -Name App-Coll01 `
  -Priority 200 -ActionType Allow -Rule $AppRule1

$Azfw.ApplicationRuleCollections = $AppRuleCollection

Set-AzFirewall -AzureFirewall $Azfw
```

Azure 防火墙包含默认情况下允许的基础结构 FQDN 的内置规则集合。 这些 FQDN 特定于平台，不能用于其他目的。 有关详细信息，请参阅[基础结构 FQDN](infrastructure-fqdns.md)。

## <a name="configure-a-network-rule"></a>配置网络规则

网络规则允许在端口 53 (DNS) 的两个 IP 地址对出站访问。

```azurepowershell
$NetRule1 = New-AzFirewallNetworkRule -Name "Allow-DNS" -Protocol UDP -SourceAddress 10.0.2.0/24 `
   -DestinationAddress 209.244.0.3,209.244.0.4 -DestinationPort 53

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 200 `
   -Rule $NetRule1 -ActionType "Allow"

$Azfw.NetworkRuleCollections = $NetRuleCollection

Set-AzFirewall -AzureFirewall $Azfw
```

### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>更改 **Srv-Work** 网络接口的主要和辅助 DNS 地址

出于测试目的在此过程中，配置服务器的主要和辅助 DNS 地址。 这并不是一项常规的 Azure 防火墙要求。

```azurepowershell
$NIC.DnsSettings.DnsServers.Add("209.244.0.3")
$NIC.DnsSettings.DnsServers.Add("209.244.0.4")
$NIC | Set-AzNetworkInterface
```

## <a name="test-the-firewall"></a>测试防火墙

现在测试防火墙，以确认它是否按预期方式工作。

1. 请注意的专用 IP 地址**Srv 工作**虚拟机：

   ```
   $NIC.IpConfigurations.PrivateIpAddress
   ```

1. 将远程桌面连接到 **Srv-Jump** 虚拟机，然后登录。 在此处，打开远程桌面连接到**Srv 工作**专用 IP 地址并登录。

3. 上**SRV 工作**，打开 PowerShell 窗口并运行以下命令：

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   这两个命令应返回显示 DNS 查询将获取通过防火墙的答案。

1. 运行以下命令：

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   Www.google.com 请求应该会成功，而且 www.microsoft.com 请求应当会失败。 此示例演示按预期运行你的防火墙规则。

现已验证防火墙规则可正常工作：

* 可以使用配置的外部 DNS 服务器解析 DNS 名称。
* 可以浏览到一个允许的 FQDN，但不能浏览到其他任何 FQDN。

## <a name="clean-up-resources"></a>清理资源

可以将你的防火墙资源保留为下一步的教程中，或如果不再需要删除**测试转发 RG**要删除防火墙相关的所有资源的资源组：

```azurepowershell
Remove-AzResourceGroup -Name Test-FW-RG
```

## <a name="next-steps"></a>后续步骤

* [教程：监视 Azure 防火墙日志](./tutorial-diagnostics.md)
