---
title: 自助密码重置策略 - Azure Active Directory
description: Azure AD 自助密码重置策略选项
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: article
ms.date: 01/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 5952d0b1568e2554f53797515cde5cb23ad95a9b
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2018
---
# <a name="password-policies-and-restrictions-in-azure-active-directory"></a>Azure Active Directory 中的密码策略和限制

本文介绍与 Azure Active Directory (Azure AD) 租户中存储的用户帐户关联的密码策略和复杂性要求。

## <a name="administrator-password-policy-differences"></a>管理员密码策略差异

Microsoft 为任意 Azure 管理员角色强制实施默认强*双门*密码重置策略。 

使用双门策略，管理员将无法使用安全问题。

 双门策略需要两条身份验证数据，如电子邮件地址*和*电话号码。 双门策略在以下情况下适用：

* 以下所有 Azure 管理员角色将受到影响：
  * 支持管理员
  * 服务支持管理员
  * 计费管理员
  * 合作伙伴一线支持人员
  * 合作伙伴二线支持人员
  * Exchange 服务管理员
  * Lync 服务管理员
  * 用户帐户管理员
  * 目录写入者
  * 全局管理员或公司管理员
  * SharePoint 服务管理员
  * 法规管理员
  * 应用程序管理员
  * 安全管理员
  * 特权角色管理员
  * Microsoft Intune 服务管理员
  * 应用程序代理服务器管理员
  * CRM 服务管理员
  * Power BI 服务管理员
  
* 如果在试用订阅中已过 30 天

  或

* 虚域存在（例如 contoso.com）

  或

* Azure AD Connect 正在从本地目录同步标识

### <a name="exceptions"></a>例外
单门策略需要一条身份验证数据，如电子邮件地址*或*电话号码。 单门策略在以下情况下适用：

* 它在试用订阅的前 30 天中

  或

* 虚域不存在 (*.onmicrosoft.com) 

  and 

  Azure AD Connect 未同步标识


## <a name="userprincipalname-policies-that-apply-to-all-user-accounts"></a>适用于所有用户帐户的 UserPrincipalName 策略

需登录到 Azure AD 的每个用户帐户都必须有唯一的与其帐户关联的用户主体名称 (UPN) 属性值。 下表概括了既适用于同步到云的本地 Active Directory 用户帐户又适用于仅限云的用户帐户的策略：

| 属性 | UserPrincipalName 要求 |
| --- | --- |
| 允许的字符 |<ul> <li>A – Z</li> <li>a - z</li><li>0 – 9</li> <li> . - \_ ! \# ^ \~</li></ul> |
| 不允许的字符 |<ul> <li>任何不分隔用户名和域的“\@\"”字符。</li> <li>不能包含紧靠在“\@\"”符号前面的点字符“.”</li></ul> |
| 长度约束 |<ul> <li>总长度不能超过 113 个字符</li><li>在“\@\"”符号前最多能有 64 个字符</li><li>在“\@\"”符号后最多能有 48 个字符</li></ul> |

## <a name="password-policies-that-only-apply-to-cloud-user-accounts"></a>仅适用于云用户帐户的密码策略

下表介绍了可应用于在 Azure AD 中创建和管理的用户帐户的可用密码策略设置：

| 属性 | 要求 |
| --- | --- |
| 允许的字符 |<ul><li>A – Z</li><li>a - z</li><li>0 – 9</li> <li>@ # $ % ^ & * - _ ! + = [ ] { } &#124; \ : ‘ , . ? / ` ~ “ ( ) ;</li></ul> |
| 不允许的字符 |<ul><li>Unicode 字符。</li><li>空格。</li><li> 仅限强密码</li></ul> |
| 密码限制 |<ul><li>至少 8 个字符，最多包含 16 个字符。</li><li>仅限强密码：需满足以下 4 条中的 3 条：<ul><li>小写字符。</li><li>大写字符。</li><li>数字 (0-9)。</li><li>符号（请参阅前面的密码限制）。</li></ul></li></ul> |
| 密码过期期限 |<ul><li>默认值：**90** 天。</li><li>可通过 Windows PowerShell 的 Azure Active Directory 模块中的 `Set-MsolPasswordPolicy` cmdlet 来配置该值。</li></ul> |
| 密码过期通知 |<ul><li>默认值：**14** 天（密码到期前）。</li><li>可使用 `Set-MsolPasswordPolicy` cmdlet 配置该值。</li></ul> |
| 密码到期 |<ul><li>默认值：**false** 天（指示已启用密码到期）。</li><li>可使用 `Set-MsolUser` cmdlet 配置单个用户帐户的值。</li></ul> |
| 密码更改历史记录 |用户更改密码时，上一个密码*不能*再次使用。 |
| 密码重置历史记录 | 用户重置忘记的密码时，上一个密码*可以*再次使用。 |
| 帐户锁定 |使用错误密码 10 次登录尝试失败之后，用户会被锁定一分钟。 后续的错误登录尝试会增加用户被锁定的时间。 |

## <a name="set-password-expiration-policies-in-azure-ad"></a>在 Azure AD 中设置密码过期策略

Microsoft 云服务的全局管理员可使用用于 Windows PowerShell 的 Microsoft Azure AD 模块将用户密码设置为不过期。 还可以使用 Windows PowerShell cmdlet 删除永不过期配置，或者查看已将哪些用户密码设置为永不过期。 

本指南适用于其他提供程序（如 Intune 和 Office 365），这些提供程序也依赖于 Azure AD 提供标识和目录服务。 密码过期是策略中唯一可更改的部分。

> [!NOTE]
> 只能将未通过目录同步进行同步的用户帐户的密码配置为永不过期。 有关目录同步的详细信息，请参阅[将 AD 与 Azure AD 连接](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。
>
>

## <a name="set-or-check-the-password-policies-by-using-powershell"></a>使用 PowerShell 设置或检查密码策略

若要开始，需要[下载并安装 Azure AD PowerShell 模块](https://docs.microsoft.com/powershell/module/Azuread/?view=azureadps-2.0)。 安装后，可以按照以下步骤配置每个字段。

### <a name="how-to-check-the-expiration-policy-for-a-password"></a>如何检查密码过期策略
1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   * 若要查看单个用户的密码是否已设置为永不过期，请使用要查看的用户的 UPN（例如 *aprilr@contoso.onmicrosoft.com*）或用户 ID 运行以下 cmdlet：`Get-MSOLUser -UserPrincipalName <user ID> | Select PasswordNeverExpires`
   * 若要查看所有用户的“密码永不过期”设置，请运行以下 cmdlet：`Get-MSOLUser | Select UserPrincipalName, PasswordNeverExpires`

### <a name="set-a-password-to-expire"></a>设置密码过期

1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   * 若要设置某一用户的密码使其过期，请使用该用户的 UPN 或用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $false`
   * 要将组织中所有用户的密码设置为过期，请使用以下 cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $false`

### <a name="set-a-password-to-never-expire"></a>将密码设置为永不过期

1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   * 若要将某一用户的密码设置为永不过期，请使用该用户的 UPN 或用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $true`
   * 要将组织中所有用户的密码设置为永不过期，请运行以下 cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $true`

   > [!WARNING]
   > 设置为 `-PasswordNeverExpires $true` 的密码仍会基于 `pwdLastSet` 属性过时。 如果将用户密码设置为永不过期，则 90 多天过后密码过期。 基于 `pwdLastSet` 属性，如果将过期更改为 `-PasswordNeverExpires $false`，则所有 `pwdLastSet` 早于 90 天的密码将需要用户在下一次登录时更改它们。 此更改可能会影响很多用户。 

## <a name="next-steps"></a>后续步骤

以下文章提供了有关通过 Azure AD 进行密码重置的更多信息：

* [如何成功推出 SSPR？](howto-sspr-deployment.md)
* [重置或更改密码](../active-directory-passwords-update-your-own-password.md)。
* [注册自助服务密码重置](../active-directory-passwords-reset-register.md)。
* [是否有许可问题？](concept-sspr-licensing.md)
* [SSPR 使用哪些数据？应为用户填充哪些数据？](howto-sspr-authenticationdata.md)
* [哪些身份验证方法可供用户使用？](concept-sspr-howitworks.md#authentication-methods)
* [什么是密码写回？我为什么关心它？](howto-sspr-writeback.md)
* [如何报告 SSPR 中的活动？](howto-sspr-reporting.md)
* [SSPR 中的所有选项有哪些？它们有哪些含义？](concept-sspr-howitworks.md)
* [我认为有些功能被破坏。如何对 SSPR 进行故障排除？](active-directory-passwords-troubleshoot.md)
* [我有在别处未涵盖的问题](active-directory-passwords-faq.md)
