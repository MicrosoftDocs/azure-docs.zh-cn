---
title: 安装和部署 Linux 指南C#代理的 IoT 预览版的 Azure 安全中心 |Microsoft Docs
description: 了解如何在 32 位和 64 位 Linux 上安装 IoT 代理在 Azure 安全中心。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: b0982203-c3c8-4a0b-8717-5b5ac4038d8c
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2019
ms.author: mlottner
ms.openlocfilehash: 5623b9870788edfb3b96ef248154e8b9f60b4593
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198434"
---
# <a name="deploy-azure-security-center-for-iot-c-based-security-agent-for-linux"></a>为 IoT 部署 Azure 安全中心C#-基于 Linux 的安全代理

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本指南介绍如何安装和部署 IoT Azure 安全中心 (ASC) C#-基于 Linux 的安全代理。

本指南介绍如何： 
> [!div class="checklist"]
> * 安装
> * 验证部署
> * 卸载代理
> * 故障排除 

## <a name="prerequisites"></a>必备组件

有关其他平台和代理版本，请参阅[选择适当的安全代理](how-to-deploy-agent.md)。

1. 若要部署安全代理，需要在安装计算机上拥有本地管理员权限。 

1. 为设备[创建安全模块](quickstart-create-security-twin.md)。

## <a name="installation"></a>安装 

若要部署安全代理，请执行以下操作：

1. 将 [Github](https://aka.ms/iot-security-github-cs) 中的最新代理版本下载到计算机。

1. 提取包的内容，并导航到 _/Install_ 文件夹。

1. 运行 `chmod +x InstallSecurityAgent.sh`，将运行权限添加到 **InstallSecurityAgent 脚本** 

1. 然后运行： 

   ```
   ./InstallSecurityAgent.sh -i -aui <authentication identity>  -aum <authentication method> -f <file path> -hn <host name>  -di <device id> -cl <certificate location kind>
   ```
   
   有关身份验证参数的详细信息，请参阅[如何配置身份验证](concept-security-agent-authentication-methods.md)。

此脚本执行以下任务：

- 安装必备组件。

- 添加服务用户（在禁用交互式登录的情况下）。

- 安装用作**守护程序**的代理 - 假设设备使用 **systemd** 进行服务管理。

- 配置 **sudoers**，以允许代理以 root 身份执行某些任务。

- 使用提供的身份验证参数配置代理。


如需更多帮助，请结合 -help 参数运行该脚本：`./InstallSecurityAgent.sh --help`

### <a name="uninstall-the-agent"></a>卸载代理

若要卸载代理，请结合 -u 参数运行该脚本：`./InstallSecurityAgent.sh -u`。 

> [!NOTE]
> 卸载时，不会删除在安装过程中安装的任何缺失的必备组件。

## <a name="troubleshooting"></a>故障排除  

1. 运行以下命令来检查部署状态：

    `systemctl status ASCIoTAgent.service`

2. 启用日志记录。  
   如果代理无法启动，请启用日志记录以获取更多信息。

   通过以下方式启用日志记录：

   1. 在任意 Linux 编辑器中打开要编辑的配置文件：

        `vi /var/ASCIoTAgent/General.config`

   1. 编辑以下值： 

      ```
      <add key="logLevel" value="Debug"/>
      <add key="fileLogLevel" value="Debug"/>
      <add key="diagnosticVerbosityLevel" value="Some" /> 
      <add key="logFilePath" value="IotAgentLog.log"/>
      ```
       **logFilePath** 值是可以配置的。 

       > [!NOTE]
       > 建议在故障排除操作完成后关闭日志记录功能。 让日志记录保持启用状态会增加日志文件大小和数据使用量。

   1. 通过运行以下命令重启代理：

       `systemctl restart ASCIoTAgent.service`

   1. 查看日志文件，了解有关故障的详细信息。  

       日志文件位置为：`/var/ASCIoTAgent/IotAgentLog.log`

       根据你在步骤 2 中为 **logFilePath** 选择的名称，更改文件位置路径。 

## <a name="next-steps"></a>后续步骤

- 阅读适用于 IoT 的 ASC 服务[概述](overview.md)
- 详细了解适用于 IoT 的 ASC [体系结构](architecture.md)
- 启用该[服务](quickstart-onboard-iot-hub.md)
- 阅读[常见问题解答](resources-frequently-asked-questions.md)
- 了解[警报](concept-security-alerts.md)