---
title: Web API 调用 web Api （移动到生产环境）-Microsoft 标识平台
description: 了解如何构建 web API 调用下游 web Api （移动到生产环境）。
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
ms.openlocfilehash: 7de892a0851e556c5046fe4b3691f39b42a9d237
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074750"
---
# <a name="web-api-that-calls-web-apis---move-to-production"></a>Web API 调用 web Api-移动到生产环境

一旦获取令牌来调用 web Api 后，可以将应用移动到生产环境。

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="learn-more"></a>了解详细信息

既然你知道如何调用的基础知识 web Api 从你自己的 web API，你可能希望本教程中，其中描述了用于生成受保护的 web API 的代码可以调用 web Api。

| 示例 | 平台 | 描述 |
|--------|----------|-------------|
| [active-directory-aspnetcore-webapi-tutorial-v2](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/tree/master/2.%20Web%20API%20now%20calls%20Microsoft%20Graph) | ASP.NET Core 2.2 Web API，桌面 (WPF) | 从使用 Microsoft 标识平台 (v2.0) 的 WPF 应用程序本身调用 ASP.NET Core 2.2 Web API 调用 Microsoft Graph， |
