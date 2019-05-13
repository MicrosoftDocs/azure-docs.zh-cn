---
title: 将 IoT DevKit 连接到远程监视解决方案加速器 - Azure | Microsoft Docs
description: 本操作方法指南介绍如何将 IoT DevKit AZ3166 设备上的传感器的遥测数据发送到远程监视解决方案加速器，以便进行监视和可视化。
author: isabelcabezasm
manager: ''
ms.service: iot-accelerators
services: iot-accelerators
ms.devlang: c
ms.topic: conceptual
ms.date: 11/29/2018
ms.author: isacabe
ms.openlocfilehash: 3551d088c1d02715bf9ace09d7eb0048bc10111e
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473471"
---
# <a name="connect-an-iot-devkit-device-to-the-remote-monitoring-solution-accelerator"></a>将 IoT DevKit 设备连接到远程监视解决方案加速器

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

本操作说明指南介绍如何在 IoT DevKit 设备上运行示例应用程序。 示例代码将遥测数据从 DevKit 设备上的传感器发送到解决方案加速器。

[MXChip IoT DevKit](https://aka.ms/iot-devkit) 是具有多种外设和传感器的集成 Arduino 兼容板。 可以使用 Visual Studio Code 中的 [Azure IoT Device Workbench](https://aka.ms/iot-workbench) 或 [Azure IoT 工具](https://aka.ms/azure-iot-tools)扩展包进行开发。 [项目目录](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/)包含有助于确定 IoT 解决方案原型的示例应用程序。

## <a name="before-you-begin"></a>开始之前

若要完成本教程中的步骤，请先执行以下任务：

* 遵循[将 IoT DevKit AZ3166 连接到云中的 Azure IoT 中心](/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)中的步骤准备好 DevKit。

## <a name="open-sample-project"></a>打开示例项目

若要在 VS Code 中打开远程监视示例，请执行以下操作：

1. 确保 IoT DevKit 未连接到计算机。 先启动 VS Code，然后将 DevKit 连接到计算机。

1. 单击 `F1` 以打开命令面板，键入并选择“Azure IoT Device Workbench:**打开示例...”** 。然后选择“IoT DevKit”作为开发板。

1. 找到“远程监视”，然后单击“打开示例”。 此时会打开一个新的 VS Code 窗口，其中显示项目文件夹：

   ![IoT Workbench，选择“远程监视”示例](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/iot-workbench-example.png)

## <a name="configure-the-device"></a>配置设备

若要在 DevKit 设备上配置 IoT 中心设备连接字符串，请执行以下操作：

1. 将 IoT DevKit 转换为“配置模式”：

    * 按住按钮 A。
    * 按下然后松开“重置”按钮。

1. 屏幕将显示 DevKit ID 和 `Configuration`。

    ![IoT DevKit 配置模式](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/devkit-configuration-mode.png)

1. 按 F1 打开命令面板，键入并选择“Azure IoT Device Workbench：配置设备设置... > 配置设备连接字符串” **。

1. 粘贴以前复制的连接字符串，然后按 **Enter** 对设备进行配置。

## <a name="build-the-code"></a>生成代码

若要生成并上传设备代码，请执行以下操作：

1. 按 `F1` 以打开命令面板，键入并选择“Azure IoT Device Workbench：上传设备代码” **：

1. VS Code 会编译代码并将其上传到 DevKit 设备：

    ![IoT Workbench：设备 - > 上传](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/iot-workbench-device-uploaded.png)

1. DevKit 设备重启并运行上传的代码。

## <a name="test-the-sample"></a>测试示例

若要验证上传到 DevKit 设备的示例应用程序是否正常运行，请完成以下步骤：

### <a name="view-the-telemetry-sent-to-remote-monitoring-solution"></a>查看发送到远程监视解决方案的遥测数据

当示例应用运行时，DevKit 设备会将遥测数据从其传感器通过 Wi-Fi 发送到解决方案加速器。 若要查看遥测数据，请执行以下操作：

1. 转到解决方案仪表板，然后单击**Device Explorer**。

1. 单击 DevKit 设备的设备名称。 在右侧选项卡上，可以实时查看 DevKit 中的遥测数据：

    ![Azure IoT 套件中的传感器数据](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/azure-iot-suite-dashboard.png)

### <a name="control-the-devkit-device"></a>控制 DevKit 设备

借助远程监视解决方案加速器，可以远程控制自己的设备。 示例代码实现中所示的三种方法**方法**部分上选择该设备时**Device Explorer**页：

![IoT DevKit 方法](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/azure-iot-suite-methods.png)

若要更改某个 DevKit LED 的颜色，请使用 **LedColor** 方法：

1. 从设备列表中选择设备名称，然后单击“作业”：

    ![创建作业](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/azure-iot-suite-job.png)

1. 使用以下值配置“作业”，然后单击“应用”：

   * 选择作业：**Run 的方法**
   * 方法名称：**LedColor**
   * 作业名称：**ChangeLedColor**

     ![作业设置](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/iot-suite-change-color.png)

1. 数秒钟后，DevKit 上 RGB LED（位于按钮 A 下面）的颜色会变化：

    ![IoT DevKit 红色 led](media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringv2/azure-iot-suite-devkit-led.png)

## <a name="clean-up-resources"></a>清理资源

如果打算继续学习教程，请将远程监视解决方案加速器保持为部署状态。

如果不再需要该解决方案加速器，请在“预配的解决方案”页中选择它，然后单击“删除解决方案”将其删除：

![删除解决方案](media/quickstart-remote-monitoring-deploy/deletesolution.png)

## <a name="problems-and-feedback"></a>问题和反馈

如果遇到问题，请参阅 [IoT DevKit 常见问题解答](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/)，或通过以下渠道联系我们：

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>后续步骤

了解如何将 DevKit 设备连接到远程监视解决方案加速器后，建议接下来学习以下教程：

* [Azure IoT 解决方案加速器概述](https://docs.microsoft.com/azure/iot-accelerators/)
* [自定义 UI](iot-accelerators-remote-monitoring-customize.md)
* [将 IoT DevKit 连接到 Azure IoT Central 应用程序](../iot-central/howto-connect-devkit.md)