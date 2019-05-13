---
title: 结合使用 Azure 经典 CLI 与 Azure 存储 | Microsoft 文档
description: 了解如何结合使用 Azure 经典命令行接口 (CLI) 与 Azure 存储，以便创建和管理存储帐户并处理 Azure blob 和文件。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 01/30/2017
ms.author: tamram
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: 88f713c5695e2453edc58d072899aa417f0514af
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147042"
---
# <a name="using-the-azure-classic-cli-with-azure-storage"></a>结合使用 Azure 经典 CLI 与 Azure 存储

## <a name="overview"></a>概述

Azure 经典 CLI 提供了一组开源且跨平台的命令，可结合 Azure 平台使用。 它提供 [Azure 门户](https://portal.azure.com)所提供的很多相同功能，此外还有各种数据访问功能。

在本指南中，将探讨如何使用 [Azure 经典 CLI](../../cli-install-nodejs.md)，以通过 Azure 存储执行各种开发和管理任务。 在使用本指南之前，我们建议下载和安装或者升级到最新版的经典 CLI。

本指南假定你了解 Azure 存储的基本概念。 本指南提供了大量的脚本，用于演示结合使用经典 CLI 与 Azure 存储的用法。 在运行每个脚本之前，请确保根据配置更新脚本变量。

> [!NOTE]
> 本指南提供了用于经典存储帐户的 Azure 经典 CLI 命令和脚本示例。 若要了解如何使用适用于“资源管理器”存储帐户的 Azure 经典 CLI 命令，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](../../virtual-machines/azure-cli-arm-commands.md#azure-storage-commands-to-manage-your-storage-objects)。
>
>

[!INCLUDE [storage-cli-versions](../../../includes/storage-cli-versions.md)]

## <a name="get-started-with-azure-storage-and-the-azure-classic-cli-in-5-minutes"></a>在 5 分钟内开始使用 Azure 存储和 Azure 经典 CLI
本指南使用 Ubuntu 作为示例，但其他 OS 平台的操作应与此类似。

**第一次使用 Azure：** 获取 Microsoft Azure 订阅，以及与此订阅关联的 Microsoft 帐户。 有关 Azure 购买选项的信息，请参阅[免费试用](https://azure.microsoft.com/pricing/free-trial/)、[购买选项](https://azure.microsoft.com/pricing/purchase-options/)和[会员套餐](https://azure.microsoft.com/pricing/member-offers/)（适用于 MSDN、Microsoft 合作伙伴网络和 BizSpark 以及其他 Microsoft 计划的成员）。

请参阅[在 Azure Active Directory (Azure AD) 中分配管理员角色](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal)，以了解有关 Azure 订阅的更多信息。

**创建 Microsoft Azure 订阅和帐户之后：**

1. 按照[安装 Azure 经典 CLI](../../cli-install-nodejs.md) 中的概要说明，下载和安装 Azure 经典 CLI。
2. 安装了经典 CLI 之后，将能够从命令行接口（Bash、终端、命令提示符）使用 azure 命令访问经典 CLI 命令。 键入 _azure_ 命令，可看到以下输出。

    ![Azure 命令输出](./media/storage-azure-cli/azure_command.png)   
3. 在命令行接口中，键入 `azure storage` 即可列出所有 Azure 存储命令，并初步了解经典 CLI 提供的功能。 可以键入带 **-h** 参数的命令名称（例如，`azure storage share create -h`），了解命令语法的详细信息。
4. 现在，我们将提供一个简单的脚本，演示用于访问 Azure 存储的基本经典 CLI 命令。 该脚本会首先要求针对存储帐户和密钥设置两个变量。 然后，该脚本将在此新存储帐户中创建新容器，并将现有图像文件 (Blob) 上传到该容器。 脚本在列出该容器中的所有 Blob 后，就会将图像文件下载到本地计算机上的目标目录。

    ```azurecli
    #!/bin/bash
    # A simple Azure storage example

    export AZURE_STORAGE_ACCOUNT=<storage_account_name>
    export AZURE_STORAGE_ACCESS_KEY=<storage_account_key>

    export container_name=<container_name>
    export blob_name=<blob_name>
    export image_to_upload=<image_to_upload>
    export destination_folder=<destination_folder>

    echo "Creating the container..."
    azure storage container create $container_name

    echo "Uploading the image..."
    azure storage blob upload $image_to_upload $container_name $blob_name

    echo "Listing the blobs..."
    azure storage blob list $container_name

    echo "Downloading the image..."
    azure storage blob download $container_name $blob_name $destination_folder

    echo "Done"
    ```

5. 在本地计算机中，打开首选的文本编辑器（例如 vim）。 在文本编辑器中键入上述脚本。
6. 现在，需要基于配置设置更新脚本变量。

   * **<storage_account_name>**：使用脚本中给定的名称，或输入存储帐户的新名称。 **重要提示：** 在 Azure 中，存储帐户名必须是唯一的。 它还必须为小写！
   * **<storage_account_key>**：存储帐户的访问密钥。
   * **<container_name>**：使用脚本中给定的名称，或输入容器的新名称。
   * <image_to_upload>：输入本地计算机上图片的路径，例如：“~/images/HelloWorld.png”。
   * **<destination_folder>**：输入用于存储从 Azure 存储下载的文件的本地目录路径，例如：“~/downloadImages”。
7. 在 vim 中更新完必需的变量以后，按组合键 `ESC`、`:`、`wq!` 保存脚本。
8. 若要运行此脚本，在 bash 控制台中键入脚本文件名即可。 运行此脚本后，应会创建包含已下载图像文件的本地目标文件夹。 以下屏幕截图显示了示例输出：

运行脚本后，应会创建包含已下载图像文件的本地目标文件夹。

## <a name="manage-storage-accounts-with-the-azure-classic-cli"></a>通过 Azure 经典 CLI 管理存储帐户
### <a name="connect-to-your-azure-subscription"></a>连接到 Azure 订阅
尽管大多数存储命令没有 Azure 订阅也可以使用，不过仍建议通过经典 CLI 连接到订阅。

### <a name="create-a-new-storage-account"></a>新建存储帐户
要使用 Azure 存储，需要一个存储帐户。 可以在将计算机配置为连接到订阅之后，创建新的 Azure 存储帐户。

```azurecli
azure storage account create <account_name>
```

存储帐户名称必须为 3 到 24 个字符，并且只能使用数字和小写字母。

### <a name="set-a-default-azure-storage-account-in-environment-variables"></a>在环境变量中设置默认的 Azure 存储帐户
可以在订阅中设置多个存储帐户。 可以选择其中的一个存储帐户，并在环境变量中将其设置为同一会话中所有存储命令的默认存储帐户。 这样，便可以在不显式指定存储帐户和密钥的情况下运行经典 CLI 存储命令。

```azurecli
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_ACCESS_KEY=<key>
```

若要设置默认存储帐户，另一种方法是使用连接字符串。 首先，通过以下命令获取连接字符串：

```azurecli
azure storage account connectionstring show <account_name>
```

然后复制输出连接字符串，并将其设置为环境变量：

```azurecli
export AZURE_STORAGE_CONNECTION_STRING=<connection_string>
```

## <a name="create-and-manage-blobs"></a>创建并管理 blob
Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。 本部分假设已熟悉 Azure Blob 存储的概念。 有关详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](../blobs/storage-dotnet-how-to-use-blobs.md)和 [Blob 服务概念](https://msdn.microsoft.com/library/azure/dd179376.aspx)。

### <a name="create-a-container"></a>创建容器
Azure 存储中的每个 Blob 都必须在容器中。 可以使用 `azure storage container create` 命令创建专用容器：

```azurecli
azure storage container create mycontainer
```

> [!NOTE]
> 有三种级别的匿名读取访问：“关”、“Blob”和“容器”。 要防止对 Blob 进行匿名访问，请将 Permission 参数设置为 **Off**。 默认情况下，新容器是专用容器，只能由帐户所有者访问。 要允许对 Blob 资源进行匿名公共读取访问，但不允许访问容器元数据或容器中的 Blob 列表，请将 Permission 参数设置为 **Blob**。 要允许对 Blob 资源、容器元数据和容器中的 Blob 列表进行完全公开读取访问，请将 Permission 参数设置为 **Container**。 有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](../blobs/storage-manage-access-to-resources.md)。
>
>

### <a name="upload-a-blob-into-a-container"></a>将 Blob 上传到容器中
Azure Blob 存储支持块 Blob 和页 Blob。 有关详细信息，请参阅 [了解块 Blob、追加 Blob 和页 Blob](https://msdn.microsoft.com/library/azure/ee691964.aspx)。

要将 blob 上传到容器，可使用 `azure storage blob upload`。 默认情况下，此命令会将本地文件上传到块 Blob。 若要指定 Blob 的类型，可以使用 `--blobtype` 参数。

```azurecli
azure storage blob upload '~/images/HelloWorld.png' mycontainer myBlockBlob
```

### <a name="download-blobs-from-a-container"></a>从容器下载 Blob
以下示例演示如何从容器下载 Blob。

```azurecli
azure storage blob download mycontainer myBlockBlob '~/downloadImages/downloaded.png'
```

### <a name="copy-blobs"></a>复制 blob
可以在存储帐户和区域内或跨存储帐户和区域异步复制 Blob。

以下示例演示如何将一个存储帐户中的 Blob 复制到另一个存储帐户。 在此示例中，我们创建了一个容器，其中的 blob 可以进行公开、匿名的访问。

```azurecli
azure storage container create mycontainer2 -a <accountName2> -k <accountKey2> -p Blob

azure storage blob upload '~/Images/HelloWorld.png' mycontainer2 myBlockBlob2 -a <accountName2> -k <accountKey2>

azure storage blob copy start 'https://<accountname2>.blob.core.windows.net/mycontainer2/myBlockBlob2' mycontainer
```

此示例将执行异步复制。 可以通过运行 `azure storage blob copy show` 操作来监视每个复制操作的状态。

请注意，所提供的用于复制操作的源 URL 必须可以公开访问，或者必须包括一个 SAS（共享访问签名）令牌。

### <a name="delete-a-blob"></a>删除 Blob
若要删除 blob，请使用以下命令：

```azurecli
azure storage blob delete mycontainer myBlockBlob2
```

## <a name="create-and-manage-file-shares"></a>创建和管理文件共享
Azure 文件使用标准 SMB 协议为应用程序提供共享存储。 Microsoft Azure 虚拟机和云服务以及本地应用程序可以通过装载的共享来共享文件数据。 你可以通过经典 CLI 管理文件共享和文件数据。 有关 Azure 文件的详细信息，请参阅 [Azure 文件简介](../files/storage-files-introduction.md)。

### <a name="create-a-file-share"></a>创建文件共享
Azure 文件共享是 Azure 中的 SMB 文件共享。 所有目录和文件都必须在文件共享中创建。 一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到存储帐户的容量限制为止。 下面的示例创建名为 **myshare** 的文件共享。

```azurecli
azure storage share create myshare
```

### <a name="create-a-directory"></a>创建目录
目录提供了进行 Azure 文件共享时可以选择的层次结构。 以下示例在文件共享中创建名为 **myDir** 的目录。

```azurecli
azure storage directory create myshare myDir
```

请注意，目录路径可以包括多个级别， *例如***a/b**。 但是，必须确保所有父目录都存在。 例如，对于路径 a/b，必须先创建目录 a，然后创建目录 b。

### <a name="upload-a-local-file-to-directory"></a>将本地文件上传到目录
以下示例将文件从 ~/temp/samplefile.txt 上传到 myDir 目录。 请编辑文件路径，使其指向本地计算机上的有效文件：

```azurecli
azure storage file upload '~/temp/samplefile.txt' myshare myDir
```

请注意，共享中的文件最大可以为 1 TB。

### <a name="list-the-files-in-the-share-root-or-directory"></a>列出共享根目录或目录中的文件
可以使用以下命令列出共享根目录或目录中的文件和子目录：

```azurecli
azure storage file list myshare myDir
```

请注意，进行列表操作时，目录名是可选的。 如果省略目录名，该命令会列出共享中根目录的内容。

### <a name="copy-files"></a>复制文件
从经典 CLI 的 0.9.8 版开始，可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。 下面，我们演示如何使用 CLI 命令执行这些复制操作。 要将文件复制到新目录中：

```azurecli
azure storage file copy start --source-share srcshare --source-path srcdir/hello.txt --dest-share destshare
    --dest-path destdir/hellocopy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString
```

要将 blob 复制到一个文件目录中：

```azurecli
azure storage file copy start --source-container srcctn --source-blob hello2.txt --dest-share hello
    --dest-path hellodir/hello2copy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString
```

## <a name="next-steps"></a>后续步骤

你可以在此处查找使用存储资源的 Azure 经典 CLI 命令参考：

* [Resource Manager 模式下的 Azure 经典 CLI 命令](../../virtual-machines/azure-cli-arm-commands.md#azure-storage-commands-to-manage-your-storage-objects)
* [Azure 服务管理模式下的 Azure 经典 CLI 命令](../../cli-install-nodejs.md)

你可能还想尝试将最新版的 [Azure CLI](../storage-azure-cli.md) 与 Resource Manager 部署模型配合使用。
