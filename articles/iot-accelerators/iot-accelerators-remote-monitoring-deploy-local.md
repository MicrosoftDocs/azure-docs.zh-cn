---
title: 在本地部署远程监视解决方案（通过 Visual Studio IDE）- Azure | Microsoft Docs
description: 本操作指南展示了如何使用 Visual Studio 将远程监视解决方案加速器部署到本地计算机，以用于测试和开发。
author: avneet723
manager: hegate
ms.author: avneet723
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/17/2019
ms.topic: conceptual
ms.openlocfilehash: a5754fc2f7e0fc36a6708bfd4d2d1d3d6f016b27
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150967"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-locally---visual-studio"></a>在本地部署远程监视解决方案加速器 - Visual Studio

[!INCLUDE [iot-accelerators-selector-local](../../includes/iot-accelerators-selector-local.md)]

本文展示了如何将远程监视解决方案加速器部署到本地计算机，用于测试和开发。 了解如何在 Visual Studio 中运行微服务。 本地微服务部署使用以下云服务：云中的 IoT Hub、Cosmos DB、Azure 流分析和 Azure 时序见解服务。

若要在本地计算机上的 Docker 中运行远程监视解决方案加速器，请参阅[在本地部署远程监视解决方案加速器 - Docker](iot-accelerators-remote-monitoring-deploy-local-docker.md)。

## <a name="prerequisites"></a>必备组件

若要部署远程监视解决方案加速器使用的 Azure 服务，需要一个有效的 Azure 订阅。

如果没有帐户，只需花费几分钟就能创建一个免费试用帐户。 有关详细信息，请参阅 [Azure 免费试用](https://azure.microsoft.com/pricing/free-trial/)。

### <a name="machine-setup"></a>计算机设置

若要完成本地部署，需要使用在本地开发计算机上安装的以下工具：

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com)
* [Visual Studio](https://visualstudio.microsoft.com/)
* [Nginx](https://nginx.org/en/download.html)
* [Node.js v8](https://nodejs.org/) - 此软件是 PCS CLI 的先决条件，脚本使用 PCS CLI 来创建 Azure 资源。 请勿使用 Node.js v10。

> [!NOTE]
> Visual Studio 适用于 Windows 和 Mac。

[!INCLUDE [iot-accelerators-local-setup](../../includes/iot-accelerators-local-setup.md)]

## <a name="run-the-microservices"></a>运行微服务

在本部分，我们将运行远程监视微服务。 在本地运行 Web UI，在 Docker 中运行设备模拟服务，并在 Visual Studio 中运行微服务。

### <a name="run-the-device-simulation-service"></a>运行设备模拟服务

打开一个新的命令提示符窗口，以确保能够访问在上一部分中由 **start.cmd** 脚本设置的环境变量。

运行以下命令，启动设备模拟服务的 Docker 容器。 该服务模拟远程监视解决方案的设备。

```cmd
<path_to_cloned_repository>\services\device-simulation\scripts\docker\run.cmd
```

### <a name="deploy-all-other-microservices-on-local-machine"></a>在本地计算机上部署其他所有微服务

以下步骤说明如何在 Visual Studio 2017 中运行远程监视微服务：

1. 启动 Visual Studio 2017
1. 打开存储库本地副本中 **services** 文件夹内的 **remote-monitoring.sln** 解决方案。
1. 在“解决方案资源管理器”中右键单击解决方案，然后单击“属性”。
1. 选择“通用属性”>“启动项目”。
1. 选择“多个启动项目”，并将以下项目的“操作”设置为“启动”：
    * WebService (asa-manager\WebService)
    * WebService (auth\WebService)
    * WebService (config\WebService)
    * WebService (device-telemetry\WebService)
    * WebService (iothub-manager\WebService)
    * WebService (storage-adapter\WebService)
1. 单击“确定”以保存选择。
1. 单击“调试”>“开始调试”，以在本地计算机上生成并运行 Web 服务。

每个 Web 服务将打开一个命令提示符和 Web 浏览器窗口。 在命令提示符下，查看正在运行的服务的输出；在浏览器窗口中可以监视状态。 请不要关闭命令提示符或网页，此操作会停止 Web 服务。

### <a name="start-the-stream-analytics-job"></a>启动流分析作业

遵循以下步骤启动流分析作业：

1. 导航到 [Azure 门户](https://portal.azure.com)。
1. 导航到为解决方案创建的**资源组**。 该资源组的名称是在运行 **start.cmd** 脚本时为解决方案选择的名称。
1. 在资源列表中单击“流分析作业”。
1. 在流分析作业的“概述”页上，单击“启动”按钮。 然后单击“启动”以立即启动该作业。

### <a name="run-the-web-ui"></a>运行 Web UI

此步骤启动 Web UI。 打开一个新的命令提示符窗口，以确保能够访问由 **start.cmd** 脚本设置的环境变量。 导航到存储库本地副本中的 **webui** 文件夹，并运行以下命令：

```cmd
npm install
npm start
```

启动完成后，在浏览器显示页面**http:\//localhost:3000 / 仪表板**。 此页面上出现的错误在意料之中。 若要在无错误的情况下查看应用程序，请完成以下步骤。

### <a name="configure-and-run-nginx"></a>配置并运行 NGINX

设置反向代理服务器，以链接本地计算机上运行的 Web 应用程序和微服务：

* 将存储库的本地副本的 **webui\scripts\localhost** 文件夹中的 **nginx.conf** 文件复制到 **nginx\conf** 安装目录。
* 运行 **nginx**。

有关运行 **nginx** 的详细信息，请参阅[适用于 Windows 的 nginx](https://nginx.org/en/docs/windows.html)。

### <a name="connect-to-the-dashboard"></a>连接到仪表板

若要访问远程监视解决方案仪表板，导航到 http:\//localhost:9000 在浏览器中的。

## <a name="clean-up"></a>清理

为避免产生不必要的费用，在完成测试后，请从 Azure 订阅中删除云服务。 若要删除这些服务，请导航到 [Azure 门户](https://ms.portal.azure.com)，并删除 **start.cmd** 脚本创建的资源组。

还可以删除在从 GitHub 克隆源代码时创建的远程监视存储库的本地副本。

## <a name="next-steps"></a>后续步骤

部署远程监视解决方案后，接下来请[探索解决方案仪表板的功能](quickstart-remote-monitoring-deploy.md)。
