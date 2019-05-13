---
title: 了解 Azure AD 应用程序代理连接器 | Microsoft 文档
description: 介绍有关 Azure AD 应用程序代理连接器的基础知识。
services: active-directory
author: CelesteDG
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 11/15/2018
ms.author: celested
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb2a2aa8204ef442bbe3a0e6ff9018cd3f153910
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406494"
---
# <a name="understand-azure-ad-application-proxy-connectors"></a>了解 Azure AD 应用程序代理连接器

连接器使 Azure AD 应用程序代理成为可能。 连接器的形式简单、易于部署和维护，且功能超强。 本文介绍连接器的概念及其工作原理，以及有关如何优化部署的建议。 

## <a name="what-is-an-application-proxy-connector"></a>什么是应用程序代理连接器？

连接器是位于本地的轻型代理，能够帮助与应用程序代理服务建立出站连接。 必须将连接器安装在能够访问后端应用程序的 Windows Server 上。 可将连接器组织成连接器组，每个组处理发往特定应用程序的流量。

## <a name="requirements-and-deployment"></a>要求和部署

若要成功部署应用程序代理，至少需要一个连接器，但我们建议部署两个或更多个连接器来提高弹性。 运行 Windows Server 2012 R2 的计算机上安装连接器或更高版本。 连接器需要能够与应用程序代理服务以及发布的本地应用程序通信。 

### <a name="windows-server"></a>Windows Server
你需要具有运行 Windows Server 2012 R2 或更高版本的服务器，可以在该服务器上安装应用程序代理连接器。 该服务器需要连接到 Azure 中的应用程序代理服务和你要发布的本地应用程序。

在安装应用程序代理连接器之前，Windows Server 需要启用 TLS 1.2。 版本低于 1.5.612.0 的现有连接器继续使用早期版本的 TLS，直到另行通知。 若要启用 TLS 1.2，请执行以下操作：

1. 设置以下注册表项：
    
    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

2. 重启服务器


有关连接器服务器的网络要求的详细信息，请参阅[开始使用应用程序代理和安装连接器](application-proxy-add-on-premises-application.md)。

## <a name="maintenance"></a>维护
连接器和服务负责处理所有的高可用性任务。 可以动态添加或删除这些连接器和服务。 每当有新请求抵达时，该请求会路由到当前可用的连接器之一。 如果某个连接器暂时不可用，它不会对此流量做出响应。

连接器是无状态的，在计算机上没有配置数据。 它们存储的唯一数据是有关连接服务的设置及其身份验证证书。 连接器在连接到服务时，将提取全部所需的配置数据，并每隔几分钟刷新这些数据。

连接器还会轮询服务器，确定是否有更高版本的连接器。 如果找到了更高的版本，连接器将更新自身。

可以在运行连接器的计算机上，使用事件日志和性能计数器。 或者，可以通过 Azure 门户的“应用程序代理”页查看连接器的状态：

 ![AzureAD 应用程序代理连接器](./media/application-proxy-connectors/app-proxy-connectors.png)

无需手动删除未使用的连接器。 当连接器运行时，它会在连接到服务时保持活动状态。 未使用的连接器标记为非活动状态，保持这种状态 10 天后将被删除。 不过，如果想要卸载某个连接器，则需要在服务器中卸载连接器服务和更新程序服务。 重新启动计算机以完全删除该服务。

## <a name="automatic-updates"></a>自动更新

Azure AD 为部署的所有连接器提供自动更新。 只要应用程序代理连接器更新程序服务保持运行，连接器就会自动更新。 如果在服务器上未看到连接器更新程序服务，需要[重新安装连接器](application-proxy-add-on-premises-application.md)才能获得所有更新。 

如果不想等连接器自动更新，可以执行手动升级。 转到连接器所在服务器上的[连接器下载页](https://download.msappproxy.net/subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/connector/download)，选择“下载”。 此过程启动本地连接器升级。 

对于有多个连接器的租户，每次自动更新每个组中的一个连接器，以防环境中出现停机现象。 

连接器更新时，可能会在以下情况下遭遇停机：  
- 你只有一个连接器，我们建议你安装第二个连接器并[创建连接器组](application-proxy-connector-groups.md)。 这将避免停机时间，并提供更高的可用性。  
- 更新开始时，连接器处于事务中间。 尽管初始事务已丢失，但浏览器应会自动重试操作，或者你可以自行刷新页面。 重新发送请求时，流量将路由到备份连接器。

要查看有关以前发布的版本及其包含的更改的信息，请参阅[应用程序代理 - 版本发布历史记录](application-proxy-release-version-history.md)。

## <a name="creating-connector-groups"></a>创建连接器组

通过连接器组，可以分配特定连接器，以便为特定的应用程序提供服务。 可以把大量连接器集合到一起，然后将每个应用程序分配给一个组。 

使用连接器组，可以更轻松地管理大型部署。 它们还为托管在不同区域中的租户改善延迟，因为可以创建基于位置的连接器组，从而仅为本地应用程序提供服务。 

有关连接器组的详细信息，请参阅[使用连接器组在单独的网络和位置上发布应用程序](application-proxy-connector-groups.md)。

## <a name="capacity-planning"></a>容量计划 

请务必确保已在连接器之间规划足够的容量以处理预期的流量。 通常，用户越多，需要的计算机容量越大。 下表概述了不同计算机可以处理的数据量。 请注意，它们全都基于预期的每秒事务数 (TPS) 而非按用户计算的，因为使用模式会变化，无法用来预测负载。 还会根据响应大小和后端应用程序响应时间，会有一些差异 - 较大的响应大小和较慢的响应时间将产生较低的最大 TPS。 我们建议使用其他计算机，以便计算机上的分布式负载大约为 50%。 额外的容量将确保高可用性和复原能力。

|内核|RAM|预期的延迟 (MS)-P99|最大 TPS|
| ----- | ----- | ----- | ----- |
|2|8|325|586|
|4|16|320|1150|
|8|32|270|1190|
|16|64|245|1200*|

\* 此计算机使用自定义设置以提高某些超出.NET 推荐的设置的默认连接限制。 我们建议先使用默认设置运行测试，再联系支持人员以为你的租户更改此限制。
 
>[!NOTE]
>在 4 核心、8 核心和 16 核心计算机之间，最大 TPS 没有多大区别。 主要区别体现在预期的延迟。  

## <a name="security-and-networking"></a>安全和网络

可将连接器安装在网络中允许连接器向应用程序代理服务发送请求的任意位置。 重要的是，运行连接器的计算机还有权访问应用。 可将连接器安装在企业网络内部，或云中运行的虚拟机上。 连接器可在外围网络，也称为外围安全区域 (DMZ) 中运行，但没有必要，网络始终是安全，因为所有流量出站。

连接器只会发送出站请求。 出站流量将依次发送到应用程序代理服务和发布的应用程序。 无需打开入站端口，因为一旦建立会话，流量就会双向流动。 也无需通过防火墙配置入站访问。 

有关配置出站防火墙规则的详细信息，请参阅[使用现有的本地代理服务器](application-proxy-configure-connectors-with-proxy-servers.md)。


## <a name="performance-and-scalability"></a>性能和可伸缩性

应用程序代理服务的规模是透明的，但对于连接器而言，规模是一种可变因素。 需要提供足够的连接器才能处理高峰流量。 由于连接器是无状态的，它们不受用户或会话数目影响。 而会对请求的数目及其有效负载大小进行响应。 使用标准 Web 流量，一台普通的计算机每秒就能处理几千个请求。 具体能够处理多少，取决于确切的计算机特征。 

连接器性能受 CPU 和网络的约束。 SSL 加密和解密依赖于 CPU 性能，而网络性能对于快速连接到应用程序和 Azure 中的联机服务非常重要。

相比之下，内存对于连接器来说不是一个很大的问题。 联机服务会处理大部分处理负载，以及所有未经身份验证的流量。 可在云中完成的所有任务会在云中完成。 

如果连接器或计算机因任何原因而变得不可用，流量便会开始流入组中的另一个连接器。 此复原也是我们建议使用多个连接器的原因。

影响性能的另一个因素是连接器之间的网络质量，包括： 

* **联机服务**：与 Azure 中应用程序代理的连接速度慢或延迟高会影响连接器性能。 为获得最佳性能，请使用 Express Route 将组织连接到 Azure。 否则，网络团队应确保以尽量高效的方式来处理 Azure 的连接。 
* **后端应用程序**：在某些情况下，连接器与后端应用程序之间存在其他可能会减慢或阻止连接的代理。 若要排查这种情况，请从连接器服务器打开浏览器，并尝试访问应用程序。 如果连接器在 Azure 中运行，而这些应用程序位于本地，则用户的体验可能与期望不符。
* **域控制器**：如果连接器使用 Kerberos 约束委派执行单一登录 (SSO)，在向后端发送请求之前，它们会联系域控制器。 连接器提供 Kerberos 票证缓存，但在繁忙的环境中，域控制器的响应能力可能会影响性能。 如果连接器在 Azure 中运行但与本地的域控制器通信，则更容易出现此问题。 

有关优化网络的详细信息，请参阅[使用 Azure Active Directory 应用程序代理时的网络拓扑注意事项](application-proxy-network-topology.md)。

## <a name="domain-joining"></a>域加入

连接器可在未加入域的计算机上运行。 但是，如果想要对使用集成 Windows 身份验证 (IWA) 的应用程序使用单一登录 (SSO)，则需要一个已加入域的计算机。 在这种情况下，必须将连接器计算机加入到可代表已发布应用程序的用户执行 [Kerberos](https://web.mit.edu/kerberos) 约束委派的域。

还可以将连接器加入到具有部分信任的域或林，或者加入到只读的域控制器。

## <a name="connector-deployments-on-hardened-environments"></a>强化的环境中的连接器部署

通常，连接器部署直截了当，无需经过特殊的配置。 但应当考虑一些特殊条件：

* 限制出站流量的组织必须[打开所需端口](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment)。
* 符合 FIPS 规范的计算机可能需要更改其配置才能允许连接器进程生成和存储证书。
* 根据网络请求发出过程锁定其环境的组织必须确保启用这两个连接器服务，以访问全部所需的端口和 IP。
* 在某些情况下，出站正向代理可能会中断双向证书身份验证，导致通信失败。

## <a name="connector-authentication"></a>连接器身份验证

为了提供安全服务，连接器必须向服务执行身份验证，而服务必须向连接器执行身份验证。 当连接器发起连接时，将使用客户端和服务器证书完成这种身份验证。 这样，管理员的用户名和密码就不会存储在连接器计算机上。

使用的证书特定于应用程序代理服务。 这些证书是在初始注册期间创建的，每隔几个月由连接器自动续订。 

如果连接器有几个月未连接到服务，其证书可能会过时。 在这种情况下，需卸载后重装连接器来触发注册。 可运行以下 PowerShell 命令：

```
Import-module AppProxyPSModule
Register-AppProxyConnector
```

## <a name="under-the-hood"></a>揭秘

连接器基于 Windows Server Web 应用程序代理，因此它们所具有的管理工具大部分相同，包括 Windows 事件日志

 ![使用事件查看器管理事件日志](./media/application-proxy-connectors/event-view-window.png)

和 Windows 性能计数器。 

 ![使用性能监视器向连接器添加计数器](./media/application-proxy-connectors/performance-monitor.png)

连接器提供管理日志和会话日志。 管理日志包括关键事件及其错误。 会话日志包括所有事务及其处理详细信息。 

要查看日志，请转到事件查看器，打开“视图”菜单，并启用“显示分析和调试日志”。 然后，启用这些日志以开始收集事件。 这些日志不会显示在 Windows Server 2012 R2 上的 Web 应用程序代理中，因为连接器基于更新的版本。

可在“服务”窗口中检查服务的状态。 连接器由两个 Windows 服务组成：实际的连接器和更新程序。 二者都必须一直运行。

 ![AzureAD 本地服务](./media/application-proxy-connectors/aad-connector-services.png)

## <a name="next-steps"></a>后续步骤


* [使用连接器组在单独的网络和位置上发布应用程序](application-proxy-connector-groups.md)
* [使用现有的本地代理服务器](application-proxy-configure-connectors-with-proxy-servers.md)
* [对应用程序代理和连接器错误进行故障排除](application-proxy-troubleshoot.md)
* [如何以无提示方式安装 Azure AD 应用程序代理连接器](application-proxy-register-connector-powershell.md)

