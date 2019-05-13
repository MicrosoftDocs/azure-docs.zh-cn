---
title: Web API 的调用下游 web Api （应用程序注册）-Microsoft 标识平台
description: 了解如何构建 web API 的调用下游 web Api （应用程序注册）
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
ms.openlocfilehash: fb03869cdea2150b6e922e2d6d81e577c3be02da
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075380"
---
# <a name="web-api-that-calls-web-apis---app-registration"></a>Web API 调用 web Api 的应用注册

调用下游 web Api 的 web API 已作为受保护的 web API 的相同注册。 因此，您将需要按照中的说明[受保护的 Web API 的应用注册](scenario-protected-web-api-app-registration.md)。

但是，因为 web 应用现在调用 web Api，它将成为机密客户端应用程序。 这就是为什么没有具有所需的额外的注册信息： 应用程序需要与 Microsoft 标识平台共享机密 （客户端凭据）。

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>API 权限

Web 应用程序代表用户为其接收持有者令牌调用 Api。 他们需要请求委托的权限。 有关详细信息，请参阅[添加权限以访问 web Api](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用程序的代码配置](scenario-web-api-call-api-app-configuration.md)
