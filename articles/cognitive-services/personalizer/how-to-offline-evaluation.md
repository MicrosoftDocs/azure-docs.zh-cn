---
title: 脱机评估
titleSuffix: Personalizer - Azure Cognitive Services
description: 了解如何使用脱机评估分析学习循环
services: cognitive-services
author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/07/2019
ms.author: edjez
ms.openlocfilehash: e99a8242e438ef5a8ab7fd917724450f8080bb41
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025411"
---
# <a name="how-to-analyze-your-learning-loop-with-an-offline-evaluation"></a>如何使用脱机评估分析学习循环


了解如何完成脱机评估并理解结果。

使用脱机评估可以衡量个性化体验创建服务相比应用程序默认行为的有效性，了解哪些特征对个性化的作用最大，并自动发现新的机器学习设置。

请阅读[脱机评估](concepts-offline-evaluation.md)了解详细信息。


## <a name="prerequisites"></a>先决条件

1. 必须配置了一个个性化体验创建服务循环
1. 个性化体验创建服务循环日志中必须至少包含 50,000 个事件，这样才能生成有意义的评估结果。

（可选）还可以事先导出学习策略文件，以便在同一项评估中进行比较和测试。

## <a name="steps-to-start-a-new-offline-evaluation"></a>启动新脱机评估的步骤

1. 在 Azure 门户中找到你的个性化循环资源。
1. 导航到“评估”部分。
1. 单击“新建评估”
1. 选择脱机评估的开始日期和结束日期。 这些日期都已过去，指定要在评估中使用的数据范围。 这些数据必须根据[数据保留期](how-to-settings.md)设置中的指定存在于日志中。
1. （可选）可以上传自己的学习策略。 
1. 指定个性化体验创建服务是否应该基于此时段内观测到的用户行为创建优化的学习策略。
1. 启动评估

## <a name="results"></a>结果

运行评估可能需要花费较长时间，具体取决于要处理的数据量、要比较的学习策略数量，以及是否请求了优化。

完成后，可以看到以下结果：

1. 学习策略的比较，包括：
    * **联机策略**：个性化体验创建服务中当前使用的学习策略
    * **基线**：应用程序的默认设置（由排名调用中发送的第一个操作确定）
    * **随机策略**：一个假想的排名行为，始终从提供的操作中返回随机操作选项。
    * **自定义策略**：启动评估时上传的附加学习策略。
    * **优化策略**：如果使用发现优化策略的选项启动了评估，则也会比较该策略，你可以下载该策略，或将其设为联机学习策略，以替换当前策略。

1. 操作和上下文[特征](concepts-features.md)的有效性。


## <a name="more-information"></a>更多信息

* 了解[脱机评估的工作原理](concepts-offline-evaluation.md)。