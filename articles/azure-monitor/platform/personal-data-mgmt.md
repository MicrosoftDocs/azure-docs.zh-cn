---
title: 关于 Azure Log Analytics 中存储的个人数据的指南 | Microsoft Docs
description: 本文介绍如何管理存储在 Azure Log Analytics 中的个人数据以及识别和删除这些数据的方法。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 05/18/2018
ms.author: magoedte
ms.openlocfilehash: 6254a32349a0f7c743c3fb4993080ca3437c8276
ms.sourcegitcommit: 9f87a992c77bf8e3927486f8d7d1ca46aa13e849
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/28/2018
ms.locfileid: "53810264"
---
# <a name="guidance-for-personal-data-stored-in-log-analytics-and-application-insights"></a>存储在 Log Analytics 和 Application Insights 中的个人数据指南

Log Analytics 是一种可能存有个人数据的数据存储。 Application Insights 将其数据存储在 Log Analytics 分区中。 本文讨论通常在 Log Analytics 和 Application Insights 中的哪些位置可找到此类数据，以及可用于处理此类数据的功能。

> [!NOTE]
> 在本文中，_日志数据_指的是发送到 Log Analytics 工作区的数据，而 _应用程序数据_ 指的是 Application Insights 收集的数据。

[!INCLUDE [gdpr-dsr-and-stp-note](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="strategy-for-personal-data-handling"></a>个人数据处理策略

下面介绍了一些可行的方法，不过最终将由你和你的公司来决定用于处理私人数据（如有）的策略。 我们从技术角度出发，按偏好顺序（偏好度从高到低）列出了这些方法：

* 如果可行，应停止收集、混淆或匿名处理私人数据，或停止以其他方式调整所收集的数据，使其不再被视为“私人数据”。 这是_目前为止_的首选方法，无需创建非常昂贵且影响很大的数据处理策略。
* 如果不可行，请尝试规范化数据，减少对数据平台和性能的影响。 例如，不记录显式用户 ID，而是创建查找数据，将用户名及其详细信息关联到可随后在其他位置记录的内部 ID。 这样一来，如果某个用户要求删除其个人信息，则只需删除查找表中对应于该用户的行就足够了。 
* 最后，如果必须收集私人数据，请就清除 API 路径和现有查询 API 路径构建相关流程，以履行在导出和删除与用户关联的任何私人数据时可能需要承担的任何义务。 

## <a name="where-to-look-for-private-data-in-log-analytics"></a>在 Log Analytics 中的何处查找私人数据？

Log Analytics 是十分灵活的存储，可在规定数据架构的同时允许使用自定义值替代每个字段。 此外，还可引入任何自定义架构。 因此，无法确切知道可在特定工作区的哪些位置找到私人数据。 但是，不妨从清单中的以下位置着手：

### <a name="log-data"></a>日志数据

* *IP 地址*：Log Analytics 跨许多不同的表收集各种 IP 信息。 例如，以下查询显示过去 24 小时内从中收集 IPv4 地址的所有表：
    ```
    search * 
    | where * matches regex @'\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' //RegEx originally provided on https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp
    | summarize count() by $table
    ```
* *用户 ID*：可在各种解决方案和表中找到用户 ID。 可使用以下搜索命令在整个数据集中查找特定用户名：
    ```
    search "[username goes here]"
    ```
请记住，不仅要查找用户可读的用户名，还要查找可直接追溯到特定用户的 GUID！
* *设备 ID*：与用户 ID 一样，设备 ID 有时被视为“私人数据”。 可使用上面针对用户 ID 列出的方法来识别可能存在此问题的表。 
* *自定义数据*：Log Analytics 允许使用各种方法进行收集：自定义日志和自定义字段、[HTTP 数据收集器 API](../../azure-monitor/platform/data-collector-api.md) 以及作为系统事件日志一部分收集的自定义数据。 所有这些数据都很有可能包含私人数据，应该进行检查以验证是否存在任何此类数据。
* *解决方案捕获的数据*：由于解决方案机制是开放式的，因此建议查看解决方案生成的所有表以确保符合性。

### <a name="application-data"></a>应用程序数据

* *IP 地址*：虽然 Application Insights 在默认情况下会将所有 IP 地址字段混淆成“0.0.0.0”，但为了保留会话信息，通常会将此值替代为实际的用户 IP。 可以使用下面的 Analytics 查询来查找特定的表，此类表的 IP 地址列中包含的值在过去 24 小时内不是“0.0.0.0”：
    ```
    search client_IP != "0.0.0.0"
    | where timestamp > ago(1d)
    | summarize numNonObfuscatedIPs_24h = count() by $table
    ```
* *用户 ID*：默认情况下，Application Insights 会使用为用户随机生成的 ID，以便进行会话跟踪。 不过，这些字段常常会被替代，改为存储与应用程序更相关的 ID。 例如：用户名、AAD GUID 等。这些 ID 通常会被视为范围内的个人数据，因此应处理得当。 我们的建议始终是尝试对这些 ID 进行混淆或匿名处理。 通常可以在其中发现这些值的字段包括：session_Id、user_Id、user_AuthenticatedId、user_AccountId、customDimensions。
* *自定义数据*：Application Insights 允许向任何数据类型追加一组自定义维度。 这些维度可以是任何数据。 使用以下查询来确定在过去 24 小时内收集的任何自定义维度：
    ```
    search * 
    | where isnotempty(customDimensions)
    | where timestamp > ago(1d)
    | project $table, timestamp, name, customDimensions 
    ```
* *内存中和传输中数据*：Application Insights 会跟踪异常、请求、依赖项调用和跟踪。 私人数据通常可以在代码和 HTTP 调用级别收集。 查看异常、请求、依赖项和跟踪表中是否存在任何此类数据。 尽可能使用[遥测初始值设定项](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling)来混淆该数据。
* *Snapshot Debugger 捕获*：使用 Application Insights 中的 [Snapshot Debugger](https://docs.microsoft.com/azure/application-insights/app-insights-snapshot-debugger) 功能时，只要在应用程序的生产实例上捕获某个异常，就可以收集调试快照。 快照会公开导致异常的完整堆栈跟踪，以及堆栈中每一步的本地变量的值。 遗憾的是，此功能不允许选择性地删除吸附点，也不允许以编程方式访问快照中的数据。 因此，如果默认的快照保留率不满足符合性要求，建议关闭此功能。

## <a name="how-to-export-and-delete-private-data"></a>如何导出和删除私人数据

如前面的[个人数据处理策略](#strategy-for-personal-data-handling)部分所述，如果可行， __强烈__ 建议重新构建数据收集策略，以禁止收集、混淆或匿名处理私人数据，或禁止以其他方式对其进行修改，使其不再被视为“私人数据”。 处理数据首先需要你和你的团队定义和自动化策略，为客户构建一个界面，以便与数据交互，并且还需持续承担维护成本。 此外，Log Analytics 和 Application Insights 的计算成本很高，大量并发查询或清除 API 调用可能会对与 Log Analytics 功能的所有其他交互产生负面影响。 尽管如此，在某些情况下，确实必须收集私人数据。 对于这些情况，数据应按本部分所述进行处理。

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-and-export"></a>查看和导出

对于查看和导出数据请求，应使用 [Log Analytics 查询 API](https://dev.loganalytics.io/) 或 [Application Insights 查询 API](https://dev.applicationinsights.io/quickstart)。 将数据形状转换为适当形状以提供给用户时，将由你实现相关逻辑。 [Azure Functions](https://azure.microsoft.com/services/functions/) 非常适合托管此类逻辑。

### <a name="delete"></a>删除

> [!WARNING]
> Log Analytics 中的删除操作具有破坏性且不可逆！ 执行时请特别小心。

我们已在隐私处理中提供*清除* API 路径。 使用此路径会带来一定的风险和潜在的性能影响，并有可能导致 Log Analytics 数据的所有聚合、度量和其他方面发生偏差，因此应谨慎使用。 有关处理私人数据的替代方法，请参阅[个人数据处理策略](#strategy-for-personal-data-handling)部分。

清除是一项高特权操作，如果未向 Azure 中的应用或用户显式授予 Azure 资源管理器中的某个角色，则任何应用或用户（甚至包括资源所有者）都无权执行该操作。 此角色为_数据清除程序_，由于可能会丢失数据，应谨慎委托。 

一旦分配该 Azure 资源管理器角色，就有两个新的 API 路径可用： 

#### <a name="log-data"></a>日志数据

* [POST purge](https://docs.microsoft.com/rest/api/loganalytics/workspaces%202015-03-20/purge) - 使用一个对象来指定要删除的数据的参数，并返回引用 GUID 
* GET purge status：POST purge 调用将返回“x-ms-status-location”标头，其中包含一个 URL，可以调用该 URL 来确定清除 API 的状态。 例如：

    ```
    x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/Microsoft.OperatonalInsights/workspaces/[WorkspaceName]/operations/purge-[PurgeOperationId]?api-version=2015-03-20
    ```

> [!IMPORTANT]
>  虽然我们预计绝大多数清除操作完成起来都比我们的 SLA 快得多，但由于这些操作会对 Log Analytics 使用的数据平台造成严重影响，因此**我们将完成清除操作所需的正式 SLA 设置为 30 天**。 

#### <a name="application-data"></a>应用程序数据

* [POST purge](https://docs.microsoft.com/rest/api/application-insights/components/purge) - 使用一个对象来指定要删除的数据的参数，并返回引用 GUID
* GET purge status：POST purge 调用将返回“x-ms-status-location”标头，其中包含一个 URL，可以调用该 URL 来确定清除 API 的状态。 例如：

   ```
   x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/microsoft.insights/components/[ComponentName]/operations/purge-[PurgeOperationId]?api-version=2015-05-01
   ```

> [!IMPORTANT]
>  虽然绝大多数清除操作完成起来会比 SLA 快得多，但由于其对 Application Insights 使用的数据平台造成的严重影响，因此**将完成清除操作所需的正式 SLA 设置为 30 天**。

## <a name="next-steps"></a>后续步骤
- 若要详细了解如何收集、处理和保护 Log Analytics 数据，请参阅 [Log Analytics 数据安全性](../../azure-monitor/platform/data-security.md)。
- 若要详细了解如何收集、处理和保护 Application Insights 数据，请参阅 [Application Insights 数据安全性](../../azure-monitor/app/data-retention-privacy.md)。
