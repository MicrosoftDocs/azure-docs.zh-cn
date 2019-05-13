---
title: 使用 Azure 数据工厂从 HTTP 源复制数据 | Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从云或本地 HTTP 源复制到支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 04/29/2019
ms.author: jingwang
ms.openlocfilehash: a668bb2e0e3381abefaac93a0fb63f0d33bac5a1
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234063"
---
# <a name="copy-data-from-an-http-endpoint-by-using-azure-data-factory"></a>使用 Azure 数据工厂从 HTTP 终结点复制数据

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1](v1/data-factory-http-connector.md)
> * [当前版本](connector-http.md)

本文概述了如何使用 Azure 数据工厂中的复制活动从 HTTP 终结点复制数据。 本文是根据总体概述复制活动的 [Azure 数据工厂中的复制活动](copy-activity-overview.md)编写的。

此 HTTP 连接器、[REST 连接器](connector-rest.md)和 [Web 表连接器](connector-web-table.md)之间的区别如下：

- **REST 连接器**专门支持从 RESTful API 复制数据； 
- **HTTP 连接器**是通用的，用于从任何 HTTP 终结点检索数据，例如下载文件。 在 REST 连接器可用之前，你可能偶然使用 HTTP 连接器从 RESTful API 复制数据，这是受支持的，但与 REST 连接器相比功能较少。
- **Web 表连接器**用于从 HTML 网页中提取表内容。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 HTTP 源复制到任何支持的接收器数据存储。 有关复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储和格式](copy-activity-overview.md#supported-data-stores-and-formats)。

可以使用此 HTTP 连接器：

- 通过 HTTP GET 或 POST 方法，从 HTTP/S 终结点检索数据。
- 使用以下身份验证之一检索数据：**Anonymous**、**Basic**、**Digest**、**Windows** 或 **ClientCertificate**。
- 按原样复制 HTTP 响应，或者使用[支持的文件格式和压缩编解码器](supported-file-formats-and-compression-codecs.md)分析该响应。

> [!TIP]
> 若要先为数据检索测试 HTTP 请求，再在数据工厂中配置 HTTP 连接器，请了解标头和正文的 API 规范要求。 可以使用 Postman 或 Web 浏览器等工具进行验证。

## <a name="get-started"></a>开始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 HTTP 连接器的数据工厂实体，以下部分提供有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

HTTP 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| 类型 | type 属性必须设置为 HttpServer。 | “是” |
| url | Web 服务器的基 URL。 | “是” |
| enableServerCertificateValidation | 指定连接到 HTTP 终结点时，是否启用服务器 SSL 证书验证。 HTTPS 服务器使用自签名证书时，将此属性设置为 false。 | “否”<br /> （默认值为 true） |
| authenticationType | 指定身份验证类型。 允许的值为：Anonymous、Basic、Digest、Windows 和 ClientCertificate。 <br><br> 有关这些身份验证类型的更多属性和 JSON 示例，请参阅此表格下面的部分。 | “是” |
| connectVia | 用于连接到数据存储的 [ Integration Runtime](concepts-integration-runtime.md)。 可使用 Azure Integration Runtime 或自承载集成运行时（如果数据存储位于专用网络）。 如果未指定，则此属性使用默认 Azure Integration Runtime。 |“否” |

### <a name="using-basic-digest-or-windows-authentication"></a>使用基本、摘要或 Windows 身份验证

将 authenticationType 属性设置为 Basic、Digest 或 Windows。 除了前面部分所述的通用属性，还指定以下属性：

| 属性 | 说明 | 需要 |
|:--- |:--- |:--- |
| userName | 用于访问 HTTP 终结点的用户名。 | “是” |
| password | 用户（userName 值）的密码。 将此字段标记为 SecureString 类型，以便安全地将其存储在数据工厂中。 此外，还可以[引用 Azure Key Vault 中存储的机密](store-credentials-in-key-vault.md)。 | “是” |

**示例**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "Basic",
            "url" : "<HTTP endpoint>",
            "userName": "<user name>",
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

### <a name="using-clientcertificate-authentication"></a>使用 ClientCertificate 身份验证

若要使用 ClientCertificate 身份验证，将 authenticationType 属性设置为ClientCertificate。 除了前面部分所述的通用属性，还指定以下属性：

| 属性 | 说明 | 需要 |
|:--- |:--- |:--- |
| embeddedCertData | Base64 编码的证书数据。 | 指定是 embeddedCertData，还是 certThumbprint。 |
| certThumbprint | 自承载集成运行时计算机的证书存储中所安装证书的指纹。 仅当在 connectVia 属性中指定自承载类型的 Integration Runtime 时适用。 | 指定是 embeddedCertData，还是 certThumbprint。 |
| password | 与证书关联的密码。 将此字段标记为 SecureString 类型，以便安全地将其存储在数据工厂中。 此外，还可以[引用 Azure Key Vault 中存储的机密](store-credentials-in-key-vault.md)。 | “否” |

如果使用 certThumbprint 进行身份验证，并在本地计算机的个人存储中安装了证书，则需要授予对自承载集成运行时的读取权限：

1. 打开 Microsoft 管理控制台 (MMC)。 添加面向“本地计算机”的“证书”管理单元。
2. 展开“证书” > “个人”，然后选择“证书”。
3. 右键单击个人存储中的证书，并选择“所有任务” > “管理私钥”。
3. 在“安全性”选项卡上，添加运行 Integration Runtime 主机服务 (DIAHostService) 的、对证书具有读取访问权限的用户帐户。

**示例 1：使用 certThumbprint**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "certThumbprint": "<thumbprint of certificate>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**示例 2：使用 embeddedCertData**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "embeddedCertData": "<Base64-encoded cert data>",
            "password": {
                "type": "SecureString",
                "value": "password of cert"
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

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 

- 有关**Parquet 和带分隔符的文本格式**，请参阅[Parquet 和带分隔符的文本格式的数据集](#parquet-and-delimited-text-format-dataset)部分。
- 其他格式，例如**ORC/Avro/JSON/二进制格式**，请参阅[其他格式数据集](#other-format-dataset)部分。

### <a name="parquet-and-delimited-text-format-dataset"></a>Parquet 和带分隔符的文本格式的数据集

要从 HTTP 复制数据**Parquet 或带分隔符的文本格式**，请参阅[Parquet 格式](format-parquet.md)并[分隔符的文本格式](format-delimited-text.md)基于格式的数据集上的文章和支持设置。 在下的 http 支持以下属性`location`中基于格式的数据集的设置：

| 属性    | 说明                                                  | 必选 |
| ----------- | ------------------------------------------------------------ | -------- |
| 类型        | 下面的类型属性`location`在数据集中必须设置为**HttpServerLocation**。 | “是”      |
| relativeUrl | 包含数据的资源的相对 URL。       | “否”       |

> [!NOTE]
> 支持的 HTTP 请求有效负载大小约为 500 KB。 如果要传递给 Web 终结点的有效负载大小大于 500 KB，请考虑以更小的区块对该有效负载进行批处理。

> [!NOTE]
> **HttpFile**类型与下一节中所述的 Parquet/文本格式的数据集作为仍受支持的用于复制/查找活动的向后兼容性。 建议以使用从长远看，此新模型和创作 UI 的 ADF 已切换为生成这些新类型。

**示例：**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "HttpServerLocation",
                "relativeUrl": "<relative url>"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

### <a name="other-format-dataset"></a>其他格式数据集

若要将数据从 HTTP 中复制**ORC/Avro/JSON/二进制格式**，支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| 类型 | 数据集的 type 属性必须设置为 HttpFile。 | “是” |
| relativeUrl | 包含数据的资源的相对 URL。 未指定此属性时，仅使用链接服务定义中指定的 URL。 | “否” |
| requestMethod | HTTP 方法。 允许的值为 Get（默认值）和 Post。 | “否” |
| additionalHeaders | 附加的 HTTP 请求标头。 | 否 |
| requestBody | HTTP 请求的正文。 | “否” |
| format | 如果要在未经分析的情况下从 HTTP 终结点按原样检索数据，并将其复制到基于文件的存储，请跳过输入和输出数据集定义中的格式部分。<br/><br/>如果要在复制期间分析 HTTP 响应内容，则支持以下文件格式类型：TextFormat、JsonFormat、AvroFormat、OrcFormat 和 ParquetFormat。 请将格式中的“type”属性设置为上述值之一。 有关详细信息，请参阅 [JSON 格式](supported-file-formats-and-compression-codecs.md#json-format)、[文本格式](supported-file-formats-and-compression-codecs.md#text-format)、[Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)。 |“否” |
| compression | 指定数据的压缩类型和级别。 有关详细信息，请参阅[受支持的文件格式和压缩编解码器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/><br/>支持的类型：**GZip**、**Deflate**、**BZip2** 和 **ZipDeflate**。<br/>支持的级别：“最佳”和“最快”。 |“否” |

> [!NOTE]
> 支持的 HTTP 请求有效负载大小约为 500 KB。 如果要传递给 Web 终结点的有效负载大小大于 500 KB，请考虑以更小的区块对该有效负载进行批处理。

**示例 1：使用 Get 方法（默认）**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        }
    }
}
```

**示例 2：使用 Post 方法**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "requestMethod": "Post",
            "requestBody": "<body for POST HTTP request>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

本部分提供 HTTP 源支持的属性列表。

有关可用于定义活动的各个部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)。 

### <a name="http-as-source"></a>HTTP 作为源

- 从副本**Parquet 和带分隔符的文本格式**，请参阅[Parquet 和带分隔符的文本格式源](#parquet-and-delimited-text-format-source)部分。
- 从等其他格式的副本**ORC/Avro/JSON/二进制格式**，请参阅[其他格式源](#other-format-source)部分。

#### <a name="parquet-and-delimited-text-format-source"></a>Parquet 和带分隔符的文本格式源

要从 HTTP 复制数据**Parquet 或带分隔符的文本格式**，请参阅[Parquet 格式](format-parquet.md)并[分隔符的文本格式](format-delimited-text.md)上基于格式的复制活动源的文章和支持的设置。 在下的 http 支持以下属性`storeSettings`基于格式的复制源中的设置：

| 属性                 | 说明                                                  | 必选 |
| ------------------------ | ------------------------------------------------------------ | -------- |
| 类型                     | 下面的类型属性`storeSettings`必须设置为**HttpReadSetting**。 | “是”      |
| requestMethod            | HTTP 方法。 <br>允许的值为 Get（默认值）和 Post。 | “否”       |
| addtionalHeaders         | 附加的 HTTP 请求标头。                             | 否       |
| requestBody              | HTTP 请求的正文。                               | “否”       |
| requestTimeout           | 用于获取响应的 HTTP 请求的超时 （TimeSpan 值）。 该值是获取响应而不是读取响应数据的超时。 默认值为 00:01:40。 | “否”       |
| maxConcurrentConnections | 若要同时连接到存储存储的连接数。 指定仅当你想要限制数据存储的并发连接。 | “否”       |

> [!NOTE]
> Parquet/分隔文本格式**HttpSource**作为仍受支持类型的复制活动源下一节中提到的是为了向后兼容。 建议以使用从长远看，此新模型和创作 UI 的 ADF 已切换为生成这些新类型。

**示例：**

```json
"activities":[
    {
        "name": "CopyFromHTTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSetting",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "HttpReadSetting",
                    "requestMethod": "Post",
                    "additionalHeaders": "<header key: header value>\n<header key: header value>\n",
                    "requestBody": "<body for POST HTTP request>"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="other-format-source"></a>其他格式源

若要将数据从 HTTP 中复制**ORC/Avro/JSON/二进制格式**，将复制活动中支持以下属性**源**部分：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| 类型 | 复制活动源的 type 属性必须设置为：HttpSource。 | “是” |
| httpRequestTimeout | 用于获取响应的 HTTP 请求的超时 （TimeSpan 值）。 该值是获取响应而不是读取响应数据的超时。 默认值为 00:01:40。  | “否” |

**示例**

```json
"activities":[
    {
        "name": "CopyFromHTTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<HTTP input dataset name>",
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
                "type": "HttpSource",
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```


## <a name="next-steps"></a>后续步骤

有关 Azure 数据工厂中复制活动支持用作源和接收器的数据存储的列表，请参阅[支持的数据存储和格式](copy-activity-overview.md#supported-data-stores-and-formats)。
