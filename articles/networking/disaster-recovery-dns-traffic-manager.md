---
title: 使用 Azure DNS 和流量管理器执行灾难恢复 | Microsoft Docs
description: 概述了使用 Azure DNS 和流量管理器的灾难恢复解决方案。
services: dns
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/08/2018
ms.author: kumud
ms.openlocfilehash: a560cc526e73f3ce7e851f2a545f9b16fa53b423
ms.sourcegitcommit: 1d257ad14ab837dd13145a6908bc0ed7af7f50a2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65501691"
---
# <a name="disaster-recovery-using-azure-dns-and-traffic-manager"></a>使用 Azure DNS 和流量管理器进行灾难恢复

灾难恢复侧重于从严重的应用程序功能丧失中恢复。 若要选择灾难恢复解决方案，业务和技术所有者必须先确定必需的灾难期间功能级别，如不可用、精简功能后部分可用、延迟可用或完全可用。
大多数企业客户选择多区域体系结构，以通过应用程序级或基础结构级故障转移复原。 客户可以选择多种方法，以通过冗余体系结构实现故障转移和高可用性。 下面是一些常用方法：

- **使用冷备用的主动-被动**：在此故障转移解决方案中，在需要故障转移前，VM 和备用区域中运行的其他设备未处于活动状态。 不过，生产环境是以备份、VM 映像或资源管理器模板的形式复制到其他区域。 这种故障转移机制经济高效，但需要较长时间才能完成整个故障转移。
 
    ![使用冷备用的主动/被动](./media/disaster-recovery-dns-traffic-manager/active-passive-with-cold-standby.png)
    
    图：使用冷备用的主动/被动灾难恢复配置

- **使用指示灯的主动/被动**：在此故障转移解决方案中，备用环境采用最简配置。 只配置运行必要的服务，就能支持关键的最简应用程序集。 在本机形式下，此方案只能执行最简功能，但如果发生故障转移，可以纵向扩展并生成附加服务来处理大量生产负载。
    
    ![使用指示灯的主动/被动](./media/disaster-recovery-dns-traffic-manager/active-passive-with-pilot-light.png)
    
    图：使用指示灯的主动/被动灾难恢复配置

- **使用热备用的主动/被动**：在此故障转移解决方案中，备用区域会进行预热并能处理基础负载，自动缩放已启用，并且所有实例都正常运行。 此解决方案不会通过缩放来处理全部生产负载，但是有用的，且所有服务都正常运行。 此解决方案是指示灯方法的增强型版本。
    
    ![使用热备用的主动/被动](./media/disaster-recovery-dns-traffic-manager/active-passive-with-warm-standby.png)
    
    图：使用热备用的主动/被动灾难恢复配置
    
若要详细了解故障转移和高可用性，请参阅 [Azure 应用程序灾难恢复](https://docs.microsoft.com/azure/architecture/resiliency/disaster-recovery-azure-applications)。


## <a name="planning-your-disaster-recovery-architecture"></a>规划灾难恢复体系结构

创建灾难恢复体系结构时，有以下两个技术现状：
-  使用部署机制在主环境和备用环境之间复制实例、数据和配置。 这种灾难恢复可通过 Microsoft Azure 合作伙伴设备/服务（如 Veritas 或 NetApp）使用 Azure Site Recovery 在本机完成。 
- 开发一种将网络/Web 流量从主站点转移到备用站点的解决方案。 这种灾难恢复可通过 Azure DNS、Azure 流量管理器 (DNS) 或第三方全局负载均衡器实现。

本文只介绍了网络和 Web 流量重定向方法。 若要了解如何设置 Azure Site Recovery，请参阅 [Azure Site Recovery 文档](https://docs.microsoft.com/azure/site-recovery/)。
DNS 是转移网络流量的最高效机制之一，因为 DNS 通常是全局的，位于数据中心外部，且不受任何区域级或可用性区域 (AZ) 级故障影响。 可以使用基于 DNS 的故障转移机制。在 Azure 中，有两个 DNS 服务可以某种方式完成相同任务，即 Azure DNS（权威 DNS）和 Azure 流量管理器（基于 DNS 的智能流量路由）。 

请务必了解本文为介绍解决方案而广泛使用的一些 DNS 概念：
- **DNS A 记录**– A 记录都是域指向 IPv4 地址的指针。 
- **CNAME 或 Canonical 名称** - 此记录类型用于指向其他 DNS 记录。 CNAME 响应不返回 IP 地址，而是返回指向包含 IP 地址的记录的指针。 
- **加权路由** - 可以关联权重和服务终结点，然后根据分配的权重来分布流量。 这种路由方法是流量管理器提供的四种流量路由机制之一。 有关详细信息，请参阅[加权路由方法](../traffic-manager/traffic-manager-routing-methods.md#weighted)。
- **优先级路由** - 优先级路由以终结点的运行状况检查为依据。 默认情况下，Azure 流量管理器将所有流量都发送到优先级最高的终结点。在发生故障或灾难后，流量管理器将流量路由到辅助终结点。 有关详细信息，请参阅[优先级路由方法](../traffic-manager/traffic-manager-routing-methods.md#priority)。

## <a name="manual-failover-using-azure-dns"></a>使用 Azure DNS 执行手动故障转移
用于灾难恢复的 Azure DNS 手动故障转移解决方案使用标准的 DNS 机制通过故障转移复原到备份站点。 Azure DNS 手动故障转移解决方案与冷备用或指示灯方法一起使用时效果最佳。 

![使用 Azure DNS 执行手动故障转移](./media/disaster-recovery-dns-traffic-manager/manual-failover-using-dns.png)

图：使用 Azure DNS 执行手动故障转移

为此解决方案做出了如下假设：
- 主终结点和辅助终结点使用不经常变化的静态 IP。 假设主站点的 IP 为 100.168.124.44，辅助站点的 IP 为 100.168.124.43。
- 主站点和辅助站点均有对应的 Azure DNS 区域。 假设主站点的终结点为 prod.contoso.com，备份站点的终结点为 dr.contoso.com。 名为 www 主应用程序的 DNS 记录\.也存在于 contoso.com。   
- TTL 不高于组织中设置的 RTO SLA。 例如，如果企业将应用程序灾难响应 RTO 设置为 60 分钟，TTL 应短于 60 分钟，最好是越低越好。 
  设置 Azure DNS 手动故障转移的具体步骤如下：
- 创建 DNS 区域
- 创建 DNS 区域记录
- 更新 CNAME 记录

### <a name="step-1-create-a-dns"></a>步骤 1：创建 DNS
创建 DNS 区域 (例如，www\.contoso.com)，如下所示：

![在 Azure 中创建 DNS 区域](./media/disaster-recovery-dns-traffic-manager/create-dns-zone.png)

图：在 Azure 中创建 DNS 区域

### <a name="step-2-create-dns-zone-records"></a>步骤 2：创建 DNS 区域记录

在此区域中创建三条记录 (例如 www\.contoso.com、 prod.contoso.com 和 dr.consoto.com) 如下所示。

![创建 DNS 区域记录](./media/disaster-recovery-dns-traffic-manager/create-dns-zone-records.png)

图：在 Azure 中创建 DNS 区域记录

在此方案、 站点、 www\.contoso.com 已 TTL 为 30 分钟，这远低于规定的 RTO，并指向生产站点 prod.contoso.com。 此配置适用于常规业务操作。 prod.contoso.com 和 dr.contoso.com 的 TTL 已设置为 300 秒或 5 分钟。 可以使用 Azure 监视服务，如 Azure Monitor、Azure App Insights 或任何合作伙伴监视解决方案（如 Dynatrace）。甚至可以使用自行开发的解决方案来监视或检测应用程序级或虚拟基础结构级故障。

### <a name="step-3-update-the-cname-record"></a>步骤 3：更新 CNAME 记录

检测到故障后，立即将记录值更改为指向 dr.contoso.com，如下所示：
       
![更新 CNAME 记录](./media/disaster-recovery-dns-traffic-manager/update-cname-record.png)

图：在 Azure 中更新 CNAME 记录

在 30 分钟内，在此期间，大多数冲突解决程序将刷新缓存的区域文件中，任何查询到 www\.contoso.com 将重定向到 dr.contoso.com。
还可以运行下面的 Azure CLI 命令来更改 CNAME 值：
 ```azurecli
   az network dns record-set cname set-record \
   --resource-group 123 \
   --zone-name contoso.com \
   --record-set-name www \
   --cname dr.contoso.com
```
这一步可以手动执行，也可以自动执行。 若要手动完成，可以使用控制台或 Azure CLI。 Azure SDK 和 API 可用于自动更新 CNAME，无需手动干预。 可以通过 Azure 函数或在第三方监视应用程序内设置自动更新，甚至可以在本地设置。

### <a name="how-manual-failover-works-using-azure-dns"></a>Azure DNS 手动故障转移的工作原理
由于 DNS 服务器不在故障转移或灾难区域内，因此不受任何故障时间影响。 这样一来，用户可以构建简单、经济高效且一直工作的故障转移方案，但前提是操作者在灾难期间联网，并能进行翻转。 如果解决方案已脚本化，必须确保运行脚本的服务器或服务应不受影响生产环境的问题影响。 另请注意，已针对区域设置低 TTL，这样全世界没有任何解析程序能够长时间缓存终结点，并且客户可以在 RTO 内访问站点。 对于冷备用和指示灯方法，由于可能需要进行一些预热和其他管理活动，还应在翻转前留出足够长的时间。

## <a name="automatic-failover-using-azure-traffic-manager"></a>使用 Azure 流量管理器执行自动故障转移
若有复杂的体系结构和多组能执行相同功能的资源，可以将 Azure 流量管理器（基于 DNS）配置为检查资源的运行状况，并将来自不正常运行的资源的流量路由到正常运行的资源。 在下面的示例中，主要区域和次要区域均采用完全部署。 这种部署包括云服务和同步数据库。 

![使用 Azure 流量管理器执行自动故障转移](./media/disaster-recovery-dns-traffic-manager/automatic-failover-using-traffic-manager.png)

图：使用 Azure 流量管理器执行自动故障转移

但是，只有主要区域在主动处理来自用户的网络请求。 只有当主要区域出现服务中断时，次要区域才会激活。 在这种情况下，会将所有新网络请求路由到次要区域。 由于数据库备份几乎是瞬间完成，因此两个负载均衡器都有可对其检查运行状况的 IP，并且实例始终正常运行。借助此拓扑，无需任何手动干预，即可执行低 RTO 的故障转移。 在主要区域发生故障后，次要故障转移区域必须可供立即投入使用。
此方案非常适合使用 Azure 流量管理器完成，其中内置了可执行各种运行状况检查的探针，包括 http/https 和 TCP。 Azure 流量管理器还包含规则引擎，可以配置为在故障发生时执行故障转移，如下所述。 假设使用流量管理器执行以下解决方案：
- 客户有静态 IP 为 100.168.124.44 的区域 #1 终结点 prod.contoso.com，以及静态 IP 为 100.168.124.43 的区域 #2 终结点 dr.contoso.com。 
-   其中每个环境都通过面向公众的属性（如负载均衡器）前置。 负载平衡器可以配置为，拥有基于 DNS 的终结点或完全限定的域名 (FQDN)，如上所述。
-   区域 2 中的所有实例与区域 1 是近实时复制关系。 此外，计算机映像是最新的，所有软件/配置数据都进行了修补并与区域 1 保持一致。  
-   自动缩放已提前预先配置。 

Azure 流量管理器自动故障转移的配置步骤如下：
1. 新建 Azure 流量管理器配置文件
2. 在流量管理器配置文件中创建终结点
3. 设置运行状况检查和故障转移配置

### <a name="step-1-create-a-new-azure-traffic-manager-profile"></a>步骤 1：新建 Azure 流量管理器配置文件
新建 Azure 流量管理器配置文件，并命名为“contoso123”，再选择“优先级”作为“路由方法”。 若有要与之关联的现有资源组，可以选择现有资源组，否则新建资源组。

![创建流量管理器配置文件](./media/disaster-recovery-dns-traffic-manager/create-traffic-manager-profile.png)

*图-创建流量管理器配置文件*

### <a name="step-2-create-endpoints-within-the-traffic-manager-profile"></a>步骤 2：在流量管理器配置文件中创建终结点

在这一步，创建指向生产站点和灾难恢复站点的终结点。 此时，选择“类型”作为外部终结点，但如果资源托管在 Azure 中，也可以选择“Azure 终结点”。 如果选择“Azure 终结点”，请选择 Azure 分配的“应用服务”或“公共 IP”作为“目标资源”。 优先级设置为“1”，因为它是区域 1 的主服务。
同样，也在流量管理器中创建灾难恢复终结点。

![创建灾难恢复终结点](./media/disaster-recovery-dns-traffic-manager/create-disaster-recovery-endpoint.png)

图：创建灾难恢复终结点

### <a name="step-3-set-up-health-check-and-failover-configuration"></a>步骤 3：设置运行状况检查和故障转移配置

在这一步，将 DNS TTL 设置为 10 秒，大多数面向 Internet 的递归解析程序都采用此设置。 此配置意味着，没有 DNS 解析程序会缓存信息超过 10 秒。 对于终结点监视设置，“路径”当前设置为“/”或根路径，但也可以将终结点设置自定义为评估路径（例如，prod.contoso.com/index）。 下面的示例展示了使用 https 作为探测协议。 不过，也可以选用 http或 tcp。 协议选择取决于最终应用程序。 “探测时间间隔”设置为 10 秒（可启用快速探测），重试次数设置为 3 次。 因此，如果三个连续时间间隔的记录结果都为故障，流量管理器就会通过故障转移复原到第二个终结点。 自动故障转移总时间的计算公式如下：故障转移时间 = TTL + 重试次数 * 探测时间间隔。在此示例中，值为 10 + 3 * 10 = 40 秒（最大）。
如果重试次数设置为 1 次，且 TTL 设置为 10 秒，那么故障转移时间为 10 + 1 * 10 = 20 秒。 请将重试次数设置为大于 1 的值，这样就不可能因误报或任何网络小跳点而执行故障转移。 


![设置运行状况检查](./media/disaster-recovery-dns-traffic-manager/set-up-health-check.png)

图：设置运行状况检查和故障转移配置

### <a name="how-automatic-failover-works-using-traffic-manager"></a>流量管理器自动故障转移的工作原理

灾难期间，将对主终结点进行探测，并将状态更改为“已降级”，而灾难恢复站点的状态一直为“联机”。 默认情况下，流量管理器将所有流量发送到主终结点（优先级最高）。 如果主终结点的状态显示为“已降级”，流量管理器会将流量路由到第二个终结点（只要它保持正常运行）。 可以视需要在流量管理器中配置更多终结点，以用作附加故障转移终结点，或用作负载均衡器在终结点之间共同均衡负载。

## <a name="next-steps"></a>后续步骤
- 详细了解 [Azure 流量管理器](../traffic-manager/traffic-manager-overview.md)。
- 详细了解 [Azure DNS](../dns/dns-overview.md)。









