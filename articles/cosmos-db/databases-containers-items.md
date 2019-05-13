---
title: 使用 Azure Cosmos DB 数据库、容器和项
description: 本文介绍如何创建和使用 Azure Cosmos DB 数据库、容器和项
author: rimman
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: rimman
ms.reviewer: sngun
ms.openlocfilehash: 215ae29c571e56b7e3af589149123083223023c7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153457"
---
# <a name="work-with-databases-containers-and-items"></a>使用数据库、容器和项

在 Azure 订阅下创建 [Azure Cosmos 帐户](account-overview.md)后，可以通过创建数据库、容器和项来管理帐户中的数据。 本文介绍以下每个实体：数据库、容器和项。 下图显示 Azure Cosmos 帐户中不同实体的层次结构：

![Azure Cosmos 帐户实体](./media/databases-containers-items/cosmos-entities.png)

## <a name="azure-cosmos-databases"></a>Azure Cosmos 数据库

可以在帐户下创建一个或多个 Azure Cosmos 数据库。 数据库类似于命名空间， 它是一组 Azure Cosmos 容器的管理单元。 下表显示如何将 Azure Cosmos 数据库映射到各种特定于 API 的实体：

| **Azure Cosmos 实体** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- |
|Azure Cosmos 数据库 | 数据库 | 密钥空间 | 数据库 | 数据库 | NA |

> [!NOTE]
> 使用表 API 帐户，会在创建第一个表时自动在 Azure Cosmos 帐户中创建默认数据库。

### <a name="operations-on-an-azure-cosmos-database"></a>对 Azure Cosmos 数据库执行的操作

可以使用 Azure Cosmos API 与 Azure Cosmos 数据库进行交互，如下所示：

| **操作** | **Azure CLI**|**SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- | --- |
|枚举所有数据库| 是 | 是 | 是（将数据库映射至密钥空间） | 是 | NA | NA |
|读取数据库| 是 | 是 | 是（将数据库映射至密钥空间） | 是 | NA | NA |
|创建新的数据库| 是 | 是 | 是（将数据库映射至密钥空间） | 是 | NA | NA |
|更新数据库| 是 | 是 | 是（将数据库映射至密钥空间） | 是 | NA | NA |


## <a name="azure-cosmos-containers"></a>Azure Cosmos 容器

Azure Cosmos 容器是预配的吞吐量和存储的缩放单元。 容器会进行水平分区，然后在多个区域间复制。 添加到容器的项以及针对容器预配的吞吐量将基于分区键自动分配给一组逻辑分区。 若要详细了解分区和分区键，请参阅[此](partition-data.md)文。 

在创建 Azure Cosmos 容器时，会在以下某个模式中配置吞吐量：

* **专用预配吞吐量**模式：针对容器预配的吞吐量是专门为该容器保留的，由 SLA 提供支持。 有关详细信息，请参阅[如何对 Azure Cosmos 容器预配吞吐量](how-to-provision-container-throughput.md)。

* **共享预配吞吐量**模式：这些容器与同一数据库中的其他容器共享预配吞吐量（不包含已配置专用预配吞吐量的容器）。 换而言之，在"共享的吞吐量"的所有容器之间共享上，对数据库的预配的吞吐量。 有关详细信息，请参阅[如何对 Azure Cosmos 数据库配置预配吞吐量](how-to-provision-database-throughput.md)。

无论使用“共享”还是“专用”预配吞吐量模式创建容器，Azure Cosmos 容器都可以灵活缩放。

Azure Cosmos 容器是与架构无关的项容器。 容器中的项可以具备任意架构。 例如，如果容器中有表示人员的项，那么该容器也能包含表示汽车的项。 默认情况下，添加到容器的所有项都会自动编入索引，不需要任何显式索引或架构管理工作。 通过在容器上配置的[索引策略](index-overview.md)，可以自定义索引行为。 

可以针对 Azure Cosmos 容器中的所选项或整个容器设置[生存时间 (TTL)](time-to-live.md)，适当地从系统清除这些项。 Azure Cosmos DB 会在这些项过期时自动将其删除。 这样还能保证对这些容器执行的查询不会返回固定边界内已过期的项。 若要了解详细信息，请参阅[如何对容器配置 TTL](how-to-time-to-live.md)。

通过使用[更改源](change-feed.md)，可以订阅针对容器的每个逻辑分区管理的操作日志。 更改源提供容器上执行的所有更新的日志，以及更新前和更新后的项的映像。 请参阅[如何使用更改源生成响应式应用程序](serverless-computing-database.md)。 还可以通过使用容器上的更改源策略来配置更改源的保留期限。 

使用 Azure Cosmos 容器可以注册[存储过程、触发器、用户定义的函数 (UDF)](stored-procedures-triggers-udfs.md) 和[合并过程](how-to-manage-conflicts.md)。 

可以在 Azure Cosmos 容器上指定一个[唯一键约束](unique-keys.md)。 通过创建唯一键策略，可确保每个逻辑分区键的一个或多个值的唯一性。 使用唯一键策略创建容器后，它可以避免创建值与唯一键约束指定的值重复的任何新的或更新项。 若要了解详细信息，请参阅[唯一键约束](unique-keys.md)。

Azure Cosmos 容器专用于特定于 API 的实体，如下所示：

| **Azure Cosmos 实体** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- |
|Azure Cosmos 容器 | 集合 | 表 | 集合 | 图形 | 表 |

### <a name="properties-of-an-azure-cosmos-container"></a>Azure Cosmos 容器的属性

Azure Cosmos 容器具备一组系统定义的属性。 根据所选的 API，其中一些属性可能不会直接公开。 下表介绍了系统定义属性的列表：

| **系统定义的属性** | **是系统生成的还是可由用户配置的** | **用途** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- | --- | --- |
|_id | 系统生成的 | 容器的唯一标识符 | 是 | 否 | 否 | 否 | 否 |
|_etag | 系统生成的 | 用于乐观并发控制的实体标记 | 是 | 否 | 否 | 否 | 否 |
|_ts | 系统生成的 | 容器上次更新的时间戳 | 是 | 否 | 否 | 否 | 否 |
|_self | 系统生成的 | 容器的可寻址 URI | 是 | 否 | 否 | 否 | 否 |
|id | 用户可配置 | 用户定义的容器唯一名称 | 是 | 是 | 是 | 是 | 是 |
|indexingPolicy | 用户可配置 | 提供更改索引路径、索引类型和索引模型的功能。 | 是 | 否 | 否 | 否 | 是 |
|TimeToLive | 用户可配置 | 提供在一段时间以后从容器自动删除项的功能。 有关详细信息，请参阅[生存时间](time-to-live.md)一文。 | 是 | 否 | 否 | 否 | 是 |
|changeFeedPolicy | 用户可配置 | 用于读取对容器中的项所做的更改。 有关详细信息，请参阅[更改源](change-feed.md)一文。 | 是 | 否 | 否 | 否 | 是 |
|uniqueKeyPolicy | 用户可配置 | 用于确保逻辑分区内一个或多个值的唯一性。 有关详细信息，请参阅[唯一键约束](unique-keys.md)一文。 | 是 | 否 | 否 | 否 | 是 |

### <a name="operations-on-an-azure-cosmos-container"></a>对 Azure Cosmos 容器执行的操作

Azure Cosmos 容器支持使用任何 Azure Cosmos API 执行的以下操作。

| **操作** | **Azure CLI** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- | --- |
| 枚举数据库中的容器 | 是 | 是 | 是 | 是 | NA | NA |
| 读取容器 | 是 | 是 | 是 | 是 | NA | NA |
| 新建容器 | 是 | 是 | 是 | 是 | NA | NA |
| 更新容器 | 是 | 是 | 是 | 是 | NA | NA |
| 删除容器 | 是 | 是 | 是 | 是 | NA | NA |

## <a name="azure-cosmos-items"></a>Azure Cosmos 项

根据所选的 API，Azure Cosmos 项可以表示集合中的一个文件、表格中的一行或者图形中的一个节点/边缘。 下表显示特定于 API 的实体和 Azure Cosmos 项之间的映射：

| **Cosmos 实体** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- |
|Azure Cosmos 项 | 文档 | 行 | 文档 | 节点或边缘 | Item |

### <a name="properties-of-an-item"></a>项的属性

每个 Azure Cosmos 项都具有以下系统定义的属性。 根据所选的 API，其中一些属性可能不会直接公开。

|**系统定义的属性** | **是系统生成的还是可由用户配置的**| **用途** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- | --- | --- |
|_id | 系统生成的 | 项的唯一标识符 | 是 | 否 | 否 | 否 | 否 |
|_etag | 系统生成的 | 用于乐观并发控制的实体标记 | 是 | 否 | 否 | 否 | 否 |
|_ts | 系统生成的 | 项上次更新的时间戳 | 是 | 否 | 否 | 否 | 否 |
|_self | 系统生成的 | 项的可寻址 URI | 是 | 否 | 否 | 否 | 否 |
|id | 任一个 | 逻辑分区内用户定义的唯一名称。 如果用户未指定 ID，系统将自动生成一个。 | 是 | 是 | 是 | 是 | 是 |
|任意用户定义的属性 | 用户定义 | 用户定义的属性以 API 本机表示形式表示（JSON、BSON、CQL 等等） | 是 | 是 | 是 | 是 | 是 |

### <a name="operations-on-items"></a>对项执行的操作

Azure Cosmos 项支持可使用任何 Azure Cosmos API 执行的以下操作。

| **操作** | **Azure CLI** | **SQL API** | **Cassandra API** | **Azure Cosmos DB 的 API for MongoDB** | **Gremlin API** | **表 API** |
| --- | --- | --- | --- | --- | --- | --- |
| 插入、替换、删除、Upsert、读取 | 否 | 是 | 是 | 是 | 是 | 是 |

## <a name="next-steps"></a>后续步骤

现在可以了解以下概念：

* [如何对 Azure Cosmos 数据库配置预配吞吐量](how-to-provision-database-throughput.md)
* [如何对 Azure Cosmos 容器配置预配吞吐量](how-to-provision-container-throughput.md)
* [逻辑分区](partition-data.md)
* [如何对 Azure Cosmos 容器配置 TTL](how-to-time-to-live.md)
* [如何使用更改源生成响应式应用程序](change-feed.md)
* [如何对 Azure Cosmos 容器配置唯一键约束](unique-keys.md)
