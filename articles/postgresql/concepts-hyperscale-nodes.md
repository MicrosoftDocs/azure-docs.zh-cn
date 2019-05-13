---
title: Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 中的节点
description: 两种类型的服务器组中的节点。
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: c6b948ed63f43f1597103d123be5ed39f42bd276
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077270"
---
# <a name="nodes-in-azure-database-for-postgresql--hyperscale-citus-preview"></a>Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 中的节点

托管类型超大规模 (Citus) （预览版） 允许 Azure Database for PostgreSQL 服务器 （称为节点） 来协调与另一个在"无共享"体系结构中。 服务器组中的节点统称为容纳更多数据，并使用更多 CPU 核心不是可以在单个服务器上。 该体系结构还允许要缩放通过将更多节点添加到的服务器组的数据库。

## <a name="coordinator-and-workers"></a>处理协调器和辅助角色

每个服务器组包含协调器节点以及多个辅助角色。 应用程序发送到处理协调器节点，以便将其传递到相关的工作人员并累积其结果的查询。 应用程序不能直接连接到辅助角色。

对于每个查询，协调器将其路由到单个工作节点，或跨多个具体取决于所需的数据是否位于单个节点或多个并行。 协调器决定要执行的操作方法是查看元数据的表。 这些表跟踪在节点之间的 DNS 名称和辅助角色节点的运行状况和数据的分布情况。

## <a name="next-steps"></a>后续步骤
- 了解如何存储节点[分布式数据](concepts-hyperscale-distributed-data.md)
