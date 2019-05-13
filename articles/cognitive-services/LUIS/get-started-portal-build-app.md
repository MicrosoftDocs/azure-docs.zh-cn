---
title: 快速入门：在 LUIS 门户中创建新应用
titleSuffix: Language Understanding - Azure Cognitive Services
description: 本快速入门介绍如何使用 LUIS 门户创建新应用。 创建应用的基本部件、意向和实体。 通过在交互式测试面板中提供用于获取预测的意向的示例用户话语，对应用进行测试。 生成应用是免费的操作，不需要 Azure 订阅。
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: quickstart
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 2e4ff7dc97e3ee72336bd4c081caf1aa1a62bc56
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146543"
---
# <a name="quickstart-create-a-new-app-in-the-luis-portal"></a>快速入门：在 LUIS 门户中创建新应用

本快速入门将在 [LUIS 门户](https://www.luis.ai)中生成新应用。 首先创建应用的基本部件、**意向**和**实体**。 然后，通过在交互式测试面板中提供用于获取预测的意向的示例用户话语，对应用进行测试。

生成应用是免费的操作，不需要 Azure 订阅。 做好部署应用的准备以后，请参阅[有关如何部署应用的快速入门](get-started-portal-deploy-app.md)。 该快速入门介绍如何创建 Azure 认知服务资源并将其分配到应用。

## <a name="create-an-app"></a>创建应用

1. 在浏览器中打开 [LUIS 门户](https://www.luis.ai)并登录。 首次登录时，需要创建一个免费的 LUIS 门户用户帐户。

1. 在上下文工具栏中选择“创建新应用”。

   [![在 LUIS 门户中创建新应用](./media/get-started-portal-build-app/create-app-in-portal.png)](./media/get-started-portal-build-app/create-app-in-portal.png#lightbox)

1. 在弹出窗口中使用以下设置配置应用，然后选择“完成”。

   |设置名称| 值 | 目的|
   |--|--|--|
   |名称|`myEnglishApp`|唯一的 LUIS 应用名称<br>必填|
   |环境|**英语**|用户的言语语言：**en-us**<br>必填|
   |说明|`App made with LUIS Portal`|应用的说明<br>可选|
   | | | |

   ![输入新应用设置](./media/get-started-portal-build-app/create-new-app-settings.png)

## <a name="create-intents"></a>创建意向

创建 LUIS 应用后，需创建意向。 意向用于分类用户提供的文本。 例如，人力资源应用可以有两项功能。 帮助用户：

 1. 找工作和申请工作
 1. 查找工作申请表

此应用的两个不同_意向_对应于以下意向：

|意向|用户提供的文本示例<br>称为“言语”|
|--|--|
|ApplyForJob|`I want to apply for the new software engineering position in Cairo.`|
|FindForm|`Where is the job transfer form hrf-123456?`|

若要创建意向，请完成以下步骤：

1. 创建应用后，会转到“生成”部分的“意向”页。 选择“创建新意向”。

   [![选择“创建新意向”按钮](./media/get-started-portal-build-app/create-new-intent-button.png)](./media/get-started-portal-build-app/create-new-intent-button.png#lightbox)

1. 输入意向名称 `FindForm`，然后选择“完成”。

   ![输入意向名称 FindForm](./media/get-started-portal-build-app/create-new-intent-dialog.png)

## <a name="add-an-example-utterance"></a>添加示例话语

在创建意向后添加示例话语。 示例话语是用户在聊天机器人或其他客户端应用程序中输入的文本。 它们可将用户文本的意向映射到 LUIS 意向。

就此示例应用程序的 `FindForm` 意向来说，示例话语将包含表格编号。 客户端应用程序需要该表格编号来履行用户的请求，因此必须将其包括在话语中。

[![输入 FindForm 意向的示例言语](./media/get-started-portal-build-app/add-example-utterance.png)](./media/get-started-portal-build-app/add-example-utterance.png#lightbox)

将以下 15 个示例话语添加到 `FindForm` 意向。

|#|示例陈述|
|--|--|
|1|正在查找 hrf-123456|
|2|人力资源表格 hrf-234591 在哪里？|
|3|hrf-345623 在哪里？|
|4|是否可以向我发送 hrf-345794|
|5|我是否需要使用 hrf-234695 来申请内部职位？|
|6|我的经理是否需要知道我正在使用 hrf-234091 申请职位|
|7|要将 hrf-234918 发送到何处？ 对方收到此表格后，我是否会收到电子邮件回复？|
|8|hrf-234555|
|9|hrf-234987 是何时更新的？|
|10|我是否要使用表格 hrf-876345 申请工程职位|
|11|是否为我的开放请求提交了 hrf-765234 的新版本？|
|12|我是否要使用 hrf-234234 申请国际职位？|
|13|hrf-234598 拼写错误|
|14|是否会根据新要求编辑 hrf-234567|
|15|hrf-123456、hrf-123123、hrf-234567|

根据设计，这些示例话语按以下方式变化：

* 言语长度
* 标点
* 选词
* 谓语时态（现在时、过去时和将来时）
* 词序



## <a name="create-a-regular-expression-entity"></a>创建正则表达式实体

若要在运行时预测响应中返回表格编号，必须将该表格标记为实体。 由于表格编号文本已高度结构化，可以使用正则表达式实体来标记它。 通过以下步骤创建该实体：

1. 在左侧菜单中选择“实体”。

1. 在“实体”页上选择“创建新实体”。

1. 输入名称 `Human Resources Form Number`，选择“正则表达式”实体类型，然后输入正则表达式 `hrf-[0-9]{6}`。 此条目匹配文本字符 `hrf-`，允许对刚好 6 位数进行匹配。

   ![输入正则表达式实体的实体信息](./media/get-started-portal-build-app/create-regular-expression-entity.png)

1. 选择“完成”。

## <a name="add-example-utterances-to-the-none-intent"></a>将话语示例添加到 None 意向

**None** 意向是回退意向，不可留空。 每为应用的其他意向添加 10 个示例话语，此意向就应该包含 1 个话语。

**None** 意向的示例言语应在客户端应用程序域的范围以外。

1. 在左侧菜单中选择“意向”，然后从意向列表中选择“None”。

1. 将以下示例言语添加到该意向：

   |None 意向示例言语|
   |--|
   |Barking dogs are annoying|
   |Order a pizza for me|
   |Penguins in the ocean|

   对于此人力资源应用，这些示例言语在域的范围以外。 如果人力资源域包括动物、食物或海洋，则应将其他示例话语用于 **None** 意向。

## <a name="train-the-app"></a>训练应用

在右上方的菜单中选择“训练”，将意向和实体模型更改应用到当前版本的应用。

## <a name="look-at-the-regular-expression-entity-in-the-example-utterances"></a>在示例言语中查找正则表达式实体

1. 在左侧菜单中选择“意向”，以验证是否能够在“FindForm”意向中找到该实体。 然后，选择“FindForm”意向。

   会标记该实体在示例言语中的显示位置。 若要查看原始文本而不是实体名称，请在工具栏中切换“实体视图”。

   [![标记有实体的所有示例言语](./media/get-started-portal-build-app/all-example-utterances-marked-with-entities.png)](./media/get-started-portal-build-app/all-example-utterances-marked-with-entities.png#lightbox)

## <a name="test-your-new-app-with-the-interactive-test-pane"></a>使用交互式测试窗格测试新应用

使用 LUIS 门户中的交互式“测试”窗格可以验证实体是否是从应用尚未看到的新话语中提取的。

1. 在右上方菜单中选择“测试”。

1. 添加新话语，然后按 Enter：

   ```Is there a form named hrf-234098```

   ![在测试窗格中测试新言语](./media/get-started-portal-build-app/test-new-utterance.png)

   排名靠前的预测意向为 **FindForm**（正确），其置信度超过 90% (0.977)。 已提取“人力资源表格编号”实体，其值为 hrf-234098。

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，如果不继续学习下一快速入门，请在顶部导航菜单中选择“我的应用”。 接着在列表中选中应用左侧的复选框，然后在列表上方的上下文工具栏中选择“删除”。

[![从“我的应用”列表中删除应用](./media/get-started-portal-build-app/delete-app.png)](./media/get-started-portal-build-app/delete-app.png#lightbox)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [2.部署应用](get-started-portal-deploy-app.md)
