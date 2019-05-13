---
title: 升级到 Azure 搜索.NET SDK 版本 9-Azure 搜索
description: 将代码从较旧版本迁移到 Azure 搜索.NET SDK 版本 9。 了解新增功能和所需的代码更改。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: f540bc304920073bcd823adcf6c9dd47cb2cf93b
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159745"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-9"></a>升级到 Azure 搜索.NET SDK 版本 9

如果你在使用版本 7.0-preview 或更早版本的[Azure 搜索.NET SDK](https://aka.ms/search-sdk)，本文有助于升级应用程序以使用版本 9。

> [!NOTE]
> 如果你想要使用版本 8.0-preview 以评估尚不是已正式发布的功能，也可以按照这篇文章中的说明从以前的版本升级到 8.0-preview。

有关包括示例的 SDK 的更多常规演练，请参阅[如何使用 .NET 应用程序中的 Azure 搜索](search-howto-dotnet-sdk.md)。

Azure 搜索.NET SDK 的版本 9 包含早期版本的许多更改。 其中一些重大更改，但它们应仅需要相对较小更改代码。 有关如何更改代码以使用新版 SDK 的说明，请参阅[升级步骤](#UpgradeSteps)。

> [!NOTE]
> 如果正在使用版本 4.0-preview 或更低版本，应首先，升级到版本 5，然后升级到版本 9。 请参阅[升级到 Azure 搜索.NET SDK 版本 5](search-dotnet-sdk-migration-version-5.md)有关的说明。
>
> Azure 搜索服务实例支持多个 REST API 版本，包括最新的版本。 可以不使用最新版本，但是我们建议迁移代码，以便使用最新版本。 使用 REST API 时，必须在每个请求中通过 api-version 参数指定 API 版本。 使用 .NET SDK 时，使用的 SDK 版本确定对应的 REST API 版本。 如果使用较旧的 SDK，即使已升级服务以支持较新的 API 版本，也可以继续运行代码，而不进行任何更改。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-9"></a>什么是版本 9 中的新增功能
针对 Azure 搜索.NET SDK 版本 9 的最新的 Azure 搜索 REST API，专门 2019年-05-06 正式发布版本。 这使得可在 .NET 应用程序中使用 Azure 搜索的新功能，如下所示：

* [认知搜索](cognitive-search-concept-intro.md)是 Azure 搜索中，用于从映像、 blob 和其他非结构化的数据源的丰富内容，使其更可搜索的 Azure 搜索索引中提取文本中的 AI 功能。
* 为支持[复杂类型](search-howto-complex-data-types.md)，可在 Azure 搜索索引中几乎所有嵌套的 JSON 结构的模型。
* [记忆式键入功能](search-autocomplete-tutorial.md)提供了一种替代方法**建议**API 来实现搜索---键入时的行为。 自动完成"完成"的单词或短语当前键入用户。
* [分析模式 JsonLines](search-howto-index-json-blobs.md)、 部分的 Azure Blob 编制索引，创建一个搜索文档，每个由换行符分隔的 JSON 实体。

### <a name="new-preview-features-in-version-80-preview"></a>版本 8.0-preview 中的新预览功能
版本 8.0-preview Azure 搜索.NET SDK 的目标 API 版本 2017年-11-11-Preview。 此版本包括所有相同功能的版本 9，加上：

* [客户托管的加密密钥](search-security-manage-encryption-keys.md)服务端加密在 rest 为的新预览功能。 除了内置的静态加密由 Microsoft 管理，您可以应用一层额外的加密密钥的唯一所有者所在。

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升级步骤
首先，按以下方法操作更新 `Microsoft.Azure.Search` 的 NuGet 引用：使用 NuGet 包管理器控制台，或者在 Visual Studio 中右键单击项目引用，然后选择“管理 NuGet 程序包...”。

在 NuGet 已下载新的程序包及其依赖项后，请重新生成项目。 项目重新生成可能会成功，具体取决于代码的结构。 如果成功，一切准备就绪！

如果生成失败，您需要修复每个生成错误。 请参阅[版本 9 中的重大更改](#ListOfChanges)详细介绍了如何解决每个可能生成错误。

可能会看到与已过时方法或属性有关的其他生成警告。 这些警告将包含有关使用哪些功能来替换已弃用功能的说明。 例如，如果你的应用程序使用`DataSourceType.DocumentDb`属性，应获取一个警告，指出"已弃用此成员。 改为使用 CosmosDb"。

在修复了任何生成错误或警告后，可以对应用程序进行更改，以利用新功能（如果愿意）。 中详细介绍了 SDK 中的新增功能[什么是版本 9 中的新增功能](#WhatsNew)。

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-9"></a>版本 9 中的重大更改

可能需要进行代码更改除重新生成你的应用程序之外的版本 9 中有几个重大更改。

> [!NOTE]
> 更改下面的列表并不详尽。 某些更改可能不会导致生成错误，但从技术上讲重大，因为它们中断依赖于早期版本的 Azure 搜索.NET SDK 程序集的程序集的二进制文件兼容性。 下面未列出了此类更改。 请升级到版本 9 以避免任何二进制文件兼容性问题时，重新生成应用程序。

### <a name="making-properties-immutable"></a>让不可变属性

多个模型类的公共属性现在是不可变的。 如果需要创建用于测试这些类的自定义实例，可以使用新的参数化构造函数：

  - `AutocompleteItem`
  - `DocumentSearchResult`
  - `DocumentSuggestResult`
  - `FacetResult`
  - `SearchResult`
  - `SuggestResult`

### <a name="changes-to-field"></a>对字段的更改

`Field`类已更改，现在它还可以表示复杂的字段。

以下`bool`属性现在是可以为 null:

  - `IsFilterable`
  - `IsFacetable`
  - `IsSearchable`
  - `IsSortable`
  - `IsRetrievable`
  - `IsKey`

这是因为这些属性现在必须`null`对于复杂的字段。 如果你有读取这些属性的代码，它必须是准备好处理`null`。 请注意，所有其他属性的`Field`一直并继续可为空，其中一些也将`null`在复杂域--尤其是以下情况下：

  - `Analyzer`
  - `SearchAnalyzer`
  - `IndexAnalyzer`
  - `SynonymMaps`

无参数构造函数`Field`变得`internal`。 从现在起，每个`Field`构造时需要显式名称和数据类型。

### <a name="simplification-of-batch-and-results-types"></a>批处理和结果类型的简化

在版本 7.0-preview 及更早版本中，封装的文档的组的各种类已组织成多个并行类层次结构：

  -  `DocumentSearchResult` 和`DocumentSearchResult<T>`继承自 `DocumentSearchResultBase`
  -  `DocumentSuggestResult` 和`DocumentSuggestResult<T>`继承自 `DocumentSuggestResultBase`
  -  `IndexAction` 和`IndexAction<T>`继承自 `IndexActionBase`
  -  `IndexBatch` 和`IndexBatch<T>`继承自 `IndexBatchBase`
  -  `SearchResult` 和`SearchResult<T>`继承自 `SearchResultBase`
  -  `SuggestResult` 和`SuggestResult<T>`继承自 `SuggestResultBase`

如果不使用泛型类型参数派生的类型是用来在"动态类型化"方案中使用，假定的使用情况`Document`类型。

从版本 8.0 preview 开始，基类和非泛型派生的类所有已删除。 动态类型化的情况下，您可以使用`IndexBatch<Document>`， `DocumentSearchResult<Document>`，依次类推。
 
### <a name="removal-of-extensibleenum"></a>删除的 ExtensibleEnum

`ExtensibleEnum`基类已删除。 从其派生的所有类都现结构，如`AnalyzerName`， `DataType`，和`DataSourceType`为例。 其`Create`方法也已被删除。 您可以删除对调用`Create`由于这些类型都可以从字符串隐式转换。 如果那样会导致编译器错误，你可以显式调用通过强制转换的转换运算符来区分类型。 例如，可以更改如下代码：

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", AnalyzerName.Create("my_email_analyzer")) { IsSearchable = true }
    },
    ...
}
```

更改为：

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", (AnalyzerName)"my_email_analyzer") { IsSearchable = true }
    },
    ...
}
```

保存这些类型的可选值的属性是现在显式的类型可以为 null 使它们可以继续是可选的。

### <a name="removal-of-facetresults-and-hithighlights"></a>删除 FacetResults 和 HitHighlights

`FacetResults`和`HitHighlights`类均已删除。 分面结果现在被类型化为`IDictionary<string, IList<FacetResult>>`并命中突出显示为`IDictionary<string, IList<string>>`。 若要解决生成错误引入此更改的快速方法是添加`using`顶部的每个文件的使用已删除的类型的别名。 例如：

```csharp
using FacetResults = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<Models.FacetResult>>;
using HitHighlights = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<string>>;
```

### <a name="change-to-synonymmap"></a>将更改为 SynonymMap 

`SynonymMap` 构造函数不再为 `SynonymMapFormat` 提供 `enum` 参数。 此枚举只有一个值，因此是多余的。 如果因此看到生成错误，只需删除对 `SynonymMapFormat` 参数的引用即可。

### <a name="miscellaneous-model-class-changes"></a>杂项模型类更改

`AutocompleteMode`属性的`AutocompleteParameters`不再是可以为 null。 如果必须将分配到此属性的代码`null`、 只需删除和属性将自动初始化为默认值。

为参数的顺序`IndexAction`构造函数已更改，现在，此构造函数是自动生成。 而不是使用构造函数，我们建议使用工厂方法`IndexAction.Upload`， `IndexAction.Merge`，依次类推。

### <a name="removed-preview-features"></a>删除了预览功能

如果从版本 8.0-preview 升级到版本为 9 版，请注意已删除使用客户托管密钥加密，因为此功能仍处于预览状态。 具体而言，`EncryptionKey`的属性`Index`和`SynonymMap`已被删除。

如果你的应用程序具有硬依赖项对此功能，您将不能升级到的 Azure 搜索.NET SDK 版本 9。 您可以继续使用版本 8.0-preview。 但是，请记住，**我们不建议在生产应用程序中使用预览版 SDK**。 预览功能仅用于评估，并且可能会更改。

> [!NOTE]
> 如果您创建加密索引或同义词映射使用的 sdk 版本 8.0-preview，仍可以使用它们，然后修改使用的 SDK 版本 9，而不会产生负面影响及其加密状态及其定义。 SDK 版本 9 不会发送`encryptionKey`属性对 REST API 和 REST API，因此不会更改该资源的加密状态。 

### <a name="behavioral-change-in-data-retrieval"></a>在数据检索中的行为更改

如果您使用的"动态类型化" `Search`， `Suggest`，或`Get`返回的类型实例的 Api `Document`，请注意，它们现在反序列化到空 JSON 数组`object[]`而不是`string[]`。

## <a name="conclusion"></a>结束语
如果需要有关如何使用 Azure 搜索 .NET SDK 的更多详细信息，请参阅 [.NET 操作指南](search-howto-dotnet-sdk.md)。

我们欢迎你对 SDK 提供反馈。 如果遇到问题，欢迎向我们寻求帮助上[Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search)。 如果找到 Bug，可以在 [Azure .NET SDK GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/issues)中提出问题。 务必在问题标题上加前缀“[Azure 搜索]”。

感谢使用 Azure 搜索！
