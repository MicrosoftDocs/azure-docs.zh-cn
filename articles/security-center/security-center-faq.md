---
title: Azure 安全中心常见问题 (FAQ) | Microsoft Docs
description: 此 FAQ 解答有关 Azure 安全中心的问题。
services: security-center
documentationcenter: na
author: monhaber
manager: barbkess
editor: ''
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/19/2019
ms.author: monhaber
ms.openlocfilehash: a029ad3fc253f35fb34bc2dd0ddbeb9614e5dc33
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231418"
---
# <a name="azure-security-center-frequently-asked-questions-faq"></a>Azure 安全中心常见问题 (FAQ)
此 FAQ 解答有关 Azure 安全中心的问题。Azure 安全中心可帮助预防、检测和响应威胁，同时增加 Microsoft Azure 资源的可见性和安全方面的可控性。

> [!NOTE]
> 自 2017 年 6 月初开始，安全中心将使用 Microsoft Monitoring Agent 来收集和存储数据。 若要了解详细信息，请参阅 [Azure 安全中心平台迁移](security-center-platform-migration.md)。 本文中的信息表示转换到 Microsoft Monitoring Agent 后的安全中心功能。
>
>

## <a name="general-questions"></a>一般问题
### <a name="what-is-azure-security-center"></a>什么是 Azure 安全中心？
Azure 安全中心有助于预防、检测和响应威胁，同时增加 Azure 资源的可见性和安全可控性。 该服务提供订阅之间的集成安全监视和策略管理，帮助检测可能被忽略的威胁，且适用于广泛的安全解决方案生态系统。

### <a name="how-do-i-get-azure-security-center"></a>如何获取Azure 安全中心？
Azure 安全中心通过 Microsoft Azure 订阅启用，可从 [Azure 门户](https://azure.microsoft.com/features/azure-portal/)访问。 （[登录到门户](https://portal.azure.com)，选“浏览”，并滚动到“安全中心”）。  

## <a name="billing"></a>计费
### <a name="how-does-billing-work-for-azure-security-center"></a>Azure 安全中心如何计费？
安全中心分两个层提供：

免费层提供 Azure 资源、基本安全策略、安全建议以及与合作伙伴安全产品和服务的集成的安全状态。

标准层增添了高级威胁检测功能，包括威胁情报、行为分析、异常检测、安全事件和威胁归属报告。 您可以开始标准层免费试用版。 若要升级，请选择安全策略中的[定价层](https://docs.microsoft.com/azure/security-center/security-center-pricing)。 若要了解详细信息，请参阅[定价页](https://azure.microsoft.com/pricing/details/security-center/)。

### <a name="how-can-i-track-who-in-my-organization-performed-pricing-tier-changes-in-azure-security-center"></a>如何跟踪我的组织中执行在 Azure 安全中心的定价层更改
因为 Azure 订阅可能具有多个管理员有权更改定价层，用户可能想要知道谁执行了定价层更改。 若要使用的可以使用 Azure 活动日志。 请查看更多说明[此处](https://techcommunity.microsoft.com/t5/Security-Identity/Tracking-Changes-in-the-Pricing-Tier-for-Azure-Security-Center/td-p/390832)

## <a name="permissions"></a>权限
Azure 安全中心使用[基于角色的访问控制 (RBAC)](../role-based-access-control/role-assignments-portal.md) 提供可在 Azure 中分配给用户、组和服务的[内置角色](../role-based-access-control/built-in-roles.md)。

安全中心会评估资源的配置以识别安全问题和漏洞。 如果分配有资源所属的订阅或资源组的“所有者”、“参与者”或“读取者”角色，则仅可在安全中心看到与资源相关的信息。

若要深入了解安全中心中的角色和允许的操作，请参阅 [Azure 安全中心中的权限](security-center-permissions.md)。

## <a name="data-collection-agents-and-workspaces"></a>数据收集、代理和工作区
安全中心从 Azure 虚拟机 (Vm)、 虚拟机规模集 (VMSS)、 IaaS 容器和监视安全漏洞和威胁 （包括在本地） 非 Azure 计算机收集数据。 数据是使用 Microsoft Monitoring Agent 收集的，它从计算机中读取各种安全相关的配置和事件日志，然后将数据复制到工作区以进行分析。

### <a name="am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center"></a>我的计费通过安全中心创建的工作区的 Azure Monitor 日志？
不。 安全中心创建的工作区配置为按节点计费的 Azure Monitor 日志时不会产生 Azure Monitor 日志费用。 安全中心的计费始终依据工作区上安装的以下安全中心安全策略和解决方案：

- **免费层** – 安全中心在默认工作区中启用“SecurityCenterFree”解决方案。 免费层不会产生费用。
- **标准层** – 安全中心在默认工作区中启用“Security”解决方案。

有关详细信息，请参阅[安全中心 定价](https://azure.microsoft.com/pricing/details/security-center/)。 定价页说明了 2017 年 6 月起安全数据存储和按比例计费的更改。

> [!NOTE]
> 安全中心创建的工作区的定价层的 log analytics 不会影响安全中心计费。
>
>

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

### <a name="what-qualifies-a-vm-for-automatic-provisioning-of-the-microsoft-monitoring-agent-installation"></a>VM 需要满足哪些条件才能在其上自动预配 Microsoft Monitoring Agent 安装？
Windows 或 Linux IaaS VM 的合格条件如下：

- 当前未在该 VM 上安装 Microsoft Monitoring Agent 扩展。
- 该 VM 处于正在运行状态。
- Windows 或 Linux [Azure 虚拟机代理](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows)安装。
- 该 VM 未用作应用程序，例如 Web 防火墙或下一代防火墙。

### <a name="can-i-delete-the-default-workspaces-created-by-security-center"></a>是否可以删除安全中心创建的默认工作区？
建议不要删除默认工作区。 安全中心使用默认工作区存储 VM 的安全数据。  删除工作区后，安全中心无法收集此数据，因此无法提供某些安全建议和提示。

若要恢复，请删除连接到已删除工作区的 VM 上的 Microsoft Monitoring Agent。 安全中心会重新安装代理并创建新的默认工作区。

### <a name="how-can-i-use-my-existing-log-analytics-workspace"></a>我如何使用现有的 Log Analytics 工作区？

可以选择现有 Log Analytics 工作区，用于存储安全中心收集的数据。 使用现有 Log Analytics 工作区的先决条件：

- 工作区必须与选定 Azure 订阅相关联。
- 至少必须拥有读取权限，才能访问工作区。

选择现有 Log Analytics 工作区的具体步骤：

1. 在“安全策略 - 数据收集”下，选择“使用另一个工作区”。

   ![使用另一个工作区][5]

2. 从下拉菜单中，选择一个工作区，用于存储所收集的数据。

   > [!NOTE]
   > 下拉菜单中只显示用户有权访问且 Azure 订阅包含的工作区。
   >
   >

3. 选择“保存”。
4. 选择“保存”后，系统就会询问是否要重新配置受监视的 VM。

   - 如果只希望在新 VM 上应用新的工作区设置，请选择“否”。 新的工作区设置只会应用于新的代理安装；新发现的 VM 没有安装 Microsoft Monitoring Agent。
   - 如果希望在所有 VM 上应用新的工作区设置，请选择“是”。 此外，所有连接到安全中心创建的工作区的 VM 也都会重新连接到新的目标工作区。

   > [!NOTE]
   > 如果选择“是”，不得删除安全中心创建的工作区，除非所有 VM 已重新连接到新的目标工作区。 如果过早删除工作区，此操作将会失败。
   >
   >

   - 选择“取消”，以取消该操作。

### 如果已作为扩展在 VM 上安装 Microsoft Monitoring Agent？<a name="mmaextensioninstalled"></a>
作为扩展安装监视代理时，扩展配置允许向单个工作区报告。 安全中心不会覆盖用户工作区的现有连接。 安全中心将在已连接的工作区中存储 VM 的安全数据，前提是已在其上安装"安全性"或"SecurityCenterFree"解决方案。 安全中心可以升级到此过程中的最新版本的扩展版本。

有关详细信息，请参阅[在代理安装的预先存在的情况下自动预配](security-center-enable-data-collection.md#preexisting)。


### 如果我有 Microsoft Monitoring Agent 是直接安装在计算机上，但不是作为扩展 （直接代理）？<a name="directagentinstalled"></a>
如果直接 （而不是作为 Azure 扩展） 在 VM 上安装 Microsoft Monitoring Agent，则安全中心将安装 Microsoft Monitoring Agent 扩展，并可以升级到最新版本的 Microsoft Monitoring agent。
安装的代理将继续向其已配置的工作区，报告，此外将报告到安全中心中配置的工作区 （支持的多宿主功能）。
如果配置的工作区是用户工作区 （不安全中心的默认工作区），你将需要安装"安全 /"SecurityCenterFree"解决方案，它为安全中心从虚拟机和计算机启动处理事件报告到的工作区。

为现有机上订阅载入到安全中心之前 2019年-03-17，当将检测到现有代理，将不会安装 Microsoft Monitoring Agent 扩展并在计算机不会受到影响。 这些计算机，请参阅"解决监视你的计算机上的代理运行状况问题"建议，若要解决在这些计算机上的代理安装问题

 有关详细信息，请参阅下一部分[如果已在 VM 上安装 SCOM 或 OMS 直接代理，会发生什么情况？](#scomomsinstalled)

### 如果我的 VM 上已安装 System Center Operations Manager (SCOM) 代理，会发生什么情况？<a name="scomomsinstalled"></a>
安全中心会安装到现有的 System Center Operations Manager 代理的 Microsoft Monitoring Agent 扩展的并排方案。 现有 SCOM 代理将继续正常情况下报告到 System Center Operations Manager 服务器。 请注意，System Center Operations Manager 代理和 Microsoft Monitoring Agent 共享常见的运行时库，将在此处理期间更新到最新版本。 请注意-如果 System Center Operations Manager 安装代理版本 2012年，则不要启用自动预配上 （在 System Center Operations Manager 服务器也是 2012年时，可管理性功能可能会丢失）。

### <a name="what-is-the-impact-of-removing-these-extensions"></a>删除这些扩展会有什么影响？
删除 Microsoft 监视扩展后，安全中心无法从 VM 收集安全数据，因此无法提供某些安全建议和提示。 安全中心会在 24 小时内确定 VM 缺少扩展并重新安装扩展。

### <a name="how-do-i-stop-the-automatic-agent-installation-and-workspace-creation"></a>如何停止自动安装代理和创建工作区？
可以在安全策略中为订阅禁用自动预配，但不建议这样做。 禁用自动预配会限制安全中心的建议和警报。 禁用自动预配的具体步骤：

1. 如果订阅配置为标准层，则打开该订阅的安全策略，并选择“免费”层。

   ![定价层][1]

2. 接下来，在“安全策略 - 数据收集”边栏选项卡上选择“关”，禁用自动预配。
   ![数据收集][2]

### <a name="should-i-opt-out-of-the-automatic-agent-installation-and-workspace-creation"></a>是否应选择禁用自动安装代理和创建工作区？

> [!NOTE]
> 如果选择禁用自动预配，请务必查看[选择禁用有何影响](#what-are-the-implications-of-opting-out-of-automatic-provisioning)和[选择禁用时的建议步骤](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)部分。
>
>

如果存在以下情况，可以选择禁用自动预配：

- 安全中心执行的自动代理安装应用到整个订阅。  无法将自动安装应用到一部分 VM。 如果使用 Microsoft Monitoring Agent 无法安装某些关键 VM，则应选择禁用自动预配。
- Microsoft Monitoring Agent (MMA) 扩展的安装会更新代理的版本。 这适用于直接代理和 SCOM 代理 （在后者中，SCOM 和 MMA 共享常见的运行时库-将更新过程中）。 如果已安装的 SCOM 代理版本为 2012 并已将其升级，则当 SCOM 服务器的版本也是 2012 时，可能会丢失管理功能。 如果已安装的 SCOM 代理版本为 2012，则应考虑选择禁用自动预配。
- 如果自定义工作区在订阅的外部（集中式工作区），则应选择禁用自动预配。 可以手动安装 Microsoft Monitoring Agent 扩展并将其连接到工作区，无需安全中心覆盖连接。
- 如果想要避免为每个订阅创建多个工作区，并且订阅中包含自己的自定义工作区，则可以采取以下两种做法：

   1. 可以选择禁用自动预配。 迁移后，根据[如何使用现有的 Log Analytics 工作区](#how-can-i-use-my-existing-log-analytics-workspace)中所述指定默认工作区设置。
   2. 或者，可以让迁移完成，在 VM 上安装 Microsoft Monitoring Agent，并将 VM 连接到所创建的工作区。 然后，在选择启用重新配置已安装代理的情况下，通过指定默认工作区设置来选择自己的自定义工作区。 有关详细信息，请参阅[如何使用现有的 Log Analytics 工作区？](#how-can-i-use-my-existing-log-analytics-workspace)

### <a name="what-are-the-implications-of-opting-out-of-automatic-provisioning"></a>选择禁用自动预配有何影响？
完成迁移后，安全中心无法从 VM 收集安全数据，因此无法提供某些安全建议和提示。 如果选择禁用，应手动安装 Microsoft Monitoring Agent。 请参阅[选择禁用时的建议步骤](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)。

### <a name="what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning"></a>选择禁用自动预配时的建议步骤是什么？

应手动安装 Microsoft Monitoring Agent 扩展，使安全中心能够从 VM 收集安全数据并提供建议和警报。 有关安装的指导，请参阅 [Windows VM 的代理安装](../virtual-machines/extensions/oms-windows.md)或 [Linux VM 的代理安装](../virtual-machines/extensions/oms-linux.md)。

可将代理连接到任何现有的自定义工作区或安全中心创建的工作区。 如果自定义工作区未启用“Security”或“SecurityCenterFree”解决方案，则需要应用某个解决方案。 若要应用，请选择自定义工作区或订阅，并通过“安全策略 – 定价层”边栏选项卡应用定价层。

   ![定价层][1]

安全中心将会根据选定的定价层在工作区中启用正确的解决方案。

### 如何删除由安全中心安装的 OMS 扩展？<a name="remove-oms"></a>
可以手动删除 Microsoft Monitoring Agent。 不建议这样操作，因为这会限制安全中心的建议和提示。

> [!NOTE]
> 启用数据收集后，安全中心会删除代理，然后重新安装。  需要禁用数据收集，然后再手动删除代理。 请参阅“如何停止自动代理安装和工作区创建？”，查看有关禁用数据收集的说明。
>
>

手动删除代理：

1.  在门户中，打开“Log Analytics”。
2.  在“Log Analytics”边栏选项卡中选择一个工作区：
3.  选择每个不需要监视的 VM 并选择“断开连接”。

   ![删除代理][3]

> [!NOTE]
> 如果 Linux VM 已有非扩展 OMS 代理，则删除该扩展也会删除代理，因此客户需要重新安装。
>
>
### <a name="how-do-i-disable-data-collection"></a>如何禁用数据收集？
默认情况下自动设置处于关闭状态。 可以随时禁用安全策略中的此设置，禁用资源的自动设置。 强烈建议使用自动设置，以获取有关系统更新、OS 漏洞和终结点保护的安全警报和建议。

若要禁用数据收集，请[登录到 Azure 门户](https://portal.azure.com)，依次选择“浏览”、“安全中心”和“策略”。 选择希望禁用自动设置的订阅。 选择订阅时，“安全策略 - 数据收集”将打开。 在“自动设置”下，选择“关闭”。

### <a name="how-do-i-enable-data-collection"></a>如何启用数据收集？
可在安全中心策略中对 Azure 订阅启用数据收集。 启用数据收集。 [登录到 Azure 门户](https://portal.azure.com)，依次选择“浏览”、“安全中心”和“策略”。 选择希望启用自动设置的订阅。 选择订阅时，“安全策略 - 数据收集”将打开。 在“自动设置”下，选择“打开”。

### <a name="what-happens-when-data-collection-is-enabled"></a>启用数据收集之后会发生什么？
启动自动设置后，安全中心可在所有受支持的 Azure VM 以及任何新建的 Azure VM 中预配 Microsoft Monitoring Agent。 强烈建议进行自动预配，但也可以手动代理安装。 [了解如何安装 Microsoft Monitoring Agent 扩展](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)。 

该代理可启用进程创建事件 4688 和事件 4688 内的 *CommandLine* 字段。 VM 上创建的新进程由事件日志记录，并由安全中心检测服务监视。 若要了解针对每个新进程记录的详细信息，请参阅 [4688 中的说明字段](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688#fields)。 该代理还收集 VM 上创建的 4688 事件，并将这些事件存储在搜索中。

该代理还为[自适应应用程序控件](security-center-adaptive-application.md)启用数据收集，安全中心会在审核模式下配置本地 AppLocker 策略以允许所有应用程序。 这将导致 AppLocker 生成事件，然后由安全中心收集和利用这些事件。 请务必注意，不会在已配置 AppLocker 策略的任何计算机上配置此策略。 

当安全中心检测到 VM 上的可疑活动时，如果已提供[安全联系信息](security-center-provide-security-contact-details.md)，将通过电子邮件通知客户。 警报也会显示在安全中心的安全警报仪表板中。



### <a name="does-the-monitoring-agent-impact-the-performance-of-my-servers"></a>Monitoring Agent 会影响服务器性能吗？
代理使用系统资源的名义帐户，对性能的影响极小。 有关性能影响和代理及扩展的详细信息，请参阅[规划和操作指南](security-center-planning-and-operations-guide.md#data-collection-and-storage)。

### <a name="where-is-my-data-stored"></a>我的数据存储在哪？
通过此代理收集的数据存储在与订阅关联的现有 Log Analytics 工作区或新工作区中。 有关详细信息，请参阅[数据安全](security-center-data-security.md)。

## 现有的 Azure Monitor 日志客户<a name="existingloganalyticscust"></a>

### <a name="does-security-center-override-any-existing-connections-between-vms-and-workspaces"></a>安全中心是否会覆盖 VM 和工作区之间的任何现有连接？
如果 VM 已将 Microsoft Monitoring Agent 作为 Azure 扩展进行安装，则安全中心不会覆盖现有工作区连接。 相反，安全中心会使用现有工作区。 将受保护 VM，前提是它报告的工作区上已安装"安全性"或"SecurityCenterFree"解决方案。 

安全中心解决方案在数据收集屏幕中选择工作区安装如果没有，并且该解决方案仅应用于相关的 Vm。 添加解决方案时，默认情况下会自动将它部署到连接到 Log Analytics 工作区的所有 Windows 和 Linux 代理。 [解决方案目标](../operations-management-suite/operations-management-suite-solution-targeting.md)可用于限定解决方案的范围。

如果 Microsoft Monitoring Agent 是直接安装到 VM 上（而不是作为 Azure 扩展进行安装），那么安全中心不会安装 Microsoft Monitoring Agent，并且安全监视也会受限。

### <a name="does-security-center-install-solutions-on-my-existing-log-analytics-workspaces-what-are-the-billing-implications"></a>安全中心是否在现有 Log Analytics 工作区上安装解决方案？ 计费会产生什么影响？
安全中心确定 VM 已连接到所创建的工作区时，会根据定价层启用此工作区上的解决方案。 由于[解决方案目标](../operations-management-suite/operations-management-suite-solution-targeting.md)，解决方案仅应用于相关的 Azure VM，因此计费保持不变。

- 免费层 – 安全中心在工作区中安装“SecurityCenterFree”解决方案。 免费层不会产生费用。
- **标准层** – 安全中心在工作区中安装“Security”解决方案。

   ![默认工作区上的解决方案][4]

### <a name="i-already-have-workspaces-in-my-environment-can-i-use-them-to-collect-security-data"></a>环境中已存在工作区，是否可以将其用于收集安全数据？
如果 VM 已将 Microsoft Monitoring Agent 作为 Azure 扩展进行安装，则安全中心会使用现有连接的工作区。 如果没有安全中心解决方案，则会在工作区安装，并且由于[解决方案目标](../operations-management-suite/operations-management-suite-solution-targeting.md)此解决方案仅适用于相关的 VM。

安全中心在 VM 上安装 Microsoft Monitoring Agent 时，会使用安全中心创建的默认工作区。

### <a name="i-already-have-security-solution-on-my-workspaces-what-are-the-billing-implications"></a>工作区已存在安全解决方案。 计费会产生什么影响？
“安全性与审核”解决方案用于启用 Azure VM 的安全中心标准层功能。 如果已在工作区上安装“安全性与审核”解决方案，则安全中心会使用现有解决方案。 计费方面没有任何更改。

## <a name="using-azure-security-center"></a>使用 Azure 安全中心
### <a name="what-is-a-security-policy"></a>什么是安全策略？
安全策略定义一组控件，这些控件是针对指定订阅中的资源建议的。 在 Azure 安全中心，用户需根据公司安全要求和应用程序类型或每个订阅中数据的敏感性，为 Azure 订阅定义策略。

Azure 安全中心中启用的安全策略将使用安全建议和监视。 若要了解有关安全策略的详细信息，请参阅[在 Azure 安全中心进行安全运行状况监视](security-center-monitoring.md)。

### <a name="who-can-modify-a-security-policy"></a>哪些用户可以修改安全策略？
若要修改安全策略，用户必须是安全管理员或是该订阅的所有者或参与者。

若要了解如何配置安全策略，请参阅[在 Azure 安全中心设置安全策略](tutorial-security-policy.md)。

### <a name="what-is-a-security-recommendation"></a>什么是安全建议？
Azure 安全中心可分析 Azure 资源的安全状态。 发现潜在的安全漏洞后会生成建议。 建议会对所需控件的整个配置过程提供指导。 示例如下：

* 预配反恶意软件可帮助识别和删除恶意软件
* 配置[网络安全组](../virtual-network/security-overview.md)和规则来控制发送到虚拟机的流量
* 设置 web 应用程序防火墙，帮助抵御针对 web 应用程序的攻击
* 部署缺少的系统更新
* 解决与推荐基线不匹配的操作系统配置

安全策略中仅已启用的推荐操作会显示在此处。

### <a name="how-can-i-see-the-current-security-state-of-my-azure-resources"></a>如何查看 Azure 资源当前安的全状态？
“安全中心概述”边栏选项卡会按计算、网络、存储和数据以及应用程序显示环境的总体安全状况。 每个资源类型都有一个指示符，该指示符显示是否已发现潜在安全漏洞。 单击每个磁贴可显示安全中心发现的安全问题列表和订阅中的资源清单。

### <a name="what-triggers-a-security-alert"></a>什么会触发安全警报？
Azure 安全中心自动从 Azure 资源、网络和合作伙伴解决方案（例如恶意软件和防火墙）收集、分析和融合数据。 检测到威胁时会创建安全警报。 示例中包括的检测项：

* 与已知的恶意 IP 地址通信的不符合安全性的虚拟机
* 使用 Windows 错误报告检测到的高级恶意软件
* 对虚拟机的暴力破解攻击
* 来自集成合作伙伴解决方案（例如反恶意软件或 Web 应用程序防火墙）的安全警报

### 为什么未保护评分值更改？ <a name="secure-score-faq"></a>
截至 2019 年 2 月，安全中心进行调整的一些建议，以更好地适应其严重程度的分数。 作为此调整后，结果可能有整体中的更改保护评分值。  有关安全分数的详细信息，请参阅[安全分数计算](security-center-secure-score.md)。

### <a name="whats-the-difference-between-threats-detected-and-alerted-on-by-microsoft-security-response-center-versus-azure-security-center"></a>Microsoft 安全响应中心与 Azure 安全中心检测和警示的威胁之间有何区别？
Microsoft 安全响应中心 (MSRC) 会执行 Azure 网络和基础结构的选择安全监视，并接收来自第三方的威胁情报和恶意投诉。 MSRC 发现不合法或未经授权的某一方访问客户数据或客户使用 Azure 不符合可接受的使用条款时，安全事件管理器会通知客户。 通常会以电子邮件方式向 Azure 安全中心中指定的安全联系人或 Azure 订阅所有者（如果未指定安全联系人）发送通知。

安全中心作为 Azure 的一项服务，可持续监视客户的 Azure 环境，并应用分析来自动广泛地检测潜在的恶意活动。 这些检测结果会作为安全警报显示在安全中心仪表板中。

### <a name="which-azure-resources-are-monitored-by-azure-security-center"></a>哪些 Azure 资源由 Azure 安全中心监视？
Azure 安全中心监视以下 Azure 资源：

* 虚拟机 (VM)（包括 [云服务](../cloud-services/cloud-services-choose-me.md)）
* 虚拟机规模集 (VMSSs)
* Azure 虚拟网络
* Azure SQL 服务
* Azure 存储帐户
* [应用服务环境](../app-service/environment/intro.md) 中的 Azure Web 应用
* 与 Azure 订阅集成的合作伙伴解决方案，例如 VM 和应用服务环境上的 Web 应用程序防火墙

此外，（包括在本地） 非 Azure 计算机也都可以通过 Azure 安全中心监视 (同时[Windows 计算机](./quick-onboard-windows-computer.md)并[Linux 计算机](./quick-onboard-linux-computer.md)支持)

## <a name="virtual-machines"></a>虚拟机
### <a name="what-types-of-virtual-machines-are-supported"></a>支持哪些类型的虚拟机？
使用[经典部署模式和 Resource Manager 部署模式](../azure-classic-rm.md)创建的虚拟机 (VM) 支持监视和建议。

有关支持平台的列表，请参阅[ Azure 安全中心支持的平台](security-center-os-coverage.md)。

### <a name="why-doesnt-azure-security-center-recognize-the-antimalware-solution-running-on-my-azure-vm"></a>Azure 安全中心为何不能识别我的 Azure VM 上运行的反恶意软件解决方案？
Azure 安全中心可识别出通过 Azure 扩展安装的反恶意软件。 例如，安全中心无法检测出预装在所提供的映像上的反恶意软件或使用自己的进程（例如配置管理系统）在虚拟机上安装的反恶意软件。

### <a name="why-do-i-get-the-message-missing-scan-data-for-my-vm"></a>为什么会显示消息称 VM“缺少扫描数据”？
没有可供 VM 扫描的数据时会出现此消息。 在 Azure 安全中心启用数据收集之后，扫描数据导入可能需要一段时间（低于一小时）。 首次导入扫描数据后可能会收到此消息，因为此时根本没有扫描数据或没有最近扫描数据。 扫描不会导入处于停止状态的 VM。 如果最近（根据 Windows 代理的保留策略，此值默认为 30 天）没有导入扫描数据，也可能会显示此消息。

### <a name="how-often-does-security-center-scan-for-operating-system-vulnerabilities-system-updates-and-endpoint-protection-issues"></a>安全中心多长时间扫描一次操作系统漏洞、系统更新和终结点保护问题？
安全中心扫描漏洞、更新和问题的延迟为：

- 操作系统安全配置 - 数据在 48 小时内更新
- 系统更新 – 数据在 24 小时内更新
- 终结点保护问题 – 数据在 8 小时内更新

安全中心通常每小时扫描一次新数据，并相应地刷新建议。 

> [!NOTE]
> 自 2017 年 6 月初开始，安全中心将使用 Microsoft Monitoring Agent 来收集和存储数据。 若要了解详细信息，请参阅 [Azure 安全中心平台迁移](security-center-platform-migration.md)。 本文中的信息表示转换到 Microsoft Monitoring Agent 后的安全中心功能。
>
>

### <a name="why-do-i-get-the-message-vm-agent-is-missing"></a>为什么会显示消息称“VM 代理缺失”？
VM 代理必须安装在 VM 上才能启用数据收集。 对于从 Azure 市场部署的 VM，默认安装 VM 代理。 有关如何在其他 VM 上安装 VM 代理的详细信息，请参阅博客文章 [VM 代理和扩展](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-2/)。


<!--Image references-->
[1]: ./media/security-center-platform-migration-faq/pricing-tier.png
[2]: ./media/security-center-platform-migration-faq/data-collection.png
[3]: ./media/security-center-platform-migration-faq/remove-the-agent.png
[4]: ./media/security-center-platform-migration-faq/solutions.png
[5]: ./media/security-center-platform-migration-faq/use-another-workspace.png
