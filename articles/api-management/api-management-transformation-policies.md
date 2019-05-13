---
title: Azure API 管理转换策略 | Microsoft Docs
description: 了解可在 Azure API 管理中使用的转换策略。
services: api-management
documentationcenter: ''
author: miaojiang
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/11/2019
ms.author: apimpm
ms.openlocfilehash: 565bcfa6c2f8a3da2ac16df0016b5adc54e27380
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407596"
---
# <a name="api-management-transformation-policies"></a>API 管理转换策略
本主题提供以下 API 管理策略的参考。 有关添加和配置策略的信息，请参阅 [API 管理中的策略](https://go.microsoft.com/fwlink/?LinkID=398186)。

##  <a name="TransformationPolicies"></a> 转换策略

-   [将 JSON 转换为 XML](api-management-transformation-policies.md#ConvertJSONtoXML) - 将请求或响应正文从 JSON 转换为 XML。

-   [将 XML 转换为 JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) - 将请求或响应正文从 XML 转换为 JSON。

-   [查找并替换正文中的字符串](api-management-transformation-policies.md#Findandreplacestringinbody) - 查找请求或响应子字符串，并将其替换为不同的子字符串。

-   [在内容中屏蔽 URL](api-management-transformation-policies.md#MaskURLSContent) - 重写（屏蔽）响应正文中的链接，使其通过网关指向等效的链接。

-   [设置后端服务](api-management-transformation-policies.md#SetBackendService) - 更改传入请求的后端服务。

-   [设置正文](api-management-transformation-policies.md#SetBody) - 设置传入和传出请求的消息正文。

-   [设置 HTTP 标头](api-management-transformation-policies.md#SetHTTPheader) - 向现有的响应和/或请求标头赋值，或者添加新的响应和/或请求标头。

-   [设置查询字符串参数](api-management-transformation-policies.md#SetQueryStringParameter) - 添加、删除请求查询字符串参数或替换其值。

-   [重写 URI](api-management-transformation-policies.md#RewriteURL) - 将请求 URL 从其公用格式转换为 Web 服务所需的格式。

-   [使用 XSLT 转换 XML](api-management-transformation-policies.md#XSLTransform) - 在请求或响应正文中将 XSL 转换应用到 XML。

##  <a name="ConvertJSONtoXML">将 JSON 转换为 XML</a>
 `json-to-xml` 策略将请求或响应正文从 JSON 转换为 XML。

### <a name="policy-statement"></a>策略语句

```xml
<json-to-xml apply="always | content-type-json" consider-accept-header="true | false" parse-date="true | false"/>
```

### <a name="example"></a>示例

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false"/>
    </outbound>
</policies>
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|json-to-xml|根元素。|“是”|

### <a name="attributes"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|应用|属性必须设置为以下值之一。<br /><br /> -   always - 始终应用转换。<br />-   content-type-json - 仅在响应的 Content-Type 标头指示存在 JSON 的情况下进行转换。|“是”|不适用|
|consider-accept-header|属性必须设置为以下值之一。<br /><br /> -   true - 如果在请求的 Accept 标头中请求了 JSON，则应用转换。<br />-   false - 始终应用转换。|“否”|true|
|parse-date|设为 `false` 时，转换时则只是简单地复制日期值|“否”|true|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站、错误时

-   **策略范围：** 全局、产品、API、操作

##  <a name="ConvertXMLtoJSON"></a> 将 XML 转换为 JSON
 `xml-to-json` 策略将请求或响应正文从 XML 转换为 JSON。 此策略可以用来根据仅用 XML 的后端 Web 服务来提升 API。

### <a name="policy-statement"></a>策略语句

```xml
<xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>
```

### <a name="example"></a>示例

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
    </outbound>
</policies>
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|xml-to-json|根元素。|“是”|

### <a name="attributes"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|kind|属性必须设置为以下值之一。<br /><br /> -   javascript-friendly - 转换后的 JSON 具有 JavaScript 开发人员熟知的形式。<br />-   direct - 转换后的 JSON 反映了原始 XML 文档的结构。|“是”|不适用|
|应用|属性必须设置为以下值之一。<br /><br /> -   always - 始终转换。<br />-   content-type-xml - 仅在响应的 Content-Type 标头指示存在 XML 的情况下进行转换。|“是”|不适用|
|consider-accept-header|属性必须设置为以下值之一。<br /><br /> -   true - 如果在请求的 Accept 标头中请求了 XML，则应用转换。<br />-   false - 始终应用转换。|“否”|true|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站、错误时

-   **策略范围：** 全局、产品、API、操作

##  <a name="Findandreplacestringinbody"></a> 在正文中查找并替换字符串
 `find-and-replace` 策略查找请求或响应子字符串并将其替换为不同的子字符串。

### <a name="policy-statement"></a>策略语句

```xml
<find-and-replace from="what to replace" to="replacement" />
```

### <a name="example"></a>示例

```xml
<find-and-replace from="notebook" to="laptop" />
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|find-and-replace|根元素。|“是”|

### <a name="attributes"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|来自|要搜索的字符串。|“是”|不适用|
|to|替换字符串。 指定一个零长度的替换字符串，以便删除搜索字符串。|“是”|不适用|

### <a name="usage"></a>使用情况
 此策略可在以下策略[节](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站、后端、错误时

-   **策略范围：** 全局、产品、API、操作

##  <a name="MaskURLSContent"></a> 在内容中屏蔽 URL
 `redirect-content-urls` 策略重写（屏蔽）响应正文中的链接，使其通过网关指向等效的链接。 在出站节中用于重写响应正文链接，使之指向网关。 在入站节中使用，以便获得相反的效果。

> [!NOTE]
>  此策略不更改任何标头值，例如如 `Location` 标头值。 若要更改标头值，请使用 [set-header](api-management-transformation-policies.md#SetHTTPheader) 策略。

### <a name="policy-statement"></a>策略语句

```xml
<redirect-content-urls />
```

### <a name="example"></a>示例

```xml
<redirect-content-urls />
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|redirect-content-urls|根元素。|“是”|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站

-   **策略范围：** 全局、产品、API、操作

##  <a name="SetBackendService"></a> 设置后端服务
 使用 `set-backend-service` 策略将传入请求重定向到一个后端，此后端不同于在 API 设置中为该操作指定的后端。 此策略将传入请求的后端服务基 URL 更改为在策略中指定的基 URL。

### <a name="policy-statement"></a>策略语句

```xml
<set-backend-service base-url="base URL of the backend service" />
```

或

```xml
<set-backend-service backend-id="identifier of the backend entity specifying base URL of the backend service" />
```

> [!NOTE]
> 后端实体可以通过管理 [API](https://docs.microsoft.com/rest/api/apimanagement/backend) 和[PowerShell](https://www.powershellgallery.com/packages?q=apimanagement) 进行管理。

### <a name="example"></a>示例

```xml
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2013-05")">
                <set-backend-service base-url="http://contoso.com/api/8.2/" />
            </when>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2014-03")">
                <set-backend-service base-url="http://contoso.com/api/9.1/" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
在此示例中，所设置的后端服务策略根据查询字符串中传递的版本值将请求路由到一个后端服务，该服务不同于在 API 中指定的服务。

后端服务基 URL 最初派生自 API 设置。 因此，请求 URL `https://contoso.azure-api.net/api/partners/15?version=2013-05&subscription-key=abcdef` 变为 `http://contoso.com/api/10.4/partners/15?version=2013-05&subscription-key=abcdef`，其中 `http://contoso.com/api/10.4/` 是在 API 设置中指定的后端服务 URL。

应用 [<choose\>](api-management-advanced-policies.md#choose) 策略语句时，后端服务基 URL 可能会再次更改为 `http://contoso.com/api/8.2` 或 `http://contoso.com/api/9.1`，具体取决于版本请求查询参数的值。 例如，如果值为 `"2013-15"`，最终请求 URL 将变为 `http://contoso.com/api/8.2/partners/15?version=2013-05&subscription-key=abcdef`。

如果需要进一步转换请求，可使用其他[转换策略](api-management-transformation-policies.md#TransformationPolicies)。 例如，在将请求路由到特定于版本的后端以后，要删除版本查询参数，可以使用[设置查询字符串参数](api-management-transformation-policies.md#SetQueryStringParameter)策略删除现在的冗余版本属性。

### <a name="example"></a>示例

```xml
<policies>
    <inbound>
        <set-backend-service backend-id="my-sf-service" sf-partition-key="@(context.Request.Url.Query.GetValueOrDefault("userId","")" sf-replica-type="primary" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
在此示例中，策略使用 userId 查询字符串作为分区键并使用该分区的主要副本，将请求路由到 Service Fabric 后端。

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|set-backend-service|根元素。|“是”|

### <a name="attributes"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|base-url|新的后端服务基 URL。|必须存在 `base-url` 或 `backend-id` 中的一个。|不适用|
|backend-id|要路由到的后端标识符。 （后端实体通过 [API](https://docs.microsoft.com/rest/api/apimanagement/backend) 和 [PowerShell](https://www.powershellgallery.com/packages?q=apimanagement) 进行管理。）|必须存在 `base-url` 或 `backend-id` 中的一个。|不适用|
|sf-partition-key|只有在后端为 Service Fabric 服务且使用“backend-id”指定时才适用。 用于从名称解析服务中解析特定分区。|“否”|不适用|
|sf-replica-type|只有在后端为 Service Fabric 服务且使用“backend-id”指定时才适用。 控制请求是否应转到分区的主要副本或次要副本。 |“否”|不适用|
|sf-resolve-condition|只有在后端为 Service Fabric 服务时才适用。 确定对 Service Fabric 后端的调用是否针对新解析重复进行的条件。|“否”|不适用|
|sf-service-instance-name|只有在后端为 Service Fabric 服务时才适用。 允许在运行时更改服务实例。 |“否”|不适用|
|sf-listener-name|只有在后端为 Service Fabric 服务且使用“backend-id”指定时才适用。 使用 Service Fabric Reliable Services 可在服务中创建多个侦听器。 当后端可靠服务具有多个侦听器时，此属性用于选择特定侦听器。 如果未指定此属性，API 管理将尝试使用没有名称的侦听器。 没有名称的侦听器对于只有一个侦听器的 Reliable Services 来说是非常典型的。 |“否”|不适用|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、后端

-   **策略范围：** 全局、产品、API、操作

##  <a name="SetBody"></a> 设置正文
 使用 `set-body` 策略设置传入和传出请求的消息正文。 可以使用 `context.Request.Body` 属性或 `context.Response.Body` 访问消息正文，具体取决于策略是在入站节中还是在出站节中。

> [!IMPORTANT]
>  请注意，默认情况下，当用户使用 `context.Request.Body` 或 `context.Response.Body` 访问消息正文时，原始的消息正文会丢失，必须将正文返回到表达式中，以便对其进行设置。 若要保留正文内容，请在访问消息时将 `preserveContent` 参数设置为 `true`。 如果 `preserveContent` 设置为 `true`，而表达式返回了不同的正文，则会使用返回的正文。
> 
>  在使用 `set-body` 策略时，请注意以下事项。
> 
> - 若要使用 `set-body` 策略返回全新的或更新的正文，则不需将 `preserveContent` 设置为 `true`，因为是在显式提供全新的正文内容。
>   -   将响应的内容保留在入站管道中并不合理，因为尚无响应。
>   -   将请求的内容保留在出站管道中并不合理，因为请求此时已发送到后端。
>   -   如果在没有消息正文的情况下使用此策略（例如在入站 GET 中使用），则会引发异常。

 有关详细信息，请参阅[上下文变量](api-management-policy-expressions.md#ContextVariables)表中的 `context.Request.Body`、`context.Response.Body`、`IMessage` 部分。

### <a name="policy-statement"></a>策略语句

```xml
<set-body>new body value as text</set-body>
```

### <a name="examples"></a>示例

#### <a name="literal-text-example"></a>文字文本示例

```xml
<set-body>Hello world!</set-body>
```

#### <a name="example-accessing-the-body-as-a-string-note-that-we-are-preserving-the-original-request-body-so-that-we-can-access-it-later-in-the-pipeline"></a>示例：访问字符串形式的正文。 请注意，我们会保留原始请求正文，以便稍后可以在管道中进行访问。

```xml
<set-body>
@{ 
    string inBody = context.Request.Body.As<string>(preserveContent: true); 
    if (inBody[0] =='c') { 
        inBody[0] = 'm'; 
    } 
    return inBody; 
}
</set-body>
```

#### <a name="example-accessing-the-body-as-a-jobject-note-that-since-we-are-not-reserving-the-original-request-body-accessing-it-later-in-the-pipeline-will-result-in-an-exception"></a>示例：访问 JObject 形式的正文。 请注意，由于我们不保留原始请求正文，因此稍后在管道进行访问将产生异常。

```xml
<set-body> 
@{ 
    JObject inBody = context.Request.Body.As<JObject>(); 
    if (inBody.attribute == <tag>) { 
        inBody[0] = 'm'; 
    } 
    return inBody.ToString(); 
} 
</set-body>

```

#### <a name="filter-response-based-on-product"></a>根据产品筛选响应
 以下示例演示了如何进行内容筛选，方法是：在使用 `Starter` 产品时删除从后端服务接收的响应中的数据元素。 有关配置和使用此策略的演示，请参阅 [Cloud Cover 第 177 集：更多的 API 管理功能与 VLAD Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) 并快进到 34:30。 若要大致了解用于此演示的 [Dark Sky Forecast API](https://developer.forecast.io/)，请从 31:50 开始观看。

```xml
<!-- Copy this snippet into the outbound section to remove a number of data elements from the response received from the backend service based on the name of the api product -->
<choose>
  <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
          response.Property (key).Remove ();
        }
        return response.ToString();
      }
    </set-body>
  </when>
</choose>
```

### <a name="using-liquid-templates-with-set-body"></a>将 Liquid 模板用于设置正文
可将 `set-body` 策略配置为使用 [Liquid](https://shopify.github.io/liquid/basics/introduction/) 模板语言以转换请求或响应正文。 如需完全重设消息格式，则此模板非常有用。

> [!IMPORTANT]
> 在“C# 模式”下配置 `set-body` 策略中使用的 Liquid 的实现。 执行筛选等操作时这点尤为重要。 例如，使用日期筛选器需要使用 Pascal 大小写和 C# 日期格式，如：
>
> {{body.foo.startDateTime| Date:"yyyyMMddTHH:mm:ddZ"}}

> [!IMPORTANT]
> 为正确绑定到使用 Liquid 模板的 XML 正文，请使用 `set-header` 策略将内容类型设置为 application/xml、text/xml 或任何以 +xml 结尾的类型；对于 JSON 正文，其必须是 application/json、text/json 或任何以 +json 结尾的类型。

#### <a name="convert-json-to-soap-using-a-liquid-template"></a>使用 Liquid 模板将 JSON 转换为 SOAP
```xml
<set-body template="liquid">
    <soap:Envelope xmlns="http://tempuri.org/" xmlns:soap="https://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
            <GetOpenOrders>
                <cust>{{body.getOpenOrders.cust}}</cust>
            </GetOpenOrders>
        </soap:Body>
    </soap:Envelope>
</set-body>
```

#### <a name="transform-json-using-a-liquid-template"></a>使用 Liquid 模板转换 JSON
```xml
{
"order": {
    "id": "{{body.customer.purchase.identifier}}",
    "summary": "{{body.customer.purchase.orderShortDesc}}"
    }
}
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|set-body|根元素。 包含正文文本，或者包含会返回正文的表达式。|“是”|

### <a name="properties"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|模板|用于更改设置正文策略运行的模板模式。 目前唯一支持的值是：<br /><br />- Liquid - 设置正文策略会使用 Liquid 模板引擎 |“否”|Liquid|

对于访问请求和响应信息，Liquid 模板可绑定到具有以下属性的上下文对象： <br />
<pre>context.
    Request.
        Url
        Method
        OriginalMethod
        OriginalUrl
        IpAddress
        MatchedParameters
        HasBody
        ClientCertificates
        Headers

    Response.
        StatusCode
        Method
        Headers
Url.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString

OriginalUrl.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString
</pre>



### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站、后端

-   **策略范围：** 全局、产品、API、操作

##  <a name="SetHTTPheader"></a> 设置 HTTP 标头
 `set-header` 策略向现有的响应和/或请求标头赋值，或者添加新的响应和/或请求标头。

 在 HTTP 消息中插入 HTTP 标头列表。 将此策略放到入站管道中后，它将为传递给目标服务的请求设置 HTTP 标头。 将此策略放到出站管道中后，它将为发送到网关客户端的响应设置 HTTP 标头。

### <a name="policy-statement"></a>策略语句

```xml
<set-header name="header name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple headers with the same name add additional value elements-->
</set-header>
```

### <a name="examples"></a>示例

#### <a name="example"></a>示例

```xml
<set-header name="some header name" exists-action="override">
    <value>20</value>
</set-header>
```

#### <a name="forward-context-information-to-the-backend-service"></a>将上下文信息转发到后端服务
 此示例演示了如何在 API 级别应用策略，以便将上下文信息提供给后端服务。 有关配置和使用此策略的演示，请参阅 [Cloud Cover 第 177 集：更多的 API 管理功能与 VLAD Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) 并快进到 10:30。 在 12:10 处有一个演示，演示了如何在开发人员门户中调用操作，以便在其中查看生效的策略。

```xml
<!-- Copy this snippet into the inbound element to forward some context information, user id and the region the gateway is hosted in, to the backend service for logging or evaluation -->
<set-header name="x-request-context-data" exists-action="override">
  <value>@(context.User.Id)</value>
  <value>@(context.Deployment.Region)</value>
</set-header>
```

 有关详细信息，请参阅[策略表达式](api-management-policy-expressions.md)和[上下文变量](api-management-policy-expressions.md#ContextVariables)。

> [!NOTE]
> 标头的多个值会连接到 CSV 字符串，例如所示： `headerName: value1,value2,value3`
>
> 例外情况包括标准化标头，其值：
> - 可能包含逗号（`User-Agent`、`WWW-Authenticate`、`Proxy-Authenticate`），
> - 可能包含日期（`Cookie`、`Set-Cookie`、`Warning`），
> - 包含日期（`Date`、`Expires`、`If-Modified-Since`、`If-Unmodified-Since`、`Last-Modified`、`Retry-After`）。
>
> 发生这些异常时多个标头值不能连接为一个字符串，并将作为单独的标头，例如传递： `User-Agent: value1`
>`User-Agent: value2`
>`User-Agent: value3`

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|set-header|根元素。|“是”|
|value|指定要设置的标头的值。 如需多个标头使用同一名称，可添加更多的 `value` 元素。|“是”|

### <a name="properties"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|exists-action|指定当标头已指定时要执行的操作。 此属性必须具有下列值之一。<br /><br /> -   override - 替换现有标头的值。<br />-   skip - 不替换现有标头值。<br />-   append - 将值追加到现有标头值。<br />-   delete - 从请求中删除标头。<br /><br /> 如果设置为 `override`，则登记多个同名的条目会导致根据所有条目（将多次列出）设置标头；结果中只会设置列出的值。|“否”|override|
|名称|指定要设置的标头的名称。|“是”|不适用|

### <a name="usage"></a>使用情况
 此策略可在以下策略[节](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站、后端、错误时

-   **策略范围：** 全局、产品、API、操作

##  <a name="SetQueryStringParameter"></a> 设置查询字符串参数
 `set-query-parameter` 策略添加、删除请求查询字符串参数或替换其值。 可用于传递后端服务所需的查询参数，这些参数是可选的或者永远不能出现在请求中。

### <a name="policy-statement"></a>策略语句

```xml
<set-query-parameter name="param name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple parameters with the same name add additional value elements-->
</set-query-parameter>
```

### <a name="examples"></a>示例

#### <a name="example"></a>示例

```xml

<set-query-parameter>
  <parameter name="api-key" exists-action="skip">
    <value>12345678901</value>
  </parameter>
  <!-- for multiple parameters with the same name add additional value elements -->
</set-query-parameter>

```

#### <a name="forward-context-information-to-the-backend-service"></a>将上下文信息转发到后端服务
 此示例演示了如何在 API 级别应用策略，以便将上下文信息提供给后端服务。 有关配置和使用此策略的演示，请参阅 [Cloud Cover 第 177 集：更多的 API 管理功能与 VLAD Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) 并快进到 10:30。 在 12:10 处有一个演示，演示了如何在开发人员门户中调用操作，以便在其中查看生效的策略。

```xml
<!-- Copy this snippet into the inbound element to forward a piece of context, product name in this example, to the backend service for logging or evaluation -->
<set-query-parameter name="x-product-name" exists-action="override">
  <value>@(context.Product.Name)</value>
</set-query-parameter>

```

 有关详细信息，请参阅[策略表达式](api-management-policy-expressions.md)和[上下文变量](api-management-policy-expressions.md#ContextVariables)。

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|set-query-parameter|根元素。|“是”|
|value|指定要设置的查询参数的值。 如需多个查询参数使用同一名称，可添加更多的 `value` 元素。|“是”|

### <a name="properties"></a>属性

|名称|描述|需要|默认|
|----------|-----------------|--------------|-------------|
|exists-action|指定当查询参数已指定时要执行的操作。 此属性必须具有下列值之一。<br /><br /> -   override - 替换现有参数的值。<br />-   skip - 不替换现有查询参数值。<br />-   append - 将值追加到现有查询参数值。<br />-   delete - 从请求中删除查询参数。<br /><br /> 如果设置为 `override`，则登记多个同名的条目会导致根据所有条目（将多次列出）设置查询参数；结果中只会设置列出的值。|“否”|override|
|名称|指定要设置的查询参数的名称。|“是”|不适用|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、后端

-   **策略范围：** 全局、产品、API、操作

##  <a name="RewriteURL"></a> 重写 URL
 `rewrite-uri` 策略将请求 URL 从其公用格式转换为 Web 服务所需的格式，如以下示例所示。

- 公共 URL - `http://api.example.com/storenumber/ordernumber`

- 请求 URL - `http://api.example.com/v2/US/hardware/storenumber&ordernumber?City&State`

  如果要将用户和/或浏览器友好的 URL 转换成 Web 服务所需的 URL 格式，则可使用此策略。 应用此策略的前提是公开备用的 URL 格式，例如简洁 URL、RESTful URL、用户友好的 URL 或 SEO 友好的 URL。这些 URL 是纯结构化 URL，不包含查询字符串，只包含资源的路径（在方案和颁发机构的后面）。 通常会出于美观、可用性或搜索引擎优化 (SEO) 目的使用这种 URL。

> [!NOTE]
>  只能使用此策略添加查询字符串参数。 不能在重写 URL 中添加额外的模板路径参数。

### <a name="policy-statement"></a>策略语句

```xml
<rewrite-uri template="uri template" copy-unmatched-params="true | false" />
```

### <a name="example"></a>示例

```xml
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/v2/US/hardware/{storenumber}&{ordernumber}?City=city&State=state" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put?c=d -->
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" copy-unmatched-params="false" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put -->
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|rewrite-uri|根元素。|“是”|

### <a name="attributes"></a>属性

|特性|描述|需要|默认|
|---------------|-----------------|--------------|-------------|
|template|包含任何查询字符串参数的实际 Web 服务 URL。 使用表达式时，整个值必须是一个表达式。|“是”|不适用|
|copy-unmatched-params|指定是否将原始 URL 模板中不存在的传入请求中的查询参数添加到重新编写模板定义的 URL|“否”|true|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略段：** 入站

-   **策略范围：** 全局、产品、API、操作

##  <a name="XSLTransform"></a> 使用 XSLT 转换 XML
 `Transform XML using an XSLT` 策略在请求或响应正文中将 XSL 转换应用到 XML。

### <a name="policy-statement"></a>策略语句

```xml
<xsl-transform>
    <parameter name="User-Agent">@(context.Request.Headers.GetValueOrDefault("User-Agent","non-specified"))</parameter>
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        <xsl:output method="xml" indent="yes" />
        <xsl:param name="User-Agent" />
        <xsl:template match="* | @* | node()">
            <xsl:copy>
                <xsl:if test="self::* and not(parent::*)">
                    <xsl:attribute name="User-Agent">
                        <xsl:value-of select="$User-Agent" />
                    </xsl:attribute>
                </xsl:if>
                <xsl:apply-templates select="* | @* | node()" />
            </xsl:copy>
        </xsl:template>
    </xsl:stylesheet>
  </xsl-transform>
```

### <a name="example"></a>示例

```xml
<policies>
  <inbound>
      <base />
  </inbound>
  <outbound>
      <base />
      <xsl-transform>
        <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
            <xsl:output omit-xml-declaration="yes" method="xml" indent="yes" />
            <!-- Copy all nodes directly-->
            <xsl:template match="node()| @*|*">
                <xsl:copy>
                    <xsl:apply-templates select="@* | node()|*" />
                </xsl:copy>
            </xsl:template>
        </xsl:stylesheet>
    </xsl-transform>
  </outbound>
</policies>
```

### <a name="elements"></a>元素

|名称|描述|需要|
|----------|-----------------|--------------|
|xsl-transform|根元素。|“是”|
|参数|用于定义在转换中使用的变量|“否”|
|xsl:stylesheet|根样式表元素。 在其中定义的所有元素和属性都遵循标准的 [XSLT 规范](https://www.w3.org/TR/xslt)|“是”|

### <a name="usage"></a>使用情况
 此策略可在以下策略[段](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#sections)和[范围](https://azure.microsoft.com/documentation/articles/api-management-howto-policies/#scopes)中使用。

-   **策略节：** 入站、出站

-   **策略范围：** 全局、产品、API、操作

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅下列主题：

+ [API 管理中的策略](api-management-howto-policies.md)
+ [策略参考](api-management-policy-reference.md)，获取策略语句及其设置的完整列表
+ [策略示例](policy-samples.md)
