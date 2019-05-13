---
title: 将 Azure 安全中心部署为 IoT Edge 模块 |Microsoft Docs
description: 了解有关如何部署 Azure 安全中心的 IoT Edge 上的 IoT 安全代理。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 32a9564d-16fd-4b0d-9618-7d78d614ce76
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/1/2019
ms.author: mlottner
ms.openlocfilehash: 5d56c878eab92a7152bc0b248cab5c2ebf7e6dde
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65198415"
---
# <a name="deploy-a-security-module-on-your-iot-edge-device"></a>部署 IoT Edge 设备上的安全模块

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

**Azure 安全中心 (ASC) 为 IoT**模块的 IoT Edge 设备提供全面的安全解决方案。
安全模块收集、 聚合，并分析来自你的操作系统和容器系统的原始安全数据到切实可行的安全建议和警报。
若要了解详细信息，请参阅[IoT Edge 的安全模块](security-edge-architecture.md)。

在本指南中，您将了解如何部署 IoT Edge 设备上的安全模块。

## <a name="deploy-security-module"></a>部署安全模块

使用以下步骤来为 IoT Edge 部署 IoT 安全模块 ASC。

### <a name="prerequisites"></a>必备组件

- 在 IoT 中心，请确保你的设备[注册为 IoT Edge 设备](https://docs.microsoft.com/azure/iot-edge/how-to-register-device-portal)。

- ASC 为 IoT Edge 模块需要[审核框架](https://linux.die.net/man/8/auditd)IoT Edge 设备上安装。

    - 在 IoT Edge 设备上运行以下命令来安装 framework:
   
      `sudo apt-get install auditd audispd-plugins`
   
    - 验证审核处于活动状态的通过运行以下命令：
   
      `sudo systemctl status auditd`
      
        预期的响应是`active (running)`。 

### <a name="deployment-using-azure-portal"></a>使用 Azure 门户进行部署

1. 从 Azure 门户中，打开**Marketplace**。

1. 选择**物联网**，然后搜索**IoT 的 Azure 安全中心**并将其选中。

   ![选择 Azure 安全中心的 IoT](media/howto/edge-onboarding-8.png)

1. 单击**创建**配置的部署。 

1. 选择的 Azure**订阅**的 IoT 中心，然后选择你**IoT 中心**。<br>选择**部署到设备**到目标的单个设备，或选择**大规模部署**面向多个设备，然后单击**创建**。 大规模部署的详细信息，请参阅[如何部署](https://docs.microsoft.com/azure/iot-edge/how-to-deploy-monitor)。 

    >[!Note] 
    >如果所选**大规模部署**，在继续之前添加的设备名称和详细信息**添加模块**下列说明中的选项卡。     

有三个步骤，为 IoT 创建 Azure 安全中心的 IoT Edge 部署。 下列各节将引导完成每个步骤。 

#### <a name="step-1-add-modules"></a>步骤 1：添加模块

1. 从**添加模块**选项卡上，**部署模块**区域中，单击**AzureSecurityCenterforIoT**。 
   
1. 更改**名称**到**azureiotsecurity**。
1. 更改**图像 URI**到**mcr.microsoft.com/ascforiot/azureiotsecurity:0.0.3**。
1. 验证是否**容器创建选项**值设置为：      
    ``` json
    {
        "NetworkingConfig": {
            "EndpointsConfig": {
                "host": {}
            }
        },
        "HostConfig": {
            "Privileged": true,
            "NetworkMode": "host",
            "PidMode": "host",
            "Binds": [
                "/:/host"
            ]
        }
    }    
    ```
1. 确认**集模块孪生的所需属性**已选中，然后更改到的配置对象：
      
    ``` json
      "properties.desired": {
        "azureiot*com^securityAgentConfiguration^1*0*0": {
        }
      }
      ```

1. 单击“ **保存**”。
1. 滚动到选项卡，然后选择底部**配置高级 Edge 运行时设置**。
   
   >[!Note]
   > 不要**不**禁用 IoT Edge 中心的 AMQP 通信。
   > Azure 安全中心的 IoT 模块要求使用 IoT Edge 中心的 AMQP 通信。
   
1. 更改**图像**下**边缘中心**到**mcr.microsoft.com/ascforiot/edgehub:1.0.9-preview**。

   >[!Note]
   > Azure 安全中心的 IoT 模块需要 IoT Edge 中心，基于 SDK 版本 1.20 分叉的的版本。
   > 通过更改映像 IoT Edge 中心，则会指示你的 IoT Edge 设备将最新稳定版本替换为 IoT Edge 中心，IoT Edge 服务不正式支持的分叉版本。

1. 验证是否**创建选项**设置为： 
         
    ``` json
    {
      "HostConfig": {
        "PortBindings": {
          "8883/tcp": [{"HostPort": "8883"}],
          "443/tcp": [{"HostPort": "443"}],
          "5671/tcp": [{"HostPort": "5671"}]
        }
      }
    }
    ```
      
1. 单击“ **保存**”。
   
1. 单击“下一步”。

#### <a name="step-2-specify-routes"></a>步骤 2：指定路由 

1. 中**指定的路由**选项卡上，设置**ASCForIoTToIoTHub**将路由到 **"从/消息/模块/azureiotsecurity/\*到 $upstream"**，然后单击**下一步**。

   ![指定路由](media/howto/edge-onboarding-9.png)

#### <a name="step-3-review-deployment"></a>步骤 3：检查部署

1. 在中**查看部署**选项卡上，查看你部署的信息，然后选择**提交**以完成部署。

## <a name="diagnostic-steps"></a>诊断步骤

如果遇到问题，容器日志是了解有关 IoT Edge 安全模块设备的状态信息的最佳方式。 可以使用本部分中的命令和工具来收集信息。

### <a name="verify-the-required-containers-are-installed-and-functioning-as-expected"></a>验证所需的容器已安装且按预期方式工作

1. 在 IoT Edge 设备上运行以下命令：
    
     `sudo docker ps`
   
1. 验证以下容器正在运行：
   
   | 名称 | IMAGE |
   | --- | --- |
   | azureIoTSecurity | mcr.microsoft.com/ascforiot/azureiotsecurity:0.0.3 |
   | edgeHub | mcr.microsoft.com/ascforiot/edgehub:1.0.9-preview |
   | edgeAgent | mcr.microsoft.com/azureiotedge-agent:1.0 |
   
   如果容器不存在所需的最低，请检查对 IoT Edge 部署清单符合建议的设置。 有关详细信息，请参阅[部署 IoT Edge 模块](#deployment-using-azure-portal)。

### <a name="inspect-the-module-logs-for-errors"></a>检查模块日志中的错误
   
1. 在 IoT Edge 设备上运行以下命令：

   `sudo docker logs azureiotsecurity`
   
1. 更详细的日志，将添加到以下环境变量**azureiotsecurity**部署模块： `logLevel=Debug`。

## <a name="next-steps"></a>后续步骤

若要了解有关配置选项的详细信息，请转到模块配置的操作方法指南。 
> [!div class="nextstepaction"]
> [模块配置操作方法指南](./how-to-agent-configuration.md)
