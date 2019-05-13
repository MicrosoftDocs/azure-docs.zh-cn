---
title: 设置 Azure 微服务的 Windows 开发环境 | Microsoft Docs
description: 安装运行时、SDK 和工具并创建本地开发群集。 完成此设置后，就可以在 Windows 上开始生成应用程序。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: chackdan
editor: ''
ms.assetid: b94e2d2e-435c-474a-ae34-4adecd0e6f8f
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/30/2019
ms.author: aljo
ms.openlocfilehash: 463b05f57ce0c85ebf1732791cb024335103b780
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153616"
---
# <a name="prepare-your-development-environment-on-windows"></a>在 Windows 上准备开发环境
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md) 
> * [Linux](service-fabric-get-started-linux.md)
> * [OSX](service-fabric-get-started-mac.md)
> 
> 

若要在 Windows 开发计算机上生成并运行 [Azure Service Fabric 应用程序][1]，请安装 Service Fabric 运行时、SDK 和工具。 此外，还需要启用 SDK 中包含的 [Windows PowerShell 脚本执行](#enable-powershell-script-execution)。

## <a name="prerequisites"></a>必备组件
### <a name="supported-operating-system-versions"></a>支持的操作系统版本
支持使用以下操作系统版本进行开发：

* Windows 7
* Windows 8/Windows 8.1
* Windows Server 2012 R2
* Windows Server 2016
* Windows 10

> [!NOTE]
> Windows 7 支持：
> - 默认情况下，Windows 7 仅包含 Windows PowerShell 2.0。 Service Fabric PowerShell cmdlet 需要 PowerShell 3.0 或更高版本。 可以从 Microsoft 下载中心[下载 Windows PowerShell 5.0][powershell5-download]。
> - 在 Windows 7 上无法使用 Service Fabric 反向代理。
>

## <a name="install-the-sdk-and-tools"></a>安装 SDK 和工具
Web 平台安装程序 (WebPI) 是安装 SDK 和工具的建议方法。 如果使用 WebPI 收到运行时错误，还可以在特定 Service Fabric 版本的发行说明中找到指向安装程序的直接链接。 发行说明可以在 [Service Fabric 团队博客](https://blogs.msdn.microsoft.com/azureservicefabric/)的各种发布公告中找到。

> [!NOTE]
> 不支持本地 Service Fabric 开发群集升级。

### <a name="to-use-visual-studio-2017"></a>使用 Visual Studio 2017
Service Fabric 工具是 Visual Studio 2017 中 Azure 开发工作负荷的一部分。 在 Visual Studio 安装过程中启用此工作负荷。
此外还需使用 Web 平台安装程序安装 Microsoft Azure Service Fabric SDK 和运行时。

* [安装 Microsoft Azure Service Fabric SDK][core-sdk]

### <a name="to-use-visual-studio-2015-requires-visual-studio-2015-update-2-or-later"></a>使用 Visual Studio 2015（要求 Visual Studio 2015 Update 2 或更高版本）
对于 Visual Studio 2015，Service Fabric 工具是使用 Web 平台安装程序与 SDK 和运行时 一起安装的：

* [安装 Microsoft Azure Service Fabric SDK 和工具][full-bundle-vs2015]

### <a name="sdk-installation-only"></a>仅安装 SDK
如果只需 SDK，可以安装此包：
* [安装 Microsoft Azure Service Fabric SDK][core-sdk]

当前版本包括：
* Service Fabric SDK 和工具 3.3.658
* Service Fabric 运行时 6.4.658
* 适用于 Visual Studio 2015 的 Service Fabric 工具 2.4.11116.1
* Visual Studio 2017 15.9 包括适用于 Visual Studio 的 Service Fabric 工具 2.4.11024.1 

有关支持版本的列表，请参阅 [Service Fabric 版本](service-fabric-versions.md)

> [!NOTE]
> 应用程序或群集升级不支持单机群集 (OneBox)；如果需要执行群集升级，或在执行应用程序升级时遇到任何问题，请删除 OneBox 群集并重新创建它。 

## <a name="enable-powershell-script-execution"></a>启用 PowerShell 脚本执行
Service Fabric 使用 Windows PowerShell 脚本创建本地开发群集和部署 Visual Studio 中的应用程序。 默认情况下，Windows 会阻止这些脚本运行。 要启用它们，必须修改 PowerShell 执行策略。 以管理员身份打开 PowerShell 并输入以下命令：

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```
## <a name="install-docker-optional"></a>安装 Docker（可选）
[Service Fabric 是容器业务流程协调程序](service-fabric-containers-overview.md)，用于跨计算机群集部署微服务。 若要在本地开发群集上运行 Windows 容器应用程序，必须先安装用于 Windows 的 Docker。 获取[用于 Windows 的 Docker CE (稳定版)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description)。 安装并启动 Docker 以后，右键单击任务栏图标，并选择“切换到 Windows 容器”。 此步骤是运行基于 Windows 的 Docker 映像所必需的。

## <a name="next-steps"></a>后续步骤
完成设置开发环境之后，便可开始生成和运行应用。

* [了解如何创建、部署和管理应用程序](service-fabric-tutorial-create-dotnet-app.md)
* [了解编程模型：Reliable Services 和 Reliable Actors](service-fabric-choose-framework.md)
* [在 GitHub 上检查 Service Fabric 代码示例](https://aka.ms/servicefabricsamples)
* [使用 Service Fabric 资源管理器可视化群集](service-fabric-visualizing-your-cluster.md)
* 了解 [Service Fabric 支持选项](service-fabric-support.md)

[1]: https://azure.microsoft.com/campaigns/service-fabric/ "Service Fabric 活动页"
[2]: https://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015 "VS 2015 WebPI 链接"
[full-bundle-dev15]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15 "Dev15 WebPI 链接"
[core-sdk]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK "Core SDK WebPI 链接"
[powershell5-download]:https://www.microsoft.com/en-us/download/details.aspx?id=50395
