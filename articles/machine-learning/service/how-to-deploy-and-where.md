---
title: 如何以及在何处部署模型
titleSuffix: Azure Machine Learning service
description: 了解如何以及在何处部署 Azure 机器学习服务模型，包括：Azure 容器实例、Azure Kubernetes 服务、Azure IoT Edge 和现场可编程门阵列。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 05/02/2019
ms.custom: seoapril2019
ms.openlocfilehash: 45421a249642abf37c89aa33e2e8a1b4a9e5e497
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507009"
---
# <a name="deploy-models-with-the-azure-machine-learning-service"></a>使用 Azure 机器学习服务部署模型

了解如何部署机器学习模型作为 web 服务在 Azure 云中，或 IoT Edge 设备。 本文档中的信息介绍了如何将部署到以下的计算目标：

| 计算目标 | 部署类型 | 描述 |
| ----- | ----- | ----- |
| [本地 web 服务](#local) | 测试/调试 | 适用于有限的测试和故障排除。
| [Azure Kubernetes 服务 (AKS)](#aks) | 实时推理 | 非常适合用于大规模生产部署。 提供了自动缩放和快速的响应时间。 |
| [Azure 容器实例 (ACI)](#aci) | 正在测试 | 非常适用于低规模，基于 CPU 的工作负荷。 |
| [Azure 机器学习计算](how-to-run-batch-predictions.md) | （预览版）批处理推理 | 运行批处理评分上无服务器计算。 支持的普通和低优先级 Vm。 |
| [Azure IoT Edge](#iotedge) | （预览版）IoT 模块 | 部署和 IoT 设备上提供机器学习模型。 |

## <a name="deployment-workflow"></a>部署工作流

为所有计算目标部署模型的过程类似：

1. 注册模型。
1. 部署模型。
1. 测试部署模型。

有关部署工作流涉及的概念的详细信息，请参阅[使用 Azure 机器学习服务管理、部署和监视模型](concept-model-management-and-deployment.md)。

## <a name="prerequisites-for-deployment"></a>部署的先决条件

- 模型。 如果你不具有定型的模型，可以使用模型中提供的依赖项文件[本教程](https://aka.ms/azml-deploy-cloud)。

- [机器学习服务的 Azure CLI 扩展](reference-azure-machine-learning-cli.md)，或[Azure 机器学习 Python SDK](https://aka.ms/aml-sdk)。

## <a id="registermodel"></a> 注册机器学习模型

模型注册表是在 Azure 云中存储和组织已训练模型的一种方式。 模型将注册到 Azure 机器学习服务工作区中。 使用 Azure 机器学习，或从其他位置训练的模型中导入，可以训练模型。 以下示例演示如何注册文件中的模型：

### <a name="register-a-model-from-an-experiment-run"></a>注册模型从试验运行

**使用 CLI Scikit-learn 示例**
```azurecli-interactive
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment
```
**使用 SDK**
```python
model = run.register_model(model_name='sklearn_mnist', model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep='\t')
```

### <a name="register-an-externally-created-model"></a>注册外部创建的模型
可以通过提供注册的外部创建的模型**本地路径**到模型。 您可以提供一个文件夹或单个文件。

**使用 Python SDK ONNX 示例：**
```python
onnx_model_url = "https://www.cntk.ai/OnnxModels/mnist/opset_7/mnist.tar.gz"
urllib.request.urlretrieve(onnx_model_url, filename="mnist.tar.gz")
!tar xvzf mnist.tar.gz

model = Model.register(workspace = ws,
                       model_path ="mnist/model.onnx",
                       model_name = "onnx_mnist",
                       tags = {"onnx": "demo"},
                       description = "MNIST image classification CNN from ONNX Model Zoo",)
```

**使用 CLI**
```azurecli-interactive
az ml model register -n onnx_mnist -p mnist/model.onnx
```

**时间估计**：大约 10 秒。

有关详细信息，请参阅 [Model 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py)的参考文档。

## <a name="how-to-deploy"></a>如何部署

若要部署为 web 服务，必须创建推理配置 (`InferenceConfig`) 和部署配置。 推理的配置来说，在指定的脚本和为模型提供服务所需的依赖项。 在部署配置指定如何计算目标上充当模型的详细信息。


### <a id="script"></a> 1.定义入口脚本和依赖项

入口脚本接收数据提交到已部署的 web 服务，并将其传递给模型。 然后，该脚本接收模型返回的响应，并将该响应返回给客户端。 **该脚本是特定于您的模型**; 它必须了解模型需要并返回的数据。

脚本包含两个函数，加载和运行模型：

* `init()`：此函数通常将模型载入全局对象。 此函数是只能运行一次你的 web 服务的 Docker 容器在启动时。

* `run(input_data)`：此函数使用模型来基于输入数据预测值。 运行的输入和输出通常使用 JSON 进行序列化和反序列化。 也可以处理原始二进制数据。 可以在将数据发送到模型之前或者返回给客户端之前转换数据。

#### <a name="optional-automatic-swagger-schema-generation"></a>（可选）自动生成的 Swagger 架构

若要自动生成 web 服务的架构、 提供输入的示例和/或输出中的其中一个已定义的类型对象的类型和示例的构造函数用于自动创建架构。 然后创建 azure 机器学习服务[OpenAPI](https://swagger.io/docs/specification/about/) (Swagger) 规范的 web 服务在部署过程。

目前支持以下类型：

* `pandas`
* `numpy`
* `pyspark`
* 标准 Python 对象

若要使用架构生成，包括`inference-schema`conda 环境文件中的包。 下面的示例使用`[numpy-support]`由于入口脚本使用 numpy 参数类型： 

#### <a name="example-dependencies-file"></a>示例依赖项文件
下面是推断的 Conda 依赖项文件的示例。
```python
name: project_environment
dependencies:
  - python=3.6.2
  - pip:
    - azureml-defaults
    - scikit-learn
    - inference-schema[numpy-support]
```

如果你想要使用自动架构生成，入口脚本**必须**导入`inference-schema`包。 

定义输入和输出中的示例格式`input_sample`和`output_sample`变量，表示 web 服务的请求和响应格式。 使用这些示例中输入和输出函数的修饰器`run()`函数。 Scikit-了解下面的示例使用架构生成。

> [!TIP]
> 在将服务部署之后, 使用`swagger_uri`属性来检索架构 JSON 文档。

#### <a name="example-entry-script"></a>示例条目脚本

下面的示例演示如何以接受和返回 JSON 数据：

**使用 Swagger 生成 Scikit 了解示例：**
```python
import json
import numpy as np
from sklearn.externals import joblib
from sklearn.linear_model import Ridge
from azureml.core.model import Model

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType

def init():
    global model
    # note here "sklearn_regression_model.pkl" is the name of the model registered under
    # this is a different behavior than before when the code is run locally, even though the code is the same.
    model_path = Model.get_model_path('sklearn_regression_model.pkl')
    # deserialize the model file back into a sklearn model
    model = joblib.load(model_path)

input_sample = np.array([[10,9,8,7,6,5,4,3,2,1]])
output_sample = np.array([3726.995])

@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        result = model.predict(data)
        # you can return any datatype as long as it is JSON-serializable
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

有关详细的示例脚本，请参阅下面的示例：

* Pytorch: [https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-pytorch](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-pytorch)
* TensorFlow: [https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-tensorflow](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-tensorflow)
* Keras: [https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-keras](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-keras)
* ONNX: [https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx/](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx/)
* 评分对二进制数据：[如何使用 web 服务](how-to-consume-web-service.md)

### <a name="2-define-your-inferenceconfig"></a>2.定义在 InferenceConfig

推理配置描述如何配置模型进行预测。 下面的示例演示如何创建推理配置：

```python
inference_config = InferenceConfig(source_directory="C:/abc",
                                   runtime= "python",
                                   entry_script="x/y/score.py",
                                   conda_file="env/myenv.yml")
```

在此示例中，配置包含以下各项：

* 包含资产需要向推理的目录
* 此模型需要 Python
* [入口脚本](#script)，用于处理 web 请求发送到已部署的服务
* 描述运行推断所需的 Python 包的 conda 文件

InferenceConfig 功能的信息，请参阅[高级的配置](#advanced-config)部分。

### <a name="3-define-your-deployment-configuration"></a>3.定义你的部署配置

在部署之前，必须定义部署配置。 部署配置是特定于将承载 web 服务的计算目标。 例如，在本地部署时必须指定其中服务接受请求的端口。

您可能还需要创建的计算资源。 例如，如果尚未这样做具有 Azure Kubernetes 服务关联与工作区。

下表提供了创建每个计算目标的部署配置的示例：

| 计算目标 | 部署配置示例 |
| ----- | ----- |
| 本地 | `deployment_config = LocalWebservice.deploy_configuration(port=8890)` |
| Azure 容器实例 | `deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |
| Azure Kubernetes 服务 | `deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |

以下各节演示如何创建部署配置，然后使用它来部署 web 服务。

## <a name="where-to-deploy"></a>部署位置

### <a id="local"></a> 在本地部署

本节在此示例使用[deploy_from_image](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#deploy-from-model-workspace--name--models--image-config--deployment-config-none--deployment-target-none-)，则需要您在执行部署之前注册的模型和图像。 有关其他部署方法的详细信息，请参阅[部署](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#deploy-workspace--name--model-paths--image-config--deployment-config-none--deployment-target-none-)并[deploy_from_model](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#deploy-from-model-workspace--name--models--image-config--deployment-config-none--deployment-target-none-)。

**若要将本地部署，需要在本地计算机上安装 docker。**

**使用 SDK**

```python
deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

**使用 CLI**

```azurecli-interactive
az ml model deploy -m sklearn_mnist:1 -ic inferenceconfig.json -dc deploymentconfig.json
```

### <a id="aci"></a> 将部署到 Azure 容器实例 （开发测试）

如果满足以下一个或多个条件，请使用 Azure 容器实例将模型部署为 Web 服务：
- 你需要快速部署并验证你的模型。
- 正在测试一个开发中的模型。 

若要查看 ACI 配额和区域可用性，请参阅[配额和区域可用性，Azure 容器实例的](https://docs.microsoft.com/azure/container-instances/container-instances-quotas)一文。

**使用 SDK**

```python
deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "aciservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

**使用 CLI**

```azurecli-interactive
az ml model deploy -m sklearn_mnist:1 -n aciservice -ic inferenceconfig.json -dc deploymentconfig.json
```

有关详细信息，请参阅 [AciWebservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py) 和 [Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.webservice?view=azure-ml-py) 类的参考文档。

### <a id="aks"></a> 将部署到 Azure Kubernetes 服务 （生产）

可以使用现有 AKS 群集，也可使用 Azure 机器学习 SDK、CLI 或 Azure 门户创建新群集。


> [!IMPORTANT]
> 对于工作区而言，创建 AKS 群集是一次性过程。 可以将此群集重复用于多个部署。
> 如果你不具有创建或附加 AKS 群集转<a href="#create-attach-aks">此处</a>。

#### 将部署到 AKS <a id="deploy-aks"></a>

你可以部署到 AKS 中使用 Azure 机器学习 CLI:
```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n aksservice -ic inferenceconfig.json -dc deploymentconfig.json
```

此外可以使用 Python SDK:
```python
aks_target = AksCompute(ws,"myaks")
deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "aksservice", [model], inference_config, deployment_config, aks_target)
service.wait_for_deployment(show_output = True)
print(service.state)
print(service.get_logs())
```

有关详细信息配置 AKS 部署，包括自动缩放，请参阅[AksWebservice.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.akswebservice)引用。

**预计时间：** 大约需要 5 分钟。

#### 创建或附加 AKS 群集 <a id="create-attach-aks"></a>
创建或附加 AKS 群集是**一个时间进程**工作区。 群集已与你的工作区关联后，您可以将其用于多个部署。 

如果你删除群集或包含该资源组，必须创建一个新的群集部署所需的下一个时间。

##### <a name="create-a-new-aks-cluster"></a>创建新的 AKS 群集
有关详细信息设置`autoscale_target_utilization`， `autoscale_max_replicas`，并`autoscale_min_replicas`，请参阅[AksWebservice.deploy_configuration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none-)引用。
下面的示例演示如何创建新的 Azure Kubernetes 服务群集：

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

创建 AKS 群集外部 Azure 机器学习 SDK 的详细信息，请参阅以下文章：
* [创建 AKS 群集](https://docs.microsoft.com/cli/azure/aks?toc=%2Fazure%2Faks%2FTOC.json&bc=%2Fazure%2Fbread%2Ftoc.json&view=azure-cli-latest#az-aks-create)
* [创建 AKS 群集 （门户）](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal?view=azure-cli-latest)


> [!IMPORTANT]
> 对于 [`provisioning_configuration()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py)，如果为 agent_count 和 vm_size 选择自定义值，则需要确保 agent_count 乘以 vm_size 的结果大于或等于 12 个虚拟 CPU。 例如，如果对 vm_size 使用“Standard_D3_v2”（有 4 个虚拟 CPU），则应该为 agent_count 选择 3 或更大的数字。

**时间估计**：大约 20 分钟。

##### <a name="attach-an-existing-aks-cluster"></a>附加现有的 AKS 群集

如果已在 Azure 订阅中有 AKS 群集并且它 1.12 版。 # # 和具有至少为 12 的虚拟 Cpu，可以使用它来部署你的映像。 下面的代码演示如何将附加现有的 AKS 1.12。 # # 为你的工作区的群集：

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'mycluster'

# Attach the cluster to your workgroup
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'mycompute', attach_config)
```

## <a name="consume-web-services"></a>使用 Web 服务
每个已部署的 web 服务提供一种 REST API，因此可以在各种编程语言中创建客户端应用程序。 如果已为你的服务中启用身份验证，你需要提供服务密钥作为请求标头中的令牌。

下面是如何调用你的服务在 Python 中的示例：
```python
import requests
import json

headers = {'Content-Type':'application/json'}

if service.auth_enabled:
    headers['Authorization'] = 'Bearer '+service.get_keys()[0]

print(headers)
    
test_sample = json.dumps({'data': [
    [1,2,3,4,5,6,7,8,9,10], 
    [10,9,8,7,6,5,4,3,2,1]
]})

response = requests.post(service.scoring_uri, data=test_sample, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

有关详细信息，请参阅[创建客户端应用程序以使用 Web 服务](how-to-consume-web-service.md)。

## <a id="update"></a> 更新 Web 服务

创建新模型时，必须手动更新每个服务，你想要使用新的模型。 若要更新 Web 服务，请使用 `update` 方法。 下面的代码演示如何更新 web 服务以使用新的模型：

```python
from azureml.core.webservice import Webservice
from azureml.core.model import Model

# register new model
new_model = Model.register(model_path = "outputs/sklearn_mnist_model.pkl",
                       model_name = "sklearn_mnist",
                       tags = {"key": "0.1"},
                       description = "test",
                       workspace = ws)

service_name = 'myservice'
# Retrieve existing service
service = Webservice(name = service_name, workspace = ws)

# Update to new model(s)
service.update(models = [new_model])
print(service.state)
print(service.get_logs())
```

## <a name="clean-up"></a>清理
若要删除已部署的 Web 服务，请使用 `service.delete()`。
若要删除已注册的模型，请使用 `model.delete()`。

有关详细信息，请参阅的参考文档[WebService.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py#delete--)，并[Model.delete()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#delete--)。

## 高级的配置设置 <a id="advanced-config"></a>

### <a id="customimage"></a> 使用自定义基本映像

在内部，InferenceConfig 创建包含模型和服务所需的其他资产的 Docker 映像。 如果未指定，则使用默认的基本映像。

创建映像以用于推断配置时，图像必须满足以下要求：

* Ubuntu 16.04 或更高版本。
* Conda 4.5。 # 或更高版本。
* Python 3.5。 # 或 3.6。 #。

若要使用自定义映像，设置`base_image`推理配置到的地址的图像的属性。 下面的示例演示如何使用从这两个公共和专用 Azure 容器注册表的映像：

```python
# use an image available in public Container Registry without authentication
inference_config.base_image = "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda"

# or, use an image available in a private Container Registry
inference_config.base_image = "myregistry.azurecr.io/mycustomimage:1.0"
inference_config.base_image_registry.address = "myregistry.azurecr.io"
inference_config.base_image_registry.username = "username"
inference_config.base_image_registry.password = "password"
```

下图 Uri 适用于由 Microsoft 提供的映像，并可以使用但未提供用户名称或密码值：

* `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0-cuda10.0-cudnn7`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0-tensorrt19.03`

若要使用这些映像，设置`base_image`到从上述列表的 URI。 将 `base_image_registry.address` 设置为 `mcr.microsoft.com`。

> [!IMPORTANT]
> 使用 CUDA 或 TensorRT 的 Microsoft 映像必须仅使用 Microsoft Azure 服务上。

将你自己的映像上传到 Azure 容器注册表的详细信息，请参阅[将第一个映像推送到专用 Docker 容器注册表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-docker-cli)。

如果您的模型训练 Azure 机器学习计算上，使用__1.0.22 版本或更高版本__的 Azure 机器学习 SDK，在定型过程中创建映像。 下面的示例演示如何使用此映像：

```python
# Use an image built during training with SDK 1.0.22 or greater
image_config.base_image = run.properties["AzureML.DerivedImageName"]
```

## <a name="other-inference-options"></a>其他推理选项

### <a id="azuremlcompute"></a> 批处理推理
创建和管理 Azure 机器学习服务的 azure 机器学习计算目标。 它们可以用于从 Azure 机器学习管道批处理预测。

使用 Azure 机器学习计算的批处理推理的演练，请阅读[如何运行批预测](how-to-run-batch-predictions.md)一文。

## <a id="iotedge"></a> IoT Edge 上的推理
部署到 edge 设备的支持处于预览状态。 有关详细信息，请参阅[部署 Azure 机器学习作为 IoT Edge 模块](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-machine-learning)一文。

## <a name="next-steps"></a>后续步骤
* [部署故障排除](how-to-troubleshoot-deployment.md)
* [使用 SSL 保护 Azure 机器学习 Web 服务](how-to-secure-web-service.md)
* [使用部署为 Web 服务的机器学习模型](how-to-consume-web-service.md)
* [使用 Application Insights 监视 Azure 机器学习模型](how-to-enable-app-insights.md)
* [为生产环境中的模型收集数据](how-to-enable-data-collection.md)
