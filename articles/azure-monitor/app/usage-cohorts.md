---
title: 在 Azure Application Insights 中使用队列 | Microsoft Docs
description: 分析具有某种共性的不同集、用户、会话、事件或操作
services: application-insights
documentationcenter: ''
author: NumberByColors
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/10/2018
ms.reviewer: mbullwin
ms.pm_owner: daviste;NumberByColors
ms.author: daviste
ms.openlocfilehash: 810ccae08b540c417a0e7dfecb6ad5a8b110cd7b
ms.sourcegitcommit: 8330a262abaddaafd4acb04016b68486fba5835b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/04/2019
ms.locfileid: "54045042"
---
# <a name="application-insights-cohorts"></a>Application Insights 队列

队列是具有某种共性的用户、会话、事件或操作集。 在 Azure Application Insights 中，队列由 Analytics 查询定义。 如果你要反复分析特定的用户或事件集，队列可让你更灵活地准确表达所需的集。

![队列窗格](./media/usage-cohorts/001.png)

## <a name="cohorts-versus-basic-filters"></a>队列与基本筛选器

队列的使用方式与筛选器类似。 但是，队列的定义是从自定义分析查询生成的，因此它们的自适应性和复杂性要高得多。 与筛选器不同，队列可以保存，因此可供其他团队成员重复使用。

对于尝试了应用中新功能的所有用户，可以定义一个队列。 可将此队列保存在 Application Insights 资源中。 今后可以轻松分析这组保存的特定用户。

> [!NOTE]
> 创建队列后，可以通过“用户”、“会话”、“事件”和“用户流”工具访问队列。

## <a name="example-engaged-users"></a>示例：参与用户

团队将参与用户定义为在给定月份使用你的应用达到五次或五次以上的任何人。 本部分为这些参与用户定义一个队列。

1. 打开“队列”工具。

2. 选择“模板库”选项卡。将会看到各种队列的模板集合。

3. 选择“参与用户 -- 按使用天数”。

    此队列有三个参数：
    * **Activities**：选择要将哪些事件和页面视图计入“用法”。
    * **Period**：月份的定义。
    * **UsedAtLeastCustom**：用户需要在某个周期内使用某个对象多少次，才能将该用户算作参与用户。

4. 请将 **UsedAtLeastCustom** 更改为 **5 天以上**，并将 **Period** 保留为默认 28 天。

    ![参与用户](./media/usage-cohorts/003.png)

    现在，此队列代表在过去 28 天内 5 个独立的日期通过任何自定义事件或页面视图发送的所有用户 ID。

5. 选择“保存”。

   > [!TIP]
   > 为队列命名，例如“Engaged Users (5+ Days)”。 根据是否希望对此 Application Insights 资源拥有访问权限的其他人查看此队列，将此队列保存到“我的报告”或“共享报告”中。

6. 选择“返回库”。

### <a name="what-can-you-do-by-using-this-cohort"></a>此队列有哪些作用？

打开“用户”工具。 在“显示”下拉框中，选择在“用户属于...”下创建的队列。

现已根据此用户队列筛选“用户”工具：

![根据特定的队列筛选“用户”窗格](./media/usage-cohorts/004.png)

需要注意几个要点：

* 不能通过普通的筛选器创建此集。 日期逻辑更高级。
* 可以使用“用户”工具中的正常筛选器进一步筛选此队列。 因此，尽管该队列是基于 28 天期限定义的，但仍可以在“用户”工具中将时间范围调整为 30、60 或 90 天。

这些筛选器支持更复杂的问题，而通过查询生成器无法表达这样的问题。 例如，对于在过去 28 天参与的人员，他们在过去 60 天的行为如何？

## <a name="example-events-cohort"></a>示例：事件队列

还可以创建事件队列。 在本部分，我们将定义一个事件队列和页面视图， 然后了解如何从其他工具使用它们。 此队列可以定义一组被团队视为 _活动用法_ 的事件，或者定义一组与特定新功能相关的事件。

1. 打开“队列”工具。

2. 选择“模板库”选项卡。将会看到各种队列的模板集合。

3. 选择“事件选取器”。

    ![事件选取器的屏幕截图](./media/usage-cohorts/006.png)

4. 在“活动”下拉框中，选择想要添加到队列中的事件。

5. 保存队列并为其命名。

## <a name="example-active-users-where-you-modify-a-query"></a>示例：在其中修改查询的活动用户

前面两个队列是使用下拉框定义的。 但是，我们也可以使用 Analytics 查询来定义队列，以获得全面灵活性。 让我们探讨如何为英国的用户创建队列。

![演练“队列”工具用法的动画](./media/usage-cohorts/cohorts0001.gif)

1. 打开“队列”工具，依次选择“模板库”选项卡、“空白用户队列”。

    ![空白用户队列](./media/usage-cohorts/001.png)

    有三个部分：
    * 一个 Markdown 文本部分，可在其中为其他团队成员更详细地描述该队列。

    * 一个参数部分，可在其中创建自己的参数，例如 **Activities**，以及前面两个示例中的其他下拉框。

    * 一个查询部分，可在其中使用分析查询定义队列。

    在查询部分[编写一个分析查询](/azure/kusto/query)。 该查询选择描述所要定义的队列的行集。 然后，“队列”工具将在查询中隐式添加一个“| summarize by user_Id”子句。 表中查询的下面提供了此数据的预览，使你能够确保查询返回结果。

    > [!NOTE]
    > 如果看不到查询，请尝试调整部分的大小，以增加其高度并显示查询。 本部分开头的 .gif 动画演示了大小调整行为。

2. 将以下文本复制并粘贴到查询编辑器中：

    ```KQL
    union customEvents, pageViews
    | where client_CountryOrRegion == "United Kingdom"
    ```

3. 选择“运行查询”。 如果表中未显示用户 ID，请更改为应用程序用户所在的国家/地区。

4. 保存并命名队列。

## <a name="frequently-asked-questions"></a>常见问题

_我已定义某些国家/地区的用户的队列。当我在“用户”工具中创建此队列，与只是基于该国家/地区设置筛选器进行比较时，看到了不同的结果。为什么？_

队列与筛选器有所不同。 假设你为英国的用户创建了一个队列（类似于上述示例中的定义），并将结果与设置筛选器“国家或地区 = 英国”进行比较。

* 队列版本将显示在当前时间范围内从英国发送了一个或多个事件的用户的所有事件。 如果按国家或地区拆分，可能会看到许多国家和地区。
* 筛选器版本只显示来自英国的事件。 不过，如果按国家或地区拆分，则只会看到“英国”。

## <a name="learn-more"></a>了解详细信息

* [Analytics 查询语言](https://go.microsoft.com/fwlink/?linkid=856587)
* [用户、会话、事件](usage-segmentation.md)
* [用户流](usage-flows.md)
* [使用情况概述](usage-overview.md)
