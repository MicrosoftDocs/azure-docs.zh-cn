---
title: 什么是墨迹识别器？ - 墨迹识别器 API
titlesuffix: Azure Cognitive Services
description: 将墨迹识别器集成到应用程序、网站、工具和其他解决方案中，以提供...
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 0ed1a72a5dc61458200b72c768ad722656b820d8
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025603"
---
# <a name="what-is-the-ink-recognizer-api"></a>什么是墨迹识别器 API？


墨迹识别器认知服务提供基于云的 REST API 用于分析和识别数字墨迹内容。 与使用光学字符识别 (OCR) 的服务不同，该 API 需要使用数字墨迹笔划数据作为输入。 数字墨迹笔划是 2D 点（X,Y 坐标，表示数字手写笔或手指的动作）的时序集。 然后，墨迹识别器会识别输入中的形状和手写内容，并返回包含所有已识别实体的 JSON 响应。

![描绘如何将墨迹笔划输入发送到 API 的流程图](media/ink-recognizer-pen-graph.png)

## <a name="features"></a>功能

使用墨迹识别器 API 可以轻松识别应用程序中的手写内容。 

|Feature  |说明  |
|---------|---------|
| 手写内容识别 | 识别采用 63 种核心[语言和区域设置](language-support.md)的手写内容。 | 
| 布局识别 | 获取有关数字墨迹内容的结构信息。 将内容分解为书写区域、段落、行、字词和项目符号列表。 然后，应用程序可以使用布局信息来生成其他特征，例如自动列表格式和形状对齐方式。 |
| 形状识别 | 识别做笔记时最常用的[几何形状](concepts/send-ink-data.md#shapes-recognized-by-the-ink-recognizer-api)。 |
| 组合形状和文本识别 | 识别哪些墨迹笔划属于形状或手写内容，并分别对它们进行分类。|

## <a name="workflow"></a>工作流

墨迹识别器 API 是一项 RESTful Web 服务，可以轻松地通过任何编程语言调用，只要该语言能够发出 HTTP 请求和分析 JSON 即可。

[!INCLUDE [cognitive-services-ink-recognizer-signup-requirements](../../../includes/cognitive-services-ink-recognizer-signup-requirements.md)]

注册后：

1. 提取墨迹笔划数据并[将其格式化](concepts/send-ink-data.md#sending-ink-data)为有效 JSON。
1. 将请求连同数据一起发送到墨迹识别器 API。
1. 通过分析返回的 JSON 消息处理 API 响应。

## <a name="next-steps"></a>后续步骤

请尝试学习以下语言的快速入门，以开始调用墨迹识别器 API。
* [C#](quickstarts/csharp.md)
* [Java](quickstarts/java.md)
* [JavaScript](quickstarts/csharp.md)

若要了解墨迹识别 API 在数字墨迹应用中的工作原理，请查看 GitHub 上的以下示例应用程序：
* [C# 和通用 Windows 平台 (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# 和 Windows Presentation Foundation(WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Javascript Web 浏览器应用](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Java 和 Android 移动应用](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Swift 和 iOS 移动应用](https://go.microsoft.com/fwlink/?linkid=2089805)
