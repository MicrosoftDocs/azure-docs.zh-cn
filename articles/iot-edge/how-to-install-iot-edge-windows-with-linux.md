---
title: 在 Windows 上安装适用于 Linux 的 Azure IoT Edge |Microsoft Docs
description: 在 Windows、 Windows Server 和 Windows IoT Core 上的 Linux 容器的 azure IoT Edge 安装说明
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: kgremban
ms.openlocfilehash: bb47a1b828084673961a6d2c5657793b4437f294
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160570"
---
# <a name="use-iot-edge-on-windows-to-run-linux-containers"></a>在 Windows 上使用 IoT Edge 运行 Linux 容器

测试使用 Windows 计算机的 Linux 设备的 IoT Edge 模块。 

在生产方案中，Windows 设备应仅运行 Windows 容器。 但是，一种常见开发方案是使用 Windows 计算机来生成适用于 Linux 设备的模块的 IoT Edge。 Windows IoT Edge 运行时，可运行 Linux 的容器**测试和开发**目的。 

本文列出了安装在 Windows x64 (AMD/Intel) 上使用 Linux 容器的 Azure IoT Edge 运行时的步骤系统。 若要了解有关 IoT Edge 运行时安装程序，包括所有安装参数的详细信息的详细信息请参阅[在 Windows 上安装 Azure IoT Edge 运行时](how-to-install-iot-edge-windows.md)。

## <a name="prerequisites"></a>必备组件

参考本部分检查你的 Windows 设备是否支持 IoT Edge，并在安装之前为容器引擎准备好该设备。 

### <a name="supported-windows-versions"></a>支持的 Windows 版本

与 Linux 容器的 azure IoT Edge 可以在以下版本的 Windows 上运行： 
* Windows 10 周年更新（内部版本 14393）或更高版本
* Windows Server 2016 或更高版本

有关最新版 IoT Edge 包含的功能的详细信息，请参阅 [Azure IoT Edge 发行说明](https://github.com/Azure/azure-iotedge/releases)。

若要在虚拟机上安装 IoT Edge，请启用嵌套虚拟化并分配至少 2-GB 内存。 如何启用嵌套虚拟化取决于所用的虚拟机监控程序。 就 Hyper-V 来说，第 2 代虚拟机已默认启用嵌套虚拟化。 如果使用 VMWare，则可通过切换开关在虚拟机上启用此功能。 

### <a name="prepare-the-container-engine"></a>准备容器引擎 

Azure IoT Edge 依赖于 [OCI 兼容的](https://www.opencontainers.org/)容器引擎。 在 Windows 计算机是 IoT Edge 安装包括 Windows 容器运行时，但你需要安装 IoT Edge 之前为 Linux 容器提供自己的运行时上运行 Windows 和 Linux 容器之间的最大配置差异。 

若要设置用于开发和测试容器的 Linux 设备的 Windows 计算机，可以使用[Docker 桌面](https://www.docker.com/docker-windows)作为容器引擎。 您需要安装 Docker 并对其进行配置[使用 Linux 容器](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers)之前安装 IoT Edge。  

如果 IoT Edge 设备是 Windows 计算机，请检查它是否符合 Hyper-V 的[系统要求](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)。

## <a name="install-iot-edge-on-a-new-device"></a>在新设备上安装 IoT Edge

>[!NOTE]
>Azure IoT Edge 软件程序包受制于程序包中的许可条款（位于 LICENSE 目录中）。 使用程序包之前请阅读这些许可条款。 安装和使用程序包即表示接受这些条款。 如果不同意许可条款，则不要使用包。

某个 PowerShell 脚本将下载并安装 Azure IoT Edge 安全守护程序。 然后，安全守护程序将启动两个运行时模块中的第一个，即 IoT Edge 代理，以便能够远程部署其他模块。 

首次在设备上安装 IoT Edge 运行时时，需要使用 IoT 中心内的标识预配该设备。 可以使用 IoT 中心提供设备连接字符串手动预配单个 IoT Edge 设备。 或者，可以使用设备预配服务自动预配设备，需要设置多个设备时，这种做法非常有用。 

你可以阅读更多有关的不同安装选项和一文中的参数[在 Windows 上安装 Azure IoT Edge 运行时](how-to-install-iot-edge-windows.md)。 安装和配置 Linux 容器的 Docker 桌面后，主安装差异声明使用 Linux **-ContainerOs**参数。 例如： 

1. 如果你尚未准备好，注册新的 IoT Edge 设备，并检索设备连接字符串。 复制连接字符串以供以后使用在本部分中。 你可以完成此步骤中使用以下工具：

   * [Azure 门户](how-to-register-device-portal.md)
   * [Azure CLI](how-to-register-device-cli.md)
   * [Visual Studio Code](how-to-register-device-vscode.md)

2. 以管理员身份运行 PowerShell。

3. **部署 IoTEdge**命令将检查你的 Windows 计算机上受支持的版本中，将启用容器功能，并下载小鲸鱼运行库 （其中未使用 Linux 容器） 和 IoT Edge 运行时。 命令默认为 Windows 容器，因此将 Linux 声明为所需的容器操作系统。 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge -ContainerOs Linux
   ```

4. 在此情况下，IoT Core 设备可能会自动重新启动。 Windows 10 或 Windows Server 的其他设备可能会提示您重新启动。 如果是这样，立即重新启动你的设备。 你的设备准备就绪后，再次以管理员身份运行 PowerShell。

5. **Initialize IoTEdge**命令将 IoT Edge 运行时配置在计算机上。 该命令默认为手动预配设备连接字符串。 再次声明为所需的容器操作系统的 Linux。 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ContainerOs Linux
   ```

6. 出现提示时，提供你在步骤 1 中检索的设备连接字符串。 设备连接字符串将物理设备与 IoT 中心中的设备 ID 相关联。 

   设备连接字符串会采用以下格式，并且不应该包括引号： `HostName={IoT hub name}.azure-devices.net;DeviceId={device name};SharedAccessKey={key}`

## <a name="verify-successful-installation"></a>验证是否成功安装

检查 IoT Edge 服务的状态。 它应列出为正在运行。  

```powershell
Get-Service iotedge
```

检查过去 5 分钟的服务日志。 

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

列出正在运行的模块。 完成新的安装后，应会看到唯一运行的模块是 **edgeAgent**。 [部署 IoT Edge 模块](how-to-deploy-modules-portal.md)后，将会看到其他模块。 

```powershell
iotedge list
```

## <a name="next-steps"></a>后续步骤

预配了安装运行时的 IoT Edge 设备后，现在可以[部署 IoT Edge 模块](how-to-deploy-modules-portal.md)。

如果无法正常安装 IoT Edge，请查看[故障排除](troubleshoot.md)页。

若要将现有安装更新到最新版本的 IoT Edge，请参阅[更新 IoT Edge 安全守护程序和运行时](how-to-update-iot-edge.md)。
