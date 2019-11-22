---
title: 快速入门：识别存储在 Blob 存储中的语音，Java - 语音服务
titleSuffix: Azure Cognitive Services
description: TBD
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 10/28/2019
ms.author: erhopf
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: c5fa12621d689287db0d91885fc0a35edb772294
ms.sourcegitcommit: c22327552d62f88aeaa321189f9b9a631525027c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2019
ms.locfileid: "73506057"
---
## <a name="prerequisites"></a>先决条件

在开始之前，请务必：

> [!div class="checklist"]
> * [创建 Azure 语音资源](../../../../get-started.md)
> * [将源文件上传到 Azure Blob](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)
> * [设置开发环境](../../../../quickstarts/setup-platform.md?tabs=dotnet)
> * [创建空示例项目](../../../../quickstarts/create-project.md?tabs=dotnet)

## <a name="open-your-project-in-eclipse"></a>在 Eclipse 中打开项目

第一步是确保项目在 Eclipse 中打开。

1. 启动 Eclipse
2. 加载项目并打开 `Main.java`。

## <a name="add-a-reference-to-gson"></a>添加对 Gson 的引用
本快速入门中将使用外部 JSON 序列化程序/反序列化程序。 对于 Java，我们选择了 [Gson](https://github.com/google/gson)。

打开 pom.xml 并添加以下引用：
[!code-xml[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/pom.xml?range=19-25)]

## <a name="start-with-some-boilerplate-code"></a>从一些样本代码入手

添加一些代码作为项目的框架。

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=1-13,95-105,206-207)]
（需要将 `YourSubscriptionKey`、`YourServiceRegion` 和 `YourFileUrl` 的值替换成自己的值。）

## <a name="json-wrappers"></a>JSON 包装器

由于 REST API 采用 JSON 格式的请求并且也返回 JSON 结果，因此我们可以仅使用字符串与之进行交互，但不建议这样做。
为了使请求和响应更易于管理，我们将声明一些用于对 JSON 进行序列化/反序列化处理的类。

开始操作并在 `Main` 之前放置声明。
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=15-93)]

## <a name="create-and-configure-an-http-client"></a>创建和配置 Http 客户端
首先，我们需要具有正确的基本 URL 和身份验证集的 Http 客户端。
将代码插入 `Main`
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=106-113)]

## <a name="generate-a-transcription-request"></a>生成听录请求
接下来，我们将生成听录请求。 将此代码添加到 `Main`
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=115-116)]

## <a name="send-the-request-and-check-its-status"></a>发送请求并查看其状态
现在我们将请求发布到语音服务并检查初始响应代码。 此响应代码将仅指示服务是否已收到请求。 该服务将在响应标头中返回一个 URL，这是它将存储听录状态的位置。
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=118-129)]

## <a name="wait-for-the-transcription-to-complete"></a>等待听录完成
由于服务以异步方式处理听录，因此需要时常轮询其状态。 我们每 5 秒查看一次。

通过检索在发布请求时收到的 URL 中的内容，可以查看状态。 内容返回后，我们将其反序列化为一个帮助程序类，使其便于交互。

下面是一个轮询代码，其中显示了除成功完成之外的所有状态，我们会在下一步完成该操作。
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=131-159,192-204)]

## <a name="display-the-transcription-results"></a>显示听录结果
服务成功完成听录后，结果将存储在可从状态响应中获取的其他 URL 中。

我们将下载该 URL 的所有内容，对 JSON 进行反序列化，并循环遍历不断输出显示文本的结果。
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java?range=6-160-190)]

## <a name="check-your-code"></a>查看代码
此时，代码应如下所示：（我们已向此版本中添加了一些注释）
[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-blob/src/quickstart/Main.java]

## <a name="build-and-run-your-app"></a>生成并运行应用

现在，可以使用语音服务构建应用并测试语音识别。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [footer](./footer.md)]
