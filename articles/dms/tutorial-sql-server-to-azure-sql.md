---
title: 教程：使用 Azure 数据库迁移服务从 SQL Server 脱机迁移到 Azure SQL 数据库中的单一/共用数据库 | Microsoft Docs
description: 了解如何使用 Azure 数据库迁移服务从本地 SQL Server 脱机迁移到 Azure SQL 数据库中的单一数据库或共用数据库。
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
ms.openlocfilehash: 22e55379c79678a2792f439137e8d58d03cd8377
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415093"
---
# <a name="tutorial-migrate-sql-server-to-a-single-database-or-pooled-database-in-azure-sql-database-offline-using-dms"></a>教程：使用 DMS 将 SQL Server 脱机迁移到 Azure SQL 数据库中的单一数据库或共用数据库

可以使用 Azure 数据库迁移服务将数据库从本地 SQL Server 实例迁移到 [Azure SQL 数据库](https://docs.microsoft.com/azure/sql-database/)。 在本教程中，将通过使用 Azure 数据库迁移服务，将还原到 SQL Server 2016（或更高版本）本地实例的 Adventureworks2012 数据库迁移到 Azure SQL 数据库中的单一数据库或共用数据库。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> - 通过使用数据迁移助手来评估本地数据库。
> - 通过使用数据迁移助手迁移示例架构。
> - 创建 Azure 数据库迁移服务的实例。
> - 使用 Azure 数据库迁移服务创建迁移项目。
> - 运行迁移。
> - 监视迁移。
> - 下载迁移报告。

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

本文介绍如何从 SQL Server 脱机迁移到 Azure SQL 数据库中的单一数据库或共用数据库。 有关联机迁移，请参阅[使用 DMS 将 SQL Server 联机迁移到 Azure SQL 数据库](tutorial-sql-server-azure-sql-online.md)。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

- 下载并安装 [SQL Server 2016 或更高版本](https://www.microsoft.com/sql-server/sql-server-downloads)（任意版本）。
- 按照[启用或禁用服务器网络协议](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure)一文中的说明启用 TCP/IP 协议（在安装 SQL Server Express 时，会默认禁用它）。
- 遵循[使用 Azure 门户在 Azure SQL 数据库中创建单一数据库](https://docs.microsoft.com/azure/sql-database/sql-database-single-database-get-started)一文中的详细信息在 Azure SQL 数据库中创建单一（或入池）数据库。

    > [!NOTE]
    > 如果你使用 SQL Server Integration Services (SSIS) 并且希望将 SSIS 项目/包的目录数据库 (SSISDB) 从 SQL Server 迁移到 Azure SQL 数据库，则当你在 Azure 数据工厂 (ADF) 中预配 SSIS 时，系统会代表你自动创建和管理目标 SSISDB。 有关如何迁移 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 包迁移到 Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)。
  
- 下载并安装[数据迁移助手](https://www.microsoft.com/download/details.aspx?id=53595) v3.3 或更高版本。
- 使用 Azure 资源管理器部署模型创建 Azure 数据库迁移服务的 Azure 虚拟网络 (VNet)，它将使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 为本地源服务器提供站点到站点连接。 有关创建 VNet 的详细信息，请参阅[虚拟网络文档](https://docs.microsoft.com/azure/virtual-network/)，尤其是提供了分步详细信息的快速入门文章。

    > [!NOTE]
    > 在 VNet 设置期间，如果将 ExpressRoute 与 Microsoft 的网络对等互连一起使用，请将以下服务[终结点](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)添加到将在其中预配服务的子网：
    > - 目标数据库终结点（例如，SQL 终结点、Cosmos DB 终结点等）
    > - 存储终结点
    > - 服务总线终结点
    >
    > Azure 数据库迁移服务缺少 Internet 连接，因此必须提供此配置。

- 请确保 VNet 网络安全组规则未阻止到 Azure 数据库迁移服务以下入站通信端口：443、53、9354、445、12000。 有关 Azure VNet NSG 流量筛选的更多详细信息，请参阅[使用网络安全组筛选网络流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)一文。
- 配置[针对数据库引擎访问的 Windows 防火墙](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 打开 Windows 防火墙，使 Azure 数据库迁移服务能够访问源 SQL Server（默认情况下为 TCP 端口 1433）。
- 如果使用动态端口运行多个命名 SQL Server 实例，则可能需要启用 SQL Browser 服务并允许通过防火墙访问 UDP 端口 1434，以便 Azure 数据库迁移服务可连接到源服务器上的命名实例。
- 在源数据库的前面使用了防火墙设备时，可能需要添加防火墙规则以允许 Azure 数据库迁移服务访问要迁移的源数据库。
- 为 Azure SQL 数据库服务器创建服务器级 IP [防火墙规则](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允许 Azure 数据库迁移服务访问目标数据库。 提供用于 Azure 数据库迁移服务的 VNET 子网范围。
- 确保用于连接到源 SQL Server 实例的凭据具有 [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql) 权限。
- 确保用于连接到目标 Azure SQL 数据库实例的凭据具有目标 Azure SQL 数据库的 CONTROL DATABASE 权限。

## <a name="assess-your-on-premises-database"></a>访问本地数据库

在将数据从本地 SQL Server 实例迁移到 Azure SQL 数据库中的单一数据库或共用数据库之前，需要对 SQL Server 数据库进行评估，了解任何可能会阻止迁移的阻碍性问题。 使用数据迁移助手 v3.3 或更高版本，按照[执行 SQL Server 迁移评估](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem)一文中所述的步骤，完成对本地数据库的评估。 所需步骤汇总如下：

1. 在数据迁移助手中，选择“新建 (+)”图标，然后选择“评估”项目类型。
2. 指定项目名称，在“源服务器类型”文本框中，选择“SQL Server”，在“目标服务器类型”文本框中，选择“Azure SQL 数据库”，然后选择“创建”，从而创建项目。

    评估迁移到 Azure SQL 数据库中的单一数据库或共用数据库的源 SQL Server 数据库时，可以选择以下一种或两种评估报告类型：

   - 检查数据库兼容性
   - 检查功能奇偶校验

     默认情况下会选择这两种报告类型。

3. 在数据迁移助手的“选项”屏幕上，选择“下一步”。
4. 在“选择源”屏幕上的“连接到服务器”对话框中，向 SQL Server 提供连接详细信息，然后选择“连接”。
5. 在“添加源”对话框中，依次选择“AdventureWorks2012”、“添加”和“开始评估”。

    > [!NOTE]
    > 如果你使用 SSIS，则 DMA 目前不支持对源 SSISDB 进行评估。 但是，将 SSIS 项目/包重新部署到由 Azure SQL 数据库托管的目标 SSISDB 时，会对其进行评估/验证。 有关如何迁移 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 包迁移到 Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)。

    评估完成后，结果将如下图所示：

    ![评估数据迁移](media/tutorial-sql-server-to-azure-sql/dma-assessments.png)

    对于 Azure SQL 数据库中国的单一数据库或共用数据库，评估结果可标识出在部署到单一数据库或共用数据库时会出现的功能奇偶一致性问题和迁移阻塞性问题。

    - SQL Server 功能奇偶校验类别在 Azure 中提供了一套全面的建议、可用的替代方法和缓解步骤，以帮助你在迁移项目中规划工作。
    - 兼容性问题类别说明了部分支持或完全不支持的功能，这些功能反映了可能会阻止本地 SQL Server 数据库迁移到 Azure SQL 数据库的兼容性问题。 此外，还提供了一些建议来帮助你解决这些问题。

6. 通过选择特定的选项来查看迁移阻塞问题和功能奇偶校验问题的评估结果。

## <a name="migrate-the-sample-schema"></a>迁移示例架构

如果对评估感到满意，并确信所选数据库适合迁移到 Azure SQL 数据库中的单一数据库或共用数据库，请使用 DMA 将架构迁移到 Azure SQL 数据库。

> [!NOTE]
> 在数据迁移助手中创建迁移项目之前，请确保已按照先决条件中的说明预配了 Azure SQL 数据库。 出于本教程的目的，假设 Azure SQL 数据库的名称是“AdventureWorksAzure”，但是你可按照自己意愿使用任意名称命名。
> [!IMPORTANT]
> 如果你使用 SSIS，则 DMA 目前不支持迁移源 SSISDB，但你可以将 SSIS 项目/包重新部署到由 Azure SQL 数据库托管的目标 SSISDB。 有关如何迁移 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 包迁移到 Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)。

要将 AdventureWorks2012 架构迁移到 Azure SQL 数据库中的单一数据库或共用数据库，请执行以下步骤：

1. 在数据迁移助手中，选择“新建 (+)”图标，然后在“项目类型”下选择“迁移”。
2. 指定项目名称，在“源服务器类型”文本框中，选择“SQL Server”，然后在“目标服务器类型”文本框中，选择“Azure SQL 数据库”。
3. 在“迁移范围”下，选择“仅架构”。

    在执行前面的步骤后，数据迁移助手界面应如下图所示：

    ![创建数据迁移助手项目](media/tutorial-sql-server-to-azure-sql/dma-create-project.png)

4. 选择“创建”来创建项目。
5. 在数据迁移助手中，指定 SQL Server 的源连接详细信息，依次选择“连接”和“AdventureWorks2012”数据库。

    ![数据迁移助手源连接详细信息](media/tutorial-sql-server-to-azure-sql/dma-source-connect.png)

6. 在“连接到目标服务器”下选择“下一步”，指定 Azure SQL 数据库的目标连接详细信息，选择“连接”，然后选择在 Azure SQL 数据库中已预配的“AdventureWorksAzure”数据库。

    ![数据迁移助手目标连接详细信息](media/tutorial-sql-server-to-azure-sql/dma-target-connect.png)

7. 选择“下一步”，以转到“选择对象”屏幕，可以在其中指定需要部署到 Azure SQL 数据库的“AdventureWorks2012”中的架构对象。

    默认情况下，选择所有对象。

    ![生成 SQL 脚本](media/tutorial-sql-server-to-azure-sql/dma-assessment-source.png)

8. 选择“生成 SQL 脚本”以创建 SQL 脚本，然后检查脚本是否有任何错误。

    ![架构脚本](media/tutorial-sql-server-to-azure-sql/dma-schema-script.png)

9. 选择“部署架构”以将架构部署到 Azure SQL 数据库，然后在部署架构后，检查目标服务器是否存在任何异常情况。

    ![部署架构](media/tutorial-sql-server-to-azure-sql/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>注册 Microsoft.DataMigration 资源提供程序

1. 登录到 Azure 门户，选择“所有服务”，然后选择“订阅”。

   ![显示门户订阅](media/tutorial-sql-server-to-azure-sql/portal-select-subscription1.png)

2. 选择要在其中创建 Azure 数据库迁移服务实例的订阅，再选择“资源提供程序”。

    ![显示资源提供程序](media/tutorial-sql-server-to-azure-sql/portal-select-resource-provider.png)

3. 搜索迁移服务，再选择“Microsoft.DataMigration”右侧的“注册”。

    ![注册资源提供程序](media/tutorial-sql-server-to-azure-sql/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>创建实例

1. 在 Azure 门户中，选择 **+ 创建资源**，搜索 Azure 数据库迁移服务，然后从下拉列表选择**Azure 数据库迁移服务**。

    ![Azure 市场](media/tutorial-sql-server-to-azure-sql/portal-marketplace.png)

2. 在“Azure 数据库迁移服务”屏幕上，选择“创建”。

    ![创建 Azure 数据库迁移服务实例](media/tutorial-sql-server-to-azure-sql/dms-create1.png)
  
3. 在“创建迁移服务”屏幕上，为服务、订阅以及新的或现有资源组指定名称。

4. 选择要在其中创建 Azure 数据库迁移服务实例的位置。

5. 选择现有的 VNet，或新建一个 VNet。

    VNet 为 Azure 数据库迁移服务提供了对源 SQL Server 和目标 Azure SQL 数据库实例的访问权限。

    若要详细了解如何在 Azure 门户中创建 VNet，请参阅[使用 Azure 门户创建虚拟网络](https://aka.ms/DMSVnet)一文。

6. 选择定价层。

    有关成本和定价层的详细信息，请参阅[价格页](https://aka.ms/dms-pricing)。

    ![配置 Azure 数据库迁移服务实例设置](media/tutorial-sql-server-to-azure-sql/dms-settings2.png)

7. 选择“创建”来创建服务。

## <a name="create-a-migration-project"></a>创建迁移项目

创建服务后，在 Azure 门户中找到并打开它，然后创建一个新的迁移项目。

1. 在 Azure 门户中，选择“所有服务”，搜索 Azure 数据库迁移服务，然后选择“Azure 数据库迁移服务”。

     ![查找 Azure 数据库迁移服务的所有实例](media/tutorial-sql-server-to-azure-sql/dms-search.png)

2. 在“Azure 数据库迁移服务”屏幕上，搜索你创建的 Azure 数据库迁移服务实例名称，然后选择该实例。

    ![查找 Azure 数据库迁移服务实例](media/tutorial-sql-server-to-azure-sql/dms-instance-search.png)

3. 选择“+ 新建迁移项目”。
4. 在“新建迁移项目”屏幕上指定项目名称，在“源服务器类型”文本框中选择“SQL Server”，在“目标服务器类型”文本框中选择“Azure SQL 数据库”，然后在“选择活动类型”中选择“脱机数据迁移”。 

    ![创建数据库迁移服务项目](media/tutorial-sql-server-to-azure-sql/dms-create-project2.png)

5. 选择“创建并运行活动”，以便创建项目并运行迁移活动。

## <a name="specify-source-details"></a>指定源详细信息

1. 在“迁移源详细信息”屏幕上，指定源 SQL Server 实例的连接详细信息。

    请确保为源 SQL Server 实例名称使用完全限定的域名 (FQDN)。 即使不能使用 DNS 名称解析，也可使用 IP 地址。

2. 如果尚未在源服务器上安装受信任的证书，请选中“信任服务器证书”复选框。

    如果没有安装受信任的证书，SQL Server 会在实例启动时生成自签名证书。 此证书用于加密客户端连接的凭据。

    > [!CAUTION]
    > 使用自签名证书加密的 SSL 连接不提供可靠的安全性。 它们易遭受中间人攻击。 不得在生产环境中或在连接到 Internet 的服务器上依赖使用自签名证书的 SSL。

   ![源详细信息](media/tutorial-sql-server-to-azure-sql/dms-source-details2.png)

    > [!IMPORTANT]
    > 如果你使用 SSIS，则 DMS 目前不支持迁移源 SSISDB，但你可以将 SSIS 项目/包重新部署到由 Azure SQL 数据库托管的目标 SSISDB。 有关如何迁移 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 包迁移到 Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)。

## <a name="specify-target-details"></a>指定目标详细信息

1. 选择“保存”，然后在“迁移目标详细信息”屏幕中，指定目标 Azure SQL 数据库服务器的连接详细信息，这是使用数据迁移助手向其部署 AdventureWorks2012 架构的预配 Azure SQL 数据库。

    ![选择目标](media/tutorial-sql-server-to-azure-sql/dms-select-target2.png)

2. 选择“保存”，然后在“映射到目标数据库”屏幕上，映射源和目标数据库以进行迁移。

    如果目标数据库包含的数据库名称与源数据库的相同，则 Azure 数据库迁移服务会默认选择目标数据库。

    ![映射到目标数据库](media/tutorial-sql-server-to-azure-sql/dms-map-targets-activity2.png)

3. 在“选择表”屏幕上选择“保存”，展开表列表，如何查看受影响字段的列表。

    请注意，Azure 数据库迁移服务会自动选择目标 Azure SQL 数据库实例上的所有空源表。 若要重新迁移已含数据的表，需要在此边栏选项卡上明确选择表。

    ![选择表](media/tutorial-sql-server-to-azure-sql/dms-configure-setting-activity2.png)

4. 在“迁移摘要”屏幕上，选择“保存”，在“活动名称”文本框中，指定迁移活动的名称。

5. 展开“验证选项”部分以显示“选择验证选项”屏幕，然后指定是否对已迁移数据库执行“架构比较”、“数据一致性”和“查询正确性”验证。

    ![选择“验证”选项](media/tutorial-sql-server-to-azure-sql/dms-configuration2.png)

6. 选择“保存”，查看摘要以确保源和目标详细信息与你先前指定的内容匹配。

    ![迁移摘要](media/tutorial-sql-server-to-azure-sql/dms-run-migration2.png)

## <a name="run-the-migration"></a>运行迁移

- 选择“运行迁移”。

    迁移活动窗口随即出现，活动的“状态”为“挂起”。

    ![活动状态](media/tutorial-sql-server-to-azure-sql/dms-activity-status1.png)

## <a name="monitor-the-migration"></a>监视迁移

1. 在迁移活动屏幕上，选择“刷新”来更新显示，直到迁移的“状态”显示为“已完成”。

    ![活动状态已完成](media/tutorial-sql-server-to-azure-sql/dms-completed-activity1.png)

2. 迁移完成后，选择“下载报告”可获取报告，其上列出了与迁移过程相关的详细信息。

3. 验证目标 Azure SQL 数据库服务器上的一个或多个目标数据库。

### <a name="additional-resources"></a>其他资源

- [使用 Azure 数据迁移服务 (DMS) 进行 SQL 迁移](https://www.microsoft.com/handsonlabs/SelfPacedLabs/?storyGuid=3b671509-c3cd-4495-8e8f-354acfa09587)练习实验室。
- 若要了解联机迁移到 Azure SQL 数据库时的已知问题和限制，请参阅 [Azure SQL 数据库联机迁移的已知问题和解决方法](known-issues-azure-sql-online.md)一文。
- 若要了解 Azure 数据库迁移服务，请参阅[什么是 Azure 数据库迁移服务？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
- 若要了解 Azure SQL 数据库，请参阅[什么是 Azure SQL 数据库服务？](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)一文。
