---
title: 教程：在 C# 代码示例中为 Azure SQL 数据库中的数据编制索引 - Azure 搜索
description: 本 C# 代码示例演示如何连接到 Azure SQL 数据库、提取可搜索的数据，并将其加载到 Azure 搜索索引中。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: na
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: c9413fb78bbb92618818ac4311ab1af8ad65576d
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025082"
---
# <a name="c-tutorial-crawl-an-azure-sql-database-using-azure-search-indexers"></a>C# 教程：使用 Azure 搜索索引器搜索 Azure SQL 数据库

了解如何配置索引器来从示例 Azure SQL 数据库中提取可搜索的数据。 [索引器](search-indexer-overview.md)是 Azure 搜索组件，用于对外部数据源进行爬网，使用内容来填充[搜索索引](search-what-is-an-index.md)。 在所有索引器中，Azure SQL 数据库索引器运用最为广泛。 

熟练掌握索引器的配置很有用，因为这样可以减少需编写和维护的代码量。 可以将索引器附加到数据源，让索引器提取数据并将其插入索引中，然后即可选择定期按计划运行索引器，以便拾取基础源中所做的更改，不必准备和推送符合架构的 JSON 数据集。

在本教程中，使用 [Azure 搜索 .NET 客户端库](https://aka.ms/search-sdk)和 .NET Core 控制台应用程序来执行以下任务：

> [!div class="checklist"]
> * 将搜索服务信息添加到应用程序设置
> * 在 Azure SQL 数据库中准备外部数据集 
> * 查看示例代码中的索引和索引器定义
> * 运行索引器代码以导入数据
> * 搜索索引
> * 查看门户中的索引器配置

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

本快速入门使用以下服务、工具和数据。 

[创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可在本教程中使用免费服务。

[Azure SQL 数据库](https://azure.microsoft.com/services/sql-database/)存储索引器所用的外部数据源。 示例解决方案提供了一个用于创建表的 SQL 数据文件。 本教程提供了创建服务和数据库的步骤。

可以使用任意版本的 [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) 来运行示例解决方案。 示例代码和说明已在免费社区版上进行了测试。

[Azure-Samples/search-dotnet-getting-started](https://github.com/Azure-Samples/search-dotnet-getting-started) 提供 Azure 示例 GitHub 存储库中的示例解决方案。 请下载并提取该解决方案。 默认情况下，解决方案是只读的。 右键单击解决方案并清除只读属性，以便可以修改文件。

> [!Note]
> 如果使用免费的 Azure 搜索服务，则仅限使用三个索引、三个索引器和三个数据源。 本教程每样创建一个。 请确保服务中有空间来接受新资源。

## <a name="get-a-key-and-url"></a>获取密钥和 URL

REST 调用需要在每个请求中使用服务 URL 和访问密钥。 搜索服务是使用这二者创建的，因此，如果向订阅添加了 Azure 搜索，则请按以下步骤获取必需信息：

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中获取 URL。 示例终结点可能类似于 `https://mydemo.search.windows.net`。

1. 在“设置” > “密钥”中，获取有关该服务的完全权限的管理员密钥。 有两个可交换的管理员密钥，为保证业务连续性而提供，以防需要滚动一个密钥。 可以在请求中使用主要或辅助密钥来添加、修改和删除对象。

![获取 HTTP 终结点和访问密钥](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

所有请求对发送到服务的每个请求都需要 API 密钥。 具有有效的密钥可以在发送请求的应用程序与处理请求的服务之间建立信任关系，这种信任关系以每个请求为基础。

## <a name="set-up-connections"></a>设置连接
所需服务的连接信息在解决方案的 **appsettings.json** 文件中指定。 

1. 在 Visual Studio 中打开 **DotNetHowToIndexers.sln** 文件。

1. 在解决方案资源管理器中打开 **appsettings.json**，以便可以填充每项设置。  

现在，可以使用 Azure 搜索服务的 URL 和管理密钥填充前两个条目。 如果终结点为 `https://mydemo.search.windows.net`，则要提供的服务名称为 `mydemo`。

```json
{
  "SearchServiceName": "Put your search service name here",
  "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
  "AzureSqlConnectionString": "Put your Azure SQL database connection string here",
}
```

最后一个条目需是现有的数据库。 将在下一步骤中创建它。

## <a name="prepare-sample-data"></a>准备示例数据

在此步骤中，请创建一个可供索引器爬网的外部数据源。 可以使用 Azure 门户和示例中的 hotels.sql 文件，在 Azure SQL 数据库中创建数据集。 Azure 搜索使用平展行集，例如从视图或查询生成的行集。 示例解决方案中的 SQL 文件创建并填充单个表。

以下练习假定没有现成的服务器或数据库，因此会指导你在步骤 2 中创建这两项。 （可选）如果有现成的资源，可以向其添加 hotels 表，从步骤 4 开始。

1. [登录到 Azure 门户](https://portal.azure.com/)。 

2. 找到或创建一个 **Azure SQL 数据库**，用于创建数据库、服务器和资源组。 可以使用默认设置和最低级别的定价层。 创建服务器的一大优势是可以指定管理员用户名和密码，这是在后面的步骤中创建和加载表所必需的。

   ![“新建数据库”页](./media/search-indexer-tutorial/indexer-new-sqldb.png)

3. 单击“创建”，部署新的服务器和数据库。 等待服务器和数据库部署。

4. 打开新数据库的“SQL 数据库”页（如果尚未打开）。 资源名称应显示为“SQL 数据库”而非“SQL Server”。

   ![“SQL 数据库”页](./media/search-indexer-tutorial/hotels-db.png)

4. 在导航窗格中，单击“查询编辑器(预览版)”。

5. 单击“登录”，输入服务器管理员的用户名和密码。

6. 单击“打开查询”，导航到 hotels.sql 所在位置。 

7. 选择该文件，然后单击“打开”。 此脚本应与以下屏幕截图类似：

   ![SQL 脚本](./media/search-indexer-tutorial/sql-script.png)

8. 单击“运行”以执行查询。 在“结果”窗格中，应看到一条 3 行内容的消息，指示查询成功。

9. 若要从此表返回一个行集，可执行以下充当验证步骤的查询：

    ```sql
    SELECT HotelId, HotelName, Tags FROM Hotels
    ```
    原型查询 `SELECT * FROM Hotels` 在查询编辑器中不起作用。 示例数据包括“位置”字段的地理坐标，目前在编辑器中无法处理。 如需要查询的其他列的列表，可执行以下语句：`SELECT * FROM sys.columns WHERE object_id = OBJECT_ID('dbo.Hotels')`

10. 有了外部数据集以后，请复制数据库的 ADO.NET 连接字符串。 在数据库的“SQL 数据库”页，转到“设置” > “连接字符串”，然后复制 ADO.NET 连接字符串。
 
    ADO.NET 连接字符串类似于以下示例，经修改后可使用有效的数据库名称、用户名和密码。

    ```sql
    Server=tcp:hotels-db.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
    ```
11. 在 Visual Studio 中将连接字符串粘贴到“AzureSqlConnectionString”中，作为 **appsettings.json** 文件中的第三个条目。

    ```json
    {
      "SearchServiceName": "<placeholder-Azure-Search-service-name>",
      "SearchServiceAdminApiKey": "<placeholder-admin-key-for-Azure-Search>",
      "AzureSqlConnectionString": "Server=tcp:hotels-db.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security  Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
    }
    ```

## <a name="understand-the-code"></a>了解代码

指定数据和配置设置后，可以生成并运行 **DotNetHowToIndexers.sln** 中的示例程序。 执行该操作之前，请抽时间了解此示例的索引和索引器定义。 相关代码在两个文件中：

  + **hotel.cs**，包含的架构用于定义索引
  + **Program.cs**，包含的函数用于创建和管理服务中的结构

### <a name="in-hotelcs"></a>在 hotel.cs 中

索引架构定义字段集合，包含的属性用于指定允许的操作，例如字段是否可以进行全文搜索、筛选或排序，如以下针对 HotelName 的字段定义所示。 

```csharp
. . . 
[IsSearchable, IsFilterable, IsSortable]
public string HotelName { get; set; }
. . .
```

架构还可以包含其他元素，包括用于提高搜索得分的计分配置文件、自定义分析器和其他构造。 但对于我们来说，架构只需进行稀疏定义，只包含在示例数据集中发现的字段。

在本教程中，索引器从一个数据源拉取数据。 在实践中，可以将多个索引器附加到同一索引，从多个数据源创建合并的可搜索索引。 可以使用同一索引-索引器对，仅数据源有所不同，也可以使用一个索引加多个索引器和数据源的组合，具体取决于你对灵活性的要求如何。

### <a name="in-programcs"></a>在 Program.cs 中

主程序包含用于创建客户端、索引、数据源和索引器的逻辑。 此代码检查是否存在同一名称的资源，如果存在则会将其删除，所依据的假设是此程序可能多次运行。

数据源对象是使用特定于 Azure SQL 数据库资源的设置配置的，这些设置包括用来利用 Azure SQL 内置[更改检测功能](https://docs.microsoft.com/sql/relational-databases/track-changes/about-change-tracking-sql-server)的[增量索引](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#capture-new-changed-and-deleted-rows)。 Azure SQL 中的 hotels 演示数据库包含一个名为 **IsDeleted** 的“软删除”列。 如果在数据库中将此列设置为 true，则索引器会从 Azure 搜索索引中删除相应的文档。

  ```csharp
  Console.WriteLine("Creating data source...");

  DataSource dataSource = DataSource.AzureSql(
      name: "azure-sql",
      sqlConnectionString: configuration["AzureSQLConnectionString"],
      tableOrViewName: "hotels",
      deletionDetectionPolicy: new SoftDeleteColumnDeletionDetectionPolicy(
          softDeleteColumnName: "IsDeleted",
          softDeleteMarkerValue: "true"));
  dataSource.DataChangeDetectionPolicy = new SqlIntegratedChangeTrackingPolicy();

  searchService.DataSources.CreateOrUpdateAsync(dataSource).Wait();
  ```

索引器对象与平台无关，无论源是什么，配置、计划和调用都是相同的。 此示例索引器包含一个计划和一个用于清除索引器历史记录的重置选项，并调用一个方法来创建和立即运行索引器。

  ```csharp
  Console.WriteLine("Creating Azure SQL indexer...");
  Indexer indexer = new Indexer(
      name: "azure-sql-indexer",
      dataSourceName: dataSource.Name,
      targetIndexName: index.Name,
      schedule: new IndexingSchedule(TimeSpan.FromDays(1)));
  // Indexers contain metadata about how much they have already indexed
  // If we already ran the sample, the indexer will remember that it already
  // indexed the sample data and not run again
  // To avoid this, reset the indexer if it exists
  exists = await searchService.Indexers.ExistsAsync(indexer.Name);
  if (exists)
  {
      await searchService.Indexers.ResetAsync(indexer.Name);
  }

  await searchService.Indexers.CreateOrUpdateAsync(indexer);

  // We created the indexer with a schedule, but we also
  // want to run it immediately
  Console.WriteLine("Running Azure SQL indexer...");

  try
  {
      await searchService.Indexers.RunAsync(indexer.Name);
  }
  catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
  {
      Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
  }
  ```



## <a name="run-the-indexer"></a>运行索引器

在此步骤中，请编译并运行程序。 

1. 在解决方案资源管理器中右键单击“DotNetHowToIndexers”，然后选择“生成”。
2. 再次右键单击“DotNetHowToIndexers”，接着单击“调试” > “启动新实例”。

程序在调试模式下执行。 控制台窗口报告每项操作的状态。

  ![SQL 脚本](./media/search-indexer-tutorial/console-output.png)

代码在 Visual Studio 中以本地方式运行，连接到 Azure 中的搜索服务，后者又会使用连接字符串连接到 Azure SQL 数据库并检索数据集。 由于需要执行如此多的操作，因此存在多个可能的故障点。不过，如果出现错误，请首先查看以下情况：

+ 在本教程中，提供的搜索服务连接信息只能使用服务名称。 如果输入了完整 URL，则操作会停在索引创建阶段，出现“无法连接”错误。

+ **appsettings.json** 中的数据库连接信息。 它应该是从门户获得的 ADO.NET 连接字符串，经修改后包括了适用于数据库的用户名和密码。 用户帐户必须有权检索数据。

+ 资源限制。 回想一下，共享（免费）服务仅限 3 个索引、索引器和数据源。 达到最大限制的服务不能创建新的对象。

## <a name="search-the-index"></a>搜索索引 

在 Azure 门户的搜索服务“概览”页，单击顶部的“搜索浏览器”，提交一些有关新索引的查询。

1. 单击顶部的“更改索引”，选择 hotels 索引。

2. 单击“搜索”按钮，发出一个空的搜索指令。 

   索引中的三个条目以 JSON 文档的形式返回。 搜索浏览器返回 JSON 格式的文档，方便你查看整个结构。

3. 接下来，输入搜索字符串：`search=river&$count=true`。 

   此查询调用 `river` 一词的全文搜索，结果包含匹配文档的计数。 在索引很大且文档成千上万甚至数百万的测试方案中，返回匹配文档的计数很有用。 在本示例中，只有一个文档与查询匹配。

4. 最后，输入一个搜索字符串，将 JSON 输出限制为感兴趣的字段：`search=river&$count=true&$select=hotelId, baseRate, description`。 

   查询响应范围缩小为选定字段，使输出更简洁。

## <a name="view-indexer-configuration"></a>查看索引器配置

所有索引器（包括刚刚以编程方式创建的）都列在门户中。 可以打开一个索引器定义，然后查看其数据源，也可以配置一个刷新计划来拾取新行和已更改行。

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中，单击“索引”、“索引器”和“数据源”对应的链接。
3. 选择单个对象以查看或修改配置设置。

   ![索引器和数据源磁贴](./media/search-indexer-tutorial/tiles-portal.png)

## <a name="clean-up-resources"></a>清理资源

完成本教程后，最快的清理方式是删除包含 Azure 搜索服务的资源组。 现在，可以删除资源组以永久删除其中的所有内容。 在门户中，资源组名称显示在 Azure 搜索服务的“概述”页上。

## <a name="next-steps"></a>后续步骤

可以将 AI 扩充算法附加到索引器管道中。 下一步，继续学习下面的教程。

> [!div class="nextstepaction"]
> [为 Azure Blob 存储中的文档编制索引](search-howto-indexing-azure-blob-storage.md)