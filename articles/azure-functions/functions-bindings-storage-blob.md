---
title: Azure Functions 的 Azure Blob 存储绑定
description: 了解如何在 Azure Functions 中使用 Azure Blob 存储触发器和绑定。
services: functions
documentationcenter: na
author: craigshoemaker
manager: jeconnoc
keywords: Azure Functions，函数，事件处理，动态计算，无服务体系结构
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 11/15/2018
ms.author: cshoe
ms.openlocfilehash: e4ec13453c204885f38b10272e76245e641fbef9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203598"
---
# <a name="azure-blob-storage-bindings-for-azure-functions"></a>Azure Functions 的 Azure Blob 存储绑定

本文介绍如何在 Azure Functions 中使用 Azure Blob 存储绑定。 Azure Functions 支持 Blob 的触发器、输入和输出绑定。 本文各部分分别介绍每种绑定：

* [blob 触发器](#trigger)
* [blob 输入绑定](#input)
* [blob 输出绑定](#output)

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE]
> 将事件网格触发器而非 Blob 存储触发器用于仅限 Blob 的存储帐户、大规模，或者降低延迟。 有关详细信息，请参阅[触发器](#trigger)部分。

## <a name="packages---functions-1x"></a>包 - Functions 1.x

[Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet 包 2.x 版中提供了 Blob 存储绑定。 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Blob) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="packages---functions-2x"></a>包 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet 包 3.x 版中提供了 Blob 存储绑定。 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs) GitHub 存储库中提供了此包的源代码。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="trigger"></a>触发器

检测到新的或更新的 Blob 时，Blob 存储触发器会启动某个函数。 Blob 内容会作为输入提供给函数。

[事件网格触发器](functions-bindings-event-grid.md)提供对 [Blob 事件](../storage/blobs/storage-blob-event-overview.md)的内置支持，也可用来在检测到新的或更新的 Blob 时启动函数。 如需示例，请参阅[使用事件网格调整图像大小](../event-grid/resize-images-on-storage-blob-upload-event.md)教程。

以下方案请使用事件网格而不是 Blob 存储触发器：

* Blob 存储帐户
* 大规模
* 最大程度地降低延迟

### <a name="blob-storage-accounts"></a>Blob 存储帐户

[Blob 存储帐户](../storage/common/storage-account-overview.md#types-of-storage-accounts)适用于 Blob 输入和输出绑定，但不适用于 Blob 触发器。 Blob 存储触发器需要使用常规用途存储帐户。

### <a name="high-scale"></a>大规模

大规模可以宽松地定义为包含 100,000 个以上的 Blob 的容器，或者定义为每秒进行 100 个以上 Blob 更新的存储帐户。

### <a name="latency-issues"></a>延迟问题

如果函数应用基于消耗计划，则当函数应用处于空闲状态时，处理新 Blob 会出现长达 10 分钟的延迟。 若要避免此延迟，可以切换到启用了 Always On 的应用服务计划。 还可以为 Blob 存储帐户使用[事件网格触发器](functions-bindings-event-grid.md)。 有关示例，请参阅[事件网格教程](../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2Fazure%2Fazure-functions%2Ftoc.json)。 

### <a name="queue-storage-trigger"></a>队列存储触发器

除了事件网格，还可以使用其他方法来处理 Blob，例如使用队列存储触发器，但后者没有针对 Blob 事件的内置支持。 在创建或更新 Blob 时，必须创建队列消息。 如需假定你已完成该操作的示例，请参阅[本文后面部分的 Blob 输入绑定示例](#input---example)。

## <a name="trigger---example"></a>触发器 - 示例

参阅语言特定的示例：

* [C#](#trigger---c-example)
* [C# 脚本 (.csx)](#trigger---c-script-example)
* [Java](#trigger---java-example)
* [JavaScript](#trigger---javascript-example)
* [Python](#trigger---python-example)

### <a name="trigger---c-example"></a>触发器 - C# 示例

以下示例演示在 `samples-workitems` 容器中添加或更新 blob 时写入日志的 [C# 函数](functions-dotnet-class-library.md)。

```csharp
[FunctionName("BlobTriggerCSharp")]        
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

blob 触发器路径 `samples-workitems/{name}` 中的字符串 `{name}` 会创建一个[绑定表达式](./functions-bindings-expressions-patterns.md)，可以在函数代码中使用它来访问触发 blob 的文件名。 有关详细信息，请参阅本文下文中的 [Blob 名称模式](#trigger---blob-name-patterns)。

有关 `BlobTrigger` 特性的详细信息，请参阅[触发器 - 特性](#trigger---attributes)。

### <a name="trigger---c-script-example"></a>触发器 - C# 脚本示例

以下示例显示了 *function.json* 文件中的一个 Blob 触发器绑定以及使用该绑定的 [Python 代码](functions-reference-python.md)。 在 `samples-workitems` [容器](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources)中添加或更新 Blob 时，该函数会写入一条日志。

下面是 function.json 文件中的绑定数据：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

blob 触发器路径 `samples-workitems/{name}` 中的字符串 `{name}` 会创建一个[绑定表达式](./functions-bindings-expressions-patterns.md)，可以在函数代码中使用它来访问触发 blob 的文件名。 有关详细信息，请参阅本文下文中的 [Blob 名称模式](#trigger---blob-name-patterns)。

有关 *function.json* 文件属性的详细信息，请参阅解释了这些属性的[配置](#trigger---configuration)部分。

下面是绑定到 `Stream` 的 C# 脚本代码：

```cs
public static void Run(Stream myBlob, string name, ILogger log)
{
   log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

下面是绑定到 `CloudBlockBlob` 的 C# 脚本代码：

```cs
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Blob;

public static void Run(CloudBlockBlob myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name}\nURI:{myBlob.StorageUri}");
}
```

### <a name="trigger---javascript-example"></a>触发器 - JavaScript 示例

以下示例显示了 *function.json* 文件中的一个 Blob 触发器绑定以及使用该绑定的 [JavaScript 代码](functions-reference-node.md)。 在 `samples-workitems` 容器中添加或更新 Blob 时，该函数会写入日志。

function.json 文件如下所示：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

blob 触发器路径 `samples-workitems/{name}` 中的字符串 `{name}` 会创建一个[绑定表达式](./functions-bindings-expressions-patterns.md)，可以在函数代码中使用它来访问触发 blob 的文件名。 有关详细信息，请参阅本文下文中的 [Blob 名称模式](#trigger---blob-name-patterns)。

有关 *function.json* 文件属性的详细信息，请参阅解释了这些属性的[配置](#trigger---configuration)部分。

JavaScript 代码如下所示：

```javascript
module.exports = function(context) {
    context.log('Node.js Blob trigger function processed', context.bindings.myBlob);
    context.done();
};
```

### <a name="trigger---python-example"></a>触发器 - Python 示例

以下示例显示了 *function.json* 文件中的一个 Blob 触发器绑定以及使用该绑定的 [Python 代码](functions-reference-python.md)。 在 `samples-workitems` [容器](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources)中添加或更新 Blob 时，该函数会写入一条日志。

function.json 文件如下所示：

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,
    "bindings": [
        {
            "name": "myblob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

blob 触发器路径 `samples-workitems/{name}` 中的字符串 `{name}` 会创建一个[绑定表达式](./functions-bindings-expressions-patterns.md)，可以在函数代码中使用它来访问触发 blob 的文件名。 有关详细信息，请参阅本文下文中的 [Blob 名称模式](#trigger---blob-name-patterns)。

有关 *function.json* 文件属性的详细信息，请参阅解释了这些属性的[配置](#trigger---configuration)部分。

下面是 Python 代码：

```python
import logging
import azure.functions as func

def main(myblob: func.InputStream):
    logging.info('Python Blob trigger function processed %s', myblob.name)
```

### <a name="trigger---java-example"></a>触发器 - Java 示例

以下示例显示了 function.json 文件中的一个 Blob 触发器绑定以及使用该绑定的 [Java 代码](functions-reference-java.md)。 在 `myblob` 容器中添加或更新 Blob 时，该函数会写入日志。

function.json 文件如下所示：

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "file",
            "type": "blobTrigger",
            "direction": "in",
            "path": "myblob/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

下面是 Java 代码：

```java
@FunctionName("blobprocessor")
public void run(
  @BlobTrigger(name = "file",
               dataType = "binary",
               path = "myblob/{name}",
               connection = "MyStorageAccountAppSetting") byte[] content,
  @BindingName("name") String filename,
  final ExecutionContext context
) {
  context.getLogger().info("Name: " + filename + " Size: " + content.length + " bytes");
}
```


## <a name="trigger---attributes"></a>触发器 - 特性

对于 [C# 类库](functions-dotnet-class-library.md)，请使用以下属性来配置 blob 触发器：

* [BlobTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/BlobTriggerAttribute.cs)

  该特性的构造函数采用一个表示要监视的容器的路径字符串，并选择性地采用某种 [Blob 名称模式](#trigger---blob-name-patterns)。 下面是一个示例：

  ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
  ```

  可以设置 `Connection` 属性来指定要使用的存储帐户，如以下示例中所示：

   ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}", Connection = "StorageConnectionAppSetting")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
   ```

  有关完整示例，请参阅[触发器 - C# 示例](#trigger---c-example)。

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  提供另一种方式来指定要使用的存储帐户。 构造函数采用包含存储连接字符串的应用设置的名称。 可以在参数、方法或类级别应用该特性。 以下示例演示类级别和方法级别：

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("BlobTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ....
  }
  ```

要使用的存储帐户按以下顺序确定：

* `BlobTrigger` 特性的 `Connection` 属性。
* 作为 `BlobTrigger` 特性应用到同一参数的 `StorageAccount` 特性。
* 应用到函数的 `StorageAccount` 特性。
* 应用到类的 `StorageAccount` 特性。
* 函数应用的默认存储帐户（“AzureWebJobsStorage”应用设置）。

## <a name="trigger---configuration"></a>触发器 - 配置

下表解释了在 function.json 文件和 `BlobTrigger` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
|类型 | 不适用 | 必须设置为 `blobTrigger`。 在 Azure 门户中创建触发器时，会自动设置此属性。|
|direction | 不适用 | 必须设置为 `in`。 在 Azure 门户中创建触发器时，会自动设置此属性。 [用法](#trigger---usage)部分中已阐述异常。 |
|**name** | 不适用 | 表示函数代码中的 Blob 的变量的名称。 |
|**路径** | **BlobPath** |要监视的[容器](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources)。  可以是某种 [Blob 名称模式](#trigger---blob-name-patterns)。 |
|**连接** | **Connection** | 包含要用于此绑定的存储连接字符串的应用设置的名称。 如果应用设置名称以“AzureWebJobs”开始，则只能在此处指定该名称的余下部分。 例如，如果将 `connection` 设置为“MyStorage”，函数运行时将会查找名为“AzureWebJobsMyStorage”的应用设置。 如果将 `connection` 留空，函数运行时将使用名为 `AzureWebJobsStorage` 的应用设置中的默认存储连接字符串。<br><br>连接字符串必须属于某个常规用途存储帐户，而不能属于[Blob 存储帐户](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>触发器 - 用法

在 C# 和 C# 脚本中，可以为触发 blob 使用以下参数类型：

* `Stream`
* `TextReader`
* `string`
* `Byte[]`
* 可序列化为 JSON 的 POCO
* `ICloudBlob`<sup>1</sup>
* `CloudBlockBlob`<sup>1</sup>
* `CloudPageBlob`<sup>1</sup>
* `CloudAppendBlob`<sup>1</sup>

<sup>1</sup> function.json 中需有 "inout" 绑定 `direction` 或 C# 类库中需有 `FileAccess.ReadWrite`。

如果尝试绑定到某个存储 SDK 类型并收到错误消息，请确保已引用[正确的存储 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

由于整个 Blob 内容都会加载到内存中，因此，只有当 Blob 较小时才建议绑定到 `string`、`Byte[]` 或 POCO。 平时，最好使用 `Stream` 或 `CloudBlockBlob` 类型。 有关详细信息，请参阅本文后面的[并发和内存使用情况](#trigger---concurrency-and-memory-usage)。

在 JavaScript 中，可以使用 `context.bindings.<name from function.json>` 访问输入 Blob 数据。

## <a name="trigger---blob-name-patterns"></a>触发器 - Blob 名称模式

可以在 *function.json* 的 `path` 属性中或者在 `BlobTrigger` 特性构造函数中指定 Blob 名称模式。 名称模式可以是[筛选器或绑定表达式](./functions-bindings-expressions-patterns.md)。 以下部分提供了有关示例。

### <a name="get-file-name-and-extension"></a>获取文件名和扩展名

以下示例演示如何分别绑定到 Blob 文件名和扩展名：

```json
"path": "input/{blobname}.{blobextension}",
```
如果 Blob 名为 *original-Blob1.txt*，则函数代码中 `blobname` 和 `blobextension` 变量的值为 *original-Blob1* 和 *txt*。

### <a name="filter-on-blob-name"></a>按 Blob 名称筛选

以下示例仅根据 `input` 容器中以字符串“original-”开头的 Blob 触发：

```json
"path": "input/original-{name}",
```

如果 Blob 名称为 *original-Blob1.txt*，则函数代码中 `name` 变量的值为 `Blob1`。

### <a name="filter-on-file-type"></a>按文件类型筛选

以下示例仅根据 *.png* 文件触发：

```json
"path": "samples/{name}.png",
```

### <a name="filter-on-curly-braces-in-file-names"></a>按文件名中的大括号筛选

若要查找文件名中的大括号，请使用两个大括号将大括号转义。 以下示例筛选名称中包含大括号的 Blob：

```json
"path": "images/{{20140101}}-{name}",
```

如果 Blob 名为 *{20140101}-soundfile.mp3*，则函数代码中的 `name` 变量值为 *soundfile.mp3*。

## <a name="trigger---metadata"></a>触发器 - 元数据

Blob 触发器提供了几个元数据属性。 这些属性可在其他绑定中用作绑定表达式的一部分，或者用作代码中的参数。 这些值的语义与 [CloudBlob](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.cloudblob?view=azure-dotnet) 类型相同。

|属性  |Type  |描述  |
|---------|---------|---------|
|`BlobTrigger`|`string`|触发 Blob 的路径。|
|`Uri`|`System.Uri`|主位置的 blob 的 URI。|
|`Properties` |[BlobProperties](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.blobproperties)|Blob 的系统属性。 |
|`Metadata` |`IDictionary<string,string>`|Blob 的用户定义元数据。|

例如，以下 C# 脚本和 JavaScript 示例会记录触发 blob 的路径，包括容器：

```csharp
public static void Run(string myBlob, string blobTrigger, ILogger log)
{
    log.LogInformation($"Full blob path: {blobTrigger}");
} 
```

```javascript
module.exports = function (context, myBlob) {
    context.log("Full blob path:", context.bindingData.blobTrigger);
    context.done();
};
```

## <a name="trigger---blob-receipts"></a>触发器 - Blob 回执

Azure Functions 运行时确保没有为相同的新 blob 或更新 blob 多次调用 blob 触发器函数。 为了确定是否已处理给定的 blob 版本，它会维护 blob 回执。

Azure Functions 将 Blob 回执存储在函数应用的 Azure 存储帐户中名为 azure-webjobs-hosts 的容器中（由 `AzureWebJobsStorage` 应用设置定义）。 Blob 回执包含以下信息：

* 触发的函数（"&lt;function app name>.Functions.&lt;function name>"，例如："MyFunctionApp.Functions.CopyBlob"）
* 容器名称
* Blob 类型（"BlockBlob" 或 "PageBlob"）
* Blob 名称
* ETag（blob 版本标识符，例如："0x8D1DC6E70A277EF"）

若要强制重新处理某个 blob，可从 azure-webjobs-hosts 容器中手动删除该 blob 的 blob 回执。 虽然重新处理可能会立即发生，保证在稍后的某个时刻发生的时间。

## <a name="trigger---poison-blobs"></a>触发器 - 有害 Blob

当给定 blob 的 blob 触发函数失败时，Azure Functions 将默认重试该函数共计 5 次。

如果 5 次尝试全部失败，Azure Functions 会将消息添加到名为 webjobs-blobtrigger-poison 的存储队列。 有害 Blob 的队列消息是包含以下属性的 JSON 对象：

* FunctionId（格式为 &lt;function app name>.Functions.&lt;function name>）
* BlobType（"BlockBlob" 或 "PageBlob"）
* ContainerName
* BlobName
* ETag（blob 版本标识符，例如："0x8D1DC6E70A277EF"）

## <a name="trigger---concurrency-and-memory-usage"></a>触发器 - 并发和内存使用情况

Blob 触发器可在内部使用队列，因此并发函数调用的最大数量受 [host.json 中的队列配置](functions-host-json.md#queues)控制。 默认设置会将并发限制到 24 个调用。 此限制分别应用于使用 blob 触发器的函数。

[消耗计划](functions-scale.md#how-the-consumption-and-premium-plans-work)将虚拟机 (VM) 上的函数应用限制为 1.5 GB 内存。 内存由每个并发执行函数实例和函数运行时本身使用。 如果 blob 触发的函数将整个 blob 加载到内存中，该函数使用的仅用于 blob 的最大内存为 24 * 最大 blob 大小。 例如，包含 3 个由 blob 触发的函数的函数应用和默认设置，其每 VM 最大并发为 3*24 = 72 个函数调用。

JavaScript 和 Java 函数会将整个 blob 加载到内存中，并且如果绑定到 `string`、`Byte[]` 或 POCO，则 C# 函数也会如此。

## <a name="trigger---polling"></a>触发器 - 轮询

如果正在监视的 blob 容器包含 10,000 多个 blob （跨所有容器），Functions 运行时会扫描日志文件，监视新的或更改 blob。 此过程可能会导致延迟。 创建 blob 之后数分钟或更长时间内可能仍不会触发函数。

> [!WARNING]
> 此外，[将“尽力”创建存储日志](/rest/api/storageservices/About-Storage-Analytics-Logging)。 不保证捕获所有事件。 在某些情况下可能会遗漏某些日志。
> 
> 如果需要更快或更可靠的 blob 处理，在创建 blob 时，请考虑创建[队列消息](../storage/queues/storage-dotnet-how-to-use-queues.md)。 然后，使用[队列触发器](functions-bindings-storage-queue.md)而不是 Blob 触发器来处理 Blob。 另一个选项是使用事件网格；请参阅教程[使用事件网格自动调整上传图像的大小](../event-grid/resize-images-on-storage-blob-upload-event.md)。
>

## <a name="input"></a>输入

使用 blob 存储输入绑定读取 blob。

## <a name="input---example"></a>输入 - 示例

参阅语言特定的示例：

* [C#](#input---c-example)
* [C# 脚本 (.csx)](#input---c-script-example)
* [Java](#input---java-examples)
* [JavaScript](#input---javascript-example)
* [Python](#input---python-example)

### <a name="input---c-example"></a>输入 - C# 示例

以下示例演示使用一个队列触发器和一个 blob 输入绑定的 [C# 函数](functions-dotnet-class-library.md)。 队列消息包含该 blob 的名称，函数记录该 blob 的大小。

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

### <a name="input---c-script-example"></a>输入 - C# 脚本示例

<!--Same example for input and output. -->

以下示例演示 function.json 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [C# 脚本 (.csx)](functions-reference-csharp.md) 代码。 此函数创建文本 blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[配置](#input---configuration)部分解释了这些属性。

C# 脚本代码如下所示：

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

### <a name="input---javascript-example"></a>输入 - JavaScript 示例

<!--Same example for input and output. -->

下面的示例展示了 function.json 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [JavaScript 代码](functions-reference-node.md)。 该函数创建 Blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[配置](#input---configuration)部分解释了这些属性。

JavaScript 代码如下所示：

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

### <a name="input---python-example"></a>输入 - Python 示例

<!--Same example for input and output. -->

下面的示例展示了 *function.json* 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [Python 代码](functions-reference-python.md)。 该函数创建 Blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "$return",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

[配置](#input---configuration)部分解释了这些属性。

下面是 Python 代码：

```python
import logging
import azure.functions as func

def main(queuemsg: func.QueueMessage, inputblob: func.InputStream) -> func.InputStream:
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    return inputblob
```

### <a name="input---java-examples"></a>输入 - Java 示例

本部分包含以下示例：

* [HTTP 触发器，使用查询字符串查找 blob 名称](#http-trigger-look-up-blob-name-from-query-string-java)
* [队列触发器，接收来自队列消息的 blob 名称](#queue-trigger-receive-blob-name-from-queue-message-java)

#### <a name="http-trigger-look-up-blob-name-from-query-string-java"></a>HTTP 触发器，使用查询字符串查找 blob 名称 (Java)

 下面的示例显示了一个 Java 函数，该函数使用 ```HttpTrigger``` 注释来接收包含 blob 存储容器中某个文件的名称的一个参数。 然后，```BlobInput``` 注释读取该文件并将其作为 ```byte[]``` 传递给该函数。

```java
  @FunctionName("getBlobSizeHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage blobSize(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    final ExecutionContext context) {
      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-receive-blob-name-from-queue-message-java"></a>队列触发器，接收来自队列消息的 blob 名称 (Java)

 下面的示例显示了一个 Java 函数，该函数使用 ```QueueTrigger``` 注释来接收包含 blob 存储容器中某个文件的名称的一个消息。 然后，```BlobInput``` 注释读取该文件并将其作为 ```byte[]``` 传递给该函数。

```java
  @FunctionName("getBlobSize")
  @StorageAccount("Storage_Account_Connection_String")
  public void blobSize(
    @QueueTrigger(
      name = "filename", 
      queueName = "myqueue-items-sample") 
    String filename,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{queueTrigger}") 
    byte[] content,
    final ExecutionContext context) {
      context.getLogger().info("The size of \"" + filename + "\" is: " + content.length + " bytes");
  }
```

在 [Java 函数运行时库](/java/api/overview/azure/functions/runtime)中，对其值将来自 Blob 的参数使用 `@BlobInput` 注释。  可以将此注释与本机 Java 类型、POJO 或使用了 `Optional<T>` 的可为 null 的值一起使用。

## <a name="input---attributes"></a>输入 - 特性

在 [C# 类库](functions-dotnet-class-library.md)中，使用 [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs)。

该特性的构造函数采用 Blob 的路径，以及一个表示读取或写入的 `FileAccess` 参数，如以下示例中所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}

```

可以设置 `Connection` 属性来指定要使用的存储帐户，如以下示例中所示：

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read, Connection = "StorageConnectionAppSetting")] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

可以使用 `StorageAccount` 特性在类、方法或参数级别指定存储帐户。 有关详细信息，请参阅[触发器 - 特性](#trigger---attributes)。

## <a name="input---configuration"></a>输入 - 配置

下表解释了在 *function.json* 文件和 `Blob` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
|类型 | 不适用 | 必须设置为 `blob`。 |
|direction | 不适用 | 必须设置为 `in`。 [用法](#input---usage)部分中已阐述异常。 |
|**name** | 不适用 | 表示函数代码中的 Blob 的变量的名称。|
|**路径** |**BlobPath** | Blob 的路径。 |
|**连接** |**Connection**| 包含要用于此绑定的[存储连接字符串](../storage/common/storage-configure-connection-string.md#create-a-connection-string-for-an-azure-storage-account)的应用设置的名称。 如果应用设置名称以“AzureWebJobs”开始，则只能在此处指定该名称的余下部分。 例如，如果将 `connection` 设置为“MyStorage”，函数运行时将会查找名为“AzureWebJobsMyStorage”的应用设置。 如果将 `connection` 留空，函数运行时将使用名为 `AzureWebJobsStorage` 的应用设置中的默认存储连接字符串。<br><br>连接字符串必须属于某个常规用途存储帐户，而不能属于[仅限 Blob 的存储帐户](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|
|不适用 | **Access** | 表示是要读取还是写入。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>输入 - 用法

在 C# 和 C# 脚本中，可以为 blob 输入绑定使用以下参数类型：

* `Stream`
* `TextReader`
* `string`
* `Byte[]`
* `CloudBlobContainer`
* `CloudBlobDirectory`
* `ICloudBlob`<sup>1</sup>
* `CloudBlockBlob`<sup>1</sup>
* `CloudPageBlob`<sup>1</sup>
* `CloudAppendBlob`<sup>1</sup>

<sup>1</sup> function.json 中需有 "inout" 绑定 `direction` 或 C# 类库中需有 `FileAccess.ReadWrite`。

如果尝试绑定到某个存储 SDK 类型并收到错误消息，请确保已引用[正确的存储 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

由于整个 Blob 内容都会加载到内存中，因此，只有当 Blob 较小时才建议绑定到 `string` 或 `Byte[]`。 平时，最好使用 `Stream` 或 `CloudBlockBlob` 类型。 有关详细信息，请参阅本文前文中的[并发和内存使用情况](#trigger---concurrency-and-memory-usage)。

在 JavaScript 中，可以使用 `context.bindings.<name from function.json>` 访问 Blob 数据。

## <a name="output"></a>输出

使用 blob 存储输出绑定写入 blob。

## <a name="output---example"></a>输出 - 示例

参阅语言特定的示例：

* [C#](#output---c-example)
* [C# 脚本 (.csx)](#output---c-script-example)
* [Java](#output---java-examples)
* [JavaScript](#output---javascript-example)
* [Python](#output---python-example)

### <a name="output---c-example"></a>输出 - C# 示例

以下示例演示使用一个 blob 触发器和两个 blob 输出绑定的 [C# 函数](functions-dotnet-class-library.md)。 在 *sample-images* 容器中创建映像 Blob 时，会触发该函数。 该函数创建该映像 Blob 的小型和中型副本。

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
{
    var imageBuilder = ImageResizer.ImageBuilder.Current;
    var size = imageDimensionsTable[ImageSize.Small];

    imageBuilder.Build(image, imageSmall,
        new ResizeSettings(size.Item1, size.Item2, FitMode.Max, null), false);

    image.Position = 0;
    size = imageDimensionsTable[ImageSize.Medium];

    imageBuilder.Build(image, imageMedium,
        new ResizeSettings(size.Item1, size.Item2, FitMode.Max, null), false);
}

public enum ImageSize { ExtraSmall, Small, Medium }

private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
    { ImageSize.ExtraSmall, (320, 200) },
    { ImageSize.Small,      (640, 400) },
    { ImageSize.Medium,     (800, 600) }
};
```

### <a name="output---c-script-example"></a>输出 - C# 脚本示例

<!--Same example for input and output. -->

以下示例演示 function.json 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [C# 脚本 (.csx)](functions-reference-csharp.md) 代码。 此函数创建文本 blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[配置](#output---configuration)部分解释了这些属性。

C# 脚本代码如下所示：

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

### <a name="output---javascript-example"></a>输出 - JavaScript 示例

<!--Same example for input and output. -->

下面的示例展示了 function.json 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [JavaScript 代码](functions-reference-node.md)。 该函数创建 Blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

[配置](#output---configuration)部分解释了这些属性。

JavaScript 代码如下所示：

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

### <a name="output---python-example"></a>输出 - Python 示例

<!--Same example for input and output. -->

下面的示例展示了 *function.json* 文件中的 blob 输入和输出绑定，以及使用这些绑定的 [Python 代码](functions-reference-python.md)。 该函数创建 Blob 的副本。 该函数由包含要复制的 Blob 名称的队列消息触发。 新 Blob 名为 *{originalblobname}-Copy*。

在 *function.json* 文件中，`queueTrigger` 元数据属性用于指定 `path` 属性中的 Blob 名称：

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "outputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

[配置](#output---configuration)部分解释了这些属性。

下面是 Python 代码：

```python
import logging
import azure.functions as func

def main(queuemsg: func.QueueMessage, inputblob: func.InputStream,
         outputblob: func.Out[func.InputStream]):
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    outputblob.set(inputblob)
```

### <a name="output---java-examples"></a>输出 - Java 示例

本部分包含以下示例：

* [HTTP 触发器，使用 OutputBinding](#http-trigger-using-outputbinding-java)
* [队列触发器，使用函数返回值](#queue-trigger-using-function-return-value-java)

#### <a name="http-trigger-using-outputbinding-java"></a>HTTP 触发器，使用 OutputBinding (Java)

 下面的示例显示了一个 Java 函数，该函数使用 ```HttpTrigger``` 注释来接收包含 blob 存储容器中某个文件的名称的一个参数。 然后，```BlobInput``` 注释读取该文件并将其作为 ```byte[]``` 传递给该函数。 ```BlobOutput``` 注释绑定到 ```OutputBinding outputItem```，然后，函数使用后者来将输入 blob 的内容写入到所配置的存储容器中。

```java
  @FunctionName("copyBlobHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage copyBlobHttp(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    @BlobOutput(
      name = "target", 
      path = "myblob/{Query.file}-CopyViaHttp")
    OutputBinding<String> outputItem,
    final ExecutionContext context) {
      // Save blob to outputItem
      outputItem.setValue(new String(content, StandardCharsets.UTF_8));

      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-using-function-return-value-java"></a>队列触发器，使用函数返回值 (Java)

 下面的示例显示了一个 Java 函数，该函数使用 ```QueueTrigger``` 注释来接收包含 blob 存储容器中某个文件的名称的一个消息。 然后，```BlobInput``` 注释读取该文件并将其作为 ```byte[]``` 传递给该函数。 ```BlobOutput``` 注释绑定到函数返回值，然后，运行时使用后者来将输入 blob 的内容写入到所配置的存储容器中。

```java
  @FunctionName("copyBlobQueueTrigger")
  @StorageAccount("Storage_Account_Connection_String")
  @BlobOutput(
    name = "target", 
    path = "myblob/{queueTrigger}-Copy")
  public String copyBlobQueue(
    @QueueTrigger(
      name = "filename", 
      dataType = "string",
      queueName = "myqueue-items") 
    String filename,
    @BlobInput(
      name = "file", 
      path = "samples-workitems/{queueTrigger}") 
    String content,
    final ExecutionContext context) {
      context.getLogger().info("The content of \"" + filename + "\" is: " + content);
      return content;
  }
```

 在 [Java 函数运行时库](/java/api/overview/azure/functions/runtime)中，对其值将写入 Blob 存储中对象的函数参数使用 `@BlobOutput` 注释。  参数类型应为 `OutputBinding<T>`，其中 T 是任何本机 Java 类型或者是一个 POJO。

## <a name="output---attributes"></a>输出 - 特性

在 [C# 类库](functions-dotnet-class-library.md)中，使用 [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs)。

该特性的构造函数采用 Blob 的路径，以及一个表示读取或写入的 `FileAccess` 参数，如以下示例中所示：

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

可以设置 `Connection` 属性来指定要使用的存储帐户，如以下示例中所示：

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write, Connection = "StorageConnectionAppSetting")] Stream imageSmall)
{
    ...
}
```

有关完整示例，请参阅[输出 - C# 示例](#output---c-example)。

可以使用 `StorageAccount` 特性在类、方法或参数级别指定存储帐户。 有关详细信息，请参阅[触发器 - 特性](#trigger---attributes)。

## <a name="output---configuration"></a>输出 - 配置

下表解释了在 function.json 文件和 `Blob` 特性中设置的绑定配置属性。

|function.json 属性 | Attribute 属性 |说明|
|---------|---------|----------------------|
|类型 | 不适用 | 必须设置为 `blob`。 |
|direction | 不适用 | 对于输出绑定，必须设置为 `out`。 [用法](#output---usage)部分中已阐述异常。 |
|**name** | 不适用 | 表示函数代码中的 Blob 的变量的名称。  设置为 `$return` 可引用函数返回值。|
|**路径** |**BlobPath** | Blob 容器的路径。 |
|**连接** |**Connection**| 包含要用于此绑定的存储连接字符串的应用设置的名称。 如果应用设置名称以“AzureWebJobs”开始，则只能在此处指定该名称的余下部分。 例如，如果将 `connection` 设置为“MyStorage”，函数运行时将会查找名为“AzureWebJobsMyStorage”的应用设置。 如果将 `connection` 留空，函数运行时将使用名为 `AzureWebJobsStorage` 的应用设置中的默认存储连接字符串。<br><br>连接字符串必须属于某个常规用途存储帐户，而不能属于[仅限 Blob 的存储帐户](../storage/common/storage-account-overview.md#types-of-storage-accounts)。|
|不适用 | **Access** | 表示是要读取还是写入。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>输出 - 用法

在 C# 和 C# 脚本中，可绑定到以下类型，以编写 blob：

* `TextWriter`
* `out string`
* `out Byte[]`
* `CloudBlobStream`
* `Stream`
* `CloudBlobContainer`<sup>1</sup>
* `CloudBlobDirectory`
* `ICloudBlob`<sup>2</sup>
* `CloudBlockBlob`<sup>2</sup>
* `CloudPageBlob`<sup>2</sup>
* `CloudAppendBlob`<sup>2</sup>

<sup>1</sup> function.json 中需有 "in" 绑定 `direction` 或 C# 类库中需有 `FileAccess.Read`。 但是，可以使用运行时提供的容器对象来执行写入操作，例如将 Blob 上传到容器。

<sup>2</sup> function.json 中需有 "inout" 绑定 `direction` 或 C# 类库中需有 `FileAccess.ReadWrite`。

如果尝试绑定到某个存储 SDK 类型并收到错误消息，请确保已引用[正确的存储 SDK 版本](#azure-storage-sdk-version-in-functions-1x)。

在异步函数中，请使用返回值或 `IAsyncCollector` 而非 `out` 参数。

由于整个 Blob 内容都会加载到内存中，因此，只有当 Blob 较小时才建议绑定到 `string` 或 `Byte[]`。 平时，最好使用 `Stream` 或 `CloudBlockBlob` 类型。 有关详细信息，请参阅本文前文中的[并发和内存使用情况](#trigger---concurrency-and-memory-usage)。


在 JavaScript 中，可以使用 `context.bindings.<name from function.json>` 访问 Blob 数据。

## <a name="exceptions-and-return-codes"></a>异常和返回代码

| 绑定 |  参考 |
|---|---|
| Blob | [Blob 错误代码](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Blob、表、队列 |  [存储错误代码](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Blob、表、队列 |  [故障排除](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>后续步骤

* [详细了解 Azure Functions 触发器和绑定](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Blob storage trigger](functions-create-storage-blob-triggered-function.md)
--->
