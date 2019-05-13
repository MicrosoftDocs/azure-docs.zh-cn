---
title: 为 Azure Cosmos DB 帐户配置 IP 防火墙
description: 了解如何配置 IP 访问控制策略，以对 Azure Cosmos DB 数据库帐户提供防火墙支持。
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 05/06/2019
ms.author: mjbrown
ms.openlocfilehash: cdf2da745cc418190f6546fffc03e2ac2c330e0e
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65068720"
---
# <a name="configure-ip-firewall-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中配置 IP 防火墙

可以使用 IP 防火墙保护存储在 Azure Cosmos DB 帐户中的数据。 Azure Cosmos DB 支持使用基于 IP 的访问控制来提供入站防火墙支持。 可通过以下方法之一为 Azure Cosmos DB 帐户设置 IP 防火墙：

* 通过 Azure 门户
* 通过使用 Azure 资源管理器模板来以声明方式进行配置
* 通过更新 **ipRangeFilter** 属性并借助 Azure CLI 或 Azure PowerShell 来以编程方式进行配置

## <a id="configure-ip-policy"></a>使用 Azure 门户配置 IP 防火墙

若要在 Azure 门户中设置 IP 访问控制策略，请转到 Azure Cosmos DB 帐户页，然后在导航菜单中选择“防火墙和虚拟网络”。 将“允许从以下位置访问”值更改为“选定的网络”，然后选择“保存”。 

![此屏幕截图显示了如何在 Azure 门户中打开“防火墙”页](./media/how-to-configure-firewall/azure-portal-firewall.png)

启用 IP 访问控制后，可在 Azure 门户中指定 IP 地址、IP 地址范围和开关。 使用开关可以访问其他 Azure 服务和 Azure 门户。 以下部分提供了有关这些开关的详细信息。

> [!NOTE]
> 为 Azure Cosmos DB 帐户启用 IP 访问控制策略后，将拒绝从 IP 地址范围允许列表外部的计算机向 Azure Cosmos DB 帐户发出的所有请求。 此外，还会阻止通过门户浏览 Azure Cosmos DB 资源，以确保访问控制的完整性。

### <a name="allow-requests-from-the-azure-portal"></a>允许来自 Azure 门户的请求

以编程的方式启用 IP 访问控制策略时，需将 Azure 门户的 IP 地址添加到 ipRangeFilter 属性以维持访问。 门户 IP 地址是：

|区域|IP 地址|
|------|----------|
|德国|51.4.229.218|
|中国|139.217.8.252|
|美国政府|52.244.48.71|
|所有其他区域|104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26|

若要允许访问 Azure 门户，可以选择“允许从 Azure 门户访问”选项，如以下屏幕截图所示： 

![此屏幕截图显示了如何启用对 Azure 门户的访问](./media/how-to-configure-firewall/enable-azure-portal.png)

### <a name="allow-requests-from-global-azure-datacenters-or-other-sources-within-azure"></a>允许来自全球的 Azure 数据中心或 Azure 中的其他源的请求

如果通过不提供静态 IP 的服务（例如 Azure 流分析和 Azure Functions）访问 Azure Cosmos DB 帐户，仍可使用 IP 防火墙来限制访问。 若要允许从此类服务访问 Azure Cosmos DB 帐户，请将 IP 地址 0.0.0.0 添加到允许的 IP 地址列表。 0.0.0.0 地址限制从 Azure 数据中心 IP 范围向 Azure Cosmos DB 帐户发出的请求。 此设置不允许任何其他 IP 范围访问 Azure Cosmos DB 帐户。

> [!NOTE]
> 该选项将防火墙配置为允许来自 Azure 的所有请求，包括来自 Azure 中部署的其他客户的订阅的请求。 此选项允许的 IP 地址较为广泛，因为限制了防火墙策略的有效性。 仅当请求并非来自虚拟网络中的静态 IP 或子网时，才使用此选项。 选择此选项将自动允许从 Azure 门户进行访问，因为 Azure 门户在 Azure 中部署。

要想允许访问 Azure 门户，可以选择“接受来自公共 Azure 数据中心内部的连接”选项，如以下屏幕截图所示： 

![此屏幕截图显示了如何在 Azure 门户中打开“防火墙”页](./media/how-to-configure-firewall/enable-azure-services.png)

### <a name="requests-from-your-current-ip"></a>来自当前 IP 的请求

为简化开发，Azure 门户将帮助你识别客户端计算机的 IP 并将其添加到允许列表中。 然后，计算机上运行的应用可以访问你的 Azure Cosmos DB 帐户。 

门户将自动检测客户端 IP 地址。 它可能是计算机的客户端 IP 地址，也可能是网络网关的 IP 地址。 请务必在将工作负荷置于生产环境之前删除此 IP 地址。 

若要将当前 IP 添加到 IP 列表，请选择“添加当前 IP”。 再选择“保存”。

![此屏幕截图显示了如何为当前 IP 配置防火墙设置](./media/how-to-configure-firewall/enable-current-ip.png)

### <a name="requests-from-cloud-services"></a>来自云服务的请求

在 Azure 中，云服务是一种使用 Azure Cosmos DB 托管中间层服务逻辑的常用方法。 若要从云服务启用对 Azure Cosmos DB 帐户的访问，必须通过[配置 IP 访问控制策略](#configure-ip-policy)，将云服务的公共 IP 地址添加到与 Azure Cosmos DB 帐户关联的 IP 地址的允许列表中。 这可确保云服务的所有角色实例都有权访问 Azure Cosmos DB 帐户。 

如以下屏幕截图所示，可以在 Azure 门户中检索云服务的 IP 地址：

![该屏幕截图显示在 Azure 门户中显示的云服务的公共 IP 地址](./media/how-to-configure-firewall/public-ip-addresses.png)

通过添加角色实例横向扩展云服务时，这些新的实例会自动获得 Azure Cosmos DB 帐户的访问权限，因为它们属于同一云服务。

### <a name="requests-from-virtual-machines"></a>来自虚拟机的请求

还可以使用[虚拟机](https://azure.microsoft.com/services/virtual-machines/)或[虚拟机规模集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)通过 Azure Cosmos DB 托管中间层服务。 要配置 Cosmos DB 帐户使其允许从虚拟机访问，必须将虚拟机和/或虚拟机规模集的公共 IP 地址配置为你的 Azure Cosmos DB 帐户允许的一个 IP 地址，方法是[配置 IP 访问控制策略](#configure-ip-policy)。 

如以下屏幕截图所示，可以在 Azure 门户中检索虚拟机的 IP 地址：

![该屏幕截图显示在 Azure 门户中显示的虚拟机的公共 IP 地址](./media/how-to-configure-firewall/public-ip-addresses-dns.png)

将虚拟机实例添加到组时，这些实例会自动获得 Azure Cosmos DB 帐户的访问权限。

### <a name="requests-from-the-internet"></a>来自 Internet 的请求

从 Internet 上的计算机访问 Azure Cosmos DB 帐户时，必须将客户端 IP 地址或计算机的 IP 地址范围添加到帐户 IP 地址的允许列表中。

## <a id="configure-ip-firewall-arm"></a>使用资源管理器模板配置 IP 防火墙

若要配置对 Azure Cosmos DB 帐户的访问控制，请确保资源管理器模板指定 **ipRangeFilter** 属性，其中包含允许的 IP 范围列表。 例如，将以下 JSON 代码添加到模板：

```json
   {
     "apiVersion": "2015-04-08",
     "type": "Microsoft.DocumentDB/databaseAccounts",
     "kind": "GlobalDocumentDB",
     "name": "[parameters('databaseAccountName')]",
     "location": "[resourceGroup().location]",
     "properties": {
       "databaseAccountOfferType": "Standard",
       "name": "[parameters('databaseAccountName')]",
       "ipRangeFilter":"183.240.196.255,104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26"
     }
   }
```

## <a id="configure-ip-firewall-cli"></a>使用 Azure CLI 配置 IP 访问控制策略

以下命令演示如何创建具有 IP 访问控制的 Azure Cosmos DB 帐户： 

```azurecli-interactive

name="<Azure Cosmos DB account name>"
resourceGroupName="<Resource group name>"

az cosmosdb create \
  --name $name \
  --kind GlobalDocumentDB \
  --resource-group $resourceGroupName \
  --max-interval 10 \
  --max-staleness-prefix 200 \
  --ip-range-filter "183.240.196.255,104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26"
```

若要更新现有帐户的防火墙设置，请运行以下命令：

```azurecli-interactive
az cosmosdb update \
      --name $name \
      --resource-group $resourceGroupName \
      --ip-range-filter "183.240.196.255,104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26"
```

## <a id="configure-ip-firewall-ps"></a>使用 PowerShell 配置 IP 访问控制策略

以下脚本演示如何使用 IP 访问控制创建 Azure Cosmos DB 帐户：

```azurepowershell-interactive

$resourceGroupName = "myResourceGroup"
$accountName = "myaccountname"

$locations = @(
    @{ "locationName"="West US"; "failoverPriority"=0 },
    @{ "locationName"="East US"; "failoverPriority"=1 }
)

# Add local machine's IP address to firewall, InterfaceAlias is your Network Adapter's name
$ipRangeFilter = Get-NetIPConfiguration | Where-Object InterfaceAlias -eq "Ethernet 2" | Select-Object IPv4Address

$consistencyPolicy = @{ "defaultConsistencyLevel"="Session" }

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$locations;
    "consistencyPolicy"=$consistencyPolicy;
    "ipRangeFilter"=$ipRangeFilter
}

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

## <a id="troubleshoot-ip-firewall"></a>排查 IP 访问控制策略的问题

可使用以下选项排查 IP 访问控制策略的问题： 

### <a name="azure-portal"></a>Azure 门户 
为 Azure Cosmos DB 帐户启用 IP 访问控制策略后，将阻止从 IP 地址范围的允许列表外部的计算机向帐户发出的所有请求。 若要启用门户数据平面操作，例如浏览容器和查询文档，需要使用门户中的“防火墙”窗格显式允许访问 Azure 门户。

### <a name="sdks"></a>SDK 
使用不在允许列表内的计算机访问 Azure Cosmos DB 资源时，将返回一般的“403 禁止访问”响应，但不提供其他任何详细信息。 验证帐户的允许 IP 列表并确保 Azure Cosmos DB 帐户中应用了正确的策略配置。 

### <a name="source-ips-in-blocked-requests"></a>受阻止请求中的源 IP
对 Azure Cosmos DB 帐户启用诊断日志记录。 这些日志显示每个请求和响应。 会记录带有 403 返回代码的防火墙相关消息。 通过筛选这些消息，可以查看已阻止请求的源 IP。 请参阅 [Azure Cosmos DB 诊断日志记录](logging.md)。

### <a name="requests-from-a-subnet-with-a-service-endpoint-for-azure-cosmos-db-enabled"></a>来自已启用 Azure Cosmos DB 服务终结点的子网的请求
来自虚拟网络中已启用 Azure Cosmos DB 服务终结点的子网的请求向 Azure Cosmos DB 帐户发送虚拟网络和子网标识。 这些请求不包含源的公共 IP，因此 IP 筛选器将拒绝它们。 若要允许从虚拟网络中的特定子网进行访问，请添加[如何为 Azure Cosmos DB 帐户配置基于虚拟网络和子网的访问](how-to-configure-vnet-service-endpoint.md)中所述的访问控制列表。 应用防火墙规则最多可能需要 15 分钟。


## <a name="next-steps"></a>后续步骤

若要为 Azure Cosmos DB 帐户配置虚拟网络服务终结点，请参阅以下文章：

* [适用于 Azure Cosmos DB 帐户的虚拟网络和子网访问控制](vnet-service-endpoint.md)
* [为 Azure Cosmos DB 帐户配置基于虚拟网络和子网的访问](how-to-configure-vnet-service-endpoint.md)

