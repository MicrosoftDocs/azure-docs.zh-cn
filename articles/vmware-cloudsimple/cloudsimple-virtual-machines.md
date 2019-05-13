---
title: VMware 解决方案 CloudSimple-Azure 虚拟机概述
description: 了解有关 CloudSimple 虚拟机和其权益。
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 77a2901ae2d81f42780110b1576c1f32c7ff397b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209489"
---
# <a name="cloudsimple-virtual-machines-overview"></a>CloudSimple 虚拟机概述

CloudSimple，可在 Azure 门户中管理 VMware Vm。  群集或从 vSphere 群集资源池由将其映射到你的订阅管理通过 Azure。  CloudSimple 虚拟机从 Azure 门户将自助服务管理的 VMware Vm。  

若要从 Azure 中创建 CloudSimple VM，在私有云 vCenter 上必须存在 VM 模板。  该模板用于自定义的操作系统和应用程序。  可以强制写入的模板 VM，以满足企业安全策略。  该模板可用于创建 Vm 并从 Azure 门户中使用的自助服务模型中使用它们。

## <a name="benefits"></a>优点

CloudSimple 虚拟机从 Azure 门户提供自助服务用户创建和管理 VMware 虚拟机的机制。

* 私有云 vCenter 上创建 CloudSimple VM
* 管理 VM 属性
  * 添加/删除磁盘
  * 添加/删除 Nic
* CloudSimple VM 的电源操作
  * 开机和关机
  * 重置 VM
* 删除 VM

## <a name="next-steps"></a>后续步骤

* 了解如何[使用 Azure 上的 VMware Vm](quickstart-create-vmware-virtual-machine.md)
* 了解如何[映射你的 Azure 订阅](https://docs.azure.cloudsimple.com/azure-subscription-mapping/)