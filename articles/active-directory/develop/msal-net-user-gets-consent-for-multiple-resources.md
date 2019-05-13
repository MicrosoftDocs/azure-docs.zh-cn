---
title: 获取同意的情况下的多个资源 （适用于.NET 的 Microsoft 身份验证库） |Azure
description: 了解用户如何获取预同意使用 Microsoft 身份验证库.NET (MSAL.NET) 的多个资源。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/30/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 45bad9a03e8eff6f22ebb99fd2ef4bcd5fecf9b5
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65158815"
---
# <a name="user-gets-consent-for-several-resources-using-msalnet"></a>用户获取多个资源，使用 MSAL.NET 的许可
Microsoft 标识平台终结点不允许您以一次性获取多个资源的令牌。 在使用 Microsoft 身份验证库.NET (MSAL.NET) 时，获取令牌方法中的作用域参数只应包含单个资源的作用域。 但是，您可以预先同意提前几个资源通过指定其他作用域使用`.WithExtraScopeToConsent`生成器方法。

> [!NOTE]
> 获取同意的情况下的多个资源的工作原理有关 Microsoft 标识平台，而不是 Azure AD B2C。 Azure AD B2C 支持仅管理员许可，用户的同意。

例如，如果已有的两个资源 2 的范围限定每个：

- https://mytenant.onmicrosoft.com/customerapi (2 个作用域`customer.read`和`customer.write`)
- https://mytenant.onmicrosoft.com/vendorapi (2 个作用域`vendor.read`和`vendor.write`)

应使用`.WithExtraScopeToConsent`具有修饰符*extraScopesToConsent*参数，如下面的示例中所示：

```csharp
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

这将为第一个 web API 帮助你访问令牌。 然后，当需要访问第二个 web API 时可以以无提示方式获取令牌缓存的令牌：

```csharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```
