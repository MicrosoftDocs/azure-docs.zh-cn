---
title: 适用于 Azure Cosmos DB 的 azure 资源管理器模板
description: 使用 Azure 资源管理器模板来创建和配置 Azure Cosmos DB。
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: c907de019b64a6a9d03206514a1147fd43c78106
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65078455"
---
# <a name="azure-resource-manager-templates-for-azure-cosmos-db"></a>适用于 Azure Cosmos DB 的 azure 资源管理器模板

下表包含 Azure Cosmos DB 的 Azure 资源管理器模板链接：

|**API 类型** | **链接到示例**| **说明** |
|---|---| ---|
|核心 (SQL) API| [创建 Azure Cosmos DB 帐户 （多主机）](manage-sql-with-resource-manager.md) | 此模板创建 SQL API 帐户在两个区域通过多主启用。 Azure Cosmos 帐户将具有两个容器的共享数据库级别的吞吐量。 |
| MongoDB API | [创建 Azure Cosmos DB 帐户 （多主机）](manage-mongodb-with-resource-manager.md) | 此模板创建 Azure Cosmos DB API 在两个区域的 mongodb 与启用多主机一起使用的帐户。 Azure Cosmos 帐户将具有两个容器的共享数据库级别的吞吐量。 |
| Cassandra API | [创建 Azure Cosmos DB 帐户 （多主机）](manage-cassandra-with-resource-manager.md) | 此模板中的两个区域的 Cassandra API 帐户创建与启用多主机。 Azure Cosmos 帐户将具有两个表的共享密钥空间级别的吞吐量。 |
| Gremlin API| [创建 Azure Cosmos DB 帐户 （多主机）](manage-gremlin-with-resource-manager.md) | 此模板创建 Gremlin API 帐户在两个区域通过多主启用。 Azure Cosmos 帐户将具有两个共享数据库级别的吞吐量的关系图。 |
| 表 API | [创建 Azure Cosmos DB 帐户 （多主机）](manage-table-with-resource-manager.md) | 此模板创建的表 API 帐户在两个区域通过多主启用。 Azure Cosmos 帐户将具有单个表。 |

> [!TIP]
> 若要使用表 API 时，请启用共享的吞吐量，请启用 Azure 门户中的帐户级吞吐量。

请参阅[适用于 Azure Cosmos DB 的 ARM 引用](/azure/templates/microsoft.documentdb/allversions)的参考文档的页。