---
title: 图像分类教程：训练模型
titleSuffix: Azure Machine Learning service
description: 学习如何使用 Azure 机器学习服务在 Python Jupyter notebook 中使用 scikit-learn 训练映像分类模型。 本教程是由两个部分构成的系列教程的第一部分。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 05/08/2019
ms.custom: seodec18
ms.openlocfilehash: 097fb3422ce3868d9ef499ad6c92c8b7fa12e852
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442065"
---
# <a name="tutorial-train-image-classification-models-with-mnist-data-and-scikit-learn-using-azure-machine-learning"></a>教程：使用 Azure 机器学习通过 MNIST 数据和 scikit-learn 训练映像分类模型

在本教程中，你将在远程计算资源上训练一个机器学习模型。 将在 Python Jupyter Notebook 中使用 Azure 机器学习服务（预览）的定型和部署工作流。  然后可以将 Notebook 用作模板，使用你自己的数据来定型机器学习。 本教程是由两个部分构成的系列教程的第一部分。  

本教程将 [MNIST](http://yann.lecun.com/exdb/mnist/) 数据集和 [scikit-learn](https://scikit-learn.org) 与 Azure 机器学习服务配合使用来定型简单的逻辑回归。 MNIST 是包含 70,000 张灰度图像的常用数据集。 每个图像是 28 x 28 像素的手写数字，代表一个从零到九的数字。 目标是创建多类分类器，以确定给定图像代表的数字。 

了解如何执行以下操作：

> [!div class="checklist"]
> * 设置开发环境。
> * 访问和检查数据。
> * 在远程群集上训练一个简单的逻辑回归模型。
> * 查看定型结果，然后注册最佳模型。

你会在[本教程的第二部分](tutorial-deploy-models-with-aml.md)学习如何选择模型并对其进行部署。 

如果没有 Azure 订阅，请在开始之前创建一个免费帐户。 立即试用 [Azure 机器学习服务免费版或付费版](https://aka.ms/AMLFree)。

>[!NOTE]
> 本文中的代码已使用 Azure 机器学习 SDK 版本 1.0.8 进行测试。

## <a name="prerequisites"></a>先决条件

跳到[设置开发环境](#start)来了解整个 Notebook 设置步骤，或遵照以下说明获取 Notebook 并在 Azure Notebooks 或自己的 Notebook 服务器中运行。  若要运行 Notebook，需要：

* 一个装有以下组件的 Python 3.6 Notebook 服务器：
    * 适用于 Python 的 Azure 机器学习 SDK
    * `matplotlib` 和 `scikit-learn`
* 教程 Notebook 和文件 **utils.py**
* 机器学习工作区
* Notebook 所在的同一目录中的工作区的配置文件

从以下任一部分获取所有这些必备组件。
 
* 使用[工作区中的云 Notebook 服务器](#azure) 
* 使用[自己的 Notebook 服务器](#server)

### <a name="azure"></a>使用工作区中的云 Notebook 服务器

可以轻松地从你自己的基于云的 Notebook 服务器着手。 创建此云资源后，[用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 就已为你安装和配置了。

[!INCLUDE [aml-azure-notebooks](../../../includes/aml-azure-notebooks.md)]

* 启动 Notebook 网页后，打开 **tutorials/img-classification-part1-training.ipynb** Notebook。


### <a name="server"></a>使用自己的 Jupyter Notebook 服务器

[!INCLUDE [aml-your-server](../../../includes/aml-your-server.md)]

 完成这些步骤后，从克隆的目录运行 **tutorials/img-classification-part1-training.ipynb** Notebook。

## <a name="start"></a>设置开发环境

开发工作的所有设置都可以在 Python Notebook 中完成。 安装包括以下操作：

* 导入 Python 包。
* 连接到工作区，以便本地计算机能够与远程资源通信。
* 创建试验来跟踪所有运行。
* 创建要用于定型的远程计算目标。

### <a name="import-packages"></a>导入包

导入在此会话中所需的 Python 包。 此外显示 Azure 机器学习 SDK 版本：

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt

import azureml.core
from azureml.core import Workspace

# check core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="connect-to-a-workspace"></a>连接到工作区

从现有工作区创建工作区对象。 `Workspace.from_config()` 读取文件 config.json 并将详细信息加载到一个名为 `ws` 的对象：

```python
# load workspace configuration from the config.json file in the current folder.
ws = Workspace.from_config()
print(ws.name, ws.location, ws.resource_group, ws.location, sep = '\t')
```

### <a name="create-an-experiment"></a>创建试验

创建一个试验来跟踪工作区中的运行。 一个工作区可有多个试验： 

```python
experiment_name = 'sklearn-mnist'

from azureml.core import Experiment
exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="create-or-attach-an-existing-compute-resource"></a>创建或附加现有的计算资源

Azure 机器学习计算是一项托管服务，可让数据科学家在 Azure 虚拟机群集上训练机器学习模型。 示例包括带 GPU 支持的 VM。 在本教程中，你将创建 Azure 机器学习计算作为训练环境。 如果工作区中尚无计算群集，以下代码将创建计算群集。

 创建计算群集需要大约 5 分钟。 如果工作区中已有计算，该代码将使用它，并跳过创建过程。


```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
import os

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpucluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = vm_size,
                                                                min_nodes = compute_min_nodes, 
                                                                max_nodes = compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

现在已有所需的包和计算资源，可以在云中定型模型。 

## <a name="explore-data"></a>浏览数据

对模型进行定型之前，需要了解用于定型的数据。 此外，还需要将数据复制到云中。 然后，云培训环境可以对其进行访问。 在本部分中，了解如何执行以下操作：

* 下载 MNIST 数据集。
* 显示一些示例图像。
* 将数据上传到云。

### <a name="download-the-mnist-dataset"></a>下载 MNIST 数据集

下载 MNIST 数据集，并将文件保存到本地 `data` 目录。 下载用于定型和测试的图像和标签：


```python
import urllib.request
import os

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok = True)

urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz', filename=os.path.join(data_folder, 'train-images.gz'))
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz', filename=os.path.join(data_folder, 'train-labels.gz'))
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename=os.path.join(data_folder, 'test-images.gz'))
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename=os.path.join(data_folder, 'test-labels.gz'))
```
将显示类似于下面的输出：```('./data/test-labels.gz', <http.client.HTTPMessage at 0x7f40864c77b8>)```

### <a name="display-some-sample-images"></a>显示一些示例图像

将压缩文件加载到 `numpy` 数组。 然后，使用 `matplotlib` 从数据集随意绘制 30 张图像，并在上方附加标签。 此步骤需要 `util.py` 文件中包含的 `load_data` 函数。 此文件包含在示例文件夹中。 确保它与此 Notebook 放在同一文件夹中。 `load_data` 函数直接将压缩文件解析为 numpy 数组：



```python
# make sure utils.py is in the same directory as this code
from utils import load_data

# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the model converge faster.
X_train = load_data(os.path.join(data_folder, 'train-images.gz'), False) / 255.0
X_test = load_data(os.path.join(data_folder, 'test-images.gz'), False) / 255.0
y_train = load_data(os.path.join(data_folder, 'train-labels.gz'), True).reshape(-1)
y_test = load_data(os.path.join(data_folder, 'test-labels.gz'), True).reshape(-1)

# now let's show some randomly chosen images from the traininng set.
count = 0
sample_size = 30
plt.figure(figsize = (16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x=10, y=-10, s=y_train[i], fontsize=18)
    plt.imshow(X_train[i].reshape(28, 28), cmap=plt.cm.Greys)
plt.show()
```

随机图像示例显示：

![随机图像示例](./media/tutorial-train-models-with-aml/digits.png)

现在你已了解这些图像的外观和预期预测结果。

### <a name="upload-data-to-the-cloud"></a>将数据上传到云

现在，通过将数据从本地计算机上传到 Azure，使该数据可以远程访问。 然后，远程定型时可以访问这些数据。 数据存储是与你的工作区相关联的方便构造，用于上传/下载数据。 可以使用数据存储与远程计算目标交互。 它由 Azure blob 存储帐户提供支持。

MNIST 文件被上传到数据存储根目录下一个名为 `mnist` 的目录：

```python
ds = ws.get_default_datastore()
print(ds.datastore_type, ds.account_name, ds.container_name)

ds.upload(src_dir=data_folder, target_path='mnist', overwrite=True, show_progress=True)
```
现在你已经拥有开始定型模型所需的一切条件。 


## <a name="train-on-a-remote-cluster"></a>在远程群集上定型

对于此任务，将作业提交到之前设置的远程定型群集。  若要提交作业：
* 创建目录
* 创建定型脚本
* 创建估算器对象
* 提交作业 

### <a name="create-a-directory"></a>创建目录

创建一个目录，将所需的代码从计算机发送到远程资源。

```python
import os
script_folder  = os.path.join(os.getcwd(), "sklearn-mnist")
os.makedirs(script_folder, exist_ok=True)
```

### <a name="create-a-training-script"></a>创建定型脚本

若要将作业提交到群集，首先创建定型脚本。 运行以下代码，以在刚创建的目录中创建名为 `train.py` 的定型脚本。

```python
%%writefile $script_folder/train.py

import argparse
import os
import numpy as np

from sklearn.linear_model import LogisticRegression
from sklearn.externals import joblib

from azureml.core import Run
from utils import load_data

# let user feed in 2 parameters, the location of the data files (from datastore), and the regularization rate of the logistic regression model
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# load train and test set into numpy arrays
# note we scale the pixel intensity values to 0-1 (by dividing it with 255.0) so the model can converge faster.
X_train = load_data(os.path.join(data_folder, 'train-images.gz'), False) / 255.0
X_test = load_data(os.path.join(data_folder, 'test-images.gz'), False) / 255.0
y_train = load_data(os.path.join(data_folder, 'train-labels.gz'), True).reshape(-1)
y_test = load_data(os.path.join(data_folder, 'test-labels.gz'), True).reshape(-1)
print(X_train.shape, y_train.shape, X_test.shape, y_test.shape, sep = '\n')

# get hold of the current run
run = Run.get_context()

print('Train a logistic regression model with regularization rate of', args.reg)
clf = LogisticRegression(C=1.0/args.reg, random_state=42)
clf.fit(X_train, y_train)

print('Predict the test set')
y_hat = clf.predict(X_test)

# calculate accuracy on the prediction
acc = np.average(y_hat == y_test)
print('Accuracy is', acc)

run.log('regularization rate', np.float(args.reg))
run.log('accuracy', np.float(acc))

os.makedirs('outputs', exist_ok=True)
# note file saved in the outputs folder is automatically uploaded into experiment record
joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')
```

请注意该脚本获取数据和保存模型的方式：

+ 定型脚本读取参数以查找包含数据的目录。 稍后提交作业时，请参考数据存储获取此参数：```parser.add_argument('--data-folder', type=str, dest='data_folder', help='data directory mounting point')```

+ 训练脚本将模型保存到一个名为**outputs** 的目录中。 此目录中编写的所有内容都会自动上传到你的工作区。 稍后，你将在本教程从此目录访问模型。 `joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')`

+ 训练脚本需要使用文件 `utils.py` 来正确加载数据集。 以下代码将 `utils.py` 复制到 `script_folder` 中，以便可以与远程资源上的训练脚本一起访问该文件。

  ```python
  import shutil
  shutil.copy('utils.py', script_folder)
  ```


### <a name="create-an-estimator"></a>创建估算器

估算器对象用于提交运行。 通过运行以下代码创建估算器以定义以下项：

* 估算器对象的名称，`est`。
* 包含脚本的目录。 此目录中的所有文件都上传到群集节点以便执行。 
* 计算目标。 在此示例中，将使用所创建的 Azure 机器学习计算群集。
* 定型脚本名称，train.py。
* 定型脚本所需的参数。 
* 定型所需的 Python 包。

在本教程中，此目标是 AmlCompute。 此脚本文件夹中的所有文件都上传到群集节点来运行。 data_folder 设置为使用数据存储 (`ds.path('mnist').as_mount()`)：

```python
from azureml.train.estimator import Estimator

script_params = {
    '--data-folder': ds.path('mnist').as_mount(),
    '--regularization': 0.8
}

est = Estimator(source_directory=script_folder,
                script_params=script_params,
                compute_target=compute_target,
                entry_script='train.py',
                conda_packages=['scikit-learn'])
```


### <a name="submit-the-job-to-the-cluster"></a>将作业提交到群集

通过提交估算器对象来运行此试验：

```python
run = exp.submit(config=est)
run
```

由于调用是异步的，因此一旦作业启动，它就会返回“正在准备”或“正在运行”状态。

## <a name="monitor-a-remote-run"></a>监视远程运行

总的来说，首次运行需要大约 10 分钟。 但对于后续运行，只要不更改脚本依赖项，将重复使用相同的映像。 因此容器启动时间要快得多。

等待时会发生以下情况：

- **映像创建**：将创建与估算器指定的 Python 环境相匹配的 Docker 映像。 映像将上传到工作区。 创建和上传映像需要大约五分钟。 

  此阶段针对每个 Python 环境发生一次，因为已缓存容器用于后续运行。 映像创建期间，日志将流式传输到运行历史记录。 可以使用这些日志监视映像创建进度。

- **缩放**：如果远程群集需要比当前可用节点更多的节点来执行运行，则会自动添加其他节点。 缩放通常需要大约五分钟。

- **Running**：在此阶段，必要的脚本和文件会发送到计算目标。 接着，装载或复制数据存储。 然后，运行 entry_script。 运行作业时，stdout 和 ./logs 目录会流式传输到运行历史记录。 可以使用这些日志监视运行进度。

- **后期处理**：运行的 ./outputs 目录将复制到工作区中的运行历史记录，以便可以访问这些结果。


可以通过多种方式查看正在运行的作业的进度。 本教程使用 Jupyter 小组件和 `wait_for_completion` 方法。 

### <a name="jupyter-widget"></a>Jupyter 小组件

查看使用 Jupyter 小组件的运行进度。 和运行提交一样，该小组件采用异步方式，并每隔 10 到 15 秒提供实时更新，直到完成作业：


```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

以下是定型结束时显示的小组件的静止快照：

![Notebook 小组件](./media/tutorial-train-models-with-aml/widget.png)

如果需要取消运行，可以遵照[这些说明](https://aka.ms/aml-docs-cancel-run)。

### <a name="get-log-results-upon-completion"></a>完成时获取日志结果

模型定型和监视在后台发生。 在运行更多代码之前，请耐心等待，直到该模型完成定型。 使用 `wait_for_completion` 显示模型定型过程何时完成： 


```python
run.wait_for_completion(show_output=False) # specify True for a verbose log
```

### <a name="display-run-results"></a>显示运行结果

现在你拥有一个在远程群集上定型的模型。 检索模型的准确性：

```python
print(run.get_metrics())
```
输出显示远程模型的准确度为 0.9204：

`{'regularization rate': 0.8, 'accuracy': 0.9204}`

下一教程将详细介绍此模型。

## <a name="register-model"></a>注册模型

定型脚本的最后一步在运行作业的群集 VM 内名为 `outputs` 的目录中编写文件 `outputs/sklearn_mnist_model.pkl`。 `outputs` 是一个专门目录，此目录中的所有内容都会自动都上传到工作区。 此内容在工作区下试验的运行记录中显示。 因此，模型文件现还在工作区中提供。

可以看到与该运行关联的文件：

```python
print(run.get_file_names())
```

在工作区中注册模型，以便稍后你或其他协作者可以查询、检查和部署该模型：

```python
# register model 
model = run.register_model(model_name='sklearn_mnist', model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep = '\t')
```

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

此外可以只删除 Azure 机器学习计算群集。 但是，自动缩放已打开，并且群集最小值为零。 因此，未使用此特定资源时，便不会产生额外计算费用：


```python
# optionally, delete the Azure Machine Learning Compute cluster
compute_target.delete()
```

## <a name="next-steps"></a>后续步骤

在本 Azure 机器学习服务教程中，已使用 Python 执行以下任务：

> [!div class="checklist"]
> * 设置开发环境。
> * 访问和检查数据。
> * 使用流行的 scikit-learn 机器学习库训练远程群集上的多个模型
> * 查看定型详细信息，然后注册最佳模型。

可以使用本系列教程的下一部分中的说明来部署此注册模型：

> [!div class="nextstepaction"]
> [教程 2 - 部署模型](tutorial-deploy-models-with-aml.md)
