---
title: 从 Azure IoT 中心控制设备快速入门 (.NET) | Microsoft Docs
description: 在本快速入门中，会运行两个示例 C# 应用程序。 一个为后端应用程序，可远程控制连接到中心的设备。 另一个应用程序可模拟连接到中心的可受远程控制的设备。
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: quickstart
ms.custom: mvc
ms.date: 06/21/2019
ms.openlocfilehash: 3e01f02026bfd788c398e580a78812d1a69f8f1d
ms.sourcegitcommit: 9add86fb5cc19edf0b8cd2f42aeea5772511810c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/09/2020
ms.locfileid: "77108948"
---
# <a name="quickstart-control-a-device-connected-to-an-iot-hub-net"></a>快速入门：控制连接到 IoT 中心的设备 (.NET)

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

IoT 中心是一项 Azure 服务，使你可以从云管理 IoT 设备，并将大量设备遥测引入云以进行存储或处理。 在本快速入门中，会使用直接方法来控制连接到 IoT 中心的模拟设备  。 可使用直接方法远程更改连接到 IoT 中心的设备的行为。

本快速入门使用两个预先编写的 .NET 应用程序：

* 从后端应用程序调用的可响应直接方法的模拟设备应用程序。 为了接收直接方法调用，此应用程序会连接到 IoT 中心上特定于设备的终结点。

* 后端应用程序，可在模拟设备上调用直接方法。 为了在设备上调用直接方法，此应用程序会连接到 IoT 中心上的服务端终结点。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>必备条件

本快速入门中运行的两个示例应用程序是使用 C# 编写的。 开发计算机上需要有 .NET Core SDK 2.1.0 或更高版本。

可以从 [.NET](https://www.microsoft.com/net/download/all) 为多个平台下载 .NET Core SDK。

可以使用以下命令验证开发计算机上 C# 的当前版本：

```cmd/sh
dotnet --version
```

运行以下命令将用于 Azure CLI 的 Microsoft Azure IoT 扩展添加到 Cloud Shell 实例。 IoT 扩展会将 IoT 中心、IoT Edge 和 IoT 设备预配服务 (DPS) 特定的命令添加到 Azure CLI。

```azurecli-interactive
az extension add --name azure-cli-iot-ext
```

如果尚未进行此操作，请从 https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip 下载 Azure IoT C# 示例并提取 ZIP 存档。

确保已在防火墙中打开端口 8883。 本快速入门中的设备示例使用 MQTT 协议，该协议通过端口 8883 进行通信。 在某些公司和教育网络环境中，此端口可能被阻止。 有关解决此问题的更多信息和方法，请参阅[连接到 IoT 中心(MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub)。

## <a name="create-an-iot-hub"></a>创建 IoT 中心

如果已完成上一[快速入门：将遥测数据从设备发送到 IoT 中心](quickstart-send-telemetry-dotnet.md)，则可以跳过此步骤。

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>注册设备

如果已完成上一[快速入门：将遥测数据从设备发送到 IoT 中心](quickstart-send-telemetry-dotnet.md)，则可以跳过此步骤。

必须先将设备注册到 IoT 中心，然后该设备才能进行连接。 在本快速入门中，将使用 Azure Cloud Shell 来注册模拟设备。

1. 在 Azure Cloud Shell 中运行以下命令，以创建设备标识。

   **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

   **MyDotnetDevice**：这是所注册的设备的名称。 建议使用 **MyDotnetDevice**，如图所示。 如果为设备选择不同名称，则可能还需要在本文中从头至尾使用该名称，并在运行示例应用程序之前在其中更新设备名称。

    ```azurecli-interactive
    az iot hub device-identity create \
      --hub-name {YourIoTHubName} --device-id MyDotnetDevice
    ```

2. 在 Azure Cloud Shell 中运行以下命令，以获取刚注册设备的_设备连接字符串_：

   **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string \
      --hub-name {YourIoTHubName} \
      --device-id MyDotnetDevice \
      --output table
    ```

    记下如下所示的设备连接字符串：

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    稍后会在快速入门中用到此值。

## <a name="retrieve-the-service-connection-string"></a>检索服务连接字符串

还需一个 IoT 中心服务连接字符串，以便后端应用程序能够连接到中心并检索消息  。 以下命令检索 IoT 中心的服务连接字符串：

```azurecli-interactive
az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
```

记下如下所示的服务连接字符串：

   `HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

稍后会在快速入门中用到此值。 此服务连接字符串与你在上一步中记录的设备连接字符串不同。

## <a name="listen-for-direct-method-calls"></a>侦听直接方法调用

模拟设备应用程序会连接到 IoT 中心上特定于设备的终结点，发送模拟遥测数据，并侦听中心的直接方法调用。 在本快速入门中，中心的直接方法调用告知设备对其发送遥测的间隔进行更改。 执行直接方法后，模拟设备会将确认发送回中心。

1. 在本地终端窗口中，导航到示例 C# 项目的根文件夹。 然后导航到 **iot-hub\Quickstarts\simulated-device-2** 文件夹。

2. 在所选文本编辑器中打开 SimulatedDevice.cs 文件  。

    将 `s_connectionString` 变量的值替换为之前记下的设备连接字符串。 然后将更改保存到 **SimulatedDevice.cs**。

3. 在本地终端窗口中，运行以下命令以安装模拟设备应用程序所需的包：

    ```cmd/sh
    dotnet restore
    ```

4. 在本地终端窗口中，运行以下命令，生成并运行模拟设备应用程序：

    ```cmd/sh
    dotnet run
    ```

    以下屏幕截图显示了模拟设备应用程序将遥测数据发送到 IoT 中心后的输出：

    ![运行模拟设备](./media/quickstart-control-device-dotnet/SimulatedDevice-1.png)

## <a name="call-the-direct-method"></a>调用直接方法

后端应用程序会连接到 IoT 中心上的服务端终结点。 应用程序通过 IoT 中心对设备进行直接方法调用，并侦听确认。 IoT 中心后端应用程序通常在云中运行。

1. 在另一本地终端窗口中，导航到示例 C# 项目的根文件夹。 然后导航到 iot-hub\Quickstarts\back-end-application 文件夹  。

2. 在所选文本编辑器中打开 BackEndApplication.cs 文件  。

    将 `s_connectionString` 变量的值替换为以前记下的服务连接字符串。 然后将更改保存到 **BackEndApplication.cs**。

3. 在本地终端窗口中，运行以下命令，安装后端应用程序所需的库：

    ```cmd/sh
    dotnet restore
    ```

4. 在本地终端窗口中，运行以下命令，生成并运行后端应用程序：

    ```cmd/sh
    dotnet run
    ```

    以下屏幕截图显示了应用程序对设备进行直接方法调用并接收确认后的输出：

    ![运行后端应用程序](./media/quickstart-control-device-dotnet/BackEndApplication.png)

    运行后端应用程序后，在运行模拟设备的控制台窗口中会出现一条消息，且其发送消息的速率也会发生变化：

    ![模拟客户端的变化](./media/quickstart-control-device-dotnet/SimulatedDevice-2.png)

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门中，已从后端应用程序调用了设备上的直接方法，并在模拟设备应用程序中响应了直接方法调用。

若要了解如何将设备到云的消息路由到云中的不同目标，请继续学习下一教程。

> [!div class="nextstepaction"]
> [教程：将遥测路由到不同的终结点进行处理](tutorial-routing.md)
