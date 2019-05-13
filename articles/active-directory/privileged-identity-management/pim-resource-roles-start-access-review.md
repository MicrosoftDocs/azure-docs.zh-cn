---
title: 在 PIM 的 Azure Active Directory 中创建 Azure 资源角色的访问评审 |Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中创建 Azure 资源角色的访问评审。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 04/29/2019
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78e4de5bbc56f95c0e903b1dac4e8481373716f3
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143522"
---
# <a name="create-an-access-review-of-azure-resource-roles-in-pim"></a>在 PIM 中创建 Azure 资源角色的访问评审

随时间变化的员工对特权的 Azure 资源角色的访问。 若要减少与过时角色分配相关的风险，应定期评审访问权限。 可以使用 Azure Active Directory (Azure AD) Privileged Identity Management (PIM) 来创建访问评审，以便 Azure 资源角色的特权。 此外可以配置会自动执行的定期访问评审。

本文介绍如何创建特权的 Azure 资源角色的一个或多个访问评审。

## <a name="prerequisites"></a>必备组件

- [特权角色管理员](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator)

## <a name="open-access-reviews"></a>打开访问评审

1. 登录到[Azure 门户](https://portal.azure.com/)与用户特权角色管理员角色的成员。

1. 打开“Azure AD Privileged Identity Management”。

1. 在左侧菜单中，单击**Azure 资源**。

1. 单击要管理的资源，如订阅或管理组。

1. 管理下，单击**访问评审**。

    ![Azure 资源的访问评审](./media/pim-resource-roles-start-access-review/access-reviews.png)


[!INCLUDE [Privileged Identity Management access reviews](../../../includes/active-directory-privileged-identity-management-access-reviews.md)]


## <a name="start-the-access-review"></a>启动访问评审

指定访问评审的设置后，单击“启动”。 访问评审会在其状态指示符列表中显示。

![访问评审列表](./media/pim-resource-roles-start-access-review/access-reviews-list.png)

默认情况下，在评审开始后不久，Azure AD 会向评审者发送一封电子邮件。 如果选择不让 Azure AD 发送电子邮件，请务必通知评审者有一个访问评审任务等待他们完成。 说明如何显示[查看对 Azure 资源角色的访问](pim-resource-roles-perform-access-review.md)。

## <a name="manage-the-access-review"></a>管理访问审阅

你可以跟踪进度，当评审者在完成其审阅**概述**访问评审页。 没有访问权限更改之前，目录中[评审完成](pim-resource-roles-complete-access-review.md)。

![访问评审进度](./media/pim-resource-roles-start-access-review/access-review-overview.png)

如果这是一次性的评审，然后访问评审期限结束后，或者管理员停止访问评审后请按照中的步骤[完成 Azure 资源角色的访问评审](pim-resource-roles-complete-access-review.md)查看并应用结果。  

若要管理一系列访问评审会议中，导航到访问评审，并你将找到计划评审中即将推出的匹配项和编辑的结束日期或添加/删除审阅者相应地。

基于所选内容中**完成设置后**，自动应用将审阅的结束日期或手动停止评审后执行。 此评审的状态将更改从**已完成**通过中间状态，如**应用**，最后到状态**应用**。 你应该会看到被拒绝的用户，如果有的话，在几分钟内从角色中删除。

## <a name="next-steps"></a>后续步骤

- [查看对 Azure 资源角色的访问](pim-resource-roles-perform-access-review.md)
- [完成 Azure 资源角色的访问评审](pim-resource-roles-complete-access-review.md)
- [创建 Azure AD 角色的访问评审](pim-how-to-start-security-review.md)
