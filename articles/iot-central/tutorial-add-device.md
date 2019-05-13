---
title: 将真实的设备添加到 Azure IoT Central 应用程序 | Microsoft Docs
description: 以操作员的身份将真实设备添加到 Azure IoT Central 应用程序。
author: sandeeppujar
ms.author: sandeepu
ms.date: 04/23/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: peterpr
ms.openlocfilehash: e9378f8d2b31bfed4c464951c427b1e9d00b7893
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64699370"
---
# <a name="tutorial-add-a-real-device-to-your-azure-iot-central-application"></a>教程：将真实设备添加到 Azure IoT Central 应用程序

本教程介绍如何在 Microsoft Azure IoT Central 应用程序中添加和配置真实的设备。

本教程由两个部分组成：

1. 首先，以操作员的身份了解如何在 Azure IoT Central 应用程序中添加和配置真实的设备。 此部分结束时，我们将会检索要在第二部分中使用的连接字符串。
2. 然后，以设备开发人员的身份了解真实设备中的代码。 我们会将第一部分检索到的连接字符串添加到示例代码。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 添加新的真实设备
> * 配置真实设备
> * 从应用程序获取真实设备的连接字符串
> * 了解客户端代码如何映射到应用程序
> * 配置真实设备的客户端代码

## <a name="prerequisites"></a>先决条件

在开始之前，构建人员至少应完成第一篇构建人员教程以创建 Azure IoT Central 应用程序：

* [定义新设备类型](tutorial-define-device-type.md)（必需）
* [配置设备的规则和操作](tutorial-configure-rules.md)（可选）
* [自定义操作员的视图](tutorial-customize-operator.md)（可选）

在开发计算机上安装 [Node.js](https://nodejs.org/) 版本 8.0.0 或更高版本。 若要检查版本，可以在命令行中运行 `node --version`。 Node.js 适用于各种操作系统。

## <a name="add-a-real-device"></a>添加真实设备

若要将真实设备添加到应用程序，请使用[定义新设备类型](tutorial-define-device-type.md)教程中创建的“连接的空调”设备模板。

1. 若要以操作员身份添加新设备，请在左侧导航菜单中选择“Device Explorer”：

   ![显示“连接的空调”的 Device Explorer 页](media/tutorial-add-device/explorer.png)

   **Device Explorer** 显示“连接的空调”设备模板和模拟设备。 当你创建设备模板时，IoT Central 会自动创建模拟设备。

2. 若要开始连接真实的连接空调设备，请依次选择“+”、“真实”：

   ![开始添加新的真实连接空调设备](media/tutorial-add-device/newreal.png)

3. 输入设备 ID（应小写）或使用建议的设备 ID。 也可输入新设备的名称，然后选择“创建”。

   ![重命名设备](media/tutorial-add-device/rename.png)

## <a name="configure-a-real-device"></a>配置真实设备

真实设备是基于“连接的空调”设备模板创建的。 可以使用“设置”来配置设备，并设置属性值来记录有关设备的信息。

1. 在“设置”页上，请注意“设置温度”状态设置为“未更新”。 在真实设备连接到应用程序并确认它已处理该设置之前，设备一直会保持此状态。

    ![设置显示正在同步](media/tutorial-add-device/settingssyncing.png)

2. 在真实的新设备的“属性”页上，服务位置和上次维修日期都是可编辑属性。 在设备连接到应用程序之前，序列号和固件版本字段为空。 这些只读值是从设备发送的，不能进行编辑。

    ![真实设备的设备属性](media/tutorial-add-device/setproperties1.png)

3. 可以查看真实设备的“度量”、“规则”和“仪表板”页。

## <a name="generate-connection-string"></a>生成连接字符串

设备开发人员需在设备上运行的代码中嵌入真实设备的连接字符串。 该连接字符串可让设备安全连接到应用程序。 以下步骤演示如何生成连接字符串并准备客户端 Node.js 代码。

## <a name="prepare-the-client-code"></a>准备客户端代码

本文中的示例代码是以 [Node.js](https://nodejs.org/) 编写的，演示了如何通过足够的代码执行以下操作：

* 以设备的形式连接到 Azure IoT Central 应用程序。
* 以连接的空调设备形式发送温度遥测数据。
* 将设备属性发送到 Azure IoT Central 应用程序。
* 对使用“设置温度”设置的操作员做出响应。
* 处理从 Azure IoT Central 应用程序发出的回显命令。

[后续步骤](#next-steps)部分列出的文章包含更多完整示例，并介绍了其他编程语言。 有关设备如何连接到 Azure IoT Central 的详细信息，请参阅[设备连接](concepts-connectivity.md)一文。

以下步骤说明如何准备 [Node.js](https://nodejs.org/) 示例：

### <a name="get-the-device-connection-information"></a>获取设备连接信息

1. 应用程序中设备实例的连接字符串是根据 IoT Central 提供的设备信息生成的。

   在真实的已连接空调的设备屏幕上，选择“连接”。

   ![显示“查看连接信息”链接的“设备”页](media/tutorial-add-device/connectionlink.png)

1. 在“设备连接”页上，记下**作用域 ID**、**设备 ID** 和**主要密钥**值。 下一步使用这些值。

   ![连接详细信息](media/tutorial-add-device/device-connect.png)

### <a name="generate-the-connection-string"></a>生成连接字符串

[!INCLUDE [iot-central-howto-connection-string](../../includes/iot-central-howto-connection-string.md)]

### <a name="prepare-the-nodejs-project"></a>准备 Node.js 项目

1. 在开发计算机上，创建名为 `connectedairconditioner` 的文件夹。

1. 在命令行环境中，导航到创建的 `connectedairconditioner` 文件夹。

1. 若要初始化 Node.js 项目，请运行以下命令并接受所有默认值：

    ```cmd/sh
    npm init
      ```

1. 若要安装所需的包，请运行以下命令：

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

1. 使用文本编辑器在 `connectedairconditioner` 文件夹中创建名为 **ConnectedAirConditioner.js** 的文件。

1. 在 **ConnectedAirConditioner.js** 文件的开头添加以下 `require` 语句：

    ```javascript
    'use strict';

    var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    var ConnectionString = require('azure-iot-device').ConnectionString;
    ```

1. 将以下变量声明添加到该文件：

    ```javascript
    var connectionString = '{your device connection string}';
    var targetTemperature = 0;
    var client = clientFromConnectionString(connectionString);
    ```

    > [!NOTE]
    > 在稍后的步骤中，需要更新占位符 `{your device connection string}`。

1. 保存到目前为止所做的更改，但请将文件保持打开状态。

## <a name="review-client-code"></a>评审客户端代码

在上一部分，我们为某个应用程序创建了一个要与 Azure IoT Central 应用程序相连接的主干 Node.js 项目。 下一步是添加用于执行以下操作的代码：

* 连接到 Azure IoT Central 应用程序。
* 将遥测数据发送到 Azure IoT Central 应用程序。
* 将设备属性发送到 Azure IoT Central 应用程序。
* 从 Azure IoT Central 应用程序接收设置。
* 处理从 Azure IoT Central 应用程序发出的回显命令。

1. 若要将温度遥测数据发送到 Azure IoT Central 应用程序，请将以下代码添加到 **ConnectedAirConditioner.js** 文件：

    ```javascript
    // Send device telemetry.
    function sendTelemetry() {
      var temperature = targetTemperature + (Math.random() * 15);
      var data = JSON.stringify({ temperature: temperature });
      var message = new Message(data);
      client.sendEvent(message, (err, res) => console.log(`Sent message: ${message.getData()}` +
        (err ? `; error: ${err.toString()}` : '') +
        (res ? `; status: ${res.constructor.name}` : '')));
    }
    ```

    要发送的 JSON 中的字段名称必须与在设备模板中为温度遥测指定的字段名称匹配。 在此示例中，字段名称为 **temperature**。

1. 若要发送设备属性（例如 **firmwareVersion** 和 **serialNumber**），请添加以下定义：

    ```javascript
    // Send device properties
    function sendDeviceProperties(twin) {
      var properties = {
        firmwareVersion: "9.75",
        serialNumber: "10001"
      };
      twin.properties.reported.update(properties, (errorMessage) => 
      console.log(` * Sent device properties ` + (errorMessage ? `Error: ${errorMessage.toString()}` : `(success)`)));
    }
    ```

1. 若要定义设备支持的设置（例如 **setTemperature**），请添加以下定义：

    ```javascript
    // Add any settings your device supports
    // mapped to a function that is called when the setting is changed.
    var settings = {
      'setTemperature': (newValue, callback) => {
        // Simulate the temperature setting taking two steps.
        setTimeout(() => {
          targetTemperature = targetTemperature + (newValue - targetTemperature) / 2;
          callback(targetTemperature, 'pending');
          setTimeout(() => {
            targetTemperature = newValue;
            callback(targetTemperature, 'completed');
          }, 5000);
        }, 5000);
      }
    };
    ```

1. 若要处理 Azure IoT Central 发来的设置，请添加以下函数，用于查找并执行相应的设备代码：

    ```javascript
    // Handle settings changes that come from Azure IoT Central via the device twin.
    function handleSettings(twin) {
      twin.on('properties.desired', function (desiredChange) {
        for (let setting in desiredChange) {
          if (settings[setting]) {
            console.log(`Received setting: ${setting}: ${desiredChange[setting].value}`);
            settings[setting](desiredChange[setting].value, (newValue, status, message) => {
              var patch = {
                [setting]: {
                  value: newValue,
                  status: status,
                  desiredVersion: desiredChange.$version,
                  message: message
                }
              }
              twin.properties.reported.update(patch, (err) => console.log(`Sent setting update for ${setting}; ` +
                (err ? `error: ${err.toString()}` : `status: success`)));
            });
          }
        }
      });
    }
    ```

    此函数将会：

    * 监视 Azure IoT Central 发送所需属性的过程。
    * 查找为了处理设置更改而要调用的相应函数。
    * 将确认发回到 Azure IoT Central 应用程序。

1. 若要响应命令，例如来自 Azure IoT Central 应用程序的 **echo**，请添加以下定义：

    ```javascript
    // Respond to the echo command
    function onCommandEcho(request, response) {
      // Display console info
      console.log(' * Echo command received');
      // Respond
      response.send(10, 'Success', function (errorMessage) {});
    }
    ```

1. 添加以下代码以完成与 Azure IoT Central 的连接，并在客户端代码中挂接函数：

    ```javascript
    // Handle device connection to Azure IoT Central.
    var connectCallback = (err) => {
      if (err) {
        console.log(`Device could not connect to Azure IoT Central: ${err.toString()}`);
      } else {
        console.log('Device successfully connected to Azure IoT Central');
        // Send telemetry measurements to Azure IoT Central every 1 second.
        setInterval(sendTelemetry, 1000);
        // Setup device command callbacks
        client.onDeviceMethod('echo', onCommandEcho);
        // Get device twin from Azure IoT Central.
        client.getTwin((err, twin) => {
          if (err) {
            console.log(`Error getting device twin: ${err.toString()}`);
          } else {
            // Send device properties once on device start up
            sendDeviceProperties(twin);
            // Apply device settings and handle changes to device settings.
            handleSettings(twin);
          }
        });
      }
    };

    client.open(connectCallback);
    ```

1. 保存到目前为止所做的更改，但请将文件保持打开状态。

## <a name="configure-client-code"></a>配置客户端代码

<!-- Add the connection string to the sample code, build, and run -->
若要配置客户端代码以连接到 Azure IoT Central 应用程序，需要添加本教程前面记下的真实设备连接字符串。

1. 在 **ConnectedAirConditioner.js** 文件中找到以下代码行：

    ```javascript
    var connectionString = '{your device connection string}';
    ```

1. 将 `{your device connection string}` 替换为真实设备的连接字符串。 你复制了在上一步中生成的连接字符串。

1. 保存对 **ConnectedAirConditioner.js** 文件所做的更改。

1. 若要运行示例，请在命令行环境中输入以下命令：

    ```cmd/sh
    node ConnectedAirConditioner.js
    ```

    > [!NOTE]
    > 运行此命令时，请确保在 `connectedairconditioner` 文件夹中操作。

1. 应用程序会在控制台上列显输出：

   ![客户端应用程序输出](media/tutorial-add-device/output.png)

1. 大约 30 秒后，设备的“度量”页上会显示遥测数据：

   ![真实遥测数据](media/tutorial-add-device/realtelemetry.png)

1. 在“设置”页上，可以看到设置现已同步。 首次连接设备后，该设备会收到设置值，并确认更改：

   ![设置已同步](media/tutorial-add-device/settingsynced.png)

1. 在“设置”页上，将设备温度设置为 **95**，并选择“更新设备”。 示例应用程序会接收并处理此更改：

   ![接收和处理设置](media/tutorial-add-device/receivesetting.png)

   > [!NOTE]
   > 有两条“设置更新”消息。 其中一条消息是处于 `pending` 状态时发送的，另一条是处于 `completed` 状态时发送的。

1. 在“度量”页上可以看到，设备正在发送较高的温度值：

    ![温度遥测数据现在较高](media/tutorial-add-device/highertemperature.png)

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="nextstepaction"]
> * 添加新的真实设备
> * 配置新设备
> * 从应用程序获取真实设备的连接字符串
> * 了解客户端代码如何映射到应用程序
> * 配置真实设备的客户端代码

将真实设备连接到 Azure IoT Central 应用程序后，建议接下来执行以下后续步骤：

以操作员身份了解如何：

* [管理设备](howto-manage-devices.md)
* [使用设备集](howto-use-device-sets.md)
* [创建自定义分析](howto-use-device-sets.md)

以设备开发人员身份了解如何：

* [准备和连接 DevKit 设备 (C)](howto-connect-devkit.md)
* [准备和连接 Raspberry Pi (Python)](howto-connect-raspberry-pi-python.md)
* [准备和连接 Raspberry Pi (C#)](howto-connect-raspberry-pi-csharp.md)
* [准备和连接 Windows 10 IoT Core 设备 (C#)](howto-connect-windowsiotcore.md)
* [将泛型 Node.js 客户端连接到 Azure IoT Central 应用程序](howto-connect-nodejs.md)
