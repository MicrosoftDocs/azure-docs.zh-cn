---
title: 在 Azure IoT Central 应用程序中创建和管理遥测规则 | Microsoft Docs
description: 可以通过 Azure IoT Central 遥测规则近乎实时地监视设备并自动调用操作（例如在触发规则时发送电子邮件）。
author: ankitgupta
ms.author: ankitgup
ms.date: 02/02/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: de4fba88784f41485b446f0f31cccb9a43092a3c
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464364"
---
# <a name="create-a-telemetry-rule-and-set-up-notifications-in-your-azure-iot-central-application"></a>在 Azure IoT Central 应用程序中创建遥测规则并设置通知

*本文适用于操作员、构建者和管理员。*

可以使用 Azure IoT Central 对连接的设备进行远程监视。 可以通过 Azure IoT Central 规则近乎实时地监视设备并自动调用操作（例如发送电子邮件或触发 Microsoft Flow）。 只需几次单击即可定义用于监视设备数据的条件并配置相应操作。 本文介绍如何创建规则来监视设备发送的遥测数据。

设备可以使用遥测度量从设备发送数字数据。 当选定的设备遥测超过指定的阈值时，会触发遥测规则。

## <a name="create-a-telemetry-rule"></a>创建遥测规则

若要创建遥测规则，必须在设备模板中至少定义一个遥测度量。 本示例使用一个发送温度和湿度遥测数据的冷冻贩卖机设备。 该规则监视设备报告的温度，并在温度超过 80 度时发送电子邮件。

1. 使用**设备模板**页上，导航到要为其添加的规则的设备模板。

1. 如果尚未创建任何规则，则会看到以下屏幕：

    ![尚无规则](media/howto-create-telemetry-rules/rules_landing_page1.png)

1. 上**规则**选项卡上，选择 **+ 新规则**若要查看可以创建的规则的类型。

1. 选择**遥测**创建规则以监视设备遥测数据。

    ![规则类型](media/howto-create-telemetry-rules/rule_types1.png)

1. 输入一个有助于在此设备模板中识别该规则的名称。

1. 若要对针对此模板创建的所有设备立即启用规则，请切换到“为此模板的所有设备启用规则”。

   ![规则详细信息](media/howto-create-telemetry-rules/rule_detail1.png)

    此规则自动应用到该设备模板下的所有设备。

### <a name="configure-the-rule-conditions"></a>配置规则条件

条件用于定义此规则监视的条件。

1. 选择**+** 旁边**条件**以添加新的条件。

1. 从“度量”下拉列表中选择要监视的遥测数据。

1. 接下来，选择“聚合”、“运算符”并提供“阈值”。
   - 聚合是可选的。 如无聚合，此规则将对每个满足此条件的遥测数据点触发。 例如，如果几乎可以立即将规则配置为触发器，当温度高于 80 则触发规则时设备将报告温度 > 80。
   - 如果选择了 Average、Min、Max、Count 等聚合函数，则用户必须提供需要评估的条件的“聚合时间窗口”。 例如，如果将时段设置为“5 分钟”，并且规则查找超过 80 的平均温度，则当平均温度至少有 5 分钟超过 80 时，该规则将会触发。 规则评估频率与“聚合时间段”相同，也就是说，此示例中规则每隔 5 分钟评估一次。

     ![条件](media/howto-create-telemetry-rules/aggregate_condition_filled_out1.png)

     >[!NOTE]
     >可在“条件”下面添加多个遥测度量。 如果指定多个条件，必须满足所有条件才能触发该规则。 每个条件通过“AND”子句隐式联接。 使用聚合时，必须聚合每个度量。

### <a name="configure-actions"></a>配置操作

本部分演示如何设置在规则触发时要执行的操作。 此规则中指定的所有条件评估结果均为 true 时会调用操作。

1. 选择“操作”旁边的“+”。 在此处可以看到可用操作的列表。  

    ![添加操作](media/howto-create-telemetry-rules/add_action1.png)

1. 选择“电子邮件”操作，在“收件人”字段中输入有效的电子邮件地址，并提供一个说明，用于在触发规则时显示在电子邮件的正文中。

    > [!NOTE]
    > 电子邮件只发送给那些已添加到应用程序中并已至少登录一次的用户。 详细了解 Azure IoT Central 中的[用户管理](howto-administer.md)。

   ![配置操作](media/howto-create-telemetry-rules/configure_action1.png)

1. 若要保存规则，请选择“保存”。 此规则在数分钟内即可生效，然后开始监视发送到应用程序的遥测数据。 满足规则中指定的条件时，规则会触发配置的电子邮件操作。

可将其他操作添加到规则，例如 Microsoft Flow 和 Webhook。 每个规则最多可添加 5 个操作。

- [Microsoft Flow 操作](howto-add-microsoft-flow.md)，在触发规则时启动 Microsoft Flow 中的工作流 
- [Webhook 操作](howto-create-webhooks.md)，在触发规则时通知其他服务

## <a name="parameterize-the-rule"></a>将规则参数化

规则可以从用作参数的**设备属性**派生某些值。 如果不同设备的遥测阈值不同，则使用参数会很有帮助。 创建规则时，请选择一个指定阈值（例如“最大理想阈值”）的设备属性，而不要提供绝对值（例如 80 度）。 当规则执行时，会将设备遥测数据与设备属性中设置的值相匹配。

使用参数能够有效地减少要为每个设备模板管理的规则数目。

也可以使用“设备属性”作为参数来配置操作。 如果将电子邮件地址存储为属性，则可在定义“收件人”地址时使用它。

## <a name="delete-a-rule"></a>删除规则

如果不再需要某项规则，可打开该规则并选择“删除”，以将其删除。 删除规则会将其从设备模板和所有关联的设备中删除。

## <a name="enable-or-disable-a-rule-for-a-device-template"></a>启用或禁用设备模板的规则

导航到设备，然后选择要启用或禁用的规则。 在规则中切换“为此模板的所有设备启用规则”按钮可为与设备模板关联的所有设备启用或禁用该规则。

## <a name="enable-or-disable-a-rule-for-a-device"></a>启用或禁用设备的规则

导航到设备，然后选择要启用或禁用的规则。 切换“启用此设备的规则”按钮可启用或禁用该设备的规则。

## <a name="next-steps"></a>后续步骤

了解如何在 Azure IoT Central 应用程序中创建规则后，可了解如下后续步骤：

- [在规则中添加 Microsoft Flow 操作](howto-add-microsoft-flow.md)
- [在规则中添加 Webhook 操作](howto-create-webhooks.md)
- [若要从一个或多个规则运行的多个操作进行分组](howto-use-action-groups.md)
- [如何管理设备](howto-manage-devices.md)
