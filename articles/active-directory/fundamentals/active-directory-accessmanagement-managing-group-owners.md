---
title: 添加或删除组所有者 - Azure Active Directory | Microsoft Docs
description: 有关如何使用 Azure Active Directory 添加或删除组所有者的说明。
services: active-directory
author: eross-msft
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: lizross
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd684e1bd48f877a74280b33b4df65d7baaa0fe7
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507176"
---
# <a name="add-or-remove-group-owners-in-azure-active-directory"></a>在 Azure Active Directory 中添加或删除组所有者
Azure Active Directory (Azure AD) 组由组所有者拥有和管理。 组所有者可以是用户或服务主体，并且能够管理包括成员身份的组。 只有现有的组所有者或组管理的管理员可以分配组 onwers。 组所有者无需成为组成员。

当组没有所有者时，管理组的管理员是仍可以用于管理组。

## <a name="add-an-owner-to-a-group"></a>向组添加所有者
有关将用户作为所有者添加到组的说明下面使用 Azure AD 门户。 若要将服务主体添加为组的所有者，按照说明进行操作，为此，使用[PowerShell](https://docs.microsoft.com/powershell/module/Azuread/Add-AzureADGroupOwner?view=azureadps-2.0)。

### <a name="to-add-a-group-owner"></a>添加组所有者
1. 使用目录的全局管理员帐户登录到 [Azure 门户](https://portal.azure.com)。

2. 依次选择“Azure Active Directory”、“组”以及要添加所有者的组（例如此示例为“MDM 策略 - 西部”）。

3. 在“MDM 策略 - 西部概述”页面上，选择“所有者”。

    ![“MDM 策略 - 西部概述”页面，其中突出显示了“所有者”选项](media/active-directory-accessmanagement-managing-group-owners/add-owners-option-overview-blade.png)

4. 在“MDM 策略 - 西部 - 所有者”页面上，选择“添加所有者”，然后搜索并选择将成为新的组所有者的用户，然后选中“选择”。

    ![“MDM 策略 - 西部 - 所有者”页面，其中突出显示了“添加所有者”选项](media/active-directory-accessmanagement-managing-group-owners/add-owners-owners-blade.png)

    选择新的所有者后，你可以刷新“所有者”页面并查看添加到所有者列表中的名字。

## <a name="remove-an-owner-from-a-group"></a>删除组所有者
使用 Azure AD 从组中删除所有者。

### <a name="to-remove-an-owner"></a>删除所有者
1. 使用目录的全局管理员帐户登录到 [Azure 门户](https://portal.azure.com)。

2. 依次选择“Azure Active Directory”、“组”以及要删除所有者的组（对于此示例为“MDM 策略 - 西部”）。

3. 在“MDM 策略 - 西部概述”页面上，选择“所有者”。

    ![“MDM 策略 - 西部概述”页面，其中突出显示了“所有者”选项](media/active-directory-accessmanagement-managing-group-owners/remove-owners-option-overview-blade.png)

4. 在“MDM 策略 - 西部 - 所有者”页面上，选择要删除的身份为组所有者的用户，从用户的信息页面中选择“删除”，然后选择“是”以确认你的决定。

    ![用户的信息页面，其中突出显示了“删除”选项](media/active-directory-accessmanagement-managing-group-owners/remove-owner-info-blade.png)

    删除所有者后，可以返回到“所有者”页面，看到该名称已从所有者列表中删除。

## <a name="next-steps"></a>后续步骤
- [使用 Azure Active Directory 组管理对资源的访问](active-directory-manage-groups.md)

- [用于配置组设置的 Azure Active Directory cmdlet](../users-groups-roles/groups-settings-cmdlets.md)

- [使用组分配对集成式 SaaS 应用的访问权限](../users-groups-roles/groups-saasapps.md)

- [将本地标识与 Azure Active Directory 集成](../hybrid/whatis-hybrid-identity.md)

- [用于配置组设置的 Azure Active Directory cmdlet](../users-groups-roles/groups-settings-v2-cmdlets.md)
