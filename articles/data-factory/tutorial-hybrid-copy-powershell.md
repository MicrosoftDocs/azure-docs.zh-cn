---
title: 使用 Azure 数据工厂将数据从 SQL Server 复制到 BLob 存储 | Microsoft Docs
description: 了解如何在 Azure 数据工厂中使用自承载 Integration Runtime 将数据从本地数据存储复制到 Azure 云。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: abnarain
ms.openlocfilehash: 49d9be9f10f0e840cfa3d027901a297de8cbf750
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60328075"
---
# <a name="tutorial-copy-data-from-an-on-premises-sql-server-database-to-azure-blob-storage"></a>教程：将数据从本地 SQL Server 数据库复制到 Azure Blob 存储
本教程使用 Azure PowerShell 创建一个数据工厂管道，用于将数据从本地 SQL Server 数据库复制到 Azure Blob 存储。 同时创建一个自承载 Integration Runtime，用其在本地数据存储和云数据存储之间移动数据。 

> [!NOTE]
> 本文不提供数据工厂服务的详细介绍。 有关详细信息，请参阅 [Azure 数据工厂简介](introduction.md)。 

将在本教程中执行以下步骤：

> [!div class="checklist"]
> * 创建数据工厂。
> * 创建自我托管的集成运行时。
> * 创建 SQL Server 和 Azure 存储链接服务。 
> * 创建 SQL Server 和 Azure Blob 数据集。
> * 创建包含复制活动的管道，用于移动数据。
> * 启动管道运行。
> * 监视管道运行。

## <a name="prerequisites"></a>先决条件
### <a name="azure-subscription"></a>Azure 订阅
如果还没有 Azure 订阅，请在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

### <a name="azure-roles"></a>Azure 角色
若要创建数据工厂实例，用于登录到 Azure 的用户帐户必须分配有“参与者”或“所有者”角色，或者必须是 Azure 订阅的管理员。 

若要查看你在订阅中拥有的权限，请转到 Azure 门户，选择右上角的用户名，然后选择“权限”。 如果可以访问多个订阅，请选择相应的订阅。 有关如何将用户添加到角色的示例说明，请参阅[使用 RBAC 和 Azure 门户管理访问权限](../role-based-access-control/role-assignments-portal.md)一文。

### <a name="sql-server-2014-2016-and-2017"></a>SQL Server 2014、2016 和 2017
在本教程中，请将本地 SQL Server 数据库用作源数据存储。 首先，在本教程中创建的数据工厂中的管道将数据从这个本地 SQL Server 数据库（源）复制到 Azure Blob 存储（接收器）。 然后，你可以在 SQL Server 数据库中创建名为 **emp** 的表，并向表中插入几个示例条目。 

1. 启动 SQL Server Management Studio。 如果此软件尚未安装在计算机上，请访问[下载 SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。 

1. 使用凭据连接到 SQL Server 实例。 

1. 创建示例数据库。 在树状视图中右键单击“数据库”，然后选择“新建数据库”。 
 
1. 在“新建数据库”窗口中输入数据库的名称，然后选择“确定”。 

1. 若要创建 **emp** 表并将一些示例数据插入到其中，请对数据库运行以下查询脚本：

   ```
       INSERT INTO emp VALUES ('John', 'Doe')
       INSERT INTO emp VALUES ('Jane', 'Doe')
       GO
   ```

1. 在树状视图中右键单击所创建的数据库，然后选择“新建查询”。

### <a name="azure-storage-account"></a>Azure 存储帐户
在本教程中，请使用常规用途的 Azure 存储帐户（具体说来就是 Azure Blob 存储）作为目标/接收器数据存储。 如果没有常规用途的 Azure 存储帐户，请参阅[创建存储帐户](../storage/common/storage-quickstart-create-account.md)。 在本教程中创建的数据工厂中的管道将数据从本地 SQL Server 数据库（源）复制到这个 Azure Blob 存储（接收器）。 

#### <a name="get-storage-account-name-and-account-key"></a>获取存储帐户名称和帐户密钥
在本教程中，请使用 Azure 存储帐户的名称和密钥。 通过以下步骤获取存储帐户的名称和密钥： 

1. 使用 Azure 用户名和密码登录到 [Azure 门户](https://portal.azure.com)。 

1. 在左窗格中选择“更多服务”，使用“存储”关键字进行筛选，然后选择“存储帐户”。

    ![搜索存储帐户](media/tutorial-hybrid-copy-powershell/search-storage-account.png)

1. 在存储帐户列表中，通过筛选找出你的存储帐户（如果需要），然后选择你的存储帐户。 

1. 在“存储帐户”窗口中选择“访问密钥”。

    ![获取存储帐户名称和密钥](media/tutorial-hybrid-copy-powershell/storage-account-name-key.png)

1. 复制“存储帐户名称”和“key1”框中的值，然后将其粘贴到记事本或其他编辑器中，在教程的后面部分使用。 

#### <a name="create-the-adftutorial-container"></a>创建 adftutorial 容器 
此部分在 Azure Blob 存储中创建名为 **adftutorial** 的 Blob 容器。 

1. 在“存储帐户”窗口中切换到“概览”，然后选择“Blob”。 

    ![选择“Blob”选项](media/tutorial-hybrid-copy-powershell/select-blobs.png)

1. 在“Blob 服务”窗口中，选择“容器”。 

    ![“添加容器”按钮](media/tutorial-hybrid-copy-powershell/add-container-button.png)

1. 在“新建容器”窗口的“名称”框中输入“adftutorial”，然后选择“确定”。 

    ![输入容器名称](media/tutorial-hybrid-copy-powershell/new-container-dialog.png)

1. 在容器列表中选择“adftutorial”。  

    ![选择容器](media/tutorial-hybrid-copy-powershell/select-adftutorial-container.png)

1. 让 **adftutorial** 的“容器”窗口保持打开状态。 在教程结束时可以使用它来验证输出。 数据工厂自动在该容器中创建输出文件夹，因此不需要你来创建。

    ![“容器”窗口](media/tutorial-hybrid-copy-powershell/container-page.png)

### <a name="windows-powershell"></a>Windows PowerShell

#### <a name="install-azure-powershell"></a>安装 Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

安装最新版的 Azure PowerShell（如果尚未在计算机上安装）。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/powershell/azure/install-Az-ps)。 

#### <a name="log-in-to-powershell"></a>登录到 PowerShell

1. 在计算机上启动 PowerShell，让其保持打开状态，直至本快速入门教程完成。 如果将它关闭再重新打开，则需再次运行这些命令。

    ![启动 PowerShell](media/tutorial-hybrid-copy-powershell/search-powershell.png)

1. 运行以下命令，然后输入用于登录 Azure 门户的 Azure 用户名和密码：
       
    ```powershell
    Connect-AzAccount
    ```        

1. 如果有多个 Azure 订阅，请运行以下命令，选择要使用的订阅。 请将 **SubscriptionId** 替换为自己的 Azure 订阅的 ID：

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```

## <a name="create-a-data-factory"></a>创建数据工厂

1. 为资源组名称定义一个变量，稍后会在 PowerShell 命令中使用该变量。 将以下命令复制到 PowerShell，指定 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)的名称（包括在双引号中，例如 `"adfrg"`），然后运行命令。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup"
    ```

1. 若要创建 Azure 资源组，请运行以下命令： 

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ``` 

    如果该资源组已存在，请勿覆盖它。 为 `$resourceGroupName` 变量分配另一个值，然后再次运行命令。

1. 为数据工厂名称定义一个变量，稍后可在 PowerShell 命令中使用该变量。 名称必须以字母或数字开头，并且只能包含字母、数字和短划线 (-) 字符。

    > [!IMPORTANT]
    >  使用全局唯一的名称更新数据工厂名称。 例如 ADFTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFTutorialFactory"
    ```

1. 定义一个用于数据工厂位置的变量： 

    ```powershell
    $location = "East US"
    ```  

1. 若要创建数据工厂，请运行以下 `Set-AzDataFactoryV2` cmdlet： 
    
    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

> [!NOTE]
> 
> * 数据工厂的名称必须全局唯一。 如果收到以下错误，请更改名称并重试。
>    ```
>    The specified data factory name 'ADFv2TutorialDataFactory' is already in use. Data factory names must be globally unique.
>    ```
> * 若要创建数据工厂实例，用于登录到 Azure 的用户帐户必须具有参与者或所有者角色，或者是 Azure 订阅的管理员。
> * 若要查看目前提供数据工厂的 Azure 区域的列表，请在以下页面上选择感兴趣的区域，然后展开“分析”以找到“数据工厂”：[可用产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/)。 数据工厂使用的数据存储（Azure 存储、Azure SQL 数据库等）和计算资源（Azure HDInsight 等）可以位于其他区域。
> 
> 

## <a name="create-a-self-hosted-integration-runtime"></a>创建自承载 Integration Runtime

在本部分，请创建一个自承载 Integration Runtime，然后将其与安装了 SQL Server 数据库的本地计算机相关联。 自承载 Integration Runtime 是一个组件，用于将数据从计算机上的 SQL Server 数据库复制到 Azure Blob 存储。 

1. 创建一个适用于 Integration Runtime 名称的变量。 使用唯一名称，并记下该名称。 本教程后面部分需要使用它。 

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```

1. 创建自我托管的集成运行时。 

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $integrationRuntimeName -Type SelfHosted -Description "selfhosted IR description"
    ``` 
    下面是示例输出：

    ```json
    Id                : /subscriptions/<subscription ID>/resourceGroups/ADFTutorialResourceGroup/providers/Microsoft.DataFactory/factories/onpremdf0914/integrationruntimes/myonpremirsp0914
    Type              : SelfHosted
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Name              : myonpremirsp0914
    Description       : selfhosted IR description
    ```

1. 若要检索所创建的 Integration Runtime 的状态，请运行以下命令：

    ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
    ```

    下面是示例输出：
    
    ```json
    Nodes                     : {}
    CreateTime                : 9/14/2017 10:01:21 AM
    InternalChannelEncryption :
    Version                   :
    Capabilities              : {}
    ScheduledUpdateDate       :
    UpdateDelayOffset         :
    LocalTimeZoneOffset       :
    AutoUpdate                :
    ServiceUrls               : {eu.frontend.clouddatahub.net, *.servicebus.windows.net}
    ResourceGroupName         : <ResourceGroup name>
    DataFactoryName           : <DataFactory name>
    Name                      : <Integration Runtime name>
    State                     : NeedRegistration
    ```

1. 若要检索可将自承载 Integration Runtime 注册到云中数据工厂服务的身份验证密钥，请运行以下命令。 复制其中一个密钥（去除引号），以便注册将在下一步安装到计算机上的自承载 Integration Runtime。 

    ```powershell
    Get-AzDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
    ```
    
    下面是示例输出：
    
    ```json
    {
        "AuthKey1":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
        "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
    }
    ```

## <a name="install-the-integration-runtime"></a>安装 Integration Runtime
1. 将 [Azure 数据工厂 Integration Runtime](https://www.microsoft.com/download/details.aspx?id=39717) 下载到本地 Windows 计算机上，然后进行安装。 

1. 在“欢迎使用 Microsoft Integration Runtime 安装向导”页上，选择“下一步”。  

1. 在“最终用户许可协议”窗口中接受许可协议的条款，然后选择“下一步”。 

1. 在“目标文件夹”窗口中，选择“下一步”。 

1. 在“准备安装 Microsoft Integration Runtime”窗口中选择“安装”。 

1. 如果看到一条警告消息，指出系统正将计算机配置为在不使用时进入睡眠或休眠模式，请选择“确定”。 

1. 如果显示“电源选项”窗口，请将其关闭，然后切换到安装窗口。 

1. 在“完成 Microsoft Integration Runtime 安装向导”页中选择“完成”。

1. 在“注册 Integration Runtime (自承载)”窗口中粘贴在上一部分保存的密钥，然后选择“注册”。 

    ![注册集成运行时](media/tutorial-hybrid-copy-powershell/register-integration-runtime.png)

    成功注册自承载 Integration Runtime 后，会显示以下消息： 

    ![已成功注册](media/tutorial-hybrid-copy-powershell/registered-successfully.png)

1. 在“新建 Integration Runtime (自承载)节点”窗口中，选择“下一步”。 

    ![“新建 Integration Runtime 节点”窗口](media/tutorial-hybrid-copy-powershell/new-integration-runtime-node-page.png)

1. 在“Intranet 信道”窗口中，选择“跳过”。  
    可以选择 TLS/SSL 认证，确保多节点 Integration Runtime 环境中的节点内通信安全。

    ![“Intranet 信道”窗口](media/tutorial-hybrid-copy-powershell/intranet-communication-channel-page.png)

1. 在“注册 Integration Runtime (自承载)”窗口中，选择“启动配置管理器”。 

1. 将节点连接到云服务后，会显示以下消息：

    ![节点已连接](media/tutorial-hybrid-copy-powershell/node-is-connected.png)

1. 执行以下操作，测试到 SQL Server 数据库的连接：

    ![“诊断”选项卡](media/tutorial-hybrid-copy-powershell/config-manager-diagnostics-tab.png)   

    a. 在“配置管理器”窗口中，切换到“诊断”选项卡。

    b. 在“数据源类型”框中，选择“SqlServer”。

    c. 输入服务器名称。

    d. 输入数据库名称。 

    e. 选择身份验证模式。 

    f. 输入用户名。 

    g. 输入与用户名关联的密码。

    h. 若要确认 Integration Runtime 能否连接到 SQL Server，请选择“测试”。  
    如果连接成功，则会显示绿色复选图标。 否则，会收到与故障相关的错误消息。 请解决问题，确保 Integration Runtime 可以连接到 SQL Server 实例。

    记下所有上述值，在本教程的后面部分使用。
    
## <a name="create-linked-services"></a>创建链接服务
若要将数据存储和计算服务链接到数据工厂，请在数据工厂中创建链接服务。 在本教程中，请将 Azure 存储帐户和本地 SQL Server 实例链接到数据存储。 链接服务包含的连接信息可供数据工厂服务用来在运行时连接到它们。 

### <a name="create-an-azure-storage-linked-service-destinationsink"></a>创建 Azure 存储链接服务（目标/接收器）
在此步骤中，请将 Azure 存储帐户链接到数据工厂。

1. 在 *C:\ADFv2Tutorial* 文件夹中，创建包含以下代码的名为 *AzureStorageLinkedService.json* 的 JSON 文件。 如果 *ADFv2Tutorial* 文件夹不存在，请创建。  

    > [!IMPORTANT]
    > 在保存文件之前，请将 \<accountName> 和 \<accountKey> 分别替换为 Azure 存储帐户的名称和密钥。 已在[先决条件](#get-storage-account-name-and-account-key)部分记下它们。

   ```json
    {
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>;EndpointSuffix=core.windows.net"
                }
            }
        },
        "name": "AzureStorageLinkedService"
    }
   ```

1. 在 PowerShell 中切换到 *C:\ADFv2Tutorial* 文件夹。

1. 若要创建链接服务 AzureStorageLinkedService，请运行以下 `Set-AzDataFactoryV2LinkedService` cmdlet： 

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
   ```

   下面是示例输出：

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

    如果收到“找不到文件”错误，请运行 `dir` 命令，确认该文件是否存在。 如果文件名的扩展为 *.txt*（例如 AzureStorageLinkedService.json.txt），请将其删除，然后再次运行 PowerShell 命令。 

### <a name="create-and-encrypt-a-sql-server-linked-service-source"></a>创建并加密 SQL Server 链接服务（源）
在此步骤中，请将本地 SQL Server 实例链接到数据工厂。

1. 使用以下代码，在 *C:\ADFv2Tutorial* 文件夹中创建名为 *SqlServerLinkedService.json* 的 JSON 文件：

    > [!IMPORTANT]
    > 根据连接到 SQL Server 时所使用的身份验证选择节。

    **使用 SQL 身份验证 (sa)：**

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<servername>;Database=<databasename>;User ID=<username>;Password=<password>;Timeout=60"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }
   ```    

    **使用 Windows 身份验证：**

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<server>;Database=<database>;Integrated Security=True"
                },
                "userName": "<user> or <domain>\\<user>",
                "password": {
                    "type": "SecureString",
                    "value": "<password>"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }    
    ```

    > [!IMPORTANT]
    > - 根据连接到 SQL Server 实例时所使用的身份验证选择节。
    > - 将 **\<integration runtime name>** 替换为 Integration Runtime 的名称。
    > - 保存文件之前，请将 **\<servername>**、**\<databasename>**、**\<username>** 和 **\<password>** 替换为 SQL Server 实例的值。
    > - 如需在用户帐户或服务器名称中使用反斜杠 (\\)，请在其前面加上转义字符 (\\)。 例如，使用 *mydomain\\\\myuser*。 

1. 若要加密敏感数据（用户名、密码等），请运行 `New-AzDataFactoryV2LinkedServiceEncryptedCredential` cmdlet。  
    这种加密可确保使用数据保护应用程序编程接口 (DPAPI) 加密凭据。 加密的凭据存储在自承载 Integration Runtime 节点本地（本地计算机）。 可将输出的有效负载重定向到包含已加密凭据的另一个 JSON 文件（在本例中为 *encryptedLinkedService.json*）。
    
   ```powershell
   New-AzDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -IntegrationRuntimeName $integrationRuntimeName -File ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
   ```

1. 运行以下命令，创建 EncryptedSqlServerLinkedService：

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -File ".\encryptedSqlServerLinkedService.json"
   ```


## <a name="create-datasets"></a>创建数据集
在此步骤中，请创建输入和输出数据集。 这些数据集代表复制操作的输入和输出数据，该复制操作可将数据从本地 SQL Server 数据库复制到 Azure Blob 存储。

### <a name="create-a-dataset-for-the-source-sql-server-database"></a>为源 SQL Server 数据库创建数据集
在此步骤中，请定义一个数据集，代表 SQL Server 数据库实例中的数据。 数据集为 SqlServerTable 类型。 它引用在上一步创建的 SQL Server 链接服务。 链接服务包含的连接信息可供数据工厂服务用来在运行时连接到 SQL Server 实例。 此数据集指定数据库中包含相应数据的 SQL 表。 在本教程中，**emp** 表包含源数据。 

1. 在 *C:\ADFv2Tutorial* 文件夹中，创建包含以下代码的名为 *SqlServerDataset.json* 的 JSON 文件：  

    ```json
    {
       "properties": {
            "type": "SqlServerTable",
            "typeProperties": {
                "tableName": "dbo.emp"
            },
            "structure": [
                 {
                    "name": "ID",
                    "type": "String"
                },
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "linkedServiceName": {
                "referenceName": "EncryptedSqlServerLinkedService",
                "type": "LinkedServiceReference"
            }
        },
        "name": "SqlServerDataset"
    }
    ```

1. 若要创建数据集 SqlServerDataset，请运行 `Set-AzDataFactoryV2Dataset` cmdlet。

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerDataset" -File ".\SqlServerDataset.json"
    ```

    下面是示例输出：

    ```json
    DatasetName       : SqlServerDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Structure         : {"name": "ID" "type": "String", "name": "FirstName" "type": "String", "name": "LastName" "type": "String"}
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-dataset-for-azure-blob-storage-sink"></a>为 Azure Blob 存储（接收器）创建数据集
在此步骤中，请定义一个数据集，表示要复制到 Azure Blob 存储的数据。 数据集为 AzureBlob 类型。 它引用此前在本教程中创建的 Azure 存储链接服务。 

链接服务包含的连接信息可供数据工厂用来在运行时连接到 Azure 存储帐户。 此数据集在 Azure 存储中指定从 SQL Server 数据库向其复制数据的文件夹。 在本教程中，该文件夹为 *adftutorial/fromonprem*，其中 `adftutorial` 是 Blob 容器，`fromonprem` 是文件夹。 

1. 在 *C:\ADFv2Tutorial* 文件夹中，创建包含以下代码的名为 *AzureBlobDataset.json* 的 JSON 文件：

    ```json
    {
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/fromonprem",
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        },
        "name": "AzureBlobDataset"
    }
    ```

1. 若要创建数据集 AzureBlobDataset，请运行 `Set-AzDataFactoryV2Dataset` cmdlet。

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureBlobDataset" -File ".\AzureBlobDataset.json"
    ```

    下面是示例输出：

    ```json
    DatasetName       : AzureBlobDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

## <a name="create-a-pipeline"></a>创建管道
本教程创建包含复制活动的管道。 该复制活动使用 SqlServerDataset 作为输入数据集，AzureBlobDataset 作为输出数据集。 源类型设置为 *SqlSource*，接收器类型设置为 *BlobSink*。

1. 在 *C:\ADFv2Tutorial* 文件夹中，创建包含以下代码的名为 *SqlServerToBlobPipeline.json* 的 JSON 文件：

    ```json
    {
       "name": "SQLServerToBlobPipeline",
        "properties": {
            "activities": [       
                {
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource"
                        },
                        "sink": {
                            "type":"BlobSink"
                        }
                    },
                    "name": "CopySqlServerToAzureBlobActivity",
                    "inputs": [
                        {
                            "referenceName": "SqlServerDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "AzureBlobDataset",
                            "type": "DatasetReference"
                        }
                    ]
                }
            ]
        }
    }
    ```

1. 若要创建管道 SQLServerToBlobPipeline，请运行 `Set-AzDataFactoryV2Pipeline` cmdlet。

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SQLServerToBlobPipeline" -File ".\SQLServerToBlobPipeline.json"
    ```

    下面是示例输出：

    ```json
    PipelineName      : SQLServerToBlobPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : onpremdf0914
    Activities        : {CopySqlServerToAzureBlobActivity}
    Parameters        :  
    ```

## <a name="create-a-pipeline-run"></a>创建管道运行
针对 SQLServerToBlobPipeline 管道启动管道运行，并捕获管道运行 ID，以便将来进行监视。

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'SQLServerToBlobPipeline'
```

## <a name="monitor-the-pipeline-run"></a>监视管道运行

1. 若要持续检查管道 SQLServerToBlobPipeline 的运行状态，请在 PowerShell 中运行以下脚本，列显最终结果：

    ```powershell
    while ($True) {
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

        if (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
            Start-Sleep -Seconds 30
        }
        else {
            Write-Host "Pipeline 'SQLServerToBlobPipeline' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
    }
    ```

    下面是示例运行的输出：

    ```jdon
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : copy
    PipelineRunId     : 4ec8980c-62f6-466f-92fa-e69b10f33640
    PipelineName      : SQLServerToBlobPipeline
    Input             :  
    Output            :  
    LinkedServiceName :
    ActivityRunStart  : 9/13/2017 1:35:22 PM
    ActivityRunEnd    : 9/13/2017 1:35:42 PM
    DurationInMs      : 20824
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    ```

1. 可以运行以下命令，获取管道 SQLServerToBlobPipeline 的运行 ID，并查看详细的活动运行结果： 

    ```powershell
    Write-Host "Pipeline 'SQLServerToBlobPipeline' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "CopySqlServerToAzureBlobActivity"}).Output.ToString()
    ```

    下面是示例运行的输出：

    ```json
    {
      "dataRead": 36,
      "dataWritten": 24,
      "rowsCopied": 2,
      "copyDuration": 3,
      "throughput": 0.01171875,
      "errors": [],
      "effectiveIntegrationRuntime": "MyIntegrationRuntime",
      "billedDuration": 3
    }
    ```

## <a name="verify-the-output"></a>验证输出
该管道自动在 `adftutorial` Blob 容器中创建名为 *fromonprem* 的输出文件夹。 确认在输出文件夹中看到 *dbo.emp.txt* 文件。 

1. 在 Azure 门户的“adftutorial”容器窗口中选择“刷新”，查看输出文件夹。

    ![创建的输出文件夹](media/tutorial-hybrid-copy-powershell/fromonprem-folder.png)
1. 选择文件夹列表中的 `fromonprem`。 
1. 确认可以看到名为 `dbo.emp.txt` 的文件。

    ![输出文件](media/tutorial-hybrid-copy-powershell/fromonprem-file.png)


## <a name="next-steps"></a>后续步骤
此示例中的管道将数据从 Azure Blob 存储中的一个位置复制到另一个位置。 你已了解如何：

> [!div class="checklist"]
> * 创建数据工厂。
> * 创建自我托管的集成运行时。
> * 创建 SQL Server 和 Azure 存储链接服务。 
> * 创建 SQL Server 和 Azure Blob 数据集。
> * 创建包含复制活动的管道，用于移动数据。
> * 启动管道运行。
> * 监视管道运行。

有关数据工厂支持的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。

若要了解如何将数据从源批量复制到目标，请继续学习以下教程：

> [!div class="nextstepaction"]
>[批量复制数据](tutorial-bulk-copy.md)
