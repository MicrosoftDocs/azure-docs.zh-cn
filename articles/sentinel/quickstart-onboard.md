---
title: 载入 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何在 Azure Sentinel 中收集数据。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: d5750b3e-bfbd-4fa0-b888-ebfab7d9c9ae
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 0061fb7946e4c35f855d1a409cb9bc210c73641a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65228311"
---
# <a name="on-board-azure-sentinel-preview"></a>载入 Azure Sentinel 预览

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

在本快速入门中，将了解如何载入 Azure Sentinel。 

为载入 Azure Sentinel，您首先需要启用 Azure Sentinel，，然后连接数据源。 Azure Sentinel 存在很多适用于 Microsoft 解决方案，可在带的框，并提供了实时的集成，包括 Microsoft 威胁防护解决方案，Microsoft 365 源，包括 Office 365，Azure AD 中，Azure ATP 连接器和Microsoft Cloud App Security，和的详细信息。 此外，内置的连接器可以拓宽非 Microsoft 解决方案的安全生态系统。 此外可以使用通用事件格式，Syslog 或 REST API 与 Azure Sentinel 连接数据源。  

数据源连接后，选择从熟练地创建仪表板的基于数据的见解传达库。 可以根据需要轻松地自定义这些仪表板。


## <a name="global-prerequisites"></a>全局系统必备组件

- 有效的 Azure 订阅，如果你没有帐户，创建[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)在开始之前。

- Log Analytics 工作区。 了解如何[创建 Log Analytics 工作区](../log-analytics/log-analytics-quick-create-workspace.md)

-  若要启用 Azure Sentinel，需要对 Azure Sentinel 工作区所在的订阅的参与者权限。 
- 若要使用 Azure Sentinel，需要对工作区所属的资源组的参与者或查看器权限
- 可能需要拥有其他权限来连接特定的数据源
 
## 启用 Azure Sentinel <a name="enable"></a>

1. 转到 Azure 门户。
2. 请确保已选择在其中创建 Azure Sentinel，订阅。 
3. 搜索 Azure Sentinel。 
   ![search](./media/quickstart-onboard/search-product.png)

1. 单击“+添加”。
1. 选择你想要使用或创建一个新的工作区。 可以在多个工作区中，运行 Azure Sentinel 但数据隔离到单个工作区。

   ![搜索](./media/quickstart-onboard/choose-workspace.png)

   >[!NOTE] 
   > - **工作区位置**务必了解流式传输到 Azure Sentinel 的所有数据都存储在所选工作区的地理位置中。  
   > - Azure 安全中心创建的默认工作区不会出现在列表中;不能在其上安装 Azure Sentinel。
   > - Azure Sentinel 可以运行工作区中，部署在以下区域：澳大利亚东南部、 加拿大中部、 印度中部、 美国东部、 美国东部 2 EUAP (Canary) 日本东部、 东南亚、 英国南部、 西欧、 美国西部 2。

6. 单击**添加 Azure Sentinel**。
  

## <a name="connect-data-sources"></a>连接数据源

Azure Sentinel 通过连接到服务并将事件和日志转发给 Azure Sentinel 创建与服务和应用程序的连接。 对于计算机和虚拟机，可以安装 Azure Sentinel 代理收集日志并将其转发到 Azure Sentinel。 有关防火墙和代理服务器，Azure Sentinel 利用 Linux Syslog 服务器。 在其上安装代理并从代理收集的日志文件，并将其转发到 Azure Sentinel。 
 
1. 单击**数据收集**。
2. 没有可以连接每个数据源的磁贴。<br>
例如，单击**Azure Active Directory**。 如果你连接此数据源，则日志流式传输所有从 Azure AD 到 Azure Sentinel。 可以选择哪种类型的日志要获取的登录日志和/或审核日志。 <br>
在底部，Azure Sentinel 提供的建议为哪些仪表板应安装每个连接器使你可以立即获取有趣见解数据。 <br> 按照安装说明进行操作或[，请参阅相关的连接指南](connect-data-sources.md)有关详细信息。 有关数据连接器的信息，请参阅[连接 Microsoft 服务](connect-data-sources.md)。

后数据源连接，你的数据启动到 Azure Sentinel 流式处理，并已准备就绪，若要开始使用。 您可以查看中的日志[的内置仪表板](quickstart-get-visibility.md)，并开始构建到 Log Analytics 中的查询[调查数据](tutorial-investigate-cases.md)。



## <a name="next-steps"></a>后续步骤
在本文档中，您学习了有关数据源连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
- Stream 中的数据[常见错误格式设备](connect-common-event-format.md)到 Azure Sentinel。
