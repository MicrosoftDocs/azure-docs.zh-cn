---
title: 为 Azure 表存储中的内容编制索引，以便进行全文搜索 - Azure 搜索
description: 了解如何使用 Azure 搜索索引器为 Azure 表存储中存储的数据编制索引。
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 5f0e7feb52b34a4bd29bef01925bf9ea8f84d7db
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024787"
---
# <a name="index-azure-table-storage-with-azure-search"></a>使用 Azure 搜索编制 Azure 表存储的索引
本文介绍如何使用 Azure 搜索对 Azure 表存储中存储的数据编制索引。

## <a name="set-up-azure-table-storage-indexing"></a>设置 Azure 表存储索引

可以使用以下资源设置 Azure 表存储索引器：

* [Azure 门户](https://ms.portal.azure.com)
* Azure 搜索 [REST API](https://docs.microsoft.com/rest/api/searchservice/Indexer-operations)
* Azure 搜索 [.NET SDK](https://aka.ms/search-sdk)

在这里，我们使用 REST API 演示流。 

### <a name="step-1-create-a-datasource"></a>步骤 1：创建数据源

数据源指定要索引的数据、访问数据所需的凭据和支持 Azure 搜索有效标识数据更改的策略。

若要为表编制索引，数据源必须具有以下属性：

- **name** 是搜索服务中数据源的唯一名称。
- **type** 必须是 `azuretable`。
- **credentials** 参数包含存储帐户连接字符串。 有关详细信息，请参阅[指定凭据](#Credentials)部分。
- **container** 设置表名称和可选查询。
    - 使用 `name` 参数指定表名称。
    - （可选）使用 `query` 参数指定查询。 

> [!IMPORTANT] 
> 为使性能更佳，请尽可能对 PartitionKey 使用筛选器。 任何其他查询会执行全表扫描，导致大型表性能不佳。 请参阅[性能注意事项](#Performance)部分。


若要创建数据源，请执行以下操作：

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "table-datasource",
        "type" : "azuretable",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-table", "query" : "PartitionKey eq '123'" }
    }   

有关创建数据源 API 的详细信息，请参阅[创建数据源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)。

<a name="Credentials"></a>
#### <a name="ways-to-specify-credentials"></a>指定凭据的方式 ####

可通过以下一种方式提供表的凭据： 

- **完全访问存储帐户连接字符串**：`DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>` 可通过转到“存储帐户”边栏选项卡  > “设置” > “密钥”（适用于经典存储帐户）或“设置” > “访问密钥”（适用于 Azure 资源管理器存储帐户），从 Azure 门户获取连接字符串。
- **存储帐户共享访问签名连接字符串**：`TableEndpoint=https://<your account>.table.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=t&sp=rl` 共享访问签名应具有容器（本例中为表）和对象（表行）的列表和读取权限。
-  **表共享访问签名**：`ContainerSharedAccessUri=https://<your storage account>.table.core.windows.net/<table name>?tn=<table name>&sv=2016-05-31&sig=<the signature>&se=<the validity end time>&sp=r` 共享访问签名应具有表的查询（读取）权限。

有关存储共享访问签名的详细信息，请参阅[使用共享访问签名](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

> [!NOTE]
> 如果使用共享访问签名凭据，则需使用续订的签名定期更新数据源凭据，以防止其过期。 如果共享访问签名凭据过期，索引器会失败并出现类似于“连接字符串中提供的凭据无效或已过期”的错误消息。  

### <a name="step-2-create-an-index"></a>步骤 2：创建索引
索引指定文档、属性和其他构造中可以塑造搜索体验的字段。

若要创建索引，请执行以下操作：

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "key", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "SomeColumnInMyTable", "type": "Edm.String", "searchable": true }
          ]
    }

有关创建索引的详细信息，请参阅[创建索引](https://docs.microsoft.com/rest/api/searchservice/create-index)。

### <a name="step-3-create-an-indexer"></a>步骤 3：创建索引器
索引器将数据源与目标搜索索引关联，并提供自动执行数据刷新的计划。 

创建索引和数据源后，可以创建索引器：

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "table-indexer",
      "dataSourceName" : "table-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }

此索引器每两小时运行一次。 （已将计划间隔设置为“PT2H”）。若要每隔 30 分钟运行一次索引器，可将间隔设置为“PT30M”。 支持的最短间隔为 5 分钟。 计划是可选的；如果省略，则索引器在创建后只运行一次。 但是，可以随时根据需要运行索引器。   

有关创建索引器 API 的详细信息，请参阅[创建索引器](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。

## <a name="deal-with-different-field-names"></a>处理不同的字段名称
有时，现有索引中的字段名称会不同于表中的属性名称。 可以使用字段映射将表中的属性名称映射到搜索索引中的字段名称。 若要详细了解字段映射，请参阅 [Azure 搜索索引器字段映射弥补数据源和搜索索引之间的差异](search-indexer-field-mappings.md)。

## <a name="handle-document-keys"></a>处理文档键
在 Azure 搜索中，文档键唯一标识某个文档。 每个搜索索引必须只有一个类型为 `Edm.String` 的键字段。 键字段对于要添加到索引的每个文档必不可少。 （事实上，它是唯一的必填字段。）

由于表行具有复合键，因此 Azure 搜索会生成一个名为 `Key` 的合成字段，该字段是分区键值和行键值的串接。 例如，如果行的 PartitionKey 为 `PK1`、RowKey 为 `RK1`，那么 `Key` 字段的值为 `PK1RK1`。

> [!NOTE]
> `Key` 值中可能含有文档键中无效的字符（如短划线）。 可以通过使用 `base64Encode` [字段映射函数](search-indexer-field-mappings.md#base64EncodeFunction)处理无效字符。 如果执行此操作，在 API 调用（如 Lookup）中传递文档键时，还请记得使用 URL-safe Base64 编码。
>
>

## <a name="incremental-indexing-and-deletion-detection"></a>增量索引和删除检测
当将表索引器设置为按计划运行时，它仅对由行的 `Timestamp` 值确定的新行或更新行重新编制索引。 无需指定更改检测策略。 系统会自动启用增量索引。

若要指示必须从索引中删除某些文档，可使用软删除策略。 不删除行，而是添加一个属性来指示删除行，并对数据源设置软删除检测策略。 例如，如果某行具有值为 `"true"` 的属性 `IsDeleted`，以下策略会将该行视为已删除：

    PUT https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-table-datasource",
        "type" : "azuretable",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "table name", "query" : "<query>" },
        "dataDeletionDetectionPolicy" : { "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy", "softDeleteColumnName" : "IsDeleted", "softDeleteMarkerValue" : "true" }
    }   

<a name="Performance"></a>
## <a name="performance-considerations"></a>性能注意事项

默认情况下，Azure 搜索使用以下查询筛选器：`Timestamp >= HighWaterMarkValue`。 由于 Azure 表在 `Timestamp` 字段上没有辅助索引，因此该类型的查询需要执行全表扫描，导致大型表查询速度慢。


下面是两种可能提高表索引性能的方法。 这两种方法都依赖于使用表分区： 

- 如果可自然地将数据分区到多个分区范围中，可为每个分区范围创建数据源和相应的索引器。 现在每个索引器仅能处理一个特定分区范围，使得查询性能更佳。 如果需编制索引的数据具有较少的固定分区，查询性能会更好：每个索引器仅执行一次分区扫描。 例如，若要创建一个数据源用来处理含有键 `000` 到 `100` 的分区范围，请使用以下查询： 
    ```
    "container" : { "name" : "my-table", "query" : "PartitionKey ge '000' and PartitionKey lt '100' " }
    ```

- 如果数据按时间分区（例如，每天或每周创建一个新分区），请考虑以下方法： 
    - 使用此格式的查询：`(PartitionKey ge <TimeStamp>) and (other filters)`。 
    - 使用[获取索引器状态 API](https://docs.microsoft.com/rest/api/searchservice/get-indexer-status)监视器索引器进度，并基于最新的成功的高使用标记值定期更新查询的 `<TimeStamp>` 条件。 
    - 借助此方法，如果需要触发完整的索引重编制，除了重置索引器外还需要重置数据源查询。 


## <a name="help-us-make-azure-search-better"></a>帮助我们改进 Azure 搜索
如果有功能请求或改进建议，请在我们的 [UserVoice 站点](https://feedback.azure.com/forums/263029-azure-search/)上提交。
