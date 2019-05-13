---
title: 将 Syslog 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Syslog 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 5dd59729-c623-4cb4-b326-bb847c8f094b
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 673b1df6094703bebcbfd9d82c1268c01d46e814
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233584"
---
# <a name="connect-your-external-solution-using-syslog"></a>连接外部解决方案是使用 Syslog

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

你可以连接支持到 Azure Sentinel Syslog 任何本地设备。 这是通过使用基于设备和 Azure Sentinel 之间在 Linux 计算机上的代理。 如果在 Azure 中 Linux 计算机，你可以从你的设备或应用程序的专用工作区在 Azure 中创建并将其连接到日志流式传输。 如果你的 Linux 计算机不是在 Azure 中，您可以将日志流式传输从你的设备的专用在其上安装适用于 Linux 代理的 VM 或计算机本地。 

> [!NOTE]
> 如果你的设备支持 Syslog CEF 连接更完整，是应选择此选项并按照中的说明[将数据连接从 CEF](connect-common-event-format.md)。

## <a name="how-it-works"></a>工作原理

Syslog 连接是使用适用于 Linux 的代理来完成。 默认情况下，适用于 Linux 代理接收事件从 Syslog 守护程序通过 UDP，但在 Linux 计算机的地方收集大量的 Syslog 事件，如 Linux 代理接收事件时从其他设备，为已修改配置的情况下使用 Syslog 守护程序和代理之间的 TCP 传输。

## <a name="connect-your-syslog-appliance"></a>连接你的 Syslog 设备

1. 在 Azure Sentinel 门户中，选择**数据连接器**，然后选择**Syslog**磁贴。
2. 如果你的 Linux 计算机不是在 Azure 中，下载并安装 Azure Sentinel**适用于 Linux 代理**上你的设备。 
1. 如果你正在 Azure 中，选择或专用于接收 Syslog 消息在 Azure Sentinel 工作区中，创建的 VM。 在 Azure Sentinel 工作区中选择 VM，并单击**Connect**顶部的左窗格。
3. 单击**配置要连接的日志**Syslog 连接器安装程序中返回。 
4. 单击**以打开配置边栏选项卡，请按此处**。
1. 选择**数据**，然后**Syslog**。
   - 请确保要发送的 Syslog 的每个设施的表中。 对于每个设施要监视，将严重性设置。 单击“应用”。
1. 在你 Syslog 的计算机，请确保要将发送这些功能。 

3. 若要使用 Log Analytics 中的 Syslog 日志相关的架构，搜索**Syslog**。




## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何连接到 Azure Sentinel Syslog 的本地设备。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。
