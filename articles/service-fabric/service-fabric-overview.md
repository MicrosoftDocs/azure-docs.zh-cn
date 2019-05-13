---
title: Azure 上的 Service Fabric 概述 | Microsoft 文档
description: Service Fabric 概览，其中应用程序由许多微服务组成，以提供扩展性和恢复能力。 Service Fabric 是一个分布式系统平台，可用于构建面向云的可扩展、可靠且易管理的应用程序。
services: service-fabric
documentationcenter: .net
author: athinanthny
manager: chackdan
editor: masnider
ms.assetid: bbcc652a-a790-4bc4-926b-e8cd966587c0
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: overview
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/22/2019
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: ecc63cac56c9aeb3d1dd3875134d53cb481e6a5c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64708197"
---
# <a name="overview-of-azure-service-fabric"></a>Azure Service Fabric 概述
Azure Service Fabric 是一款分布式系统平台，可方便用户轻松打包、部署和管理可缩放的可靠微服务和容器。 Service Fabric 还解决了开发和管理云本机应用程序面临的重大难题。 开发人员和管理员不需解决复杂的基础结构问题，只需专注于实现苛刻的任务关键型工作负荷，即那些可缩放、可靠且易于管理的工作负荷。 Service Fabric 代表了下一代平台，用于生成和管理在容器中运行的企业级单层云规模应用程序。

以下简短视频介绍了 Service Fabric 和微服务：
> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Service-Fabric/player]

## <a name="compliance"></a>合规性
Azure Service Fabric 资源提供程序在所有 Azure 区域中都可用，并符合 Azure 所具有的所有合规性证书；这包括以下内容：SOC、ISO、PCI DSS、HIPAA 和 GDRP。 请查看以下内容以获取合规性证书的完整列表：[合规性产品](https://www.microsoft.com/trustcenter/compliance/complianceofferings)

## <a name="applications-composed-of-microservices"></a>由微服务组成的应用程序 
利用 Service Fabric，可以生成和管理可缩放的可靠应用程序，其中包括在计算机的共享池（称为“群集”）中高密度运行的微服务。 它提供了复杂的轻型运行时，用于生成在容器中运行且可缩放的无状态和有状态分布式微服务。 它还提供了全面的应用程序管理功能，用于预配、部署、监视、升级/修补和删除已部署的应用程序（包括容器化服务）。

Service Fabric 为当今很多 Microsoft 服务提供技术支持，包括 Azure SQL 数据库、Azure Cosmos DB、Cortana、Microsoft Power BI、Microsoft Intune、Azure 事件中心、Azure IoT 中心、Dynamics 365、Skype for Business 以及其他许多核心 Azure 服务。

Service Fabric 专为创建云本机服务而设计，它们可以根据需要刚开始很小，随后成长为包含数百或数千台计算机的大规模服务。

当今的 Internet 规模的服务是使用微服务构建而成的。 微服务的例子包括协议网关、用户配置文件、购物车，清单处理、排队和缓存等。 Service Fabric 是微服务平台，为每个无状态或有状态微服务（或容器）命名独一无二的名称。

Service Fabric 为由这些微服务组成的应用程序提供全面的运行时和生命周期管理功能。 它在 Service Fabric 群集间部署和激活的容器内部托管微服务。 从虚拟机移动到容器可能使密度出现数量级增长。 同样，如果从容器迁移到这些容器中的微服务，也可能会出现另一个密度数量级。 例如，单个 Azure SQL 数据库群集包含数百台计算机，这些计算机运行数以万计的容器，这些容器总共托管数十万个数据库。 每个数据库都是一个 Service Fabric 有状态微服务。 

有关微服务方法的详细信息，请阅读[为什么对生成应用程序使用微服务方法？](service-fabric-overview-microservices.md)

## <a name="container-deployment-and-orchestration"></a>容器部署和业务流程
Service Fabric 是跨计算机群集部署微服务的 Microsoft [容器 Orchestrator](service-fabric-cluster-resource-manager-introduction.md)。 微服务的开发方法有多种，包括使用 [Service Fabric 编程模型](service-fabric-choose-framework.md)、[ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) 或部署[任意选定代码](service-fabric-guest-executables-introduction.md)。 重要的是，可以在同一应用程序中混合进程中的服务和容器中的服务。 如果只需要[部署和管理容器](service-fabric-containers-overview.md)，Service Fabric 是容器 Orchestrator 理想之选。

## <a name="any-os-any-cloud"></a>不限 OS 和云
Service Fabric 可以在所有环境中运行。 可以在许多环境中（例如在 Azure 或本地、Windows Server 或 Linux 中）创建 Service Fabric 群集。 甚至可以在其他公有云上创建群集。 此外，SDK 中的开发环境与生产环境完全相同，都不涉及模拟器。 也就是说，在本地开发群集上运行的内容会部署到其他环境中的群集。

![Service Fabric 平台][Image1]

对于 Windows 开发，Service Fabric .NET SDK 与 Visual Studio 和 Powershell 集成。 请参阅[在 Windows 上准备开发环境](service-fabric-get-started.md)。 对于 Linux 开发，Service Fabric Java SDK 与 Eclipse 集成，Yeoman 用于为 Java、.NET Core 和容器应用程序生成模板。 请参阅[在 Linux 上准备开发环境](service-fabric-get-started-linux.md)

有关创建群集的详细信息，请阅读[在 Windows Server 或 Linux 上创建群集](service-fabric-deploy-anywhere.md)；有关创建 Azure 群集的详细信息，请阅读[通过 Azure 门户创建群集](service-fabric-cluster-creation-via-portal.md)。

## <a name="stateless-and-stateful-microservices-for-service-fabric"></a>无状态和有状态 Service Fabric 微服务
使用 Service Fabric，可以生成包含微服务或容器的应用程序。 无状态微服务（例如网关、Web 代理）不维护除请求及其来自服务的响应之外任何可变状态。 Azure 云服务辅助角色是无状态服务的一个示例。 有状态微服务（例如，用户帐户、数据库、设备、购物车、队列）维护除请求及其响应之外的可变、授权状态。 当今的 Internet 规模应用程序包含无状态和有状态微服务的组合。 

Service Fabric 的关键区别在于，大力注重使用[内置编程模型](service-fabric-choose-framework.md)或容器化有状态服务生成有状态服务。 [应用程序方案](service-fabric-application-scenarios.md)介绍了可使用有状态服务的方案。


## <a name="application-lifecycle-management"></a>应用程序生命周期管理
Service Fabric 支持包含容器的云应用程序具有完整的应用程序生命周期和采用 CI/CD。 生命周期包括从开发到部署、到日常管理和维护，再到最终解除授权。

利用 Service Fabric 应用程序生命周期管理功能，应用程序管理员和 IT 操作人员能够使用低接触的简单工作流配置、部署、修补和监视应用程序。 这些内置的工作流极大地减少了 IT 操作人员保持应用程序持续可用的负担。

大多数应用程序都包含无状态和有状态微服务、容器以及同时部署的其他可执行文件。 通过在应用程序上采用强类型，可以使用 Service Fabric 部署多个应用程序实例。 每个实例将单独进行管理和升级。 重点是，Service Fabric 能够部署容器或任何可执行文件，并确保它们的可靠性。 例如，Service Fabric 可部署 .NET、ASP.NET Core、node.js、Windows 容器、Linux 容器、Java 虚拟机、脚本、Angular 或应用程序的其他任何组成部分。

Service Fabric 与 [Azure Pipelines](https://www.visualstudio.com/team-services/)、[Jenkins](https://jenkins.io/index.html) 和 [Octopus Deploy](https://octopus.com/) 等 CI/CD 工具集成，并可与其他任何常用 CI/CD 工具配合使用。

有关应用程序生命周期管理的详细信息，请参阅[应用程序生命周期](service-fabric-application-lifecycle.md)。 若要详细了解如何部署任意代码，请参阅[部署来宾可执行文件](service-fabric-deploy-existing-app.md)。

## <a name="key-capabilities"></a>关键功能
通过使用 Service Fabric，可以：

* 部署到 Azure 或部署到运行 Windows 或 Linux 的本地数据中心，而无需改变任何代码。 只需编写一次，即可部署到 Service Fabric 群集的任意位置。
* 使用 Service Fabric 编程模型、容器或任意代码，开发由微服务组成的可缩放应用程序。
* 开发高度可靠的无状态和有状态微服务。 使用有状态微服务，简化应用程序设计。 
* 使用新 Reliable Actors 编程模型，创建具有独立式代码和状态的云对象。
* 部署和安排容器，包括 Windows 容器和 Linux 容器。 Service Fabric 是可感知数据的有状态容器 Orchestrator。
* 几秒内就可以高密度部署应用程序，即每台计算机部署数百或数千个应用程序或容器。
* 同时部署各种不同版本的相同应用程序，且可以单独升级每个应用程序。
* 无需停机，即可管理应用程序生命周期，包括重大升级和非重大升级。
* 缩放群集中的节点数。 缩放节点数的同时，应用程序也会随之自动缩放。
* 监视并诊断应用程序的运行状况，并设置策略以执行自动修复。
* 观察资源均衡器如何跨群集安排重新分发应用程序。 Service Fabric 可从故障中恢复，并基于可用资源优化负载分布。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>后续步骤
* 更多相关信息：
  * [为什么通过微服务的方法构建应用程序？](service-fabric-overview-microservices.md)
  * [术语概述](service-fabric-technical-overview.md)
* 设置 [Windows 开发环境](service-fabric-get-started.md)  
* 设置 [Linux 开发环境](service-fabric-get-started-linux.md)
* 了解 [Service Fabric 支持选项](service-fabric-support.md)

[Image1]: media/service-fabric-overview/Service-Fabric-Overview.png
