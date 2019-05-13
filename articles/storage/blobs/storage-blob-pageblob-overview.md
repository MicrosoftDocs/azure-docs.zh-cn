---
title: Azure 页 Blob 概述 | Microsoft Docs
description: 概述了 Azure 页 Blob 及其优势，包括用例和示例脚本。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 01/03/2019
ms.author: tamram
ms.reviewer: wielriac
ms.subservice: blobs
ms.openlocfilehash: c0d5108ee235be74f6188bfc8b590069a4e790b5
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192642"
---
# <a name="overview-of-azure-page-blobs"></a>Azure 页 Blob 概述

Azure 存储提供了三种类型的 Blob 存储：块 Blob、追加 Blob 和页 Blob。 块 Blob 由块组成，非常适合用于存储文本或二进制文件，以及高效上传大型文件。 追加 Blob 也由块组成，但它们已针对追加操作进行优化，因此非常适合用于日志记录方案。 页 Blob 由总大小可达 8 TB 的 512 字节页面组成，专为频繁的随机读/写操作而设计。 页 Blob 是 Azure IaaS 磁盘的基础。 本文重点介绍页 Blob 的功能和优势。

页 Blob 是 512 字节页面的集合，提供读/写任意字节范围的功能。 因此，页 Blob 非常适用于存储基于索引的稀疏数据结构，如虚拟机和数据库的 OS 磁盘与数据磁盘。 例如，Azure SQL 数据库使用页 Blob 作为数据库的基础持久性存储。 此外，页 Blob 往往还用于支持基于范围的更新的文件。  

Azure 页 Blob 的重要功能包括 REST 接口、基础存储持久性，以及无缝迁移到 Azure 的功能。 下一部分将更详细地介绍这些功能。 此外，有两种类型的存储目前支持 Azure 页 Blob：高级存储和标准存储。 高级存储专门针对需要持续高性能和低延迟的工作负载而设计，因此，高级页 Blob 非常适合用于高性能数据存储数据库。  标准存储更具成本效益，可用于运行对延迟不太敏感的工作负荷。

## <a name="sample-use-cases"></a>示例用例

让我们从 Azure IaaS 磁盘着手，讨论页 Blob 的几种用例。 Azure 页 Blob 是 Azure IaaS 虚拟磁盘平台的主干。 Azure OS 磁盘和数据磁盘都实现为虚拟磁盘，其中的数据持久保存在 Azure 存储平台中，然后传送到虚拟机以获得最大性能。 Azure 磁盘以 Hyper-V [VHD 格式](https://technet.microsoft.com/library/dd979539.aspx)保存，并在 Azure 存储中存储为[页 Blob](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs)。 除了对 Azure IaaS VM 使用虚拟磁盘以外，页 Blob 还可实现 PaaS 和 DBaaS 方案，例如 Azure SQL 数据库服务，该服务目前使用页 Blob 存储 SQL 数据，以便针对数据库快速执行随机读写操作。 另一个示例是，如果使用 PaaS 服务作为共享媒体来访问协作式视频编辑应用程序，则页 Blob 可以实现对媒体中随机位置的快速访问。 此外，多个用户可以使用页 Blob 快速高效地编辑和合并同一媒体。 

第一方 Microsoft 服务（例如 Azure Site Recovery 和 Azure 备份）以及许多第三方开发商已使用页 Blob 的 REST 接口实现了行业领先的创新。 下面是在 Azure 上实现的一些独特方案： 
* 应用程序主导的增量快照管理：应用程序可以利用页 Blob 快照和 REST API 来保存应用程序检查点，而不会产生高昂的数据复制成本。 Azure 存储支持页 Blob 的本地快照，这类快照不要求复制整个 blob。 使用这些公共快照 API 还可以访问和复制快照之间的增量数据。
* 将应用程序和数据从本地实时迁移到云中：复制本地数据并使用 REST API 将数据直接写入 Azure 页 Blob，同时，本地 VM 可继续保持运行。 与目标同步后，可以使用该数据快速故障转移到 Azure VM。 这样，便可以在几乎不造成停机的情况下，将 VM 和虚拟磁盘从本地迁移到云中，因为数据迁移在后台发生，同时我们可以继续使用 VM，并且故障转移所需的停机时间很短（以分钟计）。
* [基于 SAS](../common/storage-dotnet-shared-access-signature-part-1.md) 的共享访问，可以实现支持并发控制的方案，例如多个读取者和单个写入者。

## <a name="page-blob-features"></a>页 Blob 功能

### <a name="rest-api"></a>REST API
请参阅以下文档，开始[使用页 Blob 进行开发](storage-dotnet-how-to-use-blobs.md)。 例如，请查看如何使用用于 .NET 的存储客户端库来访问页 Blob。 

下图描绘了帐户、容器和页 Blob 之间的整体关系。

![](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure1.png)

#### <a name="creating-an-empty-page-blob-of-a-specified-size"></a>创建指定大小的空页 Blob
为了创建页 Blob，让我们先创建一个 **CloudBlobClient** 对象，其中包含用于访问存储帐户（图 1 中的 *pbaccount*）的 Blob 存储的基 URI；另外创建 **StorageCredentialsAccountAndKey** 对象，如以下示例所示。 然后，该示例展示了如何先创建对 CloudBlobContainer 对象的引用，再创建容器 (testvhds)（如果它尚未存在）。 然后，使用 **CloudBlobContainer** 对象，通过指定要访问的页 Blob 名称 (os4.vhd)，来创建对 **CloudPageBlob** 对象的引用。 若要创建页 Blob，请调用 [CloudPageBlob.Create](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.create?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudPageBlob_Create_System_Int64_Microsoft_WindowsAzure_Storage_AccessCondition_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_) 并传入要创建的 blob 的最大大小。 *blobSize* 必须是 512 字节的倍数。

```csharp
using Microsoft.WindowsAzure.StorageClient;
long OneGigabyteAsBytes = 1024 * 1024 * 1024;
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the blob client.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Retrieve a reference to a container.
CloudBlobContainer container = blobClient.GetContainerReference("testvhds");

// Create the container if it doesn't already exist.
container.CreateIfNotExists();

CloudPageBlob pageBlob = container.GetPageBlobReference("os4.vhd");
pageBlob.Create(16 * OneGigabyteAsBytes);
```

#### <a name="resizing-a-page-blob"></a>重设页 Blob 的大小
若要在创建后重设页 Blob 的大小，请使用[重设大小](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.resize?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudPageBlob_Resize_System_Int64_Microsoft_WindowsAzure_Storage_AccessCondition_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_) API。 请求的大小应为 512 字节的倍数。
```csharp
pageBlob.Resize(32 * OneGigabyteAsBytes); 
```

#### <a name="writing-pages-to-a-page-blob"></a>将页面写入页 Blob
若要写入页面，请使用 [CloudPageBlob.WritePages](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.beginwritepages?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudPageBlob_BeginWritePages_System_IO_Stream_System_Int64_System_String_Microsoft_WindowsAzure_Storage_AccessCondition_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_System_AsyncCallback_System_Object_) 方法。  这样，便可以写入一组有序页面（最大 4MB）。 写入的偏移量必须在某个 512 字节边界处 (startingOffset % 512 == 0) 开始，在某个 512 边界 - 1 处结束。  下面的代码示例展示了如何对 blob 调用 **WritePages**：

```csharp
pageBlob.WritePages(dataStream, startingOffset); 
```

针对一组有序页面发出的写入请求在 Blob 服务中成功、已复制以实现持久性和复原能力后，即会提交写入，将向客户端返回成功响应。  

下图显示了 2 种不同的写入操作：

![](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure2.png)

1.  一个写入操作从长度 1024 字节的偏移 0 处开始 
2.  一个写入操作从长度 1024 字节的偏移 4096 处开始 

#### <a name="reading-pages-from-a-page-blob"></a>从页 Blob 中读取页面
若要读取页面，请使用 [CloudPageBlob.DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.icloudblob.downloadrangetobytearray?view=azure-dotnet) 方法从页 Blob 中读取字节范围。 这样，便可以从 Blob 中的任意偏移位置开始下载整个 Blob 或字节范围。 读取时，偏移不需要在 512 的倍数位置开始。 从 NUL 页面读取字节时，服务将返回零字节。
```csharp
byte[] buffer = new byte[rangeSize];
pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, pageBlobOffset, rangeSize); 
```
下图显示 BlobOffSet 为 256 且 rangeSize 为 4352 的读取操作。 返回的数据以橙色突出显示。 为 NUL 页面返回了零

![](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure3.png)

如果使用稀疏填充的 Blob，可以只下载有效的页面区域，以避免支付零字节的传出费用，并降低下载延迟。  若要确定哪些页面以数据为基础，请使用 [CloudPageBlob.GetPageRanges](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.getpageranges?view=azure-dotnet)。 然后，可以枚举返回的范围，并下载每个范围中的数据。 
```csharp
IEnumerable<PageRange> pageRanges = pageBlob.GetPageRanges();

foreach (PageRange range in pageRanges)
{
    // Calculate the rangeSize
    int rangeSize = (int)(range.EndOffset + 1 - range.StartOffset);

    byte[] buffer = new byte[rangeSize];

    // Read from the correct starting offset in the page blob and place the data in the bufferOffset of the buffer byte array
    pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, range.StartOffset, rangeSize); 

    // TODO: use the buffer for the page range just read
} 
```

#### <a name="leasing-a-page-blob"></a>租赁页 Blob
“租赁 Blob”操作在 Blob 上针对写入与删除操作建立和管理一把锁。 如果要从多个客户端访问页 Blob，则此操作非常有用，因为它可以确保每次只有一个客户端能够写入 Blob。 例如，Azure 磁盘利用此租赁机制来确保磁盘只能由一个 VM 管理。 锁的持续时间可以是 15 到 60 秒，也可以是无限期。 有关更多详细信息，请参阅[此文档](/rest/api/storageservices/lease-blob)。

除了丰富的 REST API 以外，页 Blob 还提供共享访问、持久性和增强的安全性。 后续的篇幅将更详细地介绍这些优势。 

### <a name="concurrent-access"></a>并发访问
页 Blob REST API 及其租赁机制可让应用程序从多个客户端访问页 Blob。 例如，假设你需要构建一个要与多名用户共享存储对象的分布式云服务。 该服务可能是向多个用户提供大型图像集合的 Web 应用程序。 实现此目的的方法之一是使用包含附加磁盘的 VM。 此方法的弊端包括：(i) 存在只能将一个磁盘附加到一个 VM 的约束，因此限制了可伸缩性和灵活性，并增大了风险。 如果该 VM 或其上运行的服务出现问题，则由于租赁机制，只能在租约过期或中断之后，才能访问图像；(ii) 使用 IaaS VM 会产生额外的成本。 

一种替代做法是直接通过 Azure 存储 REST API 使用页 Blob。 使用此方法不会产生高昂的 IaaS VM 使用成本；可直接从多个客户端访问页 Blob，因此获得完全的灵活性；不需要附加/分离磁盘，因此简化了经典部署模型；可消除 VM 出现问题的风险。 此外，在随机读/写操作方面，此方法提供的性能级别与使用磁盘时相同

### <a name="durability-and-high-availability"></a>持续性和高可用性
标准存储和高级存储都属于持久性存储，其中的页 Blob 数据始终得到复制，以确保持久性和高可用性。 有关 Azure 存储冗余的详细信息，请参阅[此文档](../common/storage-redundancy.md)。 Azure 为 IaaS 磁盘和页 Blob 不断提供企业级持久性，[年化故障率](https://en.wikipedia.org/wiki/Annualized_failure_rate)为 0%，达到行业领先水平。 也就是说，Azure 永远不会丢失客户的页 Blob 数据。 

### <a name="seamless-migration-to-azure"></a>无缝迁移到 Azure
对于有意实施其自己的自定义备份解决方案的客户和开发人员，Azure 还提供只保存增量数据的增量快照。 此功能避免了初始完整复制的成本，可大幅降低备份成本。 除了有效读取和复制差异数据的功能以外，还有一个强大的功能可以进一步推动开发人员的创新，在 Azure 上带来极佳的备份和灾难恢复 (DR) 体验。 可以使用 [Blob 快照](/rest/api/storageservices/snapshot-blob)为 Azure 上的 VM 设置自己的备份或灾难恢复解决方案，并可以使用[获取页面范围](/rest/api/storageservices/get-page-ranges) API 和[增量复制 Blob](/rest/api/storageservices/incremental-copy-blob) API 轻松复制增量数据以实现灾难恢复。 

此外，许多企业已在本地数据中心运行关键工作负荷。 若要将工作负荷迁移到云中，一个主要的考虑因素是复制数据时需要多长时间的停机，以及在交接之后出现不可预见问题的风险。 在许多情况下，停机时间可能是迁移到云的一个阻碍因素。 Azure 使用页 Blob REST API 解决了此问题，可以在几乎不会对关键工作负荷造成干扰的情况下实现云迁移。 

有关如何创建快照以及如何从快照还原页 Blob 的示例，请参阅[使用增量快照设置备份过程](../../virtual-machines/windows/incremental-snapshots.md)一文。
