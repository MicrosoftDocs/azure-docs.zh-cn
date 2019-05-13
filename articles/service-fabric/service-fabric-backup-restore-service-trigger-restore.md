---
title: 在 Azure Service Fabric 中还原备份 | Microsoft Docs
description: 使用 Service Fabric 中的定期备份和还原功能从应用程序数据的备份还原数据。
services: service-fabric
documentationcenter: .net
author: aagup
manager: chackdan
editor: aagup
ms.assetid: 802F55B6-6575-4AE1-8A8E-C9B03512FF88
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/30/2018
ms.author: aagup
ms.openlocfilehash: e4ada412547360f97e869d3312b65d869fa3df48
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413712"
---
# <a name="restoring-backup-in-azure-service-fabric"></a>在 Azure Service Fabric 中还原备份

在 Azure Service Fabric 中，可靠有状态服务和 Reliable Actors 可在完成请求和响应事务之后保持可变的权威状态。 有状态服务可能长时间关闭，或由于灾难而丢失信息。 如果发生这种情况，可能需要从最新的可接受备份还原该服务，使其能够保持正常运行。

例如，可将某个服务配置为备份其数据，以防止出现以下情况：

- **灾难恢复案例**：整个 Service Fabric 群集永久丢失。
- **数据丢失案例**：大部分服务分区副本永久丢失。
- **数据丢失案例**：服务被意外删除或损坏。 例如，管理员错误地删除了服务。
- **数据损坏案例**：服务中的 Bug 导致数据损坏。 例如，当某个服务代码升级程序将错误数据写入到可靠集合时，可能会发生数据损坏。 在此情况下，可能需要将代码和数据都还原到先前的状态。

## <a name="prerequisites"></a>必备组件

- 若要触发还原，必须为群集启用故障分析服务 (FAS)
- 备份还原服务 (BRS) 已创建备份。
- 只能在分区触发还原。
- 安装适用于配置调用 Microsoft.ServiceFabric.Powershell.Http 模块 [中预览]。

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

- 请确保群集已连接使用`Connect-SFCluster`命令之前发出任何使用 Microsoft.ServiceFabric.Powershell.Http 模块的配置请求。

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```


## <a name="triggered-restore"></a>已触发还原

可针对以下任何方案触发还原：

- 灾难恢复时进行数据还原。
- 在数据损坏/数据丢失时进行数据还原。

### <a name="data-restore-in-the-case-of-disaster-recovery"></a>发生灾难恢复时进行数据还原

如果整个 Service Fabric 群集丢失，你可以恢复可靠有状态服务和 Reliable Actors 的分区数据。 使用[包含备份存储详细信息的 GetBackupAPI](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getbackupsfrombackuplocation) 时，可以从列表中选择所需的备份。 备份枚举适用于应用程序、服务或分区。

以下示例假设丢失的群集是[为可靠有状态服务和 Reliable Actors 启用定期备份](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors)中所述的同一群集。 在本例中，部署的 `SampleApp` 已启用备份策略，备份配置为存储在 Azure 存储中。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
Get-SFBackupsFromBackupLocation -Application -ApplicationName 'fabric:/SampleApp' -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

执行某个 PowerShell 脚本，以使用 REST API 返回为 `SampleApp` 应用程序中所有分区创建的备份列表。 该 API 需要使用备份存储信息来列出可用备份。

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$BackupEntity = @{
    EntityKind = 'Application'
    ApplicationName='fabric:/SampleApp'
}

$BackupLocationAndEntityInfo = @{
    Storage = $StorageInfo
    BackupEntity = $BackupEntity
}

$body = (ConvertTo-Json $BackupLocationAndEntityInfo)
$url = "https://myalternatesfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

上述运行的示例输出：

```
BackupId                : b9577400-1131-4f88-b309-2bb1e943322c
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 20.55.16.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3334
CreationTimeUtc         : 2018-04-06T20:55:16Z
FailureError            : 
*
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
*
BackupId                : 69436834-c810-4163-9386-a7a800f78359
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.25.36.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3764
CreationTimeUtc         : 2018-04-06T21:25:36Z
FailureError            :
```

若要触发还原，请选择一个备份。 例如，用于灾难恢复的备份可能是以下备份：

```
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
```

对于还原 API，需要提供 _BackupId_ 和 _BackupLocation_ 详细信息。

此外，需要根据[分区方案](service-fabric-concepts-partitioning.md#get-started-with-partitioning)中的详述，选择备用群集中的某个目标分区。 备用群集备份将还原到在原始已丢失群集上的分区方案中指定的分区。

如果备用群集上的分区 ID 为 `1c42c47f-439e-4e09-98b9-88b8f60800c6`，则你可以通过比较范围分区 (UniformInt64Partition) 的高键值和低键值，将此 ID 映射到原始群集分区 ID `974bd92a-b395-4631-8a7f-53bd4ae9cf22`。

对于已命名分区，将比较名称值以识别备用群集中的目标分区。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

Restore-SFPartition  -PartitionId '1c42c47f-439e-4e09-98b9-88b8f60800c6' -BackupId 'b0035075-b327-41a5-a58f-3ea94b68faa4' -BackupLocation 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip' -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

使用以下[还原 API](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-restorepartition) 针对备份群集分区请求还原：

```powershell

$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$RestorePartitionReference = @{
    BackupId = 'b0035075-b327-41a5-a58f-3ea94b68faa4'
    BackupLocation = 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'
    BackupStorage  = $StorageInfo
}

$body = (ConvertTo-Json $RestorePartitionReference) 
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/1c42c47f-439e-4e09-98b9-88b8f60800c6/$/Restore?api-version=6.4" 

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

可以使用 TrackRestoreProgress 跟踪还原进度。

### <a name="data-restore-for-data-corruptiondata-loss"></a>在数据损坏/数据丢失时进行数据还原

对于数据丢失或数据损坏的情况，可靠有状态服务和 Reliable Actors 的已备份分区可以还原到任何所选备份。

以下示例是[为可靠有状态服务和 Reliable Actors 启用定期备份](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors)中示例的延续。 在此示例中，为分区启用了备份策略，服务按所需的频率在 Azure 存储中创建备份。

从 [GetBackupAPI](service-fabric-backuprestoreservice-quickstart-azurecluster.md#list-backups) 的输出中选择备份。 在此方案中，备份是从过去的同一个群集生成的。

若要触发还原，请从列表中选择一个备份。 对于当前存在的数据丢失/数据损坏，请选择以下备份：

```
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
```

对于还原 API，请提供 _BackupId_ 和 _BackupLocation_ 详细信息。 群集已启用备份，因此 Service Fabric 备份还原服务 (BRS) 可识别出关联备份策略中的正确存储位置。


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
Restore-SFPartition  -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' -BackupId 'b0035075-b327-41a5-a58f-3ea94b68faa4' -BackupLocation 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'

```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

```powershell
$RestorePartitionReference = @{
    BackupId = 'b0035075-b327-41a5-a58f-3ea94b68faa4',
    BackupLocation = 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'
}

$body = (ConvertTo-Json $RestorePartitionReference)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Restore?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

可以使用 TrackRestoreProgress 跟踪还原进度。

## <a name="track-restore-progress"></a>跟踪还原进度

可靠有状态服务或 Reliable Actor 的分区一次仅接受一个还原请求。 仅当已完成当前的还原请求后，分区才接受另一个请求。 可以同时在不同的分区上触发多个还原请求。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
    Get-SFPartitionRestoreProgress -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22'
```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetRestoreProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

$restoreResponse = (ConvertFrom-Json $response.Content)
$restoreResponse | Format-List
```

按以下顺序显示还原请求进度：

1. **Accepted**：_Accepted_ 还原状态表示已使用正确的请求参数触发请求的分区。
    ```
    RestoreState  : Accepted
    TimeStampUtc  : 0001-01-01T00:00:00Z
    RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
    RestoredLsn   : 3552
    ```
2. **InProgress**：_InProgress_ 还原状态表示正在使用请求中所述的备份来还原分区。 该分区将报告 _dataloss_ 状态。
    ```
    RestoreState  : RestoreInProgress
    TimeStampUtc  : 0001-01-01T00:00:00Z
    RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
    RestoredLsn   : 3552
    ```
    
3. **Success**、**Failure** 或 **Timeout**：请求的还原可能是在以下任一状态下完成的。 每种状态的含义和响应详细信息如下：
    - **成功**：_Success_ 还原状态表示已重新获取分区状态。 分区将报告 _RestoredEpoch_ 和 _RestoredLSN_ 状态，以及 UTC 格式的时间。

        ```
        RestoreState  : Success
        TimeStampUtc  : 2018-11-22T11:22:33Z
        RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
        RestoredLsn   : 3552
        ```        
    - **失败**：_Failure_ 还原状态表示还原请求失败。 将报告失败原因。

        ```
        RestoreState  : Failure
        TimeStampUtc  : 0001-01-01T00:00:00Z
        RestoredEpoch : 
        RestoredLsn   : 0
        ```
    - **Timeout**：_Timeout_ 还原状态表示请求超时。 请使用更大的 [RestoreTimeout](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout) 建议新的还原请求。 默认超时为 10 分钟。 在再次请求还原之前，请确保分区不处于数据丢失状态。
     
        ```
        RestoreState  : Timeout
        TimeStampUtc  : 0001-01-01T00:00:00Z
        RestoredEpoch : 
        RestoredLsn   : 0
        ```

## <a name="automatic-restore"></a>自动还原

可将 Service Fabric 群集中的可靠有状态服务和 Reliable Actors 分区配置为自动还原。 在备份策略中，将 `AutoRestore` 设置为 _true_。 启用自动还原可以在报告数据丢失情况时，从最新的分区备份还原数据。 有关详细信息，请参阅：

- [备份策略中的自动还原支持](service-fabric-backuprestoreservice-configure-periodic-backup.md#auto-restore-on-data-loss)
- [RestorePartition API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-restorepartition)
- [GetPartitionRestoreProgress API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionrestoreprogress)

## <a name="next-steps"></a>后续步骤
- [了解定期备份配置](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [备份还原 REST API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)
