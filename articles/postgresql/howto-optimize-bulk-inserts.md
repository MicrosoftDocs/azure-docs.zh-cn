---
title: 用于 PostgreSQL 的单个服务器优化 Azure 数据库上的大容量插入
description: 本文介绍如何优化大容量插入操作在 Azure Database for PostgreSQL-单个服务器。
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: c1ae29f7c498a79af09aaaf6d7aeae29561aa500
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067031"
---
# <a name="optimize-bulk-inserts-and-use-transient-data-on-an-azure-database-for-postgresql---single-server"></a>优化大容量插入和使用用于 PostgreSQL 的单个服务器的 Azure 数据库上的暂时性数据 
本文介绍如何优化 Azure Database for PostgreSQL 服务器中的批量插入操作以及使用瞬态数据。

## <a name="use-unlogged-tables"></a>使用无日志记录表
如果工作负荷操作涉及到瞬态数据或者需要批量插入大型数据集，可以考虑使用无日志记录表。

无日志记录表是一项 PostgreSQL 功能，使用它能够有效地优化批量插入。 PostgreSQL 使用预写日志 (WAL)。 它在默认情况下提供原子性和持久性。 原子性、一致性、隔离性和持久性组成了 ACID 属性。 

插入到无日志记录表中意味着 PostgreSQL 会进行插入而不会写入到事务日志中，后者本身是 I/O 操作。 因此，这些表远远快于普通表。

使用以下选项可创建无日志记录表：
- 使用语法 `CREATE UNLOGGED TABLE <tableName>` 创建新的无日志记录表。
- 使用语法 `ALTER TABLE <tableName> SET UNLOGGED` 将现有日志记录表转换为无日志记录表。  

若要撤消过程，请使用语法 `ALTER TABLE <tableName> SET LOGGED`。

## <a name="unlogged-table-tradeoff"></a>无日志记录表的利弊
无日志记录表无法防崩溃。 发生崩溃或者执行不彻底的关机后，无日志记录表将被自动截断。 此外，无日志记录表的内容不会复制到备用服务器。 基于无日志记录表创建的任何索引也不会自动取消记录。 插入操作完成后，将表转换为日志记录表，使插入操作具有持久性。

在使用无日志记录表时，某些客户工作负荷可体验到大约 15-20% 的性能改善。

## <a name="next-steps"></a>后续步骤
查看工作负荷中使用的瞬态数据和批量插入。 请参阅以下 PostgreSQL 文档：
 
- [创建表 SQL 命令](https://www.postgresql.org/docs/current/static/sql-createtable.html)
