---
title: 连接到 Azure AD 时对 MVC 项目所做的更改
description: 描述一下，使用 Visual Studio 连接服务连接到 Azure AD 时，MVC 项目会发生什么情况
services: active-directory
author: ghogen
manager: douge
ms.assetid: 8b24adde-547e-4ffe-824a-2029ba210216
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: aaddev, vs-azure
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1e40a1fd72adf353d5c4d856544482cb7bd68f43
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413990"
---
# <a name="what-happened-to-my-mvc-project-visual-studio-azure-active-directory-connected-service"></a>我的 MVC 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况？

> [!div class="op_single_selector"]
> - [入门](vs-active-directory-dotnet-getting-started.md)
> - [发生了什么情况](vs-active-directory-dotnet-what-happened.md)

本文介绍在[使用 Visual Studio 添加 Azure Active Directory 连接服务](vs-active-directory-add-connected-service.md)时，对 ASP.NET MVC 项目所做的具体更改。

有关使用连接服务的信息，请参阅[入门](vs-active-directory-dotnet-getting-started.md)。

## <a name="added-references"></a>添加的引用

影响项目文件 *.NET 引用和 `packages.config`（NuGet 引用）。

| Type | 参考 |
| --- | --- |
| .NET; NuGet | Microsoft.IdentityModel.Protocol.Extensions |
| .NET; NuGet | Microsoft.Owin |
| .NET; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET; NuGet | Microsoft.Owin.Security |
| .NET; NuGet | Microsoft.Owin.Security.Cookies |
| .NET; NuGet | Microsoft.Owin.Security.OpenIdConnect |
| .NET; NuGet | Owin |
| .NET        | System.IdentityModel |
| .NET; NuGet | System.IdentityModel.Tokens.Jwt |
| .NET        | System.Runtime.Serialization |

选择了“读取目录数据”选项时的其他引用：

| Type | 参考 |
| --- | --- |
| .NET; NuGet | EntityFramework |
| .NET        | EntityFramework.SqlServer（仅限 Visual Studio 2015） |
| .NET; NuGet | Microsoft.Azure.ActiveDirectory.GraphClient |
| .NET; NuGet | Microsoft.Data.Edm |
| .NET; NuGet | Microsoft.Data.OData |
| .NET; NuGet | Microsoft.Data.Services.Client |
| .NET; NuGet | Microsoft.IdentityModel.Clients.ActiveDirectory |
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms（仅限 Visual Studio 2015） |
| .NET; NuGet | System.Spatial |

删除了以下引用（仅限 ASP.NET 4 项目，如 Visual Studio 2015 中所示）：

| Type | 参考 |
| --- | --- |
| .NET; NuGet | Microsoft.AspNet.Identity.Core |
| .NET; NuGet | Microsoft.AspNet.Identity.EntityFramework |
| .NET; NuGet | Microsoft.AspNet.Identity.Owin |

## <a name="project-file-changes"></a>项目文件更改

- 将属性 `IISExpressSSLPort` 设置为不同的数字。
- 将属性 `WebProject_DirectoryAccessLevelKey` 设置为 0 或 1（如果选择了“读取目录数据”选项）。
- 将属性 `IISUrl` 设置为 `https://localhost:<port>/`，其中 `<port>` 匹配 `IISExpressSSLPort` 值。

## <a name="webconfig-or-appconfig-changes"></a>web.config 或 app.config 发生更改

- 添加了以下配置条目：

    ```xml
    <appSettings>
        <add key="ida:ClientId" value="<ClientId from the new Azure AD app>" />
        <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
        <add key="ida:Domain" value="<your selected Azure domain>" />
        <add key="ida:TenantId" value="<the Id of your selected Azure AD tenant>" />
        <add key="ida:PostLogoutRedirectUri" value="<project start page, such as https://localhost:44335>" />
    </appSettings>
    ```

- 在 `System.IdentityModel.Tokens.Jwt` 和 `Microsoft.IdentityModel.Protocol.Extensions` 的 `<runtime><assemblyBinding>` 节点下添加了 `<dependentAssembly>` 元素。

选择了“读取目录数据”选项时的其他更改：

- 在 `<appSettings>` 下添加了以下配置条目：

    ```xml
    <add key="ida:ClientSecret" value="<Azure AD app's new client secret>" />
    ```

- 在 `<configuration>` 下添加了以下元素；project-mdf-file 和 project-catalog-id 的值将有变化：

    ```xml
    <configSections>
      <!-- For more information on Entity Framework configuration, visit https://go.microsoft.com/fwlink/?LinkID=237468 -->
      <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>

    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\<project-mdf-file>.mdf;Initial Catalog=<project-catalog-id>;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>

    <entityFramework>
      <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
        <parameters>
          <parameter value="mssqllocaldb" />
        </parameters>
      </defaultConnectionFactory>
      <providers>
        <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      </providers>
    </entityFramework>
    ```

- 在 `Microsoft.Data.Services.Client`、`Microsoft.Data.Edm` 和 `Microsoft.Data.OData` 的 `<runtime><assemblyBinding>` 节点下添加了 `<dependentAssembly>` 元素。

## <a name="code-changes-and-additions"></a>代码更改和添加

- 向 `Controllers/HomeController.cs` 和任何其他现有控制器添加了 `[Authorize]` 属性。

- 添加了身份验证启动类 `App_Start/Startup.Auth.cs`，其中包含 Azure AD 身份验证的启动逻辑。 如果选择了“读取目录数据”选项，则此文件还包含用于接收 OAuth 代码以及用 OAuth 代码交换访问令牌的代码。

- 添加了控制器类 `Controllers/AccountController.cs`，其中包含 `SignIn` 和 `SignOut` 方法。

- 添加了分部视图 `Views/Shared/_LoginPartial.cshtml`，其中包含 `SignIn` 和 `SignOut` 的操作链接。

- 添加了分部视图 `Views/Account/SignoutCallback.cshtml`，其中包含注销 UI 的 HTML。

- 更新了 `Startup.Configuration` 方法，以包含当类存在时对 `ConfigureAuth(app)` 的调用；否则添加包含调用方法的 `Startup` 类。

- 添加了 `Connected Services/AzureAD/ConnectedService.json`（Visual Studio 2017）或 `Service References/Azure AD/ConnectedService.json`（Visual Studio 2015），其中包含 Visual Studio 用来跟踪连接服务添加的信息。

- 如果选择了“读取目录数据”选项，则已将 `Models/ADALTokenCache.cs` 和 `Models/ApplicationDbContext.cs` 添加到支持令牌缓存。 另外添加了一个控制器和视图，以演示如何使用 Azure 图形 API `Controllers/UserProfileController.cs`、`Views/UserProfile/Index.cshtml` 和 `Views/UserProfile/Relogin.cshtml` 访问用户配置文件信息

### <a name="file-backup-visual-studio-2015"></a>文件备份 (Visual Studio 2015)

添加连接服务时，Visual Studio 2015 备份已更改并已删除文件。 所有受影响的文件均保存在文件夹 `Backup/AzureAD` 中。 Visual Studio 2017 和更高版本不会创建备份。

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ManageViewModels.cs`
- `Views\Shared\_LoginPartial.cshtml`

## <a name="changes-on-azure"></a>对 Azure 的更改

- 在添加连接服务时选择的域中创建了 Azure AD 应用程序。
- 更新了应用，以便在选择了“读取目录数据”选项时包含“读取目录数据”权限。

[详细了解 Azure Active Directory](https://azure.microsoft.com/services/active-directory/)。

## <a name="next-steps"></a>后续步骤

- [Azure Active Directory 的身份验证方案](authentication-scenarios.md)
- [向 ASP.NET Web 应用添加 Microsoft 登录功能](quickstart-v1-aspnet-webapp.md)
