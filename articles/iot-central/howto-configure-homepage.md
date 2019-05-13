---
title: 配置 Azure IoT Central 应用程序仪表板 |Microsoft Docs
description: 作为生成器中，将了解如何配置默认 Azure IoT Central 应用程序仪表板。
author: dominicbetts
ms.author: dobett
ms.date: 02/13/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: b1edd4777839c947cf8f60bb6857f939553e2603
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467607"
---
# <a name="configure-the-application-dashboard"></a>配置应用程序仪表板

**仪表板**是有权访问该应用程序的用户导航到应用程序的 URL 时将加载的页。 如果选择了任一**示例 Contoso**或**示例 Devkits**应用程序模板创建应用程序中，你的应用程序具有预定义仪表板。 如果选择了**自定义应用程序**应用程序模板，你的仪表板为空。

> [!NOTE]
> 用户还可以[创建其自己的个人仪表板](howto-personalize-dashboard.md)使用而不是默认应用程序仪表板。

## <a name="add-tiles"></a>添加磁贴

下面的屏幕截图显示在从创建的应用程序中的仪表板**示例 Contoso**模板。 若要自定义应用程序的默认仪表板，请选择**编辑**顶部页面右上角。

![基于"示例 Contoso"模板的应用程序的仪表板](media/howto-configure-homepage/image1a.png)

选择**编辑**，将打开仪表板库面板。 库中包含的磁贴和仪表板基元可用于自定义仪表板。

![仪表板库](media/howto-configure-homepage/image2a.png)

例如，您可以添加**的设备设置和属性**磁贴以显示选择的设备的当前设置和属性值。 为此，请先选择“设备模板”，然后选择“设备实例”。 之后，为磁贴添加标题，然后选择要显示的**设置**或**属性**。 以下屏幕截图显示了设置和选择要添加到磁贴的属性。 选择**完成**若要将更改保存到仪表板。

![带有设置和属性详细信息的“配置设备详细信息”窗体](media/howto-configure-homepage/image3a.png)

现在，当操作员查看默认应用程序仪表板时，他们将看到的新磁贴**设置温度**为设备设置：

![带有显示的磁贴设置和属性的“仪表板”选项卡](media/howto-configure-homepage/image4a.png)

你可以浏览库中的其他磁贴类型，若要了解如何进一步自定义默认应用程序仪表板。

## <a name="next-steps"></a>后续步骤

现在，已了解如何配置 Azure IoT Central 的默认应用程序仪表板，你可以：

> [!div class="nextstepaction"]
> [了解如何准备和上传图像](howto-prepare-images.md)
