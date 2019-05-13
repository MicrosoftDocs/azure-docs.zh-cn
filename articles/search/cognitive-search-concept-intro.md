---
title: 认知搜索、数据提取、自然语言 AI 进程 - Azure 搜索
description: 使用认知技能和 AI 算法进行内容提取、自然语言处理 (NLP) 和图像处理，以便在 Azure 搜索索引中创建可搜索的内容。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.devlang: NA
ms.topic: overview
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 8201b4089759fb55fae12820f7495664c502837e
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023617"
---
# <a name="what-is-cognitive-search-in-azure-search"></a>什么是 Azure 搜索中的“认知搜索”？

认知搜索是 Azure 搜索中的 AI 功能，用于从图像、blob 和其他非结构化的数据源中提取文本 - 丰富内容以使其在 Azure 搜索索引中更易于搜索。 通过附加到索引管道的“认知技能”来实现提取和丰富。 以下方式支持 AI 丰富： 

+ “自然语言处理”技能包括[实体识别](cognitive-search-skill-entity-recognition.md)、[语言检测](cognitive-search-skill-language-detection.md)、[关键短语提取](cognitive-search-skill-keyphrases.md)、文本操作和[情绪检测](cognitive-search-skill-sentiment.md)。 通过这些技能，非结构化文本可以假定新窗体，在索引中映射为可搜索和可筛选字段。

+ “图像处理”技能包括 [光学字符识别 (OCR)](cognitive-search-skill-ocr.md) 和[视觉特征](cognitive-search-skill-image-analysis.md)标识，例如面部检测、图像解释、图像识别（名人和地标）或属性（例如颜色或图像方向）。 可以创建图像内容的文本表示形式，并且可使用 Azure 搜索的所有查询功能来搜索。

![认知搜索管道示意图](./media/cognitive-search-intro/cogsearch-architecture.png "认知搜索管道概述")

Azure 搜索中的认知技能基于认知服务 API 中的机器学习模型：[计算机视觉](https://docs.microsoft.com/azure/cognitive-services/computer-vision/)和[文本分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview)。 

数据引入阶段应用了自然语言和图形处理，同时结果会成为 Azure 搜索中可搜索索引中文档撰写的一部分。 数据作为 Azure 数据集的来源，然后使用任意所需的[内置技能](cognitive-search-predefined-skills.md)通过索引管道进行推送。 体系结构可扩展，因此如果内置技能不足，可以创建并附加[自定义技能](cognitive-search-create-custom-skill-example.md)，以集成自定义处理。 示例包括面向特定领域（例如金融、科技出版或医疗）的自定义实体模块或文档分类器。

> [!NOTE]
> 通过增大处理频率、添加更多文档或添加更多 AI 算法来扩大范围时，需要[附加可计费的认知服务资源](cognitive-search-attach-cognitive-services.md)。 调用认知服务中的 API，以及在 Azure 搜索中的文档破解阶段提取图像时，会产生费用。 提取文档中的文本不会产生费用。
>
> 内置技能执行按现有[认知服务即用即付价格](https://azure.microsoft.com/pricing/details/cognitive-services/)计费。 图像提取定价如 [Azure 搜索定价页](https://go.microsoft.com/fwlink/?linkid=2042400)所述。
## <a name="components-of-cognitive-search"></a>认知搜索的组件

认知搜索管道基于 [Azure 搜索索引器](search-indexer-overview.md)，此索引器抓取数据源，并提供端到端索引处理。 技能现已附加到索引器，根据定义的技能集截获并扩充文档。 编制索引后，可以通过搜索请求和 [Azure 搜索支持的所有查询类型](search-query-overview.md)来访问内容。  本部分引导索引器的新手完成这些步骤。

### <a name="step-1-connection-and-document-cracking-phase"></a>步骤 1：连接和文档破解阶段

在管道的开头部分包含非结构化文本或非文本内容（例如图像和扫描的文档 JPEG 文件）。 数据必须存在于可由索引器访问的 Azure 数据存储服务中。 索引器可以“破解”源文档，以从源数据提取文本。

![文档破解阶段](./media/cognitive-search-intro/document-cracking-phase-blowup.png "文档破解")

 支持的源包括 Azure Blob 存储、Azure 表存储、Azure SQL 数据库和 Azure Cosmos DB。 可从以下类型的文件提取基于文本的内容：PDF、Word、PowerPoint、CSV 文件。 有关完整列表，请参阅[支持的格式](search-howto-indexing-azure-blob-storage.md#supported-document-formats)。

### <a name="step-2-cognitive-skills-and-enrichment-phase"></a>步骤 2：认知技能和扩充阶段

扩充通过认知技能实现。这些技能执行原子操作。 例如，从 PDF 提取文本内容后，可以应用实体识别语言检测或关键短语提取，以便在索引中生成原生未在源代码中提供的新字段。 管道中使用的技能的集合统称为技能集。  

![扩充阶段](./media/cognitive-search-intro/enrichment-phase-blowup.png "扩充阶段")

技能集基于你提供的、与该技能集连接的[预定义认知技能](cognitive-search-predefined-skills.md)或[自定义技能](cognitive-search-create-custom-skill-example.md)。 技能集既可以很精简，也可以很复杂，它不仅确定处理的类型，而且还确定运算的顺序。 技能集以及定义为索引器一部分的字段映射全面指定扩充管道。 有关将所有组成部分一起提取的详细信息，请参阅[定义技能集](cognitive-search-defining-skillset.md)。

在内部，管道生成扩充文档的集合。 可以确定要将扩充文档的哪些部分映射到搜索索引中可编制索引的字段。 例如，如果应用了关键短语提取和实体识别技能，则这些新字段将成为扩充文档的部分，并可以映射到索引中的字段。 请参阅[注释](cognitive-search-concept-annotations-syntax.md)详细了解输入/输出的形成。

#### <a name="add-a-knowledgestore-element-to-save-enrichments"></a>添加用于保存扩充的 knowledgeStore 元素

[搜索服务 REST api-version=2019-05-06](search-api-preview.md) 使用 knowledgeStore 定义扩展了技能集，此定义提供了 Azure 存储连接，以及描述了如何存储扩充的投影。 

将知识存储添加到技能集，可以投影除全文搜索以外的方案的扩充表示形式。 有关详细信息，请参阅[什么是知识存储](knowledge-store-concept-intro.md)。

### <a name="step-3-search-index-and-query-based-access"></a>步骤 3：搜索索引和基于查询的访问

完成处理后，便会获得由扩充文档组成的搜索索引，这些文档在 Azure 搜索中可全文搜索。 开发者和用户可以通过[查询索引](search-query-overview.md)来访问管道生成的扩充内容。 

![带搜索图标的索引](./media/cognitive-search-intro/search-phase-blowup.png "带搜索图标的索引")

索引类似于可为 Azure 搜索创建的其他任何对象：可以使用自定义分析器进行补充、调用模糊搜索查询、添加筛选的搜索结果，或试着使用评分配置文件来为搜索结果重新造型。

索引从某个索引架构生成。该架构定义字段、属性，以及附加到特定索引的其他构造，例如评分配置文件和同义词映射。 定义并填充索引后，可以增量方式编制索引，以拾取新的和更新的源文档。 某些修改需要完全重新生成。 在架构设计稳定之前，应使用小型数据集。 有关详细信息，请参阅[如何重新生成索引](search-howto-reindex.md)。

<a name="feature-concepts"></a>

## <a name="key-features-and-concepts"></a>重要功能和概念

| 概念 | 说明| 链接 |
|---------|------------|-------|
| 技能集 | 包含技能集合的顶级命名资源。 技能集是扩充管道。 在索引编制期间索引器会调用它。 | [定义技能集](cognitive-search-defining-skillset.md) |
| 认知技能 | 扩充管道中的原子转换。 通常，它是提取或推断结构的组件，因此增强了我们对输入数据的理解。 输出几乎总是基于文本，处理是自然语言处理，或者从图像输入提取或生成文本的图像处理。 技能的输出可映射到索引中的字段，或用作下游扩充组件的输入。 技能是预定义的、由 Microsoft 提供的或自定义的（由你创建并部署）。 | [预定义技能](cognitive-search-predefined-skills.md) |
| 数据提取 | 涵盖大量处理，但与认知搜索相关，实体识别技能最常用于从不以本机方式提供信息的源中提取数据（即实体）。 | [实体识别技能](cognitive-search-skill-entity-recognition.md)| 
| 图像处理 | 从图像推断文本，例如识别某个地标，或者从图像提取文本。 常见示例包括从扫描的文档 (JPEG) 文件中提取字符的 OCR，或者在包含街道标志的照片中识别街道名称。 | [图像分析技能](cognitive-search-skill-image-analysis.md)或 [OCR 技能](cognitive-search-skill-ocr.md)
| 自然语言处理 | 进行文本处理以提供见解，并提供有关文本输入的信息。 语言检测、情绪分析和关键短语提取是属于自然语言处理的技能。  | [关键短语提取技能](cognitive-search-skill-keyphrases.md)、[语言检测技能](cognitive-search-skill-language-detection.md)、[情绪分析技能](cognitive-search-skill-sentiment.md) |
| 文档破解 | 在索引编制期间从非文本源提取或创建文本内容的过程。 光学字符识别 (OCR) 就是一个例子，但它通常是指索引器从应用程序文件中提取内容时使用的核心索引器功能。 提供源文件位置的数据源，以及提供字段映射的索引器定义都是文档破解中的两个关键因素。 | 请参阅[索引器](search-indexer-overview.md) |
| 造型 | 将文本片段整合到较大的结构，或反之，将较大的文本区块分解成易于管理的大小，以进一步执行下游处理。 | [造型程序技能](cognitive-search-skill-shaper.md)、[文本合并技能](cognitive-search-skill-textmerger.md)、[文本拆分技能](cognitive-search-skill-textsplit.md) |
| 扩充文档 | 过渡性的内部结构不能在代码中直接访问。 扩充的文档在处理期间生成，但搜索索引中只保留最终输出。 字段映射确定要将哪些数据元素添加到索引。 | 请参阅[访问扩充的文档](cognitive-search-tutorial-blob.md#access-enriched-document)。 |
| 索引器 |  一种爬网程序，它从外部数据源提取可搜索的数据和元数据，并根据索引与数据源之间字段到字段的映射填充索引，以进行文档破解。 针对认知搜索扩充，索引器会调用技能集，并包含字段映射，以便将扩充输出关联到索引中的目标字段。 索引器定义包含管道操作的所有说明和引用，运行索引器时会调用管道。 | [索引器](search-indexer-overview.md) |
| 数据源  | 由索引器用来连接 Azure 中受支持类型的外部数据源的对象。 | 请参阅[索引器](search-indexer-overview.md) |
| 索引 | Azure 搜索中的持久化搜索索引，通过定义字段结构和用法的索引架构生成。 | [Azure 搜索中的索引](search-what-is-an-index.md) | 

<a name="where-do-i-start"></a>

## <a name="where-do-i-start"></a>从哪里开始？

**步骤 1：[创建 Azure 搜索资源](search-create-service-portal.md)** 

**步骤 2：亲自体验，尝试一些快速入门操作和示例**

+ [快速入门（门户）](cognitive-search-quickstart-blob.md)
+ [教程（HTTP 请求）](cognitive-search-tutorial-blob.md)
+ [自定义技能示例 (C#)](cognitive-search-create-custom-skill-example.md)

我们建议将免费服务用于学习目的，但请注意，免费事务的数量限制为每天 20 个文档。 若要在一天内同时运行快速入门和教程，请使用较小的文件集（10 个文档），这样就可以同时运行这两个练习。

**步骤 3：查看 API**

可以对请求或 .NET SDK 使用 REST `api-version=2019-05-06`。 

这一步使用 REST API 生成认知搜索解决方案。 只会为认知搜索添加或扩展两个 API。 其他 API 的语法与正式版相同。

| REST API | 说明 |
|-----|-------------|
| [创建数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)  | 标识外部数据源的资源，提供用于创建扩充文档的源数据。  |
| [创建技能集 (api-version=2019-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 在索引编制期间协调扩充管道中所用[预定义技能](cognitive-search-predefined-skills.md)和[自定义认知技能](cognitive-search-custom-skill-interface.md)的使用的资源。 |
| [创建索引](https://docs.microsoft.com/rest/api/searchservice/create-index)  | 表示 Azure 搜索索引的架构。 索引中的字段映射到源数据中的字段，或扩充阶段生成的字段（例如，实体识别创建的组织名称的字段）。 |
| [创建索引器 (api-version=2019-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 定义索引编制期间使用的组件（包括数据源、技能、从源和中间数据结构到目标索引的字段关联，以及索引本身）的资源。 运行索引器会触发数据引入和扩充。 输出是基于索引架构的搜索索引，其中填充有源数据，并通过技能集进行了扩充。  |

**清单：典型工作流**

1. 将 Azure 源数据分解为代表性样本。 编制索引需要花费一定的时间，因此请从较少的有代表性数据集着手，然后随着解决方案的不断成熟，逐渐增加数据集的大小。

1. 在 Azure 搜索中创建[数据源对象](https://docs.microsoft.com/rest/api/searchservice/create-data-source)，以提供连接字符串用于数据检索。

1. 使用扩充步骤创建[技能集](https://docs.microsoft.com/rest/api/searchservice/create-skillset)。

1. 定义[索引架构](https://docs.microsoft.com/rest/api/searchservice/create-index)。 字段集合包含源数据中的字段。 还应该抽出其他字段，以保存扩充期间创建的内容的生成值。

1. 定义引用数据源、技能集和索引的[索引器](https://docs.microsoft.com/rest/api/searchservice/create-skillset)。

1. 在索引器中，添加 outputFieldMappings。 此节将技能集的输出（步骤 3）映射到索引架构中的输入字段（步骤 4）。

1. 发送刚刚创建的“创建索引器”请求（一个 POST 请求，其请求正文包含索引器定义）来表示 Azure 搜索中的索引器。 通过此步骤运行索引器，并调用管道。

1. 运行查询以评估结果，并修改代码以更新技能集、架构或索引器配置。

1. 重新生成管道之前[重置索引器](search-howto-reindex.md)。

有关具体问题的详细信息，请参阅[故障排除提示](cognitive-search-concept-troubleshooting.md)。

## <a name="next-steps"></a>后续步骤

+ [认知搜索文档](cognitive-search-resources-documentation.md)
+ [快速入门：在门户演练中试用认知搜索](cognitive-search-quickstart-blob.md)
+ [教程：了解认知搜索 API](cognitive-search-tutorial-blob.md)
+ [知识存储概述](knowledge-store-concept-intro.md)
+ [知识存储演练](knowledge-store-howto.md)
