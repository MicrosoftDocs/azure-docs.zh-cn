---
title: Azure Sentinel Preview 中的搜索功能 |Microsoft Docs
description: 本文介绍如何使用 Azure Sentinel 搜索功能。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 6aa9dd27-6506-49c5-8e97-cc1aebecee87
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2019
ms.author: rkarlin
ms.openlocfilehash: 754722508de248294667ee9e923e8faf3ad67f61
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236289"
---
# <a name="hunt-for-threats-with-in-azure-sentinel-preview"></a>寻找 Azure Sentinel 预览版中使用的威胁

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

如果您想要主动寻找安全威胁，Azure Sentinel 功能强大的搜索搜索和查询工具要在组织的数据源之间的安全威胁进行调查。 但是，你的系统和安全设备生成巨额的可能很难分析和筛选到事件中有意义的数据。 若要帮助安全分析师查找主动新异常未检测到的安全应用，Azure Sentinel 内置搜索查询将引导你到询问正确的问题，你的网络已有的数据中发现的问题。 

例如，一个内置的查询提供了有关您的基础结构上运行的最常见进程的数据-不希望运行的每个时间相关的警报，它们可能是完全无不妥，但你可能想要看一看查询后，有时，查看是否 there 的任何异常情况。 



通过 Azure Sentinel 搜索，你可以利用以下功能：

- 内置的查询：为了帮助您入门，起始页提供了启动的目标在于带您的预加载的查询示例和帮助你熟悉的表和查询语言。 由 Microsoft 安全研究人员在连续模式中，添加新查询，开发这些内置搜索查询并为您提供的入口点来查找新检测并找出要开始寻找微调现有查询刚开始，新的攻击。 

- 具有 IntelliSense 的功能强大的查询语言：基于可用于灵活地需要搜索下一级别的查询语言。

- 创建您自己的书签：在搜索过程中，您可能已经使用过的匹配项或发现，仪表板，或者查找异常或可疑的活动。 若要将这些项的标记，以便你可以取用它们在将来，使用书签功能。 书签，可以将项保存供稍后使用，用于创建调查的案例。 有关书签的详细信息，请参阅使用 [中搜索的书签]。

- 使用笔记本来自动执行调查：笔记本就像可以生成以逐步调查并查寻的分步操作手册。  笔记本封装在可重用的 playbook 可以与他人共享你的组织中的所有搜索步骤。 
- 查询存储的数据：若要查询的表中访问数据。 例如，您可以查询过程的创建、 DNS 事件和许多其他事件类型。

- 社区的链接：利用更高版本的社区的强大功能，若要查找其他查询和数据源。
 
## <a name="get-started-hunting"></a>开始猎

1. 在 Azure Sentinel 门户中，单击**搜索**。
  ![Azure Sentinel 开始搜索](media/tutorial-hunting/hunting-start.png)

2. 当打开**搜索**页面上，所有单个表中显示的搜索查询。 表列出了所有安全分析人员，以及创建或修改任何其他查询的 Microsoft 团队编写的查询。 每个查询提供了它，查找和其上运行的数据类型的说明。 这些模板按其各种策略-在右侧的图标进行分类的威胁，如初始访问、 持久性和渗透的类型。 您可以筛选这些搜索查询模板中使用的任何字段。 可以将任何查询保存到你的收藏夹。 通过将查询保存到收藏夹中，查询会自动在每次运行**搜索**访问的页面。 您可以创建自己的搜索查询或克隆和自定义现有的搜索查询模板。 
 
2. 单击**运行查询**中搜索查询的详细信息页，可以运行任何查询，而无需离开搜索页。  在表中显示的匹配数。 查看搜索查询和其匹配项的列表。 请查看匹配项，与杀伤链中的哪个阶段。

3. 在查询的详细信息窗格中执行的简要概述基础查询，或单击**查看查询结果**在 Log Analytics 中打开查询。 在底部，查看查询的匹配项。

4.  单击行，然后选择**添加书签**中添加行以进行调查的用户可以执行此操作看上去可疑的任何内容。 

5. 然后，返回到主**搜寻**页上，单击**书签**选项卡以查看所有可疑活动。 

6. 选择一个书签，然后单击**调查**打开调查体验。 您可以筛选书签。 例如，如果要研究市场活动，您可以创建营销活动的标记，并筛选根据市场活动的所有书签。

1. 发现的搜索查询提供了高价值的见解可能的攻击后，您还可以创建自定义检测规则基于您的查询，以警报形式的这些见解传达给你的安全事件响应程序。

 

## <a name="query-language"></a>查询语言 

在 Azure Sentinel 搜索基于 Azure Log Analytics 查询语言。 有关查询语言和支持的运算符的详细信息，请参阅[查询语言参考](https://docs.loganalytics.io/docs/Language-Reference/)。

## <a name="public-hunting-query-github-repository"></a>公共搜索查询 GitHub 存储库

请查看[搜索查询存储库](https://github.com/Azure/Orion)。 参与，并使用由我们的客户共享的示例查询。

 

## <a name="sample-query"></a>示例查询

典型查询以使用表名称后跟一系列运算符分隔\|。

在以下示例中，开始与表命名为安全事件并根据需要添加管道的元素。

1. 定义一个时间筛选器以查看仅从过去七天的记录。

2. 查询以仅显示事件 ID 4688 中添加筛选器。

3. 在命令行以包含仅的 cscript.exe 的实例上的查询中添加筛选器。

4. 项目仅列您感兴趣，并将结果限制为 1000年，然后单击**运行查询**。
5. 单击绿色三角形以运行查询。 你可以测试查询并运行它来寻找异常行为。

## <a name="useful-operators"></a>有用的运算符

查询语言功能非常强大和具有多个可用运算符，此处列出了一些有用的运算符：

**其中**-筛选到满足谓词的行子集的表。

**汇总**-生成聚合输入表中的内容的表。

**联接**-合并两个表以进行匹配的每个表中的指定列的值来形成一个新表中的行。

**计数**-输入记录集中返回的记录数。

**顶部**-按指定的列进行排序的返回前 n 个记录。

**限制**-返回到指定的行数。

**项目**-选择要包括、 重命名或删除的列，然后插入新的计算的列。

**扩展**-创建计算的列并将它们追加到结果集。

**makeset** -返回组中的一套 Expr 采用的非重复值的动态 (JSON) 数组

**查找**-查找与谓词匹配的一组表之间的行。

## <a name="save-a-query"></a>保存查询

可以创建或修改查询并将其保存为您自己的查询或与位于同一租户的用户共享。

   ![保存查询](./media/tutorial-hunting/save-query.png)

创建一个新的搜索查询：

1. 单击**新查询**，然后选择**保存**。
2. 填写所有空字段，然后选择**保存**。

   ![新建查询](./media/tutorial-hunting/new-query.png)

克隆并修改现有的搜索查询：

1. 选择你想要修改的表中的搜索查询。
2. 选择你想要修改，然后选择的查询的行中的省略号 （...）**克隆查询**。

   ![克隆查询](./media/tutorial-hunting/clone-query.png)
 

3. 修改查询，然后选择**创建**。

   ![自定义查询](./media/tutorial-hunting/custom-query.png)

## <a name="next-steps"></a>后续步骤
在本文中，您学习了如何使用 Azure Sentinel 运行搜索调查。 要详细了解 Azure Sentinel，请参阅以下文章：


- [使用笔记本来运行自动的搜索活动](notebooks.md)
- [使用书签来保存搜索时的相关信息](bookmarks.md)