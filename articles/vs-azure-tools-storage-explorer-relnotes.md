---
title: Microsoft Azure 存储资源管理器发行说明
description: Microsoft Azure 存储资源管理器发行说明
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: ''
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/12/2018
ms.author: cawa
ms.openlocfilehash: 18c52a47c291181547a9005c273dc1a329b8c207
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510954"
---
# <a name="microsoft-azure-storage-explorer-release-notes"></a>Microsoft Azure 存储资源管理器发行说明

本文包含 Azure 存储资源管理器 1.4.3 发行版及以前版本的发行说明。

[Microsoft Azure 存储资源管理器](./vs-azure-tools-storage-manage-with-storage-explorer.md)是一款独立应用，可用于在 Windows、macOS 和 Linux 上轻松处理 Azure 存储数据。

## <a name="version-180"></a>版本为 1.8.0
5/1/2019

### <a name="download-azure-storage-explorer-180"></a>下载 Azure 存储资源管理器 1.8.0
- [有关 Windows azure 存储资源管理 1.8.0](https://go.microsoft.com/fwlink/?LinkId=708343)
- [Azure 存储资源管理器 1.8.0 for Mac](https://go.microsoft.com/fwlink/?LinkId=708342)
- [适用于 Linux 的 azure 存储资源管理 1.8.0](https://go.microsoft.com/fwlink/?LinkId=722418)

### <a name="new"></a>新

* 集成的 AzCopy 版本已更新到版本 10.1.0。
* Ctrl / Cmd + R 现在可用于刷新当前具有焦点的编辑器。 [#1097](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1097)
* Azure Stack 存储 API 版本已更改为 2017年-04-17。
* ADLS 第 2 代管理访问对话框将立即使掩码保持同步的方式类似于其他 POSIX 权限的工具。 UI 还会警告您如果进行的更改将导致超过掩码的边界的用户或组权限。 [#1253](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1253)
* 有关 AzCopy 上传，现已启用标志来计算和设置的 MD5 哈希。 [#1223](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1223)


### <a name="preview-features"></a>预览功能

* 设备代码流登录现在可供预览。 若要启用此功能，请转到“预览”→“使用设备代码流登录”。 我们鼓励所有在空白登录窗口中遇到问题的用户试用此功能，因为经过证实，它是一种更可靠的登录方式。
* 与 AzCopy 集成的存储资源管理器目前可供预览。 若要启用此功能，请转到“预览”→“使用 AzCopy 来改善 Blob 上传和下载”。 使用 AzCopy 应该可以更快且更有效地完成 Blob 传输。

### <a name="fixes"></a>修复项

* 访问策略对话框中将不再在不具有过期时间的存储访问策略上设置到期日期。 [#764](https://www.github.com/Microsoft/AzureStorageExplorer/issues/764)
* 进行一些更改已到生成 SAS 对话框以确保生成 SAS 时，正确使用存储访问策略。 [#1269](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1269)
* 当尝试将上传一个非-512 字节对齐的页 Blob 的文件时，存储资源管理器现在将公开更相关的错误。 [#1050](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1050)
* 复制 Blob 容器使用的显示名称，将会失败。 现在，使用 Blob 容器的实际名称。 [#1166](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1166)
* 尝试在 ADLS 第 2 代文件夹，必须在其名称中的 unicode 字符上执行某些操作会失败。 现在应处理的所有操作。 [#980](https://www.github.com/Microsoft/AzureStorageExplorer/issues/980)

### <a name="known-issues"></a>已知问题

* 在执行非 AzCopy Blob 下载时，大型文件的 MD5 不正在验证。 这是因为存储 SDK 中的 bug。 [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* 使用 RBAC 时，存储资源管理器需要一些管理层权限才能访问存储资源。 有关详细信息，请参阅[故障排除指南](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting)。
* 在代理后面尝试访问 ADLS Gen2 Blob 可能会失败。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
   * ADLS 第 2 代
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 在 Linux 上运行存储资源管理器需要首先安装某些依赖项。 检查存储资源管理器[故障排除指南](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting?tabs=1804#linux-dependencies)有关详细信息。

## <a name="previous-releases"></a>以前的版本

* [版本 1.7.0](#version-170)
* [版本 1.6.2](#version-162)
* [版本 1.6.1](#version-161)
* [版本 1.6.0](#version-160)
* [版本 1.5.0](#version-150)
* [版本 1.4.4](#version-144)
* [版本 1.4.3](#version-143)
* [版本 1.4.2](#version-142)
* [版本 1.4.1](#version-141)
* [版本 1.3.0](#version-130)
* [版本 1.2.0](#version-120)
* [版本 1.1.0](#version-110)
* [版本 1.0.0](#version-100)
* [版本 0.9.6](#version-096)
* [版本 0.9.5](#version-095)
* [版本 0.9.4 和 0.9.3](#version-094-and-093)
* [版本 0.9.2](#version-092)
* [版本 0.9.1 和 0.9.0](#version-091-and-090)
* [版本 0.8.16](#version-0816)
* [版本 0.8.14](#version-0814)
* [版本 0.8.13](#version-0813)
* [版本 0.8.12、0.8.11 和 0.8.10](#version-0812-and-0811-and-0810)
* [版本 0.8.9 和 0.8.8](#version-089-and-088)
* [版本 0.8.7](#version-087)
* [版本 0.8.6](#version-086)
* [版本 0.8.5](#version-085)
* [版本 0.8.4](#version-084)
* [版本 0.8.3](#version-083)
* [版本 0.8.2](#version-082)
* [版本 0.8.0](#version-080)
* [版本 0.7.20160509.0](#version-07201605090)
* [版本 0.7.20160325.0](#version-07201603250)
* [版本 0.7.20160129.1](#version-07201601291)
* [版本 0.7.20160105.0](#version-07201601050)
* [版本 0.7.20151116.0](#version-07201511160)

## <a name="version-170"></a>版本 1.7.0
3/5/2019

### <a name="download-azure-storage-explorer-170"></a>下载 Azure 存储资源管理器 1.7.0
- [适用于 Windows 的 Azure 存储资源管理器 1.7.0](https://go.microsoft.com/fwlink/?LinkId=708343)
- [适用于 Mac 的 Azure 存储资源管理器 1.7.0](https://go.microsoft.com/fwlink/?LinkId=708342)
- [适用于 Linux 的 Azure 存储资源管理器 1.7.0](https://go.microsoft.com/fwlink/?LinkId=722418)

### <a name="new"></a>新

* 现在，在管理 ADLS Gen2 容器、文件或文件夹的访问权限时，可以更改所有者和拥有组。
* 在 Windows 上，从产品内部更新存储资源管理器现在属于增量安装操作。 这样应该可以加快更新的速度。 如果你偏向于全新安装，可以自行下载[安装程序](https://azure.microsoft.com/features/storage-explorer/)并手动安装。 #1089

### <a name="preview-features"></a>预览功能

* 设备代码流登录现在可供预览。 若要启用此功能，请转到“预览”→“使用设备代码流登录”。 我们鼓励所有在空白登录窗口中遇到问题的用户试用此功能，因为经过证实，它是一种更可靠的登录方式。 #938
* 与 AzCopy 集成的存储资源管理器目前可供预览。 若要启用此功能，请转到“预览”→“使用 AzCopy 来改善 Blob 上传和下载”。 使用 AzCopy 应该可以更快且更有效地完成 Blob 传输。

### <a name="fixes"></a>修复项

* 现在，在启用 AzCopy 后，可以选择要上传的 Blob 类型。 #1111
* 以前，如果为 ADLS Gen2 存储帐户启用了静态网站，然后在该帐户中附加名称和密钥，则存储资源管理器将检测不到已启用该分层命名空间。 此问题已解决。 #1081
* 在 Blob 编辑器中，不再能够按剩余保留天数或状态进行排序。 此问题已解决。 #1106
* 在版本 1.5.0 之后，存储资源管理器不再会先等待服务器端复制完成，然后在重命名或复制/粘贴操作期间报告成功结果。 此问题已解决。 #976
* 使用试验性的 AzCopy 功能时，单击“将命令复制到剪贴板”后复制的命令本身不一定始终可运行。 现在，将会复制手动运行传输所需的所有命令。 #1079
* 以前，如果在代理后面操作，则无法访问 ADLS Gen2 Blob。 这是因为存储 SDK 使用的新网络库中存在一个 bug。 1.7.0 会尝试缓解此问题，但有些人仍可能会遇到问题。 在将来的更新中会发布完整修复措施。 #1090
* 在 1.7.0 中，“保存文件”对话框现在会正确记住文件的上次保存位置。 #16
* 在“属性”窗格中，存储帐户的 SKU 层显示为帐户类型。 此问题已解决。 #654
* 有时，即使正确输入 Blob 的名称，也无法中断该 Blob 的租约。 此问题已解决。 #1070

### <a name="known-issues"></a>已知问题

* 使用 RBAC 时，存储资源管理器需要一些管理层权限才能访问存储资源。 有关详细信息，请参阅[故障排除指南](https://docs.microsoft.com/azure/storage/common/storage-explorer-troubleshooting)。
* 在代理后面尝试访问 ADLS Gen2 Blob 可能会失败。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-162"></a>版本 1.6.2
2019/1/9

### <a name="hotfixes"></a>修补程序
* 在 1.6.1 中，通过 ObjectId 添加到 ADLS Gen2 ACL 的实体（非用户）始终作为组添加。 现在，只能将组作为组添加，并将企业应用程序和服务主体等实体添加为用户。 [#1049](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1049)
* 如果 ADLS Gen2 存储帐户没有容器且附加了名称和密钥，则存储资源管理器将不会检测到存储帐户为 ADLS Gen2。 此问题已解决。 [#1048](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1048)
* 在 1.6.0 中，复制和粘贴期间的冲突不会提示解决方案。 相反，发生冲突的副本将失败。 现在，在第一次发生冲突时，你会被问及希望如何解决此问题。 [#1014](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1014)
* 由于 API 限制，已禁用“管理访问权限”对话框中 ObjectIds 的所有验证。 现在将仅对用户 UPN 进行验证。 [#954](https://www.github.com/Microsoft/AzureStorageExplorer/issues/954)
* 在“ADLS Gen2 管理访问权限”对话框中，无法修改组权限。 此问题已解决。 [#958](https://www.github.com/Microsoft/AzureStorageExplorer/issues/958)
* 已对 ADLS Gen2 编辑器添加了拖放上传支持。 [#953](https://www.github.com/Microsoft/AzureStorageExplorer/issues/953)
* ADLS Gen2 文件和文件夹的属性对话框中的 URL 属性有时缺少“/”。 此问题已解决。 [#960](https://www.github.com/Microsoft/AzureStorageExplorer/issues/960)
* 如果获取 ADLS Gen2 容器、文件或文件夹的当前权限失败，则错误现在会正确显示在活动日志中。 [#965](https://www.github.com/Microsoft/AzureStorageExplorer/issues/965)
* 已缩短用于打开文件而创建的临时路径，以降低创建长于 Windows 上的 MAX_PATH 路径的可能性。 [#93](https://www.github.com/Microsoft/AzureStorageExplorer/issues/93)
* 现在，当没有已登录用户且未附加任何资源时会正确显示“连接”对话框。 [#944](https://www.github.com/Microsoft/AzureStorageExplorer/issues/944)
* 在 1.6.0 版本中，保存非 HNS Blob 和文件会对每个属性的值进行编码。 这会导致对仅包含 ASCII 字符的值进行不必要的编码操作。 现在，仅当值包含 ASCII 字符时，才会对其进行编码。 [#986](https://www.github.com/Microsoft/AzureStorageExplorer/issues/986)
* 如果使用了 SAS，并且该 SAS 不具有读取权限，则无法将文件夹上传到非 HNS Blob 容器。 此问题已解决。 [#970](https://www.github.com/Microsoft/AzureStorageExplorer/issues/970)
* 无法取消 AzCopy 传输。 此问题已解决。 [#943](https://www.github.com/Microsoft/AzureStorageExplorer/issues/943)
* 如果某文件夹的名称包含空格，则尝试从 ADLS Gen2 Blob 容器中下载该文件夹时，AzCopy 会失败。 此问题已解决。 [#990](https://www.github.com/Microsoft/AzureStorageExplorer/issues/990)
* CosmosDB 编辑器在 1.6.0 版本中出现中断。 现已修复。 [#950](https://www.github.com/Microsoft/AzureStorageExplorer/issues/950)
        
### <a name="new"></a>新

* 现在可以使用存储资源管理器以通过 [RBAC](https://go.microsoft.com/fwlink/?linkid=2045904&clcid=0x409) 访问 Blob 数据。 如果已登录，且存储资源管理器无法检索存储帐户的密钥，那么与数据进行交互时将使用 OAuth 标记进行身份验证。
* 存储资源管理器现支持 ADLS Gen2 存储帐户。 当存储资源管理器检测到已为存储帐户启用分层命名空间时，存储帐户名称旁边会显示“（ADLS Gen2 预览版）”。 存储资源管理器可以检测登录时是否启用了分层命名空间，或者是否已对存储帐户附加了名称和密钥。 对于 ADLS Gen2 存储帐户，可使用存储资源管理器执行以下操作：
  * 创建和删除容器
  * 管理容器属性和权限（左侧）
  * 查看和导航容器内的数据
  * 创建新文件夹
  * 上传、下载、重命名并删除文件和文件夹
  * 管理文件和文件夹属性和权限（右侧）。
    
    当前不提供其他典型的 Blob 功能，例如软删除和快照。 管理权限仅在登录时可用。 此外，使用 ADLS Gen2 存储帐户时，存储资源管理器会使用 AzCopy 进行所有上传和下载操作，并且默认使用名称和密钥凭据进行所有操作（若可用）。
* 在经过强烈的用户反馈之后，中断租用可再次用于同时中断多个 Blob 上的租用。

### <a name="known-issues"></a>已知问题

* 从 ADLS Gen2 存储帐户中下载时，如果已存在某个要传输的文件，那么 AzCopy 有时会出现故障。 我们会在即将发布的修补程序中修复此缺陷。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-161"></a>版本 1.6.1
2018/12/18

### <a name="hotfixes"></a>修补程序
* 由于 API 限制，已禁用“管理访问权限”对话框中 ObjectIds 的所有验证。 现在将仅对用户 UPN 进行验证。 [#954](https://www.github.com/Microsoft/AzureStorageExplorer/issues/954)
* 在“ADLS Gen2 管理访问权限”对话框中，无法修改组权限。 此问题已解决。 [#958](https://www.github.com/Microsoft/AzureStorageExplorer/issues/958)
* 已对 ADLS Gen2 编辑器添加了拖放上传支持。 [#953](https://www.github.com/Microsoft/AzureStorageExplorer/issues/953)
* ADLS Gen2 文件和文件夹的属性对话框中的 URL 属性有时缺少“/”。 此问题已解决。 [#960](https://www.github.com/Microsoft/AzureStorageExplorer/issues/960)
* 如果获取 ADLS Gen2 容器、文件或文件夹的当前权限失败，则错误现在会正确显示在活动日志中。 [#965](https://www.github.com/Microsoft/AzureStorageExplorer/issues/965)
* 已缩短用于打开文件而创建的临时路径，以降低创建长于 Windows 上的 MAX_PATH 路径的可能性。 [#93](https://www.github.com/Microsoft/AzureStorageExplorer/issues/93)
* 现在，当没有已登录用户且未附加任何资源时会正确显示“连接”对话框。 [#944](https://www.github.com/Microsoft/AzureStorageExplorer/issues/944)
* 在 1.6.0 版本中，保存非 HNS Blob 和文件会对每个属性的值进行编码。 这会导致对仅包含 ASCII 字符的值进行不必要的编码操作。 现在，仅当值包含 ASCII 字符时，才会对其进行编码。 [#986](https://www.github.com/Microsoft/AzureStorageExplorer/issues/986)
* 如果使用了 SAS，并且该 SAS 不具有读取权限，则无法将文件夹上传到非 HNS Blob 容器。 此问题已解决。 [#970](https://www.github.com/Microsoft/AzureStorageExplorer/issues/970)
* 无法取消 AzCopy 传输。 此问题已解决。 [#943](https://www.github.com/Microsoft/AzureStorageExplorer/issues/943)
* 如果某文件夹的名称包含空格，则尝试从 ADLS Gen2 Blob 容器中下载该文件夹时，AzCopy 会失败。 此问题已解决。 [#990](https://www.github.com/Microsoft/AzureStorageExplorer/issues/990)
* CosmosDB 编辑器在 1.6.0 版本中出现中断。 现已修复。 [#950](https://www.github.com/Microsoft/AzureStorageExplorer/issues/950)
        
### <a name="new"></a>新

* 现在可以使用存储资源管理器以通过 [RBAC](https://go.microsoft.com/fwlink/?linkid=2045904&clcid=0x409) 访问 Blob 数据。 如果已登录，且存储资源管理器无法检索存储帐户的密钥，那么与数据进行交互时将使用 OAuth 标记进行身份验证。
* 存储资源管理器现支持 ADLS Gen2 存储帐户。 当存储资源管理器检测到已为存储帐户启用分层命名空间时，存储帐户名称旁边会显示“（ADLS Gen2 预览版）”。 存储资源管理器可以检测登录时是否启用了分层命名空间，或者是否已对存储帐户附加了名称和密钥。 对于 ADLS Gen2 存储帐户，可使用存储资源管理器执行以下操作：
  * 创建和删除容器
  * 管理容器属性和权限（左侧）
  * 查看和导航容器内的数据
  * 创建新文件夹
  * 上传、下载、重命名并删除文件和文件夹
  * 管理文件和文件夹属性和权限（右侧）。
    
    当前不提供其他典型的 Blob 功能，例如软删除和快照。 管理权限仅在登录时可用。 此外，使用 ADLS Gen2 存储帐户时，存储资源管理器会使用 AzCopy 进行所有上传和下载操作，并且默认使用名称和密钥凭据进行所有操作（若可用）。
* 在经过强烈的用户反馈之后，中断租用可再次用于同时中断多个 Blob 上的租用。

### <a name="known-issues"></a>已知问题

* 从 ADLS Gen2 存储帐户中下载时，如果已存在某个要传输的文件，那么 AzCopy 有时会出现故障。 我们会在即将发布的修补程序中修复此缺陷。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-160"></a>版本 1.6.0
2018/12/5

### <a name="new"></a>新

* 现在可以使用存储资源管理器以通过 [RBAC](https://go.microsoft.com/fwlink/?linkid=2045904&clcid=0x409) 访问 Blob 数据。 如果已登录，且存储资源管理器无法检索存储帐户的密钥，那么与数据进行交互时将使用 OAuth 标记进行身份验证。
* 存储资源管理器现支持 ADLS Gen2 存储帐户。 当存储资源管理器检测到已为存储帐户启用分层命名空间时，存储帐户名称旁边会显示“（ADLS Gen2 预览版）”。 存储资源管理器可以检测登录时是否启用了分层命名空间，或者是否已对存储帐户附加了名称和密钥。 对于 ADLS Gen2 存储帐户，可使用存储资源管理器执行以下操作：
  * 创建和删除容器
  * 管理容器属性和权限（左侧）
  * 查看和导航容器内的数据
  * 创建新文件夹
  * 上传、下载、重命名并删除文件和文件夹
  * 管理文件和文件夹属性和权限（右侧）。
    
    当前不提供其他典型的 Blob 功能，例如软删除和快照。 管理权限仅在登录时可用。 此外，使用 ADLS Gen2 存储帐户时，存储资源管理器会使用 AzCopy 进行所有上传和下载操作，并且默认使用名称和密钥凭据进行所有操作（若可用）。
* 在经过强烈的用户反馈之后，中断租用可再次用于同时中断多个 Blob 上的租用。

### <a name="known-issues"></a>已知问题

* 从 ADLS Gen2 存储帐户中下载时，如果已存在某个要传输的文件，那么 AzCopy 有时会出现故障。 我们会在即将发布的修补程序中修复此缺陷。
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-150"></a>版本 1.5.0
10/29/2018

### <a name="new"></a>新

* 现在可以使用 [AzCopy v10（预览版）](https://github.com/Azure/azure-storage-azcopy)来上传和下载 Blob。 若要启用此功能，请转到“试验”菜单，然后单击“使用 AzCopy 改进 Blob 上传和下载”。 启用后，AzCopy 将在以下方案中使用：
   * 通过工具栏或拖放操作将文件夹和文件上传到 Blob 容器。
   * 通过工具栏或上下文菜单下载文件夹和文件。

* 此外，在使用 AzCopy 时：
   * 可以复制 AzCopy 命令用于执行传输到剪贴板的操作。 只需在活动日志中单击“将 AzCopy 命令复制到剪贴板”即可。
   * 上传后需要手动刷新 Blob 编辑器。
   * 不支持将文件上传到追加 Blob，vhd 文件将作为页 Blob 上传，其他所有文件将作为块 Blob 上传。
   * 只有在上传或下载完成后，上传或下载期间发生的错误或冲突才会显示。

最后，将来会支持对文件共享使用 AzCopy。
* 存储资源管理器目前使用 Electron 版本 2.0.11。
* 目前，每次只能对一个 Blob 执行中断性租约。 此外，必须输入要中断其租约的 Blob 的名称。 做出此项更改的目的是为了减少意外中断租用的可能性，尤其是对于 VM。 #394
* 如果你曾经遇到过登录问题，现在可以尝试重置身份验证。 转到“帮助”菜单并单击“重置”即可访问此功能。 #419

### <a name="fix"></a>解决

* 根据强烈的用户要求，我们已重新启用默认的仿真器节点。 仍可以通过“连接”对话框添加其他仿真器连接，但如果仿真器配置为使用默认端口，则你也可以使用“本地和附加/存储帐户”下的“仿真器 * 默认端口”节点。 #669
* 存储资源管理器不再允许设置包含前导或尾随空白字符的 Blob 元数据值。 #760
* 以前，在“连接”对话框的相同页上始终会启用“登录”按钮。 现在，在适当的时候会将其禁用。 #761
* 未添加快速访问项时，快速访问不再在控制台中生成错误。

### <a name="known-issues"></a>已知问题

* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅 #537。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能对你解除阻止，请对此问题发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。 若要在向/从 Blob 容器上传或下载时解决此问题，可以使用试验性的 AzCopy 功能。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能。 在处理 Azure Stack 资源时尝试使用这些功能可能会导致意外错误。
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```


## <a name="version-144"></a>版本 1.4.4
2018/10/15

### <a name="hotfixes"></a>修补程序
* Azure 资源管理 API 版本已回滚，以对 Azure 美国政府用户解除阻止。 [#696](https://github.com/Microsoft/AzureStorageExplorer/issues/696)
* 加载微调控件现使用 CSS 动画来减少存储资源管理器所用的 GPU 数量。 [#653](https://github.com/Microsoft/AzureStorageExplorer/issues/653)

### <a name="new"></a>新
* 外部资源附加（例如 SAS 连接和模拟器）已显著改进。 现在可以：
   * 自定义要附加的资源的显示名称。 [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * 附加到使用不同端口的多个本地模拟器。 [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * 将附加的资源添加到“快速访问”。 [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* 存储资源管理器现在支持软删除。 可以:
   * 通过右键单击你的存储帐户的“Blob 容器”节点来配置软删除策略。
   * 通过在导航栏旁边的下拉列表中选择“活动的和已删除的 Blob”在 Blob 编辑器中查看软删除的 Blob。
   * 撤消删除已删除的 Blob。

### <a name="fixes"></a>修复项
* “配置 CORS 设置”操作在高级存储帐户中不再可用，因为高级存储帐户不支持 CORS。 [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* SAS 附加的服务现在有一个“共享访问签名”属性。 [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* 对于已固定到“快速访问”的 Blob 和 GPV2 存储帐户，现在可使用“设置默认访问层级”操作。 [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* 有时候，存储资源管理器无法显示经典存储帐户。 [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>已知问题
* 使用 Azure 存储仿真器或 Azurite 等仿真器时，需要让它们在默认端口上侦听连接。 否则，存储资源管理器无法连接到这些仿真器。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-143"></a>版本 1.4.3
2018/10/11

### <a name="hotfixes"></a>修补程序
* Azure 资源管理 API 版本已回滚，以对 Azure 美国政府用户解除阻止。 [#696](https://github.com/Microsoft/AzureStorageExplorer/issues/696)
* 加载微调控件现使用 CSS 动画来减少存储资源管理器所用的 GPU 数量。 [#653](https://github.com/Microsoft/AzureStorageExplorer/issues/653)

### <a name="new"></a>新
* 外部资源附加（例如 SAS 连接和模拟器）已显著改进。 现在可以：
   * 自定义要附加的资源的显示名称。 [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * 附加到使用不同端口的多个本地模拟器。 [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * 将附加的资源添加到“快速访问”。 [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* 存储资源管理器现在支持软删除。 可以:
   * 通过右键单击你的存储帐户的“Blob 容器”节点来配置软删除策略。
   * 通过在导航栏旁边的下拉列表中选择“活动的和已删除的 Blob”在 Blob 编辑器中查看软删除的 Blob。
   * 撤消删除已删除的 Blob。

### <a name="fixes"></a>修复项
* “配置 CORS 设置”操作在高级存储帐户中不再可用，因为高级存储帐户不支持 CORS。 [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* SAS 附加的服务现在有一个“共享访问签名”属性。 [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* 对于已固定到“快速访问”的 Blob 和 GPV2 存储帐户，现在可使用“设置默认访问层级”操作。 [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* 有时候，存储资源管理器无法显示经典存储帐户。 [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>已知问题
* 使用 Azure 存储仿真器或 Azurite 等仿真器时，需要让它们在默认端口上侦听连接。 否则，存储资源管理器无法连接到这些仿真器。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-142"></a>版本 1.4.2
2018/09/24

### <a name="hotfixes"></a>修补程序
* 将 Azure 资源管理 API 版本更新为 2018-07-01，以添加对新 Azure 存储帐户种类的支持。 [#652](https://github.com/Microsoft/AzureStorageExplorer/issues/652)

### <a name="new"></a>新
* 外部资源附加（例如 SAS 连接和模拟器）已显著改进。 现在可以：
   * 自定义要附加的资源的显示名称。 [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * 附加到使用不同端口的多个本地模拟器。 [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * 将附加的资源添加到“快速访问”。 [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* 存储资源管理器现在支持软删除。 可以:
   * 通过右键单击你的存储帐户的“Blob 容器”节点来配置软删除策略。
   * 通过在导航栏旁边的下拉列表中选择“活动的和已删除的 Blob”在 Blob 编辑器中查看软删除的 Blob。
   * 撤消删除已删除的 Blob。

### <a name="fixes"></a>修复项
* “配置 CORS 设置”操作在高级存储帐户中不再可用，因为高级存储帐户不支持 CORS。 [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* SAS 附加的服务现在有一个“共享访问签名”属性。 [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* 对于已固定到“快速访问”的 Blob 和 GPV2 存储帐户，现在可使用“设置默认访问层级”操作。 [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* 有时候，存储资源管理器无法显示经典存储帐户。 [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>已知问题
* 使用 Azure 存储仿真器或 Azurite 等仿真器时，需要让它们在默认端口上侦听连接。 否则，存储资源管理器无法连接到这些仿真器。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-141"></a>版本 1.4.1
2018/08/28

### <a name="hotfixes"></a>修补程序
* 首次启动时，存储资源管理器无法生成用来加密敏感数据的密钥。 当使用“快速访问”以及附加资源时，这将导致出现问题。 [#535](https://github.com/Microsoft/AzureStorageExplorer/issues/535)
* 如果你的帐户不要求其主租户进行 MFA 但要求其他一些租户进行 MFA，则存储资源管理器将无法列出订阅。 现在，在使用此类帐户登录后，存储资源管理器将要求你重新输入凭据并执行 MFA。 [#74](https://github.com/Microsoft/AzureStorageExplorer/issues/74)
* 存储资源管理器无法从“Azure 德国”和“Azure 美国政府”附加资源。 [#572](https://github.com/Microsoft/AzureStorageExplorer/issues/572)
* 如果登录到具有相同电子邮件地址的两个帐户，则存储资源管理器有时无法在树视图中显示你的资源。 [#580](https://github.com/Microsoft/AzureStorageExplorer/issues/580)
* 在较慢的 Windows 计算机上，初始屏幕有时需要很长时间才会显示。 [#586](https://github.com/Microsoft/AzureStorageExplorer/issues/586)
* 即使已存在附加的帐户或服务，连接对话框也会显示。 [#588](https://github.com/Microsoft/AzureStorageExplorer/issues/588)

### <a name="new"></a>新
* 外部资源附加（例如 SAS 连接和模拟器）已显著改进。 现在可以：
   * 自定义要附加的资源的显示名称。 [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * 附加到使用不同端口的多个本地模拟器。 [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * 将附加的资源添加到“快速访问”。 [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* 存储资源管理器现在支持软删除。 可以:
   * 通过右键单击你的存储帐户的“Blob 容器”节点来配置软删除策略。
   * 通过在导航栏旁边的下拉列表中选择“活动的和已删除的 Blob”在 Blob 编辑器中查看软删除的 Blob。
   * 撤消删除已删除的 Blob。

### <a name="fixes"></a>修复项
* “配置 CORS 设置”操作在高级存储帐户中不再可用，因为高级存储帐户不支持 CORS。 [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* SAS 附加的服务现在有一个“共享访问签名”属性。 [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* 对于已固定到“快速访问”的 Blob 和 GPV2 存储帐户，现在可使用“设置默认访问层级”操作。 [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* 有时候，存储资源管理器无法显示经典存储帐户。 [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>已知问题
* 使用 Azure 存储仿真器或 Azurite 等仿真器时，需要让它们在默认端口上侦听连接。 否则，存储资源管理器无法连接到这些仿真器。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-130"></a>版本 1.3.0
2018/07/09

### <a name="new"></a>新
* 现在支持访问静态网站使用的 $web 容器。 这使你可以轻松上传和管理网站使用的文件和文件夹。 [#223](https://github.com/Microsoft/AzureStorageExplorer/issues/223)
* macOS 上的应用栏已经重新组织。 更改包括“文件”菜单、一些快捷键更改以及应用菜单下的几个新命令。 [#99](https://github.com/Microsoft/AzureStorageExplorer/issues/99)
* 用于登录到 Azure 美国政府的颁发机构终结点已更改为 https://login.microsoftonline.us/
* 辅助功能：当屏幕阅读器处于活动状态时，键盘导航现在可以与用于在右侧显示项目的表格配合使用。 可以使用箭头键在行和列中导航，使用 Enter 调用默认操作，使用上下文菜单键打开项目的上下文菜单，使用 Shift 或 Control 进行多选。 [#103](https://github.com/Microsoft/AzureStorageExplorer/issues/103)

### <a name="fixes"></a>修复项
*  在某些计算机上，子进程需要很长时间才能启动。 当发生这种情况时，会出现“子进程无法及时启动”错误。 分配给子进程用于启动的时间现在已经从 20 秒增加到 90 秒。 如果仍然受此问题影响，请对链接的 GitHub 问题发表评论。 [#281](https://github.com/Microsoft/AzureStorageExplorer/issues/281)
* 使用没有读取权限的 SAS 时，无法上传大型 blob。 已修改上传逻辑以在这种情况下正常工作。 [#305](https://github.com/Microsoft/AzureStorageExplorer/issues/305)
* 为容器设置公共访问级别将删除所有访问策略，反之亦然。 现在，在设置两者中的任何一个时，都会保留公共访问级别和访问策略。 [#197](https://github.com/Microsoft/AzureStorageExplorer/issues/197)
* “AccessTierChangeTime”在“属性”对话框中被截断。 此问题已解决。 [#145](https://github.com/Microsoft/AzureStorageExplorer/issues/145)
* “创建新目录”对话框中缺少“Microsoft Azure 存储资源管理器 -”前缀。 此问题已解决。 [#299](https://github.com/Microsoft/AzureStorageExplorer/issues/299)
* 辅助功能：使用 VoiceOver 时很难在“添加实体”对话框中导航。 已进行改进。 [#206](https://github.com/Microsoft/AzureStorageExplorer/issues/206)
* 辅助功能：“操作和属性”窗格的“折叠/展开”按钮的背景颜色与“高对比度黑色”主题中的类似 UI 控件不一致。 颜色已更改。 [#123](https://github.com/Microsoft/AzureStorageExplorer/issues/123)
* 辅助功能：在“高对比度黑色”主题中，“属性”对话框中“X”按钮的焦点样式不可见。 此问题已解决。 [#243](https://github.com/Microsoft/AzureStorageExplorer/issues/243)
* 辅助功能：“操作”和“属性”选项卡缺少几个 aria 值，这导致了低于标准的屏幕阅读器体验。 现在已经添加了缺失的 aria 值。 [#316](https://github.com/Microsoft/AzureStorageExplorer/issues/316)
* 辅助功能：左侧的折叠树节点未被赋予 aria 扩展值 false。 此问题已解决。 [#352](https://github.com/Microsoft/AzureStorageExplorer/issues/352)

### <a name="known-issues"></a>已知问题
* 从通过 SAS URI 附加的资源（例如 Blob 容器）进行分离可能会导致一个错误，该错误会阻止其他附件正确显示。 若要解决此问题，只需刷新组节点。 有关详细信息，请参阅[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/537)。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* Azure Stack 不支持以下功能，在使用 Azure Stack 时尝试使用这些功能可能会导致出现意外错误：
   * 文件共享
   * 访问层级
   * 软删除
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-120"></a>版本 1.2.0
06/12/2018

### <a name="new"></a>新
* 如果存储资源管理器无法做到只加载一部分租户中的订阅，则会显示所有已成功加载的订阅，此外还会专门针对失败的租户显示一条错误消息。 [#159](https://github.com/Microsoft/AzureStorageExplorer/issues/159)
* 在 Windows 上，当有更新可用时，现在可以选择“关闭时更新”。 如果选择此选项，将在关闭存储资源管理器后运行更新安装程序。 [#21](https://github.com/Microsoft/AzureStorageExplorer/issues/21)
* 查看文件共享快照时，“还原快照”会添加到文件共享编辑器的上下文菜单中。[#131](https://github.com/Microsoft/AzureStorageExplorer/issues/131)
* 现在始终会启用“清除队列”按钮。[#135](https://github.com/Microsoft/AzureStorageExplorer/issues/135)
* 已重新启用对登录到 ADFS Azure Stack 的支持。 需要 Azure Stack 1804 或更高版本。 [#150](https://github.com/Microsoft/AzureStorageExplorer/issues/150)

### <a name="fixes"></a>修复项
* 如果查看了某个文件共享的快照，并且该文件共享的名称是同一存储帐户中另一文件共享的前缀，则也会列出另一文件共享的快照。 现在已修复此问题。 [#255](https://github.com/Microsoft/AzureStorageExplorer/issues/255)
* 通过 SAS 附加时，从文件共享快照还原文件会导致出错。 现在已修复此问题。 [#211](https://github.com/Microsoft/AzureStorageExplorer/issues/211)
* 以前，在查看 Blob 的快照时，如果已选择基本 Blob 和单个快照，则会启用“提升快照”操作。 现在，仅当选择单个快照时才启用该操作。 [#230](https://github.com/Microsoft/AzureStorageExplorer/issues/230)
* 以前，如果单个作业（例如下载 Blob）已启动但随后失败，则在启动相同类型的另一个作业之前，不会自动重试该作业。 现在，所有作业都会自动重试，不管有多少个排队的作业。
* 以前，为 GPV2 和 Blob 存储帐户中新建的 Blob 容器打开的编辑器不包含“访问层”列。 现在已修复此问题。 [#109](https://github.com/Microsoft/AzureStorageExplorer/issues/109)
* 以前，在通过 SAS 附加存储帐户或 Blob 容器时，“访问层”列有时不会显示。 现在，始终会显示该列，但如果未设置“访问层”，则该列会包含空值。 [#160](https://github.com/Microsoft/AzureStorageExplorer/issues/160)
* 以前，禁止设置新上传的块 Blob 的访问层。 现在已修复此问题。 [#171](https://github.com/Microsoft/AzureStorageExplorer/issues/171)
* 以前，如果使用键盘调用“将选项卡保持打开状态”按钮，则键盘焦点会丢失。 现在，焦点会移到保持打开的选项卡。 [#163](https://github.com/Microsoft/AzureStorageExplorer/issues/163)
* 以前，对于查询生成器中的查询，VoiceOver 不会提供当前运算符的有用说明。 现在，说明更有描述性。 [#207](https://github.com/Microsoft/AzureStorageExplorer/issues/207)
* 以前，各种编辑器的分页链接没有描述性。 现在，这些链接更有描述性。 [#205](https://github.com/Microsoft/AzureStorageExplorer/issues/205)
* 以前，在“添加实体”对话框中，VoiceOver 不会告知某个输入元素所在的列。 现在，元素的说明中会包含当前列的名称。 [#206](https://github.com/Microsoft/AzureStorageExplorer/issues/206)
* 以前，在聚焦时，单选按钮和复选框没有可见的边框。 现在已修复此问题。 [#237](https://github.com/Microsoft/AzureStorageExplorer/issues/237)

### <a name="known-issues"></a>已知问题
* 使用 Azure 存储仿真器或 Azurite 等仿真器时，需要让它们在默认端口上侦听连接。 否则，存储资源管理器无法连接到这些仿真器。
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-110"></a>版本 1.1.0
2018/05/09

### <a name="new"></a>新
* 存储资源管理器现在支持使用 Azurite。 请注意：与 Azurite 的连接已硬编码到默认开发终结点。
* 存储资源管理器现在仅支持 Blob 的访问层和 GPV2 存储帐户。 在[此处](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers)详细了解访问层。
* 生成 SAS 时不再需要开始时间。

### <a name="fixes"></a>修复项
* 检索美国政府帐户的订阅被中断。 现在已修复此问题。 [#61](https://github.com/Microsoft/AzureStorageExplorer/issues/61)
* 访问策略的到期时间未正确保存。 现在已修复此问题。 [#50](https://github.com/Microsoft/AzureStorageExplorer/issues/50)
* 为容器中的项目生成 SAS URL 时，项目的名称未追加到 URL。 现在已修复此问题。 [#44](https://github.com/Microsoft/AzureStorageExplorer/issues/44)
* 创建 SAS 时，过去的到期时间有时会成为默认值。 这是因为存储资源管理器使用上次使用的开始时间和到期时间作为默认值。 现在，每次打开 SAS 对话框时，都会生成一组新的默认值。 [#35](https://github.com/Microsoft/AzureStorageExplorer/issues/35)
* 在存储帐户之间进行复制时，会生成 24 小时 SAS。 如果复制持续时间超过 24 小时，复制将失败。 我们已将 SAS 的有效时间增加到持续 1 周，以减少由于 SAS 过期而导致复制失败的可能性。 [#62](https://github.com/Microsoft/AzureStorageExplorer/issues/62)
* 对于某些活动，单击“取消”并非始终有效。 现在已修复此问题。 [#125](https://github.com/Microsoft/AzureStorageExplorer/issues/125)
* 对于某些活动来说，传输速度不正确。 现在已修复此问题。 [#124](https://github.com/Microsoft/AzureStorageExplorer/issues/124)
* 视图菜单中“上一个”的拼写错误。 现在已正确拼写。 [#71](https://github.com/Microsoft/AzureStorageExplorer/issues/71)
* Windows 安装程序的最后一页有“下一步”按钮。 该按钮已更改为“完成”按钮。 [#70](https://github.com/Microsoft/AzureStorageExplorer/issues/70)
* 使用 HC Black 主题时，对话框中按钮的 Tab 焦点不可见。 现在已可见。[#64](https://github.com/Microsoft/AzureStorageExplorer/issues/64)
* 活动日志中操作的“自动解析”的大小写不正确。 现已正确。 [#51](https://github.com/Microsoft/AzureStorageExplorer/issues/51)
* 从表中删除实体时，要求确认的对话框显示错误图标。 该对话框现在使用警告图标。 [#148](https://github.com/Microsoft/AzureStorageExplorer/issues/148)

### <a name="known-issues"></a>已知问题
* 如果使用用于 Mac 的 VS 并曾经创建过自定义 AAD 配置，可能无法登录。 若要解决此问题，请删除 ~/.IdentityService/AadConfigurations 的内容。 如果这样做不能取消阻止你，请对[此问题](https://github.com/Microsoft/AzureStorageExplorer/issues/97)发表评论。
* Azurite 还没有完全实现所有存储 API。 因此，在使用 Azurite 进行开发存储时可能会出现意外的错误或行为。
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 由于 NodeJS 中的 bug，从 OneDrive 文件夹上传不正常工作。 该 bug 已修复，但尚未集成到 Electron 中。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```


## <a name="version-100"></a>版本 1.0.0
04/16/2018

### <a name="new"></a>新
* 增强的身份验证，允许存储资源管理器使用与 Visual Studio 2017 相同的帐户存储。 若要使用此功能，需要重新登录到帐户并重置已筛选的订阅。
* 对于由 AAD 提供支持的 Azure Stack 帐户，当已启用“目标 Azure Stack”时，存储资源管理器现在会检索 Azure Stack 订阅。 不再需要创建自定义登录环境。
* 添加了几个快捷方式以启用更快的导航。 这些快捷方式包括切换各种面板和在编辑器之间移动。 有关更多详细信息，请参阅“视图”菜单。
* 存储资源管理器反馈现在位于 GitHub 上。 可以通过单击左下方的“反馈”按钮或通过转到 [https://github.com/Microsoft/AzureStorageExplorer/issues](https://github.com/Microsoft/AzureStorageExplorer/issues) 来访问我们的问题页面。 请随意提出建议、报告问题、提出问题或留下任何其他形式的反馈。
* 如果遇到 SSL 证书问题但找不到有问题的证书，现在可以从命令行使用 `--ignore-certificate-errors`标志启动存储资源管理器。 使用此标志启动时，存储资源管理器会忽略 SSL 证书错误。
* 现在，在 blob 和文件项的上下文菜单中将出现“下载”选项。
* 改进了辅助功能和屏幕阅读器支持。 如果依赖于辅助功能，请参阅我们的[辅助功能文档](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-accessibility)了解详细信息。
* 存储资源管理器现在使用 Electron 1.8.3

### <a name="breaking-changes"></a>重大更改
* 存储资源管理器已切换到新的身份验证库。 在切换到该库的过程中，需要重新登录到帐户并重置已筛选的订阅
* 用于加密敏感数据的方法已更改。 这可能会导致需要重新添加某些快速访问项，并且/或者需要重新附加某些附加资源。

### <a name="fixes"></a>修复项
* 代理后面的某些用户的组 blob 上传或下载操作会被“无法解析”错误消息中断。 现在已修复此问题。
* 如果使用直接链接时需要登录，单击“登录”提示会弹出一个空白对话框。 现在已修复此问题。
* 在 Linux 上，如果存储资源管理器因 GPU 进程故障而无法启动，现在会通知你发生故障，并让你使用“--disable-gpu”开关，然后存储资源管理器会在启用该开关的情况下自动重启。
* 在“访问策略”对话框中难以识别无效访问策略。 为提高可见性，现在无效访问策略 ID 的边框为红色。
* 活动日志在活动的不同部分之间有时有大面积的空格。 现在已修复此问题。
* 在表查询编辑器中，如果让 timestamp 子句处于无效状态，然后尝试修改另一个子句，编辑器会冻结。 现在，如果检测到另一个子句中的更改，编辑器会将 timestamp 子句还原为其上一个有效状态。
* 如果在树视图中键入搜索查询时暂停，搜索会开始并且焦点会从文本框中消失。 现在，必须通过按 Enter 键，或通过单击“开始搜索”按钮来显式启动搜索。
* 右键单击“文件共享”中的文件时，有时“获取共享访问签名”命令会处于禁用状态。 现在已修复此问题。
* 如果在搜索期间筛选掉具有焦点的资源树节点，将无法使用 tab 键访问资源树，也无法使用箭头键在资源树中导航。 现在，如果具有焦点的资源树节点被隐藏，资源树中的第一个节点将自动获得焦点。
* 有时会在编辑器工具栏中看到额外的分隔符。 现在已修复此问题。
* 痕迹导航文本框有时会溢出。 现在已修复此问题。
* 一次上传多个文件时，Blob 编辑器和文件共享编辑器有时会不断刷新。 现在已修复此问题。
* “文件夹统计信息”功能在“文件共享快照管理”视图中没有任何用途。 现在该功能已禁用。
* 在 Linux 上，“文件”菜单未显示。 现在已修复此问题。
* 将文件夹上传到“文件共享”时，默认情况下，只上传了该文件夹的内容。 现在，默认行为是将该文件夹的内容上传到“文件共享”中的相应文件夹。
* 几个对话框中的按钮的顺序是相反的。 现在已修复此问题。
* 各种与安全相关的修复。

### <a name="known-issues"></a>已知问题
* 在极少数情况下，树焦点可能会停滞在“快速访问”上。 要使焦点取消停滞，可以单击“全部刷新”。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Linux 用户，需要安装 [.NET Core 2.0](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-096"></a>版本 0.9.6
2018/02/28

### <a name="fixes"></a>修复项
* 此问题使所需的 blob/文件无法在编辑器中列出。 现在已修复此问题。
* 此问题导致在快照视图之间切换时无法正确显示项。 现在已修复此问题。

### <a name="known-issues"></a>已知问题
* 存储资源管理器不支持 ADFS 帐户。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是[此处](https://github.com/Azure/azure-storage-node/issues/317)介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-095"></a>版本 0.9.5
2018/02/06

### <a name="new"></a>新

* 对文件共享快照的支持：
    * 创建和管理文件共享快照。
    * 浏览时在文件共享快照之间轻松切换视图。
    * 还原文件的先前版本。
* 预览对 Azure Data Lake Store 的支持：
    * 跨多个帐户连接到 ADLS 资源。
    * 使用 ADL URI 连接到资源并共享 ADLS。
    * 按递归方式执行基本文件/文件夹操作。
    * 将单个文件夹固定到“快速访问”中。
    * 显示文件夹统计信息。

### <a name="fixes"></a>修复项
* 启动性能得到改进。
* 修复了多个 Bug。

### <a name="known-issues"></a>已知问题
* 存储资源管理器不支持 ADFS 帐户。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-094-and-093"></a>版本 0.9.4 和 0.9.3
2018 年 1 月 21 日

### <a name="new"></a>新
* 在以下情况下，将重新使用现有的存储资源管理器窗口：
    * 打开存储资源管理器中生成的直接链接。
    * 从门户打开存储资源管理器。
    * 从 Azure 存储 VS Code 扩展（即将推出）打开存储资源管理器。
* 增加了从存储资源管理器内部打开新存储资源管理器窗口的功能。
    * 对于 Windows，文件菜单下以及任务栏的上下文菜单中有一个“新建窗口”选项。
    * 对于 Mac，应用菜单下有一个“新建窗口”选项。

### <a name="fixes"></a>修复项
* 修复了安全性问题。 请尽早升级到 0.9.4。
* 以前不会适当地清理旧活动。 这影响了长时间运行的作业的性能。 现在会正确清理旧活动。
* 涉及大量文件和目录的操作有时可能导致存储资源管理器冻结。 现在限制了对 Azure 的文件共享请求，以限制系统资源的使用。

### <a name="known-issues"></a>已知问题
* 存储资源管理器不支持 ADFS 帐户。
* “查看资源管理器”和“查看帐户管理”的快捷键应当分别为 Ctrl/Cmd+Shift+E 和 Ctrl/Cmd+Shift+A。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer --disable-gpu
    ```

* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-092"></a>版本 0.9.2
2017/11/01

### <a name="hotfixes"></a>修补程序
* 根据本地时区编辑表实体的 Edm.DateTime 值时，可能发生意外的数据更改。 编辑器现使用纯文本文本框，以便始终如一地精确控制 Edm.DateTime 值。
* 在附有名称和密钥时无法启动一组 blob 的上传/下载。 现在已修复此问题。
* 如果选择了帐户的一个或多个订阅，以前的存储资源管理器仅会提示重新验证过时帐户。 现在，即使完全筛选掉该帐户，存储资源管理器仍会提示。
* 适用于 Azure 美国政府版的终结点域不正确。 此问题已解决。
* 有时很难单击“管理帐户”面板上的“应用”按钮。 不会再发生此问题。

### <a name="new"></a>新
* 预览版对 Azure Cosmos DB 的支持：
    * [联机文档](./cosmos-db/storage-explorer.md)
    * 创建数据库和集合
    * 操作数据
    * 查询、创建或删除文档
    * 更新存储过程、用户定义的函数或触发器
    * 使用连接字符串连接到并管理数据库
* 改善了上传/下载许多小型 blob 时的性能。
* 当 blob 上传组或 blob 下载组中出现故障时增加了“全部重试”操作。
* 现在，在 blob 上传/下载期间，如果检测到网络连接断开，则存储资源管理器会暂停迭代。 然后，在重新建立网络连接后，你可以恢复迭代。
* 增加了通过上下文菜单“全部关闭”、“关闭其他”和“关闭”选项卡的能力。
* 存储资源管理器现在使用本机对话框和本机上下文菜单。
* 存储资源管理器现在更易于访问。 改进包括：
    * 针对 Windows 上的 NVDA 和 Mac 上的 VoiceOver，改进了屏幕阅读器支持
    * 改进了高对比度主题
    * 键盘 Tab 键次序和键盘焦点修复

### <a name="fixes"></a>修复项
* 过去，如果尝试下载具有无效 Windows 文件名的 blob，则操作将失败。 现在，存储资源管理器将检测 blob 名称是否有效，并且会询问你是要对其进行编码还是跳过该 blob。 存储资源管理器还会检测文件名看起来是否已编码并且在上传前会询问是否要对其进行解码。
* 在上传 blob 期间，目标 blob 容器的编辑器有时候不能正确刷新。 现在已修复此问题。
* 支持多种形式的连接字符串和回归的 SAS URI。 我们已解决了所有已知问题，但是如果遇到了进一步的问题，请向我们发送反馈。
* 在 0.9.0 版中，对于某些用户，更新通知损坏。 此问题已修复，并且受此 bug 影响的用户可[从此处](https://azure.microsoft.com/features/storage-explorer/)手动下载最新版本的存储资源管理器。

### <a name="known-issues"></a>已知问题
* 存储资源管理器不支持 ADFS 帐户。
* “查看资源管理器”和“查看帐户管理”的快捷键应当分别为 Ctrl/Cmd+Shift+E 和 Ctrl/Cmd+Shift+A。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer --disable-gpu
    ```

* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-091-and-090"></a>版本 0.9.1 和 0.9.0
10/20/2017
### <a name="new"></a>新
* 预览版对 Azure Cosmos DB 的支持：
    * [联机文档](./cosmos-db/storage-explorer.md)
    * 创建数据库和集合
    * 操作数据
    * 查询、创建或删除文档
    * 更新存储过程、用户定义的函数或触发器
    * 使用连接字符串连接到并管理数据库
* 改善了上传/下载许多小型 blob 时的性能。
* 当 blob 上传组或 blob 下载组中出现故障时增加了“全部重试”操作。
* 现在，在 blob 上传/下载期间，如果检测到网络连接断开，则存储资源管理器会暂停迭代。 然后，在重新建立网络连接后，你可以恢复迭代。
* 增加了通过上下文菜单“全部关闭”、“关闭其他”和“关闭”选项卡的能力。
* 存储资源管理器现在使用本机对话框和本机上下文菜单。
* 存储资源管理器现在更易于访问。 改进包括：
    * 针对 Windows 上的 NVDA 和 Mac 上的 VoiceOver，改进了屏幕阅读器支持
    * 改进了高对比度主题
    * 键盘 Tab 键次序和键盘焦点修复

### <a name="fixes"></a>修复项
* 过去，如果尝试下载具有无效 Windows 文件名的 blob，则操作将失败。 现在，存储资源管理器将检测 blob 名称是否有效，并且会询问你是要对其进行编码还是跳过该 blob。 存储资源管理器还会检测文件名看起来是否已编码并且在上传前会询问是否要对其进行解码。
* 在上传 blob 期间，目标 blob 容器的编辑器有时候不能正确刷新。 现在已修复此问题。
* 支持多种形式的连接字符串和回归的 SAS URI。 我们已解决了所有已知问题，但是如果遇到了进一步的问题，请向我们发送反馈。
* 在 0.9.0 版中，对于某些用户，更新通知损坏。 此问题已修复，对于受此 bug 影响的那些用户，可以手动[从此处](https://azure.microsoft.com/features/storage-explorer/)下载最新版本的存储资源管理器。

### <a name="known-issues"></a>已知问题
* 存储资源管理器不支持 ADFS 帐户。
* “查看资源管理器”和“查看帐户管理”的快捷键应当分别为 Ctrl/Cmd+Shift+E 和 Ctrl/Cmd+Shift+A。
* 当以 Azure Stack 为目标时，将某些文件作为追加 blob 进行上传可能会失败。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是因为我们使用的是此处介绍的“取消筛选”解决办法。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 存储资源管理器使用的 Electron shell 在进行某项 GPU（图形处理单元）硬件加速时出现问题。 如果存储资源管理器显示了一个空白（空的）主窗口，则可以尝试从命令行启动存储资源管理器，并通过添加 `--disable-gpu` 开关禁用 GPU 加速。

    ```
    ./StorageExplorer --disable-gpu
    ```

* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-0816"></a>版本 0.8.16
8/21/2017

### <a name="new"></a>新
* 打开 Blob 时，如果检测到更改，存储资源管理器会提示上传已下载的文件
* 增强的 Azure Stack 登录体验
* 改善了同时上传/下载多个小型文件时的性能


### <a name="fixes"></a>修复项
* 对于某些 Blob 类型，在发生上传冲突期间选择“替换”有时会导致重新开始上传。
* 在版本 0.8.15 中，上传有时会停滞在 99%。
* 将文件上传到文件共享时，如果选择上传到某个不存在的目录，上传将会失败。
* 存储资源管理器为共享访问签名和表查询生成错误的时间戳。


### <a name="known-issues"></a>已知问题
* 目前无法正常使用名称和密钥连接字符串。 下一个版本会修复此问题。 在此之前，可以使用附加名称和密钥的功能。
* 如果尝试打开具有无效 Windows 文件名的文件，下载操作会导致“找不到文件”错误。
* 对任务单击“取消”后，可能需要一段时间才能取消该任务。 这是 Azure 存储节点库的局限性。
* 完成 Blob 上传后，系统会刷新启动上传的选项卡。 这是对以前行为的更改，也将导致回转到所在容器的根目录中。
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器，使其忘记该选择。
* 帐户设置面板可能显示需重新输入凭据以筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* 对于 Ubuntu 14.04 用户，需确保 GCC 是最新版本 - 为此，可运行以下命令并重启计算机：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* 对于 Ubuntu 17.04 用户，需要安装 GConf - 通过运行以下命令，然后重启计算机即可完成：

    ```
    sudo apt-get install libgconf-2-4
    ```

### <a name="version-0814"></a>版本 0.8.14
2017/06/22

### <a name="new"></a>新

* 已将 Electron 版本更新为 1.7.2，以便利用若干关键安全更新
* 现可从帮助菜单快速访问联机疑难解答指南
* 存储资源管理器疑难解答[指南][2]
* 连接到 Azure Stack 订阅相关[说明][3]

### <a name="known-issues"></a>已知问题

* 用鼠标单击 Linux 中删除文件夹确认对话框上的按钮无反应。 解决方法是使用 Enter 键
* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择
* 同时上传 3 组以上的 blob 或文件可能导致错误
* 帐户设置面板可能显示需重新输入凭据才能筛选订阅
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* Ubuntu 14.04 安装需要更新或升级 gcc 版本 - 升级步骤如下：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

### <a name="version-0813"></a>版本 0.8.13
05/12/2017

#### <a name="new"></a>新

* 存储资源管理器疑难解答[指南][2]
* 连接到 Azure Stack 订阅相关[说明][3]

#### <a name="fixes"></a>修复项

* 已修复：文件上传很可能导致内存不足的错误
* 已修复：现可使用 PIN/智能卡登录
* 已修复：在门户中打开现适用于 Azure 中国世纪互联、Azure Germany、Azure US Government 和 Azure Stack
* 已修复：将文件夹上传到 blob 容器中时，“非法操作”错误时有发生
* 已修复：管理快照时禁用“选择全部”
* 已修复：查看基础 blob 快照的属性后，可能会覆盖其元数据

#### <a name="known-issues"></a>已知问题

* 如果选择错误的 PIN/智能卡证书，需要重启存储资源管理器使其忘记该选择
* 放大或缩小时，缩放级别可能暂时重置为默认级别
* 同时上传 3 组以上的 blob 或文件可能导致错误
* 帐户设置面板可能显示需重新输入凭据才能筛选订阅
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* Ubuntu 14.04 安装需要更新或升级 gcc 版本 - 升级步骤如下：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```


### <a name="version-0812-and-0811-and-0810"></a>版本 0.8.12、0.8.11 和 0.8.10
2017/04/07

#### <a name="new"></a>新

* 如今，如果从更新通知安装更新，存储资源管理器会自动关闭
* 对使用经常访问的资源而言，就地快速访问提供增强的体验
* 在 Blob 容器编辑器中，现可查看租用 blob 所属的虚拟机
* 现可折叠左侧面板
* 发现现与下载同时运行
* 在 Blob 容器、文件共享和表编辑器中使用统计信息来查看资源或所选内容的大小
* 现可登录到基于 Azure Active Directory (AAD) 的 Azure Stack 帐户。
* 现可将超过 32 MB 的存档文件上传到高级存储帐户
* 改进的辅助功能支持
* 现可通过转到“编辑”-&gt;“SSL 证书”-&gt;“导入证书”来添加受信任的 Base-64 编码 X.509 SSL 证书

#### <a name="fixes"></a>修复项

* 已修复：刷新帐户凭据后，树状视图有时不会自动刷新
* 已修复：为仿真器队列和表生成 SAS 会导致无效的 URL
* 已修复：现可在启用代理时扩展高级存储帐户
* 已修复：如果选择了 1 个或 0 个帐户，帐户管理页上的应用按钮无效
* 已修复：上传需要解决冲突的 blob 可能失败 - 已在 0.8.11 中修复
* 已修复：0.8.11 中正在发送的反馈损坏 - 已在 0.8.12 中修复

#### <a name="known-issues"></a>已知问题

* 升级到 0.8.10 后，需要刷新所有凭据。
* 放大或缩小时，缩放级别可能暂时重置为默认级别。
* 同时上传 3 组以上的 blob 或文件可能导致错误。
* 帐户设置面板可能显示需重新输入凭据才能筛选订阅。
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据。
* 尽管 Azure Stack 当前不支持文件共享，但附加 Azure Stack 存储帐户下仍会显示“文件共享”节点。
* Ubuntu 14.04 安装需要更新或升级 gcc 版本 - 升级步骤如下：

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```


### <a name="version-089-and-088"></a>版本 0.8.9 和 0.8.8
2017/02/23

>[!VIDEO https://www.youtube.com/embed/R6gonK3cYAc?ecver=1]

>[!VIDEO https://www.youtube.com/embed/SrRPCm94mfE?ecver=1]


#### <a name="new"></a>新

* 存储资源管理器 0.8.9 自动下载最新版本进行更新。
* 修补程序：使用门户生成的 SAS URI 附加存储帐户会导致错误。
* 现可创建、管理和提升 blob 快照。
* 现可登录 Azure 中国世纪互联、Azure Germany、Azure US Government 帐户。
* 现可更改缩放级别。 使用“视图”菜单中的选项放大、缩小和重置缩放。
* 现支持在 blob 和文件的用户元数据中使用 Unicode 字符。
* 辅助功能改进。
* 可从更新通知中查看下一版本的发行说明。 还可从“帮助”菜单查看当前发行说明。

#### <a name="fixes"></a>修复项

* 已修复：版本号现正确显示在 Windows“控制面板”中
* 已修复：搜索不再限制为 50,000 个节点
* 已修复：如果目标目录不存在，上传到文件共享始终处于旋转状态
* 已修复：提升了长时间上传和下载的稳定性

#### <a name="known-issues"></a>已知问题

* 放大或缩小时，缩放级别可能暂时重置为默认级别。
* 快速访问仅适用于基于订阅的项。 此版本不支持本地资源或通过密钥或 SAS 令牌附加的资源。
* 快速访问可能需要几秒钟时间导航至目标资源，具体取决于所拥有的资源数。
* 同时上传 3 组以上的 blob 或文件可能导致错误。

2016/12/16
### <a name="version-087"></a>版本 0.8.7

>[!VIDEO https://www.youtube.com/embed/Me4Y4jxoer8?ecver=1]

#### <a name="new"></a>新

* 在“活动”窗口中，可选择如何在更新、下载或复制会话开始时解决冲突
* 将鼠标悬停在选项卡上，以查看存储资源的完整路径
* 单击选项卡时，会在左侧导航窗格中同步其位置

#### <a name="fixes"></a>修复项

* 已修复：存储资源管理器在 Mac 上现已是受信任的应用
* 已修复：再次支持 Ubuntu 14.04
* 已修复：加载订阅时“添加帐户”UI 有时会闪烁
* 已修复：左侧导航窗格中有时不会列出所有的存储资源
* 已修复：操作窗格有时显示空白操作
* 已修复：现在会保留最后关闭的会话窗口大小
* 已修复：现可使用上下文菜单打开同一资源的多个选项卡

#### <a name="known-issues"></a>已知问题

* 快速访问仅适用于基于订阅的项。 此版本不支持本地资源或通过密钥或 SAS 令牌附加的资源
* 快速访问可能需要几秒钟时间导航至目标资源，具体取决于所拥有的资源数
* 同时上传 3 组以上的 Blob 或文件可能导致错误
* 搜索功能会处理作用范围约 50,000 个节点的搜索操作，之后，如超过此范围，则性能可能会受影响，或导致未经处理的异常
* 第一次在 macOS 上使用存储资源管理器时，可能会出现多个请求用户提供密钥链访问权限的提示。 建议选择“始终允许”，使提示不再显示

2016/11/18
### <a name="version-086"></a>版本 0.8.6

#### <a name="new"></a>新

* 现可将最常用服务固定到快速访问，从而实现轻松导航
* 现可在不同选项卡中打开多个编辑器。 单击可打开临时选项卡；双击可打开永久选项卡。也可通过单击临时选项卡，将其变为永久选项卡
* 已显著提升上传和下载的性能及稳定性，尤其是对于快速计算机上的大文件
* 现可在 blob 容器中创建空白“虚拟”文件夹
* 已重新引入通过新的增强子字符串搜索进行的限定范围搜索，因此现可通过两种方式进行搜索：
    * 全局搜索 - 在搜索文本框中输入搜索词即可
    * 限定范围的搜索 - 单击节点旁边的放大镜图标，然后在路径末尾添加一个搜索词，或右键单击并选择“从此处搜索”
* 我们已添加多个主题：浅色（默认）、深色、高对比度黑色和高对比度白色。 转到“编辑”-&gt;“主题”可更改主题首选项
* 可修改 Blob 和文件属性
* 现在支持编码 (base64) 和未编码的队列消息
* 对于 Linux，现要求 64 位操作系统。 对于此版本，仅支持 64 位 Ubuntu 16.04.1 LTS
* 已更新徽标！

#### <a name="fixes"></a>修复项

* 已修复：屏幕死机问题
* 已修复：增强的安全性
* 已修复：有时可能会出现重复的附加帐户
* 已修复：具有未定义内容类型的 blob 可能产生异常
* 已修复：不能在空表上打开“查询面板”
* 已修复：“搜索”中的各种 bug
* 已修复：单击“加载更多”时，加载的资源数从 50 增加到了 100
* 已修复：首次运行时，如已登录帐户，现默认选择该帐户的所有订阅

#### <a name="known-issues"></a>已知问题

* 此版本的存储资源管理器不在 Ubuntu 14.04 上运行
* 若要为同一资源打开多个选项卡，请勿连续单击同一资源。 单击另一资源，然后返回，然后单击原始资源以在另一选项卡中再次打开
* 快速访问仅适用于基于订阅的项。 此版本不支持本地资源或通过密钥或 SAS 令牌附加的资源
* 快速访问可能需要几秒钟时间导航至目标资源，具体取决于所拥有的资源数
* 同时上传 3 组以上的 Blob 或文件可能导致错误
* 搜索功能会处理作用范围约 50,000 个节点的搜索操作，之后，如超过此范围，则性能可能会受影响，或导致未经处理的异常

2016/10/03
### <a name="version-085"></a>版本 0.8.5

#### <a name="new"></a>新

* 现可使用门户生成的 SAS 密钥附加到存储帐户和资源

#### <a name="fixes"></a>修复项

* 已修复：搜索期间的争用条件有时导致节点无法展开
* 已修复：使用帐户名和密钥连接到存储帐户时，“使用 HTTP”无效
* 已修复：SAS 密钥（特别是门户生成的密钥）返回“尾部反斜杠”错误
* 已修复：表格导入问题
    * 有时分区键和行键会反转
    * 无法读取“null”分区键

#### <a name="known-issues"></a>已知问题

* 搜索功能会处理作用范围约 50,000 个节点的搜索操作，之后，如超过此范围，则性能可能会受影响
* Azure Stack 目前不支持文件，因此尝试展开“文件”会显示错误

2016/09/12
### <a name="version-084"></a>版本 0.8.4

>[!VIDEO https://www.youtube.com/embed/cr5tOGyGrIQ?ecver=1]

#### <a name="new"></a>新

* 生成指向存储帐户、容器、队列、表或文件共享的直接链接，以便共享和轻松访问资源 - Windows 和 Mac 操作系统支持
* 从搜索框搜索 blob 容器、表、队列、文件共享或存储帐户
* 现可在表查询生成器中对子句进行分组
* 从 SAS 附加帐户和容器中重命名并复制/粘贴 blob 容器、文件共享、表、blob、blob 文件夹、文件和目录
* 重命名和复制 blob 容器和文件共享现可保留属性和元数据

#### <a name="fixes"></a>修复项

* 已修复：表实体包含布尔或二进制属性时无法编辑

#### <a name="known-issues"></a>已知问题

* 搜索功能会处理作用范围约 50,000 个节点的搜索操作，之后，如超过此范围，则性能可能会受影响

2016/08/03
### <a name="version-083"></a>版本 0.8.3

>[!VIDEO https://www.youtube.com/embed/HeGW-jkSd9Y?ecver=1]

#### <a name="new"></a>新建

* 重命名容器、表和文件共享
* 提升了查询生成器体验
* 能够保存和加载查询
* 存储帐户或容器、队列、表或文件共享的直接链接，以便共享和轻松访问资源（即将推出仅 Windows 和 macOS 支持！）
* 能够管理和配置 CORS 规则

#### <a name="fixes"></a>修复项

* 已修复：Microsoft 帐户每 8-12 小时需要重新进行身份验证

#### <a name="known-issues"></a>已知问题

* 有时 UI 可能出现冻结 - 窗口最大化可帮助解决此问题
* macOS 安装可能需要提升的权限
* 帐户设置面板可能显示需重新输入凭据才能筛选订阅
* 重命名文件共享、blob 容器和表不会保留容器中的元数据或其他属性，例如文件共享配额、公共访问级别或访问策略
* 重命名 blob（单独地或在已重命名的 blob 容器中）不保留快照。 重命名期间保留 blob、文件和实体的所有其他属性和元数据
* 在 SAS 附加帐户中，复制或重命名资源无法正常工作

2016/07/07
### <a name="version-082"></a>版本 0.8.2

>[!VIDEO https://www.youtube.com/embed/nYgKbRUNYZA?ecver=1]

#### <a name="new"></a>新

* 存储帐户按订阅分组；通过密钥或 SAS 附加的开发存储和资源显示在（本地和附加）节点下
* 从“Azure 帐户设置”面板注销帐户
* 配置代理设置以启用和管理登录
* 创建和中断 blob 租用
* 通过单击打开 blob 容器、队列、表和文件

#### <a name="fixes"></a>修复项

* 已修复：使用 .NET 或 Java 库插入的队列消息未从 base64 正确解码
* 已修复：不向 Blob 存储帐户显示 $metrics 表
* 已修复：表节点不适用于本地（开发）存储

#### <a name="known-issues"></a>已知问题

* macOS 安装可能需要提升的权限

2016/06/15
### <a name="version-080"></a>版本 0.8.0

>[!VIDEO https://www.youtube.com/embed/ycfQhKztSIY?ecver=1]

>[!VIDEO https://www.youtube.com/embed/k4_kOUCZ0WA?ecver=1]

>[!VIDEO https://www.youtube.com/embed/3zEXJcGdl_k?ecver=1]

#### <a name="new"></a>新

* 文件共享支持：查看、上传、下载、复制文件和目录以及 SAS URI（创建和连接）
* 改进了使用 SAS URI 或帐户密钥连接到存储的用户体验
* 导出表查询结果
* 表列重新排序和自定义
* 查看含已启用指标的存储帐户 $logs blob 容器和 $metrics 表
* 改进了导出和导入行为，现包括属性值类型

#### <a name="fixes"></a>修复项

* 已修复：上传或下载大型 blob 可能导致上传/下载不完整
* 已修复：编辑、添加或导入具有数值字符串值（“1”）的实体会将其转换为双精度
* 已修复：无法在本地开发环境中展开表节点

#### <a name="known-issues"></a>已知问题

* Blob 存储帐户不可查看 $metrics 表
* 如果使用 Base64 编码对消息编码，则以编程方式添加的队列消息可能无法正确显示

2016/05/17
### <a name="version-07201605090"></a>版本 0.7.20160509.0

#### <a name="new"></a>新

* 更好的应用崩溃错误处理

#### <a name="fixes"></a>修复项

* 修复了需要登录凭据时信息栏消息有时不显示的 bug

#### <a name="known-issues"></a>已知问题

* 表：添加、编辑或导入其属性具有不明确数值（如“1”或“1.0”）的实体，并且用户尝试将其作为 `Edm.String` 发送，该值将通过客户端 API 作为 Edm.Double 返回

2016/03/31

### <a name="version-07201603250"></a>版本 0.7.20160325.0

>[!VIDEO https://www.youtube.com/embed/imbgBRHX65A?ecver=1]

>[!VIDEO https://www.youtube.com/embed/ceX-P8XZ-s8?ecver=1]

#### <a name="new"></a>新

* 表支持：实体的查看、查询、导出、导入和 CRUD 操作
* 队列支持：查看、添加消息，以及取消消息排队
* 生成存储帐户的 SAS URI
* 使用 SAS URI 连接到存储帐户
* 存储资源管理器未来更新的更新通知
* 更新的外观

#### <a name="fixes"></a>修复项

* 性能和可靠性提升

### <a name="known-issues-amp-mitigations"></a>已知问题 &amp; 缓解措施

* 大型 blob 文件无法正常下载 - 建议在我们处理此问题期间使用 AzCopy
* 如果无法找到或无法写入主文件夹，则不会检索或缓存帐户凭据
* 如果正在添加、编辑或导入具有不明确数值（如“1”或“1.0”）的属性的实体，并且用户尝试将其作为 `Edm.String` 发送，则该值将通过客户端 API 作为 Edm.Double 返回
* 导入包含多行记录的 CSV 文件时，数据可能被截或加密

2016/02/03

### <a name="version-07201601291"></a>版本 0.7.20160129.1

#### <a name="fixes"></a>修复项

* 上传、下载和复制 blob 时整体性能提升

2016/01/14

### <a name="version-07201601050"></a>版本 0.7.20160105.0

#### <a name="new"></a>新

* Linux 支持（OSX 的奇偶校验功能）
* 添加具有共享访问签名 (SAS) 密钥的 blob 容器
* 添加 Azure 中国世纪互联的存储帐户
* 添加具有自定义终结点的存储帐户
* 打开并查看内容文本和图片 blob
* 查看和编辑 blob 属性和元数据

#### <a name="fixes"></a>修复项

* 已修复：上传或下载大量 blob (500+) 有时可能导致应用白屏
* 已修复：设置 blob 容器公共访问级别时，新值在重置容器焦点前不会更新。 此外，对话框始终默认为“无公共访问”，而不是实际的当前值。
* 总体更佳的键盘/辅助功能和 UI 支持
* 痕迹导航历史记录文本在出现较长空格时换行
* SAS 对话框支持输入验证
* 即使用户凭据已过期，本地存储仍然可用
* 删除打开的 blob 容器后，右侧的 blob 资源管理器关闭

#### <a name="known-issues"></a>已知问题

* Linux 安装需要更新或升级 gcc 版本 - 升级步骤如下：
    * `sudo add-apt-repository ppa:ubuntu-toolchain-r/test`
    * `sudo apt-get update`
    * `sudo apt-get upgrade`
    * `sudo apt-get dist-upgrade`

11/18/2015
### <a name="version-07201511160"></a>版本 0.7.20151116.0

#### <a name="new"></a>新

* macOS 和 Windows 版本
* 登录以查看存储帐户 - 使用组织帐户、Microsoft 帐户、2FA 等。
* 本地开发存储（使用存储模拟器，仅限 Windows）
* Azure 资源管理器和经典资源支持
* 创建和删除 blob、队列或表
* 搜索特定 blob、队列或表
* 浏览 blob 容器的内容
* 查看和浏览目录
* 上传、下载和删除 blob 和文件夹
* 查看和编辑 blob 属性和元数据
* 生成 SAS 密钥
* 管理和创建存储访问策略 (SAP)
* 按前缀搜索 blob
* 拖放文件进行上传或下载

#### <a name="known-issues"></a>已知问题

* 设置 blob 容器公共访问级别时，新值在重置容器焦点前不会更新
* 打开设置公共访问级别的对话框时，始终显示“无公共访问”作为默认值，而不是实际的当前值
* 无法重命名已下载 blob
* 出现错误且不显示该错误时，活动日志条目有时会“停滞”在正在进行状态
* 尝试上传或下载大量 blob 时，系统有时会崩溃或完全白屏
* 取消复制操作有时无效
* 在创建容器（blob/队列/表）期间，如果输入无效名称并继续创建不同容器类型的另一容器，则不能对新类型设置焦点
* 无法创建新文件夹或重命名文件夹




[2]: https://support.microsoft.com/en-us/help/4021389/storage-explorer-troubleshooting-guide
[3]: vs-azure-tools-storage-manage-with-storage-explorer.md
