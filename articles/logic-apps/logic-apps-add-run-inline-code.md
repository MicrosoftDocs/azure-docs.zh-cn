---
title: 添加和运行代码片段-Azure 逻辑应用
description: 添加和使用 Azure 逻辑应用中的内联代码运行代码片段
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: derek1ee, LADocs
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 9ef11eb2099ff617fb4da4b9a924dc3f0550f226
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160540"
---
# <a name="add-and-run-code-snippets-by-using-inline-code-in-azure-logic-apps"></a>添加和运行代码片段通过 Azure 逻辑应用中使用内联代码

在你想要运行一段代码在逻辑应用内，你可以添加内置**内联代码**作为逻辑应用的工作流中的步骤操作。 此操作最适合你想要运行适合此方案的代码：

* 在 JavaScript 中运行。 即将推出的更多语言。
* 完成运行以五秒或更少。
* 处理数据的大小为 50 MB。
* 使用 Node.js 版本 8.11.1。 有关详细信息，请参阅[标准内置对象](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects)。

此操作运行的代码段，从该代码段将返回输出作为名为的令牌**结果**，可以在后续操作中使用逻辑应用中。 对于其他方案中你想要创建为你的代码的函数，请尝试[创建，然后调用 Azure 函数](../logic-apps/logic-apps-azure-functions.md)在逻辑应用中。

在本文中，示例逻辑应用触发器时的新电子邮件到达的 Office 365 Outlook 帐户。 代码片段提取，并在电子邮件正文中返回显示任何电子邮件地址。

![示例概述](./media/logic-apps-add-run-inline-code/inline-code-example-overview.png)

## <a name="prerequisites"></a>必备组件

* Azure 订阅。 如果没有 Azure 订阅，请[注册一个免费 Azure 帐户](https://azure.microsoft.com/free/)。

* 想要添加您的代码段，其中包括触发器逻辑应用。 如果您没有逻辑应用，请参阅[快速入门：创建第一个逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

   本主题中的示例逻辑应用将使用此 Office 365 Outlook 触发器：**新电子邮件到达时**

* [集成帐户](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)链接到逻辑应用

## <a name="add-inline-code"></a>添加内联代码

1. 如果你尚未准备好，在[Azure 门户](https://portal.azure.com)，在逻辑应用设计器中打开逻辑应用。

1. 在设计器中，添加**内联代码**操作在逻辑应用的工作流中所需的位置。

   * 若要在工作流的末尾添加操作，请选择**新步骤**。

   * 若要添加的现有步骤之间的操作，将鼠标指针移动连接这些步骤的箭头。 选择加号 (**+**)，然后选择**添加操作**。

   此示例将添加**内联代码**的 Office 365 Outlook 触发器下的操作。

   ![添加新步骤](./media/logic-apps-add-run-inline-code/add-new-step.png)

1. 下**选择操作**，在搜索框中，输入"内联代码"作为筛选器。 在操作列表中选择此操作：**执行的 JavaScript 代码**

   ![选择"执行的 JavaScript 代码"](./media/logic-apps-add-run-inline-code/select-inline-code-action.png)

   操作在设计器中出现，其中包含一些默认的示例代码，包括返回语句。

   ![使用默认示例代码的内联代码操作](./media/logic-apps-add-run-inline-code/inline-code-action-default.png)

1. 在中**代码**框中，删除示例代码中，然后输入你想要运行的代码。 编写可使在方法中，但无需定义方法签名的代码。 

   当你键入是可识别的关键字时，记忆式键入功能列表将显示，以便你可以选择从可用关键字，例如：

   ![关键字自动完成列表](./media/logic-apps-add-run-inline-code/auto-complete.png)

   此示例代码片段首先创建变量，用于存储*正则表达式*，它指定在输入文本匹配的模式。 该代码随后创建存储的触发器中的电子邮件正文数据的变量。

   ![创建变量](./media/logic-apps-add-run-inline-code/save-email-body-variable.png)

   若要使触发器和以前的操作的结果更轻松地引用，动态内容列表显示时光标位于内**代码**框。 有关此示例中，列表将显示可用的触发器中的结果包括**正文**令牌，现在可以选择。

   选择后**正文**令牌，内联代码操作解析到的令牌`workflowContext`对象，它引用电子邮件的`Body`属性值：

   ![选择结果](./media/logic-apps-add-run-inline-code/inline-code-example-select-outputs.png)

   在中**代码**框中，您的代码段可以使用只读的`workflowContext`对象作为输入。 此对象具有子属性，允许代码访问到结果中的触发器和工作流中的上一操作。
   有关详细信息，请参阅本主题后面的此部分：[引用在代码中的触发器和操作结果](#workflowcontext)。

   > [!NOTE]
   >
   > 如果您的代码段引用使用点 （.） 运算符的操作名称，则必须添加到这些操作名称[**操作**参数](#add-parameters)。 这些引用，还必须将在操作名称之间使用方括号 ([]) 和引号，例如：
   >
   > `// Correct`</br> 
   > `workflowContext.actions["my.action.name"].body`</br>
   >
   > `// Incorrect`</br>
   > `workflowContext.actions.my.action.name.body`

   内联代码操作不需要`return`语句，但从结果`return`语句是可用于通过更高版本的操作中引用**结果**令牌。 
   例如，代码段将返回通过调用结果`match()`函数，哪些查找匹配的正则表达式对电子邮件正文中。 **Compose**操作使用**结果**令牌来引用从内联结果代码操作和创建单个结果。

   ![完成的逻辑应用](./media/logic-apps-add-run-inline-code/inline-code-complete-example.png)

1. 完成后，保存逻辑应用。

<a name="workflowcontext"></a>

### <a name="reference-trigger-and-action-results-in-your-code"></a>在代码中引用的触发器和操作结果

`workflowContext`对象具有以下结构，其中包括`actions`， `trigger`，和`workflow`子属性：

```json
{
   "workflowContext": {
      "actions": {
         "<action-name-1>": @actions('<action-name-1>'),
         "<action-name-2>": @actions('<action-name-2>')
      },
      "trigger": {
         @trigger()
      },
      "workflow": {
         @workflow()
      }
   }
}
```

此表包含有关这些子属性的详细信息：

| 属性 | Type | 描述 |
|----------|------|-------|
| `actions` | 对象集合 | 从您的代码段用之前运行的操作的结果对象。 每个对象都*键值*对，其中，键是操作的名称和值相当于调用[actions() 函数](../logic-apps/workflow-definition-language-functions-reference.md#actions)与`@actions('<action-name>')`。 操作的名称使用相同的操作名称将空格替换的基础工作流定义中使用 ("") 中使用下划线 (_) 的操作名称。 此对象提供访问权限的操作属性值从当前运行的工作流实例。 |
| `trigger` | Object | 从触发器和等效于调用的结果对象[trigger() 函数](../logic-apps/workflow-definition-language-functions-reference.md#trigger)。 此对象提供对触发器属性值的访问，从当前运行的工作流实例。 |
| `workflow` | Object | 工作流对象，等效于调用[workflow （） 函数](../logic-apps/workflow-definition-language-functions-reference.md#workflow)。 此对象提供访问权限的工作流属性值，例如工作流名称、 运行的 ID 等，从当前运行的工作流实例。 |
|||

在本主题的示例中，`workflowContext`对象具有您的代码可以访问这些属性：

```json
{
   "workflowContext": {
      "trigger": {
         "name": "When_a_new_email_arrives",
         "inputs": {
            "host": {
               "connection": {
                  "name": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Web/connections/office365"
               }
            },
            "method": "get",
            "path": "/Mail/OnNewEmail",
            "queries": {
               "includeAttachments": "False"
            }
         },
         "outputs": {
            "headers": {
               "Pragma": "no-cache",
               "Content-Type": "application/json; charset=utf-8",
               "Expires": "-1",
               "Content-Length": "962095"
            },
            "body": {
               "Id": "AAMkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgBGAAAAAABmZwxUQtCGTqSPpjjMQeD",
               "DateTimeReceived": "2019-03-28T19:42:16+00:00",
               "HasAttachment": false,
               "Subject": "Hello World",
               "BodyPreview": "Hello World",
               "Importance": 1,
               "ConversationId": "AAQkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgAQ",
               "IsRead": false,
               "IsHtml": true,
               "Body": "Hello World",
               "From": "<sender>@<domain>.com",
               "To": "<recipient-2>@<domain>.com;<recipient-2>@<domain>.com",
               "Cc": null,
               "Bcc": null,
               "Attachments": []
            }
         },
         "startTime": "2019-05-03T14:30:45.971564Z",
         "endTime": "2019-05-03T14:30:50.1746874Z",
         "scheduledTime": "2019-05-03T14:30:45.8778117Z",
         "trackingId": "1cd5ffbd-f989-4df5-a96a-6e9ce31d03c5",
         "clientTrackingId": "08586447130394969981639729333CU06",
         "originHistoryName": "08586447130394969981639729333CU06",
         "code": "OK",
         "status": "Succeeded"
      },
      "workflow": {
         "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>",
         "name": "<logic-app-workflow-name>",
         "type": "Microsoft.Logic/workflows",
         "location": "<Azure-region>",
         "run": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>/runs/08586453954668694173655267965CU00",
            "name": "08586453954668694173655267965CU00",
            "type": "Microsoft.Logic/workflows/runs"
         }
      }
   }
}
```

<a name="add-parameters"></a>

## <a name="add-parameters"></a>添加参数

在某些情况下，可能需要显式要求**内联代码**操作包括从您的代码通过添加为依赖项引用的触发器或特定操作的结果**触发器**或**操作**参数。 此选项可用于在运行时找到引用的结果不是其中的方案。

> [!TIP]
> 如果计划重用您的代码，通过使用添加到属性的引用**代码**框，这样你的代码包含解析令牌引用，而不是添加触发器或操作作为显式依赖关系。

例如，假定你具有引用的代码**SelectedOption**源自**发送审批电子邮件**Office 365 Outlook 连接器的操作。 在创建时，逻辑应用引擎将分析代码以确定是否已引用的任何触发器或操作结果和自动包括这些结果。 在运行时，应收到错误中指定的被引用的触发器或操作结果没有`workflowContext`对象，可以将该触发器或操作添加为显式依赖关系。 在此示例中，您将添加**操作**参数以及指定那些**内联代码**操作中显式包括从结果**发送审批电子邮件**操作。

若要添加这些参数，请打开**添加新参数**列表，然后选择所需的参数：

   ![添加参数](./media/logic-apps-add-run-inline-code/inline-code-action-add-parameters.png)

   | 参数 | 描述 |
   |-----------|-------------|
   | **操作** | 包含上一操作的结果。 请参阅[包含操作结果](#action-results)。 |
   | **触发器** | 包括来自触发器的结果。 请参阅[包括触发器结果](#trigger-results)。 |
   |||

<a name="trigger-results"></a>

### <a name="include-trigger-results"></a>包括触发器的结果

如果选择**触发器**，系统会提示你是否包括触发器结果。

* 从**触发器**列表中，选择**是**。

<a name="action-results"></a>

### <a name="include-action-results"></a>包含操作结果

如果选择**操作**，系统会提示你想要添加的操作。 但是，在开始添加操作之前，需要将出现在逻辑应用的基础工作流定义的操作名称的版本。

* 此功能不支持变量、 循环和迭代索引。

* 在逻辑应用的工作流定义中的名称使用下划线 (_)，未一个空格。

* 有关使用点运算符 （.） 的操作名称，包括这些运算符，例如：

  `My.Action.Name`

1. 在设计器工具栏上，选择**代码视图**，并在搜索`actions`操作名称的属性。

   例如，`Send_approval_email_`的 JSON 名称**发送审批电子邮件**操作。

   ![在 JSON 中查找操作名称](./media/logic-apps-add-run-inline-code/find-action-name-json.png)

1. 若要返回到设计器视图中，在代码视图工具栏上，选择**设计器**。

1. 若要添加的第一个操作，在**操作项-1**框中，输入操作的 JSON 名称。

   ![输入第一个操作](./media/logic-apps-add-run-inline-code/add-action-parameter.png)

1. 若要添加另一项操作，请选择**添加新项**。

## <a name="reference"></a>参考

有关详细信息**执行的 JavaScript 代码**操作的结构和语法在逻辑应用的基础工作流定义中使用工作流定义语言，请参阅此操作的[参考部分](../logic-apps/logic-apps-workflow-actions-triggers.md#run-javascript-code).

## <a name="next-steps"></a>后续步骤

详细了解[适用于 Azure 逻辑应用连接器](../connectors/apis-list.md)
