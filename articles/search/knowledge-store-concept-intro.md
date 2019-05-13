---
title: 知识存储简介和概述 - Azure 搜索
description: 将扩充文档发送到 Azure 存储，随后可以在 Azure 搜索和其他应用中查看、整形和使用扩充文档。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.devlang: NA
ms.topic: overview
ms.date: 05/02/2019
ms.author: heidist
ms.openlocfilehash: 3000016de934aaa3faab96821f9747ea4b571ef7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65030085"
---
# <a name="what-is-knowledge-store-in-azure-search"></a>什么是 Azure 搜索中的知识存储？

知识存储是 Azure 搜索的一项可选功能，目前是公共预览版，可保存基于 AI 的索引管道[（认知搜索）](cognitive-search-concept-intro.md)创建的扩充文档和元数据。 知识存储由配置为管道一部分的 Azure 存储帐户提供技术支持。 启用后，搜索服务使用此存储帐户来缓存每个扩充文档的表示形式。 

如果过去使用过认知搜索，你已知道技能集可用于通过一系列扩充来迁移文档。 结果可以是 Azure 搜索索引，也可以是知识存储中的投影（此预览版中新增的）。

投影是构建数据结构以供用于下游应用的机制。 可以使用专为 Azure 存储打造的[存储资源管理器](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows)，或连接到 Azure 存储的任何应用，这为使用扩充文档开辟了新的可能性。 一些示例包括，数据科学管道和自定义分析。

![“管道中的知识存储”图](./media/knowledge-store-concept-intro/annotationstore_sans_internalcache.png "“管道中的知识存储”图")

若要使用知识存储，请将 `knowledgeStore` 元素添加到在索引管道中定义步进式操作的技能集。 在执行期间，Azure 搜索在 Azure 存储帐户中创建空间，并使用管道创建的定义和内容来填充空间。

## <a name="benefits-of-knowledge-store"></a>知识存储的优势

知识存储提供了结构、上下文和实际内容，这些收集自非结构化和半结构化数据文件（如 Blob）、已经过分析的图像文件，或甚至收集自已整形到新表单中的结构化数据。 在为此预览版撰写的[分步演练](knowledge-store-howto.md)中，你可以获得第一手信息：了解密集型 JSON 文档如何分区成子结构、如何重新构造成新结构，以及如何可用于下游进程（如机器学习和数据科学工作负荷）。

尽管了解基于 AI 的索引管道可以生成什么十分有用，但知识存储的真正强大之处是能够整形数据。 你可以从基本技能集入手，然后循环访问它以添加越来越多的结构级别，这样就能将它们合并成新结构，可用于除 Azure 搜索以外的其他应用。

知识存储的优势已枚举如下：

+ 在除搜索以外的[分析和报表工具](#tools-and-apps)中使用扩充文档。 含 Power Query 的 Power BI 就是个极具吸引力的选择，但只要是能连接到 Azure 存储，任何工具或应用都可以从你创建的知识存储中拉取文档。

+ 优化 AI 索引管道，同时调试步骤和技能集定义。 知识存储展示 AI 索引管道中的技能集定义的结果。 这些结果可用于设计更好的技能集，因为你可以清楚地看到扩充是什么样的。 可以使用 Azure 存储中的[存储资源管理器](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows)来查看知识存储的内容。

+ 将数据整形到新表单中。 整形在技能集中编码化，但重点是技能集现在可以提供此功能。 Azure 搜索中的[整形程序技能](cognitive-search-skill-shaper.md)已扩展为包含此任务。 通过整形，可以定义与数据预期用途保持一致的投影，同时保留关系。

> [!Note]
> 不熟悉基于 AI 的索引（使用认知服务）？ Azure 搜索与认知服务视觉和语言功能集成，以对图像文件使用光学字符识别 (OCR)、对文本文件使用实体识别和关键短语提取等来提取和扩充源数据。 有关详细信息，请参阅[什么是认知搜索？](cognitive-search-concept-intro.md)。

## <a name="create-a-knowledge-store"></a>创建知识存储

知识存储是技能集定义的一部分。 在此预览版中，必须使用 REST API `api-version=2019-05-06-Preview` 或门户中的“导入数据”向导，才能创建知识存储。

以下 JSON 指定 `knowledgeStore`，它是技能集的一部分，可由索引器（未显示）调用。 `knowledgeStore` 中投影的规范决定表或对象是否是在 Azure 存储中进行创建。

如果你已熟悉基于 AI 的索引，便知道技能集定义决定每个扩充文档的创建、组织和物质。

```json
{
  "name": "my-new-skillset",
  "description": 
  "Example showing knowledgeStore placement, supported in api-version=2019-05-06-Preview. You need at least one skill, most likely a Shaper skill if you are modulating data structures.",
  "skills":
  [
    {
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document/content/phrases/*",
    "inputs": [
        {
        "name": "text",
        "source": "/document/content/phrases/*"
        },
        {
        "name": "sentiment",
        "source": "/document/content/phrases/*/sentiment"
        }
    ],
    "outputs": [
        {
        "name": "output",
        "targetName": "analyzedText"
        }
    ]
    },
  ],
  "cognitiveServices": 
    {
    "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
    "description": "mycogsvcs resource in West US 2",
    "key": "<your key goes here>"
    },
  "knowledgeStore": { 
    "storageConnectionString": "<your connection string goes here>", 
    "projections": [ 
        { 
            "tables": [  
            { "tableName": "Reviews", "generatedKeyName": "ReviewId", "source": "/document/Review" , "sourceContext": null, "inputs": []}, 
            { "tableName": "Sentences", "generatedKeyName": "SentenceId", "source": "/document/Review/Sentences/*", "sourceContext": null, "inputs": []}, 
            { "tableName": "KeyPhrases", "generatedKeyName": "KeyPhraseId", "source": "/document/Review/Sentences/*/KeyPhrases", "sourceContext": null, "inputs": []}, 
            { "tableName": "Entities", "generatedKeyName": "EntityId", "source": "/document/Review/Sentences/*/Entities/*" ,"sourceContext": null, "inputs": []} 

            ], 
            "objects": [ 
                { 
                "storageContainer": "Reviews", 
                "format": "json", 
                "source": "/document/Review", 
                "key": "/document/Review/Id" 
                } 
            ]      
        }    
    ]     
    } 
}
```

## <a name="components-backing-a-knowledge-store"></a>为知识存储提供技术支持的组件

若要创建知识存储，需要以下服务和项目。

### <a name="1---source-data"></a>1 - 源数据

要扩充的数据或文档必须存在于 Azure 搜索索引器支持的 Azure 数据源中： 

* [Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

* [Azure Cosmos DB](search-howto-index-cosmosdb.md)

* [Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)

[Azure 表存储](search-howto-indexing-azure-tables.md)可用于知识存储中的出站数据，但无法用作传输到基于 AI 的索引管道的入站数据的资源。

### <a name="2---azure-search-service"></a>2 - Azure 搜索服务

你还需要 Azure 搜索服务和 REST API，以创建和配置用于数据扩充的对象。 用于创建知识存储的 REST API 是 `api-version=2019-05-06-Preview`。

Azure 搜索提供了索引器功能，索引器用于端到端驱动整个过程，从而在 Azure 存储中生成持久化扩充文档。 索引器使用数据源、索引和技能集，所有这些都是创建和填充知识存储所必需的。

| 对象 | REST API | 说明 |
|--------|----------|-------------|
| 数据源 | [创建数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)  | 标识外部 Azure 数据源的资源，数据源提供用于创建扩充文档的源数据。  |
| 技能集 | [创建技能集 (api-version=2019-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 在编制索引期间协调扩充管道中所用[内置技能](cognitive-search-predefined-skills.md)和[自定义认知技能](cognitive-search-custom-skill-interface.md)的资源。 |
| index | [创建索引](https://docs.microsoft.com/rest/api/searchservice/create-index)  | 表示 Azure 搜索索引的架构。 索引中的字段映射到源数据中的字段，或扩充阶段生成的字段（例如，实体识别创建的组织名称的字段）。 |
| 索引器 | [创建索引器 (api-version=2019-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 定义索引编制期间使用的组件（包括数据源、技能、从源和中间数据结构到目标索引的字段关联，以及索引本身）的资源。 运行索引器会触发数据引入和扩充。 输出是基于索引架构的搜索索引，其中填充有源数据，并通过技能集进行了扩充。  |

### <a name="3---cognitive-services"></a>3 - 认知服务

技能集中指定的扩充基于认知服务中的计算机视觉和语言功能。 在编制索引期间，通过技能集利用认知服务功能。 技能集是各种技能的组合，技能限于特定计算机视觉和语言功能范围内。 若要集成认知服务，请[将认知服务资源附加](cognitive-search-attach-cognitive-services.md)到技能集。

### <a name="4---storage-account"></a>4 - 存储帐户

在 Azure 存储帐户下，Azure 搜索创建 Blob 容器或表，具体视你如何配置技能集而定。 如果数据源自 Azure Blob 存储或表存储，可能已经完成创建存储帐户。 否则，需要创建 Azure 存储帐户。 Azure 存储中的表和对象包含基于 AI 的索引管道创建的扩充文档。

存储帐户是在技能集中进行指定。 在 `api-version=2019-05-06-Preview` 中，技能集定义包括知识存储定义，这样你就能提供帐户信息了。

<a name="tools-and-apps"></a>

### <a name="5---access-and-consume"></a>5 - 访问和使用

只要扩充存在于存储中，连接到 Azure Blob 存储或表存储的任何工具或技术，都可用于浏览、分析或使用内容。 请从以下列表入手：

+ [存储资源管理器](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows)：用于查看扩充文档的结构和内容。 将这视为用于查看知识存储内容的基线工具。

+ [含 Power Query 的 Power BI](https://support.office.com/article/connect-to-microsoft-azure-blob-storage-power-query-f8165faa-4589-47b1-86b6-7015b330d13e)：用于自然语言查询，或使用报表和分析工具（若有数值数据的话）。

+ [Azure 数据工厂](https://docs.microsoft.com/azure/data-factory/)：用于进一步操作。

+ Azure 搜索索引：用于对已使用[认知搜索](cognitive-search-concept-intro.md)编制索引的内容进行全文搜索。

## <a name="document-persistence"></a>文档暂留

在存储帐户中，扩充可以表示为 Azure 表存储中的表，也可以表示为 Azure Blob 存储中的对象。 回顾一下，扩充一旦存储，就可以用作源，将数据加载到其他数据库和工具中。

+ 若要表格表单中的数据采用架构识别的表示形式，表存储就非常有用。 若要通过新方式整形或重新组合元素，表存储能够提供必要的粒度。

+ Blob 存储为每个文档都创建一个全包型 JSON 表示形式。 可以在一个技能集中同时使用两个存储选项，从而获取一整套表达式。

+ Azure 搜索将内容暂留在索引中。 如果方案与搜索无关（例如，如果目标是在另一个工具中分析），可以删除管道创建的索引。 不过，也可以保留索引，并将内置工具（如[搜索资源管理器](search-explorer.md)）用作与内容交互的第三个媒体（位于存储资源管理器和分析应用后面）。

除了包含文档内容外，扩充文档还包含生成扩充的技能集版本的元数据。  

## <a name="inside-a-knowledge-store"></a>知识存储内部

知识存储由注释缓存和投影组成。 服务在内部使用缓存来缓存技能和跟踪更改结果。 投影定义与预期用途匹配的扩充的架构和结构。 每个知识存储只能有一个缓存，但可以有多个投影。 

缓存始终是 Blob 容器，但投影可以表达为表或对象：

+ 作为对象，投影映射到 Blob 存储，在存储内投影保存到容器中，其中包含数据科学管道等方案的对象或分层表示形式（采用 JSON 格式）。

+ 作为表，投影映射到表存储。 表格表示形式为数据分析或导出为数据帧以用于机器学习等方案保留关系。 然后，可以将扩充投影轻松导入其他数据存储中。 

为了适应组织中的各种顾客，可以在知识存储中创建多个投影。 开发人员可能需要访问扩充文档的完整 JSON 表示形式，而数据科学家或分析师可能则需要技能集塑形的具体或模块化数据结构。

例如，如果扩充过程的目标之一是还要创建用于定型模型的数据集，将数据投影到对象存储是使用数据科学管道中数据的一种方法。 或者，若要根据扩充文档创建快速 Power BI 仪表板，使用表格投影的效果会很好。

<!---
## Data lifecycle and billing

Each time you run the indexer, the cache in Azure storage is updated if the skillset definition or underlying source data has changed. As input documents are edited or deleted, changes are propagated through the annotation cache to the projections, ensuring that your projected data is a current representation of your inputs at the end of the indexer run. 

Generally speaking, pipeline processing can be an all-or-nothing operation, but Azure Search can process incremental changes, which saves you time and money.

If a document is new or updated, all skills are run. If only the skillset changes, reprocessing is scoped to just those skills and documents affected by your edit.

### Changes to a skillset
Suppose that you have a pipeline composed of multiple skills, operating over a large body of static data (for example, scanned documents), that takes 8 hours and costs $200 to create the knowledge store. Now suppose you need to tweak one of the skills in the skillset. Rather than starting over, Azure Search can determine which skill is affected, and reprocess only that skill. Cached data and projections that are unaffected by the change remain intact in the knowledge store.

### Changes in the data
Scenarios can vary considerably, but let's suppose instead of static data, you have volatile data that changes between indexer invocations. Given no changes to the skillset, you are charged for processing the delta of new and modified document. The timestamp information varies by data source, but for illustration, in a Blob container, Azure Search looks at the `lastmodified` date to determine which blobs need to be ingested.

> [!Note]
> While you can edit the data in the projections, any edits will be overwritten on the next pipeline invocation, assuming the document in source data is updated. 

### Deletions

Although Azure Search creates and updates structures and content in Azure storage, it does not delete them. Projections and cached documents continue to exist even when the skillset is deleted. As the owner of the storage account, you should delete a projection if it is no longer needed. 

### Tips for development

+ Start small with a representative sample of your data as you make significant changes to skillset composition. As your design finalizes, you can slowly add more data during later-stage development, and then roll in the entire data set when you are comfortable with the pipeline composition.

+ Retain control over indexer invocation. Indexers can run on a schedule, which is helpful for solutions that are rolled into production, but less helpful if you are actively developing your pipeline. During development, avoid schedules so that you don’t lose track of cache or projection state. Once your solution is in production and skillset composition is static, you can put the indexer on a schedule to pick up routine changes in the external source data. 

-->

## <a name="where-do-i-start"></a>从哪里开始？

建议将免费服务用于学习目的，但请注意，免费事务数限制为每个订阅每天 20 个文档。

如果使用多个服务，请在同一区域中创建所有服务，以实现最佳性能，并最大限度地降低成本。 无需为传输到同一区域中其他服务的入站数据或出站数据使用的带宽支付费用。

**步骤 1：[创建 Azure 搜索资源](search-create-service-portal.md)** 

**步骤 2：[创建 Azure 存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal)** 

**步骤 3：[创建认知服务资源](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)** 

**步骤 4：[开始使用门户](cognitive-search-quickstart-blob.md) - 或 - [通过 REST 和 Postman 开始使用示例数据](knowledge-store-howto.md)** 

可以使用 REST `api-version=2019-05-06-Preview` 来构造包含知识存储的基于 AI 的管道。 在最新预览版 API 中，技能集对象提供 `knowledgeStore` 定义。

## <a name="takeaways"></a>要点

知识存储具有各种优势，包括但不限于，能够在除搜索以外的方案中使用扩充文档、控制成本，以及管理扩充过程中的偏差。 只需将存储帐户添加到技能集，并使用更新后的表达式语言（如[如何开始使用知识存储](knowledge-store-howto.md)中所述），即可使用所有这些功能。 

## <a name="next-steps"></a>后续步骤

创建扩充文档的最简单方法是，使用“导入数据”向导。

> [!div class="nextstepaction"]
> [快速入门：在门户演练中试用认知搜索](cognitive-search-quickstart-blob.md)
