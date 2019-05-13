---
title: 比较 （&) 快照随时间推移重现数据
titleSuffix: Azure Machine Learning service
description: 了解如何随着时间的推移比较数据，并确保可再现性与数据集的快照
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.date: 05/02/2019
ms.openlocfilehash: 51d0dcfc543834e9a8725d11fa82b566a5132a6b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205000"
---
# <a name="compare-data-and-ensure-reproducibility-with-snapshots-preview"></a>比较数据，并确保可再现性与快照 （预览版）

在本文中，你了解到如何创建和管理快照的你[Azure 机器学习数据集](how-to-create-register-datasets.md)（集），因此您可以捕获或比较一段时间内的数据。 数据集，使其更轻松地访问和使用云中在各种方案中的数据。

**数据集快照**在创建的时将存储数据的配置文件 （摘要统计信息）。 您可以选择还实现可再现性在快照中存储数据的副本。

>[!Important]
> 快照会产生存储成本。 在快照中存储数据的副本需要更多存储空间。 使用[ `dataset.delete_snapshot()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#delete-snapshot-snapshot-name-)它们不再需要时。

## <a name="when-to-use-snapshots"></a>何时使用快照

有三个快照的主要用途：

+ **模型验证**:比较不同的快照之间训练运行或针对生产数据的数据配置文件。

+ **模型的可再现性**:通过调用在定型过程中包括的数据的快照，重新生成你的结果。

+ **随着时间的推移跟踪数据**:请参阅如何通过已从数据集发展[比较配置文件](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_snapshot.datasetsnapshot?view=azure-ml-py#compare-profiles-rhs-dataset-snapshot--include-columns-none--exclude-columns-none--histogram-compare-method--histogramcomparemethod-wasserstein--0--)
  
## <a name="prerequisites"></a>必备组件

若要创建数据集的快照，需要已注册的 Azure 机器学习数据集。 如果您确实有一个，请参阅[创建和注册数据集](how-to-create-register-datasets.md)。

## <a name="create-dataset-snapshots"></a>创建数据集的快照

若要创建数据集的快照，使用[ `dataset.create_snapshot()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?#create-snapshot-snapshot-name--compute-target-none--create-data-snapshot-false--target-datastore-none-)从 Azure 机器学习 SDK。

默认情况下，快照将存储的数据的最新的配置文件 （摘要统计信息）[数据集定义](how-to-manage-dataset-definitions.md)应用。 数据集定义中包含的数据定义任何转换步骤的记录。 它是使在数据准备工作可重现的好办法。

（可选） 还可以包含数据的副本在快照中通过添加`create_data_snapshot = True`。  此数据可用于可再现性。

此示例使用[犯罪数据采样](https://dprepdata.blob.core.windows.net/dataset-sample-files/crime.csv)和名为数据集`dataset_crime`使用本文中，创建["创建并注册数据集"](how-to-create-register-datasets.md)。

```Python
from azureml.core.workspace import Workspace
from azureml.core.dataset import Dataset
from azureml.data.dataset_snapshot import DatasetSnapshot
import datetime

# get existing workspace
workspace = Workspace.from_config()

# get existing, named dataset:
dataset = workspace.datasets['dataset_crime']

# assign name to snapshot
snapshot_name = 'snapshot_' + datetime.datetime.today().strftime('%Y%m%d%H%M%S')

# create snapshot to store profile from a remote environment
# and include copy of data
snapshot = dataset.create_snapshot(snapshot_name = snapshot_name,
                                   compute_target = remote_compute_target,
                                   create_data_snapshot = True)
```

由于以异步方式创建快照，使用[ `wait_for_completion()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_snapshot.datasetsnapshot?view=azure-ml-py#wait-for-completion-show-output-true--status-update-frequency-10-)方法来监视过程。

```Python
# monitor process every 10 seconds
snapshot.wait_for_completion(show_output=True, status_update_frequency=10)

# return snapshot just created
dataset.get_snapshot(snapshot_name)
```

输出：

```
Action status: Running
Action status: Running
Action status: Completed


DatasetSnapshot(Name: snapshot_20190411165420,
Dataset Id: 66cd0830-2f72-41e5-b677-d2d953f54821,
Workspace: "your_workspace_name",
Dataset definition version: 1,
Snapshot created date: 2019-05-11 17:24:00+00:00)
```

使用[ `dataset.delete_snapshot()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#delete-snapshot-snapshot-name-)它们不再需要时。

## <a name="find-snapshots"></a>找到快照

若要检索现有快照，请使用[ `get_snapshot()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#get-snapshot-snapshot-name-)。

若要获取给定数据集的已保存快照的列表，请使用[ `get_all_snapshots()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#get-all-snapshots--)。

```Python
# Get named snapshot for this dataset
dataset.get_snapshot(snapshot_name)

# Get all snapshots for this dataset
dataset.get_all_snapshots()
```

## <a name="get-data-and-profiles-from-snapshots"></a>从快照中获取数据和配置文件

每个快照生成数据集，其中包括摘要统计信息，并为您提供的选项保存一份你的数据配置的文件。

### <a name="get-the-data-profile"></a>获取数据配置文件

使用[ `get_profile()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#get-profile-arguments-none--generate-if-not-exist-true--workspace-none--compute-target-none-)访问数据的配置文件。  此配置文件可用于比较的培训和生产数据，例如。

```Python
snapshot.get_profile()
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
病房|FieldType.INTEGER|第|48|10.0|0.0|10.0|0.0|0.0|0.0|1|5|第|9|22.5|40|48|48|48|24.5|16.2635|264.5|0.173723|-1.51271
社区范围|FieldType.INTEGER|4|77|10.0|0.0|10.0|0.0|0.0|0.0|4|8.5|4|24|37.5|71|77|77|77|41.2|26.6366|709.511|0.112157|-1.73379

### <a name="get-the-data-from-the-snapshot"></a>从快照中获取数据

若要获取的数据集快照中保存的数据副本，生成 pandas 数据帧与[ `to_pandas_dataframe()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#to-pandas-dataframe--)方法。

如果未在快照创建期间请求数据的副本，此方法将失败。

```Python
snapshot.to_pandas_dataframe().head(3)
```

||ID|案例号|date|阻止|IUCR|主要类型|描述|地址说明|逮捕|国内|...|病房|社区范围|FBI 代码|X 坐标|Y 坐标|年龄|更新时间|纬度|经度|Location
-|--|-----------|----|-----|----|------------|-----------|--------------------|------|--------|---|----|--------------|--------|------------|------------|----|----------|--------|---------|--------
0|10498554|HZ239907|2016-04-04 23:56:00|007XX E 111TH ST|1153|欺骗性的做法|通过 300 美元的财务身份盗窃|OTHER|False|False|...|9|50|11|1183356.0|1831503.0|2016|2016-05-11 15:48:00|41.692834|-87.604319|(41.692833841, -87.60431945)
第|10516598|HZ258664|2016-04-15 17:00:00|082XX S MARSHFIELD AVE|890|THEFT|从构建|未成年人|False|False|...|21|71|6|1166776.0|1850053.0|2016|2016-05-12 15:48:00|41.744107|-87.664494|(41.744106973, -87.664494285)
2|10519196|HZ261252|2016-04-15 10:00:00|104XX S SACRAMENTO AVE|1154|欺骗性的做法|财务身份盗窃 300 美元并在列表视图|未成年人|False|False|...|19|74|11|NaN|NaN|2016|2016-05-12 15:50:00|NaN|NaN|

## <a name="next-steps"></a>后续步骤

* 请参阅 SDK[概述](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)有关设计模式和使用情况示例。

* 使用数据集快照的示例，请参阅[示例笔记本](https://aka.ms/dataset-tutorial)。
