---
title: 数据集定义和版本控制
titleSuffix: Azure Machine Learning service
description: 了解如何更新数据集定义和管理定义的生命周期
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.reviewer: larryfr
ms.date: 05/02/2019
ms.openlocfilehash: 7c861c8cdc9985caa42bd2beb5236a4f4e93e4c7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028675"
---
# <a name="update-and-manage-the-lifecycle-of-dataset-definitions"></a>更新和管理数据集定义的生命周期

了解如何更新和管理 Azure 机器学习数据集 （预览版） 使用的数据集定义。

数据集定义是用于准备你的数据集，其中包括对您的源数据和执行的转换步骤的这两个引用的方案。 一个数据集可以有多个定义，并且每个定义都有其自身的生命周期。

下面的方案是使用数据集定义的示例：

1. 生成机器学习使用你的数据集的当前定义的管道。
1. 基础数据的更改;已添加新属性。
1. 创建你将添加额外的转换，以处理新的属性的定义的新版本。

在此示例中，现有管道将继续使用原始定义版本。 新的定义版本可用于新方案，例如定型模型或创建管道。

## <a name="prerequisites"></a>必备组件

需要具有 Azure 订阅和要注册你的数据集，以便管理数据集定义的生命周期的工作区。

本文档中的示例中使用的示例文件位于[ https://dprepdata.blob.core.windows.net/dataset-sample-files/crime.csv ](https://dprepdata.blob.core.windows.net/dataset-sample-files/crime.csv)。

## <a name="update-dataset-definitions"></a>更新数据集定义

首先，我们将创建并注册你的工作区的数据集。

```python
from azureml.core import Workspace, Datastore, Dataset

# change the configuration for workspace and Datastore
subscription_id = 'your subscription id'
resource_group = 'your resource group name'
workspace_name = 'your workspace name'
datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace(subscription_id, resource_group, workspace_name)

# get a Datastore that has already been created in the workspace
dstore = Datastore.get(workspace, datastore_name)

# create an in-memory Dataset from Datastore
datapath = dstore.path('data/src/crime.csv')
dataset = Dataset.from_delimited_files(datapath)

# register the Dataset with the workspace. if a Dataset with the same name already exists, retrieve it by turning `exist_ok = True`
dataset_name = 'crime dataset'
dataset = dataset.register(workspace = workspace, 
                 name = dataset_name, 
                 description = 'crime dataset for demo', 
                 tags = {'year':'2019', 'month':'Apr'},
                 exist_ok = True)
```

第一个数据集定义 (`version_id` = 1) 创建数据集时创建。 然后每次更新数据集定义，新 version_id 将分配到的最新定义。

```python
# get the Dataset from the workspace by name
dataset = workspace.datasets['dataset_name']

# get the latest Dataset definition
ds_def = dataset.get_definition()

# update the Dataset definition to select only the columns I'm interested in
ds_def = ds_def.keep_columns(['ID', 'Arrest', 'Latitude', 'Longitude'])

# with this update, the latest definition for dataset now has `version_id = 2`
dataset = dataset.update_definition(ds_def, 'select useful column')

dataset = workspace.datasets['dataset_name']
dataset.head(5)
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
      <th>ID</th>
      <th>逮捕</th>
      <th>纬度</th>
      <th>经度</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10498554</td>
      <td>FALSE</td>
      <td>41.69283384</td>
      <td>-87.60431945</td>
    </tr>
    <tr>
      <th>第</th>
      <td>10516598</td>
      <td>FALSE</td>
      <td>41.74410697</td>
      <td>-87.66449429</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10519196</td>
      <td>FALSE</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>10519591</td>
      <td>FALSE</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>10534446</td>
      <td>FALSE</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>

此定义更新不会清除以前的数据集定义。 您仍可以访问并使用前面的定义。

```python
# specify `version_id` to get a previous definition 
ds_def_old = dataset.get_definition(version_id = 1)
ds_def_old.head(5)
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
      <th>ID</th>
      <th>案例号</th>
      <th>date</th>
      <th>街区</th>
      <th>IUCR</th>
      <th>主要类型</th>
      <th>描述</th>
      <th>地址说明</th>
      <th>逮捕</th>
      <th>国内</th>
      <th>...</th>
      <th>病房</th>
      <th>社区范围</th>
      <th>FBI 代码</th>
      <th>X 坐标</th>
      <th>Y 坐标</th>
      <th>年龄</th>
      <th>更新时间</th>
      <th>纬度</th>
      <th>经度</th>
      <th>Location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10498554</td>
      <td>HZ239907</td>
      <td>4/4/2016 23:56</td>
      <td>007XX E 111TH ST</td>
      <td>1153</td>
      <td>欺骗性的做法</td>
      <td>通过 300 美元的财务身份盗窃</td>
      <td>OTHER</td>
      <td>FALSE</td>
      <td>FALSE</td>
      <td>...</td>
      <td>9</td>
      <td>50</td>
      <td>11</td>
      <td>1183356</td>
      <td>1831503</td>
      <td>2016</td>
      <td>5/11/2016 15:48</td>
      <td>41.69283384</td>
      <td>-87.60431945</td>
      <td>(41.692833841, -87.60431945)</td>
    </tr>
    <tr>
      <th>第</th>
      <td>10516598</td>
      <td>HZ258664</td>
      <td>4/15/2016 17:00</td>
      <td>082XX S MARSHFIELD AVE</td>
      <td>890</td>
      <td>THEFT</td>
      <td>从构建</td>
      <td>未成年人</td>
      <td>FALSE</td>
      <td>FALSE</td>
      <td>...</td>
      <td>21</td>
      <td>71</td>
      <td>6</td>
      <td>1166776</td>
      <td>1850053</td>
      <td>2016</td>
      <td>5/12/2016 15:48</td>
      <td>41.74410697</td>
      <td>-87.66449429</td>
      <td>(41.744106973, -87.664494285)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10519196</td>
      <td>HZ261252</td>
      <td>4/15/2016 10:00</td>
      <td>104XX S SACRAMENTO AVE</td>
      <td>1154</td>
      <td>欺骗性的做法</td>
      <td>财务身份盗窃 300 美元并在列表视图</td>
      <td>未成年人</td>
      <td>FALSE</td>
      <td>FALSE</td>
      <td>...</td>
      <td>19</td>
      <td>74</td>
      <td>11</td>
      <td></td>
      <td></td>
      <td>2016</td>
      <td>5/12/2016 15:50</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>10519591</td>
      <td>HZ261534</td>
      <td>4/15/2016 9:00</td>
      <td>113XX S PRAIRIE AVE</td>
      <td>1120</td>
      <td>欺骗性的做法</td>
      <td>伪造</td>
      <td>未成年人</td>
      <td>FALSE</td>
      <td>FALSE</td>
      <td>...</td>
      <td>9</td>
      <td>49</td>
      <td>10</td>
      <td></td>
      <td></td>
      <td>2016</td>
      <td>5/13/2016 15:51</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>10534446</td>
      <td>HZ277630</td>
      <td>4/15/2016 10:00</td>
      <td>055XX KEDZIE 保存 N</td>
      <td>890</td>
      <td>THEFT</td>
      <td>从构建</td>
      <td>学校，公共的构建</td>
      <td>FALSE</td>
      <td>FALSE</td>
      <td>...</td>
      <td>40</td>
      <td>13</td>
      <td>6</td>
      <td></td>
      <td></td>
      <td>2016</td>
      <td>5/25/2016 15:59</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>

正如我们所看到的结果，数据集定义的第一个版本仍然可以防止所有列犯罪数据，而不考虑更高版本的更新。 例如，如果有机器学习模型当前使用的数据集的第一个版本和使用`Date`的一项功能为数据集中的列，不会遇到任何错误使用最新定义更新，使得只有`ID`， `Arrest`， `Latitude`，`Longitude`犯罪数据中的列。

## <a name="how-to-access-dataset-definitions"></a>如何访问数据集定义

若要获取的所有定义的列表，请使用`get_definitions()`。 若要获取特定版本的定义，请使用`get_definition()`。 下面的示例演示如何检索所有定义的列表，然后检索第 1 版：

```python
# list all definitions for the dataset
dataset.get_definitions()
# get version ID 1
dataset.get_definition(version_id=1)
```

输出`get_definitions()`类似于下面的示例：

```text
{'2': VersionID: 2, State: active, Created: 2019-04-19 16:43:52.439890+00:00, Modified: 2019-04-19 16:43:52.439890+00:00, Notes: select useful column,
 '1': VersionID: 1, State: active, Created: 2019-04-19 16:39:14.112046+00:00, Modified: 2019-04-19 16:44:12.912663+00:00, Notes: None}
```

一旦已检索到一个定义，可以使用机器学习模型或机器学习管道中使用它。

## <a name="manage-lifecycle-of-dataset-definitions"></a>管理数据集定义的生命周期

可以独立管理每个数据集定义的生命周期。 有三个阶段的生命周期： 活动、 不推荐使用，或已存档。

### <a name="active"></a>活动

创建新的数据集定义后，它默认处于活动状态。 

### <a name="deprecate"></a>不推荐使用

不再推荐使用情况，并替代可用时，可以推荐使用数据集定义。 在机器学习管道中使用不推荐使用的数据集定义时，获取返回一条警告消息，但不是会阻止执行。

当弃用的数据集定义，指定数据集 ID 和替换定义的数据集定义版本 ID。 此信息是存储和使用警告消息中，当您尝试使用不推荐使用的数据集定义。

```python
# get the definition that you want to deprecate
ds_def = dataset.get_definition(version_id = 1)

# deprecate it by providing the information for the replacement definition, which is recommended to be used in machine learning scenarios
ds_def.deprecate(deprecate_by_dataset_id=dataset.id, deprecated_by_definition_version=2)
```

### <a name="archive"></a>存档

当不应定义要用于任何原因 （如基础数据不再可用） 时，可以存档数据集定义。 在机器学习管道中使用已存档的数据集定义时，则错误将阻止执行。

```python
# archive the definition with `version_id = 1`
ds_def = dataset.get_definition(version_id = 1)
ds_def.archive()
```

### <a name="reactivate"></a>重新激活

您可以轻松地重新激活任何不推荐使用或已存档的数据集定义。

```python
# reactivate Dataset definition with `version_id =1` which was archived in the previous step
ds_def = dataset.get_definition(version_id = 1)
ds_def.reactivate()
```

## <a name="next-steps"></a>后续步骤

有关如何使用数据集的详细信息，请参阅[创建和注册 Azure 机器学习数据集](how-to-create-register-datasets.md)。

使用数据集的示例，请参阅[示例笔记本](https://aka.ms/dataset-tutorial)。