---
title: 使用 TensorFlow 和 Keras 的训练模型
titleSuffix: Azure Machine Learning service
description: 了解如何使用 TensorFlow 和 Keras 房产运行单节点和分布式的 TensorFlow 和 Keras 模型定型
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: minxia
author: mx-iao
ms.reviewer: sgilley
ms.date: 05/06/2019
ms.custom: seodec18
ms.openlocfilehash: 0d5751ab96dc6b44229e2b18b832a570930058ca
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442347"
---
# <a name="train-tensorflow-and-keras-models-with-azure-machine-learning-service"></a>使用 Azure 机器学习服务的训练 TensorFlow 和 Keras 模型

可以轻松地在 Azure 计算上运行 TensorFlow 培训作业通过使用[ `TensorFlow` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py) Azure 机器学习 SDK 中的估计器类。 `TensorFlow`估算器指示 Azure 机器学习服务进行深度神经网络 (DNN) 训练 TensorFlow 启用容器上运行作业。

`TensorFlow`估算器还提供了一个抽象层，对执行，这意味着可以轻松地配置不同的计算目标上的参数化的运行而无需更改训练脚本。

## <a name="get-started"></a>开始使用

由于`TensorFlow`估算器类是类似于基[ `Estimator` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py)，我们建议你第一次读取[估算器操作指南文章](how-to-train-ml-models.md)若要了解总体概念。

若要开始使用 Azure 机器学习服务[完成快速入门](quickstart-run-cloud-notebook.md)。 完成后，您必须[Azure 机器学习工作区](concept-azure-machine-learning-architecture.md#workspace)及其所有我们[示例笔记本](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml)其中包含用于定型 Dnn 使用 TensorFlow 和 Keras。

## <a name="single-node-training"></a>单节点训练

若要运行 TensorFlow 作业，请实例化[ `TensorFlow` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py)对象，并将其作为试验提交。

下面的代码实例化 TensorFlow 估算器，并将其提交试验一下。 训练脚本`train.py`将使用给定的脚本参数运行。 作业将运行于启用 GPU[计算目标](how-to-set-up-training-targets.md)，和 scikit-了解将作为的依赖项安装`train.py`。

```Python
from azureml.train.dnn import TensorFlow

# training script parameters passed as command-line arguments
script_params = {
    '--batch-size': 50,
    '--learning-rate': 0.01,
}

# TensorFlow constructor
tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params=script_params,
                    compute_target=compute_target,
                    entry_script='train.py', # relative path to your TensorFlow job
                    conda_packages=['scikit-learn'],
                    use_gpu=True)

# submit the TensorFlow job
run = exp.submit(tf_est)
```

## <a name="distributed-training"></a>分布式训练

[ `TensorFlow` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py)估算器还在 CPU 和 GPU 群集之间支持分布式的培训。 你可以轻松运行分布式的 TensorFlow 作业和 Azure 机器学习服务将管理的基础结构和为你的业务流程。

Azure 机器学习服务支持 TensorFlow 的分布式定型的两个方法：

* [基于 MPI](https://www.open-mpi.org/)分布式定型使用[Horovod](https://github.com/uber/horovod) framework
* 本机[分布式 TensorFlow](https://www.tensorflow.org/deploy/distributed)使用参数服务器方法

### <a name="horovod"></a>Horovod

[Horovod](https://github.com/uber/horovod)是开放源代码框架，用于分布式的培训开发 Uber 的。 它提供了分布式 GPU TensorFlow 作业的简便途径。

下面的示例运行两个辅助角色分布在两个节点中使用 Horovod 分布式的培训作业。

```Python
from azureml.train.dnn import TensorFlow

# Tensorflow constructor
tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params={},
                    compute_target=compute_target,
                    entry_script='train.py', # relative path to your TensorFlow job
                    node_count=2,
                    process_count_per_node=1,
                    distributed_backend='mpi', # specifies Horovod backend
                    use_gpu=True)

# submit the TensorFlow job
run = exp.submit(tf_est)
```

因此它导入训练脚本，将为您，安装 Horovod 和其依赖项。

```Python
import tensorflow as tf
import horovod
```

### <a name="parameter-server"></a>服务器参数

此外，你还可以运行[本机分布式 TensorFlow](https://www.tensorflow.org/deploy/distributed)，它使用参数服务器模型。 在此方法中，你将在一组参数服务器和工作线程中进行训练。 工作线程在训练期间计算梯度，而参数服务器聚合梯度。

下面的示例运行四个辅助角色分布在两个节点中使用参数服务器方法分布式的培训作业。

```Python
from azureml.train.dnn import TensorFlow

# Tensorflow constructor
tf_est = TensorFlow(source_directory='./my-tf-proj',
                    script_params={},
                    compute_target=compute_target,
                    entry_script='train.py', # relative path to your TensorFlow job
                    node_count=2,
                    worker_count=2,
                    parameter_server_count=1,
                    distributed_backend='ps', # specifies parameter server backend
                    use_gpu=True)

# submit the TensorFlow job
run = exp.submit(tf_est)
```

#### <a name="note-on-tfconfig"></a>请注意 `TF_CONFIG`

您还需要的网络地址和端口的群集[ `tf.train.ClusterSpec` ](https://www.tensorflow.org/api_docs/python/tf/train/ClusterSpec)，因此，Azure 机器学习设置`TF_CONFIG`为你的环境变量。

`TF_CONFIG` 环境变量是一个 JSON 字符串。 下面是介绍参数服务器变量的示例：

```
TF_CONFIG='{
    "cluster": {
        "ps": ["host0:2222", "host1:2222"],
        "worker": ["host2:2222", "host3:2222", "host4:2222"],
    },
    "task": {"type": "ps", "index": 0},
    "environment": "cloud"
}'
```

TensorFlow 的高级别[ `tf.estimator` ](https://www.tensorflow.org/api_docs/python/tf/estimator) API，TensorFlow 将分析这`TF_CONFIG`变量和生成群集为您的规范。

对于 TensorFlow 的较低级别 core Api 进行培训，分析`TF_CONFIG`变量和生成`tf.train.ClusterSpec`培训代码中。 在[此示例](https://aka.ms/aml-notebook-tf-ps)中，你将在训练脚本中执行此操作，如下所示：

```Python
import os, json
import tensorflow as tf

tf_config = os.environ.get('TF_CONFIG')
if not tf_config or tf_config == "":
    raise ValueError("TF_CONFIG not found.")
tf_config_json = json.loads(tf_config)
cluster_spec = tf.train.ClusterSpec(cluster)

```

## <a name="keras-support"></a>Keras 支持

[Keras](https://keras.io/)是受欢迎的、 高级 DNN Python API，支持 TensorFlow、 CNTK、 和作为后端 Theano。 如果将 TensorFlow 用作后端，添加 Keras 非常简单，包括`pip_package`构造函数参数。

下面的示例实例化[ `TensorFlow` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py)估算器并将其提交试验一下。 将估计器运行 Keras 训练脚本`keras_train.py`。 作业将运行于启用 gpu[计算目标](how-to-set-up-training-targets.md)通过 Keras 作为依赖项通过 pip 安装。

```Python
from azureml.train.dnn import TensorFlow

keras_est = TensorFlow(source_directory='./my-keras-proj',
                       script_params=script_params,
                       compute_target=compute_target,
                       entry_script='keras_train.py', # relative path to your TensorFlow job
                       pip_packages=['keras'], # add keras through pip
                       use_gpu=True)
```

## <a name="export-to-onnx"></a>将导出到 ONNX

若要获取与优化的推断[ONNX 运行时](concept-onnx.md)，可以将已训练的 TensorFlow 模型转换为 ONNX 格式。 请参阅[示例](https://github.com/onnx/tensorflow-onnx/blob/master/examples/call_coverter_via_python.py)。

## <a name="examples"></a>示例

可以使用不同的框架上的单节点以及分布式 TensorFlow 执行找到的工作代码示例[我们的 GitHub 页面](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning)。

## <a name="next-steps"></a>后续步骤

* [在训练期间跟踪运行指标](how-to-track-experiments.md)
* [优化超参数](how-to-tune-hyperparameters.md)
* [部署定型的模型](how-to-deploy-and-where.md)
