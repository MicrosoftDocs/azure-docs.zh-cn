---
title: 将 Azure CLI 用于 Azure 存储 | Microsoft Docs
description: 了解如何将 Azure 命令行界面 (Azure CLI) 用于 Azure 存储，以便创建和管理存储帐户并处理 Azure blob 和文件。
services: storage
author: tamram
ms.service: storage
ms.devlang: azurecli
ms.topic: article
ms.date: 06/02/2017
ms.author: tamram
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: ea7e4757aac0fccf60a44c70e9de6a63c1ec9498
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147014"
---
# <a name="using-the-azure-cli-with-azure-storage"></a>将 Azure CLI 用于 Azure 存储

开源且跨平台的 Azure CLI 提供了一组可在 Azure 平台上运行的命令。 它提供 [Azure 门户](https://portal.azure.com)所提供的很多相同功能，包括各种数据访问功能。

本指南介绍如何使用 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2) 执行多个使用 Azure 存储帐户中的资源的任务。 在使用本指南之前，我们建议下载并安装或者升级到最新版 CLI。

指南中的示例假定在 Ubuntu 上使用 Bash shell，但其他平台的执行情况应与此类似。 

[!INCLUDE [storage-cli-versions](../../../includes/storage-cli-versions.md)]

## <a name="prerequisites"></a>必备组件
本指南假设读者了解 Azure 存储的基本概念。 本指南还假定，用户能够满足下面为 Azure 和存储服务指定的帐户创建要求。

### <a name="accounts"></a>帐户
* **Azure 帐户**：如果你还没有 Azure 订阅，可以[创建一个免费 Azure 帐户](https://azure.microsoft.com/free/)。
* **存储帐户**：请参阅[关于 Azure 存储帐户](storage-create-storage-account.md)中的[创建存储帐户](storage-quickstart-create-account.md)。

### <a name="install-the-azure-cli"></a>安装 Azure CLI

按照[安装 Azure CLI](/cli/azure/install-az-cli2) 中的概要说明，下载和安装 Azure CLI。

> [!TIP]
> 如果在安装时遇到问题，请查看本文的[安装故障排除](/cli/azure/install-az-cli2)部分以及 GitHub 上的 [Install Troubleshooting](https://github.com/Azure/azure-cli/blob/master/doc/install_troubleshooting.md)（安装故障排除）指南。
>

## <a name="working-with-the-cli"></a>使用 CLI

安装 CLI 之后，可以在命令行界面（Bash、终端、命令提示符）中使用 `az` 命令访问 Azure CLI 命令。 键入 `az` 命令以查看基本命令的完整列表（下面的示例输出已被截断）：

```
     /\
    /  \    _____   _ _ __ ___
   / /\ \  |_  / | | | \'__/ _ \
  / ____ \  / /| |_| | | |  __/
 /_/    \_\/___|\__,_|_|  \___|


Welcome to the cool new Azure CLI!

Here are the base commands:

    account          : Manage subscriptions.
    acr              : Manage Azure container registries.
    acs              : Manage Azure Container Services.
    ad               : Synchronize on-premises directories and manage Azure Active Directory
                       resources.
    ...
```

在命令行接口中，执行命令 `az storage --help` 列出 `storage` 命令子组。 子组说明概述了 Azure CLI 提供的用于使用存储资源的功能。

```
Group
    az storage: Durable, highly available, and massively scalable cloud storage.

Subgroups:
    account  : Manage storage accounts.
    blob     : Object storage for unstructured data.
    container: Manage blob storage containers.
    cors     : Manage Storage service Cross-Origin Resource Sharing (CORS).
    directory: Manage file storage directories.
    entity   : Manage table storage entities.
    file     : File shares that use the standard SMB 3.0 protocol.
    logging  : Manage Storage service logging information.
    message  : Manage queue storage messages.
    metrics  : Manage Storage service metrics.
    queue    : Use queues to effectively scale applications according to traffic.
    share    : Manage file shares.
    table    : NoSQL key-value storage using semi-structured datasets.
```

## <a name="connect-the-cli-to-your-azure-subscription"></a>将 CLI 连接到 Azure 订阅

若要使用 Azure 订阅中的资源，必须首先使用 `az login` 登录到 Azure 帐户。 登录方法有多种：

* **交互式登录**：`az login`
* **使用用户名和密码登录**：`az login -u johndoe@contoso.com -p VerySecret`
  * 这不能用于 Microsoft 帐户或使用多重身份验证的帐户。
* **使用服务主体登录**：`az login --service-principal -u http://azure-cli-2016-08-05-14-31-15 -p VerySecret --tenant contoso.onmicrosoft.com`

## <a name="azure-cli-sample-script"></a>Azure CLI 示例脚本

接下来，我们将使用一个小型 shell 脚本，该脚本会发出一些基本的 Azure CLI 命令与 Azure 存储资源进行交互。 该脚本首先在存储帐户中创建新容器，然后将现有文件（作为 Blob）上传到该容器。 然后，它列出容器中的所有 Blob，最后，将文件下载到指定的本地计算机上的目标。

```bash
#!/bin/bash
# A simple Azure Storage example script

export AZURE_STORAGE_ACCOUNT=<storage_account_name>
export AZURE_STORAGE_KEY=<storage_account_key>

export container_name=<container_name>
export blob_name=<blob_name>
export file_to_upload=<file_to_upload>
export destination_file=<destination_file>

echo "Creating the container..."
az storage container create --name $container_name

echo "Uploading the file..."
az storage blob upload --container-name $container_name --file $file_to_upload --name $blob_name

echo "Listing the blobs..."
az storage blob list --container-name $container_name --output table

echo "Downloading the file..."
az storage blob download --container-name $container_name --name $blob_name --file $destination_file --output table

echo "Done"
```

**配置并运行脚本**

1. 打开最喜欢的文本编辑器，然后将前面的脚本复制并粘贴到编辑器中。

2. 接下来，更新脚本的变量以反映用户的配置设置。 按照明确的说明替换以下值：

   * \<storage_account_name\>：存储帐户的名称。
   * **\<storage_account_key\>**：存储帐户的主访问密钥或辅助访问密钥。
   * **\<container_name\>**：要创建的新容器的名称，例如“azure-cli-sample-container”。
   * **\<blob_name\>**：容器中目标 Blob 的名称。
   * **\<file_to_upload\>**：本地计算机上小文件的路径，例如：“~/images/HelloWorld.png”。
   * \<destination_file\>：目标文件路径，如“~/downloadedImage.png”。

3. 更新了必要的变量后，保存脚本并退出编辑器。 后续步骤假定已将脚本命名为 **my_storage_sample.sh**。

4. 如有必要，将脚本标记为可执行文件：`chmod +x my_storage_sample.sh`

5. 执行脚本。 例如，在 Bash 中：`./my_storage_sample.sh`

应看到类似于以下内容的输出，在脚本中指定的 **\<destination_file\>** 应出现在本地计算机上。

```
Creating the container...
{
  "created": true
}
Uploading the file...
Percent complete: %100.0
Listing the blobs...
Name       Blob Type      Length  Content Type              Last Modified
---------  -----------  --------  ------------------------  -------------------------
README.md  BlockBlob        6700  application/octet-stream  2017-05-12T20:54:59+00:00
Downloading the file...
Name
---------
README.md
Done
```

> [!TIP]
> 前面的输出采用**表**格式。 可以通过在 CLI 命令中指定 `--output` 参数来指定要使用的输出格式，或使用 `az configure` 进行全局设置。
>

## <a name="manage-storage-accounts"></a>管理存储帐户

### <a name="create-a-new-storage-account"></a>新建存储帐户
若要使用 Azure 存储，用户需要一个存储帐户。 可以在将计算机配置为连接到订阅之后，创建新的 Azure 存储帐户。

```azurecli
az storage account create \
    --location <location> \
    --name <account_name> \
    --resource-group <resource_group> \
    --sku <account_sku>
```

* `--location` [必需]：位置。 例如，“West US”。
* `--name` [必需]：存储帐户名称。 名称长度必须为 3 到 24 个字符，并且名称只能包含小写字母数字字符。
* `--resource-group` [必需]：资源组的名称。
* `--sku` [必需]：存储帐户 SKU。 允许的值：
  * `Premium_LRS`
  * `Standard_GRS`
  * `Standard_LRS`
  * `Standard_RAGRS`
  * `Standard_ZRS`

### <a name="set-default-azure-storage-account-environment-variables"></a>设置默认的 Azure 存储帐户环境变量

可以在 Azure 订阅中设置多个存储帐户。 若要选择其中一个帐户用于所有后续存储命令，可以设置这些环境变量：

首先，使用 [az storage account keys list](/cli/azure/storage/account/keys) 命令显示存储帐户密钥：

```azurecli-interactive
az storage account keys list \
    --account-name <account_name> \
    --resource-group <resource_group> \
    --output table
```

现在你有了密钥，可以将其和帐户名定义为环境变量：

```azurecli
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_KEY=<key>
```

设置默认存储帐户的另一种方法是使用连接字符串。 首先，使用 `show-connection-string` 命令获取连接字符串：

```azurecli
az storage account show-connection-string \
    --name <account_name> \
    --resource-group <resource_group>
```

然后复制输出连接字符串，并且设置 `AZURE_STORAGE_CONNECTION_STRING` 环境变量（可能需要将连接字符串括在引号中）：

```azurecli
export AZURE_STORAGE_CONNECTION_STRING="<connection_string>"
```

> [!NOTE]
> 本文下列部分中的所有示例均假定已设置 `AZURE_STORAGE_ACCOUNT` 和 `AZURE_STORAGE_KEY` 环境变量。

## <a name="create-and-manage-blobs"></a>创建并管理 blob
Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。 本部分假设读者熟悉 Azure Blob 存储的概念。 有关详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](../blobs/storage-dotnet-how-to-use-blobs.md)和 [Blob 服务概念](/rest/api/storageservices/blob-service-concepts)。

### <a name="create-a-container"></a>创建容器
Azure 存储中的每个 Blob 都必须在容器中。 可以使用 `az storage container create` 命令创建容器：

```azurecli
az storage container create --name <container_name>
```

可以通过指定可选 `--public-access` 参数，为新容器设置读取访问权限的三个级别之一：

* `off`（默认值）：容器数据是帐户所有者私有的。
* `blob`：Blob 的公共读取访问权限。
* `container`：对整个容器的公共读取和列表访问权限。

有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](../blobs/storage-manage-access-to-resources.md)。

### <a name="upload-a-blob-to-a-container"></a>将 Blob 上传到容器中
Azure Blob 存储支持块 Blob、追加 Blob 和页 Blob。 使用 `blob upload` 命令将 Blob 上传到容器：

```azurecli
az storage blob upload \
    --file <local_file_path> \
    --container-name <container_name> \
    --name <blob_name>
```

如果想直接上载到存储帐户中容器内的文件夹，请将 `--name <blob_name>` 替换为 `--name <folder/blob_name>`。

 默认情况下，`blob upload` 命令将 *.vhd 文件上传到页 Blob 或块 Blob。 若要在上传 Blob 时指定另一种类型，可以使用 `--type` 参数，允许的值为 `append`、 `block` 和 `page`。

 有关不同 Blob 类型的详细信息，请参阅 [Understanding Block Blobs, Append Blobs, and Page Blobs](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs)（了解块 Blob、追加 Blob 和页 Blob）。


### <a name="download-a-blob-from-a-container"></a>从容器下载 Blob
此示例演示如何从容器下载 Blob：

```azurecli
az storage blob download \
    --container-name mycontainer \
    --name myblob.png \
    --file ~/mydownloadedblob.png
```

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 [az storage blob list](/cli/azure/storage/blob) 命令列出容器中的 blob。

```azurecli
az storage blob list \
    --container-name mycontainer \
    --output table
```

### <a name="copy-blobs"></a>复制 blob
可以在存储帐户和区域内或跨存储帐户和区域异步复制 Blob。

以下示例演示如何将一个存储帐户中的 Blob 复制到另一个存储帐户。 我们首先在源存储帐户中创建一个容器，指定对其 blob 的公共读取访问权限。 接下来，我们将文件上传到该容器，最后，将 blob 从该容器复制到目标存储帐户中的一个容器。

```azurecli
# Create container in source account
az storage container create \
    --account-name sourceaccountname \
    --account-key sourceaccountkey \
    --name sourcecontainer \
    --public-access blob

# Upload blob to container in source account
az storage blob upload \
    --account-name sourceaccountname \
    --account-key sourceaccountkey \
    --container-name sourcecontainer \
    --file ~/Pictures/sourcefile.png \
    --name sourcefile.png

# Copy blob from source account to destination account (destcontainer must exist)
az storage blob copy start \
    --account-name destaccountname \
    --account-key destaccountkey \
    --destination-blob destfile.png \
    --destination-container destcontainer \
    --source-uri https://sourceaccountname.blob.core.windows.net/sourcecontainer/sourcefile.png
```

在上面的示例中，目标容器必须已存在于目标存储帐户中，复制操作才会成功。 另外，在 `--source-uri` 参数中指定的源 blob 必须包括一个共享的访问签名 (SAS) 令牌，或者可供公开访问，如此示例中所示。

### <a name="delete-a-blob"></a>删除 Blob
若要删除 Blob，请使用 `blob delete` 命令：

```azurecli
az storage blob delete --container-name <container_name> --name <blob_name>
```

## <a name="create-and-manage-file-shares"></a>创建和管理文件共享
Azure 文件使用服务器消息块 (SMB) 协议为应用程序提供共享存储。 Microsoft Azure 虚拟机和云服务以及本地应用程序可以通过装载的共享来共享文件数据。 可以通过 Azure CLI 管理文件共享和文件数据。 有关 Azure 文件的详细信息，请参阅 [Azure 文件简介](../files/storage-files-introduction.md)。

### <a name="create-a-file-share"></a>创建文件共享
Azure 文件共享是 Azure 中的 SMB 文件共享。 所有目录和文件都必须在文件共享中创建。 一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到存储帐户的容量限制为止。 下面的示例创建名为 **myshare** 的文件共享。

```azurecli
az storage share create --name myshare
```

### <a name="create-a-directory"></a>创建目录
目录提供了 Azure 文件共享中的层次结构。 以下示例在文件共享中创建名为 **myDir** 的目录。

```azurecli
az storage directory create --name myDir --share-name myshare
```

目录路径可以包括多个级别，例如 **dir1/dir2**。 但是，必须在创建子目录之前确保所有父目录都存在。 例如，对于路径 **dir1/dir2**，必须先创建目录 **dir1**，再创建目录 **dir2**。

### <a name="upload-a-local-file-to-a-share"></a>将本地文件上传到共享
以下示例将文件从 **~/temp/samplefile.txt** 上传到 **myshare** 文件共享的根目录。 `--source` 参数指定要上传的现有本地文件。

```azurecli
az storage file upload --share-name myshare --source ~/temp/samplefile.txt
```

与创建目录一样，可以指定共享内的目录路径，将文件上传到共享内的现有目录：

```azurecli
az storage file upload --share-name myshare/myDir --source ~/temp/samplefile.txt
```

共享中的文件最大可以为 1 TB。

### <a name="list-the-files-in-a-share"></a>列出共享中的文件
可以通过使用 `az storage file list` 命令列出共享中的文件和目录：

```azurecli
# List the files in the root of a share
az storage file list --share-name myshare --output table

# List the files in a directory within a share
az storage file list --share-name myshare/myDir --output table

# List the files in a path within a share
az storage file list --share-name myshare --path myDir/mySubDir/MySubDir2 --output table
```

### <a name="copy-files"></a>复制文件      
可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。 例如，要将文件复制到不同共享中的目录，请执行以下操作：        
        
```azurecli
az storage file copy start \
--source-share share1 --source-path dir1/file.txt \
--destination-share share2 --destination-path dir2/file.txt     
```

## <a name="create-share-snapshot"></a>创建共享快照
可以使用 `az storage share snapshot` 命令创建共享快照：

```cli
az storage share snapshot -n <share name>
```

示例输出
```json
{
  "metadata": {},
  "name": "<share name>",
  "properties": {
    "etag": "\"0x8D50B7F9A8D7F30\"",
    "lastModified": "2017-10-04T23:28:22+00:00",
    "quota": null
  },
  "snapshot": "2017-10-04T23:28:35.0000000Z"
}
```

### <a name="list-share-snapshots"></a>列出共享快照

可使用 `az storage share list --include-snapshots` 列出特定共享的共享快照

```cli
az storage share list --include-snapshots
```

**示例输出**
```json
[
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": "2017-10-04T19:44:13.0000000Z"
  },
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": "2017-10-04T19:45:18.0000000Z"
  },
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": null
  }
]
```

### <a name="browse-share-snapshots"></a>浏览共享快照
此外，还可以使用 `az storage file list` 浏览到特定共享快照以查看其内容。 用户必须指定共享名 `--share-name <snare name>` 和时间戳 `--snapshot '2017-10-04T19:45:18.0000000Z'`

```azurecli-interactive
az storage file list --share-name sharesnapshotdefs --snapshot '2017-10-04T19:45:18.0000000Z' -otable
```

**示例输出**
```
Name            Content Length    Type    Last Modified
--------------  ----------------  ------  ---------------
HelloWorldDir/                    dir
IMG_0966.JPG    533568            file
IMG_1105.JPG    717711            file
IMG_1341.JPG    608459            file
IMG_1405.JPG    652156            file
IMG_1611.JPG    442671            file
IMG_1634.JPG    1495999           file
IMG_1635.JPG    974058            file

```
### <a name="restore-from-share-snapshots"></a>从共享快照还原

可以通过使用 `az storage file download` 命令从共享快照复制或下载文件来还原文件

```azurecli-interactive
az storage file download --path IMG_0966.JPG --share-name sharesnapshotdefs --snapshot '2017-10-04T19:45:18.0000000Z'
```
**示例输出**
```
{
  "content": null,
  "metadata": {},
  "name": "IMG_0966.JPG",
  "properties": {
    "contentLength": 533568,
    "contentRange": "bytes 0-533567/533568",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentType": "application/octet-stream"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D50B5F49F7ACDF\"",
    "lastModified": "2017-10-04T19:37:03+00:00",
    "serverEncrypted": true
  }
}
```
## <a name="delete-share-snapshot"></a>删除共享快照
可以通过提供 `--snapshot` 参数和共享快照时间戳使用 `az storage share delete` 命令来删除共享快照：

```cli
az storage share delete -n <share name> --snapshot '2017-10-04T23:28:35.0000000Z' 
```

示例输出
```json
{
  "deleted": true
}
```

## <a name="next-steps"></a>后续步骤
下面是一些用于详细了解如何使用 Azure CLI 的附加资源。 

* [Azure CLI 入门](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)
* [Azure CLI 命令参考](/cli/azure)
* [GitHub 上的 Azure CLI](https://github.com/Azure/azure-cli)
