---
title: 体系结构和关键概念
titleSuffix: Azure Machine Learning service
description: 了解体系结构、 术语、 概念和组成 Azure 机器学习服务的工作流。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: Blackmist
ms.date: 04/15/2019
ms.custom: seodec18
ms.openlocfilehash: cb716e0d9f97d3ea2e9584a9fc3d7a6f57da9179
ms.sourcegitcommit: 1d257ad14ab837dd13145a6908bc0ed7af7f50a2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65502103"
---
# <a name="how-azure-machine-learning-service-works-architecture-and-concepts"></a>Azure 机器学习服务的工作原理：体系结构和概念

了解体系结构、 概念和 Azure 机器学习服务的工作流。 下图显示该服务的主要组件，以及使用该服务时的常规工作流：

[![Azure 机器学习服务体系结构和工作流](./media/concept-azure-machine-learning-architecture/workflow.png)](./media/concept-azure-machine-learning-architecture/workflow.png#lightbox)

## <a name="workflow"></a>工作流

机器学习工作流通常采用以下顺序：

1. 在 **Python** 中开发机器学习培训脚本。
1. 创建和配置**计算目标**。
1. **将脚本提交**到配置的计算目标以在该环境中运行。 在训练期间，脚本可以读取或写入**数据存储**。 并且执行记录在**工作区**中保存为**运行**，并在**试验**下分组。
1. **查询试验**了解当前和过去的运行中已记录的指标。 如果指标未指示所需结果，请循环回到步骤 1 并循环访问脚本。
1. 找到满意的运行后，在**模型注册表**中注册持久化模型。
1. 开发使用模型评分脚本和**部署模型**作为**web 服务**在 Azure 中，或设置为**IoT Edge 设备**。


> [!NOTE]
> 本文定义了 Azure 机器学习服务使用的术语和概念，但未定义 Azure 平台的术语和概念。 有关 Azure 平台术语的详细信息，请参阅 [Microsoft Azure 词汇表](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology)。

## <a name="workspace"></a>工作区

工作区是 Azure 机器学习服务的顶级资源。 它提供了一个集中的位置来处理使用 Azure 机器学习服务时创建的所有项目。

工作区保留可用于训练模型的计算目标的列表。 此外，它还保留训练运行的历史记录，包括日志、指标、输出和脚本快照。 使用此信息可以确定哪个训练运行产生最佳模型。

将模型注册到工作区。 使用已注册的模型和评分脚本以将模型部署到 Azure 容器实例时，Azure Kubernetes 服务，或作为基于 REST 的 HTTP 终结点现场可编程门列阵 (FPGA)。 还可以将映像作为模块部署到 Azure IoT Edge 设备。 在内部，创建 docker 映像以承载部署的映像。 如果需要可以指定自己的映像。

你可以创建多个工作区，并且每个工作区可由多个用户共享。 共享工作区时，可以通过将用户分配到以下角色来控制访问权限：

* 所有者
* 参与者
* 读取器

有关这些角色的详细信息，请参阅[访问管理对 Azure 机器学习工作区](how-to-assign-roles.md)一文。

创建新工作区时，它会自动创建工作区使用的几个 Azure 资源：

* [Azure 容器注册表](https://azure.microsoft.com/services/container-registry/)：注册在训练期间和部署模型时使用的 Docker 容器。
* [Azure 存储帐户](https://azure.microsoft.com/services/storage/)：用作工作区的默认数据存储。
* [Azure Application Insights](https://azure.microsoft.com/services/application-insights/)：存储有关模型的监视信息。
* [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)：存储计算目标使用的机密和工作区所需的其他敏感信息。

> [!NOTE]
> 除创建新版本以外，还可以使用现有的 Azure 服务。

下图演示了工作区的分类：

[![工作区分类](./media/concept-azure-machine-learning-architecture/azure-machine-learning-taxonomy.png)](./media/concept-azure-machine-learning-architecture/azure-machine-learning-taxonomy.png#lightbox)

## <a name="experiment"></a>试验

试验是指定的脚本中多个运行的分组。 它始终属于工作区。 当你提交运行时，需提供试验名称。 运行的信息存储在该试验下。 如果提交运行，并指定一个不存在的试验名称，则系统将使用新指定的名称自动创建一个新试验。

有关使用试验的示例，请参阅[快速入门：Azure 机器学习服务入门](quickstart-run-cloud-notebook.md)。

## <a name="model"></a>模型

简单地说，模型是一段接受输入并生成输出的代码。 创建机器学习模型将涉及选择算法、为其提供数据以及优化超参数。 培训是一个迭代过程，将生成经过培训的模型，它会封装模型在培训过程中学到的内容。

模型通过 Azure 机器学习中的运行生成。 还可以使用在 Azure 机器学习外部训练的模型。 可在 Azure 机器学习服务工作区中注册模型。

Azure 机器学习服务与框架无关。 创建模型时，可以使用任何常用的机器学习框架，例如 scikit-learn、 XGBoost、 PyTorch、 TensorFlow 和链接器。

为模型定型的示例，请参阅[教程：使用 Azure 机器学习服务训练图像分类模型](tutorial-train-models-with-aml.md)。

### <a name="model-registry"></a>模型注册表

模型注册表会跟踪 Azure 机器学习服务工作区中的所有模型。

模型按名称和版本标识。 每次使用与现有相同的名称注册模型时，注册表都会假定它是新版本。 该版本将递增并且新模型会以同一名称注册。

注册模型时，可以提供其他元数据标记，然后在搜索模型时使用这些标记。

不能删除正在使用的活动部署的模型。

有关注册模型的示例，请参阅[使用 Azure 机器学习训练映像分类模型](tutorial-train-models-with-aml.md)。

## <a name="run-configuration"></a>运行配置

运行配置是一组指令，用于定义如何在指定的计算目标中运行脚本。 该配置包括一组广泛的行为定义，例如，是使用现有 Python 环境还是使用根据规范构建的 Conda 环境。

运行配置可以保存到包含训练脚本的目录内的文件中，或构造为内存中对象以及用于提交运行。

有关示例运行配置，请参阅[选择并使用计算目标来训练模型](how-to-set-up-training-targets.md)。

## <a name="dataset"></a>数据集

Azure 机器学习数据集 （预览版） 使其更轻松地访问和使用你的数据。 数据集管理在各种情况下，例如模型训练数据和创建的管道。 使用 Azure 机器学习 SDK，可以访问基础存储、 探索和准备数据、 管理不同的数据集定义的生命周期和在培训和在生产环境中使用的数据集之间进行比较。

数据集提供用于处理中常用的格式，例如，使用的数据的方法`from_delimited_files()`或`to_pandas_dataframe()`。

有关详细信息，请参阅[创建和注册 Azure 机器学习数据集](how-to-create-register-datasets.md)。

使用数据集的示例，请参阅[示例笔记本](https://aka.ms/dataset-tutorial)。

## <a name="datastore"></a>数据存储

数据存储是通过 Azure 存储帐户进行的存储抽象。 数据存储可以使用 Azure blob 容器或 Azure 文件共享作为后端存储。 每个工作区都有默认数据存储，并且你可以注册其他数据存储。

使用 Python SDK API 或 Azure 机器学习 CLI 可从数据存储中存储和检索文件。

## <a name="compute-target"></a>计算目标

计算目标是用于运行训练脚本或托管服务部署的计算资源。 支持的计算目标为：

| 计算目标 | 培训 | 部署 |
| ---- |:----:|:----:|
| 本地计算机 | ✓ | &nbsp; |
| Azure 机器学习计算 | ✓ | &nbsp; |
| Azure 中的 Linux VM</br>（例如 Data Science Virtual Machine） | ✓ | &nbsp; |
| Azure Databricks | ✓ | &nbsp; |
| Azure Data Lake Analytics | ✓ | &nbsp; |
| Apache Spark for HDInsight | ✓ | &nbsp; |
| Azure 容器实例 | &nbsp; | ✓ |
| Azure Kubernetes 服务 | &nbsp; | ✓ |
| Azure IoT Edge | &nbsp; | ✓ |
| 现场可编程门阵列 (FPGA) | &nbsp; | ✓ |

计算目标附加到工作区。 本地计算机以外的计算目标由工作区的用户共享。

### <a name="managed-and-unmanaged-compute-targets"></a>托管和非托管的计算目标

* **托管**：由 Azure 机器学习服务创建和管理的计算目标。 这些计算目标已针对机器学习工作负荷进行优化。 截至 2018 年 12 月 4 日，Azure 机器学习计算是仅有的托管计算目标。 将来可能会添加其他托管计算目标。

    可以使用 Azure 门户、Azure 机器学习 SDK 或 Azure CLI 通过工作区直接创建机器学习计算实例。 所有其他计算目标必须在工作区外创建，然后再附加到工作区。

* **非托管**：不是由 Azure 机器学习服务管理的计算目标。 可能需要在 Azure 机器学习外部创建它们，然后在使用前将它们附加到工作区。 非托管计算目标可能需要执行额外的步骤来保持或提高机器学习工作负荷的性能。

有关为训练选择计算目标的信息，请参阅[选择并使用计算目标来训练模型](how-to-set-up-training-targets.md)。

有关为部署选择计算目标的信息，请参阅[使用 Azure 机器学习服务部署模型](how-to-deploy-and-where.md)。

## <a name="training-script"></a>培训脚本

若要定型模型，你可以指定包含培训脚本和关联文件的目录。 此外，还可指定一个试验名称，用于存储在训练期间收集的信息。 在训练期间，会将整个目录复制到训练环境（计算目标），并启动运行配置指定的脚本。 目录的快照同样存储在工作区中的试验下。

有关示例，请参阅[教程：使用 Azure 机器学习服务训练图像分类模型](tutorial-train-models-with-aml.md)。

## <a name="run"></a>运行

运行是包含以下信息的记录：

* 有关运行的元数据（时间戳、持续时间等）
* 脚本记录的指标
* 试验自动收集的或由你显式上传的输出文件
* 在运行之前包含脚本的目录的快照

提交脚本以训练模型时，会生成运行。 运行可以有零次或多次子级运行。 例如，顶级运行可以有两次子级运行，其中每个可以有其自己的子级运行。

有关查看由训练模型产生的运行次数的示例，请参阅[快速入门：Azure 机器学习服务入门](quickstart-run-cloud-notebook.md)。

## <a name="snapshot"></a>快照

提交运行时，Azure 机器学习会将包含该脚本的目录压缩为 zip 文件并将其发送到计算目标。 然后解压缩 zip 文件并运行脚本。 Azure 机器学习还将该 zip 文件存储为快照，作为运行记录的一部分。 有权限访问工作区的任何用户都可以浏览运行记录并下载快照。

## <a name="activity"></a>活动

活动表示长时间运行的操作。 以下操作是活动的示例：

* 创建或删除计算目标
* 在计算目标上运行脚本

活动可通过 SDK 或 Web UI 提供通知，使你能够轻松监视这些操作的进度。

## <a name="image"></a>映像

映像提供了一种可靠地部署模型的方法，以及使用该模型所需的所有组件。 映像包含以下项：

* 模型。
* 评分脚本或应用程序。 使用此脚本可将输入传递到模型，并返回模型的输出。
* 模型或评分脚本/应用程序所需的依赖项。 例如，你可能包括列出 Python 包依赖项的 Conda 环境文件。

Azure 机器学习可以创建两种类型的映像：

* **FPGA 映像**：部署到 Azure 中的现场可编程门阵列时使用。
* **Docker 映像**：部署到 FPGA 以外的计算目标时使用。 例如，部署到 Azure 容器实例和 Azure Kubernetes 服务时。

Azure 机器学习服务提供了一个基本映像，默认情况下使用。 你还可以提供自己的自定义映像。

有关创建映像的示例，请参阅[在 Azure 容器实例中部署映像分类模型](tutorial-deploy-models-with-aml.md)。

### <a name="image-registry"></a>映像注册表

映像注册表跟踪从模型创建的映像。 创建映像时，可以提供其他元数据标记。 元数据标记由映像注册表存储，并且可以进行查询以查找映像。

## <a name="deployment"></a>部署

部署是模型的为 web 服务，可以托管在云中或 IoT 模块集成的设备部署到您的实例化。

### <a name="web-service"></a>Web 服务

已部署的 Web 服务可以使用 Azure 容器实例、Azure Kubernetes 服务或 FPGA。 从模型、 脚本和关联的文件创建的服务。 这些被封装在一个映像，而提供的 web 服务的运行的时环境中。 映像具有负载均衡的 HTTP 终结点，可接收发送到 Web 服务的评分请求。

如果已选择启用此功能，Azure 可通过收集 Application Insight 遥测数据或模型遥测数据帮助监视 Web 服务部署。 遥测数据仅供你访问，并且存储在 Application Insights 和存储帐户实例中。

如果已启用自动缩放，Azure 将自动缩放部署。

有关将模型部署为 Web 服务的示例，请参阅[在 Azure 容器实例中部署映像分类模型](tutorial-deploy-models-with-aml.md)。

### <a name="iot-module"></a>IoT 模块

已部署 IoT 模块是一个 Docker 容器，包括模型和关联脚本或应用程序，以及任何其他依赖项。 在 edge 设备上使用 Azure IoT Edge 部署这些模块。

如果已启用监视，Azure 会从 Azure IoT Edge 模块内的模型中收集遥测数据。 遥测数据仅供你访问，并且存储在存储帐户实例中。

Azure IoT Edge 将确保模块正在运行并且监视托管它的设备。

## <a name="pipeline"></a>管道

使用机器学习管道可以创建和管理将各个机器学习阶段整合到一起的工作流。 例如，管道可以包括数据准备、模型训练、模型部署以及推断阶段。 每个阶段可以包含多个步骤，每个步骤都能够以无人参与方式在各种计算目标中运行。

有关机器学习管道与此服务的详细信息，请参阅[管道和 Azure 机器学习](concept-ml-pipelines.md)。

## <a name="logging"></a>日志记录

开发解决方案时，请在 Python 脚本中使用 Azure 机器学习 Python SDK 记录任意指标。 运行后，查询指标以确定运行是否生成了要部署的模型。

## <a name="next-steps"></a>后续步骤

若要开始使用 Azure 机器学习服务，请参阅：

* [什么是 Azure 机器学习服务？](overview-what-is-azure-ml.md)
* [创建 Azure 机器学习服务工作区](setup-create-workspace.md)
* [教程：训练模型](tutorial-train-models-with-aml.md)
* [使用资源管理器模板创建一个工作区](how-to-create-workspace-template.md)
