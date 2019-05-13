---
title: 在 Azure 存储中使用共享访问签名 (SAS) | Microsoft Docs
description: 了解使用共享访问签名 (SAS) 委派对 Azure 存储资源（包括 Blob、队列、表和文件）的访问权限。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/18/2017
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 25c562e144b635cb66c5df9b5b7bd6237ce3122c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154433"
---
# <a name="using-shared-access-signatures-sas"></a>使用共享访问签名 (SAS)

共享访问签名 (SAS) 提供一种方法，可将对存储帐户中对象的受限访问权限授予其他客户端且不必公开帐户密钥。 在本文中，我们将概述 SAS 模型、查看 SAS 最佳实践以及一些示例。

有关此处所述之外的使用 SAS 的其他代码示例，请参阅 [.NET 中的 Azure Blob 存储入门](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/)以及 [Azure 代码示例](https://azure.microsoft.com/documentation/samples/?service=storage)库提供的其他示例。 可以下载示例应用程序并运行，或在 GitHub 上浏览代码。

## <a name="what-is-a-shared-access-signature"></a>什么是共享访问签名？
共享访问签名对存储帐户中的资源提供委托访问。 通过 SAS，可以授予客户端对存储帐户中资源的访问权限，无需共享帐户密钥。 这是在应用程序中使用共享访问签名的关键之处 - SAS 是用于共享存储资源的一种安全方式，它不会危及帐户密钥。

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

通过 SAS，可以精细控制向拥有 SAS 的客户端授予的访问类型，包括：

* SAS 有效期限，包括开始时间和到期时间。
* SAS 授予的权限。 例如，Blob 的 SAS 可能授予对该 Blob 的读取和写入权限，但不授予删除权限。
* Azure 存储接受 SAS 的可选的 IP 地址或 IP 地址范围。 例如，可以指定属于组织的 IP 地址范围。
* Azure 存储接受 SAS 所依据的协议。 此可选参数可用于限制使用 HTTPS 对客户端的访问。

## <a name="when-should-you-use-a-shared-access-signature"></a>何时应使用共享访问签名？
需要将存储帐户中资源的访问权限提供给未处理存储帐户的访问密钥的任意客户端时，可以使用 SAS。 存储帐户包括主访问密钥和辅助访问密钥，这两种密钥都授予对帐户以及其中所有资源的管理访问权限。 公开这两种密钥的任何一种都会向可能的恶意或负面使用开放帐户。 共享访问签名提供一种安全的方法，允许客户端根据你显式授予的权限读取、写入和删除存储帐户中的数据，而无需帐户密钥。

SAS 通常适用于用户需要在存储帐户中读取和写入其数据的服务情形。 在存储帐户存储用户数据的情形中，有两种典型的设计模式：

1. 客户端通过执行身份验证的前端代理服务上传和下载数据。 此前端代理服务的优势在于允许验证业务规则，但对于大量数据或大量事务，创建可扩展以匹配需求的服务可能成本高昂或十分困难。

   ![方案示意图：前端代理服务](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png)   

1. 轻型服务按需对客户端进行身份验证，并生成 SAS。 在客户端接收 SAS 后，它们可以直接使用 SAS 定义的权限并且针对 SAS 允许的间隔访问存储帐户资源。 SAS 减少了通过前端代理服务路由所有数据的需要。

   ![方案示意图：SAS 提供程序服务](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png)   

许多实际服务可能会混合使用这两种方法。 例如，可能通过前端代理对某些数据进行处理和验证，同时使用 SAS 直接保存和/或读取其他数据。

此外，在某些情况下，需要使用 SAS 在复制操作中授予对源对象的访问权限：

* 将一个 Blob 复制到驻留在不同存储帐户中的另一个 Blob 时，必须使用 SAS 授予对源 Blob 的访问权限。 还可以选择使用 SAS 授予对目标 blob 的访问权限。
* 将一个文件复制到驻留在不同存储帐户中的另一个文件时，必须使用 SAS 授予对源文件的访问权限。 还可以选择使用 SAS 授予对目标文件的访问权限。
* 将一个 Blob 复制到一个文件，或将一个文件复制到一个 Blob 时，必须使用 SAS 授予对源对象的访问权限，即使源对象和目标对象驻留在同一存储帐户中也是如此。

## <a name="types-of-shared-access-signatures"></a>共享访问签名的类型
可创建两种类型的共享访问签名：

* **服务 SAS。** 服务 SAS 只能委派对以下一个存储服务中的资源的访问权限：Blob、队列、表或文件服务。 有关构造服务 SAS 令牌的深入信息，请参阅 [Constructing a Service SAS](https://msdn.microsoft.com/library/dn140255.aspx)（构造服务 SAS）和 [Service SAS Examples](https://msdn.microsoft.com/library/dn140256.aspx)（服务 SAS 示例）。
* **帐户 SAS。** 帐户 SAS 可委派对一个或多个存储服务中的资源的访问权限。 通过服务 SAS 提供的所有操作也可以通过帐户 SAS 提供。 此外，使用帐户 SAS，还可以委派对适用于给定服务的操作（例如，**获取/设置服务属性**和**获取服务统计信息**）的访问权限。还可以委派对 blob 容器、表、队列和文件共享执行读取、写入和删除操作的访问权限，而这是服务 SAS 所不允许的。 有关构造帐户 SAS 令牌的深入信息，请参阅 [Constructing an Account SAS](https://msdn.microsoft.com/library/mt584140.aspx)（构造帐户 SAS）。

## <a name="how-a-shared-access-signature-works"></a>共享访问签名的工作方式
共享访问签名是一种签名的 URI，它指向一个或多个存储资源并且包括包含一组特殊查询参数的令牌。 该令牌指示客户端可以如何访问资源。 签名是其中一个查询参数，它是由 SAS 参数构造的并且使用帐户密钥进行签名。 Azure 存储使用该签名授予对存储资源的访问权限。

下面是一个 SAS URI 示例，显示资源 URI 和 SAS 令牌：

![SAS URI 的组件](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-uri.png)   

SAS 令牌是在客户端侧生成的字符串（请参阅 [SAS 示例](#sas-examples)部分获取代码示例）。 例如，在任何情况下，Azure 存储均不会跟踪使用存储客户端库生成的 SAS 令牌。 可以在客户端上创建不限数量的 SAS 令牌。

当客户端将 SAS URI 作为请求的一部分提供到 Azure 存储时，服务会检查 SAS 参数和签名，以验证其对请求进行身份验证的有效性。 如果服务验证签名有效，则对请求进行授权。 否则，将拒绝请求，错误代码为 403（已禁止）。

## <a name="shared-access-signature-parameters"></a>共享访问签名参数
帐户 SAS 令牌和服务 SAS 令牌包括一些公用参数，但所采用的参数也有几个不同。

### <a name="parameters-common-to-account-sas-and-service-sas-tokens"></a>帐户 SAS 令牌和服务 SAS 令牌共有的参数
* **Api version** 指定要用于执行请求的存储服务版本的可选参数。
* **Service version** 必需参数，指定要用于对请求进行授权的存储服务版本。
* **Start time.** 这是 SAS 生效的时间。 共享访问签名的开始时间是可选的。 如果省略开始时间，SAS 将立即生效。 开始时间必须以 UTC（协调世界时）格式表示，并使用特殊的 UTC 指示符（“Z”），例如 `1994-11-05T13:15:30Z`。
* **Expiry time.** 这是之后 SAS 不再有效的时间。 最佳实践建议你或者为 SAS 指定到期时间，或者将其与某一存储访问策略相关联。 到期时间必须以 UTC（协调世界时）格式表示，并使用特殊的 UTC 指示符（“Z”），例如 `1994-11-05T13:15:30Z`（详见下文）。
* **权限。** 对 SAS 指定的权限指示客户端可使用 SAS 对存储资源执行哪些操作。 帐户 SAS 和服务 SAS 提供的权限不同。
* **IP.** 一个可选参数，它指定 Azure 外部要从中接受请求的一个 IP 地址或 IP 地址范围（有关 Express Route，请参阅[路由会话配置状态](../../expressroute/expressroute-workflows.md#routing-session-configuration-state)部分）。
* **Protocol.** 一个可选参数，它指定请求允许的协议。 可能的值包括“HTTPS 和 HTTP”(`https,http`)（它是默认值）或者“仅限 HTTPS”(`https`)。 请注意，“仅限 HTTP”是不允许的值。
* **Signature.** 签名由指定为部分令牌的其他参数构造，并进行加密。 使用该签名授予对指定存储资源的访问权限。

### <a name="parameters-for-a-service-sas-token"></a>服务 SAS 令牌的参数
* **存储资源。** 可以使用服务 SAS 为其委派访问权限的存储资源包括：
  * 容器和 Blob
  * 文件共享和文件
  * 队列
  * 表和表实体范围。

### <a name="parameters-for-an-account-sas-token"></a>帐户 SAS 令牌的参数
* **Service or services.** 帐户 SAS 可委派对一个或多个存储服务的访问权限。 例如，可以创建一个帐户 SAS 以委派对 Blob 和文件服务的访问权限。 也可以创建一个 SAS 以委派对所有四种服务（Blob、队列、表和文件）的访问权限。
* **存储资源类型。** 帐户 SAS 适用于一个或多个类别的存储资源，而不是特定资源。 可以创建帐户 SAS 以委派对以下项的访问权限：
  * 服务级别 API，将针对存储帐户资源调用它。 示例包括**获取/设置服务属性**、**获取服务统计信息**和**列出容器/队列/表/共享**。
  * 容器级别 API，将针对每个服务的容器对象调用它：blob 容器、队列、表和文件共享。 示例包括**创建/删除容器**、**创建/删除队列**、**创建/删除表**、**创建/删除共享**和**列出 Blob/文件和目录**。
  * 对象级别 API，将针对 blob、队列消息、表实体和文件调用它。 例如，**放置 Blob**、**查询实体**、**获取消息**和**创建文件**。

## <a name="examples-of-sas-uris"></a>SAS URI 的示例

### <a name="service-sas-uri-example"></a>服务 SAS URI 示例

下面是服务 SAS URI 的一个示例，它提供对某一 Blob 的读写权限。 该表分解了 URI 的每个部分，以便理解它是如何影响 SAS 的：

```
https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
```

| 名称 | SAS 部分 | 描述 |
| --- | --- | --- |
| Blob URI |`https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt` |Blob 的地址。 请注意，强烈建议使用 HTTPS。 |
| 存储服务版本 |`sv=2015-04-05` |对于存储服务版本 2012-02-12 和更高版本，此参数指示要使用的版本。 |
| 开始时间 |`st=2015-04-29T22%3A18%3A26Z` |以 UTC 时间格式指定。 如果想要 SAS 立即生效，则省略开始时间。 |
| 到期时间 |`se=2015-04-30T02%3A23%3A26Z` |以 UTC 时间格式指定。 |
| 资源 |`sr=b` |资源是 Blob。 |
| 权限 |`sp=rw` |SAS 授予的权限包括读取 (r) 和写入 (w)。 |
| IP 范围 |`sip=168.1.5.60-168.1.5.70` |将从中接受请求的 IP 地址范围。 |
| Protocol |`spr=https` |仅允许使用 HTTPS 的请求。 |
| 签名 |`sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D` |用于授予对 Blob 的访问权限。 该签名是利用 SHA256 算法通过“字符串到签名”和密钥进行计算，并使用 Base64 编码进行编码的 HMAC。 |

### <a name="account-sas-uri-example"></a>帐户 SAS URI 示例

下面是在令牌中使用相同的公用参数的帐户 SAS 的一个示例。 由于这些参数已在前面说明，因此不在此处对其进行说明。 下表中仅说明了特定于帐户 SAS 的参数。

```
https://myaccount.blob.core.windows.net/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B
```

| 名称 | SAS 部分 | 描述 |
| --- | --- | --- |
| 资源 URI |`https://myaccount.blob.core.windows.net/?restype=service&comp=properties` |Blob 服务终结点，包含用于获取服务属性（使用 GET 调用时）或设置服务属性（使用 SET 调用时）的参数。 |
| 服务 |`ss=bf` |该 SAS 适用于 Blob 和文件服务 |
| 资源类型 |`srt=s` |该 SAS 适用于服务级别操作。 |
| 权限 |`sp=rw` |这些权限向读取和写入操作授予访问权限。 |

鉴于权限仅限于服务级别，使用此 SAS 的可访问操作包括**获取 Blob 服务属性**（读取）和**设置 Blob 服务属性**（写入）。 但是，使用其他资源 URI，同一个 SAS 令牌还可用于委派对**获取 Blob 服务统计信息**（读取）的访问权限。

## <a name="controlling-a-sas-with-a-stored-access-policy"></a>使用存储访问策略控制 SAS
共享访问签名可以采取以下两种形式的一种：

* **临时 SAS：** 在创建临时 SAS 时，针对该 SAS 的开始时间、到期时间和权限都在 SAS URI 中指定（在省略开始时间的情况下，也可以是隐式的）。 这种类型的 SAS 可以创建为帐户 SAS 或服务 SAS。
* **具有存储访问策略的 SAS：** 存储访问策略是对资源容器（Blob 容器、表、队列或文件共享）定义的，可用于管理针对一个或多个共享访问签名的约束。 在将某一 SAS 与一个存储访问策略相关联时，该 SAS 将继承对该存储访问策略定义的约束：开始时间、到期时间和权限。

> [!NOTE]
> 目前，帐户 SAS 必须是一个临时 SAS。 帐户 SAS 尚不支持存储访问策略。

这两种形式之间的差异对于一个关键情形而言十分重要：吊销。 一个 SAS URI 就是一个 URL，因此，获取该 SAS 的任何人都可以使用它，而与谁是最初的创建者无关。 如果 SAS 是公开发布的，则世界上的任何人都可以使用它。 SAS 向所有对其进行处理的人授予资源的访问权限，直到发生以下 4 种情况之一：

1. 达到了对该 SAS 指定的到期时间。
2. 达到了对该 SAS 引用的存储访问策略指定的到期时间（如果引用存储访问策略并且该存储访问策略指定一个到期时间）。 这可能是因为经过了该间隔而发生，或者是你修改了该存储访问策略而使到期时间已经是过去时间而发生（这是用于吊销该 SAS 的一种方法）。
3. 删除了该 SAS 引用的存储访问策略，这是用于吊销 SAS 的另一种方法。 请注意，如果使用完全相同的名称重新创建该存储访问策略，则根据与该存储访问策略相关联的权限，所有现有 SAS 标记都将再次有效（假定尚未经过该 SAS 的到期时间）。 如果想要吊销该 SAS，请确保使用不同时间（如果使用将来的到期时间重新创建该访问策略）。
4. 将重新生成用于创建 SAS 的帐户密钥。 重新生成帐户密钥会导致使用该密钥的所有应用程序组件授权失败，直到这些组件更新为使用另一个有效帐户密钥或者新生成的帐户密钥。

> [!IMPORTANT]
> 共享访问签名 URI 与用于创建签名的帐户密钥和关联的存储访问策略（如果有）相关联。 如果未指定存储访问策略，则吊销共享访问签名的唯一方法是更改帐户密钥。

## <a name="authenticating-from-a-client-application-with-a-sas"></a>使用 SAS 从客户端应用程序进行身份验证
拥有 SAS 的客户端可以使用 SAS 针对没有帐户密钥的存储帐户对请求进行授权。 SAS 可包含于连接字符串中，或直接从适当的构造函数或方法中使用。

### <a name="using-a-sas-in-a-connection-string"></a>在连接字符串中使用 SAS
[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

### <a name="using-a-sas-in-a-constructor-or-method"></a>在构造函数或方法中使用 SAS
一些 Azure 存储客户端库构造函数和方法重载提供 SAS 参数，以便可以使用 SAS 对服务的请求进行授权。

例如，此处使用 SAS URI 创建对块 blob 的引用。 SAS 提供请求所需的唯一凭据。 然后将块 blob 引用用于写入操作：

```csharp
string sasUri = "https://storagesample.blob.core.windows.net/sample-container/" +
    "sampleBlob.txt?sv=2015-07-08&sr=b&sig=39Up9JzHkxhUIhFEjEH9594DJxe7w6cIRCg0V6lCGSo%3D" +
    "&se=2016-10-18T21%3A51%3A37Z&sp=rcw";

CloudBlockBlob blob = new CloudBlockBlob(new Uri(sasUri));

// Create operation: Upload a blob with the specified name to the container.
// If the blob does not exist, it will be created. If it does exist, it will be overwritten.
try
{
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
    {
        await blob.UploadFromStreamAsync(msWrite);
    }

    Console.WriteLine("Create operation succeeded for SAS {0}", sasUri);
    Console.WriteLine();
}
catch (StorageException e)
{
    if (e.RequestInformation.HttpStatusCode == 403)
    {
        Console.WriteLine("Create operation failed for SAS {0}", sasUri);
        Console.WriteLine("Additional error information: " + e.Message);
        Console.WriteLine();
    }
    else
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}

```

## <a name="best-practices-when-using-sas"></a>使用 SAS 的最佳实践
在应用程序中使用共享访问签名时，需要知道以下两个可能的风险：

* 如果 SAS 泄露，则获取它的任何人都可以使用它，这可能会损害存储帐户。
* 如果提供给客户端应用程序的 SAS 到期并且应用程序无法从服务检索新 SAS，则可能会影响该应用程序的功能。

下面这些针对使用共享访问签名的建议可帮助降低这些风险：

1. **始终使用 HTTPS** 创建或分发 SAS。 如果某一 SAS 通过 HTTP 传递并且被截取，则执行中间人攻击的攻击者能够读取 SAS、然后使用它，就像目标用户本可执行的操作一样，这可能会暴露敏感数据或者使恶意用户能够损坏数据。
2. **尽可能参照存储访问策略。** 存储访问策略使你可以选择撤销权限而不必重新生成存储帐户密钥。 将针对 SAS 的到期时间设置为很久之后的某一时间（或者无限远），并且确保定期对其进行更新以便将到期时间移到将来的更远时间。
3. **对临时 SAS 使用近期的到期时间。** 这样，即使某一 SAS 泄露，它也只会在短期内有效。 如果无法参照某一存储访问策略，该行为尤其重要。 临时到期时间还通过限制可用于上传到它的时间来限制可以写入 Blob 的数据量。
4. **如果需要，让客户端自动续订 SAS。** 客户端应在到期时间之前很久就续订 SAS，这样，即使提供 SAS 的服务不可用，客户端也有时间重试。 如果 SAS 旨在用于少量即时的短期操作，这些操作应在到期时间内完成，则上述做法可能是不必要的，因为不应续订 SAS。 但是，如果客户端定期通过 SAS 发出请求，则有效期可能就会起作用。 需要考虑的主要方面就是在以下两者间进行权衡：对短期 SAS 的需求（如前文所述）以及确保客户端尽早请求续订（以免在成功续订前因 SAS 到期而中断）。
5. **要注意 SAS 开始时间。** 如果将 SAS 的开始时间设置为**现在**，则由于时钟偏移（根据不同计算机，当前时间中的差异），在前几分钟会暂时观察到失败。 通常，将开始时间至少设置为 15 分钟前。 或者根本不设置，这会使它在所有情况下都立即生效。 同样原则也适用于到期时间 - 请记住，对于任何请求，在任一方向可能会观察到最多 15 分钟的时钟偏移。 对于使用 2012-02-12 之前的 REST 版本的客户端，未参照某一存储访问策略的 SAS 的最大持续时间是 1 小时，指定超过 1 小时持续时间的任何策略都会失败。
6. **对要访问的资源要具体。** 一个安全性最佳实践是向用户提供所需最小权限。 如果某一用户仅需要对单个实体的读取访问权限，则向该用户授予对该单个实体的读取访问权限，而不要授予针对所有实体的读取/写入/删除访问权限。 如果 SAS 泄露，这也有助于降低损失，因为攻击者手中掌握的 SAS 的权限较为有限。
7. **了解对任何使用都将向帐户收费，包括使用 SAS 所做的工作。** 如果向你提供了针对某一 Blob 的写访问权限，用户可以选择上传 200GB Blob。 如果还向用户提供了对 Blob 的读访问权限，他们可能会选择下载 Blob 10 次，对你产生 2 TB 的传出费用。 此外，提供受限权限，帮助降低恶意用户的潜在操作威胁。 使用短期 SAS 以便减少这一威胁（但要注意结束时间上的时钟偏移）。
8. **验证使用 SAS 写入的数据。** 在某一客户端应用程序将数据写入存储帐户时，请记住对于这些数据可能存在问题。 如果应用程序要求在数据可供使用前对数据进行验证或授权，应该在写入数据后、但在应用程序使用这些数据前执行此验证。 这一实践还有助于防止损坏的数据或恶意数据写入帐户，这些数据可能是正常要求 SAS 的用户写入的，也可能是利用泄露的 SAS 的用户写入的。
9. **不要总是使用 SAS。** 有时候，与针对存储帐户的特定操作相关联的风险要超过 SAS 所带来的好处。 对于此类操作，应创建一个中间层服务，该服务在执行业务规则验证、身份验证和审核后写入存储帐户。 此外，有时候以其他方式管理访问会更简单。 例如，如果想要使某一容器中的所有 Blob 都可以公开读取，则可以使该容器成为公共的，而不是为每个客户端都提供 SAS 来进行访问。
10. **使用存储分析监视应用程序。** 可以使用日志记录和指标来观察由于 SAS 提供程序服务中断或无意中删除存储访问策略而导致身份验证失败的任何高发情形。 有关其他信息，请参阅 [Azure 存储团队博客](https://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx)。

## <a name="sas-examples"></a>SAS 示例
下面是两种类型的共享访问签名（帐户 SAS 和服务 SAS）的一些示例。

若要运行这些 C# 示例，需要在项目中引用以下 NuGet 包：

* [适用于 .NET 的 Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage) 6.x 或更高版本（以便使用帐户 SAS）。
* [Azure 配置管理器](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager)

有关演示如何创建和测试 SAS 的其他示例，请参阅[存储的 Azure 代码示例](https://azure.microsoft.com/documentation/samples/?service=storage)。

### <a name="example-create-and-use-an-account-sas"></a>示例：创建并使用帐户 SAS
以下代码示例将创建一个帐户 SAS，该 SAS 对 Blob 和文件服务是有效的，并授予客户端读取、写入和列表权限，使其能够访问服务级别 API。 帐户 SAS 将协议限制为 HTTPS，因此请求必须使用 HTTPS 发出。

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use your shared key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

若要使用帐户 SAS 访问 Blob 服务的服务级别 API，请使用 SAS 和存储帐户的 Blob 存储终结点构造 Blob 客户端对象。

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "account-name", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

### <a name="example-create-a-stored-access-policy"></a>示例：创建存储访问策略
下面的代码在容器上创建存储访问策略。 可以使用访问策略指定对容器或其 Blob 上的服务 SAS 的约束。

```csharp
private static async Task CreateSharedAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new shared access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
        // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Delete
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

### <a name="example-create-a-service-sas-on-a-container"></a>示例：在容器上创建服务 SAS
下面的代码在容器上创建 SAS。 如果提供现有存储访问策略的名称，则该策略与 SAS 关联。 如果未提供存储访问策略，则代码会在容器上创建一个临时 SAS。

```csharp
private static string GetContainerSasUri(CloudBlobContainer container, string storedPolicyName = null)
{
    string sasContainerToken;

    // If no stored policy is specified, create a new access policy and define its constraints.
    if (storedPolicyName == null)
    {
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocPolicy = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List
        };

        // Generate the shared access signature on the container, setting the constraints directly on the signature.
        sasContainerToken = container.GetSharedAccessSignature(adHocPolicy, null);

        Console.WriteLine("SAS for blob container (ad hoc): {0}", sasContainerToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the container. In this case, all of the constraints for the
        // shared access signature are specified on the stored access policy, which is provided by name.
        // It is also possible to specify some constraints on an ad hoc SAS and others on the stored access policy.
        sasContainerToken = container.GetSharedAccessSignature(null, storedPolicyName);

        Console.WriteLine("SAS for blob container (stored access policy): {0}", sasContainerToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return container.Uri + sasContainerToken;
}
```

### <a name="example-create-a-service-sas-on-a-blob"></a>示例：在 Blob 上创建服务 SAS
下面的代码在 Blob 上创建 SAS。 如果提供现有存储访问策略的名称，则该策略与 SAS 关联。 如果未提供存储访问策略，则代码会在 Blob 上创建一个临时 SAS。

```csharp
private static string GetBlobSasUri(CloudBlobContainer container, string blobName, string policyName = null)
{
    string sasBlobToken;

    // Get a reference to a blob within the container.
    // Note that the blob may not exist yet, but a SAS can still be created for it.
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    if (policyName == null)
    {
        // Create a new access policy and define its constraints.
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocSAS = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create
        };

        // Generate the shared access signature on the blob, setting the constraints directly on the signature.
        sasBlobToken = blob.GetSharedAccessSignature(adHocSAS);

        Console.WriteLine("SAS for blob (ad hoc): {0}", sasBlobToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the blob. In this case, all of the constraints for the
        // shared access signature are specified on the container's stored access policy.
        sasBlobToken = blob.GetSharedAccessSignature(null, policyName);

        Console.WriteLine("SAS for blob (stored access policy): {0}", sasBlobToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return blob.Uri + sasBlobToken;
}
```

## <a name="conclusion"></a>结束语
共享访问签名用于将存储帐户的受限权限提供给不应具有帐户密钥的客户端。 因此，它们是安全模型的重要环节，适合使用 Azure 存储的任何应用程序。 如果按照本文中介绍的最佳实践执行，则可以使用 SAS 更灵活地访问存储帐户中的资源，且不会影响应用程序的安全性。

## <a name="next-steps"></a>后续步骤
* [共享访问签名第 2 部分：创建 SAS 并将其用于 Blob 存储](../blobs/storage-dotnet-shared-access-signature-part-2.md)
* [管理对容器和 blob 的匿名读取访问](../blobs/storage-manage-access-to-resources.md)
* [Delegating Access with a Shared Access Signature](https://msdn.microsoft.com/library/azure/ee395415.aspx)（使用共享访问签名委托访问）
* [介绍表和队列 SAS](https://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
