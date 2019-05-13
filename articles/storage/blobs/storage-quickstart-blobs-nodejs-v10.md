---
title: 使用 Azure Storage v10 SDK for JavaScript 上载、下载、列出和删除 blob
description: 使用 Azure 存储在 Node.js 中创建、上载和删除 blob 与容器
services: storage
author: mhopkins-msft
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 11/14/2018
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: f426ee10017533c21021d618d613dc0931767988
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149431"
---
# <a name="quickstart-upload-download-list-and-delete-blobs-using-azure-storage-v10-sdk-for-javascript"></a>快速入门：使用 Azure Storage v10 SDK for JavaScript 上载、下载、列出和删除 blob

本快速入门介绍如何在 Node.js 中使用 [Azure Storage v10 SDK for JavaScript](https://github.com/Azure/azure-storage-js) 来上载、下载、列出和删除 blob 以及管理容器。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

## <a name="download-the-sample-application"></a>下载示例应用程序

本快速入门中的[示例应用程序](https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git)是简单的 Node.js 控制台应用程序。 若要开始，请使用以下命令将存储库克隆到计算机：

```bash
git clone https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git
```

接下来，更改应用程序的文件夹：

```bash
cd azure-storage-js-v10-quickstart
```

现在，在你最喜欢的代码编辑环境中打开该文件夹。

## <a name="configure-your-storage-credentials"></a>配置存储凭据

在运行应用程序之前，必须为存储帐户提供安全凭据。 示例存储库包括名为 *.env.example* 的文件。 通过删除 *.example* 扩展名来重命名此文件，这样会生成名为 *.env* 的文件。 在 *.env* 文件中，在 *AZURE_STORAGE_ACCOUNT_NAME* 键和 *AZURE_STORAGE_ACCOUNT_ACCESS_KEY* 键的后面添加帐户名称和访问密钥值。

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

应用的输出将类似于以下示例：

```bash
Containers:
 - container-one
 - container-two
Container "demo" is created
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme-stream.md
 - readme.md
Blob downloaded blob content: "hello!"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```
如果为此快速入门使用新的存储帐户，则可能无法看到“*Containers*”标签下列出的容器名称。

## <a name="understanding-the-code"></a>了解数据
该示例首先从 Azure Blob 存储命名空间中导入多个类和函数。 导入的每个项目都将在上下文中进行讨论，因为该示例将使用这些项目。

```javascript
const {
    Aborter,
    BlobURL,
    BlockBlobURL,
    ContainerURL,
    ServiceURL,
    SharedKeyCredential,
    StorageURL,
    uploadStreamToBlockBlob
} = require('@azure/storage-blob');
```

根据适当的上下文从环境变量中读取凭据。

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').load();
}
```

在本地运行应用以进行调试时，*dotenv* 模块会加载环境变量。 值在名为 *.env* 的文件中定义，并加载到当前执行上下文中。 在生产环境中，服务器配置会提供这些值，这就是为什么此代码仅当脚本*不*在“生产”环境下运行时运行的原因。

导入下一批模块，以帮助与文件系统对接。

```javascript
const fs = require('fs');
const path = require('path');
```

这些模块的用途如下所示： 

- *fs* 是用于处理文件系统的本机 Node.js 模块

- *path* 是确定文件绝对路径所必需的，在将文件上载到 Blob 存储时使用

接下来，读取环境变量值并将其放在常量中。

```javascript
const STORAGE_ACCOUNT_NAME = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const ACCOUNT_ACCESS_KEY = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
```
下一组常量有助于揭示上载操作期间文件大小计算的意图。
```javascript
const ONE_MEGABYTE = 1024 * 1024;
const FOUR_MEGABYTES = 4 * ONE_MEGABYTE;
```
该 API 发出的请求可以设置为在给定时间间隔后超时。 [Aborter](/javascript/api/%40azure/storage-blob/aborter?view=azure-node-preview) 类负责管理请求超时的方式，而以下常量用于定义此示例中所使用的超时。
```javascript
const ONE_MINUTE = 60 * 1000;
```
### <a name="calling-code"></a>调用代码

为了支持 JavaScript 的 *async/await* 语法，所有调用代码都包装在一个名为 *execute* 的函数中。 然后按约定调用和处理 *execute*。

```javascript
async function execute() {
    // commands... 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```
以下所有代码都在 execute 函数内 `// commands...` 注释所在的位置处运行。

首先，声明相关变量以分配名称、示例内容并指向要上载到 Blob 存储的本地文件。

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello!";
const localFilePath = "./readme.md";
```

帐户凭据用于创建管道，管道负责管理将请求发送到 REST API 的方式。 管道是线程安全的，并指定重试策略、日志记录、HTTP 响应反序列化规则等项目的逻辑。

```javascript
const credentials = new SharedKeyCredential(STORAGE_ACCOUNT_NAME, ACCOUNT_ACCESS_KEY);
const pipeline = StorageURL.newPipeline(credentials);
const serviceURL = new ServiceURL(`https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net`, pipeline);
```
此代码块中使用以下类：

- [SharedKeyCredential](/javascript/api/%40azure/storage-blob/sharedkeycredential?view=azure-node-preview) 类负责包装存储帐户凭据，以将其提供给请求管道。

- [StorageURL](/javascript/api/%40azure/storage-blob/storageurl?view=azure-node-preview) 类负责创建新管道。

- [ServiceURL](/javascript/api/%40azure/storage-blob/serviceurl?view=azure-node-preview) 对 REST API 中使用的 URL 建模。 此类的实例允许执行列出容器等操作，并提供上下文信息以生成容器 URL。

*ServiceURL* 的实例与 [ContainerURL](/javascript/api/%40azure/storage-blob/containerurl?view=azure-node-preview) 和 [BlockBlobURL](/javascript/api/%40azure/storage-blob/blockbloburl?view=azure-node-preview) 实例一起用于管理存储帐户中的容器和 blob。

```javascript
const containerURL = ContainerURL.fromServiceURL(serviceURL, containerName);
const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, blobName);
```
*containerURL* 和 *blockBlobURL* 变量在整个示例中重复使用，以便对存储帐户执行操作。 

此时，存储帐户中不存在容器。 *ContainerURL* 的实例表示可以对其执行操作的 URL。 通过使用此实例，可以创建和删除容器。 此容器的位置等同于以下位置：

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo
```

*blockBlobURL* 用于管理各个 blob，允许上载、下载和删除 blob 内容。 此处的 URL 与此位置类似：

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo/quickstart.txt
```
与容器一样，块 blob 尚不存在。 稍后将使用 *blockBlobURL* 变量通过上载内容来创建 blob。

### <a name="using-the-aborter-class"></a>使用 Aborter 类

该 API 发出的请求可以设置为在给定时间间隔后超时。 *Aborter* 类负责管理请求超时的方式。以下代码会创建一个上下文，其中一组请求指定为在 30 分钟内执行。

```javascript
const aborter = Aborter.timeout(30 * ONE_MINUTE);
```
Aborter 允许通过以下方式控制请求：

- 指定为一批请求给定的时间量
- 指定单个请求必须在批处理中执行多长时间
- 允许取消请求
- 使用 *Aborter.none* 静态成员来阻止所有请求一起超时

### <a name="show-container-names"></a>显示容器名称
帐户可以存储大量容器。 以下代码演示如何以分段方式列出容器，以便你循环遍历大量容器。 *showContainerNames* 函数是 *ServiceURL* 和 *Aborter* 的传递实例。

```javascript
console.log("Containers:");
await showContainerNames(serviceURL, aborter);
```
*showContainerNames* 函数使用 *listContainersSegment* 方法从存储帐户中请求多批容器名称。
```javascript
async function showContainerNames(aborter, serviceURL) {

    let response;
    let marker;

    do {
        response = await serviceURL.listContainersSegment(aborter, marker);
        marker = response.marker;
        for(let container of response.containerItems) {
            console.log(` - ${ container.name }`);
        }
    } while (marker);
}
```
响应返回时会迭代 *containerItems*，以将名称记录到控制台。 

### <a name="create-a-container"></a>创建容器

若要创建容器，请使用 *ContainerURL* 的 *create* 方法。

```javascript
await containerURL.create(aborter);
console.log(`Container: "${containerName}" is created`);
```
由于在调用 *ContainerURL.fromServiceURL(serviceURL, containerName)* 时定义了容器的名称，因此只需调用 *create* 方法即可创建容器。

### <a name="upload-text"></a>上载文本
若要向 blob 上载文本，请使用 *upload* 方法。
```javascript
await blockBlobURL.upload(aborter, content, content.length);
console.log(`Blob "${blobName}" is uploaded`);
```
这里将文本及其长度传递给该方法。
### <a name="upload-a-local-file"></a>上载本地文件
若要向容器上载本地文件，则需要容器 URL 和该文件的路径。
```javascript
await uploadLocalFile(aborter, containerURL, localFilePath);
console.log(`Local file "${localFilePath}" is uploaded`);
```
*uploadLocalFile* 函数调用 *uploadFileToBlockBlob* 函数，以将文件路径和块 blob 的目标实例作为参数。
```javascript
async function uploadLocalFile(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath);
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    return await uploadFileToBlockBlob(aborter, filePath, blockBlobURL);
}
```
### <a name="upload-a-stream"></a>上载流
还支持上载流。 此示例将本地文件作为流打开，以传递给 upload 方法。
```javascript
await uploadStream(containerURL, localFilePath, aborter);
console.log(`Local file "${localFilePath}" is uploaded as a stream`);
```
*uploadStream* 函数调用 *uploadStreamToBlockBlob*，以将流上载到存储容器。
```javascript
async function uploadStream(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath).replace('.md', '-stream.md');
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    const stream = fs.createReadStream(filePath, {
      highWaterMark: FOUR_MEGABYTES,
    });

    const uploadOptions = {
        bufferSize: FOUR_MEGABYTES,
        maxBuffers: 5,
    };

    return await uploadStreamToBlockBlob(
                    aborter, 
                    stream, 
                    blockBlobURL, 
                    uploadOptions.bufferSize, 
                    uploadOptions.maxBuffers);
}
```
在上载期间，*uploadStreamToBlockBlob* 分配缓冲区以缓存来自流的数据，以防需要重试。 *maxBuffers* 值指定最多使用多少个缓冲区，因为每个缓冲区都会创建单独的上载请求。 理想情况下，缓冲区越多意味着速度越快，但代价是内存使用率越高。 当缓冲区数量多到使瓶颈转移到网络或磁盘而非客户端时，上载速度达到平衡。

### <a name="show-blob-names"></a>显示 blob 名称
正如帐户可以包含许多容器一样，每个容器都可能包含大量的 blob。 可以通过 *ContainerURL* 类的实例访问容器中的每个 blob。 
```javascript
console.log(`Blobs in "${containerName}" container:`);
await showBlobNames(aborter, containerURL);
```
函数 *showBlobNames* 调用 *listBlobFlatSegment*，以从容器中请求多批 blob。
```javascript
async function showBlobNames(aborter, containerURL) {

    let response;
    let marker;

    do {
        response = await containerURL.listBlobFlatSegment(aborter);
        marker = response.marker;
        for(let blob of response.segment.blobItems) {
            console.log(` - ${ blob.name }`);
        }
    } while (marker);
}
```
### <a name="download-a-blob"></a>下载 Blob
创建 blob 后，即可使用 *download* 方法下载内容。
```javascript
const downloadResponse = await blockBlobURL.download(aborter, 0);
const downloadedContent = downloadResponse.readableStreamBody.read(content.length).toString();
console.log(`Downloaded blob content: "${downloadedContent}"`);
```
响应以流的形式返回。 在此示例中，流转换为字符串以记录到控制台。
### <a name="delete-a-blob"></a>删除 Blob
*BlockBlobURL* 实例中的 *delete* 方法可从容器中删除 blob。
```javascript
await blockBlobURL.delete(aborter)
console.log(`Block blob "${blobName}" is deleted`);
```
### <a name="delete-a-container"></a>删除容器
*ContainerURL* 实例中的 *delete* 方法可从存储帐户中删除容器。
```javascript
await containerURL.delete(aborter);
console.log(`Container "${containerName}" is deleted`);
```
## <a name="clean-up-resources"></a>清理资源
写入存储帐户的所有数据都会在代码示例结束时自动删除。 

## <a name="next-steps"></a>后续步骤

本快速入门演示如何使用 Node.js 管理 Azure Blob 存储中的 blob 和容器。 若要详细了解如何使用此 SDK，请参阅 GitHub 存储库。

> [!div class="nextstepaction"]
> [Azure Storage v10 SDK for JavaScript 存储库](https://github.com/Azure/azure-storage-js)
> [JavaScript API 参考](https://docs.microsoft.com/javascript/api/overview/azure/storage/client?view=azure-node-preview)
