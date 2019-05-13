---
title: 认知搜索文档资源 - Azure 搜索
description: 与 Azure 搜索中的认知搜索工作负载相关的文章、教程、示例和博客文章的批注列表。
services: search
manager: cgronlun
author: HeidiSteen
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 225ac86efba8a32b085a5fb7fc41bed48c499eab
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65022165"
---
# <a name="documentation-resources-for-cognitive-search-workloads"></a>认知搜索工作负载的文档资源

认知搜索现已公开发布，是新的扩充层在 Azure 搜索索引中的非文本源和无差别的文本，将其转换成 Azure 搜索中全文可搜索内容中查找导致的延迟信息。

下文是认知搜索的完整文档。

## <a name="getting-started"></a>入门
+ [什么是认知搜索？](cognitive-search-concept-intro.md)
+ [快速入门：在门户中尝试认知搜索](cognitive-search-quickstart-blob.md)
+ [教程：了解认知搜索 API](cognitive-search-tutorial-blob.md)
+ [示例：创建自定义技能](cognitive-search-create-custom-skill-example.md)

## <a name="how-to-guidance"></a>操作说明指南
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [如何在技能集中引用注释](cognitive-search-concept-annotations-syntax.md)
+ [如何将字段映射到索引](cognitive-search-output-field-mapping.md)
+ [处理和提取图像中的信息](cognitive-search-concept-image-scenarios.md)
+ [如何重新生成 Azure 搜索索引](search-howto-reindex.md)
+ [如何定义自定义技能接口](cognitive-search-custom-skill-interface.md)
+ [故障排除提示](cognitive-search-concept-troubleshooting.md)

## <a name="reference"></a>参考

+ [预定义技能](cognitive-search-predefined-skills.md)
  + [Microsoft.Skills.Text.KeyPhraseSkill](cognitive-search-skill-keyphrases.md)
  + [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)
  + [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md)
  + [Microsoft.Skills.Text.MergeSkill](cognitive-search-skill-textmerger.md)
  + [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md)
  + [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)
  + [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md)
  + [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md)
  + [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md)

+ [REST API](https://docs.microsoft.com/rest/api/searchservice/)
  + [创建技能组合 (api 版本 = 2019年-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
  + [Create Indexer (api-version=2019-05-06)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)

## <a name="see-also"></a>另请参阅

+ [Azure 搜索 REST API](https://docs.microsoft.com/rest/api/searchservice/)
+ [Azure 搜索中的索引器](search-indexer-overview.md)
+ [什么是 Azure 搜索？](search-what-is-azure-search.md)
