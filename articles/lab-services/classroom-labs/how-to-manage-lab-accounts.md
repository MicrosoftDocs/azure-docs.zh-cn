---
title: 管理 Azure 实验室服务中的实验室帐户 | Microsoft Docs
description: 了解如何在 Azure 订阅中创建实验室帐户、查看所有实验室帐户，或者删除实验室帐户。
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: spelluru
ms.openlocfilehash: 6f283ce007e96547e01a01a3753ddcb60574bfc3
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412805"
---
# <a name="manage-lab-accounts-in-azure-lab-services"></a>管理 Azure 实验室服务中的实验室帐户 
Azure 实验室服务中的实验室帐户是如教室实验室的实验室托管的类型的容器。 管理员可以设置一个具有 Azure 实验室服务的实验室帐户，并为能够在帐户中创建实验室的实验室所有者提供访问权限。 本文介绍如何创建实验室帐户、查看所有实验室帐户，或者删除实验室帐户。

## <a name="create-a-lab-account"></a>创建实验室帐户
以下步骤演示了如何使用 Azure 门户通过 Azure 实验室服务创建实验室。 

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中，选择“所有服务”。 在“DEVOPS”部分中，选择“实验室帐户”。 如果选择“实验室帐户”旁边的星星 (`*`)，则会将其添加到左侧菜单上的“收藏夹”部分。 从下次开始，请在“收藏夹”下选择“实验室帐户”。

    ![“所有服务”->“实验室帐户”](../media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. 在“实验室帐户”页上，在工具栏中选择“添加”。 

    ![在“实验室帐户”页上选择“添加”](../media/tutorial-setup-lab-account/add-lab-account-button.png)
4. 在“实验室帐户”页上，执行以下操作： 
    1. 在“实验室帐户名称”中输入名称。 
    2. 选择要在其中创建实验室帐户的“Azure 订阅”。
    3. 在“资源组”中选择“新建”，然后输入资源组的名称。
    4. 为“位置”选择要在其中创建实验室帐户的位置/区域。 
    5. 选择一个现有**共享的映像库**或创建一个。 您可以为它要重复使用由其他人共享的映像库中保存的模板 VM。 共享的映像库的详细信息，请参阅[使用 Azure 实验室服务中共享的映像库](how-to-use-shared-image-gallery.md)。
    6. 对于“对等虚拟网络”，请选择实验室网络的对等虚拟网络 (VNet)。 在此帐号中创建的实验室连接到所选 VNet 并在所选 VNet 中具有对资源的访问权限。 
    7. 指定**地址范围**适用于在实验室中的 Vm。 应为无类域间路由 (CIDR) 表示法的地址范围 (示例：10.20.0.0/23). 将此地址范围中创建实验室中的虚拟机。 有关详细信息，请参阅[在实验室中为 Vm 指定的地址范围](how-to-configure-lab-accounts.md#specify-an-address-range-for-vms-in-the-lab)。    
    8. 对于“允许实验室创建者选择实验室位置”字段，请指定是否希望实验室创建者能够选择实验室位置。 默认情况下，此选项已禁用。 此选项禁用时，实验室创建者无法为他们正在创建的实验室指定位置。 实验室在距离实验室帐户最近的地理位置进行创建。 此选项启用时，实验室创建者可以在创建实验室时选择位置。      
    9. 选择“创建”。 

        ![创建实验室帐户窗口](../media/tutorial-setup-lab-account/lab-account-settings.png)
5. 选择工具栏上的**钟形图标**（**通知**），确认部署已成功，然后选择“转到资源”。 

    也可在“实验室帐户”页上选择“刷新”，然后选择已创建的实验室帐户。 

    ![创建实验室帐户窗口](../media/tutorial-setup-lab-account/go-to-lab-account.png)    
6. 会看到以下“实验室帐户”页面：

    ![“实验室帐户”页面](../media/tutorial-setup-lab-account/lab-account-page.png)

## <a name="view-lab-accounts"></a>查看实验室帐户
1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 从菜单中选择“所有资源”。 
3. 对于“类型”，选择“实验室帐户”。 
    也可按订阅、资源组、位置和标记进行筛选。 

    ![“所有资源”->“实验室帐户”](../media/how-to-manage-lab-accounts/all-resources-lab-accounts.png)

## <a name="view-and-manage-labs-in-the-lab-account"></a>在实验室帐户中查看和管理实验室

1. 在“实验室帐户”页上，选择左侧菜单中的“实验室”。

    ![帐户中的实验室](../media/how-to-manage-lab-accounts/labs-in-account.png)
1. 你会看到帐户中的**实验室列表**，其中包含以下信息： 
    1. 实验室的名称。
    2. 创建实验室的日期。 
    3. 创建实验室的用户的电子邮件地址。 
    4. 允许加入实验室的最大用户数。 
    5. 实验室的状态。 

## <a name="delete-a-lab-in-the-lab-account"></a>删除实验室帐户中的实验室
按照上一部分中的说明进行操作，以查看实验室帐户中的实验列表。

1. 选择“...(省略号)”，然后选择“删除”。 

    ![删除实验室 - 按钮](../media/how-to-manage-lab-accounts/delete-lab-button.png)
2. 在警告消息上选择“是”。 

    ![确认实验室删除](../media/how-to-manage-lab-accounts/confirm-lab-delete.png)

## <a name="delete-a-lab-account"></a>删除实验室帐户
按照上一节中的说明来显示列表中的实验室帐户。 根据以下说明删除实验室帐户： 

1. 选择要删除的**实验室帐户**。 
2. 从工具栏选择“删除”。 

    ![“实验室帐户”->“删除”按钮](../media/how-to-manage-lab-accounts/delete-button.png)
1. 键入“是”进行确认。
1. 选择“删除”。 

    ![删除实验室帐户 - 确认](../media/how-to-manage-lab-accounts/delete-lab-account-confirmation.png)


## <a name="next-steps"></a>后续步骤
请参阅以下文章：[如何配置实验室帐户](how-to-configure-lab-accounts.md)。