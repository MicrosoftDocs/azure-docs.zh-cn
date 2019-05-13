---
title: 教程：使用 Azure 数据库迁移服务将 RDS SQL Server 联机迁移到 Azure SQL 数据库或 Azure SQL 数据库托管实例 | Microsoft Docs
description: 了解使用 Azure 数据库迁移服务执行从 RDS SQL Server 到 Azure SQL 数据库或 Azure SQL 数据库托管实例的联机迁移操作。
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
ms.openlocfilehash: 982ead69ba3d206e1aa2538597927dcbeaab70e9
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416106"
---
# <a name="tutorial-migrate-rds-sql-server-to-azure-sql-database-or-an-azure-sql-database-managed-instance-online-using-dms"></a>教程：使用 DMS 将 RDS SQL Server 联机迁移到 Azure SQL 数据库或 Azure SQL 数据库托管实例
可以使用 Azure 数据库迁移服务将数据库从 RDS SQL Server 实例迁移到 [Azure SQL 数据库](https://docs.microsoft.com/azure/sql-database/)或 [Azure SQL 数据库托管实例](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index)，且几乎不用停机。 在本教程中，我们使用 Azure 数据库迁移服务，将还原到 SQL Server 2012（或更高版本） RDS SQL Server 实例的 Adventureworks2012 数据库迁移到 Azure SQL 数据库或 Azure SQL 数据库托管实例。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 创建 Azure SQL 数据库的实例，或 Azure SQL 数据库托管实例。 
> * 通过使用数据迁移助手迁移示例架构。
> * 创建 Azure 数据库迁移服务的实例。
> * 使用 Azure 数据库迁移服务创建迁移项目。
> * 运行迁移。
> * 监视迁移。
> * 下载迁移报告。

> [!NOTE]
> 使用 Azure 数据库迁移服务执行联机迁移需要基于“高级”定价层创建实例。 有关详细信息，请参阅 Azure 数据库迁移服务[定价](https://azure.microsoft.com/pricing/details/database-migration/)页。

> [!IMPORTANT]
> 为获得最佳迁移体验，Microsoft 建议在目标数据库所在的 Azure 区域中创建 Azure 数据库迁移服务的实例。 跨区域或地理位置移动数据可能会减慢迁移过程并引入错误。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文介绍如何从 RDS SQL Server 联机迁移到 Azure SQL 数据库或 Azure SQL 数据库托管实例。

## <a name="prerequisites"></a>先决条件
要完成本教程，需要：

* 创建 [RDS SQL Server 数据库](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.SQLServer.html)。
* 按照[在 Azure 门户中创建 Azure SQL 数据库](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal)一文中的详细信息创建 Azure SQL 数据库。

    > [!NOTE]
    > 若要迁移到 Azure SQL 数据库托管实例，请遵循[创建 Azure SQL 数据库托管实例](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started)一文中的详细信息，并创建名为 AdventureWorks2012 的空数据库。 
 
* 下载并安装[数据迁移助手](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) v3.3 或更高版本。
* 使用 Azure 资源管理器部署模型为 Azure 数据库迁移服务创建 Azure 虚拟网络 (VNet)。 若要迁移到 Azure SQL 数据库托管实例，请确保在用于 Azure SQL 数据库托管实例的同一 VNet 中（但在不同的子网）中创建 DMS 实例。  或者，如果对 DMS 使用不同的 VNet，则需要在两个 VNet 之间创建 VNet 对等互连。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

    > [!NOTE]
    > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
    >
    > * 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
    > * 存储终结点
    > * 服务总线终结点
    >
    > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。 

* 请确保 VNet 网络安全组规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)一文。
* 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
* 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 SQL Server（默认情况下为 TCP 端口 1433）。
* 为 Azure SQL 数据库服务器创建服务器级[防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNET 子网范围。
* 确保用于连接到源 RDS SQL Server 实例的凭据与属于“Processadmin”服务器角色的帐户相关联，并与属于要迁移的所有数据库上的“db_owner”数据库角色的帐户相关联。
* 确保用于连接到目标 Azure SQL 数据库实例的凭据具有目标 Azure SQL 数据库的 CONTROL DATABASE 权限；如果迁移到 Azure SQL 数据库托管实例，该帐户需是 sysadmin 角色的成员。
* 源 RDS SQL Server 版本必须为 SQL Server 2012 和更高版本。 若要确定 SQL Server 实例正在运行的版本，请参阅[如何确定 SQL Server 及其组件的版本、版本类别和更新级别](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an)一文。
* 在选择迁移的 RDS SQL Server 数据库和所有用户表中启用变更数据捕获 (CDC)。
    > [!NOTE]
    > 可使用以下脚本在 RDS SQL Server 数据库中启用 CDC。
    ```
    exec msdb.dbo.rds_cdc_enable_db 'AdventureWorks2012'
    ```
    > 可使用以下脚本在所有表中启用 CDC。
    ```
    use <Database name>
    go
    exec sys.sp_cdc_enable_table 
    @source_schema = N'Schema name', 
    @source_name = N'table name', 
    @role_name = NULL, 
    @supports_net_changes = 1 --for PK table 1, non PK tables 0
    GO
    ```
* 禁用目标 Azure SQL 数据库的数据库触发器。
    > [!NOTE]
    > 可以使用以下查询查找目标 Azure SQL 数据库的数据库触发器：
    ```
    Use <Database name>
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```
    有关详细信息，请参阅 [DISABLE TRIGGER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017) 一文。

## <a name="migrate-the-sample-schema"></a>迁移示例架构
使用 DMA 将架构迁移到 Azure SQL 数据库。

> [!NOTE]
> 在 DMA 中创建迁移项目之前，请确保已按照先决条件中的说明预配了 Azure SQL 数据库。 本教程假设 Azure SQL 数据库的名称是 **AdventureWorks2012**，但你可以根据需要提供任意名称。

若要将 AdventureWorks2012 架构迁移到 Azure SQL 数据库，请执行以下步骤：

1. 在数据迁移助手中，选择“新建 (+)”图标，然后在“项目类型”下选择“迁移”。
2. 指定项目名称，在“源服务器类型”文本框中，选择“SQL Server”，然后在“目标服务器类型”文本框中，选择“Azure SQL 数据库”。
3. 在“迁移范围”下，选择“仅架构”。

    在执行前面的步骤后，DMA 界面应如下图所示：

    ![创建数据迁移助手项目](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-create-project.png)

4. 选择“创建”来创建项目。
5. 在 DMA 中指定 SQL Server 的源连接详细信息，选择“连接”，然后选择“AdventureWorks2012”数据库。

    ![数据迁移助手源连接详细信息](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-source-connect.png)

6. 在“连接到目标服务器”下选择“下一步”，指定 Azure SQL 数据库的目标连接详细信息，选择“连接”，然后选择在 Azure SQL 数据库中预配的“AdventureWorksAzure”数据库。

    ![数据迁移助手目标连接详细信息](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-target-connect.png)

7. 选择“下一步”，以转到“选择对象”屏幕，可以在其中指定需要部署到 Azure SQL 数据库的“AdventureWorks2012”中的架构对象。

    默认情况下，选择所有对象。

    ![生成 SQL 脚本](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-assessment-source.png)

8. 选择“生成 SQL 脚本”以创建 SQL 脚本，然后检查脚本是否有任何错误。

    ![架构脚本](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-script.png)

9. 选择“部署架构”以将架构部署到 Azure SQL 数据库，然后在部署架构后，检查目标服务器是否存在任何异常情况。

    ![部署架构](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

   ![显示门户订阅](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-select-subscription1.png)

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-sql-server-to-azure-sql-online/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>创建实例

1. 在 Azure 门户中，选择 **+ 创建资源**，搜索 Azure 数据库迁移服务，然后从下拉列表选择**Azure 数据库迁移服务**。

    ![Azure 市场](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择要在其中创建 Azure 数据库迁移服务实例的位置。 

5. 选择现有的 VNet，或新建一个 VNet。

    VNet 为 Azure 数据库迁移服务提供了对源 SQL Server 和目标 Azure SQL 数据库实例的访问权限。

    若要详细了解如何在 Azure 门户中创建 VNet，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

6. 选择定价层；对于此联机迁移，请务必选择“高级”定价层。

    有关成本和定价层的详细信息，请参阅[价格页](https://aka.ms/dms-pricing)。

     ![配置 Azure 数据库迁移服务实例设置](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-settings3.png)

7. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

      ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索你创建的 Azure 数据库迁移服务实例名称，然后选择该实例。

     ![查找 Azure 数据库迁移服务实例](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-instance-search.png)

3. 选择“+ 新建迁移项目”。
4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“AWS RDS for SQL Server”，在“目标服务器类型”文本框中选择“Azure SQL 数据库”。

    > [!NOTE]
    > 对于目标服务器类型，请选择“Azure SQL 数据库”以同时迁移到 Azure SQL 数据库单一数据库以及 Azure SQL 数据库托管实例。

5. 在“选择活动类型”部分选择“联机数据迁移”。

    > [!IMPORTANT]
    > 请确保选择“联机数据迁移”；此方案不支持脱机迁移。

    ![创建数据库迁移服务项目](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-project4.png)

    > [!NOTE]
    > 也可以现在就选择“仅创建项目”来创建迁移项目，在以后再执行迁移。

6. 选择“保存”。

7. 选择“创建并运行活动”，以便创建项目并运行迁移活动。

    ![创建并运行数据库迁移服务活动](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-and-run-activity1.png)

## <a name="specify-source-details"></a>指定源详细信息

1. 在“迁移源详细信息”屏幕上，指定源 SQL Server 实例的连接详细信息。

    请确保为源 SQL Server 实例名称使用完全限定的域名 (FQDN)。

2. 如果尚未在源服务器上安装受信任的证书，请选中“信任服务器证书”复选框。

    如果没有安装受信任的证书，SQL Server 会在实例启动时生成自签名证书。 此证书用于加密客户端连接的凭据。

    > [!CAUTION]
    > 使用自签名证书加密的 SSL 连接不提供可靠的安全性。 它们易遭受中间人攻击。 不得在生产环境中或在连接到 Internet 的服务器上依赖使用自签名证书的 SSL。

   ![源详细信息](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-source-details3.png)

## <a name="specify-target-details"></a>指定目标详细信息

1. 选择“保存”，然后在“迁移目标详细信息”屏幕中，指定目标 Azure SQL 数据库服务器的连接详细信息，这是使用 DMA 向其部署 AdventureWorks2012 架构的预配 Azure SQL 数据库。

    ![选择目标](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-select-target3.png)

2. 选择“保存”，然后在“映射到目标数据库”屏幕上，映射源和目标数据库以进行迁移。

    如果目标数据库包含的数据库名称与源数据库的相同，则 Azure 数据库迁移服务会默认选择目标数据库。

    ![映射到目标数据库](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-map-targets-activity4.png)

3. 在“选择表”屏幕上选择“保存”，展开表列表，如何查看受影响字段的列表。

    Azure 数据库迁移服务会自动选择目标 Azure SQL 数据库实例上的所有空源表。 若要重新迁移已包含数据的表，需要在此屏幕上显式选择表。

    ![选择表](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-configure-setting-activity4.png)

4. 设置以下“高级联机迁移设置”后，选择“保存”。

    | 设置 | 说明 |
    | ------------- | ------------- |
    | **可并行加载的最大表数目** | 指定 DMS 在迁移期间并行执行的表数。 默认值为 5，但可以根据任何 POC 迁移方案的具体迁移需求，将其设置为最佳值。 |
    | **截断源表时** | 指定 DMS 在迁移期间是否截断目标表。 如果在迁移过程中截断了一个或多个表，此设置可能有所帮助。 |
    | **配置大型对象(LOB)数据设置** | 指定 DMS 是要迁移不受限制的 LOB 数据，还是将迁移的 LOB 数据限制为特定的大小。  对迁移的 LOB 数据施加限制后，将截断超出该限制的任何 LOB 数据。 对于生产迁移，我们建议选择“允许不受限制的 LOB 大小”以防止数据丢失。 指定允许不受限制的 LOB 大小时，请选中“当 LOB 大小小于指定的 KB 时以单个块的形式迁移 LOB 数据”复选框，以提高性能。 |

    ![设置高级联机迁移设置](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-advanced-online-migration-settings.png)

5. 选择“保存”，在“迁移摘要”屏幕上的“活动名称”文本框中指定迁移活动的名称，然后查看摘要，确保源和目标详细信息与此前指定的信息相符。

    ![迁移摘要](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-migration-summary.png)

## <a name="run-the-migration"></a>运行迁移

* 选择“运行迁移”。

    迁移活动窗口随即出现，活动的“状态”为“正在初始化”。

    ![活动状态 - 正在初始化](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-status2.png)

## <a name="monitor-the-migration"></a>监视迁移

1. 在迁移活动屏幕上选择“刷新”，以便更新显示，直到迁移的“状态”显示为“正在运行”。

2. 单击特定数据库即可转到“完整数据加载”和“增量数据同步”操作的迁移状态。

    ![活动状态 - 正在进行](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>执行迁移直接转换

完成初始的完整加载以后，数据库会被标记为“直接转换可供执行”。

1. 如果准备完成数据库迁移，请选择“启动直接转换”。

    ![启动直接转换](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-start-cutover.png)

2. 确保停止传入源数据库的所有事务；等到“挂起的更改”计数器显示 **0**。
3. 选择“确认”，然后选择“应用”。
4. 当数据库迁移状态显示“已完成”后，请将应用程序连接到新的目标 Azure SQL 数据库。

    ![活动状态 - 已完成](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-completed.png)

## <a name="next-steps"></a>后续步骤

* 若要了解联机迁移到 Azure SQL 数据库时的已知问题和限制，请参阅 [Azure SQL 数据库联机迁移的已知问题和解决方法](known-issues-azure-sql-online.md)一文。
* 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
* 若要了解 Azure SQL 数据库，请参阅[什么是 Azure SQL 数据库服务？](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)一文。
* 若要了解 Azure SQL 数据库托管实例的信息，请参阅 [Azure SQL 数据库托管实例](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index)页。
