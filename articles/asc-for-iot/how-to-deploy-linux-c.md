---
title: 安装和部署 IoT 代理预览版的 Azure 安全中心的 Linux C 代理指南 |Microsoft Docs
description: 了解如何在 32 位和 64 位 Linux 上安装 IoT 代理在 Azure 安全中心。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 3ccf2aec-106a-4d2c-8079-5f3e8f2afdcb
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/28/2019
ms.author: mlottner
ms.openlocfilehash: 49ebb5932e1e918330625fd0df98811873dd5cd5
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200661"
---
# <a name="deploy-azure-security-center-for-iot-c-based-security-agent-for-linux"></a>为适用于 Linux 的 IoT C 基于安全代理部署 Azure 安全中心

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本指南介绍如何安装和部署 IoT C 基于安全代理，Linux 上的 Azure 安全中心 (ASC)。

本指南介绍如何： 
> [!div class="checklist"]
> * 安装
> * 验证部署
> * 卸载代理
> * 故障排除 

## <a name="prerequisites"></a>必备组件

有关其他平台和代理版本，请参阅[选择适当的安全代理](how-to-deploy-agent.md)。

1. 若要部署安全代理，你希望 (sudo) 安装在计算机上需要本地管理员权限。

1. 为设备[创建安全模块](quickstart-create-security-twin.md)。

## <a name="installation"></a>安装 

若要安装和部署安全代理，请执行以下操作：


1. 将 [Github](https://aka.ms/iot-security-github-c) 中的最新代理版本下载到计算机。

1. 提取包的内容，并导航到 _/Install_ 文件夹。

1. 运行以下命令，将运行权限添加到 **InstallSecurityAgent 脚本**：
    
   ```
   chmod +x InstallSecurityAgent.sh
   ```

1. 接下来运行： 

   ```
   ./InstallSecurityAgent.sh -aui <authentication identity> -aum <authentication method> -f <file path> -hn <host name> -di <device id> -i
   ```
   
   有关身份验证参数的详细信息，请参阅[如何配置身份验证](concept-security-agent-authentication-methods.md)。

此脚本执行以下任务：

1. 安装必备组件。

2. 添加服务用户（在禁用交互式登录的情况下）。

3. 安装用作**守护程序**的代理 - 假设设备使用 **systemd** 进行服务管理。

4. 使用提供的身份验证参数配置代理。 

如需更多帮助，请结合 -help 参数运行该脚本： 
    
    ./InstallSecurityAgent.sh --help

### <a name="uninstall-the-agent"></a>卸载代理

若要卸载代理，请运行脚本时使用 –-uninstall 参数：

    ./InstallSecurityAgent.sh -–uninstall

## <a name="troubleshooting"></a>故障排除
运行以下命令来检查部署状态：

    systemctl status ASCIoTAgent.service


## <a name="next-steps"></a>后续步骤
- 阅读适用于 IoT 的 ASC 服务[概述](overview.md)
- 详细了解适用于 IoT 的 ASC [体系结构](architecture.md)
- 启用该[服务](quickstart-onboard-iot-hub.md)
- 阅读[常见问题解答](resources-frequently-asked-questions.md)
- 了解[安全警报](concept-security-alerts.md)