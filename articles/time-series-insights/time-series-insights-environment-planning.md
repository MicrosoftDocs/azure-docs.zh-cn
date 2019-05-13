---
title: 计划 Azure 时序见解环境的缩放 | Microsoft Docs
description: 本文介绍了在计划 Azure 时序见解环境（包括存储容量、数据保留、入口容量、监视和业务灾难恢复 (BCDR)）时如何遵循最佳做法。
services: time-series-insights
ms.service: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 04/29/2019
ms.custom: seodec18
ms.openlocfilehash: bf1f570319370fab99e2f52086bc81df259e3d35
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236507"
---
# <a name="plan-your-azure-time-series-insights-ga-environment"></a>规划 Azure 时间系列 Insights GA 环境

本文介绍如何规划 Azure 时序见解公开上市 (GA）) 环境基于预期的入口速率和数据保留要求。

## <a name="video"></a>视频

### <a name="learn-more-about-data-retention-in-azuretime-series-insights-and-how-to-plan-for-itbr"></a>了解有关数据保留期 AzureTime 时序见解，以及如何规划它的详细信息。</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

## <a name="best-practices"></a>最佳做法

若要开始使用时序见解，最好要知道没分钟想要推送的数据量以及需要存储数据的时间。  

有关时序见解 SKU 的容量和保留期的详细信息，请参阅[时序见解定价](https://azure.microsoft.com/pricing/details/time-series-insights/)。

为取得长期成功，请考虑以下属性，计划最佳的环境：

- <a href="#understand-storage-capacity">存储容量</a>
- <a href="#understand-data-retention">数据保留期</a>
- <a href="#understand-ingress-capacity">入口容量</a>
- <a href="#shape-your-events">调整您的事件</a>
- <a href="#ensure-you-have-reference-data">确保就地具有引用数据</a>

## <a name="understand-storage-capacity"></a>了解存储容量

默认情况下，Azure 时序见解保留数据基于已预配的存储量 （单位的每个单位的存储时长） 和入口。

## <a name="understand-data-retention"></a>了解数据保留

可配置时序见解环境的“数据保留期”设置，启用最长 400 天的保留期。 时序见解有两种模式，一种针对确保环境具有最新数据进行优化（默认情况下启用），另一种针对确保满足保留期限制进行优化，如果达到该环境的整体存储容量，则暂停流入。  可在 Azure 门户的环境配置页面中调整保留期并在这两种模式之间切换。

可在时序见解环境中配置最长 400 天的数据保留。

### <a name="configure-data-retention"></a>配置数据保留

1. 在 [Azure 门户](https://portal.azure.com)中，选择时序见解环境。

1. 在时序见解环境页面的“设置”标题下，选择“配置”。

1. 在“数据保留时间(天)”对话框中，输入 1 到 400 之间的值。

   [![配置保留期](media/environment-mitigate-latency/configure-retention.png)](media/environment-mitigate-latency/configure-retention.png#lightbox)

> [!TIP]
> 了解有关通过查看实现适当的数据保留策略的详细信息[如何配置保留期](./time-series-insights-how-to-configure-retention.md)。

## <a name="understand-ingress-capacity"></a>了解入口容量

计划中另一个需要注意的区域是入口容量，它是每分钟分配的派生。

从限制的角度看，数据包大小为 32 KB 的入口数据包视为 32 个事件，每个事件大小为 1 KB。 允许的最大事件大小为 32 KB；大于 32 KB 的数据包会被截断。

下表总结了每个 SKU 的入口容量：

|SKU  |事件数 / 月 / 单位  |事件大小 / 月 / 单位  |事件计数 / 分钟 / 单位  | 大小 / 分钟 / 单位   |
|---------|---------|---------|---------|---------|
|S1     |   3000 万     |  30 GB     |  720    |  720 KB   |
|S2     |   3 亿    |   300 GB   | 7,200   | 7,200 KB  |

可以在单一环境中，将 S1 或 S2 SKU 的容量增加到 10 个单位。 不可在 S1 环境与 S2 环境之间迁移。

对于入口容量，首先应该确定每月所需的入口总量。 然后确定每分钟所需的量，这也是限制和延迟会产生影响的因素。

如果数据入口中数据激增，且持续时间少于 24 小时，则时序见解可以用两倍于以上所列速度的入口速率“赶上”。

例如，如果具有一个 S1 SKU 并且入口数据的速率为每分钟 720 个事件，而激增的速率为 1440 个事件（或更少）且持续时间不到 1 小时，则不会对环境造成明显的延迟。 但是，如果速率大于每分钟 1440 个事件超过一小时，则可能会发生数据延迟，这可在环境中查看并查询。

可能无法提前知道想要推送多少数据。 在这种情况下，可以在 Azure 门户的 [Azure IoT 中心](https://docs.microsoft.com/azure/iot-hub/iot-hub-metrics)和 [Azure 事件中心](https://blogs.msdn.microsoft.com/cloud_solution_architect/2016/05/25/using-the-azure-rest-apis-to-retrieve-event-hub-metrics/)中查找数据遥测。 这些遥测有助于确定如何预配环境。 在 Azure 门户使用相应事件源的“指标”页面，查看数据遥测。 了解事件源指标后，可以更有效地计划和预配时序见解环境。

### <a name="calculate-ingress-requirements"></a>计算入口需求

- 确认入口容量高于平均每分钟的速率，并且环境足够大，能够在 1 小时内处理相当于两倍容量的预期入口。

- 如果入口激增持续时间超过 1 小时，则使用激增速率作为平均值，并预配容量足以处理激增速率的环境。

### <a name="mitigate-throttling-and-latency"></a>缓解限制和延迟

有关如何避免限制和延迟信息，请阅读有关如何对[降低延迟和限制](time-series-insights-environment-mitigate-latency.md)。

## <a name="shape-your-events"></a>塑造事件

务必要确保向 TSI 发送事件的方法支持在预配的环境的大小 （相反，您可以映射为 TSI 可以读取的事件数环境的大小和每个事件的大小）。 同样，请务必考虑在查询数据时你可能希望对其进分片以及作为筛选依据的属性。

> [!TIP]
> 查看 JSON 成型文档中的[将事件发送](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-send-events)。

## <a name="ensure-you-have-reference-data"></a>确保有引用数据

一个**引用数据集**是增加事件源中的事件的项的集合。 时序见解入口引擎将事件源中的每个事件与参考数据集中的相应数据行联接到一起。 然后即可使用此增强的事件进行查询。 该联接基于在参考数据集中定义的主键列。

请注意，参考数据不以追溯方式进行联接。 这意味着在配置并上传数据后，只会将当前和将来的入口数据匹配并联接到参考数据集。  如果打算向 TSI 发送大量历史数据并且不先在 TSI 中上传或创建参考数据，则可能必须重做你的工作（这意味着很无趣）。  

若要详细了解如何在 TSI 中创建、上传和管理参考数据，请转至[参考数据集文档](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-add-reference-data-set)。

[!INCLUDE [business-disaster-recover](../../includes/time-series-insights-business-recovery.md)]

## <a name="next-steps"></a>后续步骤

- 开始创建[在 Azure 门户中的新时序见解环境](time-series-insights-get-started.md)。

- 了解如何[添加事件中心事件源](time-series-insights-how-to-add-an-event-source-eventhub.md)向时序见解。

- 阅读有关如何[配置 IoT 中心事件源](time-series-insights-how-to-add-an-event-source-iothub.md)。
