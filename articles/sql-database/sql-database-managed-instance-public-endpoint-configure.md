---
title: 配置公共终结点-Azure SQL 数据库托管实例 |Microsoft Docs
description: 了解如何配置托管实例的公共终结点
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto, carlrab
manager: craigg
ms.date: 05/07/2019
ms.openlocfilehash: d3e68a5287e59c576f85491e6e5eba33fac080ca
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465195"
---
# <a name="configure-public-endpoint-in-azure-sql-database-managed-instance"></a>在 Azure SQL 数据库托管实例中配置公共终结点

公共终结点[托管的实例](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index)到托管实例从外部数据访问[虚拟网络](../virtual-network/virtual-networks-overview.md)。 你将能够从多租户 Azure 服务，例如 Power BI、 Azure 应用服务或在本地网络访问你的托管的实例。 通过使用托管实例上的公共终结点，不需要使用 VPN，可帮助避免 VPN 吞吐量问题。

在本文中，你将了解如何：

> [!div class="checklist"]
> - 启用你在 Azure 门户中的托管实例的公共终结点
> - 启用使用 PowerShell 将托管实例的公共终结点
> - 配置在托管的实例的网络安全组，以允许流量传输到托管的实例的公共终结点
> - 获取托管的实例的公共终结点连接字符串

## <a name="permissions"></a>权限

由于托管实例中的数据的敏感性，配置，可以启用托管的实例的公共终结点需要两步过程。 此安全措施都遵循职责分离 (SoD):

- 启用的托管实例上的公共终结点需要通过托管的实例管理员。托管的实例管理员可以上找到**概述**页 SQL 托管实例资源。
- 使用网络管理员应采取的网络安全组允许流量有关详细信息，请参阅[网络安全组权限](../virtual-network/manage-network-security-group.md#permissions)。

## <a name="enabling-public-endpoint-for-a-managed-instance-in-the-azure-portal"></a>启用在 Azure 门户中的托管实例的公共终结点

1. 启动 Azure 门户 （） <https://portal.azure.com/.>
1. 与托管实例中，打开资源组，然后选择**SQL 托管实例**你想要在配置公共终结点。
1. 上**安全**设置中，选择**虚拟网络**选项卡。
1. 在虚拟网络配置页中，选择**启用**，然后**保存**图标以更新配置。

![mi-vnet-config.png](media/sql-database-managed-instance-public-endpoint-configure/mi-vnet-config.png)

## <a name="enabling-public-endpoint-for-a-managed-instance-using-powershell"></a>启用使用 PowerShell 的托管实例的公共终结点

### <a name="enable-public-endpoint"></a>启用公共终结点

运行以下 PowerShell 命令。 替换**订阅 id**你的订阅 id。 替换**rg 名称**与资源组的托管的实例和替换**mi 名称**与你的托管实例的名称。

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi = $mi | Set-AzSqlInstance -PublicDataEndpointEnabled $true -force
```

### <a name="disable-public-endpoint"></a>禁用公共终结点

若要禁用使用 PowerShell 的公共终结点，你将执行以下命令 （和也不要忘记关闭入站端口 3342 NSG，如果将其配置）：

```powershell
Set-AzSqlInstance -PublicDataEndpointEnabled $false -force
```

## <a name="allow-public-endpoint-traffic-on-the-network-security-group"></a>允许的网络安全组上的公共终结点流量

1. 如果必须保持打开的托管实例的配置页，请导航到**概述**选项卡。否则，返回到您**SQL 托管实例**资源。 选择**虚拟网络/子网**链接，将转到虚拟网络配置页。

    ![mi-overview.png](media/sql-database-managed-instance-public-endpoint-configure/mi-overview.png)

1. 选择**子网**选项卡上左的配置窗格中的虚拟网络，并记下**安全组**托管实例。

    ![mi-vnet-subnet.png](media/sql-database-managed-instance-public-endpoint-configure/mi-vnet-subnet.png)

1. 返回到你的资源组包含你的托管的实例。 应会看到**网络安全组**上面记下名称。 选择要转到网络安全组配置页的名称。

1. 选择**入站安全规则**选项卡上，并**添加**规则的优先级高于**deny_all_inbound**规则具有以下设置： </br> </br>

    |设置  |建议的值  |描述  |
    |---------|---------|---------|
    |**源**     |任何 IP 地址或服务标记         |<ul><li>对于 Power BI 等 Azure 服务，请选择 Azure 云服务标记</li> <li>对于你的计算机或 Azure VM，请使用 NAT IP 地址</li></ul> |
    |**源端口范围**     |*         |将其保留为 * （任何） 根据源端口通常是动态分配，并因此不可预测 |
    |**目标**     |任意         |保留与任何以允许到托管的实例子网的流量的目标 |
    |**目标端口范围**     |3342         |作用域的目标端口前往 3342，这是托管的实例的公共 TDS 端点 |
    |协议     |TCP         |TDS 的托管实例使用 TCP 协议 |
    |**Action**     |允许         |允许入站的流量通过公共终结点的托管实例 |
    |**Priority**     |1300         |请确保此规则优先级高于**deny_all_inbound**规则 |

    ![mi-nsg-rules.png](media/sql-database-managed-instance-public-endpoint-configure/mi-nsg-rules.png)

    > [!NOTE]
    > 端口 3342 用于连接到托管实例，并在此时不能更改公共终结点。

## <a name="obtaining-the-managed-instance-public-endpoint-connection-string"></a>获取托管的实例的公共终结点连接字符串

1. 导航到已启用的公共终结点的 SQL 托管的实例配置页。 选择**连接字符串**选项卡上的**设置**配置。
1. 请注意，采用格式 < mi_name > 来自公共终结点主机名。**公共**。 < dns_zone >。 database.windows.net 和用于连接的端口是 3342。

    ![mi-public-endpoint-conn-string.png](media/sql-database-managed-instance-public-endpoint-configure/mi-public-endpoint-conn-string.png)

## <a name="next-steps"></a>后续步骤

- 了解如何[使用 Azure SQL 数据库托管实例通过公共终结点安全地](sql-database-managed-instance-public-endpoint-securely.md)。