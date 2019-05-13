---
title: 现场可编程门阵列 (FPGA) 有哪些？
titleSuffix: Azure Machine Learning service
description: 了解如何在 Azure 上借助 FPGA 加快模型和深度神经网络的速度。 本文简单介绍了现场可编程门阵列 (FPGA)，以及在将模型部署到 Azure FPGA 时，如何通过 Azure 机器学习服务来提供实时人工智能 (AI)。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: tedway
author: tedway
ms.reviewer: jmartens
ms.date: 04/24/2019
ms.custom: seodec18
ms.openlocfilehash: 648dc462477570a692eff588c558a18a3121e4e7
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65025251"
---
# <a name="what-are-field-programmable-gate-arrays-fpga"></a>现场可编程门阵列 (FPGA) 有哪些？

本文简单介绍了现场可编程门阵列 (FPGA)，以及在将模型部署到 Azure FPGA 时，如何通过 Azure 机器学习服务来提供实时人工智能 (AI)。

FPGA 包含一组可编程的逻辑块以及由可反复配置的互连组成的层次结构。 这种互连可以在生产之后以不同方式来配置块。 与其他芯片相比，FPGA 既有可编程性，又有很好的性能。

## <a name="fpgas-vs-cpu-gpu-and-asic"></a>FPGA 与CPU、GPU 和 ASIC 的对比

下面的示意图和表将 FPGA 与其他处理器进行了比较。

![Azure 机器学习服务 FPGA 比较示意图](./media/concept-accelerate-with-fpgas/azure-machine-learning-fpga-comparison.png)

|处理器||描述|
|---|:-------:|------|
|特定于应用程序的集成电路|ASIC|自定义电路（例如 Google 的 TensorFlow 处理器单元 (TPU)）的效率最高。 它们不能根据需求的变化重新配置。|
|现场可编程门阵列|FPGA|FPGA（例如 Azure 上提供的那些）提供接近于 ASIC 的性能。 它们也是灵活的，并且可以在一段时间后重新配置以实现新逻辑。|
|图形处理单元|GPU|进行 AI 计算时的常用选择。 GPU 提供并行处理功能，在进行图形渲染时速度快于 CPU。|
|中央处理单元|CPU|常规用途处理器，其性能无法胜任图形和视频处理。|

在 Azure 上的 Fpga 基于 Intel FPGA 设备，哪些数据科学家和开发人员用于加速实时 AI 计算。 这种启用了 FPGA 的体系结构提供了高性能、灵活性和可伸缩性，可以在 Azure 上使用。

可以通过 FPGA 确保实时推断请求的低延迟。 不需要使用异步请求（批处理）。 批处理可能导致延迟，因为需要处理更多的数据。 神经处理单元的实现不需要批处理;因此滞后时间可能很多时候更低，相比 CPU 和 GPU 处理器。

### <a name="reconfigurable-power"></a>可反复配置的功能
可以针对不同类型的机器学习模型反复配置 FPGA。 利用这种灵活性，可以更轻松地根据最优化的数值精度和所用内存模型来加速应用程序。 由于 FPGA 可以反复配置，因此可以跟上快速变化的 AI 算法的要求。

### <a name="whats-supported-on-azure"></a>Azure 支持的操作
Microsoft Azure 是世界上在 FPGA 中的最大云投资。 Azure 支持的 Fpga:

+ 图像分类和识别方案
+ TensorFlow 部署
+ DNN：ResNet 50、ResNet 152、VGG-16、SSD-VGG 和 DenseNet-121
+ Intel FPGA 硬件 

使用这种支持 FPGA 的体系结构，受训的神经网络就可以快速运行并降低延迟。 Azure 可以跨 Fpga 横向扩展你的服务并行预先训练深层神经网络 (DNN)。 DNNs 可以预先训练，可以作为深度特征化器来传输学习内容，或者使用更新的权重进行优化。

### <a name="scenarios-and-applications"></a>方案和应用

与 Azure 机器学习集成 azure Fpga。 Microsoft 使用 FPGA 进行 DNN 评估、必应搜索排名、软件定义网络 (SDN) 加速，以便将 CPU 解放出来完成其他任务，同时降低延迟。

以下方案使用 Fpga:
+ [自动化光学检查系统](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)

+ [地表覆盖制图](https://blogs.technet.microsoft.com/machinelearning/2018/05/29/how-to-use-fpgas-for-deep-learning-inference-to-perform-land-cover-mapping-on-terabytes-of-aerial-images/)

## <a name="deploy-to-fpgas-on-azure"></a>部署到 Azure 中的 FPGA

若要在 Azure 中创建图像识别服务，可以使用受支持的 DNN 作为特征化器在 Azure FPGA 上进行部署：

1. 使用[适用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 创建服务定义。 服务定义是一个文件，用于根据 TensorFlow 来描述图形（输入、特征化器、分类器）的管道。 部署命令会自动将定义和图压缩到 ZIP 文件中，然后将 ZIP 文件上传到 Azure Blob 存储。 已部署 DNN FPGA 上运行。

1. 将 SDK 与 Azure Blob 存储中的 ZIP 文件配合使用来注册模型。

1. 使用 SDK 通过已注册的模型来部署服务。

若要开始将经过训练的 DNN 模型部署到 Azure 云中的 FPGA，请参阅[将模型部署为 FPGA 上的 Web 服务](how-to-deploy-fpga-web-service.md)。


## <a name="next-steps"></a>后续步骤

请访问以下视频和博客：

+ [“超大规模”硬件：ML at scale on top of Azure + FPGA :Build 2018 (video)](https://www.youtube.com/watch?v=BMgQAHIx2eY)（超大规模的硬件：基于 Azure + FPGA 的大规模机器学习：Build 2018（视频））

+ [Inside the Microsoft FPGA-based configurable cloud (video)](https://channel9.msdn.com/Events/Build/2017/B8063)（深入了解基于 Microsoft FPGA 的可配置云（视频））

+ [用于实时 AI 的 Project Brainwave：项目主页](https://www.microsoft.com/research/project/project-brainwave/)
