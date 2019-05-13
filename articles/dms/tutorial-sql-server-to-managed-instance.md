---
title: 教程：使用 DMS 迁移到 Azure SQL 数据库托管实例 | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务从本地 SQL Server 迁移到 Azure SQL 数据库托管实例。
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
ms.openlocfilehash: c768b7548b9759e85ebfb050f0ead2dfd3c1a6a6
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415107"
---
# <a name="tutorial-migrate-sql-server-to-an-azure-sql-database-managed-instance-offline-using-dms"></a>教程：使用 DMS 将 SQL Server 脱机迁移到 Azure SQL 数据库托管实例

可使用 Azure 数据库迁移服务将数据库从本地 SQL Server 实例迁移到 [Azure SQL 数据库托管实例](../sql-database/sql-database-managed-instance.md)。 有关需要一些手动操作的其他方法，请参阅[将 SQL Server 实例迁移到 Azure SQL 数据库托管实例](../sql-database/sql-database-managed-instance-migrate.md)一文。

本教程介绍如何使用 Azure 数据库迁移服务，将 Adventureworks2012 数据库从 SQL Server 的本地实例迁移到 Azure SQL 数据库托管实例。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> - 创建 Azure 数据库迁移服务的实例。
> - 使用 Azure 数据库迁移服务创建迁移项目。
> - 运行迁移。
> - 监视迁移。
> - 下载迁移报告。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文介绍如何从 SQL Server 脱机迁移到 Azure SQL 数据库托管实例。 有关联机迁移，请参阅[使用 DMS 将 SQL Server 联机迁移到 Azure SQL 数据库托管实例](tutorial-sql-server-managed-instance-online.md)。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

- 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 [了解使用 Azure 数据库迁移服务迁移 Azure SQL 数据库托管实例的网络拓扑](https://aka.ms/dmsnetworkformi)。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

    > [!NOTE]
    > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
    > - 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
    > - 存储终结点
    > - 服务总线终结点
    >
    > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。

- 请确保 VNet 网络安全组规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)一文。
- 配置[针对源数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 SQL Server（默认情况下为 TCP 端口 1433）。
- 如果使用动态端口运行多个命名 SQL Server 实例，则可能需要启用 SQL Browser 服务并允许通过防火墙访问 UDP 端口 1434，以便 Azure 数据库迁移服务可连接到源服务器上的命名实例。
- 如果在源数据库的前面使用了防火墙设备，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库，并通过 SMB 端口 445 访问文件。
- 按照[在 Azure 门户中创建 Azure SQL 数据库托管实例](https://aka.ms/sqldbmi)一文中的详述创建 Azure SQL 数据库托管实例。
- 确保用于连接源 SQL Server 和目标托管实例的登录名是 sysadmin 服务器角色的成员。
- 创建网络共享，供 Azure 数据库迁移服务用来备份源数据库。
- 确保运行源 SQL Server 实例的服务帐户对你创建的网络共享拥有写入权限，并且源服务器的计算机帐户具有对同一共享的读/写访问权限。
- 请记下在前面创建的网络共享中拥有完全控制权限的 Windows 用户（和密码）。 Azure 数据库迁移服务可模拟用户凭据，将备份文件上传到 Azure 存储容器，以执行还原操作。
- 遵循[使用存储资源管理器管理 Azure Blob 存储资源](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container)一文中的步骤创建 Blob 容器并检索其 SAS URI。创建 SAS URI 时，请务必在策略窗口中选择所有权限（读取、写入、删除、列出）。 此详细信息可为 Azure 数据库迁移服务提供你的存储帐户容器的访问权限，以便将用于迁移数据库的备份文件上传到 Azure SQL 数据库托管实例。

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

    ![显示门户订阅](media/tutorial-sql-server-to-managed-instance/portal-select-subscriptions.png)        

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-sql-server-to-managed-instance/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-sql-server-to-managed-instance/portal-register-resource-provider.png)

## <a name="create-an-azure-database-migration-service-instance"></a>创建 Azure 数据库迁移服务实例

1. 在 Azure 门户中，选择“+ 创建资源”，搜索“Azure 数据库迁移服务”，然后从下拉列表选择“Azure 数据库迁移服务”。

     ![Azure 市场](media/tutorial-sql-server-to-managed-instance/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-sql-server-to-managed-instance/dms-create1.png)

3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择要在其中创建 DMS 实例的位置。

5. 选择一个现有 VNet，或者创建一个。

    VNet 为 Azure 数据库迁移服务提供源 SQL Server 和目标 Azure SQL 数据库托管实例的访问权限。

    有关如何在 Azure 门户中创建 VNet 的详细信息，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

    有关更多详细信息，请参阅[使用 Azure 数据库迁移服务迁移 Azure SQL 数据库托管实例的网络拓扑](https://aka.ms/dmsnetworkformi)一文。

6. 选择定价层。

    有关成本和定价层的详细信息，请参阅[价格页](https://aka.ms/dms-pricing)。

    ![创建 DMS 服务](media/tutorial-sql-server-to-managed-instance/dms-create-service2.png)

7. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务实例后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

    ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-sql-server-to-managed-instance/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索创建的实例名称，然后选择该实例。

3. 选择“+ 新建迁移项目”。

4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“SQL Server”，在“目标服务器类型”文本框中选择“Azure SQL 数据库托管实例”，然后在“选择活动类型”中选择“脱机数据迁移”。

   ![创建 DMS 项目](media/tutorial-sql-server-to-managed-instance/dms-create-project2.png)

5. 选择“创建”来创建项目。

## <a name="specify-source-details"></a>指定源详细信息

1. 在“迁移源详细信息”屏幕上，指定源 SQL Server 的连接详细信息。

2. 如果尚未在服务器上安装受信任的证书，请选中“信任服务器证书”复选框。

    如果没有安装受信任的证书，SQL Server 会在实例启动时生成自签名证书。 此证书用于加密客户端连接的凭据。

    > [!CAUTION]
    > 使用自签名证书加密的 SSL 连接不提供可靠的安全性。 它们易遭受中间人攻击。 不得在生产环境中或在连接到 Internet 的服务器上依赖使用自签名证书的 SSL。

   ![源详细信息](media/tutorial-sql-server-to-managed-instance/dms-source-details1.png)

3. 选择“保存”。

4. 在“选择源数据库”屏幕上，选择用于迁移的“Adventureworks2012”数据库。

   ![选择源数据库](media/tutorial-sql-server-to-managed-instance/dms-source-database1.png)

    > [!IMPORTANT]
    > 如果使用 SQL Server Integration Services (SSIS)，则 DMS 目前不支持将 SSIS 项目/包的目录数据库 (SSISDB) 从 SQL Server 迁移到 Azure SQL 数据库托管实例。 但是，你可以在 Azure 数据工厂 (ADF) 中预配 SSIS 并将 SSIS 项目/包重新部署到由 Azure SQL 数据库托管实例承载的目标 SSISDB。 有关如何迁移 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 包迁移到 Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)。

5. 选择“保存”。

## <a name="specify-target-details"></a>指定目标详细信息

1. 在“迁移目标详细信息”屏幕上，指定目标的连接详细信息，该目标是要将“AdventureWorks2012”数据库迁移到其中的已提前预配的 Azure SQL 数据库托管实例。

    如果尚未预配 Azure SQL 数据库托管实例，请选择相应的[链接](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started)来帮助预配实例。 仍可继续进行项目创建，然后在 Azure SQL 数据库托管实例准备就绪后返回到此特定项目以执行迁移。

     ![选择目标](media/tutorial-sql-server-to-managed-instance/dms-target-details2.png)

2. 选择“保存”。

## <a name="select-source-databases"></a>选择源数据库

1. 在“选择源数据库”屏幕上，选择要迁移的源数据库。

    ![选择源数据库](media/tutorial-sql-server-to-managed-instance/select-source-databases.png)

2. 选择“保存”。

## <a name="select-logins"></a>选择登录名

1. 在“选择登录名”屏幕上，选择要迁移的登录名。

    >[!NOTE]
    >此版本仅支持迁移 SQL 登录名。

    ![选择登录名](media/tutorial-sql-server-to-managed-instance/select-logins.png)

2. 选择“保存”。

## <a name="configure-migration-settings"></a>配置迁移设置

1. 在“配置迁移设置”屏幕上，提供以下详细信息：

    | | |
    |--------|---------|
    |**选择源备份选项** | 如果已经有一个完整备份文件供 DMS 用于数据库迁移，请选择选项“我将提供最新备份文件”。 如果希望 DMS 先进行源数据库完整备份，然后用它来进行迁移，请选择选项“我将让 Azure 数据库迁移服务创建备份文件”。 |
    |**网络位置共享** | 可让 Azure 数据库迁移服务备份源数据库的本地 SMB 网络共享。 运行源 SQL Server 实例的服务帐户必须在此网络共享中拥有写入特权。 在网络共享中提供服务器的 FQDN 或 IP 地址，例如“'\\\servername.domainname.com\backupfolder”或“\\\IP address\backupfolder”。|
    |**用户名** | 确保 Windows 用户具有对上面提供的网络共享的完全控制权限。 Azure 数据库迁移服务将模拟用户凭据，将备份文件上传到 Azure 存储容器，以执行还原操作。 如果选择已启用 TDE 的数据库进行迁移，则上述 Windows 用户必须是内置的管理员帐户，且必须对 Azure 数据库迁移服务禁用[用户帐户控制](https://docs.microsoft.com/windows/security/identity-protection/user-account-control/user-account-control-overview)，才能上传和删除证书文件。 |
    |**密码** | 用户密码。 |
    |**存储帐户设置** | SAS URI 为 Azure 数据库迁移服务提供你的存储帐户容器的访问权限，服务可将备份文件上传到此容器，并用于将数据库迁移到 Azure SQL 数据库托管实例。 [了解如何获取 Blob 容器的 SAS URI](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container)。|
    |**TDE 设置** | 若要迁移启用了透明数据加密 (TDE) 的源数据库，必须拥有目标 Azure SQL 数据库托管实例的写入权限。  从下拉菜单中选择其中的 Azure SQL 数据库托管实例已预配的订阅。  在下拉菜单中选择目标 **Azure SQL 数据库托管实例**。 |

    ![配置迁移设置](media/tutorial-sql-server-to-managed-instance/dms-configure-migration-settings3.png)

2. 选择“保存”。

## <a name="review-the-migration-summary"></a>查看迁移摘要

1. 在“迁移摘要”屏幕的“活动名称”文本框中指定迁移活动的名称。

2. 展开“验证选项”部分以显示“选择验证选项”屏幕，指定是否要验证已迁移数据库的查询正确性，然后选择“保存”。

3. 查看并验证与迁移项目关联的详细信息。

    ![迁移项目摘要](media/tutorial-sql-server-to-managed-instance/dms-project-summary2.png)

4. 选择“保存”。

## <a name="run-the-migration"></a>运行迁移

- 选择“运行迁移”。

  迁移活动窗口随即出现，活动的状态为“挂起”。

## <a name="monitor-the-migration"></a>监视迁移

1. 在“迁移活动”屏幕中，选择“刷新”以更新显示。

   ![正在进行的迁移活动](media/tutorial-sql-server-to-managed-instance/dms-monitor-migration1.png)

    可以进一步展开数据库和登录类别，以监视相应服务器对象的迁移状态。

   ![正在进行的迁移活动](media/tutorial-sql-server-to-managed-instance/dms-monitor-migration-extend.png)

2. 迁移完成后，选择“下载报告”可获取报告，其上列出了与迁移过程相关的详细信息。

3. 验证目标 Azure SQL 数据库托管实例环境上的目标数据库。

## <a name="next-steps"></a>后续步骤

- 有关介绍如何使用 T-SQL RESTORE 命令将数据库迁移到托管实例的教程，请参阅[使用 restore 命令将备份还原到托管实例](../sql-database/sql-database-managed-instance-restore-from-backup-tutorial.md)。
- 有关托管实例的信息，请参阅[什么是托管实例](../sql-database/sql-database-managed-instance.md)。
- 若要了解如何将应用连接到托管实例，请参阅[连接应用程序](../sql-database/sql-database-managed-instance-connect-app.md)。
