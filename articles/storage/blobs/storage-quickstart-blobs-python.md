---
title: Azure 快速入门 - 使用 Python 在对象存储中创建 Blob | Microsoft Docs
description: 本快速入门将在对象 (Blob) 存储中创建存储帐户和容器。 然后，使用适用于 Python 的存储客户端库将一个 Blob 上传到 Azure 存储，下载一个 Blob，然后列出容器中的 Blob。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 12/14/2018
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: 0c40d0985b0d6c967a55b1954a1cb54feeb15361
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149388"
---
# <a name="quickstart-upload-download-and-list-blobs-with-python"></a>快速入门：使用 Python 上传、下载和列出 Blob

本快速入门介绍如何使用 Python 上传、下载和列出 Azure Blob 存储的容器中的块 Blob。 Blob 是简单的对象，可以保留任何数量的文本或二进制数据（例如图像、文档、流媒体、存档数据等），并在文件共享、无架构表格和消息队列方面与 Azure 存储有所不同。 （有关详细信息，请参阅 [Azure 存储简介](/azure/storage/common/storage-introduction)。）

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

请确保已安装下述额外的必备组件：

* [Python](https://www.python.org/downloads/)
* [用于 Python 的 Azure 存储 SDK](https://github.com/Azure/azure-sdk-for-python)

## <a name="download-the-sample-application"></a>下载示例应用程序
本快速入门中的[示例应用程序](https://github.com/Azure-Samples/storage-blobs-python-quickstart.git)是基本的 Python 应用程序。  

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-python-quickstart.git 
```

此命令会将 *Azure-Samples/storage-blobs-python-quickstart* 存储库克隆到本地 git 文件夹。 若要运行 Python 程序，请在存储库的根目录中打开 *example.py* 文件。  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串
在应用程序中，请提供存储帐户名称和帐户密钥，以创建 `BlockBlobService` 对象。 从 IDE 中的解决方案资源管理器打开 *example.py* 文件。 将 `accountname` 和 `accountkey` 值替换为自己的帐户名和密钥。 

```python 
block_blob_service = BlockBlobService(account_name = 'accountname', account_key = 'accountkey') 
```

## <a name="run-the-sample"></a>运行示例
此示例在 *Documents* 文件夹中创建一个测试文件。 示例程序会将该测试文件上传到 Blob 存储，列出容器中的 blob，并使用新名称下载此文件。 

首先，通过运行 `pip install` 安装依赖项：

    pip install azure-storage-blob

接下来，运行示例。 此时会看到类似于以下输出的消息：
  
```
Temp file = C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Uploading to Blob storage as blobQuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

List blobs in the container
         Blob name: QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Downloading blob to C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078_DOWNLOADED.txt
```
继续前，请在 *Documents* 文件夹中查看这两个文件。 可以打开它们，看它们是否相同。

还可以使用工具（如 [Azure 存储资源管理器](https://storageexplorer.com)）查看 Blob 存储中的文件。 Azure 存储资源管理器是免费的跨平台工具，可用于访问存储帐户信息。 

验证文件后，按任意键可完成演示并删除测试文件。 了解此示例的用途以后，即可打开 *example.py* 文件来查看代码。 

## <a name="understand-the-sample-code"></a>了解示例代码

让我们逐步查看示例代码，了解其工作方式。

### <a name="get-references-to-the-storage-objects"></a>获取对存储对象的引用
首先，请创建对用于访问和管理 Blob 存储的对象的引用。 这些对象相互关联，并且每个对象被列表中的下一个对象使用。

* 实例化 BlockBlobService 对象，该对象指向存储帐户中的 Blob 服务。 

* 实例化 CloudBlobContainer 对象，该对象代表你正在访问的容器。 容器用于组织 blob，就像使用计算机上的文件夹组织文件一样。

有了云 Blob 容器后，请实例化 CloudBlockBlob 对象（该对象指向你感兴趣的特定 Blob）。 然后即可根据需要上传、下载和复制 Blob。

> [!IMPORTANT]
> 容器名称必须为小写。 有关容器名称和 Blob 名称的详细信息，请参阅 [Naming and referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)（命名和引用容器、Blob 和元数据）。

此部分将实例化对象，新建容器，然后设置容器的权限，以便 blob 成为公共 blob。 容器名称为 quickstartblobs。 

```python 
# Create the BlockBlockService that is used to call the Blob service for the storage account.
block_blob_service = BlockBlobService(account_name = 'accountname', account_key = 'accountkey') 
 
# Create a container called 'quickstartblobs'.
container_name = 'quickstartblobs'
block_blob_service.create_container(container_name) 

# Set the permission so the blobs are public.
block_blob_service.set_container_acl(container_name, public_access=PublicAccess.Container)
```
### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

Blob 存储支持块 blob、追加 blob 和页 blob。 块 blob 最常用，此快速入门中也使用块 blob。  

若要将文件上传到 Blob，请通过将本地驱动器上的目录名称和文件名称联接在一起来获取完整的文件路径。 然后可以使用 `create_blob_from_path` 方法将文件上传到指定的路径。 

示例代码将创建一个本地文件，以供上传和下载，并将要上传的此文件存储为 full_path_to_file，将 blob 的名称存储为 local_file_name。 以下示例将文件上传到名为“quickstartblobs”的容器。

```python
# Create a file in Documents to test the upload and download.
local_path = os.path.expanduser("~\Documents")
local_file_name = "QuickStart_" + str(uuid.uuid4()) + ".txt"
full_path_to_file = os.path.join(local_path, local_file_name)

# Write text to the file.
file = open(full_path_to_file, 'w')
file.write("Hello, World!")
file.close()

print("Temp file = " + full_path_to_file)
print("\nUploading to Blob storage as blob" + local_file_name)

# Upload the created file, use local_file_name for the blob name.
block_blob_service.create_blob_from_path(container_name, local_file_name, full_path_to_file)
```

Blob 存储支持多种上传方法。 例如，若有一个内存流，则可使用 `create_blob_from_stream` 方法而不是 `create_blob_from_path` 方法。 

块 blob 最大可以为 4.7 TB，并且可以是从 Excel 电子表格到大视频文件的任何内容。 页 Blob 主要用于支持 IaaS VM 的 VHD 文件。 追加 blob 用于日志记录，例如有时需要写入到文件，再继续添加更多信息。 存储在 Blob 存储中的大多数对象都是块 blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 `list_blobs` 方法获取容器中文件的列表。 此方法会返回一个生成器。 下面的代码检索 Blob 列表，然后循环访问它们，显示在容器中找到的 Blob 的名称。  

```python
# List the blobs in the container.
print("\nList blobs in the container")
generator = block_blob_service.list_blobs(container_name)
for blob in generator:
    print("\t Blob name: " + blob.name)
```

### <a name="download-the-blobs"></a>下载 Blob

使用 `get_blob_to_path` 方法将 Blob 下载到本地磁盘。 以下代码将下载前面部分所上传的 blob。 将 *_DOWNLOADED* 添加为 Blob 名称的前缀后，就可以在本地磁盘上同时看到这两个文件。 

```python
# Download the blob(s).
# Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.
full_path_to_file2 = os.path.join(local_path, string.replace(local_file_name, '.txt', '_DOWNLOADED.txt'))
print("\nDownloading blob to " + full_path_to_file2)
block_blob_service.get_blob_to_path(container_name, local_file_name, full_path_to_file2)
```

### <a name="clean-up-resources"></a>清理资源
如果不再需要本快速入门中上传的 Blob，可使用 `delete_container` 方法删除整个容器。 若要改为删除单个文件，请使用 `delete_blob` 方法。

```python
# Clean up resources. This includes the container and the temp files.
block_blob_service.delete_container(container_name)
os.remove(full_path_to_file)
os.remove(full_path_to_file2)
```
## <a name="resources-for-developing-python-applications-with-blobs"></a>用于开发包含 Blob 的 Python 应用程序的资源

若要详细了解如何使用 Blob 存储进行 Python 开发，请参阅下述额外资源：

### <a name="binaries-and-source-code"></a>二进制文件和源代码

- 在 GitHub 上查看、下载并安装 Azure 存储的 [Python 客户端库源代码](https://github.com/Azure/azure-storage-python)。

### <a name="client-library-reference-and-samples"></a>客户端库参考和示例

- 有关 Python 客户端库的详细信息，请查看 [Python API 参考](https://docs.microsoft.com/python/api/overview/azure/storage)。
- 浏览使用 Python 客户端库编写的 [Blob 存储示例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=python&term=blob)。

## <a name="next-steps"></a>后续步骤
 
本快速入门介绍了如何使用 Python 在本地磁盘和 Azure Blob 存储之间传输文件。 要深入了解如何使用 Blob 存储，请继续学习 Blob 存储操作说明。

> [!div class="nextstepaction"]
> [Blob 存储操作说明](./storage-python-how-to-use-blob-storage.md)
 
若要详细了解存储资源管理器和 Blob，请参阅[使用存储资源管理器管理 Azure Blob 存储资源](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
