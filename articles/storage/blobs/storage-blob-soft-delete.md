---
title: Azure 存储 blob 的软删除 | Microsoft Docs
description: Azure 存储现提供 Blob 对象软删除，目的是为了在应用程序或其他存储帐户用户错误地修改或删除数据后可以更轻松地恢复数据。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/23/2019
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 8c23e429966cf9a1e93ac46ea3ecd11744761872
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148622"
---
# <a name="soft-delete-for-azure-storage-blobs"></a>Azure 存储 Blob 的软删除
Azure 存储现提供 Blob 对象软删除，目的是为了在应用程序或其他存储帐户用户错误地修改或删除数据后可以更轻松地恢复数据。

## <a name="how-does-it-work"></a>工作原理
启用软删除后，在 blob 或 blob 快照被删除的情况下，可通过此功能保存和恢复数据。 此保护可扩展到因覆盖而擦除的 blob 数据。

被删除的数据会过渡到软删除状态，而非被永久擦除。 如果启用软删除并覆盖数据，则会生成软删除快照以保存被覆盖数据的状态。 除非显式列出，否则软删除对象不可见。 可配置软删除数据永久失效前的保持时间。

软删除向后兼容；无需对应用程序进行任何更改即可利用此功能提供的保护。 但是，[数据恢复](#recovery)会引入新的撤销删除 Blob API。

### <a name="configuration-settings"></a>配置设置
创建新帐户时，默认关闭软删除。 对于现有存储帐户，软删除也默认处于关闭状态。 在存储帐户的生存期内，可随时启用和关闭该功能。

此功能关闭后，你仍然能够访问和恢复软删除数据，前提是之前启用该功能时已保存软删除数据。 如果启用软删除，还需配置保持期。

保持期指示已软删除数据的存储和可恢复时间期限。 对于显式删除的 blob 和 blob 快照，保持期从删除数据时开始计时。 对于覆盖数据时软删除功能生成的软删除快照，保持期从生成快照时开始计时。 当前，软删除数据可保留 1 - 365 天。

可随时更改软删除保持期。 更新后的保持期仅适用于新删除的数据。 之前删除的数据的过期时间取决于删除该数据时配置的保持期。 尝试删除某个软删除的对象不会影响其到期时间。

### <a name="saving-deleted-data"></a>保存已删除的数据
在 blob 或 blob 快照被删除或覆盖的很多情况下，软删除会保存数据。

使用“放置 Blob”、“放置块”、“放置块列表”或“复制 Blob”覆盖 blob 时，将自动生成写入操作前 blob 的状态的快照。 此快照为软删除快照；除非显式列出软删除对象，否则该快照不可见。 请参阅[恢复](#recovery)部分，了解如何列出软删除对象。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-overwrite.png)

软删除数据呈现为灰色，而活动数据为蓝色。新写入的数据显示在旧数据下方。使用 B1 覆盖 B0 时会生成 B0 的软删除快照。使用 B2 覆盖 B1 时会生成 B1 的软删除快照*。

> [!NOTE]  
> 对目标 blob 的帐户启用软删除时，软删除仅对复制操作提供覆盖保护。

> [!NOTE]  
> 软删除不会对存档层中的 blob 提供覆盖保护。 如果存档层中的 blob 被任何层中的新 blob 覆盖，则被覆盖的 blob 将永久过期。

对快照调用“删除 Blob”时，该快照会被标记为软删除。 此时不会生成新的快照。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-explicit-delete-snapshot.png)

*软删除数据呈现为灰色，而活动数据为蓝色。新写入的数据显示在旧数据下方。调用**Snapshot Blob**时，B0 将变为快照，B1 成为该 blob 的活动状态。如果删除 B0 快照，它将被标记为软删除*。

如果对基础 blob（本身不是快照的任何 blob）调用“删除 Blob”，该 blob 将被标记为软删除。 与以前的行为一致，对具有活动快照的 blob 调用“删除 Blob”将返回错误。 对具有软删除快照的 blob 调用“删除 Blob”不会返回错误。 启用软删除后，仍可在单个操作中删除 blob 及其所有快照。 执行该操作会将基础 blob 和快照标记为软删除。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-explicit-include.png)

软删除数据呈现为灰色，而活动数据为蓝色。新写入的数据显示在旧数据下方。此处调用了“删除 Blob”来删除 B2 和所有相关快照 。活动 blob B2 和所有相关快照均被标记为软删除*。

> [!NOTE]  
> 覆盖软删除 blob 时，将自动生成写入操作前 blob 状态的软删除快照。 新 blob 将继承被覆盖 blob 的层级。

如果删除容器或帐户或者覆盖 blob 元数据和 blob 属性，软删除皆不会保存数据。 若要防止存储帐户被错误删除，可以使用 Azure 资源管理器配置锁定。 有关详细信息，请参阅 Azure 资源管理器文章[锁定资源以防止意外更改](../../azure-resource-manager/resource-group-lock-resources.md)。

下表详述了启用软删除后的预期行为：

| REST API 操作 | 资源类型 | 说明 | 行为更改 |
|--------------------|---------------|-------------|--------------------|
| [删除](/rest/api/storagerp/StorageAccounts/Delete) | 帐户 | 删除存储帐户，包括它包含的所有容器和 blob。                           | 无更改。 已删除帐户中的容器和 blob 不可恢复。 |
| [删除容器](/rest/api/storageservices/delete-container) | 容器 | 删除容器，包括它包含的所有 blob。 | 无更改。 已删除容器中的 blob 不可恢复。 |
| [放置 Blob](/rest/api/storageservices/put-blob) | 块 Blob、追加 Blob 和 页 Blob | 创建新的 blob 或替换容器内的现有 blob | 如果用于替换现有 blob，将自动生成调用之前的 blob 状态的快照。 对于以前软删除的 blob，当且仅当使用相同类型的 blob （块、追加或页 blob）进行替换时，才会生成快照。 如果由不同类型的 blob 替换，所有现有软删除数据都将永久过期。 |
| [删除 Blob](/rest/api/storageservices/delete-blob) | 块 Blob、追加 Blob 和 页 Blob | 标记要删除的 blob 或 blob 快照。 blob 或快照将稍后在垃圾回收过程中进行删除 | 如果用于删除 blob 快照，该快照将标记为软删除。 如果用于删除 blob，该 blob 将标记为软删除。 |
| [复制 Blob](/rest/api/storageservices/copy-blob) | 块 Blob、追加 Blob 和 页 Blob | 将源 blob 复制到相同存储帐户或其他存储帐户中的目标 blob 中。 | 如果用于替换现有 blob，将自动生成调用之前的 blob 状态的快照。 对于以前软删除的 blob，当且仅当使用相同类型的 blob （块、追加或页 blob）进行替换时，才会生成快照。 如果由不同类型的 blob 替换，所有现有软删除数据都将永久过期。 |
| [放置块](/rest/api/storageservices/put-block) | 块 Blob | 创建新块，作为块 blob 的一部分进行提交。 | 如果用于将块提交到活动 blob 中，则不发生任何更改。 如果用于将块提交到软删除的 blob 中，将创建新的 blob 并自动生成快照，以捕获软删除 blob 的状态。 |
| [放置块列表](/rest/api/storageservices/put-block-list) | 块 Blob | 通过指定构成块 blob 的块 ID 集来提交 blob。 | 如果用于替换现有 blob，将自动生成调用之前的 blob 状态的快照。 对于以前软删除的 blob，当且仅当其为块 blob 时，才会生成快照。 如果由不同类型的 blob 替换，所有现有软删除数据都将永久过期。 |
| [放置页](/rest/api/storageservices/put-page) | 页 Blob | 将一系列页写入页 Blob。 | 无更改。 通过该操作覆盖或清除的页 Blob 数据不会保存，且不可恢复。 |
| [追加块](/rest/api/storageservices/append-block) | 追加 Blob | 将数据块写入追加 Blob 的末尾。 | 无更改。 |
| [设置 Blob 属性](/rest/api/storageservices/set-blob-properties) | 块 Blob、追加 Blob 和 页 Blob | 为对 blob 定义的系统属性设置值。 | 无更改。 被覆盖的 blob 属性不可恢复。 |
| [设置 Blob 元数据](/rest/api/storageservices/set-blob-metadata) | 块 Blob、追加 Blob 和 页 Blob | 将特定 blob 的用户定义元数据设置为一个或多个名称/值对。 | 无更改。 被覆盖的 blob 元数据不可恢复。 |

请务必注意，通过调用“放置页”来覆盖或清除页 Blob 时不会自动生成快照。 虚拟机磁盘受页 Blob 支持，且使用“放置页”来写入数据。

### <a name="recovery"></a>恢复
为使恢复已删除数据更轻松，我们引入了新的“Undelete Blob”API。 对软删除的基础 blob 调用撤销删除 API 会将该 blob 及所有相关软删除快照还原为活动状态。 对活动基础 blob 调用撤销删除 API 会将所有相关软删除快照还原为活动状态。 快照还原为活动状态后与用户生成的快照相似；这些快照不会覆盖基础 blob。

若要将 blob 还原到特定的软删除快照，可对基础 blob 调用撤销删除 Blob。 然后可将该快照复制到现在处于活动状态的 blob。 也可将该快照复制到新的 blob 中。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-recover.png)

*软删除数据呈现为灰色，而活动数据为蓝色。新写入的数据显示在旧数据下方。此处对 blob B 调用了**撤销删除 Blob**，从而将基础 blob B1 和所有相关快照（此处仅为 B0）还原为活动状态。第二步中将 B0 复制到了基础 blob。此复制操作将生成 B1 的软删除快照*。

若要查看软删除 blob 和 blob 快照，可选择将已删除数据包含在列表 Blob 中。 可选择仅查看软删除的基础 blob，或者也将软删除的 blob 快照包含在内。 对于所有软删除数据，可以查数据删除的时间以及数据永久过期的剩余天数。

### <a name="example"></a>示例
启用软删除后，.NET 脚本对名为“HelloWorld”的 blob 执行上传、覆盖、拍摄快照、删除和还原操作。如下为其 .NET 脚本的控制台输出：

```bash
Upload:
- HelloWorld (is soft deleted: False, is snapshot: False)

Overwrite:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Snapshot:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Delete (including snapshots):
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: False)

Undelete:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Copy a snapshot over the base blob:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)
```

请参阅[后续步骤](#next-steps)部分，了解产生此输出的应用程序的指针。

## <a name="pricing-and-billing"></a>定价和计费
所有软删除数据按与活动数据相同的费率计费。 不会对在配置的保持期后永久删除的数据计费。 若要深入了解快照以及它们产生费用的方式，请参阅[了解快照如何产生费用](storage-blob-snapshots.md)。

不会对快照自动生成相关事务进行计费。 将按“写入操作”费率对“撤销删除 Blob ”事务进行计费。

有关 Azure Blob 存储常规价格的更多详细信息，请参阅 [Azure Blob 存储定价页](https://azure.microsoft.com/pricing/details/storage/blobs/)。

初次启用软删除时，建议使用较短的保持期，以便更好地了解因该功能产生的费用变动。

## <a name="quickstart"></a>快速入门
### <a name="azure-portal"></a>Azure 门户
若要启用软删除，请导航至“Blob 服务”下的“软删除”选项。 然后单击“已启用”并输入要保留软删除数据的天数。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-configuration.png)

若要查看软删除 blob，请选择“显示已删除 blob”复选框。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-view-soft-deleted.png)

若要查看给定 blob 的软删除快照，请选择相应 blob 并单击“查看快照”。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-view-soft-deleted-snapshots.png)

请确保已选择“显示已删除快照”复选框。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-view-soft-deleted-snapshots-check.png)

单击软删除 blob 或快照时，请注意新的 blob 属性。 这些属性指示删除对象的时间，以及 blob 或 blob 快照永久过期前的剩余天数。 如果软删除对象不是快照，还可选择撤销删除。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-properties.png)

请记住，撤销删除 blob 还将撤销删除所有相关快照。 若要撤销删除活动 blob 的软删除快照，请单击相应 blob 并选择“撤销删除所有快照”。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-undelete-all-snapshots.png)

撤销删除 blob 的快照后，可单击“提升”将快照复制到根 blob，从而将 blob 还原到快照。

![](media/storage-blob-soft-delete/storage-blob-soft-delete-portal-promote-snapshot.png)

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

若要启用软删除，请更新 blob 客户端的服务属性。 下面的示例为订阅中的帐户子集启用了软删除：

```powershell
Set-AzContext -Subscription "<subscription-name>"
$MatchingAccounts = Get-AzStorageAccount | where-object{$_.StorageAccountName -match "<matching-regex>"}
$MatchingAccounts | Enable-AzStorageDeleteRetentionPolicy -RetentionDays 7
```
可以使用以下命令验证是否启用了软删除：

```powershell
$MatchingAccounts | Get-AzStorageServiceProperty -ServiceType Blob
```

若要恢复意外删除的 blob，请对这些 blob 调用撤销删除。 请记住，如果对活动和软删除 blob 调用撤销删除 Blob，则会将所有相关软删除快照还原为活动状态。 下面的示例对容器中的所有软删除和活动 blob 调用了撤销删除：
```powershell
# Create a context by specifying storage account name and key
$ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

# Get the blobs in a given container and show their properties
$Blobs = Get-AzStorageBlob -Container $StorageContainerName -Context $ctx -IncludeDeleted
$Blobs.ICloudBlob.Properties

# Undelete the blobs
$Blobs.ICloudBlob.Undelete()
```
若要查找当前的软删除保留策略，请使用以下命令：

```azurepowershell-interactive
   $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context
```

### <a name="azure-cli"></a>Azure CLI 
若要启用软删除，请更新 blob 客户端的服务属性：

```azurecli-interactive
az storage blob service-properties delete-policy update --days-retained 7  --account-name mystorageaccount --enable true
```

若要验证软删除是否已启用，请使用以下命令： 

```azurecli-interactive
az storage blob service-properties delete-policy show --account-name mystorageaccount 
```

### <a name="python-client-library"></a>Python 客户端库
若要启用软删除，请更新 blob 客户端的服务属性：

```python
# Make the requisite imports
from azure.storage.blob import BlockBlobService
from azure.storage.common.models import DeleteRetentionPolicy

# Initialize a block blob service
block_blob_service = BlockBlobService(account_name='<enter your storage account name>', account_key='<enter your storage account key>')

# Set the blob client's service property settings to enable soft delete
block_blob_service.set_blob_service_properties(delete_retention_policy = DeleteRetentionPolicy(enabled = True, days = 7))
```

### <a name="net-client-library"></a>.NET 客户端库
若要启用软删除，请更新 blob 客户端的服务属性：

```csharp
// Get the blob client's service property settings
ServiceProperties serviceProperties = blobClient.GetServiceProperties();

// Configure soft delete
serviceProperties.DeleteRetentionPolicy.Enabled = true;
serviceProperties.DeleteRetentionPolicy.RetentionDays = RetentionDays;

// Set the blob client's service property settings
blobClient.SetServiceProperties(serviceProperties);
```

若要恢复意外删除的 blob，请对这些 blob 调用撤销删除。 请记住，如果对活动和软删除 blob 调用撤销删除 Blob，则会将所有相关软删除快照还原为活动状态。 下面的示例对容器中的所有软删除和活动 blob 调用了撤销删除：

```csharp
// Recover all blobs in a container
foreach (CloudBlob blob in container.ListBlobs(useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Deleted))
{
       await blob.UndeleteAsync();
}
```

若要恢复到特定 blob 版本，请先对 blob 调用撤销删除，然后将所需快照复制到该 blob。 下面的示例将块 blob 恢复到其最新生成的快照：

```csharp
// Undelete
await blockBlob.UndeleteAsync();

// List all blobs and snapshots in the container prefixed by the blob name
IEnumerable<IListBlobItem> allBlobVersions = container.ListBlobs(
    prefix: blockBlob.Name, useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Snapshots);

// Restore the most recently generated snapshot to the active blob    
CloudBlockBlob copySource = allBlobVersions.First(version => ((CloudBlockBlob)version).IsSnapshot &&
    ((CloudBlockBlob)version).Name == blockBlob.Name) as CloudBlockBlob;
blockBlob.StartCopy(copySource);
```

## <a name="should-i-use-soft-delete"></a>是否应该使用软删除？
如果应用程序或其他存储帐户用户可能意外修改或删除数据，则建议启用软删除。 作为数据保护策略的一部分，软删除可防止数据意外丢失。

## <a name="faq"></a>常见问题解答
**是否有使用软删除的任何特殊注意事项？**  
列出 blob 时，经常被覆盖数据启用软删除可能会导致在更高的存储容量费用和增加延迟时间。 可以通过禁用软删除将经常被覆盖的数据存储在单独的存储帐户来缓解此问题。 

**哪种存储类型可以使用软删除？**  
目前，软删除仅适用于 blob（对象）存储。

**软删除是否适用于所有存储帐户类型？**  
是的，软删除适用于 Blob 存储帐户以及常规用途（GPv1 和 GPv2）存储帐户中的 Blob。 这同时适用于标准和高级帐户。 软删除不适用于托管磁盘。

**软删除是否适用于所有存储层？**  
是的，软删除适用于所有存储层，包括热、冷和存档层。 但是，软删除对存档层中的 blob 不提供覆盖保护。

**是否可以使用“设置 Blob 层 API”将 Blob 与软删除的快照置于一层？**  
可以。 软删除的快照会保留在原始层中，但基础 Blob 会移到新层中。 

**高级存储帐户每个 blob 的快照上限为 100.软删除快照是否计入此限制？**  
不，软删除快照不记入此限制。

**是否可以对现有存储帐户启用软删除？**  
可以，对现有和新存储帐户均可配置软删除。

**如果在启用软删除的情况下删除整个帐户或容器，是否会保存所有相关 blob？**  
不会，如果删除整个帐户或容器，将永久删除所有相关 blob。 若要了解如何防止意外删除存储帐户，请参阅 Azure 资源管理器文章[锁定资源以防止意外更改](../../azure-resource-manager/resource-group-lock-resources.md)。

**能否查看已删除数据的容量指标？**  
软删除数据属于存储帐户总容量的一部分。 有关跟踪和监视存储容量的详细信息，请参阅[存储分析](../common/storage-analytics.md)一文。

**如果关闭软删除，是否仍然能够访问软删除数据？**  
可以，关闭软删除后，仍能访问和恢复未过期的软删除数据。

**能否读取和复制 blob 的软删除快照？**  
可以，但必须首先对该 blob 调用撤销删除。

**软删除是否适用于所有 blob 类型？**  
是的，软删除适用于块 Blob、追加 Blob 和页 Blob。

**软删除是否适用于虚拟机磁盘？**  
软删除适用于高级和标准非托管磁盘。 软删除只能帮助恢复由“删除 Blob”、“放置 Blob”、“放置块列表”、“放置块”和“复制 Blob”删除的数据。 通过调用放置页覆盖的数据不可恢复。

**是否需要更改现有应用程序才能使用软删除？**  
无论使用何种 API 版本，均可以使用软删除。 但是，若要列出和恢复软删除 blob 和 blob 快照，则需要使用 2017-07-29 版[存储服务 REST API](https://docs.microsoft.com/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services)或更高版本。 一般情况下，无论是否使用此功能，我们通常均建议使用最新版本。

## <a name="next-steps"></a>后续步骤
* [.NET 示例代码](https://github.com/Azure-Samples/storage-dotnet-blob-soft-delete)
* [Blob 服务 REST API](/rest/api/storageservices/blob-service-rest-api)
* [Azure 存储复制](../common/storage-redundancy.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [使用 RA-GRS 设计高度可用的应用程序](../common/storage-designing-ha-apps-with-ragrs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [Azure 存储中的灾难恢复和存储帐户故障转移（预览版）](../common/storage-disaster-recovery-guidance.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
