---
title: 图像分析认知搜索技能 - Azure 搜索
description: 在 Azure 搜索扩充管道中使用 ImageAnalysis 认知技能通过图像分析来提取语义文本。
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
ms.openlocfilehash: bb18c858a17e290a8ce2cc88dc1e7d88d21afe0f
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021904"
---
#   <a name="image-analysis-cognitive-skill"></a>图像分析认知技能

图像分析技能根据图像内容提取一组丰富的可视特征。 例如，可从图像生成标题栏、生成标记或识别名人和地标。 此技能使用认知服务中的[计算机视觉](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home)提供的机器学习模型。 

> [!NOTE]
> 当您展开作用域提高频率的处理，添加更多文档，或添加更多的 AI 算法，你将需要[附加可计费的认知服务资源](cognitive-search-attach-cognitive-services.md)。 调用认知服务中的 API，以及在 Azure 搜索中的文档破解阶段提取图像时，会产生费用。 提取文档中的文本不会产生费用。
>
> 执行的内置技能收费的现有[认知服务付款现转价格](https://azure.microsoft.com/pricing/details/cognitive-services/)。 介绍了图像提取定价[Azure 搜索定价页](https://go.microsoft.com/fwlink/?linkid=2042400)。


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Vision.ImageAnalysisSkill 

## <a name="skill-parameters"></a>技能参数

参数区分大小写。

| 参数名称     | 描述 |
|--------------------|-------------|
| defaultLanguageCode   |  表示要返回的语言的字符串。 该服务以指定的语言返回识别结果。 如果未指定此参数，则默认值为“en”。 <br/><br/>支持的语言为： <br/>en - 英语（默认） <br/> zh - 简体中文|
|visualFeatures |   表示要返回的可视特征类型的一组字符串。 有效的可视特征类型包括：  <ul><li> categories - 根据认知服务[文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/category-taxonomy)中定义的分类对图像内容进行分类。</li><li> tags - 使用与图像内容相关字词的详细列表来标记图像。</li><li>*说明*-描述映像内容具有完整的英语句子。</li><li>*人脸*-检测人脸是否存在。 如果存在，则生成位置、性别和年龄。</li><li> *ImageType* -检测图像是否为剪贴画或线条绘图。</li><li>  *颜色*-确定强调文字颜色、 主色图像是否为黑色和白色。</li><li>*成人*-检测到的映像是色情在本质上 （描绘了裸体或性别采取措施）。 也检测性暗示内容。</li></ul> 可视特征的名称区分大小写。|
| 详细信息   | 表示要返回的特定于域的详细信息的一组字符串。 有效的可视特征类型包括： <ul><li>*名人*-如果在图像中检测到标识名人。</li><li>*特征点*-如果在图像中检测到标识特征点。</li></ul>
 |

## <a name="skill-inputs"></a>技能输入

| 输入名称      | 说明                                          |
|---------------|------------------------------------------------------|
| 图像         | 复杂类型。 当前仅适用于“/document/normalized_images”字段，当 ```imageAction``` 设置为非 ```none``` 值时由 Azure Blob 索引器生成。 请参阅[此示例](#sample-output)获取详细信息。|



##  <a name="sample-definition"></a>示例定义
```json
{
    "@odata.type": "#Microsoft.Skills.Vision.ImageAnalysisSkill",
    "context": "/document/normalized_images/*",
    "visualFeatures": [
        "Tags",
        "Faces",
        "Categories",
        "Adult",
        "Description",
        "ImageType",
        "Color"
    ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "image",
            "source": "/document/normalized_images/*"
        }
    ],
    "outputs": [
        {
            "name": "categories",
            "targetName": "myCategories"
        },
        {
            "name": "tags",
            "targetName": "myTags"
        },
        {
            "name": "description",
            "targetName": "myDescription"
        },
        {
            "name": "faces",
            "targetName": "myFaces"
        },
        {
            "name": "imageType",
            "targetName": "myImageType"
        },
        {
            "name": "color",
            "targetName": "myColor"
        },
        {
            "name": "adult",
            "targetName": "myAdultCategory"
        }
    ]
}
```

##  <a name="sample-input"></a>示例输入

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "image": {
                    "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
                    "width": 500,
                    "height": 300,
                    "originalWidth": 5000,
                    "originalHeight": 3000,
                    "rotationFromOriginal": 90,
                    "contentOffset": 500
                }
            }
        }
    ]
}
```


##  <a name="sample-output"></a>示例输出

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "categories": [
                    {
                        "name": "abstract_",
                        "score": 0.00390625
                    },
                    {
                        "name": "people_",
                        "score": 0.83984375,
                        "detail": {
                            "celebrities": [
                                {
                                    "name": "Satya Nadella",
                                    "faceBoundingBox": {
                                        "left": 597,
                                        "top": 162,
                                        "width": 248,
                                        "height": 248
                                    },
                                    "confidence": 0.999028444
                                }
                            ],
                            "landmarks": [
                                {
                                    "name": "Forbidden City",
                                    "confidence": 0.9978346
                                }
                            ]
                        }
                    }
                ],
                "adult": {
                    "isAdultContent": false,
                    "isRacyContent": false,
                    "adultScore": 0.0934349000453949,
                    "racyScore": 0.068613491952419281
                },
                "tags": [
                    {
                        "name": "person",
                        "confidence": 0.98979085683822632
                    },
                    {
                        "name": "man",
                        "confidence": 0.94493889808654785
                    },
                    {
                        "name": "outdoor",
                        "confidence": 0.938492476940155
                    },
                    {
                        "name": "window",
                        "confidence": 0.89513939619064331
                    }
                ],
                "description": {
                    "tags": [
                        "person",
                        "man",
                        "outdoor",
                        "window",
                        "glasses"
                    ],
                    "captions": [
                        {
                            "text": "Satya Nadella sitting on a bench",
                            "confidence": 0.48293603002174407
                        }
                    ]
                },
                "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
                "metadata": {
                    "width": 1500,
                    "height": 1000,
                    "format": "Jpeg"
                },
                "faces": [
                    {
                        "age": 44,
                        "gender": "Male",
                        "faceBoundingBox": {
                            "left": 593,
                            "top": 160,
                            "width": 250,
                            "height": 250
                        }
                    }
                ],
                "color": {
                    "dominantColorForeground": "Brown",
                    "dominantColorBackground": "Brown",
                    "dominantColors": [
                        "Brown",
                        "Black"
                    ],
                    "accentColor": "873B59",
                    "isBwImg": false
                    },
                "imageType": {
                    "clipArtType": 0,
                    "lineDrawingType": 0
                }
            }
        }
    ]
}
```


## <a name="error-cases"></a>错误案例
在以下错误案例中，未提取任何元素。

| 错误代码 | 描述 |
|------------|-------------|
| NotSupportedLanguage | 不支持提供的语言。 |
| InvalidImageUrl | 图片 URL 格式不正确或无法访问。|
| InvalidImageFormat | 输入数据不是有效的图像。 |
| InvalidImageSize | 输入的图像太大。 |
| NotSupportedVisualFeature  | 指定的特征类型无效。 |
| NotSupportedImage | 不受支持的图片，例如儿童色情内容。 |
| InvalidDetails | 不受支持的特定于域的模型。 |

## <a name="see-also"></a>另请参阅

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [创建索引器 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
