---
title: 版本新增内容
titleSuffix: Azure Machine Learning service
description: 了解 Azure 机器学习服务的最新更新以及机器学习和数据准备 Python SDK。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: larryfr
author: Blackmist
ms.date: 05/06/2019
ms.custom: seodec18
ms.openlocfilehash: 186ecf055cdf7b10690fdb4cbc574778a841b85a
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464808"
---
# <a name="azure-machine-learning-service-release-notes"></a>Azure 机器学习服务发行说明

本文介绍 Azure 机器学习服务版本。  有关每个 SDK 的完整说明，请访问以下内容的参考文档：
+ 在 Azure 机器学习[**主要适用于 Python SDK**](https://aka.ms/aml-sdk)
+ Azure 机器学习[数据准备 SDK](https://aka.ms/data-prep-sdk)

请参阅[已知问题列表](resource-known-issues.md)了解已知 bug 和解决方法。

## <a name="2019-05-06"></a>2019-05-06

### <a name="azure-portal"></a>Azure 门户

在 Azure 门户中，您现在可以：
+ 创建和运行自动化的机器学习试验 
+ 创建用于试用 Jupyter 笔记本示例的笔记本 VM 或你自己。
+ 在机器学习服务工作区，其中包括自动机器学习、 直观的界面和托管 Notebook Vm 中的全新创作部分 （预览版）
    + 自动创建使用自动机器学习模型 
    + 使用拖放可视化界面，运行试验
    + 创建笔记本 VM 来浏览数据、 创建模型，以及部署服务。
+ 实时图表和更新运行的报表中的指标和运行详细信息页
+ 更新后的文件查看器日志、 输出和运行详细信息页中的快照。
+ 在试验选项卡中，创建新的和改进的报表体验。 
+ 添加了的能够从 Azure 机器学习服务工作区的概述页中下载的 config.json 文件。
+ 支持从 Azure Databricks 工作区创建机器学习服务工作区 

## <a name="2019-04-26"></a>2019-04-26

### <a name="azure-machine-learning-sdk-for-python-v1033"></a>Azure Machine Learning SDK for Python v1.0.33
+ **新功能**
  + _Workspace.create_方法现在接受的 CPU 和 GPU 群集的默认群集配置。
  + 如果工作区创建失败，因为已清除依赖的资源。
  + 默认 Azure 容器注册表 SKU 切换到基本。
  + 在需要运行或映像创建时，将延迟，创建 azure 容器注册表。
  + 对训练运行环境的支持。

### <a name="notebook-virtual-machine"></a>Notebook 虚拟机 

用作安全、 面向企业的宿主环境 Notebook VM 在其中可以编程机器学习试验、 部署模型作为 web 终结点和执行所有其他操作支持的 Azure 机器学习 SDK 使用 Python 的 Jupyter 笔记本。 它提供了几个功能：
+ [快速启动 VM 的预配置笔记本](quickstart-run-cloud-notebook.md) 具有最新版本的 Azure 机器学习 SDK 和相关的程序包。
+ 通过经验证的技术，例如 HTTPS，Azure Active Directory 身份验证和授权保护的访问。
+ 笔记本和 blob 存储帐户的 Azure 机器学习工作区中的代码的可靠的云存储。 您可以安全地删除你的 notebook VM，而不会丢失所做的工作。
+ 预安装了示例笔记本进行探索和试验使用 Azure 机器学习服务功能。
+ Azure Vm、 任何 VM 类型、 任何包、 任何驱动程序的完全自定义功能。 

## <a name="2019-04-26"></a>2019-04-26

### <a name="azure-machine-learning-sdk-for-python-v1033-released"></a>用于 Python v1.0.33 发布的 azure 机器学习 SDK。

+ Azure ML 硬件加速模型上[Fpga](concept-accelerate-with-fpgas.md)已正式发布。
  + 现在可以[使用 azureml 加速模型包](how-to-deploy-fpga-web-service.md)到：
    + 训练的受支持的深度神经网络 （ResNet 50、 ResNet 152、 DenseNet 121、 VGG-16 和 SSD VGG） 的权重
    + 使用传输学习使用受支持的 DNN
    + 将模型注册模型管理服务和容器化模型
    + 将模型部署到 Azure VM 中使用 Azure Kubernetes 服务 (AKS) 群集中 FPGA
  + 部署到的容器[Azure 数据框边缘](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview)服务器设备
  + 评分与 gRPC 终结点与此数据[示例](https://github.com/Azure-Samples/aml-hardware-accelerated-models)

### <a name="automated-machine-learning"></a>自动化机器学习

+ 扫描结合使用以启用动态添加 featurizers 进行性能优化的功能。 新 featurizers： 工作的证据、 目标编码、 文本目标编码、 分类距离的权重的嵌入内容
+ 智能 CV 处理训练/有效拆分内自动机器学习
+ 几个内存优化的更改和运行时性能改进
+ 模型说明中的性能改进
+ ONNX 模型转换为本地运行的
+ 添加了的 Subsampling 支持
+ 智能停止时未定义任何退出条件
+ 堆积的集合

+ 时间序列预测
  + 新预测预测的函数   
  + 现在可以使用滚动域交叉验证，时序数据
  + 添加用于配置系列滞后的时间的新功能 
  + 添加以支持滚动窗口聚合功能的新功能
  + 新的假日检测和特征化器中定义国家/地区代码时尝试设置

+ Azure Databricks
  + 启用时序预测和建模 explainabilty/interpretability 功能
  + 现在，你可以取消和恢复 （继续） 自动化机器学习试验
  + 添加了对多核处理支持

### <a name="mlops"></a>MLOps
+ **本地部署和调试进行评分的容器**<br/> 现在可以部署本地机器学习模型和评分文件和依赖关系，以确保它们按预期的行为上的快速循环访问。

+ **Introduced InferenceConfig & Model.deploy()**<br/> 模型部署现在支持与入口脚本，RunConfig 相同指定源文件夹。  此外，模型部署已简化为单个命令。

+ **Git 引用跟踪**<br/> 客户一直请求推出基本 Git 集成功能的一些时间，因为它可帮助维护的端到端审核线索。 我们在 Azure 机器学习中的 Git 相关元数据 （存储库、 提交、 干净状态） 的主要实体上实施了跟踪。 将通过 SDK 和 CLI 自动收集此信息。

+ **模型的分析和验证服务**<br/> 客户经常抱怨的难度以正确的大小与他们推理的服务关联的计算。 与我们分析服务的模型，客户可以提供示例输入，我们将分析跨 16 不同 CPU / 内存配置以确定最佳的调整大小以进行部署。

+ **推理使基础映像**<br/> 另一个常见的投诉时将从试验移到推理重新共享依赖项的困难。 使用我们新的基本映像，用于共享功能，现在可以重用您试验基础映像、 依赖项和所有推理。 这应提高部署速度并减少从内部的无缝连接到外部循环。

+ **改进了的 Swagger 架构生成体验**<br/> 我们以前的 swagger 生成的方法是容易出错且无法自动执行。 我们有一种从通过修饰器的任何 Python 函数生成 swagger 架构的新行中方法。 我们已进行开源此代码和我们的架构生成协议不会耦合到 Azure 机器学习平台。

+ **Azure 机器学习 CLI 已正式推出 (GA)**<br/> 现在可以使用单个 CLI 命令部署模型。 我们有了常见的客户反馈，没有人会从 Jupyter notebook 机器学习模型部署。 [ **CLI 参考文档**](https://aka.ms/azmlcli)已更新。


## <a name="2019-04-22"></a>2019-04-22

用于 Python v1.0.30 发布的 azure 机器学习 SDK。

[ `PipelineEndpoint` ](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline_endpoint.pipelineendpoint?view=azure-ml-py)已引入同时保持相同的终结点添加已发布管道的新版本。

## <a name="2019-04-17"></a>2019-04-17

### <a name="azure-machine-learning-data-prep-sdk-v112"></a>Azure 机器学习数据准备 SDK v1.1.2

请注意:数据准备 Python SDK 将无法再安装`numpy`和`pandas`包。 请参阅[更新的安装说明](https://aka.ms/aml-data-prep-installation)。

+ **新功能**
  + 现在可以使用透视转换。
    + 操作方法指南：[Pivot notebook](https://aka.ms/aml-data-prep-pivot-nb)
  + 现在可以本机函数中使用正则表达式。
    + 示例：
      + `dflow.filter(dprep.RegEx('pattern').is_match(dflow['column_name']))`
      + `dflow.assert_value('column_name', dprep.RegEx('pattern').is_match(dprep.value))`
  + 现在，您可以使用`to_upper` 并`to_lower` 表达式语言中的函数。
  + 现在可以看到的数据配置文件中的每列的唯一值的数量。
  + 对于某些常用的读取器步骤，您现在可以传递中`infer_column_types`参数。 如果设置为`True`，数据准备会尝试检测并自动将列类型的转换。
    + `inference_arguments` 现已弃用。
  + 现在，可以调用`Dataflow.shape`。

+ **Bug 修复和改进**
  + `keep_columns` 现在接受一个附加的可选参数`validate_column_exists`，如果该检查的结果`keep_columns`将包含的任何列。
  + （这从文件中读取） 的所有读取器步骤现在接受其他可选自变量`verify_exists`。
  + 读取 pandas 数据帧和获取数据配置文件的改进了的性能。
  + 修复了 bug，切片数据流中的单步失败，单个索引。

## <a name="2019-04-15"></a>2019-04-15

### <a name="azure-portal"></a>Azure 门户
  + 现在可以重新提交现有的远程计算群集上运行的现有脚本。 
  + 在管道选项卡上，现在可以使用新参数运行已发布的管道。 
  + 运行详细信息现在支持新的快照文件查看器。 提交特定运行时，您可以查看的目录的快照。 此外可以下载已提交启动运行 notebook。
  + 现在可以取消父中的运行，从 Azure 门户。

## <a name="2019-04-08"></a>2019-04-08

### <a name="azure-machine-learning-sdk-for-python-v1023"></a>Azure Machine Learning SDK for Python v1.0.23

+ **新功能**
  + Azure 机器学习 SDK 现在支持 Python 3.7。
  + Azure 机器学习 DNN 房产现在提供内置的多版本支持。 例如， `TensorFlow` 估算器现在接受`framework_version`参数，并且用户可以指定"1.10"或"1.12"版本。 对于当前的 SDK 版本支持的版本的列表，请调用`get_supported_versions()`需的框架类 (例如， `TensorFlow.get_supported_versions()`)。
  有关最新 SDK 版本支持的版本的列表，请参阅[DNN 估算器文档](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn?view=azure-ml-py)。

### <a name="azure-machine-learning-data-prep-sdk-v111"></a>Azure 机器学习数据准备 SDK v1.1.1

+ **新功能**
  + 您可以阅读使用 read_ * 转换的多个数据存储/数据路径/DataReference 源。
  + 您还可以对要创建一个新列的列执行以下操作： 部门、 floor、 模、 电源、 长度。
  + 数据准备现在是 Azure 机器学习诊断套件的一部分，并且将默认情况下记录诊断信息。
    + 若要关闭此功能，设置此环境变量为 true:DISABLE_DPREP_LOGGER

+ **Bug 修复和改进**
  + 常用的类和函数的改进的代码文档。
  + 修复了 bug auto_read_file 未能读取 Excel 文件中。
  + 添加了的选项覆盖在 read_pandas_dataframe 文件夹。
  + 添加了的对 Fedora 27/28 和 Ubuntu 1804 dotnetcore2 依赖项的安装，以及改进了的性能。
  + 改进了从 Azure Blob 中读取的性能。
  + 列类型检测现在支持长时间类型的列。
  + 修复了 bug 已被某些日期值显示为时间戳，而不是 Python datetime 对象。
  + 修复了 bug，其中一些类型计数已显示为双精度型值而不是整数。

  
## <a name="2019-03-25"></a>2019-03-25

### <a name="azure-machine-learning-sdk-for-python-v1021"></a>Azure Machine Learning SDK for Python v1.0.21

+ **新功能**
  + *Azureml.core.Run.create_children*方法允许多个子级的低延迟创建运行一次调用。

### <a name="azure-machine-learning-data-prep-sdk-v110"></a>Azure 机器学习数据准备 SDK v1.1.0

+ **重大更改**
  + 数据准备包的概念已被弃用并不再受支持。 而不是持久保存在一个包中的多个数据流，您可以单独保留数据流。
    + 操作方法指南：[打开和保存数据流的 notebook](https://aka.ms/aml-data-prep-open-save-dataflows-nb)

+ **新功能**
  + 数据准备现在可以识别的类型匹配的特定语义，并相应地拆分的列。 目前支持 STypes 包括： 电子邮件地址、 地理坐标 （纬度和经度）、 IPv4 和 IPv6 地址、 美国电话号码和美国邮政编码。
    + 操作方法指南：[语义类型 notebook](https://aka.ms/aml-data-prep-semantic-types-nb)
  + 数据准备现在支持从两个数值列生成结果的列执行以下操作： 减、 乘、 除、 和取模。
  + 您可以调用`verify_has_data()`上检查是否在数据流将生成的记录，如果执行的数据流。

+ **Bug 修复和改进**
  + 现在可以指定要在直方图中使用的数值列配置文件的箱数。
  + `read_pandas_dataframe`转换表现在要求具有字符串的数据帧或字节类型的列名称。
  + 修复了 bug`fill_nulls`转换，其中的值未正确填写如果列已丢失。

## <a name="2019-03-11"></a>2019-03-11

### <a name="azure-machine-learning-sdk-for-python-v1018"></a>Azure Machine Learning SDK for Python v1.0.18

 + **更改**
   + Azureml tensorboard 程序包将取代 azureml contrib tensorboard。
   + 此版本中，您可以设置用户帐户 (amlcompute) 在托管的计算群集上创建它时。 这可以通过将这些属性传递中预配配置。 您可以找到更多详细信息中的[SDK 参考文档](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py#provisioning-configuration-vm-size-----vm-priority--dedicated---min-nodes-0--max-nodes-none--idle-seconds-before-scaledown-none--admin-username-none--admin-user-password-none--admin-user-ssh-key-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--tags-none--description-none-)。

### <a name="azure-machine-learning-data-prep-sdk-v1017"></a>Azure 机器学习数据准备 SDK v1.0.17

+ **新功能**
  + 现在支持添加两个数值列来生成结果的列使用表达式语言。

+ **Bug 修复和改进**
  + 改进的文档和参数检查 random_split。
  
## <a name="2019-02-27"></a>2019-02-27

### <a name="azure-machine-learning-data-prep-sdk-v1016"></a>Azure 机器学习数据准备 SDK v1.0.16

+ **Bug 修复**
  + 修复了导致的身份验证问题的服务主体的 API 更改。

## <a name="2019-02-25"></a>2019-02-25

### <a name="azure-machine-learning-sdk-for-python-v1017"></a>Azure Machine Learning SDK for Python v1.0.17

+ **新功能**

  + Azure 机器学习现在为常用的 DNN 框架，链接器提供一流的支持。 使用[ `Chainer` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py)类用户可以轻松地训练和部署链接器模型。
    + 了解如何[使用 ChainerMN 运行分布式的培训](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/distributed-chainer/distributed-chainer.ipynb)
    + 了解如何[运行链接器使用 HyperDrive 与超参数优化](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-chainer/train-hyperparameter-tune-deploy-with-chainer.ipynb)
  + Azure 机器学习管道添加管道运行基于数据存储的修改可以触发。 管道[计划 notebook](https://aka.ms/pl-schedule)会更新，以展示此功能。

+ **Bug 修复和改进**
  + 我们添加了支持 Azure 机器学习管道为 source_directory_data_store 属性设置为的所需的数据存储 （如 blob 存储） 上[RunConfigurations](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py)提供给[PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?view=azure-ml-py)。 默认情况下步骤作为后备数据存储，可能会遇到限制问题，同时执行大量的步骤时使用 Azure 文件存储。

### <a name="azure-portal"></a>Azure 门户

+ **新功能**
  + 新的拖放功能表报表编辑器的体验。 用户可以将列从井中到表的预览将显示其中的表区域。 可以重新排列列。
  + 新的日志文件查看器
  + 尝试运行、 计算、 模型、 图像和活动选项卡中的部署的链接

### <a name="azure-machine-learning-data-prep-sdk-v1015"></a>Azure 机器学习数据准备 SDK v1.0.15

+ **新功能**
  + 数据准备现在支持从数据流写入文件流。 此外提供的功能来处理要创建新的文件名称的文件流名称。
    + 操作方法指南：[使用与文件流 notebook](https://aka.ms/aml-data-prep-file-stream-nb)

+ **Bug 修复和改进**
  + 改进了的 t-Digest 对大型数据集的性能。
  + 数据准备现在支持从数据路径的读取数据。
  + 一种热编码现在适用于布尔值和数字列。
  + 其他杂项的 bug 修复。

## <a name="2019-02-11"></a>2019-02-11

### <a name="azure-machine-learning-sdk-for-python-v1015"></a>适用于 Python 的 Azure 机器学习 SDK v1.0.15

+ **新功能**
  + Azure 机器学习管道添加 AzureBatchStep ([notebook](https://aka.ms/pl-azbatch))，HyperDriveStep (notebook) 和基于时间的计划功能 ([notebook](https://aka.ms/pl-schedule))。
  +  DataTranferStep 已更新，可以与 Azure SQL Server 和 Azure database for PostgreSQL ([notebook](https://aka.ms/pl-data-trans)) 配合使用。

+ **更改**
  + 弃用了 `PublishedPipeline.get_published_pipeline` 而改用 `PublishedPipeline.get`。
  + 弃用了 `Schedule.get_schedule` 而改用 `Schedule.get`。

### <a name="azure-machine-learning-data-prep-sdk-v1012"></a>Azure 机器学习数据准备 SDK v1.0.12

+ **新功能**
  + 数据准备现在支持使用数据存储从 Azure SQL 数据库读取数据。
 
+ **更改**
  + 改进了对大型数据的特定操作的内存性能。
  + `read_pandas_dataframe()` 现在要求指定 `temp_folder`。
  + `ColumnProfile` 的 `name` 属性已弃用 - 请改用 `column_name`。

## <a name="2019-01-28"></a>2019-01-28

### <a name="azure-machine-learning-sdk-for-python-v1010"></a>适用于 Python 的 Azure 机器学习 SDK v1.0.10

+ **更改**： 
  + Azure 机器学习 SDK 不再将 azure-cli 包作为依赖项。 具体而言，azure-cli-core 和 azure-cli-profile 已从 azureml-core 中删除。 以下是对用户有所影响的更改：
    + 如果您是执行"az login"，然后使用 azureml sdk，SDK 将在一次执行浏览器或设备代码日志。 它不会使用“az login”所创建的任何凭据状态。
    + 对于 Azure CLI 身份验证（如使用“az login”），请使用 azureml.core.authentication.AzureCliAuthentication 类。 对于 Azure CLI 身份验证，请在安装了 azureml-sdk 的 Python 环境中执行 pip install azure-cli。
    + 如果在使用服务主体执行“az login”以实现自动化，则建议使用 azureml.core.authentication.ServicePrincipalAuthentication 类，因为 azureml-sdk 不会使用 azure CLI 所创建的凭据状态。 

+ **Bug 修复**：此版本主要包含次要 Bug 修复

### <a name="azure-machine-learning-data-prep-sdk-v108"></a>Azure 机器学习数据准备 SDK v1.0.8

+ **Bug 修复**
  + 改进了获取数据配置文件的性能。
  + 修复了与错误报告相关的次要 bug。
  
### <a name="azure-portal-new-features"></a>Azure 门户：新功能
+ 报表的新拖放图表体验。 用户可以将列或属性从井中拖动到图表区域，系统会在其中基于数据类型自动选择适合于用户的图表类型。 用户可以将图表类型更改为其他适用类型或添加其他属性。

    支持的图表类型：
    - 折线图
    - 直方图
    - 堆积条形图
    - 盒须图
    - 散点图
    - 气泡图
+ 门户现在为试验动态生成报表。 当用户提交试验运行时，报表将自动生成，其中包含记录的指标和图形，以便跨不同运行进行比较。 

## <a name="2019-01-14"></a>2019-01-14

### <a name="azure-machine-learning-sdk-for-python-v108"></a>适用于 Python 的 Azure 机器学习 SDK v1.0.8

+ **Bug 修复**：此版本主要包含次要 Bug 修复

### <a name="azure-machine-learning-data-prep-sdk-v107"></a>Azure 机器学习数据准备 SDK v1.0.7

+ **新功能**
  + 数据存储改进（记录在[数据存储区操作指南](https://aka.ms/aml-data-prep-datastore-nb)中）
    + 添加了在扩展中读取和写入 Azure 文件共享和 ADLS 数据存储的功能。
    + 使用数据存储时，数据准备现在支持使用服务主体身份验证而不是交互式身份验证。
    + 添加了的对 wasb 和 wasbs url 的支持。

## <a name="2019-01-09"></a>2019-01-09

### <a name="azure-machine-learning-data-prep-sdk-v106"></a>Azure 机器学习数据准备 SDK v1.0.6

+ **Bug 修复**
  + 修复了从 Spark 上的公共可读 Azure Blob 容器读取时出现的 bug

## <a name="2018-12-20"></a>2018-12-20 

### <a name="azure-machine-learning-sdk-for-python-v106"></a>适用于 Python 的 Azure 机器学习 SDK v1.0.6
+ **Bug 修复**：此版本主要包含次要 Bug 修复

### <a name="azure-machine-learning-data-prep-sdk-v104"></a>Azure 机器学习数据准备 SDK v1.0.4

+ **新功能**
  + 现在可以通过 `to_bool` 函数将不匹配的值转换为“错误”值。 这是针对 `to_bool` 和 `set_column_types` 的新的默认不匹配行为，而旧的默认行为是将不匹配的值转换为 False。
  + 调用 `to_pandas_dataframe` 时，可以通过新的选项将数字列中的 null/缺失值解释为 NaN。
  + 增加了检查某些表达式的返回类型的功能，目的是确保类型一致性和快速失败。
  + 现在可以调用 `parse_json`，以便将列中的值作为 JSON 对象进行分析，并将其扩张到多个列中。

+ **Bug 修复**
  + 修复了一个 Bug，该 Bug 在 Python 3.5.2 中导致 `set_column_types` 崩溃。
  + 修复了一个 Bug，在使用 AML 映像连接到数据存储时，该 Bug 会崩溃。

+ **更新**
  * 有关入门教程、案例研究和操作方法指南的[示例笔记本](https://aka.ms/aml-data-prep-notebooks)。

## <a name="2018-12-04-general-availability"></a>2018-12-04：正式版

Azure 机器学习服务现已公开发布。

### <a name="azure-machine-learning-compute"></a>Azure 机器学习计算
在此版本中，我们将宣告一个通过 [Azure 机器学习计算](how-to-set-up-training-targets.md#amlcompute)获得的全新托管计算体验。 此计算目标替换适用于 Azure 机器学习的 Azure Batch AI 计算。 

此计算目标：
+ 用于模型训练和批量推断
+ 是单节点到多节点计算
+ 为用户执行群集管理和作业计划
+ 默认进行自动缩放
+ 为 CPU 和 GPU 资源提供支持 
+ 允许使用低优先级 VM 来减少成本

可以通过 Python、Azure 门户或 CLI 来创建 Azure 机器学习计算。 它必须在工作区的区域中创建，不能附加到任何其他工作区。 此计算目标使用适用于运行的 Docker 容器，并将依赖项打包，以便跨所有节点复制同一环境。

> [!Warning]
> 建议创建一个新工作区来使用 Azure 机器学习计算。 尝试从现有工作区创建 Azure 机器学习计算的用户可能会看到一个错误，但这种可能性很小。 工作区中的现有计算可以继续使用，不受影响。

### <a name="azure-machine-learning-sdk-for-python-v102"></a>适用于 Python 的 Azure 机器学习 SDK v1.0.2
+ **重大更改**
  + 此版本不再支持从 Azure 机器学习创建 VM。 仍可以附加现有的云 VM 或远程本地服务器。 
  + 我们还将删除对 BatchAI 的支持，所有这些现在由 Azure 机器学习计算提供支持。

+ **New**
  + 适用于机器学习管道：
    + [EstimatorStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimator_step.estimatorstep?view=azure-ml-py)
    + [HyperDriveStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.hyper_drive_step.hyperdrivestep?view=azure-ml-py)
    + [MpiStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.mpi_step.mpistep?view=azure-ml-py)


+ **已更新**
  + 适用于机器学习管道：
    + [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?view=azure-ml-py) 现在接受 runconfig
    + [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?view=azure-ml-py) 现在可以通过 SQL 数据源进行复制
    + SDK 中的计划功能，用于创建和更新用于运行已发布管道的计划

<!--+ **Bugs fixed**-->

### <a name="azure-machine-learning-data-prep-sdk-v052"></a>Azure 机器学习数据准备 SDK v0.5.2
+ **重大更改** 
  * `SummaryFunction.N` 已重命名为 `SummaryFunction.Count`。
  
+ **Bug 修复**
  * 通过远程运行对数据存储进行读取和写入操作时，使用最新的 AML 运行令牌。 以前，如果在 Python 中更新 AML 运行令牌，则不会使用更新的 AML 运行令牌来更新数据准备运行时。
  * 其他更清晰的错误消息
  * 当 Spark 使用 `Kryo` 序列化时，to_spark_dataframe() 将不再崩溃
  * 值计数检查器现在可以显示超过 1000 个唯一值
  * 在原始数据流没有名称的情况下，随机拆分不再失败  

+ **详细信息**
  * [Azure 机器学习数据准备 SDK](https://aka.ms/data-prep-sdk)

### <a name="docs-and-notebooks"></a>文档和笔记本
+ ML 管道
  + 全新的和更新的笔记本，适用于管道、批量范围确定和样式传输示例的入门： https://aka.ms/aml-pipeline-notebooks
  + 了解如何[创建你的第一个管道](how-to-create-your-first-pipeline.md)
  + 了解如何[使用管道运行批量预测](how-to-run-batch-predictions.md)
+ Azure 机器学习计算目标
  + [示例笔记本](https://aka.ms/aml-notebooks)现已更新，可以使用新的托管计算。
  + [了解此计算](how-to-set-up-training-targets.md#amlcompute)

### <a name="azure-portal-new-features"></a>Azure 门户：新功能
+ 在门户中创建和管理 [Azure 机器学习计算](how-to-set-up-training-targets.md#amlcompute)类型。
+ 针对 Azure 机器学习计算监视配额使用情况和[请求配额](how-to-manage-quotas.md)。
+ 实时查看 Azure 机器学习计算群集状态。
+ 增加了虚拟网络支持，方便创建 Azure 机器学习计算和 Azure Kubernetes 服务。
+ 使用现有参数重新运行已发布的管道。
+ 新的[自动化机器学习图表](how-to-track-experiments.md#auto)适用于分类模型（带模型解释功能的提升、增益、校准功能重要性图表）和回归模型（带模型解释功能的残差和功能重要性图表）。 
+ 可以在 Azure 门户中查看管道




## <a name="2018-11-20"></a>2018-11-20

### <a name="azure-machine-learning-sdk-for-python-v0180"></a>适用于 Python 的 Azure 机器学习 SDK v0.1.80

+ **重大更改** 
  * azureml.train.widgets 命名空间已移至 azureml.widgets。
  * azureml.core.compute.AmlCompute 弃用了以下类 - azureml.core.compute.BatchAICompute 和 azureml.core.compute.DSVMCompute。 后续版本中将删除后一类。 AmlCompute 类现在有一个更简单的定义，只需要 vm_size 和 max_nodes，并在提交作业时自动将集群从 0 扩展到 max_nodes。 我们的[示例笔记本](https://github.com/Azure/MachineLearningNotebooks/tree/master/training)已更新该信息，并且会提供使用示例。 我们希望你喜欢这种简化以及即将发布的版本中更多令人兴奋的功能！

### <a name="azure-machine-learning-data-prep-sdk-v051"></a>Azure 机器学习数据准备 SDK v0.5.1 

阅读[参考文档](https://aka.ms/data-prep-sdk)，了解数据准备 SDK 的详细信息。
+ **新功能**
   * 创建了一个新的 DataPrep CLI 来执行 DataPrep 包并查看数据集或数据流的数据配置文件
   * 重新设计 SetColumnType API 以提高可用性
   * 将 smart_read_file 重命名为 auto_read_file
   * 数据配置文件中现包括倾斜和峰度
   * 可采用分层采样进行采样
   * 可从包含 CSV 文件的 zip 文件中读取
   * 可使用随机拆分逐行拆分数据集（例如，进入测试训练集）
   * 可以通过调用 `.dtypes` 从数据流或数据配置文件中获取所有列数据类型
   * 可以通过调用 `.row_count` 从数据流或数据配置文件中获取行计数

+ **Bug 修复**
   * 修复长度为双线转换 
   * 修复了任何添加列后的断言 
   * 修复了 FuzzyGrouping 的问题，在某些情况下它无法检测到组
   * 修复了排序功能以遵守多列排序顺序
   * 修复和/或表达式与 `pandas` 处理它们的方式类似
   * 修复了从 dbfs 路径读取的问题
   * 使错误消息更易于理解 
   * 使用 AML 令牌在远程计算目标上读取时不再失败
   * 现不再在 Linux DSVM 上失败
   * 当非字符串值在字符串谓词中时，现不再崩溃
   * 现在，当数据流正确失败时，将处理断言错误
   * 现在支持 Azure Databricks 上的 dbutils 装载存储位置

## <a name="2018-11-05"></a>2018-11-05

### <a name="azure-portal"></a>Azure 门户 
Azure 机器学习服务的 Azure 门户具有以下更新：
  * 针对已发布的管道提供了一个新的“管道”选项卡。
  * 增加了以下支持：将现有 HDInsight 群集附加为计算目标。

### <a name="azure-machine-learning-sdk-for-python-v0174"></a>适用于 Python 的 Azure 机器学习 SDK v0.1.74

+ **重大更改** 
  * *Workspace.compute_targets、datastores、experiments、images、models 和 webservices 现在是属性而不是方法。 例如，将 Workspace.compute_targets() 替换为 Workspace.compute_targets。
  * *Run.get_context* 弃用了 *Run.get_submitted_run*。 后续版本中将删除后一种方法。
  * *PipelineData* 类现在需要使用数据存储对象作为参数而非需要 datastore_name。 类似地，*Pipeline* 接受 default_datastore 而非 default_datastore_name。

+ **新功能**
  * Azure 机器学习管道[示例 Notebook](https://github.com/Azure/MachineLearningNotebooks/tree/master/pipeline/pipeline-mpi-batch-prediction.ipynb) 现在使用 MPI 步骤。
  * Jupyter Notebook 的 RunDetails 小组件已更新，将显示管道的可视化效果。

### <a name="azure-machine-learning-data-prep-sdk-v040"></a>Azure 机器学习数据准备 SDK v0.4.0 
 
+ **新功能**
  * 向数据配置文件中添加了类型计数 
  * 现在提供了值计数和直方图
  * 数据配置文件中提供了更多百分点值
  * 汇总中提供了中值
  * 现在支持 Python 3.7
  * 将包含数据存储的数据流保存到数据准备包时，数据存储信息将持久保存为数据准备包的一部分
  * 现在支持写入到数据存储 
        
+ **修复的 Bug**
  * Linux 上现在可以正确处理 64 位无符号整数溢出
  * 修复了 smart_read 中纯文本文件的不正确文本标签
  * 字符串列类型现在显示在指标视图中
  * 类型计数已修复，它现在显示映射到单个而非各个 FieldType 的 ValueKinds
  * 将路径提供为字符串时，Write_to_csv 不再失败
  * 使用“替换”时，将“查找”保留为空将不再失败 

## <a name="2018-10-12"></a>2018-10-12

### <a name="azure-machine-learning-sdk-for-python-v0168"></a>适用于 Python 的 Azure 机器学习 SDK v0.1.68

+ **新功能**
  * 创建新工作区时的多租户支持。

+ **修复的 Bug**
  * 部署 Web 服务时，不再需要固定 pynacl 库版本。

### <a name="azure-machine-learning-data-prep-sdk-v030"></a>Azure 机器学习数据准备 SDK v0.3.0

+ **新功能**
  * 添加了方法 transform_partition_with_file(script_path)，允许用户传入 Python 文件的路径进行执行

## <a name="2018-10-01"></a>2018-10-01

### <a name="azure-machine-learning-sdk-for-python-v0165"></a>适用于 Python 的 Azure 机器学习 SDK v0.1.65
[版本 0.1.65](https://pypi.org/project/azureml-sdk/0.1.65) 包括新增功能、更多文档、bug 修复和更多[示例笔记本](https://aka.ms/aml-notebooks)。

请参阅[已知问题列表](resource-known-issues.md)了解已知 bug 和解决方法。

+ **重大更改**
  * Workspace.experiments、Workspace.models、Workspace.compute_targets、Workspace.images，Workspace.web_services 将返回字典，而以前返回列表。 请参阅 [azureml.core.Workspace](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py) API 文档。

  * 自动化机器学习从主要指标中删除了规范化的均方误差。

+ **HyperDrive**
  * 针对 Bayesian 的多种 HyperDrive bug 修复；针对获取指标调用的性能改进。 
  * 从 1.9 升级的 TensorFlow 1.10 
  * 针对冷启动的 Docker 映像优化。 
  * 作业现在报告正确状态，即便退出时显示 0 之外的错误代码。 
  * SDK 中的 RunConfig 属性验证。 
  * HyperDrive 运行对象支持类似于“常规运行：不需要传递任何参数”的取消。 
  * 小组件改进，用于维护分布式运行和 HyperDrive 运行的下拉列表值的状态。 
  * TensorBoard 和针对参数服务器修复的其他日志文件支持。 
  * 服务端上的 Intel(R) MPI 支持。 
  * 对 BatchAI 中验证期间针对分布式运行修复的参数优化的 bug 修复。 
  * 上下文管理器现可标识主要实例。 

+ **Azure 门户体验**
  * 运行详细信息中支持 log_table() 和 log_row()。 
  * 为包含 1、2 或 3 数字列和可选分类列的表格和行自动创建关系图。

+ **自动化机器学习**
  * 改进了错误处理和文档 
  * 修复了运行属性检索性能问题。 
  * 修复了连续运行的问题。 
  * 修复了集成迭代问题。
  * 修复了 MAC 操作系统上的训练挂起 bug。
  * 减少对自定义验证场景中的宏平均 PR/ROC 曲线的采样。
  * 删除额外的索引逻辑。
  * 删除了 get_output API 中的筛选器。

+ **管道**
  * 添加了方法 Pipeline.publish()，用于直接发布管道，而无需首先执行运行。   
  * 添加了方法 PipelineRun.get_pipeline_runs()，用于获取从已发布管道生成的管道运行。

+ **Project Brainwave**
  * 更新了对 FPGA 上提供的新 AI 模型的支持。

### <a name="azure-machine-learning-data-prep-sdk-v020"></a>Azure 机器学习数据准备 SDK v0.2.0
[版本 0.2.0](https://pypi.org/project/azureml-dataprep/0.2.0/) 包括以下功能和 Bug 修复：

+ **新功能**
  * 独热编码支持
  * 分位转换支持
   
+ **Bug 修复：**
  * 适用于所有 Tornado 版本，无需降级 Tornado 版本
  * 值对所有值都有意义，而不仅仅是前三个值

## <a name="2018-09-public-preview-refresh"></a>2018-09（公共预览版刷新）

新的已刷新的 Azure 机器学习版本：了解有关此版本的详细信息： https://azure.microsoft.com/blog/what-s-new-in-azure-machine-learning-service/


## <a name="next-steps"></a>后续步骤

阅读 [Azure 机器学习服务](../service/overview-what-is-azure-ml.md)的概述。
