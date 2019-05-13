---
title: 获取资源更改
description: 了解如何查找资源已更改时，并获取更改的属性的列表。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 04/20/2019
ms.topic: conceptual
ms.service: resource-graph
manager: carmonm
ms.openlocfilehash: 81cb5e62c8abc93aa5defb690628c7178fd2a869
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142542"
---
# <a name="get-resource-changes"></a>获取资源更改

资源获取通过每日使用、 重新配置，以及甚至重新部署的更改。
更改可以来自个人或通过自动化过程。 大多数更改是设计使然，但有时不是。 与过去 14 天的更改历史记录，Azure 资源图形，可以：

- 查找在 Azure 资源管理器属性上检测到更改的时间。
- 了解在更改事件中已更改的属性。

更改检测和详细信息很有价值的以下示例方案：

- 在了解事件管理过程_可能_相关更改。 更改事件的时间在特定时段内查询并评估更改详细信息。
- 保持配置管理数据库，称为 CMDB，保持最新状态。 而不是刷新所有资源和其完整的属性组的计划的频率，只能获取更改的内容。
- 了解哪些其他属性可能已更改时某个资源更改符合性状态。 这些附加属性的计算可以深入了解可能需要管理通过 Azure 策略定义其他属性。

本文介绍如何收集此信息通过资源关系图的 SDK。 若要查看此信息在 Azure 门户中的，请参阅 Azure 策略[更改历史记录](../../policy/how-to/determine-non-compliance.md#change-history-preview)或 Azure 活动日志[更改历史记录](../../../azure-monitor/platform/activity-logs-overview.md#view-change-history)。

> [!NOTE]
> 在资源图表中的更改详细信息适用于资源管理器属性。 有关跟踪虚拟机内部的更改，请参阅 Azure 自动化[更改跟踪](../../../automation/automation-change-tracking.md)或 Azure 策略[Vm 的来宾配置](../../policy/concepts/guest-configuration.md)。

> [!IMPORTANT]
> 在 Azure 资源图表中的更改历史记录处于公共预览状态。

## <a name="find-when-changes-were-detected"></a>查找时检测到更改

查看资源更改的内容的第一步是时间的要查找与该资源的时间段内相关的更改事件。 通过完成此步骤**resourceChanges** REST 终结点。

**ResourceChanges**终结点需要在请求正文中的两个参数：

- **resourceId**:要查找的更改的 Azure 资源。
- **间隔**:具有的属性_启动_并_最终_针对何时检查更改事件使用的日期**Zulu 时区 (Z)**。

示例请求正文：

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "interval": {
        "start": "2019-03-28T00:00:00.000Z",
        "end": "2019-03-31T00:00:00.000Z"
    }
}
```

使用上面的请求正文的 REST API URI **resourceChanges**是：

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChanges?api-version=2018-09-01-preview
```

响应类似于以下示例：

```json
{
    "changes": [{
            "changeId": "2db0ad2d-f6f0-4f46-b529-5c4e8c494648",
            "beforeSnapshot": {
                "timestamp": "2019-03-29T01:32:05.993Z"
            },
            "afterSnapshot": {
                "timestamp": "2019-03-29T01:54:24.42Z"
            }
        },
        {
            "changeId": "9dc352cb-b7c1-4198-9eda-e5e3ed66aec8",
            "beforeSnapshot": {
                "timestamp": "2019-03-28T10:30:19.68Z"
            },
            "afterSnapshot": {
                "timestamp": "2019-03-28T21:12:31.337Z"
            }
        }
    ]
}
```

每个检测到更改事件**resourceId**已**changeId** ，仅适用于该资源。 虽然**changeId**字符串有时可能包含其他属性，它只保证是唯一的。 更改记录都包括时间的之前和之后未拍摄快照。
在此窗口中的某个时间点发生了更改事件。

## <a name="see-what-properties-changed"></a>查看属性的更改

与**changeId**从**resourceChanges**终结点， **resourceChangeDetails** REST 终结点，然后用于获取更改事件的详细信息。

**ResourceChangeDetails**终结点需要在请求正文中的两个参数：

- **resourceId**:要查找的更改的 Azure 资源。
- **changeId**:唯一更改事件**resourceId**从收集**resourceChanges**。

示例请求正文：

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "changeId": "53dc0515-b86b-4bc2-979b-e4694ab4a556"
}
```

使用上面的请求正文的 REST API URI **resourceChangeDetails**是：

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChangeDetails?api-version=2018-09-01-preview
```

响应类似于以下示例：

```json
{
    "changeId": "53dc0515-b86b-4bc2-979b-e4694ab4a556",
    "beforeSnapshot": {
        "timestamp": "2019-03-29T01:32:05.993Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": false,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    },
    "afterSnapshot": {
        "timestamp": "2019-03-29T01:54:24.42Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": true,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    }
}
```

**beforeSnapshot**并**afterSnapshot**每人给拍摄快照时的时间和属性在该时间。 这些快照之间的某些点处发生的更改。 看一下上面的示例中，我们可以看到已更改的属性已**supportsHttpsTrafficOnly**。

若要以编程方式的结果进行比较，比较**内容**的每个快照，以确定的不同部分。 如果将整个快照进行比较**时间戳**始终显示为尽管所预期的差异。

## <a name="next-steps"></a>后续步骤

- 请参阅在中使用的语言[初学者查询](../samples/starter.md)。
- 使用中的高级，请参阅[高级查询](../samples/advanced.md)。
- 了解如何[浏览资源](../concepts/explore-resources.md)。
