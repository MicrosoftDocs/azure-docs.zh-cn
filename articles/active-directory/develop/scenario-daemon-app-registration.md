---
title: 守护程序应用程序调用 web Api （应用程序注册）-Microsoft 标识平台
description: 了解如何构建守护程序调用 web Api-应用注册
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
ms.openlocfilehash: 79a355ab226e56a3dde1df5369deda5142d47848
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076235"
---
# <a name="daemon-app-that-calls-web-apis---app-registration"></a>守护程序应用程序调用 web Api 的应用注册

守护程序应用程序中，下面是您需要了解时注册该应用。

## <a name="supported-account-types"></a>支持的帐户类型

考虑到守护程序应用程序仅有意义 Azure AD 租户中创建应用程序时，需要进行选择：

- 任一**此组织目录中的帐户**。 选择此选项是最常见的情况，如守护程序应用程序通常由业务线 (LOB) 开发人员编写。
- 或**任何组织的目录中的帐户**。 如果你是 ISV 为客户提供的实用程序工具，你将使此选项。 你将需要客户的租户管理员的批准。

## <a name="authentication---no-reply-uri-needed"></a>身份验证-没有所需的回复 URI

在其中机密客户端应用程序使用的情况下**仅**客户端凭据流，回复 URL 无需进行注册。 它不需要为应用程序配置/构造。 客户端凭据流不会使用它。

## <a name="api-permissions---app-permissions-and-admin-consent"></a>API 权限的应用权限和管理员同意

后台应用程序只能请求对 Api （非委派权限） 的应用程序权限。 在**API 的权限**后所选的应用程序注册页**添加权限**，然后选择 API 系列，选择**应用程序权限**，然后选择你的权限

![应用权限和管理员同意](media/scenario-daemon-app/app-permissions-and-admin-consent.png)

守护程序应用程序需要具有租户管理员预先同意调用 web API 的应用程序。 在同一个提供此许可**API 的权限**页上，通过租户管理员选择**授予管理员同意*我们的组织***

如果你是 ISV 构建多租户应用程序，你想要检查[部署的用例的多租户守护程序应用](scenario-daemon-production.md#deployment---case-of-multi-tenant-daemon-apps)段落。

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [守护程序应用-应用程序代码配置](./scenario-daemon-app-configuration.md)
