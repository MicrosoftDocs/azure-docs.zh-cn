---
title: 连接到 Azure 实验室服务中的对等网络 |Microsoft Docs
description: 了解如何与另一个为对等网络连接你的实验室网络。 例如，将您的本地学校/大学网络连接与在 Azure 中的实验室的虚拟网络。
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
ms.openlocfilehash: c9b305beae1b385d4714e3a80e6843c7e76a4f60
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411012"
---
# <a name="connect-your-labs-network-with-a-peer-virtual-network-in-azure-lab-services"></a>将 Azure 实验室服务中的对等虚拟网络使用实验室的网络连接 
本文提供有关对等互连实验室网络与另一个网络信息。 

## <a name="overview"></a>概述
虚拟网络对等互连可以无缝连接 Azure 虚拟网络。 建立对等互连后，出于连接目的，两个虚拟网络会显示为一个。 在对等互连的虚拟网络中的虚拟机之间的流量路由通过 Microsoft 主干基础结构，非常类似于流量通过专用 IP 地址在同一虚拟网络中的虚拟机之间的路由。 有关详细信息，请参阅[虚拟网络对等互连](../../virtual-network/virtual-network-peering-overview.md)。

您可能需要将包括以下的一些方案中的对等虚拟网络使用实验室的网络连接：

- 在实验室中的虚拟机具有连接到本地许可证服务器，以获取许可证的软件
- 在实验室中的虚拟机上大学的网络共享需要访问数据集 （或任何其他文件）。 

特定的本地网络连接到 Azure 虚拟网络可以通过[ExpressRoute](../../expressroute/expressroute-introduction.md)或[虚拟网络网关](../../vpn-gateway/vpn-gateway-about-vpngateways.md)。 Azure 实验室服务之外，必须设置这些服务。 若要了解有关将本地网络连接到 Azure 中使用 ExpressRoute 的详细信息，请参阅 [ExpressRoute 概述]） (.../expressroute/expressroute-introduction.md)。 对于本地连接使用虚拟网络网关，网关，指定虚拟网络和实验室帐户必须位于同一区域中。

## <a name="configure-at-the-time-of-lab-account-creation"></a>配置在实验室帐户创建的时间
在新的实验室帐户创建过程中，你可以选择显示中的现有虚拟网络**对等虚拟网络**下拉列表中。 所选的虚拟网络是 connected(peered) 到实验室帐户下创建的实验室。 所有虚拟机后进行创建的实验室中此更改将对等互连的虚拟网络上有权访问资源。 

![选择要对等互连的 VNet](../media/how-to-connect-peer-virtual-network/select-vnet-to-peer.png)

> [!NOTE]
> 创建实验室帐户的详细分步说明，请参阅[设置实验室帐户](tutorial-setup-lab-account.md)


## <a name="configure-after-the-lab-is-created"></a>创建实验室后配置
可以从启用相同的属性**Labs 配置**选项卡**实验室帐户**页上，如果你没有对等网络设置实验室帐户创建时。 对此设置所做的更改仅适用于在更改后创建的实验室。 您可以看到在图中，可以启用或禁用**对等虚拟网络**面向实验室的实验室帐户中。 

![启用或禁用 VNet 对等互连后创建实验室](../media/how-to-connect-peer-virtual-network/select-vnet-to-peer-existing-lab.png) 

当你选择的虚拟网络**对等虚拟网络**字段中，**允许实验室创建者选择实验室位置**选项处于禁用状态。 这是因为实验室的实验室帐户中必须是在实验室帐户才能连接对等虚拟网络中的资源所在的同一区域中。 

> [!IMPORTANT]
> 此设置的更改仅适用于后进行更改时，不到现有的实验室创建的实验室。 


## <a name="next-steps"></a>后续步骤
请参阅以下文章：

- [以管理员身份创建并管理实验室帐户](how-to-manage-lab-accounts.md)
- [以实验室所有者身份创建并管理实验室](how-to-manage-classroom-labs.md)
- [以实验室所有者身份设置并发布模板](how-to-create-manage-template.md)
- [以实验室用户身份访问教室实验室](how-to-use-classroom-lab.md)

