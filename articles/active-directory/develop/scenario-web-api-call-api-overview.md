---
title: Web API 调用下游 web Api （概述）-Microsoft 标识平台
description: 了解如何构建 web API 的调用下游 web Api （概述）。
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
ms.openlocfilehash: 497134b7f3cc535f7b3f180db13cd04ef56787db
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075395"
---
# <a name="scenario-web-api-that-calls-web-apis"></a>场景：调用 web Api 的 web API

了解构建 web API 所需的所有调用 web Api。

## <a name="prerequisites"></a>必备组件

此方案中，受保护的 web API 调用 web Api，"保护 web API"方案的基础上构建。 若要了解有关此基本方案的详细信息，请参阅[受保护的 Web API 的方案](scenario-protected-web-api-overview.md)第一个。

## <a name="overview"></a>概述

- 客户端 （web、 桌面、 移动、 或单页面应用程序）-不在下面的关系图上表示-调用受保护的 web API，并提供其"授权"Http 标头中的 JWT 持有者令牌。
- 受保护的 web API 验证令牌，然后使用 MSAL`AcquireTokenOnBehalfOf`方法来请求 （从 Azure AD) 另一个令牌，以便它可以本身，调用第二个 web API （名为下游 web API） 代表用户。
- 受保护的 web API 使用此令牌来调用下游 API。 它还可以调用`AcquireTokenSilent`更高版本以请求令牌的其他下游 Api （但仍代表相同的用户）。 `AcquireTokenSilent` 刷新令牌时需要。

![Web API 调用 web API](media/scenarios/web-api.svg)

## <a name="specifics"></a>详细信息

与 API 权限相关的应用注册的一部分是传统。 应用程序配置涉及到使用 OAuth 2.0 上的代理流来交换下游 API 的 JWT 持有者令牌针对令牌。 此令牌添加到令牌缓存中，它现已推出 web API 的控制器，并可以获取令牌以无提示方式以调用下游 Api。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-web-api-call-api-app-registration.md)
