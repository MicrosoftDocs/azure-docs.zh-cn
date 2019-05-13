---
title: 探索和准备数据 （数据集类）
titleSuffix: Azure Machine Learning service
description: 了解如何使用摘要统计信息的数据和准备数据通过数据清理、 转换和特征工程
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 05/02/19
ms.openlocfilehash: f4e7fcbe403017a6d957a60a8e5664f2e6c5ba26
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409826"
---
# <a name="explore-and-prepare-data-with-the-dataset-class-preview"></a>探索和准备数据与数据集类 （预览版）

了解如何浏览和使用准备数据[Azure 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)。 [数据集](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py)类 （预览版） 使你可以浏览和准备数据，通过提供函数，如： 采样、 汇总统计信息和智能的转换。 转换步骤保存在[数据集定义](how-to-manage-dataset-definitions.md)功能，以高度可缩放的方式处理不同的架构的多个大文件。

> [!Important]
> 某些数据集类 （预览版） 在数据准备 SDK (GA）) 上具有依赖项。 虽然转换函数可以直接通过 GA'ed[数据准备 SDK 函数](how-to-transform-data.md)，我们建议如果您要构建一个新的解决方案在本文中所述的数据集包包装。 Azure 机器学习数据集 （预览版） 允许将不仅转换你的数据，但还[快照数据](how-to-create-dataset-snapshots.md)并存储[版本控制的数据集定义](how-to-manage-dataset-definitions.md)。 数据集是数据准备 SDK，提供可用于管理 AI 解决方案中的数据集的扩展的功能的下一个版本。

## <a name="prerequisites"></a>必备组件

若要探索和准备数据，您将需要：

* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个免费帐户。 立即试用 [Azure 机器学习服务免费版或付费版](https://aka.ms/AMLFree)。

* Azure 机器学习服务工作区。 请参阅[创建 Azure 机器学习服务工作区](https://docs.microsoft.com/azure/machine-learning/service/setup-create-workspace)。

* 用于 Python 的 Azure 机器学习 SDK (版本 1.0.21 或更高版本)。 若要安装或更新到最新版本的 sdk，请参阅[安装或更新 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py)。

* Azure 机器学习数据准备 SDK。 若要安装或更新到最新版本，请参阅[安装或更新数据准备 SDK](https://docs.microsoft.com/python/api/overview/azure/dataprep/intro?view=azure-dataprep-py#install)。

* 下载示例文件，若要遵循的示例： [crime.csv](https://dprepdata.blob.core.windows.net/dataset-sample-files/crime.csv)并[city.json](https://dprepdata.blob.core.windows.net/dataset-sample-files/city.json)。

## <a name="sampling"></a>采样

需要你的数据以获取初始了解你的数据体系结构和内容的示例。 在此期间， [ `sample()` ](https://docs.microsoft.com//python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#sample-sample-strategy--arguments-)方法从数据集类支持 Top N、 简单随机的和 Stratified 采样策略。

```Python
from azureml.core.dataset import Dataset
import random

# create an in-memory Dataset from a local file
dataset = Dataset.auto_read_files('./data/crime.csv')

# create seed for Simple Random and Stratified sampling
seed = random.randint(0, 4294967295)
```

### <a name="top-n-sample"></a>前 n 个示例

前 n 个采样的数据集的前 n 个记录是您的示例。 这是有帮助，如果只想要了解哪些数据记录外观或若要查看你的数据中有哪些字段等。

```Python
top_n_sample_dataset = dataset.sample('top_n', {'n': 5})
top_n_sample_dataset.to_pandas_dataframe()
```

||ID|案例号|date|阻止|IUCR|主要类型|...|
-|--|-----------|----|-----|----|------------|---
0|10498554|HZ239907|4/4/2016 23:56|007XX E 111TH ST|1153|欺骗性的做法|...
1|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
2|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|欺骗性的做法|...
3|10519591|HZ261534|4/15/2016 9:00|113XX S PRAIRIE AVE|1120|欺骗性的做法|...
4|10534446|HZ277630|4/15/2016 10:00|055XX KEDZIE 保存 N|890|THEFT|...

### <a name="simple-random-sample"></a>简单随机抽样

在简单随机采样的数据填充每个成员具有选择作为示例的一部分的机会。 在`simple_random`采样策略，从你的数据集的记录指定的概率，根据选择，并返回修改后的数据集。 种子参数是可选的。

```Python
simple_random_sample_dataset = dataset.sample('simple_random', {'probability':0.3, 'seed': seed})
simple_random_sample_dataset.to_pandas_dataframe()
```

||ID|案例号|date|阻止|IUCR|主要类型|...|
-|--|-----------|----|-----|----|------------|---
0|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
第|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|欺骗性的做法|...
2|10534446|HZ277630|4/15/2016 10:00|055XX KEDZIE 保存 N|890|THEFT|...

### <a name="stratified-sample"></a>分层的采样

分层的示例确保，在此示例中表示总体的特定组。 在中`stratified`采样策略填充划分为阶层或子组，根据相似之处，并从每个层根据所指示的阶层权重随机选择记录`fractions`参数。

在以下示例中，我们按指定的列，进行分组的每个记录，包括根据阶层 X 权重信息中所述的记录`fractions`。 如果未指定的阶层或记录不能进行分组，示例的默认权重为 0。

```Python
# take 50% of records with `Primary Type` as `THEFT` and 20% of records with `Primary Type` as `DECEPTIVE PRACTICE` into sample Dataset
fractions = {}
fractions[('THEFT',)] = 0.5
fractions[('DECEPTIVE PRACTICE',)] = 0.2

sample_dataset = dataset.sample('stratified', {'columns': ['Primary Type'], 'fractions': fractions, 'seed': seed})

sample_dataset.to_pandas_dataframe()
```

||ID|案例号|date|阻止|IUCR|主要类型|...|
-|--|-----------|----|-----|----|------------|---
0|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
第|10534446|HZ277630|4/15/2016 10:00|055XX KEDZIE 保存 N|890|THEFT|...
2|10535059|HZ278872|4/15/2016 4:30|004XX S KILBOURN AVE|810|THEFT|...

## <a name="explore-with-summary-statistics"></a>浏览具有汇总统计信息

 检测缺失值的异常或错误计算[ `get_profile()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#get-profile-arguments-none--generate-if-not-exist-true--workspace-none--compute-target-none-)方法。 此函数获取的配置文件和摘要统计信息的数据，这有助于确定要应用的必要的数据准备操作。

```Python
dataset.get_profile()
```

||Type|Min|Max|Count|缺失计数|非缺失计数|缺失百分比|错误计数|空计数|0.1% 分位|1% 分位|5% 分位|25% 分位|50% 分位|75% 分位|95% 分位|99% 分位|99.9% 分位|平均值|标准偏差|Variance|倾斜|峰度
-|----|---|---|-----|-------------|-----------------|---------------|-----------|-----------|-------------|-----------|-----------|------------|------------|------------|------------|------------|--------------|----|------------------|--------|--------|--------
ID|FieldType.INTEGER|1.04986e+07|1.05351e+07|10.0|0.0|10.0|0.0|0.0|0.0|1.04986e+07|1.04992e+07|1.04986e+07|1.05166e+07|1.05209e+07|1.05259e+07|1.05351e+07|1.05351e+07|1.05351e+07|1.05195e+07|12302.7|1.51358e+08|-0.495701|-1.02814
案例号|FieldType.STRING|HZ239907|HZ278872|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
date|FieldType.DATE|2016-04-04 23:56:00+00:00|2016-04-15 17:00:00+00:00|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
阻止|FieldType.STRING|004XX S KILBOURN AVE|113XX S PRAIRIE AVE|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
IUCR|FieldType.INTEGER|810|1154|10.0|0.0|10.0|0.0|0.0|0.0|810|850|810|890|1136|1153|1154|1154|1154|1058.5|137.285|18847.2|-0.785501|-1.3543
主要类型|FieldType.STRING|欺骗性的做法|THEFT|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
描述|FieldType.STRING|虚假检查|通过 500 美元|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
地址说明|FieldType.STRING||学校，公共的构建|10.0|0.0|10.0|0.0|0.0|1.0||||||||||||||
逮捕|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
国内|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
检测信号|FieldType.INTEGER|531|2433|10.0|0.0|10.0|0.0|0.0|0.0|531|531|531|614|1318.5|1911|2433|2433|2433|1371.1|692.094|478994|0.105418|-1.60684
区|FieldType.INTEGER|5|24|10.0|0.0|10.0|0.0|0.0|0.0|5|5|5|6|13|19|24|24|24|13.5|6.94822|48.2778|0.0930109|-1.62325
病房|FieldType.INTEGER|第|48|10.0|0.0|10.0|0.0|0.0|0.0|第|5|1|9|22.5|40|48|48|48|24.5|16.2635|264.5|0.173723|-1.51271
社区范围|FieldType.INTEGER|4|77|10.0|0.0|10.0|0.0|0.0|0.0|4|8.5|4|24|37.5|71|77|77|77|41.2|26.6366|709.511|0.112157|-1.73379
FBI 代码|FieldType.INTEGER|6|11|10.0|0.0|10.0|0.0|0.0|0.0|6|6|6|6|11|11|11|11|11|9.4|2.36643|5.6|-0.702685|-1.59582
X 坐标|FieldType.INTEGER|1.16309e+06|1.18336e+06|10.0|7.0|3.0|0.7|0.0|0.0|1.16309e+06|1.16309e+06|1.16309e+06|1.16401e+06|1.16678e+06|1.17921e+06|1.18336e+06|1.18336e+06|1.18336e+06|1.17108e+06|10793.5|1.165e+08|0.335126|-2.33333
Y 坐标|FieldType.INTEGER|1.8315e+06|1.908e+06|10.0|7.0|3.0|0.7|0.0|0.0|1.8315e+06|1.8315e+06|1.8315e+06|1.83614e+06|1.85005e+06|1.89352e+06|1.908e+06|1.908e+06|1.908e+06|1.86319e+06|39905.2|1.59243e+09|0.293465|-2.33333
年龄|FieldType.INTEGER|2016|2016|10.0|0.0|10.0|0.0|0.0|0.0|2016|2016|2016|2016|2016|2016|2016|2016|2016|2016|0|0|NaN|NaN
更新时间|FieldType.DATE|2016-05-11 15:48:00+00:00|2016-05-27 15:45:00+00:00|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
纬度|FieldType.DECIMAL|41.6928|41.9032|10.0|7.0|3.0|0.7|0.0|0.0|41.6928|41.6928|41.6928|41.7057|41.7441|41.8634|41.9032|41.9032|41.9032|41.78|0.109695|0.012033|0.292478|-2.33333
经度|FieldType.DECIMAL|-87.6764|-87.6043|10.0|7.0|3.0|0.7|0.0|0.0|-87.6764|-87.6764|-87.6764|-87.6734|-87.6645|-87.6194|-87.6043|-87.6043|-87.6043|-87.6484|0.0386264|0.001492|0.344429|-2.33333
Location|FieldType.STRING||(41.903206037, -87.676361925)|10.0|0.0|10.0|0.0|0.0|7.0||||||||||||||

## <a name="impute-missing-values"></a>估算缺失值

在数据集中，null 值、 NaN 的和不包含内容的值被视为缺失值。 这些可以影响性能的机器学习模型，或者会导致无效的结论。 补充是一种常见的方法来解决缺失值和时，可以具有较高百分比的缺失值，不能只需删除的记录。

从前面部分中生成的数据集的配置文件的情况下，我们看到`Latitude`和`Longitude`列具有较高百分比的缺失值。 在此示例中，我们计算的平均值，并输入为这两个列的缺失值。

首先，获取与数据集的最新定义[ `get_definition()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#get-definition-version-id-none-)和削减与数据[ `keep_columns()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#keep-columns-columns--multicolumnselection-----azureml-dataprep-api-dataflow-dataflow)，因此，我们仅查看我们想要介绍的列。

```Python
from azureml.core.dataset import Dataset
import azureml.dataprep as dprep

# get the latest definition of Dataset
ds_def = dataset.get_definition()

# keep useful columns for this example
ds_def = ds_def.keep_columns(['ID', 'Arrest', 'Latitude', 'Longitude'])
ds_def.head(3)
```

||ID|逮捕| 纬度|经度|
-|---------|-----|---------|----------|
|0|10498554|False|41.692834|-87.604319|
|1|10516598|False| 41.744107 |-87.664494|
|2|10519196|False| NaN|NaN|

接下来，检查`MEAN`纬度列使用的值[ `summarize()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#summarize-summary-columns--typing-union-typing-list-azureml-dataprep-api-dataflow-summarycolumnsvalue---nonetype----none--group-by-columns--typing-union-typing-list-str---nonetype----none--join-back--bool---false--join-back-columns-prefix--typing-union-str--nonetype----none-----azureml-dataprep-api-dataflow-dataflow)函数。 此函数接受 `group_by_columns` 参数中的列数组以指定聚合级别。 `summary_columns`参数接受`SummaryColumnsValue`函数，它指定当前的列名称，新的计算的字段名称，和`SummaryFunction`来执行。

```Python
lat_mean = ds_def.summarize(group_by_columns = ['Arrest'],
                            summary_columns = [dprep.SummaryColumnsValue(column_id = 'Latitude',
                                                                         summary_column_name = 'Latitude_MEAN',
                                                                         summary_function = dprep.SummaryFunction.MEAN)])
lat_mean = lat_mean.filter(lat_mean['Arrest'] == False)
lat_mean.head(1)
```

||逮捕|Latitude_MEAN|
--|-----|--------|
|0|False|41.780049|

一旦我们验证用户输入的值，请使用[ `ImputeMissingValuesBuilder` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py)若要了解 imputes 具有计算列的固定的表达式`MIN`， `MAX`，`MEAN`值，或`CUSTOM`值。 如果指定 `group_by_columns`，将使用每组计算得出的 `MIN`、`MAX` 和 `MEAN` 按组估算缺失值。

[ `ImputeColumnArguments` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputecolumnarguments?view=azure-dataprep-pyfunction)接受 column_id 字符串和一个`ReplaceValueFunction`指定 impute 类型。 对于缺失的经度值，与基于外部知识-87 输入它。

```Python
# impute with MEAN
impute_mean = dprep.ImputeColumnArguments(column_id = 'Latitude',
                                          impute_function = dprep.ReplaceValueFunction.MEAN)

# impute with custom value -87
impute_custom = dprep.ImputeColumnArguments(column_id='Longitude',
                                            custom_impute_value=-87)
```

输入步骤可以链接在一起`ImputeMissingValuesBuilder`对象使用[ `Builders` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.builders?view=azure-dataprep-py)类函数[ `impute_missing_values()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.builders?view=azure-dataprep-py#impute-missing-values-impute-columns--typing-list-azureml-dataprep-api-builders-imputecolumnarguments----none--group-by-columns--typing-union-typing-list-str---nonetype----none-----azureml-dataprep-api-builders-imputemissingvaluesbuilder)。 `impute_columns` 参数接受 `ImputeColumnArguments` 对象的数组。

```Python
# get instance of ImputeMissingValuesBuilder
impute_builder = ds_def.builders.impute_missing_values(impute_columns=[impute_mean, impute_custom],
                                                   group_by_columns=['Arrest'])
```

调用[ `learn()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py#learn------none)函数来存储 impute 步骤中，并将它们应用于数据流对象使用[ `to_dataflow()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py#to-dataflow------azureml-dataprep-api-dataflow-dataflow)。

```Python
impute_builder.learn()
ds_def = impute_builder.to_dataflow()
ds_def.head(3)
```

以下的输出表中所示，与已处理缺少的纬度`MEAN`的值`Arrest==False`-87 与已输入组和缺少的经度。

||ID|逮捕|纬度|经度
-|---------|-----|---------|----------
0|10498554|False|41.692834|-87.604319
第|10516598|False|41.744107|-87.664494
2|10519196|False|41.780049|-87.000000

更新数据集定义一起使用， [ `update_definition()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#update-definition-definition--definition-update-message-)保留执行的转换步骤。

```Python
dataset = dataset.update_definition(ds_def, 'Impute Missing')
dataset.head(3)
```

||ID|逮捕|纬度|经度
-|---------|-----|---------|----------
0|10498554|False|41.692834|-87.604319
1|10516598|False|41.744107|-87.664494
2|10519196|False|41.780049|-87.000000

## <a name="create-assertion-rules"></a>创建声明规则

通常情况下，将数据与我们合作时清理和准备数据是只是我们需要用于生产的总数据的子集。 因此，某些作为一部分我们清理我们做的假设可能会是 false。 例如，在不断更新数据集中，最初仅包含特定范围内的数字的列可能包含广泛的后续执行中的值。 通常，这些错误将导致中断的管道或错误数据。

管道执行计算的数据创建断言的数据集支持。 这些断言让我们验证是否对数据我们假设继续准确和时，相应地处理故障。

例如，如果想要将限制`Latitude`并`Longitude`你的数据集特定的数值范围中的值[ `assert_value()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#assert-value-columns--multicolumnselection--expression--azureml-dataprep-api-expressions-expression--policy--azureml-dataprep-api-engineapi-typedefinitions-assertpolicy----assertpolicy-errorvalue--1---error-code--str----assertionfailed------azureml-dataprep-api-dataflow-dataflow)方法可确保这始终是这种情况。

```Python
from azureml.dataprep import value
from azureml.core.dataset import Dataset

# get the latest definition of the Dataset
ds_def = dataset.get_definition()

# set assertion rules for `Latitude` and `Longitude` columns
ds_def = ds_def.assert_value('Latitude', (value <= 90) & (value >= -90), error_code='InvalidLatitude')
ds_def = ds_def.assert_value('Longitude', (value <= 180) & (value >= -87), error_code='InvalidLongitude')

ds_def.get_profile()
```

||Type|Min|Max|Count|缺失计数|非缺失计数|缺失百分比|错误计数|空计数|0.1% 分位|1% 分位|5% 分位|25% 分位|50% 分位|75% 分位|95% 分位|99% 分位|99.9% 分位|平均值|标准偏差|Variance|倾斜|峰度
-|----|---|---|-----|-------------|-----------------|---------------|-----------|-----------|-------------|-----------|-----------|------------|------------|------------|------------|------------|--------------|----|------------------|--------|--------|--------
ID|FieldType.INTEGER|1.04986e+07|1.05351e+07|10.0|0.0|10.0|0.0|0.0|0.0|1.04986e+07|1.04992e+07|1.04986e+07|1.05166e+07|1.05209e+07|1.05259e+07|1.05351e+07|1.05351e+07|1.05351e+07|1.05195e+07|12302.7|1.51358e+08|-0.495701|-1.02814
逮捕|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
纬度|FieldType.DECIMAL|41.6928|41.9032|10.0|0.0|10.0|0.0|0.0|0.0|41.6928|41.7185|41.6928|41.78|41.78|41.78|41.9032|41.9032|41.9032|41.78|0.0517107|0.002674|0.837593|1.05
经度|FieldType.INTEGER|-87|-87|10.0|0.0|10.0|0.0|3.0|0.0|-87|-87|-87|-87|-87|-87|-87|-87|-87|-87|0|0|NaN|NaN

从配置文件，请参阅该`Error Count`为`Longitude`列为 3。 下面的代码筛选数据集、 检索错误，并可以看到哪些值将导致断言失败。 从此处，对代码进行调整并适当地清理您的数据。

```Python
from azureml.dataprep import col

ds_error = ds_def.filter(col('Longitude').is_error())
error = ds_error.head(10)['Longitude'][0]

print(error.originalValue)
```

    -87.60431945

## <a name="derive-columns-by-example"></a>按示例派生列

数据集的更高级的工具之一是派生列使用的所需的结果示例的功能。 这样可提供 SDK 示例，以便它可以生成代码以实现预期的转换。

```Python
from azureml.core.dataset import Dataset

# create an in-memory Dataset from a local file
dataset = Dataset.auto_read_files('./data/crime.csv')
dataset.head(3)
```

||ID|案例号|date|阻止|...|
-|---------|-----|---------|----|---
0|10498554|HZ239907|2016-04-04 23:56:00|007XX E 111TH ST|...
第|10516598|HZ258664|2016-04-15 17:00:00|082XX S MARSHFIELD AVE|...
2|10519196|HZ261252|2016-04-15 10:00:00|104XX S SACRAMENTO AVE|...

假设您需要转换的日期和时间格式为"2016年-04-04 晚上 10 点-凌晨 12 点。 在中[ `derive_column_by_example()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#derive-column-by-example-source-columns--sourcecolumns--new-column-name--str--example-data--exampledata-----azureml-dataprep-api-dataflow-dataflow)自变量，提供所需输出中的示例`example_data`参数按以下格式： *（原始输出、 所需的输出）*。

下面的代码提供所需输出的两个示例 ("2016年-04-04 23:56:00"，"2016年-04-04 10 PM-12 AM") 和 ("2016年-04-15 17:00:00"，"2016年-04-15 下午 4 点-下午 6 点")

```Python
ds_def = dataset.get_definition()
ds_def = ds_def.derive_column_by_example(
        source_columns = "Date",
        new_column_name = "Date_Time_Range",
        example_data = [("2016-04-04 23:56:00", "2016-04-04 10PM-12AM"), ("2016-04-15 17:00:00", "2016-04-15 4PM-6PM")]
    )
ds_def.keep_columns(['ID','Date','Date_Time_Range']).head(3)
```

在下表中，请注意，新列，Date_Time_Range 包含记录中指定的格式。

||ID|date|Date_Time_Range
-|--------|-----|----
0|10498554|2016-04-04 23:56:00|2016-04-04 10PM-12AM
1|10516598|2016-04-15 17:00:00|2016-04-15 4PM-6PM
2|10519196|2016-04-15 10:00:00|2016-04-15 10AM-12PM

```Python
# update Dataset definition to keep the transformation steps performed.
dataset = dataset.update_definition(ds_def, 'Derive Date_Time_Range')
```

## <a name="fuzzy-groupings"></a>模糊分组

当从不同的源收集数据时可能会遇到拼写、 大小写、 或缩写相同的实体中的变体。 自动实现标准化和 SDK 的模糊分组或聚类分析文本，功能协调这些变体。

例如，列`inspections.business.city`包含几种形式的城市名称"San Francisco"。

```Python
from azureml.core.dataset import Dataset

# create an in-memory Dataset from a local json file
dataset = Dataset.auto_read_files('./data/city.json')
dataset.head(5)
```

||inspections.business.business_id|inspections.business_name|inspections.business.address|inspections.business.city|...|
-|-----|-------------------------|------------|--|---
0|16162|快速 N Ezee 印度食品|3861 24th St|SF|...
1|67565|泰语的领军人打嗝声 Cafe|1541 TARAVAL St|旧金山|...
2|67565|泰语的领军人打嗝声 Cafe|1541 TARAVAL St|旧金山|...
3|68701|Grindz|832 clement St|SF|...
4|69186|高级支持以迎合和事件，inc.|1255 5 月 22 日 St|S.F.|...

让我们使用[ `fuzzy_group_column()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#fuzzy-group-column-source-column--str--new-column-name--str--similarity-threshold--float---0-8--similarity-score-column-name--str---none-----azureml-dataprep-api-dataflow-dataflow)方法以添加具有"San francisco"自动检测到的规范格式的列。 自变量`source_column`和`new_column_name`所需。 您还可以为：

* 创建一个新列， `similarity_score_column_name`，显示每个对原始值和规范值之间的相似性得分。

* 提供`similarity_threshold`，这是要将组合到一起的值的最低相似性得分。

```Python
# get the latest Dataset definition
ds_def = dataset.get_definition()
ds_def = ds_def.fuzzy_group_column(source_column='inspections.business.city',
                                       new_column_name='city_grouped',
                                       similarity_threshold=0.8,
                                       similarity_score_column_name='similarity_score')
ds_def.head(5)
```

||inspections.business.business_id|inspections.business_name|inspections.business.address|inspections.business.city|city_grouped|similarity_score|...|
-|-----|-------------------------|------------|--|---|---|---
0|16162|快速 N Ezee 印度食品|3861 24th St|SF|旧金山|0.814806|...
第|67565|泰语的领军人打嗝声 Cafe|1541 TARAVAL St|旧金山|旧金山|1.000000|...
2|67565|泰语的领军人打嗝声 Cafe|1541 TARAVAL St|旧金山|旧金山|1.000000|...
3|68701|Grindz|832 clement St|SF|旧金山|0.814806|...
4|69186|高级支持以迎合和事件，inc.|1255 5 月 22 日 St|S.F.|旧金山|0.814806|...

在生成的数据集定义中，不同的数据中表示"旧金山"的所有变体已规范化为相同字符串"旧金山"。

保存此模糊分组单步执行的最新数据集定义`update_definition()`。

```Python
dataset = dataset.update_definition(ds_def, 'fuzzy grouping')
```

## <a name="next-steps"></a>后续步骤

* [管理数据集定义的生命周期](how-to-manage-dataset-definitions.md)。

* 请参阅自动化的机器学习[教程](tutorial-auto-train-models.md)有关回归模型示例。

* 请参阅 SDK[概述](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)有关设计模式和使用情况示例。

* 使用数据集的示例，请参阅[示例笔记本](https://aka.ms/dataset-tutorial)。