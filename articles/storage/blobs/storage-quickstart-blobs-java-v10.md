---
title: Azure 快速入门：使用 Java 存储 SDK V10 在对象存储中创建 Blob | Microsoft Docs
description: 本快速入门介绍如何使用 Java 存储 SDK 在对象 (Azure Blob) 存储中创建容器、上传文件、列出对象以及进行下载。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 11/14/2018
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: f023f3fdbc82960f3994f8e5763310dc95ca1c2f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65152800"
---
# <a name="quickstart-upload-download-and-list-blobs-by-using-the-java-storage-sdk-v10"></a>快速入门：使用 Java 存储 SDK V10 上传、下载和列出 Blob

本快速入门介绍如何使用新的 Java 存储 SDK 上传、下载和列出 Azure Blob 存储的容器中的块 Blob。 新的 Java SDK 将响应式编程模型与 RxJava 配合使用，提供异步操作。 详细了解 RxJava 的 [Java VM 响应式扩展](https://github.com/ReactiveX/RxJava)。 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

请确保已安装下述额外的必备组件：

* 可以通过命令行使用的 [Maven](https://maven.apache.org/download.cgi)，或者任何首选的 Java 集成开发环境。
* [JDK](https://aka.ms/azure-jdks)

## <a name="download-the-sample-application"></a>下载示例应用程序

本快速入门中使用的[示例应用程序](https://github.com/Azure-Samples/storage-blobs-java-v10-quickstart)是基本的控制台应用程序。 

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-v10-quickstart.git
```

此命令会将存储库克隆到本地 git 文件夹。

项目导入完以后，请打开位于 **src/main/java/quickstart** 中的 **Quickstart.java**。

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串
此解决方案要求安全地存储存储帐户的名称和密钥。 请将其存储在运行示例的计算机的本地环境变量中。 根据操作系统按照 Linux 或 Windows 示例创建环境变量。

### <a name="linux-example"></a>Linux 示例

```
export AZURE_STORAGE_ACCOUNT="<youraccountname>"
export AZURE_STORAGE_ACCESS_KEY="<youraccountkey>"
```

### <a name="windows-example"></a>Windows 示例

```
setx AZURE_STORAGE_ACCOUNT "<youraccountname>"
setx AZURE_STORAGE_ACCESS_KEY "<youraccountkey>"
```

## <a name="run-the-sample"></a>运行示例

此示例在默认目录 **AppData\Local\Temp** 中为 Windows 用户创建一个测试文件。 然后，它会提示执行以下步骤：

1. 输入将测试文件上传到 Azure Blob 存储的命令。
2. 列出容器中的 Blob。
3. 使用新名称下载已上传的文件，以便对旧文件和新文件进行比较。 

如果要在命令行使用 Maven 运行示例，请打开 shell 并浏览到已克隆目录中的 **storage-blobs-java-v10-quickstart**。 然后输入 `mvn compile exec:java`。

如果在 Windows 上运行应用程序，则此示例显示输出。

```
Created quickstart container
Enter a command
(P)utBlob | (L)istBlobs | (G)etBlob | (D)eleteBlobs | (E)xitSample
# Enter a command :
P
Uploading the sample file into the container: https://<storageaccount>.blob.core.windows.net/quickstart
# Enter a command :
L
Listing blobs in the container: https://<storageaccount>.blob.core.windows.net/quickstart
Blob name: SampleBlob.txt
# Enter a command :
G
Get the blob: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt
The blob was downloaded to C:\Users\<useraccount>\AppData\Local\Temp\downloadedFile13097087873115855761.txt
# Enter a command :
D
Delete the blob: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt

# Enter a command :
>> Blob deleted: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt
E
Cleaning up the sample and exiting!
```

你可以控制此示例，因此请输入命令，让其运行代码。 输入区分大小写。

还可以使用工具（如 [Azure 存储资源管理器](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)）查看 Blob 存储中的文件。 Azure 存储资源管理器是免费的跨平台工具，用于访问存储帐户信息。 

验证文件。 然后，请选择 **E** 和 **Enter**，以便完成演示并删除测试文件。 了解此示例的用途以后，请打开 **Quickstart.java** 文件，查看代码。 

## <a name="understand-the-sample-code"></a>了解示例代码

以下部分逐步介绍示例代码，以便展示其工作方式。

### <a name="get-references-to-the-storage-objects"></a>获取对存储对象的引用

首先，请创建对用于访问和管理 Blob 存储的对象的引用。 这些对象相互关联， 每个对象被列表中的下一个对象使用。

1. 创建指向存储帐户的 **StorageURL** 对象的实例。

    * [StorageURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._storage_u_r_l?view=azure-java-stable) 对象是存储帐户的表示形式， 可用于生成新的管道。 
    * 管道是一组策略，用于通过授权、日志记录和重试机制处理请求和响应。 有关详细信息，请参阅 [HTTP 管道](https://github.com/Azure/azure-storage-java/wiki/Azure-Storage-Java-V10-Overview#url-types--http-pipeline)。  
    * 使用管道创建 [ServiceURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._service_u_r_l?view=azure-java-stable) 对象的实例。
    * 使用 **ServiceURL** 对象创建 [ContainerURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l?view=azure-java-stable) 的实例。
    * **ContainerURL** 是在 Blob 容器上运行操作所必需的。

2. 创建 **ContainerURL** 对象的实例，该对象代表所访问的容器。 容器组织 Blob 的方式跟计算机上的文件夹组织文件一样。

    * **ContainerURL** 可以充当访问点来访问容器服务。 
    * 可以使用 [ContainerURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l?view=azure-java-stable) 创建 [BlobURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._blob_u_r_l?view=azure-java-stable) 对象的实例。
    * **BlobURL** 是创建 Blob 所必需的。

3. 创建 **BlobURL** 对象的实例，该对象指向你感兴趣的特定 Blob。 

> [!IMPORTANT]
> 容器名称必须为小写。 有关容器名称和 Blob 名称的详细信息，请参阅 [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)（命名和引用容器、Blob 和元数据）。

### <a name="create-a-container"></a>创建容器 

在此部分，请创建 **ContainerURL** 的实例， 并使用它创建一个新容器。 示例中的容器名为 **quickstart**。 

此示例使用 [containerURL.create](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.create?view=azure-java-stable#com_microsoft_azure_storage_blob__container_u_r_l_create_Metadata_PublicAccessType_Context_) ，这样你就可以每次运行示例时都创建新容器。 也可以提前创建容器，这样就无需在代码中创建它。

```java
// Create a ServiceURL to call the Blob service. We will also use this to construct the ContainerURL
SharedKeyCredentials creds = new SharedKeyCredentials(accountName, accountKey);
// We are using a default pipeline here, you can learn more about it at https://github.com/Azure/azure-storage-java/wiki/Azure-Storage-Java-V10-Overview
final ServiceURL serviceURL = new ServiceURL(new URL("http://" + accountName + ".blob.core.windows.net"), StorageURL.createPipeline(creds, new PipelineOptions()));

// Let's create a container using a blocking call to Azure Storage
// If container exists, we'll catch and continue
containerURL = serviceURL.createContainerURL("quickstart");

try {
    ContainersCreateResponse response = containerURL.create(null, null).blockingGet();
    System.out.println("Container Create Response was " + response.statusCode());
} catch (RestException e){
    if (e instanceof RestException && ((RestException)e).response().statusCode() != 409) {
        throw e;
    } else {
        System.out.println("quickstart container already exists, resuming...");
    }
}
```

### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

Blob 存储支持块 blob、追加 blob 和页 blob。 块 blob 是最常用的。 它们会在本快速入门中使用。 

1. 为了将文件上传到 blob，需获取对目标容器中 blob 的引用。 
2. 获取 Blob 引用以后，即可使用下述 API 之一将一个文件上传到该 Blob：

   * 低级别 API。 **BlockBlobURL** 实例中的示例包括 [BlockBlobURL.upload](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._block_blob_u_r_l.upload?view=azure-java-stable#com_microsoft_azure_storage_blob__block_blob_u_r_l_upload_Flowable_ByteBuffer__long_BlobHTTPHeaders_Metadata_BlobAccessConditions_Context_)（也称 PutBlob）和 [BlockBlobURL.stageBlock](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._block_blob_u_r_l.stageblock?view=azure-java-stable)（也称 PutBLock）。 

   * 在 [TransferManager class](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._transfer_manager?view=azure-java-stable)（TransferManager 类）中提供的高级别 API。 例如，[TransferManager.uploadFileToBlockBlob](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._transfer_manager.uploadfiletoblockblob?view=azure-java-stable) 方法。 

     此操作创建 Blob（如果该 Blob 尚不存在）， 或者覆盖 Blob（如果该 Blob 已存在）。

示例代码创建用于上传和下载的本地文件。 它将要上传的文件存储为 **sourceFile**，并在 **blob** 中存储 Blob 的 URL。 以下示例将文件上传到名为“quickstart”的容器。

```java
static void uploadFile(BlockBlobURL blob, File sourceFile) throws IOException {

    FileChannel fileChannel = FileChannel.open(sourceFile.toPath());

    // Uploading a file to the blobURL using the high-level methods available in TransferManager class
    // Alternatively call the Upload/StageBlock low-level methods from BlockBlobURL type
    TransferManager.uploadFileToBlockBlob(fileChannel, blob, 8*1024*1024, null)
        .subscribe(response-> {
            System.out.println("Completed upload request.");
            System.out.println(response.response().statusCode());
        });
}
```

块 blob 可以是任何类型的文本或二进制文件。 页 blob 主要用于用于备份 IaaS VM 的 VHD 文件。 追加 Blob 用于将数据追加到末尾，通常用于日志记录。 存储在 Blob 存储中的大多数对象都是块 blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

可以使用 [ContainerURL.listBlobsFlatSegment](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.listblobsflatsegment?view=azure-java-stable) 获取容器中对象的列表。 此方法一次最多返回 5000 个对象，如果容器中还有更多需要列出的对象，则还会返回一个继续（后续）操作标记。 创建一个帮助程序函数，当上一 **listBlobsFlatSegment** 响应中存在后续操作标记时，该函数会重复进行自身调用。

```java
static void listBlobs(ContainerURL containerURL) {
    // Each ContainerURL.listBlobsFlatSegment call return up to maxResults (maxResults=10 passed into ListBlobOptions below).
    // To list all Blobs, we are creating a helper static method called listAllBlobs,
    // and calling it after the initial listBlobsFlatSegment call
    ListBlobsOptions options = new ListBlobsOptions(null, null, 10);

    containerURL.listBlobsFlatSegment(null, options)
        .flatMap(containersListBlobFlatSegmentResponse ->
            listAllBlobs(containerURL, containersListBlobFlatSegmentResponse))
                .subscribe(response-> {
                    System.out.println("Completed list blobs request.");
                    System.out.println(response.statusCode());
                });
}

private static Single <ContainersListBlobFlatSegmentResponse> listAllBlobs(ContainerURL url, ContainersListBlobFlatSegmentResponse response) {
    // Process the blobs returned in this result segment (if the segment is empty, blobs() will be null.
    if (response.body().blobs() != null) {
        for (Blob b : response.body().blobs().blob()) {
            String output = "Blob name: " + b.name();
            if (b.snapshot() != null) {
                output += ", Snapshot: " + b.snapshot();
            }
            System.out.println(output);
        }
    }
    else {
        System.out.println("There are no more blobs to list off.");
    }

    // If there is not another segment, return this response as the final response.
    if (response.body().nextMarker() == null) {
        return Single.just(response);
    } else {
        /*
        IMPORTANT: ListBlobsFlatSegment returns the start of the next segment; you MUST use this to get the next
        segment (after processing the current result segment
        */

        String nextMarker = response.body().nextMarker();

        /*
        The presence of the marker indicates that there are more blobs to list, so we make another call to
        listBlobsFlatSegment and pass the result through this helper function.
        */

        return url.listBlobsFlatSegment(nextMarker, new ListBlobsOptions(null, null,1))
            .flatMap(containersListBlobFlatSegmentResponse ->
                listAllBlobs(url, containersListBlobFlatSegmentResponse));
    }
}
```

### <a name="download-blobs"></a>下载 Blob

使用 [BlobURL.download](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._blob_u_r_l.download?view=azure-java-stable) 将 Blob 下载到本地磁盘。

以下代码将下载前面部分所上传的 blob。 它会将后缀 **_DOWNLOADED** 添加到 Blob 名称，因此可以在本地磁盘上看到这两个文件。 

```java
static void getBlob(BlockBlobURL blobURL, File sourceFile) {
    try {
        // Get the blob using the low-level download method in BlockBlobURL type
        // com.microsoft.rest.v2.util.FlowableUtil is a static class that contains helpers to work with Flowable
        blobURL.download(new BlobRange(0, Long.MAX_VALUE), null, false)
            .flatMapCompletable(response -> {
                AsynchronousFileChannel channel = AsynchronousFileChannel.open(Paths
                    .get(sourceFile.getPath()), StandardOpenOption.CREATE,  StandardOpenOption.WRITE);
                        return FlowableUtil.writeFile(response.body(), channel);
            }).doOnComplete(()-> System.out.println("The blob was downloaded to " + sourceFile.getAbsolutePath()))
            // To call it synchronously add .blockingAwait()
            .subscribe();
    } catch (Exception ex){
        System.out.println(ex.toString());
    }
}
```

### <a name="clean-up-resources"></a>清理资源

如果不需要在本快速入门中上传的 Blob，可使用 [ContainerURL.delete](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.delete?view=azure-java-stable) 删除整个容器。 此方法也会删除容器中的文件。

```java
containerURL.delete(null).blockingGet();
```

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何使用 Java 在本地磁盘和 Azure Blob 存储之间传输文件。 

> [!div class="nextstepaction"]
> [用于 Java 的存储 SDK V10 源代码](https://github.com/Azure/azure-storage-java/)
> [API 参考](https://docs.microsoft.com/java/api/overview/azure/storage/client?view=azure-java-stable)
> [详细了解 RxJava](https://github.com/ReactiveX/RxJava)
