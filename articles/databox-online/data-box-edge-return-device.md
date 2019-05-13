---
title: 返回 Azure 数据框边缘设备 |Microsoft Docs
description: 介绍如何返回 Azure 数据框边缘设备并删除设备的顺序。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 05/07/2019
ms.author: alkohli
ms.openlocfilehash: 9aeae0ab68d809b36a3316054f12a5a9657721f1
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65468599"
---
# <a name="return-your-azure-data-box-edge-device"></a>返回 Azure 数据框边缘设备

本文介绍如何擦除的数据，然后返回 Azure 数据框边缘设备。 返回设备后，您还可以删除与设备关联的资源。

在本文中，学习如何：

> [!div class="checklist"]
> * 擦除设备上的数据磁盘上的数据
> * 打开支持票证以返回你的设备
> * 设备设置包和安排提货
> * 删除 Azure 门户中的资源

## <a name="erase-data-from-the-device"></a>从设备中擦除数据

若要擦除的设备的数据磁盘上的数据，您需要重置你的设备。 你可以重置你的设备使用本地 web UI 或 PowerShell 接口。

重置之前，如果需要创建在设备上本地数据的副本。 可以将数据从设备复制到 Azure 存储容器中。

若要重置你的设备使用本地 web UI，请执行以下步骤。

1. 在本地 web UI 中，转到**维护 > 设备重置**。
2. 选择**重置设备**。

    ![重置设备](media/data-box-edge-return-device/device-reset-1.png)

3. 如果出现确认提示，请查看警告，然后选择**是**以继续。

    ![确认重置](media/data-box-edge-return-device/device-reset-2.png)  

重置将擦除设备数据磁盘上的数据。 具体取决于你的设备上的数据量，此过程大约需要 30 到 40 分钟。

或者，连接到设备以及如何使用 PowerShell 界面`Reset-HcsAppliance`cmdlet 要清除的数据磁盘中的数据。 有关详细信息，请参阅[重置设备](data-box-edge-connect-powershell-interface.md#reset-your-device)。

> [!NOTE]
> - 如果您要交换或升级到新设备，我们建议仅后收到了新的设备重置你的设备。
> - 仅重置的设备中删除设备的所有本地数据。 在云中的数据不会被删除，并收集[费用](https://azure.microsoft.com/pricing/details/storage/)。 此数据需要删除使用之类的云存储管理工具单独[Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)。

## <a name="open-a-support-ticket"></a>打开支持票证

若要开始返回过程，请执行以下步骤。

1. 与 Microsoft 支持部门，该值指示你想要返回的设备打开支持票证。 选择问题类型为**数据框边缘硬件**。

    ![打开支持票证](media/data-box-edge-return-device/open-support-ticket-1.png)  

2. Microsoft 支持工程师将与你联系。 提供寄送详细信息。
3. 如果您需要返回传送框中，可以请求它。 答案**是**问题**需要返回一个空框**。


## <a name="schedule-a-pickup"></a>安排提货

1. 关闭设备。 在本地 Web UI 中，转到“维护”>“电源设置”。
2. 选择**关闭**。 当系统提示确认，单击**是**以继续。 有关详细信息，请参阅[管理能力](data-box-gateway-manage-access-power-connectivity-mode.md#manage-power)。
3. 拔下电源线，并从设备中删除所有网络电缆。
4. 使用你自己的框或从 Azure 收到的空框准备发货包。 将设备和与在框中的设备附带的电源线。
5. 粘贴从 Azure 收到包的发货标签。
6. 与区域运营商安排提货。 如果在美国返回设备，您的运营商为 UPS。 安排提货：

    1. 给本地 UPS 打电话（特定于国家/地区的免费电话号码）。
    2. 在您调用中，用引号括起来反向发货跟踪号上打印标签所示。
    3. 如果不带引号的跟踪号，UPS 将要求您在提取期间支付额外费用。

    而不是安排提货，您还可以删除数据框边缘最接近放置位置关闭。

## <a name="delete-the-resource"></a>删除资源

在 Azure 数据中心收到设备后，设备检查的损坏或被篡改的任何迹象。

- 如果设备到达时保持不变并保持良好状态，则会停止该资源计费测定仪。 Microsoft 支持部门会联系你以确认设备已返回。 然后，可以删除与 Azure 门户中的设备关联的资源。
- 如果设备到达时损坏严重，可能会应用罚金。 有关详细信息，请参阅[上丢失或损坏设备的常见问题](https://azure.microsoft.com/pricing/details/databox/edge/)并[产品服务条款](https://www.microsoft.com/licensing/product-licensing/products)。  


可以删除 Azure 门户中的设备：
-   下订单后之前由 Microsoft 准备设备。
-   向 Microsoft 返回设备后，它在 Azure 数据中心，传递物理检查，Microsoft 支持部门调用以确认设备已返回。

如果你已激活设备的另一个订阅或位置，则 Microsoft 将在一个工作日内你的订单移动到新订阅或位置。 移动顺序后，可以删除此资源。


执行以下步骤来删除该设备和 Azure 门户中的资源。

1. 在 Azure 门户中，转到资源然后**概述**。 从命令栏中，选择**删除**。

    ![选择“删除”](media/data-box-edge-return-device/delete-resource-1.png)

2. 在中**删除设备**边栏选项卡中，键入你想要删除选择的设备的名称**删除**。

    ![确认删除](media/data-box-edge-return-device/delete-resource-2.png)

收到通知后设备和已成功删除关联的资源。

## <a name="next-steps"></a>后续步骤

- 了解如何[管理带宽](data-box-edge-manage-bandwidth-schedules.md)。
