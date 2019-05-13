---
title: 有关在索引管道中调用认知服务 API 的 C# 教程 - Azure 搜索
description: 在本教程中，在用于数据提取和转换的 Azure 搜索索引中分步完成数据提取、自然语言和图像 AI 处理的示例。
manager: eladz
author: MarkHeff
services: search
ms.service: search
ms.devlang: NA
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: maheff
ms.openlocfilehash: 1b3353cae73bb5710dc9343f1d211266d15743a2
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153213"
---
# <a name="c-tutorial-call-cognitive-services-apis-in-an-azure-search-indexing-pipeline"></a>C# 教程：在 Azure 搜索索引管道中调用认知服务 API

本教程介绍使用认知技能在 Azure 搜索中扩充编程数据的机制。 技能由自然语言处理 (NLP) 和认知服务中的图像分析功能提供支持。 通过技能集组合和配置，可以提取图像或扫描的文档文件的文本和文本表示形式。 还可以检测语言、实体、关键短语等。 最终结果是 Azure 搜索索引中的丰富附加内容，由 AI 支持的索引编制管道创建。

在本教程中，你将使用 .NET SDK 执行以下任务：

> [!div class="checklist"]
> * 创建一个索引管道，用于扩充索引路由中的示例数据
> * 应用内置技能：光学字符识别、文本合并、语言检测、文本拆分、实体识别、关键短语提取
> * 了解如何通过在技能集中将输入映射到输出来链接技能
> * 执行请求并查看结果
> * 重置索引和索引器以进一步开发

输出是 Azure 搜索中的全文搜索索引。 可以使用[同义词](search-synonyms.md)、[评分配置文件](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)、[分析器](search-analyzers.md)和[筛选器](search-filters.md)等其他标准功能来增强索引。

本教程在免费服务中运行，但免费事务数目限制为每日 20 个文档。 若要在同一天多次运行本教程，请使用可以运行更多次的较小文件集。

> [!NOTE]
> 通过增大处理频率、添加更多文档或添加更多 AI 算法来扩大范围时，需要附加可计费的认知服务资源。 调用认知服务中的 API，以及在 Azure 搜索中的文档破解阶段提取图像时，会产生费用。 提取文档中的文本不会产生费用。
>
> 内置技能的执行将按现有[认知服务即用即付价格](https://azure.microsoft.com/pricing/details/cognitive-services/)计费。 图像提取定价如 [Azure 搜索定价页](https://go.microsoft.com/fwlink/?linkid=2042400)所述。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

本教程使用了以下服务、工具和数据。 

[创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可在本教程中使用免费服务。

[创建一个 Azure 存储帐户](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)，用于存储示例数据。

[示例数据](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4)包括不同类型的小型文件集。 

[安装 Visual Studio](https://visualstudio.microsoft.com/)，以用作 IDE。

## <a name="get-a-key-and-url"></a>获取密钥和 URL

必须有 Azure 搜索服务 URL 和访问密钥，才能与此服务交互。 搜索服务是使用这二者创建的，因此，如果向订阅添加了 Azure 搜索，则请按以下步骤获取必需信息：

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中获取 URL。 示例终结点可能类似于 `https://mydemo.search.windows.net`。

1. 在“设置” > “密钥”中，获取有关该服务的完全权限的管理员密钥。 有两个可交换的管理员密钥，为保证业务连续性而提供，以防需要滚动一个密钥。 可以在请求中使用主要或辅助密钥来添加、修改和删除对象。

   ![获取 HTTP 终结点和访问密钥](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

具有有效的密钥可以在发送请求的应用程序与处理请求的服务之间建立信任关系，这种信任关系以每个请求为基础。

## <a name="prepare-sample-data"></a>准备示例数据

扩充管道从 Azure 数据源提取数据。 源数据必须源自受支持的 [Azure 搜索索引器](search-indexer-overview.md)数据源类型。 Azure 表存储不支持认知搜索。 本演练使用 Blob 存储来展示多种内容类型。

1. [登录到 Azure 门户](https://portal.azure.com)，导航到你的 Azure 存储帐户，单击“Blob”，然后单击“+ 容器”。

1. [创建一个 Blob 容器](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)用于包含示例数据。 可将“公共访问级别”设为任何有效值。 本教程假定容器名称为“basic-demo-data-pr”。

1. 打开已创建的容器，并选择命令栏中的“上传”，以上传[样本数据](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4)。

   ![Azure Blob 存储中的源文件](./media/cognitive-search-quickstart-blob/sample-data.png)

1. 加载示例文件后，获取 Blob 存储的容器名称和连接字符串。 为此，可以在 Azure 门户中转到存储帐户，选择“访问密钥”，再复制“连接字符串”字段。

   存储连接字符串应是类似于以下示例的 URL：

      ```http
      DefaultEndpointsProtocol=https;AccountName=cogsrchdemostorage;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

可通过其他方式指定连接字符串，例如，提供共享访问签名。 若要详细了解数据源凭据，请参阅[为 Azure Blob 存储编制索引](search-howto-indexing-azure-blob-storage.md#Credentials)。

## <a name="set-up-your-environment"></a>设置环境

首先，打开 Visual Studio，并新建能在 .NET Core 上运行的控制台应用项目。

### <a name="install-nuget-packages"></a>安装 NuGet 包

[Azure 搜索 .NET SDK](https://aka.ms/search-sdk) 由一些客户端库组成。借助这些库，不仅可以管理索引、数据源、索引器和技能集，还能上传和管理文档并执行查询，所有这些操作都无需处理 HTTP 和 JSON 的详细信息。 这些客户端库全部作为 NuGet 包进行分发。

对于此项目，需要安装 `Microsoft.Azure.Search` NuGet 包的版本 9，以及最新的 `Microsoft.Extensions.Configuration.Json` NuGet 包。

使用 Visual Studio 中的包管理器控制台来安装 `Microsoft.Azure.Search` NuGet 包。 若要打开包管理器控制台，请依次选择“工具” > “NuGet 包管理器” > “包管理器控制台”。 若要获取要运行的命令，请导航到 [Microsoft.Azure.Search NuGet 包页](https://www.nuget.org/packages/Microsoft.Azure.Search)，选择版本 9，并复制包管理器命令。 在包管理器控制台中，运行此命令。

若要在 Visual Studio 中安装 `Microsoft.Extensions.Configuration.Json` NuGet 包，请依次选择“工具” > “Nuget 包管理器” > “管理解决方案的 Nuget 包...”。选择“浏览”，并搜索“`Microsoft.Extensions.Configuration.Json` NuGet 包”。 找到此包后，依次选择它和项目，确认版本是否是最新稳定版，再选择“安装”。

## <a name="add-azure-search-service-information"></a>添加 Azure 搜索服务信息

必须将搜索服务信息添加到项目中，才能连接到 Azure 搜索服务。 在“解决方案资源管理器”中，右键单击项目，并依次选择“添加” > “新项...”。 将文件命名为“`appsettings.json`”，并选择“添加”。 

必须将此文件添加到输出目录中。 为此，请右键单击“`appsettings.json`”，并选择“属性”。 将“复制到输出目录”的值更改为“复制较新文件”。

将以下 JSON 复制到新 JSON 文件中。

```json
{
  "SearchServiceName": "Put your search service name here",
  "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
  "SearchServiceQueryApiKey": "Put your query API key here",
  "AzureBlobConnectionString": "Put your Azure Blob connection string here",
}
```

添加搜索服务信息和 Blob 存储帐户信息。

可以从 Azure 门户中的搜索帐户页获取搜索服务信息。 帐户名位于主页上，选择“密钥”即可找到密钥。

若要获取 Blob 连接字符串，可以在 Azure 门户中转到存储帐户，选择“访问密钥”，再复制“连接字符串”字段。

## <a name="add-namespaces"></a>添加命名空间

本教程使用各种命名空间中的多种不同类型。 若要使用这些类型，请将以下代码添加到 `Program.cs` 中。

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Extensions.Configuration;
```

## <a name="create-a-client"></a>创建客户端

创建的 `SearchServiceClient` 类的实例。

```csharp
IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
IConfigurationRoot configuration = builder.Build();
SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);
```

`CreateSearchServiceClient` 使用应用程序的配置文件 (appsettings.json) 中存储的值创建新的 `SearchServiceClient`。

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
   string searchServiceName = configuration["SearchServiceName"];
   string adminApiKey = configuration["SearchServiceAdminApiKey"];

   SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
   return serviceClient;
}
```

> [!NOTE]
> `SearchServiceClient` 类管理与搜索服务的连接。 为了避免打开太多连接，应尝试在应用程序中共享 `SearchServiceClient` 的单个实例（如果可能）。 它的方法在启用此类共享时是线程安全的。
> 
> 

## <a name="create-a-data-source"></a>创建数据源

通过调用 `DataSource.AzureBlobStorage`，新建 `DataSource` 实例。 `DataSource.AzureBlobStorage` 要求必须指定数据源名称、连接字符串和 Blob 容器名称。

尽管本教程并未使用，但也定义了软删除策略，用于根据软删除列的值来识别已删除的 Blob。 以下策略将元数据属性 `IsDeleted` 的值为 `true`的 Blob 视为已删除。

```csharp
DataSource dataSource = DataSource.AzureBlobStorage(
    name: "demodata",
    storageConnectionString: configuration["AzureBlobConnectionString"],
    containerName: "basic-demo-data-pr",
    deletionDetectionPolicy: new SoftDeleteColumnDeletionDetectionPolicy(
        softDeleteColumnName: "IsDeleted",
        softDeleteMarkerValue: "true"),
    description: "Demo files to demonstrate cognitive search capabilities.");
```

至此，已初始化 `DataSource` 对象。是时候创建数据源了。 `SearchServiceClient` 具有 `DataSources` 属性。 此属性提供创建、列出、更新或删除 Azure 搜索数据源所需的全部方法。

为了让请求成功，此方法将返回已创建的数据源。 如果请求有问题（如参数无效），此方法将抛出异常。

```csharp
try
{
    serviceClient.DataSources.CreateOrUpdate(dataSource);
}
catch (Exception e)
{
    // Handle the exception
}
```

由于这是发出的第一个请求，请检查 Azure 门户，确认是否在 Azure 搜索中创建了数据源。 在搜索服务的仪表板页上，检查“数据源”磁贴中是否包含一个新项。 可能需要等待几分钟让门户页刷新。

  ![门户中的“数据源”磁贴](./media/cognitive-search-tutorial-blob/data-source-tile.png "门户中的“数据源”磁贴")

## <a name="create-a-skillset"></a>创建技能集

在此部分中，你将定义一组要应用于数据的扩充步骤。 每个扩充步骤称为“技能”，一组扩充步骤称为“技能集”。 本教程对技能集使用以下[预定义的认知技能](cognitive-search-predefined-skills.md)：

+ [光学字符识别](cognitive-search-skill-ocr.md)：用于识别图像文件中的印刷文本和手写文本。

+ [文本合并](cognitive-search-skill-textmerger.md)：用于将字段集合中的文本合并到单个字段中。

+ [语言检测](cognitive-search-skill-language-detection.md)：识别内容的语言。

+ [文本拆分](cognitive-search-skill-textsplit.md)：用于先将大段内容拆分为较小区块，再调用关键短语提取技能和实体识别技能。 关键短语提取和实体识别接受不超过 50,000 个字符的输入。 有几个示例文件需要拆分才能保留在此限制范围内。

+ [实体识别](cognitive-search-skill-entity-recognition.md)：从 Blob 容器中的内容提取组织名称。

+ [关键短语提取](cognitive-search-skill-keyphrases.md)：取出最关键的短语。

在初始处理期间，Azure 搜索会破译每个文档，以读取不同文件格式的内容。 从源文件中找到的文本将放入一个生成的 ```content``` 字段（每个文档对应一个字段）。 因此，请将输入设置为 ```"/document/content"```，以使用此文本。 

输出可以映射到索引、用作下游技能的输入，或者既映射到索引又用作输入（在语言代码中就是这样）。 在索引中，语言代码可用于筛选。 文本分析技能使用语言代码作为输入来告知有关断字的语言规则。

若要详细了解技能集的基础知识，请参阅[如何定义技能集](cognitive-search-defining-skillset.md)。

### <a name="ocr-skill"></a>OCR 技术

OCR 技能从图像中提取文本。 此技能假定存在“normalized_images”字段。 为了生成此字段，本教程稍后会将索引器定义中的 ```"imageAction"``` 配置设置为 ```"generateNormalizedImages"```。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "image",
    source: "/document/normalized_images/*"));

List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "text",
    targetName: "text"));

OcrSkill ocrSkill = new OcrSkill(
    description: "Extract text (plain and structured) from image",
    context: "/document/normalized_images/*",
    inputs: inputMappings,
    outputs: outputMappings,
    defaultLanguageCode: OcrSkillLanguage.En,
    shouldDetectOrientation: true);
```

### <a name="merge-skill"></a>合并技能

在此部分中，你将创建合并技能，用于将文档内容字段与 OCR 技能生成的文本合并。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "text",
    source: "/document/content"));
inputMappings.Add(new InputFieldMappingEntry(
    name: "itemsToInsert",
    source: "/document/normalized_images/*/text"));
inputMappings.Add(new InputFieldMappingEntry(
    name: "offsets",
    source: "/document/normalized_images/*/contentOffset"));

List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "mergedText",
    targetName: "merged_text"));

MergeSkill mergeSkill = new MergeSkill(
    description: "Create merged_text which includes all the textual representation of each image inserted at the right location in the content field.",
    context: "/document",
    inputs: inputMappings,
    outputs: outputMappings,
    insertPreTag: " ",
    insertPostTag: " ");
```

### <a name="language-detection-skill"></a>语言检测技能

语言检测技能检测输入文本的语言，并报告在请求中提交的每个文档的单一语言代码。 我们会将语言检测技能的输出用作文本拆分技能的输入的一部分。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "text",
    source: "/document/merged_text"));

List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "languageCode",
    targetName: "languageCode"));

LanguageDetectionSkill languageDetectionSkill = new LanguageDetectionSkill(
    description: "Detect the language used in the document",
    context: "/document",
    inputs: inputMappings,
    outputs: outputMappings);
```

### <a name="text-split-skill"></a>文本拆分技能

下面的拆分技能按页面拆分文本，并将页面长度限制为 `String.Length` 度量的 4,000 个字符。 此算法会尝试将文本拆分为最大为 `maximumPageLength` 的区块。 在下面的示例中，此算法会尽可能在句子边界断开句子，所以区块大小可能略小于 `maximumPageLength`。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "text",
    source: "/document/merged_text"));
inputMappings.Add(new InputFieldMappingEntry(
    name: "languageCode",
    source: "/document/languageCode"));

List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "textItems",
    targetName: "pages"));

SplitSkill splitSkill = new SplitSkill(
    description: "Split content into pages",
    context: "/document",
    inputs: inputMappings,
    outputs: outputMappings,
    textSplitMode: TextSplitMode.Pages,
    maximumPageLength: 4000);
```

### <a name="entity-recognition-skill"></a>实体识别技能

设置此 `EntityRecognitionSkill` 实例是为了识别类别类型 `organization`。 此外，实体识别技能还可以识别类别类型 `person` 和 `location`。

请注意，“context”字段设置为包含星号的 ```"/document/pages/*"```；也就是说，将对 ```"/document/pages"``` 下的每个页面都调用扩充步骤。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "text",
    source: "/document/pages/*"));
    
List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "organizations",
    targetName: "organizations"));

List<EntityCategory> entityCategory = new List<EntityCategory>();
entityCategory.Add(EntityCategory.Organization);
    
EntityRecognitionSkill entityRecognitionSkill = new EntityRecognitionSkill(
    description: "Recognize organizations",
    context: "/document/pages/*",
    inputs: inputMappings,
    outputs: outputMappings,
    categories: entityCategory,
    defaultLanguageCode: EntityRecognitionSkillLanguage.En);
```

### <a name="key-phrase-extraction-skill"></a>关键短语提取技能

与刚刚创建的 `EntityRecognitionSkill` 实例一样，关键短语提取技能对文档的各个页面都调用。

```csharp
List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
inputMappings.Add(new InputFieldMappingEntry(
    name: "text",
    source: "/document/pages/*"));
inputMappings.Add(new InputFieldMappingEntry(
    name: "languageCode",
    source: "/document/languageCode"));

List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
outputMappings.Add(new OutputFieldMappingEntry(
    name: "keyPhrases",
    targetName: "keyPhrases"));

KeyPhraseExtractionSkill keyPhraseExtractionSkill = new KeyPhraseExtractionSkill(
    description: "Extract the key phrases",
    context: "/document/pages/*",
    inputs: inputMappings,
    outputs: outputMappings);
```

### <a name="build-and-create-the-skillset"></a>生成并创建技能集

使用已创建的技能来生成 `Skillset`。

```csharp
List<Skill> skills = new List<Skill>();
skills.Add(ocrSkill);
skills.Add(mergeSkill);
skills.Add(languageDetectionSkill);
skills.Add(splitSkill);
skills.Add(entityRecognitionSkill);
skills.Add(keyPhraseExtractionSkill);

Skillset skillset = new Skillset(
    name: "demoskillset",
    description: "Demo skillset",
    skills: skills);
```

在搜索服务中创建技能集。

```csharp
try
{
    serviceClient.Skillsets.CreateOrUpdate(skillset);
}
catch (Exception e)
{
    // Handle exception
}
```

## <a name="create-an-index"></a>创建索引

本部分通过指定要在可搜索索引中包含的字段以及每个字段的搜索特性，来定义索引架构。 字段具有某种类型，并可以采用特性来确定字段的使用方式（可搜索、可排序，等等）。 索引中的字段名称不一定要与源中的字段名称完全匹配。 在稍后的步骤中，我们将在索引器中添加字段映射以连接源-目标字段。 针对此步骤，请使用搜索应用程序相关的字段命名约定来定义索引。

本演练使用以下字段和字段类型：

| field-names: | `id`       | 内容   | languageCode | keyPhrases         | organizations     |
|--------------|----------|-------|----------|--------------------|-------------------|
| field-types: | Edm.String|Edm.String| Edm.String| List<Edm.String>  | List<Edm.String>  |


### <a name="create-demoindex-class"></a>创建 DemoIndex 类

此索引的字段是使用模型类进行定义。 模型类的每个属性都具有一些特性，这些特性决定了相应索引字段的与搜索相关的行为。 

接下来，将把模型类添加到新 C# 文件中。 右键单击项目，并依次选择“添加” > “新项...”。选择“类”，并将文件命名为“`DemoIndex.cs`”，再选择“添加”。

请务必指明要使用 `Microsoft.Azure.Search` 和 `Microsoft.Azure.Search.Models` 命名空间中的类型。

```csharp
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
```

将下面的模型类定义添加到 `DemoIndex.cs` 中，并将它添加到要在其中创建索引的同一命名空间中。

```csharp
// The SerializePropertyNamesAsCamelCase attribute is defined in the Azure Search .NET SDK.
// It ensures that Pascal-case property names in the model class are mapped to camel-case
// field names in the index.
[SerializePropertyNamesAsCamelCase]
public class DemoIndex
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsSearchable, IsSortable]
    public string Id { get; set; }

    [IsSearchable]
    public string Content { get; set; }

    [IsSearchable]
    public string LanguageCode { get; set; }

    [IsSearchable]
    public string[] KeyPhrases { get; set; }

    [IsSearchable]
    public string[] Organizations { get; set; }
}
```

至此，已定义模型类。返回到 `Program.cs`，可以轻松创建索引定义了。 此索引的名称为“demoindex”。

```csharp
var index = new Index()
{
    Name = "demoindex",
    Fields = FieldBuilder.BuildForType<DemoIndex>()
};
```

在测试期间，你可能会发现要多次尝试创建索引。 因此，请先检查要创建的索引是否已存在，再尝试创建索引。

```csharp
try
{
    bool exists = serviceClient.Indexes.Exists(index.Name);

    if (exists)
    {
        serviceClient.Indexes.Delete(index.Name);
    }

    serviceClient.Indexes.Create(index);
}
catch (Exception e)
{
    // Handle exception
}
```

若要详细了解如何定义索引，请参阅[创建索引（Azure 搜索 REST API）](https://docs.microsoft.com/rest/api/searchservice/create-index)。

## <a name="create-an-indexer-map-fields-and-execute-transformations"></a>创建索引器，映射字段，并执行转换

到目前为止，我们已创建数据源、技能集和索引。 这三个组件属于某个[索引器](search-indexer-overview.md)，该索引器将每个片段一同提取到单个多阶段操作。 若要在索引器中将这些组件捆绑在一起，必须定义字段映射。

+ 先处理 fieldMapping，再处理技能集；将数据源中的源字段映射到索引中的目标字段。 如果两端的字段名称和类型相同，则无需映射。

+ 先处理技能集，再处理 outputFieldMapping；引用不存在的 sourceFieldName，直到文档破解或扩充功能创建了它们。 targetFieldName 是索引中的字段。

除了将输入挂钩到输出外，还可以使用字段映射来平展数据结构。 有关详细信息，请参阅[如何将扩充字段映射到可搜索索引](cognitive-search-output-field-mapping.md)。

```csharp
IDictionary<string, object> config = new Dictionary<string, object>();
config.Add(
    key: "dataToExtract",
    value: "contentAndMetadata");
config.Add(
    key: "imageAction",
    value: "generateNormalizedImages");

List<FieldMapping> fieldMappings = new List<FieldMapping>();
fieldMappings.Add(new FieldMapping(
    sourceFieldName: "metadata_storage_path",
    targetFieldName: "id",
    mappingFunction: new FieldMappingFunction(
        name: "base64Encode")));
fieldMappings.Add(new FieldMapping(
    sourceFieldName: "content",
    targetFieldName: "content"));

List<FieldMapping> outputMappings = new List<FieldMapping>();
outputMappings.Add(new FieldMapping(
    sourceFieldName: "/document/pages/*/organizations/*",
    targetFieldName: "organizations"));
outputMappings.Add(new FieldMapping(
    sourceFieldName: "/document/pages/*/keyPhrases/*",
    targetFieldName: "keyPhrases"));
outputMappings.Add(new FieldMapping(
    sourceFieldName: "/document/languageCode",
    targetFieldName: "languageCode"));

Indexer indexer = new Indexer(
    name: "demoindexer",
    dataSourceName: dataSource.Name,
    targetIndexName: index.Name,
    description: "Demo Indexer",
    skillsetName: skillSet.Name,
    parameters: new IndexingParameters(
        maxFailedItems: -1,
        maxFailedItemsPerBatch: -1,
        configuration: config),
    fieldMappings: fieldMappings,
    outputFieldMappings: outputMappings);

try
{
    bool exists = serviceClient.Indexers.Exists(indexer.Name);

    if (exists)
    {
        serviceClient.Indexers.Delete(indexer.Name);
    }

    serviceClient.Indexers.Create(indexer);
}
catch (Exception e)
{
    // Handle exception
}
```

创建索引器预计需要一段时间才能完成。 即使数据集较小，分析技能也会消耗大量的计算资源。 某些技能（例如图像分析）会长时间运行。

> [!TIP]
> 创建索引器会调用管道。 如果访问数据、映射输入和输出或操作顺序出现问题，此阶段会显示这些问题。

### <a name="explore-creating-the-indexer"></a>探索如何创建索引器

代码将 ```"maxFailedItems"``` 设置为 -1，指示索引引擎在数据导入期间忽略错误。 此设置非常有用，因为演示数据源中的文档很少。 对于更大的数据源，请将值设置为大于 0。

另请注意，```"dataToExtract"``` 设置为 ```"contentAndMetadata"```。 该语句告知索引器从不同的文件格式以及与每个文件相关的元数据中自动提取内容。

提取内容后，可以设置 `imageAction`，以从数据源中的图像提取文本。 将设置为 ```"generateNormalizedImages"``` 配置的 ```"imageAction"``` 与 OCR 技能和文本合并技能相结合，指示索引器从图像中提取文本（例如，交通停车标志中的“停”一词），并将它嵌入为内容字段的一部分。 此行为将应用到文档中嵌入的图像（例如 PDF 中的图像），以及数据源（例如 JPG 文件）中的图像。

## <a name="check-indexer-status"></a>检查索引器状态

定义索引器后，提交请求时会自动运行索引器。 根据定义的认知技能，索引编制花费的时间可能会超出预期。 若要确定索引器是否仍在运行，请使用 `GetStatus` 方法。

```csharp
try
{
    IndexerExecutionInfo demoIndexerExecutionInfo = serviceClient.Indexers.GetStatus(indexer.Name);

    switch (demoIndexerExecutionInfo.Status)
    {
        case IndexerStatus.Error:
            Console.WriteLine("Indexer has error status");
            break;
        case IndexerStatus.Running:
            Console.WriteLine("Indexer is running");
            break;
        case IndexerStatus.Unknown:
            Console.WriteLine("Indexer status is unknown");
            break;
        default:
            Console.WriteLine("No indexer information");
            break;
    }
}
catch (Exception e)
{
    // Handle exception
}
```

`IndexerExecutionInfo` 表示索引器的当前状态和执行历史记录。

处理某些源文件和技能的组合时经常会出现警告，这并不总是意味着出现了问题。 在本教程中，警告是良性的（例如，JPEG 文件中没有文本输入）。
 
## <a name="verify-content"></a>验证内容

索引编制完成后，可以运行查询来返回各个字段的内容。 默认情况下，Azure 搜索返回前 50 条结果。 由于样本数据较小，因此使用默认设置即可正常操作。 但是，在处理较大的数据集时，可能需要在查询字符串中包含参数来返回更多结果。 有关说明，请参阅[如何将 Azure 搜索中的结果分页](search-pagination-page-layout.md)。

作为验证步骤，请查询所有字段的索引。

```csharp
DocumentSearchResult<DemoIndex> results;

ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

try
{
    results = indexClientForQueries.Documents.Search<DemoIndex>("*");
}
catch (Exception e)
{
    // Handle exception
}
```

`CreateSearchIndexClient` 使用应用程序的配置文件 (appsettings.json) 中存储的值创建新的 `SearchIndexClient`。 请注意，使用的是搜索服务查询 API 密钥而不是管理员密钥。

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
   string searchServiceName = configuration["SearchServiceName"];
   string queryApiKey = configuration["SearchServiceQueryApiKey"];

   SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "demoindex", new SearchCredentials(queryApiKey));
   return indexClient;
}
```

输出是索引架构，其中包含每个字段的名称、类型和特性。

针对 `"*"` 提交另一个查询，以返回单个字段的所有内容，例如 `organizations`。

```csharp
SearchParameters parameters =
    new SearchParameters
    {
        Select = new[] { "organizations" }
    };

try
{
    results = indexClientForQueries.Documents.Search<DemoIndex>("*", parameters);
}
catch (Exception e)
{
    // Handle exception
}
```

针对本练习中的其他字段（content、languageCode、keyPhrases 和 organizations）重复上述步骤。 可以使用逗号分隔列表通过 `$select` 返回多个字段。

<a name="reset"></a>

## <a name="reset-and-rerun"></a>重置并重新运行

在开发的前期试验阶段，设计迭代的最实用方法是，删除 Azure 搜索中的对象，并允许代码重新生成它们。 资源名称是唯一的。 删除某个对象后，可以使用相同的名称重新创建它。

本教程涵盖了如何检查是否有现有索引器和索引，并删除它们（若有），以便能够重新运行代码。

也可以使用门户来删除索引、索引器和技能集。

随着代码的成熟，可能需要优化重新生成策略。 有关详细信息，请参阅[如何重新生成索引](search-howto-reindex.md)。

## <a name="takeaways"></a>要点

本教程演示了通过创建组件部件（数据源、技能集、索引和索引器）生成扩充索引管道的基本步骤。

其中介绍了[预定义的技能集](cognitive-search-predefined-skills.md)、技能集定义，以及通过输入和输出将技能链接在一起的机制。 此外，还提到需要使用索引器定义中的 `outputFieldMappings`，将管道中的扩充值路由到 Azure 搜索服务中的可搜索索引。

最后，介绍了如何测试结果并重置系统以进一步迭代。 本教程提到，针对索引发出查询会返回扩充的索引管道创建的输出。 此外，本教程还介绍了如何检查索引器状态，以及在重新运行管道之前要删除的对象。

## <a name="clean-up-resources"></a>清理资源

完成本教程后，最快的清理方式是删除包含 Azure 搜索服务和 Azure Blob 服务的资源组。 假设已将这两个服务放在同一个组中，则删除该资源组会永久删除其中的所有内容，包括在本教程中创建的服务和任何存储内容。 在门户中，资源组名称显示在每个服务的“概述”页上。

## <a name="next-steps"></a>后续步骤

使用自定义技能自定义或扩展管道。 创建自定义技能并将其添加到技能集可以载入自己编写的文本或图像分析代码。

> [!div class="nextstepaction"]
> [示例：创建自定义技能](cognitive-search-create-custom-skill-example.md)
