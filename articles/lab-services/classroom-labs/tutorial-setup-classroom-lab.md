---
title: 使用 Azure 实验室服务设置课堂实验室 | Microsoft Docs
description: 在此教程中，将设置要在课堂中使用的实验室。
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/24/2019
ms.author: spelluru
ms.openlocfilehash: bdcc4349f84a35b312ecb3ad6205273b62c2e989
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64722718"
---
# <a name="tutorial-set-up-a-classroom-lab"></a>教程：设置教室实验室 
在本教程中，请设置带虚拟机的课堂实验室，供学生在课堂上使用。  

在本教程中，将执行以下操作：

> [!div class="checklist"]
> * 创建课堂实验室
> * 将用户添加到实验室
> * 将注册链接发送给学生

## <a name="prerequisites"></a>先决条件
只有充当下述角色之一的成员才能在实验室帐户中设置课堂实验室：所有者、实验室创建者或参与者。 用来创建实验室帐户的帐户会自动添加到所有者角色。

实验室所有者可以将其他用户添加到“实验室创建者”角色。 例如，实验室所有者可以将教授添加到“实验室创建者”角色。 然后，教授可以为其课堂创建带 VM 的实验室。 学生使用从教授处接收的注册链接注册到实验室。 注册以后，他们可以使用实验室中的 VM 完成课堂作业和家庭作业。 有关如何将用户添加为“实验室创建者”角色的详细步骤，请参阅[将用户添加为“实验室创建者”角色](tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role)。


## <a name="create-a-classroom-lab"></a>创建课堂实验室

1. 导航到 [Azure 实验室服务网站](https://labs.azure.com)。 请注意尚不支持 Internet Explorer 11。 
2. 选择“登录”并输入凭据。 Azure 实验室服务支持组织帐户和 Microsoft 帐户。 
3. 在“新建实验室”窗口中，执行以下操作：： 
    1. 指定实验室的**名称**。 
    2. 在实验室中指定最大**虚拟机数**。 可以在创建实验室后增加或减少 VM 数，也可以在现有实验室中进行。 有关详细信息，请参阅[更新实验室中的 VM 数](how-to-configure-student-usage.md#update-number-of-virtual-machines-in-lab)
    6. 选择“保存”。

        ![创建课堂实验室](../media/tutorial-setup-classroom-lab/new-lab-window.png)
4. 在“选择虚拟机规范”页上执行以下步骤：
    1. 选择在实验室中创建的虚拟机 (VM) 的**大小**。 目前，允许**小型**、**中型**、**大型**和 **GPU** 大小。
    3. 选择用于在实验室中创建 VM 的 **VM 映像**。 如果选择 Linux 映像，可以看到一个用来为该映像启用远程桌面连接的选项。 有关详细信息，请参阅[启用适用于 Linux 的远程桌面连接](how-to-enable-remote-desktop-linux.md)。
    4. 选择“**下一步**”。

        ![指定 VM 规范](../media/tutorial-setup-classroom-lab/select-vm-specifications.png)    
5. 在“设置凭据”页上，指定实验室中所有 VM 的默认凭据。 
    1. 指定实验室中所有 VM 的**用户名**。
    2. 指定用户的**密码**。 

        > [!IMPORTANT]
        > 记下用户名和密码。 这些信息不会再次显示。
    3. 选择“创建”。 

        ![设置凭据](../media/tutorial-setup-classroom-lab/set-credentials.png)
6. 在“配置模板”页上，查看实验室创建过程的状态。 在实验室中创建模板最长需要 20 分钟时间。 

    ![配置模板](../media/tutorial-setup-classroom-lab/configure-template.png)
7. 配置完模板后，会看到以下页： 

    ![完成配置后的“配置模板”页](../media/tutorial-setup-classroom-lab/configure-template-after-complete.png)
8. 在“配置模板”页上执行以下步骤：对于此教程来说，这些步骤是**可选**的。
    1. 选择“连接”以连接到模板 VM。 如果它是 Linux 模板 VM，请选择是要使用 SSH 还是 RDP（如果已启用 RDP）来建立连接。
    2. 在模板 VM 上安装并配置软件。     
    3. 输入模板的**说明**
9. 在模板页上选择“下一步”。 
10. 在“发布模板”页上执行以下操作。 
    1. 若要立即发布模板，请选择“发布”。  

        > [!WARNING]
        > 发布后无法取消发布。 
    2. 若要在以后发布，请选择“保存供以后发布”。 可以在完成向导后发布模板 VM。 有关如何在完成向导后进行配置和发布的详细信息，请参阅[如何管理课堂实验室](how-to-manage-classroom-labs.md)一文中的[发布模板](how-to-create-manage-template.md#publish-the-template-vm)部分。

        ![发布模板](../media/tutorial-setup-classroom-lab/publish-template.png)
11. 可以查看**发布模板的进度**。 此过程最长可能需要花费一小时。 

    ![发布模板 - 进度](../media/tutorial-setup-classroom-lab/publish-template-progress.png)
12. 成功发布模板后，会看到以下页。 选择“完成”。

    ![发布模板 - 成功](../media/tutorial-setup-classroom-lab/publish-success.png)
1. 实验室的**仪表板**随即显示。 
    
    ![课堂实验室仪表板](../media/tutorial-setup-classroom-lab/classroom-lab-home-page.png)
4. 选择左侧菜单上的“虚拟机”或选择“虚拟机”磁贴，切换到“虚拟机”页。 确认看到这些虚拟机处于“未分配”状态。 这些 VM 尚未分配给学生。 它们应该处于“已停止”状态。 可以在此页上启动学生 VM，连接到该 VM，停止该 VM，然后删除该 VM。 你可以在此页中启动这些 VM，也可以让你的学生启动它们。 

    ![处于已停止状态的虚拟机](../media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)

## <a name="add-users-to-the-lab"></a>将用户添加到实验室

1. 选择左侧菜单上的“用户”。 默认情况下，“限制访问”选项处于启用状态。 当此设置处于启用状态时，用户即使有注册链接也不能注册到实验室，除非该用户在用户列表中。 只有列表中的用户可以使用你发送的注册链接注册到实验室。 在此过程中，请将用户添加到列表。 也可关闭“限制访问”，这样一来，用户只要有注册链接就可以注册到实验室。 
2. 在工具栏上选择“添加用户”。 

    ![“添加用户”按钮](../media/how-to-configure-student-usage/add-users-button.png)
1. 在“添加用户”页上，在多个不同的行中输入电子邮件地址，或者在一行中输入以分号分隔的电子邮件地址。 

    ![添加用户电子邮件地址](../media/how-to-configure-student-usage/add-users-email-addresses.png)
4. 选择“保存”。 可以在列表中看到用户的电子邮件地址及其状态（已注册或未注册）。 

    ![用户列表](../media/how-to-configure-student-usage/users-list-new.png)


## <a name="send-an-email-with-the-registration-link"></a>发送包含注册链接的电子邮件

1. 切换到“用户”视图（如果尚未转到该页）。 
2. 在列表中选择特定或所有用户。 要选择特定用户，请选中列表第一列中的复选框。 要选择所有用户，请选中第一列标题前面的复选框（名称），或选中列表中所有用户的所有复选框。 在此列表中可以查看**邀请状态**。  在下图中，所有学生的邀请状态设置为“未发送邀请”。 

    ![选择学生](../media/tutorial-setup-classroom-lab/select-students.png)
1. 在某一行中选择**电子邮件图标（信封）**，或者在工具栏上选择“发送邀请”。 将鼠标悬停在列表中的学生姓名上也可以看到电子邮件图标。 

    ![通过电子邮件发送注册链接](../media/tutorial-setup-classroom-lab/send-email.png)
4. 在“通过电子邮件发送注册链接”页上，请按照下列步骤操作： 
    1. 键入要发送给学生的“可选邮件”。 电子邮件自动包含注册链接。 
    2. 在“通过电子邮件发送注册链接”页上，请选择“发送”。 将会看到，邀请状态先更改为“正在发送邀请”，然后更改为“已发送邀请”。 
        
        ![已发送邀请](../media/tutorial-setup-classroom-lab/invitations-sent.png)

## <a name="next-steps"></a>后续步骤
在本教程中，创建并配置了课堂实验室。 若要了解学生如何使用注册链接访问实验室中的 VM，请转到下一教程：

> [!div class="nextstepaction"]
> [连接到课堂实验室中的 VM](tutorial-connect-virtual-machine-classroom-lab.md)

