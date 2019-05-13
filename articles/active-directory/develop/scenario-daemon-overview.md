---
title: 守护程序应用程序调用 web Api （概述）-Microsoft 标识平台
description: 了解如何构建守护程序调用 web Api
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
ms.openlocfilehash: 578b7cdb38b7df3fab5885d773354a36f76a4cfb
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075875"
---
# <a name="scenario-daemon-application-that-calls-web-apis"></a>场景：后台应用程序调用 web Api

了解您需要构建一个调用 web Api 的后台程序应用程序。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="overview"></a>概述

你的应用程序可以获取令牌以调用 web API 代表本身 （而不是代表用户）。 此方案非常有用的守护程序应用程序。 它使用标准 OAuth 2.0[客户端凭据](v2-oauth2-client-creds-grant-flow.md)授予。

![守护程序应用](./media/scenario-daemon-app/daemon-app.svg)

下面是一些示例的守护程序应用的用例：

- 使用预配或管理用户，或进行批处理的目录中的进程的 web 应用程序
- 例如，（在 Windows、 windows 服务） 或 Linux 上的守护程序进程的桌面应用程序执行批处理作业或在后台运行的操作系统服务
- Web Api，需要处理不特定于用户的目录

还有一种非守护程序应用程序在其中使用客户端凭据的常见情况： 即使它们代表用户执行操作，他们需要出于技术原因访问 web API 或使用其标识的资源。 例如，为缓存访问密钥保管库或 Azure SQL 数据库中的机密。

获取其自己的身份进行的令牌的应用程序：

- 是机密客户端应用程序。 这些应用，假设他们访问独立于用户，资源需要证明其身份。 它们也是相当敏感应用程序，它们需要由 Azure Active Directory (Azure AD) 租户管理员批准。
- 已与 Azure AD 中注册的机密 （应用程序密码或证书）。 在调用 Azure AD 以获取令牌的过程，传入的是此密钥。

## <a name="specifics"></a>详细信息

> [!IMPORTANT]
>
> - 用户交互不能与后台程序应用程序。 后台应用程序需要其自己的标识。 此类型的应用程序请求访问令牌通过使用其应用程序标识并提供了自己的应用程序 ID、 凭据 （密码或证书） 和应用程序 ID URI 向 Azure AD。 身份验证成功后，守护程序从 Microsoft 标识平台终结点，然后用于调用 web API （并根据需要刷新） 接收访问令牌 （和刷新令牌）。
> - 由于用户交互不是可能的不会可以增量许可。 需要在应用程序注册配置所有所需的 API 权限和应用程序的代码只是请求以静态方式定义的权限。 这也意味着后台程序应用程序不支持增量许可。

对于开发人员，此方案的端到端体验有以下几个方面：

- 守护程序应用程序仅可在 Azure AD 租户中。 它没有意义生成尝试操作 Microsoft 个人帐户的后台应用程序。 如果您的业务线 (LOB) 应用程序开发人员，您将在租户中创建守护程序应用程序。 如果你是 ISV，你可能想要创建的多租户后台应用程序。 它将需要由每个租户管理员同意
- 期间[应用程序注册](./scenario-daemon-app-registration.md)，则**回复 URI**不需要。 您需要与 Azure AD 共享机密或证书，你需要请求应用程序的权限并授予管理员同意才可以使用这些应用程序权限。
- [应用程序配置](./scenario-daemon-app-configuration.md)需要与 Azure AD 应用程序注册期间提供共享的客户端凭据。
- [作用域](scenario-daemon-acquire-token.md#scopes-to-request)用于获取包含流必须是静态的作用域的客户端凭据的令牌。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [守护程序应用-应用注册](./scenario-daemon-app-registration.md)
