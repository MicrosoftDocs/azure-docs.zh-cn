---
title: 将模型部署为自定义语音的语音服务
titlesuffix: Azure Cognitive Services
description: 在本文档中，将学习如何创建和部署使用自定义语音门户为终结点。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: fc51c1d9d47340da85d42f7c398c8ee21c601beb
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025866"
---
# <a name="deploy-a-custom-model"></a>将自定义模型部署

已上传和检查数据，评估准确性，并且自定义模型定型后，可以部署自定义的终结点，以用于应用程序、 工具和产品。 在本文档中，将学习如何创建和部署使用自定义语音门户为终结点。

## <a name="create-a-custom-endpoint"></a>创建自定义终结点

若要创建新的自定义终结点，请选择**部署**从页面顶部的自定义语音菜单。 如果这是你首次运行时，您会注意到没有终结点表中列出。 创建一个终结点后，将使用此页用于跟踪每个部署的终结点。

接下来，选择**添加终结点**并输入**名称**并**说明**为自定义终结点。 然后选择你想要将与此终结点相关联的自定义模型。 从此页中，您还可以启用日志记录。 日志记录可用于监视终结点流量。 如果禁用，不会存储流量。

![如何将模型部署](./media/custom-speech/custom-speech-deploy-model.png)

> [!NOTE]
> 别忘了接受使用条款和定价的详细信息。

接下来，选择“创建”。 此操作将返回到**部署**页。 现在将包括到自定义终结点相对应的条目。 终结点的状态会显示其当前状态。 它可能最多 30 分钟的时间来实例化新的终结点使用自定义模型。 当部署的状态更改为**完成**，终结点是可供使用。

部署你的终结点后，终结点名称将显示为链接。 单击链接以向你的终结点，例如终结点密钥、 终结点 URL 和示例代码显示特定的信息。

## <a name="view-logging-data"></a>查看日志记录数据

日志记录数据已可供下载下**终结点 > 详细信息**。

## <a name="next-steps"></a>后续步骤

* 使用与你自定义终结点[语音 SDK](speech-sdk.md)

## <a name="additional-resources"></a>其他资源

* [准备和测试你的数据](how-to-custom-speech-test-data.md)
* [检查你的数据](how-to-custom-speech-inspect-data.md)
* [评估你的数据](how-to-custom-speech-evaluate-data.md)
* [训练模型](how-to-custom-speech-train-model.md)
* [将模型部署](how-to-custom-speech-deploy-model.md)
