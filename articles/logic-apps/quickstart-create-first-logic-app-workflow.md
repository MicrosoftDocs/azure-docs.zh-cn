---
title: 快速入门 - 创建并自动执行第一个工作流 - Azure 逻辑应用 | Microsoft Docs
description: 使用 Azure 逻辑应用创建第一个自动执行任务、流程和工作流的逻辑应用。 为系统和云服务创建用于系统集成和企业应用程序集成 (EAI) 解决方案的逻辑应用
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: quickstart
ms.custom: mvc
ms.date: 07/20/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 90c20bcd289c3bcf4a7fa3c41d245f2b7f159582
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154105"
---
# <a name="quickstart-create-your-first-automated-workflow-with-azure-logic-apps---azure-portal"></a>快速入门：使用 Azure 逻辑应用创建第一个自动化工作流 - Azure 门户

本快速入门介绍如何使用 [Azure 逻辑应用](../logic-apps/logic-apps-overview.md)构建第一个自动化工作流。 在本文中，请创建一个逻辑应用，以便定期查看网站的 RSS 源中是否有新项。 如果存在新项，逻辑应用会针对每个项发送电子邮件。 完成后，逻辑应用看起来大致与以下工作流类似：

![概述 - 逻辑应用示例](./media/quickstart-create-first-logic-app-workflow/overview.png)

若要按照本快速入门进行操作，你需要逻辑应用支持的提供商（例如 Office 365 Outlook、Outlook.com 或 Gmail）提供的电子邮件帐户。 至于其他提供商，请[查看此处的连接器列表](https://docs.microsoft.com/connectors/)。 此逻辑应用使用 Office 365 Outlook 帐户。 如果使用其他电子邮件帐户，整个步骤仍然是相同的，但 UI 可能稍有不同。 

另外，如果还没有 Azure 订阅，请[注册一个免费 Azure 帐户](https://azure.microsoft.com/free/)。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

使用 Azure 帐户凭据登录到 [Azure 门户](https://portal.azure.com)。

## <a name="create-your-logic-app"></a>创建逻辑应用 

1. 在 Azure 主菜单中，选择“创建资源” > “集成” > “逻辑应用”。

   ![创建逻辑应用](./media/quickstart-create-first-logic-app-workflow/create-logic-app.png)

1. 在“创建逻辑应用”下，提供有关逻辑应用的详细信息，如下所示。 完成后，选择“创建”。

   ![提供逻辑应用的详细信息](./media/quickstart-create-first-logic-app-workflow/create-logic-app-settings.png)

   | 属性 | 值 | 说明 | 
   |----------|-------|-------------| 
   | **名称** | MyFirstLogicApp | 逻辑应用的名称 | 
   | **订阅** | <*your-Azure-subscription-name*> | Azure 订阅的名称 | 
   | **资源组** | My-First-LA-RG | 用于组织相关资源的 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)的名称 | 
   | **位置** | 美国西部 | 用于存储逻辑应用信息的区域 | 
   | **Log Analytics** | 关闭 | 对于诊断日志记录，请保留“关闭”设置。 | 
   |||| 

1. 在 Azure 部署应用以后，逻辑应用设计器会打开并显示一个包含简介视频和常用触发器的页面。 在“模板”下选择“空白逻辑应用”。

   ![选择空白逻辑应用模板](./media/quickstart-create-first-logic-app-workflow/choose-logic-app-template.png)

接下来请添加一个[触发器](../logic-apps/logic-apps-overview.md#logic-app-concepts)，该触发器在出现新的 RSS 源项时触发。 每个逻辑应用都必须从触发器开始，该触发器在发生特定事件或特定条件得到满足的情况下触发。 每当触发器触发时，逻辑应用引擎就会创建一个逻辑应用实例来启动并运行工作流。

<a name="add-rss-trigger"></a>

## <a name="check-rss-feed-with-a-trigger"></a>使用触发器检查 RSS 源

1. 在逻辑应用设计器的搜索框下，选择“全部”。

1. 在搜索框中输入“rss”。 从触发器列表中选择此触发器：**当有源项发布时 - RSS**

   ![选择触发器：“RSS - 发布源项时”](./media/quickstart-create-first-logic-app-workflow/add-trigger-rss.png)

1. 为触发器提供如下所示的信息：

   ![使用 RSS 源、频率和间隔设置触发器](./media/quickstart-create-first-logic-app-workflow/add-trigger-rss-settings.png)

   | 属性 | 值 | 说明 | 
   |----------|-------|-------------| 
   | **RSS 源 URL** | ```http://feeds.reuters.com/reuters/topNews``` | 要监视的 RSS 源的链接 | 
   | 间隔 | 1 | 在两次检查之间需等待的时间间隔数 | 
   | **频率** | 分钟 | 两次检查的间隔的时间单位  | 
   |||| 

   时间间隔和频率合在一起，即可定义逻辑应用的触发器的计划。 
   此逻辑应用每分钟检查一次源。

1. 若要立即隐藏触发器详细信息，请单击触发器的标题栏。

   ![折叠形状即可隐藏详细信息](./media/quickstart-create-first-logic-app-workflow/collapse-trigger-shape.png)

1. 保存逻辑应用。 在设计器工具栏上，选择“保存”。

逻辑应用现已生成，但除了检查 RSS 源，不能执行任何操作。 因此，请添加一项在触发器触发时进行响应的操作。

## <a name="send-email-with-an-action"></a>使用操作发送电子邮件

现在请添加一项[操作](../logic-apps/logic-apps-overview.md#logic-app-concepts)，在 RSS 源中出现新项时发送电子邮件。

1. 在“发布源项时”触发器下，选择“新建步骤”。

   ![添加操作](./media/quickstart-create-first-logic-app-workflow/add-new-action.png)

1. 在“选择操作”和搜索框下，选择“全部”。

1. 在搜索框中，输入“发送电子邮件”。 从操作列表中，为所需的 电子邮件提供商选择“发送电子邮件”操作。

   ![选择此操作：“Office 365 Outlook - 发送电子邮件”](./media/quickstart-create-first-logic-app-workflow/add-action-send-email.png)

   若要通过对操作列表进行筛选来查找特定的应用或服务，可以先选择应用或服务：

   * 对于 Azure 工作或学校帐户，请选择“Office 365 Outlook”。 
   * 对于个人 Microsoft 帐户，请选择“Outlook.com”。

1. 如果系统要求提供凭据，请登录电子邮件帐户，这样逻辑应用就可以创建到该电子邮件帐户的连接了。

1. 在“发送电子邮件”操作中，指定需要电子邮件包括的数据。

   1. 在“收件人”框中，输入收件人的电子邮件地址。 
   为进行测试，可以使用自己的电子邮件地址。

      现在，请忽略出现的“添加动态内容”列表。 
      单击某些编辑框时，此列表会出现并显示前一步骤中提供的参数，这样即可将其作为工作流的输入包括进去。 

   1. 在“主题”框中，输入带有尾随空格的以下文本：```New RSS item:```

      ![输入电子邮件主题](./media/quickstart-create-first-logic-app-workflow/add-action-send-email-subject.png)

   1. 从“添加动态内容”列表中选择“源标题”，用于包括 RSS 项标题。

      ![动态内容列表 -“源标题”](./media/quickstart-create-first-logic-app-workflow/add-action-send-email-subject-dynamic-content.png)

      完成后，电子邮件主题如以下示例所示：

      ![添加的源标题](./media/quickstart-create-first-logic-app-workflow/add-action-send-email-feed-title.png)

      如果“For each”循环显示在设计器上，则你为数组选择了一个令牌，例如 **categories-Item** 令牌。 
      对于这些类型的令牌，设计器会自动围绕引用该令牌的操作添加此循环。 
      这样一来，逻辑应用会对每个数组项执行同一操作。 
      若要删除循环，请选择循环的标题栏上的省略号 (**...**)，然后选择“删除”。

   1. 在“正文”框中选择此文本，然后选择这些令牌作为电子邮件正文。 
   若要在编辑框中添加空行，请按 Shift + Enter。 

      ![添加电子邮件正文内容](./media/quickstart-create-first-logic-app-workflow/add-action-send-email-body.png)

      | 属性 | 说明 | 
      |----------|-------------| 
      | 源标题 | 项的标题 | 
      | 源发布日期 | 项的发布日期和时间 | 
      | 源主链接 | 项的 URL | 
      |||

1. 保存逻辑应用。

接下来，测试逻辑应用。

## <a name="run-your-logic-app"></a>运行逻辑应用

若要手动启动逻辑应用，可在设计器工具栏中选择“运行”。 或者，等待逻辑应用根据指定的计划（每隔一分钟）检索 RSS 源。 如果 RSS 源有新项，逻辑应用会为每个新项发送一封电子邮件。 否则，逻辑应用会等到下一个间隔过后才进行检查。 

例如，下面是此逻辑应用发送的一封示例电子邮件。 如果没有收到任何电子邮件，请检查垃圾邮件文件夹。

![针对新的 RSS 源项发送的电子邮件](./media/quickstart-create-first-logic-app-workflow/monitor-rss-feed-email.png)

从技术上讲，当触发器检查 RSS 源并找到新项时，触发器将会激发，并且逻辑应用引擎会创建一个逻辑应用工作流实例，用于运行工作流中的操作。
如果触发器未找到新项，则触发器不会激发，而是“跳过”实例化工作流的操作。

祝贺你，现已使用 Azure 门户成功生成并运行了第一个逻辑应用！

## <a name="clean-up-resources"></a>清理资源

不再需要此示例时，删除包含你的逻辑应用和相关资源的资源组。 

1. 在 Azure 主菜单中，转到“资源组”，然后选择逻辑应用的资源组。 在“概述”页面上，选择“删除资源组”。 

   ![“资源组”>“概览”>“删除资源组”](./media/quickstart-create-first-logic-app-workflow/delete-resource-group.png)

1. 输入资源组名称作为确认，然后选择“删除”。

   ![确认删除](./media/quickstart-create-first-logic-app-workflow/delete-resource-group-2.png)

> [!NOTE]
> 删除逻辑应用后，任何新运行都不会实例化。 所有正在进行和挂起的运行都将取消。 如果有成千上万个运行，取消操作可能需要很长时间才能完成。

## <a name="get-support"></a>获取支持

有关问题，请访问 [Azure 逻辑应用论坛](https://social.msdn.microsoft.com/Forums/home?forum=azurelogicapps)。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何创建第一个逻辑应用，以便按指定计划检查 RSS 更新（每分钟检查一次），当存在更新时执行操作（发送电子邮件）。 若要了解详细信息，请继续学习本教程，了解如何创建更高级的基于计划的工作流：

> [!div class="nextstepaction"]
> [使用基于计划的逻辑应用检查流量](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)
