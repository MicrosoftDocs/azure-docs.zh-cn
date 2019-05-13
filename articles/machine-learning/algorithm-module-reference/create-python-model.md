---
title: 创建 Python 模型：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中创建 Python 模型模型创建自定义建模或数据处理模块。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/06/2019
ROBOTS: NOINDEX
ms.openlocfilehash: ea9b50cede3e2d264ca0476b6a987ca6896c3c79
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029740"
---
# <a name="create-python-model"></a>创建 Python 模型

本文介绍如何使用**创建 Python 模型**模块从 Python 脚本创建未经训练的模型。 

可以将该模型基于 Azure 机器学习环境中的 Python 包中包含的任何学习器。 

在创建模型后，可以使用[训练模型](train-model.md)来训练该模型对数据集，这类似于 Azure 机器学习中的任何其他学习器。 将训练的模型可以传递给[评分模型](score-model.md)使用模型来进行预测。 随后可保存已训练的模型，并可以作为 web 服务发布，评分工作流。

> [!WARNING]
> 目前不能将传递到 Python 模型的得分的结果[评估模型](evaluate-model.md)。 如果你需要评估模型，您可以编写自定义 Python 脚本，并运行它使用[执行 Python 脚本](execute-python-script.md)模块。  


## <a name="how-to-configure-create-python-model"></a>如何配置创建 Python 模型

使用此模块需要中间或专业知识的 Python。 该模块支持使用已安装在 Azure 机器学习中的 Python 包中包含的任何学习器。 请参阅中的预安装的 Python 包列表[执行 Python 脚本](execute-python-script.md)。
  

本文将演示如何使用**创建 Python 模型**使用一个简单的实验。 下面是实验图。

![create-python-model](./media/module/aml-create-python-model.png)

1.  单击**创建 Python 模型**、 编辑脚本以实施您建模或处理数据管理。 可以将该模型基于 Azure 机器学习环境中的 Python 包中包含的任何学习器。


    下面是两个类 Naive Bayes 分类器的示例代码使用流行*sklearn*包。

```Python

# The script MUST define a class named AzureMLModel.
# This class MUST at least define the following three methods:
    # __init__: in which self.model must be assigned,
    # train: which trains self.model, the two input arguments must be pandas DataFrame,
    # predict: which generates prediction result, the input argument and the prediction result MUST be pandas DataFrame.
# The signatures (method names and argument names) of all these methods MUST be exactly the same as the following example.


import pandas as pd
from sklearn.naive_bayes import GaussianNB


class AzureMLModel:
    def __init__(self):
        self.model = GaussianNB()
        self.feature_column_names = list()

    def train(self, df_train, df_label):
        self.feature_column_names = df_train.columns.tolist()
        self.model.fit(df_train, df_label)

    def predict(self, df):
        return pd.DataFrame(
            {'Scored Labels': self.model.predict(df[self.feature_column_names]), 
             'probabilities': self.model.predict_proba(df[self.feature_column_names])[:, 1]}
        )


```


2. 连接**创建 Python 模型**到刚创建的模块**训练模型**和**评分模型**

3. 如果你需要评估模型，将添加[执行 Python 脚本](execute-python-script.md)和编辑 Python 脚本以实施评估。

下面是示例评估代码。

```Python


# The script MUST contain a function named azureml_main
# which is the entry point for this module.

# imports up here can be used to 
import pandas as pd

# The entry point function can contain up to two input arguments:
#   Param<dataframe1>: a pandas.DataFrame
#   Param<dataframe2>: a pandas.DataFrame
def azureml_main(dataframe1 = None, dataframe2 = None):
    
    from sklearn.metrics import accuracy_score, precision_score, recall_score, roc_auc_score, roc_curve
    import pandas as pd
    import numpy as np
    
    scores = dataframe1.ix[:, ("income", "Scored Labels", "probabilities")]
    ytrue = np.array([0 if val == '<=50K' else 1 for val in scores["income"]])
    ypred = np.array([0 if val == '<=50K' else 1 for val in scores["Scored Labels"]])    
    probabilities = scores["probabilities"]
    
    accuracy, precision, recall, auc = \
    accuracy_score(ytrue, ypred),\
    precision_score(ytrue, ypred),\
    recall_score(ytrue, ypred),\
    roc_auc_score(ytrue, probabilities)
    
    metrics = pd.DataFrame();
    metrics["Metric"] = ["Accuracy", "Precision", "Recall", "AUC"];
    metrics["Value"] = [accuracy, precision, recall, auc]
    
    return metrics,

```
