---
title: 通过 SQL Server Management Studio 使用 Azure Blockchain Workbench 数据
description: 了解如何从 SQL Server Management Studio 中连接到 Azure Blockchain Workbench 的 SQL 数据库。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: db43dd04da601650b3592dc74cdd804fae394c00
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510914"
---
# <a name="using-azure-blockchain-workbench-data-with-sql-server-management-studio"></a>通过 SQL Server Management Studio 使用 Azure Blockchain Workbench 数据

利用 Microsoft SQL Server Management Studio，可以快速编写和测试针对 Azure Blockhain Workbench 的 SQL DB 的查询。 本部分包含有关如何从 SQL Server Management Studio 中连接到 Azure Blockchain Workbench 的 SQL 数据库的分步演练。

## <a name="prerequisites"></a>必备组件

* 下载 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017)。

## <a name="connecting-sql-server-management-studio-to-data-in-azure-blockchain-workbench"></a>将 SQL Server Management Studio 连接到 Azure Blockchain Workbench 中的数据

1. 打开 SQL Server Management Studio 并选择“连接”。
2. 选择“数据库引擎”。

    ![数据库引擎](./media/data-sql-management-studio/database-engine.png)

3. 在“连接到服务器”对话框中，输入服务器名称和数据库凭据。

    如果使用由 Azure Blockchain Workbench 部署过程创建的凭据，则用户名是 **dbadmin** 并且密码是在部署过程中提供的密码。

    ![输入 SQL 凭据](./media/data-sql-management-studio/sql-creds.png)

   1. SQL Server Management Studio 显示 Azure Blockchain Workbench 数据库中的数据库、数据库视图和存储过程的列表。

      ![数据库列表](./media/data-sql-management-studio/db-list.png)

5. 若要查看与任何数据库视图关联的数据，可以使用以下步骤自动生成 select 语句。
6. 右键单击对象资源管理器中的任何数据库视图。
7. 选择“编写视图脚本为”。
8. 选择“SELECT to”。
9. 选择“新建查询编辑器窗口”。
10. 可以通过选择“新建查询”来创建新查询。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的数据库视图](database-views.md)
