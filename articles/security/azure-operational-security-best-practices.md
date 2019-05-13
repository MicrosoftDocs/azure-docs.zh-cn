---
title: 保护您的资产-Microsoft Azure 的最佳安全方案
description: 本文提供了一系列操作的最佳做法来保护你的数据、 应用程序和 Azure 中的其他资产。
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 4a4677b5db730001df75d201d8e6d3149cb928e6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409849"
---
# <a name="azure-operational-security-best-practices"></a>Azure 操作安全性最佳做法
本文提供了一系列操作的最佳做法来保护你的数据、 应用程序和 Azure 中的其他资产。

最佳做法以观点的共识以及 Azure 平台功能和特性集为基础。 随时间变化的观点和技术和定期以反映这些更改更新这篇文章。

## <a name="define-and-deploy-strong-operational-security-practices"></a>定义和部署强大的操作安全做法
Azure 操作安全性是指用户可用于在 Azure 中保护其数据、应用程序和其他资产的服务、控件和功能。 Azure 操作安全性建立在一个框架上，该框架融合了通过 Microsoft 独有的功能获得的知识，包括 [安全开发生命周期 (SDL)](https://www.microsoft.com/sdl)、[Microsoft 安全响应中心](https://www.microsoft.com/msrc?rtc=1)计划以及对网络安全威胁形态的深刻认识。

## <a name="manage-and-monitor-user-passwords"></a>管理和监视用户密码
下表列出了与管理用户密码相关的一些最佳做法：

**最佳做法**：确保在云中有密码保护的正确级别。   
**详细信息**：遵循中的指导[Microsoft 密码指南](https://www.microsoft.com/research/publication/password-guidance/)，其范围是 Microsoft 标识平台 （Azure Active Directory、 Active Directory 和 Microsoft 帐户） 的用户。

**最佳做法**：监视的用户帐户相关的可疑操作。   
**详细信息**：用于监视[风险的用户](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk)和[风险登录](../active-directory/reports-monitoring/concept-risk-events.md)通过使用 Azure AD 安全报告。

**最佳做法**：自动检测和修正高风险的密码。   
**详细信息**：[Azure AD Identity Protection](../active-directory/identity-protection/overview.md)是一项功能的 Azure AD Premium P2 版，你可以：

- 检测影响组织标识的潜在漏洞
- 配置自动响应，可检测与组织标识相关的可以操作
- 调查可疑的事件并采取适当措施来解决这些问题

## <a name="receive-incident-notifications-from-microsoft"></a>从 Microsoft 接收事件通知
请确保你的安全操作团队可以接收来自 Microsoft Azure 的事件通知。 事件通知允许你安全团队知道已破坏 Azure 资源，以便他们快速响应和修正潜在的安全风险。

在 azure 门户中，可以确保管理员联系信息包括通知安全操作的详细信息。 联系人详细信息为电子邮件地址和电话号码。

## <a name="organize-azure-subscriptions-into-management-groups"></a>组织到管理组的 Azure 订阅
如果你的组织有多个订阅，则可能需要一种方法来高效地管理这些订阅的访问权限、策略和符合性。 [Azure 管理组](../governance/management-groups/create.md)提供的订阅上的作用域级别。 将订阅组织到为管理组容器，并将管理条件应用到管理组。 管理组中的所有订阅都将自动继承应用于管理组的条件。

您可以构建灵活的管理组和订阅的目录结构。 每个目录指定了名为根管理组的单个顶级管理组。 此根管理组内置在层次结构中，包含其所有下级管理组和订阅。 根管理组允许的全局策略和 RBAC 分配，以在目录级别应用。

下面是使用管理组的一些最佳实践：

**最佳做法**：确保新的订阅应用调控元素，如策略和权限，因为当他们添加。   
**详细信息**：使用根管理组分配适用于所有 Azure 资产的企业范围的安全元素。 策略和权限的元素的示例。

**最佳做法**：对齐分段策略的管理组来提供每个段中的控件和策略保持一致的点的顶层。   
**详细信息**：创建的单个管理组的根管理组下的每个段。 不要创建根下的任何其他管理组。

**最佳做法**：限制管理组的深度，以避免混淆，影响操作和安全。   
**详细信息**：限制为三个级别，包括根层次结构。

**最佳做法**：请仔细选择要应用于整个企业与根管理组的项。   
**详细信息**：确保根管理组元素显然需要应用于每个资源，它们是影响最小。

很好的候选包括：

- 具有明确的业务影响 （例如，与数据主权相关的限制） 的法规要求
- 要求与近乎实时的潜在负面影响操作，类似于与审核效果或 RBAC 策略已经过仔细检查的权限分配

**最佳做法**：仔细规划和测试应用 （策略、 RBAC 模型中，等） 之前的所有企业范围的更改，在根管理组上。   
**详细信息**：在根管理组中的更改会影响在 Azure 上的每个资源。 它们提供的一种强大方法确保跨企业的一致性，而错误或不正确的使用情况产生负面影响生产操作。 在测试实验室或生产环境中测试对根管理组的所有更改试验。

## <a name="streamline-environment-creation-with-blueprints"></a>简化使用蓝图创建环境
[Azure 蓝图](../governance/blueprints/overview.md)服务使云架构师和中央信息技术组定义一组可重复的 Azure 资源的实现，并遵循组织的标准、 模式和要求。 Azure 蓝图使开发团队能够快速构建和建立起一组具有的内置组件和它们要创建这些环境中组织符合性的置信度的新环境。

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>监视存储服务的意外行为更改
诊断和排查在云环境中托管的分布式应用程序中的问题可能会比在传统环境中更复杂。 应用程序可以部署在 PaaS 或 IaaS 基础结构、本地、移动设备，或这些环境的某种组合中。 应用程序的网络流量可能会遍历公用和专用网络，你的应用程序可能使用多种存储技术。

应持续监视应用程序使用的存储服务是否存在任何意外行为更改（如响应时间变长）。 若要收集更详细的数据和深度分析问题，请使用日志记录。 从监视和日志记录获取的诊断信息将有助于确定应用程序所遇到问题的根本原因。 然后，可以排查该问题，并确定修正问题的相应步骤。

[Azure 存储分析](../storage/storage-analytics.md)执行日志记录并为 Azure 存储帐户提供指标数据。 建议使用此数据跟踪请求、分析使用情况趋势以及诊断存储帐户的问题。

## <a name="prevent-detect-and-respond-to-threats"></a>防范、检测和应对威胁
[Azure 安全中心](../security-center/security-center-intro.md)可帮助你预防、 检测和响应威胁，通过提供可见性增加 （和控制） 的 Azure 资源的安全性。 它提供对 Azure 订阅的集成安全监视和策略管理，帮助检测可能被忽略的威胁，且适用于各种安全解决方案。

安全中心免费层提供有限的仅 Azure 资源的安全。 标准层将这些功能扩展到本地和其他云中。 安全中心标准层可帮助你查找和修复安全漏洞、 应用访问和应用程序控制以阻止恶意活动，通过使用分析和智能，来检测威胁并受到攻击时迅速地做出响应。 可以尝试安全中心标准版，头 60 天免费。 我们建议您[将 Azure 订阅升级到安全中心标准层](../security-center/security-center-get-started.md)。

使用安全中心，以获得所有 Azure 资源的安全状态的中央视图。 一眼就可验证适当的安全控件是否配置到位且配置正确，还可快速确认任何需要注意的资源。

安全中心集成，还具有[Windows Defender 高级威胁防护 (ATP)](../security-center/security-center-wdatp.md)，其中提供了全面的终结点检测和响应 （edr 规范） 功能。 使用 Windows Defender ATP 集成可以查明异常。 您还可以检测和响应安全中心监视的服务器终结点上的高级攻击。

几乎所有的企业组织都具有一个安全信息和事件管理 (SIEM) 系统以帮助识别新出现的威胁，通过将合并来自不同信号正在收集设备日志信息。 然后，日志所分析的数据分析系统，以帮助确定什么是"有趣"中是不可避免所有日志收集和分析解决方案中的噪音。

[Azure Sentinel](../sentinel/overview.md)是一个可缩放的云本机的安全信息和事件管理 (SIEM) 和安全业务流程自动响应 （骤升至） 解决方案。 Azure Sentinel 提供智能安全分析和威胁智能，通过警报检测、 威胁可见性、 主动搜寻和自动的威胁响应。

下面是阻止、 检测和响应威胁的一些最佳做法：

**最佳做法**：使用基于云的 SIEM 增加的速度和可伸缩性的 SIEM 解决方案。   
**详细信息**：特性和功能的调查[Azure Sentinel](../sentinel/overview.md) ，并将其与当前仍的功能在本地使用。 考虑采用 Azure Sentinel 如果它满足你组织的 SIEM 要求。

**最佳做法**：查找最严重的安全漏洞，以确定调查的优先级。   
**详细信息**：查看你[Azure 安全分数](../security-center/security-center-secure-score.md)若要查看从 Azure 策略和 Azure 安全中心中内置的计划生成的建议。 这些建议有助于地址等安全更新、 endpoint protection、 加密、 安全配置、 缺少 WAF，连接到 internet 的 Vm 和更多的前几大风险。

安全分数，这基于中心的 Internet 安全 (CIS) 控件，可让你组织的 Azure 安全性，以防止外部源进行基准测试。 外部验证可帮助验证和丰富您的团队的安全策略。

**最佳做法**：监视计算机、 网络、 存储和数据服务和应用程序发现并确定潜在的安全问题的优先级的安全状况。  
**详细信息**：请按照[的安全建议](../security-center/security-center-recommendations.md)在安全中心从开始，具有最高优先级项目。

**最佳做法**：将安全中心警报集成到您的安全信息和事件管理 (SIEM) 解决方案。   
**详细信息**：与 SIEM 的大多数组织将其用作中央交换所需要的分析师响应安全警报。 生成的安全中心处理的事件发布到 Azure 活动日志，可通过 Azure Monitor 日志之一。 Azure Monitor 提供了一个综合管道，可将任何监视数据路由到 SIEM 工具。 请参阅[在安全中心集成安全解决方案](../security-center/security-center-partner-integration.md#exporting-data-to-a-siem)有关的说明。 如果使用的 Azure Sentinel，请参阅[连接 Azure 安全中心](../sentinel/connect-azure-security-center.md)。

**最佳做法**：集成 Azure 日志与 SIEM。   
**详细信息**：使用[Azure Monitor 收集数据和导出](../azure-monitor/overview.md#integrate-and-export-data)。 这种做法是启用安全事件调查的关键，并且联机日志保留期限制。 如果使用的 Azure Sentinel，请参阅[数据源连接](../sentinel/connect-data-sources.md)。

**最佳做法**：提高您的调查和搜索过程的速度，通过将终结点检测和响应 （edr 规范） 功能集成到攻击调查减少错误判断。   
**详细信息**：[启用 Windows Defender ATP 集成](../security-center/security-center-wdatp.md#enable-windows-defender-atp-integration)通过安全中心安全策略。 请考虑使用 Azure Sentinel 威胁搜寻和事件响应。

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>监视基于端到端方案的网络监视
客户在 Azure 中通过合并虚拟网络、ExpressRoute、应用程序网关和负载均衡器等网络资源来构建端到端网络。 监视适用于每个网络资源。

[Azure 网络观察程序](../network-watcher/network-watcher-monitoring-overview.md)是一项区域性服务。 其诊断和可视化工具可用于在网络方案级别监视和诊断 Azure 内部以及传入和传出 Azure 的流量的状态。

以下是网络监视和可用工具的最佳做法。

**最佳做法**：使用数据包捕获实现远程网络监视的自动化。  
**详细信息**：使用网络观察程序监视和诊断网络问题，无需登录 VM。 通过设置警报触发[数据包捕获](../network-watcher/network-watcher-alert-triggered-packet-capture.md)，并获取数据包级别上的实时性能信息访问权限。 如果遇到问题，可进行详细调查，获得更精确的诊断。

**最佳做法**：使用流日志深入了解网络流量。  
**详细信息**：使用[网络安全组流日志](../network-watcher/network-watcher-nsg-flow-logging-overview.md)更深入地了解网络流量模式。 流日志中的信息可帮助收集符合性数据、审核和监视网络安全配置文件。

**最佳做法**：诊断 VPN 连接问题。  
**详细信息**：使用网络观察程序来[诊断最常见的 VPN 网关和连接问题](../network-watcher/network-watcher-diagnose-on-premises-connectivity.md)。 不仅可以确定问题，还可以使用详细日志进一步调查。

## <a name="secure-deployment-by-using-proven-devops-tools"></a>使用经验证的 DevOps 工具确保安全部署
使用以下 DevOps 最佳做法来确保企业和团队多产且高效。

**最佳做法**：自动生成和部署服务。  
**详细信息**：[基础结构即代码](https://docs.microsoft.com/azure/devops/learn/what-is-infrastructure-as-code)是一组技术和实践，使 IT 专业人员不再需要日复一日地生成和管理模块化基础结构。 使得 IT 专业人员生成和维护新式服务器环境的方式就像是软件开发人员生成和维护应用程序代码的方式。

[Azure 资源管理器](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)允许用户使用声明性模板预配应用程序。 在单个模板中，可以部署多个服务及其依赖项。 在应用程序生命周期的每个阶段，可使用相同模板重复部署应用程序。

**最佳做法**：自动生成并部署到 Azure Web 应用或云服务。  
**详细信息**：您可以将 Azure DevOps 项目配置为[自动生成并部署](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops)到 Azure web 应用或云服务。 Azure DevOps 自动部署二进制文件后每次代码签入到 Azure 进行生成。 包生成过程与 Visual Studio 中的 Package 命令等效，而发布步骤与 Visual Studio 中的 Publish 命令等效。

**最佳做法**：自动完成发布管理。  
**详细信息**：[Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/index?azure-devops) 是实现多阶段部署和管理发布过程自动化的解决方案。 创建托管的持续部署管道，快速、轻松地频繁发布。 通过 Azure Pipelines，可以使发布过程自动化，还可以拥有预定义的批准工作流。 根据需要进行本地部署和部署到云、扩展和自定义。

**最佳做法**：在推出应用或将更新部署到生产环境之前，先检查该应用的性能。  
**详细信息**：运行基于云的[负载测试](https://docs.microsoft.com/azure/devops/test/load-test/overview.md?view=azure-devops#alternatives)到：

- 在应用中查找性能问题。
- 提高部署质量。
- 请确保应用始终可用。
- 确保应用可以处理下一次启动或市场营销活动的流量。

[Apache JMeter](https://jmeter.apache.org/)是免费的常用开源工具，其强大的社区支持。

**最佳做法**：监视应用程序性能。  
**详细信息**：[Azure Application Insights](../azure-monitor/app/app-insights-overview.md) 是多个平台上面向 Web 开发人员的可扩展应用程序性能管理 (APM) 服务。 使用 Application Insights 来监视实时 Web 应用程序。 它会自动检测性能异常。 其中包含分析工具来帮助诊断问题，了解用户在应用中实际执行了哪些操作。 Application Insights 有助于持续提高性能与可用性。

## <a name="mitigate-and-protect-against-ddos"></a>缓解和防范 DDoS
分布式拒绝服务 (DDoS) 是企图耗尽应用程序资源的一种攻击。 其目的是影响应用程序的可用性和处理合法请求的能力。 这些攻击正变得越来越复杂，且规模和影响程度越来越高。 它们可能会将任何可通过 Internet 公开访问的终结点作为目标。

设计和构建 DDoS 复原能力需要规划和设计各种故障模式。 下面是用于在 Azure 上构建 DDoS 可复原服务的最佳做法。

**最佳做法**：确保优先考虑从设计和实施到部署和操作的整个应用程序生命周期的安全性。 应用程序可能包含 bug，使相对较少的请求使用过多的资源，从而导致服务中断。  
**详细信息**：为帮助保护 Microsoft Azure 上运行的服务，应该对应用程序体系结构有充分的了解，并重点关注[软件质量的五大要素](https://docs.microsoft.com/azure/architecture/guide/pillars)。 应该清楚典型的流量大小、应用程序与其他应用程序之间的连接模型，以及向公共 Internet 公开的服务终结点。

至关重要的一点是，确保应用程序具有足够的弹性，可应对针对应用程序本身的拒绝服务攻击。 从[安全开发生命周期 (SDL)](https://www.microsoft.com/sdl) 开始，安全和隐私就已内置到 Azure 平台中。 SDL 可以解决每个开发阶段的安全性，并确保 Azure 不断更新，以变得越来越安全。

**最佳做法**：采用可[横向缩放](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out)的应用程序设计，以满足放大负载的需求，尤其是防范 DDoS 攻击。 如果应用程序依赖于服务的单个实例，则会造成单一故障点。 预配多个实例能够提高复原能力和可伸缩性。  
**详细信息**：对于 [Azure 应用服务](../app-service/app-service-value-prop-what-is.md)，请选择提供多个实例的[应用服务计划](../app-service/overview-hosting-plans.md)。

对于 Azure 云服务，请将每个角色配置为使用[多个实例](../cloud-services/cloud-services-choose-me.md)。

对于 [Azure 虚拟机](../virtual-machines/windows/overview.md)，请确保 VM 体系结构包含多个 VM，并且每个 VM 包含在[可用性集](../virtual-machines/virtual-machines-windows-manage-availability.md)中。 建议使用虚拟机规模集来实现自动缩放功能。

**最佳做法**：应用程序中的分层安全防御可以减少攻击成功的可能性。 使用 Azure 平台的内置功能对其应用程序实施安全设计。  
**详细信息**：攻击风险会随着应用程序规模（外围应用）的增大而增大。 可以使用允许列表关闭负载均衡器（[Azure 负载均衡器](../load-balancer/load-balancer-get-started-internet-portal.md)和 [Azure 应用程序网关](../application-gateway/application-gateway-create-probe-portal.md)）上不需要的公开 IP 地址空间和侦听端口，来减少外围应用。

[网络安全组](../virtual-network/security-overview.md)是缩小受攻击面的另一种方法。 可以使用[服务标记](../virtual-network/security-overview.md#service-tags)和[应用程序安全组](../virtual-network/security-overview.md#application-security-groups)来最大程度地简化安全规则的创建，并将网络安全性配置为应用程序结构的自然扩展。

应尽可能地在[虚拟网络](../virtual-network/virtual-networks-overview.md)中部署 Azure 服务。 这种做法可让服务资源通过专用 IP 地址通信。 来自虚拟网络的 Azure 服务流量默认使用公共 IP 地址作为源 IP 地址。

使用[服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md)时，服务流量会在通过虚拟网络访问 Azure 服务时改用虚拟网络专用地址作为源 IP 地址。

我们经常看到，客户本地资源会连同其在 Azure 中资源的一起受到攻击。 如果将本地环境连接到 Azure，尽量不要在公共 Internet 上公开本地资源。

Azure 具有两个 DDoS [服务产品](../virtual-network/ddos-protection-overview.md)，提供网络攻击防护：

- 基本防护默认已集成到 Azure 中，不收取额外的费用。 全球部署的 Azure 网络的规模和容量通过始终开启的监视和实时缓解措施，来防御公用网络层攻击。 基本防护无需用户配置或应用程序更改，并帮助保护所有 Azure 服务，包括 Azure DNS 等 PaaS 服务。
- 标准防护提供针对网络攻击的高级 DDoS 缓解功能。 这些功能自动经过优化，可保护特定的 Azure 资源。 在创建虚拟网络期间，可以轻松启用保护。 也可以在创建之后启用它，而不需要对应用程序或资源做出任何更改。

## <a name="enable-azure-policy"></a>启用 Azure 策略
[Azure 策略](../governance/policy/overview.md)是用于创建、 分配和管理策略的 Azure 中的服务。 这些策略规则和效果对资源强制实施，因此这些资源始终符合公司标准和服务级别协议。 Azure Policy 通过评估资源是否符合指定策略来满足此需求。

启用 Azure 策略以监视和强制执行组织的书面的策略。 通过集中管理混合云工作负荷的安全策略，这将确保向你的公司的合规性或法规安全要求。 了解如何[创建和管理策略以强制实施符合性](../governance/policy/tutorials/create-and-manage.md)。 请参阅[Azure 策略定义结构](../governance/policy/concepts/definition-structure.md)有关策略的元素的概述。

下面是一些最佳安全做法遵循后采用 Azure 策略：

**最佳做法**：策略支持多种类型的效果。 可以阅读有关中对其[Azure 策略定义结构](../governance/policy/concepts/definition-structure.md#policy-rule)。 可能由有负面影响业务运营**拒绝**效果和**修正**起作用，因此开始**审核**效果来限制从负面影响的风险策略。   
**详细信息**：[在审核模式下启动策略部署](../governance/policy/concepts/definition-structure.md#policy-rule)，然后到更高版本进度**拒绝**或**修正**。 测试和查看审核效果的结果，然后将其移到**拒绝**或**修正**。

有关详细信息，请参阅[创建和管理策略以强制实施符合性](../governance/policy/tutorials/create-and-manage.md)。

**最佳做法**：确定负责监视存在策略违规，并确保迅速采取正确的修正操作的角色。   
**详细信息**：已通过分配的角色监视符合性[Azure 门户](../governance/policy/how-to/get-compliance-data.md#portal)或通过[命令行](../governance/policy/how-to/get-compliance-data.md#command-line)。

**最佳做法**：Azure 策略是组织的书面策略的技术表示形式。 将所有 Azure 策略映射到组织的策略以减少混乱并提高一致性。   
**详细信息**：文档映射组织的文档中或位于 Azure 策略本身通过在 Azure 中添加对组织的策略的引用[策略说明](../governance/policy/concepts/definition-structure.md#display-name-and-description)或 Azure 策略[计划](../governance/policy/concepts/definition-structure.md#initiatives)说明。

## <a name="monitor-azure-ad-risk-reports"></a>监视 Azure AD 的风险报告
大多数安全违规出现在当攻击者通过窃取用户的标识来获取环境的访问权限时。 发现标识是否遭到入侵并不容易。 Azure AD 使用自适应机器学习算法和试探法来检测与用户帐户相关的可疑操作。 检测到的每个可疑操作都存储在称为“风险事件”[](../active-directory/reports-monitoring/concept-risk-events.md)的记录中。 风险事件记录在 Azure AD 安全报告。 详细信息，请阅读有关[用户风险安全报告](../active-directory/reports-monitoring/concept-user-at-risk.md)并[风险登录安全报告](../active-directory/reports-monitoring/concept-risky-sign-ins.md)。

## <a name="next-steps"></a>后续步骤
有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](security-best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com
