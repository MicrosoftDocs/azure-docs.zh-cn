---
title: 具有使用 Azure 门户和各种 Sdk 的较大的分区键创建 Azure Cosmos 容器。
description: 了解如何使用大分区键，使用 Azure 门户和不同的 Sdk 创建 Azure Cosmos DB 中的容器。
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: 46df484303237722f4eb66099748f2fcef8240b4
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205835"
---
# <a name="create-containers-with-large-partition-key"></a>创建具有大型分区密钥容器

Azure Cosmos DB 使用基于哈希的分区方案来实现数据的水平缩放。 在 3 个 2019 年 5 之前创建的所有 Azure Cosmos 容器都使用哈希函数计算基于分区键的前 100 个字节的哈希值。 如果有多个具有相同的前 100 个字节的分区键，然后这些逻辑分区被视为作为同一逻辑分区服务。 这可能导致问题，例如分区大小配额正在不正确，并应用于分区键的唯一索引。 引入了大分区键，若要解决此问题。 Azure Cosmos DB 现在支持大型分区键与值的最大为 2 KB。 

通过使用哈希函数，可以从大型分区生成唯一的哈希的增强版本的功能支持密钥的大分区键最多为 2 KB。 此哈希版本，还建议方案与高分区键基数而不考虑分区键的大小。 分区键基数被指唯一逻辑分区，例如按顺序在容器中的 ~ 30000 逻辑分区数。 本文介绍如何通过使用 Azure 门户和不同的 Sdk 的较大的分区键创建容器。 

## <a name="create-a-large-partition-key-net-sdk-v2"></a>创建一个大分区键 (.Net SDK V2)

使用.Net SDK 使用大型分区键创建容器，应指定`PartitionKeyDefinitionVersion.V2`属性。 下面的示例演示如何指定 PartitionKeyDefinition 对象中的版本属性并将其设置为 PartitionKeyDefinitionVersion.V2:

```csharp
DocumentCollection collection = await newClient.CreateDocumentCollectionAsync(
database,
     new DocumentCollection
        {
           Id = Guid.NewGuid().ToString(),
           PartitionKey = new PartitionKeyDefinition
           {
             Paths = new Collection<string> {"/longpartitionkey" },
             Version = PartitionKeyDefinitionVersion.V2
           }
         },
      new RequestOptions { OfferThroughput = 400 });
```

## <a name="create-a-large-partition-key-azure-portal"></a>创建一个大分区键 （Azure 门户） 

若要创建一个大分区键，在创建新的容器使用 Azure 门户时，检查**我的分区键是大于 100 字节**选项。 默认情况下，选择所需的所有新容器是使用较大的分区键。 如果你不需要大分区键，或早 1.18 于 Sdk 版本上运行的应用程序，请取消选中该复选框。

![创建使用 Azure 门户的大分区键](./media/large-partition-keys/large-partition-key-with-portal.png)


## <a name="supported-sdk-versions"></a>支持的 SDK 版本

带以下最低版本的 Sdk 支持大的分区键：

|SDK 类型  | 最低版本   |
|---------|---------|
|.Net     |    1.18     |
|Java 同步     |   2.4.0      |
|Java Async   |  2.5.0        |
 
## <a name="next-steps"></a>后续步骤

* [Azure Cosmos DB 中的分区](partitioning-overview.md)
* [Azure Cosmos DB 中的请求单位](request-units.md)
* [在容器和数据库上预配吞吐量](set-throughput.md)
* [使用 Azure Cosmos 帐户](account-overview.md)


