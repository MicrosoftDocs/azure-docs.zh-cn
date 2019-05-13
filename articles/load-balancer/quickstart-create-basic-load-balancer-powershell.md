---
title: 快速入门：创建基本负载均衡器 - Azure PowerShell
titlesuffix: Azure Load Balancer
description: 本快速入门介绍了如何使用 PowerShell 创建基本负载均衡器
services: load-balancer
documentationcenter: na
author: KumudD
manager: twooley
Customer intent: I want to create a Basic Load balancer so that I can load balance internet traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2019
ms.author: kumud
ms:custom: seodec18
ms.openlocfilehash: 31795b0dfb5a9815113ab232a266d9f7f8955068
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64688385"
---
# <a name="get-started"></a>快速入门：使用 Azure PowerShell 创建公共负载均衡器

本快速入门介绍了如何使用 Azure PowerShell 创建基本负载均衡器。 为了测试负载均衡器，需要部署两台运行 Windows 服务器的虚拟机 (VM)，并在 VM 之间对一个 Web 应用进行负载均衡。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果选择在本地安装并使用 PowerShell，则本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 来创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组

创建负载均衡器之前，必须使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 创建资源组。 以下示例在 *EastUS* 位置创建名为 *myResourceGroupLB* 的资源组：

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName "myResourceGroupLB" `
  -Location "EastUS"
```

## <a name="create-a-public-ip-address"></a>创建公共 IP 地址
若要通过 Internet 访问应用，需要负载均衡器的一个公共 IP 地址。 使用 [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) 创建一个公共 IP 地址。 以下示例在 *myResourceGroupLB* 资源组中创建名为 *myPublicIP* 的公用 IP 地址：

```azurepowershell-interactive
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroupLB" `
  -Location "EastUS" `
  -AllocationMethod "Static" `
  -Name "myPublicIP"
```

## <a name="create-basic-load-balancer"></a>创建基本负载均衡器

在本部分中，将为负载均衡器配置前端 IP 和后端地址池，然后创建基本负载均衡器。

### <a name="create-front-end-ip"></a>创建前端 IP

使用 [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) 创建一个前端 IP。 以下示例创建名为 *myFrontEnd* 的前端 IP 配置并附加 *myPublicIP* 地址：

```azurepowershell-interactive
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEnd" `
  -PublicIpAddress $publicIP
```

### <a name="configure-back-end-address-pool"></a>配置后端地址池

使用 [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) 创建一个后端地址池。 在剩余步骤中，VM 将连接到此后端池。 以下示例创建名为“myBackEndPool”的后端地址池：

```azurepowershell-interactive
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
```

### <a name="create-a-health-probe"></a>创建运行状况探测器
若要允许负载均衡器监视应用的状态，可以使用运行状况探测器。 运行状况探测器基于其对运行状况检查的响应，从负载均衡器中动态添加或删除 VM。 默认情况下，在 15 秒时间间隔内发生两次连续的故障后，将从负载均衡器分布中删除 VM。 可以为应用创建基于协议或特定运行状况检查页面的运行状况探测器。 

以下示例创建一个 TCP 探测。 还可创建自定义 HTTP 探测，以便执行更精细的运行状况检查。 使用自定义 HTTP 探测时，必须创建运行状况检查页，例如 healthcheck.aspx。 探测必须为负载均衡器返回 HTTP 200 OK 响应，以保持主机处于旋转状态。

若要创建 TCP 运行状况探测，请使用 [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig)。 以下示例创建名为 *myHealthProbe* 的运行状况探测，用于在 *HTTP* 端口 *80* 上监视每台 VM：

```azurepowershell-interactive
$probe = New-AzLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -RequestPath healthcheck2.aspx `
  -Protocol http `
  -Port 80 `
  -IntervalInSeconds 16 `
  -ProbeCount 2
  ```

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则用于定义将流量分配给 VM 的方式。 定义传入流量的前端 IP 配置和后端 IP 池以接收流量，同时定义所需的源端口和目标端口。 若要确保仅正常运行的 VM 接收流量，还需定义要使用的运行状况探测。

使用 [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) 创建一个负载均衡器规则。 以下示例创建名为“myLoadBalancerRule”的负载均衡器规则并均衡 *TCP* 端口 *80* 上的流量：

```azurepowershell-interactive
$lbrule = New-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

### <a name="create-the-nat-rules"></a>创建 NAT 规则

使用 [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) 创建 NAT 规则。 以下示例创建名为 *myLoadBalancerRDP1* 和 *myLoadBalancerRDP2* 的 NAT 规则来允许通过端口 4221 和 4222 建立到后端服务器的 RDP 连接：

```azurepowershell-interactive
$natrule1 = New-AzLoadBalancerInboundNatRuleConfig `
-Name 'myLoadBalancerRDP1' `
-FrontendIpConfiguration $frontendIP `
-Protocol tcp `
-FrontendPort 4221 `
-BackendPort 3389

$natrule2 = New-AzLoadBalancerInboundNatRuleConfig `
-Name 'myLoadBalancerRDP2' `
-FrontendIpConfiguration $frontendIP `
-Protocol tcp `
-FrontendPort 4222 `
-BackendPort 3389
```

### <a name="create-load-balancer"></a>创建负载均衡器

使用 [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) 创建基本负载均衡器。 以下示例使用在前面的步骤中创建的前端 IP 配置、后端池、运行状况探测、负载均衡规则和 NAT 规则创建名为 myLoadBalancer 的公用基本负载均衡器：

```azurepowershell-interactive
$lb = New-AzLoadBalancer `
-ResourceGroupName 'myResourceGroupLB' `
-Name 'MyLoadBalancer' `
-Location 'eastus' `
-FrontendIpConfiguration $frontendIP `
-BackendAddressPool $backendPool `
-Probe $probe `
-LoadBalancingRule $lbrule `
-InboundNatRule $natrule1,$natrule2
```

## <a name="create-network-resources"></a>创建网络资源

必须创建支持网络资源（虚拟网络和虚拟 NIC），才能部署某些 VM 并测试均衡器。 

### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络。 以下示例创建具有“mySubnet”的名为“myVnet”的虚拟网络：

```azurepowershell-interactive
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.2.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupLB" `
  -Location "EastUS" `
  -Name "myVnet" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnetConfig
```

### <a name="create-network-security-group"></a>创建网络安全组

创建网络安全组，以定义虚拟网络的入站连接。

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>为端口 3389 创建网络安全组规则

使用 [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则以允许通过端口 3389 创建 RDP 连接。

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleRDP' `
-Description 'Allow RDP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 1000 `
-SourceAddressPrefix Internet `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 3389
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>为端口 80 创建网络安全组规则

使用 [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则以允许通过端口 80 创建入站连接。

```azurepowershell-interactive
$rule2 = New-AzNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleHTTP' `
-Description 'Allow HTTP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 2000 `
-SourceAddressPrefix Internet `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 80
```

#### <a name="create-a-network-security-group"></a>创建网络安全组

使用 [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) 创建网络安全组。

```azurepowershell-interactive
$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'myNetworkSecurityGroup' `
-SecurityRules $rule1,$rule2
```

### <a name="create-nics"></a>创建 NIC

使用 [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 创建虚拟 NIC。 以下示例创建两个虚拟 NIC。 （在以下步骤中为应用创建的每个 VM 各使用一个虚拟 NIC）。 可随时创建其他虚拟 NIC 和 VM，并将其添加到负载均衡器：

```azurepowershell-interactive
# Create NIC for VM1
$nicVM1 = New-AzNetworkInterface `
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'MyVM1' `
-LoadBalancerBackendAddressPool $backendPool `
-NetworkSecurityGroup $nsg `
-LoadBalancerInboundNatRule $natrule1 `
-Subnet $vnet.Subnets[0]

# Create NIC for VM2
$nicVM2 = New-AzNetworkInterface `
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'MyVM2' `
-LoadBalancerBackendAddressPool $backendPool `
-NetworkSecurityGroup $nsg `
-LoadBalancerInboundNatRule $natrule2 `
-Subnet $vnet.Subnets[0]
```

### <a name="create-virtual-machines"></a>创建虚拟机

要提高应用的高可用性，请将 VM 放置在可用性集中。

使用 [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) 创建一个可用性集。 以下示例创建名为“myAvailabilitySet”的可用性集：

```azurepowershell-interactive
$availabilitySet = New-AzAvailabilitySet `
  -ResourceGroupName "myResourceGroupLB" `
  -Name "myAvailabilitySet" `
  -Location "EastUS" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

使用 [New-AzureRmNetworkInterface](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置 VM 的管理员用户名和密码：

```azurepowershell-interactive
$cred = Get-Credential
```

现在，可使用 [New-AzVM](/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例创建两台 VM 和所需的虚拟网络组件（如果它们尚不存在）。 在此示例中，上一步中创建的 NIC（VM1 和 VM2）将自动分配给虚拟机 VM1 和 VM2，因为它们具有相同的名称，并分配了相同的虚拟网络 (myVnet) 和子网 (mySubnet)。 此外，由于 NIC 与负载均衡器的后端池关联，因此 VM 会自动添加到该后端池。

```azurepowershell-interactive
for ($i=1; $i -le 2; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupLB" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

`-AsJob` 参数以后台任务的方式创建 VM，因此 PowerShell 提示符会返回到你所在的位置。 可以通过 `Job` cmdlet 查看后台作业的详细信息。 创建和配置这两台 VM 需要几分钟的时间来完成。

### <a name="install-iis-with-custom-web-page"></a>安装具有自定义网页的 IIS
 
在两台后端 VM 上安装具有自定义网页的 IIS：

1. 获取负载均衡器的公用 IP 地址。 使用 `Get-AzPublicIPAddress`，获取负载均衡器的公用 IP 地址。

   ```azurepowershell-interactive
    Get-AzPublicIPAddress `
    -ResourceGroupName "myResourceGroupLB" `
    -Name "myPublicIP" | select IpAddress
   ```
2. 使用在上一步骤中获取的公用 IP 地址创建到 VM1 的远程桌面连接。 

   ```azurepowershell-interactive

      mstsc /v:PublicIpAddress:4221  
  
   ```
3. 输入 *VM1* 的凭据来启动 RDP 会话。
4. 在 VM1 上启动 Windows PowerShell 并使用以下命令安装 IIS 服务器并更新默认的 htm 文件。
    ```azurepowershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello from" + $env:computername)
    ```
5. 关闭与 *myVM1* 之间的 RDP 连接。
6. 通过运行 `mstsc /v:PublicIpAddress:4222` 命令创建与 *myVM2* 的 RDP 连接，并为 *VM2* 重复步骤 4。

## <a name="test-load-balancer"></a>测试负载均衡器
使用 [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) 获取负载均衡器的公共 IP 地址。 以下示例获取前面创建的“myPublicIP”的 IP 地址：

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupLB" `
  -Name "myPublicIP" | select IpAddress
```

然后，可将公共 IP 地址输入 web 浏览器中。 随即显示网站，包括负载均衡器将流量分发到的 VM 的主机名，如下例所示：

![测试负载均衡器](media/quickstart-create-basic-load-balancer-powershell/load-balancer-test.png)

若要查看负载均衡器如何在运行应用的两台 VM 之间分配流量，可以强制刷新 Web 浏览器。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、VM 和所有相关的资源，可以使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupLB
```

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何创建基本负载均衡器，向其附加 VM，配置负载均衡器流量规则、运行状况探测，然后测试负载均衡器。 若要了解有关 Azure 负载均衡器的详细信息，请继续学习 Azure 负载均衡器教程。

> [!div class="nextstepaction"]
> [Azure 负载均衡器教程](tutorial-load-balancer-basic-internal-portal.md)