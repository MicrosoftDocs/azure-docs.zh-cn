---
title: include 文件
description: include 文件
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: kingdomofends
ms.author: adgera
ms.date: 04/29/2019
ms.openlocfilehash: cf07d19966c08a63b9aa50475622aa0a1e5e1600
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236497"
---
## <a name="business-disaster-recovery"></a>业务灾难恢复

本部分介绍 Azure 时序见解，使应用和服务，即使当灾难发生时运行的功能 (**业务灾难恢复**)。

### <a name="high-availability"></a>高可用性

作为一项 Azure 服务，时序见解提供某些**高可用性**功能在 Azure 区域级别使用冗余。 例如，Microsoft Azure 支持通过 Azure 的灾难恢复功能**跨区域可用性**功能。

通过 Azure （和还可用于任何时序见解实例） 提供的其他高可用性功能包括：

1. **故障转移**：Azure 提供了[异地复制和加载平衡](https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region)。
1. **数据恢复**并**存储恢复**:Azure 提供了[多个选项来保留和恢复数据](https://docs.microsoft.com/azure/architecture/resiliency/recovery-data-corruption)。
1. **站点恢复**功能通过[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/)。

请确保启用这些 Azure 功能为你的设备和用户提供全局跨区域的、 高可用性。

> [!NOTE]
> 如果 Azure 配置为启用**跨区域可用性**，不需要在 Azure 时序见解中任何其他跨区域可用性配置。

### <a name="iot-and-event-hubs"></a>IoT 和事件中心

某些 Azure IoT 服务还包括内置业务灾难恢复功能：

1. [IoT 中心高可用性灾难恢复](https://docs.microsoft.com/azure/iot-hub/iot-hub-ha-dr)包括内部区域冗余。
1. [事件中心策略](https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr)。
1. [Azure 存储冗余](https://docs.microsoft.com/azure/storage/common/storage-redundancy)。

与这些其他服务集成时序见解提供附加的灾难恢复的机会。 例如，发送到事件中心的遥测数据可能会保存到 Azure Blob 存储数据库的备份。

### <a name="time-series-insights"></a>时序见解

有几种方法来保留时序见解数据、 应用和服务继续运行，则要中断。 此外可能会确定你的 Azure 时序环境的完整、 重复的备份副本是必需的：

1. 作为特定 TSI**故障转移实例**重定向数据和流量。
1. 出于审核和数据暂留目的。

一般情况下，复制 TSI 环境的最佳方法是在备份 Azure 区域中创建第二个 TSI 环境。 事件是从您的主要事件源也发送到此辅助环境。 请确保要使用第二个、 专用的使用者组，并遵循该源业务灾难恢复指导原则 （在上面提供）。

具体而言，若要创建重复的环境：

1. 在第二个区域中创建一个环境 ([在 Azure 门户中创建新的时序见解环境](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-get-started))。
1. 为事件源创建另一个专用使用者组。
1. 该事件源连接到新环境并确保指定第二个、 专用使用者组。
1. 查看时序见解[IoT 中心](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub)并[事件中心](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-data-access)文档。

第三步：

* 如果主要区域在灾难事件期间受到影响，请将操作重新路由到备用时序见解环境。
* 使用第二个区域来备份和恢复所有 TSI 遥测数据和查询数据。

> [!IMPORTANT]
> * 请注意，可能会进行故障转移时遇到延迟。
> * 故障转移也可能导致在消息处理操作都将重新发送的瞬间出现峰值。
> * 有关详细信息，请参阅[缓解时序见解中的延迟](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-environment-mitigate-latency)。