---
title: 教程：使用 Azure 数据库迁移服务将 PostgreSQL 联机迁移到 Azure Database for PostgreSQL | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务执行从本地 PostgreSQL 到 Azure Database for PostgreSQL 的联机迁移。
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
ms.openlocfilehash: d7bd2555753df4c12404844c86be8f0339d88e23
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415689"
---
# <a name="tutorial-migrate-postgresql-to-azure-database-for-postgresql-online-using-dms"></a>教程：使用 DMS 以联机方式将 PostgreSQL 迁移到 Azure Database for PostgreSQL

可以使用 Azure 数据库迁移服务在尽量缩短停机时间的情况下，将数据库从本地 PostgreSQL 实例迁移到 [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/)。 换而言之，实现这种迁移只会对应用程序造成极短暂的停机。 本教程介绍如何在 Azure 数据库迁移服务中使用联机迁移活动将 **DVD Rental** 示例数据库从 PostgreSQL 9.6 的本地实例迁移到 Azure Database for PostgreSQL。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 使用 pg_dump 实用程序迁移示例架构。
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

* 下载并安装 [PostgreSQL 社区版](https://www.postgresql.org/download/) 9.5、9.6 或 10。 源 PostgreSQL 服务器版本必须是 9.5.11、9.6.7、10 或更高版本。 有关详细信息，请参阅[支持的 PostgreSQL 数据库版本](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions)一文。

    另外，本地 PostgreSQL 版本必须与 Azure Database for PostgreSQL 版本相符。 例如，PostgreSQL 9.5.11.5 只能迁移到 Azure Database for PostgreSQL 9.5.11，不能迁移到 9.6.7 版本。

    > [!NOTE]
    > 对于 PostgreSQL 版本 10，DMS 目前仅支持将版本 10.3 迁移到 Azure Database for PostgreSQL。

* [在 Azure Database for PostgreSQL 中创建实例](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)。  
* 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

    > [!NOTE]
    > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
    > * 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
    > * 存储终结点
    > * 服务总线终结点
    >
    > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。

* 请确保 VNet 网络安全组 (NSG) 规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)一文。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 PostgreSQL 服务器（默认情况下为 TCP 端口 5432）。
* 在源数据库的前面使用了防火墙设备时，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库。
* 为 Azure Database for PostgreSQL 创建服务器级[防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNet 子网范围。
* 可以通过两种方法来调用 CLI：

    * 在 Azure 门户右上角选择“Cloud Shell”按钮：

       ![Azure 门户中的“Cloud Shell”按钮](media/tutorial-postgresql-to-azure-postgresql-online/cloud-shell-button.png)

    * 在本地安装并运行 CLI。 CLI 2.0 是用于管理 Azure 资源的命令行工具。

       若要下载 CLI，请按照[安装 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 一文中的说明操作。 本文还列出了支持 CLI 2.0 的平台。

       若要设置适用于 Linux 的 Windows 子系统 (WSL)，请按照 [Windows 10 Installation Guide](https://docs.microsoft.com/windows/wsl/install-win10)（Windows 10 安装指南）中的说明操作

* 在 postgresql.config 文件中启用逻辑复制，并设置以下参数：

    * wal_level = **logical**
    * max_replication_slots = [槽数]，建议设置为“5 个槽”
    * max_wal_senders =[并发任务数] - max_wal_senders 参数设置可以运行的并发任务数，建议设置为“10 个任务”

## <a name="migrate-the-sample-schema"></a>迁移示例架构

若要完成所有数据库对象（例如表架构、索引和存储过程），需从源数据库提取架构并将其应用到此数据库。

1. 使用 pg_dump -s 命令为数据库创建架构转储文件。 

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    例如，若要转储 dvdrental 数据库的架构文件，请执行以下命令：
    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

    若要详细了解如何使用 pg_dump 实用程序，请参阅 [pg-dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES) 教程中的示例。

2. 在目标环境中创建一个空数据库，即 Azure Database for PostgreSQL。

    请参阅[在 Azure 门户中创建 Azure Database for PostgreSQL 服务器](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)一文，详细了解如何连接数据库，以及如何创建数据库。

3. 通过还原架构转储文件，将架构导入已创建的目标数据库。

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql 
    ```

    例如：

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. 如果架构中有外键，则迁移的初始加载和连续同步会失败。 请在 PgAdmin 或 psql 中执行以下脚本，以便在目标 (Azure Database for PostgreSQL) 中提取 drop foreign key 脚本和 add foreign key 脚本。

    
    ```
    SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ',
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema,
        tc.constraint_name as foreignkey,
        tc.table_name as tableName,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
    FROM
        information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
     ```

    运行查询结果中的 drop foreign key（第二列）。

5. 数据中的触发器（插入或更新触发器）会赶在源中的已复制数据之前在目标中强制实施数据完整性。 建议在迁移期间禁用**目标**的所有表中的触发器，然后在迁移完成后重新启用这些触发器。

    若要禁用目标数据库中的触发器，请使用以下命令：

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6. 如果任何表中有 ENUM 数据类型，建议在目标表中将其临时更新为“character varying”数据类型。 数据复制操作完成以后，请将数据类型还原为 ENUM。

## <a name="provisioning-an-instance-of-dms-using-the-cli"></a>使用 CLI 预配 DMS 的实例

1. 安装 dms 同步扩展：
   * 通过运行以下命令登录到 Azure：
       ```
       az login
       ```

   * 系统提示时，请打开 Web 浏览器并输入用于对设备进行身份验证的代码。 按照列出的说明操作。
   * 添加 dms 扩展：
       * 若要列出可用的扩展，请运行以下命令：

           ```
           az extension list-available –otable
           ```

       * 若要安装扩展，请运行以下命令：

           ```
           az extension add –n dms-preview
           ```

   * 若要验证是否已正确安装 dms 扩展，请运行以下命令：

       ```
       az extension list -otable
       ```
       应该会看到以下输出：

       ```
       ExtensionType    Name
       ---------------  ------
       whl              dms
       ```

   * 任何时候都可以通过运行以下命令来查看所有在 DMS 中受支持的命令：

       ```
       az dms -h
       ```

   * 如果有多个 Azure 订阅，请运行以下命令，以便设置需要用来预配 DMS 服务实例的订阅。

        ```
       az account set -s 97181df2-909d-420b-ab93-1bff15acb6b7
        ```

2. 通过运行以下命令来预配 DMS 的实例：

   ```
   az dms create -l [location] -n <newServiceName> -g <yourResourceGroupName> --sku-name BusinessCritical_4vCores --subnet/subscriptions/{vnet subscription id}/resourceGroups/{vnet resource group}/providers/Microsoft.Network/virtualNetworks/{vnet name}/subnets/{subnet name} –tags tagName1=tagValue1 tagWithNoValue
   ```

   例如，以下命令会创建一项具有下述参数的服务：

   * 位置：美国东部 2
   * 订阅：97181df2-909d-420b-ab93-1bff15acb6b7
   * 资源组名称：PostgresDemo
   * DMS 服务名称：PostgresCLI

   ```
   az dms create -l eastus2 -g PostgresDemo -n PostgresCLI --subnet /subscriptions/97181df2-909d-420b-ab93-1bff15acb6b7/resourceGroups/ERNetwork/providers/Microsoft.Network/virtualNetworks/AzureDMS-CORP-USC-VNET-5044/subnets/Subnet-1 --sku-name BusinessCritical_4vCores
   ```

   创建 DMS 服务的实例需要大约 10-12 分钟。

3. 若要确定 DMS 代理的 IP 地址，以便将其添加到 Postgres pg_hba.conf 文件，请运行以下命令：

    ```
    az network nic list -g <ResourceGroupName>--query '[].ipConfigurations | [].privateIpAddress'
    ```

    例如：

    ```
    az network nic list -g PostgresDemo --query '[].ipConfigurations | [].privateIpAddress'
    ```

    获得的结果应该类似于以下地址： 

    ```
    [
      "172.16.136.18"
    ]
    ```

4. 将 DMS 代理的 IP 地址添加到 Postgres pg_hba.conf 文件。

    * 在 DMS 中预配完以后，记下 DMS IP 地址。
    * 将 IP 地址添加到源中与以下条目类似的 pg_hba.conf 文件：

        ```
        host    all     all     172.16.136.18/10    md5
        host    replication     postgres    172.16.136.18/10    md5
        ```

5. 接下来请运行以下命令，创建 PostgreSQL 迁移项目：
    
    ```
    az dms project create -l <location> -g <ResourceGroupName> --service-name <yourServiceName> --source-platform PostgreSQL --target-platform AzureDbforPostgreSQL -n <newProjectName>
    ```

    例如，以下命令会创建一个使用下述参数的项目：

   * 位置：美国中西部
   * 资源组名称：PostgresDemo
   * 服务名称：PostgresCLI
   * 项目名称：PGMigration
   * 源平台：PostgreSQL
   * 目标平台：AzureDbForPostgreSql

     ```
     az dms project create -l eastus2 -n PGMigration -g PostgresDemo --service-name PostgresCLI --source-platform PostgreSQL --target-platform AzureDbForPostgreSql
     ```

6. 使用以下步骤创建 PostgreSQL 迁移任务。

    此步骤包括使用源 IP、UserID 和密码，目标 IP、UserID 和密码以及任务类型来建立连接。

   * 若要查看选项的完整列表，请运行以下命令：

       ```
       az dms project task create -h
       ```

       源和目标连接的输入参数都引用一个包含对象列表的 json 文件。

       适用于 PostgreSQL 连接的连接 JSON 对象的格式。
        
       ```
       {
                   "userName": "user name",    // if this is missing or null, you will be prompted
                   "password": null,           // if this is missing or null (highly recommended) you will
               be prompted
                   "serverName": "server name",
                   "databaseName": "database name", // if this is missing, it will default to the 'postgres'
               server
                   "port": 5432                // if this is missing, it will default to 5432
               }
       ```

   * 此外还有一个数据库选项 json 文件，该文件列出 json 对象。 对于 PostgreSQL，数据库选项 JSON 对象的格式如下所示：

       ```
       [
           {
               "name": "source database",
               "target_database_name": "target database",
           },
           ...n
       ]
       ```

   * 使用记事本创建一个 json 文件，复制以下命令并将其粘贴到文件中，然后将文件保存在 C:\DMS\source.json 中。

        ```
       {
                   "userName": "postgres",    
                   "password": null,           
               be prompted
                   "serverName": "13.51.14.222",
                   "databaseName": "dvdrental", 
                   "port": 5432                
               }
        ```

   * 创建另一个名为 target.json 的文件，将其另存为 C:\DMS\target.json。 包括以下命令：

       ```
       {
               "userName": " dms@builddemotarget",    
               "password": null,           
               "serverName": " builddemotarget.postgres.database.azure.com",
               "databaseName": "inventory", 
               "port": 5432                
           }
       ```

   * 创建一个数据库选项 json 文件，该文件将要迁移的数据库的选项以清单形式列出：

       ``` 
       [
           {
               "name": "dvdrental",
               "target_database_name": "dvdrental",
           }
       ]
       ```

   * 运行以下命令，其中包含源、目标和 DB 选项 json 文件。

       ``` 
       az dms project task create -g PostgresDemo --project-name PGMigration --source-platform postgresql --target-platform azuredbforpostgresql --source-connection-json c:\DMS\source.json --database-options-json C:\DMS\option.json --service-name PostgresCLI --target-connection-json c:\DMS\target.json –task-type OnlineMigration -n runnowtask    
       ```

     现在已成功提交一个迁移任务。

7. 若要显示任务进度，请运行以下命令：

   ```
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
   ```

   或

    ```
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output
    ```

8. 也可从 expand output 中查询 migrationState：

    ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output --query 'properties.output[].migrationState | [0]' "READY_TO_COMPLETE"
    ```

## <a name="understanding-migration-task-status"></a>了解迁移任务状态

在输出文件中，有多个指示迁移进度的参数。 有关示例，请查看下面的输出文件：

    ```
    "output": [                                 Database Level
          {
            "appliedChanges": 0,        //Total incremental sync applied after full load
            "cdcDeleteCounter": 0       // Total delete operation  applied after full load
            "cdcInsertCounter": 0,      // Total insert operation applied after full load
            "cdcUpdateCounter": 0,      // Total update operation applied after full load
            "databaseName": "inventory",
            "endedOn": null,
            "fullLoadCompletedTables": 2,   //Number of tables completed full load
            "fullLoadErroredTables": 0, //Number of tables that contain migration error
            "fullLoadLoadingTables": 0, //Number of tables that are in loading status
            "fullLoadQueuedTables": 0,  //Number of tables that are in queued status
            "id": "db|inventory",
            "incomingChanges": 0,       //Number of changes after full load
            "initializationCompleted": true,
            "latency": 0,
            "migrationState": "READY_TO_COMPLETE",  //Status of migration task. READY_TO_COMPLETE means the database is ready for cutover
            "resultType": "DatabaseLevelOutput",
            "startedOn": "2018-07-05T23:36:02.27839+00:00"
          },
          {
            "databaseCount": 1,
            "endedOn": null,
            "id": "dd27aa3a-ed71-4bff-ab34-77db4261101c",
            "resultType": "MigrationLevelOutput",
            "sourceServer": "138.91.123.10",
            "sourceVersion": "PostgreSQL",
            "startedOn": "2018-07-05T23:36:02.27839+00:00",
            "state": "PENDING",
            "targetServer": "builddemotarget.postgres.database.azure.com",
            "targetVersion": "Azure Database for PostgreSQL"
          },
          {                                     Table 1
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 0,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:20.740701+00:00",  //Full load completed time
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:15.864552+00:00",    //Full load started time
            "fullLoadTotalRows": 10,                    //Number of rows loaded in full load
            "fullLoadTotalVolumeBytes": 7056,               //Volume in Bytes in full load
            "id": "or|inventory|public|actor",          
            "lastModifiedTime": "2018-07-05T23:36:16.880174+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",                   //State of migration for this table
            "tableName": "public.catalog",              //Table name
            "totalChangesApplied": 0                //Total sync changes that applied after full load
          },
          {                                     Table 2
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 50,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:23.963138+00:00",
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:19.302013+00:00",
            "fullLoadTotalRows": 112,
            "fullLoadTotalVolumeBytes": 46592,
            "id": "or|inventory|public|address",
            "lastModifiedTime": "2018-07-05T23:36:20.308646+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",
            "tableName": "public.orders",
            "totalChangesApplied": 0
          }
        ],                          DMS migration task state
        "state": "Running", //Migration task state – Running means it is still listening to any changes that might come in                  
        "taskType": null
      },
      "resourceGroup": "PostgresDemo",
      "type": "Microsoft.DataMigration/services/projects/tasks"
    ```

## <a name="cutover-migration-task"></a>直接转换迁移任务

完全加载以后，即可对数据库进行直接转换。 完全加载以后，当新事务传入时，DMS 任务可能仍在应用所做的更改，具体取决于源服务器的繁忙程度。

若要确保所有数据都已捕获，请验证源和目标数据库的行计数。 例如，可以使用以下命令：

```
"migrationState": "READY_TO_COMPLETE", //Status of migration task. READY_TO_COMPLETE means database is ready for cutover
 "incomingChanges": 0,  //continue to check for a period of 5-10 minutes to make sure no new incoming changes that need to be applied to the target server
   "fullLoadTotalRows": 10, //full load for table 1
    "cdcDeleteCounter": 0,  //delete, insert and update counter on incremental sync after full load
    "cdcInsertCounter": 50,
    "cdcUpdateCounter": 0,
     "fullLoadTotalRows": 112,  //full load for table 2
```

1. 使用以下命令执行直接转换数据库迁移任务：

    ```
    az dms project task cutover -h
    ```

    例如：

    ```
    az dms project task cutover --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask  --database-name Inventory
    ```

2. 若要监视直接转换进度，请运行以下命令：

    ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

## <a name="service-project-task-cleanup"></a>服务、项目、任务清理

如需取消或删除任何 DMS 任务、项目或服务，请按以下顺序进行取消：

* 取消任何正在运行的任务
* 删除任务
* 删除项目
* 删除 DMS 服务

1. 若要取消正在运行的任务，请使用以下命令：

    ```
    az dms project task cancel --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
     ```

2. 若要删除正在运行的任务，请使用以下命令：
    ```
    az dms project task delete --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

3. 若要取消正在运行的项目，请使用以下命令：
     ```
    az dms project task cancel -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
     ```

4. 若要删除正在运行的项目，请使用以下命令：
    ```
    az dms project task delete -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
    ```

5. 若要删除 DMS 服务，请使用以下命令：

     ```
    az dms delete -g ProgresDemo -n PostgresCLI
     ```

## <a name="next-steps"></a>后续步骤

* 若要了解联机迁移到 Azure Database for PostgreSQL 时的已知问题和限制，请参阅 [Azure Database for PostgreSQL 联机迁移的已知问题和解决方法](known-issues-azure-postgresql-online.md)一文。
* 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
* 有关 Azure Database for PostgreSQL 的信息，请参阅[什么是 Azure Database for PostgreSQL？](https://docs.microsoft.com/azure/postgresql/overview)一文。