---
title: 将大型机计算移到 Azure 虚拟机
description: Azure 计算资源来与大型机容量的比较，因此可以迁移并更新 IBM z14 应用程序。
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: multiple
ms.openlocfilehash: 335a056a34412a7ed148613bfff59ecb30053e09
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190319"
---
# <a name="move-mainframe-compute-to-azure"></a>将大型机计算移动到 Azure

大型机具有较高的可靠性和可用性信誉，并继续为许多企业的受信任的主干。 它们通常认为具有几乎无限的可伸缩性和计算能力以及。 但是，有些企业超出最大可用大型机的功能。 如果这听起来像您一样，Azure 提供了灵活性、 范围和基础结构成本节约。

若要在 Microsoft Azure 上运行大型机工作负荷，你需要知道如何大型机的计算与 Azure 的功能比较。 根据 IBM z14 大型机 （撰写本文时最新的模型），本文介绍如何在 Azure 上获得类似结果。

若要开始，请考虑环境并排显示。 下图比较了用于运行应用程序到 Azure 的托管环境的大型机环境。

![Azure 服务和仿真环境提供可比较技术支持和简化迁移](media/mainframe-compute-azure.png)

大型机的强大功能通常用于联机事务处理 (OLTP) 系统处理数以百万计的数千个用户的更新。 这些应用程序通常使用软件进行事务处理、 屏幕处理和窗体项。 它们可能会使用客户信息控制系统 (CICS)、 信息管理系统 (IMS) 或事务界面包 (TIP)。

如图所示，在 Azure 上的 TPM 模拟器可以处理 CICS 和 IMS 工作负荷。 批处理系统仿真程序在 Azure 上的执行的作业控制语言 (JCL) 的角色。 大型机数据迁移到 Azure 数据库，例如 Azure SQL 数据库。 Azure 服务或托管在 Azure 虚拟机中的其他软件可以用于系统管理。

## <a name="mainframe-compute-at-a-glance"></a>一眼大型机计算

Z14 大型机处理器排列中最多四*抽屉*。 一个*抽屉*是只需处理器和芯片集的群集。 每个抽屉中可有六个活动的中央处理器 (CP) 芯片，并且每个 CP 10 系统控制器 (SC) 芯片。 在 Intel x86 术语中，有六个套接字，每个抽屉中，每个插槽，10 个核心和四个抽屉。 此体系结构提供 24 套接字和 240 个内核，最大长度，为 z14 大致相同。

快速 z14 CP 具有 5.2 GHz 的时钟速度。 通常情况下，z14 附带了在框中的所有 CPs。 根据需要它们正在激活。 客户通常收取至少四个小时的计算时间每月尽管实际使用情况。

大型机处理器可以配置为以下类型之一：

- 常规用途 (GP) 处理器
- 系统 z 集成信息处理器 (zIIP)
- Linux (IFL) 处理器的的集成的工具
- 系统帮助处理器 (SAP)
- 集成的耦合设施 (ICF) 处理器

## <a name="scaling-mainframe-compute-up-and-out"></a>缩放大型机计算和横向扩展

IBM 大型机提供扩展达 240 个核心 （适用于单个系统的当前 z14 大小） 的能力。 此外，IBM 大型机可以横向扩展通过名为耦合设施 (CF) 的功能。 CF 允许多个大型机系统同时访问相同的数据。 使用 CF，群集中的大型机并行 Sysplex 技术组大型机处理器。 在撰写本指南时，并行 Sysplex 功能支持 64 位的每个处理器的 32 的分组。 最多 2,048 处理器可以以这种方式横向扩展计算能力进行分组。

CF 允许计算群集来直接访问与共享数据。 用于锁定信息、 缓存信息和共享的数据资源的列表。 使用一个或多个 CFs 并行 Sysplex 可以认为的"共享的所有内容"横向扩展计算群集。 有关这些功能的详细信息，请参阅[IBM Z 上并行 Sysplex](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources) IBM 网站上。

应用程序可以使用这些功能提供向外缩放性能和高可用性。 有关如何 CICS 可使用 CF 使用并行 Sysplex 的信息，请下载[IBM CICS 和耦合设施：基础知识以外](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf)redbook。

## <a name="azure-compute-at-a-glance"></a>一眼的 azure 计算

有些人会错误地将基于 Intel 的服务器不是大型机同样强大。 但是，新的核心密集的、 基于 Intel 的系统必须作为得多的计算容量为大型机。 本部分介绍用于计算和存储的 Azure 基础结构即服务 (IaaS) 选项。 Azure 提供了平台即服务 (PaaS) 选项，但本文重点介绍提供可比较大型机容量的 IaaS 选项。

Azure 虚拟机提供大小和类型的范围中的计算能力。 在 Azure 中，虚拟 CPU (vCPU) 大致相当于在大型机上的核心。

目前，Azure 虚拟机的范围大小提供从 1 到 128 个 Vcpu。 针对特定工作负荷进行了优化的虚拟机 (VM) 类型。 例如，以下列表显示 （当前在撰写本文时） 的 VM 类型和及其建议的用法：

| 大小     | 类型和说明                                                                 |
|----------|--------------------------------------------------------------------------------------|
| D 系列 | 常规用途有 64 vCPU 和最多 3.5 GHz 的时钟速度                           |
| E 系列 | 内存优化具有最多 64 个 Vcpu                                                 |
| F 系列 | 计算优化与最多 64 个 Vcpu，且 3..7 GHz 的时钟速度                       |
| H 系列 | 针对高性能计算 (HPC) 应用程序优化                          |
| L 系列 | 存储优化的高吞吐量应用程序受等 NoSQL 数据库 |
| M 系列 | 最大计算和内存优化的虚拟机具有最多 128 个 Vcpu                        |

有关可用的虚拟机的详细信息，请参阅[虚拟机系列](https://azure.microsoft.com/pricing/details/virtual-machines/series/)。

Z14 大型机可以有多达 240 个内核。 但是，z14 大型机几乎永远不会将所有核心用于单个应用程序或工作负荷。 相反，大型机承受程度分离为逻辑分区 (Lpar) 的工作负荷和 Lpar 具有分级-MIPS （数以百万计的指令 / 秒） 或 MSU （100 万条服务单位）。 当确定 Azure，MIPS （或 MSU） 中的因素级别上运行大型机工作负荷所需的可比较的 VM 大小。

以下是常规的估计值：

-   每个 vCPU 的 150 MIPS

-   每个处理器的 1,000 MIPS

若要确定给定工作负荷中 LPAR 的正确 VM 大小，第一次优化工作负荷的 VM。 然后确定所需的 Vcpu 的数量。 保守的估计是 150 MIPS 每个 vCPU。 根据此估计值，例如，具有 16 Vcpu 的 F 系列 VM 可以轻松地支持来自与 2,400 MIPS LPAR 的 IBM Db2 工作负荷。

## <a name="azure-compute-scale-up"></a>Azure 计算向上扩展

M 系列 Vm 可以扩展 （在撰写本文时的时间） 的最高 128 Vcpu。 M 系列 VM 使用 150 MIPS 的每个 vCPU 保守的估计，等同于大约 19,000 MIPS。 有关大型机估计 MIPS 的一般规则是 1,000 MIPS 每个处理器。 Z14 大型机可以具有最多 24 个处理器，并提供大约为 24000 MIPS 单台大型机系统。

最大单一 z14 大型机具有大约 5,000 MIPS 超过可用在 Azure 中的最大 VM。 还务必要比较的工作负荷的部署方式。 如果大型机系统应用程序和关系数据库，其通常部署在同一个物理主机上，每个在其自己 LPAR。 在 Azure 上的同一解决方案通常被部署应用程序和数据库的单独的、 适当大小的 VM 使用一个 VM。

例如，如果 M64 vCPU 系统支持应用程序，并且 M96 vCPU 用于数据库，则需要大约 150 个 Vcpu，或大约 24000 MIPS 如下图所示。

![比较 24000 MIPS 的工作负荷部署](media/mainframe-compute-mips.png)

方法是将 Lpar 迁移到单独的 Vm。 然后 Azure 轻松地扩展到单台大型机系统上部署的大多数应用程序所需的大小。

## <a name="azure-compute-scale-out"></a>Azure 计算向外扩展

基于 Azure 的优势之一是解决方案的能够向外扩展。几乎无限制缩放使计算可向应用程序的容量。 Azure 支持多种方法来横向扩展计算能力：

- **负载平衡跨群集。** 在此方案中，应用程序可以使用[负载均衡器](/azure/load-balancer/load-balancer-overview)或资源管理器来分散在群集中的多个虚拟机之间的工作负荷。 如果所需容量进行计算的详细信息，更多虚拟机将添加到群集。

- **虚拟机规模集。** 在此迸发方案中，应用程序可以缩放到其他[计算资源](/azure/virtual-machine-scale-sets/overview)根据 VM 使用情况。 如果需求降低，在规模集中的 Vm 数可以也下降，确保有效地使用的计算能力。

- **PaaS 缩放。** Azure PaaS 产品/服务缩放计算资源。 例如， [Azure Service Fabric](/azure/service-fabric/service-fabric-overview)分配计算资源，以满足增加的请求量。

- **Kubernetes 群集。** 在 Azure 上的应用程序可以使用[Kubernetes 群集](/azure/aks/concepts-clusters-workloads)对于指定的资源的计算服务。 Azure Kubernetes 服务 (AKS) 是一种托管的服务，用于协调 Kubernetes 节点、 池和 Azure 上的群集。

若要选择正确的方法来横向扩展计算资源，务必了解 Azure 和大型机有何不同。 关键是如何 — 或者 — 共享数据来计算资源。 在 Azure 中，默认情况下未通常共享数据的多个 Vm。 如果数据共享所需的多个 Vm 中横向扩展计算群集，共享的数据必须位于支持此功能的资源的。 在 Azure 上，数据共享涉及存储，如下面几节讨论。

## <a name="azure-compute-optimization"></a>Azure 计算优化

可优化每个层中的 Azure 体系结构的处理。 为每个环境中使用的虚拟机和功能最适合的类型。 下图显示了用于部署 Azure 以支持使用 Db2 的 CICS 应用程序中的 Vm 的一种潜在模式。 在主站点中，以高可用性部署生产、预生产和测试 VM。 辅助站点用于备份和灾难恢复。

每个层还可以提供适当的灾难恢复服务。 例如，生产和数据库 VM 可能需要热或温恢复，而开发和测试 VM 支持冷恢复。

![支持灾难恢复的高可用性部署](media/mainframe-compute-dr.png)

## <a name="next-steps"></a>后续步骤

- [大型机迁移](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [大型机重新托管在 Azure 虚拟机](/azure/virtual-machines/workloads/mainframe-rehosting/overview)
- [将大型机存储移动到 Azure](mainframe-storage-Azure.md)

### <a name="ibm-resources"></a>IBM 资源

- [并行 Sysplex 上 IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources)
- [IBM CICS 和耦合设施：基础知识以外](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf)
- [Creating required users for a Db2 pureScale Feature installation](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)（创建所需用户以安装 Db2 pureScale 功能）
- [Db2icrt - Create instance command](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)（Db2icrt - 创建实例命令）
- [Db2 pureScale Clustered 数据库解决方案](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1)
- [IBM Data Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

### <a name="azure-government"></a>Azure Government 

- [大型机应用程序的 Microsoft Azure 政府云](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/)
- [Microsoft 和 FedRAMP](https://www.microsoft.com/TrustCenter/Compliance/FedRAMP)

### <a name="more-migration-resources"></a>更多迁移资源

- [平台现代化联盟：在 Azure 上的 IBM Db2](https://www.platformmodernization.org/pages/ibmdb2azure.aspx)
- [Azure Virtual Data Center Lift and Shift Guide](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/)（Azure 虚拟数据中心提升和迁移指南）
- [GlusterFS iSCSI](https://docs.gluster.org/en/latest/Administrator%20Guide/GlusterFS%20iSCSI/)
