---
title: 教程：使用 Azure 门户部署和配置 Azure 防火墙
description: 本教程介绍如何使用 Azure 门户部署和配置 Azure 防火墙。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 4/9/2019
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 9d7b9673101ed3b6ff85a9981ba061bc870762b1
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65405688"
---
# <a name="tutorial-deploy-and-configure-azure-firewall-using-the-azure-portal"></a>教程：使用 Azure 门户部署和配置 Azure 防火墙

控制出站网络访问是整个网络安全计划的重要组成部分。 例如，你可能想要限制对网站的访问， 或者限制可以访问的出站 IP 地址和端口。

可以控制 Azure 子网的出站网络访问的一种方法是使用 Azure 防火墙。 使用 Azure 防火墙，可以配置：

* 应用程序规则，用于定义可从子网访问的完全限定域名 (FQDN)。
* 网络规则，用于定义源地址、协议、目标端口和目标地址。

将网络流量路由到用作子网默认网关的防火墙时，网络流量受到配置的防火墙规则的控制。

在本教程中，你将创建一个包含三个子网的单个简化 VNet，以便于部署。 对于生产部署，我们建议使用[中心辐射模型](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)，其中，防火墙在其自身的 VNet 中。 工作负荷服务器在包含一个或多个子网的同一区域中的对等 VNet 内。

* **AzureFirewallSubnet** - 防火墙在此子网中。
* **Workload-SN** - 工作负荷服务器在此子网中。 此子网的网络流量通过防火墙。
* **Jump-SN** -“跳转”服务器在此子网中。 可以使用远程桌面连接到跳转服务器中的公共 IP 地址。 然后，可在跳转服务器中（使用另一个远程桌面）连接到工作负荷服务器。

![教程网络基础结构](media/tutorial-firewall-rules-portal/Tutorial_network.png)

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 设置测试网络环境
> * 部署防火墙
> * 创建默认路由
> * 配置一个应用程序规则以允许访问 www.google.com
> * 配置网络规则，以允许访问外部 DNS 服务器
> * 测试防火墙

如果需要，可以使用 [Azure PowerShell](deploy-ps.md) 完成本教程中的步骤。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="set-up-the-network"></a>设置网络

首先，创建一个资源组用于包含部署防火墙所需的资源。 然后创建 VNet、子网和测试服务器。

### <a name="create-a-resource-group"></a>创建资源组

资源组包含本教程所需的所有资源。

1. 在 [https://portal.azure.com](https://portal.azure.com) 中登录 Azure 门户。
2. 在 Azure 门户主页上，选择“资源组” > “添加”。
3. 对于“资源组名称”，请键入 **Test-FW-RG**。
4. 对于“订阅”，请选择自己的订阅。
5. 对于“资源组位置”，请选择一个位置。 创建的所有后续资源必须位于同一位置。
6. 选择“创建”。

### <a name="create-a-vnet"></a>创建 VNet

此 VNet 将包含三个子网。

1. 在 Azure 门户主页上，选择“创建资源”。
2. 在“网络”下，选择“虚拟网络”。
4. 对于“名称”，请键入 **Test-FW-VN**。
5. 对于“地址空间”，请键入 **10.0.0.0/16**。
6. 对于“订阅”，请选择自己的订阅。
7. 对于“资源组”，请选择“Test-FW-RG”。
8. 对于“位置”，请选择前面使用的同一位置。
9. 在“子网”下，为“名称”键入 **AzureFirewallSubnet**。 防火墙将位于此子网中，子网名称**必须**是 AzureFirewallSubnet。
10. 对于“地址范围”，请键入 **10.0.1.0/24**。
11. 接受其他默认设置，然后选择“创建”。

> [!NOTE]
> AzureFirewallSubnet 子网的最小大小为 /26。

### <a name="create-additional-subnets"></a>创建更多的子网

接下来，为跳转服务器和工作负荷服务器创建子网。

1. 在 Azure 门户主页上，选择“资源组” > “Test-FW-RG”。
2. 选择“Test-FW-VN”虚拟网络。
3. 选择“子网” > “+子网”。
4. 对于“名称”，请键入 **Workload-SN**。
5. 对于“地址范围”，请键入 **10.0.2.0/24**。
6. 选择“确定”。

创建名为 **Jump-SN**、地址范围为 **10.0.3.0/24** 的另一个子网。

### <a name="create-virtual-machines"></a>创建虚拟机

现在，创建跳转虚拟机和工作负荷虚拟机，并将其放入相应的子网。

1. 在 Azure 门户中，选择“创建资源”。
2. 选择“计算”，然后在“特色”列表中选择“Windows Server 2016 Datacenter”。
3. 输入虚拟机的以下值：

   |设置  |值  |
   |---------|---------|
   |资源组     |**Test-FW-RG**|
   |虚拟机名称     |**Srv-Jump**|
   |区域     |与前面相同|
   |管理员用户名     |**azureuser**|
   |密码     |**Azure123456!** -|

4. 在“入站端口规则”下，对于 **“公共入站端口”**，请选择“允许所选端口”。
5. 对于“选择入站端口”，请选择“RDP (3389)”。

6. 接受其他默认值，然后选择“下一步:**磁盘”**。
7. 接受磁盘默认值，然后选择“下一步:网络”。
8. 请确保选择 **Test-FW-VN** 作为虚拟网络，子网是 **Jump-SN**。
9. 对于“公共 IP”，请接受默认的新公共 IP 地址名称 (Srv-Jump-ip)。
11. 接受其他默认值，然后选择“下一步:**管理”**。
12. 选择“关闭”以禁用启动诊断。 接受其他默认值，然后选择“查看 + 创建”。
13. 检查摘要页上的设置，然后选择“创建”。

使用下表中的信息配置名为 **Srv-Work** 的另一个虚拟机。 剩余的配置与 Srv-Work 虚拟机相同。

|设置  |值  |
|---------|---------|
|子网|**Workload-SN**|
|公共 IP|无|
|公共入站端口|无|

## <a name="deploy-the-firewall"></a>部署防火墙

将防火墙部署到 VNet。

1. 在门户主页上，选择“创建资源”。
2. 在搜索框中键入“防火墙”，然后按 **Enter**。
3. 选择“防火墙”，然后选择“创建”。
4. 在“创建防火墙”页上，使用下表配置防火墙：

   |设置  |值  |
   |---------|---------|
   |订阅     |\<用户的订阅\>|
   |资源组     |**Test-FW-RG** |
   |名称     |**Test-FW01**|
   |位置     |选择前面使用的同一位置|
   |选择虚拟网络     |**使用现有项**：**Test-FW-VN**|
   |公共 IP 地址     |**新建**。 公共 IP 地址必须为“标准 SKU”类型。|

5. 选择“查看 + 创建”。
6. 查看摘要，然后选择“创建”以创建防火墙。

   需要花费几分钟时间来完成部署。
7. 部署完成后，转到“Test-FW-RG”资源组，然后选择“Test-FW01”防火墙。
8. 记下专用 IP 地址。 稍后在创建默认路由时需要用到此地址。

## <a name="create-a-default-route"></a>创建默认路由

对于“Workload-SN”子网，请配置要通过防火墙的出站默认路由。

1. 在 Azure 门户主页上，选择“所有服务”。
2. 在“网络”下，选择“路由表”。
3. 选择 **添加** 。
4. 对于“名称”，请键入 **Firewall-route**。
5. 对于“订阅”，请选择自己的订阅。
6. 对于“资源组”，请选择“Test-FW-RG”。
7. 对于“位置”，请选择前面使用的同一位置。
8. 选择“创建”。
9. 依次选择“刷新”、“Firewall-route”路由表。
10. 依次选择“子网”、“关联”。
11. 选择“虚拟网络” > “Test-FW-VN”。
12. 对于“子网”，请选择“Workload-SN”。 请确保仅为此路由选择 Workload-SN 子网，否则防火墙将无法正常工作。

13. 选择“确定”。
14. 依次选择“路由”、“添加”。
15. 对于“路由名称”，请键入 **fw-dg**。
16. 对于“地址前缀”，请键入 **0.0.0.0/0**。
17. 对于“下一跃点类型”，请选择“虚拟设备”。

    Azure 防火墙实际上是一个托管服务，但虚拟设备可在此场合下正常工作。
18. 对于“下一跃点地址”，请键入前面记下的防火墙专用 IP 地址。
19. 选择“确定”。

## <a name="configure-an-application-rule"></a>配置应用程序规则

这是允许出站访问 www.google.com 的应用程序规则。

1. 打开“Test-FW-RG”，然后选择“Test-FW01”防火墙。
2. 在“Test-FW01”页上的“设置”下，选择“规则”。
3. 选择“应用程序规则集合”选项卡。
4. 选择“添加应用程序规则集合”。
5. 对于“名称”，请键入 **App-Coll01**。
6. 对于“优先级”，请键入 **200**。
7. 对于“操作”，请选择“允许”。
8. 在“规则”>“目标 FQDN”下，键入 **Allow-Google** 作为**名称**。
9. 对于“源地址”，请键入 **10.0.2.0/24**。
10. 对于“协议:端口”，请键入 **http, https**。
11. 对于“目标 FQDN”，请键入 **www.google.com**
12. 选择 **添加** 。

Azure 防火墙包含默认情况下允许的基础结构 FQDN 的内置规则集合。 这些 FQDN 特定于平台，不能用于其他目的。 有关详细信息，请参阅[基础结构 FQDN](infrastructure-fqdns.md)。

## <a name="configure-a-network-rule"></a>配置网络规则

这是允许在端口 53 (DNS) 上对两个 IP 地址进行出站访问的网络规则。

1. 选择“网络规则集合”选项卡。
2. 选择“添加网络规则集合”。
3. 对于“名称”，请键入 **Net-Coll01**。
4. 对于“优先级”，请键入 **200**。
5. 对于“操作”，请选择“允许”。

6. 在“规则”下，为“名称”键入 **Allow-DNS**。
7. 对于“协议”，请选择“UDP”。
8. 对于“源地址”，请键入 **10.0.2.0/24**。
9. 对于“目标地址”，请键入 **209.244.0.3,209.244.0.4**
10. 对于“目标端口”，请键入 **53**。
11. 选择 **添加** 。

### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>更改 **Srv-Work** 网络接口的主要和辅助 DNS 地址

为了在本教程中进行测试，请配置服务器的主要和辅助 DNS 地址。 这并不是一项常规的 Azure 防火墙要求。

1. 在 Azure 门户中，打开“Test-FW-RG”资源组。
2. 选择 **Srv-Work** 虚拟机的网络接口。
3. 在“设置”下，选择“DNS 服务器”。
4. 在“DNS 服务器”下，选择“自定义”。
5. 在“添加 DNS 服务器”文本框中键入 **209.244.0.3**，在下一个文本框中键入 **209.244.0.4**。
6. 选择“保存”。
7. 重启 **Srv-Work** 虚拟机。

## <a name="test-the-firewall"></a>测试防火墙

现在测试防火墙，以确认它是否按预期方式工作。

1. 在 Azure 门户中，查看 **Srv-Work** 虚拟机的网络设置并记下专用 IP 地址。
2. 将远程桌面连接到 **Srv-Jump** 虚拟机，然后登录。 在该虚拟机中，与 **Srv-Work** 专用 IP 地址建立远程桌面连接。

3. 打开 Internet Explorer 并浏览到 https://www.google.com。
4. 出现 Internet Explorer 安全警报时，请选择“确定” > “关闭”。

   应会看到 Google 主页。

5. 浏览到 https://www.microsoft.com。

   防火墙应会阻止你访问。

现已验证防火墙规则可正常工作：

* 可以浏览到一个允许的 FQDN，但不能浏览到其他任何 FQDN。
* 可以使用配置的外部 DNS 服务器解析 DNS 名称。

## <a name="clean-up-resources"></a>清理资源

可以将防火墙资源保留到下一教程使用。不再需要时，请删除 **Test-FW-RG** 资源组，以删除与防火墙相关的所有资源。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [教程：监视 Azure 防火墙日志](./tutorial-diagnostics.md)
