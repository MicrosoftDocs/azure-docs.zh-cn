---
title: 创建、发布、回答
titleSuffix: QnA Maker - Azure Cognitive Services
description: 此基于 REST 的教程详细介绍如何以编程方式创建和发布知识库，然后通过知识库来回答问题。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.date: 01/24/2019
ms.author: diberry
ms.openlocfilehash: 355c11d4f8ae0089527a9f0d21af7c4d6070f73a
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64922372"
---
# <a name="tutorial-using-c-create-knowledge-base-then-answer-question"></a>教程：使用 C# 创建知识库并回答问题

本教程详细介绍如何以编程方式创建和发布知识库 (KB)，然后通过知识库来回答客户问题。 

> [!div class="checklist"]
> * 创建知识库 
> * 查看创建状态
> * 训练和发布知识库
> * 获取终结点信息
> * 使用 Curl 查询知识库


本快速入门调用了 QnA Maker API：

* [创建知识库 (KB)](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)
* [获取操作详细信息](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails)
* [获取知识库详细信息](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_getknowledgebasedetails) 
* [获取知识库终结点](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/endpointkeys_getendpointkeys)
* [发布](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fe) 

## <a name="prerequisites"></a>先决条件

* 最新的 [**Visual Studio Community Edition**](https://www.visualstudio.com/downloads/)。
* 必须已有一个 [QnA Maker 服务](../How-To/set-up-qnamaker-service-azure.md)。 若要检索密钥，请在仪表板的“资源管理”下选择“密钥”。 

> [!NOTE] 
> 完整的解决方案文件可从 [Azure-Samples/cognitive-services-qnamaker-csharp GitHub 存储库](https://github.com/Azure-Samples/cognitive-services-qnamaker-csharp/tree/master/documentation-samples/tutorials/create-publish-answer-knowledge-base)获得。

## <a name="create-a-knowledge-base-project"></a>创建知识库项目

[!INCLUDE [Create Visual Studio Project](../../../../includes/cognitive-services-qnamaker-quickstart-csharp-create-project.md)] 

## <a name="add-the-required-dependencies"></a>添加必需的依赖项

在 Program.cs 的顶部，将单个 _using_ 语句替换为以下行，以向项目添加必要的依赖项：

[!code-csharp[Add the required dependencies](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=1-11 "Add the required dependencies")]

## <a name="add-a-kbdetails-class"></a>添加 KBDetails 类
将此 KBDetails 类添加到命名空间括号内。 此类允许 NewtonSoft 库将 JSON 响应反序列化为 C# 类。

[!code-csharp[Add a KBDetails class](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=15-26 "Add a KBDetails class")]

## <a name="add-the-required-constants"></a>添加必需的常量

在 Program 类的顶部，添加以下常量来访问 QnA Maker：

[!code-csharp[Add the required constants](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=30-57 "Add the required constants")]

## <a name="add-the-kb-definition"></a>添加知识库定义

在常量后，添加以下知识库模型定义：

[!code-csharp[Add the KB definition](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=59-85 "Add the KB definition")]

## <a name="add-supporting-functions-and-structures"></a>添加支持函数和结构
在 Program 类中添加以下代码块：

[!code-csharp[Add supporting functions and structures](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=87-123 "Add supporting functions and structures")]

## <a name="add-a-post-request-to-create-kb"></a>添加一个 POST 请求来创建知识库

以下代码向 QnA Maker API 发出 HTTPS 请求来创建知识库并接收响应：

[!code-csharp[Add a POST request to create KB](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=124-141 "Add a POST request to create KB")]

此 API 调用将返回包括操作 ID 的 JSON 响应。 以后，此程序会使用操作 ID 来确定 KB 是否已成功创建。 

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:19:01Z",
  "lastActionTimestamp": "2018-09-26T05:19:01Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "YYYe12ff-5d04-4b73-b594-8575f9787963"
}
```

## <a name="add-get-request-to-determine-creation-status"></a>添加 GET 请求来确定创建状态

检查创建操作的状态。

[!code-csharp[Add GET request to determine creation status](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=142-151 "Add GET request to determine creation status")]

此 API 调用返回包含操作状态的 JSON 响应： 

```JSON
{
  "operationState": "Running",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:22:53Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "YYYe12ff-5d04-4b73-b594-8575f9787963"
}
```

重复调用，直到成功或失败： 

```JSON
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:23:08Z",
  "resourceLocation": "/knowledgebases/XXX7892b-10cf-47e2-a3ae-e40683adb714",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "YYYe12ff-5d04-4b73-b594-8575f9787963"
}
```

## <a name="add-createkb-method"></a>添加 CreateKB 方法

以下方法封装了用于创建知识库并检查状态的调用。  _create_ **Operation ID** 在 POST 响应标头字段 **Location** 中返回，然后在 GET 请求中用作路由的一部分。 因为知识库创建可能要花费一些时间，所以你需要重复用来检查状态的调用，直到状态为成功或失败。 操作成功后，会在 **resourceLocation** 中返回 KB ID。 

[!code-csharp[Add GET request to determine creation status](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=152-227 "Add GET request to determine creation status")]

## <a name="add-publish-method"></a>添加发布方法

成功创建知识库以后，请发布该 KB。 你可能已预期到对训练 API 的调用。 这不是此版本所需的。 

以下代码向 QnA Maker API 发出 HTTPS 请求来发布知识库并接收响应：

[!code-csharp[Add publish method](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=228-259 "Add publish method")]

对于成功的发布，该 API 调用会返回一个 204 状态，并且响应正文中没有任何内容。 快速入门代码会为 204 响应添加文本，因此你可以看到结果。

对于任何其他响应，该响应将原样返回，不做任何更改。

## <a name="generating-an-answer"></a>生成答案
为了访问 KB 以发送问题并接收最佳答案，程序需要 KB 详细信息 API 中提供的终结点主机，以及终结点 API 中提供的主终结点密钥。 这些方法与用于生成答案的方法一起位于以下部分。 

下表介绍了如何使用此数据来构造 URI：

|生成答案 URI 模板|
|--|
|https://**HOSTNAME**.azurewebsites.net/qnamaker/knowledgebases/**KBID**/generateAnswer|

主终结点作为标头传递，以便对请求进行身份验证，从而生成答案：

|标头名称|标头值|
|--|--|
|授权|`Endpoint` + **主终结点**<br>示例： `Endpoint xxxxxxx`<br>请注意 `Endpoint` 的文本与主终结点值之间的空格。 

请求正文需传递适当的 JSON：

```JSON
{
    question: "What languages does QnA Maker support?"
}
```

## <a name="get-kb-details"></a>获取 KB 详细信息
添加以下方法以获取 KB 详细信息。 这些详细信息包含 KB 的主机名。 主机名是在创建 QnA Maker 资源时输入的 QnA Maker Azure Web 服务的名称。 

[!code-csharp[Get KB Details](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=260-273 "Add publish method")]

此 API 调用返回一个 JSON 响应： 

```JSON
{
  "id": "ZZZ31e19-cba7-48d1-8594-5c4297ecc9c1",
  "hostName": "https://qnamaker-s0-s.azurewebsites.net",
  "lastAccessedTimestamp": "2018-10-11T18:10:05Z",
  "lastChangedTimestamp": "2018-10-11T18:09:37Z",
  "lastPublishedTimestamp": "2018-10-11T18:10:15Z",
  "name": "QnA Maker FAQ from quickstart",
  "userId": "AAAc3841df0b42cdb00f53a49d51a89c",
  "urls": [
    "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
    "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
  ],
  "sources": [
    "Custom Editorial"
  ]
}
```

## <a name="get-kb-endpoints"></a>获取 KB 终结点
添加以下方法，以便获取 QnA Maker 的主终结点。 这些终结点并不绑定到 KB，而是适用于 Azure 门户中与 QnA Maker 资源键相关联的所有 KB。  

[!code-csharp[Get KB Endpoints](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=274-289 "Get KB Endpoints")]

此 API 调用返回一个 JSON 响应： 

```JSON
{
  "primaryEndpointKey": "AAAb5719-e2f7-4a33-937d-7a3b4736a1be",
  "secondaryEndpointKey": "BBBcba78-c1d2-4166-b98f-c77255aefaba",
  "installedVersion": "4.2.0",
  "lastStableVersion": "4.2.0"
}
```

## <a name="get-an-answer"></a>获取答案
添加以下方法，以便获取用户问题的答案。 

[!code-csharp[Get Answer](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=290-315 "Get Answer")]

此 API 调用返回一个 JSON 响应： 

```JSON
{
  "answers": [
    {
      "questions": [
        "Does QnA Maker support non-English languages?"
      ],
      "answer": "See more details about [supported languages](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/languages-supported).\n\n\nIf you have content from multiple languages, be sure to create a separate service for each language.",
      "score": 82.19,
      "id": 11,
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "metadata": []
    }
  ]
}
```

## <a name="main-method"></a>Main 方法
main 方法显示创建、发布和生成答案所需的同步调用。 

[!code-csharp[Main method](~/samples-qnamaker-csharp/documentation-samples/tutorials/create-publish-answer-knowledge-base/QnaMakerQuickstart/Program.cs?range=316-337 "Main method")]

## <a name="build-and-run-the-program"></a>生成并运行程序

生成并运行程序。 

创建知识库以后，即可在 QnA Maker 门户 - [My knowledge bases](https://www.qnamaker.ai/Home/MyServices)（我的知识库）页中查看它。 了解如何使用它来生成答案 API 以后，即可将该 API 用于任何语言或 HTTP 请求框架。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 参考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)
