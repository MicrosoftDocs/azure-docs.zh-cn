---
title: 文本合并认知搜索技能 - Azure 搜索
description: 将字段集合中的文本合并到合并字段中。 在 Azure 搜索扩充管道中使用此认知技能。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: bbf2e524d626ac17596ded61746c26f20a6caf1b
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021830"
---
#    <a name="text-merge-cognitive-skill"></a>文本合并认知技能

文本合并技能会将字段集合中的文本合并到单个字段中。 

> [!NOTE]
> 此技能未绑定到认知服务 API，你使用它无需付费。 但是，你仍然应该[附加认知服务资源](cognitive-search-attach-cognitive-services.md)，以覆盖**免费**资源选项，该选项限制你每天进行少量的每日扩充。

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.MergeSkill

## <a name="skill-parameters"></a>技能参数

参数区分大小写。

| 参数名称     | 描述 |
|--------------------|-------------|
| insertPreTag  | 每次插入之前要包含的字符串。 默认值为 `" "`。 要忽略空格，请将值设置为 `""`。  |
| insertPostTag | 每次插入后要包含的字符串。 默认值为 `" "`。 要忽略空格，请将值设置为 `""`。  |


##  <a name="sample-input"></a>示例输入
为此技能提供可用输入的 JSON 文档有：

```json
{
  "values": [
    {
      "recordId": "1",
      "data":
      {
        "text": "The brown fox jumps over the dog",
        "itemsToInsert": ["quick", "lazy"],
        "offsets": [3, 28],
      }
    }
  ]
}
```

##  <a name="sample-output"></a>示例输出
此示例显示之前输入的输出，假设将 insertPreTag 设置为 `" "`并将 insertPostTag 设置为 `""`。 

```json
{
  "values": [
    {
      "recordId": "1",
      "data":
      {
        "mergedText": "The quick brown fox jumps over the lazy dog"
      }
    }
  ]
}
```

## <a name="extended-sample-skillset-definition"></a>扩展的示例技能集定义

使用文本合并的一个常见场景是将图像的文本表示形式（OCR 技能中的文本或图像的描述文字）合并到文档的内容字段中。 

以下示例技能使用 OCR 技能从文档中嵌入的图像中提取文本。 接下来，它会创建 merged_text 字段以包含每个图像的原始和 OCRed 文本。 可在[此处](https://docs.microsoft.com/azure/search/cognitive-search-skill-ocr)了解有关 OCR 技能的详细信息。

```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
      "description": "Extract text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": "en",
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name":"text", "source": "/document/content"
        },
        {
          "name": "itemsToInsert", "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", "source": "/document/normalized_images/*/contentOffset" 
        }
      ],
      "outputs": [
        {
          "name": "mergedText", "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```
以上示例假设存在规范化的图像字段。 要获取规范化的图像字段，请将索引器定义中的 imageAction 配置设置为 generateNormalizedImages，如下所示：

```json
{
  //...rest of your indexer definition goes here ...
  "parameters":{
    "configuration":{
        "dataToExtract":"contentAndMetadata",
        "imageAction":"generateNormalizedImages"
    }
  }
}
```

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [创建索引器 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
