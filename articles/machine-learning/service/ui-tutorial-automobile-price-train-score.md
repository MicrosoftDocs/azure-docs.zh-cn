---
title: 教程：使用可视界面预测汽车价格
titleSuffix: Azure Machine Learning service
description: 了解如何使用拖放式可视界面来训练、评分和部署机器学习模型。 本教程是有关使用线性回归预测汽车价格的、由两个部分构成的教程系列的第一部分。
author: peterclu
ms.author: peterclu
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 04/06/2019
ms.openlocfilehash: 352f695f186b656ad0f6471d8ede905d53ad5274
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234098"
---
# <a name="tutorial-predict-automobile-price-with-the-visual-interface"></a>教程：使用可视界面预测汽车价格

本教程详细介绍如何在 Azure 机器学习服务可视界面中开发预测解决方案。 本教程结束时，你将获得一个可以根据发送的技术规范预测任何汽车的价格的解决方案。

本教程[延续快速入门教程](ui-quickstart-run-experiment.md)，是**由两个部分构成的教程系列的第一部分**。 但是，不需要完成快速入门即可开始学习本教程。

本教程系列的第一部分介绍如何：

> [!div class="checklist"]
> * 导入和清理数据（与快速入门中的步骤相同）
> * 训练机器学习模型
> * 评分和评估模型

本教程系列的[第二部分](ui-tutorial-automobile-price-deploy.md)介绍如何将预测模型部署为 Azure Web 服务。

> [!NOTE]
> 我们提供了本教程的已完成版本作为示例试验。
> 在“试验”页中，转到“新增” > “示例 1 - 回归:汽车价格预测(基本)”

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Build-zero-code-machine-learning-models-with-Azure-Machine-Learning-service/player]


## <a name="create-a-workspace"></a>创建工作区

如果你有一个 Azure 机器学习服务工作区，请跳至[下一部分](#open-the-visual-interface-webpage)。 否则，现在请创建一个工作区。

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

## <a name="open-the-visual-interface-webpage"></a>打开可视界面网页

1. 在 [Azure 门户](https://portal.azure.com/)中打开你的工作区。  

1. 在工作区中，选择“可视界面”。  然后选择“启动可视界面”。  

    ![Azure 门户的屏幕截图，其中显示了如何从机器学习服务工作区访问可视界面](./media/ui-tutorial-automobile-price-train-score/launch-ui.png)

    界面网页将在新的浏览器页中打开。  

## <a name="import-and-clean-your-data"></a>导入和清理数据

首先需要清理数据。 如果你已完成快速入门，可以重复使用其中所述的数据准备试验。 如果你尚未完成快速入门，请跳过下一部分并[从新的试验开始](#start-from-a-new-experiment)。

### <a name="reuse-the-quickstart-experiment"></a>重复使用快速入门试验

1. 打开快速入门试验。

1. 在窗口底部选择“另存为”。

1. 在弹出的对话框中为该试验指定新名称。

    ![显示如何将试验重命名为“教程 - 预测汽车价格”的屏幕截图](./media/ui-tutorial-automobile-price-train-score/save-a-copy.png)

1. 实验现在看起来应当与下图类似：

    ![显示试验预期状态的屏幕截图。 汽车数据集将连接到“选择列”模块，后者将连接到“清理缺失数据”](./media/ui-tutorial-automobile-price-train-score/save-copy-result.png)

如果你已成功重复使用快速入门试验，请跳过下一部分以开始[训练模型](#train-the-model)。

### <a name="start-from-a-new-experiment"></a>从新的试验开始

如果你尚未完成快速入门，请遵循以下步骤快速创建一个用于导入和清理汽车数据集的试验。

1. 在可视界面窗口的底部选择“+新建”以创建新的试验。

1. 选择“试验” >  “空白试验”。

1. 在画布顶部选择默认试验名称“... 创建的试验”，然后将它重命名为有意义的名称。 例如“汽车价格预测”。 名称不需唯一。

1. 试验画布左侧是数据集和模块的控制板。 若要查找模块，请使用模块控制板顶部的搜索框。 在搜索框中键入“汽车”，找到标有“汽车价格数据(原始)”的数据集。 将该数据集拖放到试验画布上。

    ![显示如何查找汽车价格数据集的屏幕截图](./media/ui-tutorial-automobile-price-train-score/automobile-dataset.png)

    找到数据后，可以添加一个用于彻底删除“规范化损失”列的模块。 然后添加用于删除任何存在缺失数据的行的另一个模块。

1. 在搜索框中键入“选择列”，找到“选择数据集中的列”模块。 然后将该模块拖放到试验画布上。 使用此模块可以选择要在模型中包含或排除哪些数据列。

1. 将“汽车价格数据(原始)”数据集的输出端口连接到“选择数据集中的列”模块的输入端口。

    ![显示如何将“汽车价格数据”模块连接到“选择列”模块的 Gif 动画](./media/ui-tutorial-automobile-price-train-score/connect-modules.gif)

1. 选择“选择数据集中的列”模块，然后在“属性”窗格中选择“启动列选择器”。

   1. 在左侧选择“使用规则”

   1. 在“开头为”旁边，选择“所有列”。 这些规则指示“选择数据集中的列”传递所有列（要排除的列除外）。

   1. 在下拉列表中，选择“排除”和**列名称**，然后在文本框中键入“规范化损失”。

   1. 选择“确定”按钮关闭列选择器（右下角）。

     此时“选择数据集中的列”的属性窗格指示它将传入数据集中的所有列，但“规范化损失”除外。

1. 双击“选择数据集中的列”模块并输入“排除规范化损失”，将注释添加到该模块。 这有助于快速查看模块在试验中的运行情况。

    ![显示“选择列”模块的正确配置的屏幕截图](./media/ui-tutorial-automobile-price-train-score/select-columns.png)

1. 在搜索框中键入“清理”，找到“清理缺失数据”模块。 将“清理缺失数据”模块拖到试验画布，然后将其连接到“选择数据集中的列”模块。

1. 在“属性”窗格的“清理模式”下选择“删除整行”。 这些选项指示“清理缺失数据”通过删除存在缺失值的行来清理数据。 双击该模块并键入注释“删除缺失值行”。

![显示“清理缺失数据”模块的正确配置的屏幕截图](./media/ui-tutorial-automobile-price-train-score/clean-missing-data.png)

## <a name="train-the-model"></a>训练模型

数据准备就绪后，可以构造一个预测模型。 你将使用自己的数据来训练模型。 然后将测试模型，以确定它预测价格的准确性。

**分类** 和**回归** 是两种监督式机器学习算法。 **分类**可以从一组定义的类别预测答案，例如颜色（红、蓝或绿）。 **回归**用于预测数字。

由于你要预测价格（一个数字），因此可以使用回归算法。 本示例将使用线性回归模型。

在模型中提供一组包含价格的数据以对其进行训练。 该模型会扫描数据，查找汽车特征与其价格之间的关联。 然后按以下方式测试模型：为模型提供一组熟悉的汽车特征，并确定该模型预测已知价格的准确性如何。

将数据拆分为单独的训练数据集和测试数据集，用于模型训练和测试。

1. 在搜索框中键入“拆分数据”找到“拆分数据”模块，然后将其连接到“清理缺失数据”模块的左端口。

1. 选择刚刚连接到的“拆分数据”模块。 在“属性”窗格中，将“第一个输出集中的行部分”设置为 0.7。 这样，我们将使用 70% 的数据来训练模型，保留 30% 的数据用于测试。

    ![显示属性窗格的正确配置的屏幕截图。 “拆分数据”的值应是“拆分行”、“0.7”、“随机拆分”、“0”、“False”。](./media/ui-tutorial-automobile-price-train-score/split-data.png)

1. 双击“拆分数据”并键入注释“将数据集拆分为训练集(0.7)和测试集(0.3)”

1. 若要选择学习算法，请清除模块控制板搜索框。

1. 依次展开“机器学习”、“初始化模型”。 此时会显示多个可用于初始化机器学习算法的模块类别。

1. 对于本试验，请选择“回归” > “线性回归”，并将该模块拖到试验画布上。

    ![显示属性窗格的正确配置的屏幕截图。 “拆分数据”的值应是“拆分行”、“0.7”、“随机拆分”、“0”、“False”。](./media/ui-tutorial-automobile-price-train-score/linear-regression-module.png)

1. 找到“训练模型”模块并将其拖到试验画布上。 将“线性回归”模块的输出连接到“训练模型”模块左侧的输入，将“拆分数据”模块的训练数据输出（左端口）连接到“训练模型”模块右侧的输入。

    ![显示“训练模型”模块的正确配置的屏幕截图。 “线性回归”模块将连接到“训练模型”模块的左端口，“拆分数据”模块将连接到“训练模型”的右端口](./media/ui-tutorial-automobile-price-train-score/train-model.png)

1. 选择“训练模型”模块。 在“属性”窗格中选择“启动列选择器”，然后在“包括列名称”的旁边键入**价格**。 价格是模型要预测的值。

    ![显示列选择器模块的正确配置的屏幕截图。 “使用规则”>“包括列名称”>“价格”](./media/ui-tutorial-automobile-price-train-score/select-price.png)

    现在，试验应如下所示：
    ![显示添加“训练模型”模块后试验的正确配置的屏幕截图。](./media/ui-tutorial-automobile-price-train-score/train-graph.png)

### <a name="run-the-training-experiment"></a>运行训练试验

[!INCLUDE [aml-ui-create-training-compute](../../../includes/aml-ui-create-training-compute.md)]

## <a name="score-and-evaluate-the-model"></a>评分和评估模型

使用 70% 的数据训练模型后，可以使用该模型为另外 30% 的数据评分，确定模型的运行情况。

1. 在搜索框中键入“评分模型”找到“评分模型”模块，并将该模块拖到试验画布上。 将“训练模型”模块的输出连接到“评分模型”的左侧输入端口。 将“拆分数据”模型的测试数据输出（右端口）连接到“评分模型”的右侧输入端口。

1. 在搜索框中键入“评估”找到“评估模型”，并将该模块拖到试验画布上。 将“评分模型”模块的输出连接到“评估模型”的左侧输入。 最终试验看起来应与下图类似：

    ![显示试验的最终正确配置的屏幕截图。](./media/ui-tutorial-automobile-price-train-score/final-graph.png)

1. 使用前面所用的同一计算目标运行试验。

1. 依次选择“评分模型”的输出端口和“可视化”来查看“评分模型”模块的输出。 输出显示价格预测值，以及来自测试数据的已知值。

    ![输出可视化效果的屏幕截图，其中突出显示了“评分标签”列](./media/ui-tutorial-automobile-price-train-score/score-result.png)

1. 要查看“评估模型”模块的输出，请选择输出端口，然后选择“可视化”。

    ![显示最终试验评估结果的屏幕截图。](./media/ui-tutorial-automobile-price-train-score/evaluate-result.png)

针对模型显示了以下统计信息：

* **平均绝对误差(MAE)**：绝对误差的平均值（误差是指预测值与实际值之间的差）。
* **均方根误差(RMSE)**：对测试数据集所做预测的平均误差的平方根。
* **相对绝对误差**：相对于实际值与所有实际值平均值之间的绝对差异的绝对误差平均值。
* **相对平方误差**：相对于实际值与所有实际值平均值之间的平方差异的平方误差平均值。
* **决定系数**：也称为 R 平方值，这是一个统计指标，表示模型的数据拟合度。

每个误差统计值越小越好。 值越小，表示预测越接近实际值。 对于决定系数，其值越接近 1 (1.0)，预测就越精确。

## <a name="manage-experiments-in-azure-machine-learning-service-workspace"></a>在 Azure 机器学习服务工作区中管理试验

可以通过 Azure 机器学习服务工作区管理在可视界面中创建的试验。 使用工作区可以查看个人试验运行、诊断日志、执行图等其他详细信息。

1. 在 [Azure 门户](https://portal.azure.com/)中打开你的工作区。  

1. 在工作区中选择“试验”。 然后选择创建的试验。

    ![显示如何在 Azure 门户中导航到试验的屏幕截图](./media/ui-tutorial-automobile-price-train-score/portal-experiments.png)

    此页会显示试验及其最新运行的概览。

    ![显示 Azure 门户中试验统计信息概览的屏幕截图](./media/ui-tutorial-automobile-price-train-score/experiment-overview.png)

1. 选择某个运行编号可以查看有关特定执行的更多详细信息。

    ![详细运行报告的屏幕截图](./media/ui-tutorial-automobile-price-train-score/run-details.png)

    运行报告会实时更新。 如果你在试验中使用了“执行 Python 脚本”模块，则可以在“日志”选项卡中指定要输出的脚本日志。

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [aml-ui-cleanup](../../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>后续步骤

在本教程系列的第一部分，你已完成以下步骤：

* 重复使用快速入门中创建的试验
* 准备数据
* 训练模型
* 评分和评估模型

第二部分介绍如何将模型部署为 Azure Web 服务。

> [!div class="nextstepaction"]
> [继续部署模型](ui-tutorial-automobile-price-deploy.md)
