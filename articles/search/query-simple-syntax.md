---
title: 简单查询语法 - Azure 搜索
description: Azure 搜索中用于全文搜索查询的简单查询语法的参考文档。
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
author: brjohnstmsft
ms.author: brjohnst
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
ms.openlocfilehash: 75e2d7c493b535c984b0ef61dd9a9fae53aee80a
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024201"
---
# <a name="simple-query-syntax-in-azure-search"></a>Azure 搜索中的简单查询语法
Azure 搜索实现两种基于 Lucene 的查询语言：[简单查询分析器](https://lucene.apache.org/core/4_7_0/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html)和 [Lucene 查询分析器](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html)。 在 Azure 搜索中，简单查询语法排除了模糊/贪婪选项。  

> [!NOTE]  
>  Azure 搜索为更复杂的查询提供替代的 [Lucene 查询语法](query-lucene-syntax.md)。 若要详细了解查询分析体系结构和每种语法的好处，请参阅 [Azure 搜索中全文搜索的工作原理](search-lucene-query-architecture.md)。

## <a name="how-to-invoke-simple-parsing"></a>如何调用简单分析

简单语法为默认语法。 仅当将语法从“完整”重置为“简单”时才需要调用。 若要显式设置语法，请使用 `queryType` 搜索参数。 有效值包括 `simple|full`，其中默认值为 `simple`，`full` 则用于 Lucene。 

## <a name="query-behavior-anomalies"></a>查询行为异常

包含一个或多个词条的任何文本都被视为查询执行的有效起点。 Azure 搜索将匹配包含任何或所有词条的文档，其中包括在分析文本期间发现的任何变体。 

尽管听起来很简单，但 Azure 搜索中的查询执行的一个方面*可能会*产生意外结果，导致搜索结果增加而不是减少，因为更多的词条和运算符被添加到输入字符串中。 这种扩展是否会实际发生取决于是否包含 NOT 运算符，以及组合使用的 `searchMode` 参数设置，该参数设置确定如何根据 AND 或 OR 行为解释 NOT。 在默认值为 `searchMode=Any` 并使用 NOT 运算符的情况下，该运算会作为 OR 操作进行计算，以便 `"New York" NOT Seattle` 返回非 Seattle 的所有城市。  

通常情况下，更有可能在搜索内容的应用程序的用户交互模式中看到这些行为，其中用户更有可能在查询中包含运算符，而不是具有更多内置导航结构的电子商务网站。 有关详细信息，请参阅 [NOT 运算符](#not-operator)。 

## <a name="boolean-operators-and-or-not"></a>布尔运算符 (AND、 OR、 NOT) 

可以在一个查询字符串来生成一组丰富的条件对其找到匹配的文档中嵌入运算符。 

### <a name="and-operator-"></a>AND 运算符 `+`

AND 运算符是一个加号。 例如，`wifi+luxury` 将搜索包含 `wifi` 和 `luxury` 的文档。

### <a name="or-operator-"></a>OR 运算符 `|`

OR 运算符是一个竖条或管状字符。 例如，`wifi | luxury` 将搜索包含 `wifi` 或 `luxury` 或两者的文档。

<a name="not-operator"></a>

### <a name="not-operator--"></a>NOT 运算符 `-`

NOT 运算符是一个减号。 例如，`wifi –luxury` 将搜索包含 `wifi` 词条和/或不包含 `luxury`（和/或由 `searchMode` 控制）的文档。

> [!NOTE]  
>  `searchMode` 选项控制在没有 `+` 或 `|` 运算符的情况下，带有 NOT 运算符的词条是与查询中的其他词条进行 AND 运算还是 OR 运算。 请记住，`searchMode` 可设置为 `any`（默认）或 `all`。 如果使用 `any`，可以以包含更多结果的方式提高查询的查全率，且默认情况下将 `-` 解释为“OR NOT”。 例如，`wifi -luxury` 将匹配包含 `wifi` 词条或不包含 `luxury` 词条的文档。 如果使用 `all`，可以以包含更少结果的方式提高查询的精确度，且默认情况下将 - 解释为“AND NOT”。 例如，`wifi -luxury` 将匹配包含 `wifi` 词条且不包含“luxury”词条的文档。 这对于 `-` 运算符来说可能是更直观的行为。 因此，如果想要优化搜索精确度（而非查全率），*且*用户在搜索中频繁使用 `-` 运算符，则应考虑使用 `searchMode=all` 而不是 `searchMode=any`。

## <a name="suffix-operator"></a>后缀运算符

后缀运算符是一个星号`*`。 例如，`lux*` 将搜索包含以 `lux` 开头的词条的文档（忽略大小写）。  

## <a name="phrase-search-operator"></a>短语搜索运算符

短语运算符用引号将短语`" "`。 例如，`Roach Motel`（没有引号）会以任何顺序在任何位置搜索包含 `Roach` 和/或 `Motel` 的文档，而 `"Roach Motel"`（带引号）只会匹配包含整个短语并按该顺序排列的文档（文本分析仍然适用）。

## <a name="precedence-operator"></a>优先运算符

优先运算符将此字符串用括号`( )`。 例如，`motel+(wifi | luxury)`将搜索包含 motel 一词并的文档`wifi`或`luxury`（或两者）。  

## <a name="escaping-search-operators"></a>转义搜索运算符  

 为了将上述符号实际用于搜索文本，应将反斜杠用作其前缀对它们进行转义。 例如，`luxury\+hotel` 将生成 `luxury+hotel` 词条。 为了让更典型的情况变得简单，此规则有两个不需要进行转义的例外：  

- 仅当 NOT 运算符 `-` 是空格之后的第一个字符时才需要对其进行转义，如果它位于词条中间则不需要对其进行转义。 例如，`wi-fi` 是单一词条；而 GUID（例如 `3352CDD0-EF30-4A2E-A512-3B30AF40F3FD`）被视为单一标记。
- 仅当后缀运算符 `*` 是空格之前的最后一个字符时才需要对其进行转义，如果它位于词条中间则不需要对其进行转义。 例如，`wi*fi` 被视为单一标记。

> [!NOTE]  
>  虽然转义使标记保持在一起，但文本分析可能会将它们拆分，具体取决于分析模式。 有关详细信息，请参阅[语言支持（Azure 搜索服务 REST API）](index-add-language-analyzers.md)。  

## <a name="see-also"></a>另请参阅  

+ [搜索文档（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) 
+ [Lucene 查询语法](query-lucene-syntax.md)
+ [OData 表达式语法](query-odata-filter-orderby-syntax.md) 
