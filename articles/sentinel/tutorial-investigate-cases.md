---
title: 调查情况下使用 Azure Sentinel 预览版 |Microsoft Docs
description: 使用本教程，了解如何调查情况下使用 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: a493cd67-dc70-4163-81b8-04a9bc0232ac
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 3/6/2019
ms.author: rkarlin
ms.openlocfilehash: d1da180d3b30b57ca2b69985bf3d0261b8d70c39
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205509"
---
# <a name="tutorial-investigate-cases-with-azure-sentinel-preview"></a>教程：调查情况下使用 Azure Sentinel 预览版

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本教程帮助用户检测到威胁 Azure Sentinel。

检查完[连接的数据源](quickstart-onboard.md)要向 Azure Sentinel，一些可疑发生时收到通知。 若要使您能够执行此操作，Azure Sentinel 的，您可以创建高级可以分配的情况下生成的警报的规则和使用进行深入地调查异常和环境中的威胁。 

> [!div class="checklist"]
> * 创建用例
> * 调查案例
> * 应对威胁

## <a name="investigate-cases"></a>调查案例

一种情况可以包含多个警报。 它是一个特定的调查相关的所有证据的聚合。 基于警报定义中创建一种情况**Analytics**页。 在事例级别设置，将发出警报严重性和状态等相关的属性。 让 Azure Sentinel 知道哪些类型的所需的威胁，以及如何找到它们后，可以监视通过调查情况下检测到的威胁。 

1. 选择**情况下**。 **情况下**页可让您知道有多少用例、 多少会打开，多少你已设置为**正在**，并且关闭了多少。 对于每种情况下，您可以看到它发生的时间和用例的状态。 看看严重地决定如何处理第一个。 在中**情况下**页上，单击**警报**选项卡以查看到用例相关的所有警报。 可以在中查看这种情况的一部分之前映射的实体**实体**选项卡。您可以筛选的情况下，根据需要例如按状态或严重性。 当您查看**情况下**选项卡上，你将看到包含触发检测规则中定义的警报的未结案例**分析**。 将在顶部看到活动的情况下，新的用例在进度情况。 此外可以按严重性查看所有事例的概述。

   ![警报仪表板](./media/tutorial-investigate-cases/cases.png)

2. 若要开始调查，请单击特定的用例。 在右侧，可以看到这种情况，包括其严重性，所涉及实体的计数 （基于您的映射） 的详细的信息。 每种情况下都有唯一的 id。 根据最严重的警报包含在这种情况，确定用例的严重性。  

1. 若要在这种情况中查看有关警报以及实体的更多详细信息，请单击**查看完整的详细信息**在这种情况中页，并查看汇总的事例的信息的相关选项卡。  完整例视图将合并警报、 关联的警报和实体中的所有证据。

1. 在中**警报**选项卡上，查看警报本身的其触发时和通过多少它超出了你设定的阈值。 可以查看有关警报 – 触发该警报，每个查询，并能够对警报运行 playbook 返回的结果数的查询的所有相关信息。 若要向下钻取向下进一步到用例中，单击命中次数。 这将打开生成结果以及触发该警报在 Log Analytics 中的结果的查询。

3. 在中**实体**选项卡上，您所见，映射的所有实体的警报规则定义的一部分。 

4. 如果主动要研究一种情况，则将事例的状态设置为一个好办法**正在进行中**状态，直到关闭它。 此外可以关闭这种情况，其中**关闭已解决**的状态指示已处理事件的情况下虽然**关闭已消除**不需要处理的情况下的状态。 需要说明用于关闭种情况下你原因说明。

5. 情况下可以分配给特定用户。 通过设置这种情况，可以将所有者分配每个用例**所有者**字段。 作为未分配的所有情况下启动。 可以转到用例并筛选名称以查看你拥有的所有情况。 

5. 单击**调查**查看调查图和安全违规的带有修正步骤的作用域。 



## <a name="respond-to-threats"></a>应对威胁

Azure Sentinel 提供两个主要选项用于响应使用 playbook 的威胁。 您可以设置触发警报，或可以手动在响应警报运行 playbook 时自动运行操作手册。

- 您可以设置操作手册配置操作手册时触发警报时自动运行。 

- 通过单击可以手动运行在该警报，请从 playbook**查看 playbook** ，然后选择 playbook 运行。




## <a name="next-steps"></a>后续步骤
在本教程中，您学习了如何开始调查情况下使用 Azure Sentinel。 继续学习以下教程[如何响应使用自动化操作手册的威胁](tutorial-respond-threats-playbook.md)。
> [!div class="nextstepaction"]
> [应对威胁](tutorial-respond-threats-playbook.md)来自动执行对威胁你做出响应。

