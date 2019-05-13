---
title: 将数据源连接到 Azure Sentinel 预览版 | Microsoft Docs
description: 了解如何将数据源连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: a3b63cfa-b5fe-4aff-b105-b22b424c418a
ms.service: sentinel
ms.devlang: na
ms.topic: overview
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: b54c4ef1b188896a70cf8b2f9352d0b76874fd50
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204293"
---
# <a name="connect-data-sources"></a>连接数据源

> [!IMPORTANT]
> Azure Sentinel 目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。



若要载入 Azure Sentinel，首先需要连接到数据源。 Azure Sentinel 随附许多适用于 Microsoft 解决方案的现成可用的连接器，提供实时集成（包括 Microsoft 威胁防护解决方案）和 Microsoft 365 源（包括 Office 365、Azure AD、Azure ATP 和 Microsoft Cloud App Security，等等）。 此外，内置的连接器可以拓宽非 Microsoft 解决方案的安全生态系统。 也可以使用常用事件格式 Syslog 或 REST-API 将数据源与 Azure Sentinel 相连接。  

![数据收集器](./media/collect-data/collect-data-page.png)

## <a name="data-connection-methods"></a>数据连接方法

Azure Sentinel 支持以下数据连接方法：

- **Microsoft 服务**：<br> Microsoft 服务原生是互连的。利用 Azure 基础服务的现成集成，只需单击几点鼠标就能连接以下解决方案：
    - [Office 365](connect-office-365.md)
    - [Azure AD 审核日志和登录](connect-azure-active-directory.md)
    - [Azure 活动](connect-azure-activity.md)
    - [Azure AD Identity Protection](connect-azure-ad-Identity-protection.md)
    - [Azure 安全中心](connect-azure-security-center.md)
    - [Azure 信息保护](connect-azure-information-protection.md)
    - [Azure 高级威胁防护](connect-azure-atp.md)
    - [Cloud App Security](connect-cloud-app-security.md)
    - [Windows 安全事件](connect-windows-security-events.md) 
    - [Windows 防火墙](connect-windows-firewall.md)

- **通过 API 连接外部解决方案**：可以使用联网数据源提供的 API 连接某些数据源。 一般情况下，大多数安全技术都会提供一组 API，通过这些 API 可以检索事件日志。这些 API 连接到 Azure Sentinel，收集特定的数据类型并将其发送到 Azure Log Analytics。 通过 API 连接的设备包括：
    - [Barracuda](connect-barracuda.md)
    - [Symantec](connect-symantec.md)
- **通过代理连接外部解决方案**：可以通过代理使用 Syslog 协议将 Azure Sentinel 连接到可执行实时日志流式处理的其他所有数据源。 <br>大部分设备使用 Syslog 协议发送包含日志本身以及日志相关数据的事件消息。 日志格式各不相同，但大部分设备支持通用事件格式 (CEF) 标准。 <br>基于 OMS 代理的 Azure Sentinel 代理会将 CEF 格式的日志转换为可供 Log Analytics 引入的格式。 根据设备类型，可以直接在设备上安装代理，或者在专用的 Linux 服务器上安装代理。 适用于 Linux 的代理通过 UDP 从 Syslog 守护程序接收事件，但如果预期 Linux 计算机需要收集大量的 Syslog 事件，则会通过 TCP 将这些事件从 Syslog 守护程序发送到代理，然后从代理发送到 Log Analytics。
    - 防火墙、代理和终结点：
        - [F5](connect-f5.md)
        - [Check Point](connect-checkpoint.md)
        - [Cisco ASA](connect-cisco.md)
        - [Fortinet](connect-fortinet.md)
        - [Palo Alto](connect-paloalto.md)
        - [其他 CEF 设备](connect-common-event-format.md)
        - [其他 Syslog 设备](connect-syslog.md)
    - DLP 解决方案
    - [威胁智能提供程序](connect-threat-intelligence.md)
    - [DNS 计算机](connect-dns.md) - 直接安装在 DNS 计算机上的代理
    - Linux 服务器
    - 其他云
    
## 代理连接选项<a name="agent-options"></a>

若要将外部设备连接到 Azure Sentinel，代理必须部署在专用计算机上（VM 或本地），以支持设备与 Azure Sentinel 之间的通信。 可以自动或手动部署代理。 仅当专用计算机是在 Azure 中创建的新 VM 时，才能进行自动部署。 


![Azure 中的 CEF](./media/connect-cef/cef-syslog-azure.png)

或者，可以在现有的 Azure VM 上、在其他云中的 VM 上或者在本地计算机上手动部署代理。

![本地 CEF](./media/connect-cef/cef-syslog-onprem.png)


## <a name="next-steps"></a>后续步骤

- 若要开始使用 Azure Sentinel，需要订阅 Microsoft Azure。 如果尚无订阅，可注册[免费试用版](https://azure.microsoft.com/free/)。
- 了解如何[将数据载入到 Azure Sentinel](quickstart-onboard.md)，以及[获取数据和潜在威胁的见解](quickstart-get-visibility.md)。
