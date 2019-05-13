---
title: 自动 ML 算法选择和优化
titleSuffix: Azure Machine Learning service
description: 了解 Azure 机器学习服务如何自动选取算法，以及如何通过它来生成模型，以便使用你提供的参数和条件为模型选择最佳算法，从而节省时间。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: nacharya1
ms.author: nilesha
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 136141f5b598fd080edf3254fd01200f2742c763
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235827"
---
# <a name="what-is-automated-machine-learning"></a>什么是自动化机器学习？

自动化的机器学习，也称为 AutoML，让数据科研人员、 分析人员和开发人员，同时维护模型质量构建具有高缩放性、 效率和工作效率的机器学习模型。 

自动化机器学习生成的机器学习模型自动、 智能地选择用于训练的模型的集合，并推荐最适合您。 传统的机器学习模型开发是资源密集型需要大量域知识和时间来生成和比较数十个模型。 与自动化机器学习将加快所需获取生产的 ML 模型很好的轻松和高效的时间。

在幕后，将训练数据是采用一个定义的目标的功能，并以智能方式在循环的机器学习算法和功能选择的组合。 然后，根据训练的分数，最拟合的模型是标识并向你推荐。 

您仍然可以控制您的实验和的透明度了解发生了什么情况。 您可以定义约束和试验根据时间、 准确性或迭代次数，例如目标。 在实验中，每次迭代的培训流和由给定模型的影响最大功能，可以看到每个生成的模型。

## <a name="how-automated-ml-works"></a>如何自动机器学习工作原理

使用**Azure 机器学习服务**，可以设计和使用这些步骤运行自动化的机器学习训练实验：

1. **确定机器学习问题**待解决： 分类，预测或回归
   
1. **指定的源和标记的训练数据的格式**:Numpy 数组或 Pandas 数据帧

1. **配置计算目标，以便为模型定型**，如你[本地计算机、 Azure 机器学习计算、 远程 Vm 或 Azure Databricks](how-to-set-up-training-targets.md)。  了解有关自动化培训[上的远程资源](how-to-auto-train-remote.md)。

1. **配置自动化的机器学习参数**，确定通过不同的模型超参数设置的迭代数高级预处理/特征化，以及要在确定最佳的模型时，查询的指标。  可以配置的设置自动训练实验[在 Azure 门户](how-to-create-portal-experiments.md)或[sdk](how-to-configure-auto-train.md)。

1. **提交训练运行。** 


[![自动化的机器学习](./media/how-to-automated-ml/automated-machine-learning.png)](./media/how-to-automated-ml/automated-machine-learning.png#lightbox)

在培训期间，Azure 机器学习服务会创建多个尝试不同算法和参数的管道。 它将停止后它会命中在试验中定义的退出条件。 

此外可以检查记录运行的信息，其中包含运行期间收集的指标。 训练运行生成 Python 序列化对象 (`.pkl`文件)，其中包含的模型和数据预处理。


即使自动化模型构建，还可以[了解如何重要或相关的功能是](how-to-configure-auto-train.md#explain)到生成的模型。 

> [!VIDEO https://www.youtube.com/embed/l8c-4iDPE0M]

<a name="preprocess"></a>

## <a name="preprocessing"></a>预处理

在每个自动化机器学习实验，你的数据进行预处理使用默认的方法和 （可选） 通过高级预处理。

### <a name="automatic-preprocessing-standard"></a>自动预处理 （标准）
在每个自动化机器学习实验，你的数据进行自动缩放或规范化以帮助算法很好地运行。  在模型定型过程的以下缩放或规范化方法之一将应用于每个模型。

|缩放&nbsp;&&nbsp;规范化| 描述 |
| ------------- | ------------- |
| [StandardScaleWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html)  | 对功能进行标准化，通过删除平均值和缩放到单位方差  |
| [MinMaxScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html)  | 通过缩放该列的最小值和最大值的每个功能来转换功能  |
| [MaxAbsScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MaxAbsScaler.html#sklearn.preprocessing.MaxAbsScaler) |    通过其最大绝对值缩放每个功能 |  
| [RobustScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html) |   此情况下它们的分位范围的 Scaler 功能 |
| [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) | 使用数据的奇异值分解以将它投影到较低维空间线性维数约简 | 
| [TruncatedSVDWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.TruncatedSVD.html) |    此转换器通过截断奇异值分解 (SVD) 执行线性维数约简。 违背 PCA，此估算器不在正中数据之前计算奇异值分解。 这意味着它可以有效地处理使用 scipy.sparse 矩阵 | 
| [SparseNormalizer](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.Normalizer.html) | 每个示例 （即，数据矩阵的每个行） 与至少一个非零值组件调整独立于其他示例，以便其 norm （l1 或 l2） 等于一 | 

### <a name="advanced-preprocessing-optional-featurization"></a>高级预处理： 可选特征化

其他高级预处理和特征化，还提供，例如缺少值插补、 编码和转换。 [了解更多有关哪些特征化信息包含](how-to-create-portal-experiments.md#preprocess)。 启用此设置，但：
+ Azure 门户：选择**预处理**中的复选框**高级设置**[执行这些步骤](how-to-create-portal-experiments.md)。 
+ Python SDK:指定`"preprocess": True`有关[`AutoMLConfig`类](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py)。

## <a name="ensemble-models"></a>系综模型

使用具有自动的机器学习的系综模型训练[Caruana 系综选择算法与已排序的系综初始化](http://www.niculescu-mizil.org/papers/shotgun.icml04.revised.rev2.pdf)。 系综学习的项而不是使用单个模型的多个模型，改进了机器学习结果和预测性能。 系综迭代将显示为您运行的最后一个迭代。

## <a name="use-with-onnx-in-c-apps"></a>与中的 ONNX 配合使用C#应用程序

与 Azure 机器学习，可以使用自动化的机器学习生成 Python 模型并将其转换为 ONNX 格式。 ONNX 运行时支持C#，因此可以使用中自动生成的模型在C#的应用而无需重新编写代码或任何 REST 终结点引入的网络延迟。 请尝试此流的示例[此 Jupyter notebook 中](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-with-onnx/auto-ml-classification-with-onnx.ipynb)。

## <a name="automated-ml-across-microsoft"></a>在 Microsoft 自动化机器学习

自动化机器学习如现在还可供其他 Microsoft 解决方案：
+ 使用 Visual Studio 和 Visual Studio Code 中通过.NET 应用程序中[ML.NET](https://docs.microsoft.com/dotnet/machine-learning/automl-overview)
+ [在 HDInsight 上](../../hdinsight/spark/apache-spark-run-machine-learning-automl.md)，其中扩展中并行中的 HDInsight 群集的 Spark 上你自动机器学习训练作业。 
+ [在 Power BI 中](https://docs.microsoft.com/power-bi/service-machine-learning-automated)

## <a name="next-steps"></a>后续步骤

查看示例并了解如何使用自动化机器学习构建模型：

+ 请按照[教程：使用 Azure 自动机器学习自动训练分类模型](tutorial-auto-train-models.md)

+ 配置自动训练实验的设置： 
   + 在 Azure 门户界面[使用以下步骤](how-to-create-portal-experiments.md)。
   + 使用 Python SDK[使用以下步骤](how-to-configure-auto-train.md)。
  
 + 了解如何自动使用时序数据，训练[使用以下步骤](how-to-auto-train-forecast.md)。

+ 试用[Jupyter 笔记本示例](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/)
