---
title: 教程 - 授予对 ASP.NET Web API 的访问权限 - Azure Active Directory B2C | Microsoft Docs
description: 有关如何使用 Active Directory B2C 保护 ASP.NET Web API 并通过 ASP.NET Web 应用程序对其进行调用的教程。
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.author: davidmu
ms.date: 02/04/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 77e3eaeffba862c727e021427e5f27967fcf35bd
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64687994"
---
# <a name="tutorial-grant-access-to-an-aspnet-web-api-using-azure-active-directory-b2c"></a>教程：使用 Azure Active Directory B2C 授予对 ASP.NET Web API 的访问权限

本教程介绍如何从 ASP.NET Web 应用程序调用 Azure Active Directory (Azure AD) B2C 中受保护的 Web API 资源。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 添加 Web API 应用程序
> * 配置 Web API 的范围
> * 授予 Web API 权限
> * 将示例配置为使用此应用程序

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先决条件

完成[教程：使用 Azure Active Directory B2C 在 Web 应用程序中启用身份验证](active-directory-b2c-tutorials-web-app.md)中的步骤和先决条件。

## <a name="add-a-web-api-application"></a>添加 Web API 应用程序

Web API 资源需要先在租户中注册，然后才能接受并响应提供访问令牌的客户端应用程序所提出的受保护资源请求。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 请确保使用包含 Azure AD B2C 租户的目录，方法是单击顶部菜单中的“目录和订阅筛选器”，然后选择包含租户的目录。
3. 选择 Azure 门户左上角的“所有服务”，然后搜索并选择“Azure AD B2C”。
4. 选择“应用程序”，然后选择“添加”。
5. 输入应用程序的名称。 例如，“webapi1”。
6. 对于“包括 Web 应用/Web API”和“允许隐式流”，请选择“是”。
7. 对于“回复 URL”，请输入 Azure AD B2C 要将应用程序请求的任何令牌返回到的终结点。 本教程中的示例在本地运行并在 `https://localhost:44332` 上进行侦听。
8. 对于“应用 ID URI”，请输入 Web API 使用的标识符。 包括域在内的完整标识符 URI 是为你生成的。 例如，`https://contosotenant.onmicrosoft.com/api`。
9. 单击“创建”。
10. 在属性页上，记录在配置 Web 应用程序时要使用的应用程序 ID。

## <a name="configure-scopes"></a>配置范围

可通过范围控制对受保护资源的访问。 Web API 使用作用域实施基于作用域的访问控制。 例如，可以让 Web API 用户拥有读取和写入访问权限，或者只拥有读取访问权限。 在本教程中，请使用作用域为 Web API 定义读取和写入权限。

1. 选择“应用程序”，然后选择“webapi1”。
2. 选择“已发布的范围”。
3. 在“范围”中输入 `Hello.Read`，在“说明”中输入 `Read access to hello`。
4. 在“范围”中输入 `Hello.Write`，在“说明”中输入 `Write access to hello`。
5. 单击“ **保存**”。

可以使用已发布的范围向客户端应用程序授予访问 Web API 的权限。

## <a name="grant-permissions"></a>授予权限

若要从应用程序调用受保护的 Web API，需授予应用程序访问该 API 的权限。 在先决条件教程中，已在 Azure AD B2C 中创建名为 *webapp1* 的 Web 应用程序。 使用此应用程序调用 Web API。

1. 依次选择“应用程序”、“webapp1”。
2. 依次选择“API 访问”、“添加”。
3. 在“选择 API”下拉列表中，选择“webapi1”。
4. 在“选择范围”下拉列表中，选择之前定义的“Hello.Read”和“Hello.Write”范围。
5. 单击“确定”。

注册应用程序以调用受保护的 Web API。 用户通过 Azure AD B2C 进行身份验证，以便使用该应用程序。 该应用程序从 Azure AD B2C 获取授权，以访问受保护的 Web API。

## <a name="configure-the-sample"></a>配置示例

注册 Web API 并定义范围之后，请将 Web API 配置为使用你的 Azure AD B2C 租户。 在本教程中，可以配置一个示例 Web API。 在先决条件教程中下载的项目中包含了示例 Web API。

示例解决方案中有两个项目：

示例解决方案中包含以下两个项目：

- **TaskWebApp** - 创建和编辑任务列表。 该示例使用**注册或登录**用户流来注册或登录用户。
- **TaskService** - 支持创建、读取、更新和删除任务列表功能。 该 API 受 Azure AD B2C 的保护，由 TaskWebApp 调用。

### <a name="configure-the-web-application"></a>配置 Web 应用程序

1. 在 Visual Studio 中打开 **B2C-WebAPI-DotNet** 解决方案。
2. 打开 **TaskWebApp** 项目中的 **Web.config**。
3. 若要在本地运行 API，请使用针对 **api:TaskServiceUrl** 的 localhost 设置。 更改 Web.config，如下所示： 

    ```C#
    <add key="api:TaskServiceUrl" value="https://localhost:44332/"/>
    ```

3. 配置 API 的 URI。 这是 Web 应用程序用来发起 API 请求的 URI。 另请配置所请求的权限。

    ```C#
    <add key="api:ApiIdentifier" value="https://<Your tenant name>.onmicrosoft.com/api/" />
    <add key="api:ReadScope" value="Hello.Read" />
    <add key="api:WriteScope" value="Hello.Write" />
    ```

### <a name="configure-the-web-api"></a>配置 Web API

1. 打开 **TaskService** 项目中的 **Web.config**。
2. 配置要使用租户的 API。

    ```C#
    <add key="ida:Tenant" value="<Your tenant name>.onmicrosoft.com" />
    ```

3. 将客户端 ID 设置为 API 的注册应用程序 ID。

    ```C#
    <add key="ida:ClientId" value="<application-ID>"/>
    ```

4. 使用注册和登录用户流的名称更新用户流设置。

    ```C#
    <add key="ida:SignUpSignInUserFlowId" value="B2C_1_signupsignin1" />
    ```

5. 配置作用域设置，使之与门户中创建的设置相匹配。

    ```C#
    <add key="api:ReadScope" value="Hello.Read" />
    <add key="api:WriteScope" value="Hello.Write" />
    ```

## <a name="run-the-sample"></a>运行示例

**TaskWebApp** 和 **TaskService** 项目都需要运行。 

1. 在解决方案资源管理器中，右键单击解决方案并选择“设置启动项目...”。 
2. 选择“多启动项目”。
3. 将两个项目的“操作”更改为“启动”。
4. 单击“确定”以保存配置。
5. 按 **F5** 运行这两个应用程序。 每个应用程序将在其自身的浏览器标签页中打开。`https://localhost:44316/` 为 Web 应用程序。
    `https://localhost:44332/` 为 Web API。

6. 在 Web 应用程序中，单击“注册/登录”以登录到 Web 应用程序。 使用前面创建的帐户。 
7. 登录后，单击“待办事项列表”并创建待办事项列表项。

创建待办事项列表项时，Web 应用程序会向 Web API 发起生成待办事项列表项的请求。 受保护的 Web 应用程序将调用 Azure AD B2C 租户中的受保护 Web API。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 添加 Web API 应用程序
> * 配置 Web API 的范围
> * 授予 Web API 权限
> * 将示例配置为使用此应用程序

> [!div class="nextstepaction"]
> [教程：将标识提供者添加到 Azure Active Directory B2C 中的应用程序](tutorial-add-identity-providers.md)
