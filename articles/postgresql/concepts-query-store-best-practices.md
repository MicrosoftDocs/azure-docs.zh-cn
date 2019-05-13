---
title: Azure Database for PostgreSQL-单个服务器中的 query Store 最佳做法
description: 本文介绍用于 PostgreSQL 的单个服务器的 Azure 数据库中的查询存储最佳做法。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 798a7a3edbf11c8421848871d26ba55b5bada0b6
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067237"
---
# <a name="best-practices-for-query-store"></a>查询存储的最佳做法

**适用于：** Azure Database for PostgreSQL-9.6 和 10 台服务器

本文概述了在 Azure Database for PostgreSQL 中使用查询存储的最佳做法。

## <a name="set-the-optimal-query-capture-mode"></a>设置最佳查询捕获模式
让查询存储捕获重要的数据。 

|**pg_qs.query_capture_mode** | **方案**|
|---|---|
|_全部_  |根据所有查询及其执行频率和其他统计信息彻底分析工作负荷。 识别工作负荷中的新查询。 检测是否使用即席查询来识别用户或自动参数化的机会。 “全部”会增加资源消耗成本。 |
|顶部  |将注意力集中在热门查询 - 客户发布的查询上。
|无 |已捕获了要调查的查询集和时间窗口，并且希望消除其他查询可能引入的干扰。 “无”适用于测试和基准测试环境。 由于可能错过了跟踪和优化重要新查询的机会，因此应谨慎使用“无”。 无法恢复那些过去时间窗口上的数据。 |

查询存储还包括等待统计信息库。 还有一个额外的捕获模式查询可管理等待统计信息：可将 pgms_wait_sampling.query_capture_mode 设置为“无”或“全部”。 

> [!NOTE] 
> 将 pgms_wait_sampling.query_capture_mode 替代为 pg_qs.query_capture_mode。 若 pg_qs.query_capture_mode 设置为“无”，则 pgms_wait_sampling.query_capture_mode 设置无效。 


## <a name="keep-the-data-you-need"></a>保留所需的数据
该 pg_qs.retention_period_in_days 参数指定查询存储的数据保留期（以天为单位）。 删除较旧的查询和统计信息数据。 默认情况下，查询存储配置为将数据保留 7 天。 避免保留不打算使用的历史数据。 若需要将数据保留更长时间，请增大保留期的值。


## <a name="set-the-frequency-of-wait-stats-sampling"></a>设置等待统计信息采样频率 
该 pgms_wait_sampling.history_period 参数指定等待事件的采样频率（以毫秒为单位）。 周期越短，采样越频繁。 可检索到更多信息，但这会导致更多资源消耗。 若服务器负载不足或不需要粒度，请增加此时间段


## <a name="get-quick-insights-into-query-store"></a>快速了解查询存储
可使用 Azure 门户中的 [Query Performance Insight](concepts-query-performance-insight.md) 来快速了解查询存储的数据。 可视化显示运行时间最长的查询和最长的等待事件。

## <a name="next-steps"></a>后续步骤
- 了解如何使用 [Azure 门户](howto-configure-server-parameters-using-portal.md)或 [Azure CLI](howto-configure-server-parameters-using-cli.md) 获取或设置参数。