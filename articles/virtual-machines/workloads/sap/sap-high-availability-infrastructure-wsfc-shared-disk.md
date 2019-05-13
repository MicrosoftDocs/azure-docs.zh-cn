---
title: 针对 SAP ASCS/SCS 使用 Windows 故障转移群集和共享磁盘准备 SAP HA 的 Azure 基础结构 | Microsoft Docs
description: 了解如何针对 SAP ASCS/SCS 实例使用 Windows 故障转移群集和共享磁盘准备 SAP HA 的 Azure 基础结构。
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ec976257-396b-42a0-8ea1-01c97f820fa6
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ae3d1b36b89bb1bce1ff384bfa12a1bf643614fd
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408778"
---
# <a name="prepare-the-azure-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster-and-shared-disk-for-sap-ascsscs"></a>针对 SAP ASCS/SCS 使用 Windows 故障转移群集和共享磁盘准备 SAP HA 的 Azure 基础结构

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides
[tuning-failover-cluster-network-thresholds]:https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834

[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-9.1.1]:high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f


[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (SAP 多 SID 高可用性配置)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


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


> ![Windows][Logo_Windows] Windows
>

本文介绍使用群集共享磁盘作为群集化 SAP ASCS 实例的选项，在 Windows 故障转移群集上安装和配置高可用性 SAP 系统之前准备 Azure 基础结构所要执行的步骤。

## <a name="prerequisites"></a>必备组件

在开始安装之前，请查看以下文章：

* [体系结构指南：使用群集共享磁盘在 Windows 故障转移群集上组建 SAP ASCS/SCS 实例的群集][sap-high-availability-guide-wsfc-shared-disk]

## <a name="prepare-the-infrastructure-for-architectural-template-1"></a>准备体系结构模板 1 的基础结构
适用于 SAP 的 Azure 资源管理器模板有助于简化所需资源的部署。

Azure 资源管理器中的三层模板还支持高可用性方案。 例如体系结构模板 1 包含两个群集。 每个群集都是 SAP ASCS/SCS 和 DBMS 的 SAP 单一故障点。

可在以下位置获取本文中所述的示例方案的 Azure 资源管理器模板：

* [Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image)  
* [使用 Azure 托管磁盘的 Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image)
* [使用托管磁盘的自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

准备体系结构模板 1 的基础结构：

- 在 Azure 门户上的“参数”窗格中，从“SYSTEMAVAILABILITY”框中选择“HA”。

  ![图 1：设置 SAP 高可用性 Azure 资源管理器参数][sap-ha-guide-figure-3000]

_**图 1：** 设置 SAP 高可用性 Azure 资源管理器参数_


  模板创建：

  * **虚拟机**：
    * SAP Application Server 虚拟机：\<SAPSystemSID\>-di-\<Number\>
    * ASCS/SCS 群集虚拟机：\<SAPSystemSID\>-ascs-\<Number\>
    * DBMS 群集:\<SAPSystemSID\>-db-\<Number\>

  * **所有虚拟机的网卡以及关联的 IP 地址**：
    * \<SAPSystemSID\>-nic-di-\<Number\>
    * \<SAPSystemSID\>-nic-ascs-\<Number\>
    * \<SAPSystemSID\>-nic-db-\<Number\>

  * **Azure 存储帐户（仅限非托管磁盘）**：

  * 以下组件的**可用性组**：
    * SAP Application Server 虚拟机：\<SAPSystemSID\>-avset-di
    * SAP ASCS/SCS 群集虚拟机：\<SAPSystemSID\>-avset-ascs
    * DBMS 群集虚拟机：\<SAPSystemSID\>-avset-db

  * **Azure 内部负载均衡器**：
    * 带有 ASCS/SCS 实例的所有端口和 IP 地址 \<SAPSystemSID\>-lb-ascs
    * 带有 SQL Server DBMS 的所有端口和 IP 地址 \<SAPSystemSID\>-lb-db

  * **网络安全组**：\<SAPSystemSID\>-nsg-ascs-0  
    * 带有 \<SAPSystemSID\>-ascs-0 虚拟机的开放外部远程桌面协议 (RDP) 端口

> [!NOTE]
> 默认情况下，网卡和 Azure 内部负载均衡器的所有 IP 地址均为动态。 将它们更改为静态 IP 地址。 本文稍后介绍操作方法。
>
>

## <a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a>部署带有公司网络连接的虚拟机（跨界）以在生产中使用
对于生产 SAP 系统，使用 Azure VPN 网关或 Azure ExpressRoute 部署具有[企业网络连接（跨界）][planning-guide-2.2]的 Azure 虚拟机。

> [!NOTE]
> 可使用 Azure 虚拟网络实例。 已创建并准备好虚拟网络与子网。
>
>

1. 在 Azure 门户上的“参数”窗格中，从“NEWOREXISTINGSUBNET”框中选择“现有”。
2. 在“SUBNETID”框中，添加已准备好的 Azure 网络子网 ID 的完整字符串，这是用于部署 Azure 虚拟机的位置。
3. 若要获取所有 Azure 网络子网的列表，请运行以下 PowerShell 命令：

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   “ID”字段显示子网 ID 的值。
4. 若要获取所有子网 ID 值的列表，请运行以下 PowerShell 命令：

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   子网 ID 如下所示：

   ```
   /subscriptions/<subscription ID>/resourceGroups/<VPN name>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<subnet name>
   ```

## <a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a>部署用于测试和演示的仅限云 SAP 实例
可以在仅限云的部署模型中部署高可用性 SAP 系统。 这种部署主要用于演示和测试目的。 它不适合生产用例。

- 在 Azure 门户上的“参数”窗格中，从“NEWOREXISTINGSUBNET”框中选择“新建”。 将“SUBNETID”字段保留空白。

  SAP Azure 资源管理器模板自动创建 Azure 虚拟网络和子网。

> [!NOTE]
> 还需要在同一个 Azure 虚拟网络实例中为 Active Directory 和 DNS 服务部署至少一个专用虚拟机。 该模板不创建这些虚拟机。
>
>


## <a name="prepare-the-infrastructure-for-architectural-template-2"></a>准备体系结构模板 2 的基础结构

可以借助这个适用于 SAP 的 Azure 资源管理器模板来简化 SAP 体系结构模板 2 所需基础结构资源的部署。

可从以下位置获取适用于此部署方案的 Azure 资源管理器模板：

* [Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged)  
* [使用托管磁盘的 Azure 市场映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged)
* [使用托管磁盘的自定义映像](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


## <a name="prepare-the-infrastructure-for-architectural-template-3"></a>准备体系结构模板 3 的基础结构

可以准备基础结构并在 SAP 中配置多 SID。 例如，将附加的 SAP ASCS/SCS 实例添加到*现有*群集配置。 有关详细信息，请参阅[为现有的群集配置配置附加的 SAP ASCS/SCS 实例以便在 Azure 资源管理器中创建 SAP 多 SID 配置][sap-ha-multi-sid-guide]。

若要新建多 SID 群集，可以使用 [GitHub 上的多 SID 快速入门模板](https://github.com/Azure/azure-quickstart-templates)。

若要新建多 SID 群集，必须部署以下三个模板：

* [ASCS/SCS 模板](#ASCS-SCS-template)
* [数据库模板](#database-template)
* [应用程序服务器模板](#application-servers-template)

以下部分介绍了有关模板的详细信息以及需要在模板中提供的参数。

### <a name="ASCS-SCS-template"></a>ASCS/SCS 模板

ASCS/SCS 模板部署两个虚拟机，可以使用这些虚拟机创建用于托管多个 ASCS/SCS 实例的 Windows Server 故障转移群集。

若要设置 ASCS/SCS 多 SID 模板，请在 [ASCS/SCS 多 SID 模板][sap-templates-3-tier-multisid-xscs-marketplace-image]或[使用托管磁盘的 ASCS/SCS 多 SID 模板][sap-templates-3-tier-multisid-xscs-marketplace-image-md]中，输入以下参数的值：

- **资源前缀**：设置资源前缀，用于给部署期间创建的所有资源添加前缀。 由于资源不只属于一个 SAP 系统，因此资源的前缀并不是某个 SAP 系统的 SID。  前缀的长度必须为 3 到 6 个字符。
- **堆栈类型**：选择 SAP 系统的堆栈类型。 根据堆栈类型，Azure 负载均衡器将为每个 SAP 系统提供一个（ABAP 或仅限 Java）或两个 (ABAP + Java) 专用 IP 地址。
- **OS 类型**：选择虚拟机的操作系统。
- **SAP 系统计数**：选择要在此群集中安装的 SAP 系统数目。
- **系统可用性**：选择“HA”。
- **管理员用户名和管理员密码**：创建可用于登录计算机的新用户。
- **新子网或现有子网**：设置是要创建新的虚拟网络和子网，还是使用现有子网。 如果已有连接到本地网络的虚拟网络，请选择**现有**虚拟网络。
- **子网 ID**：如果要将 VM 部署到现有 VNet 中，并且该 VNet 中已定义了 VM 应分配到的子网，请指定该特定子网的 ID。 ID 通常类似于：

  /subscriptions/\<订阅 ID\>/resourceGroups/\<资源组名称\>/providers/Microsoft.Network/virtualNetworks/\<虚拟网络名称\>/subnets/\<子网名称\>

模板将部署一个支持多个 SAP 系统的 Azure 负载均衡器实例：

- ASCS 实例的实例编号配置为 00、10、20...
- SCS 实例的实例编号配置为 01、11、21...
- ASCS 排队复制服务器 (ERS)（仅限 Linux）实例的实例编号配置为 02、12、22...
- SCS ERS（仅限 Linux）实例的实例编号配置为 03、13、23...

负载均衡器包含 1 个 VIP（在 Linux 中包含 2 个），为 ASCS/SCS 包含 1x 个 VIP，为 ERS（仅适用于 Linux）包含 1x 个 VIP。

#### <a name="0f3ee255-b31e-4b8a-a95a-d9ed6200468b"></a> SAP ASCS/SCS 端口
以下列表包含所有负载均衡规则（其中的 x 是 SAP 系统数目，例如 1、2、3...）：
- 每个 SAP 系统的 Windows 特定端口：445、5985
- ASCS 端口（实例编号 x0）：32x0、36x0、39x0、81x0、5x013、5x014、5x016
- SCS 端口（实例编号 x1）：32x1、33x1、39x1、81x1、5x113、5x114、5x116
- Linux 上的 ASCS ERS 端口（实例编号 x2）：33x2、5x213、5x214、5x216
- Linux 上的 SCS ERS 端口（实例编号 x3）：33x3、5x313、5x314、5x316

负载均衡器配置为使用以下探测端口（其中的 x 是 SAP 系统编号，例如 1、2、3...）：
- ASCS/SCS 内部负载均衡器探测端口：620x0
- ERS 内部负载均衡器探测端口（仅限 Linux）：621x2

### <a name="database-template"></a>数据库模板

数据库模板部署一个或两个虚拟机，可以使用这些虚拟机来为一个 SAP 系统安装关系数据库管理系统 (RDBMS)。 例如，如果为 5 个 SAP 系统部署 ASCS/SCS 模板，则需要部署此模板 5 次。

若要设置数据库多 SID 模板，请在[数据库多 SID 模板][sap-templates-3-tier-multisid-db-marketplace-image]或[使用托管磁盘的数据库多 SID 模板][sap-templates-3-tier-multisid-db-marketplace-image-md]中，输入以下参数的值：

- **SAP 系统 ID**：输入要安装的 SAP 系统的 SAP 系统 ID。 该 ID 将用作所要部署的资源的前缀。
- **OS 类型**：选择虚拟机的操作系统。
- **Dbtype**：选择要在群集上安装的数据库的类型。 若要安装 Microsoft SQL Server，请选择“SQL”。 若要在虚拟机上安装 SAP HANA，请选择“HANA”。 请确保选择正确的操作系统类型。 对于 SQL，请选择“Windows”；对于 HANA，请选择一个 Linux 分发版。 连接到虚拟机的 Azure 负载均衡器被配置为支持选定数据库类型：
  * **SQL**：负载均衡器对端口 1433 进行负载均衡。 对于 SQL Server AlwaysOn 设置，请确保使用此端口。
  * **HANA**：负载均衡器对端口 35015 和 35017 进行负载均衡。 请确保使用实例编号 **50** 安装 SAP HANA。
  负载均衡器使用探测端口 62550。
- **SAP 系统大小**：设置新系统提供的 SAPS 数量。 如果不确定系统需要多少 SAPS，请咨询 SAP 技术合作伙伴或系统集成商。
- **系统可用性**：选择“HA”。
- **管理员用户名和管理员密码**：创建可用于登录计算机的新用户。
- **子网 ID**：输入在部署 ASCS/SCS 模板期间使用的子网的 ID，或部署 ASCS/SCS 模板过程中创建的子网的 ID。

### <a name="application-servers-template"></a> 应用程序服务器模板

应用程序服务器模板部署两个或更多个虚拟机，可将这些虚拟机用作一个 SAP 系统的 SAP 应用程序服务器实例。 例如，如果为 5 个 SAP 系统部署 ASCS/SCS 模板，则需要部署此模板 5 次。

若要设置应用程序服务器多 SID 模板，请在[应用程序服务器多 SID 模板][sap-templates-3-tier-multisid-apps-marketplace-image]或[使用托管磁盘的应用程序服务器多 SID 模板][sap-templates-3-tier-multisid-apps-marketplace-image-md]中，输入以下参数的值：

  -  **SAP 系统 ID**：输入要安装的 SAP 系统的 SAP 系统 ID。 该 ID 将用作所要部署的资源的前缀。
  -  **OS 类型**：选择虚拟机的操作系统。
  -  **SAP 系统大小**：新系统提供的 SAPS 的数量。 如果不确定系统需要多少 SAPS，请咨询 SAP 技术合作伙伴或系统集成商。
  -  **系统可用性**：选择“HA”。
  -  **管理员用户名和管理员密码**：创建可用于登录计算机的新用户。
  -  **子网 ID**：输入在部署 ASCS/SCS 模板期间使用的子网的 ID，或部署 ASCS/SCS 模板过程中创建的子网的 ID。


## <a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure 虚拟网络
在本例中，Azure 虚拟网络实例的地址空间是 10.0.0.0/16。 有一个名为 Subnet、地址范围为 10.0.0.0/24 的子网。 所有虚拟机和内部负载均衡器都部署在此虚拟网络中。

> [!IMPORTANT]
> 不要在来宾操作系统内对网络设置进行任何更改。 这包括 IP 地址、DNS 服务器和子网。 在 Azure 中配置所有网络设置。 动态主机配置协议 (DHCP) 服务将传播设置。
>
>

## <a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a>DNS IP 地址

若要设置所需的 DNS IP 地址，请完成以下步骤：

1. 在 Azure 门户上的“DNS 服务器”窗格中，确保虚拟网络的“DNS 服务器”选项设置为“自定义 DNS”。
2. 根据使用的网络类型选择设置。 有关详细信息，请参阅以下资源：
   * [企业网络连接（跨界）][planning-guide-2.2]：添加本地 DNS 服务器的 IP 地址。  
   可将本地 DNS 服务器扩展到正在 Azure 中运行的虚拟机。 在该情况下，可添加运行 DNS 服务器的 Azure 虚拟机的 IP 地址。
   * 对于在 Azure 中隔离的 VM 部署：在充当 DNS 服务器的同一虚拟网络实例中部署其他虚拟机。 添加已设置为运行 DNS 服务的 Azure 虚拟机的 IP 地址。

   ![图 2：为 Azure 虚拟网络配置 DNS 服务器][sap-ha-guide-figure-3001]

   _**图 2：** 为 Azure 虚拟网络配置 DNS 服务器_

   > [!NOTE]
   > 如果更改了 DNS 服务器的 IP 地址，则需要重新启动 Azure 虚拟机才能应用更改并传播新的 DNS 服务器。
   >
   >

在本例中，已在以下 Windows 虚拟机上安装并配置 DNS 服务：

| 虚拟机角色 | 虚拟机主机名 | 网卡名称 | 静态 IP 地址 |
| --- | --- | --- | --- |
| 第一台 DNS 服务器 |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| 第二台 DNS 服务器 |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

## <a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a>SAP ASCS/SCS 群集实例和 DBMS 群集实例的主机名与静态 IP 地址

对于本地部署，需要这些保留的主机名和 IP 地址：

| 虚拟主机名角色 | 虚拟主机名 | 虚拟静态 IP 地址 |
| --- | --- | --- |
| SAP ASCS/SCS 的第一个群集虚拟主机名（用于群集管理） |pr1-ascs-vir |10.0.0.42 |
| SAP ASCS/SCS 实例虚拟主机名 |pr1-ascs-sap |10.0.0.43 |
| SAP DBMS 第二个群集虚拟主机名（群集管理） |pr1-dbms-vir |10.0.0.32 |

创建群集时，请创建虚拟主机名 pr1-ascs-vir 和 pr1-dbms-vir 以及管理群集本身的关联 IP 地址。 有关如何执行此操作的信息，请参阅[收集群集配置中的群集节点][sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]。

可以在 DNS 服务器上手动创建另外两个虚拟主机 pr1-ascs-sap 和 pr1-dbms-sap 以及关联的 IP 地址。 群集 SAP ASCS/SCS 实例和群集 DBMS 实例使用以下资源。 有关如何执行此操作的信息，请参阅[为 SAP ASCS/SCS 群集实例创建虚拟主机名][sap-ha-guide-9.1.1]。

## <a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> 设置 SAP 虚拟机的静态 IP 地址
部署要在群集中使用的虚拟机之后，需要为所有虚拟机设置静态 IP 地址。 请在 Azure 虚拟网络配置中而不是来宾操作系统中执行此操作。

1. 在 Azure 门户中，选择“资源组” > “网卡” > “设置” > “IP 地址”。
2. 在“IP 地址”窗格中的“分配”下面，选择“静态”。 在“IP 地址”框中，输入要使用的 IP 地址。

   > [!NOTE]
   > 如果更改了网卡的 IP 地址，则需要重新启动 Azure 虚拟机才能应用更改。  
   >
   >

   ![图 3：为每个虚拟机的网卡设置静态 IP 地址][sap-ha-guide-figure-3002]

   _**图 3：** 为每个虚拟机的网卡设置静态 IP 地址_

   为所有网络接口（即为所有虚拟机，包括要用于 Active Directory 或 DNS 服务的虚拟机）重复此步骤。

本例使用以下虚拟机和静态 IP 地址：

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

## <a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a>为 Azure 内部负载均衡器设置静态 IP 地址

SAP Azure 资源管理器模板创建用于 SAP ASCS/SCS 实例群集和 DBMS 群集的 Azure 内部负载均衡器。

> [!IMPORTANT]
> SAP ASCS/SCS 的虚拟主机名的 IP 地址与 SAP ASCS/SCS 内部负载均衡器 pr1-lb-ascs 的 IP 地址相同。
> DBMS 的虚拟名称的 IP 地址与 DBMS 内部负载均衡器 pr1-lb-dbms 的 IP 地址相同。
>
>

为 Azure 内部负载均衡器设置静态 IP 地址：

1. 初始部署将内部负载均衡器 IP 地址设置为“动态”。 在 Azure 门户的“IP 地址”窗格中的“分配”下面，选择“静态”。
2. 将内部负载均衡器 **pr1-lb-ascs** 的 IP 地址设置为 SAP ASCS/SCS 实例的虚拟主机名 IP 地址。
3. 将内部负载均衡器 **pr1-lb-dbms** 的 IP 地址设置为 DBMS 实例的虚拟主机名 IP 地址。

   ![图 4：为 SAP ASCS/SCS 实例的内部负载均衡器设置静态 IP 地址][sap-ha-guide-figure-3003]

   _**图 4：** 为 SAP ASCS/SCS 实例的内部负载均衡器设置静态 IP 地址_

在本例中，有两个具有以下静态 IP 地址的 Azure 内部负载均衡器：

| Azure 内部负载均衡器角色 | Azure 内部负载均衡器名称 | 静态 IP 地址 |
| --- | --- | --- |
| SAP ASCS/SCS 实例的内部负载均衡器 |pr1-lb-ascs |10.0.0.43 |
| SAP DBMS 内部负载均衡器 |pr1-lb-dbms |10.0.0.33 |


## <a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a>Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则

SAP Azure 资源管理器模板创建所需的端口：
* 一个 ABAP ASCS 实例，默认实例编号为 00
* 一个 Java SCS 实例，默认实例编号为 01

安装 SAP ASCS/SCS 实例时，必须为 ABAP ASCS 实例使用默认实例编号 00，为 Java SCS 实例使用默认实例编号 01。

接下来，为 SAP NetWeaver 端口创建所需的内部负载均衡终结点：

若要创建所需的内部负载均衡终结点，请先为 SAP NetWeaver ABAP ASCS 端口创建这些负载均衡终结点：

| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 00 的 ASCS 实例）（ERS 为 10）的具体端口 |
| --- | --- | --- |
| 排队服务器 / *lbrule3200* |32\<InstanceNumber\> |3200 |
| ABAP 消息服务器 / *lbrule3600* |36\<InstanceNumber\> |3600 |
| 内部 ABAP 消息 / *lbrule3900* |39\<InstanceNumber\> |3900 |
| 消息服务器 HTTP / *Lbrule8100* |81\<InstanceNumber\> |8100 |
| SAP 启动服务 ASCS HTTP / *Lbrule50013* |5\<InstanceNumber\>13 |50013 |
| SAP 启动服务 ASCS HTTPS / *Lbrule50014* |5\<InstanceNumber\>14 |50014 |
| 排队复制 / *Lbrule50016* |5\<InstanceNumber\>16 |50016 |
| SAP 启动服务 ERS HTTP *Lbrule51013* |5\<InstanceNumber\>13 |51013 |
| SAP 启动服务 ERS HTTP *Lbrule51014* |5\<InstanceNumber\>14 |51014 |
| Windows 远程管理 (WinRM) *Lbrule5985* | |5985 |
| 文件共享 *Lbrule445* | |445 |

**表 1：** SAP NetWeaver ABAP ASCS 实例的端口号

然后，为 SAP NetWeaver Java SCS 端口创建这些负载均衡终结点：

| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 01 的 SCS 实例）（ERS 为 11）的具体端口 |
| --- | --- | --- |
| 排队服务器 / *lbrule3201* |32\<InstanceNumber\> |3201 |
| 网关服务器 / *lbrule3301* |33\<InstanceNumber\> |3301 |
| Java 消息服务器 / *lbrule3900* |39\<InstanceNumber\> |3901 |
| 消息服务器 HTTP / *Lbrule8101* |81\<InstanceNumber\> |8101 |
| SAP 启动服务 SCS HTTP / *Lbrule50113* |5\<InstanceNumber\>13 |50113 |
| SAP 启动服务 SCS HTTPS / *Lbrule50114* |5\<InstanceNumber\>14 |50114 |
| 排队复制 / *Lbrule50116* |5\<InstanceNumber\>16 |50116 |
| SAP 启动服务 ERS HTTP *Lbrule51113* |5\<InstanceNumber\>13 |51113 |
| SAP 启动服务 ERS HTTP *Lbrule51114* |5\<InstanceNumber\>14 |51114 |
| WinRM *Lbrule5985* | |5985 |
| 文件共享 *Lbrule445* | |445 |

**表 2：** SAP NetWeaver Java SCS 实例的端口号

![图 5：Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则][sap-ha-guide-figure-3004]

_**图 5：** Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则_

将负载均衡器 pr1-lb-dbms 的 IP 地址设置为 DBMS 实例的虚拟主机名 IP 地址。

### <a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a>更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则

如果想要将其他编号用于 SAP ASCS 或 SCS 实例，必须更改这些实例的名称和默认值。

1. 在 Azure 门户中，转到“**\<SID\>-lb-ascs 负载均衡器”** > “负载均衡规则”。
2. 对于属于 SAP ASCS 或 SCS 实例的所有负载均衡规则，请更改以下值：

   * 名称
   * Port
   * 后端端口

   例如，如果要将默认 ASCS 实例编号从 00 更改为 31，需要为表 1 中列出的所有端口进行更改。

   下面是端口 *lbrule3200* 的更新的示例。

   ![图 6：更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则][sap-ha-guide-figure-3005]

   _**图 6：** 更改 Azure 内部负载均衡器的 ASCS/SCS 默认负载均衡规则_

## <a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a>将 Windows 虚拟机添加到域

将静态 IP 地址分配到虚拟机后，请将虚拟机添加到域。

![图 7：将虚拟机添加到域][sap-ha-guide-figure-3006]

_**图 7：** 将虚拟机添加到域_

## <a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a>在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项

Azure 负载均衡器具有内部负载均衡器，可在连接在一段固定时间内处于空闲状态时（空闲超时）关闭连接。 对话实例中的 SAP 工作进程在需要发送第一个排队/取消排队请求时立即打开与 SAP 排队进程的连接。 这些连接通常持续保持已建立状态，直到工作进程或排队进程重新启动为止。 但是，如果连接空闲了设置的一段时间，Azure 内部负载均衡器会关闭连接。 这不是问题，因为如果连接不再存在，SAP 工作进程会重新建立与排队进程的连接。 这些活动记录在 SAP 进程的开发人员跟踪中，但它们会在这些跟踪中创建大量额外内容。 在两个群集节点上更改 TCP/IP `KeepAliveTime` 和 `KeepAliveInterval` 是一个好办法。 将 TCP/IP 参数中的这些更改与 SAP 配置文件参数组合起来，文章后面部分有介绍。

若要在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项，请先在 SAP ASCS/SCS 的两个 Windows 群集节点上添加以下 Windows 注册表项：

| 路径 | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 变量名称 |`KeepAliveTime` |
| 变量类型 |REG_DWORD（十进制） |
| 值 |120000 |
| 文档链接 |[https://technet.microsoft.com/library/cc957549.aspx](https://technet.microsoft.com/library/cc957549.aspx) |

**表 3：** 更改第一个 TCP/IP 参数

然后，在 SAP ASCS/SCS 的两个 Windows 群集节点上添加以下 Windows 注册表项：

| 路径 | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| 变量名称 |`KeepAliveInterval` |
| 变量类型 |REG_DWORD（十进制） |
| 值 |120000 |
| 文档链接 |[https://technet.microsoft.com/library/cc957548.aspx](https://technet.microsoft.com/library/cc957548.aspx) |

**表 4：** 更改第二个 TCP/IP 参数

若要应用更改，请重新启动两个群集节点。

## <a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> 为 SAP ASCS/SCS 实例设置 Windows Server 故障转移群集

为 SAP ASCS/SCS 实例设置 Windows Server故障转移群集的过程包括以下任务：

- 在群集配置中收集群集节点。
- 配置群集文件共享见证。

### <a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a>收集群集配置中的群集节点

1. 在“添加角色和功能向导”中，将故障转移群集功能添加到两个群集节点。
2. 使用故障转移群集管理器设置故障转移群集。 在故障转移群集管理器中，选择“创建群集”，仅添加第一个群集（节点 A）的名称。 暂时不要添加第二个节点；将在稍后的步骤中添加第二个节点。

   ![图 8：添加第一个群集节点的服务器或虚拟机名称][sap-ha-guide-figure-3007]

   _**图 8：** 添加第一个群集节点的服务器或虚拟机名称_

3. 输入群集的网络名称（虚拟主机名）。

   ![图 9：输入群集名称][sap-ha-guide-figure-3008]

   _**图 9：** 输入群集名称_

4. 创建群集后，运行群集验证测试。

   ![图 10：运行群集验证检查][sap-ha-guide-figure-3009]

   _**图 10：** 运行群集验证检查_

   在过程中的此时点，可忽略关于磁盘的任何警告。 将稍后添加文件共享见证和 SIOS 共享磁盘。 在此阶段，无需担心是否具有仲裁。

   ![图 11：找不到任何仲裁磁盘][sap-ha-guide-figure-3010]

   _**图 11：** 找不到任何仲裁磁盘_

   ![图 12：核心群集资源需要新 IP 地址][sap-ha-guide-figure-3011]

   _**图 12：** 核心群集资源需要新 IP 地址_

5. 更改核心群集服务的 IP 地址。 由于服务器的 IP 地址指向虚拟机节点之一，因此，在更改核心群集服务的 IP 地址之前，群集无法启动。 请在核心群集服务的 IP 资源“属性”页上执行此操作。

   例如，需要为群集虚拟主机名 pr1-ascs-vir 分配 IP 地址（在本例中为 10.0.0.42）。

   ![图 13：在“属性”对话框中更改 IP 地址][sap-ha-guide-figure-3012]

   _**图 13：** 在“属性”对话框中更改 IP 地址_

   ![图 14：分配为群集保留的 IP 地址][sap-ha-guide-figure-3013]

   _**图 14：** 分配为群集保留的 IP 地址_

6. 将群集虚拟主机名联机。

   ![图 15：群集核心服务保持正常运行，使用正确的 IP 地址][sap-ha-guide-figure-3014]

   _**图 15：** 群集核心服务保持正常运行，使用正确的 IP 地址_

7. 添加第二个群集节点。

   核心群集服务启动并运行后，可以添加第二个群集节点。

   ![图 16：添加第二个群集节点][sap-ha-guide-figure-3015]

   _**图 16：** 添加第二个群集节点_

8. 输入第二个群集节点主机的名称。

   ![图 17：输入第二个群集节点主机名][sap-ha-guide-figure-3016]

   _**图 17：** 输入第二个群集节点主机名_

   > [!IMPORTANT]
   > 请确保“将所有符合条件的存储添加到群集”复选框*不*处于选中状态。  
   >
   >

   ![图 18：请勿选中该复选框][sap-ha-guide-figure-3017]

   _**图 18：***请勿* 选中该复选框_

   可以忽略关于仲裁和磁盘的警告。 将在稍后设置仲裁和共享磁盘，如[为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition][sap-high-availability-infrastructure-wsfc-shared-disk-install-sios] 中所述。

   ![图 19：忽略关于磁盘仲裁的警告][sap-ha-guide-figure-3018]

   _**图 19：** 忽略关于磁盘仲裁的警告_


#### <a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a>配置群集文件共享见证

配置群集文件共享见证的过程包括以下任务：

- 创建文件共享。
- 在故障转移群集管理器中设置文件共享见证仲裁。

#### <a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a>创建文件共享

1. 请选择文件共享见证而不是仲裁磁盘。 SIOS DataKeeper 支持此选项。

   在本文的示例中，文件共享见证在正在 Azure 中运行的 Active Directory 或 DNS 服务器上。 文件共享见证称为 domcontr-0。 由于已配置到 Azure 的 VPN 连接（通过 VPN 网关或 Azure ExpressRoute），因此 Active Directory 或 DNS 服务在本地，并且不适合运行文件共享见证。

   > [!NOTE]
   > 如果 Active Directory 或 DNS 服务仅在本地运行，请不要在本地运行的 Active Directory 或 DNS Windows 操作系统中配置文件共享见证。 在 Azure 中运行的群集节点和本地 Active Directory 或 DNS 之间的网络延迟可能过大，从而导致连接问题。 请确保在靠近群集节点运行的 Azure 虚拟机上配置文件共享见证。  
   >
   >

   仲裁驱动器至少需要 1,024 MB 可用空间。 建议为仲裁驱动器提供 2,048 MB 可用空间。

2. 添加群集名称对象。

   ![图 20：为群集名称对象分配共享权限][sap-ha-guide-figure-3019]

   _**图 20：** 为群集名称对象分配共享权限_

   请确保权限包括针对群集名称对象（在本例中为 pr1-ascs-vir$）更改共享中的数据的授权。

3. 要将群集名称添加到列表，请选择“添加”。 更改筛选器，以便除了检查图 22 中所示的项以外，还检查计算机对象。

   ![图 21：将“对象类型”更改为包括计算机][sap-ha-guide-figure-3020]

   _**图 21：** 将“对象类型”更改为包括计算机_

   ![图 22：选中“计算机”复选框][sap-ha-guide-figure-3021]

   _**图 22：** 选中“计算机”复选框_

4. 输入群集名称对象，如图 21 所示。 由于已创建记录，因此可更改权限，如图 20 所示。

5. 选择共享的“安全性”选项卡，并为群集名称对象设置更详细的权限。

   ![图 23：为文件共享仲裁上的群集名称对象设置安全属性][sap-ha-guide-figure-3022]

   _**图 23：** 为文件共享仲裁上的群集名称对象设置安全属性_

#### <a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a>在故障转移群集管理器中设置文件共享见证仲裁

1. 打开“配置仲裁设置向导”。

   ![图 24：启动“配置群集仲裁设置向导”][sap-ha-guide-figure-3023]

   _**图 24：** 启动“配置群集仲裁设置向导”_

2. 上“选择仲裁配置选项”页上，选择“选择仲裁见证”。

   ![图 25：可供选择的仲裁配置][sap-ha-guide-figure-3024]

   _**图 25：** 可供选择的仲裁配置_

3. 在“选择仲裁见证”页上，选择“配置文件共享见证”。

   ![图 26：选择文件共享见证][sap-ha-guide-figure-3025]

   _**图 26：** 选择文件共享见证_

4. 输入文件共享的 UNC 路径（在本例中为 \\domcontr-0\FSW）。 若要查看可进行的更改列表，请选择“下一步”。

   ![图 27：定义见证共享的文件共享位置][sap-ha-guide-figure-3026]

   _**图 27：** 定义见证共享的文件共享位置_

5. 选择所需的更改，并选择“下一步”。 需要成功重新配置群集配置，如图 28 中所示：  

   ![图 28：确认已重新配置群集][sap-ha-guide-figure-3027]

   _**图 28：** 确认已重新配置群集_

成功安装 Windows 故障转移群集后，需要更改某些阈值，以使故障转移检测适合 Azure 中的情况。 博客文章 [Tuning failover cluster network thresholds][tuning-failover-cluster-network-thresholds]（调整故障转移群集网络阈值）中阐述了要更改的参数。 假设构成 ASCS/SCS 的 Windows 群集配置的两个 VM 位于同一子网中，则需要将以下参数更改为这些值：

- SameSubNetDelay = 2000
- SameSubNetThreshold = 15
- RoutingHistoryLength = 30

这些设置已经过客户测试，可以提供合理的折衷。 它们不仅具有足够的弹性，而且在发生 SAP 软件、节点或 VM 故障时，在实际出错情况下，这些设置的故障转移速度够快。

### <a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a>为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition

现在，已在 Azure 中创建有效的 Windows Server 故障转移群集配置。 若要安装 SAP ASCS/SCS 实例，需要一个共享磁盘资源。 无法在 Azure 中创建所需的共享磁盘资源。 SIOS DataKeeper Cluster Edition 是可用于创建共享磁盘资源的第三方解决方案。

为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition 的过程包括以下任务：

- 添加 Microsoft .NET Framework 3.5。
- 安装 SIOS DataKeeper。
- 设置 SIOS DataKeeper。

### <a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a>添加 .NET Framework 3.5
Windows Server 2012 R2 上不会自动激活或安装 .NET Framework 3.5。 由于 SIOS DataKeeper 要求安装 DataKeeper 的所有节点上都包含 .NET，因此，必须在群集中所有虚拟机的来宾操作系统上安装 .NET Framework 3.5。

可通过两种方法添加 .NET Framework 3.5：

- 使用 Windows 中的“添加角色和功能向导”，如图 29 所示：

  ![图 29：使用“添加角色和功能向导”安装 .NET Framework 3.5][sap-ha-guide-figure-3028]

  _**图 29：** 使用“添加角色和功能向导”安装 .NET Framework 3.5_

  ![图 30：使用“添加角色和功能向导”安装 .NET Framework 3.5 时的安装进度栏][sap-ha-guide-figure-3029]

  _**图 30：** 使用“添加角色和功能向导”安装 .NET Framework 3.5 时的安装进度栏_

- 使用 dism.exe 命令行工具。 对于此类型的安装，需要访问 Windows 安装媒体上的 SxS 目录。 在权限提升的命令提示符下输入以下命令：

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

### <a name="dd41d5a2-8083-415b-9878-839652812102"></a>安装 SIOS DataKeeper

在群集中的每个节点上安装 SIOS DataKeeper Cluster Edition。 要创建包含 SIOS DataKeeper 的虚拟共享存储，请创建同步的镜像，并模拟群集共享存储。

在安装 SIOS 软件之前，请创建 DataKeeperSvc 域用户。

> [!NOTE]
> 将 DataKeeperSvc 域用户添加到两个群集节点上的“本地管理员”组中。
>
>

安装 SIOS DataKeeper：

1. 在两个群集节点上安装 SIOS 软件。

   ![SIOS 安装程序][sap-ha-guide-figure-3030]

   ![图 31：SIOS DataKeeper 安装程序的第一页][sap-ha-guide-figure-3031]

   _**图 31：** SIOS DataKeeper 安装程序的第一页_

2. 在对话框中，选择“是”。

   ![图 32：DataKeeper 通知将会禁用服务][sap-ha-guide-figure-3032]

   _**图 32：** DataKeeper 通知将会禁用服务_

3. 在对话框中，建议选中“域或服务器帐户”。

   ![图 33：用户为 SIOS DataKeeper 选中的选项][sap-ha-guide-figure-3033]

   _**图 33：** 用户为 SIOS DataKeeper 选中的选项_

4. 输入为 SIOS DataKeeper 创建的域帐户用户名和密码。

   ![图 34：输入域用户名和密码以安装 SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**图 34：** 输入域用户名和密码以安装 SIOS DataKeeper_

5. 如图 35 所示，安装 SIOS DataKeeper 实例的许可证密钥。

   ![图 35：输入 SIOS DataKeeper 许可证密钥][sap-ha-guide-figure-3035]

   _**图 35：** 输入 SIOS DataKeeper 许可证密钥_

6. 根据提示重新启动虚拟机。

### <a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a>设置 SIOS DataKeeper

在两个节点上安装 SIOS DataKeeper 之后，开始进行配置。 配置的目的是在连接到每个虚拟机的附加磁盘之间进行同步数据复制。

1. 启动 DataKeeper 管理和配置工具，并选择“连接服务器”。

   ![图 36：SIOS DataKeeper 管理和配置工具][sap-ha-guide-figure-3036]

   _**图 36：** SIOS DataKeeper 管理和配置工具_

2. 输入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，插入第二个节点的相关数据。

   ![图 37：插入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，插入第二个节点的相关数据][sap-ha-guide-figure-3037]

   _**图 37：** 插入管理和配置工具应连接到的第一个节点的名称或 TCP/IP 地址，并在第二个步骤中，插入第二个节点的相关数据_

3. 在两个节点之间创建复制作业。

   ![图 38：创建复制作业][sap-ha-guide-figure-3038]

   _**图 38：** 创建复制作业_

   向导指导完成创建复制作业的过程。

4. 定义复制作业的名称。

   ![图 39：定义复制作业的名称][sap-ha-guide-figure-3039]

   _**图 39：** 定义复制作业的名称_

   ![图 40：定义节点（应为当前源节点）的基本数据][sap-ha-guide-figure-3040]

   _**图 40：** 定义节点（应为当前源节点）的基本数据_

5. 定义目标节点的名称、TCP/IP 地址和磁盘卷。

   ![图 41：定义当前目标节点的名称、TCP/IP 地址和磁盘卷][sap-ha-guide-figure-3041]

   _**图 41：** 定义当前目标节点的名称、TCP/IP 地址和磁盘卷_

6. 定义压缩算法。 在本例中，建议压缩复制流。 尤其是在重新同步的情况下，压缩复制流可大幅缩短重新同步的时间。 压缩需要占用虚拟机的 CPU 和 RAM 资源。 随着压缩率增加，CPU 资源量也增加。 稍后可调整此设置。

7. 需要检查的另一项设置是复制以异步还是同步方式发生。 保护 SAP ASCS/SCS 配置时，必须使用同步复制。  

   ![图 42：定义复制详细信息][sap-ha-guide-figure-3042]

   _**图 42：** 定义复制详细信息_

8. 定义是否应向 Windows Server 故障转移群集配置将复制作业所复制的卷表示为共享磁盘。 对于 SAP ASCS/SCS 配置，选择“是”，以便 Windows 群集将复制的卷视为可用作群集卷的共享磁盘。

   ![图 43：选择“是”将复制的卷设置为群集卷][sap-ha-guide-figure-3043]

   _**图 43：** 选择“是”将复制的卷设置为群集卷_

   创建卷后，DataKeeper 管理和配置工具显示复制作业处于活动状态。

   ![图 44：SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态][sap-ha-guide-figure-3044]

   _**图 44：** SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态_

   现在，故障转移群集管理器将磁盘显示为 DataKeeper 磁盘，如图 45 所示：

   ![图 45：故障转移群集管理器显示 DataKeeper 复制的磁盘][sap-ha-guide-figure-3045]

   _**图 45：** 故障转移群集管理器显示 DataKeeper 复制的磁盘_

## <a name="next-steps"></a>后续步骤

* [使用 Windows 故障转移群集和共享磁盘为 SAP ASCS/SCS 实例安装 SAP NetWeaver HA][sap-high-availability-installation-wsfc-shared-disk]
