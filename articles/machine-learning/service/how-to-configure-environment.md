---
title: 设置 Python 开发环境
titleSuffix: Azure Machine Learning service
description: 了解在使用 Azure 机器学习服务时如何配置开发环境。 在本文中，您将了解如何使用 Conda 环境、 创建配置文件，并配置你自己的基于云的 notebook 服务器、 Jupyter Notebook、 Azure Databricks、 Azure Notebooks、 Ide、 代码编辑器和数据科学虚拟机。
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.topic: conceptual
ms.date: 02/24/2019
ms.custom: seodec18
ms.openlocfilehash: 4d588374c0195e7da373766f93f6829ac2160269
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471596"
---
# <a name="configure-a-development-environment-for-azure-machine-learning"></a>配置 Azure 机器学习的开发环境

本文介绍如何将开发环境配置为使用 Azure 机器学习服务。 机器学习服务不区分平台。

在开发环境的唯一要求是 Python 3，Anaconda （适用于隔离环境中），并包含你的 Azure 机器学习工作区信息的配置文件。

本文重点介绍以下环境和工具：

* 您自己[基于云的 notebook 服务器](#notebookvm):使用你的工作站中的计算资源运行的 Jupyter 笔记本。 这是最简单的入门方法，因为已安装 Azure 机器学习 SDK。

* [Data Science Virtual Machine (DSVM)](#dsvm)：Azure 云中的预配置开发或试验环境，用于开展数据科学工作，可以部署到仅限 CPU 的 VM 实例或基于 GPU 的实例。 已安装 Python 3、Conda、Jupyter Notebook 和 Azure 机器学习 SDK。 VM 配备了用于开发机器学习解决方案的常用机器学习和深度学习框架、工具与编辑器。 对于 Azure 平台上的机器学习，它可能是最完整的开发环境。

* [Jupyter Notebook](#jupyter)：如果你已在使用 Jupyter Notebook，SDK 中包含了需要安装的某些附加功能。

* [Visual Studio Code](#vscode)：如果使用 Visual Studio Code，其中包含了一些可以安装的有用扩展。

* [Azure Databricks](#aml-databricks)：基于 Apache Spark 的常用数据分析平台。 了解如何将 Azure 机器学习 SDK 添加到群集中，以便部署模型。

* [Azure Notebooks](#aznotebooks)：Azure 云中托管的 Jupyter Notebook 服务。 此外轻松地开始，因为已安装 Azure 机器学习 SDK。  

如果还没有 Python 3 环境，或者只想了解安装 SDK 的基本步骤，请参阅[本地计算机](#local)部分。

## <a name="prerequisites"></a>必备组件

- Azure 机器学习服务工作区。 若要创建工作区，请参阅[创建 Azure 机器学习服务工作区](setup-create-workspace.md)。

工作区是您需要开始使用你自己[基于云的 notebook 服务器](#notebookvm)即[DSVM](#dsvm)， [Azure Databricks](#aml-databricks)，或[Azure Notebooks](#aznotebooks).

若要安装的 SDK 环境你[本地计算机](#local)， [Jupyter Notebook 服务器](#jupyter)或[Visual Studio Code](#vscode)还需要：

- 任一[Anaconda](https://www.anaconda.com/download/)或[Miniconda](https://conda.io/miniconda.html)包管理器。

- 在 Linux 或 macOS 上，需要 bash shell。

    > [!TIP]
    > 如果在 Linux 或 macOS 上操作，并使用除 bash 以外的 shell（例如 zsh），则在运行某些命令时可能会收到错误消息。 若要解决此问题，请使用 `bash` 命令启动新的 bash shell，然后运行命令。

- 在 Windows 上，需要命令提示符或 Anaconda 提示符（由 Anaconda 和 Miniconda 安装）。

## <a id="notebookvm"></a>基于云的笔记本服务器

若要开始使用 Azure 机器学习开发的最简单方式在 Azure 机器学习工作区中创建的笔记本服务器。

* 已安装 Azure 机器学习 SDK。
* Notebook VM 环境自动配置为使用你的工作区。
* 该资源在你的工作区中创建，并可以那里管理

若要开始开发基于云的 notebook 服务器，请参阅[快速入门：使用基于云的 notebook 服务器来开始使用 Azure 机器学习](quickstart-run-cloud-notebook.md)。


## <a id="dsvm"></a>Data Science Virtual Machine

DSVM 是自定义的虚拟机 (VM) 映像。 它专为数据科学工作而设计，其中预配置了：

  - TensorFlow、PyTorch、Scikit-learn、XGBoost 和 Azure 机器学习 SDK 等包
  - Spark Standalone 和 Drill 等常用数据科学工具
  - Azure CLI、AzCopy 和存储资源管理器等 Azure 工具
  - Visual Studio Code 和 PyCharm 等集成开发环境 (IDE)
  - Jupyter Notebook 服务器

Azure 机器学习 SDK 适用于 Ubuntu 或 Windows 版本的 DSVM。 但是，如果还计划将 DSVM 用作计算目标，则仅支持 Ubuntu。

若要使用 DSVM 作为开发环境，请执行以下操作：

1. 在以下任一环境中创建 DSVM：

    * Azure 门户：

        * [创建 Ubuntu Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro)

        * [创建 Windows Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/provision-vm)

    * Azure CLI：

        > [!IMPORTANT]
        > * 使用 Azure CLI 时，必须先使用 `az login` 命令登录到 Azure 订阅。
        >
        > * 在此步骤中使用命令时，必须提供资源组名称、VM名称、用户名和密码。

        * 若要创建 Ubuntu Data Science Virtual Machine，请使用以下命令：

            ```azurecli
            # create a Ubuntu DSVM in your resource group
            # note you need to be at least a contributor to the resource group in order to execute this command successfully
            # If you need to create a new resource group use: "az group create --name YOUR-RESOURCE-GROUP-NAME --location YOUR-REGION (For example: westus2)"
            az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:linux-data-science-vm-ubuntu:linuxdsvmubuntu:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --generate-ssh-keys --authentication-type password
            ```

        * 若要创建 Windows Data Science Virtual Machine，请使用以下命令：

            ```azurecli
            # create a Windows Server 2016 DSVM in your resource group
            # note you need to be at least a contributor to the resource group in order to execute this command successfully
            az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:dsvm-windows:server-2016:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --authentication-type password
            ```

2. DSVM 上已安装 Azure 机器学习 SDK。 若要使用包含此 SDK 的 Conda 环境，请使用以下某个命令：

    * 对于 Ubuntu DSVM：

        ```shell
        conda activate py36
        ```

    * 对于 Windows DSVM：

        ```shell
        conda activate AzureML
        ```

1. 若要验证是否可以访问 SDK 并检查版本，请使用以下 Python 代码：

    ```python
    import azureml.core
    print(azureml.core.VERSION)
    ```

1. 若要将 DSVM 配置为使用你的 Azure 机器学习服务工作区，请参阅[创建工作区配置文件](#workspace)部分。

有关详细信息，请参阅 [Data Science Virtual Machine](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)。

## <a id="local"></a>本地计算机

当你使用的是本地计算机 （这可能也是远程的虚拟机） 时，创建的 Anaconda 环境，并通过执行以下安装 SDK:

1. 下载并安装[Anaconda](https://www.anaconda.com/distribution/#download-section) （Python 3.7 版） 如果你尚未安装。

1. 打开 Anaconda 提示符，并使用以下命令创建环境：

    运行以下命令以创建环境。

    ```shell
    conda create -n myenv python=3.6.5
    ```

    然后激活的环境。

    ```shell
    conda activate myenv
    ```

    此示例将创建使用 python 3.6.5 的环境，但可以选择任何特定 subversions。 某些主要版本 （3.5 + 建议），则可能不能保证 SDK 兼容性，我们建议尝试其他版本/subversion Anaconda 环境中，如果遇到错误。 下载组件和包时，创建环境需要花费几分钟。

1. 若要启用特定于环境的 ipython 内核在新环境中运行以下命令。 这将确保预期的内核和包时，导入行为 Anaconda 环境中使用 Jupyter Notebook:

    ```shell
    conda install notebook ipykernel
    ```

    然后运行以下命令以创建内核：

    ```shell
    ipython kernel install --user
    ```

1. 使用以下命令来安装程序包：

    此命令将使用 notebook 和 automl extras 安装基本的 Azure 机器学习 SDK。 `automl`额外是大型安装，并且如果不想运行自动化的机器学习试验可以从括号删除。 `automl`额外还作为依赖项的默认情况下包括 Azure 机器学习数据准备 SDK。

     ```shell
    pip install azureml-sdk[notebooks,automl]
    ```

    使用以下命令以安装 Azure 机器学习数据准备 SDK:

    ```shell
    pip install azureml-dataprep
    ```

   > [!NOTE]
   > 如果有消息指出无法卸载 PyYAML，请改用以下命令：
   >
   > `pip install --upgrade azureml-sdk[notebooks,automl] azureml-dataprep --ignore-installed PyYAML`

   它将需要几分钟才能安装 SDK。 请参阅[安装指南](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py)的安装选项的详细信息。

1. 安装机器学习试验的其他包。

    使用以下命令之一和替换*\<新的包 >* 与你想要安装的包。 安装包通过`conda install`需要包是 （Anaconda 云中可以添加新的通道） 的当前通道的一部分。

    ```shell
    conda install <new package>
    ```

    或者，可以安装包通过`pip`。

    ```shell
    pip install <new package>
    ```

### <a id="jupyter"></a>Jupyter Notebook

Jupyter Notebook 是 [Jupyter 项目](https://jupyter.org/)的一部分。 它们提供交互式编码体验，可用于创建将实时代码与叙述性文本和图形混合在一起的文档。 Jupyter Notebook 也是与他人共享结果的好方法，因为可用于将代码部分的输出保存在文档中。 可以在各种平台上安装 Jupyter Notebook。

中的过程[本地计算机](#local)部分安装的 Anaconda 环境中运行的 Jupyter 笔记本的必需组件。 若要在 Jupyter Notebook 环境中启用这些组件，请执行以下操作：

1. 打开 Anaconda 提示符，并激活你的环境。

    ```shell
    conda activate myenv
    ```

1. 启动 Jupyter Notebook 服务器使用以下命令：

    ```shell
    jupyter notebook
    ```

1. 若要验证 Jupyter Notebook，可以使用 SDK，请创建**新建**笔记本中，选择**Python 3**作为内核，然后运行以下命令在笔记本单元格中：

    ```python
    import azureml.core
    azureml.core.VERSION
    ```

1. 如果您遇到问题，导入模块，并获得`ModuleNotFoundError`，确保 Jupyter 内核已通过在笔记本单元中运行下面的代码连接到你的环境的正确路径。

    ```python
    import sys
    sys.path
    ```

1. 若要将 Jupyter Notebook 配置为使用你的 Azure 机器学习服务工作区，请转到[创建工作区配置文件](#workspace)部分。

### <a id="vscode"></a>Visual Studio Code

Visual Studio Code 是跨平台代码编辑器。 它依赖于本地 Python 3 和 Python 支持的 Conda 安装，但它提供用于 AI 的其他工具。 它还支持从代码编辑器中选择 Conda 环境。

若要使用 Visual Studio Code 进行开发，请执行以下操作：

1. 若要了解如何使用 Visual Studio Code 进行 Python 开发，请参阅 [VSCode 中的 Python 入门](https://code.visualstudio.com/docs/python/python-tutorial)。

1. 若要选择 Conda 环境，请打开 VS Code，然后按 Ctrl-Shift-P（Linux 和 Windows）或 Command-Shift-P (Mac)。
    此时会打开“命令面板”。

1. 输入 Python:__Select Interpreter__，然后选择 Conda 环境。

1. 若要验证是否可以使用 SDK，请创建并运行包含以下代码的新 Python 文件 (.py)。

    ```python
    import azureml.core
    azureml.core.VERSION
    ```

1. 若要安装适用于 Visual Studio Code 的 Azure 机器学习扩展，请参阅 [AI 工具](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai)。

    有关详细信息，请参阅[使用适用于 Visual Studio Code 的 Azure 机器学习](how-to-vscode-tools.md)。

<a name="aml-databricks"></a>

## <a name="azure-databricks"></a>Azure Databricks
Azure Databricks 是基于 Apache Spark 的 Azure 云环境。 它提供协作 Notebook 基于环境和 CPU 或 GPU 基于的计算群集。

Azure Databricks 的工作原理与 Azure 机器学习服务：
+ 可以使用 Spark MLlib 训练模型并将模型部署到 ACI/AKS 从 Azure Databricks 中。 
+ 此外可以使用[自动化的机器学习](concept-automated-ml.md)特殊的 Azure ML SDK 使用 Azure Databricks 中的功能。
+ 用作计算目标从 Azure Databricks [Azure 机器学习管道](concept-ml-pipelines.md)。 

### <a name="set-up-your-databricks-cluster"></a>设置 Databricks 群集

创建[Databricks 群集](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal)。 安装适用于自动化机器学习在 Databricks 上的 SDK 的情况下，才会应用某些设置。
**它将需要几分钟以创建群集。**

使用以下设置：

| 设置 |应用于| 值 |
|----|---|---|
| 群集名称 |始终| yourclustername |
| Databricks 运行时 |始终| 任何非机器学习运行时（非机器学习 4.x、5.x） |
| Python 版本 |始终| 3 |
| 辅助程序 |始终| 2 个或以上 |
| 工作节点 VM 类型 <br>（确定并发迭代的数上限） |自动化机器学习<br>仅| 首选内存优化的 VM |
| 启用自动缩放 |自动化机器学习<br>仅| 取消选中 |

请等待群集运行完成，然后继续操作。

### <a name="install-the-correct-sdk-into-a-databricks-library"></a>将正确的 SDK 安装到 Databricks 库
群集正在运行，一旦[创建一个库](https://docs.databricks.com/user-guide/libraries.html#create-a-library)将附加到你的群集的相应的 Azure 机器学习 SDK 包。 

1. 选择**只有一个**选项 （支持任何其他 SDK 安装）

   |SDK&nbsp;包&nbsp;其他功能|源|PyPi&nbsp;Name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
   |----|---|---|
   |Databricks| 上传 Python Egg 或 PyPI | azureml-sdk[databricks]|
   |具有的-databricks<br> 自动化机器学习功能| 上传 Python Egg 或 PyPI | azureml-sdk[automl_databricks]|

   > [!Warning]
   > 可以不安装任何其他 SDK 其他功能。 选择上述选项 [databricks] 或 [automl_databricks] 只有一个。

   * 不要选择**将自动附加到所有群集**。
   * 选择**附加**群集名称旁边。

1. 监视错误，直到状态变为**附加**，这可能需要几分钟时间。  如果此步骤失败，请检查以下项： 

   请尝试重新启动的群集：
   1. 在左窗格中，选择“群集”。
   1. 在表中选择你的群集名称。
   1. 在“库”选项卡上，选择“重启”。
      
   另请考虑：
   + 在 Automl 的配置来说，当使用 Azure Databricks 请添加以下参数：
       1. ```max_concurrent_iterations``` 基于群集中的辅助角色节点数。 
        2. ```spark_context=sc``` 基于默认 spark 上下文。 
   + 或者，如果有旧的 SDK 版本，取消选中此选项从群集的已安装库，然后移到回收站。 安装新的 SDK 版本并重启群集。 如果完成此操作后出现问题，请分离并重新附加群集。

如果安装成功，导入的库应如下所示其中之一：
   
SDK databricks **_而无需_** 自动机器学习![databricks 的 Azure 机器学习 SDK](./media/how-to-configure-environment/amlsdk-withoutautoml.jpg)

SDK databricks **WITH**自动执行机器学习![SDK 自动安装在 Databricks 上的机器学习](./media/how-to-configure-environment/automlonadb.jpg)

### <a name="start-exploring"></a>开始浏览

进行试用：
+ 下载[notebook 存档文件](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks/Databricks_AMLSDK_1-4_6.dbc)适用于 Azure Databricks/Azure 机器学习 SDK 并[存档文件导入](https://docs.azuredatabricks.net/user-guide/notebooks/notebook-manage.html#import-an-archive)到 Databricks 群集。  
  虽然多个示例笔记本可用，但**仅[这些示例 notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks)适用于 Azure Databricks。**
  
+ 了解如何[创建包含作为培训计算 Databricks 的管道](how-to-create-your-first-pipeline.md)。

## <a id="aznotebooks"></a>Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com)（预览版）是 Azure 云中的交互式开发环境。 它是开始使用 Azure 机器学习开发的简单办法。

* 已安装 Azure 机器学习 SDK。
* 在 Azure 门户中创建了 Azure 机器学习服务工作区之后，可以单击相应的按钮将 Azure Notebook 环境自动配置为使用该工作区。

使用 [Azure 门户](https://portal.azure.com)完成 Azure Notebooks 入门。  打开你的工作区以及从**概述**部分中，选择**开始使用 Azure Notebook**。

默认情况下，Azure Notebooks 使用限制为 4GB 内存和 1GB 数据的免费服务层级。 不过，可以通过向 Azure Notebooks 项目附加 Data Science Virtual Machine 实例来解除这些限制。 有关详细信息，请参阅[管理和配置 Azure Notebooks 项目 - 计算层](/azure/notebooks/configure-manage-azure-notebooks-projects#compute-tier)。

## <a id="workspace"></a>创建工作区配置文件

工作区配置文件是一个 JSON 文件，用于告知 SDK 如何与 Azure 机器学习服务工作区进行通信。 该文件命名为 *config.json*，其格式如下：

```json
{
    "subscription_id": "<subscription-id>",
    "resource_group": "<resource-group>",
    "workspace_name": "<workspace-name>"
}
```

此 JSON 文件必须采用包含 Python 脚本或 Jupyter Notebook 的目录结构。 它可以是在相同的目录中，一个名为子目录 *.azureml*，或父目录中。

要从代码使用此文件，请使用 `ws=Workspace.from_config()`。 此代码从文件中加载信息，并连接到工作区。

可通过三种方式创建配置文件：

* **按照中的步骤[创建 Azure 机器学习服务工作区](setup-create-workspace.md#sdk)**:将在 Azure Notebooks 库中创建一个 *config.json* 文件。 该文件含包含工作区的配置信息。 可以下载 *config.json* 或将其复制到其他开发环境。

* **下载文件**:在Azure 门户中，选择工作区的“概览”部分中的“下载 config.json”[](https://ms.portal.azure.com)。

     ![Azure 门户](./media/how-to-configure-environment/configure.png)

* **以编程方式创建文件**：以下代码片段通过提供订阅 ID、资源组和工作区名称连接到工作区。 然后，它将工作区配置保存到文件中：

    ```python
    from azureml.core import Workspace

    subscription_id = '<subscription-id>'
    resource_group  = '<resource-group>'
    workspace_name  = '<workspace-name>'

    try:
        ws = Workspace(subscription_id = subscription_id, resource_group = resource_group, workspace_name = workspace_name)
        ws.write_config()
        print('Library configuration succeeded')
    except:
        print('Workspace not found')
    ```

    此代码将写入到配置文件 *.azureml/config.json*文件。


## <a name="next-steps"></a>后续步骤

- 在 Azure 机器学习中使用 MNIST 数据集[训练模型](tutorial-train-models-with-aml.md)
- 查看[适用于 Python 的 Azure 机器学习 SDK](https://aka.ms/aml-sdk) 参考文档
- 了解有关[Azure 机器学习数据准备包](https://aka.ms/data-prep-sdk)
- 