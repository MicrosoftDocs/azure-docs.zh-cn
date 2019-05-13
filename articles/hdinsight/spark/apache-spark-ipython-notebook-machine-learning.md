---
title: 教程：在 Azure HDInsight 中生成 Spark 机器学习应用程序
description: 有关如何使用 Jupyter Notebook 在 HDInsight Spark 群集中生成 Apache Spark 机器学习应用程序的分步说明。
ms.service: hdinsight
author: hrasheed-msft
ms.reviewer: jasonh
ms.custom: hdinsightactive,mvc
ms.topic: tutorial
ms.date: 11/06/2018
ms.author: hrasheed
ms.openlocfilehash: 87efac96aa0120bfcc804f7a2a49a5ac3da1036b
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64719064"
---
# <a name="tutorial-build-an-apache-spark-machine-learning-application-in-hdinsight"></a>教程：在 HDInsight 中生成 Apache Spark 机器学习应用程序 

本教程介绍如何使用 [Jupyter Notebook](https://jupyter.org/) 生成适用于 Azure HDInsight 的 [Apache Spark](https://spark.apache.org/) 机器学习应用程序。 

[MLib](https://spark.apache.org/docs/1.1.0/mllib-guide.html) 是 Spark 的可缩放机器学习库，由常见学习算法和实用工具（包括分类、回归、聚集、协作筛选、维数约简以及底层优化基元）组成。

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 开发 Apache Spark 机器学习应用程序

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件：

必须具备以下项：

* 完成[在 Azure HDInsight 中创建 Apache Spark 群集](apache-spark-jupyter-spark-sql.md)。

## <a name="understand-the-data-set"></a>了解数据集

应用程序使用所有群集默认提供的示例 HVAC.csv 数据。 该文件位于 \HdiSamples\HdiSamples\SensorSampleData\hvac。 数据显示了安装有 HVAC 系统的一些建筑物的目标温度和实际温度。 **System** 列代表系统 ID，**SystemAge** 列代表建筑物安装 HVAC 系统的年数。 在指定系统 ID 和系统年数的情况下，可使用这些数据来预测建筑物的温度比目标温度高还是低。

![用于 Spark 机器学习示例的数据的快照](./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-understand-data.png "用于 Spark 机器学习示例的数据的快照")

## <a name="develop-a-spark-machine-learning-application-using-spark-mllib"></a>使用 Spark MLlib 开发 Spark 机器学习应用程序

在此应用程序中，将使用 Spark [ML 管道](https://spark.apache.org/docs/2.2.0/ml-pipeline.html)来执行文档分类。 ML 管道提供了一组统一的高级 API，它们构建在数据帧之上，可帮助用户创建和优化实际机器学习管道。 在管道中，可将文档分割成单字、将单字转换成数字特征向量，最后使用特征向量和标签创建预测模型。 执行下列步骤创建应用程序。

1. 使用 PySpark 内核创建 Jupyter Notebook。 有关说明，请参阅[创建 Jupyter Notebook](./apache-spark-jupyter-spark-sql.md#create-a-jupyter-notebook)。
2. 导入此方案所需的类型。 将以下代码段粘贴到空白单元格中，然后按 **Shift+Enter**。 

    ```PySpark
    from pyspark.ml import Pipeline
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml.feature import HashingTF, Tokenizer
    from pyspark.sql import Row

    import os
    import sys
    from pyspark.sql.types import *

    from pyspark.mllib.classification import LogisticRegressionWithSGD
    from pyspark.mllib.regression import LabeledPoint
    from numpy import array
    ```
3. 加载数据 (hvac.csv)，分析数据，并使用它来训练模型。 

    ```PySpark
    # Define a type called LabelDocument
    LabeledDocument = Row("BuildingID", "SystemInfo", "label")

    # Define a function that parses the raw CSV file and returns an object of type LabeledDocument
    def parseDocument(line):
        values = [str(x) for x in line.split(',')]
        if (values[3] > values[2]):
            hot = 1.0
        else:
            hot = 0.0        

        textValue = str(values[4]) + " " + str(values[5])

        return LabeledDocument((values[6]), textValue, hot)

    # Load the raw HVAC.csv file, parse it using the function
    data = sc.textFile("/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

    documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
    training = documents.toDF()
    ```

    在此代码片段中，定义用于比较实际温度与目标温度的函数。 如果实际温度较高，则表示建筑物处于高温状态，以值 **1.0** 表示。 否则建筑物处于低温状态，用值 **0.0** 表示。 

4. 设置包括三个阶段的 Spark 机器学习管道：tokenizer、hashingTF 和 lr。 

    ```PySpark
    tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
    hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
    lr = LogisticRegression(maxIter=10, regParam=0.01)
    pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])
    ```

    有关管道及其工作原理的详细信息，请参阅 <a href="https://spark.apache.org/docs/latest/ml-guide.html#how-it-works" target="_blank">Apache Spark 机器学习管道</a>。

5. 将管道拟合到培训文档中。
   
    ```PySpark
    model = pipeline.fit(training)
    ```

6. 验证训练文档以根据应用程序的进度创建检查点。
   
    ```PySpark
    training.show()
    ```
   
    输出类似于：

    ```
    +----------+----------+-----+
    |BuildingID|SystemInfo|label|
    +----------+----------+-----+
    |         4|     13 20|  0.0|
    |        17|      3 20|  0.0|
    |        18|     17 20|  1.0|
    |        15|      2 23|  0.0|
    |         3|      16 9|  1.0|
    |         4|     13 28|  0.0|
    |         2|     12 24|  0.0|
    |        16|     20 26|  1.0|
    |         9|      16 9|  1.0|
    |        12|       6 5|  0.0|
    |        15|     10 17|  1.0|
    |         7|      2 11|  0.0|
    |        15|      14 2|  1.0|
    |         6|       3 2|  0.0|
    |        20|     19 22|  0.0|
    |         8|     19 11|  0.0|
    |         6|      15 7|  0.0|
    |        13|      12 5|  0.0|
    |         4|      8 22|  0.0|
    |         7|      17 5|  0.0|
    +----------+----------+-----+
    ```

    将输出与原始 CSV 文件进行比较。 例如，CSV 文件中第一行包含此数据：

    ![Spark 机器学习示例的输出数据快照](./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-output-data.png "Spark 机器学习示例的输出数据快照")

    请注意，实际温度比目标温度低的情况表示建筑物处于低温状态。 因此在训练输出中，第一行中的 **label** 值为 **0.0**，表示建筑物并非处于高温状态。

7. 准备要对其运行已训练模型的数据集。 为此，将传递系统 ID 和系统年数（以训练输出中的 **SystemInfo** 表示），模型将预测具有该系统 ID 和系统年数的建筑物的温度是较高（以 1.0 表示）还是较低（以 0.0 表示）。
   
    ```PySpark   
    # SystemInfo here is a combination of system ID followed by system age
    Document = Row("id", "SystemInfo")
    test = sc.parallelize([(1L, "20 25"),
                    (2L, "4 15"),
                    (3L, "16 9"),
                    (4L, "9 22"),
                    (5L, "17 10"),
                    (6L, "7 22")]) \
        .map(lambda x: Document(*x)).toDF() 
    ```
8. 最后，对测试数据进行预测。 
   
    ```PySpark
    # Make predictions on test documents and print columns of interest
    prediction = model.transform(test)
    selected = prediction.select("SystemInfo", "prediction", "probability")
    for row in selected.collect():
        print row
    ```

    输出类似于：

    ```   
    Row(SystemInfo=u'20 25', prediction=1.0, probability=DenseVector([0.4999, 0.5001]))
    Row(SystemInfo=u'4 15', prediction=0.0, probability=DenseVector([0.5016, 0.4984]))
    Row(SystemInfo=u'16 9', prediction=1.0, probability=DenseVector([0.4785, 0.5215]))
    Row(SystemInfo=u'9 22', prediction=1.0, probability=DenseVector([0.4549, 0.5451]))
    Row(SystemInfo=u'17 10', prediction=1.0, probability=DenseVector([0.4925, 0.5075]))
    Row(SystemInfo=u'7 22', prediction=0.0, probability=DenseVector([0.5015, 0.4985]))
    ```
   
   从预测中的第一行可以看出，对于 ID 为 20 且系统年数为 25 的 HVAC 系统，建筑物处于高温状态 (**prediction=1.0**)。 DenseVector (0.49999) 的第一个值对应于预测 0.0，第二个值 (0.5001) 对应于预测 1.0。 在输出中，即使第二个值只稍高一点，模型也仍旧显示 **prediction=1.0**。
10. 关闭笔记本以释放资源。 为此，请在 Notebook 的“文件”菜单中选择“关闭并停止”。 此操作会关闭 Notebook。

## <a name="use-anaconda-scikit-learn-library-for-spark-machine-learning"></a>将 Anaconda scikit-learn 库用于 Spark 机器学习
HDInsight 中的 Apache Spark 群集包含 Anaconda 库。 它还包括适用于机器学习的 scikit-learn 库。 该库还包含用于直接从 Jupyter 笔记本生成示例应用程序的各种数据集。 有关使用 scikit-learn 库的示例，请参阅 [https://scikit-learn.org/stable/auto_examples/index.html](https://scikit-learn.org/stable/auto_examples/index.html)。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

* 开发 Apache Spark 机器学习应用程序

继续学习下一教程，了解如何将 IntelliJ IDEA 用于 Spark 作业。 

> [!div class="nextstepaction"]
> [使用 IntelliJ 创建 Scala Maven 应用程序](./apache-spark-create-standalone-application.md)

