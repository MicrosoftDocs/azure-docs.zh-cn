---
title: 使用 Azure HDInsight 对 HBase 进行故障排除
description: 获取有关使用 HBase 和 Azure HDInsight 的常见问题的解答。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.custom: hdinsightactive, seodec18
ms.topic: conceptual
ms.date: 12/06/2018
ms.openlocfilehash: 6ba17a3839390ed5fe503a6fe57b63d8fb119138
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64713492"
---
# <a name="troubleshoot-apache-hbase-by-using-azure-hdinsight"></a>使用 Azure HDInsight 对 Apache HBase 进行故障排除

了解处理 Apache Ambari 中的 Apache HBase 有效负载时的最常见问题及其解决方法。

## <a name="how-do-i-run-hbck-command-reports-with-multiple-unassigned-regions"></a>如何对多个未分配区域运行 hbck 命令报告？

运行 `hbase hbck` 命令时，可能会出现的一条常见错误消息是“多个区域未分配，或者区域链中出现漏洞”。

在 HBase Master UI 中，可以看到所有区域服务器中非均衡区域的数目。 然后，可以运行 `hbase hbck` 命令查看区域链中的漏洞。

漏洞可能是脱机区域造成的，因此请先修复分配问题。 

若要使未分配的区域恢复正常状态，请完成以下步骤：

1. 使用 SSH 登录到 HDInsight HBase 群集。
2. 若要与 Apache ZooKeeper shell 进行连接，请运行 `hbase zkcli` 命令。
3. 运行 `rmr /hbase/regions-in-transition` 命令或 `rmr /hbase-unsecure/regions-in-transition` 命令。
4. 若要从 `hbase zkcli` shell 退出，请使用 `exit` 命令。
5. 打开 Apache Ambari UI，并重启 Active HBase Master 服务。
6. 再次运行 `hbase hbck` 命令（不带任何选项）。 检查此命令的输出以确保分配了所有区域。


## <a name="how-do-i-fix-timeout-issues-with-hbck-commands-for-region-assignments"></a>使用 hbck 命令进行区域分配时，如何解决超时问题？

### <a name="issue"></a>问题

使用 `hbck` 命令时出现超时问题的可能原因是多个区域长时间处于“正在转换”状态。 在 HBase Master UI 中可以看到这些区域显示为脱机。 由于有大量区域正在尝试进行转换，因此，HBase Master 可能会超时并且无法使那些区域恢复联机。

### <a name="resolution-steps"></a>解决步骤

1. 使用 SSH 登录到 HDInsight HBase 群集。
2. 若要与 Apache ZooKeeper shell 进行连接，请运行 `hbase zkcli` 命令。
3. 运行 `rmr /hbase/regions-in-transition` 或 `rmr /hbase-unsecure/regions-in-transition` 命令。
4. 若要退出 `hbase zkcli` shell，请使用 `exit` 命令。
5. 在 Ambari UI 中，重启 Active HBase Master 服务。
6. 再次运行命令 `hbase hbck -fixAssignments`。

## <a name="how-do-i-force-disable-hdfs-safe-mode-in-a-cluster"></a>如何在群集中强制禁用 HDFS 安全模式？

### <a name="issue"></a>问题

本地 Apache Hadoop 分布式文件系统 (HDFS) 在 HDInsight 群集上的安全模式下停止响应。

### <a name="detailed-description"></a>详细说明

此错误的原因可能是运行以下 HDFS 命令时发生失败：

```apache
hdfs dfs -D "fs.default.name=hdfs://mycluster/" -mkdir /temp
```

尝试运行该命令时可能会出现的错误如下所示：

```apache
hdiuser@hn0-spark2:~$ hdfs dfs -D "fs.default.name=hdfs://mycluster/" -mkdir /temp
17/04/05 16:20:52 WARN retry.RetryInvocationHandler: Exception while invoking ClientNamenodeProtocolTranslatorPB.mkdirs over hn0-spark2.2oyzcdm4sfjuzjmj5dnmvscjpg.dx.internal.cloudapp.net/10.0.0.22:8020. Not retrying because try once and fail.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /temp. Name node is in safe mode.
It was turned on manually. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1359)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:4010)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:1102)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:630)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:640)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2313)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2309)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1724)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2307)
        at org.apache.hadoop.ipc.Client.getRpcResponse(Client.java:1552)
        at org.apache.hadoop.ipc.Client.call(Client.java:1496)
        at org.apache.hadoop.ipc.Client.call(Client.java:1396)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:233)
        at com.sun.proxy.$Proxy10.mkdirs(Unknown Source)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.mkdirs(ClientNamenodeProtocolTranslatorPB.java:603)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:278)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:194)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:176)
        at com.sun.proxy.$Proxy11.mkdirs(Unknown Source)
        at org.apache.hadoop.hdfs.DFSClient.primitiveMkdir(DFSClient.java:3061)
        at org.apache.hadoop.hdfs.DFSClient.mkdirs(DFSClient.java:3031)
        at org.apache.hadoop.hdfs.DistributedFileSystem$24.doCall(DistributedFileSystem.java:1162)
        at org.apache.hadoop.hdfs.DistributedFileSystem$24.doCall(DistributedFileSystem.java:1158)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirsInternal(DistributedFileSystem.java:1158)
        at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirs(DistributedFileSystem.java:1150)
        at org.apache.hadoop.fs.FileSystem.mkdirs(FileSystem.java:1898)
        at org.apache.hadoop.fs.shell.Mkdir.processNonexistentPath(Mkdir.java:76)
        at org.apache.hadoop.fs.shell.Command.processArgument(Command.java:273)
        at org.apache.hadoop.fs.shell.Command.processArguments(Command.java:255)
        at org.apache.hadoop.fs.shell.FsCommand.processRawArguments(FsCommand.java:119)
        at org.apache.hadoop.fs.shell.Command.run(Command.java:165)
        at org.apache.hadoop.fs.FsShell.run(FsShell.java:297)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:90)
        at org.apache.hadoop.fs.FsShell.main(FsShell.java:350)
mkdir: Cannot create directory /temp. Name node is in safe mode.
```

### <a name="probable-cause"></a>可能的原因

HDInsight 群集已减少到很少的节点。 节点数低于或接近于 HDFS 复制因子。

### <a name="resolution-steps"></a>解决步骤 

1. 运行以下命令获取 HDInsight 群集上的 HDFS 状态：

   ```apache
   hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -report
   ```

   ```apache
   hdiuser@hn0-spark2:~$ hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -report
   Safe mode is ON
   Configured Capacity: 3372381241344 (3.07 TB)
   Present Capacity: 3138625077248 (2.85 TB)
   DFS Remaining: 3102710317056 (2.82 TB)
   DFS Used: 35914760192 (33.45 GB)
   DFS Used%: 1.14%
   Under replicated blocks: 0
   Blocks with corrupt replicas: 0
   Missing blocks: 0
   Missing blocks (with replication factor 1): 0

   -------------------------------------------------
   Live datanodes (8):

   Name: 10.0.0.17:30010 (10.0.0.17)
   Hostname: 10.0.0.17
   Decommission Status : Normal
   Configured Capacity: 421547655168 (392.60 GB)
   DFS Used: 5288128512 (4.92 GB)
   Non DFS Used: 29087272960 (27.09 GB)
   DFS Remaining: 387172253696 (360.58 GB)
   DFS Used%: 1.25%
   DFS Remaining%: 91.85%
   Configured Cache Capacity: 0 (0 B)
   Cache Used: 0 (0 B)
   Cache Remaining: 0 (0 B)
   Cache Used%: 100.00%
   Cache Remaining%: 0.00%
   Xceivers: 2
   Last contact: Wed Apr 05 16:22:00 UTC 2017
   ...

   ```
2. 还可以使用以下命令检查 HDInsight 群集上的 HDFS 完整性：

   ```apache
   hdiuser@hn0-spark2:~$ hdfs fsck -D "fs.default.name=hdfs://mycluster/" /
   ```

   ```apache
   Connecting to namenode via http://hn0-spark2.2oyzcdm4sfjuzjmj5dnmvscjpg.dx.internal.cloudapp.net:30070/fsck?ugi=hdiuser&path=%2F
   FSCK started by hdiuser (auth:SIMPLE) from /10.0.0.22 for path / at Wed Apr 05 16:40:28 UTC 2017
   ....................................................................................................

   ....................................................................................................
   ..................Status: HEALTHY
   Total size:    9330539472 B
   Total dirs:    37
   Total files:   2618
   Total symlinks:                0 (Files currently being written: 2)
   Total blocks (validated):      2535 (avg. block size 3680686 B)
   Minimally replicated blocks:   2535 (100.0 %)
   Over-replicated blocks:        0 (0.0 %)
   Under-replicated blocks:       0 (0.0 %)
   Mis-replicated blocks:         0 (0.0 %)
   Default replication factor:    3
   Average block replication:     3.0
   Corrupt blocks:                0
   Missing replicas:              0 (0.0 %)
   Number of data-nodes:          8
   Number of racks:               1
   FSCK ended at Wed Apr 05 16:40:28 UTC 2017 in 187 milliseconds

   The filesystem under path '/' is HEALTHY
   ```

3. 如果确定没有缺失、损坏或复制不足的块，或者这些块可以忽略，请运行以下命令使名称节点退出安全模式：

   ```apache
   hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -safemode leave
   ```


## <a name="how-do-i-fix-jdbc-or-sqlline-connectivity-issues-with-apache-phoenix"></a>如何解决 Apache Phoenix 的 JDBC 或 SQLLine 连接问题？

### <a name="resolution-steps"></a>解决步骤

若要与 Apache Phoenix 进行连接，必须提供活动 Apache ZooKeeper 节点的 IP 地址。 确保 sqlline.py 尝试连接的 ZooKeeper 服务已启动且正在运行。
1. 使用 SSH 登录到 HDInsight 群集。
2. 输入以下命令：
                
   ```apache
           "/usr/hdp/current/phoenix-client/bin/sqlline.py <IP of machine where Active Zookeeper is running"
   ```

   > [!Note] 
   > 可以从 Ambari UI 获取活动 ZooKeeper 节点的 IP 地址。 转到“HBase” > “快速链接” > “ZK (活动)” > “Zookeeper 信息”。**\*** 

3. 如果 sqlline.py 能够连接到 Phoenix 并且未超时，则运行以下命令来验证 Phoenix 的可用性和运行状况：

   ```apache
           !tables
           !quit
   ```      
4. 如果可正常运行此命令，则表示没有问题。 用户提供的 IP 地址可能不正确。 但是，如果命令长时间暂停，随后显示以下错误，请转到步骤 5。

   ```apache
           Error while connecting to sqlline.py (Hbase - phoenix) Setting property: [isolation, TRANSACTION_READ_COMMITTED] issuing: !connect jdbc:phoenix:10.2.0.7 none none org.apache.phoenix.jdbc.PhoenixDriver Connecting to jdbc:phoenix:10.2.0.7 SLF4J: Class path contains multiple SLF4J bindings. 
   ```

5. 从头节点 (hn0) 运行以下命令来诊断 phoenix SYSTEM.CATALOG 表的情况：

   ```apache
            hbase shell
                
           count 'SYSTEM.CATALOG'
   ```

   此命令应返回类似于下方所示的错误： 

   ```apache
           ERROR: org.apache.hadoop.hbase.NotServingRegionException: Region SYSTEM.CATALOG,,1485464083256.c0568c94033870c517ed36c45da98129. is not online on 10.2.0.5,16020,1489466172189) 
   ```
6. 在 Apache Ambari UI 中完成以下步骤，以在所有 ZooKeeper 节点上重启 HMaster 服务：

    1. 在 HBase 的“摘要”部分中，转到“HBase” > “Active HBase Master”。 
    2. 在“组件”部分中，重启 HBase Master 服务。
    3. 为所有剩余的“Standby HBase Master”服务重复以上步骤。 

HBase Master 服务可能需要花费长达五分钟才能稳定下来并完成恢复过程。 几分钟后，重复 sqlline.py 命令以确认 SYSTEM.CATALOG 表已开启并且可以查询。 

当 SYSTEM.CATALOG 表恢复正常后，Phoenix 的连接问题应当会自动得以解决。


## <a name="what-causes-a-master-server-to-fail-to-start"></a>哪些因素会导致主服务器无法启动？

### <a name="error"></a>错误 

发生原子重命名失败。

### <a name="detailed-description"></a>详细说明

在启动过程中，HMaster 会完成大量的初始化步骤。 这些步骤包括将数据从暂存 (.tmp) 文件夹移动到数据文件夹。 HMaster 还会查看预写日志 (WAL) 文件夹中是否存在任何无响应的区域服务器，等等。 

在启动期间，HMaster 会对这些文件夹执行基本的 `list` 命令。 在任何时候，如果 HMaster 发现其中任何一个文件夹中存在意外的文件，则会引发异常并且不会启动。  

### <a name="probable-cause"></a>可能的原因

在区域服务器日志中，尝试查明文件创建时间线，并查看在该时间附近是否有任何进程崩溃。 （可联系 HBase 支持人员来协助执行此操作。）这可以帮助我们提供更可靠的机制以避免出现此 bug 并确保正常的进程关闭。

### <a name="resolution-steps"></a>解决步骤

检查调用堆栈并尝试确定哪个文件夹可能导致问题（例如，是 WAL 文件夹还是 .tmp 文件夹）。 然后，通过 Cloud Explorer 或 HDFS 命令尝试找到问题文件。 通常，这是一个 \*-renamePending.json 文件。 （\*-renamePending.json 文件是用于在 WASB 驱动程序中实现原子重命名操作的日记文件。 由于此实现中的 bug，在发生进程崩溃之类的问题后，这些文件可能会保留。）可以通过 Cloud Explorer 或 HDFS 命令强制删除此文件。 

有时，此位置还可能存在名称类似于 *$$$.$$$* 的临时文件。 必须使用 HDFS `ls` 命令查看此文件，而不能在 Cloud Explorer 中查看。 若要删除此文件，请使用 HDFS 命令 `hdfs dfs -rm /\<path>\/\$\$\$.\$\$\$`。  

运行这些命令后，HMaster 应会立即启动。 

### <a name="error"></a>错误

区域 xxx 的 *hbase: meta* 中未列出服务器地址。

### <a name="detailed-description"></a>详细说明

Linux 群集上可能会出现一条消息，指出 *hbase: meta* 表未联机。 运行 `hbck` 时可能会报告“在任何区域中都未发现 hbase: meta 表 replicaId 0”。 问题可能是重启 HBase 后，HMaster 无法初始化。 在 HMaster 日志中，你会看到消息：“区域 hbase 的 hbase: meta 中未列出服务器地址：\<区域名称\>”。  

### <a name="resolution-steps"></a>解决步骤

1. 在 HBase shell 中输入以下命令（根据情况更改实际值）：  

   ```apache
   > scan 'hbase:meta'  
   ```

   ```apache
   > delete 'hbase:meta','hbase:backup <region name>','<column name>'  
   ```

2. 删除 *hbase: namespace* 条目。 此条目可能是扫描 *hbase: namespace* 表时报告的相同错误。

3. 为了使 HBase 恢复运行状态，在 Ambari UI 中重启 Active HMaster 服务。  

4. 在 HBase shell 中，运行以下命令使所有脱机表联机：

   ```apache 
   hbase hbck -ignorePreCheckPermission -fixAssignments 
   ```

### <a name="additional-reading"></a>其他阅读材料

[无法处理 HBase 表](https://stackoverflow.com/questions/4794092/unable-to-access-hbase-table)


### <a name="error"></a>错误

HMaster 超时时出现类似于“java.io.IOException:超时 300000 毫秒，等待分配命名空间表”的致命异常。

### <a name="detailed-description"></a>详细说明

如果有大量表和区域且在重启 HMaster 服务时未刷新，则会遇到此问题。 重启可能失败且出现上述错误消息。  

### <a name="probable-cause"></a>可能的原因

这是 HMaster 服务的已知问题。 一般群集启动任务可能会花费很长时间。 由于命名空间表尚未分配，因此 HMaster 会关闭。 只有存在大量未刷新的数据且五分钟超时时间不够用时，才会出现此问题。
  
### <a name="resolution-steps"></a>解决步骤

1. 在 Apache Ambari UI 中，转到“HBase” > “配置”。 在自定义 hbase-site.xml 文件中添加以下设置： 

   ```apache
   Key: hbase.master.namespace.init.timeout Value: 2400000  
   ```

2. 重启所需的服务（HMaster，可能还有其他 HBase 服务）。  


## <a name="what-causes-a-restart-failure-on-a-region-server"></a>哪些因素会导致在区域服务器上重启失败？

### <a name="issue"></a>问题

可以遵循以下最佳做法来防止区域服务器重启失败。 我们建议在计划重启 HBase 区域服务器时，暂停繁重的工作负荷活动。 如果在关闭过程中应用程序继续与区域服务器进行连接，则这会将区域服务器重启操作拖慢几分钟。 另外，最好是先刷新所有表。 有关如何刷新表的参考信息，请参阅 [HDInsight HBase：如何通过刷新表改进 Apache HBase 群集重启时间](https://web.archive.org/web/20190112153155/ https://blogs.msdn.microsoft.com/azuredatalake/2016/09/19/hdinsight-hbase-how-to-improve-hbase-cluster-restart-time-by-flushing-tables/)。

如果通过 Apache Ambari UI 在 HBase 区域服务器上开始重启操作，马上就会看到区域服务器关闭，但不会立即重启。 

下面是幕后发生的情况： 

1. Ambari 代理向区域服务器发送停止请求。
2. Ambari 代理等待 30 秒让区域服务器正常关闭。 
3. 如果应用程序继续与区域服务器进行连接，该服务器不会立即关闭。 在关闭之前，30 秒超时就会到期。 
4. 30 秒之后，Ambari 代理向区域服务器发送强制终止 (`kill -9`) 命令。 可以在 ambari-agent 日志（位于相应工作节点的 /var/log/ 目录中）查看此情况：

   ```apache
           2017-03-21 13:22:09,171 - Execute['/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh --config /usr/hdp/current/hbase-regionserver/conf stop regionserver'] {'only_if': 'ambari-sudo.sh  -H -E t
           est -f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >/dev/null 2>&1', 'on_timeout': '! ( ambari-sudo.sh  -H -E test -
           f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >/dev/null 2>&1 ) || ambari-sudo.sh -H -E kill -9 `ambari-sudo.sh  -H 
           -E cat /var/run/hbase/hbase-hbase-regionserver.pid`', 'timeout': 30, 'user': 'hbase'}
           2017-03-21 13:22:40,268 - Executing '! ( ambari-sudo.sh  -H -E test -f /var/run/hbase/hbase-hbase-regionserver.pid && ps -p `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid` >
           /dev/null 2>&1 ) || ambari-sudo.sh -H -E kill -9 `ambari-sudo.sh  -H -E cat /var/run/hbase/hbase-hbase-regionserver.pid`'. Reason: Execution of 'ambari-sudo.sh su hbase -l -s /bin/bash -c 'export  
           PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/var/lib/ambari-agent ; /usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh --config /usr/hdp/curre
           nt/hbase-regionserver/conf stop regionserver was killed due timeout after 30 seconds
           2017-03-21 13:22:40,285 - File['/var/run/hbase/hbase-hbase-regionserver.pid'] {'action': ['delete']}
           2017-03-21 13:22:40,285 - Deleting File['/var/run/hbase/hbase-hbase-regionserver.pid']
   ```
   由于这种突然关闭，尽管区域服务器进程已停止，但与该进程关联的端口可能没有释放。 此状况可能会导致在启动区域服务器时发生 AddressBindException，如以下日志中所示。 可以在区域服务器启动失败的工作节点上的 /var/log/hbase 目录中的 region-server.log 内对此进行验证。 

   ```apache

   2017-03-21 13:25:47,061 ERROR [main] regionserver.HRegionServerCommandLine: Region server exiting
   java.lang.RuntimeException: Failed construction of Regionserver: class org.apache.hadoop.hbase.regionserver.HRegionServer
   at org.apache.hadoop.hbase.regionserver.HRegionServer.constructRegionServer(HRegionServer.java:2636)
   at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.start(HRegionServerCommandLine.java:64)
   at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.run(HRegionServerCommandLine.java:87)
   at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
   at org.apache.hadoop.hbase.util.ServerCommandLine.doMain(ServerCommandLine.java:126)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.main(HRegionServer.java:2651)
        
   Caused by: java.lang.reflect.InvocationTargetException
   at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
   at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
   at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
   at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.constructRegionServer(HRegionServer.java:2634)
   ... 5 more
        
   Caused by: java.net.BindException: Problem binding to /10.2.0.4:16020 : Address already in use
   at org.apache.hadoop.hbase.ipc.RpcServer.bind(RpcServer.java:2497)
   at org.apache.hadoop.hbase.ipc.RpcServer$Listener.<init>(RpcServer.java:580)
   at org.apache.hadoop.hbase.ipc.RpcServer.<init>(RpcServer.java:1982)
   at org.apache.hadoop.hbase.regionserver.RSRpcServices.<init>(RSRpcServices.java:863)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.createRpcServices(HRegionServer.java:632)
   at org.apache.hadoop.hbase.regionserver.HRegionServer.<init>(HRegionServer.java:532)
   ... 10 more
        
   Caused by: java.net.BindException: Address already in use
   at sun.nio.ch.Net.bind0(Native Method)
   at sun.nio.ch.Net.bind(Net.java:463)
   at sun.nio.ch.Net.bind(Net.java:455)
   at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223)
   at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74)
   at org.apache.hadoop.hbase.ipc.RpcServer.bind(RpcServer.java:2495)
   ... 15 more
   ```

### <a name="resolution-steps"></a>解决步骤

1. 在开始重启之前尝试降低 HBase 区域服务器上的负载。 
2. 另外（如果步骤 1 没有帮助），请尝试使用以下命令手动重启工作节点上的区域服务器：

   ```apache
   sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh stop regionserver"
   sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh start regionserver"   
   ```

### <a name="see-also"></a>另请参阅
[使用 Azure HDInsight 进行故障排除](../../hdinsight/hdinsight-troubleshoot-guide.md)
