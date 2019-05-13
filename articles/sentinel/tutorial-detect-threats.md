---
title: 调查警报与 Azure Sentinel 预览版 |Microsoft Docs
description: 使用本教程，了解如何调查警报与 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: b5fbc5ac-68b2-4024-9c1b-bd3cc41a66d0
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/20/2019
ms.author: rkarlin
ms.openlocfilehash: 319ec5d09a6daddb5c1fc36f680ee6d0d856e337
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205428"
---
# <a name="tutorial-detect-threats-with-azure-sentinel-preview"></a>教程：检测到威胁 Azure Sentinel 预览版

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

检查完[连接的数据源](quickstart-onboard.md)要向 Azure Sentinel，一些可疑发生时收到通知。 若要使您能够执行此操作，Azure Sentinel 的，您可以创建高级可以分配的情况下生成的警报的规则和使用进行深入地调查异常和环境中的威胁。 

本教程帮助用户检测到威胁 Azure Sentinel。
> [!div class="checklist"]
> * 创建检测规则
> * 应对威胁

## <a name="create-detection-rules"></a>创建检测规则

若要调查情况下，首先需要创建检测规则。 

> [!NOTE]
> 在 Azure Sentinel 中生成的警报都可以通过[Microsoft Graph Security](https://aka.ms/securitygraphdocs)。 请参阅[Microsoft Graph 安全警报文档](https://aka.ms/graphsecurityreferencebetadocs)更多详细信息和集成合作伙伴。

检测规则基于类型的威胁和异常的可能是你想要立即，了解有关在环境中可疑确保它们进行显示、 调查，并进行修正。 

1. 在 Azure 门户中 Azure Sentinel 下，选择**Analytics**。

   ![分析](./media/tutorial-detect-threats/alert-rules.png)

2. 在顶部菜单栏中，单击 **+ 添加**。  

   ![创建警报规则](./media/tutorial-detect-threats/create-alert-rule.png)

3. 下**创建警报规则**，提供一个描述性名称，并设置**严重性**根据需要。 

4. 在 Log Analytics 中创建查询，并将其粘贴到**设置预警规则**字段。 下面是一个查询示例，当在 Azure 活动中创建异常数量的资源时将通知你。

        AzureActivity
        | where OperationName == "Create or Update Virtual Machine" or OperationName == "Create Deployment"
        | where ActivityStatus == "Succeeded"
        | make-series dcount(ResourceId)  default=0 on EventSubmissionTimestamp in range(ago(7d), now(), 1d) by Caller

5. 在中**实体映射**部分中，使用下的字段**实体类型**在查询中的列映射到由 Azure Sentinel 识别实体字段。 对于每个字段映射到相应的实体字段的 Log Analytics 中创建的查询中的相关列。 选择下面的相关列名称**属性**。 每个实体包含多个字段，例如 SID、 GUID，等等。可以将根据任何字段，而不仅仅是上部级别实体的实体映射。

6. 定义警报触发器条件下的**警报触发**。 这将定义触发警报的条件。 

7. 设置**频率**的查询的运行频率-作为经常每 5 分钟或很少一天一次。 

8. 设置**期**来控制在查询运行的数据量的时间窗口，它可以每隔一小时跨运行 60 分钟的数据。

9. 您还可以设置**抑制**。 如果想要停止的对同一事件触发的重复警报抑制非常有用。 在这种方式，可以停止的特定段触发的警报。 这可以帮助您避免了同一事件的重复警报，并允许您为一段时间内阻止连续警报。 例如，如果**警报计划****频率**设置为 60 分钟，并**警报计划期间**设置为两个小时和查询结果超过定义阈值，它会触发一个警报两次之后当它首次检测过去 60 分钟，, 并再次时在第一个 60 分钟内的数据进行采样的 2 小时。 我们建议，如果触发警报，抑制应该在警报时间段中设置的时间量内。 在本示例中，你可能想要设置的 60 分钟，阻止功能，以便在最近一小时内发生的事件才会触发警报。

8. 粘贴到查询后**设置预警规则**字段中，可以立即看到模拟下的提醒**逻辑警报模拟**，以便您能够了解的数据量将您创建的警报的特定时间间隔内生成。 这将取决于您为设置**频率**并**阈值**。 如果你看到，一般情况下，你将触发警报太过频繁，将想要设置的更高版本的结果数，以便它高于平均基线。

9. 单击**创建**初始化警报规则。 创建警报后，一种情况被创建包含的警报。 中的行，可以看到定义的检测规则**安全分析**选项卡。此外可以查看每个规则的触发的警报的匹配数。 此列表中可以启用、 禁用或删除每个规则。 您还可以右-选择在每个警报编辑、 禁用、 克隆、 显示匹配项，或删除规则所对应的行末尾的省略号 （...）。 **Analytics**页是一个库的所有活动警报规则，包括模板您启用并根据模板创建警报规则。

1. 警报规则的结果所示**情况下**页上，您可以在其中会审[调查情况下](tutorial-investigate-cases.md)，并修正的威胁。



## <a name="respond-to-threats"></a>应对威胁

Azure Sentinel 提供两个主要选项用于响应使用 playbook 的威胁。 您可以设置触发警报，或可以手动在响应警报运行 playbook 时自动运行操作手册。

- 设置操作手册配置操作手册时触发警报时自动运行。 

- 手动运行通过单击该警报，请在从 playbook**查看 playbook** ，然后选择 playbook 运行。




## <a name="next-steps"></a>后续步骤
在本教程中，您学习了如何开始使用 Azure Sentinel 的威胁检测。 

若要了解如何自动执行对威胁，你做出响应[如何响应使用自动化操作手册的威胁](tutorial-respond-threats-playbook.md)。
> [!div class="nextstepaction"]
> [应对威胁](tutorial-respond-threats-playbook.md)来自动执行对威胁你做出响应。

