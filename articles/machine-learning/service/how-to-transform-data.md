---
title: 转换：数据准备 Python SDK
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习数据准备 SDK 转换和清理数据。 转换方法可用于添加列、筛选掉不需要的行或列，以及估算缺失值。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: jmartens
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: db23c8af7eaa4a86691ccb0bb831ce2cc28d635c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471836"
---
# <a name="transform-data-with-the-azure-machine-learning-data-prep-sdk"></a>使用 Azure 机器学习数据准备 SDK 转换数据

在本文中，你将了解转换数据使用的不同方法`azureml-dataprep`包。 包提供了可以轻松添加列、 筛选出不需要的行或列，并输入缺失值的函数。 请参阅完整的参考文档[azureml dataprep 包](https://aka.ms/data-prep-sdk)。

> [!Important]
> 如果要构建一个新的解决方案，请尝试[Azure 机器学习数据集](how-to-explore-prepare-data.md)（预览版） 来转换数据、 快照数据和存储版本控制的数据集定义。 数据集是数据准备 SDK，提供可用于管理 AI 解决方案中的数据集的扩展的功能的下一个版本。 如果您使用`azureml-dataprep`包来创建数据流而不是使用转换与`azureml-datasets`的包装，以创建数据集，您将不能以供以后使用快照或版本控制的数据集。

本操作说明展示了以下任务的示例：

- 使用表达式添加列
- [估算缺失值](#impute-missing-values)
- [按示例派生列](#derive-column-by-example)
- [筛选](#filtering)
- [自定义 Python 转换](#custom-python-transforms)

## <a name="add-column-using-an-expression"></a>使用表达式添加列

Azure 机器学习数据准备 SDK 包含 `substring` 表达式，可用于根据现有列计算值，再将计算出的值放入新列中。 下面的示例先加载数据，再尝试将输入数据添加到相应列中。

```Python
import azureml.dataprep as dprep

# loading data
dflow = dprep.read_csv(path=r'data\crime0-10.csv')
dflow.head(3)
```

||ID|案例号|date|阻止|IUCR|主要类型|描述|地址说明|逮捕|国内|...|病房|社区范围|FBI 代码|X 坐标|Y 坐标|年龄|更新时间|纬度|经度|Location|
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|0|10140490|HY329907|07/05/2015 11:50:00 PM|050XX N NEWLAND AVE|0820|THEFT|$500 AND UNDER|STREET|false|false|...|41|10|06|1129230|1933315|2015|07/12/2015 12:42:46 PM|41.973309466|-87.800174996|(41.973309466, -87.800174996)|
|第|10139776|HY329265|07/05/2015 11:30:00 PM|011XX W MORSE AVE|0460|BATTERY|SIMPLE|STREET|false|true|...|49|第|08B|1167370|1946271|2015|07/12/2015 12:42:46 PM|42.008124017|-87.65955018|(42.008124017, -87.65955018)|
|2|10140270|HY329253|07/05/2015 11:20:00 PM|121XX S FRONT AVE|0486|BATTERY|DOMESTIC BATTERY SIMPLE|STREET|false|true|...|9|53|08B|||2015|07/12/2015 12:42:46 PM|


使用 `substring(start, length)` 表达式从“案例号”列中提取前缀，再将该字符串放入新列“`Case Category`”中。 将 `substring_expression` 变量传递给 `expression` 参数会创建一个新的计算列，该列在每条记录上执行表达式。

```Python
substring_expression = dprep.col('Case Number').substring(0, 2)
case_category = dflow.add_column(new_column_name='Case Category',
                                    prior_column='Case Number',
                                    expression=substring_expression)
case_category.head(3)
```

||ID|案例号|案例类别|date|阻止|IUCR|主要类型|描述|地址说明|逮捕|国内|...|病房|社区范围|FBI 代码|X 坐标|Y 坐标|年龄|更新时间|纬度|经度|Location|
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|------|
|0|10140490|HY329907|HY|07/05/2015 11:50:00 PM|050XX N NEWLAND AVE|0820|THEFT|$500 AND UNDER|STREET|false|false|...|41|10|06|1129230|1933315|2015|07/12/2015 12:42:46 PM|41.973309466|-87.800174996|(41.973309466, -87.800174996)|
|第|10139776|HY329265|HY|07/05/2015 11:30:00 PM|011XX W MORSE AVE|0460|BATTERY|SIMPLE|STREET|false|true|...|49|第|08B|1167370|1946271|2015|07/12/2015 12:42:46 PM|42.008124017|-87.65955018|(42.008124017, -87.65955018)|
|2|10140270|HY329253|HY|07/05/2015 11:20:00 PM|121XX S FRONT AVE|0486|BATTERY|DOMESTIC BATTERY SIMPLE|STREET|false|true|...|9|53|08B|||2015|07/12/2015 12:42:46 PM|


使用 `substring(start)` 表达式仅从“案例号”列中提取编号并新建一列。 使用 `to_number()` 函数将其转换为数值数据类型，并将字符串列名作为参数传递。

```Python
substring_expression2 = dprep.col('Case Number').substring(2)
case_id = dflow.add_column(new_column_name='Case Id',
                              prior_column='Case Number',
                              expression=substring_expression2)
case_id = case_id.to_number('Case Id')
```

## <a name="impute-missing-values"></a>估算缺失值

SDK 可以估算指定列中的缺失值。 下面的示例先加载纬度和经度值，再尝试估算输入数据中的缺失值。

```python
import azureml.dataprep as dprep

# loading input data
dflow = dprep.read_csv(r'data\crime0-10.csv')
dflow = dflow.keep_columns(['ID', 'Arrest', 'Latitude', 'Longitude'])
dflow = dflow.to_number(['Latitude', 'Longitude'])
dflow.head(3)
```

||ID|逮捕|纬度|经度|
|-----|------|-----|------|-----|
|0|10140490|false|41.973309|-87.800175|
|第|10139776|false|42.008124|-87.659550|
|2|10140270|false|NaN|NaN|

第三条记录缺少纬度和经度值。 若要输入这些缺失值，请使用[ `ImputeMissingValuesBuilder` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py)若要了解一个固定的表达式。 它可使用计算得出的 `MIN`、`MAX`、`MEAN` 值或 `CUSTOM` 值来估算列。 如果指定 `group_by_columns`，将使用每组计算得出的 `MIN`、`MAX` 和 `MEAN` 按组估算缺失值。

检查`MEAN`纬度列使用的值[ `summarize()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#summarize-summary-columns--typing-union-typing-list-azureml-dataprep-api-dataflow-summarycolumnsvalue---nonetype----none--group-by-columns--typing-union-typing-list-str---nonetype----none--join-back--bool---false--join-back-columns-prefix--typing-union-str--nonetype----none-----azureml-dataprep-api-dataflow-dataflow)函数。 此函数接受 `group_by_columns` 参数中的列数组以指定聚合级别。 `summary_columns` 参数接受 `SummaryColumnsValue` 调用。 此函数调用指定当前列名、新计算字段的名称和要执行的 `SummaryFunction`。

```python
dflow_mean = dflow.summarize(group_by_columns=['Arrest'],
                       summary_columns=[dprep.SummaryColumnsValue(column_id='Latitude',
                                                                 summary_column_name='Latitude_MEAN',
                                                                 summary_function=dprep.SummaryFunction.MEAN)])
dflow_mean = dflow_mean.filter(dprep.col('Arrest') == 'false')
dflow_mean.head(1)
```

||逮捕|Latitude_MEAN|
|-----|-----|----|
|0|false|41.878961|

纬度的 `MEAN` 值看上去是准确的，用 `ImputeColumnArguments` 函数来估算它。 此函数接受 `column_id` 字符串，并使用 `ReplaceValueFunction` 指定估算类型。 对于缺失的经度值，将根据外部知识使用 42 进行估算。

可以使用生成器函数 `impute_missing_values()` 将估算步骤链接到一个 `ImputeMissingValuesBuilder` 对象中。 `impute_columns` 参数接受 `ImputeColumnArguments` 对象的数组。 调用 `learn()` 函数来存储估算步骤，然后使用 `to_dataflow()` 应用于数据流对象。

```python
# impute with MEAN
impute_mean = dprep.ImputeColumnArguments(column_id='Latitude',
                                          impute_function=dprep.ReplaceValueFunction.MEAN)
# impute with custom value 42
impute_custom = dprep.ImputeColumnArguments(column_id='Longitude',
                                            custom_impute_value=42)
# get instance of ImputeMissingValuesBuilder
impute_builder = dflow.builders.impute_missing_values(impute_columns=[impute_mean, impute_custom],
                                                   group_by_columns=['Arrest'])

impute_builder.learn()
dflow_imputed = impute_builder.to_dataflow()
dflow_imputed.head(3)
```

||ID|逮捕|纬度|经度|
|-----|------|-----|------|-----|
|0|10140490|false|41.973309|-87.800175|
|第|10139776|false|42.008124|-87.659550|
|2|10140270|false|41.878961|42.000000|

如以上结果所示，缺少的纬度是使用 `Arrest=='false'` 组的 `MEAN` 值进行估算。 缺少的经度是使用 42 进行估算。

```python
imputed_longitude = dflow_imputed.to_pandas_dataframe()['Longitude'][2]
assert imputed_longitude == 42
```

## <a name="derive-column-by-example"></a>按示例派生列

Azure 机器学习数据准备 SDK 中更高级的一个工具能使用所需结果的示例派生列。 这样一来，只需为 SDK 提供示例，它便能生成可实现预期转换的代码。

```python
import azureml.dataprep as dprep
dflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/BostonWeather.csv')
dflow.head(4)
```

||DATE|REPORTTPYE|HOURLYDRYBULBTEMPF|HOURLYRelativeHumidity|HOURLYWindSpeed|
|----|----|----|----|----|----|
|0|1/1/2015 0:54|FM-15|22|50|10|
|第|1/1/2015 1:00|FM-12|22|50|10|
|2|1/1/2015 1:54|FM-15|22|50|10|
|3|1/1/2015 2:54|FM-15|22|50|11|

假设需要将此文件与日期和时间格式为“Mar 10, 2018 | 2AM-4AM”的数据集联接起来。

```python
builder = dflow.builders.derive_column_by_example(source_columns=['DATE'], new_column_name='date_timerange')
builder.add_example(source_data=dflow.iloc[1], example_value='Jan 1, 2015 12AM-2AM')
builder.preview(count=5) 
```

||DATE|date_timerange|
|----|----|----|
|0|1/1/2015 0:54|Jan 1, 2015 12AM-2AM|
|第|1/1/2015 1:00|Jan 1, 2015 12AM-2AM|
|2|1/1/2015 1:54|Jan 1, 2015 12AM-2AM|
|3|1/1/2015 2:54|Jan 1, 2015 2AM-4AM|
|4|1/1/2015 3:54|Jan 1, 2015 2AM-4AM|

上面的代码先创建派生列的生成器。 你提供了要考虑的源列数组 (`DATE`)，以及要添加的新列的名称。 你传入了第二行（索引 1），并提供了派生列的预期值，作为第一个示例。

最后，你调用了 `builder.preview(skip=30, count=5)`，并能在源列旁边看到派生列。 虽然格式正确，但日期值都是“Jan 1, 2015”。

现在，传入向下查看行需要跳过 (`skip`) 最上面几行对应的数字。

> [!NOTE]
> Preview() 函数将跳过的行，但不是重新编号输出索引。 在以下示例中，表中索引 0 对应于索引 30 数据流中。

```python
builder.preview(skip=30, count=5)
```

||DATE|date_timerange|
|-----|-----|-----|
|0|1/1/2015 22:54|Jan 1, 2015 10PM-12AM|
|第|1/1/2015 23:54|Jan 1, 2015 10PM-12AM|
|2|1/1/2015 23:59|Jan 1, 2015 10PM-12AM|
|3|1/2/2015 0:54|Feb 1, 2015 12AM-2AM|
|4|1/2/2015 1:00|Feb 1, 2015 12AM-2AM|

此处可以发现生成的程序存在问题。 仅根据你上面提供的一个示例，派生程序选择了将日期分析为“日/月/年”，但这并不符合本示例的需求。 若要解决此问题，特定记录索引为目标，并提供另一个示例使用`add_example()`函数在`builder`变量。

```python
builder.add_example(source_data=dflow.iloc[3], example_value='Jan 2, 2015 12AM-2AM')
builder.preview(skip=30, count=5)
```

||DATE|date_timerange|
|-----|-----|-----|
|0|1/1/2015 22:54|Jan 1, 2015 10PM-12AM|
|1|1/1/2015 23:54|Jan 1, 2015 10PM-12AM|
|2|1/1/2015 23:59|Jan 1, 2015 10PM-12AM|
|3|1/2/2015 0:54|Jan 2, 2015 12AM-2AM|
|4|1/2/2015 1:00|Jan 2, 2015 12AM-2AM|

现在正确处理的行 2015 年 1 月 2 日作为"2015 年 1 月 2 日"，但如果您查看超出索引 76 派生列，您看到的值派生列中具有执行任何操作。

```python
builder.preview(skip=75, count=5)
```


||DATE|date_timerange|
|-----|-----|-----|
|0|1/3/2015 7:00|2015 年 1 月 3 日上午 6 点-8|
|1|1/3/2015 7:54|2015 年 1 月 3 日上午 6 点-8|
|2|1/29/2015 6:54|无|
|3|1/29/2015 7:00|无|
|4|1/29/2015 7:54|无|

```python
builder.add_example(source_data=dflow.iloc[77], example_value='Jan 29, 2015 6AM-8AM')
builder.preview(skip=75, count=5)
```

||DATE|date_timerange|
|-----|-----|-----|
|0|1/3/2015 7:00|2015 年 1 月 3 日上午 6 点-8|
|第|1/3/2015 7:54|2015 年 1 月 3 日上午 6 点-8|
|2|1/29/2015 6:54|2015 年 1 月 29 日上午 6 点-8|
|3|1/29/2015 7:00|2015 年 1 月 29 日上午 6 点-8|
|4|1/29/2015 7:54|2015 年 1 月 29 日上午 6 点-8|

 若要查看当前示例中派生的调用列表`list_examples()`生成器对象。

```python
examples = builder.list_examples()
```

| |DATE|示例|example_id|
| -------- | -------- | -------- | -------- |
|0|1/1/2015 1:00|Jan 1, 2015 12AM-2AM|-1|
|1|1/2/2015 0:54|Jan 2, 2015 12AM-2AM|-2|
|2|1/29/2015 20:54|Jan 29, 2015 8PM-10PM|-3|


在某些情况下如果你想要删除的示例，不正确，则可以传递中任意一种`example_row`pandas 数据帧，从或`example_id`值。 例如，如果您运行`builder.delete_example(example_id=-1)`，它将删除第一个转换示例。


调用`to_dataflow()`生成器，它返回数据流与添加所需的派生列。

```python
dflow = builder.to_dataflow()
df = dflow.to_pandas_dataframe()
```

## <a name="filtering"></a>筛选

SDK 包括方法[ `drop_columns()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#drop-columns-columns--multicolumnselection-----azureml-dataprep-api-dataflow-dataflow)并[ `filter()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py)以便你可以筛选出列或行。

### <a name="initial-setup"></a>初始设置

```python
import azureml.dataprep as dprep
from datetime import datetime
dflow = dprep.read_csv(path='https://dprepdata.blob.core.windows.net/demo/green-small/*')
dflow.head(5)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Store_and_fwd_flag|RateCodeID|Pickup_longitude|Pickup_latitude|Dropoff_longitude|Dropoff_latitude|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|0|无|无|无|无|无|无|无|无|无|无|无|无|无|
|第|2013-08-01 08:14:37|2013-08-01 09:09:06|N|第|0|0|0|0|第|.00|0|0|21.25|
|2|2013-08-01 09:13:00|2013-08-01 11:38:00|N|第|0|0|0|0|2|.00|0|0|75|
|3|2013-08-01 09:48:00|2013-08-01 09:49:00|N|5|0|0|0|0|1|.00|0|第|2.1|
|4|2013-08-01 10:38:35|2013-08-01 10:38:51|N|第|0|0|0|0|1|.00|0|0|3.25|

### <a name="filtering-columns"></a>筛选列

若要筛选列，请使用 `drop_columns()`。 此方法采用要删除的列列表或更复杂的自变量调用[ `ColumnSelector` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.columnselector?view=azure-dataprep-py)。

#### <a name="filtering-columns-with-list-of-strings"></a>使用字符串列表筛选列

在下面的示例中，`drop_columns()` 需要使用字符串列表。 每个字符串应与要删除的相应列完全匹配。

```python
dflow = dflow.drop_columns(['Store_and_fwd_flag', 'RateCodeID'])
dflow.head(2)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Pickup_longitude|Pickup_latitude|Dropoff_longitude|Dropoff_latitude|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|0|无|无|无|无|无|无|无|无|无|无|无|
|1|2013-08-01 08:14:37|2013-08-01 09:09:06|0|0|0|0|第|.00|0|0|21.25|

#### <a name="filtering-columns-with-regex"></a>使用正则表达式筛选列

也可以使用 `ColumnSelector` 表达式，删除与正则表达式匹配的列。 下面的示例删除与表达式 `Column*|.*longitude|.*latitude` 匹配的所有列。

```python
dflow = dflow.drop_columns(dprep.ColumnSelector('Column*|.*longitud|.*latitude', True, True))
dflow.head(2)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|
|0|无|无|无|无|无|无|无|
|第|2013-08-01 08:14:37|2013-08-01 09:09:06|1|.00|0|0|21.25|

## <a name="filtering-rows"></a>筛选行

若要筛选行，请使用 `filter()`。 此方法需要使用 Azure 机器学习数据准备 SDK 表达式作为参数，并返回包含表达式评估为 True 的行的新数据流。 表达式使用表达式生成器（`col`、`f_not`、`f_and`、`f_or`）和常规运算符（>、<、>=、<=、==、!=）生成。

### <a name="filtering-rows-with-simple-expressions"></a>使用简单表达式筛选行

使用表达式生成器 `col`，将列名指定为字符串参数 `col('column_name')`。 将此表达式与一个标准运算符（>、<、>=、<=、==、!=）组合使用生成像 `col('Tip_amount') > 0` 这样的表达式。 最后，将生成的表达式传入 `filter()` 函数中。

在下面的示例中，`dflow.filter(col('Tip_amount') > 0)` 返回新数据流，其中包含 `Tip_amount` 值大于 0 的行。

> [!NOTE] 
> `Tip_amount` 先转换为数字，这便于生成将它与其他数字值进行比较的表达式。

```python
dflow = dflow.to_number(['Tip_amount'])
dflow = dflow.filter(dprep.col('Tip_amount') > 0)
dflow.head(2)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|
|0|2013-08-01 19:33:28|2013-08-01 19:35:21|5|.00|0.08|0|4.58|
|1|2013-08-05 13:16:38|2013-08-05 13:18:24|1|.00|0.30|0|3.8|

### <a name="filtering-rows-with-complex-expressions"></a>使用复杂表达式筛选行

若要使用复杂表达式进行筛选，请结合使用一个或多个简单表达式与表达式生成器 `f_not`、`f_and` 或 `f_or`。

在下面的示例中，`dflow.filter()` 返回新数据流，其中包含 `'Passenger_count'` 小于 5 且 `'Tolls_amount'` 大于 0 的行。

```python
dflow = dflow.to_number(['Passenger_count', 'Tolls_amount'])
dflow = dflow.filter(dprep.f_and(dprep.col('Passenger_count') < 5, dprep.col('Tolls_amount') > 0))
dflow.head(2)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|
|0|2013-08-08 12:16:00|2013-08-08 12:16:00|1.0|.00|2.25|5.00|19.75|
|第|2013-08-12 14:43:53|2013-08-12 15:04:50|1.0|5.28|6.46|5.33|32.29|

还可以结合使用多个表达式生成器来创建嵌套表达式，从而筛选行。

> [!NOTE]
> `lpep_pickup_datetime` 和 `Lpep_dropoff_datetime` 先转换为日期/时间，这便于生成将它与其他日期/时间值进行比较的表达式。

```python
dflow = dflow.to_datetime(['lpep_pickup_datetime', 'Lpep_dropoff_datetime'], ['%Y-%m-%d %H:%M:%S'])
dflow = dflow.to_number(['Total_amount', 'Trip_distance'])
mid_2013 = datetime(2013,7,1)
dflow = dflow.filter(
    dprep.f_and(
        dprep.f_or(
            dprep.col('lpep_pickup_datetime') > mid_2013,
            dprep.col('Lpep_dropoff_datetime') > mid_2013),
        dprep.f_and(
            dprep.col('Total_amount') > 40,
            dprep.col('Trip_distance') < 10)))
dflow.head(2)
```

||lpep_pickup_datetime|Lpep_dropoff_datetime|Passenger_count|Trip_distance|Tip_amount|Tolls_amount|Total_amount|
|-----|-----|-----|-----|-----|-----|-----|-----|
|0|2013-08-13 06:11:06+00:00|2013-08-13 06:30:28+00:00|1.0|9.57|7.47|5.33|44.80|
|第|2013-08-23 12:28:20+00:00|2013-08-23 12:50:28+00:00|2.0|8.22|8.08|5.33|40.41|

## <a name="custom-python-transforms"></a>自定义 Python 转换

最简单的转换方式是编写自己的脚本，这样的情况始终存在。 SDK 提供了三个可用于自定义 Python 脚本的扩展点。

- 新脚本列
- 新脚本筛选器
- 转换分区

纵向扩展和横向扩展运行时支持所有扩展。 使用这些扩展点的关键优势是，无需拉取所有数据，即可创建数据框。 自定义 Python 代码像其他转换一样按分区批量运行，通常是并行的。

### <a name="initial-data-preparation"></a>初始数据准备

首先，从 Azure Blob 加载一些数据。

```python
import azureml.dataprep as dprep
col = dprep.col

dflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv', skip_rows=1)
dflow.head(2)
```

| |stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|------|
|0|阿拉巴马州|第|101710|Hale County|10171002158| |
|第|阿拉巴马州|第|101710|Hale County|10171002162| |

减少数据集和执行一些基本的转换，包括删除列、 替换值和转换类型。

```python
dflow = dflow.keep_columns(['stnam', 'leanm10', 'ncessch', 'MAM_MTH00numvalid_1011'])
dflow = dflow.replace_na(columns=['leanm10', 'MAM_MTH00numvalid_1011'], custom_na_list='.')
dflow = dflow.to_number(['ncessch', 'MAM_MTH00numvalid_1011'])
dflow.head(2)
```

| |stnam|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|
|0|阿拉巴马州|Hale County|1.017100e+10|无|
|第|阿拉巴马州|Hale County|1.017100e+10|无|

使用以下筛选器查找空值。

```python
dflow.filter(col('MAM_MTH00numvalid_1011').is_null()).head(2)
```

| |stnam|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|
|0|阿拉巴马州|Hale County|1.017100e+10|无|
|第|阿拉巴马州|Hale County|1.017100e+10|无|

### <a name="transform-partition"></a>转换分区

使用[ `transform_partition()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#transform-partition-script--str-----azureml-dataprep-api-dataflow-dataflow)以替换所有 null 值为 0。 下面的代码按分区运行，而不是一次性对整个数据集运行。 也就是说，对于大型数据集，下面的代码可能会在运行时处理数据时按分区并行运行。

Python 脚本必须定义一个名为 `transform()` 的函数，该函数接受 `df` 和 `index` 两个参数。 `df` 参数是包含分区数据的 panda dataframe，`index` 参数是分区的唯一标识符。 转换函数可以充分编辑在 dataframe 中传递的内容，但是必须返回 dataframe。 Python 脚本导入的任何库都必须位于运行数据流的环境中。

```python
df = df.transform_partition("""
def transform(df, index):
    df['MAM_MTH00numvalid_1011'].fillna(0,inplace=True)
    return df
""")
df.head(2)
```

||stnam|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|
|0|阿拉巴马州|Hale County|1.017100e+10|0.0|
|第|阿拉巴马州|Hale County|1.017100e+10|0.0|

### <a name="new-script-column"></a>新脚本列

若要创建新的列具有县名称和状态名称，以及利用状态名称，可以使用 Python 脚本。 若要执行此操作，请使用[ `new_script_column()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#new-script-column-new-column-name--str--insert-after--str--script--str-----azureml-dataprep-api-dataflow-dataflow)上数据流的方法。

Python 脚本必须定义一个名为 `newvalue()` 的函数，该函数接受一个自变量 `row`。 `row` 自变量是 dict (`key`:column name, `val`: current value)，会为数据集中的每一行将该自变量传递给该函数。 此函数必须返回要在新列中使用的值。 Python 脚本导入的任何库都必须位于运行数据流的环境中。

```python
dflow = dflow.new_script_column(new_column_name='county_state', insert_after='leanm10', script="""
def newvalue(row):
    return row['leanm10'] + ', ' + row['stnam'].title()
""")
dflow.head(2)
```

||stnam|leanm10|county_state|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|
|0|阿拉巴马州|Hale County|Hale County, Alabama|1.017100e+10|0.0|
|第|阿拉巴马州|Hale County|Hale County, Alabama|1.017100e+10|0.0|

### <a name="new-script-filter"></a>新脚本筛选器

生成使用 Python 表达式[ `new_script_filter()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#new-script-filter-script--str-----azureml-dataprep-api-dataflow-dataflow)来筛选数据集 Hale 不在新的行`county_state`列。 若要保留行，表达式返回 `True`；若要删除行，表达式返回 `False`。

```python
dflow = dflow.new_script_filter("""
def includerow(row):
    val = row['county_state']
    return 'Hale' not in val
""")
dflow.head(2)
```

||stnam|leanm10|county_state|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|
|0|阿拉巴马州|Jefferson County|Jefferson County, Alabama|1.019200e+10|1.0|
|第|阿拉巴马州|Jefferson County|Jefferson County, Alabama|1.019200e+10|0.0|

## <a name="next-steps"></a>后续步骤

* 请参阅 Azure 机器学习数据准备 SDK[教程](tutorial-data-prep.md)有关解决特定方案的示例
