---
title: 远程监视中的电子邮件操作 - Azure | Microsoft Docs
description: 本操作指南介绍了如何将电子邮件操作添加到新的或现有的规则。
author: asdonald
manager: hegate
ms.author: asdonald
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 11/12/2018
ms.topic: conceptual
ms.openlocfilehash: fbb5f92258ff31dd7077bb1ade7fa7e5644c8bac
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466939"
---
# <a name="add-an-email-action"></a>添加电子邮件操作

电子邮件操作有助于确保你不会错过警报。 可以向现有规则添加电子邮件操作，也可以在创建新规则时添加电子邮件操作。

要完成本操作指南中的步骤，需要在 Azure 订阅中部署远程监视解决方案加速器实例。

要创建或修改规则，你必须是[“管理员”或具有正确的权限](iot-accelerators-remote-monitoring-rbac.md)。

## <a name="edit-an-existing-rule"></a>编辑现有规则

请按照以下步骤将电子邮件操作添加到现有规则：

1. 导航到远程监视解决方案。

1. 从“仪表板”导航到“规则”页面：

    ![“规则”页面](./media/iot-accelerators-remote-monitoring-email-actions/rules-email.png)

1. 单击要修改的现有规则旁边的复选框，然后单击顶部的“编辑”。 随即出现可编辑的“规则”面板。

1. 在“操作”部分中，将“已启用电子邮件”切换为“开”。

1. 首次在解决方案加速器中启用电子邮件操作时，必须[登录 Outlook](#outlook)。

1. 在收件人框中输入电子邮件地址，然后按每个电子邮件地址的“Enter”键以添加：

    ![地址项](./media/iot-accelerators-remote-monitoring-email-actions/address-email.png)

1. 输入电子邮件的主题。

1. 以纯文本格式输入电子邮件收件人的任何其他备注。 如果[编辑电子邮件模板](#htmledit)，可以使用 HTML 格式设置。

1. 确保“规则状态”设置为“已启用”。

1. 单击“应用”。

## <a name="create-a-new-rule"></a>创建新规则

创建新规则时，请按照以下步骤添加电子邮件操作：

1. 导航到远程监视解决方案。

1. 从“仪表板”导航到“规则”页面：

    ![“规则”页面](./media/iot-accelerators-remote-monitoring-email-actions/rules-email.png)

1. 请按照[创建规则部分](iot-accelerators-remote-monitoring-automate.md#create-a-rule)中的步骤进行操作。 请按照[创建高级规则](iot-accelerators-remote-monitoring-automate.md#create-an-advanced-rule)部分中的步骤进行操作，直到设置“严重级别”。 此时请勿单击“应用”。

1. 在“操作”部分中，将“已启用电子邮件”切换为“开”。

1. 首次在解决方案加速器中启用电子邮件操作时，必须[登录 Outlook](#outlook)。

1. 在收件人框中输入电子邮件地址，然后按每个电子邮件地址的“Enter”键以添加：

    ![地址项](./media/iot-accelerators-remote-monitoring-email-actions/address-email.png)

1. 输入电子邮件的主题。

1. 以纯文本格式输入电子邮件收件人的任何其他备注。 如果[编辑电子邮件模板](#htmledit)，可以使用 HTML 格式设置。

1. 确保“规则状态”设置为“已启用”。

1. 单击“应用”。

现已启用包含电子邮件操作的规则。 每次触发操作时，新的电子邮件都会发送给收件人。

## 登录 Outlook <a name="outlook"></a>

首次在解决方案加速器中启用电子邮件操作时，必须登录 Outlook。 此操作设置了发送电子邮件通知的电子邮件帐户。

> [!NOTE]
> 应只为解决方案加速器通知创建特定的 Outlook 帐户，并在启用你的第一个电子邮件操作时使用该帐户。

### <a name="contributor-role-outlook-setup"></a>参与者角色 Outlook 安装向导

如果订阅中具有“参与者”角色的某个人部署了解决方案加速器，则该应用程序没有足够的权限来通过 Web UI 设置和验证电子邮件操作。

在开始之前，请创建 Outlook 帐户以用于从解决方案加速器发送电子邮件通知。

以下步骤演示了如何手动设置和验证电子邮件操作：

1. 导航到 [Azure 门户](https://portal.azure.com)。

1. 导航到解决方案加速器的资源组。

1. 单击“office365-connector”：

    ![API 连接](./media/iot-accelerators-remote-monitoring-email-actions/apiconnector1.png)

1. 单击横幅，开始授权过程：

    ![授权](./media/iot-accelerators-remote-monitoring-email-actions/connector1.png)

1. 单击“授权”。 系统会提示你进行登录。 用于登录的帐户应是该应用程序用于发送电子邮件通知的电子邮件地址：

    ![授权按钮](./media/iot-accelerators-remote-monitoring-email-actions/authorize.png)

1. 在底部单击“保存”。 如果横幅消失，则授权将会成功。

1. 要更改发送通知的电子邮件地址，请单击“编辑 API 连接”。

    ![更改电子邮件](./media/iot-accelerators-remote-monitoring-email-actions/editemail1.png)

### <a name="owner-role-outlook-setup"></a>所有者角色 Outlook 设置

如果订阅中具有“所有者”角色的某个人部署了解决方案加速器，则该应用程序可以验证是否已通过 Web UI 正确设置了电子邮件操作。

在开始之前，请创建 Outlook 帐户以用于从解决方案加速器发送电子邮件通知。

以下步骤可以帮助你登录并设置电子邮件操作：

1. 单击以登录 Outlook。 随即转到 Azure 门户：

   ![登录 Outlook](./media/iot-accelerators-remote-monitoring-email-actions/owneroutlook-email.png)

1. 单击“授权”。 系统会提示你进行登录。 用于登录的帐户应是该应用程序用于发送电子邮件通知的电子邮件地址：

1. 单击“ **保存**”。 返回到解决方案加速器并刷新页面。

1. 如果已成功配置电子邮件通知，则会显示以下消息：

   ![Outlook 登录成功](./media/iot-accelerators-remote-monitoring-email-actions/success-email.png)

## 自定义电子邮件 HTML <a name="htmledit"></a>

现成的远程监视解决方案加速器为电子邮件操作提供了基本的 HTML 模板。 电子邮件模板使用电子邮件操作设置中的值。 下面是电子邮件示例：

![电子邮件示例](./media/iot-accelerators-remote-monitoring-email-actions/emailtemplate1.png)

以下步骤介绍如何编辑 HTML 电子邮件模板。 例如，可以包含更多信息或添加自定义图像：

1. 克隆 Java 或 .NET 远程监视 GitHub 存储库：

1. 导航到电子邮件模板位置：
  
    `Dotnet: device-telemetry\ActionsAgent\data\EmailTemplate.html`
  
    `Java device-telemetry/app/resources/data/EmailTemplate.html`

1. 可以在此模板中添加或删除任何参数以自定义消息。 此外，还可以根据需要添加、删除或替换调用：

    例如，在 .NET 代码中：`emailTemplate = emailTemplate.Replace("${subject}", emailAction.GetSubject());`

    例如，在 Java 代码中：`this.emailTemplate.replace("${subject}", emailAction.GetSubject());`

1. 模板中的参数采用 `${...}` 的形式。 要删除参数，请删除所需的行。 要添加参数，请添加包含要插入的值的行。

1. 要添加图像或自定义文本，请直接更新 EmailTemplate.HTML 文件。

## <a name="throttling"></a>限制

远程监视解决方案加速器使用 Outlook 发送电子邮件通知。 Outlook 将发送的电子邮件数量限制为[每 1 分钟 30 封电子邮件](https://docs.microsoft.com/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits#receiving-and-sending-limits)。 接收电子邮件的电子邮件客户端也可以限制每分钟收到的电子邮件数量。 有关限制事宜，请查看特定电子邮件客户端。 设置规则的电子邮件通知时，规则应计算至少一分钟的平均值，而不是使用即时值：

![平均值计算](./media/iot-accelerators-remote-monitoring-email-actions/calculation-email.png)

## <a name="next-steps"></a>后续步骤

本指南介绍了如何在远程监视解决方案中将电子邮件操作添加到新的或现有规则。 本指南还介绍了如何编辑定义消息格式的 HTML。

建议下一步了解[如何使用警报并修复设备问题](iot-accelerators-remote-monitoring-maintain.md)。
