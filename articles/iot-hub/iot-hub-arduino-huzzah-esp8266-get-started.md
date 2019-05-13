---
title: ESP8266 到云 - 将 Feather HUZZAH ESP8266 连接到 Azure IoT 中心 | Microsoft Docs
description: 在本教程中了解如何设置 Adafruit Feather HUZZAH ESP8266 并将其连接到 Azure IoT 中心，使其能够将数据发送到 Azure 云平台。
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/11/2018
ms.author: wesmc
ms.openlocfilehash: 00d0e3a749c4af8faacfed5f725ce16e0492c399
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409078"
---
# <a name="connect-adafruit-feather-huzzah-esp8266-to-azure-iot-hub-in-the-cloud"></a>将 Adafruit Feather HUZZAH ESP8266 连接到云中的 Azure IoT 中心

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

![DHT22、Feather HUZZAH ESP8266 与 IoT 中心之间的连接](./media/iot-hub-arduino-huzzah-esp8266-get-started/1_connection-hdt22-feather-huzzah-iot-hub.png)

## <a name="what-you-do"></a>准备工作

将 Adafruit Feather HUZZAH ESP8266 连接到创建的 Azure IoT 中心。 然后，在 ESP8266 上运行一个示例应用程序，用于从 DHT22 传感器收集温度和湿度数据。 最后，将传感器数据发送到 IoT 中心。

> [!NOTE]
> 如果使用其他 ESP8266 开发板，仍可使用这些步骤将其连接到 IoT 中心。 根据所用的 ESP8266 开发板，可能需要重新配置 `LED_PIN`。 例如，如果使用 AI-Thinker 提供的 ESP8266，可将此参数从 `0` 更改为 `2`。 还没有工具包？ 从 [Azure 网站](https://azure.com/iotstarterkits)获取。

## <a name="what-you-learn"></a>学习内容

* 如何创建 IoT 中心以及注册 Feather HUZZAH ESP8266 的设备
* 如何将 Feather HUZZAH ESP8266 与传感器和计算机相连接
* 如何在 Feather HUZZAH ESP8266 上运行示例应用程序来收集传感器数据
* 如何将传感器数据发送到 IoT 中心

## <a name="what-you-need"></a>所需条件

![本教程所需的部件](./media/iot-hub-arduino-huzzah-esp8266-get-started/2_parts-needed-for-the-tutorial.png)

若要完成此操作，需要使用 Feather HUZZAH ESP8266 初学者工具包中的以下部件：

* Feather HUZZAH ESP8266 开发板
* Micro USB 转 Type A USB 线缆

还需要为开发环境做好以下准备：

* 一个有效的 Azure 订阅。 如果没有 Azure 帐户，只需几分钟时间就能[创建一个免费的 Azure 试用帐户](https://azure.microsoft.com/free/)。
* 运行 Windows 或 Ubuntu 的 Mac 或 PC。
* Feather HUZZAH ESP8266 要连接到的无线网络。
* 建立 Internet 连接，以便下载配置工具。
* [适用于 Arduino 的 Visual Studio Code 扩展](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-arduino)。

> [!Note]
> 适用于 Arduino 的 Visual Studio Code 扩展使用的 Arduino IDE 版本必须是 1.6.8 或更高版本。 早期版本不适用于 AzureIoT 库。

如果没有传感器，以下各项是可选的。 也可以使用模拟的传感器数据。

* Adafruit DHT22 温度和湿度传感器
* 试验板
* M/M 跳线

## <a name="create-an-iot-hub"></a>创建 IoT 中心

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

### <a name="retrieve-connection-string-for-iot-hub"></a>检索 IoT 中心的连接字符串

[!INCLUDE [iot-hub-include-find-connection-string](../../includes/iot-hub-include-find-connection-string.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>在 IoT 中心内注册新设备

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="connect-feather-huzzah-esp8266-with-the-sensor-and-your-computer"></a>将 Feather HUZZAH ESP8266 与传感器和计算机相连接

在此部分中，将传感器连接到开发板。 然后，将设备插入计算机以作后用。

### <a name="connect-a-dht22-temperature-and-humidity-sensor-to-feather-huzzah-esp8266"></a>将 DHT22 温度和湿度传感器连接到 Feather HUZZAH ESP8266

按如下所示，使用试验板和跳线建立连接。 如果没有传感器，请跳过本部分，因为可以改用模拟的传感器数据。

![连接参考](./media/iot-hub-arduino-huzzah-esp8266-get-started/17_connections_on_breadboard.png)

对于传感器引脚，请使用以下接线：

| 启动（传感器）           | 结束（开发板）            | 线缆颜色   |
| -----------------------  | ---------------------- | ------------  |
| VDD（引脚 31F）            | 3V（引脚 58H）           | 红线     |
| DATA（引脚 32F）           | GPIO 2（引脚 46A）       | 蓝线    |
| GND（引脚 34F）            | GND（引脚 56I）          | 黑线   |

有关详细信息，请参阅 [Adafruit DHT22 传感器设置](https://learn.adafruit.com/dht/connecting-to-a-dhtxx-sensor)和 [Adafruit Feather HUZZAH Esp8266 引出线](https://learn.adafruit.com/adafruit-feather-huzzah-esp8266/using-arduino-ide?view=all#pinouts)。

现在，Feather Huzzah ESP8266 应已连接到正常工作的传感器。

![将 DHT22 与 Feather Huzzah 相连接](media/iot-hub-arduino-huzzah-esp8266-get-started/8_connect-dht22-feather-huzzah.png)

### <a name="connect-feather-huzzah-esp8266-to-your-computer"></a>将 Feather HUZZAH ESP8266 连接到计算机

按如下所示，使用 Micro USB 转 Type A USB 线缆将 Feather HUZZAH ESP8266 连接到计算机。

![将 Feather Huzzah 连接到计算机](media/iot-hub-arduino-huzzah-esp8266-get-started/9_connect-feather-huzzah-computer.png)

### <a name="add-serial-port-permissions-ubuntu-only"></a>添加串行端口权限（仅适用于 Ubuntu）

如果使用 Ubuntu，请确保有权在 Feather HUZZAH ESP8266 的 USB 端口上操作。 若要添加串行端口权限，请执行以下步骤：

1. 在终端中运行以下命令：

   ```bash
   ls -l /dev/ttyUSB*
   ls -l /dev/ttyACM*
   ```

   将返回以下输出之一：

   * crw-rw---- 1 root uucp xxxxxxxx
   * crw-rw---- 1 root dialout xxxxxxxx

   请注意，在输出中，`uucp` 或 `dialout` 是 USB 端口的组所有者名称。

2. 运行以下命令，将用户添加到该组中：

   ```bash
   sudo usermod -a -G <group-owner-name> <username>
   ```

   `<group-owner-name>` 是在上一步骤中获取的组所有者名称。 `<username>` 是 Ubuntu 用户名。

3. 需要注销 Ubuntu，并重新登录，更改才会显示。

## <a name="collect-sensor-data-and-send-it-to-your-iot-hub"></a>收集传感器数据并将其发送到 IoT 中心

在本部分，你会在 Feather HUZZAH ESP8266 上部署并运行一个示例应用程序。 该示例应用程序会使 Feather HUZZAH ESP8266 上的 LED 闪烁，并将从 DHT22 传感器收集的温度和湿度数据发送到 IoT 中心。

### <a name="get-the-sample-application-from-github"></a>从 GitHub 获取示例应用程序

该示例应用程序托管在 GitHub 中。 从 GitHub 中克隆包含该示例应用程序的示例存储库。 若要克隆示例存储库，请执行以下步骤：

1. 打开命令提示符或终端窗口。

2. 转到用于存储示例应用程序的文件夹。

3. 运行以下命令：

   ```bash
   git clone https://github.com/Azure-Samples/iot-hub-feather-huzzah-client-app.git
   ```

   接下来，在 Visual Studio Code 中安装 Feather HUZZAH ESP8266 的包。

4. 打开存储示例应用程序的文件夹。

5. 在 Visual Studio Code 中打开 app 文件夹中的 app.ino 文件。

   ![在 Visual Studio Code 中打开示例应用程序](media/iot-hub-arduino-huzzah-esp8266-get-started/10_vscode-open-sample-app.png)

6. 在 Visual Studio Code 中，输入 `F1`。

7. 类型**Arduino** ，然后选择**arduino 开发：Board Manager**。

8. 在“Arduino 开发板管理器”选项卡中，单击“其他 URL”。

   ![VS Code Arduino 开发板管理器](media/iot-hub-arduino-huzzah-esp8266-get-started/11_vscode-arduino-board-manager.png)

9. 在“用户设置”窗口，在文件末尾复制并粘贴以下内容

   ```json
   "arduino.additionalUrls": "https://arduino.esp8266.com/stable/package_esp8266com_index.json"
   ```

   ![在 VS Code 中配置 Arduino 包 URL](media/iot-hub-arduino-huzzah-esp8266-get-started/12_vscode-package-url.png)

10. 保存文件并关闭“用户设置”选项卡。

11. 单击“刷新包索引”。 完成刷新后，搜索 esp8266。

12. 单击 esp8266 的“安装”按钮。

    Boards Manager 指示 ESP8266 安装了版本 2.2.0 或更高版本。

    ![已安装 esp8266 包](media/iot-hub-arduino-huzzah-esp8266-get-started/13_vscode-esp8266-installed.png)

13. 输入`F1`，然后键入**Arduino** ，然后选择**arduino 开发：板配置**。

14. 单击“所选开发板:”旁边的框，键入“esp8266”，然后选择“Adafruit HUZZAH ESP8266 (esp8266)”。

    ![选择 esp8266 开发板](media/iot-hub-arduino-huzzah-esp8266-get-started/14_vscode-select-esp8266.png)

### <a name="install-necessary-libraries"></a>安装所需的库

1. 在 Visual Studio Code 中，输入`F1`，然后键入**Arduino** ，然后选择**arduino 开发：库管理器**。

2. 逐个搜索以下库名称。 对于找到的每个库，单击“安装”。
   * `AzureIoTHub`
   * `AzureIoTUtility`
   * `AzureIoTProtocol_MQTT`
   * `ArduinoJson`
   * `DHT sensor library`
   * `Adafruit Unified Sensor`

### <a name="dont-have-a-real-dht22-sensor"></a>没有真正的 DHT22 传感器？

如果没有真正的 DHT22 传感器，示例应用程序可以模拟温度和湿度数据。 若要设置示例应用程序以使用模拟的数据，请执行以下步骤：

1. 打开 `app` 文件夹中的 `config.h` 文件。

2. 找到以下代码行并将值从 `false` 更改为 `true`：

   ```c
   define SIMULATED_DATA true
   ```

   ![将示例应用程序配置为使用模拟数据](media/iot-hub-arduino-huzzah-esp8266-get-started/15_vscode-configure-app-use-simulated-data.png)

3. 保存文件。

### <a name="deploy-the-sample-application-to-feather-huzzah-esp8266"></a>将示例应用程序部署到 Feather HUZZAH ESP8266

1. 在 Visual Studio Code 中，单击**\<选择串行端口 >** 状态栏，并单击 Feather HUZZAH ESP8266 的串行端口。

2. 输入`F1`，然后键入**Arduino** ，然后选择**arduino 开发：上传**生成和部署到 Feather HUZZAH ESP8266 的示例应用程序。

### <a name="enter-your-credentials"></a>输入您的凭据

上传成功完成后，按照以下步骤输入凭据：

1. 打开 Arduino IDE，单击“工具” > “串行监视器”。

2. 在串行监视器窗口的右下角，可以看到两个下拉列表。

3. 在左侧下拉列表中选择“No line ending”（无行尾）。

4. 在右侧下拉列表中选择“115200 baud”（115200 波特率）。

5. 在串行监视器窗口顶部的输入框中输入以下信息（如果系统要求提供），并单击“Send”（发送）。

   * Wi-Fi SSID
   * Wi-Fi 密码
   * 设备连接字符串

> [!Note]
> 凭据信息将存储在 Feather HUZZAH ESP8266 的 EEPROM 中。 如果在 Feather HUZZAH ESP8266 开发板上单击重置按钮，示例应用程序将询问是否要擦除这些信息。 输入 `Y` 以擦除这些信息。 按提示再次提供这些信息。

### <a name="verify-the-sample-application-is-running-successfully"></a>验证示例应用程序是否成功运行

如果串行监视器窗口中显示以下输出并且 Feather HUZZAH ESP8266 上的 LED 闪烁，则表示示例应用程序已成功运行。

![Arduino IDE 中的最终输出](media/iot-hub-arduino-huzzah-esp8266-get-started/16_arduino-ide-final-output.png)

## <a name="read-the-messages-received-by-your-hub"></a>读取中心收到的消息

若要监视 IoT 中心从设备收到的消息的一种方法是使用用于 Visual Studio Code 的 Azure IoT 工具。 若要了解详细信息，请参阅[使用针对 Visual Studio Code 进行发送和接收消息在设备与 IoT 中心之间的 Azure IoT 工具](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md)。

有关更多方法来处理设备发送的数据，继续学习下一节。

## <a name="next-steps"></a>后续步骤

现已成功将 Feather HUZZAH ESP8266 连接到 IoT 中心，并将捕获的传感器数据发送到了 IoT 中心。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]