---
title: 教程 - 在本机客户端应用程序中启用身份验证 - Azure Active Directory B2C | Microsoft Docs
description: 有关如何使用 Azure Active Directory B2C 为 .NET 桌面应用程序提供用户登录功能的教程。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.author: davidmu
ms.date: 02/04/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 7ca7a670e8d660efb43861312b2670efbaac6638
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190246"
---
# <a name="tutorial-enable-authentication-in-a-native-client-application-using-azure-active-directory-b2c"></a>教程：使用 Azure Active Directory B2C 在本机客户端应用程序中启用身份验证

本教程展示了如何在 Windows Presentation Foundation (WPF) 桌面应用程序中使用 Azure Active Directory (Azure AD) B2C 执行用户登录和注册。 应用程序可以使用 Azure AD B2C 通过开放式标准协议对社交帐户、企业帐户和 Azure Active Directory 帐户进行身份验证。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 添加本机客户端应用程序
> * 将示例配置为使用此应用程序
> * 使用用户流进行登录

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

- [创建用户流](tutorial-create-user-flows.md)，以便在应用程序中启用用户体验。 
- 安装 [Visual Studio 2019](https://www.visualstudio.com/downloads/)，其中包含 **.NET 桌面开发**与 **ASP.NET 和 Web 开发**工作负荷。

## <a name="add-the-native-client-application"></a>添加本机客户端应用程序

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 请确保使用包含 Azure AD B2C 租户的目录，方法是单击顶部菜单中的“目录和订阅筛选器”，然后选择包含租户的目录。
3. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“Azure AD B2C”。
4. 选择“应用程序”，然后选择“添加”。
5. 输入应用程序的名称。 例如，“nativeapp1”。
6. 对于“包括 Web 应用/Web API”，请选择“否”。
7. 对于“包括本机客户端”，请选择“是”。
8. 对于“重定向 URI”，请使用自定义方案输入有效的重定向 URI。 选择重定向 URI 时，有两个重要的注意事项：

    - **唯一** - 每个应用程序的重定向 URI 方案应是唯一的。 在示例 `com.onmicrosoft.contoso.appname://redirect/path` 中，`com.onmicrosoft.contoso.appname` 为方案。 应遵循此模式。 如果两个应用程序共享同一方案，则用户应选择一个应用程序。 如果用户的选择不正确，登录会失败。
    - **完整** - 重定向 URI 必须同时包含方案和路径。 路径必须在域之后包含至少一个正斜杠。 例如，`//contoso/` 有效，而 `//contoso` 会失败。 确保重定向 URI 不包含特殊字符，例如下划线。

9. 单击“创建”。
10. 在属性页上，记下你在配置示例时将使用的应用程序 ID。

## <a name="configure-the-sample"></a>配置示例

在本教程中，你将配置一个可从 GitHub 下载的示例。 示例 WPF 桌面应用程序演示了如何在 Azure AD B2C 中注册、登录和调用受保护的 Web API。 从 GitHub [下载 zip 文件](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop/archive/master.zip)、[浏览存储库](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop)或克隆示例。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop.git
```

若要更改应用设置，请将 `<your-tenant-name>` 替换为你的租户名称，将 `<application-ID`> 替换为你记下的应用程序 ID。

1. 在 Visual Studio 中打开 `active-directory-b2c-wpf` 解决方案。
2. 在 `active-directory-b2c-wpf` 项目中，打开 **App.xaml.cs** 文件并进行以下更新：

    ```C#
    private static string Tenant = "<your-tenant-name>.onmicrosoft.com";
    private static string ClientId = "<application-ID>";
    ```

3. 将 **PolicySignUpSignIn** 变量更新为你创建的用户流的名称。

    ```C#
    public static string PolicySignUpSignIn = "B2C_1_signupsignin1";
    ```

## <a name="run-the-sample"></a>运行示例

按 **F5** 生成并运行示例。

### <a name="sign-up-using-an-email-address"></a>使用电子邮件地址注册

1. 单击“登录”以用户身份登录。 这将使用 **B2C_1_signupsignin1** 用户流。
2. Azure AD B2C 会显示带注册链接的登录页面。 由于你还没有帐户，因此请单击“立即注册”链接。 
3. 注册工作流会显示一个页面，用于收集用户的标识并通过电子邮件地址对其进行验证。 注册工作流还收集用户的密码和请求的属性（在用户流中定义）。

    请使用有效的电子邮件地址，并使用验证码进行验证。 设置密码。 输入请求的属性的值。 

    ![注册工作流](media/active-directory-b2c-tutorials-desktop-app/sign-up-workflow.png)

4. 单击“创建”，在 Azure AD B2C 租户中创建本地帐户。

现在，用户可以使用其电子邮件地址登录并使用桌面应用了。

> [!NOTE]
> 如果单击“调用 API”按钮，将会收到“未经授权”错误。 收到此错误是因为你正在尝试从演示租户访问资源。 因为你的访问令牌仅对你的 Azure AD 租户有效，所以该 API 调用未经授权。 请继续学习下一教程来为你的租户创建受保护的 Web API。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 添加本机客户端应用程序
> * 将示例配置为使用此应用程序
> * 使用用户流进行登录

> [!div class="nextstepaction"]
> [教程：使用 Azure Active Directory B2C 从桌面应用授予对 Node.js Web API 的访问权限](active-directory-b2c-tutorials-spa-webapi.md)
