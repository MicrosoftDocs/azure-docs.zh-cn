---
title: 已弃用的认知技能 - Azure 搜索
description: 此页包含一系列被视为已弃用的认知搜索技能，这些技能在不久的将来将不再获得支持。
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
ms.openlocfilehash: f773cd298c8faaac90b30d88a74e8ddcb51c3afa
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021956"
---
# <a name="deprecated-cognitive-search-skills"></a>已弃用的认知搜索技能

本文档介绍别视为已弃用的认知技能。 对内容使用以下指南：

* 技能名称：将要弃用的技能的名称，映射到 @odata.type 属性。
* 最后一个可用的 API 版本：Azure 搜索公共 API 的最后一个版本，通过它可以创建/更新技能集，其中包含相应的已弃用技能。
* 结束支持：支持的最后一天，这一天过后相应的技能会被视为不受支持的技能。 以前创建的技能集应该仍然可以继续使用，但建议用户不要使用已弃用的技能。
* 建议：通过向前迁移路径来使用支持的技能。 建议用户按照建议来继续接受支持。

## <a name="microsoftskillstextnamedentityrecognitionskill"></a>Microsoft.Skills.Text.NamedEntityRecognitionSkill

### <a name="last-available-api-version"></a>最后一个可用的 API 版本

2019-05-06-Preview

### <a name="end-of-support"></a>结束支持

2019 年 2 月 15 日

### <a name="recommendations"></a>建议 

改用 [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md)。 它提供 NamedEntityRecognitionSkill 的大多数功能，且质量更高。 它还在复杂的输出字段中提供更丰富的信息。

若要迁移到[实体识别技能](cognitive-search-skill-entity-recognition.md)，需对技能定义进行下述一个或多个更改。 可以使用[更新技能集 API](https://docs.microsoft.com/rest/api/searchservice/update-skillset) 更新技能定义。

> [!NOTE]
> 目前不支持作为概念的置信度分数。 `minimumPrecision` 参数存在于 `EntityRecognitionSkill` 上，可供将来使用，并可实现后向兼容。

1. （必需）将 `@odata.type` 从 `"#Microsoft.Skills.Text.NamedEntityRecognitionSkill"` 更改为 `"#Microsoft.Skills.Text.EntityRecognitionSkill"`。

2. （可选）若要使用 `entities` 输出，请改用来自 `EntityRecognitionSkill` 的 `namedEntities` 复杂集合输出。 可以使用技能定义中的 `targetName`，将它映射到名为 `entities` 的注释。

3. （可选）如果不显式指定 `categories`，则除了那些受 `NamedEntityRecognitionSkill` 支持的类别，`EntityRecognitionSkill` 可能还会返回不同类型的类别。 如果此行为不合适，请确保将 `categories` 参数显式设置为 `["Person", "Location", "Organization"]`。

    示例迁移定义

    * 简单迁移

        （之前）NamedEntityRecognition 技能定义
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
        （之后）EntityRecognition 技能定义
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
    
    * 稍微复杂的迁移

        （之前）NamedEntityRecognition 技能定义
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "entities"
            }
            ]
        }
        ```
        （之后）EntityRecognition 技能定义
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person", "Location", "Organization" ],
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "namedEntities",
                "targetName": "entities"
            }
            ]
        }
        ```

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [实体识别技能](cognitive-search-skill-entity-recognition.md)
