---
title: 数据复制或移动到 Azure 存储使用 AzCopy v10 |Microsoft Docs
description: 使用 AzCopy 命令行实用工具来移动或复制数据到或从 blob、 data lake 和文件内容。 从本地文件将数据复制到 Azure 存储，或者在存储帐户中或存储帐户之间复制数据。 轻松地将数据迁移到 Azure 存储。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 04/23/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: d05cbd30565ced73352736508fc4bcc376985554
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149033"
---
# <a name="transfer-data-with-azcopy-v10"></a>使用 AzCopy v10 传输数据

AzCopy 是用于向 / 从 Microsoft Azure Blob 和文件存储复制数据的命令行实用程序。 AzCopy 提供了一个经过重新设计的命令行接口，并将新的体系结构的可靠的数据传输。 使用 AzCopy 可在文件系统与存储帐户之间或者在存储帐户之间复制数据。

## <a name="whats-new-in-azcopy-v10"></a>AzCopy v10 中的新增功能

- 将文件系统同步到 Azure Blob 存储，或反之。 使用 `azcopy sync <source> <destination>`。 非常适合增量复制方案。
- 支持 Azure Data Lake Storage Gen2 API。 使用`myaccount.dfs.core.windows.net`为调用 Data Lake 存储第 2 代 Api 的 URI。
- 支持将整个帐户（仅限 Blob 服务）复制到另一个帐户。
- 支持从 Amazon Web Services S3 存储桶中复制数据。
- 使用新的[从 URL 放置块](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API 来支持帐户到帐户的复制。 无需向客户端传输数据，因此数据传输速度更快。
- 列出或删除给定路径中的文件和 Blob。
- 支持在路径中使用通配符模式，并支持 --exclude 标志。
- 使用每个 AzCopy 实例创建作业顺序和相关的日志文件。 可以查看和重启以前的作业，以及恢复失败的作业。 AzCopy 还会在失败后自动重试传输。
- 提供常规性能改进。

## <a name="download-and-install-azcopy"></a>下载并安装 AzCopy

### <a name="latest-production-version-v10"></a>最新生产版本 (v10)

下载最新版本的 AzCopy:
- [Windows](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Linux](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

### <a name="latest-azcopy-supporting-table-storage-service-v73"></a>支持表存储服务 (v7.3) 的最新 AzCopy

下载[支持向/从 Microsoft Azure 表存储服务复制数据的 AzCopy v7.3](https://aka.ms/downloadazcopynet)。

## <a name="post-installation-steps"></a>安装后的步骤

AzCopy 不需要进行安装。 只需打开首选的命令行应用程序，并浏览到 `azcopy.exe` 所在的文件夹。 如果需要，可将 AzCopy 文件夹位置添加到系统路径以方便使用。

## <a name="authentication-options"></a>身份验证选项

使用 Azure 存储进行身份验证时，AzCopy 支持以下选项：
- **Azure Active Directory** (适用于**Blob 和数据湖存储第 2 代服务**)。 使用```.\azcopy login```使用 Azure Active Directory 进行登录。  用户应具有["存储 Blob 数据参与者"角色分配](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac)要写入到 Blob 存储与 Azure Active Directory 身份验证。 对于通过 Azure 资源的管理的标识的身份验证，使用`azcopy login --identity`。
- **共享访问签名令牌 [支持 Blob 和文件服务]**。 在命令行中将共享访问签名 (SAS) 令牌追加到 Blob 路径即可使用该令牌。 可以使用 Azure 门户、[存储资源管理器](https://blogs.msdn.microsoft.com/jpsanders/2017/10/12/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer/)、[PowerShell](https://docs.microsoft.com/powershell/module/az.storage/new-azstorageblobsastoken) 或所选的其他工具生成 SAS 令牌。 有关详细信息，请参阅[示例](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2)。

## <a name="getting-started"></a>入门

> [!TIP]
> **更喜欢图形用户界面？**
>
> 在可以简化 Azure 存储数据管理的桌面客户端 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)中，可以使用 AzCopy 来加速与 Azure 存储之间进行的数据传输。
>
> 在存储资源管理器中的“预览”菜单下启用 AzCopy。
> ![在 Azure 存储资源管理器中启用 AzCopy 作为传输引擎](media/storage-use-azcopy-v10/enable-azcopy-storage-explorer.jpg)

AzCopy v10 具有自有案可稽的语法。 当已登录到 Azure Active Directory 时，常规语法看起来如下所示：

```azcopy
.\azcopy <command> <arguments> --<flag-name>=<flag-value>

# Examples if you have logged into the Azure Active Directory:
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/container" --recursive=true
.\azcopy cp "C:\local\path\myfile" "https://account.blob.core.windows.net/container/myfile"
.\azcopy cp "C:\local\path\*" "https://account.blob.core.windows.net/container"

# Examples if you're using SAS tokens to authenticate:
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/container?st=2019-04-05T04%3A10%3A00Z&se=2019-04-13T04%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=Qdihej%2Bsbg4AiuyLVyQZklm9pSuVGzX27qJ508wi6Es%3D" --recursive=true
.\azcopy cp "C:\local\path\myfile" "https://account.blob.core.windows.net/container/myfile?st=2019-04-05T04%3A10%3A00Z&se=2019-04-13T04%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=Qdihej%2Bsbg4AiuyLVyQZklm9pSuVGzX27qJ508wi6Es%3D"
```

以下语法介绍如何获取可用命令的列表：

```azcopy
.\azcopy --help
# To use the alias instead
.\azcopy -h
```

若要查看特定命令的帮助页和示例，请运行以下命令：

```azcopy
.\azcopy <cmd> --help
# Example:
.\azcopy cp -h
```

## <a name="create-a-blob-container-or-file-share"></a>创建 blob 容器或文件共享 

**创建 blob 容器**

```azcopy
.\azcopy make "https://account.blob.core.windows.net/container-name"
```

**创建文件共享**

```azcopy
.\azcopy make "https://account.file.core.windows.net/share-name"
```

**使用 Azure 数据湖存储第 2 代创建 blob 容器**

如果已在 Blob 存储帐户上启用分层命名空间，可以使用以下命令以创建新的 blob 容器上传的文件。

```azcopy
.\azcopy make "https://account.dfs.core.windows.net/top-level-resource-name"
```

## <a name="copy-data-to-azure-storage"></a>将数据复制到 Azure 存储

使用 copy 命令将数据从源传输到目标。 源或目标可以是：
- 本地文件系统
- Azure Blob/虚拟目录/容器 URI
- Azure 文件/目录/文件共享 URI
- Azure Data Lake Storage Gen2 文件系统/目录/文件 URI

```azcopy
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
# Using the alias instead 
.\azcopy cp <source path> <destination path> --<flag-name>=<flag-value>
```

以下命令将该文件夹下的所有文件上都传`C:\local\path`到容器的递归`mycontainer1`，则创建`path`目录容器中。 当`--put-md5`提供标志、 AzCopy 计算并存储中的每个文件的 md5 哈希`Content-md5`属性对应的 blob 以供将来使用。

```azcopy
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true --put-md5
```

以下命令将文件夹 `C:\local\path` 下的所有文件上传到容器 `mycontainer1`（不递归到子目录中）：

```azcopy
.\azcopy cp "C:\local\path\*" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --put-md5
```

若要查找更多示例，请使用以下命令：

```azcopy
.\azcopy cp -h
```

## <a name="copy-blob-data-between-two-storage-accounts"></a>将两个存储帐户之间的 Blob 数据复制

在两个存储帐户之间复制数据使用的是[从 URL 放置块](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API，而不使用客户端计算机的网络带宽。 直接在两个 Azure 存储服务器之间复制数据，而 AzCopy 只是协调复制操作。 此选项目前仅适用于 Blob 存储。

若要复制所有两个存储帐户之间的 Blob 数据，请使用以下命令：
```azcopy
.\azcopy cp "https://myaccount.blob.core.windows.net/<sastoken>" "https://myotheraccount.blob.core.windows.net/<sastoken>" --recursive=true
```

若要将 Blob 容器复制到另一个 Blob 容器，使用以下命令：
```azcopy
.\azcopy cp "https://myaccount.blob.core.windows.net/mycontainer/<sastoken>" "https://myotheraccount.blob.core.windows.net/mycontainer/<sastoken>" --recursive=true
```

## <a name="copy-a-vhd-image-to-a-storage-account"></a>将 VHD 映像复制到存储帐户

默认情况下 AzCopy 将数据上载到块 blob。 若要上传文件，因为追加 Blob 还是页 Blob 使用标志`--blob-type=[BlockBlob|PageBlob|AppendBlob]`。

```azcopy
.\azcopy cp "C:\local\path\mydisk.vhd" "https://myotheraccount.blob.core.windows.net/mycontainer/mydisk.vhd<sastoken>" --blob-type=PageBlob
```

## <a name="sync-incremental-copy-and-delete-blob-storage-only"></a>同步：增量复制和删除（仅适用于 Blob 存储）

sync 命令通过比较文件名和上次修改时间戳将源目录的内容同步到目标中的目录。 在提供 `--delete-destination=prompt|true` 标志的情况下，可选择性地通过此操作删除目标文件（如果这些文件在源中不存在）。 默认会禁用删除行为。 

> [!NOTE] 
> 请谨慎使用 `--delete-destination` 标志。 在同步操作中启用删除行为之前，请启用[软删除](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete)功能，防止帐户中发生意外删除。 
>
> 将 `--delete-destination` 设置为 true 时，AzCopy 会在不提示用户的情况下，从目标中删除源中不存在的文件。 如果希望系统提示确认，请使用 `--delete-destination=prompt`。

若要将本地文件系统同步到存储帐户，请使用以下命令：

```azcopy
.\azcopy sync "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true
```

还可将 Blob 容器同步到本地文件系统：

```azcopy
# The SAS token isn't required for Azure Active Directory authentication.
.\azcopy sync "https://account.blob.core.windows.net/mycontainer1" "C:\local\path" --recursive=true
```

此命令根据上次修改时间戳以增量方式将源同步到目标。 如果添加或删除源中的文件，AzCopy 将执行相同操作的目标中。 在删除之前，AzCopy 会提示确认。

## <a name="copy-data-from-amazon-web-services-aws-s3"></a>从 Amazon Web Services (AWS) S3 复制数据

若要使用 AWS S3 存储桶进行身份验证，设置以下环境变量：

```
# For Windows:
set AWS_ACCESS_KEY_ID=<your AWS access key>
set AWS_SECRET_ACCESS_KEY=<AWS secret access key>
# For Linux:
export AWS_ACCESS_KEY_ID=<your AWS access key>
export AWS_SECRET_ACCESS_KEY=<AWS secret access key>
# For MacOS
export AWS_ACCESS_KEY_ID=<your AWS access key>
export AWS_SECRET_ACCESS_KEY=<AWS secret access key>
```

若要复制到 Blob 容器的存储桶，发出以下命令：

```
.\azcopy cp "https://s3.amazonaws.com/mybucket" "https://myaccount.blob.core.windows.net/mycontainer?<sastoken>" --recursive
```

有关使用 AzCopy，从 AWS S3 的数据复制的详细信息，请参阅页面[此处](https://github.com/Azure/azure-storage-azcopy/wiki/Copy-from-AWS-S3)。

## <a name="advanced-configuration"></a>高级配置

### <a name="configure-proxy-settings"></a>配置代理设置

若要配置 AzCopy v10 的代理设置，请使用以下命令设置环境变量 https_proxy：

```cmd
# For Windows:
set https_proxy=<proxy IP>:<proxy port>
# For Linux:
export https_proxy=<proxy IP>:<proxy port>
# For MacOS
export https_proxy=<proxy IP>:<proxy port>
```

### <a name="optimize-throughput"></a>优化吞吐量

设置环境变量 AZCOPY_CONCURRENCY_VALUE，以配置并发请求数并控制吞吐量性能和资源消耗。 默认情况下，该值设置为 300。 减小该值将限制 AzCopy v10 使用的带宽和 CPU。

```cmd
# For Windows:
set AZCOPY_CONCURRENCY_VALUE=<value>
# For Linux:
export AZCOPY_CONCURRENCY_VALUE=<value>
# For MacOS
export AZCOPY_CONCURRENCY_VALUE=<value>
# To check the current value of the variable on all the platforms
.\azcopy env
# If the value is blank then the default value is currently in use
```

### <a name="change-the-location-of-the-log-files"></a>更改日志文件的位置

你可以更改日志文件的位置以满足需要或者避免填满 OS 磁盘。

```cmd
# For Windows:
set AZCOPY_LOG_LOCATION=<value>
# For Linux:
export AZCOPY_LOG_LOCATION=<value>
# For MacOS
export AZCOPY_LOG_LOCATION=<value>
# To check the current value of the variable on all the platforms
.\azcopy env
# If the value is blank, then the default value is currently in use
```
### <a name="change-the-default-log-level"></a>更改默认日志级别 

AzCopy 日志级别默认设置为 INFO。 如果想要降低日志详细程度以节省磁盘空间，请使用 ``--log-level`` 选项覆盖该设置。 可用日志级别为：DEBUG、INFO、WARNING、ERROR、PANIC 和 FATAL。

### <a name="review-the-logs-for-errors"></a>查看日志中的错误

以下命令将从 04dc9ca9-158f-7945-5933-564021086c79 日志中获取状态为 UPLOADFAILED 的所有错误：

```azcopy
cat 04dc9ca9-158f-7945-5933-564021086c79.log | grep -i UPLOADFAILED
```
## <a name="troubleshooting"></a>故障排除

AzCopy 将创建日志文件和每个作业的计划文件。 可以使用日志调查并解决任何潜在问题。 日志包含失败状态（UPLOADFAILED、COPYFAILED 和 DOWNLOADFAILED）、完整路径以及失败原因。 作业日志和计划文件在 Windows 上位于 %USERPROFILE\\.azcopy 文件夹中，在 Mac 和 Linux 上位于 $HOME\\.azcopy 文件夹中。

> [!IMPORTANT]
> 提交到 Microsoft 支持部门 （或故障排除问题涉及任何第三方），请求时共享你想要执行的命令经过修订的版本。 这可以确保不会意外地与任何人共享 SAS。 可以在日志文件的开头找到经修订的版本。

### <a name="view-and-resume-jobs"></a>查看和恢复作业

每个传输操作都将创建一个 AzCopy 作业。 使用以下命令查看作业的历史记录：

```azcopy
.\azcopy jobs list
```

若要查看作业统计信息，请使用以下命令：

```azcopy
.\azcopy jobs show <job-id>
```

若要按状态筛选传输，请使用以下命令：

```azcopy
.\azcopy jobs show <job-id> --with-status=Failed
```

使用以下命令恢复失败/取消的作业。 此命令使用连同 SAS 令牌的标识符，因为它不是永久性出于安全原因：

```azcopy
.\azcopy jobs resume <jobid> --source-sas="<sastokenhere>"
.\azcopy jobs resume <jobid> --destination-sas="<sastokenhere>"
```

## <a name="next-steps"></a>后续步骤

如有任何疑问、问题或一般反馈，请通过 [GitHub](https://github.com/Azure/azure-storage-azcopy) 提交。


