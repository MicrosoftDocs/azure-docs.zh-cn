---
title: 使用 Azure 备份与 PowerShell 来备份和恢复 Azure VM
description: 介绍如何使用 Azure 备份与 PowerShell 来备份和恢复 Azure VM
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: raynew
ms.openlocfilehash: 10af40a1f671d5871204ff465395c8c3619671f7
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232489"
---
# <a name="back-up-and-restore-azure-vms-with-powershell"></a>使用 PowerShell 备份和恢复 Azure VM

本文介绍如何使用 PowerShell cmdlet 在 [Azure 备份](backup-overview.md)恢复服务保管库中备份和还原 Azure VM。

本文介绍如何执行以下操作：

> [!div class="checklist"]
> * 创建恢复服务保管库并设置保管库上下文。
> * 定义备份策略
> * 应用备份策略以保护多个虚拟机
> * 对保护的虚拟机触发按需备份作业 在备份（或保护）虚拟机之前，必须完成[先决条件](backup-azure-arm-vms-prepare.md)中的步骤来准备好保护 VM 的环境。

## <a name="before-you-start"></a>开始之前

- [详细了解](backup-azure-recovery-services-vault-overview.md)恢复服务保管库。
- [查看](backup-architecture.md#architecture-direct-backup-of-azure-vms)Azure VM 备份的体系结构[了解如何](backup-azure-vms-introduction.md)备份过程中，和[查看](backup-support-matrix-iaas.md)支持、 限制和先决条件。
- 查看恢复服务的 PowerShell 对象层次结构。

## <a name="recovery-services-object-hierarchy"></a>恢复服务对象层次结构

下图汇总了对象层次结构。

![恢复服务对象层次结构](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

查看 Azure 库中的 **Az.RecoveryServices** [cmdlet 参考](https://docs.microsoft.com/powershell/module/Az.RecoveryServices/?view=azps-1.4.0)。

## <a name="set-up-and-register"></a>设置和注册

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

开始时，请执行以下操作：

1. [下载最新版本的 PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)

2. 键入以下命令查找可用的 Azure 备份 PowerShell cmdlet：

    ```powershell
    Get-Command *azrecoveryservices*
    ```

    将会显示 Azure 备份、Azure Site Recovery 和恢复服务保管库的别名和 cmdlet。 下图是你将看到的示例。 这不是 cmdlet 的完整列表。

    ![恢复服务列表](./media/backup-azure-vms-automation/list-of-recoveryservices-ps.png)

3. 使用 **Connect-AzAccount** 登录到 Azure 帐户。 此 cmdlet 打开一个网页，提示输入帐户凭据：

    * 或者，可以使用 **-Credential** 参数，在 **Connect-AzAccount** cmdlet 中将帐户凭据包含为参数。
    * 如果是代表租户的 CSP 合作伙伴，则需使用 tenantID 或租户主域名将客户指定为一名租户。 例如:**Connect-AzAccount -Tenant "fabrikam.com"**

4. 由于一个帐户可以有多个订阅，因此请将要使用的订阅与帐户关联在一起：

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

5. 首次使用 Azure 备份时，必须使用 **[Register-AzResourceProvider](https://docs.microsoft.com/powershell/module/az.resources/register-azresourceprovider)** cmdlet 将 Azure 恢复服务提供程序注册到订阅。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. 可使用以下命令验证提供程序是否已成功注册：

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

    在命令输出中，RegistrationState 应更改为 Registered。 如果未按此更改，只需再次运行 **[Register-AzResourceProvider](https://docs.microsoft.com/powershell/module/az.resources/register-azresourceprovider)** cmdlet 即可。


## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

以下步骤引导创建恢复服务保管库。 恢复服务保管库不同于备份保管库。

1. 恢复服务保管库是一种 Resource Manager 资源，因此需要将它放在资源组中。 可以使用现有的资源组，也可以使用 **[New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)** cmdlet 创建资源组。 创建资源组时，请指定资源组的名称和位置。  

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```
2. 使用 [New-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/new-azrecoveryservicesvault?view=azps-1.4.0) cmdlet 创建恢复服务保管库。 确保为保管库指定的位置与用于资源组的位置是相同的。

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```
3. 指定要使用的存储冗余类型；可以使用[本地冗余存储 (LRS)](../storage/common/storage-redundancy-lrs.md) 或[异地冗余存储 (GRS)](../storage/common/storage-redundancy-grs.md)。 以下示例显示，testvault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperty  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

   > [!TIP]
   > 许多 Azure 备份 cmdlet 要求使用恢复服务保管库对象作为输入。 出于此原因，在变量中存储备份恢复服务保管库对象可提供方便。
   >
   >

## <a name="view-the-vaults-in-a-subscription"></a>在订阅中查看保管库

若要查看订阅中的所有保管库，请使用 [Get-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesvault?view=azps-1.4.0)：

```powershell
Get-AzRecoveryServicesVault
```

输出与以下示例类似，请注意提供的关联 ResourceGroupName 和 Location（位置）。

```
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```


## <a name="back-up-azure-vms"></a>备份 Azure VM

使用恢复服务保管库保护虚拟机。 应用保护前，请先设置保管库上下文（保管库中受保护的数据类型）并验证保护策略。 保护策略是指备份作业运行时以及每个备份快照的保留时长的计划。

### <a name="set-vault-context"></a>设置保管库上下文

在 VM 上启用保护之前，请使用 [Set-AzRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext?view=azps-1.4.0) 来设置保管库上下文。 设置保管库上下文后，它将应用于所有后续 cmdlet。 以下示例为保管库 *testvault* 设置保管库上下文。

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="modifying-storage-replication-settings"></a>修改存储复制设置

使用[集 AzRecoveryServicesBackupProperty](https://docs.microsoft.com/powershell/module/az.recoveryservices/Set-AzRecoveryServicesBackupProperty)命令设置保管库的存储复制配置为 LRS/GRS

```powershell
$vault= Get-AzRecoveryServicesVault -name "testvault"
Set-AzRecoveryServicesBackupProperty -Vault $vault -BackupStorageRedundancy GeoRedundant/LocallyRedundant
```

> [!NOTE]
> 只有在没有受保管库保护的备份项的情况下，才能修改存储冗余性。

### <a name="create-a-protection-policy"></a>创建保护策略

在创建恢复服务保管库时，它附带了默认的保护和保留策略。 默认保护策略在每天的指定时间触发备份作业。 默认保留策略将每日恢复点保留 30 天。 可以使用默认策略快速保护 VM，以后再使用不同的详细信息编辑该策略。

若要查看保管库中可用的保护策略，请使用 **[Get-AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy)。 可以使用此 cmdlet 获取特定策略，或者查看与某个工作负荷类型关联的策略。 以下示例获取适用于工作负荷类型 AzureVM 的策略。

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM"
```

输出类似于以下示例：

```
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM
```

> [!NOTE]
> PowerShell 中 BackupTime 字段的时区是 UTC。 但是，在 Azure 门户中显示备份时间时，该时间会根据本地时区调整。
>
>

一个备份保护策略至少与一个保留策略相关联。 保留策略定义了在将恢复点删除之前将其保留多长时间。

- 使用 [Get-AzRecoveryServicesBackupRetentionPolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject) 查看默认保留策略。
- 类似地，可以使用 [Get-AzRecoveryServicesBackupSchedulePolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject) 获取默认计划策略。
- [New-AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy) cmdlet 创建用于保存备份策略信息的 PowerShell 对象。
- 计划和保留策略对象将用作 New-AzRecoveryServicesBackupProtectionPolicy cmdlet 的输入。

默认情况下，会在“计划策略对象”中定义开始时间。 请使用以下示例将开始时间更改为所需的开始时间。 所需的开始时间也应采用 UTC 格式。 以下示例假设在进行每日备份时，所需的开始时间为 UTC 时间凌晨 1:00。

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

以下示例将计划策略和保留策略存储在变量中。 此示例使用这些变量来定义在创建保护策略 *NewPolicy* 时要使用的参数。

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType "AzureVM" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

输出类似于以下示例：

```
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM
```

### <a name="enable-protection"></a>启用保护

定义保护策略后，仍然必须为项目启用策略。 请使用 [Enable-AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) 来启用保护。 启用保护需要两个对象 - 项和策略。 将策略与保管库关联之后，会在策略计划中定义的时间触发备份工作流。

以下示例使用策略 NewPolicy 启用对项目 V2VM 的保护。 示例会基于 VM 是否加密以及加密类型而有所不同。

在非加密的资源管理器 VM 上启用保护：

```powershell
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

若要在加密 VM（使用 BEK 和 KEK 加密）上启用保护，必须向 Azure 备份服务授予权限来读取密钥保管库中的密钥和机密。

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

要在加密的 VM（仅使用 BEK 加密）上启用保护，必须为 Azure 备份服务授予从密钥保管库读取密码的权限。

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToSecrets backup,get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

> [!NOTE]
> 如果要使用 Azure 政府版云，则使用值 ff281ffe-705c-4f53-9f37-a40e6f2c68f3 参数 ServicePrincipalName 中[集 AzKeyVaultAccessPolicy](https://docs.microsoft.com/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) cmdlet。
>

## <a name="monitoring-a-backup-job"></a>监视备份作业

可以在不使用 Azure 门户的情况下监视长时间运行的操作，例如备份作业。 若要获取正在进行的作业的状态，请使用 [Get-AzRecoveryservicesBackupJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) cmdlet。 此 cmdlet 获取特定保管库的备份作业，并且该保管库是在保管库上下文中指定的。 以下示例将正在进行的作业的状态获取为数组，并将状态存储在 $joblist 变量中。

```powershell
$joblist = Get-AzRecoveryservicesBackupJob –Status "InProgress"
$joblist[0]
```

输出类似于以下示例：

```
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM             Backup               InProgress            4/23/2016                5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

与其使用额外的不必要的代码来轮询这些作业的完成情况，不如使用 [Wait-AzRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) cmdlet。 该 cmdlet 暂停操作的执行，直到作业完成或达到了指定的超时值。

```powershell
Wait-AzRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200
```

## <a name="manage-azure-vm-backups"></a>管理 Azure VM 备份

### <a name="modify-a-protection-policy"></a>修改保护策略

若要修改保护策略，请使用 [Set-AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) 修改 SchedulePolicy 或 RetentionPolicy 对象。

#### <a name="modifying-scheduled-time"></a>修改计划的时间

创建保护策略时，它是默认情况下分配开始时间。 下面的示例演示如何修改保护策略的开始时间。

````powershell
$SchPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z" (This is the time that the customer wants to start the backup)
$UtcTime = $UtcTime.ToUniversalTime()
$SchPol.ScheduleRunTimes[0] = $UtcTime
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -SchedulePolicy $SchPol
````

#### <a name="modifying-retention"></a>修改保留期

以下示例将恢复点保留期更改为 365 天。

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
$retPol.DailySchedule.DurationCountInDays = 365
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy $RetPol
```

#### <a name="configuring-instant-restore-snapshot-retention"></a>配置即时还原快照保留期

> [!NOTE]
> 从 Az PS 版本 1.6.0 开始，用户可以使用 Powershell 在策略中更新即时还原快照保留期

````powershell
PS C:\> $bkpPol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM"
$bkpPol.SnapshotRetentionInDays=7
PS C:\> Set-AzureRmRecoveryServicesBackupProtectionPolicy -policy $bkpPol
````

默认值为 2，用户可以将值设置为 1 到 5。 每周备份策略的保留期设置为 5，不能更改。

### <a name="trigger-a-backup"></a>触发备份

请使用 [Backup-AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) 来触发备份作业。 如果是初始备份，则其为完整备份。 后续备份将创建增量副本。 下面的示例将 VM 备份将保留 60 天。

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM"
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
$job = Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

输出类似于以下示例：

```
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM              Backup              InProgress          4/23/2016                  5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

> [!NOTE]
> PowerShell 中 StartTime 和 EndTime 字段的时区是 UTC。 但是，在 Azure 门户中显示时间时，该时间会根据本地时区调整。
>
>

### <a name="change-policy-for-backup-items"></a>更改备份项策略

用户可以修改现有策略或更改备份项的策略从 Policy1 到策略 2。 若要切换的备份项策略，只需提取相关的策略和备份项并使用[启用 AzRecoveryServices](https://docs.microsoft.com/powershell/module/az.recoveryservices/Enable-AzRecoveryServicesBackupProtection?view=azps-1.5.0)命令作为参数的备份项。

````powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName>
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType AzureVM -BackupManagementType AzureVM -Name "<BackupItemName>"
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1
````

此命令等待，直到配置备份已完成并返回以下输出。

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
TestVM           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="stop-protection"></a>停止保护

#### <a name="retain-data"></a>保留数据

如果用户想要停止保护，他们可以使用[禁用 AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/Disable-AzRecoveryServicesBackupProtection?view=azps-1.5.0) PS cmdlet。 这会停止计划的备份，但才备份数据现在永久保留。

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
````

#### <a name="delete-backup-data"></a>删除备份数据

若要在保管库中完全删除备份数据存储，只需添加-RemoveRecoveryPoints' 标志/切换到['disable' 保护命令](#retain-data)。

````powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
````

## <a name="restore-an-azure-vm"></a>还原 Azure VM

使用 Azure 门户还原 VM 与使用 PowerShell 还原 VM 存在很大区别。 如果使用 PowerShell，则从恢复点创建磁盘和配置信息时，还原操作即完成。 还原操作不会创建虚拟机。 若要通过磁盘创建虚拟机，请参阅[通过存储磁盘创建 VM](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) 部分。 如果不想还原整个 VM，而是要从 Azure VM 备份还原或恢复几个文件，请参阅[文件恢复部分](backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup)。

> [!Tip]
> 还原操作不会创建虚拟机。
>
>

下图显示了从 RecoveryServicesVault 到 BackupRecoveryPoint 的对象层次结构。

![显示 BackupContainer 的恢复服务对象层次结构](./media/backup-azure-vms-arm-automation/backuprecoverypoint-only.png)

若要还原备份数据，请确定已备份项目以及保留了时间点数据的恢复点。 请使用 [Restore-AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) 将数据从保管库还原到帐户。

还原 Azure VM 的基本步骤是：

* 选择 VM。
* 选择恢复点。
* 还原磁盘。
* 通过存储磁盘创建 VM。

### <a name="select-the-vm"></a>选择 VM

若要获取用于标识正确备份项的 PowerShell 对象，请从保管库中的容器开始，按对象层次结构进行操作。 若要选择代表 VM 的容器，请使用 [Get-AzRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) cmdlet，然后通过管道将其传递给 [Get-AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) cmdlet。

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM"
```

### <a name="choose-a-recovery-point"></a>选择一个恢复点

使用 [Get-AzRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) cmdlet 列出备份项的所有恢复点。 然后选择要还原的恢复点。 如果不确定要使用的恢复点，最好选择列表中最新的 RecoveryPointType = AppConsistent 恢复点。

在以下脚本中，变量 **$rp** 是一个数组，其中包含所选备份项在过去七天的恢复点。 该数组按时间进行反向排序，以最新的恢复点作为索引 0。 使用标准 PowerShell 数组索引选取恢复点。 在示例中，$rp[0] 选择最新的恢复点。

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0]
```

输出类似于以下示例：

```powershell
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="restore-the-disks"></a>还原磁盘

请使用 [Restore-AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) cmdlet 将备份项的数据和配置还原到某个恢复点。 确定恢复点后，将其用作 -RecoveryPoint 参数的值。 在上述示例中，$rp[0] 是要使用的恢复点。 在下面的示例代码中，$rp[0] 是还原磁盘时要使用的恢复点。

还原磁盘和配置信息：

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG"
$restorejob
```

#### <a name="restore-managed-disks"></a>还原托管磁盘

> [!NOTE]
> 如果备份的 VM 具有托管磁盘，并且你想要将其还原为托管磁盘，我们从 Azure PowerShell RM 模块 v 6.7.0 引入了相关功能。 更高版本
>
>

提供了附加参数 TargetResourceGroupName 来指定托管磁盘要还原到的 RG。 

> [!NOTE]
> 强烈建议使用 **TargetResourceGroupName** 参数来还原托管磁盘，因为它可以显著提高性能。 此外，从 Azure Powershell Az 模块 1.0 开始，在使用托管磁盘进行还原时，此参数是必需的
>
>


```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks"
```

VMConfig.JSON 文件将还原到存储帐户，托管磁盘将还原到指定的目标 RG。

输出类似于以下示例：

```powershell
WorkloadName     Operation          Status               StartTime                 EndTime            JobID
------------     ---------          ------               ---------                 -------          ----------
V2VM              Restore           InProgress           4/23/2016 5:00:30 PM                        cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

请使用 [Wait-AzRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) cmdlet 等待还原作业完成。

```powershell
Wait-AzRecoveryServicesBackupJob -Job $restorejob -Timeout 43200
```

还原作业完成后，请使用 [Get-AzRecoveryServicesBackupJobDetails](https://docs.microsoft.com/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) cmdlet 获取还原操作的详细信息。 JobDetails 属性提供重建 VM 所需的信息。

```powershell
$restorejob = Get-AzRecoveryServicesBackupJob -Job $restorejob
$details = Get-AzRecoveryServicesBackupJobDetails -Job $restorejob
```

还原磁盘以后，转到下一部分来了解如何创建 VM。

## <a name="create-a-vm-from-restored-disks"></a>从还原的磁盘创建 VM

还原磁盘后，使用以下步骤通过磁盘创建和配置虚拟机。

> [!NOTE]
> 若要使用已还原的磁盘创建加密 VM，则 Azure 角色必须有权执行 **Microsoft.KeyVault/vaults/deploy/action** 操作。 如果角色不具有此权限，可通过此操作创建自定义角色。 有关详细信息，请参阅 [Custom Roles in Azure RBAC](../role-based-access-control/custom-roles.md)（Azure RBAC 中的自定义角色）。
>
>

> [!NOTE]
> 还原磁盘后，现在可以获得一个部署模板，你可以直接使用该模板创建新的 VM。 没有更多的不同的 PS cmdlet 来创建加密/非加密的托管/非托管 VM。

生成的作业详细信息提供可以查询和部署的模板 URI。

```powershell
   $properties = $details.properties
   $templateBlobURI = $properties["Template Blob Uri"]
```

只需部署模板来创建新的 VM，如[此处](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)所述。

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment ResourceGroupName ExampleResourceGroup -TemplateUri $templateBlobURI -storageAccountType Standard_GRS
```

下一节列出使用“VMConfig”文件创建 VM 所需的步骤。

> [!NOTE]
> 强烈建议使用前面详细介绍的部署模板来创建 VM。 本部分（1-6 点）将在不久后弃用。

1. 查询已还原磁盘属性以获取作业详细信息。

   ```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $configBlobName = $properties["Config Blob Name"]
   ```

2. 设置 Azure 存储上下文和还原 JSON 配置文件。

   ```powershell
   Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName "testvault"
   $destination_path = "C:\vmconfig.json"
   Get-AzStorageBlobContent -Container $containerName -Blob $configBlobName -Destination $destination_path
   $obj = ((Get-Content -Path $destination_path -Raw -Encoding Unicode)).TrimEnd([char]0x00) | ConvertFrom-Json
   ```

3. 使用 JSON 配置文件来创建 VM 配置。

   ```powershell
   $vm = New-AzVMConfig -VMSize $obj.'properties.hardwareProfile'.vmSize -VMName "testrestore"
   ```

4. 附加 OS 磁盘和数据磁盘。 此步骤提供多种托管和加密的 VM 配置的示例。 请使用适合 VM 配置的示例。

   * **非托管的非加密 VM** - 对于非托管的非加密 VM，请使用以下示例。

       ```powershell
       Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
       $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
       foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
       {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
       }
       ```

   * **具有 Azure AD 的非托管加密 VM（仅限 BEK）**- 对于具有 Azure AD 的非托管加密 VM（仅使用 BEK 加密），需要先将密码还原到密钥保管库，然后才能附加磁盘。 有关详细信息，请参阅[从 Azure 备份恢复点还原加密的虚拟机](backup-azure-restore-key-secret.md)。 以下示例展示了如何为加密的 VM 附加 OS 和数据磁盘。 设置 OS 磁盘时，请确保提及相关的 OS 类型。

      ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $dekUrl = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows/Linux
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
      foreach($dd in $obj.'properties.storageProfile'.dataDisks)
      {
       $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
      }
      ```

   * **具有 Azure AD 的非托管加密 VM（BEK 和 KEK）**- 对于具有 Azure AD 的非托管加密 VM（使用 BEK 和 KEK 加密），先将密钥和密码还原到密钥保管库，然后再附加磁盘。 有关详细信息，请参阅[从 Azure 备份恢复点还原加密的虚拟机](backup-azure-restore-key-secret.md)。 以下示例展示了如何为加密的 VM 附加 OS 和数据磁盘。

      ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $kekUrl = "https://ContosoKeyVault.vault.azure.net:443/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
      $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
      foreach($dd in $obj.'properties.storageProfile'.dataDisks)
     {
     $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
     }
      ```

   * **不具有 Azure AD 的非托管加密 VM（仅限 BEK）**- 对于不具有 Azure AD 的非托管加密 VM（仅使用 BEK 加密），如果源 keyVault/密码不可用，可使用[从 Azure 备份恢复点还原非加密虚拟机](backup-azure-restore-key-secret.md)中的过程，将密码还原到密钥保管库。 然后执行以下脚本在已还原的 OS Blob 上设置加密详细信息（此步骤不需要数据 Blob）。 可以从已还原的 keyVault 提取 $dekurl。<br>

   仅当源 keyVault/密码不可用时，才需要执行下面的脚本。

      ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""}}]}"
      $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
      $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
      $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
      $osBlob.ICloudBlob.SetMetadata()
      ```

    密码可用且在 OS Blob 上设置加密详细信息后，请使用下面给出的脚本附加磁盘。<br>

    如果源 keyVault/密码已经可用，则无需执行以上脚本。

      ```powershell
      Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
      foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
      {
      $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
      }
      ```

   * **不具有 Azure AD 的非托管加密 VM（BEK和 KEK）**- 对于不具有 Azure AD 的非托管加密 VM（使用 BEK 和 KEK 加密），如果源 keyVault/密钥/密码不可用，可使用[从 Azure 备份恢复点还原非加密虚拟机](backup-azure-restore-key-secret.md)中的过程，将密钥和密码还原到密钥保管库。 然后执行以下脚本在已还原的 OS Blob 上设置加密详细信息（此步骤不需要数据 Blob）。 可以从已还原的 keyVault 提取 $dekurl 和 $kekurl。

   仅当源 keyVault/密钥/密码不可用时，才需要执行下面的脚本。

    ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
      $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""},""keyEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""keyUrl"":""$kekUrl""}}]}"
      $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
      $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
      $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
      $osBlob.ICloudBlob.SetMetadata()
      ```
   密钥/密码可用且在 OS Blob 上设置加密详细信息后，请使用下面给出的脚本附加磁盘。

    如果源 keyVault/密钥/密码可用，则无需执行以上脚本。

    ```powershell
      Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
      foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
      {
      $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
      }
      ```

   * 托管的非加密 VM -对于托管的非加密 VM，将附加还原的托管磁盘。 有关详细信息，请参阅[使用 PowerShell 将数据磁盘附加到 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。

   * **具有 Azure AD 的托管加密 VM（仅限 BEK）**- 对于具有 Azure AD 的托管加密 VM（仅使用 BEK 加密），将附加还原的托管磁盘。 有关详细信息，请参阅[使用 PowerShell 将数据磁盘附加到 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。

   * **具有 Azure AD 的托管加密 VM（BEK 和 KEK）**- 对于具有 Azure AD 的托管加密 VM（使用 BEK 和 KEK 加密），将附加还原的托管磁盘。 有关详细信息，请参阅[使用 PowerShell 将数据磁盘附加到 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。

   * **不具有 Azure AD 的托管加密 VM（仅限 BEK）**- 对于不具有 Azure AD 的托管加密 VM（仅使用 BEK 加密），如果源 keyVault/密码不可用，可使用[从 Azure 备份恢复点还原非加密虚拟机](backup-azure-restore-key-secret.md)中的过程，将密码还原到密钥保管库。然后执行以下脚本在已还原的 OS 磁盘上设置加密详细信息（此步骤不需要数据磁盘）。 可以从已还原的 keyVault 提取 $dekurl。

     仅当源 keyVault/密码不可用时，才需要执行下面的脚本。  

     ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
      $diskupdateconfig = Set-AzDiskUpdateDiskEncryptionKey -DiskUpdate $diskupdateconfig -SecretUrl $dekUrl -SourceVaultId $keyVaultId  
      Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
      ```

     密码可用且在 OS 磁盘上设置加密详细信息后，若要附加还原的托管磁盘，请参阅[使用 PowerShell 将数据磁盘附加到 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。

   * **不具有 Azure AD 的托管加密 VM（BEK和 KEK）**- 对于不具有 Azure AD 的托管加密 VM（使用 BEK 和 KEK 加密），如果源 keyVault/密钥/密码不可用，可使用[从 Azure 备份恢复点还原非加密虚拟机](backup-azure-restore-key-secret.md)中的过程，将密钥和密码还原到密钥保管库。 然后执行以下脚本在已还原的 OS 磁盘上设置加密详细信息（此步骤不需要数据磁盘）。 可以从已还原的 keyVault 提取 $dekurl 和 $kekurl。

   仅当源 keyVault/密钥/密码不可用时，才需要执行下面的脚本。

   ```powershell
     $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
     $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
     $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
     $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
     $diskupdateconfig = Set-AzDiskUpdateDiskEncryptionKey -DiskUpdate $diskupdateconfig -SecretUrl $dekUrl -SourceVaultId $keyVaultId  
     $diskupdateconfig = Set-AzDiskUpdateKeyEncryptionKey -DiskUpdate $diskupdateconfig -KeyUrl $kekUrl -SourceVaultId $keyVaultId  
     Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    密钥/密码可用且在 OS 磁盘上设置加密详细信息后，若要附加还原的托管磁盘，请参阅[使用 PowerShell 将数据磁盘附加到 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。

5. 设置网络设置。

    ```powershell
    $nicName="p1234"
    $pip = New-AzPublicIpAddress -Name $nicName -ResourceGroupName "test" -Location "WestUS" -AllocationMethod Dynamic
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName "test" -Location "WestUS" -Name "testvNET" -AddressPrefix 10.0.0.0/16
    $virtualNetwork | Set-AzVirtualNetwork
    $vnet = Get-AzVirtualNetwork -Name "testvNET" -ResourceGroupName "test"
    $subnetindex=0
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName "test" -Location "WestUS" -SubnetId $vnet.Subnets[$subnetindex].Id -PublicIpAddressId $pip.Id
    $vm=Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
    ```

6. 创建虚拟机。

    ```powershell  
    New-AzVM -ResourceGroupName "test" -Location "WestUS" -VM $vm
    ```

7. 推送 ADE 扩展。

   * **对于具有 Azure AD 的 VM** - 使用以下命令手动对数据磁盘启用加密  

     **仅限 BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

     **BEK 和 KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId  -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

   * **对于不具有 Azure AD 的 VM** - 使用以下命令手动对数据磁盘启用加密。

     如果在命令执行期间出现请求 AADClientID，则需要更新 Azure PowerShell。

     **仅限 BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

      **BEK 和 KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

## <a name="restore-files-from-an-azure-vm-backup"></a>从 Azure VM 备份还原文件

除了可以还原磁盘以外，还可以从 Azure VM 备份还原单个文件。 还原文件功能提供对恢复点中所有文件的访问权限。 通过文件资源管理器管理文件，与普通文件的管理方法相同。

从 Azure VM 备份还原文件的基本步骤如下：

* 选择 VM
* 选择一个恢复点
* 装载恢复点的磁盘
* 复制所需的文件
* 卸载磁盘

### <a name="select-the-vm"></a>选择 VM

若要获取用于标识正确备份项的 PowerShell 对象，请从保管库中的容器开始，按对象层次结构进行操作。 若要选择代表 VM 的容器，请使用 [Get-AzRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) cmdlet，然后通过管道将其传递给 [Get-AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) cmdlet。

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM"
```

### <a name="choose-a-recovery-point"></a>选择一个恢复点

使用 [Get-AzRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) cmdlet 列出备份项的所有恢复点。 然后选择要还原的恢复点。 如果不确定要使用的恢复点，最好选择列表中最新的 RecoveryPointType = AppConsistent 恢复点。

在以下脚本中，变量 **$rp** 是一个数组，其中包含所选备份项在过去七天的恢复点。 该数组按时间进行反向排序，以最新的恢复点作为索引 0。 使用标准 PowerShell 数组索引选取恢复点。 在示例中，$rp[0] 选择最新的恢复点。

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0]
```

输出类似于以下示例：

```powershell
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="mount-the-disks-of-recovery-point"></a>装载恢复点的磁盘

请使用 [Get-AzRecoveryServicesBackupRPMountScript](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprpmountscript) cmdlet 获取用于装载恢复点的所有磁盘的脚本。

> [!NOTE]
> 这些磁盘作为 iSCSI 附加磁盘装载到运行此脚本的计算机中。 装载将立刻开始，并且不会产生任何费用。
>
>

```powershell
Get-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0]
```

输出类似于以下示例：

```powershell
OsType  Password        Filename
------  --------        --------
Windows e3632984e51f496 V2VM_wus2_8287309959960546283_451516692429_cbd6061f7fc543c489f1974d33659fed07a6e0c2e08740.exe
```

在要在它上面恢复文件的计算机上运行此脚本。 要执行该脚本，必须输入提供的密码。 附加磁盘后，使用 Windows 文件资源管理器浏览新的卷和文件。 有关详细信息，请参阅备份文章[从 Azure 虚拟机备份恢复文件](backup-azure-restore-files-from-vm.md)。

### <a name="unmount-the-disks"></a>卸载磁盘

复制所需的文件后，请使用 [Disable-AzRecoveryServicesBackupRPMountScript](https://docs.microsoft.com/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackuprpmountscript) 卸载磁盘。 请务必卸载磁盘，以便删除对恢复点文件的访问权限。

```powershell
Disable-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0]
```

## <a name="next-steps"></a>后续步骤

如果你更愿意使用 PowerShell 来处理 Azure 资源，请查看 PowerShell 文章：[为 Windows Server 部署和管理备份](backup-client-automation.md)。 如果管理 DPM 备份，请参阅[为 DPM 部署和管理备份](backup-dpm-automation.md)。
