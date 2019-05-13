---
title: Azure 备份：监视 Azure 备份保护工作负荷
description: 监视使用 Azure 门户的 Azure 备份工作负荷
services: backup
author: pvrk
manager: shivamg
keywords: Azure 备份;警报;
ms.service: backup
ms.topic: conceptual
ms.date: 03/05/2019
ms.author: pullabhk
ms.assetid: 86ebeb03-f5fa-4794-8a5f-aa5cbbf68a81
ms.openlocfilehash: ab7d2c0af4bc71733a7995b7e781f0facbfbb29f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236439"
---
# <a name="monitoring-azure-backup-workloads"></a>监视 Azure 备份工作负荷

Azure 备份提供基于备份的要求和基础结构拓扑 （在本地或 Azure） 的多个备份解决方案。 任何备份用户或管理员应该会看到这怎么回事跨所有解决方案，需重要的应用场景中收到通知。 本文详细介绍了由 Azure 备份服务提供的监视和通知功能。

## <a name="backup-jobs-in-recovery-services-vault"></a>在恢复服务保管库的备份作业

Azure 备份提供内置监视和警报功能适用于由 Azure 备份保护的工作负荷。 在恢复服务保管库设置，**监视**部分提供了内置作业和警报。

![RS 保管库内置监视](media/backup-azure-monitoring-laworkspace/rs-vault-inbuiltmonitoring.png)

执行如配置备份、 备份、 还原、 删除备份等操作时，将生成作业。

以下 Azure 备份解决方案中的作业将如下所示：

  - Azure VM 备份
  - Azure 文件备份
  - 例如，SQL azure 工作负荷备份
  - Azure 备份代理 (MAB)

不会显示作业从 System Center Data Protection Manager (SC DPM)，Microsoft Azure 备份服务器 (MABS)。

> [!NOTE]
> Azure 工作负荷，例如 SQL 备份 Azure Vm 中有大量的备份作业。 例如，可以运行每隔 15 分钟的日志备份。 因此，对于此类数据库工作负荷中，将显示唯一用户触发操作。 不显示计划的备份操作。

## <a name="backup-alerts-in-recovery-services-vault"></a>在恢复服务保管库的备份警报

警报是主要方案，其中用户会收到通知，以便它们可以采取相关措施。 **备份警报**部分显示了由 Azure 备份服务生成的警报。 这些警报由服务定义和用户不能自定义创建的任何警报。

### <a name="alert-scenarios"></a>警报方案
以下方案被指由服务可报警的方案。

  - 备份/还原失败
  - 备份已成功但出现警告的 Azure 备份代理 (MAB)
  - 停止保护并保留数据/停止保护并删除数据

### <a name="exceptions-when-an-alert-is-not-raised"></a>不会引发警报时出现的异常
有几个例外情况不在失败时引发警报时，它们是：

  - 用户显式取消正在运行的作业
  - 该作业会失败，因为另一个备份作业正在进行 （无要作用于此处，因为我们只需等待上一个作业完成）
  - VM 备份作业会失败，因为备份 Azure VM 不再存在

上述异常旨在从了解这些操作 （主要用户触发） 的结果中显示立即门户/PS/CLI 客户端。 因此，用户立即注意的是，无需通知。

### <a name="alerts-from-the-following-azure-backup-solutions-are-shown-here"></a>来自以下 Azure 备份解决方案的警报将如下所示：

  - Azure VM 备份
  - Azure 文件备份
  - 例如，SQL azure 工作负荷备份
  - Azure 备份代理 (MAB)

> [!NOTE]
> 此处不显示警报从 System Center Data Protection Manager (SC DPM)，Microsoft Azure 备份服务器 (MABS)。

### <a name="alert-types"></a>警报类型
根据警报的严重性，可以在三种类型中定义警报：

  - **严重**：在原则、 任何备份或恢复失败 （计划或用户触发） 将导致生成警报，并将显示为严重警报和破坏性操作，例如删除备份。
  - 警告：如果备份操作成功，但有几个警告，它们会作为警告性警报中列出。
  - **信息性**:到目前为止，Azure 备份服务不生成任何信息性警报。

## <a name="notification-for-backup-alerts"></a>备份警报通知

> [!NOTE]
> 可以仅通过 Azure 门户完成的通知配置。 不支持 PS/CLI/REST API/Azure 资源管理器模板支持。

一旦引发警报，通知用户。 Azure 备份提供了内置的通知机制，通过电子邮件。 一个可以指定单个电子邮件地址或通讯组列表时生成警报获得通知。 此外可以选择是否要获取有关每个警报的通知或按每小时摘要分组，然后获得通知。

![RS 保管库内置电子邮件通知](media/backup-azure-monitoring-laworkspace/rs-vault-inbuiltnotification.png)

配置通知后，您将收到欢迎使用或介绍性电子邮件。 这会确认 Azure 备份，可以向这些地址中发送电子邮件时引发警报。<br>

如果频率设置为每小时摘要和警报已引发并解决一小时内，则它不会即将发布的每小时摘要的一部分。

> [!NOTE]
>
> * 如果如破坏性操作**停止保护并删除数据**是执行，会生成一个警报，并向订阅所有者、 管理员和共同管理员发送电子邮件，即使通知未配置为恢复服务保管库。
> * 对于成功的作业使用配置通知[Log Analytics](backup-azure-monitoring-use-azuremonitor.md#using-log-analytics-workspace)。

## <a name="next-steps"></a>后续步骤

[监视使用 Azure Monitor 的 Azure 备份工作负荷](backup-azure-monitoring-use-azuremonitor.md)
