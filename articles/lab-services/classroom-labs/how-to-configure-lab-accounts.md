---
title: 在 Azure 实验室服务中配置实验室帐户 |Microsoft Docs
description: 了解如何在创建后配置实验室帐户。
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
ms.openlocfilehash: ba469c038f04a31a57e798b97b5120bec573feae
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65414044"
---
# <a name="configure-lab-accounts-in-azure-lab-services"></a>在 Azure 实验室服务中配置实验室帐户 
Azure 实验室服务中的实验室帐户是如教室实验室的实验室托管的类型的容器。 管理员可以设置一个具有 Azure 实验室服务的实验室帐户，并为能够在帐户中创建实验室的实验室所有者提供访问权限。 本文介绍如何创建实验室帐户、查看所有实验室帐户，或者删除实验室帐户。

## <a name="connect-with-a-peer-virtual-network"></a>使用对等虚拟网络连接
若要连接到实验室的虚拟网络对等网络作为虚拟网络，请按照下列步骤：

1. 上**实验室帐户**页上，选择**实验室配置**在左侧菜单中。

    ![实验室配置页](../media/how-to-manage-lab-accounts/labs-configuration-page.png) 
1. 有关**对等虚拟网络**，选择**已启用**或**禁用**。 默认值是**禁用**。 若要启用对等虚拟网络，请执行以下步骤操作： 
    1. 选择“启用”。
    2. 选择**VNet**从下拉列表。 
3. 在工具栏上选择“保存”。 

此帐户中创建的实验室连接到所选虚拟网络。 他们可以访问所选虚拟网络中的资源。 有关详细信息，请参阅[实验室的网络连接 Azure 实验室服务中的对等虚拟网络与](how-to-connect-peer-virtual-network.md)。

当你选择的虚拟网络**对等虚拟网络**字段中，**允许实验室创建者选择实验室位置**选项处于禁用状态。 这是因为实验室的实验室帐户中必须是在实验室帐户才能连接对等虚拟网络中的资源所在的同一区域中。 

## <a name="allow-lab-creator-to-pick-location-for-the-lab"></a>允许实验室创建者选择实验室位置
您可以允许实验室创建者在实验室帐户的位置不同的位置中创建实验室，通过执行以下步骤： 

1. 上**实验室帐户**页上，选择**实验室配置**在左侧菜单中。
2. 有关**允许实验室创建者选择实验室位置**，选择**已启用**如果你想要能够选择的位置在实验室的实验室创建者。 如果已禁用，实验室会自动创建实验室帐户所在的同一位置中。 
    
    此字段将被禁用时选择的虚拟网络**对等虚拟网络**字段。 这是因为实验室的实验室帐户中必须是为实验室帐户，以便他们能够访问对等虚拟网络中的资源位于同一区域。 
1. 在工具栏上选择“保存”。 

    ![配置实验室位置设置](../media/how-to-manage-lab-accounts/labs-configuration-page-lab-location.png)


## <a name="specify-an-address-range-for-vms-in-the-lab"></a>在实验室中为 Vm 指定的地址范围
以下过程包含在实验室中为 Vm 指定的地址范围的步骤。 如果更新以前指定的范围，修改后的地址范围仅适用于已进行了更改后创建的 Vm。 

指定应该牢记于心的地址范围时，以下是一些限制。 

- 前缀必须小于或等于 23。 
- 如果虚拟网络对等互连到实验室帐户，提供的地址范围不能与从对等互连的虚拟网络的地址范围重叠。

1. 上**实验室帐户**页上，选择**实验室配置**在左侧菜单中。
2. 有关**地址范围**字段中，将在实验室中创建的 vm 指定的地址范围。 应为无类域间路由 (CIDR) 表示法的地址范围 (示例：10.20.0.0/23). 将此地址范围中创建实验室中的虚拟机。
3. 在工具栏上选择“保存”。 

    ![配置地址范围](../media/how-to-manage-lab-accounts/labs-configuration-page-address-range.png)

## <a name="add-a-user-to-the-lab-creator-role"></a>将用户添加为“实验室创建者”角色
若要在实验室帐户中设置课堂实验室，用户必须是实验室帐户中“实验室创建者”角色的成员。 用来创建实验室帐户的帐户会自动添加到此角色。 如果打算使用同一用户帐户创建课堂实验室，可以跳过此步骤。 若要使用其他用户帐户创建教室实验室，请执行以下步骤： 

为教师提供为课程创建实验室的权限，并将他们添加为**实验室创建者**角色：

1. 在“实验室帐户”页上，选择“访问控制(IAM)”，然后单击工具栏中的“+ 添加角色分配”。 

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
请参阅以下文章：

- [以实验室所有者身份创建并管理实验室](how-to-manage-classroom-labs.md)
- [以实验室所有者身份设置并发布模板](how-to-create-manage-template.md)
- [以实验室所有者身份配置并控制实验室的使用](how-to-configure-student-usage.md)
- [以实验室用户身份访问教室实验室](how-to-use-classroom-lab.md)
