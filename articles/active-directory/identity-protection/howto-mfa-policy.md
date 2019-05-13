---
title: 如何在 Azure Active Directory Identity Protection 中配置多重身份验证注册策略| Microsoft Docs
description: 了解如何配置 Azure AD Identity Protection 多重身份验证注册策略。
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: article
ms.date: 05/01/2019
author: MicrosoftGuyJFlo
manager: daveba
ms.author: joflore
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1f4083ddf849842358f7699badca6598e56e4dee
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139370"
---
# <a name="how-to-configure-the-azure-multi-factor-authentication-registration-policy"></a>如何：配置 Azure 多重身份验证注册策略

Azure AD Identity Protection 可帮助你管理通过配置条件性访问策略，以要求无论要登录到什么应用程序的 MFA 注册多重身份验证 (MFA) 注册的推出。 本文介绍策略可以用途以及如何对其进行配置。

## <a name="what-is-the-azure-multi-factor-authentication-registration-policy"></a>Azure 多重身份验证注册策略是什么？

Azure 多重身份验证提供了一种方法来验证对正在使用多个而不仅仅是用户名和密码。 它提供了第二层与用户登录的安全性。为了使用户能够对 MFA 提示做出响应，它们必须先注册为 Azure 多重身份验证。

我们建议需要 Azure 多重身份验证的用户登录，因为它：

- 提供强式身份验证和一系列简单的验证选项
- 在准备你的组织在 Identity Protection 风险事件从保护和恢复关键的作用

有关 MFA 的详细信息，请参阅[什么是 Azure 多重身份验证？](../authentication/howto-mfa-getstarted.md)

## <a name="how-do-i-access-the-registration-policy"></a>如何访问注册策略？

MFA 注册策略位于 [Azure AD Identity Protection 页面](https://portal.azure.com/#blade/Microsoft_AAD_ProtectionCenter/IdentitySecurityDashboardMenuBlade/SignInPolicy)上的“配置”部分中。

![MFA 策略](./media/howto-mfa-policy/1014.png)

## <a name="policy-settings"></a>策略设置

配置 MFA 注册策略时，需要进行以下配置更改：

- 用户和组策略应用于的。 请记住要排除你的组织[紧急访问帐户](../users-groups-roles/directory-emergency-access.md)。

    ![用户和组](./media/howto-mfa-policy/11.png)

- 若要强制实施的所需的控件**需要 Azure MFA 注册**

    ![访问](./media/howto-mfa-policy/12.png)

- 强制实施策略应设置为**上**。

    ![强制实施策略](./media/howto-mfa-policy/14.png)

- **保存**策略

## <a name="user-experience"></a>用户体验

如需相关用户体验的概述，请参阅：

- [多重身份验证注册流程](flows.md#multi-factor-authentication-registration)。  
- [Azure AD Identity Protection 中的登录体验](flows.md)。  

## <a name="next-steps"></a>后续步骤

若要获取 Azure AD Identity Protection 的概述，请参阅 [Azure AD Identity Protection 概述](overview.md)。
