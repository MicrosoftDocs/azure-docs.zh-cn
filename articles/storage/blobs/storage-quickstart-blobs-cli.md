---
title: Azure 快速入门 - 使用 Azure CLI 在对象存储中创建 blob | Microsoft Docs
description: 本快速入门将在对象 (Blob) 存储中使用 Azure CLI。 然后，使用该 CLI 将一个 Blob 上传到 Azure 存储，下载一个 Blob，然后列出容器中的 Blob。
services: storage
author: tamram
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 11/14/2018
ms.author: tamram
ms.reviewer: seguler
ms.openlocfilehash: 0b74876d153a775993678ad50ed311c524480f76
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149597"
---
# <a name="quickstart-upload-download-and-list-blobs-using-the-azure-cli"></a>快速入门：使用 Azure CLI 上传、下载和列出 Blob

Azure CLI 是 Azure 的命令行体验，用于管理 Azure 资源。 可以在浏览器中将它与 Azure Cloud Shell 配合使用。 也可将它安装在 macOS、Linux 或 Windows 上，然后从命令行运行它。 本快速入门介绍了如何使用 Azure CLI 通过 Azure Blob 存储来上传和下载数据。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，此快速入门教程要求运行 Azure CLI 2.0.4 版或更高版本。 运行 `az --version` 即可确定你的版本。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli)。

[!INCLUDE [storage-quickstart-tutorial-intro-include-cli](../../../includes/storage-quickstart-tutorial-intro-include-cli.md)]

## <a name="create-a-container"></a>创建容器

始终将 Blob 上传到容器中。 可以整理 Blob 组，就像在计算机的文件夹中整理文件一样。

可以使用 [az storage container create](/cli/azure/storage/container) 命令创建用于存储 blob 的容器。

```azurecli-interactive
az storage container create --name mystoragecontainer
```

## <a name="upload-a-blob"></a>上传 blob

Blob 存储支持块 blob、追加 blob 和页 blob。 存储在 Blob 存储中的大多数文件都存储为块 blob。 必须将数据添加到现有的 blob 中且不能修改该 blob 的现有内容时（例如进行日志记录时），使用追加 blob。 页 blob 支持 IaaS 虚拟机的 VHD 文件。

首先，创建要上传到 Blob 的文件。
如果使用 Azure Cloud Shell，请使用以下方法来创建 `vi helloworld` 文件：当文件打开时，按“插入”，键入“Hello world”，然后按 **Esc** 并输入 `:x`，再按 **Enter**。

此示例使用 [az storage blob upload](/cli/azure/storage/blob) 命令将 Blob 上传到在上一个步骤中创建的容器中。

```azurecli-interactive
az storage blob upload \
    --container-name mystoragecontainer \
    --name blobName \
    --file ~/path/to/local/file
```

如果使用了前述方法在 Azure Cloud Shell 中创建文件，则可改用此 CLI 命令（请注意，不需指定路径，因为此文件是在基目录创建的，而通常是需要指定路径的）：

```azurecli-interactive
az storage blob upload \
    --container-name mystoragecontainer \
    --name helloworld \
    --file helloworld
```

此操作将创建 Blob（如果该 Blob 尚不存在），或者覆盖 Blob（如果该 Blob 已存在）。 上传尽可能多的文件，然后继续操作。

若要同时上传多个文件，则可使用 [az storage blob upload-batch](/cli/azure/storage/blob) 命令。

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 [az storage blob list](/cli/azure/storage/blob) 命令列出容器中的 blob。

```azurecli-interactive
az storage blob list \
    --container-name mystoragecontainer \
    --output table
```

## <a name="download-a-blob"></a>下载 Blob

使用 [az storage blob download](/cli/azure/storage/blob) 命令下载之前上传的 Blob。

```azurecli-interactive
az storage blob download \
    --container-name mystoragecontainer \
    --name blobName \
    --file ~/destination/path/for/file
```

## <a name="data-transfer-with-azcopy"></a>使用 AzCopy 传输数据

若要按可编写脚本的方式高性能地传输 Azure 存储中的数据，还可使用 [AzCopy](../common/storage-use-azcopy-linux.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 实用工具。 可使用 AzCopy 将数据传输到 Blob、文件和表存储或将数据从中传出。

作为一个快速示例，下面是用于将名为 myfile.txt 的文件上传到 mystoragecontainer 容器中的 AzCopy 命令。

```bash
azcopy \
    --source /mnt/myfiles \
    --destination https://mystorageaccount.blob.core.windows.net/mystoragecontainer \
    --dest-key <storage-account-access-key> \
    --include "myfile.txt"
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要你的资源组中的任何一个资源（包括使用本教程创建的存储帐户），可使用 [az group delete](/cli/azure/group) 命令删除该资源组。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在此快速入门教程中，介绍了如何在本地磁盘和 Azure Blob 存储中的容器之间传输文件。 若要详细了解如何使用 Azure 存储中的 blob，请继续通过本教程了解如何使用 Azure Blob 存储。

> [!div class="nextstepaction"]
> [如何：通过 Azure CLI 对 Blob 存储执行操作](storage-how-to-use-blobs-cli.md)
