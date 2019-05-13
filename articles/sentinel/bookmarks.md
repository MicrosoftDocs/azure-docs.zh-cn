---
title: 跟踪的数据，而在 Sentinel 使用搜索书签预览的 Azure 搜索 |Microsoft Docs
description: 本文介绍如何使用 Azure Sentinel 搜索书签来跟踪数据。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 320ccdad-8767-41f3-b083-0bc48f1eeb37
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2019
ms.author: rkarlin
ms.openlocfilehash: b1a438b9645dbb37d852eb0092355850d816872d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65207479"
---
# <a name="keep-track-of-data-during-hunting"></a>在搜索期间跟踪的数据

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
 
威胁搜索通常需要查看巨额的日志数据查找的恶意行为的证据。 在此过程中，调查人员找到他们想要记住，巩固概念，并作为验证潜在的假设和了解安全威胁的完整故事的一部分进行分析的事件。
搜索书签可以帮助您执行此操作，请通过保留以及您认为相关的查询结果的 Log Analytics 中运行的查询。 此外可以记录你上下文的观测值并按说明和标记中添加引用你的发现。 标有书签的数据会显示给你和你的团队成员轻松协作。   

您可以重新访问你用书签标记的数据在任何时候上**书签**选项卡**搜索**页。 您可以使用筛选和搜索选项可以快速为您当前的调查中查找特定的数据。 或者，可以查看用书签标记的数据中直接**HuntingBookmark** Log Analytics 中的表。 这使您筛选、 汇总，并将标有书签的数据与其他数据源，从而轻松查找证据确凿联接。

此外可以通过单击可视化你用书签标记的数据，**调查**。 这将启动调查体验在其中可以查看、 调查并以可视方式通信你使用的是交互式的实体关系图关系图和时间线的发现。


## <a name="run-a-log-analytics-query-from-azure-sentinel"></a>从 Azure Sentinel 运行 Log Analytics 查询

1. 在 Azure Sentinel 门户中，单击**搜索**运行可疑和异常行为的查询。

1. 若要运行搜索市场活动，选择一个搜索查询并在左侧，查看结果。 

1. 单击**查看查询结果**在搜索查询**详细信息**页可以查看的查询结果在 Log Analytics 中。 下面是您看到是否运行自定义的 SSH 暴力攻击查询的示例。
  
   ![显示结果](./media/bookmarks/ssh-bruteforce-example.png)

## <a name="add-a-bookmark"></a>添加书签

1. 在 Log Analytics 查询结果列表中，展开包含您感兴趣的信息的行。

4. 选择行末尾处的省略号 （...），然后选择**添加搜索书签**。
5. 在右侧，在**详细信息**页上，更新的名称，并添加标记和说明以帮助你确定是什么有趣的项。
6. 单击**保存**提交所做的更改。 用书签标记的所有数据与其他调查人员共享，协作的调查体验的第一步。

   ![显示结果](./media/bookmarks/add-bookmark-la.png)

 
> [!NOTE]
> 此外可以使用任意启动从 Azure Sentinel 日志分析日志页上或创建上飞入 Log Analytics 页上，从搜索页上打开的查询的 Log Analytics 查询中使用书签。 你将无法再添加书签，如果您启动 Log Analytics 将从外部 Azure Sentinel。 

## <a name="view-and-update-bookmarks"></a>查看和更新书签 

1. 在 Azure Sentinel 门户中，单击**搜索**。 
2. 单击**书签**选项卡的中间页后，可以查看的书签列表。
3. 使用搜索框或筛选器选项来查找特定书签。
4. 下面的网格中选择各个书签右侧详细信息窗格中查看书签详细信息。
5. 若要更新标记和说明，单击可编辑的文本框，然后单击**保存**保留所做的更改。

   ![显示结果](./media/bookmarks/view-update-bookmarks.png)

## <a name="view-bookmarked-data-in-log-analytics"></a>在 Log Analytics 中查看用书签标记的数据 

有多个选项为 Log Analytics 中查看你用书签标记的数据。 

若要查看标有书签的查询、 结果或历史记录的最简单方法是通过选择在所需的书签**书签**表，以使用详细信息窗格中提供的链接。 选项包括： 
- 单击**视图查询**Log Analytics 中查看源查询。  
- 单击**查看书签历史记录**若要查看所有书签元数据，包括： 谁做了更新，更新后的值，并更新发生的时间。 

- 此外可以通过单击查看所有书签的原始书签数据**书签日志**书签网格的上方。 该视图将显示具有关联的元数据的搜索书签表中所有书签。 可以使用 KQL 查询以筛选出您正在寻找的特定书签的最新版本。  


> [!NOTE]
> 一个书签，在创建和中显示时之间可以有长的延迟 （以分钟计） **HuntingBookmark**表。 建议创建您的书签，然后对其进行分析后的数据引入。 

## <a name="delete-a-bookmark"></a>删除一个书签
如果你想要删除书签，请执行以下： 
1.  打开第**搜索书签**选项卡。 
2.  选择目标书签。
3.  选择省略号 （...） 末尾的行并选择**删除书签**。
    
正在删除该书签从列表中删除的书签**书签**选项卡。Log Analytics"HuntingBookmark"表仍将包含上一书签条目，但会更改的最新条目**SoftDelete**值设为 true，就可以更轻松筛选出旧的书签。  删除一个书签，则不删除任何实体与其他书签或警报相关联的调查体验。 


## <a name="next-steps"></a>后续步骤

在本文中，您学习了如何运行在 Azure Sentinel 中使用书签搜索调查。 要详细了解 Azure Sentinel，请参阅以下文章：


- [主动寻找威胁](hunting.md)
- [使用笔记本来运行自动的搜索活动](notebooks.md)