---
title: 桌面应用程序调用 web Api （概述）-Microsoft 标识平台
description: 了解如何构建桌面应用调用 web Api （概述）
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
ms.openlocfilehash: 44d31011ca70bbebaf994b5fb80a45eee8dbde40
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65076940"
---
# <a name="scenario-desktop-app-that-calls-web-apis"></a>场景：桌面应用程序调用 web Api

了解您需要构建桌面应用调用 web Api

## <a name="prerequisites"></a>必备组件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>入门

如果尚未安装，请按照以下.NET 桌面快速入门或 UWP 快速入门中创建第一个应用程序：

> [!div class="nextstepaction"]
> [快速入门：获取令牌并调用 Microsoft Graph API 从 Windows 桌面应用](./quickstart-v2-windows-desktop.md)


> [!div class="nextstepaction"]
> [快速入门：获取令牌，然后从 UWP 应用调用 Microsoft Graph API](./quickstart-v2-uwp.md)

## <a name="overview"></a>概述

编写桌面应用程序，并且你想要用户登录到你的应用程序，并调用 web Api，如 Microsoft Graph、 其他 Microsoft Api 或你自己的 web API。 有以下几种可能性：

- 如果桌面应用程序支持图形控件，例如如果 Windows.Form 应用程序或 WPF 应用程序，可以使用交互式令牌获取。
- 对于托管的 Windows 应用程序，还有可能加入 Windows 域的计算机上运行的应用程序或 AAD 联接以获取令牌以无提示方式使用集成 Windows 身份验证。
- 最后，尽管不建议这样做，您可以在公共客户端应用程序中使用用户名/密码。 它仍需要在某些情况下 （如 DevOps)，但要注意，使用它将施加约束你的应用程序。 例如，它不能在需要执行多重身份验证 （条件性访问） 用户登录。 你的应用程序不会有益的单一登录 (SSO)。

  它也是针对新式身份验证的原则，并仅提供由于历史原因。

  ![桌面应用程序](media/scenarios/desktop-app.svg)

- 如果您正在编写可移植的命令行工具-可能在 Linux 或 Mac 上运行的.NET Core 应用程序-你将无法使用任一的交互式身份验证 (如.NET Core 不提供[Web 浏览器](https://aka.ms/msal-net-uses-web-browser))，也不集成Windows 身份验证。 在这种情况下的最佳选项是使用设备代码流。 此流还可用于应用程序，而浏览器中，如 iOT 应用程序

  ![Browserless 应用程序](media/scenarios/device-code-flow-app.svg)

## <a name="specifics"></a>详细信息

桌面应用程序有很多 specificities，这主要取决于是否您的应用程序是否使用交互式身份验证。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [桌面应用程序-应用注册](scenario-desktop-app-registration.md)
