---
title: 使用 Azure 数据工厂从 Dynamics AX 中复制数据（预览版）| Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 Dynamics AX 复制到受支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 12/13/2018
ms.author: jingwang
ms.openlocfilehash: 05bd4fdd220b47b11dfed9857dbc8dbe25b236df
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61347773"
---
# <a name="copy-data-from-dynamics-ax-by-using-azure-data-factory-preview"></a>使用 Azure 数据工厂从 Dynamics AX 中复制数据（预览版）

本文概述了如何使用 Azure 数据工厂中的复制活动从 Dynamics AX 源复制数据。 本文是根据总体概述复制活动的 [Azure 数据工厂中的复制活动](copy-activity-overview.md)编写的。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 Dynamics AX 复制到任何受支持的接收器数据存储。 有关复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储和格式](copy-activity-overview.md#supported-data-stores-and-formats)。

具体而言，此 Dynamics AX 连接器支持将 **OData 协议**与**服务主体身份验证**配合使用来从 Dynamics AX 中复制数据。

>[!TIP]
>还可以使用此连接器从 **Dynamics 365 Finance and Operations** 复制数据。 请参阅 Dynamics 365 的 [OData 支持](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/data-entities/odata)和[身份验证方法](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/data-entities/services-home-page#authentication)。

## <a name="get-started"></a>开始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 Dynamics AX 连接器的数据工厂实体，以下部分提供了有关用于定义这些实体的属性的详细信息。

## <a name="prerequisites"></a>必备组件

若要使用服务主体身份验证，请执行以下步骤：

1. 遵循[将应用程序注册到 Azure AD 租户](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant)，在 Azure Active Directory (Azure AD) 中注册一个应用程序实体。 记下下面的值，这些值用于定义链接服务：

    - 应用程序 ID
    - 应用程序密钥
    - 租户 ID

2. 转到 Dynamics AX，向此服务主体授予用于访问 Dynamics AX 的正确权限。

## <a name="linked-service-properties"></a>链接服务属性

Dynamics AX 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | **type** 属性必须设置为 **DynamicsAX**。 |是 |
| url | Dynamics AX（或 Dynamics 365 Finance and Operations）实例 OData 终结点。 |是 |
| servicePrincipalId | 指定应用程序的客户端 ID。 | 是 |
| servicePrincipalKey | 指定应用程序的密钥。 将此字段标记为 **SecureString** 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 | 是 |
| tenant | 指定应用程序的租户信息（域名或租户 ID）。 将鼠标悬停在 Azure 门户右上角进行检索。 | 是 |
| aadResourceId | 指定你请求授权的 AAD 资源。 例如，如果动态 URL 为 `https://sampledynamics.sandbox.operations.dynamics.com/data/`，则相应的 AAD 资源通常为 `https://sampledynamics.sandbox.operations.dynamics.com`。 | 是 |
| connectVia | 用于连接到数据存储的 [ Integration Runtime](concepts-integration-runtime.md)。 可选择 Azure Integration Runtime 或自承载集成运行时（如果数据存储位于专用网络）。 如果未指定，则使用默认 Azure Integration Runtime。 |否 |

**示例**

```json
{
    "name": "DynamicsAXLinkedService",
    "properties": {
        "type": "DynamicsAX",
        "typeProperties": {
            "url": "<Dynamics AX instance OData endpoint>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource, e.g. https://sampledynamics.sandbox.operations.dynamics.com>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}

```

## <a name="dataset-properties"></a>数据集属性

本部分提供了 Dynamics AX 数据集支持的属性列表。

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集和链接服务](concepts-datasets-linked-services.md)。 

要从 Dynamics AX 复制数据，请将数据集的 **type** 属性设置为 **DynamicsAXResource**。 支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 **type** 属性必须设置为 **DynamicsAXResource**。 | 是 |
| path | Dynamics AX OData 实体的路径。 | 是 |

**示例**

```json
{
    "name": "DynamicsAXResourceDataset",
    "properties": {
        "type": "DynamicsAXResource",
        "typeProperties": {
            "path": "<entity path e.g. dd04tentitySet>"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics AX linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

本部分提供了 Dynamics AX 源支持的属性列表。

有关可用于定义活动的各个部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)。 

### <a name="dynamics-ax-as-source"></a>作为源的 Dynamics AX

若要从 Dynamics AX 复制数据，请将复制活动中的 **source** 类型设置为 **DynamicsAXSource**。 复制活动 **source** 节支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 **type** 属性必须设置为 **DynamicsAXSource**。 | 是 |
| query | 用于筛选数据的 OData 查询选项。 示例：`"?$select=Name,Description&$top=5"`。<br/><br/>**注意**：连接器会从以下组合 URL 复制数据：`[URL specified in linked service]/[path specified in dataset][query specified in copy activity source]`。 有关详细信息，请参阅 [OData URL 组件](https://www.odata.org/documentation/odata-version-3-0/url-conventions/)。 | 否 |

**示例**

```json
"activities":[
    {
        "name": "CopyFromDynamicsAX",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics AX input dataset name>",
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
                "type": "DynamicsAXSource",
                "query": "$top=10"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>后续步骤

有关 Azure 数据工厂中复制活动支持用作源和接收器的数据存储的列表，请参阅[支持的数据存储和格式](copy-activity-overview.md##supported-data-stores-and-formats)。
