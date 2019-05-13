---
title: 教程 - 从单页应用程序授予对 ASP.NET Core Web API 的访问权限 - Azure Active Directory B2C | Microsoft Docs
description: 有关如何从单页应用使用 Active Directory B2C 保护 .NET Core Web API 并调用该 API 的教程。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.author: davidmu
ms.date: 02/04/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 13fedae2798311a59a5cee2805ce9e09b1bd5a0f
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64724673"
---
# <a name="tutorial-grant-access-to-an-aspnet-core-web-api-from-a-single-page-application-using-azure-active-directory-b2c"></a>教程：从单页应用程序使用 Azure Active Directory B2C 授予对 ASP.NET Core Web API 的访问权限

本教程介绍如何从单页应用程序调用受 Azure Active Directory (Azure AD) B2C 保护的 ASP.NET Core Web API 资源。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 添加 Web API 应用程序
> * 配置 Web API 的范围
> * 授予 Web API 权限
> * 将示例配置为使用此应用程序

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

完成[教程：使单页应用能够使用帐户通过 Azure Active Directory B2C 进行身份验证](active-directory-b2c-tutorials-spa.md)。

## <a name="add-a-web-api-application"></a>添加 Web API 应用程序

Web API 资源需要先在租户中注册，然后才能接受并响应提供访问令牌的客户端应用程序所提出的受保护资源请求。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 请确保使用包含 Azure AD B2C 租户的目录，方法是单击顶部菜单中的“目录和订阅筛选器”，然后选择包含租户的目录。
3. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“Azure AD B2C”。
4. 选择“应用程序”，然后选择“添加”。
5. 输入应用程序的名称。 例如，“webapi1”。
6. 对于“包括 Web 应用/Web API”和“允许隐式流”，请选择“是”。
7. 对于“回复 URL”，请输入 Azure AD B2C 要将应用程序请求的任何令牌返回到的终结点。 本教程中的示例在本地运行并在 `https://localhost:5000` 上进行侦听。
8. 对于“应用 ID URI”，请输入 Web API 使用的标识符。 包括域在内的完整标识符 URI 是为你生成的。 例如，`https://contosotenant.onmicrosoft.com/api`。
9. 单击“创建”。
10. 在属性页上，记录在配置 Web 应用程序时要使用的应用程序 ID。

## <a name="configure-scopes"></a>配置范围

可通过范围控制对受保护资源的访问。 Web API 使用作用域实施基于作用域的访问控制。 例如，可以让某些用户拥有读取和写入访问权限，让另一些用户拥有只读权限。 在本教程中，你将定义对 Web API 的读取权限。

1. 依次选择“应用程序”、“webapi1”。
2. 选择“已发布的范围”。
3. 在“范围”中输入 `Hello.Read`，在“说明”中输入 `Read access to hello`。
4. 在“范围”中输入 `Hello.Write`，在“说明”中输入 `Write access to hello`。
5. 单击“ **保存**”。

可以使用发布的作用域向客户端应用授予对 Web API 的权限。

## <a name="grant-permissions"></a>授予权限

若要从应用程序调用受保护的 Web API，需授予应用程序访问该 API 的权限。 在先决条件教程中，已在 Azure AD B2C 中创建名为 *webapp1* 的 Web 应用程序。 使用此应用程序调用 Web API。

1. 依次选择“应用程序”、“webapp1”。
2. 依次选择“API 访问”、“添加”。
3. 在“选择 API”下拉列表中，选择“webapi1”。
4. 在“选择范围”下拉列表中，选择之前定义的“Hello.Read”和“Hello.Write”范围。
5. 单击“确定”。

这将注册“我的示例单页应用”，以便调用受保护的 **Hello Core API**。 用户通过 Azure AD B2C 进行身份验证，以使用单页应用程序。 单页应用从 Azure AD B2C 获取授权，以访问受保护的 Web API。

## <a name="configure-the-sample"></a>配置示例

注册 Web API 并定义范围之后，请将 Web API 代码配置为使用你的 Azure AD B2C 租户。 本教程将配置一个可从 GitHub 下载的示例 .NET Core Web 应用程序。 从 GitHub [下载 zip 文件](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapi/archive/master.zip)或克隆示例 Web 应用。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapi.git
```

### <a name="configure-the-web-api"></a>配置 Web API

1. 在 Visual Studio 中打开 **B2C-WebAPI.sln** 解决方案。
2. 打开 **appsettings.json** 文件。 更新以下值来将 Web API 配置为使用你的租户：

    ```javascript
    "AzureAdB2C": 
      {
        "Tenant": "<your tenant name>.onmicrosoft.com", 
        "ClientId": "<application-ID>",
        "Policy": "B2C_1_signupsignin1>",
        "ScopeRead": "Hello.Read"  
      },
    ```

#### <a name="enable-cors"></a>启用 CORS

若要允许单页应用程序调用 ASP.NET Core Web API，需要启用 [CORS](https://docs.microsoft.com/aspnet/core/security/cors)。

1. 在 **Startup.cs** 中，将 CORS 添加到 `ConfigureServices()` 方法。

    ```C#
    public void ConfigureServices(IServiceCollection services) {
      services.AddCors();
    ```

2. 在 **Startup.cs** 中，在 `Configure()` 方法中配置 CORS。

    ```C#
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory) {
      app.UseCors(builder =>
        builder.WithOrigins("http://localhost:6420").AllowAnyHeader().AllowAnyMethod());
    ```

3. 打开“属性”下的 launchSettings.json 文件，找到 iisSettings applicationURL 设置，然后将端口号设置为已向 API 回复 URL `http://localhost:5000` 注册的一个数字。

### <a name="configure-the-single-page-application"></a>配置单页应用程序

单页应用程序使用 Azure AD B2C 完成用户注册、登录，并调用受保护的 ASP.NET Core Web API。 更新单页应用程序以调用 .NET Core Web API。

若要更改应用设置，请执行以下操作：

1. 打开 `index.html` 文件。
2. 使用 Azure AD B2C 租户注册信息来配置示例。 在以下代码中，将租户名称添加到 **b2cScopes**，并将 **webApi** 值更改为以前记录的 *applicationURL* 值：

    ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    var applicationConfig = {
        clientID: '<application-ID>',
        authority: "https://<your-tenant-name>.b2clogin.com/tfp/<your-tenant-name>.onmicrosoft.com/B2C_1_signupsignin1",
        b2cScopes: ["https://<Your tenant name>.onmicrosoft.com/api/Hello.Read"],
        webApi: 'http://localhost:5000/api/values',
    };
    ```

## <a name="run-the-spa-application-and-web-api"></a>运行 SPA 应用程序和 Web API

需要同时运行 Node.js 单页应用程序和 .NET Core Web API。

### <a name="run-the-aspnet-core-web-api"></a>运行 ASP.NET Core Web API 

在 Visual Studio 中按 **F5** 来调试 **B2C-WebAPI.sln** 解决方案。

当项目启动时，会在你的默认浏览器中打开一个网页，宣告该 Web API 已可供请求使用。

### <a name="run-the-single-page-app"></a>运行单页应用

1. 启动 Node.js 命令提示符。
2. 切换到包含 Node.js 示例的目录。 例如 `cd c:\active-directory-b2c-javascript-msal-singlepageapp`
3. 运行以下命令：
    ```
    npm install && npm update
    node server.js
    ```

    控制台窗口将显示该应用程序所在的端口号。
    
    ```
    Listening on port 6420...
    ```

4. 使用浏览器导航到地址 `http://localhost:6420` 以查看该应用程序。
5. 使用[在单页应用程序 (JavaScript) 中使用 Azure Active Directory B2C 对用户进行身份验证](active-directory-b2c-tutorials-spa.md)中使用的电子邮件地址和密码进行登录。
6. 单击“调用 API”。

在使用用户帐户进行注册或登录后，该示例会调用受保护的 Web API 并返回一个结果。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 添加 Web API 应用程序
> * 配置 Web API 的范围
> * 授予 Web API 权限
> * 将示例配置为使用此应用程序

> [!div class="nextstepaction"]
> [教程：将标识提供者添加到 Azure Active Directory B2C 中的应用程序](tutorial-add-identity-providers.md)
