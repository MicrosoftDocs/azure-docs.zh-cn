---
title: Azure 订阅限制和配额
description: 提供常见的 Azure 订阅和服务限制、配额和约束的列表。 此文章包括有关如何增加限制以及最大值的信息。
services: multiple
author: rothja
manager: jeffreyg
tags: billing
ms.assetid: 60d848f9-ff26-496e-a5ec-ccf92ad7d125
ms.service: billing
ms.topic: article
ms.date: 04/19/2019
ms.author: byvinyal
ms.openlocfilehash: cb0203e3413ab525d6885f23c5936c64e1bef189
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233701"
---
# <a name="azure-subscription-and-service-limits-quotas-and-constraints"></a>Azure 订阅和服务限制、配额和约束
本文列出了一些最常见的 Microsoft Azure 限制，有时也称为配额。 本文当前并不涵盖所有 Azure 服务。 随着时间推移，将扩展和更新，包含更多服务列表。

若要了解有关 Azure 定价的详细信息，请参阅[Azure 定价概述](https://azure.microsoft.com/pricing/)。 可通过使用评估你的成本[定价计算器](https://azure.microsoft.com/pricing/calculator/)。 您也可以转到特定服务的定价详细信息页等[Windows Vm](https://azure.microsoft.com/pricing/details/virtual-machines/#Windows)。 有关帮助管理成本的提示，请参阅[通过 Azure 计费和成本管理来防止意外成本](billing/billing-getting-started.md)。

> [!NOTE]
> 如果你想要提高限制或配额超出默认限制[打开免费的联机客户支持请求](azure-resource-manager/resource-manager-quota-errors.md)。 限制不能将提高到超过下表中显示的最大限制值。 如果没有最大限制列，则资源没有可调整的限制。
>
> [免费试用版订阅](https://azure.microsoft.com/offers/ms-azr-0044p)不符合增加限制或配额。 如果有[免费试用版订阅](https://azure.microsoft.com/offers/ms-azr-0044p)，可将其升级到[即用即付](https://azure.microsoft.com/offers/ms-azr-0003p/)订阅。 有关详细信息，请参阅[将 Azure 免费试用版订阅升级到即用即付订阅](billing/billing-upgrade-azure-subscription.md)并[免费试用版订阅常见问题解答](https://azure.microsoft.com/free/free-account-faq)。
>

## <a name="limits-and-azure-resource-manager"></a>限制和 Azure 资源管理器
现可以将多个 Azure 资源组合为单个 Azure 资源组。 当使用资源组时，在区域级别使用 Azure 资源管理器管理以前针对全局的限制。 有关 Azure 资源组的详细信息，请参阅[Azure 资源管理器概述](azure-resource-manager/resource-group-overview.md)。

在以下列表中的限制，新表时使用 Azure 资源管理器反映在限制的任何差异。 例如，没有**订阅限制**表和一个**订阅数限制-Azure 资源管理器**表。 当某个限制同时适用于这两种方案时，它仅显示第一个表中。 除非另有说明，否则限制是跨所有区域的全局限制。

> [!NOTE]
> Azure 资源组中的资源配额是每个区域由你的订阅，不根据订阅可访问的服务管理配额一样。 让我们以 vCPU 配额为例。 若要请求增加配额对 Vcpu 的支持，您必须决定你想要在哪些区域中使用多少 Vcpu。 您然后进行 Azure 资源组 vCPU 配额的特定请求的数量和所需的区域。 如果需要使用在西欧的 30 个 Vcpu 那里运行应用程序，则专门请求在西欧的 30 个 Vcpu。 只有西欧具有 30 个 vCPU 配额，不是任何其他区域-增加 vCPU 配额。
> <!-- -->
> 因此，请考虑针对任意区域中的工作负荷来确定所需的 Azure 资源组配额， 然后在需将内容部署到其中的每个区域中请求相应的量。 若要了解如何确定特定区域的当前配额，请参阅[排查部署问题](resource-manager-common-deployment-errors.md)。
>
>

## <a name="service-specific-limits"></a>服务特定的限制
* [Active Directory](#active-directory-limits)
* [API 管理](#api-management-limits)
* [应用服务](#app-service-limits)
* [应用程序网关](#application-gateway-limits)
* [Application Insights](#application-insights-limits)
* [自动化](#automation-limits)
* [用于 Redis 的 Azure 缓存](#azure-cache-for-redis-limits)
* [Azure 云服务](#azure-cloud-services-limits)
* [Azure Cosmos DB](#azure-cosmos-db-limits)
* [Azure Database for MySQL](#azure-database-for-mysql)
* [Azure Database for PostgreSQL](#azure-database-for-postgresql)
* [Azure DNS](#azure-dns-limits)
* [Azure 防火墙](#azure-firewall-limits)
* [Azure Kubernetes 服务](#azure-kubernetes-service-limits)
* [Azure Maps](#azure-maps-limits)
* [Azure Monitor](#monitor-limits)
* [Azure Policy](#azure-policy-limits)
* [Azure 搜索](#azure-search-limits)
* [Azure SignalR 服务](#azure-signalr-service-limits)
* [备份](#backup-limits)
* [批处理](#batch-limits)
* [BizTalk 服务](#biztalk-services-limits)
* [容器实例](#container-instances-limits)
* [容器注册表](#container-registry-limits)
* [内容交付网络](#content-delivery-network-limits)
* [数据工厂](#data-factory-limits)
* [Data Lake Analytics](#data-lake-analytics-limits)
* [Data Lake Store](#data-lake-store-limits)
* [数据库迁移服务](#database-migration-service-limits)
* [事件网格](#event-grid-limits)
* [事件中心](#event-hubs-limits)
* [第一道防线服务](#azure-front-door-service-limits)
* [标识管理器](#identity-manager-limits)
* [IoT 中心](#iot-hub-limits)
* [IoT 中心设备预配服务](#iot-hub-device-provisioning-service-limits)
* [Key Vault](#key-vault-limits)
* [负载均衡器](#load-balancer)
* [Log Analytics](#log-analytics-limits)
* [媒体服务](#media-services-limits)
* [移动服务](#mobile-services-limits)
* [多重身份验证](#multi-factor-authentication-limits)
* [网络](#networking-limits)
* [网络观察程序](#network-watcher-limits)
* [通知中心](#notification-hubs-limits)
* [公共 IP 地址](#publicip-address)
* [资源组](#resource-group-limits)
* [基于角色的访问控制](#role-based-access-control-limits)
* [计划程序](#scheduler-limits)
* [服务总线](#service-bus-limits)
* [Site Recovery](#site-recovery-limits)
* [SQL 数据库](#sql-database-limits)
* [SQL 数据仓库](#sql-data-warehouse-limits)
* [存储](#storage-limits)
* [StorSimple 系统](#storsimple-system-limits)
* [流分析](#stream-analytics-limits)
* [订阅](#subscription-limits)
* [流量管理器](#traffic-manager-limits)
* [虚拟机](#virtual-machines-limits)
* [虚拟机规模集](#virtual-machine-scale-sets-limits)

### <a name="subscription-limits"></a>订阅限制
#### <a name="subscription-limits---azure-service-management-classic-deployment-model"></a>订阅数限制-Azure 服务管理 （经典部署模型）
[!INCLUDE [azure-subscription-limits](../includes/azure-subscription-limits.md)]

#### <a name="subscription-limits---azure-resource-manager"></a>订阅限制 - Azure 资源管理器
使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。 未列出尚未更改与 Azure 资源管理器的限制。 请参阅上的表，了解这些限制。

有关资源管理器 API 读写限制的信息，请参阅[限制资源管理器请求](resource-manager-request-limits.md)。

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../includes/azure-subscription-limits-azure-resource-manager.md)]

### <a name="resource-group-limits"></a>资源组限制
[!INCLUDE [azure-resource-groups-limits](../includes/azure-resource-groups-limits.md)]

### <a name="virtual-machines-limits"></a>虚拟机限制
#### <a name="virtual-machines-limits"></a>虚拟机限制
[!INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]

#### <a name="virtual-machines-limits---azure-resource-manager"></a>虚拟机限制 - Azure 资源管理器
使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。 未列出尚未更改与 Azure 资源管理器的限制。 请参阅上的表，了解这些限制。

[!INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

#### <a name="shared-image-gallery-limits"></a>共享映像库限制

有限制，对于每个订阅，使用共享映像库部署资源：
- 每个订阅，每个区域 100 共享的图像库
- 1,000 图像定义，每个订阅，每个区域
- 每个订阅，每个区域 10,000 映像版本

### <a name="virtual-machine-scale-sets-limits"></a>虚拟机规模集限制
[!INCLUDE [virtual-machine-scale-sets-limits](../includes/azure-virtual-machine-scale-sets-limits.md)]

### <a name="container-instances-limits"></a>容器实例限制
[!INCLUDE [container-instances-limits](../includes/container-instances-limits.md)]

### <a name="container-registry-limits"></a>容器注册表限制
下表详细介绍了“基本”、“标准”和“高级”[服务层级](./container-registry/container-registry-skus.md)的功能和限制。

[!INCLUDE [container-registry-limits](../includes/container-registry-limits.md)]

### <a name="azure-kubernetes-service-limits"></a>Azure Kubernetes 服务限制
[!INCLUDE [container-service-limits](../includes/container-service-limits.md)]

### <a name="networking-limits"></a>网络限制
[!INCLUDE [azure-virtual-network-limits](../includes/azure-virtual-network-limits.md)]

#### <a name="expressroute-limits"></a>ExpressRoute 限制
[!INCLUDE [expressroute-limits](../includes/expressroute-limits.md)]

#### <a name="application-gateway-limits"></a>应用程序网关限制

除非另有说明，否则下表适用于 v1、v2、标准和 WAF SKU。
[!INCLUDE [application-gateway-limits](../includes/application-gateway-limits.md)]

#### <a name="network-watcher-limits"></a>网络观察程序限制
[!INCLUDE [network-watcher-limits](../includes/network-watcher-limits.md)]

#### <a name="traffic-manager-limits"></a>流量管理器限制
[!INCLUDE [traffic-manager-limits](../includes/traffic-manager-limits.md)]

#### <a name="azure-dns-limits"></a>Azure DNS 限制
[!INCLUDE [dns-limits](../includes/dns-limits.md)]

#### <a name="azure-firewall-limits"></a>Azure 防火墙限制
[!INCLUDE [azure-firewall-limits](../includes/firewall-limits.md)]

#### <a name="azure-front-door-service-limits"></a>Azure Front Door 服务限制
[!INCLUDE [azure-front-door-service-limits](../includes/front-door-limits.md)]

### <a name="storage-limits"></a>存储限制
<!--like # storage accts -->
[!INCLUDE [azure-storage-limits](../includes/azure-storage-limits.md)]

有关存储帐户限制的详细信息，请参阅[Azure 存储可伸缩性和性能目标](storage/common/storage-scalability-targets.md)。

#### <a name="storage-resource-provider-limits"></a>存储资源提供程序限制 

[!INCLUDE [azure-storage-limits-azure-resource-manager](../includes/azure-storage-limits-azure-resource-manager.md)]

#### <a name="azure-blob-storage-limits"></a>Azure Blob 存储限制
[!INCLUDE [storage-blob-scale-targets](../includes/storage-blob-scale-targets.md)]

#### <a name="azure-files-limits"></a>Azure 文件限制
有关 Azure 文件限制的详细信息，请参阅[Azure 文件可伸缩性和性能目标](storage/files/storage-files-scale-targets.md)。

[!INCLUDE [storage-files-scale-targets](../includes/storage-files-scale-targets.md)]

#### <a name="azure-file-sync-limits"></a>Azure 文件同步限制
[!INCLUDE [storage-sync-files-scale-targets](../includes/storage-sync-files-scale-targets.md)]

#### <a name="azure-queue-storage-limits"></a>Azure 队列存储限制
[!INCLUDE [storage-queues-scale-targets](../includes/storage-queues-scale-targets.md)]

#### <a name="azure-table-storage-limits"></a>Azure 表存储限制
[!INCLUDE [storage-tables-scale-targets](../includes/storage-tables-scale-targets.md)]

<!-- conceptual info about disk limits -- applies to unmanaged and managed -->
#### <a name="virtual-machine-disk-limits"></a>虚拟机磁盘限制
[!INCLUDE [azure-storage-limits-vm-disks](../includes/azure-storage-limits-vm-disks.md)]

有关详细信息，请参阅[虚拟机大小](virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

#### <a name="managed-virtual-machine-disks"></a>托管虚拟机磁盘

[!INCLUDE [azure-storage-limits-vm-disks-managed](../includes/azure-storage-limits-vm-disks-managed.md)]

#### <a name="unmanaged-virtual-machine-disks"></a>非托管虚拟机磁盘

[!INCLUDE [azure-storage-limits-vm-disks-standard](../includes/azure-storage-limits-vm-disks-standard.md)]

[!INCLUDE [azure-storage-limits-vm-disks-premium](../includes/azure-storage-limits-vm-disks-premium.md)]

### <a name="azure-cloud-services-limits"></a>Azure 云服务限制
[!INCLUDE [azure-cloud-services-limits](../includes/azure-cloud-services-limits.md)]

### <a name="app-service-limits"></a>应用服务限制
以下应用服务限制包括 Web 应用、移动应用、API 应用的限制。

[!INCLUDE [azure-websites-limits](../includes/azure-websites-limits.md)]

### <a name="scheduler-limits"></a>计划程序限制
[!INCLUDE [scheduler-limits-table](../includes/scheduler-limits-table.md)]

### <a name="batch-limits"></a>Batch 限制
[!INCLUDE [azure-batch-limits](../includes/azure-batch-limits.md)]

### <a name="biztalk-services-limits"></a>BizTalk 服务限制
下表显示了 Azure BizTalk 服务的限制。

[!INCLUDE [biztalk-services-service-limits](../includes/biztalk-services-service-limits.md)]

### <a name="azure-cosmos-db-limits"></a>Azure Cosmos DB 限制
Azure Cosmos DB 是全局缩放数据库，可对吞吐量和存储进行缩放，以处理应用程序的任何需求。 如果你有 Azure Cosmos DB 提供的规模有任何疑问，发送电子邮件发送到askcosmosdb@microsoft.com。

### <a name="azure-database-for-mysql"></a>Azure Database for MySQL
有关 Azure Database for MySQL 限制，请参阅 [Azure Database for MySQL 中的限制](mysql/concepts-limits.md)。

### <a name="azure-database-for-postgresql"></a>Azure Database for PostgreSQL
有关 Azure Database for PostgreSQL 限制，请参阅 [Azure Database for PostgreSQL 中的限制](postgresql/concepts-limits.md)。

### <a name="azure-search-limits"></a>Azure 搜索限制
定价层决定了搜索服务的容量和限制。 层包括：

* **免费**多租户服务，与其他 Azure 订户共享仅用于评估和小型开发项目。
* 基本层为规模较小的生产工作负荷提供专用计算资源，并为高可用查询工作负荷提供最多 3 个副本。
* **标准**，其中包括 S1、 S2、 S3 和 S3 High Density 为更大的生产工作负荷。 以便你可以选择最符合您工作负荷的配置文件的资源配置的标准层中有多个级别。

**基于订阅的限制**

[!INCLUDE [azure-search-limits-per-subscription](../includes/azure-search-limits-per-subscription.md)]

**基于搜索服务的限制**

[!INCLUDE [azure-search-limits-per-service](../includes/azure-search-limits-per-service.md)]

若要更详细地了解关于限制的详细信息（如文档大小、每秒查询数、密钥、请求和响应），请参阅 [Azure 搜索中的服务限制](search/search-limits-quotas-capacity.md)。

### <a name="media-services-limits"></a>媒体服务限制
[!INCLUDE [azure-mediaservices-limits](../includes/azure-mediaservices-limits.md)]

### <a name="content-delivery-network-limits"></a>内容交付网络限制
[!INCLUDE [cdn-limits](../includes/cdn-limits.md)]

### <a name="mobile-services-limits"></a>移动服务限制
[!INCLUDE [mobile-services-limits](../includes/mobile-services-limits.md)]

### <a name="monitor-limits"></a>监视限制
[!INCLUDE [monitoring-limits](../includes/monitoring-limits.md)]

### <a name="notification-hubs-limits"></a>通知中心限制
[!INCLUDE [notification-hub-limits](../includes/notification-hub-limits.md)]

### <a name="event-hubs-limits"></a>事件中心限制
[!INCLUDE [azure-servicebus-limits](../includes/event-hubs-limits.md)]

### <a name="service-bus-limits"></a>服务总线限制
[!INCLUDE [azure-servicebus-limits](../includes/service-bus-quotas-table.md)]

### <a name="iot-hub-limits"></a>IoT 中心限制
[!INCLUDE [azure-iothub-limits](../includes/iot-hub-limits.md)]

### <a name="iot-hub-device-provisioning-service-limits"></a>IoT 中心设备预配服务限制
[!INCLUDE [azure-iotdps-limits](../includes/iot-dps-limits.md)]

### <a name="data-factory-limits"></a>数据工厂限制
[!INCLUDE [azure-data-factory-limits](../includes/azure-data-factory-limits.md)]

### <a name="data-lake-analytics-limits"></a>Data Lake Analytics 限制
[!INCLUDE [azure-data-lake-analytics-limits](../includes/azure-data-lake-analytics-limits.md)]

### <a name="data-lake-store-limits"></a>Data Lake Store 限制
[!INCLUDE [azure-data-lake-store-limits](../includes/azure-data-lake-store-limits.md)]

### <a name="database-migration-service-limits"></a>数据库迁移服务限制
[!INCLUDE [database-migration-service-limits](../includes/database-migration-service-limits.md)]

### <a name="stream-analytics-limits"></a>流分析限制
[!INCLUDE [stream-analytics-limits-table](../includes/stream-analytics-limits-table.md)]

### <a name="active-directory-limits"></a>Active Directory 限制
[!INCLUDE [AAD-service-limits](../includes/active-directory-service-limits-include.md)]

### <a name="event-grid-limits"></a>事件网格限制
[!INCLUDE [event-grid-limits](../includes/event-grid-limits.md)]

### <a name="azure-maps-limits"></a>Azure Maps 限制
[!INCLUDE [maps-limits](../includes/maps-limits.md)]

### <a name="azure-policy-limits"></a>Azure Policy 限制
[!INCLUDE [policy-limits](../includes/azure-policy-limits.md)]

### <a name="storsimple-system-limits"></a>StorSimple 系统限制
[!INCLUDE [storsimple-limits-table](../includes/storsimple-limits-table.md)]

### <a name="log-analytics-limits"></a>Log Analytics 限制
[!INCLUDE [operational-insights-limits](../includes/operational-insights-limits.md)]

### <a name="backup-limits"></a>备份限制
[!INCLUDE [azure-backup-limits](../includes/azure-backup-limits.md)]

### <a name="azure-signalr-service-limits"></a>Azure SignalR 服务限制
[!INCLUDE [signalr-service-limits](../includes/signalr-service-limits.md)]

### <a name="site-recovery-limits"></a>站点恢复限制
[!INCLUDE [site-recovery-limits](../includes/site-recovery-limits.md)]

### <a name="application-insights-limits"></a>Application Insights 限制
[!INCLUDE [application-insights-limits](../includes/application-insights-limits.md)]

### <a name="api-management-limits"></a>API 管理限制
[!INCLUDE [api-management-service-limits](../includes/api-management-service-limits.md)]

### <a name="azure-cache-for-redis-limits"></a>Azure Redis 缓存限制
[!INCLUDE [redis-cache-service-limits](../includes/redis-cache-service-limits.md)]

### <a name="key-vault-limits"></a>密钥保管库限制
[!INCLUDE [key-vault-limits](../includes/key-vault-limits.md)]

### <a name="multi-factor-authentication-limits"></a>多重身份验证限制
[!INCLUDE [azure-mfa-service-limits](../includes/azure-mfa-service-limits.md)]

### <a name="automation-limits"></a>自动化限制
[!INCLUDE [automation-limits](../includes/azure-automation-service-limits.md)]

### <a name="identity-manager-limits"></a>标识管理器限制
[!INCLUDE [automation-limits](~/includes/managed-identity-limits.md)]

### <a name="role-based-access-control-limits"></a>基于角色的访问控制限制
[!INCLUDE [role-based-access-control-limits](../includes/role-based-access-control-limits.md)]

### <a name="sql-database-limits"></a>SQL 数据库限制
有关 SQL 数据库限制，请参阅[单一数据库的 SQL 数据库资源限制](sql-database/sql-database-vcore-resource-limits-single-databases.md)，[弹性池和入池的数据库的 SQL 数据库资源限制](sql-database/sql-database-vcore-resource-limits-elastic-pools.md)，和[SQL 数据库资源限制托管实例](sql-database/sql-database-managed-instance-resource-limits.md)。

### <a name="sql-data-warehouse-limits"></a>SQL 数据仓库限制
有关 SQL 数据仓库限制，请参阅[SQL 数据仓库资源限制](sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md)。

## <a name="see-also"></a>另请参阅
- [了解 Azure 限制和增加](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
- [Azure 的虚拟机和云服务大小](virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Azure 云服务的大小](cloud-services/cloud-services-sizes-specs.md)
