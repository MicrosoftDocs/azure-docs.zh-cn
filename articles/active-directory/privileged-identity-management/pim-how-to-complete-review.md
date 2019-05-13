---
title: 完成 Azure AD PIM-Azure Active Directory 中的角色的访问评审 |Microsoft Docs
description: 了解如何完成在 Azure AD Privileged Identity Management (PIM) 中的 Azure AD 角色的访问评审并查看结果
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 06/06/2017
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: decba8f46432891a44a2b231890da4214bd5b1bb
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65140988"
---
# <a name="complete-an-access-review-of-azure-ad-roles-in-pim"></a>完成 Azure AD PIM 中的角色的访问评审
[访问评审开始](pim-how-to-start-security-review.md)后，特权角色管理员可以评审特权访问。 Azure Active Directory (Azure AD) Privileged Identity Management (PIM) 会自动发送一封电子邮件，提示用户审阅其访问权限。 如果用户未收到电子邮件，可以向他们发送[如何执行访问评审](pim-how-to-perform-security-review.md)的相关说明。

访问评审期限结束，或者所有用户已完成其自评审后，请按照本文中的步骤管理评审并查看结果。

## <a name="manage-access-reviews"></a>管理访问评审
1. 转到 [Azure 门户](https://portal.azure.com/)，并在仪表板上选择“Azure AD Privileged Identity Management”应用程序。
2. 选择仪表板的“访问审阅”部分。
3. 选择要管理的访问审阅。

在访问评审的详细信息边栏选项卡上，有大量用于管理该评审的选项。

![PIM 访问审阅按钮 - 屏幕截图](./media/pim-how-to-complete-review/PIM_review_buttons.png)

### <a name="remind"></a>提醒
如果设置了用于用户审阅自身的访问审阅，“提醒”按钮将发送一条通知。 

### <a name="stop"></a>停止
所有访问审阅都有结束日期，但可以使用“停止”按钮提前结束。 如果此时还有未审阅的用户，他们在停止审阅后将无法再得到审阅。 停止后，无法重新开始审阅。

### <a name="apply"></a>应用
结束访问审阅后，“应用”按钮将实现审阅结果，因为结束日期已到或已手动停止它。 如果在审阅中拒绝了用户的访问，在此步骤中将删除其角色分配。  

### <a name="export"></a>导出
如果要手动应用访问评审的结果，可以导出该评审。 “导出”按钮将开始下载 CSV 文件。 可以在 Excel 或可打开 CSV 文件的其他程序中管理结果。

### <a name="delete"></a>删除
如果不想要进一步了解审阅，请将其删除。 “删除”按钮可从 PIM 应用程序中删除审阅。

> [!IMPORTANT]
> 在发生删除前不会收到任何警告，因此请确保要删除该审阅。 

## <a name="next-steps"></a>后续步骤

- [在 PIM 中启动 Azure AD 角色的访问评审](pim-how-to-start-security-review.md)
- [在 PIM 中执行我的 Azure AD 角色的访问评审](pim-how-to-perform-security-review.md)
