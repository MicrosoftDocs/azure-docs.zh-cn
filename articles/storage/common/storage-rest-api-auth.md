---
title: 调用包括身份验证的 Azure 存储服务 REST API 操作 | Microsoft 文档
description: 调用包括身份验证的 Azure 存储服务 REST API 操作
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 19272e93739d98962ab6818e1c2626ac9e0ac6d9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204444"
---
# <a name="using-the-azure-storage-rest-api"></a>使用 Azure 存储 REST API

本文演示如何使用 Blob 存储服务 REST API 以及如何对服务调用进行身份验证。 本文内容是从对 REST 无甚了解、而且也不知道如何进行 REST 调用的开发人员角度编写的。 我们来看一下有关 REST 调用的参考文档，并了解如何将其运用到实际的 REST 调用中 – 哪些字段可以在哪里设置？ 了解如何设置 REST 调用后，你可以利用这一知识使用任何其他存储服务 REST API。

## <a name="prerequisites"></a>必备组件 

应用程序在存储帐户的 blob 存储中列出容器。 若要尝试本文中的代码，需准备以下各项： 

* 使用以下工作负荷安装 [Visual Studio 2017](https://www.visualstudio.com/visual-studio-homepage-vs.aspx)：
    - Azure 开发

* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 通用存储帐户。 如果还没有存储帐户，请参阅[创建存储帐户](storage-quickstart-create-account.md)。

* 本文中将举例说明如何列出存储帐户中的容器。 若要查看输出，请在开始之前，将一些容器添加到存储帐户中的 blob 存储。

## <a name="download-the-sample-application"></a>下载示例应用程序

该示例应用程序是以 C# 编写的控制台应用程序。

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-dotnet-rest-api-with-auth.git
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开 Visual Studio 解决方案，请找到 storage-dotnet-rest-api-with-auth 文件夹并打开，然后双击 StorageRestApiAuth.sln。 

## <a name="what-is-rest"></a>什么是 REST？

REST 是指表述性状态转移。 有关具体定义，请参阅 [Wikipedia](https://en.wikipedia.org/wiki/Representational_state_transfer)。

基本上，REST 是在调用 API 或使 API 可调用时使用的体系结构。 无论一方发生什么，以及在发送或接收 REST 调用时使用什么其他软件，它都不会受影响。 你可以编写一个在 Mac、Windows、Linux、Android 手机或平板电脑、iPhone、iPod 或网站上运行的应用程序，并为所有这些平台使用相同的 REST API。 调用 REST API 时，可以传入和/或传出数据。 REST API 不关心从中进行调用的平台 - 重要的是在请求中传递的信息以及在响应中提供的数据。

了解如何使用 REST 是一项非常有用的技能。 Azure 产品团队会频繁发布新功能。 很多时候，新功能可通过 REST 接口访问，但尚未通过所有存储客户端库或 UI 显示（如 Azure 门户）。 如果要始终使用最新且最好的功能，则需要学习 REST。 此外，如果想要编写你自己的库以便与 Azure 存储进行交互，或者想要使用没有 SDK 或存储客户端库的编程语言访问 Azure 存储，则可以使用 REST API。

## <a name="about-the-sample-application"></a>关于示例应用程序

示例应用程序列出了存储帐户中的容器。 一旦了解 REST API 文档中的信息如何关联到实际代码后，其他 REST 调用将更容易理解。 

若参阅 [Blob 服务 REST API](/rest/api/storageservices/Blob-Service-REST-API)，你将会了解到所有可以在 blob 存储中执行的操作。 存储客户端库是 REST API 的包装器 – 它们可使你轻松访问存储而无需直接使用 REST API。 但如上所述，有时你会想要使用 REST API 而不是存储客户端库。

## <a name="rest-api-reference-list-containers-api"></a>REST API 参考：列出容器 API

让我们看一下 REST API 参考中的 [ListContainers](/rest/api/storageservices/List-Containers2) 操作页面，以便了解请求中某些字段的出处，并在下一部分中使用代码进行响应。

**请求方法**：GET。 此谓词是你指定为请求对象属性的 HTTP 方法。 此谓词的其他值包括 HEAD、PUT 和 DELETE，具体将取决于正在调用的 API。

**请求 URI**： https://myaccount.blob.core.windows.net/?comp=list 这是基于 blob 存储帐户终结点 `http://myaccount.blob.core.windows.net` 和资源字符串 `/?comp=list` 创建的。

[URI 参数](/rest/api/storageservices/List-Containers2#uri-parameters)：调用 ListContainers 时还可以使用其他查询参数。 其中有些参数为调用超时（以秒计）和前缀，后者用于筛选。

另一个有用参数是 maxresults:，如果可用容器超过此值，则响应正文将包含一个 NextMarker 元素，指示要在下一个请求中返回的下一个容器。 若要使用此功能，可提供 NextMarker 值，作为发出下一个请求时 URI 中的 marker 参数。 使用此功能时，它类似于通过结果进行分页。 

若要使用其他参数，请将它们追加到带有值的资源字符串，如下例所示：

```
/?comp=list&timeout=60&maxresults=100
```

[请求标头](/rest/api/storageservices/List-Containers2#request-headers)：本部分列出了必需和可选的请求标头。 至少需要三个标头：Authorization 标头、x-ms-date（包含请求的 UTC 时间）和 x-ms-version（指定要使用的 REST API 版本）。 可以选择将 x-ms-client-request-id 包含在标头中 – 可以将此字段的值设置为任何内容；该值将在启用日志记录时写入存储分析日志。

[请求正文](/rest/api/storageservices/List-Containers2#request-body)：ListContainers 没有请求正文。 上传 blob 时，会在所有 PUT 操作上使用请求正文，以及 SetContainerAccessPolicy，以允许在要应用的存储访问策略的 XML 列表中发送 blob。 有关存储访问策略，将在[使用共享访问签名 (SAS)](storage-dotnet-shared-access-signature-part-1.md) 一文中展开讨论。

[响应状态代码](/rest/api/storageservices/List-Containers2#status-code)：告知你需要知道的任何状态代码。 在此示例中，HTTP 状态代码可以是 200。 有关 HTTP 状态代码的完整列表，请参阅[状态代码定义](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。 若要查看特定于存储 REST API 的错误代码，请参阅[常见的 REST API 错误代码](/rest/api/storageservices/common-rest-api-error-codes)

[响应标头](/rest/api/storageservices/List-Containers2#response-headers)：其中包括 Content Type；x-ms-request-id（传入的请求ID，如适用）；x-ms-version（指示所使用的 BLOB 服务的版本）和 Date（UTC，告知发出请求的时间）。

[响应正文](/rest/api/storageservices/List-Containers2#response-body)：此字段是提供请求数据的 XML 结构。 在此示例中，响应是容器及其属性的列表。

## <a name="creating-the-rest-request"></a>创建 REST 请求

开始之前的几个注意事项 – 针对在生产中运行时的安全性，请始终使用 HTTPS 而不是 HTTP。 出于本次练习的目的，应使用 HTTP 以便查看请求和响应数据。 若要查看实际 REST 调用中的请求和响应信息，可以下载 [Fiddler](https://www.telerik.com/fiddler) 或类似应用。 在 Visual Studio 解决方案中，存储帐户名称和密钥在类中为硬编码，ListContainersAsyncREST 方法会将存储帐户名称和存储帐户密钥传递给用于创建 REST 请求各个组件的方法。 在实际应用中，存储帐户名称和密钥将驻留在配置文件、环境变量中，或从 Azure Key Vault 中检索。

在我们的示例项目中，用于创建授权标头的代码位于单独的类中，你可以获取整个类并将其添加到你自己的解决方案中，然后“按原样”使用。 授权标头代码适用于 Azure 存储的大多数 REST API 调用。

要生成请求（这是一个 HttpRequestMessage 对象），请转到 Program.cs 中的 ListContainersAsyncREST。 用于生成请求的步骤如下： 

* 创建要用于调用服务的 URI。 
* 创建 HttpRequestMessage 对象并设置有效负载。 有效负载对于 ListContainersAsyncREST 为 null，因为我们未传入任何内容。
* 添加 x-ms-date 和 x-ms-version 的请求标头。
* 获取授权标头并添加。

你需要一些基本信息： 

*  对于 ListContainers，方法是 `GET`。 在实例化请求时设置此值。 
*  资源是指示正在调用的 API 的 URI 查询部分，因此，值为 `/?comp=list`。 如前文所述，该资源位于显示有关 [ListContainers API](/rest/api/storageservices/List-Containers2) 信息的参考文档页上。
*  URI 是通过为该存储帐户创建 Blob 服务终结点并连结该资源而构建的。 请求 URI 的值最终为 `http://contosorest.blob.core.windows.net/?comp=list`。
*  对于 ListContainers，requestBody 为 null 并且没有任何额外标头。

不同 API 可能有其他参数传入，如 ifMatch。 你可能使用 ifMatch 的一个示例是调用 PutBlob 时。 在这种情况下，将 ifMatch 设置为 eTag，如果你提供的 eTag 与 blob 上的当前 eTag 匹配，那么它只更新 blob。 如果其他人自检索 eTag 后已更新 blob，则其更改不会被重写。 

首先，设置 `uri` 和 `payload`。 

```csharp
// Construct the URI. This will look like this:
//   https://myaccount.blob.core.windows.net/resource
String uri = string.Format("http://{0}.blob.core.windows.net?comp=list", storageAccountName);

// Set this to whatever payload you desire. Ours is null because 
//   we're not passing anything in.
Byte[] requestPayload = null;
```

接下来，实例化请求，将方法设置为 `GET` 并提供 URI。

```csharp 
//Instantiate the request message with a null payload.
using (var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, uri)
{ Content = (requestPayload == null) ? null : new ByteArrayContent(requestPayload) })
{
```

添加 x-ms-date 和 x-ms-version 的请求标头。 此代码中的这个位置也是你在其中添加调用所需的任何其他请求标头的位置。 在此示例中，没有其他标头。 API 传入其他标头的一个示例是 SetContainerACL。 对于 Blob 存储，它将添加名为“x-ms-blob-public-acces”的标头和访问级别的值。

```csharp
    // Add the request headers for x-ms-date and x-ms-version.
    DateTime now = DateTime.UtcNow;
    httpRequestMessage.Headers.Add("x-ms-date", now.ToString("R", CultureInfo.InvariantCulture));
    httpRequestMessage.Headers.Add("x-ms-version", "2017-07-29");
    // If you need any additional headers, add them here before creating
    //   the authorization header. 
```

调用创建授权标头的方法，并将其添加到请求标头。 本文的后面部分将介绍如何创建授权标头。 方法名称为 GetAuthorizationHeader，你可以在此代码段中看到：

```csharp
    // Get the authorization header and add it.
    httpRequestMessage.Headers.Authorization = AzureStorageAuthenticationHelper.GetAuthorizationHeader(
        storageAccountName, storageAccountKey, now, httpRequestMessage);
```

此时，`httpRequestMessage` 包含 REST 请求，并带有授权标头。 

## <a name="call-the-rest-api-with-the-request"></a>使用请求调用 REST API

至此，你已经有了请求，接下来即可调用 SendAsync 来发送 REST 请求。 SendAsync 调用 API，并获取响应。 检查响应状态代码（可以是 200），然后分析响应。 在这种情况下，你将获取到一个容器的 XML 列表。 让我们看一下用于调用 GetRESTRequest 方法的代码，以创建请求、执行请求，然后检查对容器列表的响应。

```csharp 
    // Send the request.
    using (HttpResponseMessage httpResponseMessage = 
      await new HttpClient().SendAsync(httpRequestMessage, cancellationToken))
    {
        // If successful (status code = 200), 
        //   parse the XML response for the container names.
        if (httpResponseMessage.StatusCode == HttpStatusCode.OK)
        {
            String xmlString = await httpResponseMessage.Content.ReadAsStringAsync();
            XElement x = XElement.Parse(xmlString);
            foreach (XElement container in x.Element("Containers").Elements("Container"))
            {
                Console.WriteLine("Container name = {0}", container.Element("Name").Value);
            }
        }
    }
}
```

如果在调用 SendAsync 时运行网络探测器（如 [Fiddler](https://www.telerik.com/fiddler)），则会看到请求和响应信息。 让我们一起看一下。 存储帐户的名称是 contosorest。

**请求：**

```
GET /?comp=list HTTP/1.1
```

请求标头：

```
x-ms-date: Thu, 16 Nov 2017 23:34:04 GMT
x-ms-version: 2014-02-14
Authorization: SharedKey contosorest:1dVlYJWWJAOSHTCPGiwdX1rOS8B4fenYP/VrU0LfzQk=
Host: contosorest.blob.core.windows.net
Connection: Keep-Alive
```

执行后返回的状态代码和响应标头：

```
HTTP/1.1 200 OK
Content-Type: application/xml
Server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 3e889876-001e-0039-6a3a-5f4396000000
x-ms-version: 2017-07-29
Date: Fri, 17 Nov 2017 00:23:42 GMT
Content-Length: 1511
```

**响应正文 (XML)：** 对于 ListContainers，将显示容器及其属性列表。

```xml  
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults 
  ServiceEndpoint="http://contosorest.blob.core.windows.net/">
  <Containers>
    <Container>
      <Name>container-1</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:39:48 GMT</Last-Modified>
        <Etag>"0x8D46CBD5A7C301D"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-2</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:40:50 GMT</Last-Modified>
        <Etag>"0x8D46CBD7F49E9BD"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-3</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:41:10 GMT</Last-Modified>
        <Etag>"0x8D46CBD8B243D68"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-4</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:41:25 GMT</Last-Modified>
        <Etag>"0x8D46CBD93FED46F"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        </Properties>
      </Container>
      <Container>
        <Name>container-5</Name>
        <Properties>
          <Last-Modified>Thu, 16 Mar 2017 22:41:39 GMT</Last-Modified>
          <Etag>"0x8D46CBD9C762815"</Etag>
          <LeaseStatus>unlocked</LeaseStatus>
          <LeaseState>available</LeaseState>
        </Properties>
      </Container>
  </Containers>
  <NextMarker />
</EnumerationResults>
```

现在已了解如何创建请求、调用服务和分析结果，接下来我们来看下如何创建授权标头。 创建标头比较复杂，但好消息是，代码一旦运行成功，它将适用于所有存储服务 REST API。

## <a name="creating-the-authorization-header"></a>创建授权标头

> [!TIP]
> Azure 存储现在支持 blob 和队列的 Azure Active Directory (Azure AD) 集成。 Azure AD 提供更简单的 Azure 存储请求授权体验。 有关如何使用 Azure AD 授权 REST 操作的详细信息，请参阅 [Authenticate with Azure Active Directory](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-azure-active-directory)（使用 Azure Active Directory 进行身份验证）。 有关 Azure AD 与 Azure 存储集成的概述，请参阅[使用 Azure Active Directory 对 Azure 存储的访问权限进行身份验证](storage-auth-aad.md)。

有一篇文章从概念上（无代码）介绍了如何执行 [Azure 存储服务的身份验证](/rest/api/storageservices/Authorization-for-the-Azure-Storage-Services)。
让我们就基于此篇文章准确提取所需的内容并显示代码。

首先，使用“共享密钥”身份验证。 授权标头格式如下所示：

```  
Authorization="SharedKey <storage account name>:<signature>"  
```

签名字段是基于哈希的消息身份验证代码 (HMAC)，该代码通过请求创建并使用 SHA256 算法计算而得，然后使用 Base64 编码进行编码。 是否明白了？ （不要急，你还没有听说过标准化一词。）

此代码段演示了共享密钥签名字符串的格式：

```csharp  
StringToSign = VERB + "\n" +  
               Content-Encoding + "\n" +  
               Content-Language + "\n" +  
               Content-Length + "\n" +  
               Content-MD5 + "\n" +  
               Content-Type + "\n" +  
               Date + "\n" +  
               If-Modified-Since + "\n" +  
               If-Match + "\n" +  
               If-None-Match + "\n" +  
               If-Unmodified-Since + "\n" +  
               Range + "\n" +  
               CanonicalizedHeaders +  
               CanonicalizedResource;  
```

其中大部分字段都很少用到。 对于 Blob 存储，你可以指定谓词、md5、内容长度、规范化标头和规范化资源。 可以将其他内容留空（但将其置于 `\n` 中，使其知道它们为空）。

规范化标头和规范化资源有哪些？ 问得好。 事实上，规范化究竟是什么意思？ Microsoft Word 甚至无法将其作为一个词识别。 下面是 [Wikipedia 有关标准化的介绍](https://en.wikipedia.org/wiki/Canonicalization)：在计算机科学中，标准化（有时称为规范化或正则化）是将有多种可能表示形式的数据转换为“标准”、“常规”，或规范格式的过程。 在正常情况下，这意味着获取项目列表（例如，在规范化标头情况下的标头），并将它们标准化为所需的格式。 基本上，Microsoft 会决定使用一种格式，而你需要匹配它。

让我们从这两个规范化字段开始，因为需要它们来创建授权标头。

**规范化标头**

若要创建此值，请检索以“x-ms-”开头的标头并对其进行排序，然后将它们格式化为 `[key:value\n]` 字符串实例，并将其连结到一个字符串中。 在此示例中，规范化标头如下所示： 

```
x-ms-date:Fri, 17 Nov 2017 00:44:48 GMT\nx-ms-version:2017-07-29\n
```

以下是用于创建该输出的代码：

```csharp 
private static string GetCanonicalizedHeaders(HttpRequestMessage httpRequestMessage)
{
    var headers = from kvp in httpRequestMessage.Headers
        where kvp.Key.StartsWith("x-ms-", StringComparison.OrdinalIgnoreCase)
        orderby kvp.Key
        select new { Key = kvp.Key.ToLowerInvariant(), kvp.Value };

    StringBuilder headersBuilder = new StringBuilder();

    // Create the string in the right format; this is what makes the headers "canonicalized" --
    //   it means put in a standard format. https://en.wikipedia.org/wiki/Canonicalization
    foreach (var kvp in headers)
    {
        headersBuilder.Append(kvp.Key);
        char separator = ':';

        // Get the value for each header, strip out \r\n if found, then append it with the key.
        foreach (string headerValue in kvp.Value)
        {
            string trimmedValue = headerValue.TrimStart().Replace("\r\n", string.Empty);
            headersBuilder.Append(separator).Append(trimmedValue);

            // Set this to a comma; this will only be used
            // if there are multiple values for one of the headers.
            separator = ',';
        }

        headersBuilder.Append("\n");
    }

    return headersBuilder.ToString();
}
```

**规范化资源**

此部分签名字符串表示请求指向的存储帐户。 请记住，请求 URI 是 `<http://contosorest.blob.core.windows.net/?comp=list>`，使用实际帐户名（在此情况下为 `contosorest`）。 在此示例中，将返回：

```
/contosorest/\ncomp:list
```

如果你有查询参数，还将包括这些参数。 以下是代码，该代码还处理其他查询参数和具有多个值的查询参数。 请记住，需要生成此代码，使其适用于所有 REST API，因此，建议包括所有可能性，即使 ListContainers 方法不需要所有这些参数。

```csharp 
private static string GetCanonicalizedResource(Uri address, string storageAccountName)
{
    // The absolute path will be "/" because for we're getting a list of containers.
    StringBuilder sb = new StringBuilder("/").Append(storageAccountName).Append(address.AbsolutePath);

    // Address.Query is the resource, such as "?comp=list".
    // This ends up with a NameValueCollection with 1 entry having key=comp, value=list.
    // It will have more entries if you have more query parameters.
    NameValueCollection values = HttpUtility.ParseQueryString(address.Query);

    foreach (var item in values.AllKeys.OrderBy(k => k))
    {
        sb.Append('\n').Append(item).Append(':').Append(values[item]);
    }

    return sb.ToString();
}
```

现在，已设置规范化字符串，我们接着来看如何创建授权标头本身。 首先，创建一个如前文所述的 StringToSign 格式的消息签名字符串。 在代码中使用注释会更容易解释这一概念，因此，下面提供了返回授权标头的最后一种方法：

```csharp
internal static AuthenticationHeaderValue GetAuthorizationHeader(
    string storageAccountName, string storageAccountKey, DateTime now,
    HttpRequestMessage httpRequestMessage, string ifMatch = "", string md5 = "")
{
    // This is the raw representation of the message signature.
    HttpMethod method = httpRequestMessage.Method;
    String MessageSignature = String.Format("{0}\n\n\n{1}\n{5}\n\n\n\n{2}\n\n\n\n{3}{4}",
                method.ToString(),
                (method == HttpMethod.Get || method == HttpMethod.Head) ? String.Empty
                  : httpRequestMessage.Content.Headers.ContentLength.ToString(),
                ifMatch,
                GetCanonicalizedHeaders(httpRequestMessage),
                GetCanonicalizedResource(httpRequestMessage.RequestUri, storageAccountName),
                md5);

    // Now turn it into a byte array.
    byte[] SignatureBytes = Encoding.UTF8.GetBytes(MessageSignature);

    // Create the HMACSHA256 version of the storage key.
    HMACSHA256 SHA256 = new HMACSHA256(Convert.FromBase64String(storageAccountKey));

    // Compute the hash of the SignatureBytes and convert it to a base64 string.
    string signature = Convert.ToBase64String(SHA256.ComputeHash(SignatureBytes));

    // This is the actual header that will be added to the list of request headers.
    AuthenticationHeaderValue authHV = new AuthenticationHeaderValue("SharedKey",
        storageAccountName + ":" + Convert.ToBase64String(SHA256.ComputeHash(SignatureBytes)));
    return authHV;
}
```

运行此代码时，生成的 MessageSignature 如下所示：

```
GET\n\n\n\n\n\n\n\n\n\n\n\nx-ms-date:Fri, 17 Nov 2017 01:07:37 GMT\nx-ms-version:2017-07-29\n/contosorest/\ncomp:list
```

下面是 AuthorizationHeader 的最终值：

```
SharedKey contosorest:Ms5sfwkA8nqTRw7Uury4MPHqM6Rj2nfgbYNvUKOa67w=
```

AuthorizationHeader 是发出响应前放置在请求标头中的最后一个标头。

这涵盖了你需要知道的所有信息以及代码，可以将一个类组合在一起创建一个用于调用存储服务 REST API 的请求。

## <a name="how-about-another-example"></a>另一个示例呢？ 

让我们看一下如何更改代码以对容器 container-1 调用 ListBlobs。 这与清单容器的代码几乎完全相同，惟一的区别在于 URI 以及解析响应的方式。 

如果查看 [ListBlobs](/rest/api/storageservices/List-Blobs) 的参考文档，将发现该方法是 GET，RequestURI 为：

```
https://myaccount.blob.core.windows.net/container-1?restype=container&comp=list
```

在 ListContainersAsyncREST 中，更改将 URI 设置为 ListBlobs API 的代码。 容器名称为 container-1。

```csharp
String uri = 
    string.Format("http://{0}.blob.core.windows.net/container-1?restype=container&comp=list",
      storageAccountName);

```

然后，在你处理响应时，更改代码以查找 blob 而不是容器。

```csharp
foreach (XElement container in x.Element("Blobs").Elements("Blob"))
{
    Console.WriteLine("Blob name = {0}", container.Element("Name").Value);
}
```

在运行此示例时，将获得如下结果：

规范化标头：

```
x-ms-date:Fri, 17 Nov 2017 05:16:48 GMT\nx-ms-version:2017-07-29\n
```

规范化资源：

```
/contosorest/container-1\ncomp:list\nrestype:container
```

MessageSignature：

```
GET\n\n\n\n\n\n\n\n\n\n\n\nx-ms-date:Fri, 17 Nov 2017 05:16:48 GMT
  \nx-ms-version:2017-07-29\n/contosorest/container-1\ncomp:list\nrestype:container
```

AuthorizationHeader：

```
SharedKey contosorest:uzvWZN1WUIv2LYC6e3En10/7EIQJ5X9KtFQqrZkxi6s=
```

以下值来自 [Fiddler](https://www.telerik.com/fiddler)：

**请求：**

```
GET http://contosorest.blob.core.windows.net/container-1?restype=container&comp=list HTTP/1.1
```

请求标头：

```
x-ms-date: Fri, 17 Nov 2017 05:16:48 GMT
x-ms-version: 2017-07-29
Authorization: SharedKey contosorest:uzvWZN1WUIv2LYC6e3En10/7EIQJ5X9KtFQqrZkxi6s=
Host: contosorest.blob.core.windows.net
Connection: Keep-Alive
```

执行后返回的状态代码和响应标头：

```
HTTP/1.1 200 OK
Content-Type: application/xml
Server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 7e9316da-001e-0037-4063-5faf9d000000
x-ms-version: 2017-07-29
Date: Fri, 17 Nov 2017 05:20:21 GMT
Content-Length: 1135
```

**响应正文 (XML)：** 此 XML 响应显示 blob 及其属性列表。 

```xml
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults 
    ServiceEndpoint="http://contosorest.blob.core.windows.net/" ContainerName="container-1">
    <Blobs>
        <Blob>
            <Name>DogInCatTree.png</Name>
            <Properties><Last-Modified>Fri, 17 Nov 2017 01:41:14 GMT</Last-Modified>
            <Etag>0x8D52D5C4A4C96B0</Etag>
            <Content-Length>419416</Content-Length>
            <Content-Type>image/png</Content-Type>
            <Content-Encoding />
            <Content-Language />
            <Content-MD5 />
            <Cache-Control />
            <Content-Disposition />
            <BlobType>BlockBlob</BlobType>
            <LeaseStatus>unlocked</LeaseStatus>
            <LeaseState>available</LeaseState>
            <ServerEncrypted>true</ServerEncrypted>
            </Properties>
        </Blob>
        <Blob>
            <Name>GuyEyeingOreos.png</Name>
            <Properties>
                <Last-Modified>Fri, 17 Nov 2017 01:41:14 GMT</Last-Modified>
                <Etag>0x8D52D5C4A25A6F6</Etag>
                <Content-Length>167464</Content-Length>
                <Content-Type>image/png</Content-Type>
                <Content-Encoding />
                <Content-Language />
                <Content-MD5 />
                <Cache-Control />
                <Content-Disposition />
                <BlobType>BlockBlob</BlobType>
                <LeaseStatus>unlocked</LeaseStatus>
                <LeaseState>available</LeaseState>
                <ServerEncrypted>true</ServerEncrypted>
            </Properties>
            </Blob>
        </Blobs>
    <NextMarker />
</EnumerationResults>
```

## <a name="summary"></a>摘要

在本文中，你学习了如何向 blob 存储 REST API 发出请求，以检索容器列表或容器中的 blob 列表。 此外，还了解了如何创建 REST API 调用的授权签名，如何在 REST 请求中使用它，以及如何检查该响应。

## <a name="next-steps"></a>后续步骤

* [Blob 服务 REST API](/rest/api/storageservices/blob-service-rest-api)
* [文件服务 REST API](/rest/api/storageservices/file-service-rest-api)
* [Queue Service REST API](/rest/api/storageservices/queue-service-rest-api)（队列服务 REST API）
