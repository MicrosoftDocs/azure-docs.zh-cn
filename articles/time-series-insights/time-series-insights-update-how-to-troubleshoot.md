---
title: 对 Azure 时序见解预览版进行诊断和故障排除 | Microsoft Docs
description: 了解如何诊断和排查 Azure 时序见解预览版的问题。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: e4a63bfd4e82147fe3324e146f2aaff8889da87e
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472353"
---
# <a name="diagnose-and-troubleshoot"></a>诊断和故障排除

本文汇总了在使用 Azure 时序见解预览版环境时可能会遇到的多个常见问题。 本文还介绍了每个问题的可能原因和解决方案。

## <a name="problem-i-cant-find-my-environment-in-the-preview-explorer"></a>问题：找不到我的环境中的预览版资源管理器

如果无权访问时序见解环境，则可能会发生此问题。 用户需要读者级别访问角色才能查看其时序见解环境。 若要验证当前访问级别并授予其他访问权限，请在 [Azure 门户](https://portal.azure.com/)中访问时序见解资源上的“数据访问策略”部分。

  [![环境](media/v2-update-diagnose-and-troubleshoot/environment.png)](media/v2-update-diagnose-and-troubleshoot/environment.png#lightbox)

## <a name="problem-no-data-is-seen-in-the-preview-explorer"></a>问题： 预览版资源管理器中不查看任何数据

在 [Azure 时序见解预览版资源管理器](https://insights.timeseries.azure.com/preview)中未看见数据有多个常见原因。

- 事件源可能未接收数据。

    验证事件源（即事件中心或 IoT 中心）是否从标记或实例接收数据。 若要进行验证，请转到 Azure 门户中资源的概览页。

    [![仪表板见解](media/v2-update-diagnose-and-troubleshoot/dashboard-insights.png)](media/v2-update-diagnose-and-troubleshoot/dashboard-insights.png#lightbox)

- 事件源数据不是 JSON 格式。

    时序见解仅支持 JSON 数据。 有关 JSON 示例，请参阅[支持的 JSON 形状](./how-to-shape-query-json.md)。

- 事件源密钥缺少所需权限。

  * 对于 IoT 中心，需提供具有“服务连接”权限的密钥。

    [![配置](media/v2-update-diagnose-and-troubleshoot/configuration.png)](media/v2-update-diagnose-and-troubleshoot/configuration.png#lightbox)

  * 如上图所示，“iothubowner”和“服务”策略均可使用，因为两者都具有“服务连接”权限。
  * 对于事件中心，需提供具有“侦听”权限的密钥。
  
    [![权限](media/v2-update-diagnose-and-troubleshoot/permissions.png)](media/v2-update-diagnose-and-troubleshoot/permissions.png#lightbox)

  * 如上图所示，“读取”和“管理”策略均可使用，因为两者都具有“侦听”权限。

- 提供的使用者组并非时序见解所独有。

    IoT 中心或事件中心注册期间，请指定用于读取数据的使用者组。 请不要共享该使用者组。 如果共享了此使用者组，则基础事件中心会随机自动断开一个读取器的连接。 请提供唯一的使用者组，供时序见解从中读取。

- 在预配时指定的时序 ID 属性不正确、缺失或为 null。

    如果在预配环境时时序 ID 属性配置不正确，则可能会发生此问题。 有关详细信息，请参阅[选择时序 ID 的最佳做法](./time-series-insights-update-how-to-id.md)。 目前无法更新现有时序见解环境来使用其他时序 ID。

## <a name="problem-some-data-shows-but-some-is-missing"></a>问题： 某些数据，但某些缺少

可能在发送数据时没有提供时序 ID。

- 如果在发送事件时有效负载中没有时序 ID 字段，则可能会发生此问题。 有关详细信息，请参阅[支持的 JSON 形状](./how-to-shape-query-json.md)。

- 可能因环境受限而发生此问题。

    > [!NOTE]
    > 目前，时序见解支持的最大引入速率为 6 Mbps。

## <a name="problem-my-event-sources-timestamp-property-name-doesnt-work"></a>问题： 我的事件源的时间戳属性名称不起作用

请确保名称和值符合以下规则：

* Timestamp 属性名称区分大小写。
* 来自事件源的 Timestamp 属性值（采用 JSON 字符串形式）的格式为 `yyyy-MM-ddTHH:mm:ss.FFFFFFFK`。 `“2008-04-12T12:53Z”` 是此类字符串的一个示例。

若要确保捕获 Timestamp 属性名称并让其正常运行，最简单的方法是使用时序见解预览版资源管理器。 在时序见解预览版资源管理器中使用此图表，在提供 Timestamp 属性名称以后选择一个时间段。 右键单击所做的选择，然后选择“浏览事件”选项。 第一个列标头为 Timestamp 属性名称。 它应该有 `($ts)` 位于 `Timestamp` 一词的旁边，而不是：

* `(abc)`，指示时序见解将数据值作为字符串来读取。
* 日历图标，指示时序见解将数据值作为日期/时间来读取。
* `#`，指示时序见解将数据值作为整数来读取。

如果 Timestamp 属性未显式指定，则会将事件的 IoT 中心或事件中心的“排队时间”用作默认的时间戳。

## <a name="problem-i-cant-view-or-edit-my-time-series-model"></a>问题：我无法查看或编辑我的时序模型

- 你可能在访问时序见解 S1 或 S2 环境。

   时序模型仅在 PAYG 环境中受支持。 若要详细了解如何从时序见解预览版资源管理器访问 S1/S2 环境，请参阅[可视化资源管理器中的数据](./time-series-insights-update-explorer.md)。

   [![Access](media/v2-update-diagnose-and-troubleshoot/access.png)](media/v2-update-diagnose-and-troubleshoot/access.png#lightbox)

- 你可能无权查看和编辑此模型。

   用户需要有参与者级别访问权限才能编辑和查看其时序模型。 若要验证当前访问级别并授予其他访问权限，请在 Azure 门户中访问时序见解资源上的“数据访问策略”部分。

## <a name="problem-all-my-instances-in-the-preview-explorer-lack-a-parent"></a>问题： 我预览资源管理器中的所有实例都没有父级

如果环境未定义时序模型层次结构，则可能会发生此问题。 有关详细信息，请参阅[使用时序模型](./time-series-insights-update-how-to-tsm.md)。

  [![时序模型](media/v2-update-diagnose-and-troubleshoot/tsm.png)](media/v2-update-diagnose-and-troubleshoot/tsm.png#lightbox)

## <a name="next-steps"></a>后续步骤

- 阅读[使用时序模型](./time-series-insights-update-how-to-tsm.md)。

- 了解如何[支持的 JSON 形状](./how-to-shape-query-json.md)。
