---
title: include 文件
description: include 文件
services: active-directory
author: rolyon
ms.service: active-directory
ms.topic: include
ms.date: 04/29/2019
ms.author: rolyon
ms.custom: include file
ms.openlocfilehash: 364d4a11772e6bb72e2e258503f3cce49dc61453
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141842"
---
## <a name="create-one-or-more-access-reviews"></a>创建一个或多个访问评审

1. 单击**新建**若要创建新的访问评审。

1. 命名访问评审。 可选择为评审提供说明。 名称和说明向评审者显示。

    ![创建访问评审 - 评审名称和说明](./media/active-directory-privileged-identity-management-access-reviews/name-description.png)

1. 设置“开始日期”。 默认情况下，访问评审只进行一次，从创建的时候开始，在一个月内结束。 可以更改开始和结束日期，使访问评审在将来的时间开始，并持续所需的天数。

    ![创建访问评审 - 开始和结束日期](./media/active-directory-privileged-identity-management-access-reviews/start-end-dates.png)

1. 若要使访问评审再次发生，请更改**频率**设置从**一次**到**每周**，**每月**， **每季度**，**每年一次**，或**半年一次**。 使用**持续时间**滑块或文本框中，可以定义多长时间内每段评论各定期系列的将是打开的审阅者的输入。 例如，每月评审的最长持续时间可以设置为 27 天，以免评审时间重叠。

1. 使用“结束”设置指定如何结束定期访问评审系列。 系列的结束方式有三种：持续运行，无限期地开始评审；运行至指定日期；运行至已完成定义的评审数目。 另一个用户管理员或另一个全局管理员可以停止序列创建之后通过更改中的日期**设置**，以便它在该日期结束。

1. 在中**用户**部分中，选择您想要查看的成员身份的一个或多个角色。

    ![创建访问评审的用户](./media/active-directory-privileged-identity-management-access-reviews/users.png)

    > [!NOTE]
    > 选择多个角色将创建多个访问评审。 例如，选择五个角色将创建五个单独的访问评审。

    如果要创建 Azure AD 角色的访问评审，以下显示了检查成员身份列表的示例。

    ![创建访问评审的审阅角色成员身份](./media/active-directory-privileged-identity-management-access-reviews/review-membership.png)

    如果要创建 Azure 资源角色的访问评审，以下显示了检查成员身份列表的示例。

    ![创建访问评审的审阅角色成员身份](./media/active-directory-privileged-identity-management-access-reviews/review-membership-azure-resource-roles.png)

1. 在中**审阅者**部分中，选择一个或多个人来评审所有用户。 也可以选择让成员评审自己的访问权限。

    ![创建访问评审 - 评审者](./media/active-directory-privileged-identity-management-access-reviews/reviewers.png)

    - **所选用户**-如果不知道谁需要访问使用此选项。 使用此选项，可以将审阅分配给资源所有者或组管理员完成。
    - **成员 （自我）** -使用此选项可让用户评审其自己的角色分配。

### <a name="upon-completion-settings"></a>完成后的设置

1. 若要指定评审完成后发生的情况，请展开“完成后的设置”部分。

    ![完成后的设置](./media/active-directory-privileged-identity-management-access-reviews/upon-completion-settings.png)

1. 若要自动删除被拒绝用户的访问权限，请将“将结果自动应用到资源”设置为“启用”。 若要在评审完成后手动应用结果，请将开关设置为“禁用”。

1. 使用“如果审阅者未答复”列表指定对于审阅者在评审期限内未评审的用户要执行的操作。 此设置不影响审阅者已手动评审的用户。 如果最终的审阅者决策是“拒绝”，则会删除用户的访问权限。

    - **不更改** - 将用户访问权限保持不变
    - **删除访问权限** - 删除用户的访问权限
    - **批准访问权限** - 批准用户的访问权限
    - **采用建议** - 根据系统的建议拒绝或批准用户的持续访问权限

### <a name="advanced-settings"></a>高级设置

1. 若要指定其他设置，请展开“高级设置”部分。

    ![高级设置](./media/active-directory-privileged-identity-management-access-reviews/advanced-settings.png)

1. 将“显示建议”设置为“启用”，以基于用户的访问权限信息向评审者显示系统建议。

1. 将“需要提供审批原因”设置为“启用”，以要求审阅者提供批准原因。

1. 将“邮件通知”设置为“启用”，以便在访问评审开始时让 Azure AD 向评审者发送电子邮件通知，并在评审完成时向管理员发送电子邮件通知。

1. 将“提醒”设置为“启用”，让 Azure AD 向尚未完成其审阅的审阅者发送访问评审正在进行的提醒。
