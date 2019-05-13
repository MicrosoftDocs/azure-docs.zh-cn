---
title: SAP NetWeaver 的 Azure 虚拟机高可用性 | Microsoft 文档
description: Azure 虚拟机上的 SAP NetWeaver 高可用性指南
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: eb7919c6f4ff1b3cf2480333273a98f2cca9a223
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204943"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver"></a>针对 SAP NetWeaver 的 Azure 虚拟机高可用性

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (SAP 多 SID 高可用性配置)


[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/resource-group-overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md



Azure 虚拟机是面向在最短时间内需要计算、存储和网络资源的组织的解决方案，没有冗长的采购周期。 可以使用 Azure 虚拟机来部署经典应用程序，例如，基于 SAP NetWeaver 的 ABAP、Java 和 ABAP+Java 堆栈。 无需添置本地资源即可提高可靠性和可用性。 Azure 虚拟机支持跨界连接，因此可将 Azure 虚拟机集成到组织的本地域、私有云和 SAP 系统布局中。

本文介绍使用 Azure Resource Manager 部署模型在 Azure 中部署高可用性 SAP系统时可以采取的步骤。 本文逐步讲解如何完成以下主要任务：

* 查找适当的 SAP 说明和安装指南：已在 [资源][sap-ha-guide-2] 部分中列出。 本文对 SAP 安装文档和 SAP 说明（帮助在特定平台上安装和部署 SAP 软件的主要资源）作了补充。
* 了解 Azure Resource Manager 部署模型与 Azure 经典部署模型之间的差异。
* 了解 Windows Server 故障转移群集仲裁模式，以选择适用于 Azure 部署的模型。
* 了解 Azure 服务中的 Windows Server 故障转移群集共享存储。
* 了解如何在 Azure 中保护单一故障点组件（例如高级商业应用程序编程 (ABAP) SAP Central Services (ASCS)/SAP Central Services (SCS) 和数据库管理系统 (DBMS)）以及冗余组件（例如 SAP 应用程序服务器）。
* 遵循一个循序渐进的示例，使用 Azure 资源管理器在 Windows Server 故障转移群集中安装和配置高可用性 SAP 系统。
* 详细了解在 Azure 中使用 Windows Server 故障转移群集所需的步骤，但在本地部署中无需执行这些步骤。

为了简化部署和配置，本文使用 SAP 三层高可用性 Resource Manager 模板。 该模板会自动部署高可用性 SAP 系统所需的整个基础结构。 该基础结构还支持 SAP 系统的 SAP 应用程序性能标准 (SAPS) 大小调整。

## <a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a>先决条件
在开始之前，请确保符合以下部分中所述的先决条件。 此外，请务必查看 [资源][sap-ha-guide-2] 部分中列出的所有资源。

在本文中，为[使用托管磁盘的三层 SAP NetWeaver](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md/) 使用资源管理器模板。 有关模板的有用概述，请参阅 [SAP Azure Resource Manager 模板](https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/16/azure-quickstart-templates-for-sap/)。

## <a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a> 资源
这些文章介绍 Azure 中的 SAP 部署：

* [针对 SAP NetWeaver 的 Azure 虚拟机规划和实施][planning-guide]
* [针对 SAP NetWeaver 的 Azure 虚拟机部署][deployment-guide]
* [SAP NetWeaver 的 Azure 虚拟机 DBMS 部署][dbms-guide]
* [SAP NetWeaver 的 Azure 虚拟机高可用性（本指南）][sap-ha-guide]

> [!NOTE]
> 在可能的情况下，本文中会提供 SAP 安装指南的参考链接（请参阅 [SAP 安装指南][sap-installation-guides]）。 如需了解先决条件和安装过程的信息，仔细阅读 SAP NetWeaver 安装指南是一个好办法。 本文仅介绍可在 Azure 虚拟机上针对基于 SAP NetWeaver 的系统执行的特定任务。
>
>

以下 SAP 说明与 Azure 中的 SAP 主题相关：

| 说明文档编号 | 标题 |
| --- | --- |
| [1928533] |Azure 上的 SAP 应用程序：支持的产品和大小 |
| [2015553] |Microsoft Azure 上的 SAP：支持先决条件 |
| [1999351] |适用于 SAP 的增强型 Azure 监视 |
| [2178632] |Microsoft Azure 上的 SAP 关键监控指标 |
| [1999351] |Windows 上的虚拟化：增强型监视 |
| [2243692] |为 SAP DBMS 实例使用 Azure 高级 SSD 存储 |

详细了解 [Azure 订阅的限制][azure-subscription-service-limits-subscription]，包括常规默认限制和最大限制。

## <a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>Azure Resource Manager 与 Azure 经典部署模型的高可用性 SAP
Azure Resource Manager 与 Azure 经典部署模型的不同之处体现在以下方面：

- 资源组
- Azure 内部负载均衡器与 Azure 资源组的依赖关系
- 对 SAP 多 SID 方案的支持

### <a name="f76af273-1993-4d83-b12d-65deeae23686"></a> 资源组
在 Azure Resource Manager 中，可以使用资源组来管理 Azure 订阅中的所有应用程序资源。 使用集成方法时，资源组中的所有资源具有相同的生命周期。 例如，所有资源同时创建并同时删除。 了解有关 [资源组](../../../azure-resource-manager/resource-group-overview.md#resource-groups)的详细信息。

### <a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a>Azure 内部负载均衡器与 Azure 资源组的依赖关系

在 Azure 经典部署模型中，Azure 内部负载均衡器（Azure 负载均衡器服务）与云服务之间存在依赖关系。 每个内部负载均衡器都需要一个云服务。

在 Azure 资源管理器中，需将每个 Azure 资源放入一个 Azure 资源组，这同样适用于 Azure 负载均衡器。 但是，不需要每个 Azure 负载均衡器都对应 1 个 Azure 资源组，例如 1 个 Azure 资源组可以包含多个 Azure 负载均衡器。 环境更简单、更灵活。 

### <a name="support-for-sap-multi-sid-scenarios"></a>SAP 多 SID 方案支持

在 Azure Resource Manager 中，可以在一个群集中安装多个 SAP 系统标识符 (SID) ASCS/SCS 实例。 之所以能够使用多 SID 实例，是因为支持对每个 Azure 负载均衡器使用多个 IP 地址。

若要使用 Azure 经典部署模型，请遵循 [Azure 中的 SAP NetWeaver：配合 SIOS DataKeeper 使用 Azure 中的 Windows Server 故障转移群集来组建 SAP ASCS/SCS 实例的群集](https://go.microsoft.com/fwlink/?LinkId=613056)中所述的过程。

> [!IMPORTANT]
> 强烈建议针对 SAP 安装使用 Azure Resource Manager 部署模型。 它提供经典部署模型所不具备的多种优势。 了解有关 Azure [部署模型][virtual-machines-azure-resource-manager-architecture-benefits-arm]的详细信息。   
>
>

## <a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a> Windows Server 故障转移群集
Windows Server 故障转移群集是 Windows 中高可用性 SAP ASCS/SCS 安装和 DBMS 的基础。

故障转移群集是由 1+n 个独立服务器（节点）构成的组，这些服务器配合工作以提高应用程序和服务的可用性。 如果发生节点故障，Windows Server 故障转移群集会计算发生的故障次数并维护状况良好的群集，以提供应用程序和服务。 可从不同的仲裁模式中进行选择，以实现故障转移群集。

### <a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a> 仲裁模式
使用 Windows Server 故障转移群集时，可从四种仲裁模式中选择：

* **节点多数**。 群集的每个节点都可投票。 只有获取多数票（也就是票数过半）时，群集才能正常运行。 建议针对节点数目为奇数的群集使用此选项。 例如，如果包含七个节点的群集中有三个节点发生故障，群集仍可继续运行，因为多数节点正常。  
* **节点和磁盘多数**。 每个节点和群集存储中的指定磁盘（磁盘见证）在处于可用和通信中状态时均可投票。 只有获取多数票（也就是票数过半）时，群集才能正常运行。 此模式适用于节点数目为偶数的群集环境。 如果半数节点和该磁盘在线，群集就保持正常运行状态。
* **节点和文件共享多数**。 每个节点加上管理员创建的指定文件共享（文件共享见证）均可投票，无论节点和文件共享是否可用且处于通信中状态，都是如此。 只有获取多数票（也就是票数过半）时，群集才能正常运行。 此模式适用于节点数目为偶数的群集环境。 类似于“节点与磁盘多数”模式，但它使用见证文件共享而不是见证磁盘。 此模式很容易实现，但如果文件共享本身不具有高可用性，则它可能变成单一故障点。
* **无多数：仅磁盘**。 只要有一个节点可用且正与群集存储中的特定磁盘进行通信，群集就拥有仲裁。 只有也在与该磁盘进行通信的节点能够加入群集。 建议不要使用此模式。
 

## <a name="fdfee875-6e66-483a-a343-14bbaee33275"></a> 本地 Windows Server 故障转移群集
图 1 显示的群集有两个节点。 如果节点之间的网络连接失败，并且两个节点仍保持运行，仲裁磁盘或文件共享将确定哪个节点将继续提供群集的应用程序与服务。 有权访问仲裁磁盘或文件共享的节点就是可以确保服务可继续的节点。

由于本示例使用双节点群集，因此我们使用“节点与文件共享多数”仲裁模式。 “节点与磁盘多数”也是有效的选项。 在生产环境中，建议使用仲裁磁盘。 可以使用网络和存储系统技术使其实现高可用性。

![图 1：Azure 中 SAP ASCS/SCS 的 Windows Server 故障转移群集配置示例][sap-ha-guide-figure-1000]

_**图 1：** Azure 中 SAP ASCS/SCS 的 Windows Server 故障转移群集配置示例_

### <a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a> 共享存储
图 1 还显示了包含两个节点的共享存储群集。 在本地共享存储群集中，群集中的所有节点都检测到共享存储。 锁定机制可防止数据损坏。 所有节点都可以检测另一个节点是否发生故障。 如果一个节点发生故障，剩余的节点会取得存储资源的所有权，确保服务的可用性。

> [!NOTE]
> 对于某些 DBMS 应用程序（例如 SQL Server），无需使用共享磁盘即可实现高可用性。 SQL Server Always On 将 DBMS 数据和日志从一个群集节点的本地磁盘复制到另一个群集节点的本地磁盘。 在此情况下，Windows 群集配置不需要共享磁盘。
>
>

### <a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a> 网络和名称解析
客户端计算机可通过 DNS 服务器提供的虚拟 IP 地址和虚拟主机名来访问群集。 本地节点与 DNS 服务器可以处理多个 IP 地址。

在典型设置中，可使用两个或更多个网络连接：

* 与存储建立的专用连接
* 用于检测信号的群集内部网络连接
* 供客户端用来连接到群集的公共网络

## <a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a> Azure 中的 Windows Server 故障转移群集
相比于裸机或私有云部署，Azure 虚拟机要求执行额外的步骤来配置 Windows Server 故障转移群集。 构建共享群集磁盘时，需要为 SAP ASCS/SCS 实例设置多个 IP 地址和虚拟主机名。

本文介绍在 Azure 中构建 SAP 高可用性中心服务群集时所要了解的重要概念与其他步骤。 文中会说明如何设置第三方工具 SIOS DataKeeper，以及配置 Azure 内部负载均衡器。 可以使用这些工具在 Azure 中通过文件共享见证创建 Windows 故障转移群集。

![图 2：Azure 中未使用共享磁盘的 Windows Server 故障转移群集配置][sap-ha-guide-figure-1001]

_**图 2：** Azure 中未使用共享磁盘的 Windows Server 故障转移群集配置_

### <a name="1a464091-922b-48d7-9d08-7cecf757f341"></a> Azure 中使用 SIOS DataKeeper 的共享磁盘
需要为高可用性 SAP ASCS/SCS 实例使用群集共享存储。 从 2016 年 9 月开始，Azure 不再提供可用于创建共享存储群集的共享存储。 可以使用第三方软件 SIOS DataKeeper Cluster Edition 创建镜像存储，模拟群集共享存储。 SIOS 解决方案提供实时同步的数据复制。 为群集创建共享磁盘资源的方法包括：

1. 将额外磁盘附加到 Windows 群集配置中的每个虚拟机 (VM)。
2. 在两个虚拟机节点上运行 SIOS DataKeeper Cluster Edition。
3. 配置 SIOS DataKeeper Cluster Edition，以便它将额外磁盘附加卷的内容从源虚拟机映射到目标虚拟机的额外磁盘附加卷。 SIOS DataKeeper 抽象化源和目标本地卷，并以单个共享磁盘形式呈现给 Windows Server 故障转移群集。

获取有关 [SIOS DataKeeper](https://us.sios.com/products/datakeeper-cluster/)的详细信息。

![图 3：Azure 中使用 SIOS DataKeeper 的 Windows Server 故障转移群集配置][sap-ha-guide-figure-1002]

_**图 3：** Azure 中使用 SIOS DataKeeper 的 Windows Server 故障转移群集配置_

> [!NOTE]
> 对于某些 DBMS 产品（如 SQL Server），无需共享磁盘来实现高可用性。 SQL Server Always On 将 DBMS 数据和日志从一个群集节点的本地磁盘复制到另一个群集节点的本地磁盘。 在这种情况下，Windows 群集配置不需要共享磁盘。
>
>

### <a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a> Azure 中的名称解析
Azure 云平台不提供配置虚拟 IP 地址（例如浮动 IP 地址）的选项。 需要使用一种替代解决方案来设置虚拟 IP 地址，用于访问云中的群集资源。
Azure 在 Azure 负载均衡器服务中提供一个内部负载均衡器。 使用内部负载均衡器，客户端可以通过群集虚拟 IP 地址访问群集。
需要在包含群集节点的资源组中部署内部负载均衡器。 然后，使用内部负载均衡器的探测端口来配置所有必要的端口转发规则。
客户端可以通过虚拟主机名进行连接。 DNS 服务器解析群集 IP 地址，内部负载均衡器处理向活动群集节点的端口转发。

## <a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a>Azure 基础结构即服务 (IaaS) 中的 SAP NetWeaver 高可用性
若要实现 SAP 应用程序高可用性（例如为 SAP 软件组件），需要保护以下组件：

* SAP 应用程序服务器实例
* SAP ASCS/SCS 实例
* DBMS 服务器

有关在高可用性方案中保护 SAP 组件的详细信息，请参阅 [针对 SAP NetWeaver 的 Azure 虚拟机规划和实施][planning-guide-11]。

### <a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a> 高可用性 SAP 应用程序服务器
对于 SAP 应用程序服务器和对话实例，通常不需要使用特定的高可用性解决方案。 可以通过冗余实现高可用性，并在 Azure 虚拟机的不同实例中配置多个对话实例。 至少应该将两个 SAP 应用程序实例安装在 Azure 虚拟机的两个实例中。

![图 4：高可用性 SAP 应用程序服务器][sap-ha-guide-figure-2000]

_**图 4：** 高可用性 SAP 应用程序服务器_

必须将所有托管 SAP 应用程序服务器实例的虚拟机放置在同一个 Azure 可用性集中。 Azure 可用性集可确保：

* 所有虚拟机都是相同升级域的一部分。 例如，升级域可确保虚拟机不会在计划内维护停机期间同时更新。
* 所有虚拟机都是相同容错域的一部分。 例如，容错域可确保部署虚拟机，使得任何单一故障点不会影响所有虚拟机的可用性。

详细了解如何 [管理虚拟机的可用性][virtual-machines-manage-availability]。

仅限非托管磁盘：由于 Azure 存储帐户是潜在的单一故障点，因此务必至少拥有两个 Azure 存储帐户，并且至少要将两个虚拟机分配到其中。 在理想的设置中，运行 SAP 对话实例的每个虚拟机部署在不同的存储帐户中。

### <a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a> 高可用性 SAP ASCS/SCS 实例
图 5 显示高可用性 SAP ASCS/SCS 实例的示例。

![图 5：高可用性 SAP ASCS/SCS 实例][sap-ha-guide-figure-2001]

_**图 5：** 高可用性 SAP ASCS/SCS 实例_

#### <a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a>在 Azure 中使用 Windows Server 故障转移群集实现 SAP ASCS/SCS 实例高可用性
相比于裸机或私有云部署，Azure 虚拟机要求执行额外的步骤来配置 Windows Server 故障转移群集。 若要构建 Windows 故障转移群集，需要使用一个共享群集磁盘、多个 IP 地址、多个虚拟主机名，以及 Azure 内部负载均衡器，组建 SAP ASCS/SCS 实例的群集。 本文稍后将作详细介绍。

![图 6：Azure 中使用 SIOS DataKeeper 的 SAP ASCS/SCS 配置的 Windows Server 故障转移群集配置][sap-ha-guide-figure-1002]

_**图 6：** Azure 中使用 SIOS DataKeeper 的 SAP ASCS/SCS 配置的 Windows Server 故障转移群集_

### <a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a>高可用性 DBMS 实例
DBMS 也是 SAP 系统中的单一联系点。 需要使用高可用性解决方案来保护它。 图 7 显示在 Azure 中使用 Windows Server 故障转移群集和 Azure 内部负载均衡器的 SQL Server AlwaysOn 高可用性解决方案。 SQL Server Always On 使用自身的 DBMS 复制功能复制 DBMS 数据和日志文件。 在此情况下，无需使用群集共享磁盘，从而简化了整个设置。

![图 7：使用 SQL Server Always On 的高可用性 SAP DBMS 示例][sap-ha-guide-figure-2003]

_**图 7：** 使用 SQL Server Always On 的高可用性 SAP DBMS 示例_

有关使用 Azure Resource Manager 部署模型在 Azure 中将 SQL Server 组建成群集的详细信息，请参阅以下文章：

* [使用 Resource Manager 在 Azure 虚拟机中手动配置 AlwaysOn 可用性组][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]
* [在 Azure 中为 AlwaysOn 可用性组配置 Azure 内部负载均衡器][virtual-machines-windows-portal-sql-alwayson-int-listener]

## <a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a>端到端高可用性部署方案

### <a name="deployment-scenario-using-architectural-template-1"></a>使用体系结构模板 1 的部署方案

图 8 显示 Azure 中 **一个** SAP 系统的 SAP NetWeaver 高可用性体系结构示例。 此方案设置如下：

- 一个专用群集用于 SAP ASCS/SCS 实例。
- 一个专用群集用于 DBMS 实例。
- SAP 应用程序服务器实例部署在其自身的专用 VM 中。

![图 8：SAP 高可用性体系结构模板 1：包含 ASCS/SCS 和 DBMS 实例的专用群集][sap-ha-guide-figure-2004]

_**图 8：** SAP 高可用性体系结构模板 1：包含 ASCS/SCS 和 DBMS 实例的专用群集_

### <a name="deployment-scenario-using-architectural-template-2"></a>使用体系结构模板 2 的部署方案

图 9 显示 Azure 中 **一个** SAP 系统的 SAP NetWeaver 高可用性体系结构示例。 此方案设置如下：

- 一个专用群集 **同时** 用于 SAP ASCS/SCS 实例和 DBMS。
- SAP 应用程序服务器实例部署在其自身的专用 VM 中。

![图 9：SAP 高可用性体系结构模板 2：包含 ASCS/SCS 的专用群集和 DBMS 的专用群集][sap-ha-guide-figure-2005]

_**图 9：** SAP 高可用性体系结构模板 2：包含 ASCS/SCS 的专用群集和 DBMS 的专用群集_

### <a name="deployment-scenario-using-architectural-template-3"></a>使用体系结构模板 3 的部署方案

图 10 显示了 Azure 中**两个** SAP 系统（分别带有标记 &lt;SID1&gt; 和 &lt;SID2&gt;）的 SAP NetWeaver 高可用性体系结构示例。 此方案的设置方式如下：

- 一个专用群集**同时**用于 SAP ASCS/SCS SID1 实例*和* SAP ASCS/SCS SID2 实例（一个群集）。
- 一个专用群集用于 DBMS SID1，另一个专用群集用于 DBMS SID2（两个群集）。
- SAP 系统 SID1 的 SAP 应用程序服务器实例有其自身的专用 VM。
- SAP 系统 SID2 的 SAP 应用程序服务器实例具有自身的专用 VM。

![图 10：SAP 高可用性体系结构模板 3：为不同的 ASCS/SCS 实例使用一个专用群集][sap-ha-guide-figure-6003]

_**图 10：** SAP 高可用性体系结构模板 3：为不同的 ASCS/SCS 实例使用一个专用群集_

## <a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a> 准备基础结构

### <a name="prepare-the-infrastructure-for-architectural-template-1"></a>为体系结构模板 1 准备基础结构
适用于 SAP 的 Azure Resource Manager 模板有助于简化部署所需的资源。

Azure Resource Manager 中的三层模板还支持高可用性方案，例如体系结构模板 1 包含两个群集。 每个群集是 SAP ASCS/SCS 和 DBMS 的 SAP 单一故障点。

可在以下位置获取本文介绍的示例方案的 Azure Resource Manager 模板：

* [Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image)  
* [使用托管磁盘的 Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image)
* [使用托管磁盘的自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

为体系结构模板 1 准备基础结构：

- 在 Azure 门户中的“参数”边栏选项卡上，从“SYSTEMAVAILABILITY”框中选择“HA”。

  ![图 11：设置 SAP 高可用性 Azure 资源管理器参数][sap-ha-guide-figure-3000]

_**图 11：** 设置 SAP 高可用性 Azure 资源管理器参数_


  模板会创建：

  * **虚拟机**：
    * SAP Application Server 虚拟机：<*SAPSystemSID*>-di-<*Number*>
    * ASCS/SCS 群集虚拟机：<*SAPSystemSID*>-ascs-<*Number*>
    * DBMS 群集：<*SAPSystemSID*>-db-<*Number*>

  * **所有虚拟机的网卡以及关联的 IP 地址**：
    * <*SAPSystemSID*>-nic-di-<*Number*>
    * <*SAPSystemSID*>-nic-ascs-<*Number*>
    * <SAPSystemSID>-nic-db-<Number>

  * Azure 存储帐户（仅限非托管磁盘）

  * 以下组件的**可用性组**：
    * SAP Application Server 虚拟机：<*SAPSystemSID*>-avset-di
    * SAP ASCS/SCS 群集虚拟机：<*SAPSystemSID*>-avset-ascs
    * DBMS 群集虚拟机：<*SAPSystemSID*>-avset-db

  * **Azure 内部负载均衡器**：
    * 带有 ASCS/SCS instance 的所有端口和 IP 地址 <*SAPSystemSID*>-lb-ascs
    * 带有 SQL Server DBMS 的所有端口和 IP 地址 <*SAPSystemSID*>-lb-db

  * **网络安全组**：<*SAPSystemSID*>-nsg-ascs-0  
    * 带有 <*SAPSystemSID*>-ascs-0 虚拟机的开放外部远程桌面协议 (RDP) 端口

> [!NOTE]
> 默认情况下，网卡和 Azure 内部负载均衡器的所有 IP 地址均为“动态”。 请将其更改为 **静态** IP 地址。 本文稍后介绍操作方法。
>
>

### <a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> 部署带有公司网络连接的虚拟机（跨场地）以在生产中使用
对于生产 SAP 系统，使用 Azure 站点到站点 (VPN) 或 Azure ExpressRoute 部署具有[企业网络连接（跨界）][planning-guide-2.2]的 Azure 虚拟机。

> [!NOTE]
> 可使用 Azure 虚拟网络实例。 已创建并准备好虚拟网络与子网。
>
>

1. 在 Azure 门户中的“参数”边栏选项卡上，从“NEWOREXISTINGSUBNET”框中选择“现有”。
2. 在“SUBNETID”框中，添加已准备好的“Azure 网络 SubnetID”的完整字符串，这是打算用于部署 Azure 虚拟机的位置。
3. 若要获取所有 Azure 网络子网的列表，请运行以下 PowerShell 命令：

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   “ID”字段显示“SUBNETID”。
4. 若要获取所有 **SUBNETID** 值的列表，请运行以下 PowerShell 命令：

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   **SUBNETID** 如下所示：

   ```
   /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>
   ```

### <a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> 部署用于测试和演示的仅限云 SAP 实例
可以在仅限云的部署模型中部署高可用性 SAP 系统。 这种部署主要用于演示和测试目的。 它不适合生产用例。

- 在 Azure 门户中的“参数”边栏选项卡上，从“NEWOREXISTINGSUBNET”框中选择“新建”。 将“SUBNETID”字段保留空白。

  SAP Azure Resource Manager 模板自动创建 Azure 虚拟网络和子网。

> [!NOTE]
> 还需要在同一个 Azure 虚拟网络实例中，针对 Active Directory 和 DNS 至少部署一个专用虚拟机。 模板不会创建这些虚拟机。
>
>


### <a name="prepare-the-infrastructure-for-architectural-template-2"></a>为体系结构模板 2 准备基础结构

可以借助此适用于 SAP 的 Azure Resource Manager 模板来简化 SAP 体系结构模板 2 所需基础结构资源的部署。

可在以下位置获取此部署方案的 Azure Resource Manager 模板：

* [Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged)  
* [使用托管磁盘的 Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged)
* [使用托管磁盘的自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


### <a name="prepare-the-infrastructure-for-architectural-template-3"></a>为体系结构模板 3 准备基础结构

可以准备基础结构并配置适用于 **多 SID**的 SAP。 例如，将附加的 SAP ASCS/SCS 实例添加到*现有*群集配置。 有关详细信息，请参阅 [在现有群集配置中配置附加的 SAP ASCS/SCS 实例以在 Azure Resource Manager 中创建 SAP 多 SID 配置][sap-ha-multi-sid-guide]。

若要创建新的多 SID 群集，可以使用 [GitHub 上的多 SID 快速入门模板](https://github.com/Azure/azure-quickstart-templates)。
若要创建新的多 SID 群集，需要部署以下三个模板：

* [ASCS/SCS 模板](#ASCS-SCS-template)
* [数据库模板](#database-template)
* [应用程序服务器模板](#application-servers-template)

以下部分中包含有关模板及需要在模板中提供的参数的更多详细信息。

#### <a name="ASCS-SCS-template"></a> ASCS/SCS 模板

ASCS/SCS 模板部署两个虚拟机，可以使用这两个虚拟机创建托管多个 ASCS/SCS 实例的 Windows Server 故障转移群集。

若要设置 ASCS/SCS 多 SID 模板，请在 [ASCS/SCS 多 SID 模板][sap-templates-3-tier-multisid-xscs-marketplace-image]或[使用托管磁盘的 ASCS/SCS 多 SID 模板][sap-templates-3-tier-multisid-xscs-marketplace-image-md]中，输入以下参数的值：

  - **资源前缀**。  设置资源前缀，它的作用是向部署期间创建的所有资源添加前缀。 由于资源不只属于一个 SAP 系统，因此资源的前缀并不是某个 SAP 系统的 SID。  前缀必须介于 **三到六个字符**之间。
  - **堆栈类型**。 选择 SAP 系统的堆栈类型。 根据堆栈类型，Azure 负载均衡器为每个 SAP 系统提供一个（ABAP 或仅限 Java）或两个 (ABAP + Java) 专用 IP 地址。
  -  **OS 类型**。 选择虚拟机的操作系统。
  -  **SAP 系统计数**。 选择要在此群集中安装的 SAP 系统数目。
  -  **系统可用性**。 选择“HA” 。
  -  **管理员用户名和管理员密码**。 创建可用于登录计算机的新用户。
  -  **新的或现有的子网**。 设置是否创建新的虚拟网络和子网，还是使用现有子网。 如果已有连接到本地网络的虚拟网络，请选择 **现有**虚拟网络。
  -  **子网 ID**。如果要将 VM 部署到现有 VNet 中，并且该 VNet 中已定义了 VM 应分配到的子网，请指定该特定子网的 ID。 ID 通常如下所示：/subscriptions/<订阅 ID>/resourceGroups/<资源组名称>/providers/Microsoft.Network/virtualNetworks/<虚拟网络名称>/subnets/<子网名称>

模板部署一个支持多个 SAP 系统的 Azure 负载均衡器实例。

- 为实例编号 00、10、20... 配置 ASCS 实例
- 为实例编号 01、11、21... 配置 SCS 实例
- 为实例编号 02、12、22... 配置 ASCS 排队复制服务器 (ERS)（仅适用于 Linux）实例
- SCS ERS（仅限 Linux）实例的实例编号配置为 03、13、23...

负载均衡器包含 1（Linux 为 2 个）个 VIP，一个 VIP 用于 ASCS/SCS，一个 VIP 用于 ERS（仅限 Linux）。

以下列表包含所有负载均衡规则（其中 x 是 SAP 系统数，例如 1、2、3...）：
- 每个 SAP 系统的 Windows 特定端口：445、5985
- ASCS 端口（实例编号 x0）：32x0、36x0、39x0、81x0、5x013、5x014、5x016
- SCS 端口（实例编号 x1）：32x1、33x1、39x1、81x1、5x113、5x114、5x116
- Linux 上的 ASCS ERS 端口（实例编号 x2）：33x2、5x213、5x214、5x216
- Linux 上的 SCS ERS 端口（实例编号 x3）：33x3、5x313、5x314、5x316

负载均衡器将配置为使用以下探测端口（其中 x 是 SAP 系统数，例如 1、2、3...）：
- ASCS/SCS 内部负载均衡器探测端口：620x0
- ERS 内部负载均衡器探测端口（仅限 Linux）：621x2

#### <a name="database-template"></a>数据库模板

数据库模板部署一个或两个虚拟机，可用于为一个 SAP 系统安装关系数据库管理系统 (RDBMS)。 例如，如果为 5 个 SAP 系统部署 ASCS/SCS 模板，则需要部署此模板 5 次。

若要设置数据库多 SID 模板，请在[数据库多 SID 模板][sap-templates-3-tier-multisid-db-marketplace-image]或[使用托管磁盘的数据库多 SID 模板][sap-templates-3-tier-multisid-db-marketplace-image-md]中，输入以下参数的值：

- **SAP 系统 ID**。输入要安装的 SAP 系统的 SAP 系统 ID。 该 ID 将用作所要部署的资源的前缀。
- **OS 类型**。 选择虚拟机的操作系统。
- **Dbtype**。 选择要在群集上安装的数据库的类型。 如果要安装 Microsoft SQL Server，请选择“SQL”  。 如果打算在虚拟机上安装 SAP HANA，请选择“HANA”  。 请确保选择正确的操作系统类型：对于 SQL，请选择“Windows”  ；对于 HANA，请选择一个 Linux 分发版。 连接到虚拟机的 Azure 负载均衡器将配置为支持所选的数据库类型：
  * **SQL**。 负载均衡器对端口 1433 进行负载均衡。 请确保在 SQL Server Always On 设置中使用此端口。
  * **HANA**。 负载均衡器将对端口 35015 和 35017 进行负载均衡。 请确保安装实例编号为 **50**的 SAP HANA。
  负载均衡器将使用探测端口 62550。
- **SAP 系统大小**。 设置新系统要提供的 SAPS 数量。 如果不确定系统需要多少 SAPS，请咨询 SAP 技术合作伙伴或系统集成商。
- **系统可用性**。 选择“HA” 。
- **管理员用户名和管理员密码**。 创建可用于登录计算机的新用户。
- **子网 ID**。输入在部署 ASCS/SCS 模板期间使用的子网的 ID，或部署 ASCS/SCS 模板过程中创建的子网的 ID。

#### <a name="application-servers-template"></a> 应用程序服务器模板

应用程序服务器模板部署两个或更多个虚拟机，可将这些虚拟机用作一个 SAP 系统的 SAP 应用程序服务器实例。 例如，如果为 5 个 SAP 系统部署 ASCS/SCS 模板，则需要部署此模板 5 次。

若要设置应用程序服务器多 SID 模板，请在[应用程序服务器多 SID 模板][sap-templates-3-tier-multisid-apps-marketplace-image]或[使用托管磁盘的应用程序服务器多 SID 模板][sap-templates-3-tier-multisid-apps-marketplace-image-md]中，输入以下参数的值：

  -  **SAP 系统 ID**。输入要安装的 SAP 系统的 SAP 系统 ID。 该 ID 将用作所要部署的资源的前缀。
  -  **OS 类型**。 选择虚拟机的操作系统。
  -  **SAP 系统大小**。 新系统将提供的 SAPS 数目。 如果不确定系统需要多少 SAPS，请咨询 SAP 技术合作伙伴或系统集成商。
  -  **系统可用性**。 选择“HA” 。
  -  **管理员用户名和管理员密码**。 创建可用于登录计算机的新用户。
  -  **子网 ID**。输入在部署 ASCS/SCS 模板期间使用的子网的 ID，或部署 ASCS/SCS 模板过程中创建的子网的 ID。


### <a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure 虚拟网络
在本例中，Azure 虚拟网络的地址空间为 10.0.0.0/16。 有一个名为 **Subnet**、地址范围为 10.0.0.0/24 的子网。 所有虚拟机和内部负载均衡器部署在此虚拟网络中。

> [!IMPORTANT]
> 请不要对来宾操作系统中的网络设置进行任何更改。 这包括 IP 地址、DNS 服务器和子网。 在 Azure 中配置所有网络设置。 动态主机配置协议 (DHCP) 服务会传播设置。
>
>

### <a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a>DNS IP 地址

若要设置所需的 DNS IP 地址，请执行以下步骤。

1. 在 Azure 门户中的“DNS 服务器”边栏选项卡上，确保虚拟网络的“DNS 服务器”选项设置为“自定义 DNS”。
2. 根据使用的网络类型选择设置。 有关详细信息，请参阅以下资源：
   * [企业网络连接（跨界）][planning-guide-2.2]：添加本地 DNS 服务器的 IP 地址。  
   可以将本地 DNS 服务器扩展到 Azure 中运行的虚拟机。 在该情况下，可添加运行 DNS 服务器的 Azure 虚拟机的 IP 地址。
   * 对于 Azure 中隔离的 VM 部署：在充当 DNS 服务器的同一虚拟网络实例中部署其他虚拟机。 添加已设置为运行 DNS 服务的 Azure 虚拟机的 IP 地址。

   ![图 12：为 Azure 虚拟网络配置 DNS 服务器][sap-ha-guide-figure-3001]

   _**图 12：** 为 Azure 虚拟网络配置 DNS 服务器_

   > [!NOTE]
   > 如果更改了 DNS 服务器的 IP 地址，则需要重新启动 Azure 虚拟机才能应用更改并传播新的 DNS 服务器。
   >
   >

在本例中，已在以下 Windows 虚拟机上安装并配置 DNS 服务：

| 虚拟机角色 | 虚拟机主机名 | 网卡名称 | 静态 IP 地址 |
| --- | --- | --- | --- |
| 第一台 DNS 服务器 |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| 第二台 DNS 服务器 |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

### <a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a>SAP ASCS/SCS 群集实例和 DBMS 群集实例的主机名与静态 IP 地址

对于本地部署，需要以下保留的主机名和 IP 地址：

| 虚拟主机名角色 | 虚拟主机名 | 虚拟静态 IP 地址 |
| --- | --- | --- |
| SAP ASCS/SCS 的第一个群集虚拟主机名（用于群集管理） |pr1-ascs-vir |10.0.0.42 |
| SAP ASCS/SCS 实例虚拟主机名 |pr1-ascs-sap |10.0.0.43 |
| SAP DBMS 的第二个群集虚拟主机名（群集管理） |pr1-dbms-vir |10.0.0.32 |

创建群集时，请创建虚拟主机名 **pr1-ascs-vir** 和 **pr1-dbms-vir** 以及管理群集本身的关联 IP 地址。 有关如何执行此操作的信息，请参阅 [收集群集配置中的群集节点][sap-ha-guide-8.12.1]。

可以在 DNS 服务器中手动创建另外两个虚拟主机名 **pr1-ascs-sap** 和 **pr1-dbms-sap**，以及关联的 IP 地址。 群集 SAP ASCS/SCS 实例和群集 DBMS 实例使用以下资源。 有关如何执行此操作的信息，请参阅 [为群集 SAP ASCS/SCS 实例创建虚拟主机名][sap-ha-guide-9.1.1]。

### <a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> 设置 SAP 虚拟机的静态 IP 地址
部署要在群集中使用的虚拟机之后，需要为所有虚拟机设置静态 IP 地址。 请在 Azure 虚拟网络配置中而不是来宾操作系统中执行此操作。

1. 在 Azure 门户中，选择“资源组” > “网卡” > “设置” > “IP 地址”。
2. 在“IP 地址”边栏选项卡中的“分配”下面，选择“静态”。 在“IP 地址”框中，输入要使用的 IP 地址。

   > [!NOTE]
   > 如果更改了网卡的 IP 地址，则需要重新启动 Azure 虚拟机才能应用更改。  
   >
   >

   ![图 13：为每个虚拟机的网卡设置静态 IP 地址][sap-ha-guide-figure-3002]

   _**图 13：** 为每个虚拟机的网卡设置静态 IP 地址_

   为所有网络接口（即为所有虚拟机，包括要用于 Active Directory/DNS 服务的虚拟机）重复此步骤。

本示例使用以下虚拟机和静态 IP 地址：

| 虚拟机角色 | 虚拟机主机名 | 网卡名称 | 静态 IP 地址 |
| --- | --- | --- | --- |
| 第一个 SAP 应用程序服务器实例 |pr1-di-0 |pr1-nic-di-0 |10.0.0.50 |
| 第二个 SAP 应用程序服务器实例 |pr1-di-1 |pr1-nic-di-1 |10.0.0.51 |
| ... |... |... |... |
| 最后一个 SAP 应用程序服务器实例 |pr1-di-5 |pr1-nic-di-5 |10.0.0.55 |
| ASCS/SCS 实例的第一个群集节点 |pr1-ascs-0 |pr1-nic-ascs-0 |10.0.0.40 |
| ASCS/SCS 实例的第二个群集节点 |pr1-ascs-1 |pr1-nic-ascs-1 |10.0.0.41 |
| DBMS 实例的第一个群集节点 |pr1-db-0 |pr1-nic-db-0 |10.0.0.30 |
| DBMS 实例的第二个群集节点 |pr1-db-1 |pr1-nic-db-1 |10.0.0.31 |

### <a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a>为 Azure 内部负载均衡器设置静态 IP 地址

SAP Azure Resource Manager 模板可创建用于 SAP ASCS/SCS 实例和 DBMS 实例的 Azure 内部负载均衡器。

> [!IMPORTANT]
> SAP ASCS/SCS 的虚拟主机名的 IP 地址与 SAP ASCS/SCS 内部负载均衡器 **pr1-lb-ascs**的 IP 地址相同。
> DBMS 的虚拟名称的 IP 地址与 DBMS 内部负载均衡器 **pr1-lb-dbms**的 IP 地址相同。
>
>

为 Azure 内部负载均衡器设置静态 IP 地址：

1. 初始部署将内部负载均衡器 IP 地址设置为“动态”。 在 Azure 门户的“IP 地址”边栏选项卡中的“分配”下面，选择“静态”。
2. 将内部负载均衡器 **pr1-lb-ascs** 的 IP 地址设置为 SAP ASCS/SCS 实例的虚拟主机名 IP 地址。
3. 请将内部负载均衡器 **pr1-lb-dbms** 的 IP 地址设置为 DBMS 实例的虚拟主机名的 IP 地址。

   ![图 14：为 SAP ASCS/SCS 实例的内部负载均衡器设置静态 IP 地址][sap-ha-guide-figure-3003]

   _**图 14：** 为 SAP ASCS/SCS 实例的内部负载均衡器设置静态 IP 地址_

在本例中，有两个具有以下静态 IP 地址的 Azure 内部负载均衡器：

| Azure 内部负载均衡器角色 | Azure 内部负载均衡器名称 | 静态 IP 地址 |
| --- | --- | --- |
| SAP ASCS/SCS 实例的内部负载均衡器 |pr1-lb-ascs |10.0.0.43 |
| SAP DBMS 内部负载均衡器 |pr1-lb-dbms |10.0.0.33 |


### <a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a>Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则

SAP Azure Resource Manager 模板创建所需的端口：
* 默认实例编号为 **00**
* 一个 Java SCS 实例，默认实例编号为 **01**

安装 SAP ASCS/SCS 实例时，必须为 ABAP ASCS 实例使用默认的实例编号 **00**，为 Java SCS 实例使用默认的实例编号 **01**。

接下来，为 SAP NetWeaver 端口创建所需的内部负载均衡终结点：

若要创建所需的内部负载均衡终结点，请先为 SAP NetWeaver ABAP ASCS 端口创建以下负载均衡终结点：

| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 00 的 ASCS 实例）（ERS 为 10）的具体端口 |
| --- | --- | --- |
| 排队服务器 / *lbrule3200* |32<*InstanceNumber*> |3200 |
| ABAP 消息服务器 / *lbrule3600* |36<*InstanceNumber*> |3600 |
| 内部 ABAP 消息 / *lbrule3900* |39<*InstanceNumber*> |3900 |
| 消息服务器 HTTP / *Lbrule8100* |81<*InstanceNumber*> |8100 |
| SAP 启动服务 ASCS HTTP / *Lbrule50013* |5<*InstanceNumber*>13 |50013 |
| SAP 启动服务 ASCS HTTPS / *Lbrule50014* |5<*InstanceNumber*>14 |50014 |
| 排队复制 / *Lbrule50016* |5<*InstanceNumber*>16 |50016 |
| SAP 启动服务 ERS HTTP *Lbrule51013* |5<*InstanceNumber*>13 |51013 |
| SAP 启动服务 ERS HTTP *Lbrule51014* |5<*InstanceNumber*>14 |51014 |
| Win RM *Lbrule5985* | |5985 |
| 文件共享 *Lbrule445* | |445 |

_**表 1：** SAP NetWeaver ABAP ASCS 实例的端口号_

然后，为 SAP NetWeaver Java SCS 端口创建以下负载均衡终结点：

| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 01 的 SCS 实例）（ERS 为 11）的具体端口 |
| --- | --- | --- |
| 排队服务器 / *lbrule3201* |32<*InstanceNumber*> |3201 |
| 网关服务器 / *lbrule3301* |33<*InstanceNumber*> |3301 |
| Java 消息服务器 / *lbrule3900* |39<*InstanceNumber*> |3901 |
| 消息服务器 HTTP / *Lbrule8101* |81<*InstanceNumber*> |8101 |
| SAP 启动服务 SCS HTTP / *Lbrule50113* |5<*InstanceNumber*>13 |50113 |
| SAP 启动服务 SCS HTTPS / *Lbrule50114* |5<*InstanceNumber*>14 |50114 |
| 排队复制 / *Lbrule50116* |5<*InstanceNumber*>16 |50116 |
| SAP 启动服务 ERS HTTP *Lbrule51113* |5<*InstanceNumber*>13 |51113 |
| SAP 启动服务 ERS HTTP *Lbrule51114* |5<*InstanceNumber*>14 |51114 |
| Win RM *Lbrule5985* | |5985 |
| 文件共享 *Lbrule445* | |445 |

_**表 2：** SAP NetWeaver Java SCS 实例的端口号_

![图 15：Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则][sap-ha-guide-figure-3004]

_**图 15：** Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则_

将负载均衡器 **pr1-lb-dbms** 的 IP 地址设置为 DBMS 实例的虚拟主机名 IP 地址。

### <a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> 更改 Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则

如果想要将其他编号用于 SAP ASCS 或 SCS 实例，则必须从默认值更改其端口的名称和值。

1. 在 Azure 门户中，依次选择“<SID>-lb-ascs 负载均衡器” > “负载均衡规则”。
2. 针对属于 SAP ASCS 或 SCS 实例的所有负载均衡规则，请更改以下值：

   * 名称
   * 端口
   * 后端端口

   例如，要将默认 ASCS 实例编号从 00 更改为 31，则需要针对表 1 中所列的所有端口进行更改。

   下面是端口 *lbrule3200*的更新示例。

   ![图 16：更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则][sap-ha-guide-figure-3005]

   _**图 16：** 更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则_

### <a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> 将 Windows 虚拟机添加到域

将静态 IP 地址分配给虚拟机之后，请将虚拟机添加到域。

![图 17：将虚拟机添加到域][sap-ha-guide-figure-3006]

_**图 17：** 将虚拟机添加到域_

### <a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> 在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项

Azure Load Balancer 包含一个内部负载均衡器，在连接空闲一段时间后（空闲超时）关闭连接。 对话实例中的 SAP 工作进程在需要发送第一个排队/取消排队请求时立即打开与 SAP 排队进程的连接。 这些连接通常持续保持已建立状态，直到工作进程或排队进程重新启动为止。 但是，如果连接空闲一段时间，Azure 内部负载均衡器会关闭连接。 这不是问题，因为如果连接不再存在，SAP 工作进程会重新建立与排队进程的连接。 这些活动已记录在 SAP 进程的开发人员跟踪中，但它们会在这些跟踪中创建大量额外的内容。 最好是同时更改两个群集节点上的 TCP/IP `KeepAliveTime` 和 `KeepAliveInterval` 。 本文稍后会介绍如何将 TCP/IP 参数的这些更改与 SAP 配置文件参数的这些更改进行合并。

若要在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项，首先，请在 SAP ASCS/SCS 的两个 Windows 群集节点上添加以下 Windows 注册表项：

| 路径 | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 变量名称 |`KeepAliveTime` |
| 变量类型 |REG_DWORD（十进制） |
| 值 |120000 |
| 文档链接 |[https://technet.microsoft.com/library/cc957549.aspx](https://technet.microsoft.com/library/cc957549.aspx) |

_**表 3：** 更改第一个 TCP/IP 参数_

然后，在 SAP ASCS/SCS 实例的两个 Windows 群集节点上添加此 Windows 注册表项：

| 路径 | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 变量名称 |`KeepAliveInterval` |
| 变量类型 |REG_DWORD（十进制） |
| 值 |120000 |
| 文档链接 |[https://technet.microsoft.com/library/cc957548.aspx](https://technet.microsoft.com/library/cc957548.aspx) |

_**表 4：** 更改第二个 TCP/IP 参数_

**若要应用更改，请重新启动两个群集节点。**

### <a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> 为 SAP ASCS/SCS 实例设置 Windows Server 故障转移群集

为 SAP ASCS/SCS 实例设置 Windows Server 故障转移群集包括下列任务：

- 收集群集配置中的群集节点
- 配置群集文件共享见证

#### <a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> 收集群集配置中的群集节点

1. 在“添加角色和功能向导”中，将故障转移群集添加到两个群集节点。
2. 使用故障转移群集管理器设置故障转移群集。 在故障转移群集管理器中选择“创建群集”，并只添加第一个群集节点 A 的名称。暂时不要添加第二个节点；将在后面的步骤中添加第二个节点。

   ![图 18：添加第一个群集节点的服务器或虚拟机名称][sap-ha-guide-figure-3007]

   _**图 18：** 添加第一个群集节点的服务器或虚拟机名称_

3. 输入群集的网络名称（虚拟主机名）。

   ![图 19：输入群集名称][sap-ha-guide-figure-3008]

   _**图 19：** 输入群集名称_

4. 创建群集后，运行群集验证测试。

   ![图 20：运行群集验证检查][sap-ha-guide-figure-3009]

   _**图 20：** 运行群集验证检查_

   在过程中的此时点，可忽略关于磁盘的任何警告。 将稍后添加文件共享见证和 SIOS 共享磁盘。 在此阶段不必担心仲裁问题。

   ![图 21：找不到任何仲裁磁盘][sap-ha-guide-figure-3010]

   _**图 21：** 找不到任何仲裁磁盘_

   ![图 22：核心群集资源需要新 IP 地址][sap-ha-guide-figure-3011]

   _**图 22：** 核心群集资源需要新 IP 地址_

5. 更改核心群集服务的 IP 地址。 在更改核心群集服务的 IP 地址之前，群集无法启动，因为服务器的 IP 地址指向其中一个虚拟机节点。 请在核心群集服务的 IP 资源“属性”页上执行此操作。

   例如，需要为群集虚拟主机名 **pr1-ascs-vir** 分配 IP 地址（在本例中为 **10.0.0.42**）。

   ![图 23：在“属性”对话框中更改 IP 地址][sap-ha-guide-figure-3012]

   _**图 23：** 在“属性”对话框中更改 IP 地址_

   ![图 24：分配为群集保留的 IP 地址][sap-ha-guide-figure-3013]

   _**图 24：** 分配为群集保留的 IP 地址_

6. 将群集虚拟主机名联机。

   ![图 25：群集核心服务保持正常运行，使用正确的 IP 地址][sap-ha-guide-figure-3014]

   _**图 25：** 群集核心服务保持正常运行，使用正确的 IP 地址_

7. 添加第二个群集节点。

   核心群集服务启动并运行后，可以添加第二个群集节点。

   ![图 26：添加第二个群集节点][sap-ha-guide-figure-3015]

   _**图 26：** 添加第二个群集节点_

8. 输入第二个群集节点主机的名称。

   ![图 27：输入第二个群集节点主机名][sap-ha-guide-figure-3016]

   _**图 27：** 输入第二个群集节点主机名_

   > [!IMPORTANT]
   > **切勿**选中“将所有符合条件的存储添加到群集”复选框。  
   >
   >

   ![图 28：请勿选中该复选框][sap-ha-guide-figure-3017]

   _**图 28：** **请勿** 选中该复选框_

   可以忽略关于仲裁和磁盘的警告。 稍后会根据[为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition][sap-ha-guide-8.12.3]中所述设置仲裁和共享磁盘。

   ![图 29：忽略关于磁盘仲裁的警告][sap-ha-guide-figure-3018]

   _**图 29：** 忽略关于磁盘仲裁的警告_


#### <a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> 配置群集文件共享见证

配置群集文件共享见证包括下列任务：

- 创建文件共享
- 在故障转移群集管理器中设置文件共享见证仲裁

##### <a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> 创建文件共享

1. 请选择文件共享见证而不是仲裁磁盘。 SIOS DataKeeper 支持此选项。

   在本文的示例中，文件共享见证位于 Azure 中运行的 Active Directory/DNS 服务器上。 文件共享见证称为 **domcontr-0**。 由于已配置到 Azure 的 VPN 连接（通过站点到站点 VPN 或 Azure ExpressRoute），因此 Active Directory/DNS 服务在本地，并且不适合运行文件共享见证。

   > [!NOTE]
   > 如果 Active Directory/DNS 服务仅在本地运行，请不要在本地运行的 Active Directory/DNS Windows 操作系统上配置文件共享见证。 在 Azure 中与本地 Active Directory DNS 中运行的群集节点之间的网络延迟可能太大，会造成连接问题。 请务必在运行位置靠近群集节点的 Azure 虚拟机上配置文件共享见证。  
   >
   >

   仲裁驱动器至少需要 1,024 MB 可用空间。 建议为仲裁驱动器留出 2,048 MB 可用空间。

2. 添加群集名称对象。

   ![图 30：为群集名称对象分配共享权限][sap-ha-guide-figure-3019]

   _**图 30：** 为群集名称对象分配共享权限_

   请确保权限包括针对群集名称对象（在本例中为 **pr1-ascs-vir$**）更改共享中的数据的授权。

3. 要将群集名称添加到列表，请选择“添加”。 更改筛选器，以便除了检查图 31 中所示的项以外，还检查计算机对象。

   ![图 31：将“对象类型”更改为包括计算机][sap-ha-guide-figure-3020]

   _**图 31：** 将“对象类型”更改为包括计算机_

   ![图 32：选中“计算机”复选框][sap-ha-guide-figure-3021]

   _**图 32：** 选中“计算机”复选框_

4. 输入群集名称对象，如图 31 所示。 由于已创建记录，因此可以更改权限，如图 30 所示。

5. 选择共享的“安全性”  选项卡，并为群集名称对象设置更详细的权限。

   ![图 33：为文件共享仲裁上的群集名称对象设置安全属性][sap-ha-guide-figure-3022]

   _**图 33：** 为文件共享仲裁上的群集名称对象设置安全属性_

##### <a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> 在故障转移群集管理器中设置文件共享见证

1. 打开“配置仲裁设置向导”。

   ![图 34：启动“配置群集仲裁设置向导”][sap-ha-guide-figure-3023]

   _**图 34：** 启动“配置群集仲裁设置向导”_

2. 上“选择仲裁配置”页上，选择“选择仲裁见证”。

   ![图 35：可供选择的仲裁配置][sap-ha-guide-figure-3024]

   _**图 35：** 可供选择的仲裁配置_

3. 在“选择仲裁见证”页上，选择“配置文件共享见证”。

   ![图 36：选择文件共享见证][sap-ha-guide-figure-3025]

   _**图 36：** 选择文件共享见证_

4. 输入文件共享的 UNC 路径（在本例中为 \\domcontr-0\FSW）。 若要查看可进行的更改列表，请选择“下一步”。

   ![图 37：定义见证共享的文件共享位置][sap-ha-guide-figure-3026]

   _**图 37：** 定义见证共享的文件共享位置_

5. 选择所需的更改，并选择“下一步”。 需要成功重新配置群集配置，如图 38 中所示。  

   ![图 38：确认已重新配置群集][sap-ha-guide-figure-3027]

   _**图 38：** 确认已重新配置群集_

成功安装 Windows 故障转移群集后，需要更改某些阈值，以使故障转移检测适合于 Azure 中的情况。 要更改的参数均记录在此博客中： https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/。 假定用于生成 ASCS/SCS 的 Windows 群集配置的两个 VM 位于同一子网中，需要将以下参数更改为这些值：
- SameSubNetDelay = 2
- SameSubNetThreshold = 15

一方面，这些设置已经过客户测试，并且为具有足够弹性提供了良好的折中办法。 另一方面，在发生 SAP 软件或节点/VM 故障时的实际出错情况下，这些设置提供了足够快速的故障转移。 

### <a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> 为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition

现在，已在 Azure 中具有有效的 Windows Server 故障转移群集配置。 但是，若要安装 SAP ASCS/SCS 实例，需有一个共享磁盘资源。 无法在 Azure 中创建所需的共享磁盘资源。 SIOS DataKeeper Cluster Edition 是可用于创建共享磁盘资源的第三方解决方案。

为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition 包括下列任务：

- 添加 .NET Framework 3.5
- 安装 SIOS DataKeeper
- 设置 SIOS DataKeeper

#### <a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> 添加 .NET Framework 3.5
Microsoft .NET Framework 3.5 不会在 Windows Server 2012 R2 上自动激活或安装。 由于 SIOS DataKeeper 需要在安装 DataKeeper 的所有节点上安装 .NET Framework，因此必须在群集中所有虚拟机的来宾操作系统上安装 .NET Framework 3.5。

可通过两种方法添加 .NET Framework 3.5：

- 使用 Windows 中的“添加角色和功能向导”，如图 39 所示。

  ![图 39：使用“添加角色和功能向导”安装 .NET Framework 3.5][sap-ha-guide-figure-3028]

  _**图 39：** 使用“添加角色和功能向导”安装 .NET Framework 3.5_

  ![图 40：使用“添加角色和功能向导”安装 .NET Framework 3.5 时的安装进度栏][sap-ha-guide-figure-3029]

  _**图 40：** 使用“添加角色和功能向导”安装 .NET Framework 3.5 时的安装进度栏_

- 使用命令行工具 dism.exe。 对于这种类型的安装，需要访问 Windows 安装媒体中的 SxS 目录。 在权限提升的命令提示符下键入：

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

#### <a name="dd41d5a2-8083-415b-9878-839652812102"></a> 安装 SIOS DataKeeper

在群集中的每个节点上安装 SIOS DataKeeper Cluster Edition。 如果要使用 SIOS DataKeeper 创建虚拟共享存储，请创建同步的镜像，并模拟群集共享存储。

在安装 SIOS 软件前，创建域用户 **DataKeeperSvc**。

> [!NOTE]
> 请将 **DataKeeperSvc** 用户添加到两个群集节点上的“本地管理员”组中。
>
>

安装 SIOS DataKeeper：

1. 在两个群集节点上安装 SIOS 软件。

   ![SIOS 安装程序][sap-ha-guide-figure-3030]

   ![图 41：SIOS DataKeeper 安装程序的第一页][sap-ha-guide-figure-3031]

   _**图 41：** SIOS DataKeeper 安装程序的第一页_

2. 在图 42 所示的对话框中，选择“是”。

   ![图 42：DataKeeper 通知将会禁用服务][sap-ha-guide-figure-3032]

   _**图 42：** DataKeeper 通知将会禁用服务_

3. 在图 43 所示的对话框中，建议选择“域或服务器帐户”。

   ![图 43：用户为 SIOS DataKeeper 选中的选项][sap-ha-guide-figure-3033]

   _**图 43：** 用户为 SIOS DataKeeper 选中的选项_

4. 输入为 SIOS DataKeeper 创建的域帐户用户名和密码。

   ![图 44：输入域用户名和密码以安装 SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**图 44：** 输入域用户名和密码以安装 SIOS DataKeeper_

5. 如图 45 所示，安装 SIOS DataKeeper 实例的许可证密钥。

   ![图 45：输入 SIOS DataKeeper 许可证密钥][sap-ha-guide-figure-3035]

   _**图 45：** 输入 SIOS DataKeeper 许可证密钥_

6. 根据提示重新启动虚拟机。

#### <a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> 设置 SIOS DataKeeper

在两个节点上安装 SIOS DataKeeper 后，需要开始配置。 配置的目的是在连接到每个虚拟机的附加磁盘之间进行同步数据复制。

1. 启动 DataKeeper 管理和配置工具，并选择“连接服务器”。 （在图 46 中，此选项标有红色圆圈。）

   ![图 46：SIOS DataKeeper 管理和配置工具][sap-ha-guide-figure-3036]

   _**图 46：** SIOS DataKeeper 管理和配置工具_

2. 输入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，输入第二个节点的相关数据。

   ![图 47：插入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，插入第二个节点的相关数据][sap-ha-guide-figure-3037]

   _**图 47：** 插入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，插入第二个节点的相关数据_

3. 在两个节点之间创建复制作业。

   ![图 48：创建复制作业][sap-ha-guide-figure-3038]

   _**图 48：** 创建复制作业_

   将有一个向导引导完成创建复制作业的过程。
4. 定义源节点的名称、TCP/IP 地址和磁盘卷。

   ![图 49：定义复制作业的名称][sap-ha-guide-figure-3039]

   _**图 49：** 定义复制作业的名称_

   ![图 50：定义节点（应为当前源节点）的基本数据][sap-ha-guide-figure-3040]

   _**图 50：** 定义节点（应为当前源节点）的基本数据_

5. 定义目标节点的名称、TCP/IP 地址和磁盘卷。

   ![图 51：定义节点（应为当前目标节点）的基本数据][sap-ha-guide-figure-3041]

   _**图 51：** 定义节点（应为当前目标节点）的基本数据_

6. 定义压缩算法。 在本示例中，建议压缩复制流。 尤其是在重新同步的情况下，压缩复制流可大幅缩短重新同步时间。 请注意，压缩会占用虚拟机的 CPU 和 RAM 资源。 随着压缩率增大，占用的 CPU 资源量也会随着增多。 以后也可以调整此设置。

7. 另一个需要检查的设置是复制是以异步还是同步方式进行。 *保护 SAP ASCS/SCS 配置时，必须使用同步复制*。  

   ![图 52：定义复制详细信息][sap-ha-guide-figure-3042]

   _**图 52：** 定义复制详细信息_

8. 定义是否应向 Windows Server 故障转移群集配置将复制作业所复制的卷表示为共享磁盘。 对于 SAP ASCS/SCS 配置，选择“是”，以便 Windows 群集将复制的卷视为可用作群集卷的共享磁盘。

   ![图 53：选择“是”将复制的卷设置为群集卷][sap-ha-guide-figure-3043]

   _**图 53：** 选择“是”将复制的卷设置为群集卷_

   创建卷后，DataKeeper 管理和配置工具显示复制作业处于活动状态。

   ![图 54：SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态][sap-ha-guide-figure-3044]

   _**图 54：** SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态_

   现在，故障转移群集管理器将磁盘显示为 DataKeeper 磁盘，如图 55 所示。

   ![图 55：故障转移群集管理器显示 DataKeeper 复制的磁盘][sap-ha-guide-figure-3045]

   _**图 55：** 故障转移群集管理器显示 DataKeeper 复制的磁盘_

## <a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a> 安装 SAP NetWeaver 系统

我们不会介绍 DBMS 安装，因为安装因所使用的 DBMS 系统而异。 但是，本文假设 DBMS 在高可用性方面的疑虑已通过不同 DBMS 供应商为 Azure 提供的功能支持而获得解决。 例如，适用于 SQL Server 的 AlwaysOn 或数据库镜像，以及适用于 Oracle 数据库的 Oracle Data Guard。 在本文使用的方案中，并未对 DBMS 添加更多保护。

当不同的 DBMS 服务与 Azure 中这种群集 SAP ASCS/SCS 配置交互时，不存在任何特殊注意事项。

> [!NOTE]
> SAP NetWeaver ABAP 系统、Java 系统和 ABAP+Java 系统的安装过程几乎完全相同。 最明显的差别在于，SAP ABAP 系统只有一个 ASCS 实例。 SAP Java 系统只有一个 SCS 实例， 而 SAP ABAP+Java 系统则有一个 ASCS，以及一个在同一 Microsoft 故障转移群集组中运行的 SCS 实例。 将明确说明每个 SAP NetWeaver 安装堆栈的所有安装差异。 可以假设其他所有部分相同。  
>
>

### <a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a>安装包含高可用性 ASCS/SCS 实例的 SAP 系统

> [!IMPORTANT]
> 切勿将页面文档放在 DataKeeper 镜像卷上。 DataKeeper 不支持镜像卷。 可将页面文件保留在 Azure 虚拟机的临时驱动器 D 中，这是默认设置。 如果它尚未在该位置，请将 Windows 页面文件移到 Azure 虚拟机的 D: 驱动器。
>
>

安装包含高可用性 ASCS/SCS 实例的 SAP 系统包括下列任务：

- 为群集 SAP ASCS/SCS 实例创建虚拟主机名
- 安装 SAP 的第一个群集节点
- 修改 ASCS/SCS 实例的 SAP 配置文件
- 添加探测端口
- 打开 Windows 防火墙探测端口

#### <a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a> 为群集 SAP ASCS/SCS 实例创建虚拟主机名

1. 在 Windows DNS 管理器中为 ASCS/SCS 实例的虚拟主机名创建 DNS 条目。

   > [!IMPORTANT]
   > 分配给 ASCS/SCS 实例虚拟主机名的 IP 地址必须与分配给 Azure 负载均衡器 (<SID>-lb-ascs) 的 IP 地址相同。  
   >
   >

   虚拟 SAP ASCS/SCS 主机名 (**pr1-ascs-sap**) 的 IP 地址与 Azure 负载均衡器 (**pr1-lb-ascs**) 的 IP 地址相同。

   ![图 56：定义 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 条目][sap-ha-guide-figure-3046]

   _**图 56：** 定义 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 条目_

2. 若要定义分配给虚拟主机名的 IP 地址，请选择“DNS 管理器” > “域”。

   ![图 57：SAP ASCS/SCS 群集配置的新虚拟名称和 TCP/IP 地址][sap-ha-guide-figure-3047]

   _**图 57：** SAP ASCS/SCS 群集配置的新虚拟名称和 TCP/IP 地址_

#### <a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> 安装 SAP 第一个群集节点

1. 在群集节点 A 上（例如，在 **pr1-ascs-0** 主机上）执行“第一个群集节点”选项。
2. 若要保留 Azure 内部负载均衡器的默认端口，请选择：

   * **ABAP 系统**：**ASCS** 实例编号 **00**
   * **Java 系统**：**SCS** 实例编号 **01**
   * **ABAP+Java 系统**：**ASCS** 实例编号 **00** 和 **SCS** 实例编号 **01**

   若要为 ABAP ASCS 实例使用 00 以外的编号，为 Java SCS 实例使用 01 以外的编号，首先需要根据[更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则][sap-ha-guide-8.9]中所述，更改 Azure 内部负载均衡器的默认负载均衡规则。

接下来的几个任务在标准 SAP 安装文档中未介绍。

> [!NOTE]
> SAP 安装文档介绍了如何安装第一个 ASCS/SCS 群集节点。
>
>

#### <a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> 修改 ASCS/SCS 实例的 SAP 配置文件

需要添加新的配置文件参数。 此配置文件参数可避免 SAP 工作进程与排队服务器之间的连接在空闲时间太长时关闭。 [在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项][sap-ha-guide-8.11]中已提到了出现该问题的情景。 该部分还介绍了对一些基本 TCP/IP 连接参数所做的两项更改。 在第二个步骤中，需将排队服务器设置为发送 `keep_alive` 信号，以便连接不会达到 Azure 内部负载均衡器的空闲阈值。

修改 ASCS/SCS 实例的 SAP 配置文件：

1. 将此配置文件参数添加到 SAP ASCS/SCS 实例配置文件：

   ```
   enque/encni/set_so_keepalive = true
   ```
   在本例中，路径为：

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

   例如，添加到 SAP SCS 实例配置文件和相应的路径：

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2. 若要应用更改，请重新启动 SAP ASCS/SCS 实例。

#### <a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> 添加探测端口

使用内部负载均衡器探测功能，让整个群集配置使用 Azure 负载均衡器。 Azure 内部负载均衡器通常在参与的虚拟机之间平均分配传入的工作负荷。 但是，这在某些群集配置中由于只有一个实例处于活动状态而不起作用。 其他实例处于被动状态，并且无法接受任何工作负荷。 当 Azure 内部负载均衡器只将任务分配给主动实例时，探测功能才起作用。 通过探测功能，内部负载均衡器可检测到哪些实例处于活动状态，仅定位带有工作负荷的实例。

添加探测端口：

1. 通过运行以下 PowerShell 命令检查当前 **ProbePort** 设置。 在群集配置中的某个虚拟机上执行该检查。

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
   ```

2. 定义探测端口。 默认探测端口号为 **0**。 本示例使用探测端口 **62000**。

   ![图 58：群集配置探测端口默认为 0][sap-ha-guide-figure-3048]

   _**图 58：** 群集配置探测端口默认为 0_

   端口号在 SAP Azure Resource Manager 模板中定义。 可在 PowerShell 中分配端口号。

   若要为 SAP <SID> IP 群集资源设置新的 ProbePort 值，请运行以下 PowerShell 脚本。 更新环境的 PowerShell 变量。 脚本运行后，系统会提示重新启动 SAP 群集组，以便激活所做的更改。

   ```powershell
   $SAPSID = "PR1"      # SAP <SID>
   $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

   Clear-Host
   $SAPClusterRoleName = "SAP $SAPSID"
   $SAPIPresourceName = "SAP $SAPSID IP"
   $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
   $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
   $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
   $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
   $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
   $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
   $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

   $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }

   Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

   Write-Host
   Write-Host "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." -ForegroundColor Cyan
   Write-Host
   Write-Host "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." -ForegroundColor Cyan
   Write-Host

   $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

   Write-Host

   $ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"

   if($ActivateChanges -eq "yes"){
   Write-Host
   Write-Host "Activating changes..." -ForegroundColor Cyan

   Write-Host
   write-host "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..." -ForegroundColor Cyan
   Stop-ClusterResource -Name $SAPIPresourceName
   sleep 5

   Write-Host "Starting SAP cluster role '$SAPClusterRoleName' ..." -ForegroundColor Cyan
   Start-ClusterGroup -Name $SAPClusterRoleName

   Write-Host "New ProbePort parameter is active." -ForegroundColor Green
   Write-Host

   Write-Host "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" -ForegroundColor Cyan
   Write-Host
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
   }else
   {
   Write-Host "Changes are not activated."
   }
   ```

   将 **SAP <*SID*>** 群集角色联机之后，验证**ProbePort**是否已设置为新值。

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

   ```

   ![图 59：设置新值后探测群集端口][sap-ha-guide-figure-3049]

   _**图 59：** 设置新值后探测群集端口_

#### <a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a>打开 Windows 防火墙探测端口

需要在两个群集节点上打开 Windows 防火墙探测端口。 使用以下脚本打开 Windows 防火墙探测端口。 更新环境的 PowerShell 变量。

  ```powershell
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

**ProbePort** 设置为 **62000**。 现在，可从其他主机（例如 **ascsha-dbas**）访问文件共享 **\\\ascsha-clsap\sapmnt**。

### <a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> 安装数据库实例

若要安装数据库实例，请遵循 SAP 安装文档中所述的过程。

### <a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> 安装第二个群集节点

若要安装第二个群集，请遵循 SAP 安装指南中的步骤。

### <a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> 更改 SAP ERS Windows 服务实例的启动类型

将两个群集节点上的 SAP 排队复制服务器 (ERS) Windows 服务的启动类型更改为“自动(延迟启动)”。

![图 60：将 SAP ERS 实例的服务类型更改为自动延迟][sap-ha-guide-figure-3050]

_**图 60：** 将 SAP ERS 实例的服务类型更改为自动延迟_

### <a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> 安装 SAP 主应用程序服务器

在已指定为托管 PAS 的虚拟机上安装主应用程序服务器 (PAS) 实例 <*SID*>-di-0。 这种安装不需要考虑 Azure 还是 DataKeeper 相关的设置。

### <a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a>安装 SAP 附加应用程序服务器

在指定用于托管 SAP 应用程序服务器实例的所有虚拟机上安装 SAP 附加应用程序服务器 (AAS)。 例如，在 <*SID*>-di-1 到 <*SID*>-di-&lt;n&gt; 上。

> [!NOTE]
> 现已完成高可用性 SAP NetWeaver 系统的安装。 接下来，继续执行故障转移测试。
>


## <a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> 测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制
可以使用故障转移群集管理器和 SIOS DataKeeper 管理与配置工具，轻松测试及监视 SAP ASCS/SCS 实例故障转移与 SIOS 磁盘复制。

### <a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> SAP ASCS/SCS 实例在群集节点 A 上运行

**SAP PR1** 群集组在群集节点 A（例如在 **pr1-ascs-0**）上运行。 将属于 **SAP PR1** 群集组并由 ASCS/SCS 实例使用的共享磁盘驱动器 S 分配到群集节点 A。

![图 61：故障转移群集管理器：SAP <SID> 群集组在群集节点 A 上运行][sap-ha-guide-figure-5000]

_**图 61：** 故障转移群集管理器：SAP <*SID*> 群集组在群集节点 A 上运行_

在 SIOS DataKeeper 管理和配置工具中，可以看到共享磁盘数据以同步方式从群集节点 A 上的源卷 S 复制到群集节点 B 上的目标卷 S。例如，从 **pr1-ascs-0 [10.0.0.40]** 复制到 **pr1-ascs-1 [10.0.0.41]**。

![图 62：在 SIOS DataKeeper 中，将本地卷从群集节点 A 复制到群集节点 B][sap-ha-guide-figure-5001]

_**图 62：** 在 SIOS DataKeeper 中，将本地卷从群集节点 A 复制到群集节点 B_

### <a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> 从节点 A 到节点 B 的故障转移

1. 选择以下选项之一，开始将 SAP <*SID*> 群集组从群集节点 A 故障转转到群集节点 B 的故障转移：
   - 使用故障转移群集管理器  
   - 使用故障转移群集 PowerShell

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPClusterGroup = "SAP $SAPSID"
   Move-ClusterGroup -Name $SAPClusterGroup

   ```
2. 在 Windows 来宾操作系统中重新启动群集节点 A（这会启动将 SAP <*SID*> 群集组从节点 A 故障转移到节点 B 的自动故障转移）。  
3. 在 Azure 门户中重新启动群集节点 A（这会启动将 SAP <*SID*> 群集组从节点 A 故障转移到节点 B 的自动故障转移）。  
4. 使用 Azure PowerShell 重新启动群集节点 A（这会启动将 SAP <*SID*> 群集组从节点 A 故障转移到节点 B 的自动故障转移）。

   故障转移后，SAP <*SID*> 群集组会在群集节点 B 上运行。例如，在 **pr1-ascs-1** 上运行。

   ![图 63：在故障转移群集管理器中，SAP <SID> 群集组在群集节点 B 上运行][sap-ha-guide-figure-5002]

   _**图 63**：在故障转移群集管理器中，SAP <*SID*> 群集组在群集节点 B 上运行_

   共享磁盘现在已装载到群集节点 B。SIOS DataKeeper 正在将数据从群集节点 B 上的源卷 S 复制到群集节点 A 上的目标卷 S。例如，从 **pr1-ascs-1 [10.0.0.41]** 复制到 **pr1-ascs-0 [10.0.0.40]**。

   ![图 64：SIOS DataKeeper 将本地卷从群集节点 B 复制到群集节点 A][sap-ha-guide-figure-5003]

   _**图 64：** SIOS DataKeeper 将本地卷从群集节点 B 复制到群集节点 A_
