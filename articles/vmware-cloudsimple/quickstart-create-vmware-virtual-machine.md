---
title: Azure 的 VMware 解决方案 CloudSimple 快速入门-使用 Azure 上的 VMware Vm
description: 了解如何配置和使用 VMware Vm 从 Azure 门户使用 Azure CloudSimple VMware 解决方案
author: sharaths-cs
ms.author: dikamath
ms.date: 04/11/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: c0bb8d7a5a1ea30b704b44c9337cd28043597ff7
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209528"
---
# <a name="quickstart---consume-vmware-vms-on-azure"></a>快速入门-使用 Azure 上的 VMware Vm

若要在 Azure 门户中创建虚拟机，请为你的订阅使用 CloudSimple 管理员已启用的虚拟机模板。 这些虚拟机模板位于 VMware 基础结构上。

## <a name="cloudsimple-vm-creation-on-azure-requires-a-vm-template"></a>在 Azure 上的 CloudSimple VM 创建所需的 VM 模板

从 vCenter UI 在私有云上创建虚拟机。 若要创建模板，请按照中的说明[克隆到 vSphere Web 客户端中的模板的虚拟机](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-FE6DE4DF-FAD0-4BB0-A1FD-AFE9A40F4BFE.html)。 在私有云 vCenter 上存储的 VM 模板。

## <a name="create-a-virtual-machine-in-the-azure-portal"></a>在 Azure 门户中创建虚拟机

1. 选择“所有服务”。

2. 搜索**CloudSimple 虚拟机**。

3. 单击“添加”。

    ![创建 CloudSimple 虚拟机](media/create-cloudsimple-virtual-machine.png)

4. 输入基本信息，请单击**下一步： 大小**。

    ![创建 CloudSimple 虚拟机的基础知识](media/create-cloudsimple-virtual-machine-basic-info.png)

    | 字段 | 描述 |
    | ------------ | ------------- |
    | 订阅 | 与您的私有云关联的 azure 订阅。  |
    | 资源组 | 要向其分配 VM 的资源组。 可以选择现有组或新建一个。 |
    | 名称 | 若要确定 VM 的名称。  |
    | Location | 在其中承载此 VM 的 azure 区域。  |
    | 私有云 | 你想要创建虚拟机 CloudSimple 私有云。 |
    | 资源池 | VM 的映射的资源池。 选择从可用的资源池。 |
    | vSphere 模板 | VM 的 vSphere 模板。  |
    | 用户名 | 用户名的虚拟机管理员 （适用于 Windows 模板）|
    | 密码 |  （适用于 Windows 模板） VM 管理员密码。 |
    | 确认密码 | 确认密码 |

5. 选择的核心数和内存容量的 VM，然后单击**下一步： 配置**。 如果你想要公开与来宾操作系统的完整 CPU 虚拟化，请选中复选框。 需要硬件虚拟化的应用程序可以无需二进制转换或半虚拟化的虚拟机上运行。 有关详细信息，请参阅 VMware 文章<a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html" target="_blank">公开 VMware 硬件辅助虚拟化</a>。

    ![创建 CloudSimple 虚拟机的大小](media/create-cloudsimple-virtual-machine-size.png)

6. 下表中所述配置网络接口和磁盘，然后单击**查看 + 创建**。

    ![创建 CloudSimple 虚拟机-配置](media/create-cloudsimple-virtual-machine-configurations.png)

    对于网络接口，请单击**添加网络接口**和配置下列设置。
    
    | 控制 | 描述 |
    | ------------ | ------------- |
    | 名称 | 输入用于标识接口的名称。  |
    | 网络 | 从私有云 vSphere 中配置的分布式的端口组的列表中选择。  |
    | 适配器 | 从为 VM 配置的可用类型列表中选择 vSphere 适配器。 有关详细信息，请参阅 VMware 知识库文章<a href="https://kb.vmware.com/s/article/1001805" target="_blank">选择用于你的虚拟机的网络适配器</a>。 |
    | 在启动开机 | 选择是否要在启动 VM 时，启用 NIC 硬件。 默认值是**启用**。 |

    对于磁盘，请单击**添加磁盘**和配置下列设置。

    | 项 | 描述 | 
    | ------------ | ------------- | 
    | 名称 | 输入用于标识磁盘的名称。  | 
    | 大小 | 选择其中一个可用的大小。  | 
    | SCSI 控制器 | 选择该磁盘的 SCSI 控制器。  |
    | 模式 | 确定如何在快照中加入该磁盘。 选择以下选项之一： <br> 的永久独立于：向磁盘写入的所有数据将永久都写入。<br> 的非持久性独立于：当你关闭或重置虚拟机，向磁盘写入的更改将被丢弃。  独立的非持久性模式，可始终处于相同的状态重启 VM。 有关详细信息，请参阅<a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html" target="_blank">VMware 文档</a>。

7. 完成验证后，查看设置，然后单击**创建**。 若要进行任何更改，请单击顶部的选项卡，或单击。

    ![创建 CloudSimple 虚拟机-查看](media/create-cloudsimple-virtual-machine-review.png)

## <a name="next-steps"></a>后续步骤

* [查看 CloudSimple 虚拟机的列表](https://docs.azure.cloudsimple.com/azure-create-vm/#view-list-of-cloudsimple-virtual-machines)
* [从 Azure 管理 CloudSimple 虚拟机](https://docs.azure.cloudsimple.com/azure-manage-vm/)
