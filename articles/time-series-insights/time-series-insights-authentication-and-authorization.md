---
title: 如何在 Azure 时序见解中使用 API 进行身份验证和授权 | Microsoft Docs
description: 本文介绍如何为调用 Azure 时序见解 API 的自定义应用程序配置身份验证和授权。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 05/07/2019
ms.custom: seodec18
ms.openlocfilehash: 5fb2802bfe9cc0a4d3297e6fa749e5b94008c616
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472613"
---
# <a name="authentication-and-authorization-for-azure-time-series-insights-api"></a>Azure 时序见解 API 的身份验证和授权

本文介绍如何配置调用 Azure 时序见解 API 的自定义应用程序中使用的身份验证和授权。

> [!TIP]
> 阅读有关[授予数据访问](./time-series-insights-data-access.md)到你在 Azure Active Directory 中的时序见解环境。

## <a name="service-principal"></a>服务主体

这和以下各部分介绍如何配置应用程序代表应用程序访问时序见解 API。 应用程序可使用然后查询或将引用数据发布的应用程序凭据而非用户凭据在时序见解环境中。

## <a name="best-practices"></a>最佳做法

当你具有的应用程序须访问时序见解：

1. 设置 Azure Active Directory 应用程序。
1. [将数据访问策略分配](./time-series-insights-data-access.md)时序见解环境中。

使用应用程序而不是你的用户凭据最好自：

* 可以将权限分配给应用标识，不同于你自己的权限。 通常情况下，这些权限仅限于应用需要的权限。 例如，可仅允许应用读取特定时序见解环境中的数据。
* 您无需更改应用的凭据，如果用户职责改变。
* 执行无人参与的脚本时，可使用证书或应用程序密钥自动进行身份验证。

以下各节演示如何通过 Azure 门户执行这些步骤。 本文重点介绍单租户应用程序应用程序应只有一个组织中运行。 通常将使用在组织中运行的业务线应用程序的单租户应用程序。

## <a name="set-up-summary"></a>设置摘要

设置流程包含三个步骤：

1. 在 Azure Active Directory 中创建应用程序。
1. 授权此应用程序访问时序见解环境。
1. 使用应用程序 ID 和密钥从一个令牌`https://api.timeseries.azure.com/`。 然后可使用该令牌调用时序见解 API。

## <a name="detailed-setup"></a>详细的设置

1. 在 Azure 门户中，依次选择 Azure Active Directory > “应用注册” > “新应用程序注册”。

   [![Azure Active Directory 中的新应用程序注册](media/authentication-and-authorization/active-directory-new-application-registration.png)](media/authentication-and-authorization/active-directory-new-application-registration.png#lightbox)

1. 命名应用程序，选择类型“Web 应用/API”，然后为“登录 URL”选择任意有效 URI，单击“创建”。

   [![在 Azure Active Directory 中创建应用程序](media/authentication-and-authorization/active-directory-create-web-api-application.png)](media/authentication-and-authorization/active-directory-create-web-api-application.png#lightbox)

1. 选择新创建的应用程序，并将其应用程序 ID 复制到喜爱的文本编辑器中。

   [![复制应用程序 ID](media/authentication-and-authorization/active-directory-copy-application-id.png)](media/authentication-and-authorization/active-directory-copy-application-id.png#lightbox)

1. 选择“密钥”，输入密钥名称，选择到期时间，然后单击“保存”。

   [![选择应用程序密钥](media/authentication-and-authorization/active-directory-application-keys.png)](media/authentication-and-authorization/active-directory-application-keys.png#lightbox)

   [![输入密钥名称和到期时间，然后单击保存](media/authentication-and-authorization/active-directory-application-keys-save.png)](media/authentication-and-authorization/active-directory-application-keys-save.png#lightbox)

1. 将密钥复制到喜爱的文本编辑器中。

   [![复制应用程序密钥](media/authentication-and-authorization/active-directory-copy-application-key.png)](media/authentication-and-authorization/active-directory-copy-application-key.png#lightbox)

1. 对于时序见解环境，请选择“数据访问策略”，然后单击“添加”。

   [![将新的数据访问策略添加到时序见解环境](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png#lightbox)

1. 在中**选择用户**对话框框中，粘贴应用程序名称 (从**步骤 2**) 或应用程序 ID (从**步骤 3**)。

   [![在选择用户对话框中查找应用程序](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png#lightbox)

1. 选择的角色 (**读取器**用于查询数据，**参与者**查询数据和更改引用数据) 并单击**确定**。

   [![在选择角色对话框中选择读取者或参与者](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png#lightbox)

1. 通过单击保存策略**确定**。

1. 使用应用程序 ID (从**第 3 步**) 和应用程序密钥 (从**第 5 步**) 以获取代表应用程序的令牌。 随后可在应用程序调用时序见解 API 时，将令牌传入 `Authorization` 标头。

    如果正在使用 C#，可使用以下代码代表应用程序获取令牌。 有关完整示例，请参阅[使用 C# 查询数据](time-series-insights-query-data-csharp.md)。

    ```csharp
    // Enter your Active Directory tenant domain name
    var tenant = "YOUR_AD_TENANT.onmicrosoft.com";
    var authenticationContext = new AuthenticationContext(
        $"https://login.microsoftonline.com/{tenant}",
        TokenCache.DefaultShared);

    AuthenticationResult token = await authenticationContext.AcquireTokenAsync(
        // Set the resource URI to the Azure Time Series Insights API
        resource: "https://api.timeseries.azure.com/",
        clientCredential: new ClientCredential(
            // Application ID of application registered in Azure Active Directory
            clientId: "YOUR_APPLICATION_ID",
            // Application key of the application that's registered in Azure Active Directory
            clientSecret: "YOUR_CLIENT_APPLICATION_KEY"));

    string accessToken = token.AccessToken;
    ```

使用**应用程序 ID**并**密钥**中应用程序以使用 Azure 时序见解进行身份验证。

## <a name="next-steps"></a>后续步骤

- 有关调用时序见解 API 的示例代码，请参阅[使用 C# 查询数据](time-series-insights-query-data-csharp.md)。

- 有关 API 参考信息，请参阅[查询 API 参考](/rest/api/time-series-insights/ga-query-api)。

- 了解如何[创建服务主体](../active-directory/develop/howto-create-service-principal-portal.md)。
