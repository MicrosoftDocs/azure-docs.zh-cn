---
title: 调用 web Api （获取应用令牌）-Microsoft 标识平台的桌面应用程序
description: 了解如何构建桌面应用调用 web Api (获取令牌的应用程序 |)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: e1fe9594471c6e8f723afff2def940bb675e04fb
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407006"
---
# <a name="desktop-app-that-calls-web-apis---acquire-a-token"></a>调用 web Api-的桌面应用程序获取令牌

构建您后`IPublicClientApplication`，将使用它来获取令牌，你将使用它来调用 web API。

## <a name="recommended-pattern"></a>建议的模式

Web API 定义的其`scopes`。 在应用程序中提供的任何的体验将想要使用的模式是：

- 系统地尝试通过调用从令牌缓存中获取令牌 `AcquireTokenSilent`
- 如果此调用失败，则使用`AcquireToken`想要使用的流 (此处由`AcquireTokenXX`)

```CSharp
AuthenticationResult result;
var accounts = await app.GetAccountsAsync();
IAccount account = ChooseAccount(accounts); // for instance accounts.FirstOrDefault
                                            // if the app manages is at most one account  
try
{
 result = await app.AcquireTokenSilent(scopes, account)
                   .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
  result = await app.AcquireTokenXX(scopes, account)
                    .WithOptionalParameterXXX(parameter)
                    .ExecuteAsync();
}
```

下面是现在获取令牌的桌面应用程序的各种方法的详细信息

## <a name="acquiring-a-token-interactively"></a>以交互方式获取令牌

下面的示例显示了最少的代码来读取用户的配置文件使用 Microsoft Graph 以交互方式获取的令牌。

```CSharp
string[] scopes = new string["user.read"];
var app = PublicClientApplicationBuilder.Create(clientId).Build();
var accounts = await app.GetAccountsAsync();
AuthenticationResult result;
try
{
 result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
             .ExecuteAsync();
}
catch(MsalUiRequiredException)
{
 result = await app.AcquireTokenInteractive(scopes)
             .ExecuteAsync();
}
```

### <a name="mandatory-parameters"></a>必需参数

`AcquireTokenInteractive` 只有一个必需参数``scopes``，其中包含定义为其令牌是必需的作用域的字符串的枚举。 如果令牌是针对 Microsoft Graph，可以是所需作用域的每个 Microsoft 图形 API 的 api 参考中找到名为"权限"部分中。 例如，向[列出用户的联系人](https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/user_list_contacts)，将需要使用作用域"User.Read"，"Contacts.Read"。 另请参阅[Microsoft Graph 权限引用](https://developer.microsoft.com/graph/docs/concepts/permissions_reference)。

您需要在 Android 上，还指定父活动 (使用`.WithParentActivityOrWindow`，如下所示)，以便在令牌获取返回到该父活动后交互。 调用时，如果未指定该，将引发异常`.ExecuteAsync()`。

### <a name="specific-optional-parameters"></a>特定的可选参数

#### <a name="withparentactivityorwindow"></a>WithParentActivityOrWindow

交互式、 UI 是重要。 `AcquireTokenInteractive` 有一个特定的可选参数启用指定的平台支持，父 UI。 在桌面应用程序中使用时`.WithParentActivityOrWindow`具有不同的类型取决于平台：

```CSharp
// net45
WithParentActivityOrWindow(IntPtr windowPtr)
WithParentActivityOrWindow(IWin32Window window)

// Mac
WithParentActivityOrWindow(NSWindow window)

// .Net Standard (this will be on all platforms at runtime, but only on NetStandard at build time)
WithParentActivityOrWindow(object parent).
```

备注：

- 在.NET 标准版，预期`object`是`Activity`在 Android 上，`UIViewController`在 iOS 上，`NSWindow`在 MAC 上，和一个`IWin32Window`或`IntPr`在 Windows 上。
- 在 Windows 中，必须调用`AcquireTokenInteractive`从 UI 线程，以便嵌入式浏览器获取相应的 UI 同步上下文。  不从 UI 线程调用可能导致消息不抽取正确和/或死锁与 UI 的方案。 一种方法从 UI 线程调用 MSAL，如果你不在 UI 线程上的已是使用`Dispatcher`wpf。
- 如果使用 WPF 中，若要在 WPF 控件中获取一个窗口，则可以使用`WindowInteropHelper.Handle`类。 调用是然后，在 WPF 控件中 (`this`):
  
  ```CSharp
  result = await app.AcquireTokenInteractive(scopes)
                    .WithParentActivityOrWindow(new WindowInteropHelper(this).Handle)
                    .ExecuteAsync();
  ```

#### <a name="withprompt"></a>WithPrompt

`WithPrompt()` 用于通过指定一条提示控制与用户交互功能

<img src="https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png" width="25%" />

类定义以下常量：

- ``SelectAccount``： 将强制 STS 以显示帐户选择对话框包含用户对其具有一个会话的帐户。 当应用程序开发人员想要让用户在不同的标识之间进行选择时，此选项非常有用。 此选项驱动器 MSAL 发送``prompt=select_account``到标识提供程序。 默认情况下，此选项，它执行的很好地提供尽可能最佳的体验根据可用信息 （帐户、 用户和等等的会话的状态。 ...).不要更改它，除非有充分的理由来执行此操作。
- ``Consent``： 使应用程序开发人员强制用户即使授予许可后之前提示同意。 在这种情况下，MSAL 将发送`prompt=consent`到标识提供程序。 此选项可以使用某些安全方面的考虑应用程序中的组织的监管要求每次使用应用程序时，则用户会看到同意对话框。
- ``ForceLogin``： 使应用程序开发人员能够提示用户输入凭据由服务即使不需要此用户提示。 此选项很有用，如果获取令牌失败，以便用户可以重新登录。 在这种情况下，MSAL 将发送`prompt=login`到标识提供程序。 同样，我们已经看到了一些安全方面的考虑应用程序中使用组织管理的要求，用户 relogs-在每次访问应用程序的特定部分。
- ``Never`` （适用于.NET 4.5 和 WinRT 仅） 不会提示用户，但改为将尝试使用存储在隐藏的嵌入式的 web 视图中的 cookie (如下所示：Web 视图中 MSAL.NET）。 使用此选项可能会失败，并在这种情况下`AcquireTokenInteractive`将引发异常来通知，需要 UI 交互，并将需要使用另一个`Prompt`参数。
- ``NoPrompt``：不会向标识提供者发送任何提示。 此选项才适用于 Azure AD B2C 编辑配置文件策略 (请参阅[B2C 细节](https://aka.ms/msal-net-b2c-specificities))。

#### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

使用此修饰符是一种高级方案中要预先同意提前几个资源的用户 (不想要使用通常用于 MSAL.NET 增量许可和 / Microsoft 标识平台 v2.0)。 有关详细信息，请参阅[操作方法： 让用户同意使用多个资源提前](scenario-desktop-production.md#how-to-have--the-user-consent-upfront-for-several-resources)。

```CSharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

#### <a name="withcustomwebui"></a>WithCustomWebUi

##### <a name="withcustomwebui-is-an-extensibility-point"></a>WithCustomWebUi 是一个扩展点

`WithCustomWebUi` 是一个扩展点允许你提供自己的 UI 中公共客户端应用程序，并允许用户通过 /Authorize 终结点的标识提供者并让其登录并同意。 然后，MSAL.NET 可以兑换的身份验证代码并获取令牌。 它对于实例用于在 Visual Studio 中有电子产生应用程序 （例如 VS 反馈） 提供的 web 交互，但它留给 MSAL.NET 来完成大部分工作。 此外可以使用它如果您想要提供 UI 自动化。 MSAL.NET 公共客户端应用程序中使用 PKCE 标准 ([RFC 7636-代码 Exchange OAuth 公共客户端的 Proof Key](https://tools.ietf.org/html/rfc7636)) 以确保遵守安全：仅 MSAL.NET 可以兑换代码。

  ```CSharp
  using Microsoft.Identity.Client.Extensions;
  ```

##### <a name="how-to-use-withcustomwebui"></a>如何使用 WithCustomWebUi

若要使用`.WithCustomWebUI`，你需要：
  
  1. 实现`ICustomWebUi`接口 (请参阅[此处](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/src/Microsoft.Identity.Client/Extensibility/ICustomWebUI.cs#L32-L70)。 基本上需要实现一种方法`AcquireAuthorizationCodeAsync`接受 （由 MSAL.NET 计算） 的授权代码 URL，让用户通过与标识提供程序的交互并返回回依据将标识提供程序的 URL调用返回 （包括授权代码） 实现。 如果遇到问题，您的实现应引发`MsalExtensionException`能够进行很好地合作，使用 MSAL 的异常。
  2. 在你`AcquireTokenInteractive`调用时，可以使用`.WithCustomUI()`修饰符将您的自定义的实例传递 web UI

     ```CSharp
     result = await app.AcquireTokenInteractive(scopes)
                       .WithCustomWebUi(yourCustomWebUI)
                       .ExecuteAsync();
     ```

##### <a name="examples-of-implementation-of-icustomwebui-in-test-automation---seleniumwebui"></a>在测试自动化-SeleniumWebUI ICustomWebUi 的实现的示例

MSAL.NET 团队都重新编写，我们的 UI 测试来利用此扩展性机制。 在您感兴趣的情况下可以有一看[SeleniumWebUI](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/tests/Microsoft.Identity.Test.Integration/Infrastructure/SeleniumWebUI.cs#L15-L160) MSAL.NET 源代码中的类

#### <a name="other-optional-parameters"></a>其他可选参数

了解有关所有其他可选参数的详细信息`AcquireTokenInteractive`的参考文档从[AcquireTokenInteractiveParameterBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.apiconfig.acquiretokeninteractiveparameterbuilder?view=azure-dotnet-preview#methods)

## <a name="integrated-windows-authentication"></a>集成的 Windows 身份验证

如果你想要域用户所登录的域或 Azure AD 加入的计算机，需要使用：

```csharp
AcquireTokenByIntegratedWindowsAuth(IEnumerable<string> scopes)
```

### <a name="constraints"></a>约束

- AcquireTokenByIntegratedWindowsAuth (IWA) 才可用于**联合**仅限用户，即，用户在 Active Directory 中创建，由 Azure Active Directory 提供支持。 直接在 AAD 中，而无需 AD 后备-中创建用户**托管**用户-不能使用此身份验证流。 此限制不会影响用户名/密码流。
- IWA 是.NET Framework、.NET Core 和 UWP 平台编写的应用
- IWA 未跳过 MFA （多重身份验证）。 如果配置了 MFA，IWA 可能会失败如果 MFA 质询是必需的因为 MFA 需要用户交互。
  > [!NOTE]
  > 这个是比较棘手。 IWA 是非交互式的但 2FA 需要用户交互功能。 您不控制当标识提供者请求使用 2FA 来执行时，租户管理员。 我们观察值，从 2FA 时需要时通过 VPN 未连接到公司网络，并通过 VPN 连接时，有时甚至从不同国家/地区，登录。 不要指望具有确定性的一组规则，Azure Active Directory 使用 AI 来不断了解是否需要使用 2FA。 如果 IWA 失败，您应回退到用户提示 （交互式身份验证或设备代码流）。

- 颁发机构传入`PublicClientApplicationBuilder`必须是：
  - 租户 ed (窗体`https://login.microsoftonline.com/{tenant}/`其中`tenant`是 guid 表示的租户 ID 或与租户关联的域。
  - 对于任何工作和学校帐户 (`https://login.microsoftonline.com/organizations/`)

  > 不支持 Microsoft 个人帐户 （不能使用 /common 或 /consumers 租户）

- 因为集成 Windows 身份验证是无提示的流程：
  - 应用程序的用户必须具有此前已表示同意使用应用程序
  - 或租户管理员必须具有以前同意使用应用程序的租户中的所有用户。
  - 换句话说：
    - 或者您作为开发人员已按下**授予**自己，在 Azure 门户上的按钮
    - 或租户管理员已按下**Grant/revoke {租户域} 的管理员同意**按钮**API 权限**选项卡上的应用程序的注册 (请参阅[将权限添加到访问 web Api](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis#add-permissions-to-access-web-apis))
    - 或您提供便于用户同意该应用程序 (请参阅[请求单个用户同意](https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#requesting-individual-user-consent))
    - 或提供了一种方法为租户管理员同意使用应用程序 (请参阅[管理员同意](https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#requesting-consent-for-an-entire-tenant))

- 此流可用于.net 桌面、.net core 和 Windows 通用 (UWP) 应用。 .NET core 上只获取用户名的重载均可用，如.NET Core 平台不能请求到 OS 的用户名。
  
有关许可的详细信息，请参阅[v2.0 权限和许可](https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent)

### <a name="how-to-use-it"></a>如何使用

您通常只需要一个参数 (`scopes`)。 但是根据您的 Windows 管理员有的策略，安装程序的方式，将很可能在 windows 计算机上的应用程序不允许以查找已登录的用户。 在这种情况下，使用第二种方法`.WithUsername()`，然后将传入的登录用户为 UPN 格式的用户名`joe@contoso.com`。

下面的示例将最新的情况下，提供各类异常可以获取和及其缓解措施的说明

```CSharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app = PublicClientApplicationBuilder
      .Create(clientId)
      .WithAuthority(authority)
      .Build();

 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
      .ExecuteAsync();
 }
 else
 {
  try
  {
   result = await app.AcquireTokenByIntegratedWindowsAuth(scopes)
      .ExecuteAsync(CancellationToken.None);
  }
  catch (MsalUiRequiredException ex)
  {
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'.Send an interactive authorization request for this user and resource.

   // you need to get user consent first. This can be done, if you are not using .NET Core (which does not have any Web UI)
   // by doing (once only) an AcquireToken interactive.

   // If you are using .NET core or don't want to do an AcquireTokenInteractive, you might want to suggest the user to navigate
   // to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read

   // AADSTS50079: The user is required to use multi-factor authentication.
   // There is no mitigation - if MFA is configured for your tenant and AAD decides to enforce it,
   // you need to fallback to an interactive flows such as AcquireTokenInteractive or AcquireTokenByDeviceCode
   }
   catch (MsalServiceException ex)
   {
    // Kind of errors you could have (in ex.Message)

    // MsalServiceException: AADSTS90010: The grant type is not supported over the /common or /consumers endpoints. Please use the /organizations or tenant-specific endpoint.
    // you used common.
    // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted or otherwise organizations

    // MsalServiceException: AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
    // Explanation: this can happen if your application was not registered as a public client application in Azure AD
    // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   }
   catch (MsalClientException ex)
   {
      // Error Code: unknown_user Message: Could not identify logged in user
      // Explanation: the library was unable to query the current Windows logged-in user or this user is not AD or AAD
      // joined (work-place joined users are not supported).

      // Mitigation 1: on UWP, check that the application has the following capabilities: Enterprise Authentication,
      // Private Networks (Client and Server), User Account Information

      // Mitigation 2: Implement your own logic to fetch the username (e.g. john@contoso.com) and use the
      // AcquireTokenByIntegratedWindowsAuth form that takes in the username

      // Error Code: integrated_windows_auth_not_supported_managed_user
      // Explanation: This method relies on an a protocol exposed by Active Directory (AD). If a user was created in Azure
      // Active Directory without AD backing ("managed" user), this method will fail. Users created in AD and backed by
      // AAD ("federated" users) can benefit from this non-interactive method of authentication.
      // Mitigation: Use interactive authentication
   }
 }


 Console.WriteLine(result.Account.Username);
}
```

可能的修饰符 AcquireTokenByIntegratedWindowsAuthentication 的列表，请参阅[AcquireTokenByIntegratedWindowsAuthParameterBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.apiconfig.acquiretokenbyintegratedwindowsauthparameterbuilder?view=azure-dotnet-preview#methods)

## <a name="username--password"></a>用户名/密码

此外可以通过提供用户名和密码获取令牌。 此流是有限的不推荐，但有仍使用必要的情况。

### <a name="this-flow-isnt-recommended"></a>不建议使用此流

此流**不建议这样做**因为询问用户其密码的应用程序并不安全。 有关此问题的详细信息，请参阅[这篇文章](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/)。 用于获取令牌以无提示方式在 Windows 已加入域的计算机上的首选流程[集成 Windows 身份验证](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Integrated-Windows-Authentication)。 否则还可以使用[设备代码流](https://aka.ms/msal-net-device-code-flow)

> 虽然这是在某些情况下 （DevOps 方案） 很有用，如果你想要在其中提供你 onw UI 在交互式方案中使用用户名/密码，您应认真考虑如何将从它移开。 通过使用用户名/密码，是为提供向上执行大量操作：

> - 核心的现代的标识租户： 获取应用密码，重播。 因为我们有这一概念会被截取的共享机密。
> 这是与无密码不兼容。
> - 需要执行 MFA 的用户将无法登录 （因为无需交互）
> - 用户将无法进行单一登录

### <a name="constraints"></a>约束

以下约束也适用：

- 用户名/密码流并不符合条件性访问和多重身份验证：因此，如果您的应用程序运行在 Azure AD 租户中的租户管理员需要多重身份验证，不能使用此流。 许多组织执行该操作。
- 它只适用工作和学校帐户 (而不是 MSA)
- 该流位于.net 桌面和.net core，但不是在 UWP

### <a name="b2c-specifics"></a>B2C 详细信息

[详细信息与 B2C 配合使用 ROPC](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics#resource-owner-password-credentials-ropc-with-b2c)。

### <a name="how-to-use-it"></a>如何使用它？

`IPublicClientApplication`包含方法 `AcquireTokenByUsernamePassword`

以下示例提供简化的情况

```CSharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuild.Create(clientId)
       .WithAuthority(authority)
       .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password
    securePassword.AppendChar(c);  // keystroke by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                     securePassword)
                      .ExecuteAsync();
  }
  catch(MsalException)
  {
   // See details below
  }
 }
 Console.WriteLine(result.Account.Username);
}
```

下面的示例将最新的情况下，提供各类异常可以获取和及其缓解措施的说明

```CSharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuild.Create(clientId)
                                   .WithAuthority(authority)
                                   .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password keystroke
    securePassword.AppendChar(c);  // by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                    securePassword)
                    .ExecuteAsync();
  }
  catch (MsalUiRequiredException ex) when (ex.Message.Contains("AADSTS65001"))
  {
   // Here are the kind of error messages you could have, and possible mitigations

   // ------------------------------------------------------------------------
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource.

   // Mitigation: you need to get user consent first. This can be done either statically (through the portal),
   /// or dynamically (but this requires an interaction with Azure AD, which is not possible with
   // the username/password flow)
   // Statically: in the portal by doing the following in the "API permissions" tab of the application registration:
   // 1. Click "Add a permission" and add all the delegated permissions corresponding to the scopes you want (for instance
   // User.Read and User.ReadBasic.All)
   // 2. Click "Grant/revoke admin consent for <tenant>") and click "yes".
   // Dynamically, if you are not using .NET Core (which does not have any Web UI) by
   // calling (once only) AcquireTokenInteractive.
   // remember that Username/password is for public client applications that is desktop/mobile applications.
   // If you are using .NET core or don't want to call AcquireTokenInteractive, you might want to:
   // - use device code flow (See https://aka.ms/msal-net-device-code-flow)
   // - or suggest the user to navigate to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ErrorCode: invalid_grant
   // SubError: basic_action
   // MsalUiRequiredException: AADSTS50079: The user is required to use multi-factor authentication.
   // The tenant admin for your organization has chosen to oblige users to perform multi-factor authentication.
   // Mitigation: none for this flow
   // Your application cannot use the Username/Password grant.
   // Like in the previous case, you might want to use an interactive flow (AcquireTokenInteractive()),
   // or Device Code Flow instead.
   // Note this is one of the reason why using username/password is not recommended;
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // Message = "AADSTS70002: Error validating credentials.
   // AADSTS50126: Invalid username or password
   // In the case of a managed user (user from an Azure AD tenant opposed to a
   // federated user, which would be owned
   // in another IdP through ADFS), the user has entered the wrong password
   // Mitigation: ask the user to re-enter the password
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // MsalServiceException: ADSTS50034: To sign into this application the account must be added to
   // the {domainName} directory.
   // or The user account does not exist in the {domainName} directory. To sign into this application,
   // the account must be added to the directory.
   // The user was not found in the directory
   // Explanation: wrong username
   // Mitigation: ask the user to re-enter the username.
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_request")
  {
   // ------------------------------------------------------------------------
   // AADSTS90010: The grant type is not supported over the /common or /consumers endpoints.
   // Please use the /organizations or tenant-specific endpoint.
   // you used common.
   // Mitigation: as explained in the message from Azure AD, the authority you use in the application needs
   // to be tenanted or otherwise "organizations". change the
   // "Tenant": property in the appsettings.json to be a GUID (tenant Id), or domain name (contoso.com)
   // if such a domain is registered with your tenant
   // or "organizations", if you want this application to sign-in users in any Work and School accounts.
   // ------------------------------------------------------------------------

  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "unauthorized_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS700016: Application with identifier '{clientId}' was not found in the directory '{domain}'.
   // This can happen if the application has not been installed by the administrator of the tenant or consented
   // to by any user in the tenant.
   // You may have sent your authentication request to the wrong tenant
   // Cause: The clientId in the appsettings.json might be wrong
   // Mitigation: check the clientId and the app registration
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
   // Explanation: this can happen if your application was not registered as a public client application in Azure AD
   // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException)
  {
   throw;
  }

  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user_type")
  {
   // Message = "Unsupported User Type 'Unknown'. Please see https://aka.ms/msal-net-up"
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "user_realm_discovery_failed")
  {
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user. That's for instance the case
   // if you use a phone number
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user")
  {
   // the username was probably empty
   // ex.Message = "Could not identify the user logged into the OS. See https://aka.ms/msal-net-iwa for details."
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "parsing_wstrust_response_failed")
  {
   // ------------------------------------------------------------------------
   // In the case of a Federated user (that is owned by a federated IdP, as opposed to a managed user owned in an Azure AD tenant)
   // ID3242: The security token could not be authenticated or authorized.
   // The user does not exist or has entered the wrong password
   // ------------------------------------------------------------------------
  }
 }

 Console.WriteLine(result.Account.Username);
}
```

有关详细信息，可应用于所有修饰符`AcquireTokenByUsernamePassword`，请参阅[AcquireTokenByUsernamePasswordParameterBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.apiconfig.acquiretokenbyusernamepasswordparameterbuilder?view=azure-dotnet-preview#methods)

## <a name="command-line-tool-without-web-browser"></a>命令行工具 （而无需 web 浏览器）

### <a name="device-code-flow-why-and-how"></a>设备代码流原因？以及如何？

如果您要编写一个命令行工具 （即没有 Web 控件），并且不能或不想要使用的上一个流，你将需要使用`AcquireTokenWithDeviceCode`。

使用 Azure AD 的交互式身份验证需要 web 浏览器 (有关详细信息，请参阅[web 浏览器的使用情况](https://aka.ms/msal-net-uses-web-browser))。 但是，若要对设备或操作系统中不提供 Web 浏览器上的用户进行身份验证，设备代码流使用户能够使用另一台设备 （例如另一台计算机或移动电话） 进行签名以交互方式。 通过使用设备代码流，应用程序获取令牌通过专门设计用于这些设备/OS 一个两步过程。 此类应用程序的示例包括 iOT 或命令行工具 (CLI) 上运行的应用程序。 思路是：

1. 每当用户身份验证是必需的应用程序提供的代码并要求用户使用其他设备 （如连接到 internet 的智能手机） 来导航到的 URL (例如， `https://microsoft.com/devicelogin`)，将提示用户输入的代码。 完成后，web 页面将引导用户完成了正常的身份验证体验，包括许可提示和多重身份验证，如有必要。

2. 身份验证成功后的命令行应用将收到通过返回通道所需的标记并将其用于执行其所需的 web API 调用。

### <a name="code"></a>代码

`IPublicClientApplication`包含一个名为方法 `AcquireTokenWithDeviceCode`

```CSharp
 AcquireTokenWithDeviceCode(IEnumerable<string> scopes,
                            Func<DeviceCodeResult, Task> deviceCodeResultCallback)
```

此方法将作为参数：

- `scopes`请求的访问令牌
- 将收到一个回调 `DeviceCodeResult`

  ![image](https://user-images.githubusercontent.com/13203188/56024968-7af1b980-5d11-11e9-84c2-5be2ef306dc5.png)

下面的示例代码显示的最新的情况下，可以获取的异常和及其缓解方法的类型的说明。

```CSharp
static async Task<AuthenticationResult> GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication pca = PublicClientApplicationBuilder
      .Create(clientId)
      .WithAuthority(authority)
      .Build();

 AuthenticationResult result = null;
 var accounts = await app.GetAccountsAsync();

 // All AcquireToken* methods store the tokens in the cache, so check the cache first
 try
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
       .ExecuteAsync();
 }
 catch (MsalUiRequiredException ex)
 {
  // A MsalUiRequiredException happened on AcquireTokenSilent.
  // This indicates you need to call AcquireTokenInteractive to acquire a token
  System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");
 }

 try
 {
  result = await app.AcquireTokenWithDeviceCode(scopes,
      deviceCodeCallback =>
  {
       // This will print the message on the console which tells the user where to go sign-in using
       // a separate browser and the code to enter once they sign in.
       // The AcquireTokenWithDeviceCode() method will poll the server after firing this
       // device code callback to look for the successful login of the user via that browser.
       // This background polling (whose interval and timeout data is also provided as fields in the
       // deviceCodeCallback class) will occur until:
       // * The user has successfully logged in via browser and entered the proper code
       // * The timeout specified by the server for the lifetime of this code (typically ~15 minutes) has been reached
       // * The developing application calls the Cancel() method on a CancellationToken sent into the method.
       //   If this occurs, an OperationCanceledException will be thrown (see catch below for more details).
       Console.WriteLine(deviceCodeResult.Message);
       return Task.FromResult(0);
  }).ExecuteAsync();

  Console.WriteLine(result.Account.Username);
  return result;
 }
 catch (MsalServiceException ex)
 {
  // Kind of errors you could have (in ex.Message)

  // AADSTS50059: No tenant-identifying information found in either the request or implied by any provided credentials.
  // Mitigation: as explained in the message from Azure AD, the authoriy needs to be tenanted. you have probably created
  // your public client application with the following authorities:
  // https://login.microsoftonline.com/common or https://login.microsoftonline.com/organizations

  // AADSTS90133: Device Code flow is not supported under /common or /consumers endpoint.
  // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted

  // AADSTS90002: Tenant <tenantId or domain you used in the authority> not found. This may happen if there are
  // no active subscriptions for the tenant. Check with your subscription administrator.
  // Mitigation: if you have an active subscription for the tenant this might be that you have a typo in the
  // tenantId (GUID) or tenant domain name.
 }
 catch (OperationCanceledException ex)
 {
  // If you use a CancellationToken, and call the Cancel() method on it, then this may be triggered
  // to indicate that the operation was cancelled.
  // See https://docs.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads
  // for more detailed information on how C# supports cancellation in managed threads.
 }
 catch (MsalClientException ex)
 {
  // Verification code expired before contacting the server
  // This exception will occur if the user does not manage to sign-in before a time out (15 mins) and the
  // call to `AcquireTokenWithDeviceCode` is not cancelled in between
 }
}
```

## <a name="file-based-token-cache"></a>基于文件的令牌缓存

在 MSAL.NET 中，默认会提供内存中令牌缓存。

### <a name="serialization-is-customizable-in-windows-desktop-apps-and-web-appsweb-apis"></a>序列化是 Windows 桌面应用程序和 web 应用/web Api 中可自定义

对于.NET Framework 和.NET core，如果您不执行任何额外，内存中令牌缓存持续时间的应用程序的持续时间。 若要了解为什么要序列化未提供默认情况下，请记住 MSAL.NET 桌面/核心应用程序可以是控制台或 Windows 应用程序 （它们将具有访问文件系统），**还要**Web 应用程序或 web API。 这些 Web 应用和 web Api 可能使用一些特定的缓存机制，如数据库、 分布式缓存的 redis 缓存，依此类推。 若要让.NET 桌面或核心中的永久性令牌缓存应用程序，你将需要自定义序列化。

类和接口中令牌缓存序列化涉及是以下类型：

- ``ITokenCache``用于定义事件以订阅令牌缓存的序列化请求，以及要序列化或反序列化处各种格式的缓存的方法 (ADAL v3.0，MSAL 2.x 和 MSAL 3.x = ADAL 5.0 版)
- ``TokenCacheCallback`` 是传递给事件的回调，可让你处理序列化。 将类型的参数调用它们``TokenCacheNotificationArgs``。
- ``TokenCacheNotificationArgs`` 仅提供``ClientId``的应用程序和对用户为其标记为可用的引用

  ![image](https://user-images.githubusercontent.com/13203188/56027172-d58d1480-5d15-11e9-8ada-c0292f1800b3.png)

> [!IMPORTANT]
> MSAL.NET 将为你创建令牌缓存，当你调用应用程序的 `GetUserTokenCache` 和 `GetAppTokenCache` 方法时，它会提供 `IToken` 缓存。 您不应该自行实现接口。 实现自定义令牌缓存序列化时，你的责任是：
>
> - 对 `BeforeAccess` 和 `AfterAccess`“事件”做出反应。 `BeforeAccess`委托是负责反序列化缓存，而`AfterAccess`一种类型负责序列化缓存。
> - 其中的一部分事件存储或加载 Blob，这些 Blob 将通过事件参数传递到所需的任何存储。

如果您要编写一个公共客户端应用程序 （桌面版），或机密客户端应用程序 (web 应用/web API，守护程序应用) 的令牌缓存序列化具体取决于不同的策略包括。

MSAL V2.x 以来有多个选项，具体取决于你想要序列化仅为 MSAL.NET 格式 （统一的格式高速缓存，通常是使用 MSAL，而且还跨平台），缓存是否还想要支持[旧](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization)令牌缓存的 ADAL V3 的序列化。

自定义令牌缓存序列化以 SSO 之间共享状态 ADAL.NET 3.x，ADAL.NET 5.x 和 MSAL.NET 介绍部分的下面的示例： [active-directory-dotnet-v1-to-v2](https://github.com/Azure-Samples/active-directory-dotnet-v1-to-v2)

### <a name="simple-token-cache-serialization-msal-only"></a>简单令牌缓存序列化（仅限 MSAL）

下面是适用于桌面应用程序的令牌缓存的自定义序列化的简单实现示例。 此处与应用程序相同的文件夹中的文件中的用户令牌缓存。

通过调用生成应用程序后，启用序列化``TokenCacheHelper.EnableSerialization()``传递应用程序 `UserTokenCache`

```CSharp
app = PublicClientApplicationBuilder.Create(ClientId)
    .Build();
TokenCacheHelper.EnableSerialization(app.UserTokenCache);
```

此帮助器类如下面的代码段所示：

```CSharp
static class TokenCacheHelper
 {
  public static void EnableSerialization(ITokenCache tokenCache)
  {
   tokenCache.SetBeforeAccess(BeforeAccessNotification);
   tokenCache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// Path to the token cache
  /// </summary>
  public static readonly string CacheFilePath = System.Reflection.Assembly.GetExecutingAssembly().Location + ".msalcache.bin3";

  private static readonly object FileLock = new object();


  private static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeMsalV3(File.Exists(CacheFilePath)
            ? ProtectedData.Unprotect(File.ReadAllBytes(CacheFilePath),
                                      null,
                                      DataProtectionScope.CurrentUser)
            : null);
   }
  }

  private static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     // reflect changesgs in the persistent store
     File.WriteAllBytes(CacheFilePath,
                         ProtectedData.Protect(args.TokenCache.SerializeMsalV3(),
                                                 null,
                                                 DataProtectionScope.CurrentUser)
                         );
    }
   }
  }
 }
```

基于文件的序列化程序 （适用于 Windows、 Mac 和 linux 上运行的桌面应用程序） 的公共客户端应用程序是从可用的产品质量令牌缓存的预览[Microsoft.Identity.Client.Extensions.Msal](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Msal)开放源代码库。 可以通过以下 Nuget 包将此程序包含在应用程序中：[Microsoft.Identity.Client.Extensions.Msal](https://www.nuget.org/packages/Microsoft.Identity.Client.Extensions.Msal/)。

> 免责声明。 Microsoft.Identity.Client.Extensions.Msal 库是通过 MSAL.NET 扩展。 这些库中的类可能会全面 MSAL.NET 以后，按原样或进行重大更改。

### <a name="dual-token-cache-serialization-msal-unified-cache--adal-v3"></a>双令牌缓存序列化 （统一的 MSAL 缓存 + ADAL V3）

如果你想要实现令牌缓存序列化都具有统一缓存格式 (共有 ADAL.NET 4.x 和 MSAL.NET 2.x 中，并与其他 MSALs 同代的或更低版本，在同一平台上)，您可以从获得灵感，下面的代码:

```CSharp
string appLocation = Path.GetDirectoryName(Assembly.GetEntryAssembly().Location;
string cacheFolder = Path.GetFullPath(appLocation) + @"..\..\..\..");
string adalV3cacheFileName = Path.Combine(cacheFolder, "cacheAdalV3.bin");
string unifiedCacheFileName = Path.Combine(cacheFolder, "unifiedCache.bin");

IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
                                    .Build();
FilesBasedTokenCacheHelper.EnableSerialization(app.UserTokenCache,
                                               unifiedCacheFileName,
                                               adalV3cacheFileName);

```

这一次如以下代码所示的帮助器类：

```CSharp
using System;
using System.IO;
using System.Security.Cryptography;
using Microsoft.Identity.Client;

namespace CommonCacheMsalV3
{
 /// <summary>
 /// Simple persistent cache implementation of the dual cache serialization (ADAL V3 legacy
 /// and unified cache format) for a desktop applications (from MSAL 2.x)
 /// </summary>
 static class FilesBasedTokenCacheHelper
 {
  /// <summary>
  /// Get the user token cache
  /// </summary>
  /// <param name="adalV3CacheFileName">File name where the cache is serialized with the
  /// ADAL V3 token cache format. Can
  /// be <c>null</c> if you don't want to implement the legacy ADAL V3 token cache
  /// serialization in your MSAL 2.x+ application</param>
  /// <param name="unifiedCacheFileName">File name where the cache is serialized
  /// with the Unified cache format, common to
  /// ADAL V4 and MSAL V2 and above, and also across ADAL/MSAL on the same platform.
  ///  Should not be <c>null</c></param>
  /// <returns></returns>
  public static void EnableSerialization(ITokenCache cache, string unifiedCacheFileName, string adalV3CacheFileName)
  {
   usertokenCache = cache;
   UnifiedCacheFileName = unifiedCacheFileName;
   AdalV3CacheFileName = adalV3CacheFileName;

   usertokenCache.SetBeforeAccess(BeforeAccessNotification);
   usertokenCache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// Token cache
  /// </summary>
  static ITokenCache usertokenCache;

  /// <summary>
  /// File path where the token cache is serialized with the unified cache format
  /// (ADAL.NET V4, MSAL.NET V3)
  /// </summary>
  public static string UnifiedCacheFileName { get; private set; }

  /// <summary>
  /// File path where the token cache is serialized with the legacy ADAL V3 format
  /// </summary>
  public static string AdalV3CacheFileName { get; private set; }

  private static readonly object FileLock = new object();

  public static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeAdalV3(ReadFromFileIfExists(AdalV3CacheFileName));
    try
    {
     args.TokenCache.DeserializeMsalV3(ReadFromFileIfExists(UnifiedCacheFileName));
    }
    catch(Exception ex)
    {
     // Compatibility with the MSAL v2 cache if you used one
     args.TokenCache.DeserializeMsalV2(ReadFromFileIfExists(UnifiedCacheFileName));
    }
   }
  }

  public static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     WriteToFileIfNotNull(UnifiedCacheFileName, args.TokenCache.SerializeMsalV3());
     if (!string.IsNullOrWhiteSpace(AdalV3CacheFileName))
     {
      WriteToFileIfNotNull(AdalV3CacheFileName, args.TokenCache.SerializeAdalV3());
     }
    }
   }
  }

  /// <summary>
  /// Read the content of a file if it exists
  /// </summary>
  /// <param name="path">File path</param>
  /// <returns>Content of the file (in bytes)</returns>
  private static byte[] ReadFromFileIfExists(string path)
  {
   byte[] protectedBytes = (!string.IsNullOrEmpty(path) && File.Exists(path))
       ? File.ReadAllBytes(path) : null;
   byte[] unprotectedBytes = encrypt ?
       ((protectedBytes != null) ? ProtectedData.Unprotect(protectedBytes, null, DataProtectionScope.CurrentUser) : null)
       : protectedBytes;
   return unprotectedBytes;
  }

  /// <summary>
  /// Writes a blob of bytes to a file. If the blob is <c>null</c>, deletes the file
  /// </summary>
  /// <param name="path">path to the file to write</param>
  /// <param name="blob">Blob of bytes to write</param>
  private static void WriteToFileIfNotNull(string path, byte[] blob)
  {
   if (blob != null)
   {
    byte[] protectedBytes = encrypt
      ? ProtectedData.Protect(blob, null, DataProtectionScope.CurrentUser)
      : blob;
    File.WriteAllBytes(path, protectedBytes);
   }
   else
   {
    File.Delete(path);
   }
  }

  // Change if you want to test with an un-encrypted blob (this is a json format)
  private static bool encrypt = true;
 }
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [从桌面应用调用 web API](scenario-desktop-call-api.md)
