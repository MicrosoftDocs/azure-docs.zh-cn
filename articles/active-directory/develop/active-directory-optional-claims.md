---
title: 了解如何向 Azure AD 应用程序提供可选声明 | Microsoft Docs
description: 有关如何将自定义或附加的声明添加到 Azure Active Directory 颁发的 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 的指南。
documentationcenter: na
author: CelesteDG
services: active-directory
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/27/2019
ms.author: celested
ms.reviewer: paulgarn, hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2fd7b05a5411c03e1324871fbff3c29061ce7b3d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139245"
---
# <a name="how-to-provide-optional-claims-to-your-azure-ad-app"></a>如何：提供向 Azure AD 应用程序的可选声明

应用程序开发人员可以使用此功能来指定要在发送到其应用程序的令牌中添加哪些声明。 使用可选声明可以：

- 选择要包含在应用程序令牌中的附加声明。
- 更改 Azure AD 在令牌中返回的某些声明的行为。
- 添加和访问应用程序的自定义声明。

有关标准声明的列表，请参阅[访问令牌](access-tokens.md)并[id_token](id-tokens.md)声明文档。 

尽管可选声明支持同时 v1.0 和 v2.0 格式标记，以及 SAML 令牌中，但它们提供大部分其值从 1.0 版到 2.0 版移动时。 [v2.0 Azure AD 终结点](active-directory-appmodel-v2-overview.md)的目标之一是缩小令牌大小，以确保客户端获得最佳性能。 因此，以前包含在访问令牌和 ID 令牌中的多个声明不再在 v2.0 令牌中提供，必须根据应用程序专门请求这些声明。

**表 1：适用性**

| 帐户类型 | V1.0 令牌 | V2.0 令牌  |
|--------------|---------------|----------------|
| Microsoft 个人帐户  | 不适用  | 支持|
| Azure AD 帐户      | 支持 | 支持 |

## <a name="v10-and-v20-optional-claims-set"></a>V1.0 和 V2.0 可选声明集

下面列出了默认可对应用程序使用的可选声明集。 若要为应用程序添加自定义可选声明，请参阅下面的[目录扩展](#configuring-directory-extension-optional-claims)。 时将声明添加到**访问令牌**，这将应用于请求的访问令牌*有关*的应用程序 (web API)，不是那些*通过*应用程序。 这可确保无论哪个客户端访问你的 API，正确的数据都存在于用于对你的 API 进行身份验证的访问令牌中。

> [!NOTE]
> 其中的大多数声明可包含在 v1.0 和 v2.0 令牌的 JWT 中，但不可包含在 SAML 令牌中，“令牌类型”列中指明的声明除外。 使用者帐户支持这些声明，在"用户类型"列中标记的一个子集。  许多所列声明不适用于使用者用户 (它们有没有租户，因此`tenant_ctry`没有值)。  

**表 2：V1.0 和 V2.0 可选声明集**

| 名称                       |  描述   | 令牌类型 | 用户类型 | 说明  |
|----------------------------|----------------|------------|-----------|--------|
| `auth_time`                | 用户上次进行身份验证的时间。 请参阅 OpenID Connect 规范。| JWT        |           |  |
| `tenant_region_scope`      | 资源租户的区域 | JWT        |           | |
| `home_oid`                 | 对于来宾用户，表示该用户在用户主租户中的对象 ID。| JWT        |           | |
| `sid`                      | 用于每个会话用户注销的会话 ID。 | JWT        |  个人和 Azure AD 帐户。   |         |
| `platf`                    | 设备平台    | JWT        |           | 限制为可以验证设备类型的托管设备。|
| `verified_primary_email`   | 源自用户的 PrimaryAuthoritativeEmail      | JWT        |           |         |
| `verified_secondary_email` | 源自用户的 SecondaryAuthoritativeEmail   | JWT        |           |        |
| `enfpolids`                | 强制实施的策略 ID。 针对当前用户评估的策略 ID 列表。 | JWT |  |  |
| `vnet`                     | VNET 说明符信息。 | JWT        |           |      |
| `fwd`                      | IP 地址。| JWT    |   | 添加请求方客户端（如果位于 VNET 中）的原始 IPv4 地址 |
| `ctry`                     | 用户所在的国家/地区 | JWT |  | Azure AD 返回 `ctry` 可选声明（如果存在）且声明的值是标准的双字母国家/地区代码，例如 FR、JP、SZ 等。 |
| `tenant_ctry`              | 资源租户所在的国家/地区 | JWT | | |
| `xms_pdl`          | 首选数据位置   | JWT | | 对于多地域的租户，这是显示该用户所在的地理区域的 3 个字母代码。 有关详细信息，请参阅[首选的数据位置有关的 Azure AD Connect 文档](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-feature-preferreddatalocation)。<br/>例如：`APC` 表示“亚太”。 |
| `xms_pl`                   | 用户首选语言  | JWT ||用户的首选语言（如果已设置）。 在来宾访问方案中，源自其主租户。 已格式化 LL-CC（“en-us”）。 |
| `xms_tpl`                  | 租户首选语言| JWT | | 资源租户的首选语言（如果已设置）。 已格式化 LL（“en”）。 |
| `ztdid`                    | 零接触部署 ID | JWT | | 用于 [Windows AutoPilot](https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-10-autopilot) 的设备标识 |
| `email`                    | 此用户的可寻址电子邮件（如果此用户有）。  | JWT、SAML | MSA, AAD | 如果用户是租户中的来宾，则默认包含此值。  对于托管用户（租户内部的用户），必须通过此可选声明进行请求，或者仅在 v2.0 上使用 OpenID 范围进行请求。  对于托管用户，必须在 [Office 管理门户](https://portal.office.com/adminportal/home#/users)中设置电子邮件地址。| 
| `groups`| 可选的组声明格式设置 |JWT、SAML| |与中的 GroupMembershipClaims 设置结合使用[应用程序清单](reference-app-manifest.md)，其必须也设置。 有关详细信息，请参阅[组声明](#Configuring-group-optional claims)下面。 组声明的详细信息请参阅[如何配置组声明](../hybrid/how-to-connect-fed-group-claims.md)
| `acct`             | 租户中的用户帐户状态。 | JWT、SAML | | 如果用户是租户的成员，则该值为 `0`。 如果他们是来宾，则该值为 `1`。 |
| `upn`                      | UserPrincipalName 声明。 | JWT、SAML  |           | 尽管会自动包含此声明，但可以将它指定为可选声明，以附加额外的属性，在来宾用例中修改此声明的行为。  |

### <a name="v20-optional-claims"></a>v2.0 可选声明

始终包括在 1.0 版 Azure AD 令牌中，但不是包含在 v2.0 令牌中，除非请求这些声明。 这些声明是仅适用于 Jwt （ID 令牌和访问令牌）。 

**表 3：仅限 V2.0 的可选声明**

| JWT 声明     | 名称                            | 描述                                | 说明 |
|---------------|---------------------------------|-------------|-------|
| `ipaddr`      | IP 地址                      | 客户端从中登录的 IP 地址。   |       |
| `onprem_sid`  | 本地安全标识符 |                                             |       |
| `pwd_exp`     | 密码过期时间        | 密码过期的日期时间。 |       |
| `pwd_url`     | 更改密码 URL             | 用户更改密码时可以访问的 URL。   |   |
| `in_corp`     | 企业网络内部        | 表示客户端是否从企业网络登录。 如果它们不声明不包括。   |  以 MFA 中的[可信 IP](../authentication/howto-mfa-mfasettings.md#trusted-ips) 设置为基础。    |
| `nickname`    | 别名                        | 用户的附加名称，不同于名字或姓氏。 | 
| `family_name` | 姓氏                       | 提供了最后一个名称、 姓氏或家族名称的用户的用户对象中定义。 <br>"family_name":"Miller" | MSA 和 AAD 中受支持   |
| `given_name`  | 名字                      | 提供了第一个或用户对象上设置为"给定"的用户的名称。<br>"given_name":"Frank"                   | MSA 和 AAD 中受支持  |
| `upn`         | 用户主体名称 | 可以与 username_hint 参数一起使用的用户标识符。  不是用户的持久标识符，不应当用于关键数据。 | 有关声明配置，请参阅下面的[附加属性](#additional-properties-of-optional-claims)。 |

### <a name="additional-properties-of-optional-claims"></a>可选声明的附加属性

可以配置某些可选声明来更改声明的返回方式。 这些附加属性主要用于帮助迁移具有不同数据预期的本地应用程序（例如，`include_externally_authenticated_upn_without_hash` 可帮助迁移无法处理 UPN 中的井号标记 (`#`) 的客户端）

**表 4：用于配置可选声明的值**

| 属性名称  | 附加属性名称 | 描述 |
|----------------|--------------------------|-------------|
| `upn`          |                          | 可用于 SAML 和 JWT 响应，以及 v1.0 和 v2.0 令牌。 |
|                | `include_externally_authenticated_upn`  | 包含资源租户中存储的来宾 UPN。 例如： `foo_hometenant.com#EXT#@resourcetenant.com` |             
|                | `include_externally_authenticated_upn_without_hash` | 同上，不过，井号标记 (`#`) 已替换为下划线 (`_`)，例如 `foo_hometenant.com_EXT_@resourcetenant.com` |

#### <a name="additional-properties-example"></a>附加属性示例

```json
 "optionalClaims": 
   {
       "idToken": [ 
             { 
                "name": "upn", 
            "essential": false,
                "additionalProperties": [ "include_externally_authenticated_upn"]  
              }
        ]
}
```

此 OptionalClaims 对象会导致返回到客户端的 ID 令牌包含另一个 UPN 及其他主租户和资源租户信息。 仅当用户是租户中的来宾（使用不同的 IDP 进行身份验证）时，这才会更改令牌中的 `upn` 声明。 

## <a name="configuring-optional-claims"></a>配置可选声明

可以通过修改应用程序清单来配置应用程序的可选声明 （请参阅下面的示例）。 有关详细信息，请参阅[了解 Azure AD 应用程序清单文章](reference-app-manifest.md)。

**示例架构：**

```json
"optionalClaims":  
   {
      "idToken": [
            {
                  "name": "auth_time", 
                  "essential": false
             }
      ],
      "accessToken": [
             {
                    "name": "ipaddr", 
                    "essential": false
              }
      ],
      "saml2Token": [
              {
                    "name": "upn", 
                    "essential": false
               },
               {
                    "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                    "source": "user", 
                    "essential": false
               }
       ]
   }
```

### <a name="optionalclaims-type"></a>OptionalClaims 类型

声明应用程序请求的可选声明。 应用程序可以配置为要在它能够从安全令牌服务收到的每种令牌（共三种，即 ID 令牌、访问令牌和 SAML 2 令牌）中返回可选声明。 应用程序可以配置为要在每种令牌类型中返回一组不同的可选声明。 Application 实体的 OptionalClaims 属性是一个 OptionalClaims 对象。

**表 5：OptionalClaims 类型属性**

| 名称        | 类型                       | 描述                                           |
|-------------|----------------------------|-------------------------------------------------------|
| `idToken`     | 集合 (OptionalClaim) | 在 JWT ID 令牌中返回的可选声明。 |
| `accessToken` | 集合 (OptionalClaim) | 在 JWT 访问令牌中返回的可选声明。 |
| `saml2Token`  | 集合 (OptionalClaim) | 在 SAML 令牌中返回的可选声明。   |

### <a name="optionalclaim-type"></a>OptionalClaim 类型

包含与应用程序或服务主体关联的可选声明。 [OptionalClaims](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#optionalclaims-type) 类型的 IdToken、accessToken 和 saml2Token 属性是一个 OptionalClaim 集合。
如果特定的声明支持这样做，则还可以使用 AdditionalProperties 字段修改 OptionalClaim 的行为。

**表 6：OptionalClaim 类型属性**

| 名称                 | 类型                    | 描述                                                                                                                                                                                                                                                                                                   |
|----------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                 | Edm.String              | 可选声明的名称。                                                                                                                                                                                                                                                                           |
| `source`               | Edm.String              | 声明的源（目录对象）。 扩展属性提供预定义声明和用户定义的声明。 如果源值为 null，则声明是预定义的可选声明。 如果源值为 user，则 name 属性中的值是来自用户对象的扩展属性。 |
| `essential`            | Edm.Boolean             | 如果值为 true，则必须使用客户端指定的声明，以确保为最终用户请求的特定任务提供顺利的授权体验。 默认值为 false。                                                                                                             |
| `additionalProperties` | 集合 (Edm.String) | 声明的附加属性。 如果此集合中存在某个属性，该属性将修改 name 属性中指定的可选声明的行为。                                                                                                                                               |
## <a name="configuring-directory-extension-optional-claims"></a>配置目录扩展的可选声明

除了标准的可选声明集之外，还可以配置标记以包含目录架构扩展。 有关详细信息，请参阅[目录架构扩展](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions)。 使用此功能可以附加应用可以使用的附加用户信息 – 例如，用户设置的附加标识符或重要配置选项。 

> [!Note]
> 目录架构扩展功能只能在 AAD 中使用，因此，如果应用程序清单请求自定义扩展，而 MSA 用户登录到你的应用，则不会返回这些扩展。

### <a name="directory-extension-formatting"></a>目录扩展格式设置

对于扩展属性，请在应用程序清单中使用扩展的完整名称（格式：`extension_<appid>_<attributename>`）。 `<appid>`必须匹配请求声明的应用程序的 ID。 

在 JWT 中，将使用以下命名格式发出这些声明：`extn.<attributename>`。

在 SAML 令牌中，将使用以下 URI 格式发出这些声明：`http://schemas.microsoft.com/identity/claims/extn.<attributename>`。

## <a name="configuring-group-optional-claims"></a>配置组的可选声明

   > [!NOTE]
   > 若要发出的用户和组从本地同步的组名称的能力是公共预览版

本部分介绍了用于更改从默认组 objectID 属性从本地 Windows Active Directory 同步的组声明中使用的组属性的可选声明下的配置选项
> [!IMPORTANT]
> 请参阅[使用 Azure Active Directory 中配置应用程序的组声明](../hybrid/how-to-connect-fed-group-claims.md)了解更多详细信息，包括从在本地属性的组声明的公共预览版的重要注意事项。

1. 在门户中-> Azure Active Directory-> 应用程序注册-> 选择应用程序-> 清单

2. 通过更改 groupMembershipClaim 启用组成员身份声明

   有效值为：

   - "全部"
   - "SecurityGroup"
   - "DistributionList"
   - "DirectoryRole"

   例如：

   ```json
   "groupMembershipClaims": "SecurityGroup"
   ```

   默认情况下将在组中发出组 Objectid 声明值。  若要修改的声明值以包含本地组属性，或声明类型更改为角色，请按如下所示使用 OptionalClaims 配置：

3. 设置组名称配置可选声明。

   如果你想向令牌中的组要包含在本地 AD 组属性的可选声明部分中指定的标记类型的可选声明应应用于、 请求的可选声明和所需的任何其他属性的名称。  可以列出多个令牌类型：

   - OIDC ID 令牌的 idToken
   - accessToken OAuth/OIDC 访问令牌
   - SAML 令牌 Saml2Token。

   > [!NOTE]
   > Saml2Token 类型适用于 SAML1.1 和 SAML2.0 格式标记

   对于每个相关的标记类型，修改要使用 OptionalClaims 部分在清单中的组声明。 OptionalClaims 架构如下所示：

   ```json
   {
   "name": "groups",
   "source": null,
   "essential": false,
   "additionalProperties": []
   }
   ```

   | 可选声明架构 | 值 |
   |----------|-------------|
   | **名称：** | 必须是"组" |
   | **源：** | 不使用。 省略或指定 null |
   | **重要：** | 不使用。 省略或指定 false |
   | **additionalProperties:** | 其他属性的列表。  Valid options are "sam_account_name", “dns_domain_and_sam_account_name”, “netbios_domain_and_sam_account_name”, "emit_as_roles" |

   在只有一个"sam_account_name"，"dns_domain_and_sam_account_name"additionalProperties"netbios_domain_and_sam_account_name"是必需的。  如果存在多个，使用第一个和任何其他人所忽略。

   某些应用程序需要有关角色声明中的用户组信息。  若要更改要从组声明与角色声明，将"emit_as_roles"添加到附加属性的声明类型。  将角色声明中发出组值。

   > [!NOTE]
   > 如果使用"emit_as_roles"任何应用程序角色配置为用户分配将不会出现在角色声明

**示例：** 发出组作为 dnsDomainName\sAMAccountName 格式中的 OAuth 访问令牌中的组名称

```json
"optionalClaims": {
    "accessToken": [{
        "name": "groups",
        "additionalProperties": ["dns_domain_and_sam_account_name"]
    }]
}
 ```

若要发出角色声明在 SAML 和 OIDC ID 令牌中作为返回 netbiosDomain\sAMAccountName 格式中的组名称：

```json
"optionalClaims": {
    "saml2Token": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }],

    "idToken": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }]
 }

 ```

## <a name="optional-claims-example"></a>可选声明示例

本部分通过一个场景来演练如何对应用程序使用可选声明功能。
可以使用多个选项来更新应用程序标识配置中的属性，以启用和配置可选声明：
-   可以修改应用程序清单。 下面的示例将使用此方法执行配置。 请先阅读[了解 Azure AD 应用程序清单文档](https://docs.microsoft.com/azure/active-directory/develop/active-directory-application-manifest)中的清单简介。
-   也可以编写使用[图形 API](https://docs.microsoft.com/azure/active-directory/develop/active-directory-graph-api) 的应用程序来更新应用程序。 图形 API 参考指南中的[实体和复杂类型参考](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#optionalclaims-type)可帮助你配置可选声明。

**示例：** 在以下示例中，我们将修改某个应用程序的清单，以便将声明添加到用于该应用程序的访问令牌、ID 令牌和 SAML 令牌。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 通过身份验证后，在页面右上角选择 Azure AD 租户。
1. 选择左侧的“应用注册”。
1. 在列表中找到要为其配置可选声明的应用程序并单击它。
1. 在应用程序页面中，单击“清单”打开内联清单编辑器。 
1. 可使用此编辑器直接编辑清单。 该清单遵循 [Application 实体](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest)的架构，保存后会自动设置格式。 新元素将添加到 `OptionalClaims` 属性。

    ```json
      "optionalClaims": 
      {
            "idToken": [ 
                  { 
                        "name": "upn", 
                        "essential": false, 
                        "additionalProperties": [ "include_externally_authenticated_upn"]  
                  }
            ],
            "accessToken": [ 
                  {
                        "name": "auth_time", 
                        "essential": false
                  }
            ],
            "saml2Token": [ 
                  { 
                        "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                        "source": "user", 
                        "essential": true
                  }
            ]
      }

    ```

    在本例中，已将不同的可选声明添加到应用程序可以接收的每种令牌。 ID 令牌现在会包含联合用户的完整格式 UPN (`<upn>_<homedomain>#EXT#@<resourcedomain>`)。 其他客户端请求此应用程序的访问令牌现在将包含 auth_time 声明。 SAML 令牌现在会包含 skypeId 目录架构扩展（在本例中，此应用的应用 ID 为 ab603c56068041afb2f6832e2a17e237）。 SAML 令牌会将 Skype ID 公开为 `extension_skypeId`。

1. 更新完清单后，请单击“保存”以保存清单

## <a name="next-steps"></a>后续步骤

详细了解 Azure AD 提供的标准声明。

- [ID 令牌](id-tokens.md)
- [访问令牌](access-tokens.md)
