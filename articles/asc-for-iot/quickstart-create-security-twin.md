---
title: 为适用于 IoT 的 Azure 安全中心预览版创建安全模块孪生 | Microsoft Docs
description: 了解如何创建与“适用于 IoT 的 Azure 安全中心”(ASC for IoT) 配合使用的模块孪生。
services: asc-for-iot
ms.service: ascforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: c782692e-1284-4c54-9d76-567bc13787cc
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/26/2019
ms.author: mlottner
ms.openlocfilehash: af952b59a2f4924be79c0ac7dc728f1d9bd0d641
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200719"
---
# <a name="quickstart-create-an-azureiotsecurity-module-twin"></a>快速入门：创建 azureiotsecurity 模块孪生

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本快速入门介绍如何为新设备创建单个 _azureiotsecurity_ 模块孪生，或者为 IoT 中心内的所有设备批量创建模块孪生。  

## <a name="understanding-azureiotsecurity-module-twins"></a>了解 azureiotsecurity 模块孪生 

对于在 Azure 中生成的 IoT 解决方案，设备孪生在设备管理和流程自动化方面发挥着关键作用。 

适用于 IoT 的 Azure 安全中心 (ASC) 可与现有的 IoT 设备管理平台完全集成，使你能够管理设备的安全状态，以及利用现有的设备控制功能。
ASC for IoT 集成是使用 IoT 中心孪生机制实现的。  

请参阅 [IoT 中心模块孪生](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-module-twins)详细了解 Azure IoT 中心内模块孪生的一般概念。 
 
ASC for IoT 利用模块孪生机制，并为每个设备维护一个名为 _azureiotsecurity_ 的安全模块孪生。
该安全模块孪生保存每个设备的所有设备安全性相关信息。 
 
若要利用 ASC for IoT 功能，需要对服务中的每个设备创建、配置并使用这些安全模块孪生。  

## <a name="create-azureiotsecurity-module-twin"></a>创建 azureiotsecurity 模块孪生 

可通过两种方式创建 _azureiotsecurity_ 模块孪生：
1. [模块批处理脚本](https://aka.ms/iot-security-github-create-module) - 使用默认配置为新的设备或者不包含模块孪生的设备自动创建模块孪生。
2. 使用每个设备的特定配置单独手动编辑每个模块孪生。

>[!NOTE] 
> 使用批处理方法不会覆盖现有的 azureiotsecurity 模块孪生。 使用批处理方法只会为尚不包含安全模块孪生的设备创建新的模块孪生。 

请参阅[代理配置](how-to-agent-configuration.md)，了解如何修改或更改现有模块孪生的配置。 

若要手动为设备创建新的 _azureiotsecurity_ 模块孪生，请遵照以下说明操作： 

1. 在 IoT 中心，找到并选择要为其创建安全模块孪生的设备。
1. 单击设备，然后单击“添加模块标识”。
1. 在“模块标识名称”字段中输入 **azureiotsecurity**。

1. 单击“ **保存**”。 

## <a name="verify-creation-of-a-module-twin"></a>验证是否要创建模块孪生

验证特定的设备是否存在安全模块孪生：

1. 在 Azure IoT 中心，从“资源管理器”菜单中选择“IoT 设备”。    
1. 输入设备 ID，或者在“查询设备字段”中选择一个选项，然后单击“查询设备”。 
    ![查询设备](./media/quickstart/verify-security-module-twin.png)
1. 选择该设备或双击它，以打开“设备详细信息”页。 
1. 选择“模块标识”菜单，在与设备关联的模块标识列表中，确认是否存在 **azureiotsecurity** 模块。 
    ![与设备关联的模块](./media/quickstart/verify-security-module-twin-3.png)


若要详细了解如何自定义 ASC for IoT 模块孪生的属性，请参阅[代理配置](how-to-agent-configuration.md)。

## <a name="next-steps"></a>后续步骤

转到下一篇文章，了解如何配置自定义警报...

> [!div class="nextstepaction"]
> [配置自定义警报](quickstart-create-custom-alerts.md)
