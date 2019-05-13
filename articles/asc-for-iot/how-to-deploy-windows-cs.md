---
title: Azure 安全中心的 IoT 代理预览版的 Windows 安装 |Microsoft Docs
description: 了解有关如何安装适用于 32 位或 64 位 Windows 设备上 IoT 代理 Azure 安全中心。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 2cf6a49b-5d35-491f-abc3-63ec24eb4bc2
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/19/2019
ms.author: mlottner
ms.openlocfilehash: 64fc576885bb9d9c3c46aafd808db65d2f8ff77f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200603"
---
# <a name="deploy-an-azure-security-center-for-iot-c-based-security-agent-for-windows"></a>为 IoT 部署 Azure 安全中心C#-基于 Windows 的安全代理

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本指南介绍如何安装适用于 IoT Azure 安全中心 (ASC) C#-基于 Windows 上的安全代理。

本指南介绍如何： 
> [!div class="checklist"]
> * 安装
> * 验证部署
> * 卸载代理
> * 故障排除 

## <a name="prerequisites"></a>必备组件

有关其他平台和代理风格，请参阅[选择适当的安全代理](how-to-deploy-agent.md)。

1. 你想要安装在计算机上的本地管理员权限。 

1. 为设备[创建安全模块](quickstart-create-security-twin.md)。

## <a name="installation"></a>安装 

若要安装安全代理，请执行以下操作：

1. 若要安装适用于 IoT Windows ASCC#代理在设备上，下载最新版本为您的计算机与 ASC iot [GitHub 存储库](https://github.com/Azure/Azure-IoT-Security-Agent-CS)。

2. 提取包的内容，并导航到 /Install 文件夹。

3. 以管理员身份打开 Windows PowerShell。 
    1. 运行 ```Unblock-File .\InstallSecurityAgent.ps1```，将运行权限添加到 InstallSecurityAgent 脚本
    
        然后运行：

    ```
    .\InstallSecurityAgent.ps1 -Install -aui <authentication identity> -aum <authentication method> -f <file path> -hn <host name> -di <device id> -cl <certificate location kind>
    ```
    
    例如：
    
    ```
    .\InstallSecurityAgent.ps1 -Install -aui Device -aum SymmetricKey -f c:\Temp\Key.txt -hn MyIotHub.azure-devices.net -di Mydevice1 -cl store
    ```
    
    有关身份验证参数的详细信息，请参阅[如何配置身份验证](concept-security-agent-authentication-methods.md)。

此脚本执行以下任务：

- 安装必备组件。

- 添加服务用户（在禁用交互式登录的情况下）。

- 将代理作为**系统服务**安装。

- 使用提供的身份验证参数配置代理。


如需其他帮助，请在 PowerShell 中使用 Get-Help 命令 <br>Get-Help 示例：  
    ```Get-Help .\InstallSecurityAgent.ps1```

### <a name="verify-deployment-status"></a>验证部署状态

- 运行以下命令来检查代理部署状态：<br>
    ```sc.exe query "ASC IoT Agent"```

### <a name="uninstall-the-agent"></a>卸载代理

若要卸载代理，请执行以下操作：

1. 运行以下 PowerShell 脚本，将 **-mode** 参数设置为 **Uninstall**。  

    ```
    .\InstallSecurityAgent.ps1 -Uninstall
    ``` 

## <a name="troubleshooting"></a>故障排除

如果代理无法启动，请启用日志记录（日志记录默认设置为“禁用”）以获取更多信息。

若要启用日志记录，请执行以下操作：

1. 打开配置文件 (General.config) 进行编辑使用标准文件编辑器。

1. 编辑以下值：

   ```xml
   <add key="logLevel" value="Debug" />
   <add key="fileLogLevel" value="Debug"/> 
   <add key="diagnosticVerbosityLevel" value="Some" /> 
   <add key="logFilePath" value="IoTAgentLog.log" />
   ```

    > [!NOTE]
    > 建议在故障排除操作完成后关闭日志记录功能。 让日志记录保持启用状态会增加日志文件大小和数据使用量。 

1. 运行以下 PowerShell 或命令行，以便重启代理：

    **Powershell**
     ```
     Restart-Service "ASC IoT Agent"
     ```
     
   或

    **CMD**
     ```
     sc.exe stop "ASC IoT Agent" 
     sc.exe start "ASC IoT Agent" 
     ```

1. 查看日志文件，了解有关故障的详细信息。

   日志文件位置：`%WinDir%/System32/IoTAgentLog.log`


## <a name="next-steps"></a>后续步骤
- 阅读适用于 IoT 的 ASC 服务[概述](overview.md)
- 详细了解适用于 IoT 的 ASC [体系结构](architecture.md)
- 启用该[服务](quickstart-onboard-iot-hub.md)
- 阅读[常见问题解答](resources-frequently-asked-questions.md)
- 了解[警报](concept-security-alerts.md)