---
title: 配置安全性以访问和管理 Azure 时序见解预览版 | Microsoft Docs
description: 本文介绍如何将安全性和权限配置为管理访问策略和数据访问策略以保护 Azure 时序见解预览版。
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
ms.openlocfilehash: adb30cc3af50b71cdc03e41d847bd4d824cffea6
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236391"
---
# <a name="grant-data-access-to-an-environment"></a>授予对环境的数据访问权限

本文讨论了两种类型的 Azure 时序见解预览版访问策略。

## <a name="sign-in-to-tsi"></a>登录到 TSI

1. 登录到 [Azure 门户](https://portal.azure.com/)。
1. 查找时序见解环境。 在“搜索”框中输入 `Time Series`。 在搜索结果中选择“时序环境”。
1. 从列表中选择时序见解环境。

## <a name="grant-data-access"></a>授予数据访问权限

遵循以下步骤向用户主体授予数据访问权限。

1. 选择“数据访问策略”，然后选择“+ 添加”。

    [![数据访问一](media/data-access/data-access-one.png)](media/data-access/data-access-one.png#lightbox)

1. 选择“选择用户”。 搜索用户名称或电子邮件地址，查找要添加的用户。 单击“选择”确认选择。

    [![两个访问数据-](media/data-access/data-access-two.png)](media/data-access/data-access-two.png#lightbox)

1. 选择“选择角色”。 为用户选择相应的访问角色：

    * 若要允许用户更改引用数据，以及与环境的其他用户共享保存的查询和透视，请选择“参与者”。

    * 否则请选择“读者”，允许用户查询环境中的数据，以及在环境中保存个人的非共享查询。

   选择“确定”，确认角色选择。

    [![数据访问三](media/data-access/data-access-three.png)](media/data-access/data-access-three.png#lightbox)

1. 在“选择用户角色”页中，选择“确定”。

    [![数据访问四](media/data-access/data-access-four.png)](media/data-access/data-access-four.png#lightbox)

1. 确认“数据访问策略”页列出了用户和每个用户的角色。

    [![五个数据访问](media/data-access/data-access-five.png)](media/data-access/data-access-five.png#lightbox)

## <a name="provide-guest-access-from-another-aad-tenant"></a>提供从另一个 AAD 租户来宾访问

`Guest` 不是管理角色。 它是一个术语，是指从一个租户被邀请到另一个租户的帐户。 在将来宾帐户邀请到租户目录后，它就可以像任何其他帐户一样拥有相同的访问控制。 可以使用“访问控制(IAM)”边栏选项卡授予对时序见解环境的管理访问权限。 也可以通过“数据访问策略”边栏选项卡授予对环境中数据的访问权限。 若要详细了解 Azure Active Directory (Azure AD) 租户来宾访问权限，请阅读[在 Azure 门户中添加 Azure Active Directory B2B 协作用户](https://docs.microsoft.com/azure/active-directory/b2b/add-users-administrator)。

请按照以下步骤将来宾对时序见解环境的访问权限授予另一个租户的 Azure AD 用户。

1. 选择“数据访问策略”，然后选择“+ 邀请”。

    [![Data-access-six](media/data-access/data-access-six.png)](media/data-access/data-access-six.png#lightbox)

1. 输入要邀请的用户的电子邮件地址。 此电子邮件地址必须与 Azure AD 关联。 你可以根据情况在邀请中包括个人消息。

    [![数据访问七](media/data-access/data-access-seven.png)](media/data-access/data-access-seven.png#lightbox)

1. 查找屏幕上显示的确认气泡。

    [![Data-access-eight](media/data-access/data-access-eight.png)](media/data-access/data-access-eight.png#lightbox)

1. 选择“选择用户”。 搜索已邀请的来宾用户的电子邮件地址，找到要添加的用户。 单击“选择”确认选择。

    [![数据访问九](media/data-access/data-access-nine.png)](media/data-access/data-access-nine.png#lightbox)

1. 选择“选择角色”。 为来宾用户选择相应的访问角色：

    * 若要允许用户更改引用数据，以及与环境的其他用户共享保存的查询和透视，请选择“参与者”。

    * 否则请选择“读者”，允许用户查询环境中的数据，以及在环境中保存个人的非共享查询。

   选择“确定”，确认角色选择。

    [![数据访问十](media/data-access/data-access-ten.png)](media/data-access/data-access-ten.png#lightbox)

1. 在“选择用户角色”页中，选择“确定”。

1. 确认“数据访问策略”页列出了来宾用户和每个来宾用户的角色。

    [![数据访问十一](media/data-access/data-access-eleven.png)](media/data-access/data-access-eleven.png#lightbox)

1. 现在，来宾用户必须按步骤访问他们被邀请到的位于 Azure 租户中的环境。 首先，他们需要接受你发送给他们的邀请。 这份邀请通过电子邮件发送到你在步骤 5 中使用的电子邮件地址。 他们选择“开始”即可接受邀请。

    [![Data-access-twelve](media/data-access/data-access-twelve.png)](media/data-access/data-access-twelve.png#lightbox)

1. 接下来，来宾用户接受与管理员组织关联的权限。

    [![十三点访问的数据-](media/data-access/data-access-thirteen.png)](media/data-access/data-access-thirteen.png#lightbox)

1. 当来宾用户登录到你用来邀请他们的电子邮件地址后，他们就可以接受邀请，然后转到 insights.azure.com。 在那里，他们可以选择屏幕右上角的电子邮件地址旁的头像。

    [![十四点访问的数据-](media/data-access/data-access-fourteen.png)](media/data-access/data-access-fourteen.png#lightbox)

1. 接下来，来宾用户从目录下拉菜单中选择你的 Azure 租户。 此租户是你邀请他们访问的租户。

    [![数据访问十五](media/data-access/data-access-fifteen.png)](media/data-access/data-access-fifteen.png#lightbox)

当来宾用户选择你的租户后，他们会看到你为他们提供访问权限的时序见解环境。 他们现在还具有与提供这些与中的角色相关联的所有功能**步骤 5**。

## <a name="next-steps"></a>后续步骤

* 了解[如何将 Azure 事件中心事件源添加到](./time-series-insights-how-to-add-an-event-source-eventhub.md)时序见解环境。

* 发送[事件到事件源](./time-series-insights-send-events.md)。

* [在时序见解预览版资源管理器中查看环境](./time-series-insights-update-explorer.md)。
