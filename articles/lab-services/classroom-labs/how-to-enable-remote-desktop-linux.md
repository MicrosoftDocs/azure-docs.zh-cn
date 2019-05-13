---
title: 适用于 Linux Azure 实验室服务中启用远程桌面 |Microsoft Docs
description: 了解如何为 Linux 虚拟机的实验室中 Azure 实验室服务启用远程桌面。
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
ms.date: 03/28/2019
ms.author: spelluru
ms.openlocfilehash: 9ad6f82d7b9bd7c4957df1dd37d0f2ddf7462e9e
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410954"
---
# <a name="enable-and-use-remote-desktop-for-linux-virtual-machines-in-a-lab-in-azure-lab-services"></a>启用和使用 Azure 实验室服务中的实验室中的 Linux 虚拟机的远程桌面
本文介绍如何执行以下任务：

- 为 Linux VM 启用远程桌面
- 如何教师可以连接到的模板 VM 通过远程桌面连接 (RDP)。
- 学生如何向学生 VM 通过 RDP 连接

## <a name="enable-remote-desktop-for-linux-vm"></a>为 Linux VM 启用远程桌面
在过程中创建实验室，教师可以启用**远程桌面连接**有关**Linux**映像。 **启用远程桌面连接**何时选择该模板的 Linux 映像，则显示选项。 启用此选项后，教师可以连接到模板，VM 和学生 Vm 通过 RDP （远程桌面）。 

![启用远程桌面连接的 Linux 映像](../media/how-to-enable-remote-desktop-linux/enable-rdp-option.png)

> [!IMPORTANT] 
> 启用**远程桌面连接**仅打开**RDP** Linux 计算机上的端口。 您作为一名教师，连接到 Linux 计算机首次使用 SSH 和安装 RDP 和 GUI 包，以便可以连接到更高版本使用 RDP 的 Linux 计算机。 然后，你**发布**图像，以便学生可以通过中的 RDP 连接到 Linux Vm 的学生。 

## <a name="supported-operating-systems"></a>支持的操作系统
目前，以下操作系统支持远程桌面连接：

- openSUSE Leap 42.3
- 基于 CentOS 的 7.5
- Debian 9“Stretch”
- Ubuntu Server 16.04 LTS

## <a name="teachers-connecting-to-the-template-vm-using-rdp"></a>连接到 VM 的模板的教师使用 RDP
教师必须连接到的模板 VM 首先，使用 SSH 和在其上安装 RDP 和 GUI 包。 然后，教师可以使用以下步骤连接到使用 RDP 在 Linux 虚拟机： 

您看到**远程桌面**选项以在创建实验室时连接到 VM 的模板。 

![在创建时连接到通过 RDP 的模板](../media/how-to-enable-remote-desktop-linux/connect-at-creation.png)

您看到**远程桌面**启动后创建实验室的实验室的主页和模板 VM 上的选项。 如果未启动，启动 VM 的模板。 

![创建实验室后连接到通过 RDP 的模板](../media/how-to-enable-remote-desktop-linux/rdp-after-lab-creation.png) 

当选择**RDP**选项，它将下载的 RDP 文件。 您将其打开以连接到 Linux 计算机。 

## <a name="teachers-connecting-to-a-student-vm-using-rdp"></a>连接到 VM 的学生的教师使用 RDP
实验室所有者 （教师/教授） 可以连接到 VM 的学生通过切换到**虚拟机**查看，并选择**连接**图标。 在此之前，必须教师**发布**通过 RDP 和 GUI 包在其上安装的模板映像。 

![连接到 VM 的学生的教师](../media/how-to-enable-remote-desktop-linux/teacher-connect-to-student-vm.png)

## <a name="students-connecting-to-the-student-vm"></a>连接到 VM 的学生的学生
实验室所有者 （教师/教授） 后，学生可以中它们对 Linux Vm 的 RDP**发布**模板在计算机上安装的 VM 通过 RDP 和 GUI 包。 下面是相关步骤： 

1. 当一名学生在登录到 Labs 门户网站直接 (`https://labs.azure.com`) 或通过使用注册链接 (`https://labs.azure.com/register/<registrationCode>`)，显示一个磁贴的每个实验室学生有权访问。 
2. 在磁贴中，选择**启动**如果 VM 已停止。 
3. 选择“连接”。 此操作下载 RDP 文件到你的计算机。 将其保存并打开以连接到 Linux 计算机通过 RDP。 

    ![学生 VM 的 RDP 下载](../media/how-to-enable-remote-desktop-linux/student-rdp-download.png)

    您仍可以使用 SSH 连接到 Linux VM。 选择 **...（省略号）** 才能看到 SSH 选项。 
    
    ![学生 VM 的 ssh 配合使用](../media/how-to-enable-remote-desktop-linux/student-ssh.png)

    复制并保存上的 SSH 连接字符串**连接到虚拟机**对话框。 从 SSH 终端使用此连接字符串（如 [Putty](https://www.putty.org/)）连接到虚拟机。 

## <a name="next-steps"></a>后续步骤
请参阅以下文章：

- [以管理员身份创建并管理实验室帐户](how-to-manage-lab-accounts.md)
- [以实验室所有者身份创建并管理实验室](how-to-manage-classroom-labs.md)
- [以实验室所有者身份设置并发布模板](how-to-create-manage-template.md)
- [以实验室用户身份访问教室实验室](how-to-use-classroom-lab.md)

