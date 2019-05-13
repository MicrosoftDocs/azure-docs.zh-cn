---
title: 如何使用适用于 Node.js v2 的客户端库在 Azure 存储中创建 Blob
description: 在对象 (Blob) 存储中创建存储帐户和容器。 随后，使用适用于 Node.js v2 的 Azure 存储客户端库将一个 Blob 上传到 Azure 存储，下载一个 Blob，然后列出容器中的 Blob。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: conceptual
ms.date: 02/04/2019
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: 7ca1eacd1add7453833b4b9ad1d4bd0fbef13ca7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149421"
---
# <a name="how-to-upload-download-and-list-blobs-using-the-client-library-for-nodejs-v2"></a>如何使用适用于 Node.js v2 的客户端库上传、下载和列出 Blob

本操作指南介绍如何使用适用于 Node.js v2 的客户端库通过 Azure Blob 存储来上传、下载和列出 Blob。

> [!TIP]
> 适用于 Node.js 的 Azure 存储客户端库的最新版本是 v10。 Microsoft 建议尽可能使用最新版本的客户端库。 若要开始使用 v10，请参阅[快速入门：使用适用于 JavaScript v10 的 Azure 存储客户端库上传、下载、列出和删除 Blob（预览版）](storage-quickstart-blobs-nodejs-v10.md)。

## <a name="prerequisites"></a>必备组件

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

在 [Azure 门户](https://portal.azure.com/#create/Microsoft.StorageAccount-ARM)中创建 Azure 存储帐户。 有关如何创建帐户的帮助，请参阅[创建存储帐户](../common/storage-quickstart-create-account.md)。

## <a name="download-the-sample-application"></a>下载示例应用程序

[示例应用程序](https://github.com/Azure-Samples/storage-blobs-node-quickstart.git)是简单的 Node.js 控制台应用程序。 若要开始，请使用以下命令将存储库克隆到计算机：

```bash
git clone https://github.com/Azure-Samples/storage-blobs-node-quickstart.git
```

若要打开应用程序，请找到 *storage-blobs-node-quickstart* 文件夹，然后在最常用的代码编辑环境中将其打开。

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串

在运行应用程序之前，必须为存储帐户提供连接字符串。 示例存储库包括名为 *.env.example* 的文件。 可以通过删除 *.example* 扩展来重命名此文件，这样会生成名为 *.env* 的文件。 在 *.env* 文件中，请将连接字符串值添加到 *AZURE_STORAGE_CONNECTION_STRING* 键之后。

## <a name="install-required-packages"></a>安装所需程序包

在应用程序目录中运行 *npm install*，以安装应用程序所需的包。

```bash
npm install
```

## <a name="run-the-sample"></a>运行示例
安装依赖项以后，即可发出以下命令，以便运行示例：

```bash
npm start
```

此脚本的输出将类似于以下内容：

```bash
Containers:
 - container-one
 - container-two
Container "demo" is created
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme.md
Blob downloaded blob content: "hello Blob SDK"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```

如果为此示例使用新的存储帐户，则可能无法看到“*Containers*”标签下列出的任何容器名称。

## <a name="understanding-the-code"></a>了解数据
第一个表达式用于将值加载到环境变量中。

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').load();
}
```

在本地运行应用以进行调试时，*dotenv* 模块会加载环境变量。 值在名为 *.env* 的文件中定义，并加载到当前执行上下文中。 在生产上下文中，服务器配置会提供这些值，这就是为什么此代码仅当脚本不在“生产”上下文下运行时才运行的原因。

```javascript
const path = require('path');
const storage = require('azure-storage');
```

模块的用途如下： 

将名为 *.env* 的文件加载到当前执行上下文中
- 若要确定要上传到 Blob 存储的文件的绝对路径，*path* 是必需的
- *azure-storage* 是适用于 Node.js 的 [Azure 存储客户端库](https://docs.microsoft.com/javascript/api/azure-storage)模块

接下来，将 **blobService** 变量初始化为 Azure Blob 服务的新实例。

```javascript
const blobService = storage.createBlobService();
```

在以下实现中，每个 *blobService* 函数都包装在 *Promise* 中，因此可以访问 JavaScript 的 *async* 函数和 *await* 运算符，以便简化 [Azure 存储 API](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest) 的回调性质。 针对每个函数返回成功的响应后，promise 会使用相关的数据和特定于操作的消息进行解析。

### <a name="list-containers"></a>列出容器

*listContainers* 函数调用 [listContainersSegmented](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest)，后者返回组中容器的集合。

```javascript
const listContainers = async () => {
    return new Promise((resolve, reject) => {
        blobService.listContainersSegmented(null, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `${data.entries.length} containers`, containers: data.entries });
            }
        });
    });
};
```

组的大小可以通过 [ListContainersOptions](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice.listcontaineroptions?view=azure-node-latest) 进行配置。 调用 *listContainersSegmented* 会返回 [ContainerResult](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice.containerresult?view=azure-node-latest) 实例数组形式的 Blob 元数据。 结果以 5,000 增量批次（段）的方式返回。 如果容器中有 5,000 个以上的 Blob，则结果中会包含一个 *continuationToken* 值。 若要列出 Blob 容器中的后续段，可以将继续标记作为第二个参数传递回 *listContainersSegment* 中。

### <a name="create-a-container"></a>创建容器

*createContainer* 函数调用 [createContainerIfNotExists](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest) 并为 Blob 设置相应的访问级别。

```javascript
const createContainer = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.createContainerIfNotExists(containerName, { publicAccessLevel: 'blob' }, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Container '${containerName}' created` });
            }
        });
    });
};
```

**createContainerIfNotExists** 的第二个参数（选项）接受一个值作为 [publicAccessLevel](https://docs.microsoft.com/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest)。 如果 *publicAccessLevel* 的值为 *blob*，则表示会公开特定的 Blob 数据。 此设置不同于容器级别的访问权限，后者授予列出容器内容的权限。

使用 **createContainerIfNotExists** 时，应用程序可以多次运行 *createContainer* 命令，在容器存在的情况下也不会返回错误。 在生产环境中，通常只调用 **createContainerIfNotExists** 一次，因为在应用程序中，从头至尾使用的是同一容器。 在这种情况下，可以通过门户或 Azure CLI 提前创建容器。

### <a name="upload-text"></a>上传文本

*uploadString* 函数调用 [createBlockBlobFromText](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest)，以便将任意字符串写入（或覆盖）到 Blob 容器。

```javascript
const uploadString = async (containerName, blobName, text) => {
    return new Promise((resolve, reject) => {
        blobService.createBlockBlobFromText(containerName, blobName, text, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Text "${text}" is written to blob storage` });
            }
        });
    });
};
```
### <a name="upload-a-local-file"></a>上传本地文件

*uploadLocalFile* 函数通过 [createBlockBlobFromLocalFile](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#createblockblobfromlocalfile-string--string--string--errororresult-blobresult--) 将文件从文件系统上传和写入（或覆盖）到 Blob 存储中。 

```javascript
const uploadLocalFile = async (containerName, filePath) => {
    return new Promise((resolve, reject) => {
        const fullPath = path.resolve(filePath);
        const blobName = path.basename(filePath);
        blobService.createBlockBlobFromLocalFile(containerName, blobName, fullPath, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Local file "${filePath}" is uploaded` });
            }
        });
    });
};
```
其他适用于将内容上传到 Blob 中的方法涉及到使用[文本](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest-string--string--string---buffer--errororresult-blobresult--)和[流](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#createblockblobfromstream-string--string--stream-readable--number--errororresult-blobresult--)。 为了验证文件是否已上传到 Blob 存储，可以使用 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)查看帐户中的数据。

### <a name="list-the-blobs"></a>列出 Blob

*listBlobs* 函数在调用 [listBlobsSegmented](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#listblobssegmented-string--continuationtoken--errororresult-listblobsresult--) 方法后会返回一个列表，其中包含容器中的 Blob 元数据。 

```javascript
const listBlobs = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.listBlobsSegmented(containerName, null, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `${data.entries.length} blobs in '${containerName}'`, blobs: data.entries });
            }
        });
    });
};
```

调用 *listBlobsSegmented* 会返回 [BlobResult](https://docs.microsoft.com/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice.blobresult?view=azure-node-latest) 实例数组形式的 Blob 元数据。 结果以 5,000 增量批次（段）的方式返回。 如果容器中有 5,000 个以上的 Blob，则结果中会包含一个 **continuationToken** 值。 若要列出 Blob 容器中的后续段，可以将继续标记作为第二个参数传递回 **listBlobSegmented** 中。

### <a name="download-a-blob"></a>下载 Blob

*downloadBlob* 函数使用 [getBlobToText](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest) 将 Blob 的内容下载到给定的绝对文件路径。

```javascript
const downloadBlob = async (containerName, blobName) => {
    const dowloadFilePath = path.resolve('./' + blobName.replace('.txt', '.downloaded.txt'));
    return new Promise((resolve, reject) => {
        blobService.getBlobToText(containerName, blobName, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Blob downloaded "${data}"`, text: data });
            }
        });
    });
};
```
此处显示的实现更改了源并将 Blob 的内容作为字符串返回。 也可将 Blob 作为[流](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest)来下载，或者直接下载到[本地文件](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest)。

### <a name="delete-a-blob"></a>删除 Blob

*deleteBlob* 函数调用 [deleteBlobIfExists](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#deleteblobifexists-string--string--errororresult-boolean--) 函数。 顾名思义，如果 Blob 已删除，此函数不返回错误。

```javascript
const deleteBlob = async (containerName, blobName) => {
    return new Promise((resolve, reject) => {
        blobService.deleteBlobIfExists(containerName, blobName, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Block blob '${blobName}' deleted` });
            }
        });
    });
};
```

### <a name="delete-a-container"></a>删除容器

可以通过对 Blob 服务调用 *deleteContainer* 方法并传入容器名称来删除容器。

```javascript
const deleteContainer = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.deleteContainer(containerName, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Container '${containerName}' deleted` });
            }
        });
    });
};
```

### <a name="calling-code"></a>调用代码

为了支持 JavaScript 的 *async/await* 语法，所有调用代码都包装在一个名为 *execute* 的函数中。 然后按约定调用并处理 execute。

```javascript
async function execute() {
    // commands 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```
以下所有代码都在 execute 函数内 `// commands` 注释所在的位置处运行。

首先，声明相关变量以分配名称、示例内容并指向要上传到 Blob 存储的本地文件。

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello Node SDK";
const localFilePath = "./readme.md";
let response;
```

若要列出存储帐户中的容器，请调用 listContainers 函数，然后返回的容器列表会记录到输出窗口。

```javascript
console.log("Containers:");
response = await listContainers();
response.containers.forEach((container) => console.log(` -  ${container.name}`));
```

容器列表可用以后，即可使用数组的 *findIndex* 方法来查看要创建的容器是否已存在。 如果该容器不存在，则会创建它。

```javascript
const containerDoesNotExist = response.containers.findIndex((container) => container.name === containerName) === -1;

if (containerDoesNotExist) {
    await createContainer(containerName);
    console.log(`Container "${containerName}" is created`);
}
```
接下来，将字符串和本地文件上传到 Blob 存储。

```javascript
await uploadString(containerName, blobName, content);
console.log(`Blob "${blobName}" is uploaded`);

response = await uploadLocalFile(containerName, localFilePath);
console.log(response.message);
```
列出 Blob 的过程与列出容器的过程相同。 调用 *listBlobs* 会返回容器中 Blob 的数组，这些 Blob 会记录到输出窗口中。

```javascript
console.log(`Blobs in "${containerName}" container:`);
response = await listBlobs(containerName);
response.blobs.forEach((blob) => console.log(` - ${blob.name}`));
```

若要下载某个 Blob，请捕获响应并用其来访问 Blob 的值。 在响应中，readableStreamBody 会转换为字符串并记录到输出窗口。

```javascript
response = await downloadBlob(containerName, blobName);
console.log(`Downloaded blob content: "${response.text}"`);
```

最终，Blob 和容器会从存储帐户中删除。

```javascript
await deleteBlob(containerName, blobName);
console.log(`Blob "${blobName}" is deleted`);

await deleteContainer(containerName);
console.log(`Container "${containerName}" is deleted`);
```

## <a name="clean-up-resources"></a>清理资源
写入存储帐户的所有数据都会在代码示例结束时自动删除。 

## <a name="resources-for-developing-nodejs-applications-with-blobs"></a>用于通过 Blob 开发 Node.js 应用程序的资源

请查看以下其他资源，了解如何通过 Blob 存储进行 Node.js 开发：

### <a name="binaries-and-source-code"></a>二进制文件和源代码

- 在 GitHub 上查看并安装 Azure 存储的 [Node.js 客户端库源代码](https://github.com/Azure/azure-storage-node)。

### <a name="client-library-reference-and-samples"></a>客户端库参考和示例

- 参阅 [Node.js API 参考](https://docs.microsoft.com/javascript/api/overview/azure/storage)，详细了解 Node.js 客户端库。
- 浏览使用 Node.js 客户端库编写的 [Blob 存储示例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=nodejs&term=blob)。

## <a name="next-steps"></a>后续步骤

本文介绍了如何使用 Node.js 在本地磁盘和 Azure Blob 存储之间上传文件。 若要深入了解如何使用 Blob 存储，请转到 GitHub 存储库。

> [!div class="nextstepaction"]
> [Microsoft Azure Storage SDK for Node.js and JavaScript for Browsers](https://github.com/Azure/azure-storage-node)（Microsoft Azure Storage SDK for Node.js 和 JavaScript for Browsers）
