---
title: 将非分区 Azure Cosmos DB 容器迁移到已分区容器
description: 了解如何将所有现有非分区容器迁移到已分区容器。
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: e8aaee8cb7df794652b2c560c5250921c1a4e060
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160630"
---
# <a name="migrate-non-partitioned-containers-to-partitioned-containers"></a>将非分区容器迁移到已分区容器

Azure Cosmos DB 支持创建容器没有分区键。 目前可以通过使用 Azure CLI 和具有的版本低于或等于到 2.x 的 Azure Cosmos DB Sdk (.Net、 Java、 NodeJs) 创建未分区的容器。 无法创建未分区的容器，使用 Azure 门户。 但是，此类非分区容器不是弹性，并且具有固定的存储容量为 10 GB 和吞吐量限制为 10 个 K RU/秒。

旧的非分区容器，并且应将你现有的非分区容器迁移到已分区容器到缩放存储和吞吐量。 Azure Cosmos DB 提供一种将非分区容器迁移到已分区容器的系统定义机制。 本文档介绍如何自动迁移到已分区容器的所有现有非分区容器。 仅当使用中的所有语言的 Sdk 的 V3 版本，可以充分利用自动迁移功能。

> [!NOTE] 
> 目前，不能使用本文档中所述的步骤来迁移 Azure Cosmos DB MongoDB api 和 Gremlin API 帐户。 

## <a name="migrate-container-using-the-system-defined-partition-key"></a>迁移容器使用系统定义的分区键

若要支持迁移，Azure Cosmos DB 定义名为的系统定义分区键`/_partitionkey`没有分区键的所有容器。 迁移容器后，无法更改分区键定义。 例如，迁移到已分区容器的容器的定义将如下所示： 

```json
{
    "Id": "CollId" 
  "partitionKey": {
    "paths": [
      "/_partitionKey"
    ],
    "kind": "Hash"
  },
}
```
 
迁移容器后，可以通过填充创建文档`_partitionKey`属性以及文档的其他属性。 `_partitionKey`属性表示的分区键的文档。 

选择适当的分区键很重要，若要以最佳方式利用预配的吞吐量。 有关详细信息，请参阅[如何选择分区键](partitioning-overview.md)一文。 

> [!NOTE]
> 仅当使用 Sdk 的所有语言中的最新/V3 版本，可以充分利用系统定义的分区键。

下面的示例演示创建包含的系统定义的分区键的文档并读取该文档的示例代码：

**文档的 JSON 表示形式**

```csharp
DeviceInformationItem = new DeviceInformationItem
{
   "id": "elevator/PugetSound/Building44/Floor1/1",
   "deviceId": "3cf4c52d-cc67-4bb8-b02f-f6185007a808",
   "_partitionKey": "3cf4c52d-cc67-4bb8-b02f-f6185007a808"
} 

public class DeviceInformationItem
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }

    [JsonProperty(PropertyName = "deviceId")]
    public string DeviceId { get; set; }

    [JsonProperty(PropertyName = "_partitionKey")]
    public string PartitionKey {get {return this.DeviceId; set; }
}

CosmosContainer migratedContainer = database.Containers["testContainer"];

DeviceInformationItem deviceItem = new DeviceInformationItem() {
  Id = "1234", 
  DeviceId = "3cf4c52d-cc67-4bb8-b02f-f6185007a808"
} 

CosmosItemResponse<DeviceInformationItem > response = 
  await migratedContainer.Items.CreateItemAsync(
    deviceItem.PartitionKey, 
    deviceItem
  );

// Read back the document providing the same partition key
CosmosItemResponse<DeviceInformationItem> readResponse = 
  await migratedContainer.Items.ReadItemAsync<DeviceInformationItem>( 
    partitionKey:deviceItem.PartitionKey, 
    id: device.Id
  ); 

```

有关完整示例，请参阅[.Net 示例](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/CodeSamples)GitHub 存储库。 
                      
## <a name="migrate-the-documents"></a>迁移文档

尽管容器定义的分区键属性得到了增强，在容器的文档将不自动迁移。 这意味着系统分区键属性`/_partitionKey`路径不自动添加到现有的文档。 您需要对现有文档读取不使用分区键创建的文档进行重新分区和重写它们返回与`_partitionKey`的文档中的属性。 

## <a name="access-documents-that-dont-have-a-partition-key"></a>没有分区密钥的访问文档

应用程序可以访问的情况下使用名为"CosmosContainerSettings.NonePartitionKeyValue"的特殊的系统属性中没有分区键的现有文档，这是非迁移文档的值。 在所有 CRUD 和查询操作中，可以使用此属性。 下面的示例演示一个示例，用于从 NonePartitionKey 读取单个文档。 

```csharp
CosmosItemResponse<DeviceInformationItem> readResponse = 
await migratedContainer.Items.ReadItemAsync<DeviceInformationItem>( 
  partitionKey: CosmosContainerSettings.NonePartitionKeyValue, 
  id: device.Id
); 

```

有关如何对文档进行重新分区的完整示例，请参阅[.Net 示例](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/CodeSamples)GitHub 存储库。 

## <a name="compatibility-with-sdks"></a>使用 Sdk 的兼容性

较旧版本的 Azure Cosmos DB Sdk 如 V2.x.x 和 V1.x.x 不支持系统定义的分区键属性。 因此，当从较旧的 SDK 读取容器定义，它不包含任何分区键定义和这些容器的行为将与之前完全相同。 使用 Sdk 的较旧版本生成的应用程序继续使用非分区按原样而不进行任何更改。 

如果已迁移的容器使用的最新/V3 版本 SDK，并且开始填充新的文档中的系统定义分区键，您不能访问 （读取、 更新、 删除、 查询） 此类文档从较旧的 Sdk 不再。

## <a name="next-steps"></a>后续步骤

* [Azure Cosmos DB 中的分区](partitioning-overview.md)
* [Azure Cosmos DB 中的请求单位](request-units.md)
* [在容器和数据库上预配吞吐量](set-throughput.md)
* [使用 Azure Cosmos 帐户](account-overview.md)