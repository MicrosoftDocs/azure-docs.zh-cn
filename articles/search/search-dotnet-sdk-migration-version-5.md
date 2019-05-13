---
title: 升级到 Azure 搜索 .NET SDK 版本 5 - Azure 搜索
description: 从旧版本将代码迁移到 Azure 搜索 .NET SDK 版本 5。 了解新增功能和所需的代码更改。
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.custom: seodec2018
ms.openlocfilehash: 8382884b4ce2965dee4acf191f82eb012b670713
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147476"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-5"></a>升级到 Azure 搜索 .NET SDK 版本 5

如果使用的是版本 4.0-preview 或更早版本的 [Azure 搜索 .NET SDK](https://aka.ms/search-sdk)，本文有助于升级应用程序，以便使用版本 5。

有关包括示例的 SDK 的更多常规演练，请参阅[如何使用 .NET 应用程序中的 Azure 搜索](search-howto-dotnet-sdk.md)。

Azure 搜索 .NET SDK 的版本 5 包含了某些针对早期版本进行的更改。 这些更改主要涉及次要版本，因此更改代码的工作量并不是太大。 有关如何更改代码以使用新版 SDK 的说明，请参阅[升级步骤](#UpgradeSteps)。

> [!NOTE]
> 如果使用的是版本 2.0-preview 或更早版本，则应先升级到版本 3，再升级到版本 5。 有关说明，请参阅[升级到 Azure 搜索 .NET SDK 版本 3](search-dotnet-sdk-migration.md)。
>
> Azure 搜索服务实例支持多个 REST API 版本，包括最新的版本。 可以不使用最新版本，但是我们建议迁移代码，以便使用最新版本。 使用 REST API 时，必须在每个请求中通过 api-version 参数指定 API 版本。 使用 .NET SDK 时，使用的 SDK 版本确定对应的 REST API 版本。 如果使用较旧的 SDK，即使已升级服务以支持较新的 API 版本，也可以继续运行代码，而不进行任何更改。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-5"></a>版本 5 中的新增功能
Azure 搜索 .NET SDK 的版本 5 针对 Azure 搜索 REST API 的最新正式发布版本，具体来说就是 2017-11-11。 这使得可在 .NET 应用程序中使用 Azure 搜索的新功能，如下所示：

* [同义词](search-synonyms.md)
* 现可以编程方式访问索引器执行历史记录中的警告（有关更多详细信息，请参阅 [.NET 引用](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexerexecutionresult?view=azure-dotnet)中 `IndexerExecutionResult` 的 `Warning` 属性）。
* 支持 .NET Core 2。
* 新的包结构支持使用仅需要的 SDK 的部件（有关详细信息，请参阅[版本 5 中的重大更改](#ListOfChanges)）。

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升级步骤
首先，按以下方法操作更新 `Microsoft.Azure.Search` 的 NuGet 引用：使用 NuGet 包管理器控制台，或者在 Visual Studio 中右键单击项目引用，然后选择“管理 NuGet 程序包...”。

在 NuGet 已下载新的程序包及其依赖项后，请重新生成项目。 项目重新生成可能会成功，具体取决于代码的结构。 如果成功，一切准备就绪！

如果生成失败，应该会看到如下所示的生成错误：

    The name 'SuggesterSearchMode' does not exist in the current context

下一步是修复生成错误。 有关出错原因和修复方法的详细信息，请参阅[版本 5 中的重大更改](#ListOfChanges)。

请注意，由于更改了 Azure 搜索 .NET SDK 的包，必须重新生成应用程序才能使用版本 5。 有关这些更改的详细信息，请参阅[版本 5 中的重大更改](#ListOfChanges)。

可能会看到与已过时方法或属性有关的其他生成警告。 这些警告将包含有关使用哪些功能来替换已弃用功能的说明。 例如，如果应用程序使用 `IndexingParametersExtensions.DoNotFailOnUnsupportedContentType` 方法，则出现一个警告“此行为现在默认启用，因此不再需要调用此方法。”

在修复了任何生成错误或警告后，可以对应用程序进行更改，以利用新功能（如果愿意）。 有关 SDK 中的新功能的详细信息，请参阅[版本 5 中的新增功能](#WhatsNew)。

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-5"></a>版本 5 中的重大更改

### <a name="new-package-structure"></a>新的包结构

版本 5 中最本质的重大更改是 `Microsoft.Azure.Search` 程序集及其内容已被分成四个独立的程序集，现在以四个独立的 NuGet 包进行分发：

 - `Microsoft.Azure.Search`：这是一个元数据包，其中包含所有其他 Azure 搜索包作为依赖项。 如果从 SDK 的早期版本进行升级，只需升级此包，重新生成就可开始使用新版本。
 - `Microsoft.Azure.Search.Data`：如果使用 Azure 搜索开发 .NET 应用程序，则使用此包，并且只需查询或更新索引中的文档。 如果还需要创建或更新索引、同义词映射或其他服务级资源，请改用 `Microsoft.Azure.Search` 包。
 - `Microsoft.Azure.Search.Service`：如果在 .NET 中开发自动化以管理 Azure 搜索索引、同义词映射、索引器、数据源或其他服务级资源，请使用此包。 如果只需要查询或更新索引中的文档，请改用 `Microsoft.Azure.Search.Data` 包。 如果需要 Azure 搜索的所有功能，请改用 `Microsoft.Azure.Search` 包。
 - `Microsoft.Azure.Search.Common`：Azure 搜索 .NET 库需要的常见类型。 不应在应用程序中直接使用此包；它只是用作依赖项。
 
由于许多类型在程序集之间移动，所以从技术上来说这是重大更改。 这就是为什么需要重新生成应用程序才能升级到 SDK 的版本 5。

除重新生成应用程序之外，版本 5 中只有少量可能需要更改代码的其他重大更改。

### <a name="change-to-suggesters"></a>对建议器的更改 

`Suggester` 构造函数不再为 `SuggesterSearchMode` 提供 `enum` 参数。 此枚举只有一个值，因此是多余的。 如果因此看到生成错误，只需删除对 `SuggesterSearchMode` 参数的引用即可。

### <a name="removed-obsolete-members"></a>删除了过时成员

可能会看到与更早版本中标记为已过时的方法或属性相关的生成错误，这些方法或属性随后已在版本 5 中删除。 如果遇到此类错误，可按下面所述解决它们：

- 如果使用 `IndexingParametersExtensions.IndexStorageMetadataOnly` 方法，请改用 `SetBlobExtractionMode(BlobExtractionMode.StorageMetadata)`。
- 如果使用 `IndexingParametersExtensions.SkipContent` 方法，请改用 `SetBlobExtractionMode(BlobExtractionMode.AllMetadata)`。

### <a name="removed-preview-features"></a>删除了预览功能

如果要从版本 4.0-preview 升级到版本 5，请注意，JSON 数组和 CSV 对 Blob 索引器的分析支持已删除，因为这些功能仍处于预览状态。 具体而言，`IndexingParametersExtensions` 类的以下方法已删除：

- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

如果应用程序硬依赖于这些功能，则将无法升级到 Azure 搜索 .NET SDK 的版本 5。 可继续使用版本 4.0-preview。 但是，请记住，**我们不建议在生产应用程序中使用预览版 SDK**。 预览功能仅用于评估，并且可能会更改。

## <a name="conclusion"></a>结束语
如果需要有关如何使用 Azure 搜索 .NET SDK 的更多详细信息，请参阅 [.NET 操作指南](search-howto-dotnet-sdk.md)。

我们欢迎你对 SDK 提供反馈。 如果遇到问题，欢迎向我们寻求帮助上[Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search)。 如果找到 Bug，可以在 [Azure .NET SDK GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/issues)中提出问题。 务必在问题标题上加前缀“[Azure 搜索]”。

感谢使用 Azure 搜索！
