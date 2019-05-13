---
title: Microsoft 标识平台 UWP 入门 | Azure
description: 通用 Windows 平台应用程序 (UWP) 如何通过 Microsoft 标识平台终结点调用需要访问令牌的 API。
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/11/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 448858efeaae4c3e2a41d41181e9ec74b03223f6
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138254"
---
# <a name="call-microsoft-graph-api-from-a-universal-windows-platform-application-xaml"></a>从通用 Windows 平台应用程序 (XAML) 调用 Microsoft 图形 API

> [!div renderon="docs"]
> [!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

本指南介绍本机通用 Windows 平台 (UWP) 应用程序如何请求访问令牌，然后调用 Microsoft 图形 API。 本指南也适用于其他需要从 Microsoft 标识平台终结点请求访问令牌的 API。

在本指南结束时，应用程序将使用个人帐户调用受保护的 API。 示例包括 outlook.com、live.com 等等。 应用程序还将调用任何使用 Azure Active Directory (Azure AD) 的公司或组织提供的工作和学校帐户。

>[!NOTE]
> 本指南需要安装了通用 Windows 平台开发的 Visual Studio 2017。 请参阅[设置](https://docs.microsoft.com/windows/uwp/get-started/get-set-up)，获取有关如何下载和配置 Visual Studio 以开发通用 Windows 平台应用的说明。

## <a name="how-this-guide-works"></a>本指南的工作原理

![显示本教程生成的示例应用的工作原理](./media/tutorial-v2-windows-uwp/uwp-intro.svg)

本指南创建的示例 UWP 应用程序查询从 Microsoft 标识平台终结点接受令牌的 Microsoft 图形 API 或 Web API。 在此方案中，通过 Authorization 标头向 HTTP 请求添加了令牌。 Microsoft 身份验证库 (MSAL) 处理令牌获取和续订。

## <a name="nuget-packages"></a>NuGet 包

本指南使用以下 NuGet 包：

|库|说明|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Microsoft 身份验证库|

## <a name="set-up-your-project"></a>设置项目

本部分逐步说明如何将 Windows 桌面 .NET 应用程序 (XAML) 与“登录 Microsoft”集成。 然后，该应用程序可以查询需要令牌的 Web API（例如 Microsoft 图形 API）。

本指南创建的应用程序显示用来查询图形 API 的按钮、注销按钮和显示调用结果的文本框。

> [!NOTE]
> 想要改为下载此示例的 Visual Studio 项目？ [下载项目](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip)并跳到[应用程序注册](#register-your-application "应用程序注册步骤")步骤，在运行代码示例前对其进行配置。

### <a name="create-your-application"></a>创建应用程序

1. 在 Visual Studio 中，选择“文件” > “新建” > “项目”。
2. 在“模板”下，选择“Visual C#”。
3. 选择“空白应用(通用 Windows)”。
4. 为应用命名，然后选择“确定”。
5. 如果出现提示，请选择任意版本作为“目标”版本和“最低”版本，然后选择“确定”。

    >![最低版本和目标版本](./media/tutorial-v2-windows-uwp/vs-minimum-target.png)

## <a name="add-microsoft-authentication-library-to-your-project"></a>向项目添加 Microsoft 身份验证库
1. 在 Visual Studio 中，选择“工具” > “NuGet 包管理器” > “包管理器控制台”。
2. 在“包管理器控制台”窗口中复制并粘贴以下命令：

    ```powershell
    Install-Package Microsoft.Identity.Client -IncludePrerelease
    ```

> [!NOTE]
> 此命令将安装 [Microsoft 身份验证库](https://aka.ms/msal-net)。 MSAL 获取、缓存和刷新用于访问受 Microsoft 标识平台保护的 API 的用户令牌。

## <a name="create-your-applications-ui"></a>创建应用程序 UI

系统会自动创建 **MainPage.xaml** 文件作为项目模板的一部分。 打开此文件，并遵照说明操作：

* 将应用程序的 **Grid** 节点替换为以下代码：

    ```xml
    <Grid>
        <StackPanel Background="Azure">
            <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
                <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
                <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
            </StackPanel>
            <TextBlock Text="API Call Results" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
            <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
            <TextBlock Text="Token Info" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
            <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
        </StackPanel>
    </Grid>
    ```
    
## <a name="use-msal-to-get-a-token-for-microsoft-graph-api"></a>使用 MSAL 获取 Microsoft 图形 API 的令牌

本部分介绍如何使用 MSAL 获取 Microsoft 图形 API 的令牌。

1.  在 **MainPage.xaml.cs** 中，将 MSAL 的引用添加到该类：

    ```csharp
    using Microsoft.Identity.Client;
    ```

2. 将 <code>MainPage</code> 类的代码替换为以下代码：

    ```csharp
    public sealed partial class MainPage : Page
    {
        //Set the API Endpoint to Graph 'me' endpoint
        string graphAPIEndpoint = "https://graph.microsoft.com/v1.0/me";

        //Set the scope for API call to user.read
        string[] scopes = new string[] { "user.read" };

        // Below are the clientId (Application Id) of your app registration and the tenant information. 
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        // - Te content of Tenant by the information about the accounts allowed to sign-in in your application:
        //   - For Work or School account in your org, use your tenant ID, or domain
        //   - for any Work or School accounts, use organizations
        //   - for any Work or School accounts, or Microsoft personal account, use common
        //   - for Microsoft Personal account, use consumers
        private const string ClientId = "0b8b0665-bc13-4fdc-bd72-e0227b9fc011";        

        public IPublicClientApplication PublicClientApp { get; } 

        public MainPage()
        {
          this.InitializeComponent();

          PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AadAuthorityAudience.AzureAdAndPersonalMicrosoftAccount)
                .WithLogging((level, message, containsPii) =>
                {
                    Debug.WriteLine($"MSAL: {level} {message} ");
                }, LogLevel.Warning, enablePiiLogging:false,enableDefaultPlatformLogging:true)
                .WithUseCorporateNetwork(true)
                .Build();
        }

        /// <summary>
        /// Call AcquireTokenAsync - to acquire a token requiring user to sign-in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
         AuthenticationResult authResult = null;
         ResultText.Text = string.Empty;
         TokenInfoText.Text = string.Empty;

         // It's good practice to not do work on the UI thread, so use ConfigureAwait(false) whenever possible.            
         IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false); 
         IAccount firstAccount = accounts.FirstOrDefault();

         try
         {
          authResult = await PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                                  .ExecuteAsync();
         }
         catch (MsalUiRequiredException ex)
         {
          // A MsalUiRequiredException happened on AcquireTokenSilent.
          // This indicates you need to call AcquireTokenInteractive to acquire a token
          System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

          try
          {
           authResult = await PublicClientApp.AcquireTokenInteractive(scopes)
                                                      .ExecuteAsync()
                                                      .ConfigureAwait(false);
           }
           catch (MsalException msalex)
           {
            await DisplayMessageAsync($"Error Acquiring Token:{System.Environment.NewLine}{msalex}");
           }
          }
          catch (Exception ex)
          {
           await DisplayMessageAsync($"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}");
           return;
          }

          if (authResult != null)
          {
           var content = await GetHttpContentWithToken(graphAPIEndpoint,
                                                       authResult.AccessToken).ConfigureAwait(false);

           // Go back to the UI thread to make changes to the UI
           await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
           {
            ResultText.Text = content;
            DisplayBasicTokenInfo(authResult);
            this.SignOutButton.Visibility = Visibility.Visible;
           });
          }
        }
    ```

### <a name="more-information"></a>详细信息

#### <a name="get-a-user-token-interactively"></a>以交互方式获取用户令牌

调用 `AcquireTokenInteractive` 方法将出现提示用户进行登录的窗口。 当用户首次需要访问受保护的资源时，应用程序通常会要求用户进行交互式登录。 当用于获取令牌的无提示操作失败时，用户也可能需要登录。 例如，当用户的密码过期时。

#### <a name="get-a-user-token-silently"></a>以无提示方式获取用户令牌

`AcquireTokenSilent` 方法处理令牌获取和续订，无需进行任何用户交互。 首次执行 `AcquireTokenInteractive` 并提示用户输入凭据后，应使用 `AcquireTokenSilent` 方法请求后续调用的令牌，因为此方法以无提示方式获取令牌。 MSAL 将处理令牌缓存和续订。

最终，`AcquireTokenSilent` 方法会失败。 失败可能是因为用户已注销，或者在另一设备上更改了密码。 MSAL 检测到可以通过请求交互式操作解决问题时，它将引发 `MsalUiRequiredException` 异常。 应用程序可以通过两种方式处理此异常：

* 它可以立即调用 `AcquireTokenInteractive`。 此调用会导致系统提示用户进行登录。 此模式通常用于联机应用程序，这类应用程序中没有可供用户使用的脱机内容。 此引导式设置生成的示例遵循此模式。 首次运行示例时可以看到其正在运行。
  * 由于没有用户使用过该应用程序，因此 `accounts.FirstOrDefault()` 包含一个 null 值，并且引发 `MsalUiRequiredException` 异常。
  * 此示例中的代码随后通过调用 `AcquireTokenInteractive` 来处理此异常。 此调用会导致系统提示用户进行登录。

* 或者，它会向用户呈现视觉指示，要求用户进行交互式登录。 然后，用户可以选择适当的时间进行登录。 也可以让应用程序稍后重试 `AcquireTokenSilent`。 当用户可以使用其他应用程序功能而不导致中断时，会频繁使用此模式。 例如，当脱机内容在应用程序中可用时。 在这种情况下，用户可以决定何时需要登录来访问受保护的资源或刷新过期信息。 或者，可让应用程序决定在网络临时不可用又还原后，是否重试 `AcquireTokenSilent`。

## <a name="call-microsoft-graph-api-by-using-the-token-you-just-obtained"></a>使用刚获得的令牌调用 Microsoft 图形 API

* 将以下新方法添加到 **MainPage.xaml.cs**。 此方法用于通过 `Authorization` 标头对图形 API 执行 `GET` 请求：

    ```csharp
    /// <summary>
    /// Perform an HTTP GET request to a URL using an HTTP Authorization header
    /// </summary>
    /// <param name="url">The URL</param>
    /// <param name="token">The token</param>
    /// <returns>String containing the results of the GET operation</returns>
    public async Task<string> GetHttpContentWithToken(string url, string token)
    {
        var httpClient = new System.Net.Http.HttpClient();
        System.Net.Http.HttpResponseMessage response;
        try
        {
            var request = new System.Net.Http.HttpRequestMessage(System.Net.Http.HttpMethod.Get, url);
            // Add the token in Authorization header
            request.Headers.Authorization = 
              new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
            response = await httpClient.SendAsync(request);
            var content = await response.Content.ReadAsStringAsync();
            return content;
        }
        catch (Exception ex)
        {
            return ex.ToString();
        }
    }
    ```

### <a name="more-information-on-making-a-rest-call-against-a-protected-api"></a>对受保护 API 进行 REST 调用的详细信息

在此示例应用程序中，使用 `GetHttpContentWithToken` 方法对需要令牌的受保护资源发出 HTTP `GET` 请求。 然后，此方法将内容返回给调用方。 此方法在 **HTTP Authorization** 标头中添加获取的令牌。 此示例中的资源是 Microsoft Graph API **me** 终结点，用于显示用户的个人资料信息。
<!--end-collapse-->

## <a name="add-a-method-to-sign-out-the-user"></a>添加方法以注销用户

* 若要注销用户，请将以下方法添加到 **MainPage.xaml.cs**：

    ```csharp
    /// <summary>
    /// Sign out the current user
    /// </summary>
    private async void SignOutButton_Click(object sender, RoutedEventArgs e)
    {
        IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync
                                                              .ConfigureAwait(false);
        IAccount firstAccount = accounts.FirstOrDefault();

        try
        {
            await PublicClientApp.RemoveAsync(firstAccount).ConfigureAwait(false);
            await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
            {
                ResultText.Text = "User has signed-out";
                this.CallGraphButton.Visibility = Visibility.Visible;
                    this.SignOutButton.Visibility = Visibility.Collapsed;
                });
            }
            catch (MsalException ex)
            {
                ResultText.Text = $"Error signing-out user: {ex.Message}";
            }
        }
    ```

> [!NOTE]
> MSAL.NET 使用异步方法来获取令牌或操作帐户，因此你需要负责在 UI 线程中执行 UI-ed 操作（所以有 `Dispatcher.RunAsync`）并执行对调用 `ConfigureAwait(false)` 的预防措施

### <a name="more-information-on-sign-out"></a>有关注销的详细信息

`SignOutButton_Click` 方法从 MSAL 用户缓存中删除用户。 此方法有效告知 MSAL 要忘记当前用户。 然后，仅当获取令牌的未来请求设置为交互式时，该请求才会成功。
此示例中的应用程序支持单个用户。 但是，MSAL 也支持可同时登录多个帐户的方案。 用户在其中具有多个帐户的电子邮件应用程序就是一个示例。

## <a name="display-basic-token-information"></a>显示基本令牌信息

* 将以下方法添加到 **MainPage.xaml.cs**，以显示有关令牌的基本信息：

    ```csharp
    /// <summary>
    /// Display basic information contained in the token. Needs to be called from the UI thead.
    /// </summary>
    private void DisplayBasicTokenInfo(AuthenticationResult authResult)
    {
        TokenInfoText.Text = "";
        if (authResult != null)
        {
            TokenInfoText.Text += $"User Name: {authResult.Account.Username}" + Environment.NewLine;
            TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
        }
    }
    ```

### <a name="more-information"></a>详细信息

通过 OpenID 连接获取的 ID 令牌还包含与用户相关的一小部分信息。 `DisplayBasicTokenInfo` 显示令牌中包含的基本信息。 例如，显示用户的显示名称和 ID、令牌过期日期，以及表示访问令牌本身的字符串。 如果多次选择“调用 Microsoft 图形 API”按钮，将会发现后续请求重复使用了同一令牌。 而且还会发现，在 MSAL 决定续订令牌时，过期日期也延长了。

## <a name="register-your-application"></a>注册应用程序

现在需要在 Microsoft 应用程序注册门户中注册应用程序：

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
1. 如果帐户存在于多个 Azure AD 租户中，请选择页面顶部的菜单右上角的 `Directory + Subscription`，并将门户会话切换为所需的 Azure AD 租户。
1. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)页。
1. 选择“新注册”。
   - 在“名称”部分输入一个会显示给应用用户的有意义的应用程序名称，例如 `UWP-App-calling-MSGraph`。
   - 在“支持的帐户类型”部分，选择“任何组织目录中的帐户和个人 Microsoft 帐户(例如 Skype、Xbox、Outlook.com)”。
   - 选择“注册”以创建应用程序。
1. 在应用的“概述”页上，找到“应用程序(客户端) ID”值，并记下该值以供后续使用。 返回 Visual Studio，打开 MainPage.xaml.cs，并将 ClientId 的值替换为刚刚记下的应用程序 ID：
1. 在应用的页面列表中，选择“身份验证”。
   1. 在“重定向 URI”部分的重定向 URI 列表中：
   1. 在“类型”列中选择“公共客户端(移动和桌面)”。
   1. 在“重定向 URI”列中输入 `urn:ietf:wg:oauth:2.0:oob`。
1. 选择“保存”。
1. 在应用的页面列表中，选择“API 权限”
   - 单击“添加权限”按钮，然后
   - 确保已选中“Microsoft API”的选项卡
   - 在“常用 Microsoft API”部分中，单击“Microsoft Graph”
   - 在“委托的权限”部分中，确保已勾选正确的权限：User.Read。 如有必要，请使用搜索框。
   - 选择“添加权限”按钮

## <a name="enable-integrated-authentication-on-federated-domains-optional"></a>在联盟域中启用集成身份验证（可选）

与 Azure AD 联盟域一起使用时，若要启用 Windows 集成身份验证，应用程序清单必须启用其他功能：

1. 双击“Package.appxmanifest”。
2. 选择“功能”选项卡并确保启用以下设置：

    - 企业身份验证
    - 专用网络(客户端和服务器)
    - 共享用户证书

> [!IMPORTANT]
> 默认情况下，未为此示例配置[集成 Windows 身份验证](https://aka.ms/msal-net-iwa)。 请求“企业身份验证”或“共享用户证书”功能的应用程序需要由 Windows 应用商店进行的更高级别的验证。 此外，并非所有开发人员都希望执行更高级别的验证。 仅当需要使用 Azure AD 联盟域进行 Windows 集成身份验证时，才启用此设置。

## <a name="test-your-code"></a>测试代码

若要测试应用程序，请按 F5 在 Visual Studio 中运行项目。 将显示主窗口：

![应用程序的用户界面](./media/tutorial-v2-windows-uwp/testapp-ui.png)

准备好测试后，选择“调用 Microsoft Graph API”。 然后使用 Azure AD 组织帐户或 Microsoft 帐户（例如 live.com 或 outlook.com）登录。 如果是首次测试，会看到要求用户登录的窗口：

![登录页](./media/tutorial-v2-windows-uwp/sign-in-page.png)

### <a name="consent"></a>同意

首次登录应用程序时，会看到如下所示的许可屏幕。 选择“是”显式许可访问：

![访问许可屏幕](./media/tutorial-v2-windows-uwp/consentscreen.png)

### <a name="expected-results"></a>预期结果

“API 调用结果”屏幕上会显示 Microsoft 图形 API 调用返回的用户个人资料信息：

![“API 调用结果”屏幕](./media/tutorial-v2-windows-uwp/uwp-results-screen.PNG)

“令牌信息”框中还会显示通过 `AcquireTokenInteractive` 或 `AcquireTokenSilent` 获得的令牌的相关基本信息：

|属性  |格式  |说明 |
|---------|---------|---------|
|**用户名** |<span>user@domain.com</span> |用于标识用户的用户名。|
|**令牌到期** |DateTime |令牌的过期时间。 MSAL 根据需要通过续订令牌来延长到期日期。|

### <a name="more-information-about-scopes-and-delegated-permissions"></a>有关作用域和委派权限的详细信息

Microsoft 图形 API 需要 *user.read* 作用域来读取用户的个人资料。 默认情况下，在应用程序注册门户中注册的每个应用程序中，都会自动添加此作用域。 Microsoft Graph 的其他 API 以及后端服务器的自定义 API 可能需要其他作用域。 Microsoft 图形 API 需要 *Calendars.Read* 作用域来列出用户的日历。

若要在应用程序上下文中访问用户的日历，请将 *Calendars.Read* 委派权限添加到应用程序注册信息。 然后，将 *Calendars.Read* 作用域添加到 `acquireTokenSilent` 调用。

> [!NOTE]
> 增加作用域数量时，用户可能会收到接受其他许可的提示。

## <a name="known-issues"></a>已知问题

### <a name="issue-1"></a>问题 1

在 Azure AD 联盟域上登录应用程序时，可能会收到以下错误消息之一：

* 在请求中未找到有效的客户端证书。
* 在用户的证书存储中未找到有效的证书。
* 请重试选择不同的身份验证方法。

原因：未启用企业功能和证书功能。

**解决方案：** 按照[联盟域中的集成身份验证](#enable-integrated-authentication-on-federated-domains-optional)中的步骤操作。

### <a name="issue-2"></a>问题 2

[在联盟域中启用集成身份验证](#enable-integrated-authentication-on-federated-domains-optional)，并尝试在 Windows 10 计算机上使用 Windows Hello 登录到配置了多重身份验证的环境。 此时会显示证书列表。 但如果选择使用 PIN，则永远不会显示 PIN 窗口。

原因：在 Windows 10 桌面版上运行的 UWP 应用程序中的 Web 身份验证代理存在已知限制。 该代理在 Windows 10 手机版上可正常工作。

**解决方法：** 选择“使用其他选项登录”。 然后选择“使用用户名和密码登录”。 选择“提供密码”。 然后完成手机身份验证过程。

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]
