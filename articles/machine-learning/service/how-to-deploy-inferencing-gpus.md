---
title: 如何部署用于推断具有 GPU 的深度学习模型
titleSuffix: Azure Machine Learning service
description: 了解如何将深度学习模型部署为 web 服务使用 GPU 进行推断。 在本文中，Tensorflow 模型部署到的 Azure Kubernetes 服务群集。 该群集使用已启用 GPU 的虚拟机以托管 web 服务和评分推断请求。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: vaidyas
author: csteegz
ms.reviewer: larryfr
ms.date: 05/02/2019
ms.openlocfilehash: 5cc0fe0526937245d3ca913afc477f0259e2afd4
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65515169"
---
# <a name="how-to-do-gpu-inferencing"></a>如何执行 GPU 推断

了解如何使用 GPU 推断的机器学习模型部署为 web 服务。 在本文中，您将了解如何使用 Azure 机器学习服务来部署示例 Tensorflow 深入学习模型。 模型部署到使用已启用 GPU 的 VM 来承载服务的 Azure Kubernetes 服务 (AKS) 群集。 当请求被发送到服务时，该模型将使用 GPU 进行推断。

Gpu 提供高度可并行化的计算上 Cpu 的性能优于。 定型和推断深度学习模型 （尤其是对于大型批处理中的请求） 是很好的使用情况下为 Gpu。  

此示例将演示如何将保存的 TensorFlow 模型部署到 Azure 机器学习。 

## <a name="goals-and-prerequisites"></a>目标和先决条件

遵照说明操作：
* 创建启用了 GPU AKS 群集
* 部署具有 Tensorflow GPU 的模型

先决条件：
* Azure 机器学习服务工作区
* Python
* Tensorflow SavedModel 注册。 若要了解如何注册模型请参阅[部署模型](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-and-where#registermodel)

这篇文章基于[部署到 AKS Tensorflow 模型](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/production-deploy-to-aks-gpu/production-deploy-to-aks-gpu.ipynb)，它使用保存的 TensorFlow 模型并将部署到 AKS 群集。 但是，使用的计分概要文件和环境文件的小改动是适用于任何支持 Gpu 的机器学习框架。  

## <a name="provision-aks-cluster-with-gpus"></a>使用 Gpu 的预配 AKS 群集
Azure 有许多不同的 GPU 选项，所有这些可用于推断。 请参阅[的 N 系列列表](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#n-series)有关功能和成本的完整明细。 

AKS 使用 Azure 机器学习服务的详细信息，请参阅[如何部署和其中的文章。](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-and-where#create-a-new-cluster)

```python
# Provision AKS cluster with GPU machine
prov_config = AksCompute.provisioning_configuration(vm_size="Standard_NC6")

# Create the cluster
aks_target = ComputeTarget.create(
    workspace=ws, name=aks_name, provisioning_configuration=prov_config
)

aks_target.wait_for_deployment()
```

> [!IMPORTANT]
> Azure 将向你收费，只要 AKS 群集预配。 请务必删除 AKS 群集，完成后使用它。


## <a name="write-entry-script"></a>编写入口脚本

将以下内容保存到工作目录为`score.py`。 此文件将用于对图像评分与将它们发送到你的服务。 此文件加载已保存模型，TensorFlow，然后在每个 POST 请求将传递到 TensorFlow 会话输入的图像并返回生成的分数。
其他推断框架需要不同的计分概要文件。

```python
import tensorflow as tf
import numpy as np
import ujson
from azureml.core.model import Model
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse

def init():
    global session
    global input_name
    global output_name
    
    session = tf.Session()

    model_path = Model.get_model_path('resnet50')
    model = tf.saved_model.loader.load(session, ['serve'], model_path)
    if len(model.signature_def['serving_default'].inputs) > 1:
        raise ValueError("This score.py only supports one input")
    input_name = [tensor.name for tensor in model.signature_def['serving_default'].inputs.values()][0]
    output_name = [tensor.name for tensor in model.signature_def['serving_default'].outputs.values()]
    

@rawhttp
def run(request):
    if request.method == 'POST':
        reqBody = request.get_data(False)
        resp = score(reqBody)
        return AMLResponse(resp, 200)
    if request.method == 'GET':
        respBody = str.encode("GET is not supported")
        return AMLResponse(respBody, 405)
    return AMLResponse("bad request", 500)

def score(data):
    result = session.run(output_name, {input_name: [data]})
    return ujson.dumps(result[1])

if __name__ == "__main__":
    init()
    with open("lynx.jpg", 'rb') as f: #load file for testing locally
        content = f.read()
        print(score(content))

```

## <a name="define-conda-environment"></a>定义 Conda 环境
创建一个名为的 conda 环境文件`myenv.yml`以指定你的服务的依赖关系。 务必要指定要使用`tensorflow-gpu`以获得更快的性能。
```yaml
name: aml-accel-perf
channels:
  - defaults
dependencies:
  - tensorflow-gpu = 1.12
  - numpy
  - ujson
  - pip:
    - azureml-core
    - azureml-contrib-services
```

## <a name="define-gpu-inferenceconfig"></a>定义 GPU InferenceConfig

创建[ `InferenceConfig` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py)它指定要启用 GPU。 这将确保 CUDA 随你的映像。

```python
from azureml.core.model import Model
from azureml.core.model import InferenceConfig

aks_service_name ='gpu-rn'
gpu_aks_config = AksWebservice.deploy_configuration(autoscale_enabled = False, 
                                                    num_replicas = 3, 
                                                    cpu_cores=2, 
                                                    memory_gb=4)
model = Model(ws,"resnet50")

inference_config = InferenceConfig(runtime= "python", 
                                   entry_script="score.py",
                                   conda_file="myenv.yml", 
                                   gpu_enabled=True)
```

有关详细信息，请参阅[InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py)并[AksServiceDeploymentConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py)。
## <a name="deploy-the-model"></a>部署模型

将模型部署到 AKS 群集，并等待它来创建你的服务。

```python
aks_service = Model.deploy(ws,
                           models=[model],
                           inference_config=inference_config, 
                           deployment_config=aks_config,
                           deployment_target=aks_target,
                           name=aks_service_name)

aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

> [!NOTE]
> Azure 机器学习服务不会将部署一个具有模型`InferenceConfig`到而无需 GPU 群集需要 GPU。

有关详细信息，请参阅[模型](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py)。

## <a name="issue-sample-query-to-deployed-model"></a>模型部署到的问题示例查询

示例查询发给你已部署的模型。 此模型将评分为 post 请求发送到它任何 jpeg 图像。 

```python
scoring_url = aks_service.scoring_uri
api_key = aks_service.get_key()(0)
IMAGEURL = "https://upload.wikimedia.org/wikipedia/commons/thumb/6/68/Lynx_lynx_poing.jpg/220px-Lynx_lynx_poing.jpg"

headers = {'Authorization':('Bearer '+ api_key)}
img_data = read_image_from(IMAGEURL).read()
r = requests.post(scoring_url, data = img_data, headers=headers)
```

> [!IMPORTANT]
> 为优化延迟和吞吐量，客户端应位于终结点所在的同一 Azure 区域中。  目前是在美国东部 Azure 区域创建 API。

## <a name="cleaning-up-the-resources"></a>清理资源

与此演示程序完成后，请删除你的资源。

> [!IMPORTANT]
> Azure 将向你基于多长时间部署 AKS 群集。 请确保使用它完成后清理。

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>后续步骤

* [部署模型上 FPGA](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-fpga-web-service)
* [部署具有 ONNX 模型](https://docs.microsoft.com/azure/machine-learning/service/how-to-build-deploy-onnx#deploy)
* [训练的 DNN Tensorflow 模型](https://docs.microsoft.com/azure/machine-learning/service/how-to-train-tensorflow)
