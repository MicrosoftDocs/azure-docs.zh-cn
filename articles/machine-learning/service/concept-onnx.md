---
title: 高性能、 跨平台使用 ONNX 推理
titleSuffix: Azure Machine Learning service
description: 了解有关 ONNX 和 ONNX 运行时对加速模型信息
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: prasantp
author: prasanthpul
ms.date: 04/24/2019
ms.custom: seodec18
ms.openlocfilehash: a8bc46011b00a0c63eddd2799ac1309b5754472e
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442421"
---
# <a name="onnx-and-azure-machine-learning-create-and-accelerate-ml-models"></a>ONNX 和 Azure 机器学习：创建并加速机器学习模型

了解如何使用[开放神经网络交换](https://onnx.ai)(ONNX) 可帮助优化机器学习模型。

优化机器学习的推理模型是很难，因为您需要调整的模型和推理库来充分利用的硬件功能。 问题变得非常困难，如果你想要获取不同的平台 (云/edge，CPU/GPU，等等) 上获得最佳性能，因为其中一个具有不同的功能和特征。 如果您有使用不同的框架，需要在多个平台上运行的模型会增加复杂性。 它是非常较长时间进行优化的框架和硬件的所有不同组合。 需要一个解决方案，可以在您首选的框架中一次训练和在云或边缘上任意位置运行。 这是 ONNX 传入的位置。

Microsoft 和合作伙伴社区创建 ONNX 为一个开放标准，用于表示机器学习模型。 从模型[许多框架](https://onnx.ai/supported-tools)包括 TensorFlow、 PyTorch、 Scikit-learn、 Keras、 Chainer、 MXNet、 和 MATLAB 可以要导出或转换为标准 ONNX 格式。 ONNX 格式模型后，它们可以在各种平台和设备上运行。

[ONNX 运行时](https://github.com/Microsoft/onnxruntime)是高效的推理引擎，ONNX 模型部署到生产环境。 它适用于云和边缘，并适用于 Linux 和 Windows，mac。 编写的C++，它还具有 C、 Python 和C#Api。 ONNX 运行时为所有 ONNX ML 规范都提供支持，还集成了如 TensorRT NVidia 的 Gpu 上的不同硬件上的加速器。

ONNX 运行时可在高缩放性 Microsoft 服务，例如必应、 Office 和认知服务。 性能提升取决于多种因素，但这些 Microsoft 服务看过__平均 2 倍的 CPU 上的性能增益__。 ONNX 运行时还用作数百个数以百万计的设备上的 Windows 机器学习的一部分。 使用 Azure 机器学习服务，可以使用运行时。 通过使用 ONNX 运行时，你可以从大量生产级优化，测试，并持续改进中受益。

[![ONNX 数据流关系图显示培训、 转换器和部署](media/concept-onnx/onnx.png)](./media/concept-onnx/onnx.png#lightbox)

## <a name="get-onnx-models"></a>获取 ONNX 模型

可通过以下几种方式获取 ONNX 模型：
+ Azure 机器学习服务中的新 ONNX 模型进行定型 （请参阅这篇文章底部的示例）
+ 将现有模型从另一种格式转换为 ONNX (请参阅[教程](https://github.com/onnx/tutorials)) 
+ 获取从预先训练的 ONNX 模型[ONNX 模型 Zoo](https://github.com/onnx/models) （请参阅这篇文章底部的示例）
+ 从 [Azure 自定义影像服务](https://docs.microsoft.com/azure/cognitive-services/Custom-Vision-Service/)生成自定义 ONNX 模型 

包括图像分类、 对象检测和文本处理的多个模型可以表示为 ONNX 模型。 但是某些模型可能无法成功转换。 如果遇到这种情况下，请在各自所用的转换器的 GitHub 中提出问题。 你可以继续使用现有的格式模型，直到解决该问题。

## <a name="deploy-onnx-models-in-azure"></a>在 Azure 中部署 ONNX 模型

使用 Azure 机器学习服务，可以部署、管理和监视 ONNX 模型。 使用标准[部署工作流](concept-model-management-and-deployment.md)和 ONNX 运行时，可以创建在云中托管的 REST 终结点。 请参阅示例 Jupyter notebook 来亲自尝试一下这篇文章的末尾。 

### <a name="install-and-use-onnx-runtime-with-python"></a>安装和使用 ONNX 运行时与 Python 配合使用

ONNX 运行时的 Python 包位于[PyPi.org](https://pypi.org) ([CPU](https://pypi.org/project/onnxruntime)， [GPU](https://pypi.org/project/onnxruntime-gpu))。 请阅读[系统要求](https://github.com/Microsoft/onnxruntime#system-requirements)之前安装。 

 若要安装 Python ONNX 运行时，使用以下命令之一： 
```python   
pip install onnxruntime       # CPU build
pip install onnxruntime-gpu   # GPU build
``` 

若要在 Python 脚本中调用 ONNX 运行时，请使用：    
```python   
import onnxruntime  
session = onnxruntime.InferenceSession("path to model") 
``` 

模型附带的文档通常会指示有关使用模型的输入和输出。 还可使用可视化工具（如 [Netron](https://github.com/lutzroeder/Netron)）查看模型。 ONNX 运行时还可以查询模型元数据、输入和输出：    
```python   
session.get_modelmeta() 
first_input_name = session.get_inputs()[0].name 
first_output_name = session.get_outputs()[0].name   
``` 

若要推理模型，请使用 `run`，并传入要返回的输出列表（如果需要所有输出，则保留为空）和输入值的映射。 结果是输出列表。  
```python   
results = session.run(["output1", "output2"], {"input1": indata1, "input2": indata2})   
results = session.run([], {"input1": indata1, "input2": indata2})   
``` 

有关完整的 Python API 参考，请参阅 [ONNX 运行时参考文档](https://aka.ms/onnxruntime-python)。    

## <a name="examples"></a>示例

有关创建和部署 ONNX 模型的示例笔记本，请参阅 [how-to-use-azureml/deployment/onnx](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx)。

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="more-info"></a>详细信息

详细了解 ONNX 或为项目做出贡献：
+ [ONNX 项目网站](https://onnx.ai)
+ [GitHub 上的 ONNX 代码](https://github.com/onnx/onnx)

详细了解 ONNX 运行时或者为项目做出贡献：
+ [ONNX 运行时 GitHub 存储库](https://github.com/Microsoft/onnxruntime)


