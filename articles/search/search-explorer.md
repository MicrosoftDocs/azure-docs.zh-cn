---
title: Azure 门户中用于查询数据的搜索浏览器工具 - Azure 搜索
description: 使用搜索浏览器等 Azure 门户工具在 Azure 搜索中查询索引。 使用高级语法输入搜索词或完全限定的搜索字符串。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 392699182859a090c13304f63d28a78b95a65ec7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024026"
---
# <a name="search-explorer-for-querying-data-in-azure-search"></a>Azure 搜索中用于查询数据的搜索浏览器 

本文介绍如何使用搜索浏览器在 Azure 门户中查询现有 Azure 搜索索引。 可以使用搜索浏览器，向服务中的任何现有索引提交简单或完整的 Lucene 查询字符串。 

   ![门户中的搜索资源管理器命令](./media/search-explorer/search-explorer-cmd2.png "Search explorer command in porta")


有关入门的帮助，请参阅[启动搜索资源管理器](#start-search-explorer)。

## <a name="basic-search-strings"></a>基本搜索字符串

以下示例假定为内置的房地产示例索引。 有关创建此索引的帮助，请参阅[快速入门：Azure 门户中的导入、索引和查询](search-get-started-portal.md)。

### <a name="example-1---empty-search"></a>示例 1 - 空搜索

若要首先查看内容，请执行空搜索，方法是单击“搜索”，不提供搜索词。 空搜索作为第一个查询十分有用，因为它返回全部文档，以便查看文档组合。 空搜索没有搜索级别，按任意顺序返回文档（所有文档都为 `"@search.score": 1`）。 默认情况下，搜索请求中会返回 50 个文档。

空搜索的等效语法是 `*` 或 `search=*`。

   ```Input
   search=*
   ```

   **结果**
   
   ![空查询示例](./media/search-explorer/search-explorer-example-empty.png "不合格查询或空查询示例")

### <a name="example-2---free-text-search"></a>示例 2 - 自定义文本搜索

自由格式查询（带或不带运算符）可用于模拟从自定义应用发送到 Azure 搜索的用户定义查询。 请注意，如果提供查询词或表达式，将用到搜索级别。 以下示例对自定义文本搜索进行了说明。

   ```Input
   Seattle apartment "Lake Washington" miele OR thermador appliance
   ```

   **结果**

   可以使用 Ctrl-F 在结果中搜索感兴趣的特定字词。

   ![自定义文本查询示例](./media/search-explorer/search-explorer-example-freetext.png "Free text query example")

### <a name="example-3---count-of-matching-documents"></a>示例 3 - 匹配文档计数 

添加“$count”以获取在索引中找到的匹配项数。 在空搜索中，计数是指索引中的文档总数。 在限定搜索中，计数是与查询输入匹配的文档数。

   ```Input1
   $count=true
   ```
   **结果**

   ![文档计数示例](./media/search-explorer/search-explorer-example-count.png "索引中匹配文档的计数")

### <a name="example-4---restrict-fields-in-search-results"></a>示例 4 - 限制搜索结果中的字段

添加“$select”将结果限制为显式命名的字段，以便在“搜索资源管理器”中获得可读性更强的输出。 若要保留搜索字符串“$count=true”，请在参数前面加上 &。 

   ```Input
   search=seattle condo&$select=listingId,beds,baths,description,street,city,price&$count=true
   ```

   **结果**

   ![限制字段示例](./media/search-explorer/search-explorer-example-selectfield.png "限制搜索结果中的字段")

### <a name="example-5---return-next-batch-of-results"></a>示例 5 - 返回下一批结果

Azure 搜索根据搜索级别返回前 50 个匹配项。 若要获取下一组匹配的文档，请附加“$top=100,&$skip=50”，这会将结果集增加为 100 个文档（默认值为 50，最大值为 1000）并跳过前 50 个文档。 前面提到，需要提供搜索条件，例如查询词或表达式，以便获得排列好的结果。 请注意，搜索分数随搜索结果中搜索的深入而降低。

   ```Input
   search=seattle condo&$select=listingId,beds,baths,description,street,city,price&$count=true&$top=100,&$skip=50
   ```

   **结果**

   ![批处理搜索结果](./media/search-explorer/search-explorer-example-topskip.png "返回下一批的搜索结果")

## <a name="filter-expressions-greater-than-less-than-equal-to"></a>筛选表达式（大于、小于、等于）

如果要指定精确条件搜索，而不是进行自定义文本搜索，请使用“$filter”参数。 此示例搜索大于 3 间的卧室：`search=seattle condo&$filter=beds gt 3&$count=true`

   ![筛选表达式](./media/search-explorer/search-explorer-example-filter.png "按条件筛选")

## <a name="order-by-expressions"></a>Order-by 表达式

添加“$orderby”按搜索分数之外的其他字段对结果进行排序。 可用于测试此功能的示例表达式如下 `search=seattle condo&$select=listingId,beds,price&$filter=beds gt 3&$count=true&$orderby=price asc`

   ![Orderby 表达式](./media/search-explorer/search-explorer-example-ordery.png "更改排序顺序")

“$filter”和“$orderby”表达式都是 OData 构造。 有关详细信息，请参阅 [Filter OData syntax](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)（筛选器 OData 语法）。

<a name="start-search-explorer"></a>

## <a name="how-to-start-search-explorer"></a>如何启动搜索资源管理器

1. 在 [Azure 门户](https://portal.azure.com)中，从仪表板打开搜索服务页，或者在服务列表中[查找服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。

2. 在“服务概述”页上，单击“搜索资源管理器”。

   ![门户中的搜索资源管理器命令](./media/search-explorer/search-explorer-cmd2.png "Search explorer command in porta")

3. 选择要查询的索引。

   ![选择要查询的索引](./media/search-explorer/search-explorer-changeindex-se2.png "选择索引")

4. （可选）设置 API 版本。 默认选择当前通用的 API 版本，但如果要使用的语法是基于某个版本的，可以选择预览版或旧版 API。

5. 选择索引和 API 版本后，在搜索栏中输入搜索词或完全限定的查询表达式，然后单击“搜索”执行搜索。

   ![输入搜索词并单击“搜索”](./media/search-explorer/search-explorer-query-string-example.png "Enter search terms and click Search")

在“搜索资源管理器”中搜索的提示：

+ 结果会作为详细的 JSON 文档返回，以便可以完整地查看文档的构建情况和内容。 可以使用示例中显示的查询表达式来限制返回的字段。

+ 文档由标记为在索引中“可检索”的所有字段构成。 若要在门户中查看索引属性，请单击搜索概述页面上“索引”列表中的“realestate-us-sample”。

+ 自由格式查询类似于在商用 Web 浏览器中输入的内容，可用于测试最终用户体验。 例如，假设有一个内置的房地产示例索引，可以输入“华盛顿湖西雅图公寓”，再使用 Ctrl-F 在搜索结果中查找字词。 

+ 必须使用 Azure 搜索支持的语法组织查询和筛查表达式。 默认为[简单语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)，但可选择使用[完整 Lucene](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 进行更强大的查询。 [筛选表达式](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)采用的是 OData 语法。


## <a name="next-steps"></a>后续步骤

以下资源提供更多的查询语法信息和示例。

 + [简单的查询语法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 
 + [Lucene 查询语法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 
 + [Lucene 查询示例](search-query-lucene-examples.md) 
 + [OData 筛选器表达式语法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) 