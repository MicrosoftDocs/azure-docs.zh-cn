---
title: 为 Azure Blob 存储内容编制索引，以便进行全文搜索 - Azure 搜索
description: 了解如何使用 Azure 搜索为 Azure Blob 存储编制索引，以及从文档中提取文本。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: e55d596cfaf34c177f6dc43c27aaac37da87d2f7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024870"
---
# <a name="indexing-documents-in-azure-blob-storage-with-azure-search"></a>使用 Azure 搜索为 Azure Blob 存储中的文档编制索引
本文说明如何使用 Azure 搜索服务为存储在 Azure Blob 存储中的文档（例如 PDF、Microsoft Office 文档和其他多种常用格式的文档）编制索引。 首先，本文说明了设置和配置 Blob 索引器的基础知识。 其次，本文更加深入地探讨了你可能会遇到的行为和场景。

## <a name="supported-document-formats"></a>支持的文档格式
Blob 索引器可从以下文档格式提取文本：

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

## <a name="setting-up-blob-indexing"></a>设置 Blob 索引
可以使用以下方式设置 Azure Blob 存储索引器：

* [Azure 门户](https://ms.portal.azure.com)
* Azure 搜索 [REST API](https://docs.microsoft.com/rest/api/searchservice/Indexer-operations)
* Azure 搜索 [.NET SDK](https://aka.ms/search-sdk)

> [!NOTE]
> 某些功能（例如字段映射）在门户中尚不可用，必须以编程方式使用。
>

在这里，我们使用 REST API 演示流。

### <a name="step-1-create-a-data-source"></a>步骤 1：创建数据源
数据源指定要编制索引的数据、访问数据所需的凭据和有效标识数据更改（新行、修改的行或删除的行）的策略。 一个数据源可供同一搜索服务中的多个索引器使用。

若要为 Blob 编制索引，数据源必须具有以下属性：

* **name** 是搜索服务中数据源的唯一名称。
* **type** 必须是 `azureblob`。
* **credentials** 以 `credentials.connectionString` 参数的形式提供存储帐户连接字符串。 有关详细信息请参阅下方的[如何指定凭据](#Credentials)。
* **container** 指定存储帐户中的容器。 默认情况下，容器中的所有 Blob 都可检索。 如果只想为特定虚拟目录中的 Blob 编制索引，可以使用可选的 **query** 参数指定该目录。

创建数据源：

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "<optional-virtual-directory-name>" }
    }   

有关创建数据源 API 的详细信息，请参阅[创建数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)。

<a name="Credentials"></a>
#### <a name="how-to-specify-credentials"></a>如何指定凭据 ####

可通过以下一种方式提供 blob 容器的凭据：

- **完全访问存储帐户连接字符串**：`DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>` 您可以从 Azure 门户获取连接字符串，通过导航到存储帐户边栏选项卡 > 设置 > （对于经典存储帐户） 的密钥或设置 > 访问密钥 （适用于 Azure 资源管理器存储帐户）。
- **存储帐户共享访问签名** (SAS) 连接字符串：`BlobEndpoint=https://<your account>.blob.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=b&sp=rl` SAS 应具有容器和对象（本例中为 blob）的列表和读取权限。
-  **容器共享访问签名**：`ContainerSharedAccessUri=https://<your storage account>.blob.core.windows.net/<container name>?sv=2016-05-31&sr=c&sig=<the signature>&se=<the validity end time>&sp=rl` SAS 应具有容器的列表和读取权限。

有关存储共享访问签名的详细信息，请参阅[使用共享访问签名](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

> [!NOTE]
> 如果使用 SAS 凭据，则需使用续订的签名定期更新数据源凭据，以防止其过期。 如果 SAS 凭据过期，索引器会失败，出现类似于 `Credentials provided in the connection string are invalid or have expired.` 的错误消息。  

### <a name="step-2-create-an-index"></a>步骤 2：创建索引
索引指定文档、属性和其他构造中可以塑造搜索体验的字段。

下面介绍了如何使用可搜索 `content` 字段创建索引，以存储从 Blob 中提取的文本：   

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }

有关创建索引的详细信息，请参阅[创建索引](https://docs.microsoft.com/rest/api/searchservice/create-index)

### <a name="step-3-create-an-indexer"></a>步骤 3：创建索引器
索引器将数据源与目标搜索索引关联，并提供自动执行数据刷新的计划。

创建索引和数据源后，就可以准备创建索引器了：

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "blob-indexer",
      "dataSourceName" : "blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }

此索引器每隔两小时运行一次（已将计划间隔设置为“PT2H”）。 若要每隔 30 分钟运行一次索引器，可将间隔设置为“PT30M”。 支持的最短间隔为 5 分钟。 计划是可选的 - 如果省略，则索引器在创建后只运行一次。 但是，可以随时根据需要运行索引器。   

有关创建索引器 API 的更多详细信息，请参阅[创建索引器](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。

## <a name="how-azure-search-indexes-blobs"></a>Azure 搜索如何为 Blob 编制索引

根据具体的[索引器配置](#PartsOfBlobToIndex)，Blob 索引器可以仅为存储元数据编制索引（如果只关注元数据，而无需为 Blob 的内容编制索引，则此功能非常有用）、为存储元数据和内容元数据编制索引，或者同时为元数据和文本内容编制索引。 默认情况下，索引器提取元数据和内容。

> [!NOTE]
> 默认情况下，包含结构化内容（例如 JSON 或 CSV）的 lob 以单一文本区块的形式编制索引。 如果你想要 JSON 和 CSV blob 编制索引的结构化方式，请参阅[JSON blob 编制索引](search-howto-index-json-blobs.md)并[CSV blob 编制索引](search-howto-index-csv-blobs.md)有关详细信息。
>
> 复合或嵌入式文档（例如 ZIP 存档，或者嵌入了带附件 Outlook 电子邮件的 Word 文档）也以单一文档的形式编制索引。

* 文档的文本内容将提取到名为 `content` 的字符串字段中。

> [!NOTE]
> Azure 搜索会根据定价层限制提取的文本数量：免费层为 32,000 个字符，基本层为 64,000 个字符，标准层、标准 S2 层和标准 S3 层为 400 万个字符。 已截断的文本会在索引器状态响应中出现一条警告。  

* Blob 中用户指定的元数据属性（如果有）将逐字提取。
* 标准 Blob 元数据属性将提取到以下字段中：

  * **metadata\_storage\_name** (Edm.String) - Blob 的文件名。 例如，对于 Blob /my-container/my-folder/subfolder/resume.pdf 而言，此字段的值是 `resume.pdf`。
  * **metadata\_storage\_path** (Edm.String) - Blob 的完整 URI（包括存储帐户）。 例如： `https://myaccount.blob.core.windows.net/my-container/my-folder/subfolder/resume.pdf`
  * **metadata\_storage\_content\_type** (Edm.String) - 用于上传 Blob 的代码指定的内容类型。 例如，`application/octet-stream`。
  * **metadata\_storage\_last\_modified** (Edm.DateTimeOffset) - 上次修改 Blob 的时间戳。 Azure 搜索服务使用此时间戳来识别已更改的 Blob，避免在初次编制索引之后再次为所有内容编制索引。
  * **metadata\_storage\_size** (Edm.Int64) - Blob 大小，以字节为单位。
  * **metadata\_storage\_content\_md5** (Edm.String) - Blob 内容的 MD5 哈希（如果有）。
* 特定于每种文档格式的元数据属性将提取到[此处](#ContentSpecificMetadata)所列的字段。

无需在搜索索引中针对上述所有属性定义字段 - 系统只捕获应用程序所需的属性。

> [!NOTE]
> 通常，现有索引中的字段名称与文档提取期间所生成的字段名称不同。 可以使用**字段映射**将 Azure 搜索服务提供的属性名称映射到搜索索引中的字段名称。 后面会提供字段映射的用法示例。
>
>

<a name="DocumentKeys"></a>
### <a name="defining-document-keys-and-field-mappings"></a>定义文档键和字段映射
在 Azure 搜索中，文档键唯一标识某个文档。 每个搜索索引只能有一个类型为 Edm.String 的键字段。 键字段对于要添加到索引的每个文档必不可少（事实上它是唯一的必填字段）。  

应该仔细考虑要将提取的哪个字段映射到索引的键字段。 候选字段包括：

* **metadata\_storage\_name** - 这可能是一个便利的候选项，但是请注意，1) 名称可能不唯一，因为不同的文件夹中可能有同名的 Blob；2) 名称中包含的字符可能在文档键中无效，例如短划线。 可以使用 `base64Encode` [字段映射函数](search-indexer-field-mappings.md#base64EncodeFunction)处理无效的字符 - 如果这样做，请记得在将这些字符传入 Lookup 等 API 调用时为文档键编码。 （例如，在 .NET 中，可以使用 [UrlTokenEncode 方法](https://msdn.microsoft.com/library/system.web.httpserverutility.urltokenencode.aspx)实现此目的）。
* **metadata\_storage\_path** - 使用完整路径可确保唯一性，但是路径必定会包含 `/` 字符，而该字符[在文档键中无效](https://docs.microsoft.com/rest/api/searchservice/naming-rules)。  如上所述，可以选择使用 `base64Encode` [函数](search-indexer-field-mappings.md#base64EncodeFunction)为键编码。
* 如果上述所有做法都不起作用，可将一个自定义元数据属性添加到 Blob。 但是，这种做法需要通过 Blob 上传过程将该元数据属性添加到所有 Blob。 由于键是必需的属性，因此没有该属性的所有 Blob 都无法编制索引。

> [!IMPORTANT]
> 如果索引中没有键字段的显式映射，Azure 搜索服务会自动使用 `metadata_storage_path`（上述第二个选项）作为键并对该键进行 base-64 编码。
>
>

本示例选择 `metadata_storage_name` 字段作为文档键。 同时，假设索引具有名为 `key` 的键字段，以及一个用于存储文档大小的 `fileSize` 字段。 若要连接所需的元素，请在创建或更新索引器时指定以下字段映射：

    "fieldMappings" : [
      { "sourceFieldName" : "metadata_storage_name", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
      { "sourceFieldName" : "metadata_storage_size", "targetFieldName" : "fileSize" }
    ]

要将所有元素合并在一起，可按如下所示添加字段映射，并为现有索引器的键启用 base-64 编码：

    PUT https://[service name].search.windows.net/indexers/blob-indexer?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "dataSourceName" : " blob-datasource ",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "fieldMappings" : [
        { "sourceFieldName" : "metadata_storage_name", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
        { "sourceFieldName" : "metadata_storage_size", "targetFieldName" : "fileSize" }
      ]
    }

> [!NOTE]
> 有关字段映射的详细信息，请参阅[此文](search-indexer-field-mappings.md)。
>
>

<a name="WhichBlobsAreIndexed"></a>
## <a name="controlling-which-blobs-are-indexed"></a>控制要为哪些 Blob 编制索引
可以控制要为哪些 Blob 编制索引，以及要跳过哪些 Blob。

### <a name="index-only-the-blobs-with-specific-file-extensions"></a>只为具有特定文件扩展名的 Blob 编制索引
使用 `indexedFileNameExtensions` 索引器配置参数可以做到只为具有指定扩展名的 Blob 编制索引。 值是包含文件扩展名（包括前置句点）逗号分隔列表的字符串。 例如，如果只要为 .PDF 和 .DOCX Blob 编制索引，请执行以下操作：

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "indexedFileNameExtensions" : ".pdf,.docx" } }
    }

### <a name="exclude-blobs-with-specific-file-extensions"></a>排除具有特定文件扩展名的 Blob
使用 `excludedFileNameExtensions` 配置参数可在编制索引时排除具有特定文件扩展名的 Blob。 值是包含文件扩展名（包括前置句点）逗号分隔列表的字符串。 例如，若要为所有 Blob 编制索引，但要排除具有 .PNG 和 .JPEG 扩展名的 Blob，请执行以下操作：

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "excludedFileNameExtensions" : ".png,.jpeg" } }
    }

如果同时存在 `indexedFileNameExtensions` 和 `excludedFileNameExtensions` 参数，Azure 搜索服务会先查找 `indexedFileNameExtensions`，再查找 `excludedFileNameExtensions`。 这意味着，如果两个列表中存在同一个文件扩展名，将从索引编制中排除该扩展名。

<a name="PartsOfBlobToIndex"></a>
## <a name="controlling-which-parts-of-the-blob-are-indexed"></a>控制要为 Blob 中的哪些部分编制索引

可以使用 `dataToExtract` 配置参数控制要为 Blob 中的哪些部分编制索引。 该参数采用以下值：

* `storageMetadata` - 指定仅为[标准 Blob 属性和用户指定的元数据](../storage/blobs/storage-properties-metadata.md)编制索引。
* `allMetadata` - 指定要为从 Blob 内容中提取的存储元数据和[内容类型特定的元数据](#ContentSpecificMetadata)编制索引。
* `contentAndMetadata` - 指定要为从 Blob 中提取的所有元数据和文本内容编制索引。 这是默认值。

例如，如果只要为存储元数据编制索引，请使用：

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "dataToExtract" : "storageMetadata" } }
    }

### <a name="using-blob-metadata-to-control-how-blobs-are-indexed"></a>使用 Blob 元数据来控制如何为 Blob 编制索引

上述配置参数适用于所有 Blob。 有时，你可能想要控制为*个体 Blob* 编制索引的方式。 为此，可以添加以下 Blob 元数据属性和值：

| 属性名称 | 属性值 | 说明 |
| --- | --- | --- |
| AzureSearch_Skip |"true" |指示 Blob 索引器完全跳过该 Blob， 既不尝试提取元数据，也不提取内容。 如果特定的 Blob 反复失败并且中断编制索引过程，此属性非常有用。 |
| AzureSearch_SkipContent |"true" |此属性等效于[上面](#PartsOfBlobToIndex)所述的与特定 Blob 相关的 `"dataToExtract" : "allMetadata"` 设置。 |

<a name="DealingWithErrors"></a>
## <a name="dealing-with-errors"></a>处理错误

默认情况下，Blob 索引器一旦遇到包含不受支持内容类型（例如图像）的 Blob 时，就会立即停止。 当然，可以使用 `excludedFileNameExtensions` 参数跳过某些内容类型。 但是，可能需要在未事先了解所有可能的内容类型的情况下，为 Blob 编制索引。 要在遇到了不受支持的内容类型时继续编制索引，可将 `failOnUnsupportedContentType` 配置参数设置为 `false`：

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false } }
    }

对于某些 blob，Azure 搜索无法确定其内容类型，或无法处理其他受支持内容类型的文档。 若要忽略此故障模式，将 `failOnUnprocessableDocument` 配置参数设置为 false：

      "parameters" : { "configuration" : { "failOnUnprocessableDocument" : false } }

Azure 搜索会限制编入索引的 blob 的大小。 这些限制记录在 [Azure 搜索中的服务限制](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity)中。 过大的 blob 会被默认视为错误。 但是，如果将 `indexStorageMetadataOnlyForOversizedDocuments` 配置参数设为 true，你仍可以对过大 blob 的存储元数据编制索引： 

    "parameters" : { "configuration" : { "indexStorageMetadataOnlyForOversizedDocuments" : true } }

如果在任意处理点（无论是在解析 blob 时，还是在将文档添加到索引时）发生错误，仍然可以继续索引。 若要忽略特定的错误数，将 `maxFailedItems` 和 `maxFailedItemsPerBatch` 配置参数设置为所需值。 例如：

    {
      ... other parts of indexer definition
      "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 10 }
    }

## <a name="incremental-indexing-and-deletion-detection"></a>增量索引和删除检测
将 Blob 索引器设置为按计划运行时，它将只根据 Blob 的 `LastModified` 时间戳，为更改的 Blob 重新编制索引。

> [!NOTE]
> 无需指定更改检测策略 - 系统会自动启用增量索引。

若要支持删除文档，请使用“软删除”方法。 如果彻底删除 Blob，相应的文档不会从搜索索引中删除。 应该改用以下步骤：  

1. 将一个自定义元数据属性添加到 Blob，告知 Azure 搜索该 Blob 已按逻辑方式删除
2. 在数据源上配置软删除检测策略
3. 索引器处理 Blob 后（如索引器状态 API 所示），可以使用物理方式删除该 Blob

例如，如果某个 Blob 具有值为 `true` 的元数据属性 `IsDeleted`，以下策略会将该 Blob 视为已删除：

    PUT https://[service name].search.windows.net/datasources/blob-datasource?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : "my-folder" },
        "dataDeletionDetectionPolicy" : {
            "@odata.type" :"#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",     
            "softDeleteColumnName" : "IsDeleted",
            "softDeleteMarkerValue" : "true"
        }
    }   

## <a name="indexing-large-datasets"></a>为大型数据集编制索引

Blob 编制索引可能是一个耗时的过程。 如果有几百万个 Blob 需要编制索引，可以将数据分区，并使用多个索引器来并行处理数据，从而加快索引编制的速度。 设置方法如下：

- 将数据分区到多个 Blob 容器或虚拟文件夹
- 设置多个 Azure 搜索数据源，为每个容器或文件夹各设置一个。 若要指向某个 Blob 文件夹，请使用 `query` 参数：

    ```
    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : "my-folder" }
    }
    ```

- 为每个数据源创建相应的索引器。 所有索引器可以指向同一目标搜索索引。  

- 服务中的每个搜索单位在任何给定的时间都只能运行一个索引器。 只有当索引器实际上并行运行时，如上所述创建多个索引器才很有用。 若要并行运行多个索引器，请通过创建合适数量的分区和副本来横向扩展搜索服务。 例如，如果搜索服务有 6 个搜索单位（例如，2 个分区 x 3 个副本），则 6 个索引器可以同时运行，导致索引吞吐量增加六倍。 若要详细了解缩放和容量规划，请参阅[在 Azure 搜索中缩放用于查询和索引工作负荷的资源级别](search-capacity-planning.md)。

## <a name="indexing-documents-along-with-related-data"></a>对文档以及相关数据进行索引

你可能希望从索引中的多个源“组装”文档。 例如，你可能希望将 blob 中的文本与 Cosmos DB 中存储的其他元数据进行合并。 甚至可以将推送索引 API 与各种索引器一起使用来基于多个部件搭建搜索文档。 

若要使此方式可行，所有索引器和其他组件需要针对文档键达成一致。 有关详细演练，请参阅外部文章：[将文档与 Azure 搜索中的其他数据相结合](https://blog.lytzen.name/2017/01/combine-documents-with-other-data-in.html)。

<a name="IndexingPlainText"></a>
## <a name="indexing-plain-text"></a>为纯文本编制索引 

如果所有 blob 都包含采用同一编码的纯文本，则可以通过使用**文本分析模式**显著提高索引编制性能。 若要使用文本分析模式，请将 `parsingMode` 配置属性设置为 `text`：

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "parsingMode" : "text" } }
    }

默认情况下将采用 `UTF-8` 编码。 若要指定不同的编码，请使用 `encoding` 配置属性： 

    {
      ... other parts of indexer definition
      "parameters" : { "configuration" : { "parsingMode" : "text", "encoding" : "windows-1252" } }
    }


<a name="ContentSpecificMetadata"></a>
## <a name="content-type-specific-metadata-properties"></a>特定于内容类型的元数据属性
下表汇总了针对每种文档格式执行的处理，并说明了 Azure 搜索提取的元数据属性。

| 文档格式/内容类型 | 特定于内容类型的元数据属性 | 处理详细信息 |
| --- | --- | --- |
| HTML (`text/html`) |`metadata_content_encoding`<br/>`metadata_content_type`<br/>`metadata_language`<br/>`metadata_description`<br/>`metadata_keywords`<br/>`metadata_title` |剥离 HTML 标记并提取文本 |
| PDF (`application/pdf`) |`metadata_content_type`<br/>`metadata_language`<br/>`metadata_author`<br/>`metadata_title` |提取文本，包括嵌入的文档（不包括图像） |
| DOCX (application/vnd.openxmlformats-officedocument.wordprocessingml.document) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |提取文本，包括嵌入的文档 |
| DOC (application/msword) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |提取文本，包括嵌入的文档 |
| XLSX (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |提取文本，包括嵌入的文档 |
| XLS (application/vnd.ms-excel) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |提取文本，包括嵌入的文档 |
| PPTX (application/vnd.openxmlformats-officedocument.presentationml.presentation) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |提取文本，包括嵌入的文档 |
| PPT (application/vnd.ms-powerpoint) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |提取文本，包括嵌入的文档 |
| MSG (application/vnd.ms-outlook) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_to`<br/>`metadata_message_cc`<br/>`metadata_message_bcc`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_subject` |提取文本，包括附件 |
| ZIP (application/zip) |`metadata_content_type` |从存档中的所有文档提取文本 |
| XML (application/xml) |`metadata_content_type`</br>`metadata_content_encoding`</br> |剥离 XML 标记并提取文本 |
| JSON (application/json) |`metadata_content_type`</br>`metadata_content_encoding` |提取文本<br/>注意：如果需要从 JSON Blob 提取多个文档字段，请参阅[为 JSON Blob 编制索引](search-howto-index-json-blobs.md)了解详细信息 |
| EML (message/rfc822) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_to`<br/>`metadata_message_cc`<br/>`metadata_creation_date`<br/>`metadata_subject` |提取文本，包括附件 |
| RTF（应用程序/rtf） |`metadata_content_type`</br>`metadata_author`</br>`metadata_character_count`</br>`metadata_creation_date`</br>`metadata_page_count`</br>`metadata_word_count`</br> | 提取文本|
| 纯文本 (text/plain) |`metadata_content_type`</br>`metadata_content_encoding`</br> | 提取文本|


## <a name="help-us-make-azure-search-better"></a>帮助我们改进 Azure 搜索
如果想要请求新功能或者在改进方面有什么看法，敬请通过 [UserVoice 站点](https://feedback.azure.com/forums/263029-azure-search/)告诉我们。
