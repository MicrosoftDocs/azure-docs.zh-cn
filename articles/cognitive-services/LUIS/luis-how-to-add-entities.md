---
title: 添加实体
titleSuffix: Language Understanding - Azure Cognitive Services
description: 创建实体来从用户语言理解 (LUIS) 应用中的查询文本中提取关键数据。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 04/01/2019
ms.author: diberry
ms.openlocfilehash: 241e89ac7fa78184e7c55f9e8065e1534cea9143
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148733"
---
# <a name="create-entities-without-utterances"></a>创建不包含话语的实体

实体表示要提取的话语中的字词或短语。 实体表示包括一系列类似对象 （位置、 内容、 人员、 事件或概念） 的类。 实体描述与意向相关的信息，它们有时对于应用执行任务至关重要。 （之前或之后） 中添加查询文本转意向或间隔时，可以创建实体添加到意向的语音样本。

可在 LUIS 应用中通过“实体”页上的“实体列表”来编辑或删除实体。 LUIS 提供两种主要的实体类型：[预生成实体](luis-reference-prebuilt-entities.md)和自己的[自定义实体](luis-concept-entity-types.md#types-of-entities)。

创建机器学习的内容的实体时，需要将标记该实体中的所有意图的所有示例查询文本中。

<a name="add-prebuilt-entity"></a>

## <a name="add-a-prebuilt-entity-to-your-app"></a>将预生成的实体添加到您的应用程序

添加到应用程序的常见预生成实体为 number 和 datetimeV2。 

1. 在应用中，从“生成”部分的左侧面板中选择“实体”。
 
1. 在“实体”页上，选择“添加预生成实体”。

1. 在“添加预生成实体”对话框中，选择 **number** 和 **datetimeV2** 预生成实体。 然后选择“完成”。

    ![“添加预生成实体”对话框屏幕截图](./media/add-entities/list-of-prebuilt-entities.png)

<a name="add-simple-entities"></a>

## <a name="add-simple-entities-for-single-concepts"></a>添加单个概念的简单实体

简单实体描述单一概念。 使用以下过程创建一个实体，用于提取“人力资源”或“运营”等公司部门名称。   

1. 在应用中选择“生成”部分，在左侧面板中选择“实体”，然后选择“创建新实体”。

1. 在弹出对话框中，在“实体名称”框中键入 `Location`，从“实体类型”列表选择“简单”，然后选择“完成”。

    创建此实体后，转到包含该实体的示例话语所在的所有意向。 选择示例话语中的文本，并将文本标记为实体。 

    [短语列表](luis-concept-feature.md)通常用于提升简单实体的信号。

<a name="add-regular-expression-entities"></a>

## <a name="add-regular-expression-entities-for-highly-structured-concepts"></a>添加高度结构化概念的正则表达式实体

基于提供的正则表达式，正则表达式实体用来从话语中拉取数据。 

1. 在应用中，从左侧导航栏选择“实体”，然后选择“创建新实体”。

1. 在弹出对话框中，在“实体名称”框中输入 `Human resources form name`，从“实体类型”列表中选择“正则表达式”，输入正则表达式 `hrf-[0-9]{6}`，然后选择“完成”。 

    此正则表达式匹配的原义字符`hrf-`，然后使用 6 位数来表示窗体号人力资源窗体。

<a name="add-composite-entities"></a>

## <a name="add-composite-entities-to-group-into-a-parent-child-relationship"></a>添加复合实体分组到父-子关系

可以通过创建复合实体定义不同类型的实体之间的关系。 在以下示例中，实体包含正则表达式和预生成的名称实体。  

在话语 `Send hrf-123456 to John Smith` 中，文本 `hrf-123456` 将与人力资源[正则表达式](#add-regular-expression-entities)匹配，并提取包含预生成实体 personName 的 `John Smith`。 每个实体是更大父实体的一部分。 

1. 在应用中，从“生成”部分的左侧导航栏中选择“实体”，然后选择“添加预生成实体”。

1. 添加预生成实体 **PersonName**。 有关说明，请参阅[添加预生成实体](#add-prebuilt-entity)。 

1. 从左侧导航栏选择“实体”，然后选择“创建新实体”。

1. 在弹出对话框的“实体名称”框中输入 `SendHrForm`，然后从“实体类型”列表选择“复合”。

1. 选择“添加子级”，添加新的子级。

1. 在“子级 #1”中，从列表选择实体 number。

1. 在“子级 #2”中，从列表中选择实体“人力资源表单名称”。 

1. 选择“完成”。

<a name="add-pattern-any-entities"></a>

## <a name="add-patternany-entities-to-capture-free-form-entities"></a>添加 Pattern.any 捕获自由格式的实体的实体

[Pattern.any](luis-concept-entity-types.md) 实体仅在[模式](luis-how-to-model-intent-pattern.md)中有效，在意向中无效。 此类实体有助于 LUIS 查找不同长度和字词选择的实体结尾。 由于此实体会在模式中使用，因此，LUIS 可识别实体末尾在话语模板中的位置。

如果应用包含 `FindHumanResourcesForm` 意向，则提取的表单标题可能会干扰意向预测。 若要澄清表单标题中的字词，请在模式中使用 Pattern.any。 LUIS 预测会从话语开始。 首先，针对实体检查话语并进行匹配，找到实体时，接着检查和匹配模式。 

在话语 `Where is Request relocation from employee new to the company on the server?` 中，表单标题比较棘手，因为标题结束和其他话语开始的位置不是很明显。 标题可以是任意顺序的字词，包括单字、有标点的复杂短语和无意义排序的字词。 通过模式，可以在可提取完整、确切实体的位置创建实体。 找到标题后，预测 `FindHumanResourcesForm` 意向，因为这是模式的意向。

1. 在“生成”部分的左侧面板中选择“实体”，然后选择“创建新实体”。

1. 在“添加实体”对话框的“实体名称”框中输入 `HumanResourcesFormTitle`，然后选择“Pattern.any”作为“实体类型”。

    若要使用 pattern.any 实体，请在“模式”页的“提升应用性能”部分中，通过正确的大括号语法（例如 `Where is **{HumanResourcesFormTitle}** on the server?`）添加模式。

    如果发现模式在包含 Pattern.any 时错误提取实体，请使用[显式列表](luis-concept-patterns.md#explicit-lists)来更正此问题。 

<a name="add-a-role-to-pattern-based-entity"></a>

## <a name="add-a-role-to-distinguish-different-contexts"></a>添加角色来区分不同的上下文

角色是基于上下文在命名子类型。 它现已推出包括预先生成和机器学习实体的所有实体。 

为角色的语法是**`{Entityname:Rolename}`** 其中实体名称后跟一个冒号，则角色名称。 例如，`Move {personName} from {LocationUsingRoles:Origin} to {LocationUsingRoles:Destination}`。

1. 在“生成”部分的左侧面板中选择“实体”。

1. 选择“创建新实体”。 输入 `LocationUsingRoles` 名称。 选择类型“简单”，然后选择“完成”。 

1. 在左侧面板中选择“实体”，然后选择在前一步骤中创建的新实体 **LocationUsingRoles**。

1. 在“角色名称”文本框中，输入角色 `Origin` 的名称并按 Enter。 添加 `Destination` 的第二个角色名。 

    ![将 Origin 角色添加到 Location 实体的屏幕截图](./media/add-entities/roles-enter-role-name-text.png)

<a name="add-list-entities"></a>

## <a name="add-list-entities-for-exact-matches"></a>添加列表实体的完全匹配项

列表实体表示一组固定、封闭的相关单词。 

对于人力资源应用，可以创建所有部门以及任何部门同义词的列表。 创建实体时，不需了解所有值。 可以在查看真实的用户表述后使用同义词添加更多值。

1. 在“生成”部分的左侧面板中选择“实体”，然后选择“创建新实体”。

1. 在“添加实体”对话框中，在“实体名称”框中键入 `Department`，然后选择“列表”作为“试题类型”。 选择“完成”。
  
1. 通过此列表实体页可以添加规范化名称。 在“值”文本中，输入列表中的某个部门名称（例如 `HumanResources`），然后按键盘上的 Enter。 

1. 在规范化值的右侧输入同义词，并在输入每个项后按键盘上的 Enter。

1. 如果想要更多适用于此列表的规范化项，请选择“建议”，查看[语义字典](luis-glossary.md#semantic-dictionary)的选项。

    ![选择推荐功能以查看选项的屏幕截图](./media/add-entities/hr-list-2.png)


1. 在建议列表中选择一项添加为规范化值，或者选择“全部添加”添加所有项。 
    可使用以下 JSON 格式将值导入现有列表实体：

    ```JSON
    [
        {
            "canonicalForm": "Blue",
            "list": [
                "navy",
                "royal",
                "baby"
            ]
        },
        {
            "canonicalForm": "Green",
            "list": [
                "kelly",
                "forest",
                "avacado"
            ]
        }
    ]  
    ```

<a name="change-entity-type"></a>

## <a name="do-not-change-entity-type"></a>不能更改实体类型

LUIS 不允许更改实体类型，因为它不知道构造该实体要添加或删除的内容。 若要更改类型，最好创建一个名称稍微不同的、类型正确的新实体。 实体创建后，在每句话语中，删除旧标记实体名称并添加新的实体名称。 重新标记所有话语后，即可删除旧实体。 

<a name="create-a-pattern-from-an-utterance"></a>

## <a name="create-a-pattern-from-an-example-utterance"></a>创建示例查询文本从一种模式

请参阅[在意向或实体页上通过现有表述添加模式](luis-how-to-model-intent-pattern.md#add-pattern-from-existing-utterance-on-intent-or-entity-page)。

## <a name="train-your-app-after-changing-model-with-entities"></a>在对模型进行实体更改后对应用进行定型

添加、编辑或删除实体后，请针对更改来[定型](luis-how-to-train.md)和[发布](luis-how-to-publish-app.md)应用以影响终结点查询。 

## <a name="next-steps"></a>后续步骤

有关预生成实体的详细信息，请参阅 [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 项目。 

有关实体在 JSON 终结点查询响应中的显示方式的信息，请参阅[数据提取](luis-concept-data-extraction.md)

因为已添加意向、话语和实体，所以有了基本的 LUIS 应用。 了解如何[定型](luis-how-to-train.md)、[测试](luis-interactive-test.md)和[发布](luis-how-to-publish-app.md)应用。
 
