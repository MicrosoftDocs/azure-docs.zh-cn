---
title: 通过 Azure Api 的 SaaS 销售 |Azure Marketplace
description: 介绍如何通过市场 API 创建 SaaS 套餐。
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.topic: reference
ms.date: 09/17/2018
ms.author: pabutler
ms.openlocfilehash: a76fb2989320c64ad85b0f41f17798e2d9c743e1
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64941942"
---
# <a name="saas-sell-through-azure---apis"></a>通过 Azure 销售 SaaS - API

本文介绍如何使用 API 创建 SaaS 套餐。 如果已选择“通过 Azure 销售”，则必须通过 API 来订阅你的 SaaS 套餐。  本文划分为两个部分：

-   SaaS 服务与 Azure 市场之间的服务到服务身份验证
-   API 方法和终结点

以下 API 可帮助你将 SaaS 服务与 Azure 相集成：

-   解决
-   订阅
-   转换
-   取消订阅

下图显示了新客户的订阅流，以及这些 API 的使用时机：

![SaaS 套餐 API 流](./media/saas-offer-publish-api-flow.png)


## <a name="service-to-service-authentication-between-saas-service-and-azure-marketplace"></a>SaaS 服务与 Azure 市场之间的服务到服务身份验证

Azure 不会对 SaaS 服务公开给其最终用户的身份验证施加任何约束。 但是，当 SaaS 服务与 Azure 市场 API 通信时，将在 Azure Active Directory (Azure AD) 应用程序的上下文中完成身份验证。

以下部分介绍如何创建 Azure AD 应用程序。


### <a name="register-an-azure-ad-application"></a>注册 Azure AD 应用程序

任何想要使用 Azure AD 功能的应用程序都必须先在 Azure AD 租户中注册。 此注册过程涉及到提供有关应用程序的 Azure AD 详细信息，例如，该应用程序所在位置的 URL、对用户进行身份验证后用于发送答复的 URL、用于标识应用程序的 URI，等等。

若要使用 Azure 门户注册新应用程序，请执行以下步骤：

1. 登录到 [Azure 门户](https://portal.azure.com/)。
2. 如果你的帐户有权访问多个租户，请在右上角单击该帐户，并将门户会话设置为所需的 Azure AD 租户。
3. 在左侧导航窗格中，依次单击“Azure Active Directory”服务、“应用注册”、“新建应用程序注册”。

   ![SaaS AD 应用注册](./media/saas-offer-app-registration.png)

4. 在“创建”页上，输入应用程序的注册信息：
   - **名称**：输入有意义的应用程序名称
   - **应用程序类型**： 
     - 为安装在设备本地的[客户端应用程序](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#client-application)选择“本机”。 此设置用于 OAuth 公共[本机客户端](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#native-client)。
     - 为安装在安全服务器上的[客户端应用程序](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#client-application)和[资源/API 应用程序](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#resource-server)选择“Web 应用/API”。 此设置用于 OAuth 机密性 [Web 客户端](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#web-client)和公共的[基于用户代理的客户端](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#user-agent-based-client)。
     相同的应用程序还可以公开客户端和资源/API。
   - **登录 URL**：对于 Web 应用/API 应用程序，请提供应用的基 URL。 例如， **http:\//localhost:31544**可能是在本地计算机上运行的 web 应用的 URL。 然后，用户将使用此 URL 登录到 Web 客户端应用程序。
   - **重定向 URI**：对于本机应用程序，请提供 Azure AD 返回令牌响应时所用的 URI。 输入值特定于应用程序，例如**http:\//MyFirstAADApp**。

     ![SaaS AD 应用注册](./media/saas-offer-app-registration-2.png)有关 Web 应用程序或本机应用程序的具体示例，请查看 [Azure AD 开发人员指南](https://docs.microsoft.com/azure/active-directory/develop/active-directory-developers-guide)的“快速入门”部分中提供的快速入门引导设置。

5. 完成后，单击“创建”。 Azure AD 会将唯一的应用程序 ID 分配给应用程序，同时你会转到应用程序的注册主页。 根据应用程序是 Web 应用程序还是本机应用程序，会提供不同的选项用于将更多功能添加到应用程序。

   **注意：** 默认情况下，新注册的应用程序配置为只允许同一租户中的用户登录到应用程序。

<a name="api-methods-and-endpoints"></a>API 方法和终结点
-------------------------

以下部分介绍可用于为 SaaS 套餐启用订阅的 API 方法和终结点。

### <a name="get-a-token-based-on-the-azure-ad-app"></a>基于 Azure AD 应用获取令牌

HTTP 方法

`GET`

*请求 URL*

**https://login.microsoftonline.com/*{tenantId}*/oauth2/token**

*URI 参数*

|  **参数名称**  | **必需**  | **说明**                               |
|  ------------------  | ------------- | --------------------------------------------- |
| tenantId             | True          | 已注册的 AAD 应用程序的租户 ID   |
|  |  |  |


请求标头

|  **标头名称**  | **必需** |  **说明**                                   |
|  --------------   | ------------ |  ------------------------------------------------- |
|  Content-Type     | True         | 与请求关联的内容类型。 默认值为 `application/x-www-form-urlencoded`。  |
|  |  |  |


*请求正文*

| **属性名称**   | **必需** |  **说明**                                                          |
| -----------------   | -----------  | ------------------------------------------------------------------------- |
|  Grant_type         | True         | 授权类型。 默认值为 `client_credentials`。                    |
|  Client_id          | True         |  与 Azure AD 应用关联的客户端/应用标识符。                  |
|  client_secret      | True         |  与 Azure AD 应用关联的密码。                               |
|  资源           | True         |  为其请求令牌的目标资源。 默认值为 `62d94f6c-d599-489b-a797-3e10e42fbe22`。 |
|  |  |  |


*响应*

|  **名称**  | 类型       |  **说明**    |
| ---------- | -------------  | ------------------- |
| 200 正常    | TokenResponse  | 请求成功   |
|  |  |  |

*TokenResponse*

示例响应令牌：

``` json
  {
      "token_type": "Bearer",
      "expires_in": "3600",
      "ext_expires_in": "0",
      "expires_on": "15251…",
      "not_before": "15251…",
      "resource": "62d94f6c-d599-489b-a797-3e10e42fbe22",
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayIsImtpZCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayJ9…"
  }               
```

### <a name="marketplace-api-endpoint-and-api-version"></a>市场 API 终结点和 API 版本

Azure 市场 API 的终结点为 `https://marketplaceapi.microsoft.com`。

当前 API 版本为 `api-version=2017-04-15`。


### <a name="resolve-subscription"></a>解析订阅

用户针对解析终结点执行 POST 操作可将市场令牌解析为永久性资源 ID。  资源 ID 是 SAAS 订阅的唯一标识符。 

当用户重定向到 ISV 的网站时，URL 将在查询参数中包含一个令牌。 ISV 应当使用此令牌，并发出请求来解析它。 响应包含资源的唯一 SAAS 订阅 ID、名称、产品/服务 ID 和计划。 此令牌的有效期仅为一小时。

*请求*

**POST**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions/resolve?api-version=2017-04-15**

|  **参数名称** |     **说明**                                      |
|  ------------------ |     ---------------------------------------------------- |
|  api-version        |  用于此请求的操作的版本。   |
|  |  |


*标头*

| **标头密钥**     | **必需** | **说明**                                                                                                                                                                                                                  |
|--------------------|--------------|-----------------------------------------------------------|
| x-ms-requestid     | 否           | 唯一的字符串值，用于跟踪来自客户端的请求，最好是 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。  |
| x-ms-correlationid | 否           | 在客户端上执行的操作的唯一字符串值。 此值将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| Content-type       | 是          | `application/json`                                        |
| authorization      | 是          | JSON Web 令牌 (JWT) 持有者令牌。                    |
| x-ms-marketplace-token| 是| 将用户从 Azure 重定向到 SaaS ISV 网站时 URL 中的令牌查询参数。 **注意：** 此令牌仅在 1 小时内有效。 此外，URL 会对来自浏览器的令牌值进行解码，然后再使用它。|
|  |  |  |
  

响应正文

``` json
{
    "id": "",
    "subscriptionName": "",
    "offerId": "",
    "planId": "",
}
```

| **参数名称** | **数据类型** | **说明**                       |
|--------------------|---------------|---------------------------------------|
| id                 | String        | SaaS 订阅的 ID。          |
| subscriptionName| String| 用户订阅 SaaS 服务时在 Azure 中设置的 SaaS 订阅名称。|
| OfferId            | String        | 用户订阅的套餐 ID。 |
| planId             | String        | 用户订阅的计划 ID。  |
|  |  |  |


响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                                         |
|----------------------|--------------------| --------------------------------------------------------------------------------------- |
| 200                  | `OK`                 | 已成功解析令牌。                                                            |
| 400                  | `BadRequest`         | 缺少必需的标头，或指定了无效的 api-version。 无法解析令牌，因为其中任意一个令牌格式不正确或已过期（令牌仅在生成后的 1 小时内有效）。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                                 |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                                |
| 503                  | `ServiceUnavailable` | 服务暂时关闭，请稍后重试。                                        |
|  |  |  |


响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 否           | 仅为 429 响应设置此值。                                                                   |
|  |  |  |


### <a name="subscribe"></a>订阅

订阅终结点允许用户开始订阅给定计划的 SaaS 服务，并在商务系统中启用计费。

**PUT**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions/*{subscriptionId}*?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| subscriptionId      | 通过解析 API 解析令牌后获取的 saas 订阅唯一 ID。                              |
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

|  **标头密钥**        | **必需** |  **说明**                                                  |
| ------------------     | ------------ | --------------------------------------------------------------------------------------- |
| x-ms-requestid         |   否         | 唯一的字符串值，用于跟踪来自客户端的请求，最好是 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| x-ms-correlationid     |   否         | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| If-Match/If-None-Match |   否         |   强验证程序 ETag 值。                                                          |
| content-type           |   是        |    `application/json`                                                                   |
|  authorization         |   是        |    JSON Web 令牌 (JWT) 持有者令牌。                                               |
| x-ms-marketplace-session-mode| 否 | 订阅 SaaS 套餐时用于启用试运行模式的标志。 如果已设置，则订阅是免费的。 此值可用于 ISV 测试方案。 请将其设置为 **‘dryrun’**|
|  |  |  |

*正文*

``` json
{
    "lanId": "",
}
```

| **元素名称** | **数据类型** | **说明**                      |
|------------------|---------------|--------------------------------------|
| planId           | （必需）字符串        | 用户订阅的 SaaS 服务的计划 ID。  |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                           |
|----------------------|--------------------|---------------------------------------------------------------------------|
| 202                  | `Accepted`           | 收到的给定计划的 SaaS 订阅激活码。                   |
| 400                  | `BadRequest`         | 缺少必需的标头，或 JSON 正文的格式不正确。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                   |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                  |
| 409                  | `Conflict`           | 订阅中正在执行另一项操作。                     |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                  |
| 503                  | `ServiceUnavailable` | 服务暂时关闭，请稍后重试。                          |
|  |  |  |

对于 202 响应，请跟踪“Operation-location”标头中的请求操作状态。 身份验证与其他市场 API 相同。

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 是          | 客户端检查状态的间隔。                                                       |
| Operation-Location | 是          | 链接到资源以获取操作状态。                                                        |
|  |  |  |

### <a name="change-plan-endpoint"></a>更改计划终结点

用户可以使用更改终结点将其当前已订阅的计划转换为新计划。

**PATCH**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions/*{subscriptionId}*?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| subscriptionId      | SaaS 订阅的 ID。                              |
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

| **标头密钥**          | **必需** | **说明**                                                                                                                                                                                                                  |
|-------------------------|--------------|---------------------------------------------------------------------------------------------------------------------|
| x-ms-requestid          | 否           | 唯一的字符串值，用于跟踪来自客户端的请求。 建议使用 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。   |
| x-ms-correlationid      | 否           | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| If-Match /If-None-Match | 否           | 强验证程序 ETag 值。                              |
| content-type            | 是          | `application/json`                                        |
| authorization           | 是          | JSON Web 令牌 (JWT) 持有者令牌。                    |
|  |  |  |

*正文*

```json
{
    "planId": ""
}
```

|  **元素名称** |  **数据类型**  | **说明**                              |
|  ---------------- | -------------   | --------------------------------------       |
|  planId           |  （必需）字符串         | 用户订阅的 SaaS 服务的计划 ID。          |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                           |
|----------------------|--------------------|---------------------------------------------------------------------------|
| 202                  | `Accepted`           | 收到的给定计划的 SaaS 订阅激活码。                   |
| 400                  | `BadRequest`         | 缺少必需的标头，或 JSON 正文的格式不正确。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                   |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                  |
| 409                  | `Conflict`           | 订阅中正在执行另一项操作。                     |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                  |
| 503                  | `ServiceUnavailable` | 服务暂时关闭，请稍后重试。                          |
|  |  |  |

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 是          | 客户端检查状态的间隔。                                                       |
| Operation-Location | 是          | 链接到资源以获取操作状态。                                                        |
|  |  |  |

### <a name="delete-subscription"></a>删除订阅

用户对订阅终结点执行 Delete 操作可以删除具有给定 ID 的订阅。

*请求*

**DELETE**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions/*{subscriptionId}*?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| subscriptionId      | SaaS 订阅的 ID。                              |
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

| **标头密钥**     | **必需** | **说明**                                                                                                                                                                                                                  |
|--------------------|--------------| ----------------------------------------------------------|
| x-ms-requestid     | 否           | 唯一的字符串值，用于跟踪来自客户端的请求。 建议使用 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。                                                           |
| x-ms-correlationid | 否           | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| authorization      | 是          | JSON Web 令牌 (JWT) 持有者令牌。                    |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                           |
|----------------------|--------------------|---------------------------------------------------------------------------|
| 202                  | `Accepted`           | 收到的给定计划的 SaaS 订阅激活码。                   |
| 400                  | `BadRequest`         | 缺少必需的标头，或 JSON 正文的格式不正确。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                   |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                  |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                  |
| 503                  | `ServiceUnavailable` | 服务暂时关闭。 请稍后重试。                          |
|  |  |  |

对于 202 响应，请跟踪“Operation-location”标头中的请求操作状态。 身份验证与其他市场 API 相同。

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 是          | 客户端检查状态的间隔。                                                       |
| Operation-Location | 是          | 链接到资源以获取操作状态。                                                        |
|   |  |  |

### <a name="get-operation-status"></a>获取操作状态

用户可以使用此终结点跟踪已触发的异步操作（订阅/取消订阅/更改计划）的状态。

*请求*

**GET**

**https://marketplaceapi.microsoft.com/api/saas/operations/*{operationId}*?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| operationId         | 触发的操作的唯一 ID。                |
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

| **标头密钥**     | **必需** | **说明**                                                                                                                                                                                                                  |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 否           | 唯一的字符串值，用于跟踪来自客户端的请求。 建议使用 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。   |
| x-ms-correlationid | 否           | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。  |
| authorization      | 是          | JSON Web 令牌 (JWT) 持有者令牌。                    |
|  |  |  | 

响应正文

```json
{
    "id": "",
    "status": "",
    "resourceLocation": "",
    "created": "",
    "lastModified": ""
}
```

| **参数名称** | **数据类型** | **说明**                                                                                                                                               |
|--------------------|---------------|-------------------------------------------------------------------------------------------|
| id                 | String        | 操作的 ID。                                                                      |
| status             | 枚举          | 操作状态，值为下列其中一项：`In Progress`、`Succeeded` 或 `Failed`。          |
| resourceLocation   | String        | 链接到已创建或修改的订阅。 这可以帮助客户端获取更新的状态发布操作。 不会对 `Unsubscribe` 操作设置此值。 |
| created            | DateTime      | 操作创建时间 (UTC)。                                                           |
| lastModified       | DateTime      | 上次更新操作的时间 (UTC)。                                                      |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                              |
|----------------------|--------------------|------------------------------------------------------------------------------|
| 200                  | `OK`                 | 已成功解析 Get 请求，且正文包含响应。    |
| 400                  | `BadRequest`         | 缺少必需的标头，或指定了无效的 api-version。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                      |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                     |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                     |
| 503                  | `ServiceUnavailable` | 服务暂时关闭，请稍后重试。                             |
|  |  |  |

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 是          | 客户端检查状态的间隔。                                                       |
|  |  |  |

### <a name="get-subscription"></a>获取订阅

用户对订阅终结点执行 Get 操作可以检索具有给定资源标识符的订阅。

*请求*

**GET**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions/*{subscriptionId}*?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| subscriptionId      | SaaS 订阅的 ID。                              |
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

| **标头密钥**     | **必需** | **说明**                                                                                           |
|--------------------|--------------|-----------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 否           | 唯一的字符串值，用于跟踪来自客户端的请求，最好是 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。                                                           |
| x-ms-correlationid | 否           | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| authorization      | 是          | JSON Web 令牌 (JWT) 持有者令牌。                                                                    |
|  |  |  |

响应正文

```json
{
    "id": "",
    "saasSubscriptionName": "",
    "offerId": "",
    "planId": "",
    "saasSubscriptionStatus": "",
    "created": "",
    "lastModified": ""
}
```

| **参数名称**     | **数据类型** | **说明**                               |
|------------------------|---------------|-----------------------------------------------|
| id                     | String        | Azure 中 SaaS 订阅资源的 ID。    |
| offerId                | String        | 用户订阅的套餐 ID。         |
| planId                 | String        | 用户订阅的计划 ID。          |
| saasSubscriptionName   | String        | SaaS 订阅的名称。                |
| saasSubscriptionStatus | 枚举          | 操作状态。  下列类型作之一：  <br/> - `Subscribed`：订阅处于活动状态。  <br/> - `Pending`：用户创建了资源，但 ISV 尚未激活该资源。   <br/> - `Unsubscribed`：用户已取消订阅。   <br/> - `Suspended`：用户已暂停订阅。   <br/> - `Deactivated`：Azure 订阅已暂停。  |
| created                | DateTime      | 订阅创建时间戳值 (UTC)。 |
| lastModified           | DateTime      | 订阅修改时间戳值 (UTC)。 |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                              |
|----------------------|--------------------|------------------------------------------------------------------------------|
| 200                  | `OK`                 | 已成功解析 Get 请求，且正文包含响应。    |
| 400                  | `BadRequest`         | 缺少必需的标头，或指定了无效的 api-version。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                      |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                     |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                     |
| 503                  | `ServiceUnavailable` | 服务暂时关闭，请稍后重试。                             |
|  |  |  |

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 否           | 客户端检查状态的间隔。                                                       |
| eTag               | 是          | 链接到资源以获取操作状态。                                                        |
|  |  |  |

### <a name="get-subscriptions"></a>获取订阅

用户对订阅终结点执行 Get 操作可以检索 ISV 提供的所有套餐的所有订阅。

*请求*

**GET**

**https://marketplaceapi.microsoft.com/api/saas/subscriptions?api-version=2017-04-15**

| **参数名称**  | **说明**                                       |
|---------------------|-------------------------------------------------------|
| api-version         | 用于此请求的操作的版本。 |
|  |  |

*标头*

| **标头密钥**     | **必需** | **说明**                                           |
|--------------------|--------------|-----------------------------------------------------------|
| x-ms-requestid     | 否           | 唯一的字符串值，用于跟踪来自客户端的请求。 建议使用 GUID。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。             |
| x-ms-correlationid | 否           | 在客户端上执行的操作的唯一字符串值。 此值用于将客户端操作生成的所有事件与服务器端的事件相关联。 如果未提供此值，则系统会生成一个值，并在响应标头中提供该值。 |
| authorization      | 是          | JSON Web 令牌 (JWT) 持有者令牌。                    |
|  |  |  |

响应正文

```json
{
    "id": "",
    "saasSubscriptionName": "",
    "offerId": "",
    "planId": "",
    "saasSubscriptionStatus": "",
    "created": "",
    "lastModified": ""
}
```

| **参数名称**     | **数据类型** | **说明**                               |
|------------------------|---------------|-----------------------------------------------|
| id                     | String        | Azure 中 SaaS 订阅资源的 ID。    |
| offerId                | String        | 用户订阅的套餐 ID。         |
| planId                 | String        | 用户订阅的计划 ID。          |
| saasSubscriptionName   | String        | SaaS 订阅的名称。                |
| saasSubscriptionStatus | 枚举          | 操作状态。  下列类型作之一：  <br/> - `Subscribed`：订阅处于活动状态。  <br/> - `Pending`：用户创建了资源，但 ISV 尚未激活该资源。   <br/> - `Unsubscribed`：用户已取消订阅。   <br/> - `Suspended`：用户已暂停订阅。   <br/> - `Deactivated`：Azure 订阅已暂停。  |
| created                | DateTime      | 订阅创建时间戳值 (UTC)。 |
| lastModified           | DateTime      | 订阅修改时间戳值 (UTC)。 |
|  |  |  |

响应代码

| **HTTP 状态代码** | **错误代码**     | **说明**                                                              |
|----------------------|--------------------|------------------------------------------------------------------------------|
| 200                  | `OK`                 | 已成功解析 Get 请求，且正文包含响应。    |
| 400                  | `BadRequest`         | 缺少必需的标头，或指定了无效的 api-version。 |
| 403                  | `Forbidden`          | 调用方无权执行此操作。                      |
| 404                  | `NotFound`           | 找不到具有给定 ID 的订阅                                     |
| 429                  | `RequestThrottleId`  | 服务正忙于处理请求，请稍后重试。                     |
| 503                  | `ServiceUnavailable` | 服务暂时关闭。 请稍后重试。                             |
|  |  |  |

响应标头

| **标头密钥**     | **必需** | **说明**                                                                                        |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------|
| x-ms-requestid     | 是          | 从客户端收到的请求 ID。                                                                   |
| x-ms-correlationid | 是          | 如果由客户端传递，则为关联 ID，否则此值为服务器关联 ID。                   |
| x-ms-activityid    | 是          | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。 |
| Retry-After        | 否           | 客户端检查状态的间隔。                                                       |
|  |  |  |

### <a name="saas-webhook"></a>SaaS Webhook

SaaS webhook 用于主动将更改通知给 SaaS 服务。 此 POST API 应当不进行身份验证并由 Microsoft 服务调用。 在对 webhook 通知采取操作之前，SaaS 服务应当调用操作 API 来进行验证和授权。 

*正文*

``` json
  { 
    "id": "be750acb-00aa-4a02-86bc-476cbe66d7fa",
    "activityId": "be750acb-00aa-4a02-86bc-476cbe66d7fa",
    "subscriptionId":"cd9c6a3a-7576-49f2-b27e-1e5136e57f45",
    "offerId": "sampleSaaSOffer", // Provided with "Update" action
    "publisherId": "contoso", 
    "planId": "silver",     // Provided with "Update" action
    "action": "Activate", // Activate/Delete/Suspend/Reinstate/Update
    "timeStamp": "2018-12-01T00:00:00"
  }
```

| **参数名称**     | **数据类型** | **说明**                               |
|------------------------|---------------|-----------------------------------------------|
| id  | String       | 触发的操作的唯一 ID。                |
| activityId   | String        | 唯一的字符串值，用于跟踪来自服务的请求。 此值用于任何核对操作。               |
| subscriptionId                     | String        | Azure 中 SaaS 订阅资源的 ID。    |
| offerId                | String        | 用户订阅的套餐 ID。 仅随“更新”操作提供。        |
| publisherId                | String        | SaaS 产品/服务的发布者 ID         |
| planId                 | String        | 用户订阅的计划 ID。 仅随“Update”操作提供。          |
| action                 | String        | 触发此通知的操作。 可能的值 - Activate、Delete、Suspend、Reinstate、Update          |
| timeStamp                 | String        | 此通知的触发时间的时间戳值（采用 UTC 格式）。          |
|  |  |  |
