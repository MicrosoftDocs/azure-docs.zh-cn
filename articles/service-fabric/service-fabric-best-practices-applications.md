---
title: Azure Service Fabric 应用程序设计最佳做法 |Microsoft Docs
description: 有关开发 Service Fabric 应用程序的最佳实践。
services: service-fabric
documentationcenter: .net
author: markfussell
manager: chackdan
editor: ''
ms.assetid: 19ca51e8-69b9-4952-b4b5-4bf04cded217
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/26/2019
ms.author: msfussell
ms.openlocfilehash: 55f043effc7cdb102acea856e89c58f660d0cde5
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237742"
---
# <a name="azure-service-fabric-application-design-best-practices"></a>Azure Service Fabric 应用程序设计最佳做法

本文提供了用于在 Service Fabric 上构建应用程序和服务的最佳实践指南。
 
## <a name="get-familiar-with-service-fabric"></a>熟悉如何使用 Service Fabric
* [因此，你想要了解 Service Fabric？](service-fabric-content-roadmap.md)
* 阅读有关[Service Fabric 应用程序方案](service-fabric-application-scenarios.md)
* 然后了解编程模型选项具有[Service Fabric 编程模型概述](service-fabric-choose-framework.md)



## <a name="application-design-guidance"></a>应用程序设计指南
熟悉[一般体系结构](https://docs.microsoft.com/azure/architecture/reference-architectures/microservices/service-fabric)的 Service Fabric 应用程序并将其[设计注意事项](https://docs.microsoft.com/azure/architecture/reference-architectures/microservices/service-fabric#design-considerations)。

### <a name="choose-an-api-gateway"></a>选择 API 网关
使用 API 网关服务，则可以扩大的后端服务进行通信。使用的最常见的 API 网关服务包括：

- [Azure API 管理](https://docs.microsoft.com/azure/service-fabric/service-fabric-api-management-overview)，这是[与 Service Fabric 集成](https://docs.microsoft.com/azure/service-fabric/service-fabric-tutorial-deploy-api-management)
- [Azure IoT 中心](https://docs.microsoft.com/azure/iot-hub/)或[Azure 事件中心](https://docs.microsoft.com/azure/event-hubs/)使用[ServiceFabricProcessor](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/ServiceFabricProcessor)从事件中心分区读取
- [Træfik 反向代理](https://blogs.msdn.microsoft.com/azureservicefabric/2018/04/05/intelligent-routing-on-service-fabric-with-traefik/)使用[Azure Service Fabric 提供程序](https://docs.traefik.io/configuration/backends/servicefabric/)
- [Azure 应用程序网关](https://docs.microsoft.com/azure/application-gateway/)注意： 这不直接集成使用 Service Fabric 和 Azure API 管理通常是首选的解决方案
- 构建您自己[ASP.NET Core](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-communication-aspnetcore) web 应用程序网关

### <a name="choose-stateless-services"></a>选择无状态服务
我们建议您始终首先构建无状态服务使用[Reliable Services](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-introduction) Azure 数据库、 Azure cosmos Db 或 Azure 存储中存储状态。 无状态外部化是大多数开发人员更熟悉方法，使您还利用查询功能存储区上。  

### <a name="when-to-choose-stateful-services"></a>何时选择有状态服务
如果你有实现低延迟方案并需要保留的数据靠近计算，请考虑有状态服务。 一些示例包括 IoT 数字孪生设备、 游戏状态、 会话状态缓存数据库中的数据和长时间运行的工作流来跟踪对其他服务的调用。

确定数据保留时间范围：

- 缓存的数据-使用缓存中延迟到外部存储是一个问题。 使用有状态服务，如你自己的数据缓存或考虑使用[开源 SoCreate service fabric-分布式-缓存](https://github.com/SoCreate/service-fabric-distributed-cache)。 在此方案中，可能会丢失缓存中的所有数据，并不重要。
- 时间限制数据-您需要保留数据关闭来计算延迟一段时间，但可以承受丢失在该数据*灾难*方案。 例如，在许多 IoT 解决方案中的数据必须能够接近计算，但如果此数据丢失，然后的特定数据点，例如通过过去的几天内，计算温度平均值记录不重要。 此外在此方案中您不通常关心的单个数据点，仅计算平均值定期写入到外部存储的备份。  
- 长期数据的可靠集合可以永久存储数据。 但是，在这种情况下需要[做好灾难恢复](https://docs.microsoft.com/azure/service-fabric/service-fabric-disaster-recovery)包括[配置定期备份策略](https://docs.microsoft.com/azure/service-fabric/service-fabric-backuprestoreservice-configure-periodic-backup)群集。 实际上，您配置会发生什么情况中的灾难方案销毁群集，需要创建新的群集，以及如何部署新应用程序实例和从最新备份中恢复。

节省成本并提高可用性：
- 您可以降低与有状态服务的成本，因为不会产生从远程存储的数据访问和事务成本，并且没有无需使用另一个服务，例如 Azure Redis。
- 使用有状态服务，主要用于存储而不是针对计算既昂贵又不建议这样做。 成本低廉的本地存储与计算中视为有状态服务。
- 通过删除其他服务上的依赖项，可以提高服务可用性。 具有与 HA 群集中的管理状态隔离来自其他服务停机或延迟问题。

## <a name="how-to-properly-work-with-reliable-services"></a>如何正确使用 Reliable Services
可靠的服务，可轻松地创建无状态和有状态服务。 读取[Reliable Services 简介](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-introduction)
- 始终遵循[取消标记](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-lifecycle#stateful-service-primary-swaps)中`RunAsync()`无状态和有状态服务的方法和`ChangeRole()`对于有状态服务的方法。 如果没有，Service Fabric 不知道如果可以关闭你的服务。 例如，不履行取消令牌，就可能导致在更长的时间的应用程序升级时间。
-   打开和关闭[通信侦听器](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-communication)及时并遵守取消标记。
-   永远不会混合使用同步代码和异步代码。 例如，不要使用`.GetAwaiter().GetResult()`异步调用; 中它需要是异步*一直*通过调用堆栈。

## <a name="how-to-properly-work-with-reliable-actors"></a>如何使用 Reliable Actors 正常工作
Reliable Actors，可轻松地创建有状态、 虚拟执行组件。 读取[Reliable Actors 简介](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-introduction)

- 强烈建议考虑使用 pub/sub 消息之间缩放应用程序的执行组件。 例如，[开源 SoCreate pub/sub](https://service-fabric-pub-sub.socreate.it/)或[Azure 服务总线](https://docs.microsoft.com/azure/service-bus/)。
- 使参与者状态作为[尽可能细粒度](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-state-management#best-practices)。
- 管理[执行组件的生命周期](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-state-management#best-practices)。 如果您不打算再次使用它们，请删除执行组件。 使用时更是如此[VolatileState 提供程序](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-state-management#state-persistence-and-replication)所有状态存储在内存中。
- 由于其[基于轮次的并发](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-introduction#concurrency)actors 最好用作独立的对象。 不要创建多参与者、 同步方法调用 （每个最有可能成为单独的网络调用） 的关系图或有循环的执行组件请求;这些将明显影响性能和规模。
- 不要混合同步代码和异步代码;它需要一直是异步的以防止出现性能问题。
- 不在执行组件进行长时间运行的调用，它将阻止其他对相同的参与者，由于基于轮次的并发调用。
- 如果与使用其他服务通信[Service Fabric 远程处理](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-communication-remoting)，并且您创建`ServiceProxyFactory`，然后创建在工厂[执行组件服务](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-using)级别和*不*执行组件级别。


## <a name="application-diagnostics"></a>应用程序诊断
- 在添加彻底[应用程序日志记录](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-generation-app)服务调用。 它有助于诊断方案，其中服务相互调用。 例如，当一个-> B-> C-> 的 D 调用任何位置; 可能失败如果没有足够的日志记录，，很难诊断。 如果由于调用卷太多日志记录服务至少请务必记录错误和警告。

## <a name="iot-and-messaging-applications"></a>IoT 和消息传递应用程序
从消息中读取时[Azure IoT 中心](https://docs.microsoft.com/azure/iot-hub/)或[Azure 事件中心](https://docs.microsoft.com/azure/event-hubs/)使用[ServiceFabricProcessor](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/ServiceFabricProcessor)与 Service Fabric Reliable Services 维护的集成从事件中心读取的状态进行分区，并将新消息推送到你的服务通过`IEventProcessor::ProcessEventsAsync()`方法。


## <a name="design-guidance-on-azure"></a>在 Azure 上设计指南
* 请访问[Azure 体系结构中心](https://docs.microsoft.com/azure/architecture/microservices/)有关设计指南[Azure 上构建微服务](https://docs.microsoft.com/azure/architecture/microservices/)

* 请访问[开始使用 Azure 游戏](https://docs.microsoft.com/gaming/azure/)有关设计指南[游戏服务中使用 Service Fabric](https://docs.microsoft.com/gaming/azure/reference-architectures/multiplayer-synchronous-sf)
