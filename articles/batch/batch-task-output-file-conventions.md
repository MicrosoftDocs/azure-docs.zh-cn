---
title: 使用适用于 .NET 的文件约定库将作业和任务输出保存到 Azure 存储 - Azure Batch | Microsoft Docs
description: 了解如何使用适用于 .NET 的 Azure Batch 文件约定库将 Batch 任务和作业输出保存到 Azure 存储，并在 Azure 门户中查看保存的输出。
services: batch
documentationcenter: .net
author: laurenhughes
manager: jeconnoc
editor: ''
ms.assetid: 16e12d0e-958c-46c2-a6b8-7843835d830e
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 11/14/2018
ms.author: lahugh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 96f1cb60dbb7cf08224e8566852cf47fe5f0fa1c
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203563"
---
# <a name="persist-job-and-task-data-to-azure-storage-with-the-batch-file-conventions-library-for-net"></a>使用适用于 .NET 的 Batch 文件约定库将作业和任务数据保存到 Azure 存储

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

保存任务数据的方法之一是使用[适用于 .NET 的 Azure Batch 文件约定库][nuget_package]。 文件约定库简化了在 Azure 存储中存储和检索任务输出数据的过程。 可以在任务代码和客户端代码中使用文件约定库 &mdash; 在任务代码中用于保存文件，在客户端代码中用于列出和检索文件。 任务代码还可以使用该库来检索上游任务的输出（例如，在[任务依赖项](batch-task-dependencies.md)方案中）。

若要使用文件约定库检索输出文件，可以查找给定作业或任务的文件：按 ID 和用途列出这些文件即可。 不需要知道文件的名称或位置。 例如，可以使用文件约定库列出给定任务的所有中间文件，或获取给定作业的预览文件。

> [!TIP]
> 对于在使用虚拟机配置创建的池上运行的任务和作业管理器任务，从版本 2017-05-01 开始，Batch 服务 API 支持将此类任务的输出数据保存到 Azure 存储。 Batch 服务 API 提供了一种简单的方法来从创建任务的代码内部保存输出，可以替代文件约定库。 可以修改 Batch 客户端应用程序来保存输出，而无需更新任务运行的应用程序。 有关详细信息，请参阅[使用 Batch 服务 API 将任务数据保存到 Azure 存储](batch-task-output-files.md)。

## <a name="when-do-i-use-the-file-conventions-library-to-persist-task-output"></a>何时使用文件约定库保存任务输出？

Azure Batch 提供多种方式来保存任务输出。 文件约定最适合以下情况：

- 可以轻松修改任务运行的应用程序的代码，以使用文件约定库来保存文件。
- 希望在任务仍然运行时，将数据流式传输到 Azure 存储。
- 希望保存使用云服务配置或虚拟机配置创建的池中的数据。
- 作业中的客户端应用程序或其他任务需要根据 ID 或用途定位并下载任务输出文件。
- 想要在 Azure 门户中查看任务输出。

如果你的情形不同于上面列出的情形，则可能需要考虑其他方式。 有关保存任务输出的其他选项的详细信息，请参阅[将作业和任务输出保存到 Azure 存储](batch-task-output.md)。

## <a name="what-is-the-batch-file-conventions-standard"></a>什么是 Batch 文件约定标准？

[Batch 文件约定标准](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)为输出文件要写入到的目标容器和 Blob 路径提供命名方案。 可在 Azure 门户中查看 Azure 存储中保存的、遵守文件约定标准的文件。 门户可以识别命名约定，因此可以显示遵守该约定的文件。

适用于 .NET 的文件约定库根据文件约定标准，自动为存储容器和任务输出文件命名。 文件约定库还提供根据作业 ID、任务 ID 或用途在 Azure 存储中查询输出文件的方法。

如果使用除 .NET 之外的语言进行开发，则可以在应用程序中自行实现文件约定标准。 有关详细信息，请参阅[实现 Batch 文件约定标准](batch-task-output.md#implement-the-batch-file-conventions-standard)。

## <a name="link-an-azure-storage-account-to-your-batch-account"></a>将 Azure 存储帐户链接到 Batch 帐户

若要使用文件约定库将输出数据保存到 Azure 存储，必须先将 Azure 存储帐户链接到 Batch 帐户。 如果尚未这样做，请使用 [Azure 门户](https://portal.azure.com)将存储帐户链接到 Batch 帐户：

1. 导航到 Azure 门户中的 Batch 帐户。
1. 在“设置”下面，选择“存储帐户”。
1. 如果目前没有任何存储帐户与 Batch 帐户关联，请单击“存储帐户(无)”。
1. 从订阅列表中选择一个存储帐户。 为获得最佳性能，请使用运行任务的 Batch 帐户所在的区域中的 Azure 存储帐户。

## <a name="persist-output-data"></a>保存输出数据

若要使用文件约定库保存作业和任务输出数据，请在 Azure 存储中创建一个容器，然后将输出保存到该容器。 在任务代码中使用[适用于 .NET 的 Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage)将任务输出上传到该容器。 

有关在 Azure 存储中使用容器和 Blob 的详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。

> [!WARNING]
> 使用文件约定库保存的所有作业和任务输出存储在同一个容器中。 如果大量的任务同时尝试保存文件，可能会强制实施[存储限制](../storage/common/storage-performance-checklist.md#blobs)。

### <a name="create-storage-container"></a>创建存储容器

若要将任务输出保存到 Azure 存储，首先请通过调用 [CloudJob][net_cloudjob].[PrepareOutputStorageAsync][net_prepareoutputasync] 创建容器。 此扩展方法采用 [CloudStorageAccount][net_cloudstorageaccount] 对象作为参数。 它会创建根据文件约定标准命名的容器，以便通过 Azure 门户和本文稍后介绍的检索方法发现其内容。

我们通常将创建容器的代码放在客户端应用程序中 &mdash; 即创建池、作业和任务的应用程序。

```csharp
CloudJob job = batchClient.JobOperations.CreateJob(
    "myJob",
    new PoolInformation { PoolId = "myPool" });

// Create reference to the linked Azure Storage account
CloudStorageAccount linkedStorageAccount =
    new CloudStorageAccount(myCredentials, true);

// Create the blob storage container for the outputs
await job.PrepareOutputStorageAsync(linkedStorageAccount);
```

### <a name="store-task-outputs"></a>存储任务输出

在 Azure 存储中准备好一个容器后，任务可以使用文件约定库中的 [TaskOutputStorage][net_taskoutputstorage] 类将输出保存到该容器。

在任务代码中，请先创建一个 [TaskOutputStorage][net_taskoutputstorage] 对象，然后，当任务完成其工作时，会调用 [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync] 方法将其输出保存到 Azure 存储。

```csharp
CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");

TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
    linkedStorageAccount, jobId, taskId);

/* Code to process data and produce output file(s) */

await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");
```

[TaskOutputStorage](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage).[SaveAsync](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync#overloads) 方法的 `kind` 参数将保存的文件分类。 有四个预定义的 [TaskOutputKind][net_taskoutputkind] 类型：`TaskOutput`、`TaskPreview`、`TaskLog` 和 `TaskIntermediate.`。也可以定义输出的自定义类别。

以后在 Batch 中查询给定任务的已保存输出时，可以使用这些输出类型来指定要列出哪种类型的输出。 换而言之，列出某个任务的输出时，可以根据某种输出类型来筛选列表。 例如，“列出任务 *109* 的预览输出。” 本文稍后的“检索输出”部分中会详细介绍如何列出和检索输出。

> [!TIP]
> 输出种类还决定了特定文件在 Azure 门户中的显示位置：TaskOutput 类别的文件显示在“任务输出文件”下，TaskLog 文件显示在“任务日志”下。

### <a name="store-job-outputs"></a>存储作业输出

除了存储任务输出以外，还可以存储与整个作业关联的输出。 例如，在电影渲染作业的合并任务中，可以将完全渲染的电影保存为作业输出。 作业完成后，客户端应用程序可以列出并检索该作业的输出，而不需要查询各个任务。

通过调用 [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] 方法存储作业输出，并指定 [JobOutputKind][net_joboutputkind] 和文件名：

```csharp
CloudJob job = new JobOutputStorage(acct, jobId);
JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);

await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");
```

与用于任务输出的 **TaskOutputKind** 类型一样，可以使用 [JobOutputKind][net_joboutputkind] 类型来对作业的保存文件进行分类。 以后可以使用此参数查询（列出）特定的输出类型。 **JobOutputKind** 类型包括输出和预览类别，并支持创建自定义类别。

### <a name="store-task-logs"></a>存储任务日志

除了在任务或作业完成时将文件保存到持久性存储以外，可能还需要保存执行某个任务期间更新的文件 &mdash; 例如，日志文件或 `stdout.txt` 和 `stderr.txt`。 为此，Azure Batch 文件约定库提供了 [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync] 方法。 使用 [SaveTrackedAsync][net_savetrackedasync]，可以跟踪对节点上的文件所做的更新（按照指定的间隔），并将这些更新保持到 Azure 存储。

在以下代码片段中，我们会在执行任务期间，每隔 15 秒使用 [SaveTrackedAsync][net_savetrackedasync] 更新 Azure 存储中的 `stdout.txt`：

```csharp
TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
string logFilePath = Path.Combine(
    Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

// The primary task logic is wrapped in a using statement that sends updates to
// the stdout.txt blob in Storage every 15 seconds while the task code runs.
using (ITrackedSaveOperation stdout =
        await taskStorage.SaveTrackedAsync(
        TaskOutputKind.TaskLog,
        logFilePath,
        "stdout.txt",
        TimeSpan.FromSeconds(15)))
{
    /* Code to process data and produce output file(s) */

    // We are tracking the disk file to save our standard output, but the
    // node agent may take up to 3 seconds to flush the stdout stream to
    // disk. So give the file a moment to catch up.
     await Task.Delay(stdoutFlushDelay);
}
```

带有注释的部分 `Code to process data and produce output file(s)` 是任务通常会执行的代码的占位符。 例如，代码可能会从 Azure 存储下载数据，并对其执行转换或计算。 此代码片段的重要部分演示了如何在 `using` 块中包装此类代码，以定期使用 [SaveTrackedAsync][net_savetrackedasync] 更新文件。

节点代理是一个程序，它在池中的每个节点上运行，并在节点与 Batch 服务之间提供命令和控制接口。 此 `using` 块的末尾必须有 `Task.Delay` 调用，确保节点代理有时间将标准输出的内容刷新到节点上的 stdout.txt 文件。 若没有此延迟，可能会遗漏最后几秒的输出。 并非所有文件都需要此延迟。

> [!NOTE]
> 启用 **SaveTrackedAsync** 文件跟踪时，只会在 Azure 存储中保存被跟踪文件的追加内容。 此方法只应该用于跟踪非轮转的日志文件，或使用文件末尾的追加操作进行写入的其他文件。

## <a name="retrieve-output-data"></a>检索输出数据

使用 Azure Batch 文件约定库检索保存的输出时，以任务和作业为中心的方式执行此操作。 可以请求给定任务或作业的输出，而无需知道输出在 Azure 存储中的路径，甚至不需要知道其文件名。 可以按任务 ID 或作业 ID 请求输出文件。

以下代码片段将循环访问某个作业的任务，列显有关该任务的输出文件的一些信息，然后从存储空间下载该任务的文件。

```csharp
foreach (CloudTask task in myJob.ListTasks())
{
    foreach (OutputFileReference output in
        task.OutputStorage(storageAccount).ListOutputs(
            TaskOutputKind.TaskOutput))
    {
        Console.WriteLine($"output file: {output.FilePath}");

        output.DownloadToFileAsync(
            $"{jobId}-{output.FilePath}",
            System.IO.FileMode.Create).Wait();
    }
}
```

## <a name="view-output-files-in-the-azure-portal"></a>在 Azure 门户中查看输出文件

Azure 门户将显示使用 [Batch 文件约定标准](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)保存到链接的 Azure 存储帐户的任务输出文件和日志。 可以使用所选的语言自行实现这些约定，或者在 .NET 应用程序中使用文件约定库。

若要在门户中显示输出文件，必须满足以下要求：

1. 将 Azure 存储帐户链接到 Batch 帐户。
1. 保存输出时遵循存储容器和文件的预定义命名约定。 可以在文件约定库的[自述文件][github_file_conventions_readme]中找到这些约定的定义。 如果使用 [Azure Batch 文件约定][nuget_package]库保存输出，则会根据文件约定标准保存文件。

若要在 Azure 门户中查看任务输出文件和日志，请导航到要查看其输出的任务，然后单击“保存的输出文件”或“保存的日志”。 下图显示了 ID 为“007”的任务的“保存的输出文件”：

![Azure 门户中的“任务输出”边栏选项卡][2]

## <a name="code-sample"></a>代码示例

[PersistOutputs][github_persistoutputs] 示例项目是 GitHub 上的 [Azure Batch 代码示例][github_samples]之一。 此 Visual Studio 解决方案演示如何使用 Azure Batch 文件约定库将任务输出保存到持久性存储。 若要运行该示例，请遵循以下步骤：

1. 在 **Visual Studio 2017** 中打开项目。
2. 将 Batch 和存储**帐户凭据**添加到 Microsoft.Azure.Batch.Samples.Common 项目中的 **AccountSettings.settings**。
3. **生成**（但不要运行）该解决方案。 根据提示还原所有 NuGet 包。
4. 使用 Azure 门户上传 **PersistOutputsTask** 的[应用程序包](batch-application-packages.md)。 在 .zip 包中包含 `PersistOutputsTask.exe` 及其依赖程序集，将应用程序 ID 设置为“PersistOutputsTask”，将应用程序包版本设置为“1.0”。
5. **启动**（运行）**PersistOutputs** 项目。
6. 当系统提示选择用于运行示例的持久化技术时，请输入 **1**，以运行使用文件约定库保存任务输出的示例。 

## <a name="next-steps"></a>后续步骤

### <a name="get-the-batch-file-conventions-library-for-net"></a>获取适用于 .NET 的 Batch 文件约定库

[NuGet][nuget_package] 上提供了适用于 .NET 的 Batch 文件约定库。 该库使用新方法扩展了 [CloudJob][net_cloudjob] 和 [CloudTask][net_cloudtask] 类。 另请参阅文件约定库的[参考文档](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.conventions.files)。

GitHub 上的用于 .NET 的 Microsoft Azure SDK 存储库中提供了文件约定库的[源代码][github_file_conventions]。 

### <a name="explore-other-approaches-for-persisting-output-data"></a>探索保存输出数据的其他方法

- 有关保存任务和作业数据的概述，请参阅[将作业和任务输出保存到 Azure 存储](batch-task-output.md)。
- 若要了解如何使用 Batch 服务 API 保存输出数据，请参阅[使用 Batch 服务 API 将任务数据保存到 Azure 存储](batch-task-output-files.md)。

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Batch/FileConventions
[github_file_conventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_cloudstorageaccount]: https://docs.microsoft.com/java/api/com.microsoft.azure.storage._cloud_storage_account
[net_cloudtask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_fileconventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[net_joboutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputkind.aspx
[net_joboutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.aspx
[net_joboutputstorage_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync.aspx
[net_msdn]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[net_prepareoutputasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync.aspx
[net_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync.aspx
[net_savetrackedasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync.aspx
[net_taskoutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputkind.aspx
[net_taskoutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.aspx
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: https://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "门户中“保存的输出文件”和“保存的日志”选择器"
[2]: ./media/batch-task-output/task-output-02.png "Azure 门户中的“任务输出”边栏选项卡"
