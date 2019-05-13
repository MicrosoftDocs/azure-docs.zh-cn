---
title: 本地化字符串 ID - Azure Active Directory B2C | Microsoft Docs
description: 在 Azure Active Directory B2C 中使用自定义策略的 api.signuporsignin ID 指定内容定义的 ID。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: c6cabcc89e66e9fbf153f2931bfe8b9b70c5f426
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64682702"
---
# <a name="localization-string-ids"></a>本地化字符串 ID

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Localization 元素使你能够在用户旅程的策略中支持多个区域设置或语言。 本文提供了可在策略中使用的本地化 ID 列表。 若要熟悉 UI 本地化，请参阅[本地化](localization.md)。

## <a name="sign-up-or-sign-in-page-elements"></a>注册或登录页面元素

以下 ID 用于包含 `api.signuporsignin` ID 的内容定义。

| ID | 默认值 |
| -- | ------------- |
| **local_intro_email** | 使用现有帐户登录 |
| **logonIdentifier_email** | 电子邮件地址 |
| **requiredField_email** | 请输入电子邮件地址 |
| **invalid_email** | 请输入有效的电子邮件地址 |
| **email_pattern** | ^[a-zA-Z0-9.!#$%&’'*+/=?^_\`{\|}~-]+@[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)*$ |
| **local_intro_username** | 使用用户名登录 |
| **logonIdentifier_username** | 用户名 |
| **requiredField_username** | 请输入用户名 |
| **password** | 密码 |
| **requiredField_password** | 请输入密码 |
| **invalid_password** | 你输入的密码格式不正确。 |
| **forgotpassword_link** | 忘记密码了? |
| **createaccount_intro** | 没有帐户？ |
| **createaccount_link** | 立即注册 |
| **divider_title** | 或 |
| **cancel_message** | 用户忘记了密码 |
| **button_signin** | 登录 |
| **social_intro** | 使用社交帐户登录 |
  **remember_me** |使我保持登录状态|
| **unknown_error** | 登录时遇到问题。 请稍后重试。 |

下面的示例演示如何在注册或登录页中使用某些用户界面元素：

![注册或登录页面 UX 元素](./media/localization-string-ids/localization-susi.png)

标识提供者的 ID 在用户旅程 ClaimsExchange 元素中配置。 若要本地化标识提供者的标题，请将 ElementType 设置为 `ClaimsProvider`，同时将 StringId 设置为 `ClaimsExchange` 的 ID。

```XML
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="MicrosoftExchange" TechnicalProfileReferenceId="MSA-OIDC" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccount" />
  </ClaimsExchanges>
</OrchestrationStep>
```

下面的示例将 Facebook 标识提供者本地化为阿拉伯语：

```XML
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">فيس بوك</LocalizedString>
```

## <a name="sign-up-or-sign-in-error-messages"></a>注册或登录错误消息

| ID | 默认值 |
| -- | ------------- |
| **UserMessageIfInvalidPassword** | 密码不正确。 |
| **UserMessageIfClaimsPrincipalDoesNotExist** | 似乎找不到你的帐户。 |
| **UserMessageIfOldPasswordUsed** | 你可能使用的是旧密码。 |  
| **DefaultMessage** | 无效的用户名或密码。 |  
| **UserMessageIfUserAccountDisabled** | 帐户已锁定。 请联系支持人员解除锁定，然后重试。 |  
| **UserMessageIfUserAccountLocked** | 帐户暂时锁定以防止未经授权的使用。 请稍后重试。 |  
| **AADRequestsThrottled** | 目前请求过多。 请等待一段时间，然后重试。 |  

## <a name="sign-up-and-self-asserted-pages-user-interface-elements"></a>注册和自断言页用户界面元素

以下是包含 `api.localaccountsignup` ID 的内容定义或任何以 `api.selfasserted` 开头的内容定义的 ID，如 `api.selfasserted.profileupdate` 和 `api.localaccountpasswordreset`。

| ID | 默认值 |
| -- | ------------- |
| **ver_sent** | 验证码已发送到: |
| **ver_but_default** | 默认 |
| **cancel_message** | 用户已取消输入自断言信息 |
| **preloader_alt** | 请稍候 |
| **ver_but_send** | 发送验证代码 |
| **alert_yes** | 是 |
| **error_fieldIncorrect** | 一个或多个字段填写不正确。 请检查条目并重试。 |
| **year** | 年龄 |
| **verifying_blurb** | 正在处理你的信息，请稍候。 |
| **button_cancel** | 取消 |
| **ver_fail_no_retry** | 你进行不正确尝试的次数过多。 请稍后重试。 |
| **month** | 月份 |
| **ver_success_msg** | 已验证电子邮件地址。 现在可以继续。 |
| **months** | 一月、二月、三月、四月、五月、六月、七月、八月、九月、十月、十一月、十二月 |
| **ver_fail_server** | 我们无法验证你的电子邮件地址。 请输入有效的电子邮件地址，然后重试。 |
| **error_requiredFieldMissing** | 缺少必填字段。 请填写所有必填字段，然后重试。 |
| **initial_intro** | 请提供以下详细信息。 |
| **ver_but_resend** | 发送新验证码 |
| **button_continue** | 创建 |
| **error_passwordEntryMismatch** | 密码输入字段不一致。 请在这两个字段中输入相同密码，然后重试。 |
| **ver_incorrect_format** | 格式错误。 |
| **ver_but_edit** | 更改电子邮件 |
| **ver_but_verify** | 验证验证码 |
| **alert_no** | 否 |
| **ver_info_msg** | 验证码已发送到收件箱。 请将其复制到下面的输入框。 |
| **day** | 日期 |
| **ver_fail_throttled** | 用于验证此电子邮件地址的请求过多。 请稍等片刻，然后重试。 |
| **helplink_text** | 这是什么？ |
| **ver_fail_retry** | 验证码不正确。 请重试。 |
| **alert_title** | 取消输入详细信息 |
| **required_field** | 此信息是必需的。 |
| **alert_message** | 确实要取消输入详细信息吗? |
| **ver_intro_msg** | 验证是必需的。 请单击“发送”按钮。 |
| **ver_input** | 验证码 |

## <a name="sign-up-and-self-asserted-pages-error-messages"></a>注册和自断言页错误消息

| ID | 默认值 |
| -- | ------------- |
| **UserMessageIfClaimsPrincipalAlreadyExists** | 已存在具有指定 ID 的用户。 请选择其他用户。 |
| **UserMessageIfClaimNotVerified** | 未验证声明: {0} |
| **UserMessageIfIncorrectPattern** | 模式错误: {0} |
| **UserMessageIfMissingRequiredElement** | 缺少必需的元素: {0} |
| **UserMessageIfValidationError** | 验证错误: {0} |
| **UserMessageIfInvalidInput** | {0} 具有无效输入。 |
| **ServiceThrottled** | 目前请求过多。 请等待一段时间，然后重试。 |

下面的示例演示如何在注册页中使用某些用户界面元素：

![注册页 UX 元素](./media/localization-string-ids/localization-sign-up.png)

下面的示例演示在用户单击“发送验证码”按钮后如何在注册页中使用某些用户界面元素：

![注册页电子邮件验证 UX 元素](./media/localization-string-ids/localization-email-verification.png)


## <a name="phone-factor-authentication-page-user-interface-elements"></a>电话身份验证页用户界面元素

以下是包含 `api.phonefactor` ID 的内容定义的 ID。 

| ID | 默认值 |
| -- | ------------- |
| **button_verify** | 致电我 |
| **country_code_label** | 国家/地区代码 |
| **cancel_message** | 用户已取消多重身份验证 |
| **text_button_send_second_code** | 发送新验证码 |
| **code_pattern** | \\d{6} |
| **intro_mixed** | 我们记录了以下号码。 我们能够发送短信验证码或致电来验证你的身份。 |
| **intro_mixed_p** | 我们记录了以下号码。 选择一个号码，以便我们能够通过电话或发送短信验证码来验证你的身份。 |
| **button_verify_code** | 验证验证码 |
| **requiredField_code** | 请输入收到的验证码 |
| **invalid_code** | 请输入收到的 6 位验证码 |
| **button_cancel** | 取消 |
| **local_number_input_placeholder_text** | 电话号码 |
| **button_retry** | 重试 |
| **alternative_text** | 我没有电话 |
| **intro_phone_p** | 我们记录了以下号码。 选择一个号码，以便我们能够通过电话来验证你的身份。 |
| **intro_phone** | 我们记录了以下号码。 我们将通过电话来验证你的身份。 |
| **enter_code_text_intro** | 请在下面输入验证码，或  |
| **intro_entry_phone** | 请在下面输入一个号码，以便我们能够通过电话来验证你的身份。 |
| **intro_entry_sms** | 请在下面输入一个号码，以便我们能够发送短信验证码来验证你的身份。 |
| **button_send_code** | 发送验证码 |
| **invalid_number** | 请输入有效电话号码 |
| **intro_sms** | 我们记录了以下号码。 我们将发送短信验证码来验证你的身份。 |
| **intro_entry_mixed** | 请在下面输入一个号码，以便我们能够发送短信验证码或通过电话来验证你的身份。 |
| **number_pattern** | ^\\+(?:[0-9][\\x20-]?){6,14}[0-9]$ |
| **intro_sms_p** |我们记录了以下号码。 选择一个号码，以便我们能够发送短信验证码来验证你的身份。 |
| **requiredField_countryCode** | 请选择国家/地区代码 |
| **requiredField_number** | 请输入电话号码 |
| **country_code_input_placeholder_text** |国家或地区 |
| **number_label** | 电话号码 |
| **error_tryagain** | 你提供的电话号码占线或不可用。 请检查号码并重试。 |
| **error_incorrect_code** | 你输入的验证码与我们的记录不一致。 请重试，或请求获取新验证码。 |
| **countryList** | {\"默认\":\"国家/地区\"，\"AF\":\"阿富汗\"，\"AX\":\"奥兰群岛\"，\"AL\":\"阿尔巴尼亚\"，\"DZ\":\"阿尔及利亚\"，\"AS\":\"美属萨摩亚\"，\"AD\":\"安道尔\"，\"AO\":\"安哥拉\"，\"AI\":\"安圭拉\"，\"AQ\":\"南极洲\"，\"AG\":\"安提瓜和巴布达\"，\"AR\":\"阿根廷\"，\"AM\":\"亚美尼亚\"，\"AW\":\"阿鲁巴\"，\"AU\":\"澳大利亚\"，\"在\":\"奥地利\"，\"AZ\":\"阿塞拜疆\"，\"BS\":\"巴哈马\"，\"BH\":\"巴林\"，\"BD\":\"孟加拉国\"，\"BB\":\"巴巴多斯\"，\"通过\":\"白俄罗斯\"，\"BE\":\"比利时\"，\"BZ\":\"伯利兹\"，\"BJ\":\"贝宁\"，\"BM\":\"百慕大\"，\"BT\":\"不丹\"，\"BO\":\"玻利维亚\"，\"BQ\":\"博内尔岛\"，\"BA\":\"波斯尼亚和黑塞哥维那\"，\"BW\":\"博茨瓦纳<span class="notransla class=""></span class="notransla>离岛\",\"VI\":\"美属维尔京群岛\",\"UG\":\"乌干达\",\"UA\":\"乌克兰\",\"AE\":\"阿拉伯联合酋长国\",\"GB\":\"大不列颠及北爱尔兰联合王国\",\"美国\":\"美国\",\"UY\":\"乌拉圭\",\"UZ\":\"乌兹别克斯坦\",\"VU\":\"瓦努阿图\",\"VA\":\"梵蒂冈城\",\"VE\":\"委内瑞拉\",\"VN\":\"越南\",\"WF\":\"瓦利斯和富图纳\",\"YE\":\"也门\",\"ZM\":\"赞比亚\",\"ZW\":\"津巴布韦\"} |
| **error_448** | 你提供的电话号码打不通。 |
| **error_449** | 用户已超出重试次数上限。 |
| **verification_code_input_placeholder_text** | 验证码 |

下面的示例演示如何在 MFA 注册页中使用某些用户界面元素：

![注册页电子邮件验证 UX 元素](./media/localization-string-ids/localization-mfa1.png)

下面的示例演示如何在 MFA 验证页中使用某些用户界面元素：

![注册页电子邮件验证 UX 元素](./media/localization-string-ids/localization-mfa2.png)







