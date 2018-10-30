---
title: 在 Microsoft Excel 中使用 Azure Blockchain Workbench 数据
description: 了解如何在 Microsoft Excel 中加载和查看 Azure Blockchain Workbench SQL DB 数据。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: 3c257a47c796636d0b86aa6b246b17c0fd39bae3
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240745"
---
# <a name="view-azure-blockchain-workbench-data-with-microsoft-excel"></a>使用 Microsoft Excel 查看 Azure Blockchain Workbench 数据

可以使用 Microsoft Excel 查看 Azure Blockchain Workbench SQL DB 中的数据。 本文提供了执行以下操作所需的步骤：

* 从 Microsoft Excel 连接到 Blockchain Workbench 数据库
* 查看 Blockchain Workbench 数据库表和视图
* 将 Blockchain Workbench 视图数据加载到 Excel 中

## <a name="connect-to-the-blockchain-workbench-database"></a>连接到 Blockchain Workbench 数据库

若要连接到 Blockchain Workbench 数据库，请执行以下操作：

1. 打开 Microsoft Excel。
2. 在“数据”选项卡上，选择“获取数据”。
3. 选择“从 Azure”，然后选择“从 Azure SQL 数据库”。

   ![连接到 Azure SQL 数据库](./media/data-excel/connect-sql-db.png)

4. 在“SQL Server 数据库”对话框中：

    * 对于“服务器”，输入 Blockchain Workbench 服务器的名称。
    * 对于“数据库(可选)”，输入数据库的名称。

   ![提供数据库服务器和数据库](./media/data-excel/provide-server-db.png)

5. 在“SQL Server 数据库”对话框导航栏中，选择“数据库”。 输入用户名和密码，然后选择“连接”。

    > [!NOTE]
    > 如果使用在 Azure Blockchain Workbench 部署过程中创建的凭据，则**用户名**为 `dbadmin`。 **密码**是在部署 Blockchain Workbench 时创建的密码。
    
   ![提供凭据来访问数据库](./media/data-excel/provide-credentials.png)

## <a name="look-at-database-tables-and-views"></a>查看数据库表和视图

连接到数据库后，“Excel 导航器”对话框将会打开。 可以使用导航器查看数据库中的表和视图。 视图设计用来提供报告，并且其名称以 **vw** 作为前缀。

   ![Excel 导航器中的视图预览](./media/data-excel/excel-navigator.png)

## <a name="load-view-data-into-an-excel-workbook"></a>将视图数据加载到 Excel 工作簿中

接下来的示例展示了如何将数据从视图加载到 Excel 工作簿中。

1. 在“导航器”滚动条中，选择“vwContractAction”视图。 “vwContractAction”预览显示与 Blockchain Workbench 数据库中的某个合约相关的所有操作。
2. 选择“加载”来检索该视图中的所有数据并将其置于 Excel 工作簿中。

   ![从视图加载的数据](./media/data-excel/view-data.png)

现在，你已加载了数据，可以使用 Excel 功能使用来自 Azure Blockchain Workbench 数据库的元数据和交易记录数据创建自己的报表了。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的数据库视图](database-views.md)