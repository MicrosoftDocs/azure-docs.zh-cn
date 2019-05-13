---
title: 批处理测试
titleSuffix: Azure Cognitive Services
description: 本教程演示如何使用批处理测试在应用中查找话语预测问题并进行修复。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 03/29/2019
ms.author: diberry
ms.openlocfilehash: af04ca19961abcfc7ee218824a4a1a804f7ad79c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146162"
---
# <a name="tutorial-batch-test-data-sets"></a>教程：成批测试数据集

本教程演示如何使用批处理测试在应用中查找话语预测问题并进行修复。  

批测试允许使用一组已知的已标记话语和实体来验证活动的定型模型的状态。 在 JSON 格式的批处理文件中，添加话语并设置要在话语中预测的所需实体标签。 

批处理测试的要求：

* 每个测试的最大话语量为 1000 个。 
* 没有重复项。 
* 允许实体类型： 简单和复合的只有加工获知的实体。 批处理测试仅适用于机器学习意向和实体。

使用本教程以外的应用时，请不要使用已经添加到意向的示例话语。 

**本教程介绍如何执行下列操作：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 导入示例应用
> * 创建批处理测试文件 
> * 运行批处理测试
> * 查看测试结果
> * 修复错误 
> * 重新测试批处理

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="import-example-app"></a>导入示例应用

继续使用上一个教程中创建的名为 **HumanResources** 的应用。 

请执行以下步骤：

1.  下载并保存[应用 JSON 文件](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/tutorials/custom-domain-review-HumanResources.json)。

2. 将 JSON 导入到新应用中。

3. 在“管理”部分的“版本”选项卡上，克隆版本并将其命名为 `batchtest`。 克隆非常适合用于演练各种 LUIS 功能，且不会影响原始版本。 由于版本名称用作 URL 路由的一部分，因此该名称不能包含任何在 URL 中无效的字符。 

4. 将应用定型。

## <a name="batch-file"></a>批处理文件

1. 在文本编辑器中创建 `HumanResources-jobs-batch.json` 或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/tutorials/HumanResources-jobs-batch.json)它。 

2. 在 JSON 格式的批处理文件中，使用想要在测试中预测的意向添加话语。 

   [!code-json[Add the intents to the batch test file](~/samples-luis/documentation-samples/tutorials/HumanResources-jobs-batch.json "Add the intents to the batch test file")]

## <a name="run-the-batch"></a>运行批处理

1. 选择顶部导航栏的“测试”。 

2. 选择右侧面板中的“批处理测试面板”。 

    [![LUIS 应用的屏幕截图，其中突出显示了“批处理测试面板”](./media/luis-tutorial-batch-testing/hr-batch-testing-panel-link.png)](./media/luis-tutorial-batch-testing/hr-batch-testing-panel-link.png#lightbox)

3. 选择“导入数据集”。

    [![LUIS 应用的屏幕截图，其中突出显示了“导入数据集”](./media/luis-tutorial-batch-testing/hr-import-dataset-button.png)](./media/luis-tutorial-batch-testing/hr-import-dataset-button.png#lightbox)

4. 选择 `HumanResources-jobs-batch.json` 文件的文件位置。

5. 命名数据集 `intents only`，然后选择“完成”。

    ![选择文件](./media/luis-tutorial-batch-testing/hr-import-new-dataset-ddl.png)

6. 选择“运行”按钮。 

7. 选择“查看结果”。

8. 查看图和图例中的结果。

    [![LUIS 应用的批处理测试结果的屏幕截图](./media/luis-tutorial-batch-testing/hr-intents-only-results-1.png)](./media/luis-tutorial-batch-testing/hr-intents-only-results-1.png#lightbox)

## <a name="review-batch-results"></a>查看批处理结果

批处理图表将结果显示在四个象限中。 在图表右侧是一个筛选器。 默认情况下，筛选器设置为列表中的第一个意向。 筛选器包含所有意图和仅简单和复合实体。 选择[图表的一个部分](luis-concept-batch-test.md#batch-test-results)或图表中的一个点时，关联的话语显示在图表下方。 

鼠标悬停在图表上时，鼠标滚轮可以放大或缩小图表中的显示。 当图表上有许多点紧密地聚集在一起时，这是非常有用的。 

图表分为四个象限，其中两个部分以红色显示。 这些是要关注的部分。 

### <a name="getjobinformation-test-results"></a>GetJobInformation 测试结果

显示在筛选器中的 GetJobInformation 测试结果显示四种预测中有 2 种是成功的。 选择右上象限上方的名称“误报”，查看图表下方的话语。 

![LUIS 批处理测试话语](./media/luis-tutorial-batch-testing/hr-applyforjobs-false-positive-results.png)

为什么两个话语被预测为 ApplyForJob，而不是正确的意向 GetJobInformation？ 两个意向在字词的选择和排列方式方面密切相关。 在复合实体中包装此外，ApplyForJob 的示例几乎是 GetJobInformation 的三倍。 示例话语的这种不平衡对 ApplyForJob 意向有利。 

请注意，这两个意向都有相同的错误计数。 一个意向中的错误预测也会影响另一个意向。 由于错误地预测了一个意向的话语，也错误地未预测另一个意向，因此二者都有错误。 

![LUIS 批处理测试筛选器错误](./media/luis-tutorial-batch-testing/hr-intent-error-count.png)

对应于“误报”部分中顶点的话语为 `Can I apply for any database jobs with this resume?` 和 `Can I apply for any database jobs with this resume?`。 对于第一个话语，单词 `resume` 仅在 ApplyForJob 中使用过。 对于第二个话语，单词 `apply` 仅在 ApplyForJob 意向中使用过。

## <a name="fix-the-app"></a>修复应用

本部分的目标是通过修复应用，正确预测 GetJobInformation 的所有话语。 

一个看似快速的解决方法是将这些批处理文件话语添加到正确的意向。 但这不是你想要做的。 你想让 LUIS 正确地预测这些话语，而无需将其添加为示例。 

可能还想知道如何从 ApplyForJob 中删除话语，直到话语数量与 GetJobInformation 中相同。 这可能会修复测试结果，但会阻碍 LUIS 下一次准确地预测该意向。 

第一个解决方法是向 GetJobInformation 添加更多话语。 第二个解决方法是减少针对 ApplyForJob 意向的单词（如 `resume` 和 `apply`）的权重。 

### <a name="add-more-utterances"></a>添加更多话语

1. 选择顶部导航面板中的“测试”按钮，关闭批处理测试面板。 

2. 从意向列表中选择“GetJobInformation”。 

3. 添加更多长度、字词选择和排列方式不同的话语，确保包含术语 `resume`、`c.v.` 和 `apply`：

    |GetJobInformation 意向的示例话语|
    |--|
    |仓库中新的存货管理员工作是否要求通过简历申请？|
    |目前哪里有屋顶工作？|
    |我听说有一份医疗编码工作需要简历。|
    |我想找份工作帮助大学生编写简历。 |
    |这是我的简历，我正在使用计算机在社区大学找一份新的工作。|
    |有哪些儿童和家庭护理方面的职位可供选择？|
    |报社是否需要实习生？|
    |我的简历 显示了我擅长分析采购、预算和亏损。 是否有此类工作？|
    |现在是否有钻地工作？|
    |我当了 8 年 EMS 司机。 是否有新工作？|
    |有新的食物处理工作需要申请吗？|
    |有多少新的庭院工作岗位？|
    |是否有新的劳资关系和谈判 HR 职位？|
    |我有图书馆和档案管理硕士学位。 是否有任何新职位？|
    |今天城市中有照顾 13 岁儿童的保姆工作吗？|

    不要在话语中标记“工作”实体。 本教程的本部分仅侧重于意向预测。

4. 通过选择右上角导航中的“定型”来定型应用。

## <a name="verify-the-new-model"></a>验证新模型

为了验证批处理测试中的话语是否被正确地预测，请再次运行批处理测试。

1. 选择顶部导航栏的“测试”。 如果批处理结果仍处于打开状态，请选择“返回到列表”。  

2. 选择批处理名称右侧的省略号 (...) 按钮，然后选择“运行数据集”。 请等待批处理测试完成。 请注意，“查看结果”按钮现在为绿色。 这意味着整个批处理已成功运行。

3. 选择“查看结果”。 所有意向的名称左侧都应有绿色图标。 

    ![LUIS 的屏幕截图，其中已突出显示“批处理结果”按钮](./media/luis-tutorial-batch-testing/hr-batch-test-intents-no-errors.png)

## <a name="create-batch-file-with-entities"></a>使用实体创建批处理文件 

若要验证批处理测试中的实体，需要在批处理 JSON 文件中标记实体。 使用仅机器学习的内容的实体： 简单和复合实体。 不要添加非机器学习实体，因为它们总是通过正则表达式或显式文本匹配找到的。

总字（[令牌](luis-glossary.md#token)）计数的实体的变化会影响预测质量。 请确保提供给具有标记话语的意向的定型数据包括各种长度的实体。 

首次编写和测试批处理文件时，最好从知道有用的一些话语和实体以及认为可能错误预测的一些话语和实体开始。 这有助于快速专注于问题区域。 使用未预测的几个不同的“工作”名称测试 GetJobInformation 和 ApplyForJob 意向之后，开发了此批处理测试文件，用于查看“工作”实体的某些值是否存在预测问题。 

测试话语中提供的“工作”实体的值通常是一个或两个词，其中有几个示例有更多词。 如果自己的人力资源应用通常有多个词的工作名称，该应用中带有“工作”实体标记的示例话语将无法正常工作。

1. 在文本编辑器（如 [VSCode](https://code.visualstudio.com/)）中创建 `HumanResources-entities-batch.json`，或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/tutorials/HumanResources-entities-batch.json)它。


2. 在 JSON 格式的批处理文件中，添加一个对象数组，其中包含具有想要在测试中预测的意向的话语以及话语中任何实体的位置。 由于实体是基于令牌的，因此请确保启动和停止字符上的每个实体。 不要以空格开始或结束话语。 这会导致批处理文件导入过程中出现错误。  

   [!code-json[Add the intents and entities to the batch test file](~/samples-luis/documentation-samples/tutorials/HumanResources-entities-batch.json "Add the intents and entities to the batch test file")]


## <a name="run-the-batch-with-entities"></a>使用实体运行批处理

1. 选择顶部导航栏的“测试”。 

2. 选择右侧面板中的“批处理测试面板”。 

3. 选择“导入数据集”。

4. 选择 `HumanResources-entities-batch.json` 文件的文件系统位置。

5. 命名数据集 `entities`，然后选择“完成”。

6. 选择“运行”按钮。 请等待测试完成。

7. 选择“查看结果”。

[!INCLUDE [Entity roles in batch testing - currently not supported](../../../includes/cognitive-services-luis-roles-not-supported-in-batch-testing.md)]

## <a name="review-entity-batch-results"></a>查看实体批处理结果

图表打开，所有意向都已正确预测。 向下滚动以查找具有错误的实体预测在右侧的筛选器中。 

1. 在筛选器中选择“工作”实体。

    ![筛选器中的错误实体预测](./media/luis-tutorial-batch-testing/hr-entities-filter-errors.png)

    图表更改以显示实体预测。 

2. 选择图表左下象限的“漏报”。 然后使用组合键 control + E 切换到令牌视图。 

    [![实体预测的令牌视图](./media/luis-tutorial-batch-testing/token-view-entities.png)](./media/luis-tutorial-batch-testing/token-view-entities.png#lightbox)
    
    查看图表下方的话语会发现，当“工作”名称包括 `SQL` 时，会出现一致性错误。 查看示例话语和“工作”短语列表，SQL 仅使用一次，并且仅作为较大工作名称的一部分，`sql/oracle database administrator`。

## <a name="fix-the-app-based-on-entity-batch-results"></a>基于实体批处理结果修复应用

修复应用需要 LUIS 正确地确定 SQL 作业的变体。 对于该修补程序有多个选项。 

* 显式添加更多示例话语，使用 SQL 并将这些词标记为“工作”实体。 
* 将更多 SQL 作业显式添加到短语列表

这些任务会保留以供执行。

在正确预测实体之前添加[模式](luis-concept-patterns.md)不会解决该问题。 这是因为在检测到模式中的所有实体之前，该模式将不会匹配。 

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>后续步骤

本教程使用了批处理测试来查找当前模型存在的问题。 修复了模型并使用批处理文件重新测试以验证更改是否正确。

> [!div class="nextstepaction"]
> [了解模式](luis-tutorial-pattern.md)

