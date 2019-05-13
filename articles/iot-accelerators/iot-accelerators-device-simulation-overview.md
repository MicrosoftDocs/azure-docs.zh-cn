---
title: 设备模拟概述 - Azure | Microsoft Docs
description: 介绍设备模拟解决方案加速器及其功能。
author: dominicbetts
manager: philmea
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc
ms.date: 12/03/2018
ms.author: dobett
ms.openlocfilehash: f58eb05ed582cf18157a76f4d637d72a228f4e96
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464872"
---
# <a name="device-simulation-solution-accelerator-overview"></a>设备模拟解决方案加速器概述

在基于云的 IoT 解决方案中，设备在连接到云终结点后即可发送遥测数据，例如温度、位置和状态。 解决方案可以使用该遥测数据，方便你采取行动或汲取见解。

开发 IoT 解决方案时，试验和测试是该过程的重要部分。 在此过程中，模拟始终是一项重要的工具。 设备模拟的功能如下：

* 通过即时调整模拟设备行为，可快速启动原型让其运行，然后进行迭代。 通过此过程证明想法令人满意之后，再购买昂贵的硬件。 可以通过 Web UI 创建自定义设备，在数秒内生成原型设备。
* 通过模拟实际的设备行为，验证解决方案是否可以按照从设备到解决方案的预期方式正常工作。 可以采用 JavaScript 编写复杂的设备行为，以生成逼真的模拟遥测。
* 通过模拟正常、峰值和超出峰值的负载条件，对解决方案进行缩放测试。 缩放测试还有助于调整运行解决方案所需的 Azure 资源的大小。

![无人机模拟示例](media/iot-accelerators-device-simulation-overview/dronesimulation.png)

使用设备模拟时，可以定义模拟真实设备所需的设备模型。 该模型包括消息格式、孪生属性和方法。 还可以使用 JavaScript 模拟复杂的设备行为。

可以为一到一千台连接到 IoT 中心的设备运行模拟。 为了方便测试，可以选择为单独的环境部署 IoT 中心和设备模拟。

设备模拟可以免费使用。 但是，设备模拟需要部署到云中的 Azure 订阅并消耗 Azure 资源。 如果设备模拟不符合要求，你可以复制并修改 [GitHub 上提供的源代码](https://github.com/Azure/device-simulation-dotnet)。

## <a name="sample-simulations"></a>示例模拟

部署设备模拟时，会得到一些示例模拟和示例设备。 可以通过这些示例来学习如何使用设备模拟。 若要开始操作，请运行一个[模拟 10 辆卡车的示例模拟](quickstart-device-simulation-deploy.md)。 也可[使用已提供的多个示例设备中的一个来创建自己的模拟](iot-accelerators-device-simulation-create-simulation.md)。

![模拟配置](media/iot-accelerators-device-simulation-overview/samplesimulation1.png)

## <a name="custom-simulated-devices"></a>自定义模拟设备

可以使用设备模拟[创建自定义设备模型](iot-accelerators-device-simulation-create-custom-device.md)，以便在模拟中使用。 例如，可以定义一个新的冰箱设备模型，以便发送温度和湿度遥测数据。 自定义模拟设备适用于简单的设备行为，可以使用随机的、递增的或递减的遥测值。

![创建设备模型](media/iot-accelerators-device-simulation-overview/adddevicemodel.png)

## <a name="advanced-simulated-devices"></a>高级模拟设备

需要对设备发送的遥测值进行更多控制时，可以使用高级设备模型。 高级设备模型启用 JavaScript 支持，可以操作已发送的遥测值。 例如，可以模拟一辆在炎热的晴天停靠的车辆的内部温度 - 当外部温度上升时，内部温度会呈指数上升。

可以通过高级设备模型来[创建并上传自己的设备模型](iot-accelerators-device-simulation-advanced-device.md)，其中包含一个 JSON 设备定义文件和相应的 JavaScript 文件。

可以通过高级设备模型执行以下操作：

* 指定从设备发送的消息格式以及遥测类型。
* 使用自定义脚本生成遥测值，这些遥测值保留设备在一段时间的状态。
* 使用自定义脚本指定模拟设备对方法的响应方式。

## <a name="next-steps"></a>后续步骤

本文介绍了设备模拟解决方案加速器及其功能。 若要开始使用解决方案加速器，请继续学习快速入门：

> [!div class="nextstepaction"]
> [在 Azure 中部署并运行 IoT 设备模拟](quickstart-device-simulation-deploy.md)
