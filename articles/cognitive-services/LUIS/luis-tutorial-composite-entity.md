---
title: 复合实体教程
titleSuffix: Azure Cognitive Services
description: 添加复合实体来将提取的各种类型的数据捆绑到单个内含实体中。 通过捆绑数据，客户端应用程序可以轻松提取各种数据类型的相关数据。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 02/19/2019
ms.author: diberry
ms.openlocfilehash: a5cedec4937da276a4848498ad29a723f6e41ba3
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65073703"
---
# <a name="tutorial-group-and-extract-related-data"></a>教程：对相关的数据进行分组和提取
在本教程中，添加复合实体来将提取的各种类型的数据捆绑到单个内含实体中。 通过捆绑数据，客户端应用程序可以轻松提取各种数据类型的相关数据。

复合实体的目的是将相关实体分组为父类别实体。 在创建复合实体之前，这些现有信息都是单独的实体。 

复合实体非常适合此类数据，因为此类数据：

* 彼此相关。 
* 使用各种实体类型。
* 需要由客户端应用作为一个信息单元进行分组和处理。

**本教程介绍如何执行下列操作：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 导入示例应用
> * 创建意向
> * 添加复合实体 
> * 定型
> * 发布
> * 从终结点获取意向和实体

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="import-example-app"></a>导入示例应用

1.  从列表实体教程中下载并保存[应用 JSON 文件](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/tutorials/build-app/tutorial_list.json)。

2. 将 JSON 导入到新应用中。

3. 在“管理”部分的“版本”选项卡上，克隆版本并将其命名为 `composite`。 克隆非常适合用于演练各种 LUIS 功能，且不会影响原始版本。 由于版本名称用作 URL 路由的一部分，因此该名称不能包含任何在 URL 中无效的字符。

## <a name="composite-entity"></a>复合实体

在此应用中，部门名称是在 **Department** 列表实体中定义的并且包括同义词。 

**TransferEmployeeToDepartment** 意向具有请求将员工移动到新部门的示例话语。 

此意向的示例话语包括：

|示例陈述|
|--|
|将 John W. Smith 转到会计部门|
|将 Jill Jones 转到研发部门|
 
移动请求应当包括部门名称和员工姓名。 

## <a name="add-the-personname-prebuilt-entity-to-help-with-common-data-type-extraction"></a>添加 PersonName 预构建实体以帮助提取常见数据类型

LUIS 为常见数据提取提供多个预生成的实体。 

1. 从顶部导航栏中选择“生成”，然后从左侧导航菜单中选择“实体”。

1. 选择“管理预生成的实体”按钮。

1. 从预生成实体的列表中选择 **[PersonName](luis-reference-prebuilt-person.md)**，然后选择“完成”。

    ![在“预生成的实体”对话框中选择的数字的屏幕截图](./media/luis-tutorial-composite-entity/add-personname-prebuilt-entity.png)

    此实体可以帮助你向客户端应用程序中添加名称识别。

## <a name="create-composite-entity-from-example-utterances"></a>基于示例话语创建复合实体

1. 从左侧导航栏中选择“意向”。

1. 从意向列表中选择“TransferEmployeeToDepartment”。

1. 在查询文本`place John Jackson in engineering`，选择 personName 实体`John Jackson`，然后选择**将包装在复合实体**以下查询文本的弹出菜单列表中。 

    ![在下拉对话框中选择换行组合的屏幕截图](./media/luis-tutorial-composite-entity/hr-create-composite-entity-1.png)

1. 然后立即选择最后一个实体，话语中的 `engineering`。 在所选字词下面绘制的绿色条指示复合实体。 在弹出菜单中，输入复合名称 `TransferEmployeeInfo`，然后选择 Enter。 

    ![在下拉列表对话框中输入复合名称的屏幕截图](./media/luis-tutorial-composite-entity/hr-create-composite-entity-2.png)

1. 在“想要创建哪种类型的实体?”中，所有必需字段都在列表中：`personName` 和 `Department`。 选择“完成”。 请注意，预构建实体 personName 已添加到复合实体中。 如果可能有预构建实体出现在复合实体的开始标记和结束标记之间，则复合实体必须包含那些预构建实体。 如果未包括预构建实体，则不能正确预测复合实体，但可以正确预测每个个体元素。

    ![在下拉列表对话框中输入复合名称的屏幕截图](./media/luis-tutorial-composite-entity/hr-create-composite-entity-3.png)

## <a name="label-example-utterances-with-composite-entity"></a>使用复合实体标记示例话语

1. 在每个示例话语中，选择应在复合中的最左侧实体。 然后选择“在复合实体中包装”.

1. 选择复合实体中的最后一个单词，然后从弹出菜单中选择“TransferEmployeeInfo”。 

1. 验证意向中的所有话语都已使用复合实体进行标记。 

## <a name="train-the-app-so-the-changes-to-the-intent-can-be-tested"></a>训练应用，以便可以测试对意向所做的更改 

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-the-app-so-the-trained-model-is-queryable-from-the-endpoint"></a>发布应用，以便可以从终结点查询已训练的模型

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entity-prediction-from-endpoint"></a>从终结点获取意向和实体预测结果 

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 将光标定位到地址中 URL 的末尾，并输入 `Move Jill Jones to DevOps`。 最后一个查询字符串参数为 `q`，表示话语查询。 

    由于此测试是为了验证是否正确提取复合，因此测试可以包括现有的示例话语或新话语。 一个有效的测试是在复合实体中包含所有子实体。

    ```json
    {
      "query": "Move Jill Jones to DevOps",
      "topScoringIntent": {
        "intent": "TransferEmployeeToDepartment",
        "score": 0.9882747
      },
      "intents": [
        {
          "intent": "TransferEmployeeToDepartment",
          "score": 0.9882747
        },
        {
          "intent": "None",
          "score": 0.00925369747
        }
      ],
      "entities": [
        {
          "entity": "jill jones",
          "type": "builtin.personName",
          "startIndex": 5,
          "endIndex": 14
        },
        {
          "entity": "devops",
          "type": "Department",
          "startIndex": 19,
          "endIndex": 24,
          "resolution": {
            "values": [
              "Development Operations"
            ]
          }
        },
        {
          "entity": "jill jones to devops",
          "type": "TransferEmployeeInfo",
          "startIndex": 5,
          "endIndex": 24,
          "score": 0.9607566
        }
      ],
      "compositeEntities": [
        {
          "parentType": "TransferEmployeeInfo",
          "value": "jill jones to devops",
          "children": [
            {
              "type": "builtin.personName",
              "value": "jill jones"
            },
            {
              "type": "Department",
              "value": "devops"
            }
          ]
        }
      ]
    }
    ```

   此话语返回复合实体数组。 每个实体都有类型和值。 若要查找每个子实体的更高精度，请使用复合数组项中的类型和值的组合来查找实体数组中的相应项。  

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="related-information"></a>相关信息

* [列表实体教程](luis-quickstart-intents-only.md)
* [复合实体](luis-concept-entity-types.md)概念信息
* [如何训练](luis-how-to-train.md)
* [如何发布](luis-how-to-publish-app.md)
* [如何在 LUIS 门户中测试](luis-interactive-test.md)


## <a name="next-steps"></a>后续步骤

本教程创建了一个复合实体来封装现有实体。 这使得客户端应用程序能够发现一组不同数据类型的相关数据以便继续进行对话。 此人力资源应用的客户端应用程序可能会询问需要在哪一天的什么时间开始和结束搬迁。 它还可能会询问有关本次搬迁的其他后勤信息，例如实际电话。 

> [!div class="nextstepaction"] 
> [了解如何使用短语列表添加简单实体](luis-quickstart-primary-and-secondary-data.md)  
