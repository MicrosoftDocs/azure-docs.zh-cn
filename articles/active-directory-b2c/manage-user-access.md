---
title: 在 Azure Active Directory B2C 中管理用户访问 | Microsoft Docs
description: 了解如何识别未成年人、 收集的出生和国家/地区数据的日期和使用 Azure AD B2C 应用程序中获取接受的使用条款。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/24/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 88123cc24359daaf1c6fc7e3ceeed8f77f717c9a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228021"
---
# <a name="manage-user-access-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中管理用户访问

本文介绍如何使用 Azure Active Directory (AD) B2C 管理用户对应用程序的访问。 应用程序中的访问管理包括：

- 识别未成年人，以及控制用户对应用程序的访问。
- 要求未成年人在使用应用程序之前经过家长的同意。
- 从用户收集出生和国家/地区的数据。
- 捕获使用条款的同意状态和限制访问。

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

## <a name="control-minor-access"></a>控制未成年人访问

应用程序和组织可以决定阻止未成年人使用并非面向此类受众的应用程序和服务。 或者，应用程序和组织可以决定接受未成年人访问，随后管理家长的同意，并根据企业规则和法规条例的规定，向未成年人提供许可的体验。 

如果已将某个用户识别为未成年人，则你可以将 Azure AD B2C 中的用户流设置为以下三个选项之一：

- **将已签名的 JWT id_token 发回到应用程序**：在目录中注册用户，并将令牌返回到应用程序。 然后，应用程序将通过应用企业规则继续运行。 例如，应用程序可能继续采取家长同意过程。 若要使用此方法，请选择从应用程序接收 **ageGroup** 和 **consentProvidedForMinor** 声明。

- **将未签名的 JSON 令牌发送到应用程序**：Azure AD B2C 告知应用程序，用户是未成年人，并提供该用户的家长同意状态。 然后，应用程序将通过应用企业规则继续运行。 JSON 令牌不会在应用程序中成功完成身份验证。 应用程序必须根据 JSON 令牌中包含的声明处理未经身份验证的用户，这些声明可能包括 **name**、**email**、**ageGroup** 和 **consentProvidedForMinor**。

- **阻止用户**：如果用户是未成年人，并且未获得家长同意，则 Azure AD B2C 可以告知该用户他（她）已被阻止。 不会颁发令牌，访问将被阻止，并且不会在注册旅程期间创建用户帐户。 若要实现此通知，可以提供适当的 HTML/CSS 内容页来告知用户，并显示相应的选项。 应用程序不需要对新的注册采取进一步的措施。

## <a name="get-parental-consent"></a>获得家长同意

根据应用程序的管制，认证为成年人的用户可能需要提供家长同意。 Azure AD B2C 不提供相应的体验来认证个人的年龄，然后允许认证的成年人向未成年人提供家长同意。 此体验必须由应用程序或其他服务提供程序提供。

下面是收集家长同意的用户流示例：

1. [Azure Active Directory 图形 API](/previous-versions/azure/ad/graph/api/api-catalog) 操作将用户识别为未成年人，并将用户数据以未签名 JSON 令牌的形式返回给应用程序。

2. 应用程序处理 JSON 令牌，并向未成年人显示一个屏幕，告知他（她）需要家长同意，并请求家长在线同意。 

3. Azure AD B2C 显示可让用户正常登录的登录旅程，并向应用程序颁发一个令牌，该令牌设置为包含 **legalAgeGroupClassification ="minorWithParentalConsent"**。 应用程序收集家长的电子邮件地址，并验证该家长是否为成年人。 为此，它会使用受信任的源，例如身份证颁发机构、执照验证或信用卡证明。 如果验证成功，则应用程序会提示未成年人使用 Azure AD B2C 用户流登录。 如果同意被拒绝（例如 **legalAgeGroupClassification ="minorWithoutParentalConsent"**），则 Azure AD B2C 会向应用程序返回 JSON 令牌（并非登录名），以重启同意过程。 可以选择性地自定义用户流，让未成年人或成年人重获未成年人帐户的访问权限，方法是向记录的未成年人电子邮件地址或成年人电子邮件地址发送一个注册码。

4. 应用程序提供一个选项让未成年人撤消同意。

5. 未成年人或成人撤消同意时，可以使用 Azure AD 图形 API 将 **consentProvidedForMinor** 更改为 **denied**。 或者，应用程序可以选择删除已撤消其同意的未成年人。 可以选择性地自定义用户流，让经过身份验证的未成年人（或使用未成年人帐户的家长）撤消同意。 Azure AD B2C 将 **consentProvidedForMinor** 记录为 **denied**。

有关 **legalAgeGroupClassification**、**consentProvidedForMinor** 和 **ageGroup** 的详细信息，请参阅[用户资源类型](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/user)。 有关自定义属性的详细信息，请参阅[使用自定义属性来收集有关用户的信息](active-directory-b2c-reference-custom-attr.md)。 使用 Azure AD 图形 API 解决扩展属性时，必须使用长版本的属性，例如 *extension_18b70cf9bb834edd8f38521c2583cd86_dateOfBirth*:2011-01-01T00:00:00Z。

## <a name="gather-date-of-birth-and-countryregion-data"></a>收集的出生和国家/地区的数据的日期

应用程序可能依赖于 Azure AD B2C 将在注册期间收集从所有用户的出生日期 (DOB) 和国家/地区的信息。 如果此信息不存在，在下一次身份验证（登录）旅程期间，应用程序可以请求用户提供此信息。 用户无法继续，而无需提供他们 DOB 和国家/地区的信息。 Azure AD B2C 使用信息来确定是否对单个被认为是根据该国家/地区的法规标准次要。 

自定义的用户流可以收集 DOB 和国家/地区信息和使用 Azure AD B2C 的声明转换来确定**ageGroup**并保存结果 （或直接保留的 DOB 和国家/地区信息） 中目录。

以下步骤演示用于从用户的出生日期计算 **ageGroup** 的逻辑：

1. 尝试按列表中的国家/地区代码查找国家/地区。 如果未找到该国家/地区，则回退到 **Default**。

2. 如果 country 元素中存在 **MinorConsent** 节点：

    a. 计算用户必须在哪个日期出生，才能将其视为成年人。 例如，如果当前日期为 2015 年 3 月 14 日，**MinorConsent** 为 18，则出生日期必须晚于 2000 年 3 月 14。

    b. 将最小出生日期与实际出生日期相比较。 如果最小出生日期在用户的实际出生日期之前，则计算会返回 **Minor** 作为年龄组计算结果。

3. 如果 country 元素中存在 **MinorNoConsentRequired** 节点，请使用 **MinorNoConsentRequired** 中的值重复步骤 2a 和 2b。 如果最小出生日期在用户的实际出生日期之前，则 2b 的输出将返回 **MinorNoConsentRequired**。 

4. 如果任何计算都未返回 true，则计算会返回 **Adult**。

如果应用程序已通过其他方法中可靠地收集 DOB 或国家/地区的数据，应用程序可能会使用图形 API 来更新用户记录的此信息。 例如:

- 如果已知某个用户是成年人，则使用 **Adult** 值更新目录属性 **ageGroup**。
- 如果已知某个用户是未成年人，则使用 **Minor** 值更新目录属性 **ageGroup**，并相应地设置 **consentProvidedForMinor**。

有关收集 DOB 数据的详细信息，请参阅[在 Azure AD B2C 中使用年龄门控](basic-age-gating.md)。

## <a name="capture-terms-of-use-agreement"></a>捕获用户条款的同意状态

开发应用程序时，我们通常会从用户目录捕获用户在其应用程序中接受使用条款的情况，其中规定不允许或只允许未成年人参与应用程序。 但是，有时可以使用 Azure AD B2C 用户流来收集用户对使用条款的接受状态、限制为只有在接受条款的情况下才允许访问，并强制要求接受未来对使用条款所做的更改（基于上次接受日期，以及使用条款最新版本的发布日期）。

**使用条款**还可以包含“同意与第三方共享数据。” 可以根据当地法规和企业规则统一收集用户对这两个条件的接受状态，或者可以允许用户接受其中一个条件，而不接受另一个条件。

以下步骤描述如何管理使用条款：

1. 使用图形 API 和扩展属性记录使用条款的接受状态以及接受日期。 可以使用内置和自定义的用户流实现此目的。 我们建议创建并使用 **extension_termsOfUseConsentDateTime** 和 **extension_termsOfUseConsentVersion** 属性。

2. 创建标有“接受使用条款”的所需复选框，并记录注册期间的结果。 可以使用内置和自定义的用户流实现此目的。

3. Azure AD B2C 会存储使用条款协议和用户的接受状态。 可以使用图形 API，通过读取用于记录响应的扩展属性（例如读取 **termsOfUseTestUpdateDateTime**）来查询任何用户的状态。 可以使用内置和自定义的用户流实现此目的。

4. 将接受日期与使用条款最新版本的发布日期进行比较，要求接受更新的使用条款。 只能使用自定义用户流来比较日期。 使用扩展属性 **extension_termsOfUseConsentDateTime** 并比较 **termsOfUseTextUpdateDateTime** 声明的值。 如果接受日期更早，则通过显示自我断言的屏幕来强制接受新条款。 否则使用策略逻辑阻止访问。

5. 将接受版本号与最近接受的版本号进行比较，要求接受更新的使用条款。 只能使用自定义用户流比较版本号。 使用扩展属性 **extension_termsOfUseConsentDateTime** 并比较 **extension_termsOfUseConsentVersion** 声明的值。 如果接受日期更早，则通过显示自我断言的屏幕来强制接受新条款。 否则使用策略逻辑阻止访问。

在以下情况下，可以捕获使用条款的接受状态：

- 新用户正在注册。 显示使用条款，并存储接受结果。
- 用户正在登录，并已事先接受了最新或现行的使用条款。 不显示使用条款。
- 用户正在登录，且尚未接受最新或现行的使用条款。 显示使用条款，并存储接受结果。
- 用户正在登录，并已接受旧版使用条款，该版本现已更新为最新版本。 显示使用条款，并存储接受结果。

下图显示了建议的用户流：

![接受用户流](./media/manage-user-access/user-flow.png) 

下面是声明中基于 DateTime 的使用条款许可状态示例：

```
<ClaimsTransformations>
  <ClaimsTransformation Id="GetNewUserAgreeToTermsOfUseConsentDateTime" TransformationMethod="GetCurrentDateTime">
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentDateTime" TransformationClaimType="currentDateTime" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="IsTermsOfUseConsentRequired" TransformationMethod="IsTermsOfUseConsentRequired">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="extension_termsOfUseConsentDateTime" TransformationClaimType="termsOfUseConsentDateTime" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="termsOfUseTextUpdateDateTime" DataType="dateTime" Value="2098-01-30T23:03:45" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="result" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations>
```

下面是声明中基于 Version 的使用条款许可状态示例：

```
<ClaimsTransformations>
  <ClaimsTransformation Id="GetEmptyTermsOfUseConsentVersionForNewUser" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value=""/>
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="GetNewUserAgreeToTermsOfUseConsentVersion" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value="V1"/>
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="compareTo" DataType="string" Value="V1" />
      <InputParameter Id="operator" DataType="string" Value="not equal" />
      <InputParameter Id="ignoreCase" DataType="string" Value="true" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations> 
```

## <a name="next-steps"></a>后续步骤

- 若要了解如何删除和导出用户数据，请参阅[管理用户数据](manage-user-data.md)。
