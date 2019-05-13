---
title: 使用 Azure 门户将数据导入搜索索引 - Azure 搜索
description: 了解如何使用 Azure 门户中的“导入数据”向导从 Cosmos DB、Blob 存储、表存储、SQL 数据库和 Azure VM 上的 SQL Server 抓取 Azure 数据。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: a0eefe38fdffd04bb95826f960771bd6430ea687
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024778"
---
# <a name="import-data-wizard-for-azure-search"></a>Azure 搜索的导入数据向导

Azure 门户在“Azure 搜索”仪表板上提供了“导入数据”向导，用于将数据加载到索引中。 在后台，向导配置和调用数据源、索引和索引器（自动执行索引过程的几个步骤）： 

* 连接到同一 Azure 订阅的外部数据源。
* （可选）集成光学字符识别或自然语言处理以从非结构化数据中提取文本。
* 生成基于数据采样和外部数据源元数据的索引。
* 抓取数据源以获取可搜索内容，序列化 JSON 文档并将其加载到索引中。

向导无法连接到预定义索引或运行现有索引器，但在向导中，你可以配置新索引或索引器以支持所需的结构和行为。

不熟悉 Azure 搜索？ 逐步执行[快速入门：导入、 索引和查询使用门户工具](search-get-started-portal.md)若要导入和索引使用测试驱动型**导入数据**和内置的房地产示例数据集。

## <a name="start-importing-data"></a>开始导入数据

本部分介绍如何启动向导，并提供每个步骤的高级概述。

1. 在 [Azure 门户](https://portal.azure.com)中，从仪表板打开搜索服务页，或者在服务列表中[查找服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。

2. 在顶部服务概述页中，单击“导入数据”。

   ![门户中的导入数据命令](./media/search-import-data-portal/import-data-cmd2.png "启动导入数据向导")

   > [!NOTE]
   > 可从其他 Azure 服务中启动“导入数据”，包括 Azure Cosmos DB、Azure SQL 数据库和 Azure Blob 存储。 在服务概述页上的左侧导航窗格中寻找“添加 Azure 搜索”。

3. 向导将打开到“连接到数据”，其中可选择要用于此导入的外部数据源。 关于此步骤，有几件事情需要了解，因此请务必阅读[数据源导入](#data-source-inputs)部分，了解更多详细信息。

   ![门户中的导入数据向导](./media/search-import-data-portal/import-data-wizard-startup.png "Azure 搜索的导入数据向导")

4. 下一步是“添加认知搜索”，如果想要在图像文件中包含文本的可选字符识别 (OCR)，或对非结构化数据进行文本分析。 对于此任务，可拉取认知服务中的 AI 算法。 此步骤由两个部分组成：
  
   首先，[将认知服务资源附加到](cognitive-search-attach-cognitive-services.md) Azure 搜索技能集。
  
   其次，选择要包含在技能集里的 AI 扩充。 有关演练演示，请参阅此[快速入门](cognitive-search-quickstart-blob.md)。

   如果只想导入数据，请跳过此步骤，直接转到索引定义。

5. 下一步是“自定义目标索引”，可在其中接受或修改出现在向导中的索引架构。 该向导通过从外部数据源采样数据和读取元数据来推断字段和数据类型。

   对于每个字段，[检查索引属性](#index-definition)以启用特定行为。 如果没有选择任何属性，索引将不可用。 

6. 下一步是“创建索引器”，这是此向导的一种产品。 索引器是一种爬网程序，可从外部 Azure 数据源中提取可搜索的数据和元数据。 选择数据源并附加技能集（可选）和索引，可在遍历向导的每个步骤时一直配置索引器。

   为索引器提供名称，单击“提交”，开始导入进程。 

可单击“索引器”列表中的索引器，监视门户中的索引进程。 加载文档以后，针对所定义索引的文档计数会增加。 有时候，门户页选取最新更新需要几分钟时间。

加载首个文档后，即可通过索引进行查询。 可针对此任务使用[搜索浏览器](search-explorer.md)。

<a name="data-source-inputs"></a>

## <a name="data-source-inputs"></a>数据源输入

“导入数据”向导创建一个持久数据源对象，可指定外部数据源的连接信息。 数据源对象专门用于[索引器](search-indexer-overview.md)，可以为以下数据源创建： 

* [Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Azure Cosmos DB](search-howto-index-cosmosdb.md)
* [Azure Blob 存储](search-howto-indexing-azure-blob-storage.md)
* [Azure 表存储](search-howto-indexing-azure-tables.md)（不支持[认知搜索](cognitive-search-concept-intro.md)管道）

您可以仅导入从单个表、 数据库视图或等效的数据结构，但结构可以包括分层或嵌套子结构。 有关详细信息，请参阅[如何建模复杂类型](search-howto-complex-data-types.md)。

运行该向导之前，应创建此数据结构，它必须包含的内容。 不会运行**导入数据**空的数据源向导。

|  选项 | 描述 |
| ---------- | ----------- |
| **现有数据源** |如果已在搜索服务中定义索引器，则可以为另一个导入选择现有数据源定义。 在 Azure 搜索中，索引器仅使用数据源对象。 可以以编程方式或通过“导入数据”向导创建数据源对象。|
| **示例**| Azure 搜索托管可用于了解有关 Azure 搜索中的导入和查询请求的免费全球 Azure SQL 数据库。 请参阅[快速入门：使用门户工具进行导入、编制索引和查询](search-get-started-portal.md)用于演练。 |
| **Azure SQL 数据库** |可以在此页上或通过 ADO.NET 连接字符串，指定服务名称、具有读取权限的数据库用户的凭据和数据库名称。 选择要查看或自定义属性的连接字符串选项。 <br/><br/>必须在此页上指定提供行集的表或视图。 连接成功后会显示此选项，并提供下拉列表以便可以进行选择。 |
| **Azure VM 上的 SQL Server** |指定完全限定的服务名、用户 ID 和密码以及数据库作为连接字符串。 若要使用此数据源，以前必须已在加密连接的本地存储中安装了证书。 如需说明，请参阅[与 Azure 搜索的 SQL VM 连接](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)。 <br/><br/>必须在此页上指定提供行集的表或视图。 连接成功后会显示此选项，并提供下拉列表以便可以进行选择。 |
| **Cosmos DB** |要求包括帐户、数据库和集合。 集合中的所有文档都将包含在索引中。 可以定义查询以平展或筛选行集，或将查询留空。 在此向导不需要查询。|
| **Azure Blob 存储** |要求包括存储帐户和容器。 （可选）如果 blob 名称遵循用于分组的虚拟命名约定，可以将名称的虚拟目录部分指定为容器下的某个文件夹。 有关详细信息，请参阅[为 Blob 存储编制索引](search-howto-indexing-azure-blob-storage.md)。 |
| **Azure 表存储** |要求包括存储帐户和表名。 （可选）可以指定一个查询来检索表的子集。 有关详细信息，请参阅[为表存储编制索引](search-howto-indexing-azure-tables.md)。 |


<a name="index-definition"></a>

## <a name="index-attributes"></a>索引属性

“导入数据”向导生成索引，将使用从输入数据源获取的文档对其进行填充。 

对于功能索引，请确保定义了以下元素。

1. 必须将一个字段标记为“密钥”，用于唯一标识每个文档。 “密钥”必须是“Edm.string”。 

   如果字段值包含空格或短划线，则必须在“高级选项”下的“创建索引器”步骤中设置“Base 64 编码密钥”选项以禁止验证检查这些字符。

1. 为每个字段设置索引属性。 如果选择无属性，索引实质上为空，所需密钥字段除外。 至少为每个字段选择一个或多个这些属性。
   
   + “可检索”在搜索结果中返回该字段。 用于提供内容以搜索结果的每个字段必须有此属性。 设置此字段不会明显影响索引大小。
   + “可筛选”允许在筛选表达式中引用该字段。 在 $filter 表达式中使用的每个字段必须有此属性。 筛选表达式用于精确匹配项。 由于文本字符串保持不变，因此需要额外的存储空间来容纳逐字内容。
   + “可搜索”启用全文搜索。 在自由格式查询或查询表达式中使用的每个字段必须有此属性。 为标记为“可搜索”的每个字段创建反向索引。

1. （可选）根据需要设置这些属性：

   + “可排序”允许在排序中使用该字段。 在 $Orderby 表达式中使用的每个字段必须有此属性。
   + “可查找”为分面导航启用该字段。 只有也标记为“可筛选”的字段可标记为“可查找”。

1. 如果需要强化了语言的索引和查询，请设置“分析器”。 默认值为“标准 Lucene”，但如果想要使用 Microsoft 的分析器以进行高级词汇处理（如解决不规则名词和动词形式），可选择“Microsoft 英语”。

   + 选择“可搜索”以启用“分析器”列表。
   + 选择该列表中提供的分析器。 
   
   目前，只能指定语言分析器。 使用自定义分析器或非语言分析器（如关键字、模式等）将需要代码。 有关分析器的详细信息，请参阅[为多语言文档创建索引](search-language-support.md)。

1. 选中“建议器”复选框，可对所选字段启用“提前键入查询建议”。


## <a name="next-steps"></a>后续步骤
若要了解有关索引器的详细信息，请查看以下链接：

* [为 Azure SQL 数据库编制索引](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [为 Azure Cosmos DB 编制索引](search-howto-index-cosmosdb.md)
* [为 Blob 存储编制索引](search-howto-indexing-azure-blob-storage.md)
* [为表存储编制索引](search-howto-indexing-azure-tables.md)