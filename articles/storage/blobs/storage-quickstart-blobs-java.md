---
title: 如何使用适用于 Java v7 的客户端库在 Azure 存储中创建 Blob | Microsoft Docs
description: 在对象 (Blob) 存储中创建存储帐户和容器。 随后，使用适用于 Java v7 的 Azure 存储客户端库将一个 Blob 上传到 Azure 存储，下载一个 Blob，然后列出容器中的 Blob。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: conceptual
ms.date: 02/04/2019
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: f7cae5b3c7b0a7da6420674635ff9c3420a6436a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154410"
---
# <a name="how-to-upload-download-and-list-blobs-using-the-client-library-for-java-v7"></a>如何使用适用于 Java v7 的客户端库上传、下载和列出 Blob

本操作指南介绍如何使用适用于 Java v7 的客户端库上传、下载和列出 Azure Blob 存储容器中的块 Blob。

> [!TIP]
> 适用于 Java 的 Azure 存储客户端库的最新版本是 v10。 Microsoft 建议尽可能使用最新版本的客户端库。 若要开始使用 v10，请参阅[快速入门：使用 Java 存储 SDK V10 上传、下载和列出 Blob](storage-quickstart-blobs-java-v10.md)。

## <a name="prerequisites"></a>必备组件

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

同时在 [Azure 门户](https://portal.azure.com/#create/Microsoft.StorageAccount-ARM)中创建 Azure 存储帐户。 有关如何创建帐户的帮助，请参阅[创建存储帐户](../common/storage-quickstart-create-account.md)。

确保满足以下先决条件：

* 安装含有 Maven 集成的 IDE。

* 或者，从命令行安装和配置 Maven。

本指南使用具有“适用于 Java 开发者的 Eclipse IDE”配置的 [Eclipse](https://www.eclipse.org/downloads/)。

## <a name="download-the-sample-application"></a>下载示例应用程序

[示例应用程序](https://github.com/Azure-Samples/storage-blobs-java-quickstart)是基本的控制台应用程序。  

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-quickstart.git
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开项目，请启动 Eclipse 并关闭欢迎屏幕。 选择“文件”，然后选择“从文件系统打开项目”。 请确保已选中“检测并配置项目性质”。 选择“目录”，然后导航到存储克隆存储库的位置。 在克隆的存储库中，选择 **blobAzureApp** 文件夹。 请确保 blobAzureApp 项目显示为 Eclipse 项目，然后选择“完成”。

完成项目导入后，打开 AzureApp.java（位于 src/main/java 内的 blobQuickstart.blobAzureApp 中），并替换 `storageConnectionString` 字符串中的 `accountname` 和 `accountkey`。 然后运行应用程序。 以下部分是有关如何完成这些任务的具体说明。

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]    

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串
    
在应用程序中，必须为存储帐户提供连接字符串。 打开 AzureApp.Java 文件。 找到 `storageConnectionString` 变量，然后粘贴在上一部分复制的连接字符串值。 `storageConnectionString` 变量看起来应该类似于以下代码示例：

```java
public static final String storageConnectionString =
"DefaultEndpointsProtocol=https;" +
"AccountName=<account-name>;" +
"AccountKey=<account-key>";
```

## <a name="run-the-sample"></a>运行示例

此示例应用程序在默认目录（对于 Windows 用户，为 *C:\Users\<user>\AppData\Local\Temp*）中创建一个测试文件，将其上传到 Blob 存储，列出容器中的 Blob，然后下载具有新名称的文件，以便比较旧文件和新文件。 

使用 Maven 在命令行运行示例。 打开 shell 并导航到已克隆目录中的 blobAzureApp。 然后输入 `mvn compile exec:java`。 

如果打算在 Windows 上运行应用程序，请参阅以下示例显示的输出。

```
Azure Blob storage quick start sample
Creating container: quickstartcontainer
Creating a sample file at: C:\Users\<user>\AppData\Local\Temp\sampleFile514658495642546986.txt
Uploading the sample file 
URI of blob is: https://myexamplesacct.blob.core.windows.net/quickstartcontainer/sampleFile514658495642546986.txt
The program has completed successfully.
Press the 'Enter' key while in the console to delete the sample files, example container, and exit the application.

Deleting the container
Deleting the source, and downloaded files
```

检查示例文件的默认目录（对于 Windows 用户，为 *C:\Users\<user>\AppData\Local\Temp*），再继续操作。 从控制台窗口复制 blob 的 URL，将其粘贴到浏览器，查看 Blob 存储中的文件的内容。 如果将目录中的示例文件与 Blob 存储中的内容进行比较，则会发现它们是相同的。 

  >[!NOTE]
  >还可以使用工具（如 [Azure 存储资源管理器](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)）查看 Blob 存储中的文件。 Azure 存储资源管理器是免费的跨平台工具，可用于访问存储帐户信息。

验证文件后，按 **Enter** 键可完成演示并删除测试文件。 现在已了解此示例的用途，打开 AzureApp.java 文件可查看代码。 

## <a name="understand-the-sample-code"></a>了解示例代码

接下来逐步介绍示例代码，以便展示其工作方式。

### <a name="get-references-to-the-storage-objects"></a>获取对存储对象的引用

首先创建对用于访问和管理 Blob 存储的对象的引用。 这些对象相互关联 - 每个对象被列表中的下一个对象使用。

* 创建指向存储帐户的 [CloudStorageAccount](/java/api/com.microsoft.azure.management.storage.storageaccount) 对象的实例。

    “CloudStorageAccount”对象是存储帐户的表示形式，允许用户以编程方式设置和访问存储帐户属性。 使用“CloudStorageAccount”对象，可创建访问 blob 服务所需的“CloudBlobClient”实例。

* 创建 CloudBlobClient 对象的实例，该对象指向存储帐户中的 [Blob 服务](/java/api/com.microsoft.azure.storage.blob._cloud_blob_client)。

    “CloudBlobClient”提供对 Blob 服务的访问点，允许用户以编程方式设置和访问 Blob 存储属性。 使用“CloudBlobClient”，可创建“CloudBlobContainer”对象的实例，创建容器需要该实例。

* 创建 [CloudBlobContainer](/java/api/com.microsoft.azure.storage.blob._cloud_blob_container) 对象的实例，该对象代表所访问的容器。 使用容器来组织 Blob，就像使用计算机上的文件夹组织文件一样。    

    有了 **CloudBlobContainer** 后，就可以创建 [CloudBlockBlob](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob) 对象（该对象指向你感兴趣的特定 Blob）的实例，然后执行上传、下载、复制等操作。

> [!IMPORTANT]
> 容器名称必须为小写。 有关容器的详细信息，请参阅[命名和引用容器、Blob 和元数据](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

### <a name="create-a-container"></a>创建容器

在本部分中，将创建对象的实例、创建新容器，并对容器设置权限，使 blob 公开，只需 URL 即可对其进行访问。 容器名为 quickstartcontainer。 

此示例使用 [CreateIfNotExists](/java/api/com.microsoft.azure.storage.blob._cloud_blob_container.createifnotexists) ，因为我们想要每次运行示例时都创建新容器。 在生产环境中，应用程序从头至尾都使用相同的容器，因此建议仅调用一次 **CreateIfNotExists**。 或者可以提前创建容器，这样就无需在代码中创建它。

```java
// Parse the connection string and create a blob client to interact with Blob storage
storageAccount = CloudStorageAccount.parse(storageConnectionString);
blobClient = storageAccount.createCloudBlobClient();
container = blobClient.getContainerReference("quickstartcontainer");

// Create the container if it does not exist with public access.
System.out.println("Creating container: " + container.getName());
container.createIfNotExists(BlobContainerPublicAccessType.CONTAINER, new BlobRequestOptions(), new OperationContext());
```

### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

若要将文件上传到块 Blob，请获取对目标容器中的 Blob 的引用。 有了 blob 引用后，可以通过使用 [CloudBlockBlob.Upload](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.upload) 将数据上传到其中。 此操作会创建 Blob（如果尚未存在），或者覆盖 Blob（若已存在）。

示例代码创建一个用于上传和下载的本地文件，存储作为“源”上传的文件和 blob 中的 blob 名称。 以下示例将文件上传到名为“quickstartcontainer”的容器。

```java
//Creating a sample file
sourceFile = File.createTempFile("sampleFile", ".txt");
System.out.println("Creating a sample file at: " + sourceFile.toString());
Writer output = new BufferedWriter(new FileWriter(sourceFile));
output.write("Hello Azure!");
output.close();

//Getting a blob reference
CloudBlockBlob blob = container.getBlockBlobReference(sourceFile.getName());

//Creating blob and uploading file to it
System.out.println("Uploading the sample file ");
blob.uploadFromFile(sourceFile.getAbsolutePath());
```

有多个可以与 Blob 存储配合使用的 `upload` 方法，其中包括 [upload](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.upload)、[uploadBlock](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.uploadblock)、[uploadFullBlob](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.uploadfullblob)、[uploadStandardBlobTier](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.uploadstandardblobtier)、[uploadText](/java/api/com.microsoft.azure.storage.blob._cloud_block_blob.uploadtext)。 例如，如果有字符串，可以使用 `UploadText` 方法，而不是 `Upload` 方法。 

块 blob 可以是任何类型的文本或二进制文件。 页 Blob 主要用于支持 IaaS VM 的 VHD 文件。 将追加 Blob 用于日志记录，例如有时需要写入到文件，再继续添加更多信息。 存储在 Blob 存储中的大多数对象都是块 blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

可以使用 [CloudBlobContainer.ListBlobs](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._cloud_blob_container.listblobs) 获取容器中的文件列表。 下面的代码检索 blob 列表，然后循环访问它们，显示找到的 blob 的 URI。 可以从命令窗口中复制 URI，然后将其粘贴到浏览器以查看文件。

```java
//Listing contents of container
for (ListBlobItem blobItem : container.listBlobs()) {
    System.out.println("URI of blob is: " + blobItem.getUri());
}
```

### <a name="download-blobs"></a>下载 Blob

使用 [CloudBlob.DownloadToFile](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._cloud_blob.downloadtofile) 将 blob 下载到本地磁盘。

以下代码下载上一部分上传的 blob，对 blob 名称添加“_DOWNLOADED”后缀，以便可以在本地磁盘上看到两个文件。 

```java
// Download blob. In most cases, you would have to retrieve the reference
// to cloudBlockBlob here. However, we created that reference earlier, and 
// haven't changed the blob we're interested in, so we can reuse it. 
// Here we are creating a new file to download to. Alternatively you can also pass in the path as a string into downloadToFile method: blob.downloadToFile("/path/to/new/file").
downloadedFile = new File(sourceFile.getParentFile(), "downloadedFile.txt");
blob.downloadToFile(downloadedFile.getAbsolutePath());
```

### <a name="clean-up-resources"></a>清理资源

如果不再需要上传的 Blob，可使用 [CloudBlobContainer.DeleteIfExists](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._cloud_blob_container.deleteifexists) 删除整个容器。 此方法也会删除容器中的文件。

```java
try {
if(container != null)
    container.deleteIfExists();
} catch (StorageException ex) {
System.out.println(String.format("Service error. Http code: %d and error code: %s", ex.getHttpStatusCode(), ex.getErrorCode()));
}

System.out.println("Deleting the source, and downloaded files");

if(downloadedFile != null)
downloadedFile.deleteOnExit();
        
if(sourceFile != null)
sourceFile.deleteOnExit();
```

## <a name="next-steps"></a>后续步骤

本文介绍如何使用 Java 在本地磁盘和 Azure Blob 存储之间传输文件。 若要详细了解 Java 的用法，请转到 GitHub 源代码存储库。

> [!div class="nextstepaction"]
> [适用于 Java 的 Microsoft Azure 存储 SDK v10](https://github.com/azure/azure-storage-java) 
> [Java API 参考](https://docs.microsoft.com/java/azure/)
> [适用于 Java 的代码示例](../common/storage-samples-java.md)
