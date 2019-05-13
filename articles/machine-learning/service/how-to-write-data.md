---
title: 写入：数据准备 Python SDK
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习数据准备 SDK 写入数据。 可在数据流中的任何位置写出数据，也可以将数据写入我们支持的任何位置（本地文件系统、Azure Blob 存储和 Azure Data Lake Storage）的文件中。
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
ms.openlocfilehash: 0275d27a0a27d0279886f6f7fd15b14d312a44ea
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472005"
---
# <a name="write-and-configure-data--with-the-azure-machine-learning-data-prep-sdk"></a>编写并使用 Azure 机器学习数据准备 SDK 配置数据

在本文中，你将了解用于写入数据使用不同的方法[Azure 机器学习数据准备 Python SDK](https://aka.ms/data-prep-sdk)以及如何配置该数据以供试验使用[Azure 机器学习 SDK for Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).  可以在数据流中的任何点写入输出数据。 写入会添加到生成的数据流，步骤和步骤运行每次运行数据流。 数据写入多个分区文件以实现并行写入。

> [!Important]
> 如果要构建一个新的解决方案，请尝试[Azure 机器学习数据集](how-to-explore-prepare-data.md)（预览版） 来转换数据、 快照数据和存储版本控制的数据集定义。 数据集是数据准备 SDK，提供可用于管理 AI 解决方案中的数据集的扩展的功能的下一个版本。
> 如果您使用`azureml-dataprep`包来创建数据流而不是使用转换与`azureml-datasets`的包装，以创建数据集，您将不能以供以后使用快照或版本控制的数据集。

由于对管道中有多少写步骤没有限制，因此你可以轻松添加更多写步骤来获取中间结果以用于故障排除或用于其他管道。

每次运行写步骤时，都会完全拉取数据流中的数据。 例如，具有三个写步骤的数据流将三次读取并处理数据集中的每个记录。

## <a name="supported-data-types-and-location"></a>支持的数据类型和位置

支持以下文件格式
-   带分隔符的文件（CSV、TSV 等）
-   Parquet 文件

使用 Azure 机器学习数据准备 Python SDK，可以写入到的数据：
+ 本地文件系统
+ Azure Blob 存储
+ Azure Data Lake Storage

## <a name="spark-considerations"></a>Spark 注意事项

在 Spark 中运行数据流时，必须写入空文件夹。 尝试运行向现有文件夹进行写入的操作将会失败。 确保目标文件夹为空，或为每次运行使用不同的目标位置，否则写入将失败。

## <a name="monitoring-write-operations"></a>监视写入操作

为方便你进行监视，在写入完成后，会生成一个名为 SUCCESS 的哨兵文件。 该文件可帮助你确定中间写入的完成时间，而无需等待整个管道的完成。

## <a name="example-write-code"></a>示例写入代码

对于此示例中，首先将数据加载到数据流使用`auto_read_file()`。 你将通过不同格式重复使用此数据。

```python
import azureml.dataprep as dprep
t = dprep.auto_read_file('./data/fixed_width_file.txt')
t = t.to_number('Column3')
t.head(5)
```

示例输出：

| | Column1 | Column2 | Column3 | Column4 | Column5 | Column6 | Column7 | Column8 | Column9 |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
|0| 10000.0 | 99999.0 | 无 | 否 | 否 | ENRS | NaN | NaN | NaN |   
|第| 10003.0 | 99999.0 | 无 | 否 | 否 | ENSO | NaN | NaN | NaN |   
|2| 10010.0 | 99999.0 | 无 | 否 | JN | ENJA | 70933.0 | -8667.0 | 90.0 |
|3| 10013.0 | 99999.0 | 无 | 否 | 否 |      | NaN | NaN | NaN |
|4| 10014.0 | 99999.0 | 无 | 否 | 否 | ENSO | 59783.0 | 5350.0 |  500.0|

### <a name="delimited-file-example"></a>带分隔符的文件示例

下面的代码使用[ `write_to_csv()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#write-to-csv-directory-path--destinationpath--separator--str--------na--str----na---error--str----error------azureml-dataprep-api-dataflow-dataflow)函数将数据写入到带分隔符的文件。

```python
# Create a new data flow using `write_to_csv` 
write_t = t.write_to_csv(directory_path=dprep.LocalFileOutput('./test_out/'))

# Run the data flow to begin the write operation.
write_t.run_local()

written_files = dprep.read_csv('./test_out/part-*')
written_files.head(5)
```

示例输出：

| | Column1 | Column2 | Column3 | Column4 | Column5 | Column6 | Column7 | Column8 | Column9 |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
|0| 10000.0 | 99999.0 | 错误 | 否 | 否 | ENRS | NaN    | NaN | NaN |   
|第| 10003.0 | 99999.0 | 错误 | 否 | 否 | ENSO |    NaN | NaN | NaN |   
|2| 10010.0 | 99999.0 | 错误 | 否 | JN | ENJA |    70933.0 | -8667.0 | 90.0 |
|3| 10013.0 | 99999.0 | 错误 | 否 | 否 |     | NaN | NaN | NaN |
|4| 10014.0 | 99999.0 | 错误 | 否 | 否 | ENSO |    59783.0 | 5350.0 |  500.0|

在前面的输出中，由于未正确分析数字，数值列中出现了多个错误。 默认情况下，写入 CSV 时，NULL 值将替换为字符串“ERROR”。

在写入调用中添加参数，并指定一个字符串用于表示 NULL 值。

```python
write_t = t.write_to_csv(directory_path=dprep.LocalFileOutput('./test_out/'), 
                         error='BadData',
                         na='NA')
write_t.run_local()
written_files = dprep.read_csv('./test_out/part-*')
written_files.head(5)
```

前面的代码生成了以下输出：

| | Column1 | Column2 | Column3 | Column4 | Column5 | Column6 | Column7 | Column8 | Column9 |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
|0| 10000.0 | 99999.0 | BadData | 否 | 否 | ENRS | NaN  | NaN | NaN |   
|第| 10003.0 | 99999.0 | BadData | 否 | 否 | ENSO |  NaN | NaN | NaN |   
|2| 10010.0 | 99999.0 | BadData | 否 | JN | ENJA |  70933.0 | -8667.0 | 90.0 |
|3| 10013.0 | 99999.0 | BadData | 否 | 否 |   | NaN | NaN | NaN |
|4| 10014.0 | 99999.0 | BadData | 否 | 否 | ENSO |  59783.0 | 5350.0 |  500.0|

### <a name="parquet-file-example"></a>Parquet 文件示例

类似于`write_to_csv()`，则[ `write_to_parquet()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#write-to-parquet-file-path--typing-union--destinationpath--nonetype----none--directory-path--typing-union--destinationpath--nonetype----none--single-file--bool---false--error--str----error---row-groups--int---0-----azureml-dataprep-api-dataflow-dataflow)函数将返回写入 Parquet 数据流运行时执行的步骤与新的数据流。

```python
write_parquet_t = t.write_to_parquet(directory_path=dprep.LocalFileOutput('./test_parquet_out/'),
error='MiscreantData')
```

运行数据流来启动写入操作。

```python
write_parquet_t.run_local()

written_parquet_files = dprep.read_parquet_file('./test_parquet_out/part-*')
written_parquet_files.head(5)
```

前面的代码生成了以下输出：

|   | Column1 | Column2 | Column3 | Column4 | Column5 | Column6 | Column7 | Column8 | Column9 |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |-------- |
|0| 10000.0 | 99999.0 | MiscreantData | 否 | 否 | ENRS | MiscreantData | MiscreantData | MiscreantData |
|第| 10003.0 | 99999.0 | MiscreantData | 否 | 否 | ENSO | MiscreantData | MiscreantData | MiscreantData |   
|2| 10010.0 | 99999.0 | MiscreantData | 否| JN| ENJA|   70933.0|    -8667.0 |90.0|
|3| 10013.0 | 99999.0 | MiscreantData | 否| 否| |   MiscreantData|    MiscreantData|    MiscreantData|
|4| 10014.0 | 99999.0 | MiscreantData | 否| 否| ENSO|   59783.0|    5350.0| 500.0|

## <a name="configure-data-for-automated-machine-learning-training"></a>配置用于自动化的机器学习训练的数据

传递到您新写入的数据文件[ `AutoMLConfig` ](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py#automlconfig)对象为自动化的机器学习训练做准备。 

下面的代码示例说明了如何将您的数据流转换为 Pandas 数据帧和随后，将其拆分为训练和测试自动化的机器学习训练数据集。

```Python
from azureml.train.automl import AutoMLConfig
from sklearn.model_selection import train_test_split

dflow = dprep.auto_read_file(path="")
X_dflow = dflow.keep_columns([feature_1,feature_2, feature_3])
y_dflow = dflow.keep_columns("target")

X_df = X_dflow.to_pandas_dataframe()
y_df = y_dflow.to_pandas_dataframe()

X_train, X_test, y_train, y_test = train_test_split(X_df, y_df, test_size=0.2, random_state=223)

# flatten y_train to 1d array
y_train.values.flatten()

#configure 
automated_ml_config = AutoMLConfig(task = 'regression',
                               X = X_train.values,  
                   y = y_train.values.flatten(),
                   iterations = 30,
                       Primary_metric = "AUC_weighted",
                       n_cross_validation = 5
                       )

```

如果不需要任何中间数据准备步骤类似于前面的示例中，可以传递直接插入到数据流`AutoMLConfig`。

```Python
automated_ml_config = AutoMLConfig(task = 'regression', 
                   X = X_dflow,   
                   y = y_dflow, 
                   iterations = 30, 
                   Primary_metric = "AUC_weighted",
                   n_cross_validation = 5
                   )
```

## <a name="next-steps"></a>后续步骤
* 请参阅 SDK[概述](https://aka.ms/data-prep-sdk)有关设计模式和使用情况示例 
* 请参阅自动化的机器学习[教程](tutorial-auto-train-models.md)有关回归模型示例
