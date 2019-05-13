---
title: 将 Azure Log Analytics 数据导入 Power BI | Microsoft 文档
description: Power BI 是 Microsoft 的一种基于云的业务分析服务，它为不同数据集的分析提供了丰富的可视化功能和报告。  本文介绍如何配置并将 Log Analytics 数据导入 Power BI 且将其配置为自动刷新。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 83edc411-6886-4de1-aadd-33982147b9c3
ms.service: log-analytics
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/01/219
ms.author: bwren
ms.openlocfilehash: 2db6ddf57802f6fcf38cfc3ad7094ed94eaca3d8
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234204"
---
# <a name="import-azure-monitor-log-data-into-power-bi"></a>Azure Monitor 日志数据导入到 Power BI


[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) 是 Microsoft 的一种基于云的业务分析服务，它为不同数据集的分析提供了丰富的可视化功能和报告。  可以导入 Power BI 数据集的 Azure Monitor 日志查询的结果以便可以充分利用其功能，例如合并不同来源的数据和共享 web 和移动设备上的报表。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="overview"></a>概述
若要从中导入数据[Log Analytics 工作区](manage-access.md)在 Azure 监视器中导入 Power BI，您可以创建基于 Power BI 中的数据集[日志查询](../log-query/log-query-overview.md)Azure 监视器中。  每次刷新数据集时都会运行查询。  随后你即可生成使用数据集中的数据的 Power BI 报表。  若要在 Power BI 中创建数据集，你可以将查询从 Log Analytics 导出为 [Power Query (M) 语言](https://msdn.microsoft.com/library/mt807488.aspx)。  然后，你可以使用其在 Power BI Desktop 中创建查询，再将其作为数据集发布到 Power BI。  此过程的详细信息如下所述。

![Log Analytics 到 Power BI](media/powerbi/overview.png)

## <a name="export-query"></a>导出查询
首先，创建[日志查询](../log-query/log-query-overview.md)返回要填充 Power BI 数据集的数据。  随后将该查询导出为 Power BI Desktop 可以使用的 [Power Query (M) 语言](https://msdn.microsoft.com/library/mt807488.aspx)。

1. [在 Log Analytics 中创建日志查询](../log-query/get-started-portal.md)以提取你的数据集的数据。
2. 选择**导出** > **Power BI Query (M)**。  这将查询导出到名为文本文件**PowerBIQuery.txt**。 

    ![导出日志搜索](media/powerbi/export-analytics.png)

3. 打开该文本文件并复制其内容。

## <a name="import-query-into-power-bi-desktop"></a>将查询导入到 Power BI Desktop
Power BI Desktop 是一个桌面应用程序，用于创建可发布到 Power BI 的数据集和报表。  此外可以使用它来创建使用从 Azure Monitor 中导出的 Power Query 语言的查询。 

1. 请安装 [Power BI Desktop](https://powerbi.microsoft.com/desktop/)（如果你尚未安装），然后打开该应用程序。
2. 选择“获取数据” > “空查询”，以打开一个新查询。  然后选择“高级编辑器”并将导出文件的内容粘贴到查询中。 单击“完成”。

    ![Power BI Desktop 查询](media/powerbi/desktop-new-query.png)

5. 查询将运行并显示其结果。  系统可能会提示你输入凭据以连接到 Azure。  
6. 为查询键入一个描述性的名称。  默认值为“Query1”。 单击“关闭并应用”，以向报表添加数据集。

    ![Power BI Desktop](media/powerbi/desktop-results.png)



## <a name="publish-to-power-bi"></a>发布到 Power BI
当你发布到 Power BI 时，将创建数据集和报表。  如果你是在 Power BI Desktop 中创建报表，则发布的报表将包含你的数据。  如果不是，则会创建一个空白报表。  你可以在 Power BI 中修改报表或基于数据集创建新报表。

1. 基于你的数据创建报表。  请使用 [Power BI Desktop 文档](https://docs.microsoft.com/power-bi/desktop-report-view)（如果你不熟悉此应用程序）。  
1. 如果你已准备好将其发送到 Power BI，请单击“发布”。  
1. 出现提示时，请在 Power BI 帐户中选择目标。  除非你有特定的目标，否则请使用“我的工作区”。

    ![Power BI Desktop 发布](media/powerbi/desktop-publish.png)

1. 发布完成后，单击“在 Power BI 中打开”，以在 Power BI 中打开新数据集。


### <a name="configure-scheduled-refresh"></a>配置计划的刷新
在 Power BI 中创建的数据集中包含的数据与你以前在 Power BI Desktop 中看到的数据一样。  需要刷新定期要再次运行查询并使用 Azure Monitor 中的最新数据填充它的数据集。  

1. 单击要在其中上传报表的工作区并选择“数据集”菜单。 
1. 选择你的新数据集旁边的上下文菜单，然后选择“设置”。 
1. 在“数据源凭据”下，应显示一条消息，提示凭据无效。  这是因为你尚未提供在刷新数据时要使用的数据集的凭据。  
1. 单击**编辑凭据**并有权访问 Azure Monitor 中的 Log Analytics 工作区中指定的凭据。 如果需要双因素身份验证，请选择**OAuth2**有关**身份验证方法**，会提示您使用您的凭据登录。

    ![Power BI 计划](media/powerbi/powerbi-schedule.png)

5. 在“计划的刷新”下，开启“使你的数据保持为最新”选项。  你可以视情况更改“刷新频率”和运行刷新的一个或多个特定时间。

    ![Power BI 刷新](media/powerbi/powerbi-schedule-refresh.png)



## <a name="next-steps"></a>后续步骤
* 了解有关[日志搜索](../log-query/log-query-overview.md)的信息以生成可导出到 Power BI 的查询。
* 详细了解如何[Power BI](https://powerbi.microsoft.com)来构建基于 Azure Monitor 日志导出的可视化效果。