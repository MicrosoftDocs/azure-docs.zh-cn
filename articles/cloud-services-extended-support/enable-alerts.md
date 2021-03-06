---
title: '使用 Azure 门户在云服务中启用监视 (扩展支持) '
description: 使用 Azure 门户为云服务 (扩展支持) 实例启用监视
ms.topic: how-to
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 4591253e1a305632b7f41afe692f297d71366382
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "98744256"
---
# <a name="enable-monitoring-for-cloud-services-extended-support-using-the-azure-portal"></a>使用 Azure 门户为云服务启用监视 (扩展支持) 

本文介绍如何在现有云服务上启用警报 (扩展支持) 部署。 

## <a name="add-monitoring-rules"></a>添加监视规则
1. 登录到 [Azure 门户](https://portal.azure.com)。 
2. 选择要为其启用警报) 部署的云服务 (扩展支持。 
3. 选择 " **警报** " 边栏选项卡。 

    :::image type="content" source="media/enable-alerts-1.png" alt-text="图像显示在 Azure 门户中选择 &quot;警报&quot; 选项卡。":::

4. 选择 " **新建警报** " 图标。
     :::image type="content" source="media/enable-alerts-2.png" alt-text="图像显示选择 &quot;添加新警报&quot; 选项。":::

5. 根据要跟踪的度量值输入所需的条件和所需的操作。 您可以基于单个度量值或活动日志定义规则。 

     :::image type="content" source="media/enable-alerts-3.png" alt-text="Image 显示了向警报添加条件的位置。":::

     :::image type="content" source="media/enable-alerts-4.png" alt-text="图像显示配置信号逻辑。":::

     :::image type="content" source="media/enable-alerts-5.png" alt-text="图像显示配置操作组逻辑。":::

6. 完成设置警报后，保存所做的更改，并根据配置的指标，将开始查看 " **警报** " 边栏选项卡。

## <a name="next-steps"></a>后续步骤 
- 查看 [云服务的常见问题 (](faq.md) 扩展支持) 。
- ) 使用 [Azure 门户](deploy-portal.md)、 [PowerShell](deploy-powershell.md)、 [模板](deploy-template.md) 或 [Visual Studio](deploy-visual-studio.md)部署云服务 (扩展支持。