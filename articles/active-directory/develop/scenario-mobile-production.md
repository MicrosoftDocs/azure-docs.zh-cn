---
title: 调用 web Api （移动到生产环境）-Microsoft 标识平台的移动应用
description: 了解如何构建一个移动应用，调用 Web Api （移动到生产环境）
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
ms.openlocfilehash: d37d2de561a6f5841bf17a47fef86ad7639750d5
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074945"
---
# <a name="mobile-app-that-calls-web-apis---move-to-production"></a>移动应用程序调用 web Api-移动到生产环境

本文提供有关提高质量和可靠性的应用程序以将其移到生产环境的详细信息。

## <a name="handling-errors-in-mobile-applications"></a>在移动应用程序中处理错误

在我们到目前为止已突出显示不同流中，有各种可以导致的错误条件。 若要处理的主要方案是无提示的失败和回退到交互。 有还应考虑包括任何网络的情况下，服务中断、 必需的管理员同意和其他特定于方案的情况下的生产环境的附加条件。

每个 MSAL 库具有的深入探讨到处理这些条件的示例代码和 wiki 内容。

- [MSAL Android Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-android)
- [MSAL iOS Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki)
- [MSAL.NET Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki)

## <a name="mitigating-and-investigating-issues"></a>缓解和调查的问题

收集的数据将帮助您诊断问题的应用程序。 有关详细信息的数据类型可以收集，请参阅每个 MSAL 平台 wiki。

- 遇到问题时，用户可能会要求有关的帮助。 最佳做法是以捕获和存储日志，并允许用户将某个位置上传。 MSAL 提供了捕获身份验证。 有关详细的信息的日志记录扩展
- 如果可用，则启用遥测数据通过 MSAL 来收集有关用户如何登录到您的应用程序数据。

## <a name="testing-your-app"></a>测试应用程序

请务必测试对应用程序[集成检查列表](identity-platform-integration-checklist.md)。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]
