---
title: 在认知搜索管道中创建技能集 - Azure 搜索
description: 定义数据提取、自然语言处理或图像分析步骤，从 Azure 搜索使用的数据扩充和提取结构化信息。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 9eedf0be6089764c8111ae81d558f7e65af0a66d
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021788"
---
# <a name="how-to-create-a-skillset-in-an-enrichment-pipeline"></a>如何在扩充管道中创建技能集

认知搜索可提取和扩充数据，使之能够在 Azure 搜索中可供搜索。 我们将提取和扩充步骤称作认知技能，这些技能将合并成索引编制期间所引用的技能集。 可以使用方面的技能[内置技能](cognitive-search-predefined-skills.md)或自定义技能 (请参阅[示例： 创建自定义技能](cognitive-search-create-custom-skill-example.md)有关详细信息)。

本文介绍如何对想要使用的技能创建扩充管道。 技能集将附加到 Azure 搜索[索引器](search-indexer-overview.md)。 本文介绍的管道设计的一个部分是构造技能集本身。 

> [!NOTE]
> 管道设计的另一个部分是指定[下一步骤](#next-step)所述的索引器。 索引器定义包括对技能的引用，以及用于将目标索引中的输入连接到输出的字段映射。

请记住以下要点：

+ 只能为每个索引器创建一个技能集。
+ 技能组合必须具有至少一种技能。
+ 可以创建相同类型的多个技能（例如，图像分析技能的变体）。

## <a name="begin-with-the-end-in-mind"></a>一开始就想到最终结果

建议的初始步骤是确定要从原始数据提取哪些数据，以及如何在搜索解决方案中使用该数据。 创建整个扩充管道的演示有助于确定所需的步骤。

假设我们要处理一系列金融分析师评论。 对于每个文件，我们需要提取公司名称和一般性的评论情绪。 此外，可能还需要编写自定义的扩充器，以便使用必应实体搜索服务来查找有关公司的其他信息，例如，该公司经营哪种业务。 实质上，我们需要提取针对每个文档编制索引的如下所述的信息：

| 记录文本 | 公司 | 情绪 | 公司说明 |
|--------|-----|-----|-----|
|sample-record| ["Microsoft", "LinkedIn"] | 0.99 | ["Microsoft Corporation is an American multinational technology company ..." , "LinkedIn is a business- and employment-oriented social networking..."]

下图演示了一个虚构的扩充管道：

![虚构的扩充管道](media/cognitive-search-defining-skillset/sample-skillset.png "虚构的扩充管道")


对管道包含的内容进行适当的构思后，可以表达用于提供这些步骤的技能集。 在功能上，在将索引器定义上传到 Azure 搜索时，即会表达该技能集。 若要详细了解如何上传索引器，请参阅[索引器文档](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。


在图中，文档破解步骤会自动发生。 实质上，Azure 搜索知道如何打开已知的文件，并创建一个内容字段，其中包含从每个文档中提取的文本。 白框是内置的扩充器，“必应实体搜索”虚线框表示要创建的自定义扩充器。 如图所示，该技能集包含三个技能。

## <a name="skillset-definition-in-rest"></a>REST 中的技能集定义

技能定义为技能数组。 每个技能定义其输入的源，以及生成的输出的名称。 使用[创建技能 REST API](https://docs.microsoft.com/rest/api/searchservice/create-skillset) 可以定义对应于上图的技能集： 

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2019-05-06
api-key: [admin key]
Content-Type: application/json
```

```json
{
  "description": 
  "Extract sentiment from financial records, extract company names, and then find additional information about each company mentioned.",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "organizations"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "Calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
  ]
}
```

## <a name="create-a-skillset"></a>创建技能集

创建技能集时，可以提供说明，使技能集具有自述性。 说明是可选的，但可用于跟踪技能集的用途。 由于技能集是不允许注释的 JSON 文档，因此必须为其使用 `description` 元素。

```json
{
  "description": 
  "This is our first skill set, it extracts sentiment from financial records, extract company names, and then finds additional information about each company mentioned.",
  ...
}
```

技能集的下一个片段是技能数组。 可将每个技能视为扩充的基元。 每个技能在此扩充管道中执行小型任务。 每个技能接受一个输入（或一组输入），并返回一些输出。 下面几节重点介绍如何指定内置和自定义技能，将技能链接在一起，通过输入和输出引用。 输入可以来自源数据或来自另一个技能。 输出可映射到搜索索引中的字段，或用作下游技能的输入。

## <a name="add-built-in-skills"></a>添加内置的技能

让我们看看是内置的第一个技能[实体识别技能](cognitive-search-skill-entity-recognition.md):

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "organizations"
        }
      ]
    }
```

* 具有每个内置的技能`odata.type`， `input`，和`output`属性。 技能特定的属性提供适用于该技能的附加信息。 对于实体识别，`categories` 是一组固定的实体类型中，可由预先训练的模型识别的一个实体。

* 每个技能应包含 ```"context"```。 上下文表示发生操作的级别。 在上面的技能，上下文是整个文档，这意味着实体识别技能调用一次，每个文档。 输出也会在该级别生成。 更具体地说，将生成 ```"organizations"``` 作为 ```"/document"``` 的成员。 在下游技能中，可以使用 ```"/document/organizations"``` 的形式引用此新建信息。  如果未显式设置 ```"context"``` 字段，则默认上下文是文档。

* 技能包含一个名为“text”的输入，其源输入设置为 ```"/document/content"```。 技能 （实体识别） 对*内容*由 Azure blob 索引器创建的每个文档，这是标准字段的字段。 

* 该技能包含一个名为 ```"organizations"``` 的输出。 输出只会在处理期间存在。 若要将此输出链接到下游技能的输入，请以 ```"/document/organizations"``` 的形式引用输出。

* 对于特定的文档，```"/document/organizations"``` 的值是从文本提取的组织数组。 例如：

  ```json
  ["Microsoft", "LinkedIn"]
  ```

在某些情况下，需要单独引用数组的每个元素。 例如，假设我们要将 ```"/document/organizations"``` 的每个元素单独传递给另一个技能（例如自定义的必应实体搜索扩充器）。 可以通过在路径中添加星号，来引用该数组的每个元素：```"/document/organizations/*"``` 

第二个情绪提取技能遵循与第一个扩充器相同的模式。 它采用 ```"/document/content"``` 作为输入，并返回每个内容实例的情绪评分。 由于未显式设置 ```"context"``` 字段，输出 (mySentiment) 现在是 ```"/document"``` 的子级。

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
```

## <a name="add-a-custom-skill"></a>添加自定义技能

回顾自定义必应实体搜索扩充器的结构：

```json
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "This skill calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      }
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
```

此定义是在扩充过程中调用某个 Web API 的[自定义技能](cognitive-search-custom-skill-web-api.md)。 对于实体识别找到的每个组织，此技术的调用 web API 来查找该组织的说明。 扩充引擎会在内部协调处理何时调用 Web API，以及如何流式传输收到的信息。 但是，必须在 JSON 中提供调用此自定义 API 所需的初始化（例如所需的 uri、httpHeaders 和 inputs）。 有关为扩充管道创建自定义 Web API 的指导，请参阅[如何定义自定义接口](cognitive-search-custom-skill-interface.md)。

请注意，“上下文”字段设置为包含星号的 ```"/document/organizations/*"```，这意味着，将对 ```"/document/organizations"``` 下的每个组织调用扩充步骤。 

将为识别到的每个组织生成输出（在本例中为公司说明）。 引用下游步骤中的说明时（例如，在关键短语提取中），应该使用路径 ```"/document/organizations/*/description"``` 执行此操作。 

## <a name="add-structure"></a>添加结构

技能集基于非结构化数据生成结构化信息。 下面是一个示例：

*"In its fourth quarter, Microsoft logged $1.1 billion in revenue from LinkedIn, the social networking company it bought last year.The acquisition enables Microsoft to combine LinkedIn capabilities with its CRM and Office capabilities.Stockholders are excited with the progress so far."*

可能的结果是下图所示的生成结构：

![示例输出结构](media/cognitive-search-defining-skillset/enriched-doc.png "示例输出结构")

到目前为止，仅限内部使用的、 只内存和使用仅在 Azure 搜索索引中已被此结构。 添加了知识应用商店，可以保存搜索以外使用的形状的正。

## <a name="add-a-knowledge-store"></a>添加知识存储

[知识应用商店](knowledge-store-concept-intro.md)是在 Azure 搜索中用于保存丰富的文档的预览功能。 您创建的知识存储由 Azure 存储帐户，是丰富的数据进入其中的存储库。 

知识存储定义添加到技能组合。 有关整个过程的演练，请参阅[如何开始使用知识 store](knowledge-store-howto.md)。

```json
"knowledgeStore": {
  "storageConnectionString": "<an Azure storage connection string>",
  "projections" : [
    {
      "tables": [ ]
    },
    {
      "objects": [
        {
          "storageContainer": "containername",
          "source": "/document/EnrichedShape/",
          "key": "/document/Id"
        }
      ]
    }
  ]
}
```

您可以选择将丰富的文档另存为表，与保留的层次结构关系或 blob 存储中的 JSON 文档。 从任何方面的技能的技能的输出可以作为投影的输入来源。 如果想要的数据投影到一个特定形状，更新[整形程序技能](cognitive-search-skill-shaper.md)现在可以建模为你要使用的复杂类型。 

<a name="next-step"></a>

## <a name="next-steps"></a>后续步骤

熟悉扩充管道和技能集后，请继续阅读[如何在技能集中引用注释](cognitive-search-concept-annotations-syntax.md)或[如何将输出映射到索引中的字段](cognitive-search-output-field-mapping.md)。 
