---
title: Web 应用登录用户 （移动到生产环境）-Microsoft 标识平台
description: 了解如何生成 web 应用登录用户 （移动到生产环境）
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
ms.openlocfilehash: d41ad2518f885bbaa02dda3b01f0c02e9fc1d217
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074705"
---
# <a name="web-app-that-signs-in-users---move-to-production"></a>登录用户的移动到生产环境的 web 应用

现在，知道如何获取令牌以调用 web Api，了解如何将其移到生产环境。

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>后续步骤

### <a name="calling-web-apis-scenario"></a>调用 web Api 方案

一次 web 应用登录用户，它可以调用 web Api 代表登录的用户。 从 web 应用调用 web Api 是以下方案中的对象：

> [!div class="nextstepaction"]
> [调用 web Api 的 web 应用](scenario-web-app-call-api-overview.md)

### <a name="deep-dive---web-app-tutorial"></a>深入探讨-web 应用教程

了解如何使用 ASP.NET Core 教程登录用户的其他方法： [ms-标识-aspnetcore 的 web 应用的教程](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial)。 这是一个与生产就绪代码包括如何添加登录的 web 应用的渐进式教程。

<!--- Removed the diagram as it's already shown in the above link to GitHub

![Tutorial overview](media/scenarios/aspnetcore-webapp-tutorial.svg)

--->
