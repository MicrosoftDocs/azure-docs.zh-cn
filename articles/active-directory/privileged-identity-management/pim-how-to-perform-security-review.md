---
title: 查看对 PIM 的 Azure Active Directory 中的 Azure AD 角色的访问 |Microsoft Docs
description: 了解如何查看在 Azure AD Privileged Identity Management (PIM) 中的 Azure AD 角色的访问。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 06/21/2018
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3850af026ea13e1920aea65e18358ebb04ef0d25
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65140967"
---
# <a name="review-access-to-azure-ad-roles-in-pim"></a>评审在 PIM 中的 Azure AD 角色的访问权限

Azure Active Directory (AD) Privileged Identity Management (PIM) 简化了企业管理对 Azure AD 中的资源和 Office 365 或 Microsoft Intune 等其他 Microsoft online services 特许访问权限。  

如果已被分配到某个管理角色，组织中的特权角色管理员可能要求定期确认仍然需要使用该角色来完成工作。 可以通过你所收到的电子邮件中的链接，或者直接转到 [Azure 门户](https://portal.azure.com)来执行此操作。 请遵循本文中的步骤对分配的角色执行自我审查。

若为特权角色管理员或全局管理员，并有意了解访问评审，请参阅[如何开始访问评审](pim-how-to-start-security-review.md)，了解详细信息。

## <a name="add-the-privileged-identity-management-application"></a>添加 Privileged Identity Management 应用程序
可以使用 [Azure 门户](https://portal.azure.com/)中的 Azure AD Privileged Identity Management (PIM) 应用程序执行审查。  如果门户中没有 Azure AD Privileged Identity Management 应用程序，请遵循以下步骤开始操作。

1. 登录到 [Azure 门户](https://portal.azure.com/)。
2. 选择 Azure 门户右上角的用户名，并选择要操作的目录。
3. 选择“所有服务”，并使用“筛选器”文本框搜索“Azure AD Privileged Identity Management”。
4. 选中“固定到仪表板”，并单击“创建”。 Privileged Identity Management 应用程序会打开。

## <a name="approve-or-deny-access"></a>批准或拒绝访问权限
批准或拒绝访问权限时，只是告诉审查人你是否仍要使用此角色。 如果想要继续充当此角色，请选择“批准”；如果不再需要此访问权限，请选择“拒绝”。 状态不会立即更改，只会在审查人应用结果之后才更改。
请遵循以下步骤来查找并完成访问权限审查：

1. 在 PIM 应用程序中，选择“审查特权访问权限”。 如果有任何挂起的访问权限审查，它们会显示在 Azure AD 的“访问权限审查”边栏选项卡中。
2. 选择想要完成的审查。
3. 除非该审查是由你创建的，否则你会显示为该审查中的唯一用户。 选中姓名旁边的复选标记。
4. 选择“批准”或“拒绝”。 可能需要在“提供原因”文本框中提供做出该决定的原因。  
5. 关闭“审查 Azure AD 角色”边栏选项卡。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>后续步骤

- [在 PIM 中对 Azure 资源角色执行访问评审](pim-resource-roles-perform-access-review.md)
