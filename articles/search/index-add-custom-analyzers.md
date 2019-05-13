---
title: 添加自定义分析器 - Azure 搜索
description: 修改 Azure 搜索全文搜索查询中所用的文本 tokenizer 和字符筛选器。
ms.date: 02/14/2019
services: search
ms.service: search
ms.topic: conceptual
author: Yahnoosh
ms.author: jlembicz
ms.manager: cgronlun
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 5983c2036ff206825a82072b57ca2b9bb44ca678
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021663"
---
# <a name="add-custom-analyzers-to-an-azure-search-index"></a>向 Azure 搜索索引添加自定义分析器

自定义分析器是[文本分析器](search-analyzers.md)的一种特定类型，包含现有 tokenizer 和可选筛选器的用户定义组合。 通过以新方式组合 tokenizer 和筛选器，可以在搜索引擎中自定义文本处理以得到特定结果。 例如，可以使用*字符筛选器*创建自定义分析器，以在标记文本输入之前删除 HTML 标记。

 可以定义多个自定义分析器来改变筛选器组合，但每个字段只能使用一个分析器进行索引分析，一个分析器进行搜索分析。 有关自定义分析器外观的说明，请参见[自定义分析器示例](search-analyzers.md#Custom-analyzer-example)。

## <a name="overview"></a>概述

 简单来说，[全文搜索引擎](search-lucene-query-architecture.md)的作用是以能够进行有效查询和检索的方式处理和存储文档。 从较高层面来说，就是从文档中提取重要字词，将它们放入索引，然后使用索引查找与给定查询的字词匹配的文档。 从文档和搜索查询中提取字词的过程称为“词法分析”。 执行词法分析的组件称为“分析器”。

 在 Azure 搜索中，可以从[分析器](#AnalyzerTable)表中的一组预定义语言不可知分析器或[语言分析器（Azure 搜索服务 REST API）](index-add-language-analyzers.md)中列出的语言特定分析器中进行选择。 也可以选择定义自己的自定义分析器。  

 自定义分析器允许控制将文本转换为可索引且可搜索标记的过程。 它是一种用户定义的配置，由一个预定义的 tokenizer、一个或多个标记筛选器以及一个或多个字符筛选器组成。 Tokenizer 负责将文本分解成多个标记，标记筛选器负责修改 tokenizer 发出的标记。 在由 tokenizer 处理输入文本之前，可应用字符筛选器来准备输入文本。 例如，字符筛选器可以替换某些字符或符号。

 自定义分析器支持的常见方案包括：  

- 拼音搜索。 添加一个拼音筛选器，以便根据字词的发音方式而不是拼写方式进行搜索。  

- 禁用词法分析。 使用关键字分析器创建未分析的可搜索字段。  

- 快速前缀/后缀搜索。 添加 Edge N 元语法标记筛选器以对字词前缀编制索引，从而启用快速前缀匹配。 将其与反向标记筛选器组合使用，以执行后缀匹配。  

- 自定义词汇切分。 例如，借助空格 tokenizer，通过将空格用作分隔符来将句子分解成标记  

- ASCII 折叠。 添加标准 ASCII 折叠筛选器以规范化搜索词中的音调符号，如 ö 或 ê。  

  本页面提供了受支持的分析器、tokenizer、标记筛选器和字符筛选器的列表。 你还可以找到索引定义更改的说明以及用法示例。 有关 Azure 搜索实现中使用的基础技术的更多背景信息，请参阅[分析包摘要 (Lucene)](https://lucene.apache.org/core/4_10_0/core/org/apache/lucene/codecs/lucene410/package-summary.html)。 有关分析器配置的示例，请参阅[在 Azure 搜索中添加分析器](search-analyzers.md#examples)。

## <a name="validation-rules"></a>验证规则  
 分析器、tokenizer、标记筛选器和字符筛选器的名称必须是唯一的，不能与任何预定义的分析器、tokenizer、标记筛选器或字符筛选器相​​同。 有关已使用的名称，请参阅[属性参考](#PropertyReference)。

## <a name="create-custom-analyzers"></a>创建自定义分析器
 可以在创建索引时定义自定义分析器。 本部分介绍用于指定自定义分析器的语法。 也可以通过查看[在 Azure 搜索中添加分析器](search-analyzers.md#examples)中的示例定义来熟悉该语法。  

 分析器定义包括名称、类型、一个或多个字符筛选器、最多一个 tokenizer，以及一个或多个用于后期词汇切分处理的标记筛选器。 字符筛选器在词汇切分前应用。 标记筛选器和字符筛选器按从左到右的顺序应用。

 `tokenizer_name`是标记器的名称`token_filter_name_1`和`token_filter_name_2`是令牌筛选器的名称和`char_filter_name_1`并`char_filter_name_2`是字符的名称筛选器 (请参阅[标记化器](#Tokenizers)， [令牌筛选器](#TokenFilters)和 Char 筛选器表的有效的值)。

分析器定义是较大索引的一部分。 有关索引其余部分的信息，请参阅[创建索引 API](https://docs.microsoft.com/rest/api/searchservice/create-index)。

```
"analyzers":(optional)[
   {
      "name":"name of analyzer",
      "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
      "charFilters":[
         "char_filter_name_1",
         "char_filter_name_2"
      ],
      "tokenizer":"tokenizer_name",
      "tokenFilters":[
         "token_filter_name_1",
         "token_filter_name_2"
      ]
   },
   {
      "name":"name of analyzer",
      "@odata.type":"#analyzer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"charFilters":(optional)[
   {
      "name":"char_filter_name",
      "@odata.type":"#char_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenizers":(optional)[
   {
      "name":"tokenizer_name",
      "@odata.type":"#tokenizer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenFilters":(optional)[
   {
      "name":"token_filter_name",
      "@odata.type":"#token_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
]
```

> [!NOTE]  
>  你创建的自定义分析器不会在 Azure 门户中公开。 添加自定义分析器的唯一方法是在定义索引时通过使用调用 API 的代码。  

 在索引定义中，可以将此节放在创建索引请求正文中的任意位置，但通常将它放在末尾：  

```
{
  "name": "name_of_index",
  "fields": [ ],
  "suggesters": [ ],
  "scoringProfiles": [ ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) { },
  "analyzers":(optional)[ ],
  "charFilters":(optional)[ ],
  "tokenizers":(optional)[ ],
  "tokenFilters":(optional)[ ]
}
```

只有在设置自定义选项时，才向索引添加字符筛选器、tokenizer 和标记筛选器的定义。 若要按原样使用现有筛选器或 tokenizer，请在分析器定义中按名称指定它。

<a name="Testing custom analyzers"></a>

## <a name="test-custom-analyzers"></a>测试自定义分析器

可以使用 [REST API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer) 中的**测试分析器操作**来查看分析器如何将给定文本分解成多个标记。

**请求**
```
  POST https://[search service name].search.windows.net/indexes/[index name]/analyze?api-version=[api-version]
  Content-Type: application/json
    api-key: [admin key]

  {
     "analyzer":"my_analyzer",
     "text": "Vis-à-vis means Opposite"
  }
```
**响应**
```
  {
    "tokens": [
      {
        "token": "vis_a_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "vis_à_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "means",
        "startOffset": 10,
        "endOffset": 15,
        "position": 1
      },
      {
        "token": "opposite",
        "startOffset": 16,
        "endOffset": 24,
        "position": 2
      }
    ]
  }
```

## <a name="update-custom-analyzers"></a>更新自定义分析器

定义分析器、tokenizer、标记筛选器或字符筛选器后，便无法修改它。 仅当 `allowIndexDowntime` 标志在索引更新请求中设置为 true 时，才可向现有索引添加新的上述内容：

```
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=[api-version]&allowIndexDowntime=true
```

此操作将使索引离线至少几秒钟，从而导致索引和查询请求失败。 索引的性能和写入可用性可能会在更新索引后的几分钟内处于受损状态，对于非常大的索引，持续时间更长，但这些影响只是暂时的，最终将自行解除。

 <a name="ReferenceIndexAttributes"></a>

## <a name="analyzer-reference"></a>分析器引用

下表列出了索引定义的分析器、tokenizer、标记筛选器和字符筛选器节的配置属性。 索引中的分析器、tokenizer 或筛选器的结构均由这些属性组成。 有关赋值信息，请参阅[属性参考](#PropertyReference)。

### <a name="analyzers"></a>分析器

对于分析器，索引属性取决于你使用的是预定义分析器还是自定义分析器。

#### <a name="predefined-analyzers"></a>预定义分析器

|||  
|-|-|  
|名称|它必须仅包含字母、数字、空格、短划线或下划线，只能以字母数字字符开头和结尾，且最多包含 128 个字符。|  
|Type|分析器类型来自受支持分析器列表。 请参阅下面[分析器](#AnalyzerTable)表中的 **analyzer_type** 列。|  
|选项|必须是下面[分析器](#AnalyzerTable)表中列出的预定义分析器的有效选项。|  

#### <a name="custom-analyzers"></a>自定义分析器

|||  
|-|-|  
|名称|它必须仅包含字母、数字、空格、短划线或下划线，只能以字母数字字符开头和结尾，且最多包含 128 个字符。|  
|Type|必须是“#Microsoft.Azure.Search.CustomAnalyzer”。|  
|CharFilters|设置为[字符筛选器](#CharFilter)表中列出的预定义字符筛选器之一或索引定义中指定的自定义字符筛选器。|  
|分词器|必需。 设置为下面 [Tokenizer](#Tokenizers) 表中列出的预定义 tokenizer 之一或索引定义中指定的自定义 tokenizer。|  
|TokenFilters|设置为[标记筛选器](#TokenFilters)表中列出的预定义标记筛选器之一或索引定义中指定的自定义标记筛选器。|  

<a name="CharFilter"></a>

### <a name="char-filters"></a>字符筛选器

 字符筛选器用于在 tokenizer 处理输入文本之前准备输入文本。 例如，它们可以替换某些字符或符号。 可以在自定义分析器中使用多个字符筛选器。 字符筛选器按列出的顺序运行。  

|||  
|-|-|  
|名称|它必须仅包含字母、数字、空格、短划线或下划线，只能以字母数字字符开头和结尾，且最多包含 128 个字符。|  
|Type|字符筛选器类型来自受支持字符筛选器列表。 请参阅下面[字符筛选器](#CharFilter)表中的 **char_filter_type** 列。|  
|选项|必须是给定[字符筛选器](#CharFilter)类型的有效选项。|  

### <a name="tokenizers"></a>Tokenizer

 Tokenizer 将连续文本划分为一系列标记，例如将一个句子分解成多个字词。  

 可以为每个自定义分析器指定一个 tokenizer。 如果需要多个 tokenizer，则可以创建多个自定义分析器，并在索引架构中逐个字段地分配它们。  
自定义分析器可以使用带有默认或自定义选项的预定义 tokenizer。  

|||  
|-|-|  
|名称|它必须仅包含字母、数字、空格、短划线或下划线，只能以字母数字字符开头和结尾，且最多包含 128 个字符。|  
|Type|Tokenizer 名称来自受支持 tokenizer 列表。 请参阅下面 [Tokenizer](#Tokenizers) 表中的 **tokenizer_type** 列。|  
|选项|必须是下面 [Tokenizer](#Tokenizers) 表中列出的给定 tokenizer 类型的有效选项。|  

### <a name="token-filters"></a>标记筛选器

 标记筛选器用于筛选出或修改由 tokenizer 生成的标记。 例如，可以指定将所有字符转换为小写的小写筛选器。   
可以在自定义分析器中使用多个标记筛选器。 标记筛选器按列出的顺序运行。  

|||  
|-|-|  
|名称|它必须仅包含字母、数字、空格、短划线或下划线，只能以字母数字字符开头和结尾，且最多包含 128 个字符。|  
|Type|标记筛选器名称来自受支持标记筛选器列表。 请参阅下面[标记筛选器](#TokenFilters)表中的 **token_filter_type** 列。|  
|选项|必须是给定标记筛选器类型的[标记筛选器](#TokenFilters)。|  

<a name="PropertyReference"></a>  

## <a name="property-reference"></a>属性参考

本部分提供索引中自定义分析器、tokenizer、字符筛选器或标记筛选器的定义中指定的属性的有效值。 使用 Apache Lucene 实现的分析器、tokenizer 和筛选器已链接到相应的 Lucene API 文档。

<a name="AnalyzerTable"></a>

###  <a name="predefined-analyzers-reference"></a>预定义分析器参考

|**analyzer_name**|**analyzer_type**  <sup>1</sup>|**说明和选项**|  
|-|-|-|  
|[keyword](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)| （仅当有可用的选项时，类型才适用） |将某个字段的整个内容视为单个标记。 此分析器可用于邮政编码、ID 和某些产品名称等数据。|  
|[pattern](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/PatternAnalyzer.html)|PatternAnalyzer|通过正则表达式模式将文本灵活地分解成多个词条。<br /><br /> 选项<br /><br /> lowercase (type: bool) - 确定词条是否为小写。 默认值为 true。<br /><br /> [pattern](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html?is-external=true) (type: string) - 用于匹配标记分隔符的正则表达式模式。 默认值为 \w+。<br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (type: string) - 正则表达式标志。 默认值为空字符串。 允许的值：CANON_EQ、CASE_INSENSITIVE、COMMENTS、DOTALL、LITERAL、MULTILINE、UNICODE_CASE、UNIX_LINES<br /><br /> stopwords (type: string array) - 非索引字列表。 默认为空列表。|  
|[simple](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/SimpleAnalyzer.html)|（仅当有可用的选项时，类型才适用） |在非字母处划分文本并将其转换为小写。 |  
|[standard](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) <br />（也称为 standard.lucene）|StandardAnalyzer|标准 Lucene 分析器，由标准 tokenizer、小写筛选器和停止筛选器组成。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值为 255。 超过最大长度的标记将被拆分。 可以使用的最大标记长度为 300 个字符。<br /><br /> stopwords (type: string array) - 非索引字列表。 默认为空列表。|  
|standardasciifolding.lucene|（仅当有可用的选项时，类型才适用） |带 Ascii 折叠筛选器的标准分析器。 |  
|[stop](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/StopAnalyzer.html)|StopAnalyzer|在非字母处划分文本，应用小写和非索引字标记筛选器。<br /><br /> 选项<br /><br /> stopwords (type: string array) - 非索引字列表。 默认值为英语预定义列表。 |  
|[whitespace](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html)|（仅当有可用的选项时，类型才适用） |使用空格 tokenizer 的分析器。 超过 255 个字符的标记将被拆分。|  

 <sup>1</sup> 分析器类型在代码中始终带有前缀“#Microsoft.Azure.Search”，因此，“PatternAnalyzer”实际上会被指定为“#Microsoft.Azure.Search.PatternAnalyzer”。 为简洁起见，我们删除了该前缀，但在代码中需要使用该前缀。 
 
analyzer_type 仅适用于可自定义的分析器。 如果没有选项（比如 keyword 分析器），则没有关联的 #Microsoft.Azure.Search 类型。


<a name="CharFilter"></a>

###  <a name="char-filters-reference"></a>字符筛选器参考

在下表中，使用 Apache Lucene 实现的字符筛选器已链接到相应的 Lucene API 文档。

|**char_filter_name**|**char_filter_type** <sup>1</sup>|**说明和选项**|  
|-|-|-|
|[html_strip](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html)|（仅当有可用的选项时，类型才适用）  |一个字符筛选器，它尝试去除 HTML 构造。|  
|[mapping](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)|MappingCharFilter|一个字符筛选器，它应用使用 mappings 选项定义的映射。 匹配具有贪婪性（给定点的最长模式匹配获胜）。 允许替换为空字符串。<br /><br /> 选项<br /><br /> mappings (type: string array) - 以下格式的映射列表：“a=>b”（出现的所有字符“a”均替换为字符“b”）。 必需。|  
|[pattern_replace](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html)|PatternReplaceCharFilter|一个字符筛选器，用于替换输入字符串中的字符。 它使用正则表达式来标识要保留的字符序列，并使用替换模式来标识要替换的字符。 例如，input text = "aa  bb aa bb", pattern="(aa)\\\s+(bb)" replacement="$1#$2", result = "aa#bb aa#bb"。<br /><br /> 选项<br /><br /> pattern (type: string) - 必需。<br /><br /> replacement (type: string) - 必需。|  

 <sup>1</sup> 字符筛选器类型在代码中始终带有前缀“#Microsoft.Azure.Search”，因此，“MappingCharFilter”实际上会被指定为“#Microsoft.Azure.Search.MappingCharFilter”。 为缩小表的宽度，我们删除了该前缀，但请记住在代码中包含该前缀。 请注意该 char_filter_type 仅提供的可自定义的筛选器。 如果没有选项（比如 html_strip），则没有关联的 #Microsoft.Azure.Search 类型。

<a name="Tokenizers"></a>

###  <a name="tokenizers-reference"></a>Tokenizer 参考

在下表中，使用 Apache Lucene 实现的 tokenizer 已链接到相应的 Lucene API 文档。

|**tokenizer_name**|**tokenizer_type** <sup>1</sup>|**说明和选项**|  
|-|-|-|  
|[经典](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/ClassicTokenizer.html)|ClassicTokenizer|基于语法的 tokenizer，适合处理大多数欧洲语言文档。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值：255，最大值：300。 超过最大长度的标记将被拆分。|  
|[edgeNGram](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html)|EdgeNGramTokenizer|将来自 Edge 的输入标记为给定大小的 n 元语法。<br /><br /> 选项<br /><br /> minGram (type: int) - 默认值：1，最大值：300。<br /><br /> maxGram (type: int) - 默认值：2，最大值：300。 必须大于 minGram。<br /><br /> tokenChars (type: string array) - 要保留在标记中的字符类。 允许的值： <br />“letter”、“digit”、“whitespace”、“punctuation”、“symbol”。 默认为空数组 - 保留所有字符。 |  
|[keyword_v2](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html)|KeywordTokenizerV2|将整个输入作为单个标记发出。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值：256，最大值：300。 超过最大长度的标记将被拆分。|  
|[letter](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/LetterTokenizer.html)|（仅当有可用的选项时，类型才适用）  |在非字母处划分文本。 超过 255 个字符的标记将被拆分。|  
|[lowercase](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/LowerCaseTokenizer.html)|（仅当有可用的选项时，类型才适用）  |在非字母处划分文本并将其转换为小写。 超过 255 个字符的标记将被拆分。|  
| microsoft_language_tokenizer| MicrosoftLanguageTokenizer| 使用特定于语言的规则划分文本。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度，默认值：255，最大值：300。 超过最大长度的标记将被拆分。 首先将超过 300 个字符的标记拆分为长度为 300 的标记，然后根据设置的 maxTokenLength 逐一拆分这些标记。<br /><br />isSearchTokenizer (type: bool) - 如果用作搜索 tokenizer，则设置为 true；如果用作索引 tokenizer，则设置为 false。 <br /><br /> language (type: string) - 要使用的语言，默认值为“english”。 允许的值包括：<br />“bangla”、“bulgarian”、“catalan”、“chineseSimplified”、“chineseTraditional”、“croatian”、“czech”、“danish”、“dutch”、“english”、“french”、“german”、“greek”、“gujarati”、“hindi”、“icelandic”、“indonesian”、“italian”、“japanese”、“kannada”、“korean”、“malay”、“malayalam”、“marathi”、“norwegianBokmaal”、“polish”、“portuguese”、“portugueseBrazilian”、“punjabi”、“romanian”、“russian”、“serbianCyrillic”、“serbianLatin”、“slovenian”、“spanish”、“swedish”、“tamil”、“telugu”、“thai”、“ukrainian”、“urdu”、“vietnamese” |
| microsoft_language_stemming_tokenizer | MicrosoftLanguageStemmingTokenizer| 使用特定于语言的规则划分文本，并将各字词缩减为其原形<br /><br /> 选项<br /><br />maxTokenLength (type: int) - 最大标记长度，默认值：255，最大值：300。 超过最大长度的标记将被拆分。 首先将超过 300 个字符的标记拆分为长度为 300 的标记，然后根据设置的 maxTokenLength 逐一拆分这些标记。<br /><br /> isSearchTokenizer (type: bool) - 如果用作搜索 tokenizer，则设置为 true；如果用作索引 tokenizer，则设置为 false。<br /><br /> language (type: string) - 要使用的语言，默认值为“english”。 允许的值包括：<br />“arabic”、“bangla”、“bulgarian”、“catalan”、“croatian”、“czech”、“danish”、“dutch”、“english”、“estonian”、“finnish”、“french”、“german”、“greek”、“gujarati”、“hebrew”、“hindi”、“hungarian”、“icelandic”、“indonesian”、“italian”、“kannada”、“latvian”、“lithuanian”、“malay”、“malayalam”、“marathi”、“norwegianBokmaal”、“polish”、“portuguese”、“portugueseBrazilian”、“punjabi”、“romanian”、“russian”、“serbianCyrillic”、“serbianLatin”、“slovak”、“slovenian”、“spanish”、“swedish”、“tamil”、“telugu”、“turkish”、“ukrainian”、“urdu” |
|[nGram](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenizer.html)|NGramTokenizer|将输入标记为给定大小的 n 元语法。<br /><br /> 选项<br /><br /> minGram (type: int) - 默认值：1，最大值：300。<br /><br /> maxGram (type: int) - 默认值：2，最大值：300。 必须大于 minGram。 <br /><br /> tokenChars (type: string array) - 要保留在标记中的字符类。 允许的值：“letter”、“digit”、“whitespace”、“punctuation”、“symbol”。 默认为空数组 - 保留所有字符。 |  
|[path_hierarchy_v2](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/path/PathHierarchyTokenizer.html)|PathHierarchyTokenizerV2|用于路径式层次结构的 tokenizer。<br /><br /> 选项<br /><br /> delimiter (type: string) - 默认值：'/。<br /><br /> replacement (type: string) - 如果设置该选项，则替换分隔符字符。 默认值与分隔符的值相同。<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值：300，最大值：300。 长度超过 maxTokenLength 的路径将被忽略。<br /><br /> reverse (type: bool) - 如果为 true，则按相反顺序生成标记。 默认值：false。<br /><br /> skip (type: bool) - 要跳过的初始标记。 默认值为 0。|  
|[pattern](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/pattern/PatternTokenizer.html)|PatternTokenizer|此 tokenizer 使用正则表达式模式匹配来构造不同的标记。<br /><br /> 选项<br /><br /> pattern (type: string) - 正则表达式模式。 默认值为 \w+。<br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (type: string) - 正则表达式标志。 默认值为空字符串。 允许的值：CANON_EQ、CASE_INSENSITIVE、COMMENTS、DOTALL、LITERAL、MULTILINE、UNICODE_CASE、UNIX_LINES<br /><br /> group (type: int) - 要提取到标记中的组。 默认值为 -1 (split)。|  
|[standard_v2](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/StandardTokenizer.html)|StandardTokenizerV2|按照 [Unicode 文本分段规则](https://unicode.org/reports/tr29/)划分文本。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值：255，最大值：300。 超过最大长度的标记将被拆分。|  
|[uax_url_email](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/UAX29URLEmailTokenizer.html)|UaxUrlEmailTokenizer|将 URL 和电子邮件标记为一个标记。<br /><br /> 选项<br /><br /> maxTokenLength (type: int) - 最大标记长度。 默认值：255，最大值：300。 超过最大长度的标记将被拆分。|  
|[whitespace](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/WhitespaceTokenizer.html)|（仅当有可用的选项时，类型才适用） |在空格处划分文本。 超过 255 个字符的标记将被拆分。|  

 <sup>1</sup> Tokenizer 类型在代码中始终带有前缀“#Microsoft.Azure.Search”，因此，“ClassicTokenizer”实际上会被指定为“#Microsoft.Azure.Search.ClassicTokenizer”。 为缩小表的宽度，我们删除了该前缀，但请记住在代码中包含该前缀。 请注意该 tokenizer_type 仅提供给可自定义的标记化器。 如果没有选项（比如 letter tokenizer），则没有关联的 #Microsoft.Azure.Search 类型。

<a name="TokenFilters"></a>

###  <a name="token-filters-reference"></a>标记筛选器参考

在下表中，使用 Apache Lucene 实现的标记筛选器已链接到相应的 Lucene API 文档。

|**token_filter_name**|**token_filter_type** <sup>1</sup>|**说明和选项**|  
|-|-|-|  
|[arabic_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizationFilter.html)|（仅当有可用的选项时，类型才适用）  |一个标记筛选器，它应用阿拉伯语规范化程序来规范化正字法。|  
|[apostrophe](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/tr/ApostropheFilter.html)|（仅当有可用的选项时，类型才适用）  |去除撇号后面的所有字符（包括撇号本身）。 |  
|[asciifolding](https://lucene.apache.org/core/4_0_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html)|AsciiFoldingTokenFilter|将不在前 127 个 ASCII 字符（“基本拉丁语”Unicode 块）中的字母、数字和符号 Unicode 字符转换为其 ASCII 等效字符（如果存在）。<br /><br /> 选项<br /><br /> preserveOriginal (type: bool) - 如果为 true，则保留原始标记。 默认值为 false。|  
|[cjk_bigram](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/cjk/CJKBigramFilter.html)|CjkBigramTokenFilter|形成 CJK 词条的二元语法，这些词条从 StandardTokenizer 生成。<br /><br /> 选项<br /><br /> ignoreScripts (type: string array) - 要忽略的脚本。 允许的值包括：“han”、“hiragana”、“katakana”、“hangul”。 默认为空列表。<br /><br /> outputUnigrams (type: bool) - 如果总是要输出单元语法和二元语法，则设置为 true。 默认值为 false。|  
|[cjk_width](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)|（仅当有可用的选项时，类型才适用）  |规范化 CJK 宽度差异。 将全角 ASCII 变体折叠成等效的基本拉丁语，将半角片假名变体折叠成等效的假名。 |  
|[经典](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/ClassicFilter.html)|（仅当有可用的选项时，类型才适用）  |删除英语所有格，以及首字母缩写词中的点。 |  
|[common_grams](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html)|CommonGramTokenFilter|在编制索引时为经常出现的词条构造二元语法。 此外，仍将为单个词条编制索引并叠加二元语法。<br /><br /> 选项<br /><br /> commonWords (type: string array) - 常见字词集。 默认为空列表。 必需。<br /><br /> ignoreCase (type: bool) - 如果为 true，则匹配不区分大小写。 默认值为 false。<br /><br /> queryMode (type: bool) - 生成二元语法，然后删除常见字词和常见字词后跟的单个词条。 默认值为 false。|  
|[dictionary_decompounder](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/compound/DictionaryCompoundWordTokenFilter.html)|DictionaryDecompounderTokenFilter|分解在许多日耳曼语系中找到的复合词。<br /><br /> 选项<br /><br /> wordList (type: string array) - 要匹配的字词列表。 默认为空列表。 必需。<br /><br /> minWordSize (type: int) - 只处理超过此长度的字词。 默认值为 5。<br /><br /> minSubwordSize (type: int) - 仅输出超过此长度的子字。 默认值为 2。<br /><br /> maxSubwordSize (type: int) - 仅输出短于此长度的子字。 默认值为 15。<br /><br /> onlyLongestMatch (type: bool) - 仅添加要输出的最长匹配子字。 默认值为 false。|  
|[edgeNGram_v2](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenFilter.html)|EdgeNGramTokenFilterV2|从输入标记的前面或后面开始生成给定大小的 n 元语法。<br /><br /> 选项<br /><br /> minGram (type: int) - 默认值：1，最大值：300。<br /><br /> maxGram (type: int) - 默认值：2，最大值：300。 必须大于 minGram。<br /><br /> side (type: string) - 指定应从输入的哪一侧生成 n 元语法。 允许的值：“front”、“back” |  
|[elision](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html)|ElisionTokenFilter|删除省音。 例如，“l'avion”(the plane) 转换为“avion”(plane)。<br /><br /> 选项<br /><br /> articles (type: string array) - 要删除的一组冠词。 默认为空列表。 如果没有设置冠词列表，默认情况下会删除所有法语冠词。|  
|[german_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)|（仅当有可用的选项时，类型才适用）  |根据 [German2 snowball 算法](https://snowballstem.org/algorithms/german2/stemmer.html)的试探方法规范化德语字符。|  
|[hindi_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizationFilter.html)|（仅当有可用的选项时，类型才适用）  |规范化印地语文本，以消除拼写变体中的一些差异。 |  
|[indic_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizationFilter.html)|IndicNormalizationTokenFilter|规范化印地语文本的 Unicode 表示形式。
|[keep](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeepWordFilter.html)|KeepTokenFilter|一个标记筛选器，仅保留具有包含在指定字词列表中的文本的标记。<br /><br /> 选项<br /><br /> keepWords (type: string array) - 要保留的字词列表。 默认为空列表。 必需。<br /><br /> keepWordsCase (type: bool) - 如果为 true，则首先小写所有字词。 默认值为 false。|  
|[keyword_marker](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordMarkerFilter.html)|KeywordMarkerTokenFilter|将词条标记为关键字。<br /><br /> 选项<br /><br /> keywords (type: string array) - 要标记为关键字的字词列表。 默认为空列表。 必需。<br /><br /> ignoreCase (type: bool) - 如果为 true，则首先小写所有字词。 默认值为 false。|  
|[keyword_repeat](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordRepeatFilter.html)|（仅当有可用的选项时，类型才适用）  |将每个传入标记发出两次，一次作为关键字，一次作为非关键字。 |  
|[kstem](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html)|（仅当有可用的选项时，类型才适用）  |适用于英语的高性能 kstem 筛选器。 |  
|[length](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/LengthFilter.html)|LengthTokenFilter|删除太长或太短的字词。<br /><br /> 选项<br /><br /> min (type: int) - 最小数目。 默认值：0，最大值：300。<br /><br /> max (type: int) - 最大数目。 默认值：300，最大值：300。|  
|[limit](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/LimitTokenCountFilter.html)|Microsoft.Azure.Search.LimitTokenFilter|编制索引时限制标记数量。<br /><br /> 选项<br /><br /> maxTokenCount (type: int) - 要生成的最大标记数。 默认值为 1。<br /><br /> consumeAllTokens (type: bool) - 是否在达到 maxTokenCount 的情况下，还必须使用输入中的所有标记。 默认值为 false。|  
|[lowercase](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html)|（仅当有可用的选项时，类型才适用）  |将标记文本规范化为小写。 |  
|[nGram_v2](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenFilter.html)|NGramTokenFilterV2|生成给定大小的 n 元语法。<br /><br /> 选项<br /><br /> minGram (type: int) - 默认值：1，最大值：300。<br /><br /> maxGram (type: int) - 默认值：2，最大值：300。 必须大于 minGram。|  
|[pattern_capture](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/pattern/PatternCaptureGroupTokenFilter.html)|PatternCaptureTokenFilter|使用 Java 正则表达式发出多个标记，一个或多个模式中的每个捕获组一个标记。<br /><br /> 选项<br /><br /> patterns (type: string array) - 与每个标记匹配的模式的列表。 必需。<br /><br /> preserveOriginal (type: bool) - 如果设置为 true，那么即使其中一个模式匹配，也返回原始标记，默认值：true |  
|[pattern_replace](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceFilter.html)|PatternReplaceTokenFilter|一个标记筛选器，它向流中的每个标记应用模式，并将匹配项替换为指定的替换字符串。<br /><br /> 选项<br /><br /> pattern (type: string) - 必需。<br /><br /> replacement (type: string) - 必需。|  
|[persian_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizationFilter.html)|（仅当有可用的选项时，类型才适用） |为波斯语应用规范化。 |  
|[phonetic](https://lucene.apache.org/core/4_10_3/analyzers-phonetic/org/apache/lucene/analysis/phonetic/package-tree.html)|PhoneticTokenFilter|为拼音匹配项创建标记。<br /><br /> 选项<br /><br /> encoder (type: string) - 要使用的拼音编码器。 允许的值包括：“metaphone”、“doubleMetaphone”、“soundex”、“refinedSoundex”、“caverphone1”、“caverphone2”、“cologne”、“nysiis”、“koelnerPhonetik”、“haasePhonetik”、“beiderMorse”。 默认值：“metaphone”。 默认值为 metaphone。<br /><br /> 有关详细信息，请参阅[编码器](https://commons.apache.org/proper/commons-codec/archives/1.10/apidocs/org/apache/commons/codec/language/package-frame.html)。<br /><br /> replace (type: bool) - 如果编码的标记应替换原始标记，则为 true；如果应将其添加为同义词，则为 false。 默认值为 true。|  
|[porter_stem](https://lucene.apache.org/core/4_10_3/analyzers-common/org/tartarus/snowball/ext/PorterStemmer.html)|（仅当有可用的选项时，类型才适用）  |根据 [Porter 词干分解算法](https://tartarus.org/~martin/PorterStemmer/)转换标记流。 |  
|[reverse](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/reverse/ReverseStringFilter.html)|（仅当有可用的选项时，类型才适用）  |反转标记字符串。 |  
|[scandinavian_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)|（仅当有可用的选项时，类型才适用）  |规范化可互换的斯堪的纳维亚语字符的使用。 |  
|[scandinavian_folding](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)|（仅当有可用的选项时，类型才适用）  |折叠斯堪的纳维亚语字符 åÅäæÄÆ->a 和 öÖøØ->o。 它还排斥双元音 aa、ae、ao、oe 和 oo 的使用，只留下第一个元音。 |  
|[shingle](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/shingle/ShingleFilter.html)|ShingleTokenFilter|创建标记组合作为单个标记。<br /><br /> 选项<br /><br /> maxShingleSize (type: int) - 默认值为 2。<br /><br /> minShingleSize (type: int) - 默认值为 2。<br /><br /> outputUnigrams (type: bool) - 如果为 true，则输出流包含输入标记（单元语法）和瓦形。 默认值为 true。<br /><br /> outputUnigramsIfNoShingles (type: bool) - 如果为 true，则在没有可用瓦形时覆盖 outputUnigrams==false 的行为。 默认值为 false。<br /><br /> tokenSeparator (type: string) - 联接相邻标记以形成瓦形时使用的字符串。 默认值为“ ”。<br /><br /> filterToken (type: string) - 要为每个没有标记的位置插入的字符串。 默认值为“_”。|  
|[snowball](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/snowball/SnowballFilter.html)|SnowballTokenFilter|Snowball 标记筛选器。<br /><br /> 选项<br /><br /> language (type: string) - 允许的值包括：“armenian”、“basque”、“catalan”、“danish”、“dutch”、“english”、“finnish”、“french”、“german”、“german2”、“hungarian”、“italian”、“kp”、“lovins”、“norwegian”、“porter”、“portuguese”、“romanian”、“russian”、“spanish”、“swedish”、“turkish”|  
|[sorani_normalization](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizationFilter.html)|SoraniNormalizationTokenFilter|规范化索拉尼语文本的 Unicode 表示形式。<br /><br /> 选项<br /><br /> 无。|  
|stemmer|StemmerTokenFilter|特定于语言的词干分解筛选器。<br /><br /> 选项<br /><br /> language (type: string) - 允许的值包括： <br /> -   [“arabic”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/ar/ArabicStemmer.html)<br />-   [“armenian”](https://snowballstem.org/algorithms/armenian/stemmer.html)<br />-   [“basque”](https://snowballstem.org/algorithms/basque/stemmer.html)<br />-   [“brazilian”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/br/BrazilianStemmer.html)<br />-   "bulgarian"<br />-   [“catalan”](https://snowballstem.org/algorithms/catalan/stemmer.html)<br />-   [“czech”](https://portal.acm.org/citation.cfm?id=1598600)<br />-   [“danish”](https://snowballstem.org/algorithms/danish/stemmer.html)<br />-   [“dutch”](https://snowballstem.org/algorithms/dutch/stemmer.html)<br />-   [“dutchKp”](https://snowballstem.org/algorithms/kraaij_pohlmann/stemmer.html)<br />-   [“english”](https://snowballstem.org/algorithms/porter/stemmer.html)<br />-   [“lightEnglish”](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf)<br />-   [“minimalEnglish”](https://www.researchgate.net/publication/220433848_How_effective_is_suffixing)<br />-   [“possessiveEnglish”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/en/EnglishPossessiveFilter.html)<br />-   [“porter2”](https://snowballstem.org/algorithms/english/stemmer.html)<br />-   [“lovins”](https://snowballstem.org/algorithms/lovins/stemmer.html)<br />-   [“finnish”](https://snowballstem.org/algorithms/finnish/stemmer.html)<br />-   "lightFinnish"<br />-   [“french”](https://snowballstem.org/algorithms/french/stemmer.html)<br />-   [“lightFrench”](https://dl.acm.org/citation.cfm?id=1141523)<br />-   [“minimalFrench”](https://dl.acm.org/citation.cfm?id=318984)<br />-   "galician"<br />-   "minimalGalician"<br />-   [“german”](https://snowballstem.org/algorithms/german/stemmer.html)<br />-   [“german2”](https://snowballstem.org/algorithms/german2/stemmer.html)<br />-   [“lightGerman”](https://dl.acm.org/citation.cfm?id=1141523)<br />-   "minimalGerman"<br />-   [“greek”](https://sais.se/mthprize/2007/ntais2007.pdf)<br />-   "hindi"<br />-   [“hungarian”](https://snowballstem.org/algorithms/hungarian/stemmer.html)<br />-   [“lightHungarian”](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   [“indonesian”](https://www.illc.uva.nl/Publications/ResearchReports/MoL-2003-02.text.pdf)<br />-   [“irish”](https://snowballstem.org/otherapps/oregan/)<br />-   [“italian”](https://snowballstem.org/algorithms/italian/stemmer.html)<br />-   [“lightItalian”](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   [“sorani”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/ckb/SoraniStemmer.html)<br />-   [“latvian”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/lv/LatvianStemmer.html)<br />-   [“norwegian”](https://snowballstem.org/algorithms/norwegian/stemmer.html)<br />-   [“lightNorwegian”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   [“minimalNorwegian”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   [“lightNynorsk”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   [“minimalNynorsk”](https://lucene.apache.org/core/4_9_0/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   [“portuguese”](https://snowballstem.org/algorithms/portuguese/stemmer.html)<br />-   [“lightPortuguese”](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   [“minimalPortuguese”](https://www.inf.ufrgs.br/~buriol/papers/Orengo_CLEF07.pdf)<br />-   [“portugueseRslp”](https://www.inf.ufrgs.br//~viviane/rslp/index.htm)<br />-   [“romanian”](https://snowballstem.org/otherapps/romanian/)<br />-   [“russian”](https://snowballstem.org/algorithms/russian/stemmer.html)<br />-   [“lightRussian”](https://doc.rero.ch/lm.php?url=1000%2C43%2C4%2C20091209094227-CA%2FDolamic_Ljiljana_-_Indexing_and_Searching_Strategies_for_the_Russian_20091209.pdf)<br />-   [“spanish”](https://snowballstem.org/algorithms/spanish/stemmer.html)<br />-   [“lightSpanish”](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   [“swedish”](https://snowballstem.org/algorithms/swedish/stemmer.html)<br />-   "lightSwedish"<br />-   [“turkish”](https://snowballstem.org/algorithms/turkish/stemmer.html)|  
|[stemmer_override](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/StemmerOverrideFilter.html)|StemmerOverrideTokenFilter|所有字典词干派生形式的词条均标记为关键字，这将阻止链中的向下词干分解。 必须放在任何词干分解筛选器之前。<br /><br /> 选项<br /><br /> rules (type: string array) - 采用“word => stem”格式的词干分解规则，例如“ran => run”。 默认为空列表。  必需。|  
|[stopwords](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/StopFilter.html)|StopwordsTokenFilter|从标记流中删除非索引字。 默认情况下，该筛选器使用预定义的英语非索引字列表。<br /><br /> 选项<br /><br /> stopwords (type: string array) - 非索引字列表。 如果指定了 stopwordsList，则无法指定该选项。<br /><br /> stopwordsList (type: string) - 一个预定义的非索引字列表。 如果指定了 stopwords，则无法指定该选项。 允许值包括：“arabic”、“armenian”、“basque”、“brazilian”、“bulgarian”、“catalan”、“czech”、“danish”、“dutch”、“english”、“finnish”、“french”、“galician”、“german”、“greek”、“hindi”、“hungarian”、“indonesian”、“irish”、“italian”、“latvian”、“norwegian”、“persian”、“portuguese”、“romanian”、“russian”、“sorani”、“spanish”、“swedish”、“thai”、“turkish”，默认值：“english”。 如果指定了 stopwords，则无法指定该选项。 <br /><br /> ignoreCase (type: bool) - 如果为 true，则首先小写所有字词。 默认值为 false。<br /><br /> removeTrailing (type: bool) - 如果为 true，则忽略最后一个搜索词（如果它是一个非索引字）。 默认值为 true。
|[synonym](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/synonym/SynonymFilter.html)|SynonymTokenFilter|匹配标记流中的单个或多个字词同义词。<br /><br /> 选项<br /><br /> synonyms (type: string array) - 必需。 以下两种格式之一的同义词列表：<br /><br /> -incredible, unbelievable, fabulous => amazing - => 符号左侧的所有词条均替换为其右侧的所有词条。<br /><br /> -incredible, unbelievable, fabulous, amazing - 以逗号分隔的等效字词列表。 设置展开选项可更改此列表的解释方式。<br /><br /> ignoreCase (type: bool) - 用于匹配的大小写折叠输入。 默认值为 false。<br /><br /> expand (type: bool) - 如果为 true，则同义词列表中的所有字词（如果未使用 => 表示法）将相互映射。 <br />以下列表：incredible, unbelievable, fabulous, amazing 等效于：incredible, unbelievable, fabulous, amazing => incredible, unbelievable, fabulous, amazing<br /><br />- 如果为 false，则以下列表：incredible, unbelievable, fabulous, amazing 等效于：incredible, unbelievable, fabulous, amazing => incredible。|  
|[trim](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/TrimFilter.html)|（仅当有可用的选项时，类型才适用）  |剪裁标记中的前导和尾随空格。 |  
|[truncate](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/TruncateTokenFilter.html)|TruncateTokenFilter|将词条截断为特定长度。<br /><br /> 选项<br /><br /> length (type: int) - 默认值：300，最大值：300。 必需。|  
|[unique](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/RemoveDuplicatesTokenFilter.html)|UniqueTokenFilter|筛选出与前一个标记具有相同文本的标记。<br /><br /> 选项<br /><br /> onlyOnSamePosition (type: bool) - 如果设置该选项，则仅在同一位置删除重复项。 默认值为 true。|  
|[uppercase](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/UpperCaseFilter.html)|（仅当有可用的选项时，类型才适用）  |将标记文本规范化为大写。 |  
|[word_delimiter](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterFilter.html)|WordDelimiterTokenFilter|将字词拆分为子字，并对子字组执行可选转换。<br /><br /> 选项<br /><br /> generateWordParts (type: bool) - 导致生成字词的各个部分，例如“AzureSearch”变成“Azure”“Search”。 默认值为 true。<br /><br /> generateNumberParts (type: bool) - 导致生成数字子字。 默认值为 true。<br /><br /> catenateWords (type: bool) - 导致运行次数最多的字词部分连接在一起，例如“Azure-Search”变成“AzureSearch”。 默认值为 false。<br /><br /> catenateNumbers (type: bool) - 导致运行次数最多的数字部分连接在一起，例如“1-2”变成“12”。 默认值为 false。<br /><br /> catenateAll (type: bool) - 导致所有子字部分连接在一起，例如“Azure-Search-1”变成“AzureSearch1”。 默认值为 false。<br /><br /> splitOnCaseChange (type: bool) - 如果为 true，则在大小写发生变化时拆分字词，例如“AzureSearch”变成“Azure”“Search”。 默认值为 true。<br /><br /> preserveOriginal - 使原始字词保留并添加到子字列表中。 默认值为 false。<br /><br /> splitOnNumerics (type: bool) - 如果为 true，则在数字处拆分，例如“Azure1Search”变成“Azure”“1”“Search”。 默认值为 true。<br /><br /> stemEnglishPossessive (type: bool) - 导致为每个子字删除尾随的“'s”。 默认值为 true。<br /><br /> protectedWords (type: string array) - 要防止分隔的标记。 默认为空列表。|  

 <sup>1</sup> 标记筛选器类型在代码中始终带有前缀“#Microsoft.Azure.Search”，因此，“ArabicNormalizationTokenFilter”实际上会被指定为“#Microsoft.Azure.Search.ArabicNormalizationTokenFilter”。  为缩小表的宽度，我们删除了该前缀，但请记住在代码中包含该前缀。  

> [!NOTE]
> 需要将自定义分析器配置为不生成超过 300 个字符的标记。 对包含此类标记的文档编制索引时会失败。 若要剪裁或忽略它们，请分别使用 **TruncateTokenFilter** 和 **LengthTokenFilter**。


## <a name="see-also"></a>另请参阅  
 [Azure 搜索服务 REST](https://docs.microsoft.com/rest/api/searchservice/)   
 [Azure 搜索中的分析器 > 示例](search-analyzers.md#examples)    
 [创建索引（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/create-index)  
