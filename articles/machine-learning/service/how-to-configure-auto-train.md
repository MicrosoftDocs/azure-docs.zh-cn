---
title: 创建自动化机器学习试验
titleSuffix: Azure Machine Learning service
description: 自动化机器学习将为你选择一种算法，并生成随时可用于部署的模型。 了解可用于配置自动化机器学习试验的选项。
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 64ba7096f181371a378708e024f46bce17449e98
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510587"
---
# <a name="configure-automated-ml-experiments-in-python"></a>在 Python 中配置自动化的机器学习实验

在本指南中，了解如何定义自动化机器学习试验使用的各种配置设置[Azure 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)。 自动化机器学习将自动选择算法和超参数，并生成随时可用于部署的模型。 可以使用多个选项来配置自动化机器学习试验。

若要查看自动化机器学习试验的示例，请参阅[教程：使用自动化机器学习训练分类模型](tutorial-auto-train-models.md)或[使用云中的自动化机器学习训练模型](how-to-auto-train-remote.md)。

自动化机器学习提供的配置选项：

* 选择试验类型：分类、 回归或时间序列预测
* 数据源、格式和提取数据
* 选择计算目标：本地或远程
* 自动化机器学习试验设置
* 运行自动化机器学习试验
* 探索模型指标
* 注册和部署模型

如果您不喜欢的任何代码体验，还可以[创建自动化的机器学习在 Azure 门户中的试验](how-to-create-portal-experiments.md)。

## <a name="select-your-experiment-type"></a>选择试验类型

在开始试验之前，应确定要解决的机器学习问题类型。 自动化机器学习支持分类、回归和预测任务类型。

在自动化和优化过程中，自动化机器学习支持以下算法。 用户不需要指定算法。 尽管 DNN 算法可在定型期间，自动化机器学习不生成 DNN 模型。

分类 | 回归 | 时间序列预测
|-- |-- |--
[逻辑回归](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression)| [弹性网络](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)| [弹性网络](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)
[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)|[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)|[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)
[渐进提升](https://scikit-learn.org/stable/modules/ensemble.html#classification)|[渐进提升](https://scikit-learn.org/stable/modules/ensemble.html#regression)|[渐进提升](https://scikit-learn.org/stable/modules/ensemble.html#regression)
[决策树](https://scikit-learn.org/stable/modules/tree.html#decision-trees)|[决策树](https://scikit-learn.org/stable/modules/tree.html#regression)|[决策树](https://scikit-learn.org/stable/modules/tree.html#regression)
[K 近邻](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)|[K 近邻](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)|[K 近邻](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)
[线性 SVC](https://scikit-learn.org/stable/modules/svm.html#classification)|[LARS Lasso](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)|[LARS Lasso](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)
[C 支持向量分类 (SVC)](https://scikit-learn.org/stable/modules/svm.html#classification)|[随机梯度下降 (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)|[随机梯度下降 (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)
[随机林](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)|[随机林](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)|[随机林](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)
[极端随机树](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)|[极端随机树](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)|[极端随机树](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)
[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)|[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)| [Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)
[DNN 分类器](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNClassifier)|[DNN 回归量](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNRegressor) | [DNN 回归量](https://www.tensorflow.org/api_docs/python/tf/estimator/DNNRegressor)|
[DNN 线性分类器](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearClassifier)|[线性回归量](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearRegressor)|[线性回归量](https://www.tensorflow.org/api_docs/python/tf/estimator/LinearRegressor)
[朴素贝叶斯](https://scikit-learn.org/stable/modules/naive_bayes.html#bernoulli-naive-bayes)|
[随机梯度下降 (SGD)](https://scikit-learn.org/stable/modules/sgd.html#sgd)|


## <a name="data-source-and-format"></a>数据源和格式
自动化机器学习支持驻留在本地桌面上或云中（例如 Azure Blob 存储）的数据。 可将数据读取成 scikit-learn 支持的数据格式。 可将数据读取成：
* Numpy 数组 X（特征）和 y（目标变量，也称为标签）
* Pandas 数据帧

示例：

*   Numpy 数组

    ```python
    digits = datasets.load_digits()
    X_digits = digits.data
    y_digits = digits.target
    ```

*   Pandas 数据帧

    ```python
    import pandas as pd
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"')
    # get integer labels
    df = df.drop(["Label"], axis=1)
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42)
    ```

## <a name="fetch-data-for-running-experiment-on-remote-compute"></a>在远程计算中提取用于运行试验的数据

如果使用远程计算来运行试验，必须将数据提取包装在单独的 Python 脚本 `get_data()` 中。 此脚本在运行自动化机器学习试验的远程计算中运行。 `get_data` 消除了通过网络为每个迭代提取数据的需要。 如果不使用 `get_data`，则在远程计算中运行时，试验将会失败。

下面是 `get_data` 的示例：

```python
%%writefile $project_folder/get_data.py
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
def get_data(): # Burning man 2016 data
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"')
    # get integer labels
    le = LabelEncoder()
    le.fit(df["Label"].values)
    y = le.transform(df["Label"].values)
    df = df.drop(["Label"], axis=1)
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42)
    return { "X" : df, "y" : y }
```

在 `AutoMLConfig` 对象中，指定 `data_script` 参数并提供 `get_data` 脚本文件的路径，如下所示：

```python
automl_config = AutoMLConfig(****, data_script=project_folder + "/get_data.py", **** )
```

`get_data` 脚本可以返回：

密钥 | Type | 与以下对象互斥    | 描述
---|---|---|---
X | Pandas 数据帧或 Numpy 数组 | data_train、label、columns |  用于训练的所有特征
y | Pandas 数据帧或 Numpy 数组 |   label   | 用于训练的标签数据。 对于分类，应是一个整数数组。
X_valid | Pandas 数据帧或 Numpy 数组   | data_train、label | （可选）用于验证的所有特征。 如果未指定，则在 train 与 validate 之间拆分 X
y_valid |   Pandas 数据帧或 Numpy 数组 | data_train、label | （可选）用于验证的标签数据。 如果未指定，则在 train 与 validate 之间拆分 y
sample_weight | Pandas 数据帧或 Numpy 数组 |   data_train、label、columns| （可选）每个样本的权重值。 需要为数据点分配不同的权重时使用
sample_weight_valid | Pandas 数据帧或 Numpy 数组 | data_train、label、columns |    （可选）每个验证样本的权重值。 如果未指定，则在 train 与 validate 之间拆分 sample_weight
data_train |    Pandas 数据帧 |  X、y、X_valid、y_valid |    用于训练的所有数据（特征+标签）
label | string  | X、y、X_valid、y_valid |  data_train 中的哪个列表示标签
列 | 字符串数组  ||  （可选）用于特征的列的白名单
cv_splits_indices   | 整数数组 ||  （可选）用于拆分数据以进行交叉验证的索引列表

### <a name="load-and-prepare-data-using-data-prep-sdk"></a>加载和准备数据使用数据准备 SDK
自动化的机器学习试验支持数据加载和转换使用数据准备 SDK。 使用 SDK 可以

>* 使用分析参数推理（编码、分隔符和标头）从多种文件类型进行加载
>* 在文件加载时使用推理进行类型转换
>* 支持 MS SQL Server 和 Azure Data Lake Storage 连接
>* 使用表达式添加列
>* 估算缺失值
>* 按示例派生列
>* 筛选
>* 自定义 Python 转换

若要了解 DataPrep SDK，请参阅[如何准备要建模的数据](how-to-load-data.md)一文。
下面是使用 DataPrep SDK 加载数据的示例。
```python
# The data referenced here was pulled from `sklearn.datasets.load_digits()`.
simple_example_data_root = 'https://dprepdata.blob.core.windows.net/automl-notebook-data/'
X = dprep.auto_read_file(simple_example_data_root + 'X.csv').skip(1)  # Remove the header row.
# You can use `auto_read_file` which intelligently figures out delimiters and datatypes of a file.

# Here we read a comma delimited file and convert all columns to integers.
y = dprep.read_csv(simple_example_data_root + 'y.csv').to_long(dprep.ColumnSelector(term='.*', use_regex = True))
```

## <a name="train-and-validation-data"></a>训练和验证数据

可以通过 get_data() 或直接在 `AutoMLConfig` 方法中指定单独的训练集和验证集。

## <a name="cross-validation-split-options"></a>交叉验证拆分选项

### <a name="k-folds-cross-validation"></a>K 折交叉验证

使用 `n_cross_validations` 设置指定交叉验证的数目。 训练数据集将随机拆分为大小相等的 `n_cross_validations` 折。 在每个交叉验证轮次，某个折将用于验证剩余折上训练的模型。 重复此过程 `n_cross_validations` 次，直到每个折作为验证集使用了一次。 将报告在所有 `n_cross_validations` 轮次中获得的平均评分，并基于整个训练数据集重新训练相应的模型。 

### <a name="monte-carlo-cross-validation-repeated-random-sub-sampling"></a>Monte Carlo 交叉验证 （重复随机子采样）

使用 `validation_size` 指定应该用于验证的训练数据集百分比，并使用 `n_cross_validations` 指定交叉验证的数目。 在每个交叉验证轮次，将随机选择 `validation_size` 大小的子集来验证基于剩余数据训练的模型。 最后，将报告在所有 `n_cross_validations` 轮次中获得的平均评分，并基于整个训练数据集重新训练相应的模型。 Monte Carlo 不支持时序预测。

### <a name="custom-validation-dataset"></a>自定义验证数据集

使用自定义验证数据集不接受随机拆分时，通常时序数据或不平衡的数据。 可以指定自己的验证数据集。 将会根据指定的验证数据集而不是随机数据集来评估模型。

## <a name="compute-to-run-experiment"></a>用于运行试验的计算环境

接下来，确定要在何处训练模型。 自动化机器学习训练试验可在以下计算选项中运行：
*   本地台式机或便携式计算机等本地计算机  – 如果数据集较小，并且仍处于探索阶段，则通常使用此选项。
*   云中的远程计算机 – [Azure 机器学习托管计算](concept-azure-machine-learning-architecture.md#managed-and-unmanaged-compute-targets)是一个托管服务，可用于在 Azure 虚拟机群集上训练机器学习模型。

有关包含本地和远程计算目标的示例 Notebook，请参阅 [GitHub 站点](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning)。

*   你的 Azure 订阅中的 Azure Databricks 群集。 您可以找到有关详细信息-[自动机器学习的安装程序 Azure Databricks 群集](how-to-configure-environment.md#azure-databricks)

请参阅[GitHub 站点](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/azure-databricks/automl)例如使用 Azure Databricks notebook。

<a name='configure-experiment'></a>

## <a name="configure-your-experiment-settings"></a>配置试验设置

可以使用多个选项来配置自动化机器学习试验。 通过实例化 `AutoMLConfig` 对象来设置这些参数。 有关参数的完整列表，请参阅 [AutoMLConfig 类](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py)。  

示例包括：

1.  分类试验使用加权 AUC 作为主要指标，每次迭代的最大时间为 12,000 秒，试验在完成 50 次迭代和 2 个交叉验证折后结束。

    ```python
    automl_classifier = AutoMLConfig(
        task='classification',
        primary_metric='AUC_weighted',
        max_time_sec=12000,
        iterations=50,
        blacklist_models='XGBoostClassifier',
        X=X,
        y=y,
        n_cross_validations=2)
    ```
2.  下面是设置为在 100 次迭代后结束的回归试验示例，每次迭代最长持续 600 秒，并完成 5 个交叉验证折。

    ```python
    automl_regressor = AutoMLConfig(
        task='regression',
        max_time_sec=600,
        iterations=100,
        whitelist_models='kNN regressor'
        primary_metric='r2_score',
        X=X,
        y=y,
        n_cross_validations=5)
    ```

三种不同`task`参数值确定算法要应用的列表。  使用 `whitelist` 或 `blacklist` 参数可进一步修改迭代，从而包含或排除可用算法。 可以在中找到支持的型号的列表[SupportedAlgorithms 类](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.constants.supportedalgorithms?view=azure-ml-py)。

## <a name="primary-metric"></a>主要指标
主要指标;如中所示上面的示例将确定该度量值以用于在模型定型过程进行优化。 可以选择的主要指标取决于您选择任务类型。 下面是可用的度量值的列表。

|分类 | 回归 | 时间序列预测
|-- |-- |--
|accuracy| spearman_correlation | spearman_correlation
|AUC_weighted | normalized_root_mean_squared_error | normalized_root_mean_squared_error
|average_precision_score_weighted | r2_score | r2_score
|norm_macro_recall | normalized_mean_absolute_error | normalized_mean_absolute_error
|precision_score_weighted |

## <a name="data-preprocessing--featurization"></a>数据预处理和特征化

在每个自动化机器学习实验，你的数据是[自动缩放和规范化](concept-automated-ml.md#preprocess)帮助算法很好地运行。  但是，您还可以启用其他预处理/特征化，例如缺少值插补、 编码和转换。 [了解更多有关哪些特征化信息包含](how-to-create-portal-experiments.md#preprocess)。 

若要启用此特征化，请指定`"preprocess": True`有关[`AutoMLConfig`类](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py)。

## <a name="time-series-forecasting"></a>时间序列预测
对于时间序列预测任务类型必须要定义的其他参数。
1. time_column_name-这是列的一个必需的参数，用于训练数据包含日期/时间序列中定义的名称。 
1. max_horizon-这将定义您希望预测出基于定型数据的周期的时间长度。 例如如果必须使用每日的时间粒度的定型数据，则定义延伸的范围缩小天内你想要用于定型模型。
1. grain_column_names-这将定义包含定型数据中的各个时序数据的列的名称。 例如，如果预测的特定品牌的应用商店的销售额，您将应用商店和品牌的列定义为粒度列。

这些示例，请参阅以下正在使用的设置，提供了 notebook 示例[此处](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-orange-juice-sales/auto-ml-forecasting-orange-juice-sales.ipynb)。

```python
# Setting Store and Brand as grains for training.
grain_column_names = ['Store', 'Brand']
nseries = data.groupby(grain_column_names).ngroups

# View the number of time series data with defined grains
print('Data contains {0} individual time-series.'.format(nseries))
```

```python
time_series_settings = {
    'time_column_name': time_column_name,
    'grain_column_names': grain_column_names,
    'drop_column_names': ['logQuantity'],
    'max_horizon': n_test_periods
}

automl_config = AutoMLConfig(task='forecasting',
                             debug_log='automl_oj_sales_errors.log',
                             primary_metric='normalized_root_mean_squared_error',
                             iterations=10,
                             X=X_train,
                             y=y_train,
                             n_cross_validations=5,
                             path=project_folder,
                             verbosity=logging.INFO,
                             **time_series_settings)
```

## <a name="run-experiment"></a>运行试验

提交试验以运行和生成模型。 将 `AutoMLConfig` 传递给 `submit` 方法以生成模型。

```python
run = experiment.submit(automl_config, show_output=True)
```

>[!NOTE]
>首先在新的计算机上安装依赖项。  最长可能需要在 10 分钟后才会显示输出。
>将 `show_output` 设置为 `True` 可在控制台上显示输出。

## <a name="exit-criteria"></a>退出条件 
那里几个选项可以定义要完成你的试验。
1. 没有条件-如果未定义任何退出实验将继续，直到执行任何进一步的操作不对主要指标的参数。 
1. 数量的迭代-定义要运行试验的迭代数。 你可以可选添加 iteration_timeout_minutes 来定义在每次迭代每分钟的时间限制。
1. 在一段时间-你可以定义多长时间以分钟为单位试验时，应该在运行设置中使用 experiment_timeout_minutes 后退出。
1. 已达到一个分数-使用的 experiment_exit_score 您可以选择在达到基于主要指标的分数后完成试验后退出。

## <a name="explore-model-metrics"></a>探索模型指标
如果在笔记本中操作，可以在小组件或内联单元中查看结果。 有关更多详细信息，请参阅[跟踪和评估模型](how-to-track-experiments.md#view-run-details)。


### <a name="classification-metrics"></a>分类指标
在分类任务的每次迭代中保存以下指标。

|指标|描述|计算|其他参数
--|--|--|--|
AUC_macro| AUC 是接收方操作特性曲线下面的区域。 Macro 是每个类的 AUC 算术平均值。  | [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="macro"|
AUC_Micro| AUC 是接收方操作特性曲线下面的区域。 通过组合每个类中的真报率和误报率来全局计算 Micro| [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="micro"|
AUC_Weighted  | AUC 是接收方操作特性曲线下面的区域。 Weighted 是每个类的评分算术平均值，按每个类中的真实实例数加权| [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html)|average="weighted"
accuracy|Accuracy 是与真实标签完全匹配的预测标签百分比。 |[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html) |无|
average_precision_score_macro|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 Macro 是每个类的平均精度评分算术平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="macro"|
average_precision_score_micro|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 通过组合每个交接中的真报率和误报率来全局计算 Micro|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="micro"|
average_precision_score_weighted|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 Weighted 是每个类的平均精度评分算术平均值，按每个类中的真实实例数加权|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="weighted"|
balanced_accuracy|平衡准确度是每个类的召回率算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
f1_score_macro|F1 评分是精度和召回率的调和平均值。 Macro 是每个类的 F1 评分算术平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="macro"|
f1_score_micro|F1 评分是精度和召回率的调和平均值。 通过统计真报率、漏报率和误报率总值来全局计算 Micro|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="micro"|
f1_score_weighted|F1 评分是精度和召回率的调和平均值。 按每个类的 F1 评分类频率计算的加权平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="weighted"|
log_loss|这是（多项式） 逻辑回归及其扩展（例如神经网络）中使用的损失函数，在给定概率分类器的预测的情况下，定义为真实标签的负对数可能性。 对于在 {0,1} 中包含真实标签 yt，且包含 yt=1 的估计概率 yp 的单个样本，对数损失为 -log P(yt&#124;yp) = -(yt log(yp) + (1 - yt) log(1 - yp))|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html)|无|
norm_macro_recall|规范化宏召回率是已规范化的宏召回率，因此，随机性能的评分为 0，完美性能的评分为 1。 可以通过公式 norm_macro_recall := (recall_score_macro - R)/(1 - R) 来计算此值，其中，R 是随机预测的 recall_score_macro 预期值（例如，对于二元分类，R=0.5；对于 C 类分类问题，R=(1/C)）|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average = "macro"，(recall_score_macro - R)/(1 - R)，其中，R 是随机预测的 recall_score_macro 预期值（例如，对于二元分类，R=0.5；对于 C 类分类问题，R=(1/C)）|
precision_score_macro|Precision 是标记为特定类的，实际位于该类中的元素百分比。 Macro 是每个类的精度算术平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="macro"|
precision_score_micro|Precision 是标记为特定类的，实际位于该类中的元素百分比。 通过统计真报率和误报率总值来全局计算 Micro|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="micro"|
precision_score_weighted|Precision 是标记为特定类的，实际位于该类中的元素百分比。 Weighted 是每个类的精度算术平均值，按每个类中的真实实例数加权|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="weighted"|
recall_score_macro|Recall 是实际位于某个类中的已正确标记的元素百分比。 Macro 是每个类的召回率算术平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
recall_score_micro|Recall 是实际位于某个类中的已正确标记的元素百分比。 通过统计真报率和漏报率总值来全局计算 Micro|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="micro"|
recall_score_weighted|Recall 是实际位于某个类中的已正确标记的元素百分比。 Weighted 是每个类的召回率算术平均值，按每个类中的真实实例数加权|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="weighted"|
weighted_accuracy|加权准确度是当分配给每个示例的权重等于该示例的真实类中的真实实例比例时的准确度|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html)|sample_weight 是等于目标中每个元素的该类比例的向量|

### <a name="regression-and-time-series-forecasting-metrics"></a>回归和时间序列预测指标
在回归或预测任务的每次迭代中保存以下指标。

|指标|描述|计算|其他参数
--|--|--|--|
explained_variance|解释方差是数学模型计算给定数据集的方差时遵循的比例。 它是原始数据方差与误差方差之间的递减百分比。 如果误差平均值为 0，则它等于解释方差。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.explained_variance_score.html)|无|
r2_score|R2 是与输出平均值的基线模型相比，平方误差的确定系数或递减百分比。 如果误差平均值为 0，则它等于解释方差。|[计算](https://scikit-learn.org/0.16/modules/generated/sklearn.metrics.r2_score.html)|无|
spearman_correlation|斯皮尔曼相关是两个数据集之间的关系单一性的非参数测量法。 与皮尔逊相关不同，斯皮尔曼相关不假设两个数据集呈正态分布。 与其他相关系数一样，此参数在 -1 和 +1 之间变化，0 表示不相关。 -1 或 +1 相关表示确切的单一关系。 正相关表示 y 随着 x 的递增而递增。 负相关表示 y 随着 x 的递增而递减。|[计算](https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.stats.spearmanr.html)|无|
mean_absolute_error|平均绝对误差是目标与预测之间的差的预期绝对值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|无|
normalized_mean_absolute_error|规范化平均绝对误差是平均绝对误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|除以数据范围|
median_absolute_error|平均绝对误差是目标与预测之间的所有绝对差的中间值。 此损失值可靠地反映离群值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|无|
normalized_median_absolute_error|规范化中间绝对误差是中间绝对误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|除以数据范围|
root_mean_squared_error|均方根误差是目标与预测之间的预期平方差的平方根|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|无|
normalized_root_mean_squared_error|规范化均方根误差是均方根误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|除以数据范围|
root_mean_squared_log_error|均方根对数误差是预期平方对数误差的平方根|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|无|
normalized_root_mean_squared_log_error|规范化均方根对数误差指均方根对数误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|除以数据范围|


## <a name="understand-automated-ml-models"></a>了解自动化机器学习模型

使用自动化的机器学习生成的任何模型包括以下步骤：
+ 自动执行特征工程 (如果预处理 = True)
+ 缩放/规范化和 hypermeter 值的算法

我们使透明来自动化机器学习的 fitted_model 输出中获取此信息。

```python
automl_config = AutoMLConfig(…)
automl_run = experiment.submit(automl_config …)
best_run, fitted_model = automl_run.get_output()
```

### <a name="automated-feature-engineering"></a>自动化的功能工程

请参阅的预处理列表和[自动执行特征工程](concept-automated-ml.md#preprocess)发生这种情况时预处理 = True。  

请看以下示例：
+ 有 4 个输入的功能：A （数值） （数值） B、 C （数值）、 D (DateTime)
+ 数字特征 C 被删除，因为它是包含所有唯一值的 ID 列
+ 数字功能 A 和 B 具有缺失值，因此处理由平均值
+ DateTime 功能 D 是到 11 不同工程特征的特征化

使用这两个 Api 上条拟合模型的第一步，若要了解详细信息。  请参阅[此示例笔记本](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand)。

+ API 1:`get_engineered_feature_names()`返回，工程的特征名称的列表。

  用法: 
  ```python
  fitted_model.named_steps['timeseriestransformer']. get_engineered_feature_names ()
  ```

  ```
  Output: ['A', 'B', 'A_WASNULL', 'B_WASNULL', 'year', 'half', 'quarter', 'month', 'day', 'hour', 'am_pm', 'hour12', 'wday', 'qday', 'week']
  ```

  此列表包括所有，工程的特征名称。

  >[!Note]
  >任务使用 timeseriestransformer = 预测，使用回归或分类任务 datatransformer。 

+ API 2:`get_featurization_summary()`返回特征化所有输入功能的摘要。

  用法: 
  ```python
  fitted_model.named_steps['timeseriestransformer'].get_featurization_summary()
  ```

  >[!Note]
  >任务使用 timeseriestransformer = 预测，使用回归或分类任务 datatransformer。

  输出：
  ```
  [{'RawFeatureName': 'A',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'B',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'C',
    'TypeDetected': 'Numeric',
    'Dropped': 'Yes',
    'EngineeredFeatureCount': 0,
    'Tranformations': []},
   {'RawFeatureName': 'D',
    'TypeDetected': 'DateTime',
    'Dropped': 'No',
    'EngineeredFeatureCount': 11,
    'Tranformations': ['DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime']}]
  ```
  
   其中：
   
   |输出|定义|
   |----|--------|
   |RawFeatureName|从提供的数据集的输入的功能/列名称。| 
   |TypeDetected|检测到数据类型的输入功能。|
   |删除|指示是否输入的功能不删除，或者使用。|
   |EngineeringFeatureCount|通过自动化的功能工程转换生成的特征数。|
   |转换|转换应用到输入功能，可生成工程的特征的列表。|  

### <a name="scalingnormalization-and-algorithm-with-hypermeter-values"></a>缩放/规范化和 hypermeter 值的算法：

若要了解管道的缩放/规范化和算法/超参数值，请使用 fitted_model.steps。 [了解有关缩放/规范化](concept-automated-ml.md#preprocess)。 下面是示例输出：

```
[('RobustScaler', RobustScaler(copy=True, quantile_range=[10, 90], with_centering=True, with_scaling=True)), ('LogisticRegression', LogisticRegression(C=0.18420699693267145, class_weight='balanced', dual=False, fit_intercept=True, intercept_scaling=1, max_iter=100, multi_class='multinomial', n_jobs=1, penalty='l2', random_state=None, solver='newton-cg', tol=0.0001, verbose=0, warm_start=False))
```

若要获取更多详细信息，请使用此帮助器函数中所示[此示例笔记本](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification/auto-ml-classification.ipynb)。

```python
from pprint import pprint
def print_model(model, prefix=""):
    for step in model.steps:
        print(prefix + step[0])
        if hasattr(step[1], 'estimators') and hasattr(step[1], 'weights'):
            pprint({'estimators': list(e[0] for e in step[1].estimators), 'weights': step[1].weights})
            print()
            for estimator in step[1].estimators:
                print_model(estimator[1], estimator[0]+ ' - ')
        else:
            pprint(step[1].get_params())
            print()
                
print_model(fitted_model)
```

下面是示例输出：

+ 使用特定算法 (与在此情况下 RobustScalar LogisticRegression) 的管道：

  ```
  RobustScaler
  {'copy': True,
   'quantile_range': [10, 90],
   'with_centering': True,
   'with_scaling': True}
  
  LogisticRegression
  {'C': 0.18420699693267145,
   'class_weight': 'balanced',
   'dual': False,
   'fit_intercept': True,
   'intercept_scaling': 1,
   'max_iter': 100,
   'multi_class': 'multinomial',
   'n_jobs': 1,
   'penalty': 'l2',
   'random_state': None,
   'solver': 'newton-cg',
   'tol': 0.0001,
   'verbose': 0,
   'warm_start': False}
  ```
  
+ 使用集合方法中的管道：在这种情况下，它是 2 个不同的管道的系综

  ```
  prefittedsoftvotingclassifier
  {'estimators': ['1', '18'],
  'weights': [0.6666666666666667,
              0.3333333333333333]}

  1 - RobustScaler
  {'copy': True,
   'quantile_range': [25, 75],
   'with_centering': True,
   'with_scaling': False}
  
  1 - LightGBMClassifier
  {'boosting_type': 'gbdt',
   'class_weight': None,
   'colsample_bytree': 0.2977777777777778,
   'importance_type': 'split',
   'learning_rate': 0.1,
   'max_bin': 30,
   'max_depth': 5,
   'min_child_samples': 6,
   'min_child_weight': 5,
   'min_split_gain': 0.05263157894736842,
   'n_estimators': 200,
   'n_jobs': 1,
   'num_leaves': 176,
   'objective': None,
   'random_state': None,
   'reg_alpha': 0.2631578947368421,
   'reg_lambda': 0,
   'silent': True,
   'subsample': 0.8415789473684211,
   'subsample_for_bin': 200000,
   'subsample_freq': 0,
   'verbose': -10}
  
  18 - StandardScalerWrapper
  {'class_name': 'StandardScaler',
   'copy': True,
   'module_name': 'sklearn.preprocessing.data',
   'with_mean': True,
   'with_std': True}
  
  18 - LightGBMClassifier
  {'boosting_type': 'goss',
   'class_weight': None,
   'colsample_bytree': 0.2977777777777778,
   'importance_type': 'split',
   'learning_rate': 0.07894947368421053,
   'max_bin': 30,
   'max_depth': 6,
   'min_child_samples': 47,
   'min_child_weight': 0,
   'min_split_gain': 0.2631578947368421,
   'n_estimators': 400,
   'n_jobs': 1,
   'num_leaves': 14,
   'objective': None,
   'random_state': None,
   'reg_alpha': 0.5789473684210527,
   'reg_lambda': 0.7894736842105263,
   'silent': True,
   'subsample': 1,
   'subsample_for_bin': 200000,
   'subsample_freq': 0,
   'verbose': -10}
  ```
  
<a name="explain"></a>

## <a name="explain-the-model-interpretability"></a>解释模型 (interpretability)

使用自动化机器学习可以了解特征重要性。  在训练过程中，可以获取模型的全局特征重要性。  对于分类方案，还可以获取类级特征重要性。  必须提供验证数据集 (X_valid) 才能获取特征重要性。

可通过两种方式生成特征重要性。

*   完成试验后，可以针对任何迭代使用 `explain_model` 方法。

    ```python
    from azureml.train.automl.automlexplainer import explain_model

    shap_values, expected_values, overall_summary, overall_imp, per_class_summary, per_class_imp = \
        explain_model(fitted_model, X_train, X_test)

    #Overall feature importance
    print(overall_imp)
    print(overall_summary)

    #Class-level feature importance
    print(per_class_imp)
    print(per_class_summary)
    ```

*   若要查看所有迭代的特征重要性，请在 AutoMLConfig 中将 `model_explainability` 标志设置为 `True`。

    ```python
    automl_config = AutoMLConfig(task = 'classification',
                                 debug_log = 'automl_errors.log',
                                 primary_metric = 'AUC_weighted',
                                 max_time_sec = 12000,
                                 iterations = 10,
                                 verbosity = logging.INFO,
                                 X = X_train,
                                 y = y_train,
                                 X_valid = X_test,
                                 y_valid = y_test,
                                 model_explainability=True,
                                 path=project_folder)
    ```

    完成后，可以使用 retrieve_model_explanation 方法检索特定迭代的特征重要性。

    ```python
    from azureml.train.automl.automlexplainer import retrieve_model_explanation

    shap_values, expected_values, overall_summary, overall_imp, per_class_summary, per_class_imp = \
        retrieve_model_explanation(best_run)

    #Overall feature importance
    print(overall_imp)
    print(overall_summary)

    #Class-level feature importance
    print(per_class_imp)
    print(per_class_summary)
    ```

可以在 Azure 门户中的工作区内可视化特征重要性图表。 在 Notebook 中使用 Jupyter 小组件时，也会显示该图表。 若要详细了解图表，请参阅[示例 Azure 机器学习服务 Notebook](samples-notebooks.md) 一文。

```Python
from azureml.widgets import RunDetails
RunDetails(local_run).show()
```
![特征重要性图形](./media/how-to-configure-auto-train/feature-importance.png)

## <a name="next-steps"></a>后续步骤

详细了解[如何以及在何处部署模型](how-to-deploy-and-where.md)。

详细了解如何[如何训练回归模型使用自动机器学习](tutorial-auto-train-models.md)或[如何训练使用自动远程资源的机器学习](how-to-auto-train-remote.md)。
