---
title: 通过 Azure 应用程序网关配置端到端 SSL
description: 本文介绍如何使用 PowerShell 通过 Azure 应用程序网关配置端到端 SSL
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 4/8/2019
ms.author: victorh
ms.openlocfilehash: a4ce1ad347742886e7d89a32bbeb60c2e0281409
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198564"
---
# <a name="configure-end-to-end-ssl-by-using-application-gateway-with-powershell"></a>使用 PowerShell 通过应用程序网关配置端到端 SSL

## <a name="overview"></a>概述

Azure 应用程序网关支持对流量进行端到端加密。 应用程序网关终止应用程序网关上的 SSL 连接。 网关随后将路由规则应用于流量、重新加密数据包，并根据定义的路由规则将数据包转发到适当的后端服务器。 来自 Web 服务器的任何响应都会经历相同的过程返回最终用户。

应用程序网关支持定义自定义 SSL 选项。 除支持定义要使用的密码套件和优先级顺序外，它还支持禁用以下协议版本：TLSv1.0、TLSv1.1 和 TLSv1.2。 若要详细了解可配置的 SSL 选项，请参阅 [SSL 策略概述](application-gateway-SSL-policy-overview.md)。

> [!NOTE]
> SSL 2.0 和 SSL 3.0 默认处于禁用状态且无法启用。 这些版本被视为不安全的版本，不能用于应用程序网关。

![方案图像][scenario]

## <a name="scenario"></a>场景

在此方案中，可学习如何通过 PowerShell 使用端到端 SSL 创建应用程序网关。

此方案将：

* 创建名为 appgw-rg 的资源组。
* 创建名为 appgwvnet，地址空间为 10.0.0.0/16 的虚拟网络。
* 创建名为“appgwsubnet”和“appsubnet”的两个子网。
* 创建支持端到端 SSL 加密且限制 SSL 协议版本和密码套件的小型应用程序网关。

## <a name="before-you-begin"></a>开始之前

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

若要对应用程序网关配置端到端 SSL，需要网关证书和后端服务器证书。 网关证书用于根据 SSL 协议规范派生对称密钥。 然后使用对称密钥加密和解密发送到网关的流量。 网关证书需要采用个人信息交换 (PFX) 格式。 此文件格式适用于导出私钥，后者是应用程序网关对流量进行加解密所必需的。

若要加密端到端 SSL，后端必须已加入应用程序网关的允许列表。 将后端服务器的公用证书上传到应用程序网关。 添加证书后，可确保应用程序网关仅与已知后端实例通信。 从而进一步保护端到端通信。

配置过程在以下部分中介绍。

## <a name="create-the-resource-group"></a>创建资源组

本部分指导创建资源组，其中包含应用程序网关。

1. 登录到 Azure 帐户。

   ```powershell
   Connect-AzAccount
   ```

2. 选择要用于此方案的订阅。

   ```powershell
   Select-Azsubscription -SubscriptionName "<Subscription name>"
   ```

3. 创建资源组。 （若要使用现有资源组，请跳过此步骤。）

   ```powershell
   New-AzResourceGroup -Name appgw-rg -Location "West US"
   ```

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>为应用程序网关创建虚拟网络和子网

以下示例创建一个虚拟网络和两个子网。 一个子网用于托管应用程序网关。 另一个子网用于可托管 Web 应用程序的后端。

1. 分配要用于应用程序网关的子网地址范围。

   ```powershell
   $gwSubnet = New-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24
   ```

   > [!NOTE]
   > 应适当调整为应用程序网关配置的子网的大小。 最多可以为 10 个实例配置应用程序网关。 每个实例从子网获取 1 个 IP 地址。 子网太小可能会对应用程序网关的向外缩放造成负面影响。
   >

2. 分配要用于后端地址池的地址范围。

   ```powershell
   $nicSubnet = New-AzVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24
   ```

3. 创建具有上述步骤中定义的子网的虚拟网络。

   ```powershell
   $vnet = New-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet
   ```

4. 检索要用于后续步骤的虚拟网络资源和子网资源。

   ```powershell
   $vnet = Get-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
   $nicSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet
   ```

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>创建前端配置的公共 IP 地址

创建要用于应用程序网关的公共 IP 资源。 此公共 IP 地址会用于后续步骤之一。

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName appgw-rg -Name 'publicIP01' -Location "West US" -AllocationMethod Dynamic
```

> [!IMPORTANT]
> 应用程序网关不支持使用通过定义的域标签创建的公共 IP 地址。 仅支持具有动态创建的域标签的公共 IP 地址。 如果需要应用程序网关具有友好 DNS 名称，建议使用 CNAME 记录作为别名。

## <a name="create-an-application-gateway-configuration-object"></a>创建应用程序网关配置对象

在创建应用程序网关之前设置所有配置项。 以下步骤将创建应用程序网关资源所需的配置项。

1. 创建应用程序网关 IP 配置。 此设置配置应用程序网关要使用的子网。 当应用程序网关启动时，它从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。 请记住，每个实例需要一个 IP 地址。

   ```powershell
   $gipconfig = New-AzApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet
   ```

2. 创建前端 IP 配置。 此设置将专用或公共 IP 地址映射到应用程序网关的前端。 以下步骤将上述步骤中的公共 IP 地址与前端 IP 配置关联。

   ```powershell
   $fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip
   ```

3. 使用后端 Web 服务器的 IP 地址配置后端 IP 地址池。 这些 IP 地址将接收来自前端 IP 终结点的网络流量。 使用自己的应用程序 IP 地址端点替换样本中的 IP 地址。

   ```powershell
   $pool = New-AzApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3
   ```

   > [!NOTE]
   > 完全限定的域名 (FQDN) 也是可用于替换后端服务器 IP 地址的有效值。 可通过 -BackendFqdns 开关启用它。 

4. 配置公共 IP 终结点的前端 IP 端口。 此端口是最终用户连接到的端口。

   ```powershell
   $fp = New-AzApplicationGatewayFrontendPort -Name 'port01'  -Port 443
   ```

5. 配置应用程序网关的证书。 此证书用于加密和解密应用程序网关上的流量。

   ```powershell
   $passwd = ConvertTo-SecureString  <certificate file password> -AsPlainText -Force 
   $cert = New-AzApplicationGatewaySSLCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password $passwd 
   ```

   > [!NOTE]
   > 此示例配置用于 SSL 连接的证书。 该证书需采用 .pfx 格式，并且密码长度必须为 4 到 12 个字符。

6. 创建应用程序网关的 HTTP 侦听器。 分配要使用的前端 IP 配置、端口和 SSL 证书。

   ```powershell
   $listener = New-AzApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SSLCertificate $cert
   ```

7. 上传要在已启用 SSL 的后端池资源上使用的证书。

   > [!NOTE]
   > 默认探测从后端的 IP 地址上的*默认* SSL 绑定获取公钥，并将其收到的公钥值与用户在此处提供的公钥值进行比较。 
   > 
   > 如果正在后端使用主机头和服务器名称指示 (SNI)，则检索到的公钥可能不是流量预期流向的站点。 如有疑问，请访问后端服务器上的 https://127.0.0.1/，确认用于默认 SSL 绑定的证书。 本部分使用该请求中的公钥。 如果对 HTTPS 绑定使用主机头和 SNI，但未从后端服务器的 https://127.0.0.1/ 手动浏览器请求收到响应和证书，则必须在其上设置默认 SSL 绑定。 如果不这样做，探测会失败，后端不会列入允许名单。

   ```powershell
   $authcert = New-AzApplicationGatewayAuthenticationCertificate -Name 'whitelistcert1' -CertificateFile C:\cert.cer
   ```

   > [!NOTE]
   > 在上一步中提供的证书应该是后端上存在的.pfx 证书的公钥。 以索赔、证据和推理 (CER) 格式导出后端服务器上安装的证书（不是根证书），将其用在此步骤。 此步骤会将后端加入应用程序网关的允许列表。

   如果使用的是应用程序网关 v2 SKU，则创建受信任的根证书而不是身份验证证书。 有关详细信息，请参阅[应用程序网关的端到端 SSL 概述](ssl-overview.md#end-to-end-ssl-with-the-v2-sku)：

   ```powershell
   $trustedRootCert01 = New-AzApplicationGatewayTrustedRootCertificate -Name "test1" -CertificateFile  <path to root cert file>
   ```

8. 配置应用程序网关后端 HTTP 设置。 将上述步骤中上传的证书分配给 HTTP 设置。

   ```powershell
   $poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert
   ```

   对于应用程序网关 v2 SKU，请使用以下命令：

   ```powershell
   $poolSetting01 = New-AzApplicationGatewayBackendHttpSettings -Name “setting01” -Port 443 -Protocol Https -CookieBasedAffinity Disabled -TrustedRootCertificate $trustedRootCert01 -HostName "test1"
   ```

9. 创建配置负载均衡器行为的负载均衡器路由规则。 在此示例中，创建基本轮循机制规则。

   ```powershell
   $rule = New-AzApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
   ```

10. 配置应用程序网关的实例大小。 可用大小为 **Standard\_Small**、**Standard\_Medium** 和 **Standard\_Large**。  对于容量，可用值为 1 到 10。

    ```powershell
    $sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
    ```

    > [!NOTE]
    > 进行测试时，可以选择 1 作为实例计数。 必须知道的是，2 以下的实例计数不受 SLA 支持，因此不建议使用。 小型网关用于开发/测试，不用于生产。

11. 配置要在应用程序网关上使用的 SSL 策略。 应用程序网关支持设置 SSL 协议最低版本的功能。

    以下值是可以定义的协议版本的列表：

    - **TLSV1_0**
    - **TLSV1_1**
    - **TLSV1_2**
    
    以下示例将最低协议版本设置为 TLSv1_2 并仅启用 TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256、TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384 和 TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256。

    ```powershell
    $SSLPolicy = New-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -PolicyType Custom
    ```

## <a name="create-the-application-gateway"></a>创建应用程序网关

使用上述所有步骤创建应用程序网关。 网关创建过程需要花费较长时间。

```powershell
$appgw = New-AzApplicationGateway -Name appgateway -SSLCertificates $cert -ResourceGroupName "appgw-rg" -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SSLPolicy $SSLPolicy -AuthenticationCertificates $authcert -Verbose
```

## <a name="limit-ssl-protocol-versions-on-an-existing-application-gateway"></a>限制现有应用程序网关上的 SSL 协议版本

上述步骤指导创建具有端到端 SSL 并禁用特定 SSL 协议版本的应用程序。 以下示例禁用现有应用程序网关上的特定 SSL 策略。

1. 检索要更新的应用程序网关。

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```

2. 定义 SSL 策略。 如下示例禁用了 TLSv1.0 和 TLSv1.1，仅允许密码套件 TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256、TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384 和 TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256。

   ```powershell
   Set-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -PolicyType Custom -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -ApplicationGateway $gw

   ```

3. 最后，更新网关。 最后一步耗时较长。 完成后，应用程序网关上即已配置端到端 SSL。

   ```powershell
   $gw | Set-AzApplicationGateway
   ```

## <a name="get-an-application-gateway-dns-name"></a>获取应用程序网关 DNS 名称

创建网关后，下一步是配置前端以进行通信。 使用公共 IP 时，应用程序网关需要动态分配的 DNS 名称，这会造成不便。 若要确保最终用户能够访问应用程序网关，可以使用 CNAME 记录指向应用程序网关的公共终结点。 有关详细信息，请参阅[在 Azure 中配置自定义域名](../cloud-services/cloud-services-custom-domain-name-portal.md)。 

若要配置别名，可使用附加到应用程序网关的 PublicIPAddress 元素检索应用程序网关及其关联的 IP/DNS 名称的详细信息。 使用应用程序网关的 DNS 名称来创建 CNAME 记录，使两个 Web 应用程序都指向此 DNS 名称。 不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

```powershell
Get-AzPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>后续步骤

请参阅 [Web 应用程序防火墙概述](application-gateway-webapplicationfirewall-overview.md)，详细了解如何通过应用程序网关的 Web 应用程序防火墙强化 Web 应用程序的安全性。

[scenario]: ./media/application-gateway-end-to-end-SSL-powershell/scenario.png
