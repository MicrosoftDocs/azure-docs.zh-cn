---
title: 将 HBase 群集迁移到新版本 - Azure HDInsight
description: 如何将 HBase 群集迁移到新版本。
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: ashishth
ms.openlocfilehash: a152b815daeefa4c199af9b159eee8e5783971e2
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143320"
---
# <a name="migrate-an-apache-hbase-cluster-to-a-new-version"></a>将 Apache HBase 群集迁移到新版本

本文介绍更新到较新版本的 Azure HDInsight 上的 Apache HBase 群集所需的步骤。

> [!NOTE]  
> 升级造成的停机时间应该很短，以分钟计。 停机是执行刷新所有内存中数据的步骤，然后在新群集上配置和重启服务造成的。 根据节点数目、数据量和其他变数，结果会有所不同。

## <a name="review-apache-hbase-compatibility"></a>检查 Apache HBase 兼容性

在升级 Apache HBase 之前，请确保源群集和目标群集上的 HBase 版本兼容。 有关详细信息，请参阅 [Apache HDInsight 提供的 Hadoop 组件和版本](../hdinsight-component-versioning.md)。

> [!NOTE]  
> 我们强烈建议查看 [HBase 书册](https://hbase.apache.org/book.html#upgrading)中的版本兼容性矩阵。

下面是示例版本兼容性矩阵。 Y 表示兼容性，N 表示潜在的不兼容：

| 兼容性类型 | 主版本| 次版本 | 修补程序 |
| --- | --- | --- | --- |
| 客户端-服务器网络兼容性 | N | Y | Y |
| 服务器-服务器兼容性 | N | Y | Y |
| 文件格式兼容性 | N | Y | Y |
| 客户端 API 兼容性 | N | Y | Y |
| 客户端二进制文件兼容性 | N | N | Y |
| **服务器端受限的 API 兼容性** |  |  |  |
| Stable | N | Y | Y |
| 不断变化 | N | N | Y |
| 不稳定 | N | N | N |
| 依赖项兼容性 | N | Y | Y |
| 操作兼容性 | N | N | Y |

> [!NOTE]  
> HBase 版本发行说明中应会阐述任何重大的不兼容性。

## <a name="upgrade-with-same-apache-hbase-major-version"></a>使用相同的 Apache HBase 主版本升级

若要在 Azure HDInsight 上的将 Apache HBase 群集升级，完成以下步骤：

1. 请确保应用程序与新版本兼容，如 HBase 兼容性矩阵和发行说明中所述。 在运行 HDInsight 和 HBase 目标版本的群集中测试应用程序。

2. 使用相同的存储帐户、不同的容器名称[设置新的目标 HDInsight 群集](../hdinsight-hadoop-provision-linux-clusters.md)：

    ![使用相同的存储帐户，但创建不同的容器](./media/apache-hbase-migrate-new-version/same-storage-different-container.png)

3. 刷新源 HBase 群集，这是要升级的群集。 HBase 将传入的数据写入名为 _memstore_ 的内存中存储。 Memstore 达到特定大小后，HBase 刷新到磁盘以长期存储在群集的存储帐户。 删除旧群集时，将回收 memstores，这可能会丢失数据。 若要将每个表的 memstore 手动刷新到磁盘，请运行以下脚本。 Azure 的 [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/flush_all_tables.sh) 中提供了此脚本的最新版本。

    ```bash
    #!/bin/bash
    
    #-------------------------------------------------------------------------------#
    # SCRIPT TO FLUSH ALL HBASE TABLES.
    #-------------------------------------------------------------------------------#
    
    LIST_OF_TABLES=/tmp/tables.txt
    HBASE_SCRIPT=/tmp/hbase_script.txt
    TARGET_HOST=$1
    
    usage ()
    {
        if [[ "$1" == "-h" ]] || [[ "$1" == "--help" ]]
        then
            cat << ...
    
    Usage: 
    
    $0 [hostname]
    
    Providing hostname is optional and not required when the script is executed within HDInsight cluster with access to 'hbase shell'.
    
    However hostname should be provided when executing the script as a script-action from HDInsight portal.
    
    For Example:
    
        1.  Executing script inside HDInsight cluster (where 'hbase shell' is 
            accessible):
    
            $0 
    
            [No need to provide hostname]
    
        2.  Executing script from HDinsight Azure portal:
    
            Provide Script URL.
    
            Provide hostname as a parameter (i.e. hn0, hn1 or wn2 etc.).
    ...
            exit
        fi
    }
    
    validate_machine ()
    {
        THIS_HOST=`hostname`
    
        if [[ ! -z "$TARGET_HOST" ]] && [[ $THIS_HOST  != $TARGET_HOST* ]]
        then
            echo "[INFO] This machine '$THIS_HOST' is not the right machine ($TARGET_HOST) to execute the script."
            exit 0
        fi
    }
    
    get_tables_list ()
    {
    hbase shell << ... > $LIST_OF_TABLES 2> /dev/null
        list
        exit
    ...
    }
    
    add_table_for_flush ()
    {
        TABLE_NAME=$1
        echo "[INFO] Adding table '$TABLE_NAME' to flush list..."
        cat << ... >> $HBASE_SCRIPT
            flush '$TABLE_NAME'
    ...
    }
    
    clean_up ()
    {
        rm -f $LIST_OF_TABLES
        rm -f $HBASE_SCRIPT
    }
    
    ########
    # MAIN #
    ########
    
    usage $1
    
    validate_machine
    
    clean_up
    
    get_tables_list
    
    START=false
    
    while read LINE 
    do 
        if [[ $LINE == TABLE ]] 
        then
            START=true
            continue
        elif [[ $LINE == *row*in*seconds ]]
        then
            break
        elif [[ $START == true ]]
        then
            add_table_for_flush $LINE
        fi
    
    done < $LIST_OF_TABLES
    
    cat $HBASE_SCRIPT
    
    hbase shell $HBASE_SCRIPT << ... 2> /dev/null
    exit
    ...
    
    ```
    
4. 停止引入到旧 HBase 群集。
5. 为确保刷新 memstore 中的所有最新数据，请再次运行前面的脚本。
6. 登录到[Apache Ambari](https://ambari.apache.org/)旧群集上 (https://OLDCLUSTERNAME.azurehdidnsight.net)并停止 HBase 服务。 当系统提示您确认要停止服务时，选中复选框为 HBase 启用维护模式。 有关连接和使用 Ambari 的详细信息，请参阅[使用 Ambari Web UI 管理 HDInsight 群集](../hdinsight-hadoop-manage-ambari.md)。

    ![在 Ambari 中，单击服务 > HBase > 服务操作下停止](./media/apache-hbase-migrate-new-version/stop-hbase-services.png)

    ![选中“为 HBase 启用维护模式”复选框，然后确认](./media/apache-hbase-migrate-new-version/turn-on-maintenance-mode.png)

7. 新的 HDInsight 群集上登录到 Ambari。 将 `fs.defaultFS` HDFS 设置更改为指向原始群集所用的容器名称。 此设置位于“HDFS”>“配置”>“高级”>“高级 core-site”下。

    ![在 Ambari 中，单击服务 > HDFS > 配置 > 高级](./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png)

    ![在 Ambari 中更改容器名称](./media/apache-hbase-migrate-new-version/change-container-name.png)

8. **如果不使用增强的写入功能使用 HBase 群集，跳过此步骤。它只需要具有增强的写入功能的 HBase 群集。**
   
   更改`hbase.rootdir`路径以指向原始群集的容器。

    ![在 Ambari 中，更改 HBase rootdir 的容器名称](./media/apache-hbase-migrate-new-version/change-container-name-for-hbase-rootdir.png)
1. 如果您要为 4.0 升级 HDInsight 3.6，请按照以下步骤，否则请跳到步骤 10:
    1. 重新启动所需的所有服务在 Ambari 中选择**Services** > **重新启动所有必需**。
    1. 停止 HBase 服务。
    1. 通过 ssh 连接到 Zookeeper 节点，并执行[zkCli](https://github.com/go-zkcli/zkcli)命令`rmr /hbase-unsecure`从 Zookeeper 中删除 HBase 根 znode。
    1. 重启 HBase。
1. 如果您要升级到 4.0 除了任何其他 HDInsight 版本，请按照下列步骤：
    1. 保存所做更改。
    1. 根据 Ambari 中的指示重启全部所需的服务。
1. 将应用程序指向新群集。

    > [!NOTE]  
    > 升级时，应用程序的静态 DNS 会更改。 不要硬编码此 DNS，可以在域名的 DNS 设置中配置一个指向群集名称的 CNAME。 另一种做法是使用应用程序的、无需重新部署即可更新的配置文件。

12. 启动引入，确定一切是否按预期正常运行。
13. 如果新群集符合预期，请删除原始群集。

## <a name="next-steps"></a>后续步骤

若要详细了解 [Apache HBase](https://hbase.apache.org/) 以及如何升级 HDInsight 群集，请参阅以下文章：

* [将 HDInsight 群集升级到更新版本](../hdinsight-upgrade-cluster.md)
* [使用 Apache Ambari Web UI 监视和管理 Azure HDInsight](../hdinsight-hadoop-manage-ambari.md)
* [Apache Hadoop 组件和版本](../hdinsight-component-versioning.md)
* [使用 Apache Ambari 优化配置](../hdinsight-changing-configs-via-ambari.md#apache-hbase-optimization-with-the-ambari-web-ui)
