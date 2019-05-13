---
title: 备份和还原使用 Azure 备份和 PowerShell 的 Azure 文件
description: 备份和还原使用 Azure 备份和 PowerShell 的 Azure 文件。
author: pvrk
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 03/05/2018
ms.author: pullabhk
ms.openlocfilehash: 986414d0bac24d0c7e37b34df473346742fa97fd
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204182"
---
# <a name="back-up-and-restore-azure-files-with-powershell"></a>备份和还原 Azure 文件和 PowerShell

本文介绍如何使用 Azure PowerShell 来备份和恢复使用 Azure 文件文件共享[Azure 备份](backup-overview.md)恢复服务保管库。 

本教程介绍了如何完成以下操作：

> [!div class="checklist"]
> * 设置 PowerShell，并将注册 Azure 恢复服务提供程序。
> * 创建恢复服务保管库。
> * 配置 Azure 文件共享的备份。
> * 运行备份作业。
> * 还原已备份的 Azure 文件共享或从共享单个文件。
> * 监视备份和还原作业。


## <a name="before-you-start"></a>开始之前

- [详细了解](backup-azure-recovery-services-vault-overview.md)恢复服务保管库。
- 阅读有关预览功能的信息[Azure 文件共享备份](backup-azure-files.md)。
- 查看恢复服务的 PowerShell 对象层次结构。


## <a name="recovery-services-object-hierarchy"></a>恢复服务对象层次结构

下图汇总了对象层次结构。

![恢复服务对象层次结构](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

查看 Azure 库中的 **Az.RecoveryServices** [cmdlet 参考](/powershell/module/az.recoveryservices)。


## <a name="set-up-and-install"></a>设置和安装

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

设置 PowerShell，如下所示：

1. [下载最新版本的 Azure PowerShell](/powershell/azure/install-az-ps)。 所需的最低版本为 1.0.0。

2. 查找与此命令的 Azure 备份 PowerShell cmdlet:

    ```powershell
    Get-Command *azrecoveryservices*
    ```
3. 查看别名和 cmdlet 的 Azure 备份，Azure Site Recovery 和恢复服务保管库会显示。 下面是你可能会看到的示例。 它不是 cmdlet 的完整列表。

    ![恢复服务 cmdlet 列表](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

3. 登录到你的 Azure 帐户**Connect AzAccount**。
4. 在显示 web 页上，系统会提示输入你的帐户凭据。

    - 或者，可以将帐户凭据包含作为参数传入**Connect AzAccount** cmdlet 并结合 **-凭据**。
    - 如果你是代表租户的 CSP 合作伙伴，请将客户指定为使用其 tenantID 或租户主域名对租户。 例如，Connect-AzAccount -Tenant fabrikam.com。

4. 你想要使用的帐户，因为一个帐户可以有多个订阅的订阅关联。

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

5. 首次使用 Azure 备份时，请使用 Register-AzResourceProvider cmdlet 将 Azure 恢复服务提供程序注册到订阅。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. 验证已成功注册提供程序：

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```
7. 在命令输出中，确认**RegistrationState**更改为**已注册**。 如果没有打开，请运行**寄存器 AzResourceProvider**再次 cmdlet。



## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

请按照以下步骤创建恢复服务保管库。

- 恢复服务保管库是一种资源管理器资源，因此必须将其放在资源组中。 可以使用现有资源组，也可以使用 New-AzResourceGroup cmdlet 创建资源组。 创建资源组时，请指定资源组的名称和位置。 

1. 保管库位于资源组中。 如果您不具备现有的资源组中，创建一个具有新[新建 AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=azps-1.4.0)。 在此示例中，我们将创建新的资源组在美国西部区域中。

   ```powershell
   New-AzResourceGroup -Name "test-rg" -Location "West US"
   ```
2. 使用[新建 AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/New-AzRecoveryServicesVault?view=azps-1.4.0) cmdlet 来创建保管库。 请为保管库指定与资源组相同的位置。

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```
3. 指定要用于保管库存储冗余类型。

   - 可以使用[本地冗余存储](../storage/common/storage-redundancy-lrs.md)或[异地冗余存储](../storage/common/storage-redundancy-grs.md)。
   - 下面的示例设置 **-BackupStorageRedundancy**选项[集 AzRecoveryServicesBackupProperties](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty)为 cmd **testvault**设置为**异地冗余**。

     ```powershell
     $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
     Set-AzRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
     ```

### <a name="view-the-vaults-in-a-subscription"></a>在订阅中查看保管库

若要查看订阅中的所有保管库，请使用 [Get-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesvault?view=azps-1.4.0)。

```powershell
Get-AzRecoveryServicesVault
```

类似于以下输出。 请注意，提供关联的资源组和位置。

```powershell
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>设置保管库上下文

将保管库对象存储在变量中，并设置保管库上下文。

- 许多 Azure 备份 cmdlet 需要恢复服务保管库对象作为输入，，因此可以方便地存储在变量中的保管库对象。
- 保管库上下文是在保管库中受保护的数据的类型。 使用设置该[集 AzRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext?view=azps-1.4.0)。 设置的上下文后，它适用于所有后续 cmdlet。


以下示例为 testvault 设置保管库上下文。

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>提取的保管库 ID

我们计划弃用按照 Azure PowerShell 指南设置保管库上下文。 相反，可以存储或提取保管库 ID，并将其传递给相关命令，如下所示：

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

## <a name="configure-a-backup-policy"></a>配置备份策略

备份策略指定备份计划，并应保留多长时间的备份恢复点：

- 一个备份策略至少与一个保留策略相关联。 保留策略定义了在将恢复点删除之前将其保留的时限。
- 默认备份策略保持期使用视图[Get AzRecoveryServicesBackupRetentionPolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject?view=azps-1.4.0)。
- 默认备份策略计划使用的视图[Get AzRecoveryServicesBackupSchedulePolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject?view=azps-1.4.0)。
-  您使用[新建 AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy?view=azps-1.4.0) cmdlet 创建新的备份策略。 输入计划和保留策略对象。

以下示例将计划策略和保留策略存储在变量中。 然后，它使用这些变量作为参数的新策略 (**NewAFSPolicy**)。 “NewAFSPolicy”进行每日备份，并将其保留 30 天。

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureFiles"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

类似于以下输出。

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewAFSPolicy           AzureFiles            AzureStorage              10/24/2017 1:30:00 AM
```



## <a name="enable-backup"></a>启用备份

定义备份策略后，你可以使用策略的 Azure 文件共享的保护。

### <a name="retrieve-a-backup-policy"></a>检索备份策略

提取具有相关的策略对象[Get AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy?view=azps-1.4.0)。 若要获取特定策略，或查看与工作负荷类型关联的策略，请使用此 cmdlet。

#### <a name="retrieve-a-policy-for-a-workload-type"></a>检索用于工作负荷类型的策略

下面的示例检索策略的工作负荷类型**AzureFiles**。

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureFiles"
```

类似于以下输出。

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
dailyafs             AzureFiles         AzureStorage         1/10/2018 12:30:00 AM
```
> [!NOTE]
> PowerShell 中“BackupTime”字段的时区是通用协调时间 (UTC)。 在 Azure 门户中显示备份时间时，该时间会根据本地时区进行调整。

### <a name="retrieve-a-specific-policy"></a>检索特定的策略

以下策略检索名为“dailyafs”的备份策略。

```powershell
$afsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "dailyafs"
```

### <a name="enable-backup-and-apply-policy"></a>启用备份并将策略应用

使用启用保护[启用 AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection?view=azps-1.4.0)。 策略将与保管库相关联后，会根据策略计划触发备份。

下面的示例启用对 Azure 文件共享的保护**testAzureFileShare**存储帐户中**testStorageAcct**，与策略**dailyafs**。

```powershell
Enable-AzRecoveryServicesBackupProtection -StorageAccountName "testStorageAcct" -Name "testAzureFS" -Policy $afsPol
```

此命令将等到保护作业配置完成为止，然后提供如下所示的输出。

```cmd
WorkloadName       Operation            Status                 StartTime                                                                                                         EndTime                   JobID
------------             ---------            ------               ---------                                  -------                   -----
testAzureFS       ConfigureBackup      Completed            11/12/2018 2:15:26 PM     11/12/2018 2:16:11 PM     ec7d4f1d-40bd-46a4-9edb-3193c41f6bf6
```

## <a name="trigger-an-on-demand-backup"></a>触发按需备份

使用[备份 AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem?view=azps-1.4.0)运行按需备份受保护的 Azure 文件共享。

1. 存储帐户和文件从共享容器保存备份数据与保管库中检索[Get AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer)。
2. 若要启动备份作业，请使用 [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem) 获取有关 VM 的信息。
3. 使用按需备份的频率[备份 AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem)。

运行按需备份，如下所示：
    
```powershell
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType "AzureFiles" -Name "testAzureFS"
$job =  Backup-AzRecoveryServicesBackupItem -Item $afsBkpItem
```

该命令返回一个要跟踪的、带有 ID 的作业，如以下示例所示。

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS       Backup               Completed            11/12/2018 2:42:07 PM     11/12/2018 2:42:11 PM     8bdfe3ab-9bf7-4be6-83d6-37ff1ca13ab6
```

创建备份时使用 Azure 文件共享快照，因此通常作业在命令返回此输出时完成。

### <a name="modify-the-protection-policy"></a>修改保护策略

若要更改用于备份 Azure 文件共享所使用的策略，请使用[启用 AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection?view=azps-1.4.0)。 指定相关备份项和新的备份策略。

以下示例将“testAzureFS”保护策略从“dailyafs”更改为“monthlyafs”。

```powershell
$monthlyafsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "monthlyafs"
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType AzureFiles -Name "testAzureFS"
Enable-AzRecoveryServicesBackupProtection -Item $afsBkpItem -Policy $monthlyafsPol
```

## <a name="restore-azure-file-shares-and-files"></a>还原 Azure 文件共享和文件

你可以还原整个文件共享上的特定文件。 你可以还原到原始位置或备用位置。 

### <a name="fetch-recovery-points"></a>提取恢复点

使用[Get AzRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint?view=azps-1.4.0)若要列出备份项的所有恢复点。

在下面的脚本：

- 在变量 **$rp**是在过去七天从所选备份项的恢复点的数组。
- 该数组按时间进行反向排序，以最新的恢复点作为索引 0。
- 使用标准 PowerShell 数组索引选取恢复点。
- 在示例中，$rp[0] 选择最近的恢复点。

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $afsBkpItem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()

$rp[0] | fl
```

类似于以下输出。

```powershell
FileShareSnapshotUri : https://testStorageAcct.file.core.windows.net/testAzureFS?sharesnapshot=2018-11-20T00:31:04.00000
                       00Z
RecoveryPointType    : FileSystemConsistent
RecoveryPointTime    : 11/20/2018 12:31:05 AM
RecoveryPointId      : 86593702401459
ItemName             : testAzureFS
Id                   : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Micros                      oft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;teststorageRG;testStorageAcct/protectedItems/AzureFileShare;testAzureFS/recoveryPoints/86593702401462
WorkloadType         : AzureFiles
ContainerName        : storage;teststorageRG;testStorageAcct
ContainerType        : AzureStorage
BackupManagementType : AzureStorage
```
选择了相关的恢复点后，你还原的文件共享或文件到原始位置或备用位置。

### <a name="restore-an-azure-file-share-to-an-alternate-location"></a>将 Azure 文件共享还原到备用位置

使用[还原 AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem?view=azps-1.4.0)还原到所选的恢复点。 指定这些参数来标识备用位置： 

- **TargetStorageAccountName**：要将备份内容还原到的存储帐户。 目标存储帐户应与保管库位于同一位置。
- **TargetFileShareName**：目标存储帐户中要将备份内容还原到的文件共享。
- **TargetFolder**：文件共享中要将数据还原到的文件夹。 如果要将备份内容还原到根文件夹，请将目标文件夹值指定为空字符串。
- **ResolveConflict**：与还原的数据发生冲突时提供的说明。 接受“覆盖”或“跳过”。

使用参数运行该 cmdlet，如下所示：

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -ResolveConflict Overwrite
```

该命令返回一个要跟踪的、带有 ID 的作业，如以下示例所示。

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS        Restore              InProgress           12/10/2018 9:56:38 AM                               9fd34525-6c46-496e-980a-3740ccb2ad75
```

### <a name="restore-an-azure-file-to-an-alternate-location"></a>将 Azure 文件还原到备用位置

使用[还原 AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem?view=azps-1.4.0)还原到所选的恢复点。 指定这些参数来确定另一个位置，以及唯一标识你想要还原的文件。

* **TargetStorageAccountName**：要将备份内容还原到的存储帐户。 目标存储帐户应与保管库位于同一位置。
* **TargetFileShareName**：目标存储帐户中要将备份内容还原到的文件共享。
* **TargetFolder**：文件共享中要将数据还原到的文件夹。 如果要将备份内容还原到根文件夹，请将目标文件夹值指定为空字符串。
* **SourceFilePath**：文件共享中要还原的文件的绝对路径，字符串格式。 此路径与 Get-AzStorageFile PowerShell cmdlet 中使用的路径相同。
* **SourceFileType**：选择的是目录还是文件。 接受“目录”或“文件”。
* **ResolveConflict**：与还原的数据发生冲突时提供的说明。 接受“覆盖”或“跳过”。

其他参数 （SourceFilePath 和 SourceFileType） 被与你想要还原单个文件。

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

此命令将返回具有要跟踪的 ID 的作业上一节中所示。

### <a name="restore-azure-file-shares-and-files-to-the-original-location"></a>将 Azure 文件共享和文件还原到原始位置

在还原到原始位置时，不需要指定目标和目标相关的参数。 仅“ResolveConflict”必须提供。

#### <a name="overwrite-an-azure-file-share"></a>覆盖 Azure 文件共享

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -ResolveConflict Overwrite
```

#### <a name="overwrite-an-azure-file"></a>覆盖 Azure 文件

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

## <a name="track-backup-and-restore-jobs"></a>跟踪备份和还原作业

按需备份和还原操作返回时显示的作业 ID，以及您[运行按需备份](#trigger-an-on-demand-backup)。 使用[Get AzRecoveryServicesBackupJobDetails](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob?view=azps-1.4.0) cmdlet 来跟踪作业进度和详细信息。

```powershell
$job = Get-AzRecoveryServicesBackupJob -JobId 00000000-6c46-496e-980a-3740ccb2ad75 -VaultId $vaultID

 $job | fl


IsCancellable        : False
IsRetriable          : False
ErrorDetails         : {Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureFileShareJobErrorInfo}
ActivityId           : 00000000-5b71-4d73-9465-8a4a91f13a36
JobId                : 00000000-6c46-496e-980a-3740ccb2ad75
Operation            : Restore
Status               : Failed
WorkloadName         : testAFS
StartTime            : 12/10/2018 9:56:38 AM
EndTime              : 12/10/2018 11:03:03 AM
Duration             : 01:06:24.4660027
BackupManagementType : AzureStorage

$job.ErrorDetails

 ErrorCode ErrorMessage                                          Recommendations
 --------- ------------                                          ---------------
1073871825 Microsoft Azure Backup encountered an internal error. Wait for a few minutes and then try the operation again. If the issue persists, please contact Microsoft support.
```
## <a name="next-steps"></a>后续步骤
[了解有关](backup-azure-files.md)在 Azure 门户中备份 Azure 文件。
