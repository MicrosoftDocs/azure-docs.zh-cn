---
title: Azure Database for MySQL 中的只读副本。
description: 本文介绍 Azure Database for MySQL 的只读副本。
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 04/30/2019
ms.openlocfilehash: be592cb6bb7c041fab0a2f96a338f4f4bb0ff00a
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510929"
---
# <a name="read-replicas-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的只读副本

读取的副本功能，可将数据从 Azure Database for MySQL 服务器复制到只读的服务器。 可以从主服务器复制到最多五个副本。 使用 MySQL 引擎的本机二进制日志 (binlog) 文件基于位置的复制技术以异步方式更新副本。 若要了解有关 binlog 复制的详细信息，请参阅 [MySQL binlog 复制概述](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html)。

> [!IMPORTANT]
> 在与主服务器，在同一区域或所选的任何其他 Azure 区域中，可以创建只读的副本。 跨区域复制当前处于公共预览状态。

副本是管理类似于常规的 Azure Database for MySQL 服务器的新服务器。 每个只读副本按照预配计算资源的 vCore 数量以及每月 GB 存储量计费。

如需了解有关 MySQL 复制功能和问题的详细信息，请参阅 [MySQL 复制文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html)。

## <a name="when-to-use-a-read-replica"></a>何时使用只读副本

只读副本功能可帮助改善读取密集型工作负荷的性能与规模。 读取工作负载可以与副本服务器隔离，而写入工作负载可以定向到主服务器。

常见方案是让 BI 和分析工作负载将只读副本用作报告的数据源。

由于副本是只读的，它们不能直接缓解主服务器上的写入容量负担。 此功能并非面向写入密集型工作负荷。

读取的副本功能使用 MySQL 异步复制。 该功能不适用于同步复制方案。 主服务器与副本之间存在明显的延迟。 副本上的数据最终将与主服务器上的数据保持一致。 对于能够适应这种延迟的工作负荷，可以使用此功能。

读取副本可以增强灾难恢复计划。 如果没有区域性灾难并且主服务器不可用，可以将定向到另一个区域中的副本工作负荷。 若要执行此操作，首先让通过停止复制函数接受写入的副本。 然后可以将你的应用程序通过更新连接字符串来重定向。 了解详细信息[停止复制](#stop-replication)部分。

## <a name="create-a-replica"></a>创建副本

如果主服务器不的任何现有副本服务器，主将第一次重新启动以准备用于复制的本身。

创建副本工作流启动时，创建空的 Azure Database for MySQL 服务器。 新服务器中填充了主服务器上的数据。 创建时间取决于主服务器上的数据量，以及自上次每周完整备份以来所经历的时间。 具体所需时间从几分钟到几小时不等。

> [!NOTE]
> 如果尚未在服务器上设置存储警报，我们建议进行设置。 当服务器即将达到其存储限制（这会影响复制）时，警报可以向你发出通知。

了解如何[在 Azure 门户中创建只读副本](howto-read-replicas-portal.md)。

## <a name="connect-to-a-replica"></a>连接到副本

创建副本时，该副本不会继承主服务器的防火墙规则或 VNet 服务终结点。 必须单独为副本设置这些规则。

副本从主服务器继承其管理员帐户。 主服务器上的所有用户帐户将复制到只读副本。 只能使用主服务器上可用的用户帐户连接到只读副本。

您可以通过连接到副本使用其主机名和有效的用户帐户，就像常规的 Azure Database for MySQL 服务器上。 对于名为的服务器**myreplica**使用的管理员用户名**myadmin**，可以通过使用 mysql CLI 连接到副本：

```bash
mysql -h myreplica.mysql.database.azure.com -u myadmin@myreplica -p
```

在提示符下，输入用户帐户的密码。

## <a name="monitor-replication"></a>监视复制

Azure Database for MySQL 提供**复制延迟 （秒）** 在 Azure Monitor 指标。 此指标仅适用于副本。

使用计算此指标`seconds_behind_master`指标可在 MySQL 的`SHOW SLAVE STATUS`命令。

设置警报来通知你的复制延迟时达到不可接受的工作负荷的值。

## <a name="stop-replication"></a>停止复制

可以停止主服务器与副本之间的复制。 在主服务器与只读副本之间停止复制后，副本将成为独立服务器。 独立服务器中的数据是启动“停止复制”命令时副本上可用的数据。 独立服务器与主服务器不同步。

当您选择停止复制到副本时，它将丢失所有链接到其以前的主节点和其他副本。 主控形状及其副本之间没有自动故障转移。

> [!IMPORTANT]
> 独立服务器不能再次成为副本。
> 在只读副本上停止复制之前，请确保副本包含所需的全部数据。

了解如何[停止复制到副本](howto-read-replicas-portal.md)。

## <a name="considerations-and-limitations"></a>注意事项和限制

### <a name="pricing-tiers"></a>定价层

只读副本当前仅适用于“常规用途”和“内存优化”的定价层。

### <a name="master-server-restart"></a>主服务器重启

如果为没有现有副本的主服务器创建副本，主服务器将首先重启以便为复制准备自身。 请考虑这一点并在非高峰期执行这些操作。

### <a name="new-replicas"></a>新副本

为新的 Azure Database for MySQL 服务器创建了一个只读的副本。 无法将现有的服务器设为副本。 无法创建另一个只读副本的副本。

### <a name="replica-configuration"></a>副本配置

副本是使用与主服务器相同的服务器配置创建的。 创建一个副本之后，可以独立地更改多个设置从主服务器： 计算代、 vcore 数、 存储、 备份保留期和 MySQL 引擎版本。 定价层也可以独立更改，但“基本”层除外。

> [!IMPORTANT]
> 将主服务器的配置更新为新值之前，请将副本配置更新为与这些新值相等或更大的值。 此操作可确保副本与主服务器发生的任何更改保持同步。

### <a name="stopped-replicas"></a>停止的副本

如果停止主服务器和读取的副本之间的复制，已停止的副本将变成接受读取和写入的独立服务器。 独立服务器不能再次成为副本。

### <a name="deleted-master-and-standalone-servers"></a>删除的主服务器和独立服务器

删除主服务器后，将对所有只读副本停止复制。 这些副本服务器将成为独立服务器。 将删除主服务器本身。

### <a name="user-accounts"></a>用户帐户

主服务器上的用户将复制到只读副本。 只能使用主服务器上可用的用户帐户连接到只读副本。

### <a name="server-parameters"></a>服务器参数

若要防止数据变得不同步，并以避免潜在数据丢失或损坏，某些服务器参数会被锁定时使用读取副本更新。

以下的 server 参数被锁定主和副本服务器上：
- [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-multiple-tablespaces.html) 
- [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators)

[ `event_scheduler` ](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_event_scheduler)参数被锁定在副本服务器上。 

### <a name="other"></a>其他

- 不支持全局事务标识符 (GTID)。
- 不支持创建副本服务器的副本。
- 内存中的表可能会导致副本服务器变得不同步。这是 MySQL 复制技术的限制。 有关详细信息，请阅读 [MySQL 参考文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features-memory.html)中的更多信息。
- 确保主服务器表具有主键。 缺少主键可能会导致主服务器与副本服务器之间的复制延迟。
- 查看 [MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html)中 MySQL 复制限制的完整列表

## <a name="next-steps"></a>后续步骤

- 了解如何[使用 Azure 门户创建和管理只读副本](howto-read-replicas-portal.md)
- 了解如何[使用 Azure CLI 创建和管理只读副本](howto-read-replicas-cli.md)
