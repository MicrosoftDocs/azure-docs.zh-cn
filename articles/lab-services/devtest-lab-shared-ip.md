---
title: 了解 Azure 开发测试实验室中的共享的 IP 地址 |Microsoft 文档
description: 了解 Azure 开发测试实验室如何使用共享 IP 地址以最大程度地减少访问实验室 VM 所需的公共 IP 地址。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/12/2019
ms.author: spelluru
ms.openlocfilehash: f7c9feedddab1aea031cb3a8879e868aae04df00
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236869"
---
# <a name="understand-shared-ip-addresses-in-azure-devtest-labs"></a>了解 Azure 开发测试实验室中的共享的 IP 地址

Azure 开发测试实验室允许实验室 VM 共享相同的 IP 地址，以最大程度地减少访问各个实验室 VM 所需的公共 IP 地址数。  本文介绍了共享 IP 的工作原理及其相关的配置选项。

## <a name="shared-ip-setting"></a>共享 IP 设置

在创建实验室时，在其中创建虚拟网络的子网。  默认情况下，创建此子网时会将“启用共享公共 IP”设置为“是”。  此配置会为整个子网配置一个公共 IP 地址。  若要深入了解如何配置虚拟网络和子网，请参阅[在 Azure 开发测试实验室中配置虚拟网络](devtest-lab-configure-vnet.md)。

![新建实验室子网](media/devtest-lab-shared-ip/lab-subnet.png)

对于现有的实验室，可以通过选择“配置和策略”>“虚拟网络”启用此选项。 然后，从列表中选择虚拟网络并为所选的子网选择“启用共享的公共 IP”。 如果不想在实验室 Vm 之间共享一个公共 IP 地址，也可以禁用此选项在所有实验室中。

在此实验室中创建的任何 VM 都将默认设置为使用共享 IP。  在创建 VM 时，此设置可以观察到在**高级设置**页**IP 地址配置**。

![新建 VM](media/devtest-lab-shared-ip/new-vm.png)

- **共享：** 为创建的所有 Vm**共享**放入一个资源组 (RG)。 将向该 RG 分配一个 IP 地址，RG 中的所有 VM 将使用该 IP 地址。
- **公共：** 您创建的每个 VM 都有其自己的 IP 地址，并在其自己的资源组中创建。
- **专用：** 您创建的每个 VM 使用专用 IP 地址。 不能直接从 internet 与远程桌面连接到此 VM。

每当启用了共享 ip 的 VM 添加到子网时，开发测试实验室将自动将 VM 添加到负载均衡器，并将分配的公共 IP 地址，一个 TCP 端口号转发到 VM 上的 RDP 端口。  

## <a name="using-the-shared-ip"></a>使用共享 IP

- **Linux 用户：** 通过 ssh 连接到虚拟机使用的 IP 地址或完全限定的域名后, 跟一个冒号和端口。 例如，在下图中，用来连接到 VM 的 RDP 地址是 `mydevtestlab597975021002.eastus.cloudapp.azure.com:50661`。

  ![VM 示例](media/devtest-lab-shared-ip/vm-info.png)

- **Windows 用户：** 选择**Connect**下载预配置的 RDP 文件，并访问 VM 在 Azure 门户上的按钮。

## <a name="next-steps"></a>后续步骤

* [在 Azure 开发测试实验室中定义实验室策略](devtest-lab-set-lab-policy.md)
* [在 Azure 开发测试实验室中配置虚拟网络](devtest-lab-configure-vnet.md)





