---
title: 什么是个性化体验创建服务
titleSuffix: Azure Cognitive Services
description: Azure 个性化体验创建服务是基于云的 API 服务，可让你选择要显示给用户的最佳体验，并从其实时行为中学习信息。
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: 7eb85aa38815b8fcdfbe68518122563e1b579e17
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150397"
---
# <a name="what-is-personalizer"></a>什么是个性化体验创建服务？

Azure 个性化体验创建服务是基于云的 API 服务，可让你选择要显示给用户的最佳体验，并从其实时行为中学习信息。

* 提供有关用户和内容的信息，并接收要显示给用户的排名最高的操作。 
* 无需清理和标记数据即可使用个性化体验创建服务。
* 在方便的时候向个性化体验创建服务提供反馈。 
* 查看实时分析。 
* 在较大规模的数据科学工作中使用个性化体验创建服务来验证现有试验。

## <a name="how-does-personalizer-work"></a>个性化体验创建服务的工作原理

个性化体验创建服务使用机器学习模型来发现上下文中排名最高的操作。 客户端应用程序会提供可能操作的列表，其中包含有关这些操作的信息、有关上下文的信息，并且可能包含有关用户、设备等的信息。个性化体验创建服务会确定要执行的操作。 一旦客户端应用程序使用所选的操作，就会向个性化体验创建服务提供奖励评分形式的反馈。 反馈循环完成后，个性化体验创建服务会自动更新自身的模型以用于将来的排名。

## <a name="how-do-i-use-the-personalizer"></a>如何使用个性化体验创建服务？

![使用个性化体验创建服务来选择要向用户显示的视频](media/what-is-personalizer/personalizer-example-highlevel.png)

1. 在应用中选择要个性化的体验。
1. 在 Azure 门户中创建并配置个性化服务
1. 结合有关用户的信息（特征）以及内容（操作）使用 SDK 调用个性化体验创建服务。 使用个性化体验创建服务之前，无需提供清理的带标签的数据。 
1. 在客户端应用程序中，向用户显示个性化体验创建服务选择的操作。
1. 使用 SDK 向个性化体验创建服务提供反馈，指示用户是否选择了个性化体验创建服务的操作。 此反馈是一个奖励评分，值通常为 -1 到 1。
1. 在 Azure 门户中查看分析，以评估系统的工作状态以及数据为个性化提供的帮助大小。

## <a name="where-can-i-use-personalizer"></a>可在哪种场合下使用个性化体验创建服务？

例如，客户端应用程序可以添加个性化体验创建服务来执行以下操作：

* 个性化要在新闻网站上突出显示的文章。    
* 优化网站上的广告位置。
* 在购物网站上显示个性化的“推荐商品”。
* 建议要应用到特定照片的用户界面元素（例如筛选器）。
* 选择用于澄清用户意向或建议操作的聊天机器人响应。
* 确定用户要在业务流程的下一步骤中优先处理的操作建议。

## <a name="personalization-for-developers"></a>面向开发人员的个性化

个性化体验创建服务有两个 API：

* 发送有关用户的信息（特征）以及要个性化的内容（操作）。 个性化体验创建服务使用排名最高的操作做出响应。
* 将有关排名状态的反馈发送到个性化体验创建服务，该反馈通常是 0 到 1 的数字（在上一部分指出为 -1 到 1）。 

![个性化事件的基本顺序](media/what-is-personalizer/personalization-intro.png)

## <a name="next-steps"></a>后续步骤

* [快速入门：以 C# 创建反馈循环](csharp-quickstart-commandline-feedback-loop.md)
* [使用交互式演示](https://personalizationdemo.azurewebsites.net/)
