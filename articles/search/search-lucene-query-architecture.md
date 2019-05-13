---
title: 全文搜索引擎 (Lucene) 体系结构 - Azure 搜索
description: 解释与 Azure 搜索相关的全文搜索的 Lucene 查询处理和文档检索概念。
manager: jlembicz
author: yahnoosh
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: jlembicz
ms.custom: seodec2018
ms.openlocfilehash: cb8e42400847ba9fb13568d05dffac86d9cec85f
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024625"
---
# <a name="how-full-text-search-works-in-azure-search"></a>Azure 搜索中全文搜索的工作原理

本文面向需要更深入了解 Azure 搜索中 Lucene 全文搜索工作原理的开发人员。 对于文本查询，在大多数情况下，Azure 搜索都会顺利地提供预期结果，但偶尔也会收到看上去“不靠谱”的结果。 在这种情况下，如果对 Lucene 查询执行的四个阶段（查询分析、词法分析、文档匹配和评分）有一定的背景知识，则有助于确定要对提供所需结果的查询参数或索引配置进行哪些特定的更改。 

> [!Note] 
> Azure 搜索使用 Lucene 进行全文搜索，但 Lucene 集成并不彻底。 我们将有选择地公开和扩展 Lucene 功能，使情景对于 Azure 搜索变得重要。 

## <a name="architecture-overview-and-diagram"></a>体系结构概述和关系图

处理全文搜索查询时，首先会分析查询文本以提取搜索词。 搜索引擎使用索引来检索包含匹配词的文档。 单个查询词有时会分解并重新构造成新的形式，以便从更大的角度来确定可将哪些内容视为潜在的匹配项。 然后，根据分配给每个匹配文档的相关性评分将结果集排序。 排名列表中靠前的匹配项将返回给调用方应用程序。

再次指出，查询执行包括四个阶段： 

1. 查询分析 
2. 词法分析 
3. 文档检索 
4. 评分 

以下关系图演示了用于处理搜索请求的组件。 

 ![Azure 搜索中 Lucene 查询体系结构的关系图][1]


| 关键组件 | 功能说明 | 
|----------------|------------------------|
|**查询分析器** | 将查询词与查询运算符区分开来，并创建要发送到搜索引擎的查询结构（查询树）。 |
|**分析器** | 针对查询词执行词法分析。 此过程可能涉及到查询词的转换、删除或扩展。 |
|**索引** | 一个有效的数据结构，用于存储和组织从索引文档中提取的可搜索词。 |
|**搜索引擎** | 根据倒排索引的内容检索文档并为其评分。 |

## <a name="anatomy-of-a-search-request"></a>搜索请求的剖析

搜索请求是一个完整的规范，描述了应在结果集中返回哪些内容。 最简单的搜索请求形式是不包含任何类型的条件的空查询。 比较现实的搜索请求示例包含参数、多个查询词，其范围可能限定为某些字段，另外，可能还包含筛选表达式和排序规则。  

以下示例是可以使用 [REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 发送到 Azure 搜索的一个搜索请求。  

~~~~
POST /indexes/hotels/docs/search?api-version=2019-05-06
{
    "search": "Spacious, air-condition* +\"Ocean view\"",
    "searchFields": "description, title",
    "searchMode": "any",
    "filter": "price ge 60 and price lt 300",
    "orderby": "geo.distance(location, geography'POINT(-159.476235 22.227659)')", 
    "queryType": "full" 
}
~~~~

对于此请求，搜索引擎将执行以下操作：

1. 筛选出其中的价格至少为 $60 且小于 $300 的文档。
2. 执行查询。 在此示例中，搜索查询包括短语和字词：`"Spacious, air-condition* +\"Ocean view\""`（用户通常不会输入标点，但此示例中包含标点，目的是为了方便解释分析器如何处理标点）。 对于此查询，搜索引擎将在 `searchFields` 指定的说明和标题字段中扫描包含“Ocean view”的文档，此外，还会根据字词“spacious”或者以前缀“air-condition”开头的字词执行搜索。 `searchMode` 参数用于匹配任一字词（默认）；如果未明确指定字词 (`+`)，则匹配所有字词。
3. 根据与给定地理位置的距离将酒店结果集排序，然后将其返回到调用方应用程序。 

本文的大部分内容介绍如何处理*搜索查询*：`"Spacious, air-condition* +\"Ocean view\""`。 筛选和排序不属于本文的介绍范畴。 有关详细信息，请参阅[搜索 API 参考文档](https://docs.microsoft.com/rest/api/searchservice/search-documents)。

<a name="stage1"></a>
## <a name="stage-1-query-parsing"></a>阶段 1：查询分析 

如前所述，查询字符串是请求的第一行： 

~~~~
 "search": "Spacious, air-condition* +\"Ocean view\"", 
~~~~

查询分析器会将运算符（例如本示例中的 `*` 和 `+`）与搜索词区分开来，并将搜索查询解构成受支持类型的*子查询*： 

+ 针对独立字词（例如 spacious）的*字词查询*
+ 针对带引号字词（例如 ocean view）的*短语查询*
+ 针对字词后接前缀运算符 `*`（例如 air-condition）的*前缀查询*

有关支持的查询类型的完整列表，请参阅 [Lucene 查询语法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)

与子查询关联的运算符确定是“必须”还是“应该”满足该查询，才将某个文档视为匹配项。 例如，由于使用了 `+` 运算符，`+"Ocean view"` 表示“必须”满足查询。 

查询分析器将传递给搜索引擎的子查询重新构造成*查询树*（表示查询的内部结构）。 在查询分析的第一个阶段，查询树如下所示。  

 ![searchmode 设置为 any 的布尔查询][2]

### <a name="supported-parsers-simple-and-full-lucene"></a>支持的分析器：简单和完整 Lucene 

 Azure 搜索公开两种不同的查询语言：`simple`（默认）和 `full`。 通过使用搜索请求设置 `queryType` 参数，可让查询分析器知道你选择的查询语言，这样，它就知道如何解释运算符和语法。 [简单查询语言](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)比较直观而且可靠，通常适合用于按原样解释用户输入，而无需客户端的处理。 它支持 Web 搜索引擎中常见的查询运算符。 [完整 Lucene 查询语言](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)：可通过设置 `queryType=full` 获取该语言。它添加了对更多运算符和查询类型（例如通配符、模糊查询、正则表达式和限定字段的查询）的支持，从而扩展了默认的简单查询语言。 例如，在简单查询语法中发送的正则表达式将解释为查询字符串而不是表达式。 本文中的示例请求使用完整 Lucene 查询语言。

### <a name="impact-of-searchmode-on-the-parser"></a>searchMode 对分析器的影响 

影响分析的另一个搜索请求参数是 `searchMode` 参数。 该参数控制布尔查询的默认运算符：any（默认）或 all。  

如果 `searchMode=any`（默认设置），则 spacious 与 air-condition 之间的空间分隔符为 OR (`||`)，因此，示例查询文本等效于： 

~~~~
Spacious,||air-condition*+"Ocean view" 
~~~~

显式运算符（例如 `+"Ocean view"` 中的 `+`）在布尔查询构造中没有歧义（*必须*匹配字词）。 剩余字词的解释方式不太明确：spacious 和 air-condition。 搜索引擎是否应该根据 ocean view *和* spacious *和* air-condition 查找匹配项？ 或者，是否应该查找 ocean view 加上*任何一个*剩余的字词？ 

默认情况下 (`searchMode=any`)，搜索引擎采用更广泛的解释。 *应该*匹配任一字段，反映“or”的语义。 上面所示的初始查询树包含两个“should”运算符，显示了默认行为。  

假设我们现在设置为 `searchMode=all`。 在这种情况下，空格被解释为“and”运算。 文档中必须包含每个剩余的字词，才能将该文档视为匹配项。 生成的示例查询将按以下方式解释： 

~~~~
+Spacious,+air-condition*+"Ocean view"
~~~~

此查询的修改查询树如下所示，其中的匹配文档是所有三个子查询的交集： 

 ![searchmode 设置为 all 的布尔查询][3]

> [!Note] 
> 运行代表性查询时，选择 `searchMode=any` 而不选择 `searchMode=all` 是最明智的决定。 经常使用运算符（搜索文档存储时就经常这样做）的用户可能会发现，如果 `searchMode=all` 能够告知布尔查询构造，则结果会更直观。 有关 `searchMode` 与运算符之间的交互作用的详细信息，请参阅[简单查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)。

<a name="stage2"></a>
## <a name="stage-2-lexical-analysis"></a>阶段 2：词法分析 

构造查询树之后，词法分析器将处理*字词查询*和*短语查询*。 分析器接受分析器提供给它的文本输入，处理文本，并发回标记化的字词，以便在查询树中整合。 

词法分析的最常见形式是*语言分析*，这种分析可以根据给定语言特定的规则转换查询词： 

* 将查询词化简为单词的词根形式 
* 删除不必要的单词（非索引字，例如英语中的“the”或“and”） 
* 将复合词分解为不同的组成部分 
* 转换单词的大小写 

所有这些操作往往会消除用户提供的文本输入与存储在索引中的字词之间的差异。 此类操作超出了文本处理的范围，需要精通语言本身。 为了添加这一层语言认知力，Azure 搜索支持 Lucene 和 Microsoft 提供的众多[语言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

> [!Note]
> 根据具体的情景，分析要求时而简单，时而繁琐。 可以通过选择某个预定义的分析器或者创建自己的[自定义分析器](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search)来控制词法分析的复杂性。 可将分析器的分析范围限定为可搜索的字段，并且可将分析器指定为字段定义的一部分。 这样，便可以根据每个字段运行不同的词法分析。 如果未指定分析器，将使用*标准* Lucene 分析器。

在本示例中，分析之前的初始查询树包含字词“Spacious,”，其中使用了大写的 S 以及一个逗号。查询分析器会将逗号解释为查询字词的一部分（逗号不被视为查询语言运算符）。  

当默认分析器处理该字词时，会将“ocean view”和“spacious”转换为小写，并删除逗号字符。 修改后的查询树如下所示： 

 ![包含已分析字词的布尔查询][4]

### <a name="testing-analyzer-behaviors"></a>测试分析器的行为 

可以使用[分析 API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer) 测试分析器的行为。 提供要分析的文本，查看给定的分析器会生成哪些字词。 例如，若要查看标准分析器如何处理文本“air-condition”，可以发出以下请求：

~~~~
{
    "text": "air-condition",
    "analyzer": "standard"
}
~~~~

标准分析器会将输入文本分解成以下两个标记，使用起始和结束偏移（用于命中项突出显示）以及文本的位置（用于短语匹配）等属性来批注输入文本：

~~~~
{
  "tokens": [
    {
      "token": "air",
      "startOffset": 0,
      "endOffset": 3,
      "position": 0
    },
    {
      "token": "condition",
      "startOffset": 4,
      "endOffset": 13,
      "position": 1
    }
  ]
}
~~~~

<a name="exceptions"></a>

### <a name="exceptions-to-lexical-analysis"></a>词法分析的例外情况 

词法分析仅适用于需要完整字词的查询类型 – 字词查询或短语查询， 而不适用于使用不完整字词的查询类型 – 前缀查询、通配符查询、正则表达式查询，或者模糊查询。 这些查询类型（包括前缀查询，在本示例中包含字词 `air-condition*`）将直接添加到查询树，会绕过分析阶段。 针对这些类型的查询字词执行的唯一转换操作是转换为小写。

<a name="stage3"></a>

## <a name="stage-3-document-retrieval"></a>阶段 3：文档检索 

文档检索是否在索引中查找包含匹配词的文档。 最好是通过一个示例来理解此阶段。 我们从一个采用以下简单架构的酒店索引着手： 

~~~~
{
    "name": "hotels",
    "fields": [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "title", "type": "Edm.String", "searchable": true },
        { "name": "description", "type": "Edm.String", "searchable": true }
    ] 
} 
~~~~

进一步假设此索引包含以下四个文档： 

~~~~
{
    "value": [
        {
            "id": "1",
            "title": "Hotel Atman",
            "description": "Spacious rooms, ocean view, walking distance to the beach."
        },
        {
            "id": "2",
            "title": "Beach Resort",
            "description": "Located on the north shore of the island of Kauaʻi. Ocean view."
        },
        {
            "id": "3",
            "title": "Playa Hotel",
            "description": "Comfortable, air-conditioned rooms with ocean view."
        },
        {
            "id": "4",
            "title": "Ocean Retreat",
            "description": "Quiet and secluded"
        }
    ]
}
~~~~

**字词的索引编制方式**

了解索引的一些基本知识有助于理解检索。 存储的单位是一个倒排索引，每个可搜索字段对应一个索引。 在倒排索引中，有一个来自所有文档的所有字词的排序列表。 每个字词映射到出现该字词的文档列表，以下示例清晰演示了这种映射。

要在倒排索引中生成字词，搜索引擎将针对文档内容执行词法分析，这类似于查询处理期间执行的操作：

1. 根据分析器的配置，执行将文本输入传递给分析器、转换为小写、去除标点等操作。 
2. 令牌是文本分析的输出。
3. 将词语添加到索引。

我们经常（但不是非要这样做）使用相同的分析器来执行搜索和索引编制操作，使查询词看上去更像是索引中的字词。

> [!Note]
> Azure 搜索允许通过附加的 `indexAnalyzer` 和 `searchAnalyzer` 字段参数来指定使用不同的分析器执行索引和搜索。 如果未指定，使用设置分析器`analyzer`属性用于索引编制和搜索。  

**示例文档的倒排索引**

继续使用前面的示例，对于**标题**字段，倒排索引如下所示：

| 术语 | 文档列表 |
|------|---------------|
| atman | 第 |
| beach | 2 |
| hotel | 1, 3 |
| ocean | 4  |
| playa | 3 |
| resort | 3 |
| retreat | 4 |

在标题字段中，只有“酒店”显示在以下两个文档中：1、3。

对于**说明**字段，索引如下所示：

| 术语 | 文档列表 |
|------|---------------|
| air | 3
| and | 4
| beach | 1
| conditioned | 3
| comfortable | 3
| distance | 1
| island | 2
| kauaʻi | 2
| located | 2
| north | 2
| ocean | 1, 2, 3
| of | 2
| on |2
| quiet | 4
| rooms  | 1, 3
| secluded | 4
| shore | 2
| spacious | 1
| the | 1, 2
| 至 | 第
| view | 1, 2, 3
| walking | 第
| 替换为 | 3


**根据编制索引的字词匹配查询词**

定义倒排索引后，我们继续使用前面的示例查询，了解如何针对该示例查询查找匹配的文档。 回想一下，最终的查询树如下所示： 

 ![包含已分析字词的布尔查询][4]

在查询执行过程中，会针对可搜索字段单独执行各个查询。 

+ 字词查询“spacious”匹配文档 1 (Hotel Atman)。 

+ 前缀查询“air-condition*”不匹配任何文档。 

  这种行为有时会让开发人员感到混淆。 尽管字词 air-conditioned 在文档中存在，但它已被默认分析器拆分成两个字词。 前面已经提到，不会分析包含部分字词的前缀查询。 因此，将在倒置索引中查找包含前缀“air-condition”的字词，但不会找到。

+ 短语查询“ocean view”将在原始文档中查找字词“ocean”和“view”并检查字词的近似性。 文档 1、2 和 3 的说明字段匹配此查询。 请注意，文档 4 的标题中包含字词 ocean，但不被视为匹配项，因为我们要查找的是短语“ocean view”而不是单个单词。 

> [!Note]
> 除非使用 `searchFields` 参数限制了字段集（如示例搜索请求中所示），否则，将针对 Azure 搜索索引中的所有可搜索字段单独执行搜索查询。 将返回与任一选定字段匹配的文档。 

总而言之，对于上述查询，匹配的文档为 1、2、3。 

## <a name="stage-4-scoring"></a>阶段 4：评分  

将为搜索结果集中的每个文档分配一个相关性评分。 相关性评分的作用是提高能够为搜索查询所表示的用户问题提供最佳答案的文档的排名。 评分是根据匹配的字词的统计属性计算的。 评分公式的核心是 [TF/IDF（字词频率-逆向文档频率）](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)。 在包含不常见和常见字词的查询中，TF/IDF 会提升包含不常见字词的结果。 例如，在包含所有 Wikipedia 文章的假想索引中，对于匹配查询 *the president* 的文档，匹配 *president* 的文档的相关性被视为高于匹配 *the* 的文档。


### <a name="scoring-example"></a>评分示例

回想一下与示例查询匹配的三个文档：
~~~~
search=Spacious, air-condition* +"Ocean view"  
~~~~
~~~~
{
  "value": [
    {
      "@search.score": 0.25610128,
      "id": "1",
      "title": "Hotel Atman",
      "description": "Spacious rooms, ocean view, walking distance to the beach."
    },
    {
      "@search.score": 0.08951007,
      "id": "3",
      "title": "Playa Hotel",
      "description": "Comfortable, air-conditioned rooms with ocean view."
    },
    {
      "@search.score": 0.05967338,
      "id": "2",
      "title": "Ocean Resort",
      "description": "Located on a cliff on the north shore of the island of Kauai. Ocean view."
    }
  ]
}
~~~~

文档 1 与查询的匹配程度最高，因为其说明字段中同时出现了字词 *spacious* 和所需的短语 *ocean view*。 后面的两个文档仅匹配短语 *ocean view*。 你可能会感到惊讶，尽管文档 2 和 3 都匹配相同的查询短语，但它们的相关性评分却不相同。 这是因为，评分公式除了包含 TF/IDF 以外，还包含其他组成部分。 在本例中，为文档 3 分配的评分略高，因为其说明更短。 请学习 [Lucene 的实际评分公式](https://lucene.apache.org/core/4_0_0/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html)，了解字段长度和其他因素如何影响相关性评分。

某些查询类型（通配符、前缀、正则表达式）始终会给文档总评分贡献一个常量分数。 这样，便可以在结果中包含通过查询扩展找到的匹配项，但不会影响排名。 

有一个示例演示了这种行为的原因。 通配符搜索（包括前缀搜索）在定义上有歧义，因为输入是一个不完整的字符串，可能会匹配极大量的相异字词（假设输入为“tour*”，这样就会找到“tours”、“tourettes”和“tourmaline”的匹配项）。 由于这些结果的性质，我们无法合理推断出哪些字词的相关性高于其他字词。 为此，在为通配符、前缀和正则表达式类型的查询中的结果评分时，我们会忽略字词频率。 在包含不完整和完整字词的多部分搜索请求中，来自不完整输入的结果将与常量评分合并，以免出现偏差并返回潜在的意外匹配项。

### <a name="score-tuning"></a>评分优化

在 Azure 搜索中，可以使用两种方法优化相关性评分：

1. **评分配置文件**可以根据一组规则提升结果排名列表中的文档。 在本示例中，我们可以认为标题字段中匹配的文档的相关性高于说明字段中匹配的文档。 此外，如果索引包含每家酒店的价格字段，我们可以根据较低的价格提升文档。 详细了解如何[将评分配置文件添加到搜索索引](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。
2. **字词提升**（只能在完整 Lucene 查询语法中使用）提供可应用到查询树任何部分的提升运算符 `^`。 在本示例中，我们可以不搜索前缀 *air-condition*\*，而是搜索确切的字词 *air-condition* 或前缀，但是，由于在字词查询中应用了提升运算符，与该确切字词匹配的文档会获得更高的排名：*air-condition^2||air-condition*\*。 详细了解[字词提升](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search#bkmk_termboost)。


### <a name="scoring-in-a-distributed-index"></a>在分布式索引评分

Azure 搜索中的所有索引会自动拆分成多个分片，使我们能够在服务扩展或缩减期间，快速地在多个节点之间分配索引。 发出某个搜索请求时，会单独针对每个分片发出该请求。 然后，来自每个分片的结果会合并，并按评分排序（如果未定义其他排序）。 必须知道，评分函数根据分片中所有文档内的字词反向文档频率为查询字词频率加权，而不是根据所有分片加权！

这意味着，如果相同的文档驻留在不同的分片中，其相关性评分*可能*不同。 幸运的是，随着索引中的文档数由于字词分布越来越均匀而不断增多，这种差异往往会消失。 无法预料任意给定文档会放置在哪个分片上。 但是，假设文档键不会更改，该文档始终会分配到同一个分片。

一般而言，如果顺序稳定性非常重要，则文档评分并不是用于文档排序的最佳属性。 例如，假设两个文档具有相同的评分，则无法保证以后运行同一个查询时，会先显示哪个文档。 文档评分只能让你大致了解某个文档的相关性相对于结果集中其他文档的高低程度。

## <a name="conclusion"></a>结束语

Internet 搜索引擎取得的成功提高了人们对私有数据运行全文搜索的预期。 对于几乎所有类型的搜索体验，我们现在都会预期引擎理解我们的意图，即使搜索词拼写不当或者不完整。 我们甚至预期可以根据近似的字词或者我们从未实际指定的同义词执行匹配。

从技术角度看，全文搜索非常复杂，需要采用高深的语言分析和系统性方法，通过提取、扩展和转换查询词进行处理并提供相关的结果。 除了固有的复杂性以外，还有许多因素会影响查询的结果。 因此，在尝试处理意外结果时投入一些时间来了解全文搜索的机制可以获得实实在在的好处。  

本文探讨了 Azure 搜索上下文中的全文搜索。 我们希望本文提供了足够的背景知识，可让你识别潜在的原因和解决常见的查询问题。 

## <a name="next-steps"></a>后续步骤

+ 生成示例索引，尝试不同的查询并查看结果。 有关说明，请参阅[在门户中生成和查询索引](search-get-started-portal.md#query-index)。

+ 通过[搜索文档](https://docs.microsoft.com/rest/api/searchservice/search-documents#bkmk_examples)示例部分或者通过门户中“搜索资源管理器”的[简单查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)尝试其他查询语法。

+ 如果想要在搜索应用程序中优化排名，请查看[评分配置文件](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。

+ 了解如何应用[语言特定的词法分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

+ [配置自定义分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)，针对特定的字段尽量简化处理或者进行专门处理。

+ 在此演示网站并排[比较标准和英语分析器](https://alice.unearth.ai/)。 

## <a name="see-also"></a>另请参阅

[搜索文档 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

[简单的查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 

[完整 Lucene 查询语法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 

[处理搜索结果](https://docs.microsoft.com/azure/search/search-pagination-page-layout)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
[2]: ./media/search-lucene-query-architecture/azSearch-queryparsing-should2.png
[3]: ./media/search-lucene-query-architecture/azSearch-queryparsing-must2.png
[4]: ./media/search-lucene-query-architecture/azSearch-queryparsing-spacious2.png
