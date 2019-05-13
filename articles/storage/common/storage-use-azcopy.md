---
title: 使用 AzCopy on Windows 将数据复制或移动到 Azure 存储 | Microsoft Docs
description: 使用 AzCopy on Windows 实用程序将数据移动或复制到 blob、表和文件内容或从 blob、表和文件内容移动或复制数据。 从本地文件将数据复制到 Azure 存储，或者在存储帐户中或存储帐户之间复制数据。 轻松地将数据迁移到 Azure 存储。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 01/03/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: ead5729496253b553ea453a9d6ce33b6b673e027
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149043"
---
# <a name="transfer-data-with-the-azcopy-on-windows"></a>使用 AzCopy on Windows 传输数据

AzCopy 是一个命令行实用程序，专用于使用旨在实现最佳性能的简单命令将数据复制到 Microsoft Azure Blob、文件和表存储以及从这些位置复制数据。 可在文件系统和存储帐户之间或在存储帐户之间复制数据。  

> [!IMPORTANT]
> 本文介绍了较旧版本的 AzCopy。
>若要安装最新版本的 AzCopy，请参阅[AzCopy v10](storage-use-azcopy-v10.md)。

如果您选择安装较旧版本的 AzCopy (AzCopy v8.1)，则可以下载的多个版本。 AzCopy on Windows 提供 Windows 样式的命令行选项。 [AzCopy on Linux](storage-use-azcopy-linux.md) 面向 Linux 平台，它提供 POSIX 样式的命令行选项。 本文涉及到 AzCopy on Windows。

## <a name="download-and-install-azcopy-v81-on-windows"></a>下载并安装在 Windows 上的 AzCopy (v8.1)

下载[在 Windows 上的 AzCopy (v8.1)](https://aka.ms/downloadazcopy)。

#### <a name="azcopy-on-windows-81-release-notes"></a>AzCopy on Windows 8.1 发行说明

- 最新版本不再支持表服务。 如果使用表导出功能，请下载 AzCopy 7.3 版本。
- 使用 .NET Core 2.1 构建，现在所有 .NET Core 依赖项都打包在安装中。
- 添加了 OAuth 身份验证支持。 使用 ```azcopy login``` 通过 Azure Active Directory 登录。

### <a name="azcopy-with-table-support-v73"></a>带表支持的 Azcopy (v7.3)

下载[带表支持的 AzCopy 7.3](https://aka.ms/downloadazcopynet)。

### <a name="post-installation-step"></a>安装后步骤

使用安装程序安装 AzCopy on Windows 后，打开一个命令窗口，然后导航到计算机上的 AzCopy 安装目录，`AzCopy.exe` 可执行文件位于该目录中。 如果需要，可以将 AzCopy 安装位置添加到系统路径。 默认情况下，AzCopy 安装到 `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy` 或 `%ProgramFiles%\Microsoft SDKs\Azure\AzCopy`。

## <a name="writing-your-first-azcopy-command"></a>编写第一条 AzCopy 命令

AzCopy 命令的基本语法是：

```azcopy
AzCopy /Source:<source> /Dest:<destination> [Options]
```

以下示例演示了将数据复制到 Microsoft Azure Blob、文件和表以及从这些位置复制数据的各种情况。 请参阅 [AzCopy 参数](#azcopy-parameters)部分，了解每个示例中使用的参数的详细说明。

## <a name="download-blobs-from-blob-storage"></a>从 Blob 存储下载 Blob

让我们了解使用 AzCopy 下载 Blob 的多种方式。

### <a name="download-a-single-blob"></a>下载单个 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"
```

请注意，如果文件夹 `C:\myfolder` 不存在，AzCopy 会创建该文件夹并将 `abc.txt` 下载到新文件夹中。

### <a name="download-a-single-blob-from-the-secondary-region"></a>从次要区域下载单个 Blob

```azcopy
AzCopy /Source:https://myaccount-secondary.blob.core.windows.net/mynewcontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"
```

请注意，必须启用读取访问权限异地冗余存储才能访问次要区域。

### <a name="download-all-blobs-in-a-container"></a>下载容器中的所有 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /S
```

假定指定的容器中存在以下 blob：  

    abc.txt
    abc1.txt
    abc2.txt
    vd1\a.txt
    vd1\abcd.txt

在下载操作完成后，目录 `C:\myfolder` 中将包含以下文件：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\vd1\a.txt
    C:\myfolder\vd1\abcd.txt

如果未指定选项 `/S`，则不会下载任何 Blob。

### <a name="download-blobs-with-a-specific-prefix"></a>下载具有特定前缀的 Blob

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:a /S
```

假定指定的容器中存在以下 blob。 将下载所有以前缀 `a` 开头的 Blob：

    abc.txt
    abc1.txt
    abc2.txt
    xyz.txt
    vd1\a.txt
    vd1\abcd.txt

下载操作完成后，文件夹 `C:\myfolder` 中将包含以下文件：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt

前缀适用于虚拟目录，后者构成了 blob 名称的第一个部分。 在上面所示的示例中，虚拟目录与指定的前缀不匹配，因此不会下载它。 此外，如果未指定选项 `/S`，则 AzCopy 不会下载任何 Blob。

### <a name="set-the-last-modified-time-of-exported-files-to-be-same-as-the-source-blobs"></a>将已导出文件的上次修改时间设置为与源 blob 相同

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT
```

还可以根据 Blob 的上次修改时间将其从下载操作中排除。 例如，如果想要排除其上次修改时间与目标文件相同或晚于目标文件的 blob，则添加 `/XN` 选项：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XN
```

如果想要排除其上次修改时间与目标文件相同或早于目标文件的 Blob，请添加 `/XO` 选项：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XO
```

## <a name="upload-blobs-to-blob-storage"></a>将 Blob 上传到 Blob 存储

让我们了解使用 AzCopy 上传 Blob 的多种方式。

### <a name="upload-a-single-blob"></a>上传单个 Blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:"abc.txt"
```

如果指定的目标容器不存在，AzCopy 将创建它并将文件上传到其中。

### <a name="upload-a-single-blob-to-a-virtual-directory"></a>将单个 Blob 上传到虚拟目录

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer/vd /DestKey:key /Pattern:abc.txt
```

如果指定的虚拟目录不存在，AzCopy 将上传文件以在 Blob 名称中包含虚拟目录（例如上例中的 `vd/abc.txt`）。

### <a name="upload-all-blobs-in-a-folder"></a>上传文件夹中的所有 Blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /S
```

指定选项 `/S` 会以递归方式将指定目录的内容上传到 Blob 存储，这意味着也会上传所有的子文件夹及其文件。 例如，假定以下文件存在于文件夹 `C:\myfolder` 中：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\subfolder\a.txt
    C:\myfolder\subfolder\abcd.txt

上传操作完成后，容器中将包含以下文件：

    abc.txt
    abc1.txt
    abc2.txt
    subfolder\a.txt
    subfolder\abcd.txt

如果未指定选项 `/S`，AzCopy 不会以递归方式上传。 上传操作完成后，容器中将包含以下文件：

    abc.txt
    abc1.txt
    abc2.txt

### <a name="upload-blobs-matching-a-specific-pattern"></a>上传与特定模式匹配的 Blob

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:a* /S
```

假定以下文件存在于文件夹 `C:\myfolder` 中：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt
    C:\myfolder\xyz.txt
    C:\myfolder\subfolder\a.txt
    C:\myfolder\subfolder\abcd.txt

上传操作完成后，容器中将包含以下文件：

    abc.txt
    abc1.txt
    abc2.txt
    subfolder\a.txt
    subfolder\abcd.txt

如果未指定选项 `/S`，AzCopy 仅上传不会在虚拟目录中驻留的 blob：

    C:\myfolder\abc.txt
    C:\myfolder\abc1.txt
    C:\myfolder\abc2.txt

### <a name="specify-the-mime-content-type-of-a-destination-blob"></a>指定目标 blob 的 MIME 内容类型

默认情况下，AzCopy 将目标 blob 的内容类型设置为 `application/octet-stream`。 从 3.1.0 版开始，可以通过选项 `/SetContentType:[content-type]` 显示指定内容类型。 此语法会在上传操作中设置所有 blob 的内容类型。

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType:video/mp4
```

如果指定不带任何值的 `/SetContentType`，AzCopy 会根据文件扩展名设置每个 Blob 或文件的内容类型。

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType
```

## <a name="copy-blobs-in-blob-storage"></a>复制 Blob 存储中的 Blob

让我们了解使用 AzCopy 将 Blob 从一个位置复制到另一个位置的多种方法。

### <a name="copy-a-single-blob-from-one-container-to-another-within-the-same-storage-account"></a>将单个 Blob 从一个容器复制到同一存储帐户中的另一个容器 

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt
```

在存储帐户内复制某个 blob 时，将执行[服务器端复制](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### <a name="copy-a-single-blob-from-one-storage-account-to-another"></a>将单个 Blob 从一个存储帐户复制到另一个存储帐户

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
```

在跨存储帐户复制某个 blob 时，将执行[服务器端复制](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### <a name="copy-a-single-blob-from-the-secondary-region-to-the-primary-region"></a>将单个 Blob 从次要区域复制到主要区域

```azcopy
AzCopy /Source:https://myaccount1-secondary.blob.core.windows.net/mynewcontainer1 /Dest:https://myaccount2.blob.core.windows.net/mynewcontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
```

请注意，必须启用读取访问权限异地冗余存储才能访问辅助存储。

### <a name="copy-a-single-blob-and-its-snapshots-from-one-storage-account-to-another"></a>将单个 Blob 及其快照从一个存储帐户复制到另一个存储帐户

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot
```

复制操作完成后，目标容器中将包含 Blob 及其快照。 假定上面的示例中的 blob 具有两个快照，则容器将包括以下 blob 和快照：

    abc.txt
    abc (2013-02-25 080757).txt
    abc (2014-02-21 150331).txt

### <a name="copy-all-blobs-in-a-container-to-another-storage-account"></a>将容器中的所有 Blob 复制到另一个存储帐户 

```azcopy
AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 
/Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /S
```

指定选项 /S 会以递归方式上传指定容器的内容。 有关详细信息和示例，请参阅[上传文件夹中的所有 Blob](#upload-all-blobs-in-a-folder)。

### <a name="synchronously-copy-blobs-from-one-storage-account-to-another"></a>将 Blob 从一个存储帐户同步复制到另一个存储帐户

默认情况下，AzCopy 以异步方式复制两个存储终结点之间的数据。 因此，复制操作使用空闲的带宽容量在后台运行，没有规定 blob 复制速率的 SLA，AzCopy 会定期检查复制状态直到复制完成或失败。

`/SyncCopy` 选项确保复制操作的速度一致。 AzCopy 通过下载 blob，将 blob 从指定的源复制到本地内存，然后将上传到 Blob 存储目标，以实现同步复制。

```azcopy
AzCopy /Source:https://myaccount1.blob.core.windows.net/myContainer/ /Dest:https://myaccount2.blob.core.windows.net/myContainer/ /SourceKey:key1 /DestKey:key2 /Pattern:ab /SyncCopy
```

与异步复制相比，`/SyncCopy` 可能会产生额外的对外费用，建议在与源存储帐户所在的同一区域的 Azure VM 中使用该选项，以避免对外费用。

## <a name="download-files-from-file-storage"></a>从文件存储下载文件

让我们了解使用 AzCopy 下载文件的多种方式。

### <a name="download-a-single-file"></a>下载单个文件

```azcopy
AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt
```

如果指定的源是 Azure 文件共享，则必须指定确切的文件名（*例如* `abc.txt`）以下载单个文件，或者指定选项 `/S` 以递归方式下载该共享中的所有文件。 尝试同时指定文件模式和选项 `/S` 会导致错误。

### <a name="download-all-files-in-a-directory"></a>下载目录中的所有文件

```azcopy
AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S
```

请注意，不会下载空文件夹。

## <a name="upload-files-to-an-azure-file-share"></a>将文件上传到 Azure 文件共享

让我们了解使用 AzCopy 上传文件的多种方式。

### <a name="upload-a-single-file"></a>上传单个文件

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:abc.txt
```

### <a name="upload-all-files-in-a-folder"></a>上传文件夹中的所有文件

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /S
```

请注意，不会上传空文件夹。

### <a name="upload-files-matching-a-specific-pattern"></a>上传与特定模式匹配的文件

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:ab* /S
```

## <a name="copy-files-in-file-storage"></a>复制文件存储中的文件

让我们了解使用 AzCopy 复制 Azure 文件共享中的文件的多种方式。

### <a name="copy-from-one-file-share-to-another"></a>从一个文件共享复制到另一个文件共享

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S
```
在跨文件共享复制某个文件时，将执行[服务器端复制](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### <a name="copy-from-an-azure-file-share-to-blob-storage"></a>从 Azure 文件共享复制到 Blob 存储

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare/ /Dest:https://myaccount2.blob.core.windows.net/mycontainer/ /SourceKey:key1 /DestKey:key2 /S
```
将文件从文件共享复制到 Blob 时，将执行[服务器端复制](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### <a name="copy-a-blob-from-blob-storage-to-an-azure-file-share"></a>将 Blob 从 Blob 存储复制到 Azure 文件共享

```azcopy
AzCopy /Source:https://myaccount1.blob.core.windows.net/mycontainer/ /Dest:https://myaccount2.file.core.windows.net/myfileshare/ /SourceKey:key1 /DestKey:key2 /S
```
将文件从 Blob 复制到文件共享时，会执行[服务器端复制](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### <a name="synchronously-copy-files"></a>同步复制文件

可以指定选项 `/SyncCopy`，以从文件存储到文件存储、从文件存储到 Blob 存储以及从 Blob 存储到文件存储同步复制数据，AzCopy 通过将源数据下载到本地内存并再将其上传到目标以实现此同步操作。 将应用标准传出费用。

```azcopy
AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy
```

从文件存储复制到 Blob 存储时，默认的 Blob 类型是块 Blob；用户可以指定选项 `/BlobType:page` 以更改目标 Blob 类型。

请注意，`/SyncCopy` 可能产生额外的数据传出费用，而异步复制则不会。 在与源存储帐户所在的同一区域的 Azure VM 中，建议使用此选项，避免产生数据传出费用。

## <a name="export-data-from-table-storage"></a>从表存储导出数据

让我们了解如何使用 AzCopy 从 Azure 表存储导出数据。

### <a name="export-a-table"></a>导出表

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key
```

AzCopy 将一个清单文件写入到指定的目标文件夹。 在导入过程中使用该清单文件查找必要的数据文件并执行数据验证。 默认情况下，清单文件使用以下命名约定：

    <account name>_<table name>_<timestamp>.manifest

用户还可以指定选项 `/Manifest:<manifest file name>` 以设置清单文件名。

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /Manifest:abc.manifest
```

### <a name="split-an-export-from-table-storage-into-multiple-files"></a>将表存储的导出拆分为多个文件

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/mytable/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100
```

AzCopy 在已拆分的数据文件名称中使用*卷索引*来区分多个文件。 卷索引由两个部分组成：*分区键范围索引*和*拆分文件索引*。 两个索引都是从零开始。

如果用户未指定选项 `/PKRS`，则分区键范围索引是 0。

例如，假设 AzCopy 在用户指定选项 `/SplitSize` 后生成了两个数据文件。 生成的数据文件名称可能是：

    myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
    myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

请注意，选项 `/SplitSize` 的最小可能值为 32 MB。 如果指定的目标是 Blob 存储，无论用户是否指定了选项 `/SplitSize`，AzCopy 会在数据文件的大小达到 blob 的大小限制 (200GB) 时拆分数据文件。

### <a name="export-a-table-to-json-or-csv-data-file-format"></a>将表导出为 JSON 或 CSV 数据文件格式

默认情况下，AzCopy 会将表导出为 JSON 数据文件。 可以指定选项 `/PayloadFormat:JSON|CSV` 以将表导出为 JSON 或 CSV。

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PayloadFormat:CSV
```

指定 CSV 有效负载格式时，AzCopy 还会为每个数据文件生成文件扩展名为 `.schema.csv` 的架构文件。

### <a name="export-table-entities-concurrently"></a>并发导出表实体

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PKRS:"aa#bb"
```

当用户指定了选项 `/PKRS` 时，AzCopy 会启动并发操作来导出实体。 每个操作导出一个分区键范围。

请注意，并发操作的数量还受选项 `/NC` 的控制。 当复制表实体时，AzCopy 使用核心处理器的数量作为 `/NC` 的默认值，即使未指定 `/NC` 也是如此。 当用户指定了选项 `/PKRS` 时，AzCopy 将使用以下两个值中的较小者（分区键范围和隐式或显式指定的并发操作数量）来确定要启动的并发操作的数量。 有关详细信息，请在命令行中键入 `AzCopy /?:NC`。

### <a name="export-a-table-to-blob-storage"></a>将表导出到 Blob 存储

```azcopy
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:https://myaccount.blob.core.windows.net/mycontainer/ /SourceKey:key1 /Destkey:key2
```

AzCopy 使用以下命名约定在 blob 容器中生成一个 JSON 数据文件：

    <account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

生成的 JSON 数据文件遵循最小元数据的负载格式。 有关此负载格式的详细信息，请参阅 [Payload Format for Table Service Operations](https://msdn.microsoft.com/library/azure/dn535600.aspx)（表服务操作的有效负载格式）。

请注意，将表导出到 blob 中时，AzCopy 会将表实体下载到本地临时数据文件，再将这些实体上传到 blob。 这些临时数据文件会放入默认路径为“<code>%LocalAppData%\Microsoft\Azure\AzCopy</code>”的日志文件文件夹中，可指定选项 /Z:[journal-file-folder] 以更改日志文件文件夹的位置，从而更改临时数据文件的位置。 临时数据文件的大小由表实体的大小和使用选项 /SplitSize 指定的大小所决定，尽管本地磁盘中的临时数据文件在上传到 blob 后会被立即删除，但请确保在删除之前，拥有足够的本地磁盘空间来存储这些临时数据文件。

## <a name="import-data-into-table-storage"></a>将数据导入表存储

让我们了解如何使用 AzCopy 将数据导入 Azure 表存储。

### <a name="import-a-table"></a>导入表

```azcopy
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.core.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace
```

选项 `/EntityOperation` 指示如何将实体插入到表中。 可能的值包括：

* `InsertOrSkip`：跳过现有实体，或者插入新实体（如果它不存在于表中）。
* `InsertOrMerge`：合并现有实体，或者插入新实体（如果它不存在于表中）。
* `InsertOrReplace`：替换现有实体，或者插入新实体（如果它不存在于表中）。

请注意，在导入方案中不能指定选项 `/PKRS`。 与导出方案不同（在导出方案中必须指定 `/PKRS` 选项才会启动并发操作），在导入表时，AzCopy 会默认启动并发操作。 启动的并发操作的默认数量与核心处理器的数量相等；但是，可以通过选项 `/NC` 指定一个不同的并发数量。 有关详细信息，请在命令行中键入 `AzCopy /?:NC`。

请注意，AzCopy 仅支持导入 JSON 文件，不支持导入 CSV 文件。 AzCopy 不支持来自用户创建的 JSON 和清单文件的表导入。 这两类文件必须来自 AzCopy 表导出。 若要避免错误，请不要修改导出的 JSON 或清单文件。

### <a name="import-entities-into-a-table-from-blob-storage"></a>将实体从 Blob 存储导入表中

假定 Blob 容器包含如下内容：表示 Azure 表的 JSON 文件及其随附的清单文件。

    myaccount_mytable_20140103T112020.manifest
    myaccount_mytable_20140103T112020_0_0_0AF395F1DC42E952.json

可以使用 blob 容器中的清单文件运行以下命令将实体导入表中：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:https://myaccount.table.core.windows.net/mytable /SourceKey:key1 /DestKey:key2 /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:"InsertOrReplace"
```

## <a name="other-azcopy-features"></a>AzCopy 的其他功能

让我们了解 AzCopy 的其他一些功能。

### <a name="only-copy-data-that-doesnt-exist-in-the-destination"></a>仅复制目标中不存在的数据

`/XO` 和 `/XN` 参数分别用于阻止复制较早或较新的源资源。 如果只想复制目标中不存在的源资源，可以在 AzCopy 命令中指定这两个参数：

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /XO /XN

    /Source:C:\myfolder /Dest:http://myaccount.file.core.windows.net/myfileshare /DestKey:<destkey> /S /XO /XN

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:http://myaccount.blob.core.windows.net/mycontainer1 /SourceKey:<sourcekey> /DestKey:<destkey> /S /XO /XN

注意，当源或目标为表时，则不支持此操作。

### <a name="use-a-response-file-to-specify-command-line-parameters"></a>使用响应文件指定命令行参数

```azcopy
AzCopy /@:"C:\responsefiles\copyoperation.txt"
```

可以在响应文件中包括任何 AzCopy 命令行参数。 AzCopy 会像处理在命令行上指定的参数一样处理该文件中的参数，使用该文件的内容执行直接替换。

假定有一个名为 `copyoperation.txt` 的响应文件，其中包含以下行： 可以在同一行或多行中指定

    /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

每个 AzCopy 参数：

    /Source:http://myaccount.blob.core.windows.net/mycontainer
    /Dest:C:\myfolder
    /SourceKey:<sourcekey>
    /S
    /Y

如果将参数拆分到两行（如此处所示的 `/sourcekey` 参数），AzCopy 会失败：

    http://myaccount.blob.core.windows.net/mycontainer
     C:\myfolder
    /sourcekey:
    <sourcekey>
    /S
    /Y

### <a name="use-multiple-response-files-to-specify-command-line-parameters"></a>使用多个响应文件指定命令行参数

假定有一个名为 `source.txt` 的响应文件，该文件指定了一个源容器：

    /Source:http://myaccount.blob.core.windows.net/mycontainer

有一个名为 `dest.txt` 的响应文件，该文件在文件系统中指定了一个目标文件夹：

    /Dest:C:\myfolder

并且有一个名为 `options.txt` 的响应文件，该文件指定了 AzCopy 的选项：

    /S /Y

要使用这些响应文件（都位于目录 `C:\responsefiles` 中）调用 AzCopy，请使用以下命令：

```azcopy
AzCopy /@:"C:\responsefiles\source.txt" /@:"C:\responsefiles\dest.txt" /SourceKey:<sourcekey> /@:"C:\responsefiles\options.txt"   
```

AzCopy 会像在命令行上包括了所有单个参数一样来处理此命令：

```azcopy
AzCopy /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y
```

### <a name="specify-a-shared-access-signature-sas"></a>指定共享访问签名 (SAS)

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceSAS:SAS1 /DestSAS:SAS2 /Pattern:abc.txt
```

此外，还可以在容器 URI 上指定一个 SAS：

```azcopy
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1/?SourceSASToken /Dest:C:\myfolder /S
```

### <a name="journal-file-folder"></a>日志文件文件夹

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者通过此选项指定的文件夹中是否存在日志文件。 如果这两个位置中都不存在日志文件，AzCopy 则会将操作视为新操作并生成一个新的日志文件。

如果存在日志文件，AzCopy 会检查输入的命令行是否与该日志文件中的命令行相匹配。 如果两个命令行相匹配，AzCopy 则将恢复未完成的操作。 如果它们不匹配，系统会提示用户是选择覆盖该日志文件以启动新操作，还是取消当前操作。

如果想要为日志文件使用默认位置：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z
```

如果省略了选项 `/Z`，或者指定了选项 `/Z` 但未指定文件夹路径（如上所示），AzCopy 则会在默认位置中创建日志文件，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。 如果日志文件已存在，AzCopy 则将继续根据日志文件恢复操作。

如果想要为日志文件指定自定义位置：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z:C:\journalfolder\
```

如果日志文件尚不存在，此示例将创建日志文件。 如果它已存在，AzCopy 则会根据该日志文件恢复操作。

如果想要恢复 AzCopy 操作：

```azcopy
AzCopy /Z:C:\journalfolder\
```

此示例将恢复可能没有完成的上一操作。

### <a name="generate-a-log-file"></a>生成日志文件

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V
```

如果指定了选项 `/V` 但未提供详细日志的文件路径，AzCopy 则将在默认位置中创建日志文件，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。

或者，可以在自定义位置中创建日志文件：

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V:C:\myfolder\azcopy1.log
```

请注意，如果在选项 `/V` 后指定了相对路径，例如 `/V:test/azcopy1.log`，则会在当前工作目录中名为 `test` 的子文件夹内创建详细日志。

### <a name="specify-the-number-of-concurrent-operations-to-start"></a>指定要启动的并发操作的数量

选项 `/NC` 指定并发复制操作的数量。 默认情况下，AzCopy 会启动一定数量的并发操作以提高数据传输吞吐量。 对于表操作，并发操作的数量与所拥有的处理器数相等。 对于 Blob 和文件操作，并发操作数等于所拥有的处理器数的 8 倍。 如果正在低带宽网络中运行 AzCopy，则可为 /NC 指定较低的数量以避免由于资源争用所导致的故障。

### <a name="run-azcopy-against-the-azure-storage-emulator"></a>针对 Azure 存储模拟器运行 AzCopy

可针对 Blob 的 [Azure 存储模拟器](storage-use-emulator.md)运行 AzCopy：

```azcopy
AzCopy /Source:https://127.0.0.1:10000/myaccount/mycontainer/ /Dest:C:\myfolder /SourceKey:key /SourceType:Blob /S
```

还可以针对表运行 AzCopy：

```azcopy
AzCopy /Source:https://127.0.0.1:10002/myaccount/mytable/ /Dest:C:\myfolder /SourceKey:key /SourceType:Table
```

### <a name="automatically-determine-content-type-of-a-blob"></a>自动确定 Blob 的内容类型

AzCopy 根据将内容类型存储到文件扩展名映射的 JSON 文件确定 blob 的内容类型。 此 JSON 文件命名为 AzCopyConfig.json，并且位于 AzCopy 目录中。 如果你的文件类型不在列表中，可以将映射追加到 JSON 文件中：

```
{
  "MIMETypeMapping": {
    ".myext": "text/mycustomtype",
    .
    .
  }
}
```     

## <a name="azcopy-parameters"></a>AzCopy 参数

以下描述了 AzCopy 的参数。 还可以从命令行键入下列命令之一以获取如何使用 AzCopy 的帮助信息：

* 若要获取 AzCopy 的详细命令行帮助信息，请键入：`AzCopy /?`
* 若要获取任何 AzCopy 参数的详细帮助信息，请键入：`AzCopy /?:SourceKey`
* 若要获取命令行示例，请键入：`AzCopy /?:Sample`

### <a name="sourcesource"></a>/Source:"source"

指定要从中复制数据的源。 源可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。

**适用于：** Blob、文件、表

### <a name="destdestination"></a>/Dest:"destination"

指定要复制到的目标。 目标可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。

**适用于：** Blob、文件、表

### <a name="patternfile-pattern"></a>/Pattern:"file-pattern"

指定文件模式，它指示要复制哪些文件。 /Pattern 参数的行为是由源数据的位置以及是否存在递归模式选项决定的。 递归模式是通过选项 /S 指定的。

如果指定的源是文件系统中的一个目录，则标准通配符将生效，并且会将该目录中的文件与提供的文件模式进行匹配。 如果指定了选项 /S，则 AzCopy 还会将该目录下的任何子文件夹中的所有文件与指定模式进行匹配。

如果指定的源是一个 blob 容器或虚拟目录，则不会应用通配符。 如果指定了选项 /S，则 AzCopy 会将指定的文件模式解释为 blob 前缀。 如果未指定选项 /S，则 AzCopy 会将确切的 blob 名称与文件模式进行匹配。

如果指定的源是 Azure 文件共享，则必须指定确切的文件名（如 abc.txt）以复制单个文件，或指定选项 /S 以递归方式复制该共享中的所有文件。 尝试同时指定文件模式和选项 /S 会导致错误。

当 /Source 是 blob 容器或 blob 虚拟目录时，AzCopy 使用区分大小写匹配，而在所有其他情况下则使用不区分大小写匹配。

未指定文件模式时使用的默认文件模式为 *。* 对于 Azure 存储位置，请使用空前缀。 不支持指定多个文件模式。

**适用于：** Blob、文件

### <a name="destkeystorage-key"></a>/DestKey:"storage-key"

指定目标资源的存储帐户密钥。

**适用于：** Blob、文件、表

### <a name="destsassas-token"></a>/DestSAS:"sas-token"

指定对目标具有读写权限的共享访问签名 (SAS)（如果适用）。 请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。

如果目标资源是 blob 容器、文件共享或表，则可以指定此选项，后跟 SAS 令牌，或者可以将 SAS 指定为目标 blob 容器、文件共享或表 URI 的一部分，而不使用此选项。

如果源和目标都是 blob，则目标 blob 必须与源 blob 位于同一个存储帐户中。

**适用于：** Blob、文件、表

### <a name="sourcekeystorage-key"></a>/SourceKey:"storage-key"

指定源资源的存储帐户密钥。

**适用于：** Blob、文件、表

### <a name="sourcesassas-token"></a>/SourceSAS:"sas-token"

指定对源具有读取和列出权限的共享访问签名（如果适用）。 请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。

如果源资源是 blob 容器，并且既未提供密钥又未提供 SAS，则可以通过匿名访问读取 blob 容器。

如果源是文件共享或表，必须提供一个键或某一 SAS。

**适用于：** Blob、文件、表

### <a name="s"></a>/S

指定复制操作的递归模式。 在递归模式下，AzCopy 会复制与指定的文件模式相匹配的所有 blob 或文件，包括子文件夹中的对象。

**适用于：** Blob、文件

### <a name="blobtypeblock--page--append"></a>/BlobType:"block" | "page" | "append"

指定目标 Blob 是块 Blob、页 Blob 还是追加 Blob。 仅在要上传 Blob 时，此选项才适用。 否则会发生错误。 如果目标是一个 Blob 并且未指定此选项，则默认情况下 AzCopy 将创建块 Blob。

**适用于：** Blob

### <a name="checkmd5"></a>/CheckMD5

计算已下载的数据的 MD5 哈希，并验证存储在 blob 或文件的 Content-MD5 属性中的 MD5 哈希是否与计算得到的哈希匹配。 如果值不匹配，AzCopy 将无法下载数据。 默认情况下，MD5 检查处于关闭状态，因此，必须指定此选项以在下载数据时执行 MD5 检查。

请注意，Azure 存储不保证为 blob 或文件所存储的 MD5 哈希是最新的。 每当 blob 或文件被修改时，客户端都需要负责对 MD5 进行更新。 对于磁盘映像（托管或非托管磁盘），Azure VM 不会随磁盘内容的更改更新 MD5 值，因此，在下载磁盘映像时，/CheckMD5 将引发错误。

在将 Azure blob 或文件上传到服务后，AzCopy v8 始终会为其设置 Content-MD5 属性。  

**适用于：** Blob、文件

### <a name="snapshot"></a>/Snapshot

指示是否传输快照。 只有当源是 blob 时，此选项才有效。

传输的 Blob 快照会按以下格式重命名：blob-name (snapshot-time).extension

默认情况下，不会复制快照。

**适用于：** Blob

### <a name="vverbose-log-file"></a>/V:[verbose-log-file]

将详细的状态消息输出到日志文件中。

默认情况下，详细日志文件在 `%LocalAppData%\Microsoft\Azure\AzCopy` 中会被命名为 AzCopyVerbose.log。 如果为此选项指定了现有的文件位置，详细日志会追加到该文件中。  

**适用于：** Blob、文件、表

### <a name="zjournal-file-folder"></a>/Z:[journal-file-folder]

指定用于恢复某一操作的日志文件文件夹。

AzCopy 始终支持对被中断的操作进行恢复。

如果未指定此选项，或者未指定其文件夹路径，AzCopy 则会在默认位置中创建日志文件，默认位置为 %LocalAppData%\Microsoft\Azure\AzCopy。

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者通过此选项指定的文件夹中是否存在日志文件。 如果这两个位置中都不存在日志文件，AzCopy 则会将操作视为新操作并生成一个新的日志文件。

如果存在日志文件，AzCopy 会检查输入的命令行是否与该日志文件中的命令行相匹配。 如果两个命令行相匹配，AzCopy 则将恢复未完成的操作。 如果它们不匹配，系统会提示用户是选择覆盖该日志文件以启动新操作，还是取消当前操作。

成功完成操作后，将删除该日志文件。

请注意，不支持通过由以前版本的 AzCopy 创建的日志文件来恢复操作。

**适用于：** Blob、文件、表

### <a name="parameter-file"></a>/@:"parameter-file"

指定包含参数的文件。 AzCopy 会像处理在命令行上指定参数一样处理文件中的参数。

在响应文件中，可以在单个行上指定多个参数，也可以将每个参数指定在其单独的行上。 请注意，单个参数不能跨多个行。

响应文件可包括以 # 符号开头的命令行。

可以指定多个响应文件。 但请注意，AzCopy 不支持嵌套的响应文件。

**适用于：** Blob、文件、表

### <a name="y"></a>/Y

取消所有的 AzCopy 确认提示。 未指定 /XO 和 /XN 时，此选项还允许只写 SAS 令牌用于数据上传方案。

**适用于：** Blob、文件、表

### <a name="l"></a>/L

仅指定列出操作；不复制任何数据。

AzCopy 使用此选项解释为在没有此选项 /L 的情况下，模拟运行命令行，并对复制的对象数量进行计数，可以同时指定选项 /V 以检查哪些对象要复制到详细日志中。

此选项的行为也由源数据的位置、是否存在递归模式选项 /S 以及文件模式选项 /Pattern 所决定。

使用此选项时，AzCopy 需要此源位置的列出和读取权限。

**适用于：** Blob、文件

### <a name="mt"></a>/MT

将下载的文件的上次修改时间设置为与源 blob 或文件的上次修改时间相同。

**适用于：** Blob、文件

### <a name="xn"></a>/XN

排除较新的源资源。 如果源的上次修改时间同于或晚于目标，不会复制该资源。

**适用于：** Blob、文件

### <a name="xo"></a>/XO
排除较旧的源资源。 如果源的上次修改时间同于或早于目标，不会复制该资源。

**适用于：** Blob、文件

### <a name="a"></a>/A

仅上传设置了存档属性的文件。

**适用于：** Blob、文件

### <a name="iarashcnetoi"></a>/IA:[RASHCNETOI]

仅上传设置了任何指定属性的文件。

可用的属性包括：

* R 表示只读文件
* A 表示用于存档的文件
* S 表示系统文件
* H 表示隐藏文件
* C 表示压缩文件
* N 表示普通文件
* E 表示加密文件
* T 表示临时文件
* O 表示脱机文件
* I 表示未编制索引的文件

**适用于：** Blob、文件

### <a name="xarashcnetoi"></a>/XA:[RASHCNETOI]

排除设置了任何指定属性的文件。

可用的属性包括：

* R 表示只读文件
* A 表示用于存档的文件
* S 表示系统文件
* H 表示隐藏文件
* C 表示压缩文件
* N 表示普通文件
* E 表示加密文件
* T 表示临时文件
* O 表示脱机文件
* I 表示未编制索引的文件

**适用于：** Blob、文件

### <a name="delimiterdelimiter"></a>/Delimiter:"delimiter"

指示用于分隔 blob 名称中的虚拟目录的分隔符字符。

默认情况下，AzCopy 使用 / 作为分隔符字符。 不过，AzCopy 支持使用任何常见字符（例如 @、# 或 %）作为分隔符。 如果需要在命令行上包括这些特殊字符之一，请将文件名用双引号引起来。

此选项仅适用于下载 blob。

**适用于：** Blob

### <a name="ncnumber-of-concurrent-operations"></a>/NC:"number-of-concurrent-operations"

指定并发操作的数量。

默认情况下，AzCopy 会启动一定数量的并发操作以提高数据传输吞吐量。 请注意，在低带宽环境中，大量的并发操作可能会压垮网络连接，并且会阻碍操作彻底完成。 请根据实际可用的网络带宽限制并发操作。

并发操作的上限为 512。

**适用于：** Blob、文件、表

### <a name="sourcetypeblob--table"></a>/SourceType:"Blob" | "Table"

指定 `source` 资源是本地开发环境中可用的一个 Blob，在存储模拟器中运行。

**适用于：** Blob、表

### <a name="desttypeblob--table"></a>/DestType:"Blob" | "Table"

指定 `destination` 资源是本地开发环境中可用的一个 Blob，在存储模拟器中运行。

**适用于：** Blob、表

### <a name="pkrskey1key2key3"></a>/PKRS:"key1#key2#key3#..."

对分区键范围进行拆分以便并行导出表数据，这可以提高导出操作的速度。

如果未指定此选项，AzCopy 将使用单个线程来导出表实体。 例如，如果用户指定了 /PKRS:"aa#bb"，AzCopy 则将启动三个并发操作。

每个操作将导出三个分区键范围中的一个，如下所示：

  [first-partition-key, aa)

  [aa, bb)

  [bb, last-partition-key]

**适用于：** 表

### <a name="splitsizefile-size"></a>/SplitSize:"file-size"

指定已导出文件的拆分大小（单位为 MB），允许的最小值为 32。

如果未指定此选项，AzCopy 会将表数据导出到单个文件。

如果将表数据导出到一个 blob，并且已导出文件的大小达到了 200 GB 的 blob 大小限制，AzCopy 会拆分导出的文件，即使未指定此选项也是如此。

**适用于：** 表

### <a name="entityoperationinsertorskip--insertormerge--insertorreplace"></a>/EntityOperation:"InsertOrSkip" | "InsertOrMerge" | "InsertOrReplace"

指定表数据导入行为。

* InsertOrSkip - 跳过现有实体，或者插入新实体（如果它不存在于表中）。
* InsertOrMerge - 合并现有实体，或者插入新实体（如果它不存在于表中）。
* InsertOrReplace - 替换现有实体，或者插入新实体（如果它不存在于表中）。

**适用于：** 表

### <a name="manifestmanifest-file"></a>/Manifest:"manifest-file"

指定表导出和导入操作的清单文件。

此选项在导出操作过程中是可选的，如果未指定此选项，AzCopy 会生成具有预定义名称的清单文件。

在导入操作期间用于定位数据文件，此选项是必要的。

**适用于：** 表

### <a name="synccopy"></a>/SyncCopy

指示是否要以同步方式在两个 Azure 存储终结点之间复制 blob 或文件。

AzCopy 默认情况下使用服务器端的异步复制。 指定此选项以执行同步复制，可将 blob 或文件下载到本地内存，然后将其上传到 Azure 存储。

在 Blob 存储内复制文件、文件存储内复制文件或者从 Bolb 存储将文件复制到文件存储时，可使用此选项，反之亦然。

**适用于：** Blob、文件

### <a name="setcontenttypecontent-type"></a>/SetContentType:"content-type"

指定目标 blob 或文件的 MIME 内容类型。

默认情况下，AzCopy 将 blob 或文件的内容类型设置为 application/octet-stream。 通过显式指定此选项的值，可设置所有 blob 或文件的内容类型。

如果指定此选项不带值，AzCopy 会根据 blod 或文件扩展名设置每个 blob 或文件的内容类型。

**适用于：** Blob、文件

### <a name="payloadformatjson--csv"></a>/PayloadFormat:"JSON" | "CSV"

指定表导出数据文件的格式。

如果未指定此选项，则默认情况下，AzCopy 以 JSON 格式导出表数据文件。

**适用于：** 表

## <a name="known-issues-and-best-practices"></a>已知问题和最佳实践

让我们了解一些已知问题和最佳做法。

### <a name="limit-concurrent-writes-while-copying-data"></a>限制复制数据时的并发写入

在使用 AzCopy 复制 blob 或文件时，请记住，在复制数据时其他应用程序可能正在修改该数据。 如果可能，请确保要复制的数据在复制操作期间不会被修改。 例如，当复制与 Azure 虚拟机关联的 VHD 时，请确保当前没有其他应用程序正在向该 VHD 进行写入。 执行此操作的一个好方法是租用要复制的资源。 另外，还可以先创建 VHD 的快照，并复制该快照。

如果在复制 blob 或文件时无法阻止其他应用程序向其进行写入，请记住，在作业完成时，复制的资源可能不再与源资源完全相同。

### <a name="enable-fips-compliant-md5-algorithms-for-azcopy-when-you-use-fips-compliant-algorithms-for-encryption-hashing-and-signing"></a>当进行“使用适用于加密、哈希和签名的 FIPS 兼容算法”时，请启用适用于 AzCopy、与 FIPS 兼容的 MD5 算法。

默认情况下，在复制对象时，如有需要 AzCopy 启动 FIPS 兼容的 MD5 设置的某些安全需求时，AzCopy 则会使用 .NET MD5 实现来计算 MD5。

可以创建属性为 `AzureStorageUseV1MD5` 的 app.config 文件 `AzCopy.exe.config`，并将其与 AzCopy.exe 分开放。

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
      <appSettings>
        <add key="AzureStorageUseV1MD5" value="false"/>
      </appSettings>
    </configuration>

对于属性“AzureStorageUseV1MD5”：

* 如果值为 True（默认值），AzCopy 会使用 .NET MD5 实现。
* 如果值为 False，AzCopy 会使用 FIPS 兼容的 MD5 算法。

Windows 中默认已禁用 FIPS 兼容的算法。 可在计算机上更改此策略设置。 在“运行”窗口（按 Windows 键 + R 键）中键入 secpol.msc 打开“本地安全策略”窗口。 在“安全设置”窗口中，导航到“安全设置” > “本地策略” > “安全选项”。 找到“**系统加密:将 FIPS 兼容算法用于加密、哈希和签名”策略。** 双击该策略，查看“安全设置”列中显示的值。 

## <a name="next-steps"></a>后续步骤

有关 Azure 存储和 AzCopy 的详细信息，请参阅以下资源：

### <a name="azure-storage-documentation"></a>Azure 存储文档：
* [Azure 存储简介](../storage-introduction.md)
* [如何通过 .NET 使用 Blob 存储](../blobs/storage-dotnet-how-to-use-blobs.md)
* [如何通过 .NET 使用文件存储](../storage-dotnet-how-to-use-files.md)
* [如何通过 .NET 使用表存储](../../cosmos-db/table-storage-how-to-use-dotnet.md)
* [如何创建、管理或删除存储帐户](../storage-create-storage-account.md)
* [使用 AzCopy on Linux 传输数据](storage-use-azcopy-linux.md)

### <a name="azure-storage-blog-posts"></a>Azure 存储博客文章：
* [Introducing Azure Storage Data Movement Library Preview](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)（Azure 存储数据移动库预览版简介）
* [AzCopy:Introducing synchronous copy and customized content type](https://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)（AzCopy：同步复制和自定义内容类型简介）
* [AzCopy:Announcing General Availability of AzCopy 3.0 plus preview release of AzCopy 4.0 with Table and File support](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)（AzCopy：宣布公开发行支持表和文件的 AzCopy 3.0 增强预览版本 AzCopy 4.0）
* [AzCopy:Optimized for Large-Scale Copy Scenarios](https://go.microsoft.com/fwlink/?LinkId=507682)（AzCopy：针对大规模复制方案进行优化）
* [AzCopy:Support for read-access geo-redundant storage](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)（AzCopy：支持读取访问异地冗余存储）
* [AzCopy:Transfer data with restartable mode and SAS token](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)（AzCopy：使用可重启的模式和 SAS 令牌传输数据）
* [AzCopy:Using cross-account Copy Blob](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)（AzCopy：使用跨帐户复制 Blob）
* [AzCopy:Uploading/downloading files for Azure Blobs](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)（AzCopy：为 Azure Blob 上传/下载文件）
