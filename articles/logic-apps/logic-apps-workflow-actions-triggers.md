---
title: 工作流定义语言-Azure 逻辑应用中的触发器和操作类型的引用
description: 在 Azure 逻辑应用的工作流定义语言中的触发器和操作类型的参考指南
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
ms.topic: reference
ms.date: 05/06/2019
ms.openlocfilehash: 503bd6cfee1c19d2342ec9f535b3945178ab3ea0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65136602"
---
# <a name="reference-for-trigger-and-action-types-in-workflow-definition-language-for-azure-logic-apps"></a>为 Azure 逻辑应用工作流定义语言中的触发器和操作类型的引用

本参考文档介绍用于标识触发器和操作在逻辑应用的基础工作流定义中，这是所述，通过验证的常规类型[工作流定义语言](../logic-apps/logic-apps-workflow-definition-language.md)。
若要查找特定的连接器触发器和操作，可以在逻辑应用中使用，请参阅下的列表[连接器概述](https://docs.microsoft.com/connectors/)。

<a name="triggers-overview"></a>

## <a name="triggers-overview"></a>触发器概述

每个工作流包括定义所做的实例化并启动工作流调用的触发器。 以下为常规触发器类型：

* 轮询触发器 - 定期检查服务的终结点

* 推送触发器 - 创建终结点的订阅并提供回叫 URL，以便该终结点可在发生指定事件或数据可用时通知触发器。 然后触发器等待终结点的响应，接着才触发。 

触发器都具有以下顶级元素，但有一些是可选元素：  
  
```json
"<trigger-name>": {
   "type": "<trigger-type>",
   "inputs": { "<trigger-inputs>" },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "conditions": [ "<array-with-conditions>" ],
   "runtimeConfiguration": { "<runtime-config-options>" },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
},
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <trigger-name> | String | 触发器的名称 | 
| <trigger-type> | String | 触发器类型，例如“Http”或“ApiConnection” | 
| <trigger-inputs> | JSON 对象 | 定义触发器行为的输入 | 
| <time-unit> | String | 用于描述触发器触发频率的时间单位：“秒”、“分钟”、“小时”、“天”、“周”、“月” | 
| <number-of-time-units> | Integer | 指定触发器触发频率的值，即触发器再次触发之前需等待的时间单位数 <p>下面是最小和最大间隔： <p>- 月：1-16 个月 </br>- 天：1-500 天 </br>- 小时：1-12,000 小时 </br>- 分钟：1-72,000 分钟 </br>- 秒：1-9,999,999 秒<p>例如，如果间隔为 6，频率为“月”，则重复周期为每 6 个月。 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <array-with-conditions> | Array | 数组，其中包含一个或多个决定是否运行工作流的[条件](#trigger-conditions)。 仅适用于触发器。 | 
| <runtime-config-options> | JSON 对象 | 通过设置 `runtimeConfiguration` 属性可更改触发器运行时行为。 有关详细信息，请参阅[运行时配置设置](#runtime-config-options)。 | 
| <splitOn-expression> | String | 对于返回数组的触发器，可指定一个将数组项[拆分或解除批处理*到多个工作流实例进行处理的表达式*](#split-on-debatch)。 | 
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

## <a name="trigger-types-list"></a>触发器类型列表

每个触发器类型用于定义其行为的接口和输入不同。 

### <a name="built-in-triggers"></a>内置触发器

| 触发器类型 | 描述 | 
|--------------|-------------| 
| [**HTTP**](#http-trigger) | 检查或轮询任何终结点。 此终结点必须使用“202”异步模式或返回数组，符合特定的触发约定。 | 
| [**HTTPWebhook**](#http-webhook-trigger) | 为逻辑应用创建一个可调用的终结点，但调用指定的 URL 来注册或注销。 |
| [**Recurrence**](#recurrence-trigger) | 根据定义的计划执行。 可以设置在将来某个日期和时间执行此触发器。 根据频率，还可指定运行工作流的次数和天数。 | 
| [**Request**](#request-trigger)  | 为逻辑应用创建一个可调用的终结点，此类触发器也称为“手动”触发器。 相关示例请参阅[使用 HTTP 终结点调用、触发或嵌套工作流](../logic-apps/logic-apps-http-endpoint.md)。 | 
||| 

### <a name="managed-api-triggers"></a>托管的 API 触发器

| 触发器类型 | 描述 | 
|--------------|-------------| 
| [**ApiConnection**](#apiconnection-trigger) | 使用 [Microsoft 托管 API](../connectors/apis-list.md) 检查或轮询终结点。 | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-trigger) | 通过调用 [Microsoft 托管 API](../connectors/apis-list.md) 为逻辑应用创建可调用的终结点以进行订阅和取消订阅。 | 
||| 

## <a name="triggers---detailed-reference"></a>触发器 - 详细参考

<a name="apiconnection-trigger"></a>

### <a name="apiconnection-trigger"></a>APIConnection 触发器  

此触发器通过使用 [Microsoft 托管 API](../connectors/apis-list.md) 检查或轮询终结点，因此该触发器的参数可能基于终结点而有所不同。 此触发器定义中的许多部分是可选的。 触发器的行为取决于是否包含部分。

```json
"<APIConnection_trigger_name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": { "<retry-behavior>" },
      "queries": { "<query-parameters>" }
   },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <APIConnection_trigger_name> | String | 触发器的名称 | 
| <connection-name> | String | 工作流使用的托管 API 连接的名称 | 
| <method-type> | String | 与托管 API 通信的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”、“DELETE” | 
| <api-operation> | String | 要调用的 API 操作 | 
| <time-unit> | String | 用于描述触发器触发频率的时间单位：“秒”、“分钟”、“小时”、“天”、“周”、“月” | 
| <number-of-time-units> | Integer | 指定触发器触发频率的值，即触发器再次触发之前需等待的时间单位数 <p>下面是最小和最大间隔： <p>- 月：1-16 个月 </br>- 天：1-500 天 </br>- 小时：1-12,000 小时 </br>- 分钟：1-72,000 分钟 </br>- 秒：1-9,999,999 秒<p>例如，如果间隔为 6，频率为“月”，则重复周期为每 6 个月。 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <query-parameters> | JSON 对象 | 要包括在 API 调用中的任何查询参数。 例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <max-runs> | Integer | 默认情况下，工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <splitOn-expression> | String | 对于返回数组的触发器，此表达式引用要使用的数组，从而可为每个数组项创建和运行一个工作流实例，而不是使用“for each”循环。 <p>例如，此表达式表示触发器正文内容中返回的数组中的某一项：`@triggerbody()?['value']` |
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 |
||||

*输出*
 
| 元素 | Type | 描述 |
|---------|------|-------------|
| headers | JSON 对象 | 响应的标头 |
| body | JSON 对象 | 响应的正文 |
| 状态代码 | Integer | 响应中的状态代码 |
|||| 

*示例*

此触发器定义每天检查 Office 365 Outlook 帐户收件箱中的电子邮件： 

```json
"When_a_new_email_arrives": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "get",
      "path": "/Mail/OnNewEmail",
      "queries": {
          "fetchOnlyWithAttachment": false,
          "folderPath": "Inbox",
          "importance": "Any",
          "includeAttachments": false
      }
   },
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

<a name="apiconnectionwebhook-trigger"></a>

### <a name="apiconnectionwebhook-trigger"></a>ApiConnectionWebhook 触发器

此触发器使用 [Microsoft 托管 API](../connectors/apis-list.md) 向终结点发送订阅请求，提供此终结点可将响应发送到的回叫 URL，并等待终结点响应。 有关详细信息，请参阅[终结点订阅](#subscribe-unsubscribe)。

```json
"<ApiConnectionWebhook_trigger_name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
          "NotificationUrl": "@{listCallbackUrl()}"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "retryPolicy": { "<retry-behavior>" },
      "queries": "<query-parameters>"
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <connection-name> | String | 工作流使用的托管 API 连接的名称 | 
| <body-content> | JSON 对象 | 要作为有效负载发送到托管 API 的任何消息内容 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <query-parameters> | JSON 对象 | 要包括在 API 调用中的任何查询参数 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <max-runs> | Integer | 默认情况下，工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <splitOn-expression> | String | 对于返回数组的触发器，此表达式引用要使用的数组，从而可为每个数组项创建和运行一个工作流实例，而不是使用“for each”循环。 <p>例如，此表达式表示触发器正文内容中返回的数组中的某一项：`@triggerbody()?['value']` |
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

*示例*

此触发器定义订阅 Office 365 Outlook API，提供 API 终结点的回叫 URL，并在新邮件到达时等待终结点响应。

```json
"When_a_new_email_arrives_(webhook)": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
         "NotificationUrl": "@{listCallbackUrl()}" 
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "path": "/MailSubscription/$subscriptions",
      "queries": {
          "folderPath": "Inbox",
          "hasAttachment": "Any",
          "importance": "Any"
      }
   },
   "splitOn": "@triggerBody()?['value']"
}
```

<a name="http-trigger"></a>

### <a name="http-trigger"></a>HTTP 触发器

此触发器基于指定的重复计划检查或轮询指定的终结点。 终结点的响应决定工作流是否运行。

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<endpoint-URL>",
      "headers": { "<header-content>" },
      "body": "<body-content>",
      "authentication": { "<authentication-method>" },
      "retryPolicy": { "<retry-behavior>" },
      "queries": "<query-parameters>"
   },
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <method-type> | String | 用于轮询指定终结点的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”、“DELETE” | 
| <endpoint-URL> | String | 要轮询的终结点的 HTTP 或 HTTPS URL <p>最大字符串大小：2 KB | 
| <time-unit> | String | 用于描述触发器触发频率的时间单位：“秒”、“分钟”、“小时”、“天”、“周”、“月” | 
| <number-of-time-units> | Integer | 指定触发器触发频率的值，即触发器再次触发之前需等待的时间单位数 <p>下面是最小和最大间隔： <p>- 月：1-16 个月 </br>- 天：1-500 天 </br>- 小时：1-12,000 小时 </br>- 分钟：1-72,000 分钟 </br>- 秒：1-9,999,999 秒<p>例如，如果间隔为 6，频率为“月”，则重复周期为每 6 个月。 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <header-content> | JSON 对象 | 与请求一同发送的标头 <p>例如，设置请求的语言和类型： <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <body-content> | String | 要作为有效负载与请求一同发送的消息内容 | 
| <authentication-method> | JSON 对象 | 请求用于身份验证的方法。 有关详细信息，请参阅[计划程序出站身份验证](../scheduler/scheduler-outbound-authentication.md)。 除计划程序外，还支持 `authority` 属性。 如果未指定此值，则使用默认值 `https://login.windows.net`，但也可使用其他值，例如 `https://login.windows\-ppe.net`。 |
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 |  
 <query-parameters> | JSON 对象 | 要包括在请求中的任何查询参数 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到请求。 | 
| <max-runs> | Integer | 默认情况下，工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

*输出*

| 元素 | Type | 描述 |
|---------|------|-------------| 
| headers | JSON 对象 | 响应的标头 | 
| body | JSON 对象 | 响应的正文 | 
| 状态代码 | Integer | 响应中的状态代码 | 
|||| 

传入请求的要求

为很好地配合逻辑应用进行工作，终结点必须符合特定触发器模式或协定，并识别以下属性：  
  
| 响应 | 需要 | 描述 | 
|----------|----------|-------------| 
| 状态代码 | 是 | “200 OK”状态代码启动运行。 其他任何状态代码均不会启动运行。 | 
| 重试间隔标头 | 否 | 逻辑应用再次轮询终结点之前的秒数 | 
| Location 标头 | 否 | 在下一个轮询间隔要调用的 URL。 如果未指定，将使用原始 URL。 | 
|||| 

不同请求的示例行为

| 状态代码 | 重试间隔 | 行为 | 
|-------------|-------------|----------|
| 200 | {无} | 运行工作流，然后在定义的重复周期后再次检查是否有更多数据。 | 
| 200 | 10 秒 | 运行工作流，然后在 10 秒后再次检查是否有更多数据。 |  
| 202 | 60 秒 | 不会触发工作流。 下一次尝试将在一分钟后发生，也需要遵循定义的重复周期。 如果定义的重复周期不到一分钟，重试间隔标头优先。 否则，将使用定义的重复周期。 | 
| 400 | {无} | 错误的请求，不会运行工作流。 如果未定义 `retryPolicy`，将使用默认策略。 在达到重试次数后，触发器会在定义的重复周期后再次检查是否有数据。 | 
| 500 | {无}| 服务器错误，不会运行工作流。 如果未定义 `retryPolicy`，将使用默认策略。 在达到重试次数后，触发器会在定义的重复周期后再次检查是否有数据。 | 
|||| 

<a name="http-webhook-trigger"></a>

### <a name="httpwebhook-trigger"></a>HTTPWebhook 触发器  

此触发器创建一个可通过调用指定终结点 URL 来注册订阅的终结点，使逻辑应用可被调用。 在工作流中创建此触发器时，传出请求会进行调用以注册订阅。 这样，该触发器便可开始侦听事件。 当某个操作使该触发器无效时，传出请求会自动进行调用以取消订阅。 有关详细信息，请参阅[终结点订阅](#subscribe-unsubscribe)。

此外，还可对 HTTP Webhook 触发器指定[异步限制](#asynchronous-limits)。
该触发器的行为取决于使用或省略的部分。 

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<endpoint-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "retryPolicy": { "<retry-behavior>" }
         },
      },
      "unsubscribe": {
         "method": "<method-type>",
         "url": "<endpoint-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" }
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

某些值对 `"subscribe"` 和 `"unsubscribe"` 对象均可用，例如 <method-type>。

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <method-type> | String | 用于订阅请求的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” | 
| <endpoint-subscribe-URL> | String | 要将订阅请求发送到的终结点 URL | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <method-type> | String | 用于取消请求的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” | 
| <endpoint-unsubscribe-URL> | String | 要将取消请求发送到的终结点 URL | 
| <body-content> | String | 要在订阅请求或取消订阅请求中发送的任何消息内容 | 
| <authentication-method> | JSON 对象 | 请求用于身份验证的方法。 有关详细信息，请参阅[计划程序出站身份验证](../scheduler/scheduler-outbound-authentication.md)。 |
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <max-runs> | Integer | 默认情况下，所有工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

*输出* 

| 元素 | Type | 描述 |
|---------|------|-------------| 
| headers | JSON 对象 | 响应的标头 | 
| body | JSON 对象 | 响应的正文 | 
| 状态代码 | Integer | 响应中的状态代码 | 
|||| 

*示例*

此触发器创建指定终结点的订阅，提供唯一的回叫 URL，并等待新发布的技术文章。

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "POST",
         "uri": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{listCallbackUrl()}",
            "hub.mode": "subscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         },
      },
      "unsubscribe": {
         "method": "POST",
         "url": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{workflow().endpoint}@{listCallbackUrl()}",
            "hub.mode": "unsubscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         }
      }
   }
}
```

<a name="recurrence-trigger"></a>

### <a name="recurrence-trigger"></a>重复触发器  

此触发器的运行基于指定的重复计划，借助它可轻松创建定期运行的工作流。 

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
      "startTime": "<start-date-time-with-format-YYYY-MM-DDThh:mm:ss>",
      "timeZone": "<time-zone>",
      "schedule": {
         // Applies only when frequency is Day or Week. Separate values with commas.
         "hours": [ <one-or-more-hour-marks> ], 
         // Applies only when frequency is Day or Week. Separate values with commas.
         "minutes": [ <one-or-more-minute-marks> ], 
         // Applies only when frequency is Week. Separate values with commas.
         "weekDays": [ "Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday" ] 
      }
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <time-unit> | String | 用于描述触发器触发频率的时间单位：“秒”、“分钟”、“小时”、“天”、“周”、“月” | 
| <number-of-time-units> | Integer | 指定触发器触发频率的值，即触发器再次触发之前需等待的时间单位数 <p>下面是最小和最大间隔： <p>- 月：1-16 个月 </br>- 天：1-500 天 </br>- 小时：1-12,000 小时 </br>- 分钟：1-72,000 分钟 </br>- 秒：1-9,999,999 秒<p>例如，如果间隔为 6，频率为“月”，则重复周期为每 6 个月。 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <start-date-time-with-format-YYYY-MM-DDThh:mm:ss> | String | 采用以下格式的启动日期和时间： <p>如果指定时区，则为 YYYY-MM-DDThh:mm:ss <p>-或- <p>如果不指定时区，则为 YYYY-MM-DDThh:mm:ssZ <p>例如，如果需要 2017 年 9 月 18 日下午 2:00，则指定“2017-09-18T14:00:00”并指定时区（如“太平洋标准时间”），或仅指定“2017-09-18T14:00:00Z”，而不指定时区。 <p>**注意：** 此开始时间必须遵循 [UTC 日期时间格式](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)中的 [ISO 8601 日期时间规范](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)，但没有 [UTC 时差](https://en.wikipedia.org/wiki/UTC_offset)。 如果未指定时区，则必须在末尾添加字母“Z”（无空格）。 这个“Z”指等效的[航海时间](https://en.wikipedia.org/wiki/Nautical_time)。 <p>对于简单计划，启动时间即第一次循环；而对于复杂计划，触发器不会在启动时间之前执行。 有关启动日期和时间的详细信息，请参阅[创建和计划定期运行任务](../connectors/connectors-native-recurrence.md)。 | 
| <time-zone> | String | 仅当指定启动时间时才适用，因为此触发器不接受 [UTC 时差](https://en.wikipedia.org/wiki/UTC_offset)。 指定要应用的时区。 | 
| <one-or-more-hour-marks> | 整数或整数数组 | 如果为 `frequency` 指定“Day”或“Week”，可以从 0 到 23 范围内指定一个或多个整数（用逗号分隔），作为一天中要运行工作流的时间点。 <p>例如，如果指定“10”、“12”和“14”，则会将上午 10 点、中午 12 点和下午 2 点作为小时标记。 | 
| <one-or-more-minute-marks> | 整数或整数数组 | 如果为 `frequency` 指定“Day”或“Week”，可以从 0 到 59 范围内指定一个或多个整数（用逗号分隔），作为要运行工作流的分钟。 <p>例如，可以指定“30”作为分钟标记并使用前面示例中的当天小时时间，这样，便可以指定10:30 AM、12:30 PM 和 2:30 PM 作为开始时间。 | 
| 工作日 | 字符串或字符串数组 | 如果 `frequency` 指定为“周”，则可以指定一天或多天（用逗号分隔）作为运行工作流的时间：“星期一”、“星期二”、“星期三”、“星期四”、“星期五”、“星期六”和“星期日” | 
| <max-runs> | Integer | 默认情况下，所有工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

*示例 1*

此基本重复触发器每日都会运行：

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

*示例 2*

可以指定激发触发器的启动日期和时间。 此重复触发器在指定日期启动，然后每日触发：

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1,
      "startTime": "2017-09-18T00:00:00Z"
   }
}
```

*示例 3*

此重复触发器在 2017 年 9 月 9 日下午 2:00 启动，并在每周一上午 10:30、中午 12:30 和下午 2:30（太平洋标准时间）触发：

``` json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "hours": [ 10, 12, 14 ],
         "minutes": [ 30 ],
         "weekDays": [ "Monday" ]
      },
      "startTime": "2017-09-07T14:00:00",
      "timeZone": "Pacific Standard Time"
   }
}
```

有关此触发器的详细信息和示例，请参阅[创建和计划定期运行任务](../connectors/connectors-native-recurrence.md)。

<a name="request-trigger"></a>

### <a name="request-trigger"></a>请求触发器

该触发器通过创建可接受传入请求的终结点，使逻辑应用可被调用。 对于此终结点，提供一个用于描述和验证触发器从传入请求接收的有效负载或输入的 JSON 架构。 此架构还使触发器属性更易于从工作流的后续操作中引用。 

若要调用此触发器，必须使用 `listCallbackUrl` API，[工作流服务 REST API](https://docs.microsoft.com/rest/api/logic/workflows) 中对此 API 进行了说明。 若要了解如何将此触发器用作 HTTP 终结点，请参阅[使用 HTTP 终结点调用、触发或嵌套工作流](../logic-apps/logic-apps-http-endpoint.md)。

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "<method-type>",
      "relativePath": "<relative-path-for-accepted-parameter>",
      "schema": {
         "type": "object",
         "properties": { 
            "<property-name>": {
               "type": "<property-type>"
            }
         },
         "required": [ "<required-properties>" ]
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      },
   },
   "operationOptions": "<operation-option>"
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <property-name> | String | JSON 架构中属性的名称，描述有效负载 | 
| <property-type> | String | 属性的类型 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <method-type> | String | 传入请求必须用以调用逻辑应用的方法：“GET”、“PUT”、“POST”、“PATCH”、“DELETE” |
| <relative-path-for-accepted-parameter> | String | 终结点的 URL 可接受的参数的相对路径 | 
| <required-properties> | Array | 需要值的一个或多个属性 | 
| <max-runs> | Integer | 默认情况下，所有工作流实例运行在同一时间，或并行达[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要通过设置新的 <count> 值更改此限制，请参阅[更改触发器并发](#change-trigger-concurrency)。 | 
| <max-runs-queue> | Integer | 如果您的工作流已在运行的最大实例数，你可以基于`runtimeConfiguration.concurrency.runs`属性，任何新的运行被放入此队列[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 
| <operation-option> | String | 通过设置 `operationOptions` 属性可更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

*示例*

此触发器指定传入请求必须使用 HTTP POST 方法调用触发器，并且包含验证传入请求输入的架构： 

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "POST",
      "schema": {
         "type": "object",
         "properties": {
            "customerName": {
               "type": "String"
            },
            "customerAddress": { 
               "type": "Object",
               "properties": {
                  "streetAddress": {
                     "type": "string"
                  },
                  "city": {
                     "type": "string"
                  }
               }
            }
         }
      }
   }
}
```

<a name="trigger-conditions"></a>

## <a name="trigger-conditions"></a>触发条件

对于任何触发器且仅对于触发器，可包括一个数组，其中包含一个或多个决定工作流是否应该运行的条件表达式。 若要添加`conditions`属性设置为工作流中的触发器代码视图编辑器中打开逻辑应用。

例如，可通过引用 `conditions` 属性中触发器的状态代码，指定触发器仅在站点返回内部服务器错误时触发：

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Hour",
      "interval": 1
   },
   "conditions": [ {
      "expression": "@equals(triggers().code, 'InternalServerError')"
   } ]
}
```

默认情况下，触发器仅在获得“200 OK”响应后触发。 当表达式引用触发器的状态代码时，将替换触发器的默认行为。 因此，若要触发器针对多个状态代码（例如状态代码“200”和“201”）触发，必须包含此表达式作为条件： 

`@or(equals(triggers().code, 200),equals(triggers().code, 201))` 

<a name="split-on-debatch"></a>

## <a name="trigger-multiple-runs"></a>触发多个运行

触发器可能会返回一个可供逻辑应用处理的数组，但有时候，“for each”循环可能会花过长的时间来处理每个数组项。 此时可改用触发器中的 **SplitOn** 属性，对数组执行解除批处理操作。 解除批处理拆分数组项并开始运行每个数组项的新工作流实例。 多种情况下可以使用此方法。例如，需要轮询一个终结点，而该终结点可能在不同的轮询间隔期之间返回多个新项。
若要了解 **SplitOn** 在单个逻辑应用运行中可以处理的最大数组项数，请参阅[限制和配置](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 

> [!NOTE]
> 无法对同步响应模式使用 SplitOn。 任何使用 **SplitOn** 并包括一个响应操作的工作流都会异步运行并即时发送 `202 ACCEPTED` 响应。

如果触发器的 Swagger 文件描述的有效负载是一个数组，则会自动向触发器添加 **SplitOn** 属性。 否则，请将此属性添加到其数组需要解除批处理的响应有效负载中。 

*示例*

假设某个 API 返回以下响应： 
  
```json
{
   "Status": "Succeeded",
   "Rows": [ 
      { 
         "id": 938109380,
         "name": "customer-name-one"
      },
      {
         "id": 938109381,
         "name": "customer-name-two"
      }
   ]
}
```
 
逻辑应用只需要 `Rows` 中的数组的内容，因此，可以按以下示例所示创建触发器：

``` json
"HTTP_Debatch": {
   "type": "Http",
    "inputs": {
        "uri": "https://mydomain.com/myAPI",
        "method": "GET"
    },
   "recurrence": {
      "frequency": "Second",
      "interval": 1
    },
    "splitOn": "@triggerBody()?.Rows"
}
```

> [!NOTE]
> 如果使用 `SplitOn` 命令，则无法获取数组外部的属性。 因此，就此示例来说，不能在从 API 返回的响应中获取 `status` 属性。
> 
> 为了避免在不存在 `Rows` 属性的情况下发生故障，本示例使用了 `?` 运算符。

工作流定义现在可以使用 `@triggerBody().name` 获取 `name` 值，即第一个运行中的 `"customer-name-one"` 和第二个运行中的 `"customer-name-two"`。 因此，触发器的输出如以下示例所示：

```json
{
   "body": {
      "id": 938109380,
      "name": "customer-name-one"
   }
}
```

```json
{
   "body": {
      "id": 938109381,
      "name": "customer-name-two"
   }
}
```

<a name="actions-overview"></a>

## <a name="actions-overview"></a>操作概述

Azure 逻辑应用提供多种操作类型，每个类型均具有定义操作的唯一行为的不同输入。 

操作具有以下高级元素，但有一些是可选元素：

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": { 
      "<input-name>": { "<input-value>" },
      "retryPolicy": "<retry-behavior>" 
   },
   "runAfter": { "<previous-trigger-or-action-status>" },
   "runtimeConfiguration": { "<runtime-config-options>" },
   "operationOptions": "<operation-option>"
},
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------|
| <action-name> | String | 操作的名称 | 
| <action-type> | String | 操作类型，例如“Http”或“ApiConnection”| 
| <input-name> | String | 定义操作行为的输入的名称 | 
| <input-value> | 各种 | 输入值，可为字符串、整数、JSON 对象等 | 
| <previous-trigger-or-action-status> | JSON 对象 | 在此当前操作可以运行之前，必须立即运行的触发器或操作的名称和结果状态 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------|
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅“重试策略”。 | 
| <runtime-config-options> | JSON 对象 | 对于某些操作，可通过设置 `runtimeConfiguration` 属性在运行时更改操作的行为。 有关详细信息，请参阅[运行时配置设置](#runtime-config-options)。 | 
| <operation-option> | String | 对于某些操作，可通过设置 `operationOptions` 属性更改默认行为。 有关详细信息，请参阅[操作选项](#operation-options)。 | 
|||| 

## <a name="action-types-list"></a>操作类型列表

以下为一些常用操作类型： 

* [内置操作类型](#built-in-actions)，例如以下示例等： 

  * 用于通过 HTTP 或 HTTPS 调用终结点的 [HTTP](#http-action)

  * 用于响应请求的 [Response](#response-action)

  * [**执行的 JavaScript 代码**](#run-javascript-code)运行 JavaScript 代码片段

  * 用于调用 Azure Functions 的 [Function](#function-action)

  * 数据操作操作，例如 [Join](#join-action)、[Compose](#compose-action)、[Table](#table-action)、[Select](#select-action) 以及其他从各种输入创建或转换数据的操作

  * 用于调用另一个逻辑应用工作流的 [Workflow](#workflow-action)

* [托管的 API 操作类型](#managed-api-actions)，例如调用由 Microsoft 托管的各种连接器和 API（例如 Azure 服务总线、Office 365 Outlook、Power BI、Azure Blob 存储、OneDrive 和 GitHub 等）的 [ApiConnection](#apiconnection-action) 和 [ApiConnectionWebHook](#apiconnectionwebhook-action)

* 包含其他操作且有助于整理工作流执行的[控制工作流操作类型](#control-workflow-actions)，例如 [If](#if-action)、[Foreach](#foreach-action)、[Switch](#switch-action)、[Scope](#scope-action) 和 [Until](#until-action)

<a name="built-in-actions"></a>

### <a name="built-in-actions"></a>内置操作

| 操作类型 | 描述 | 
|-------------|-------------| 
| [Compose](#compose-action) | 从输入创建单个输出，可具有多种类型。 | 
| [**执行的 JavaScript 代码**](#run-javascript-code) | 运行符合特定条件的 JavaScript 代码片段。 有关代码要求和的详细信息，请参阅[添加和运行的代码段的内联代码](../logic-apps/logic-apps-add-run-inline-code.md)。 |
| [Function](#function-action) | 调用 Azure Function。 | 
| [**HTTP**](#http-action) | 调用 HTTP 终结点。 | 
| [Join](#join-action) | 基于数组中的所有项创建一个字符串，并使用指定的分隔符字符分隔这些项。 | 
| [Parse JSON](#parse-json-action) | 基于 JSON 内容中的属性创建用户友好型令牌。 然后可通过将令牌包含在逻辑应用中来引用这些属性。 | 
| [Query](#query-action) | 基于条件或筛选器使用另一个数组中的项创建数组。 | 
| [Response](#response-action) | 创建针对传入调用或请求的响应。 | 
| [Select](#select-action) | 通过基于指定映射转换另一个数组中的项，使用 JSON 对象创建数组。 | 
| [Table](#table-action) | 根据数组创建 CSV 或 HTML 表。 | 
| [Terminate](#terminate-action) | 停止正在主动运行的工作流。 | 
| [Wait](#wait-action) | 将工作流暂停指定的时间段或暂停到指定日期和时间。 | 
| [Workflow](#workflow-action) | 将一个工作流嵌套在另一个工作流内。 | 
||| 

<a name="managed-api-actions"></a>

### <a name="managed-api-actions"></a>托管的 API 操作

| 操作类型 | 描述 | 
|-------------|-------------|  
| [**ApiConnection**](#apiconnection-action) | 使用 [Microsoft 托管 API](../connectors/apis-list.md) 调用 HTTP 终结点。 | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-action) | 工作方式类似于 HTTP Webhook，但使用 [Microsoft 托管 API](../connectors/apis-list.md)。 | 
||| 

<a name="control-workflow-actions"></a>

### <a name="control-workflow-actions"></a>控制工作流操作

这些操作有助于控制工作流执行且包含其他操作。 从某一控制工作流操作外部，可直接引用该控制工作流操作内的操作。 例如，假如你在某一范围内拥有一个 `Http` 操作，则可从此工作流任意位置引用 `@body('Http')` 表达式。 但是，控制工作流操作内部存在的操作仅可在相同控制工作流结构中的其他操作“之后运行”。

| 操作类型 | 描述 | 
|-------------|-------------| 
| [ForEach](#foreach-action) | 在循环中对数组中的每个项执行相同的操作。 | 
| [If](#if-action) | 基于指定条件为 true 还是为 false 来运行操作。 | 
| [Scope](#scope-action) | 基于组状态从一组操作中运行操作。 | 
| [Switch](#switch-action) | 当表达式、对象或令牌的值匹配各事例指定的值时，运行被组织为事例的操作。 | 
| [Until](#until-action) | 在循环中运行操作，直至指定条件为 true。 | 
|||  

## <a name="actions---detailed-reference"></a>操作 - 详细参考

<a name="apiconnection-action"></a>

### <a name="apiconnection-action"></a>APIConnection 操作

此操作将 HTTP 请求发送到 [Microsoft 托管 API](../connectors/apis-list.md)，且需要有关该 API 和参数的信息以及对有效连接的引用。 

``` json
"<action-name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<api-name>']['connectionId']"
         },
         "<other-action-specific-input-properties>"        
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": "<retry-behavior>",
      "queries": { "<query-parameters>" },
      "<other-action-specific-properties>"
    },
    "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <action-name> | String | 连接器提供的操作的名称 | 
| <api-name> | String | 用于连接的 Microsoft 托管 API 的名称 | 
| <method-type> | String | 用于调用 API 的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” | 
| <api-operation> | String | 要调用的 API 操作 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <other-action-specific-input-properties> | JSON 对象 | 应用于此指定操作的任何其他输入属性 | 
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <query-parameters> | JSON 对象 | 要包括在 API 调用中的任何查询参数。 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <other-action-specific-properties> | JSON 对象 | 应用于此指定操作的任何其他属性 | 
|||| 

*示例*

此定义演示 Office 365 Outlook 连接器（属于 Microsoft 托管 API）的“发送电子邮件”操作： 

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "Thank you for your membership!",
         "Subject": "Hello and welcome!",
         "To": "Sophie.Owen@contoso.com"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "POST",
      "path": "/Mail"
    },
    "runAfter": {}
}
```

<a name="apiconnection-webhook-action"></a>

### <a name="apiconnectionwebhook-action"></a>APIConnectionWebhook 操作

此操作使用 [Microsoft 托管 API](../connectors/apis-list.md) 通过 HTTP 向终结点发送订阅请求，提供此终结点可将响应发送到的回叫 URL，并等待终结点响应。 有关详细信息，请参阅[终结点订阅](#subscribe-unsubscribe)。

```json
"<action-name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<api-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "retryPolicy": "<retry-behavior>",
         "queries": { "<query-parameters>" },
         "<other-action-specific-input-properties>"
      },
      "unsubscribe": {
         "method": "<method-type>",
         "uri": "<api-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "<other-action-specific-properties>"
      },
   },
   "runAfter": {}
}
```

某些值对 `"subscribe"` 和 `"unsubscribe"` 对象均可用，例如 <method-type>。

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <action-name> | String | 连接器提供的操作的名称 | 
| <method-type> | String | 用于从终结点订阅或取消订阅的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” | 
| <api-subscribe-URL> | String | 用于订阅 API 的 URI | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <api-unsubscribe-URL> | String | 用于取消订阅 API 的 URI | 
| <header-content> | JSON 对象 | 请求中发送的任何标头 <p>例如，在请求中设置语言和类型： <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <body-content> | JSON 对象 | 请求中发送的任何消息内容 | 
| <authentication-method> | JSON 对象 | 请求用于身份验证的方法。 有关详细信息，请参阅[计划程序出站身份验证](../scheduler/scheduler-outbound-authentication.md)。 |
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <query-parameters> | JSON 对象 | 要包括在 API 调用中的任何查询参数 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <other-action-specific-input-properties> | JSON 对象 | 应用于此指定操作的任何其他输入属性 | 
| <other-action-specific-properties> | JSON 对象 | 应用于此指定操作的任何其他属性 | 
|||| 

还可像指定 [HTTP 异步限制](#asynchronous-limits)一样，针对 ApiConnectionWebhook 操作指定限制。

<a name="compose-action"></a>

### <a name="compose-action"></a>Compose 操作

此操作基于多个输入（包括表达式）创建一个单一输出。 输出和输入均可具有 Azure 逻辑应用本机支持的任何类型，例如数组、JSON 对象、XML 和二进制文件。
此操作的输出之后可用于其他操作。 

```json
"Compose": {
   "type": "Compose",
   "inputs": "<inputs-to-compose>",
   "runAfter": {}
},
```

*必需* 

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <inputs-to-compose> | 任意 | 用于创建一个单一输出的输入 | 
|||| 

*示例 1*

<!-- markdownlint-disable MD038 -->
此操作定义合并 `abcdefg ` 与尾随空格和值 `1234`：
<!-- markdownlint-enable MD038 -->

```json
"Compose": {
   "type": "Compose",
   "inputs": "abcdefg 1234",
   "runAfter": {}
},
```

以下为此操作创建的输出：

`abcdefg 1234`

*示例 2*

此操作定义合并包含 `abcdefg` 的字符串变量与包含 `1234` 的整数变量：

```json
"Compose": {
   "type": "Compose",
   "inputs": "@{variables('myString')}@{variables('myInteger')}",
   "runAfter": {}
},
```

以下为此操作创建的输出：

`"abcdefg1234"`

<a name="run-javascript-code"></a>

### <a name="execute-javascript-code-action"></a>执行的 JavaScript 代码操作

此操作运行的 JavaScript 代码片段，通过将结果返回`Result`更高版本的操作可以引用的令牌。

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "<JavaScript-code-snippet>",
      "explicitDependencies": {
         "actions": [ <previous-actions> ],
         "includeTrigger": true
      }
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 |
|-------|------|-------------|
| <*JavaScript-code-snippet*> | 多种多样 | 你想要运行的 JavaScript 代码。 有关代码要求和的详细信息，请参阅[添加和运行的代码段的内联代码](../logic-apps/logic-apps-add-run-inline-code.md)。 <p>在中`code`属性，您的代码段可以使用只读的`workflowContext`对象作为输入。 此对象具有子属性，允许代码访问到结果中的触发器和工作流中的上一操作。 有关详细信息`workflowContext`对象，请参阅[引用在代码中的触发器和操作结果](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext)。 |
||||

*在某些情况下所需*

`explicitDependencies`属性指定你想要显式包括触发器、 上一操作，或者二者均为你的代码片段的依赖项中的结果。 有关添加这些依赖关系的详细信息，请参阅[内联代码的参数添加](../logic-apps/logic-apps-add-run-inline-code.md#add-parameters)。 

有关`includeTrigger`特性，可以指定`true`或`false`值。

| 值 | Type | 描述 |
|-------|------|-------------|
| <*previous-actions*> | 字符串数组 | 使用指定的操作名称数组。 使用工作流定义中出现操作名称在其中使用下划线 (_)，而不是空格的操作名称 ("")。 |
||||

*示例 1*

此操作将运行代码，以便获取在逻辑应用的名称作为结果返回的文本"Hello world 从 < 逻辑应用名称 >"。 在此示例中，该代码引用工作流的名称通过访问`workflowContext.workflow.name`通过只读属性`workflowContext`对象。 有关使用详细信息`workflowContext`对象，请参阅[引用在代码中的触发器和操作结果](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext)。

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var text = \"Hello world from \" + workflowContext.workflow.name;\r\n\r\nreturn text;"
   },
   "runAfter": {}
}
```

*示例 2*

此操作中的 Office 365 Outlook 帐户到达新的电子邮件时触发的逻辑应用运行代码。 逻辑应用还使用将内容转发从批准的请求以及接收电子邮件发送审批电子邮件操作。 

代码的触发器中提取电子邮件地址`Body`属性，并返回与这些电子邮件地址`SelectedOption`属性值从审批操作。 操作显式包括发送审批电子邮件操作中的依赖项作为`explicitDependencies`  >  `actions`属性。

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var re = /(([^<>()\\[\\]\\\\.,;:\\s@\"]+(\\.[^<>()\\[\\]\\\\.,;:\\s@\"]+)*)|(\".+\"))@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))/g;\r\n\r\nvar email = workflowContext.trigger.outputs.body.Body;\r\n\r\nvar reply = workflowContext.actions.Send_approval_email_.outputs.body.SelectedOption;\r\n\r\nreturn email.match(re) + \" - \" + reply;\r\n;",
      "explicitDependencies": {
         "actions": [
            "Send_approval_email_"
         ]
      }
   },
   "runAfter": {}
}
```



<a name="function-action"></a>

### <a name="function-action"></a>函数操作

此操作调用先前创建的 [Azure 函数](../azure-functions/functions-create-first-azure-function.md)。

```json
"<Azure-function-name>": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "<Azure-function-ID>"
      },
      "method": "<method-type>",
      "headers": { "<header-content>" },
      "body": { "<body-content>" },
      "queries": { "<query-parameters>" } 
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------|  
| <Azure-function-ID> | String | 要调用的 Azure 函数的资源 ID。 下面是此值的格式：<p>“/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group>/providers/Microsoft.Web/sites/<Azure-function-app-name>/functions/<Azure-function-name>” | 
| <method-type> | String | 用于调用函数的 HTTP 方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” <p>如果未指定，则默认方法为“POST”。 | 
||||

可选

| 值 | Type | 描述 | 
|-------|------|-------------|  
| <header-content> | JSON 对象 | 与调用一同发送的任何标头 <p>例如，在请求中设置语言和类型： <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <body-content> | JSON 对象 | 请求中发送的任何消息内容 | 
| <query-parameters> | JSON 对象 | 要包括在 API 调用中的任何查询参数 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <other-action-specific-input-properties> | JSON 对象 | 应用于此指定操作的任何其他输入属性 | 
| <other-action-specific-properties> | JSON 对象 | 应用于此指定操作的任何其他属性 | 
||||

保存逻辑应用时，逻辑应用引擎会对所引用的函数执行下述检查：

* 工作流必须具有该函数的访问权限。

* 工作流只能使用标准 HTTP 触发器或泛型 JSON Webhook 触发器。 

  逻辑应用引擎获取并缓存在运行时使用的触发器 URL。 但是，如果任何操作使缓存的 URL 失效，则此 Function 操作会在运行时失败。 若要解决此问题，请再次保存逻辑应用，以便逻辑应用再次获取和缓存此触发器 URL。

* 函数不能定义任何路由。

* 仅允许“函数”和“匿名”授权级别。 

*示例*

此操作定义调用先前创建的“GetProductID”函数：

```json
"GetProductID": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "/subscriptions/<XXXXXXXXXXXXXXXXXXXX>/resourceGroups/myLogicAppResourceGroup/providers/Microsoft.Web/sites/InventoryChecker/functions/GetProductID"
      },
      "method": "POST",
      "headers": { 
          "x-ms-date": "@utcnow()"
       },
      "body": { 
          "Product_ID": "@variables('ProductID')"
      }
   },
   "runAfter": {}
}
```

<a name="http-action"></a>

### <a name="http-action"></a>HTTP 操作

此操作向指定终结点发送请求并检查响应，确定是否应运行工作流。 

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<HTTP-or-HTTPS-endpoint-URL>"
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <method-type> | String | 用于发送请求的方法：“GET”、“PUT”、“POST”、“PATCH”或“DELETE” | 
| <HTTP-or-HTTPS-endpoint-URL> | String | 要调用的 HTTP 或 HTTPS 终结点。 最大字符串大小：2 KB | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <header-content> | JSON 对象 | 与请求一同发送的任何标头 <p>例如，设置语言和类型： <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <body-content> | JSON 对象 | 请求中发送的任何消息内容 | 
| <retry-behavior> | JSON 对象 | 自定义状态代码为 408、429 和 5XX 的间歇性故障以及任何连接异常的重试行为。 有关详细信息，请参阅[重试策略](../logic-apps/logic-apps-exception-handling.md#retry-policies)。 | 
| <query-parameters> | JSON 对象 | 要包括在请求中的任何查询参数 <p>例如，`"queries": { "api-version": "2018-01-01" }` 对象将 `?api-version=2018-01-01` 添加到调用。 | 
| <other-action-specific-input-properties> | JSON 对象 | 应用于此指定操作的任何其他输入属性 | 
| <other-action-specific-properties> | JSON 对象 | 应用于此指定操作的任何其他属性 | 
|||| 

*示例*

此操作定义通过向指定终结点发送请求来获取最新资讯：

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://mynews.example.com/latest"
   }
}
```

<a name="join-action"></a>

### <a name="join-action"></a>Join 操作

此操作基于数组中的所有项创建一个字符串，并使用指定的分隔符字符分隔这些项。 

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": <array>,
      "joinWith": "<delimiter>"
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <array> | Array | 提供源项的数组或表达式。 如果指定表达式，请将表达式括于双引号内。 | 
| <*delimiter*> | 单字符字符串 | 分隔字符串中每个项的字符 | 
|||| 

*示例*

假设先前已创建了一个包含此整数数组的“myIntegerArray”变量： 

`[1,2,3,4]` 

此操作定义通过在表达式中使用 `variables()` 函数来获取变量中的值，并使用这些值（以逗号隔开）创建此字符串：`"1,2,3,4"`

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": "@variables('myIntegerArray')",
      "joinWith": ","
   },
   "runAfter": {}
}
```

<a name="parse-json-action"></a>

### <a name="parse-json-action"></a>Parse JSON 操作

此操作从 JSON 内容中的属性创建用户友好的字段或令牌。 之后可改用令牌在逻辑应用中访问这些属性。 例如，要使用 Azure 服务总线和 Azure Cosmos DB 等服务的 JSON 输出时，可将此操作包含在逻辑应用中，以便可更轻松地引用该输出中的数据。 

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": "<JSON-source>",
         "schema": { "<JSON-schema>" }
      },
      "runAfter": {}
},
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <JSON-source> | JSON 对象 | 要分析的 JSON 内容 | 
| <JSON-schema> | JSON 对象 | 描述基础 JSON 内容的 JSON 架构，操作将该架构用于分析源 JSON 内容。 <p>**提示**：在逻辑应用设计器中，可提供该架构或提供示例有效负载，以便操作可生成该架构。 | 
|||| 

*示例*

此操作定义中创建这些令牌可用于在工作流中，但仅在操作中运行的以下**分析 JSON**操作： 

`FirstName`、`LastName` 和 `Email`

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": {
         "Member": {
            "Email": "Sophie.Owen@contoso.com",
            "FirstName": "Sophie",
            "LastName": "Owen"
         }
      },
      "schema": {
         "type": "object",
         "properties": {
            "Member": {
               "type": "object",
               "properties": {
                  "Email": {
                     "type": "string"
                  },
                  "FirstName": {
                     "type": "string"
                  },
                  "LastName": {
                     "type": "string"
                  }
               }
            }
         }
      }
   },
   "runAfter": { }
},
```

在此示例中，“content”属性指定操作要分析的 JSON 内容。 此外，还可提供此 JSON 内容作为生成该架构的相同有效负载。

```json
"content": {
   "Member": { 
      "FirstName": "Sophie",
      "LastName": "Owen",
      "Email": "Sophie.Owen@contoso.com"
   }
},
```

“schema”属性指定用于描述 JSON 内容的 JSON 架构：

```json
"schema": {
   "type": "object",
   "properties": {
      "Member": {
         "type": "object",
         "properties": {
            "FirstName": {
               "type": "string"
            },
            "LastName": {
               "type": "string"
            },
            "Email": {
               "type": "string"
            }
         }
      }
   }
}
```

<a name="query-action"></a>

### <a name="query-action"></a>Query 操作

此操作基于指定条件或筛选器使用另一个数组中的项创建数组。

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": <array>,
      "where": "<condition-or-filter>"
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <array> | Array | 提供源项的数组或表达式。 如果指定表达式，请将表达式括于双引号内。 |
| <condition-or-filter> | String | 用于筛选源数组中的项的条件 <p>**注意**：如果没有任何值满足此条件，则该操作会创建一个空数组。 |
|||| 

*示例*

此操作定义创建一个值大于指定值 2 的数组：

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": [ 1, 3, 0, 5, 4, 2 ],
      "where": "@greater(item(), 2)"
   }
}
```

<a name="response-action"></a>

### <a name="response-action"></a>Response 操作  

此操作创建 HTTP 请求响应的有效负载。 

```json
"Response" {
    "type": "Response",
    "kind": "http",
    "inputs": {
        "statusCode": 200,
        "headers": { <response-headers> },
        "body": { <response-body> }
    },
    "runAfter": {}
},
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <response-status-code> | Integer | 发送到传入请求的 HTTP 状态代码。 默认代码为“200 OK”，但此代码可为以 2xx、4xx 或 5xx（非 3xxx）开头的任何有效状态代码。 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <response-headers> | JSON 对象 | 要包括在响应中的一个或多个标头 | 
| <response-body> | 各种 | 响应正文，可为字符串、JSON 对象甚至上一个操作的二进制内容 | 
|||| 

*示例*

此操作定义创建一个包含指定状态代码、消息正文和消息标头的 HTTP 请求响应：

```json
"Response": {
   "type": "Response",
   "inputs": {
      "statusCode": 200,
      "body": {
         "ProductID": 0,
         "Description": "Organic Apples"
      },
      "headers": {
         "x-ms-date": "@utcnow()",
         "content-type": "application/json"
      }
   },
   "runAfter": {}
}
```

*限制*

不同于其他操作，Response 操作具有一些特殊限制： 

* 工作流仅可在以 HTTP 请求触发器开始时才能使用 Response 操作，即工作流必须由 HTTP 请求触发。

* 工作流可在任意位置使用 Response 操作，但 Foreach 循环和 Until 循环（包括序列循环和并行分支）内除外。 

* 仅当 Response 操作所需的所有操作都在 [HTTP 请求超时限制](../logic-apps/logic-apps-limits-and-config.md#request-limits)内完成时，原始 HTTP 请求才会获取工作流的响应。

  但是，如果工作流调用另一个逻辑应用作为嵌套工作流，则父级工作流在嵌套工作流完成之前将处于等待状态，而不管嵌套工作流完成需要多久时间。

* 工作流在使用 Response 操作和异步响应模式时，无法还在触发器定义中使用 splitOn 命令，因为该命令会创建多个运行。 使用 PUT 方法时请检查是否存在这种情况，如果为 true，则会返回“错误的请求”响应。

  否则，如果工作流使用 splitOn 命令和 Response 操作，则工作流会异步运行并立即返回响应“202 ACCEPTED”。

* 当工作流的执行到达 Response 操作但传入请求已接收响应时，Response 操作会由于冲突而被标记为“Failed”。 因此，逻辑应用运行也会被标记为“Failed”状态。

<a name="select-action"></a>

### <a name="select-action"></a>选择操作

此操作通过基于指定映射转换另一个数组中的项，创建包含 JSON 对象的数组。 输出数组和源数组的项数始终相同。 虽然无法更改输出数组中的对象数，但可以添加或删除这些对象的属性以及属性值。 `select` 属性至少指定一个键值对，此键值对定义用于转换源数组中的项的映射。 键值对表示输出数组中所有对象的某个属性和该属性的值。 

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": <array>,
      "select": { 
          "<key-name>": "<expression>",
          "<key-name>": "<expression>"        
      }
   },
   "runAfter": {}
},
```

*必需* 

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <array> | Array | 提供源项的数组或表达式。 确保将表达式放入双引号内。 <p>**注意**：如果源数组为空，则该操作会创建一个空数组。 | 
| <*key-name*> | String | 从 <expression>  分配给结果的属性名称<p>若要为输出数组中的所有对象添加一个新属性，请提供该属性的 <key-name> 以及属性值的 <expression>。 <p>若要从数组的所有对象中删除属性，请删除该属性的 <key-name>。 | 
| <*expression*> | String | 转换源数组中的项并将结果分配给 <key-name>的表达式 | 
|||| 

Select 操作创建一个数组作为输出，因此，任何想要使用此输出的操作必须接受数组或者该数组必须转换为使用者操作接受的类型。 例如，若要将此输出数组转换为字符串，可将此数组传递到 Compose 操作，然后在其他操作中引用 Compose 操作的输出。

*示例*

此操作定义基于一个整数数组创建 JSON 对象数组。 此操作循环访问此源数组，使用 `@item()` 表达式获取每个整数值，然后将每个值分配给每个 JSON 对象中的“`number`”属性： 

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": [ 1, 2, 3 ],
      "select": { 
         "number": "@item()" 
      }
   },
   "runAfter": {}
},
```

以下为此操作创建的数组：

`[ { "number": 1 }, { "number": 2 }, { "number": 3 } ]`

若要在其他操作中使用此数组输出，请将此输出传递到 Compose 操作中：

```json
"Compose": {
   "type": "Compose",
   "inputs": "@body('Select')",
   "runAfter": {
      "Select": [ "Succeeded" ]
   }
},
```

然后可将 Compose 操作的输出用于其他操作中，例如“Office 365 Outlook - 发送电子邮件”操作：

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "@{outputs('Compose')}",
         "Subject": "Output array from Select and Compose actions",
         "To": "<your-email@domain>"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "post",
      "path": "/Mail"
   },
   "runAfter": {
      "Compose": [ "Succeeded" ]
   }
},
```

<a name="table-action"></a>

### <a name="table-action"></a>表操作

此操作根据数组创建 CSV 或 HTML 表。 对于包含 JSON 对象的数组，此操作根据对象的属性名称自动创建列标头。 对于包含其他数据类型的数组，必须指定列标头和值。 例如，此数组包含该操作可用于列标头名称的“ID”和“Product_Name”属性：

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]` 

```json
"Create_<CSV | HTML>_table": {
   "type": "Table",
   "inputs": {
      "format": "<CSV | HTML>",
      "from": <array>,
      "columns": [ 
         {
            "header": "<column-name>",
            "value": "<column-value>"
         },
         {
            "header": "<column-name>",
            "value": "<column-value>"
         } 
      ]
   },
   "runAfter": {}
}
```

*必需* 

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <CSV 或 HTML>| String | 要创建的表的格式 | 
| <array> | Array | 为表提供源项的数组或表达式 <p>**注意**：如果源数组为空，则该操作会创建一个空表。 | 
|||| 

可选

若要指定或自定义列标头和值，请使用 `columns` 数组。 `header-value` 对具有相同标头名称时，其值显示在该标头名称下相同的列中。 否则，每个唯一的标头定义一个唯一列。

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <column-name> | String | 列的标头名称 | 
| <column-value> | 任意 | 该列中的值 | 
|||| 

*示例 1*

假设先前已创建了一个当前包含此数组的“myItemArray”变量： 

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]`

此操作定义根据“myItemArray”变量创建一个 CSV 表。 `from` 属性使用的表达式通过使用 `variables()` 函数从“myItemArray”获取数组： 

```json
"Create_CSV_table": {
   "type": "Table",
   "inputs": {
      "format": "CSV",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

以下为此操作创建的 CSV 表： 

```
ID,Product_Name 
0,Apples 
1,Oranges 
```

*示例 2*

此操作定义根据“myItemArray”变量创建一个 HTML 表。 `from` 属性使用的表达式通过使用 `variables()` 函数从“myItemArray”获取数组： 

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

以下为此操作创建的 HTML 表： 

<table><thead><tr><th>ID</th><th>Product_Name</th></tr></thead><tbody><tr><td>0</td><td>苹果</td></tr><tr><td>第</td><td>Oranges</td></tr></tbody></table>

*示例 3*

此操作定义根据“myItemArray”变量创建一个 HTML 表。 但是，本示例使用“Stock_ID”和“Description”替代默认列标头名称，并在“Description”列的值中添加了“Organic”一词。

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')",
      "columns": [ 
         {
            "header": "Stock_ID",
            "value": "@item().ID"
         },
         {
            "header": "Description",
            "value": "@concat('Organic ', item().Product_Name)"
         }
      ]
    },
   "runAfter": {}
},
```

以下为此操作创建的 HTML 表： 

<table><thead><tr><th>Stock_ID</th><th>描述</th></tr></thead><tbody><tr><td>0</td><td>Organic Apples</td></tr><tr><td>1</td><td>Organic Oranges</td></tr></tbody></table>

<a name="terminate-action"></a>

### <a name="terminate-action"></a>Terminate 操作

此操作停止工作流实例运行、 取消正在进行中的任何操作，将跳过所有剩余的操作，并返回指定的状态。 例如，当逻辑应用由于错误状态而必须完全退出时，可使用此 Terminate 操作。 此操作不影响已经完成的操作，且不能出现在 Foreach 和 Until 循环（包括序列循环）内。 

```json
"Terminate": {
   "type": "Terminate",
   "inputs": {
       "runStatus": "<status>",
       "runError": {
            "code": "<error-code-or-name>",
            "message": "<error-message>"
       }
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <status> | String | 运行返回的状态：“失败”、“已取消”或者“已成功” |
|||| 

可选

仅在“runStatus”属性设为“Failed”状态时，“runStatus”对象的属性才适用。

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <error-code-or-name> | String | 错误的代码或名称 |
| <error-message> | String | 消息或文本，描述错误和应用用户可采取的任何操作 | 
|||| 

*示例*

此操作定义停止工作流运行，将运行状态设为“Failed”，并返回状态、错误代码和错误消息：

```json
"Terminate": {
    "type": "Terminate",
    "inputs": {
        "runStatus": "Failed",
        "runError": {
            "code": "Unexpected response",
            "message": "The service received an unexpected response. Please try again."
        }
   },
   "runAfter": {}
}
```

<a name="wait-action"></a>

### <a name="wait-action"></a>Wait 操作  

此操作将工作流执行暂停指定时间间隔或暂停到指定时间（仅二者之一）。 

指定时间间隔

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": <number-of-units>,
         "unit": "<interval>"
      }
   },
   "runAfter": {}
},
```

指定时间

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "<date-time-stamp>"
      }
   },
   "runAfter": {}
},
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <number-of-units> | Integer | 对于 Delay 操作，要等待的单位数 | 
| <*interval*> | String | 对于“延迟”操作，等待间隔时间为：“秒”、“分钟”、“小时”、“天”、“周”、“月” | 
| <date-time-stamp> | String | 对于 Delay Until 操作，执行的恢复日期和时间。 该值必须使用 [UTC 日期时间格式](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)。 | 
|||| 

*示例 1*

此操作定义将工作流暂停 15 分钟：

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": 15,
         "unit": "Minute"
      }
   },
   "runAfter": {}
},
```

*示例 2*

此操作定义将工作流暂停到指定时间：

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "2017-10-01T00:00:00Z"
      }
   },
   "runAfter": {}
},
```

<a name="workflow-action"></a>

### <a name="workflow-action"></a>Workflow 操作

此操作调用先前创建的另一个逻辑应用，这意味着可以包含和重复使用其他逻辑应用工作流。 如果子级逻辑应用返回响应，则还可将子级或嵌套逻辑应用的输出用于嵌套逻辑应用之后的操作中。

逻辑应用引擎检查对要调用的触发器的访问权限，确保你有权访问该触发器。 此外，嵌套逻辑应用还必须满足以下条件：

* 触发器（例如 [Request](#request-trigger) 或 [HTTP 触发器](#http-trigger)）使嵌套应用可调用

* 与父级逻辑应用相同的 Azure 订阅

* 若要将嵌套逻辑应用的输出用于父级逻辑应用中，嵌套逻辑应用必须具有 [Response](#response-action) 操作 

```json
"<nested-logic-app-name>": {
   "type": "Workflow",
   "inputs": {
      "body": { "<body-content" },
      "headers": { "<header-content>" },
      "host": {
         "triggerName": "<trigger-name>",
         "workflow": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group>/providers/Microsoft.Logic/<nested-logic-app-name>"
         }
      }
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <nested-logic-app-name> | String | 要调用的逻辑应用的名称 | 
| <trigger-name> | String | 要调用的嵌套逻辑应用中的触发器的名称 | 
| <Azure-subscription-ID> | String | 嵌套逻辑应用的 Azure 订阅 ID |
| <Azure-resource-group> | String | 嵌套逻辑应用的 Azure 资源组名称 |
| <nested-logic-app-name> | String | 要调用的逻辑应用的名称 |
||||

可选

| 值 | Type | 描述 | 
|-------|------|-------------|  
| <header-content> | JSON 对象 | 与调用一同发送的任何标头 | 
| <body-content> | JSON 对象 | 与调用一同发送的任何消息内容 | 
||||

*输出*

此操作的输出基于嵌套应用的 Response 操作而有所不同。 如果嵌套逻辑应用不包含 Response 操作，则输出为空。

*示例*

“Start_search”操作成功完成后，此工作流操作定义调用名为“Get_product_information”的另一个逻辑应用（该逻辑应用在指定输入中传递）： 

```json
"actions": {
   "Start_search": { <action-definition> },
   "Get_product_information": {
      "type": "Workflow",
      "inputs": {
         "body": {
            "ProductID": "24601",
         },
         "host": {
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/InventoryManager-RG/providers/Microsoft.Logic/Get_product_information",
            "triggerName": "Find_product"
         },
         "headers": {
            "content-type": "application/json"
         }
      },
      "runAfter": { 
         "Start_search": [ "Succeeded" ]
      }
   }
},
```

## <a name="control-workflow-action-details"></a>控制工作流操作详细信息

<a name="foreach-action"></a>

### <a name="foreach-action"></a>Foreach 操作

此循环操作循环访问数组并针对每个数组项执行操作。 默认情况下，“for each”循环可以多达最大循环数并行运行。 有关此最大值，请参阅[限制和配置](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。了解[如何创建“for each”循环](../logic-apps/logic-apps-control-flow-loops.md#foreach-loop)。

```json
"For_each": {
   "type": "Foreach",
   "actions": { 
      "<action-1>": { "<action-definition-1>" },
      "<action-2>": { "<action-definition-2>" }
   },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": <count>
      }
    },
    "operationOptions": "<operation-option>"
}
```

*必需* 

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <action-1...n> | String | 在每个数组项上运行的操作的名称 | 
| <action-definition-1...n> | JSON 对象 | 运行的操作的定义 | 
| <for-each-expression> | String | 用于引用指定数组中每个项的表达式 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <*count*> | Integer | 默认情况下，“for each”循环迭代同时或并行（最多达到[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)）运行。 若要通过设置新的 <count> 值更改此限制，请参阅[更改“for each”循环并发](#change-for-each-concurrency)。 | 
| <operation-option> | String | 若要按顺序而不是并行运行“for each”循环，请将 <operation-option> 设为 `Sequential` 或将 <count> 设为 `1`（仅二者之一）。 有关详细信息，请参阅[按顺序运行“for each”循环](#sequential-for-each)。 | 
|||| 

*示例*

此“for each”循环为数组中的每个项发送一封电子邮件，其中包含来自传入电子邮件的附件。 该循环将一封电子邮件（含附件）发送给审阅此附件的人员。

```json
"For_each": {
   "type": "Foreach",
   "actions": {
      "Send_an_email": {
         "type": "ApiConnection",
         "inputs": {
            "body": {
               "Body": "@base64ToString(items('For_each')?['Content'])",
               "Subject": "Review attachment",
               "To": "Sophie.Owen@contoso.com"
                },
            "host": {
               "connection": {
                  "id": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {}
      }
   },
   "foreach": "@triggerBody()?['Attachments']",
   "runAfter": {}
}
```

为了仅指定一个作为触发器输出传递的数组，此表达式从触发器正文获取 <array-name> 数组。 为避免在不存在数组的情况下出现故障，该表达式使用 `?` 运算符：

`@triggerBody()?['<array-name>']` 

<a name="if-action"></a>

### <a name="if-action"></a>If 操作

此操作为条件语句，可评估表示条件的表达式并基于条件为 true 还是 false 来运行不同的分支。 如果条件为 true，则该条件将标记为“Succeeded”状态。 了解[如何创建条件语句](../logic-apps/logic-apps-control-flow-conditional-statement.md)。

``` json
"Condition": {
   "type": "If",
   "expression": { "<condition>" },
   "actions": {
      "<action-1>": { "<action-definition>" }
   },
   "else": {
      "actions": {
        "<action-2>": { "<action-definition" }
      }
   },
   "runAfter": {}
}
```

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <condition> | JSON 对象 | 要评估的条件（可以为表达式） | 
| <action-1> | JSON 对象 | <condition> 评估结果为 true 时要运行的操作 | 
| <action-definition> | JSON 对象 | 操作的定义 | 
| <action-2> | JSON 对象 | <condition> 评估结果为 false 时要运行的操作 | 
|||| 

`actions` 或 `else` 对象中的操作会获取以下状态：

* "Succeeded"：运行且成功
* "Failed"：运行但失败
* "Skipped"：未运行各自的分支

*示例*

此条件指定当整数变量值大于零时，工作流检查网站。 如果变量为零或小于零，则工作流检查另一个网站。

```json
"Condition": {
   "type": "If",
   "expression": {
      "and": [ {
         "greater": [ "@variables('myIntegerVariable')", 0 ] 
      } ]
   },
   "actions": { 
      "HTTP - Check this website": {
         "type": "Http",
         "inputs": {
         "method": "GET",
            "uri": "http://this-url"
         },
         "runAfter": {}
      }
   },
   "else": {
      "actions": {
         "HTTP - Check this other website": {
            "type": "Http",
            "inputs": {
               "method": "GET",
               "uri": "http://this-other-url"
            },
            "runAfter": {}
         }
      }
   },
   "runAfter": {}
}
```  

#### <a name="how-conditions-use-expressions"></a>条件如何使用表达式

下面是一些示例，显示了如何在条件中使用表达式：
  
| JSON | 结果 | 
|------|--------| 
| "expression": "@parameters('<hasSpecialAction>')" | 仅针对布尔表达式，该条件对任何评估结果为 true 的值进行传递。 <p>若要将其他类型转换为布尔值，请使用以下函数：`empty()` 或 `equals()`。 | 
| "expression": "@greater(actions('<action>').output.value, parameters('<threshold>'))" | 针对比较函数，此操作仅在 <action> 的输出大于 <threshold> 值时运行。 | 
| "expression": "@or(greater(actions('<action>').output.value, parameters('<threshold>')), less(actions('<same-action>').output.value, 100))" | 针对逻辑函数和创建嵌套布尔表达式，此操作在 <action> 的输出大于 <threshold> 值或低于 100 时运行。 | 
| "expression": "@equals(length(actions('<action>').outputs.errors), 0))" | 可以使用数组函数检查该数组是否具有项。 仅当 `errors` 数组为空时才运行该操作。 | 
||| 

<a name="scope-action"></a>

### <a name="scope-action"></a>Scope 操作

此操作按照逻辑将操作分组到范围，以便在该范围内的操作完成运行后，获取这些操作的自身状态。 然后，可使用该范围的状态确定是否运行其他操作。 了解[如何创建范围](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)。

```json
"Scope": {
   "type": "Scope",
   "actions": {
      "<inner-action-1>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<inner-action-2>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   }
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------|  
| <inner-action-1...n> | JSON 对象 | 在范围内运行的一个或多个操作 |
| <action-inputs> | JSON 对象 | 每个操作的输入 |
|||| 

<a name="switch-action"></a>

### <a name="switch-action"></a>Switch 操作

此操作也称 switch 语句，它将其他操作组织到事例中，并为每个事例（如有默认事例，则除外）分配一个值。 在运行工作流时，Switch 操作将比较表达式、对象或令牌的值与每个事例的指定值。 如果 Switch 操作找到匹配事例，则工作流仅运行该事例的操作。 在每次运行 Switch 操作时，仅存在一个匹配事例或无任何匹配事例存在。 如果不存在匹配事例，则 Switch 操作将运行默认操作。 了解[如何创建 switch 语句](../logic-apps/logic-apps-control-flow-switch-statement.md)。

``` json
"Switch": {
   "type": "Switch",
   "expression": "<expression-object-or-token>",
   "cases": {
      "Case": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      },
      "Case_2": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      }
   },
   "default": {
      "actions": {
         "<default-action-name>": { "<default-action-definition>" }
      }
   },
   "runAfter": {}
}
```

*必需*

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <expression-object-or-token> | 多种多样 | 要计算的表达式、JSON 对象或令牌 | 
| <action-name> | String | 要针对匹配事例运行的操作的名称 | 
| <action-definition> | JSON 对象 | 要针对匹配事例运行的操作的定义 | 
| <matching-value> | 多种多样 | 要与计算结果比较的值 | 
|||| 

可选

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <default-action-name> | String | 无匹配事例存在时要运行的默认操作的名称 | 
| <default-action-definition> | JSON 对象 | 无匹配事例存在时要运行的操作的定义 | 
|||| 

*示例*

此操作定义评估答复审核请求电子邮件的人员选择了“Approve”还是“Reject”。 基于所作选择，Switch 操作将对匹配事例运行操作，即向答复者发送另一封电子邮件，但每个事例中的用词不同。 

``` json
"Switch": {
   "type": "Switch",
   "expression": "@body('Send_approval_email')?['SelectedOption']",
   "cases": {
      "Case": {
         "actions": {
            "Send_an_email": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your approval.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}
         },
         "case": "Approve"
      },
      "Case_2": {
         "actions": {
            "Send_an_email_2": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your response.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}     
         },
         "case": "Reject"
      }
   },
   "default": {
      "actions": { 
         "Send_an_email_3": { 
            "type": "ApiConnection",
            "inputs": {
               "Body": "Please respond with either 'Approve' or 'Reject'.",
               "Subject": "Please respond", 
               "To": "Sophie.Owen@contoso.com"
            },
            "host": {
               "connection": {
                  "name": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {} 
      }
   },
   "runAfter": {
      "Send_approval_email": [ 
         "Succeeded"
      ]
   }
}
```

<a name="until-action"></a>

### <a name="until-action"></a>Until 操作

此循环操作包含在指定条件为 true 之前一直运行的操作。 在运行所有其他操作后，此循环会最后检查此条件。 可在 `"actions"` 对象中包含多个操作，且该操作必须至少定义一个限制。 了解[如何创建“until”循环](../logic-apps/logic-apps-control-flow-loops.md#until-loop)。 

```json
 "Until": {
   "type": "Until",
   "actions": {
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   },
   "expression": "<condition>",
   "limit": {
      "count": <loop-count>,
      "timeout": "<loop-timeout>"
   },
   "runAfter": {}
}
```

| 值 | Type | 描述 | 
|-------|------|-------------| 
| <action-name> | String | 要在循环内运行的操作的名称 | 
| <action-type> | String | 要运行的操作类型 | 
| <action-inputs> | 各种 | 要运行的操作的输入 | 
| <condition> | String | 当循环中的所有操作都运行完成后要计算的条件或表达式 | 
| <loop-count> | Integer | 针对操作可运行的最大循环数的限制。 默认 `count` 值为 60。 | 
| <loop-timeout> | String | 针对循环可运行的最长时间的限制。 默认 `timeout` 值为 `PT1H`，即要求的 [ISO 8601 格式](https://en.wikipedia.org/wiki/ISO_8601)。 |
|||| 

*示例*

此循环操作定义向指定 URL 发送一个 HTTP 请求，直到满足以下某个条件： 

* 该请求获得一个含有“200 OK”状态代码的响应。
* 该循环已运行 60 次。
* 该循环已运行一小时。

```json
 "Run_until_loop_succeeds_or_expires": {
    "type": "Until",
    "actions": {
        "Http": {
            "type": "Http",
            "inputs": {
                "method": "GET",
                "uri": "http://myurl"
            },
            "runAfter": {}
        }
    },
    "expression": "@equals(outputs('Http')['statusCode', 200])",
    "limit": {
        "count": 60,
        "timeout": "PT1H"
    },
    "runAfter": {}
}
```

<a name="subscribe-unsubscribe"></a>

## <a name="webhooks-and-subscriptions"></a>Webhook 和订阅

基于 Webhook 的触发器和操作不会定期检查终结点，而是等待这些终结点中的指定事件或数据。 这些触发器和操作通过提供一个回叫 URL 来订阅终结点，终结点可向该 URL 发送响应。

当工作流出现任何形式的更改时，都将发生 `subscribe` 调用，例如续订凭据或触发器或操作的输入参数发生更改时。 该调用使用与标准 HTTP 操作相同的参数。 

当操作使得触发器或操作无效时，会自动发生 `unsubscribe` 调用，例如：

* 删除或禁用触发器。 
* 删除或禁用工作流。 
* 删除或禁用订阅。 

为支持这些调用，`@listCallbackUrl()` 表达式为此触发器或操作返回唯一的“回叫 URL”。 此 URL 代表使用服务 REST API 的终结点的唯一标识符。 此函数的参数与 webhook 触发器或操作相同。

<a name="asynchronous-limits"></a>

## <a name="change-asynchronous-duration"></a>更改异步持续时间

对于触发器和操作，均可通过添加 `limit.timeout` 属性的方式将异步模式的持续时间限制为指定时间间隔。 这样，如果该指定间隔结束时操作尚未完成，则会使用 `ActionTimedOut` 代码将操作的状态标记为 `Cancelled`。 `timeout` 属性使用 [ISO 8601 格式](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)。 

``` json
"<trigger-or-action-name>": {
   "type": "Workflow | Webhook | Http | ApiConnectionWebhook | ApiConnection",
   "inputs": {},
   "limit": {
      "timeout": "PT10S"
   },
   "runAfter": {}
}
```

<a name="runtime-config-options"></a>

## <a name="runtime-configuration-settings"></a>运行时配置设置

可通过触发器或操作定义中的以下 `runtimeConfiguration` 属性更改触发器和操作的默认运行时行为。

| 属性 | Type | 描述 | 触发器或操作 | 
|----------|------|-------------|-------------------| 
| `runtimeConfiguration.concurrency.runs` | Integer | 更改[*默认限制*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)上可以运行在同一时间，或并行的工作流实例数。 此值可限制后端系统接收的请求数。 <p>将 `runs` 属性设置为 `1` 与将 `operationOptions` 属性设置为 `SingleInstance` 的作用相同。 可以设置其中任一属性，但不能同时设置二者。 <p>若要更改此默认限制，请参阅[更改触发器并发](#change-trigger-concurrency)或[按顺序触发实例](#sequential-trigger)。 | 所有触发器 | 
| `runtimeConfiguration.concurrency.maximumWaitingRuns` | Integer | 更改[*默认限制*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)上可等待您的工作流已在运行最大并发实例时运行的工作流实例数。 可在 `concurrency.runs` 属性中更改并发限制。 <p>若要更改此默认限制，请参阅[更改等待的运行限制](#change-waiting-runs)。 | 所有触发器 | 
| `runtimeConfiguration.concurrency.repetitions` | Integer | 更改针对可同时或并行运行的“for each”循环迭代数的[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 <p>将 `repetitions` 属性设置为 `1` 与将 `operationOptions` 属性设置为 `SingleInstance` 的作用相同。 可以设置其中任一属性，但不能同时设置二者。 <p>若要更改默认限制，请参阅[更改“for each”并发](#change-for-each-concurrency)或[按顺序运行“for each”循环](#sequential-for-each)。 | 操作： <p>[Foreach](#foreach-action) | 
| `runtimeConfiguration.paginationPolicy.minimumItemCount` | Integer | 对于支持且已启用分页的特定操作，此值指定*最小*数字要检索的结果。 <p>若要启用分页，请参阅[使用分页获取大容量数据、 项或结果](../logic-apps/logic-apps-exceed-default-page-size-with-pagination.md) | 操作：随其而变化 |
||||| 

<a name="operation-options"></a>

## <a name="operation-options"></a>操作选项

可通过触发器或操作定义中的以下 `operationOptions` 属性更改触发器和操作的默认行为。

| 操作选项 | Type | 描述 | 触发器或操作 | 
|------------------|------|-------------|-------------------| 
| `DisableAsyncPattern` | String | 以同步方式而非异步方式运行基于 HTTP 的操作。 <p><p>若要设置此选项，请参阅[同步运行操作](#asynchronous-patterns)。 | 操作： <p>[ApiConnection](#apiconnection-action), <br>[HTTP](#http-action)、 <br>[响应](#response-action) | 
| `OptimizedForHighThroughput` | String | 将针对每 5 分钟的操作执行数的[默认限制](../logic-apps/logic-apps-limits-and-config.md#throughput-limits)更改为[最大限制](../logic-apps/logic-apps-limits-and-config.md#throughput-limits)。 <p><p>若要设置此选项，请参阅[在高吞吐量模式下运行](#run-high-throughput-mode)。 | 所有操作 | 
| `Sequential` | String | 每次运行一个“for each”循环迭代，而不是同时并行运行所有迭代。 <p>此选项与将 `runtimeConfiguration.concurrency.repetitions` 属性设置为 `1` 的作用相同。 可以设置其中任一属性，但不能同时设置二者。 <p><p>若要设置此选项，请参阅[按顺序运行“for each”循环](#sequential-for-each)。| 操作： <p>[Foreach](#foreach-action) | 
| `SingleInstance` | String | 按顺序对每个逻辑应用实例运行此触发器，并在等待上一个活动运行完成后，再触发下一个逻辑应用实例。 <p><p>此选项与将 `runtimeConfiguration.concurrency.runs` 属性设置为 `1` 的作用相同。 可以设置其中任一属性，但不能同时设置二者。 <p>若要设置此选项，请参阅[按顺序触发实例](#sequential-trigger)。 | 所有触发器 | 
||||

<a name="change-trigger-concurrency"></a>

### <a name="change-trigger-concurrency"></a>更改触发器并发

默认情况下，逻辑应用实例以并发方式同时或并行（最多达到[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)）运行。 因此，前面的工作流实例完成运行之前，将触发触发器的每个实例。 此限制可控制后端系统接收的请求数。 

若要更改此默认限制，可使用代码视图编辑器或逻辑应用设计器，因为通过设计器更改并发设置会添加或更新基础触发器定义中的 `runtimeConfiguration.concurrency.runs` 属性，反之亦然。 此属性可控制可以并行运行的工作流实例的最大次数。 

> [!NOTE] 
> 如果通过设计器或代码视图编辑器将触发器设为按顺序运行，请勿在代码视图编辑器中将触发器的 `operationOptions` 属性设置为 `SingleInstance`。 否则会出现验证错误。 有关详细信息，请参阅[按顺序触发实例](#sequential-trigger)。

#### <a name="edit-in-code-view"></a>在代码视图中编辑 

在基础触发器定义中，添加一个值位于 `1` - `50`（含）范围的 `runtimeConfiguration.concurrency.runs` 属性，或将该属性的值更新为该范围内的一个值。

以下示例将并发运行数限制为 10 个实例：

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 10
      }
   }
}
```

#### <a name="edit-in-logic-apps-designer"></a>在逻辑应用设计器中编辑

1. 在触发器的右上角选择省略号 (...) 按钮，然后选择“设置”。

2. 在“并发控制”下，将“限制”设为“开启”。 

3. 将“并行度”滑块拖到所需值处。 若要按顺序运行逻辑应用，请将滑块值拖到 1。

<a name="change-for-each-concurrency"></a>

### <a name="change-for-each-concurrency"></a>更改“for each”并发

默认情况下，“for each”循环迭代同时或并行（最多达到[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)）运行。 若要更改此默认限制，可使用代码视图编辑器或逻辑应用设计器，因为通过设计器更改并发设置会添加或更新基础“for each”操作定义中的 `runtimeConfiguration.concurrency.repetitions` 属性，反之亦然。 此属性控制可并行运行的最大迭代数。

> [!NOTE] 
> 如果通过设计器或代码视图编辑器将“for each”操作设为按顺序运行，请勿在代码视图编辑器中将此操作的 `operationOptions` 属性设置为 `Sequential`。 否则会出现验证错误。 有关详细信息，请参阅[按顺序运行“for each”循环](#sequential-for-each)。

#### <a name="edit-in-code-view"></a>在代码视图中编辑 

在基础“for each”定义中，添加一个值位于 `1` - `50`（含）范围的 `runtimeConfiguration.concurrency.repetitions` 属性，或将该属性的值更新为该范围内的一个值。 

以下示例将并发运行数限制为 10 个迭代：

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 10
      }
   }
}
```

#### <a name="edit-in-logic-apps-designer"></a>在逻辑应用设计器中编辑

1. 在“For each”操作中，从右上角选择省略号 (...) 按钮，然后选择“设置”。

2. 在“并发控制”下，将“并发控制”设置为“开启”。 

3. 将“并行度”滑块拖到所需值处。 若要按顺序运行逻辑应用，请将滑块值拖到 1。

<a name="change-waiting-runs"></a>

### <a name="change-waiting-runs-limit"></a>更改等待的运行限制

默认情况下，所有逻辑应用工作流实例都运行在同一时间，同时，或并行最多[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)。 之前处于活动状态的工作流实例完成运行之前，将触发触发器的每个实例。 尽管你可以[来更改此默认限制](#change-trigger-concurrency)，当工作流实例的数量达到了新的并发限制，必须等待新的任何其他实例运行。 

可等待的运行数也具有[默认限制](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits)，该默认限制可以更改。 但是，逻辑应用达到等待运行数限制后，逻辑应用引擎不再接受新的运行。 请求和 webhook 触发器返回 429 错误，并且重复的触发器会开始跳过轮询尝试。

若要更改等待运行数的默认限制，请在基础触发器定义中添加 `runtimeConfiguration.concurency.maximumWaitingRuns` 属性，并将其值设为 `0` 和 `100` 之间的某个值。 

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "maximumWaitingRuns": 50
      }
   }
}
```

<a name="sequential-trigger"></a>

### <a name="trigger-instances-sequentially"></a>按顺序触发实例

若要运行每个逻辑应用工作流实例在前一个实例完成之后才运行，设置要按顺序运行的触发器。 可使用代码视图编辑器或逻辑应用设计器，因为通过设计器更改并发设置还会添加或更新基础触发器定义中的 `runtimeConfiguration.concurrency.runs` 属性，反之亦然。 

> [!NOTE] 
> 如果通过设计器或代码视图编辑器将触发器设为按顺序运行，请勿在代码视图编辑器中将触发器的 `operationOptions` 属性设置为 `Sequential`。 否则会出现验证错误。 

#### <a name="edit-in-code-view"></a>在代码视图中编辑

在触发器定义中，设置以下任一属性，但不可同时设置二者。 

将 `runtimeConfiguration.concurrency.runs` 属性设为 `1`：

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 1
      }
   }
}
```

或

将 `operationOptions` 属性设为 `SingleInstance`：

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "operationOptions": "SingleInstance"
}
```

#### <a name="edit-in-logic-apps-designer"></a>在逻辑应用设计器中编辑

1. 在触发器的右上角选择省略号 (...) 按钮，然后选择“设置”。

2. 在“并发控制”下，将“限制”设为“开启”。 

3. 将“并行度”滑块拖到数字 `1`。 

<a name="sequential-for-each"></a>

### <a name="run-for-each-loops-sequentially"></a>按顺序运行“for each”循环

若要仅在上一迭代完成运行后才运行“for each”循环迭代，请将“for each”操作设为按顺序运行。 可使用代码视图编辑器或逻辑应用设计器，因为通过设计器更改操作的并发还会添加或更新基础操作定义中的 `runtimeConfiguration.concurrency.repetitions` 属性，反之亦然。 

> [!NOTE] 
> 如果通过设计器或代码视图编辑器将“for each”操作设为按顺序运行，请勿在代码视图编辑器中将此操作的 `operationOptions` 属性设置为 `Sequential`。 否则会出现验证错误。 

#### <a name="edit-in-code-view"></a>在代码视图中编辑

在操作定义中，设置以下任一属性，但不可同时设置二者。 

将 `runtimeConfiguration.concurrency.repetitions` 属性设为 `1`：

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 1
      }
   }
}
```

或

将 `operationOptions` 属性设为 `Sequential`：

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "operationOptions": "Sequential"
}
```

#### <a name="edit-in-logic-apps-designer"></a>在逻辑应用设计器中编辑

1. 在“For each”操作的右上角选择省略号 (...) 按钮，然后选择“设置”。

2. 在“并发控制”下，将“并发控制”设置为“开启”。 

3. 将“并行度”滑块拖到数字 `1`。 

<a name="asynchronous-patterns"></a>

### <a name="run-actions-synchronously"></a>同步运行操作

默认情况下，所有基于 HTTP 的操作都遵循标准异步操作模式。 此模式指定当基于 HTTP 的操作向指定终结点发送请求时，远程服务器发回“202 ACCEPTED”响应。 此回复意味着服务器已接受请求以便进行处理。 逻辑应用引擎持续检查响应的位置标头所指定的 URL，直至处理停止（即为任何非 202 响应）。

但是，由于请求具有超时限制，因此，对于长时间运行的操作，可通过在操作的输入下添加 `operationOptions` 属性并将该属性设为 `DisableAsyncPattern` 来禁用异步行为。
  
```json
"<some-long-running-action>": {
   "type": "Http",
   "inputs": { "<action-inputs>" },
   "operationOptions": "DisableAsyncPattern",
   "runAfter": {}
}
```

<a name="run-high-throughput-mode"></a>

### <a name="run-in-high-throughput-mode"></a>在高吞吐量模式下运行

对于单个逻辑应用运行，每 5 分钟执行的操作数具有[默认限制](../logic-apps/logic-apps-limits-and-config.md#throughput-limits)。 若要将此限制提高到可能的[最大值](../logic-apps/logic-apps-limits-and-config.md#throughput-limits)，请将 `operationOptions` 属性设为 `OptimizedForHighThroughput`。 此设置将逻辑应用置于“高吞吐量”模式之中。 

> [!NOTE]
> 高吞吐量模式处于预览状态。 此外，还可根据需要在多个逻辑应用之间分配工作负荷。

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": { "<action-inputs>" },
   "operationOptions": "OptimizedForHighThroughput",
   "runAfter": {}
}
```

<a name="connector-authentication"></a>

## <a name="authenticate-http-triggers-and-actions"></a>对 HTTP 触发器和操作进行身份验证

HTTP 终结点支持不同类型的身份验证。 可为以下 HTTP 触发器和操作设置身份验证：

* [HTTP](../connectors/connectors-native-http.md)
* [HTTP + Swagger](../connectors/connectors-native-http-swagger.md)
* [HTTP Webhook](../connectors/connectors-native-webhook.md)

下面是可以设置的身份验证类型：

* [基本身份验证](#basic-authentication)
* [客户端证书身份验证](#client-certificate-authentication)
* [Azure Active Directory (Azure AD) OAuth 身份验证](#azure-active-directory-oauth-authentication)

> [!IMPORTANT]
> 确保保护逻辑应用工作流定义所处理的任何敏感信息。 使用安全的参数，并根据需要对数据进行编码。 有关使用和保护参数的详细信息，请[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

<a name="basic-authentication"></a>

### <a name="basic-authentication"></a>基本身份验证

对于使用 Azure Active Directory 的[基本身份验证](../active-directory-b2c/active-directory-b2c-custom-rest-api-netfw-secure-basic.md)，触发器或操作定义可以包括 `authentication` JSON 对象，它具有下表指定的属性。 要在运行时访问参数值，可以使用[工作流定义语言](https://aka.ms/logicappsdocs)提供的 `@parameters('parameterName')` 表达式。 

| 属性 | 需要 | Value | 描述 | 
|----------|----------|-------|-------------| 
| type | 是 | "Basic" | 要使用的身份验证类型，此处为“Basic” | 
| **username** | 是 | "@parameters('userNameParam')" | 用于对目标服务终结点访问进行身份验证的用户名 |
| **password** | 是 | "@parameters('passwordParam')" | 用于对目标服务终结点访问进行身份验证的密码 |
||||| 

在此示例 HTTP 操作定义中，`authentication` 节指定 `Basic` 身份验证。 有关使用和保护参数的详细信息，请[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "Basic",
         "username": "@parameters('userNameParam')",
         "password": "@parameters('passwordParam')"
      }
  },
  "runAfter": {}
}
```

> [!IMPORTANT]
> 确保保护逻辑应用工作流定义所处理的任何敏感信息。 使用安全的参数，并根据需要对数据进行编码。 有关保护参数的详细信息，请参阅[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

<a name="client-certificate-authentication"></a>

### <a name="client-certificate-authentication"></a>客户端证书身份验证

对于使用 Azure Active Directory 的[基于证书的身份验证](../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md)，触发器或操作定义可以包括 `authentication` JSON 对象，它具有下表指定的属性。 要在运行时访问参数值，可以使用[工作流定义语言](https://aka.ms/logicappsdocs)提供的 `@parameters('parameterName')` 表达式。 有关可以使用的客户端证书数的限制，请参阅 [Azure 逻辑应用的限制和配置](../logic-apps/logic-apps-limits-and-config.md)。

| 属性 | 需要 | Value | 描述 |
|----------|----------|-------|-------------|
| type | 是 | "ClientCertificate" | 安全套接字层 (SSL) 客户端证书使用的身份验证类型。 虽然支持自签名证书，但不支持用于 SSL 的自签名证书。 |
| **pfx** | 是 | "@parameters('pfxParam') | 个人信息交换 (PFX) 文件中的 base64 编码内容 |
| **password** | 是 | "@parameters('passwordParam')" | 用于访问 PFX 文件的密码 |
||||| 

在此示例 HTTP 操作定义中，`authentication` 节指定 `ClientCertificate` 身份验证。 有关使用和保护参数的详细信息，请[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "ClientCertificate",
         "pfx": "@parameters('pfxParam')",
         "password": "@parameters('passwordParam')"
      }
   },
   "runAfter": {}
}
```

> [!IMPORTANT]
> 确保保护逻辑应用工作流定义所处理的任何敏感信息。 使用安全的参数，并根据需要对数据进行编码。 有关保护参数的详细信息，请参阅[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

<a name="azure-active-directory-oauth-authentication"></a>

### <a name="azure-active-directory-ad-oauth-authentication"></a>Azure Active Directory (AD) OAuth 身份验证

对于 [Azure AD OAuth 身份验证](../active-directory/develop/authentication-scenarios.md)，触发器或操作定义可以包括 `authentication` JSON 对象，它具有下表指定的属性。 要在运行时访问参数值，可以使用[工作流定义语言](https://aka.ms/logicappsdocs)提供的 `@parameters('parameterName')` 表达式。

| 属性 | 需要 | Value | 描述 |
|----------|----------|-------|-------------|
| type | 是 | `ActiveDirectoryOAuth` | 要使用的身份验证类型，即“ActiveDirectoryOAuth”（代表 Azure AD OAuth） |
| **authority** | 否 | <*URL-for-authority-token-issuer*> | 提供身份验证令牌的颁发机构的 URL |
| **tenant** | 是 | <*tenant-ID*> | Azure AD 租户的租户 ID |
| **audience** | 是 | <*resource-to-authorize*> | 要用于授权的资源，例如 `https://management.core.windows.net/` |
| **clientId** | 是 | <*client-ID*> | 请求授权的应用的客户端 ID |
| **credentialType** | 是 | “Certificate”或“Secret” | 客户端用来请求授权的凭据类型。 此属性和值不会显示在基础定义中，但确定了凭据类型的所需参数。 |
| **pfx** | 是（仅适用于 "Certificate" 凭据类型） | "@parameters('pfxParam') | 个人信息交换 (PFX) 文件中的 base64 编码内容 |
| **password** | 是（仅适用于 "Certificate" 凭据类型） | "@parameters('passwordParam')" | 用于访问 PFX 文件的密码 |
| **secret** | 是（仅适用于 "Secret" 凭据类型） | “@parameters('secretParam')” | 用于请求授权的客户端密码 |
|||||

在此示例 HTTP 操作定义中，`authentication` 节指定 `ActiveDirectoryOAuth` 身份验证和“Secret”凭据类型。 有关使用和保护参数的详细信息，请[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "ActiveDirectoryOAuth",
         "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47",
         "audience": "https://management.core.windows.net/",
         "clientId": "34750e0b-72d1-4e4f-bbbe-664f6d04d411",
         "secret": "@parameters('secretParam')"
     }
   },
   "runAfter": {}
}
```

> [!IMPORTANT]
> 确保保护逻辑应用工作流定义所处理的任何敏感信息。 使用安全的参数，并根据需要对数据进行编码。 有关保护参数的详细信息，请参阅[保护逻辑应用](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)。

## <a name="next-steps"></a>后续步骤

* 详细了解[工作流定义语言](../logic-apps/logic-apps-workflow-definition-language.md)
