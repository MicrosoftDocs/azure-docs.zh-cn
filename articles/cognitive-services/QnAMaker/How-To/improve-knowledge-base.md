---
title: 改进知识库 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 通过主动学习，可以根据用户提交的内容为问题和答案对提供替代问题建议，从而提高知识库的质量。 查看这些建议后，可以将其添加到现有问题，也可以拒绝它们。 知识库不会自动更改。 你必须接受的建议进行的任何更改才会生效。 这些建议会添加问题，但不会更改或删除现有问题。
author: diberry
manager: nitinme
services: cognitive-services
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 03/21/2019
ms.author: diberry
ms.openlocfilehash: 3e54e8dcb6efa9251262c651730376a0d04edcf9
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65144977"
---
# <a name="use-active-learning-to-improve-your-knowledge-base"></a>使用 active 学习来改善您的知识库

通过主动学习，可以根据用户提交的内容为问题和答案对提供替代问题建议，从而提高知识库的质量。 查看这些建议后，可以将其添加到现有问题，也可以拒绝它们。 

知识库不会自动更改。 必须先接受建议才能使更改生效。 这些建议会添加问题，但不会更改或删除现有问题。

## <a name="what-is-active-learning"></a>主动学习是什么？

QnA Maker 通过隐式和显式反馈学习新的问题变体。
 
* 隐式反馈 - 排名程序了解到用户问题具有多个答案且分数非常接近时，将其视为反馈。 
* 显式反馈 - 从知识库返回分数变化很小的多个答案时，客户端应用程序会询问用户哪个问题是正确的问题。 使用训练 API 将用户的显式反馈发送给 QnA Maker。 

这两种方法都为排名程序提供了类似的集中查询。

## <a name="how-active-learning-works"></a>主动学习的工作原理

根据 QnA Maker 针对任何给定查询返回的数量最少的答案分数来触发主动学习。 如果分数差范围很小，则对于每个可能的答案，将查询视为可能的建议。 

所有建议通过相似度聚集在一起，并且根据最终用户的特定查询频率显示替代问题排名靠前的建议。 在终结点获得合理数量和类型的使用查询的情况下，主动学习可提供最佳建议。

当处于群集状态 5 或更多类似的查询时，每隔 30 分钟，QnA Maker 推荐到知识库设计器以接受或拒绝的基于用户的问题。

一旦问题建议 QnA Maker 门户中，需要审阅和接受或拒绝这些建议。 

## <a name="upgrade-your-version-to-use-active-learning"></a>升级版本以使用主动学习

运行时版本 4.4.0 及更高版本支持主动学习。 如果你的知识库是在早期版本上创建的，请[升级运行时](troubleshooting-runtime.md#how-to-get-latest-qnamaker-runtime-updates)以使用此功能。 

## <a name="best-practices"></a>最佳做法

有关使用主动学习的最佳做法，请参阅[最佳做法](../Concepts/best-practices.md#active-learning)。

## <a name="score-proximity-between-knowledge-base-questions"></a>知识库问题之间的分数接近

当问题的分数置信度很高（例如 80%）时，考虑进行主动学习的分数范围较广，大约在 10% 以内。 随着置信度分数降低（例如 40%），分数范围也会降低，大约在 4% 以内。 

用于确定置信度的算法并不是一个简单的计算。 前面示例中的范围并不是固定的，仅用作了解算法影响的参考。

## <a name="turn-on-active-learning"></a>启用主动学习

默认情况下，主动学习处于关闭状态。 启用它可以查看建议的问题。 

1. 选择**发布**发布知识库。 主动学习查询从 GenerateAnswer API 预测终结点仅收集。 Qna Maker 门户中测试窗格中的查询不会影响主动学习。

1. 若要打开主动学习，请单击“名称”，转到 QnA Maker 门户右上角的“[服务设置](https://www.qnamaker.ai/UserSettings)”。  

    ![打开主动学习建议的问题备选方法，从服务设置页。 在右上方菜单中，选择您的用户名称，然后选择服务设置。](../media/improve-knowledge-base/Endpoint-Keys.png)


1. 找到 QnA Maker 服务，然后切换到“主动学习”。 

    [![在服务设置页中，切换主动学习功能。如果您不能切换功能，您可能需要升级你的服务。](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png)](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png#lightbox)

    启用“主动学习”后，知识库会根据用户提交的问题定期建议新的问题。 可以通过再次切换设置来禁用“主动学习”。

## <a name="add-active-learning-suggestion-to-knowledge-base"></a>向知识库添加主动学习建议

1. 若要查看建议的问题，在**编辑**知识库页上，选择**视图选项**，然后选择**显示主动学习建议**。 

    [![在门户的编辑部分中，选择显示的建议，若要查看主动学习新问题替代项。](../media/improve-knowledge-base/show-suggestions-button.png)](../media/improve-knowledge-base/show-suggestions-button.png#lightbox)

1. 筛选要通过选择显示仅建议的问题和答案对包含的知识库**筛选器由建议**。

    [![使用建议切换筛选器来查看仅主动学习问题的建议替代方法。](../media/improve-knowledge-base/filter-by-suggestions.png)](../media/improve-knowledge-base/filter-by-suggestions.png#lightbox)

1.  每个包含建议的问题部分都会显示带有复选标记的新问题，`✔` 表示接受问题，`x` 表示拒绝建议。 选中复选标记可以添加问题。 

    [![选择或通过选择绿色的复选标记或红色删除标记拒绝主动学习问题的建议替代方法。](../media/improve-knowledge-base/accept-active-learning-suggestions.png)](../media/improve-knowledge-base/accept-active-learning-suggestions.png#lightbox)

    可以通过选择“全部添加”或“全部拒绝”来添加或删除“所有建议”。

1. 选择“保存并训练”，将所做的更改保存到知识库。

1. 选择**发布**以允许从 GenerateAnswer API 能够访问这些更改。

    当处于群集状态 5 或更多类似的查询时，每隔 30 分钟，QnA Maker 推荐到知识库设计器以接受或拒绝的基于用户的问题。

## <a name="determine-best-choice-when-several-questions-have-similar-scores"></a>当几个问题具有相似的分数时，确定最佳选择

当某个问题的分数与其他问题的分数过于接近时，客户端应用程序开发人员可以选择要求进行说明。

### <a name="use-the-top-property-in-the-generateanswer-request"></a>在 GenerateAnswer 请求中使用 top 属性

在向 QnA Maker 提交问题以获得答案时，JSON 主体的一部分允许返回多个最佳答案：

```json
{
    "question": "wi-fi",
    "isTest": false,
    "top": 3
}
```

当客户端应用程序（例如聊天机器人）收到响应时，返回排名靠前的 3 个问题：

```json
{
    "answers": [
        {
            "questions": [
                "Wi-Fi Direct Status Indicator"
            ],
            "answer": "**Wi-Fi Direct Status Indicator**\n\nStatus bar icons indicate your current Wi-Fi Direct connection status:  \n\nWhen your device is connected to another device using Wi-Fi Direct, '$  \n\n+ •+ ' Wi-Fi Direct is displayed in the Status bar.",
            "score": 74.21,
            "id": 607,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Wi-Fi - Connections"
            ],
            "answer": "**Wi-Fi**\n\nWi-Fi is a term used for certain types of Wireless Local Area Networks (WLAN). Wi-Fi communication requires access to a wireless Access Point (AP).",
            "score": 74.15,
            "id": 599,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Turn Wi-Fi On or Off"
            ],
            "answer": "**Turn Wi-Fi On or Off**\n\nTurning Wi-Fi on makes your device able to discover and connect to compatible in-range wireless APs.  \n\n1.  From a Home screen, tap ::: Apps > e Settings .\n2.  Tap Connections > Wi-Fi , and then tap On/Off to turn Wi-Fi on or off.",
            "score": 69.99,
            "id": 600,
            "source": "Bugbash KB.pdf",
            "metadata": []
        }
    ]
}
```

### <a name="client-application-follow-up-when-questions-have-similar-scores"></a>在问题具有相似的分数时的客户应用程序跟进

客户端应用程序显示所有问题，并且可以让用户选择最能代表其意图的问题。 

一旦用户选择一个现有问题，客户端应用程序以使用 QnA Maker 训练 API 的反馈方式发送用户的选择。 此反馈完成活动学习反馈循环。 

使用[Azure 智能机器人应用程序示例](https://aka.ms/activelearningsamplebot)若要查看端到端方案中的主动学习。

## <a name="train-api"></a>训练 API

主动学习反馈训练 API POST 请求发送到 QnA Maker。 API 签名是：

```http
POST https://<QnA-Maker-resource-name>.azurewebsites.net/qnamaker/knowledgebases/<knowledge-base-ID>/train
Authorization: EndpointKey <endpoint-key>
Content-Type: application/json
{"feedbackRecords": [{"userId": "1","userQuestion": "<question-text>","qnaId": 1}]}
```

|HTTP 请求属性|名称|Type|目的|
|--|--|--|--|
|URL 路由参数|知识库 ID|string|知识库的 GUID。|
|主机子域|QnAMaker 资源名称|string|在 Azure 订阅中将 QnA Maker 的主机名。 这是可在设置页上后发布该知识库。 |
|标头|Content-Type|string|发送到 API 的正文的媒体类型。 默认值为： `application/json`|
|标头|授权|string|终结点密钥 (EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)。|
|POST 正文|JSON 对象|JSON|培训反馈|

JSON 正文具有多个设置：

|JSON 正文属性|Type|目的|
|--|--|--|--|
|`feedbackRecords`|数组|反馈的列表。|
|`userId`|string|接受建议的问题的人员的用户 ID。 用户 ID 格式是由您决定。 例如，电子邮件地址可以是您的体系结构中的有效用户 ID。 可选。|
|`userQuestion`|string|问题的确切文本。 必需。|
|`qnaID`|数字|在中找到的问题的 ID [GenerateAnswer 响应](metadata-generateanswer-usage.md#generateanswer-response-properties)。 |

一个 JSON 正文示例如下所示：

```json
{
    "feedbackRecords": [
        {
            "userId": "1",
            "userQuestion": "<question-text>",
            "qnaId": 1
        }
    ]
}
```

成功响应将返回 204，而且没有 JSON 响应正文中的状态。 

<a name="active-learning-is-saved-in-the-exported-apps-tsv-file"></a>

## <a name="active-learning-is-saved-in-the-exported-knowledge-base"></a>导出知识库中保存主动学习

当你的应用程序启用，主动学习和导出应用程序中， `SuggestedQuestions` tsv 文件中的列将保留主动学习数据。 

`SuggestedQuestions`列为的隐式的信息的 JSON 对象 (`autosuggested`) 和显式 (`usersuggested`) 的反馈。 单个用户提交的问题此 JSON 对象的一个示例`help`是：

```JSON
[
    {
        "clusterHead": "help",
        "totalAutoSuggestedCount": 1,
        "totalUserSuggestedCount": 0,
        "alternateQuestionList": [
            {
                "question": "help",
                "autoSuggestedCount": 1,
                "userSuggestedCount": 0
            }
        ]
    }
]
```

当你重新导入此应用时，主动学习继续收集的信息，建议您的知识库的建议。 

## <a name="next-steps"></a>后续步骤
 
> [!div class="nextstepaction"]
> [将用于 GenerateAnswer API 元数据](metadata-generateanswer-usage.md)
