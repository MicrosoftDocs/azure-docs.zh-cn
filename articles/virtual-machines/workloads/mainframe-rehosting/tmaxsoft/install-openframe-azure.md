---
title: 在 Azure 虚拟机上安装 TmaxSoft OpenFrame
description: 重新托管使用 TmaxSoft OpenFrame 环境在 Azure 虚拟机 (Vm) 在 IBM z/OS 大型机工作负荷。
services: virtual-machines-linux
documentationcenter: ''
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: virtual-machines-linux
ms.openlocfilehash: b69ded2591478a477cd142decb39218841c9ac62
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410114"
---
# <a name="install-tmaxsoft-openframe-on-azure"></a>在 Azure 上安装 TmaxSoft OpenFrame

了解如何设置 Azure 上的 OpenFrame 环境适用于开发、 演示、 测试或生产工作负荷。 本教程将指导你完成每个步骤。

OpenFrame 包括在 Azure 创建大型机仿真环境的多个组件。 例如，OpenFrame 联机服务替换大型机中间件如 IBM 客户信息控制系统 (CICS)，并 OpenFrame 批处理与其 TJES 的组件替换 IBM 大型机作业条目子系统 (JES)。

OpenFrame 适用于任何关系数据库，包括 Oracle Database、 Microsoft SQL Server、 IBM Db2 和 MySQL。 此安装 OpenFrame 使用 TmaxSoft Tibero 关系数据库。 OpenFrame 和 Tibero Linux 操作系统上运行。 本教程会安装 CentOS 7.3，尽管可以使用其他受支持的 Linux 发行版。OpenFrame 应用程序服务器和 Tibero 数据库安装在一台虚拟机 (VM) 上。

本教程将指导你逐步 OpenFrame 套件组件的安装。 一些必须单独安装。

Main OpenFrame 组件：

- 所需的安装包。
- Tibero 数据库。
- OpenFrame 中的应用程序使用开放式数据库连接 (ODBC) 与 Tibero 数据库进行通信。
- OpenFrame 基数，来管理整个系统的中间件。
- OpenFrame 批，将替换大型机的批处理系统的解决方案。
- TACF，控制用户对系统和资源访问的服务模块。
- ProSort，批次事务的排序工具。
- OFCOBOL，解释大型机的 COBOL 程序的编译器。
- OFASM，编译器将解释大型机的组装器程序。
- OpenFrame 服务器类型 C (OSC)，将替换大型机的中间件和 IBM CICS 的解决方案。
- Java 企业用户解决方案 (JEUS)，经过认证的 Java Enterprise Edition 6 的 web 应用程序服务器。
- OFGW，提供一个 3270 侦听器 OpenFrame 网关组件。
- OFManager，一个解决方案，可以在 web 环境中的 OpenFrame 的操作和管理功能。

其他所需的 OpenFrame 组件：

- OSI，取代了大型机中间件和 IMS DC 的解决方案。
- TJES，提供了大型机的 JES 环境的解决方案。
- OFTSAM，可以 (V) SAM 文件在打开的系统中使用的解决方案。
- OFHiDB，取代了大型机的解决方案的 IMS DB。
- OFPLI，编译器将解释大型机的 PL/我程序。
- PROTRIEVE，执行大型机语言 CA Easytrieve 的解决方案。
- OFMiner，分析大型机资产，然后将其迁移到 Azure 的解决方案。

## <a name="architecture"></a>体系结构

下图提供了安装在本教程中的 OpenFrame 7.0 体系结构组件的概述：

![OpenFrame 组件](media/openframe-02.png)

## <a name="azure-system-requirements"></a>Azure 系统要求

下表列出了在 Azure 上安装的要求。
<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>要求</th><th>描述</th></tr>
</thead>
<tbody>
<tr><td>在 Azure 上支持的 Linux 分发
</td>
<td>
Linux x86 2.6 （32 位、 64 位）<br/>
Red Hat 7.x<br/>
CentOS 7.x<br/>
</td>
</tr>
<tr><td>硬件
</td>
<td>内核数：2 （最低配置）<br/>
内存:4 GB （最小值）<br/>
交换空间：1 GB （最小值）<br/>
硬盘：100 GB （最小值）<br/>
</td>
</tr>
<tr><td>Windows 用户的的可选软件
</td>
<td>PuTTY:在本指南中用于配置 VM 的功能<br/>
WinSCP:常用的 SFTP 客户端和 FTP 客户端可以使用<br/>
Windows 的 eclipse:开发平台支持的 TmaxSoft<br/>
（Microsoft Visual Studio 不支持这一次）
</td>
</tr>
</tbody>
</table>

<!-- markdownlint-enable MD033 -->

## <a name="prerequisites"></a>必备组件

计划上花费几天来组合所有所需的软件和完成所有手动过程。

开始之前，请执行以下操作：

- 获取从 TmaxSoft OpenFrame 安装介质。 如果你是现有 TmaxSoft 客户，请与的许可副本的 TmaxSoft 代表联系。 否则，请求从试用版[TmaxSoft](https://www.tmaxsoft.com/contact/)。

- 通过发送电子邮件发送到请求 OpenFrame 文档<support@tmaxsoft.com>。

- 如果还没有一个，获取 Azure 订阅。 此外可以创建[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)在开始之前。

- 可选。 设置站点到站点 VPN 隧道或限制对 Azure VM 到你的组织中的允许用户访问的 jumpbox。 此步骤不是必需的但它是一种最佳做法。

## <a name="set-up-a-vm-on-azure-for-openframe-and-tibero"></a>为 OpenFrame 和 Tibero 设置 Azure 上的 VM

您可以设置在 OpenFrame 环境中使用各种部署模式，但下面的过程演示如何部署 OpenFrame 应用程序服务器和一个虚拟机上的 Tibero 数据库。 在大型环境中和相当大的工作负荷，最佳做法是将部署其自己的更好的性能的 VM 上单独的数据库。

**若要创建的 VM**

1. 转到 Azure 门户，网址<https://portal.azure.com>并登录到你的帐户。

2. 单击“虚拟机”。

    ![在 Azure 门户中的资源列表](media/vm-01.png)

3. 单击“添加”。

    ![在 Azure 门户中添加选项](media/vm-02.png)

4. 右侧**操作系统**，单击**详细**。

     ![在 Azure 门户中的更多选项](media/vm-03.png)

5. 单击**基于 CentOS 的 7.3**以完全正确，请按照本演练中也可以选择另一个受支持的 Linux 分发版。

     ![在 Azure 门户中的操作系统选项](media/vm-04.png)

6. 在中**基础知识**设置中，输入**名称**，**用户名**，**身份验证类型**，**订阅**（即用即付是付款的 AWS 样式），并**资源组**(使用一个现有或创建 TmaxSoft 组)。

7. 完成后 (包括的公钥/私钥对**身份验证类型**)，单击**提交**。

> [!NOTE]
> 如果使用的公共 SSH 密钥**身份验证类型**，请参阅下一部分，以生成公钥/私钥对中的步骤，然后恢复此处的步骤。

### <a name="generate-a-publicprivate-key-pair"></a>生成公钥/私钥对

如果使用的 Windows 操作系统，则需要 PuTTYgen 生成公钥/私钥对。

可以自由地共享公共密钥，但私钥应保密的完全和永远不应与第三方共享。 后生成的密钥，必须粘贴**SSH 公钥**到配置 — 实际上，将其上传到 Linux VM。 它存储内部授权\_中的关键字\~/.ssh 目录的用户帐户的主目录。 Linux VM 然后就能够识别和验证的连接，提供关联后**SSH 私钥**SSH 客户端 （在本例中，PuTTY） 中。

当向新个体访问 VM: 

- 每个新个体生成使用 PuTTYgen 自己公共/专用密钥。
- 个人单独存储其自己的私钥，并将公钥信息发送到 VM 的管理员。
- 管理员将对公钥的内容粘贴\~/.ssh/authorized\_的密钥文件。
- 通过 PuTTY 连接新个体。

**若要生成公钥/私钥对**

1.  下载从 PuTTYgen<https://www.putty.org/>和使用所有默认设置安装它。

2.  若要打开 PuTTYgen，定位到 PuTTY 安装目录 c： 驱动器中\\Program Files\\PuTTY。

    ![PuTTY 接口](media/puttygen-01.png)

3.  单击“生成”。

    ![PuTTY 密钥生成器对话框](media/puttygen-02.png)

4.  生成之后, 将保存的公钥和私钥。 中的公钥的内容粘贴**SSH 公钥**一部分**创建虚拟机\>基础知识**窗格 （步骤 6 和 7 中上一节中所示）。

    ![PuTTY 密钥生成器对话框](media/puttygen-03.png)

### <a name="configure-vm-features"></a>配置 VM 的功能

1. 在 Azure 门户中，在**选择大小**边栏选项卡中，选择所需的 Linux 计算机的硬件设置。 *最小*安装 Tibero 和 OpenFrame 要求是 2 个 Cpu 和 4 GB RAM，此示例安装中所示：

    ![创建虚拟机的基础知识](media/create-vm-01.png)

2. 单击**3 个设置**并使用默认设置来配置可选功能。
3. 查看付款详细信息。

    ![创建虚拟机-购买](media/create-vm-02.png)

4. 提交你的选择。 Azure 开始将 VM 部署。 此过程通常需要几分钟时间。

5. 部署 VM 时，将显示其仪表板，其中显示了在配置过程中选择的所有设置。 请记下的**公共 IP 地址**。

    ![Azure 仪表板上 tmax](media/create-vm-03.png)

6. 打开 PuTTY。

7. 有关**主机名**、 键入你的用户名和公共 IP 地址复制。 例如，**用户名\@publicip**。

    ![PuTTY 配置对话框](media/putty-01.png)

8. 在中**类别**框中，单击**连接\>SSH\>身份验证**。提供的路径你**私钥**文件。

    ![PuTTY 配置对话框](media/putty-02.png)

9. 单击**打开**启动 PuTTY 窗口。 如果成功，你将连接到新的 CentOS VM 在 Azure 上运行。

10. 若要以根用户身份登录，请键入**sudo bash**。

    ![在命令窗口中的根用户登录](media/putty-03.png)

## <a name="set-up-the-environment-and-packages"></a>设置环境和包

现在，创建 VM 并登录，必须执行几个安装程序步骤并安装所需的预安装包。

1. 将名称映射**ofdemo**通过使用 vi 编辑主机文件的本地 IP 地址 (`vi /etc/hosts`)。 假设我们的 IP 是 192.168.96.148 ofdemo，它在更改前：

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    <IP Address>    <your hostname>
    ```

     这是在更改后：

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    192.168.96.148   ofdemo
    ```

2. 创建组和用户：

    ```vi
    [root@ofdemo ~]# adduser -d /home/oframe7 oframe7 
    [root@ofdemo ~]# passwd oframe7
    ```

3. 更改用户 oframe7 的密码：

    ```vi
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.
    ```

4. 更新 /etc/sysctl.conf 中的内核参数：

    ```vi
    [root@ofdemo ~]# vi /etc/sysctl.conf
    kernel.shmall = 7294967296 
    kernel.sem = 10000 32000 10000 10000
    ```

5. 刷新动态无需重新启动内核参数：

    ```vi
    [root@ofdemo ~]# /sbin/sysctl -p
    ```

6. 获取所需的包：请确保服务器已连接到 Internet、 下载以下程序包，并安装它们：

     - dos2unix
     - glibc
     - glibc.i686 glibc.x86\_64
     - libaio
     - ncurses

          > [!NOTE]
          > 在安装后 ncurses 包，创建以下符号链接：
         ```
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so.2
         ```

     - gcc
     - gcc-c++
     - libaio-devel.x86\_64
     - 跟踪来
     - ltrace
     - gdb

7. 对于 Java RPM 安装中，执行以下步骤：

```
root@ofdemo ~]# rpm -ivh jdk-7u79-linux-x64.rpm
[root@ofdemo ~]# vi .bash_profile

# JAVA ENV
export JAVA_HOME=/usr/java/jdk1.7.0_79/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

[root@ofdemo ~]# source /etc/profile
[root@ofdemo ~]# java –version

java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

[root@ofdemo ~]# echo $JAVA_HOME /usr/java/jdk1.7.0_79/
```

## <a name="install-the-tibero-database"></a>安装 Tibero 数据库

Tibero 在 Azure 上提供 OpenFrame 环境中的若干关键功能：

- Tibero 用作各种系统函数 OpenFrame 内部数据存储区。
- VSAM 文件，其中包括 KSDS、 RRDS 和 ESDS，用于数据存储在内部使用 Tibero 数据库。
- TACF 数据存储库存储在 Tibero 中。
- OpenFrame 目录信息存储在 Tibero。
- Tibero 数据库可以作为 IBM Db2 的替代，用于存储应用程序数据。

**若要安装 Tibero**

1. 验证存在 Tibero 二进制安装程序文件，并查看版本号。
2. 将 Tibero 软件复制到 Tibero 用户帐户 (oframe)。 例如:

    ```
    [oframe7@ofdemo ~]$ tar -xzvf tibero6-bin-6_rel_FS04-linux64-121793-opt-tested.tar.gz 
    [oframe7@ofdemo ~]$ mv license.xml /opt/tmaxdb/tibero6/license/
    ```

3. 打开.bash\_第六部分中的配置文件 (`vi .bash_profile`) 并在其中粘贴以下：

    ```
    # Tibero6 ENV
    export TB_HOME=/opt/tmaxdb/tibero6 
    export TB_SID=TVSAM export TB_PROF_DIR=$TB_HOME/bin/prof 
    export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH 
    export PATH=$TB_HOME/bin:$TB_HOME/client/bin:$PATH
    ```

4. 若要执行 bash 配置文件中的，在命令提示符下键入：

    ```
    source .bash_profile
    ```

5. 生成的提示文件 （Tibero 配置文件），然后在 vi 中打开它。 例如:

    ```
    [oframe7@ofdemo ~]$ sh $TB_HOME/config/gen_tip.sh
    [oframe7@ofdemo ~]$ vi $TB_HOME/config/$TB_SID.tip
    ```

6. 修改\$TB\_HOME/client/config/tbdsn.tbr 并改为将 127.0.0.1 oflocalhost 所示：

    ```
    TVSAM=( 
    (INSTANCE=(HOST=127.0.0.1)
        (PT=8629)
    (DB_NAME=TVSAM)
          )
     )
    ```

7. 创建数据库。 将显示以下输出：

    ```
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NOMOUNT mode).
     /--------------------- newmount sql ------------------------/
    create database character set MSWIN949 national character set UTF16;
    /-----------------------------------------------------------/
    Database created.
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    /opt/tmaxdb/tibero6/bin/tbsvr
    ………………………..
    Creating agent table...
    Done.
    For details, check /opt/tmaxdb/tibero6/instance/TVSAM/log/system_init.log.
    ************************************************** 
    * Tibero Database TVSAM is created successfully on Fri Aug 12 19:10:43 UTC 2016.
    *     Tibero home directory ($TB_HOME) =
    *         /opt/tmaxdb/tibero6
    *     Tibero service ID ($TB_SID) = TVSAM
    *     Tibero binary path =
    *         /opt/tmaxdb/tibero6/bin:/opt/tmaxdb/tibero6/client/bin
    *     Initialization parameter file =
    *         /opt/tmaxdb/tibero6/config/TVSAM.tip
    * 
    * Make sure that you always set up environment variables $TB_HOME and
    * $TB_SID properly before you run Tibero.
     ******************************************************************************
    ```

8. 若要回收 Tibero，首先关闭它使用`tbdown`命令。 例如:

    ```
    [oframe7@ofdemo ~]$$ tbdown 
    Tibero instance terminated (NORMAL mode).
    ```

9. 现在启动 Tibero 使用`tbboot`。 例如:

    ```
    [oframe7@ofdemo ~]$ tbboot
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof. Listener port = 8629

    Tibero 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    ```

10. 若要创建表空间，请访问使用 SYS 的数据库用户 (sys/tmax)，然后创建必要的表空间的默认音量和 TACF:

    ```
    [oframe7@ofdemo ~]$ tbsql tibero/tmax
    tbSQL 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Connected to Tibero.
    ```

11. 现在，键入以下 SQL 命令：

    ```
    SQL> create tablespace "DEFVOL" datafile 'DEFVOL.dbf' size 500M autoextend on; create tablespace "TACF00" datafile 'TACF00.dbf' size 500M autoextend on; create tablespace "OFM_REPOSITORY" datafile 'ofm_repository.dbf' size 300M autoextend on;
    SQL> Tablespace 'DEFVOL' created.
    SQL> Tablespace 'TACF00' created.
    SQL> Tablespace ' OFM_REPOSITORY ' created.
    SQL> SQL> Disconnected.
    ```

12. 启动 Tibero 并验证 Tibero 进程正在运行：

    ```
    [oframe7@ofdemo ~]$ tbboot 
    ps -ef | egrep tbsvr
    ```

输出：

![Tibero 输出](media/tibero-01.png)

## <a name="install-odbc"></a>安装 ODBC

OpenFrame 中的应用程序使用 ODBC API 的开源 unixODBC 项目提供 Tibero 数据库进行通信。

若要安装 ODBC:

1. 验证 unixODBC 2.3.4.tar.gz 安装程序文件是否存在，或使用`wget unixODBC-2.3.4.tar.gz`命令。 例如:

     ```
     [oframe7@ofdemo ~]$ wget ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.4.tar.gz
     ```

2. 将解压缩二进制文件。 例如:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf unixODBC-2.3.4.tar.gz
     ```

3. 导航到 unixODBC 2.3.4 目录并使用该检查的计算机信息生成生成文件。 例如:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ ./configure --prefix=/opt/tmaxapp/unixODBC/ --sysconfdir=/opt/tmaxapp/unixODBC/etc
     ```

     默认情况下，unixODBC 安装在 /usr /local 因此`--prefix`将传递一个值，以更改的位置。 同样，配置文件安装在 /etc 中默认情况下，因此`--sysconfdir`传递所需的位置的值。

4. 执行生成文件： `[oframe7@ofdemo unixODBC-2.3.4]$ make`

5. 编译后程序目录中复制的可执行文件。 例如:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ make install
     ```

6. 使用 vi 编辑 bash 配置文件 (`vi ~/.bash_profile`) 并添加以下：

     ```
     # UNIX ODBC ENV 
     export ODBC_HOME=$HOME/unixODBC 
     export PATH=$ODBC_HOME/bin:$PATH 
     export LD_LIBRARY_PATH=$ODBC_HOME/lib:$LD_LIBRARY_PATH 
     export ODBCINI=$HOME/unixODBC/etc/odbc.ini 
     export ODBCSYSINI=$HOME
     ```

7. 用于 ODBC。 相应地编辑以下文件。 例如:

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ source ~/.bash_profile

     [oframe7@ofdemo ~]$ cd

     [oframe7@ofdemo ~]$ odbcinst -j unixODBC 2.3.4
     DRIVERS............: /home/oframe7/odbcinst.ini
     SYSTEM DATA SOURCES: /home/oframe7/odbc.ini
     FILE DATA SOURCES..: /home/oframe7/ODBCDataSources
     USER DATA SOURCES..: /home/oframe7/unixODBC/etc/odbc.ini
     SQLULEN Size.......: 8
     SQLLEN Size........: 8
     SQLSETPOSIROW Size.: 8

     [oframe7@ofdemo ~]$ vi odbcinst.ini

     [Tibero]
     Description = Tibero ODBC driver for Tibero6
     Driver = /opt/tmaxdb/tibero6/client/lib/libtbodbc.so
     Setup = 
     FileUsage = 
     CPTimeout = 
     CPReuse = 
     Driver Logging = 7

     [ODBC]
     Trace = NO 
     TraceFile = /home/oframe7/odbc.log 
     ForceTrace = Yes 
     Pooling = No 
     DEBUG = 1

     [oframe7@ofdemo ~]$ vi odbc.ini

     [TVSAM]
     Description = Tibero ODBC driver for Tibero6 
     Driver = Tibero 
     DSN = TVSAM 
     SID = TVSAM 
     User = tibero 
     password = tmax
     ```

8. 创建符号链接并验证 Tibero 数据库连接：

     ```
     [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbc.so $ODBC_HOME/lib/libodbc.so.1 [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbcinst.so 
     $ODBC_HOME/lib/libodbcinst.so.1

     [oframe7@ofdemo lib]$ isql TVSAM tibero tmax
     ```

将显示以下输出：

![ODBC 输出显示了连接到 SQL](media/odbc-01.png)

## <a name="install-openframe-base"></a>安装 OpenFrame 基础

基本应用程序服务器安装之前 OpenFrame 使用来管理在 Azure 上，包括服务器处理的事务系统的各个服务。

**若要安装 OpenFrame 基础**

1. 请确保 Tibero 安装成功，然后验证以下 OpenFrame\_基准 7 成本\_0\_Linux\_x86\_64.bin 安装程序文件和 base.properties 配置文件是否存在。

2. 使用以下特定于 Tibero 的信息更新 bash 配置文件：

     ```bash
     alias ofhome='cd $OPENFRAME_HOME'
     alias ulog='cd $OPENFRAME_HOME/log/tmax/ulog'
     alias sysjcl='cd $OPENFRAME_HOME/volume_default/SYS1.JCLLIB'
     alias sysload='cd $OPENFRAME_HOME/volume_default/SYS1.LOADLIB'
     alias sysproc='cd $OPENFRAME_HOME/volume_default/SYS1.PROCLIB'
     alias oscsrc='cd $OPENFRAME_HOME/osc/oivp'
     alias osisrc='cd $OPENFRAME_HOME/osi/oivp'
     alias defvol='cd $OPENFRAME_HOME/volume_default'
     ```

3. 执行 bash 配置文件：`[oframe7@ofdemo ~]$ . .bash_profile`
4. 请确保 Tibero 进程正在运行。 例如:

     ```linux
     [oframe7@ofdemo ~]$ ps -ef|grep tbsvr
     ```

    ![基本](media/base-01.png)

     > [!IMPORTANT]
     > 请确保在开始安装之前 Tibero。

5. 生成在许可证[technet.tmaxsoft.com](https://technet.tmaxsoft.com/en/front/main/main.do)和 PUT OpenFrame Base、 批处理、 TACF OSC 的许可证的相应文件夹中：

     ```
     [oframe7@ofdemo ~]$ cp license.dat /opt/tmaxapp/OpenFrame/core/license/
     [oframe7@ofdemo ~]$ cp lictjes.dat lictacf.dat licosc.dat $OPENFRAME_HOME/license/
     ```

6. 下载 OpenFrame 基数的二进制文件和 base.properties 文件：

     ```
     [oframe7@ofdemo ~]$ vi base.properties
     OPENFRAME_HOME= <appropriate location for installation> ex. /opt/tmaxapp/OpenFrame TP_HOST_NAME=<your IP Hostname> ex. ofdemo 
     TP_HOST_IP=<your IP Address> ex. 192.168.96.148 
     TP_SHMKEY=63481 
     TP_TPORTNO=6623 
     TP_UNBLOCK_PORT=6291 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     MASCAT_NAME=SYS1.MASTER.ICFCAT 
     MASCAT_CREATE=YES 
     DEFAULT_VOLSER=DEFVOL 
     VOLADD_DEFINE=YES TSAM_USERNAME=tibero 
     TSAM_PASSWORD=tmax 
     TSAM_DATABASE=oframe 
     DATASET_SHMKEY=63211 
     DSLOCK_DATA=SYS1.DSLOCK.DATA 
     DSLOCK_LOG=SYS1.DSLOCK.LOG 
     DSLOCK_SEQ=dslock_seq.dat 
     DSLOCK_CREATE=YES 
     OPENFRAME_LICENSE_PATH=/opt/tmaxapp/license/OPENFRAME TMAX_LICENSE_PATH=/opt/tmaxapp/license/TMAX
     ```

7. 执行安装程序使用 base.properties 文件。 例如:

    ```
    [oframe7@ofdemo ~]$ chmod a+x OpenFrame_Base7_0_Linux_x86_64.bin 
    [oframe7@ofdemo ~]$ ./OpenFrame_Base7_0_Linux_x86_64.bin -f base.properties
    ```

    完成后，安装完整的消息是显示。

8. 验证 OpenFrame 基目录结构使用`ls -ltr`命令。 例如:

     ```
     [oframe7@ofdemo OpenFrame]$ ls -ltr
     total 44

     drwxrwxr-x. 4 oframe7 oframe7 61 Nov 30 16:57 UninstallerData 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 bin 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 cpm drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 data 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 include 
     drwxrwxr-x. 2 oframe7 oframe7 8192 Nov 30 16:57 lib 
     drwxrwxr-x. 6 oframe7 oframe7 48 Nov 30 16:57 log 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 profile 
     drwxrwxr-x. 7 oframe7 oframe7 62 Nov 30 16:57 sample 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 schema 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 temp 
     drwxrwxr-x. 3 oframe7 oframe7 16 Nov 30 16:57 shared 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 license 
     drwxrwxr-x. 23 oframe7 oframe7 4096 Nov 30 16:58 core 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 config 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 scripts 
     drwxrwxr-x. 2 oframe7 oframe7 25 Nov 30 16:58 volume_default
     ```

9. 开始 OpenFrame 基：

     ```
     [oframe7@ofdemo ~]$ cp /usr/lib/libtermcap.so.2 $TMAXDIR/lib
     Startup Tmax Server
     [oframe7@ofdemo ~]$ tmboot
     ```

     ![tmboot 命令输出](media/base-02.png)

10. 确认进程状态已准备在 si 使用 tmadmin 命令。 中显示 RDY**状态**进程的每个列：

     ![tmadmin 命令输出](media/base-03.png)

11. 关闭 OpenFrame 基：

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(vtammgr:43) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLH downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLL downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TLM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-openframe-batch"></a>安装 OpenFrame 批处理

OpenFrame 批处理模拟大型机批处理环境的多个组件组成，用于在 Azure 上运行的批处理作业。

**若要安装批处理**

1. 请确保基本安装成功，然后验证 OpenFrame\_Batch7\_0\_Fix2\_MVS\_Linux\_x86\_64.bin 安装程序文件和batch.properties 配置文件有：

2. 在命令提示符处，键入`vi batch.properties`编辑使用 vi batch.properties 文件。

3. 按如下所示修改参数：

     ```
     OPENFRAME_HOME = /opt/tmaxapp/OpenFrame
     DEFAULT_VOLSER=DEFVOL 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     RESOURCE_SHMKEY=66991 
     #JOBQ_DATASET_CREATE=YES 
     #OUTPUTQ_DATASET_CREATE=YES 
     DEFAULT_JCLLIB_CREATE=YES 
     DEFAULT_PROCLIB_CREATE=YES 
     DEFAULT_USERLIB_CREATE=YES 
     TJES_USERNAME=tibero 
     TJES_PASSWORD=tmax 
     TJES_DATABASE=oframe 
     BATCH_TABLE_CREATE=YES
     ```

4. 若要执行批处理安装程序中的，在命令提示符下键入：

     ```
     ./OpenFrame_Batch7_0_Fix2_MVS_Linux_x86_64.bin -f batch.properties
     ```

5. 安装完成后，通过键入启动已安装的 OpenFrame 套件`tmboot`在命令提示符处。

    ![tmboot 输出](media/tmboot-01.png)

6. 类型`tmadmin`在命令提示符下，若要检查 OpenFrame 过程。

    ![Tmax 管理屏幕](media/tmadmin-01.png)

7. 执行以下命令：

     ```
     $$2 NODE1 (tmadm): quit 
     ADM quit for node (NODE1)
     ```

8. 使用`tmdown`命令来启动和关闭批处理：

     ```
     [oframe7@ofdemo ~]$tmdown
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:44) downed: Wed Sep  7 16:01:46 2016
     TMDOWN: SERVER(vtammgr: 43) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:45) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:46) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjinit:55) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:49) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:51) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:52) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:53) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLL downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TLM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLH downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-tacf"></a>安装 TACF

TACF 管理器是一个 OpenFrame 服务模块，用于控制用户对系统和通过 RACF 安全资源访问权限。

**若要安装 TACF**

1. 验证 OpenFrame\_Tacf7\_0\_Fix2\_Linux\_x86\_64.bin 安装程序文件和 tacf.properties 配置文件是否存在。
2. 请确保批处理安装成功，然后使用 vi 打开 tacf.properties 文件 (`vi tacf.properties`)。
3. 修改 TACF 参数：

     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame 
     USE_OS_AUTH=NO 
     TACF_USERNAME=tibero 
     TACF_PASSWORD=tmax 
     TACF_DATABASE=oframe 
     TACF_TABLESPACE=TACF00 
     TACF_TABLE_CREATE=YES
     ```

4. 完成后 TACF 安装程序，应用 TACF 环境变量。 在命令提示符处，键入：

     ```
     source \~/.bash\_profile
     ```

5. 执行 TACF 安装程序。 在命令提示符处，键入：

     ```
     ./OpenFrame_Tacf7_0_Fix2_Linux_x86_64.bin -f tacf.properties
     ```

     输出如下所示：

     ```
     Wed Dec 07 17:36:42 EDT 2016
     Free Memory: 18703 kB 
     Total Memory: 28800 kB

     4 Command Line Args: 
     0:  -f 1:  tacf.properties 
     2:  -m 
     3:  SILENT 
     java.class.path: 
     /tmp/install.dir.41422/InstallerData 
     /tmp/install.dir.41422/InstallerData/installer.zip 
     ZGUtil.CLASS_PATH: 
     /tmp/install.dir.41422/InstallerData 
     tmp/install.dir.41422/InstallerData/installer.zip 
     sun.boot.class.path: 
     /tmp/install.dir.41422/Linux/resource/jre/lib/resources.jar /tmp/install.dir.41422/Linux/resource/jre/lib/rt.jar /tmp/install.dir.41422/Linux/resource/jre/lib/sunrsasign.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jsse.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jce.jar /tmp/install.dir.41422/Linux/resource/jre/lib/charsets.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jfr.jar /tmp/install.dir.41422/Linux/resource/jre/classes
     ```

6. 在命令提示符处，键入`tmboot`重启 OpenFrame。 输出如下所示：

     ```
     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/7 
     TMBOOT: TMM is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLL is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLH is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: TLM(tlm) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsasvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrlhsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdsedt) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrcmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofruisvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsmlog) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(vtammgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjschd) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjinit) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjhist) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjspbk) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrpmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmtsmgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(tmsvr) is starting: Wed Sep  7 17:48:53 2016
     ```

7. 验证进程状态是否可以使用`tmadmin`在`si`命令。 例如:

     ```
     [oframe7\@ofdemo \~]\$ tmadmin
     ```

     在中**状态**列中，RDY 显示：

    ![在状态列中的 RDY](media/tmboot-02.png)

8. 执行以下命令：

     ```
     $$2 NODE1 (tmadm): quit 
     DM quit for node (NODE1)

     [oframe7@ofdemo ~]$ tacfmgr 
     Input USERNAME  : ROOT 
     Input PASSWORD  : SYS1

     TACFMGR: TACF MANAGER START!!!
     QUIT TACFMGR: TACF MANAGER END!!!

     [oframe7@ofdemo ~]$ tmdow
     ```

9. 关闭服务器使用`tmdown`命令。 输出如下所示：

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(tmsvr:60) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLL downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLH downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TLM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-prosort"></a>安装 ProSort

ProSort 是在批处理事务，用于对数据进行排序的实用工具。

**若要安装 ProSort**

1. 请确保批处理安装是否成功，，然后验证**prosort bin prosort\_2sp3 linux64 2123 opt.tar.gz**安装程序文件是否存在。

2. 执行安装程序使用的属性文件。 在命令提示符处，键入：

     ```
     tar -zxvf prosort-bin-prosort\_2sp3-linux64-2123-opt.tar.gz
     ```

3. 将 prosort 目录移动到主位置。 在命令提示符处，键入：

     ```
     mv prosort /opt/tmaxapp/prosort
     ```

4. 创建一个许可证子目录并将该许可证文件复制。 例如:

     ```
     cd /opt/tmaxapp/prosort 
     mkdir license 
     cp /opt/tmaxsw/oflicense/prosort/license.xml /opt/tmaxapp/prosort/license
     ```

5. 在 vi 中打开 bash.profile (`vi .bash_profile`) 并更新它，如下所示：

     ```bash
     #       PROSORT

     PROSORT_HOME=/opt/tmaxapp/prosort 
     PROSORT_SID=gbg 
     PATH=$PATH:$PROSORT_HOME/bin LD_LIBRARY_PATH=$PROSORT_HOME/lib:$LD_LIBRARY_PATH LIBPATH$PROSORT_HOME/lib:$LIBPATH 
     export PROSORT_HOME PROSORT_SID 
     PATH LD_LIBRARY_PATH LIBPATH 
     PATH=$PATH:$OPENFRAME_HOME/shbin 
     export PATH
     ```

6. 若要执行 bash 配置文件中的，在命令提示符处，键入： `. .bash_profile`

7. 创建配置文件。 例如:

     ```
     oframe@oframe7: cd /opt/tmaxapp/prosort/config 
     oframe@oframe7: ./gen_tip.sh 
     Using PROSORT_SID "gbg"
      /home/oframe7/prosort/config/gbg.tip generated
     ```

8. 创建符号链接。 例如:

     ```
     oframe@oframe7: cd /opt/tmaxapp/OpenFrame/util/ 
     oframe@oframe7home/oframe7/OpenFrame/util :  ln -s DFSORT SORT
     ```

9. 通过执行验证 ProSort 安装`prosort -h`命令。 例如:

     ```
     oframe@oframe7: prosort -h

     Usage: prosort [options] [sort script files]
     options ------
     -h             Display this information 
     -v             Display version information 
     -s             Display state information 
     -j             Display profile information 
     -x             Use SyncSort compatible mode
     ```

## <a name="install-ofcobol"></a>安装 OFCOBOL

OFCOBOL 是解释大型机的 COBOL 程序 OpenFrame 编译器。 

**若要安装 OFCOBOL**

1. 请确保批处理/联机安装成功，然后验证 OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin 安装程序文件是否存在。

2. 若要执行 OFCOBOL 安装程序，在命令提示符处，键入：

     ```
      ./OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin
     ```

3. 阅读许可协议，然后按 Enter 以继续。

4. 接受许可协议。 安装完成后，就会出现以下：

     ```
     Choose Install Folder 
     --------------------
     Where would you like to install?
     Default Install Folder: /home/oframe7/OFCOBOL

     ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT : /opt/tmaxapp/OFCOBOL

     INSTALL FOLDER IS: /opt/tmaxapp/OFCOBOL 
     IS THIS CORRECT? (Y/N): Y[oframe7@ofdemo ~]$ vi .bash_profile

     ============================================================================ Installing... 
     ------------
     [==================|==================|==================|==================] 
     [------------------|------------------|------------------|------------------]

     =============================================================================== Installation Complete 
     --------------------
     Congratulations. OpenFrame_COBOL has been successfully installed
     PRESS <ENTER> TO EXIT THE INSTALLER
     ```

5. 在 vi 中打开的 bash 配置文件 (`vi .bash_profile`)，并验证该更新与 OFCOBOL 变量。
6. 执行 bash 配置文件。 在命令提示符处，键入：

     ```
      source ~/.bash_profile
     ```

7. 将 OFCOBOL 许可证复制到安装文件夹。 例如:
     ```
     mv licofcob.dat $OFCOB_HOME/license
     ```
8. 转到 OpenFrame tjclrun.conf 配置文件，并在 vi 中打开它。 例如:
     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

   下面是在更改之前的 SYSLIB 节：
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib
     ```
   下面是在更改后的 SYSLIB 部分：
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib :${ODBC_HOME}/lib 
     :${OFCOB_HOME}/lib
     ```
9. 查看 OpenFrame\_COBOL\_InstallLog.log 文件位于 vi 并验证是否有任何错误。 例如:
     ```
     [oframe7@ofdemo ~]$ vi $OFCOB_HOME/UninstallerData/log/OpenFrame_COBOL_InstallLog.log 
     …….. 
     Summary 
     ------
     Installation: Successful. 
     131 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalError
     ```
10. 使用`ofcob --version`命令并查看要验证安装的版本号。 例如:

     ```
     [oframe7@ofdemo ~]$ ofcob --version 
     OpenFrame COBOL Compiler 3.0.54 
     CommitTag:: 645f3f6bf7fbe1c366a6557c55b96c48454f4bf
     ```

11. 重新启动 OpenFrame 使用`tmdown/tmboot`命令。

## <a name="install-ofasm"></a>安装 OFASM

OFASM 是解释大型机的组装器程序 OpenFrame 编译器。

**若要安装 OFASM**

1. 请确保批处理/联机安装成功，然后验证 OpenFrame\_ASM3\_0\_Linux\_x86\_64.bin 安装程序文件是否存在。

2. 执行安装程序。 例如:

     ```
     [oframe7@ofdemo ~]$ ./OpenFrame_ASM3_0_Linux_x86_64.bin
     ```

3. 阅读许可协议，然后按 Enter 以继续。
4. 接受许可协议。
5. 验证 bash 配置文件更新与 OFASM 变量。 例如:

     ```
     [oframe7@ofdemo ~]$ source .bash_profile
     [oframe7@ofdemo ~]$ ofasm --version 
     # TmaxSoft OpenFrameAssembler v3 r328 
     (3ff35168d34f6e2046b96415bbe374160fcb3a34)

     [oframe7@ofdemo OFASM]$ vi .bash_profile

     # OFASM ENV 
     export OFASM_HOME=/opt/tmaxapp/OFASM 
     export OFASM_MACLIB=$OFASM_HOME/maclib/free_macro 
     export PATH="${PATH}:$OFASM_HOME/bin:" 
     export LD_LIBRARY_PATH="./:$OFASM_HOME/lib:$LD_LIBRARY_PATH"
     ```

6. 在 vi 打开 OpenFrame tjclrun.conf 配置文件并对其进行编辑，如下所示：

     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

     下面是 [SYSLIB] 节*之前*更改：

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib
     ```

     下面是 [SYSLIB] 节*后*更改：

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib:${OFASM_HOME}/lib
     ```

7. 打开 OpenFrame\_ASM\_InstallLog.log 文件位于 vi 并验证是否有任何错误。 例如:

     ```
     [oframe7@ofdemo ~]$ vi 
     $OFASM_HOME/UninstallerData/log/OpenFrame_ASM_InstallLog.log 
     …….. 
     Summary 
     ------

     Installation: Successful.

     55 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalErrors
     ```

8. 通过发出以下命令之一重启 OpenFrame:

     ```
     tmdown / tmboot
     ```

     — 或 —

     ```
     oscdown / oscboot
     ```

## <a name="install-osc"></a>安装 OSC

OSC 是类似于支持高速 OLTP 事务和其他管理功能的 IBM CICS 的 OpenFrame 环境。

**若要安装 OSC**

1. 请确保基本安装成功，然后验证 OpenFrame\_OSC7\_0\_Fix2\_Linux\_x86\_64.bin 安装程序文件和 osc.properties 配置文件存在。
2. 编辑 osc.properties 文件中的以下参数：
     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame OSC_SYS_OSC_NCS_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_NCS OSC_APP_OSC_TC_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_TC
     ```

3. 执行安装程序使用的属性文件所示：

     ```
     [oframe7@ofdemo ~]$ chmod a+x OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin [oframe7@ofdemo ~]$ ./OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin -f osc.properties
     ```

     完成后，显示"安装已完成"消息。

4. 验证已由 OSC 变量更新 bash 配置文件。
5. 查看 OpenFrame\_OSC7\_0\_Fix2\_InstallLog.log 文件。 应该会看到类似下面的屏幕：

     ```
     Summary 
     ------ 
     Installation: Successful.

     233 Successes
     0 Warnings
     0 NonFatalErrors
     0 FatalError
     ```

6. 使用 vi 打开 ofsys.seq 配置文件。 例如:

     ```
     vi $OPENFRAME_HOME/config/ofsys.seq
     ```

7. 在中\#基准和\#批处理部分，如所示编辑的参数。

     ```
     Before changes
     #BASE
     ofrsasvr
     ofrlhsvr
     ofrdmsvr
     ofrdsedt
     ofrcmsvr
     ofruisvr
     ofrsmlog
     vtammgr
     TPFMAGENT

     #BATCH 
     #BATCH#obmtsmgr
     #BATCH#ofrpmsvr
     #BATCH#obmjmsvr
     #BATCH#obmjschd
     #BATCH#obmjinit
     #BATCH#obmjhist
     #BATCH#obmjspbk
     #TACF #TACF#tmsvr

     After changes  #BATCH
     #BASE          obmtsmgr 
     ofrsasvr       ofrpmsvr
     ofrlhsvr       obmjmsvr
     ofrdmsvr       obmjschd
     ofrdsedt       obmjinit
     ofrcmsvr       obmjhist
     ofruisvr       obmjspbk
     ofrsmlog
     vtammgr        #TACF
     TPFMAGENT      tmsvr
    ```

8. 将许可证文件复制。 例如:

     ```
     [oframe7@ofdemo ~]$ cp /home/oframe7/oflicense/ofonline/licosc.dat 

     $OPENFRAME_HOME/license

     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/license 
     oframe@oframe7/OpenFrame/license / ls -l 
     -rwxr-xr-x. 1 oframe mqm 80 Sep 12 01:37 licosc.dat 
     -rwxr-xr-x. 1 oframe mqm 80 Sep  8 09:40 lictacf.dat 
     -rwxrwxr-x. 1 oframe mqm 80 Sep  3 11:54 lictjes.da
     ```

9. 若要启动和关闭的情况下 OSC，初始化 CICS 区域共享内存，通过键入`osctdlinit OSCOIVP1`在命令提示符处。

10. 运行`oscboot`OSC 启动。 输出如下所示：

     ```
     OSCBOOT : pre-processing       [ OK ]

     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/12 
          TMBOOT: TMM is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLL is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLH is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: TLM(tlm) is starting: Mon Sep 12 01:40:25 2016 
     ```

11. 若要验证进程状态为准备就绪，请使用`tmadmin`si 命令。 所有进程应都显示在 RDY**状态**列。

    ![进程显示 RDY](media/tmadmin-02.png)

12. OSC 使用关闭`oscdown`命令。

## <a name="install-jeus"></a>安装 JEUS

JEUS （Java 企业用户解决方案） 提供了 OpenFrame web 应用程序服务器的表示层。

安装前 JEUS，安装 Apache Ant 程序包，其中提供的库和安装 JEUS 所需的命令行工具。

**若要安装 Apache Ant**

1. 下载 Ant 二进制使用`wget`命令。 例如:

     ```
     wget http://apache.mirror.cdnetworks.com/ant/binaries/apacheant-1.9.7-bin.tar.gz
     ```

2. 使用`tar`实用工具来解压缩二进制文件并将其移动到相应的位置。 例如:

     ```
     tar -xvzf apache-ant-1.9.7-bin.tar.gz
     ```

3. 为提高效率，创建符号链接：

     ```
     ln -s apache-ant-1.9.7 ant
     ```

4. 在 vi 中打开的 bash 配置文件 (`vi .bash_profile`) 并将其更新以下变量：

     ```
     # Ant ENV
     export ANT_HOME=$HOME/ant 
     export PATH=$HOME/ant/bin:$PATH
     ```

5.  将应用修改后的环境变量。 例如:

     ```
     [oframe7\@ofdemo \~]\$ source \~/.bash\_profile
     ```

**若要安装 JEUS**

1. 展开安装程序使用`tar`实用程序。 例如:

     ```
     [oframe7@ofdemo ~]$ tar -zxvf jeus704.tar.gz
     ```

2. 创建**jeus**文件夹 (`mkdir jeus7`) 并解压缩二进制文件。
3. 将更改为**安装程序**目录 （或 JEUS 参数用于在自己的环境）。 例如:

     ```
     [oframe7@ofdemo ~]$ cd jeus7/setup/
     ```

4. 执行`ant clean-all`之前执行生成。 输出如下所示：

     ```
     Buildfile: /home/oframe7jeus7/setup/build.xml

     clean-bin:
     delete-domain:
     [echo] Deleting a domain configuration: domain = jeus_domain
     delete-nodesxml:
     clean-config:
     clean-all:
     BUILD SUCCESSFUL
     Total time: 0 seconds
     ```

5.  请域配置 template.properties 文件的备份。 例如:

     ```
     [oframe7@ofdemo ~]$ cp domain-config-template.properties domain-configtemplate.properties.bkp
     ```

6. 在 vi 中打开域配置 template.properties 文件：

     ```
     [oframe7\@ofdemo setup]\$ vi domain-config-template.properties
     ```

7. 将 `jeus.password=jeusadmin nodename=Tmaxsoft` 更改为 `jeus.password=tmax1234 nodename=ofdemo`

8. 执行`ant install`命令以生成 JEUS。
9.  更新.bash\_与 JEUS 变量所示的配置文件：

     ```
     # JEUS ENV 
     export JEUS_HOME=/opt/tmaxui/jeus7 PATH="/opt/tmaxui/jeus7/bin:/opt/tmaxui/jeus7/lib/system:/opt/tmaxui/jeus7/webserver/bin:$ {PATH}" 
     export PATH
     ```

10. 执行 bash 配置文件。 例如:

     ```
     [oframe7@ofdemo setup]$ . .bash_profile
     ```

11. 可选。 创建简单的关闭和启动 JEUS 组件的别名：

     ```     
     # JEUS alias

     alias dsboot='startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin'
     alias msboot='startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin' 
     alias msdown=‘jeusadmin -u administrator -p tmax1234 "stop-server server1“’ 
     alias dsdown=‘jeusadmin -domain jeus_domain -u administrator -p tmax1234 "local-shutdown“’
     ```

12. 若要验证安装，请启动域管理服务器所示：

     ```
     [oframe7@ofdemo ~]$ startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin
     ```

13. 验证 web 登录使用的语法：

     ```
     http://<IP>:<port>/webadmin/login
     ```

     例如，<http://192.168.92.133:9736/webadmin/login.>出现登录屏幕：
    
     ![JEUS WebAdmin 登录屏幕](media/jeus-01.png)

     > [!NOTE]
     > 如果使用端口安全性遇到任何问题，打开端口 9736 或禁用防火墙 (`systemctl stop firewall`)。

14. 若要更改对 server1 的主机名，请单击**锁定和编辑**，然后单击**server1**。 在服务器窗口中更改的主机名，如下所示：

    1.  更改**Nodename**到**ofdemo**。
    2.  单击**确定**窗口右侧。
    3.  单击**将更改应用**左下角的窗口和有关说明，在输入*主机名更改*。

    ![JEUS WebAdmin 屏幕](media/jeus-02.png)

15. 验证配置成功在确认屏幕。

    ![jeus_domain Server screen](media/jeus-03.png)

16. 启动托管的服务器进程"server1"使用以下命令：

     ```
     [oframe7@ofdemo ~]$ startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin
     ```

## <a name="install-ofgw"></a>安装 OFGW

OFGW 是 OpenFrame 网关支持 3270 终端模拟器和 OSI 基之间的通信和管理终端模拟器和 OSI 之间的会话。

**若要安装 OFGW**

1. 请确保 JEUS 已安装成功，然后验证 OFGW7\_0\_1\_Generic.bin 安装程序文件是否存在。
2. 执行安装程序。 例如:

     ```
     [oframe7@ofdemo ~]$ ./OFGW7_0_1_Generic.bin
     ````

3. 使用以下位置的相应提示：
     -   JEUS 主目录
     -   JEUS 域名
     -   JEUS 服务器名称
     -   Tibero 驱动程序
     -   Tmax 节点 ID ofdemo

4. 接受其余的默认值，然后按 Enter 以退出安装程序。

5. 验证 OFGW 的 URL 按预期工作：

     ```
     Type URL 
     http://192.168.92.133:8088/webterminal/ and press enter
      < IP >               :8088/webterminal/
     ```

     出现以下屏幕：

    ![OpenFrame WebTerminal](media/ofgw-01.png)

## <a name="install-ofmanager"></a>安装 OFManager

OFManager 为 OpenFrame 提供操作和管理功能在 web 环境中。

**若要安装 OFManager**

1. 验证 OFManager7\_Generic.bin 安装程序文件是否存在。
2. 执行安装程序。 例如:

     ```
     OFManager7_Generic.bin
     ```

3.  按 Enter 以继续，然后接受许可协议。
4.  选择安装文件夹。
5.  接受默认值。
6.  选择 Tibero 作为数据库。
7.  按 Enter 以退出安装程序。
8.  验证 OFManager 的 URL 按预期工作：

     ```
     Type URL http://192.168.92.133:8088/ofmanager and press enter <  IP >  : < PORT >  ofmanager Enter ID:   ROOT 
     Password: SYS1
     ```

开始屏幕将显示：

![Tmax OpenFrame Manager 登录屏幕](media/ofmanager-01.png)

完成 OpenFrame 组件的安装。

## <a name="next-steps"></a>后续步骤

如果正在考虑大型机迁移，我们扩展的合作伙伴生态系统，可帮助你。 有关选择合作伙伴解决方案的详细指南，请参阅[平台现代化联盟](https://www.platformmodernization.org/pages/mainframe.aspx)。

-   [Azure 入门](https://docs.microsoft.com/azure/)
-   [Host Integration Server (HIS) 文档](https://docs.microsoft.com/host-integration-server/)
-   [Azure 虚拟数据中心提升和转移指南](https://blogs.msdn.microsoft.com/azurecat/2018/03/12/new-whitepaper-azure-virtual-datacenter-lift-and-shift-guide/)
