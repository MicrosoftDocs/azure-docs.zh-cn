---
title: 在 Microsoft Power BI 中使用 Azure Blockchain Workbench 数据
description: 了解如何在 Microsoft Power BI 中加载和查看 Azure Blockchain Workbench SQL DB 数据。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: mmercuri
manager: femila
ms.openlocfilehash: b1020389ef28c18c03536d686cd47ef0c65b9204
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240417"
---
# <a name="using-azure-blockchain-workbench-data-with-microsoft-power-bi"></a>通过 Microsoft Power BI 使用 Azure Blockchain Workbench 数据

利用 Microsoft Power BI，可以轻松使用 Power BI Desktop 从 SQL DB 数据库生成强大的报表并将其发布到 [https://www.powerbi.com](http://www.powerbi.com)。

本文包含了有关以下操作的分步演练：从 PowerBI Desktop 中连接到 Azure Blockchain Workbench 的 SQL 数据库，创建报表，以及将报表部署到 powerbi.com。

## <a name="prerequisites"></a>先决条件

* 下载 [PowerBI Desktop](https://aka.ms/pbidesktopstore)。

## <a name="connecting-powerbi-to-data-in-azure-blockchain-workbench"></a>将 PowerBI 连接到 Azure Blockchain Workbench 中的数据

1.  打开 Power BI Desktop。
2.  选择“获取数据”。

    ![获取数据](./media/data-powerbi/get-data.png)
3.  从数据源类型列表中选择“SQL Server”。

4.  在对话框中提供服务器和数据库名称。 指定是要导入数据还是要执行 **DirectQuery**。 选择“确定”。

    ![选择 SQL Server](./media/data-powerbi/select-sql.png)

5.  提供数据库凭据来访问 Azure Blockchain Workbench。 选择“数据库”并输入凭据。

    如果使用由 Azure Blockchain Workbench 部署过程创建的凭据，则用户名是 **dbadmin** 并且密码是在部署过程中提供的密码。

    ![SQL DB 设置](./media/data-powerbi/db-settings.png)

6.  在连接到数据库后，“导航器”对话框将显示数据库中可用的表和视图。 视图设计用来提供报告并且全部以 **vw** 作为前缀。

    ![导航器](./media/data-powerbi/navigator.png)

7.  选择要包括的视图。 对于演示用途，我们包括了 **vwContractAction**，该视图提供了已对某个合约执行的所有操作的详细信息。

    ![选择视图](./media/data-powerbi/select-views.png)

现在可以像往常那样使用 Power BI 创建和发布报表了。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench 中的数据库视图](database-views.md)