---
title: include 文件
description: include 文件
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 05/06/2019
ms.openlocfilehash: cf35651f7dd839e8792029851b9bfe278036624c
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028510"
---
在计算目标，连接到你的工作区的计算资源上运行试验。  一旦创建计算目标，可以重用其用于将来的运行。

1. 选择**运行**底部以运行试验。

     ![运行试验](./media/aml-ui-create-training-compute/run-experiment.png)

1. 当**安装程序计算目标**对话框出现时，如果你的工作区已有的计算资源，你现可选择它。  否则，请选择**新建**。

    > [!NOTE]
    > 可视界面仅可以在机器学习计算目标上运行试验。 不会显示其他计算目标。

1. 提供计算资源的名称。

1. 选择“运行”。

    ![设置计算目标](./media/aml-ui-create-training-compute/set-compute.png)

    现在将创建的计算资源。 查看在实验的右上角的状态。 

    > [!NOTE]
    > 需要大约 5 分钟，若要创建的计算资源。 创建资源后，可以重用它，并跳过以后用来运行这段等待时间。
    >
    > 计算资源将为 0 个节点的自动缩放时处于空闲状态以节省成本。  当您在延迟后再次使用它时，可能会再次遇到的等待时间大约为 5 分钟时，它会回纵向。
