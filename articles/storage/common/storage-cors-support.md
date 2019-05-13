---
title: 跨域资源共享 (CORS) 支持 | Microsoft Docs
description: 了解如何为 Microsoft Azure 存储服务启用 CORS 支持。
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 2/22/2017
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: bb296db0d97382deac984369704777de5d5cb362
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147692"
---
# <a name="cross-origin-resource-sharing-cors-support-for-the-azure-storage-services"></a>对 Azure 存储服务的跨域资源共享 (CORS) 支持
从版本 2013-08-15 开始，Azure 存储服务支持 Blob、表、队列和文件服务的跨域资源共享 (CORS)。 CORS 是一项 HTTP 功能，使在一个域中运行的 Web 应用程序能够访问另一个域中的资源。 Web 浏览器实施一种称为[同源策略](https://www.w3.org/Security/wiki/Same_Origin_Policy)的安全限制，防止网页调用不同域中的 API；CORS 提供了一种安全的方法，允许一个域（源域）调用其他域中的 API。 有关 CORS 的详细信息，请参阅 [CORS 规范](https://www.w3.org/TR/cors/)。

可分别为每个存储服务设置 CORS 规则，方式请参阅 [Set Blob Service Properties](https://msdn.microsoft.com/library/hh452235.aspx)（设置 Blob 服务属性）、[Set Queue Service Properties](https://msdn.microsoft.com/library/hh452232.aspx)（设置队列服务属性）和 [Set Table Service Properties](https://msdn.microsoft.com/library/hh452240.aspx)（设置表服务属性）。 为服务设置 CORS 规则后，会对从另一个域对服务发出的经过正确授权的请求进行评估，以根据指定的规则确定是否允许该请求。

> [!NOTE]
> 请注意，CORS 不是一种身份验证机制。 在启用 CORS 的情况下针对存储资源发出的任何请求必须具有适当的身份验证签名，或者必须是针对公共资源发出的。
> 
> 

## <a name="understanding-cors-requests"></a>了解 CORS 请求
来自源域的 CORS 请求可能由两个单独的请求组成：

* 预检请求，它查询服务施加的 CORS 限制。 仅当请求方法是[简单方法](https://www.w3.org/TR/cors/)（即 GET、HEAD 或 POST）时，才需要预检请求。
* 实际请求，它针对所需资源发出。

### <a name="preflight-request"></a>预检请求
预检请求查询由帐户所有者为存储服务设定的 CORS 限制。 Web 浏览器（或其他用户代理）发送一个包含请求标头、方法和源域的 OPTIONS 请求。 存储服务基于一组预配置的 CORS 规则评估预期操作，这些规则规定针对存储资源的实际请求中可以指定哪些源域、请求方法和请求标头。

如果为服务启用了 CORS，并且有一条 CORS 规则与预检请求相匹配，则服务将使用状态代码 200 (OK) 进行响应，并在响应中包含所需的 Access-Control 标头。

如果没有为服务启用 CORS，或者不存在与预检请求相匹配的 CORS 规则，服务将使用状态代码 403 (Forbidden) 进行响应。

如果 OPTIONS 请求不包含所需的 CORS 标头（Origin 和 Access-Control-Request-Method 标头），服务将使用状态代码 400 (Bad request) 进行响应。

请注意，将针对服务（Blob、队列和表）而不是针对请求的资源对预检请求进行评估。 帐户所有者必须已经在帐户服务属性中启用了 CORS，请求才能成功。

### <a name="actual-request"></a>实际请求
接受预检请求并返回响应后，浏览器会根据存储资源分发实际请求。 如果预检请求被拒绝，浏览器会立即拒绝实际请求。

实际请求会被视为针对存储服务的普通请求。 请求中的 Origin 标头表示该请求是一个 CORS 请求，服务会检查匹配的 CORS 规则。 如果找到匹配项，将向响应中添加 Access-Control 标头并将其发送回客户端。 如果找不到匹配项，则不会返回 CORS Access-Control 标头。

## <a name="enabling-cors-for-the-azure-storage-services"></a>为 Azure 存储服务启用 CORS
CORS 规则在服务级别设置，因此需要分别为每个服务（Blob、队列和表）启用或禁用 CORS。 默认情况下，对每个服务禁用 CORS。 要启用 CORS，需要使用版本 2013-08-15 或更高版本设置适当的服务属性，并向服务属性中添加 CORS 规则。 若要详细了解如何为服务启用或禁用 CORS 以及如何设置 CORS 规则，请参阅 [Set Blob Service Properties](https://msdn.microsoft.com/library/hh452235.aspx)（设置 Blob 服务属性）、[Set Queue Service Properties](https://msdn.microsoft.com/library/hh452232.aspx)（设置队列服务属性）和 [Set Table Service Properties](https://msdn.microsoft.com/library/hh452240.aspx)（设置表服务属性）。

下面是通过“设置服务属性”操作指定的一个 CORS 规则示例：

```xml
<Cors>    
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
        <MaxAgeInSeconds>200</MaxAgeInSeconds>
    </CorsRule>
<Cors>
```

下面描述了 CORS 规则中包含的每个元素：

* **AllowedOrigins**：允许通过 CORS 发出存储服务请求的源域。 源域是从中发出请求的域。 请注意，来源必须与用户代理发送到服务的来源完全相同，包括大小写。 也可以使用通配符“*”允许所有源域通过 CORS 发出请求。 在上面的示例中，域 http:\//www.contoso.com 和 http:\//www.fabrikam.com 可以使用 CORS 发出服务请求。
* **AllowedMethods**：源域可用于 CORS 请求的方法（HTTP 请求谓词）。 在上面的示例中，只允许 PUT 和 GET 请求。
* **AllowedHeaders**：源域可以在 CORS 请求上指定的请求标头。 在上面的示例中，允许所有以 x-ms-meta-data、x-ms-meta-target 和 x-ms-meta-abc 开头的元数据标头。 请注意，通配符“*”表示允许任何以指定前缀开头的标头。
* **ExposedHeaders**：可以在 CORS 请求响应中发送并由浏览器向请求发出方公开的响应标头。 在上面的示例中，指示浏览器公开任何以 x-ms-meta 开头的标头。
* **MaxAgeInSeconds**：浏览器应缓存预检 OPTIONS 请求的最长时间。

Azure 存储服务支持为 **AllowedHeaders** 和 **ExposedHeaders** 两个元素指定带前缀的标头。 若要允许某个标头类别，可以为该类别指定一个通用前缀。 例如，如果指定 *x-ms-meta** 作为带前缀的标头，将会建立一条与 x-ms-meta 开头的所有标头相匹配的规则。

以下限制适用于 CORS 规则：

* 每个存储服务（Blob、表和队列）最多可以指定五个 CORS 规则。
* 请求中的所有 CORS 规则设置（XML 标记除外）的最大大小不应超过 2 KB。
* 允许的标头、公开的标头或允许的源的长度不应超过 256 个字符。
* 允许的标头和公开的标头可能是：
  * 文本标头，其中提供了准确的标头名称，如 **x-ms-meta-processed**。 最多可在请求中指定 64 个文本标头。
  * 带前缀的标头，其中提供了标头的前缀，如 **x-ms-meta-data**\*。 以此方式指定前缀将允许或公开以给定前缀开头的所有标头。 最多可在请求中指定 2 个带前缀的标头。
* 在 **AllowedMethods** 元素中指定的方法（或 HTTP 谓词）必须与 Azure 存储服务 API 支持的方法一致。 支持的方法为 DELETE、GET、HEAD、MERGE、POST、OPTIONS 和 PUT。

## <a name="understanding-cors-rule-evaluation-logic"></a>了解 CORS 规则的评估逻辑
当存储服务收到预检请求或实际请求时，它会根据你通过适当的“设置服务属性”操作为该服务设定的 CORS 规则评估该请求。 按照 CORS 规则在“设置服务属性”操作的请求正文中的设置顺序对它们进行评估。

按如下所示评估 CORS 规则：

1. 首先，根据 **AllowedOrigins** 元素中列出的域检查请求的源域。 如果列表中包含源域，或者通过通配符“*”允许了所有域，则规则评估会继续进行。 如果列表中不包含源域，则请求失败。
2. 接下来，根据 **AllowedMethods** 元素中列出的方法检查请求的方法（或 HTTP 谓词）。 如果列表中包含所用方法，则规则评估会继续进行；否则，请求失败。
3. 如果请求的源域和方法与某个规则相匹配，则选择该规则来处理请求，并且不再评估其他规则。 但是，必须先根据 **AllowedHeaders** 元素中列出的标头对请求中指定的标头进行检查，请求才能成功。 如果发送的标头与允许的标头不匹配，请求会失败。

由于将按照规则在请求正文中出现的顺序处理规则，因此最佳做法建议在列表中最先指定与来源有关的限制性最强的规则，以便先对这些规则进行评估。 在列表的最后指定限制性较弱的规则，例如，允许所有来源的规则。

### <a name="example--cors-rules-evaluation"></a>示例 – CORS 规则评估
以下示例显示用于为存储服务设置 CORS 规则的操作的部分请求正文。 有关构造请求的详细信息，请参阅 [Set Blob Service Properties](https://msdn.microsoft.com/library/hh452235.aspx)（设置 Blob 服务属性）、[Set Queue Service Properties](https://msdn.microsoft.com/library/hh452232.aspx)（设置队列服务属性）和 [Set Table Service Properties](https://msdn.microsoft.com/library/hh452240.aspx)（设置表服务属性）。

```xml
<Cors>
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com</AllowedOrigins>
        <AllowedMethods>PUT,HEAD</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-blob-content-type, x-ms-blob-content-disposition</AllowedHeaders>
    </CorsRule>
    <CorsRule>
        <AllowedOrigins>*</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-blob-content-type, x-ms-blob-content-disposition</AllowedHeaders>
    </CorsRule>
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com</AllowedOrigins>
        <AllowedMethods>GET</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-client-request-id</AllowedHeaders>
    </CorsRule>
</Cors>
```

接下来，考虑以下 CORS 请求：

| 请求 |  |  | 响应 |  |
| --- | --- | --- | --- | --- |
| **方法** |**源** |**请求标头** |**规则匹配** |**结果** |
| **PUT** |http:\//www.contoso.com |x-ms-blob-content-type |第一条规则 |Success |
| **GET** |http:\//www.contoso.com |x-ms-blob-content-type |第二条规则 |成功 |
| **GET** |http:\//www.contoso.com |x-ms-client-request-id |第二条规则 |失败 |

第一个请求与第一条规则相匹配，源域与允许的来源相匹配，方法与允许的方法相匹配，标头与允许的标头相匹配，所以第一个请求成功。

第二个请求不与第一条规则相匹配，因为方法与允许的方法不相符。 但是它却与第二条规则相匹配，因此它也成功完成。

第三个请求与第二条规则中的源域和方法相匹配，因此不再评估其他规则。 但是，第二条规则不允许 *x-ms-client-request-id header* 标头，因此，尽管第三条规则的语义允许它成功，该请求仍会失败。

> [!NOTE]
> 虽然此示例先显示一个限制性较弱的规则，然后显示一个限制性更强的规则，但通常情况下，最佳做法是先列出限制性最强的规则。
> 
> 

## <a name="understanding-how-the-vary-header-is-set"></a>了解如何设置 Vary 标头
*Vary* 标头是一个由一组请求标头字段组成的标准 HTTP/1.1 标头，这些字段可告知浏览器或用户代理服务器选择用于处理该请求的条件。 *Vary* 标头主要用于代理、浏览器和 CDN 缓存，它们使用该标头来确定应如何缓存响应。 有关详细信息，请参阅 [Vary 标头](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)的规范。

当浏览器或其他用户代理缓存 CORS 请求的响应时，源域将作为允许的来源缓存。 如果第二个域在缓存处于活动时对存储资源发出相同请求，用户代理会检索缓存的源域。 由于第二个域与缓存的域不匹配，因此原本应该成功的请求会失败。 在某些情况下，Azure 存储会将 Vary 标头设置为 **Origin**，以便在发出请求的域不同于缓存的来源时，指示用户代理将后续 CORS 请求发送到服务。

在以下情况下，Azure 存储会针对实际 GET/HEAD 请求，将 *Vary* 标头设置为 **Origin**：

* 当请求来源与 CORS 规则定义的允许来源完全匹配时。 要想成为精确匹配项，CORS 规则不能包含 ' * ' 通配符。
* 不存在与请求源匹配的规则，但为存储服务启用了 CORS。

如果 GET/HEAD 请求与允许所有来源的 CORS 规则匹配，响应将指示允许所有来源，并且用户代理缓存会在缓存处于活动时允许来自任何源域的后续请求。

请注意，对于使用 GET/HEAD 之外的方法的请求，存储服务不会设置 Vary 标头，因为用户代理不会缓存对这些方法的响应。

下表指示了 Azure 存储如何基于前面提供的情况响应 GET/HEAD 请求：

| 请求 | 帐户设置和规则评估结果 |  |  | 响应 |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| **请求中存在 Origin 标头** |**为此服务指定了 CORS 规则** |**存在允许所有源 (*) 的匹配规则** |**存在精确匹配域的匹配规则** |**响应包含设置为 Origin 的 Vary 标头** |**响应包含 Access-Control-Allowed-Origin：“*”** |**响应包含 Access-Control-Exposed-Headers** |
| 否 |否 |否 |否 |否 |否 |否 |
| 否 |是 |否 |否 |是 |否 |否 |
| 否 |是 |是 |否 |否 |是 |是 |
| 是 |否 |否 |否 |否 |否 |否 |
| 是 |是 |否 |是 |是 |否 |是 |
| 是 |是 |否 |否 |是 |否 |否 |
| 是 |是 |是 |否 |否 |是 |是 |

## <a name="billing-for-cors-requests"></a>CORS 请求的计费
如果已按下述文章中的方式对帐户的任一存储服务启用了 CORS，会对成功的预检请求计费：[Set Blob Service Properties](https://msdn.microsoft.com/library/hh452235.aspx)（设置 Blob 服务属性）、[Set Queue Service Properties](https://msdn.microsoft.com/library/hh452232.aspx)（设置队列服务属性）或 [Set Table Service Properties](https://msdn.microsoft.com/library/hh452240.aspx)设置表服务属性）。 为了尽可能减少费用，可考虑将 CORS 规则中的 **MaxAgeInSeconds** 元素设置为一个较大的值，以便用户代理可以缓存请求。

将不会对失败的预检请求计费。

## <a name="next-steps"></a>后续步骤
[Set Blob Service Properties](https://msdn.microsoft.com/library/hh452235.aspx)（设置 Blob 服务属性）

[Set Queue Service Properties](https://msdn.microsoft.com/library/hh452232.aspx)（设置队列服务属性）

[Set Table Service Properties](https://msdn.microsoft.com/library/hh452240.aspx)（设置表服务属性）

[W3C 跨域资源共享规范](https://www.w3.org/TR/cors/)

