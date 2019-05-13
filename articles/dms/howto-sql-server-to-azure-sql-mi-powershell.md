---
title: 使用数据库迁移服务和 PowerShell 将 SQL Server 迁移到 Azure SQL 数据库托管实例 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 从本地 SQL Server 迁移到 Azure SQL 数据库托管实例。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 04/29/2019
ms.openlocfilehash: d83410efd26f8c2078d3abdb01d061db0b83d33d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233731"
---
# <a name="migrate-sql-server-on-premises-to-an-azure-sql-database-managed-instance-using-azure-powershell"></a>将本地 SQL Server 迁移到使用 Azure PowerShell 为 Azure SQL 数据库托管实例
在本文中，你迁移**Adventureworks2016**数据库还原到的 SQL Server 2005 的本地实例或上面到 Azure SQL 数据库实例通过管理 Microsoft Azure PowerShell。 您可以使用迁移数据库从本地 SQL Server 实例到 Azure SQL 数据库托管实例`Az.DataMigration`在 Microsoft Azure PowerShell 模块。

在本文中，学习如何：
> [!div class="checklist"]
>
> * 创建资源组。
> * 创建 Azure 数据库迁移服务的实例。
> * 创建迁移项目中的 Azure 数据库迁移服务实例。
> * 运行迁移。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文包含有关如何执行联机和脱机迁移的详细信息。

## <a name="prerequisites"></a>必备组件

若要完成这些步骤，需满足以下条件：

* [SQL Server 2016 或更高版本](https://www.microsoft.com/sql-server/sql-server-downloads)（任何版本）。
* 本地副本**AdventureWorks2016**数据库，这是可供下载[此处](https://docs.microsoft.com/sql/samples/adventureworks-install-configure?view=sql-server-2017)。
* 启用 TCP / IP 协议，使用 SQL Server Express 安装的情况下会默认禁用该协议。 遵循[启用或禁用服务器网络协议](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure)一文启用 TCP/IP 协议。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* Azure 订阅。 如果没有订阅，请在开始之前[创建一个免费帐户](https://azure.microsoft.com/free/)。
* Azure SQL 数据库托管的实例。 可以按照本文中的详细信息创建 Azure SQL 数据库托管的实例[创建 Azure SQL 数据库托管的实例](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started)。
* 若要下载并安装[数据迁移助手](https://www.microsoft.com/download/details.aspx?id=53595)v3.3 或更高版本。
* 创建使用 Azure 资源管理器部署模型，它提供 Azure 数据库迁移服务与你的本地源服务器的站点到站点连接使用的 Azure 虚拟网络 (VNet) [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)或[VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)。
* 在本文中所述，使用数据迁移助手，你的本地数据库和架构迁移已完成的评估[执行 SQL Server 迁移评估](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem)。
* 若要下载并安装`Az.DataMigration`模块 (版本 0.7.2 或更高版本) 从通过使用 PowerShell 库[Install-module PowerShell cmdlet](https://docs.microsoft.com/powershell/module/powershellget/Install-Module?view=powershell-5.1)。
* 若要确保用于连接到源 SQL Server 实例的凭据具有[CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql)权限。
* 托管的实例，确保用于连接到目标 Azure SQL 数据库的凭据在目标 Azure SQL 数据库托管实例数据库具有 CONTROL DATABASE 权限。

    > [!IMPORTANT]
    > 对于联机迁移，必须已经安装了 Azure Active Directory 凭据。 有关详细信息，请参阅文章[使用门户创建 Azure AD 应用程序和服务主体可访问资源的](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)。

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>登录到 Microsoft Azure 订阅

使用 PowerShell 中登录到你的 Azure 订阅。 有关详细信息，请参阅文章[使用 Azure PowerShell 登录](https://docs.microsoft.com/powershell/azure/authenticate-azureps)。

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

使用创建的资源组[ `New-AzResourceGroup` ](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)命令。

下面的示例创建名为的资源组*myResourceGroup*中*美国东部*区域。

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>创建 Azure 数据库迁移服务的实例

可以通过 `New-AzDataMigrationService` cmdlet 创建 Azure 数据库迁移服务的新实例。
此 cmdlet 需要以下必需参数：

* Azure 资源组名称。 可以使用[ `New-AzResourceGroup` ](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)命令以创建前面所示的 Azure 资源组，并提供其名称作为参数。
* 服务名称。 对应于 Azure 数据库迁移服务的所需的唯一服务名称的字符串。
* *位置*。 指定服务的位置。 指定 Azure 数据中心位置，例如美国西部或亚洲东南部。
* Sku。 此参数对应于 DMS Sku 名称。 当前支持的 Sku 名称为*Basic_1vCore*， *Basic_2vCores*， *GeneralPurpose_4vCores*。
* 虚拟子网标识符。 可以使用 cmdlet [ `New-AzVirtualNetworkSubnetConfig` ](https://docs.microsoft.com//powershell/module/az.network/new-azvirtualnetworksubnetconfig)创建子网。

下面的示例创建一个名为服务*MyDMS*中的资源组*MyDMSResourceGroup*位于*美国东部*使用名为的虚拟网络区域*MyVNET*和名为的子网*MySubnet*。

> [!IMPORTANT]
> 以下代码段是为脱机迁移，不需要的基于高级 SKU 的 Azure 数据库迁移服务实例。 为联机迁移-Sku 参数的值必须包括高级 SKU。

```powershell
$vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>创建迁移项目

在创建 Azure 数据库迁移服务实例以后，创建迁移项目。 Azure 数据库迁移服务项目需要源和目标实例的连接信息，以及要在项目中迁移的数据库的列表。

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>创建源和目标连接的数据库连接信息对象

可以使用创建数据库连接信息对象`New-AzDmsConnInfo`cmdlet，需要以下参数：

* ServerType。 请求的数据库连接的类型，例如 SQL、Oracle 或 MySQL。 将 SQL 用于 SQL Server 和 Azure SQL。
* DataSource。 名称或 SQL Server 实例或 Azure SQL 数据库实例的 IP。
* AuthType。 连接的身份验证类型，可以为 SqlAuthentication 或 WindowsAuthentication。
* *TrustServerCertificate*。 此参数设置一个值，指示通道进行加密时绕过证书链以验证信任。 值可以是`$true`或`$false`。

下面的示例创建名为 SQL Server 的源的连接信息对象*MySourceSQLServer*使用 sql 身份验证：

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

下面的示例演示创建名为 targetmanagedinstance.database.windows.net 使用 sql 身份验证的 Azure SQL 数据库托管的实例服务器连接信息：

```powershell
$targetConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource "targetmanagedinstance.database.windows.net" `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$false
```

### <a name="provide-databases-for-the-migration-project"></a>为迁移项目提供数据库

创建一系列`AzDataMigrationDatabaseInfo`作为 Azure 数据库迁移服务项目，可作为参数提供用于创建项目的一部分指定数据库的对象。 可以使用 cmdlet`New-AzDataMigrationDatabaseInfo`若要创建`AzDataMigrationDatabaseInfo`。

下面的示例创建`AzDataMigrationDatabaseInfo`项目**AdventureWorks2016**数据库并将其添加到要为项目创建作为参数提供的列表。

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>创建项目对象

最后，可以创建名为的 Azure 数据库迁移服务项目*MyDMSProject*位于*美国东部*使用`New-AzDataMigrationProject`并添加先前创建的源和目标连接和要迁移的数据库的列表。

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLMI `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>创建并启动迁移任务

接下来，创建并启动 Azure 数据库迁移服务任务。 此任务要求目标和源以及要迁移的数据库表的列表的连接凭据信息和作为必备组件创建的项目与已提供的信息。

### <a name="create-credential-parameters-for-source-and-target"></a>创建源和目标的凭据参数

创建连接的安全凭据[PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0)对象。

下面的示例演示如何创建*PSCredential*对象的源和目标连接，提供密码作为字符串变量 *$sourcePassword*和 *$targetPassword*。

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-backup-fileshare-object"></a>创建备份的文件共享对象

现在，创建一个文件共享对象，该对象表示的本地到 Azure 数据库迁移服务可以备份源数据库使用的 SMB 网络共享`New-AzDmsFileShare`cmdlet。

```powershell
$backupPassword = ConvertTo-SecureString -String $password -AsPlainText -Force
$backupCred = New-Object System.Management.Automation.PSCredential ($backupUserName, $backupPassword)

$backupFileSharePath="\\10.0.0.76\SharedBackup"
$backupFileShare = New-AzDmsFileShare -Path $backupFileSharePath -Credential $backupCred
```

### <a name="create-selected-database-object"></a>创建选定的数据库对象

下一步是选择源和目标数据库使用`New-AzDmsSelectedDB`cmdlet。

下面的示例是从 SQL Server 的单个数据库迁移到 Azure SQL 数据库托管实例：

```powershell
$selectedDbs = @()
$selectedDbs += New-AzDmsSelectedDB -MigrateSqlServerSqlDbMi `
  -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -BackupFileShare $backupFileShare `
```

如果整个 SQL Server 实例需要提升并转移到 Azure SQL 数据库托管实例，则下面提供了一个循环，以便从源使所有数据库。 在以下示例中，对于 $Server、 $SourceUserName 和 $SourcePassword，SQL Server 详细信息提供你的源。

```powershell
$Query = "(select name as Database_Name from master.sys.databases where Database_id>4)";
$Databases= (Invoke-Sqlcmd -ServerInstance "$Server" -Username $SourceUserName
-Password $SourcePassword -database master -Query $Query)
$selectedDbs=@()
foreach($DataBase in $Databases.Database_Name)
    {
      $SourceDB=$DataBase
      $TargetDB=$DataBase
      
$selectedDbs += New-AzureRmDmsSelectedDB -MigrateSqlServerSqlDbMi `
                                              -Name $SourceDB `
                                              -TargetDatabaseName $TargetDB `
                                              -BackupFileShare $backupFileShare
      }
```

### <a name="sas-uri-for-azure-storage-container"></a>Azure 存储容器的 SAS URI

创建包含 SAS URI 的变量，使 Azure 数据库迁移服务能够访问它要将备份文件上传到的存储帐户容器。

```powershell
$blobSasUri="https://mystorage.blob.core.windows.net/test?st=2018-07-13T18%3A10%3A33Z&se=2019-07-14T18%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=qKlSA512EVtest3xYjvUg139tYSDrasbftY%3D"
```

### <a name="additional-configuration-requirements"></a>其他配置要求

有几个其他要求需要解决，但具体取决于是否正在执行脱机或联机迁移方面有所不同。

#### <a name="offline-migrations"></a>脱机迁移

对于脱机迁移，执行以下其他配置任务。

* **选择登录名**。 创建要迁移，在下面的示例所示的登录名的列表：

    ```powershell
    $selectedLogins = @(“user1”, “user2”)
    ```

    > [!IMPORTANT]
    > 目前，Azure 数据库迁移服务仅支持迁移 SQL 登录名。

* **选择代理作业**。 创建列表的代理作业以迁移在下面的示例所示：

    ```powershell
    $selectedAgentJobs = @("agentJob1", "agentJob2")
    ```

    > [!IMPORTANT]
    > 目前，Azure 数据库迁移服务仅支持使用 T-SQL 的子系统的作业步骤的作业。

#### <a name="online-migrations"></a>在线迁移

对于联机的迁移，执行以下其他配置任务。

* **配置 Azure Active Directory 应用**

    ```powershell
    $pwd = "Azure App Key"
    $appId = "Azure App Id"
    $AppPasswd = ConvertTo-SecureString $pwd -AsPlainText -Force
    $app = New-AzDmsAdApp -ApplicationId $appId -AppKey $AppPasswd
    ```

* **配置存储资源**

    ```powershell
    $storageResourceId = "Storage Resource Id"
    ```

### <a name="create-and-start-the-migration-task"></a>创建并启动迁移任务

使用 `New-AzDataMigrationTask` cmdlet 创建并启动迁移任务。

#### <a name="specify-parameters"></a>指定参数

##### <a name="common-parameters"></a>通用参数

无论执行脱机或联机迁移， `New-AzDataMigrationTask` cmdlet 需要以下参数：

* TaskType。 要创建的迁移任务的类型。对于从 SQL Server 到 Azure SQL 数据库托管实例的迁移类型，此项应为 *MigrateSqlServerSqlDbMi*。 
* ResourceGroupName。 要在其中创建任务的 Azure 资源组的名称。
* ServiceName。 要在其中创建任务的 Azure 数据库迁移服务实例。
* ProjectName。 要在其中创建任务的 Azure 数据库迁移服务项目的名称。 
* TaskName。 要创建的任务的名称。 
* *SourceConnection*。 AzDmsConnInfo 表示源 SQL Server 连接对象。
* *TargetConnection*。 AzDmsConnInfo 对象，表示目标 Azure SQL 数据库托管实例连接。
* SourceCred。 [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) 对象，用于连接到源服务器。
* TargetCred。 [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) 对象，用于连接到目标服务器。
* SelectedDatabase。 AzDataMigrationSelectedDB 对象，表示源和目标数据库的映射。
* *BackupFileShare*。 表示 Azure 数据库迁移服务可在其上创建源数据库备份的本地网络共享的 FileShare 对象。
* *BackupBlobSasUri*。 为 Azure 数据库迁移服务提供存储帐户容器访问权限的 SAS URI，该服务将备份文件上传到该容器。 了解如何获取 Blob 容器的 SAS URI。
* *SelectedLogins*。 要迁移的选定登录名列表。
* *SelectedAgentJobs*。 要迁移的选定代理作业列表。

##### <a name="additional-parameters"></a>其他参数

`New-AzDataMigrationTask` Cmdlet 也需要是唯一的正在执行的迁移，脱机或联机，类型的参数。

* **脱机迁移**。 对于脱机迁移， `New-AzDataMigrationTask` cmdlet 还需要以下参数：

  * *SelectedLogins*。 要迁移的选定登录名列表。
  * *SelectedAgentJobs*。 要迁移的选定代理作业列表。

* **在线迁移**。 对于联机迁移， `New-AzDataMigrationTask` cmdlet 还需要以下参数。

* *AzureActiveDirectoryApp*。 Active Directory 应用程序。
* *StorageResourceID*。 存储帐户的资源 ID。

#### <a name="create-and-start-an-offline-migration-task"></a>创建并启动脱机迁移任务

以下示例创建并启动名为脱机迁移任务**myDMSTask**:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMi `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -BackupFileShare $backupFileShare `
  -BackupBlobSasUri $blobSasUri `
  -SelectedLogins $selectedLogins `
  -SelectedAgentJobs $selectedJobs `
```

#### <a name="create-and-start-an-online-migration-task"></a>创建并启动联机迁移任务

以下示例创建并启动名为联机迁移任务**myDMSTask**:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMiSync `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -BackupFileShare $backupFileShare `
  -SelectedDatabase $selectedDbs `
  -AzureActiveDirectoryApp $app `
  -StorageResourceId $storageResourceId
```

## <a name="monitor-the-migration"></a>监视迁移

若要监视的迁移，请执行以下任务。

1. 将迁移的所有详细信息整合到一个名为 $CheckTask 变量。

    若要组合迁移详细信息，例如属性、 状态和与迁移相关联的数据库信息，请使用下面的代码段：

    ```powershell
    $CheckTask= Get-AzDataMigrationTask     -ResourceGroupName myResourceGroup `
                                            -ServiceName $service.Name `
                                        -ProjectName $project.Name `
                                            -Name myDMSTask `
                                            -ResultType DatabaseLevelOutput `
                        -Expand 
    Write-Host ‘$CheckTask.ProjectTask.Properties.Output’
    ```

2. 使用`$CheckTask`变量以获取迁移任务的当前状态。

    若要使用`$CheckTask`变量来获取迁移任务的当前状态，你可以监视运行的查询任务的状态属性，如下面的示例中所示的迁移任务：

    ```powershell
    if (($CheckTask.ProjectTask.Properties.State -eq "Running") -or ($CheckTask.ProjectTask.Properties.State -eq "Queued"))
    {
      write-host "migration task running"
    }
    Else if($CheckTask.ProjectTask.Properties.State -eq "Succeeded")
    { 
      write-host "Migration task is completed Successfully"
    }
    Else if($CheckTask.ProjectTask.Properties.State -eq "Failed" -or $CheckTask.ProjectTask.Properties.State -eq "FailedInputValidation"  -or $CheckTask.ProjectTask.Properties.State -eq "Faulted")
    { 
      write-host “Migration Task Failed”
    }
    ```

## <a name="performing-the-cutover-online-migrations-only"></a>执行直接转换 （联机迁移仅）

具有联机迁移执行完整备份和还原数据库，则和工作然后还原事务日志存储在 BackupFileShare 中继续进行。

当 Azure SQL 数据库托管实例中的数据库的新数据更新，并与源数据库保持同步时，你可以执行直接转换。

下面的示例将在完成 cutover\migration。 用户调用此命令在他们的判断。

```powershell
$command = Invoke-AzDmsCommand -CommandType CompleteSqlMiSync `
                               -ResourceGroupName myResourceGroup `
                               -ServiceName $service.Name `
                               -ProjectName $project.Name `
                               -TaskName myDMSTask `
                               -DatabaseName "Source DB Name"
```

## <a name="deleting-the-instance-of-azure-database-migration-service"></a>正在删除 Azure 数据库迁移服务的实例

在迁移完毕后，可以删除的 Azure 数据库迁移服务实例：

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="additional-resources"></a>其他资源

有关其他迁移方案 （源/目标对） 的信息，请参阅 Microsoft[数据库迁移指南](https://datamigration.microsoft.com/)。

## <a name="next-steps"></a>后续步骤

一文中了解有关 Azure 数据库迁移服务的详细信息[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)。
