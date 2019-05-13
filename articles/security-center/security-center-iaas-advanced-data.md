---
title: Azure 安全中心中的 IaaS 的高级数据安全 |Microsoft Docs
description: " 了解如何启用 Azure 安全中心中的 IaaS 的高级的数据安全性。 "
services: security-center
documentationcenter: na
author: monhaber
manager: barbkess
editor: monhaber
ms.assetid: ba46c460-6ba7-48b2-a6a7-ec802dd4eec2
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/2019
ms.author: monhaber
ms.openlocfilehash: e601bbaa0d15078fc2b19b5b7c536e3a1f6d20ad
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442742"
---
# <a name="advanced-data-security-for-sql-servers-on-iaas"></a>IaaS 上的 SQL 服务器的高级的数据安全性
IaaS 上的 SQL 服务器的高级的数据安全性是高级 SQL 安全功能的统一的包。 它当前包括用于呈现和缓解潜在的数据库漏洞和检测异常活动，可能指示威胁到你的数据库的功能。

为 IaaS SQL server 提供了此安全性基于中使用相同的基本技术[Azure SQL 数据库高级数据安全包](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)。


## <a name="overview"></a>概述

高级的数据安全 (ADS) 提供了一套高级 SQL 安全功能，包括漏洞评估和高级威胁防护。

* [漏洞评估](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)是一项易于配置的服务，可以发现、跟踪并帮助修正潜在的数据库漏洞。 它可深入了解你的安全状态，并包括解决安全问题和增强数据库 fortifications 的步骤。
* [高级威胁防护](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)可检测异常活动表示异常和潜在有害尝试访问或使用您的 SQL server。 它持续监视数据库的可疑活动和异常数据库访问模式上提供的面向操作的安全警报。 这些警报提供可疑活动详细信息，以及建议的操作，以帮助调查和缓解威胁。

## <a name="get-started-with-ads-for-iaas"></a>开始使用广告 iaas

以下步骤帮助你开始使用广告 iaas。

### <a name="set-up-ads-for-iaas"></a>为 IaaS 设置广告

**在开始之前**:您需要 Log Analytics 工作区来存储所分析的安全日志。 如果你没有订阅，则可以轻松地中所述创建一个[Azure 门户中创建 Log Analytics 工作区](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)。

1. 承载 SQL server 到 Log Analytics 工作区将 VM 连接。 有关说明，请参阅[连接 Windows 计算机连接到 Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/agent-windows)。

1. 从 Azure Marketplace 中，转到[SQL 高级数据安全解决方案](https://ms.portal.azure.com/#create/Microsoft.SQLAdvancedDataSecurity)。
（您可以找到它为在下图中，请参阅使用 marketplace 搜索选项。）**SQL 高级数据安全**页随即打开。

    ![IaaS 的高级的数据安全](./media/security-center-advanced-iaas-data/sql-advanced-data-security.png)

1. 单击**创建**。 显示工作区。

    ![创建高级的数据安全](./media/security-center-advanced-iaas-data/sql-advanced-data-create.png)

1. 选择要使用，并单击工作区**创建**。

   ![选择工作区](./media/security-center-advanced-iaas-data/sql-workspace.png)

1. 重新启动[SQL server VM 的](https://docs.microsoft.com/sql/database-engine/configure-windows/start-stop-pause-resume-restart-sql-server-services?view=sql-server-2017)。


## <a name="explore-and-investigate-security-alerts"></a>了解和调查安全警报

您可以查看和管理当前的安全警报。

1. 单击**安全中心** > **安全警报**，，然后单击一个警报。

    ![查找警报](./media/security-center-advanced-iaas-data/find-alert.png)

1. 从**受攻击资源**列中，单击受到攻击的资源。

1. 若要查看警报详细信息用于调查当前威胁和解决潜在威胁的操作，向下滚动**的常规信息**页上，然后在**修正步骤**部分中，单击**调查步骤**链接。

    ![修正步骤](./media/security-center-advanced-iaas-data/remediation-steps.png)

1. 若要查看与触发的警报相关联的日志，请转到**Log analytics 工作区**然后执行以下步骤：

     > [!NOTE]
     > 如果**Log analytics 工作区**不会出现在左侧菜单中，单击**的所有服务**，并搜索**Log analytics 工作区**。

    1. 确保显示的列**定价层**并**WorkspaceID**列。 (**Log analytics 工作区** > **编辑列**，将添加**定价层**并**WorkspaceID**。)

     ![编辑列](./media/security-center-advanced-iaas-data/edit-columns.png)

    1. 单击具有警报的日志的工作区。

    1. 下**常规**菜单上，单击**日志**

    1. 旁边单击眼睛**SQLAdvancedThreatProtection**表。 列出的日志。

     ![查看日志](./media/security-center-advanced-iaas-data/view-logs.png)

## <a name="set-up-email-notification-for-atp-alerts"></a>设置 ATP 警报的电子邮件通知 

你可以设置生成 ASC 警报时接收电子邮件通知的收件人列表。 电子邮件包含指向所有相关的详细信息在 Azure 安全中心警报的直接链接。 

1. 转到**安全中心** > **安全策略**和相关订阅单击的行中**编辑设置 >**。

    ![订阅设置](./media/security-center-advanced-iaas-data/subscription-settings.png)

1. 从**设置**菜单上，单击**电子邮件通知**。 
1. 在中**电子邮件地址**文字框中，输入要接收通知的电子邮件地址。 通过使用逗号 （，） 分隔的电子邮件地址，可以输入多个电子邮件地址。  例如admin1@mycompany.com，admin2@mycompany.com，admin3@mycompany.com

      ![电子邮件设置](./media/security-center-advanced-iaas-data/email-settings.png)

1. 在中**电子邮件通知**设置，设置以下选项：
  
    * **发送的高严重性警报的电子邮件通知**:而不是发送电子邮件中的所有警报，将仅的高严重性警报发送。
    * **此外向订阅所有者发送电子邮件通知**:也将通知发送到订阅所有者。

1. 在顶部**电子邮件通知**屏幕上，单击**保存**。

  > [!NOTE]
  > 请务必单击**保存**关闭窗口，或新前**电子邮件通知**将不会保存设置。

## <a name="explore-vulnerability-assessment-reports"></a>了解漏洞评估报告

漏洞评估仪表板概述了跨所有数据库的评估结果。 您可以查看分发数据库按照 SQL Server 版本，以及与传递数据库和总体摘要的故障根据风险分布的检查失败的摘要。

可以查看你的漏洞评估结果和直接从 Log Analytics 的报表。

1. 导航到 Log Analytics 工作区与广告解决方案。
1. 导航到**解决方案**，然后选择**SQL 漏洞评估**解决方案。
1. 在中**摘要**窗格中，单击**查看摘要**，然后选择你**SQL 漏洞评估报告**。

    ![SQL 评估报告](./media/security-center-advanced-iaas-data/ads-sql-server-1.png)

    报表仪表板将加载。 请确保时间范围设置为至少具有**过去 7 天**由于按固定计划每 7 天一次对数据库运行漏洞评估扫描。

    ![设置过去 7 天内](./media/security-center-advanced-iaas-data/ads-sql-server-2.png)

1. 若要深入了解更多详细信息，请单击上的任何仪表板元素。 例如:

   1. 单击在漏洞签入时**失败检查摘要**部分以查看具有此检查的结果的 Log Analytics 表在所有数据库。 首先列出了那些产生的结果。

   1. 然后，单击通过查看每个漏洞，包括漏洞说明和影响、 状态、 关联的风险，以及此数据库上的实际结果的详细信息。 此外可以查看已运行以执行此检查，并且修正信息用于解决此漏洞的实际查询。

    ![选择工作区](./media/security-center-advanced-iaas-data/ads-sql-server-3.png)

    ![选择工作区](./media/security-center-advanced-iaas-data/ads-sql-server-4.png)

1. 漏洞评估结果数据，进行切片和切块的数据根据需要，可以运行任何 Log Analytics 查询。

## <a name="advanced-threat-protection-for-sql-servers-on-iaas-alerts"></a>IaaS 警报的高级威胁防护功能用于 SQL Server
通过异常和潜在有害尝试访问或使用 SQL Server 将生成警报。 这些事件可以触发以下警报：

### <a name="anomalous-access-pattern-alerts"></a>异常访问模式的警报

* **从异常位置访问：** 当 SQL Server 的访问模式发生更改，有人从异常的地理位置登录到 SQL Server 时，会触发此警报。 可能的原因：
     * 攻击者或前者的恶意使用已访问你的 SQL Server。
     * 合法用户已从新位置访问你的 SQL Server。
* **从可能有害的应用程序访问**： 他警报触发时的可能有害的应用程序用于访问数据库。 可能的原因：
     * 攻击者尝试破坏 SQL 使用常见攻击工具。
     * 合法渗透测试操作中。
* **来自陌生主体的访问**：当 SQL Server 的访问模式发生更改，有人使用异常的主体（SQL 用户）登录到 SQL Server 时，会触发此警报。 可能的原因：
     * 攻击者或前者的恶意使用已访问你的 SQL Server。 
     * 合法用户已使用新的主体访问从 SQL Server。
* **暴力破解 SQL 凭据**：当有人使用不同的凭据异常登录并失败很多次时，会触发此警报。 可能的原因：
     * 攻击者尝试破坏使用暴力破解 SQL。
     * 合法渗透测试操作中。

### <a name="potential-sql-injection-attacks-coming"></a>潜在 SQL 注入式攻击 （多）

* **存在易受 SQL 注入攻击的漏洞**：当某个应用程序在数据库中生成错误的 SQL 语句时，会触发此警报。 此警报会指示可能存在易受 SQL 注入攻击的漏洞。 可能的原因：
     * 应用程序代码中的缺陷导致构造出错误的 SQL 语句
     * 应用程序代码或存储过程在构造错误的 SQL 语句时无法清理用户输入，使该语句被 SQL 注入攻击利用
* **潜在 SQL 注入**：当 SQL 攻击有效利用已识别到的应用程序漏洞时，会触发此警报。 这意味着，攻击者正在尝试使用有漏洞的应用程序代码或存储过程注入恶意 SQL 语句。
