---
title: Azure Service Fabric 中的按需备份 | Microsoft Docs
description: 使用 Service Fabric 中的备份和还原功能来实现应用程序数据的按需备份。
services: service-fabric
documentationcenter: .net
author: aagup
manager: chackdan
editor: aagup
ms.assetid: 02DA262A-EEF6-4F90-842E-FFC4A09003E5
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/30/2018
ms.author: aagup
ms.openlocfilehash: bed3402de83984cae9134fe44058980ec18861b3
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413936"
---
# <a name="on-demand-backup-in-azure-service-fabric"></a>Azure Service Fabric 中的按需备份

可将可靠有状态服务和 Reliable Actors 的数据进行备份，以解决灾难或数据丢失的情况。

Azure Service Fabric 提供[定期备份数据](service-fabric-backuprestoreservice-quickstart-azurecluster.md)和按需备份数据的功能。 按需备份很有用，因为它可防止由于在基础服务或其环境中的计划更改而导致的数据丢失/数据损坏。

在手动触发服务或服务环境操作之前，按需备份功能有助于捕获服务的状态。 例如，如果你在升级或降级服务时对服务二进制文件做出了更改。 在这种情况下，按需备份可以帮助防止应用程序代码 bug 导致数据损坏。
## <a name="prerequisites"></a>必备组件

- 安装适用于配置调用 Microsoft.ServiceFabric.Powershell.Http 模块 [中预览]。

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

- 请确保群集已连接使用`Connect-SFCluster`命令之前发出任何使用 Microsoft.ServiceFabric.Powershell.Http 模块的配置请求。

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```


## <a name="triggering-on-demand-backup"></a>触发按需备份

按需备份要求提供存储详细信息用于上传备份文件。 可在定期备份策略或按需备份请求中指定按需备份位置。

### <a name="on-demand-backup-to-storage-specified-by-a-periodic-backup-policy"></a>按需备份到定期备份策略指定的存储

可将定期备份策略配置为使用可靠有状态服务或 Reliable Actor 的某个分区，以便在存储中创建额外的按需备份。

以下用例是[为可靠有状态服务和 Reliable Actors 启用定期备份](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors)中的方案的延续。 在本例中，你启用了一个备份策略，以使用某个分区，并按设置的频率在 Azure 存储中创建备份。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' 

```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

使用 [BackupPartition](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition) API 为分区 ID `974bd92a-b395-4631-8a7f-53bd4ae9cf22` 的按需备份设置触发。

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

使用 [GetBackupProgress](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) API 启用[按需备份进度](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress)跟踪。

### <a name="on-demand-backup-to-specified-storage"></a>指定存储的按需备份

可以请求可靠有状态服务或 Reliable Actor 的分区的按需备份。 请在按需备份请求中提供存储信息。


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' -AzureBlobStore -ConnectionString  'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

使用 [BackupPartition](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition) API 为分区 ID `974bd92a-b395-4631-8a7f-53bd4ae9cf22` 的按需备份设置触发。 包括以下 Azure 存储信息：

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$OnDemandBackupRequest = @{
    BackupStorage = $StorageInfo
}

$body = (ConvertTo-Json $OnDemandBackupRequest)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

可以使用 [GetBackupProgress](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) API 设置[按需备份进度](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress)跟踪。

## <a name="tracking-on-demand-backup-progress"></a>跟踪按需备份进度

可靠有状态服务或 Reliable Actor 的分区一次仅接受一个按需备份请求。 仅当已完成当前的按需备份请求后，才能接受另一个请求。

不同的分区可以同时触发按需备份请求。


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

Get-SFPartitionBackupProgress -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22'

```
#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetBackupProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3' 
$backupResponse = (ConvertFrom-Json $response.Content) 
$backupResponse
```

按需备份请求可处于以下状态：

- **Accepted**：备份已针对分区启动，且正在进行。
  ```
  BackupState             : Accepted
  TimeStampUtc            : 0001-01-01T00:00:00Z
  BackupId                : 00000000-0000-0000-0000-000000000000
  BackupLocation          :
  EpochOfLastBackupRecord :
  LsnOfLastBackupRecord   : 0
  FailureError            :
  ```
- **Success**、**Failure** 或 **Timeout**：请求的按需备份可能是在以下任一状态下完成的：
  - **成功**：_Success_ 备份状态表示已成功完成备份状态。 响应将提供分区的 _BackupEpoch_ 和 _BackupLSN_ 以及 UTC 格式的时间。
    ```
    BackupState             : Success
    TimeStampUtc            : 2018-11-21T20:00:01Z
    BackupId                : 5d64b697-6acd-45a4-adbd-3d75e0078081
    BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-11-21 20.00.01.zip
    EpochOfLastBackupRecord : @{DataLossNumber=131873018908156893; ConfigurationNumber=8589934592}
    LsnOfLastBackupRecord   : 36
    FailureError            :
    ```
  - **失败**：_Failure_ 备份状态表示在备份分区状态期间发生失败。 响应中会指明失败的原因。
    ```
    BackupState             : Failure
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_BACKUPCOPIER_UNEXPECTED_ERROR; Message=An error occurred during this operation.  Please check the trace logs for more details.}
    ```
  - **Timeout**：_Timeout_ 备份状态表示在给定的时间内无法创建分区状态备份。 默认超时值为 10 分钟。 如果发生这种情况，请使用更大的 [BackupTimeout](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout) 发起新的按需备份请求。
    ```
    BackupState             : Timeout
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_TIMEOUT; Message=The request of backup has timed out.}
    ```

## <a name="next-steps"></a>后续步骤

- [了解定期备份配置](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [BackupRestore REST API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)
