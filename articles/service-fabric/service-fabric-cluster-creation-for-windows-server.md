---
title: 创建独立的 Azure Service Fabric 群集 | Microsoft Docs
description: 在运行 Windows Server 的任何本地或任意云计算机（物理或虚拟）上创建 Azure Service Fabric 群集。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: chackdan
editor: ''
ms.assetid: 31349169-de19-4be6-8742-ca20ac41eb9e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/21/2019
ms.author: dekapur
ms.openlocfilehash: ed775bfca2db02b9bfddebb85bbd3f1f668cf3e0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142696"
---
# <a name="create-a-standalone-cluster-running-on-windows-server"></a>创建在 Windows Server 上运行的独立群集
可以使用 Azure Service Fabric 在运行 Windows Server 的任何虚拟机或计算机上创建 Service Fabric 群集。 这意味着可以在包含一组相互连接的 Windows Server 计算机的任何环境（无论是本地环境还是任何云提供商所提供的环境）中部署和运行 Service Fabric 应用程序。 Service Fabric 提供了一个安装程序包，用于创建名为“Windows Server 独立包”的 Service Fabric 群集。

本文将逐步指导完成创建 Service Fabric 独立群集的过程。

> [!NOTE]
> 此独立 Windows Server 包从商业角度而言可在无需付费，并且可能用于生产部署。 此包可能包含“预览”形式的新 Service Fabric 功能。 向下滚动到 [此包中包括的预览功能](#previewfeatures_anchor) 预览功能列表部分。 现在，可以[下载 EULA 的副本](https://go.microsoft.com/fwlink/?LinkID=733084)。
> 
> 

<a id="getsupport"></a>

## <a name="get-support-for-the-service-fabric-for-windows-server-package"></a>获取用于 Windows Server 的 Service Fabric 包的支持
* 在 [Azure Service Fabric 论坛](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=AzureServiceFabric?)中询问社区关于 Windows Server 的 Service Fabric 独立包的信息。
* 开具 [Service Fabric 专业支持](https://support.microsoft.com/oas/default.aspx?prid=16146)票证。  在[此处](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0)了解有关 Microsoft 专业支持的详细信息。
* 还可以获取作为 [Microsoft 顶级支持](https://support.microsoft.com/en-us/premier)的一部分的对此包的支持。
* 有关详细信息，请参阅 [Azure Service Fabric 支持选项](https://docs.microsoft.com/azure/service-fabric/service-fabric-support)。
* 若要出于支持目的收集日志，请运行 [Service Fabric 独立日志收集器](service-fabric-cluster-standalone-package-contents.md)。

<a id="downloadpackage"></a>

## <a name="download-the-service-fabric-for-windows-server-package"></a>下载用于 Windows Server 的 Service Fabric 包
若要创建群集，请使用可在此处找到的用于 Windows Server 的 Service Fabric 包（Windows Server 2012 R2 和更高版本）： <br>
[下载链接 - Service Fabric 独立包 - Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690)

在 [此处](service-fabric-cluster-standalone-package-contents.md)查找有关包内容的详细信息。

在创建群集时自动下载 Service Fabric 运行时包。 如果通过未连接到 Internet 的计算机部署，请从此处下载带外的运行时包： <br>
[下载链接 - Service Fabric 运行时 - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)

在此处查找独立群集配置示例： <br>
[独特群集配置示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

<a id="createcluster"></a>

## <a name="create-the-cluster"></a>创建群集
可以通过安装包安装多个示例性的群集配置文件。 *ClusterConfig.Unsecure.DevCluster.json* 是最简单的群集配置：在单台计算机上运行一个不受保护的三节点群集。  其他配置文件描述的是受 X.509 证书或 Windows 安全措施保护的单机或多机群集。  就本教程来说，无需修改任何默认的配置设置，但仍请查看配置文件，熟悉一下这些设置。  **nodes** 节描述了群集中的三个节点：名称、IP 地址、[节点类型、容错域和升级域](service-fabric-cluster-manifest.md#nodes-on-the-cluster)。  **properties** 节定义群集的[安全性、可靠级别、诊断集合和节点类型](service-fabric-cluster-manifest.md#cluster-properties)。

本文中创建的群集是不安全的。  任何人都可以进行匿名连接并执行管理操作，因此生产型群集应始终使用 X.509 证书或 Windows 安全措施来确保安全性。  安全性只能在群集创建时配置，不能在创建群集以后启用安全性。 更新配置文件以启用[证书安全性](service-fabric-windows-cluster-x509-security.md)或 [Windows 安全性](service-fabric-windows-cluster-windows-security.md)。 请阅读[保护群集](service-fabric-cluster-security.md)，详细了解 Service Fabric 群集安全性。

### <a name="step-1-create-the-cluster"></a>步骤 1：创建群集

#### <a name="scenario-a-create-an-unsecured-local-development-cluster"></a>方案 A：创建不受保护的本地开发群集
可以使用[示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)中包含的 *ClusterConfig.Unsecure.DevCluster.json* 文件将 Service Fabric 部署到一个计算机开发群集。

将独立包解压缩到计算机，将示例配置文件复制到本地计算机，然后通过管理员 PowerShell 会话从独立包文件夹中运行 *CreateServiceFabricCluster.ps1* 脚本。

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA
```

有关故障排除的详细信息，请参阅[计划和准备群集部署](service-fabric-cluster-standalone-deployment-preparation.md)中的环境设置部分。

如果已完成运行开发方案，则可以引用“[删除群集](#removecluster_anchor)”部分的步骤，从计算机中删除 Service Fabric 群集。 

#### <a name="scenario-b-create-a-multi-machine-cluster"></a>方案 B：创建多台计算机的群集
在完成[计划和准备群集部署](service-fabric-cluster-standalone-deployment-preparation.md)中详述的计划和准备步骤后，就可以使用群集配置文件创建生产群集。

部署和配置群集的群集管理员必须拥有计算机的管理员权限。 不能在域控制器上安装 Service Fabric。

1. 独立包中的 *TestConfiguration.ps1* 脚本可以用作最佳做法分析器，验证群集能否在给定环境中部署。 [部署准备](service-fabric-cluster-standalone-deployment-preparation.md)列出了先决条件和环境要求。 请运行以下脚本，验证你能否创建开发群集：  

    ```powershell
    .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.json
    ```

    应该会看到与下面类似的输出。 如果底部字段“Passed”的返回值为“True”，那么已通过完整性检查，并且根据输入配置群集看似可以部署。

    ```powershell
    Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
    Running Best Practices Analyzer...
    Best Practices Analyzer completed successfully.
    
    LocalAdminPrivilege        : True
    IsJsonValid                : True
    IsCabValid                 : True
    RequiredPortsOpen          : True
    RemoteRegistryAvailable    : True
    FirewallAvailable          : True
    RpcCheckPassed             : True
    NoConflictingInstallations : True
    FabricInstallable          : True
    Passed                     : True
    ```

2. 创建群集：运行“CreateServiceFabricCluster.ps1”脚本，以便在配置中的每台计算机上部署 Service Fabric 群集。 
    ```powershell
    .\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -AcceptEULA
    ```

> [!NOTE]
> 部署跟踪被写入到运行 CreateServiceFabricCluster.ps1 PowerShell 脚本的 VM/计算机上。 这些信息可以从运行脚本的目录中的子文件夹 DeploymentTraces 中找到。 要确定是否已将 Service Fabric 正确部署到计算机，请根据群集配置文件 FabricSettings 部分中的详述找到 FabricDataRoot 目录（默认为 c:\ProgramData\SF）中安装的文件。 同样，可以看到 FabricHost.exe 和 Fabric.exe 进程在任务管理器中运行。
> 
> 

#### <a name="scenario-c-create-an-offline-internet-disconnected-cluster"></a>方案 C：创建脱机（Internet 断开）群集
创建群集时自动下载 Service Fabric 运行时包。 将群集部署到未连接 Internet 的计算机时，需要单独下载 Service Fabric 运行时包，并在创建群集时提供指向它的路径。
运行时包可从另一台连接到 Internet 的计算机单独下载：[下载链接 - Service Fabric 运行时 - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)。 将运行时包复制到从中部署脱机群集的位置，并通过运行包含 `-FabricRuntimePackagePath` 参数的 `CreateServiceFabricCluster.ps1` 创建群集，如此示例所示： 

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -FabricRuntimePackagePath .\MicrosoftAzureServiceFabric.cab
```

*.\ClusterConfig.json* 和 *.\MicrosoftAzureServiceFabric.cab* 分别是群集配置和运行时 .cab 文件的路径。

### <a name="step-2-connect-to-the-cluster"></a>步骤 2：连接至群集
连接到群集以验证群集是否正在运行且可用。 ServiceFabric PowerShell 模块与运行时一起安装。  可以从其中一个群集节点或使用 Service Fabric 运行时从远程计算机连接到群集。  [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) cmdlet 可建立到群集的连接。

若要连接到不安全的群集，请运行以下 PowerShell 命令：

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>
```

例如：
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.2345:19000
```

有关如何连接到群集的其他示例，请参阅[连接到安全群集](service-fabric-connect-to-secure-cluster.md)。 连接到群集以后，请使用 [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) cmdlet 显示群集中节点的列表，以及每个节点的状态信息。 每个节点的 **HealthState** 应该为“正常”。

```powershell
PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> Get-ServiceFabricNode |Format-Table

NodeDeactivationInfo NodeName IpAddressOrFQDN NodeType  CodeVersion  ConfigVersion NodeStatus NodeUpTime NodeDownTime HealthState
-------------------- -------- --------------- --------  -----------  ------------- ---------- ---------- ------------ -----------
                     vm2      localhost       NodeType2 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm1      localhost       NodeType1 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm0      localhost       NodeType0 5.6.220.9494 0                     Up 00:02:43   00:00:00              OK
```

### <a name="step-3-visualize-the-cluster-using-service-fabric-explorer"></a>步骤 3：使用 Service Fabric Explorer 可视化群集
[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 是一项很好的工具，适用于可视化群集和管理应用程序。  Service Fabric Explorer 是一项在群集中运行的服务，使用浏览器导航到 [http://localhost:19080/Explorer](http://localhost:19080/Explorer) 即可访问该服务。

群集仪表板提供了群集的概览，包括应用程序和节点运行状况的摘要。 节点视图显示群集的物理布局。 对于给定的节点，可以检查已在该节点上部署代码的应用程序。

![Service Fabric Explorer][service-fabric-explorer]

## <a name="add-and-remove-nodes"></a>添加和删除节点
当业务需要改变时，可以向独立 Service Fabric 群集添加或删除节点。 参阅[向 Service Fabric 独立群集添加或删节点](service-fabric-cluster-windows-server-add-remove-nodes.md)以了解详细步骤。

<a id="removecluster" name="removecluster_anchor"></a>
## <a name="remove-a-cluster"></a>删除群集
若要删除群集，请运行包文件夹中的 *RemoveServiceFabricCluster.ps1* Powershell 脚本，并传入 JSON 配置文件的路径。 可以选择性地指定删除日志的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理员访问权限的任何计算机上运行此脚本。 运行此脚本的计算机不一定是群集的一部分。

```powershell
# Removes Service Fabric from each machine in the configuration
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -Force
```

```powershell
# Removes Service Fabric from the current machine
.\CleanFabric.ps1
```

<a id="telemetry"></a>

## <a name="telemetry-data-collected-and-how-to-opt-out-of-it"></a>收集遥测数据以及如何选择不使用它
默认情况下，产品收集 Service Fabric 使用情况的遥测数据来改进产品。 安装过程中，分析器运行的最佳做法检查与 [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1) 的连接。 如果不可访问，安装会失败，除非选择不使用遥测。

1. 遥测管道尝试每天一次将以下数据上传到 [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1)。 这是尽力而为的上传，不会影响群集功能。 仅从运行故障转移管理器的主节点发送遥测数据。 其他节点不发送遥测数据。
2. 遥测数据由以下项组成：

* 服务数量
* ServiceType 数量
* 应用程序数量
* ApplicationUpgrade 数量
* FailoverUnit 数量
* InBuildFailoverUnit 数量
* UnhealthyFailoverUnit 数量
* 副本数量
* InBuildReplica 数量
* StandByReplica 数量
* OfflineReplica 数量
* CommonQueueLength
* QueryQueueLength
* FailoverUnitQueueLength
* CommitQueueLength
* 节点数量
* IsContextComplete：True/False
* ClusterId：这是为每个群集随机生成的 GUID
* ServiceFabricVersion
* 从其上传遥测数据的虚拟机或计算机的 IP 地址

若要禁用遥测，请将以下内容添加到群集配置中的*属性*：*enableTelemetry: false*。

<a id="previewfeatures" name="previewfeatures_anchor"></a>

## <a name="preview-features-included-in-this-package"></a>此包中包括的预览功能
无。


> [!NOTE]
> 从[适用于 Windows Server 的独立群集的新 GA 版本（版本 5.3.204.x）](https://azure.microsoft.com/blog/azure-service-fabric-for-windows-server-now-ga/)开始，可以将群集手动或自动升级到未来版本。 请参阅[升级独立 Service Fabric 群集版本](service-fabric-cluster-upgrade-windows-server.md)文档以获取详细信息。
> 
> 

## <a name="next-steps"></a>后续步骤
* [使用 PowerShell 部署和删除应用程序](service-fabric-deploy-remove-applications.md)
* [独立 Windows 群集的配置设置](service-fabric-cluster-manifest.md)
* [在独立 Service Fabric 群集中添加或删除节点](service-fabric-cluster-windows-server-add-remove-nodes.md)
* [升级独立 Service Fabric 群集版本](service-fabric-cluster-upgrade-windows-server.md)
* [使用运行 Windows 的 Azure VM 创建独立 Service Fabric 群集](service-fabric-cluster-creation-with-windows-azure-vms.md)
* [使用 Windows 安全性保护 Windows 上的独立群集](service-fabric-windows-cluster-windows-security.md)
* [使用 X509 证书保护 Windows 上的独立群集](service-fabric-windows-cluster-x509-security.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
[service-fabric-explorer]: ./media/service-fabric-cluster-creation-for-windows-server/sfx.png
