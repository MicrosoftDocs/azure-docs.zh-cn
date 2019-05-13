---
title: Azure Service Fabric 应用程序和群集最佳做法 | Microsoft Docs
description: 用于管理 Service Fabric 群集和应用程序的最佳做法。
services: service-fabric
documentationcenter: .net
author: peterpogorski
manager: chackdan
editor: ''
ms.assetid: 19ca51e8-69b9-4952-b4b5-4bf04cded217
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/24/2019
ms.author: pepogors
ms.openlocfilehash: 051d6b1129724ce4e8a67bde4e56ebe61cd832f3
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231379"
---
# <a name="azure-service-fabric-application-and-cluster-best-practices"></a>Azure Service Fabric 应用程序和群集最佳做法

若要成功管理 Azure Service Fabric 应用程序和群集，有一些我们强烈建议进行优化的生产环境; 可靠性所执行的操作请执行本文档中定义的操作，然后选择其中一个我们[Azure 示例 Service Fabric 群集模板](https://github.com/Azure-Samples/service-fabric-cluster-templates)开始设计你的生产解决方案或修改现有模板以合并这些做法。

## <a name="security"></a>安全 

* [安全性最佳做法](service-fabric-best-practices-security.md)

## <a name="networking"></a>网络

* [网络最佳做法](service-fabric-best-practices-networking.md)

## <a name="compute-planning-and-scaling"></a>计算规划和缩放

* [计算缩放最佳做法](service-fabric-best-practices-capacity-scaling.md)
* [计算容量规划](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-capacity)

## <a name="infrastructure-as-code"></a>基础结构即代码

* [用于实现基础结构即代码的最佳做法](service-fabric-best-practices-infrastructure-as-code.md)

## <a name="monitoring-and-diagnostics"></a>监视和诊断

* [群集监视和诊断最佳做法](service-fabric-best-practices-monitoring.md)

## <a name="application-design"></a>应用程序设计

* [应用程序设计最佳实践](service-fabric-best-practices-applications.md)

## <a name="checklist"></a>清单

完成上面的所有部分后，请确保你已集成了生产就绪情况核对清单中的所有最佳做法：
* [Azure Service Fabric 生产就绪情况核对清单](https://docs.microsoft.com/azure/service-fabric/service-fabric-production-readiness-checklist)

## <a name="next-steps"></a>后续步骤

* 在运行 Windows Server 的 VM 或计算机上创建群集：[创建适用于 Windows Server 的 Service Fabric 群集](service-fabric-cluster-creation-for-windows-server.md)
* 在运行 Linux 的 VM 或计算机上创建群集：[创建 Linux 群集](service-fabric-cluster-creation-via-portal.md)
* 故障排除：[Service Fabric 故障排除指南](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides)