---
title: 文本完全匹配
titleSuffix: Azure Cognitive Services
description: 获取与项的预定义列表匹配的数据。 列表中的每个项可以有也是完全匹配的同义词
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: df37e7aad2420d0bc280121634d49675ae29ee5a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236418"
---
# <a name="tutorial-get-exact-text-matched-data-from-an-utterance"></a>教程：从话语中获取文本完全匹配的数据

本教程介绍如何获取与项的预定义列表匹配的实体数据。 

**本教程介绍如何执行下列操作：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 创建应用
> * 添加意向
> * 添加列表实体 
> * 定型 
> * 发布
> * 从终结点获取意向和实体

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="what-is-a-list-entity"></a>什么是列表实体？

列表实体是话语中的字词的文本完全匹配项。 

列表中的每个项可以包含一个同义词列表。 对于人力资源应用，可以通过多个关键信息来识别某个公司部门，例如官方名称、常用缩略词和计费部门代码。 

人力资源应用需要确定某位员工所调动到的部门。 

出现以下情况时，列表实体非常适合此类数据：

* 数据值是已知的集。
* 此集不超出此实体类型的最大 LUIS [边界](luis-boundaries.md)。
* 话语中的文本是同义项或规范名称的完全匹配。 LUIS 不会使用除文本完全匹配项之外的列表。 仅仅有列表实体无法解析词干、复数形式和其他变体。 若要管理变体，请考虑使用带有可选文本语法的[模式](luis-concept-patterns.md#syntax-to-mark-optional-text-in-a-template-utterance)。 

## <a name="create-a-new-app"></a>创建新应用

[!INCLUDE [Follow these steps to create a new LUIS app](../../../includes/cognitive-services-luis-create-new-app-steps.md)]

## <a name="create-an-intent-to-transfer-employees-to-a-different-department"></a>创建将员工调动到其他部门的意向

1. [!INCLUDE [Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 选择“创建新意向”。 

3. 在弹出对话框中输入 `TransferEmployeeToDepartment`，然后选择“完成”。 

    ![“创建新意向”对话框的屏幕截图](./media/luis-quickstart-intent-and-list-entity/hr-create-new-intent-ddl.png)

4. 将示例陈述添加到意向。

    |示例陈述|
    |--|
    |将 John W. Smith 转到会计部门|
    |将 Jill Jones 转到研发部门|
    |1234 部门新增一位名为 Bill Bradstreet 的成员|
    |将 John Jackson 安排在工程部门 |
    |将 Debra Doughtery 转到内部销售部门|
    |将 Jill Jones 转到 IT 部门|
    |将 Alice Anderson 转到开发运营部门|
    |将 Carl Chamerlin 转到财务部门|
    |将 Steve Standish 转到 1234 部门|
    |将 Tanner Thompson 转到 3456 部门|

    [![带有示例话语的意图屏幕截图](media/luis-quickstart-intent-and-list-entity/intent-transfer-employee-to-department.png "Screenshot of intent with example utterances")](media/luis-quickstart-intent-and-list-entity/intent-transfer-employee-to-department.png#lightbox)

    [!INCLUDE [Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]  

## <a name="department-list-entity"></a>部门列表实体

“TransferEmployeeToDepartment”意向有示例话语以后，LUIS 需了解什么是部门。 

每个项的主规范名称是部门名称。 每个规范名称的同义词的示例如下： 

|规范名称|同义词|
|--|--|
|计帐|acct<br>accting<br>3456|
|开发操作|Devops<br>4949|
|工程|eng<br>enging<br>4567|
|财务|fin<br>2020|
|信息技术|IT<br>2323|
|内部销售|isale<br>insale<br>1414|
|研究和开发|R&D<br>1234|

1. 在左侧面板中选择“实体”。

1. 选择“创建新实体”。

1. 在实体弹出对话框中，输入 `Department` 作为实体名称，输入“列表”作为实体类型。 选择“完成”。  

    [![创建新实体弹出项对话框的屏幕截图](media/luis-quickstart-intent-and-list-entity/create-new-list-entity-named-department.png "Screenshot of creating new entity pop-up dialog")](media/luis-quickstart-intent-and-list-entity/create-new-list-entity-named-department.png#lightbox)

1. 在“部门”实体页上，输入 `Accounting` 作为新值。

1. 对于同义词，请从上表中添加同义词。

1. 继续添加所有规范名称及其同义词。 

## <a name="add-example-utterances-to-the-none-intent"></a>将话语示例添加到 None 意向 

[!INCLUDE [Follow these steps to add the None intent to the app](../../../includes/cognitive-services-luis-create-the-none-intent.md)]

## <a name="train-the-app-so-the-changes-to-the-intent-can-be-tested"></a>训练应用，以便可以测试对意向所做的更改 

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-the-app-so-the-trained-model-is-queryable-from-the-endpoint"></a>发布应用，以便可以从终结点查询已训练的模型

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entity-prediction-from-endpoint"></a>从终结点获取意向和实体预测结果

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)] 

1. 将光标定位到地址中 URL 的末尾，并输入 `shift Joe Smith to IT`。 最后一个查询字符串参数为 `q`，表示陈述查询 (**q**uery)。 此陈述不同于标记的任何陈述，因此，它非常适合用于测试，测试结果应返回提取了 `Department` 的 `TransferEmployeeToDepartment` 意向。

   ```json
    {
      "query": "shift Joe Smith to IT",
      "topScoringIntent": {
        "intent": "TransferEmployeeToDepartment",
        "score": 0.9775754
      },
      "intents": [
        {
          "intent": "TransferEmployeeToDepartment",
          "score": 0.9775754
        },
        {
          "intent": "None",
          "score": 0.0154493852
        }
      ],
      "entities": [
        {
          "entity": "it",
          "type": "Department",
          "startIndex": 19,
          "endIndex": 20,
          "resolution": {
            "values": [
              "Information Technology"
            ]
          }
        }
      ]
    }
   ```

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="related-information"></a>相关信息

* [列表实体](luis-concept-entity-types.md#list-entity)概念信息
* [如何训练](luis-how-to-train.md)
* [如何发布](luis-how-to-publish-app.md)
* [如何在 LUIS 门户中测试](luis-interactive-test.md)


## <a name="next-steps"></a>后续步骤
本教程创建了新意向，添加了示例话语，然后创建了列表实体以便从话语中提取确切的文本匹配项。 在训练并发布应用后，向终结点发出的查询识别了意向，并返回了提取的数据。

继续使用此应用，[添加复合实体](luis-tutorial-composite-entity.md)。

> [!div class="nextstepaction"]
> [将带有角色的预生成实体添加到应用](tutorial-entity-roles.md)

