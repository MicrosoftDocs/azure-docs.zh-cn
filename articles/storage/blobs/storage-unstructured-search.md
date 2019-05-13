---
title: 教程：在 Azure Blob 存储中搜索非结构化数据
description: 教程：使用 Azure 搜索来搜索 Blob 存储中的非结构化数据。
author: roygara
services: storage
ms.service: storage
ms.topic: tutorial
ms.date: 12/13/2018
ms.author: rogarana
ms.custom: mvc
ms.openlocfilehash: 8c11d2e6d9a902707d3fd98f09d3474a5d2a5f64
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150789"
---
# <a name="tutorial-search-unstructured-data-in-cloud-storage"></a>教程：在云存储中搜索非结构化数据

本教程介绍如何使用 Azure Blob 存储中存储的数据通过 [Azure 搜索](../../search/search-what-is-azure-search.md)来搜索非结构化数据。 非结构化数据或者不按预定义的方式组织，或者不具备数据模型。 例如，一个 .txt 文件。

本教程要求读者有一个 Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建资源组
> * 创建存储帐户
> * 创建容器
> * 将数据上传到容器
> * 在门户中创建搜索服务
> * 将搜索服务连接到存储帐户
> * 创建数据源
> * 配置索引
> * 创建索引器
> * 使用搜索服务搜索容器

## <a name="prerequisites"></a>先决条件

每个存储帐户都必须属于 Azure 资源组。 资源组是对 Azure 资源进行分组的逻辑容器。 创建存储帐户时，可以选择创建新的资源组，或者使用现有的资源组。 本教程将创建新的资源组。

登录到 [Azure 门户](https://portal.azure.com)。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

示例数据集已准备好，可在本教程中使用。 下载 [clinical-trials.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials.zip) 并将其解压缩到所在的文件夹中。

此示例包含从[clinicaltrials.gov](https://clinicaltrials.gov/ct2/results) 获取的文本文件。 本教程将以这些文本文件为例，说明如何使用 Azure 搜索服务来搜索它们。

## <a name="create-a-container"></a>创建容器

这些容器类似于文件夹，用于存储 blob。

在本教程中，可以使用单个容器来存储从 clinicaltrials.gov 获取的文本文件。

1. 在 Azure 门户中转到自己的存储帐户。

2. 在“Blob 服务”下选择“浏览 Blob”。

3. 添加新容器。

4. 将容器命名为 **data**，并选择“容器”作为公共访问级别。

5. 选择“确定”创建容器。

   ![非结构化搜索](media/storage-unstructured-search/storageactinfo.png)

## <a name="upload-the-example-data"></a>上传示例数据

现在，你已拥有一个容器，可以将示例数据上传到该容器中。

1. 选择容器，然后选择“上传”。

2. 选择“文件”字段旁边的蓝色文件夹图标，浏览到示例数据解压缩到的本地文件夹。

3. 选择所有已解压缩的文件，然后选择“打开”

4. 选择“上传”，开始上传过程。

   ![非结构化搜索](media/storage-unstructured-search/upload.png)

上传过程可能需要花费片刻时间。

上传完成后，返回到 data 容器，以确认文本文件是否已上传。

  ![非结构化搜索](media/storage-unstructured-search/clinicalfolder.png)

## <a name="create-a-search-service"></a>创建搜索服务

Azure 搜索是一种搜索即服务云解决方案，它提供开发人员 API 和工具，以便基于数据添加丰富的搜索体验。

本教程使用某个搜索服务来搜索从 clinicaltrials.gov 获取的文本文件。

1. 在 Azure 门户中转到自己的存储帐户。

2. 向下滚动并选择“BLOB 服务”下的“添加 Azure 搜索”。

3. 在“导入数据”下，选择“选取服务”。

4. 选择“单击此处创建一个新的搜索服务”。

5. 在“新建搜索服务”**的**“URL”字段中，输入搜索服务的唯一名称。

6. 在“资源组”下，选择“使用现有项”并选择前面创建的资源组。

7. 为“定价层”选择“免费”层，然后单击“选择”。

8. 选择“创建”，创建搜索服务。

   ![非结构化搜索](media/storage-unstructured-search/createsearch2.png)

## <a name="connect-your-search-service-to-your-container"></a>将搜索服务连接到容器

现在，你已拥有一个搜索服务，可以将其附加到 blob 存储。 本部分将分步介绍选择数据源、创建索引，以及创建索引器的过程。

1. 转到存储帐户。

2. 选择“BLOB 服务”下的“添加 Azure 搜索”。

3. 在“导入数据”中选择“搜索服务”，然后单击在上一部分创建的搜索服务。 此时会打开“新建数据源”。

### <a name="create-a-data-source"></a>创建数据源

  数据源指定要编制索引的数据以及如何访问数据。 一个数据源可供同一搜索服务多次使用。

1. 输入数据源的名称。 在“要提取的数据”下，选择“内容和元数据”。 数据源指定要对 blob 的哪些部分编制索引。

2. 由于所用的 Blob 是文本文件，因此请将“分析模式”设置为“文本”。

   ![非结构化搜索](media/storage-unstructured-search/datasources.png)

3. 选择“存储容器”，列出可用的存储帐户。

4. 选择自己的存储帐户，然后选择前面创建的容器。

   ![非结构化搜索](media/storage-unstructured-search/datacontainer.png)

5. 单击“选择”返回到“新建数据源”，然后选择“确定”以继续。

### <a name="configure-the-index"></a>配置索引

  索引是来自可搜索的数据源的字段集合。 在这些字段中设置并配置参数，使搜索服务知道要以哪种方式搜索数据。

1. 在“导入数据”中，选择“自定义目标索引”。

2. 在“索引名称”字段中输入索引的名称。

3. 选中“metadata_storage_name”下的“可检索”属性的复选框。

   ![非结构化搜索](media/storage-unstructured-search/valuestoselect.png)

4. 选择“确定”，显示“创建索引器”。

编制索引的参数和为这些参数提供的属性都非常重要。 参数指定要存储哪些数据，属性指定如何存储这些数据。

“字段名称”列包含参数。 下表提供了可用属性及其说明的列表。

#### <a name="field-attributes"></a>字段属性

| 属性 | 说明 |
| --- | --- |
| *键* |为每个文档提供唯一 ID 以便查找文档的字符串。 每个索引必须有一个 key。 只有一个字段可以是 key，并且此字段类型必须设置为 Edm.String。 |
| *Retrievable* |指定是否可以在搜索结果中返回字段。 |
| *Filterable* |允许在筛选查询中使用字段。 |
| *Sortable* |允许查询使用此字段对搜索结果排序。 |
| *Facetable* |允许在分面导航结构中使用字段进行用户自主筛选。 通常，包含重复值的字段最适合分面导航，这些重复值可用于将文档（例如，同属一个品牌或服务类别的多个文档）组合在一起。 |
| *Searchable* |将字段标记为可全文搜索。 |

### <a name="create-an-indexer"></a>创建索引器

  索引器将数据源与搜索索引关联，并提供对数据重新编制索引的计划。

1. 在“名称”字段中输入一个名称，然后选择“确定”。

   ![非结构化搜索](media/storage-unstructured-search/exindexer.png)

2. 随后将返回到“导入数据”。 选择“确定”完成连接过程。

现在已成功将 blob 关联到搜索服务。 几分钟后，门户中才能显示索引已填充。 但是，搜索服务会立即开始编制索引，因此，可以即刻开始搜索。

## <a name="search-your-text-files"></a>搜索文本文件

若要搜索文件，请打开新创建的搜索服务的索引内的搜索资源管理器。

以下步骤介绍在何处可以找到搜索资源管理器，并提供一些示例查询：

1. 转到所有资源，并查找新建的搜索服务。

   ![非结构化搜索](media/storage-unstructured-search/exampleurl.png)

2. 选择索引将其打开。

   ![非结构化搜索](media/storage-unstructured-search/overview.png)

3. 选择“搜索资源管理器”打开搜索资源管理器，可在其中对数据进行实时查询。

   ![非结构化搜索](media/storage-unstructured-search/indexespane.png)

4. 在查询字符串字段为空时选择“搜索”。 空查询返回 blob 中的*所有*数据。

   ![非结构化搜索](media/storage-unstructured-search/emptySearch.png)

### <a name="perform-a-full-text-search"></a>执行全文搜索

在“查询字符串”字段中输入 **Myopia**，然后选择“搜索”。 此步骤会启动文件内容的搜索，并返回包含单词“Myopia”的一部分内容。

  ![非结构化搜索](media/storage-unstructured-search/secondMyopia.png)

### <a name="perform-a-system-properties-search"></a>执行系统属性搜索

除全文搜索以外，还可以使用 `$select` 参数创建按系统属性搜索的查询。

在查询字符串中输入 `$select=metadata_storage_name` 并按 **Enter**。 这只会返回该特定字段。

查询字符串直接修改 URL，因此不允许有空格。 若要搜索多个字段，请使用逗号，如：`$select=metadata_storage_name,metadata_storage_path`

`$select` 参数只能用于在定义索引时标记为“可检索”的字段。

  ![非结构化搜索](media/storage-unstructured-search/metadatasearchunstructured.png)

现在已经完成了本教程，并具有一个可搜索的非结构化数据集。

## <a name="clean-up-resources"></a>清理资源

若要删除所创建的资源，最简单的方法是删除资源组。 删除资源组还会删除该组中包含的所有资源。 在以下示例中，删除资源组会删除存储帐户和资源组本身。

1. 在 Azure 门户中，转到订阅中的资源组列表。
2. 选择要删除的资源组。
3. 选择“删除资源组”按钮，然后在删除字段中输入资源组的名称。
4. 选择“删除”。

## <a name="next-steps"></a>后续步骤

访问以下链接，详细了解如何使用 Azure 搜索为文档编制索引：

> [!div class="nextstepaction"]
> [使用 Azure 搜索为 Azure Blob 存储中的文档编制索引](../../search/search-howto-indexing-azure-blob-storage.md)
