---
title: Azure 容器实例-方面的常见问题
description: 有关与 Azure 容器实例服务相关的常见问题的答案
services: container-instances
author: dkkapur
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 4/25/2019
ms.author: dekapur
ms.openlocfilehash: 5657ac9f10c42c2201641c9af447b7ad2e6a3507
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079010"
---
# <a name="frequently-asked-questions-about-azure-container-instances"></a>有关 Azure 容器实例常见问题

本文介绍 Azure 容器实例有关的常见问题。

## <a name="deployment"></a>部署

### <a name="how-large-can-my-container-image-be"></a>我的容器映像可以有多大？

Azure 容器实例上的可部署容器映像的最大大小为 15 GB。 你可以部署较大的图像根据在时间点的确切可用性部署，但这不保证。

容器映像的大小会影响其所需的时间部署，通常你想要保留你的容器映像越小越好。

### <a name="how-can-i-speed-up-the-deployment-of-my-container"></a>如何提高我的容器的部署速度

由于部署时间的主要决定因素之一是图像大小，查找有关如何缩减大小。 删除不需要或减小图像中的层 （通过选择较浅的基础 OS 映像） 的图层。 例如，如果您在运行 Linux 容器，请考虑使用 Alpine 为您的基础映像而不是完整的 Ubuntu Server。 同样，对于 Windows 容器，使用 Nano Server 基本映像在可能的情况。 

此外应检查在 Azure 容器映像中，通过提供预先缓存映像列表[列出缓存映像](/rest/api/container-instances/listcachedimages)API。 您可能能够切出的映像层为预先缓存映像之一。 

查看更多[详细指南](container-instances-troubleshooting.md#container-takes-a-long-time-to-start)上减小容器启动时间。

### <a name="what-windows-base-os-images-are-supported"></a>支持哪些 Windows 基本操作系统映像映像？

#### <a name="windows-server-2016-base-images"></a>Windows Server 2016 基础映像

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `10.0.14393.x`， `sac2016`
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2016`，  `10.0.14393.x`

> [!NOTE]
> 不支持基于 1709年或 1803年的半年频道发布的 Windows 映像。

#### <a name="windows-server-2019-and-client-base-images-preview"></a>Windows Server 2019 和客户端基础映像 （预览版）

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `1809`， `10.0.17763.x`
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2019`， `1809`， `10.0.17763.x`
* [Windows](https://hub.docker.com/_/microsoft-windows): `1809`， `10.0.17763.x` 

### <a name="what-net-or-net-core-image-layer-should-i-use-in-my-container"></a>在我的容器中应使用何种.NET 或.NET Core 映像层？ 

使用满足你的要求的最小图像。 对于 Linux，可以使用*运行时 alpine*自.NET Core 2.1 发布以来一直支持.NET Core 映像。 对于 Windows，如果您使用的完整的.NET Framework，然后需要使用 Windows Server Core 映像 (仅限运行时的图像，如*4.7.2-windowsservercore-ltsc2016*)。 仅限运行时映像更小，但不是支持需要.NET SDK 的工作负荷。

## <a name="availability-and-quotas"></a>可用性和配额

### <a name="how-many-cores-and-memory-should-i-allocate-for-my-containers-or-the-container-group"></a>多少个核心和内存应该我分配的我的容器或容器组？

这确实取决于你的工作负荷。 从小规模着手并测试性能，以查看你的容器执行的操作。 [监视 CPU 和内存资源使用情况](container-instances-monitor.md)，以及如何将核心或基于类型的容器中部署的进程的内存。 

请确保也检查[资源可用性](container-instances-region-availability.md#availability---general)区域中部署 CPU 内核数和每个容器组的可用内存的上限。 

### <a name="what-underlying-infrastructure-does-aci-run-on"></a>ACI 上运行哪些底层基础结构？

Azure 容器实例的目标是为无服务器的容器按需服务，因此我们希望您能致力于制定你的容器，而不用担心基础结构 ！ 对于那些想或想要进行比较的性能，在 ACI 在上运行的各种 sku 时，Azure Vm 集主要来自 F 和 D 系列。 我们预计这随着我们不断开发和优化该服务在将来的更改。 

### <a name="i-want-to-deploy-thousand-of-cores-on-aci---can-i-get-my-quota-increased"></a>我想要部署千条 ACI 上的内核-获取我增加的配额？
 
是 （有时）。 请参阅[配额和限制](container-instances-quotas.md)一文，了解当前配额和限制可以通过请求增加。

### <a name="can-i-deploy-with-more-than-4-cores-and-16-gb-of-ram"></a>可以使用超过 4 个核心和 16 GB RAM 进行部署

还不可以。 目前，这些是容器组的最大值。 与特定的要求或请求联系 Azure 支持部门。 

### <a name="when-will-aci-be-in-a-specific-region"></a>ACI 何时在特定区域？

发布当前区域可用性[这里](container-instances-region-availability.md#availability---general)，并保持最新信息，请通过[列表功能](/rest/api/container-instances/listcapabilities)API。 如果有某个特定区域的需求，请联系 Azure 支持。

## <a name="features-and-scenarios"></a>功能和方案

### <a name="how-do-i-scale-a-container-group"></a>如何缩放容器组？

目前，缩放不可用的容器或容器组。 如果需要运行多个实例，使用我们的 API 来实现自动化和创建更多容器组创建到服务的请求。 

### <a name="what-features-are-available-to-instances-running-in-a-custom-vnet"></a>可用于自定义 VNet 中运行的实例有哪些功能？

你可以部署所选的 Azure 虚拟网络中的容器组和委派到容器组，以在 Azure 资源之间路由 VNet 中的流量的专用 Ip。 容器组部署到虚拟网络当前处于预览状态，并在正式版 (GA) 之前此功能的某些方面可能会更改。 请参阅[预览版限制](container-instances-vnet.md#preview-limitations)更新信息。

## <a name="pricing"></a>定价

### <a name="when-does-the-meter-start-running"></a>测定仪何时开始运行？

我们开始拉取第一个容器的映像 （适用于新的部署） 或容器组重新启动 （如果已部署），直到停止的容器组的时间通过计算容器组持续时间。 详细信息，请参阅[容器实例定价](https://azure.microsoft.com/pricing/details/container-instances/)。

### <a name="do-i-stop-being-charged-when-my-containers-are-stopped"></a>停止容器停止时，你被收费？

计量器停止运行后停止整个容器组。 只要运行容器组中的容器，我们将资源保存在以防你想要再次启动容器。 

## <a name="next-steps"></a>后续步骤

* [了解详细信息](container-instances-overview.md)了解 Azure 容器实例。
* [排查常见问题](container-instances-troubleshooting.md)Azure 容器实例中。