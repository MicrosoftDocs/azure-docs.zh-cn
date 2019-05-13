---
title: 使用 Azure 实验室服务设置实验室帐户 | Microsoft Docs
description: 本教程介绍如何使用 Azure 实验室服务设置实验室帐户。
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/24/2019
ms.author: spelluru
ms.openlocfilehash: 0977c4537e409b59be7f9031c488b3317f9f2f0f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415796"
---
# <a name="tutorial-set-up-a-lab-account-with-azure-lab-services"></a>教程：使用 Azure 实验室服务设置实验室帐户
在 Azure 实验室服务中，实验室帐户充当管理组织实验室的中心帐户。 在你的实验室帐户中，授予他人创建实验室的权限，并设置适用于实验室帐户下所有实验室的策略。 在本教程中，了解如何以实验室管理员身份创建实验室帐户。 

在本教程中，将执行以下操作：

> [!div class="checklist"]
> * 创建实验室帐户
> * 将用户添加为“实验室创建者”角色
> * 指定可供实验室所有者使用的市场映像

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。

## <a name="create-a-lab-account"></a>创建实验室帐户
以下步骤演示了如何使用 Azure 门户通过 Azure 实验室服务创建实验室。 

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中，选择“所有服务”。 在“DEVOPS”部分中，选择“实验室服务”。 如果选择“实验室服务”旁边的星星 (`*`)，则会将其添加到左侧菜单上的“收藏夹”部分。 从下次开始，请在“收藏夹”下选择“实验室服务”。

    ![“所有服务”->“实验室服务”](../media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. 在“实验室服务”页上，在工具栏中选择“添加”。 

    ![在“实验室帐户”页上选择“添加”](../media/tutorial-setup-lab-account/add-lab-account-button.png)
4. 在“实验室帐户”页上，执行以下操作： 
    1. 在“实验室帐户名称”中输入名称。 
    2. 选择要在其中创建实验室帐户的“Azure 订阅”。
    3. 在“资源组”中选择“新建”，然后输入资源组的名称。
    4. 为“位置”选择要在其中创建实验室帐户的位置/区域。 
    5. 选择一个现有**共享映像库**或者创建一个。 你可以将模板 VM 保存在共享映像库中，以便供其他人重复使用。 有关共享映像库的详细信息，请参阅[使用 Azure 实验室服务中的共享映像库](how-to-use-shared-image-gallery.md)。 
    6. 对于“对等虚拟网络”，请选择实验室网络的对等虚拟网络 (VNet)。 在此帐号中创建的实验室连接到所选 VNet 并在所选 VNet 中具有对资源的访问权限。 
    7. 为实验室中的 VM 指定**地址范围**。 此地址范围应当采用无类别域际路由 (CIDR) 表示法（示例：10.20.0.0/23）。 将在此地址范围内创建实验室中的虚拟机。 有关详细信息，请参阅[为实验室中的 VM 指定地址范围](how-to-configure-lab-accounts.md#specify-an-address-range-for-vms-in-the-lab)。
    8. 对于“允许实验室创建者选择实验室位置”字段，请指定是否希望实验室创建者能够选择实验室位置。 默认情况下，此选项已禁用。 此选项禁用时，实验室创建者无法为他们正在创建的实验室指定位置。 实验室在距离实验室帐户最近的地理位置进行创建。 此选项启用时，实验室创建者可以在创建实验室时选择位置。 
    9. 选择“创建”。 

        ![创建实验室帐户窗口](../media/tutorial-setup-lab-account/lab-account-settings.png)
5. 选择工具栏上的**钟形图标**（**通知**），确认部署已成功，然后选择“转到资源”。 

    也可在“实验室帐户”页上选择“刷新”，然后选择已创建的实验室帐户。 

    ![创建实验室帐户窗口](../media/tutorial-setup-lab-account/go-to-lab-account.png)    
6. 会看到以下“实验室帐户”页面：

    ![“实验室帐户”页面](../media/tutorial-setup-lab-account/lab-account-page.png)

## <a name="add-a-user-to-the-lab-creator-role"></a>将用户添加为“实验室创建者”角色
若要在实验室帐户中设置课堂实验室，用户必须是实验室帐户中“实验室创建者”角色的成员。 用来创建实验室帐户的帐户会自动添加到此角色。 如果打算使用同一用户帐户创建课堂实验室，可以跳过此步骤。 若要使用其他用户帐户创建教室实验室，请执行以下步骤： 

为教师提供为课程创建实验室的权限，并将他们添加为**实验室创建者**角色：

1. 在“实验室帐户”页上，选择“访问控制(IAM)”，选择工具栏上的“添加”，然后选择工具栏上的“+ 添加角色分配”。 

    ![“访问控制”->“添加角色分配”按钮](../media/tutorial-setup-lab-account/add-role-assignment-button.png)
1. 在“添加角色分配”页上，选择“实验室创建者”作为“角色”，选择想要添加到“实验室创建者”角色的用户，然后选择“保存”。 

    ![添加实验室创建者](../media/tutorial-setup-lab-account/add-lab-creator.png)

## <a name="specify-marketplace-images-available-to-lab-creators"></a>指定可供实验室创建者使用的市场映像
作为实验室帐户所有者，你可以指定可供实验室创建者用来在实验室帐户中创建实验室的市场映像。 

1. 在左侧的菜单上选择“市场映像”。 默认情况下，可以看到映像的完整列表（包括启用的和禁用的）。 可以通过从顶部的下拉列表中选择“仅启用的/仅禁用的”选项对列表进行筛选来仅查看启用的/禁用的映像。 
    
    ![“市场映像”页](../media/tutorial-setup-lab-account/marketplace-images-page.png)

    列表中显示的市场映像只是满足以下条件的映像：
        
    - 创建单个 VM。
    - 使用 Azure 资源管理器预配 VM
    - 不需要购买额外的许可计划
2. 若要**禁用**已启用的市场映像，请执行下列操作之一： 
    1. 选择最后一列中的“...”（省略号）并选择“禁用映像”。 

        ![禁用一个映像](../media/tutorial-setup-lab-account/disable-one-image.png) 
    2. 通过选中列表中映像名称前面的复选框从列表中选择一个或多个映像，然后选择“禁用所选映像”。 

        ![禁用多个映像](../media/tutorial-setup-lab-account/disable-multiple-images.png) 
1. 类似地，若要**启用**市场映像，请执行下列操作之一： 
    1. 选择最后一列中的“...”（省略号）并选择“启用映像”。 
    2. 通过选中列表中映像名称前面的复选框从列表中选择一个或多个映像，然后选择“启用所选映像”。 

## <a name="next-steps"></a>后续步骤
在本教程中，已创建实验室帐户。 要了解如何专业地创建教室实验室，请继续下一教程：

> [!div class="nextstepaction"]
> [设置课堂实验室](tutorial-setup-classroom-lab.md)

