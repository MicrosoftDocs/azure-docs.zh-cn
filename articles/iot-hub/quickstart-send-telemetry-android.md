---
title: 快速入门：向 Azure IoT 中心发送遥测数据 (Android) | Microsoft Docs
description: 在本快速入门中，请运行一个示例 Android 应用程序，以便向 IoT 中心发送模拟遥测数据，以及从 IoT 中心读取需在云中处理的遥测数据。
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/15/2019
ms.author: wesmc
ms.openlocfilehash: 6c7428a4f34f0be64423c42efc06667cb18aa025
ms.sourcegitcommit: 64def2a06d4004343ec3396e7c600af6af5b12bb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/19/2020
ms.locfileid: "77471262"
---
# <a name="quickstart-send-iot-telemetry-from-an-android-device"></a>快速入门：从 Android 设备发送 IoT 遥测数据

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

在本快速入门中，请通过运行在物理或模拟设备上的 Android 应用程序将遥测数据发送到 Azure IoT 中心。 IoT 中心是一项 Azure 服务，用于将大量遥测数据从 IoT 设备引入云中进行存储或处理。 本快速入门使用预先编写的 Android 应用程序来发送遥测数据。 遥测数据将通过 Azure Cloud Shell 从 IoT 中心读取。 运行此应用程序前，请先创建 IoT 中心并在中心注册设备。

## <a name="prerequisites"></a>必备条件

* 具有活动订阅的 Azure 帐户。 [免费创建一个](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

* [带 Android SDK 27 的 Android Studio](https://developer.android.com/studio/)。 有关详细信息，请参阅 [Android 安装](https://developer.android.com/studio/install)。 本文中的示例使用 Android SDK 27。

* [一个示例 Android 应用程序](https://github.com/Azure-Samples/azure-iot-samples-java/tree/master/iot-hub/Samples/device/AndroidSample)。 此示例是 [azure-iot-samples-java](https://github.com/Azure-Samples/azure-iot-samples-java) 存储库的一部分。

* 端口 8883 在防火墙中处于打开状态。 本快速入门中的设备示例使用 MQTT 协议，该协议通过端口 8883 进行通信。 在某些公司和教育网络环境中，此端口可能被阻止。 有关解决此问题的更多信息和方法，请参阅[连接到 IoT 中心(MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="add-azure-iot-extension"></a>添加 Azure IoT 扩展

运行以下命令将用于 Azure CLI 的 Microsoft Azure IoT 扩展添加到 Cloud Shell 实例。 IoT 扩展会将 IoT 中心、IoT Edge 和 IoT 设备预配服务 (DPS) 特定的命令添加到 Azure CLI。

```azurecli-interactive
az extension add --name azure-cli-iot-ext
```

## <a name="create-an-iot-hub"></a>创建 IoT 中心

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>注册设备

必须先将设备注册到 IoT 中心，然后该设备才能进行连接。 在本快速入门中，将使用 Azure Cloud Shell 来注册模拟设备。

1. 在 Azure Cloud Shell 中运行以下命令，以创建设备标识。

   **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

   **MyAndroidDevice**：这是所注册的设备的名称。 建议使用 **MyAndroidDevice**，如图所示。 如果为设备选择不同名称，则可能还需要在本文中从头至尾使用该名称，并在运行示例应用程序之前在其中更新设备名称。

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyAndroidDevice
    ```

2. 在 Azure Cloud Shell 中运行以下命令，以获取刚注册设备的_设备连接字符串_：

    **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyAndroidDevice --output table
    ```

    记下如下所示的设备连接字符串：

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}`

    稍后会在本快速入门中使用此值来发送遥测数据。

## <a name="send-simulated-telemetry"></a>发送模拟遥测数据

1. 在 Android Studio 中打开 GitHub 示例 Android 项目。 此项目位于克隆的或下载的 [azure-iot-sample-java](https://github.com/Azure-Samples/azure-iot-samples-java) 存储库副本的以下目录中。

        \azure-iot-samples-java\iot-hub\Samples\device\AndroidSample

2. 在 Android Studio 中打开示例项目的 *gradle.properties*，将 **Device_Connection_String** 占位符替换为此前记下的设备连接字符串。

    ```
    DeviceConnectionString=HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}
    ```

3. 在 Android Studio 中，单击“文件”   >   “将项目与 Gradle 文件同步”。 验证生成是否已完成。

   > [!NOTE]
   > 如果项目同步失败，可能是由于以下某个原因：
   >
   > * 项目中引用的 Android Gradle 插件和 Gradle 的版本对于 Android Studio 版本来说已经过时。 请按照[这些说明](https://developer.android.com/studio/releases/gradle-plugin)操作，以引用并安装正确版本的插件和 Gradle 以进行安装。
   > * Android SDK 的许可协议尚未签署。 请按照 Build 输出中的说明签署许可协议并下载 SDK。

4. 生成完成以后，请单击“运行”   >   “运行‘应用’”。 将应用配置为在物理 Android 设备或 Android 模拟器上运行。 若要详细了解如何在物理设备或模拟器上运行 Android 应用，请参阅[运行您的应用](https://developer.android.com/training/basics/firstapp/running-app)。

5. 待应用加载以后，请单击“启动”按钮，开始将遥测数据发送到 IoT 中心： 

    ![应用程序](media/quickstart-send-telemetry-android/sample-screenshot.png)


## <a name="read-the-telemetry-from-your-hub"></a>从中心读取遥测数据

在本部分，将使用具有 [IoT 扩展](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot?view=azure-cli-latest)的 Azure Cloud Shell 监视 Android 设备发送的设备消息。

1. 通过 Azure Cloud Shell 运行以下命令以建立连接并从 IoT 中心读取消息：

   **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    以下屏幕截图显示了 IoT 中心接收 Android 设备发送的遥测数据后的输出：

      ![使用 Azure CLI 读取设备消息](media/quickstart-send-telemetry-android/read-data.png)
## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>后续步骤

本快速入门设置了 IoT 中心、注册了设备、使用 Android 应用程序发送了模拟遥测数据到中心，并使用 Azure Cloud Shell 读取了中心的遥测数据。

若要了解如何从后端应用程序控制模拟设备，请继续阅读下一快速入门教程。

> [!div class="nextstepaction"]
> [快速入门：控制连接到 IoT 中心的设备](quickstart-control-device-android.md)
