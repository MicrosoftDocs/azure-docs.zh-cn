---
title: 服务运行状况概述 | Microsoft Docs
description: 有关当前和将来的 Azure 服务问题及维护对 Azure 应用造成的影响的个性化信息。
services: service-health
author: stephbaron
ms.author: stbaron
documentationcenter: service-health
ms.service: service-health
ms.topic: article
ms.workload: Supportability
ms.date: 03/27/2018
ms.openlocfilehash: d5385777134b911aa08b751978c44597f79acbc7
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442395"
---
# <a name="service-health"></a>服务运行状况
服务运行状况提供可定制的仪表板，用于跟踪 Azure 服务在使用区域中的运行状况。 在此仪表板中，可跟踪活动事件，例如正在处理的服务问题、即将开始的计划内维护或相关运行状况公告。 当事件转变为非活动状态时，将在运行状况历史记录中保留最多 90 天。 最后，可使用服务运行状况仪表板创建和管理服务运行状况警报，受服务问题影响时会主动发出通知。

## <a name="service-health-events"></a>服务运行状况事件
服务运行状况跟踪三种可能会影响资源的运行状况事件：
1. **服务问题** - Azure 服务中目前已经造成了影响的问题。 
2. **计划内维护** - 即将进行的且将来可能会影响服务可用性的维护。  
3. **运行状况通告** - Azure 服务中发生的、需要引以注意的更改。 示例包括已弃用某些 Azure 功能，或超出了使用配额。

> [!NOTE]
> 若要查看服务运行状况事件，用户必须具有订阅读者角色。

## <a name="get-started-with-service-health"></a>服务运行状况入门
若要启动服务运行状况仪表板，请在门户仪表板上选择“服务运行状况”磁贴。 如果以前删除了该磁贴或者正在使用自定义仪表板，请在“更多服务”（仪表板左下角）中搜索“服务运行状况”服务。

![服务运行状况入门](./media/service-health-overview/azure-service-health-overview-1.png)

## <a name="see-current-issues-which-impact-your-services"></a>查看当前影响了服务的问题
“服务问题”视图显示 Azure 服务中当前正在影响资源的所有问题。 可以了解问题的开始时间，以及受影响的服务和区域。 还可以阅读最新的更新，了解 Azure 正在采取哪些措施来解决问题。 

![管理服务问题](./media/service-health-overview/azure-service-health-overview-2.png)

选择“潜在影响”选项卡查看自己拥有的、可能被问题影响的特定资源列表。 可以下载这些资源的 CSV 列表，以便与团队共享。

![管理服务问题 - 影响](./media/service-health-overview/azure-service-health-overview-4.png)

## <a name="get-links-and-downloadable-explanations"></a>获取链接和可下载的说明 
可以获取要在问题管理系统中使用的问题链接。 可以下载 PDF 文件（有时是 CSV 文件），以便与无法访问 Azure 门户的人员共享。   

![管理服务问题 - 问题管理](./media/service-health-overview/azure-service-health-overview-3.png)

## <a name="get-support-from-microsoft"></a>获取 Microsoft 支持
如果即使在解决问题之后，资源仍处于错误状态，请联系支持人员。  请使用页面右侧的支持链接。  

## <a name="pin-a-personalized-health-map-to-your-dashboard"></a>将个性化运行状况图固定到仪表板
筛选服务运行状况，显示业务关键型订阅、区域和资源类型。 保存筛选结果，将个性化运行状况全局图固定到门户仪表板。 

![筛选个性化运行状况图](./media/service-health-overview/azure-service-health-overview-6a.png)

![固定个性化运行状况图](./media/service-health-overview/azure-service-health-overview-6b.png)

## <a name="configure-service-health-alerts"></a>配置服务运行状况警报
服务运行状况与 Azure Monitor 集成，当业务关键型资源受到影响时，它可以通过电子邮件、短信和 Webhook 通知发出警报。 针对相应的服务运行状况事件设置活动日志警报。 使用“操作组”将该警报路由到组织中的相应人员。 有关详细信息，请参阅[配置服务运行状况的警报](../azure-monitor/platform/alerts-activity-log-service-notifications.md)

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OaXt]

## <a name="next-steps"></a>后续步骤
设置警报，以便收到运行状况问题的通知。 有关详细信息，请参阅[的最佳实践设置 Azure 服务运行状况警报](https://www.youtube.com/watch?v=k5d5ca8K6tc&list=PLLasX02E8BPBBSqygdRvlTnHfp1POwE8K&index=6&t=0s)。 
