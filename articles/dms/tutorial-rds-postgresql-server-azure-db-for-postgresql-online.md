---
title: 教程：使用 Azure 数据库迁移服务将 RDS PostgreSQL 联机迁移到 Azure Database for PostgreSQL | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务执行从 RDS PostgreSQL 到 Azure Database for PostgreSQL 的联机迁移。
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
ms.openlocfilehash: 1ee4d546ce823f48a597331276813b42d91e01e4
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415985"
---
# <a name="tutorial-migrate-rds-postgresql-to-azure-database-for-postgresql-online-using-dms"></a>教程：使用 DMS 以将 RDS PostgreSQL 联机迁移到 Azure Database for PostgreSQL

可以使用 Azure 数据库迁移服务将 RDS PostgreSQL 实例中的数据库迁移到 [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/)，在迁移期间，源数据库可保持联机状态。 换而言之，实现这种迁移只会对应用程序造成极短暂的停机。 本教程介绍如何在 Azure 数据库迁移服务中使用联机迁移活动将 **DVD Rental** 示例数据库从 RDS PostgreSQL 9.6 实例迁移到 Azure Database for PostgreSQL。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
>
> * 使用 pg_dump 实用工具迁移示例架构。
> * 创建 Azure 数据库迁移服务的实例。
> * 使用 Azure 数据库迁移服务创建迁移项目。
> * 运行迁移。
> * 监视迁移。

> [!NOTE]
> 使用 Azure 数据库迁移服务执行联机迁移需要基于“高级”定价层创建实例。 有关详细信息，请参阅 Azure 数据库迁移服务[定价](https://azure.microsoft.com/pricing/details/database-migration/)页。

> [!IMPORTANT]
> 为获得最佳迁移体验，Microsoft 建议在目标数据库所在的 Azure 区域中创建 Azure 数据库迁移服务的实例。 跨区域或地理位置移动数据可能会减慢迁移过程并引入错误。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文介绍如何从 PostgreSQL 的本地实例联机迁移到 Azure Database for PostgreSQL。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

* 下载并安装 [PostgreSQL 社区版](https://www.postgresql.org/download/) 9.5、9.6 或 10。 源 PostgreSQL 服务器版本必须是 9.5.11、9.6.7、10 或更高版本。 有关详细信息，请参阅[支持的 PostgreSQL 数据库版本](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions)一文。

    另外，RDS PostgreSQL 版本必须与 Azure Database for PostgreSQL 版本相符。 例如，RDS PostgreSQL 9.5.11.5 只能迁移到 Azure Database for PostgreSQL 9.5.11，不能迁移到 9.6.7 版本。

    > [!NOTE]
    > 对于 PostgreSQL 版本 10，DMS 目前仅支持将版本 10.3 迁移到 Azure Database for PostgreSQL。

* 创建 [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal) 的实例。 有关如何使用 pgAdmin 连接到 PostgreSQL 服务器的详细信息，请参阅此文档[部分](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal#connect-to-the-postgresql-server-using-pgadmin)。
* 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。
* 请确保 VNet 网络安全组规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)一文。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 PostgreSQL 服务器（默认情况下为 TCP 端口 5432）。
* 在源数据库的前面使用了防火墙设备时，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库。
* 为 Azure Database for PostgreSQL 服务器创建服务器级[防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNet 子网范围。

### <a name="set-up-aws-rds-postgresql-for-replication"></a>设置 AWS RDS PostgreSQL 用于复制

1. 若要创建新的参数组，请遵照 AWS 在[使用 DB 参数组](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html)中提供的说明操作。
2. 在 Azure 数据库迁移服务中使用主用户名连接到源。 如果使用的帐户不是主用户帐户，该帐户必须具有 rds_superuser 角色和 rds_replication 角色。 rds_replication 角色可以授予管理逻辑槽以及使用逻辑槽流式传输数据的权限。
3. 使用以下配置创建新的参数组：a. 将 DB 参数组中的 rds.logical_replication 参数设置为 1。
    b. max_wal_senders =[并发任务数] - max_wal_senders 参数设置可以运行的并发任务数，建议设置为 10 个任务。
    c. max_replication_slots = [槽数]，建议设置为 5 个槽。
4. 将创建的参数组关联到 RDS PostgreSQL 实例。

## <a name="migrate-the-schema"></a>迁移架构

1. 从源数据库提取架构并将其应用到目标数据库，以完成所有数据库对象（例如表架构、索引和存储过程）的迁移。

    仅迁移架构的最简单方法是结合 -s 选项使用 pg_dump。 有关详细信息，请参阅 Postgres pg_dump 教程中的[示例](https://www.postgresql.org/docs/9.6/app-pgdump.html#PG-DUMP-EXAMPLES)。

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    例如，若要转储 **dvdrental** 数据库的架构文件，请使用以下命令：

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

2. 在目标服务（即 Azure Database for PostgreSQL）中创建一个空数据库。 若要连接和创建数据库，请参阅以下文章之一：

    * [使用 Azure 门户创建 Azure Database for PostgreSQL 服务器](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)
    * [使用 Azure CLI 创建用于 PostgreSQL 的 Azure 数据库](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-azure-cli)

3. 将架构导入目标服务（即 Azure Database for PostgreSQL）。 若要还原架构转储文件，请运行以下命令：

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    例如：

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. 如果架构中有外键，则迁移的初始加载和连续同步会失败。 若要在目标 (Azure Database for PostgreSQL) 中提取 drop foreign key 脚本和 add foreign key 脚本，请在 PgAdmin 或 psql 中运行以下脚本：

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
      AND KCU.REFERENCED_TABLE_SCHEMA = 'sakila') Queries
      GROUP BY SchemaName;
    ```

5. 运行查询结果中的 drop foreign key（第二列），以删除外键。

6. 如果数据中包含触发器（insert 或 update 触发器），该触发器会在从源复制数据之前在目标中强制实施数据完整性。 建议在迁移期间禁用目标的所有表中的触发器，然后在迁移完成后再启用这些触发器。

    在目标数据库中禁用触发器：

    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

   ![显示门户订阅](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-subscription1.png)

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>创建 Azure 数据库迁移服务的实例

1. 在 Azure 门户中，选择 **+ 创建资源**，搜索 Azure 数据库迁移服务，然后从下拉列表选择**Azure 数据库迁移服务**。

    ![Azure 市场](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择要在其中创建 Azure 数据库迁移服务实例的位置。

5. 选择现有的 VNet，或新建一个 VNet。

    VNet 为 Azure 数据库迁移服务提供源 PostgreSQL 实例和目标 Azure Database for PostgreSQL 实例的访问权限。

    若要详细了解如何在 Azure 门户中创建 VNet，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

6. 选择定价层；对于此联机迁移，请务必选择“高级:4vCores”定价层。

    ![配置 Azure 数据库迁移服务实例设置](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-settings4.png)

7. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

      ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索你创建的 Azure 数据库迁移服务实例名称，然后选择该实例。

     ![查找 Azure 数据库迁移服务实例](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-instance-search.png)

3. 选择“+ 新建迁移项目”。
4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“AWS RDS for PostgreSQL”，在“目标服务器类型”文本框中选择“Azure Database for PostgreSQL”。
5. 在“选择活动类型”部分选择“联机数据迁移”。

    > [!IMPORTANT]
    > 请确保选择“联机数据迁移”；此方案不支持脱机迁移。

    ![创建数据库迁移服务项目](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-create-project5.png)

    > [!NOTE]
    > 也可以现在就选择“仅创建项目”来创建迁移项目，在以后再执行迁移。

6. 选择“保存”。

7. 选择“创建并运行活动”，以便创建项目并运行迁移活动。

    > [!NOTE]
    > 请在项目创建边栏选项卡中记下设置联机迁移所要满足的先决条件。

## <a name="specify-source-details"></a>指定源详细信息

* 在“迁移源详细信息”屏幕上，指定源 PostgreSQL 实例的连接详细信息。

   ![源详细信息](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-source-details4.png)

## <a name="specify-target-details"></a>指定目标详细信息

1. 选择“保存”，然后在“目标详细信息”屏幕上指定目标 Azure Database for PostgreSQL 服务器的连接详细信息，该服务器是提前预配的，具有使用 pg_dump 部署的 **DVD Rentals** 架构。

    ![选择目标](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-select-target4.png)

2. 选择“保存”，然后在“映射到目标数据库”屏幕上，映射源和目标数据库以进行迁移。

    如果目标数据库包含的数据库名称与源数据库的相同，则 Azure 数据库迁移服务默认会选择目标数据库。

    ![映射到目标数据库](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-map-targets-activity5.png)

3. 选择“保存”，在“迁移摘要”屏幕上的“活动名称”文本框中指定迁移活动的名称，然后查看摘要，确保源和目标详细信息与此前指定的信息相符。

    ![迁移摘要](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>运行迁移

* 选择“运行迁移”。

    迁移活动窗口随即出现，活动的“状态”为“正在初始化”。

## <a name="monitor-the-migration"></a>监视迁移

1. 在迁移活动屏幕上选择“刷新”，以便更新显示，直到迁移的“状态”显示为“正在运行”。

    ![活动状态 - 正在运行](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-activity-status3.png)

2. 在“数据库名称”下选择特定数据库即可转到“完整数据加载”和“增量数据同步”操作的迁移状态。

    “完整数据加载”会显示初始加载迁移状态，而“增量数据同步”则会显示变更数据捕获 (CDC) 状态。

    ![库存屏幕 - 完整数据加载](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-full-load.png)

    ![库存屏幕 - 增量数据同步](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>执行迁移直接转换

完成初始的完整加载后，数据库会被标记为“准备好交接”。

1. 如果准备完成数据库迁移，请选择“启动直接转换”。

    ![开始交接](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-start-cutover.png)

2. 确保停止传入源数据库的所有事务；等到“挂起的更改”计数器显示 **0**。
3. 选择“确认”，然后选择“应用”。
4. 当数据库迁移状态显示“已完成”后，请将应用程序连接到新的目标 Azure Database for PostgreSQL 数据库。

将 PostgreSQL 的本地实例联机迁移到 Azure Database for PostgreSQL 的过程现已完成。

## <a name="next-steps"></a>后续步骤

* 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
* 有关 Azure Database for PostgreSQL 的信息，请参阅[什么是 Azure Database for PostgreSQL？](https://docs.microsoft.com/azure/postgresql/overview)一文。
* 如有其他问题，请向[咨询 Azure 数据库迁移](mailto:AskAzureDatabaseMigrations@service.microsoft.com)别名发送电子邮件。
