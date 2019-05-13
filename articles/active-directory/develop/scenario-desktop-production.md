---
title: 桌面应用程序调用 web Api （移动到生产环境）-Microsoft 标识平台
description: 了解如何构建桌面应用调用 web Api （移动到生产环境）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/18/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3ca66a41f26c54bf04273682d14889a36b688c70
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075125"
---
# <a name="desktop-app-that-calls-web-apis---move-to-production"></a>桌面应用程序调用 web Api-移动到生产环境

本文提供详细信息来改进应用程序进一步并将其移到生产环境。

## <a name="handling-errors-in-desktop-applications"></a>在桌面应用程序中处理错误

在不同的流中，已了解如何处理无提示的流的错误 （代码段所示）。 您也可以看到有交互的情况下需要 （增量许可和条件性访问）。

## <a name="how-to-have--the-user-consent-upfront-for-several-resources"></a>如何让用户同意提前为多个资源

> [!NOTE]
> 多个资源的工作原理有关 Microsoft 标识平台，而不是 Azure Active Directory (Azure AD) B2C 中获得许可。 Azure AD B2C 支持仅管理员许可，用户的同意。

Microsoft 标识平台 (v2.0) 终结点不允许您以一次性获取多个资源的令牌。 因此，`scopes`参数只能包含单个资源的作用域。 您可以确保，用户预许可给多个资源使用`extraScopesToConsent`参数。

例如，如果您有两个资源，其中有两个范围每个：

- `https://mytenant.onmicrosoft.com/customerapi` -2 个作用域与`customer.read`和 `customer.write`
- `https://mytenant.onmicrosoft.com/vendorapi` -2 个作用域与`vendor.read`和 `vendor.write`

应使用`.WithAdditionalPromptToConsent`修饰符具有`extraScopesToConsent`参数。

例如：

```CSharp
string[] scopesForCustomerApi = new string[]
{
  "https://mytenant.onmicrosoft.com/customerapi/customer.read",
  "https://mytenant.onmicrosoft.com/customerapi/customer.write"
};
string[] scopesForVendorApi = new string[]
{
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.read",
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.write"
};

var accounts = await app.GetAccountsAsync();
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithAccount(accounts.FirstOrDefault())
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

此调用将获取第一个 web API 的访问令牌。

当您需要调用第二个 web API 时，您可以调用：

```CSharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```

### <a name="microsoft-personal-account-requires-reconsenting-each-time-the-app-is-run"></a>Microsoft 个人帐户需要 reconsenting 每次运行该应用程序

对于 Microsoft 个人帐户的用户，对每个本机客户端 （桌面/移动应用） 调用来授权同意 reprompting 是预期的行为。 本机客户端标识是本质上是不安全 （与机密客户端应用程序的交换的 Microsoft 标识平台，以证明其身份的机密）。 Microsoft 标识平台选择通过提示用户同意，授权该应用程序每次来缓解此安全的使用者服务。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]
