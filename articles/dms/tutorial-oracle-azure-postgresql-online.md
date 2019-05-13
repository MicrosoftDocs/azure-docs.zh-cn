---
title: 教程：使用 Azure 数据库迁移服务将 Oracle 联机迁移到 Azure Database for PostgreSQL | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务将本地或虚拟机中的 Oracle 联机迁移到 Azure Database for PostgreSQL。
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
ms.openlocfilehash: b73249a9f72e4616f6d36e16f110913278f04590
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415606"
---
# <a name="tutorial-migrate-oracle-to-azure-database-for-postgresql-online-using-dms-preview"></a>教程：使用 DMS（预览版）将 Oracle 联机迁移到 Azure Database for PostgreSQL

可以使用 Azure 数据库迁移服务在尽量缩短停机时间的情况下，将本地或虚拟机中的 Oracle 数据库迁移到 [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/)。 换而言之，完成这种迁移只会对应用程序造成极短暂的停机。 本教程介绍如何使用 Azure 数据库迁移服务中的联机迁移活动，将 **HR** 示例数据库从 Oracle 11g 的本地或虚拟机实例迁移到 Azure Database for PostgreSQL。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 使用 ora2pg 工具评估迁移工作量。
> * 使用 ora2pg 工具迁移示例架构。
> * 创建 Azure 数据库迁移服务的实例。
> * 使用 Azure 数据库迁移服务创建迁移项目。
> * 运行迁移。
> * 监视迁移。

> [!NOTE]
> 使用 Azure 数据库迁移服务执行联机迁移需要基于“高级”定价层创建实例。

> [!IMPORTANT]
> 为获得最佳迁移体验，Microsoft 建议在目标数据库所在的 Azure 区域中创建 Azure 数据库迁移服务的实例。 跨区域或地理位置移动数据可能会减慢迁移过程并引入错误。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文介绍如何从 Oracle 联机迁移到 Azure Database for PostgreSQL。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

* 下载并安装 [Oracle 11g 发行版 2（Standard Edition、Standard Edition One 或 Enterprise Edition）](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)。
* 从[此处](https://docs.oracle.com/database/121/COMSC/installation.htm#COMSC00002)下载示例 **HR** 数据库。
* 在 [Windows](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows.pdf) 或 [Linux](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Linux.pdf) 中下载并安装 ora2pg。
* [在 Azure Database for PostgreSQL 中创建实例](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)。
* 参考[此文档](https://docs.microsoft.com/azure/postgresql/tutorial-design-database-using-azure-portal)中的说明连接到该实例并创建数据库。
* 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

  > [!NOTE]
  > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
  > * 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
  > * 存储终结点
  > * 服务总线终结点
  >
  > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。

* 请确保 VNet 网络安全组规则 (NSG) 未阻止 Azure 数据库迁移服务的以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)一文。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 Oracle 服务器（默认使用 TCP 端口 1521）。
* 在源数据库的前面使用了防火墙设备时，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库。
* 为 Azure Database for PostgreSQL 创建服务器级[防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNet 子网范围。
* 启用对源 Oracle 数据库的访问。

  > [!NOTE]
  > 用户需有 DBA 角色才能连接到 Oracle 源。

  * 需要在 Azure 数据库迁移服务中为增量同步启用存档重做日志，以捕获数据更改。 遵循以下步骤配置 Oracle 源：
    * 运行以下命令，使用 SYSDBA 特权登录：

      ```
      sqlplus (user)/(password) as sysdba
      ```

    * 运行以下命令关闭数据库实例。

      ```
      SHUTDOWN IMMEDIATE;
      ```

      等待出现确认消息 `'ORACLE instance shut down'`。

    * 运行以下命令启动新实例并装载（但不要打开）数据库，以启用或禁用存档：

      ```
      STARTUP MOUNT;
      ```

      必须装载数据库；等待出现确认消息“Oracle 实例已启动”。

    * 运行以下命令更改数据库存档模式：

      ```
      ALTER DATABASE ARCHIVELOG;
      ```

    * 运行以下命令打开数据库以进行正常操作：

      ```
      ALTER DATABASE OPEN;
      ```

      可能需要重启才能显示 ARC 文件。

    * 若要验证，请运行以下命令：

      ```
      SELECT log_mode FROM v$database;
      ```

      应会收到响应 `'ARCHIVELOG'`。 如果响应为 `'NOARCHIVELOG'`，则表示不符合要求。

  * 使用以下选项之一为复制启用补充日志记录。

    * **选项 1**。
      更改数据库级别补充日志记录，以涵盖包含 PK 和唯一索引的所有表。 检测查询将返回 `'IMPLICIT'`。

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY, UNIQUE) COLUMNS;
      ```

      更改表级别补充日志记录。 仅针对包含数据操作但不包含 PK 或唯一索引的表运行。

      ```
      ALTER TABLE [TABLENAME] ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    * **选项 2**。
      更改数据库级别补充日志记录以涵盖所有表，检测查询将返回 `'YES'`。

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
      ```

      更改表级别补充日志记录。 遵循以下逻辑，针对每个表仅运行一条语句。

      如果表包含主键：

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
      ```

      如果表包含唯一索引：

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG GROUP (first unique index columns) ALWAYS;
      ```

      否则，请运行以下命令：

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    若要验证，请运行以下命令：

      ```
      SELECT supplemental_log_data_min FROM v$database;
      ```

    应会收到响应 `'YES'`。

> [!IMPORTANT]
> 对于此方案的公共预览版，Azure 数据库迁移服务支持 Oracle 版本 10g 或 11g。 运行 Oracle 版本 12c 或更高版本的客户应注意，ODBC 驱动程序必须至少使用身份验证协议版本 8 连接到 Oracle。 对于版本 12c 或更高版本的 Oracle 源，必须按如下所述配置身份验证协议：
>
> * 更新 SQLNET.ORA：
>
>    ```
>    SQLNET.ALLOWED_LOGON_VERSION_CLIENT = 8
>    SQLNET.ALLOWED_LOGON_VERSION_SERVER = 8
>    ```
>
> * 重启计算机，使新设置生效。
> * 更改现有用户的密码：
>
>    ```
>    ALTER USER system IDENTIFIED BY {pswd}
>    ```
>
>   有关详细信息，请参阅[此网页](http://www.dba-oracle.com/t_allowed_login_version_server.htm)。
>
> 最后，请记住，更改身份验证协议可能会影响客户端身份验证。

## <a name="assess-the-effort-for-an-oracle-to-azure-database-for-postgresql-migration"></a>评估将 Oracle 迁移到 Azure Database for PostgreSQL 所要投入的工作量

我们建议使用 ora2pg 来评估从 Oracle 迁移到 Azure Database for PostgreSQL 所要投入的工作量。 使用 `ora2pg -t SHOW_REPORT` 指令创建一份报告，以列出所有 Oracle 对象、估算的迁移成本（以开发人员天数为单位），以及在转换过程中可能需要特别注意的某些数据库对象。

大多数客户会花费相当多的时间来审阅评估报告以及考虑自动和手动转换工作量。

若要配置并运行 ora2pg 来创建评估报告，请参阅[有关从 Oracle 迁移到 Azure Database for PostgreSQL 的 Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf)中的“迁移前：评估”部分。 [此处](http://ora2pg.darold.net/report.html)提供了一份示例 ora2pg 评估报告用于参考。

## <a name="export-the-oracle-schema"></a>导出 Oracle 架构

我们建议使用 ora2pg 将 Oracle 架构和其他 Oracle 对象（类型、过程、函数等）转换为与 Azure Database for PostgreSQL 兼容的架构。 ora2pg 中的许多指令可帮助你预定义某些数据类型。 例如，可以使用 `DATA_TYPE` 指令将所有 NUMBER(*,0) 替换为 bigint 而不是 NUMERIC(38)。

可以运行 ora2pg 导出 .sql 文件中的每个数据库对象。 然后，可以先检查 .sql 文件，再使用 psql 将其导入到 Azure Database for PostgreSQL；也可以在 PgAdmin 中执行 .SQL 脚本。

```
psql -f [FILENAME] -h [AzurePostgreConnection] -p 5432 -U [AzurePostgreUser] -d database 
```

例如：

```
psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database
```

若要配置并运行 ora2pg 来转换架构，请参阅[有关从 Oracle 迁移到 Azure Database for PostgreSQL 的 Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf)中的“迁移：架构和数据”部分。

## <a name="set-up-the-schema-in-azure-database-for-postgresql"></a>在 Azure Database for PostgreSQL 中设置架构

Oracle 默认将 schema.table.column 保留为全大写，而 PostgreSQL 将 schema.table.column 保留为小写。 要让 Azure 数据库迁移服务开始将 Oracle 中的数据转移到 Azure Database for PostgreSQL，schema.table.column 的大小写格式必须与 Oracle 源相同。

例如，如果 Oracle 源的架构为 “HR”.”EMPLOYEES”.”EMPLOYEE_ID”，则 PostgreSQL 架构必须使用相同的格式。

为了确保 Oracle 和 Azure Database for PostgreSQL 的 schema.table.column 大小写格式相同，我们建议执行以下步骤。

> [!NOTE]
> 可以使用不同的方法来派生大写架构。 我们正在努力改进并自动化此步骤。

1. 使用 ora2pg 导出小写的架构。 在表创建 SQL 脚本中，使用大写的“SCHEMA”手动创建架构。
2. 将剩余的 Oracle 对象（例如触发器、序列、过程、类型和函数）导入到 Azure Database for PostgreSQL 中。
3. 若将 TABLE 和 COLUMN 转换为大写，请运行以下脚本：

   ```
   -- INPUT: schema name
   set schema.var = “HR”;

   -- Generate statements to rename tables and columns
   SELECT 1, 'SET search_path = "' ||current_setting('schema.var')||'";'
   UNION ALL 
   SELECT 2, 'alter table "'||c.relname||'" rename '||a.attname||' to "'||upper(a.attname)||'";'
   FROM pg_class c
   JOIN pg_attribute a ON a.attrelid = c.oid
   JOIN pg_type t ON a.atttypid = t.oid
   LEFT JOIN pg_catalog.pg_constraint r ON c.oid = r.conrelid
    AND r.conname = a.attname
   WHERE c.relnamespace = (select oid from pg_namespace where nspname=current_setting('schema.var')) AND a.attnum > 0 AND c.relkind ='r'
   UNION ALL
   SELECT 3, 'alter table '||c.relname||' rename to "'||upper(c.relname)||'";'
   FROM pg_catalog.pg_class c
    LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
   WHERE c.relkind ='r' AND n.nspname=current_setting('schema.var')
   ORDER BY 1;
   ```

* 删除目标数据库中的外键，以运行完整加载。 有关可用于删除外键的脚本，请参阅[此文](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online)中的“迁移示例架构”部分。
* 使用 Azure 数据库迁移服务运行完整加载和同步。
* 当目标 Azure Database for PostgreSQL 实例中的数据与源同步后，在 Azure 数据库迁移服务中执行数据库交接。
* 若要将 SCHEMA、TABLE 和 COLUMN 转换为小写（如果应用程序查询要求 Azure Database for PostgreSQL 的架构采用小写），请运行以下脚本：

  ```
  -- INPUT: schema name
  set schema.var = hr;
  
  -- Generate statements to rename tables and columns
  SELECT 1, 'SET search_path = "' ||current_setting('schema.var')||'";'
  UNION ALL
  SELECT 2, 'alter table "'||c.relname||'" rename "'||a.attname||'" to '||lower(a.attname)||';'
  FROM pg_class c
  JOIN pg_attribute a ON a.attrelid = c.oid
  JOIN pg_type t ON a.atttypid = t.oid
  LEFT JOIN pg_catalog.pg_constraint r ON c.oid = r.conrelid
     AND r.conname = a.attname
  WHERE c.relnamespace = (select oid from pg_namespace where nspname=current_setting('schema.var')) AND a.attnum > 0 AND c.relkind ='r'
  UNION ALL
  SELECT 3, 'alter table "'||c.relname||'" rename to '||lower(c.relname)||';'
  FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
  WHERE c.relkind ='r' AND n.nspname=current_setting('schema.var')
  ORDER BY 1;
  ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

   ![显示门户订阅](media/tutorial-oracle-azure-postgresql-online/portal-select-subscriptions.png)

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-oracle-azure-postgresql-online/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-oracle-azure-postgresql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>创建 DMS 实例

1. 在 Azure 门户中，选择 **+ 创建资源**，搜索 Azure 数据库迁移服务，然后从下拉列表选择**Azure 数据库迁移服务**。

    ![Azure 市场](media/tutorial-oracle-azure-postgresql-online/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-oracle-azure-postgresql-online/dms-create2.png)
  
3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择现有的 VNet，或新建一个 VNet。

    VNet 为 Azure 数据库迁移服务提供源 Oracle 和目标 Azure Database for PostgreSQL 实例的访问权限。

    若要详细了解如何在 Azure 门户中创建 VNet，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

5. 选择定价层。

    有关成本和定价层的详细信息，请参阅[价格页](https://aka.ms/dms-pricing)。

    ![配置 Azure 数据库迁移服务实例设置](media/tutorial-oracle-azure-postgresql-online/dms-settings5.png)

6. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

    ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-oracle-azure-postgresql-online/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索你创建的 Azure 数据库迁移服务实例名称，然后选择该实例。

    ![查找 Azure 数据库迁移服务实例](media/tutorial-oracle-azure-postgresql-online/dms-instance-search.png)

3. 选择“+ 新建迁移项目”。
4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“Oracle”，在“目标服务器类型”文本框中选择“Azure Database for PostgreSQL”。
5. 在“选择活动类型”部分选择“联机数据迁移”。

   ![创建数据库迁移服务项目](media/tutorial-oracle-azure-postgresql-online/dms-create-project5.png)

   > [!NOTE]
   > 也可以现在就选择“仅创建项目”来创建迁移项目，在以后再执行迁移。

6. 选择“保存”，注意成功使用 Azure 数据库迁移服务执行联机迁移所要满足的要求，然后选择“创建并运行活动”。

## <a name="specify-source-details"></a>指定源详细信息

* 在“添加源详细信息”屏幕上，指定源 Oracle 实例的连接详细信息。

  ![“添加源详细信息”屏幕](media/tutorial-oracle-azure-postgresql-online/dms-add-source-details.png)

## <a name="upload-oracle-oci-driver"></a>上传 Oracle OCI 驱动程序

1. 选择“保存”，然后在“安装 OCI 驱动程序”屏幕上，登录到你的 Oracle 帐户并从[此处](https://www.oracle.com/technetwork/topics/winx64soft-089540.html#ic_winx64_inst)下载驱动程序“instantclient-basiclite-windows.x64-12.2.0.1.0.zip”（37,128,586 字节）（SHA1 校验和：865082268）。
2. 将该驱动程序下载到某个共享文件夹中。

   确保该文件夹已与使用最低只读访问权限指定的用户名共享。 Azure 数据库迁移服务将访问并读取该共享，以通过模拟指定的用户名将 OCI 驱动程序上传到 Azure。

   指定的用户名必须是 Windows 用户帐户。

   ![OCI 驱动程序安装](media/tutorial-oracle-azure-postgresql-online/dms-oci-driver-install.png)

## <a name="specify-target-details"></a>指定目标详细信息

1. 选择“保存”，然后在“目标详细信息”屏幕上指定目标 Azure Database for PostgreSQL 服务器的连接详细信息，这是 **HR** 架构已部署到的 Azure Database for PostgreSQL 的提前预配实例。

    ![“目标详细信息”屏幕](media/tutorial-oracle-azure-postgresql-online/dms-add-target-details1.png)

2. 选择“保存”，然后在“映射到目标数据库”屏幕上，映射源和目标数据库以进行迁移。

    如果目标数据库包含的数据库名称与源数据库的相同，则 Azure 数据库迁移服务默认会选择目标数据库。

    ![映射到目标数据库](media/tutorial-oracle-azure-postgresql-online/dms-map-target-details.png)

3. 选择“保存”，在“迁移摘要”屏幕上的“活动名称”文本框中指定迁移活动的名称，然后查看摘要，确保源和目标详细信息与此前指定的信息相符。

    ![迁移摘要](media/tutorial-oracle-azure-postgresql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>运行迁移

* 选择“运行迁移”。

  迁移活动窗口随即出现，活动的“状态”为“正在初始化”。

## <a name="monitor-the-migration"></a>监视迁移

1. 在迁移活动屏幕上选择“刷新”，以便更新显示，直到迁移的“状态”显示为“正在运行”。

     ![活动状态 - 正在运行](media/tutorial-oracle-azure-postgresql-online/dms-activity-running.png)

2. 在“数据库名称”下选择特定数据库即可转到“完整数据加载”和“增量数据同步”操作的迁移状态。

    完整数据加载会显示初始加载迁移状态，而增量数据同步则会显示变更数据捕获 (CDC) 状态。

     ![活动状态 - 完整加载已完成](media/tutorial-oracle-azure-postgresql-online/dms-activity-full-load-completed.png)

     ![活动状态 - 增量数据同步](media/tutorial-oracle-azure-postgresql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>执行迁移直接转换

完成初始的完整加载以后，数据库会被标记为“直接转换可供执行”。

1. 如果准备完成数据库迁移，请选择“启动直接转换”。

2. 确保停止传入源数据库的所有事务；等到“挂起的更改”计数器显示 **0**。

   ![启动直接转换](media/tutorial-oracle-azure-postgresql-online/dms-start-cutover.png)

3. 依次选择“确认”、“应用”。
4. 当数据库迁移状态显示“已完成”后，请将应用程序连接到新的目标 Azure Database for PostgreSQL 实例。

 > [!NOTE]
 > 由于 PostgreSQL 的 schema.table.column 默认采用小写，你可以使用本文前面的“在 Azure Database for PostgreSQL 中设置架构”部分中的脚本，将大写转换为小写。

## <a name="next-steps"></a>后续步骤

* 若要了解联机迁移到 Azure Database for PostgreSQL 时的已知问题和限制，请参阅 [Azure Database for PostgreSQL 联机迁移的已知问题和解决方法](known-issues-azure-postgresql-online.md)一文。
* 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
* 有关 Azure Database for PostgreSQL 的信息，请参阅[什么是 Azure Database for PostgreSQL？](https://docs.microsoft.com/azure/postgresql/overview)一文。
