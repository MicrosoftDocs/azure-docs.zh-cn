---
title: 在 Azure VPN 网关上配置 BGP：资源管理器和 CLI | Microsoft Docs
description: 本文将介绍如何使用 Azure 资源管理器和 CLI 通过 Azure VPN 网关配置 BGP。
services: vpn-gateway
documentationcenter: na
author: yushwang
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/25/2018
ms.author: yushwang
ms.openlocfilehash: 51402196c8429797b644357822a1e3c08982b384
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209510"
---
# <a name="how-to-configure-bgp-on-an-azure-vpn-gateway-by-using-cli"></a>如何使用 CLI 在 Azure VPN 网关上配置 BGP

本文将帮助用户使用 Azure 资源管理器部署模型和 Azure CLI 在跨界站点到站点 (S2S) VPN 连接和 VNet 到 VNet 连接（即虚拟网络之间的连接）上启用 BGP。

## <a name="about-bgp"></a>关于 BGP

BGP 是通常在 Internet 上使用的，用于在两个或更多网络之间交换路由和可访问性信息的标准路由协议。 BGP 使 VPN 网关和本地 VPN 设备（称为 BGP 对等节点或邻域）能够交换路由。 路由会向这两个网关提供有关前缀可用性和可访问性的信息，以便通过所涉及的网关或路由器。 此外，BGP 还可以通过将 BGP 网关从一个 BGP 对等节点获知的路由传播到所有其他 BGP 对等节点，以实现在多个网络之间传输路由。

有关 BGP 优点的详细信息，以及使用 BGP 的技术要求和注意事项，请参阅 [Azure VPN 网关的 BGP 概述](vpn-gateway-bgp-overview.md)。

本文有助于执行以下任务：

* [为 VPN 网关启用 BGP](#enablebgp)（必需）

  然后可完成以下任一部分或者同时完成两者：

* [使用 BGP 建立跨界连接](#crossprembgp)
* [使用 BGP 建立 VNet 到 VNet 连接](#v2vbgp)

这三个部分中的每一部分都构成用于在网络连接中启用 BGP 的基本构建基块。 如果完成所有这三个部分，可生成如下图所示的拓扑：

![BGP 拓扑](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

可将这些部分组合在一起，生成更复杂的多跃点传输网络来满足需求。

## <a name ="enablebgp"></a>为 VPN 网关启用 BGP

需要先完成此部分，才可执行另外两个配置部分中的任何步骤。 以下配置步骤设置如下图所示的 Azure VPN 网关的 BGP 参数：

![BGP 网关](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>开始之前

安装最新版本的 CLI 命令（2.0 或更高版本）。 有关安装 CLI 命令的信息，请参阅[安装 Azure CLI](/cli/azure/install-azure-cli) 和 [Azure CLI 入门](/cli/azure/get-started-with-azure-cli)。

### <a name="step-1-create-and-configure-testvnet1"></a>步骤 1：创建并配置 TestVNet1

#### <a name="Login"></a>1.连接到订阅

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

#### <a name="2-create-a-resource-group"></a>2.创建资源组

以下示例在“eastus”位置创建名为 TestRG1 的资源组。 如果在想要创建虚拟网络的区域中已经有了一个资源组，则可改用该资源组。

```azurecli
az group create --name TestBGPRG1 --location eastus
```

#### <a name="3-create-testvnet1"></a>3.创建 TestVNet1

以下示例创建一个名为 TestVNet1 的虚拟网络和三个子网：GatewaySubnet、FrontEnd 和 Backend。 替换值时，请务必始终将网关子网特意命名为 GatewaySubnet。 如果命名为其他名称，网关创建会失败。

第一个命令创建前端地址空间和 FrontEnd 子网。 第二个命令为后端子网创建额外的地址空间。 第三个和第四个命令创建 BackEnd 子网和 GatewaySubnet。

```azurecli
az network vnet create -n TestVNet1 -g TestBGPRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24 
 
az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestBGPRG1 
 
az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestBGPRG1 --address-prefix 10.12.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestBGPRG1 --address-prefix 10.12.255.0/27 
```

### <a name="step-2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>步骤 2：使用 BGP 参数为 TestVNet1 创建 VPN 网关

#### <a name="1-create-the-public-ip-address"></a>1.创建公共 IP 地址

请求公共 IP 地址。 将向为虚拟网络创建的 VPN 网关分配公共 IP 地址。

```azurecli
az network public-ip create -n GWPubIP -g TestBGPRG1 --allocation-method Dynamic 
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2.使用 AS 编号创建 VPN 网关

为 TestVNet1 创建虚拟网络网关。 BGP 需要基于路由的 VPN 网关。 此外，还需要额外的参数 `-Asn`，为 TestVNet1 设置自治系统编号 (ASN)。 创建网关可能需要一点时间（45 分钟或更久）才能完成。 

如果使用 `--no-wait` 参数运行该命令，则不会显示任何反馈或输出。 `--no-wait` 参数允许在后台创建网关。 但并不意味着 VPN 网关会立即创建。

```azurecli
az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address GWPubIP -g TestBGPRG1 --vnet TestVNet1 --gateway-type Vpn --sku HighPerformance --vpn-type RouteBased --asn 65010 --no-wait
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3.获取 Azure BGP 对等节点 IP 地址

创建网关后，需要在 Azure VPN 网关上获取 BGP 对等节点 IP 地址。 需要此地址才能将 VPN 网关配置为本地 VPN 设备的 BGP 对等节点。

运行以下命令，并检查输出顶部的 `bgpSettings` 部分：

```azurecli
az network vnet-gateway list -g TestBGPRG1 
 
  
"bgpSettings": { 
      "asn": 65010, 
      "bgpPeeringAddress": "10.12.255.30", 
      "peerWeight": 0 
    }
```

创建网关后，可以使用此网关通过 BGP 建立跨界连接或 VNet 到 VNet 连接。

## <a name ="crossprembgp"></a>使用 BGP 建立跨界连接

要建立跨界连接，需要创建本地网关来表示本地 VPN 设备。 然后将 Azure VPN 网关与本地网关连接在一起。 虽然这些步骤与创建其他连接的步骤类似，但它们包括指定 BGP 配置参数所需的其他属性。

![跨界的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)


### <a name="step-1-create-and-configure-the-local-network-gateway"></a>步骤 1：创建和配置本地网关

此练习将继续生成图中所示的配置。 请务必将值替换为用于配置的值。 使用本地网络网关时，请记住以下事项：

* 本地网关可以与 VPN 网关在相同的位置和资源组中，也可以在不同的位置和资源组中。 此示例演示网关在不同位置的不同资源组中。
* 需要为本地网关声明的最小前缀是 VPN 设备上的 BGP 对等节点 IP 地址中的主机地址。 在这种情况下，它是 / 32 前缀 10.51.255.254/32。
* 提醒一下，在本地网络与 Azure 虚拟网络之间必须使用不同的 BGP ASN。 如果它们是相同的，则需要更改 VNet ASN（如果本地 VPN 设备已使用该 ASN 与其他 BGP 邻居对等）。

请确保已完成此练习的[为 VPN 网关启用 BGP](#enablebgp) 部分，并且仍与订阅 1 连接，然后再继续操作。 请注意，在此示例中会创建新的资源组。 另请注意，本地网关的两个附加参数：`Asn` 和 `BgpPeerAddress`。

```azurecli
az group create -n TestBGPRG5 -l eastus2 
 
az network local-gateway create --gateway-ip-address 23.99.221.164 -n Site5 -g TestBGPRG5 --local-address-prefixes 10.51.255.254/32 --asn 65050 --bgp-peering-address 10.51.255.254
```

### <a name="step-2-connect-the-vnet-gateway-and-local-network-gateway"></a>步骤 2：连接 VNet 网关和本地网关

在此步骤中，创建从 TestVNet1 到 Site5 的连接。 必须指定 `--enable-bgp` 参数，以便为此连接启用 BGP。 

在此示例中，虚拟网关和本地网关位于不同的资源组中。 网关位于不同的资源组中时，必须指定两个网关的整个资源 ID，以便在虚拟网络之间建立连接。

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1.获取 VNet1GW 的资源 ID

使用以下命令的输出，获取 VNet1GW 的资源 ID：

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

在输出中，找到 `"id":` 行。 引号中的值是在下一部分创建连接所必需的。

示例输出：

```
{ 
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65010, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
  }, 
  "enableBgp": true, 
  "etag": "W/\"<your etag number>\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
```

将 `"id":` 后的值复制到文本编辑器（例如记事本），这样就可以在创建连接时轻松粘贴它们。 

```
"id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
```

#### <a name="2-get-the-resource-id-of-site5"></a>2.获取 Site5 的资源 ID

使用以下命令，从输出中获取 Site5 的资源 ID：

```azurecli
az network local-gateway show -n Site5 -g TestBGPRG5
```

#### <a name="3-create-the-testvnet1-to-site5-connection"></a>3.创建 TestVNet1 到 Site5 的连接

在此步骤中，创建从 TestVNet1 到 Site5 的连接。 如前所述，同一 Azure VPN 网关可以同时具有 BGP 连接和非 BGP 连接。 除非在连接属性中启用了 BGP，否则 Azure 不会为此连接启用 BGP，即使已在这两个网关上配置了 BGP 参数。 将该订阅 ID 替换为自己的订阅 ID。

```azurecli
az network vpn-connection create -n VNet1ToSite5 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "abc123" --local-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG5/providers/Microsoft.Network/localNetworkGateways/Site5 --no-wait
```

在此练习中，以下示例列出了要在本地 VPN 设备上的 BGP 配置部分中输入的参数：

```
Site5 ASN            : 65050
Site5 BGP IP         : 10.52.255.254
Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
Azure VNet ASN       : 65010
Azure VNet BGP IP    : 10.12.255.30
Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

几分钟后，应会建立连接。 建立 IPsec 连接后，可开始 BGP 对等会话。

## <a name ="v2vbgp"></a>使用 BGP 建立 VNet 到 VNet 连接

此部分使用 BGP 添加 VNet 到 VNet 连接，如下图所示： 

![VNet 到 VNet 的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

以下说明延续前面各部分中的步骤。 必须完成[为 VPN 网关启用 BGP](#enablebgp) 部分，才能使用 BGP 创建和配置 TestVNet1 和 VPN 网关。

### <a name="step-1-create-testvnet2-and-the-vpn-gateway"></a>步骤 1：创建 TestVNet2 和 VPN 网关

必须确保新虚拟网络的 IP 地址空间 TestVNet2 不与任何 VNet 范围重叠。

在本示例中，虚拟网络属于同一订阅。 可在不同订阅之间设置 VNet 到 VNet 连接。 若要了解详细信息，请参阅[配置 VNet 到 VNet 的连接](vpn-gateway-howto-vnet-vnet-cli.md)。 请确保在创建连接时添加 `-EnableBgp $True`，以启用 BGP。

#### <a name="1-create-a-new-resource-group"></a>1.创建新的资源组

```azurecli
az group create -n TestBGPRG2 -l westus
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2.在新资源组中创建 TestVNet2

第一个命令创建前端地址空间和 FrontEnd 子网。 第二个命令为后端子网创建额外的地址空间。 第三个和第四个命令创建 BackEnd 子网和 GatewaySubnet。

```azurecli
az network vnet create -n TestVNet2 -g TestBGPRG2 --address-prefix 10.21.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.21.0.0/24 
 
az network vnet update -n TestVNet2 --address-prefixes 10.21.0.0/16 10.22.0.0/16 -g TestBGPRG2 
 
az network vnet subnet create --vnet-name TestVNet2 -n BackEnd -g TestBGPRG2 --address-prefix 10.22.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet2 -n GatewaySubnet -g TestBGPRG2 --address-prefix 10.22.255.0/27
```

#### <a name="3-create-the-public-ip-address"></a>3.创建公共 IP 地址

请求公共 IP 地址。 将向为虚拟网络创建的 VPN 网关分配公共 IP 地址。

```azurecli
az network public-ip create -n GWPubIP2 -g TestBGPRG2 --allocation-method Dynamic
```

#### <a name="4-create-the-vpn-gateway-with-the-as-number"></a>4.使用 AS 编号创建 VPN 网关

为 TestVNet2 创建虚拟网络网关。 必须覆盖 Azure VPN 网关上的默认 ASN。 连接的虚拟网络的 ASN 必须不同，才能启用 BGP 和传输路由。
 
```azurecli
az network vnet-gateway create -n VNet2GW -l westus --public-ip-address GWPubIP2 -g TestBGPRG2 --vnet TestVNet2 --gateway-type Vpn --sku Standard --vpn-type RouteBased --asn 65020 --no-wait
```

### <a name="step-2-connect-the-testvnet1-and-testvnet2-gateways"></a>步骤 2：连接 TestVNet1 和 TestVNet2 网关

在此步骤中，创建从 TestVNet1 到 Site5 的连接。 必须指定 `--enable-bgp` 参数，以便为此连接启用 BGP。

在以下示例中，虚拟网关和本地网关位于不同的资源组中。 网关位于不同的资源组中时，必须指定两个网关的整个资源 ID，以便在虚拟网络之间建立连接。 

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1.获取 VNet1GW 的资源 ID 

从以下命令的输出中获取 VNet1GW 的资源 ID：

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

#### <a name="2-get-the-resource-id-of-vnet2gw"></a>2.获取 VNet2GW 的资源 ID

从以下命令的输出中获取 VNet2GW 的资源 ID：

```azurecli
az network vnet-gateway show -n VNet2GW -g TestBGPRG2
```

#### <a name="3-create-the-connections"></a>3.创建连接

创建从 TestVNet1 到 TestVNet2 的连接，以及从 TestVNet2 到 TestVNet1 的连接。 将该订阅 ID 替换为自己的订阅 ID。

```azurecli
az network vpn-connection create -n VNet1ToVNet2 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW
```

```azurecli
az network vpn-connection create -n VNet2ToVNet1 -g TestBGPRG2 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW --enable-bgp -l westus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
```

> [!IMPORTANT]
> 为这两个连接启用 BGP。
> 
> 

完成这些步骤后，可在几分钟内建立连接。 完成 VNet 到 VNet 的连接后，BGP 对等会话即可运行。

## <a name="next-steps"></a>后续步骤

连接完成后，即可将虚拟机添加到虚拟网络。 相关步骤，请参阅[创建虚拟机](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
