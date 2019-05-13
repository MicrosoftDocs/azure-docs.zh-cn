---
title: '教程：使用 Power BI 在 Azure HDInsight 中分析 Apache Spark 数据 '
description: 使用 Microsoft Power BI 来可视化存储在 HDInsight 群集中的 Apache Spark 数据
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,mvc
ms.topic: tutorial
ms.date: 05/07/2018
ms.openlocfilehash: ed7beeadc0a550a28d1f936702aabeb45823b677
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64699607"
---
# <a name="tutorial-analyze-apache-spark-data-using-power-bi-in-hdinsight"></a>教程：使用 Power BI 在 HDInsight 中分析 Apache Spark 数据 

了解如何使用 [Microsoft Power BI](https://powerbi.microsoft.com/) 在 [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/) 中可视化 [Apache Spark](https://spark.apache.org/) 群集中的数据。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 使用 Power BI 可视化 Spark 数据

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

* **完成文章[教程：在 Azure HDInsight 中的 Apache Spark 群集上加载数据并运行查询](./apache-spark-load-data-run-query.md)**。
* **Power BI**：[Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) 和 [Power BI 试用订阅](https://app.powerbi.com/signupredirect?pbi_source=web)（可选）。


## <a name="verify-the-data"></a>验证数据

[上一教程](apache-spark-load-data-run-query.md)中创建的 [Jupyter Notebook](https://jupyter.org/) 包含用于创建 `hvac` 表的代码。 此表基于在所有 HDInsight Spark 群集上均可用的 CSV 文件（位于 \HdiSamples\HdiSamples\SensorSampleData\hvac\hvac.csv）。 使用以下过程验证数据。

1. 从 Jupyter 笔记本中，粘贴以下代码，然后按 Shift+Enter。 该代码验证表是否存在。

    ```PySpark
    %%sql
    SHOW TABLES
    ```

    输出如下所示：

    ![在 Spark 中显示表](./media/apache-spark-use-bi-tools/show-tables.png)

    如果在开始本教程之前关闭笔记本，这会清除 `hvactemptable`，使其不包含在输出中。  只有元存储中存储的 Hive 表（由“isTemporary”列下的“False”表示）可从 BI 工具中进行访问。 在本教程中，将连接到创建的 hvac 表。

2. 将以下代码粘贴到空白单元格中，然后按 Shift+Enter。 该代码验证表中的数据。

    ```PySpark
    %%sql
    SELECT * FROM hvac LIMIT 10
    ```

    输出如下所示：

    ![在 Spark 中显示 hvac 表中的行](./media/apache-spark-use-bi-tools/select-limit.png)

3. 请在 Notebook 的“文件”菜单中单击“关闭并停止”。 关闭笔记本以释放资源。 

## <a name="visualize-the-data"></a>可视化数据

在本部分，将使用 Power BI 从 Spark 群集数据创建可视化效果、报表和仪表板。 

### <a name="create-a-report-in-power-bi-desktop"></a>在 Power BI Desktop 中创建报表
使用 Spark 的前几步是连接到 Power BI Desktop 中的集群，从集群中加载数据，并根据此数据创建基本的可视化效果。

> [!NOTE]  
> 本文中所示的连接器当前处于预览状态。 通过 [Power BI 社区](https://community.powerbi.com/)站点或 [Power BI Ideas](https://ideas.powerbi.com/forums/265200-power-bi-ideas)（Power BI 想法）提供反馈。

1. 打开 [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/)。
1. 从“开始”选项卡上，单击“获取数据”，然后单击“更多”。

    ![从 HDInsight Apache Spark 中获取数据到 Power BI Desktop](./media/apache-spark-use-bi-tools/hdinsight-spark-power-bi-desktop-get-data.png "从 Apache Spark BI 中获取数据到 Power BI")


2. 在搜索框中输入 `Spark`，选择 Azure HDInsight Spark，然后单击“连接”。

    ![从 Apache Spark BI 中获取数据到 Power BI](./media/apache-spark-use-bi-tools/apache-spark-bi-import-data-power-bi.png "从 Apache Spark BI 中获取数据到 Power BI")

3. 输入群集 URL（格式为 `mysparkcluster.azurehdinsight.net`），选择 DirectQuery，然后单击“确定”。

    Spark 可使用任一数据连接模式。 若使用 DirectQuery，则报告中会显示更改，但不刷新整个数据集。 如果导入数据，则必须刷新数据集以查看更改。 若要深入了解如何以及何时使用 DirectQuery，请参阅[在 Power BI 中使用 DirectQuery](https://powerbi.microsoft.com/documentation/powerbi-desktop-directquery-about/)。 

4. 输入 HDInsight 登录帐户信息，然后单击“连接”。 默认帐户名为 admin。

5. 选择 `hvac` 表，稍后即可看到数据预览，然后单击“加载”。

    ![Spark 群集用户名和密码](./media/apache-spark-use-bi-tools/apache-spark-bi-select-table.png "Spark 群集用户名和密码")

    Power BI Desktop 拥有连接到 Spark 群集和从 `hvac` 表中加载数据所需的信息。 该表及表中各列显示在“字段”窗格中。  请参阅下面的屏幕截图：

6. 可视化每栋建筑物的目标温度与实际温度之间的差异： 

    1. 在“可视化”窗格中，选择“分区图”。 
    2. 将“BuildingID”字段拖到“轴”，并将“ActualTemp”和“TargetTemp”字段拖至“值”。

        ![使用 Apache Spark BI 创建 Spark 数据可视化](./media/apache-spark-use-bi-tools/apache-spark-bi-add-value-columns.png "使用 Apache Spark BI 创建 Spark 数据可视化")

        关系图如下所示：

        ![使用 Apache Spark BI 创建 Spark 数据可视化](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph-sum.png "使用 Apache Spark BI 创建 Spark 数据可视化")

        默认情况下，视觉效果会显示 **ActualTemp** 和 **TargetTemp** 的总和。 单击“可视化效果”窗格中 ActualTemp 和 TragetTemp 旁边的向下键，可看到已选中“总和”。

    3. 单击“可视化效果”窗格中 ActualTemp 和 TragetTemp 旁边的向下键，选择“平均值”以获得每栋建筑物的实际温度与目标温度的平均值。

        ![使用 Apache Spark BI 创建 Spark 数据可视化](./media/apache-spark-use-bi-tools/apache-spark-bi-average-of-values.png "使用 Apache Spark BI 创建 Spark 数据可视化")

        数据可视化效果应与屏幕截图中类似。 在视觉效果上移动光标可获取相关数据的工具提示。

        ![使用 Apache Spark BI 创建 Spark 数据可视化](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph.png "使用 Apache Spark BI 创建 Spark 数据可视化")

7. 依次单击“文件”和“保存”，再输入文件名 `BuildingTemperature.pbix` 。 

### <a name="publish-the-report-to-the-power-bi-service-optional"></a>将报表发布到 Power BI 服务（可选）

Power BI 服务允许在整个组织中共享报表和仪表板。 在本部分，首先发布数据集和报表。 然后，将报表固定到仪表板。 仪表板通常用于重点处理报表中的数据子集；此报表中只有一个可视化效果，但仍可用于完成此步骤。

1. 打开 Power BI Desktop。
2. 在“开始”选项卡上，单击“发布”。

    ![从 Power BI Desktop 发布](./media/apache-spark-use-bi-tools/apache-spark-bi-publish.png "从 Power BI Desktop 发布")

2. 选择要将数据集发布和报告到的工作区，然后单击“选择”。 在下图中，默认选择“我的工作区”。

    ![选择要将数据集发布和报告到的工作区](./media/apache-spark-use-bi-tools/apache-spark-bi-select-workspace.png "选择要将数据集发布和报告到的工作区") 

3. 发布成功后，单击“在 Power BI 中打开 BuildingTemperature.pbix”。

    ![发布成功，请单击输入凭据](./media/apache-spark-use-bi-tools/apache-spark-bi-publish-success.png "发布成功，请单击输入凭据") 

4. 在 Power BI 服务中，单击“输入凭据”。

    ![在 Power BI 服务中输入凭据](./media/apache-spark-use-bi-tools/apache-spark-bi-enter-credentials.png "在 Power BI 服务中输入凭据")

5. 单击“编辑凭据”。

    ![在 Power BI 服务中编辑凭据](./media/apache-spark-use-bi-tools/apache-spark-bi-edit-credentials.png "在 Power BI 服务中编辑凭据")

6. 输入 HDInsight 登录帐户信息，然后单击“登录”。 默认帐户名为 admin。

    ![登录 Spark 群集](./media/apache-spark-use-bi-tools/apache-spark-bi-sign-in.png "登录 Spark 群集")

7. 在左窗格中，转到“工作区” > “我的工作区” > “报表”，然后单击 BuildingTemperature。

    ![报表位于左窗格的报表下](./media/apache-spark-use-bi-tools/apache-spark-bi-service-left-pane.png "报表位于左窗格的报表下")

    还可在左窗格中的“数据集”下找到 BuildingTemperature。

    现可在 Power BI 服务中使用 Power BI Desktop 中创建的视觉对象。 

8. 将游标悬停在可视化效果上，然后单击右上角的固定图标。

    ![Power BI 服务中的报表](./media/apache-spark-use-bi-tools/apache-spark-bi-service-report.png "Power BI 服务中的报表")

9. 选择“新的仪表板”，输入名称 `Building temperature`，再单击“固定”。

    ![固定到新的仪表板](./media/apache-spark-use-bi-tools/apache-spark-bi-pin-dashboard.png "固定到新仪表板")

10. 在报表中，单击“转到仪表板”。 

视觉对象已固定到仪表板 - 可在报表中添加其他视觉对象，并将其固定在同一仪表板上。 有关报表和仪表板的详细信息，请参阅 [Power BI 中的报表](https://powerbi.microsoft.com/documentation/powerbi-service-reports/)和 [Power BI 中的仪表板](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/)。

<!--
## <a name="tableau"></a>Use Tableau Desktop 

> [!NOTE]
> This section is applicable only for Spark 1.5.2 clusters created in Azure HDInsight.
>
>

1. Install [Tableau Desktop](https://www.tableau.com/products/desktop) on the computer where you are running this Apache Spark BI tutorial.

2. Make sure that computer also has Microsoft Spark ODBC driver installed. You can install the driver from [here](https://go.microsoft.com/fwlink/?LinkId=616229).

1. Launch Tableau Desktop. In the left pane, from the list of server to connect to, click **Spark SQL**. If Spark SQL is not listed by default in the left pane, you can find it by click **More Servers**.
2. In the Spark SQL connection dialog box, provide the values as shown in the screenshot, and then click **OK**.

    ![Connect to a cluster for Apache Spark BI](./media/apache-spark-use-bi-tools/connect-to-tableau-apache-spark-bi.png "Connect to a cluster for Apache Spark BI")

    The authentication drop-down lists **Microsoft Azure HDInsight Service** as an option, only if you installed the [Microsoft Spark ODBC Driver](https://go.microsoft.com/fwlink/?LinkId=616229) on the computer.
3. On the next screen, from the **Schema** drop-down, click the **Find** icon, and then click **default**.

    ![Find schema for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-find-schema-apache-spark-bi.png "Find schema for Apache Spark BI")
4. For the **Table** field, click the **Find** icon again to list all the Hive tables available in the cluster. You should see the **hvac** table you created earlier using the notebook.

    ![Find table for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-find-table-apache-spark-bi.png "Find table for Apache Spark BI")
5. Drag and drop the table to the top box on the right. Tableau imports the data and displays the schema as highlighted by the red box.

    ![Add tables to Tableau for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-add-table-apache-spark-bi.png "Add tables to Tableau for Apache Spark BI")
6. Click the **Sheet1** tab at the bottom left. Make a visualization that shows the average target and actual temperatures for all buildings for each date. Drag **Date** and **Building ID** to **Columns** and **Actual Temp**/**Target Temp** to **Rows**. Under **Marks**, select **Area** to use an area map for Spark data visualization.

     ![Add fields for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-add-fields.png "Add fields for Spark data visualization")
7. By default, the temperature fields are shown as aggregate. If you want to show the average temperatures instead, you can do so from the drop-down, as shown in the following screenshot:

    ![Take average of temperature for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-average-temperature.png "Take average of temperature for Spark data visualization")

8. You can also super-impose one temperature map over the other to get a better feel of difference between target and actual temperatures. Move the mouse to the corner of the lower area map until you see the handle shape highlighted in a red circle. Drag the map to the other map on the top and release the mouse when you see the shape highlighted in red rectangle.

    ![Merge maps for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-merge-maps.png "Merge maps for Spark data visualization")

     Your data visualization should change as shown in the screenshot:

    ![Tableau output for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-tableau-output.png "Tableau output for Spark data visualization")
9. Click **Save** to save the worksheet. You can create dashboards and add one or more sheets to it.
-->

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

- 使用 Power BI 可视化 Apache Spark 数据。

请前进到下一篇文章，了解如何将在 Spark 中注册的数据拉取到 Power BI 等 BI 分析工具中。 
> [!div class="nextstepaction"]
> [运行 Apache Spark 流式处理作业](apache-spark-eventhub-streaming.md)

