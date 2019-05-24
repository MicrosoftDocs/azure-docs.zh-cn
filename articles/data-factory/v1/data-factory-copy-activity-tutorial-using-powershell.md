---
title: 教程：使用 Azure PowerShell 创建移动数据的管道 | Microsoft Docs
description: 本教程使用 Azure PowerShell 创建包含复制活动的 Azure 数据工厂管道。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 71087349-9365-4e95-9847-170658216ed8
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 13f67bfe0902a528d16b6a967f9d4ac189100406
ms.sourcegitcommit: 0dd053b447e171bc99f3bad89a75ca12cd748e9c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/26/2019
ms.locfileid: "58482397"
---
# <a name="tutorial-create-a-data-factory-pipeline-that-moves-data-by-using-azure-powershell"></a>教程：使用 Azure PowerShell 创建移动数据的数据工厂管道
> [!div class="op_single_selector"]
> * [概述与先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [复制向导](data-factory-copy-data-wizard-tutorial.md)
> * [Azure 门户](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure 资源管理器模板](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> 本文适用于数据工厂版本 1。 如果使用的是数据工厂服务的当前版本，请参阅[复制活动教程](../quickstart-create-data-factory-powershell.md)。 

本文介绍如何使用 PowerShell 创建数据工厂，以便通过管道将数据从 Azure Blob 存储复制到 Azure SQL 数据库。 如果不熟悉 Azure 数据工厂，请在学习本教程之前，先通读 [Azure 数据工厂简介](data-factory-introduction.md)一文。   

本教程会创建包含以下一个活动的管道：复制活动。 复制活动可以将数据从支持的数据存储复制到支持的接收器数据存储。 如需可以用作源和接收器的数据存储的列表，请参阅[支持的数据存储](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 该活动由全球可用的服务提供支持，能以安全、可靠、可缩放的方式在各种数据存储区间复制数据。 有关复制活动的详细信息，请参阅[数据移动活动](data-factory-data-movement-activities.md)。

一个管道可以有多个活动。 而且，可以通过将一个活动的输出数据集设置为另一个活动的输入数据集，链接两个活动（两个活动先后运行）。 有关详细信息，请参阅[管道中的多个活动](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

> [!NOTE]
> 本文不会介绍所有数据工厂 cmdlet。 有关这些 cmdlet 的综合文档，请参阅[数据工厂 cmdlet 参考](/powershell/module/az.datafactory)。
> 
> 本教程中的数据管道将数据从源数据存储复制到目标数据存储。 有关如何使用 Azure 数据工厂转换数据的教程，请参阅[教程：使用 Hadoop 群集构建用于转换数据的管道](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- 完成[教程先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一文中列出的先决条件步骤。
- 安装 **Azure PowerShell**。 遵循[如何安装和配置 Azure PowerShell](/powershell/azure/install-Az-ps) 中的说明。

## <a name="steps"></a>Steps
下面是本教程中要执行的步骤：

1. 创建 Azure **数据工厂**。 本步骤创建名为 ADFTutorialDataFactoryPSH 的数据工厂。 
1. 在数据工厂中创建“链接服务”。 此步骤将创建两个链接服务，其类型分别为：Azure 存储和 Azure SQL 数据库。 
    
    AzureStorageLinkedService 链接将 Azure 存储帐户链接到数据工厂。 根据[先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)创建了一个容器并将数据上传到该存储帐户。   

    AzureSqlLinkedService 将 Azure SQL 数据库链接到数据工厂。 从 Blob 存储复制的数据存储在该数据库中。 根据[先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)在该数据库中创建了一个 SQL 表。   
1. 在数据工厂中创建输入和输出**数据集**。  
    
    Azure 存储链接服务指定一个连接字符串，数据工厂服务在运行时使用该字符串连接到 Azure 存储帐户。 输入 Blob 数据集指定容器以及包含输入数据的文件夹。  

    类似地，Azure SQL 数据库链接服务指定一个连接字符串，数据工厂服务在运行时使用该字符串连接到 Azure SQL 数据库。 输出 SQL 表数据集在数据库中指定一个表，数据将从 Blob 存储复制到该表中。
1. 在数据工厂中创建**管道**。 本步骤创建包含复制活动的管道。   
    
    复制活动将数据从 Azure Blob 存储中的 Blob 复制到 Azure SQL 数据库中的表。 可以通过管道中的复制活动，将数据从任何支持的源复制到任何支持的目标。 有关支持的数据存储的列表，请参阅[数据移动活动](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文。 
1. 监视管道。 本步骤使用 PowerShell **监视**输入和输出数据集的切片。

## <a name="create-a-data-factory"></a>创建数据工厂
> [!IMPORTANT]
> 完成[本教程的先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)（如果尚未这样做）。   

数据工厂可以包含一个或多个数据管道。 管道可以包含一个或多个活动。 例如，将数据从源复制到目标数据存储的复制活动，以及运行 Hive 脚本来将输入数据转换为产品输出数据的 HDInsight Hive 活动。 首先，在此步骤中创建数据工厂。

1. 启动 **PowerShell**。 在本教程结束之前，请将 Azure PowerShell 保持打开状态。 如果将它关闭再重新打开，则需要再次运行下述命令。

    运行以下命令并输入用于登录 Azure 门户的用户名和密码：

    ```powershell
    Connect-AzAccount
    ```   
   
    运行以下命令查看此帐户的所有订阅：

    ```powershell
    Get-AzSubscription
    ```

    运行以下命令选择要使用的订阅。 将 **&lt;NameOfAzureSubscription**&gt; 替换为 Azure 订阅的名称：

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```
1. 运行以下命令，创建名为 **ADFTutorialResourceGroup** 的 Azure 资源组：

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```
    
    本教程中的某些步骤假设使用名为 **ADFTutorialResourceGroup**的资源组。 如果使用不同的资源组，需使用该资源组取代本教程中的 ADFTutorialResourceGroup。
1. 运行 **New-AzDataFactory** cmdlet，创建名为 **ADFTutorialDataFactoryPSH** 的数据工厂：  

    ```powershell
    $df=New-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"
    ```
    此名称可能已使用。 因此，可以通过添加前缀或后缀（例如：ADFTutorialDataFactoryPSH05152017）使数据工厂的名称变得唯一，并再次运行该命令。  

请注意以下几点：

* Azure 数据工厂的名称必须全局唯一。 如果收到以下错误，请更改名称（例如改为 yournameADFTutorialDataFactoryPSH）。 执行本教程中的步骤时，请使用此名称取代 ADFTutorialFactoryPSH。 对于数据工厂项目，请参阅 [Data Factory - Naming Rules](data-factory-naming-rules.md)（数据工厂 - 命名规则）。

    ```
    Data factory name “ADFTutorialDataFactoryPSH” is not available
    ```
* 只有 Azure 订阅的参与者或管理员才可以创建数据工厂实例。
* 数据工厂名称可能在将来被注册为 DNS 名称，因此将公开可见。
* 可能看到以下错误：“该订阅未注册，无法使用命名空间 Microsoft.DataFactory”。 请执行以下操作之一，并重试发布：

  * 在 Azure PowerShell 中运行以下命令，注册数据工厂提供程序。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

    运行以下命令，确认数据工厂提供程序是否已注册：

    ```powershell
    Get-AzResourceProvider
    ```
  * 使用 Azure 订阅登录到 [Azure 门户](https://portal.azure.com)。 在 Azure 门户中转到“数据工厂”边栏选项卡，或创建一个数据工厂。 此操作会自动注册提供程序。

## <a name="create-linked-services"></a>创建链接服务
可在数据工厂中创建链接服务，将数据存储和计算服务链接到数据工厂。 在本教程中，请不要使用任何计算服务，例如 Azure HDInsight 或 Azure Data Lake Analytics。 可以使用两个数据存储，其类型为 Azure 存储（源）和 Azure SQL 数据库（目标）。 

因此，请创建两个分别名为 AzureStorageLinkedService 和 AzureSqlLinkedService 的链接服务，其类型分别为：AzureStorage 和 AzureSqlDatabase。  

AzureStorageLinkedService 链接将 Azure 存储帐户链接到数据工厂。 已根据[先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)，在此存储帐户中创建了一个容器并上传了数据。   

AzureSqlLinkedService 将 Azure SQL 数据库链接到数据工厂。 从 Blob 存储复制的数据存储在该数据库中。 根据[先决条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)在该数据库中创建了 emp 表。 

### <a name="create-a-linked-service-for-an-azure-storage-account"></a>为 Azure 存储帐户创建链接服务
本步骤将 Azure 存储帐户链接到数据工厂。

1. 在 **C:\ADFGetStartedPSH** 文件夹中，创建包含以下内容的名为 **AzureStorageLinkedService.json** 的 JSON 文件：（创建 ADFGetStartedPSH 文件夹（如果不存在）。）

    > [!IMPORTANT]
    > 将 &lt;accountname&gt; 和 &lt;accountkey&gt; 分别替换为 Azure 存储帐户的名称和密钥，然后保存文件。 

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        }
     }
    ``` 
1. 在 **Azure PowerShell** 中，切换到 **ADFGetStartedPSH** 文件夹。
1. 运行 **New-AzDataFactoryLinkedService** cmdlet 以创建链接服务：**AzureStorageLinkedService**。 此 cmdlet 以及本教程中使用的其他数据工厂 cmdlet 要求传递 **ResourceGroupName** 和 **DataFactoryName** 参数的值。 或者，可以传递 New-AzDataFactory cmdlet 返回的 DataFactory 对象，而无需在每次运行 cmdlet 时键入 ResourceGroupName 和 DataFactoryName。 

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureStorageLinkedService.json
    ```
    下面是示例输出：

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ``` 

    创建此链接服务的另一方式是，指定资源组名称和数据工厂名称，而不指定 DataFactory 对象。  

    ```powershell
    New-AzDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName <Name of your data factory> -File .\AzureStorageLinkedService.json
    ```

### <a name="create-a-linked-service-for-an-azure-sql-database"></a>为 Azure SQL 数据库创建链接服务
在此步骤中，将 Azure SQL 数据库链接到数据工厂。

1. 在 C:\ADFGetStartedPSH 文件夹中，创建包含以下内容的名为 AzureSqlLinkedService.json 的 JSON 文件：

    > [!IMPORTANT]
    > 将 &lt;servername&gt;、&lt;databasename&gt;、&lt;username@servername&gt; 和 &lt;password&gt; 分别替换为 Azure SQL 服务器名称、数据库名称、用户帐户名和密码。
    
    ```json
    {
        "name": "AzureSqlLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
     }
    ```
1. 运行以下命令创建链接服务：

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json
    ```
    
    下面是示例输出：

    ```
    LinkedServiceName : AzureSqlLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ```

   确认为 SQL 数据库服务器启用了“允许访问 Azure 服务”设置。 若要验证并启用此设置，请执行以下步骤：

    1. 登录到 [Azure 门户](https://portal.azure.com)
    1. 单击左侧的“更多服务”，并单击“数据库”类别中的“SQL Server”。
    1. 在 SQL Server 列表中选择服务器。
    1. 在 SQL Server 边栏选项卡上，单击“显示防火墙设置”链接。
    1. 在“防火墙设置”边栏选项卡中，单击“允许访问 Azure 服务”旁边的“打开”。
    1. 单击工具栏上的“保存”。 

## <a name="create-datasets"></a>创建数据集
在上一步骤中，已创建用于将 Azure 存储帐户和 Azure SQL 数据库链接到数据工厂的链接服务。 本步骤定义两个名为 InputDataset 和 OutputDataset 的数据集，表示存储在数据存储中的输入和输出数据，这些数据存储分别由 AzureStorageLinkedService 和 AzureSqlLinkedService 引用。

Azure 存储链接服务指定一个连接字符串，数据工厂服务在运行时使用该字符串连接到 Azure 存储帐户。 输入 Blob 数据集 (InputDataset) 指定容器以及包含输入数据的文件夹。  

类似地，Azure SQL 数据库链接服务指定一个连接字符串，数据工厂服务在运行时使用该字符串连接到 Azure SQL 数据库。 输出 SQL 表数据集 (OututDataset) 在数据库中指定一个表，数据将从 Blob 存储复制到该表中。 

### <a name="create-an-input-dataset"></a>创建输入数据集
本步骤在 AzureStorageLinkedService 链接服务代表的 Azure 存储中创建名为 InputDataset 的数据集，该数据集指向 Blob 容器 (adftutorial) 根文件夹中的 Blob 文件 (emp.txt)。 如果不指定 fileName 的值（或者跳过此步骤），则会将输入文件夹中的所有 Blob 复制到目标。 在本教程中，请为 fileName 指定一个值。  

1. 在 **C:\ADFGetStartedPSH** 文件夹中，创建包含以下内容的名为 **InputDataset.json** 的 JSON 文件：

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "emp.txt",
                "folderPath": "adftutorial/",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "external": true,
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
     }
    ```

    下表提供了代码片段中使用的 JSON 属性的描述：

    | 属性 | 说明 |
    |:--- |:--- |
    | type | type 属性设置为 **AzureBlob**，因为数据驻留在 Azure Blob 存储中。 |
    | linkedServiceName | 表示前面创建的 **AzureStorageLinkedService**。 |
    | folderPath | 指定 Blob **容器**以及包含输入 Blob 的**文件夹**。 在本教程中，adftutorial 是 Blob 容器，文件夹是根文件夹。 | 
    | fileName | 此属性是可选的。 如果省略此属性，将选取 folderPath 中的所有文件。 在本教程中，为 fileName 指定了 **emp.txt**，因此仅选取该文件进行处理。 |
    | format -> type |输入文件为文本格式，因此我们使用 **TextFormat**。 |
    | columnDelimiter | 输入文件中的列以**逗号字符 (`,`)** 分隔。 |
    | frequency/interval | frequency 设置为 **Hour**，interval 设置为 **1**，表示**每小时**获取一次输入切片。 换言之，数据工厂服务每小时在指定的 Blob 容器 (**adftutorial**) 的根文件夹中查找输入数据。 它会查找管道开始和结束时间范围内的数据，而不是范围外的数据。  |
    | external | 如果数据不是由该管道生成的，此属性设置为 **true**。 本教程中的输入数据位于 emp.txt 文件中，此文件不是由该管道生成的，因此将此属性设置为 true。 |

    有关这些 JSON 属性的详细信息，请参阅 [Azure Blob 连接器](data-factory-azure-blob-connector.md#dataset-properties)一文。
1. 运行以下命令创建数据工厂数据集。

    ```powershell  
    New-AzDataFactoryDataset $df -File .\InputDataset.json
    ```
    下面是示例输出：

    ```
    DatasetName       : InputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureBlobDataset
    Policy            : Microsoft.Azure.Management.DataFactories.Common.Models.Policy
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

### <a name="create-an-output-dataset"></a>创建输出数据集
本步骤的此部分创建名为 **OutputDataset**的输出数据集。 此数据集指向 Azure SQL 数据库中 **AzureSqlLinkedService**所代表的 SQL 表。 

1. 在 **C:\ADFGetStartedPSH** 文件夹中，创建包含以下内容的名为 **OutputDataset.json** 的 JSON 文件：

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```

    下表提供了代码片段中使用的 JSON 属性的描述：

    | 属性 | 说明 |
    |:--- |:--- |
    | type | type 属性设置为 **AzureSqlTable**，因为数据复制到 Azure SQL 数据库中的表。 |
    | linkedServiceName | 表示前面创建的 **AzureSqlLinkedService**。 |
    | tableName | 指定一个表，以便将数据复制到其中。 | 
    | frequency/interval | frequency 设置为 **Hour**，interval 设置为 **1**，表示输出切片在管道开始和结束时间范围内（而不是范围外）**每小时**生成一次。  |

    数据库的 emp 表包含三列 – **ID**、**FirstName** 和 **LastName**。 ID 是标识列，因此只需在此处指定 **FirstName** 和 **LastName**。

    有关这些 JSON 属性的详细信息，请参阅 [Azure SQL 连接器](data-factory-azure-sql-connector.md#dataset-properties)一文。
1. 运行以下命令创建数据工厂数据集。

    ```powershell   
    New-AzDataFactoryDataset $df -File .\OutputDataset.json
    ```

    下面是示例输出：

    ```
    DatasetName       : OutputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureSqlTableDataset
    Policy            :
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

## <a name="create-a-pipeline"></a>创建管道
本步骤创建管道，其中包含使用 **InputDataset** 作为输入和使用 **OutputDataset** 作为输出的**复制活动**。

目前，输出数据集驱动计划。 在本教程中，输出数据集配置为每小时生成一个切片。 管道的开始时间和结束时间相差一天，即 24 小时。 因此，管道会生成 24 个输出数据集切片。 


1. 在 **C:\ADFGetStartedPSH** 文件夹中，创建包含以下内容的名为 **ADFTutorialPipeline.json** 的 JSON 文件：

    ```json   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "SqlSink",
                "writeBatchSize": 10000,
                "writeBatchTimeout": "60:00:00"
              }
            },
            "Policy": {
              "concurrency": 1,
              "executionPriorityOrder": "NewestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
        ],
        "start": "2017-05-11T00:00:00Z",
        "end": "2017-05-12T00:00:00Z"
      }
    } 
    ```
    请注意以下几点：
   
   - 在 activities 节中，只有一个活动的 **type** 设置为 **Copy**。 有关复制活动的详细信息，请参阅[数据移动活动](data-factory-data-movement-activities.md)。 在数据工厂解决方案中，也可以使用[数据转换活动](data-factory-data-transformation-activities.md)。
   - 活动的输入设置为 **InputDataset**，活动的输出设置为 **OutputDataset**。 
   - 在 **typeProperties** 节中，**BlobSource** 指定为源类型，**SqlSink** 指定为接收器类型。 有关复制活动支持的数据存储（以源和接收器的形式存在）的完整列表，请参阅[支持的数据存储](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 若要了解如何使用特定的受支持的数据存储（以源/接收器的形式存在），请单击表中的链接。  
     
     将 **start** 属性的值替换为当前日期，将 **end** 值替换为下一个日期。 可以仅指定日期部分，跳过日期时间的时间部分。 例如，“2016-02-03”等效于“2016-02-03T00:00:00Z”
     
     开始和结束日期时间必须采用 [ISO 格式](https://en.wikipedia.org/wiki/ISO_8601)。 例如：2016-10-14T16:32:41Z。 **结束** 时间是可选的，但本教程使用该时间。 
     
     如果未指定 **end** 属性的值，则以“**开始时间 + 48 小时**”计算。 若要无限期运行管道，请指定 **9999-09-09** 作为 **end** 属性的值。
     
     在上述示例中，由于每小时生成一个数据切片，因此共有 24 个数据切片。

     有关管道定义中 JSON 属性的说明，请参阅[创建管道](data-factory-create-pipelines.md)一文。 有关复制活动定义中 JSON 属性的说明，请参阅[数据移动活动](data-factory-data-movement-activities.md)一文。 有关 BlobSource 支持的 JSON 属性的说明，请参阅 [Azure Blob 连接器](data-factory-azure-blob-connector.md)一文。 有关 SqlSink 支持的 JSON 属性的说明，请参阅 [Azure SQL 数据库连接器](data-factory-azure-sql-connector.md)一文。
1. 运行以下命令创建数据工厂表。

    ```powershell   
    New-AzDataFactoryPipeline $df -File .\ADFTutorialPipeline.json
    ```

    下面是示例输出： 

    ```
    PipelineName      : ADFTutorialPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.PipelinePropertie
    ProvisioningState : Succeeded
    ```

祝贺你！ 现已成功创建 Azure 数据工厂，可以使用管道将数据从 Azure Blob 存储复制到 Azure SQL 数据库了。 

## <a name="monitor-the-pipeline"></a>监视管道
本步骤使用 Azure PowerShell 监视 Azure 数据工厂的运行情况。

1. 将 &lt;DataFactoryName&gt; 替换为数据工厂的名称，然后运行 **Get-AzDataFactory** 并将输出分配给变量 $df。

    ```powershell  
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name <DataFactoryName>
    ```

    例如：
    ```powershell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH0516
    ```
    
    然后运行“输出 $df 的内容”，会看到以下输出： 
    
    ```
    PS C:\ADFGetStartedPSH> $df
    
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DataFactoryId     : 6f194b34-03b3-49ab-8f03-9f8a7b9d3e30
    ResourceGroupName : ADFTutorialResourceGroup
    Location          : West US
    Tags              : {}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DataFactoryProperties
    ProvisioningState : Succeeded
    ```
1. 运行 **Get-AzDataFactorySlice**，获取有关 **OutputDataset**（管道的输出数据集）的所有切片的详细信息。  

    ```powershell   
    Get-AzDataFactorySlice $df -DatasetName OutputDataset -StartDateTime 2017-05-11T00:00:00Z
    ```

   此设置应与管道 JSON 中的 **Start** 值匹配。 应会看到 24 个切片，从当日 12 AM 到下一日 12 AM，每小时各有一个。

   下面是输出中的三个示例切片： 

    ``` 
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 11:00:00 PM
    End               : 5/12/2017 12:00:00 AM
    RetryCount        : 0
    State             : Ready
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 9:00:00 PM
    End               : 5/11/2017 10:00:00 PM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0   

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 8:00:00 PM
    End               : 5/11/2017 9:00:00 PM
    RetryCount        : 0
    State             : Waiting
    SubState          : ConcurrencyLimit
    LatencyStatus     :
    LongRetryCount    : 0
    ```
1. 运行 **Get-AzDataFactoryRun**，获取**特定**切片的活动运行详细信息。 从前一命令的输出复制日期时间值，以便指定 StartDateTime 参数的值。 

    ```powershell  
    Get-AzDataFactoryRun $df -DatasetName OutputDataset -StartDateTime "5/11/2017 09:00:00 PM"
    ```

   下面是示例输出： 

    ```
    Id                  : c0ddbd75-d0c7-4816-a775-704bbd7c7eab_636301332000000000_636301368000000000_OutputDataset
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : ADFTutorialDataFactoryPSH0516
    DatasetName         : OutputDataset
    ProcessingStartTime : 5/16/2017 8:00:33 PM
    ProcessingEndTime   : 5/16/2017 8:01:36 PM
    PercentComplete     : 100
    DataSliceStart      : 5/11/2017 9:00:00 PM
    DataSliceEnd        : 5/11/2017 10:00:00 PM
    Status              : Succeeded
    Timestamp           : 5/16/2017 8:00:33 PM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : CopyFromBlobToSQL
    PipelineName        : ADFTutorialPipeline
    Type                : Copy  
    ```

有关数据工厂 cmdlet 的综合文档，请参阅[数据工厂 cmdlet 参考](/powershell/module/az.datafactory)。

## <a name="summary"></a>摘要
本教程创建了一个 Azure 数据工厂，用于将数据从 Azure Blob 复制到 Azure SQL 数据库。 已使用 Visual PowerShell 创建数据工厂、链接服务、数据集和管道。 下面是本教程中执行的高级步骤：  

1. 创建了 Azure **数据工厂**。
1. 创建了**链接服务**：

   a. 一个 **Azure 存储**链接服务，用于链接保存输入数据的 Azure 存储帐户。     
   b. 一个 **Azure SQL** 链接服务，用于链接保存输出数据的 SQL 数据库。
1. 创建了 **数据集** ，用于描述管道的输入和输出数据。
1. 创建了包含**复制活动**的**管道**，其中 **BlobSource** 为源，**SqlSink** 为接收器。

## <a name="next-steps"></a>后续步骤
在本教程中，在复制操作中使用了 Azure Blob 存储作为源数据存储，使用了 Azure SQL 数据库作为目标数据存储。 下表列出了复制活动支持的充当源和目标的数据存储： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要了解如何通过数据存储复制数据，请单击表中数据存储的链接。 

