---
title: 使用 DPS 自动预配 Windows 设备 - Azure IoT Edge | Microsoft Docs
description: 使用 Windows 计算机上的模拟设备通过设备预配服务测试 Azure IoT Edge 的自动设备预配
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/09/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: 01247dfc0046ef722d70fe48f7ab8ee63f685962
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153558"
---
# <a name="create-and-provision-a-simulated-tpm-edge-device-on-windows"></a>在 Windows 上创建和预配模拟 TPM Edge 设备

可以使用[设备预配服务](../iot-dps/index.yml)自动预配 Azure IoT Edge 设备，就像预配未启用 Edge 的设备一样。 如果你不熟悉自动预配过程，请在继续操作之前查看[自动预配的概念](../iot-dps/concepts-auto-provisioning.md)。

本文介绍如何使用以下步骤，在模拟的 Edge 设备上测试自动预配：

* 创建 IoT 中心设备预配服务 (DPS) 的实例。
* 使用用于确保硬件安全性的模拟受信任平台模块 (TPM) 在 Windows 计算机上创建一个模拟设备。
* 为设备创建个人注册。
* 安装 IoT Edge 运行时并将设备连接到 IoT 中心。

## <a name="prerequisites"></a>必备组件

* 一台 Windows 开发计算机。 本文使用 Windows 10。
* 活动的 IoT 中心。

## <a name="set-up-the-iot-hub-device-provisioning-service"></a>设置 IoT 中心设备预配服务

在 Azure 中创建 IoT 中心设备预配服务的新实例，并将其链接到 IoT 中心。 可以遵照[设置 IoT 中心 DPS](../iot-dps/quick-setup-auto-provision.md) 中的说明操作。

运行设备预配服务后，从概述页复制“ID 范围”的值。 配置 IoT Edge 运行时时，需要使用此值。

## <a name="simulate-a-tpm-device"></a>模拟 TPM 设备

在 Windows 开发计算机上创建模拟 TPM 设备。 检索设备的“注册 ID”和“认可密钥”，并使用它们在 DPS 中创建个人注册条目。

在 DPS 中创建注册时，可以声明“初始设备孪生状态”。 在设备孪生中可以设置标记，以便按解决方案中所需的任何指标（例如区域、环境、位置或设备类型）将设备分组。 这些标记用于创建[自动部署](how-to-deploy-monitor.md)。

选择要用来创建模拟设备的 SDK 语言，并遵循本文中的步骤，直到创建了个人注册为止。

创建个人注册时，请选择“启用”，将 Windows 开发计算机上的模拟 TPM 设备声明为“IoT Edge设备”。

模拟设备和个人注册指南：

* [C](../iot-dps/quick-create-simulated-device.md)
* [Java](../iot-dps/quick-create-simulated-device-tpm-java.md)
* [C#](../iot-dps/quick-create-simulated-device-tpm-csharp.md)
* [Node.js](../iot-dps/quick-create-simulated-device-tpm-node.md)
* [Python](../iot-dps/quick-create-simulated-device-tpm-python.md)

创建个人注册后，保存“注册 ID”的值。 配置 IoT Edge 运行时时，需要使用此值。

## <a name="install-the-iot-edge-runtime"></a>安装 IoT Edge 运行时

完成上一部分后，应该会发现新设备在 IoT 中心内列为 IoT Edge 设备。 现在，需要在设备上安装 IoT Edge 运行时。

IoT Edge 运行时部署在所有 IoT Edge 设备上。 该运行时的组件在容器中运行，允许你将其他容器部署到设备，以便在边缘上运行代码。  

遵照说明在运行上一部分所述的模拟 TPM 的设备上安装 IoT Edge 运行时。 确保将 IoT Edge 运行时配置为自动预配而不是手动预配。

在设备上安装 IoT Edge 之前，请先了解 DPS **ID 范围**和设备**注册 ID**。

[安装和自动预配 IoT Edge](how-to-install-iot-edge-windows.md#option-2-install-and-automatically-provision)

## <a name="verify-successful-installation"></a>验证是否成功安装

如果运行时成功启动，则可以进入 IoT 中心，开始将 IoT Edge 模块部署到你的设备。 在设备上使用以下命令验证是否已成功安装并启动运行时。  

检查 IoT Edge 服务的状态。

```powershell
Get-Service iotedge
```

检查过去 5 分钟的服务日志。


```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

列出正在运行的模块。

```powershell
iotedge list
```

## <a name="next-steps"></a>后续步骤

使用设备预配服务注册过程可以在预配新设备的同时，设置设备 ID 和设备孪生标记。 可以在自动设备管理中，使用这些值将单个设备或设备组指定为目标。 了解如何[使用 Azure 门户大规模部署和监视 IoT Edge 模块](how-to-deploy-monitor.md)，或[使用 Azure CLI](how-to-deploy-monitor-cli.md) 执行此操作
