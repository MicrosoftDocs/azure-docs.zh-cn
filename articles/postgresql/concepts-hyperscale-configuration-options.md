---
title: Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 性能选项
description: 超大规模 (Citus) 服务器组，包括节点计算、 存储和区域的选项。
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: e8c1c2e51ca14ae9b17f0d7efb20552cdd55139b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65077285"
---
# <a name="azure-database-for-postgresql--hyperscale-citus-preview-performance-options"></a>Azure Database for PostgreSQL-超大规模 (Citus) （预览版） 性能选项

## <a name="compute-and-storage"></a>计算和存储
 
超大规模 (Citus) 的服务器组中，可以选择单独的辅助角色节点的计算和存储设置和协调器节点。  计算资源以 vCore 的形式提供，代表基础硬件的逻辑 CPU。 预配的存储大小是指供你超大规模 (Citus) 的服务器组中的处理协调器和辅助角色节点的容量。 存储包括数据库文件、 临时文件、 事务日志和 Postgres 服务器日志。 此外预配的存储总量为每个辅助角色和协调器节点定义可用的 I/O 容量。
 
|                       | 工作节点           | 协调器节点      |
|-----------------------|-----------------------|-----------------------|
| 计算、 vcore 数       | 4, 8, 16, 32          | 4, 8, 16, 32          |
| 每个 vCore，GiB 内存 | 8                     | 4                     |
| 存储大小 （tib）     | 0.5, 1, 2             | 0.5, 1, 2             |
| 存储类型          | 常规用途 (SSD) | 常规用途 (SSD) |
| IOPS                  | 最多 3 个 IOPS/GiB      | 最多 3 个 IOPS/GiB      |


## <a name="regions"></a>区域
以下 Azure 区域中提供了超大规模 (Citus) 服务器组：
* 美国东部 2
* 东南亚
* 西欧
* 美国西部 2

## <a name="pricing"></a>定价
有关最新定价信息，请参阅服务的[定价页](https://azure.microsoft.com/pricing/details/postgresql/)。
若要查看所需的配置的成本[Azure 门户](https://portal.azure.com/#create/Microsoft.PostgreSQLServer)上会显示每月成本**配置**选项卡基于您选择的选项。 如果没有 Azure 订阅，可使用 Azure 定价计算器获取估计的价格。 上[Azure 定价计算器](https://azure.microsoft.com/pricing/calculator/)网站上，选择**添加项**，展开**数据库**类别，然后选择**Azure Database for PostgreSQL-超大规模 (Citus)** 自定义选项。
 
## <a name="next-steps"></a>后续步骤
了解如何[在门户中创建一个超大规模 (Citus) 服务器组](quickstart-create-hyperscale-portal.md)。
