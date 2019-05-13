---
title: 教程 - 使用 Azure PowerShell 自动缩放规模集 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 随 CPU 需求的增减自动缩放虚拟机规模集
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 7a592a7d0d8c9d32de83c92b258c4678dc3f8166
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60188254"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-azure-powershell"></a>教程：使用 Azure PowerShell 自动缩放虚拟机规模集

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

创建规模集时，可定义想运行的 VM 实例数。 若应用程序需要更改，可自动增加或减少 VM 实例数。 通过自动缩放功能，可随客户需求的改变而进行调整，或在应用的整个生命周期内响应应用程序性能更改。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 对规模集使用自动缩放
> * 创建和使用自动缩放规则
> * 对 VM 实例进行压力测试并触发自动缩放规则
> * 在需求下降时自动横向缩减

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

有一个已知问题影响到 Azure PowerShell 模块版本 6.8.1 或更高版本，包括 Azure Cloud Shell 的当前版本。 本教程只能使用 Azure PowerShell 模块版本 6.0.0 到 6.8.0 运行。 运行 `Get-Module -ListAvailable AzureRM` 即可查找版本。 如果在本地运行 PowerShell，则还需运行 `Connect-AzureRmAccount` 来创建与 Azure 的连接。


## <a name="create-a-scale-set"></a>创建规模集
若要更轻松地创建自动缩放规则，请为规模集定义几个变量。 以下示例为 myResourceGroup 资源组和美国东部区域内名为 myScaleSet 的规模集定义变量。 使用 [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription) 获取订阅 ID。 如果帐户关联了多个订阅，则仅返回第一个订阅。 按照如下所示，调整名称和订阅 ID：

```azurepowershell-interactive
$mySubscriptionId = (Get-AzureRmSubscription)[0].Id
$myResourceGroup = "myResourceGroup"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
```

现在，使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 创建虚拟机规模集。 若要将流量分配到单独的 VM 实例，则还要创建负载均衡器。 负载均衡器包含的规则可在 TCP 端口 80 上分配流量，并允许 TCP 端口 3389 上的远程桌面流量，以及 TCP 端口 5985 上的 PowerShell 远程流量。 出现提示时，请针对规模集中的 VM 实例提供自己的所需管理凭据：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName $myResourceGroup `
  -VMScaleSetName $myScaleSet `
  -Location $myLocation `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer"
```

创建和配置所有的规模集资源和 VM 需要几分钟时间。

## <a name="create-a-rule-to-autoscale-out"></a>创建规则，以便自动横向扩展
如果应用程序需求提高，规模集中 VM 实例上的负载将会增大。 如果这种负载增大持续稳定，而不只是短暂的需求，那么可以配置自动缩放规则来增加规模集中的 VM 实例数。 创建这些 VM 实例及部署应用程序后，规模集会开始通过负载均衡器将流量分配到这些实例和应用程序。 可以控制要监视的指标（例如 CPU 或磁盘）、应用程序负载必须处于给定阈值内的时间，以及要添加到规模集的 VM 实例数。

平均 CPU 负载大于 70% 持续了 5 分多钟时，请使用 [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) 创建规则增加规模集中的 VM 实例数。 触发规则时，VM 实例数增加 3。

此规则使用以下参数：

| 参数               | 说明                                                                                                         | 值          |
|-------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
| *-MetricName*           | 监视和应用规模集操作的性能指标。                                                   | CPU 百分比 |
| *-TimeGrain*            | 为进行而收集指标分析的频率。                                                                   | 1 分钟       |
| *-MetricStatistic*      | 定义如何聚合已收集的指标以便分析。                                                | 平均值        |
| *-TimeWindow*           | 比较指标与阈值之前监视的时长。                                   | 5 分钟      |
| *-Operator*             | 用于比较指标数据和阈值的运算符。                                                     | 大于   |
| *-Threshold*            | 使自动缩放规则触发操作的值。                                                      | 70%            |
| *-ScaleActionDirection* | 定义应用规则时，规模集应扩展还是缩减。                                             | 增加       |
| *–ScaleActionScaleType* | 表明 VM 实例数应该根据特定值进行更改。                                    | 更改计数   |
| *-ScaleActionValue*     | 规则触发时，应更改 VM 实例的百分比。                                            | 3              |
| *-ScaleActionCooldown*  | 为使自动缩放操作有时间生效，再次应用规则前需要等待的时间。 | 5 分钟      |

以下示例将创建名为 *myRuleScaleOut* 的对象，用于保存此扩展规则。 -MetricResourceId 使用以前为订阅 ID、资源组名称和规模集名称定义的变量：

```azurepowershell-interactive
$myRuleScaleOut = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -TimeGrain 00:01:00 `
  -MetricStatistic "Average" `
  -TimeWindow 00:05:00 `
  -Operator "GreaterThan" `
  -Threshold 70 `
  -ScaleActionDirection "Increase" `
  –ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 3 `
  -ScaleActionCooldown 00:05:00
```


## <a name="create-a-rule-to-autoscale-in"></a>创建规则，以便自动横向缩减
在夜间或周末，应用程序需求可能会降低。 如果这种负载降低在一段时间内持续稳定，可以配置自动缩放规则来减少规模集中的 VM 实例数。 这种横向缩减操作可以减少运行规模集所需的成本，因为只运行满足当前需求所需的实例数。

平均 CPU 负载小于 30% 持续了 5 分钟时，请使用 [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) 创建另一规则来减少规模集中的 VM 实例数。 触发规则时，VM 实例数减少 1。以下示例将创建名为 *myRuleScaleDown* 的对象，用于保存此扩展规则。 -MetricResourceId 使用以前为订阅 ID、资源组名称和规模集名称定义的变量：

```azurepowershell-interactive
$myRuleScaleIn = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator "LessThan" `
  -MetricStatistic "Average" `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection "Decrease" `
  –ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 1
```


## <a name="define-an-autoscale-profile"></a>定义自动缩放配置文件
若要将自动缩放规则与规模集相关联，请创建一个配置文件。 自动缩放配置文件定义了默认值、最小规模集容量和最大规模集容量，并与自动缩放规则关联。 使用 [New-AzureRmAutoscaleProfile](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleProfile) 创建自动缩放配置文件。 以下示例设置了默认值，以及最小容量 2 个 VM 实例、最大容量 10 个 VM。 然后附加前几步中创建的横向扩展和横向缩减规则：

```azurepowershell-interactive
$myScaleProfile = New-AzureRmAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleOut,$myRuleScaleIn `
  -Name "autoprofile"
```


## <a name="apply-autoscale-rules-to-a-scale-set"></a>将自动缩放规则应用于规模集
最后一步是将自动缩放配置文件应用于规模集。 随后，规模集便能根据应用程序需求自动进行横向扩展或缩减。 使用 [Add-AzureRmAutoscaleSetting](/powershell/module/AzureRM.Insights/Add-AzureRmAutoscaleSetting) 应用自动缩放配置文件，如下所示：

```azurepowershell-interactive
Add-AzureRmAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -AutoscaleProfile $myScaleProfile
```


## <a name="generate-cpu-load-on-scale-set"></a>在规模集上生成 CPU 负载
若要测试自动缩放规则，请在规模集的 VM 实例上生成一些 CPU 负载。 这种模拟的 CPU 负载会导致自动缩放规则以横向扩展的方式增加 VM 实例数。 随着模拟的 CPU 负载下降，自动缩放规则会进行横向缩减，减少 VM 实例数。

若要列出连接到规模集中的 VM 实例所需的 NAT 端口，请先使用 [Get-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancer) 获取负载均衡器对象， 然后使用 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) 查看入站 NAT 规则：

```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzureRmLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzureRmLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

以下示例输出显示了实例名称、负载均衡器的公共 IP 地址，以及可以通过 NAT 规则将流量转发到其中的端口号：

```powershell
Name        Protocol FrontendPort BackendPort
----        -------- ------------ -----------
myRDPRule.0 Tcp             50001        3389
myRDPRule.1 Tcp             50002        3389
```

此规则的 *Name* 与以前的 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm) 命令中显示的 VM 实例的名称相符。 例如，若要连接到 VM 实例 *0*，请使用 *myRDPRule.0* 并连接到端口 *50001*。 若要连接到 VM 实例 *1*，请使用 *myRDPRule.1* 中的值并连接到端口 *50002*。

使用 [Get-AzureRmPublicIpAddress](/powershell/module/AzureRM.Network/Get-AzureRmPublicIpAddress) 查看负载均衡器的公共 IP 地址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIP | Select IpAddress
```

示例输出：

```powershell
IpAddress
---------
52.168.121.216
```

创建连接到第一个 VM 实例所需的远程连接。 指定所需 VM 实例对应的你自己的公共 IP 地址和端口号，如前述命令所示。 出现提示时，输入创建规模集时使用的凭据（在示例命令中，默认为 *azureuser* 和 P\@ssw0rd!）。 如果使用 Azure Cloud Shell，请从本地 PowerShell 命令提示符或远程桌面客户端执行此步骤。 以下示例连接到 VM 实例 *0*：

```powershell
mstsc /v 52.168.121.216:50001
```

登录后，从任务栏打开 Internet Explorer。

- 选择“确定”，接受“使用推荐的安全性、隐私和兼容性设置”这一提示。
- 在地址栏中键入 *http://download.sysinternals.com/files/CPUSTRES.zip* 。
- 由于 Internet Explorer 增强型安全配置已启用，因此请选择“添加”，以便将 *http://download.sysinternals.com* 域添加到受信任站点的列表。
- 提示查找下载的文件时，请选择“打开”，然后选择“运行”，以便运行 *CPUSTRES.EXE* 工具。

若要生成一些 CPU 负载，请勾选“活动”线程所对应的两个框。 从两个线程的“活动”下拉菜单中选择“最大”。 可以打开任务管理器来确认 VM 上的 CPU 负载是否已达到 100%。

![CPU Stress 实用程序可以在 VM 实例上生成负载](media/tutorial-autoscale-powershell/cpu-stress-load.PNG)

让此远程桌面连接会话保持打开状态，打开另一个远程桌面连接会话。 连接到第二个 VM 实例，所使用的端口号是在前面的 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) cmdlet 中列出的：

```powershell
mstsc /v 52.168.121.216:50002
```

登录到第二个 VM 实例以后，重复以前的步骤，以便下载并运行 *CPUSTRES.EXE*。 再次启动两个“活动”线程，并将活动设置为“最大”。

请让两个远程桌面连接会话保持打开状态，以便 **CPU Stress** 工具能够继续运行。


## <a name="monitor-the-active-autoscale-rules"></a>监视活动的自动缩放规则
若要监视规模集中的 VM 实例数，请使用 **while**。 自动缩放规模需要 5 分钟的时间才能启动横向扩展过程，以便响应由每个 VM 实例上的 **CPUStress* 生成的 CPU 负载：

```azurepowershell-interactive
while (1) {Get-AzureRmVmssVM `
    -ResourceGroupName $myResourceGroup `
    -VMScaleSetName $myScaleSet; sleep 10}
```

达到 CPU 阈值以后，自动缩放规则会增加规模集中的 VM 实例数。 以下输出显示，在规模集进行自动横向扩展时创建了 3 个 VM：

```powershell
ResourceGroupName         Name Location          Sku Capacity InstanceID ProvisioningState
-----------------         ---- --------          --- -------- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_2   eastus Standard_DS2                   2         Succeeded
MYRESOURCEGROUP   myScaleSet_3   eastus Standard_DS2                   3         Succeeded
MYRESOURCEGROUP   myScaleSet_4   eastus Standard_DS2                   4          Creating
MYRESOURCEGROUP   myScaleSet_5   eastus Standard_DS2                   5          Creating
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Creating
```

在连接到每个 VM 实例的远程桌面连接会话中，关闭 **CPU Stress** 工具。 此时整个规模集的平均 CPU 负载回到正常。 另一个 5 分钟后，自动缩放规则会缩减 VM 实例数。 横向缩减操作会首先删除 ID 值最高的 VM 实例。 如果规模集使用可用性集或可用性区域，则缩减操作将均匀分布到这些 VM 实例上。 以下示例输出显示，在规模集进行自动横向缩减时删除了一个 VM 实例：

```powershell
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Deleting
```

使用 `Ctrl-c` 退出 *while*。 规模集继续每 5 分钟横向缩减一次，每次删除一个 VM 实例，直至达到最小实例计数 2。


## <a name="clean-up-resources"></a>清理资源
若要删除规模集和其他资源，请使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 删除资源组及其所有资源。 `-Force` 参数将确认是否希望删除资源，不会显示询问是否删除的额外提示。 `-AsJob` 参数会使光标返回提示符处，不会等待操作完成。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>后续步骤
本教程介绍了如何使用 Azure PowerShell 自动进行规模集的横向缩减或扩展：

> [!div class="checklist"]
> * 对规模集使用自动缩放
> * 创建和使用自动缩放规则
> * 对 VM 实例进行压力测试并触发自动缩放规则
> * 在需求下降时自动横向缩减

如需更多的虚拟机规模集操作示例，请参阅下面的 Azure PowerShell 示例脚本：

> [!div class="nextstepaction"]
> [适用于 Azure PowerShell 的规模集脚本示例](powershell-samples.md)
