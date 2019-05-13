---
title: 教程：结合使用 ONNX 模型和 Windows ML - 自定义视觉服务
titlesuffix: Azure Cognitive Services
description: 了解如何创建使用从 Azure 认知服务导出的 ONNX 模型的 Windows UWP 应用。
services: cognitive-services
author: larryfr
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 03/21/2019
ms.author: larryfr
ms.openlocfilehash: ceb45048d160603a3ee619e3ea0e69a0c188a9ad
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64725783"
---
# <a name="tutorial-use-an-onnx-model-from-custom-vision-with-windows-ml-preview"></a>教程：结合使用自定义视觉服务 ONNX 模型和 Windows ML（预览版）

了解如何结合使用从自定义影像服务导出的 ONNX 模型和 Windows ML（预览版）。

本文档介绍了如何结合使用从自定义影像服务导出的 ONNX 文件和 Windows ML。 同时还展示了示例 Windows UWP 应用。 示例中随附可以进行识别的已训练模型。 此外，还分步介绍了如何对此示例使用自己的模型。

> [!div class="checklist"]
> * 关于示例应用
> * 获取示例代码
> * 运行示例
> * 使用自己的模型

## <a name="prerequisites"></a>先决条件

* Windows 10 17738 或更高版本

* 适用于内部版本 17738 或更高版本的 Windows SDK

* 启用了“通用 Windows 平台开发”工作负载的 Visual Studio 2017 版本 15.7 或更高版本。

* 已启用的“开发人员模式”。 有关详细信息，请参阅[启用设备以用于开发](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development)文档。

## <a name="about-the-example-app"></a>关于示例应用

应用是通用 Windows UWP 应用。 可以通过它从计算机中选择一个图像，然后将图像发送至模型。 模型返回的标记和分数显示在图像旁边。

## <a name="get-the-example-code"></a>获取示例代码

可以从 [https://github.com/Azure-Samples/cognitive-services-onnx12-customvision-sample/](https://github.com/Azure-Samples/cognitive-services-onnx12-customvision-sample/) 获取此示例应用。

## <a name="run-the-example"></a>运行示例

1. 按 `F5` 键，通过 Visual Studio 启动应用。 系统可能会提示启用“开发人员模式”。 有关详细信息，请参阅[启用设备以用于开发](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development)文档。

1. 当应用程序启动时，请使用相关按钮来选择要评分的图像。

## <a name="use-your-own-model"></a>使用自己的模型

要使用自己的模型，请按照以下步骤操作：

1. 使用自定义视觉服务[创建和训练](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier)一个分类器。 若要导出模型，请选择一个“压缩”域，例如“常规(压缩)”。 若要导出现有的分类器，请选择右上角的齿轮图标，将域转换为压缩域。 在“设置”中，选择一个压缩模型，进行保存，然后训练项目。  

1. 转到“性能”选项卡即可[导出模型](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model)。选择一个使用压缩域训练的迭代，此时会显示“导出”按钮。 选择“导出”、“ONNX”、“ONNX1.2”，然后选择“导出”。 文件就绪以后，请选择“下载”按钮。

1. 将 ONNX 文件拖到项目的 __Assets__ 文件夹中。 

1. 在解决方案资源管理器中，右键单击 Assets 文件夹，然后选择“添加现有项”。 选择 ONNX 文件。

1. 在解决方案资源管理器中，从 Assets 文件夹选择 ONNX 文件。 更改文件的以下属性：

    * __生成操作__ -> __内容__
    * __复制到输出目录__ -> __如果较新则复制__

1. 将 `_onnxFileNames` 变量更改为 ONNX 文件的名称。 另请将 `ClassLabel` 修改为模型包含的标签数。

1. 生成并运行。

1. 单击按钮选择要评估的图像。

## <a name="next-steps"></a>后续步骤

要了解导出和使用自定义影像服务模型的其他方法，请参阅以下文档：

* [导出模型](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model)
* [在 Android 应用程序中使用导出的 Tensorflow 模型](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [在 Swift iOS 应用中使用导出的 CoreML 模型](https://go.microsoft.com/fwlink/?linkid=857726)
* [在带 Xamarin 的 iOS 应用程序中使用导出的 CoreML 模型](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)

有关结合使用 ONNX 模型和 Windows ML 的详细信息，请参阅[使用 Windows ML 将模型集成到应用](/windows/ai/windows-ml/integrate-model)文档。
