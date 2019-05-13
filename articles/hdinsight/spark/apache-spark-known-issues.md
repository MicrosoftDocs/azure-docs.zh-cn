---
title: 排查 Azure HDInsight 中 Apache Spark 群集的相关问题
description: 了解与 Azure HDInsight 中的 Apache Spark 群集相关的问题以及如何解决这些问题。
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: hrasheed
ms.openlocfilehash: 51a0ee6f2d928d79e60ca9976d7651c70867a41f
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64717604"
---
# <a name="known-issues-for-apache-spark-cluster-on-hdinsight"></a>HDInsight 上的 Apache Spark 群集的已知问题

本文档记述了 HDInsight Spark 公共预览版的所有已知问题。  

## <a name="apache-livy-leaks-interactive-session"></a>Apache Livy 泄漏交互式会话
如果 [Apache Livy](https://livy.incubator.apache.org/) 在某个交互式会话仍保持活动状态的情况下重启（通过 [Apache Ambari](https://ambari.apache.org/) 重启或由于头节点 0 虚拟机重启导致），则会泄漏交互式作业会话。 因此，新作业可能会停滞在“已接受”状态。

**缓解：**

请使用以下步骤解决该问题：

1. 通过 SSH 连接到头节点。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 运行以下命令，以查找通过 Livy 启动的交互式作业的应用程序 ID。 
   
        yarn application –list
   
    如果在未指定显式名称的情况下通过 Livy 交互式对话启动作业，则默认的作业名称将为 Livy。 对于由 [Jupyter Notebook](https://jupyter.org/) 启动的 Livy 对话，作业名称以 remotesparkmagics_* 开头。 
3. 运行以下命令以终止这些作业。 
   
        yarn application –kill <Application ID>

新作业将开始运行。 

## <a name="spark-history-server-not-started"></a>Spark History Server未启动
创建群集后，Spark History Server 不自动启动。  

**缓解：** 

从 Ambari 手动启动 History Server。

## <a name="permission-issue-in-spark-log-directory"></a>Spark 日志目录中的权限问题
使用 spark-submit 提交作业时 hdiuser 会收到以下错误：

```
java.io.FileNotFoundException: /var/log/spark/sparkdriver_hdiuser.log (Permission denied)
```
并且不会写入任何驱动程序日志。 

**缓解：**

1. 将 hdiuser 添加到 Hadoop 组。 
2. 创建群集后，提供对 /var/log/spark 的 777 权限。 
3. 使用 Ambari 将 Spark 日志位置更新为具有 777 权限的目录。  
4. 以 sudo 身份运行 spark-submit。  

## <a name="spark-phoenix-connector-is-not-supported"></a>不支持 Spark-Phoenix 连接器

HDInsight Spark 群集不支持 Spark-Phoenix 连接器。

**缓解：**

必须改用 Spark-HBase 连接器。 相关说明请参阅[如何使用 Spark-HBase 连接器](https://web.archive.org/web/20190112153146/https://blogs.msdn.microsoft.com/azuredatalake/2016/07/25/hdinsight-how-to-use-spark-hbase-connector/)。

## <a name="issues-related-to-jupyter-notebooks"></a>Jupyter 笔记本的相关问题
下面是与 Jupyter 笔记本相关的一些已知问题。

### <a name="notebooks-with-non-ascii-characters-in-filenames"></a>笔记本的文件名中包含非 ASCII 字符
不要在 Jupyter Notebook 文件名中使用非 ASCII 字符。 如果尝试通过 Jupyter UI 来上传具有非 ASCII 文件名的文件，则上传将失败且不会显示任何错误消息。 Jupyter 不会让你上传文件，但是也不会引发可见的错误。

### <a name="error-while-loading-notebooks-of-larger-sizes"></a>加载大型笔记本时发生错误
加载大型笔记本时，可能会看到错误 **`Error loading notebook`**。  

**缓解：**

收到此错误并不表示数据已损坏或丢失。  笔记本仍在磁盘上的 `/var/lib/jupyter` 中，可以通过 SSH 连接到群集来访问它。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](../hdinsight-hadoop-linux-use-ssh-unix.md)。

一旦使用 SSH 连接到群集后，即可将笔记本从群集复制到本地计算机（使用 SCP 或 WinSCP）作为备份，以免丢失笔记本中的重要数据。 然后，可以使用端口 8001 通过 SSH 隧道（不通过网关）连接到头节点来访问 Jupyter。  可从该处清除笔记本的输出并将其重新保存，以尽量减小笔记本的大小。

若要防止今后发生此错误，必须遵循一些最佳实践：

* 必须保持较小的笔记本大小。 发回到 Jupyter 的所有 Spark 作业输出都将保存在笔记本中。  一般而言，Jupyter 的最佳用法是避免对大型 RDD 或数据帧运行 `.collect()`；相反，如果想要查看 RDD 的内容，请考虑运行 `.take()` 或 `.sample()`，以使输出不至于过大。
* 此外，在保存笔记本时，请清除所有输出单元以减小大小。

### <a name="notebook-initial-startup-takes-longer-than-expected"></a>笔记本初次启动花费的时间比预期要长
在使用 Spark magic 的 Jupyter 笔记本中，第一个代码语句可能需要花费一分钟以上。  

**解释：**

这发生在运行第一个代码单元时。 它在后台启动设置会话配置，以及设置 SQL、Spark 和 Hive 上下文。 设置这些上下文后，第一个语句才运行，因此让人觉得完成语句需要花费很长时间。

### <a name="jupyter-notebook-timeout-in-creating-the-session"></a>创建会话时 Jupyter 笔记本超时
如果 Spark 群集的资源不足，Jupyter 笔记本中的 Spark 和 PySpark 内核在尝试创建会话时会超时。 

**缓解措施：** 

1. 通过以下方式释放 Spark 群集中的一些资源：
   
   * 转到“关闭并停止”菜单或单击笔记本资源管理器中的“关闭”，以停止其他 Spark 笔记本。
   * 通过 YARN 停止其他 Spark 应用程序。
2. 重新启动先前尝试启动的笔记本。 现在应有足够的资源用于创建会话。

## <a name="see-also"></a>另请参阅
* [概述：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>方案
* [Apache Spark 与 BI：将 HDInsight 中的 Spark 与 BI 工具配合使用来执行交互式数据分析](apache-spark-use-bi-tools.md)
* [Apache Spark 与机器学习：使用 HDInsight 中的 Spark 来通过 HVAC 数据分析建筑物温度](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark 与机器学习：使用 HDInsight 中的 Spark 预测食品检验结果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Apache Spark 分析网站日志](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>创建和运行应用程序
* [使用 Scala 创建独立的应用程序](apache-spark-create-standalone-application.md)
* [使用 Apache Livy 在 Apache Spark 群集中远程运行作业](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和扩展
* [使用适用于 IntelliJ IDEA 的 HDInsight 工具插件创建和提交 Spark Scala 应用程序](apache-spark-intellij-tool-plugin.md)
* [使用适用于 IntelliJ IDEA 的 HDInsight 工具插件远程调试 Apache Spark 应用程序](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [在 HDInsight 上的 Apache Spark 群集中使用 Apache Zeppelin 笔记本](apache-spark-zeppelin-notebook.md)
* [在 HDInsight 的 Apache Spark 群集中可用于 Jupyter Notebook 的内核](apache-spark-jupyter-notebook-kernels.md)
* [Use external packages with Jupyter notebooks（将外部包与 Jupyter 笔记本配合使用）](apache-spark-jupyter-notebook-use-external-packages.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster（在计算机上安装 Jupyter 并连接到 HDInsight Spark 群集）](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理资源
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight（跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业）](apache-spark-job-debugging.md)

