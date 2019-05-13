---
title: 受保护的 web API-应用注册 |Azure
description: 了解如何构建一个受保护的 Web API 和你需要注册该应用程序的信息。
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
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
ms.openlocfilehash: a1d8c8fcf84cd008957fcdb7cd14c4a07d9f3643
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074900"
---
# <a name="protected-web-api---app-registration"></a>受保护的 web API-应用注册

本文介绍了受保护的 web API 的应用程序注册具体信息。

请参阅[快速入门：注册一个应用程序的 Microsoft 标识平台](quickstart-register-app.md)有关如何注册应用程序的常见步骤。

## <a name="accepted-token-version"></a>接受令牌的版本

Microsoft 标识平台终结点可以颁发两种类型的令牌： v1.0 令牌和 v2.0 令牌。 您可以了解有关在这些令牌的详细信息[访问令牌](access-tokens.md)。 接受令牌的版本取决于**支持的帐户类型**创建你的应用程序时选择：

- 如果的值**支持的帐户类型**是**中的任何组织的目录和个人 Microsoft 帐户 (例如 Skype，Xbox、 Outlook.com) 帐户**，接受令牌的版本将为 v2.0。
- 否则，接受令牌的版本将 v2.0。

一旦你创建应用程序，你可以接受令牌版本通过执行以下步骤：

1. 在 Azure 门户中，选择您的应用程序，然后选择**清单**为你的应用。
2. 在清单中，搜索 **"accessTokenAcceptedVersion"**，并确保其值是**2**。 此属性允许 Azure AD 知道 web API 接受 v2.0 令牌。 如果它是**null**，接受令牌的版本将为 1.0 版。
3. 选择“保存”。

> [!NOTE]
> 它是对 web API 来决定它接受令牌版本 （1.0 版或 v2.0）。 当客户端请求的 web API 使用 Microsoft 标识平台 v2.0 终结点的标记时，他们将获得一个标记，指示 web API 接受的版本。

## <a name="no-redirect-uri"></a>没有重定向 URI

无需注册的重定向 URI，因为没有用户以交互方式是登录的 web Api。

## <a name="expose-an-api"></a>公开 API

特定于 web Api 的另一个设置是公开的 API 和公开作用域。

### <a name="resource-uri-and-scopes"></a>资源 URI 和作用域

作用域的形式通常是`resourceURI/scopeName`。 Microsoft Graph 的作用域具有类似的快捷方式`User.Read`，但此字符串是一种方便的`https://graph.microsoft.com/user.read`。

在应用程序注册期间你将需要定义以下参数：

- 默认情况下，应用程序注册门户的一个资源 URI-建议你使用`api://{clientId}`。 此资源 URI 是唯一的但它不是人工可读。 可以更改它，但请确保它是唯一的。
- 一个或多个作用域

作用域也会提供给最终用户使用你的应用程序的许可屏幕上显示。 因此，您将需要提供描述作用域的相应字符串：

- 最终用户查看
- 查看租户管理员，可以授予管理员同意

### <a name="how-to-expose-the-api"></a>如何将 API 公开

1. 选择**公开一个 API**应用程序注册中的部分和：
   1. 选择“添加范围”。
   1. 接受建议的应用程序 ID URI (api:// {clientId}) 通过选择**保存并继续**。
   1. 输入以下参数：
      - 有关**作用域名称**，使用`access_as_user`。
      - 有关**人员便可以同意**，请确保**管理员和用户**选择选项。
      - 在中**管理员许可显示名称**，类型`Access TodoListService as a user`。
      - 在中**管理员许可说明**，类型`Accesses the TodoListService Web API as a user`。
      - 在中**用户许可显示名称**，类型`Access TodoListService as a user`。
      - 在中**用户许可说明**，类型`Accesses the TodoListService Web API as a user`。
      - 保持**状态**设置为**已启用**。
      - 选择**添加作用域**。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用程序的代码配置](scenario-protected-web-api-app-configuration.md)
