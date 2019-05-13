---
title: 预配和管理 Azure 时序预览版 | Microsoft Docs
description: 了解如何预配和管理 Azure 时序见解预览版。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: ce24fb8c62432e50fe04de23d2abbee1ec120c6c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471632"
---
# <a name="provision-and-manage-azure-time-series-insights-preview"></a>预配和管理 Azure 时序见解预览版

本文介绍如何使用 [Azure 门户](https://portal.azure.com/)创建和管理 Azure 时序见解预览版环境。

## <a name="overview"></a>概述

Azure 时序见解预览版环境是即用即付 (PAYG) 环境。

预配 Azure 时序见解预览版环境时，，请创建两个 Azure 资源：

* Azure 时序见解预览版环境  
* Azure 存储常规用途 v1 帐户
  
了解[如何规划环境](./time-series-insights-update-plan.md)。

>[!IMPORTANT]
> 对于预览版，请确保你使用的 Azure 存储常规用途 v1 (GPv1) 帐户。

可选择将每个 Azure 时序见解预览版环境与一个事件源关联。 有关详细信息，请阅读[添加事件中心源](./time-series-insights-how-to-add-an-event-source-eventhub.md)和[添加 IoT 中心源](./time-series-insights-how-to-add-an-event-source-iothub.md)。 需要在此步骤提供一个时间戳 ID 属性和一个唯一使用者组。 这样做可确保环境能够访问相应的事件。

预配完成后，可根据业务要求修改访问策略和其他环境属性。

## <a name="create-the-environment"></a>创建环境

以下步骤介绍如何创建 Azure 时序见解预览版环境：

1. 在“SKU”菜单下选择“PAYG”按钮。 提供一个环境名称，并选择要使用的订阅组和资源组。 然后，选择一个支持的位置，以便在其中托管环境。

   [![创建 Azure 时序见解实例。](media/v2-update-manage/manage_three.PNG)](media/v2-update-manage/manage_three.PNG#lightbox)

1. 输入时序 ID。

    >[!NOTE]
    > * 时序 ID 区分大小写且不可变。 （一经设置，不可更改。）
    > * 时序 ID 最多可以是三个键。
    > * 有关选择时序 ID 的详细信息，请阅读[选择时序 ID](./time-series-insights-update-how-to-id.md)。

1. 创建一个 Azure 存储帐户，方法是选择存储帐户名称并指定复制选项。 这样做会自动创建 Azure 存储常规用途 v1 帐户。 该帐户将在之前选择的 Azure 时序见解预览版环境所在的区域中创建。

    [![创建你的实例的 Azure 存储帐户](media/v2-update-manage/manage_five.PNG)](media/v2-update-manage/manage_five.PNG#lightbox)

1. 可以选择添加事件源。

   * 时序见解支持使用 [Azure IoT 中心](./time-series-insights-how-to-add-an-event-source-iothub.md)和 [Azure 事件中心](./time-series-insights-how-to-add-an-event-source-eventhub.md)作为选项。 虽然在创建环境时只能添加单个事件源，但可以在以后添加其他事件源。 最好创建唯一的使用者组，确保所有事件对 Azure 时序见解预览版实例可见。 在添加事件源时，可以选择现有的使用者组，也可以创建新的使用者组。

   * 还应选择相应的时间戳属性。 默认情况下，Azure Time 时序见解对每个事件源使用消息排队时间。

     > [!TIP]
     > 在批处理事件或历史数据上传方案中，消息排队时间可能不是要使用的最佳配置设置。 确保验证在这种情况下你是决定使用还是不使用 Timestamp 属性。

     [![事件源选项卡](media/v2-update-manage/manage_two.PNG)](media/v2-update-manage/manage_two.PNG#lightbox)

1. 确认环境是否已使用所需设置进行预配。

    [![查看 + 创建选项卡](media/v2-update-manage/manage_three.PNG)](media/v2-update-manage/manage_three.PNG#lightbox)

## <a name="manage-the-environment"></a>管理环境

可以使用 Azure 门户管理 Azure 时序见解预览版环境。 以下是使用 Azure 门户管理 PAYG Azure 时序见解预览版环境与管理 S1 或 S2 环境的主要区别：

* Azure 门户的“概览”边栏选项卡在 Azure 时序见解中保持不变，以下方面除外：
  * 删除了容量，因为此概念与 PAYG 环境无关。
  * 添加了时序 ID 属性。 它决定了数据的分区方式。
  * 删除了引用数据集。
  * 显示的 URL 会将你定向到 [Azure 时序见解预览版资源管理器](./time-series-insights-update-explorer.md)。
  * 列出了 Azure 存储帐户名称。

* Azure 门户的“配置”边栏选项卡在 Azure 时序见解预览版中已删除，因为 PAYG 环境不可配置。

* Azure 门户的“参考数据”边栏选项卡在 Azure 时序见解预览版中已删除，因为参考数据不是 PAYG 环境的组件。

[![在 Azure 门户中的时间系列 Insights 预览环境](media/v2-update-manage/manage_four.PNG)](media/v2-update-manage/manage_four.PNG#lightbox)

## <a name="next-steps"></a>后续步骤

- 阅读[规划环境](./time-series-insights-update-plan.md)。

- 了解如何[添加事件中心源](./time-series-insights-how-to-add-an-event-source-eventhub.md)。

- 配置[IoT 中心源](./time-series-insights-how-to-add-an-event-source-iothub.md)。