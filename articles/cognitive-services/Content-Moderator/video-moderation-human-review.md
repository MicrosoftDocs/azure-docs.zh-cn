---
title: 通过人工评审审查视频 - 内容审查器
titlesuffix: Azure Cognitive Services
description: 使用计算机辅助视频审查和人工审阅工具来审查是否有不当内容
services: cognitive-services
author: sanjeev3
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: sajagtap
ms.openlocfilehash: a6c467d3153400815e37a5d461766140abd1fa32
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228125"
---
# <a name="video-moderation-with-human-review"></a>通过人工评审审查视频

使用内容审查器的计算机辅助[视频审查](video-moderation-api.md)并[人工审阅工具](Review-Tool-User-Guide/human-in-the-loop.md)审查视频和成人 （显式） 的脚本和挑逗性 （建议） 的内容，以获得最佳的结果你的业务。

## <a name="video-trained-classifier-preview"></a>视频定型分类器（预览）

计算机辅助视频分类是通过图像定型模型或视频定型模型实现。 与图像定型视频分类器不同，Microsoft 的成人和挑逗性视频分类器是通过视频进行定型。 这种方法提高了匹配质量。

## <a name="shot-detection"></a>镜头检测

输出分类详细信息时，附加视频智能有助于更灵活地分析视频。 Microsoft 视频审查服务提供镜头级信息，而不是仅仅输出帧。 现在可以视需要选择是在镜头一级，还是在帧一级分析视频。

## <a name="key-frame-detection"></a>关键帧检测

视频审查服务仅识别和输出可能完整（良好）的帧，而不是定期输出帧。 借助这种功能，可以高效生成帧，从而实现帧级成人内容和挑逗性内容分析。

下面摘取的代码行展示了包含潜在镜头、关键帧以及成人内容和挑逗性内容分数的部分响应：

```json
"fragments":[  
  {  
    "start":0,
    "duration":18000
  },
  {  
    "start":18000,
    "duration":3600,
    "interval":3600,
    "events":[  
      [  
        {  
          "reviewRecommended":false,
          "adultScore":0.00001,
          "racyScore":0.03077,
          "index":5,
          "timestamp":18000,
          "shotIndex":0
        }
      ]
    ]
  },
  {  
    "start":18386372,
    "duration":119149,
    "interval":119149,
    "events":[  
      [  
        {  
          "reviewRecommended":true,
          "adultScore":0.00000,
          "racyScore":0.91902,
          "index":5085,
          "timestamp":18386372,
          "shotIndex":62
        }
      ]
    ]
```

## <a name="visualization-for-human-reviews"></a>方便人工审阅的直观界面

对于更注意细微差别的情况，企业需要有人工审阅解决方案，用于呈现视频及其帧和计算机分配的标记。 审阅视频和帧的人工审查方可以全面了解见解、更改标记和提交决定。

![视频“审阅”工具的默认视图](images/video-review-default-view.png)

## <a name="player-view-for-video-level-review"></a>方便视频级审阅的播放器视图

视频播放器视图可显示潜在成人内容和挑逗性内容帧，方便做出视频级二进制决定。 人工审阅者使用各种速度选项来导航视频，从而检查场景。 他们通过切换标记来确认自己的决定。

![视频“审阅”工具的播放器视图](images/video-review-player-view.PNG)

## <a name="frames-view-for-detailed-reviews"></a>方便细致审阅的帧视图

借助基于帧的视图，可以执行细致视频审阅，从而逐帧分析。 人工审阅者审阅并选择一个或多个帧，并通过切换标记来确认自己的决定。 下一步是，根据需要对冒犯性帧或内容进行编修。

![视频“审阅”工具的帧视图](images/video-review-frames-view-apply-tags.PNG)

## <a name="transcript-moderation"></a>脚本审查

视频通常有旁白，也需要审查旁白是否有冒犯性语音内容。 可以使用 Azure Media Indexer 服务将语音转文本，并使用内容审查器的审阅 API 提交脚本，以供在“审阅”工具中执行文本审查。

![视频“审阅”工具的脚本视图](images/video-review-transcript-view.png)

## <a name="next-steps"></a>后续步骤

- 开始学习[视频审查快速入门](video-moderation-api.md)。
- 了解如何根据审查输出为人工审阅者生成[视频审阅](video-reviews-quickstart-dotnet.md)。
- 向视频审阅添加[视频脚本审阅](video-transcript-reviews-quickstart-dotnet.md)。
- 查看有关如何开发[完整视频审查解决方案](video-transcript-moderation-review-tutorial-dotnet.md)的详细教程。