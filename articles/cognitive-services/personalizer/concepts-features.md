---
title: 功能：操作和上下文 - 个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: 个性化体验创建服务使用特征（有关操作和上下文的信息）来做出更好的排名建议。 特征可能非常宽泛，也可能特定于某个项。
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: ebe7f9307fcfa39d6cb133203a4c17243ad390c5
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025499"
---
# <a name="features-are-information-about-actions-and-context"></a>特征是指有关操作和上下文的信息

个性化体验创建服务的工作原理是学习应用程序在给定的上下文中应向用户显示哪些内容。

个性化体验创建服务使用**特征**（有关**当前上下文**的信息）来选择最佳的**操作**。 特征表示你认为有助于个性化的，以实现更佳效果的所有信息。 特征可能非常宽泛，也可能特定于某个项。 

例如，**特征**可能与以下对象相关：

* 用户，例如 `UserID`。 
* 内容，视频是 `Documentary`、`Movie` 还是 `TV Series`，或者某个零售商品在商店中是否有货。
* 当前时间段，例如星期几。

个性化体验创建服务不会规定、限制或固定可为操作和上下文发送的特征：

* 可为某些操作发送某些特征；如果没有这些特征，可以不为某些操作发送这些特征。 例如，电视连续剧可能包含电影所不具备的某些属性。
* 某些特征只在某些时间才会提供。 例如，移动应用程序提供的信息可能比网页更多。 
* 可以不断地添加和删除有关上下文与操作的特征。 个性化体验创建服务会持续从提供的信息中学习。
* 每个上下文必须至少有一个特征。 个性化体验创建服务不支持空上下文。 如果你每次只发送固定的上下文，则个性化体验创建服务只会选择与操作中的特征相关的排名操作。 
* 个性化体验创建服务会尝试选择在任何时间都最适合每个人的操作。

## <a name="supported-feature-types"></a>支持的特征类型

个性化体验创建服务支持字符串、数字和布尔类型的特征。

应在请求中省略不存在的特征。 避免发送包含 null 值的特征，因为在训练模型时，此类特征将作为具有“null”值的现有特征进行处理。

## <a name="categorize-features-with-namespaces"></a>使用命名空间将特征分类

个性化体验创建服务提取在命名空间中组织的特征。 你需要确定是否要在应用程序中使用命名空间，以及应该使用哪些命名空间。 命名空间用于分组有关类似主题的特征，或者来自特定源的特征。

下面是应用程序使用的特征命名空间示例：

* User_Profile_from_CRM
* 时间
* Mobile_Device_Info
* http_user_agent
* VideoResolution
* UserDeviceInfo
* 天气
* Product_Recommendation_Ratings
* current_time
* NewsArticle_TextAnalytics

可以根据自己的约定来命名特征命名空间，只要它们是有效的 JSON 键即可。

在以下 JSON 中，`user`、`state` 和 `device` 是特征命名空间。

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "name":"Doug"
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true
            }
        }
    ]
}
```

## <a name="how-to-make-feature-sets-more-effective-for-personalizer"></a>如何使得特征集对于个性化体验创建服务更为有效

一个良好的特征集可以帮助个性化体验创建服务了解如何预测出能够促成最佳效果的操作。 

假设要将遵循以下建议的特征发送到个性化体验创建服务排名 API：

* 有足够的特征可以促成个性化。 针对内容的精度越高，所需的特征就越多。

* 有足够的多样化密度特征。 如果将许多的项分组到少数几个桶中，则认为特征是密集的。 例如，可将数千个视频分类为“长”（时长超过 5 分钟）和“短”（时长短于 5 分钟）。 这是一个非常密集的特征。 另一方面，数千个项可能有一个名为“标题”的属性，而每个项的此属性几乎永远不会使用相同的值。 这是一个极不密集的特征，也称为稀疏特征。  

使用高密度特征可帮助个性化体验创建服务在不同的项之间进行推理学习。 但是，如果只存在少数几个特征，并且这些特征过于密集，则个性化体验创建服务只会使用少数几个可供选择的桶，来尝试精确针对内容。

### <a name="improve-feature-sets"></a>改善特征集 

通过执行脱机评估来分析用户行为。 这样，便可以查看以往的数据，以确定哪些特征对积极的效果贡献最大，以及哪些特征对此效果的贡献不大。 可以查看哪些特征起到了作用。需要由你以及你的应用程序找到要发送给个性化体验创建服务以进一步改善效果的更好特征。

这些部分提供了有关改善发送到个性化体验创建服务的特征的常见做法。

#### <a name="make-features-more-dense"></a>使特征更密集

可以通过编辑特征集，使其变得更大、更多或更稀疏，来改善特征集。

例如，精度为秒的时间戳是非常稀疏的特征。 可以通过将时间分类为“上午”、“中午”、“下午”等，来提高此特征的密度（更有效）。


#### <a name="expand-feature-sets-with-extrapolated-information"></a>使用推理出的信息扩展特征集

评估可从已有信息派生的未探索属性也可以获得更多的特征。 例如，在科幻电影列表个性化中，用户的行为在周末与工作日是否不同？ 可将时间扩展为包含“周末”或“工作日”属性。 国家法定节假日是否会推动观众对某类电影的兴趣？ 例如，在相关的场合中，可以使用“万圣节”属性。 雨天是否会明显影响许多人选择观看电影？ 如果使用时间和地点，则天气服务可以提供此类信息，你可以将此信息添加为额外的特征。 

#### <a name="expand-feature-sets-with-artificial-intelligence-and-cognitive-services"></a>使用人工智能和认知服务扩展特征集

人工智能和随时可运行的认知服务可对个性化体验创建服务做出强有力的补充。 

通过使用人工智能服务预处理项，可以自动提取可能与个性化相关的信息。

例如：

* 可以通过[视频索引器](https://azure.microsoft.com/services/media-services/video-indexer/)运行电影文件，以提取场景元素、文本、情绪和许多其他属性。 然后，可以提高这些属性的密度，以反映原始项元数据所不具备的特征。 
* 可以通过对象检测运行图像，通过情绪检测运行人脸，等等。
* 可以通过使用必应知识图形等提取实体、情绪以及扩展实体来扩充文本中的信息。

可以使用其他多种 [Azure 认知服务](https://www.microsoft.com/cognitive-services)，例如

* [实体链接](../entitylinking/home.md)
* [文本分析](../text-analytics/overview.md)
* [情感](../emotion/home.md)
* [计算机视觉](../computer-vision/home.md)

## <a name="actions-represent-a-list-of-options"></a>操作表示选项列表

每个操作：

* 具有一个 ID。
* 具有特征列表。
* 特征列表可以很大（数百个），但我们建议评估特征的有效性，并删除对结果不起作用的特征。 
* **操作**中的特征不一定与个性化体验创建服务使用的**上下文**中的特征相关联。
* 操作特征可以存在于某些操作中，而不存在于其他操作中。 
* 特定操作 ID 的特征可能只会在某一天才提供，过后就不会提供。 

如果特征集比较稳定，则个性化体验创建服务的机器学习算法的表现会更好；但是，如果特征集不断变化，则排名调用不会失败。

为操作排名时，请不要发送超过 50 个操作。 这些操作可以是相同的 50 个操作，也可以是不同的操作。 例如，如果你为某个电子商务应用程序创建了一个包含 10,000 个商品的产品目录，则可以使用建议或筛选引擎来确定客户可能最喜欢的 40 个商品，并使用个性化体验创建服务找到在当前上下文中会产生最大回报的商品（例如，用户会添加到购物篮的商品）。


### <a name="examples-of-actions"></a>操作示例

发送到排名 API 的操作取决于尝试个性化的内容。

下面是一些示例：

|目的|操作|
|--|--|
|个性化要在新闻网站上突出显示的文章。|每个操作是一篇潜在的新闻文章。|
|优化网站上的广告位置。|每个操作是一个布局或有关创建广告布局的规则（例如，在顶部排列、在右侧排列、使用小图像、使用大图像）。|
|在购物网站上显示推荐商品的个性化排名。|每个操作是一个具体的产品。|
|建议要应用到特定照片的用户界面元素（例如筛选器）。|每个操作可以是不同的筛选器。|
|选择用于澄清用户意向或建议操作的聊天机器人响应。|每个操作是一个有关如何解释响应的选项。|
|选择要在搜索结果列表顶部显示的内容|每个操作是最相关的少数几条搜索结果之一。|


### <a name="examples-of-features-for-actions"></a>操作特征示例

下面是典型的操作特征示例。 具体的特征在很大程度上取决于每个应用程序。

* 包含操作特征的特征。 例如，这是一部电影还是电视连续剧？
* 有关用户在过去如何与此操作交互的特征。 例如，人口统计特征 A 和 B 的人员基本上都看过这部电影，该电影往往上映过多次。
* 有关用户如何查看操作的特征。 例如，缩略图中显示的电影海报是否包含人脸、汽车或布局？

### <a name="load-actions-from-the-client-application"></a>从客户端应用程序加载操作

操作中的特征往往来自于内容管理系统、目录和推荐器系统。 应用程序负责从现有的相关数据库和系统中加载有关操作的信息。 如果操作不会更改，或者每次加载操作都会对性能产生不必要的影响，则你可以在应用程序中添加逻辑用于缓存此信息。

### <a name="prevent-actions-from-being-ranked"></a>防止为操作排名

在某些情况下，你不希望向用户显示某些操作。 防止将某个操作排名在最前面的最佳方法是不要将此操作包含在排名 API 操作列表中的第一个位置。

在某些情况下，只能稍后在业务逻辑中确定是否要向用户显示排名 API 调用的最终操作。 对于这种情况，应使用非活动事件。

## <a name="json-format-for-actions"></a>操作的 JSON 格式

调用排名时，需要发送多个可供选择的操作：

```json
{
    "actions": [
    {
      "id": "pasta",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "medium"
        },
        {
          "nutritionLevel": 5,
          "cuisine": "italian"
        }
      ]
    },
    {
      "id": "ice cream",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none"
        },
        {
          "nutritionalLevel": 2
        }
      ]
    },
    {
      "id": "juice",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none"
        },
        {
          "nutritionLevel": 5
        },
        {
          "drink": true
        }
      ]
    },
    {
      "id": "salad",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "low"
        },
        {
          "nutritionLevel": 8
        }
      ]
    }
  ]
}
```

## <a name="examples-of-context-information"></a>上下文信息示例

上下文信息取决于每个应用程序和用例，但往往包括如下所述的信息：

* 有关用户的人口统计和档案信息。
* 从 HTTP 标头提取的信息（例如用户代理），或派生自 HTTP 信息的信息，例如基于 IP 地址的反向地理查找。
* 有关当前时间的信息，例如星期几、是否为周末、上午还是下午、是否为长假期，等等。
* 从移动应用程序提取的信息，例如位置、动作或电池电量。
* 用户行为的历史聚合 - 例如，此用户最喜欢观看的电影流派是什么。

应用程序负责从现有的相关数据库、传感器和系统中加载有关上下文的信息。 如果上下文信息不会更改，则可以在应用程序中添加逻辑，以便在将此信息发送到排名 API 之前将其缓存。

## <a name="json-format-for-context"></a>上下文的 JSON 格式 

上下文表示为发送到排名 API 的 JSON 对象：

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "name":"Doug"
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true
            }
        }
    ]
}
```

## <a name="next-steps"></a>后续步骤

[强化学习](concepts-reinforcement-learning.md) 