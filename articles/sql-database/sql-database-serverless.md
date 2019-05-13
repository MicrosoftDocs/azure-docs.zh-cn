---
title: Azure SQL 数据库无服务器 （预览） |Microsoft Docs
description: 本文介绍新的无服务器计算层，并将其与现有的预配的计算层
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: sstein, carlrab
manager: craigg
ms.date: 05/07/2019
ms.openlocfilehash: 7f850f309034d128efef89ea842db41d35b8491e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235740"
---
# <a name="sql-database-serverless-preview"></a>SQL 数据库无服务器 （预览版）

## <a name="what-is-the-serverless-compute-tier"></a>无服务器计算层是什么

SQL 数据库无服务器 （预览版） 是费用的计算基于每个第二个单一数据库使用的计算层。 无服务器是价格性能优化为单一数据库与可以承受一定的延迟在计算预热后空闲使用期间的突发性使用模式。
与此相反，公开可用产品/服务的 SQL 数据库现在帐单中计算每小时预配。 此预配的计算层是价格性能优化的单一数据库或弹性池，具有更高版本不能承受计算预热中的任何延迟的平均使用情况。

无服务器的计算机层中的数据库是通过它可以使用的计算范围和 autopause 延迟进行参数化。

![无服务器的计费](./media/sql-database-serverless/serverless-billing.png)

### <a name="performance"></a>性能

- `MinVcore` 和`MaxVcore`个可配置参数来定义的数据库可用的计算能力的范围。 内存和 IO 限制是指定的 vCore 范围成正比。  
- Autopause 延迟是时间的定义的数据库必须自动暂停之前处于非活动状态段的可配置参数。 下一次登录时自动恢复数据库。

### <a name="pricing"></a>定价

- 无服务器数据库的总费用是计算帐单和存储的收费的总和。
计算的计费基于使用的 vcore 数和每秒使用的内存量。
- 计费的最小计算基于最小 vcore 数和最小内存。
- 在数据库暂停时，仅的存储器计费。

## <a name="scenarios"></a>方案

无服务器是性价比单一数据库可以承受一定的延迟在计算预热后空闲使用期间的突发性使用模式与针对进行了优化。 预配的计算层是价格性能优化具有更高版本不能承受计算预热中的任何延迟的平均使用情况的单个或共用数据库。

下表比较了与预配的计算层的无服务器计算级别：

||无服务器计算|已预配计算|
|---|---|---|
|**典型使用方案**|具有突发、 不可预测的使用情况与非活动的高峰期交织在一起的数据库|数据库或弹性池具有多个常规使用情况|
|**性能管理工作**|较低|更高|
|**计算缩放**|自动|手动|
|**计算响应能力**|非活动期后较低|即时|
|**计费粒度**|每秒|每小时|
|

### <a name="scenarios-well-suited-for-serverless-compute"></a>方案非常适用于无服务器计算

- 单一数据库与突发性使用模式与处于非活动状态的高峰期交织在一起可以受益于价格节省的金额以每秒所使用的计算量计费。
- 单一数据库与资源需求，很难进行预测和客户的首选委托到服务的计算大小调整。
- 预配的计算层中的单个数据库经常更改性能级别。

### <a name="scenarios-well-suited-for-provisioned-compute"></a>非常适用于预配的计算方案

- 单一数据库与更多常规模式和多个庞大的因此计算随时间推移的利用率。
- 不能容忍性能之间做出取舍所得的详细信息的数据库频繁的内存修整或延迟 autoresuming 从暂停状态。
- 包含可以合并到单个服务器和使用弹性池的更好的价格优化突发性使用模式的多个数据库。


## <a name="purchasing-model-and-service-tier"></a>购买模型和服务层

无服务器的 SQL 数据库目前仅支持常规用途层中的购买模型的 vCore 生成 5 硬件上。

## <a name="autoscaling"></a>自动缩放

### <a name="scaling-responsiveness"></a>缩放的响应能力

一般情况下，数据库是在计算机上运行具有足够的容量来满足资源请求而不发生中断的任意数量的计算中设置的限制请求`maxVcores`值。 有时，如果计算机无法满足资源需求的几分钟内，会发生自动平衡负载。 在负载平衡除外操作结束时在短时间内，将删除连接时，数据库保持联机状态。

### <a name="memory-management"></a>内存管理

无服务器数据库的内存回收更频繁地比的预配数据库。 此行为是重要中无服务器的控制成本。 与不同的预配的计算，从 SQL 缓存回收内存从无服务器数据库时 CPU 或缓存使用率较低。

## <a name="autopause-and-autoresume"></a>自动暂停和 autoresume

### <a name="autopause"></a>自动暂停

如果所有以下条件成立 autopause 延迟的持续时间内，会触发自动暂停：

- 会话数目 = 0
- CPU = 0 （用户运行的工作负荷中的用户池）

若要禁用自动暂停，如果需要，提供了一个选项。

### <a name="autoresume"></a>Autoresume

如果以下任何条件成立在任何时候，会触发 Autoresume:

|Feature|Autoresume 触发器|
|---|---|
|身份验证和授权|登录|
|威胁检测|启用/禁用在数据库或服务器级别的威胁检测设置<br>修改数据库或服务器级别的威胁检测设置|
|数据发现和分类|添加、 修改、 删除或查看敏感度标签|
|审核|查看审核记录。<br>更新或查看审核策略|
|数据屏蔽|添加、 修改、 删除或查看数据屏蔽规则|
|透明数据加密|视图状态或透明数据加密状态|
|查询 （性能） 数据存储|修改或查看查询存储设置;自动优化|
|自动调整|自动调整建议，例如自动索引的应用和验证|
|数据库复制|作为副本创建数据库<br>导出到 BACPAC 文件|
|SQL 数据同步|中心和成员数据库的可配置计划上运行，或手动执行之间的同步|
|修改特定数据库的元数据|添加数据库的新标记<br>更改最大 vcore 数，最小 Vcore autopause 延迟|
|SQL Server Management Studio (SSMS)|使用 SSMS 版本 18 和在服务器中打开新查询窗口中的任何数据库将继续进行在同一台服务器中的任何已自动暂停数据库。 如果使用 SSMS 版本 17.9.1 intellisense 处于关闭状态的则此行为不会发生。|

### <a name="connectivity"></a>连接性

如果暂停无服务器数据库，然后将恢复数据库首次登录并返回一个错误，指出数据库将不可用。 后恢复数据库时，该登录名必须重试建立连接。 数据库客户端连接重试逻辑应该不需要进行修改。

### <a name="latency"></a>延迟

自动暂停或 autoresume 无服务器数据库连接的延迟时间是通常大约 1 分钟。

### <a name="feature-support"></a>支持的功能

Autopausing 和 autoresuming 不支持以下功能。 也就是说，如果使用了以下功能，然后在数据库保持联机无论数据库处于不活动状态的持续时间：

- 异地复制 （活动异地复制和自动故障转移组）
- 长期备份保留 (LTR)
- 使用 SQL 数据同步中的同步数据库。


## <a name="on-boarding-into-the-serverless-compute-tier"></a>载入到无服务器计算层级

创建新的数据库或移动到无服务器计算层的现有数据库遵循相同的模式与创建的新数据库中预配计算层以及涉及以下两个步骤：

1. 指定服务目标名称。 下表显示了可用的服务层和在公共预览版中当前可用的计算大小。

   |服务层|计算大小|
   |---|---|
   |常规用途|GP_S_Gen5_1|
   |常规用途|GP_S_Gen5_2|
   |常规用途|GP_S_Gen5_4|

2. （可选） 指定要更改其默认值的最小 vcore 数和自动暂停的延迟。 下表显示了这些参数的可用值。

   |参数|值选项|默认值|
   |---|---|---|---|
   |最小 vCore 数|{0.5、 1、 2、 4} 的任何不超过最大 vcore 数|0.5 个 Vcore|
   |Autopause 延迟|最小值：360 分钟 （6 小时）<br>最大值：10080 分钟 （7 天）<br>增量：60 分钟<br>禁用自动暂停:-1|360 分钟|

> [!NOTE]
> 使用 T-SQL 将现有数据库移到无服务器或更改其计算大小目前不支持，但可以通过 Azure 门户或 PowerShell 完成。

### <a name="create-new-database-using-the-azure-portal"></a>使用 Azure 门户创建新数据库

请参阅[快速入门：在 Azure SQL 数据库使用 Azure 门户中创建单个数据库](sql-database-single-database-get-started.md)。

### <a name="create-new-database-using-powershell"></a>使用 PowerShell 创建新数据库

下面的示例定义名为最小 vcore 数和 autopause 延迟使用默认值的 GP_S_Gen5_4 的服务目标的无服务器计算级别创建新的数据库。

无服务器需要较新版本的 PowerShell 不是目前在库中，因此运行`Update-Module Az.Sql`以获取最新的启用了无服务器的 cmdlet。

```powershell
New-AzSqlDatabase `
  -ResourceGroupName $resourceGroupName `
  -ServerName $serverName `
  -DatabaseName $databaseName `
  -ComputeModel Serverless `
  -Edition GeneralPurpose `
  -ComputeGeneration Gen5 `
  -MinVcore 0.5 `
  -MaxVcore 2 `
  -AutoPauseDelay 720
```

### <a name="move-existing-database-into-the-serverless-compute-tier"></a>将现有数据库移到无服务器计算层

下面的示例将现有单一数据库的预配的计算层中移入无服务器计算层。 此示例使用默认值为最小 Vcore、 最大 vcore 数和 autopause 延迟。

```powershell
Set-AzSqlDatabase
  -ResourceGroupName $resourceGroupName `
  -ServerName $serverName `
  -DatabaseName $databaseName `
  -Edition GeneralPurpose `
  -ComputeModel Serverless `
  -ComputeGeneration Gen5 `
  -MinVcore 1 `
  -MaxVcore 4 `
  -AutoPauseDelay 1440
```

### <a name="move-a-database-out-of-the-serverless-compute-tier"></a>将无服务器计算层外的数据库移动

无服务器数据库可以移动到预配的计算层中，将预配的计算数据库移动到无服务器计算层相同的方式。

## <a name="modify-serverless-configuration-parameters"></a>修改无服务器配置参数

### <a name="maximum-vcores"></a>最大 vCore 数

修改最大 vcore 数使用执行[集 AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase)命令，在 PowerShell 中使用`MaxVcore`参数。

### <a name="minimum-vcores"></a>最小 vCore 数

修改最大 vcore 数使用执行[集 AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase)命令，在 PowerShell 中使用`MinVcore`参数。

### <a name="autopause-delay"></a>Autopause 延迟

修改最大 vcore 数使用执行[集 AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase)命令，在 PowerShell 中使用`AutoPauseDelay`参数。

## <a name="monitor-serverless-database"></a>监视无服务器数据库

### <a name="resources-used-and-billed"></a>资源使用和计费

由下列实体来封装的无服务器数据库的资源：

#### <a name="app-package"></a>应用包

应用程序包是一个数据库，而不考虑数据库是在无服务器或预配计算层中的大多数外部资源管理边界。 应用包中包含的 SQL 实例和外部服务该组合在一起的作用域使用的 SQL 数据库中的数据库的所有用户和系统资源。 外部服务的示例包括 R 和全文搜索。 SQL 实例通常决定跨应用包的整体资源利用率。

#### <a name="user-resource-pool"></a>用户资源池

用户资源池是内部数据库，而不考虑数据库是在无服务器或预配计算层中的大多数资源管理边界。 用户资源池的 CPU 和 IO 范围的用户工作负荷生成的 DDL 查询 （例如，CREATE、 ALTER 等） 和 DML 查询 （例如，SELECT、 INSERT、 UPDATE、 DELETE、 等等。）。 这些查询通常表示应用包中的使用率的最重要的比例。

### <a name="metrics"></a>度量值

|实体|指标|描述|单位|
|---|---|---|---|
|应用包|app_cpu_percent|相对于应用程序允许的最大 vcore 数应用使用的 vcore 数的百分比。|百分比|
|应用包|app_cpu_billed|计算报表期间应用的计费量。 在此期间支付的金额是此度量值和 vCore 单位价格的产品。<br>此指标的值均由随时间的 CPU 最大值使用聚合和使用每秒的内存。<br>如果使用量小于最小为组设置的最小 vcore 数和最小内存量，然后预配的最小量计费。  要比较 CPU 与内存出于计费目的，通过重新以 gb 为单位的 3 GB 的内存量缩放每个 vCore 内存规范化为单位的 Vcore。|vCore 秒|
|应用包|app_memory_percent|相对于应用程序允许的最大内存的应用使用的内存的百分比。|百分比|
|用户池|cpu_percent|Vcore 数由用户工作负荷相对于允许的用户工作负荷的最大 vcore 数的百分比。|百分比|
|用户池|data_IO_percent|用户工作负荷的允许数据由用户工作负荷相对于最大数据 IOPS 的 IOPS 的百分比。|百分比|
|用户池|log_IO_percent|日志的百分比 MB/秒的相对最大日志 MB/秒的用户工作负载使用允许用户工作负荷。|百分比|
|用户池|workers_percent|使用相对于允许的用户工作负荷的最大工作进程的用户工作负荷的辅助进程的百分比。|百分比|
|用户池|sessions_percent|会话的用户工作负载相对于允许的用户工作负荷的最大会话使用的百分比。|百分比|
____

> [!NOTE]
> 在 Azure 门户中的度量值均位于单一数据库下的数据库窗格**监视**。

### <a name="pause-and-resume-status"></a>暂停和恢复状态

在 Azure 门户中，列出它所包含的数据库的服务器的概述窗格中显示数据库状态。 为数据库概述窗格中还显示数据库状态。

使用以下 PowerShell 命令来查询暂停和恢复数据库的状态：

```powershell
Get-AzSqlDatabase `
  -ResourceGroupName $resourcegroupname `
  -ServerName $servername `
  -DatabaseName $databasename `
  | Select -ExpandProperty "Status"
```

## <a name="resource-limits"></a>资源限制

有关资源限制，请参阅[无服务器计算层](sql-database-vCore-resource-limits-single-databases.md#serverless-compute-tier)

## <a name="billing"></a>计费

计费每秒量是计算的使用 CPU 和内存使用每秒的最大值。 如果使用的 CPU 量并且使用的内存小于最小金额为每个预配，则计费预配的量。 要比较 CPU 与内存出于计费目的，通过重新以 gb 为单位的 3 GB 的内存量缩放每个 vCore 内存规范化为单位的 Vcore。

- **资源计费**:CPU 和内存
- **量计费 （$）**: vCore 单价 * 最大值 (最小 Vcore、 使用的 vcore 数、 最小内存 GB * 1/3，内存使用的 GB * 1/3) 
- **计费频率**:每秒

按以下度量值公开了计费的计算量：

- **指标**: app_cpu_billed （vCore 秒为单位）
- **定义**： 最大值 (最小 Vcore、 使用的 vcore 数、 最小内存 GB * 1/3，内存使用的 GB * 1/3) *
- **报告频率**:每分钟

> [!NOTE]
> \* 此数量是计算每秒，超过 1 分钟聚合。

**示例**：请考虑以下使用情况与使用 GP_S_Gen5_4，在一小时内的数据库：

|时间 （小时： 分钟）|app_cpu_billed （vCore 秒为单位）|
|---|---|
|0:01|63|
|0:02|123|
|0:03|95|
|0:04|54|
|0:05|41|
|0:06 - 1:00|1255|
||总计:1631|

假设计算单位价格为 $0.2609/vCore/hour。 然后使用以下公式确定支付这一个小时内的计算： **$0.2609/vCore/hour * 1631 vCore 秒 * 1 小时/3600 秒 = $0.1232**

## <a name="available-regions"></a>可用区域

无服务器计算层在除以下区域之外的所有区域均提供：澳大利亚中部、 中国东部、 中国北部、 法国南部、 德国中部、 德国东北部、 印度西部、 韩国南部、 南非西部、 英国北部、 英国南部、 英国西部和美国中西部

## <a name="next-steps"></a>后续步骤

有关资源限制，请参阅[无服务器计算层资源限制](sql-database-vCore-resource-limits-single-databases.md#serverless-compute-tier)。