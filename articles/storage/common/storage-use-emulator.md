---
title: 使用 Azure 存储模拟器进行开发和测试 | Microsoft Docs
description: Azure 存储模拟器为开发和测试 Azure 存储应用程序提供了免费的本地开发环境。 了解如何对请求进行授权、如何从应用程序连接到模拟器以及如何使用命令行工具。
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: article
ms.date: 08/10/2018
ms.author: mhopkins
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: 5f55228c80142b2a21af585cb04d16f148460af0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149100"
---
# <a name="use-the-azure-storage-emulator-for-development-and-testing"></a>使用 Azure 存储模拟器进行开发和测试

Microsoft Azure 存储模拟器提供了一个模拟 Azure Blob、队列和表服务以进行开发的本地环境。 使用存储模拟器，可以在本地针对存储服务测试应用程序，而无需创建 Azure 订阅且不会产生任何费用。 如果对应用程序在模拟器中的工作情况感到满意，则可以切换到在云中使用 Azure 存储帐户。

## <a name="get-the-storage-emulator"></a>获取存储模拟器
存储模拟器作为 [Microsoft Azure SDK](https://azure.microsoft.com/downloads/) 的一部分提供。 此外，还可使用[独立安装程序](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409)（直接下载）来安装存储模拟器。 若要安装存储模拟器，必须在计算机上具有管理权限。

存储模拟器目前仅在 Windows 上运行。 若考虑使用 Linux 的存储模拟器，一个选择是使用由社区维护的开源存储模拟器 [Azurite](https://github.com/azure/azurite)。

> [!NOTE]
> 在一个版本的存储模拟器中创建的数据不保证在使用不同版本时可以访问。 如果需要长期保存数据，建议将该数据存储在 Azure 存储帐户中，而不是在存储模拟器中。
> 
> 存储模拟器依赖于特定版本的 OData 库。 不支持将存储模拟器使用的 OData DLL 替换为其他版本，这样做可能会导致意外行为。 不过，可以使用存储服务支持的任何版本的 OData 向模拟器发送请求。

## <a name="how-the-storage-emulator-works"></a>存储模拟器的工作原理
存储模拟器使用本地 Microsoft SQL Server 实例和本地文件系统来模拟 Azure 存储服务。 默认情况下，存储模拟器使用 Microsoft SQL Server 2012 Express LocalDB 中的数据库。 可以选择将存储模拟器配置为访问 SQL Server 的本地实例而不是 LocalDB 实例。 有关详细信息，请参阅本文后面的[启动并初始化存储模拟器](#start-and-initialize-the-storage-emulator)部分。

存储模拟器使用 Windows 身份验证连接到 SQL Server 或 LocalDB。

存储模拟器与 Azure 存储服务之间存在一些功能差异。 有关这些差异的详细信息，请参阅本文后面的[存储模拟器与 Azure 存储之间的差异](#differences-between-the-storage-emulator-and-azure-storage)部分。

## <a name="start-and-initialize-the-storage-emulator"></a>启动和初始化存储模拟器

若要启动 Azure 存储模拟器：
1. 选择“开始”按钮或按“Windows”键。
2. 开始键入 `Azure Storage Emulator`。
3. 从所示应用程序的列表中选择该模拟器。

存储模拟器启动时，会显示“命令提示符”窗口。 可使用此控制台窗口启动和停止存储模拟器、清除数据、获取状态和初始化模拟器。 有关详细信息，请参阅本文后面的[存储模拟器命令行工具参考](#storage-emulator-command-line-tool-reference)部分。

运行模拟器时，在 Windows 任务栏通知区域中会显示一个图标。

关闭存储模拟器的“命令提示符”窗口后，存储模拟器将继续运行。 若要重新显示“存储模拟器”控制台窗口，请执行上述步骤，就像启动存储模拟器一样。

首次运行存储模拟器时，会为用户初始化本地存储环境。 初始化过程在 LocalDB 中创建一个数据库，并为每个本地存储服务保留 HTTP 端口。

存储模拟器默认安装到 `C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator`。

> [!TIP]
> 可使用 [Microsoft Azure 存储资源管理器](https://storageexplorer.com)处理本地存储模拟器资源。 安装并启动存储模拟器后，在存储资源管理器资源树中的“存储帐户”下查找“(开发)”。
>

### <a name="initialize-the-storage-emulator-to-use-a-different-sql-database"></a>初始化存储模拟器以使用其他的 SQL 数据库

可以使用存储模拟器命令行工具初始化存储模拟器，使其指向默认 LocalDB 实例以外的其他 SQL 数据库实例：

1. 如[启动和初始化存储模拟器](#start-and-initialize-the-storage-emulator)部分中所述，打开“存储模拟器”控制台窗口。
1. 在控制台窗口中，键入以下命令，其中 `<SQLServerInstance>` 是 SQL Server 实例的名称。 若要使用 LocalDB，请指定 `(localdb)\MSSQLLocalDb` 作为 SQL Server 实例。

   `AzureStorageEmulator.exe init /server <SQLServerInstance>`

   也可以使用以下命令，该命令指示模拟器使用默认 SQL Server 实例：

   `AzureStorageEmulator.exe init /server .`

   或者，可以使用以下命令将数据库重新初始化为默认的 LocalDB 实例：

   `AzureStorageEmulator.exe init /forceCreate`

有关这些命令的详细信息，请参阅[存储模拟器命令行工具参考](#storage-emulator-command-line-tool-reference)。

> [!TIP]
> 可使用 [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) 管理 SQL Server 实例，包括 LocalDB 安装。 在 SMSS“连接到服务器”对话框的“服务器名称:”字段中，指定 `(localdb)\MSSQLLocalDb` 以连接到 LocalDB 实例。

## <a name="authenticating-requests-against-the-storage-emulator"></a>针对存储模拟器的请求进行身份验证
安装并启动存储模拟器后，可针对此模拟器测试代码。 与云中的 Azure 存储一样，针对存储模拟器发出的每个请求都必须经过授权，除非它是匿名请求。 可以使用共享密钥身份验证或使用共享访问签名 (SAS) 针对存储模拟器的请求进行授权。

### <a name="authorize-with-shared-key-credentials"></a>使用共享密钥凭据进行授权
[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

有关连接字符串的详细信息，请参阅[配置 Azure 存储连接字符串](../storage-configure-connection-string.md)。

### <a name="authorize-with-a-shared-access-signature"></a>使用共享访问签名进行授权

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

某些 Azure 存储客户端库（诸如 Xamarin 库）仅支持使用共享的访问签名 (SAS) 令牌进行身份验证。 可使用[存储资源管理器](https://storageexplorer.com/)之类的工具或其他支持共享密钥身份验证的应用程序创建 SAS 令牌。

还可使用 Azure PowerShell 来生成 SAS 令牌。 以下示例会生成可完全访问 blob 容器的 SAS 令牌：

1. 若尚未安装 Azure PowerShell，请进行安装（建议使用最新版 Azure PowerShell cmdlet 安装）。 有关安装说明，请参阅[安装和配置 Azure PowerShell](/powershell/azure/install-Az-ps)。
2. 打开 Azure PowerShell 并运行以下命令，将 `CONTAINER_NAME` 替换为所选名称：

```powershell
$context = New-AzStorageContext -Local

New-AzStorageContainer CONTAINER_NAME -Permission Off -Context $context

$now = Get-Date

New-AzStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri
```

得到的新容器的共享访问签名 URI 应类似于以下内容：

```
http://127.0.0.1:10000/devstoreaccount1/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss
```

此示例中创建的共享访问签名的有效期为一天。 该签名将授予对容器内 Blob 的完整访问权限（读取、写入、删除和列出）。

有关共享访问签名的详细信息，请参阅[在 Azure 存储中使用共享访问签名 (SAS)](../storage-dotnet-shared-access-signature-part-1.md)。

## <a name="addressing-resources-in-the-storage-emulator"></a>对存储模拟器中的资源进行寻址
存储模拟器的服务终结点不同于 Azure 存储帐户。 之所以不同，是因为本地计算机不执行域名解析，因此要求存储模拟器终结点是本地地址。

对 Azure 存储帐户中的资源进行寻址时，请使用以下方案。 帐户名称是 URI 主机名的一部分，要寻址的资源是 URI 路径的一部分：

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

例如，下面的 URI 是 Azure 存储帐户的 Blob 中的有效地址：

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

但在存储模拟器中，因为本地计算机不执行域名解析，帐户名称将是 URI 路径（而非主机名）的一部分。 可对存储模拟器中的资源使用以下 URI 格式：

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

例如，以下地址可用于访问存储模拟器中的 blob：

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

存储模拟器的服务终结点是：

* Blob 服务：`http://127.0.0.1:10000/<account-name>/<resource-path>`
* 队列服务：`http://127.0.0.1:10001/<account-name>/<resource-path>`
* 表服务：`http://127.0.0.1:10002/<account-name>/<resource-path>`

### <a name="addressing-the-account-secondary-with-ra-grs"></a>使用 RA-GRS 对帐户辅助副本进行寻址
存储模拟器从 3.1 版开始，支持读取访问地域冗余复制 (RA-GRS)。 对于同时位于云中和本地模拟器中的存储资源，可以通过在帐户名称后面追加 -secondary 来访问辅助位置。 例如，以下地址可用于访问使用存储模拟器中的只读辅助副本的 blob：

`http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt`

> [!NOTE]
> 若要使用存储模拟器以编程方式访问次要副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。 请参阅[适用于 .NET 的 Microsoft Azure 存储客户端库](https://msdn.microsoft.com/library/azure/dn261237.aspx)，获取详细信息。
>
>

## <a name="storage-emulator-command-line-tool-reference"></a>存储模拟器命令行工具参考
从版本 3.0 开始，启动存储模拟器时会显示控制台窗口。 可使用控制台窗口中的命令行来启动和停止模拟器、查询状态以及执行其他操作。

> [!NOTE]
> 如果已安装 Microsoft Azure 计算模拟器，则在启动存储模拟器时，会显示一个系统任务栏图标。 右键单击该图标可显示一个菜单，其中提供了启动和停止存储模拟器的图形化方式。
>
>

### <a name="command-line-syntax"></a>命令行语法
`AzureStorageEmulator.exe [start] [stop] [status] [clear] [init] [help]`

### <a name="options"></a>选项
若要查看选项列表，请在命令提示符下键入 `/help`。

| 选项 | 描述 | 命令 | 参数 |
| --- | --- | --- | --- |
| **启动** |启动存储模拟器。 |`AzureStorageEmulator.exe start [-inprocess]` |*-inprocess*：在当前进程中启动仿真器，而不是创建新的进程。 |
| **Stop** |停止存储模拟器。 |`AzureStorageEmulator.exe stop` | |
| **Status** |打印存储模拟器的状态。 |`AzureStorageEmulator.exe status` | |
| **Clear** |清除命令行上指定的所有服务中的数据。 |`AzureStorageEmulator.exe clear [blob] [table] [queue] [all]` |*blob*：清除 blob 数据。 <br/>*queue*：清除队列数据。 <br/>*table*：清除表数据。 <br/>*all*：清除所有服务中的所有数据。 |
| **Init** |执行一次性初始化以设置模拟器。 |<code>AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate&#124;-skipcreate] [-reserveports&#124;-unreserveports] [-inprocess]</code> |*-server serverName\instanceName*：指定托管 SQL 实例的服务器。 <br/>*-sqlinstance instanceName*：指定要在默认服务器实例中使用的 SQL 实例的名称。 <br/>*-forcecreate*：强制创建 SQL 数据库，即使它已存在。 <br/>*-skipcreate*：跳过创建 SQL 数据库的步骤。 此命令优先于 -forcecreate。<br/>*-reserveports*：尝试保留与服务关联的 HTTP 端口。<br/>*-unreserveports*：尝试取消保留与服务关联的 HTTP 端口。 此命令优先于 -reserveports。<br/>*-inprocess*：在当前进程中执行初始化，而不是生成新的进程。 如果更改端口保留设置，必须使用提升的权限启动当前进程。 |

## <a name="differences-between-the-storage-emulator-and-azure-storage"></a>存储模拟器与 Azure 存储之间的差异
因为存储模拟器是在本地的 SQL 实例中运行的模拟环境，所以模拟器与云中的 Azure 存储帐户之间存在功能差异：

* 存储模拟器只支持单一固定的帐户和众所周知的身份验证密钥。
* 存储模拟器不是可扩展的存储服务，并且不支持大量并发客户端。
* 如 [对存储模拟器中的资源进行寻址](#addressing-resources-in-the-storage-emulator)中所述，存储模拟器与 Azure 存储帐户中的资源以不同方式寻址。 存在这种差异是因为在云中可进行域名解析，但在本地计算机上不提供域名解析。
* 存储模拟器帐户从 3.1 版开始，支持读取访问地域冗余复制 (RA-GRS)。 在模拟器中，所有帐户都已启用 RA-GRS，在主要和次要副本之间不会有任何延迟。 获取 Blob 服务统计信息、获取队列服务统计信息和获取表服务统计信息操作在帐户辅助上受支持，并且将始终根据基础 SQL 数据库返回 `LastSyncTime` 响应元素的值作为当前时间。
* 文件服务和 SMB 协议服务终结点当前在存储模拟器中不受支持。
* 如果使用模拟器尚不支持的存储服务版本，则存储模拟器将返回 VersionNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求）。

### <a name="differences-for-blob-storage"></a>Blob 存储的差异
以下差异适用于模拟器中的 Blob 存储：

* 存储模拟器仅支持最大为 2 GB 的 Blob。
* 存储模拟器中 blob 名称的最大长度为 256 个字符，而 Azure 存储中 blob 名称的最大长度为 1024 个字符。
* 增量复制允许复制被覆盖的 blob 中的快照，这会在服务上返回失败消息。
* “获取页面范围差异”在使用增量复制 Blob 复制的快照之间不起作用。
* 对存在于存储模拟器中并具有活动租约的 Blob 执行的放置 Blob 操作可能会成功，即使在请求中未指定租约 ID。
* 追加 Blob 操作不受模拟器支持。 尝试对追加 Blob 执行的操作将返回 FeatureNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求）。

### <a name="differences-for-table-storage"></a>表存储的差异
以下差异适用于模拟器中的表存储：

* 存储模拟器中表服务的日期属性仅支持 SQL Server 2005 所支持的范围（要求其晚于 1753 年 1 月 1 日）。 1753 年 1 月 1 日之前的所有日期都会更改为此值。 日期的精度仅限于 SQL Server 2005 的精度，这意味着日期将精确到 1/300 秒。
* 存储模拟器支持小于 512 个字节的分区键和行键属性值（每个）。 此外，帐户名称、表名和键属性名称合在一起的总大小不能超过 900 个字节。
* 存储模拟器中的表中的某行的总大小被限制为小于 1 MB。
* 在存储模拟器中，数据类型 `Edm.Guid` 或 `Edm.Binary` 的属性仅支持查询筛选器字符串中的 `Equal (eq)` 和 `NotEqual (ne)` 比较运算符。

### <a name="differences-for-queue-storage"></a>队列存储的差异
模拟器中的队列存储没有任何差异。

## <a name="storage-emulator-release-notes"></a>存储模拟器发行说明

### <a name="version-57"></a>版本 5.7
修复了启用日志记录时可能导致崩溃的 bug。

### <a name="version-56"></a>版本 5.6
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2018-03-28 版本的存储服务。

### <a name="version-55"></a>版本 5.5
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2017-11-09 版本的存储服务。
* 已添加对 blob **Created** 属性的支持，该属性返回 blob 的创建时间。

### <a name="version-54"></a>版本 5.4
为了提高安装稳定性，模拟器在安装时不再尝试预留端口。 如果需要端口预留，请使用 **init** 命令的 *-reserveports* 选项进行指定。

### <a name="version-53"></a>版本 5.3
存储模拟器现在支持 Blob、队列和表服务终结点上的 2017-07-29 版本的存储服务。

### <a name="version-52"></a>版本 5.2
* 存储模拟器现在支持 Blob、队列和表服务终结点上 2017-04-17 版本的存储服务。
* 修复了表属性值未正确编码的 bug。

### <a name="version-51"></a>版本 5.1
修复了一个 Bug。出现该 Bug 时，存储模拟器会在某些不包含服务的响应中返回 `DataServiceVersion` 标头。

### <a name="version-50"></a>版本 5.0
* 存储模拟器安装程序不再检查现有的 MSSQL 和 .NET Framework 是否已安装。
* 存储模拟器安装程序不再在安装过程中创建数据库。 仍会在启动过程中视需要创建数据库。
* 创建数据库不再需要特权提升。
* 进行启动不再需要保留端口。
* 将以下选项添加到 `init`：`-reserveports`（需提升）、`-unreserveports`（需提升）和 `-skipcreate`。
* 系统托盘图标上的“存储模拟器 UI”选项现在可启动命令行界面。 不再提供旧的 GUI。
* 删除或重命名了某些 DLL。

### <a name="version-46"></a>版本 4.6
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2016-05-31 版的存储服务。

### <a name="version-45"></a>版本 4.5
* 修复了重命名后备数据库时导致存储模拟器初始化和安装失败的 bug。

### <a name="version-44"></a>版本 4.4
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-12-11 版本的存储服务。
* 现在，在处理大量 blob 时，存储模拟器的 blob 数据垃圾回收效率更高了。
* 修复了导致容器 ACL XML 的验证方式与存储服务的验证方式稍有不同的 bug。
* 修复了有时会导致在不正确的时区报告最大和最小日期时间值的 bug。

### <a name="version-43"></a>版本 4.3
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-07-08 版本的存储服务。

### <a name="version-42"></a>版本 4.2
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-04-05 版本的存储服务。

### <a name="version-41"></a>4.1 版
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-02-21 版本的存储服务，但是新的“追加 Blob”功能例外。
* 如果使用模拟器尚不支持的存储服务版本，则模拟器会返回一条有意义的错误消息。 建议使用最新版本的模拟器。 如果遇到 VersionNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求），请下载最新版本的存储模拟器。
* 修复了在并发合并操作期间发生争用情况下导致表实体数据不正确的错误。

### <a name="version-40"></a>4.0 版
* 存储模拟器可执行文件已重命名为 *AzureStorageEmulator.exe*。

### <a name="version-32"></a>版本 3.2
* 存储模拟器现在支持 Blob、队列和表服务终结点上的 2014-02-14 版本的存储服务。 文件服务终结点目前在存储模拟器中不受支持。 请参阅 [Versioning for the Azure Storage Services](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services)（Azure 存储服务的版本控制）以了解有关 2014-02-14 版本的详细信息。

### <a name="version-31"></a>版本 3.1
* 在存储模拟器中现在支持读取访问异地冗余存储 (RA-GRS)。 获取 Blob 服务统计信息、获取队列服务统计信息和获取表服务统计信息 API 在帐户辅助副本上受支持，并且将始终根据基础 SQL 数据库返回 LastSyncTime 响应元素的值作为当前时间。 若要使用存储模拟器以编程方式访问次要副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。 请参阅适用于 .NET 的 Microsoft Azure 存储客户端库参考获取详细信息。

### <a name="version-30"></a>版本 3.0
* Azure 存储模拟器中不再与计算模拟器在同一个包中提供。
* 为支持可编脚本的命令行界面，已弃用存储模拟器图形用户界面。 有关命令行界面的详细信息，请参阅“存储模拟器命令行工具参考”。 图形界面将继续存在于 3.0 版中，但仅在安装了计算模拟器的情况下通过右键单击系统托盘图标并选择“显示存储模拟器用户界面”来访问。
* 现在完全支持版本 2013年-08-15 的 Azure 存储服务。 （以前仅存储模拟器 2.2.1 预览版本支持此版本。）

## <a name="next-steps"></a>后续步骤

* 评估跨平台的、由社区维护的开源存储模拟器 [Azurite](https://github.com/arafato/azurite)。 
* [使用 .NET 的 Azure 存储示例](../storage-samples-dotnet.md)包含开发应用程序时可使用的多个代码示例的链接。
* 可使用 [Microsoft Azure 存储资源管理器](https://storageexplorer.com)处理云存储帐户和存储模拟器中的资源。
