---
title: 在 Azure Vm 上的 SAP 入门 |Microsoft Docs
description: 了解如何在 Microsoft Azure 中的虚拟机 (Vm) 运行的 SAP 解决方案
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/30/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e637b52489bb92b66e185d37db16f7004bf47537
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147750"
---
# <a name="use-azure-to-host-and-run-sap-workload-scenarios"></a>使用 Azure 来托管和运行 SAP 工作负荷方案

当你使用 Microsoft Azure 时，可以可靠地可缩放、 合规且经企业认证的平台上运行你任务关键型 SAP 工作负荷和方案。 获取的可伸缩性、 灵活性和 Azure 的成本节约。 Microsoft 和 SAP 之间扩大了合作伙伴关系，您可以通过在 Azure 中开发和测试和生产方案运行 SAP 应用程序并获得完全支持。 从 SAP NetWeaver 到 SAP S/4HANA，SAP BI Linux 到 Windows，和 SAP HANA 到 SQL，我们必须将你的需求。

除了托管与 Azure 上的不同 DBMS 的 SAP NetWeaver 方案，你可以托管其他 SAP 工作负荷方案，在 Azure 上的 SAP BI 等。 

Azure 的 SAP HANA 的唯一性是分开设置 Azure 的产品/服务。 若要启用托管更多内存和 CPU 资源要求苛刻的 SAP 方案涉及 SAP HANA，Azure 提供客户专用裸机硬件的使用。 使用此解决方案来运行用于 S/4HANA 或其他 SAP HANA 工作负荷需要最多 24 TB （120 TB 横向扩展） 的内存的 SAP HANA 部署。 

托管在 Azure 中的 SAP 工作负荷方案还可以创建标识集成和单一登录的要求。 使用 Azure Active Directory (Azure AD) 连接不同的 SAP 组件和 SAP 软件作为-服务 (SaaS) 或平台即服务 (PaaS) 产品/服务时，可能出现这种情况。 此类集成和与 Azure AD 单一登录方案的列表和描述并在"AAD SAP 标识集成和单一登录。"部分中所述 SAP 实体

## <a name="latest-changes"></a>最新的更改

- 新版[Azure HANA 大型实例控制通过 Azure 门户](hana-li-portal.md)

- 新版[使用 Azure NetApp 文件的 SAP 应用程序的 SUSE Linux Enterprise Server 上的 Azure Vm 上的 SAP NetWeaver 的高可用性](high-availability-guide-suse-netapp-files.md)

- Clarification on 澄清**Linux OS 参数 net.ipv4.tcp_timestamps**设置与 Azure 结合使用负载均衡器

- 新版[与 Azure 可用性区域的 SAP 工作负荷配置](sap-ha-availability-zones.md)

- 发布 [SAP 工作负荷规划和部署清单](sap-deployment-checklist.md)




## <a name="sap-hana-on-azure-large-instances"></a>Azure 上的 SAP HANA（大型实例）

一系列文档将引导你完成 SAP HANA on Azure （大型实例），或简称为 HANA 大型实例。 HANA 大型实例的以下几个方面的信息，请参阅：

- [Azure 上的 SAP HANA（大型实例）的概述](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)
- [Azure 上的 SAP HANA（大型实例）体系结构](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-architecture)
- [基础结构和连接到 Azure （大型实例） 上的 SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity)
- [在 Azure （大型实例） 上安装 SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-installation)
- [Azure （大型实例） 上的 SAP HANA 的高可用性和灾难恢复](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery)
- [疑难解答和监视 Azure （大型实例） 上的 SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/troubleshooting-monitoring)

后续步骤：

- 读取[概述和体系结构的 SAP HANA on Azure （大型实例）](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)



## <a name="sap-hana-on-azure-virtual-machines"></a>Azure 虚拟机上的 SAP HANA
文档的此部分介绍 SAP HANA 的不同方面。 作为先决条件，你应熟悉的主体的 Azure 服务提供的 Azure IaaS 的基本服务。 因此，您需了解 Azure 计算、 存储和网络。 许多这些使用者处理中的 SAP NetWeaver 相关[Azure 规划指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)。 

在 Azure 上的 HANA 上的信息，请参阅以下文章和其 subarticles:

- [快速入门：在 Azure VM 上手动安装单实例 SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-get-started)
- [在 Azure 上部署 SAP S/4HANA 或 BW/4HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-s4h)
- [Azure 上的 SAP HANA 基础结构配置和操作](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)
- [Azure 虚拟机的 SAP HANA 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-overview)
- [一个 Azure 区域内的 SAP HANA 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-one-region)
- [跨 Azure 区域的 SAP HANA 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-across-regions)
- [Azure 虚拟机上的 SAP HANA 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability)
- [Azure 虚拟机上的 SAP HANA 备份指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)
- [文件级别的 SAP HANA Azure 备份](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-file-level)
- [基于存储快照的 SAP HANA 备份](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-storage-snapshots)

 

## <a name="sap-netweaver-deployed-on-azure-virtual-machines"></a>部署 Azure 虚拟机上的 SAP NetWeaver
本部分列出的 SAP NetWeaver 和 Business One 在 Azure 上的规划和部署文档。 文档重点介绍基础知识和非 HANA 数据库使用与在 Azure 上的 SAP 工作负荷。 文档和以实现高可用性的文章也是在 Azure 中，HANA 高可用性的基础，如：

- [SAP Business One Azure 虚拟机上](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/business-one-azure)
- [在 Azure 上为 SAP ERP 6.0 部署 SAP IDES EHP7 SP3](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-ides-erp6-erp7-sp3-sql)
- [Microsoft Azure SUSE Linux Vm 上运行的 SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/suse-quickstart)
- [SAP NetWeaver 的 Azure 虚拟机规划和实施指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)
- [SAP NetWeaver 的 Azure 虚拟机部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/deployment-guide)
- [使用 Site Recovery 保护多层 SAP NetWeaver 应用程序部署](https://docs.microsoft.com/azure/site-recovery/site-recovery-sap)
- [适用于 Azure 的 SAP LaMa 连接器](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/lama-installation)

在 Azure 上的 SAP 工作负荷下的非 HANA 数据库的信息，请参阅：

- [部署适用于 SAP 工作负荷的 Azure 虚拟机 DBMS 的注意事项](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)
- [适用于 SAP NetWeaver 的 SQL Server Azure 虚拟机 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_sqlserver)
- [适用于 SAP 工作负荷的 Oracle Azure 虚拟机 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_oracle)
- [适用于 SAP 工作负荷的 IBM DB2 Azure 虚拟机 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm)
- [适用于 SAP 工作负荷的 SAP ASE Azure 虚拟机 DBMS 部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_sapase)
- [在 Azure Vm 上的 SAP MaxDB、 Live 缓存和内容服务器部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_maxdb)

有关 Azure 上的 SAP HANA 数据库的信息，请参阅部分"Azure 虚拟机上的 SAP HANA"。

在 Azure 上的 SAP 工作负荷的高可用性的信息，请参阅：

- [Azure 虚拟机上 SAP NetWeaver 的高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-guide-start)

本文档将指向其他各种体系结构和方案的文档。 在更高版本的方案文档中提供了指向介绍部署和配置不同的高可用性方法的详细技术文档。 显示如何建立和配置 SAP NetWeaver 工作负荷的高可用性的其他文档介绍 Linux 和 Windows 操作系统。


有关 Azure Active Directory (Azure AD) 和 SAP 服务和单一登录之间的集成的信息，请参阅：

- [教程：将 Azure Active Directory 与 SAP Cloud for Customer 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-customer-cloud-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [教程：将 Azure Active Directory 与 SAP Cloud Platform Identity Authentication 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [教程：将 Azure Active Directory 与 SAP Cloud Platform 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [教程：将 Azure Active Directory 与 SAP NetWeaver 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [教程：将 Azure Active Directory 与 SAP Business ByDesign 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/sapbusinessbydesign-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [教程：将 Azure Active Directory 与 SAP HANA 集成](https://docs.microsoft.com/azure/active-directory/saas-apps/saphana-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [S/4HANA 环境：Fiori Launchpad SAML 单一登录与 Azure AD](https://blogs.sap.com/2017/02/20/your-s4hana-environment-part-7-fiori-launchpad-saml-single-sing-on-with-azure-ad/)

有关与 SAP 组件的 Azure 服务集成的信息，请参阅：

- [在 Power BI Desktop 中使用 SAP HANA](https://docs.microsoft.com/power-bi/desktop-sap-hana)
- [DirectQuery 和 SAP HANA](https://docs.microsoft.com/power-bi/desktop-directquery-sap-hana)
- [在 Power BI Desktop 中使用 SAP BW 连接器](https://docs.microsoft.com/power-bi/desktop-sap-bw-connector) 
- [Azure 数据工厂提供 SAP HANA 和业务数据仓库数据集成](https://azure.microsoft.com/blog/azure-data-factory-offer-sap-hana-and-business-warehouse-data-integration)




