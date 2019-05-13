---
title: 运行并行工作负荷 - Azure Batch .NET
description: 教程 - 在 Azure Batch 中使用 Batch .NET 客户端库通过 ffmpeg 并行转码媒体文件
services: batch
author: laurenhughes
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/21/2018
ms.author: lahugh
ms.custom: mvc
ms.openlocfilehash: c55839f7ec4e4c28086912c476a193e31561c3f4
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234131"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-net-api"></a>教程：使用 .NET API 通过 Azure Batch 运行并行工作负荷

使用 Azure Batch 在 Azure 中高效运行大规模并行和高性能计算 (HPC) 批处理作业。 本教程通过一个 C# 示例演示了如何使用 Batch 运行并行工作负荷。 你可以学习常用的 Batch 应用程序工作流，以及如何以编程方式与 Batch 和存储资源交互。 学习如何：

> [!div class="checklist"]
> * 将应用程序包添加到 Batch 帐户
> * 通过 Batch 和存储帐户进行身份验证
> * 将输入文件上传到存储
> * 创建运行应用程序所需的计算节点池
> * 创建用于处理输入文件的作业和任务
> * 监视任务执行情况
> * 检索输出文件

本教程使用 [ffmpeg](https://ffmpeg.org/) 开源工具将 MP4 媒体文件并行转换为 MP3 格式。 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

* 适用于 Linux、macOS 或 Windows 的 [Visual Studio 2017](https://www.visualstudio.com/vs) 或 [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1)。

* Batch 帐户和关联的 Azure 存储帐户。 若要创建这些帐户，请参阅 Batch 快速入门（使用 [Azure 门户](quick-create-portal.md)或 [Azure CLI](quick-create-cli.md)）。

* [Windows 64 位版 ffmpeg 3.4](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-3.4-win64-static.zip) (.zip)。 将 zip 文件下载到本地计算机。 本教程只需 zip 文件。 不需将文件解压缩或安装在本地。

## <a name="sign-in-to-azure"></a>登录 Azure

在 [https://portal.azure.com](https://portal.azure.com) 中登录 Azure 门户。

## <a name="add-an-application-package"></a>添加应用程序包

使用 Azure 门户，将 ffmpeg 作为[应用程序包](batch-application-packages.md)添加到 Batch 帐户。 应用程序包有助于管理任务应用程序及其到池中计算节点的部署。 

1. 在 Azure 门户中，单击“更多服务” > “Batch 帐户”，然后单击 Batch 帐户的名称。
3. 单击“应用程序” > “添加”。
4. 对于“应用程序 ID”，请输入 *ffmpeg* 和包版本 *3.4*。 选择此前下载的 ffmpeg zip 文件，然后单击“确定”。 ffmpeg 应用程序包添加到 Batch 帐户。

![添加应用程序包](./media/tutorial-parallel-dotnet/add-application.png)

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-and-run-the-sample"></a>下载并运行示例

### <a name="download-the-sample"></a>下载示例

从 GitHub [下载或克隆示例应用](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial)。 若要使用 Git 客户端克隆示例应用存储库，请使用以下命令：

```
git clone https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial.git
```

导航到包含 Visual Studio 解决方案文件 `BatchDotNetFfmpegTutorial.sln` 的目录。

在 Visual Studio 中打开解决方案文件，使用为帐户获取的值更新 `Program.cs` 中的凭据字符串。 例如：

```csharp
// Batch account credentials
private const string BatchAccountName = "mybatchaccount";
private const string BatchAccountKey  = "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==";
private const string BatchAccountUrl  = "https://mybatchaccount.mybatchregion.batch.azure.com";

// Storage account credentials
private const string StorageAccountName = "mystorageaccount";
private const string StorageAccountKey  = "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ==";
```

[!INCLUDE [batch-credentials-include](../../includes/batch-credentials-include.md)]

另请确保解决方案中 ffmpeg 应用程序包的 ID 和版本与上传到 Batch 帐户的 ffmpeg 包相同。

```csharp
const string appPackageId = "ffmpeg";
const string appPackageVersion = "3.4";
```

### <a name="build-and-run-the-sample-project"></a>生成并运行示例项目

在 Visual Studio 中构建并运行应用程序，或在命令行中使用 `dotnet build` 和 `dotnet run` 命令。 运行应用程序后，请查看代码，了解应用程序的每个部分的作用。 例如，在 Visual Studio 中：

* 右键单击解决方案资源管理器中的解决方案，然后单击“生成解决方案”。 

* 出现提示时，请确认还原任何 NuGet 包。 如果需要下载缺少的包，请确保 [NuGet 包管理器](https://docs.nuget.org/consume/installing-nuget)已安装。

然后运行它。 运行示例应用程序时，控制台输出如下所示。 在执行期间启动池的计算节点时，会遇到暂停并看到`Monitoring all tasks for 'Completed' state, timeout in 00:30:00...`。 

```
Sample start: 11/19/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [WinFFmpegPool]...
Creating job [WinFFmpegJob]...
Adding 5 tasks to job [WinFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]...

Sample end: 11/19/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

转到 Azure 门户中的 Batch 帐户，监视池、计算节点、作业和任务。 例如，若要查看池中计算节点的热度地图，请单击“池” > “WinFFmpegPool”。

任务正在运行时，热度地图如下所示：

![池热度地图](./media/tutorial-parallel-dotnet/pool.png)

以默认配置运行应用程序时，典型的执行时间大约为 **10 分钟**。 池创建过程需要最多时间。

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>查看代码

以下部分将示例应用程序细分为多个执行步骤，用于处理 Batch 服务中的工作负荷。 在阅读本文的其余内容时，请参考解决方案中的文件 `Program.cs`，因为我们并没有讨论示例中的每行代码。

### <a name="authenticate-blob-and-batch-clients"></a>对 Blob 和 Batch 客户端进行身份验证

为了与关联的存储帐户交互，应用使用用于 .NET 的 Azure 存储客户端库。 它使用 [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) 创建对帐户的引用，使用共享密钥身份验证进行身份验证， 然后创建 [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient)。

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                                StorageAccountName, StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

应用创建的 [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) 对象用于创建和管理 Batch 服务中的池、作业和任务。 示例中的 Batch 客户端使用共享密钥身份验证。 Batch 还支持通过 [Azure Active Directory](batch-aad-auth.md) 进行身份验证，以便对单个用户或无人参与应用程序进行身份验证。

```csharp
BatchSharedKeyCredentials sharedKeyCredentials = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(sharedKeyCredentials))
...
```

### <a name="upload-input-files"></a>上传输入文件

应用将 `blobClient` 对象传递至 `CreateContainerIfNotExistAsync` 方法，以便为输入文件（MP4 格式）创建一个存储容器，并为任务输出创建一个容器。

```csharp
CreateContainerIfNotExistAsync(blobClient, inputContainerName;
CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

然后，文件从本地 `InputFiles` 文件夹上传到输入容器。 存储中的文件定义为 Batch [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) 对象，Batch 随后可以将这些对象下载到计算节点。 

上传文件时，涉及到 `Program.cs` 中的两个方法：

* `UploadResourceFilesToContainerAsync`：返回 ResourceFile 对象的集合，并在内部调用 `UploadResourceFileToContainerAsync` 以上传在 `inputFilePaths` 参数中传递的每个文件。
* `UploadResourceFileToContainerAsync`：将每个文件作为 Blob 上传到输入容器。 上传文件后，它会获取该 Blob 的共享访问签名 (SAS) 并返回代表它的 ResourceFile 对象。

```csharp
string inputPath = Path.Combine(Environment.CurrentDirectory, "InputFiles");

List<string> inputFilePaths = new List<string>(Directory.GetFileSystemEntries(inputPath, "*.mp4",
    SearchOption.TopDirectoryOnly));

List<ResourceFile> inputFiles = await UploadResourceFilesToContainerAsync(
  blobClient,
  inputContainerName,
  inputFilePaths);
```

若要详细了解如何使用 .NET 将文件作为 Blob 上传到存储帐户，请参阅[使用 .NET 上传、下载和列出 blob](../storage/blobs/storage-quickstart-blobs-dotnet.md)。

### <a name="create-a-pool-of-compute-nodes"></a>创建计算节点池

然后，该示例会调用 `CreatePoolIfNotExistAsync` 以在 Batch 帐户中创建计算节点池。 这个定义的方法使用 [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) 方法设置节点数、VM 大小和池配置。 在这里，[VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) 对象指定对 Azure 市场中发布的 Windows Server 映像的 [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference)。 Batch 支持 Azure 市场中的各种 VM 映像以及自定义 VM 映像。

节点数和 VM 大小使用定义的常数进行设置。 Batch 支持专用节点和[低优先级](batch-low-pri-vms.md)节点。可以在池中使用这其中的一种，或者两种都使用。 专用节点为池保留。 低优先级节点在 Azure 有剩余 VM 容量时以优惠价提供。 如果 Azure 没有足够的容量，低优先级节点会变得不可用。 默认情况下，此示例创建的池只包含 5 个大小为 *Standard_A1_v2* 的低优先级节点。

ffmpeg 应用程序部署到计算节点的方法是添加对池配置的 [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference)。

[CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) 方法将池提交到 Batch 服务。

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2016-Datacenter-smalldisk",
    version: "latest");

VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.windows amd64");

pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: DedicatedNodeCount,
    targetLowPriorityComputeNodes: LowPriorityNodeCount,
    virtualMachineSize: PoolVMSize,
    virtualMachineConfiguration: virtualMachineConfiguration);

pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
    new ApplicationPackageReference {
    ApplicationId = appPackageId,
    Version = appPackageVersion}};

await pool.CommitAsync();  
```

### <a name="create-a-job"></a>创建作业

Batch 作业可指定在其中运行任务的池以及可选设置，例如工作的优先级和计划。 此示例通过调用 `CreateJobAsync` 创建一个作业。 这个定义的方法使用 [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) 方法在池中创建作业。

[CommitAsync](/dotnet/api/microsoft.azure.batch.cloudjob.commitasync) 方法将作业提交到 Batch 服务。 作业一开始没有任务。

```csharp
CloudJob job = batchClient.JobOperations.CreateJob();
job.Id = JobId;
job.PoolInformation = new PoolInformation { PoolId = PoolId };

await job.CommitAsync();
```

### <a name="create-tasks"></a>创建任务

此示例通过调用 `AddTasksAsync` 方法来创建 [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) 对象的列表，从而在作业中创建任务。 每个 `CloudTask` 都运行 ffmpeg，使用 [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline) 属性处理输入 `ResourceFile` 对象。 ffmpeg 此前已在创建池时安装在每个节点上。 在这里，命令行运行 ffmpeg 将每个输入 MP4（视频）文件转换为 MP3（音频）文件。

此示例在运行命令行后为 MP3 文件创建 [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) 对象。 每个任务的输出文件（在此示例中为一个）都会使用任务的 [OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles) 属性上传到关联的存储帐户中的一个容器。 我们在前面的代码示例中获取了共享访问签名 URL (`outputContainerSasUrl`)，用于提供对输出容器的写权限。 请注意 `outputFile` 对象上设置的条件。 只有在任务成功完成后 (`OutputFileUploadCondition.TaskSuccess`)，任务中的输出文件才会上传到容器。 在 GitHub 上查看完整的[代码示例](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial)，进一步了解实现的详细信息。

然后，示例使用 [AddTaskAsync](/dotnet/api/microsoft.azure.batch.joboperations.addtaskasync) 方法将任务添加到作业，使任务按顺序在计算节点上运行。

```csharp
 // Create a collection to hold the tasks added to the job.
List<CloudTask> tasks = new List<CloudTask>();

for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);

    // Define task command line to convert each input file.
    string appPath = String.Format("%AZ_BATCH_APP_PACKAGE_{0}#{1}%", appPackageId, appPackageVersion);
    string inputMediaFile = inputFiles[i].FilePath;
    string outputMediaFile = String.Format("{0}{1}",
        System.IO.Path.GetFileNameWithoutExtension(inputMediaFile),
        ".mp3");
    string taskCommandLine = String.Format("cmd /c {0}\\ffmpeg-3.4-win64-static\\bin\\ffmpeg.exe -i {1} {2}", appPath, inputMediaFile, outputMediaFile);

    // Create a cloud task (with the task ID and command line)
    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };

    // Task output file
    List<OutputFile> outputFileList = new List<OutputFile>();
    OutputFileBlobContainerDestination outputContainer = new OutputFileBlobContainerDestination(outputContainerSasUrl);
    OutputFile outputFile = new OutputFile(outputMediaFile,
       new OutputFileDestination(outputContainer),
       new OutputFileUploadOptions(OutputFileUploadCondition.TaskSuccess));
    outputFileList.Add(outputFile);
    task.OutputFiles = outputFileList;
    tasks.Add(task);
}

// Add tasks as a collection
await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
return tasks
```

### <a name="monitor-tasks"></a>监视任务

Batch 将任务添加到作业时，该服务自动对任务排队并进行计划，方便其在关联的池中的计算节点上执行。 Batch 根据指定的设置处理所有任务排队、计划、重试和其他任务管理工作。

监视任务的执行有许多方法。 此示例定义的 `MonitorTasks` 方法仅在已完成的情况下状态为“任务失败”或“任务成功”时进行报告。 `MonitorTasks` 代码指定 [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel)，只选择有关任务的最少信息，十分高效。 然后，它会创建 [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor)，以便提供用于监视任务状态的帮助器实用程序。 在 `MonitorTasks` 中，示例会在某个时限内等待所有任务达到 `TaskState.Completed` 状态。 然后，它会终止作业，并对虽已完成但仍遇到故障（例如退出代码非零）的任务进行报告。

```csharp
TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
try
{
    await taskStateMonitor.WhenAll(addedTasks, TaskState.Completed, timeout);
}
catch (TimeoutException)
{
    batchClient.JobOperations.TerminateJob(jobId);
    Console.WriteLine(incompleteMessage);
    return false;
}
batchClient.JobOperations.TerminateJob(jobId);
 Console.WriteLine(completeMessage);
...

```

## <a name="clean-up-resources"></a>清理资源

运行任务之后，应用自动删除所创建的输入存储容器，并允许你选择是否删除 Batch 池和作业。 BatchClient 的 [JobOperations](/dotnet/api/microsoft.azure.batch.batchclient.joboperations) 和 [PoolOperations](/dotnet/api/microsoft.azure.batch.batchclient.pooloperations) 类都有相应的删除方法（在确认删除时调用）。 虽然作业和任务本身不收费，但计算节点收费。 因此，建议只在需要的时候分配池。 删除池时会删除节点上的所有任务输出。 但是，输出文件保留在存储帐户中。

若不再需要资源组、Batch 帐户和存储帐户，请将其删除。 为此，请在 Azure 门户中选择 Batch 帐户所在的资源组，然后单击“删除资源组”。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 将应用程序包添加到 Batch 帐户
> * 通过 Batch 和存储帐户进行身份验证
> * 将输入文件上传到存储
> * 创建运行应用程序所需的计算节点池
> * 创建用于处理输入文件的作业和任务
> * 监视任务执行情况
> * 检索输出文件

如需更多示例，以便了解如何使用 .NET API 来计划和处理 Batch 工作负荷，请参阅 GitHub 上的示例。

> [!div class="nextstepaction"]
> [Batch C# 示例](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp)
