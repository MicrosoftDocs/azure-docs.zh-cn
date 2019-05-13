---
title: 管理已向 Azure 文件同步注册的服务器 | Microsoft Docs
description: 了解如何向 Azure 文件同步存储同步服务注册和注销 Windows Server。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 6b9996716621137945b5aeb2f0699bf1bddba4a6
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65189983"
---
# <a name="manage-registered-servers-with-azure-file-sync"></a>管理已向 Azure 文件同步注册的服务器
借助 Azure 文件同步，既可将组织的文件共享集中在 Azure 文件中，又不失本地文件服务器的灵活性、性能和兼容性。 它通过将 Windows Server 转换为 Azure 文件共享的快速缓存来实现这一点。 你可以使用 Windows Server 上的任意可用协议在本地访问数据（包括 SMB、NFS 和 FTPS），并且可以在世界各地获取所需的缓存数。

下面的文章说明如何向存储同步服务注册服务器并进行管理。 若要了解如何部署端到端的 Azure 文件同步，请参阅[如何部署 Azure 文件同步](storage-sync-files-deployment-guide.md)。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="registerunregister-a-server-with-storage-sync-service"></a>向存储同步服务注册/注销服务器
向 Azure 文件同步注册服务器可在 Windows Server 和 Azure 之间建立信任关系。 这种关系随后可用于创建服务器上的服务器终结点，该终结点表示应与 Azure 文件共享（也称为云终结点）同步的特定文件夹。 

### <a name="prerequisites"></a>必备组件
若要向存储同步服务注册服务器，首先必须确保服务器满足以下先决条件：

* 服务器必须运行支持的 Windows Server 版本。 有关详细信息，请参阅 [Azure 文件同步系统要求和互操作性](storage-sync-files-planning.md#azure-file-sync-system-requirements-and-interoperability)。
* 确保已部署存储同步服务。 有关如何部署存储同步服务的详细信息，请参阅[如何部署 Azure 文件同步](storage-sync-files-deployment-guide.md)。
* 确保服务器已连接到 Internet，并且 Azure 可以访问。
* 使用服务器管理器 UI 禁用适用于管理员的 IE 增强的安全配置。
    
    ![突出显示“IE 增强的安全配置”的服务器管理器 UI](media/storage-sync-files-server-registration/server-manager-ie-config.png)

* 确保服务器上安装了 Azure PowerShell 模块。 如果服务器是故障转移群集的成员，则群集中的每个节点都需要 Az 模块。 有关如何安装 Az 模块的更多详细信息可以在[安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps) 上找到。

    > [!Note]  
    > 我们建议使用最新版本的 Az PowerShell 模块注册/注销服务器。 如果 Az 包之前已安装在此服务器上（并且此服务器上的 PowerShell 版本是 5.* 或更高版本），则可以使用 `Update-Module` cmdlet 更新此包。 
* 如果你在环境中利用网络代理服务器，请在服务器上为要利用的同步代理配置代理设置。
    1. 确定代理 IP 地址和端口号
    2. 编辑以下这两个文件：
        * C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config
        * C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
    3. 在上面两个文件中的 /System.ServiceModel 下添加图 1（此部分下方）中的行，将 127.0.0.1:8888 更换为正确的 IP 地址（替换 127.0.0.1）和正确的端口号（替换 8888）：
    4. 通过命令行设置 WinHTTP 代理设置：
        * 显示代理：netsh winhttp show proxy
        * 设置代理：netsh winhttp set proxy 127.0.0.1:8888
        * 重置代理：netsh winhttp reset proxy
        * 如果这在安装了代理之后进行设置，则重启我们的同步代理：net stop filesyncsvc
    
```XML
    Figure 1:
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
    </system.net>
```    

### <a name="register-a-server-with-storage-sync-service"></a>向存储同步服务注册服务器
服务器必须先向存储同步服务注册，然后才能在 Azure 文件同步同步组中用作服务器终结点。 服务器一次只能向一个存储同步服务注册。

#### <a name="install-the-azure-file-sync-agent"></a>安装 Azure 文件同步代理
1. [下载 Azure 文件同步代理](https://go.microsoft.com/fwlink/?linkid=858257)。
2. 启动 Azure 文件同步代理安装程序。
    
    ![Azure 文件同步代理安装程序的第一个窗格](media/storage-sync-files-server-registration/install-afs-agent-1.png)

3. 请务必使用 Microsoft 更新启用 Azure 文件同步代理更新。 这非常重要，因为服务器包的关键安全修补和功能增强通过 Microsoft 更新提供。

    ![确保在 Azure 文件同步代理安装程序的“Microsoft 更新”窗格中启用 Microsoft 更新](media/storage-sync-files-server-registration/install-afs-agent-2.png)

4. 如果以前尚未注册服务器，则服务器注册 UI 会在完成安装之后立即弹出。

> [!Important]  
> 如果服务器是故障转移群集的成员，则需要在群集中的每个节点上安装 Azure 文件同步代理。

#### <a name="register-the-server-using-the-server-registration-ui"></a>使用服务器注册 UI 注册服务器
> [!Important]  
> 云解决方案提供商 (CSP) 订阅不能使用服务器注册 UI。 请改用 PowerShell（在本部分下面介绍）。

1. 如果服务器注册 UI 在完成 Azure 文件同步代理安装之后未立即启动，则可以通过执行 `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe` 来手动启动它。
2. 单击“登录”以访问 Azure 订阅。 

    ![服务器注册 UI 的打开对话框](media/storage-sync-files-server-registration/server-registration-ui-1.png)

3. 从对话框中选取正确的订阅、资源组和存储同步服务。

    ![存储同步服务信息](media/storage-sync-files-server-registration/server-registration-ui-2.png)

4. 在预览版中，需要再进行一次登录来完成过程。 

    ![“登录”对话框](media/storage-sync-files-server-registration/server-registration-ui-3.png)

> [!Important]  
> 如果服务器是故障转移群集的成员，则每台服务器都需要运行服务器注册。 在 Azure 门户中查看已注册的服务器时，Azure 文件同步会自动将每个节点识别为相同故障转移群集的成员，并相应地将它们分组在一起。

#### <a name="register-the-server-with-powershell"></a>使用 PowerShell 注册服务器
也可以通过 PowerShell 执行服务器注册。 这是云解决方案提供商 (CSP) 订阅唯一支持的服务器注册方法：

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll"
Login-AzStorageSync -SubscriptionID "<your-subscription-id>" -TenantID "<your-tenant-id>"
Register-AzStorageSyncServer -SubscriptionId "<your-subscription-id>" - ResourceGroupName "<your-resource-group-name>" - StorageSyncService "<your-storage-sync-service-name>"
```

### <a name="unregister-the-server-with-storage-sync-service"></a>向存储同步服务注销服务器
向存储同步服务注销服务器需要几个步骤。 让我们看看如何正确注销服务器。

> [!Warning]  
> 不要尝试通过取消注册和注册服务器或者删除并重新创建服务器终结点来解决同步、云分层或 Azure 文件同步的任何其他方面的问题，除非 Microsoft 工程师明确指示这样做。 取消注册服务器和删除服务器终结点是一种破坏性操作，并且在重新创建注册的服务器和服务器终结点后，具有服务器终结点的卷上的分层文件将不会“重新连接”到 Azure 文件共享上的其位置，这将导致同步错误。 另请注意，存在于服务器终结点命名空间之外的分层文件可能会永久丢失。 即使从未启用云分层，分层文件也可能存在于服务器终结点中。

#### <a name="optional-recall-all-tiered-data"></a>（可选）召回所有分层数据
如果希望在删除 Azure 文件同步后当前分层的文件可用（即，这是一个生产环境，而不是测试环境），请召回每个包含服务器终结点的卷上的所有文件。 对所有服务器终结点禁用云分层，然后运行以下 PowerShell cmdlet：

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <a-volume-with-server-endpoints-on-it>
```

> [!Warning]  
> 如果承载服务器终结点的本地卷空间不足，无法召回所有分层数据，则 `Invoke-StorageSyncFileRecall` cmdlet 会失败。  

#### <a name="remove-the-server-from-all-sync-groups"></a>从所有同步组中删除服务器
必须删除服务器的所有服务器终结点，然后才能在存储同步服务中注销该服务器。 可以通过 Azure 门户完成此操作：

1. 导航到在其中注册服务器的存储同步服务。
2. 在存储同步服务中的每个同步组中，删除此服务器的所有服务器终结点。 可以通过在“同步组”窗格中右键单击相关服务器终结点来完成。

    ![从同步组中删除服务器终结点](media/storage-sync-files-server-registration/sync-group-server-endpoint-remove-1.png)

这还可以通过简单的 PowerShell 脚本来实现：

```powershell
Connect-AzAccount

$storageSyncServiceName = "<your-storage-sync-service>"
$resourceGroup = "<your-resource-group>"

Get-AzStorageSyncGroup -ResourceGroupName $resourceGroup -StorageSyncServiceName $storageSyncServiceName | ForEach-Object { 
    $syncGroup = $_; 
    Get-AzStorageSyncServerEndpoint -ParentObject $syncGroup | Where-Object { $_.ServerEndpointName -eq $env:ComputerName } | ForEach-Object { 
        Remove-AzStorageSyncServerEndpoint -InputObject $_ 
    } 
}
```

#### <a name="unregister-the-server"></a>注销服务器
现在已召回所有数据并已从所有同步组中删除服务器，服务器可以进行注销。 

1. 在 Azure 门户中，导航到存储同步服务的“已注册的服务器”部分。
2. 右键单击要注销的服务器，然后单击“注销服务器”。

    ![注销服务器](media/storage-sync-files-server-registration/unregister-server-1.png)

## <a name="ensuring-azure-file-sync-is-a-good-neighbor-in-your-datacenter"></a>确保 Azure 文件同步在数据中心运作良好 
由于 Azure 文件同步不是在数据中心运行的唯一服务，因此建议限制 Azure 文件同步的网络和存储使用情况。

> [!Important]  
> 限制设置过低将影响 Azure 文件同步同步和召回的性能。

### <a name="set-azure-file-sync-network-limits"></a>设置 Azure 文件同步网络限制
可通过使用 `StorageSyncNetworkLimit` cmdlet 限制 Azure 文件同步的网络利用率。

> [!Note]  
> 访问分层文件或使用 Invoke-StorageSyncFileRecall cmdlet 时，网络限制不适用。

例如，可以创建新的中止值来确保 Azure 文件同步在工作周的上午 9 点到下午 5 点（17:00 点）之间不超过 10 Mbps： 

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
New-StorageSyncNetworkLimit -Day Monday, Tuesday, Wednesday, Thursday, Friday -StartHour 9 -EndHour 17 -LimitKbps 10000
```

可以使用以下 cmdlet 查看限制：

```powershell
Get-StorageSyncNetworkLimit # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

若要删除网络限制，请使用 `Remove-StorageSyncNetworkLimit`。 例如，可使用以下命令删除所有网络限制：

```powershell
Get-StorageSyncNetworkLimit | ForEach-Object { Remove-StorageSyncNetworkLimit -Id $_.Id } # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

### <a name="use-windows-server-storage-qos"></a>使用 Windows Server 存储 QoS 
如果 Azure 文件同步在 Windows Server 虚拟主机上运行的虚拟机中进行托管，可以使用存储 QoS（存储服务质量）来调整存储 IO 消耗量。 存储 QoS 策略可以设置为最大值（或限制，如上述的 StorageSyncNetwork 限制）或设置为最小值（或预留）。 如果设置为最小值而非最大值，则允许 Azure 文件同步在没有其他工作负载使用的情况下突发使用可用的存储宽带。 有关详细信息，请参阅[存储服务质量](https://docs.microsoft.com/windows-server/storage/storage-qos/storage-qos-overview)。

## <a name="see-also"></a>另请参阅
- [规划 Azure 文件同步部署](storage-sync-files-planning.md)
- [部署 Azure 文件同步](storage-sync-files-deployment-guide.md)
- [监视 Azure 文件同步](storage-sync-files-monitoring.md)
- [对 Azure 文件同步进行故障排除](storage-sync-files-troubleshoot.md)
