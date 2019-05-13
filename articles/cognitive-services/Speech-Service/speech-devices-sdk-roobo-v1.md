---
title: 语音设备 SDK Roobo 智能音频开发人员工具包 v1-语音服务
titleSuffix: Azure Cognitive Services
description: 先决条件和开始使用语音设备 SDK Roobo 智能音频适用于开发人员工具包 v1 的说明。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: erhopf
ms.openlocfilehash: 7c1a13a44d9db8ed029ce798f0bb34944a1a65a7
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409058"
---
# <a name="device-roobo-smart-audio-dev-kit"></a>设备:Roobo 智能音频的开发人员工具包

本文提供了 Roobo 智能音频开发工具包的设备特定信息。

## <a name="set-up-the-development-kit"></a>设置开发工具包

1. 开发工具包拥有两个 micro USB 连接器。 左侧的连接器用于为开发工具包接通电源，在下图中突出显示为 Power。 右侧的连接器用于控制开发工具包，在图中标记为 Debug。

    ![连接开发工具包](media/speech-devices-sdk/qsg-1.png)

1. 使用 micro USB 数据线为开发工具包接通电源，将电源端口连接到 PC 或电源适配器。 顶部板下的绿色电源指示灯将亮起。

1. 若要控制开发工具包，请使用第二个 micro USB 电缆连接计算机调试端口。 务必使用高品质的数据线，以确保可靠的通信。

1. 摆放好开发工具包以进行环形或线性配置。

    |开发工具包配置|方向|
    |-----------------------------|------------|
    |环形|竖直，麦克风面向天花板|
    |线性|在其一端，麦克风面向你（如下图所示）|

    ![线性开发工具包方向](media/speech-devices-sdk/qsg-2.png)

1. 安装证书和设置的权限的声音的设备。 在命令提示符窗口中键入以下命令：

   ```powershell
   adb push C:\SDSDK\Android-Sample-Release\scripts\roobo_setup.sh /data/
   adb shell
   cd /data/
   chmod 777 roobo_setup.sh
   ./roobo_setup.sh
   exit
   ```

    > [!NOTE]
    > 这些命令使用 Android Debug Bridge `adb.exe`，它是 Android Studio 安装的一部分。 此工具位于 C:\Users\[用户名]\AppData\Local\Android\Sdk\platform-tools 中。 可将该目录添加到你的路径，以便更轻松地调用 `adb`。 否则，必须在调用 `adb` 的每个命令中指定安装 adb.exe 的完整路径。
    >
    > 如果看到错误`no devices/emulators found`然后检查您的 USB 电缆连接，并且是高质量电缆。 可使用 `adb devices` 检查并确保计算机可与开发工具包通信，因为它将返回设备列表。
    >
    > [!TIP]
    > 将电脑的麦克风和扬声器调为静音，以确保使用开发工具包的麦克风。 这可以避免电脑中的音频意外触发设备。

1. 如果希望将扬声器连接到开发工具包，可将其连接到音频线路输出。应该选择与 3.5mm 模拟插件良好质量演讲者。

    ![Vysor 音频](media/speech-devices-sdk/qsg-14.png)

## <a name="development-information"></a>开发的信息

有关开发的详细信息，请参阅[Roobo 开发指南](http://dwn.roo.bo/server_upload/ddk/ROOBO%20Dev%20Kit-User%20Guide.pdf)。

## <a name="audio"></a>收听验证码

Roobo 提供捕获所有音频闪存的工具。 它可能有助于排查音频问题。 已针对每个开发工具包配置提供该工具的一个版本。 上[Roobo 站点](https://ddk.roobo.com/)，选择你的设备，并选择**Roobo 工具**在页面底部的链接。

## <a name="next-steps"></a>后续步骤

* [运行 Android 示例应用](speech-devices-sdk-android-quickstart.md)
