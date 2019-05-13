---
title: 为机器学习使用 Visual Studio Code
titleSuffix: Azure Machine Learning service
description: 了解如何安装 Azure Machine Learning for Visual Studio Code 并在 Azure 机器学习中创建试验。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: shwinne
author: swinner95
ms.date: 12/04/2018
ms.custom: seodec18
ms.openlocfilehash: 19873256f8253fff75cfd42df7b876106a9e98e5
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464791"
---
# <a name="get-started-with-azure-machine-learning-for-visual-studio-code"></a>Azure Machine Learning for Visual Studio Code 入门

在本文中，将了解如何使用适用于 Azure 机器学习的 Visual Studio Code 扩展训练和部署机器学习和深度学习模型通过在 Visual Studio Code 中使用 Azure 机器学习服务。

Azure 机器学习服务提供支持本地和远程计算目标上运行的试验。 对于每个试验，可跟踪多次运行，因为通常需要以迭代方式尝试不同的技术、超参数等。 Azure 机器学习可用于跟踪自定义指标和试验运行，以确保数据科学可再现性和可审核性。

此外可以将这些模型部署为测试和生产需求。

## <a name="prerequisites"></a>必备组件

+ 如果没有 Azure 订阅，请在开始之前创建一个免费帐户。 请尝试[Azure 机器学习服务的免费或付费版](https://aka.ms/AMLFree)。

+ 必须安装 Visual Studio Code。 Visual Studio Code 是轻型但功能强大的源代码编辑器，在桌面上运行。 它附带了对 Python 和其他编程语言的内置支持。 如果你尚未安装 Visual Studio Code[了解如何在](https://code.visualstudio.com/docs/setup/setup-overview)。

+ [安装 Python 3.5 或更高版本](https://www.anaconda.com/download/)。


## <a name="install-the-extension-for-azure-machine-learning-for-visual-studio-code"></a>安装 Visual Studio Code 的 Azure 机器学习的扩展

当你安装 Azure 机器学习扩展时，两个多个扩展插件将自动安装 （如果有 internet 访问权限）。 它们[Azure 帐户扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)并[Microsoft Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python)。

若要使用 Azure 机器学习，您需要将 Visual Studio Code 变成 Python 集成开发环境 (IDE)。 您需要使用的 Microsoft Python 扩展名[Visual Studio Code 中的 Python](https://code.visualstudio.com/docs/languages/python)。 Azure 机器学习扩展将自动安装此扩展。 扩展使 Visual Studio Code 的极好的 IDE，并遵循多种 Python 解释器的任何操作系统上。 Microsoft Python 扩展使用所有 Visual Studio Code 的强大功能来提供记忆式键入功能、 IntelliSense、 语法检查、 调试和单元测试。 扩展还允许您以 Python 环境，包括虚拟和 conda 环境之间轻松切换。 有关编辑、 运行和调试 Python 代码，请参阅[Python 你好 world 教程](https://code.visualstudio.com/docs/python/python-tutorial)。

若要安装 Azure 机器学习扩展：

1. 打开 Visual Studio Code。

1. 在 web 浏览器中，转到[Visual Studio Code 扩展 （预览版） 的 Azure 机器学习](https://aka.ms/vscodetoolsforai)。

1. 在此网页上，选择**安装**。 

1. 在扩展选项卡上选择**安装**。

1. 在 Visual Studio Code 中，打开扩展一个欢迎使用选项卡，（具有以下屏幕截图中红色边框） 的 Azure 符号添加到活动栏。

   ![在 Visual Studio Code 活动栏上的 azure 图标](./media/vscode-tools-for-ai/azure-activity-bar.png)

1. 在对话框中，选择**Sign In**并按照提示使用 Azure 进行身份验证。 
   
   Azure 帐户扩展，它随 Visual Studio Code 扩展在 Azure 机器学习一起安装，可帮助你使用 Azure 帐户进行身份验证。 有关命令的列表，请参阅有关页面[Azure 帐户扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)。

> [!Tip] 
> 请查看[IntelliCode 适用于 Visual Studio Code （预览版） 的扩展](https://go.microsoft.com/fwlink/?linkid=2006060)。 IntelliCode 提供一组的 AI 辅助功能在 Python 中，intellisense 如推断最相关 autocompletions 基于当前代码上下文。

## <a name="install-the-azure-machine-learning-sdk"></a>安装 Azure Machine Learning SDK

1. 请确保 Python 3.5 或更高版本安装，并识别的 Visual Studio Code。 如果立即安装，请重新启动 Visual Studio Code 和[选择 Python 解释器](https://code.visualstudio.com/docs/python/python-tutorial)。

1. 在集成终端窗口中，指定要使用的 Python 解释器。 或按 Enter 以使用默认 Python 解释器。

   ![选择解释器](./media/vscode-tools-for-ai/python.png)

1. 在窗口的右下角中，会显示通知，指示正在自动安装 Azure 机器学习 SDK。 新创建的 Python 环境为本地和私有的并且它使用 Azure 机器学习服务的 Visual Studio Code 先决条件。

   ![安装用于 Python 的 Azure 机器学习 SDK](./media/vscode-tools-for-ai/runtimedependencies.png)

## <a name="get-started-with-azure-machine-learning"></a>开始使用 Azure 机器学习

在开始训练和部署机器学习模型，在 Visual Studio Code 中的，需要创建之前[Azure 机器学习服务工作区](concept-azure-machine-learning-architecture.md#workspace)在云中。 此工作区将包含你的模型和资源。 

若要创建一个工作区并添加第一个试验：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

   [![创建工作区](./media/vscode-tools-for-ai/CreateaWorkspace.gif)](./media/vscode-tools-for-ai/CreateaWorkspace.gif#lightbox)


1. 右键单击你的 Azure 订阅，然后选择“创建工作区”。 此时将显示一个列表。 在示例动画图像、 订阅名称是**免费试用版**，并在工作区是**TeamWorkspace**。 

1. 从列表中选择资源组或创建一个新的使用命令面板中的向导。

1. 在该字段中，为新工作区键入一个唯一且明确的名称。 在示例图中，名为工作区**TeamWorkspace**。

1. 选择 enter 键以创建新的工作区。 它显示在树中，订阅名称的下方。

1. 右键单击**实验**节点，然后选择**创建实验**从上下文菜单。  这些试验使用 Azure 机器学习跟踪你的运行。

1. 在字段中，输入你的试验的名称。 在下面的示例的屏幕，名为试验**MNIST**。
 
1. 选择 enter 键以创建新的实验。 实验显示在树中，以下工作区名称。

1. 在工作区中，您可以右键单击要将其设置为的试验**Active**试验。 **Active**实验是你当前的试验。 Visual Studio Code 中的打开文件夹将链接到云环境中的此试验。 此文件夹应包含本地 Python 脚本。

现在将实验历史记录中存储关键指标。 同样，你训练的模型将自动上传到 Azure 机器学习并存储以及试验指标和日志。 

[![附加 Visual Studio Code 中的文件夹](./media/vscode-tools-for-ai/CreateAnExperiment.gif)](./media/vscode-tools-for-ai/CreateAnExperiment.gif#lightbox)


## <a name="create-and-manage-compute-targets"></a>创建和管理计算目标

使用 Visual Studio Code 的 Azure 机器学习，可以准备数据、 定型模型，和将其部署为本地和远程计算目标上。

该扩展 Azure 机器学习支持多个远程计算目标。 有关详细信息，请参阅的完整列表支持[Azure 机器学习的计算目标](how-to-set-up-training-targets.md)。

### <a name="create-compute-targets-for-azure-machine-learning-in-visual-studio-code"></a>在 Visual Studio Code 的 Azure 机器学习中创建计算目标

若要创建计算目标：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

2. 在树视图中，展开 Azure 订阅和 Azure 机器学习服务工作区。 在以下示例图中，订阅名称是**免费试用版**，并在工作区是**TeamWorkspace**。 

3. 在工作区节点下，右键单击“计算”节点，再选择“创建计算”。

4. 从列表中选择计算目标类型。 

5. 在命令面板中，选择虚拟机大小。

6. 在命令面板中，在字段中，输入计算目标的名称。 

7. 在 JSON 配置文件中打开新选项卡上，指定任何高级的属性。 可以指定属性，例如最大节点计数。

8. 完成后配置计算目标，在窗口的右下角，选择**提交**。

下面是如何创建 Azure 机器学习计算 (AMLCompute) 的示例：

[![在 Visual Studio Code 中创建 AML 计算](./media/vscode-tools-for-ai/CreateARemoteCompute.gif)](./media/vscode-tools-for-ai/CreateARemoteCompute.gif#lightbox)

#### <a name="the-run-configuration-file"></a>运行的配置文件

Visual Studio Code 扩展会自动在本地计算机上创建本地计算目标和运行的配置为本地和 docker 环境 您可以找到相关联的计算目标节点下的运行的配置文件。 

## <a name="train-and-tune-models"></a>定型和优化模型

使用用于 Visual Studio Code （预览版） Azure 机器学习快速循环访问代码、 单步执行和调试，并使用你的解决方案进行源代码管理。 

若要通过使用 Azure 机器学习在本地运行你的试验：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

1. 在树视图中，展开 Azure 订阅和 Azure 机器学习服务工作区。 

1. 在工作区节点中，展开**计算**节点并右击**运行配置**你想要使用的计算。 

1. 选择“运行试验”。

1. 从文件资源管理器，选择你想要运行的脚本。 

1. 选择**查看试验运行**以查看集成的 Azure 机器学习门户，以监视运行并查看已训练的模型。

下面是如何在本地运行试验的示例：

[![在本地运行试验](./media/vscode-tools-for-ai/RunExperimentLocally.gif)](./media/vscode-tools-for-ai/RunExperimentLocally.gif#lightbox)

### <a name="use-remote-computes-for-experiments-in-visual-studio-code"></a>Visual Studio Code 中的实验中使用远程计算资源

若要使用远程计算目标的培训，您需要创建运行的配置文件。 此文件不仅指示 Azure 机器学习在何处运行试验，还指示如何准备环境。

#### <a name="the-conda-dependencies-file"></a>conda 依赖项文件

默认情况下，创建一个新的 conda 环境和安装依赖关系的管理。 但是，必须指定依赖项并在其版本*aml_config/conda_dependencies.yml*文件。 

默认值的以下代码段*aml_config/conda_dependencies.yml*指定`tensorflow=1.12.0`。 如果未指定的依赖项版本，将使用的最新版本。 可以在配置文件中添加其他依赖项。

```yaml
# The dependencies defined in this file will be automatically provisioned for runs with userManagedDependencies=False.

name: project_environment
dependencies:
  # The python interpreter version.

  # Currently Azure ML only supports 3.5.2 and later.

- python=3.6.2
- tensorflow=1.12.0

- pip:
    # Required packages for AzureML execution, history, and data preparation.

  - --index-url https://azuremlsdktestpypi.azureedge.net/sdk-release/Preview/E7501C02541B433786111FE8E140CAA1
  - --extra-index-url https://pypi.python.org/simple
  - azureml-defaults

```

若要运行远程计算机上的使用 Azure 机器学习试验的计算目标：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

1. 在树视图中，展开 Azure 订阅和 Azure 机器学习服务工作区。 

1. 在编辑器窗口中，右键单击您的 Python 脚本，并选择**AML:在 Azure 中作为试验运行**。 

1. 在命令面板中，选择计算目标。 

1. 在命令面板中，在字段中，输入运行的配置名称。 

1. 编辑*conda_dependencies.yml*文件指定实验的运行时依赖项。 然后在窗口的右下角，选择**提交**。 

1. 选择**查看试验运行**以查看集成的 Azure 机器学习门户，以监视运行并查看已训练的模型。

下面是如何在远程计算目标上运行试验的示例：

[![远程目标上运行试验](./media/vscode-tools-for-ai/runningOnARemoteTarget.gif)](./media/vscode-tools-for-ai/runningOnARemoteTarget.gif#lightbox)


## <a name="deploy-and-manage-models"></a>部署和管理模型
在 Azure 机器学习，可以部署和管理机器学习模型在云中和在边缘。 

### <a name="register-your-model-to-azure-machine-learning-from-visual-studio-code"></a>从 Visual Studio Code 将模型注册到 Azure 机器学习

现在，已训练模型，您可以在工作区中注册它。 您可以跟踪并部署已注册的模型。

若要注册您的模型：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

1. 在树视图中，展开 Azure 订阅和 Azure 机器学习服务工作区。

1. 在工作区节点下，右键单击“模型”，再选择“注册模型”。

1. 在命令面板中，在字段中，输入模型名称。 

1. 从列表中，选择是否要上传**模型文件**（适用于单个模型） 或**模型文件夹**（适用于具有多个文件，如 TensorFlow 模型）。 

1. 选择你的文件夹或文件。

1. 配置完您的模型属性，在窗口的右下角，选择**提交**。 

下面是如何注册到 Azure 机器学习模型的示例：

[![将模型注册到 AML](./media/vscode-tools-for-ai/RegisteringAModel.gif)](./media/vscode-tools-for-ai/RegisteringAModel.gif#lightbox)


### <a name="deploy-your-service-from-visual-studio-code"></a>从 Visual Studio Code 部署服务

在 Visual Studio Code 中，可以部署到 web 服务：
+ Azure 容器实例 (ACI) 进行测试。
+ Azure Kubernetes 服务 (AKS) 用于生产。

您不必创建 ACI 容器要提前测试，因为动态创建 ACI 容器。 但是，您确实需要预先配置 AKS 群集。 有关详细信息，请参阅[部署模型与 Azure 机器学习服务](how-to-deploy-and-where.md)。

若要部署的 web 服务：

1. 在 Visual Studio Code 活动栏中，选择 Azure 图标。 此时会显示“Azure 机器学习”边栏。

1. 在树视图中，展开 Azure 订阅和 Azure 机器学习服务工作区。

1. 在工作区节点下，展开“模型”节点。

1. 右键单击你想要部署，并选择的模型**从已注册的模型部署服务**从上下文菜单。

1. 在命令面板中，选择你想要将部署到计算目标。 

1. 在命令面板中，在字段中，输入此服务的名称。  

1. 在命令面板中，选择以浏览和选择的脚本文件在键盘上 Enter 键。

1. 在命令面板中，选择以浏览和选择的 conda 依赖项文件在键盘上 Enter 键。

1. 配置完您的服务属性，在窗口的右下角，选择**提交**部署。 在服务属性文件中，可以指定本地 docker 文件或一个 schema.json 文件。

此时，Web 服务已部署。

下面是如何部署 web 服务的示例：

[![部署 web 服务](./media/vscode-tools-for-ai/CreatingAnImage.gif)](./media/vscode-tools-for-ai/CreatingAnImage.gif#lightbox)

### <a name="use-keyboard-shortcuts"></a>使用键盘快捷键

可以使用键盘来访问在 Visual Studio Code 的 Azure 机器学习功能。 最重要的键盘快捷方式，要知道是 Ctrl + Shift + P，后者将显示命令面板。 从命令面板中，您可以访问所有 Visual Studio Code 中，包括最常见的操作的键盘快捷方式的功能。

[![针对 Visual Studio Code 的 Azure 机器学习的键盘快捷方式](./media/vscode-tools-for-ai/commands.gif)](./media/vscode-tools-for-ai/commands.gif#lightbox)

## <a name="next-steps"></a>后续步骤

* 有关如何使用 Visual Studio Code 之外的 Azure 机器学习训练的演练，请参阅[教程：使用 Azure 机器学习训练模型](tutorial-train-models-with-aml.md)。
* 有关如何编辑、 运行和调试本地代码的演练，请参阅[Python 你好 world 教程](https://code.visualstudio.com/docs/python/python-tutorial)。
