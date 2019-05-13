---
title: 将墨迹数据发送到墨迹识别器 API
titlesuffix: Azure Cognitive Services
description: 了解如何针对不同的应用程序调用墨迹分析器 API
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 5a212c7332d085c15baabef8650572162c47903d
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025451"
---
# <a name="send-ink-data-to-the-ink-recognizer-api"></a>将墨迹数据发送到墨迹识别器 API 

数字墨迹是以数字形式表示手写和绘制等输入内容的技术。 此功能通常是使用一个数字化器实现的，该数字化器可以捕获输入设备（例如手写笔）的动作。 随着设备中不断推出丰富的数字墨迹书写体验，可以通过人工智能和机器学习来识别任意上下文中的书写形状和文本。 使用墨迹识别器 API 可以发送墨迹笔划及获取其详细信息。 

## <a name="the-ink-recognizer-api-vs-ocr-services"></a>墨迹识别器 API 与OCR 服务

墨迹识别器 API 不使用光学字符识别 (OCR)。 OCR 服务通过处理图像中的像素数据来提供手写内容与文本的识别。 此功能有时称为脱机识别。 手写识别器 API 需要处理使用输入设备时捕获的数字墨迹笔划数据。 与 OCR 服务相比，以这种方式处理数字墨迹数据可以生成更准确的识别结果。 

## <a name="sending-ink-data"></a>发送墨迹数据

墨迹识别器 API 需要使用 X 和 Y 坐标来表示输入设备创建的墨迹笔划：从设备接触到检测面开始，到设备离开检测面为止。 每个笔划的点必须是逗号分隔值字符串，采用以下示例所示的 JSON 格式。 此外，每个请求中的每个墨迹笔划必须具有唯一的 ID。 如果在同一请求中重复该 ID，则 API 会返回错误。 若要获得最准确的识别结果，小数点后面至少需要有八位数。 假定画布的原点 (0,0) 位于墨迹画布的左上角。

> [!NOTE]
> 以下示例是无效的 JSON。 在 [GitHub](https://go.microsoft.com/fwlink/?linkid=2089909) 上可以找到完整的墨迹识别器 JSON 请求。
 
```json
{
  "language": "en-US",
  "strokes": [
   {
    "id": 43,
    "points": 
        "5.1365, 12.3845,
        4.9534, 12.1301,
        4.8618, 12.1199,
        4.7906, 12.2217,
        4.7906, 12.5372,
        4.8211, 12.9849,
        4.9534, 13.6667,
        5.0958, 14.4503,
        5.3299, 15.2441,
        5.6555, 16.0480,
        ..."
   },
    ...
  ]
}
```

## <a name="ink-recognizer-response"></a>墨迹识别器响应

墨迹识别器 API 返回有关它从墨迹内容中识别到的对象的分析响应。 响应包含描述不同墨迹笔划之间的关系的识别单元。 例如，创建不同独立形状的笔划将包含在不同的单元中。 每个单元包含有关其墨迹笔划的详细信息，这包括识别的对象、其坐标及其他绘制属性。

## <a name="shapes-recognized-by-the-ink-recognizer-api"></a>墨迹识别器 API 识别的形状

墨迹识别器 API 可以识别笔记中最常用的形状。 下图显示了一些基本示例。 有关该 API 识别的形状和其他墨迹内容的完整列表，请参阅 [API 参考文章](https://go.microsoft.com/fwlink/?linkid=2089907)。 

![墨迹识别器 API 识别的形状列表](../media/shapes.png)

## <a name="recommended-calling-patterns"></a>建议的调用模式

可以根据应用程序以不同的模式调用墨迹识别器 REST API。 

### <a name="user-initiated-api-calls"></a>用户发起的 API 调用

如果生成的应用采用用户输入（例如，笔记或批注应用），则你可以在该应用中控制何时要将哪种墨迹发送到墨迹识别器 API。 当画布上同时存在文本和形状，并且用户想要针对文本和形状执行不同的操作时，此功能特别有用。 请考虑添加选择功能（例如套索或其他几何选择工具），使用户能够选择要将哪些内容发送到 API。  

### <a name="app-initiated-api-calls"></a>应用发起的 API 调用

也可以让应用在超时后调用墨迹识别器 API。 通过将当前墨迹笔划定期发送到 API，可以存储创建的识别结果，同时改进 API 的响应时间。 例如，在检测到用户已完成一行手写文本后，可将该文本发送到 API。 

提前获取识别结果可以获得墨迹笔划之间的关系特征。 例如，哪些笔划共同构成了同一个单词、行、列表、段落或形状。 此信息可以增强应用的墨迹选择功能，例如，可以一次性选择多组笔划。

## <a name="integrate-the-ink-recognizer-api-with-windows-ink"></a>将墨迹识别器 API 与 Windows Ink 集成

[Windows Ink](https://docs.microsoft.com/windows/uwp/design/input/pen-and-stylus-interactions) 提供所需的工具和技术用于在多种不同的设备上实现数字墨迹体验。 可以将 Windows Ink 平台与墨迹识别 API 相结合，以创建可以显示和解释数字墨迹笔划的应用程序。

## <a name="next-steps"></a>后续步骤

* [什么是墨迹识别器 API？](../overview.md)
* [墨迹识别器 REST API 参考](https://go.microsoft.com/fwlink/?linkid=2089907)

* 开始使用以下语言发送数字墨迹笔划数据：
    * [C#](../quickstarts/csharp.md)
    * [Java](../quickstarts/java.md)
    * [JavaScript](../quickstarts/javascript.md)