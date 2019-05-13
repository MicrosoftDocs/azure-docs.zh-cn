---
title: 使用 Azure 数据工厂中的 Databricks Notebook 活动运行 Databricks Notebook
description: 了解如何使用 Azure 数据工厂中的 Databricks Notebook 活动针对 Databricks 作业群集运行 Databricks Notebook。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 03/12/2018
ms.author: abnarain
ms.reviewer: douglasl
ms.openlocfilehash: ca208ded54d17c2392d764e4cba67e5f24e03fd3
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60655043"
---
# <a name="run-a-databricks-notebook-with-the-databricks-notebook-activity-in-azure-data-factory"></a>使用 Azure 数据工厂中的 Databricks Notebook 活动运行 Databricks Notebook

本教程介绍如何使用 Azure 门户创建 Azure 数据工厂管道，以便针对 Databricks 作业群集执行 Databricks Notebook， 并在执行过程中向 Databricks Notebook 传递 Azure 数据工厂参数。

在本教程中执行以下步骤：

  - 创建数据工厂。

  - 创建使用 Databricks Notebook 活动的管道。

  - 触发管道运行。

  - 监视管道运行。

如果还没有 Azure 订阅，可以在开始前创建一个 [免费帐户](https://azure.microsoft.com/free/) 。

有关此功能的十一分钟介绍和演示，请观看以下视频：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/ingest-prepare-and-transform-using-azure-databricks-and-data-factory/player]

## <a name="prerequisites"></a>先决条件

  - **Azure Databricks 工作区**。 [创建 Databricks 工作区](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal)或使用现有的 Databricks 工作区。 在 Azure Databricks 工作区中创建 Python Notebook。 然后使用 Azure 数据工厂执行 Notebook 并向其传递参数。

## <a name="create-a-data-factory"></a>创建数据工厂

1.  启动  **Microsoft Edge**  或  **Google Chrome**  Web 浏览器。 目前，仅 Microsoft Edge 和 Google Chrome Web 浏览器支持数据工厂 UI。

1.  在左侧菜单中选择“创建资源”，然后依次选择“分析”、“数据工厂”。 ****  **** ****

    ![新建数据工厂](media/transform-data-using-databricks-notebook/new-azure-data-factory-menu.png)

1.  在“新建数据工厂” **** 窗格中，在“名称”下输入  **ADFTutorialDataFactory** 。 ****

    Azure 数据工厂的名称必须全局唯一 **。 如果看到以下错误，请更改数据工厂的名称。 （例如，使用  **\<yourname\>ADFTutorialDataFactory**）。 有关数据工厂项目的命名规则，请参阅 [数据工厂 - 命名规则](https://docs.microsoft.com/azure/data-factory/naming-rules) 一文。

    ![为新数据工厂提供一个名称](media/transform-data-using-databricks-notebook/new-azure-data-factory.png)

1.  对于“订阅”，请选择要在其中创建数据工厂的 Azure 订阅。 ****

1.  对于“资源组” ****，请执行以下步骤之一：
    
    - 选择“使用现有资源组”，并从下拉列表选择现有的资源组。 ****
    
    - 选择“新建”，并输入资源组的名称。 ****

    本快速入门中的一些步骤假定你为资源组使用  **ADFTutorialResourceGroup**  名称。 若要了解有关资源组的信息，请参阅 [使用资源组管理 Azure 资源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。

1.  对于“版本” ****，选择“V2” ****。

1.  对于“位置” ****，选择数据工厂所在的位置。

    若要查看目前提供数据工厂的 Azure 区域的列表，请在以下页面上选择感兴趣的区域，然后展开“分析”以找到“数据工厂”：[可用产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/)。 数据工厂使用的数据存储（例如 Azure 存储和 Azure SQL 数据库）和计算资源（例如 HDInsight）可以位于其他区域。
1.  选择“创建” ****。


1.  创建完成后，会显示“数据工厂”页。 ****  选择“创作和监视”磁贴，在单独的选项卡中启动数据工厂 UI 应用程序。 **** 

    ![启动数据工厂 UI 应用程序](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image4.png)

## <a name="create-linked-services"></a>创建链接服务

在本部分，请创作 Databricks 链接服务。 此链接服务包含 Databricks 群集的连接信息：

### <a name="create-an-azure-databricks-linked-service"></a>创建 Azure Databricks 链接服务

1.  在“入门”页的左侧面板中，切换到“编辑”选项卡。 ****  **** 

    ![编辑新的链接服务](media/transform-data-using-databricks-notebook/get-started-page.png)

1.  选择窗口底部的“连接”，然后选择“+ 新建”。 ****  ****
    
    ![创建新连接](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image6.png)

1.  在“新建链接服务”窗口中，选择“计算” \>“Azure Databricks”，然后选择“继续”。 ****  **** **** ****
    
    ![指定 Databricks 链接服务](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image7.png)

1.  在“新建链接服务” **** 窗口中，完成以下步骤：
    
    1.  对于“名称”，请输入 ***AzureDatabricks\_LinkedService***
    
    1.  选择你将在其中运行 notebook 的相应 **Databricks 工作区**

    1.  对于“选择群集”，请选择“新建作业群集”
    
    1.  对于“域/区域”，应自动填充信息

    1.  至于“访问令牌”，请从 Azure Databricks 工作区生成。 可以在[此处](https://docs.databricks.com/api/latest/authentication.html#generate-token)找到步骤。

    1.  对于群集版本，请选择 4.2（带有 Apache Spark 2.3.1、Scala 2.11）

    1.  对于“群集节点类型”，请在本教程的“常规用途(HDD)”类别下选择“Standard\_D3\_v2”。 
    
    1.  对于“辅助角色”，输入 **2**。
    
    1.  选择“完成”

        ![完成创建链接服务的操作](media/transform-data-using-databricks-notebook/new-databricks-linkedservice.png)

## <a name="create-a-pipeline"></a>创建管道

1.  选择  **+** （加号）按钮，然后在菜单上选择“管道”。 **** 

    ![用于创建新管道的按钮](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image9.png)

1.  创建在**管道**中使用的**参数**。 稍后请将此参数传递给 Databricks Notebook 活动。 在空管道中单击“参数”选项卡，然后单击“新建”并将其命名为 '**name**'。

    ![新建参数](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image10.png)

    ![创建名称参数](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image11.png)

1.  在“活动” **** 工具箱中，展开“Databricks” ****。 将“Notebook” **** 活动从“活动” **** 工具箱拖到管道设计器图面。

    ![将 Notebook 拖至设计器图面](media/transform-data-using-databricks-notebook/new-adf-pipeline.png)

1.  在底部“Databricks Notebook” **** 活动窗口的属性中，完成以下步骤：

    a. 切换到“Azure Databricks” **** 选项卡。

    b. 选择  **AzureDatabricks\_LinkedService** （在上一过程中创建的）。

    c. 切换到“设置”选项卡

    c. 浏览以选择 Databricks **Notebook 路径**。 让我们在此处创建一个 Notebook 并指定路径。 可以通过后续数个步骤获取 Notebook 路径。

       1. 启动 Azure Databricks 工作区

       1. 在工作区中创建**新文件夹**，将其称之为 **adftutorial**。

          ![创建新文件夹](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image13.png)

       1. 在 adftutorial 文件夹下[创建新的笔记本](https://docs.databricks.com/user-guide/notebooks/index.html#creating-a-notebook) (Python) 并将其命名为 mynotebook，然后单击“创建”。

          ![创建新的 Notebook](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image14.png)

          ![设置新 Notebook 的属性](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image15.png)

       1. 在新创建的 Notebook“mynotebook”中添加以下代码：

           ```
           # Creating widgets for leveraging parameters, and printing the parameters

           dbutils.widgets.text("input", "","")
           dbutils.widgets.get("input")
           y = getArgument("input")
           print ("Param -\'input':")
           print (y)
           ```

           ![创建参数的小组件](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image16.png)

       1. 在此示例中，“Notebook 路径”为 **/adftutorial/mynotebook**

1.  切换回“数据工厂 UI 创作工具”。 导航到“Notebook1 活动”下的“设置”选项卡。

    a.  通过“添加参数”向 Notebook 活动添加参数。 使用的参数与此前添加到“管道”的参数相同。

       ![添加参数](media/transform-data-using-databricks-notebook/new-adf-parameters.png)

    b.  将参数命名为 input，并以 \@pipeline().parameters.name 表达式的形式提供值。

1.  若要验证管道，请选择工具栏中的“验证”按钮。 ****  若要关闭验证窗口，请选择  **\>\>** （向右键头）按钮。

    ![验证管道](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image18.png)

1.  选择“全部发布” ****。 数据工厂 UI 会将实体（链接服务和管道）发布到 Azure 数据工厂服务。

    ![发布新数据工厂实体](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image19.png)

## <a name="trigger-a-pipeline-run"></a>触发管道运行

选择工具栏中的“触发器”，然后选择“立即触发”。 ****  ****

![选择“立即触发”命令](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image20.png)

“管道运行”对话框询问 **name** 参数。 使用 **/path/filename** 作为此处的参数。 单击“完成”。

![提供一个值作为名称参数](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image21.png)

## <a name="monitor-the-pipeline-run"></a>监视管道运行

1.  切换到“监视器” **** 选项卡。 确认可以看到一个管道运行。 创建执行 Notebook 所在的 Databricks 作业群集需要大约 5-8 分钟。

    ![监视管道](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image22.png)

1.  定期选择“刷新”以检查管道运行的状态。 **** 

1.  若要查看与管道运行相关联的活动运行，请在“操作”列中选择“查看活动运行”。 **** **** 

    ![查看活动运行](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image23.png)

选择顶部的“管道”链接可以切换回到管道运行视图。 **** 

## <a name="verify-the-output"></a>验证输出

可以登录到“Azure Databricks 工作区”，转到“群集”，然后就可以看到“作业”状态为“待执行”、“正在运行”或“已终止”。

![查看作业群集和作业](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image24.png)

可以先单击“作业名称”，然后通过导航来查看更多详细信息。 成功运行后，即可验证传递的参数和 Python Notebook 的输出。

![查看运行详细信息和输出](media/transform-data-using-databricks-notebook/databricks-output.png)

## <a name="next-steps"></a>后续步骤

本示例中的管道先触发 Databricks Notebook 活动，然后向其传递参数。 你已了解如何：

  - 创建数据工厂。

  - 创建使用 Databricks Notebook 活动的管道。

  - 触发管道运行。

  - 监视管道运行。
