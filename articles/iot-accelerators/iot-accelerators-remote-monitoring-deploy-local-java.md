---
title: 在本地部署远程监视解决方案（通过 IntelliJ IDE）- Azure | Microsoft Docs
description: 本操作指南展示了如何使用 IntelliJ 将远程监视解决方案加速器部署到本地计算机，以用于测试和开发。
author: v-krghan
manager: dominicbetts
ms.author: v-krghan
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/24/2019
ms.topic: conceptual
ms.openlocfilehash: 996111fbe23000182dab774ba3bbad0cc6435824
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412721"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-locally---intellij"></a>在本地部署远程监视解决方案加速器 - IntelliJ

[!INCLUDE [iot-accelerators-selector-local](../../includes/iot-accelerators-selector-local.md)]

本文展示了如何将远程监视解决方案加速器部署到本地计算机，用于测试和开发。 你将了解如何在 IntelliJ 中运行微服务。 本地微服务部署使用以下云服务：云中的 IoT Hub、Cosmos DB、Azure 流分析和 Azure 时序见解服务。

若要在本地计算机上的 Docker 中运行远程监视解决方案加速器，请参阅[在本地部署远程监视解决方案加速器 - Docker](iot-accelerators-remote-monitoring-deploy-local-docker.md)。

## <a name="prerequisites"></a>必备组件

若要部署远程监视解决方案加速器使用的 Azure 服务，需要一个有效的 Azure 订阅。

如果没有帐户，只需花费几分钟就能创建一个免费试用帐户。 有关详细信息，请参阅[Azure 免费试用版](https://azure.microsoft.com/pricing/free-trial/)。

### <a name="machine-setup"></a>计算机设置

若要完成本地部署，需要使用在本地开发计算机上安装的以下工具：

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com)
* [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [IntelliJ 社区版](https://www.jetbrains.com/idea/download/)
* [IntelliJ 插件 Scala](https://plugins.jetbrains.com/plugin/1347-scala)
* [IntelliJ 插件 SBT](https://plugins.jetbrains.com/plugin/5007-sbt)
* [IntelliJ 插件 SBT 执行器](https://plugins.jetbrains.com/plugin/7247-sbt-executor)
* [Nginx](https://nginx.org/en/download.html)
* [Node.js v8](https://nodejs.org/) - 此软件是 PCS CLI 的先决条件，脚本使用 PCS CLI 来创建 Azure 资源。 请勿使用 Node.js v10。

> [!NOTE]
> IntelliJ IDE 适用于 Windows 和 Mac。

[!INCLUDE [iot-accelerators-local-setup-java](../../includes/iot-accelerators-local-setup-java.md)]

## <a name="run-the-microservices"></a>运行微服务

在本部分，我们将运行远程监视微服务。 在本机运行 Web UI，在 Docker 中运行设备模拟、身份验证和 ASA 管理器服务，并在 IntelliJ 中运行微服务。

### <a name="run-the-device-simulation-service"></a>运行设备模拟服务

打开一个新的命令提示符窗口，以确保能够访问在上一部分中由 **start.cmd** 脚本设置的环境变量。

运行以下命令，启动设备模拟服务的 Docker 容器。 该服务模拟远程监视解决方案的设备。

```cmd
<path_to_cloned_repository>\services\device-simulation\scripts\docker\run.cmd
```

### <a name="run-the-auth-service"></a>运行身份验证服务

打开一个新的命令提示符窗口，并运行以下命令启动身份验证服务的 Docker 容器。 该服务可用于管理有权访问 Azure IoT 解决方案的用户。

```cmd
<path_to_cloned_repository>\services\auth\scripts\docker\run.cmd
```

### <a name="run-the-asa-manager-service"></a>运行 ASA 管理器服务

打开一个新的命令提示符窗口，并运行以下命令启动 ASA 管理器服务的 Docker 容器。 该服务可用于管理 Azure 流分析 (ASA) 作业，包括设置配置以及启动、停止和监视其状态。

```cmd
<path_to_cloned_repository>\services\asa-manager\scripts\docker\run.cmd
```

### <a name="deploy-all-other-microservices-on-local-machine"></a>在本地计算机上部署其他所有微服务

以下步骤展示了如何在 IntelliJ 中运行远程监视微服务：

#### <a name="import-project"></a>导入项目

1. 启动 IntelliJ IDE
1. 选择“导入项目”，然后选择“azure-iot-pcs-remote-monitoring-java\services\build.sbt”

#### <a name="create-run-configurations"></a>创建运行配置

1. 选择“运行”>“编辑配置”
1. 选择“添加新配置”>“sbt 任务” 
1. 输入**名称**并将**任务**输入为“run” 
1. 根据要运行的服务选择**工作目录**
1. 单击“应用”>“确定”以保存你的选择。
1. 创建以下服务的运行配置：
    * WebService (services\config)
    * WebService (services\device-telemetry)
    * WebService (services\iothub-manager)
    * WebService (services\storage-adapter)

例如下, 图显示添加的服务配置：

[![Add-Configuration](./media/deploy-locally-intellij/run-configurations.png)](./media/deploy-locally-intellij/run-configurations.png#lightbox)


#### <a name="create-compound-configuration"></a>创建复合配置

1. 若要运行所有服务，请同时选择“添加新配置”>“复合”
1. 输入**名称**并**添加 sbt 任务**
1. 单击“应用”>“确定”以保存你的选择。

例如下, 图显示了将所有 sbt 任务都添加到单个配置：

[![Add-All-Services](./media/deploy-locally-intellij/all-services.png)](./media/deploy-locally-intellij/all-services.png#lightbox)

单击“运行”，以在本地计算机上生成并运行 Web 服务。

每个 Web 服务将打开一个命令提示符和 Web 浏览器窗口。 在命令提示符下，查看正在运行的服务的输出；在浏览器窗口中可以监视状态。 请不要关闭命令提示符或网页，此操作会停止 Web 服务。


若要访问服务的状态，可以导航到以下 URL：
* IoT 中心管理器 [http://localhost:9002/v1/status](http://localhost:9002/v1/status)
* 设备遥测 [http://localhost:9004/v1/status](http://localhost:9004/v1/status)
* 配置 [http://localhost:9005/v1/status](http://localhost:9005/v1/status)
* 存储适配器 [http://localhost:9022/v1/status](http://localhost:9022/v1/status)


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
