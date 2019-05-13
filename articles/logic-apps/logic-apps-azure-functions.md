---
title: 添加和使用 Azure Functions 的 Azure 逻辑应用中运行代码
description: 添加和使用 Azure Functions 的 Azure 逻辑应用中运行代码
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.topic: article
ms.date: 08/20/2018
ms.reviewer: klam, LADocs
ms.openlocfilehash: e371a6abe32a1a41d3babeaa27aaec3e30bd3323
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142311"
---
# <a name="add-and-run-code-by-using-azure-functions-in-azure-logic-apps"></a>添加并在 Azure 逻辑应用中使用 Azure Functions 运行代码

当你想要运行逻辑应用中执行特定的代码时，可以创建您自己的函数与[Azure Functions](../azure-functions/functions-overview.md)。 此服务可帮助您创建 Node.js， C#，和F#代码，使你无需构建一个完整的应用或运行代码的基础结构。 还能[从 Azure Functions 内部调用逻辑应用](#call-logic-app)。
Azure Functions 在云中提供无服务器计算，且对执行任务非常有用，如以下示例：

* 通过 Node.js 或 C# 函数扩展逻辑应用的行为。
* 在逻辑应用工作流中执行计算。
* 在逻辑应用中应用高级格式设置或计算字段。

若要运行而无需创建 Azure 函数代码片段，了解如何[添加并运行内联代码](../logic-apps/logic-apps-add-run-inline-code.md)。

## <a name="prerequisites"></a>必备组件

* Azure 订阅。 如果没有 Azure 订阅，请[注册一个免费 Azure 帐户](https://azure.microsoft.com/free/)。

* 一个 Azure 函数应用，它是 Azure Functions 和你的 Azure 函数的容器。 若没有函数应用，请先[创建函数应用](../azure-functions/functions-create-first-azure-function.md)。 然后才可以在逻辑应用设计器中[在逻辑应用外部单独](#create-function-external)创建函数或[从逻辑应用内部](#create-function-designer)创建函数。

  无论现有函数和新函数，在使用逻辑应用方面都具有相同的要求：

  * 函数应用具备的 Azure 订阅必须与逻辑应用相同。

  * 函数使用 HTTP 触发器，例如适用于 JavaScript 或 C# 的 HTTP 触发器函数模板。 

    此 HTTP 触发器模板可从逻辑应用接受具有 `application/json` 类型的内容。 
    将 Azure 函数添加到逻辑应用中时，逻辑应用设计器在 Azure 订阅内显示基于此模板创建的自定义函数。 

  * 函数不会使用自定义路由，除非定义了 [OpenAPI 定义](../azure-functions/functions-openapi-definition.md)（以前称为 [Swagger 文件](https://swagger.io/)）。 
  
  * 如果已对函数定义了 OpenAPI 定义，逻辑应用设计器会提供更丰富的函数参数使用体验。 在逻辑应用查找并访问具有 OpenAPI 定义的函数之前，请先[按以下步骤设置函数应用](#function-swagger)。

* 要在其中添加函数的逻辑应用（加入[触发器](../logic-apps/logic-apps-overview.md#logic-app-concepts)是逻辑应用中的第一步） 

  在添加可运行函数的操作之前，必须使用触发器启动逻辑应用。

  如果不熟悉逻辑应用，请查看[什么是 Azure 逻辑应用](../logic-apps/logic-apps-overview.md)和[快速入门：创建第一个逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

<a name="create-function-external"></a>

## <a name="create-functions-outside-logic-apps"></a>在逻辑应用外部创建函数

在 [Azure 门户](https://portal.azure.com)中创建 Azure 函数应用，该应用必须具备与逻辑应用一样的 Azure 订阅，然后创建 Azure 函数。
若是初次接触创建 Azure Functions，则需要了解如何[在 Azure 门户中创建第一个函数](../azure-functions/functions-create-first-azure-function.md)，不过请注意有关创建可从逻辑应用调用的函数的要求：

* 请确保选择适用于 JavaScript 或 C# 的 HTTP 触发器函数模板。

  ![HTTP 触发器 - JavaScript 或 C#](./media/logic-apps-azure-functions/http-trigger-function.png)

<a name="function-swagger"></a>

* （可选）若为函数[生成了 API 定义](../azure-functions/functions-openapi-definition.md)（以前称为 [Swagger 文件](https://swagger.io/)），则在逻辑应用设计器中使用函数参数时能得到更丰富的体验。 若要设置函数应用以使其可查找和使用具备 Swagger 描述的函数，请按照以下步骤操作：

  1. 确保函数应用正在运行。

  2. 按照以下步骤，在函数应用中，设置[跨源资源共享 (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) 以便允许所有来源：

     1. 在“函数应用”列表中，选择函数应用，然后选择“平台功能”>“CORS”。

        ![选择函数应用 >“平台功能”>“CORS”](./media/logic-apps-azure-functions/function-platform-features-cors.png)

     2. 在“CORS”下，添加  **通配符，但删除列表中的所有其他来源并选择“保存”**`*`。

        ![将 CORS 设为通配符“*”](./media/logic-apps-azure-functions/function-platform-features-cors-origins.png)

### <a name="access-property-values-inside-http-requests"></a>访问 HTTP 请求中的属性值

Webhook 函数能接受 HTTP 请求作为输入，并将这些请求传递至其他函数。 例如，尽管逻辑应用具有[转换 DateTime 值的函数](../logic-apps/workflow-definition-language-functions-reference.md)，但此基本示例 JavaScript 函数演示了如何访问传递给函数的请求对象中的属性，并对该属性值执行操作。 为访问对象内的属性，此示例使用[点 (.) 运算符](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Property_accessors)： 

```javascript
function convertToDateString(request, response){
   var data = request.body;
   response = {
      body: data.date.ToDateString();
   }
}
```

下面是此函数内部发生的情况：

1. 该函数创建 `data` 变量并将 `request` 对象内的 `body` 对象分配给该变量。 该函数使用点 (.) 运算符引用 `request` 对象内的 `body` 对象： 

   ```javascript
   var data = request.body;
   ```

2. 该函数现在可以通过 `data` 变量访问 `date` 属性，并通过调用 `ToDateString()` 函数将该属性值从 DateTime 类型转换为 DateString 类型。 该函数还通过函数响应中的 `body` 属性返回结果： 

   ```javascript
   body: data.date.ToDateString();
   ```

现已创建自己的 Azure 函数，请按步骤[将函数添加到逻辑应用](#add-function-logic-app)。

<a name="create-function-designer"></a>

## <a name="create-functions-inside-logic-apps"></a>在逻辑应用内部创建函数

若要在逻辑应用编辑器中从逻辑应用内部创建 Azure 函数，必须先具备一个 Azure 函数应用，这是函数的容器。 若没有函数应用，请先创建一个。 请参阅[在 Azure 门户中创建第一个函数](../azure-functions/functions-create-first-azure-function.md)。 

1. 在 [Azure 门户](https://portal.azure.com)的逻辑应用设计器中打开逻辑应用。 

2. 按照适用于自身方案的步骤，创建并添加函数：

   * 如果处于逻辑应用工作流最后一步，请选择“新步骤”。

   * 如果介于逻辑应用工作流中现有步骤之间，请将鼠标移至箭头上，选择加号 (+)，然后选择“添加操作”。

3. 在搜索框中输入“Azure Functions”作为筛选器。
在操作列表中选择此操作：**选择 Azure 函数 - Azure Functions** 

   ![找到“Azure Functions”](./media/logic-apps-azure-functions/find-azure-functions-action.png)

4. 从函数应用列表中选择自己的函数应用。 操作列表打开后，选择此操作：**Azure 函数-创建新的函数**

   ![选择函数应用](./media/logic-apps-azure-functions/select-function-app-create-function.png)

5. 在函数定义编辑器中定义函数：

   1. 在“函数名称”框中提供函数的名称。 

   2. 在“代码”框中，将代码添加至函数模板，包括函数运行完成后想返回至逻辑应用的响应和有效负载。 

      ![定义函数](./media/logic-apps-azure-functions/function-definition.png)

      模板代码中的 `context` 对象表示逻辑应用通过后续步骤中“请求正文”字段发送的消息。 
      要从函数内访问 `context` 对象的属性，请使用如下语法： 

      `context.body.<property-name>`

      例如，要引用 `context` 对象内的 `content` 属性，请使用如下语法： 

      `context.body.content`

      此模板代码还包含一个 `input` 变量，此变量存储来自 `data` 参数的值，因此函数可对该值执行操作。 
      在 JavaScript 函数内，`data` 变量也是 `context.body` 的一种快捷方式。

      > [!NOTE]
      > 此处的 `body` 属性适用于 `context` 对象，但与来自操作输出的 Body 令牌不同，你可能也会希望将后者传递到函数。 
 
   3. 完成后，选择“创建”。

6. 在“请求正文”框中，提供函数的输入，其格式必须为 JavaScript 对象表示法 (JSON) 对象。 

   此输入是逻辑应用发送到函数的上下文对象或消息。 点击“请求正文”字段时，界面会显示动态内容列表，以便你可以为先前步骤中的输出选择令牌。 本示例指定上下文有效负载包含一个名为 `content` 的属性，该属性具有来自电子邮件触发器的 From 令牌的值：

   ![“请求正文”示例 - 上下文对象有效负载](./media/logic-apps-azure-functions/function-request-body-example.png)

   此处的上下文对象没有强制转换为字符串，因此对象的内容被直接添加到 JSON 有效负载中。 但是，如果该上下文对象不是传递字符串、JSON 对象或 JSON 数组的 JSON 令牌，则会出现错误。 因此，假如本示例使用 Received Time 令牌，则可通过添加双引号将此上下文对象强制转换为字符串：  

   ![将对象强制转换为字符串](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

7. 若要指定其他详细信息，例如要使用的方法、请求标头或查询参数，请选择“显示高级选项”。

<a name="add-function-logic-app"></a>

## <a name="add-existing-functions-to-logic-apps"></a>将现有函数添加至逻辑应用

若要从逻辑应用调用现有 Azure 函数，可以添加 Azure 函数，具体方法与在逻辑应用设计器中执行的任何其他操作一样。 

1. 在 [Azure 门户](https://portal.azure.com)的逻辑应用设计器中打开逻辑应用。 

2. 在要添加函数的步骤下，选择“新建步骤” > “添加操作”。 

3. 在搜索框中输入“Azure Functions”作为筛选器。
在操作列表中选择此操作：**选择 Azure 函数 - Azure Functions** 

   ![找到“Azure Functions”](./media/logic-apps-azure-functions/find-azure-functions-action.png)

4. 从函数应用列表中选择自己的函数应用。 在显示的函数列表中选择函数。 

   ![选择函数应用和 Azure 函数](./media/logic-apps-azure-functions/select-function-app-existing-function.png)

   对于具备 API 定义（Swagger 描述）的函数以及那些[设置为可供逻辑应用查找和访问](#function-swagger)的函数，可以选择“Swagger 操作”:

   ![选择函数应用“Swagger 操作”以及 Azure 函数](./media/logic-apps-azure-functions/select-function-app-existing-function-swagger.png)

5. 在“请求正文”框中，提供函数的输入，其格式必须为 JavaScript 对象表示法 (JSON) 对象。 

   此输入是逻辑应用发送到函数的上下文对象或消息。 点击“请求正文”字段时，界面会显示动态内容列表，以便你可以为先前步骤中的输出选择令牌。 本示例指定上下文有效负载包含一个名为 `content` 的属性，该属性具有来自电子邮件触发器的 From 令牌的值：

   ![“请求正文”示例 - 上下文对象有效负载](./media/logic-apps-azure-functions/function-request-body-example.png)

   此处的上下文对象没有强制转换为字符串，因此对象的内容被直接添加到 JSON 有效负载中。 但是，如果该上下文对象不是传递字符串、JSON 对象或 JSON 数组的 JSON 令牌，则会出现错误。 因此，假如本示例使用 Received Time 令牌，则可通过添加双引号将此上下文对象强制转换为字符串： 

   ![将对象强制转换为字符串](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

6. 若要指定其他详细信息，例如要使用的方法、请求标头或查询参数，请选择“显示高级选项”。

<a name="call-logic-app"></a>

## <a name="call-logic-apps-from-functions"></a>从函数调用逻辑应用

若要从 Azure 函数内部触发逻辑应用，则必须使用提供可调用终结点的触发器启动该逻辑应用。 例如，可使用 HTTP 触发器、请求触发器、Azure 队列触发器或事件网格触发器启动逻辑应用。 在函数内，向触发器的 URL 发送一个 HTTP POST 请求，并加入需要逻辑应用处理的有效负载。 有关详细信息，请参阅[调用、触发器或嵌套逻辑应用](../logic-apps/logic-apps-http-endpoint.md)。 

## <a name="get-support"></a>获取支持

* 有关问题，请访问 [Azure 逻辑应用论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交功能建议或对功能建议进行投票，请访问[逻辑应用用户反馈网站](https://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>后续步骤

* 了解[逻辑应用连接器](../connectors/apis-list.md)
