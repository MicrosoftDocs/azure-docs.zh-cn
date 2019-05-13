---
title: Azure AD B2C （适用于.NET 的 Microsoft 身份验证库） |Azure
description: 使用 Microsoft 身份验证库.NET (MSAL.NET) 使用 Azure AD B2C 时，了解有关特定注意事项。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5c608518a9eb80d807297f010778ae452c0f61f5
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075770"
---
# <a name="use-msalnet-to-sign-in-users-with-social-identities"></a>使用 MSAL.NET 具有社交标识的用户登录

可以使用 MSAL.NET 若要通过使用具有社交标识的用户登录[Azure Active Directory B2C (Azure AD B2C)](https://aka.ms/aadb2c)。 Azure AD B2C 是围绕策略这一概念构建的。 在 MSAL.NET，指定策略转换为提供证书颁发机构。

- 实例化的公共客户端应用程序，需要在颁发机构中指定的策略。
- 当你想要应用策略时，需要调用的重写`AcquireTokenInteractive`包含`authority`参数。

此页是 msal 3.x。 如果您有兴趣 MSAL 2.x，请参阅[MSAL 中的 Azure AD B2C 细节 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-Specifics-MSAL-2.x)。

## <a name="authority-for-a-azure-ad-b2c-tenant-and-policy"></a>用于 Azure AD B2C 租户和策略的颁发机构

若要使用的颁发机构是`https://login.microsoftonline.com/tfp/{tenant}/{policyName}`其中：

- `tenant` 是的在 Azure AD B2C 租户名称 
- `policyName` 要将应用 (例如"b2c_1_susi"的登录-在/注册) 的策略的名称。

Azure AD B2C 的当前指导原则是使用`b2clogin.com`用作颁发机构。 例如，`$"https://{your-tenant-name}.b2clogin.com/tfp/{your-tenant-ID}/{policyname}"`。 有关详细信息，请参阅此[文档](/azure/active-directory-b2c/b2clogin)。

## <a name="instantiating-the-application"></a>实例化应用程序

生成程序时，你需要提供了的权限。

```csharp
// Azure AD B2C Coordinates
public static string Tenant = "fabrikamb2c.onmicrosoft.com";
public static string ClientID = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
public static string PolicySignUpSignIn = "b2c_1_susi";
public static string PolicyEditProfile = "b2c_1_edit_profile";
public static string PolicyResetPassword = "b2c_1_reset";

public static string AuthorityBase = $"https://fabrikamb2c.b2clogin.com/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
public static string AuthorityEditProfile = $"{AuthorityBase}{PolicyEditProfile}";
public static string AuthorityPasswordReset = $"{AuthorityBase}{PolicyResetPassword}";

application = PublicClientApplicationBuilder.Create(ClientID)
               .WithB2CAuthority(Authority)
               .Build();
```

## <a name="acquire-a-token-to-apply-a-policy"></a>获取令牌以应用策略

获取 Azure AD b2c 令牌公共客户端应用程序中的受保护的 API 需要使用替代与颁发机构：

```csharp
IEnumerable<IAccount> accounts = await application.GetAccountsAsync();
AuthenticationResult ar = await application .AcquireToken(scopes, parentWindow)
                                            .WithAccount(GetAccountByPolicy(accounts, policy))
                                            .ExecuteAsync();
```

替换为：

- `policy` 作为一项的上一个字符串 (例如`PolicySignUpSignIn`)。
- `GetAccountByPolicy(IEnumerable<IAccount>, string)` 是方法，用于查找给定策略的帐户。 例如：

  ```csharp
  private IAccount GetAccountByPolicy(IEnumerable<IAccount> accounts, string policy)
  {
   foreach (var account in accounts)
   {
    string userIdentifier = account.HomeAccountId.ObjectId.Split('.')[0];
    if (userIdentifier.EndsWith(policy.ToLower()))
     return account;
   }
   return null;
  }
  ```

应用策略 （例如让最终用户编辑其配置文件或重置其密码），当前可通过调用`AcquireTokenInteractive`。 在这两种策略的情况下不使用返回的令牌 / 身份验证结果。

## <a name="special-case-of-editprofile-and-resetpassword-policies"></a>EditProfile 和 ResetPassword 策略的特殊情况

当你想要提供你的最终用户在其中使用社交标识登录的体验，然后编辑其配置文件时要应用的 Azure AD B2C EditProfile 策略。 若要执行此操作，方法是通过调用`AcquireTokenInteractive`通过特定颁发机构的策略和设置为提示`Prompt.NoPrompt`以避免帐户选择对话框中显示 （如用户已登录的）

```csharp
private async void EditProfileButton_Click(object sender, RoutedEventArgs e)
{
 IEnumerable<IAccount> accounts = await app.GetAccountsAsync();
 try
 {
  var authResult = await app.AcquireToken(scopes:App.ApiScopes)
                               .WithAccount(GetUserByPolicy(accounts, App.PolicyEditProfile)),
                               .WithPrompt(Prompt.NoPrompt),
                               .WithB2CAuthority(App.AuthorityEditProfile)
                               .ExecuteAsync();
  DisplayBasicTokenInfo(authResult);
 }
 catch
 {
  . . .
 }
}
```
## <a name="resource-owner-password-credentials-ropc-with-azure-ad-b2c"></a>资源所有者密码凭据 (ROPC) 与 Azure AD B2C
有关 ROPC 流的更多详细信息，请参阅此[文档](v2-oauth-ropc.md)。

此流**不建议这样做**因为询问用户其密码的应用程序的不安全。 有关此问题的详细信息，请参阅[这篇文章](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/)。 

通过使用用户名/密码，你将为提供向上执行大量操作：
- 核心的现代的标识租户： 获取应用密码，重播。 因为我们有这一概念会被截取的共享机密。 这是与无密码不兼容。
- 需要执行 MFA 的用户将无法登录 （因为无需交互）。
- 用户将无法进行单一登录。

### <a name="configure-the-ropc-flow-in-azure-ad-b2c"></a>在 Azure AD B2C 配置 ROPC 流
在 Azure AD B2C 租户中，创建一个新的用户流，并选择**使用 ROPC 登录**。 这将为租户启用 ROPC 策略。 请参阅[配置资源所有者密码凭据流](/azure/active-directory-b2c/configure-ropc)的更多详细信息。

`IPublicClientApplication` 包含一种方法：
```csharp
AcquireTokenByUsernamePassword(
            IEnumerable<string> scopes,
            string username,
            SecureString password)
```

此方法将作为参数：
- *作用域*请求的访问令牌。
- 一个*用户名*。
- SecureString*密码*用户。

请记住使用了包含 ROPC 策略的权限。

### <a name="limitations-of-the-ropc-flow"></a>ROPC 流的限制
 - ROPC 流**仅适用于本地帐户**(其中注册到 Azure AD B2C 使用电子邮件或用户名)。 如果对任何支持的 Azure AD B2C 的标识提供程序联合此流不起作用 (Facebook、 Google、 等等。)。
 - 目前，没有**从 Azure AD B2C 返回没有 id_token**实现从 MSAL ROPC 流时。 这意味着不能创建一个帐户对象，以便在缓存中，会导致没有帐户，并且没有用户。 AcquireTokenSilent 流不会在此方案中。 但是，ROPC 不显示 UI，因此将对用户体验产生任何影响。

## <a name="google-auth-and-embedded-webview"></a>Google 身份验证和嵌入式 web 视图

如果你是 Azure AD B2C 开发人员使用 Google 作为标识提供者我们建议你使用系统浏览器，因为 Google 不允许[嵌入 webview 从身份验证](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html)。 目前，`login.microsoftonline.com`是使用 Google 的受信任颁发机构。 使用此颁发机构将使用嵌入式 web 视图。 但是使用`b2clogin.com`不是 Google，使用受信任颁发机构，因此用户将无法进行身份验证。

我们将提供的更新 wiki，这[问题](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/688)情况发生变化时。

## <a name="caching-with-azure-ad-b2c-in-msalnet"></a>使用 Azure AD B2C 中 MSAL.Net 缓存 

### <a name="known-issue-with-azure-ad-b2c"></a>与 Azure AD B2C 的已知的问题

支持 MSAL.Net[令牌缓存](/dotnet/api/microsoft.identity.client.tokencache?view=azure-dotnet)。 缓存密钥的令牌基于标识提供程序返回的声明。 当前 MSAL.Net 需要两个声明来生成令牌缓存密钥：  
- `tid` 这是 Azure AD 租户 ID，并 
- `preferred_username` 

许多 Azure AD B2C 方案中缺少这两个这些声明。 

对客户的影响是，在尝试显示用户名字段，你是否得到了"在令牌响应中的缺少"作为值？ 如果是这样，这是因为 Azure AD B2C 不返回值中的 preferred_username IdToken 由于使用社交帐户和外部标识提供者 (Idp) 的限制。 Azure AD 返回 preferred_username 的值，因为它知道谁是用户，但对于 Azure AD B2C 中，因为用户可以使用本地帐户、 Facebook、 Google、 GitHub、 登录等有不是 Azure AD b2c 要用于 preferred_username 一致的值。 若要取消 MSAL 阻止从推出与 ADAL 缓存兼容性，我们决定时处理的 Azure AD B2C 帐户 IdToken 不返回任何内容的 preferred_username 时，对我们这端使用"令牌响应中的缺少"。 MSAL 必须返回 preferred_username 以在库间保持缓存兼容性的值。

### <a name="workarounds"></a>解决方法

#### <a name="mitigation-for-the-missing-tenant-id"></a>缺少租户 id 的缓解措施

建议的解决方法是使用[缓存策略](#acquire-a-token-to-apply-a-policy)

或者，可以使用`tid`声明，如果使用的[B2C 自定义策略](https://aka.ms/ief)，因为它提供的功能，以返回到应用程序的其他声明。 若要详细了解[声明转换](/azure/active-directory-b2c/claims-transformation-technical-profile)

#### <a name="mitigation-for-missing-from-the-token-response"></a>"令牌响应中缺少"的缓解措施
一种方法是使用"name"声明作为首选的用户名。 在此提到过程[B2C 文档](/azure/active-directory-b2c/active-directory-b2c-reference-policies#frequently-asked-questions)->"在返回的声明栏中选择你希望在发回给你成功配置文件编辑体验后的应用程序的授权令牌中返回的声明。 例如，选择显示名称、 邮政编码。"

## <a name="next-steps"></a>后续步骤 

下面的示例中提供了有关获取令牌以交互方式使用 MSAL.NET 的 Azure AD B2C 应用程序的更多详细信息。

| 示例 | 平台 | 描述|
|------ | -------- | -----------|
|[active-directory-b2c-xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Xamarin iOS、 Xamarin Android UWP | 简单的 Xamarin Forms 应用，它展示了如何使用 MSAL.NET 通过 Azure AD B2C 用户进行身份验证和访问 Web API 使用生成的令牌。|
