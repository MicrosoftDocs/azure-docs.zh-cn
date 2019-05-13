---
title: 正则表达式实体
titleSuffix: Azure Cognitive Services
description: 使用正则表达式实体从话语中提取带有一致格式的数据。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/07/2018
ms.author: diberry
ms.openlocfilehash: 71104ecf0514b61e4f0d224d25f2ace9457f3cd3
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145536"
---
# <a name="tutorial-get-well-formatted-data-from-the-utterance"></a>教程：从话语中获取格式正确的数据
在此教程中，创建使用“正则表达式”实体从话语中提取带有一致格式的数据的应用。

**本教程介绍如何执行下列操作：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 创建新应用 
> * 添加意向
> * 添加正则表达式实体 
> * 定型
> * 发布
> * 从终结点获取意向和实体

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="regular-expression-entities"></a>正则表达式实体

此应用使用正则表达式实体的目的是从陈述中提取格式正确的人力资源 (HR) 表单编号。 尽管话语的意向始终是使用机器学习确定的，但此特定实体类型不是机器学习的内容。 

**示例话语包括：**

|示例陈述|
|--|
|HRF-123456 在哪里？|
|HRF-123234 的作者是谁？|
|HRF-456098 是以法语发布的吗？|
|HRF-456098|
|HRF-456098 日期？|
 
在以下情况下，正则表达式非常适合此类数据：

* 数据有良好的格式。


## <a name="create-a-new-app"></a>创建新应用

[!INCLUDE [Follow these steps to create a new LUIS app](../../../includes/cognitive-services-luis-create-new-app-steps.md)]

## <a name="create-intent-for-finding-form"></a>创建查找格式的意向

1. [!INCLUDE [Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

1. 选择“创建新意向”。 

1. 在弹出对话框中输入 `FindForm`，然后选择“完成”。 

    ![“创建新意向”对话框的屏幕截图，其中的搜索框中包含“实用工具”](./media/luis-quickstart-intents-regex-entity/create-new-intent-ddl.png)

1. 将示例陈述添加到意向。

    |示例陈述|
    |--|
    |What is the URL for hrf-123456?|
    |Where is hrf-345678?|
    |When was hrf-456098 updated?|
    |Did John Smith update hrf-234639 last week?|
    |How many versions of hrf-345123 are there?|
    |Who needs to authorize form hrf-123456?|
    |How many people need to sign off on hrf-345678?|
    |hrf-234123 date?|
    |author of hrf-546234?|
    |title of hrf-456234?|

    [![“意向”页的屏幕截图，其中已突出显示新陈述](./media/luis-quickstart-intents-regex-entity/findform-intent.png)](./media/luis-quickstart-intents-regex-entity/findform-intent.png#lightbox)

    [!INCLUDE [Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]  

## <a name="use-the-regular-expression-entity-for-well-formatted-data"></a>将正则表达式实体用于格式正确的数据
要与表单编号匹配的正则表达式实体是 `hrf-[0-9]{6}`。 此正则表达式匹配文本字符 `hrf-`，但忽略大小写和区域性变体。 它完全匹配数字 0-9 中的 6 个数字。

HRF 代表 `human resources form`。

在话语添加到意向时，LUIS 会对话语执行词汇切分。 这些陈述的词汇切分会在连字符的前面和后面添加空格 (`Where is HRF - 123456?`)。在对陈述执行词汇切分之前，正则表达式将应用到原始格式的陈述。 由于正则表达式应用到原始格式，因此，它不需要处理单词边界。 

遵循以下步骤创建一个正则表达式实体，让 LUIS 知道 HRF 编号格式是什么：

1. 在左侧面板中选择“实体”。

1. 在“实体”页上选择“创建新实体”按钮。 

1. 在弹出对话框中，输入新实体名称 `HRF-number`，选择“正则表达式”作为实体类型，输入 `hrf-[0-9]{6}` 作为“正则表达式”值，然后选择“完成”。

    ![用于设置新实体属性的弹出对话框屏幕截图](./media/luis-quickstart-intents-regex-entity/create-regex-entity.png)

1. 从左侧菜单中选择“意向”，然后选择“FindForm”意向查看话语中标记的正则表达式。 

    [![“标记包含现有实体和正则表达式模式的陈述”的屏幕截图](./media/luis-quickstart-intents-regex-entity/labeled-utterances-for-entity.png)](./media/luis-quickstart-intents-regex-entity/labeled-utterances-for-entity.png#lightbox)

    因为该实体不是机器学习的实体，所以在创建该实体后，它会立即应用到话语并显示在 LUIS 网站中。

## <a name="add-example-utterances-to-the-none-intent"></a>将话语示例添加到 None 意向 

[!INCLUDE [Follow these steps to add the None intent to the app](../../../includes/cognitive-services-luis-create-the-none-intent.md)]

## <a name="train-the-app-before-testing-or-publishing"></a>在测试或发布前训练应用

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-the-app-to-query-from-the-endpoint"></a>发布应用，从终结点进行查询

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entity-prediction-from-endpoint"></a>从终结点获取意向和实体预测结果

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 将光标定位到地址中 URL 的末尾，并输入 `When were HRF-123456 and hrf-234567 published in the last year?`。 最后一个查询字符串参数为 `q`，表示陈述查询 (**q**uery)。 此陈述不同于标记的任何陈述，因此，它非常适合用于测试，测试结果应返回包含两个表单编号（`HRF-123456` 和 `hrf-234567`）的 `FindForm` 意向。

    ```json
    {
      "query": "When were HRF-123456 and hrf-234567 published in the last year?",
      "topScoringIntent": {
        "intent": "FindForm",
        "score": 0.9988884
      },
      "intents": [
        {
          "intent": "FindForm",
          "score": 0.9988884
        },
        {
          "intent": "None",
          "score": 0.00204812363
        }
      ],
      "entities": [
        {
          "entity": "hrf-123456",
          "type": "HRF-number",
          "startIndex": 10,
          "endIndex": 19
        },
        {
          "entity": "hrf-234567",
          "type": "HRF-number",
          "startIndex": 25,
          "endIndex": 34
        }
      ]
    }
    ```

    LUIS 将使用正则表达式实体提取命名数据，从编程角度讲，这对于接收 JSON 响应的客户端应用程序而言会更有帮助。


## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="related-information"></a>相关信息

* [正则表达式](luis-concept-entity-types.md#regular-expression-entity)实体概念
* [如何训练](luis-how-to-train.md)
* [如何发布](luis-how-to-publish-app.md)
* [如何在 LUIS 门户中测试](luis-interactive-test.md)

## <a name="next-steps"></a>后续步骤
本教程创建了新意向，添加了示例话语，然后创建了正则表达式实体以便从话语中提取格式良好的数据。 在训练并发布应用后，向终结点发出的查询识别了意向，并返回了提取的数据。

> [!div class="nextstepaction"]
> [了解列表实体](luis-quickstart-intent-and-list-entity.md)

