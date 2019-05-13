---
title: 快速入门：创建标准负载均衡器 - Azure PowerShell
titlesuffix: Azure Load Balancer
description: 本快速入门介绍如何使用 PowerShell 创建标准负载均衡器
services: load-balancer
documentationcenter: na
author: KumudD
manager: twooley
Customer intent: I want to create a Standard Load balancer so that I can load balance internet traffic to VMs.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2019
ms.author: kumud
ms:custom: seodec18
ms.openlocfilehash: 513d3931c31ca94b4089139992bceb6f679caa98
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467705"
---
# <a name="quickstart-create-a-standard-load-balancer-using-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建标准负载均衡器

本快速入门介绍如何使用 Azure PowerShell 创建标准负载均衡器。 为了测试负载均衡器，需要部署三台运行 Windows 服务器的虚拟机 (VM)，并在 VM 之间对一个 Web 应用进行负载均衡。 若要了解有关标准负载均衡器的详细信息，请参阅[什么是标准负载均衡器](load-balancer-standard-overview.md)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果选择在本地安装并使用 PowerShell，则本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 来创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组

创建负载均衡器之前，必须使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 创建资源组。 以下示例在 *EastUS* 位置创建名为 *myResourceGroupSLB* 的资源组：

```azurepowershell
$rgName='MyResourceGroupSLB'
$location='eastus'
New-AzResourceGroup -Name $rgName -Location $location
```

## <a name="create-a-public-ip-address"></a>创建公共 IP 地址

若要通过 Internet 访问应用，需要负载均衡器的一个公共 IP 地址。 使用 [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) 创建一个公共 IP 地址。 以下示例在 *myResourceGroupSLB* 资源组中创建名为 *myPublicIP* 的公共 IP 地址：

```azurepowershell
$publicIp = New-AzPublicIpAddress `
 -ResourceGroupName $rgName `
 -Name 'myPublicIP' `
 -Location $location `
 -AllocationMethod static `
 -SKU Standard
```

## <a name="create-standard-load-balancer"></a>创建标准负载均衡器

在本部分中，将为负载均衡器配置前端 IP 和后端地址池，然后创建标准负载均衡器。

### <a name="create-front-end-ip"></a>创建前端 IP

使用 [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) 创建一个前端 IP。 以下示例创建名为 *myFrontEnd* 的前端 IP 配置并附加 *myPublicIP* 地址：

```azurepowershell
$feip = New-AzLoadBalancerFrontendIpConfig -Name 'myFrontEndPool' -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>配置后端地址池

使用 [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) 创建一个后端地址池。 在剩余步骤中，VM 将连接到此后端池。 以下示例创建名为“myBackEndPool”的后端地址池：

```azurepowershell-interactive
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'
```

### <a name="create-a-health-probe"></a>创建运行状况探测器
若要允许负载均衡器监视应用的状态，可以使用运行状况探测器。 运行状况探测器基于其对运行状况检查的响应，从负载均衡器中动态添加或删除 VM。 默认情况下，在 15 秒时间间隔内发生两次连续的故障后，将从负载均衡器分布中删除 VM。 可以为应用创建基于协议或特定运行状况检查页面的运行状况探测器。

以下示例创建一个 TCP 探测。 还可创建自定义 HTTP 探测，以便执行更精细的运行状况检查。 使用自定义 HTTP 探测时，必须创建运行状况检查页，例如 healthcheck.aspx。 探测必须为负载均衡器返回 HTTP 200 OK 响应，以保持主机处于旋转状态。

若要创建 TCP 运行状况探测，请使用 [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig)。 以下示例创建名为 *myHealthProbe* 的运行状况探测，用于在 *HTTP* 端口 *80* 上监视每台 VM：

```azurepowershell
$probe = New-AzLoadBalancerProbeConfig `
 -Name 'myHealthProbe' `
 -Protocol Http -Port 80 `
 -RequestPath / -IntervalInSeconds 360 -ProbeCount 5
```

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则
负载均衡器规则用于定义将流量分配给 VM 的方式。 定义传入流量的前端 IP 配置和后端 IP 池以接收流量，同时定义所需源和目标端口。 若要确保仅正常运行的 VM 接收流量，还需定义要使用的运行状况探测。

使用 [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) 创建一个负载均衡器规则。 以下示例创建名为“myLoadBalancerRule”的负载均衡器规则并均衡 *TCP* 端口 *80* 上的流量：

```azurepowershell
$rule = New-AzLoadBalancerRuleConfig `
  -Name 'myLoadBalancerRuleWeb' -Protocol Tcp `
  -Probe $probe -FrontendPort 80 -BackendPort 80 `
  -FrontendIpConfiguration $feip `
  -BackendAddressPool $bePool
```

### <a name="create-the-nat-rules"></a>创建 NAT 规则

使用 [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) 创建 NAT 规则。 以下示例创建名为 *myLoadBalancerRDP1* 和 *myLoadBalancerRDP2* 的 NAT 规则来允许通过端口 4221 和 4222 建立到后端服务器的 RDP 连接：

```azurepowershell
$natrule1 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP1' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp -FrontendPort 4221 `
  -BackendPort 3389

$natrule2 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP2' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp `
  -FrontendPort 4222 `
  -BackendPort 3389

$natrule3 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP3' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp `
  -FrontendPort 4223 `
  -BackendPort 3389
```

### <a name="create-load-balancer"></a>创建负载均衡器

使用 [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) 创建标准负载均衡器。 以下示例使用在前面的步骤中创建的前端 IP 配置、后端池、运行状况探测、负载均衡规则和 NAT 规则创建名为 myLoadBalancer 的公用标准负载均衡器：

```azurepowershell
$lb = New-AzLoadBalancer `
  -ResourceGroupName $rgName `
  -Name 'MyLoadBalancer' `
  -SKU Standard `
  -Location $location `
  -FrontendIpConfiguration $feip `
  -BackendAddressPool $bepool `
  -Probe $probe `
  -LoadBalancingRule $rule `
  -InboundNatRule $natrule1,$natrule2,$natrule3
```

## <a name="create-network-resources"></a>创建网络资源
必须创建支持网络资源（虚拟网络和虚拟 NIC），才能部署某些 VM 并测试均衡器。 

### <a name="create-a-virtual-network"></a>创建虚拟网络
使用 [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络。 以下示例创建具有“mySubnet”的名为“myVnet”的虚拟网络：

```azurepowershell
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.2.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupSLB" `
  -Location $location `
  -Name "myVnet" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnetConfig
```
### <a name="create-public-ip-addresses-for-the-vms"></a>创建 VM 的公共 IP 地址

若要使用 RDP 连接访问 VM，需要为 VM 创建公共 IP 地址。 由于此方案使用了标准负载均衡器，因此必须使用 [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) 为 VM 创建标准公共 IP 地址。

```azurepowershell
$RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static
 

$RdpPublicIP_2 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_2" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static


$RdpPublicIP_3 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_3" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static

```
### <a name="create-network-security-group"></a>创建网络安全组
创建网络安全组，以定义虚拟网络的入站连接。

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>为端口 3389 创建网络安全组规则
使用 [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则以允许通过端口 3389 创建 RDP 连接。

```azurepowershell

$rule1 = New-AzNetworkSecurityRuleConfig -Name 'myNetworkSecurityGroupRuleRDP' -Description 'Allow RDP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 1000 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 3389
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>为端口 80 创建网络安全组规则
使用 [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则以允许通过端口 80 创建入站连接。

```azurepowershell
$rule2 = New-AzNetworkSecurityRuleConfig -Name 'myNetworkSecurityGroupRuleHTTP' -Description 'Allow HTTP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 2000 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 80
```

#### <a name="create-a-network-security-group"></a>创建网络安全组

使用 [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) 创建网络安全组。

```azurepowershell
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RgName -Location $location `
-Name 'myNetworkSecurityGroup' -SecurityRules $rule1,$rule2
```

### <a name="create-nics"></a>创建 NIC
使用 [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 创建虚拟 NIC，并将其关联到在前面步骤中创建的公共 IP 地址和网络安全组。 以下示例创建三个虚拟 NIC。 （在以下步骤中为应用创建的每个 VM 各使用一个虚拟 NIC）。 可随时创建其他虚拟 NIC 和 VM，并将其添加到负载均衡器：

```azurepowershell
# Create NIC for VM1
$nicVM1 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic1' -PublicIpAddress $RdpPublicIP_1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule1 -Subnet $vnet.Subnets[0]

$nicVM2 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic2' -PublicIpAddress $RdpPublicIP_2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule2 -Subnet $vnet.Subnets[0]

$nicVM3 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic3' -PublicIpAddress $RdpPublicIP_3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule3 -Subnet $vnet.Subnets[0]
```

### <a name="create-virtual-machines"></a>创建虚拟机

使用 [New-AzureRmNetworkInterface](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置 VM 的管理员用户名和密码：

```azurepowershell
$cred = Get-Credential
```

现在，可使用 [New-AzVM](/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例创建两台 VM 和所需的虚拟网络组件（如果它们尚不存在）。 在此示例中，在前一步骤中创建的 NIC（*MyNic1*、*MyNic2* 和 *MyNic3*）将分配到虚拟机 *myVM1*、*myVM2* 和 *VM3*。 此外，由于 NIC 与负载均衡器的后端池关联，因此 VM 会自动添加到该后端池。

```azurepowershell

# ############## VM1 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM1' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM1' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
 | Add-AzVMNetworkInterface -Id $nicVM1.Id

# Create a virtual machine
$vm1 = New-AzVM -ResourceGroupName $rgName -Zone 1 -Location $location -VM $vmConfig

# ############## VM2 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM2' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM2' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
 | Add-AzVMNetworkInterface -Id $nicVM2.Id

# Create a virtual machine
$vm2 = New-AzVM -ResourceGroupName $rgName -Zone 2 -Location $location -VM $vmConfig

# ############## VM3 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM3' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM3' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
| Add-AzVMNetworkInterface -Id $nicVM3.Id

# Create a virtual machine
$vm3 = New-AzVM -ResourceGroupName $rgName -Zone 3 -Location $location -VM $vmConfig
```

创建和配置这三个 VM 需要几分钟的时间来完成。

### <a name="install-iis-with-a-custom-web-page"></a>安装具有自定义网页的 IIS

在两台后端 VM 上安装具有自定义网页的 IIS：

1. 使用 `Get-AzPublicIPAddress` 获取三个 VM 的公共 IP 地址。

   ```azurepowershell
     $vm1_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_1").IpAddress
     $vm2_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_2").IpAddress
     $vm3_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_3").IpAddress
    ```
2. 按如下所示，使用 VM 的公共 IP 地址来与 *myVM1*、*myVM2* 和 *myVM3* 建立远程桌面连接： 

   ```azurepowershell    
     mstsc /v:$vm1_rdp_ip
     mstsc /v:$vm2_rdp_ip
     mstsc /v:$vm3_rdp_ip
   
    ```

3. 输入每个 VM 的凭据以启动 RDP 会话。
4. 在每个 VM 上启动 Windows PowerShell 并使用以下命令安装 IIS 服务器并更新默认的 htm 文件。

    ```azurepowershell
    # Install IIS
     Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from host " + $env:computername)
    ```

5. 断开与 *myVM1*、*myVM2* 和 *myVM3* 的 RDP 连接。


## <a name="test-load-balancer"></a>测试负载均衡器
使用 [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) 获取负载均衡器的公共 IP 地址。 以下示例获取前面创建的“myPublicIP”的 IP 地址：

```azurepowershell
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupSLB" `
  -Name "myPublicIP" | select IpAddress
```

然后，可将公共 IP 地址输入 web 浏览器中。 随即显示网站，包括负载均衡器将流量分发到的 VM 的主机名，如下例所示：

![测试负载均衡器](media/quickstart-create-basic-load-balancer-powershell/load-balancer-test.png)

若要查看负载均衡器如何在运行应用的所有 3 个 VM 之间分配流量，可强制刷新 web 浏览器。 

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、VM 和所有相关的资源，可以使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurepowershell
Remove-AzResourceGroup -Name myResourceGroupSLB
```

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何创建标准负载均衡器，向其附加 VM，配置负载均衡器流量规则、运行状况探测，然后测试负载均衡器。 若要了解有关 Azure 负载均衡器的详细信息，请继续学习 Azure 负载均衡器教程。

> [!div class="nextstepaction"]
> [Azure 负载均衡器教程](tutorial-load-balancer-basic-internal-portal.md)
