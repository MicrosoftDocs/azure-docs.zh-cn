---
title: 接收有关 Azure 服务通知的活动日志警报
description: 在 Azure 服务发生时，通过短信、电子邮件或 webhook 接收通知。
author: shawntabrizi
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 06/09/2018
ms.author: shtabriz
ms.openlocfilehash: 3b3c967cd43745a4ae87fefc578282f5427a5f79
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65405702"
---
# <a name="create-activity-log-alerts-on-service-notifications"></a>创建有关服务通知的活动日志警报
## <a name="overview"></a>概述
本文演示如何使用 Azure 门户设置活动日志警报，用于通知服务运行状况。  

当 Azure 将服务运行状况通知发送到 Azure 订阅时，你可以收到警报。 可以基于以下内容配置警报：

- 服务运行状况通知的类别（服务问题、计划内维护、运行状况公告）。
- 受影响的订阅。
- 受影响的服务。
- 受影响的区域。

> [!NOTE]
> 服务运行状况通知不会发送有关资源运行状况事件的警报。

还可以配置向其发送警报的人员：

- 选择现有操作组。
- 创建新操作组（可以用于将来的警报）。

若要了解有关操作组的详细信息，请参阅[创建和管理操作组](../../azure-monitor/platform/action-groups.md)。

有关如何使用 Azure 资源管理器模板配置服务运行状况通知警报的信息，请参阅[资源管理器模板](alerts-activity-log.md)。

### <a name="watch-a-video-on-setting-up-your-first-azure-service-health-alert"></a>观看视频设置第一个 Azure 服务运行状况警报

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OaXt]

## <a name="alert-and-new-action-group-using-azure-portal"></a>使用 Azure 门户发出警报和新建操作组
1. 在[门户](https://portal.azure.com)中，选择“服务运行状况”。

    ![“服务运行状况”服务](media/alerts-activity-log-service-notifications/home-servicehealth.png)

1. 在“警报”部分中，选择“运行状况警报”。

    ![“运行状况警报”选项卡](media/alerts-activity-log-service-notifications/alerts-blades-sh.png)

1. 选择“创建服务运行状况警报”，并填写字段。

    ![“创建服务运行状况警报”命令](media/alerts-activity-log-service-notifications/service-health-alert.png)

1. 选择要针对其发出警报的**订阅**、**服务**和**区域**。

    ![“添加活动日志警报”对话框](media/alerts-activity-log-service-notifications/activity-log-alert-new-ux.png)

> [!NOTE]
> 此订阅用于保存活动日志警报。 警报资源部署到此订阅，并在其中监视活动日志事件。

1. 选择要针对其发出警报的**事件类型**：“服务问题”、“计划内维护”和“运行状况公告” 

1. 通过输入**警报规则名称**和**说明**定义警报详细信息。

1. 选择要将警报保存到的**资源组**。

1. 通过选择“新建操作组”创建一个新操作组。 在“操作组名称”框中输入名称，然后在“短名称”框中输入名称。 在触发此警报时，将引用已发送通知中的短名称。

    ![创建新的操作组](media/alerts-activity-log-service-notifications/action-group-creation.png)

1. 通过提供接收方来定义接收方的列表：

    a. **名称**：输入接收方的名称、别名或标识符。

    b. **操作类型**：选择短信、电子邮件、Webhook、Azure 应用等。

    c. **详细信息**：根据所选操作类型，输入电话号码、电子邮件地址、Webhook URI 等。

1. 选择“确定”以创建操作组，然后选择“创建警报规则”以完成警报。

在几分钟内，警报将处于活动状态，并根据创建期间指定的条件开始触发。

了解如何[为现有问题管理系统配置 Webhook 通知](../../service-health/service-health-alert-webhook-guide.md)。 有关活动日志警报的 webhook 架构的信息，请参阅 [Azure 活动日志警报的 Webhook](../../azure-monitor/platform/activity-log-alerts-webhook.md)。

>[!NOTE]
>这些步骤中定义的操作组可以作为现有操作组重复用于所有未来的警报定义。
>
>

## <a name="alert-with-existing-action-group-using-azure-portal"></a>使用 Azure 门户通过现有操作组发出警报

1. 执行上一节中的步骤 1 至 7 来创建服务运行状况通知。 

1. 在“定义操作组”下，单击“选择操作组”按钮。 选择适当的操作组。

1. 选择“添加”以添加操作组，然后选择“创建警报规则”以完成警报。

在几分钟内，警报将处于活动状态，并根据创建期间指定的条件开始触发。

## <a name="alert-and-new-action-group-using-the-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板发出警报和新建操作组

下面是创建以电子邮件为目标的操作组，并为目标订阅启用所有服务运行状况通知的示例。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "actionGroups_name": {
            "defaultValue": "SubHealth",
            "type": "String"
        },
        "activityLogAlerts_name": {
            "defaultValue": "ServiceHealthActivityLogAlert",
            "type": "String"
        },
        "emailAddress":{
            "type":"string"
        }
    },
    "variables": {
        "alertScope":"[concat('/','subscriptions','/',subscription().subscriptionId)]"
    },
    "resources": [
        {
            "comments": "Action Group",
            "type": "microsoft.insights/actionGroups",
            "name": "[parameters('actionGroups_name')]",
            "apiVersion": "2017-04-01",
            "location": "Global",
            "tags": {},
            "scale": null,
            "properties": {
                "groupShortName": "[parameters('actionGroups_name')]",
                "enabled": true,
                "emailReceivers": [
                    {
                        "name": "[parameters('actionGroups_name')]",
                        "emailAddress": "[parameters('emailAddress')]"
                    }
                ],
                "smsReceivers": [],
                "webhookReceivers": []
            },
            "dependsOn": []
        },
        {
            "comments": "Service Health Activity Log Alert",
            "type": "microsoft.insights/activityLogAlerts",
            "name": "[parameters('activityLogAlerts_name')]",
            "apiVersion": "2017-04-01",
            "location": "Global",
            "tags": {},
            "scale": null,
            "properties": {
                "scopes": [
                    "[variables('alertScope')]"
                ],
                "condition": {
                    "allOf": [
                        {
                            "field": "category",
                            "equals": "ServiceHealth"
                        },
                        {
                            "field": "properties.incidentType",
                            "equals": "Incident"
                        }
                    ]
                },
                "actions": {
                    "actionGroups": [
                        {
                            "actionGroupId": "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]",
                            "webhookProperties": {}
                        }
                    ]
                },
                "enabled": true,
                "description": ""
            },
            "dependsOn": [
                "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
            ]
        }
    ]
}
```

## <a name="manage-your-alerts"></a>管理警报

创建警报之后，可在“监视”的“警报”部分中查看。 选择要管理的警报：

* 编辑它。
* 删除它。
* 如果要暂时停止或恢复接收警报的通知，可“禁用”或“启用”它。

## <a name="next-steps"></a>后续步骤
- 了解如何[的最佳实践设置 Azure 服务运行状况警报](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa)。
- 了解如何[安装程序的 Azure 服务运行状况的移动推送通知](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw)。
- 了解如何[为现有问题管理系统配置 Webhook 通知](../../service-health/service-health-alert-webhook-guide.md)。
- 了解[服务运行状况通知](../../azure-monitor/platform/service-notifications.md)。
- 了解[通知速率限制](../../azure-monitor/platform/alerts-rate-limiting.md)。
- 查看[活动日志警报 webhook 架构](../../azure-monitor/platform/activity-log-alerts-webhook.md)。
- 获取[活动日志警报概述](../../azure-monitor/platform/alerts-overview.md)，了解如何接收警报。 
- 详细了解[操作组](../../azure-monitor/platform/action-groups.md)。
