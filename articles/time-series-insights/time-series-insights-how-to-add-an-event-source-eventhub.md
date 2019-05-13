---
title: 向 Azure 时序见解添加事件中心事件源 | Microsoft Docs
description: 本文介绍如何将已连接至 Azure 事件中心的事件源添加到时序见解环境。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 05/01/2019
ms.custom: seodec18
ms.openlocfilehash: 41d3e72d978a210c2d68365ade5d8cb42c24aad5
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147611"
---
# <a name="add-an-event-hub-event-source-to-your-time-series-insights-environment"></a>将事件中心事件源添加到时序见解环境

本文介绍如何使用 Azure 门户将从 Azure 事件中心读取数据的事件源添加到 Azure 时序见解环境。

> [!NOTE]
> 本文中介绍的步骤适用于时序见解 GA 版和时序见解预览版环境。

## <a name="prerequisites"></a>必备组件

- 创建时序见解环境中所述[创建 Azure 时序见解环境](./time-series-insights-update-create-environment.md)。
- 创建事件中心。 请参阅[使用 Azure 门户创建事件中心命名空间和事件中心](../event-hubs/event-hubs-create.md)。
- 事件中心必须有发送进来的活动消息事件。 了解如何[使用.NET Framework 将事件发送到 Azure 事件中心](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md)。
- 在事件中心创建专用使用者组，以供时序见解环境使用。 每个时序见解事件源都必须具有自己的专用使用者组，该组不与其他使用者共享。 如果多个读取器使用同一使用者组中的事件，则所有读取器都可能出现故障。 有一项限制，即每个事件中心只能有 20 个使用者组。 有关详细信息，请参阅[事件中心编程指南](../event-hubs/event-hubs-programming-guide.md)。

### <a name="add-a-consumer-group-to-your-event-hub"></a>将使用者组添加到事件中心

应用程序使用使用者组从 Azure 事件中心提取数据。 提供专用使用者组，仅供此时序见解环境使用，以可靠地从事件中心读取数据。

若要将新使用者组添加到事件中心，请执行以下操作：

1. 在 Azure 门户中，找到并打开自己的事件中心。

1. 在“实体”下选择“使用者组”，然后选择“使用者组”。

   [![事件中心-添加使用者组](media/time-series-insights-how-to-add-an-event-source-eventhub/5-event-hub-consumer-group.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/5-event-hub-consumer-group.png#lightbox)

1. 在“使用者组”页上，输入一个新的唯一值作为**名称**。  创建新的事件源时，请在时序见解环境中使用此相同名称。

1. 选择“创建”。

## <a name="add-a-new-event-source"></a>添加新的事件源

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 查找现有时序见解环境。 在左侧菜单中选择“所有资源”，然后选择时序见解环境。

1. 在“环境拓扑”下，依次选择“事件源”、“添加”。

   [![在事件源下选择添加按钮](media/time-series-insights-how-to-add-an-event-source-eventhub/1-event-sources.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/1-event-sources.png#lightbox)

1. 输入一个值作为特定于时序见解环境的“事件源名称”，如 **event-stream**。

1. 对于“源”，选择“事件中心”。

1. 选择适当的值作为“导入”选项：
   - 如果在其中一个订阅中有现有的事件中心，请选择“从可用订阅使用事件中心”。 此选项是最简单的方法。
   - 如果事件中心在订阅外部，或者你希望选择高级选项，请选择“手动提供事件中心设置”。

   [![在新事件源窗格中，输入前三个参数值](media/time-series-insights-how-to-add-an-event-source-eventhub/2-import-option.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/2-import-option.png#lightbox)

1. 下表介绍的属性是“通过可用订阅使用事件中心”选项所需的：

   [![订阅和事件中心详细信息](media/time-series-insights-how-to-add-an-event-source-eventhub/3-new-event-source.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/3-new-event-source.png#lightbox)

   | 属性 | 描述 |
   | --- | --- |
   | 订阅 ID | 选择其中创建有此事件中心的订阅。
   | 服务总线命名空间 | 选择包含事件中心的 Azure 服务总线命名空间。
   | 事件中心名称 | 选择事件中心的名称。
   | 事件中心策略名称 | 选择共享访问策略。 可以在事件中心的“配置”选项卡上创建共享访问策略。每个共享访问策略具有名称、所设权限以及访问密钥。 事件源的共享访问策略必须具有“读取”权限。
   | 事件中心策略密钥 | 密钥值可能已预先填充。
   | 事件中心使用者组 | 从事件中心读取事件的使用者组。 强烈建议为事件源使用专用的使用者组。 |
   | 事件序列化格式 | 目前，JSON 是唯一可用的序列化格式。 事件消息必须采用此格式，否则将无法读取任何数据。 |
   | 时间戳属性名称 | 若要确定此值，需要了解发送到事件中心的消息数据的消息格式。 此值是消息数据中你想要用作事件时间戳的特定事件属性的“名称”。 该值区分大小写。 如果留空，则事件源中的“事件排队时间”将用作事件时间戳。 |

1. 下表介绍“手动提供事件中心设置”选项所需的属性：

   | 属性 | 描述 |
   | --- | --- |
   | 订阅 ID | 其中创建有此事件中心的订阅。
   | 资源组 | 在其中创建此事件中心的资源组。
   | 服务总线命名空间 | 服务总线命名空间是包含一组消息传递实体的容器。 创建新的事件中心后，还创建了 Service Bus 命名空间。
   | 事件中心名称 | 事件中心的名称。 创建事件中心时，还为它提供了一个特定的名称。
   | 事件中心策略名称 | 共享访问策略。 可以在事件中心的“配置”选项卡上创建共享访问策略。每个共享访问策略具有名称、所设权限以及访问密钥。 事件源的共享访问策略必须具有“读取”权限。
   | 事件中心策略密钥 | 用于对服务总线命名空间的访问权限进行身份验证的共享访问密钥。 在此处输入主密钥或辅助密钥。
   | 事件中心使用者组 | 从事件中心读取事件的使用者组。 强烈建议为事件源使用专用的使用者组。
   | 事件序列化格式 | 目前，JSON 是唯一可用的序列化格式。 事件消息必须采用此格式，否则将无法读取任何数据。 |
   | 时间戳属性名称 | 若要确定此值，需要了解发送到事件中心的消息数据的消息格式。 此值是消息数据中你想要用作事件时间戳的特定事件属性的“名称”。 该值区分大小写。 如果留空，则事件源中的“事件排队时间”将用作事件时间戳。 |

1. 添加已添加到事件中心的专用时序见解使用者组名称。

1. 选择“创建”。

   [![选择创建](media/time-series-insights-how-to-add-an-event-source-eventhub/4-create-button.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/4-create-button.png#lightbox)

   创建事件源以后，时序见解就会自动将数据流式传输到环境中。

## <a name="next-steps"></a>后续步骤

* [定义数据访问策略](time-series-insights-data-access.md)，以保护数据。

* [发送事件](time-series-insights-send-events.md)到事件源。

* 在[时序见解资源管理器中访问环境](https://insights.timeseries.azure.com)。
