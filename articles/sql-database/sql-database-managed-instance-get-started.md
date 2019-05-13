---
title: Azure 门户：创建 SQL 数据库托管实例 | Microsoft Docs
description: 创建要访问的 SQL 数据库托管实例、网络环境和客户端 VM。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlrab
manager: craigg
ms.date: 05/07/2019
ms.openlocfilehash: 09ab4cd44515d90df0e36e2775f50ac33955fe75
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466259"
---
# <a name="quickstart-create-an-azure-sql-database-managed-instance"></a>快速入门：创建 Azure SQL 数据库托管实例

本快速入门逐步说明如何在 Azure 门户中创建 Azure SQL 数据库[托管实例](sql-database-managed-instance.md)。

> [!IMPORTANT]
> 有关限制，请参阅[支持的区域](sql-database-managed-instance-resource-limits.md#supported-regions)和[支持的订阅类型](sql-database-managed-instance-resource-limits.md#supported-subscription-types)。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)。 

## <a name="create-a-managed-instance"></a>创建托管实例

下列步骤展示了如何创建托管实例。

1. 在 Azure 门户的左上角，选择“创建资源”。
2. 找到“托管实例”，然后选择“Azure SQL 托管实例”。
3. 选择“创建”。

   ![创建托管实例](./media/sql-database-managed-instance-get-started/managed-instance-create.png)

4. 根据下表中的信息，在“SQL 托管实例”窗体中填充要求的信息。

   | 设置| 建议的值 | Description |
   | ------ | --------------- | ----------- |
   | **订阅** | 你的订阅。 | 你有权在其中创建新资源的订阅。 |
   |**托管实例名称**|任何有效的名称。|请参阅[命名规则和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)，了解什么是有效的名称。|
   |**托管实例管理员登录名**|任何有效的用户名。|请参阅[命名规则和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)，了解什么是有效的名称。 不要使用“serveradmin”，因为这是保留的服务器级角色。|
   |**密码**|任何有效的密码。|密码必须至少 16 个字符，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm)。|
   |**时区**|托管实例要观察的时区。|有关详细信息，请参阅[时区](sql-database-managed-instance-timezone.md)。|
   |**排序规则**|要为托管实例使用的排序规则。|如果从 SQL Server 迁移数据库，请使用 `SELECT SERVERPROPERTY(N'Collation')` 检查源排序规则并使用该值。 有关排序规则的信息，请参阅[设置或更改服务器排序规则](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-server-collation)。|
   |**位置**|要在其中创建托管实例的位置。|有关区域的信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。|
   |**虚拟网络**|选择“创建新的虚拟网络”或有效的虚拟网络和子网。| 如果网络或子网不可用，则必须先[对其进行修改以满足网络要求](sql-database-managed-instance-configure-vnet-subnet.md)，然后再选择它作为新托管实例的目标。 有关为托管实例配置网络环境所要满足的要求的信息，请参阅[为托管实例配置虚拟网络](sql-database-managed-instance-connectivity-architecture.md)。 |
   |**启用公共终结点**   |选中此选项可启用公共终结点   |若要通过公共数据终结点访问托管实例，需要选中“启用公共终结点”。| 
   |**允许的访问来源**   |选择一个选项： <ul> <li>**Azure 服务**</li> <li>**Internet**</li> <li>**不允许访问**</li></ul>   |在门户体验中可以使用公共终结点配置安全组。 </br> </br> 根据具体的方案选择以下选项之一： </br> <ul> <li>Azure 服务 - 从 Power BI 或其他多租户服务连接时，建议使用此选项。 </li> <li> Internet - 用于测试，可让你快速运转托管实例。 不建议在生产环境中使用。 </li> <li> 不允许访问 - 此选项会创建一个“拒绝”安全规则。 需要修改此规则才能通过公共终结点访问托管实例。 </li> </ul> </br> 有关公共终结点安全性的详细信息，请参阅[通过公共终结点安全使用 Azure SQL 数据库托管实例](sql-database-managed-instance-public-endpoint-securely.md)。|
   |**连接类型**|选择“代理”或“重定向”连接类型。|有关连接类型的详细信息，请参阅 [Azure SQL 数据库连接策略](sql-database-connectivity-architecture.md#connection-policy)。|
   |**资源组**|新的或现有的资源组。|如需有效的资源组名称，请参阅 [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)（命名规则和限制）。|

   ![托管实例窗体](./media/sql-database-managed-instance-get-started/managed-instance-create-form.png)

5. 若要将托管实例用作实例故障转移组辅助实例，请选择“签出”并指定 DnsAzurePartner 托管实例。 此功能以预览版提供，未在以下屏幕截图中显示。
6. 选择“定价层”，以设置计算和存储资源的大小并查看定价层选项。 默认值是具有 32 GB 内存和 16 个 vCore 的“常规用途”定价层。
7. 使用滑块或文本框指定存储量和虚拟核心数。
8. 完成后，选择“应用”以保存所做的选择。 
9. 选择“创建”，部署托管实例。
10. 选择“通知”图标以查看部署状态。

    ![托管实例部署进度](./media/sql-database-managed-instance-get-started/deployment-progress.png)

11. 选择“正在进行的部署”以打开“托管实例”窗口，进一步监视部署进度。 

> [!IMPORTANT]
> 子网中第一个实例的部署时间通常远超后续实例的部署时间。 请勿因持续时间超出预期而取消部署操作。 在子网中创建第二个托管实例仅需要花费数分钟时间。

## <a name="review-resources-and-retrieve-your-fully-qualified-server-name"></a>查看资源并检索你的完全限定的服务器名称

部署成功完成后，请查看创建的资源，并检索完全限定的服务器名称以在后续快速入门中使用。

1. 打开托管实例的资源组。 查看在[创建托管实例](#create-a-managed-instance)快速入门中为该资源组创建的资源。

   ![托管实例资源](./media/sql-database-managed-instance-get-started/resources.png)

2. 选择路由表以查看创建的用户定义的路由 (UDR) 表。

   ![路由表](./media/sql-database-managed-instance-get-started/route-table.png)

3. 在路由表中，查看用于从托管实例虚拟网络或在其中路由流量的条目。 若要手动创建或配置路由表，必须确保在路由表中创建这些条目。

   ![本地托管实例子网的条目](./media/sql-database-managed-instance-get-started/udr.png)

4. 返回资源组并选择网络安全组，查看安全规则。

   ![网络安全组](./media/sql-database-managed-instance-get-started/network-security-group.png)

5. 查看入站和出站安全规则。 如果已经为托管实例配置了公共终结点，请参阅[配置公共终结点](sql-database-managed-instance-public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group)一文了解详细信息。

   ![安全规则](./media/sql-database-managed-instance-get-started/security-rules.png)

6. 返回资源组并选择你的托管实例。

   ![托管实例](./media/sql-database-managed-instance-get-started/managed-instance.png)

7. 在“概述”选项卡上，找到“主机”属性。 复制托管实例的完全限定主机地址，以在下一个快速入门中使用。

   ![主机名](./media/sql-database-managed-instance-get-started/host-name.png)

   名称类似于 **your_machine_name.a1b2c3d4e5f6.database.windows.net**。

## <a name="next-steps"></a>后续步骤

- 若要了解如何连接到托管实例：
  - 有关应用程序的连接选项概述，请参阅[将应用程序连接到托管实例](sql-database-managed-instance-connect-app.md)。
  - 有关介绍如何从 Azure 虚拟机连接到托管实例的快速入门，请参阅[配置 Azure 虚拟机连接](sql-database-managed-instance-configure-vm.md)。
  - 有关介绍如何使用点到站点连接从本地客户端计算机连接到托管实例的快速入门，请参阅[配置点到站点连接](sql-database-managed-instance-configure-p2s.md)。
- 若要将现有 SQL Server 数据库从本地还原到托管实例： 
    - 使用 [Azure 数据库迁移服务 (DMS)](../dms/tutorial-sql-server-to-managed-instance.md) 从数据库备份文件还原。 
    - 使用 [T-SQL RESTORE 命令](sql-database-managed-instance-get-started-restore.md) 从数据库备份文件还原。
- 有关使用内置故障排除智能对托管实例数据库性能进行的高级监视，请参阅[使用 Azure SQL Analytics 监视 Azure SQL 数据库](../azure-monitor/insights/azure-sql.md)。
