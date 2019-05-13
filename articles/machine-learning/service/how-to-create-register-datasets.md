---
title: 创建并注册你的工作区的数据集
titleSuffix: Azure Machine Learning service
description: 了解如何从各种源创建数据集，并注册你的工作区的数据集
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 05/02/19
ms.openlocfilehash: 938f13524b22f34f4becc936885d1611cb854df1
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510492"
---
# <a name="create-and-register-azure-machine-learning-datasets-preview"></a>创建并注册 Azure 机器学习数据集 （预览）

在本文中，你将了解 Azure 机器学习工作流以创建并注册数据集，以及如何跨本地和远程试验以供重复使用访问它们。

Azure 机器学习数据集 （预览版） 使其更轻松地访问和使用你的数据。 数据集管理各种方案，例如模型训练和创建管道中的数据。 使用[Azure 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)，可以使用流行的格式中的数据、 访问基础存储、 探索和准备数据、 管理不同的数据集定义的生命周期和比较中使用的数据集之间培训和在生产环境中。

## <a name="prerequisites"></a>必备组件

若要创建并注册数据集需要：

* Azure 订阅。 如果没有 Azure 订阅，请在开始之前创建一个免费帐户。 立即试用 [Azure 机器学习服务免费版或付费版](https://aka.ms/AMLFree)。

* Azure 机器学习服务工作区。 请参阅[创建 Azure 机器学习服务工作区](https://docs.microsoft.com/azure/machine-learning/service/setup-create-workspace)。

* Azure 机器学习 Python SDK。 若要安装或更新到最新版本的 sdk，请参阅[安装或更新 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py)。

## <a name="create-datasets-from-local-files"></a>从本地文件创建数据集

从本地计算机中加载文件，通过指定的文件或文件夹路径[ `auto_read_files()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#auto-read-files-path--include-path-false-)方法从`Dataset`类。  此方法而无需你指定的文件类型或分析自变量执行以下步骤：

* 推断并设置分隔符。
* 跳过空记录在文件顶部。
* 推断并设置标题行。
* 推断和转换列数据类型。

```Python
from azureml.core.dataset import Dataset

dataset = Dataset.auto_read_files('./data/crime.csv')
```

或者，使用特定于文件的函数来显式控制分析你的文件。 目前，数据集 SDK 支持[分隔](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-delimited-files-path--separator------header--promoteheadersbehavior-allfileshavesameheaders--3---encoding--fileencoding-utf8--0---quoting-false--infer-column-types-true--skip-rows-0--skip-mode--skiplinesbehavior-norows--0---comment-none--include-path-false--archive-options-none-)， [Excel](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-excel-files-path--sheet-name-none--use-column-headers-false--skip-rows-0--include-path-false--infer-column-types-true-)， [Parquet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-parquet-files-path--include-path-false-)，[二进制](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-binary-files-path-)，并[json](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-json-files-path--encoding--fileencoding-utf8--0---flatten-nested-arrays-false--include-path-false-)文件格式。

## <a name="create-datasets-from-azure-datastores"></a>从 Azure 数据存储创建数据集

若要从 Azure 数据存储创建数据集，请确保：

* 验证具有参与者或所有者访问到已注册 Azure 数据存储。

* 导入[ `Workspace` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py)并[ `Datastore` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#definition)和`Dataset`SDK 中的包。

```Python
from azureml.core.workspace import Workspace
from azureml.core.datastore import Datastore
from azureml.core.dataset import Dataset

datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace.from_config()
```

 `get()`方法检索工作区中的现有数据存储。

```
dstore = Datastore.get(workspace, datastore_name)
```

使用`from_delimited_files()`方法来读取带分隔符的文件，并创建未注册的数据集。

```Python
# create an in-memory Dataset on your local machine
datapath = dstore.path('data/src/crime.csv')
dataset = Dataset.from_delimited_files(datapath)

# returns the first 5 rows of the Dataset as a pandas Dataframe.
dataset.head(5)
```

||ID|案例号|date|阻止|IUCR|主要类型|描述|地址说明|逮捕|国内|...|病房|社区范围|FBI 代码|X 坐标|Y 坐标|年龄|更新时间|纬度|经度|Location|
|--|--|---|---|---|---|----|------|-------|------|-----|---|----|----|-----|-----|------|----|-----|----|----|-----
|0|10498554|HZ239907|4/4/2016 23:56|007XX E 111TH ST|1153|欺骗性的做法|通过 300 美元的财务身份盗窃|OTHER|FALSE|FALSE|...|9|50|11|1183356|1831503|2016|5/11/2016 15:48|41.69283384|-87.60431945|(41.692833841, -87.60431945)|
第|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT| 从构建|未成年人|FALSE|FALSE|...|21|71|6|1166776|1850053|2016|5/12/2016 15:48|41.74410697|-87.66449429|(41.744106973, -87.664494285)
2|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|欺骗性的做法|财务身份盗窃 300 美元并在列表视图|未成年人|FALSE|FALSE|...|19|74|11|||2016|5/12/2016 15:50
3|10519591|HZ261534|4/15/2016 9:00|113XX S PRAIRIE AVE|1120|欺骗性的做法|伪造|未成年人|FALSE|FALSE|...|9|49|10|||2016|5/13/2016 15:51
4|10534446|HZ277630|4/15/2016 10:00|055XX KEDZIE 保存 N|890|THEFT|从构建|学校，公共的构建|FALSE|FALSE|...|40|13|6|||2016|5/25/2016 15:59|

## <a name="register-your-datasets-with-workspace"></a>向工作区中注册你的数据集

使用[ `register()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#register-workspace--name--description-none--tags-none--visible-true--exist-ok-false--update-if-exist-false-)方法以将数据集注册到工作区以共享和重用您的组织内以及跨各种试验。

```Python
dataset = dataset.register(workspace = workspace,
                           name = 'dataset_crime',
                           description = 'Training data',
                           exist_ok = False
                           )
```

>[!NOTE]
> 默认参数设置为`register()`是`exist_ok = False`。 如果你尝试注册具有相同名称的数据集，而无需更改此设置会发生错误。

`register()`方法返回与参数的设置，已注册的数据集`exist_ok = True`。

```Python
dataset = dataset.register(workspace = workspace,
                           name = 'dataset_crime',
                           description = 'Training data',
                           exist_ok = True
                           )
```

使用`list()`若要查看所有工作区中已注册的数据集。

```Python
Dataset.list(workspace_name)
```

前面的代码产生以下结果：

```Python
[Dataset(Name: dataset_crime,
         Workspace: workspace_name)]
```

## <a name="access-datasets-in-workspace"></a>工作区中访问数据集

已注册的数据集是可访问，并且可使用本机、 远程以及在 Azure 机器学习计算等计算群集上。 若要在试验中重复使用你已注册的数据集和计算环境，请使用以下代码以按名称获取工作区和已注册的数据集。

```Python
workspace = Workspace.from_config()

dataset = workspace.datasets['dataset_crime']
```

## <a name="next-steps"></a>后续步骤

* [探索和准备数据集](how-to-explore-prepare-data.md)。
* [管理数据集定义的生命周期](how-to-manage-dataset-definitions.md)。
* 使用数据集的示例，请参阅[示例笔记本](https://aka.ms/dataset-tutorial)。
