---
title: 使用索引器进行自动索引的字段映射 - Azure 搜索
description: 针对字段名称和数据表示的差异配置帐户的 Azure 搜索索引器字段映射。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: d3e0d876550b0c3baf89f3f13e0458fc97e11351
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025215"
---
# <a name="field-mappings-in-azure-search-indexers"></a>Azure 搜索索引器中的字段映射
使用 Azure 搜索索引器时，偶尔可能发现自己处于输入数据与目标索引架构不完全匹配的情形。 在这些情况下，可以使用**字段映射**将数据转换为所需形状。

在某些情况下，字段映射会很有用：

* 数据源具有字段 `_id`，但 Azure 搜索不允许字段名称以下划线开头。 字段映射允许“重命名”字段。
* 例如，你想要使用相同的数据来源数据填充索引中的多个字段，因为想要将不同的分析器应用于这些字段。 字段映射让可以“分叉”数据源字段。
* 需要对数据进行 Base64 编码或解码。 字段映射支持多个**映射函数**，包括用于 Base64 编码和解码的函数。   

## <a name="setting-up-field-mappings"></a>设置字段映射
当使用[创建索引器](https://msdn.microsoft.com/library/azure/dn946899.aspx) API 创建新索引器时，可以添加字段映射。 使用[更新索引器](https://msdn.microsoft.com/library/azure/dn946892.aspx) API，可以管理编制索引索引器上的字段映射。

字段映射由 3 部分组成：

1. `sourceFieldName`，它表示数据源中的字段。 此属性是必需项。
2. 可选的 `targetFieldName`，它表示搜索索引中的字段。 如果已省略，则使用数据源中相同的名称。
3. 可选的 `mappingFunction`，它可以使用几个预定义函数中的一个来转换数据。 函数的完整列表[如下](#mappingFunctions)。

字段映射将添加到索引器定义上的 `fieldMappings` 数组中。

例如，下面介绍如何适应字段名称的差异：

```JSON

PUT https://[service name].search.windows.net/indexers/myindexer?api-version=[api-version]
Content-Type: application/json
api-key: [admin key]
{
    "dataSourceName" : "mydatasource",
    "targetIndexName" : "myindex",
    "fieldMappings" : [ { "sourceFieldName" : "_id", "targetFieldName" : "id" } ]
}
```

索引器可以有多个字段映射。 例如，下面介绍如何“分叉”某个字段：

```JSON

"fieldMappings" : [
    { "sourceFieldName" : "text", "targetFieldName" : "textStandardEnglishAnalyzer" },
    { "sourceFieldName" : "text", "targetFieldName" : "textSoundexAnalyzer" }
]
```

> [!NOTE]
> Azure 搜索使用不区分大小写的比较，来解析字段映射中的字段和函数名称。 此操作很方便（大小写无需全都正确），但这表示数据源或索引无法具有仅大小写不同的字段。  
>
>

<a name="mappingFunctions"></a>

## <a name="field-mapping-functions"></a>字段映射函数
当前支持以下函数：

* [base64Encode](#base64EncodeFunction)
* [base64Decode](#base64DecodeFunction)
* [extractTokenAtPosition](#extractTokenAtPositionFunction)
* [jsonArrayToStringCollection](#jsonArrayToStringCollectionFunction)

<a name="base64EncodeFunction"></a>

## <a name="base64encode"></a>base64Encode
执行输入字符串的 *URL 安全* Base64 编码。 假定输入采用 UTF-8 进行编码。

### <a name="sample-use-case---document-key-lookup"></a>示例用例 - 查找文档密钥
Azure 搜索文档密钥中只能使用 URL 安全字符（因为客户必须要能够使用[查找 API](https://docs.microsoft.com/rest/api/searchservice/lookup-document) 等来查找文档）。 如果数据包含 URL 不安全的字符，但希望使用它来填充搜索索引中的密钥字段，可使用此函数。 对密钥编码后，可以使用 base64 解码以检索原始值。 有关详细信息，请参阅 [base64 编码和解码](#base64details)部分。

#### <a name="example"></a>示例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : { "name" : "base64Encode" }
  }]
```

### <a name="sample-use-case---retrieve-original-key"></a>示例用例 - 检索原始密钥
Blob 索引器将 blob 路径元数据用作文档密钥来索引 blob。 检索已编码文档密钥后，需要解码该路径并下载 blob。

#### <a name="example"></a>示例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : { "name" : "base64Encode", "parameters" : { "useHttpServerUtilityUrlTokenEncode" : false } }
  }]
 ```

如果无需根据密钥查找文档，也无需对已编码内容进行解码，则可以忽略映射函数的 `parameters`（`useHttpServerUtilityUrlTokenEncode` 默认设置为 `true`）。 否则，请参阅 [base64 详细信息](#base64details)部分，确定要使用哪些设置。

<a name="base64DecodeFunction"></a>

## <a name="base64decode"></a>base64Decode
执行输入字符串的 Base64 解码。 输入假定为 *URL 安全* Base64 编码的字符串。

### <a name="sample-use-case"></a>示例用例
Blob 自定义元数据值必须以 ASCII 进行编码。 可以使用 Base64 编码来表示 blob 自定义元数据中的任意 UTF-8 字符串。 但是，为了使搜索有意义，可以在填充搜索索引时，使用此函数将编码的数据转换回“常规”字符串。

#### <a name="example"></a>示例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "Base64EncodedMetadata",
    "targetFieldName" : "SearchableMetadata",
    "mappingFunction" : { "name" : "base64Decode", "parameters" : { "useHttpServerUtilityUrlTokenDecode" : false } }
  }]
```

如果不指定任何 `parameters`，则 `useHttpServerUtilityUrlTokenDecode` 的默认值为 `true`。 请参阅 [base64 详细信息](#base64details)部分，确定要使用哪些设置。

<a name="base64details"></a>

### <a name="details-of-base64-encoding-and-decoding"></a>base64 编码和解码详细信息
Azure 搜索支持两种 base64 编码：HttpServerUtility URL 令牌和无填充 URL 安全 base64 编码。 如果想对文档密钥编码以便查找、对值编码以供索引器解码或解码由索引器编码的字段，需要使用相同的编码作为映射函数。

如果将用于编码或解码的 `useHttpServerUtilityUrlTokenEncode` 或 `useHttpServerUtilityUrlTokenDecode` 参数分别设置为 `true`，则 `base64Encode` 的行为与 [HttpServerUtility.UrlTokenEncode](https://msdn.microsoft.com/library/system.web.httpserverutility.urltokenencode.aspx) 类似，`base64Decode` 的行为与 [HttpServerUtility.UrlTokenDecode](https://msdn.microsoft.com/library/system.web.httpserverutility.urltokendecode.aspx) 类似。

如果没有使用完整的.NET Framework（即使用的是 .NET Core 或其他编程环境）来生成模拟 Azure 搜索行为的键值，则应将 `useHttpServerUtilityUrlTokenEncode` 和 `useHttpServerUtilityUrlTokenDecode` 设置为 `false`。 Base64 编码和解码实用工具函数可能与 Azure 搜索不同，具体取决于所使用的库。

下表比较了对字符串 `00>00?00` 进行不同的 base64 编码的结果。 若要确定 base64 函数所需的其他处理（如有），请对字符串 `00>00?00` 应用库编码函数，然后比较输出和预期的输出 `MDA-MDA_MDA`。

| 编码 | Base64 编码输出 | 库编码后的其他处理 | 库解码前的其他处理 |
| --- | --- | --- | --- |
| 带填充的 Base64 | `MDA+MDA/MDA=` | 使用 URL 安全字符并删除填充 | 使用标准 base64 字符并添加填充 |
| 不带填充的 Base64 | `MDA+MDA/MDA` | 使用 URL 安全字符 | 使用标准 base64 字符 |
| 带填充的 URL 安全 Base64 | `MDA-MDA_MDA=` | 删除填充 | 添加填充 |
| 不带填充的 URL 安全 Base64 | `MDA-MDA_MDA` | 无 | 无 |

<a name="extractTokenAtPositionFunction"></a>

## <a name="extracttokenatposition"></a>extractTokenAtPosition
使用指定的分隔符拆分字符串字段，并在所生成拆分的指定位置处选取令牌。

例如，如果输入是 `Jane Doe`，`delimiter` 是 `" "`（空格）并且 `position` 是 0，则结果为 `Jane`；如果 `position` 是 1，则结果是 `Doe`。 如果位置引用的令牌不存在，则会返回错误。

### <a name="sample-use-case"></a>示例用例
数据源包含 `PersonName` 字段，并且想要为其编制索引作为两个单独的 `FirstName` 和 `LastName` 字段。 可以使用此函数来拆分将空格字符用作分隔符的输入。

### <a name="parameters"></a>parameters
* `delimiter`：在拆分输入字符串时，用作分隔符的字符串。
* `position`：在拆分输入字段串后要选取的位置，以零为底的整数。    

### <a name="example"></a>示例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "FirstName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 0 } }
  },
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "LastName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 1 } }
  }]
```

<a name="jsonArrayToStringCollectionFunction"></a>

## <a name="jsonarraytostringcollection"></a>jsonArrayToStringCollection
将已格式化为 JSON 字符串数组的字符串转换为可用于填充索引中 `Collection(Edm.String)` 字段的字符串数组。

例如，如果输入字符串是 `["red", "white", "blue"]`，类型 `Collection(Edm.String)` 的目标字段由 `red`、`white` 和 `blue` 这三个值填充。 对于无法分析为 JSON 字符串数组的输入值，则会返回错误。

### <a name="sample-use-case"></a>示例用例
Azure SQL 数据库不具有能自然映射到 Azure 搜索中 `Collection(Edm.String)` 字段的内置数据类型。 要填充字符串集合字段，请将源数据格式为 JSON 字符串数组，并使用此函数。

### <a name="example"></a>示例
```JSON

"fieldMappings" : [
  { "sourceFieldName" : "tags", "mappingFunction" : { "name" : "jsonArrayToStringCollection" } }
]
```


## <a name="help-us-make-azure-search-better"></a>帮助我们改进 Azure 搜索
如果有功能请求或改进建议，请通过我们的 [UserVoice 站点](https://feedback.azure.com/forums/263029-azure-search/)与我们联系。
