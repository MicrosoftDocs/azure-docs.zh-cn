---
title: 删除 VNet 后删除 Azure SQL 数据库托管实例 |Microsoft Docs
description: 了解如何删除 VNet 后删除 Azure SQL 数据库托管实例。
services: sql-database
ms.service: sql-database
ms.subservice: management
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: douglas, carlrab, sstein
manager: craigg
ms.date: 05/07/2019
ms.openlocfilehash: 95d1681c9ff9981990d873a58a2d01833d690e0f
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411981"
---
# <a name="delete-subnet-after-deleting-azure-sql-database-managed-instance"></a>删除子网后删除 Azure SQL 数据库托管实例

本文提供有关如何删除最后一个 Azure SQL 数据库托管实例驻留在其中后，手动删除子网的指导原则。

[虚拟群集](sql-database-managed-instance-connectivity-architecture.md#virtual-cluster-connectivity-architecture)，具有包含已删除托管的实例将保留 12 个小时从实例删除。 虚拟群集保持活动状态，通过设计，以启用更快地创建托管实例位于同一子网。 在此期间，无法删除与虚拟群集相关联的子网。

通过手动删除虚拟群集的可能是空的虚拟群集使用的子网的即时发布。 可以通过 Azure 门户或虚拟群集 API 实现的虚拟群集删除。

> [!NOTE]
> 虚拟群集应包含不到为才能成功删除托管的实例。

## <a name="delete-virtual-cluster-from-azure-portal"></a>Azure 门户中删除虚拟群集

若要删除虚拟群集使用 Azure 门户，请搜索使用内置搜索的虚拟群集资源。

![搜索虚拟群集。](./media/sql-database-managed-instance-delete-virtual-cluster/virtual-clusters-search.png)

后查找要删除的虚拟群集，请选择此资源，并选择删除选项。 系统将提示您确认要虚拟群集删除。

![删除虚拟群集。](./media/sql-database-managed-instance-delete-virtual-cluster/virtual-clusters-delete.png)

在 Azure 门户通知中提供虚拟群集已删除的确认。 成功删除虚拟群集立即释放以备今后重复使用的子网。

## <a name="delete-virtual-cluster-using-api"></a>删除使用 API 的虚拟群集

若要删除的虚拟群集通过 API 使用 URI 参数中指定[虚拟群集删除方法](https://docs.microsoft.com/rest/api/sql/virtualclusters/delete)。

## <a name="next-steps"></a>后续步骤

- 有关概述，请参阅[什么是托管实例？](sql-database-managed-instance.md)。
- 了解[托管实例中的连接体系结构](sql-database-managed-instance-connectivity-architecture.md)。
- 了解如何[修改托管实例的现有虚拟网络](sql-database-managed-instance-configure-vnet-subnet.md)
- 有关如何创建虚拟网络、创建托管实例，以及从数据库备份还原数据库的教程，请参阅[创建 Azure SQL 数据库托管实例](sql-database-managed-instance-get-started.md)。
- 有关 DNS 问题，请参阅[配置自定义 DNS](sql-database-managed-instance-custom-dns.md)。
