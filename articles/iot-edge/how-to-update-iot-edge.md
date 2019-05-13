---
title: 更新设备上的 IoT Edge 版本 - Azure IoT Edge | Microsoft Docs
description: 如何将 IoT Edge 设备更新为运行最新版本的安全守护程序和 IoT Edge 运行时
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/17/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: a3b6327b9e05b039696cc1743fc2d16c5e945e26
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65152634"
---
# <a name="update-the-iot-edge-security-daemon-and-runtime"></a>更新 IoT Edge 安全守护程序和运行时

当 IoT Edge 服务发布新版本时，可能需要更新 IoT Edge 设备，使其获得最新功能并改善安全性。 本文提供有关在新版本推出时如何更新 IoT Edge 设备的信息。 

若要转移到较新的版本，需要更新 IoT Edge 设备的两个组件。 第一个组件是安全守护程序，它在设备上运行并在设备启动时启动运行时模块。 目前，只能从设备本身更新安全守护程序。 第二个组件是由 IoT Edge 中心和 IoT Edge 代理模块组成的运行时。 根据部署的构造方式，可以从设备或者远程更新运行时。 

若要查找最新版本的 Azure IoT Edge，请参阅 [Azure IoT Edge 版本](https://github.com/Azure/azure-iotedge/releases)。

>[!IMPORTANT]
>如果在 Windows 设备上运行 Azure IoT Edge，则在下述某个条件适用于设备的情况下，请勿将其更新到版本 1.0.5： 
>* 尚未将设备升级到 Windows 内部版本 17763。 IoT Edge 版本 1.0.5 不支持低于 17763 的 Windows 内部版本。
>* 在 Windows 设备上运行 Java 或 Node.js 模块。 跳过版本 1.0.5，即使已将 Windows 设备更新到最新的内部版本。 
>
>有关 IoT Edge 版本 1.0.5 的详细信息，请参阅 [1.0.5 发行说明](https://github.com/Azure/azure-iotedge/releases/tag/1.0.5)。 若要详细了解如何不让开发工具更新到最新版本，请参阅 [IoT 开发人员博客](https://devblogs.microsoft.com/iotdev/)。


## <a name="update-the-security-daemon"></a>更新安全守护程序

IoT Edge 安全守护程序是一个本机组件，需要使用 IoT Edge 设备上的包管理器进行更新。 

使用命令 `iotedge version` 检查设备上运行的安全守护程序的版本。 

### <a name="linux-devices"></a>Linux 设备

在 Linux 设备上，请使用 apt-get 或相应的包管理器来更新安全守护程序。 

```bash
apt-get update
apt-get install libiothsm iotedge
```

### <a name="windows-devices"></a>Windows 设备

在 Windows 设备上使用 PowerShell 脚本更新安全守护程序。 该脚本自动拉取安全守护程序的最新版本。 

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Update-IoTEdge -ContainerOs <Windows or Linux>
```

运行更新 IoTEdge 命令从你的设备，以及两个运行时容器映像中删除安全守护程序。 （如果使用的 Windows 容器），config.yaml 文件将保持在设备，以及小鲸鱼容器引擎中的数据。 保持配置信息，则意味着无需提供的连接字符串或再次在更新过程中为你的设备的设备预配服务信息。 

如果你想要安装特定版本的安全守护程序，下载相应的 Microsoft Azure IoTEdge.cab 文件从[IoT Edge 释放](https://github.com/Azure/azure-iotedge/releases)。 然后，使用 `-OfflineInstallationPath` 参数指向文件位置。 有关详细信息，请参阅[脱机安装](how-to-install-iot-edge-windows.md#offline-installation)。

## <a name="update-the-runtime-containers"></a>更新运行时容器

更新 IoT Edge 代理和 IoT Edge 中心容器的方式取决于在部署中使用的是滚动标记（如 1.0）还是特定标记（如 1.0.2）。 

使用 `iotedge logs edgeAgent` 或 `iotedge logs edgeHub` 命令检查设备上目前安装的 IoT Edge 代理和 IoT Edge 中心模块的版本。 

  ![在日志中查找容器版本](./media/how-to-update-iot-edge/container-version.png)

### <a name="understand-iot-edge-tags"></a>了解 IoT Edge 标记

IoT Edge 代理和 IoT Edge 中心映像使用与之关联的 IoT Edge 版本进行标记。 可通过两种不同的方法对运行时映像使用标记： 

* **滚动更新标记** - 仅使用版本号的前两个值来获取匹配这些数字的最新映像。 例如，每当有新版本指向最新的 1.0.x 版时，就更新 1.0。 如果 IoT Edge 设备的容器运行时重新提取映像，则运行时模块会更新到最新版本。 建议在开发时使用此方法。 Azure 门户中的部署默认使用滚动更新标记。 
* **特定标记** - 使用版本号的所有三个值，以显式设置映像版本。 例如，1.0.2 在其初始版本发布后不会更改。 准备好更新时，可以在部署清单中声明新的版本号。 建议在生产环境中使用此方法。

### <a name="update-a-rolling-tag-image"></a>更新滚动更新标记映像

如果在部署中使用滚动更新标记（例如 mcr.microsoft.com/azureiotedge-hub:**1.0**），则需要在设备上强制实施容器运行时，以提取最新版本的映像。 

从 IoT Edge 设备中删除本地版本的映像。 在 Windows 计算机上，卸载安全守护程序时也会删除运行时映像，因此不需再次执行此步骤。 

```bash
docker rmi mcr.microsoft.com/azureiotedge-hub:1.0
docker rmi mcr.microsoft.com/azureiotedge-agent:1.0
```

可能需要使用 `-f`（强制）标志来删除映像。 

IoT Edge 服务将提取最新版本的运行时映像，并自动在设备上将其重新启动。 

### <a name="update-a-specific-tag-image"></a>更新特定标记映像

如果在部署中使用特定标记（例如 mcr.microsoft.com/azureiotedge-hub:**1.0.2**），则只需更新部署清单中的标记，并将更改应用到设备即可。 

在 Azure 门户中，运行时部署映像在“配置高级 Edge 运行时设置”部分中声明。 

![配置高级 Edge 运行时设置](./media/how-to-update-iot-edge/configure-runtime.png)

在 JSON 部署清单中，更新 **systemModules** 节中的模块映像。 

```json
"systemModules": {
  "edgeAgent": {
    "type": "docker",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-agent:1.0.2",
      "createOptions": ""
    }
  },
  "edgeHub": {
    "type": "docker",
    "status": "running",
    "restartPolicy": "always",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-hub:1.0.2",
      "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}], \"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
    }
  }
},
```

## <a name="next-steps"></a>后续步骤

查看最新的 [Azure IoT Edge 版本](https://github.com/Azure/azure-iotedge/releases)。

持续关注[物联网博客](https://azure.microsoft.com/blog/topics/internet-of-things/)中的最新更新和公告 