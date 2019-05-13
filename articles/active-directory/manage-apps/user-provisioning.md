---
title: Azure AD 中的自动 SaaS 应用用户预配 | Microsoft Docs
description: 介绍可以如何使用 Azure AD 进行自动化设置、取消设置，并不断跨多个第三方 SaaS 应用程序更新用户帐户。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/02/2019
ms.author: celested
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: 67956b3369394f68d067fc4753a859c066428aea
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191499"
---
# <a name="automate-user-provisioning-and-deprovisioning-to-saas-applications-with-azure-active-directory"></a>Azure Active Directory SaaS 应用程序的自动化用户预配和取消预配

## <a name="what-is-automated-user-provisioning-for-saas-apps"></a>什么是 SaaS 应用的自动化用户预配？
Azure Active Directory (Azure AD) 使你能够自动创建、 维护和删除的云中的用户标识 ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) 应用程序，如 Dropbox、 Salesforce、 ServiceNow 和的详细信息。

> [!VIDEO https://www.youtube.com/embed/_ZjARPpI6NI]

**此功能允许你：**

- 在新人加入团队或组织时，在适当的系统中为他们自动创建新帐户。
- 在新人离开团队或组织时，在适当的系统中自动停用其帐户。
- 确保应用和系统中的标识基于目录中的更改或人力资源系统保持最新。
- 将非用户对象（例如组）预配到支持它们的应用程序。

**自动的用户预配还包括此功能：**

- 在源与目标系统之间匹配现有标识的功能。
- 可自定义的属性映射，定义应将哪些用户数据从源系统流向目标系统。
- 可选电子邮件警报，用于设置错误。
- 报告和活动日志，用于帮助进行监视和故障排除。

## <a name="why-use-automated-provisioning"></a>为何要使用自动化预配？

使用此功能的一些常见动机包括：

- 避免与手动预配过程相关的成本、低效和人为错误。
- 避免产生与托管和维护定制开发的预配解决方案和脚本相关的成本。
- 保护你的组织他们离开组织时立即关键 SaaS 应用中删除用户的标识。
- 轻松地导入大量用户的特定 SaaS 应用程序或系统。
- 具有一组策略，以确定谁预配，谁可以登录到应用程序。

## <a name="how-does-automatic-provisioning-work"></a>自动化预配的工作原理
    
**Azure AD 预配服务**用户预配到 SaaS 应用和其他系统通过连接到每个应用程序供应商提供的用户管理 API 终结点。 这些用户管理 API 终结点允许 Azure AD 以编程方式创建、更新和删除用户。 对于所选应用程序，预配服务，还可以还创建、 更新和删除其他标识相关的对象，例如组和角色。 

![预配](./media/user-provisioning/provisioning0.PNG)
*图 1：Azure AD 预配服务*

![出站预配](./media/user-provisioning/provisioning1.PNG)
*图 2：从 Azure AD 到常见 SaaS 应用程序的“出站”用户预配工作流*

![出站预配](./media/user-provisioning/provisioning2.PNG)
*图 3：从常见人力资本管理 (HCM) 应用程序到 Azure Active Directory 和 Windows Server Active Directory 的“入站”用户预配工作流*


## <a name="what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning"></a>可在哪些应用程序和系统中使用 Azure AD 自动用户预配？

Azure AD 功能预先集成了对许多常用 SaaS 应用和人力资源系统，支持和实现的 SCIM 2.0 标准特定部分的应用的一般性支持。

### <a name="pre-integrated-applications"></a>预先集成的应用程序

有关 Azure AD 中支持预先集成预配连接器的所有应用程序的列表，请参阅[支持用户预配的应用程序列表教程](../saas-apps/tutorial-list.md)。

如需联系 Azure AD 工程团队以请求其他应用程序的预配支持，请通过 [Azure Active Directory 反馈论坛](https://feedback.azure.com/forums/374982-azure-active-directory-application-requests/filters/new?category_id=172035)提交消息。

> [!NOTE]
> 为了使应用程序支持自动化用户预配，它必须首先提供必要的、允许外部程序自动执行创建、维护和删除用户操作的用户管理 API。 因此，不是所有的 SaaS 应用都能兼容此功能。 对于不支持用户管理 Api 的应用，Azure AD 工程团队可生成连接到这些应用程序的预配连接器和这项工作按当前和潜在客户的需求设置优先级。 

### <a name="connecting-applications-that-support-scim-20"></a>连接支持 SCIM 2.0 的应用程序

若要了解如何常规连接实现基于 SCIM. 2.0 的用户管理 API 的应用程序，请参阅[使用 SCIM 自动将用户和组从 Azure Active Directory 预配到应用程序](use-scim-to-provision-users-and-groups.md)。

    
## <a name="how-do-i-set-up-automatic-provisioning-to-an-application"></a>如何设置为自动预配到应用程序？

> [!VIDEO https://www.youtube.com/embed/pKzyts6kfrw]

使用 Azure Active Directory 门户配置 Azure AD 预配为所选应用程序的服务。

1. 打开 **[Azure Active Directory 门户](https://aad.portal.azure.com)**。

1. 选择**企业应用程序**在左窗格中。 显示了已配置的所有应用的列表。

1. 选择 **+ 新应用程序**以添加应用程序。 添加以下具体取决于你的方案：

   - **添加你自己的应用**选项支持自定义开发的 SCIM 集成。

   - 中的所有应用程序**从库中的添加** > **特色应用程序**部分支持自动预配。 有关其他应用程序，请参阅[支持用户预配的应用程序列表教程](../saas-apps/tutorial-list.md)。

1. 提供的所有详细信息，并选择**添加**。 新的应用添加到企业应用程序的列表，并打开到其应用程序管理屏幕。

1. 选择**预配**来管理用户帐户预配应用程序设置。

   ![设置](./media/user-provisioning/provisioning_settings0.PNG)

1. 选择自动选项**预配模式**以指定的管理员凭据、 映射、 启动和停止，以及同步设置。

   - 展开**管理员凭据**输入 Azure AD 连接到应用程序的用户管理 API 所需的凭据。 本部分中，您还可以启用电子邮件通知，如果在凭据失败，或预配作业进入[隔离](#quarantine)。

   - 展开**映射**查看和编辑 Azure AD 之间流动的用户属性和目标应用程序预配或更新用户帐户的时间。 如果目标应用程序支持，可以使用本部分 （可选） 配置的组和用户帐户预配。 若要打开右侧，可以在其中查看并自定义用户属性映射编辑器在表中选择一个映射。
   
     **范围筛选器**告知预配服务的用户和组在源系统中的应预配或取消预配到目标系统。 在中**属性映射**窗格中，选择**源对象范围**对特定属性值进行筛选。 例如，可以指定只能将“Department”属性为“Sales”的用户列入预配范围。 有关详细信息，请参阅[使用范围筛选器](define-conditional-rules-for-provisioning-user-accounts.md)。
    
     有关详细信息，请参阅[自定义属性映射](customize-application-attributes.md)。

   - **设置**控制预配服务的应用程序，包括是否当前正在运行的操作。 **作用域**菜单，可以指定只有分配的用户和组应在范围内进行预配，还是应该预配 Azure AD 目录中的所有用户。 有关“分配的”用户和组的信息，请参阅[在 Azure Active Directory 中将用户或组分配到企业应用](assign-user-or-group-access-portal.md)。

在应用程序管理屏幕中，选择**审核日志**若要查看记录的所有操作的情况下运行的 Azure AD 预配服务。 有关详细信息，请参阅[预配报告指南](check-status-user-account-provisioning.md)。

![设置](./media/user-provisioning/audit_logs.PNG)

> [!NOTE]
> 还可以使用 [Microsoft 图形 API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview) 配置和管理 Azure AD 用户预配服务。


## <a name="what-happens-during-provisioning"></a>预配期间会发生什么情况？

如果 Azure AD 是源系统，预配服务将使用 [Azure AD 图形 API 的差异查询功能](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-differential-query)来监视用户和组。 预配服务针对源系统和目标系统运行初始同步，然后运行定期的增量同步。 

### <a name="initial-sync"></a>初始同步

当启动预配服务时，将首次同步运行：

1. 查询源系统中的所有用户和组，并检索[属性映射](customize-application-attributes.md)中定义的所有属性。
2. 使用配置的任何[分配](assign-user-or-group-access-portal.md)或[基于属性的范围筛选器](define-conditional-rules-for-provisioning-user-accounts.md)筛选返回的用户和组。
3. 当某个用户分配或预配范围内，该服务的查询匹配的用户使用指定的目标系统[匹配属性](customize-application-attributes.md#understanding-attribute-mapping-properties)。 示例：如果源系统中的 userPrincipal 名称是匹配的属性并映射到目标系统中的 userName，则预配服务会在目标系统中查询与源系统中 userPrincipal 名称值匹配的 userName。
4. 如果在目标系统中找不到匹配的用户，创建使用从源系统返回的属性。 创建用户帐户后，预配服务将检测并缓存新用户，用于运行对该用户所有将来的操作的目标系统的 ID。
5. 如果找到匹配的用户，则它是使用由源系统提供的属性进行更新。 匹配的用户帐户后，预配服务将检测并缓存新用户，用于运行对该用户所有将来的操作的目标系统的 ID。
6. 如果属性映射包含"reference"属性，该服务会创建并将链接引用的对象在目标系统上的其他更新。 例如，用户可能在目标系统中有“Manager”属性，该属性链接到目标系统中创建的另一个用户。
7. 保留为更高版本的增量同步提供的起始点在初始同步结束时水印。

某些应用程序例如 ServiceNow、 G Suite 和框支持不仅预配用户，而且还预配组及其成员。 在这些情况下，如果在启用了组预配[映射](customize-application-attributes.md)、 预配服务同步用户和组，和更高版本进行同步的组成员身份。 

### <a name="incremental-syncs"></a>增量同步

在初始同步后将所有其他同步：

1. 在源系统中查询自上次存储水印以来已更新的所有用户和组。
2. 使用配置的任何[分配](assign-user-or-group-access-portal.md)或[基于属性的范围筛选器](define-conditional-rules-for-provisioning-user-accounts.md)筛选返回的用户和组。
3. 当某个用户分配或预配范围内，该服务的查询匹配的用户使用指定的目标系统[匹配属性](customize-application-attributes.md#understanding-attribute-mapping-properties)。
4. 如果在目标系统中找不到匹配的用户，创建使用从源系统返回的属性。 创建用户帐户后，预配服务将检测并缓存新用户，用于运行对该用户所有将来的操作的目标系统的 ID。
5. 如果找到匹配的用户，则它是使用由源系统提供的属性进行更新。 如果它是新分配的帐户相匹配，预配服务会检测并缓存新用户，用于运行对该用户所有将来的操作的目标系统的 ID。
6. 如果属性映射包含"reference"属性，该服务会创建并将链接引用的对象在目标系统上的其他更新。 例如，用户可能在目标系统中有“Manager”属性，该属性链接到目标系统中创建的另一个用户。
7. 如果从范围中删除了以前在预配范围内的某个用户（包括未分配的用户），该服务会通过更新在目标系统中禁用该用户。
8. 如果在源系统中禁用或软删除了以前在预配范围内的某个用户，该服务会通过更新在目标系统中禁用该用户。
9. 如果在源系统中硬删除了以前在预配范围内的某个用户，该服务会在目标系统中删除该用户。 在 Azure AD 中，用户是硬删除 30 天后，它们已软删除。
10. 保留新水印增量同步，从而为更高版本的增量同步提供起点的末尾。

>[!NOTE]
> 可以选择性地禁用**创建**，**更新**，或**删除**通过使用操作**目标对象操作**中的复选框[映射](customize-application-attributes.md)部分。 还可以使用“accountEnabled”等字段通过属性映射来控制用于在更新期间禁用用户的逻辑。

预配服务将继续无限期，在中定义的间隔运行连续的增量同步[特定于每个应用程序的教程](../saas-apps/tutorial-list.md)，直到发生以下事件之一：

- 使用 Azure 门户或使用相应的图形 API 命令手动停止了该服务 
- 在 Azure 门户中使用“清除状态并重启”选项，或使用相应的图形 API 命令触发了新的初始同步。 此操作将清除所有存储的水印，并导致重新评估所有源对象。
- 由于属性映射或范围筛选器中的更改而触发了新的初始同步。 此操作也清除所有存储的水印，并导致重新评估所有源对象。
- 预配过程错误率较高，因为也隔离状态 （见下文），并保留在隔离区四周以上。 在此情况下，该服务会自动禁用。

### <a name="errors-and-retries"></a>错误和重试

如果不能添加、 更新或删除目标系统中由于出现错误，在目标系统中的单个用户，然后重试操作在下一个同步周期中。 如果用户操作继续失败，则重试频率将开始递减，并逐渐减少到每天只尝试一次。 若要解决错误，管理员必须检查[审核日志](check-status-user-account-provisioning.md)"进程托管"的事件以确定根本原因并采取相应的措施。 常见的失败包括：

- 未在源系统中填充目标系统中所需的某个用户属性
- 用户使用其没有唯一约束在目标系统中，在源系统中的属性值和相同的值存在另一个用户记录中

若要解决这些失败，可在源系统中调整受影响用户的属性值，或调整属性映射，以避免冲突。   

### <a name="quarantine"></a>隔离

如果大部分或全部一致地针对目标系统所做的调用失败由于出现错误 （例如管理员凭据无效），然后预配作业进入"隔离"状态。 此状态指示在[预配摘要报告](check-status-user-account-provisioning.md)和通过电子邮件已在 Azure 门户中配置了电子邮件通知。 

进入隔离区后，增量同步的频率会逐渐减少为每天一次。 

所有问题的错误修复和下一个同步周期开始后，将从隔离区删除预配作业。 如果预配作业在隔离区中保留四周以上，则会禁用预配作业。


## <a name="how-long-will-it-take-to-provision-users"></a>预配用户需要多长时间？

性能取决于是否预配作业正在运行初始同步还是增量同步。

有关**初始同步**，作业时间取决于许多因素，包括预配，范围内的用户和组的数量以及用户和组在源系统中的总数。 本部分中的下文中汇总了影响初始同步性能的因素的完整列表。

对于**增量同步**，作业时间取决于在该同步周期中检测到的更改数量。 如果有少于 5000 个用户或组成员身份更改，则作业可以在单个增量同步周期内完成。 

下表总结了常见的预配方案的同步时间。 在这些方案中，源系统是 Azure AD，目标系统是 SaaS 应用程序。 同步时间派生自 ServiceNow、 工作区、 Salesforce 和 G Suite 的 SaaS 应用程序的同步作业的统计分析。


| 作用域配置 | 作用域中的用户、组和成员 | 初始同步时间 | 增量同步时间 |
| -------- | -------- | -------- | -------- |
| 仅同步已分配的用户和组 |  < 1,000 |  < 30 分钟 | < 30 分钟 |
| 仅同步已分配的用户和组 |  1,000 - 10,000 | 142 - 708 分钟 | < 30 分钟 |
| 仅同步已分配的用户和组 |   10,000 - 100,000 | 1,170 - 2,340 分钟 | < 30 分钟 |
| 同步 Azure AD 中的所有用户和组 |  < 1,000 | < 30 分钟  | < 30 分钟 |
| 同步 Azure AD 中的所有用户和组 |  1,000 - 10,000 | < 30 - 120 分钟 | < 30 分钟 |
| 同步 Azure AD 中的所有用户和组 |  10,000 - 100,000  | 713 - 1,425 分钟 | < 30 分钟 |
| 同步 Azure AD 中的所有用户|  < 1,000  | < 30 分钟 | < 30 分钟 |
| 同步 Azure AD 中的所有用户 | 1,000 - 10,000  | 43 - 86 分钟 | < 30 分钟 |


对于“仅同步已分配的用户和组”配置，可以使用以下公式来确定大概的最小和最大预计**初始同步**时间：

    Minimum minutes =  0.01 x [Number of assigned users, groups, and group members]
    Maximum minutes = 0.08 x [Number of assigned users, groups, and group members] 
    
影响**初始同步**的完成时间的因素汇总：

- 预配范围内用户和组的总数。

- 源系统 (Azure AD) 中存在的用户、组和组成员的总数。

- 是否中预配范围内的用户与目标应用程序中的现有用户匹配，或者需要创建第一次。 为其所有用户都创建的第一次的同步作业需要在大约*两次长时间*作为同步的作业与现有用户为其匹配所有用户。

- [审核日志](check-status-user-account-provisioning.md)中的错误数。 如果有许多错误，并且预配服务已进入“隔离”状态，则性能较低。    

- 目标系统实现的请求速率限制。 某些目标系统实施请求速率限制和限制，这可能会影响在大规模同步操作期间的性能。 在这些情况下，太快地接收太多请求的应用可能会拖慢其响应速率或关闭连接。 为提高性能，连接器需要进行调整，以不高于应用可以应对的处理速率的速率来向应用发送请求。 Microsoft 构建的预配连接器进行此调整。 

- 已分配的组的数量和大小。 同步已分配的组比同步用户花费的时间要长。 已分配的组的数量和大小都会影响性能。 如果应用程序[为组对象同步启用了映射](customize-application-attributes.md#editing-group-attribute-mappings)，则除了用户之外，还会同步组名称和成员身份等组属性。 这些额外的同步比仅同步用户对象需要花费更长的时间。


## <a name="how-can-i-tell-if-users-are-being-provisioned-properly"></a>如何判断用户预配是否正确？

预配服务在用户运行的所有操作都记录在 Azure AD 审核日志。 这包括所有读取和写入操作对源和目标系统和用户数据读取或每个操作过程中写入的。

有关如何读取审核日志在 Azure 门户中的信息，请参阅[预配报告指南](check-status-user-account-provisioning.md)。


## <a name="how-do-i-troubleshoot-issues-with-user-provisioning"></a>如何排查用户预配问题？

有关如何排查自动用户预配问题的基于方案的指导，请参阅[在应用程序中配置和预配用户时出现问题](application-provisioning-config-problem.md)。


## <a name="what-are-the-best-practices-for-rolling-out-automatic-user-provisioning"></a>推出自动用户预配的最佳做法是什么？

> [!VIDEO https://www.youtube.com/embed/MAy8s5WSe3A]

有关向应用程序执行出站用户预配的示例分步部署计划，请参阅[有关用户预配的标识部署指南](https://aka.ms/userprovisioningdeploymentplan)。

## <a name="more-frequently-asked-questions"></a>更多常见问题

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-b2b-users-in-azure-ad"></a>SaaS 应用自动用户预配是否适用于 Azure AD 中的 B2B 用户？

是，可以使用 Azure AD 用户在 Azure AD 到 SaaS 应用程序中预配服务预配 B2B （或来宾） 用户。

但是，若要登录到使用 Azure AD 的 SaaS 应用程序的 B2B 用户，SaaS 应用程序必须具有其基于 SAML 的单一登录功能特定的方式配置。 有关如何配置 SaaS 应用程序支持的详细信息从登录 B2B 用户，请参阅[B2B 协作配置 SaaS 应用]( https://docs.microsoft.com/azure/active-directory/b2b/configure-saas-apps)。

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-dynamic-groups-in-azure-ad"></a>SaaS 应用的自动用户预配是否适用于 Azure AD 中的动态组？

是的。 Azure AD 用户预配服务时配置为"同步仅分配的用户和组"，可以预配或取消其预配的 SaaS 应用中的用户基于他们是否是成员[动态组](../users-groups-roles/groups-create-rule.md)。 动态组也适用于“同步所有用户和组”选项。

但是，动态组的使用可能会影响从 Azure AD 到 SaaS 应用程序的端到端用户预配的整体性能。 当使用动态组，请记住以下注意事项和建议：

- SaaS 应用程序中的用户在动态组中预配或取消预配的速度取决于动态组评估成员身份更改的速度。 有关如何检查动态组的处理状态的信息，请参阅[检查成员身份规则的处理状态](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-create-rule)。

- 当使用动态组时，规则必须慎重考虑与用户预配和为成员身份会导致在取消预配事件丢失的一点是，取消预配。

### <a name="does-automatic-user-provisioning-to-saas-apps-work-with-nested-groups-in-azure-ad"></a>SaaS 应用的自动用户预配是否适用于 Azure AD 中的嵌套组？

不是。 当配置为"同步仅分配的用户和组"，不能读取或嵌套组中的用户预配 Azure AD 用户预配服务。 才能够读取和预配了显式分配的组的直接成员的用户。

这是“应用程序基于组的分配”的限制，该限制还会影响单一登录，请参阅[使用组管理对 SaaS 应用程序的访问](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-saasapps )进行了解。

解决此问题，您应明确指定 (或其他[作用域中](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts)) 包含需要预配的用户的组。

### <a name="is-provisioning-between-azure-ad-and-a-target-application-using-an-encrypted-channel"></a>Azure AD 之间设置和使用加密的通道的目标应用程序？

可以。 我们使用的服务器目标的 HTTPS SSL 加密。 

## <a name="related-articles"></a>相关文章

- [有关如何集成 SaaS 应用的教程列表](../saas-apps/tutorial-list.md)
- [为用户预配自定义属性映射](customize-application-attributes.md)
- [为属性映射编写表达式](functions-for-customizing-application-data.md)
- [用于用户预配的作用域筛选器](define-conditional-rules-for-provisioning-user-accounts.md)
- [使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配](use-scim-to-provision-users-and-groups.md)
- [Azure AD 同步 API 概述](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-overview)
