---
title: 本快速入门介绍如何使用 C 将模拟的 X.509 设备预配到 Azure IoT 中心 | Microsoft Docs
description: 本快速入门使用单独注册。 在本快速入门中，我们将使用适用于 Azure IoT 中心设备预配服务的 C 设备 SDK 创建和预配模拟的 X.509 设备。
author: wesmc7777
ms.author: wesmc
ms.date: 04/10/2019
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 2a27ec63ea4ae25823c604612bd3bcf3a057da61
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "61249208"
---
# <a name="quickstart-provision-an-x509-simulated-device-using-the-azure-iot-c-sdk"></a>快速入门：使用 Azure IoT C SDK 预配 X.509 模拟设备

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

本快速入门介绍如何在 Windows 开发计算机上创建和运行 X.509 设备模拟器。 你将使用设备预配服务实例的注册，将此模拟设备配置为分配到 IoT 中心。 将使用 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) 中的示例代码来模拟设备的启动序列。 将根据预配服务的注册来识别该设备，然后将其分配到 IoT 中心。

如果不熟悉自动预配过程，请查看[自动预配的概念](concepts-auto-provisioning.md)。 另外，在继续学习本快速入门之前，请确保已完成[通过 Azure 门户设置 IoT 中心设备预配服务](./quick-setup-auto-provision.md)中的步骤。 

Azure IoT 设备预配服务支持两类注册：
- [注册组](concepts-service.md#enrollment-group)：用于注册多个相关设备。
- [单独注册](concepts-service.md#individual-enrollment)：用于注册单个设备。

本文将演示单个注册。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>先决条件

* 已启用[“使用 C++ 的桌面开发”](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/)工作负荷的 Visual Studio 2015 或 [Visual Studio 2017](https://www.visualstudio.com/vs/)。
* 已安装最新版本的 [Git](https://git-scm.com/download/)。


<a id="setupdevbox"></a>

## <a name="prepare-a-development-environment-for-the-azure-iot-c-sdk"></a>为 Azure IoT C SDK 准备开发环境

在本部分，你将准备一个开发环境，用于生成包含 X.509 启动序列示例代码的 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c)。

1. 下载 [CMake 生成系统](https://cmake.org/download/)。

    在进行 `CMake` 安装**之前**，必须在计算机上安装 Visual Studio 必备组件（Visual Studio 和“使用 C++ 的桌面开发”工作负荷）。 满足先决条件并验证下载内容后，安装 CMake 生成系统。

2. 打开命令提示符或 Git Bash shell。 执行以下命令克隆 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) GitHub 存储库：
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```
    应该预料到此操作需要几分钟才能完成。


3. 在 git 存储库的根目录中创建 `cmake` 子目录，并导航到该文件夹。 

    ```cmd/sh
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

4. 代码示例使用 X.509 证书通过 X.509 身份验证提供证明。 运行以下命令，生成特定于你的开发客户端平台的 SDK 版本。 将在 `cmake` 目录中生成模拟设备的 Visual Studio 解决方案。 

    ```cmd
    cmake -Duse_prov_client:BOOL=ON ..
    ```
    
    如果 `cmake` 找不到 C++ 编译器，则可能会在运行以上命令时出现生成错误。 如果出现这种情况，请尝试在 [Visual Studio 命令提示符](https://docs.microsoft.com/dotnet/framework/tools/developer-command-prompt-for-vs)窗口中运行该命令。 

    生成成功后，最后的几个输出行如下所示：

    ```cmd/sh
    $ cmake -Duse_prov_client:BOOL=ON ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```



<a id="portalenroll"></a>

## <a name="create-a-self-signed-x509-device-certificate"></a>创建自签名的 X.509 设备证书

本部分将使用自签名的 X.509 证书，请务必记住以下几点：

* 自签名证书仅用于测试，不应在生产环境中使用。
* 自签名证书的默认过期日期为一年。

你将使用来自 Azure IoT C SDK 的示例代码创建要与模拟设备的个人注册条目一起使用的证书。

1. 启动 Visual Studio 并打开名为 `azure_iot_sdks.sln` 的新解决方案文件。 此解决方案文件位于先前在 azure-iot-sdk-c git 存储库的根目录中创建的 `cmake` 文件夹中。

2. 在 Visual Studio 菜单中，选择“生成” > “生成解决方案”以生成解决方案中的所有项目。

3. 在 Visual Studio 的“解决方案资源管理器”窗口中，导航到 **Provision\_Tools** 文件夹。 右键单击“dice\_device\_enrollment”项目，然后选择“设置为启动项目”。 

4. 在 Visual Studio 菜单中，选择“调试” > “开始执行(不调试)”以运行该解决方案。 在输出窗口中，当系统提示时输入 **i** 完成单个注册。 

    输出窗口会显示在本地为模拟设备生成的自签名 X.509 证书。 将输出复制到剪贴板，从 **-----BEGIN CERTIFICATE-----** 开始，到第一个 **-----END CERTIFICATE-----** 结束，确保将这两行也包括进去。 只需要输出窗口中的第一个证书。
 
5. 使用文本编辑器将证书保存到名为 **_X509testcert.pem_** 的新文件中。 


## <a name="create-a-device-enrollment-entry-in-the-portal"></a>在门户中创建设备注册项

1. 登录到 Azure 门户，单击左侧菜单上的“所有资源”按钮，打开设备预配服务。

2. 选择“管理注册”选项卡，然后单击顶部的“添加个人注册”按钮。 

3. 在“添加注册”中输入以下信息，然后单击“保存”按钮。

    - **机制：** 选择“X.509”作为标识证明机制。
    - **主要证书 .pem 或 .cer 文件：** 单击“选择文件”，选择前面创建的证书文件 X509testcert.pem。
    - **IoT 中心设备 ID：** 输入 test-docs-cert-device 作为设备的 ID。

      [![在门户中为 X.509 证明添加单个注册](./media/quick-create-simulated-device-x509/device-enrollment.png)](./media/quick-create-simulated-device-x509/device-enrollment.png#lightbox)

      成功注册以后，X.509 设备会在“单独注册”选项卡的“注册 ID”列下显示为 **riot-device-cert**。 



<a id="firstbootsequence"></a>

## <a name="simulate-first-boot-sequence-for-the-device"></a>模拟设备的首次启动顺序

在本部分更新向设备预配服务实例发送设备启动序列的示例代码。 此启动序列使得设备可被识别并分配到与设备预配服务实例链接的 IoT 中心。



1. 在 Azure 门户中，选择设备预配服务的“概述”选项卡，记下“ID 范围”的值。

    ![从门户边栏选项卡中提取设备预配服务终结点信息](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. 在 Visual Studio 的“解决方案资源管理器”窗口中，导航到 **Provision\_Samples** 文件夹。 展开名为 **prov\_dev\_client\_sample** 的示例项目。 展开“源文件”，打开 **prov\_dev\_client\_sample.c**。

3. 找到 `id_scope` 常量，将值替换为前面复制的“ID 范围”值。 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

4. 在同一文件中找到 `main()` 函数的定义。 确保 `hsm_type` 变量设置为 `SECURE_DEVICE_TYPE_X509` 而不是 `SECURE_DEVICE_TYPE_TPM`，如下所示。

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    hsm_type = SECURE_DEVICE_TYPE_X509;
    ```

5. 右键单击“prov\_dev\_client\_sample”项目，然后选择“设为启动项目”。 

6. 在 Visual Studio 菜单中，选择“调试” > “开始执行(不调试)”以运行该解决方案。 在重新生成项目的提示中单击“是”，以便在运行项目之前重新生成项目。

    以下输出示例显示预配设备客户端示例成功启动，然后连接到预配服务实例来获取 IoT 中心信息并注册：

    ```cmd
    Provisioning API Version: 1.2.7

    Registering... Press enter key to interrupt.

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: 
    test-docs-hub.azure-devices.net, deviceId: test-docs-cert-device    
    ```

7. 在门户中导航到已链接到预配服务的 IoT 中心，然后单击“IoT 设备”选项卡。将模拟的 X.509 设备成功预配到中心以后，设备 ID 会显示在“IoT 设备”边栏选项卡上，“状态”为“已启用” 你可能需要单击顶部的“刷新”按钮。 

    ![设备注册到 IoT 中心](./media/quick-create-simulated-device/hub-registration.png) 


## <a name="clean-up-resources"></a>清理资源

如果打算继续使用和探索设备客户端示例，请勿清理在本快速入门中创建的资源。 如果不打算继续学习，请通过以下步骤删除通过本快速入门创建的所有资源。

1. 关闭计算机上的设备客户端示例输出窗口。
1. 在 Azure 门户的左侧菜单中单击“所有资源”，然后选择设备预配服务。 打开服务的“管理注册”，然后单击“个人注册”选项卡。选择在本快速入门中注册的设备的“注册 ID”，然后单击顶部的“删除”按钮。 
1. 在 Azure 门户的左侧菜单中单击“所有资源”，然后选择 IoT 中心。 打开中心的“IoT 设备”，选择在本快速入门中注册的设备的“设备 ID”，然后单击顶部的“删除”按钮。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何在 Windows 计算机上创建模拟 X.509 设备，以及如何使用门户中的 Azure IoT 中心设备预配服务将其预配到 IoT 中心。 若要了解如何以编程方式注册 X.509 设备，请继续阅读快速入门中关于 X.509 设备的编程注册内容。 

> [!div class="nextstepaction"]
> [Azure 快速入门 - 将 X.509 设备注册到 Azure IoT 中心设备预配服务](quick-enroll-device-x509-java.md)
