---
title: Azure SQL 数据库和 SQL 数据仓库数据发现和分类 | Microsoft Docs
description: Azure SQL 数据库数据发现和分类
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: vainolo
ms.author: arib
ms.reviewer: vanto
manager: craigg
ms.date: 03/22/2019
ms.openlocfilehash: e451b7837a1cff4bbeaecd1573dc860524caf4d3
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142659"
---
# <a name="azure-sql-database-and-sql-data-warehouse-data-discovery--classification"></a>Azure SQL 数据库和 SQL 数据仓库数据发现和分类

数据发现和分类（当前为预览版）提供了内置于 Azure SQL 数据库的高级功能，可用于发现、分类、标记和保护数据库中的敏感数据。 & 
发现最敏感的数据（业务、财务、医疗保健、个人身份数据 (PII)，等等）并进行分类可在组织的信息保护方面发挥关键作用。 它可以作为基础结构，用于：

- 帮助满足数据隐私标准和法规符合性要求。
- 各种安全方案，如监视（审核）并在敏感数据存在异常访问时发出警报。
- 控制对包含高度敏感数据的数据库的访问并强化其安全性。

数据发现和分类包含在[高级数据安全](sql-database-advanced-data-security.md) (ADS) 产品/服务中，该产品是高级 SQL 安全功能统一软件包。 可通过中心 SQL ADS 门户访问和管理数据发现和分类。

> [!NOTE]
> 本文档是关于 Azure SQL 数据库和 Azure SQL 数据仓库的。 为简单起见，在提到 SQL 数据库和 SQL 数据仓库时，本文统称 SQL 数据库。 对于 SQL Server（本地），请参阅 [SQL 数据发现和分类](https://go.microsoft.com/fwlink/?linkid=866999)。

## <a id="subheading-1"></a>什么是数据发现和分类

数据发现和分类引入了一套高级的服务和新的 SQL 功能，形成了新的 SQL 信息保护模式，旨在保护数据，而不仅仅是数据库：

- **发现和建议**

  分类引擎扫描数据库，并识别包含潜在敏感数据的列。 使用此功能可以通过 Azure 门户轻松地查看和应用适当的分类建议。

- **标记**

  使用 SQL 引擎中引入的新分类元数据属性，可在列上永久标记敏感度分类标签。 然后，此元数据可用于基于敏感度的高级审核和保护方案。

- **查询结果集敏感度**

  实时计算查询结果集的敏感度以供审核。

- **可见性**

  在门户中详细的仪表板中可以查看数据库分类状态。 此外，还可以下载用于符合性和审核目的以及其他需求的报表（Excel 格式）。

## <a id="subheading-2"></a>发现、分类和标记敏感列

以下部分介绍有关发现、分类和标记含数据库中的敏感数据的列的步骤，以及查看数据库当前分类状态和导出报告的步骤。

分类包含两个元数据属性：

- 标签 - 主要分类属性，用于定义列中存储数据的敏感度级别。  
- 信息类型 - 为列中存储数据的类型提供其他粒度。

## <a name="define-and-customize-your-classification-taxonomy"></a>定义和自定义分类

SQL 数据发现和分类附带了一组内置的敏感度标签和一组内置的信息类型和发现逻辑。 现在，可以自定义此分类并专门针对你的环境定义分类构造的集合和级别。

分类的定义和自定义是在一个中心位置针对你的整个 Azure 租户进行的。 该位置在 [Azure 安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)内，是你的安全策略的一部分。 只有对租户根管理组具有管理权限的人员可以执行此任务。

作为信息保护策略管理的一部分，你可以定义自定义标签，对其进行分级，并将其与选定的一组信息类型相关联。 你还可以添加自己的自定义信息类型，并为其配置字符串模式，这些模式将添加到发现逻辑以用于识别数据库中此类型的数据。
可以在[信息保护策略操作指南](https://go.microsoft.com/fwlink/?linkid=2009845&clcid=0x409)中详细了解如何自定义和管理策略。

在定义租户级策略后，可以继续使用自定义的策略对各个数据库进行分类。

## <a name="classify-your-sql-database"></a>对 SQL 数据库进行分类

1. 转到 [Azure 门户](https://portal.azure.com)。

2. 导航到 Azure SQL 数据库窗格“安全”标题下的“高级数据安全”。 单击以启用“高级数据安全”，然后单击“数据发现和分类(预览)”卡。

   ![扫描数据库](./media/sql-data-discovery-and-classification/data_classification.png)

3. “概述”选项卡包含数据库当前分类状态的摘要，其中包括所有已分类列的详细列表，你还可以筛选此列表，仅查看特定的架构部分、信息类型和标签。 如果尚未对任何列进行分类，请[跳到步骤 5](#step-5)。

   ![当前分类状态摘要](./media/sql-data-discovery-and-classification/2_data_classification_overview_dashboard.png)

4. 要下载 Excel 格式的报表，请单击窗口顶部菜单中的“导出”选项。

   ![导出到 Excel](./media/sql-data-discovery-and-classification/3_data_classification_export_report.png)

5. <a id="step-5"></a>要开始对数据进行分类，请单击窗口顶部的“分类”选项卡。

    ![对数据进行分类](./media/sql-data-discovery-and-classification/4_data_classification_classification_tab_click.png)

6. 分类引擎会扫描数据库，查找包含潜在敏感数据的列，并提供“建议列分类”列表。 查看并应用分类建议：

   - 要查看建议的列分类列表，请单击窗口底部的“建议”面板：

      ![对数据进行分类](./media/sql-data-discovery-and-classification/5_data_classification_recommendations_panel.png)

   - 查看建议列表 - 要接受特定列的建议，请选中相关行左侧列中的复选框。 还可以通过选中建议表格表头中的复选框，将所有建议标记为“已接受”。

       ![查看建议列表](./media/sql-data-discovery-and-classification/6_data_classification_recommendations_list.png)

   - 要应用所选建议，请单击蓝色的“接受所选建议”按钮。

      ![应用建议](./media/sql-data-discovery-and-classification/7_data_classification_accept_selected_recommendations.png)

7. 此处，还可选择手动分类列表，或者根据建议进行分类：

   - 单击窗口顶部菜单中的“添加分类”。

      ![手动添加分类](./media/sql-data-discovery-and-classification/8_data_classification_add_classification_button.png)

   - 在打开的“上下文”窗口中，选择“架构”>“表格”>“想要分类的列”，以及“信息类型”和“敏感度标签”。 然后单击“上下文”窗口底部的蓝色“添加分类”按钮。

      ![选择要进行分类的列](./media/sql-data-discovery-and-classification/9_data_classification_manual_classification.png)

8. 要完成分类并使用新的分类元数据永久标记数据库的列，请在窗口顶部菜单中单击“保存”。

   ![保存](./media/sql-data-discovery-and-classification/10_data_classification_save.png)

## <a id="subheading-3"></a>审核对敏感数据的访问

信息保护范例的一个重要方面是能够监视对敏感数据的访问。 [Azure SQL 数据库审核](sql-database-auditing.md) 已经过增强，在审核日志中包含了名为 data_sensitivity_information 的新字段，该字段会记录查询返回的实际数据的敏感度分类（标签）。

![审核日志](./media/sql-data-discovery-and-classification/11_data_classification_audit_log.png)

## <a id="subheading-4"></a>管理数据分类使用 T-SQL

可以使用 T-SQL 添加/删除列分类，以及检索整个数据库的所有分类。

> [!NOTE]
> 如果使用 T-SQL 管理标签，则不会验证组织信息保护策略中是否存在添加到列的标签（门户建议中显示的标签集）。 因此，是否要验证这一点完全由你决定。

- 添加/更新一列或多列分类：[添加敏感度分类](https://docs.microsoft.com/sql/t-sql/statements/add-sensitivity-classification-transact-sql)
- 删除一列或多列分类：[删除敏感度分类](https://docs.microsoft.com/sql/t-sql/statements/drop-sensitivity-classification-transact-sql)
- 查看数据库上的所有分类：[sys.sensitivity_classifications](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-sensitivity-classifications-transact-sql)

此外，还可以使用 REST API 通过编程方式管理分类。 已发布的 REST API 支持以下操作：

- [创建或更新](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/createorupdate) - 创建或更新给定列的敏感度标签
- [删除](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/delete) - 删除给定列的敏感度标签
- [获取](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/get) - 获取给定列的敏感度标签
- [按数据库列出当前项](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/listcurrentbydatabase) - 获取给定数据库的当前敏感度标签
- [按数据库列出建议项](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/listrecommendedbydatabase) - 获取给定数据库的建议敏感度标签

## <a name="manage-data-discovery-and-classification-using-azure-powershell"></a>管理数据发现和分类使用 Azure PowerShell

可以使用 PowerShell 获取 Azure SQL 数据库和托管的实例中的所有建议的列。

### <a name="powershell-cmdlets-for-azure-sql-database"></a>适用于 Azure SQL 数据库 PowerShell Cmdlet

- [Get-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasesensitivityclassification)
- [Set-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabasesensitivityclassification)
- [Remove-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqldatabasesensitivityclassification)
- [Get-AzSqlDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasesensitivityrecommendation)

### <a name="powershell-cmdlets-for-managed-instance"></a>托管实例的 PowerShell Cmdlet

- [Get-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityclassification)
- [Set-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstancedatabasesensitivityclassification)
- [Remove-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqlinstancedatabasesensitivityclassification)
- [Get-AzSqlInstanceDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityrecommendation)

## <a name="permissions"></a>权限

以下内置角色可以读取 Azure SQL 数据库的数据分类：`Owner`、`Reader`、`Contributor`、`SQL Security Manager`、`User Access Administrator`。

以下内置角色可以修改 Azure SQL 数据库的数据分类：`Owner`、`Contributor`、`SQL Security Manager`。

详细了解 [Azure 资源的 RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

## <a id="subheading-5"></a>后续步骤

- 详细了解[高级数据安全](sql-database-advanced-data-security.md)。
- 请考虑配置 [Azure SQL 数据库审核](sql-database-auditing.md) 来监视和审核对已分类敏感数据的访问。

<!--Anchors-->
[SQL data discovery & classification overview]: #subheading-1
[Discovering, classifying & labeling sensitive columns]: #subheading-2
[Auditing access to sensitive data]: #subheading-3
[Automated/Programmatic classification]: #subheading-4
[Next Steps]: #subheading-5
