---
title: 快速入门：使用 Azure CLI 管理 Azure 文件共享
description: 通过本快速入门了解如何使用 Azure CLI 来管理 Azure 文件。
services: storage
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 43a5a72ac32d8ed3510cecb505f5e62cf91d7106
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64710820"
---
# <a name="quickstart-create-and-manage-azure-file-shares-using-azure-cli"></a>快速入门：使用 Azure CLI 创建和管理 Azure 文件共享
本指南介绍通过 Azure CLI 来使用 [Azure 文件共享](storage-files-introduction.md)的基本知识。 Azure 文件共享与其他文件共享一样，只不过是存储在云中并由 Azure 平台提供支持。 Azure 文件共享支持行业标准 SMB 协议，可以跨多个计算机、应用程序和实例进行文件共享。 

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果决定在本地安装并使用 Azure CLI，则必须运行 Azure CLI 2.0.4 或更高版本才能执行本文中的步骤。 若要查找 Azure CLI 版本，请运行 **az --version**。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0](/cli/azure/install-azure-cli)。 

默认情况下，Azure CLI 命令返回 JavaScript 对象表示法 (JSON)。 JSON 是通过 REST API 发送和接收消息的标准方式。 为了便于处理 JSON 响应，本文中的某些示例使用基于 Azure CLI 命令的 *query* 参数。 该参数使用 [JMESPath 查询语言](http://jmespath.org/)来分析 JSON。 若要详细了解如何按照 JMESPath 查询语言的规范来使用 Azure CLI 命令的结果，请参阅 [JMESPath tutorial](http://jmespath.org/tutorial.html)（JMESPath 教程）。

## <a name="sign-in-to-azure"></a>登录 Azure
如果在本地使用 Azure CLI，请打开一个提示符窗口并登录到 Azure（如果尚未这样做）。

```bash 
az login
```

## <a name="create-a-resource-group"></a>创建资源组
资源组是在其中部署和管理 Azure 资源的逻辑容器。 如果还没有 Azure 资源组，可以使用 [az group create](/cli/azure/group) 命令创建一个。 

以下示例在“美国东部”位置创建名为“myResourceGroup”的资源组：

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-storage-account"></a>创建存储帐户
存储帐户是一个存储共享池，在其中可以部署 Azure 文件共享或其他存储资源，例如 Blob 或队列。 一个存储帐户可以包含无数个文件共享。 一个共享可以存储无数个文件，直到达到存储帐户的容量限制为止。

以下示例使用 [az storage account create](/cli/azure/storage/account) 命令创建名为 mystorageaccount\<随机数字\> 的存储帐户，然后将该存储帐户的名称置于 `$STORAGEACCT` 变量中。 存储帐户名称必须是唯一的，因此请确保将“mystorageacct”替换为唯一名称。

```azurecli-interactive 
STORAGEACCT=$(az storage account create \
    --resource-group "myResourceGroup" \
    --name "mystorageacct" \
    --location eastus \
    --sku Standard_LRS \
    --query "name" | tr -d '"')
```

### <a name="get-the-storage-account-key"></a>获取存储帐户密钥
存储帐户密钥控制对存储帐户中资源的访问。 这些密钥是在创建存储帐户时自动创建的。 可以使用 [az storage account keys list](/cli/azure/storage/account/keys) 命令获取存储帐户的存储帐户密钥： 

```azurecli-interactive 
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```

## <a name="create-an-azure-file-share"></a>创建 Azure 文件共享
现在可以创建第一个 Azure 文件共享了。 请使用 [az storage share create](/cli/azure/storage/share) 命令创建文件共享。 以下示例创建名为 *myshare* 的 Azure 文件共享： 

```azurecli-interactive
az storage share create \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --name "myshare" 
```

共享名称只能包含小写字母、数字和单个连字符（但不能以连字符开头）。 有关为文件共享和文件命名的完整详细信息，请参阅 [命名和引用共享、目录、文件和元数据](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata)。

## <a name="use-your-azure-file-share"></a>使用 Azure 文件共享
Azure 文件提供两种在 Azure 文件共享中使用文件和文件夹的方法：行业标准[服务器消息块 (SMB) 协议](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx)和[文件 REST 协议](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api)。 

若要通过 SMB 装载文件共享，请参阅下述基于 OS 的文档：
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)
- [Windows](storage-how-to-use-files-windows.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>将 Azure 文件共享与文件 REST 协议配合使用 
可以直接使用文件 REST 协议（手动进行 REST HTTP 调用），但最常见的使用文件 REST 协议的方式是使用 Azure CLI、[Azure PowerShell 模块](storage-how-to-use-files-powershell.md)或 Azure 存储 SDK，所有这些方式都可以在所选脚本/编程语言中为文件 REST 协议提供很好的包装器。  

我们预计，在使用 Azure 文件时，大多数情况下需要通过 SMB 协议来使用 Azure 文件共享，因为这样可以使用那些预期可以使用的现有应用程序和工具，但某些情况下，使用文件 REST API 比使用 SMB 更具优势，例如：

- 需要通过 Azure Bash Cloud Shell（不能通过 SMB 来装载文件共享）来浏览文件共享。
- 需在无法装载 SMB 共享的客户端（例如未将端口 445 解除阻止的本地客户端）中执行脚本或应用程序。
- 需利用无服务器资源，例如 [Azure Functions](../../azure-functions/functions-overview.md)。 

以下示例介绍如何使用 Azure CLI 通过文件 REST 协议来操作 Azure 文件共享。 

### <a name="create-a-directory"></a>创建目录
若要在 Azure 文件共享的根目录中创建名为 *myDirectory* 的新目录，请使用 [`az storage directory create`](/cli/azure/storage/directory) 命令：

```azurecli-interactive
az storage directory create \
   --account-name $STORAGEACCT \
   --account-key $STORAGEKEY \
   --share-name "myshare" \
   --name "myDirectory" 
```

### <a name="upload-a-file"></a>上传文件
若要演示如何使用 [`az storage file upload`](/cli/azure/storage/file) 命令来上传文件，请首先在 Cloud Shell 暂存驱动器上创建要上传的文件。 以下示例创建并上传该文件：

```azurecli-interactive
date > ~/clouddrive/SampleUpload.txt

az storage file upload \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare" \
    --source "~/clouddrive/SampleUpload.txt" \
    --path "myDirectory/SampleUpload.txt"
```

如果在本地运行 Azure CLI，请将 `~/clouddrive` 替换为计算机上的现有路径。

上传文件以后，即可使用 [`az storage file list`](/cli/azure/storage/file) 命令，确保文件已上传到 Azure 文件共享：

```azurecli-interactive
az storage file list \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare" \
    --path "myDirectory" \
    --output table
```

### <a name="download-a-file"></a>下载文件
可以使用 [`az storage file download`](/cli/azure/storage/file) 命令下载已上传到 Cloud Shell 暂存驱动器的文件的副本：

```azurecli-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists, because you've run this example before
rm -rf ~/clouddrive/SampleDownload.txt

az storage file download \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare" \
    --path "myDirectory/SampleUpload.txt" \
    --dest "~/clouddrive/SampleDownload.txt"
```

### <a name="copy-files"></a>复制文件
一项常见的任务是将文件从一个文件共享复制到另一个文件共享，或者将文件在文件共享和 Azure Blob 存储容器之间来回复制。 为了演示此功能，请创建一个新的共享。 请使用 [az storage file copy](/cli/azure/storage/file/copy) 命令将上传的文件复制到该新共享： 

```azurecli-interactive
az storage share create \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --name "myshare2"

az storage directory create \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare2" \
    --name "myDirectory2"

az storage file copy start \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --source-share "myshare" \
    --source-path "myDirectory/SampleUpload.txt" \
    --destination-share "myshare2" \
    --destination-path "myDirectory2/SampleCopy.txt"
```

现在，如果列出新共享中的文件，则应看到已复制的文件：

```azurecli-interactive
az storage file list \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare2" \
    --output table
```

虽然 `az storage file copy start` 命令可以方便地用于 Azure 文件共享和 Azure Blob 存储容器之间的文件移动，但我们仍建议你使用 AzCopy 进行较大型 （就要移动的文件的数量或大小而言）的移动。详细了解 [Linux 版 AzCopy](../common/storage-use-azcopy-linux.md) 和 [Windows 版 AzCopy](../common/storage-use-azcopy.md)。 AzCopy 必须安装在本地。 AzCopy 在 Cloud Shell 中不可用。 

## <a name="create-and-manage-share-snapshots"></a>创建和管理共享快照
可以通过 Azure 文件共享执行的另一项有用的任务是创建共享快照。 快照保存 Azure 文件共享在某个时间点的副本。 共享快照类似于你可能已经熟悉的某些操作系统技术：

- 适用于 Linux 系统的[逻辑卷管理器 (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) 快照。
- 适用于 macOS 的 [Apple 文件系统 (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) 快照
- 适用于 Windows 文件系统（例如 NTFS 和 ReFS）的[卷影复制服务 (VSS)](https://docs.microsoft.com/windows/desktop/VSS/volume-shadow-copy-service-portal) 可以使用 [`az storage share snapshot`](/cli/azure/storage/share) 命令创建共享快照：

```azurecli-interactive
SNAPSHOT=$(az storage share snapshot \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --name "myshare" \
    --query "snapshot" | tr -d '"')
```

### <a name="browse-share-snapshot-contents"></a>浏览共享快照内容
可以浏览共享快照的内容，只需将变量 `$SNAPSHOT` 中捕获的共享快照的时间戳传递给 `az storage file list` 命令即可：

```azurecli-interactive
az storage file list \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare" \
    --snapshot $SNAPSHOT \
    --output table
```

### <a name="list-share-snapshots"></a>列出共享快照
若要查看为共享生成的快照的列表，请使用以下命令：

```azurecli-interactive
az storage share list \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --include-snapshot \
    --query "[? name=='myshare' && snapshot!=null]" | tr -d '"'
```

### <a name="restore-from-a-share-snapshot"></a>从共享快照还原
可以使用此前用过的 `az storage file copy start` 命令还原某个文件。 首先，请删除已上传的 SampleUpload.txt 文件，这样才能将其从快照还原：

```azurecli-interactive
# Delete SampleUpload.txt
az storage file delete \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --share-name "myshare" \
    --path "myDirectory/SampleUpload.txt"
 # Build the source URI for a snapshot restore
URI=$(az storage account show \
    --resource-group "myResourceGroup" \
    --name $STORAGEACCT \
    --query "primaryEndpoints.file" | tr -d '"')
 URI=$URI"myshare/myDirectory/SampleUpload.txt?sharesnapshot="$SNAPSHOT
 # Restore SampleUpload.txt from the share snapshot
az storage file copy start \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --source-uri $URI \
    --destination-share "myshare" \
    --destination-path "myDirectory/SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>删除共享快照
可以使用 [`az storage share delete`](/cli/azure/storage/share) 命令删除共享快照。 所使用的变量包含对 `--snapshot` 参数的 `$SNAPSHOT` 引用：

```azurecli-interactive
az storage share delete \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --name "myshare" \
    --snapshot $SNAPSHOT
```

## <a name="clean-up-resources"></a>清理资源
完成后，可以使用 [`az group delete`](/cli/azure/group) 命令删除资源组和所有相关的资源： 

```azurecli-interactive 
az group delete --name "myResourceGroup"
```

也可逐一删除资源。
- 若要删除为本文创建的 Azure 文件共享，请执行以下命令：

    ```azurecli-interactive
    az storage share delete \
        --account-name $STORAGEACCT \
        --account-key $STORAGEKEY \
        --name "myshare" \
        --delete-snapshots include

    az storage share delete \
        --account-name $STORAGEACCT \
        --account-key $STORAGEKEY \
        --name "myshare2" \
        --delete-snapshots include
    ```

- 若要删除存储帐户本身，请执行以下命令 （这会隐式删除已创建的 Azure 文件共享，以及任何其他可能已创建的存储资源，例如 Azure Blob 存储容器）：

    ```azurecli-interactive
    az storage account delete \
        --resource-group "myResourceGroup" \
        --name $STORAGEACCT \
        --yes
    ```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [什么是 Azure 文件？](storage-files-introduction.md)
