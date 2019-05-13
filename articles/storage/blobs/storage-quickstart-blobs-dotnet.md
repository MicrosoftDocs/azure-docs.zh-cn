---
title: 快速入门：使用 .NET 在对象存储中创建 blob - Azure 存储
description: 本快速入门介绍如何使用 .NET 的 Azure 存储客户端库在 Blob（对象）存储中创建容器和 blob。 接下来，将介绍如何将 blob 下载到本地计算机，以及如何在容器中列出所有 blob。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 05/09/2019
ms.author: mhopkins
ms.reviewer: seguler
ms.openlocfilehash: 2728ac93abd18f929dc6e82cfb471ce33fe8866f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508028"
---
# <a name="quickstart-use-net-to-create-a-blob-in-object-storage"></a>快速入门：使用 .NET 在对象存储中创建 blob

本快速入门介绍如何使用 .NET 的 Azure 存储客户端库在 Blob（对象）存储中创建容器和 blob。 接下来，将介绍如何将 blob 下载到本地计算机，以及如何在容器中列出所有 blob。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

接下来，请下载并安装适用于操作系统的 .NET Core 2.1。 如果运行的是 Windows，可以安装 Visual Studio 并根据偏好使用 .NET Framework。 也可选择安装一个可以在操作系统中使用的编辑器。

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

- 安装 [.NET Core for Windows](https://www.microsoft.com/net/download/windows) 或 [.NET Framework](https://www.microsoft.com/net/download/windows)（Visual Studio for Windows 已随附）
- 安装 [Visual Studio for Windows](https://www.visualstudio.com/)。 如果使用的是 .NET Core，则可以根据需要安装 Visual Studio。  

有关在 .NET Core 与 .NET Framework 之间做出选择的信息，请参阅[为服务器应用选择 .NET Core 或 .NET Framework](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server)。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

- 安装[用于 Linux 的 .NET Core](https://www.microsoft.com/net/download/linux)
- （可选）安装 [Visual Studio Code](https://www.visualstudio.com/) 和 [C# 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp&dotnetid=963890049.1518206068)

# <a name="macostabmacos"></a>[macOS](#tab/macos)

- 安装[用于 macOS 的 .NET Core](https://www.microsoft.com/net/download/macos)。
- （可选）安装[用于 Mac 的 Visual Studio](https://www.visualstudio.com/vs/visual-studio-mac/)

---

## <a name="download-the-sample-application"></a>下载示例应用程序

本快速入门中使用的示例应用程序是基本的控制台应用程序。 可以浏览 [GitHub](https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart) 上的示例应用程序。

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。

```bash
git clone https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart.git
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开 Visual Studio 解决方案，请找到 *storage-blobs-dotnet-quickstart* 文件夹并将其打开，然后双击 *storage-blobs-dotnet-quickstart.sln*。

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串

若要运行应用程序，必须为存储帐户提供连接字符串。 此示例应用程序从环境变量中读取连接字符串，并使用它对 Azure 存储请求进行授权。

复制连接字符串以后，请将其写入运行应用程序的本地计算机的新环境变量中。 若要设置环境变量，请打开控制台窗口，并遵照适用于操作系统的说明。 将 `<yourconnectionstring>` 替换为实际的连接字符串：

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

添加环境变量后，可能需要重启任何正在运行的、需要读取环境变量的程序（包括控制台窗口）。 例如，如果使用 Visual Studio 作为编辑器，请在运行示例之前重启 Visual Studio。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```bash
export storageconnectionstring=<yourconnectionstring>
```

添加环境变量后，请从控制台窗口运行 `source ~/.bashrc`，使更改生效。

# <a name="macostabmacos"></a>[macOS](#tab/macos)

编辑 .bash_profile，然后添加环境变量：

```bash
export storageconnectionstring=<yourconnectionstring>
```

添加环境变量后，请从控制台窗口运行 `source .bash_profile`，使更改生效。

---

## <a name="run-the-sample"></a>运行示例

此示例在本地 **MyDocuments** 文件夹中创建一个测试文件，然后将其上传到 Blob 存储。 此示例然后会列出容器中的 Blob，并使用新名称下载文件，以便对旧文件和新文件进行比较。

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

如果使用 Visual Studio 作为编辑器，可以按 **F5** 运行应用程序。

否则，请导航到应用程序目录，并使用 `dotnet run` 命令运行应用程序。

```console
dotnet run
```

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

导航到应用程序目录，使用 `dotnet run` 命令运行应用程序。

```console
dotnet run
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

导航到应用程序目录，使用 `dotnet run` 命令运行应用程序。

```console
dotnet run
```

---

示例应用程序的输出类似于以下示例：

```output
Azure Blob storage - .NET Quickstart sample

Created container 'quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f'

Temp file = C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt
Uploading to Blob storage as blob 'QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt'

Listing blobs in container.
https://storagesamples.blob.core.windows.net/quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f/QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt

Downloading blob to C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db_DOWNLOADED.txt

Press any key to delete the sample files and example container.
```

按 **Enter** 键时，应用程序会删除存储容器和文件。 在删除之前，请在 **MyDocuments** 文件夹中查看这两个文件。 可以打开它们，然后就会观察到它们完全相同。 从控制台窗口复制 Blob 的 URL，将其粘贴到浏览器中，查看 Blob 的内容。

验证文件后，按任意键可完成演示并删除测试文件。 既然现在已了解此示例的用途，打开 Program.cs 文件可查看代码。

## <a name="understand-the-sample-code"></a>了解示例代码

接下来介绍示例代码，探讨其工作方式。

### <a name="try-parsing-the-connection-string"></a>尝试分析连接字符串

此示例所执行的第一项操作是检查环境变量是否包含一个连接字符串，该字符串在经过分析后可以创建一个指向存储帐户的 [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) 对象。 若要检查连接字符串是否有效，请使用 [TryParse](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse) 方法。 如果 **TryParse** 成功，它会初始化 *storageAccount* 变量并返回 **true**。

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable on the machine running the application called storageconnectionstring.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell or application needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
CloudStorageAccount storageAccount;
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with operations against Blob storage here.
    ...
}
else
{
    // Otherwise, let the user know that they need to define the environment variable.
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
    Console.WriteLine("Press any key to exit the sample application.");
    Console.ReadLine();
}
```

### <a name="create-the-container-and-set-permissions"></a>创建容器并设置权限

接下来，此示例创建一个容器并设置其权限，使容器中的任何 Blob 都是公开的。 如果某个 Blob 是公开的，则任何客户端都可以对其进行匿名访问。

若要创建容器，请先创建 [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient) 对象的实例，该对象指向存储帐户中的 Blob 存储。 接下来，请创建 [CloudBlobContainer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer) 对象的实例，然后创建容器。

在此示例中，示例调用 [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync) 方法来创建容器。 GUID 值会追加到容器名称，确保其是唯一的。 在生产环境中，通常情况下，首选使用 [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync) 方法来创建容器的前提是该方法不存在，以免出现名称冲突。

> [!IMPORTANT]
> 容器名称必须为小写。 有关命名容器和 Blob 的详细信息，请参阅[命名和引用容器、Blob 和元数据](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

```csharp
// Create the CloudBlobClient that represents the Blob storage endpoint for the storage account.
CloudBlobClient cloudBlobClient = storageAccount.CreateCloudBlobClient();

// Create a container called 'quickstartblobs' and append a GUID value to it to make the name unique.
CloudBlobContainer cloudBlobContainer = cloudBlobClient.GetContainerReference("quickstartblobs" + Guid.NewGuid().ToString());
await cloudBlobContainer.CreateAsync();

// Set the permissions so the blobs are public.
BlobContainerPermissions permissions = new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
};
await cloudBlobContainer.SetPermissionsAsync(permissions);
```

### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

接下来，示例将本地文件上传到块 Blob。 代码示例在需要获取 **CloudBlockBlob** 对象的引用时，会针对前面部分创建的容器调用 [GetBlockBlobReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.getblockblobreference) 方法。 然后，它会通过调用 [UploadFromFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromfileasync) 方法将所选文件上传到 Blob。 此方法将创建 Blob（如果该 Blob 尚不存在），或者覆盖 Blob（如果该 Blob 已存在）。

```csharp
// Create a file in your local MyDocuments folder to upload to a blob.
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "QuickStart_" + Guid.NewGuid().ToString() + ".txt";
sourceFile = Path.Combine(localPath, localFileName);
// Write text to the file.
File.WriteAllText(sourceFile, "Hello, World!");

Console.WriteLine("Temp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

// Get a reference to the blob address, then upload the file to the blob.
// Use the value of localFileName for the blob name.
CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
```

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

示例使用 [ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync) 方法列出容器中的 Blob。 就此示例来说，只向容器添加了一个 Blob，因此列表操作只返回那个 Blob。

如果一次调用需要返回的 Blob 数过多（默认为超过 5000 个），则 **ListBlobsSegmentedAsync** 方法会返回总结果集的一部分和一个继续令牌。 若要检索下一部分的 Blob，请提供前一调用返回的继续令牌，依此类推，直至继续令牌为 null。 继续令牌为 null 表明已检索所有 Blob。 示例代码演示了如何根据最佳做法使用继续令牌。

```csharp
// List the blobs in the container.
Console.WriteLine("List blobs in container.");
BlobContinuationToken blobContinuationToken = null;
do
{
    var results = await cloudBlobContainer.ListBlobsSegmentedAsync(null, blobContinuationToken);
    // Get the value of the continuation token returned by the listing call.
    blobContinuationToken = results.ContinuationToken;
    foreach (IListBlobItem item in results.Results)
    {
        Console.WriteLine(item.Uri);
    }
} while (blobContinuationToken != null); // Loop while the continuation token is not null.

```

### <a name="download-blobs"></a>下载 Blob

接下来，示例使用 [DownloadToFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync) 方法将以前创建的 Blob 下载到本地文件系统。 示例代码向 Blob 名称添加后缀“_DOWNLOADED”，这样你就可以在本地文件系统中看到这两个文件。

```csharp
// Download the blob to a local file, using the reference created earlier.
// Append the string "_DOWNLOADED" before the .txt extension so that you can see both files in MyDocuments.
destinationFile = sourceFile.Replace(".txt", "_DOWNLOADED.txt");
Console.WriteLine("Downloading blob to {0}", destinationFile);
await cloudBlockBlob.DownloadToFileAsync(destinationFile, FileMode.Create);  
```

### <a name="clean-up-resources"></a>清理资源

示例可以使用 [CloudBlobContainer.DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync) 来删除整个容器，从而清除所创建的资源。 也可根据需要删除本地文件。

```csharp
Console.WriteLine("Press the 'Enter' key to delete the sample files, example container, and exit the application.");
Console.ReadLine();
// Clean up resources. This includes the container and the two temp files.
Console.WriteLine("Deleting the container");
if (cloudBlobContainer != null)
{
    await cloudBlobContainer.DeleteIfExistsAsync();
}
Console.WriteLine("Deleting the source, and downloaded files");
File.Delete(sourceFile);
File.Delete(destinationFile);
```

## <a name="resources-for-developing-net-applications-with-blobs"></a>用于通过 Blob 开发 .NET 应用程序的资源

请查看以下其他资源，了解如何通过 Blob 存储进行 .NET 开发：

### <a name="binaries-and-source-code"></a>二进制文件和源代码

- 下载 Azure Blob 存储的最新版 [.NET 客户端库](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/)的 NuGet 包。
- 查看 GitHub 上的[适用于 .NET 的 Microsoft Azure 存储 Blob SDK 源代码](https://github.com/Azure/azure-storage-net/tree/master/Blob)。

### <a name="client-library-reference-and-samples"></a>客户端库参考和示例

- 参阅 [.NET API 参考](https://docs.microsoft.com/dotnet/api/overview/azure/storage)，详细了解 .NET 客户端库。
- 浏览使用 .NET 客户端库编写的 [Blob 存储示例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=blob)。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何使用 .NET 上传、下载和列出 Blob。

若要了解如何创建一个 Web 应用，以便将映像上传到 Blob 存储，请继续学习：

> [!div class="nextstepaction"]
> [上传和处理图像](storage-upload-process-images.md)

- 若要详细了解 .NET Core，请参阅 [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/)（.NET 10 分钟入门）。
- 若要了解可以通过用于 Windows 的 Visual Studio 部署的示例应用程序，请参阅 [.NET Photo Gallery Web Application Sample with Azure Blob Storage](https://azure.microsoft.com/resources/samples/storage-blobs-dotnet-webapp/)（.NET 照片库 Web 应用程序示例与 Azure Blob 存储）。
