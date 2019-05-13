---
title: SQL 数据库的 XEvent 事件文件代码 | Microsoft 文档
description: 提供一个双阶段代码示例的 PowerShell 和 Transact-SQL，该示例演示 Azure SQL 数据库的扩展事件中的事件文件目标。 完成此方案部分必须用到 Azure 存储空间。
services: sql-database
ms.service: sql-database
ms.subservice: monitor
ms.custom: ''
ms.devlang: PowerShell
ms.topic: conceptual
author: MightyPen
ms.author: genemi
ms.reviewer: jrasnik
manager: craigg
ms.date: 03/12/2019
ms.openlocfilehash: ce559e50d5a34ebad9113f0e21dcb732adc40dd2
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233793"
---
# <a name="event-file-target-code-for-extended-events-in-sql-database"></a>SQL 数据库中扩展事件的事件文件目标代码

[!INCLUDE [sql-database-xevents-selectors-1-include](../../includes/sql-database-xevents-selectors-1-include.md)]

需要完整的代码示例来可靠捕获和报告扩展事件的信息。

在 Microsoft SQL Server 中，[事件文件目标](https://msdn.microsoft.com/library/ff878115.aspx)用于将事件输出存储在本地硬盘驱动器文件中。 但是，此类文件并不适用于 Azure SQL 数据库。 我们改为使用 Azure 存储空间服务来支持事件文件目标。

本主题演示了一个两阶段代码示例：

* PowerShell：用于在云中创建 Azure 存储空间容器。
* Transact-SQL：
  
  * 将 Azure 存储容器分配到事件文件目标。
  * 创建和启动事件会话，等等。

## <a name="prerequisites"></a>必备组件

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> PowerShell Azure 资源管理器模块仍受 Azure SQL 数据库的支持，但所有未来的开发都是针对 Az.Sql 模块的。 若要了解这些 cmdlet，请参阅 [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/)。 Az 模块和 AzureRm 模块中的命令参数大体上是相同的。

* Azure 帐户和订阅。 可以注册[免费试用版](https://azure.microsoft.com/pricing/free-trial/)。
* 可以在其中创建表的任何数据库。
  
  * 或者，也可以快速[创建一个 **AdventureWorksLT** 演示数据库](sql-database-get-started.md)。
* SQL Server Management Studio (ssms.exe)，最好是每月更新版。 
  可从以下位置下载最新的 ssms.exe：
  
  * 标题为[下载 SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 的主题。
  * [直接指向下载位置的链接。](https://go.microsoft.com/fwlink/?linkid=616025)
* 必须安装 [Azure PowerShell 模块](https://go.microsoft.com/?linkid=9811175)。
  
  * 这些模块提供 **New-AzStorageAccount** 等命令。

## <a name="phase-1-powershell-code-for-azure-storage-container"></a>阶段 1：Azure 存储容器的 PowerShell 代码

此 PowerShell 是两阶段代码示例的第 1 阶段。

脚本开头的命令将清除以前可能运行后的结果，并且可重复运行。

1. 将 PowerShell 脚本粘贴到 Notepad.exe 等简单的文本编辑器中，并将脚本保存为扩展名为 **.ps1** 的文件。
2. 以管理员身份启动 PowerShell ISE。
3. 在提示符下键入<br/>`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`<br/>然后按 Enter。
4. 在 PowerShell ISE 中打开 **.ps1** 文件。 运行该脚本。
5. 该脚本先启动一个新窗口，供你在其中登录到 Azure。
   
   * 如果想要重新运行脚本而不中断会话，可以很方便地选择注释禁止 **Add-AzureAccount** 命令。

![在准备运行脚本之前，必须准备好已装有 Azure 模块的 PowerShell ISE。][30_powershell_ise]

### <a name="powershell-code"></a>PowerShell 代码

此 PowerShell 脚本假定你已安装 Az 模块。 有关信息，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。

```powershell
## TODO: Before running, find all 'TODO' and make each edit!!

cls;

#--------------- 1 -----------------------

'Script assumes you have already logged your PowerShell session into Azure.
But if not, run  Connect-AzAccount (or  Connect-AzAccount), just one time.';
#Connect-AzAccount;   # Same as  Connect-AzAccount.

#-------------- 2 ------------------------

'
TODO: Edit the values assigned to these variables, especially the first few!
';

# Ensure the current date is between
# the Expiry and Start time values that you edit here.

$subscriptionName    = 'YOUR_SUBSCRIPTION_NAME';
$resourceGroupName   = 'YOUR_RESOURCE-GROUP-NAME';

$policySasExpiryTime = '2018-08-28T23:44:56Z';
$policySasStartTime  = '2017-10-01';

$storageAccountLocation = 'YOUR_STORAGE_ACCOUNT_LOCATION';
$storageAccountName     = 'YOUR_STORAGE_ACCOUNT_NAME';
$contextName            = 'YOUR_CONTEXT_NAME';
$containerName          = 'YOUR_CONTAINER_NAME';
$policySasToken         = ' ? ';

$policySasPermission = 'rwl';  # Leave this value alone, as 'rwl'.

#--------------- 3 -----------------------

# The ending display lists your Azure subscriptions.
# One should match the $subscriptionName value you assigned
#   earlier in this PowerShell script. 

'Choose an existing subscription for the current PowerShell environment.';

Select-AzSubscription -Subscription $subscriptionName;

#-------------- 4 ------------------------

'
Clean up the old Azure Storage Account after any previous run, 
before continuing this new run.';

If ($storageAccountName)
{
    Remove-AzStorageAccount `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName;
}

#--------------- 5 -----------------------

[System.DateTime]::Now.ToString();

'
Create a storage account. 
This might take several minutes, will beep when ready.
  ...PLEASE WAIT...';

New-AzStorageAccount `
    -Name              $storageAccountName `
    -Location          $storageAccountLocation `
    -ResourceGroupName $resourceGroupName `
    -SkuName           'Standard_LRS';

[System.DateTime]::Now.ToString();
[System.Media.SystemSounds]::Beep.Play();

'
Get the access key for your storage account.
';

$accessKey_ForStorageAccount = `
    (Get-AzStorageAccountKey `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName
        ).Value[0];

"`$accessKey_ForStorageAccount = $accessKey_ForStorageAccount";

'Azure Storage Account cmdlet completed.
Remainder of PowerShell .ps1 script continues.
';

#--------------- 6 -----------------------

# The context will be needed to create a container within the storage account.

'Create a context object from the storage account and its primary access key.
';

$context = New-AzStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey  $accessKey_ForStorageAccount;

'Create a container within the storage account.
';

$containerObjectInStorageAccount = New-AzStorageContainer `
    -Name    $containerName `
    -Context $context;

'Create a security policy to be applied to the SAS token.
';

New-AzStorageContainerStoredAccessPolicy `
    -Container  $containerName `
    -Context    $context `
    -Policy     $policySasToken `
    -Permission $policySasPermission `
    -ExpiryTime $policySasExpiryTime `
    -StartTime  $policySasStartTime;

'
Generate a SAS token for the container.
';
Try
{
    $sasTokenWithPolicy = New-AzStorageContainerSASToken `
        -Name    $containerName `
        -Context $context `
        -Policy  $policySasToken;
}
Catch 
{
    $Error[0].Exception.ToString();
}

#-------------- 7 ------------------------

'Display the values that YOU must edit into the Transact-SQL script next!:
';

"storageAccountName: $storageAccountName";
"containerName:      $containerName";
"sasTokenWithPolicy: $sasTokenWithPolicy";

'
REMINDER: sasTokenWithPolicy here might start with "?" character, which you must exclude from Transact-SQL.
';

'
(Later, return here to delete your Azure Storage account. See the preceding  Remove-AzStorageAccount -Name $storageAccountName)';

'
Now shift to the Transact-SQL portion of the two-part code sample!';

# EOFile
```


记下 PowerShell 脚本结束时输出的几个命名值。 必须将这些值编辑成阶段 2 中使用的 Transact-SQL 脚本。

## <a name="phase-2-transact-sql-code-that-uses-azure-storage-container"></a>阶段 2：使用 Azure 存储容器的 Transact-SQL 代码

* 在此代码示例的第 1 阶段，已运行 PowerShell 脚本来创建 Azure 存储容器。
* 接下来在第 2 阶段，以下 Transact-SQL 脚本必须使用该容器。

脚本以用于在可能的上次运行后进行清理的命令开头，且可重复运行。

PowerShell 脚本在结束时输出了几个命名值。 必须编辑 Transact-SQL 脚本才能使用这些值。 在 Transact-SQL 脚本中查找 **TODO** 以找到编辑点。

1. 打开 SQL Server Management Studio (ssms.exe)。
2. 连接到 Azure SQL 数据库。
3. 单击打开新的查询窗格。
4. 将以下 Transact-SQL 脚本粘贴到查询窗格中。
5. 在脚本中查找每个 **TODO** 并进行适当的编辑。
6. 保存并运行该脚本。


> [!WARNING]
> 之前 PowerShell 脚本生成的 SAS 密钥值可能以“?”（问号）开头。 在以下 T-SQL 脚本中使用 SAS 密钥时，必须*删除前导“?”*。 否则，安全性可能会阻止操作。


### <a name="transact-sql-code"></a>Transact-SQL 代码

```sql
---- TODO: First, run the earlier PowerShell portion of this two-part code sample.
---- TODO: Second, find every 'TODO' in this Transact-SQL file, and edit each.

---- Transact-SQL code for Event File target on Azure SQL Database.


SET NOCOUNT ON;
GO


----  Step 1.  Establish one little table, and  ---------
----  insert one row of data.


IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'gmTabEmployee')
BEGIN
    DROP TABLE gmTabEmployee;
END
GO


CREATE TABLE gmTabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO gmTabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO


------  Step 2.  Create key, and  ------------
------  Create credential (your Azure Storage container must already exist).


IF NOT EXISTS
    (SELECT * FROM sys.symmetric_keys
        WHERE symmetric_key_id = 101)
BEGIN
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '0C34C960-6621-4682-A123-C7EA08E3FC46' -- Or any newid().
END
GO


IF EXISTS
    (SELECT * FROM sys.database_scoped_credentials
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        WHERE name = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent')
BEGIN
    DROP DATABASE SCOPED CREDENTIAL
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent] ;
END
GO


CREATE
    DATABASE SCOPED
    CREDENTIAL
        -- use '.blob.',   and not '.queue.' or '.table.' etc.
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    WITH
        IDENTITY = 'SHARED ACCESS SIGNATURE',  -- "SAS" token.
        -- TODO: Paste in the long SasToken string here for Secret, but exclude any leading '?'.
        SECRET = 'sv=2014-02-14&sr=c&si=gmpolicysastoken&sig=EjAqjo6Nu5xMLEZEkMkLbeF7TD9v1J8DNB2t8gOKTts%3D'
    ;
GO


------  Step 3.  Create (define) an event session.  --------
------  The event session has an event with an action,
------  and a has a target.

IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'gmeventsessionname240b')
BEGIN
    DROP
        EVENT SESSION
            gmeventsessionname240b
        ON DATABASE;
END
GO


CREATE
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE

    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE 'UPDATE gmTabEmployee%'
            )
    ADD TARGET
        package0.event_file
            (
            -- TODO: Assign AzureStorageAccount name, and the associated Container name.
            -- Also, tweak the .xel file name at end, if you like.
            SET filename =
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
            )
    WITH
        (MAX_MEMORY = 10 MB,
        MAX_DISPATCH_LATENCY = 3 SECONDS)
    ;
GO


------  Step 4.  Start the event session.  ----------------
------  Issue the SQL Update statements that will be traced.
------  Then stop the session.

------  Note: If the target fails to attach,
------  the session must be stopped and restarted.

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = START;
GO


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
GO


ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = STOP;
GO


-------------- Step 5.  Select the results. ----------

SELECT
        *, 'CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS!' as [CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS],
        CAST(event_data AS XML) AS [event_data_XML]  -- TODO: In ssms.exe results grid, double-click this cell!
    FROM
        sys.fn_xe_file_target_read_file
            (
                -- TODO: Fill in Storage Account name, and the associated Container name.
                -- TODO: The name of the .xel file needs to be an exact match to the files in the storage account Container (You can use Storage Account explorer from the portal to find out the exact file names or you can retrieve the name using the following DMV-query: select target_data from sys.dm_xe_database_session_targets. The 3rd xml-node, "File name", contains the name of the file currently written to.)
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b',
                null, null, null
            );
GO


-------------- Step 6.  Clean up. ----------

DROP
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE;
GO

DROP DATABASE SCOPED CREDENTIAL
    -- TODO: Assign AzureStorageAccount name, and the associated Container name.
    [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    ;
GO

DROP TABLE gmTabEmployee;
GO

PRINT 'Use PowerShell Remove-AzStorageAccount to delete your Azure Storage account!';
GO
```


如果运行脚本时无法附加目标，必须停止再重新启动事件会话：

```sql
ALTER EVENT SESSION ... STATE = STOP;
GO
ALTER EVENT SESSION ... STATE = START;
GO
```


## <a name="output"></a>输出

完成 Transact-SQL 脚本后，请单击 **event_data_XML** 列标题下的单元格。 一个**\<事件 >** 元素，系统会显示一条 UPDATE 语句。

下面是一个**\<事件 >** 测试期间生成的元素：


```xml
<event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T19:18:45.420Z">
  <data name="state">
    <value>0</value>
    <text>Normal</text>
  </data>
  <data name="line_number">
    <value>5</value>
  </data>
  <data name="offset">
    <value>148</value>
  </data>
  <data name="offset_end">
    <value>368</value>
  </data>
  <data name="statement">
    <value>UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe'</value>
  </data>
  <action name="sql_text" package="sqlserver">
    <value>

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
</value>
  </action>
</event>
```


前面的 Transact-SQL 脚本使用以下系统函数来读取 event_file：

* [sys.fn_xe_file_target_read_file (Transact-SQL)](https://msdn.microsoft.com/library/cc280743.aspx)

用于查看扩展事件数据的高级选项的说明可在此处获取：

* [扩展事件的目标数据的高级视图](https://msdn.microsoft.com/library/mt752502.aspx)


## <a name="converting-the-code-sample-to-run-on-sql-server"></a>转换代码示例以在 SQL Server 上运行

假设要在 Microsoft SQL Server 上运行上述 Transact-SQL 示例。

* 为简单起见，会想要将 Azure 存储容器完全替换为一个简单文件（例如 **C:\myeventdata.xel**）。 该文件将写入托管 SQL Server 的计算机的本地硬盘驱动器上。
* 不需要为 **CREATE MASTER KEY** 和 **CREATE CREDENTIAL** 使用任何类型的 Transact-SQL 语句。
* 在 **CREATE EVENT SESSION** 语句的 **ADD TARGET** 子句中，将对 **filename=** 分配的 Http 值替换为完整路径字符串（例如 **C:\myfile.xel**）。
  
  * 此操作不涉及任何 Azure 存储帐户。

## <a name="more-information"></a>详细信息

有关 Azure 存储空间服务中帐户和容器的详细信息，请参阅：

* [如何通过 .NET 使用 Blob 存储](../storage/blobs/storage-dotnet-how-to-use-blobs.md)
* [命名和引用容器、Blob 与元数据](https://msdn.microsoft.com/library/azure/dd135715.aspx)
* [使用根容器](https://msdn.microsoft.com/library/azure/ee395424.aspx)
* [第 1 课：在 Azure 容器上创建存储访问策略和共享访问签名](https://msdn.microsoft.com/library/dn466430.aspx)
  * [第 2 课：使用共享访问签名创建 SQL Server 凭据](https://msdn.microsoft.com/library/dn466435.aspx)
* [Microsoft SQL Server 扩展事件](https://docs.microsoft.com/sql/relational-databases/extended-events/extended-events)

<!--
Image references.
-->

[30_powershell_ise]: ./media/sql-database-xevent-code-event-file/event-file-powershell-ise-b30.png

