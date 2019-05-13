---
title: 在 FPGA 上部署模型
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务通过在 FPGA 上运行模型部署 Web 服务来实现超低延迟推断。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: larryfr
ms.author: tedway
author: tedway
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 249a21bf9eeb3913826971fd1aae136197d264c4
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149613"
---
# <a name="deploy-a-model-as-a-web-service-on-an-fpga-with-azure-machine-learning-service"></a>使用 Azure 机器学习服务将模型部署为 FPGA 上的 Web 服务

可将模型作为 web 服务部署上[字段可编程门阵列 (Fpga)](concept-accelerate-with-fpgas.md)与 Azure 机器学习硬件加速模型。 使用 FPGA 可实现超低的延迟推断，即使只有一个批数据大小。

当前提供了以下模型：
  - ResNet 50
  - ResNet 152
  - DenseNet-121
  - VGG-16
  - SSD-VGG

Fpga 在这些 Azure 区域均提供：
  - 美国东部
  - 东南亚
  - 西欧
  - 美国西部 2

> [!IMPORTANT]
> 若要优化的延迟和吞吐量，您将数据发送到 FPGA 模型的客户端应为上面 （部署到该模型的一个） 的区域之一。

## <a name="prerequisites"></a>必备组件

- Azure 订阅。  如果您确实有一个，请在开始之前创建一个免费帐户。 立即试用 [Azure 机器学习服务免费版或付费版](https://aka.ms/AMLFree)。

- FPGA 配额。  使用 Azure CLI 查看是否有配额。
    ```shell
    az vm list-usage --location "eastus" -o table
    ```

    其他位置相距``southeastasia``， ``westeurope``，和``westus2``。

    在"名称"列下，寻找"标准 PBS 系列 Vcpu"，并确保有最少为 6 Vcpu 下"CurrentValue。"

    如果没有配额，则提交请求表单[此处](https://aka.ms/accelerateAI)。

- 已安装 Azure 机器学习服务工作区，以及适用于 Python 的 Azure 机器学习 SDK。 了解如何通过[如何配置开发环境](how-to-configure-environment.md)一文满足这些先决条件。
 
- 硬件加速的模型的 Python SDK:

    ```shell
    pip install --upgrade azureml-accel-models
    ```

## <a name="sample-notebooks"></a>示例笔记本

为方便起见，[示例笔记本](https://aka.ms/aml-accel-models-notebooks)适用于下面的示例和其他示例。

## <a name="create-and-containerize-your-model"></a>创建和容器化您的模型

此文档将介绍如何创建 TensorFlow 的关系图以预处理输入的图像，使其在 FPGA 上使用 ResNet 50 特征化器，然后运行通过 ImageNet 数据集上训练的分类器的功能。

遵照说明操作：

* 定义 TensorFlow 模型
* 转换模型
* 部署模型
* 使用已部署的模型
* 删除已部署的服务

### <a name="load-azure-ml-workspace"></a>加载 Azure 机器学习工作区

加载你的 Azure 机器学习工作区。

```python
import os
import tensorflow as tf
 
from azureml.core import Workspace

ws = Workspace.from_config()
print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep = '\n')
```

### <a name="preprocess-image"></a>预处理图像

Web 服务的输入是 JPEG 图像。  第一步是进行解码 JPEG 图像，并对其进行预处理。  JPEG 图像被视为字符串并将该结果是 tensors 将 ResNet 50 模型的输入。

```python
# Input images as a two-dimensional tensor containing an arbitrary number of images represented a strings
import azureml.accel.models.utils as utils
tf.reset_default_graph()

in_images = tf.placeholder(tf.string)
image_tensors = utils.preprocess_array(in_images)
print(image_tensors.shape)
```

### <a name="load-featurizer"></a>负载特征化器

初始化模型并下载 ResNet50 的量化版本的 TensorFlow 检查点以用作特征化器。  通过导入其他深度神经网络，可能会替换"QuantizedResnet50"中使用以下代码段：

- QuantizedResnet152
- QuantizedVgg16
- Densenet121

```python
from azureml.accel.models import QuantizedResnet50
save_path = os.path.expanduser('~/models')
model_graph = QuantizedResnet50(save_path, is_frozen = True)
feature_tensor = model_graph.import_graph_def(image_tensors)
print(model_graph.version)
print(feature_tensor.name)
print(feature_tensor.shape)
```

### <a name="add-classifier"></a>添加分类器

已在 ImageNet 数据集上训练该分类器。  有关传输学习和培训您自定义的权重的示例包括可用的集中[示例笔记本](https://aka.ms/aml-notebooks)。

```python
classifier_output = model_graph.get_default_classifier(feature_tensor)
print(classifier_output)
```

### <a name="save-the-model"></a>保存模型

现在，已加载预处理器、 ResNet 50 特征化器和分类器，将为模型的图形和相关联的变量。

```python
model_name = "resnet50"
model_save_path = os.path.join(save_path, model_name)
print("Saving model in {}".format(model_save_path))

with tf.Session() as sess:
    model_graph.restore_weights(sess)
    tf.saved_model.simple_save(sess, model_save_path,
                                   inputs={'images': in_images},
                                   outputs={'output_alias': classifier_output})
```

### <a name="save-input-and-output-tensors"></a>保存输入和输出 tensors
在预处理和分类器步骤期间创建输入和输出 tensors 将需要为模型转换和推理。

```python
input_tensors = in_images.name
output_tensors = classifier_output.name

print(input_tensors)
print(output_tensors)
```

> [!IMPORTANT]
> 保存输入和输出 tensors，因为稍后需要它们的模型转换和推断请求。

可用模型，并将相应的默认分类器的输出如下 tensors，这是你将使用在推断过程如果使用默认分类器。

+ Resnet50, QuantizedResnet50 ``
output_tensors = "classifier_1/resnet_v1_50/predictions/Softmax:0"
``
+ Resnet152 QuantizedResnet152 ``
output_tensors = "classifier/resnet_v1_152/predictions/Softmax:0"
``
+ Densenet121 QuantizedDensenet121 ``
output_tensors = "classifier/densenet121/predictions/Softmax:0"
``
+ Vgg16 QuantizedVgg16 ``
output_tensors = "classifier/vgg_16/fc8/squeezed:0"
``
+ SsdVgg QuantizedSsdVgg ``
output_tensors = ['ssd_300_vgg/block4_box/Reshape_1:0', 'ssd_300_vgg/block7_box/Reshape_1:0', 'ssd_300_vgg/block8_box/Reshape_1:0', 'ssd_300_vgg/block9_box/Reshape_1:0', 'ssd_300_vgg/block10_box/Reshape_1:0', 'ssd_300_vgg/block11_box/Reshape_1:0', 'ssd_300_vgg/block4_box/Reshape:0', 'ssd_300_vgg/block7_box/Reshape:0', 'ssd_300_vgg/block8_box/Reshape:0', 'ssd_300_vgg/block9_box/Reshape:0', 'ssd_300_vgg/block10_box/Reshape:0', 'ssd_300_vgg/block11_box/Reshape:0']
``

### <a name="register-model"></a>注册模型

[注册](./concept-model-management-and-deployment.md)你创建的模型。  添加标记和其他模型的元数据可帮助你跟踪已训练的模型。

```python
from azureml.core.model import Model

registered_model = Model.register(workspace = ws,
                                  model_path = model_save_path,
                                  model_name = model_name)

print("Successfully registered: ", registered_model.name, registered_model.description, registered_model.version, sep = '\t')
```

如果你已经注册了一个模型并想要将其加载，您可能会检索它。

```python
from azureml.core.model import Model
model_name = "resnet50"
# By default, the latest version is retrieved. You can specify the version, i.e. version=1
registered_model = Model(ws, name="resnet50")
print(registered_model.name, registered_model.description, registered_model.version, sep = '\t')
```

### <a name="convert-model"></a>转换模型

TensorFlow 图形转换为开放神经网络交换格式 ([ONNX](https://onnx.ai/))。  将需要提供的输入和输出 tensors 中，名称和你的客户端将使用这些名称，当您使用 web 服务。

```python
from azureml.accel import AccelOnnxConverter

convert_request = AccelOnnxConverter.convert_tf_model(ws, registered_model, input_tensors, output_tensors)

# If it fails, you can run wait_for_completion again with show_output=True.
convert_request.wait_for_completion(show_output = False)

# If the above call succeeded, get the converted model
converted_model = convert_request.result
print("\nSuccessfully converted: ", converted_model.name, converted_model.url, converted_model.version, 
      converted_model.id, converted_model.created_time, '\n')
```

### <a name="create-docker-image"></a>创建 Docker 映像

转换后的模型和所有依赖项添加到 Docker 映像。  然后，将此 Docker 映像部署，将其实例化。  支持的部署目标云或边缘设备中包括 AKS 如下所述[Azure 数据框边缘](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview)。  此外可以为已注册的 Docker 映像中添加标记和说明。

```python
from azureml.core.image import Image
from azureml.accel import AccelContainerImage

image_config = AccelContainerImage.image_configuration()
# Image name must be lowercase
image_name = "{}-image".format(model_name)

image = Image.create(name = image_name,
                     models = [converted_model],
                     image_config = image_config, 
                     workspace = ws)
image.wait_for_creation(show_output = False)
```

按标记中列出的映像并获取详细的日志的任何调试。

```python
for i in Image.list(workspace = ws):
    print('{}(v.{} [{}]) stored at {} with build log {}'.format(i.name, i.version, i.creation_state, i.image_location, i.image_build_log_uri))
```

## <a name="model-deployment"></a>模型部署

### <a name="deploy-to-the-cloud"></a>将部署到云

若要将模型部署为大规模生产 Web 服务，请使用 Azure Kubernetes 服务 (AKS)。 可以创建新组使用 Azure 机器学习 SDK、 CLI 或 Azure 门户。

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Specify the Standard_PB6s Azure VM
prov_config = AksCompute.provisioning_configuration(vm_size = "Standard_PB6s",
                                                    agent_count = 1)

aks_name = 'my-aks-cluster'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws, 
                                  name = aks_name, 
                                  provisioning_configuration = prov_config)
```

AKS 部署可能需要大约 15 分钟。  检查部署是否成功。

```python
aks_target.wait_for_completion(show_output = True)
print(aks_target.provisioning_state)
print(aks_target.provisioning_errors)
```

将容器部署到 AKS 群集。
```python
from azureml.core.webservice import Webservice, AksWebservice

# For this deployment, set the web service configuration without enabling auto-scaling or authentication for testing
aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                num_replicas=1,
                                                auth_enabled = False)

aks_service_name ='my-aks-service'

aks_service = Webservice.deploy_from_image(workspace = ws,
                                           name = aks_service_name,
                                           image = image,
                                           deployment_config = aks_config,
                                           deployment_target = aks_target)
aks_service.wait_for_deployment(show_output = True)
```

#### <a name="test-the-cloud-service"></a>测试云服务
Docker 映像支持 gRPC 和 TensorFlow 提供服务"预测"API。  使用示例客户端来调入要从模型中获取预测的 Docker 映像。  提供了示例客户端代码：
- [Python](https://github.com/Azure/aml-real-time-ai/blob/master/pythonlib/amlrealtimeai/client.py)
- [C#](https://github.com/Azure/aml-real-time-ai/blob/master/sample-clients/csharp)

如果你想要使用 TensorFlow 提供服务，则可以[下载示例客户端](https://www.tensorflow.org/serving/setup)。

```python
# Using the grpc client in Azure ML Accelerated Models SDK package
from azureml.accel.client import PredictionClient

address = aks_service.scoring_uri
ssl_enabled = address.startswith("https")
address = address[address.find('/')+2:].strip('/')
port = 443 if ssl_enabled else 80

# Initialize AzureML Accelerated Models client
client = PredictionClient(address=address,
                          port=port,
                          use_ssl=ssl_enabled,
                          service_name=aks_service.name)
```

由于此分类器定型[ImageNet](http://www.image-net.org/)数据集，将类映射到用户可读的标签。

```python
import requests
classes_entries = requests.get("https://raw.githubusercontent.com/Lasagne/Recipes/master/examples/resnet50/imagenet_classes.txt").text.splitlines()

# Score image with input and output tensor names
results = client.score_file(path="./snowleopardgaze.jpg", 
                             input_name=input_tensors, 
                             outputs=output_tensors)

# map results [class_id] => [confidence]
results = enumerate(results)
# sort results by confidence
sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
# print top 5 results
for top in sorted_results[:5]:
    print(classes_entries[top[0]], 'confidence:', top[1])
```

### <a name="clean-up-the-service"></a>清理服务
删除 web 服务、 图像和模型 （必须按此顺序完成由于没有依赖项）。

```python
aks_service.delete()
aks_target.delete()
image.delete()
registered_model.delete()
converted_model.delete()
```

## <a name="deploy-to-a-local-edge-server"></a>将部署到本地边缘服务器

所有[Azure 数据框边缘设备](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview
)FPGA 包含用于运行模型。  只有一个模型可以一次运行在 FPGA。  若要运行不同的模型，只需部署一个新的容器。 可以在找到说明和示例代码[此 Azure 示例](https://github.com/Azure-Samples/aml-hardware-accelerated-models)。

## <a name="secure-fpga-web-services"></a>保护 FPGA Web 服务

有关如何保护 FPGA Web 服务的信息，请参阅[保护 Web 服务](how-to-secure-web-service.md)文档。

## <a name="pbs-family-vms"></a>PBS 系列 Vm

PBS 系列 Azure Vm 包含 Intel Arria 10 Fpga。  它将显示为"标准 PBS 系列 Vcpu"时检查 Azure 配额分配。  PB6 VM 具有六个 Vcpu，且一个 FPGA，并且它将自动预配的 Azure 机器学习模型部署到 FPGA 的一部分。  仅用于 Azure ML 中，并且它不能运行任意轨道。  例如，您将不能 flash 与轨道 FPGA，以执行加密、 编码等。 