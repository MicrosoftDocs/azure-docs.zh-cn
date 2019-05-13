---
title: 如何开始使用知识存储（预览版）- Azure 搜索
description: 了解如何分步将 Azure 搜索 AI 索引管道创建的扩充文档发送到 Azure 存储帐户中的知识存储。 随后，可以在 Azure 搜索和其他应用中查看、整形和使用扩充文档。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.topic: quickstart
ms.date: 05/08/2019
ms.author: heidist
ms.openlocfilehash: d9006e3fcfc9691b9f3eec4b86c545fd3fea9f8a
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471745"
---
# <a name="how-to-get-started-with-knowledge-store"></a>如何开始使用知识存储

[知识存储](knowledge-store-concept-intro.md)是 Azure 搜索中新增的一项预览功能，可保存索引管道中创建的 AI 扩充，以便在其他应用中进行知识挖掘。 还可以使用已保存的扩充来了解和优化 Azure 搜索索引管道。

知识存储是通过技能集进行定义。 对于常规的 Azure 搜索全文搜索方案，技能集旨在提供 AI 扩充，让内容更易于搜索。 对于知识挖掘方案，技能集的作用是创建、填充和存储多个数据结构，用于在其他应用和流程中进行分析或建模。

在本练习中，你将从示例数据、服务和工具入手，了解创建和使用首个知识存储的基本工作流（以技能集定义为重点）。

## <a name="prerequisites"></a>先决条件

本快速入门使用以下服务、工具和数据。 

+ [创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可在本教程中使用免费服务。 

+ [创建一个 Azure 存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)，用于存储示例数据。 知识存储位于 Azure 存储中。 

+ 在 S0 即用即付层[创建认知服务资源](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)，用于广泛访问 AI 扩充中使用的一整套技能。 此资源和你的 Azure 搜索服务需位于同一区域。

+ 使用 [Postman 桌面应用](https://www.getpostman.com/)，将请求发送到 Azure 搜索。

+ [Postman 集合](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Caselaw)，其中包含创建数据源、索引、技能集和索引器的已准备请求。 多个对象定义太长，无法添加到本文中。 必须获取此集合，才能完整查看索引和技能集定义。

+ [Caselaw 示例数据](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/caselaw)，源自 [Caselaw Access Project](https://case.law/bulk/download/)“公共批量数据”下载页。 具体而言，本练习使用首次下载（“阿肯色州”）的前 10 个文档。 对于本练习，我们将包含 10 个文档的示例上传到了 GitHub 中。

## <a name="get-a-key-and-url"></a>获取密钥和 URL

REST 调用需要在每个请求中使用服务 URL 和访问密钥。 搜索服务是使用这二者创建的，因此，如果向订阅添加了 Azure 搜索，则请按以下步骤获取必需信息：

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中获取 URL。 示例终结点可能类似于 `https://mydemo.search.windows.net`。

1. 在“设置” > “密钥”中，获取有关该服务的完全权限的管理员密钥。 有两个可交换的管理员密钥，为保证业务连续性而提供，以防需要滚动一个密钥。 可以在请求中使用主要或辅助密钥来添加、修改和删除对象。

    ![获取 HTTP 终结点和访问密钥](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

所有请求对发送到服务的每个请求都需要 API 密钥。

## <a name="prepare-sample-data"></a>准备示例数据

1. [登录到 Azure 门户](https://portal.azure.com)，导航到你的 Azure 存储帐户，单击“Blob”，然后单击“+ 容器”。

1. [创建一个 Blob 容器](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)用于包含示例数据。 使用容器名称“caselaw-test”。 可将“公共访问级别”设为任何有效值。

1. 创建容器后，将其打开，然后在命令栏中选择“上传”。

   ![在命令栏中上传](media/search-semi-structured-data/upload-command-bar.png "在命令栏中上传")

1. 转到包含 caselaw-sample.json 示例文件的文件夹。 选择此文件，再单击“上传”。


## <a name="set-up-postman"></a>设置 Postman

启动 Postman 并导入 Caselaw Postman 集合。 或者设置一系列 HTTP 请求。 如果不熟悉此工具，请参阅[使用 Postman 探索 Azure 搜索 REST API](search-fiddler.md) 了解详细信息。

+ 本演练中每个调用的请求方法是 **PUT** 或 **POST**。
+ 请求头 (2) 包括以下内容：“Content-type”设置为“application/json”，“api-key”设置为“管理密钥”（“管理密钥”是搜索主密钥的占位符）。 
+ 请求正文是调用的实际内容所在的位置。 

  ![半结构化搜索](media/search-semi-structured-data/postmanoverview.png)

我们将使用 Postman 向搜索服务发出四个 API 调用，以依次创建数据源、索引、技能集和索引器。 数据源包含指向 Azure 存储帐户和 JSON 数据的指针。 搜索服务在导入数据时建立连接。

[创建技能集](#create-skillset)是本演练的重点，它指定了扩充步骤，以及如何将数据暂留在知识存储中。

URL 终结点必须指定 api-version，并且每个调用都应返回“201 已创建”。 用于创建支持知识存储的技能集的预览版 api-version 是 `2019-05-06-Preview`（区分大小写）。

在 REST 客户端中，执行以下 API 调用。

## <a name="create-a-data-source"></a>创建数据源

[创建数据源 API](https://docs.microsoft.com/rest/api/searchservice/create-data-source) 创建 Azure 搜索对象，用于指定要为其编制索引的数据。

此调用的终结点为 `https://[service name].search.windows.net/datasources?api-version=2019-05-06-Preview` 

1. 请将 `[service name]` 替换为搜索服务的名称。 

2. 对于此调用，请求正文必须包含存储帐户连接字符串和 Blob 容器名称。 可以在 Azure 门户中存储帐户的“访问密钥”内找到连接字符串。 

   执行此调用前，请务必先替换请求正文中的连接字符串和 Blob 容器名称。

    ```json
    {
        "name": "caselaw-ds",
        "description": null,
        "type": "azureblob",
        "subtype": null,
        "credentials": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<YOUR-STORAGE-ACCOUNT>;AccountKey=<YOUR-STORAGE-KEY>;EndpointSuffix=core.windows.net"
        },
        "container": {
            "name": "<YOUR-BLOB-CONTAINER-NAME>",
            "query": null
        },
        "dataChangeDetectionPolicy": null,
        "dataDeletionDetectionPolicy": null
    }
    ```

3. 发送请求。 响应应为“201”，并且响应正文应与你提供的请求有效负载看起来几乎完全相同。

    ```json
    {
        "name": "caselaw-ds",
        "description": null,
        "type": "azureblob",
        "subtype": null,
        "credentials": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your storage key>;EndpointSuffix=core.windows.net"
        },
        "container": {
            "name": "<your blob container name>",
            "query": null
        },
        "dataChangeDetectionPolicy": null,
        "dataDeletionDetectionPolicy": null
    }
    ```

## <a name="create-an-index"></a>创建索引
    
第二次调用的是[创建索引 API](https://docs.microsoft.com/rest/api/searchservice/create-data-source)，用于创建可存储所有可搜索数据的 Azure 搜索索引。 索引指定所有字段、参数和属性。

不一定需要索引才能进行知识挖掘，但只有提供索引，才能运行索引器。 

此调用的 URL 为 `https://[service name].search.windows.net/indexes?api-version=2019-05-06-Preview`

1. 请将 `[service name]` 替换为搜索服务的名称。

2. 将 Postman 集合的“创建索引”请求中的索引定义复制到请求正文中。 索引定义有上百行，因太长而无法在本文中打印显示。 

   索引的外壳由以下元素组成。 

   ```json
   {
      "name": "caselaw",
      "defaultScoringProfile": null,
      "fields": [],
      "scoringProfiles": [],
      "corsOptions": null,
      "suggesters": [],
      "analyzers": [],
      "tokenizers": [],
      "tokenFilters": [],
      "charFilters": [],
      "encryptionKey": null
   }
   ```

3. `fields` 集合包含大量索引定义。 其中包括简单字段、含嵌套子结构的[复杂字段](search-howto-complex-data-types.md)和集合。

   在第 302-384 行上，查看 `casebody` 的字段定义。 请注意，如果需要分层表示形式，复杂字段中可能有其他复杂字段。

   ```json
   {
    "name": "casebody",
    "type": "Edm.ComplexType",
    "fields": [
        {
            "name": "status",
            "type": "Edm.String",
            "searchable": true,
            "filterable": true,
            "retrievable": true,
            "sortable": true,
            "facetable": true,
            "key": false,
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "analyzer": null,
            "synonymMaps": []
        },
        {
            "name": "data",
            "type": "Edm.ComplexType",
            "fields": [
                {
                    "name": "head_matter",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": false,
                    "retrievable": true,
                    "sortable": false,
                    "facetable": false,
                    "key": false,
                    "indexAnalyzer": null,
                    "searchAnalyzer": null,
                    "analyzer": null,
                    "synonymMaps": []
                },
                {
                    "name": "opinions",
                    "type": "Collection(Edm.ComplexType)",
                    "fields": [
                        {
                            "name": "author",
                            "type": "Edm.String",
                            "searchable": true,
                            "filterable": true,
                            "retrievable": true,
                            "sortable": false,
                            "facetable": true,
                            "key": false,
                            "indexAnalyzer": null,
                            "searchAnalyzer": null,
                            "analyzer": null,
                            "synonymMaps": []
                        },
                        {
                            "name": "text",
                            "type": "Edm.String",
                            "searchable": true,
                            "filterable": false,
                            "retrievable": true,
                            "sortable": false,
                            "facetable": false,
                            "key": false,
                            "indexAnalyzer": null,
                            "searchAnalyzer": null,
                            "analyzer": null,
                            "synonymMaps": []
                        },
                        {
                            "name": "type",
                            "type": "Edm.String",
                            "searchable": true,
                            "filterable": true,
                            "retrievable": true,
                            "sortable": false,
                            "facetable": true,
                            "key": false,
                            "indexAnalyzer": null,
                            "searchAnalyzer": null,
                            "analyzer": null,
                            "synonymMaps": []
                        }
                    ]
                },
    . . .
   ```

4. 发送请求。 

   响应应为“201”，如下面的示例（展示前几个字段）所示：

    ```json
    {
        "name": "caselaw",
        "defaultScoringProfile": null,
        "fields": [
            {
                "name": "id",
                "type": "Edm.String",
                "searchable": true,
                "filterable": true,
                "retrievable": true,
                "sortable": true,
                "facetable": true,
                "key": true,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            {
                "name": "name",
                "type": "Edm.String",
                "searchable": true,
                "filterable": true,
                "retrievable": true,
                "sortable": true,
                "facetable": true,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
      . . .
    ```

<a name="create-skillset"></a>

## <a name="create-a-skillset-and-knowledge-store"></a>创建技能集和知识存储

[创建技能集 API](https://docs.microsoft.com/rest/api/searchservice/create-skillset) 创建 Azure 搜索对象，用于指定要调用的认知技能、如何将技能链接在一起，以及如何指定知识存储（这也是本演练最重要的部分）。

此调用的终结点为 `https://[service name].search.windows.net/skillsets?api-version=2019-05-06-Preview`

1. 请将 `[service name]` 替换为搜索服务的名称。

2. 将 Postman 集合的“创建技能集”请求中的技能集定义复制到请求正文中。 技能集定义有上百行，因太长而无法在本文中打印显示，但它是本演练的重点。

   技能集的外壳由以下元素组成。 `skills` 集合定义内存中扩充，而 `knowledgeStore` 定义则指定如何存储输出。 `cognitiveServices` 定义是与 AI 扩充引擎的连接。

   ```json
   {
    "name": "caselaw-ss",
    "description": null,
    "skills": [],
    "cognitiveServices": [],
    "knowledgeStore": []
   }
   ```

3. 首先，设置 `cognitiveServices` 和 `knowledgeStore` 密钥以及连接字符串。 在此示例中，这些字符串位于技能集定义后面，靠近请求正文末尾。 使用在 S0 层预配的、位于 Azure 搜索所在的同一区域中的认知服务资源。

    ```json
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "YOUR-SAME-REGION-S0-COGNITIVE-SERVICES-RESOURCE",
        "key": "YOUR-COGNITIVE-SERVICES-KEY"
    },
    "knowledgeStore": {
        "storageConnectionString": "YOUR-STORAGE-ACCOUNT-CONNECTION-STRING",
    ```

3. 查看技能集合，特别是第 85 行和第 170 行上的整形程序技能。 整形程序技能非常重要，因为它汇编要用于知识挖掘的数据结构。 在技能集执行期间，这些结构仅在内存中，但在转到下一步时，你便会了解如何将此输出保存到知识存储中，以供进一步探索。

   下面的代码片段截取自第 217 行。 

    ```json
    "name": "Opinions",
    "source": null,
    "sourceContext": "/document/casebody/data/opinions/*",
    "inputs": [
        {
            "name": "Text",
            "source": "/document/casebody/data/opinions/*/text"
        },
        {
            "name": "Author",
            "source": "/document/casebody/data/opinions/*/author"
        },
        {
            "name": "Entities",
            "source": null,
            "sourceContext": "/document/casebody/data/opinions/*/text/pages/*/entities/*",
            "inputs": [
                {
                    "name": "Entity",
                    "source": "/document/casebody/data/opinions/*/text/pages/*/entities/*/value"
                },
                {
                    "name": "EntityType",
                    "source": "/document/casebody/data/opinions/*/text/pages/*/entities/*/category"
                }
            ]
        }
    ]
   . . .
   ```

3. 查看 `knowledgeStore` 中的 `projections` 元素（从第 262 行开始）。 投影指定知识存储组合。 投影是在表对象对中进行指定，但目前一次只能指定一个。 正如首个投影所示，`tables` 已指定，但 `objects` 未指定。 在第二个投影中，正好相反。

   在 Azure 存储中，表是在你创建的每个表的表存储中进行创建，并且每个对象在 Blob 存储中都有一个容器。

   Blob 对象通常包含扩充的完整表达式。 表通常包含部分扩充，采用按特定用途排列的组合形式。 此示例显示了 Cases 表和 Opinions 表，但未显示 Entities、Attorneys、Judges 和 Parties 等其他表。

    ```json
    "projections": [
        {
            "tables": [
                {
                    "tableName": "Cases",
                    "generatedKeyName": "CaseId",
                    "source": "/document/Case"
                },
                {
                    "tableName": "Opinions",
                    "generatedKeyName": "OpinionId",
                    "source": "/document/Case/OpinionsSnippets/*"
                }
            ],
            "objects": []
        },
        {
            "tables": [],
            "objects": [
                {
                    "storageContainer": "enrichedcases",
                    
                    "source": "/document/CaseFull"
                }
            ]
        }
    ]
    ```

5. 发送请求。 响应应为“201”，如下面的示例（展示响应的第一个部分）所示。

    ```json
    {
    "name": "caselaw-ss",
    "description": null,
    "skills": [
        {
            "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
            "name": "SplitSkill#1",
            "description": null,
            "context": "/document/casebody/data/opinions/*/text",
            "defaultLanguageCode": "en",
            "textSplitMode": "pages",
            "maximumPageLength": 5000,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/casebody/data/opinions/*/text
                }
            ],
            "outputs": [
                {
                    "name": "textItems",
                    "targetName": "pages"
                }
            ]
        },
        . . .
    ```

## <a name="create-and-run-an-indexer"></a>创建并运行索引器

[创建索引器 API](https://docs.microsoft.com/rest/api/searchservice/create-indexer) 创建并立即执行索引器。 到目前为止已创建的所有定义都是通过这一步进行实现。 索引器立即运行，因为它不存在于服务中。 当它存在后，对现有索引器的 POST 调用是一项更新操作。

此调用的终结点为 `https://[service name].search.windows.net/indexers?api-version=2019-05-06-Preview`

1. 请将 `[service name]` 替换为搜索服务的名称。 

2. 对于此调用，请求正文指定索引器名称。 数据源和索引是索引器所必需的。 技能集对索引器是可选的，但对 AI 扩充是必需的。

    ```json
    {
        "name": "caselaw-idxr",
        "description": null,
        "dataSourceName": "caselaw-ds",
        "skillsetName": "caselaw-ss",
        "targetIndexName": "caselaw",
        "disabled": null,
        "schedule": null,
        "parameters": {
            "batchSize": 1,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration": {
                "parsingMode": "jsonLines"
            }
        },
        "fieldMappings": [],
        "outputFieldMappings": [
            {
                "sourceFieldName": "/document/casebody/data/opinions/*/text/pages/*/people/*",
                "targetFieldName": "people",
                "mappingFunction": null
            },
            {
                "sourceFieldName": "/document/casebody/data/opinions/*/text/pages/*/organizations/*",
                "targetFieldName": "orginizations",
                "mappingFunction": null
            },
            {
                "sourceFieldName": "/document/casebody/data/opinions/*/text/pages/*/locations/*",
                "targetFieldName": "locations",
                "mappingFunction": null
            },
            {
                "sourceFieldName": "/document/Case/OpinionsSnippets/*/Entities/*",
                "targetFieldName": "entities",
                "mappingFunction": null
            },
            {
                "sourceFieldName": "/document/casebody/data/opinions/*/text/pages/*/keyPhrases/*",
                "targetFieldName": "keyPhrases",
                "mappingFunction": null
            }
        ]
    }
    ```

3. 发送请求。 响应应为“201”，并且响应正文应与你提供的请求有效负载看起来几乎完全相同（为简单起见，已经过剪裁）。

    ```json
    {
        "name": "caselaw-idxr",
        "description": null,
        "dataSourceName": "caselaw-ds",
        "skillsetName": "caselaw-ss",
        "targetIndexName": "caselaw",
        "disabled": null,
        "schedule": null,
        "parameters": {
            "batchSize": 1,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration": {
                "parsingMode": "jsonLines"
            }
        },
        "fieldMappings": [],
        "outputFieldMappings": [
            {
                "sourceFieldName": "/document/casebody/data/opinions/*/text/pages/*/people/*",
                "targetFieldName": "people",
                "mappingFunction": null
            }
        ]
    }
    ```

## <a name="explore-knowledge-store"></a>浏览知识存储

导入首个文档后，可立即开始浏览。 对于此任务，请使用门户中的[存储资源管理器](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-storage-explorer)。

知识存储已从 Azure 搜索完全分离，认识到这一点很重要。 Azure 搜索索引和知识存储都包含数据表示形式和内容，但除此之外均不相同。 索引用于全文搜索、筛选搜索，以及 Azure 搜索支持的所有方案。 或者，仅使用知识存储更进一步，同时附加其他工具来分析内容。

## <a name="takeaways"></a>要点

至此，已在 Azure 存储中创建首个知识存储，并使用存储资源管理器来查看扩充。 这是处理已存储扩充的基本体验。 

## <a name="next-steps"></a>后续步骤

整形程序技能为你省却一切麻烦，让你能轻松创建可组合成新形状的具体数据表单。 下一步是，查看此技能的参考页，以详细了解它的使用方法。

> [!div class="nextstepaction"]
> [整形程序技能参考](cognitive-search-skill-shaper.md)


<!---
## Keep This

How to convert unformatted JSON into an indented JSON document structure that allows you to quickly identify nested structures. Useful for creating an index that includes complex types.

1. Use Visual Studio Code.
2. Open data.jsonl
--->