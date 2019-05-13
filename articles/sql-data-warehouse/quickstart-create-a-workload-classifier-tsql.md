---
title: 快速入门：创建工作负荷分类器 - T-SQL | Microsoft Docs
description: 使用 T-SQL 创建具有较高重要性的工作负荷分类器
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: quickstart
ms.subservice: workload management
ms.date: 05/01/2019
ms.author: rortloff
ms.reviewer: jrasnick
ms.openlocfilehash: 2074ea1028f6a5e8a0251c6d0c9519361db6ba8d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150385"
---
# <a name="quickstart-create-a-workload-classifier-using-t-sql"></a>快速入门：使用 T-SQL 创建工作负荷分类器

在本快速入门中，我们将为组织的 CEO 快速创建一个具有较高重要性的工作负荷分类器。 此工作负荷分类器允许 CEO 查询的优先级高于队列中重要性更低的其他查询。

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

> [!NOTE]
> 创建 SQL 数据仓库可能会导致新的计费服务。  有关详细信息，请参阅 [Azure SQL 数据仓库定价](https://azure.microsoft.com/pricing/details/sql-data-warehouse/)。
>
>

## <a name="prerequisites"></a>先决条件

本快速入门假设读者已有一个 SQL 数据仓库，并且拥有“控制数据库”权限。 如果需要创建一个 SQL 数据仓库，可使用[创建并连接 - 门户](create-data-warehouse-portal.md)创建名为“mySampleDataWarehouse”的数据仓库。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)。

## <a name="create-login-for-theceo"></a>为 TheCEO 创建登录名

使用 [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql) 在 `master` 数据库中为“TheCEO”创建 SQL Server 身份验证登录名。

```sql
IF NOT EXISTS (SELECT * FROM sys.sql_logins WHERE name = 'TheCEO')
BEGIN
CREATE LOGIN [TheCEO] WITH PASSWORD='<strongpassword>'
END
;
```

## <a name="create-user"></a>创建用户

在 mySampleDataWarehouse 中[创建用户](/sql/t-sql/statements/create-user-transact-sql?view=azure-sqldw-latest)“TheCEO”

```sql
IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = 'THECEO')
BEGIN
CREATE USER [TheCEO] FOR LOGIN [TheCEO]
END
;
```

## <a name="create-a-workload-classifier"></a>创建工作负荷分类器

为“TheCEO”创建具有较高重要性的[工作负荷分类器](/sql/t-sql/statements/create-workload-classifier-transact-sql?view=azure-sqldw-latest)。

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO];
CREATE WORKLOAD CLASSIFIER [wgcTheCEO]
WITH (WORKLOAD_GROUP = 'xlargerc'
      ,MEMBERNAME = 'TheCEO'
      ,IMPORTANCE = HIGH);
```

## <a name="view-existing-classifiers"></a>查看现有分类器

```sql
SELECT * FROM sys.workload_management_workload_classifiers
```

## <a name="clean-up-resources"></a>清理资源

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO]
DROP USER [TheCEO]
;
```

我们会针对数据仓库中存储的数据，按数据仓库单位收费。 这些计算和存储资源是分开计费的。

- 如果想要将数据保留在存储中，可以在不使用数据仓库时暂停计算。 如果暂停计算资源，则你只需支付数据存储费用。 准备好处理数据时，可以恢复计算。
- 如果不想支付将来的费用，则可以删除数据仓库。

遵循以下步骤清理资源。

1. 登录到 [Azure 门户](https://portal.azure.com)，选择你的数据仓库。

    ![清理资源](media/load-data-from-azure-blob-storage-using-polybase/clean-up-resources.png)

2. 若要暂停计算，请选择“暂停”按钮。 暂停数据仓库后，可看到“启动”按钮。  若要恢复计算，请选择“启动”。

3. 若要删除数据仓库以免产生计算或存储费用，请选择“删除”。

4. 要删除所创建的 SQL 服务器，请选择上图所示的“mynewserver-20180430.database.windows.net”，然后选择“删除”。  请谨慎执行此删除操作，因为删除服务器的同时也会删除分配给该服务器的所有数据库。

5. 若要删除资源组，请选择“myResourceGroup”，然后选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

现已创建一个工作负荷分类器。 以 TheCEO 身份运行一些查询，以查看该分类器的执行情况。 参阅 [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql) 以查看查询和分配的重要性。

有关 SQL 数据仓库工作负荷管理的详细信息，请参阅 [SQL 数据仓库工作负荷重要性](sql-data-warehouse-workload-importance.md)和 [SQL 数据仓库工作负荷分类](sql-data-warehouse-workload-classification.md)。
