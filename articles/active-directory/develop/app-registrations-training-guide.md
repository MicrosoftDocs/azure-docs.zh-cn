---
title: 在 Azure 门户的培训指南-Azure 中的应用注册
description: 使用设备代码授予生成嵌入式无浏览器身份验证流。
services: active-directory
documentationcenter: ''
author: archieag
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: aragra
ms.reviewer: lenalepa, keyam
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 118c6ecb16d325a384246a0b3d9e685f6f6f04ee
ms.sourcegitcommit: e7d4881105ef17e6f10e8e11043a31262cfcf3b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2019
ms.locfileid: "64870112"
---
# <a name="training-guide-app-registrations-in-the-azure-portal"></a>培训指南：在 Azure 门户中的应用注册  

您可以在新发现了大量改进[应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)体验在 Azure 门户中。 如果您更熟悉旧的体验，使用此培训指南帮助你开始使用新的体验。

## <a name="key-changes"></a>密钥更改

- 应用注册并不限于要么**web 应用 /API**或**本机**应用。 对于所有这些通过注册相应的重定向 Uri，可以使用相同的应用注册。
- 登录的支持的旧体验应用组织 (Azure AD) 帐户仅。 应用注册为单租户 （支持从目录中注册应用程序仅组织帐户），并且无法被修改为多租户 （支持所有的组织帐户）。 新的体验，可注册应用，可以支持这两个这些选项，以及第三个选项： 所有组织帐户，以及个人 Microsoft 帐户。
- 仅登录到 Azure 门户中使用组织帐户时提供传统体验的。 使用新的体验，可以使用不与目录相关联的个人 Microsoft 帐户。

## <a name="list-of-applications"></a>应用程序列表

- 新的应用程序列表显示了通过旧应用程序已注册的应用程序在 Azure 门户 （在 Azure AD 帐户登录的应用程序） 以及应用程序的注册体验已注册但[应用程序注册门户](https://apps.dev.microsoft.com/)(在 Azure AD 中登录的应用和个人 Microsoft 帐户)。
- 新的应用程序列表中没有**应用程序类型**列 （因为单个应用注册可以是几种类型） 和具有两个其他列：**上创建**列和一个**证书& 机密**的凭据已注册的应用上显示的状态 （当前、 很快到期，或已过期） 的列。

## <a name="new-app-registration"></a>新的应用注册

在旧体验中，若要注册的应用需要提供：**名称**，**应用程序类型**，和**单一登录 URL/重定向 URI**。 已创建的应用的 Azure AD 仅单租户应用程序，这意味着它们仅支持从目录中注册应用程序的组织帐户。

在新体验中，你必须提供**名称**应用程序，然后选择**支持的帐户类型**。 你可以根据需要提供**重定向 URI**。 如果提供的重定向 URI，将需要指定它是 web/公用 （移动和桌面）。 有关详细信息如何注册使用新的应用注册的应用体验，请参阅[本快速入门](quickstart-register-app.md)。

## <a name="the-legacy-properties-page"></a>旧的属性页

旧的体验有**属性**没有的新体验的页。 **属性**边栏选项卡具有以下字段：**名称**，**对象 ID**，**应用程序 ID**，**应用程序 ID URI**，**徽标**，**主页 URL****注销 URL**，**服务条款 URL**，**隐私声明 URL**，**应用程序类型**，和**多租户。**

下面是可以在其中找到的等效功能在新的体验：

- **名称**，**徽标**，**主页 URL**，**服务条款 URL**，并且**隐私声明 URL**现在处于启用状态的应用的**品牌**页。
- **对象 ID**并**应用程序 （客户端） ID**位于**概述**页。
- 由控制的功能**多租户**已被取代的旧体验中切换**支持的帐户类型**上**身份验证**页。 有关如何将多租户映射到支持的帐户类型选项的详细信息，请参阅[本快速入门](quickstart-modify-supported-accounts.md)。
- **注销 URL**目前所在**身份验证**页。
- **应用程序类型**不再是有效字段。 相反，重定向 Uri (可在上找到**身份验证**页) 确定哪些应用类型受支持。
- **应用程序 ID URI**现在称为**应用程序 ID URI**上可以找到这**公开 API**边栏选项卡。 在旧体验中，此属性是自动注册使用以下格式： `https://{tenantdomain}/{appID}` (例如， `https://microsoft.onmicrosoft.com/aeb4be67-a634-4f20-9a46-e0d4d4f1f96d`)。 在新的格式，它是自动生成作为`api://{appID}`，但它需要显式保存。

## <a name="reply-urlsredirect-urls"></a>回复 Url/重定向 Url

在旧体验中，应用程序必须**答复 Url**页。 在新体验中，可以应用的上找到回复 Url**身份验证**部分。 此外，它们被称为**重定向 Uri**。 此外，重定向的格式已更改的 Uri。 它们都需要与应用程序类型 （web 或公共） 相关联。 此外，出于安全原因，通配符和 http:// 方案不支持 (除 http://localhost)。

## <a name="keyscertificates--secrets"></a>密钥/证书和机密

在旧体验中，应用程序必须**密钥**页。 在新体验中，它已更名为**证书和机密**。 此外，**公钥**嘿 **证书**并**密码**称为**客户端机密**。

## <a name="required-permissionsapi-permissions"></a>所需的权限/API 权限

- 在旧体验中，应用程序必须**所需的权限**页。 在新体验中，它已更名为**API 权限**。
- 在旧体验中，选择 API，无法从 Microsoft Api 或搜索租户中的服务主体的小列表中进行选择。 在新体验中，您可以选择从多个选项卡：**Microsoft Api**，**我的组织使用的 Api**，或**我的 Api**。 在搜索栏**Api 我的组织**在租户中使用服务主体在选项卡搜索。 

   > [!NOTE]
   > 如果你的应用程序未与租户相关联，不会看到此选项卡。 有关如何请求使用的新体验的权限的详细信息，请参阅[本快速入门](quickstart-configure-app-access-web-apis.md)。

- 旧的体验有**授予的权限**顶部的按钮**请求的权限**页。 在新体验，没有**授予许可**节替换**授予管理员许可**上的应用按钮**API 权限**部分。 此外，有一些差异方式按钮函数：
   - 在旧体验中，逻辑各不相同具体取决于已登录用户和所请求的权限。 逻辑是：
      - 如果只请求用户可同意的权限，并且已登录用户不是管理员，用户就能够授予所请求的权限的用户同意。
      - 如果所请求至少一个要求管理员同意的权限，并且已登录用户不是管理员，用户将收到错误，当尝试授予许可。
      - 如果已登录用户是管理员，管理员同意已授予对所有请求的权限。
   - 在新体验中，只有管理员才能授予许可。 当管理员选择**授予管理员同意**按钮，管理员同意授予对所有请求的权限。

## <a name="deleting-an-app-registration"></a>正在删除的应用注册

在旧体验中，应用程序必须为要删除的单租户。 多租户应用的情况下，删除按钮已被禁用。 在新体验中，应用可以删除处于任何状态，但必须确认该操作。 有关删除应用程序注册信息的详细信息，请参阅[本快速入门](quickstart-remove-app.md)。

## <a name="application-manifest"></a>应用程序清单

旧的和新体验使用不同版本的清单编辑器中的 JSON 格式。 有关详细信息，请参阅[应用程序清单](reference-app-manifest.md)。

## <a name="new-ui"></a>新的用户界面

没有可以以前仅使用清单编辑器或设置 API，或不存在的属性的新用户界面。

- **隐式授权流**(oauth2AllowImplicitFlow) 可以位于**身份验证**页。 与在旧体验中，您可以启用**访问令牌**或**id 令牌**，和 / 或。
- **通过此 API 定义作用域**(oauth2Permissions) 和**授权客户端应用程序**(preAuthorizedApplications) 可通过配置**公开 API**页。 有关如何配置应用以进行 web API 和公开的权限/作用域的详细信息，请参阅[本快速入门](quickstart-configure-app-expose-web-apis.md)。
- **发行者域**(这向用户显示上[应用程序的许可提示](application-consent-experience.md)) 上可以找到**品牌边栏选项卡**页。 有关如何配置发布服务器域的详细信息，请参阅[本操作说明](howto-configure-publisher-domain.md)。

## <a name="limitations"></a>限制

新体验中有以下限制：

- 新体验目前不在 Azure AD B2C 租户中可用。
- 客户端机密 （应用程序密码） 的格式不同于传统的体验，中断 CLI。
- 在 UI 中不支持更改受支持的帐户的值。 您需要使用应用程序清单，除非您单租户和多租户的 Azure AD 之间的切换。
