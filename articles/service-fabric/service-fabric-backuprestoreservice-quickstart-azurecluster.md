---
title: Azure Service Fabric 中的定期备份和还原 | Microsoft Docs
description: 使用 Service Fabric 的定期备份和还原功能来实现应用程序数据的定期数据备份。
services: service-fabric
documentationcenter: .net
author: hrushib
manager: chackdan
editor: hrushib
ms.assetid: FAA58600-897E-4CEE-9D1C-93FACF98AD1C
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/29/2018
ms.author: hrushib
ms.openlocfilehash: 9bce408215cef540604a72109bc5b29ebc3359e7
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413803"
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

Service Fabric 中的备份和还原服务可以轻松自动备份存储在有状态服务中的信息。 定期备份应用程序数据是防止数据丢失和服务不可用的基础。 Service Fabric 提供可选的备份和还原服务，因此无需编写任何其他代码，便可配置有状态可靠服务（包括角色服务）的定期备份。 它还有助于还原以前执行的备份。 


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
* 具有 Fabric 6.2 及更高版本的 Service Fabric 群集。 应在 Windows Server 上安装群集。 有关使用 Azure 资源模板创建 Service Fabric 群集的步骤，请参阅此[文章](service-fabric-cluster-creation-via-arm.md)。
* 用于加密机密的 X.509 证书，连接到存储以存储备份时需要此机密。 请参阅[文章](service-fabric-cluster-creation-via-arm.md)，了解如何获取或创建 X.509 证书。
* 使用 Service Fabric SDK 3.0 或更高版本生成的 Service Fabric 可靠有状态应用程序。 对于面向.NET Core 2.0 的应用程序，应用程序应使用生成 Service Fabric SDK 版本 3.1 或更高版本。
* 创建 Azure 存储帐户，用于存储应用程序备份。
* 安装适用于配置调用 Microsoft.ServiceFabric.Powershell.Http 模块 [中预览]。

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

* 请确保群集已连接使用`Connect-SFCluster`命令之前发出任何使用 Microsoft.ServiceFabric.Powershell.Http 模块的配置请求。

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>启用备份和还原服务

### <a name="using-azure-portal"></a>使用 Azure 门户

启用`Include backup restore service`下的复选框`+ Show optional settings`中`Cluster Configuration`选项卡。

![启用通过门户的备份还原服务][1]


### <a name="using-azure-resource-manager-template"></a>使用 Azure 资源管理器模板
首先，需要在群集中启用备份和还原服务。 获取要部署的群集的模板。 可使用[示例模板](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype)或创建资源管理器模板。 通过以下步骤启用备份和还原服务：

1. 检查 `apiversion` 是否针对 `Microsoft.ServiceFabric/clusters` 资源设置为 `2018-02-01`，如果没有，请按以下代码片段所示进行更新：

    ```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. 现在，通过在 `properties` 部分下添加以下 `addonFeatures` 部分来启用备份和还原服务，如以下代码片段所示： 

    ```json
        "properties": {
            ...
            "addonFeatures":  ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```
3. 配置 X.509 证书以用于加密凭据。 此步骤非常重要，可确保在保留之前对提供用于连接存储的凭据进行加密。 通过在 `fabricSettings` 部分下添加以下 `BackupRestoreService` 部分来配置加密证书，如以下代码片段所示： 

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

4. 通过前述更改更新群集模板后，应用更改并等待部署/升级完成。 完成后，备份和还原服务开始在群集中运行。 此服务的 URI 为 `fabric:/System/BackupRestoreService`，并且此服务可位于 Service Fabric Explorer 中系统服务部分下。 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>启用可靠有状态服务和 Reliable Actors 的定期备份
让我们通过一些步骤来启用可靠有状态服务和 Reliable Actors 的定期备份。 这些步骤假定
- 通过备份和还原服务，使用 X.509 安全性安装群集。
- 在群集上部署了可靠有状态服务。 在本快速入门指南中，应用程序 URI 为 `fabric:/SampleApp`，属于此应用程序的可靠有状态服务的 URI 为 `fabric:/SampleApp/MyStatefulService`。 使用单个分区部署此服务，分区 ID 为 `974bd92a-b395-4631-8a7f-53bd4ae9cf22`。
- 具有管理员角色的客户端证书安装计算机上 CurrentUser 证书存储位置的“我的”（个人）存储名称中，可从其中调用以下脚本。 本示例使用 `1b7ebe2174649c45474a4819dafae956712c31d3` 作为此证书的指纹。 有关访问客户端证书的详细信息，请参阅[适用于 Service Fabric 客户端的基于角色的访问控制](service-fabric-cluster-security-roles.md)。

### <a name="create-backup-policy"></a>创建备份策略

第一步是创建描述备份计划的备份策略、备份数据的目标存储、策略名称、触发完整备份之前允许的最大递增备份以及备份存储的保留策略。 

有关备份存储，请使用上面创建的 Azure 存储帐户。 容器 `backup-container` 配置为存储备份。 在备份上传期间，将创建具有该名称的容器（如果该容器尚未存在）。 使用 Azure 存储帐户的有效连接字符串填充 `ConnectionString`，并将 `account-name` 替换为你的存储帐户名，将 `account-key` 替换为你的存储帐户密钥。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

执行以下 PowerShell cmdlet 用于创建新的备份策略。 请将 `account-name` 替换为你的存储帐户名，将 `account-key` 替换为你的存储帐户密钥。

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container' -Basic -RetentionDuration '10.00:00:00'

```

#### <a name="rest-call-using-powershell"></a>使用 PowerShell 的 rest 调用

执行以下 PowerShell 脚本，调用所需的 REST API 来创建新策略。 请将 `account-name` 替换为你的存储帐户名，将 `account-key` 替换为你的存储帐户密钥。

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
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
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

```

> [!IMPORTANT]
> 由于运行时中的问题，请确保保留策略中的保留期配置为小于 24 天，否则将导致备份还原服务进入仲裁丢失后副本故障转移。

### <a name="enable-periodic-backup"></a>启用定期备份
在定义备份策略以满足应用程序的数据保护要求后，备份策略应与应用程序相关联。 根据需要，备份策略可与应用程序、服务或分区相关联。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell

Enable-SFApplicationBackup -ApplicationId 'SampleApp' -BackupPolicyName 'BackupPolicy1'

```
#### <a name="rest-call-using-powershell"></a>使用 PowerShell 的 rest 调用

执行以下 PowerShell 脚本，调用所需的 REST API，将上面步骤中创建的名为 `BackupPolicy1` 的备份策略与应用程序 `SampleApp` 相关联。

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
``` 

### <a name="verify-that-periodic-backups-are-working"></a>验证定期备份是否正常工作

在应用程序级别启用备份后，属于应用程序下的可靠有状态服务和 Reliable Actors 的所有分区将根据关联的备份策略开始定期备份。 

![分区备份运行状况事件][0]

### <a name="list-backups"></a>列出备份

可使用 GetBackups API 来枚举属于应用程序的可靠有状态服务和 Reliable Actors 的所有分区的关联备份。 可为应用程序、服务或分区枚举备份。

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell 中使用 Microsoft.ServiceFabric.Powershell.Http 模块

```powershell
    
Get-SFApplicationBackupList -ApplicationId WordCount
```

#### <a name="rest-call-using-powershell"></a>使用 PowerShell 的 rest 调用

执行以下 PowerShell 脚本，调用 HTTP API 来枚举为 `SampleApp` 应用程序内所有分区创建的备份。

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

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

## <a name="limitation-caveats"></a>限制/注意事项
- Service Fabric PowerShell cmdlet 是在预览模式下。
- Linux 上不支持 Service Fabric 群集。

## <a name="known-issues"></a>已知问题
- 确保保留期配置为小于 24 天。 


## <a name="next-steps"></a>后续步骤
- [了解定期备份配置](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [备份还原 REST API 参考](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/PartitionBackedUpHealthEvent_Azure.png
[1]: ./media/service-fabric-backuprestoreservice/enable-backup-restore-service-with-portal.png

