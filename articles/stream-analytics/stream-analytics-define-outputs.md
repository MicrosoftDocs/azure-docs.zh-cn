---
title: 了解 Azure 流分析的输出
description: 本文介绍 Azure 流分析提供的数据输出选项，包括用于分析结果的 Power BI。
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 3/25/2019
ms.custom: seodec18
ms.openlocfilehash: 3fab76613bb992b29ceeef12cf5f410c5c3b208d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205534"
---
# <a name="understand-outputs-from-azure-stream-analytics"></a>了解 Azure 流分析的输出
本文介绍可用于 Azure Stream Analytics 作业输出的类型。 输出可帮助存储和保存流分析作业的结果。 通过使用输出数据，您可以执行更多业务分析和数据仓库的数据。

在设计时在 Stream Analytics 查询，请参阅输出的名称通过[INTO 子句](https://msdn.microsoft.com/azure/stream-analytics/reference/into-azure-stream-analytics)。 可以使用单个输出每个作业或每个流式处理作业 （如果需要使用它们） 通过提供在查询中的多个 INTO 子句的多个输出。

若要创建、 编辑和测试 Stream Analytics 作业输出，可以使用[Azure 门户](stream-analytics-quick-create-portal.md#configure-job-output)， [Azure PowerShell](stream-analytics-quick-create-powershell.md#configure-output-to-the-job)， [.NET API](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.streamanalytics.ioutputsoperations?view=azure-dotnet)， [REST API](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-output)，并[Visual Studio](stream-analytics-quick-create-vs.md)。

一些输出类型支持[分区](#partitioning)。 [输出的批大小](#output-batch-size)更改以优化吞吐量。


## <a name="azure-data-lake-store"></a>Azure Data Lake Store
流分析支持 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)。 Azure Data Lake Store 是大数据分析工作负荷的企业范围的超大规模存储库。 可以使用 Data Lake Store 来存储任何大小、 类型和引入速度进行操作和探索性分析的数据。 Stream Analytics 必须要有权访问 Data Lake Store。

流分析中的 Azure Data Lake Store 输出当前不可在 Azure 中国（世纪互联）和 Azure 德国 (T-Systems International) 区域中使用。

### <a name="authorize-an-azure-data-lake-store-account"></a>授权 Azure Data Lake Store 帐户

1. 当选择 Data Lake Store 作为 Azure 门户中的输出时，系统会提示向现有 Data Lake Store 实例的连接进行授权。

   ![授权连接到 Data Lake Store](./media/stream-analytics-define-outputs/06-stream-analytics-define-outputs.png)

2. 如果你已有 Data Lake Store 访问权限，请选择**立即授权**。 弹出一个页面，并指示**将重定向到授权**。 授权成功后，你会看到与页面，可用于配置 Data Lake Store 输出。

3. Data Lake Store 帐户进行身份验证后，你可以为自己的 Data Lake Store 输出配置属性。

   ![将 Data Lake Store 定义为流分析输出](./media/stream-analytics-define-outputs/07-stream-analytics-define-outputs.png)

下表列出了属性名称及其说明，若要配置 Data Lake Store 输出。   

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 | 在查询中用于将查询输出定向到 Data Lake Store 的友好名称。 |
| 帐户名 | 要将发送输出的 Data Lake Store 帐户的名称。 您会看到在订阅中可用的 Data Lake Store 帐户的下拉列表。 |
| 路径前缀模式 | 用于写入你指定的 Data Lake Store 帐户中的文件的文件路径。 可以指定一个或多个实例的 {date} 和 {time} 变量：<br /><ul><li>示例 1：folder1/logs/{date}/{time}</li><li>示例 2：folder1/logs/{date}</li></ul><br />创建的文件夹结构的时间戳遵循 UTC 并不是本地时间。<br /><br />如果文件路径模式不包含尾部反斜杠 （/），则文件路径中的最后一个模式视为文件名前缀。 <br /><br />在这些情况下会创建新文件：<ul><li>在输出架构中进行更改</li><li>外部或内部重启作业</li></ul> |
| 日期格式 | 可选。 如果在前缀路径中使用日期令牌，可以选择组织文件所采用的日期格式。 示例：YYYY/MM/DD |
|时间格式 | 可选。 如果在前缀路径中使用时间令牌，可以指定组织文件所采用的时间格式。 目前唯一支持的值是 HH。 |
| 事件序列化格式 | 输出数据的序列化格式。 支持 JSON、CSV 和 Avro。|
| 编码 | 如果您使用 CSV 或 JSON 格式，则必须指定一种编码。 目前，UTF-8 是仅受支持的编码格式。|
| 分隔符 | 仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。|
| 格式 | 仅适用于 JSON 序列化。 **分隔行**指定输出的格式由一个新行分隔各个 JSON 对象。 **数组**指定输出的格式为 JSON 对象的数组。 仅当作业停止或流分析移动到下个时间段时，才关闭此数组。 一般情况下，最好使用分隔行 JSON，因为它不需要任何特殊处理，同时仍被写入输出文件。|

### <a name="renew-data-lake-store-authorization"></a>续订 Data Lake Store 授权
如果自作业创建后或上次身份验证后更改了密码，则需要重新对 Data Lake Store 帐户进行身份验证。 如果你不重新进行身份验证，你的作业不会生成输出结果，并显示一个错误，指示需要重新授权操作日志中。 

目前，身份验证令牌需要进行手动刷新具有 Data Lake Store 输出的所有作业每隔 90 天。 您可以克服此限制由[进行身份验证通过托管标识 （预览）](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-managed-identities-adls)。

若要续订授权：

1. 选择**停止**若要停止的作业。
1. 请转到 Data Lake Store 输出并选择**续订授权**链接。

   短的时间，弹出页指示**将重定向到授权**。 如果授权是成功，页指示**授权已成功续订**，然后自动关闭。 
   
1. 选择**保存**页的底部。 然后可以重新启动工作**上次停止时间**以避免数据丢失。

![在输出中续订 Data Lake Store 授权](./media/stream-analytics-define-outputs/08-stream-analytics-define-outputs.png)

## <a name="sql-database"></a>SQL 数据库
可以使用[Azure SQL 数据库](https://azure.microsoft.com/services/sql-database/)作为输出的是在本质的数据或依赖关系数据库中托管的内容的应用程序。 Stream Analytics 作业写入到 SQL 数据库中的现有表。 表架构必须与字段和它们中作业的输出的类型完全匹配。 此外可以指定[Azure SQL 数据仓库](https://azure.microsoft.com/documentation/services/sql-data-warehouse/)作为输出通过 SQL 数据库输出选项。 若要了解有关如何提高写入吞吐量，请参阅[Stream Analytics 与 Azure SQL 数据库作为输出](stream-analytics-sql-output-perf.md)一文。 

下表列出了属性名称和用于创建 SQL 数据库输出及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |在查询中使用的友好名称，用于将查询输出定向到此数据库。 |
| 数据库 | 发送输出的位置的数据库的名称。 |
| 服务器名称 | SQL 数据库服务器名称。 |
| 用户名 | 对数据库具有写权限的用户名。 Stream Analytics 仅支持 SQL 身份验证。 |
| 密码 | 用于连接到数据库的密码。 |
| 表 | 将写入输出的表名称。 表名称是区分大小写。 此表的架构应与字段数量以及作业输出生成其类型完全匹配。 |
|继承分区方案| 用于继承前一查询步骤的分区方案，若要启用完全并行拓扑与多个编写器的表选项。 有关详细信息，请参阅从 [Azure 流分析输出到 Azure SQL 数据库](stream-analytics-sql-output-perf.md)。|
|匹配批数| 与每个批量发送的记录数的建议的限制插入事务。|

> [!NOTE]
> 目前，流分析中的作业输出支持 Azure SQL 数据库产品/服务。 不支持附加了数据库运行 SQL Server 的 Azure 虚拟机。 这在将来的版本中可能会有所改变。
>

## <a name="blob-storage"></a>Blob 存储
Azure Blob 存储提供了用于在云中存储大量非结构化数据的经济高效且可缩放解决方案。 有关 Blob 存储及其用法的简介，请参阅[如何使用 blob](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。

下表列出了属性名称和用于创建 blob 输出及其说明。

| 属性名称       | 说明                                                                      |
| ------------------- | ---------------------------------------------------------------------------------|
| 输出别名        | 查询中使用的友好名称，用于将查询输出定向到此 blob 存储。 |
| 存储帐户     | 要将发送输出的存储帐户的名称。               |
| 存储帐户密钥 | 与存储帐户关联的密钥。                              |
| 存储容器   | 对 Azure Blob 服务中存储的 blob 进行逻辑分组。 将 blob 上传到 Blob 服务时，必须为该 blob 指定一个容器。 |
| 路径模式 | 可选。 用于写入指定的容器中的 blob 文件路径模式。 <br /><br /> 在路径模式中，您可以选择使用日期和时间变量的一个或多个实例指定 blob 写入的频率： <br /> {date}、{time} <br /><br />可以使用自定义 blob 分区从事件数据中指定一个自定义 {field} 名称来对 blob 进行分区。 字段名称是字母数字，并且可以包含空格、连字符和下划线。 对自定义字段的限制包括以下内容： <ul><li>字段名称不区分大小写。 例如，服务无法区分列"ID"和列"id。</li><li>不允许嵌套的字段。 相反，别名在查询中使用作业"平展"字段。</li><li>表达式不能用作字段名称。</li></ul> <br />通过此功能可以在路径中使用自定义日期/时间格式说明符配置。 一次只能指定一个自定义日期和时间格式，并用 {datetime:\<specifier>} 关键字括起来。 允许输入\<说明符 > 为 yyyy、 MM、 M、 dd、 d、 HH、 H、 mm、 m、 ss 或 s。 {Datetime:\<说明符 >} 关键字可用于多个时间在路径中窗体自定义日期/时间的配置。 <br /><br />示例： <ul><li>示例 1：cluster1/logs/{date}/{time}</li><li>示例 2：cluster1/logs/{date}</li><li>示例 3：cluster1/{client_id}/{date}/{time}</li><li>示例 4：cluster1/{datetime:ss}/{myField}，其中查询是：SELECT data.myField AS myField FROM Input;</li><li>示例 5：cluster1/year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}</ul><br />创建的文件夹结构的时间戳遵循 UTC 并不是本地时间。<br /><br />文件命名使用以下约定： <br /><br />{路径前缀模式}/schemaHashcode_Guid_Number.extension<br /><br />示例输出文件：<ul><li>Myoutput/20170901/00/45434_gguid_1.csv</li>  <li>Myoutput/20170901/01/45434_gguid_1.csv</li></ul> <br />有关此功能的详细信息，请参阅[Azure Stream Analytics 自定义 blob 输出分区](stream-analytics-custom-path-patterns-blob-storage-output.md)。 |
| 日期格式 | 可选。 如果在前缀路径中使用日期令牌，可以选择组织文件所采用的日期格式。 示例：YYYY/MM/DD |
| 时间格式 | 可选。 如果在前缀路径中使用时间令牌，可以指定组织文件所采用的时间格式。 目前唯一支持的值是 HH。 |
| 事件序列化格式 | 输出数据的序列化格式。 支持 JSON、CSV 和 Avro。 |
| 编码    | 如果您使用 CSV 或 JSON 格式，则必须指定一种编码。 目前，UTF-8 是仅受支持的编码格式。 |
| 分隔符   | 仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 格式      | 仅适用于 JSON 序列化。 **分隔行**指定输出的格式由一个新行分隔各个 JSON 对象。 **数组**指定输出的格式为 JSON 对象的数组。 仅当作业停止或流分析移动到下个时间段时，才关闭此数组。 一般情况下，最好使用分隔行 JSON，因为它不需要任何特殊处理，同时仍被写入输出文件。 |

当使用 Blob 存储作为输出时，在 blob 中，在以下情况下创建新文件：

* 如果文件超出了允许的最大块数（目前为 50,000）。 但未达到最大允许的 blob 大小，可能会达到块的最大允许的数目。 例如，如果输出率很高，则可以看到每个块的字节更多，并且文件大小会更大。 输出率较低时，每个块都有较少的数据，且文件大小较小。
* 如果在输出中，架构更改，并且输出格式需要固定的架构 （CSV 和 Avro）。
* 如果作业重新启动，可选择在外部由用户停止或启动，或在内部进行系统维护或错误恢复。
* 如果完全分区查询，并为每个输出分区创建新文件。
* 如果用户删除文件或存储帐户的容器。
* 如果输出是使用路径前缀模式中，分区的时间，则使用新 blob 时查询移动到下一个小时。
* 如果将输出分区通过自定义字段，并且每个分区键如果创建新的 blob 不存在。
* 如果将输出分区通过自定义字段，其中分区键基数超过 8,000，和每个分区密钥创建新的 blob。

## <a name="event-hubs"></a>事件中心
[Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)服务是具有高扩展性的发布 - 订阅事件引入器。 事件中心每秒可收集数百万个事件。 Stream Analytics 作业的输出将成为另一个流式处理作业的输入时的事件中心作为输出的一种用途。

你需要配置数据流从事件中心作为输出的几个参数。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 | 在查询中用于将查询输出定向到此事件中心的友好名称。 |
| 事件中心命名空间 |一组消息传递实体的容器。 创建新的事件中心，还可以创建事件中心命名空间。 |
| 事件中心名称 | 事件中心输出的名称。 |
| 事件中心策略名称 | 共享的访问策略，它可以在事件中心上创建**配置**选项卡。每个共享访问策略具有名称、所设权限以及访问密钥。 |
| 事件中心策略密钥 | 用于进行身份验证访问事件中心命名空间共享的访问密钥。 |
| 分区键列 | 可选。 包含事件中心输出的分区键的列。 |
| 事件序列化格式 | 输出数据的序列化格式。 支持 JSON、CSV 和 Avro。 |
| 编码 | 对于 CSV 和 JSON，目前只支持 UTF-8 这种编码格式。 |
| 分隔符 | 仅适用于 CSV 序列化。 流分析支持一些常用分隔符，以便采用 CSV 格式序列化数据。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 格式 | 仅适用于 JSON 序列化。 **分隔行**指定输出的格式由一个新行分隔各个 JSON 对象。 **数组**指定输出的格式为 JSON 对象的数组。 仅当作业停止或流分析移动到下个时间段时，才关闭此数组。 一般情况下，最好使用分隔行 JSON，因为它不需要任何特殊处理，同时仍被写入输出文件。 |
| 属性列 | 可选。 需要附加为传出消息而不是有效负载的用户属性的逗号分隔的列。 有关此功能的详细信息位于部分[输出的自定义元数据属性](#custom-metadata-properties-for-output)。 |

## <a name="power-bi"></a>Power BI
可以使用[Power BI](https://powerbi.microsoft.com/)作为 Stream Analytics 作业，以提供丰富的可视化体验的分析结果的输出。 用于操作仪表板、 生成报告以及进行指标驱动的报道，可以使用此功能。

流分析中的 Power BI 输出当前不可在 Azure 中国（世纪互联）和 Azure 德国 (T-Systems International) 区域中使用。

### <a name="authorize-a-power-bi-account"></a>向 Power BI 帐户授权
1. 当 Power BI 被选为 Azure 门户中的输出时，系统会提示向现有的 Power BI 用户授权或创建新的 Power BI 帐户。
   
   ![授权的 Power BI 用户若要配置输出](./media/stream-analytics-define-outputs/01-stream-analytics-define-outputs.png)

2. 如果您尚不具有一个，，然后选择创建新帐户**立即授权**。 将显示以下页：
   
   ![从 Azure 帐户进行 Power BI 身份验证](./media/stream-analytics-define-outputs/02-stream-analytics-define-outputs.png)

3. 提供有关如何进行授权的 Power BI 输出的工作或学校帐户。 如果你尚未登录 Power bi，请选择**立即注册**。 用于 Power BI 的工作或学校帐户可能不同于您现在正在登录所用的 Azure 订阅帐户。

### <a name="configure-the-power-bi-output-properties"></a>配置 Power BI 输出属性
Power BI 帐户身份验证后，你可以为自己的 Power BI 输出配置属性。 下表列出了属性名称和配置 Power BI 输出及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |提供在查询中用于将查询输出定向到此 Power BI 输出的友好名称。 |
| 组工作区 |若要启用与其他 Power BI 用户共享数据，您可以在 Power BI 帐户中选择组，或选择**我的工作区**如果不想写入组。 更新现有组需要对 Power BI 重新进行身份验证。 |
| 数据集名称 |提供你想要使用的 Power BI 输出数据集名称。 |
| 表名称 |在 Power BI 输出的数据集下提供表名称。 目前，流分析作业的 Power BI 输出在 1 个数据集中只能有 1 个表。 |

有关配置 Power BI 输出和仪表板的演练，请参阅[Azure Stream Analytics 和 Power BI](stream-analytics-power-bi-dashboard.md)一文。

> [!NOTE]
> 不要显式创建数据集和表在 Power BI 仪表板中。 启动作业和作业开始向 Power BI 发送输出时，会自动填充数据集和表。 如果作业查询没有生成任何结果，不被创建数据集和表。 如果 Power BI 已经有一个数据集和 Stream Analytics 作业中提供的一个与同名的表，是覆盖现有数据。
>

### <a name="create-a-schema"></a>创建架构
如果它们尚不存在，azure Stream Analytics 将创建用户的 Power BI 数据集和表架构。 在所有其他情况下，表可以更新使用新值。 目前，只有一个表中数据集可存在。 

Power BI 使用的先入先出 (FIFO) 保留策略。 数据将收集在表中，直到它达到 200,000 个行。

### <a name="convert-a-data-type-from-stream-analytics-to-power-bi"></a>将从 Stream Analytics 的数据类型转换为 Power BI
如果输出架构更改，Azure 流分析会在运行时动态更新数据模型。 列名称更改、列类型更改，以及添加或删除列，这些都会进行跟踪。

此表涵盖了从的数据类型转换[Stream Analytics 的数据类型](https://msdn.microsoft.com/library/azure/dn835065.aspx)到 Power BI[实体数据模型 (EDM) 类型](https://docs.microsoft.com/dotnet/framework/data/adonet/entity-data-model)，如果 Power BI 数据集和表不存在。

从流分析 | 到 Power BI
-----|-----
bigint | Int64
nvarchar(max) | String
datetime | Datetime
浮点数 | Double
记录数组 | 字符串类型，常量值"IRecord"或"IArray"

### <a name="update-the-schema"></a>更新架构
流分析根据输出中的第一组事件推断数据模型架构。 更高版本，如有必要，数据模型架构进行更新以适应可能并不符合原始架构的传入事件。

避免`SELECT *`查询，以防止跨行动态架构更新。 除了潜在性能影响，它可能会导致不确定性的结果所需的时间。 选择需要在 Power BI 仪表板上显示的确切字段。 此外，数据值应与所选数据类型相符。


上一步/当前 | Int64 | String | Datetime | Double
-----------------|-------|--------|----------|-------
Int64 | Int64 | String | String | Double
Double | Double | String | String | Double
String | String | String | String | String 
datetime | String | String |  datetime | String


### <a name="renew-power-bi-authorization"></a>续订 Power BI 授权
如果你的 Power BI 帐户密码更改后 Stream Analytics 作业已创建或上次进行身份验证，则需要重新进行身份验证 Stream Analytics。 如果在 Azure Active Directory (Azure AD) 租户中配置了多重身份验证，还需要每两周续订一次 Power BI 授权。 此问题的症状是没有作业输出，并且操作日志存在“验证用户错误”：

  ![Power BI 验证用户错误](./media/stream-analytics-define-outputs/03-stream-analytics-define-outputs.png)

要解决此问题，请停止正在运行的作业并转到 Power BI 输出。 选择“续订授权”链接，并在“上次停止时间”重新启动工作以避免数据丢失。

  ![续订 Power BI 授权以获得输出](./media/stream-analytics-define-outputs/04-stream-analytics-define-outputs.png)

## <a name="table-storage"></a>表存储
[Azure 表存储](../storage/common/storage-introduction.md) 提供了具有高可用性且可大规模缩放的存储，因此应用程序可以自动缩放以满足用户需求。 表存储是 Microsoft 的 NoSQL 键/属性存储，这可用于结构化数据的约束较少的架构。 Azure 表存储可用于持久地存储数据，方便进行高效的检索。

下表列出了属性名称和用于创建表输出及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此表存储。 |
| 存储帐户 |要将发送输出的存储帐户的名称。 |
| 存储帐户密钥 |与存储帐户关联的访问密钥。 |
| 表名称 |表的名称。 如果不存在，则会创建表。 |
| 分区键 |包含分区键的输出列的名称。 分区键是构成实体主键的第一个部分的表中分区的唯一标识符。 它是可以为最多为 1 KB 大小的字符串值。 |
| 行键 |包含行键的输出列的名称。 行键是实体在分区中的唯一标识符。 行键构成了实体主键的第二部分。 行键是可以为最多为 1 KB 大小的字符串值。 |
| 批大小 |批处理操作的记录数。 默认值 (100) 对大部分作业来说都已足够。 请参阅[表批处理操作规范](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.table._table_batch_operation)有关修改此设置的详细信息。 |

## <a name="service-bus-queues"></a>服务总线队列
[服务总线队列](https://msdn.microsoft.com/library/azure/hh367516.aspx)提供先进先出消息传递到一个或多个竞争使用者。 通常情况下，接收和处理添加到队列的临时顺序接收方的消息。 每个消息接收并处理由一个消息使用者。

下表列出了属性名称和用于创建队列输出及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |在查询中用于将查询输出定向到此服务总线队列的友好名称。 |
| 服务总线命名空间 |一组消息传递实体的容器。 |
| 队列名称 |服务总线队列的名称。 |
| 队列策略名称 |在创建队列时，还可以在队列创建共享的访问策略**配置**选项卡。每个共享访问策略具有名称、所设权限以及访问密钥。 |
| 队列策略密钥 |用于对服务总线命名空间的访问权限进行身份验证的共享访问密钥。 |
| 事件序列化格式 |输出数据的序列化格式。 支持 JSON、CSV 和 Avro。 |
| 编码 |对于 CSV 和 JSON，目前只支持 UTF-8 这种编码格式。 |
| 分隔符 |仅适用于 CSV 序列化。 流分析支持一些常用分隔符，以便采用 CSV 格式序列化数据。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 格式 |仅适用于 JSON 类型。 **分隔行**指定输出的格式由一个新行分隔各个 JSON 对象。 **数组**指定输出的格式为 JSON 对象的数组。 |
| 属性列 | 可选。 需要附加为传出消息而不是有效负载的用户属性的逗号分隔的列。 有关此功能的详细信息位于部分[输出的自定义元数据属性](#custom-metadata-properties-for-output)。 |

分区数[基于服务总线 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分区键是每个分区的唯一整数值。

## <a name="service-bus-topics"></a>服务总线主题
服务总线队列提供的一对一通信方法从发送方到接收方。 [服务总线主题](https://msdn.microsoft.com/library/azure/hh367516.aspx)提供一个多形式的通信。

下表列出了属性名称和用于创建主题输出及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |在查询中用于将查询输出定向到此服务总线主题的友好名称。 |
| 服务总线命名空间 |一组消息传递实体的容器。 创建新的事件中心后，还创建了 Service Bus 命名空间。 |
| 主题名称 |主题是消息传递实体，类似于事件中心和队列。 它们旨在从设备和服务收集事件流。 创建主题时，它还具有提供特定的名称。 发送到主题的消息不可用，除非创建了订阅，因此请确保没有主题下的一个或多个订阅。 |
| 主题策略名称 |时创建的主题，还可以在主题上创建共享的访问策略**配置**选项卡。每个共享访问策略具有名称、所设权限以及访问密钥。 |
| 主题策略密钥 |用于对服务总线命名空间的访问权限进行身份验证的共享访问密钥。 |
| 事件序列化格式 |输出数据的序列化格式。 支持 JSON、CSV 和 Avro。 |
| 编码 |如果您使用 CSV 或 JSON 格式，则必须指定一种编码。 目前，UTF-8 是仅受支持的编码格式。 |
| 分隔符 |仅适用于 CSV 序列化。 流分析支持一些常用分隔符，以便采用 CSV 格式序列化数据。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 属性列 | 可选。 需要附加为传出消息而不是有效负载的用户属性的逗号分隔的列。 有关此功能的详细信息位于部分[输出的自定义元数据属性](#custom-metadata-properties-for-output)。 |

分区数[基于服务总线 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分区键是每个分区的唯一整数值。

## <a name="azure-cosmos-db"></a>Azure Cosmos DB
[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/)是一种全球分布式的数据库服务，提供不设限的弹性缩放全球范围内、 丰富查询和自动索引经由与架构无关的数据模型。 若要了解有关 Stream Analytics 的 Azure Cosmos DB 集合选项，请参阅[使用 Azure Cosmos DB 作为输出的 Stream Analytics](stream-analytics-documentdb-output.md)一文。

流分析中的 Azure Cosmos DB 输出当前不可在 Azure 中国（世纪互联）和 Azure 德国 (T-Systems International) 区域中使用。

> [!Note]
> 在此期间，Azure Stream Analytics 仅支持连接到 Azure Cosmos DB 通过使用 SQL API。
> 尚不支持使用其他 Azure Cosmos DB API。 如果使用其他 API 将 Azure 流分析指向 创建的 Azure Cosmos DB 帐户，则可能无法正确存储数据。

下表描述了用于创建 Azure Cosmos DB 输出的属性。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 | 用于在流分析查询中引用此输出的别名。 |
| 接收器 | Azure Cosmos DB。 |
| 导入选项 | 请执行以下某**从订阅选择 Cosmos DB**或**提供 Cosmos DB 设置手动**。
| 帐户 ID | Azure Cosmos DB 帐户的名称或终结点 URI。 |
| 帐户密钥 | Azure Cosmos DB 帐户的共享访问密钥。 |
| 数据库 | Azure Cosmos DB 数据库名称。 |
| 集合名称模式 | 集合名称或集合要使用的模式。 <br />使用可选的 {partition} 令牌，其中分区从 0 开始，可以构造集合名称格式。 两个示例：  <br /><ul><li> _MyCollection_:必须存在一个名为"MyCollection"的集合。</li>  <li> _MyCollection {partition}_:基于分区依据列。</li></ul> 分区依据列集合必须存在："MyCollection0，""MyCollection1"、"MyCollection2，"依次类推。 |
| 分区键 | 可选。 这仅当需要在集合名称模式中使用 {partition} 标记。<br /> 分区键是用于指定跨集合分区输出的密钥输出事件中字段的名称。<br /> 对于单个集合输出，可以使用任何任意输出列。 例如，PartitionId。 |
| 文档 ID |可选。 用于指定哪些 insert 或 update 操作所基于的主键输出事件中字段的名称。

## <a name="azure-functions"></a>Azure Functions
Azure Functions 是一种无服务器计算服务，可用于代码需运行，而无需显式预配或管理基础结构。 它允许实现由 Azure 或合作伙伴服务中发生的事件触发的代码。 Azure functions 响应触发此功能使其自然输出 Azure Stream Analytics。 此输出适配器可让用户将 Stream Analytics 连接到 Azure Functions 和运行脚本或一段代码以响应各种事件。

流分析中的 Azure Functions 输出当前不可在 Azure 中国（世纪互联）和 Azure 德国 (T-Systems International) 区域中使用。

Azure 流分析通过 HTTP 触发器调用 Azure Functions。 Azure Functions 输出适配器现做了以下可配置属性：

| 属性名称 | 描述 |
| --- | --- |
| Function App |Azure Functions 应用的名称。 |
| 函数 |Azure Functions 应用程序中函数的名称。 |
| 密钥 |如果你想要使用另一个订阅中的 Azure Function，就可以做到通过提供该键来访问你的函数。 |
| 最大批处理大小 |一个属性，它允许您设置到 Azure 函数发送每个输出批的最大大小。 输入单元以字节为单位。 默认情况下，此值为 262,144 字节 (256 KB)。 |
| 最大批处理计数  |允许您发送到 Azure Functions 的每个批次中指定的最大事件数属性。 默认值为 100。 |

当 Azure Stream Analytics 收到 413 ("http 请求实体过大") 从一个 Azure 函数的异常，它可以减少发送到 Azure Functions 的批的大小。 在 Azure Function 代码中，使用此异常以确保 Azure 流分析不会发送过大的批。 此外，请确保该函数中使用的最大批处理计数和大小值与在 Stream Analytics 门户中输入的值一致。

此外，在这种情况中的任何事件登录时间窗口中，不生成任何输出。 因此， **computeResult**函数不调用。 此行为与内置窗口化聚合函数一致。

## <a name="custom-metadata-properties-for-output"></a>输出的自定义元数据属性 

可以将查询列作为用户属性附加到传出消息。 这些列不进入有效负载。 属性是字典上输出消息的形式存在。 *键*是列的名称和*值*是属性字典中列的值。 Stream Analytics 的所有数据类型都支持记录和数组除外。  

支持的输出： 
* 服务总线队列 
* 服务总线主题 
* 事件中心 

在以下示例中，我们将添加两个字段`DeviceId`和`DeviceStatus`的元数据。 
* 查询：`select *, DeviceId, DeviceStatus from iotHubInput`
* 输出配置： `DeviceId,DeviceStatus`

![属性列](./media/stream-analytics-define-outputs/10-stream-analytics-property-columns.png)

以下屏幕截图显示了输出消息属性中通过事件中心检查[服务总线资源管理器](https://github.com/paolosalvatori/ServiceBusExplorer)。

![事件自定义属性](./media/stream-analytics-define-outputs/09-stream-analytics-custom-properties.png)

## <a name="partitioning"></a>分区

下表汇总了分区支持，以及每个输出类型的输出写入器数目：

| 输出类型 | 分区支持 | 分区键  | 输出写入器数目 |
| --- | --- | --- | --- |
| Azure Data Lake Store | “是” | 使用 {date} 和 {time} 令牌在路径前缀模式。 选择日期格式，例如 YYYY/MM/DD、 DD/MM/YYYY 或 MM DD YYYY。 HH 用于时间格式。 | 按照[完全可并行化的查询](stream-analytics-scale-jobs.md)的输入分区。 |
| Azure SQL 数据库 | “是” | 根据在查询中的 PARTITION BY 子句。 | 按照[完全可并行化的查询](stream-analytics-scale-jobs.md)的输入分区。 若要了解有关实现的详细信息更好地写入吞吐量性能时将数据加载到 Azure SQL 数据库，请参阅[到 Azure SQL 数据库的 Azure Stream Analytics 输出](stream-analytics-sql-output-perf.md)。 |
| Azure Blob 存储 | “是” | 使用 {date} 和 {time} 令牌在事件的字段中的路径模式。 选择日期格式，例如 YYYY/MM/DD、 DD/MM/YYYY 或 MM DD YYYY。 HH 用于时间格式。 可以通过单个自定义事件属性 {fieldname} 或 {datetime:\<specifier>} 对 blob 输出进行分区。 | 按照[完全可并行化的查询](stream-analytics-scale-jobs.md)的输入分区。 |
| Azure 事件中心 | “是” | “是” | 按分区对齐方式变化。<br /> 当与上游 （上一个） 的查询步骤同样对齐事件中心输出的分区键时的编写器数是与事件中心输出中的分区数相同。 每个编写器使用[EventHubSender 类](/dotnet/api/microsoft.servicebus.messaging.eventhubsender?view=azure-dotnet)将事件发送到特定分区。 <br /> 当事件中心输出的分区键与上游 （上一个） 的查询步骤不符时，编写器数是该前面的步骤中的分区数相同。 每个编写器使用[SendBatchAsync 类](/dotnet/api/microsoft.servicebus.messaging.eventhubclient.sendasync?view=azure-dotnet)中**EventHubClient**将事件发送到输出的所有分区。 |
| Power BI | “否” | 无 | 不适用。 |
| Azure 表存储 | “是” | 任何输出列。  | 按照[完全并行化的查询](stream-analytics-scale-jobs.md)的输入分区。 |
| Azure 服务总线主题 | “是” | 自动选择。 分区数基于[服务总线 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分区键是每个分区的唯一整数值。| 与输出主题中的分区数量相同。  |
| Azure 服务总线队列 | “是” | 自动选择。 分区数基于[服务总线 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分区键是每个分区的唯一整数值。| 与输出队列中的分区数量相同。 |
| Azure Cosmos DB | “是” | 集合名称模式中使用 {partition} 标记。 {Partition} 值基于在查询中的 PARTITION BY 子句。 | 按照[完全并行化的查询](stream-analytics-scale-jobs.md)的输入分区。 |
| Azure Functions | “否” | 无 | 不适用。 |

如果输出适配器未分区，则一个输入分区中缺少数据将导致延迟最多可达延迟到达的时间量。 在这种情况下，输出合并到单个写入者，可能会在管道中造成瓶颈。 若要了解有关晚到策略的详细信息，请参阅[Azure Stream Analytics 事件顺序注意事项](stream-analytics-out-of-order-and-late-events.md)。

## <a name="output-batch-size"></a>输出批大小
Azure Stream Analytics 使用可变大小的批处理来处理事件并将写入到输出。 通常 Stream Analytics 引擎不会一次写入一条消息，并使用批处理以提高效率。 如果传入和传出事件的速率较高，Stream Analytics 将使用更大的批次。 输出速率低时，使用较小的批来保证低延迟。

下表介绍了某些输出批处理时的注意事项：

| 输出类型 | 最大消息大小 | 批大小优化 |
| :--- | :--- | :--- |
| Azure Data Lake Store | 请参阅[Data Lake 存储限制](../azure-subscription-service-limits.md#data-lake-store-limits)。 | 使用每个写入操作最多为 4 MB。 |
| Azure SQL 数据库 | 每个单个大容量插入的 10,000 最大行数。<br />100 最小行数进行单个大容量插入。 <br />请参阅[Azure SQL 限制](../sql-database/sql-database-resource-limits.md)。 |  每个批处理最初是大容量插入具有最大批处理大小。 可以拆分批处理中一半 （直到达到最小批大小） 基于从 SQL 重试的错误。 |
| Azure Blob 存储 | 请参阅[Azure 存储限制](../azure-subscription-service-limits.md#storage-limits)。 | 最大 blob 的块大小为 4 MB。<br />最大 blob bock 计数为 50,000。 |
| Azure 事件中心  | 每个消息的 256 KB。 <br />请参阅[事件中心限制](../event-hubs/event-hubs-quotas.md)。 |  当输入/输出分区未对齐时，每个事件中单独打包**EventData** ，直到达到最大消息大小 (1 MB 的高级 SKU) 的批中发送。 <br /><br />  如果输入/输出分区为 aligned，多个事件会打包到单个**EventData**实例，直到达到最大消息的大小，并发送。  |
| Power BI | 请参阅[Power BI Rest API 限制](https://msdn.microsoft.com/library/dn950053.aspx)。 |
| Azure 表存储 | 请参阅[Azure 存储限制](../azure-subscription-service-limits.md#storage-limits)。 | 默认值为 100 个实体，每个单个事务。 可以将其配置为根据需要较小的值。 |
| Azure 服务总线队列   | 每个消息的 256 KB。<br /> 请参阅[服务总线限制](../service-bus-messaging/service-bus-quotas.md)。 | 使用每个消息的单个事件。 |
| Azure 服务总线主题 | 每个消息的 256 KB。<br /> 请参阅[服务总线限制](../service-bus-messaging/service-bus-quotas.md)。 | 使用每个消息的单个事件。 |
| Azure Cosmos DB   | 请参阅[Azure Cosmos DB 限制](../azure-subscription-service-limits.md#azure-cosmos-db-limits)。 | 批大小和写入的频率进行动态调整基于 Azure Cosmos DB 的响应。 <br /> 没有从 Stream Analytics 任何预先确定的限制。 |
| Azure Functions   | | 默认批大小为 262,144 字节 (256 KB)。 <br /> 每个批处理的默认事件计数为 100。 <br /> 批大小是可配置的，可在流分析[输出选项](#azure-functions)中增加或减少。

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> 
> [快速入门：使用 Azure 门户创建流分析作业](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301
