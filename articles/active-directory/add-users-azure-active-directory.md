---
title: 在 Azure Active Directory 中添加或删除用户 | Microsoft Docs
description: 介绍如何在 Azure Active Directory 中添加新用户或删除现有用户
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 01/08/2018
ms.author: curtand
ms.reviewer: jeffsta
ms.custom: it-pro
ms.openlocfilehash: 3e8b7061e57c1baf222e5f565a5b7efc6b997507
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2018
---
# <a name="quickstart-add-new-users-to-azure-active-directory"></a>快速入门：将新用户添加到 Azure Active Directory
本文介绍如何使用 Azure 门户或通过同步本地 Windows Server AD 用户帐户数据，在组织的 Azure Active Directory (Azure AD) 租户中删除或添加组织中的用户。 

## <a name="add-cloud-based-users"></a>添加基于云的用户
1. 使用属于目录全局管理员的帐户登录到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 选择“Azure Active Directory”，然后选择“用户和组”。
3. 在“用户和组”中，选择“所有用户”，然后选择“新建用户”。
   ![选择“添加”命令](./media/add-users-azure-active-directory/add-user.png)
4. 输入用户的详细信息，如**名称**和**用户名**。 用户名的域名部分必须是初始默认域名“[domain name].onmicrosoft.com”，或已验证的非联合[自定义域名](add-custom-domain.md)（例如“contoso.com”）。
5. 复制或以其他方式记下生成的用户密码，以便在此过程完成后可以提供给用户。
6. （可选）可以打开“个人资料”、“组”或“目录角色”并在其中填写用户信息。 有关用户和管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色](active-directory-assign-admin-roles-azure-portal.md)。
7. 在“用户”上，选择“创建”。
8. 以安全方式将生成的密码分发给新用户，以便用户可以登录。

> [!TIP]
> 此外，你还可以从本地 Windows Server AD 同步用户帐户数据。 Microsoft 的标识解决方案跨越本地和基于云的功能，创建单一用户标识对所有资源进行身份验证和授权，而不考虑其位置。 我们称此为混合标识。 [Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 可用于集成本地目录与 Azure Active Directory 实现混合标识情景。 这样，便可以为集成到 Azure AD 的 Office 365、Azure 和 SaaS 应用程序的用户提供一个通用标识。 

## <a name="delete-users-from-azure-ad"></a>从 Azure AD 中删除用户
1. 使用属于目录全局管理员的帐户登录到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 选择“用户和组”。
3. 在“用户和组”边栏选项卡中，从列表中选择要删除的用户。 
4. 在所选用户的边栏选项卡中，选择“概述”，并在命令栏中选择“删除”。
   ![选择“添加”命令](./media/add-users-azure-active-directory/delete-user.png)


### <a name="learn-more"></a>了解更多信息 
* [添加另一个目录中的来宾用户](active-directory-b2b-what-is-azure-ad-b2b.md) 
* [在 Azure AD 中为用户分配角色](active-directory-users-assign-role-azure-portal.md)
* [管理用户个人资料](active-directory-users-profile-azure-portal.md)
* [还原已删除的用户](active-directory-users-restore.md)



## <a name="next-steps"></a>后续步骤
在本快速入门中，你已了解如何将新用户添加到 Azure AD Premium。 

可使用以下链接在 Azure 门户中的 Azure AD 中创建新用户。

> [!div class="nextstepaction"]
> [将用户添加到 Azure AD](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/All users) 
