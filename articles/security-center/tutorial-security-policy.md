---
title: 使用安全策略 | Microsoft Docs
description: 本文介绍如何使用 Azure 安全中心的安全策略。
services: security-center
documentationcenter: na
author: monhaber
manager: barbkess
editor: ''
ms.assetid: 2d248817-ae97-4c10-8f5d-5c207a8019ea
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 5/05/2019
ms.author: monhaber
ms.openlocfilehash: 0465f12e3da6acc2ec8f7259ee9f040ccd3ce88e
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236307"
---
# <a name="working-with-security-policies"></a>使用安全策略

本文介绍如何配置安全策略，以及如何在安全中心查看这些策略。 Azure 安全中心会自动在每个载入的订阅上分配其[内置安全策略](security-center-policy-definitions.md)。 可以在 [Azure Policy](../governance/policy/overview.md) 中配置这些策略，由此也可跨管理组和跨多个订阅设置策略。

有关如何使用 PowerShell 设置策略的说明，请参阅[快速入门：使用 Azure PowerShell 模块创建策略分配以识别不合规资源](../governance/policy/assign-policy-powershell.md)。

>[!NOTE]
> 安全中心使用 Azure Policy 开始集成。 现有客户将自动迁移到 Azure Policy 中新的内置计划，而不是安全中心中的旧的安全策略。 除了 Azure Policy 中存在新计划外，此更改不会影响资源或环境。

## <a name="what-are-security-policies"></a>什么是安全策略？
安全策略定义了工作负载的相应配置，有助于确保用户遵守公司或法规方面的安全要求。 在 Azure Policy 中，可定义 Azure 订阅策略，并根据工作负载类型或数据机密性进行量身定制。 例如，使用受管制数据（如个人身份信息）的应用程序可能需要比其他工作负载更高级别的安全性。 若要跨订阅或管理组设置策略，请在 [Azure Policy](../governance/policy/overview.md) 中进行设置。

安全策略驱动在 Azure 安全中心获得的安全建议。 可以使用它们监视符合性以帮助识别潜在漏洞和缓解威胁。 若要详细了解如何确定适合你的选项，请参阅[内置安全性策略](security-center-policy-definitions.md)列表。

当启用安全中心时，内置到安全中心的安全策略将作为内置的计划反映在 Azure Policy 中，位于“安全中心”类别下。 内置的计划自动分配给安全中心注册的所有订阅（免费或标准层）。 内置的计划仅包含审核策略。


### <a name="management-groups"></a>管理组
如果你的组织有多个订阅，则可能需要一种方法来高效地管理这些订阅的访问权限、策略和符合性。 Azure 管理组提供订阅上的作用域级别。 可将订阅组织到名为“管理组”的容器中，并将管理策略应用到管理组。 管理组中的所有订阅都将自动继承应用于管理组的策略。 为每个目录指定了一个称为“根”管理组的顶级管理组。 此根管理组内置在层次结构中，包含其所有下级管理组和订阅。 此根管理组允许在目录级别应用全局策略和 RBAC 分配。 若要设置用于 Azure 安全中心的管理组，请按照[为 Azure 安全中心实现租户级可见性](security-center-management-groups.md)中的说明进行操作。

> [!NOTE]
> 务必要了解管理组和订阅的层次结构。 请参阅[使用 Azure 管理组来组织资源](../governance/management-groups/index.md#root-management-group-for-each-directory)来了解有关管理组、根管理和管理组访问权限的详细信息。
>

## <a name="how-security-policies-work"></a>安全策略工作原理
安全中心自动为每个 Azure 订阅创建默认的安全策略。 可以在 Azure Policy 内编辑策略，执行以下操作：
- 创建新的策略定义。
- 跨管理组和订阅分配策略，这些管理组和订阅可以代表整个组织，或者组织中的某个业务部门。
- 监视策略符合性。

有关 Azure Policy 的详细信息，请参阅[创建和管理策略以强制实施符合性](../governance/policy/tutorials/create-and-manage.md)。

Azure Policy 由以下组件构成：

- “策略”是一项规则。
- “计划”是一个策略集合。
- “分配”是将计划或策略应用于特定的范围（管理组、订阅或资源组）。

## <a name="view-security-policies"></a>查看安全策略

要在安全中心内查看安全策略，请执行以下操作：

1. 在“安全中心”仪表板中，选择“安全策略”。

    ![“策略管理”窗格](./media/security-center-policies/security-center-policy-mgt.png)

   在“策略管理”屏幕中，可以看到管理组数、订阅数、工作区数以及管理组结构。

   > [!NOTE]
   > - “安全中心”仪表板在“订阅覆盖范围”下显示的订阅数可能会高于在“策略管理”下显示的订阅数。 订阅覆盖范围显示标准订阅、免费订阅和“未覆盖”订阅的数量。 “未覆盖”订阅未启用“安全中心”，并且不会显示在“策略管理”下。
   >

   表中的列显示了：

   - **策略计划分配** – 分配给订阅或管理组的“安全中心”[内置策略](security-center-policy-definitions.md)和计划。
   - **覆盖范围** – 标识管理组、订阅或工作区在其上运行的定价层：免费或标准。  若要详细了解安全中心的定价层，请参阅[定价](security-center-pricing.md)。
   - **设置** – 订阅有“编辑设置”链接。 选择“编辑设置”，可为每个订阅或管理组更新[安全中心设置](security-center-policies-overview.md)。
   - **安全分数** - [安全分数](security-center-secure-score.md)可衡量工作负载安全状况的安全程度，并帮助确定改进建议的优先级。

2. 选择想要查看其策略的订阅或管理组。

   - “安全策略”屏幕反映在所选订阅或管理组上分配的策略所执行的操作。
   - 在顶部，使用提供的链接打开适用于订阅或管理组的每个策略“分配”。 可以使用链接访问分配以及编辑或禁用策略。 例如，如果发现特定策略分配有效地拒绝终结点保护，则可以使用该链接访问策略以及辑或禁用它。
   - 在策略列表中，可以看到策略有效应用于订阅或管理组。 这意味着将考虑适用于该范围的每个策略的设置，并提供策略所执行操作的累计效果。 例如，如果一个分配禁用此策略，而另一个设置为 AuditIfNotExist，则累计效果适用于 AuditIfNotExist。 更积极的效果始终优先。
   - 策略的效果可以是：追加、审核、AuditIfNotExists、拒绝、DeployIfNotExists 和禁用。 有关如何应用效果的详细信息，请参阅[了解策略效果](../governance/policy/concepts/effects.md)。

   ![策略屏幕](./media/security-center-policies/policy-screen.png)

> [!NOTE]
> - 查看已分配的策略时，可以看到多个分配并且可以看到每个分配如何自行配置。

## <a name="edit-security-policies"></a>编辑安全策略
可以在 [Azure Policy](../governance/policy/tutorials/create-and-manage.md) 中为每个 Azure 订阅和管理组编辑默认的安全策略。 若要修改安全策略，你必须是该订阅或包含型管理组的所有者、参与者或安全管理员。

有关如何在 Azure Policy 中编辑安全策略的说明，请参阅[创建和管理策略以强制实施符合性](../governance/policy/tutorials/create-and-manage.md)。

你可以在 Azure Policy 门户中通过 REST API 或 Windows PowerShell 编辑安全策略。 以下示例为使用 REST API 进行编辑的相关说明。


## <a name="disable-security-policies"></a>禁用安全策略
如果默认安全策略生成的不是为您的环境相关的建议，则可以通过禁用的策略定义的发送建议将其停止。
有关建议的进一步信息，请参阅[管理的安全建议](security-center-recommendations.md)。

1. 在安全中心中，从**策略和符合性**部分中，单击**安全策略**。

   ![策略管理](./media/tutorial-security-policy/policy-management.png)

2. 单击你想要禁用该建议的订阅或管理组。

   > [!Note]
   > 请记住，管理组将其策略适用于其订阅。 因此，如果禁用订阅的策略，并且在订阅属于仍使用相同的策略的管理组，您将继续接收策略建议。 仍将从管理级别应用该策略，且仍将生成建议。

1. 单击分配的策略。

   ![禁用策略](./media/tutorial-security-policy/security-policy.png)

1. 在中**参数**部分中，搜索调用你想要禁用，该建议的策略，从下拉列表中，选择**已禁用**

   ![禁用策略](./media/tutorial-security-policy/disable-policy.png)
1. 单击“ **保存**”。
   > [!Note]
   > 禁用策略更改可能需要 12 小时才会生效。


### <a name="configure-a-security-policy-using-the-rest-api"></a>使用 REST API 配置安全策略

Azure 安全中心与 Azure Policy 实现了本机集成，借助它，可以利用 Azure Policy 的 REST API 来创建策略分配。 以下说明演示如何创建策略分配以及如何自定义现有的分配。 

Azure Policy 中的重要概念： 

-  **策略定义** 是规则 

-  **计划** 是策略定义（规则）的集合 

-  **分配** 是将计划或策略应用于特定的范围（管理组、订阅等） 

安全中心有一项内置计划，它包括中心内的所有安全策略。 为评估对 Azure 资源的安全中心策略，应对管理组或想访问的订阅创建一个分配。  

内置计划默认启用所有安全中心策略。 你可以选择禁用内置计划中的某些策略，例如：可用将“Web 应用程序防火墙”策略的效果参数更改为“禁用”，应用除它以外的所有安全中心策略。 

### <a name="api-examples"></a>API 示例

在下面的示例中，替换以下三个变量：

- **{scope}**，用于输入要应用策略的管理组或订阅的名称。
- **{policyAssignmentName}**，用于输入[相关策略分配的名称](#policy-names)。
- **{name}**，用于输入你的名字或批准策略更改的管理员的名字。

本示例演示如何对订阅或管理组分配内置的安全中心计划
 
    PUT  
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 

    Request Body (JSON) 

    { 

      "properties":{ 

    "displayName":"Enable Monitoring in Azure Security Center", 

    "metadata":{ 

    "assignedBy":"{Name}" 

    }, 

    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 

    "parameters":{}, 

    } 

    } 

本示例演示如何对订阅分配内置的安全中心计划，且禁用以下策略： 

- 系统更新 ("systemUpdatesMonitoringEffect") 

- 安全配置 ("systemConfigurationsMonitoringEffect") 

- 终结点保护 ("endpointProtectionMonitoringEffect") 


    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
    
    请求正文 (JSON) 
    
    { 
    
      "properties":{ 
    
    "displayName":"启用监视 Azure 安全中心"， 
    
    "元数据": { 
    
    "assignedBy":"{Name}" 
    
    }, 
    
    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 
    
    "parameters":{ 
    
    "systemUpdatesMonitoringEffect": {"value":"已禁用"}， 
    
    "systemConfigurationsMonitoringEffect": {"value":"已禁用"}， 
    
    "endpointProtectionMonitoringEffect": {"value":"已禁用"}， 
    
    }, 
    
     } 
    
    } 

此示例演示如何删除分配：

    DELETE   
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 


### 策略名引用 <a name="policy-names"></a>

|安全中心内的策略名|Azure Policy 中显示的策略名 |策略效果参数名|
|----|----|----|
|SQL 加密 |监视 Azure 安全中心内未加密的 SQL 数据库 |sqlEncryptionMonitoringEffect| 
|SQL 审核 |监视 Azure 安全中心内未审核的 SQL 数据库 |sqlAuditingMonitoringEffect|
|系统更新 |监视 Azure 安全中心内系统更新的缺失情况 |systemUpdatesMonitoringEffect|
|存储加密 |审核存储帐户是否缺少 blob 加密 |storageEncryptionMonitoringEffect|
|JIT 网络访问 |监视 Azure 安全中心内可能的网络即时(JIT)访问 |jitNetworkAccessMonitoringEffect |
|自适应应用程序控制 |监视 Azure 安全中心内列入允许列表的可能的应用 |adaptiveApplicationControlsMonitoringEffect|
|网络安全组 |监视 Azure 安全中心内规则较宽松的网络访问 |networkSecurityGroupsMonitoringEffect| 
|安全配置 |监视 Azure 安全中心的 OS 漏洞 |systemConfigurationsMonitoringEffect| 
|终结点保护 |监视 Azure 安全中心 Endpoint Protection 的缺失情况 |endpointProtectionMonitoringEffect |
|磁盘加密 |监视 Azure 安全中心内未加密的 VM 磁盘 |diskEncryptionMonitoringEffect|
|漏洞评估 |监视 Azure 安全中心的 VM 漏洞 |vulnerabilityAssessmentMonitoringEffect|
|Web 应用程序防火墙 |监视 Azure 安全中心内未受保护的 Web 应用程序 |webApplicationFirewallMonitoringEffect |
|下一代防火墙 |监视 Azure 安全中心内未受保护的网络终结点| |


### <a name="who-can-edit-security-policies"></a>谁可以编辑安全策略？
安全中心使用基于角色的访问控制 (RBAC)，提供可以分配给 Azure 中用户、组和服务的内置角色。 用户打开安全中心时，只能看到其有权访问的资源的相关信息。 这意味着，向用户分配了资源所属的订阅或资源组的“所有者”、“参与者”或“读者”角色。 除这些角色外，还有两个特定的安全中心角色：

- 安全读取者：有权查看安全中心，包括建议、警报、策略和运行状况，但无法进行更改。
- 安全管理员：拥有与安全读取者相同的查看权限，不同之处在于该角色有权更新安全策略、驳回建议和关闭警报。



## <a name="next-steps"></a>后续步骤
本文介绍了如何在 Azure Policy 中编辑安全策略。 若要详细了解安全中心，请参阅以下文章：

* [Azure 安全中心规划和操作指南](security-center-planning-and-operations-guide.md)：了解如何规划和了解有关 Azure 安全中心的设计注意事项。
* [在 Azure 安全中心进行安全运行状况监视](security-center-monitoring.md)：了解如何监视 Azure 资源的运行状况。
* [在 Azure 安全中心内管理和响应安全警报](security-center-managing-and-responding-alerts.md)：了解如何管理和响应安全警报。
* [通过 Azure 安全中心监视合作伙伴解决方案](security-center-partner-solutions.md)：了解如何监视合作伙伴解决方案的运行状况。
* [在 Azure 安全中心内实现租户级公开范围](security-center-management-groups.md)：了解如何为 Azure 安全中心设置管理组。
* [Azure 安全中心常见问题解答](security-center-faq.md)：获取服务使用方面的常见问题解答。
* [Azure 安全博客](https://blogs.msdn.com/b/azuresecurity/)：查找关于 Azure 安全性及合规性的博客文章。

若要了解有关 Azure Policy 的详细信息，请参阅[什么是 Azure Policy？](../governance/policy/overview.md)
