---
title: 使用 Azure 数据工厂从 Cassandra 复制数据 | Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 Cassandra 复制到支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: jingwang
ms.openlocfilehash: 743dad6032547f8f535543413adff416efb56ac0
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60640072"
---
# <a name="copy-data-from-cassandra-using-azure-data-factory"></a>使用 Azure 数据工厂从 Cassandra 复制数据
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1](v1/data-factory-onprem-cassandra-connector.md)
> * [当前版本](connector-cassandra.md)

本文概述了如何使用 Azure 数据工厂中的复制活动从 Cassandra 数据库复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 Cassandra 数据库复制到任何支持的接收器数据存储。 有关复制活动支持作为源/接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)表。

具体而言，此 Cassandra 连接器支持：

- Cassandra 2.X 版和 3.x 版。
- 使用基本或匿名身份验证复制数据。

>[!NOTE]
>对于自承载集成运行时上运行的活动，从 IR 版本 3.7 以及更高版本开始，Cassandra 3.x 受支持。

## <a name="prerequisites"></a>必备组件

要从不可公开访问的 Cassandra 数据库复制数据，需要设置自承载集成运行时。 要了解详细信息，请参阅[自承载集成运行时](create-self-hosted-integration-runtime.md)一文。 集成运行时提供内置 Cassandra 驱动程序，因此从/向 Cassandra 复制数据时，无需手动安装任何驱动程序。

## <a name="getting-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 Cassandra 连接器的数据工厂实体，以下部分提供有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Cassandra 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type |type 属性必须设置为：**Cassandra** |是 |
| host |Cassandra 服务器的一个或多个 IP 地址或主机名。<br/>指定以逗号分隔的 IP 地址或主机名列表，以同时连接到所有服务器。 |是 |
| port |Cassandra 服务器用来侦听客户端连接的 TCP 端口。 |否（默认值为 9042） |
| authenticationType | 用于连接 Cassandra 数据库的身份验证类型。<br/>允许值包括：基本和匿名。 |是 |
| username |为用户帐户指定用户名。 |是（如果 authenticationType 设置为 Basic）。 |
| password |指定用户帐户的密码。 将此字段标记为 SecureString 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 |是（如果 authenticationType 设置为 Basic）。 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果可以公开访问数据存储，则可以使用自承载集成运行时或 Azure Integration Runtime 时。 如果未指定，则使用默认 Azure Integration Runtime。 |否 |

>[!NOTE]
>当前不支持使用 SSL 连接到 Cassandra。

**示例：**

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "Cassandra",
        "typeProperties": {
            "host": "<host>",
            "authenticationType": "Basic",
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

有关可用于定义数据集的各部分和属性的完整列表，请参阅数据集一文。 本部分提供 Cassandra 数据集支持的属性列表。

要从 Cassandra 复制数据，请将数据集的 type 属性设置为“CassandraTable”。 支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为：**CassandraTable** | 是 |
| keyspace |Cassandra 数据库中密钥空间或架构的名称。 |否（如果指定了“CassandraSource”的“query”） |
| tableName |Cassandra 数据库中表的名称。 |否（如果指定了“CassandraSource”的“query”） |

**示例：**

```json
{
    "name": "CassandraDataset",
    "properties": {
        "type": "CassandraTable",
        "linkedServiceName": {
            "referenceName": "<Cassandra linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "keySpace": "<keyspace name>",
            "tableName": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性


有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Cassandra 源支持的属性列表。

### <a name="cassandra-as-source"></a>以 Cassandra 作为源

要从 Cassandra 复制数据，请将复制活动中的源类型设置为“CassandraSource”。 复制活动源部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为：**CassandraSource** | 是 |
| query |使用自定义查询读取数据。 SQL-92 查询或 CQL 查询。 请参阅 [CQL reference](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)（CQL 参考）。 <br/><br/>使用 SQL 查询时，请指定 keyspace name.table name 来表示要查询的表。 |否（如果指定了数据集中的“tableName”和“keyspace”）。 |
| consistencyLevel |一致性级别指定在将数据返回到客户端应用程序之前必须响应读取请求的副本的数量。 Cassandra 会检查指定数量的副本，以使数据满足读取请求。 有关详细信息，请参阅 [Configuring data consistency](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html)（配置数据一致性）。<br/><br/>允许值包括：**ONE**、**TWO**、**THREE**、**QUORUM**、**ALL**、**LOCAL_QUORUM**、**EACH_QUORUM** 和 **LOCAL_ONE**。 |否（默认值为 `ONE`） |

**示例：**

```json
"activities":[
    {
        "name": "CopyFromCassandra",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Cassandra input dataset name>",
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
                "type": "CassandraSource",
                "query": "select id, firstname, lastname from mykeyspace.mytable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-cassandra"></a>Cassandra 的数据类型映射

从 Cassandra 复制数据时，以下映射用于从 Cassandra 数据类型映射到 Azure 数据工厂临时数据类型。 若要了解复制活动如何将源架构和数据类型映射到接收器，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。

| Cassandra 数据类型 | 数据工厂临时数据类型 |
|:--- |:--- |
| ASCII |String |
| BIGINT |Int64 |
| BLOB |Byte[] |
| BOOLEAN |Boolean |
| DECIMAL |Decimal |
| DOUBLE |Double |
| FLOAT |Single |
| INET |String |
| INT |Int32 |
| TEXT |String |
| TIMESTAMP |DateTime |
| TIMEUUID |Guid |
| UUID |Guid |
| VARCHAR |String |
| VARINT |Decimal |

> [!NOTE]
> 对于集合类型（映射、集、列表等），请参阅[通过虚拟表使用 Cassandra 集合类型](#work-with-collections-using-virtual-table)部分。
>
> 不支持用户定义的类型。
>
> 二进制列和字符串列的长度不能大于 4000。
>

## <a name="work-with-collections-using-virtual-table"></a>通过虚拟表使用集合

Azure 数据工厂使用内置的 ODBC 驱动程序连接到 Cassandra 数据库，并从其中复制数据。 对于包括映射、集和列表在内的集合类型，该驱动程序会将数据重新规范化到相应的虚拟表中。 具体而言，如果表中包含任何集合列，则该驱动程序会生成以下虚拟表：

* 基表，其中包含与实际表相同的数据（集合列除外）。 基表使用与其所表示的实际表相同的名称。
* 对于每个集合列都会生成一个虚拟表，这会扩展嵌套数据。 使用实际表名称、分隔符“*vt*”和列名称命名表示集合的虚拟表。

虚拟表引用实际表中的数据，以使驱动程序能访问非规范化的数据。 有关详细信息，请参阅示例部分。 通过查询和联接虚拟表，可访问 Cassandra 集合的内容。

### <a name="example"></a>示例

例如，下面的“ExampleTable”是一个 Cassandra 数据库表，其中包含名为“pk_int”的整数主键列、文本列命名值、列表列、映射列和名为“StringSet”的集列。

| pk_int | 值 | 列出 | 映射 | StringSet |
| --- | --- | --- | --- | --- |
| 1 |“示例值 1” |["1", "2", "3"] |{"S1": "a", "S2": "b"} |{"A", "B", "C"} |
| 3 |“示例值 3” |["100", "101", "102", "105"] |{"S1": "t"} |{"A", "E"} |

该驱动程序会生成多个虚拟表来表示此单个表。 虚拟表中的外键列会引用实际表中的主键列，并指示该虚拟表行对应哪一个实际表行。

第一个虚拟表是名为“ExampleTable”的基表，如下表所示： 

| pk_int | 值 |
| --- | --- |
| 1 |“示例值 1” |
| 3 |“示例值 3” |

除了会在此表中被省略但在其他虚拟表中展开的集合外，该基表包含与原始数据库表相同的数据。

下表中显示的虚拟表会重新规范化列表、映射和 StringSet 列中的数据。 名称以“_index”或“_key”结尾的列指示数据在原始列表或映射中的位置。 名称以“_value”结尾的列包含从集合中展开的数据。

**表“ExampleTable_vt_List”：**

| pk_int | List_index | List_value |
| --- | --- | --- |
| 1 |0 |1 |
| 1 |1 |2 |
| 1 |2 |3 |
| 3 |0 |100 |
| 3 |1 |101 |
| 3 |2 |102 |
| 3 |3 |103 |

**表“ExampleTable_vt_Map”：**

| pk_int | Map_key | Map_value |
| --- | --- | --- |
| 1 |S1 |A |
| 1 |S2 |b |
| 3 |S1 |t |

**表“ExampleTable_vt_StringSet”：**

| pk_int | StringSet_value |
| --- | --- |
| 1 |A |
| 1 |B |
| 1 |C |
| 3 |A |
| 3 |E |

## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md##supported-data-stores-and-formats)。
