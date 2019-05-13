---
title: 为 Azure-SSIS 集成运行时配置性能 | Microsoft Docs
description: 了解如何配置高性能 Azure-SSIS Integration Runtime 的属性
services: data-factory
ms.date: 01/10/2018
ms.topic: conceptual
ms.service: data-factory
ms.workload: data-services
author: swinarko
ms.author: sawinark
ms.reviewer: ''
manager: craigg
ms.openlocfilehash: 42c69653a002446552da998320a43730dfdaadf5
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232533"
---
# <a name="configure-the-azure-ssis-integration-runtime-for-high-performance"></a>配置高性能 Azure-SSIS Integration Runtime

本文介绍如何配置高性能 Azure-SSIS Integration Runtime (IR)。 通过 Azure-SSIS IR 可以在 Azure 中部署和运行 SQL Server Integration Services (SSIS) 包。 有关 Azure-SSIS IR 的详细信息，请参阅 [Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime) 一文。 有关在 Azure 上部署和运行 SSIS 包的详细信息，请参阅[将 SQL Server Integration Services 工作负荷直接迁移到云](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview)。

> [!IMPORTANT]
> 本文包含 SSIS 开发团队成员进行的内部测试所得出的性能结果和观察数据。 结果可能不同。 在完成配置设置（会影响成本和性能）前进行自我测试。

## <a name="properties-to-configure"></a>要配置的属性

配置脚本的以下部分显示创建 Azure-SSIS Integration Runtime 时可以配置的属性。 有关完整的 PowerShell 脚本和说明，请参阅[将 SQL Server Integration Services 包部署到 Azure](tutorial-deploy-ssis-packages-azure-powershell.md)。

```powershell
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime information - This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[specify a name for your Azure-SSIS IR]"
$AzureSSISDescription = "[specify a description for your Azure-SSIS IR]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, while Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to max(2 x number of cores, 8) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL to provide SAS URI of blob container where your custom setup script and its associated files are stored
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use Azure SQL Database with virtual network service endpoints/Managed Instance/on-premises data, Azure Resource Manager virtual network is recommended, Classic virtual network will be deprecated soon
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Please use the same subnet as the one used with your Azure SQL Database with virtual network service endpoints or a different subnet than the one used for your Managed Instance

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name or Managed Instance name.DNS prefix].database.windows.net" # WARNING: Please ensure that there is no existing SSISDB, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure Active Directory (AAD)
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for AAD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for AAD authentication]"
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database or leave it empty for Managed Instance]"
```

## <a name="azuressislocation"></a>AzureSSISLocation
AzureSSISLocation 是 Integration Runtime 辅助角色节点的位置。 该辅助角色节点始终连接到 Azure SQL 数据库上的 SSIS 目录数据库 (SSISDB)。 将 AzureSSISLocation 位置设置为与托管 SSISDB 的 SQL 数据库服务器相同，确保集成运行时尽可能有效运行。

## <a name="azuressisnodesize"></a>AzureSSISNodeSize
数据工厂（包括 Azure-SSIS IR）支持以下选项：
-   Standard\_A4\_v2
-   Standard\_A8\_v2
-   Standard\_D1\_v2
-   Standard\_D2\_v2
-   Standard\_D3\_v2
-   Standard\_D4\_v2
-   标准\_D2\_v3
-   标准\_D4\_v3
-   标准\_D8\_v3
-   标准\_D16\_v3
-   标准\_D32\_v3
-   标准\_D64\_v3
-   标准\_E2\_v3
-   标准\_E4\_v3
-   标准\_E8\_v3
-   标准\_E16\_v3
-   标准\_E32\_v3
-   标准\_E64\_v3

在 SSIS 工程团队进行的非官方内部测试中，D 系列表现得比 A 系列更加适合 SSIS 包执行。

-   D 系列的性价比 A 系列更高版本，v3 系列的性价比 v2 系列更高版本。
-   D 系列的吞吐量高于 A 系列相同的价格和 v3 系列的吞吐量高于 v2 系列相同的价格。
-   Azure SSIS IR 的 v2 系列节点不是适用于自定义安装程序，因此请改为使用 v3 系列节点。 如果已使用 v2 系列节点，请切换到尽快使用 v3 系列节点。
-   E 系列可为内存优化 VM 大小提供更高内存 CPU 比率比其他计算机。如果为包所需的大量内存，则可以考虑选择 E 系列 VM。

### <a name="configure-for-execution-speed"></a>配置执行速度
如果没有很多要运行的包并且需要包快速运行，则使用下表中的信息选择一个适合方案的虚拟机类型。

此数据代表单个辅助角色节点上的单个包执行。 包从 Azure Blob 存储加载包含名和姓列 300 万条以上记录，生成全名列，然后将写入将全名超过 20 个字符到 Azure Blob 存储的记录。

![SSIS Integration Runtime 包执行速度](media/configure-azure-ssis-integration-runtime-performance/ssisir-execution-speedV2.png)

### <a name="configure-for-overall-throughput"></a>配置整体吞吐量

如果有大量包要运行且最关注整体吞吐量，则使用下表中的信息选择适合方案的虚拟机类型。

![SSIS Integration Runtime 最大整体吞吐量](media/configure-azure-ssis-integration-runtime-performance/ssisir-overall-throughputV2.png)

## <a name="azuressisnodenumber"></a>AzureSSISNodeNumber

AzureSSISNodeNumber 调整 Integration Runtime 的可伸缩性。 Integration Runtime 的吞吐量与 AzureSSISNodeNumber 成正比。 首先将 AzureSSISNodeNumber 设为一个较小值，监视 Integration Runtime 的吞吐量，然后针对方案调整值。 若要重新配置辅助角色节点数，请参阅[管理 Azure-SSIS Integration Runtime](manage-azure-ssis-integration-runtime.md)。

## <a name="azuressismaxparallelexecutionspernode"></a>AzureSSISMaxParallelExecutionsPerNode

已经在使用强大的辅助角色节点运行包时，增加 AzureSSISMaxParallelExecutionsPerNode 可能增加 Integration Runtime 的整体吞吐量。 对于 Standard_D1_v2 节点，支持每个节点 1-4 个并行执行。 对于所有其他类型的节点，支持每个节点 1 max(2 x number of cores, 8) 并行执行。 如果你想**AzureSSISMaxParallelExecutionsPerNode**超出我们所支持的最大值，可以开具支持票证，我们可以增加您需要使用 Azure Powershell 更新的最大值为你和之后**AzureSSISMaxParallelExecutionsPerNode**。
可以基于包的成本和辅助角色节点的以下配置估计合适的值。 有关详细信息，请参阅[常规用途虚拟机大小](../virtual-machines/windows/sizes-general.md)。

| 大小             | vCPU | 内存：GiB | 临时存储 (SSD) GiB | 最大临时存储吞吐量：IOPS/读取 MBps/写入 MBps | 最大的数据磁盘/吞吐量：IOPS | 最大 NIC 数/预期网络性能 (Mbps) |
|------------------|------|-------------|------------------------|------------------------------------------------------------|-----------------------------------|------------------------------------------------|
| Standard\_D1\_v2 | 1    | 3.5         | 50                     | 3000/46/23                                             | 2/2x500                         | 2 / 750                                        |
| Standard\_D2\_v2 | 2    | 7           | 100                    | 6000/93/46                                             | 4/4x500                         | 2 / 1500                                       |
| Standard\_D3\_v2 | 4    | 14          | 200                    | 12000/187/93                                           | 8/8x500                         | 4 / 3000                                       |
| Standard\_D4\_v2 | 8    | 28          | 400                    | 24000/375/187                                          | 16/16x500                       | 8 / 6000                                       |
| Standard\_A4\_v2 | 4    | 8           | 40                     | 4000/80/40                                             | 8/8x500                         | 4 / 1000                                       |
| Standard\_A8\_v2 | 8    | 16          | 80                     | 8000/160/80                                            | 16/16x500                       | 8 / 2000                                       |
| 标准\_D2\_v3 | 2    | 8           | 50                     | 3000/46/23                                             | 4 / 6 x 500                         | 2 / 1000                                       |
| 标准\_D4\_v3 | 4    | 16          | 100                    | 6000/93/46                                             | 8 / 12x500                        | 2 / 2000                                       |
| 标准\_D8\_v3 | 8    | 32          | 200                    | 12000/187/93                                           | 16 / 24x500                       | 4 / 4000                                       |
| 标准\_D16\_v3| 16   | 64          | 400                    | 24000/375/187                                          | 32 / 48 x 500                        | 8 / 8000                                       |
| 标准\_D32\_v3| 32   | 128         | 800                    | 48000/750/375                                          | 32 / 96x500                       | 8 / 16000                                      |
| 标准\_D64\_v3| 64   | 256         | 1600                   | 96000 / 1000 / 500                                         | 32 / 192x500                      | 8 / 30000                                      |
| 标准\_E2\_v3 | 2    | 16          | 50                     | 3000/46/23                                             | 4 / 6 x 500                         | 2 / 1000                                       |
| 标准\_E4\_v3 | 4    | 32          | 100                    | 6000/93/46                                             | 8 / 12x500                        | 2 / 2000                                       |
| 标准\_E8\_v3 | 8    | 64          | 200                    | 12000/187/93                                           | 16 / 24x500                       | 4 / 4000                                       |
| 标准\_E16\_v3| 16   | 128         | 400                    | 24000/375/187                                          | 32 / 48x500                       | 8 / 8000                                       |
| 标准\_E32\_v3| 32   | 256         | 800                    | 48000/750/375                                          | 32 / 96x500                       | 8 / 16000                                      |
| 标准\_E64\_v3| 64   | 432         | 1600                   | 96000 / 1000 / 500                                         | 32 / 192x500                      | 8 / 30000                                      |

以下是为 AzureSSISMaxParallelExecutionsPerNode 属性设置正确值的指南： 

1. 首先将它设置为一个较小值。
2. 少量增加该值，以检查是否提升整体吞吐量。
3. 在整体吞吐量达到最大值时停止增加值。

## <a name="ssisdbpricingtier"></a>SSISDBPricingTier

SSISDBPricingTier 是 Azure SQL 数据库上 SSIS 目录数据库 (SSISDB) 的定价层。 此设置影响 IR 实例中的最大辅助角色节点数、包执行排队速度以及执行日志加载速度。

-   如果不关注包执行排队和执行日志加载速度，可以选择最低的数据库定价层。 使用基本定价的 Azure SQL 数据库在 Integration Runtime 实例中支持 8 个辅助角色节点。

-   如果辅助角色节点计数大于 8 或者核心计数大于 50，则选择一个比基本版更强大的数据库。 否则，数据库会成为 Integration Runtime 实例的瓶颈，且整体性能会受到负面影响。

-   如果日志记录级别设置为详细，请选择更强大的数据库，例如 s3。 根据我们看到的非正式的内部测试，s3 定价层可以支持使用 2 个节点、 128 并行计数和详细日志记录级别执行的 SSIS 包。

还可以基于 Azure 门户上提供的[数据库事务单元](../sql-database/sql-database-what-is-a-dtu.md) (DTU) 使用情况信息调整数据库定价层。

## <a name="design-for-high-performance"></a>高性能设计
设计要在 Azure 上运行的 SSIS 包不同于设计本地执行的包。 不是将多个独立任务组合在同一包中，而是将它们分离到多个包中，从而提升在 Azure-SSIS IR 中执行的效率。 为每个包创建包执行，使它们不用等待彼此完成。 此方法受益于 Azure-SSIS Integration Runtime 的可伸缩性并提升整体吞吐量。

## <a name="next-steps"></a>后续步骤
详细了解 Azure-SSIS Integration Runtime。 请参阅 [Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime)。
