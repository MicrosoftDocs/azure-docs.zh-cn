---
title: 客户密码箱适用于 Microsoft Azure
description: 客户密码箱适用于 Microsoft Azure，Microsoft 可能需要访问客户数据时，提供对云提供程序访问控制技术概述。
author: cabailey
ms.service: security
ms.topic: article
ms.author: cabailey
manager: barbkess
ms.date: 05/07/2019
ms.openlocfilehash: 468e392cd2c45d79cbb24f8d737a6e83fbcd2725
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079265"
---
# <a name="customer-lockbox-for-microsoft-azure"></a>客户密码箱适用于 Microsoft Azure

> [!NOTE]
> 若要使用此功能，你的组织必须具有[Azure 支持计划](https://azure.microsoft.com/support/plans/)使用的最低级别**开发人员**。

客户密码箱适用于 Microsoft Azure 提供了一个接口，使客户能够查看并批准或拒绝的客户数据的访问请求。 Microsoft 工程师需要在支持请求过程中访问客户数据的情况下使用它。

这篇文章介绍如何启动、 跟踪，并且为更高版本的评论和审核存储客户密码箱请求。

客户密码箱现已正式发布，并且当前支持对虚拟机的远程桌面访问。

## <a name="workflow"></a>工作流

以下步骤概述了客户密码箱请求的典型工作流。

1. 有人在组织其 Azure 工作负荷的问题。

2. 此人用于诊断问题，但不能修复此错误后，他们打开支持票证[Azure 门户](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac)。 票证分配给 Azure 客户支持工程师。

3. Azure 支持工程师检查服务请求，并确定下一步的步骤来解决此问题。

4. 如果支持工程师不能通过使用标准工具和遥测数据来解决此问题下, 一步是通过使用实时 (JIT) 访问服务请求提升的权限。 此请求可从原始的支持工程师。 或者，可以将它从不同的工程师因为问题上报给 Azure DevOps 团队。

5. 访问后提交请求由 Azure 工程师，在实时服务评估请求，如考虑到几个因素：
    - 资源的范围
    - 请求者是一个独立的标识或使用多重身份验证
    - 权限级别
    
    基于 JIT 规则，此请求还可能包括从内部 Microsoft 审批者审批。 例如，审批者可能是客户支持主管或运营经理。

6. 当请求需要直接访问客户数据时，发出一个客户密码箱请求。 例如，远程桌面访问客户的虚拟机。
    
    对请求进行现在**通知客户**状态，等待客户的获得批准才能授予的访问权限。

7. 在客户组织中，有的用户[所有者角色](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-rbac-roles)适用于 Azure 订阅接收一封电子邮件来自 Microsoft，以通知他们有关未完成的访问请求信息。 对于客户密码箱请求，此人是指定审批者。
    
    示例电子邮件：
    
    ![Azure 客户密码箱-电子邮件通知](./media/azure-customer-lockbox/customer-lockbox-email-notification.png)

8. 电子邮件通知提供了指向**客户密码箱**Azure 门户边栏选项卡。 使用此链接，指定审批者在登录到 Azure 门户中查看任何挂起的请求其组织了客户密码箱：
    
    ![Azure 客户密码箱-登陆页面](./media/azure-customer-lockbox/customer-lockbox-landing-page.png)
    
   请求将为四天保留客户队列中。 此时间后自动过期访问请求并被授予 Microsoft 工程师没有访问权限。

9. 若要获取挂起的请求的详细信息，请指定审批者可以选择的密码箱请求**挂起的请求**:
    
    ![Azure 客户密码箱-查看挂起的请求](./media/azure-customer-lockbox/customer-lockbox-pending-requests.png)

10. 此外可以选择指定审批者**服务请求 ID**若要查看已由原始用户创建支持票证请求。 此信息提供了有关为什么 Microsoft 支持部门产生了兴趣、 上下文和报告的问题的历史记录。 例如：
    
    ![Azure 客户密码箱-查看支持票证请求](./media/azure-customer-lockbox/customer-lockbox-support-ticket.png)

11. 查看请求之后, 指定审批者选择**批准**或**拒绝**:
    
    ![Azure 客户密码箱-选择批准或拒绝](./media/azure-customer-lockbox/customer-lockbox-approval.png)
    
    由于所选内容：
    - **批准**:向 Microsoft 工程师授予访问权限。 八个小时的默认时间段为授予访问权限。
    - **拒绝**:由 Microsoft 工程师的提升的访问权限请求被拒绝并采取任何进一步的操作。

出于审核目的，在此工作流所执行的操作记录在[客户密码箱请求日志](#auditing-logs)。

## <a name="auditing-logs"></a>审核日志

客户密码箱日志存储在活动日志中。 在 Azure 门户中，选择**活动日志**若要查看与客户密码箱请求相关的审核信息。 可以筛选特定操作，如：
- **拒绝密码箱请求**
- **创建密码箱请求**
- **密码箱请求审批**
- **密码箱请求到期**

示例：

![Azure 客户密码箱-活动日志](./media/azure-customer-lockbox/customer-lockbox-activitylogs.png)

## <a name="supported-services-and-scenarios"></a>受支持的服务和方案

以下服务和方案当前已公开客户密码箱的可用性。

### <a name="remote-desktop-access-to-virtual-machines"></a>对虚拟机的远程桌面访问

客户密码箱当前可用于虚拟机的远程桌面访问请求。 支持以下工作负荷：
- 平台即服务 (PaaS) 的版本 1
- 基础结构即服务 (IaaS) 的 Windows 和 Linux (仅 Azure Resource Manager)
- 虚拟机规模集-Windows 和 Linux

> [!NOTE]
> IaaS 从经典实例不受客户密码箱。 如果您有 IaaS 从经典实例上运行的工作负荷，我们建议您将其迁移从经典部署模型到 Resource Manager 部署模型。 有关说明，请参阅[平台支持的迁移的 IaaS 资源从经典部署模型到 Azure 资源管理器](../virtual-machines/windows/migration-classic-resource-manager-overview.md)。

#### <a name="detailed-audit-logs"></a>详细的审核日志

对于涉及远程桌面访问的方案，可以使用 Windows 事件日志以查看由 Microsoft 工程师所执行的操作。 请考虑使用 Azure 安全中心收集的事件日志并将数据复制到工作区进行分析。 有关详细信息，请参阅[Azure 安全中心中的数据收集](../security-center/security-center-enable-data-collection.md)。

## <a name="exclusions"></a>排除在外

客户密码箱请求不会触发以下的工程支持方案中：

- Microsoft 工程师需要执行的活动，超出了标准操作过程。 例如，若要恢复或还原意外或不可预测的方案中的服务。

- Microsoft 工程师访问 Azure 平台一部分的故障排除，并会无意中有权访问客户数据。 例如，Azure 网络团队执行故障排除导致数据包捕获的网络设备上。 但是，如果客户加密数据在传输过程中时，工程师不能读取数据。

## <a name="next-steps"></a>后续步骤

客户密码箱是自动提供给所有客户具有[Azure 支持计划](https://azure.microsoft.com/support/plans/)使用的最低级别**开发人员**。

当你具有符合条件的支持计划时，执行任何操作不需要由你才能客户密码箱。 如果不需要此操作继续进行从你的组织中有人提出支持票证，客户密码箱请求是由 Microsoft 工程师启动。
