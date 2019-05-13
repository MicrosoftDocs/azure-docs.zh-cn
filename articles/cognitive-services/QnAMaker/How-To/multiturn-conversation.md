---
title: 多打开会话
titleSuffix: Azure Cognitive Services
description: 使用提示和上下文来管理多个将，称为多轮次，到另一个问题从智能机器人。 多轮次是已备份的能力相互间会话的前一个问题的上下文会影响下一步的问题和答案。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 24158d5949c0a31e5444c609cbfced970bdeca07
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471970"
---
# <a name="use-follow-up-prompts-to-create-multiple-turns-of-a-conversation"></a>使用后续的提示操作以创建多个打开的会话

使用后续的提示和上下文来管理多个将，称为_多轮次_，到另一个问题从智能机器人。

从学习[演示视频](https://aka.ms/multiturnexample)。

## <a name="what-is-a-multi-turn-conversation"></a>什么是多轮次会话？

不能在一个单一回合中完成某些类型的会话。 在设计您的客户端应用程序 （聊天机器人） 对话时，用户可以询问一个问题就需要进行筛选和优化，以便确定正确的答案。 此流通过问题可通过提供与用户**跟进提示**。

当用户请求时的问题时，QnA Maker 返回答案_和_任何后续提示。 这可以提供为选择的后续问题。 

## <a name="example-multi-turn-conversation-with-chat-bot"></a>与聊天机器人的示例多打开会话

聊天机器人管理会话，问题通过问题，并可用于确定最终答案。

![中的对话式流，通过提供提示中显示为选项以继续此对话的答案管理多轮次对话框系统中会话状态。](../media/conversational-context/conversation-in-bot.png)

在上图中，用户的问题需要返回答案前将其优化。 中的知识，在问题 (#1) 包含四个后续的提示，显示在聊天机器人为四个选项 (#2)。 

当用户选择一个选择 (#3) 时，然后显示下一个列表中的优化选项 (#4)。 这可以继续 (#5)，直到确定正确的和最终答案 (6)。

您需要更改你的客户端应用程序管理上下文的会话。

## <a name="create-a-multi-turn-conversation-from-documents-structure"></a>从文档的结构创建多打开会话
当您创建某一知识库时，您将看到可选复选框以启用多轮次提取。 如果选择此选项，导入文档时，可以从结构隐式多打开会话。 如果存在该结构，QnA Maker 将为你创建的后续提示 QnA 对。 仅可以从 Url、 PDF 或 DOCX 推断出多轮次结构文件。 

下图的 Microsoft Surface [PDF 文件](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf)旨在用作手动。 

![![如果导入文档时，上下文会话可以是隐式从结构。 如果存在该结构，QnA Maker 的后续提示 QnA 对为用户创建，作为文档导入的一部分。](../ media/conversational-context/import-file-with-conversational-structure.png）](../media/conversational-context/import-file-with-conversational-structure.png#lightbox)

在导入 PDF 文档，QnA Maker 确定从该结构以创建对话式流的后续提示。 

![![在导入 PDF 文档，QnA Maker 确定从该结构以创建对话式流的后续提示。 ](../media/conversational-context/surface-manual-pdf-follow-up-prompt.png)](../media/conversational-context/surface-manual-pdf-follow-up-prompt.png#lightbox)

## <a name="filter-questions-and-answers-by-context"></a>筛选上下文中的问题和解答

1. 减少到只是那些具备上下文会话显示的问题和答案对。 选择**查看选项**，然后选择**显示上下文 （预览版）**。 添加带有后续提示符的第一个问题和答案对之前，列表将为空。 

    ![筛选的问题和回答上下文对话对](../media/conversational-context/filter-question-and-answers-by-context.png)

## <a name="add-new-qna-pair-as-follow-up-prompt"></a>添加新的 QnA 对后续提示

1. 选择**添加 QnA 对**。 
1. 输入新的问题文本`Give feedback.`使用的答案`What kind of feedback do you have?`。

1. 在中**答案**对于此问题，选择的列**添加后续提示符**。 
1. **跟进提示符**弹出对话框中，可搜索一个现有的问题或输入一个新问题。 在此过程中，输入的文本`Feedback on an QnA Maker service`，对于**显示文本**。 
1. 检查**仅限上下文的**。 **上下文限**选项指示此用户文本就很容易理解_仅_如果在上一步回答这个问题中提供。 此方案中，提示的文本没有任何意义作为一个独立的问题，才有意义的前一个问题上下文中。
1. 在**链接以回答**文字框中，输入答案， `How would you rate QnA Maker?`。
1. 选择**新建**然后选择**保存**。 

    ![创建新提示 QnA](../media/conversational-context/create-child-prompt-from-parent.png)

    这创建新的问题和答案对并将链接作为后续提示所选的问题。 **上下文**列中的，为这两个问题，指示后续提示关系。 

    ![![这两个问题的上下文列指示后续提示关系。](../ media/conversational-context/child-prompt-created.png)](../media/conversational-context/child-prompt-created.png#lightbox)

1. 选择**添加后续提示符**为`Give feedback`添加另一个后续提示的问题。 
1. 输入创建新问题`Feedback on an existing feature`，答案`Which feature would you like to give feedback on?`。  

1.  检查**仅限上下文的**。 **上下文限**选项指示此用户文本就很容易理解_仅_如果在上一步回答这个问题中提供。 此方案中，提示的文本没有任何意义作为一个独立的问题，才有意义的前一个问题上下文中。
1.  选择“保存”。 

    这创建一个新问题并链接到面对的提示问题作为问题`Give feedback`问题。
    
    在顶部的问题在此情况下，包含两个后续提示链接到前一个问题， `Give feedback`。

    ![![在此情况下，顶部的问题有两个后续提示链接到前一个问题，提供反馈。](../ media/conversational-context/all-child-prompts-created.png)](../media/conversational-context/all-child-prompts-created.png#lightbox)

1. 选择**保存并训练**定型知识库的新问题。 

## <a name="add-existing-qna-pair-as-follow-up-prompt"></a>将现有的 QnA 对添加为后续提示符

1. 如果你想要链接现有 QnA 对作为后续提示符下，选择的问题和答案对的行。
1. 选择**添加后续提示符**该行中。
1. 在弹出对话框中，输入在搜索框中的问题文本。 返回所有匹配项。 选择希望作为执行后续操作，并检查的问题**上下文限**，然后选择**保存**。 

    一旦 ou 已添加后续提示符下，务必选中**保存并训练**。
  
## <a name="add-metadata-to-follow-up-prompts"></a>将元数据添加到后续的提示 

中的知识时提示问题答案对链接到, 跟进提示、 元数据筛选器将首先应用，然后返回后续工作。

1. 对于两个后续 QnA 对中，将添加到每个的元数据：

    |问题|添加元数据|
    |--|--|
    |`Feedback on an QnA Maker service`|"功能":"all"|
    |`Feedback on an existing feature`|"Feature":"one"|
    
    ![将元数据添加到后续提示符处，以便它可以从服务的会话响应中进行筛选](../media/conversational-context/add-metadata-feature-to-follow-up-prompt.png) 

1. 保存并训练。 

    如果要将发送问题`Give feedback`与元数据筛选器`Feature`值为`all`，将返回的 QnA 对将使用该元数据。 这两个 QnA 对不会返回，因为它们都不匹配筛选器。 

## <a name="test-the-qna-set-to-get-all-the-follow-up-prompts"></a>QnA 设置为获取所有后续的测试会提示

在测试具有跟进问题的提示**测试**窗格中，选择**启用多轮次**，并输入您的问题。 响应包括后续提示。

![测试时，问题在测试窗格中，响应将包括后续提示。](../media/conversational-context/test-pane-with-question-having-follow-up-prompts.png)

## <a name="json-request-to-return-initial-answer-and-follow-up-prompts"></a>JSON 请求返回初始答案和后续提示

使用空`context`对象来请求用户的问题的答案，并包括后续提示。 

```JSON
{
  "question": "accounts and signing in",
  "top": 30,
  "userId": "Default",
  "isTest": false,
  "context": {}
}
```

## <a name="json-response-to-return-initial-answer-and-follow-up-prompts"></a>要返回初始答案和后续提示的 JSON 响应

上一节所请求的答案和任何后续提示`Accounts and signing in`。 响应包括提示信息，位于`answers[0].context`，包含要向用户显示的文本。 

```JSON
{
    "answers": [
        {
            "questions": [
                "Accounts and signing in"
            ],
            "answer": "**Accounts and signing in**\n\nWhen you set up your Surface, an account is set up for you. You can create additional accounts later for family and friends, so each person using your Surface can set it up just the way he or she likes. For more info, see All about accounts on Surface.com. \n\nThere are several ways to sign in to your Surface Pro 4: ",
            "score": 86.96,
            "id": 37,
            "source": "surface-pro-4-user-guide-EN .pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 38,
                        "qna": null,
                        "displayText": "Use the sign-in screen"
                    },
                    {
                        "displayOrder": 1,
                        "qnaId": 39,
                        "qna": null,
                        "displayText": "Use Windows Hello to sign in"
                    },
                    {
                        "displayOrder": 2,
                        "qnaId": 40,
                        "qna": null,
                        "displayText": "Sign out"
                    }
                ]
            }
        },
        {
            "questions": [
                "Use the sign-in screen"
            ],
            "answer": "**Use the sign-in screen**\n\n1.  \n\nTurn on or wake your Surface by pressing the power button. \n\n2.  \n\nSwipe up on the screen or tap a key on the keyboard. \n\n3.  \n\nIf you see your account name and account picture, enter your password and select the right arrow or press Enter on your keyboard. \n\n4.  \n\nIf you see a different account name, select your own account from the list at the left. Then enter your password and select the right arrow or press Enter on your keyboard. ",
            "score": 32.27,
            "id": 38,
            "source": "surface-pro-4-user-guide-EN .pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        },
        {
            "questions": [
                "Sign out"
            ],
            "answer": "**Sign out**\n\nHere's how to sign out: \n\n Go to Start , and right-click your name. Then select Sign out. ",
            "score": 27.0,
            "id": 40,
            "source": "surface-pro-4-user-guide-EN .pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        }
    ],
    "debugInfo": null
}
```

`prompts`阵列可提供中的文本`displayText`属性和`qnaId`值以便可以显示这些问题的答案为中的会话流的下一步显示选项。 

## <a name="json-request-to-return-non-initial-answer-and-follow-up-prompts"></a>JSON 请求返回非初始答案和后续提示

填充`context`要包含以前的上下文对象。

在以下 JSON 请求中，当前的问题是`Use Windows Hello to sign in`和前一个问题是`Accounts and signing in`。 

```JSON
{
  "question": "Use Windows Hello to sign in",
  "top": 30,
  "userId": "Default",
  "isTest": false,
  "qnaId": 39,
  "context": {
    "previousQnAId": 37,
    "previousUserQuery": "accounts and signing in"
  }
}
``` 

##  <a name="json-response-to-return-non-initial-answer-and-follow-up-prompts"></a>要返回非初始答案和后续提示的 JSON 响应

QnA Maker _GenerateAnswer_ JSON 响应包括中的后续提示`context`属性中的第一项`answers`对象：

```JSON
{
    "answers": [
        {
            "questions": [
                "Give feedback"
            ],
            "answer": "What kind of feedback do you have?",
            "score": 100.0,
            "id": 288,
            "source": "Editorial",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 291,
                        "qna": null,
                        "displayText": "Feedback on an QnA Maker service"
                    },
                    {
                        "displayOrder": 0,
                        "qnaId": 292,
                        "qna": null,
                        "displayText": "Feedback on an existing feature"
                    }
                ]
            }
        }
    ]
}
```

## <a name="displaying-prompts-and-sending-context-in-the-client-application"></a>显示的提示，并在客户端应用程序中发送上下文 

客户端应用程序显示用户提示显示为建议的操作或按钮的选项的所有问题。
然后，客户端应用程序将当前的 QnA 对与用户查询存储为上下文与下一步的用户查询一起传递。 

使用此[Bot Framework 示例](https://aka.ms/qnamakermultiturnsample)若要查看多轮次对话框工作端到端 QnA Maker 智能机器人中。


## <a name="prompt-order-supported-in-api"></a>在 API 中受支持的提示顺序

编辑 api 仅支持 JSON 响应中返回的提示顺序。 

## <a name="next-steps"></a>后续步骤

了解有关上下文的对话的详细信息[对话框示例](https://aka.ms/qnamakermultiturnsample)或了解详细信息[概念智能机器人应用程序设计的多轮次会话](https://docs.microsoft.com/azure/bot-service/bot-builder-conversations?view=azure-bot-service-4.0)。

> [!div class="nextstepaction"]
> [迁移知识库](../Tutorials/migrate-knowledge-base.md)
