---
title: 教程：使用 Azure 数据库迁移服务将 MySQL 联机迁移到 Azure Database for MySQL | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务执行从本地 MySQL 到 Azure Database for MySQL 的联机迁移。
services: dms
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 05/08/2019
ms.openlocfilehash: 409fa3a501b80e225fff299980f374f73f0dbf13
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415732"
---
# <a name="tutorial-migrate-mysql-to-azure-database-for-mysql-online-using-dms"></a>教程：使用 DMS 以联机方式将 MySQL 迁移到 Azure Database for MySQL

可以使用 Azure 数据库迁移服务在尽量缩短停机时间的情况下，将数据库从本地 MySQL 实例迁移到 [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/)。 换句话说，可以在尽量减少应用程序故障时间的情况下进行迁移。 本教程介绍如何在 Azure 数据库迁移服务中使用联机迁移活动将 **Employees** 示例数据库从 MySQL 5.7 的本地实例迁移到 Azure Database for MySQL。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 使用 mysqldump 实用程序迁移示例架构。
> * 创建 Azure 数据库迁移服务的实例。
> * 使用 Azure 数据库迁移服务创建迁移项目。
> * 运行迁移。
> * 监视迁移。

> [!NOTE]
> 使用 Azure 数据库迁移服务执行联机迁移需要基于“高级”定价层创建实例。

> [!IMPORTANT]
> 为获得最佳迁移体验，Microsoft 建议在目标数据库所在的 Azure 区域中创建 Azure 数据库迁移服务的实例。 跨区域或地理位置移动数据可能会减慢迁移过程并引入错误。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

* 下载并安装 [MySQL 社区版](https://dev.mysql.com/downloads/mysql/) 5.6 或 5.7。 本地 MySQL 版本必须与 Azure Database for MySQL 版本相符。 例如，MySQL 5.6 只能迁移到 Azure Database for MySQL 5.6，不能升级到 5.7。
* [在 Azure Database for MySQL 中创建一个实例](https://docs.microsoft.com/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal)。 有关如何使用 Azure 门户连接和创建数据库的详细信息，请参阅[使用 MySQL Workbench 进行连接并查询数据](https://docs.microsoft.com/azure/mysql/connect-workbench)一文。  
* 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

    > [!NOTE]
    > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
    > * 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
    > * 存储终结点
    > * 服务总线终结点
    >
    > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。

* 请确保 VNet 网络安全组规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)一文。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 MySQL Server（默认情况下为 TCP 端口 3306）。
* 在源数据库的前面使用了防火墙设备时，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库。
* 为 Azure Database for MySQL 创建服务器级[防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNet 子网范围。
* 源 MySQL 必须使用支持的 MySQL 社区版。 若要确定 MySQL 实例的版本，请在 MySQL 实用程序或 MySQL Workbench 中运行以下命令：

    ```
    SELECT @@version;
    ```

* Azure Database for MySQL 仅支持 InnoDB 表。 若要将 MyISAM 表转换为 InnoDB，请参阅 [Converting Tables from MyISAM to InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html)（将表从 MyISAM 转换为 InnoDB）一文

* 通过以下配置在源数据库的 my.ini (Windows) 或 my.cnf (Unix) 文件中启用二进制日志记录。

    * **server_id** = 1 或更高版本（仅适用于 MySQL 5.6）
    * log-bin =\<path>（仅与 MySQL 5.6 相关）

        例如：log-bin = E:\MySQL_logs\BinLog
    * **binlog_format** = row
    * **Expire_logs_days** = 5（建议不要使用零；仅适用于 MySQL 5.6）
    * **Binlog_row_image** = full（仅适用于 MySQL 5.6）
    * **log_slave_updates** = 1

* 用户必须具有 ReplicationAdmin 角色的以下权限：

    * **复制客户端** - 仅仅是“更改处理”任务所需的。 换句话说，“仅完整加载”任务不需要此权限。
    * **复制副本** - 仅仅是“更改处理”任务所需的。 换句话说，“仅完整加载”任务不需要此权限。
    * **超级** - 仅在 MySQL 5.6.6 之前的版本中需要。

## <a name="migrate-the-sample-schema"></a>迁移示例架构

若要完成所有数据库对象（例如表架构、索引和存储过程），需从源数据库提取架构并将其应用到此数据库。 若要提取架构，可以将 mysqldump 与 `--no-data` 参数配合使用。

假设本地系统中有 MySQL Employees 示例数据库，则使用 mysqldump 进行架构迁移时所需的命令如下：

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

例如：

```
mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
```

若要将架构导入到 Azure Database for MySQL 目标，请运行以下命令：

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

例如：

```
mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
 ```

如果架构中有外键，则迁移的初始加载和连续同步会失败。  请在 MySQL Workbench 中执行以下脚本，以便提取 DROP FOREIGN KEY 脚本和 ADD FOREIGN KEY 脚本。

```
SET group_concat_max_len = 8192;
    SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
    FROM
    (SELECT
    KCU.REFERENCED_TABLE_SCHEMA as SchemaName,
    KCU.TABLE_NAME,
    KCU.COLUMN_NAME,
    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
    FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
    WHERE
      KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
      AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
  AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries
  GROUP BY SchemaName;
 ```

运行查询结果中的 DROP FOREIGN KEY（第二列），以便删除外键。

> [!IMPORTANT]
> 如果使用备份导入数据，请在执行 mysqldump 时手动删除或使用 --skip-definer 命令删除 CREATE DEFINER 命令。 DEFINER 需要使用超级特权来创建，并在 Azure Database for MySQL 中受到限制。

如果数据中有触发器（插入或更新触发器），该触发器会赶在源中的已复制数据之前在目标中强制实施数据完整性。 建议在迁移期间禁用目标的所有表中的触发器，然后在迁移完成后再启用这些触发器。

若要禁用目标数据库中的触发器，请使用以下命令：

```
SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

   ![显示门户订阅](media/tutorial-mysql-to-azure-mysql-online/portal-select-subscriptions.png)

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-mysql-to-azure-mysql-online/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-mysql-to-azure-mysql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>创建 DMS 实例

1. 在 Azure 门户中，选择 **+ 创建资源**，搜索 Azure 数据库迁移服务，然后从下拉列表选择**Azure 数据库迁移服务**。

    ![Azure 市场](media/tutorial-mysql-to-azure-mysql-online/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-mysql-to-azure-mysql-online/dms-create1.png)
  
3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择现有的 VNet，或新建一个 VNet。

    VNet 为 Azure 数据库迁移服务提供了对源 SQL Server 和目标 Azure SQL 数据库实例的访问权限。

    若要详细了解如何在 Azure 门户中创建 VNet，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

5. 选择定价层。

    有关成本和定价层的详细信息，请参阅[价格页](https://aka.ms/dms-pricing)。

    ![配置 Azure 数据库迁移服务实例设置](media/tutorial-mysql-to-azure-mysql-online/dms-settings3.png)

6. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

      ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-mysql-to-azure-mysql-online/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索你创建的 Azure 数据库迁移服务实例名称，然后选择该实例。

     ![查找 Azure 数据库迁移服务实例](media/tutorial-mysql-to-azure-mysql-online/dms-instance-search.png)

3. 选择“+ 新建迁移项目”。
4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“MySQL”，在“目标服务器类型”文本框中选择“AzureDbForMySQL”。
5. 在“选择活动类型”部分选择“联机数据迁移”。

    ![创建数据库迁移服务项目](media/tutorial-mysql-to-azure-mysql-online/dms-create-project4.png)

    > [!NOTE]
    > 也可以现在就选择“仅创建项目”来创建迁移项目，在以后再执行迁移。

6. 选择“保存”，记下成功使用 DMS 迁移数据需要满足的要求，然后选择“创建和运行活动”。

## <a name="specify-source-details"></a>指定源详细信息

1. 在“添加源详细信息”屏幕上，指定源 MySQL 实例的连接详细信息。

    ![“添加源详细信息”屏幕](media/tutorial-mysql-to-azure-mysql-online/dms-add-source-details.png)

## <a name="specify-target-details"></a>指定目标详细信息

1. 选择“保存”，然后在“目标详细信息”屏幕中指定目标 Azure Database for MySQL 服务器的连接详细信息，这是使用 mysqldump 向其部署 Employees 架构的 Azure Database for MySQL 的提前预配实例。

    ![“目标详细信息”屏幕](media/tutorial-mysql-to-azure-mysql-online/dms-add-target-details.png)

2. 选择“保存”，然后在“映射到目标数据库”屏幕上，映射源和目标数据库以进行迁移。

    如果目标数据库包含的数据库名称与源数据库的相同，则 Azure 数据库迁移服务会默认选择目标数据库。

    ![映射到目标数据库](media/tutorial-mysql-to-azure-mysql-online/dms-map-target-details.png)

3. 选择“保存”，在“迁移摘要”屏幕上的“活动名称”文本框中指定迁移活动的名称，然后查看摘要，确保源和目标详细信息与此前指定的信息相符。

    ![迁移摘要](media/tutorial-mysql-to-azure-mysql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>运行迁移

* 选择“运行迁移”。

    迁移活动窗口随即出现，活动的“状态”为“正在初始化”。

## <a name="monitor-the-migration"></a>监视迁移

1. 在迁移活动屏幕上选择“刷新”，以便更新显示，直到迁移的“状态”显示为“完成”。

     ![活动状态 - 完成](media/tutorial-mysql-to-azure-mysql-online/dms-activity-completed.png)

2. 在“数据库名称”下选择特定数据库即可转到“完整数据加载”和“增量数据同步”操作的迁移状态。

    完整数据加载会显示初始加载迁移状态，而增量数据同步则会显示变更数据捕获 (CDC) 状态。

     ![活动状态 - 完整加载已完成](media/tutorial-mysql-to-azure-mysql-online/dms-activity-full-load-completed.png)

     ![活动状态 - 增量数据同步](media/tutorial-mysql-to-azure-mysql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>执行迁移直接转换

完成初始的完整加载以后，数据库会被标记为“直接转换可供执行”。

1. 如果准备完成数据库迁移，请选择“启动直接转换”。

    ![启动直接转换](media/tutorial-mysql-to-azure-mysql-online/dms-start-cutover.png)

2. 确保停止传入源数据库的所有事务；等到“挂起的更改”计数器显示 **0**。
3. 选择“确认”，然后选择“应用”。
4. 当数据库迁移状态显示“已完成”后，请将应用程序连接到新的目标 Azure SQL 数据库。

## <a name="next-steps"></a>后续步骤

* 若要了解联机迁移到 Azure Database for MySQL 时的已知问题和限制，请参阅 [Azure Database for MySQL 联机迁移的已知问题和解决方法](known-issues-azure-mysql-online.md)一文。
* 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
* 若要了解 Azure Database for MySQL，请参阅[什么是 Azure Database for MySQL？](https://docs.microsoft.com/azure/mysql/overview)一文。
