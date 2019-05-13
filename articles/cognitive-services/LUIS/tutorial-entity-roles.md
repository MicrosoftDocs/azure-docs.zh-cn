---
title: 包含角色的上下文数据 - 语言理解
titleSuffix: Azure Cognitive Services
description: 基于上下文查找相关的数据。 例如，对于从一个建筑物和办公室到另一个建筑物和办公室的物理移动，源位置和目标位置是相关的。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: a0ab928ef3b8551e3e20ff3c4b16533c80ee4b7d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149250"
---
# <a name="tutorial-extract-contextually-related-data-from-an-utterance"></a>教程：从陈述中提取上下文相关的数据

在本教程中，基于上下文查找相关的数据片段。 例如，从一个城市转移到另一个城市的原位置和目的地。 可能同时需要这两个数据片段，并且它们彼此相关。  

角色可与任何预生成的或自定义的实体类型配合使用，并可在示例言语和模式中使用。 

**本教程介绍如何执行下列操作：**

> [!div class="checklist"]
> * 创建新应用
> * 添加意向 
> * 使用角色获取来源和目标信息
> * 定型
> * 发布
> * 从终结点获取意向和实体角色

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="related-data"></a>相关数据

此应用确定某位员工从原始城市移动到目的地城市的位置。 它使用 GeographyV2 预生成实体来识别城市名称，并使用角色来确定言语中的位置类型（来源和目标）。

当要提取的实体数据具有以下特征时，应使用角色：

* 在言语的上下文中彼此相关。
* 使用特定的词语选项来指示每个角色。 这些词汇的示例包括：from/to（从/到）、leaving/headed to（离开/前往）、away from/toward（离开/前往）。
* 这两个角色经常出现在同一个言语中，使 LUIS 能够从这种频繁的上下文用法中学习信息。
* 需要由客户端应用作为一个信息单元进行分组和处理。

## <a name="create-a-new-app"></a>创建新应用

[!INCLUDE [Follow these steps to create a new LUIS app](../../../includes/cognitive-services-luis-create-new-app-steps.md)]

## <a name="create-an-intent-to-move-employees-between-cities"></a>创建在城市之间移动员工的意向

1. [!INCLUDE [Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

1. 选择“创建新意向”。 

1. 在弹出对话框中输入 `MoveEmployeeToCity`，然后选择“完成”。 

    ![“创建新意向”对话框的屏幕截图](./media/tutorial-entity-roles/create-new-intent-move-employee-to-city.png)

1. 将示例陈述添加到意向。

    |示例陈述|
    |--|
    |将 John W.Smith 从西雅图派往奥兰多|
    |将 Jill Jones 从西雅图调至开罗|
    |将 John Jackson 从坦帕调至亚特兰大 |
    |将 Debra Doughtery 从芝加哥派往塔尔萨|
    |将 Jill Jones 从开罗派往坦帕|
    |将 Alice Anderson 从雷德蒙德调至奥克兰|
    |将 Carl Chamerlin 从旧金山派往雷德蒙德|
    |将 Steve Standish 从圣地亚哥调至贝尔维尤 |
    |将 Tanner Thompson 从堪萨斯城派往芝加哥|

    [![LUIS 的屏幕截图，在 MoveEmployee 意向中有新陈述](./media/tutorial-entity-roles/hr-enter-utterances.png)](./media/tutorial-entity-roles/hr-enter-utterances.png#lightbox)

## <a name="add-prebuilt-entity-geographyv2"></a>添加预生成实体 geographyV2

预生成实体 geographyV2 提取位置信息，包括城市名称。 由于言语中有两个城市名称在上下文中彼此相关，因此请使用角色来提取该上下文。

1. 在左侧导航栏中选择“实体”。

1. 选择“添加预生成实体”，然后在搜索栏中选择 `geo` 来筛选预生成实体。 

    ![将 geographyV2 预生成实体添加到应用](media/tutorial-entity-roles/add-geographyV2-prebuilt-entity.png)
1. 选中该复选框，然后选择“完成”。
1. 在“实体”列表中，选择“geographyV2”打开新实体。 
1. 添加两个角色：`Origin` 和 `Destination`。 

    ![将角色添加到预生成实体](media/tutorial-entity-roles/add-roles-to-prebuilt-entity.png)
1. 在左侧导航栏中选择“意向”，然后选择“MoveEmployeeToCity”意向。 请注意，城市名称标有预生成实体 **geogrpahyV2**。
1. 在列表的第一个言语中，选择来源位置。 此时会显示一个下拉菜单。 在列表中选择“geographyV2”，然后在弹出的菜单中选择“来源”。
1. 使用上一步骤中所述的方法来标记所有言语中位置的所有角色。 


## <a name="add-example-utterances-to-the-none-intent"></a>将话语示例添加到 None 意向 

[!INCLUDE [Follow these steps to add the None intent to the app](../../../includes/cognitive-services-luis-create-the-none-intent.md)]

## <a name="train-the-app-so-the-changes-to-the-intent-can-be-tested"></a>训练应用，以便可以测试对意向所做的更改 

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-the-app-so-the-trained-model-is-queryable-from-the-endpoint"></a>发布应用，以便可以从终结点查询已训练的模型

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entity-prediction-from-endpoint"></a>从终结点获取意向和实体预测结果

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]


1. 将光标定位到地址栏中 URL 的末尾，并输入 `Please move Carl Chamerlin from Tampa to Portland`。 最后一个查询字符串参数为 `q`，表示陈述**查询**。 此话语不同于标记的任何话语，因此，它非常适用于测试，测试结果应返回提取了实体的 `MoveEmployee` 意向。

    ```json
    {
      "query": "Please move Carl Chamerlin from Tampa to Portland",
      "topScoringIntent": {
        "intent": "MoveEmployeeToCity",
        "score": 0.979823351
      },
      "intents": [
        {
          "intent": "MoveEmployeeToCity",
          "score": 0.979823351
        },
        {
          "intent": "None",
          "score": 0.0156363435
        }
      ],
      "entities": [
        {
          "entity": "geographyV2",
          "role": "Destination",
          "startIndex": 41,
          "endIndex": 48,
          "score": 0.6044041
        },
        {
          "entity": "geographyV2",
          "role": "Origin",
          "startIndex": 32,
          "endIndex": 36,
          "score": 0.739491045
        }
      ]
    }
    ```
    
    预测了正确的意向，并且实体数组在对应的 **entities** 属性中同时具有来源和目标角色。
    
## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="related-information"></a>相关信息

* [实体的概念](luis-concept-entity-types.md)
* [角色的概念](luis-concept-roles.md)
* [预生成实体列表](luis-reference-prebuilt-entities.md)
* [如何训练](luis-how-to-train.md)
* [如何发布](luis-how-to-publish-app.md)
* [如何在 LUIS 门户中测试](luis-interactive-test.md)
* [角色](luis-concept-roles.md)

## <a name="next-steps"></a>后续步骤

在本教程中，已创建了一个新的意向，并为源位置和目标位置的根据上下文学习的数据添加了示例陈述。 在训练并发布应用后，客户端应用程序可以使用该信息来创建包含相关信息的移动票证。

> [!div class="nextstepaction"] 
> [了解如何添加复合实体](luis-tutorial-composite-entity.md) 
