---
title: 附加或分离 Azure 实验室服务中共享的映像库 |Microsoft Docs
description: 了解如何将共享的映像库附加到 Azure 实验室服务中的实验室。
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
ms.openlocfilehash: de4e9fb4b15f4c346926fe46f23255c668204c2e
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413894"
---
# <a name="attach-or-detach-a-shared-image-gallery-in-azure-lab-services"></a>附加或分离 Azure 实验室服务中共享的映像库
教师/实验室管理员可以在 Azure 中保存的模板 VM 映像[共享的映像库](../../virtual-machines/windows/shared-image-galleries.md)，以便其他人重复使用。 第一步，实验室管理员将一个现有的共享的映像库附加到实验室帐户。 一旦将附加的共享的映像库，实验室的实验室帐户中创建可以将图像保存到共享的映像库。 其他教师可以从共享的映像库创建它们的类模板中选择此映像。 

本文介绍如何附加或分离到实验室帐户共享的映像库。 

## <a name="configure-at-the-time-of-lab-account-creation"></a>配置在实验室帐户创建的时间
当创建实验室帐户时，可以将共享的映像库附加到实验室帐户。 您可以从下拉列表中选择一个现有共享的映像库，或创建一个新。 若要创建并附加到实验室帐户共享的映像库，请选择**新建**，为库中，输入一个名称，并输入**确定**。 

![在创建实验室帐户时配置共享的映像库](../media/how-to-use-shared-image-gallery/new-lab-account.png)

## <a name="configure-after-the-lab-account-is-created"></a>创建实验室帐户后配置
创建实验室帐户后，可以执行以下任务：

- 创建并附加共享的映像库
- 将共享的映像库连接到实验室帐户
- 从实验室帐户共享的映像库中分离

## <a name="create-and-attach-a-shared-image-gallery"></a>创建并附加共享的映像库
1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中，选择“所有服务”。 在“DEVOPS”部分中，选择“实验室服务”。 如果选择“实验室服务”旁边的星星 (`*`)，则会将其添加到左侧菜单上的“收藏夹”部分。 从下次开始，请在“收藏夹”下选择“实验室服务”。

    ![“所有服务”->“实验室服务”](../media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. 选择实验室帐户，请参阅**实验室帐户**页。 
4. 选择**共享映像库**左侧的菜单中，然后选择 **+ 创建**工具栏上。  

    ![创建共享的图像库按钮](../media/how-to-use-shared-image-gallery/new-shared-image-gallery-button.png)
5. 在**创建共享的映像库**窗口中，输入**名称**的库，然后输入**确定**。 

    ![创建共享的映像库窗口](../media/how-to-use-shared-image-gallery/create-shared-image-gallery-window.png)

    Azure 实验室服务创建共享的映像库，并将其附加到实验室帐户。 在此实验室帐户中创建的所有实验室都具有访问权限的附加共享的映像库。 

    ![附加的图像库](../media/how-to-use-shared-image-gallery/image-gallery-in-list.png)

    在底部窗格中，可以看到共享的映像库中的映像。 在此新的库中有任何映像。 当将映像上载到库中时，你可以在此页上看到它们。     

    默认情况下启用的附加共享的映像库中的所有映像。 您可以通过启用或禁用所选的图像列表中选择它们并使用**启用所选的映像**或**禁用所选的图像**按钮。

## <a name="attach-an-existing-shared-image-gallery"></a>附加现有的共享的映像库
以下过程演示如何将现有的共享的映像库连接到实验室帐户。 

1. 上**实验室帐户**页上，选择**共享图像库**左侧的菜单中，然后选择**附加**工具栏上。 

    ![共享映像库-添加按钮](../media/how-to-use-shared-image-gallery/sig-attach-button.png)
5. 上**附加现有的共享映像库**页上，选择共享的映像库，然后选择**确定**。

    ![选择现有的库](../media/how-to-use-shared-image-gallery/select-image-gallery.png)
6. 请参阅以下屏幕： 

    ![我的库列表中](../media/how-to-use-shared-image-gallery/my-gallery-in-list.png)
    
    在此示例中，有尚没有图像在共享的映像库中。

    Azure 实验室服务标识作为参与者添加到共享的映像库中的附加到实验室。 它允许教师 / 共享的映像库的 IT 管理员要保存虚拟机映像。 在此实验室帐户中创建的所有实验室都具有访问权限的附加共享的映像库。 

    默认情况下启用的附加共享的映像库中的所有映像。 您可以通过启用或禁用所选的图像列表中选择它们并使用**启用所选的映像**或**禁用所选的图像**按钮。 

## <a name="save-an-image-to-the-shared-image-gallery"></a>将图像保存到共享的映像库
附加共享的映像库后，可以保存或将图像上载到共享的映像库，以便可以重用由其他教师实验室帐户管理员或教师。 将图像上载到共享的映像库的说明，请参阅[共享映像库概述](../../virtual-machines/windows/shared-images.md)。 

> [!NOTE]
> 原因是当前，教室实验室用户界面 (UI) 不支持将实验室图像保存到共享的映像库。 

## <a name="detach-a-shared-image-gallery"></a>分离共享的映像库
只有一个共享的映像库可以附加到实验室。 如果你想要将附加另一个共享的映像库，分离之前附加新的当前一个。 若要从你的实验室共享的映像库中分离，请选择**分离**的工具栏上，并确认分离操作。 

![从实验室帐户共享的映像库中分离](../media/how-to-use-shared-image-gallery/detach.png)

## <a name="next-steps"></a>后续步骤
若要了解有关如何将实验室图像保存到共享的映像库或者使用共享的映像库中的映像创建 VM，请参阅[如何使用共享的映像库](how-to-use-shared-image-gallery.md)。

有关详细信息在一般情况下共享图像库，请参阅[共享的映像库](../../virtual-machines/windows/shared-image-galleries.md)。
