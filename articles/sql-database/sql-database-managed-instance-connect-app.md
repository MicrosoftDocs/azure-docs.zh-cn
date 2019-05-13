---
title: Azure SQL 数据库托管实例 - 连接应用程序 | Microsoft Docs
description: 本文介绍如何将应用程序连接到 Azure SQL 数据库托管实例。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
manager: craigg
ms.date: 11/09/2018
ms.openlocfilehash: 52a9cfa52cd63715addadcbfb367510ded56fd76
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142725"
---
# <a name="connect-your-application-to-azure-sql-database-managed-instance"></a>将应用程序连接到 Azure SQL 数据库托管实例

在确定如何以及在何处托管应用程序时，目前有多个选项可供选择。

可以选择使用 Azure 应用服务或 Azure 的某些虚拟网络 (VNet) 集成选项（例如 Azure 应用服务环境、虚拟机、虚拟机规模集）将应用程序托管在云中。 还可以采用混合云方法，将应用程序保留在本地。

不管选择哪个选项，都可将应用程序连接到托管实例。  

![高可用性](./media/sql-database-managed-instance/application-deployment-topologies.png)

## <a name="connect-an-application-inside-the-same-vnet"></a>在同一 VNet 中连接应用程序

这是最简单的方案。 即使位于不同子网，VNet 中的虚拟机也能直接相互连接。 这意味着，只需相应地设置连接字符串，就能连接 Azure 应用程序环境或虚拟机中的应用程序。  

## <a name="connect-an-application-inside-a-different-vnet"></a>在不同的 VNet 中连接应用程序

此方案略微复杂，因为托管实例在其自身的 VNet 中具有专用 IP 地址。 若要建立连接，应用程序需要访问部署了托管实例的 VNet。 因此，首先需要在应用程序与托管实例 VNet 之间建立连接。 要正常实施此方案，VNet 不一定要位于同一个订阅中。

可通过两个选项连接 VNet：

- [Azure 虚拟网络对等互连](../virtual-network/virtual-network-peering-overview.md)
- VNet 到 VNet VPN 网关（[Azure 门户](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)、[PowerShell](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)、[Azure CLI](../vpn-gateway/vpn-gateway-howto-vnet-vnet-cli.md)）

对等互连是首选的选项，因为对等互连使用 Microsoft 主干网络，因此，从连接角度看，对等互连 VNet 中的虚拟机与同一 VNet 中虚拟机之间的延迟没有明显差别。 VNet 对等互连限于相同区域内的网络。  

> [!IMPORTANT]
> 由于[全球虚拟网络对等互连的限制](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints)，用于托管实例的 VNet 对等互连方案限于相同区域内的网络。

## <a name="connect-an-on-premises-application"></a>连接本地应用程序

只能通过专用 IP 地址访问托管实例。 若要从本地访问它，需在应用程序与托管实例 VNet 之间建立站点到站点连接。

可以使用两个选项从本地连接到 Azure VNet：

- 站点到站点 VPN 连接（[Azure 门户](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)、[PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)、[Azure CLI](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)）
- [ExpressRoute](../expressroute/expressroute-introduction.md) 连接  

如果已成功建立本地到 Azure 的连接，但无法与托管实例建立连接，请检查防火墙中是否在 SQL 端口 1433 上打开了出站连接，并且打开了 11000-12000 端口范围，以便重定向。

## <a name="connect-an-application-on-the-developers-box"></a>在开发人员工具箱中连接应用程序

只能通过专用 IP 地址访问托管实例，因此，若要从开发人员工具箱访问托管实例，首先需要在开发人员工具箱与托管实例 VNet 之间建立连接。 为此，请使用本机 Azure 证书身份验证配置与 VNet 的点到站点连接。 有关详细信息，请参阅[配置点到站点连接，以便从本地计算机连接到 Azure SQL 数据库托管实例](sql-database-managed-instance-configure-p2s.md)。

## <a name="connect-from-on-premises-with-vnet-peering"></a>通过 VNet 对等互连从本地进行连接

通过客户实现的另一方案是将 VPN 网关安装在单独的虚拟网络中，从承载托管实例的虚拟网络进行订阅。 然后，将两个虚拟网络对等互连。 下面的示例性体系结构图介绍了实现方法。

![VNet 对等互连](./media/sql-database-managed-instance-connect-app/vnet-peering.png)

设置基本的体系结构以后，需修改某些设置，使 VPN 网关能够看到承载托管实例的虚拟网络中的 IP 地址。 为此，请在“对等互连设置”下进行下述很具体的更改。

1. 在承载 VPN 网关的 VNet 中，转到“对等互连”，然后转到进行托管实例对等互连的 VNet 连接，再单击“允许网关传输”。
2. 在承载托管实例的 VNet 中，转到“对等互连”，然后转到进行 VPN 网关对等互连的 VNet 连接，再单击“使用远程网关”。

## <a name="connect-an-azure-app-service-hosted-application"></a>连接 Azure 应用服务托管应用程序

只能通过专用 IP 地址访问托管实例，因此，若要从 Azure 应用服务访问托管实例，首先需要在应用程序与托管实例 VNet 之间建立连接。 请参阅[将应用与 Azure 虚拟网络集成](../app-service/web-sites-integrate-with-vnet.md)。  

有关故障排除信息，请参阅[排查 VNet 和应用程序问题](../app-service/web-sites-integrate-with-vnet.md#troubleshooting)。 如果无法建立连接，请尝试[同步网络配置](sql-database-managed-instance-sync-network-configuration.md)。

将 Azure 应用服务连接到托管实例的一种特殊情况是将 Azure 应用服务集成到已与托管实例 VNet 建立对等互连的网络。 对于这种情况，需要设置以下配置：

- 托管实例 VNet 不得使用网关  
- 托管实例 VNet 中必须设置“使用远程网关”选项
- 对等互连的 VNet 中必须设置“允许网关传输”选项

下图演示了此方案：

![集成的应用对等互连](./media/sql-database-managed-instance/integrated-app-peering.png)

>[!NOTE]
>VNet 集成功能不会将应用与包含 ExpressRoute 网关的 VNet 集成。 即使以共存模式配置 ExpressRoute 网关，VNet 集成也不会实现。 如果需要通过 ExpressRoute 连接访问资源，则可以使用 VNet 中运行的应用服务环境。

## <a name="troubleshooting-connectivity-issues"></a>排查连接问题

若要排查连接问题，请查看以下内容：

- 如果无法从同一 VNet 的不同子网中的 Azure 虚拟机连接到托管实例，请检查是否在 VM 子网上设置了可能会阻止访问的网络安全组。另请注意，需在 SQL 端口 1433 上以及 11000-12000 范围的端口上打开出站连接，因为在 Azure 边界内通过重定向进行连接时，这些都是必需的。
- 对于与 VNet 关联的路由表，请确保将“BGP 传播”设置为“启用”。
- 如果使用 P2S VPN，请在 Azure 门户中检查配置，确定能否看到“入口/出口”编号。 如果编号不为零，则表示 Azure 在本地进行流量的出入路由。

   ![入口/出口编号](./media/sql-database-managed-instance-connect-app/ingress-egress-numbers.png)

- 查看客户端计算机（运行 VPN 客户端的计算机）是否针对你需要访问的所有 VNet 设置了路由条目。 路由存储在 `%AppData%\ Roaming\Microsoft\Network\Connections\Cm\<GUID>\routes.txt` 中。

   ![route.txt](./media/sql-database-managed-instance-connect-app/route-txt.png)

   如此图所示，每个涉及的 VNet 有两个条目，此外还有第三个条目，对应于在门户中配置的 VPN 终结点。

   检查路由的另一种方式是执行以下命令。 输出显示到各种子网的路由：

   ```cmd
   C:\ >route print -4
   ===========================================================================
   Interface List
   14...54 ee 75 67 6b 39 ......Intel(R) Ethernet Connection (3) I218-LM
   57...........................rndatavnet
   18...94 65 9c 7d e5 ce ......Intel(R) Dual Band Wireless-AC 7265
   1...........................Software Loopback Interface 1
   Adapter===========================================================================

   IPv4 Route Table
   ===========================================================================
   Active Routes:
   Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.83.72.1     10.83.74.112     35
         10.0.0.0    255.255.255.0         On-link       172.26.34.2     43
         10.4.0.0    255.255.255.0         On-link       172.26.34.2     43
   ===========================================================================
   Persistent Routes:
   None
   ```

- 如果使用 VNet 对等互连，请确保遵循相关说明来设置[允许网关传输并使用远程网关](#connect-from-on-premises-with-vnet-peering)。

## <a name="required-versions-of-drivers-and-tools"></a>所需的驱动程序和工具版本

如果要连接到托管实例，建议使用以下最低版本的工具和驱动程序：

| 驱动程序/工具 | Version |
| --- | --- |
|.NET framework | 4.6.1（或 .NET Core） |
|ODBC 驱动程序| v17 |
|PHP 驱动程序| 5.2.0 |
|JDBC 驱动程序| 6.4.0 |
|Node.js 驱动程序| 2.1.1 |
|OLEDB 驱动程序| 18.0.2.0 |
|SSMS| 18.0 或[更高版本](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) |
|[SMO](https://docs.microsoft.com/sql/relational-databases/server-management-objects-smo/sql-server-management-objects-smo-programming-guide) | [150](https://www.nuget.org/packages/Microsoft.SqlServer.SqlManagementObjects)或更高版本 |

## <a name="next-steps"></a>后续步骤

- 有关托管实例的信息，请参阅[什么是托管实例](sql-database-managed-instance.md)。
- 有关演示如何新建托管实例的教程，请参阅[创建托管实例](sql-database-managed-instance-get-started.md)。
