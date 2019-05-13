---
title: 处理 Azure 中 Windows VM 的维护通知 | Microsoft Docs
description: 查看 Azure 中运行的 Windows 虚拟机的维护通知并开始自助式维护。
services: virtual-machines-windows
documentationcenter: ''
author: shants123
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/30/2019
ms.author: shants
ms.openlocfilehash: fccc99f78d038a5f96b9dfe01b575dedcdcb4cdc
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65405615"
---
# <a name="handling-planned-maintenance-notifications-for-windows-virtual-machines"></a>处理 Windows 虚拟机的计划内维护通知

Azure 定期执行更新，以提高虚拟机的主机基础结构的可靠性、性能及安全性。 更新包括如下更改：修补托管环境或升级以及解除硬件授权。 大多数此类更新在执行时不会影响托管的虚拟机。 但是，也会存在更新产生影响的情况：

- 如果维护不需重启，Azure 会在更新主机时使用就地迁移来暂停 VM。 各容错域将逐一应用这些无需重启的维护操作，如果收到任何警告运行状况信号，则进度停止。 

- 如果维护需重启，你会收到一个通知，其中会说明计划维护的时间。 在这些情况下，系统将提供一个时间窗口，通常为 30 天中，可以开始维护自己，当它适用于您。


需要重新启动的计划内维护是按批进行计划的。 每个批具有不同的作用域（区域）。

- 一个批从向客户发送通知开始。 默认情况下，向订阅所有者和共同所有者发送通知。 可以使用 Azure [活动日志警报](../../azure-monitor/platform/activity-logs-overview.md)，向通知添加更多收件人和消息传送选项（如电子邮件、短信和 Webhook）。  
- 在通知时会提供自助时段。 在此窗口中，通常为 30 天，期间可以找到的虚拟机包含在此批中和主动开始根据自己的计划需求的维护。
- 自助时段过后，就会开始计划内维护时段。 在此时段的某个时刻，Azure 会计划所需的维护，并将其应用于虚拟机。 

设置这两个时段的目的是，在了解 Azure 何时将自动启动维护时，提供足够的时间来启动维护和重新启动虚拟机。


可以使用 Azure 门户、PowerShell、REST API 和 CLI 查询 VM 的维护时段并启动自助式维护。

  
## <a name="should-you-start-maintenance-during-the-self-service-window"></a>是否应在自助时段启动维护？  

可以先阅读以下指南，然后再决定是否使用此功能按自己的时间来启动维护。

> [!NOTE] 
> 自助维护不一定适用于所有 VM。 若要确定是否可以对 VM 进行主动重新部署，请在维护状态中查找“立即启动”。 自助维护目前不适用于云服务（Web/辅助角色）和 Service Fabric。


使用**可用性集**进行部署时，不建议进行自助维护，因为这些可用性集是高度可用的设置，在任何给定的时间都只会影响一个更新域。 
- 让 Azure 触发维护。 对于需要重启的维护，请注意，各更新域将逐一完成维护，更新域不一定按顺序接收维护，并且更新域之间有 30 分钟的暂停。 
- 如果担心暂时性丢失部分容量（1/更新域计数），可以在维护期间分配更多的实例，这样即可轻松地进行弥补。
- 对于无需重启的维护，将在容错域级别应用更新。
    
以下情况**请勿**使用自助维护： 
- 如果频繁关闭 VM，不管是使用手动方式、使用开发测试实验室、使用自动关闭还是按计划来完成，都可能会还原维护状态，从而导致停机时间延长。
- VM 的生存期短，已确定在维护结束之前就会被删除。 
- 工作负荷的状态为“大”，存储在本地（临时）磁盘中，需要在更新后进行维护。 
- 经常需要重设 VM 大小的情况，可能还原维护状态。
- 在采用的计划事件允许对工作负荷进行主动故障转移或正常关闭的情况下，在启动维护性关闭之前的 15 分钟

如果打算在计划性维护阶段不间断地运行 VM，而且上述禁忌均不适用，则可**使用**自助维护。 

以下情况最好使用自助维护：

- 需要向管理层或最终客户告知确切的维护时段。 
- 需要在给定的日期之前完成维护。 
- 需要控制维护顺序，例如，应用程序为多层应用程序，需要确保安全地进行恢复。
- 在两个更新域 (UD) 之间，需要的 VM 恢复时间超出 30 分钟。 为了控制更新域之间的时间，一次只能在一个更新域 (UD) 的 VM 上触发维护。

 

[!INCLUDE [virtual-machines-common-maintenance-notifications](../../../includes/virtual-machines-common-maintenance-notifications.md)]

## <a name="check-maintenance-status-using-powershell"></a>使用 PowerShell 检查维护状态

还可以使用 Azure Powershell 查看 VM 计划何时维护。 使用 `-status` 参数时可通过 [Get-AzVM](https://docs.microsoft.com/powershell/module/az.compute/get-azvm) cmdlet 获得计划内维护信息。
 
仅当有计划内维护时，才会返回维护信息。 如果未计划任何影响 VM 的维护，该 cmdlet 不返回任何维护信息。 

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

```powershell
Get-AzVM -ResourceGroupName rgName -Name vmName -Status
```

在 MaintenanceRedeployStatus 下返回以下属性： 

| 值 | 描述   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | 指示此时是否可以在 VM 上启动维护 |
| PreMaintenanceWindowStartTime         | 可以在 VM 上启动维护的自助式维护时段的起点 |
| PreMaintenanceWindowEndTime           | 可以在 VM 上启动维护的自助式维护时段的终点 |
| MaintenanceWindowStartTime            | Azure 在 VM 上启动维护的计划内维护时段的起点 |
| MaintenanceWindowEndTime              | Azure 在 VM 上启动维护的计划内维护时段的终点 |
| LastOperationResultCode               | 上次尝试在 VM 上启动维护的结果 |



还可以通过使用 [Get-AzVM](https://docs.microsoft.com/powershell/module/az.compute/get-azvm) 并且不指定 VM 来获取资源组中所有 VM 的维护状态。
 
```powershell
Get-AzVM -ResourceGroupName rgName -Status
```

以下 PowerShell 函数获取订阅 ID，并输出计划维护的 VM 列表。

```powershell

function MaintenanceIterator
{
    Select-AzSubscription -SubscriptionId $args[0]

    $rgList= Get-AzResourceGroup 

    for ($rgIdx=0; $rgIdx -lt $rgList.Length ; $rgIdx++)
    {
        $rg = $rgList[$rgIdx]        
    $vmList = Get-AzVM -ResourceGroupName $rg.ResourceGroupName 
        for ($vmIdx=0; $vmIdx -lt $vmList.Length ; $vmIdx++)
        {
            $vm = $vmList[$vmIdx]
            $vmDetails = Get-AzVM -ResourceGroupName $rg.ResourceGroupName -Name $vm.Name -Status
              if ($vmDetails.MaintenanceRedeployStatus )
            {
                Write-Output "VM: $($vmDetails.Name)  IsCustomerInitiatedMaintenanceAllowed: $($vmDetails.MaintenanceRedeployStatus.IsCustomerInitiatedMaintenanceAllowed) $($vmDetails.MaintenanceRedeployStatus.LastOperationMessage)"               
            }
          }
    }
}

```

### <a name="start-maintenance-on-your-vm-using-powershell"></a>使用 PowerShell 在 VM 上启动维护

如果 **IsCustomerInitiatedMaintenanceAllowed** 设置为 true，以下命令使用上一部分中函数的信息，在 VM 上启动维护。

```powershell
Restart-AzVM -PerformMaintenance -name $vm.Name -ResourceGroupName $rg.ResourceGroupName 
```

## <a name="classic-deployments"></a>经典部署

如果你仍在使用由经典部署模型部署的旧 VM，则可以使用 PowerShell 查询 VM，并启动维护。

若要获取 VM 的维护状态，请键入：

```
Get-AzureVM -ServiceName <Service name> -Name <VM name>
```

若要在经典 VM 上启动维护，请键入：

```
Restart-AzureVM -InitiateMaintenance -ServiceName <service name> -Name <VM name>
```


## <a name="faq"></a>常见问题解答


**问：为什么需要立即重新启动虚拟机？**

**答:** 虽然对 Azure 平台的大多数更新和升级不会影响虚拟机的可用性，但在某些情况下无法避免重新启动 Azure 中托管的虚拟机。 我们累积了多个需要重启服务器的更改，这将导致重新启动虚拟机。

**问：如果我按建议使用可用性集实现高可用性，我是否安全？**

**答:** 对于部署在可用性集或虚拟机规模集中的虚拟机，我们有一个概念：更新域 (UD)。 执行维护时，Azure 遵循 UD 约束，不会从不同 UD（在同一可用性集中）重新启动虚拟机。  Azure 还会至少等待 30 分钟，然后才移到下一组虚拟机。 

有关高可用性的详细信息，请参阅 [Azure 中虚拟机的区域和可用性](regions-and-availability.MD)。

**问：如何收到有关计划内维护的通知？**

**答:** 一次计划内维护是通过将计划设置到一个或多个 Azure 区域启动的。 不久以后，电子邮件通知将发送到订阅所有者（每个订阅一封电子邮件）。 可以使用活动日志警报配置此通知的其他通道和收件人。 如果将虚拟机部署到已安排计划内维护的区域，将不会收到通知，而是需要检查 VM 的维护状态。

**问：我在门户、PowerShell 或 CLI 中看不到计划内维护的任何指示。是哪里出错了？**

**答:** 在计划内维护期间，与计划内维护相关的信息仅适用于将受到该次计划内维护影响的 VM。 换而言之，如果你看不到数据，则可能是这次维护已完成（或未启动）或虚拟机已在更新的服务器中托管。

**问：有什么方法可以知道虚拟机受影响的确切时间？**

**答:** 设置计划时，我们定义了一个长达几天的时间段。 但是，服务器（和 VM）在此时间窗口内的确切排序是未知的。 想要知道其 VM 确切时间的客户可以使用[计划事件](scheduled-events.md)并从虚拟机中查询，这样就会在 VM 重启前 15 分钟收到通知。

**问：重新启动虚拟机需要多长时间？**

**答:** 根据 VM 的大小，在自助维护时段内，重新启动最多可能需要几分钟时间。 当 Azure 在计划性维护时段内启动重启时，重启通常需要 25 分钟左右。 请注意，如果使用云服务（Web/辅助角色）、虚拟机规模集或可用性集，则在计划性维护时段内每组 VM (UD) 之间有 30 分钟的可用时间。 

**问：使用虚拟机规模集时的体验如何？**

**答:** 虚拟机规模集现在可使用计划内维护。 有关如何启动自助维护的说明，请参阅 [VMSS 的计划内维护](../../virtual-machine-scale-sets/virtual-machine-scale-sets-maintenance-notifications.md)文档。

**问：使用云服务（Web/辅助角色）和 Service Fabric 时的体验如何？**

**答:** 虽然这些平台会受到计划内维护的影响，但使用这些平台的客户可以安全地进行操作，因为在任何给定的时间，只有单个升级域 (UD) 中的 VM 受影响。 自助维护目前不适用于云服务（Web/辅助角色）和 Service Fabric。

**问：我在 VM 上看不到任何维护信息，是哪里出错了？**

**答:** 有很多原因会导致在 VM 上看不到任何维护信息：
1.  使用的是标记为“Microsoft 内部”的订阅。
2.  VM 未计划进行维护。 可能是这次维护已结束、已取消或已修改，因此你的 VM 不再受其影响。
3.  未将“维护”列添加到 VM 列表视图。 虽然我们已向默认视图添加此列，但配置为查看非默认列的客户必须手动将“维护”列添加到其 VM 列表视图。

**问：我的 VM 已计划进行第二次维护，为什么？**

**答:** 多种用例都会看到在完成维护性重新部署后，VM 仍进行计划性维护：
1.  我们已取消这次维护，并使用不同的有效负载重新启动它。 可能是我们已检测到出错的有效负载，只需部署其他有效负载。
2.  由于硬件故障，已在另一个节点上对 VM 进行服务修复
3.  选择了停止（解除分配）VM 并将其重启
4.  已经为 VM 启用了**自动关闭**


## <a name="next-steps"></a>后续步骤

了解如何使用[计划事件](scheduled-events.md)从 VM 内注册维护事件。
