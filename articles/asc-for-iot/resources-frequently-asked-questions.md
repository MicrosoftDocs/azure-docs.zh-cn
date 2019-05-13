---
title: 常见 IoT 预览版 Azure 安全中心的问题 |Microsoft Docs
description: 找到解答有关常见问题 Azure 安全中心的 IoT 功能和服务。
services: asc-for-iot
ms.service: ASCforiot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: 97fda6c2-1ecb-491f-b48d-41788bd7e0d3
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/25/2019
ms.author: mlottner
ms.openlocfilehash: f15cd14ddc3a489db3abbbd2665d242470b5a821
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65200710"
---
# <a name="azure-security-center-for-iot-frequently-asked-questions"></a>适用于 IoT 的 azure 安全中心常见问题  

> [!IMPORTANT]
> 适用于 IoT 的 Azure 安全中心目前为公共预览版。
> 此预览版不附带服务级别协议，我们不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

本文提供适用于 IoT 的一些常见问题和解答有关 Azure 安全中心 (ASC) 列表。 

## <a name="does-azure-provide-support-for-iot-security"></a>Azure 是否为保障 IoT 的安全提供支持？

Azure 提供了用于监视和管理 IoT 安全性作为整体安全解决方案通过 Azure 安全中心的一部分的集成的视图。 如果你是应用程序开发人员，您可以使用 IoT 中心视图来管理你的 IoT 应用程序安全性。

## <a name="what-is-azures-unique-value-proposition-for-iot-security"></a>什么是 Azure 的独特的价值主张，为 IoT 安全性？

适用于 IoT 的 ASC 使企业能够扩展到整个 IoT 解决方案其现有网络安全视图。 Azure 提供了您的业务解决方案，使您能够利用与业务相关的操作的端到端视图和决策基于您企业的安全状况，收集的数据。 通过结合使用 Azure IoT、 Azure IoT Edge、 Azure Sphere、 Azure 中心的安全和 Azure 安全中心使您能够创建所需的解决方案具有所需的安全。

## <a name="who-is-asc-for-iot-made-for"></a>对于针对 IoT 进行，ASC 是谁？ 

Iot ASC 集成在 Azure IoT 中心安全性并提供对日常业务解决方案的管理安全操作。 适用于 IoT 的 ASC 还集成到 Azure 安全中心功能和用于监视和管理 IoT 安全性作为整体安全解决方案的一部分提供的集成的视图。

## <a name="how-does-asc-for-iot-compare-to-the-competition"></a>适用于 IoT 的 ASC 与竞争对手相比如何？

虽然其他解决方案提供一套功能，使客户能够创建他们自己的解决方案，ASC 为 IoT 提供了一个唯一的端到端 IoT 安全解决方案，跨所有相关 Azure 资源的安全提供较宽的视图。 Azure，可以快速部署和与 IoT 中心模块孪生的与现有设备管理工具轻松集成完全集成。

## <a name="do-i-have-to-be-an-azure-security-center-asc-customer"></a>我是否必须是 Azure 安全中心 (ASC) 客户？

不可以，但建议。 而无需 ASC，ASC iot 接收有限的连接的资源数据，并提供潜在的攻击面、 威胁和潜在攻击的有限的分析。 

## <a name="do-i-have-to-be-an-azure-iot-customer"></a>我是否必须是 Azure IoT 客户？

可以。 适用于 IoT 的 ASC 依赖于 Azure IoT 连接和基础结构。

## <a name="do-i-have-to-install-an-agent"></a>我是否需要安装代理？

将 IoT 设备上的安装代理并不强制性启用 Microsoft ASC iot 要求。 你可以选择以下三个选项，获得根据你选择的监视和管理功能的不同级别的安全性：

1. 安装 IoT 安全代理 ASC 使用或不进行修改。 此选项提供增强的安全性深入了解设备行为和访问的最高级别。

2. 创建您自己的代理并为 IoT 安全消息架构实现 Microsoft ASC。 此选项启用 Microsoft ASC 的 IoT 设备安全代理程序之上的分析工具的使用的情况。

3. IoT 设备上没有安全代理安装。 此选项使 IoT 中心通信，使用监视降低的安全监视和管理功能。 

## <a name="what-does-the-asc-for-iot-agent-do"></a>ASC 为 IoT 代理有什么作用？

ASC IoT 代理提供设备配置、 行为和访问 （通过扫描配置）、 进程和连接的设备级别的威胁覆盖范围。 IoT 安全代理 ASC 不会扫描与业务相关的数据或活动。

## <a name="where-i-can-get-the-asc-for-iot-security-agent"></a>从何处可以获取有关 IoT 安全代理 ASC？

IoT 安全代理 ASC 是开放源代码并在 GitHub 上在 32 位和 64 位 Windows 和 Linux 版本中可用： https://github.com/Azure/Azure-Security-IoT-Preview

## <a name="where-does-the-asc-for-iot-agent-get-installed"></a>其中 IoT 代理 ASC 获取在安装？ 

可以在 GitHub 中找到详细的安装和代理部署信息： https://github.com/Azure/Azure-Security-IoT-Preview

## <a name="what-are-the-dependencies-and-prerequisites-of-the-agent"></a>依赖项和代理的先决条件是什么？

适用于 IoT 的 ASC 支持各种不同的平台。 请参阅[受支持设备平台](how-to-deploy-agent.md)来验证对特定设备的支持。 

## <a name="which-data-is-collected-by-the-agent"></a>由代理收集哪些数据？

连接、 访问、 防火墙配置、 进程列表和 OS 基线由代理收集。

## <a name="how-much-data-will-the-agent-generate"></a>代理将生成数据量？

代理数据生成根据设备、 应用程序、 连接类型和客户代理配置。 由于设备与 IoT 解决方案之间的高可变性，我们建议首先部署实验室或测试设置来观察、 学习并设置适合您的需求，同时测量生成的数据量的特定配置中的代理。 启动该服务后, ASC 为 IoT 代理提供了有关优化代理吞吐量操作建议来帮助你进行配置和自定义过程。

## <a name="how-can-i-control-my-billing"></a>如何控制我计费？

ASC 为 IoT 提供了可配置的代理，扫描、 数据缓冲区和创建自定义警报，增加或减少代理生成的数据量的功能。

## <a name="do-agent-messages-use-up-quota-from-iot-hub"></a>是否会从 IoT 中心的配额使用代理消息？

可以。 代理传输的数据以你的 IoT 中心配额计数。 

## <a name="what-next-ive-installed-an-agent-and-dont-see-any-activities-or-logs"></a>接下来要做什么？ 我已安装代理，并且看不到任何活动或日志

1. 检查[代理类型适合你的设备的指定的操作系统平台](how-to-deploy-agent.md)

1. 确认[在设备上运行代理](how-to-agent-configuration.md)。

2. 检查[服务已成功启用](quickstart-onboard-iot-hub.md)到**安全**在 IoT 中心。 

3. 检查设备是否[ASC 为 IoT 模块与 IoT 中心中配置](quickstart-create-security-twin.md)。  

如果活动或日志仍不可用，请为更多帮助的 IoT 合作伙伴联系你 ASC。

## <a name="what-happens-when-the-internet-connection-stops-working"></a>当 internet 连接会停止工作时，会发生什么情况？

代理继续运行并存储数据，只要设备正在运行。 数据存储在根据大小配置的安全消息缓存。 当设备重新获得连接时，安全消息将继续发送。 

## <a name="if-the-device-is-restarted-will-the-security-agent-self-recover"></a>如果重新启动设备时，将安全代理自行恢复？

安全代理用于与每个设备重新启动的自动重新运行。

## <a name="can-the-agent-affect-the-performance-of-the-device-or-other-installed-software"></a>代理可能会影响设备或其他已安装的软件的性能？

该代理与任何其他应用程序/进程占用的计算机资源，并应不会中断正常的设备活动。 其安装和配置与结合使用的代理运行在设备上的资源消耗。 我们建议在包含环境中，以及与其他 IoT 应用程序和功能，互操作性测试代理配置，然后再尝试在生产环境中部署。

## <a name="im-making-some-maintenance-on-the-device-can-i-turn-off-the-agent"></a>在设备上，我正在进行某些维护。 可以关闭代理？

无法关闭代理。

## <a name="is-there-a-way-to-test-if-the-agent-is-working-correctly"></a>有一种方法来测试代理是否正常工作？ 

如果代理停止通信，或者无法发送安全消息**设备是无提示**生成警报。

## <a name="can-i-create-my-own-alerts"></a>可以创建我自己的警报？

可以。 可以在组预先确定的行为，例如 IP 地址和打开的端口上设置自定义的警报。 请参阅[创建自定义警报](quickstart-create-custom-alerts.md)若要了解有关自定义警报以及如何使它们的详细信息。 

## <a name="where-can-i-see-logs-can-i-customize-logs"></a>在何处可以查看日志？ 可以自定义日志？

- 查看警报和建议使用已连接的 Log Analytics 工作区。 在工作区中配置的存储大小和持续时间。

- 此外可以在 Log Analytics 帐户中存储安全代理中的原始数据。 更改此选项的配置之前，请考虑大小、 持续时间、 存储要求和关联的成本。 

## <a name="why-should-i-add-asc-for-iot-to-the-module-identity-what-is-it-used-for"></a>为什么应添加 ASC iot 模块标识？ 用途

为 IoT 模块 ASC 用于代理配置和管理。


## <a name="next-steps"></a>后续步骤

若要详细了解如何开始使用 ASC iot，请参阅以下文章：


- 适用于 IoT 读取 ASC[概述](overview.md)
- 验证[服务先决条件](service-prerequisites.md)
- 了解有关如何对[开始](getting-started.md)
- 了解[ASC 为 IoT 安全警报](concept-security-alerts.md)

