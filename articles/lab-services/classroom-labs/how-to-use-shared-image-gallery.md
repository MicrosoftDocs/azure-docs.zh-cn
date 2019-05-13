---
title: 使用 Azure 实验室服务中共享的映像库 |Microsoft Docs
description: 了解如何配置要使用共享的映像库，以便用户可以与其他共享映像和另一个用户可以使用图像来在实验室中创建模板 VM 的实验室帐户。
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
ms.openlocfilehash: 8d8b6fffe197d4180b091518dcd1615d0e0b9d19
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412856"
---
# <a name="use-a-shared-image-gallery-in-azure-lab-services"></a>使用 Azure 实验室服务中共享的映像库
本文介绍如何教师/实验室管理员可以将保存模板的虚拟机映像，以便其他人重复使用。 这些映像保存在 Azure 中[共享的映像库](../../virtual-machines/windows/shared-image-galleries.md)。 第一步，实验室管理员将一个现有的共享的映像库附加到实验室帐户。 一旦将附加的共享的映像库，实验室的实验室帐户中创建可以将图像保存到共享的映像库。 其他教师可以从共享的映像库创建它们的类模板中选择此映像。 

## <a name="prerequisites"></a>必备组件
- 通过使用创建共享的映像库[Azure PowerShell](../../virtual-machines/windows/shared-images.md)或[Azure CLI](../../virtual-machines/linux/shared-images.md)。
- 已附加到实验室帐户共享的映像库。 有关分步说明，请参阅[如何附加或分离共享的映像库](how-to-attach-detach-shared-image-gallery.md)。


## <a name="save-an-image-to-the-shared-image-gallery"></a>将图像保存到共享的映像库
附加共享的映像库后，教师可以保存或将图像上载到共享的映像库，以便可以重用由其他教师。 将图像上载到共享的映像库的说明，请参阅[共享映像库概述](../../virtual-machines/windows/shared-images.md)。 

> [!NOTE]
> 原因是当前，教室实验室用户界面 (UI) 不支持将实验室图像保存到共享的映像库。 

## <a name="use-an-image-from-the-shared-image-gallery"></a>使用共享的映像库中的图像
教师/教授可以在创建新实验室过程选取了模板的共享的映像库中的可用的自定义映像。

![从库中使用虚拟机映像](../media/how-to-use-shared-image-gallery/use-shared-image.png)

## <a name="next-steps"></a>后续步骤
有关共享的映像库的详细信息，请参阅[共享的映像库](../../virtual-machines/windows/shared-image-galleries.md)。
