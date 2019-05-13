---
title: 使用 Apache Maven 构建 Java HBase 客户端的 Azure HDInsight
description: 了解如何使用 Apache Maven 构建基于 Java 的 Apache HBase 应用程序，然后将其部署到 Azure HDInsight 上的 HBase。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,seodec18
ms.topic: conceptual
ms.date: 04/16/2019
ms.openlocfilehash: a4c601e81390efa3bb53a6f07225bb6e939bc9bb
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64726444"
---
# <a name="build-java-applications-for-apache-hbase"></a>构建适用于 Apache HBase 的 Java 应用程序

了解如何使用 Java 创建 [Apache HBase](https://hbase.apache.org/) 应用程序。 然后，将该应用程序用于 Azure HDInsight 上的 HBase。

本文档中的步骤使用 [Apache Maven](https://maven.apache.org/) 创建和构建项目。 Maven 是一种软件项目管理和综合工具，可用于为 Java 项目构建软件、文档和报告。

## <a name="prerequisites"></a>必备组件

* HDInsight 上的 Apache HBase 群集。 请参阅[Apache HBase 入门](./apache-hbase-tutorial-get-started-linux.md)。

* [Java 开发人员工具包 (JDK) 版本 8](https://aka.ms/azure-jdks)。

* 根据 Apache 要求正确[安装](https://maven.apache.org/install.html)的 [Apache Maven](https://maven.apache.org/download.cgi)。  Maven 是 Java 项目的项目生成系统。

* SSH 客户端。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

* 如果使用 PowerShell，则需要[AZ 模块](https://docs.microsoft.com/powershell/azure/overview)。

* 文本编辑器。 本文使用 Microsoft 记事本。

> [!IMPORTANT]  
> Azure PowerShell cmdlet [Get AzHDInsightCluster](https://docs.microsoft.com/powershell/module/az.hdinsight/get-azhdinsightcluster)并[Get AzHDInsightJobOutput](https://docs.microsoft.com/powershell/module/az.hdinsight/get-azhdinsightjoboutput)当前不起作用时[安全传输](../../storage/common/storage-require-secure-transfer.md)上的存储帐户启用.

## <a name="test-environment"></a>测试环境
本文使用的环境是一台运行 Windows 10 的计算机。  命令在命令提示符下执行，各种文件使用记事本进行编辑。 相应地修改您的环境。

在命令提示符下，输入以下命令以创建工作环境：

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>创建 Maven 项目

1. 输入以下命令以创建一个名为的 Maven 项目**hbaseapp**:

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    cd hbaseapp
    mkdir conf
    ```

    此命令会在当前位置创建名为 `hbaseapp` 的目录，其中包含基本 Maven 项目。 第二个命令将更改到的工作目录`hbaseapp`。 第三条命令创建稍后要使用的新目录 `conf`。 `hbaseapp` 目录包含以下项：

    * `pom.xml`：项目对象模型 (](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)POM)，其中包含用于生成项目的信息和配置详细信息。
    * `src\main\java\com\microsoft\examples`：包含应用程序代码。
    * `src\test\java\com\microsoft\examples`：包含应用程序的测试。

2. 删除生成的示例代码。 输入以下命令，删除生成的测试和应用程序文件 `AppTest.java` 与 `App.java`：

    ```cmd
    DEL src\main\java\com\microsoft\examples\App.java
    DEL src\test\java\com\microsoft\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>更新项目对象模型

Pom.xml 文件的完整参考，请参阅 https://maven.apache.org/pom.html。  输入以下命令打开 `pom.xml`：

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>添加依赖项

在 `pom.xml` 的 `<dependencies>` 节中添加以下文本：

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.14.1-HBase-1.1</version>
</dependency>
```  

此部分指示项目需要 **hbase-client** 和 **phoenix-core** 组件。 在编译时，会从默认 Maven 存储库下载这些依赖项。 可以使用 [Maven 中央存储库](https://search.maven.org/artifact/org.apache.hbase/hbase-client/1.1.2/jar)搜索来了解有关此依赖项的详细信息。

> [!IMPORTANT]  
> hbase-client 的版本号必须与 HDInsight 群集随附的 Apache HBase 版本匹配。 可以使用下表来查找正确的版本号。

| HDInsight 群集版本 | 要使用的 Apache HBase 版本 |
| --- | --- |
| 3.6 | 1.1.2 |
| 4.0 | 2.0.0 |

有关 HDInsight 版本和组件的详细信息，请参阅 [HDInsight 提供哪些不同的 Apache Hadoop 组件](../hdinsight-component-versioning.md)。

### <a name="build-configuration"></a>生成配置

Maven 插件允许自定义项目的构建阶段。 此节用于添加插件、资源和其他生成配置选项。

将以下代码添加到`pom.xml`文件，然后保存并关闭该文件。 此文本必须位于文件中的 `<project>...</project>` 标记内，例如 `</dependencies>` 和 `</project>` 之间。

```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <resources>
    <resource>
        <directory>${basedir}/conf</directory>
        <filtering>false</filtering>
        <includes>
        <include>hbase-site.xml</include>
        </includes>
    </resource>
    </resources>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.1</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
            <goals>
            <goal>shade</goal>
            </goals>
        </execution>
        </executions>
    </plugin>
    </plugins>
</build>
```

此部分将配置包含与 HBase 有关的配置信息的资源 (`conf/hbase-site.xml`)。

> [!NOTE]  
> 也可以通过代码设置配置值。 请参阅 `CreateTable` 示例中的注释。

此部分还将配置 [Apache Maven 编译器插件](https://maven.apache.org/plugins/maven-compiler-plugin/)和 [Apache Maven 阴影插件](https://maven.apache.org/plugins/maven-shade-plugin/)。 该编译器插件用于编译拓扑。 该阴影插件用于防止在由 Maven 构建的 JAR 程序包中复制许可证。 此插件用于防止 HDInsight 群集在运行时出现“重复的许可证文件”错误。 将 maven-shade-plugin 用于 `ApacheLicenseResourceTransformer` 实现可防止发生此错误。

maven-shade-plugin 还会生成 uber jar，其中包含应用程序所需的所有依赖项。

### <a name="download-the-hbase-sitexml"></a>下载 hbase-site.xml

使用以下命令将 HBase 配置从 HDInsight 群集复制到 `conf` 目录。 替换为`CLUSTERNAME`与你的 HDInsight 群集名称，然后输入命令：

```cmd
scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml
```

## <a name="create-the-application"></a>创建应用程序

### <a name="implement-a-createtable-class"></a>实现 CreateTable 类

输入以下命令以创建并打开一个新文件`CreateTable.java`。 选择**是**在提示符下，若要创建新的文件。

```cmd
notepad src\main\java\com\microsoft\examples\CreateTable.java
```

将以下 Java 代码复制并粘贴到新文件中。 然后关闭该文件。

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

public class CreateTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Example of setting zookeeper values for HDInsight
    // in code instead of an hbase-site.xml file
    //
    // config.set("hbase.zookeeper.quorum",
    //            "zookeepernode0,zookeepernode1,zookeepernode2");
    //config.set("hbase.zookeeper.property.clientPort", "2181");
    //config.set("hbase.cluster.distributed", "true");
    //
    //NOTE: Actual zookeeper host names can be found using Ambari:
    //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts"

    //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
    config.set("zookeeper.znode.parent","/hbase-unsecure");

    // create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // create the table...
    HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
    // ... with two column families
    tableDescriptor.addFamily(new HColumnDescriptor("name"));
    tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
    admin.createTable(tableDescriptor);

    // define some people
    String[][] people = {
        { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
        { "2", "Franklin", "Holtz", "franklin@contoso.com" },
        { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
        { "4", "Rae", "Schroeder", "rae@contoso.com" },
        { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
        { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

    HTable table = new HTable(config, "people");

    // Add each person to the table
    //   Use the `name` column family for the name
    //   Use the `contactinfo` column family for the email
    for (int i = 0; i< people.length; i++) {
        Put person = new Put(Bytes.toBytes(people[i][0]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
        person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
        table.put(person);
    }
    // flush commits and close the table
    table.flushCommits();
    table.close();
    }
}
```

此代码是`CreateTable`类，该类将创建名为的表`people`并使用一些预定义的用户填充它。

### <a name="implement-a-searchbyemail-class"></a>实现 SearchByEmail 类

输入以下命令以创建并打开一个新文件`SearchByEmail.java`。 选择**是**在提示符下，若要创建新的文件。

```cmd
notepad src\main\java\com\microsoft\examples\SearchByEmail.java
```

将以下 Java 代码复制并粘贴到新文件中。 然后关闭该文件。

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.filter.RegexStringComparator;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.util.GenericOptionsParser;

public class SearchByEmail {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Use GenericOptionsParser to get only the parameters to the class
    // and not all the parameters passed (when using WebHCat for example)
    String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
    if (otherArgs.length != 1) {
        System.out.println("usage: [regular expression]");
        System.exit(-1);
    }

    // Open the table
    HTable table = new HTable(config, "people");

    // Define the family and qualifiers to be used
    byte[] contactFamily = Bytes.toBytes("contactinfo");
    byte[] emailQualifier = Bytes.toBytes("email");
    byte[] nameFamily = Bytes.toBytes("name");
    byte[] firstNameQualifier = Bytes.toBytes("first");
    byte[] lastNameQualifier = Bytes.toBytes("last");

    // Create a regex filter
    RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
    // Attach the regex filter to a filter
    //   for the email column
    SingleColumnValueFilter filter = new SingleColumnValueFilter(
        contactFamily,
        emailQualifier,
        CompareOp.EQUAL,
        emailFilter
    );

    // Create a scan and set the filter
    Scan scan = new Scan();
    scan.setFilter(filter);

    // Get the results
    ResultScanner results = table.getScanner(scan);
    // Iterate over results and print  values
    for (Result result : results ) {
        String id = new String(result.getRow());
        byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
        String firstName = new String(firstNameObj);
        byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
        String lastName = new String(lastNameObj);
        System.out.println(firstName + " " + lastName + " - ID: " + id);
        byte[] emailObj = result.getValue(contactFamily, emailQualifier);
        String email = new String(emailObj);
        System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
    }
    results.close();
    table.close();
    }
}
```

`SearchByEmail`类可用于查询按电子邮件地址的行。 由于它使用正则表达式筛选器，因此，可以在使用类时提供字符串或正则表达式。

### <a name="implement-a-deletetable-class"></a>实现 DeleteTable 类

输入以下命令以创建并打开一个新文件`DeleteTable.java`。 选择**是**在提示符下，若要创建新的文件。

```cmd
notepad src\main\java\com\microsoft\examples\DeleteTable.java
```

将以下 Java 代码复制并粘贴到新文件中。 然后关闭该文件。

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;

public class DeleteTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // Disable, and then delete the table
    admin.disableTable("people");
    admin.deleteTable("people");
    }
}
```

`DeleteTable`类用于清除在此示例通过禁用并删除创建的表创建的 HBase 表`CreateTable`类。

## <a name="build-and-package-the-application"></a>生成并打包应用程序

1. 在 `hbaseapp` 目录中，使用以下命令来构建包含应用程序的 JAR 文件：

    ```cmd
    mvn clean package
    ```

    此命令构建应用程序并将其打包到一个 .jar 文件中。

2. 命令完成之后，`hbaseapp/target` 目录包含一个名为 `hbaseapp-1.0-SNAPSHOT.jar` 的文件。

   > [!NOTE]  
   > `hbaseapp-1.0-SNAPSHOT.jar` 文件是一个 uber jar。 它包含运行应用程序所需的所有依赖项。

## <a name="upload-the-jar-and-run-jobs-ssh"></a>上传 JAR 并运行作业 (SSH)

以下步骤使用 `scp` 将 JAR 复制到 Apache HBase on HDInsight 群集的主要头节点。 然后使用 `ssh` 命令连接到群集并直接在头节点上运行示例。

1. 将该 jar 文件上传到群集。 替换为`CLUSTERNAME`与你的 HDInsight 群集名称，然后输入以下命令：

    ```cmd
    scp ./target/hbaseapp-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:hbaseapp-1.0-SNAPSHOT.jar
    ```

2. 连接到 HBase 群集。 替换为`CLUSTERNAME`与你的 HDInsight 群集名称，然后输入以下命令：

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

 3. 若要创建 HBase 表使用的 Java 应用程序，使用以下命令在你打开 ssh 连接：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable
    ```

    此命令会创建名为 **people** 的一个 HBase 表，并在其中填充数据。

4. 若要搜索表中存储的电子邮件地址，请使用以下命令：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com
    ```

    将生成以下结果：

        Franklin Holtz - ID: 2
        Franklin Holtz - franklin@contoso.com - ID: 2
        Rae Schroeder - ID: 4
        Rae Schroeder - rae@contoso.com - ID: 4
        Gabriela Ingram - ID: 6
        Gabriela Ingram - gabriela@contoso.com - ID: 6

5. 若要删除表，请使用以下命令：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable
    ```

## <a name="upload-the-jar-and-run-jobs-powershell"></a>上传 JAR 并运行作业 (PowerShell)

以下步骤使用 Azure PowerShell [AZ 模块](https://docs.microsoft.com/powershell/azure/new-azureps-module-az)若要将该 jar 文件上传到你的 Apache HBase 群集的默认存储。 然后使用 HDInsight cmdlet 远程运行示例。

1. 安装和配置 AZ 模块中，创建名为的文件后`hbase-runner.psm1`。 将以下文本用作此文件的内容：

   ```powershell
    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.CreateTable"
    -clusterName "MyHDInsightCluster"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "contoso.com"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "^r" -showErr
    #>

    function Start-HBaseExample {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
    #The class to run
    [Parameter(Mandatory = $true)]
    [String]$className,

    #The name of the HDInsight cluster
    [Parameter(Mandatory = $true)]
    [String]$clusterName,

    #Only used when using SearchByEmail
    [Parameter(Mandatory = $false)]
    [String]$emailRegex,

    #Use if you want to see stderr output
    [Parameter(Mandatory = $false)]
    [Switch]$showErr
    )

    Set-StrictMode -Version 3

    # Is the Azure module installed?
    FindAzure

    # Get the login for the HDInsight cluster
    $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

    # The JAR
    $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

    # The job definition
    $jobDefinition = New-AzHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName $className `
        -Arguments $emailRegex

    # Get the job output
    $job = Start-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
    if($showErr)
    {
    Write-Host "STDERR"
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds `
                -DisplayOutputType StandardError
    }
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds
    }

    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    -Container "MyContainer"
    #>

    function Add-HDInsightFile {
        [CmdletBinding(SupportsShouldProcess = $true)]
        param(
            #The path to the local file.
            [Parameter(Mandatory = $true)]
            [String]$localPath,

            #The destination path and file name, relative to the root of the container.
            [Parameter(Mandatory = $true)]
            [String]$destinationPath,

            #The name of the HDInsight cluster
            [Parameter(Mandatory = $true)]
            [String]$clusterName,

            #If specified, overwrites existing files without prompting
            [Parameter(Mandatory = $false)]
            [Switch]$force
        )

        Set-StrictMode -Version 3

        # Is the Azure module installed?
        FindAzure

        # Get authentication for the cluster
        $creds=Get-Credential

        # Does the local path exist?
        if (-not (Test-Path $localPath))
        {
            throw "Source path '$localPath' does not exist."
        }

        # Get the primary storage container
        $storage = GetStorage -clusterName $clusterName

        # Upload file to storage, overwriting existing files if -force was used.
        Set-AzStorageBlobContent -File $localPath `
            -Blob $destinationPath `
            -force:$force `
            -Container $storage.container `
            -Context $storage.context
    }

    function FindAzure {
        # Is there an active Azure subscription?
        $sub = Get-AzSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Connect-AzAccount
        }
    }

    function GetStorage {
        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName
        )
        $hdi = Get-AzHDInsightCluster -ClusterName $clusterName
        # Does the cluster exist?
        if (!$hdi)
        {
            throw "HDInsight cluster '$clusterName' does not exist."
        }
        # Create a return object for context & container
        $return = @{}
        $storageAccounts = @{}

        # Get storage information
        $resourceGroup = $hdi.ResourceGroup
        $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
        $container=$hdi.DefaultStorageContainer
        $storageAccountKey=(Get-AzStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
        # Get the resource group, in case we need that
        $return.resourceGroup = $resourceGroup
        # Get the storage context, as we can't depend
        # on using the default storage context
        $return.context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
        # Get the container, so we know where to
        # find/store blobs
        $return.container = $container
        # Return storage accounts to support finding all accounts for
        # a cluster
        $return.storageAccount = $storageAccountName
        $return.storageAccountKey = $storageAccountKey

        return $return
    }
    # Only export the verb-phrase things
    export-modulemember *-*
   ```

    此文件包含两个模块：

   * **Add-HDInsightFile** - 用于将文件上传到群集
   * **Start-HBaseExample** - 用于运行以前创建的类

2. 保存`hbase-runner.psm1`文件中`hbaseapp`目录。

3. 使用 Azure PowerShell 注册模块。 打开新的 Azure PowerShell 窗口并通过替换来编辑下面的命令`CLUSTERNAME`与群集的名称。 然后，输入以下命令：

    ```powershell
    cd C:\HDI\hbaseapp
    $myCluster = "CLUSTERNAME"
    Import-Module .\hbase-runner.psm1
    ```

4. 使用以下命令将 `hbaseapp-1.0-SNAPSHOT.jar` 上传到群集。

    ```powershell
    Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName $myCluster
    ```

    出现提示时，输入群集登录名 (admin) 和密码。 此命令将 `hbaseapp-1.0-SNAPSHOT.jar` 上传到群集的主存储中的 `example/jars` 位置。

5. 若要使用 `hbaseapp` 创建表，请使用以下命令：

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName $myCluster
    ```

    出现提示时，输入群集登录名 (admin) 和密码。

    此命令会在 HDInsight 群集上的 HBase 中创建一个名为 **people** 的表。 此命令在控制台窗口中不显示任何输出。

6. 若要在表中搜索条目，请使用以下命令：

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName $myCluster -emailRegex contoso.com
    ```

    出现提示时，输入群集登录名 (admin) 和密码。

    此命令使用 `SearchByEmail` 类搜索 `contactinformation` 列系列和 `email` 列包含字符串 `contoso.com` 的任何行。 应该会收到以下结果：

          Franklin Holtz - ID: 2
          Franklin Holtz - franklin@contoso.com - ID: 2
          Rae Schroeder - ID: 4
          Rae Schroeder - rae@contoso.com - ID: 4
          Gabriela Ingram - ID: 6
          Gabriela Ingram - gabriela@contoso.com - ID: 6

    将 **fabrikam.com** 用于 `-emailRegex` 值会返回电子邮件字段中包含 **fabrikam.com** 的用户。 还可以使用正则表达式作为搜索词。 例如，**^r** 返回以字母“r”开头的电子邮件地址。

7. 若要删除表，请使用以下命令：

    ```PowerShell
    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName $myCluster
    ```

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>使用 Start-HBaseExample 时无结果或意外结果

使用 `-showErr` 参数可查看运行作业时生成的标准错误 (STDERR)。

## <a name="next-steps"></a>后续步骤

[了解如何将 SQuirreL SQL 与 Apache HBase 配合使用](apache-hbase-phoenix-squirrel-linux.md)
