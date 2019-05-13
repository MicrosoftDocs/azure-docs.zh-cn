---
title: Azure Service Fabric 中的定期备份和还原 | Microsoft Docs
description: 使用 Service Fabric 的定期备份和还原功能来实现应用程序数据的定期数据备份。
services: service-fabric
documentationcenter: .net
author: hrushib
manager: chackdan
editor: hrushib
ms.assetid: FAADBCAB-F0CF-4CBC-B663-4A6DCCB4DEE1
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/29/2018
ms.author: hrushib
ms.openlocfilehash: 28378b4b769e0d0e70a82a45baac0872d1476036
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413641"
---
# <a name="periodic-backup-and-restore-in-azure-service-fabric"></a>Azure Service Fabric 中的定期备份和还原
> [!div class="op_single_selector"]
> * [Azure 上的群集](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [独立群集](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service Fabric 是一种分布式系统平台，用于轻松开发和管理基于微服务的可靠的分布式云应用程序。 它允许运行无状态和有状态的微服务。 有状态服务可在请求和响应或完整的事务之外维持可变的权威状态。 如果有状态服务长时间不可用或由于灾难而丢失信息，可能需要还原到其状态的某个最近备份，以便在其备份后继续提供服务。

Service Fabric 跨多个节点复制状态，确保服务高度可用。 即使群集中的一个节点出现故障，服务也将继续可用。 然而，在某些情况下，仍然需要服务数据能够可靠应对更广泛的故障。
 
例如，服务可能要备份其数据，以防止出现以下情况：
- 整个 Service Fabric 群集永久丢失。
- 大部分服务分区副本永久丢失
- 状态被意外删除或受损而引起管理错误。 例如，具有足够权限的管理员错误地删除了服务。
- 服务中的 bug 导致数据损坏。 例如，当某个服务代码升级程序开始将错误数据写入到可靠集合中时可能发生此情况。 在此情况下，代码和数据可能都必须还原到先前的状态。
- 离线数据处理。 对于商业智能来说使用离线处理的数据很方便，此处理是独立于生成数据的服务进行的。

Service Fabric 提供了一个内置 API，用于执行时间点[备份和还原](service-fabric-reliable-services-backup-restore.md)。 应用程序开发者可使用这些 API 定期备份服务状态。 此外，如果服务管理员想要在特定时间从服务外部触发备份，就像在升级应用程序之前一样，开发者需要将备份（和还原）作为服务的 API 公开。 维护备份是以上操作的额外成本。 例如，你可能希望每半小时进行 5 次递增备份，然后进行完整备份。 完整备份后，可删除以前的递增备份。 此方法需要额外的代码，因而在应用程序开发期间产生额外成本。

定期备份应用程序数据是管理分布式应用程序以及防范数据丢失或长时间丢失服务可用性的基本要求。 Service Fabric 提供可选的备份和还原服务，因此无需编写任何其他代码，便可配置有状态可靠服务（包括角色服务）的定期备份。 它还有助于还原以前执行的备份。 

Service Fabric 提供了一组 API 以实现与定期备份和还原功能相关的以下功能：

- 通过支持将备份上传到（外部）存储位置，计划可靠有状态服务和 Reliable Actors 的定期备份。 受支持的存储位置
    - Azure 存储
    - 文件共享（本地）
- 枚举备份
- 触发即席备份的分区
- 使用之前的备份还原分区
- 暂时暂停备份
- 备份的保留期管理（即将推出）

## <a name="prerequisites"></a>必备组件
* 具有 Fabric 6.2 及更高版本的 Service Fabric 群集。 应在 Windows Server 上安装群集。 有关下载所需包的步骤，请参阅[文章](service-fabric-cluster-creation-for-windows-server.md)。
* 用于加密机密的 X.509 证书，连接到存储以存储备份时需要此机密。 请参阅[文章](service-fabric-windows-cluster-x509-security.md)，了解如何获取或创建一个自签名的 X.509 证书。

* 使用 Service Fabric SDK 3.0 或更高版本生成的 Service Fabric 可靠有状态应用程序。 对于面向 .Net Core 2.0 的应用程序，应使用 Service Fabric SDK 3.1 或更高版本生成应用程序。
* 安装适用于配置调用 Microsoft.ServiceFabric.Powershell.Http 模块 [中预览]。

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

* 请确保群集已连接使用`Connect-SFCluster`命令之前发出任何使用 Microsoft.ServiceFabric.Powershell.Http 模块的配置请求。

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>启用备份和还原服务
首先，需要在群集中启用备份和还原服务。 获取要部署的群集的模板。 可使用[示例模板](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)。 通过以下步骤启用备份和还原服务：

1. 检查在群集配置文件中 `apiversion` 是否设置为了 `10-2017`，如果没有，请按以下代码片段所示进行更新：

    ```json
    {
        "apiVersion": "10-2017",
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        ...
    }
    ```

2. 现在，通过在 `properties` 部分下添加以下 `addonFeatures` 部分来启用备份和还原服务，如以下代码片段所示： 

    ```json
        "properties": {
            ...
            "addonFeatures": ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```

3. 配置 X.509 证书以用于加密凭据。 此步骤非常重要，可确保在保留之前对提供用于连接存储的凭据（如果有）进行加密。 通过在 `fabricSettings` 部分下添加以下 `BackupRestoreService` 部分来配置加密证书，如以下代码片段所示： 

    ```json
    "properties": {
        ...
        "addonFeatures": ["BackupRestoreService"],
        "fabricSettings": [{
            "name": "BackupRestoreService",
            "parameters":  [{
                "name": "SecretEncryptionCertThumbprint",
                "value": "[Thumbprint]"
            }]
        }
        ...
    }
    ```

4. 通过前述更改更新群集配置文件后，应用更改并等待部署/升级完成。 完成后，备份和还原服务开始在群集中运行。 此服务的 URI 为 `fabric:/System/BackupRestoreService`，并且此服务可位于 Service Fabric Explorer 中系统服务部分下。 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>启用可靠有状态服务和 Reliable Actors 的定期备份
让我们通过一些步骤来启用可靠有状态服务和 Reliable Actors 的定期备份。 这些步骤假定
- 通过备份和还原服务安装群集。
- 在群集上部署了可靠有状态服务。 在本快速入门指南中，应用程序 URI 为 `fabric:/SampleApp`，属于此应用程序的可靠有状态服务的 URI 为 `fabric:/SampleApp/MyStatefulService`。 使用单个分区部署此服务，分区 ID 为 `23aebc1e-e9ea-4e16-9d5c-e91a614fefa7`。  

### <a name="create-backup-policy"></a>创建备份策略

第一步是创建描述备份计划的备份策略、备份数据的目标存储、策略名称、触发完整备份之前允许的最大递增备份以及备份存储的保留策略。 

对于备份存储，请创建文件共享并为所有 Service Fabric 节点计算机提供对此文件共享的读写访问权限。 此示例假定名为 `BackupStore` 的共享存在于 `StorageServer` 上。


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -FileShare -Path '\\StorageServer\BackupStore' -Basic -RetentionDuration '10.00:00:00'

```
#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

执行以下 PowerShell 脚本，调用所需的 REST API 来创建新策略。

```powershell
$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
}   

$StorageInfo = @{
    Path = '\\StorageServer\BackupStore'
    StorageKind = 'FileShare'
}

$RetentionPolicy = @{ 
    RetentionPolicyType = 'Basic'
    RetentionDuration =  'P10D'
}

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
    RetentionPolicy = $RetentionPolicy
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "http://localhost:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
```

> [!IMPORTANT]
> 由于运行时中的问题，请确保保留策略中的保留期配置为小于 24 天，否则将导致备份还原服务进入仲裁丢失后副本故障转移。

### <a name="enable-periodic-backup"></a>启用定期备份
在定义策略以满足应用程序的数据保护要求后，备份策略应与应用程序相关联。 根据需要，备份策略可与应用程序、服务或分区相关联。


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
Enable-SFApplicationBackup -ApplicationId 'SampleApp' -BackupPolicyName 'BackupPolicy1'
```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用
执行以下 PowerShell 脚本，调用所需的 REST API，将上面步骤中创建的名为 `BackupPolicy1` 的备份策略与应用程序 `SampleApp` 相关联。

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "http://localhost:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
``` 

### <a name="verify-that-periodic-backups-are-working"></a>验证定期备份是否正常工作

对应用程序启用备份后，属于应用程序下的可靠有状态服务和 Reliable Actors 的所有分区将根据关联的备份策略开始定期备份。

![分区备份运行状况事件][0]

### <a name="list-backups"></a>列出备份

可使用 GetBackups API 来枚举属于应用程序的可靠有状态服务和 Reliable Actors 的所有分区的关联备份。 根据需要，可为应用程序、服务或分区枚举备份。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>Powershell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
    Get-SFApplicationBackupList -ApplicationId WordCount     
```

#### <a name="rest-call-using-powershell"></a>使用 Powershell 的 rest 调用

执行以下 PowerShell 脚本，调用 HTTP API 来枚举为 `SampleApp` 应用程序内所有分区创建的备份。

```powershell
$url = "http://localhost:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get

$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

上述运行的示例输出：

```
BackupId                : d7e4038e-2c46-47c6-9549-10698766e714
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.39.40.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2058
CreationTimeUtc         : 2018-04-01T19:39:40Z
FailureError            : 

BackupId                : 8c21398a-2141-4133-b4d7-e1a35f0d7aac
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.54.38.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2237
CreationTimeUtc         : 2018-04-01T19:54:38Z
FailureError            : 

BackupId                : fc75bd4c-798c-4c9a-beee-e725321f73b2
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 20.09.44.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2437
CreationTimeUtc         : 2018-04-01T20:09:44Z
FailureError            : 
```

## <a name="known-issues"></a>已知问题
- 确保保留期配置为小于 24 天。 
- 十进制分隔符不为“.”的区域设置中不会出现备份还原服务
- 备份还原服务未能在使用基于 gMSA 的安全进行保护的群集上出现。

## <a name="limitation-caveats"></a>限制/注意事项
- Service Fabric PowerShell cmdlet 是在预览模式下。
- Linux 上不支持 Service Fabric 群集。

## <a name="next-steps"></a>后续步骤
- [了解定期备份配置](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [备份还原 REST API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/PartitionBackedUpHealthEvent.png

