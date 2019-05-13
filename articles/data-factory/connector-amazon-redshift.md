---
title: 使用 Azure 数据工厂从 Amazon Redshift 复制数据 |Microsoft Docs
description: 了解如何使用 Azure 数据工厂将数据从 Amazon Redshift 复制到受支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/07/2018
ms.author: jingwang
ms.openlocfilehash: 9e1dde57dc1903e87704bd55fb0b942b7cc349e5
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61261797"
---
# <a name="copy-data-from-amazon-redshift-using-azure-data-factory"></a>使用 Azure 数据工厂从 Amazon Redshift 复制数据
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1](v1/data-factory-amazon-redshift-connector.md)
> * [当前版本](connector-amazon-redshift.md)


本文概述了如何使用 Azure 数据工厂中的复制活动从 Amazon Redshift 复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 Amazon Redshift 复制到任何支持的接收器数据存储。 有关复制活动支持作为源/接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)表。

具体而言，通过使用查询或内置 Redshift UNLOAD 支持，此 Amazon Redshift 连接器支持从 Redshift 检索数据。

> [!TIP]
> 若要在从 Redshift 复制大量数据时获得最佳性能，请考虑通过 Amazon S3 使用内置的 Redshift UNLOAD。 有关详细信息，请参阅[使用 UNLOAD 从Amazon Redshift 复制数据](#use-unload-to-copy-data-from-amazon-redshift)部分。

## <a name="prerequisites"></a>必备组件

* 如果使用[自承载集成运行时](create-self-hosted-integration-runtime.md)将数据复制到本地数据存储，请授权集成运行时（使用计算机的 IP 地址）对 Amazon Redshift 群集的访问权限。 有关说明，请参阅[授予对群集的访问权限](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html)。
* 如果要将数据复制到 Azure 数据存储，请参阅 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)，了解 Azure 数据中心使用的计算 IP 地址和 SQL 范围。

## <a name="getting-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 Amazon Redshift 连接器的数据工厂实体，以下部分提供了有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Amazon Redshift 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | Type 属性必须设置为：**AmazonRedshift** | 是 |
| server |Amazon Redshift 服务器的 IP 地址或主机名。 |是 |
| port |Amazon Redshift 服务器用于侦听客户端连接的 TCP 端口数。 |否，默认值为 5439 |
| database |Amazon Redshift 数据库名称。 |是 |
| username |有权访问数据库的用户的名称。 |是 |
| password |用户帐户密码。 将此字段标记为 SecureString 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果数据存储位于专用网络，则可以使用 Azure Integration Runtime 或自承载集成运行时。 如果未指定，则使用默认 Azure Integration Runtime。 |否 |

**示例：**

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "<server name>",
            "database": "<database name>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅数据集一文。 本部分提供 Amazon Redshift 数据集支持的属性列表。

要从 Amazon Redshift 复制数据，请将数据集的 type 属性设置为“RelationalTable”。 支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为：**RelationalTable** | 是 |
| tableName | Amazon Redshift 中的表的名称。 | 否（如果指定了活动源中的“query”） |

**示例**

```json
{
    "name": "AmazonRedshiftDataset",
    "properties":
    {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Amazon Redshift linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Amazon Redshift 源支持的属性列表。

### <a name="amazon-redshift-as-source"></a>作为源的 Amazon Redshift

要从 Amazon Redshift 复制数据，请将复制活动中的源类型设置为“AmazonRedshiftSource”。 复制活动源部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为：**AmazonRedshiftSource** | 是 |
| query |使用自定义查询读取数据。 例如：从 MyTable 中选择 *。 |否（如果指定了数据集中的“tableName”） |
| redshiftUnloadSettings | 使用 Amazon Redshift UNLOAD 时的属性组。 | 否 |
| s3LinkedServiceName | 表示通过指定“AmazonS3”类型的链接服务名称，将用作临时存储的 Amazon S3。 | 是（如果使用的是 UNLOAD） |
| bucketName | 指示 S3 Bucket 以存储临时数据。 如果未提供，数据工厂服务将自动生成它。  | 是（如果使用的是 UNLOAD） |

**示例：复制活动中使用 UNLOAD 的 Amazon Redshift 源**

```json
"source": {
    "type": "AmazonRedshiftSource",
    "query": "<SQL query>",
    "redshiftUnloadSettings": {
        "s3LinkedServiceName": {
            "referenceName": "<Amazon S3 linked service>",
            "type": "LinkedServiceReference"
        },
        "bucketName": "bucketForUnload"
    }
}
```

在下一节中，深入了解如何使用 UNLOAD 高效地从 Amazon Redshift 复制数据。

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>使用 UNLOAD 从Amazon Redshift 复制数据

[UNLOAD](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) 是 Amazon Redshift 提供的一种机制，可将查询结果卸载到 Amazon 简单存储服务 (Amazon S3) 上的一个或多个文件中。 Amazon 推荐使用此方式从 Redshift 复制大数据集。

**示例：使用 UNLOAD、暂存复制和 PolyBase 将数据从 Amazon Redshift 复制到 Azure SQL 数据仓库**

对于此用例，复制活动按“redshiftUnloadSettings”中的配置将数据从 Amazon Redshift 卸载到 Amazon S3，然后按“stagingSettings”中指定的内容将数据从 Amazon S3 复制到 Azure Blob，最后使用 PolyBase 将数据载入 SQL 数据仓库。 所有临时格式均由复制活动正确处理。

![Redshift 到 SQL DW 复制工作流](media/copy-data-from-amazon-redshift/redshift-to-sql-dw-copy-workflow.png)

```json
"activities":[
    {
        "name": "CopyFromAmazonRedshiftToSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "AmazonRedshiftDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AmazonRedshiftSource",
                "query": "select * from MyTable",
                "redshiftUnloadSettings": {
                    "s3LinkedServiceName": {
                        "referenceName": "AmazonS3LinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "bucketName": "bucketForUnload"
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": "AzureStorageLinkedService",
                "path": "adfstagingcopydata"
            },
            "dataIntegrationUnits": 32
        }
    }
]
```

## <a name="data-type-mapping-for-amazon-redshift"></a>Amazon Redshift 的数据类型映射

从 Amazon Redshift 复制数据时，以下映射用于从 Amazon Redshift 数据类型映射到 Azure 数据工厂临时数据类型。 若要了解复制活动如何将源架构和数据类型映射到接收器，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。

| Amazon Redshift 数据类型 | 数据工厂临时数据类型 |
|:--- |:--- |
| BIGINT |Int64 |
| BOOLEAN |String |
| CHAR |String |
| DATE |DateTime |
| DECIMAL |Decimal |
| DOUBLE PRECISION |Double |
| INTEGER |Int32 |
| REAL |Single |
| SMALLINT |Int16 |
| TEXT |String |
| TIMESTAMP |DateTime |
| VARCHAR |String |

## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md##supported-data-stores-and-formats)。
