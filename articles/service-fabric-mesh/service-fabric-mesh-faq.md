---
title: Azure Service Fabric 网格常见问题 | Microsoft Docs
description: 了解 Azure Service Fabric 网格的常见问题和解答。
services: service-fabric-mesh
keywords: ''
author: chackdan
ms.author: chackdan
ms.date: 4/23/2019
ms.topic: troubleshooting
ms.service: service-fabric-mesh
manager: jeanpaul.connock
ms.openlocfilehash: 950f9ac89b9d3224db29b32fe2d1e403ccc98116
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143279"
---
# <a name="commonly-asked-service-fabric-mesh-questions"></a>有关 Service Fabric 网格的常见问题

Azure Service Fabric 网格是一个完全托管的服务，由此开发者可部署微服务应用程序，而无需管理虚拟机、存储或网络。 本文提供常见问题的解答。

## <a name="how-do-i-report-an-issue-or-ask-a-question"></a>如何报告问题或提问？

在 [service-fabric-mesh-preview GitHub 存储库](https://aka.ms/sfmeshissues)中询问问题、从 Microsoft 工程师处获得解答、报告问题。

## <a name="quota-and-cost"></a>配额和成本

### <a name="what-is-the-cost-of-participating-in-the-preview"></a>参与预览版的成本是多少？

目前可免费用于将应用程序或容器部署到网格预览。 请继续观看 5 月中的更新要启用计费。 不过，我们建议您删除的资源将部署并不会使其保持运行，除非你积极地测试它们。

### <a name="is-there-a-quota-limit-of-the-number-of-cores-and-ram"></a>内核和 RAM 的数量是否有配额限制？

可以。 每个订阅的配额是：

- 应用程序数量：5
- 每个应用程序的核心数：12
- 每个应用程序的总内存：48 GB
- 网络和入口终结点：5
- 可以附加的 Azure 卷：10
- 服务副本数：3
- 可以部署的最大容器限制为 4 个内核，16GB RAM。
- 可以将部分核心以 0.5 个核心的增量分配给容器（最多 6 个核心）。

### <a name="how-long-can-i-leave-my-application-deployed"></a>可以将已部署应用程序保留多长时间？

当前，我们将应用程序的生存期限制为两天。 这是为了充分利用分配给预览版的空闲核心。 因此，仅允许你将给定的部署持续运行 48 小时，经过该时间量后，就会关闭该部署。

如果看到此情况发生，可以通过在 Azure CLI 中运行 `az mesh app show` 命令来验证系统是否已将其关闭。 检查是否返回了 `"status": "Failed", "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue."` 

例如： 

```cli
~$ az mesh app show --resource-group myResourceGroup --name helloWorldApp
{
  "debugParams": null,
  "description": "Service Fabric Mesh HelloWorld Application!",
  "diagnostics": null,
  "healthState": "Ok",
  "id": "/subscriptions/1134234-b756-4979-84re-09d671c0c345/resourcegroups/myResourceGroup/providers/Microsoft.ServiceFabricMesh/applications/helloWorldApp",
  "location": "eastus",
  "name": "helloWorldApp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "serviceNames": [
    "helloWorldService"
  ],
  "services": null,
  "status": "Failed",
  "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue.",
  "tags": {},
  "type": "Microsoft.ServiceFabricMesh/applications",
  "unhealthyEvaluation": null
}
```

若要删除资源组，请使用 `az group delete <nameOfResourceGroup>` 命令。

## <a name="deployments"></a>部署

### <a name="what-container-images-are-supported"></a>支持哪些容器映像？

如果是在 Windows Fall Creators Update（版本 1709）计算机上进行开发，则只能使用 Windows 1709 版本的 Docker 映像。

如果是在 Windows 10 April 2018 Update（版本 1803）计算机上进行开发，则可以使用 Windows 1709 或 Windows 1803 版本的 Docker 映像。

以下容器 OS 映像可以用来部署服务：

- Windows - windowsservercore 和 nanoserver
    - Windows Server 1709
    - Windows Server 1803
    - Windows Server 1809
    - Windows Server 2019 LTSC
- Linux
    - 无已知限制

> [!NOTE]
> Visual Studio 工具网格尚不支持部署到 Windows Server 2019 和 1809年容器。

### <a name="what-types-of-applications-can-i-deploy"></a>可以部署哪些类型的应用程序？ 

你可以部署为应用程序资源 （请参阅上面有关配额的详细信息） 放置在适合的限制范围内的容器中运行的任何内容。 如果我们检测到使用网格用于运行非法工作负荷或滥用系统 （即挖掘），然后我们保留终止您的部署和阻止列表从服务上运行你的订阅的权限。 请联系我们如果你有正在运行的特定工作负荷上的任何问题。 

## <a name="developer-experience-issues"></a>开发人员体验问题

### <a name="dns-resolution-from-a-container-doesnt-work"></a>来自容器的 DNS 解析不起作用

在某些情况下，从容器到 Service Fabric DNS 服务的传出 DNS 查询可能会失败。 正在调查此问题。 缓解措施：

- 使用 Windows Fall Creators 更新（版本 1709）或更高版本作为基容器映像。
- 如果单独的服务名称无效，请尝试完全限定的名称：ServiceName.ApplicationName。
- 在服务的 Docker 文件中添加 `EXPOSE <port>`，其中的端口是在其上公开服务的端口。 例如：

```Dockerfile
EXPOSE 80
```

### <a name="dns-does-not-work-the-same-as-it-does-for-service-fabric-development-clusters-and-in-mesh"></a>DNS 的工作方式不同于 Service Fabric 开发群集的以及网格中的工作方式

可能需要在本地开发群集和 Azure 网格中以不同的方式引用服务。

在本地开发群集中，请使用 `{serviceName}.{applicationName}`。 在 Azure Service Fabric 网格中，请使用 `{servicename}`。 

Azure 网格目前不支持跨应用程序进行 DNS 解析。

有关在 Windows 10 上运行 Service Fabric 开发群集的其他已知 DNS 问题，请参阅：[调试 Windows 容器](/azure/service-fabric/service-fabric-how-to-debug-windows-containers)和[已知的 DNS 问题](https://docs.microsoft.com/azure/service-fabric/service-fabric-dnsservice#known-issues)。

### <a name="networking"></a>网络

在本地计算机上运行应用时，ServiceFabric 网络 NAT 可能会消失。 若要诊断是否发生了这种情况，请从命令提示符处运行以下命令：

`docker network ls`，然后注意 `servicefabric_nat` 是否已列出。  如果没有列出，则请运行以下命令：`docker network create -d=nat --subnet 10.128.0.0/24 --gateway 10.128.0.1 servicefabric_nat`

这样就可以解决问题，即使应用已部署到本地且处于不正常状态。

### <a name="issues-running-multiple-apps"></a>运行多个应用的问题

可能会遇到需要跨所有应用程序解决的 CPU 可用性和限制问题。 缓解措施：
- 创建一个五节点群集。
- 在已部署应用中，减少服务的 CPU 使用量。 例如，在服务的 service.yaml 文件中，将 `cpu: 1.0` 更改为 `cpu: 0.5`

不能将多个应用程序部署到一个单节点群集。 缓解措施：
- 将多个应用部署到一个本地群集时，请使用一个五节点群集。
- 删除当前未测试的应用。

### <a name="vs-tooling-has-limited-support-for-windows-containers"></a>VS 工具的支持有限的 Windows 容器

Visual Studio 工具仅支持与 Windows Server 1709 和 1803年的基础 OS 版本现在部署 Windows 容器。 

## <a name="feature-gaps-and-other-known-issues"></a>功能差距和其他已知问题

### <a name="after-deploying-my-application-the-network-resource-associated-with-it-does-not-have-an-ip-address"></a>部署应用程序后，与其相关联的网络资源没有 IP 地址

有一个已知的问题，即 IP 地址不会立即变得可用。 几分钟后检查网络资源的状态，查看相关的 IP 地址。

### <a name="my-application-fails-to-access-the-right-networkvolume-resource"></a>应用程序无法访问正确的网络/卷资源

在应用程序模型中，请使用网络和卷的完整资源 ID 来访问相关资源。 下面是快速入门示例中的一个示例：

```json
"networkRefs": [
    {
    "name":  "[resourceId('Microsoft.ServiceFabric/networks', 'SbzVotingNetwork')]" 
    }
]
```

### <a name="when-i-scale-out-all-of-my-containers-are-affected-including-running-ones"></a>向外扩展时，所有的容器都受到影响，包括正在运行的容器

这是一个 Bug，目前我们正在实现一个修复程序。

## <a name="next-steps"></a>后续步骤

若要详细了解 Service Fabric 网格，请阅读[概述](service-fabric-mesh-overview.md)。
