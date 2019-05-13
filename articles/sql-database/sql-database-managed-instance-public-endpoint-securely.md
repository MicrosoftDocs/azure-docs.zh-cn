---
title: 保护公共终结点托管的实例-Azure SQL 数据库托管的实例 |Microsoft Docs
description: 在 Azure 中安全地使用公共终结点，与托管实例
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto, carlrab
manager: craigg
ms.date: 05/08/2019
ms.openlocfilehash: f06677b66c8f6586fec8cc5dfe97b1515b741e9c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470280"
---
# <a name="use-an-azure-sql-database-managed-instance-securely-with-public-endpoints"></a>使用公共终结点安全的方式使用 Azure SQL 数据库托管的实例

Azure SQL 数据库托管的实例可以通过提供用户连接[公共终结点](../virtual-network/virtual-network-service-endpoints-overview.md)。 此文章介绍了如何进行此配置更安全。

## <a name="scenarios"></a>方案

SQL 数据库托管的实例提供了专用终结点以允许来自其虚拟网络中的连接。 默认选项是提供最大的隔离。 但是，有需要提供的公共终结点连接的方案：

- 托管的实例必须与多-租户仅限的平台即服务 (PaaS) 产品/服务集成。
- 您需要的数据交换时使用 VPN 相比，更高的吞吐量。
- 公司策略禁止在企业网络内的 PaaS。

## <a name="deploy-a-managed-instance-for-public-endpoint-access"></a>部署公共终结点访问的托管的实例

尽管不是必需的但具有公共终结点访问的托管实例的常见部署模型是专用的独立虚拟网络中创建实例。 在此配置中，使用虚拟网络，仅对虚拟群集隔离。 并不重要的托管的实例的 IP 地址空间与公司网络的 IP 地址空间重叠。

## <a name="secure-data-in-motion"></a>动态安全数据

如果客户端驱动程序支持加密，托管的实例数据流量会始终加密。 托管的实例和其他 Azure 虚拟机或 Azure 服务之间发送的数据永远不会离开 Azure 的主干。 如果没有托管的实例和本地网络之间的连接，我们建议你使用 Azure ExpressRoute Microsoft 对等互连。 ExpressRoute 可帮助您避免通过公共 internet 移动数据。 对于托管的实例的专用连接，仅专用对等互连可以使用。

## <a name="lock-down-inbound-and-outbound-connectivity"></a>锁定入站和出站连接

下图显示了建议的安全配置：

![阻止入站和出站连接的安全配置](media/sql-database-managed-instance-public-endpoint-securely/managed-instance-vnet.png)

托管的实例都有[专用公共终结点地址](sql-database-managed-instance-find-management-endpoint-ip-address.md)。 在客户端的出站防火墙和网络安全组规则中，设置此公共终结点 IP 地址来限制出站连接。

若要确保托管实例的流量来自受信任的源，我们建议从与已知的 IP 地址的源连接。 使用网络安全组来限制对端口 3342 上的托管的实例公共终结点的访问。

当客户端需要从本地网络启动连接时，请确保起始地址转换为一组已知的 IP 地址。 如果不能这样做，（例如移动员工在典型方案），我们建议你使用[点到站点 VPN 连接和专用终结点](sql-database-managed-instance-configure-p2s.md)。

如果连接从 Azure 启动的我们建议流量来自于已知分配[虚拟 IP 地址](../virtual-network/virtual-networks-reserved-public-ip.md)（例如，虚拟机）。 若要使管理更轻松的虚拟 IP (VIP) 地址，你可能想要使用[公共 IP 地址前缀](../virtual-network/public-ip-address-prefix.md)。

## <a name="next-steps"></a>后续步骤

- 了解如何配置管理实例的公共终结点：[配置公共终结点](sql-database-managed-instance-public-endpoint-configure.md)
