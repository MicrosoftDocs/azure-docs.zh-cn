---
title: SAP 工作负荷规划和部署清单 | Microsoft Docs
description: 用于在 Azure 上规划和部署 SAP 工作负荷的清单
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/01/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 93fae0babdee5eac87d50679fdd5b2b938c4df2e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236873"
---
# <a name="sap-workload-on-azure-planning-and-deployment-checklist"></a>Azure 上的 SAP 工作负荷规划和部署清单 

此清单适用于将 SAP NetWeaver、S/4HANA 和 Hybris 应用程序迁移到 Azure 基础结构即服务的客户。  在项目进行期间，客户和/或 SAP 合作伙伴应审核此清单。 注意，许多检查都在项目开始时和规划阶段进行。 部署完成后，已部署的 Azure 基础结构或 SAP 软件版本的基本更改可能会变得复杂。 在整个项目的重要节点查看此清单。  在小问题变严重之前可以将其检测出来，并且有足够的时间来重新设计和测试任何必要的更改。 该清单从未声称完整。 根据自己的具体情况，可能需要执行更多检查。 

收集的清单不包括独立于 Azure 的任务。  示例：SAP 应用程序接口在迁移到 Azure 公共云或托管提供程序期间发生更改。    

此清单还可用于已部署的系统。 自部署以来，可能就已添加了写入加速器、可用性区域和新 VM 类型等新功能。  因此，定期查看清单以确保了解 Azure 平台中的新功能非常有用。 

## <a name="project-preparation-and-planning-phase"></a>项目准备和规划阶段
在此阶段，计划将 SAP 工作负荷迁移到 Azure 公共云。 讨论和定义的最小实体和项目集如下：

1. 高级设计文档 - 本文档应包含以下各项：
    1. Azure 上 SAP 组件和应用程序的当前清单以及目标应用程序清单
    2. 创建和处理责任分配矩阵 (RACI)。RACI 用于定义不同参与方的职责和任务分配。 从较高的层面开始，逐渐提高粒度级，以规划好吞吐量和首批部署
    2. 高级解决方案体系结构
    3. 关于部署到 Azure 区域的决定。 有关 Azure 区域的列表，请查看 [Azure 区域](https://azure.microsoft.com/global-infrastructure/regions/)。 有关每个 Azure 区域中可用的服务，请查看[每个区域可用的产品](https://azure.microsoft.com/global-infrastructure/services/)一文
    4. 网络体系结构，以从本地连接到 Azure。 开始熟悉 [Azure 的虚拟数据中心蓝图](https://docs.microsoft.com/azure/architecture/vdc/)
    5. 在 Azure 中运行业务影响较高的数据的安全原则。 相关阅读材料，请参阅 [Azure 安全文档](https://docs.microsoft.com/azure/security/)
2.  技术设计文档 - 包含：
    1.  解决方案块状图 
    2.  对 Azure 中的计算、存储和网络组件进行大小调整。 有关对 Azure VM 进行 SAP 大小调整的信息，请参阅 SAP 支持说明 [#1928533](https://launchpad.support.sap.com/#/notes/1928533) 
    3.  业务连续性和灾难恢复体系结构
    4.  详细的 OS、DB、内核和 SAP 支持包版本。 Azure VM 中并非支持 SAP NetWeaver 或 S/4HANA 支持的任何 OS 版本。 DBMS 版本也是如此。 为了使 SAP 版本、DBMS 版本或 OS 版本能够在 SAP 和 Azure 支持的窗口中保持一致，并在必要时进行升级，必须检查以下源代码。 必须使用 SAP 和 Azure 支持的版本组合，以获得 SAP 和 Microsoft 的全面支持。 必要时，需要计划升级某些软件组件。 以下位置提供了有关受支持的 SAP、OS 和 DBMS 软件的更多详细信息：
        1.  SAP 支持说明 [#1928533](https://launchpad.support.sap.com/#/notes/1928533)。 本说明定义了 Azure VM 支持的最低 OS 版本。 同时定义了大多数非 HANA 数据库所需的最低数据库版本。 此外，本说明还介绍了对 SAP 支持的不同 Azure VM 类型的 SAP 大小调整。
        2.  SAP 支持说明 [#2039619](https://launchpad.support.sap.com/#/notes/2039619)。 该说明定义了 Azure 上的 Oracle 支持对照表。 注意，对于 SAP 工作负荷，Oracle 仅支持 Windows 和 Oracle Linux 作为 Azure 中的来宾 OS。 此支持声明也适用于运行 SAP 实例的 SAP 应用层。 但是，Oracle 不支持通过 Pacemaker 在 Oracle Linux 中为 SAP Central Services 提供高可用性。 如果需要在 Oracle Linux 上实现 ASCS 的高可用性，需要利用适用于 Linux 的 SIOS 保护套件。 有关详细的 SAP 认证数据，请查看 SAP 支持说明 [#1662610 - 适用于 Linux 的 SIOS 保护套件的支持详细信息](https://launchpad.support.sap.com/#/notes/1662610)。 对于 Windows，可以结合用作 DBMS 层的 Oracle，来支持 SAP 所支持的、用于 SAP Central Services 的 Windows 故障转移群集解决方案。 
        3.  SAP 支持说明 [#2235581](https://launchpad.support.sap.com/#/notes/2235581)，获取不同 OS 版本的 SAP HANA 支持对照表
        4.  [此处](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)列出了 SAP HANA 支持的 Azure VM 和 [HANA 大型实例](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)
        5.  [SAP 产品可用性对照表](https://support.sap.com/en/)
        6.  其他 SAP 特定产品的其他 SAP 说明  
    5.  我们建议对 SAP 生产系统使用严格的 3 层设计。 建议不要在同一个 VM 上将 ASCS 与 APP 服务器结合使用。  在 Azure 中作为来宾 OS 的 Windows 支持使用 SAP 中央服务的多 SID 群集配置。 而 Azure 中的 Linux 操作系统不支持 SAP 中心服务多 SID 群集配置。 在以下位置可以找到有关 Windows 来宾 OS 用例的文档：
        1.  [使用 Azure 上的 Windows Server 故障转移群集和共享磁盘实现 SAP ASCS/SCS 实例多 SID 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-ascs-ha-multi-sid-wsfc-shared-disk)
        2.  [在 Azure 上使用 Windows Server 故障转移群集和文件共享实现 SAP ASCS/SCS 实例的多 SID 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-ascs-ha-multi-sid-wsfc-file-share)
    6.  高可用性和灾难恢复体系结构
        1.  根据 RTO 和 RPO 定义需要怎样的高可用性和灾难恢复体系结构
        2.  对于同一区域内的高可用性，请查看 Azure 中所需的 DBMS。 大多数 DBMS 提供同步热备用服务器的同步方法，我们建议将其用于生产系统。 另请查看不同数据库的 SAP 相关文档，从[部署适用于 SAP 工作负荷的 Azure 虚拟机 DBMS 的注意事项](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)开始
            1.  对于 DBMS 层，**不**支持使用采用共享磁盘配置的 Windows 故障转移群集服务，如[此处](https://docs.microsoft.com/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server?view=sql-server-2017)的 SQL Server 文档所述。 替代的解决方案包括：
                1.  [SQL Server AlwaysOn](https://docs.microsoft.com/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups) 
                2.  [Oracle Data Guard](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard)
                3.  [HANA 系统复制](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)
        3.  对于跨不同 Azure 区域的灾难恢复，请查看不同 DBMS 供应商提供的可能方案。 其中大部分都支持异步复制或日志传送
        4.  对于 SAP 应用层，请定义是否在同一 Azure 区域或 DR 区域中运行业务回归测试系统（理想情况下为生产部署的副本）。 在后一种情况下，你可以将该业务回归系统作为生产的 DR 目标
        5.  如果决定不在 DR 站点中放置非生产系统，可将 Azure Site Recovery 视为将 SAP 应用层复制到 Azure DR 区域的可行方法。 另请参阅[为多层 SAP NetWeaver 应用部署设置灾难恢复](https://docs.microsoft.com/azure/site-recovery/site-recovery-sap) 
        6.  如果决定使用组合的 HA/DR 配置，则可以利用 [Azure 可用性区域](https://docs.microsoft.com/azure/availability-zones/az-overview)使自己熟悉可用性区域均可用的 Azure 区域，同时熟悉可通过两个可用性区域之间增加的网络延迟引入的限制  
3.  客户/合作伙伴应创建所有 SAP 接口（SAP 和非 SAP）的清单。 
4.  基础服务设计 - 此设计将包括以下项目
    1.  Active Directory 和 DNS 设计
    2.  Azure 中的网络拓扑和不同 SAP 系统的分配
    3.  [基于角色的访问](https://docs.microsoft.com/azure/role-based-access-control/overview)结构，适用于管理 Azure 中的基础结构和 SAP 应用程序的不同团队
    3.  资源组拓扑 
    4.  [标记策略](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags#tags-and-billing)
    5.  VM 和其他基础结构组件和/或逻辑名称的命名约定
5.  Microsoft 顶级支持协定 - 确定 MS 技术部客户经理 (TAM)。 有关 SAP 的支持要求，请阅读 SAP 支持说明 [#2015553](https://launchpad.support.sap.com/#/notes/2015553) 
6.  定义 Azure 订阅数和不同订阅的核心配额。 根据需要[打开支持请求以增加 Azure 订阅的配额](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request) 
7.  用于将 SAP 数据迁移到 Azure 的数据缩减和数据迁移计划。 对于 SAP NetWeaver 系统，SAP 提供了有关如何限制大量数据量的指南。 SAP 发布了有关在 SAP ERP 系统中进行数据管理的[见解深刻的指南](https://help.sap.com/http.svc/rc/2eb2fba8f8b1421c9a37a8d7233da545/7.0/en-US/Data_Management_Guide_Version_70E.PDF)。 但是，某些内容通常适用于 NetWeaver 和 S/4HANA 系统。
8.  定义并确定自动部署方法。 对于 Azure 上基础结构部署，其自动化目标是以确定性的方式进行部署，并获得确定性的结果。 许多客户都使用基于 Powershell 或 CLI 的脚本。 不过各种开源技术均可用于为 SAP 部署 Azure 基础结构，甚至可以安装 SAP 软件。 可以在 GitHub 中找到示例：
    1.  [Azure 云中的自动 SAP 部署](https://github.com/Azure/sap-hana)
    2.  [SAP HANA 安装](https://github.com/AzureCAT-GSI/SAP-HANA-ARM)
9.  定义客户、系统集成商、Microsoft 和其他相关各方之间的常规设计和部署评审节奏

 
## <a name="pilot-phase-strongly-recommended"></a>试点阶段（强烈推荐）
 
试点阶段可以在项目规划和准备阶段之前运行，也可以并行运行。 该阶段还可用于测试规划和准备阶段确定的方法和设计。 试点阶段可以延伸到实际的概念证明。 建议在试点部署期间设置和验证完整的 HA/DR 解决方案和安全设计。 在某些客户案例中，可伸缩性测试也可以在此阶段进行。 另一些客户则将 SAP 沙盒系统的部署用作试点阶段。 因此，我们假设你确定了一个要迁移到 Azure 以便运行试点的系统。

1. 优化将数据传输到 Azure 的过程。 高度依赖于通过客户情况下传输[Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/)从本地是最快如果快速线路有足够的带宽。 对于其他客户来说，上网速度更快
2. 在 SAP 异构平台迁移的情况下，这涉及到数据库数据的导出和导入、测试以及优化导出和导入阶段。 对于涉及 SQL Server 作为目标平台的大型迁移，可以在[此处](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAP-OS-DB-Migration-to-SQL-Server-8211-FAQ-v6-2-April-2017/ba-p/368070)找到建议。 在合并迁移和 SAP 版本升级并实现某些源和目标 DBMS 平台组合（如 [SUM 2.0 SP03 的数据库迁移选项 (DMO)](https://launchpad.support.sap.com/#/notes/2631152) 中所述）时，如果不需要合并的版本升级或 [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) 过程，你可以采用迁移监视器/SWPM 的方法。 
   1.  导出到源、导出上传到 Azure 的文件和导入性能。  使导出和导入之间的重叠最大化
   2.  评估目标和目标平台之间的数据库容量，以反映到基础结构的大小调整中    
   3.  验证并优化执行时间 
3. 技术验证 
   1. VM 类型
      1.  再次验证 SAP 支持说明、SAP HANA 硬件目录和 SAP PAM 上的资源，以确保支持的 Azure VM、这些 VM 类型支持的 OS 版本以及支持的 SAP 和 DBMS 版本没有发生更改
      2.  再次验证 Azure 上部署的应用程序和基础结构的大小。 如果要移动现有应用程序，通常可以从使用的基础结构和 [SAP 基准网页](https://www.sap.com/dmc/exp/2018-benchmark-directory/#/sd)中获取必要的 SAPS，并将其与 SAP 支持说明 [#1928533](https://launchpad.support.sap.com/#/notes/1928533) 中列出的 SAPS 编号进行比较。 另请注意[这篇文章](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAPS-ratings-on-Azure-VMs-8211-where-to-look-and-where-you-can/ba-p/368208)
      3.  评估和测试 Azure VM 的大小，了解在规划阶段选择的不同 VM 类型的最大存储吞吐量和网络吞吐量。 数据可在以下位置找到：
          1.  [Azure 中 Windows 虚拟机的大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json)。 重要的是在调整大小时要考虑“最大非缓存磁盘吞吐量”
          2.  [Azure 中 Linux 虚拟机的大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json)：重要的是在调整大小时要考虑“最大非缓存磁盘吞吐量”
   2. 存储
      1.  使用[Azure 标准 SSD 存储](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types#standard-ssd)代表 SAP 应用程序层 Vm 和非性能敏感的 DBMS 部署最小
      2.  我们建议不应使用[Azure 标准 HDD 磁盘](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types#standard-hdd)一般情况下
      2.  使用[Azure 高级存储](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types#premium-ssd)对于任何远程是性能敏感的 DBMS Vm
      2.  使用 [Azure 托管磁盘](https://azure.microsoft.com/services/managed-disks/)
      3.  对于使用 M 系列的 DBMS 日志驱动器，请使用 Azure 写入加速器。 请注意[写入加速器](https://docs.microsoft.com/azure/virtual-machines/linux/how-to-enable-write-accelerator)中提供的写入加速器限制和使用情况
      4.  对于不同的 DBMS 类型，请查看[与 SAP 相关的 DBMS 通用文档](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)以及通用文档指向的特定于 DBMS 的文档
      5.  对于 SAP HANA，[Azure 上 的 SAP HANA 基础结构配置和操作](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)中提供了详细信息
      6.  请勿使用设备 ID 将 Azure 数据磁盘装载到 Azure Linux VM 中。 而应该使用全局唯一标识符 (UUID)。 例如，使用图形化工具装载 Azure 数据磁盘时应小心。 仔细检查 /etc/fstab 中的条目，确保使用 UUID 装载磁盘
          1.  在[此处](https://docs.microsoft.com/azure/virtual-machines/linux/attach-disk-portal#connect-to-the-linux-vm-to-mount-the-new-disk)可以找到更多详细信息
   3. 网络
      1.  测试和评估 VNet 基础结构以及跨不同 Azure 虚拟网络或在不同 Azure 虚拟网络内进行的 SAP 应用程序分发
          1.  在所基于的单一 Azure 虚拟网络内，评估中心和辐射虚拟网络体系结构或微分割的方法
              1.  由于[对等互连 Azure VNet](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) 之间的数据交换而产生的成本。 请查看[虚拟网络定价](https://azure.microsoft.com/pricing/details/virtual-network/)了解成本
              2.  当虚拟网络子网中托管的应用程序或 VM 存在安全风险时，与更改 NSG 以隔离虚拟网络中的子网相比，在 Azure 虚拟网络之间快速断开对等互连的优势
              3.  对本地、外部和 Azure 中构建的虚拟数据中心之间的网络流量集中进行日志记录和审核
          2.  评估和测试 SAP 应用层和 SAP DBMS 层之间的数据路径。 
              1.  不支持在 SAP 应用程序与 SAP NetWeaver、Hybris 或基于 S/4HANA 的 SAP 系统的 DBMS 层之间的通信路径中配置任何 [Azure 网络虚拟设备](https://azure.microsoft.com/solutions/network-appliances/)
              2.  不支持将 SAP 应用层和 SAP DBMS 放置在未对等互连的不同 Azure 虚拟网络中
              3.  定义 SAP 应用层和 SAP DBMS 层之间的路由时支持 [Azure ASG 和 NSG 规则](https://docs.microsoft.com/azure/virtual-network/security-overview)
          3.  确保在 SAP 应用层和 SAP DBMS 层上使用的 VM 启用了 [Azure 加速网络](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/)。 注意，需要不同的 OS 级别来支持 Azure 中的加速网络：
              1.  Windows Server 2012 R2 或更高版本
              2.  SUSE Linux 12 SP3 或更高版本
              3.  RHEL 7.4 或更高版本
              4.  Oracle Linux 7.5。 使用 RHCKL 内核时，版本需为 3.10.0-862.13.1.el7。 使用 Oracle UEK 内核时，需要版本 5
          4.   根据 SAP 支持说明 [#500235](https://launchpad.support.sap.com/#/notes/500235) 和 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 测试和评估 SAP 应用层 VM 和 DBMS VM 之间的网络延迟。 根据 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 评估网络延迟的结果。 网络延迟应该在中等且较好的范围内。 例外情况为 VM 和 HANA 大型实例单元之间的流量（如[此处](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-network-architecture#networking-architecture-for-hana-large-instance)所述）
          5.   确保 ILB 部署设置为使用直接服务器返回。 如果将 Azure ILB 用于 DBMS 层上的高可用性配置，此设置将减少延迟
          6.   如果与 Linux 来宾操作系统检查 Linux 网络参数结合使用 Azure 负载均衡器**net.ipv4.tcp_timestamps**设置为**0**。 针对较旧版本的 SAP 中的建议，请注意[#2382421](https://launchpad.support.sap.com/#/notes/2382421)。 SAP 说明同时会更新以反映这一事实，该参数必须设置为 0，以与 Azure 负载均衡器配合工作。
   4. 高可用性和灾难恢复部署。 
      1. 如果在未定义特定 Azure 可用性区域的情况下部署 SAP 应用层，请确保在[可用性集](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability)中部署所有运行 SAP 对话框实例或运行单个 SAP 系统的中间件实例的 VM。 
         1.   如果不需要 SAP 中心服务和 DBMS 的高可用性，则可以将这些 VM 部署到与 SAP 应用层相同的可用性集中
      2. 如果使用被动副本保护 SAP 中心服务和 DBMS 层以实现高可用性，请将 SAP 中心服务的两个节点放在一个单独的可用性集中，将两个 DBMS 节点放在另一个可用性集中
      3. 如果部署到 Azure 可用性区域，则无法利用可用性集。 但是，你需要确保将主动和被动中心服务节点部署到两个不同的可用性区域中，这两个可用性区域显示区域之间的最小延迟。
         1.   注意，如果跨可用性区域为 DBMS 和 SAP 中心服务层创建 Windows 或 Pacemaker 故障转移群集，则需要使用 [Azure 标准负载均衡器](https://docs.microsoft.com/azure/load-balancer/load-balancer-standard-availability-zones)。 [基本负载均衡器](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview#skus)不能用于区域部署 
   5. 超时设置
      1. 检查不同 SAP 实例的 SAP NetWeaver 开发人员跟踪，并确保排队服务器和 SAP 工作进程之间没有出现连接中断。 通过设置以下两个注册表参数，可以避免这些连接中断：
         1.   HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveTime = 120000 - 另请参阅[本文](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10))
         2.   HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveInterval = 120000 - 另请参阅[本文](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) 
      2. 为了避免在本地部署的 SAP GUI 接口和 Azure 中部署的 SAP 应用层之间的 GUI 超时，请检查是否在 default.pfl 或实例配置文件中设置了以下参数：
         1.   rdisp/keepalive_timeout = 3600
         2.   rdisp/keepalive = 20
      3. 如果使用 Windows 故障转移群集配置，请确保为 Azure 正确设置了用于对非响应节点做出响应的时间。 Microsoft 的[调整故障转移群集网络阈值](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834)一文列出了参数并介绍了这些参数如何影响故障转移敏感性。 假设群集节点都位于同一子网，应更改以下参数：
         1.   SameSubNetDelay = 2000
         2.   SameSubNetThreshold = 15
         3.   RoutingHistorylength = 30
4. 测试高可用性和灾难恢复过程
   1. 通过关闭 VM（Windows 来宾 OS）或使操作系统处于崩溃模式（Linux 来宾 OS）来模拟故障转移情况，以确定故障转移配置是否正常运行。 
   2. 测量执行故障转移所需的时间。 如果时间过长，请考虑：
      1.   对于 SUSE Linux，请使用 SBD 设备而非 Azure 隔离代理来加速故障转移
      2.   对于 SAP HANA，如果重新加载数据的时间过长，请考虑预配更多存储带宽
   3. 测试备份/还原顺序和时间，并在必要时进行优化。 确保不仅仅是备份时间足够。 另外请测试还原，并花费时间来完成还原活动。 如果 RTO 依赖于数据库或 VM 还原过程，请确保还原时间在 RTO SLA 范围内
   4. 测试跨区域 DR 功能和体系结构
5. 安全检查
   1.  测试实现的基于 Azure 角色的访问 (RBAC) 体系结构的有效性。 目标是分离和限制不同团队的访问和权限。 例如，SAP 基础团队成员应该可以部署 VM 并将 Azure 存储中的磁盘分配到给定的 Azure 虚拟网络中。 但是，SAP 基础团队应该不能创建自己的虚拟网络或更改现有虚拟网络的设置。 另一方面，网络团队的成员应该不能将 VM 部署到运行 SAP 应用程序和 DBMS VM 的虚拟网络中。 网络团队的成员也应该不能更改 VM 的属性，甚至删除 VM 或磁盘。  
   2.  验证 [NSG 和 ASC](https://docs.microsoft.com/azure/virtual-network/security-overview) 规则是否可以正常工作并屏蔽受保护资源
   3.  确保所有需要加密的资源都已加密。 定义和执行备份证书、存储和访问这些证书以及恢复加密实体的流程。 
   4.  从 OS 支持的角度来讲，尽可能使用 [Azure 磁盘加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption-faq)和/或 OS 磁盘
   5.  检查是否使用了太多的加密层。 使用 Azure 磁盘加密，然后另外使用某种 DBMS 透明数据库加密方法，确实具有一定意义。
6. 性能测试
   1.  在基于 SAP 跟踪和测量的 SAP 中，将前 10 个在线报告与当前的实现进行比较（如适用） 
   2.  在基于 SAP 跟踪和测量的 SAP 中，将前 10 个批处理作业与当前的实现进行比较（如适用） 
   3.  在基于 SAP 跟踪和测量的 SAP 中，比较通过接口传输到 SAP 系统的数据。 专注于您知道，传输现在将不同的位置，如从本地转移到 Azure 之间的接口 


## <a name="non-production-phase"></a>非生产阶段 
在此阶段，我们假定在成功试点或 PoC 之后，开始将非生产 SAP 系统部署到 Azure 中。 概念证明中的所有学知识和经验都应该适应这种部署。 PoC 中列出的所有标准和步骤也适用于此类部署。 在此阶段，通常将开发系统、单元测试系统和业务回归测试系统部署到 Azure 中。 建议 SAP 应用程序流程中至少要有一个非生产系统具有完整的高可用性配置，因为将来的生产系统将会需要。 在该阶段需要考虑的其他步骤如下：  

1.  在将系统从旧平台移至 Azure 之前，请先收集 CPU 使用情况、存储吞吐量和 IOPS 数据等资源消耗数据。 尤其是来自 DBMS 层单元（也包括应用层单元）的数据。 此外，还应测量网络和存储延迟。
2.  记录系统的可用性使用情况时间模式。 目标是确定非生产系统是否需要全天候可用，还是可以在一个月或一周的某些阶段关闭非生产系统
3.  测试并定义是为 Azure 中的 VM 创建自己的 OS 映像还是使用 Azure 映像库中的映像。 如果使用的是 Azure 库中的映像，请确保使用合适的映像（提供了与 OS 供应商的支持合同）。 对于某些 OS 供应商，Azure 库建议提供自己的许可证映像。 对于其他 OS 映像，Azure 报价中包含此项支持。 如果决定创建自己的 OS 映像，可以在这些文章中找到文档：
    1.  可以根据[此文档](https://docs.microsoft.com/azure/virtual-machines/windows/capture-image-resource)构建在 Azure 中部署的 Windows VM 的通用映像
    2.  可以根据[此文档](https://docs.microsoft.com/azure/virtual-machines/linux/capture-image)构建在 Azure 中部署的 Linux VM 的通用映像
3.  如果使用 Azure VM 库中的 SUSE 和 Red Hat Linux 映像，则需要使用 Linux 供应商在 Azure VM 库中提供的适用于 SAP 的映像
4.  确保你满足 SAP 对 Microsoft 支持协议的支持要求。 在 SAP 支持说明 [#2015553](https://launchpad.support.sap.com/#/notes/2015553) 中可以找到相关信息。 对于 HANA 大型实例，请参阅文档[加入要求](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-onboarding-requirements)
4.  请确保相关人员收到[计划内维护通知](https://azure.microsoft.com/blog/a-new-planned-maintenance-experience-for-your-virtual-machines/)，以便你可以及时选择停机时间并重启 VM
5.  在 [Channel9](https://channel9.msdn.com/) 之类的频道上，按照自己的步调查看采用 Microsoft 演示文稿 的 Azure 文档，以获得可能适用于部署的新功能
6.  检查与 Azure 相关的 SAP 说明，比如新的 VM SKU 或受支持的 OS 和 DBMS 新版本的支持说明 [#1928533](https://launchpad.support.sap.com/#/notes/1928533)。 在定价中将新 VM 类型与旧 VM 类型进行比较，以便你能够部署最具性价比的 VM
7.  再次验证 SAP 支持说明、SAP HANA 硬件目录和 SAP PAM 上的资源，以确保 Azure 支持的 VM、这些 VM 中支持的 OS 版本以及 SAP 和 DBMS 版本没有发生更改
8.  查看[此处](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)了解 Azure 中新的 HANA 认证 SKU，并将定价与计划的 SKU 进行比较，最终选择更具性价比的单元 
9.  调整部署脚本以利用新的 VM 类型，并集成你想要使用的 Azure 的新功能
10. 完成基础结构的部署后，根据 SAP 支持说明 [#500235](https://launchpad.support.sap.com/#/notes/500235) 和 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 测试和评估 SAP 应用层 VM 和 DBMS VM 之间的网络延迟。 根据 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 评估网络延迟的结果。 网络延迟应该在中等且较好的范围内。 例外情况为 VM 和 HANA 大型实例单元之间的通信（如[此处](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-network-architecture#networking-architecture-for-hana-large-instance)所述）。 确保[针对 SAP 工作负荷的 Azure 虚拟机 DBMS 部署注意事项](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general#azure-network-considerations)和 [Azure 上的 SAP HANA 基础结构配置和操作](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)中提及的所有限制均不适用于你的部署
11. 在应用工作负荷之前，执行“概念证明”阶段中列出的所有其他检查
12. 在应用工作负荷时，记录 Azure 中这些系统的资源消耗，并与从旧平台获得的记录进行比较。 如果发现差异较大，请调整未来部署的 VM 大小。 请记住，在缩小规模的情况下，VM 的存储和网络带宽也将减少：
    1.  [Azure 中 Windows 虚拟机的大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json)。 
    2.  [Azure 中 Linux 虚拟机的大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json) 
13. 处理系统复制功能和流程。 目标是使你能够轻松地复制开发系统或测试系统，以便项目团队能够快速地获得新系统。 将 [SAP LaMa](https://wiki.scn.sap.com/wiki/display/ATopics/SAP+Landscape+Management+%28SAP+LaMa%29+at+a+Glance) 视为执行此类任务的工具。
14. 优化和完善团队基于 Azure 角色的访问、权限和流程，以确保一方具有职责分离。 另一方面，你想要所有团队都能够在 Azure 基础结构中执行其任务。
15. 练习、测试和记录高可用性和灾难恢复过程，使你的员工能够执行这些任务。 找出缺点并调整要集成到部署中的新 Azure 功能

 
## <a name="production-preparation-phase"></a>生产准备阶段 
在此阶段，你想要收集非生产部署的所有经验和知识，并将其应用于未来的生产部署中。 除了以前的阶段之外，你还需要准备当前托管位置和 Azure 之间的数据传输工作。 

1.  在迁移到 Azure 之前，先完成生产系统的必要 SAP 版本升级
2.  与业务所有者就生产系统迁移后需要进行的功能和业务测试达成一致
    1.  确保使用当前托管位置中的源系统执行所有这些测试。 你想要避免在系统迁移到 Azure 之后的首次执行测试
2.  测试到 Azure 生产迁移过程。 如果未在同一时间范围内将所有生产系统移至 Azure，请构建需要位于同一托管位置的生产系统组。 练习和测试数据迁移。 类似的通用方法列表：
    1.  使用 DBMS 方法（如备份/还原）与 SQL Server AlwaysOn、HANA系统复制或日志传送结合使用，以在 Azure 中播种和同步数据库内容
    2.  对较小的数据库使用备份/还原
    3.  使用 SAP SWPM 工具中实现的 SAP 迁移监视器执行异类迁移
    4.  如果需要与 SAP 版本升级组合，请使用 [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) 过程。 请记住，并非所有源 DBMS 和目标 DBMS 之间的组合都受支持。 在有关 DMO 不同版本的特定 SAP 支持说明中可以找到更多信息。 例如，[SUM 2.0 SP04 的数据库迁移选项 (DMO)](https://launchpad.support.sap.com/#/notes/2644872)
    5.  在需要移动备份或 SAP 导出文件的情况下，测试通过 Internet 或 ExpressRoute 进行数据传输是否更好。 对于通过 Internet 移动数据的情况，可能需要更改一些你需要为将来的生产系统设置的 NSG/ASG 安全规则
3.  在将系统从旧平台移至 Azure 之前，请先收集 CPU 使用情况、存储吞吐量和 IOPS 数据等资源消耗数据。 尤其是来自 DBMS 层单元（也包括应用层单元）的数据。 此外，还应测量网络和存储延迟。
4.  再次验证 SAP 支持说明、SAP HANA 硬件目录和 SAP PAM 上的资源，以确保 Azure 支持的 VM、这些 VM 中支持的 OS 版本以及 SAP 和 DBMS 版本没有发生更改 
4.  将部署脚本调整为你决定的 VM 类型和 Azure 功能的最新更改
5.  在部署基础结构和应用程序之后，需要进行一系列检查以验证：
    1.  使用正确的属性和存储大小部署正确的 VM 类型
    2.  检查 VM 是否在正确且所需的 OS 版本和修补程序上，并且是统一的
    3.  检查强化 VM 是否符合要求且统一
    4.  检查是否已安装和部署了正确的应用程序版本和修补程序
    5.  VM 已按计划部署到 Azure 可用性集中
    6.  Azure 高级存储用于延迟敏感磁盘或需要[单个 VM SLA 为 99.9%](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/) 的情况
    7.  检查 Azure 写入加速器的正确部署
        1.  确保在需要 Azure 写入加速器支持的磁盘上正确构建 VM、存储空间或条带集
            1.  查看[在 Linux 上配置软件 RAID](https://docs.microsoft.com/azure/virtual-machines/linux/configure-raid)
            2.  查看[在 Azure 中的 Linux VM 上配置 LVM](https://docs.microsoft.com/azure/virtual-machines/linux/configure-lvm)
    8.  [Azure 托管磁盘](https://azure.microsoft.com/services/managed-disks/)是专用的
    9.  VM 已部署到正确的可用性集和可用性区域中
    10. 确保在 SAP 应用层和 SAP DBMS 层上使用的 VM 启用了 [Azure 加速网络](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/)
    11. 在 SAP 应用程序与SAP NetWeaver，Hybris或S / 4HANA的SAP系统的DBMS层之间的通信路径中未放置 [Azure 网络虚拟设备](https://azure.microsoft.com/solutions/network-appliances/)
    12. ASG 和 NSG 规则允许按需要和计划进行通信，并在需要时阻止通信
    13. 如前所述的超时设置已正确设置
    14. 根据 SAP 支持说明 [#500235](https://launchpad.support.sap.com/#/notes/500235) 和 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 测试和评估 SAP 应用层 VM 和 DBMS VM 之间的网络延迟。 根据 SAP 支持说明 [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E) 评估网络延迟的结果。 网络延迟应该在中等且较好的范围内。 例外情况为 VM 和 HANA 大型实例单元之间的流量（如[此处](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-network-architecture#networking-architecture-for-hana-large-instance)所述）
    15. 检查是否在必要时部署加密并使用加密方法
    16. 检查接口和其他应用程序是否可以连接新部署的基础结构
6.  创建用于响应 Azure 计划内维护的 playbook。 在计划内维护的情况下，定义要重新启动的系统和 VM 的顺序
    

## <a name="go-live-phase"></a>投入使用阶段
对于投入使用阶段，需要确保遵循早期阶段开发的 playbook。 执行已测试和训练的步骤。 请勿接收配置和流程中的最新更改。 除此之外，请应用以下度量值：

1. 验证 Azure 门户监视和其他监视工具是否正常工作。  推荐的工具是 Perfmon (Windows) 或 SAR (Linux)： 
    1.  CPU 计数器 
        1.  平均 CPU 时间 - 总计（所有 CPU）
        2.  平均 CPU 时间 - 每单个处理器（m128 VM 上的 128 个处理器）
        3.  CPU 时间内核 - 每单个处理器
        4.  CPU 时间用户 - 每单个处理器
    5.  内存 
        1.  免费内存
        2.  传入的内存页/秒
        3.  传出的内存页/秒
    4.  磁盘 
        1.  磁盘读取 kb/秒 - 每单个磁盘 
        2.  磁盘读取/秒 - 每单个磁盘
        3.  磁盘读取 ms/read - 每单个磁盘
        4.  磁盘写入 kb/秒 - 每单个磁盘 
        5.  磁盘写入/秒 - 每单个磁盘
        6.  磁盘写入 ms/read - 每单个磁盘
    5.  网络 
        1.  传入的网络数据包/秒
        2.  传出的网络数据包/秒
        3.  传入的网络 kb/秒
        4.  传出的网络 kb /秒 
2.  完成数据的迁移后，执行与业务所有者协定的所有验证测试。 仅接受具有原始源系统结果的验证测试结果
3.  检查接口是否可以正常工作以及其他应用程序是否可以与新部署的生产系统进行通信
4.  通过 SAP 事务 STMS 检查传输和更正系统
5.  当发布的系统用于生产后，执行数据库备份
6.  当发布的系统用于生产后，为 SAP 应用层 VM 执行 VM 备份
7.  针对曾不属于当前投入使用阶段但与在此阶段迁移到 Azure 的 SAP 系统进行通信的 SAP 系统，需要在 SM51 中重置主机名缓冲区。 此步骤将删除与迁移到 Azure 中的应用程序实例的名称关联的旧缓存 IP 地址  


## <a name="post-production"></a>后期生产阶段
在此阶段对系统进行监视、操作和管理。 从 SAP 的角度来看，可以使用以前的托管位置执行常见任务。 所要执行的特定于 Azure 的任务：

1. 分析高收费系统的 Azure 账单
2. 优化 VM 端和存储端的性价比
3. 优化可以关闭的时间系统  


## <a name="next-steps"></a>后续步骤
查阅文档：

- [SAP NetWeaver 的 Azure 虚拟机规划和实施指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)
- [SAP NetWeaver 的 Azure 虚拟机部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/deployment-guide)
- [部署适用于 SAP 工作负荷的 Azure 虚拟机 DBMS 的注意事项](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)

