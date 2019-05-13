---
title: 管理适用于 PostgreSQL 的 Azure 门户中的单个服务器的 Azure 数据库的只读的副本
description: 了解如何管理只读的副本 Azure Database for PostgreSQL-从 Azure 门户的单个服务器。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 87371f91d9ea1f556d0f78beebd73b8a28977b71
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510396"
---
# <a name="create-and-manage-read-replicas-in-azure-database-for-postgresql---single-server-from-the-azure-portal"></a>创建和管理用于 PostgreSQL 的 Azure 门户中的单个服务器的 Azure 数据库中的只读的副本

本文介绍如何使用 Azure 门户在 Azure Database for PostgreSQL 中创建和管理只读副本。 若要详细了解只读副本，请参阅[概述](concepts-read-replicas.md)。

> [!IMPORTANT]
> 在与主服务器，在同一区域或所选的任何其他 Azure 区域中，可以创建只读的副本。 跨区域复制当前处于公共预览状态。


## <a name="prerequisites"></a>必备组件
用作主服务器的 [Azure Database for PostgreSQL 服务器](quickstart-create-server-database-portal.md)。

## <a name="prepare-the-master-server"></a>准备主服务器
必须使用以下步骤在“常规用途”和“内存优化”层中准备主服务器。 主服务器通过 azure.replication_support 参数设置为复制准备。 复制参数更改时，重新启动服务器是必需的更改才能生效。 在 Azure 门户中，这两个步骤都封装由一个按钮**启用复制支持**。

1. 在 Azure 门户中，选择用作主服务器的现有 Azure Database for PostgreSQL 服务器。

2. 在服务器边栏中下,**设置**，选择**复制**。

3. 选择**启用复制支持**。 

   ![启用复制的支持](./media/howto-read-replicas-portal/enable-replication-support.png)

4. 确认你想要启用复制的支持。 此操作将重新启动主服务器。 

   ![确认启用复制支持](./media/howto-read-replicas-portal/confirm-enable-replication.png)
   
5. 一旦完成该操作后，您将收到两个 Azure 门户通知。 没有用于更新服务器参数为一个通知。 没有立即遵循的服务器重新启动的另一条通知。

   ![成功通知-启用](./media/howto-read-replicas-portal/success-notifications-enable.png)

6. 刷新 Azure 门户页以更新复制工具栏。 现在可以创建为此服务器的只读的副本。

   ![已更新的工具栏](./media/howto-read-replicas-portal/updated-toolbar.png)
   
启用复制的支持是每个主服务器是一次性操作。 一个**禁用复制支持**按钮提供为方便起见。 我们不建议禁用复制支持，除非你确信您永远不会将此主服务器上创建副本。 无法在主服务器具有现有的副本时禁用复制支持。


## <a name="create-a-read-replica"></a>创建只读副本
若要创建只读副本，请遵循以下步骤：

1. 选择用作主服务器的现有 Azure Database for PostgreSQL 服务器。 

2. 在服务器边栏中下,**设置**，选择**复制**。

3. 选择“添加副本”。

   ![添加副本](./media/howto-read-replicas-portal/add-replica.png)

4. 输入只读副本的名称。 

    ![为副本命名](./media/howto-read-replicas-portal/name-replica.png)

5. 选择该副本的位置。 可以在任何 Azure 区域中创建的副本。 默认位置是主服务器相同。

    ![选择位置](./media/howto-read-replicas-portal/location-replica.png)

6. 选择“确定”以确认创建该副本。

副本是使用与主服务器相同的服务器配置创建的。 创建副本后，可以独立于主服务器更改多项设置：计算代系、vCore 数、存储和备份保留期。 定价层也可以独立更改，但“基本”层除外。

> [!IMPORTANT]
> 将主服务器的配置更新为新值之前，请将副本配置更新为与这些新值相等或更大的值。 此操作可确保副本与主服务器发生的任何更改保持同步。

创建只读副本后，可以在“复制”窗口中查看它：

![在“复制”窗口中查看新副本](./media/howto-read-replicas-portal/list-replica.png)
 

## <a name="stop-replication"></a>停止复制
可以停止主服务器与只读副本之间的复制。

> [!IMPORTANT]
> 停止复制到主服务器和只读副本后，无法撤消该操作。 只读副本将成为支持读取和写入的独立服务器。 独立服务器不能再次成为副本。

若要在 Azure 门户中停止主服务器与只读副本之间的复制，请遵循以下步骤：

1. 在 Azure 门户中，选择 Azure Database for PostgreSQL 主服务器。

2. 在服务器菜单中的“设置”下，选择“复制”。

3. 选择要停止复制的副本服务器。

   ![选择副本](./media/howto-read-replicas-portal/select-replica.png)
 
4. 选择“停止复制”。

   ![选择停止复制](./media/howto-read-replicas-portal/select-stop-replication.png)
 
5. 选择“确定”以停止复制。

   ![确认停止复制](./media/howto-read-replicas-portal/confirm-stop-replication.png)
 

## <a name="delete-a-master-server"></a>删除主服务器
若要删除主服务器，请遵循删除独立 Azure Database for PostgreSQL 服务器的相同步骤。 

> [!IMPORTANT]
> 删除主服务器后，将停止复制到所有只读副本。 只读副本将成为支持读取和写入的独立服务器。

若要在 Azure 门户中删除服务器，请遵循以下步骤：

1. 在 Azure 门户中，选择 Azure Database for PostgreSQL 主服务器。

2. 此时会打开该服务器的“概述”页。 选择“删除”。

   ![在服务器的“概述”页上，选择删除主服务器](./media/howto-read-replicas-portal/delete-server.png)
 
3. 输入要删除的主服务器的名称。 选择“删除”以确认删除主服务器。

   ![确认删除主服务器](./media/howto-read-replicas-portal/confirm-delete.png)
 

## <a name="delete-a-replica"></a>删除副本
可以像删除主服务器一样删除只读副本。

- 在 Azure 门户中，打开只读副本的“概述”页。 选择“删除”。

   ![在副本的“概述”页上，选择删除该副本](./media/howto-read-replicas-portal/delete-replica.png)
 
也可以在“复制”窗口中遵循以下步骤删除只读副本：

1. 在 Azure 门户中，选择 Azure Database for PostgreSQL 主服务器。

2. 在服务器菜单中的“设置”下，选择“复制”。

3. 选择要删除的只读副本。

   ![选择要删除的副本](./media/howto-read-replicas-portal/select-replica.png)
 
4. 选择“删除副本”。

   ![选择“删除副本”](./media/howto-read-replicas-portal/select-delete-replica.png)
 
5. 输入要删除的副本的名称。 选择“删除”以确认删除该副本。

   ![确认删除副本](./media/howto-read-replicas-portal/confirm-delete-replica.png)
 

## <a name="monitor-a-replica"></a>监视副本
可以使用两个指标来监视只读副本。

### <a name="max-lag-across-replicas-metric"></a>“副本的最大滞后时间”指标
“副本的最大滞后时间”指标显示主服务器与滞后时间最长的副本之间的滞后时间（以字节为单位）。 

1.  在 Azure 门户中，选择 Azure Database for PostgreSQL 主服务器。

2.  选择“指标”。 在“指标”窗口中，选择“副本的最大滞后时间”。

    ![监视副本的最大滞后时间](./media/howto-read-replicas-portal/select-max-lag.png)
 
3.  对于“聚合”，请选择“最大”。


### <a name="replica-lag-metric"></a>“副本滞后时间”指标
“副本滞后时间”指标显示自从在副本上最后一次执行重放事务以来所经历的时间。 如果主服务器上未发生任何事务，则该指标会反映此滞后时间。

1. 在 Azure 门户中，选择 Azure Database for PostgreSQL 只读副本。

2. 选择“指标”。 在“指标”窗口中，选择“副本滞后时间”。

   ![监视副本滞后时间](./media/howto-read-replicas-portal/select-replica-lag.png)
 
3. 对于“聚合”，请选择“最大”。 
 
## <a name="next-steps"></a>后续步骤
详细了解 [Azure Database for PostgreSQL 中的只读副本](concepts-read-replicas.md)。