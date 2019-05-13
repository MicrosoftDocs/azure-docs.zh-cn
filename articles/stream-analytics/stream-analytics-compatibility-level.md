---
title: 了解 Azure 流分析作业的兼容性级别
description: 了解如何设置 Azure 流分析作业的兼容性级别，并了解最新兼容性级别中的重大更改
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 5/2/2019
ms.openlocfilehash: c1b2875e6899d2301a4c4b564882214dc7bc4981
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205462"
---
# <a name="compatibility-level-for-azure-stream-analytics-jobs"></a>Azure 流分析作业的兼容性级别

本文介绍 Azure Stream Analytics 中的兼容性级别选项。 Stream Analytics 是托管的服务，与定期进行功能更新和性能改进。 大多数服务的运行时更新将自动供最终用户。 

但是，在服务中的一些新功能可能会带来了重大更改，如更改行为的一个现有作业，或更改数据的方式占用中正在运行的作业。 你可以保留现有 Stream Analytics 作业离开降低安全设置的兼容性级别，而不进行主要更改正在运行。 有关最新的运行时行为准备就绪后，可以选择在通过提高兼容性级别。 

## <a name="choose-a-compatibility-level"></a>选择兼容性级别

兼容性级别控制流分析作业的运行时行为。 

Azure Stream Analytics 当前支持三种兼容性级别：

* 1.0-以前的行为
* 1.1-默认行为
* 1.2 （预览版）-在计算中的最新改进的最新行为

在公开上市的 Azure Stream Analytics 几年前引入了原始 1.0 兼容性级别。

当创建新的 Stream Analytics 作业时，它是最好使用最新的兼容性级别创建。 启动作业设计依赖于最新的行为，以避免添加的变更和复杂程度更高版本上。

## <a name="set-the-compatibility-level"></a>将兼容级别设置

在 Azure 门户或通过使用，可以设置 Stream Analytics 作业的兼容性级别[创建作业 REST API 调用](/rest/api/streamanalytics/stream-analytics-job)。

若要更新在 Azure 门户中作业的兼容性级别：

1. 使用[Azure 门户](https://portal.azure.com)以定位到 Stream Analytics 作业。
2. **停止**更新兼容级别之前，该作业。 如果作业处于运行状态，则无法更新兼容性级别。
3. 下**配置**标题下方，选择**兼容性级别**。
4. 选择所需的兼容性级别值。
5. 选择**保存**页的底部。

![Azure 门户中的流分析兼容性级别](media/stream-analytics-compatibility-level/stream-analytics-compatibility.png)

在更新兼容性级别时，T-SQL 编译器会使用与所选兼容性级别相对应的语法来验证作业。

## <a name="compatibility-level-12-preview"></a>兼容性级别 1.2 （预览版）

在兼容级别 1.2 中引入了以下重大更改：

### <a name="geospatial-functions"></a>地理空间函数

**上一级别：** 使用 azure Stream Analytics 的地理计算。

**1.2 级别：** Azure Stream Analytics，可计算几何投影的地理坐标。 没有任何更改的地理空间函数签名中。 但是，它们的语义是稍有不同，从而允许比之前的更精确计算。

Azure Stream Analytics 支持地理空间引用数据编制索引。 包含地理空间元素的引用数据创建索引的速度更快的联接计算。

已更新的地理空间函数自带熟知文本 (WKT) 地理空间信息格式的完整的表现力。 您可以使用 GeoJson 指定以前不支持其他地理空间信息组件。

有关详细信息，请参阅[更新到 Azure Stream Analytics – 云和 IoT Edge 中的地理空间功能](https://azure.microsoft.com/blog/updates-to-geospatial-functions-in-azure-stream-analytics-cloud-and-iot-edge/)。

### <a name="parallel-query-execution-for-input-sources-with-multiple-partitions"></a>具有多个分区的输入源的并行查询执行

**上一级别：** Azure Stream Analytics 查询需要使用 PARTITION BY 子句在输入的源分区并行查询处理的数据。

**1.2 级别：** 如果在输入的源分区，可以并行化查询逻辑，Azure Stream Analytics 创建单独的查询实例，并在并行运行计算。

### <a name="native-bulk-api-integration-with-cosmosdb-output"></a>CosmosDB 输出与本机大容量 API 集成

**上一级别：** Upsert 行为*插入或合并*。

**1.2 级别：** CosmosDB 输出与本机大容量 API 集成最大化吞吐量，并有效地处理限制请求。 有关详细信息，请参阅[Azure Stream Analytics 输出到 Azure Cosmos DB 页](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-documentdb-output#improved-throughput-with-compatibility-level-12)。

Upsert 行为*插入或替换*。

### <a name="datetimeoffset-when-writing-to-sql-output"></a>DateTimeOffset 时写入到 SQL 输出

**上一级别：**[DateTimeOffset](https://docs.microsoft.com/sql/t-sql/data-types/datetimeoffset-transact-sql?view=sql-server-2017)类型已调整为 UTC。

**1.2 级别：** DateTimeOffset 无法再进行调整。

### <a name="strict-validation-of-prefix-of-functions"></a>严格的验证函数的前缀

**上一级别：** 没有函数前缀严格验证。

**1.2 级别：** Azure Stream Analytics 具有严格的验证函数的前缀。 将前缀添加到内置函数将导致错误。 例如，`myprefix.ABS(…)`不受支持。

将前缀添加到内置聚合还会导致错误。 例如，`myprefix.SUM(…)`不受支持。

使用前缀"system"的任何用户定义的函数将导致错误。

### <a name="disallow-array-and-object-as-key-properties-in-cosmos-db-output-adapter"></a>禁止为 Cosmos DB 输出适配器中的键属性的数组和对象

**上一级别：** 数组和对象类型支持为键属性。

**1.2 级别：** 为键属性不再支持数组和对象类型。

## <a name="compatibility-level-11"></a>兼容性级别 1.1

在兼容性级别 1.1 中引入了以下重大更改：

### <a name="service-bus-xml-format"></a>服务总线 XML 格式

**1.0 级别：** Azure 流分析使用 DataContractSerializer，因此消息内容包括 XML 标记。 例如:

`@\u0006string\b3http://schemas.microsoft.com/2003/10/Serialization/\u0001{ "SensorId":"1", "Temperature":64\}\u0001`

**1.1 级别：** 消息内容只包含流，没有其他的标记。 例如： `{ "SensorId":"1", "Temperature":64}`

### <a name="persisting-case-sensitivity-for-field-names"></a>字段名称保留区分大小写

**1.0 级别：** 在由 Azure 流分析引擎处理时，字段名称更改为小写。

**1.1 级别：** Azure Stream Analytics 引擎处理它们时要保留字段名称区分大小写。

> [!NOTE]
> 保留区分大小写尚不适用于使用 Edge 环境托管的流分析作业。 因此，如果在 Edge 上托管作业，所有字段名称都将转换为小写。

### <a name="floatnandeserializationdisabled"></a>FloatNaNDeserializationDisabled

**1.0 级别：** CREATE TABLE 命令不使用 NaN（非数值。 例如，Infinity、-Infinity）在 FLOAT 列类型中筛选事件，因为对于这些数字来说，事件不在已记录的范围之内。

**1.1 级别：** CREATE TABLE 允许指定强架构。 流分析引擎验证数据是否符合此架构。 使用这一模型，该命令可以通过 NaN 值筛选事件。

### <a name="disable-automatic-upcast-for-datetime-strings-in-json"></a>禁用自动向上转换为 json 格式的日期时间字符串

**1.0 级别：** JSON 分析器会自动将包含日期/时间/时区信息的字符串值向上转换为日期/时间类型，然后转换为 UTC。 此行为会导致丢失时区信息。

**1.1 级别：** 不再将包含日期/时间/时区信息的字符串值自动向上转换为日期/时间类型。 因此会保留时区信息。

## <a name="next-steps"></a>后续步骤

* [Azure 流分析输入的故障排除](stream-analytics-troubleshoot-input.md)
* [Stream Analytics 资源运行状况](stream-analytics-resource-health.md)
