---
title: 常见问题解答 (FAQ) - Azure 搜索
description: 获取有关 Microsoft Azure 搜索服务（Microsoft Azure 上的云托管搜索服务）的常见问题的解答。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 08/03/2017
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: c77f26187914b2c6e52426bb2a07303b22ccb2b0
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024000"
---
# <a name="azure-search---frequently-asked-questions-faq"></a>Azure 搜索 - 常见问题解答 (FAQ)

 查找与 Azure 搜索有关的概念、代码和方案的常见问题的答案。

## <a name="platform"></a>平台

### <a name="how-is-azure-search-different-from-full-text-search-in-my-dbms"></a>Azure 搜索与 DBMS 中的全文搜索有何不同？

Azure 搜索支持多个数据源、[针对多种语言的语言分析](https://docs.microsoft.com/rest/api/searchservice/language-support)[对有趣和异常数据输入的自定义分析](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)、通过[计分概要文件](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)控制搜索优先级，以及 typeahead、搜索词突出显示和分面导航等用户体验功能。 它还包含其他功能，如同义词和丰富的查询语法，但通常来说，这些不属于区别性功能。

### <a name="what-is-the-difference-between-azure-search-and-elasticsearch"></a>Azure 搜索和 Elasticsearch 有何不同？

比较搜索技术时，客户经常询问 Azure 搜索 与 Elasticsearch 的具体对比信息。 如果客户为搜索应用程序项目选择 Azure 搜索而非 Elasticsearch，通常是因为我们让关键任务变得更加简单，或他们需要与其他 Microsoft 技术的内部集成：

+ Azure 搜索是完全托管的云服务，提供充足冗余时（2 份副本用于读取访问，3 份副本用于读写访问），拥有 99.9% 的服务级别协议 (SLA)。
+ Microsoft 的[自然语言处理器](https://docs.microsoft.com/rest/api/searchservice/language-support)提供领先的语言分析。  
+ [Azure 搜索索引器](search-indexer-overview.md)可为初始和增量索引抓取多种 Azure 数据源。
+ 如果需要对查询或索引卷的波动快速响应，可使用 Azure 门户中的[滑块控制](search-manage.md#scale-up-or-down)，或运行 [PowerShell 脚本](search-manage-powershell.md)直接绕过分片管理。  
+ [计分和优化功能](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)提供搜索引擎无法单独提供的影响搜索优先级评分的方法。

### <a name="can-i-pause-azure-search-service-and-stop-billing"></a>能否暂停 Azure 搜索服务并停止计费？

无法暂停服务。 创建服务时，计算和存储资源是分配给用户单独使用的。 无法按需发布和回收此类资源。

## <a name="indexing-operations"></a>索引操作

### <a name="backup-and-restore-or-download-and-move-indexes-or-index-snapshots"></a>备份和还原 （或下载和移动）索引或索引快照？

尽管可随时[获取索引定义](https://docs.microsoft.com/rest/api/searchservice/get-index)，但并无索引提取、快照或备份-还原功能用于将云中运行的“已填充”索引下载到本地系统，或将它移动到另一个 Azure 搜索服务。

索引由写入的代码创建和填充，仅在云中的 Azure 搜索上运行。 通常，如果客户希望将一个索引移动到另一个服务，会通过编辑代码以使用新的终结点，然后重新运行索引。 如果希望能够拍摄快照或备份索引，请在 [User Voice](https://feedback.azure.com/forums/263029-azure-search/suggestions/8021610-backup-snapshot-of-index) 上投票。

### <a name="can-i-restore-my-index-or-service-once-it-is-deleted"></a>删除后能否还原索引或服务？

否，不能还原索引或服务。 如果删除 Azure 搜索索引，则该操作是最终的，无法恢复索引。 当删除 Azure 搜索服务时，会永久删除该服务中的所有索引。 此外，如果删除包含一项或多项 Azure 搜索服务的 Azure 资源组，则会永久删除所有服务。  

还原索引、索引器、数据源和技能集等资源需要通过代码重新创建它们。 对于索引，必须对来自外部源的数据重新编制索引。 因此，强烈建议在其他数据存储（如 Azure SQL 数据库或 Cosmos DB）保留原始数据的主控副本或备份。

### <a name="can-i-index-from-sql-database-replicas-applies-to-azure-sql-database-indexershttpsdocsmicrosoftcomazuresearchsearch-howto-connecting-azure-sql-database-to-azure-search-using-indexers"></a>能否从 SQL 数据库副本（适用于 [Azure SQL 数据库索引器](https://docs.microsoft.com/azure/search/search-howto-connecting-azure-sql-database-to-azure-search-using-indexers)）进行索引？

从头开始创建索引时，对使用主要或次要副本作为数据源没有任何限制。 然而，使用增量更新（基于已更改的记录）刷新索引时需要主要副本。 此需求来自于 SQL 数据库，它仅确保主要副本上的更改跟踪。 如果尝试为索引刷新工作负荷使用次要副本，则无法保证获得所有数据。

## <a name="search-operations"></a>搜索操作

### <a name="can-i-search-across-multiple-indexes"></a>能否跨多个索引进行搜索？

不，不支持此操作。 搜索始终限制在单一索引内。

### <a name="can-i-restrict-search-index-access-by-user-identity"></a>可以按用户标识限制搜索索引访问？

可以使用 `search.in()` 筛选器实现[安全筛选器](https://docs.microsoft.com/azure/search/search-security-trimming-for-azure-search)。 使用 [Azure Active Directory(AAD) 等标识管理服务](https://docs.microsoft.com/azure/search/search-security-trimming-for-azure-search-with-aad)可很好地编写筛选器，并基于定义的用户组成员身份裁剪搜索结果。

### <a name="why-are-there-zero-matches-on-terms-i-know-to-be-valid"></a>为什么确定有效的术语没有匹配项？

最常见的情况是不了解每种查询类型支持不同的搜索行为和语言分析级别。 全文搜索是主要的工作负荷，包括将术语分解成词根形式的语言分析阶段。 查询分析的这种特性拓宽了可能的匹配范围，因为标记化的术语能够匹配更多变体。

但是，通配符查询、模糊查询和正则表达式查询的分析方法与常规词或短语查询不同，并且当查询与单词在搜索索引中的分析形式不匹配时可能会导致再次调用性能不佳。 有关查询解析和分析的详细信息，请参阅[查询体系结构](https://docs.microsoft.com/azure/search/search-lucene-query-architecture)。

### <a name="my-wildcard-searches-are-slow"></a>通配符搜索速度较慢。

大多数通配符搜索查询（如前缀、模糊和正则表达式）会使用搜索索引中匹配的词在内部重写。 这一扫描搜索索引的额外处理会增加延迟时间。 此外，广泛搜索查询（例如 `a*`），可能会使用许多词重写，因此速度可能会非常慢。 对于高性能通配符搜索，请考虑定义[自定义分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)。

### <a name="why-is-the-search-rank-a-constant-or-equal-score-of-10-for-every-hit"></a>为什么每个搜索词的搜索优先级是一个常数，或都等于 1.0？

默认情况下，根据[匹配术语的统计属性](search-lucene-query-architecture.md#stage-4-scoring)对搜索结果打分，在结果集中从高到低排序。 但某些查询类型（通配符、前缀、正则表达式）始终会给文档总评分贡献一个常数分数。 这是设计的行为。 Azure 搜索设置一个常数分数后，便可以在结果中包含通过查询扩展找到的匹配项，且不会影响排名。

例如，假设在通配符搜索中输入“tour*”，会产生匹配结果“tours”、“tourettes”和“tourmaline”。 由于这些结果的性质，我们无法合理推断出哪些字词的相关性高于其他字词。 因此，在为通配符、前缀和正则表达式类型的查询结果评分时，我们会忽略字词频率。 建立在不完整输入上的搜索结果获得一个常数分数，以避免可能的意外匹配偏差。

## <a name="design-patterns"></a>设计模式

### <a name="what-is-the-best-approach-for-implementing-localized-search"></a>实现本地化搜索的最佳方法是什么？

涉及到支持相同索引中的不同区域设置（语言）时，大多数客户会选择专用字段而非集合。 通过区域设置特定字段可分配适当的分析器。 例如，将 Microsoft 法语分析器分配给包含法语字符串的字段。 这样也简化了筛选过程。 如果已知在 fr-fr 页面上启动了一个查询，则可将搜索结果限制为该字段。 或者，创建一个[计分概要文件](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)以增加该字段的相关性。 Azure 搜索支持 [50 多种语言分析器](https://docs.microsoft.com/azure/search/search-language-support)，可从中进行选择。

## <a name="next-steps"></a>后续步骤

问题是否与缺少功能相关？ 请在 [User Voice 网站](https://feedback.azure.com/forums/263029-azure-search)上请求该功能。

## <a name="see-also"></a>另请参阅

 [StackOverflow：Azure 搜索](https://stackoverflow.com/questions/tagged/azure-search)   
 [Azure 搜索中全文搜索的工作原理](search-lucene-query-architecture.md)  
 [什么是 Azure 搜索？](search-what-is-azure-search.md)
