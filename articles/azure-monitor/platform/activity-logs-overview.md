---
title: Azure 活动日志概述
description: 了解什么是 Azure 活动日志，以及如何通过它了解发生在 Azure 订阅中的事件。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: johnkem
ms.subservice: logs
ms.openlocfilehash: d9583f232a7afd6ab64421d57bbf14a45299e374
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138194"
---
# <a name="monitor-subscription-activity-with-the-azure-activity-log"></a>使用 Azure 活动日志监视订阅活动

Azure 活动日志是一种方便用户深入了解 Azure 中发生的订阅级别事件的订阅日志。 这包括从 Azure 资源管理器操作数据到服务运行状况事件更新的一系列数据。 活动日志之前称为“审核日志”或“操作日志”，因为“管理”类别报告订阅的控制面事件。 通过活动日志，可确定订阅中资源上进行的任何写入操作 (PUT, POST, DELETE) 的“什么操作、谁操作和操作时间”等信息。 还可以了解该操作和其他相关属性的状态。 活动日志未包括读取 (GET) 操作或针对使用经典/“RDFE”模型的资源的操作。

![活动日志与其他类型的日志](./media/activity-logs-overview/Activity_Log_vs_other_logs_v5.png)

图 1：活动日志与其他类型的日志

活动日志不同于[诊断日志](diagnostic-logs-overview.md)。 活动日志提供有关从外部（“控制面”）对资源所执行操作的数据。 诊断日志由资源发出，并提供有关该资源（“数据平面”）的操作信息。

> [!WARNING]
> Azure 活动日志主要用于在 Azure 资源管理器中发生的活动。 它不跟踪使用经典/RDFE 模型的资源。 某些经典资源类型在 Azure 资源管理器中具有代理资源提供程序（例如 Microsoft.ClassicCompute）。 如果通过 Azure 资源管理器使用这些代理资源提供程序与经典资源类型进行交互，则操作会显示在活动日志中。 如果在 Azure 资源管理器代理外部与经典资源类型进行交互，则操作只会记录在操作日志中。 可以在门户的一个单独部分中浏览操作日志。
>
>

可以通过 Azure 门户、CLI、PowerShell cmdlet 和 Azure 监视器 REST API 从活动日志检索事件。

> [!NOTE]
> [新型警报](../../azure-monitor/platform/alerts-overview.md)在创建和管理活动日志警报规则时提供了增强的体验。  [了解详细信息](../../azure-monitor/platform/alerts-activity-log.md)。

## <a name="categories-in-the-activity-log"></a>活动日志中的类别
活动日志包含多个数据类别。 有关这些类别的架构的完整详细信息，请参阅[此文章](../../azure-monitor/platform/activity-log-schema.md)。 其中包括：
* 管理 - 此类别包含通过资源管理器执行的所有创建、更新、删除和行动操作的记录。 此类别中的事件类型的示例包括“创建虚拟机”和“删除网络安全组”。用户或应用程序通过资源管理器所进行的每一个操作都会作为特定资源类型上的操作建模。 如果操作类型为“写入”、“删除”或“操作”，则该操作的开始、成功或失败记录都会记录在管理类别中。 管理类别还包括任何对订阅中基于角色的访问控制进行的更改。
* 服务运行状况 - 此类别包含 Azure 中发生的任何服务运行状况事件的记录。 此类别的一个事件类型示例是“美国东部的 SQL Azure 正处于故障时间”。 服务运行状况事件有 5 种：必需操作、辅助恢复、事件、维护、信息或安全性，仅当订阅中存在受事件影响的资源时，它们才出现。
* **资源运行状况** - 此类别包含 Azure 资源发生的任何资源运行状况事件的记录。 你将在此类别中看到的事件类型的示例是“虚拟机运行状况已更改为不可用”。 资源运行状况事件可以表示以下四种运行状况状态之一：“Available”、“Unavailable”、“Degraded”和“Unknown”。 此外，资源运行状况事件可以分为“平台启动”或“用户启动”。
* 警报 - 此类别包含所有 Azure 警报的激活记录。 可在此类别中看到的事件类型示例如“过去 5 分钟内，myVM 上的 CPU 百分比已超过 80%”。 许多 Azure 系统都具有警报概念 - 可定义某种类型的规则，并在条件匹配该规则时接收通知。 每当支持的 Azure 警报类型“激活”或满足生成通知的条件时，激活记录也会推送到此类别的活动日志中。
* 此类别包含基于订阅中定义的任何自动缩放设置的自动缩放引擎操作相关的所有事件记录。 可在此类别中看到的事件类型示例如“自动缩放扩展操作失败”。 使用自动缩放，可在支持的资源类型中，通过自动缩放设置基于日期和/或负载（指标）数据来自动增加或减少实例的数量。 满足纵向扩展或缩减条件时，开始、成功或失败的事件会记录到此类别中。
* **建议** - 此类别包含 Azure 顾问提供的建议事件。
* **安全性** - 此类别包含 Azure 安全中心生成的任何警报记录。 可在此类别中看到的事件类型示例为“执行了可疑的双扩展名文件”。
* **Policy** - 此类别包含 Azure Policy 执行的所有效果操作的记录。 在此类别中看到的事件类型示例包括“审核”和“拒绝”。 Policy 执行的每个操作建模为对资源执行的操作。

## <a name="event-schema-per-category"></a>每个类别的事件架构

[请参阅此文章，了解每个类别的活动日志事件架构。](../../azure-monitor/platform/activity-log-schema.md)

## <a name="what-you-can-do-with-the-activity-log"></a>可以对活动日志执行的操作

可以对活动日志执行的部分操作如下：

![Azure 活动日志](./media/activity-logs-overview/Activity_Log_Overview_v3.png)


* 在 **Azure 门户**中查询和查看活动日志。
* [根据活动日志事件创建警报](../../azure-monitor/platform/activity-log-alerts.md)。
* [到 Stream**事件中心**](../../azure-monitor/platform/activity-logs-stream-event-hubs.md) ，方便第三方服务或 Power BI 等自定义分析解决方案引入。
* 在 Power BI 使用数据进行分析[ **Power BI 内容包**](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs/)。
* [将活动日志保存到**存储帐户**进行存档或手动检查](../../azure-monitor/platform/archive-activity-log.md)。 可以使用“日志配置文件”指定保留时间（天）。
* 通过 PowerShell Cmdlet、CLI 或 REST API 查询活动日志。
* 视图[更改历史记录](#view-change-history)对于某些事件

## <a name="query-the-activity-log-in-the-azure-portal"></a>在 Azure 门户中查询活动日志

> [!NOTE] 
> 活动日志将日志保留 90 天后端中。 如果你想要保留的数据超出此限制，请配置**日志配置文件**如下所述。 

在 Azure 门户中，可在多个位置查看活动日志：
* 可通过在左侧导航窗格中的“所有服务”下搜索活动日志进行访问的“活动日志”。
* 默认情况下在左侧导航窗格中显示的“监视”。 活动日志是 Azure Monitor 的一部分。
* 大多数**资源**，例如，虚拟机的配置边栏选项卡。 活动日志是大多数资源边栏选项卡的一部分，单击它可自动筛选出与该特定资源相关的事件。

在 Azure 门户中，可通过以下字段筛选活动日志：
* 时间跨度 - 事件的开始时间和结束时间。
* 类别 - 上述的事件类别。
* 订阅 - 一个或多个 Azure 订阅名称。
* 资源组 - 这些订阅中的一个或多个资源组。
* 资源（名称）- 特定资源的名称。
* 资源类型 - 资源的类型，例如 Microsoft.Compute/virtualmachines。
* 操作名称 - Azure 资源管理器操作的名称，例如 Microsoft.SQL/servers/Write。
* 严重性 - 事件的严重性级别（信息、警告、错误、严重）。
* 事件发起者 -“调用方”或执行操作的用户。
* 开放搜索 - 这是一个开放的文本搜索框，可在所有事件的所有字段中搜索该字符串。

定义了一组筛选器后，可以将查询固定到 Azure 仪表板上，以便始终关注特定事件。

若要获得更强大的功能，可以单击“日志”图标，这会在[收集和分析活动日志解决方案](../../azure-monitor/platform/collect-activity-logs.md)中显示活动日志数据。 “活动日志”边栏选项卡提供对日志的基础筛选/浏览体验，但 Azure Monitor 日志功能能够以更强大的方式对数据进行透视、查询和可视化。

## <a name="export-the-activity-log-with-a-log-profile"></a>使用日志配置文件导出活动日志

**日志配置文件**控制如何导出活动日志。 可以使用日志配置文件配置：

* 应将活动日志发送到何处：存储帐户或事件中心
* 应发送哪些事件类别（写入、删除、操作）。 *日志配置文件中“类别”的含义与活动日志事件中不同。在日志配置文件中，“类别”表示操作类型（写入、删除、操作）。在活动日志事件中，“类别”属性表示事件的来源或类型（例如，管理、服务运行状况、警报，等等）。*
* 应该导出哪些区域（位置）。 确保包含“全局”，因为活动日志中的事件多为全局事件。
* 活动日志应当在存储帐户中保留多长时间。
    - 保留期为 0 天表示永久保留日志。 如果不需永久保留，则可将该值设置为 1 到 365 之间的任意天数。
    - 如果设置了保留策略，但禁止将日志存储在存储帐户中（例如，如果仅选择了“事件中心”或“Log Analytics”选项），则保留策略无效。
    - 保留策略按天应用，因此在一天结束时 (UTC)，会删除当天已超过保留策略期限的日志。 例如，假设保留策略的期限为一天，则在今天开始时，会删除前天的日志。 删除过程从午夜 (UTC) 开始，但请注意，可能最多需要 24 小时才能将日志从存储帐户中删除。

可以使用与发出日志的订阅不同的订阅中的存储帐户或事件中心命名空间。 配置此设置的用户必须对两个订阅都具有合适的 RBAC 访问权限。

这些设置可以通过门户中活动日志边栏选项卡上的“导出”选项进行配置。 也可以[使用 Azure 监视器 REST API](https://msdn.microsoft.com/library/azure/dn931927.aspx)、PowerShell cmdlet 或 CLI 以编程方式对其进行配置。 一个订阅只能有一个日志配置文件。

### <a name="configure-log-profiles-using-the-azure-portal"></a>通过 Azure 门户配置日志配置文件

可以在 Azure 门户中使用“导出到事件中心”选项将活动日志流式传输到事件中心，或者将其存储在存储帐户中。

1. 使用门户左侧的菜单导航到“活动日志”。

    ![在门户中导航到“活动日志”](./media/activity-logs-overview/activity-logs-portal-navigate-v2.png)
2. 单击边栏选项卡顶部的“导出到事件中心”按钮。

    ![门户中的“导出”按钮](./media/activity-logs-overview/activity-logs-portal-export-v2.png)
3. 在显示的边栏选项卡中，可以选择：
   * 要导出事件的区域
   * 要保存事件的存储帐户
   * 要在存储中保留这些事件的天数。 设置为 0 天将永久保留日志。
   * 需要在其中创建事件中心，以便流式传输这些事件的服务总线命名空间。

     ![“导出活动日志”边栏选项卡](./media/activity-logs-overview/activity-logs-portal-export-blade.png)
4. 单击“保存”保存这些设置。 这些设置将立即应用于你的订阅。

### <a name="configure-log-profiles-using-the-azure-powershell-cmdlets"></a>通过 Azure PowerShell Cmdlet 配置日志配置文件

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

#### <a name="get-existing-log-profile"></a>获取现有的日志配置文件

```powershell
Get-AzLogProfile
```

#### <a name="add-a-log-profile"></a>添加日志配置文件

```powershell
Add-AzLogProfile -Name my_log_profile -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Location global,westus,eastus -RetentionInDays 90 -Category Write,Delete,Action
```

| 属性 | 需要 | 描述 |
| --- | --- | --- |
| Name |是 |日志配置文件的名称。 |
| StorageAccountId |否 |应该将活动日志保存到其中的存储帐户的资源 ID。 |
| serviceBusRuleId |否 |服务总线命名空间（需在其中创建事件中心）的服务总线规则 ID。 是以下格式的字符串：`{service bus resource ID}/authorizationrules/{key name}`。 |
| Location |是 |要为其收集活动日志事件的逗号分隔区域的列表。 |
| RetentionInDays |是 |事件的保留天数，介于 1 到 2147483647 之间。 值为零时，将无限期（永久）存储日志。 |
| 类别 |否 |应收集的事件类别的逗号分隔列表。 可能值包括：Write、Delete 和 Action。 |

#### <a name="remove-a-log-profile"></a>删除日志配置文件

```powershell
Remove-AzLogProfile -name my_log_profile
```

### <a name="configure-log-profiles-using-the-azure-cli"></a>使用 Azure CLI 配置日志配置文件

#### <a name="get-existing-log-profile"></a>获取现有的日志配置文件

```azurecli
az monitor log-profiles list
az monitor log-profiles show --name <profile name>
```

`name` 属性应为日志配置文件的名称。

#### <a name="add-a-log-profile"></a>添加日志配置文件

```azurecli
az monitor log-profiles create --name <profile name> \
    --locations <location1 location2 ...> \
    --location <location> \
    --categories <category1 category2 ...>
```

有关使用 CLI 创建监视器配置文件的完整文档，请参阅 [CLI 命令参考](/cli/azure/monitor/log-profiles#az-monitor-log-profiles-create)

#### <a name="remove-a-log-profile"></a>删除日志配置文件

```azurecli
az monitor log-profiles delete --name <profile name>
```

## <a name="view-change-history"></a>查看更改历史记录

当查看活动日志，它可以帮助以了解期间所发生的更改事件时间。 您可以查看此信息与更改历史记录。

导航到使用在门户的左侧菜单的活动日志。 选择你想要查看深入到活动日志中的事件。 选择**更改历史记录 （预览版）** 选项卡以查看任何与事件关联的更改。

![更改事件的历史记录列表](./media/activity-logs-overview/change-history-event.png)

如果有与事件相关联的任何更改，将看到可以选择的更改的列表。 此操作将打开**更改历史记录 （预览版）** 页。 此页上，请参阅对资源更改。 您可以看到从下面的示例，我们就能够不仅可看到 VM 大小，但什么以前的 VM 大小更改之前，它已更改为已更改。

![显示差异的更改历史记录页](./media/activity-logs-overview/change-history-event-details.png)

若要了解有关更改历史记录的详细信息，请参阅[获取资源更改](../../governance/resource-graph/how-to/get-resource-changes.md)。

## <a name="next-steps"></a>后续步骤

* [详细了解活动日志（以前称为审核日志）](../../azure-resource-manager/resource-group-audit.md)
* [将 Azure 活动日志流式传输到事件中心](../../azure-monitor/platform/activity-logs-stream-event-hubs.md)
