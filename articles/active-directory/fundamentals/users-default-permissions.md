---
title: 默认用户权限 - Azure Active Directory | Microsoft Docs
description: 了解 Azure Active Directory 中各种可用的用户权限。
services: active-directory
author: eross-msft
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 02/16/2019
ms.author: lizross
ms.reviewer: vincesm
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: fb05ee4d6e05cb8b56756a761a519e5903b78bbd
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507095"
---
# <a name="what-are-the-default-user-permissions-in-azure-active-directory"></a>Azure Active Directory 中的默认用户权限是什么？
在 Azure Active Directory (Azure AD) 中，所有用户都被授予一组默认权限。 用户的访问权限由用户的类型、其[角色分配](active-directory-users-assign-role-azure-portal.md)及其对单个对象的所有权构成。 本文将会介绍这些默认权限，并将成员和来宾用户的默认权限进行比较。 只能在 Azure AD 的用户设置中更改默认用户权限。

## <a name="member-and-guest-users"></a>成员和来宾用户
获得的默认权限集取决于用户是租户的本机成员（成员用户），还是用户作为 B2B 协作来宾（来宾用户）从另一个目录转入。 有关添加来宾用户的详细信息，请参阅[什么是 Azure AD B2B 协作？](../b2b/what-is-b2b.md)。
* 成员用户可以注册应用程序、管理自己的个人资料照片和手机号码、更改自己的密码，以及邀请 B2B 来宾。 此外，用户可以读取所有目录信息（少数用户除外）。 
* 来宾用户的目录权限受到限制。 例如，来宾用户只能浏览自己的个人资料信息，而不能浏览租户中的其他信息。 但是，来宾用户可以通过提供用户主体名称或 objectId 来检索有关另一用户的信息。 来宾用户可以读取他们所属组的属性，包括组成员身份，而不考虑“来宾用户权限处于限制状态”设置。 来宾无法查看有关任何其他租户对象的信息。

默认情况下，来宾的默认权限受到限制。 可将来宾添加到管理员角色，从而向他们授予角色中包含的完全读取和写入权限。 此外还有一条限制，即来宾邀请其他来宾的能力。 将“来宾可邀请”设置为“否”会阻止来宾邀请其他来宾。 有关操作方法，请参阅[委托 B2B 协作邀请](../b2b/delegate-invitations.md)。 若要向来宾用户授予成员用户默认拥有的权限，请将“来宾用户权限处于限制状态”设置为“否”。 此设置向来宾用户授予默认的成员用户权限，并允许将来宾添加到管理角色。

## <a name="compare-member-and-guest-default-permissions"></a>比较成员和来宾的默认权限

**区域** | **成员用户权限** | **来宾用户权限**
------------ | --------- | ----------
用户和联系人 | 读取用户和联系人的所有公共属性<br>邀请来宾<br>更改自己的密码<br>管理自己的手机号码<br>管理自己的照片<br>使自己的刷新令牌失效 | 读取自己的属性<br>读取其他用户和联系人的显示名称、电子邮件、登录名、照片、用户主体名称和用户类型属性<br>更改自己的密码
组 | 创建安全组<br>创建 Office 365 组<br>读取组的所有属性<br>读取非隐藏的组成员身份<br>读取加入的组的隐藏 Office 365 组成员身份<br>管理用户拥有的组的属性、所有权和成员身份<br>将来宾添加到拥有的组<br>管理动态成员身份设置<br>删除拥有的组<br>还原拥有的 Office 365 组 | 读取组的所有属性<br>读取非隐藏的组成员身份<br>读取加入的组的隐藏 Office 365 组成员身份<br>管理拥有的组<br>将来宾添加到拥有的组（如果允许）<br>删除拥有的组<br>还原拥有的 Office 365 组<br>读取他们所属组的属性，包括成员身份。
应用程序 | 注册（创建）新应用程序<br>读取已注册的应用程序和企业应用程序的属性<br>管理拥有的应用程序的应用程序属性、分配和凭据<br>创建或删除用户的应用程序密码<br>删除拥有的应用程序<br>还原拥有的应用程序 | 读取已注册的应用程序和企业应用程序的属性<br>管理拥有的应用程序的应用程序属性、分配和凭据<br>删除拥有的应用程序<br>还原拥有的应用程序
设备 | 读取设备的所有属性<br>管理拥有的设备的所有属性<br> | 无权限<br>删除拥有的设备<br>
Directory | 读取所有公司信息<br>读取所有域<br>读取所有合作伙伴协定 | 读取显示名称和已验证的域
角色和范围 | 读取所有管理角色和成员身份<br>读取管理单元的所有属性和成员身份 | 无权限 
订阅 | 读取所有订阅<br>启用服务计划成员 | 无权限
策略 | 读取策略的所有属性<br>管理拥有的策略的所有属性 | 无权限

## <a name="to-restrict-the-default-permissions-for-member-users"></a>限制成员用户的默认权限

可通过以下方式限制成员用户的默认权限。

权限 | 设置说明
---------- | ------------
用户可以注册应用程序 | 将此选项设置为否可阻止用户创建应用程序注册。 功能则可以返回到特定的个人通过将它们添加到应用程序开发人员角色授予。
允许用户将工作或学校帐户与领英相连接 | 将此选项设置为否可阻止用户使用其 LinkedIn 帐户连接其工作或学校帐户。  请参阅[LinkedIn 帐户连接数据共享和同意](https://docs.microsoft.com/azure/active-directory/users-groups-roles/linkedin-user-consent)有关详细信息。
能够创建安全组 | 将此选项设置为“否”可阻止用户创建安全组。 全局管理员和用户管理员仍可创建安全组。 有关操作方法，请参阅[用于配置组设置的 Azure Active Directory cmdlet](../users-groups-roles/groups-settings-cmdlets.md)。
能够创建 Office 365 组 | 将此选项设置为“否”可阻止用户创建 Office 365 组。 将此选项设置为“某些”可让选定的一组用户创建 Office 365 组。 全局管理员和用户管理员仍可创建 Office 365 组。 有关操作方法，请参阅[用于配置组设置的 Azure Active Directory cmdlet](../users-groups-roles/groups-settings-cmdlets.md)。
限制访问 Azure AD 管理门户 | 将此选项设置为“否”可阻止用户访问 Azure Active Directory。
能够读取其他用户 | 此设置仅可在 PowerShell 中使用。 将此设置为 $false 可阻止所有非管理员用户从目录读取用户信息。 这不会阻止读取其他 Microsoft 服务（如 Exchange Online）中的用户信息。 此设置适用于特殊情况，因此不建议将此设置为 $false。

## <a name="object-ownership"></a>对象所有权

### <a name="application-registration-owner-permissions"></a>应用程序注册所有者权限
当某个用户注册某个应用程序时，该用户将自动添加为该应用程序的所有者。 所有者可以管理应用程序的元数据，例如应用请求的名称和权限。 他们还可以管理应用程序的特定于租户的配置，例如 SSO 配置和用户分配。 所有者还可以添加或删除其他所有者。 与全局管理员不同，所有者只能管理他们拥有的应用程序。

<!-- ### Enterprise application owner permissions

When a user adds a new enterprise application, they are automatically added as an owner for the tenant-specific configuration of the application. As an owner, they can manage the tenant-specific configuration of the application, such as the SSO configuration, provisioning, and user assignments. An owner can also add or remove other owners. Unlike Global Administrators, owners can manage only the applications they own. <!--To assign an enterprise application owner, see *Assigning Owners for an Application*.-->

### <a name="group-owner-permissions"></a>组所有者权限

当某个用户创建某个组时，该用户将自动添加为该组的所有者。 所有者可以管理组的属性（例如名称），以及管理组成员身份。 所有者还可以添加或删除其他所有者。 与全局管理员和用户管理员不同，所有者只能管理他们拥有的组。 若要分配组所有者，请参阅[管理组的所有者](active-directory-accessmanagement-managing-group-owners.md)。

## <a name="next-steps"></a>后续步骤

* 若要详细了解如何分配 Azure AD 管理员角色，请参阅[在 Azure Active Directory 中向用户分配管理员角色](active-directory-users-assign-role-azure-portal.md)
* 若要了解有关如何在 Microsoft Azure 中控制资源访问的详细信息，请参阅 [了解 Azure 中的资源访问权限](../../role-based-access-control/rbac-and-directory-admin-roles.md)
* 有关 Azure Active Directory 如何与 Azure 订阅相关联的详细信息，请参阅 [How Azure subscriptions are associated with Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)（Azure 订阅与 Azure Active Directory 的关联方式）
* [管理用户](add-users-azure-active-directory.md)
