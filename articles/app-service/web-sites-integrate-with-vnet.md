---
title: 将应用与 Azure 虚拟网络集成 - Azure 应用服务
description: 演示如何将 Azure 应用服务中的应用连接到新的或现有的 Azure 虚拟网络
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
ms.assetid: 90bc6ec6-133d-4d87-a867-fcf77da75f5a
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/12/2018
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: b97fe5d638b5967f533e489a67d4235bbc5a3db1
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198582"
---
# <a name="integrate-your-app-with-an-azure-virtual-network"></a>将应用与 Azure 虚拟网络进行集成
本文档介绍 Azure 应用服务虚拟网络集成功能，并说明如何在 [Azure 应用服务](https://go.microsoft.com/fwlink/?LinkId=529714) 中使用应用对其进行设置。 使用 [Azure 虚拟网络][VNETOverview] (VNet) 可将多个 Azure 资源置于无法通过 Internet 路由的网络中。 然后可以使用 VPN 技术将这些网络连接到本地网络。 

Azure 应用服务有两种形式。 

1. 支持除独立定价计划以外的全部定价计划的多租户系统
2. 部署到 VNet 中的应用服务环境 (ASE)。 

本文档介绍 VNet 集成功能，该功能用于多租户应用服务。  如果应用在[应用服务环境][ASEintro]中，则该应用已处于 VNet 中且不需要使用 VNet 集成功能来获取同一 VNet 中的资源。

VNet 集成功能允许 Web 应用访问虚拟网络中的资源，但不允许通过虚拟网络对 Web 应用进行专用访问。 专用站点访问指的是仅可从专用网络（例如 Azure 虚拟网络内）对应用进行访问。 专用站点访问仅适用于部署了内部负载均衡器 (ILB) 的 ASE。 有关使用 ILB ASE 的详细信息，请先参阅此文章：[创建和使用 ILB ASE][ILBASE]。 

VNet 集成通常用于实现应用对 VNet 中的数据库和运行 Web 服务的访问。 使用 VNet 集成时，不需要公开 VM 中应用程序的公共终结点，可以改用无法通过 Internet 路由的专用地址。 

VNet 集成功能：

* 需要“标准”、“高级”或“高级 V2”定价计划 
* 适用于经典 VNet 或资源管理器 VNet 
* 支持 TCP 和 UDP
* 适用于 Web 应用、移动应用、API 应用和 Function 应用
* 允许应用一次只连接到 1 个 VNet
* 允许在应用服务计划中一次最多集成 5 个 VNet 
* 允许在应用服务计划中由多个应用使用同一个 VNet
* 需要使用点到站点 VPN 配置的虚拟网络网关
* 由于网关上的 SLA，可实现 99.9% 的 SLA

VNet 集成不支持某些功能，其中包括：

* 装载驱动器
* AD 集成 
* NetBios
* 专用站点访问
* 访问 ExpressRoute 上的资源 
* 访问服务终结点上的资源 

### <a name="getting-started"></a>入门
将 Web 应用连接到虚拟网络之前，需要牢记以下几点：

* 在将目标虚拟网络连接到应用之前，必须借助基于路由的网关启用点到站点 VPN。 
* VNet 所在的订阅必须与应用服务计划 (ASP) 所在的订阅相同。
* 与 VNet 集成的应用使用为该 VNet 指定的 DNS。

## <a name="enabling-vnet-integration"></a>启用 VNet 集成

VNet 集成功能目前提供处于预览阶段的新版本。 该版本不依赖于点到站点 VPN，且同样支持对 ExpressRoute 和服务终结点上的资源进行访问。 若要详细了解这个新的预览版的功能，请转到本文档的末尾处。 

### <a name="set-up-a-gateway-in-your-vnet"></a>在 VNet 中设置网关 ###

如果已使用点到站点地址配置网关，则可以跳至“配置 VNet 与应用的集成”这一步。  
若要创建网关，请执行以下操作：

1. 在 VNet 中[创建网关子网][creategatewaysubnet]。  

1. [创建 VPN 网关][creategateway]。 选择基于路由的 VPN 类型。

1. [设置点到站点地址][setp2saddresses]。 如果网关不在基本 SKU 中，则必须在点到站点配置中禁用 IKEV2 并选择 SSTP。 地址空间必须位于下列其中一个地址块中：

* 10.0.0.0/8 - 这指的是从 10.0.0.0 到 10.255.255.255 的 IP 地址范围
* 172.16.0.0/12 - 这指的是从 172.16.0.0 到 172.31.255.255 的 IP 地址范围 
* 192.168.0.0/16 - 这指的是从 192.168.0.0 到 192.168.255.255 的 IP 地址范围

如果只是创建的网关应用 VNet 集成服务中使用，则不需要的证书上载。 创建网关可能需要 30 分钟。 若要将应用与 VNet 集成，必须先预配网关。 

### <a name="configure-vnet-integration-with-your-app"></a>使用应用配置 VNet 集成 ###

要在应用上启用 VNet 集成，请执行以下步骤： 

1. 在 Azure 门户中转到该应用并打开应用设置，然后选择“网络”>“VNet 集成”。 在配置 VNet 集成前，需将 ASP 缩放至标准或更佳的 SKU。 
 ![VNet 集成 UI][1]

1. 选择“添加 VNet”。 
 ![添加 VNet 集成][2]

1. 选择 VNet。 
  ![选择 VNet][8]
  
完成最后一步后，应用将会重启。  

## <a name="how-the-system-works"></a>系统的工作方式
VNet 集成功能基于点到站点 VPN 技术。 Azure 应用服务中的应用托管在多租户系统中，它可以阻止直接在 VNet 中预配应用的操作。 点到站点技术将网络访问限制于托管应用的虚拟机。 应用受到限制，只能通过混合连接或 VNet 集成向外发送流量至 Internet。 

![VNet 集成的工作原理][3]

## <a name="managing-the-vnet-integrations"></a>管理 VNet 集成
连接到 VNet 以及断开其连接的功能在应用级别执行。 可能影响多个应用的 VNet 集成的操作在应用服务计划级别执行。 可以通过应用 UID 获取 VNet 的详细信息。 同样的信息也会显示在 ASP 级别。 

 ![VNet 详细信息][4]

在“网络功能状态”页中，可查看应用是否已连接到 VNet。 如果 VNet 网关因某种原因而关闭，状态则会显示为未连接。 

现在应用级别的 VNet 集成 UI 中获得的信息与你通过 ASP 获得的详细信息是相同的。 下面是信息中的各项内容：

* VNet 名称 - 链接至虚拟网络 UI
* 位置 - 反映 VNet 的位置。 与其他位置的 VNet 集成可能会导致应用出现延迟。 
* 证书状态 - 反映证书在应用服务计划和 VNet 之间的同步状态。
* 网关状态 - 如果网关因某种原因而关闭，则应用无法访问 VNet 中的资源。 
* VNet 地址空间 - 显示 VNet 的 IP 地址空间。 
* 点到站点地址空间 - 显示 VNet 的点到站点 IP 地址空间。 对 VNet 进行调用时，应用的发件人地址将为这些地址之一。 
* 站点到站点地址空间 - 可以使用站点到站点 VPN 将 VNet 连接到本地资源或其他 VNet。 使用该 VPN 连接定义的 IP 范围如下所示。
* DNS 服务器 - 显示配置了 VNet 的 DNS 服务器。
* 路由到 VNet 的 IP 地址 - 显示路由的地址块，这些地址块用于驱动流量进入 VNet 

在 VNet 集成的应用视图中，能够执行的唯一操作是断开应用与当前所连接到的 VNet 的连接。 若要断开应用与 VNet 的连接，请选择“断开连接”。 在从 VNet 断开连接时，应用将会重启。 断开连接操作不会更改 VNet。 VNet 及其包括网关在内的配置都保持不变。 如果随后想要删除 VNet，则需先删除其中的资源（包括网关）。 

若要访问 ASP VNet 集成 UI，请打开 ASP UI 并选择“网络”。  在 VNet 集成下，选择“单击此处可配置”以打开网络功能状态 UI。

![ASP VNet 集成信息][5]

ASP VNet 集成 UI 会显示 ASP 中的应用使用的所有 VNet。 应用服务计划中任意数量的应用最多可能连接 5 个 VNet。 每个应用只能配置一个集成。 要查看每个 VNet 的详细信息，请单击感兴趣的 VNet。 此处有两种操作可以执行。

* **同步网络**。 同步网络操作确保证书与网络信息是同步的。如果添加或更改 VNet 的 DNS，则需执行“同步网络”操作。 此操作会重启所有使用此 VNet 的应用。
* **添加路由** 添加路由会驱动出站流量进入 VNet。

**路由** 在 VNet 中定义的路由，用于将流量从应用导入 VNet。 如果需要将其他出站流量发送到 VNet 中，则可以在此处添加地址块。   

**证书** 启用 VNet 集成后，必须进行证书交换以确保连接的安全性。 除了证书，还有 DNS 配置、路由以及其他类似的用于描述网络的内容。
如果更改了证书或网络信息，则需单击“同步网络”。 单击“同步网络”会导致应用与 VNet 之间的连接出现短暂的中断。 虽然应用不会重启，但失去连接会导致站点功能失常。 

## <a name="accessing-on-premises-resources"></a>访问本地资源
应用可以通过与具备站点到站点连接的 VNet 集成来访问本地资源。 若要访问本地资源，需要使用点到站点地址块更新本地 VPN 网关路由。 先设置站点到站点 VPN，接着应通过用于配置该 VPN 的脚本来正确地设置路由。 如果在创建站点到站点地址后才添加点到站点 VPN，则需手动更新路由。 具体操作信息取决于每个网关，在此不作说明。 此外，不能使用站点到站点 VPN 连接配置 BGP。

> [!NOTE]
> VNet 集成功能不会将应用与包含 ExpressRoute 网关的 VNet 集成。 即使以[共存模式][VPNERCoex]配置 ExpressRoute 网关，VNet 集成也不会生效。 若要通过 ExpressRoute 连接访问资源，可以使用 VNet 中运行的[应用服务环境][ASE]。 
> 
> 

## <a name="peering"></a>对等互连
可以使用 VNet 集成访问与所连接的 VNet 对等互连的 VNet 中的资源。 若要配置对等互连以使用应用，请执行以下操作：

1. 在应用所连接的 VNet 上添加对等互连连接。 在添加对等互连连接时，启用“允许虚拟网络访问”并单击“允许转发流量”和“允许网关传输”。
1. 在与所连接的 VNet 对等互连的 VNet 上添加对等互连连接。 在目标 VNet 上添加对等互连连接时，启用“允许虚拟网络访问”并单击“允许转发流量”和“允许远程网关”。
1. 转到门户中的“应用服务计划”>“网络”>“VNet 集成 UI”。  选择应用连接的 VNet。 在路由部分下，添加与应用所连接的 VNet 对等互连的 VNet 的地址范围。  


## <a name="pricing-details"></a>定价详细信息
使用 VNet 集成功能涉及到三种相关费用：

* ASP 定价层要求
* 数据传输费用
* VPN 网关费用。

应用需要属于“标准”、“高级”或“高级 V2”应用服务计划。 可在此处了解这些费用的更多详细信息：[应用服务定价][ASPricing]。 

数据出口方面也存在费用，即使 VNet 在同一数据中心也是如此。 [数据传输定价详细信息][DataPricing]中对这些费用进行了说明。 

点到站点 VPN 所需的 VNet 网关会产生费用。 [VPN 网关定价][VNETPricing]页上介绍了详细信息。

## <a name="troubleshooting"></a>故障排除
虽然此功能容易设置，但这并不意味着你就不会遇到问题。 如果在访问所需终结点时遇到问题，可以使用某些实用程序来测试从应用控制台发出的连接。 可以使用两种控制台。 一种是 Kudu 控制台，另一种是 Azure 门户中的控制台。 若要访问应用中的 Kudu 控制台，请转到“工具”->“Kudu”。 这相当于访问 [sitename].scm.azurewebsites.net。 打开后，转到“调试控制台”选项卡。若要访问 Azure 门户托管的控制台，请在应用中转到“工具”->“控制台”。 

#### <a name="tools"></a>工具
由于存在安全约束，**ping**、**nslookup** 和 **tracert** 工具无法通过控制台来使用。 为了填补这方面的空白，我们添加了两种单独的工具。 为了测试 DNS 功能，我们添加了名为 nameresolver.exe 的工具。 语法为：

    nameresolver.exe hostname [optional: DNS Server]

可以使用 **nameresolver** 来检查应用所依赖的主机名。 可以通过这种方式来测试 DNS 是否配置错误，或者测试你是否无权访问 DNS 服务器。

下一工具适用于测试与主机的 TCP 连接情况，以及端口组合情况。 该工具名为 **tcpping**，语法为：

    tcpping.exe hostname [optional: port]

**tcpping** 实用程序会告知是否可访问特定主机和端口。 仅满足以下条件才会显示成功：存在侦听主机和端口组合的应用程序，且可从应用对指定主机和端口进行网络访问。

#### <a name="debugging-access-to-vnet-hosted-resources"></a>针对 VNet 托管的资源进行访问权限调试
许多因素会阻止应用访问特定的主机和端口。 大多数情况下为以下三种情况：

* **存在防火墙。** 如果存在防火墙，则会发生 TCP 超时。 本例中的 TCP 超时为 21 秒。 使用 **tcpping** 工具测试连接性。 除了防火墙外，还有多种原因可能导致 TCP 超时。 
* **DNS 不可访问。** DNS 超时时间为每个 DNS 服务器 3 秒。 如果具有 2 个 DNS 服务器，则超时为 6 秒。 使用 nameresolver 查看 DNS 是否正常工作。 请记住，不能使用 nslookup，因其没有使用为 VNet 配置的 DNS。
* **点到站点地址范围无效。** 点到站点 IP 范围点需要处于 RFC 1918 专用 IP 范围 (10.0.0.0-10.255.255.255 / 172.16.0.0-172.31.255.255 / 192.168.0.0-192.168.255.255) 内。 如果范围的 IP 超出上述范围，则操作无法正常运行。 

如果这些情况不能解决问题，请首先查看以下简单因素： 

* 网关在门户中是否显示为已启动？
* 证书是否显示为处于同步状态？
* 是否有人更改了网络配置却没有在受影响的 ASP 中执行“同步网络”操作？ 

如果网关处于关闭状态，则将其重新启动。 如果证书不同步，则请转到 VNet 集成的 ASP 视图，并单击“同步网络”。 如果怀疑 VNet 配置存在与 ASP 不同步的更改，请单击“同步网络”。  “同步网络”操作将重启 ASP 中任何与该 VNet 集成的应用。 

如果这些都正常，则需进行更深入的故障诊断：

* 是否存在其他应用在使用 VNet 集成访问同一 VNet 中的资源？ 
* 是否能够转到应用控制台并使用 tcpping 来访问 VNet 中的任何其他资源？ 

如果上面这两个问题中有一个的回答为“是”，则说明 VNet 集成功能正常，问题出在其他地方。 这种情况下，解决问题会更加困难，因为并没有简单的方法可以查看你为何无法访问主机:端口。 部分原因包括：

* 在主机上开启了防火墙，导致无法从点到站点 IP 范围访问应用程序端口。 跨子网通常需要公共访问权限。
* 目标主机已关闭
* 应用程序已关闭
* IP 或主机名错误
* 应用程序所侦听的端口不同于所期望的端口。 可以使用终结点主机上的“netstat -aon”匹配进程 ID 和侦听端口。 
* 网络安全组的配置方式导致无法从点到站点 IP 范围访问应用程序主机和端口

请记住，由于不知道应用会使用点到站点 IP 范围中的哪个 IP，因此需允许整个范围中的 IP 进行访问。 

其他调试步骤包括：

* 连接到 VNet 中的某个 VM，尝试在该处访问资源主机:端口。 若要针对 TCP 访问权限进行测试，请使用 PowerShell 命令 test-netconnection。 语法为：

      test-netconnection hostname [optional: -Port]

* 在某个 VM 中启动应用程序，测试能否在应用的控制台中访问该主机和端口

#### <a name="on-premises-resources"></a>本地资源 ####

如果应用无法访问本地资源，则请检查是否能够通过 VNet 访问该资源。 请使用 test-netconnection PowerShell 命令来针对 TCP 访问权限进行测试。 如果 VM 无法访问本地资源，请确保站点到站点 VPN 连接正常。 如果正常，则请检查前述项目以及本地网关配置和状态。 

如果 VNet 托管的 VM 能够访问本地系统但应用无法访问，则可能是由于以下某个原因：

* 在本地网关中未使用点到站点 IP 范围对路由进行配置
* 网络安全组阻止点到站点 IP 范围中的 IP 进行访问
* 本地防火墙阻止来自点到站点 IP 范围的流量


## <a name="powershell-automation"></a>PowerShell 自动化

可以使用 PowerShell 将应用服务与 Azure 虚拟网络集成。 对于准备好运行的脚本，请参阅[将 Azure 应用服务中的应用连接到 Azure 虚拟网络](https://gallery.technet.microsoft.com/scriptcenter/Connect-an-app-in-Azure-ab7527e3)。

## <a name="hybrid-connections-and-app-service-environments"></a>混合连接和应用服务环境
可以通过 3 大功能来访问 VNet 托管的资源。 它们是：

* VNet 集成
* 混合连接
* 应用服务环境

混合连接要求在网络中安装名为混合连接管理器 (HCM) 的中继代理。 HCM 需要能够连接到 Azure 以及应用程序。 混合连接不需要远程网络的入站 Internet 可访问终结点，但却是 VPN 连接所需的。 HCM 只能在 Windows 上运行，最多可以运行 5 个实例以确保为用户提供高可用性。 不过，混合连接只支持 TCP，而且每个 HC 终结点必须与特定的主机:端口组合匹配。 

应用服务环境功能允许在 VNet 中运行 Azure 应用服务的单个租户实例。 如果应用在应用服务环境中，则不需要任何额外步骤即可访问 VNet 中的资源。 应用服务环境与您的应用程序在功能更强大的辅助角色上运行，并可以扩展最多 100 个 ASP 实例。 应用服务环境适用于所有网络功能，包括 ExpressRoute 和服务终结点。  

尽管使用起来存在功能重叠的情况，但这些功能却无法彼此替代。 需要知道使用哪种功能取决于具体需要。 例如：

* 如果你是一名开发人员，且希望在 Azure 中运行一个站点并让其访问你办公桌下的工作站上的数据库，则可使用混合连接，那样最轻松。 
* 如果是一家大型组织，需要在公有云中存储大量 Web 属性并在自己的网络中对其进行管理，则最好是使用应用服务环境。 
* 如果有多个应用需访问 VNet 中的资源，则可采用 VNet 集成。 

如果 VNet 已连接到本地网络，则使用 VNet 集成或应用服务环境可以轻松地使用本地资源。 如果 VNet 尚未连接到本地网络，则与安装 HCM 相比，通过 VNet 设置站点到站点 VPN 的开销会多得多。 

除了功能差异，还有定价差异。 应用服务环境功能是我们推出的高级服务，不仅提供最多的网络配置可能性，还提供其他重要功能。 VNet 集成适用于标准或高级 ASP，尤其适合在 VNet 中通过多租户应用服务安全地使用资源。 混合连接目前依托于 BizTalk 帐户，该帐户的定价级别最低为免费，价格随所需使用量的增加逐渐增加。 但是，在需要使用多个网络时，混合连接是再适合不过的了，可以通过混合连接访问远不止 100 个的独立网络中的资源。 

## <a name="new-vnet-integration"></a>新建 VNet 集成 ##

有一种新版的 VNet 集成功能，该功能不依赖于点到站点 VPN 技术。 与预先存在的功能不同，该项新的预览功能会与 ExpressRoute 和服务终结点配合。 

只能通过更新的 Azure 应用服务缩放单元使用此新功能。 如果可以扩展至高级 V2，则属于较新的应用服务缩放单元。 门户中的 VNet 集成 UI 会告知应用能否使用新的 VNet 集成功能。 

此新版本为预览版，具有以下特征。

* 不需网关即可使用新的 VNet 集成功能
* 可以通过 ExpressRoute 连接来访问资源，除了与连接 ExpressRoute 的 VNet 集成以外，不需要任何其他配置。
* 应用和 VNet 必须位于同一区域中
* 新功能要求在资源管理器 VNet 中有一个未使用的子网。
* 应用必须属于能纵向扩展至高级 V2 的 Azure 应用服务部署。
* 应用服务计划必须是“标准”、“高级”或“高级 V2”计划
* 处于预览状态的新功能不支持生产工作负荷
* 新的 VNet 集成功能不适用于应用服务环境中的应用。
* 不能删除带有集成应用的 VNet。  
* 新的 VNet 集成功能尚不支持使用路由表和全球对等互联。  
* 为每个应用服务计划实例使用一个地址。 由于子网大小在分配后不能更改，因此在使用子网时，其大小应该超出你的最大规模大小。 建议的大小是包含 32 个地址的 /27，因为它所容纳的应用服务计划可以扩大到 20 个实例。
* 可以通过新的 VNet 集成功能来使用服务终结点所保护的资源。 若要实现这一点，请在用于 VNet 集成的子网上启用服务终结点。

若要使用此新功能，请执行以下操作：

1. 转至门户中的“网络 UI”。 如果应用可以使用新功能，将看到使用新的预览功能的功能。  

   ![选择新的预览版 VNet 集成][6]

1. 选择“添加 VNet (预览)”。  

1. 选择希望与之集成的资源管理器 VNet，然后新建一个子网或从预先存在的空子网中选择一个。 集成只需不到一分钟即可完成。 在集成期间，应用会重启。  集成完成后，将能查看所集成的子网的详细信息，同时顶部会出现显示该功能处于预览状态的横幅。

   ![选择 VNet 和子网][7]

若要让应用能使用为 VNet 配置的 DNS 服务器，请为应用创建应用程序设置，其中名称为 WEBSITE_DNS_SERVER，值为该服务器的 IP 地址。  如果有辅助 DNS 服务器，请创建另一个应用程序设置，其中名称为 WEBSITE_DNS_ALT_SERVER，值为该服务器的 IP 地址。 

若要从 VNet 断开应用，请选择“断开连接”。 该操作将重启 Web 应用。 

通过新的 VNet 集成功能可以使用服务终结点。  若要将服务终结点和应用配合使用，请使用新的 VNet 集成连接至所选 VNet，然后在用于集成的子网上配置服务终结点。 

<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-app.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-addvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-howitworks.png
[4]: ./media/web-sites-integrate-with-vnet/vnetint-details.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-aspdetails.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-newvnet.png
[7]: ./media/web-sites-integrate-with-vnet/vnetint-newvnetdetails.png
[8]: ./media/web-sites-integrate-with-vnet/vnetint-selectvnet.png

<!--Links-->
[VNETOverview]: https://azure.microsoft.com/documentation/articles/virtual-networks-overview/ 
[AzurePortal]: https://portal.azure.com/
[ASPricing]: https://azure.microsoft.com/pricing/details/app-service/
[VNETPricing]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[DataPricing]: https://azure.microsoft.com/pricing/details/data-transfers/
[V2VNETP2S]: https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/
[ASEintro]: environment/intro.md
[ILBASE]: environment/create-ilb-ase.md
[V2VNETPortal]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md
[VPNERCoex]: ../expressroute/expressroute-howto-coexist-resource-manager.md
[ASE]: environment/intro.md
[creategatewaysubnet]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#gatewaysubnet
[creategateway]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#creategw
[setp2saddresses]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#addresspool
