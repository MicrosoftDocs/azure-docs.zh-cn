---
title: Azure 快速入门 - 运行 Batch 作业 - .NET
description: 使用 Batch .NET 客户端库快速运行 Batch 作业和任务。
services: batch
author: laurenhughes
manager: jeconnoc
ms.service: batch
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 11/29/2018
ms.author: lahugh
ms.custom: mvc
ms.openlocfilehash: 6c332861bcaf228e4c1e0a619cf4f84e3be0ba15
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204150"
---
# <a name="quickstart-run-your-first-azure-batch-job-with-the-net-api"></a>快速入门：使用 .NET API 运行第一个 Azure Batch 作业

本快速入门通过基于 Azure Batch .NET API 生成的 C# 应用程序运行 Azure Batch 作业。 此应用将多个输入数据文件上传到 Azure 存储，然后创建包含 Batch 计算节点（虚拟机）的*池*。 再然后，它创建一个示例*作业*，以便运行*任务*，在池中使用基本命令来处理每个输入文件。 完成本快速入门以后，你会了解 Batch 服务的重要概念，并可使用更逼真的工作负荷进行更大规模的 Batch 试用。

![快速入门应用工作流](./media/quick-run-dotnet/sampleapp.png)

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

* 适用于 Linux、macOS 或 Windows 的 [Visual Studio 2017](https://www.visualstudio.com/vs) 或 [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1)。 

* Batch 帐户和关联的 Azure 存储帐户。 若要创建这些帐户，请参阅 Batch 快速入门（使用 [Azure 门户](quick-create-portal.md)或 [Azure CLI](quick-create-cli.md)）。 

## <a name="sign-in-to-azure"></a>登录 Azure

在 [https://portal.azure.com](https://portal.azure.com) 中登录 Azure 门户。

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-the-sample"></a>下载示例

从 GitHub [下载或克隆示例应用](https://github.com/Azure-Samples/batch-dotnet-quickstart)。 若要使用 Git 客户端克隆示例应用存储库，请使用以下命令：

```
git clone https://github.com/Azure-Samples/batch-dotnet-quickstart.git
```

导航到包含 Visual Studio 解决方案文件 `BatchDotNetQuickstart.sln` 的目录。

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

## <a name="build-and-run-the-app"></a>生成并运行应用

若要查看运行中的 Batch 工作流，请在 Visual Studio 中构建并运行应用程序，或在命令行中使用 `dotnet build` 和 `dotnet run` 命令。 运行应用程序后，请查看代码，了解应用程序的每个部分的作用。 例如，在 Visual Studio 中：

* 右键单击解决方案资源管理器中的解决方案，然后单击“生成解决方案”。 

* 出现提示时，请确认还原任何 NuGet 包。 如果需要下载缺少的包，请确保 [NuGet 包管理器](https://docs.nuget.org/consume/installing-nuget)已安装。

然后运行它。 运行示例应用程序时，控制台输出如下所示。 在执行期间启动池的计算节点时，会遇到暂停并看到`Monitoring all tasks for 'Completed' state, timeout in 00:30:00...`。 任务会排队，在第一个计算节点运行后马上运行。 转到 [Azure 门户](https://portal.azure.com)中的 Batch 帐户，监视池、计算节点、作业和任务。

```
Sample start: 11/16/2018 4:02:54 PM

Container [input] created.
Uploading file taskdata0.txt to container [input]...
Uploading file taskdata1.txt to container [input]...
Uploading file taskdata2.txt to container [input]...
Creating pool [DotNetQuickstartPool]...
Creating job [DotNetQuickstartJob]...
Adding 3 tasks to job [DotNetQuickstartJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
```

任务完成后，会看到每个任务的输出，如下所示：

```
Printing task output.
Task: Task0
Node: tvm-2850684224_3-20171205t000401z
Standard out:
Batch processing began with mainframe computers and punch cards. Today it still plays a central role in business, engineering, science, and other pursuits that require running lots of automated tasks....
stderr:
...
```

以默认配置运行应用程序时，典型的执行时间大约为 5 分钟。 初始池设置需要最多时间。 若要再次运行该作业，请从以前的运行中删除该作业，不要删除池。 在预配置的池中，该作业数秒即可完成。


## <a name="review-the-code"></a>查看代码

本快速入门中的 .NET 应用执行以下操作：

* 将三个小的文本文件上传到 Azure 存储帐户中的 Blob 容器。 这些文件是供 Batch 处理的输入。
* 创建一个池，其中包含运行 Windows Server 的计算节点。
* 创建一个作业和三个任务，它们需要在节点上运行。 每个任务都使用 Windows 命令行来处理一个输入文件。 
* 显示文件返回的任务。

有关详细信息，请参阅文件 `Program.cs` 和以下部分。 

### <a name="preliminaries"></a>初步操作

为了与存储帐户交互，应用使用用于 .NET 的 Azure 存储客户端库。 它使用 [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) 创建帐户引用，并据此创建 [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient)。

```csharp
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

应用使用 `blobClient` 引用在存储帐户中创建容器，然后将数据文件上传到该容器。 存储中的文件定义为 Batch [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) 对象，Batch 随后可以将这些对象下载到计算节点。

```csharp
List<string> inputFilePaths = new List<string>
{
    "taskdata0.txt",
    "taskdata1.txt",
    "taskdata2.txt"
};

List<ResourceFile> inputFiles = new List<ResourceFile>();

foreach (string filePath in inputFilePaths)
{
    inputFiles.Add(UploadFileToContainer(blobClient, inputContainerName, filePath));
}
```

应用创建的 [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) 对象用于创建和管理 Batch 服务中的池、作业和任务。 示例中的 Batch 客户端使用共享密钥身份验证。 （Batch 还支持 Azure Active Directory 身份验证。）

```csharp
BatchSharedKeyCredentials cred = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(cred))
...
```

### <a name="create-a-pool-of-compute-nodes"></a>创建计算节点池

为了创建 Batch 池，应用使用 [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) 方法设置节点数、VM 大小和池配置。 在这里，[VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) 对象指定对 Azure 市场中发布的 Windows Server 映像的 [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference)。 Batch 支持 Azure 市场中的各种 Linux 和 Windows Server 映像以及自定义 VM 映像。

节点数 (`PoolNodeCount`) 和 VM 大小 (`PoolVMSize`) 是定义的常数。 此示例默认创建的池包含 2 个大小为 *Standard_A1_v2* 的节点。 就此快速示例来说，建议的大小在性能和成本之间达成了很好的平衡。

[Commit](/dotnet/api/microsoft.azure.batch.cloudpool.commit) 方法将池提交到 Batch 服务。

```csharp

private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        publisher: "MicrosoftWindowsServer",
        offer: "WindowsServer",
        sku: "2016-datacenter-smalldisk",
        version: "latest");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
...

```

### <a name="create-a-batch-job"></a>创建 Batch 作业

Batch 作业是对一个或多个任务进行逻辑分组。 作业包含任务的公用设置，例如优先级以及运行任务的池。 应用使用 [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) 方法在池中创建作业。

[Commit](/dotnet/api/microsoft.azure.batch.cloudjob.commit) 方法将作业提交到 Batch 服务。 作业一开始没有任务。

```csharp
try
{
    CloudJob job = batchClient.JobOperations.CreateJob();
    job.Id = JobId;
    job.PoolInformation = new PoolInformation { PoolId = PoolId };

    job.Commit();
}
...
```

### <a name="create-tasks"></a>创建任务

此应用创建 [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) 对象的列表。 每个任务都使用 [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline) 属性来处理输入 `ResourceFile` 对象。 在示例中，命令行运行 Windows `type` 命令来显示输入文件。 此命令是一个用于演示的简单示例。 使用 Batch 时，可以在命令行中指定应用或脚本。 Batch 提供多种将应用和脚本部署到计算节点的方式。

然后，应用使用 [AddTask](/dotnet/api/microsoft.azure.batch.joboperations.addtask) 方法将任务添加到作业，使任务按顺序在计算节点上运行。

```csharp
for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);
    string inputFilename = inputFiles[i].FilePath;
    string taskCommandLine = String.Format("cmd /c type {0}", inputFilename);

    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };
    tasks.Add(task);
}

batchClient.JobOperations.AddTask(JobId, tasks);
```

### <a name="view-task-output"></a>查看任务输出

应用创建 [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor) 来监视任务，确保其完成。 然后，应用使用 [CloudTask.ComputeNodeInformation](/dotnet/api/microsoft.azure.batch.cloudtask.computenodeinformation) 属性来显示每个已完成任务生成的 `stdout.txt` 文件。 如果任务成功运行，任务命令的输出将写入到 `stdout.txt`：

```csharp
foreach (CloudTask task in completedtasks)
{
    string nodeId = String.Format(task.ComputeNodeInformation.ComputeNodeId);
    Console.WriteLine("Task: {0}", task.Id);
    Console.WriteLine("Node: {0}", nodeId);
    Console.WriteLine("Standard out:");
    Console.WriteLine(task.GetNodeFile(Constants.StandardOutFileName).ReadAsString());
}
```

## <a name="clean-up-resources"></a>清理资源

应用自动删除所创建的存储容器，并允许你选择是否删除 Batch 池和作业。 只要有节点在运行，就会对池收费，即使没有计划作业。 不再需要池时，请将其删除。 删除池时会删除节点上的所有任务输出。

若不再需要资源组、Batch 帐户和存储帐户，请将其删除。 为此，请在 Azure 门户中选择 Batch 帐户所在的资源组，然后单击“删除资源组”。

## <a name="next-steps"></a>后续步骤

本快速入门运行了使用 Batch .NET API 生成的小应用，目的是创建 Batch 池和 Batch 作业。 该作业运行了示例任务，并下载了在节点上产生的输出。 了解 Batch 服务的重要概念以后，即可使用更逼真的工作负荷进行更大规模的 Batch 试用。 若要详细了解 Azure Batch 并使用实际的应用程序演练并行工作负荷，请继续学习 Batch .NET 教程。


> [!div class="nextstepaction"]
> [使用 .NET 处理并行工作负荷](tutorial-parallel-dotnet.md)
