---
title: 准备 Azure 资源，以便对本地计算机进行灾难恢复
description: 了解如何使用 Azure Site Recovery 准备 Azure，对本地 Hyper-V VM 进行灾难恢复。
author: rayne-wiselman
ms.service: site-recovery
services: site-recovery
ms.topic: tutorial
ms.date: 04/08/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 29189a5919a01fcb897758fb64ca9e84b9381fb6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410906"
---
# <a name="prepare-azure-resources-for-disaster-recovery-of-on-premises-machines"></a>准备 Azure 资源，以便对本地计算机进行灾难恢复

 [Azure Site Recovery](site-recovery-overview.md) 通过在计划内和计划外中断期间使商业应用保持运行状态，来帮助实现业务连续性和灾难恢复 (BCDR)。 Site Recovery 管理并安排本地计算机和 Azure 虚拟机 (VM) 的灾难恢复，包括复制、故障转移和恢复。

本文是一部系列教程的第一篇文章，介绍如何为本地 Hyper-V VM 设置灾难恢复。

> [!NOTE]
> 我们已编写相应的教程来介绍方案的最简单部署路径。 这些教程尽可能使用默认选项，并且不显示所有可能的设置和路径。 有关详细信息，请参阅每个相应方案的“操作指南”部分。

本教程介绍如何在将本地 VM (Hyper-V) 复制到 Azure 时准备 Azure 组件。 将了解如何执行以下操作：

> [!div class="checklist"]
> * 验证 Azure 帐户是否具有复制权限。
> * 创建一个 Azure 存储帐户用于存储复制的计算机的映像。
> * 创建一个恢复服务保管库，用于存储 VM 和其他复制组件的元数据与配置信息。
> * 设置 Azure 网络。 在故障转移后创建的 Azure VM 将加入此网络。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="sign-in"></a>登录

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="verify-account-permissions"></a>验证帐户权限

如果你刚刚创建了一个免费的 Azure 帐户，那么你就是该订阅的管理员。 如果你不是管理员，请要求管理员分配你所需的权限。 若要为新虚拟机启用复制，必须有权执行以下操作：

- 在所选资源组中创建 VM。
- 在所选虚拟网络中创建 VM。
- 向所选存储帐户进行写入。

若要完成这些任务，应为帐户分配“虚拟机参与者”内置角色。 若要在保管库中管理 Site Recovery 操作，应为帐户分配“Site Recovery 参与者”内置角色。

## <a name="create-a-storage-account"></a>创建存储帐户

已复制计算机的映像保存在 Azure 存储中。 在从本地故障转移到 Azure 时，会从该存储中创建 Azure VM。 存储帐户必须位于与恢复服务保管库相同的区域。

1. 在 [Azure 门户](https://portal.azure.com)菜单中，选择“创建资源” > “存储” > “存储帐户 - blob、文件、表、队列”。
2. 在“创建存储帐户”中，输入帐户的名称。  选择的名称必须在 Azure 中唯一，长度为 3 到 24 个字符，且只能使用小写字母和数字。 本教程使用 **contosovmsacct1910171607**。
3. 在“部署模型”中，选择“资源管理器”。
4. 在“帐户类型”中，选择“存储(常规用途 v1)”。 请不要选择 blob 存储。
5. 在“复制”中，选择默认的“读取访问异地冗余存储”作为存储冗余。 将“需要安全传输”设置保留为“已禁用”。
6. 在“性能”中，选择“标准”。 接下来，在“访问层级”中选择默认选项“热”。
7. 在“订阅”中，选择要在其中创建新存储帐户的订阅。
8. 在“资源组”中，输入新的资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 本教程使用 **ContosoRG**。
9. 在“位置”中，选择存储帐户的地理位置。 本教程使用“西欧”。
10. 选择“创建”以创建存储帐户。

   ![创建存储帐户](media/tutorial-prepare-azure/create-storageacct.png)

## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

1. 在 Azure 门户中选择“+创建资源”，然后在 Azure 市场中搜索“恢复服务”。
2. 选择“备份和站点恢复(OMS)”。 接下来，在“备份和站点恢复”页上，选择“创建”。
1. 在“恢复服务保管库”>“名称”中，输入一个易记名称用于标识该保管库。 在本教程中，请使用 ContosoVMVault。
2. 在**资源组**中，选择现有资源组或创建新资源组。 本教程使用 **contosoRG**。
3. 在“位置”中，选择该保管库要放到的区域。 本教程使用“西欧”。
4. 若要从仪表板快速访问保管库，请选择“固定到仪表板” > “创建”。

![创建新的保管库](./media/tutorial-prepare-azure/new-vault-settings.png)

新保管库显示在“仪表板” > “所有资源”中，以及“恢复服务保管库”主页上。

## <a name="set-up-an-azure-network"></a>设置 Azure 网络

在故障转移后创建的 Azure VM 会加入此网络。

1. 在 [Azure 门户](https://portal.azure.com)中，选择“创建资源” > “网络” > “虚拟网络”。 保留选择“资源管理器”作为部署模型。
2. 在“名称”中，输入网络名称。 名称在 Azure 资源组中必须唯一。 本教程使用 **ContosoASRnet**。
3. 指定要在其中创建网络的资源组。 本教程使用现有资源组 **contosoRG**。
4. 在“地址范围”中，输入 **10.0.0.0/24** 作为网络范围。 此网络没有子网。
5. 在“订阅”中，选择要在其中创建网络的订阅。
6. 在“位置”中，选择“西欧”。 该网络必须位于与恢复服务保管库相同的区域中。
7. 保留基本 DDoS 防护的默认选项，网络上没有服务终结点。
8. 选择“创建”。

![创建虚拟网络](media/tutorial-prepare-azure/create-network.png)

创建虚拟网络需要几秒钟的时间。 创建后，可在 Azure 门户仪表板中看到它。

## <a name="useful-links"></a>有用链接

学习内容：
- [Azure 网络](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)
- [托管磁盘](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview)



## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [准备本地 Hyper-V 基础结构以灾难恢复到 Azure](hyper-v-prepare-on-premises-tutorial.md)
