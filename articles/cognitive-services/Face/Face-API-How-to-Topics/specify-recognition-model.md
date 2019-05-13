---
title: 如何指定一个认证模型-人脸 API
titleSuffix: Azure Cognitive Services
description: 本文将演示如何选择要与 Azure 人脸 API 应用程序使用的识别模型。
services: cognitive-services
author: longl
manager: nitinme
ms.service: cognitive-services
ms.component: face-api
ms.topic: conceptual
ms.date: 03/28/2019
ms.author: longl
ms.openlocfilehash: 88b0ac853c64e1e32a2d1c429bdf8655158f030d
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411469"
---
# <a name="specify-a-face-recognition-model"></a>指定人脸识别模型

本指南演示如何指定人脸检测、 标识和相似性搜索使用 Azure 人脸 API 的人脸识别模型。

人脸 API 使用机器学习模型上执行操作的人脸在图像中。 我们继续改进根据客户反馈以及改进在研究中，我们的模型的准确性和我们提供作为模型更新这些改进。 开发人员可以选择指定他们希望使用; 人脸识别模型的版本他们可以选择最适合其用例的模型。

如果您是新用户，我们建议使用最新模型。 继续阅读以了解如何避免模型冲突时在不同的人脸操作中指定它。 如果你是高级的用户，而且不确定是否应切换到最新模型，请跳到[评估不同的模型](#evaluate-different-models)部分评估新模型并使用当前数据集的结果进行比较。

## <a name="prerequisites"></a>必备组件

您应熟悉 AI 人脸检测和标识的概念。 如果不是，首先看到这些操作方法指南：

* [如何检测图像中人脸](HowtoDetectFacesinImage.md)
* [如何识别图像中的人脸](HowtoIdentifyFacesinImage.md)

## <a name="detect-faces-with-specified-model"></a>检测人脸与指定的模型

人脸检测标识人脸的视觉特征点，并查找其边界框的位置。 此外提取人脸的功能，并将其存储在标识中使用。 所有这些信息构成一个人脸的表示形式。

识别模型时使用人脸特征提取，因此执行检测操作时，可以指定模型版本。

使用时[人脸-检测]API，将分配模型版本具有`recognitionModel`参数。 可用值有：

* `recognition_01`
* `recognition_02`

或者，您可以指定_returnRecognitionModel_参数 (默认**false**) 以指示是否_recognitionModel_应在响应中返回。 这样的请求 URL[人脸-检测]REST API 将如下所示：

`https://westus.api.cognitive.microsoft.com/face/v1.0/detect[?returnFaceId][&returnFaceLandmarks][&returnFaceAttributes][&recognitionModel][&returnRecognitionModel]
&subscription-key=<Subscription key>`

如果使用客户端库，则可以分配的值为`recognitionModel`通过传递一个表示版本字符串。
如果将其分配的默认模型版本 (_recognition_01_) 将使用。 请参阅下面的代码示例.NET 客户端库。

```csharp
string imageUrl = "https://news.microsoft.com/ceo/assets/photos/06_web.jpg";
var faces = await faceServiceClient.Face.DetectWithUrlAsync(imageUrl, true, true, recognitionModel: "recognition_02", returnRecognitionModel: true);
```

## <a name="identify-faces-with-specified-model"></a>使用指定的模型中识别人脸

人脸 API 可以从图像中提取人脸数据并将其与**Person**对象 (通过[添加人脸](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b)API 调用，例如)，和多个**人员**对象可以是一起存储在**person Group**。 然后，可与比较新的人脸**person Group** (使用[人脸-识别]调用)，并可以标识该组中的匹配人员。

一个**person Group**应具有一个唯一识别模型的所有**人员**s，并且您可以指定此使用`recognitionModel`参数在创建组时 ([PersonGroup - 创建]或[LargePersonGroup - 创建])。 如果不指定此参数，原始`recognition_01`使用模型。 一个组将始终使用创建，识别模型和新的人脸就会变得与此模型关联它们添加到它;这不能组的创建后更改。 若要查看哪些模型**person Group**配置，使用[PersonGroup - Get] API _returnRecognitionModel_参数设置为**true**.

请参阅下面的代码示例.NET 客户端库。

```csharp
// Create an empty PersonGroup with "recognition_02" model
string personGroupId = "mypersongroupid";
await faceServiceClient.PersonGroup.CreateAsync(personGroupId, "My Person Group Name", recognitionModel: "recognition_02");
```

在此代码中， **person Group** id`mypersongroupid`创建后，它将设置为使用和_recognition_02_模型提取人脸特征。

相应地，您需要指定要使用检测人脸时要对此进行比较的模型**person Group** (通过[人脸-检测]API)。 您使用的模型应始终为与一致**person Group**的配置; 否则，该操作将失败由于不兼容的模型。

在没有变化[人脸-识别]API; 您只需在检测中指定的模型版本。

## <a name="find-similar-faces-with-specified-model"></a>查找相似人脸与指定的模型

此外可以指定相似性搜索一个认证模型。 可以分配使用的模型版本`recognitionModel`时创建的人脸列表[FaceList-创建]API 或[LargeFaceList - Create]。 如果不指定此参数，原始`recognition_01`使用模型。 人脸列表将始终使用创建，识别模型和新的人脸就会变得与此模型关联它们添加到它;这不能创建后更改。 若要查看哪些人脸列表配置使用的模型，请使用[FaceList - Get]使用 API _returnRecognitionModel_参数设置为**true**。

请参阅下面的代码示例.NET 客户端库。

```csharp
await faceServiceClient.FaceList.CreateAsync(faceListId, "My face collection", recognitionModel: "recognition_02");
```

此代码将创建一个名为的人脸列表`My face collection`，并使用_recognition_02_模型来提取特征。 搜索类似人脸的新检测到人脸此人脸列表时面临的必须已检测到 ([人脸-检测]) 使用_recognition_02_模型。 上一节中，模型必须保持一致。

在没有变化[人脸-查找相似]API; 您仅在检测中指定的模型版本。

## <a name="verify-faces-with-specified-model"></a>验证与指定的模型的人脸

[Face - Verify]API 检查两张脸是否属于同一个人。 在验证 API 中识别模型方面没有变化，但您只能比较具有相同的模型检测到的人脸。 因此，两张脸都需要已检测到使用`recognition_01`或`recognition_02`。

## <a name="evaluate-different-models"></a>评估不同的模型

如果你想要比较的性能_recognition_01_并_recognition_02_你将需要对数据模型：

1. 创建两个**person Group**与_recognition_01_并_recognition_02_分别。
1. 使用您的图像数据检测人脸并注册到**Person**这两个的 s **person Group**s，并触发培训处理与[PersonGroup - 训练] API。
1. 使用测试[人脸-识别]同时**person Group**s 并比较结果。

如果正常情况下指定置信度阈值 （零和确定有多大把握模型必须是以识别人脸的那个之间的值），您可能需要为不同的模型使用不同的阈值。 一个模型的阈值并不旨在共享到另一个并不一定生成相同的结果。

## <a name="next-steps"></a>后续步骤

在本文中，您学习了如何指定要使用不同的人脸服务 Api 的识别模型。 接下来，按照快速入门，若要开始使用人脸检测。

* [检测图像中人脸](../quickstarts/csharp-detect-sdk.md)

[人脸-检测]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d
[人脸-查找相似]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237
[人脸-识别]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239
[Face - Verify]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a（人脸 - 验证）
[PersonGroup - 创建]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
[PersonGroup - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246
[PersonGroup Person - Add Face]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b
[PersonGroup - 训练]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249
[LargePersonGroup - 创建]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d
[FaceList-创建]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b
[FaceList - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c
[LargeFaceList - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc
