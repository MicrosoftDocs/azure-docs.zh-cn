---
title: 将 Windows IoT Core 设备连接到 Azure IoT Central 应用程序 | Microsoft Docs
description: 了解如何以设备开发人员的身份将 MXChip IoT DevKit 设备连接到 Azure IoT Central 应用程序。
author: miriambrus
ms.author: miriamb
ms.date: 04/05/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: e8d4ab46c598580a3a87f4344202f2700926bf5c
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510323"
---
# <a name="connect-a-windows-iot-core-device-to-your-azure-iot-central-application"></a>将 Windows IoT Core 设备连接到 Azure IoT Central 应用程序

本文介绍如何以设备开发人员的身份将 Windows IoT Core 设备连接到 Microsoft Azure IoT Central 应用程序。

## <a name="before-you-begin"></a>开始之前

若要完成本文中的步骤，需要以下各项：

- 基于“示例 Devkit”应用程序模板创建的 Azure IoT Central 应用程序。 有关详细信息，请参阅[创建应用程序快速入门](quick-deploy-iot-central.md)。

- 运行 Windows 10 IoT Core 操作系统的设备。 有关详细信息，请参阅[设置 Windows 10 IoT Core 设备](https://docs.microsoft.com/windows/iot-core/tutorials/quickstarter/devicesetup)。

- 与在开发计算机[Node.js](https://nodejs.org/) 8.0.0 版或更高版本。 若要检查版本，可以在命令行中运行 `node --version`。 Node.js 适用于各种操作系统。

## <a name="the-sample-devkits-application"></a>示例 Devkits 应用程序

从“示例 Devkit”应用程序模板创建的应用程序包含一个具有以下特征的 Windows IoT Core 设备模板：

- 设备的遥测度量：**湿度**，**温度**，和**压力**。
- 设置控制**风扇转速**。
- 设备属性**骰子数字**和云属性**位置**。

设备模板的配置的完整详细信息，请参阅[Windows IoT Core 设备模板详细信息](#device-template-details)。

## <a name="add-a-real-device"></a>添加真实设备

Azure IoT Central 应用程序中使用**Device Explorer**页，将添加从实际设备**Windows 10 IoT Core**设备模板。 请记下的设备连接详细信息 (**作用域 ID**，**设备 ID**，并**主键**)。 有关详细信息，请参阅[获取连接信息](howto-generate-connection-string.md#get-connection-information)。

## <a name="prepare-the-device"></a>准备设备

若要连接到 IoT 中心设备，它需要一个连接字符串。

[!INCLUDE [iot-central-howto-connection-string](../../includes/iot-central-howto-connection-string.md)]

若要访问的连接字符串的设备代码，将其保存在名为的文件**connection.string.iothub**文件夹中`C:\Data\Users\DefaultAccount\Documents\`Windows 10 IoT Core 设备上。

若要将复制**connection.string.iothub**文件从桌面计算机`C:\Data\Users\DefaultAccount\Documents\`你的设备上的文件夹，可以使用[Windows Device Portal](https://docs.microsoft.com/windows/iot-core/manage-your-device/deviceportal):

1. 使用 web 浏览器导航到你的设备上 Windows Device Portal。
1. 若要浏览你的设备上的文件，请选择**应用程序 > 文件资源管理器**。
1. 导航到**用户 Folders\Documents**。 然后将上传**connection.string.iothub**文件：

    ![上传连接字符串](media/howto-connect-windowsiotcore/device-portal.png)

## <a name="deploy-and-run"></a>部署和运行

若要部署和运行示例应用程序在设备上，可以使用[Windows Device Portal](https://docs.microsoft.com/windows/iot-core/manage-your-device/deviceportal):

1. 使用 web 浏览器导航到你的设备上 Windows Device Portal。
1. 若要部署和运行**Azure IoT 中心客户端**应用程序中，选择**应用 > 快速运行示例**。 然后选择**Azure IoT 中心客户端**。
1. 然后选择**部署并运行**。

    ![部署和运行](media/howto-connect-windowsiotcore/quick-run.png)

后几分钟的时间，您可以查看遥测数据从你的设备在 IoT Central 应用程序中。

[Windows Device Portal](https://docs.microsoft.com/windows/iot-core/manage-your-device/deviceportal)包括可用于你的设备进行故障排除的工具：

- **应用程序管理器**页可用于控制设备上运行的应用。
- 如果你没有连接到你的设备的监视器，则可以使用**设备设置**页后，可以捕获你的设备的屏幕截图。 例如:

    ![应用程序的屏幕截图](media/howto-connect-windowsiotcore/iot-hub-foreground-client.png)

## <a name="download-the-source-code"></a>下载源代码

如果你想要浏览和修改客户端应用程序的源代码，您可以从中进行下载[Windows iotcore 示例 GitHub 存储库](https://github.com/Microsoft/Windows-iotcore-samples/blob/master/Samples/Azure/IoTHubClients)。

## <a name="device-template-details"></a>设备模板详细信息

从“示例 Devkit”应用程序模板创建的应用程序包含一个具有以下特征的 Windows IoT Core 设备模板：

### <a name="telemetry-measurements"></a>遥测数据度量

| 字段名     | 单位  | 最小值 | 最大值 | 小数位数 |
| -------------- | ------ | ------- | ------- | -------------- |
| 湿度       | %      | 0       | 100     | 0              |
| temp           | °C     | -40     | 120     | 0              |
| 压力       | hPa    | 260     | 1260    | 0              |

### <a name="settings"></a>设置

数字设置

| 显示名称 | 字段名 | 单位 | 小数位数 | 最小值 | 最大值 | 初始 |
| ------------ | ---------- | ----- | -------------- | ------- | ------- | ------- |
| 风扇速度    | fanSpeed   | RPM   | 0              | 0       | 1000    | 0       |

### <a name="properties"></a>属性

| 类型            | 显示名称 | 字段名 | 数据类型 |
| --------------- | ------------ | ---------- | --------- |
| 设备属性 | 模具号   | dieNumber  | 数字    |
| 文本            | 位置     | 位置   | 不适用       |

## <a name="next-steps"></a>后续步骤

现在，已了解如何将 Windows IoT Core 设备连接到 Azure IoT Central 应用程序，建议下一步是了解如何[设置自定义设备模板](howto-set-up-template.md)IoT 设备。
