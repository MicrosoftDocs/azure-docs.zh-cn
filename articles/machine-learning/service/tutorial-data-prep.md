---
title: 回归模型教程：准备数据
titleSuffix: Azure Machine Learning service
description: 本教程的第一部分介绍如何在 Python 中准备数据，以便使用 Azure 机器学习 SDK 进行回归建模。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sihhu
ms.author: MayMSFT
ms.reviewer: trbye
ms.date: 03/29/2019
ms.custom: seodec18
ms.openlocfilehash: 6e7aa231cabea9fbd76cd028f9fdc698a4dac5d6
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470268"
---
# <a name="tutorial-prepare-data-for-regression-modeling"></a>教程：为回归建模准备数据

在本教程中，你将学习如何使用 [Azure 机器学习数据准备包](https://aka.ms/data-prep-sdk)来为回归建模准备数据。 运行各种转换，以便筛选并组合两个不同的纽约市出租车数据集。

本教程是由两个部分构成的系列教程的第一部分。 完成这一系列的教程以后，即可根据数据特性训练一个模型，以便预测出租车打车费用。 这些特性包括上车日期和时间、乘客数和上车位置。

本教程介绍以下操作：

> [!div class="checklist"]
> * 设置 Python 环境并导入包。
> * 加载两个包含不同字段名称的数据集。
> * 清理数据，删除异常数据。
> * 使用智能转换来转换数据，以便创建新特性。
> * 保存可以在回归模型中使用的数据流对象。

## <a name="prerequisites"></a>先决条件

跳到[设置开发环境](#start)来了解整个 Notebook 设置步骤，或遵照以下说明获取 Notebook 并在 Azure Notebooks 或自己的 Notebook 服务器中运行。 若要运行 Notebook，需要：

* 一个装有以下组件的 Python 3.6 Notebook 服务器：
    * 适用于 Python 的 Azure 机器学习数据准备 SDK
* 教程 Notebook

* 使用[工作区中的云 Notebook 服务器](#azure) 
* 使用[自己的 Notebook 服务器](#server)

### <a name="azure"></a>使用工作区中的云 Notebook 服务器

可以轻松地从你自己的基于云的 Notebook 服务器着手。 创建此云资源后，[用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 就已为你安装和配置了。

[!INCLUDE [aml-azure-notebooks](../../../includes/aml-azure-notebooks.md)]

* 启动 Notebook 网页后，运行 **tutorials/regression-part1-data-prep.ipynb** Notebook。

### <a name="server"></a>使用自己的 Jupyter Notebook 服务器

执行这些步骤，在计算机上创建本地 Jupyter Notebook 服务器。  完成这些步骤后，运行 **tutorials/regression-part1-data-prep.ipynb** Notebook。

1. 完成 [[Azure 机器学习 Python 快速入门](quickstart-run-local-notebook.md)](setup-create-workspace.md#python)中的安装步骤来创建 Miniconda 环境。  如果愿意，可以跳过**创建工作区**部分，但是本教程系列的[第 2 部分](tutorial-auto-train-models.md)将需要使用工作区。
1. 使用 `pip install azureml-dataprep` 在你的环境中安装数据准备 SDK。
1. 克隆 [GitHub 存储库](https://aka.ms/aml-notebooks)。

    ```
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. 从克隆目录启动 notebook 服务器。

    ```shell
    jupyter notebook
    ```

## <a name="start"></a>设置开发环境

开发工作的所有设置都可以在 Python Notebook 中完成。 安装包括以下操作：

* 安装 SDK
* 导入 Python 包

### <a name="install-and-import-packages"></a>安装并导入包

使用以下命令安装必要的包（如果尚未安装）。

```shell
pip install "azureml-dataprep[pandas]>=1.1.0,<1.2.0"
```

导入该 SDK。

```python
import azureml.dataprep as dprep
```

> [!IMPORTANT]
> 确保安装最新版本。 本教程不适用于低于 1.1.0 的版本

## <a name="load-data"></a>加载数据

将两个不同的纽约市出租车数据集下载到数据流对象中。 这两个数据集的字段稍有不同。 `auto_read_file()` 方法自动识别输入文件类型。

```python
from IPython.display import display
dataset_root = "https://dprepdata.blob.core.windows.net/demo"

green_path = "/".join([dataset_root, "green-small/*"])
yellow_path = "/".join([dataset_root, "yellow-small/*"])

green_df_raw = dprep.read_csv(path=green_path, header=dprep.PromoteHeadersMode.GROUPED)
# auto_read_file automatically identifies and parses the file type, which is useful when you don't know the file type.
yellow_df_raw = dprep.auto_read_file(path=yellow_path)

display(green_df_raw.head(5))
display(yellow_df_raw.head(5))
```

`Dataflow` 对象类似于数据帧，表示针对数据执行的一系列松散评估的不可变操作。 可以通过调用不同的转换并筛选可用方法来添加操作。 在 `Dataflow` 中添加操作后，始终会获得新的 `Dataflow` 对象。

## <a name="cleanse-data"></a>清理数据

现在，请在某些变量中填充将要应用到所有数据流的快捷方式转换。 `drop_if_all_null` 变量用于删除所有字段均为 null 的记录。 `useful_columns` 变量保存列说明的数组，这些说明保留在每个数据流中。

```python
all_columns = dprep.ColumnSelector(term=".*", use_regex=True)
drop_if_all_null = [all_columns, dprep.ColumnRelationship(dprep.ColumnRelationship.ALL)]
useful_columns = [
    "cost", "distance", "dropoff_datetime", "dropoff_latitude", "dropoff_longitude",
    "passengers", "pickup_datetime", "pickup_latitude", "pickup_longitude", "store_forward", "vendor"
]
```

首先处理绿色出租车数据，使之具有有效的形状，能够与黄色出租车数据组合在一起。 使用已创建的快捷方式转换变量来调用 `replace_na()`、`drop_nulls()` 和 `keep_columns()` 函数。 另外，请将 dataframe 中的所有列重命名，使之与 `useful_columns` 变量中的名称匹配。


```python
green_df = (green_df_raw
    .replace_na(columns=all_columns)
    .drop_nulls(*drop_if_all_null)
    .rename_columns(column_pairs={
        "VendorID": "vendor",
        "lpep_pickup_datetime": "pickup_datetime",
        "Lpep_dropoff_datetime": "dropoff_datetime",
        "lpep_dropoff_datetime": "dropoff_datetime",
        "Store_and_fwd_flag": "store_forward",
        "store_and_fwd_flag": "store_forward",
        "Pickup_longitude": "pickup_longitude",
        "Pickup_latitude": "pickup_latitude",
        "Dropoff_longitude": "dropoff_longitude",
        "Dropoff_latitude": "dropoff_latitude",
        "Passenger_count": "passengers",
        "Fare_amount": "cost",
        "Trip_distance": "distance"
     })
    .keep_columns(columns=useful_columns))
green_df.head(5)
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vendor</th>
      <th>pickup_datetime</th>
      <th>dropoff_datetime</th>
      <th>store_forward</th>
      <th>pickup_longitude</th>
      <th>pickup_latitude</th>
      <th>dropoff_longitude</th>
      <th>dropoff_latitude</th>
      <th>passengers</th>
      <th>distance</th>
      <th>cost</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>2013-08-01 08:14:37</td>
      <td>2013-08-01 09:09:06</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>.00</td>
      <td>21.25</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2013-08-01 09:13:00</td>
      <td>2013-08-01 11:38:00</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>.00</td>
      <td>74.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>2013-08-01 09:48:00</td>
      <td>2013-08-01 09:49:00</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>.00</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>2013-08-01 10:38:35</td>
      <td>2013-08-01 10:38:51</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>.00</td>
      <td>3.25</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2013-08-01 11:51:45</td>
      <td>2013-08-01 12:03:52</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>.00</td>
      <td>8.5</td>
    </tr>
  </tbody>
</table>
</div>

对黄色出租车数据运行相同的转换步骤。 这些函数确保从数据集中删除 null 数据，这有助于提高机器学习模型的准确度。

```python
yellow_df = (yellow_df_raw
    .replace_na(columns=all_columns)
    .drop_nulls(*drop_if_all_null)
    .rename_columns(column_pairs={
        "vendor_name": "vendor",
        "VendorID": "vendor",
        "vendor_id": "vendor",
        "Trip_Pickup_DateTime": "pickup_datetime",
        "tpep_pickup_datetime": "pickup_datetime",
        "Trip_Dropoff_DateTime": "dropoff_datetime",
        "tpep_dropoff_datetime": "dropoff_datetime",
        "store_and_forward": "store_forward",
        "store_and_fwd_flag": "store_forward",
        "Start_Lon": "pickup_longitude",
        "Start_Lat": "pickup_latitude",
        "End_Lon": "dropoff_longitude",
        "End_Lat": "dropoff_latitude",
        "Passenger_Count": "passengers",
        "passenger_count": "passengers",
        "Fare_Amt": "cost",
        "fare_amount": "cost",
        "Trip_Distance": "distance",
        "trip_distance": "distance"
    })
    .keep_columns(columns=useful_columns))
yellow_df.head(5)
```

针对绿色出租车数据调用 `append_rows()` 函数，以追加黄色出租车数据。 此时会创建新的组合 dataframe。

```python
combined_df = green_df.append_rows([yellow_df])
```

### <a name="convert-types-and-filter"></a>转换类型和筛选器

检查上车和下车的坐标摘要统计信息，了解数据的分布情况。 首先定义一个 `TypeConverter` 对象，将纬度和经度字段更改为十进制类型。 接下来调用 `keep_columns()` 函数，使输出仅限纬度和经度字段，然后调用 `get_profile()` 函数。 这些函数调用创建数据流的精简视图，以便仅显示纬度/经度字段，并更方便地评估缺失的坐标或超出范围的坐标。


```python
decimal_type = dprep.TypeConverter(data_type=dprep.FieldType.DECIMAL)
combined_df = combined_df.set_column_types(type_conversions={
    "pickup_longitude": decimal_type,
    "pickup_latitude": decimal_type,
    "dropoff_longitude": decimal_type,
    "dropoff_latitude": decimal_type
})
combined_df.keep_columns(columns=[
    "pickup_longitude", "pickup_latitude",
    "dropoff_longitude", "dropoff_latitude"
]).get_profile()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Min</th>
      <th>Max</th>
      <th>Count</th>
      <th>缺失计数</th>
      <th>非缺失计数</th>
      <th>缺失百分比</th>
      <th>错误计数</th>
      <th>空计数</th>
      <th>0.1% 分位</th>
      <th>1% 分位</th>
      <th>5% 分位</th>
      <th>25% 分位</th>
      <th>50% 分位</th>
      <th>75% 分位</th>
      <th>95% 分位</th>
      <th>99% 分位</th>
      <th>99.9% 分位</th>
      <th>标准偏差</th>
      <th>平均值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>pickup_longitude</th>
      <td>FieldType.DECIMAL</td>
      <td>-115.179337</td>
      <td>0.000000</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-88.114046</td>
      <td>-73.961840</td>
      <td>-73.961964</td>
      <td>-73.947693</td>
      <td>-73.922097</td>
      <td>-73.846670</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>18.792672</td>
      <td>-68.833579</td>
    </tr>
    <tr>
      <th>pickup_latitude</th>
      <td>FieldType.DECIMAL</td>
      <td>0.000000</td>
      <td>40.919121</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>40.682889</td>
      <td>40.675541</td>
      <td>40.721075</td>
      <td>40.756159</td>
      <td>40.803909</td>
      <td>40.849406</td>
      <td>40.870681</td>
      <td>40.891244</td>
      <td>10.345967</td>
      <td>37.936742</td>
    </tr>
    <tr>
      <th>dropoff_longitude</th>
      <td>FieldType.DECIMAL</td>
      <td>-115.179337</td>
      <td>0.000000</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-87.699611</td>
      <td>-73.984734</td>
      <td>-73.985777</td>
      <td>-73.956250</td>
      <td>-73.928948</td>
      <td>-73.866208</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>18.696526</td>
      <td>-68.896978</td>
    </tr>
    <tr>
      <th>dropoff_latitude</th>
      <td>FieldType.DECIMAL</td>
      <td>0.000000</td>
      <td>41.008934</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>7722.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>40.662763</td>
      <td>40.654851</td>
      <td>40.717821</td>
      <td>40.756534</td>
      <td>40.784688</td>
      <td>40.852437</td>
      <td>40.879289</td>
      <td>40.937291</td>
      <td>10.290780</td>
      <td>37.963774</td>
    </tr>
  </tbody>
</table>



从摘要统计信息输出中可以看到，有的坐标缺失，有的坐标不在纽约市（通过主观分析来确定）。 筛选掉位置在城市边缘之外的坐标。 链接 `filter()` 函数中的列筛选器命令，定义每个字段的最小和最大边界。 然后再次调用 `get_profile()` 函数，验证转换。


```python
latlong_filtered_df = (combined_df
    .drop_nulls(
        columns=["pickup_longitude", "pickup_latitude", "dropoff_longitude", "dropoff_latitude"],
        column_relationship=dprep.ColumnRelationship(dprep.ColumnRelationship.ANY)
    )
    .filter(dprep.f_and(
        dprep.col("pickup_longitude") <= -73.72,
        dprep.col("pickup_longitude") >= -74.09,
        dprep.col("pickup_latitude") <= 40.88,
        dprep.col("pickup_latitude") >= 40.53,
        dprep.col("dropoff_longitude") <= -73.72,
        dprep.col("dropoff_longitude") >= -74.09,
        dprep.col("dropoff_latitude") <= 40.88,
        dprep.col("dropoff_latitude") >= 40.53
    )))
latlong_filtered_df.keep_columns(columns=[
    "pickup_longitude", "pickup_latitude",
    "dropoff_longitude", "dropoff_latitude"
]).get_profile()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Min</th>
      <th>Max</th>
      <th>Count</th>
      <th>缺失计数</th>
      <th>非缺失计数</th>
      <th>缺失百分比</th>
      <th>错误计数</th>
      <th>空计数</th>
      <th>0.1% 分位</th>
      <th>1% 分位</th>
      <th>5% 分位</th>
      <th>25% 分位</th>
      <th>50% 分位</th>
      <th>75% 分位</th>
      <th>95% 分位</th>
      <th>99% 分位</th>
      <th>99.9% 分位</th>
      <th>标准偏差</th>
      <th>平均值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>pickup_longitude</th>
      <td>FieldType.DECIMAL</td>
      <td>-74.078156</td>
      <td>-73.736481</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-74.076314</td>
      <td>-73.962542</td>
      <td>-73.962893</td>
      <td>-73.948975</td>
      <td>-73.927856</td>
      <td>-73.866662</td>
      <td>-73.830438</td>
      <td>-73.823160</td>
      <td>-73.769750</td>
      <td>0.048711</td>
      <td>-73.913865</td>
    </tr>
    <tr>
      <th>pickup_latitude</th>
      <td>FieldType.DECIMAL</td>
      <td>40.575485</td>
      <td>40.879852</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>40.632884</td>
      <td>40.713105</td>
      <td>40.711600</td>
      <td>40.721403</td>
      <td>40.758142</td>
      <td>40.805145</td>
      <td>40.848855</td>
      <td>40.867567</td>
      <td>40.877690</td>
      <td>0.048348</td>
      <td>40.765226</td>
    </tr>
    <tr>
      <th>dropoff_longitude</th>
      <td>FieldType.DECIMAL</td>
      <td>-74.085747</td>
      <td>-73.720871</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-74.078828</td>
      <td>-73.985650</td>
      <td>-73.985813</td>
      <td>-73.959041</td>
      <td>-73.936681</td>
      <td>-73.884846</td>
      <td>-73.815507</td>
      <td>-73.776697</td>
      <td>-73.733471</td>
      <td>0.055961</td>
      <td>-73.920718</td>
    </tr>
    <tr>
      <th>dropoff_latitude</th>
      <td>FieldType.DECIMAL</td>
      <td>40.583530</td>
      <td>40.879734</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>7059.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>40.597741</td>
      <td>40.695376</td>
      <td>40.695115</td>
      <td>40.727549</td>
      <td>40.758160</td>
      <td>40.788378</td>
      <td>40.850372</td>
      <td>40.867968</td>
      <td>40.878586</td>
      <td>0.050462</td>
      <td>40.759487</td>
    </tr>
  </tbody>
</table>

### <a name="split-and-rename-columns"></a>拆分和重命名列

查看 `store_forward` 列的数据配置文件。 此字段是一个布尔标志。如果出租车在完成行程后未连接到服务器，因此必须先将行程数据存储在内存中，然后在连接后将数据转发到服务器，则此标志的值为 `Y`。


```python
latlong_filtered_df.keep_columns(columns='store_forward').get_profile()
```




<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Min</th>
      <th>Max</th>
      <th>Count</th>
      <th>缺失计数</th>
      <th>非缺失计数</th>
      <th>缺失百分比</th>
      <th>错误计数</th>
      <th>空计数</th>
      <th>0.1% 分位</th>
      <th>1% 分位</th>
      <th>5% 分位</th>
      <th>25% 分位</th>
      <th>50% 分位</th>
      <th>75% 分位</th>
      <th>95% 分位</th>
      <th>99% 分位</th>
      <th>99.9% 分位</th>
      <th>标准偏差</th>
      <th>平均值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>store_forward</th>
      <td>FieldType.STRING</td>
      <td>N</td>
      <td>Y</td>
      <td>7059.0</td>
      <td>99.0</td>
      <td>6960.0</td>
      <td>0.014025</td>
      <td>0.0</td>
      <td>0.0</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>



注意，`store_forward` 列中的数据配置文件输出显示，数据不一致且存在缺失值或 null 值。 使用 `replace()` 和 `fill_nulls()` 函数将这些值替换为字符串“N”：


```python
replaced_stfor_vals_df = latlong_filtered_df.replace(columns="store_forward", find="0", replace_with="N").fill_nulls("store_forward", "N")
```

执行 `distance` 字段上的 `replace` 函数。 此函数将重新格式化错误标记为 `.00` 的距离值，并用 0 填充任何 null 值。 将 `distance` 字段转换为数值格式。 这些错误的数据点可能是出租车内部数据收集系统中的异常导致。


```python
replaced_distance_vals_df = replaced_stfor_vals_df.replace(columns="distance", find=".00", replace_with=0).fill_nulls("distance", 0)
replaced_distance_vals_df = replaced_distance_vals_df.to_number(["distance"])
```

将上车和下车日期/时间值拆分为相应的日期和时间列。 使用 `split_column_by_example()` 函数进行拆分。 在此示例中，`split_column_by_example()` 函数的可选 `example` 参数已省略。 因此，函数会自动根据数据来确定在何处进行拆分。


```python
time_split_df = (replaced_distance_vals_df
    .split_column_by_example(source_column="pickup_datetime")
    .split_column_by_example(source_column="dropoff_datetime"))
time_split_df.head(5)
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vendor</th>
      <th>pickup_datetime</th>
      <th>pickup_datetime_1</th>
      <th>pickup_datetime_2</th>
      <th>dropoff_datetime</th>
      <th>dropoff_datetime_1</th>
      <th>dropoff_datetime_2</th>
      <th>store_forward</th>
      <th>pickup_longitude</th>
      <th>pickup_latitude</th>
      <th>dropoff_longitude</th>
      <th>dropoff_latitude</th>
      <th>passengers</th>
      <th>distance</th>
      <th>cost</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>2013-08-01 17:22:00</td>
      <td>2013-08-01</td>
      <td>17:22:00</td>
      <td>2013-08-01 17:22:00</td>
      <td>2013-08-01</td>
      <td>17:22:00</td>
      <td>N</td>
      <td>-73.937767</td>
      <td>40.758480</td>
      <td>-73.937767</td>
      <td>40.758480</td>
      <td>1</td>
      <td>0.0</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2013-08-01 17:24:00</td>
      <td>2013-08-01</td>
      <td>17:24:00</td>
      <td>2013-08-01 17:25:00</td>
      <td>2013-08-01</td>
      <td>17:25:00</td>
      <td>N</td>
      <td>-73.937927</td>
      <td>40.757843</td>
      <td>-73.937927</td>
      <td>40.757843</td>
      <td>1</td>
      <td>0.0</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>2013-08-06 06:51:19</td>
      <td>2013-08-06</td>
      <td>06:51:19</td>
      <td>2013-08-06 06:51:36</td>
      <td>2013-08-06</td>
      <td>06:51:36</td>
      <td>N</td>
      <td>-73.937721</td>
      <td>40.758404</td>
      <td>-73.937721</td>
      <td>40.758369</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>2013-08-06 13:26:34</td>
      <td>2013-08-06</td>
      <td>13:26:34</td>
      <td>2013-08-06 13:26:57</td>
      <td>2013-08-06</td>
      <td>13:26:57</td>
      <td>N</td>
      <td>-73.937691</td>
      <td>40.758419</td>
      <td>-73.937790</td>
      <td>40.758358</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2013-08-06 13:27:53</td>
      <td>2013-08-06</td>
      <td>13:27:53</td>
      <td>2013-08-06 13:28:08</td>
      <td>2013-08-06</td>
      <td>13:28:08</td>
      <td>N</td>
      <td>-73.937805</td>
      <td>40.758396</td>
      <td>-73.937775</td>
      <td>40.758450</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
  </tbody>
</table>
</div>

将 `split_column_by_example()` 函数生成的列重命名，使用有意义的名称。

```python
renamed_col_df = (time_split_df
    .rename_columns(column_pairs={
        "pickup_datetime_1": "pickup_date",
        "pickup_datetime_2": "pickup_time",
        "dropoff_datetime_1": "dropoff_date",
        "dropoff_datetime_2": "dropoff_time"
    }))
renamed_col_df.head(5)
```

调用 `get_profile()` 函数，以查看完成所有清理步骤之后的完整摘要统计信息。

```python
renamed_col_df.get_profile()
```

## <a name="transform-data"></a>转换数据

将上车和下车日期进一步拆分为星期几、月份日期和月份值。 若要确定是星期几，请使用 `derive_column_by_example()` 函数。 此函数采用示例对象的数组参数，这些对象定义输入数据和首选输出。 此函数会自动确定你的首选转换。 对于上车和下车时间列，请使用不带示例参数的 `split_column_by_example()` 函数将时间拆分为小时、分钟和秒。

生成这些新特性后，请通过 `drop_columns()` 函数删除原始字段，因为新生成的特性是首选特性。 重命名其余字段，使用有意义的说明。

以这种方式转换数据来创建基于时间的新特征可以提高机器学习模型的准确度。 例如，为工作日生成新特征有助于在星期日期与出租车费用价格之间建立关系，因为某些星期日期的用车需求更高，其价格也更高。


```python
transformed_features_df = (renamed_col_df
    .derive_column_by_example(
        source_columns="pickup_date",
        new_column_name="pickup_weekday",
        example_data=[("2009-01-04", "Sunday"), ("2013-08-22", "Thursday")]
    )
    .derive_column_by_example(
        source_columns="dropoff_date",
        new_column_name="dropoff_weekday",
        example_data=[("2013-08-22", "Thursday"), ("2013-11-03", "Sunday")]
    )

    .split_column_by_example(source_column="pickup_time")
    .split_column_by_example(source_column="dropoff_time")
    # The following two calls to split_column_by_example reference the column names generated from the previous two calls.
    .split_column_by_example(source_column="pickup_time_1")
    .split_column_by_example(source_column="dropoff_time_1")
    .drop_columns(columns=[
        "pickup_date", "pickup_time", "dropoff_date", "dropoff_time",
        "pickup_date_1", "dropoff_date_1", "pickup_time_1", "dropoff_time_1"
    ])

    .rename_columns(column_pairs={
        "pickup_date_2": "pickup_month",
        "pickup_date_3": "pickup_monthday",
        "pickup_time_1_1": "pickup_hour",
        "pickup_time_1_2": "pickup_minute",
        "pickup_time_2": "pickup_second",
        "dropoff_date_2": "dropoff_month",
        "dropoff_date_3": "dropoff_monthday",
        "dropoff_time_1_1": "dropoff_hour",
        "dropoff_time_1_2": "dropoff_minute",
        "dropoff_time_2": "dropoff_second"
    }))

transformed_features_df.head(5)
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>vendor</th>
      <th>pickup_datetime</th>
      <th>pickup_weekday</th>
      <th>pickup_hour</th>
      <th>pickup_minute</th>
      <th>pickup_second</th>
      <th>dropoff_datetime</th>
      <th>dropoff_weekday</th>
      <th>dropoff_hour</th>
      <th>dropoff_minute</th>
      <th>dropoff_second</th>
      <th>store_forward</th>
      <th>pickup_longitude</th>
      <th>pickup_latitude</th>
      <th>dropoff_longitude</th>
      <th>dropoff_latitude</th>
      <th>passengers</th>
      <th>distance</th>
      <th>cost</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>2013-08-01 17:22:00</td>
      <td>星期四</td>
      <td>17</td>
      <td>22</td>
      <td>00</td>
      <td>2013-08-01 17:22:00</td>
      <td>星期四</td>
      <td>17</td>
      <td>22</td>
      <td>00</td>
      <td>N</td>
      <td>-73.937767</td>
      <td>40.758480</td>
      <td>-73.937767</td>
      <td>40.758480</td>
      <td>1</td>
      <td>0.0</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2013-08-01 17:24:00</td>
      <td>星期四</td>
      <td>17</td>
      <td>24</td>
      <td>00</td>
      <td>2013-08-01 17:25:00</td>
      <td>星期四</td>
      <td>17</td>
      <td>25</td>
      <td>00</td>
      <td>N</td>
      <td>-73.937927</td>
      <td>40.757843</td>
      <td>-73.937927</td>
      <td>40.757843</td>
      <td>1</td>
      <td>0.0</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>2013-08-06 06:51:19</td>
      <td>星期二</td>
      <td>06</td>
      <td>51</td>
      <td>19</td>
      <td>2013-08-06 06:51:36</td>
      <td>星期二</td>
      <td>06</td>
      <td>51</td>
      <td>36</td>
      <td>N</td>
      <td>-73.937721</td>
      <td>40.758404</td>
      <td>-73.937721</td>
      <td>40.758369</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>2013-08-06 13:26:34</td>
      <td>星期二</td>
      <td>13</td>
      <td>26</td>
      <td>34</td>
      <td>2013-08-06 13:26:57</td>
      <td>星期二</td>
      <td>13</td>
      <td>26</td>
      <td>57</td>
      <td>N</td>
      <td>-73.937691</td>
      <td>40.758419</td>
      <td>-73.937790</td>
      <td>40.758358</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2013-08-06 13:27:53</td>
      <td>星期二</td>
      <td>13</td>
      <td>27</td>
      <td>53</td>
      <td>2013-08-06 13:28:08</td>
      <td>星期二</td>
      <td>13</td>
      <td>28</td>
      <td>08</td>
      <td>N</td>
      <td>-73.937805</td>
      <td>40.758396</td>
      <td>-73.937775</td>
      <td>40.758450</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.3</td>
    </tr>
  </tbody>
</table>
</div>

注意，数据显示通过派生的转换生成的上车和下车日期和时间组件是正确的。 删除 `pickup_datetime` 和 `dropoff_datetime` 列，因为已不再需要它们（小时、分钟和秒等粒度时间特征更适合用于模型训练）。


```python
processed_df = transformed_features_df.drop_columns(columns=["pickup_datetime", "dropoff_datetime"])
```

请使用类型推断功能自动检查每个字段的数据类型，并显示推断结果。


```python
type_infer = processed_df.builders.set_column_types()
type_infer.learn()
type_infer
```

`type_infer` 生成的输出如下所示。

    Column types conversion candidates:
    'pickup_weekday': [FieldType.STRING],
    'pickup_hour': [FieldType.DECIMAL],
    'pickup_minute': [FieldType.DECIMAL],
    'pickup_second': [FieldType.DECIMAL],
    'dropoff_hour': [FieldType.DECIMAL],
    'dropoff_minute': [FieldType.DECIMAL],
    'dropoff_second': [FieldType.DECIMAL],
    'store_forward': [FieldType.STRING],
    'pickup_longitude': [FieldType.DECIMAL],
    'dropoff_longitude': [FieldType.DECIMAL],
    'passengers': [FieldType.DECIMAL],
    'distance': [FieldType.DECIMAL],
    'vendor': [FieldType.STRING],
    'dropoff_weekday': [FieldType.STRING],
    'pickup_latitude': [FieldType.DECIMAL],
    'dropoff_latitude': [FieldType.DECIMAL],
    'cost': [FieldType.DECIMAL]

根据数据来看，推断结果看起来是正确的。 现在请将类型转换应用到数据流。


```python
type_converted_df = type_infer.to_dataflow()
type_converted_df.get_profile()
```

在打包数据流之前，请对数据集运行两个最终筛选器。 若要消除错误捕获的数据点，请根据 `cost` 和 `distance` 变量值都大于零的记录筛选数据流。 此步骤可以大幅提高机器学习模型的准确度，因为成本或距离为零的数据点表示严重离群值，会导致丧失预测准确度。

```python
final_df = type_converted_df.filter(dprep.col("distance") > 0)
final_df = final_df.filter(dprep.col("cost") > 0)
```

目前已经有了完全转换和准备好的数据流对象，可以在机器学习模型中使用了。 SDK 包含对象序列化功能，其用法如以下代码中所示。

```python
import os

file_path = os.path.join(os.getcwd(), "dflows.dprep")
final_df.save(file_path)
```

## <a name="clean-up-resources"></a>清理资源

若要继续学习教程第二部分，需要当前目录中的 **dflows.dprep** 文件。

如果不打算继续学习第二部分，请删除当前目录中的 **dflows.dprep** 文件。 不管是在本地还是在 [Azure Notebooks](https://notebooks.azure.com/) 中运行此执行操作，均请删除此文件。

## <a name="next-steps"></a>后续步骤

在本教程的第一部分，你完成了以下操作：

> [!div class="checklist"]
> * 设置开发环境。
> * 加载并清理数据集。
> * 使用智能转换根据示例来预测逻辑。
> * 合并并打包用于机器学习训练的数据集。

现在可以在本教程的第二部分使用此训练数据了：

> [!div class="nextstepaction"]
> [教程（第二部分）：训练回归模型](tutorial-auto-train-models.md)
