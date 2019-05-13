---
title: 配置 Azure Service Fabric 独立群集 | Microsoft Docs
description: 了解如何配置独立的或本地 Azure Service Fabric 群集。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: chackdan
editor: ''
ms.assetid: 0c5ec720-8f70-40bd-9f86-cd07b84a219d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/12/2018
ms.author: dekapur
ms.openlocfilehash: ae7fbef864634e47866de13384871a98b8ce4675
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209704"
---
# <a name="configuration-settings-for-a-standalone-windows-cluster"></a>独立 Windows 群集的配置设置
本文介绍可使用 ClusterConfig.json 文件设置的独立 Azure Service Fabric 群集的配置设置。 需要使用该文件指定有关群集节点、安全配置以及有关容错域和升级域的网络拓扑信息。  更改或添加配置设置后，可以[创建一个独立的群集](service-fabric-cluster-creation-for-windows-server.md)，也可以[升级独立群集的配置](service-fabric-cluster-config-upgrade-windows-server.md)。

[下载独立的 Service Fabric 包](service-fabric-cluster-creation-for-windows-server.md#downloadpackage)时还会附带 ClusterConfig.json 示例。 名称中包含“DevCluster”的示例可使用逻辑节点创建所有三个节点都在同一台计算机上的群集。 在这些节点中，必须至少将一个节点标记为主节点。 此群集类型可用于开发或测试环境。 不支持将它用作生产群集。 名称中包含“MultiMachine”的示例可帮助创建生产等级群集，其中的每个节点位于不同的计算机上。 这些群集的主节点数取决于群集的[可靠性级别](#reliability)。 在版本 5.7 API 版本 05-2017 中，我们删除了可靠性级别属性。 取而代之的是，我们的代码将计算群集的最优可靠性级别。 请勿尝试在版本 5.7 及以上版本中设置此属性的值。

* ClusterConfig.Unsecure.DevCluster.json 和 ClusterConfig.Unsecure.MultiMachine.json 分别说明如何创建不安全的测试群集和生产群集。

* ClusterConfig.Windows.DevCluster.json 和 ClusterConfig.Windows.MultiMachine.json 说明如何创建使用 [Windows 安全性](service-fabric-windows-cluster-windows-security.md)保护的测试群集和生产群集。

* ClusterConfig.X509.DevCluster.json 和 ClusterConfig.X509.MultiMachine.json 说明如何创建使用[基于 X509 证书的安全性](service-fabric-windows-cluster-x509-security.md)保护的测试群集和生产群集。

现在，让我们查看 ClusterConfig.json 文件的各个部分。

## <a name="general-cluster-configurations"></a>常规群集配置
常规群集配置包括特定于群集的配置，如以下 JSON 代码片段中所示：

```json
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "01-2017",
```

可为 Service Fabric 群集指定任何友好名称，只需将该名称分配到 name 变量即可。 clusterConfigurationVersion 是群集的版本号。 每次升级 Service Fabric 群集时，都应该递增该编号。 请将 apiVersion 保留为默认值。

## <a name="nodes-on-the-cluster"></a>群集上的节点
可以使用 nodes 节配置 Service Fabric 群集上的节点，如以下代码片段中所示：
```json
"nodes": [{
    "nodeName": "vm0",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD0"
}, {
    "nodeName": "vm1",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType1",
    "faultDomain": "fd:/dc1/r1",
    "upgradeDomain": "UD1"
}, {
    "nodeName": "vm2",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType2",
    "faultDomain": "fd:/dc1/r2",
    "upgradeDomain": "UD2"
}],
```

一个 Service Fabric 群集必须至少包含三个节点。 可以根据设置向此节添加更多节点。 下表说明了每个节点的配置设置：

| **节点配置** | **说明** |
| --- | --- |
| nodeName |可以为节点提供任何友好名称。 |
| iPAddress |打开命令窗口并键入 `ipconfig`，找出节点的 IP 地址。 记下 IPV4 地址，并将其分配给 iPAddress 变量。 |
| nodeTypeRef |可以为每个节点分配不同的节点类型。 [节点类型](#node-types)在以下节中定义。 |
| faultDomain |容错域可让群集管理员定义可能因共享的物理依赖项而同时发生故障的物理节点。 |
| upgradeDomain |升级域描述几乎在相同时间关闭以进行 Service Fabric 升级的节点集。 可以选择将哪些节点分配到哪些升级域，因为这不受任何物理要求的限制。 |

## <a name="cluster-properties"></a>群集属性
ClusterConfig.json 中的属性部分用于配置群集，如下所示：

### <a name="reliability"></a>可靠性
reliabilityLevel 的概念定义可在群集的主节点上运行的 Service Fabric 系统服务副本或实例数。 它会确定这些服务以及群集的可靠性。 在群集创建和升级过程中，由系统计算该值。

### <a name="diagnostics"></a>诊断
在 diagnosticsStore 节中可以配置参数，以便能够诊断和排查节点或群集故障，如以下代码片段中所示： 

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "FileShare",
    "IsEncrypted": "false",
    "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
}
```

metadata 用于描述群集诊断，可以根据具体的情况进行设置。 这些变量有助于收集 ETW 跟踪日志、故障转储和性能计数器。 有关 ETW 跟踪日志的详细信息，请阅读 [Tracelog](https://msdn.microsoft.com/library/windows/hardware/ff552994.aspx) 和 [ETW 跟踪](https://msdn.microsoft.com/library/ms751538.aspx)。 可将所有日志（包含[故障转储](https://blogs.technet.microsoft.com/askperf/2008/01/08/understanding-crash-dump-files/)和[性能计数器](https://msdn.microsoft.com/library/windows/desktop/aa373083.aspx)）定向到计算机上的 connectionString 文件夹。 还可以使用 AzureStorage 来存储诊断信息。 请参阅以下示例代码片段：

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "AzureStorage",
    "IsEncrypted": "false",
    "connectionstring": "xstore:DefaultEndpointsProtocol=https;AccountName=[AzureAccountName];AccountKey=[AzureAccountKey]"
}
```

### <a name="security"></a>安全
对于安全的 Service Fabric 独立群集，必须使用 security 节。 以下代码片段显示了该部分的一部分内容：

```json
"security": {
    "metadata": "This cluster is secured using X509 certificates.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    . . .
}
```

metadata 用于描述安全群集，可根据具体的情况进行设置。 ClusterCredentialType 和 ServerCredentialType 确定群集与节点将要实现的安全类型。 这些属性可设置为*X509*基于证书的安全性或*Windows*基于 Active Directory 的安全性。 security 节的余下设置基于安全类型。 若要了解如何填充 security 节的余下设置，请参阅[独立群集中基于证书的安全性](service-fabric-windows-cluster-x509-security.md)，或[独立群集中的 Windows 安全性](service-fabric-windows-cluster-windows-security.md)。

### <a name="node-types"></a>节点类型
nodeTypes 节描述群集中的节点类型。 一个群集必须指定至少一个节点类型，如以下代码片段所示： 

```json
"nodeTypes": [{
    "name": "NodeType0",
    "clientConnectionEndpointPort": "19000",
    "clusterConnectionEndpointPort": "19001",
    "leaseDriverEndpointPort": "19002"
    "serviceConnectionEndpointPort": "19003",
    "httpGatewayEndpointPort": "19080",
    "reverseProxyEndpointPort": "19081",
    "applicationPorts": {
        "startPort": "20575",
        "endPort": "20605"
    },
    "ephemeralPorts": {
        "startPort": "20606",
        "endPort": "20861"
    },
    "isPrimary": true
}]
```

name 是此特定节点类型的友好名称。 要创建这种类型的节点，请[如前所述](#nodes-on-the-cluster)，将该节点的友好名称分配到其 nodeTypeRef 变量。 对于每个节点类型，请定义要使用的连接终结点。 可为这些连接终结点选择任意端口号，只要不与此群集中的任何其他终结点冲突即可。 在多节点群集中，根据 [reliabilityLevel](#reliability)，将有一个或多个主节点（即，isPrimary 设置为 *true*）。 若要详细了解主节点类型和非主节点类型，请参阅 [Service Fabric 群集容量规划注意事项](service-fabric-cluster-capacity.md)，了解有关 nodeTypes 和 reliabilityLevel 的信息。 

#### <a name="endpoints-used-to-configure-the-node-types"></a>用于配置节点类型的终结点
* clientConnectionEndpointPort 是使用客户端 API 时，客户端用来连接群集的端口。 
* clusterConnectionEndpointPort 是节点相互通信时使用的端口。
* leaseDriverEndpointPort 是群集租用驱动程序用来判断节点是否仍处于活动状态的端口。 
* serviceConnectionEndpointPort 是节点上部署的应用程序和服务用来与该特定节点上的 Service Fabric 客户端通信的端口。
* httpGatewayEndpointPort 是 Service Fabric Explorer 用来连接群集的端口。
* ephemeralPorts 重写 [OS 使用的动态端口](https://support.microsoft.com/kb/929851)。 Service Fabric 使用其中的一部分端口作为应用程序端口，剩余的端口供 OS 使用。 它还会将此范围映射到 OS 中的现有范围，因此，无论何时，都可以使用示例 JSON 文件中指定的范围。 确保起始端口与结束端口至少相差 255。 如果此差过小，可能会遇到冲突，因为此范围与 OS 共享。 若要查看配置的动态端口范围，请运行 `netsh int ipv4 show dynamicport tcp`。
* applicationPorts 是 Service Fabric 应用程序使用的端口。 应用程序端口范围的大小应足以满足应用程序的终结点要求。 此范围在计算机上的动态端口范围中应是独占的，即按配置中设置的 ephemeralPorts 范围。 每当需要新端口时，Service Fabric 将使用这些端口，并负责为这些端口打开防火墙。 
* reverseProxyEndpointPort 是可选的反向代理终结点。 有关详细信息，请参阅 [Service Fabric 反向代理](service-fabric-reverseproxy.md)。 

### <a name="log-settings"></a>日志设置
在 fabricSettings 节中，可以设置 Service Fabric 数据和日志的根目录。 只能在初次创建群集时自定义这些目录。 请参阅此节的以下示例代码片段：

```json
"fabricSettings": [{
    "name": "Setup",
    "parameters": [{
        "name": "FabricDataRoot",
        "value": "C:\\ProgramData\\SF"
    }, {
        "name": "FabricLogRoot",
        "value": "C:\\ProgramData\\SF\\Log"
}]
```

建议使用非 OS 驱动器作为 FabricDataRoot 和 FabricLogRoot。 此类驱动器提供更高的可靠性，可防止 OS 停止响应的情况。 如果只自定义数据根目录，则会将日志根目录放置在比数据根目录低一级的位置。

### <a name="stateful-reliable-services-settings"></a>有状态 Reliable Services 设置
在 KtlLogger 节中，可以设置 Reliable Services 的全局配置设置。 有关这些设置的详细信息，请阅读[配置有状态 Reliable Services](service-fabric-reliable-services-configuration.md)。 以下示例演示如何更改创建的共享事务日志，以备份有状态服务的任何可靠集合：

```json
"fabricSettings": [{
    "name": "KtlLogger",
    "parameters": [{
        "name": "SharedLogSizeInMB",
        "value": "4096"
    }]
}]
```

### <a name="add-on-features"></a>附加功能
若要配置附加功能，请将 apiVersion 配置为 04-2017 或更高，并按如下所示配置 addonFeatures：

```json
"apiVersion": "04-2017",
"properties": {
    "addOnFeatures": [
        "DnsService",
        "RepairManager"
    ]
}
```

### <a name="container-support"></a>容器支持
若要为 Windows Server 容器和独立群集的 Hyper-V 容器启用容器支持，必须启用 DnsService 附加功能。

## <a name="next-steps"></a>后续步骤
根据独立群集设置配置一个完整的 ClusterConfig.json 文件后，即可可部署群集。 请遵循[创建独立 Service Fabric 群集](service-fabric-cluster-creation-for-windows-server.md)中所述的步骤。 

如果已部署了独立群集，还可以[升级独立群集的配置](service-fabric-cluster-config-upgrade-windows-server.md)。 

了解如何[使用 Service Fabric Explorer 可视化群集](service-fabric-visualizing-your-cluster.md)。

