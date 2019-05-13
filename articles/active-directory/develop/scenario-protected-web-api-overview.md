---
title: 受保护的 Web API-概述 |Azure
description: 了解如何生成受保护的 web API （概述）。
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
ms.openlocfilehash: c5bc0075e6604bd1f94531040f2e4a0628e70667
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074885"
---
# <a name="scenario-protected-web-api"></a>场景：受保护的 web API

在此方案中，我们将展示如何将公开 web API 和如何保护它，以便只有经过身份验证的用户可以访问 API。 你将想要启用与工作和学校帐户或个人 Microsoft 个人帐户已经过身份验证的用户使用你的 web API。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="specifics"></a>详细信息

下面是一些需要了解保护 web Api 的详细信息：

- 应用注册必须公开至少一个作用域。 接受的 web API 的令牌版本取决于受众中登录。
- Web API 的代码的配置必须验证调用 web API 时使用的令牌。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用注册](scenario-protected-web-api-app-registration.md)
