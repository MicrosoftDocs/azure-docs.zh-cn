---
title: 移动应用程序调用 web Api 的应用程序的代码配置 |Microsoft 标识平台
description: 了解如何构建移动应用程序调用 web Api （应用程序的代码配置）
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9eac9d1dfce79ac4ad9d49a6cfe6b7dee7f6681a
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075155"
---
# <a name="mobile-app-that-calls-web-apis---app-registration"></a>调用 web Api 的应用注册的移动应用

本文包含有关创建移动应用程序的应用程序注册说明。

## <a name="supported-account-types"></a>支持的帐户类型

在移动应用程序中支持的帐户类型取决于你想要启用的体验和您的应用程序所面向的用户。

## <a name="platform-configuration-and-redirect-uris"></a>平台配置和重定向 Uri  

在创建移动应用时，最关键的注册步骤是重定向 URI。 这可以通过设置[身份验证边栏选项卡中的平台配置](https://aka.ms/MobileAppReg)。

这种体验将启用应用以获取单一登录 (SSO) 通过 Microsoft Authenticator （和 Android 上的 Intune 公司门户） 以及支持设备管理策略。

如果想要手动配置的重定向 URI，可以通过应用程序清单来执行此操作。 以下是建议的格式：

- ***iOS***: `msauth.<BUNDLE_ID>://auth`
- ***Android***: `msauth://<PACKAGE_NAME>/<SIGNATURE_HASH>`
  - 可以使用通过 KeyTool 命令的版本或调试密钥生成 Android 签名哈希。

## <a name="api-permissions"></a>API 权限

移动应用程序代表已登录用户调用 Api。 您的应用程序需要请求委托的权限，也称为作用域。 具体取决于所需的体验，这可以以静态方式通过 Azure 门户或动态地在运行时。 静态注册权限允许管理员轻松地批准您的应用程序，建议使用。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [获取令牌](scenario-mobile-acquire-token.md)
