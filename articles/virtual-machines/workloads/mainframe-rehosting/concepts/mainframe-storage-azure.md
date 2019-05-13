---
title: 将大型机存储移动到 Azure 存储
description: 高度可缩放的 Azure 存储资源可帮助基于大型机的组织迁移并更新 IBM z14 应用程序。
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: storage
ms.openlocfilehash: dc78f87d9b47745119da91b8ed1f8f6c8572968c
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190443"
---
# <a name="move-mainframe-storage-to-azure"></a>将大型机存储移动到 Azure

若要在 Microsoft Azure 上运行大型机工作负荷，需要知道与 Azure 大型机的功能有何差别。 高度可缩放的存储资源可帮助组织实现现代化，而不放弃它们依赖于应用程序开始。

Azure 提供了类似于大型机的功能和相当于 IBM z14 基于系统 （在撰写本文时最新的模型） 的存储容量。 本文介绍如何在 Azure 上获得类似结果。

## <a name="mainframe-storage-at-a-glance"></a>一眼大型机存储

IBM 大型机的特点是两种方式存储。 首先，直接访问存储设备 (DASD)。 第二个是顺序存储。 用于管理存储，大型机提供了数据设施存储管理子系统 (DFSMS)。 它管理数据访问的各种存储设备。

[DASD](https://en.wikipedia.org/wiki/Direct-access_storage_device)指的是一个单独的设备的辅助 （不在内存中） 存储，它允许用于直接访问的数据的唯一地址。 最初，术语 DASD 应用到旋转磁盘、 磁性罐、 或数据单元格。 但是，现在一词还可以应用于的固态存储设备 (Ssd)，存储区域网络 (San)、 网络附加存储 (NAS) 和光驱。 对于本文档的目的，DASD 指的是旋转磁盘、 San、 和 Ssd。

与 DASD 存储相比在大型机上的顺序存储是指设备，例如磁带驱动器的起点，从访问，然后读取或写入的行中数据的位置。

存储设备通常使用纤程连接 （光纤连接） 连接或直接在大型机的 IO 总线使用上访问[HiperSockets](https://www.ibm.com/support/knowledgecenter/zosbasics/com.ibm.zos.znetwork/znetwork_85.htm)，一种用于与服务器上的分区之间的高速通信 IBM 技术虚拟机监控程序。

大多数的大型机系统存储分为两种类型：

- *联机存储*（也称为热存储） 所需的日常操作。 DASD 存储通常用于此目的。 但是，顺序存储，例如每天磁带备份 （逻辑或物理），还可以用于此目的。

- *存档存储*（也称为冷存储） 不能保证要装载在给定的时间。 相反，它是装载，并根据需要访问。 存档存储通常被实现使用顺序磁带备份 （逻辑或物理） 进行存储。

## <a name="mainframe-versus-io-latency-and-iops"></a>IO 延迟和 IOPS 与大型机

大型机通常用于需要高性能 IO 和低的 IO 延迟的应用程序。 它们可以执行此操作使用针对 IO 设备和 HiperSockets 光纤连接连接。 当 HiperSockets 用于应用程序和设备直接连接到大型机的 IO 通道时，可以实现以微秒为单位的延迟。

## <a name="azure-storage-at-a-glance"></a>一眼的 azure 存储

Azure 基础结构--服务型 ([IaaS](https://azure.microsoft.com/overview/what-is-iaas/)) 存储选项提供了可比较大型机容量。

Microsoft 提供千万亿字节存储在 Azure 中托管的应用程序的价值，并且你有多个存储选项。 这些涵盖了从高性能的 SSD 存储到的成本较低的 blob 存储以大容量存储和存档。 此外，Azure 提供存储的数据冗余选项-这需要更多工作来在大型机环境中进行设置。

Azure 存储是可用作[Azure 磁盘](/azure/virtual-machines/windows/managed-disks-overview)， [Azure 文件](/azure/storage/files/storage-files-introduction)，并[Azure Blob](/azure/storage/blobs/storage-blobs-overview)根据下表总结了。 详细了解如何[何时使用每个](https://docs.microsoft.com/azure/storage/common/storage-decide-blobs-files-disks)。

<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>Type</th><th>描述</th><th>在下列情况下使用：</th></tr>
</thead>
<tbody>
<tr><td>Azure 文件
</td>
<td>
提供 SMB 接口、 客户端库和一个<a href="https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api">REST</a>接口，允许从任何位置访问存储的文件。
</td>
<td><ul>
<li>提升和转移到云的应用程序时应用程序使用本机文件系统 Api 在 Azure 中运行的其他应用程序之间共享数据。</li>
<li>开发和调试工具，需要从多个 Vm 访问存储中。</li>
</ul>
</td>
</tr>
<tr><td>Azure Blob
</td>
<td>提供客户端库和一个<a href="https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api">REST</a>非结构化数据进行存储和访问块 blob 中大规模的接口。 还支持 <a href="/azure/storage/blobs/data-lake-storage-introduction">Azure Data Lake Storage Gen2</a>，用于企业大数据分析解决方案。
</td>
<td><ul>
<li>应用程序中支持流式处理和随机访问控制方案。</li>
<li>可以从任何位置访问应用程序数据。</li>
<li>在 Azure 上构建企业数据湖，并执行大数据分析。</li>
</ul></td>
</tr>
<tr><td>Azure 磁盘
</td>
<td>提供客户端库和一个<a href="https://docs.microsoft.com/rest/api/compute/disks">REST</a>可以将数据持久存储和附加虚拟硬盘从访问接口。
</td>
<td><ul>
<li>提升和转移使用本机文件系统 Api 来读取和写入到永久性磁盘数据的应用程序。</li>
<li>不需要从磁盘附加到在 VM 外部进行访问的数据存储。</li>
</ul></td>
</tr>
</tbody>
</table>
<!-- markdownlint-enable MD033 -->

## <a name="azure-hot-online-and-cold-archive-storage"></a>Azure （联机） 热和冷 （存档） 存储

存储给定系统的类型取决于系统，包括存储大小、 吞吐量和 IOPS 的要求。 对于 DASD 类型存储在大型机上，在 Azure 上的应用程序都改为通常使用 Azure 磁盘，固态硬盘存储。 对于大型机存档存储，在 Azure 上使用 blob 存储。

SSDs 提供在 Azure 上的最高的存储性能。 （本文档撰写） 提供了以下选项：

| Type         | 大小           | IOPS                  |
|--------------|----------------|-----------------------|
| 超级 SSD    | 4 GB 到 64 TB  | 1200 到 160000 IOPS |
| 高级·SSD  | 32 GB 到 32 TB | 12 到 15,000 名 IOPS     |
| 标准 SSD | 32 GB 到 32 TB | 12 到 2,000 位 IOPS      |

Blob 存储在 Azure 上提供最大存储量。 除存储大小，Azure 还提供托管和非托管的存储。 使用托管存储，Azure 将处理管理的基础的存储帐户。 使用非托管存储时，用户负责的相应大小的 Azure 存储帐户设置以满足存储要求。

## <a name="next-steps"></a>后续步骤

- [大型机迁移](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [大型机重新托管在 Azure 虚拟机](/azure/virtual-machines/workloads/mainframe-rehosting/overview)
- [将大型机计算移动到 Azure](mainframe-compute-Azure.md)
- [确定何时使用 Azure Blob、 Azure 文件或 Azure 磁盘](https://docs.microsoft.com/azure/storage/common/storage-decide-blobs-files-disks)
- [标准 SSD 托管磁盘用于 Azure VM 工作负荷](https://docs.microsoft.com/azure/virtual-machines/windows/disks-standard-ssd)

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
