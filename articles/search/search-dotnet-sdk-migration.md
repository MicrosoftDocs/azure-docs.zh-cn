---
title: 升级到 Azure 搜索 .NET SDK 版本 3 - Azure 搜索
description: 从旧版本将代码迁移到 Azure 搜索 .NET SDK 版本 3。 了解新增功能和所需的代码更改。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: d41c2b541bf80448d180a1d081c255e5bf754e5e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147334"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-3"></a>升级到 Azure 搜索 .NET SDK 版本 3

<!--- DETAILS in the word doc
cosmosdb
NER v1 skill 
Indexer execution result errors no longer have status
the data source API will no longer return in the response of any REST operation, the connection string specified by the user.
--->

如果使用的是版本 2.0-preview 或更早版本的 [Azure 搜索 .NET SDK](https://aka.ms/search-sdk)，本文有助于升级应用程序，以便使用版本 3。

有关包括示例的 SDK 的更多常规演练，请参阅[如何使用 .NET 应用程序中的 Azure 搜索](search-howto-dotnet-sdk.md)。

版本 3 的 Azure 搜索 .NET SDK 包含了某些针对早期版本进行的更改。 这些更改主要涉及次要版本，因此更改代码的工作量并不是太大。 有关如何更改代码以使用新版 SDK 的说明，请参阅[升级步骤](#UpgradeSteps)。

> [!NOTE]
> 如果使用的是版本 1.0.2-preview 或更早版本，则应该先升级到版本 1.1，再升级到版本 3。 有关说明，请参阅[升级到 Azure 搜索 .NET SDK 版本 1.1](search-dotnet-sdk-migration-version-1.md)。
>
> Azure 搜索服务实例支持多个 REST API 版本，包括最新的版本。 可以不使用最新版本，但是我们建议迁移代码，以便使用最新版本。 使用 REST API 时，必须在每个请求中通过 api-version 参数指定 API 版本。 使用 .NET SDK 时，使用的 SDK 版本确定对应的 REST API 版本。 如果使用较旧的 SDK，即使已升级服务以支持较新的 API 版本，也可以继续运行代码，而不进行任何更改。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-3"></a>版本 3 中的新增功能
版本 3 的 Azure 搜索 .NET SDK 针对 Azure 搜索 REST API 的最新正式发布版本，具体来说就是 2016-09-01。 这使得可以在 .NET 应用程序中使用 Azure 搜索的许多新功能，如下所示：

* [自定义分析器](https://aka.ms/customanalyzers)
* [Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)和 [Azure 表存储](search-howto-indexing-azure-tables.md)索引器支持
* 通过[字段映射](search-indexer-field-mappings.md)实现的索引器自定义
* 用于支持安全并发更新索引定义、索引器和数据源的 ETag 支持
* 支持通过修饰模型类并使用新的 `FieldBuilder` 类，以声明方式构建索引字段定义。
* 对 .NET Core 和 .NET Portable Profile 111 的支持

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升级步骤
首先，按以下方法操作更新 `Microsoft.Azure.Search` 的 NuGet 引用：使用 NuGet 包管理器控制台，或者在 Visual Studio 中右键单击项目引用，然后选择“管理 NuGet 程序包...”。

在 NuGet 已下载新的程序包及其依赖项后，请重新生成项目。 项目重新生成可能会成功，具体取决于代码的结构。 如果成功，一切准备就绪！

如果生成失败，应该会看到如下所示的生成错误：

    Program.cs(31,45,31,86): error CS0266: Cannot implicitly convert type 'Microsoft.Azure.Search.ISearchIndexClient' to 'Microsoft.Azure.Search.SearchIndexClient'. An explicit conversion exists (are you missing a cast?)

下一步是修复生成错误。 有关出错原因和修复方法的详细信息，请参阅[版本 3 中的重大更改](#ListOfChanges)。

可能会看到与已过时方法或属性有关的其他生成警告。 这些警告将包含有关使用哪些功能来替换已弃用功能的说明。 例如，如果应用程序使用了 `IndexingParameters.Base64EncodeKeys` 属性，应该会收到显示 `"This property is obsolete. Please create a field mapping using 'FieldMapping.Base64Encode' instead."` 的警告

在修复了任何生成错误后，可以对应用程序进行更改，以利用新功能（如果愿意）。 有关 SDK 中的新功能的详细信息，请参阅[版本 3 中的新增功能](#WhatsNew)。

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-3"></a>版本 3 中的重大更改
除重新生成应用程序之外，版本 3 中只有少量可能需要更改代码的重大更改。

### <a name="indexesgetclient-return-type"></a>Indexes.GetClient 返回类型
`Indexes.GetClient` 方法具有新的返回类型。 以前，它返回 `SearchIndexClient`，但在版本 2.0-preview 中已将此项更改为 `ISearchIndexClient`，而该更改将传递给版本 3。 这是为了支持想要通过返回 `ISearchIndexClient` 的模拟实现，来模拟测试单元的 `GetClient` 方法的客户。

#### <a name="example"></a>示例
如果代码如下所示：

```csharp
SearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

若要修复任何生成错误，可以更改为如下所示的代码：

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

### <a name="analyzername-datatype-and-others-are-no-longer-implicitly-convertible-to-strings"></a>AnalyzerName、DataType 等不再隐式转换为字符串
Azure 搜索 .NET SDK 中有许多派生自 `ExtensibleEnum` 的类型。 以前，这些类型都隐式转换为 `string` 类型。 但是，在这些类的 `Object.Equals` 实现中发现了 bug，修复 bug 需要禁止此隐式转换。 仍允许显式转换为 `string`。

#### <a name="example"></a>示例
如果代码如下所示：

```csharp
var customTokenizerName = TokenizerName.Create("my_tokenizer"); 
var customTokenFilterName = TokenFilterName.Create("my_tokenfilter"); 
var customCharFilterName = CharFilterName.Create("my_charfilter"); 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        customTokenizerName,  
        new[] { customTokenFilterName },  
        new[] { customCharFilterName }), 
}; 
```

若要修复任何生成错误，可以更改为如下所示的代码：

```csharp
const string CustomTokenizerName = "my_tokenizer"; 
const string CustomTokenFilterName = "my_tokenfilter"; 
const string CustomCharFilterName = "my_charfilter"; 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        CustomTokenizerName,  
        new TokenFilterName[] { CustomTokenFilterName },  
        new CharFilterName[] { CustomCharFilterName })
}; 
```

### <a name="removed-obsolete-members"></a>删除了过时成员

可能会看到与版本 2.0-preview 中标记为已过时的方法或属性相关的生成错误，这些方法或属性随后已在版本 3 中删除。 如果遇到此类错误，可按下面所述解决它们：

- 如果使用此构造函数：`ScoringParameter(string name, string value)`，请改为使用这个：`ScoringParameter(string name, IEnumerable<string> values)`
- 如果使用 `ScoringParameter.Value` 属性，请改为使用 `ScoringParameter.Values` 属性或 `ToString` 方法。
- 如果使用 `SearchRequestOptions.RequestId` 属性，请改为使用 `ClientRequestId` 属性。

### <a name="removed-preview-features"></a>删除了预览功能

如果要从版本 2.0-preview 升级到版本 3，请注意，JSON 和 CSV 对 Blob 索引器的分析支持已删除，因为这些功能仍处于预览状态。 具体而言，`IndexingParametersExtensions` 类的以下方法已删除：

- `ParseJson`
- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

如果应用程序硬依赖于这些功能，则将不能升级到版本 3 的 Azure 搜索 .NET SDK。 可以继续使用版本 2.0-preview。 但是，请记住，**我们不建议在生产应用程序中使用预览版 SDK**。 预览功能仅用于评估，并且可能会更改。

## <a name="conclusion"></a>结束语
如果需要有关如何使用 Azure 搜索 .NET SDK 的更多详细信息，请参阅 [.NET 操作指南](search-howto-dotnet-sdk.md)。

我们欢迎你对 SDK 提供反馈。 如果遇到问题，欢迎向我们寻求帮助上[Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search)。 如果找到 Bug，可以在 [Azure .NET SDK GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/issues)中提出问题。 务必在问题标题上加前缀“[Azure 搜索]”。

感谢使用 Azure 搜索！
