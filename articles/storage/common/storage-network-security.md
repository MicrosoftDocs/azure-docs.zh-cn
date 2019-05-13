---
title: 配置 Azure 存储防火墙和虚拟网络 | Microsoft Docs
description: 配置存储帐户的分层网络安全性。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 0fa252fa2cdab7bb3790153f2d118fea13739852
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153516"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>配置 Azure 存储防火墙和虚拟网络

Azure 存储提供分层安全模型。 借助此模型，可保护存储帐户，使其仅可供受支持的一组特定网络访问。 配置网络规则后，仅通过指定网络组请求数据的应用程序才能访问存储帐户。

在网络规则生效后访问存储帐户的应用程序需要在请求中提供适当的授权。 支持通过 Azure Active Directory (Azure AD) 凭据（适用于 Blob 和队列）、有效的帐户访问密钥或 SAS 令牌进行授权。

> [!IMPORTANT]
> 默认情况下，除非请求来自在 Azure 虚拟网络 (VNet) 内运行的服务，否则开启存储帐户的防火墙规则会阻止数据传入请求。 被阻止的请求包括来自其他 Azure 服务、来自 Azure 门户、来自日志记录和指标服务等的请求。
>
> 可通过允许服务实例的子网，授予在 VNet 内运行的 Azure 服务相应的访问权限。 通过下一部分介绍的[例外](#exceptions)机制，启用有限数量的方案。 若要访问 Azure 门户，需要从设置的可信边界（IP 或 VNet）内的计算机进行访问。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="scenarios"></a>方案

将存储帐户配置为默认拒绝来自所有网络的流量（包括 Internet 流量）的访问。 然后授予来自特定 VNet 的流量相应的访问权限。 借助此配置，可为应用程序生成安全网络边界。 还可向公共 Internet IP 地址范围授予访问权限，支持来自特定 Internet 或本地客户端的连接。

对于面向 Azure 存储的所有网络协议（包括 REST 和 SMB），将强制实施网络规则。 若要使用 Azure 门户、存储资源管理器和 AZCopy 等工具访问数据，需要提供显式网络规则。

可将网络规则应用于现有存储帐户，也可在创建新存储帐户时应用网络规则。

一旦应用网络规则，就会对所有请求强制实施这些规则。 用于向特定 IP 地址授予访问权限的 SAS 令牌可限制令牌持有者的访问权限，但不会越过已配置的网络规则授予新的访问权限。

虚拟机磁盘流量（包括装载和卸载操作以及磁盘 IO）不受网络规则影响。 对页 blob 的 REST 访问受网络规则保护。

经典存储帐户不支持防火墙和虚拟网络。

可通过创建例外，使用应用了网络规则的存储帐户中的非托管磁盘来备份和还原 VM。 此过程在本文的[例外](#exceptions)部分中记录。 防火墙例外不适用于托管磁盘，因为它们已由 Azure 托管。

## <a name="change-the-default-network-access-rule"></a>更改默认网络访问规则

默认情况下，存储帐户接受来自任何网络上客户端的连接。 若要限制为仅允许选定网络访问，必须先更改默认操作。

> [!WARNING]
> 更改网络规则可能会使应用程序无法正常连接到 Azure 存储。 除非还应用了**授予**访问权限的特定网络规则，否则将默认网络规则设置为“拒绝”会阻止对数据的所有访问。 在将默认规则更改为拒绝访问之前，务必先使用网络规则对所有许可网络授予访问权限。

### <a name="managing-default-network-access-rules"></a>管理默认网络访问规则

可以通过 Azure 门户、PowerShell 或 CLIv2 管理存储帐户的默认网络访问规则。

#### <a name="azure-portal"></a>Azure 门户

1. 转至要保护的存储帐户。

1. 单击名为“防火墙和虚拟网络”的设置菜单。

1. 若要默认拒绝访问，请选择允许从“所选网络”进行访问。 若要允许来自所有网络的流量，请选择允许从“所有网络”进行访问。

1. 单击“保存”应用所做的更改。

#### <a name="powershell"></a>PowerShell

1. 安装 [Azure PowerShell](/powershell/azure/install-Az-ps) 并 [登录](/powershell/azure/authenticate-azureps)。

1. 显示存储帐户默认规则的状态。

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").DefaultAction
    ```

1. 将默认规则设置为默认拒绝网络访问。

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Deny
    ```

1. 将默认规则设置为默认允许网络访问。

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Allow
    ```

#### <a name="cliv2"></a>CLIv2

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)。

1. 显示存储帐户默认规则的状态。

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.defaultAction
    ```

1. 将默认规则设置为默认拒绝网络访问。

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Deny
    ```

1. 将默认规则设置为默认允许网络访问。

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Allow
    ```

## <a name="grant-access-from-a-virtual-network"></a>允许从虚拟网络进行访问

可将存储帐户配置为仅允许从特定 VNet 进行访问。

在 VNet 内为 Azure 存储启用[服务终结点](/azure/virtual-network/virtual-network-service-endpoints-overview)。 此终结点为流量提供到 Azure 存储服务的最优路径。 虚拟网络和子网的标识也随每个请求进行传输。 管理员随后可以配置存储帐户的网络规则，允许从 VNet 中的特定子网接收请求。 通过这些网络规则获得访问权限的客户端必须继续满足存储帐户的授权要求，才能访问数据。

每个存储帐户最多支持 100 条虚拟网络规则，这些规则可与 [IP 网络规则](#grant-access-from-an-internet-ip-range)组合使用。

### <a name="available-virtual-network-regions"></a>可用的虚拟网络区域

服务终结点一般在位于同一 Azure 区域的虚拟网络和服务实例之间运行。 将服务终结点与 Azure 存储配合使用时，此范围扩大到包含[配对区域](/azure/best-practices-availability-paired-regions)。 服务终结点可以在区域性故障转移期间提供连续性，并允许访问读取访问权限异地冗余存储 (RA-GRS) 实例。 允许从虚拟网络访问存储帐户的网络规则同样允许访问所有 RA-GRS 实例。

在计划区域性服务中断期间的灾难恢复时，应该在配对区域中提前创建 VNet。 为 Azure 存储启用服务终结点，并提供允许从这些备用虚拟网络进行访问的网络规则。 然后将这些规则应用于异地冗余存储帐户。

> [!NOTE]
> 服务终结点不适用于位于虚拟网络所在区域和指定区域对之外的流量。 可以将允许从虚拟网络访问存储帐户的网络规则仅应用于存储帐户主区域或指定配对区域中的存储帐户。

### <a name="required-permissions"></a>所需权限

若要向存储帐户应用虚拟网络规则，用户必须对要添加的子网拥有适当的权限。 所需的权限为*向子网加入服务*权限，该权限包含在*存储帐户参与者*内置角色中。 该权限还可以添加到自定义角色定义中。

存储帐户和获得访问权限的虚拟网络可以位于不同的订阅中，但这些订阅必须属于同一个 Azure AD 租户。

### <a name="managing-virtual-network-rules"></a>管理虚拟网络规则

可以通过 Azure 门户、PowerShell 或 CLIv2 管理存储帐户的虚拟网络规则。

#### <a name="azure-portal"></a>Azure 门户

1. 转至要保护的存储帐户。

1. 单击名为“防火墙和虚拟网络”的设置菜单。

1. 检查并确保已选择允许从“所选网络”进行访问。

1. 若要使用新的网络规则向虚拟网络授予访问权限，请在“虚拟网络”下，单击“添加现有虚拟网络”，选择“虚拟网络”和“子网”选项，然后单击“添加”。 若要创建新的虚拟网络并授予其访问权限，请单击“添加新的虚拟网络”。 提供创建新的虚拟网络所需的信息，然后单击“创建”。

    > [!NOTE]
    > 如果之前没有为所选的虚拟网络和子网配置 Azure 存储的服务终结点，则可在执行此操作时进行配置。

1. 若要删除虚拟网络或子网规则，请单击“...”打开虚拟网络或子网的上下文菜单，然后单击“删除”。

1. 单击“保存”应用所做的更改。

#### <a name="powershell"></a>PowerShell

1. 安装 [Azure PowerShell](/powershell/azure/install-Az-ps) 并 [登录](/powershell/azure/authenticate-azureps)。

1. 列出虚拟网络规则。

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").VirtualNetworkRules
    ```

1. 在现有虚拟网络和子网上启用 Azure 存储的服务终结点。

    ```powershell
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.0.0.0/24" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    ```

1. 为虚拟网络和子网添加网络规则。

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

1. 为虚拟网络和子网删除网络规则。

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

#### <a name="cliv2"></a>CLIv2

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)。

1. 列出虚拟网络规则。

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query virtualNetworkRules
    ```

1. 在现有虚拟网络和子网上启用 Azure 存储的服务终结点。

    ```azurecli
    az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.Storage"
    ```

1. 为虚拟网络和子网添加网络规则。

    ```azurecli
    $subnetid=(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

1. 为虚拟网络和子网删除网络规则。

    ```azurecli
    $subnetid=(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

## <a name="grant-access-from-an-internet-ip-range"></a>允许从 Internet IP 范围进行访问

可以将存储帐户配置为允许从特定的公共 Internet IP 地址范围进行访问。 此配置向基于 Internet 的特定服务和本地网络授予访问权限，并阻止一般 Internet 流量。

使用 [CIDR 表示法](https://tools.ietf.org/html/rfc4632)以 *16.17.18.0/24* 的形式，或使用单独的 IP 地址（如 *16.17.18.19*）提供允许的 Internet 地址范围。

   > [!NOTE]
   > 不支持使用“/31”或“/32”前缀大小的小型地址范围。 这些范围应使用单独的 IP 地址规则配置。

IP 网络规则仅适用于**公共 Internet** IP 地址。 IP 规则不允许使用为专用网络保留的 IP 地址范围（如 [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3) 中所定义）。 专用网络包括以 _10.*_、_172.16.*_ - _172.31.*_ 和 _192.168.*_ 开头的地址。

   > [!NOTE]
   > IP 网络规则对源自与存储帐户相同的 Azure 区域的请求不起作用。 请使用[虚拟网络规则](#grant-access-from-a-virtual-network)来允许相同区域的请求。

目前仅支持 IPV4 地址。

每个存储帐户最多支持 100 条 IP 网络规则，这些规则可与 [虚拟网络规则](#grant-access-from-a-virtual-network)组合使用。

### <a name="configuring-access-from-on-premises-networks"></a>配置从本地网络的访问

若要使用 IP 网络规则授予本地网络访问存储帐户的权限，则必须标识网络所用的面向 Internet 的 IP 地址。 若要获得帮助，请联系网络管理员。

如果是在本地使用 [ExpressRoute](/azure/expressroute/expressroute-introduction)，则在进行公共对等互连或 Microsoft 对等互连时，需标识所用的 NAT IP 地址。 进行公共对等互连时，每条 ExpressRoute 线路默认情况下会使用两个 NAT IP 地址。当流量进入 Microsoft Azure 网络主干时，会向 Azure 服务流量应用这些地址。 进行 Microsoft 对等互连时，所用 NAT IP 地址由客户或服务提供商提供。 若要允许访问服务资源，必须在资源 IP 防火墙设置中允许这些公共 IP 地址。 若要查找公共对等互连 ExpressRoute 线路 IP 地址，请通过 Azure 门户[开具 ExpressRoute 支持票证](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。 详细了解[适用于 ExpressRoute 公共对等互连和 Microsoft 对等互连的 NAT](/azure/expressroute/expressroute-nat#nat-requirements-for-azure-public-peering)。

### <a name="managing-ip-network-rules"></a>管理 IP 网络规则

可以通过 Azure 门户、PowerShell 或 CLIv2 管理存储帐户的 IP 网络规则。

#### <a name="azure-portal"></a>Azure 门户

1. 转至要保护的存储帐户。

1. 单击名为“防火墙和虚拟网络”的设置菜单。

1. 检查并确保已选择允许从“所选网络”进行访问。

1. 若要向 Internet IP 范围授予访问权限，请在“防火墙” > “地址范围”下输入 IP 地址或地址范围（采用 CIDR 格式）。

1. 若要删除某个 IP 网络规则，请单击该地址范围旁边的垃圾桶图标。

1. 单击“保存”应用所做的更改。

#### <a name="powershell"></a>PowerShell

1. 安装 [Azure PowerShell](/powershell/azure/install-Az-ps) 并 [登录](/powershell/azure/authenticate-azureps)。

1. 列出 IP 网络规则。

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").IPRules
    ```

1. 为单个 IP 地址添加网络规则。

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

1. 为 IP 地址范围添加网络规则。

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

1. 为单个 IP 地址删除网络规则。

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

1. 为 IP 地址范围删除网络规则。

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

#### <a name="cliv2"></a>CLIv2

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)。

1. 列出 IP 网络规则。

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query ipRules
    ```

1. 为单个 IP 地址添加网络规则。

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

1. 为 IP 地址范围添加网络规则。

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

1. 为单个 IP 地址删除网络规则。

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

1. 为 IP 地址范围删除网络规则。

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

## <a name="exceptions"></a>例外

大多数情况下，网络规则可以实现安全的网络配置。 但是，在某些情况下，必须允许例外才能启用完整功能。 可以为存储帐户针对受信任的 Microsoft 服务和存储分析数据访问配置例外。

### <a name="trusted-microsoft-services"></a>受信任的 Microsoft 服务

某些与存储帐户交互的 Microsoft 服务在网络上运行，但这些网络无法通过网络规则获得访问权限。

若要帮助此类服务按预期方式工作，请允许受信任的 Microsoft 服务集绕过网络规则。 这些服务随后会使用强身份验证访问存储帐户。

如果启用“允许受信任的 Microsoft 服务...”例外，以下服务（在订阅中注册后）有权访问存储帐户：

|服务|资源提供程序名称|目的|
|:------|:---------------------|:------|
|Azure 备份|Microsoft.Backup|在 IAAS 虚拟机中运行非托管磁盘的备份和还原。 （不是托管磁盘的必需操作）。 [了解详细信息](/azure/backup/backup-introduction-to-azure-backup)。|
|Azure Data Box|Microsoft.DataBox|启用导入到 Azure 中使用 Data Box 的数据。 [了解详细信息](/azure/databox/data-box-overview)。|
|Azure 开发测试实验室|Microsoft.DevTestLab|自定义映像创建和项目安装。 [了解详细信息](/azure/devtest-lab/devtest-lab-overview)。|
|Azure 事件网格|Microsoft.EventGrid|启用 Blob 存储事件发布并允许事件网格发布到存储队列。 了解有关 [blob 存储事件](/azure/event-grid/event-sources)和[发布到队列](/azure/event-grid/event-handlers)的信息。|
|Azure 事件中心|Microsoft.EventHub|使用事件中心捕获功能存档数据。 [了解详细信息](/azure/event-hubs/event-hubs-capture-overview)。|
|Azure HDInsight|Microsoft.HDInsight|预配新的 HDInsight 群集的默认文件系统的初始内容。 [了解详细信息](https://azure.microsoft.com/blog/enhance-hdinsight-security-with-service-endpoints/)。|
|Azure Monitor|Microsoft.Insights|允许将监视数据写入受保护存储帐户[了解详细信息](/azure/monitoring-and-diagnostics/monitoring-roles-permissions-security)。|
|Azure 网络|Microsoft.Networking|存储和分析网络流量日志。 [了解详细信息](/azure/network-watcher/network-watcher-packet-capture-overview)。|
|Azure Site Recovery|Microsoft.SiteRecovery |通过启用 Azure IaaS 虚拟机的复制来配置灾难恢复。 如果使用启用了防火墙的缓存存储帐户、源存储帐户或目标存储帐户，则这是必需的。  [了解详细信息](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-enable-replication)。|
|Azure SQL 数据仓库|Microsoft.Sql|允许导入和导出方案中使用 PolyBase 的特定 SQL 数据库实例。 [了解详细信息](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview)。|

### <a name="storage-analytics-data-access"></a>存储分析数据访问

在某些情况下，需要从网络边界外访问读取诊断日志和指标。 可以授予网络规则例外来允许对存储帐户日志文件和/或指标表进行读取访问。 [详细了解如何使用存储分析。](/azure/storage/storage-analytics)

### <a name="managing-exceptions"></a>管理例外

可以通过 Azure 门户、PowerShell 或 Azure CLI v2 管理网络规则例外。

#### <a name="azure-portal"></a>Azure 门户

1. 转至要保护的存储帐户。

1. 单击名为“防火墙和虚拟网络”的设置菜单。

1. 检查并确保已选择允许从“所选网络”进行访问。

1. 在“例外”下，选择要允许的例外。

1. 单击“保存”应用所做的更改。

#### <a name="powershell"></a>PowerShell

1. 安装 [Azure PowerShell](/powershell/azure/install-Az-ps) 并 [登录](/powershell/azure/authenticate-azureps)。

1. 显示存储帐户的网络规则例外。

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount").Bypass
    ```

1. 配置存储帐户的网络规则例外。

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass AzureServices,Metrics,Logging
    ```

1. 删除存储帐户的网络规则例外。

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass None
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则，删除例外操作不会有任何效果。

#### <a name="cliv2"></a>CLIv2

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)。

1. 显示存储帐户的网络规则例外。

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.bypass
    ```

1. 配置存储帐户的网络规则例外。

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass Logging Metrics AzureServices
    ```

1. 删除存储帐户的网络规则例外。

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass None
    ```

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则，删除例外操作不会有任何效果。

## <a name="next-steps"></a>后续步骤

在[服务终结点](/azure/virtual-network/virtual-network-service-endpoints-overview)中了解有关 Azure 网络服务终结点的详细信息。

在 [Azure 存储安全指南](storage-security-guide.md)中深入了解 Azure 存储安全。
