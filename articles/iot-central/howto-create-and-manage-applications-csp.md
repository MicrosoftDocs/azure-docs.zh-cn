---
title: 以 CSP 身份创建和管理 Azure IoT Central 应用程序 | Microsoft Docs
description: 作为 CSP，如何代表客户创建 Azure IoT Central 应用程序。
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 04/03/2019
ms.topic: conceptual
manager: philmea
ms.openlocfilehash: 43fe0047022173cc20f58688920fa95b95cc6c22
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473531"
---
# <a name="as-a-csp-create-and-manage-an-azure-iot-central-application-on-behalf-of-your-customer"></a>作为 CSP，代表客户创建和管理 Azure IoT Central 应用程序 

Microsoft 云解决方案提供商 (CSP) 计划是 Microsoft 经销商计划。 其目的是为我们的渠道合作伙伴提供一站式计划，以转售所有 Microsoft Commercial Online Services。 详细了解[云解决方案提供商计划](https://partner.microsoft.com/cloud-solution-provider)。

作为 CSP，可以通过 [Microsoft 合作伙伴中心](https://partnercenter.microsoft.com/partner/home)代表客户创建和管理 Microsoft Azure IoT Central 应用程序。 当 CSP 代表客户创建 Azure IoT Central 应用程序时，就像其他 CSP 托管的 Azure 服务一样，CSP 管理客户的计费。 Azure IoT Central 的费用将显示在 Microsoft 合作伙伴中心的总账单中。

若要开始，请在 Microsof 合作伙伴门户上登录你的帐户，然后选择要为其创建 Azure IoT Central 应用程序的客户。 从左侧导航栏导航到客户的服务管理。

![Microsoft 合作伙伴中心, 客户视图](media/howto-create-application-csp/image1.png)

Azure IoT Central 将列为可用于管理的服务。 选择要创建新的应用程序或管理此客户的现有应用程序的页面上的 Azure IoT 中心链接。

![可用于管理的 Azure IoT Central](media/howto-create-application-csp/image2.png)

登陆 Azure IoT Central 的“应用程序管理器”页。 Azure IoT Central 保留你来自 Microsoft 合作伙伴中心的上下文，并且你来管理该特定客户。 你会在“应用程序管理器”页的标题中看到此确认。 从这里，你可以导航到之前为该客户创建的现有应用程序来管理它或为客户创建新的应用程序。

![创建适用于 CSP 的管理器](media/howto-create-application-csp/image3.png)

若要创建 Azure IoT Central 应用程序，请选择**新的应用程序**磁贴。 这将加载“应用程序创建”页。 必须完成此页上的所有字段，然后选择“创建”。 你将找到有关以下每个字段的详细信息。

![适用于 CSP 的“创建应用程序”页](media/howto-create-application-csp/image4.png)

![适用于 CSP 的“创建应用程序”页](media/howto-create-application-csp/image4-1.png)

## <a name="payment-plan"></a>付款计划

作为 CSP，你只能创建即用即付应用程序。 若要向客户展示 Azure IoT Central，你可以单独创建试用版应用程序。 在 [Azure IoT Central 定价页](https://azure.microsoft.com/pricing/details/iot-central/)上详细了解试用版和即用即付应用程序。

## <a name="application-name"></a>应用程序名称

应用程序的名称显示在“应用程序管理器”页上和每个 Azure IoT Central 应用程序中。 可以为 Azure IoT Central 应用程序选择任意名称。 选择一个在自己和组织中的其他人看来合适的名称。

## <a name="application-url"></a>应用程序 URL

应用程序 URL 是应用程序的链接。 可以在浏览器中保存其书签或将其与他人共享。

输入应用程序的名称时，系统会自动创建应用程序 URL。 可以根据需要为应用程序选择另一个 URL。 每个 Azure IoT Central URL 在 Azure IoT Central 中必须唯一。 如果所选 URL 已被使用，则会看到一条错误消息。

## <a name="directory"></a>Directory

由于 Azure IoT Central 有你用于管理已在 Microsoft 合作伙伴门户中选择的客户的上下文，因此在“目录”字段中只显示该客户的 Azure Active Directory 租户。 

Azure Active Directory 租户包含用户标识、凭据和其他组织信息。 可能会有多个 Azure 订阅与单个 Azure Active Directory 租户相关联。

有关详细信息，请参阅 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/)。

## <a name="azure-subscription"></a>Azure 订阅

有了 Azure 订阅，就可以创建 Azure 服务的实例。 Azure IoT Central 会自动查找你有权访问的客户的所有 Azure 订阅，并将其显示在“创建应用程序”页的下拉列表中。 选择用于创建新 Azure IoT Central 应用程序的 Azure 订阅。

如果没有 Azure 订阅，则可在 Microsoft 合作伙伴中心创建一个。 创建 Azure 订阅以后，请导航回“创建应用程序”页。 新订阅显示在“Azue 订阅”下拉列表中。

若要了解详细信息，请参阅 [Azure 订阅](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing)。

## <a name="region"></a>区域

选择要在其中创建 Azure IoT Central 应用程序的区域。 通常应选择在地理上最靠近设备的区域，以便优化性能。

若要了解详细信息，请参阅 [Azure 区域](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#azure-regions)。

可以在[可用产品(按区域)](https://azure.microsoft.com/regions/services/)页面上查看提供 Azure IoT Central 的区域。

> [!Note]
> 选择一个区域后，就不能在以后将应用程序移到其他区域。

## <a name="application-template"></a>应用程序模板

可以为新的 Azure IoT Central 应用程序选择一个可用的应用程序模板。 应用程序模板可能包含入门所需的预定义项目（例如设备模板和仪表板）。

| 应用程序模板 | 描述 |
| -------------------- | ----------- |
| 自定义应用程序   | 创建一个空的应用程序，以便在其中填充你自己的设备模板和设备。 |
| 示例 Contoso       | 创建一个应用程序，其中包含用于简单的已连接设备的设备模板。 通过此模板来完成 Azure IoT Central 的入门。 |
| 示例 Devkit       | 创建一个应用程序，其中的设备模板可以用来连接 MXChip 或 Raspberry Pi 设备。 如果你是在这其中的某个设备上进行代码试验的设备开发人员，请使用此模板。 |

## <a name="next-steps"></a>后续步骤

了解如何以 CSP 身份创建 Azure IoT Central 应用程序后，建议接下来执行以下步骤：

> [!div class="nextstepaction"]
> [管理应用程序](howto-administer.md)