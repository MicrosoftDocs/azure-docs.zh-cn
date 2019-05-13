---
title: 为模型训练创建和使用计算目标
titleSuffix: Azure Machine Learning service
description: 为机器学习模型训练配置训练环境（计算目标）。 可以轻松地在训练环境之间切换。 在本地开始训练。 如果需要横向扩展，请切换到基于云的计算目标。
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 01/07/2019
ms.custom: seodec18
ms.openlocfilehash: c49b9d5fdc0c17f16f1c80471a00dd53625dc6e8
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236950"
---
# <a name="set-up-compute-targets-for-model-training"></a>设置模型训练的计算目标

使用 Azure 机器学习服务可以在不同的资源或环境（统称为[__计算目标__](concept-azure-machine-learning-architecture.md#compute-target)）中训练模型。 计算目标可以是本地计算机，也可以是云资源，例如 Azure 机器学习计算、Azure HDInsight 或远程虚拟机。  还可以为模型部署创建计算目标，如[“部署模型的位置和方式”](how-to-deploy-and-where.md)中所述。

可以使用 Azure 机器学习 SDK、Azure 门户或 Azure CLI 创建和管理计算目标。 如果通过其他服务（例如 HDInsight 群集）创建了计算目标，可以通过将其附加到 Azure 机器学习服务工作区来使用它们。
 
本文介绍如何使用各种计算目标进行模型训练。  适用于所有计算目标的步骤遵循相同的工作流：
1. __创建__计算目标（如果没有）。
2. 将计算目标__附加__到工作区。
3. __配置__计算目标，使其包含脚本所需的 Python 环境和包依赖项。


>[!NOTE]
> 本文中的代码已使用 Azure 机器学习 SDK 版本 1.0.6 进行测试。

## <a name="compute-targets-for-training"></a>训练的计算目标

Azure 机器学习服务为不同的计算目标提供不同的支持。 典型的模型开发生命周期从开发/试验少量的数据开始。 在此阶段，我们建议使用本地环境。 例如，本地计算机或基于云的 VM。 针对更大的数据集扩展训练或执行分布式训练时，我们建议使用 Azure 机器学习计算来创建可在每次提交运行时自动缩放的单节点或多节点群集。 你也可以附加自己的计算资源，不过，为各种方案提供的支持可能有所不同，详情如下：


|训练的计算目标| GPU 加速 | 自动<br/> 超参数优化 | 自动</br> 机器学习 | Azure 机器学习管道 |
|----|:----:|:----:|:----:|:----:|
|[本地计算机](#local)| 可能 | &nbsp; | ✓ | &nbsp; |
|[Azure 机器学习计算](#amlcompute)| ✓ | ✓ | ✓ | ✓ |
|[远程 VM](#vm) | ✓ | ✓ | ✓ | ✓ |
|[Azure Databricks](how-to-create-your-first-pipeline.md#databricks)| &nbsp; | &nbsp; | ✓ | ✓ |
|[Azure Data Lake Analytics](how-to-create-your-first-pipeline.md#adla)| &nbsp; | &nbsp; | &nbsp; | ✓ |
|[Azure HDInsight](#hdinsight)| &nbsp; | &nbsp; | &nbsp; | ✓ |
|[Azure Batch](#azbatch)| &nbsp; | &nbsp; | &nbsp; | ✓ |

**所有计算目标都可重复用于多个训练作业**。 例如，将远程 VM 附加到你的工作区后，可以将其重复用于多个作业。

> [!NOTE]
> Azure 机器学习计算资源可以创建为持久资源，也可以在你请求运行时动态创建。 基于运行的创建在训练运行完成后会删除计算目标，因此无法重复使用以这种方式创建的计算目标。

## <a name="whats-a-run-configuration"></a>什么是运行配置？

训练时，通常会在本地计算机上开始，然后在不同的计算目标上运行该训练脚本。 使用 Azure 机器学习服务可以在各种计算目标上运行脚本，而无需更改脚本。 

只需使用**运行配置**为每个计算目标定义环境即可。  然后，当你想要在不同的计算目标上运行训练试验时，可以指定该计算的运行配置。 

本文的最后详细介绍了如何[提交试验](#submit)。

### <a name="manage-environment-and-dependencies"></a>管理环境和依赖项

创建运行配置时，需要确定如何管理计算目标上的环境和依赖项。 

#### <a name="system-managed-environment"></a>系统管理的环境

若要通过 [Conda](https://conda.io/docs/) 管理 Python 环境和脚本依赖项，请使用系统管理的环境。 默认已采用系统管理的环境，这是最常见的选项。 系统管理的环境在远程计算目标上非常有用，尤其是无法配置该目标时。 

只需使用 [CondaDependency 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.conda_dependencies.condadependencies?view=azure-ml-py)指定每个包依赖项即可。然后，Conda 将在工作区中的 **aml_config** 目录内创建名为 **conda_dependencies.yml** 的、包含包依赖项列表的文件，并在你提交训练试验时设置 Python 环境。 

新环境的初始设置可能需要几分钟才能完成，具体取决于所需依赖项的大小。 只要包列表保持不变，就只需设置一次。
  
以下代码演示了一个需要 scikit-learn 的系统管理环境示例：
    
[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/runconfig.py?name=run_system_managed)]

#### <a name="user-managed-environment"></a>用户管理的环境

对于用户管理的环境，你需要负责设置环境，并在计算目标上安装训练脚本所需的每个包。 如果已配置训练环境（例如，在本地计算机上），可以通过将 `user_managed_dependencies` 设置为 True 来跳过设置步骤。 Conda 将不检查你的环境，也不会安装任何组件。

以下代码演示了一个为用户管理的环境配置训练运行的示例：

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/runconfig.py?name=run_user_managed)]
  
## <a name="set-up-compute-targets-with-python"></a>使用 Python 设置计算目标

使用以下部分配置这些计算目标：

* [本地计算机](#local)
* [Azure 机器学习计算](#amlcompute)
* [远程虚拟机](#vm)
* [Azure HDInsight](#hdinsight)


### <a id="local"></a>本地计算机

1. **创建和附加**：无需创建或附加计算目标即可将本地计算机用作训练环境。  

1. **配置**：将本地计算机用作计算目标时，训练代码将在[开发环境](how-to-configure-environment.md)中运行。  如果该环境已包含所需的 Python 包，请使用用户管理的环境。

 [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=run_local)]

附加计算并配置运行后，下一步是[提交训练运行](#submit)。

### <a id="amlcompute"></a>Azure 机器学习计算

Azure 机器学习计算是一个托管的计算基础结构，可让用户轻松创建单节点或多节点计算。 该计算是在工作区区域内部创建的，是可与工作区中的其他用户共享的资源。 提交作业时，计算会自动扩展，并可以放入 Azure 虚拟网络。 计算在容器化环境中执行，将模型的依赖项打包在 [Docker 容器](https://www.docker.com/why-docker)中。

可以使用 Azure 机器学习计算在云中的 CPU 或 GPU 计算节点群集之间分配训练进程。 有关包括 GPU 的 VM 大小的详细信息，请参阅 [GPU 优化的虚拟机大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu)。

Azure 机器学习计算对可以分配的核心数等属性实施默认限制。 有关详细信息，请参阅[管理和请求 Azure 资源的配额](https://docs.microsoft.com/azure/machine-learning/service/how-to-manage-quotas)。


可以在计划运行时按需创建 Azure 机器学习计算环境，或者将其创建为持久性资源。

#### <a name="run-based-creation"></a>基于运行的创建

可将 Azure 机器学习计算创建为运行时的计算目标。 将自动为运行创建计算。 完成运行后，会自动删除计算。 

> [!NOTE]
> 若要指定要使用的节点的最大数量，通常可以设置`node_count`到的节点数。 目前 (04/04/2019) 可以避免这一工作的 bug。 作为一种解决方法，使用`amlcompute._cluster_max_node_count`运行配置的属性。 例如，`run_config.amlcompute._cluster_max_node_count = 5`。

> [!IMPORTANT]
> Azure 机器学习计算的基于运行的创建功能目前为预览版。 如果使用自动化超参数优化或自动化机器学习，请不要使用基于运行的创建。 若要使用超参数优化或自动化机器学习，请改为创建[持久性计算](#persistent)目标。

1.  **创建、附加和配置**：基于运行的创建功能将使用运行配置执行所有必要的步骤来创建、附加和配置计算目标。  

  [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute.py?name=run_temp_compute)]


附加计算并配置运行后，下一步是[提交训练运行](#submit)。

#### <a id="persistent"></a>持久性计算

可在不同的作业中重复使用持久性 Azure 机器学习计算。 计算可与工作区中的其他用户共享，完成每个作业后可以保留。

1. **创建和附加**：若要在 Python 中创建持久性 Azure 机器学习计算资源，请指定 **vm_size** 和 **max_nodes** 属性。 然后，Azure 机器学习将对其他属性使用智能默认值。 计算在不使用时自动缩减为零个节点。   按需创建专用 VM 来运行作业。
    
    * **vm_size**：Azure 机器学习计算创建的节点的 VM 系列。
    * **max_nodes**：在 Azure 机器学习计算中运行作业时自动扩展到的最大节点数。
    
   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=cpu_cluster)]

   还可以在创建 Azure 机器学习计算时配置多个高级属性。 使用这些属性可以创建固定大小的持久性群集，或者在订阅中的现有 Azure 虚拟网络内创建持久性群集。  有关详细信息，请参阅 [AmlCompute 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py
    )。
    
   或者，可以[在 Azure 门户中](#portal-create)创建并附加持久性 Azure 机器学习计算资源。

1. **配置**：为持久性计算目标创建运行配置。

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=run_amlcompute)]

附加计算并配置运行后，下一步是[提交训练运行](#submit)。


### <a id="vm"></a>远程虚拟机

Azure 机器学习还支持将自己的计算资源附加到工作区。 任意远程 VM 就是这样一种资源类型（前提是可从 Azure 机器学习服务访问它）。 该资源可以是 Azure VM，也可以是组织内部或本地的远程服务器。 具体而言，在指定 IP 地址和凭据（用户名和密码，或 SSH 密钥）的情况下，可以使用任何可访问的 VM 进行远程运行。

可以使用系统生成的 conda 环境、现有的 Python 环境或 Docker 容器。 若要在 Docker 容器中执行，必须在 VM 上运行 Docker 引擎。 需要一个比本地计算机更灵活的基于云的开发/试验环境时，此功能特别有用。

请对此方案使用 Data Science Virtual Machine (DSVM) 作为 Azure VM。 此 VM 在 Azure 中预配置了数据科学和 AI 开发环境。 此 VM 提供精选的工具和框架用于满足整个机器学习开发生命周期的需求。 有关如何将 DSVM 与 Azure 机器学习配合使用的详细信息，请参阅[配置开发环境](https://docs.microsoft.com/azure/machine-learning/service/how-to-configure-environment#dsvm)。

1. **创建**：创建 DSVM，然后使用它来训练模型。 若要创建此资源，请参阅[预配适用于 Linux (Ubuntu) 的 Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro)。

    > [!WARNING]
    > Azure 机器学习仅支持运行 Ubuntu 的虚拟机。 创建 VM 或选择现有 VM 时，必须选择使用 Ubuntu 的 VM。

1. **附加**：若要附加现有虚拟机作为计算目标，必须提供虚拟机的完全限定域名 (FQDN)、用户名和密码。 在本示例中，请将 \<fqdn> 替换为 VM 的 FQDN，或替换为公共 IP 地址。 请将 \<username> 和 \<password> 替换为 VM 的 SSH 用户名和密码。

   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   attach_config = RemoteCompute.attach_configuration(address = "<fqdn>",
                                                    ssh_port=22,
                                                    username='<username>',
                                                    password="<password>")

   # If you authenticate with SSH keys instead, use this code:
   #                                                  ssh_port=22,
   #                                                  username='<username>',
   #                                                  password=None,
   #                                                  private_key_file="<path-to-file>",
   #                                                  private_key_passphrase="<passphrase>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   或者，可以[使用 Azure 门户](#portal-reuse)将 DSVM 附加到工作区。

1. **配置**：为 DSVM 计算目标创建运行配置。 Docker 与 conda 用于在 DSVM 上创建和配置训练环境。

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/dsvm.py?name=run_dsvm)]


附加计算并配置运行后，下一步是[提交训练运行](#submit)。

### <a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight 是用于大数据分析的热门平台。 该平台提供的 Apache Spark 可用于训练模型。

1. **创建**：先创建 HDInsight 群集，然后使用它来训练模型。 若要在 HDInsight 群集中创建 Spark，请参阅[在 HDInsight 中创建 Spark 群集](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql)。 

    创建群集时，必须指定 SSH 用户名和密码。 请记下这些值，因为在将 HDInsight 用作计算目标时需要用到这些值。
    
    创建群集后，使用主机名 \<clustername>-ssh.azurehdinsight.net 连接到该群集，其中，\<clustername> 是为该群集提供的名称。 

1. **附加**：若要将 HDInsight 群集作为计算目标附加，必须提供该 HDInsight 群集的主机名、用户名和密码。 下面的示例使用 SDK 将群集附加到工作区。 在该示例中，请将 \<clustername> 替换为群集名称。 请将 \<username> 和 \<password> 替换为群集的 SSH 用户名和密码。

   ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase
    attach_config = HDInsightCompute.attach_configuration(address='<clustername>-ssh.azureinsight.net', 
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   或者，可以[使用 Azure 门户](#portal-reuse)将 HDInsight 群集附加到工作区。

1. **配置**：为 HDI 计算目标创建运行配置。 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]


附加计算并配置运行后，下一步是[提交训练运行](#submit)。


### <a id="azbatch"></a>Azure 批处理 

使用 azure Batch 在云中高效运行大规模并行和高性能计算 (HPC) 应用程序。 可以在 Azure 机器学习管道中使用 AzureBatchStep，若要将作业提交到计算机的 Azure Batch 池。

若要附加 Azure Batch 用作计算目标，必须使用 Azure 机器学习 SDK，并提供以下信息：

-   **Azure Batch 计算名称**:一个友好名称以用于在工作区中的计算
-   **Azure Batch 帐户名称**:Azure Batch 帐户的名称
-   **资源组**：包含 Azure Batch 帐户的资源组。

下面的代码演示如何将附加 Azure Batch 用作计算目标：

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

batch_compute_name = 'mybatchcompute' # Name to associate with new compute in workspace

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>" # Name of the Batch account
batch_resource_group = "<batch_resource_group>" # Name of the resource group which contains this account

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

## <a name="set-up-compute-in-the-azure-portal"></a>在 Azure 门户中设置计算

可以在 Azure 门户中访问与工作区关联的计算目标。  可以使用门户执行以下操作：

* [查看](#portal-view)附加到工作区的计算目标
* 在工作区中[创建计算目标](#portal-create)
* [附加](#portal-reuse)在工作区外部创建的计算目标

创建目标并将其附加到工作区后，可以通过一个 `ComputeTarget` 对象在运行配置中使用该目标： 

```python
from azureml.core.compute import ComputeTarget
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

### <a id="portal-view"></a>查看计算目标


若要查看工作区的计算目标，请使用以下步骤：

1. 导航到 [Azure 门户](https://portal.azure.com)，并打开你的工作区。 
1. 在“应用程序”下，选择“计算”。

    ![查看“计算”选项卡](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace.png)

### <a id="portal-create"></a>创建计算目标

遵循上述步骤查看计算目标的列表。 然后使用以下步骤创建计算目标： 

1. 选择加号 (+) 添加计算目标。

    ![添加计算目标](./media/how-to-set-up-training-targets/add-compute-target.png) 

1. 输入计算目标的名称。 

1. 选择“机器学习计算”作为用于训练的计算类型。 

    >[!NOTE]
    >Azure 机器学习计算是可在 Azure 门户中创建的唯一一种托管计算资源。  创建其他所有计算资源后，可以附加这些资源。

1. 填写表单。 提供必需属性的值，尤其是“VM 系列”，以及用于运转计算的**最大节点数**。  

    ![填写表单](./media/how-to-set-up-training-targets/add-compute-form.png) 

1. 选择“创建”。


1. 通过在列表中选择计算目标来查看创建操作的状态：

    ![选择一个计算目标以查看创建操作的状态](./media/how-to-set-up-training-targets/View_list.png)

1. 然后会看到相应计算目标的详细信息： 

    ![查看计算目标详细信息](./media/how-to-set-up-training-targets/compute-target-details.png) 



### <a id="portal-reuse"></a>附加计算目标

若要使用在 Azure 机器学习服务工作区外部创建的计算目标，必须附加它们。 附加计算目标会使其可供你的工作区使用。

遵循上述步骤查看计算目标的列表。 然后使用以下步骤来附加计算目标： 

1. 选择加号 (+) 添加计算目标。 
1. 输入计算目标的名称。 
1. 选择要为训练附加的计算类型：

    > [!IMPORTANT]
    > 并非所有计算类型都可以从 Azure 门户附加。 目前，可为训练附加的计算类型包括：
    >
    > * 远程 VM
    > * Azure Databricks（在机器学习管道中使用）
    > * Azure Data Lake Analytics（在机器学习管道中使用）
    > * Azure HDInsight

1. 填写表单，并提供必需属性的值。

    > [!NOTE]
    > Microsoft 建议使用 SSH 密钥，因为它们比密码更安全。 密码很容易受到暴力破解攻击。 SSH 密钥依赖于加密签名。 若要了解如何创建用于 Azure 虚拟机的 SSH 密钥，请参阅以下文档：
    >
    > * [在 Linux 或 macOS 上创建和使用 SSH 密钥](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)
    > * [在 Windows 上创建和使用 SSH 密钥](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)

1. 选择“附加”。 
1. 通过在列表中选择计算目标来查看附加操作的状态。

## <a name="set-up-compute-with-the-cli"></a>使用 CLI 设置计算

可以使用适用于 Azure 机器学习服务的 [CLI 扩展](reference-azure-machine-learning-cli.md)访问与工作区关联的计算目标。  可以使用 CLI 执行以下操作：

* 创建托管计算目标
* 更新托管计算目标
* 附加托管计算目标

有关详细信息，请参阅[资源管理](reference-azure-machine-learning-cli.md#resource-management)。

## <a id="submit"></a>提交训练运行

创建运行配置后，可以使用它来运行试验。  对于所有类型的计算目标，用于提交训练运行的代码模式都是相同的：

1. 创建要运行的试验
1. 提交运行任务。
1. 等待运行任务完成。

> [!IMPORTANT]
> 当提交培训运行时，快照包含训练脚本的目录的创建，并发送到计算目标。 它还存储为你的工作区中的实验的一部分。 如果更改文件和提交运行同样，将上载更改的文件。
>
> 若要防止文件被包含在快照中，创建[.gitignore](https://git-scm.com/docs/gitignore)或`.amlignore`文件的目录中，并向其中添加文件。 `.amlignore`文件使用相同的语法，作为模式[.gitignore](https://git-scm.com/docs/gitignore)文件。 如果这两个文件存在，`.amlignore`文件具有优先权。
> 
> 有关详细信息，请参阅[快照](concept-azure-machine-learning-architecture.md#snapshot)。

### <a name="create-an-experiment"></a>创建试验

首先，在工作区中创建一个试验。

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=experiment)]

### <a name="submit-the-experiment"></a>提交试验

使用 `ScriptRunConfig` 对象提交该试验。  此对象包含：

* **source_directory**：包含训练脚本的源目录
* **script**：标识训练脚本
* **run_config**：运行配置，其中定义训练位置。

例如，若要使用[本地目标](#local)配置：

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/local.py?name=local_submit)]

使用不同的运行配置（例如 [amlcompute 目标](#amlcompute)）将同一试验切换为在不同的计算目标中运行：

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=amlcompute_submit)]

或者可以：

* 根据[使用评估器训练机器学习模型](how-to-train-ml-models.md)中所述，使用 `Estimator` 对象提交试验。 
* [使用 CLI 扩展](reference-azure-machine-learning-cli.md#experiments)提交试验。

## <a name="notebook-examples"></a>Notebook 示例

请参阅以下 Notebook，其中提供了有关使用各种计算目标进行训练的示例：
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>后续步骤

* [教程：训练模型](tutorial-train-models-with-aml.md)使用一个托管计算目标来训练模型。
* 了解如何[有效地优化超参数](how-to-tune-hyperparameters.md)来构建更好的模型。
* 训练模型后，了解[如何以及在何处部署模型](how-to-deploy-and-where.md)。
* 查看 [RunConfiguration 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py) SDK 参考。
* [通过 Azure 虚拟网络使用 Azure 机器学习服务](how-to-enable-virtual-network.md)
