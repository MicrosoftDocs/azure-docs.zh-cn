---
title: Web 应用登录用户 （应用程序注册）-Microsoft 标识平台
description: 了解如何生成 web 应用登录用户 （应用程序注册）
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
ms.openlocfilehash: e6f512911811535818f4ad857c5c3b956870f619
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074540"
---
# <a name="web-app-that-signs-in-users---app-registration"></a>登录用户的应用注册的 web 应用

此页说明登录用户的 web 应用的应用程序注册具体信息。

若要注册你的应用程序，可以使用：

- [Web 应用快速入门](#register-an-app-using-the-quickstarts)-除了创建应用程序很好第一次体验外，在 Azure 门户中的快速入门包含名为的按钮**进行此更改对我来说**。 可以使用此按钮以设置您需要甚至对于现有应用程序的属性。 你将需要调整为你自己的大小写这些属性的值。 具体而言，您的应用程序的 web API URL 可能会使其不同于建议默认情况下，还将影响注销 URI。
- Azure 门户，转到[手动注册你的应用程序](#register-an-app-using-azure-portal)
- PowerShell 和命令行工具

## <a name="register-an-app-using-the-quickstarts"></a>注册使用快速入门应用

如果你导航到此链接，则可以创建启动 web 应用程序的创建：

- [ASP.NET Core](https://aka.ms/aspnetcore2-1-aad-quickstart-v2)
- [ASP.NET](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs)

### <a name="register-an-app-using-azure-portal"></a>注册使用 Azure 门户的应用

> [!NOTE]
> 若要使用该门户是在 Microsoft Azure 公有云或国家/地区或主权云中运行应用程序具体取决于不同。 有关详细信息，请参阅[国家/地区云](./authentication-national-cloud.md#app-registration-endpoints)

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。 或者，登录到 Azure 门户选择的国家/地区云。
1. 如果你的帐户有权访问多个租户，在右上角中，选择你的帐户和将门户会话设置为所需的 Azure AD 租户。
1. 在左侧导航窗格中，选择**Azure Active Directory**服务，，然后选择**应用注册** > **新注册**。
1. 出现“注册应用程序”页后，请输入应用程序的注册信息：
   - 选择你的应用程序的支持的帐户类型 (请参阅[支持帐户类型](./v2-supported-account-types.md))
   - 在“名称”部分输入一个会显示给应用用户的有意义的应用程序名称，例如 `AspNetCore-WebApp`。
   - 在**回复 URL**，例如添加您的应用程序，回复 URL `https://localhost:44321/`，然后选择**注册**。
1. 选择“身份验证”菜单，然后添加以下信息：
- 在“回复 URL”中添加 `https://localhost:44321/signin-oidc`，然后选择“注册”。
- 在中**高级设置**部分中，设置**登出 URL**到`https://localhost:44321/signout-oidc`。
- 在“隐式授权”下，勾选“ID 令牌”。
- 选择“保存”。

### <a name="register-an-app-using-powershell"></a>注册应用程序使用 PowerShell

> [!NOTE]
> 当前 Azure AD PowerShell 仅当创建应用程序与以下受支持的帐户类型：
>
> - MyOrg （仅限此组织目录中的帐户）
> - AnyOrg （在任何组织的目录中的帐户）。
>
> 如果你想要创建的应用程序登录用户使用其个人 Microsoft 帐户 (例如 Skype，XBox、 Outlook.com)，可以先创建一个多租户应用程序 (支持的帐户类型 = 在任何组织的目录中的帐户)，然后将更改`signInAudience`从 Azure 门户在应用程序清单中的属性。 这在步骤的详细信息中所述[1.3](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-3-AnyOrgOrPersonal#step-1-register-the-sample-with-your-azure-ad-tenant)的 ASP.NET Core 教程 （和可以归纳为任何语言中的 web 应用）。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用程序的代码配置](scenario-web-app-sign-user-app-configuration.md)
