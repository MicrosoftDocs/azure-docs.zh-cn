---
title: 使用 Azure Active Directory 条件性访问配置身份验证会话管理
description: 自定义 Azure AD 身份验证会话配置频率和浏览器会话持续性中包括用户登录。
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0ff9ad850b111cf080447b699d35b4ef8205e006
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190222"
---
# <a name="configure-authentication-session-management-with-conditional-access"></a>使用条件性访问配置身份验证会话管理

在复杂的部署，组织可能需要限制身份验证会话。 某些情况下可能包括：

* 从非托管或共享设备的资源访问
* 从外部网络访问敏感信息
* 高影响的用户
* 关键业务应用程序

条件性访问控制允许你创建面向你的组织内的特定用例，而不会影响所有用户的策略。

有关如何配置策略深入详细信息之前, 看一下默认配置。

## <a name="user-sign-in-frequency"></a>用户登录频率

在登录频率定义前要求用户重新登录尝试访问资源时的时间段。

在频率的用户登录的 Azure Active Directory (Azure AD) 默认配置是 90 天的滚动窗口。 通常要求用户输入凭据似乎是合理的操作，但它可以引起反感： 经过培训，可以输入其凭据，而无需考虑可能无意中的用户将其提供给恶意凭据提示。

这可能听起来很惊人不要求用户重新登录 90 天，实际上与 IT 策略的任何冲突将撤消该会话。 一些示例包括 （但不限于） 密码更改、 转为不兼容的设备或禁用的帐户。 你可以显式也[撤销用户的会话使用 PowerShell](https://docs.microsoft.com/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0)。 Azure AD 的默认配置是"不要求用户提供其凭据，如果未更改其会话的安全状况"。

登录频率设置适用于已实现根据标准 OATH2 或 OIDC 协议的应用。 大多数 Microsoft 本机应用的 Windows、 Mac 和移动符合的设置。

## <a name="persistence-of-browsing-sessions"></a>浏览会话持久性

持久的浏览器会话允许用户保持登录状态后关闭并重新打开其浏览器窗口。

浏览器会话暂留的 Azure AD 默认允许用户选择是否通过显示"保持登录？"来保存会话的个人设备上 身份验证成功后提示。 如果使用的指南一文中的 AD FS 中配置浏览器暂留[AD FS 单一登录设置](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/ad-fs-single-sign-on-settings#enable-psso-for-office-365-users-to-access-sharepoint-online
)，我们将遵守该策略，并保留在 Azure AD 会话。 您还可以配置是否在租户中的用户将看到"保持登录？" 通过更改公司品牌一文中使用的指南在 Azure 门户中的窗格中的相应设置提示[自定义 Azure AD 登录页](../fundamentals/customize-branding.md)。

## <a name="configuring-authentication-session-controls"></a>配置身份验证会话控件

条件性访问是 Azure AD Premium 功能，需要 premium 许可证。 若要了解有关条件访问的详细信息，请参阅[什么是 Azure Active Directory 中的条件访问？](overview.md#license-requirements-for-using-conditional-access)

> [!WARNING]
> 如果使用的[可配置令牌生存期](../develop/active-directory-configurable-token-lifetimes.md)功能目前处于公共预览状态，请注意，我们不支持创建具有相同的用户或应用程序组合两个不同的策略： 一个具有此功能，使用另一个可配置令牌生存期功能。 Microsoft 计划在年 10 月 15 日停用的可配置令牌生存期功能，并将其替换为条件性访问身份验证会话管理功能。  

### <a name="policy-1-sign-in-frequency-control"></a>策略 1：在登录频率控件

1. 创建新策略
1. 选择所需的所有客户的环境，包括目标云应用的条件。

   > [!NOTE]
   > 建议将用于 Exchange Online 和 SharePoint Online 等的最佳用户体验的关键 Microsoft Office 应用相同的身份验证提示频率设置。

1. 转到**访问控制** > **会话**单击**登录频率**
1. 第一个文本框中输入所需的值的天数和时数
1. 选择的值**小时**或**天**从下拉列表中
1. 保存策略

![在频率为登录配置的条件性访问策略](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-sign-in-frequency.png)

在 Azure AD 注册的 Windows 设备登录到该设备被视为一条提示。 例如，如果频率为 24 小时后，Office 应用程序中配置了登录，Azure AD 上的用户注册的 Windows 设备将通过登录到该设备满足频率策略登录并且将在不提示再次打开 Office 应用程序时。

如果已配置为在同一个浏览器会话中运行的另一个 web 应用的不同登录频率，最严格的策略将应用于这两个应用因为在同一个浏览器会话中运行的所有应用都共享一个会话令牌。

### <a name="policy-2-persistent-browser-session"></a>策略 2：持久性浏览器会话

1. 创建新策略
1. 选择所有所需的条件。

   > [!NOTE]
   > 请注意，此控件需要选择"所有云应用"作为条件

1. 转到**访问控制** > **会话**单击**持久的浏览器会话**
1. 从下拉列表中选择一个值
1. 保存策略

![为持久的浏览器配置的条件性访问策略](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-persistent-browser.png)

> [!NOTE]
> Azure AD 条件访问中的永久性浏览器会话配置将覆盖"保持登录？" 公司品牌为同一用户在 Azure 门户中的窗格中，如果已配置这两个策略中的设置。

## <a name="validation"></a>验证

使用假设工具模拟用户的登录名，以目标应用程序和基于策略的配置方式的其他条件。 身份验证会话管理控件将显示在该工具的结果。

![如果工具结果的条件性访问](media/howto-conditional-access-session-lifetime/conditional-access-what-if-tool-result.png)

## <a name="policy-deployment"></a>策略部署

若要确保你的策略按预期工作，建议的最佳做法是在将其推广到生产环境之前对其进行测试。 理想情况下，使用一个测试租户来验证新策略是否按预期方式工作。 有关详细信息，请参阅文章[Azure Active Directory 中条件性访问的最佳做法](best-practices.md)。

## <a name="next-steps"></a>后续步骤

* 如果你想要知道如何配置条件性访问策略，请参阅文章[使用 Azure Active Directory 条件性访问的特定应用的要求使用 MFA](app-based-mfa.md)。
* 如果你已准备好配置你的环境的条件性访问策略，请参阅文章[Azure Active Directory 中条件性访问的最佳做法](best-practices.md)。
