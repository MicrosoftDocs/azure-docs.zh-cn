---
title: 对 Azure 文件同步进行故障排除 | Microsoft Docs
description: 对 Azure 文件同步的常见问题进行故障排除
services: storage
author: jeffpatt24
ms.service: storage
ms.topic: article
ms.date: 01/31/2019
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: 2893960c3351b1f8a5caf0c69ca961851528007d
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510840"
---
# <a name="troubleshoot-azure-file-sync"></a>对 Azure 文件同步进行故障排除
使用 Azure 文件同步，即可将组织的文件共享集中在 Azure 文件中，同时又不失本地文件服务器的灵活性、性能和兼容性。 Azure 文件同步可将 Windows Server 转换为 Azure 文件共享的快速缓存。 可以使用 Windows Server 上可用的任意协议本地访问数据，包括 SMB、NFS 和 FTPS。 并且可以根据需要在世界各地具有多个缓存。

本文旨在帮助排查和解决在 Azure 文件同步部署中可能遇到的问题。 此外，还介绍了在需要对问题进行更深入调查的情况下，如何从系统收集重要日志。 如果本文未能涵盖你的问题，欢迎通过以下渠道联系我们（以升序排列）：

1. [Azure 存储论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)。
2. [Azure 文件 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files)。
3. Microsoft 支持部门。 若要创建新的支持请求，请在 Azure 门户中的“帮助”选项卡上，选择“帮助和支持”按钮，然后选择“新建支持请求”。

## <a name="im-having-an-issue-with-azure-file-sync-on-my-server-sync-cloud-tiering-etc-should-i-remove-and-recreate-my-server-endpoint"></a>我在服务器上遇到 Azure 文件同步问题（同步、云分层等）。 是否应删除并重新创建服务器终结点？
[!INCLUDE [storage-sync-files-remove-server-endpoint](../../../includes/storage-sync-files-remove-server-endpoint.md)]

## <a name="agent-installation-and-server-registration"></a>代理安装和服务器注册
<a id="agent-installation-failures"></a>**排查代理安装失败问题**  
如果 Azure 文件同步代理安装失败，请在安装代理的过程中，在权限提升的命令提示符下运行以下命令，以启用日志记录：

```
StorageSyncAgent.msi /l*v AFSInstaller.log
```

查看 installer.log 确定安装失败的原因。

<a id="agent-installation-on-DC"></a>**Active Directory 域控制器上的代理安装失败**  
如果尝试在 Active Directory 域控制器上安装同步代理，且该控制器中 PDC 角色所有者位于 Windows Server 2008 R2 上或更低的 OS 版本上，则可能遇到同步代理安装失败的问题。

要解决此问题，请将 PDC 角色转移到另一运行 Windows Server 2012 R2 或更高版本的域控制器，然后安装同步。

<a id="server-registration-missing"></a>**服务器未在 Azure 门户中的“已注册的服务器”下列出**  
如果对于存储同步服务，服务器未在“已注册的服务器”下列出：
1. 登录到要注册的服务器。
2. 打开文件资源管理器，然后转到存储同步代理安装目录（默认位置为 C:\Program Files\Azure\StorageSyncAgent）。 
3. 运行 ServerRegistration.exe 并完成向导中的操作，将服务器注册到存储同步服务。

<a id="server-already-registered"></a>**服务器注册在安装 Azure 文件同步代理期间显示以下消息：“此服务器已注册”** 

![包含“服务器已注册”错误消息的服务器注册对话框屏幕截图](media/storage-sync-files-troubleshoot/server-registration-1.png)

如果以前向存储同步服务注册了服务器，则会显示此消息。 若要从当前存储同步服务中注销服务器并向新存储同步服务进行注册，请完成[从 Azure 文件同步中注销服务器](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service)中所述的步骤。

如果存储同步服务中的“已注册服务器”下未列出该服务器，请在想要注销的服务器上运行以下 PowerShell 命令：

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Reset-StorageSyncServer
```

> [!Note]  
> 如果该服务器是群集的一部分，可以使用可选的 *Reset-StorageSyncServer -CleanClusterRegistration* 参数来同时删除群集注册。

<a id="web-site-not-trusted"></a>**注册服务器时，出现许多“网站不受信任”的响应。为什么？**  
如果在服务器注册过程中启用了“Internet Explorer 增强的安全性”策略，则会出现此问题。 有关如何以正确的方式禁用“Internet Explorer 增强的安全性”策略的详细信息，请参阅[准备 Windows Server 以使用 Azure 文件同步](storage-sync-files-deployment-guide.md#prepare-windows-server-to-use-with-azure-file-sync)，以及[如何部署 Azure 文件同步](storage-sync-files-deployment-guide.md)。

## <a name="sync-group-management"></a>同步组管理
<a id="cloud-endpoint-using-share"></a>**云终结点创建失败，并出现以下错误：“指定的 Azure 文件共享已被其他 CloudEndpoint 使用”**  
如果 Azure 文件共享已被其他云终结点使用，则会出现此问题。 

如果看到此消息，并且 Azure 文件共享当前未被云终结点使用，请完成以下步骤清除 Azure 文件共享上的 Azure 文件同步元数据：

> [!Warning]  
> 删除云终结点当前正在使用的 Azure 文件共享上的元数据会导致 Azure 文件同步操作失败。 

1. 在 Azure 门户中，转到 Azure 文件共享。  
2. 右键单击该 Azure 文件共享，并选择“编辑元数据”。
3. 右键单击“SyncService”，并选择“删除”。

<a id="cloud-endpoint-authfailed"></a>**云终结点创建失败，并出现以下错误：“AuthorizationFailed”**  
当用户帐户缺少足够权限来创建云终结点时，将出现此问题。 

要创建云终结点，用户帐户必须具有下列 Microsoft 授权权限：  
* 读取：获取角色定义
* 写入：创建或更新自定义角色定义
* 读取：获取角色分配
* 写入：创建角色分配

以下内置角色具有所需的 Microsoft 授权权限：  
* 所有者
* 用户访问管理员

若要确定自己的用户帐户角色否具有适当的权限，请执行以下操作：  
1. 在 Azure 门户中，选择“资源组”。
2. 选择存储帐户所在的资源组，再选择“访问控制(IAM)”。
3. 选择“角色分配”选项卡。
4. 为用户帐户选择**角色**（例如“所有者”或“参与者”）。
5. 在“资源提供程序”列表中，选择“Microsoft 授权”。 
    * “角色分配”应具有“读取”和“写入”权限。
    * “角色定义”应具有“读取”和“写入”权限。

<a id="server-endpoint-createjobfailed"></a>**服务器终结点创建失败，并出现以下错误：“MgmtServerJobFailed”（错误代码：-2134375898）**  
如果服务器终结点路径位于系统卷上并启用了云分层，则会出现此问题。 系统卷上不支持云分层。 要在系统卷上创建服务器终结点，请在创建服务器终结点时禁用云分层。

<a id="server-endpoint-deletejobexpired"></a>**服务器终结点删除失败，并出现以下错误：“MgmtServerJobExpired”**                
如果服务器处于脱机状态或无网络连接，则会出现此问题。 如果服务器不再可用，请在门户中注销要删除服务器终结点的服务器。 要删除服务器终结点，请按照[使用 Azure 文件同步注销服务器](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service)中所述的步骤操作。

<a id="server-endpoint-provisioningfailed"></a>**无法打开服务器终结点属性页或更新云分层策略**  
如果服务器终结点上的管理操作失败，则可能出现此问题。 如果未在 Azure 门户中打开服务器终结点属性页，则在服务器中使用 PowerShell 命令更新服务器终结点可修复此问题。 

```powershell
# Get the server endpoint id based on the server endpoint DisplayName property
Get-AzStorageSyncServerEndpoint `
    -ResourceGroupName myrgname `
    -StorageSyncServiceName storagesvcname `
    -SyncGroupName mysyncgroup | `
Tee-Object -Variable serverEndpoint

# Update the free space percent policy for the server endpoint
Set-AzStorageSyncServerEndpoint `
    -InputObject $serverEndpoint
    -CloudTiering `
    -VolumeFreeSpacePercent 60
```
<a id="server-endpoint-noactivity"></a>**服务器终结点的运行状态为“无活动”或“挂起”，已注册服务器边栏选项卡上的服务器状态为“显示脱机”**  

如果存储同步监视器进程未运行，或者服务器由于代理或防火墙而无法与 Azure 文件同步服务通信，则可能会发生此问题。

若要解决此问题，请执行以下步骤：

1. 在服务器上打开任务管理器，并验证存储同步监视器 (AzureStorageSyncMonitor.exe) 进程是否正在运行。 如果该进程未运行，请首先尝试重启服务器。 如果重启服务器无法解决此问题，请升级到最新的 Azure 文件同步[代理版本](https://docs.microsoft.com/azure/storage/files/storage-files-release-notes)。
2. 验证是否正确配置了防火墙和代理设置：
    - 如果服务器位于防火墙后面，请验证端口 443 是否允许出站。 如果防火墙限制到特定域的流量，请确认可以访问防火墙[文档](https://docs.microsoft.com/azure/storage/files/storage-sync-files-firewall-and-proxy#firewall)中列出的域。
    - 如果服务器位于代理后面，请按照代理[文档](https://docs.microsoft.com/azure/storage/files/storage-sync-files-firewall-and-proxy#proxy)中的步骤配置适用于整个计算机或特定于应用的代理设置。

<a id="endpoint-noactivity-sync"></a>服务器终结点的运行状态为“无活动”，已注册服务器边栏选项卡上的服务器状态为“脱机”  

服务器终结点运行状态为“无活动”的意味着服务器终结点在过去两小时内未记录同步活动。

服务器终结点可能会出于以下原因而未记录同步活动：

- 安装了代理版本 4.3.0.0 或之前版本，且服务器具有活动的 VSS 同步会话 (SnapshotSync)。 某个服务器终结点的 VSS 同步会话处于活动状态时，在 VSS 同步会话完成之前，同一卷上的其他服务器终结点无法启动同步会话。 若要解决此问题，请安装代理版本 5.0.2.0 或更高版本，这些版本在 VSS 同步会话时处于活动状态时支持在一个卷上进行多个服务器终结点同步。

    若要检查服务器上当前的同步活动，请参阅[如何监视当前同步会话的进度？](#how-do-i-monitor-the-progress-of-a-current-sync-session)。

- 服务器已达到最大并发同步会话数。 
    - 代理版本 4.x 及更高版本：限制因可用系统资源而异。
    - 代理版本 3.x：每个处理器 2 个活动的同步会话，或每个服务器最多 8 个活动的同步会话。

> [!Note]  
> 如果已注册服务器边栏选项卡上的服务器状态为“显示脱机”，请执行[服务终结点运行状态为“无活动”或“挂起”，已注册服务器边栏选项卡上的服务器状态为“显示脱机”](#server-endpoint-noactivity)部分中记录的步骤。

## <a name="sync"></a>同步
<a id="afs-change-detection"></a>**我通过 SMB 或门户在 Azure 文件共享中直接创建了一个文件，该文件同步到同步组中的服务器需要多长时间？**  
[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="serverendpoint-pending"></a>**服务器终结点运行状况有多个小时处于挂起状态**  
如果创建云终结点并使用包含数据的 Azure 文件共享，则可能会出现此问题。 在云与服务器终结点之间同步文件之前，扫描 Azure 文件共享中的更改的更改枚举作业必须已完成。 完成该作业所需的时间取决于 Azure 文件共享中命名空间的大小。 更改枚举作业完成后，服务器终结点运行状况应会更新。

### <a id="broken-sync"></a>如何监视同步运行状况？
# <a name="portaltabportal1"></a>[门户](#tab/portal1)
在每个同步组中，可以向下钻取到单个服务器终结点，以查看最后几个已完成的同步会话的状态。 如果出现绿色的“运行状况”列，并且“文件未同步”的值为 0，则表示同步按预期工作。 否则，请参阅下面的常见同步错误列表，以及文件不同步问题的处理方法。 

![Azure 门户的屏幕截图](media/storage-sync-files-troubleshoot/portal-sync-health.png)

# <a name="servertabserver"></a>[服务器](#tab/server)
转到服务器的遥测日志（可在事件查看器中通过 `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry` 找到）。 事件 9102 对应于已完成的同步会话；有关最新同步状态，请查找 ID 为 9102 的最新事件。 SyncDirection 会告知此会话是上传还是下载。 如果 HResult 为 0，则表示同步会话已成功。 如果 HResult 不为 0，则表示同步期间出错；有关常见错误的列表，请参阅下文。 如果 PerItemErrorCount 大于 0，则表示某些文件或文件夹未正确同步。 有可能 HResult 为 0，但 PerItemErrorCount 大于 0。

下面是成功上传的示例。 为简洁起见，下面只列出了每个 9102 事件中包含的某些值。 

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: 0, 
SyncFileCount: 2, SyncDirectoryCount: 0,
AppliedFileCount: 2, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0,
TransferredFiles: 2, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

相反，不成功的上传可能如下所示：

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: -2134364065,
SyncFileCount: 0, SyncDirectoryCount: 0, 
AppliedFileCount: 0, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0, 
TransferredFiles: 0, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

有时，同步会话虽然总体失败或者出现非零值的 PerItemErrorCount，但仍能继续进行，并且某些文件能够成功同步。 可以在 Applied* 字段（AppliedFileCount、AppliedDirCount、AppliedTombstoneCount 和 AppliedSizeBytes）中看到这种状态，其中告知了有多少个会话成功。 如果在某一行中看到多个同步会话失败，但 Applied* 计数增加，则应该提供一定的同步时间以便重试，然后再开具支持票证。

---

### <a name="how-do-i-monitor-the-progress-of-a-current-sync-session"></a>如何监视当前同步会话的进度？
# <a name="portaltabportal1"></a>[门户](#tab/portal1)
在同步组中，转到有问题的服务器终结点，并在“同步活动”部分查看当前同步会话中已上传或下载的文件计数。 请注意，此状态会延迟大约 5 分钟，如果同步会话足够小，可在此时间段内完成，则可能不会在门户中报告此会话。 

# <a name="servertabserver"></a>[服务器](#tab/server)
在服务器上的遥测日志中查看最近的 9302 事件（在事件查看器中，转到“应用程序和服务日志\Microsoft\FileSync\Agent\Telemetry”）。 此事件指示当前同步会话的状态。 TotalItemCount 表示要同步多少个文件，AppliedItemCount 表示到目前为止已同步的文件数，PerItemErrorCount 表示同步失败的文件数（请参阅下文了解如何处理此问题）。

```
Replica Sync Progress. 
ServerEndpointName: <CI>sename</CI>, SyncGroupName: <CI>sgname</CI>, ReplicaName: <CI>rname</CI>, 
SyncDirection: Upload, CorrelationId: {AB4BA07D-5B5C-461D-AAE6-4ED724762B65}. 
AppliedItemCount: 172473, TotalItemCount: 624196. AppliedBytes: 51473711577, 
TotalBytes: 293363829906. 
AreTotalCountsFinal: true. 
PerItemErrorCount: 1006.
```
---

### <a name="how-do-i-know-if-my-servers-are-in-sync-with-each-other"></a>如何知道我的服务器是否已彼此同步？
# <a name="portaltabportal1"></a>[门户](#tab/portal1)
对于给定同步组中的每个服务器，请确保：
- 上传和下载会话的“上次尝试同步”时间戳是最近的时间。
- 上传和下载会话的状态为绿色。
- “同步活动”字段显示的要同步的剩余文件数很少或者为零。
- 上传和下载会话的“文件未同步”字段值为 0。

# <a name="servertabserver"></a>[服务器](#tab/server)
查看由每个服务器的遥测事件日志中的 9102 事件标记的已完成同步会话（在事件查看器中，转到 `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry`）。 

1. 在任何给定的服务器上，需要确保最近的上传和下载会话已成功完成。 为此，请检查上传和下载操作的 HResult 与 PerItemErrorCount 是否为 0（SyncDirection 字段会指示给定的会话是上传还下载会话）。 请注意，如果未看到最近完成的同步会话，原因有可能是同步会话当前正在进行；如果刚刚添加或修改了大量数据，则这种情况符合预期。
2. 如果服务器在云中完全处于最新状态，并且不包含要朝任一方向同步的更改，则你会看到空的同步会话。 这些状态由上传和下载事件指示。在这些事件中，Sync* 字段（SyncFileCount、SyncDirCount、SyncTombstoneCount 和 SyncSizeBytes）为 0，表示没有要同步的内容。请注意，在数据变动率较高的服务器上，可能不会出现这些空的同步会话，因为始终有新的内容需要同步。如果没有任何同步活动，同步会话应每隔 30 分钟发生一次。 
3. 如果所有服务器在云中处于最新状态，则意味着这些服务器的最近上传和下载会话是空的同步会话，此时可以合理地确定系统在整体上是同步的。 
    
请注意，如果直接在 Azure 文件共享中进行更改，则在更改枚举运行之前（每隔 24 小时发生一次），Azure 文件同步不会检测此项更改。 有可能服务器在云中处于最新状态，但事实上它缺少 Azure 文件共享中所做的最新更改。 

---

### <a name="how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing"></a>如何确定是否有特定的文件或文件夹未同步？
对于任何给定的同步会话，如果服务器上的 PerItemErrorCount 或门户中“文件未同步”计数大于 0，则表示某些项未能同步。文件和文件夹的某些特征导致它们无法同步。 这些特征可能是永久性的，需要采取明确的措施才能恢复同步，例如，从文件或文件夹名称中删除不支持的字符。 它们也可能是暂时性的，也就是说，该文件或文件夹可自动恢复同步；例如，包含开放句柄的文件在关闭后可自动恢复同步。 当 Azure 文件同步引擎检测到此类问题时，会生成错误日志，可以分析这些日志以列出当前未正确同步的项。

若要查看这些错误，请运行 **FileSyncErrorsReport.ps1** PowerShell 脚本（位于 Azure 文件同步代理的代理安装目录中），以识别由于包含开放句柄、不支持的字符或存在其他问题而未能同步的文件。 ItemPath 字段告知相对于根同步目录的文件位置。 请参阅以下常见同步错误列表获取补救步骤。

#### <a name="troubleshooting-per-filedirectory-sync-errors"></a>根据文件/目录同步错误进行故障排除
**ItemResults 日志 - 按项列出的同步错误**  

| HRESULT | HRESULT（十进制） | 错误字符串 | 问题 | 补救 |
|---------|-------------------|--------------|-------|-------------|
| 0x80c80207 | -2134375929 | ECS_E_SYNC_CONSTRAINT_CONFLICT | 由于尚未同步某个相关的文件夹，无法同步文件或目录更改。 在同步相关的更改后，此项将会同步。 | 无需采取措施。 |
| 0x7b | 123 | ERROR_INVALID_NAME | 文件或目录名称无效。 | 重命名有问题的文件或目录。 有关详细信息，请参阅[处理不支持的字符](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#handling-unsupported-characters)。 |
| 0x8007007b | -2147024773 | STIERR_INVALID_DEVICE_NAME | 文件或目录名称无效。 | 重命名有问题的文件或目录。 有关详细信息，请参阅[处理不支持的字符](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#handling-unsupported-characters)。 |
| 0x80c80018 | -2134376424 | ECS_E_SYNC_FILE_IN_USE | 文件已被使用，因此无法将其同步。 不再使用该文件时，会将其同步。 | 无需采取措施。 Azure 文件同步每天在服务器上创建临时 VSS 快照一次，以同步包含开放句柄的文件。 |
| 0x80c8031d | -2134375651 | ECS_E_CONCURRENCY_CHECK_FAILED | 文件已更改，但同步尚未检测到此项更改。检测到此项更改后，同步将会恢复。 | 无需采取措施。 |
| 0x80c8603e | -2134351810 | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED | 无法同步该文件，因为已达到 Azure 文件共享限制。 | 要解决此问题，请参阅疑难解答指南中的[达到 Azure 文件共享存储限制](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#-2134351810)部分。 |
| 0x80070005 | -2147024891 | E_ACCESSDENIED | 此错误可能由于以下原因： 文件已由不受支持解决方案 （如 NTFS EFS) 加密、 文件有挂起状态的删除或文件位于 DFS-R 只读复制文件夹 | 如果文件已加密不受支持的解决方案，解密文件，并使用支持的加密解决方案。 有关支持解决方案的列表，请参阅计划指南中的[加密解决方案](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#encryption-solutions)部分。 如果文件处于删除待定状态，一旦关闭所有打开的文件句柄，将删除该文件。 如果在 DFS-R 只读复制文件夹位于该文件，则 Azure 文件同步不支持服务器终结点上 DFS-R 只读复制文件夹。 请参阅[规划指南](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning#distributed-file-system-dfs)有关详细信息。
| 0x20 | 32 | ERROR_SHARING_VIOLATION | 文件已被使用，因此无法将其同步。 不再使用该文件时，会将其同步。 | 无需采取措施。 |
| 0x80c80017 | -2134376425 | ECS_E_SYNC_OPLOCK_BROKEN | 同步期间更改了文件，因此需要重新同步。 | 无需采取措施。 |

#### <a name="handling-unsupported-characters"></a>处理不受支持的字符
如果 FileSyncErrorsReport.ps1 PowerShell 脚本显示不受支持的字符导致失败（错误代码 0x7b 和 0x8007007b），请从相关的文件名中删除或重命名错误的字符。 PowerShell 可能会以问号或空框的形式列显这些字符，因为其中的大多数字符没有标准的视觉编码。 [评估工具](storage-sync-files-planning.md#evaluation-tool)可用于标识不受支持的字符。

下表包含 Azure 文件同步尚不支持的所有 Unicode 字符。

| 字符集 | 字符计数 |
|---------------|-----------------|
| <ul><li>0x0000009D（osc 操作系统命令）</li><li>0x00000090（dcs 设备控制字符串）</li><li>0x0000008F (ss3 single shift three)</li><li>0x00000081（高八位字节预设）</li><li>0x0000007F（del 删除）</li><li>0x0000008D（ri 反向换行符）</li></ul> | 6 |
| 0x0000FDD0 - 0x0000FDEF（阿拉伯语显示格式 a） | 32 |
| 0x0000FFF0-0x0000FFFF（特殊字符） | 16 |
| <ul><li>0x0001FFFE - 0x0001FFFF = 2（非字符）</li><li>0x0002FFFE - 0x0002FFFF = 2（非字符）</li><li>0x0003FFFE - 0x0003FFFF = 2（非字符）</li><li>0x0004FFFE - 0x0004FFFF = 2（非字符）</li><li>0x0005FFFE - 0x0005FFFF = 2（非字符）</li><li>0x0006FFFE - 0x0006FFFF = 2（非字符）</li><li>0x0007FFFE - 0x0007FFFF = 2（非字符）</li><li>0x0008FFFE - 0x0008FFFF = 2（非字符）</li><li>0x0009FFFE - 0x0009FFFF = 2（非字符）</li><li>0x000AFFFE - 0x000AFFFF = 2（非字符）</li><li>0x000BFFFE - 0x000BFFFF = 2（非字符）</li><li>0x000CFFFE - 0x000CFFFF = 2（非字符）</li><li>0x000DFFFE - 0x000DFFFF = 2（非字符）</li><li>0x000EFFFE - 0x000EFFFF = 2（未定义）</li><li>0x000FFFFE - 0x000FFFFF = 2（补充专用区域）</li></ul> | 30 |
| 0x0010FFFE、0x0010FFFF | 2 |

### <a name="common-sync-errors"></a>常见同步错误
<a id="-2147023673"></a>**同步会话已取消。**  

| | |
|-|-|
| **HRESULT** | 0x800704c7 |
| **HRESULT（十进制）** | -2147023673 | 
| **错误字符串** | ERROR_CANCELLED |
| **所需的补救措施** | “否” |

同步会话可能出于各种原因失败，包括服务器正在重启或更新、VSS 快照，等等。尽管此错误看起来需要保持跟进，但可以放心地将其忽略，除非它持续了好几个小时。

<a id="-2147012889"></a>**无法与服务建立连接。**    

| | |
|-|-|
| **HRESULT** | 0x80072ee7 |
| **HRESULT（十进制）** | -2147012889 | 
| **错误字符串** | WININET_E_NAME_NOT_RESOLVED |
| **所需的补救措施** | “是” |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134376372"></a>**服务限制了用户请求。**  

| | |
|-|-|
| **HRESULT** | 0x80c8004c |
| **HRESULT（十进制）** | -2134376372 |
| **错误字符串** | ECS_E_USER_REQUEST_THROTTLED |
| **所需的补救措施** | “否” |

无需采取措施；服务器会重试。 如果此错误持续了几个小时，请创建支持请求。

<a id="-2134364065"></a>**同步无法访问云终结点中指定的 Azure 文件共享。**  

| | |
|-|-|
| **HRESULT** | 0x80c8305f |
| **HRESULT（十进制）** | -2134364065 |
| **错误字符串** | ECS_E_CANNOT_ACCESS_EXTERNAL_STORAGE_ACCOUNT |
| **所需的补救措施** | “是” |

之所以发生此错误，是因为 Azure 文件同步代理无法访问 Azure 文件共享。无法访问的可能原因是 Azure 文件共享或托管它的存储帐户不再存在。 可以执行以下步骤来排查此错误：

1. [验证存储帐户是否存在。](#troubleshoot-storage-account)
2. [检查并确保存储帐户不包含任何网络规则。](#troubleshoot-network-rules)
3. [确保 Azure 文件共享存在。](#troubleshoot-azure-file-share)
4. [确保 Azure 文件同步有权访问存储帐户。](#troubleshoot-rbac)

<a id="-2134364064"></a><a id="cannot-resolve-storage"></a>**无法解析所用的存储帐户名。**  

| | |
|-|-|
| **HRESULT** | 0x80C83060 |
| **HRESULT（十进制）** | -2134364064 |
| **错误字符串** | ECS_E_STORAGE_ACCOUNT_NAME_UNRESOLVED |
| **所需的补救措施** | “是” |

1. 检查是否可从服务器解析存储 DNS 名称。

    ```powershell
    Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 443
    ```
2. [验证存储帐户是否存在。](#troubleshoot-storage-account)
3. [检查并确保存储帐户不包含任何网络规则。](#troubleshoot-network-rules)

<a id="-1906441138"></a>**由于同步数据库出现问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x8e5e044e |
| **HRESULT（十进制）** | -1906441138 |
| **错误字符串** | JET_errWriteConflict |
| **所需的补救措施** | “是” |

如果 Azure 文件同步使用的内部数据库出现问题，则会发生此错误。出现此问题时，请创建支持请求，到时我们将与你取得联系，并帮助解决此问题。

<a id="-2134364053"></a>服务器上安装的 Azure 文件同步代理版本不受支持。  

| | |
|-|-|
| **HRESULT** | 0x80C8306B |
| **HRESULT（十进制）** | -2134364053 |
| **错误字符串** | ECS_E_AGENT_VERSION_BLOCKED |
| **所需的补救措施** | “是” |

如果服务器上安装的 Azure 文件同步代理版本不受支持，则会出现此错误。 若要解决此问题，请[升级]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#upgrade-paths)到[受支持的代理版本]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#supported-versions)。

<a id="-2134351810"></a>**达到了 Azure 文件共享存储限制。**  

| | |
|-|-|
| **HRESULT** | 0x80c8603e |
| **HRESULT（十进制）** | -2134351810 |
| **错误字符串** | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED |
| **所需的补救措施** | “是” |

达到 Azure 文件共享存储限制时，将发生此错误。如果对 Azure 文件共享应用了配额，或者用量超过了 Azure 文件共享的限制，则可能会发生这种情况。 有关详细信息，请参阅 [Azure 文件共享的当前限制](storage-files-scale-targets.md)。

1. 导航到存储同步服务中的同步组。
2. 选择同步组中的云终结点。
3. 记下打开的窗格中的 Azure 文件共享名称。
4. 选择链接的存储帐户。 如果此链接不起作用，则表示引用的存储帐户已删除。

    ![显示云终结点详细信息窗格（包含存储帐户的链接）的屏幕截图。](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

5. 选择“文件”查看文件共享列表。
6. 单击云终结点引用的 Azure 文件共享所在行末尾的三点图标。
7. 检查“用量”是否低于“配额”。 请注意，除非已指定备用配额，否则配额将与 [Azure 文件共享的最大大小](storage-files-scale-targets.md)匹配。

    ![Azure 文件共享属性的屏幕截图。](media/storage-sync-files-troubleshoot/file-share-limit-reached-1.png)

如果共享已满且未设置配额，则解决此问题的可行方法之一是，将当前服务器终结点的每个子文件夹放入其自己的单独同步组中自己的服务器终结点内。 这样，每个子文件夹就会同步到单个 Azure 文件共享。

<a id="-2134351824"></a>**找不到 Azure 文件共享。**  

| | |
|-|-|
| **HRESULT** | 0x80c86030 |
| **HRESULT（十进制）** | -2134351824 |
| **错误字符串** | ECS_E_AZURE_FILE_SHARE_NOT_FOUND |
| **所需的补救措施** | “是” |

当 Azure 文件共享不可访问时，将发生此错误。 故障排除：

1. [验证存储帐户是否存在。](#troubleshoot-storage-account)
2. [确保 Azure 文件共享存在。](#troubleshoot-azure-file-share)

如果 Azure 文件共享已删除，则需要创建新的文件共享，然后重新创建同步组。 

<a id="-2134364042"></a>**此 Azure 订阅暂停时，同步将会暂停。**  

| | |
|-|-|
| **HRESULT** | 0x80C83076 |
| **HRESULT（十进制）** | -2134364042 |
| **错误字符串** | ECS_E_SYNC_BLOCKED_ON_SUSPENDED_SUBSCRIPTION |
| **所需的补救措施** | “是” |

当 Azure 订阅暂停时，将发生此错误。 还原 Azure 订阅后，会重新启用同步。 有关详细信息，请参阅[为何禁用我的 Azure 订阅？如何重新激活它？](../../billing/billing-subscription-become-disable.md)。

<a id="-2134364052"></a>**为存储帐户配置了防火墙或虚拟网络。**  

| | |
|-|-|
| **HRESULT** | 0x80c8306c |
| **HRESULT（十进制）** | -2134364052 |
| **错误字符串** | ECS_E_MGMT_STORAGEACLSNOTSUPPORTED |
| **所需的补救措施** | “是” |

如果由于设置了存储帐户防火墙或者存储帐户属于虚拟网络而导致 Azure 文件共享不可访问，则会发生此错误。 Azure 文件同步尚不支持此功能。 故障排除：

1. [验证存储帐户是否存在。](#troubleshoot-storage-account)
2. [检查并确保存储帐户不包含任何网络规则。](#troubleshoot-network-rules)

删除这些规则即可解决此问题。 

<a id="-2134375911"></a>**由于同步数据库出现问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x80c80219 |
| **HRESULT（十进制）** | -2134375911 |
| **错误字符串** | ECS_E_SYNC_METADATA_WRITE_LOCK_TIMEOUT |
| **所需的补救措施** | “否” |

此错误通常会自行解决，其原因包括：

* 同步组中服务器之间的文件更改较多。
* 单个文件和目录中出现大量的错误。

如果此错误持续了几个小时，请创建支持请求，我们将与你取得联系，并帮助解决此问题。

<a id="-2146762487"></a>**服务器无法建立安全连接。云服务收到意外的证书。**  

| | |
|-|-|
| **HRESULT** | 0x800b0109 |
| **HRESULT（十进制）** | -2146762487 |
| **错误字符串** | CERT_E_UNTRUSTEDROOT |
| **所需的补救措施** | “是” |

如果组织使用 SSL 终止代理，或恶意实体正在截获服务器与 Azure 文件同步服务之间的通信，则会发生此错误。 如果确定这是预期行为（因为组织使用 SSL 终止代理），请跳过证书验证并改用注册表覆盖。

1. 创建 SkipVerifyingPinnedRootCertificate 注册表值。

    ```powershell
    New-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Azure\StorageSync -Name SkipVerifyingPinnedRootCertificate -PropertyType DWORD -Value 1
    ```

2. 在注册的服务器上重启同步服务。

    ```powershell
    Restart-Service -Name FileSyncSvc -Force
    ```

通过设置此注册表值，Azure 文件同步代理将在服务器和云服务之间传输数据时接受本地受信任的任何 SSL 证书。

<a id="-2147012894"></a>**无法与服务建立连接。**  

| | |
|-|-|
| **HRESULT** | 0x80072ee2 |
| **HRESULT（十进制）** | -2147012894 |
| **错误字符串** | WININET_E_TIMEOUT |
| **所需的补救措施** | “是” |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134375680"></a>**由于身份验证出现问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x80c80300 |
| **HRESULT（十进制）** | -2134375680 |
| **错误字符串** | ECS_E_SERVER_CREDENTIAL_NEEDED |
| **所需的补救措施** | “是” |

此错误可由以下原因引起：

- 服务器时间不正确
- 服务器终结点删除失败
- 用于身份验证的证书已过期。 
    若要检查证书是否已过期，请执行以下步骤：  
    1. 打开“证书”MMC 管理单元，选择“计算机帐户”，然后导航到“证书(本地计算机)”\“个人”\“证书”。
    2. 检查客户端身份验证证书是否过期。

如果服务器时间正确，请执行以下步骤来解决此问题：

1. 验证是否已安装了 Azure 文件同步代理版本 4.0.1.0 或更高版本。
2. 在服务器上运行以下 PowerShell 命令：

    ```powershell
    Reset-AzStorageSyncServerCertificate -ResourceGroupName <string> -StorageSyncServiceName <string>
    ```

<a id="-1906441711"></a><a id="-2134375654"></a><a id="doesnt-have-enough-free-space"></a>**服务器终结点所在卷的磁盘空间不足。**  

| | |
|-|-|
| **HRESULT** | 0x8e5e0211 |
| **HRESULT（十进制）** | -1906441711 |
| **错误字符串** | JET_errLogDiskFull |
| **所需的补救措施** | “是” |
| | |
| **HRESULT** | 0x80c8031a |
| **HRESULT（十进制）** | -2134375654 |
| **错误字符串** | ECS_E_NOT_ENOUGH_LOCAL_STORAGE |
| **所需的补救措施** | “是” |

此错误的原因是卷已填满。 此错误的常见原因是服务器终结点外部的文件用尽了卷上的空间。 请通过添加更多的服务器终结点、将文件移到其他卷，或增大服务器终结点所在卷的大小，来释放卷上的空间。

<a id="-2134364145"></a><a id="replica-not-ready"></a>**服务尚未准备好与此服务器终结点同步。**  

| | |
|-|-|
| **HRESULT** | 0x80c8300f |
| **HRESULT（十进制）** | -2134364145 |
| **错误字符串** | ECS_E_REPLICA_NOT_READY |
| **所需的补救措施** | “否” |

此错误的原因是 Azure 文件共享上发生直接更改，并且更改检测正在进行。 更改检测完成后，将开始同步。

[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="-2134375877"></a><a id="-2134375908"></a><a id="-2134375853"></a>**由于许多单独的文件出现问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x80c8023b |
| **HRESULT（十进制）** | -2134364145 |
| **错误字符串** | ECS_E_SYNC_METADATA_KNOWLEDGE_SOFT_LIMIT_REACHED |
| **所需的补救措施** | “是” |
| | |
| **HRESULT** | 0x80c8021c |
| **HRESULT（十进制）** | -2134375908 |
| **错误字符串** | ECS_E_SYNC_METADATA_KNOWLEDGE_LIMIT_REACHED |
| **所需的补救措施** | “是” |
| | |
| **HRESULT** | 0x80c80253 |
| **HRESULT（十进制）** | -2134375853 |
| **错误字符串** | ECS_E_TOO_MANY_PER_ITEM_ERRORS |
| **所需的补救措施** | “是” |

如果发生许多的文件同步错误，同步会话可能无法开始。 <!-- To troubleshoot this state, see [Troubleshooting per file/directory sync errors]().-->

> [!NOTE]
> Azure 文件同步每天在服务器上创建临时 VSS 快照一次，以同步包含开放句柄的文件。

<a id="-2134376423"></a>**由于服务器终结点路径出现问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x80c80019 |
| **HRESULT（十进制）** | -2134376423 |
| **错误字符串** | ECS_E_SYNC_INVALID_PATH |
| **所需的补救措施** | “是” |

确保路径存在、位于本地 NTFS 卷上，且不是重新分析点或现有服务器终结点。

<a id="-2134375817"></a>同步失败，因为筛选器驱动程序版本与代理版本不兼容  

| | |
|-|-|
| **HRESULT** | 0x80C80277 |
| **HRESULT（十进制）** | -2134375817 |
| **错误字符串** | ECS_E_INCOMPATIBLE_FILTER_VERSION |
| **所需的补救措施** | “是” |

发生此错误的原因是加载的云分层筛选器驱动程序 (StorageSync.sys) 版本与 Storage Sync Agent (FileSyncSvc) 服务不兼容。 如果已升级 Azure 文件同步代理，请重启服务器以完成安装。 如果错误继续发生，请卸载代理，重启服务器并重新安装 Azure 文件同步代理。

<a id="-2134376373"></a>**服务当前不可用。**  

| | |
|-|-|
| **HRESULT** | 0x80c8004b |
| **HRESULT（十进制）** | -2134376373 |
| **错误字符串** | ECS_E_SERVICE_UNAVAILABLE |
| **所需的补救措施** | “否” |

此错误的原因是 Azure 文件同步服务不可用。 当 Azure 文件同步服务再次可用时，此错误将自行解决。

<a id="-2134375922"></a>**由于同步数据库出现暂时性的问题，同步失败。**  

| | |
|-|-|
| **HRESULT** | 0x80c8020e |
| **HRESULT（十进制）** | -2134375922 |
| **错误字符串** | ECS_E_SYNC_METADATA_WRITE_LEASE_LOST |
| **所需的补救措施** | “否” |

此错误的原因是同步数据库出现内部问题。 当 Azure 文件同步重试同步时，此错误将自行解决。 如果此错误持续了较长时间，请创建支持请求，到时我们将与你取得联系，并帮助解决此问题。

### <a name="common-troubleshooting-steps"></a>常见故障排除步骤
<a id="troubleshoot-storage-account"></a>**验证存储帐户是否存在。**  
# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)
1. 导航到存储同步服务中的同步组。
2. 选择同步组中的云终结点。
3. 记下打开的窗格中的 Azure 文件共享名称。
4. 选择链接的存储帐户。 如果此链接不起作用，则表示引用的存储帐户已删除。
    ![显示云终结点详细信息窗格（包含存储帐户的链接）的屏幕截图。](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
# Variables for you to populate based on your configuration
$region = "<Az_Region>"
$resourceGroup = "<RG_Name>"
$syncService = "<storage-sync-service>"
$syncGroup = "<sync-group>"

# Log into the Azure account
Connect-AzAccount

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = [System.String[]]@()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the " + `
        " selected Azure Region or the region is mistyped.")
}

# Check to ensure resource group exists
$resourceGroups = [System.String[]]@()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    throw [System.Exception]::new("The provided resource group $resourceGroup does not exist.")
}

# Check to make sure the provided Storage Sync Service
# exists.
$syncServices = [System.String[]]@()

Get-AzStorageSyncService -ResourceGroupName $resourceGroup | ForEach-Object {
    $syncServices += $_.StorageSyncServiceName
}

if ($syncServices -notcontains $syncService) {
    throw [System.Exception]::new("The provided Storage Sync Service $syncService does not exist.")
}

# Check to make sure the provided Sync Group exists
$syncGroups = [System.String[]]@()

Get-AzStorageSyncGroup -ResourceGroupName $resourceGroup -StorageSyncServiceName $syncService | ForEach-Object {
    $syncGroups += $_.SyncGroupName
}

if ($syncGroups -notcontains $syncGroup) {
    throw [System.Exception]::new("The provided sync group $syncGroup does not exist.")
}

# Get reference to cloud endpoint
$cloudEndpoint = Get-AzStorageSyncCloudEndpoint `
    -ResourceGroupName $resourceGroup `
    -StorageSyncServiceName $syncService `
    -SyncGroupName $syncGroup

# Get reference to storage account
$storageAccount = Get-AzStorageAccount | Where-Object { 
    $_.Id -eq $cloudEndpoint.StorageAccountResourceId
}

if ($storageAccount -eq $null) {
    throw [System.Exception]::new("The storage account referenced in the cloud endpoint does not exist.")
}
```
---

<a id="troubleshoot-network-rules"></a>**检查并确保存储帐户不包含任何网络规则。**  
# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)
1. 进入存储帐户后，在存储帐户的左侧选择“防火墙和虚拟网络”。
2. 在存储帐户中，“允许来自所有网络的访问”单选按钮应已选中。
    ![显示已禁用存储帐户防火墙和网络规则的屏幕截图。](media/storage-sync-files-troubleshoot/file-share-inaccessible-2.png)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
if ($storageAccount.NetworkRuleSet.DefaultAction -ne 
    [Microsoft.Azure.Commands.Management.Storage.Models.PSNetWorkRuleDefaultActionEnum]::Allow) {
    throw [System.Exception]::new("The storage account referenced contains network " + `
        "rules which are not currently supported by Azure File Sync.")
}
```
---

<a id="troubleshoot-azure-file-share"></a>**确保 Azure 文件共享存在。**  
# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)
1. 在左侧目录中单击“概述”，返回存储帐户主页。
2. 选择“文件”查看文件共享列表。
3. 检查云终结点引用的文件共享是否显示在文件共享列表中（在上述步骤 1 中应已记下此共享名称）。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $cloudEndpoint.AzureFileShareName -and
    $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    throw [System.Exception]::new("The Azure file share referenced by the cloud endpoint does not exist")
}
```
---

<a id="troubleshoot-rbac"></a>**确保 Azure 文件同步有权访问存储帐户。**  
# <a name="portaltabazure-portal"></a>[门户](#tab/azure-portal)
1. 在左侧的目录上单击“访问控制(IAM)”。
1. 单击“角色分配”选项卡以列出有权访问你的存储帐户的用户和应用程序（*服务主体*）。
1. 检查“混合文件同步服务”是否与“读取器和数据访问”角色一起显示在列表中。 

    ![存储帐户的访问控制选项卡中的混合文件同步服务服务主体的屏幕截图](media/storage-sync-files-troubleshoot/file-share-inaccessible-3.png)

    如果“混合文件同步服务”没有出现在列表中，请执行以下步骤：

    - 单击“添加”。
    - 在“角色”字段中，选择“读者和数据访问”。
    - 在“选择”字段中，键入“混合文件同步服务”，选择角色并单击“保存”。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell    
$role = Get-AzRoleAssignment -Scope $storageAccount.Id | Where-Object { $_.DisplayName -eq "Hybrid File Sync Service" }

if ($role -eq $null) {
    throw [System.Exception]::new("The storage account does not have the Azure File Sync " + `
                "service principal authorized to access the data within the " + ` 
                "referenced Azure file share.")
}
```
---

### <a name="how-do-i-prevent-users-from-creating-files-containing-unsupported-characters-on-the-server"></a>如何阻止用户在服务器上创建包含不受支持字符的文件？
可以使用[文件服务器资源管理器 (FSRM) 文件屏蔽](https://docs.microsoft.com/windows-server/storage/fsrm/file-screening-management)来阻止在服务器上创建名称中包含不受支持字符的文件。 可能需要使用 PowerShell 完成此操作，因为大多数不受支持的字符不可打印。因此首先需要将其十六进制表示形式强制转换为字符。

首先使用 [New-FsrmFileGroup cmdlet](https://docs.microsoft.com/powershell/module/fileserverresourcemanager/new-fsrmfilegroup) 创建 FSRM 文件组。 此示例定义了只包含两个不受支持字符的组，但你可以在文件组中包含任意数量的字符。

```powershell
New-FsrmFileGroup -Name "Unsupported characters" -IncludePattern @(("*"+[char]0x00000090+"*"),("*"+[char]0x0000008F+"*"))
```

定义 FSRM 文件组后，可以使用 New-FsrmFileScreen cmdlet 创建 FSRM 文件屏蔽。

```powershell
New-FsrmFileScreen -Path "E:\AFSdataset" -Description "Filter unsupported characters" -IncludeGroup "Unsupported characters"
```

> [!Important]  
> 请注意，只能使用文件屏蔽来阻止创建 Azure 文件同步不支持的字符。如果在其他方案中使用文件屏蔽，同步会持续尝试将 Azure 文件共享中的文件下载到服务器，并且会由于文件屏蔽而受到阻止，导致较高的传出数据。 

## <a name="cloud-tiering"></a>云分层 
云分层中存在两个故障路径：

- 文件可能无法层，这意味着，Azure 文件同步未成功尝试将某个文件分层到 Azure 文件。
- 文件可能无法撤回，这意味着，当某个用户尝试访问已分层的文件时，Azure 文件同步文件系统筛选器 (StorageSync.sys) 无法下载数据。

通过任一故障路径可能发生两个主要故障类：

- 云存储故障
    - 暂时性存储服务可用性问题。 有关详细信息，请参阅 [Azure 存储的服务级别协议 (SLA)](https://azure.microsoft.com/support/legal/sla/storage/v1_2/)。
    - Azure 文件共享不可访问。 删除仍属于同步组中的云终结点的 Azure 文件共享时，通常会发生此故障。
    - 存储帐户不可访问。 删除仍包含 Azure 文件共享的存储帐户，且该文件共享属于同步组中的云终结点时，通常会发生此故障。 
- 服务器故障 
  - 未加载 Azure 文件同步文件系统筛选器 (StorageSync.sys)。 为了响应分层/撤回请求，必须加载 Azure 文件同步文件系统筛选器。 可能会出于多种原因而未能加载该筛选器，但最常见的原因是管理员已手动将其卸载。 要使 Azure 文件同步正常运行，始终必须加载 Azure 文件同步文件系统筛选器。
  - 重分析点缺失、损坏或出现其他形式的中断。 重分析点是文件中的特殊数据结构，它由两个部分组成：
    1. 一个重分析标记，向操作系统指示 Azure 文件同步文件系统筛选器 (StorageSync.sys) 可能需要针对文件的 IO 执行某项操作。 
    2. 重分析数据，向文件系统筛选器指示关联云终结点（Azure 文件共享）上的文件的 URI。 
        
       重分析点损坏的最常见原因是管理员尝试修改标记或其数据。 
  - 网络连接问题。 若要分层或撤回文件，服务器必须已建立 Internet 连接。

以下部分说明如何排查云分层问题，并确定问题是云存储问题还是服务器问题。

<a id="monitor-tiering-activity"></a>**如何监视服务器上的分层活动**  
若要监视服务器上的分层活动，请使用遥测事件日志（位于“事件查看器”中的“应用程序和服务\Microsoft\FileSync\代理”下）中的事件 ID 9003、9016 和 9029。

- 事件 ID 9003 提供服务器终结点的错误分布情况。 例如错误总数、错误代码，等等。请注意，将为每个错误代码记录一个事件。
- 事件 ID 9016 提供卷的副本创建结果。 例如，可用空间百分比、会话中创建的文件副本数、无法创建副本的文件数，等等。
- 事件 ID 9029 提供服务器终结点的副本创建会话信息。 例如，会话中尝试的文件数、会话分层的文件数、已分层的文件数，等等。

<a id="monitor-recall-activity"></a>**如何监视服务器上的重新调用活动**  
若要监视服务器上的回调活动，请使用遥测事件日志（位于“事件查看器”中的“应用程序和服务\Microsoft\FileSync\代理”下）中的事件 ID 9005、9006、9009 和 9059。

- 事件 ID 9005 提供服务器终结点的重新调用可靠性。 例如，访问的唯一文件总数、访问失败的唯一文件总数，等等。
- 事件 ID 9006 提供服务器终结点的重新调用错误分布情况。 例如，失败的请求总数、错误代码，等等。请注意，将为每个错误代码记录一个事件。
- 事件 ID 9009 提供服务器终结点的回调会话信息。 例如，DurationSeconds、CountFilesRecallSucceeded、CountFilesRecallFailed 等。
- 事件 ID 9059 提供服务器终结点的应用程序回调分布情况。 例如，ShareId、应用程序名称和 TotalEgressNetworkBytes。

<a id="files-fail-tiering"></a>**排查文件无法分层的问题**  
如果文件无法分层到 Azure 文件：

1. 在事件查看器中，查看位于 Applications and Services\Microsoft\FileSync\Agent 下的遥测、操作和诊断事件日志。 
   1. 验证文件是否在 Azure 文件共享中存在。

      > [!NOTE]
      > 文件必须先同步到 Azure 文件共享，然后才能分层。

   2. 验证服务器是否已建立 Internet 连接。 
   3. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
       - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

> [!NOTE]
> 如果文件无法分层，则每小时在“遥测”事件日志中记录事件 ID 9003 一次（为每个错误代码记录一个事件）。 如果需要其他信息才能诊断问题，请使用“操作”和“诊断”事件日志。

<a id="files-fail-recall"></a>排查无法重新调用文件的问题  
如果无法重新调用文件：
1. 在事件查看器中，查看位于 Applications and Services\Microsoft\FileSync\Agent 下的遥测、操作和诊断事件日志。
    1. 验证文件是否在 Azure 文件共享中存在。
    2. 验证服务器是否已建立 Internet 连接。 
    3. 打开服务 MMC 管理单元，然后验证存储同步代理服务 (FileSyncSvc) 是否正在运行。
    4. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
        - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

> [!NOTE]
> 如果文件无法重新调用，则每小时在“遥测”事件日志中记录事件 ID 9006 一次（为每个错误代码记录一个事件）。 如果需要其他信息才能诊断问题，请使用“操作”和“诊断”事件日志。

<a id="files-unexpectedly-recalled"></a>**排查在服务器上意外重新调用文件的问题**  
读取大量文件的防病毒、备份和其他应用程序会导致意外的重新调用，除非这些应用程序遵循脱机属性并跳过读取这些文件的内容。 跳过支持此选项的产品的脱机文件可帮助避免在执行防病毒软件扫描或备份作业等操作时出现意外的重新调用。

请咨询软件供应商，了解如何配置其解决方案以跳过读取脱机文件。

在其他一些情况下（例如，在文件资源管理器中浏览文件时），也可能出现意外的重新调用。 在服务器上的文件资源管理器中打开包含云分层文件的文件夹可能导致意外的重新调用。 在服务器上启用防病毒解决方案时更是如此。

> [!NOTE]
>使用遥测事件日志中的事件 ID 9059 确定导致重新调用的应用程序。 此事件为服务器终结点提供应用程序重新调用分发，并每小时记录一次。

## <a name="general-troubleshooting"></a>常规故障排除
如果在服务器上遇到 Azure 文件同步问题，请先完成以下步骤：
1. 在事件查看器中，查看遥测、操作和诊断事件日志。
    - 同步、分层和重新调用问题记录在 Applications and Services\Microsoft\FileSync\Agent 下的遥测、诊断和操作事件日志中。
    - 与管理服务器相关的（例如配置设置）问题记录在 Applications and Services\Microsoft\FileSync\Management 下的操作和诊断事件日志中。
2. 验证 Azure 文件同步服务是否正在服务器上运行。
    - 打开服务 MMC 管理单元，然后验证存储同步代理服务 (FileSyncSvc) 是否正在运行。
3. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
    - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

如果问题未得到解决，请运行 AFSDiag 工具：
1. 创建用于保存 AFSDiag 输出的目录（例如，C:\Output）。
2. 打开权限提升的 PowerShell 窗口并运行以下命令（在每条命令后面按 Enter）：

    ```powershell
    cd "c:\Program Files\Azure\StorageSyncAgent"
    Import-Module .\afsdiag.ps1
    Debug-Afs c:\output # Note: Use the path created in step 1.
    ```

3. 对于 Azure 文件同步内核模式跟踪级别，请输入 **1**（除非指定为创建更详细的跟踪）并按 Enter。
4. 对于 Azure 文件同步用户模式跟踪级别，请输入 **1**（除非指定为创建更详细的跟踪）并按 Enter。
5. 再现问题。 完成后，输入 **D**。
6. 随即会将一个包含日志和跟踪文件的 .zip 文件保存到指定的输出目录。

## <a name="see-also"></a>另请参阅
- [监视 Azure 文件同步](storage-sync-files-monitoring.md)
- [Azure 文件常见问题解答](storage-files-faq.md)
- [在 Windows 中排查 Azure 文件问题](storage-troubleshoot-windows-file-connection-problems.md)
- [在 Linux 中排查 Azure 文件问题](storage-troubleshoot-linux-file-connection-problems.md)
