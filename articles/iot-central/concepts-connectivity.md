---
title: Azure IoT Central 中的设备连接 | Microsoft Docs
description: 本文介绍与 Azure IoT Central 中的设备连接相关的重要概念
author: dominicbetts
ms.author: dobett
ms.date: 04/09/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: timlt
ms.openlocfilehash: f11d1c6b727799f2cc67c6f5012dc4d117556d2e
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467809"
---
# <a name="device-connectivity-in-azure-iot-central"></a>Azure IoT Central 中的设备连接

本文介绍与 Microsoft Azure IoT Central 中的设备连接相关的重要概念。

使用 azure IoT Central [Azure IoT 中心设备预配服务 (DPS)](https://docs.microsoft.com/azure/iot-dps/about-iot-dps)来管理所有设备注册和连接。

使用分发点启用：

- 若要支持载入和连接设备进行大规模的 IoT 中心。
- 你可以生成设备凭据和无需注册的设备通过 IoT Central UI 配置设备脱机。
- 设备能够连接使用共享访问签名 (SAS)。
- 使用行业标准 X.509 证书进行连接的设备。
- 您可以使用您自己的设备 Id 在 IoT 中心注册设备。 使用您自己的设备 Id 可以简化与现有后端办公系统集成。
- 用于将设备连接到 IoT 中心的单个、 一致方法。

本文介绍了以下四个用例：

1. [快速连接单个设备使用 SAS](#connect-a-single-device)
1. [连接设备进行大规模使用 SAS](#connect-devices-at-scale-using-sas)
1. [连接设备进行大规模使用 X.509 证书](#connect-devices-using-x509-certificates)这是推荐用于生产环境的方法。
1. [直接连接而无需首先注册设备](#connect-without-registering-devices)

## <a name="connect-a-single-device"></a>连接单个设备

通过 IoT Central 试用或测试设备时，此方法非常有用。 可以从 IoT 中心应用程序使用的设备连接信息以生成设备的连接字符串。 有关详细步骤，请参阅[如何生成一个设备连接字符串以连接到 Azure IoT Central 应用程序](howto-generate-connection-string.md)。

## <a name="connect-devices-at-scale-using-sas"></a>连接设备进行大规模使用 SAS

若要大规模使用 SAS 连接到 IoT 中心的设备，需要注册，然后设置设备：

### <a name="register-devices-in-bulk"></a>批量注册设备

若要注册大量设备与 IoT Central 应用程序，使用 CSV 文件[导入设备 Id 和设备名称](howto-manage-devices.md#import-devices)。

若要检索的导入设备的连接信息[从 IoT 中心应用程序中导出 CSV 文件](howto-manage-devices.md#export-devices)。

> [!NOTE]
> 若要了解如何注册的情况下第一个 IoT Central 中的连接的设备，请参阅[第一个设备注册不连接](#connect-without-registering-devices)。

### <a name="set-up-your-devices"></a>设置你的设备

使用从设备代码中的导出文件的连接信息以使设备能够连接并将数据发送到 IoT，IoT Central 应用程序。 有关将设备连接的详细信息，请参阅[后续步骤](#next-steps)。

## <a name="connect-devices-using-x509-certificates"></a>使用 X.509 证书的设备连接

在生产环境中，使用 X.509 证书是 IoT 中心的建议的设备身份验证机制。 有关详细信息，请参阅[使用 X.509 CA 证书进行设备身份验证](../iot-hub/iot-hub-x509ca-overview.md)。

以下步骤介绍如何将设备连接到 IoT 中心使用 X.509 证书：

1. IoT Central 应用程序中，在_添加和验证中间或根的 X.509 证书_你要用于生成设备证书：

    - 导航到**管理 > 设备连接 > 证书 (X.509)** 并添加 X.509 根或中间证书，你要用于生成叶设备证书。

      ![连接设置](media/concepts-connectivity/connection-settings.png)

      如果你有安全漏洞或主要证书设置为过期，请使用辅助证书为了减少停机时间。 你可以继续使用辅助证书，而更新的主证书的设备预配。

    - 验证证书所有权可以确保证书上的传者具有证书的私钥。 若要验证的证书：
        - 选择的按钮旁边**验证码**生成代码。
        - 创建 X.509 验证证书与上一步中生成的验证码。 将证书保存为.cer 文件。
        - 上传已签名的验证证书并选择**验证**。

          ![连接设置](media/concepts-connectivity/verify-cert.png)

1. 使用 CSV 文件_导入并注册设备_IoT Central 应用程序中。

1. _设置你的设备。_ 使用上传的根证书生成叶证书。 使用**设备 ID**为叶证书中的 CNAME 值。 设备 ID 应为全小写。 然后程序你的设备预配服务的信息。 当设备打开第一个问题时，从分发点检索 IoT Central 应用程序及其连接信息。

### <a name="further-reference"></a>进一步引用

- [RaspberryPi](https://aka.ms/iotcentral-docs-Raspi-releases) 的示例实现。

- [C 编写的示例设备客户端](https://github.com/Azure/azure-iot-sdk-c/blob/dps_symm_key/provisioning_client/devdoc/using_provisioning_client.md)

### <a name="for-testing-purposes-only"></a>仅出于测试目的

仅用于测试，可以使用这些实用程序生成的 CA 证书和设备证书。

- 如果你正在使用 DevKit 设备，这[命令行工具](https://aka.ms/iotcentral-docs-dicetool)生成可以将添加到你的 IoT Central 应用程序来验证证书的 CA 证书。

- 使用此[命令行工具](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md )到：
  - 创建证书链。 按照步骤 2 中 GitHub 文章。
  - 将证书另存为.cer 文件上传到 IoT 中心应用程序。
  - 使用 IoT Central 应用程序的验证代码生成的验证证书。 按照步骤 3 中 GitHub 文章。
  - 创建你的设备使用设备 Id 作为该工具的参数的叶证书。 按照步骤 4 中 GitHub 文章。

## <a name="connect-without-registering-devices"></a>连接而无需注册设备

IoT 中心启用的关键方案是 oem 可以批量制造设备可以连接到 IoT Central 应用程序而不必首先注册。 制造商必须生成合适的凭据，并在工厂中配置设备。 当第一次打开设备时，它将自动连接到 IoT 中心应用程序。 IoT Central 运算符必须批准设备才可以 stat 发送数据。

下图概述了此流：

![连接设置](media/concepts-connectivity/device-connection-flow1.png)

以下步骤介绍此过程的更多详细信息。 具体取决于是否要使用 SAS 或 X.509 证书进行设备身份验证，步骤稍有不同：

1. 配置的连接设置：

    - **X.509 证书：**[添加和验证根/中间证书](#connect-devices-using-x509-certificates)并使用它在以下步骤中生成设备证书。
    - **SAS：** 复制主密钥。 此密钥是 IoT 中心应用程序组 SAS 密钥。 使用密钥生成下一个步骤中的设备的 SAS 密钥。
    ![连接设置 SAS](media/concepts-connectivity/connection-settings-sas.png)

1. 生成你的设备凭据
    - **X.509 证书中：** 生成你使用的根或中间证书添加到 IoT 中心应用程序的设备叶证书。 请确保使用小写**设备 ID**为叶证书中的 CNAME。 对于测试目的，使用这[命令行工具](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md )生成设备证书。
    - **SAS：** 使用此[命令行工具](https://www.npmjs.com/package/dps-keygen)生成设备的 SAS 密钥。 使用组**Primary Key**从上一步。 设备 ID 必须为小写。

      若要安装[密钥生成器实用程序](https://github.com/Azure/dps-keygen)，运行以下命令：

      ```cmd/sh
      npm i -g dps-keygen
      ```

      若要从组 SAS 主键生成设备密钥，请运行以下命令：

      ```cmd/sh
      dps-keygen -mk:<Primary_Key(GroupSAS)> -di:<device_id>
      ```

1. 若要设置你的设备，flash 与每个设备**作用域 ID**，**设备 ID**，并**X.509 设备证书**或者**SAS 密钥**。

1. 然后打开适用于其能够连接到 IoT 中心应用程序的设备。 在切换设备时，它是第一次连接到分发点以检索其 IoT 中心注册信息的过程。

1. 已连接的设备最初显示为**未关联的设备**上**Device Explorer**页。 设备预配状态为“已注册”。 **将关联**设备连接到适当的设备模板和批准设备连接到 IoT 中心应用程序。 然后，设备可以从 IoT 中心检索连接字符串，并开始发送数据。 设备预配已完成，现已预配状态**已设置**。

## <a name="provisioning-status"></a>当前设置状态

当实际设备连接到 IoT Central 应用程序，其预配状态更改，如下所示：

1. 设备预配状态是第一个**已注册**。 此状态表示设备在 IoT Central 中创建且具有设备 id。 在注册设备时：
    - 在添加新的真实设备**Device Explorer**页。
    - 使用添加一组设备**导入**上**Device Explorer**页。
    - 在不手动在注册设备**Device Explorer**页上，但使用有效的凭据连接并显示为**非关联**上的设备**Device Explorer**页。

1. 设备预配状态更改为**已设置**连接到 IoT Central 应用程序具有有效凭据的设备完成预配步骤。 在此步骤中，设备从 IoT 中心检索的连接字符串。 现在，设备可以连接到 IoT 中心，并开始发送数据。

1. 操作员可以阻止设备。 设备被阻止时，它不能将数据发送到 IoT 中心应用程序。 已阻止设备的预配状态已**已阻止**。 它可以继续发送数据之前，操作员必须重置设备。 当运算符将取消阻止的设备预配状态为以前的值，返回**Registered**或**已设置**。

## <a name="sdk-support"></a>SDK 支持

Azure 设备 Sdk 的产品/服务最简单的方法为您实现您的设备代码。 以下设备 SDK 已发布：

- [适用于 C 的 Azure IoT SDK](https://github.com/azure/azure-iot-sdk-c)
- [适用于 Python 的 Azure IoT SDK](https://github.com/azure/azure-iot-sdk-python)
- [适用于 Node.js 的 Azure IoT SDK](https://github.com/azure/azure-iot-sdk-node)
- [适用于 Java 的 Azure IoT SDK](https://github.com/azure/azure-iot-sdk-java)
- [适用于 .NET 的 Azure IoT SDK](https://github.com/azure/azure-iot-sdk-csharp)

每个设备都使用可以标识设备的唯一连接字符串进行连接。 设备只能连接到 IoT 中心注册所在。 在 Azure IoT Central 应用程序中创建真实的设备时，应用程序生成来构造连接字符串使用所需的信息`dps-keygen`。

### <a name="sdk-features-and-iot-hub-connectivity"></a>SDK 功能和 IoT 中心连接

与 IoT 中心进行的所有设备通信都使用以下 IoT 中心连接选项：

- [设备到云的消息传递](../iot-hub/iot-hub-devguide-messages-d2c.md)
- [设备孪生](../iot-hub/iot-hub-devguide-device-twins.md)

下表对 Azure IoT Central 设备功能映射到 IoT 中心功能的具体情况进行了汇总：

| Azure IoT Central | Azure IoT 中心 |
| ----------- | ------- |
| 度量：遥测 | 设备到云的消息传递 |
| 设备属性 | 设备孪生报告属性 |
| 设置 | 设备孪生所需的和报告的属性 |

若要详细了解如何使用设备 SDK，请查看下面的某篇文章以获取示例代码：

- [将泛型 Node.js 客户端连接到 Azure IoT Central 应用程序](howto-connect-nodejs.md)
- [将 Raspberry Pi 设备连接到 Azure IoT Central 应用程序](howto-connect-raspberry-pi-python.md)
- [将 DevDiv 工具包设备连接到 Azure IoT Central 应用程序](howto-connect-devkit.md)。

### <a name="protocols"></a>协议

设备 SDK 支持下述用于连接到 IoT 中心的网络协议：

- MQTT
- AMQP
- HTTPS

若要了解这些不同的协议以及选择协议的指南，请参阅[选择通信协议](../iot-hub/iot-hub-devguide-protocols.md)。

如果设备无法使用这些受支持协议中的任何一种，可以使用 Azure IoT Edge 进行协议转换。 IoT Edge 支持其他边缘智能方案，可以将处理从 Azure IoT Central 应用程序卸载到边缘。

## <a name="security"></a>安全

在设备与 Azure IoT Central 之间交换的所有数据都经过加密。 如果设备已连接到任何面向设备的 IoT 中心终结点，则 IoT 中心会对从该设备发出的所有请求进行身份验证。 为了避免通过网络交换凭据，设备使用签名的令牌进行身份验证。 有关详细信息，请参阅[控制对 IoT 中心的访问](../iot-hub/iot-hub-devguide-security.md)。

## <a name="next-steps"></a>后续步骤

现在，你已了解有关在 Azure IoT 中心的设备连接性，下面是建议的后续步骤：

- [准备和连接 DevKit 设备](howto-connect-devkit.md)
- [准备和连接 Raspberry Pi](howto-connect-raspberry-pi-python.md)
- [将泛型 Node.js 客户端连接到 Azure IoT Central 应用程序](howto-connect-nodejs.md)
- [C SDK:预配设备客户端 SDK](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_provisioning_client.md)
