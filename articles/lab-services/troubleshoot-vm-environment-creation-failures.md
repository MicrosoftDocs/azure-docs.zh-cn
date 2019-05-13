---
title: 排查 VM 和环境创建失败问题 Azure 开发测试实验室 |Microsoft Docs
description: 了解如何进行故障排除的虚拟机 (VM) 和 Azure 开发测试实验室环境创建失败。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2019
ms.author: spelluru
ms.openlocfilehash: 7baa5e4c113e6c21c6123ac7c8399533a7dfb358
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410306"
---
# <a name="troubleshoot-virtual-machine-vm-and-environment-creation-failures-in-azure-devtest-labs"></a>虚拟机 (VM) 和 Azure 开发测试实验室环境创建失败问题进行故障排除
如果计算机名称无效或将要违反实验室策略，开发测试实验室可以为您提供警告。 有时，您能看到红色`X`旁边实验室 VM 或环境状态，告知你出现了问题。  本文提供了可用于查找潜在问题，并希望在以后避免此问题的一些技巧。

## <a name="portal-notifications"></a>门户通知
如果使用 Azure 门户，查看的第一个位置是**通知面板**。  通知面板，可通过单击主命令栏上**钟形图标**，告诉您是否实验室 VM 或环境创建是成功还是失败。  如果出现故障，你会看到与创建失败关联的错误消息。 详细信息通常提供进一步的信息来帮助你解决此问题。 在以下示例中，虚拟机创建失败由于的核心不足。 详细的消息告诉您如何修复此问题，并请求增加核心配额。

![Azure 门户通知](./media/troubleshoot-vm-environment-creation-failures/portal-notification.png)


## <a name="activity-logs"></a>活动日志
如果要尝试在 VM 或环境创建后的某个时间调查失败，请查看活动日志。 本部分演示如何查找虚拟机和环境的日志。

## <a name="activity-logs-for-virtual-machines"></a>虚拟机的活动日志

1. 在你的实验室主页上，选择要启动的 VM**虚拟机**页。
2. 上**虚拟机**页上，在**监视**部分中的左侧菜单中，选择**活动日志**若要查看与 VM 关联的所有日志。
3. 在活动日志项，选择失败的操作。 通常情况下，失败的操作称为`Write Virtualmachines`。
4. 在右窗格中，切换到 JSON 选项卡。请参阅中的日志的 JSON 视图的详细信息。

    ![为 vm 活动日志](./media/troubleshoot-vm-environment-creation-failures/vm-activity-log.png)
5. 直到您找到 JSON 日志中查找`statusMessage`属性。 它提供主要错误消息和更多详细信息，如果适用。 以下 JSON 是本文前面部分所示示例中为带引号的核心已超过错误的。

    ```json
    "properties": {
        "statusCode": "Conflict",
        "statusMessage": "{\"status\":\"Failed\",\"error\":{\"code\":\"ResourceDeploymentFailure\",\"message\":\"The resource operation completed with terminal provisioning state 'Failed'.\",\"details\":[{\"code\":\"OperationNotAllowed\",\"message\":\"Operation results in exceeding quota limits of Core. Maximum allowed: 100, Current in use: 100, Additional requested: 8. Please read more about quota increase at https://aka.ms/corequotaincrease.\"}]}}",
    },
    ```

## <a name="activity-log-for-an-environment"></a>活动日志，了解环境

若要查看环境创建的活动日志，请按照下列步骤：

1. 在你的实验室主页上，选择**配置和策略**在左侧菜单中。
2. 上**配置和策略**页上，选择**活动日志**菜单上。
3. 在日志中的活动列表中的失败位置查找并选择它。
4. 在右窗格中，切换到 JSON 选项卡，并寻找**statusMessage**。

    ![环境活动日志](./media/troubleshoot-vm-environment-creation-failures/envirionment-activity-log.png)

## <a name="resource-manager-template-deployment-logs"></a>资源管理器模板部署日志
如果您的环境或虚拟机通过自动化创建，是在一个最后一个位置以查找错误的信息。 这是 Azure 资源管理器模板部署日志。 通过自动化创建实验室资源时，它通常是通过 Azure 资源管理器模板部署。 请参阅[https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates ](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates)有关创建开发测试实验室资源的示例 Azure 资源管理器模板。

若要查看实验室模板部署日志，请按照下列步骤：

1. 启动实验室所在的资源组页。
2. 选择**部署**下的左侧菜单上**设置**。
3. 查找具有失败状态的部署并选择它。
4. 上**部署**页上，选择**操作详细信息**失败的操作的链接。
5. 请参阅有关在中失败的操作的详细信息**操作的详细信息**窗口。

## <a name="next-steps"></a>后续步骤
请参阅[故障排除的项目失败](devtest-lab-troubleshoot-artifact-failure.md)