---
title: JavaScript 单页面应用程序方案概述-Microsoft 标识平台
description: 了解如何构建集成 Microsoft 标识平台的单页面应用程序 （方案概述）。
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 07a21e83f304f3e1acc0ed4033d832dd8e901ac9
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076355"
---
# <a name="scenario-single-page-application"></a>场景：单页应用程序

了解所有需要生成单页面应用程序 (SPA)。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Prerequisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>入门

可以通过以下 JavaScript SPA 快速入门中创建第一个应用程序：

> [!div class="nextstepaction"]
> [快速入门：单页面应用程序](./quickstart-v2-javascript.md)

## <a name="overview"></a>概述

许多现代 web 应用程序是作为客户端使用 JavaScript 或如 Angular、 Vue.js 和 React.js SPA 框架编写的单页面应用程序。 这些应用程序的 web 浏览器中运行，并且具有不同的身份验证特征与传统服务器端 web 应用程序。 使用 Microsoft 标识平台，在用户登录并获取令牌以访问后端服务或 web Api 使用的单页面应用程序[OAuth 2.0 隐式流](./v2-oauth2-implicit-grant-flow.md)。 隐式流允许应用程序获取 ID 令牌，表示已经过身份验证的用户和访问所需令牌来调用受保护的 Api。

![单页应用程序](./media/scenarios/spa-app.svg)

此身份验证流不包括使用跨平台 JavaScript 框架，如 Electron，React 本机应用程序方案。 因为它们需要进一步的功能与本机平台进行交互。

## <a name="specifics"></a>详细信息

启用此方案中的为应用程序所需的以下方面：

* 使用 Azure AD 应用程序注册涉及启用隐式流和设置令牌返回到重定向 URI。
* 应用程序配置与注册的应用程序属性，如应用程序 id。
* 使用 MSAL 库来执行身份验证流，以登录和获取令牌。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-spa-app-registration.md)
