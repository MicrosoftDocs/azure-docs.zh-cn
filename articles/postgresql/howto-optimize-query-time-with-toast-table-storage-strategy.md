---
title: 通过使用 TOAST 表存储策略的 Azure 数据库中的 PostgreSQL-单个服务器来优化查询时
description: 本文介绍如何优化查询时使用 Azure 数据库上的 TOAST 表存储策略用于 PostgreSQL 的单个服务器。
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: ac1dc43a2b89bc1cc748947ec08e6ada87edbfcb
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65066979"
---
# <a name="optimize-query-time-with-the-toast-table-storage-strategy"></a>使用 TOAST 表存储策略优化查询时间 
本文介绍如何使用超大属性存储技术 (TOAST) 表存储策略优化查询时间。

## <a name="toast-table-storage-strategies"></a>TOAST 表存储策略
可通过四种不同的策略在能够使用 TOAST 的磁盘上存储列。 它们代表压缩操作和外部存储的各种组合。 可在数据类型级别和列级别设置策略。
- “普通”策略可同时防止压缩或外部存储。 该策略禁止为 varlena 类型使用单字节标头。 “普通”策略是可用于无法使用 TOAST 的数据类型的列的唯一策略。
- “扩展”策略允许压缩和行外存储。 “扩展”策略是大多数能够使用 TOAST 的数据类型的默认策略。 首先尝试压缩。 如果行仍然过大，则尝试外部存储。
- “外部”策略允许行外存储，但不允许压缩。 使用“外部”策略可以提高宽文本和 bytea 列的子字符串操作速度。 此速度提升会同时增加占用的存储空间。 这些操作经过优化，在外部值未压缩时只提取必需的部分。
- “主要”策略允许压缩，但不允许行外存储。 仍可能为这类列采用外部存储，但仅用作最后的办法。 只有在没有其他办法能够缩小行来使其适合页面的情况下，才采用这种办法。

## <a name="use-toast-table-storage-strategies"></a>使用 TOAST 表存储策略
如果查询访问的数据类型可以使用 TOAST，请考虑使用“主要”策略而不是默认的“扩展”策略选项，以减少查询时间。 “主要”策略未排除外部存储。 如果查询不访问能够使用 TOAST 的数据类型，则保留“扩展”选项可能有利。 这样，主表中的大部分行会进入共享的缓冲区缓存中，从而有助于提高性能。

如果某个工作负荷使用的架构包含宽表和较多的字符，请考虑使用 PostgreSQL TOAST 表。 例如，某个客户表包含 350 个以上的列，其中多个列包含的字符数超过 255 个。 转换为 TOAST 表“主要”策略后，其基准查询时间从 4203 秒减少到 467 秒。 效率提升了 89%。

## <a name="next-steps"></a>后续步骤
查看工作负荷的前述特征。 

查看以下 PostgreSQL 文档： 
- [第 68 章，数据库物理存储](https://www.postgresql.org/docs/current/storage-toast.html) 