---
title: 将大量随机数据以并行方式上传到 Azure 存储 | Microsoft Docs
description: 了解如何使用 Azure SDK 将大量随机数据以并行方式上传到 Azure 存储帐户
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.custom: mvc
ms.subservice: blobs
ms.openlocfilehash: 0673d97f755d7e01d42d0be7c611720ff1e4ad01
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65187773"
---
# <a name="upload-large-amounts-of-random-data-in-parallel-to-azure-storage"></a>将大量随机数据以并行方式上传到 Azure 存储

本教程是一个系列中的第二部分。 本教程演示如何部署将大量随机数据上传到 Azure 存储帐户的应用程序。

本系列教程的第二部分将介绍如何：

> [!div class="checklist"]
> * 配置连接字符串
> * 构建应用程序
> * 运行应用程序
> * 验证连接数

Azure Blob 存储提供可缩放的服务来存储数据。 为了尽可能提高应用程序的性能，建议了解 blob 存储的工作方式。 了解 Azure Blob 的限制非常重要，要深入了解这些限制，请访问：[Blob 存储可伸缩性目标](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#azure-blob-storage-scale-targets)。

在使用 Blob 设计高性能应用程序时，[分区命名](../common/storage-performance-checklist.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#subheading47)是另一个重要因素。 Azure 存储使用基于范围的分区方案来进行缩放和负载均衡。 此配置意味着具有相似命名约定或前缀的文件转到相同分区。 此逻辑还包括文件上传到的容器的名称。 本教程中使用名称为 GUID 的文件以及随机生成的内容。 然后将这些文件和内容上传到五个使用随机名称的不同容器。

## <a name="prerequisites"></a>先决条件

若要完成本教程，必须先完成上一存储教程：[为可缩放的应用程序创建虚拟机和存储帐户][previous-tutorial]。

## <a name="remote-into-your-virtual-machine"></a>远程登录到虚拟机

在本地计算机上使用以下命令创建与虚拟机的远程桌面会话。 将 IP 地址替换为虚拟机的 publicIPAddress。 出现提示时，输入创建虚拟机时使用的凭据。

```
mstsc /v:<publicIpAddress>
```

## <a name="configure-the-connection-string"></a>配置连接字符串

在 Azure 门户中，导航到存储帐户。 在存储帐户的“设置”下选择“访问密钥”。 从主密钥或辅助密钥复制“连接字符串”。 登录到上一教程中创建的虚拟机。 以管理员身份打开“命令提示符”，并使用 `/m` 开关运行 `setx` 命令，该命令可保存计算机设置环境变量。 重载“命令提示符”后，环境变量才可用。 替换以下示例中的 \<storageConnectionString\>：

```
setx storageconnectionstring "<storageConnectionString>" /m
```

完成后，打开另一“命令提示符”，导航到 `D:\git\storage-dotnet-perf-scale-app` 并键入 `dotnet build` 以生成应用程序。

## <a name="run-the-application"></a>运行应用程序

导航到 `D:\git\storage-dotnet-perf-scale-app`。

键入 `dotnet run` 运行应用程序。 首次运行 `dotnet` 时，它会填充本地程序包高速缓存，以加快恢复速度并实现脱机访问。 完成此命令需要最多一分钟，并且仅完成一次。

```
dotnet run
```

应用程序创建五个随机命名的容器，并开始将暂存目录中的文件上传到存储帐户。 应用程序将最小线程设置和 [DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit(v=vs.110).aspx) 设置为 100，以确保在运行应用程序时允许大量并发连接。

除设置线程和连接限制设置外，还需将 [UploadFromStreamAsync ](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromstreamasync?view=azure-dotnet) 方法的 [BlobRequestOptions](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions?view=azure-dotnet) 配置为使用并行，并禁用 MD5 哈希验证。 文件以 100 mb 的块上传，此配置提高了性能，但如果网络性能不佳，可能成本高昂，因为如果出现失败，会重试整个 100 mb 的块。

|属性|值|说明|
|---|---|---|
|[ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount?view=azure-dotnet)| 8| 上传时，此设置将 blob 分为多个块。 为获得最佳性能，此值应为内核数的 8 倍。 |
|[DisableContentMD5Validation](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.disablecontentmd5validation?view=azure-dotnet)| true| 该属性禁用对上传内容的 MD5 哈希检查。 禁用 MD5 验证可加快传输速度。 但是不能确认传输文件的有效性或完整性。   |
|[StoreBlobContentMD5](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.storeblobcontentmd5?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_StoreBlobContentMD5)| false| 该属性确定是否计算和存储文件的 MD5 哈希。   |
| [RetryPolicy](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.retrypolicy?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_RetryPolicy)| 2 秒回退，最多重试 10 次 |确定请求的重试策略。 重试连接失败，在此示例中，[ExponentialRetry](/dotnet/api/microsoft.azure.batch.common.exponentialretry?view=azure-dotnet) 策略配置为 2 秒回退，最多可重试 10 次。 当应用程序快要达到 [blob 存储可伸缩性目标](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#azure-blob-storage-scale-targets)时，此设置非常重要。  |

下例显示了 `UploadFilesAsync` 任务：

```csharp
private static async Task UploadFilesAsync()
{
    // Create random 5 characters containers to upload files to.
    CloudBlobContainer[] containers = await GetRandomContainersAsync();
    var currentdir = System.IO.Directory.GetCurrentDirectory();

    // path to the directory to upload
    string uploadPath = currentdir + "\\upload";
    Stopwatch time = Stopwatch.StartNew();
    try
    {
        Console.WriteLine("Iterating in directory: {0}", uploadPath);
        int count = 0;
        int max_outstanding = 100;
        int completed_count = 0;

        // Define the BlobRequestOptions on the upload.
        // This includes defining an exponential retry policy to ensure that failed connections are retried with a backoff policy. As multiple large files are being uploaded
        // large block sizes this can cause an issue if an exponential retry policy is not defined.  Additionally parallel operations are enabled with a thread count of 8
        // This could be should be multiple of the number of cores that the machine has. Lastly MD5 hash validation is disabled for this example, this improves the upload speed.
        BlobRequestOptions options = new BlobRequestOptions
        {
            ParallelOperationThreadCount = 8,
            DisableContentMD5Validation = true,
            StoreBlobContentMD5 = false
        };
        // Create a new instance of the SemaphoreSlim class to define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        List<Task> tasks = new List<Task>();
        Console.WriteLine("Found {0} file(s)", Directory.GetFiles(uploadPath).Count());

        // Iterate through the files
        foreach (string path in Directory.GetFiles(uploadPath))
        {
            // Create random file names and set the block size that is used for the upload.
            var container = containers[count % 5];
            string fileName = Path.GetFileName(path);
            Console.WriteLine("Uploading {0} to container {1}.", path, container.Name);
            CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

            // Set block size to 100MB.
            blockBlob.StreamWriteSizeInBytes = 100 * 1024 * 1024;
            await sem.WaitAsync();

            // Create tasks for each file that is uploaded. This is added to a collection that executes them all asyncronously.  
            tasks.Add(blockBlob.UploadFromFileAsync(path, null, options, null).ContinueWith((t) =>
            {
                sem.Release();
                Interlocked.Increment(ref completed_count);
            }));
            count++;
        }

        // Creates an asynchronous task that completes when all the uploads complete.
        await Task.WhenAll(tasks);

        time.Stop();

        Console.WriteLine("Upload has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());

        Console.ReadLine();
    }
    catch (DirectoryNotFoundException ex)
    {
        Console.WriteLine("Error parsing files in the directory: {0}", ex.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

以下示例是截断的应用程序输出，该应用程序在 Windows 系统上运行。

```
Created container https://mystorageaccount.blob.core.windows.net/9efa7ecb-2b24-49ff-8e5b-1d25e5481076
Created container https://mystorageaccount.blob.core.windows.net/bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b
Created container https://mystorageaccount.blob.core.windows.net/9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f
Created container https://mystorageaccount.blob.core.windows.net/47646f1a-c498-40cd-9dae-840f46072180
Created container https://mystorageaccount.blob.core.windows.net/38b2cdab-45fa-4cf9-94e7-d533837365aa
Iterating in directory: D:\git\storage-dotnet-perf-scale-app\upload
Found 50 file(s)
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\1d596d16-f6de-4c4c-8058-50ebd8141e4d.txt to container 9efa7ecb-2b24-49ff-8e5b-1d25e5481076.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\242ff392-78be-41fb-b9d4-aee8152a6279.txt to container bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\38d4d7e2-acb4-4efc-ba39-f9611d0d55ef.txt to container 9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\45930d63-b0d0-425f-a766-cda27ff00d32.txt to container 47646f1a-c498-40cd-9dae-840f46072180.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\5129b385-5781-43be-8bac-e2fbb7d2bd82.txt to container 38b2cdab-45fa-4cf9-94e7-d533837365aa.
...
Upload has been completed in 142.0429536 seconds. Press any key to continue
```

### <a name="validate-the-connections"></a>验证连接

在上载文件的同时，可以验证存储帐户的并发连接数。 打开“命令提示符”并键入 `netstat -a | find /c "blob:https"`。 此命令显示当前使用 `netstat` 打开的连接数。 下例显示的输出与自己运行该教程时看到的输出类似。 如该示例所示，上传随机文件到存储帐户时，打开了 800 个连接。 此值在整个上传过程中不断更改。 通过以并行块区块的形式进行上传，可显著减少传输内容所需的时间。

```
C:\>netstat -a | find /c "blob:https"
800

C:\>
```

## <a name="next-steps"></a>后续步骤

本系列的第二部分介绍了以并行方式将大量随机数据上传到存储帐户的方法，例如如何：

> [!div class="checklist"]
> * 配置连接字符串
> * 构建应用程序
> * 运行应用程序
> * 验证连接数

进入本系列的第三部分，从存储帐户下载大量数据。

> [!div class="nextstepaction"]
> [从 Azure 存储下载大量随机数据](storage-blob-scalable-app-download-files.md)

[previous-tutorial]: storage-blob-scalable-app-create-vm.md
