---
title: Azure 快速入门 - 使用 PHP 在对象存储中创建 blob | Microsoft Docs
description: 快速了解如何使用 PHP 将对象转移到 Azure Blob 存储或从 Azure Blob 存储转移对象
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: php
ms.topic: quickstart
ms.date: 11/14/2018
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: 790c6a372021542daa1098e94209c91b7bcecfd5
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149454"
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-php"></a>使用 PHP 将对象转移到 Azure Blob 存储或从 Azure Blob 存储转移对象
本快速入门介绍如何使用 PHP 上传、下载和列出 Azure Blob 存储的容器中的块 Blob。 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

请确保已安装下述额外的必备组件：

* [PHP](https://php.net/downloads.php)
* [用于 PHP 的 Azure 存储 SDK](https://github.com/Azure/azure-storage-php)

## <a name="download-the-sample-application"></a>下载示例应用程序
本快速入门中使用的[示例应用程序](https://github.com/Azure-Samples/storage-blobs-php-quickstart.git)是基本的 PHP 应用程序。  

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-php-quickstart.git
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开 PHP 示例应用程序，请查找 storage-blobs-php-quickstart 文件夹，然后打开 phpqs.php 文件。  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串
在应用程序中，必须提供存储帐户名称和帐户密钥，以创建应用程序的 **BlobRestProxy** 实例。 建议将这些标识符存储在运行应用程序的本地计算机的环境变量中。 根据操作系统按照下面的一个示例创建环境变量。 将 youraccountname 和 youraccountkey 值分别替换为帐户名称和密钥。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```bash
export ACCOUNT_NAME=<youraccountname>
export ACCOUNT_KEY=<youraccountkey>
```

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```cmd
setx ACCOUNT_NAME=<youraccountname>
setx ACCOUNT_KEY=<youraccountkey>
```
---

## <a name="configure-your-environment"></a>配置环境
从本地 git 文件夹中获取此文件夹，将其置于 PHP 服务器提供的目录中。 然后，打开作用域为该目录的命令提示符并输入：`php composer.phar install`

## <a name="run-the-sample"></a>运行示例
此示例在“.”文件夹中创建一个测试文件。 示例程序会将该测试文件上传到 Blob 存储，列出容器中的 blob，并使用新名称下载此文件。 

运行示例。 以下输出是运行应用程序时返回的输出的示例：
  
```
Uploading BlockBlob: HelloWorld.txt
These are the blobs present in the container: HelloWorld.txt: https://myexamplesacct.blob.core.windows.net/blockblobsleqvxd/HelloWorld.txt

This is the content of the blob uploaded: Hello Azure!
```
按显示的键时，示例程序会删除存储容器和文件。 继续前，请在服务器的文件夹中查看这两个文件。 可以打开它们，并看到它们完全相同。

还可以使用工具（如 [Azure 存储资源管理器](https://storageexplorer.com)）查看 Blob 存储中的文件。 Azure 存储资源管理器是免费的跨平台工具，可用于访问存储帐户信息。 

验证文件后，按任意键可完成演示并删除测试文件。 了解此示例的用途以后，即可打开 example.rb 文件来查看代码。 

## <a name="understand-the-sample-code"></a>了解示例代码

接下来逐步介绍示例代码，以便展示其工作方式。

### <a name="get-references-to-the-storage-objects"></a>获取对存储对象的引用
首先创建对用于访问和管理 Blob 存储的对象的引用。 这些对象相互关联，并且每个对象被列表中的下一个对象使用。

* 创建 Azure 存储 **BlobRestProxy** 对象的实例，用于设置连接凭据。 
* 创建 **BlobService** 对象，使之指向存储帐户中的 Blob 服务。 
* 创建**容器**对象，让其代表要访问的容器。 容器用于组织 blob，就像使用计算机上的文件夹组织文件一样。

有了云 **blobClient** 容器对象后，可以创建**块** Blob 对象，使之指向你感兴趣的特定 Blob， 然后执行上传、下载、复制等操作。

> [!IMPORTANT]
> 容器名称必须为小写。 有关容器名称和 blob 名称的详细信息，请参阅[命名和引用容器、Blob 和元数据](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

此部分设置 Azure 存储客户端的实例，实例化 Blob 服务对象，创建新的容器，然后设置容器的权限，使 Blob 公开。 容器名称为 quickstartblobs。 

```PHP
    # Setup a specific instance of an Azure::Storage::Client
    $connectionString = "DefaultEndpointsProtocol=https;AccountName=".getenv('account_name').";AccountKey=".getenv('account_key');
    
    // Create blob client.
    $blobClient = BlobRestProxy::createBlobService($connectionString);
    
    # Create the BlobService that represents the Blob service for the storage account
    $createContainerOptions = new CreateContainerOptions();
    
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);
    
    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    $containerName = "blockblobs".generateRandomString();

    try    {
        // Create container.
        $blobClient->createContainer($containerName, $createContainerOptions);
```

### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

Blob 存储支持块 blob、追加 blob 和页 blob。 块 blob 是最常用的 blob，此快速入门中使用的便是它。  

若要将文件上传到 blob，请通过将本地驱动器上的目录名称和文件名称联接在一起来获取文件的完整路径。 然后可以使用 **createBlockBlob()** 方法将文件上传到指定的路径。 

示例代码使用一个本地文件并将其上传到 Azure。 在代码中，该文件存储为 **myfile**，Blob 的名称存储为 **fileToUpload**。 以下示例将文件上传到名为“quickstartblobs”的容器。

```PHP
    $myfile = fopen("HelloWorld.txt", "w") or die("Unable to open file!");
    fclose($myfile);

    # Upload file as a block blob
    echo "Uploading BlockBlob: ".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    
    $content = fopen($fileToUpload, "r");

    //Upload blob
    $blobClient->createBlockBlob($containerName, $fileToUpload, $content);
```

若要对块 Blob 进行部分更新，请使用 **createblocklist()** 方法。 块 blob 最大可以为 4.7 TB，并且可以是从 Excel 电子表格到大视频文件的任何内容。 页 blob 主要用于用于备份 IaaS VM 的 VHD 文件。 追加 blob 用于日志记录，例如有时需要写入到文件，再继续添加更多信息。 追加 Blob 应在单编写器模型中使用。 存储在 Blob 存储中的大多数对象都是块 blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

可以使用 **listBlobs()** 方法获取容器中文件的列表。 下面的代码检索 blob 列表，然后循环访问它们，显示在容器中找到的 blob 的名称。  

```PHP
    $listBlobsOptions = new ListBlobsOptions();
    $listBlobsOptions->setPrefix("HelloWorld");
    
    echo "These are the blobs present in the container: ";
    
    do{
        $result = $blobClient->listBlobs($containerName, $listBlobsOptions);
        foreach ($result->getBlobs() as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
        
        $listBlobsOptions->setContinuationToken($result->getContinuationToken());
    } while($result->getContinuationToken());
```

### <a name="get-the-content-of-your-blobs"></a>获取 Blob 的内容

使用 **getBlob()** 方法获取 Blob 的内容 以下代码显示在前面部分上传的 Blob 的内容。

```PHP
    $blob = $blobClient->getBlob($containerName, $fileToUpload);
    fpassthru($blob->getContentStream());
```

### <a name="clean-up-resources"></a>清理资源
如果不再需要本快速入门中上传的 Blob，可使用 **deleteContainer()** 方法删除整个容器。 如果不再需要已创建的文件，请使用 **deleteBlob()** 方法将文件删除。

```PHP
    // Delete blob.
    echo "Deleting Blob".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    $blobClient->deleteBlob($_GET["containerName"], $fileToUpload);

    // Delete container.
    echo "Deleting Container".PHP_EOL;
    echo $_GET["containerName"].PHP_EOL;
    echo "<br />";
    $blobClient->deleteContainer($_GET["containerName"]);

    //Deleting local file
    echo "Deleting file".PHP_EOL;
    echo "<br />";
    unlink($fileToUpload);   
```

## <a name="resources-for-developing-php-applications-with-blobs"></a>用于开发包含 Blob 的 PHP 应用程序的资源

查看以下附加资源，了解如何使用 Blob 存储进行 PHP 开发：

- 在 GitHub 上查看、下载并安装 Azure 存储的 [PHP 客户端库源代码](https://github.com/Azure/azure-storage-php)。
- 浏览使用 PHP 客户端库编写的 [Blob 存储示例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=php&term=blob)。

## <a name="next-steps"></a>后续步骤
 
本快速入门介绍了如何使用 PHP 在本地磁盘和 Azure Blob 存储之间转移文件。 若要详细了解 PHP 的用法，请转到 PHP 开发人员中心。

> [!div class="nextstepaction"]
> [PHP 开发人员中心](https://azure.microsoft.com/develop/php/)


若要详细了解存储资源管理器和 Blob，请参阅[使用存储资源管理器管理 Azure Blob 存储资源](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
