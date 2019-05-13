---
title: 从 Office 365 使用 Azure 数据工厂复制数据 |Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 Office 365 复制到支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: jingwang
ms.openlocfilehash: 9ca3cbb1ef46c7fe53b6b16bda40ebef245613f3
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415660"
---
# <a name="copy-data-from-office-365-into-azure-using-azure-data-factory"></a>将数据从 Office 365 复制到 Azure 中使用 Azure 数据工厂

Azure 数据工厂允许你以可缩放的方式将 Office 365 租户中的丰富组织数据引入 Azure，并生成分析应用程序和基于这些有价值的数据资产提取见解。 与 Privileged Access Management 的集成为 Office 365 中组织有序的有价值的数据提供安全访问控制。  有关 Microsoft Graph 数据连接的详细信息，请参阅[此链接](https://docs.microsoft.com/graph/data-connect-concept-overview)。

本文概述了如何使用 Azure 数据工厂中的复制活动从 Office 365 复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

现在，在单次复制活动可以只**将数据复制到 Office 365 [Azure Blob 存储](connector-azure-blob-storage.md)， [Azure 数据湖存储 Gen1](connector-azure-data-lake-store.md)，并[Azure 数据湖存储第 2 代](connector-azure-data-lake-storage.md)采用 JSON 格式**（键入 setOfObjects）。 如果要将 Office 365 加载到其他类型的或其他格式的数据存储，可以将第一个副本活动与后续复制活动链接在一起，以进一步将数据加载到任何[支持的 ADF 目标存储](copy-activity-overview.md#supported-data-stores-and-formats)（请参阅“支持的数据存储和格式”表中的“作为接收器支持”列）。

>[!IMPORTANT]
>- 包含数据工厂和接收器数据存储的 Azure 订阅必须位于与 Office 365 租户相同的 Azure Active Directory (Azure AD) 租户下。
>- 确保用于复制活动的 Azure Integration Runtime 区域以及目标在 Office 365 租户用户邮箱所在的同一区域中。 若要了解如何确定 Azure IR 位置，请参阅[此处](concepts-integration-runtime.md#integration-runtime-location)。 有关受支持的 Office 区域和对应的 Azure 区域列表，请参阅[此处的表](https://docs.microsoft.com/graph/data-connect-datasets#regions)。
>- 服务主体身份验证是 Azure Blob 存储、 Azure 数据湖存储 Gen1 和 Azure 数据湖存储第 2 代作为目标存储区支持的唯一身份验证机制。

## <a name="prerequisites"></a>必备组件

若要将数据从 Office 365 复制到 Azure，需要完成下列必备步骤：

- Office 365 租户管理员必须完成载入操作，如[此处](https://docs.microsoft.com/graph/data-connect-get-started)所述。
- 在 Azure Active Directory 中创建和配置 Azure AD Web 应用程序。  有关说明，请参阅[创建 Azure AD 应用程序](../active-directory/develop/howto-create-service-principal-portal.md#create-an-azure-active-directory-application)。
- 记下下面的值，这些值将用于定义 Office 365 的链接服务：
    - 租户 ID。 有关说明，请参阅[获取租户 ID](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-id)。
    - 应用程序 ID 和身份验证密钥。  有关说明，请参阅[获取应用程序 ID 和身份验证密钥](../active-directory/develop/howto-create-service-principal-portal.md#get-application-id-and-authentication-key)。
- 添加用户标识，将作为 Azure AD Web 应用程序的所有者发出数据访问请求（从 Azure AD Web 应用程序>设置>所有者>添加所有者）。 
    - 用户标识必须位于你从中获取数据的 Office 365 组织中，并且不能是来宾用户。

## <a name="approving-new-data-access-requests"></a>批准新的数据访问请求

如果这是你首次请求此上下文（要访问的数据表、要将数据加载到的目标帐户和发出数据访问请求的用户标识的组合）的数据，则复制活动状态将显示为“正在进行”；仅当单击[“操作”下的“详细信息”](copy-activity-overview.md#monitoring)链接时，状态才显示为“正在请求许可”。  在继续执行数据提取之前，数据访问审批者组的成员需要在 Privileged Access Management 中审批该请求。

有关审批者如何批准数据访问请求的信息，请参阅[此处](https://docs.microsoft.com/graph/data-connect-tips#approve-pam-requests-via-office-365-admin-portal)，以及有关与 Privileged Access Management 的全面集成（包括如何设置数据访问审批者组）的说明，请参阅[此处](https://docs.microsoft.com/graph/data-connect-pam)。

## <a name="policy-validation"></a>策略验证

如果 ADF 作为托管应用程序的一部分创建，并且 Azure 策略分配在管理资源组的资源上进行，那么对于运行的每个副本活动，ADF 都将检查以确保强制实施策略分配。 有关受支持的策略列表，请参阅[此处](https://docs.microsoft.com/graph/data-connect-policies#policies)。

## <a name="getting-started"></a>入门

>[!TIP]
>有关使用 Office 365 连接器的演练，请参阅[从 Office 365 加载数据](load-office-365-data.md)一文。

可以使用以下工具或 SDK 之一创建包含复制活动的管道。 选择链接导航到相关教程，其中涵盖有关创建包含复制活动的管道的分步说明。 

- [Azure 门户](quickstart-create-data-factory-portal.md)
- [.NET SDK](quickstart-create-data-factory-dot-net.md)
- [Python SDK](quickstart-create-data-factory-python.md)
- [Azure PowerShell](quickstart-create-data-factory-powershell.md)
- [REST API](quickstart-create-data-factory-rest-api.md)
- [Azure 资源管理器模板](quickstart-create-data-factory-resource-manager-template.md)。 

对于特定于 Office 365 连接器的数据工厂实体，以下部分提供有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Office 365 链接服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | type 属性必须设置为：**Office365** | “是” |
| office365TenantId | Office 365 帐户所属的 Azure 租户 ID。 | “是” |
| servicePrincipalTenantId | 指定 Azure AD Web 应用程序所在的租户信息。 | “是” |
| servicePrincipalId | 指定应用程序的客户端 ID。 | 是 |
| servicePrincipalKey | 指定应用程序的密钥。 将此字段标记为 SecureString，以便安全地将其存储在数据工厂中。 | “是” |
| connectVia | 用于连接到数据存储的 Integration Runtime。  如果未指定，则使用默认 Azure Integration Runtime。 | “否” |

>[!NOTE]
> office365TenantId 和 servicePrincipalTenantId 之间的差异和提供的相应值：
>- 如果你是一名企业开发人员，开发便于自己组织使用的针对 Office 365 数据的应用程序，则应该为这两个属性提供相同的租户 ID，即你的组织 AAD 租户 ID。
>- 如果你是为客户开发应用程序的 ISV 开发人员，那么 office365TenantId 将是客户的（应用程序安装程序）AAD 租户 ID，servicePrincipalTenantId 则为公司的 AAD 租户 ID。

**示例：**

```json
{
    "name": "Office365LinkedService",
    "properties": {
        "type": "Office365",
        "typeProperties": {
            "office365TenantId": "<Office 365 tenant id>",
            "servicePrincipalTenantId": "<AAD app service principal tenant id>",
            "servicePrincipalId": "<AAD app service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<AAD app service principal key>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供 Office 365 数据集支持的属性列表。

若要从 Office 365 复制数据，支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为：**Office365Table** | “是” |
| tableName | 要从 Office 365 中提取的数据集的名称。 有关支持提取的 Office 365 数据集列表，请参阅[此处](https://docs.microsoft.com/graph/data-connect-datasets#datasets)。 | “是” |
| allowedGroups | 组选择谓词。  使用此属性以选择将为其检索数据的最多 10 个用户组。  如果未不指定任何组，则将为整个组织返回的数据。 | “否” |
| userScopeFilterUri | 当`allowedGroups`属性未指定，则可以使用应用于整个租户来筛选要从 Office 365 中提取的特定行的谓词表达式。 谓词的格式应与 Microsoft Graph Api，则查询格式例如匹配`https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'`。 | “否” |
| dateFilterColumn | 日期时间筛选器列的名称。 使用此属性来限制哪些 Office 365 的提取数据的时间范围。 | Yes，如果数据集有一个或多个日期时间列。 请参阅[此处](https://docs.microsoft.com/graph/data-connect-filtering#filtering)有关的数据集需要此日期时间筛选器的列表。 |
| startTime | 启动要作为筛选依据的日期时间值。 | 是如果`dateFilterColumn`指定 |
| endTime | 结束日期时间值进行筛选。 | 是如果`dateFilterColumn`指定 |

**示例**

```json
{
    "name": "DS_May2019_O365_Message",
    "properties": {
        "type": "Office365Table",
        "linkedServiceName": {
            "referenceName": "<Office 365 linked service name>",
            "type": "LinkedServiceReference"
        },
        "structure": [
            {
                "name": "Id",
                "type": "String",
                "description": "The unique identifier of the event."
            },
            {
                "name": "CreatedDateTime",
                "type": "DateTime",
                "description": "The date and time that the event was created."
            },
            {
                "name": "LastModifiedDateTime",
                "type": "DateTime",
                "description": "The date and time that the event was last modified."
            },
            {
                "name": "ChangeKey",
                "type": "String",
                "description": "Identifies the version of the event object. Every time the event is changed, ChangeKey changes as well. This allows Exchange to apply changes to the correct version of the object."
            },
            {
                "name": "Categories",
                "type": "String",
                "description": "The categories associated with the event. Format: ARRAY<STRING>"
            },
            {
                "name": "OriginalStartTimeZone",
                "type": "String",
                "description": "The start time zone that was set when the event was created. See DateTimeTimeZone for a list of valid time zones."
            },
            {
                "name": "OriginalEndTimeZone",
                "type": "String",
                "description": "The end time zone that was set when the event was created. See DateTimeTimeZone for a list of valid time zones."
            },
            {
                "name": "ResponseStatus",
                "type": "String",
                "description": "Indicates the type of response sent in response to an event message. Format: STRUCT<Response: STRING, Time: STRING>"
            },
            {
                "name": "iCalUId",
                "type": "String",
                "description": "A unique identifier that is shared by all instances of an event across different calendars."
            },
            {
                "name": "ReminderMinutesBeforeStart",
                "type": "Int32",
                "description": "The number of minutes before the event start time that the reminder alert occurs."
            },
            {
                "name": "IsReminderOn",
                "type": "Boolean",
                "description": "Set to true if an alert is set to remind the user of the event."
            },
            {
                "name": "HasAttachments",
                "type": "Boolean",
                "description": "Set to true if the event has attachments."
            },
            {
                "name": "Subject",
                "type": "String",
                "description": "The text of the event's subject line."
            },
            {
                "name": "Body",
                "type": "String",
                "description": "The body of the message associated with the event.Format: STRUCT<ContentType: STRING, Content: STRING>"
            },
            {
                "name": "Importance",
                "type": "String",
                "description": "The importance of the event: Low, Normal, High."
            },
            {
                "name": "Sensitivity",
                "type": "String",
                "description": "Indicates the level of privacy for the event: Normal, Personal, Private, Confidential."
            },
            {
                "name": "Start",
                "type": "String",
                "description": "The start time of the event. Format: STRUCT<DateTime: STRING, TimeZone: STRING>"
            },
            {
                "name": "End",
                "type": "String",
                "description": "The date and time that the event ends. Format: STRUCT<DateTime: STRING, TimeZone: STRING>"
            },
            {
                "name": "Location",
                "type": "String",
                "description": "Location information of the event. Format: STRUCT<DisplayName: STRING, Address: STRUCT<Street: STRING, City: STRING, State: STRING, CountryOrRegion: STRING, PostalCode: STRING>, Coordinates: STRUCT<Altitude: DOUBLE, Latitude: DOUBLE, Longitude: DOUBLE, Accuracy: DOUBLE, AltitudeAccuracy: DOUBLE>>"
            },
            {
                "name": "IsAllDay",
                "type": "Boolean",
                "description": "Set to true if the event lasts all day. Adjusting this property requires adjusting the Start and End properties of the event as well."
            },
            {
                "name": "IsCancelled",
                "type": "Boolean",
                "description": "Set to true if the event has been canceled."
            },
            {
                "name": "IsOrganizer",
                "type": "Boolean",
                "description": "Set to true if the message sender is also the organizer."
            },
            {
                "name": "Recurrence",
                "type": "String",
                "description": "The recurrence pattern for the event. Format: STRUCT<Pattern: STRUCT<Type: STRING, `Interval`: INT, Month: INT, DayOfMonth: INT, DaysOfWeek: ARRAY<STRING>, FirstDayOfWeek: STRING, Index: STRING>, `Range`: STRUCT<Type: STRING, StartDate: STRING, EndDate: STRING, RecurrenceTimeZone: STRING, NumberOfOccurrences: INT>>"
            },
            {
                "name": "ResponseRequested",
                "type": "Boolean",
                "description": "Set to true if the sender would like a response when the event is accepted or declined."
            },
            {
                "name": "ShowAs",
                "type": "String",
                "description": "The status to show: Free, Tentative, Busy, Oof, WorkingElsewhere, Unknown."
            },
            {
                "name": "Type",
                "type": "String",
                "description": "The event type: SingleInstance, Occurrence, Exception, SeriesMaster."
            },
            {
                "name": "Attendees",
                "type": "String",
                "description": "The collection of attendees for the event. Format: ARRAY<STRUCT<EmailAddress: STRUCT<Name: STRING, Address: STRING>, Status: STRUCT<Response: STRING, Time: STRING>, Type: STRING>>"
            },
            {
                "name": "Organizer",
                "type": "String",
                "description": "The organizer of the event. Format: STRUCT<EmailAddress: STRUCT<Name: STRING, Address: STRING>>"
            },
            {
                "name": "WebLink",
                "type": "String",
                "description": "The URL to open the event in Outlook Web App."
            },
            {
                "name": "Attachments",
                "type": "String",
                "description": "The FileAttachment and ItemAttachment attachments for the message. Navigation property. Format: ARRAY<STRUCT<LastModifiedDateTime: STRING, Name: STRING, ContentType: STRING, Size: INT, IsInline: BOOLEAN, Id: STRING>>"
            },
            {
                "name": "BodyPreview",
                "type": "String",
                "description": "The preview of the message associated with the event. It is in text format."
            },
            {
                "name": "Locations",
                "type": "String",
                "description": "The locations where the event is held or attended from. The location and locations properties always correspond with each other. Format:  ARRAY<STRUCT<DisplayName: STRING, Address: STRUCT<Street: STRING, City: STRING, State: STRING, CountryOrRegion: STRING, PostalCode: STRING>, Coordinates: STRUCT<Altitude: DOUBLE, Latitude: DOUBLE, Longitude: DOUBLE, Accuracy: DOUBLE, AltitudeAccuracy: DOUBLE>, LocationEmailAddress: STRING, LocationUri: STRING, LocationType: STRING, UniqueId: STRING, UniqueIdType: STRING>>"
            },
            {
                "name": "OnlineMeetingUrl",
                "type": "String",
                "description": "A URL for an online meeting. The property is set only when an organizer specifies an event as an online meeting such as a Skype meeting"
            },
            {
                "name": "OriginalStart",
                "type": "DateTime",
                "description": "The start time that was set when the event was created in UTC time."
            },
            {
                "name": "SeriesMasterId",
                "type": "String",
                "description": "The ID for the recurring series master item, if this event is part of a recurring series."
            }
        ],
        "typeProperties": {
            "tableName": "BasicDataSet_v0.Event_v1",
            "dateFilterColumn": "CreatedDateTime",
            "startTime": "2019-04-28T16:00:00.000Z",
            "endTime": "2019-05-05T16:00:00.000Z",
            "userScopeFilterUri": "https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Office 365 源支持的属性列表。

### <a name="office-365-as-source"></a>Office 365 即源

要从 Office 365 复制数据，请将复制活动中的源类型设置为“Office365Source”。 复制活动源部分不支持其他属性。

**示例：**

```json
"activities": [
    {
        "name": "CopyFromO365ToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Office 365 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "Office365Source"
            },
            "sink": {
                "type": "BlobSink"
            }
        }
    }
]
```

## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。
