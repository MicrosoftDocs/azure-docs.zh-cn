---
title: 用于非英语搜索查询的多语言索引 - Azure 搜索
description: Azure 搜索支持 56 种语言，通过 Microsoft 中的 Lucene 和自然语言处理技术利用语言分析器。
author: yahnoosh
manager: jlembicz
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: jlembicz
ms.custom: seodec2018
ms.openlocfilehash: cc9f271c1c79f34ba62fa22d6ce4fd6bf16738f1
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025270"
---
# <a name="create-an-index-for-documents-in-multiple-languages-in-azure-search"></a>在 Azure 搜索中为多语言文档创建索引
> [!div class="op_single_selector"]
>
> * [门户](search-language-support.md)
> * [REST](https://msdn.microsoft.com/library/azure/dn879793.aspx)
> * [.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.search.models.analyzername.aspx)
>
>

发掘语言分析器的强大功能，就如同在索引定义中的可搜索字段上设置属性一样简单。 现在，可以在门户中执行此步骤。

下面是 Azure 搜索服务的 Azure 门户边栏选项卡的屏幕截图，供用户定义索引架构。 通过此边栏选项卡，用户可以创建所有字段并为它们创建分析器属性。

> [!IMPORTANT]
> 如同从头创建新索引时，或将新字段添加到现有索引时，只能在字段定义期间设置语言分析器。 确保在创建字段时完全指定所有属性，其中包括分析器。 保存更改后，将无法编辑属性或更改分析器类型。
>
>

## <a name="define-a-new-field-definition"></a>定义新的字段定义
1. 登录 [Azure 门户](https://portal.azure.com)，并打开搜索服务的服务边栏选项卡。
2. 在服务仪表板顶部的命令栏中单击“添加索引”即可启动新的索引，或打开现有索引，在添加至现有索引的新字段上设置分析器。
3. “字段”边栏选项卡随即出现，显示可供定义索引架构的选项，包括用于选择语言分析器的“分析器”选项卡。
4. 在“字段”中，通过提供名称、选择数据类型并设置属性进行字段定义，进而将字段标记为可全文搜索、可在搜索结果中检索、可用于分面导航结构以及可排序等。
5. 在移动到下一个字段前，打开“分析器”选项卡。

![][1]
*若要选择分析器，请单击“字段”边栏选项卡上的“分析器”选项卡*

## <a name="choose-an-analyzer"></a>选择分析器
1. 滚动以找到定义的字段。
2. 如果尚未将字段标记为可搜索，请立即单击复选框以将其标记为“可搜索”。
3. 单击“分析器”区域以显示可用的分析器列表。
4. 选择要使用的分析器。

![][2]
*为每个字段选择一个支持的分析器*

默认情况下，所有可搜索字段会使用[标准 Lucene 分析器](https://lucene.apache.org/core/4_10_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html)（不限语言）。 若要查看支持的分析器的完整列表，请参阅 [Azure 搜索中的语言支持](https://msdn.microsoft.com/library/azure/dn879793.aspx)。

为字段选择语言分析器后，它用于该字段的每个索引和搜索请求。 当使用不同的分析器针对多个字段发出查询时，查询由每个字段相应的分析器独立处理。

许多 Web 和移动应用程序使用不同的语言来为世界各地的用户提供服务。 可通过为每种支持的语言创建字段来为此类方案定义索引。

![][3]
*每种支持的语言都有描述字段的索引定义*

如果已知发出查询的代理的语言，可使用 **searchFields** 查询参数，将搜索请求的范围限制为特定字段。 以下查询将仅针对波兰文描述发出：

`https://[service name].search.windows.net/indexes/[index name]/docs?search=darmowy&searchFields=description_pl&api-version=2019-05-06`

使用“搜索资源管理器”粘贴类似上述内容的查询，可以从门户查询索引。 从服务边栏选项卡的命令栏可以获取搜索资源管理器。 有关详细信息，请参阅[在门户中查询 Azure 搜索索引](search-explorer.md)。

有时，发出查询的代理的语言未知，在此情况下，可以针对所有字段同时发出查询。 如果需要，可以使用[计分配置文件](https://msdn.microsoft.com/library/azure/dn798928.aspx)来定义采用特定语言的结果首选项。 在下面的示例中，与波兰文和法文的匹配项相比，英文描述中提供的匹配项的评分更高：

    "scoringProfiles": [
      {
        "name": "englishFirst",
        "text": {
          "weights": { "description_en": 2 }
        }
      }
    ]

`https://[service name].search.windows.net/indexes/[index name]/docs?search=Microsoft&scoringProfile=englishFirst&api-version=2019-05-06`

如果是一名 .NET 开发人员，请注意，可以使用 [Azure 搜索 .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Search) 来设置语言分析器。 最新版本还提供对 Microsoft 语言分析器的支持。

<!-- Image References -->
[1]: ./media/search-language-support/AnalyzerTab.png
[2]: ./media/search-language-support/SelectAnalyzer.png
[3]: ./media/search-language-support/IndexDefinition.png
