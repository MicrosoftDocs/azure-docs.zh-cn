---
title: 将认知服务资源与技能集联系起来 - Azure 搜索
description: 说明如何将认知服务一体化订阅附加到 Azure 搜索中的认知扩充管道。
manager: cgronlun
author: LuisCabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: b979609374afbd11bde0e15ce540e8930315482f
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472477"
---
# <a name="attach-a-cognitive-services-resource-with-a-skillset-in-azure-search"></a>将认知服务资源与 Azure 搜索中的技能集联系起来 

AI 算法驱动器[认知索引管道](cognitive-search-concept-intro.md)用于处理 Azure 搜索中的非结构化的数据。 这些算法基于[认知服务资源](https://azure.microsoft.com/services/cognitive-services/)，包括[计算机视觉](https://azure.microsoft.com/services/cognitive-services/computer-vision/)（用于图像分析和光学字符识别 (OCR)），以及[文本分析](https://azure.microsoft.com/services/cognitive-services/text-analytics/)（用于实体识别、关键短语提取和其他扩充操作）。

可以免费扩充有限数量的文档，或者附加付费的认知服务资源来处理更大、更频繁的工作负荷。 本文介绍如何将认知服务资源关联到认知技能集，以便在 [Azure 搜索索引编制](search-what-is-an-index.md)期间扩充数据。

如果管道中包含与认知服务 API 无关的技能，则你还应附加认知服务资源。 这样做会替代仅允许你每天执行少量扩充操作的“免费”资源。 未绑定到认知服务 API 的技能不收费。 这些技能包括：[自定义技能](cognitive-search-create-custom-skill-example.md)、[文本合并器](cognitive-search-skill-textmerger.md)、[文本拆分器](cognitive-search-skill-textsplit.md)和[整形程序](cognitive-search-skill-shaper.md)。

> [!NOTE]
> 通过增大处理频率、添加更多文档或添加更多 AI 算法来扩大范围时，需要附加可计费的认知服务资源。 调用认知服务中的 API，以及在 Azure 搜索中的文档破解阶段提取图像时，会产生费用。 提取文档中的文本不会产生费用。
>
> 内置技能执行按现有[认知服务即用即付价格](https://azure.microsoft.com/pricing/details/cognitive-services/)计费。 图像提取定价如 [Azure 搜索定价页](https://go.microsoft.com/fwlink/?linkid=2042400)所述。

## <a name="use-free-resources"></a>使用免费资源

可以使用有限的免费处理选项来完成认知搜索教程和快速入门练习。 

**免费 （有限正）** 被限制为每一天，每个订阅 20 个文档。

1. 打开“导入数据”向导。

   ![导入数据命令](media/search-get-started-portal/import-data-cmd2.png "导入数据命令")

1. 选择数据源，然后转到“添加认知搜索(可选)”。 有关此向导的分步演练，请参阅[使用门户工具进行导入、索引编制和查询](search-get-started-portal.md)。

1. 展开“附加认知服务”，然后选择“免费(有限扩充)”。

   ![展开的“附加认知服务”部分](./media/cognitive-search-attach-cognitive-services/attach1.png "展开的“附加认知服务”")

转到下一步骤“添加扩充”。 有关门户中提供的技能的说明，请参阅认知搜索快速入门中的[步骤 2：添加认知技能](cognitive-search-quickstart-blob.md#create-the-enrichment-pipeline)。

## <a name="use-billable-resources"></a>使用付费资源

对于每日执行 20 项以上扩充操作的工作负荷，你需要附加收费的认知服务资源。 

只有调用认知服务 API 的技能才收费。 对于不基于 API 的技能，例如[自定义技能](cognitive-search-create-custom-skill-example.md)、[文本合并器](cognitive-search-skill-textmerger.md)、[文本拆分器](cognitive-search-skill-textsplit.md)和[整形程序](cognitive-search-skill-shaper.md)，则不收费。

1. 打开**导入数据**向导中，选择数据源，并继续**添加认知搜索 （可选）**。 

1. 展开**附加认知服务**，然后选择**创建新的认知服务资源**。 一个新选项卡将打开，以便您可以创建资源。 

   ![创建认知服务资源](./media/cognitive-search-attach-cognitive-services/cog-services-create.png "创建认知服务资源")

1. 在位置中，选择 Azure 搜索所在的同一区域。 这是出于性能原因，必需的但也使无效出站带宽费用，跨区域。

1. 在定价层中，选择**S0**若要获取的认知服务功能，包括返回 Azure 搜索使用的预定义的技能的视觉和语言功能的多功能一体集合。 

   S0 层上，您可以找到费率针对特定工作负荷上[认知服务定价页面](https://azure.microsoft.com/pricing/details/cognitive-services/)。
  
   + 在中**选择产品/服务**，请确保*认知服务*处于选中状态。
   + 在语言功能，针对的费率*文本分析标准*适用于 AI 的索引。 
   + 在下，视觉功能，针对的费率*计算机视觉 S1*应用。

1. 单击“创建”预配新的认知服务资源。 

1. 返回到包含“导入数据”向导的上一选项卡。 单击“刷新”显示该认知服务资源，然后选择该资源。

   ![选择的认知服务资源](./media/cognitive-search-attach-cognitive-services/attach2.png "选择的认知服务资源")

1. 展开“添加扩充”部分，选择要对数据运行的特定认知技能，然后继续完成余下的流程。 有关门户中提供的技能的说明，请参阅认知搜索快速入门中的[步骤 2：添加认知技能](cognitive-search-quickstart-blob.md#create-the-enrichment-pipeline)。

## <a name="attach-an-existing-skillset-to-a-cognitive-services-resource"></a>将现有技能集附加到认知服务资源

如果你有现有的技能集，可将其附加到新的或不同的认知服务资源。

1. 在“服务概述”页上，单击“技能集”。

   ![“技能集”选项卡](./media/cognitive-search-attach-cognitive-services/attach-existing1.png "“技能集”选项卡")

1. 单击技能集的名称，然后选择现有资源或新建资源。 单击“确定”以确认所做的更改。 

   ![技能集资源列表](./media/cognitive-search-attach-cognitive-services/attach-existing2.png "技能集资源列表")

如前所述，“免费(有限扩充)”选项限制为每日 20 个文档，“创建新的认知服务资源”用于预配新的付费资源。 如果创建了新资源，请选择“刷新”以刷新认知服务资源的列表，然后选择该资源。

## <a name="attach-cognitive-services-programmatically"></a>以编程方式附加认知服务

以编程方式定义技能集时，请将 `cognitiveServices` 节添加到该技能集。 在该节中，包含要与该技能集关联的认知服务资源的键。 如前所述，该资源必须位于 Azure 搜索所在的同一区域。 另外请包含 `@odata.type`，并将其设置为 `#Microsoft.Azure.Search.CognitiveServicesByKey`。 

以下示例演示了此模式。 请注意定义底部的 cognitiveServices 节

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2019-05-06
api-key: [admin key]
Content-Type: application/json
```
```json
{
    "name": "skillset name",
    "skills": 
    [
      {
        "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
        "categories": [ "Organization" ],
        "defaultLanguageCode": "en",
        "inputs": [
          {
            "name": "text", "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "organizations", "targetName": "organizations"
          }
        ]
      }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "mycogsvcs",
        "key": "<your key goes here>"
    }
}
```

## <a name="example-estimate-costs"></a>示例：估算成本

若要估算认知搜索索引编制的相关成本，请先构想一下文档的大致结构，以便能够得出一些数字。 例如，使用以下似近值估算成本：

+ 1000 个 PDF
+ 每个 PDF 有 6 个页面
+ 每个页面包含一个图像（共 6000 个图像）
+ 每个页面包含 3000 个字符

假设管道包含的图像和文本提取，光学字符识别 (OCR) 的映像，与每个 PDF 文档破解和实体识别的组织。 

在此练习中，我们将为每笔交易应用最贵的价格。 考虑到阶梯定价，实际成本可能更低。 请参阅[认知服务定价](https://azure.microsoft.com/pricing/details/cognitive-services)。

1. 破解包含文本和图像内容的文档时，文本提取目前是免费的。 对于 6000 个图像，假设每提取 1000 个图像需要 $1，则此步骤的成本是 $6.00。

2. 对于英语环境中 6000 个图像的 OCR，OCR 认知技能使用最佳算法 (DescribeText)。 假设每分析 1,000 个图像的成本为 $2.50，则这一步需要支付 $15.00。

3. 提取实体时，每页总共有 3 个文本记录。 每条记录包含 1,000 个字符。 每页 3 个文本记录 * 6,000 页 = 18,000 个文本记录。 假设 1000 个文本记录的成本为 $2.00，则这一步的成本为 $36.00。

综合起来，在使用上述技能集引入 1,000 个这种性质的 PDF 文档时，需要支付大约 $57.00。 

## <a name="next-steps"></a>后续步骤
+ [Azure 搜索定价页](https://azure.microsoft.com/pricing/details/search/)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [创建技能组合 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何映射扩充的域](cognitive-search-output-field-mapping.md)
