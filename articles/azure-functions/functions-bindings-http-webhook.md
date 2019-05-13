---
title: Azure Functions HTTP 触发器和绑定
description: 了解如何在 Azure Functions 中使用 HTTP 触发器和绑定。
services: functions
documentationcenter: na
author: craigshoemaker
manager: jeconnoc
keywords: Azure Functions, Functions, 事件处理, webhook, 动态计算, 无服务体系结构, HTTP, API, REST
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 11/21/2017
ms.author: cshoe
ms.openlocfilehash: a1d66cf4506e3b8f58572576db908812f4e2be07
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62104355"
---
# <a name="azure-functions-http-triggers-and-bindings"></a>Azure Functions HTTP 触发器和绑定

本文介绍如何在 Azure Functions 中使用 HTTP 触发器和输出绑定。

HTTP 触发器可进行自定义以响应 [Webhook](https://en.wikipedia.org/wiki/Webhook)。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

本文中的代码默认使用 Functions 2.x 语法，该语法使用 .NET Core。 有关 1.x 语法的信息，请参阅 [1.x functions 模板](https://github.com/Azure/azure-functions-templates/tree/v1.x/Functions.Templates/Templates)。

## <a name="packages---functions-1x"></a>包 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.Http](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http) NuGet 包 1.x 版本中提供了 HTTP 绑定。 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.Http) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-2x"></a>包 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Http](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http) NuGet 包 3.x 版本中提供了 HTTP 绑定。 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Http/) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package](../../includes/functions-package-auto.md)]

## <a name="trigger"></a>触发器

借助 HTTP 触发器，可以使用 HTTP 请求调用函数。 可以使用 HTTP 触发器生成无服务器 API 和响应 Webhook。

默认情况下，在 Functions 1.x 中，HTTP 触发器返回“HTTP 200 正常”和空的正文；在 Functions 2.x 中返回“HTTP 204 无内容”和空的正文。 若要修改该响应，请配置 [HTTP 输出绑定](#output)。

## <a name="trigger---example"></a>触发器 - 示例

参阅语言特定的示例：

* [C#](#trigger---c-example)
* [C# 脚本 (.csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [Java](#trigger---java-examples)
* [JavaScript](#trigger---javascript-example)
* [Python](#trigger---python-example)

### <a name="trigger---c-example"></a>触发器 - C# 示例

以下示例显示一个在查询字符串或 HTTP 请求正文中查找 `name` 参数的 [C# 函数](functions-dotnet-class-library.md)。 请注意，返回值用于输出绑定，但不需要返回值属性。

```cs
[FunctionName("HttpTriggerCSharp")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] 
    HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

### <a name="trigger---c-script-example"></a>触发器 - C# 脚本示例

以下示例演示 function.json 文件中的一个触发器绑定以及使用该绑定的 [C# 脚本函数](functions-reference-csharp.md)。 该函数在查询字符串或 HTTP 请求的正文中查找 `name` 参数。

function.json 文件如下所示：

```json
{
    "disabled": false,
    "bindings": [
        {
            "authLevel": "function",
            "name": "req",
            "type": "httpTrigger",
            "direction": "in",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "name": "$return",
            "type": "http",
            "direction": "out"
        }
    ]
}
```

[配置](#trigger---configuration)部分解释了这些属性。

下面是绑定到 `HttpRequest` 的 C# 脚本代码：

```cs
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

可以绑定到自定义对象而不是 `HttpRequest`。 此对象从请求正文进行创建，并分析为 JSON。 同样，可以将类型传递给 HTTP 响应输出绑定，并作为响应正文随 200 状态代码一同返回。

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static string Run(Person person, ILogger log)
{   
    return person.Name != null
        ? (ActionResult)new OkObjectResult($"Hello, {person.Name}")
        : new BadRequestObjectResult("Please pass an instance of Person.");
}

public class Person {
     public string Name {get; set;}
}
```

### <a name="trigger---f-example"></a>触发器 - F# 示例

以下示例演示 *function.json* 文件中的一个触发器绑定以及使用该绑定的 [F# 函数](functions-reference-fsharp.md)。 该函数在查询字符串或 HTTP 请求的正文中查找 `name` 参数。

function.json 文件如下所示：

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[配置](#trigger---configuration)部分解释了这些属性。

F# 代码如下所示：

```fsharp
open System.Net
open System.Net.Http
open FSharp.Interop.Dynamic

let Run(req: HttpRequestMessage) =
    async {
        let q =
            req.GetQueryNameValuePairs()
                |> Seq.tryFind (fun kv -> kv.Key = "name")
        match q with
        | Some kv ->
            return req.CreateResponse(HttpStatusCode.OK, "Hello " + kv.Value)
        | None ->
            let! data = Async.AwaitTask(req.Content.ReadAsAsync<obj>())
            try
                return req.CreateResponse(HttpStatusCode.OK, "Hello " + data?name)
            with e ->
                return req.CreateErrorResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
    } |> Async.StartAsTask
```

需要一个使用 NuGet 引用 `FSharp.Interop.Dynamic` 和 `Dynamitey` 程序集的 `project.json` 文件，如以下示例中所示：

```json
{
  "frameworks": {
    "net46": {
      "dependencies": {
        "Dynamitey": "1.0.2",
        "FSharp.Interop.Dynamic": "3.0.0"
      }
    }
  }
}
```

### <a name="trigger---javascript-example"></a>触发器 - JavaScript 示例

以下示例演示 function.json 文件中的一个触发器绑定以及使用该绑定的 [JavaScript 函数](functions-reference-node.md)。 该函数在查询字符串或 HTTP 请求的正文中查找 `name` 参数。

function.json 文件如下所示：

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

[配置](#trigger---configuration)部分解释了这些属性。

JavaScript 代码如下所示：

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

### <a name="trigger---python-example"></a>触发器 - Python 示例

以下示例演示 function.json 文件中的一个触发器绑定以及使用该绑定的 [Python 函数](functions-reference-python.md)。 该函数在查询字符串或 HTTP 请求的正文中查找 `name` 参数。

function.json 文件如下所示：

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

[配置](#trigger---configuration)部分解释了这些属性。

下面是 Python 代码：

```python
import logging
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

### <a name="trigger---java-examples"></a>触发器 - Java 示例

* [从查询字符串中读取参数](#read-parameter-from-the-query-string-java)
* [从 POST 请求中读取正文](#read-body-from-a-post-request-java)
* [从路由中读取参数](#read-parameter-from-a-route-java)
* [从 POST 请求中读取 POJO 正文](#read-pojo-body-from-a-post-request-java)

下面的示例说明 function.json 文件中的 HTTP 触发器绑定，以及使用该绑定的相应 [Java 函数](functions-reference-java.md)。 

function.json 文件如下所示：

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "anonymous",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

#### <a name="read-parameter-from-the-query-string-java"></a>从查询字符串中读取参数 (Java)  

以下示例从查询字符串中读取名为 ```id``` 的参数，然后使用该参数构建返回到客户端的 JSON 文档（内容类型为 ```application/json```）。 

```java
    @FunctionName("TriggerStringGet")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.GET}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("GET parameters are: " + request.getQueryParameters());

        // Get named parameter
        String id = request.getQueryParameters().getOrDefault("id", "");

        // Convert and display
        if (id.isEmpty()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String name = "fake_name";
            final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                         "\"description\": \"" + name + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-body-from-a-post-request-java"></a>从 POST 请求中读取正文 (Java)  

以下示例将 POST 请求的正文读取为 ```String```，然后使用该正文构建返回到客户端的 JSON 文档（内容类型为 ```application/json```）。

```java
    @FunctionName("TriggerStringPost")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.POST}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Request body is: " + request.getBody().orElse(""));

        // Check request body
        if (!request.getBody().isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String body = request.getBody().get();
            final String jsonDocument = "{\"id\":\"123456\", " + 
                                         "\"description\": \"" + body + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-parameter-from-a-route-java"></a>从路由中读取参数 (Java)  

此示例读取名为 ```id``` 的必选参数，并从路由路径中读取可选参数 ```name```，然后使用这两个参数构建返回到客户端的 JSON 文档（内容类型为 ```application/json```）。 T

```java
    @FunctionName("TriggerStringRoute")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.GET}, 
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "trigger/{id}/{name=EMPTY}") // name is optional and defaults to EMPTY
            HttpRequestMessage<Optional<String>> request,
            @BindingName("id") String id,
            @BindingName("name") String name,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Route parameters are: " + id);

        // Convert and display
        if (id == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                         "\"description\": \"" + name + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-pojo-body-from-a-post-request-java"></a>从 POST 请求中读取 POJO 正文 (Java)  

下面是此示例中引用的 ```ToDoItem``` 类的代码：

```java

public class ToDoItem {

  private String id;
  private String description;  

  public ToDoItem(String id, String description) {
    this.id = id;
    this.description = description;
  }

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }
  
  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}

```

此示例读取 POST 请求的正文。 该请求正文会自动反序列化为 ```ToDoItem``` 对象，然后以内容类型 ```application/json``` 返回到客户端。 当 ```ToDoItem``` 参数分配给 ```HttpMessageResponse.Builder``` 类的 ```body``` 属性时，它会由 Functions 运行时序列化。

```java
    @FunctionName("TriggerPojoPost")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.POST}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<ToDoItem>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Request body is: " + request.getBody().orElse(null));

        // Check request body
        if (!request.getBody().isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final ToDoItem body = request.getBody().get();
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(body)
                          .build();
        }
    }
```

## <a name="trigger---attributes"></a>触发器 - 特性

在 [C# 类库](functions-dotnet-class-library.md)中，使用 [HttpTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions.Http/HttpTriggerAttribute.cs) 特性。

可以在特性构造函数参数中设置授权级别和允许的 HTTP 方法，Webhook 类型和路由模板有相应的属性。 有关这些设置的详细信息，请参阅[触发器 - 配置](#trigger---configuration)。 下面是某个方法签名中的 `HttpTrigger` 特性：

```csharp
[FunctionName("HttpTriggerCSharp")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequest req)
{
    ...
}
```

有关完整示例，请参阅[触发器 - C# 示例](#trigger---c-example)。

## <a name="trigger---configuration"></a>触发器 - 配置

下表解释了在 function.json 文件和 `HttpTrigger` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
| **type** | 不适用| 必需 - 必须设置为 `httpTrigger`。 |
| direction | 不适用| 必需 - 必须设置为 `in`。 |
| name | 不适用| 必需 - 在请求或请求正文的函数代码中使用的变量名称。 |
| <a name="http-auth"></a>**authLevel** |  AuthLevel |确定请求中需要提供的密钥（如果有），以便调用此函数。 授权级别可以是以下值之一： <ul><li><code>anonymous</code>&mdash;无需 API 密钥。</li><li><code>function</code>&mdash;特定于函数的 API 密钥是必需的。 如果未提供任何值，该值为默认值。</li><li><code>admin</code>&mdash;无需主密钥。</li></ul> 有关详细信息，请参阅有关[授权密钥](#authorization-keys)的部分。 |
| methods |方法 | HTTP 方法的数组，该函数将响应此方法。 如果未指定，该函数将响应所有 HTTP 方法。 参阅[自定义 HTTP 终结点](#customize-the-http-endpoint)。 |
| route | Route | 定义路由模板，控制函数将响应的请求 URL。 如果未提供任何值，则默认值为 `<functionname>`。 有关详细信息，请参阅[自定义 HTTP 终结点](#customize-the-http-endpoint)。 |
| webHookType | WebHookType | 仅支持 1.x 版运行时。<br/><br/>将 HTTP 触发器配置为充当指定提供程序的 [webhook](https://en.wikipedia.org/wiki/Webhook) 接收器。 如果未设置此属性，请不要设置 `methods` 属性。 Webhook 类型可以是以下值之一：<ul><li><code>genericJson</code>&mdash;不包含特定提供程序逻辑的常规用途 webhook 终结点。 此设置会将请求限制为仅请求使用 HTTP POST 以及内容类型为 `application/json`。</li><li><code>github</code>&mdash;该函数响应 [GitHub Webhook](https://developer.github.com/webhooks/)。 不要对 GitHub Webhook 使用 authLevel 属性。 有关详细信息，请参阅本文后面的“GitHub Webhook”部分。</li><li><code>slack</code>&mdash;该函数响应 [Slack Webhook](https://api.slack.com/outgoing-webhooks)。 不要对 Slack Webhook 使用 authLevel 属性。 有关详细信息，请参阅本文后面的“Slack Webhook”部分。</li></ul>|

## <a name="trigger---usage"></a>触发器 - 用法

对于 C# 和 F # 函数，可以将触发器输入的类型声明为 `HttpRequest` 或自定义类型。 如果选择 `HttpRequest`，会获得对请求对象的完全访问权限。 对于自定义类型，运行时会尝试分析 JSON 请求正文，以设置对象属性。

对于 JavaScript 函数，Functions 运行时提供请求正文而不是请求对象。 有关详细信息，请参阅 [JavaScript 触发器示例](#trigger---javascript-example)。

### <a name="customize-the-http-endpoint"></a>自定义 HTTP 终结点

默认情况下，创建 HTTP 触发器的函数时，可通过以下格式的路由对该函数进行寻址：

    http://<yourapp>.azurewebsites.net/api/<funcname>

在 HTTP 触发器的输入绑定中，可以使用可选 `route` 属性自定义此路由。 例如，以下 function.json 文件定义了 HTTP 触发器的 `route` 属性：

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

通过此配置，现在可以通过以下路由对该函数进行寻址，而不是通过原始路由寻址。

```
http://<yourapp>.azurewebsites.net/api/products/electronics/357
```

这使得函数代码可以支持地址中的两个参数：“category”和“id”。可以将任何 [Web API 路由约束](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints)与参数配合使用。 以下 C# 函数代码使用了这两个参数。

```csharp
public static Task<IActionResult> Run(HttpRequest req, string category, int? id, ILogger log)
{
    if (id == null)
    {
        return (ActionResult)new OkObjectResult($"All {category} items were requested.");
    }
    else
    {
        return (ActionResult)new OkObjectResult($"{category} item with id = {id} has been requested.");
    }
    
    // -----
    log.LogInformation($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
}
```

以下是使用相同路由参数的 Node.js 函数代码。

```javascript
module.exports = function (context, req) {

    var category = context.bindingData.category;
    var id = context.bindingData.id;

    if (!id) {
        context.res = {
            // status defaults to 200 */
            body: "All " + category + " items were requested."
        };
    }
    else {
        context.res = {
            // status defaults to 200 */
            body: category + " item with id = " + id + " was requested."
        };
    }

    context.done();
}
```

默认情况下，所有函数路由的前缀均为 api。 还可以使用 [host.json](functions-host-json.md) 文件中的 `http.routePrefix` 属性自定义或删除前缀。 以下示例通过将空字符串用于 host.json 文件中的前缀删除 api 路由前缀。

```json
{
    "http": {
    "routePrefix": ""
    }
}
```

### <a name="working-with-client-identities"></a>使用客户端标识

如果函数应用使用[应用服务身份验证/授权](../app-service/overview-authentication-authorization.md)，则可通过代码查看有关已验证身份的客户端的信息。 此信息以[平台注入的请求标头](../app-service/app-service-authentication-how-to.md#access-user-claims)的形式提供。 

还可从绑定数据中读取此信息。 此功能仅可用于 Functions 2.x 运行时， 而且它目前仅可用于 .NET 语言。

在 .NET 语言中，此信息以 [ClaimsPrincipal](https://docs.microsoft.com/dotnet/api/system.security.claims.claimsprincipal?view=netstandard-2.0) 的形式提供。 ClaimsPrincipal 作为请求上下文的一部分提供，如以下示例中所示：

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

或者，可直接将 ClaimsPrincipal 作为其他参数包含在函数签名中：

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}

```

### <a name="authorization-keys"></a>授权密钥

Functions 允许使用密钥使其难以在开发过程中访问 HTTP 函数终结点。  标准 HTTP 触发器可能会要求在请求中出现此类 API 密钥。 

> [!IMPORTANT]
> 虽然密钥可以帮助你在开发过程中对 HTTP 终结点进行模糊处理，它们不应作为一种方法来保护生产环境中的 HTTP 触发器。 若要了解详细信息，请参阅[在生产环境中保护 HTTP 终结点](#secure-an-http-endpoint-in-production)。

> [!NOTE]
> 在 Functions 1.x 运行时中，Webhook 提供程序可以使用密钥以多种方式对请求授权，具体取决于提供程序支持何种方式。 [Webhook 和密钥](#webhooks-and-keys)对此进行了说明。 2.x 版运行时不包括对 Webhook 提供程序的内置支持。

有两种类型的密钥：

* **主机密钥**：由 Function App 中的所有函数共享这些密钥。 这些密钥用作 API 密钥时，可以访问 Function App 中的任何函数。
* **函数密钥**：这些密钥仅适用于在其下定义它们的特定函数。 这些密钥用作 API 密钥时，只允许访问该函数。

命名每个密钥方便引用，并且在函数和主机级别存在名为“default”的默认密钥。 函数密钥优先于主机密钥。 如果为两个密钥定义的名称相同，则使用函数密钥。

每个函数应用也有一个特殊的主密钥。 此密钥是名为 `_master` 的主机密钥，提供对运行时 API 的管理访问权限。 无法撤消此密钥。 当设置 `admin` 授权级别，请求必须使用主密钥；任何其他密钥会导致授权失败。

> [!CAUTION]  
> 由于函数应用中提升的权限由主密钥所授予，因此不应与第三方共享此密钥或在本机客户端应用程序中分发此密钥。 选择管理员授权级别时，请务必审慎行事。

### <a name="obtaining-keys"></a>获取密钥

密钥作为 Function App 的一部分存储在 Azure 中，并进行了静态加密。 若要查看密钥，请创建新的密钥或将密钥滚动到新值，导航到 [Azure 门户](https://portal.azure.com)中的某个 HTTP 触发函数并选择“管理”。

![在门户中管理函数密钥。](./media/functions-bindings-http-webhook/manage-function-keys.png)

没有任何受支持的 API 可用于以编程方式获取函数密钥。

### <a name="api-key-authorization"></a>API 密钥的授权

大多数 HTTP 触发器模板都需要在请求中获取 API 密钥。 因此 HTTP 请求通常如以下 URL 所示：

    https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>

该密钥可以包含在名为 `code` 的查询字符串变量中（如上所示）。 也可以包含在 `x-functions-key` HTTP 头中。 密钥的值可以为任意为函数定义的函数密钥，也可以为任意主机密钥。

可以允许匿名请求，它不需要密钥。 可能还需要使用主密钥。 可使用绑定 JSON 中的 `authLevel` 属性更改默认授权级别。 有关详细信息，请参阅[触发器 - 配置](#trigger---configuration)。

> [!NOTE]
> 本地运行函数时，将禁用授权，而不考虑指定的身份验证级别设置。 发布到 Azure 后，将强制执行触发器中的 `authLevel` 设置。



### <a name="secure-an-http-endpoint-in-production"></a>在生产环境中保护 HTTP 终结点

若要全面保护生产环境中的函数终结点，应考虑实现以下函数应用级别的安全选项之一：

* 打开函数应用的“应用服务身份验证/授权”。 应用服务平台允许使用 Azure Active Directory (AAD) 和多个第三方标识提供者对客户端进行身份验证。 可以使用此函数来实现函数的自定义授权规则，并且可以从函数代码处理用户信息。 若要了解详细信息，请参阅 [Azure 应用服务中的身份验证和授权](../app-service/overview-authentication-authorization.md)以及[使用客户端标识](#working-with-client-identities)。

* 使用 Azure API 管理 (APIM) 对请求进行身份验证。 APIM 为传入请求提供了各种 API 安全选项。 若要了解详细信息，请参阅 [API 管理身份验证策略](../api-management/api-management-authentication-policies.md)。 有了 APIM，可以配置函数应用以接受仅来自 APIM 实例 IP 地址的请求。 若要了解详细信息，请参阅 [IP 地址限制](ip-addresses.md#ip-address-restrictions)。

* 将函数应用部署到 Azure 应用服务环境 (ASE)。 ASE 提供要在其中运行函数的专用托管环境。 ASE 允许配置单个前端网关，可以使用它对所有传入请求进行身份验证。 有关详细信息，请参阅[为应用服务环境配置 Web 应用程序防火墙 (WAF)](../app-service/environment/app-service-app-service-environment-web-application-firewall.md)。

在使用这些函数应用级别的安全方法之一时，应将 HTTP 触发函数身份验证级别设置为 `anonymous`。

### <a name="webhooks"></a>Webhook

> [!NOTE]
> Webhook 模式仅适用于 1.x 版 Functions 运行时。 进行此更改是为了提高 2.x 版中 HTTP 触发器的性能。

在 1.x 版中，Webhook 模板为 Webhook 有效负载提供了额外的验证。 在 2.x 版中，基本 HTTP 触发器仍正常工作，且是针对 Webhook 的推荐方法。 

#### <a name="github-webhooks"></a>GitHub Webhook

要响应 GitHub webhook，首先请创建包含 HTTP 触发器的函数，并将 webHookType 属性设置为 `github`。 然后将其 URL 和 API 密钥复制到 GitHub 存储库的“添加 Webhook”页。 

![](./media/functions-bindings-http-webhook/github-add-webhook.png)

#### <a name="slack-webhooks"></a>Slack Webhook

Slack webhook 为用户生成令牌，而非让用户指定它，所以必须使用 Slack 中的令牌配置特定于函数的密钥。 请参阅[授权密钥](#authorization-keys)。

### <a name="webhooks-and-keys"></a>Webhook 和密钥

Webhook 授权由属于 HTTP 触发器的 webhook 接收器组件处理，其机制因 webhook 类型而异。 每种机制都依赖于一个密钥。 默认情况下，使用名为“default”的函数密钥。 要使用其他密钥，请将 webhook 提供程序配置为使用以下方式之一的请求发送密钥名称：

* **查询字符串**：提供程序通过 `clientid` 查询字符串参数（例如，`https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?clientid=<KEY_NAME>`）传递密钥名称。
* **请求标头**：提供程序通过 `x-functions-clientid` 标头传递密钥名称。

## <a name="trigger---limits"></a>触发器 - 限制

HTTP 请求长度限制为 100 MB（104,857,600 字节），并且 URL 长度限制为 4 KB（4,096 字节）。 这些限制由运行时的 [Web.config 文件](https://github.com/Azure/azure-webjobs-sdk-script/blob/v1.x/src/WebJobs.Script.WebHost/Web.config)的 `httpRuntime` 元素指定。

如果使用 HTTP 触发器的函数未在大约 2.5 分钟内完成，网关将超时并返回 HTTP 502 错误。 该函数将继续运行，但将无法返回 HTTP 响应。 对于长时间运行的函数，我们建议你遵循异步模式，并返回可以 ping 通请求状态的位置。 有关函数可以运行多长时间的信息，请参阅[缩放和托管 - 消耗计划](functions-scale.md#consumption-plan)。

## <a name="trigger---hostjson-properties"></a>触发器 - host.json 属性

[host.json](functions-host-json.md) 文件包含控制 HTTP 触发器行为的设置。

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="output"></a>输出

通过 HTTP 输出绑定响应 HTTP 请求发送者。 此绑定需要使用 HTTP 触发器，利用此绑定，可以自定义与触发器请求相关联的响应。 如果未提供 HTTP 输出绑定，在 Functions 1.x 中，HTTP 触发器返回“HTTP 200 正常”和空的正文；在 Functions 2.x 中返回“HTTP 204 无内容”和空的正文。

## <a name="output---configuration"></a>输出 - 配置

下表解释了在 function.json 文件中设置的绑定配置属性。 在 C# 类库中，没有与这些 function.json 属性对应的属性。

|属性  |说明  |
|---------|---------|
| **type** |必须设置为 `http`。 |
| direction | 必须设置为 `out`。 |
|name | 在响应的函数代码中使用的变量名称，或者 `$return` 以使用返回值。 |

## <a name="output---usage"></a>输出 - 用法

若要发送 HTTP 响应，请使用语言标准响应模式。 在 C# 或 C# 脚本中，使函数返回类型为 `IActionResult` 或 `Task<IActionResult>`。 在 C# 中，不需要返回值属性。

有关示例响应，请参阅[触发器示例](#trigger---example)。

## <a name="next-steps"></a>后续步骤

[详细了解 Azure Functions 触发器和绑定](functions-triggers-bindings.md)
