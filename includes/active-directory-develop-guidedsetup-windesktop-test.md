---
title: include 文件
description: include 文件
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/10/2019
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: a11b291ab89dc9f8159e00e1f2304706f041068e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "60297604"
---
## <a name="test-your-code"></a>测试代码

要在 Visual Studio 中运行项目，请选择 **F5**. 随即显示应用程序 MainWindow，如下所示：

![测试应用程序](./media/active-directory-develop-guidedsetup-windesktop-test/samplescreenshot.png)

首次运行该应用程序并选择“调用 Microsoft Graph API”按钮时，系统会提示登录。 使用 Azure Active Directory 帐户（工作或学校帐户）或 Microsoft 帐户（live.com、outlook.com）进行测试。

![登录应用程序](./media/active-directory-develop-guidedsetup-windesktop-test/signinscreenshot.png)

### <a name="provide-consent-for-application-access"></a>许可应用程序访问

首次登录到应用程序时，还会提示你同意允许应用程序访问你的个人资料并使你登录，如下所示：

![许可应用程序访问](./media/active-directory-develop-guidedsetup-windesktop-test/consentscreen.png)

### <a name="view-application-results"></a>查看应用程序结果

在登录后，应当会显示由 Microsoft Graph API 调用返回的用户配置文件信息。 结果会显示在“API 调用结果”框中。 有关通过调用 `AcquireTokenInteractive` 或 `AcquireTokenSilent` 获取的令牌的基本信息，应当会在“令牌信息”框中显示。 结果包含以下属性：

|属性  |格式  |说明 |
|---------|---------|---------|

|**Username** |<span>user@domain.com</span> |用于标识用户的用户名。| |**令牌到期** |DateTime |令牌到期的时间。 MSAL 根据需要通过续订令牌来延长到期日期。|


<!--start-collapse-->
### <a name="more-information-about-scopes-and-delegated-permissions"></a>有关作用域和委派权限的详细信息

Microsoft Graph API 需要 *user.read* 作用域来读取用户的个人资料。 默认情况下，在应用程序注册门户中注册的每个应用程序中，都会自动添加此作用域。 Microsoft Graph 的其他 API 以及后端服务器的自定义 API 可能需要其他作用域。 Microsoft Graph API 需要 *Calendars.Read* 作用域来列出用户的日历。

若要在应用程序上下文中访问用户的日历，请将 *Calendars.Read* 委派权限添加到应用程序注册信息。 然后，将 *Calendars.Read* 作用域添加到 `acquireTokenSilent` 调用。

>[!NOTE]
>当你增加作用域数量时，可能会提示用户另外进行许可。

<!--end-collapse-->

[!INCLUDE [Help and support](./active-directory-develop-help-support-include.md)]
