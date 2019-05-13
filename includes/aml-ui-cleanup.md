---
title: include 文件
description: include 文件
services: machine-learning
ms.service: machine-learning
ms.custom: include file
ms.topic: include
author: sgilley
ms.author: sgilley
ms.date: 05/06/2019
ms.openlocfilehash: 623e993dfbe6bbb3297fa6470865ab1a04f55b37
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028585"
---
>[!IMPORTANT]
>为系统必备组件的其他 Azure 机器学习服务教程和操作方法文章，可以使用你创建的资源。


### <a name="delete-everything"></a>删除所有内容

如果不打算使用您创建的任何内容，删除整个资源组，因此不会产生任何费用：

1. 在 Azure 门户中，选择**资源组**在窗口的左侧。
 
   ![在 Azure 门户中删除资源组](./media/aml-ui-cleanup/delete-resources.png)

1. 在列表中，选择你创建的资源组。

1. 在窗口的右侧，选择省略号按钮 (**...**).

1. 选择“删除资源组”。

删除资源组也会删除在可视界面中创建的所有资源。  

### <a name="delete-only-the-compute-target"></a>删除仅计算目标

此处创建计算目标*自动措施*到零个节点时不使用。 这是为了尽可能减少费用。 如果你想要删除计算目标，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com)中打开你的工作区。

    ![删除计算目标](./media/aml-ui-cleanup/delete-compute-target.png)

1. 在中**计算**部分的工作区中，选择的资源。

1. 选择“删除”。

### <a name="delete-individual-assets"></a>删除各项资产

在其中创建试验的可视界面，通过选择它们，并选择删除单个资产**删除**按钮。

![删除试验](./media/aml-ui-cleanup/delete-experiment.png)
