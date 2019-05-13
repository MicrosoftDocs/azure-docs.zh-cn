---
title: 选择并将其部署为 IoT 代理预览版的 Azure 安全中心 |Microsoft Docs
description: 了解有关如何选择并将其部署为 IoT 设备上的 IoT 安全代理的 Azure 安全中心。
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
ms.date: 03/27/2019
ms.author: mlottner
ms.openlocfilehash: 5c76cce5a3d9add8a1dcd8acc439a70c4a8655a9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200565"
---
# <a name="select-and-deploy-a-security-agent-on-your-iot-device"></a>选择并部署 IoT 设备上的安全代理

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

适用于 IoT 的 azure 安全中心 (ASC) 提供参考体系结构的安全代理，监视和从 IoT 设备收集数据。
请参阅[安全代理参考体系结构](security-agent-architecture.md)若要了解详细信息。

代理作为开放源代码项目开发的有两种类型： <br> [C](https://aka.ms/iot-security-github-c)，并[ C# ](https://aka.ms/iot-security-github-cs)。

在本文中，学习如何： 
> [!div class="checklist"]
> * 比较安全代理版本
> * 发现支持的代理的平台
> * 选择你的解决方案的正确的代理风格

## <a name="understand-security-agent-options"></a>了解安全代理选项

IoT 安全代理类型的每个 ASC 提供相同的功能，集，还支持类似的配置选项。 

基于 C 的安全代理具有较低的内存占用量，，具有较少的可用资源的设备的最佳选择。 

|     | 基于 C 的安全代理 | C#-基于安全代理 |
| --- | ----------- | --------- |
| 开放源 | 可用下[MIT 许可](https://en.wikipedia.org/wiki/MIT_License)中[Github](https://aka.ms/iot-security-github-cs) | 可用下[MIT 许可](https://en.wikipedia.org/wiki/MIT_License)中[Github](https://aka.ms/iot-security-github-c) |
| 开发语言    | C | C# |
| 支持的 Windows 平台？ | 否 | 是 |
| Windows 先决条件 | --- | [WMI](https://docs.microsoft.com/windows/desktop/wmisdk/) |
| 支持的 Linux 平台？ | 是的 x64 和 x86 | 是的仅限 x64 |
| Linux 系统必备组件 | libunwind8、 libcurl3、 uuid 运行时、 审核、 audispd 插件 | libunwind8, libcurl3, uuid-runtime, auditd, audispd-plugins, sudo, netstat, iptables |
| 磁盘空间占用量 | 10.5 MB | 90MB |
| 内存占用量 （上平均） | 5.5 MB | 33MB |
| [身份验证](concept-security-agent-authentication-methods.md)到 IoT 中心 | 是 | 是 |
| 安全数据[集合](how-to-agent-configuration.md#supported-security-events) | 是 | 是 |
| 事件聚合 | 是 | 是 |
| 通过远程配置[安全模块孪生](concept-security-module.md) | 是 | 是 |


## <a name="choose-an-agent-flavor"></a>选择代理风格 

回答以下有关将 IoT 设备选择正确的代理的问题：

- 您是否正在使用_Windows Server_或_Windows IoT Core_？ 

    [部署C#-基于 Windows 的安全代理](how-to-deploy-windows-cs.md)。

- 您使用的是 Linux 版本具有 x86 体系结构？ 

    [部署适用于 Linux 的基于 C 的安全代理](how-to-deploy-linux-c.md)。

- 您使用的 Linux 分发带有 x64 体系结构？

    可以使用任一代理风格。 <br>
    [部署适用于 Linux 的基于 C 的安全代理](how-to-deploy-linux-c.md)和/或[部署C#-基于 Linux 的安全代理](how-to-deploy-linux-cs.md)。

这两个代理版本提供了相同的功能集，并支持类似的配置选项。
请参阅[安全代理比较](how-to-deploy-agent.md#understand-security-agent-options)若要了解详细信息。

## <a name="supported-platforms"></a>支持的平台

以下列表包含所有当前支持的平台。

|ASC IoT 代理 |操作系统 |体系结构 |
|--------------|------------|--------------|
|C|Ubuntu 16.04 |   x64|
|C|Ubuntu 18.04 |   x64|
|C|Debian 9 |   x64、 x86|
|C#|Ubuntu 16.04    |x64|
|C#|Ubuntu 18.04    |x64|
|C#|Debian 9    |x64|
|C#|Windows Server 2016|    X64|
|C#|Windows 10 IoT 核心版 17763 |x64|

## <a name="next-steps"></a>后续步骤

若要了解有关配置选项的详细信息，请转到代理配置的操作方法指南。 
> [!div class="nextstepaction"]
> [代理配置操作指南](./how-to-agent-configuration.md)
