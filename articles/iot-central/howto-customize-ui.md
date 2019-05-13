---
title: 自定义 Azure IoT 中心 UI |Microsoft Docs
description: 如何自定义你的 Azure IoT 中心应用程序的主题和帮助链接
author: dominicbetts
ms.author: dobett
ms.date: 04/25/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: 0256396cd228898f3852772b113e6064a0656746
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237652"
---
# <a name="customize-the-azure-iot-central-ui"></a>自定义 Azure IoT 中心 UI 

*本文适用于管理员。*

IoT 中心，可以通过应用自定义主题和修改为指向您自己的自定义帮助的资源的帮助链接来自定义你的应用程序的 UI。 下面的屏幕截图显示了使用标准主题的页：

![标准 IoT 中心主题](./media/howto-customize-ui/standard-ui.png)

下面的屏幕截图显示了突出显示的自定义 UI 元素中使用自定义的屏幕截图的页：

![自定义 IoT 中心主题](./media/howto-customize-ui/themed-ui.png)

## <a name="create-theme"></a>创建主题

若要创建自定义主题，请导航到**自定义应用程序**页面**管理**部分：

![IoT Central 主题](./media/howto-customize-ui/themes.png)

在此页上，可以自定义应用程序的以下方面：

### <a name="application-logo"></a>应用程序徽标

PNG 图像，不大于 1 mb 以内，带有透明背景。 IoT Central 应用程序标题栏上左侧显示此徽标。

如果徽标图像包括应用程序的名称，则可以隐藏的应用程序名称文本。 有关详细信息，请参阅[管理应用程序设置](./howto-administer.md#manage-application-settings)。

### <a name="browser-icon-favicon"></a>浏览器图标 (favicon)

PNG 图像，不能大于 32 x 32 像素，透明背景。 Web 浏览器可以使用此映像在地址栏、 历史记录、 书签和浏览器选项卡中。

### <a name="browser-colors"></a>浏览器的颜色

您可以更改页标题的颜色和用于重音按钮和其他突出显示的颜色。 使用一个六个字符的十六进制颜色值格式`##ff6347`。 有关详细信息**十六进制值**颜色表示法，请参阅[HTML 颜色](https://www.w3schools.com/html/html_colors.asp)。

> [!NOTE]
> 你可以始终还原为默认选项上**自定义应用程序**页。

### <a name="changes-for-operators"></a>运算符的更改

如果管理员将创建一个自定义主题，则运算符和应用程序的其他用户可以不能再选择中的一个主题**设置**。

## <a name="replace-help-links"></a>将帮助链接

若要向你的运算符和其他用户提供自定义帮助的信息，可以修改应用程序中的链接**帮助**菜单。

若要修改的帮助链接，导航到**自定义帮助**页面**管理**部分：

![自定义 IoT Central 帮助链接](./media/howto-customize-ui/help-links.png)

此外可以将新条目添加到帮助菜单，并删除默认项：

![自定义的 IoT Central 帮助](./media/howto-customize-ui/custom-help.png)

> [!NOTE]
> 你可以始终还原为默认帮助链接上**自定义帮助**页。

## <a name="next-steps"></a>后续步骤

现在，已了解如何自定义 IoT Central 应用程序中的用户界面，下面是一些建议的后续步骤：

> [!div class="nextstepaction"]
> [管理你的应用程序](./howto-administer.md)
> [配置应用程序仪表板](./howto-configure-homepage.md)