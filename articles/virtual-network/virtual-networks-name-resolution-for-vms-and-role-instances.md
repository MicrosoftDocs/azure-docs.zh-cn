---
title: Azure 虚拟网络中资源的名称解析
titlesuffix: Azure Virtual Network
description: Azure IaaS 名称解析方案、混合解决方案、不同的云服务之间、Active Directory 和使用自己的 DNS 服务器。
services: virtual-network
documentationcenter: na
author: rohinkoul
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/25/2019
ms.author: rohink
ms.openlocfilehash: e0f3de95cfd4a18294e5e8e2adcf3b52a7487dbb
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411358"
---
# <a name="name-resolution-for-resources-in-azure-virtual-networks"></a>Azure 虚拟网络中资源的名称解析

可能需要允许虚拟机 (VM) 以及部署在虚拟网络中的其他资源相互通信，具体取决于如何使用 Azure 托管 IaaS、PaaS 和混合解决方案。 尽管可以通过使用 IP 地址实现通信，但使用容易记住且不会更改的名称要简单得多。 

当部署在虚拟网络中的资源需要将域名解析到内部 IP 地址时，它们可以使用两种方法之一：

* [Azure 提供的名称解析](#azure-provided-name-resolution)
* [使用自己的 DNS 服务器的名称解析](#name-resolution-that-uses-your-own-dns-server)（可能会将查询转发到 Azure 提供的 DNS 服务器）

使用的名称解析类型取决于资源需要以怎样的方式进行相互通信。 下表说明了各种方案及相应的名称解析解决方案：

> [!NOTE]
> 根据具体的场景，可能需要使用目前以公共预览版提供的 Azure DNS 专用区域功能。 有关详细信息，请参阅[在专用域中使用 Azure DNS](../dns/private-dns-overview.md)。
>

| **方案** | **解决方案** | **后缀** |
| --- | --- | --- |
| 位于相同虚拟网络的 VM 或位于相同云服务的 Azure 云服务角色实例之间的名称解析。 | [Azure DNS 专用区域](../dns/private-dns-overview.md)或 [Azure 提供的名称解析](#azure-provided-name-resolution) |主机名或 FQDN |
| 位于不同虚拟网络的 VM 或位于不同云服务的角色实例之间的名称解析。 |[Azure DNS 专用区域](../dns/private-dns-overview.md)或客户托管的 DNS 服务器，该服务器在虚拟网络之间转发查询，并由 Azure 进行解析（DNS 代理）。 请参阅[使用自己的 DNS 服务器进行名称解析](#name-resolution-that-uses-your-own-dns-server)。 |仅 FQDN |
| 通过 Azure 应用服务（Web 应用、函数或自动程序）实现的名称解析：对同一虚拟网络中的角色实例或 VM 使用虚拟网络集成。 |客户托管的 DNS 服务器，该服务器在虚拟网络之间转发查询，并由 Azure 进行解析（DNS 代理）。 请参阅[使用自己的 DNS 服务器进行名称解析](#name-resolution-that-uses-your-own-dns-server)。 |仅 FQDN |
| 从应用服务 Web 应用到同一虚拟网络中 VM 之间的名称解析。 |客户托管的 DNS 服务器，该服务器在虚拟网络之间转发查询，并由 Azure 进行解析（DNS 代理）。 请参阅[使用自己的 DNS 服务器进行名称解析](#name-resolution-that-uses-your-own-dns-server)。 |仅 FQDN |
| 从应用服务 Web 应用到不同虚拟网络中 VM 之间的名称解析。 |客户托管的 DNS 服务器，该服务器在虚拟网络之间转发查询，并由 Azure 进行解析（DNS 代理）。 请参阅“使用自己的 DNS 服务器进行名称解析”。 |仅 FQDN |
| 解析来自 Azure 中 VM 或角色实例的本地计算机和服务名称。 |客户托管的 DNS 服务器（例如本地域控制器、本地只读域控制器或使用区域传送同步的 DNS 辅助服务器）。 请参阅[使用自己的 DNS 服务器进行名称解析](#name-resolution-that-uses-your-own-dns-server)。 |仅 FQDN |
| 解析本地计算机中的 Azure 主机名。 |将查询转发到相应虚拟网络中客户托管的 DNS 代理服务器，该代理服务器将查询转发到 Azure 进行解析。 请参阅[使用自己的 DNS 服务器进行名称解析](#name-resolution-that-uses-your-own-dns-server)。 |仅 FQDN |
| 针对内部 IP 的反向 DNS。 |[使用自己的 DNS 服务器的名称解析](#name-resolution-that-uses-your-own-dns-server)。 |不适用 |
| 位于不同云服务（而非虚拟网络）中的 VM 或角色实例之间的名称解析。 |不适用。 不同云服务中的 VM 和角色实例之间的连接在虚拟网络外部不受支持。 |不适用|

## <a name="azure-provided-name-resolution"></a>Azure 提供的名称解析

除公共 DNS 名称解析之外，Azure 还为驻留在相同虚拟网络或云服务中的 VM 和角色实例提供内部名称解析。 云服务中的 VM 和实例共享相同的 DNS 后缀，因此仅使用主机名便可。 但在使用经典部署模型部署的虚拟网络中，不同云服务具有不同的 DNS 后缀。 在这种情况下，需要使用 FQDN 解析不同云服务的名称。 在使用 Azure 资源管理器部署模型部署的虚拟网络内，整个虚拟网络中的 DNS 后缀一致，因此无需 FQDN。 DNS 名称可分配给 VM 和网络接口。 虽然 Azure 提供的名称解析不需要任何配置，但并不适合所有部署方案，参见上表详细说明。

> [!NOTE]
> 在使用云服务 Web 和辅助角色的情况下，还可以使用 Azure 服务管理 REST API 访问角色实例的内部 IP 地址。 有关详细信息，请参阅[服务管理 REST API 参考](https://msdn.microsoft.com/library/azure/ee460799.aspx)。 地址基于角色名称和实例编号。 
>
>

### <a name="features"></a>功能

Azure 提供的名称解析包括以下功能：
* 易于使用。 不需要配置。
* 高可用性。 无需创建和管理自己 DNS 服务器的群集。
* 可以结合自己的 DNS 服务器使用该服务，来解析本地主机名和 Azure 主机名。
* 可以在同一云服务中的 VM 和角色实例之间使用名称解析，无需 FQDN。
* 可以在使用 Azure 资源管理器部署模型的虚拟网络中的 VM 之间使用名称解析，无需 FQDN。 经典部署模型中的虚拟网络需要使用 FQDN 来解析不同云服务中的名称。 
* 可以使用最能描述部署的主机名，而不必使用自动生成的名称。

### <a name="considerations"></a>注意事项

使用 Azure 提供的名称解析时要注意的问题：
* 不能修改 Azure 创建的 DNS 后缀。
* 不能手动注册自己的记录。
* 不支持 WINS 和 NetBIOS。 在 Windows 资源管理器中看不到 VM。
* 主机名必须符合 DNS。 名称只能使用 0-9、a-z 和“-”，并且不能以“-”开头或结尾。
* DNS 查询流量按照 VM 进行限制。 限制不会影响大部分应用程序。 如果遵循请求限制，请确保启用客户端缓存。 有关详细信息，请参阅 [DNS 客户端配置](#dns-client-configuration)。
* 在经典部署模型中，每个虚拟网络仅注册前 180 个云服务中的 VM。 此限制不适用于 Azure 资源管理器中的虚拟网络。
* Azure DNS IP 地址为 168.63.129.16。 这是一个静态 IP 地址，将不会更改。

## <a name="dns-client-configuration"></a>DNS 客户端配置

本部分介绍客户端缓存和客户端重试。

### <a name="client-side-caching"></a>客户端缓存

不是每个 DNS 查询都需要跨网络发送。 通过解析本地缓存中的重复性 DNS 查询，客户端缓存有助于减少延迟和提高网络信号的恢复能力。 DNS 记录包含生存时间 (TTL) 机制，这允许缓存尽可能长时间存储记录，而不影响记录刷新。 因此，客户端缓存适用于大多数情况。

默认 Windows DNS 客户端具有内置的 DNS 缓存。 默认情况下，某些 Linux 发行版不包括缓存。 如果没有本地缓存，请向每个 Linux VM 添加 DNS 缓存。

有许多不同的 DNS 缓存包可用（例如 dnsmasq）。 下面介绍如何在最常见的发行版上安装 dnsmasq：

* **Ubuntu（使用 resolvconf）**：
  * 使用 `sudo apt-get install dnsmasq` 安装 dnsmasq 包。
* **SUSE（使用 netconf）**：
  * 使用 `sudo zypper install dnsmasq` 安装 dnsmasq 包。
  * 使用 `systemctl enable dnsmasq.service` 启用 dnsmasq 服务。 
  * 使用 `systemctl start dnsmasq.service` 启动 dnsmasq 服务。 
  * 编辑 **/etc/sysconfig/network/config** 并将 *NETCONFIG_DNS_FORWARDER=""* 更改为 *dnsmasq*。
  * 使用 `netconfig update` 更新 resolv.con，将缓存设置为本地 DNS 解析程序。
* **CentOS （使用 NetworkManager）**:
  * 使用 `sudo yum install dnsmasq` 安装 dnsmasq 包。
  * 使用 `systemctl enable dnsmasq.service` 启用 dnsmasq 服务。
  * 使用 `systemctl start dnsmasq.service` 启动 dnsmasq 服务。
  * 将“prepend domain-name-servers 127.0.0.1;”添加到“/etc/dhclient-eth0.conf”。
  * 使用 `service network restart` 重启网络服务，以将缓存设置为本地 DNS 解析程序。

> [!NOTE]
> dnsmasq 包只是适用于 Linux 的众多 DNS 缓存中的一个。 在使用之前，请检查其是否适合特定需求，且没有安装其他缓存。
>
>
    
### <a name="client-side-retries"></a>客户端重试

DNS 主要是一个 UDP 协议。 因为 UDP 协议无法保证消息传递，所以重试逻辑在DNS 协议本身中处理。 每个 DNS 客户端（操作系统）可能会表现出不同的重试逻辑，具体取决于创建者的偏好：

* Windows 操作系统在 1 秒后重试，然后再在 2 秒后、4 秒后和额外 4 秒后再次重试。 
* 默认 Linux 设置在 5 秒后重试。 我们建议将重试规范更改为 5 次，每隔 1 秒一次。

使用 `cat /etc/resolv.conf` 检查 Linux VM 上的当前设置。 查看“options”行，例如：

```bash
options timeout:1 attempts:5
```

resolv.conf 文件通常是自动生成的，不应进行编辑。 添加 *options* 行的具体步骤因发行版而异：

* **Ubuntu**（使用 resolvconf）：
  1. 将 options 行添加到 /etc/resolveconf/resolv.conf.d/tail。
  2. 运行 `resolvconf -u` 以进行更新。
* **SUSE**（使用 netconf）：
  1. 将“timeout:1 attempts:5”添加到“/etc/sysconfig/network/config”中的 NETCONFIG_DNS_RESOLVER_OPTIONS="" 参数。
  2. 运行 `netconfig update` 以进行更新。
* **CentOS** （使用 NetworkManager）：
  1. 将“echo "options timeout:1 attempts:5"”添加到“/etc/NetworkManager/dispatcher.d/11-dhclient”。
  2. 使用 `service network restart` 进行更新。

## <a name="name-resolution-that-uses-your-own-dns-server"></a>使用自己的 DNS 服务器的名称解析

本部分介绍 VM、角色实例以及 Web 应用。

### <a name="vms-and-role-instances"></a>VM 和角色实例

Azure 提供的功能可能无法满足名称解析的需求。 例如，可能需要使用 Windows Server Active Directory 域，解析虚拟网络之间的 DNS 名称。 考虑到这些情况，Azure 提供了使用自己的 DNS 服务器的功能。

虚拟网络中的 DNS 服务器可将 DNS 查询转发到 Azure 的递归解析程序。 这样，便可以解析该虚拟网络中的主机名。 例如，在 Azure 中运行的域控制器 (DC) 可以响应自身域的 DNS 查询，而将所有其他查询转发到 Azure。 转发查询，VM 就可以（通过 DC）查看本地资源以及（通过转发器）查看 Azure 提供的主机名。 可以通过虚拟 IP 168.63.129.16 访问 Azure 中的递归解析程序。

DNS 转发还可用于在虚拟网络之间进行 DNS 解析，可以通过本地计算机来解析 Azure 提供的主机名。 若要解析 VM 的主机名，DNS 服务器 VM 必须驻留在同一虚拟网络中，并且必须配置为将主机名查询转发到 Azure。 每个虚拟网络的 DNS 后缀不同，因此可以使用条件性转发规则将 DNS 查询发送到正确的虚拟网络进行解析。 下图显示了两个虚拟网络和一个本地网络使用本方法在虚拟网络之间进行 DNS 解析。 DNS 转发器示例可在 [Azure 快速入门模板库](https://azure.microsoft.com/documentation/templates/301-dns-forwarder/)和 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder) 中获取。

> [!NOTE]
> 角色实例可对同一虚拟网络中的 VM 执行名称解析， 方法是使用由 VM 主机名和 **internal.cloudapp.net** DNS 后缀组成的 FQDN。 但是，在这种情况下，仅当角色实例在[角色架构（.cscfg 文件）](https://msdn.microsoft.com/library/azure/jj156212.aspx)中定义了 VM 名称时，名称解析才会成功。
> `<Role name="<role-name>" vmName="<vm-name>">`
>
> 需要在另一个虚拟网络中执行 VM 名称解析的角色实例（使用 **internal.cloudapp.net** 后缀的 FQDN）必须使用本部分所述的方法（在两个虚拟网络之间进行自定义 DNS 服务器转发）。
>

![虚拟网络之间的 DNS 示意图](./media/virtual-networks-name-resolution-for-vms-and-role-instances/inter-vnet-dns.png)

使用 Azure 提供的名称解析时，Azure 动态主机配置协议 (DHCP) 将为每个 VM 提供内部 DNS 后缀 (**.internal.cloudapp.net**)。 此后缀可实现主机名解析，因为主机名记录位于 **internal.cloudapp.net** 区域中。 使用自己的名称解析解决方案时，不会向 VM 提供此后缀，因为该后缀会干扰其他 DNS 体系结构（例如已加入域的方案）。 相反，Azure 会提供没有实际功能的占位符 (reddog.microsoft.com)。

如果需要，可以使用 PowerShell 或 API 确定内部 DNS 后缀：

* 对于 Azure 资源管理器部署模型中的虚拟网络，后缀是可通过[网络接口卡 REST API](https://docs.microsoft.com/rest/api/virtualnetwork/networkinterfaces)，则[Get AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) PowerShell cmdlet 和[az 网络 nic show](/cli/azure/network/nic#az-network-nic-show) Azure CLI 命令。
* 在经典部署模型中，可以通过 [Get Deployment API](https://msdn.microsoft.com/library/azure/ee460804.aspx) 调用或 [Get-AzureVM -Debug](/powershell/module/servicemanagement/azure/get-azurevm) cmdlet 获取该后缀。

如果不想将查询转发到 Azure，应提供自己的 DNS 解析。 DNS 解决方案需要：

* 提供合适的主机名解析（例如，通过 [DDNS](virtual-networks-name-resolution-ddns.md)）。 如果使用 DDNS，则可能需要禁用 DNS 记录清理。 Azure DHCP 租约时间很长，进行清理可能会导致 DNS 记录过早删除。 
* 提供适当的递归式解析来解析外部域名。
* 可以从其所服务的客户端进行访问（端口 53 上的 TCP 和 UDP），并可访问 Internet。
* 禁止从 Internet 进行访问，减少外部代理带来的威胁。

> [!NOTE]
> 为获得最佳性能，在将 Azure VM 用作 DNS 服务器时，应禁用 IPv6。 [公共 IP 地址](virtual-network-public-ip-address.md) 应分配给每个 DNS 服务器 VM。 如需获取更多性能分析和优化（使用 Windows Server 作为 DNS 服务器时），请参阅[递归 Windows DNS Server 2012 R2 的名称解析性能](https://blogs.technet.com/b/networking/archive/2015/08/19/name-resolution-performance-of-a-recursive-windows-dns-server-2012-r2.aspx)。
> 
> 

### <a name="web-apps"></a>Web 应用
假设你需要执行从使用应用服务生成的、已链接到某个虚拟网络的 Web 应用到同一虚拟网络中的 VM 的名称解析。 除了设置具有 DNS 转发程序（可向 Azure 转发查询）的自定义 DNS 服务器（虚拟 IP 为 168.63.129.16）以外，还需要执行以下步骤：
1. 根据[将应用与虚拟网络集成](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)中所述，为 Web 应用启用虚拟网络集成（如果尚未启用）。
2. 在 Azure 门户中，对于托管 Web 应用的应用服务计划，选择“网络”>“虚拟网络集成”下的“同步网络”。

    ![虚拟网络名称解析的屏幕截图](./media/virtual-networks-name-resolution-for-vms-and-role-instances/webapps-dns.png)

如需从使用应用服务构建的链接到虚拟网络的 Web 应用对不同虚拟网络中的 VM 执行名称解析，需要同时在两个虚拟网络中使用自定义 DNS 服务器，如下所示：

* 在某个也可向 Azure 中的递归解析程序（虚拟 IP 为 168.63.129.16）转发查询的 VM 上的目标虚拟网络中设置 DNS 服务器。 DNS 转发器示例可在 [Azure 快速入门模板库](https://azure.microsoft.com/documentation/templates/301-dns-forwarder)和 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder) 中获取。 
* 在某个 VM 上的源虚拟网络中设置 DNS 转发程序。 将此 DNS 转发器配置为向目标虚拟网络中的 DNS 服务器转发查询。
* 在源虚拟网络的设置中配置源 DNS 服务器。
* 遵照[将应用与虚拟网络集成](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)中的说明，为 Web 应用启用虚拟网络集成以链接到源虚拟网络。
* 在 Azure 门户中，对于托管 Web 应用的应用服务计划，选择“网络”>“虚拟网络集成”下的“同步网络”。

## <a name="specify-dns-servers"></a>指定 DNS 服务器
使用自己的 DNS 服务器时，Azure 允许为每个虚拟网络指定多个 DNS 服务器。 也可以针对每个网络接口（适用于 Azure 资源管理器）或云服务（适用于经典部署模型）指定多个 DNS 服务器。 为网络接口或云服务指定 DNS 服务器时，其优先级高于为虚拟网络指定的 DNS 服务器。

> [!NOTE]
> 不应直接在 Windows VM 中编辑网络连接属性，例如 DNS 服务器 IP。 这是因为，在更换虚拟网络适配器后，可能会在服务修复过程中擦除这些属性。
>
>

使用 Azure 资源管理器部署模型时，可为虚拟网络和网络接口指定 DNS 服务器。 有关详细信息，请参阅[管理虚拟网络](manage-virtual-network.md)和[管理网络接口](virtual-network-network-interface.md)。

> [!NOTE]
> 如果为虚拟网络选择自定义 DNS 服务器，则必须至少指定一个 DNS 服务器 IP 地址；否则，虚拟网络将忽略配置，而改用由 Azure 提供的 DNS。
>
>

使用经典部署模型时，可以在 Azure 门户或[网络配置文件](https://msdn.microsoft.com/library/azure/jj157100)中指定虚拟网络的 DNS 服务器。 对于云服务器，可以通过[服务配置文件](https://msdn.microsoft.com/library/azure/ee758710)或者在 PowerShell 中使用 [New-AzureVM](/powershell/module/servicemanagement/azure/new-azurevm) 指定 DNS 服务器。

> [!NOTE]
> 如果更改已部署的虚拟网络或虚拟机的 DNS 设置，则需重启每个受影响的 VM，所做的更改才会生效。
>
>

## <a name="next-steps"></a>后续步骤

Azure 资源管理器部署模型

* [管理虚拟网络](manage-virtual-network.md)
* [管理网络接口](virtual-network-network-interface.md)

经典部署模型：

* [Azure 服务配置架构](https://msdn.microsoft.com/library/azure/ee758710)
* [虚拟网络配置架构](https://msdn.microsoft.com/library/azure/jj157100)
* [使用网络配置文件配置虚拟网络](virtual-networks-using-network-configuration-file.md)
