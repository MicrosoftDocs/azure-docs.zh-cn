---
title: 什么是 Azure Redis 缓存？ | Microsoft Docs
description: 了解什么是 Azure Redis 缓存及其常用方式。
services: cache
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache
ms.devlang: na
ms.topic: overview
ms.date: 03/26/2018
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: fd790d27c958bf982f95b98426c6ab4d94c5f17f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412738"
---
# <a name="azure-cache-for-redis-description"></a>Azure Redis 缓存说明

Azure Redis 缓存基于热门软件 [Redis](https://redis.io/)。 它通常用作缓存，目的是提高极其依赖后端数据存储的系统的性能和可伸缩性。 提高性能的方式是将经常访问的数据暂时复制到靠近应用程序的快速存储。 使用 [Azure Redis 缓存](https://redis.io/)时，此快速存储位于 Azure Redis 缓存的内存中，而不是通过数据库从磁盘加载。

Azure Redis 缓存也可用作内存中数据结构存储、分布式非关系数据库以及消息代理。 若要提高应用程序性能，可充分利用 Redis 引擎低延迟、高吞吐量的性能特点。

Azure Redis 缓存提供了安全的专用 Redis 缓存。 Azure Redis 缓存由 Microsoft 进行管理并承载在 Azure 中，可供 Azure 内外的任何应用程序访问。

## <a name="using-azure-cache-for-redis"></a>使用 Azure Redis 缓存

可以通过多种模式使用 Azure Redis 缓存来支持应用程序体系结构或提高应用程序性能。 部分最常用的模式包括：

| 模式      | 说明                                        |
| ------------ | -------------------------------------------------- |
| [缓存端](cache-web-app-cache-aside-leaderboard.md) | 由于数据库可能很大，因此不建议将整个数据库加载到缓存中。 通常使用[缓存端](https://docs.microsoft.com/azure/architecture/patterns/cache-aside)模式，只在需要时才将数据项加载到缓存中。 系统在更改后端数据时，也会同时更新分布到其他客户端的缓存。 另外，系统还可以设置数据项的过期时间，或者通过逐出策略将数据更新重新加载到缓存中。|
| [内容缓存](cache-aspnet-output-cache-provider.md) | 大多数通过模板生成的网页会带有页眉、页脚、工具栏、菜单，等等。这些网页实际上不经常变化，不应以动态方式生成。 与使用后端数据存储相比，使用内存中缓存（例如 Azure Redis 缓存）可以让 Web 服务器快速访问此类静态内容。 此模式可减少以动态方式生成内容所需的处理时间和服务器负荷。 这样可以提高 Web 服务器的响应能力，减少处理负荷所需的服务器数。 Azure Redis 缓存提供 Redis 输出缓存提供程序，支持对 ASP.NET 使用此模式。|
| [用户会话缓存](cache-aspnet-session-state-provider.md) | 此模式通常用于购物车和其他用户历史记录类型的信息。Web 应用程序可能需要将此类信息与用户 Cookie 相关联。 在 Cookie 中存储过多内容可能会对性能造成负面影响，因为 Cookie 会变大，并且每次请求都需要传递和验证 Cookie。 常用解决方案是使用 Cookie 作为键来查询后端数据库中的数据。 使用内存中缓存（例如 Azure Redis 缓存）将信息与用户关联在速度上要比与整个关系数据库交互快得多。 |
| 作业和消息队列 | 当应用程序收到请求时，通常还需要额外的时间来执行与请求相关联的操作。 通常会将运行时间较长的操作添加到队列中，留待以后处理（可能由其他服务器处理）。 这种将工作推迟的方法称为任务队列。 多种软件组件专用于提供任务队列支持。 Azure Redis 缓存也以分布式队列的方式提供此支持。|
| 分布式事务 | 通常会要求应用程序能够以单个操作（原子操作）的方式对后端数据存储执行一系列命令。 所有命令都必须成功，否则，所有命令都必须回退到初始状态。 Azure Redis 缓存支持通过单个操作以[事务](https://redis.io/topics/transactions)形式执行一批命令。 |

## <a name="azure-cache-for-redis-offerings"></a>Azure Redis 缓存的产品/服务

Azure Redis 缓存在下述层中提供：

| 层 | 说明 |
|---|---|
基本 | 单节点缓存。 此层支持多种内存大小 (250 MB - 53 GB)。 此层适用于开发/测试和非关键型工作负荷。 基本层没有服务级别协议 (SLA) |
| 标准 | 在 Microsoft 托管的双节点（主/辅）配置中提供复制的缓存，并提供高可用性 SLA (99.9%) |
| 高级 | 高级层是面向企业的层。 高级层缓存支持更多的功能，吞吐量更高，延迟更低。 高级层中的缓存部署在更强大的硬件上，其性能优于基本层或标准层。 这种优势意味着，在缓存大小相同的情况下，高级层的吞吐量大于标准层。 |

> [!TIP]
> 有关高级缓存大小、吞吐量和带宽的详细信息，请参阅 [Azure Redis 缓存常见问题解答](cache-faq.md#what-azure-cache-for-redis-offering-and-size-should-i-use)。
>

可以在缓存创建以后将其提升到更高的层。 不允许降到更低的层。 有关缩放的分步说明，请参阅[如何缩放 Azure Redis 缓存](cache-how-to-scale.md)和[如何自动执行缩放操作](cache-how-to-scale.md#how-to-automate-a-scaling-operation)。

### <a name="feature-comparison"></a>功能比较

[Azure Redis 缓存定价](https://azure.microsoft.com/pricing/details/cache/)页对每个层进行了详细比较。 下表介绍了每个层支持的部分功能：

| 功能说明 | 高级 | 标准 | 基本 |
| ------------------- | :-----: | :------: | :---: |
| [服务级别协议 (SLA)](https://azure.microsoft.com/support/legal/sla/cache/v1_0/) |✔|✔|-|
| [Redis 数据持久性](cache-how-to-premium-persistence.md) |✔|-|-|
| [Redis 群集](cache-how-to-premium-clustering.md) |✔|-|-|
| [通过防火墙规则确保安全性](cache-configure.md#firewall) |✔|✔|✔|
| 传输中加密 |✔|✔|✔|
| [通过 VNet 增强安全性和隔离性](cache-how-to-premium-vnet.md) |✔|-|-|
| [导入/导出](cache-how-to-import-export-data.md) |✔|-|-|
| [计划的更新](cache-administration.md#schedule-updates) |✔|✔|✔|
| [异地复制](cache-how-to-geo-replication.md) |✔|-|-|
| [重新启动](cache-administration.md#reboot) |✔|✔|✔|

## <a name="next-steps"></a>后续步骤

* [ASP.NET Web 应用快速入门](cache-web-app-howto.md)创建简单的使用 Azure Redis 缓存的 ASP.NET Web 应用。
* [.NET 快速入门](cache-dotnet-how-to-use-azure-redis-cache.md)创建使用 Azure Redis 缓存的 .NET 应用。
* [.NET Core 快速入门](cache-dotnet-core-quickstart.md)创建使用 Azure Redis 缓存的 .NET Core 应用。
* [Node.js 快速入门](cache-nodejs-get-started.md)创建简单的使用 Azure Redis 缓存的 Node.js 应用。
* [Java 快速入门](cache-java-get-started.md)创建使用 Azure Redis 缓存的简单 Java 应用。
* [Python 快速入门](cache-python-get-started.md)创建使用 Azure Redis 缓存的简单 Python 应用。
