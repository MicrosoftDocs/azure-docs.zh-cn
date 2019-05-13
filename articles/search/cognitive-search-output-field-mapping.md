---
title: 将认知搜索扩充输入字段映射到输出字段 - Azure 搜索
description: 提取并扩充源数据字段，然后映射到 Azure 搜索索引中的输出字段。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 506acee6cd9cd3c50e10f1c45768230564eeaaf1
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65022096"
---
# <a name="how-to-map-enriched-fields-to-a-searchable-index"></a>如何将扩充字段映射到可搜索索引

在本文中，你将了解如何将扩充输入字段映射到可搜索索引中的输出字段。 [定义技能组合](cognitive-search-defining-skillset.md)后，你必须将直接添加值的任何技能的输出字段映射到搜索索引中的给定字段。 将内容从扩充文档移动到索引中必须进行字段映射。


## <a name="use-outputfieldmappings"></a>使用 outputFieldMappings
要映射字段，请按如下所示将 `outputFieldMappings` 添加到索引器定义：

```http
PUT https://[servicename].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
api-key: [admin key]
Content-Type: application/json
```

请求正文的结构如下：

```json
{
    "name": "myIndexer",
    "dataSourceName": "myDataSource",
    "targetIndexName": "myIndex",
    "skillsetName": "myFirstSkillSet",
    "fieldMappings": [
        {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "id",
            "mappingFunction": {
                "name": "base64Encode"
            }
        }
    ],
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/content/organizations/*/description",
            "targetFieldName": "descriptions"
        },
        {
            "sourceFieldName": "/document/content/organizations",
            "targetFieldName": "orgNames"
        },
        {
            "sourceFieldName": "/document/content/sentiment",
            "targetFieldName": "sentiment"
        }
    ]
}
```
对于每个输出字段映射，请设置扩充字段的名称 (sourceFieldName)，以及如索引中引用的字段名称 (targetFieldName)。

sourceFieldName 中的路径可以表示一个元素或多个元素。 在上述示例中，```/document/content/sentiment``` 表示单个数字值，而 ```/document/content/organizations/*/description``` 表示多个组织说明。 如果有多个元素，它们将“平展”成包含每个元素的数组。 更具体地说，对于 ```/document/content/organizations/*/description``` 示例，“描述”字段中的数据在编制索引之前将类似于说明的平面数组：

```
 ["Microsoft is a company in Seattle","LinkedIn's office is in San Francisco"]
```
## <a name="next-steps"></a>后续步骤
将扩充字段映射到可搜索字段后，你可以设置每个可搜索字段中的字段属性[作为索引定义的一部分](search-what-is-an-index.md)。

有关字段映射的详细信息，请参阅 [Azure 搜索索引器中的字段映射](search-indexer-field-mappings.md)。
