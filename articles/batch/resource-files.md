---
title: 创建和使用资源文件-Azure Batch |Microsoft Docs
description: 了解如何从各种输入源创建 Azure Batch 资源文件。
services: batch
author: laurenhughes
manager: jeconnoc
ms.service: batch
ms.topic: article
ms.date: 03/14/2019
ms.author: lahugh
ms.openlocfilehash: 113faffb0ebac50a67c96ce21e0ee2c1564bb4fc
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65405643"
---
# <a name="creating-and-using-resource-files"></a>创建和使用资源文件

Azure Batch 任务通常需要某种形式的数据处理。 资源文件是向 Batch 虚拟机 (VM) 通过任务提供此数据的方法。 所有类型的任务都支持资源文件： 任务，启动任务，作业准备任务、 作业释放任务等。这篇文章介绍如何创建资源文件并将其放置在 VM 上的几个常见的方法。  

资源文件是一种机制来将数据放到在批处理中的 VM，但灵活的数据类型以及如何使用它。 有，但是，一些常见用例：

1. 预配使用启动任务上的资源文件的每个 VM 上的公共文件
1. 预配任务处理输入的数据

例如，常见的文件可能是用于安装任务所要运行的应用程序的启动任务上的文件。 输入的数据可能是原始图像或视频数据或要处理的批处理的任何信息。

## <a name="types-of-resource-files"></a>类型的资源文件

有几个不同的选项可用于生成资源文件。 资源文件的创建过程各不相同，具体取决于原始数据的存储位置。

用于创建资源文件的选项：

- [存储容器 URL](#storage-container-url):从 Azure 中的任何存储容器生成资源文件
- [存储容器名称](#storage-container-name):从 Azure 存储帐户链接到 Batch 中的容器的名称生成一个资源文件
- [Web 终结点](#web-endpoint):从任何有效的 HTTP URL 生成资源文件

### <a name="storage-container-url"></a>存储容器 URL

使用存储容器 URL 意味着您可以访问在 Azure 中使用正确的权限的任何存储容器中的文件。

在此C#的示例中，具有已文件上传到 Azure 存储容器作为 blob 存储。 若要访问需创建一个资源文件的数据，我们首先需要获取对存储容器访问权限。

创建共享的访问签名 (SAS) URI 访问的存储容器的正确权限。 设置过期时间和针对该 SAS 的权限。 在这种情况下，没有指定开始时间，因此该 SAS 将立即生效并到期后会生成两个小时。

```csharp
SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
{
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
    Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List
};
```

> [!NOTE]
> 为容器访问权限，你必须使`Read`并`List`权限，而使用 blob 访问权限，您只需`Read`权限。

后配置的权限，创建 SAS 令牌，并设置访问存储容器 SAS URL 的格式。 对于存储容器中，使用带有格式的 SAS URL 生成的资源文件[ `FromStorageContainerUrl` ](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.resourcefile.fromstoragecontainerurl?view=azure-dotnet)。

```csharp
CloudBlobContainer container = blobClient.GetContainerReference(containerName);

string sasToken = container.GetSharedAccessSignature(sasConstraints);
string containerSasUrl = String.Format("{0}{1}", container.Uri, sasToken);

ResourceFile inputFile = ResourceFile.FromStorageContainerUrl(containerSasUrl);
```

生成 SAS URL 的替代方法是启用匿名公共读取访问对容器及其 blob Azure Blob 存储中。 通过此操作，而无需共享帐户密钥，而无需 SAS 可以授予对这些资源的只读访问。 公共读取访问权限是通常用于方案想要始终可以进行匿名读取访问特定 blob。 如果这种情况下适合你的解决方案，请参阅[匿名访问 blob](../storage/blobs/storage-manage-access-to-resources.md)一文，了解有关管理对 blob 数据的访问的详细信息。

### <a name="storage-container-name"></a>存储容器名称

无需配置和创建的 SAS URL，可以使用 Azure 存储容器的名称以访问 blob 数据。 存储容器中链接到 Batch 帐户，称为自动存储帐户的 Azure 存储帐户使用到的需求。 使用自动存储帐户的存储容器名称，可以绕过配置，创建一个用于访问存储容器的 SAS URL。

在此示例中，我们假定数据要用于创建资源文件已在 Azure 存储帐户链接到 Batch 帐户。 如果还没有自动存储帐户，请参阅中的步骤[创建 Batch 帐户](batch-account-create-portal.md)有关如何创建和将帐户链接的详细信息。

通过使用链接的存储帐户，无需创建和配置指向存储容器的 SAS URL。 相反，提供链接的存储帐户中的存储容器的名称。

```csharp
ResourceFile inputFile = ResourceFile.FromAutoStorageContainer(containerName);
```

### <a name="web-endpoint"></a>Web 终结点

不上传到 Azure 存储的数据仍可用于创建资源文件。 可以指定任何有效的 HTTP URL，其中包含输入的数据。 批处理 api，提供的 URL，然后将数据用于创建资源文件。

在下面的示例C#虚构 GitHub 终结点上承载的示例中，输入数据。 API 检索有效的 web 终结点中的文件，并生成一个资源文件，可供您的任务。 这种情况下，需要不使用凭据。

```csharp
ResourceFile inputFile = ResourceFile.FromUrl("https://github.com/foo/file.txt", filePath);
```

## <a name="tips-and-suggestions"></a>提示和建议

每个 Azure Batch 任务使用文件以不同的方式，这就是为什么批处理有选项可用于管理任务上的文件。 以下方案不适合在全面，但改为介绍几个常见的情况下，提供建议。

### <a name="many-resource-files"></a>多个资源文件

在 Batch 作业可能包含多个任务都使用相同、 公共文件。 如果多个任务之间共享常见的任务文件，使用应用程序包以包含而不是使用资源文件的文件可能是更好的选择。 应用程序包提供下载速度的优化。 此外，应用程序包中的数据缓存之间的任务，因此，如果任务文件不经常更改，应用程序包可能会非常适合于你的解决方案。 使用应用程序包，您不必手动管理多个资源文件或生成 SAS Url 来访问 Azure 存储中的文件。 在使用 Azure 存储来存储和应用程序包的计算节点部署后台的工作批次。

如果每个任务具有唯一的该任务的多个文件，资源文件很可能是最佳选择。 通常使用唯一的文件的任务需要更新或更换，这并不那么容易，如何处理应用程序的包内容。 资源文件可更加灵活地更新、 添加或编辑单个文件。

### <a name="number-of-resource-files-per-task"></a>每个任务的资源文件的数量

如果有多个任务中指定的几百个资源文件，批处理可能会拒绝它太大的任务。 最好保持较小的最小化任务本身上的资源文件的任务。

如果没有方法来最大程度减少文件数量在任务需求，可以通过创建单个资源文件引用资源文件的存储容器来优化任务。 若要执行此操作，将资源文件放入 Azure 存储容器，并使用不同的资源文件的"容器"模式。 Blob 前缀选项用于指定要为你的任务下载的文件的集合。

## <a name="next-steps"></a>后续步骤

- 了解如何[应用程序包](batch-application-packages.md)作为资源文件的替代方法。
- 有关使用容器的资源文件的详细信息，请参阅[容器工作负荷](batch-docker-container-workloads.md)。
- 若要了解如何收集和保存你的任务中的输出数据，请参阅[保存作业和任务输出](batch-task-output.md)。
- 了解适用于生成批处理解决方案的[批处理 API 和工具](batch-apis-tools.md)。
