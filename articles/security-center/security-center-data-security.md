---
title: Azure 安全中心数据安全性 | Microsoft Docs
description: 本文档介绍如何在 Azure 安全中心管理数据和确保数据安全性。
services: security-center
documentationcenter: na
author: rkarlin
manager: barbkess
editor: ''
ms.assetid: 33f2c9f4-21aa-4f0c-9e5e-4cd1223e39d7
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/28/2018
ms.author: rkarlin
ms.openlocfilehash: cd91b83bc808d811fc50293fbf1726d609ad5b46
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234085"
---
# <a name="azure-security-center-data-security"></a>Azure 安全中心数据安全性
为了帮助客户防止、检测和应对威胁，Azure 安全中心将收集和处理安全相关的数据，包括配置信息、元数据、事件日志、故障转储文件，等等。 从编程到服务运营，Microsoft 都严格遵守相关法规与安全准则。

本文介绍如何在 Azure 安全中心管理数据和确保数据安全性。

## <a name="data-sources"></a>数据源
Azure 安全中心将分析以下源中的数据，提供安全状态视图、识别漏洞、建议缓解措施，并检测现行的威胁：

- Azure 服务：通过与 Azure 服务的资源提供程序通信，使用已部署的 Azure 服务的配置信息。
- 网络流量：使用从 Microsoft 基础结构中采样的网络流量元数据，例如源/目标 IP/端口、数据包大小以及网络协议。
- 合作伙伴解决方案：使用来自集成合作伙伴解决方案（例如防火墙和反恶意软件解决方案）的安全警报。
- 虚拟机和服务器：使用虚拟机中的配置信息，以及有关安全事件的信息，例如 Windows 事件和审核日志、IIS 日志、系统日志消息和故障转储文件。 此外，在创建警报时，Azure 安全中心可以生成受影响 VM 磁盘的快照，并从 VM 磁盘提取与该警报相关的、用于取证目的的计算机项目，例如注册表文件。


## <a name="data-protection"></a>数据保护
**数据隔离**：服务中每个组件的数据都保持逻辑隔离。 所有数据均按组织进行标记。 此标记方式贯穿数据的整个生命周期，在服务的每个层强制实施。

**数据访问**：为了提供安全建议及调查潜在安全威胁，Microsoft 人员可能会访问 Azure 服务收集或分析的信息，包括故障转储文件、进程创建事件、VM 磁盘快照和项目，其中可能会意外地包括虚拟机中的客户数据或个人数据。 我们遵守 [Microsoft Online Services 条款和隐私声明](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)，其中指出，Microsoft 不会出于广告或类似的商业目的利用客户数据，或者从客户数据衍生信息。 我们只会根据需要将客户数据用于向用户提供 Azure 服务，包括用于与提供这些服务相对应的目的。 用户保留对客户数据的所有权限。

**数据使用**：Microsoft 使用多个租户所使用的模式和威胁情报增强预防和检测威胁的能力；执行过程中遵循[隐私声明](https://www.microsoft.com/privacystatement/en-us/OnlineServices/Default.aspx)中所述的隐私承诺。

## <a name="data-location"></a>数据位置

**工作区**：为以下地区指定工作区，并且从 Azure 虚拟机收集的数据（包括故障转储和某些类型的警报数据）都存储在最近的工作区中。

| VM 地区                              | 工作区地区 |
|-------------------------------------|---------------|
| 美国、 巴西，南非 | 美国 |
| 加拿大                              | 加拿大        |
| 欧洲 （不包括英国）   | 欧洲        |
| 英国                      | 英国 |
| 亚太地区 （不包括印度、 日本、 韩国、 中国）   | 亚太区  |
| 韩国                              | 亚太区  |
| 印度                               | 印度         |
| 日本                               | 日本         |
| 中国                               | 中国         |
| 澳大利亚                           | 澳大利亚     |


VM 磁盘快照存储在 VM 磁盘所在的同一存储帐户中。

对于在其他环境中（例如本地）运行的虚拟机和服务器，可以指定存储收集的数据的工作区和区域。

**Azure 安全中心存储**：安全警报（包括合作伙伴警报）相关信息根据相关的 Azure 资源位置在区域范围内进行存储，而安全运行状况状态和建议的相关信息则根据客户位置集中存储在美国或欧洲。
Azure 安全中心收集故障转储文件的临时副本并对其进行分析，目的是找出攻击者尝试利用漏洞并成功进行了破坏的证据。 Azure 安全中心在工作区所在的同一地理位置执行此分析，分析完成后，将删除临时副本。

计算机项目集中存储在 VM 所在的同一区域。


## <a name="managing-data-collection-from-virtual-machines"></a>管理从虚拟机进行的数据收集

在 Azure 中启用安全中心后，即为每个 Azure 订阅启用了数据收集功能。 还可在 Azure 安全中心的“安全策略”部分，为订阅启用数据收集。 启用数据收集功能后，Azure 安全中心即可在所有受支持的现有 Azure 虚拟机以及任何新创建的虚拟机中预配 Microsoft Monitoring Agent。
Microsoft Monitoring Agent 扫描各种安全相关配置和事件，并将其收集到 [Windows 事件跟踪](https://msdn.microsoft.com/library/windows/desktop/bb968803.aspx) (ETW) 的跟踪中。 另外，在运行计算机的过程中，操作系统会引发事件日志事件。 此类数据的示例包括：操作系统类型和版本、操作系统日志（Windows 事件日志）、正在运行的进程、计算机名称、IP 地址、已登录用户、租户 ID。 Microsoft Monitoring Agent 读取事件日志条目和 ETW 跟踪，并将其复制到工作区进行分析。 Microsoft Monitoring Agent 还可将故障转储文件复制到工作区、启用进程创建事件，以及启用命令行审核。

如果使用 Azure 安全中心免费版，也可以在“安全策略”中从虚拟机禁用数据收集。 标准层上的订阅需启用数据收集。 即使禁用数据收集，也仍会启用 VM 磁盘快照和项目收集。

## <a name="data-consumption"></a>数据使用

客户可以使用与安全中心相关的数据，这些数据来自不同的数据流，如下所示：

* **Azure 活动**：所有安全警报、经审核的安全中心[实时](https://docs.microsoft.com/azure/security-center/security-center-just-in-time)请求，以及由[自适应应用程序控件](https://docs.microsoft.com/azure/security-center/security-center-adaptive-application)生成的所有警报。
* **Azure Monitor 日志**： 所有安全警报。


> [!NOTE]
> 也可通过 REST API 使用安全建议。 有关详细信息，请阅读 [Security Resource Provider REST API Reference](https://msdn.microsoft.com/library/mt704034(Azure.100).aspx)（安全资源提供程序 REST API 参考）。

## <a name="see-also"></a>另请参阅
本文档介绍如何在 Azure 安全中心管理数据和确保数据安全性。 若要详细了解 Azure 安全中心，请参阅：

* [Azure Security Center Planning and Operations Guide](security-center-planning-and-operations-guide.md) （Azure 安全中心规划和操作指南）- 了解如何规划并理解设计注意事项，以便采用 Azure 安全中心。
* [Azure 安全中心的安全性运行状况监视](security-center-monitoring.md) - 了解如何监视 Azure 资源的运行状况
* [管理和响应 Azure 安全中心的安全警报](security-center-managing-and-responding-alerts.md) - 了解如何管理和响应安全警报
* [Monitoring partner solutions with Azure Security Center](security-center-partner-solutions.md) （通过 Azure 安全中心监视合作伙伴解决方案）- 了解如何监视合作伙伴解决方案的运行状况。
* [Azure 安全中心常见问题解答](security-center-faq.md) - 查找有关使用服务的常见问题
* [Azure 安全性博客](https://blogs.msdn.com/b/azuresecurity/) - 查找关于 Azure 安全性及合规性的博客文章
