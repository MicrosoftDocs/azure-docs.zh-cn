---
title: 为 Azure AD 中的企业应用程序自定义 SAML 令牌中颁发的声明 | Microsoft Docs
description: 了解如何为 Azure AD 中的企业应用程序自定义 SAML 令牌中颁发的声明。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: f1daad62-ac8a-44cd-ac76-e97455e47803
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/03/2019
ms.author: celested
ms.reviewer: luleon, paulgarn, jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: b137b8cd4e3a2b7a308170904e9b3d09b11137f9
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231340"
---
# <a name="how-to-customize-claims-issued-in-the-saml-token-for-enterprise-applications"></a>如何：为企业应用程序自定义 SAML 令牌中颁发的声明

目前，Azure Active Directory (Azure AD) 支持单一登录 (SSO) 与大多数企业应用程序，包括这两个预先集成的 Azure AD 应用库以及自定义应用程序中的应用程序。 当用户通过 Azure AD 使用 SAML 2.0 协议向应用程序进行身份验证时，Azure AD 会将令牌发送到应用程序（通过 HTTP POST）。 然后，应用程序验证并使用该令牌将用户登录，而不是提示输入用户名和密码。 这些 SAML 令牌包含有关称为用户的信息片段*声明*。

“声明”是标识提供者在为某个用户颁发的令牌中陈述的有关该用户的信息。 在 [SAML 令牌](https://en.wikipedia.org/wiki/SAML_2.0)中，此数据通常包含在 SAML 属性语句中。 用户的唯一 ID（也称为名称标识符）通常显示在 SAML 主题中。

默认情况下，Azure AD 颁发 SAML 令牌中包含应用程序到`NameIdentifier`可唯一地标识用户的 Azure AD 中声明的值为用户的用户名 （也称为用户主体名称）。 SAML 令牌还含有其他声明，其中包含用户的电子邮件地址、名字和姓氏。

若要查看或编辑 SAML 令牌中颁发给应用程序的声明，请在 Azure 门户中打开应用程序。 然后打开**用户属性和声明**部分。

![用户属性和声明部分](./media/active-directory-saml-claims-customization/sso-saml-user-attributes-claims.png)

有两个可能的原因使你可能需要编辑 SAML 令牌中颁发的声明：

* 应用程序需要`NameIdentifier`或 NameID 声明为非 Azure AD 中存储的用户名 （或用户主体名称）。
* 应用程序已编写为需要一组不同的声明 URI 或声明值。

## <a name="editing-nameid"></a>编辑 NameID

若要编辑 NameID （名称标识符值）：

1. 打开**标识符值命名**页。
1. 选择你想要将应用于该属性的转换的属性。 或者，您可以指定他 NameID 声明具有所需的格式。

   ![编辑 NameID （名称标识符） 值](./media/active-directory-saml-claims-customization/saml-sso-manage-user-claims.png)

### <a name="nameid-format"></a>NameID 格式

如果 SAML 请求中包含具有特定格式的 NameIDPolicy 元素，Azure AD 将接受请求中的格式。

如果 SAML 请求不包含的元素的 NameIDPolicy，Azure AD 将使用您指定的格式发出 NameID。 如果没有格式指定，则 Azure AD 将使用与所选的声明源相关联的默认源格式。

从**选择名称标识符格式**下拉列表中，你可以选择以下选项之一。

| NameID 格式 | 描述 |
|---------------|-------------|
| **默认** | Azure AD 将使用默认源格式。 |
| **持久性** | Azure AD 将使用的永久作为 NameID 格式。 |
| **电子邮件地址** | Azure AD 将使用电子邮件地址作为 NameID 格式。 |
| **未指定** | Azure AD 将使用未指定作为 NameID 格式。 |
| **暂时性故障** | Azure AD 将使用暂时性作为 NameID 格式。 |

若要了解有关 NameIDPolicy 属性的详细信息，请参阅[单一登录 SAML 协议](single-sign-on-saml-protocol.md)。

### <a name="attributes"></a>属性

为 `NameIdentifier`（或 NameID）声明选择所需的源。 可以从以下选项中选择。

| 名称 | 描述 |
|------|-------------|
| 电子邮件 | 用户的电子邮件地址 |
| userprincipalName | 用户主体名称 (UPN) 的用户 |
| onpremisessamaccount | 已从本地 Azure AD 同步的 SAM 帐户名 |
| objectid | Azure AD 中的用户的 objectid |
| employeeid | 用户 employeeid 的 |
| 目录扩展 | 目录扩展[使用 Azure AD Connect 同步从本地 Active Directory 同步](../hybrid/how-to-connect-sync-feature-directory-extensions.md) |
| 扩展属性 1-15 | 用于扩展 Azure AD 架构的本地扩展属性 |

有关详细信息，请参阅[表 3:每个源的有效 ID 值](active-directory-claims-mapping.md#table-3-valid-id-values-per-source)。

### <a name="special-claims---transformations"></a>特殊声明转换

此外可以使用声明转换函数。

| 函数 | 描述 |
|----------|-------------|
| **ExtractMailPrefix()** | 从电子邮件地址或用户主体名称中删除的域后缀。 这只会提取传递用户名的第一部分（例如，“joe_smith”而不是 joe_smith@contoso.com）。 |
| **Join()** | 将属性与已验证的域联接。 如果所选用户标识符值具有域，则将提取用户名以追加所选的已验证域。 例如，如果选择电子邮件 (joe_smith@contoso.com) 作为用户标识符值，并选择 contoso.onmicrosoft.com 作为已验证的域，则将生成 joe_smith@contoso.onmicrosoft.com。 |
| **ToLower()** | 将所选属性的字符转换为小写字符。 |
| **ToUpper()** | 将所选属性的字符转换为大写字符。 |

## <a name="adding-application-specific-claims"></a>添加特定于应用程序的声明

若要添加特定于应用程序的声明：

1. 在中**用户属性和声明**，选择**添加新声明**以打开**管理用户声明**页。
1. 输入**名称**的声明。 值不严格需要遵循 URI 模式，每个 SAML 规格。如果您需要的 URI 模式，可以将它放**Namespace**字段。
1. 选择**源**声明在何处检索它的值。 可以从源属性下拉列表中选择用户属性，也可以以声明方式发出之前将转换应用于的用户属性。

### <a name="application-specific-claims---transformations"></a>特定于应用程序的声明的转换

此外可以使用声明转换函数。

| 函数 | 描述 |
|----------|-------------|
| **ExtractMailPrefix()** | 从电子邮件地址或用户主体名称中删除的域后缀。 这只会提取传递用户名的第一部分（例如，“joe_smith”而不是 joe_smith@contoso.com）。 |
| **Join()** | 通过联接两个属性创建一个新值。 （可选） 可以使用两个属性之间的分隔符。 |
| **ToLower()** | 将所选属性的字符转换为小写字符。 |
| **ToUpper()** | 将所选属性的字符转换为大写字符。 |
| **Contains()** | 如果输入与匹配指定的值将输出的属性或常量。 否则，可以指定另一个输出，如果没有匹配项。<br/>例如，如果你想要发出的声明中，值是用户的电子邮件地址，如果它包含在域"@contoso.com"，否则你想要输出的用户主体名称。 若要执行此操作，需要配置以下值：<br/>*参数 1(input)*: user.email<br/>*值*:"@contoso.com"<br/>参数 2 （输出）： user.email<br/>参数 3 （如果没有匹配项，则为输出）： user.userprincipalname |
| **EndWith()** | 如果输入结尾的指定值，则输出的属性或常量。 否则，可以指定另一个输出，如果没有匹配项。<br/>例如，如果你想要发出的声明中，值是用户的 employeeid 如果 employeeid 结尾"000"，否则你想要输出的扩展属性。 若要执行此操作，需要配置以下值：<br/>*参数 1(input)*: user.employeeid<br/>*值*："000"<br/>参数 2 （输出）： user.employeeid<br/>参数 3 （如果没有匹配项，则为输出）： user.extensionattribute1 |
| **StartWith()** | 如果输入开头指定的值，则输出的属性或常量。 否则，可以指定另一个输出，如果没有匹配项。<br/>例如，如果你想要发出其中的值是用户的 employeeid，如果国家/地区开始使用"美国"的声明，否则你想要输出的扩展属性。 若要执行此操作，需要配置以下值：<br/>*参数 1(input)*: user.country<br/>*值*："我们"<br/>参数 2 （输出）： user.employeeid<br/>参数 3 （如果没有匹配项，则为输出）： user.extensionattribute1 |
| **Extract （)-之后匹配** | 匹配指定的值后返回的子字符串。<br/>例如，如果输入的值为"Finance_BSimon"，匹配的值为"Finance_"，则声明的输出是"是 bsimon 作为用户名"。 |
| **Extract （)-之前匹配** | 返回子字符串，直到它与指定的值匹配。<br/>例如，如果输入的值为"BSimon_US"，匹配的值为"_US"，则声明的输出是"是 bsimon 作为用户名"。 |
| **Extract （)-之间匹配** | 返回子字符串，直到它与指定的值匹配。<br/>例如，如果输入的值为"Finance_BSimon_US"，第一个匹配值是"Finance_"，第二个匹配的值为"_US"，则声明的输出是"是 bsimon 作为用户名"。 |
| **ExtractAlpha()-前缀** | 返回前缀按字母顺序排列字符串的一部分。<br/>例如，如果输入的值为"BSimon_123"，然后它将返回"是 bsimon 作为用户名"。 |
| **ExtractAlpha() - Suffix** | 返回字符串的后缀按字母顺序排列部分。<br/>例如，如果输入的值为"123_Simon"，然后它将返回"是 bsimon 作为用户名"。 |
| **ExtractNumeric() - Prefix** | 返回字符串的前缀数字部分。<br/>例如，如果输入的值为"123_BSimon"，然后它将返回"123"。 |
| **ExtractNumeric() - Suffix** | 返回字符串的数字后缀部分。<br/>例如，如果输入的值为"BSimon_123"，然后它将返回"123"。 |
| **IfEmpty()** | 如果输入为 null 或为空，输出的属性或常量。<br/>例如，如果你想要输出的属性存储在 extensionattribute 给定用户的 employeeid 是否为空。 若要执行此操作，需要配置以下值：<br/>参数 1(input): user.employeeid<br/>参数 2 （输出）： user.extensionattribute1<br/>参数 3 （如果没有匹配项，则为输出）： user.employeeid |
| **IfNotEmpty()** | 如果输入不是 null 或为空，输出的属性或常量。<br/>例如，如果你想要输出的属性存储在 extensionattribute 给定用户的 employeeid 是否不为空。 若要执行此操作，需要配置以下值：<br/>参数 1(input): user.employeeid<br/>参数 2 （输出）： user.extensionattribute1 |

如果需要其他转换，提交在您的创意[Azure AD 中的反馈论坛](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160599)下*SaaS 应用程序*类别。

## <a name="next-steps"></a>后续步骤

* [Azure AD 中的应用程序管理](../manage-apps/what-is-application-management.md)
* [针对不在 Azure AD 应用程序库中的应用程序配置单一登录](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [排查基于 SAML 的单一登录的问题](howto-v1-debug-saml-sso-issues.md)
