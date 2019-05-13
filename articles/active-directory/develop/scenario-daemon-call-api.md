---
title: 守护程序应用程序调用 web Api (调用 web Api)-Microsoft 标识平台
description: 了解如何构建守护程序应用调用 web Api (调用 web Api)
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
ms.openlocfilehash: aff375f996126d9e8b64361fc0e5673c25d30c19
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076265"
---
# <a name="daemon-app-that-calls-web-apis---call-a-web-api-from-the-app"></a>守护程序应用程序调用 web Api-从应用调用 web API

守护程序应用程序可以从.NET 守护程序应用程序调用 web API 或调用多个预先批准的 web Api。

## <a name="calling-a-web-api-from-a-net-daemon-application"></a>从.NET 守护程序应用程序调用 web API

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

<!--
More includes will come later for Python and Java
-->

## <a name="calling-several-apis"></a>调用了几个 Api

对于后台程序应用程序，web Api 的调用需要预先批准。 不会有任何增量许可与守护程序应用 （没有无需用户交互）。 租户管理员需要预先同意该应用程序和 API 的所有权限。 如果你想要调用的多个 Api，你将需要获取每个资源的令牌每次调用`AcquireTokenForClient`。 MSAL 将使用应用程序令牌缓存以避免不必要的服务调用。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [守护程序应用-移动到生产环境](./scenario-daemon-production.md)