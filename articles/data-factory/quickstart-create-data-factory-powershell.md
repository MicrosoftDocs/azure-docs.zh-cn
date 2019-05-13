---
title: 使用 Azure 数据工厂复制 Blob 存储中的数据 | Microsoft Docs
description: 创建一个 Azure 数据工厂，将数据从 Azure Blob 存储中的一个位置复制到另一位置。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: quickstart
ms.date: 01/22/2018
ms.author: jingwang
ms.openlocfilehash: 07b5b039e0069702b613619c54eb7eda46bf3051
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60313182"
---
# <a name="quickstart-create-an-azure-data-factory-using-powershell"></a>快速入门：使用 PowerShell 创建 Azure 数据工厂

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [版本 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [当前版本](quickstart-create-data-factory-powershell.md)

此快速入门介绍了如何使用 PowerShell 创建 Azure 数据工厂。 在此数据工厂中创建的管道会将数据从 Azure Blob 存储中的一个文件夹**复制**到另一个文件夹。 有关如何使用 Azure 数据工厂**转换**数据的教程，请参阅[教程：使用 Spark 转换数据](transform-data-using-spark.md)。

> [!NOTE]
> 本文不提供数据工厂服务的详细介绍。 有关 Azure 数据工厂服务的介绍，请参阅 [Azure 数据工厂简介](introduction.md)。

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)]

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

按[如何安装和配置 Azure PowerShell](/powershell/azure/install-Az-ps) 中的说明安装最新的 Azure PowerShell 模块。

#### <a name="log-in-to-powershell"></a>登录到 PowerShell

1. 在计算机上启动 **PowerShell**。 在完成本快速入门之前，请将 PowerShell 保持打开状态。 如果将它关闭再重新打开，则需要再次运行这些命令。

2. 运行以下命令，并输入用于登录 Azure 门户的同一 Azure 用户名和密码：

    ```powershell
    Connect-AzAccount
    ```

3. 运行以下命令查看此帐户的所有订阅：

    ```powershell
    Get-AzSubscription
    ```

4. 如果看到多个订阅与帐户相关联，请运行以下命令，选择要使用的订阅。 请将 **SubscriptionId** 替换为自己的 Azure 订阅的 ID：

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"
    ```

## <a name="create-a-data-factory"></a>创建数据工厂

1. 为资源组名称定义一个变量，稍后会在 PowerShell 命令中使用该变量。 将以下命令文本复制到 PowerShell，在双引号中指定 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)的名称，然后运行命令。 例如：`"ADFQuickStartRG"`。

     ```powershell
    $resourceGroupName = "ADFQuickStartRG";
    ```

    如果该资源组已存在，请勿覆盖它。 为 `$ResourceGroupName` 变量分配另一个值，然后再次运行命令

2. 若要创建 Azure 资源组，请运行以下命令：

    ```powershell
    $ResGrp = New-AzResourceGroup $resourceGroupName -location 'East US'
    ```

    如果该资源组已存在，请勿覆盖它。 为 `$ResourceGroupName` 变量分配另一个值，然后再次运行命令。

3. 定义一个用于数据工厂名称的变量。 

    > [!IMPORTANT]
    >  更新数据工厂名称，使之全局唯一。 例如 ADFTutorialFactorySP1127。

    ```powershell
    $dataFactoryName = "ADFQuickStartFactory";
    ```

4. 若要创建数据工厂，请运行下面的 **Set-AzDataFactoryV2** cmdlet，使用 $ResGrp 变量中的 Location 和 ResourceGroupName 属性：

    ```powershell
    $DataFactory = Set-AzDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName `
        -Location $ResGrp.Location -Name $dataFactoryName
    ```

请注意以下几点：

* Azure 数据工厂的名称必须全局唯一。 如果收到以下错误，请更改名称并重试。

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```

* 若要创建数据工厂实例，用于登录到 Azure 的用户帐户必须属于**参与者**或**所有者**角色，或者是 Azure 订阅的**管理员**。

* 若要查看目前提供数据工厂的 Azure 区域的列表，请在以下页面上选择感兴趣的区域，然后展开“分析”以找到“数据工厂”：[各区域的产品可用性](https://azure.microsoft.com/global-infrastructure/services/)。 数据工厂使用的数据存储（Azure 存储、Azure SQL 数据库，等等）和计算资源（HDInsight 等）可以位于其他区域中。

## <a name="create-a-linked-service"></a>创建链接服务

在数据工厂中创建链接服务，将数据存储和计算服务链接到数据工厂。 在本快速入门中，请创建一个 Azure 存储链接服务，用作源存储和接收器存储。 链接服务包含的连接信息可供数据工厂服务用来在运行时连接到它。

1. 在 **C:\ADFv2QuickStartPSH** 文件夹中，创建包含以下内容的名为 **AzureStorageLinkedService.json** 的 JSON 文件：（创建 ADFv2QuickStartPSH 文件夹（如果不存在）。）

    > [!IMPORTANT]
    > 将 &lt;accountName&gt; 和 &lt;accountKey&gt; 分别替换为 Azure 存储帐户的名称和密钥，然后保存文件。

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": {
                    "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>;EndpointSuffix=core.windows.net",
                    "type": "SecureString"
                }
            }
        }
    }
    ```

    如果使用记事本，请在“另存为”对话框中选择“所有文件”作为“另存为类型”字段的值。 否则，会为文件添加 `.txt` 扩展。 例如，`AzureStorageLinkedService.json.txt`。 如果先在文件资源管理器中创建该文件，然后再在记事本中将其打开，则可能看不到 `.txt` 扩展，因为系统默认设置“隐藏已知文件类型的扩展名”选项。 在执行下一步骤之前删除 `.txt` 扩展名。

2. 在 **PowerShell** 中，切换到 **ADFv2QuickStartPSH** 文件夹。

    ```powershell
    Set-Location 'C:\ADFv2QuickStartPSH'
    ```

3. 运行 **Set-AzDataFactoryV2LinkedService** cmdlet 来创建链接服务：**AzureStorageLinkedService**。

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureStorageLinkedService" `
        -DefinitionFile ".\AzureStorageLinkedService.json"
    ```

    下面是示例输出：

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

## <a name="create-a-dataset"></a>创建数据集

在此步骤中，请定义一个数据集，表示从源复制到接收器的数据。 数据集为 **AzureBlob** 类型。 它引用在上一步创建的 **Azure 存储链接服务**。 它采用一个参数来构造 **folderPath** 属性。 对于输入数据集，管道中的复制活动将输入路径作为此参数的值来传递。 同样，对于输出数据集，复制活动将输出路径作为此参数的值来传递。 

1. 在 **C:\ADFv2QuickStartPSH** 文件夹中创建一个名为 **BlobDataset.json** 的 JSON 文件，使其包含以下内容：

    ```json
    {
        "name": "BlobDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "@{dataset().path}"
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            },
            "parameters": {
                "path": {
                    "type": "String"
                }
            }
        }
    }
    ```

2. 若要创建数据集 **BlobDataset**，请运行 **Set-AzDataFactoryV2Dataset** cmdlet。

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName -Name "BlobDataset" `
        -DefinitionFile ".\BlobDataset.json"
    ```

    下面是示例输出：

    ```console
    DatasetName       : BlobDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

## <a name="create-a-pipeline"></a>创建管道

在此快速入门中，所创建管道包含的一个活动采用两个参数 - 输入 Blob 路径和输出 Blob 路径。 这些参数的值是在触发/运行管道时设置的。 复制活动使用在上一步中创建的同一 blob 数据集作为输入和输出。 当该数据集用作输入数据集时，即指定了输入路径。 并且，当该数据集用作输出数据集时，即指定了输出路径。

1. 在 **C:\ADFv2QuickStartPSH** 文件夹中创建一个名为 **Adfv2QuickStartPipeline.json** 的 JSON 文件，使其包含以下内容：

    ```json
    {
        "name": "Adfv2QuickStartPipeline",
        "properties": {
            "activities": [
                {
                    "name": "CopyFromBlobToBlob",
                    "type": "Copy",
                    "inputs": [
                        {
                            "referenceName": "BlobDataset",
                            "parameters": {
                                "path": "@pipeline().parameters.inputPath"
                            },
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "BlobDataset",
                            "parameters": {
                                "path": "@pipeline().parameters.outputPath"
                            },
                            "type": "DatasetReference"
                        }
                    ],
                    "typeProperties": {
                        "source": {
                            "type": "BlobSource"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    }
                }
            ],
            "parameters": {
                "inputPath": {
                    "type": "String"
                },
                "outputPath": {
                    "type": "String"
                }
            }
        }
    }
    ```

2. 若要创建管道 **Adfv2QuickStartPipeline**，请运行 **Set-AzDataFactoryV2Pipeline** cmdlet。

    ```powershell
    $DFPipeLine = Set-AzDataFactoryV2Pipeline `
        -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName `
        -Name "Adfv2QuickStartPipeline" `
        -DefinitionFile ".\Adfv2QuickStartPipeline.json"
    ```

## <a name="create-a-pipeline-run"></a>创建管道运行

在此步骤中，将为管道参数设置值：将 **inputPath** 和 **outputPath** 的值替换为源和接收器 blob 路径的实际值。 然后，使用这些参数创建管道运行。

1. 在 **C:\ADFv2QuickStartPSH** 文件夹中创建一个名为 **PipelineParameters.json** 的 JSON 文件，使其包含以下内容：

    ```json
    {
        "inputPath": "adftutorial/input",
        "outputPath": "adftutorial/output"
    }
    ```
2. 运行 **Invoke-AzDataFactoryV2Pipeline** cmdlet 来创建一个管道运行并传入参数值。 此 cmdlet 返回管道运行 ID，用于将来的监视。

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline `
        -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName `
        -PipelineName $DFPipeLine.Name `
        -ParameterFile .\PipelineParameters.json
    ```

## <a name="monitor-the-pipeline-run"></a>监视管道运行

1. 运行以下 PowerShell 脚本，持续检查管道运行状态，直到完成数据复制为止。 在 PowerShell 窗口中复制/粘贴以下脚本，然后按 ENTER。

    ```powershell
    while ($True) {
        $Run = Get-AzDataFactoryV2PipelineRun `
            -ResourceGroupName $ResGrp.ResourceGroupName `
            -DataFactoryName $DataFactory.DataFactoryName `
            -PipelineRunId $RunId

        if ($Run) {
            if ($run.Status -ne 'InProgress') {
                Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
                $Run
                break
            }
            Write-Output "Pipeline is running...status: InProgress"
        }

        Start-Sleep -Seconds 10
    }
    ```

    下面是管道运行的示例输出：

    ```console
    Pipeline is running...status: InProgress
    Pipeline run finished. The status is:  Succeeded
    
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : SPTestFactory0928
    RunId             : 0000000000-0000-0000-0000-0000000000000
    PipelineName      : Adfv2QuickStartPipeline
    LastUpdated       : 9/28/2017 8:28:38 PM
    Parameters        : {[inputPath, adftutorial/input], [outputPath, adftutorial/output]}
    RunStart          : 9/28/2017 8:28:14 PM
    RunEnd            : 9/28/2017 8:28:38 PM
    DurationInMs      : 24151
    Status            : Succeeded
    Message           :
    ```

    你可能会看到以下错误：

    ```console
    Activity CopyFromBlobToBlob failed: Failed to detect region of linked service 'AzureStorage' : 'AzureStorageLinkedService' with error '[Region Resolver] Azure Storage failed to get address for DNS. Warning: System.Net.Sockets.SocketException (0x80004005): No such host is known
    ```

    如果看到该错误，请执行以下步骤：

    1. 在 AzureStorageLinkedService.json 中，确认 Azure 存储帐户的名称和密钥是否正确。
    2. 验证连接字符串的格式是否正确。 例如，属性 AccountName 和 AccountKey 通过分号 (`;`) 字符隔开。
    3. 如果帐户名称和帐户密钥带尖括号，请删除尖括号。
    4. 下面是一个连接字符串示例：

        ```json
        "connectionString": {
            "value": "DefaultEndpointsProtocol=https;AccountName=mystorageaccountname;AccountKey=mystorageaccountkey;EndpointSuffix=core.windows.net",
            "type": "SecureString"
        }
        ```

    5. 按[创建链接服务](#create-a-linked-service)部分的步骤重新创建链接服务。
    6. 按[创建管道运行](#create-a-pipeline-run)部分的步骤重新运行管道。
    7. 再次运行当前的监视命令，监视新的管道运行。

2. 运行以下脚本来检索复制活动运行详细信息，例如，读取/写入的数据的大小。

    ```powershell
    Write-Output "Activity run details:"
    $Result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineRunId $RunId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $Result

    Write-Output "Activity 'Output' section:"
    $Result.Output -join "`r`n"

    Write-Output "Activity 'Error' section:"
    $Result.Error -join "`r`n"
    ```
3. 确认你看到了与活动运行结果的以下示例输出类似的输出：

    ```console
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : SPTestFactory0928
    ActivityName      : CopyFromBlobToBlob
    PipelineRunId     : 00000000000-0000-0000-0000-000000000000
    PipelineName      : Adfv2QuickStartPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, copyDuration, throughput...}
    LinkedServiceName :
    ActivityRunStart  : 9/28/2017 8:28:18 PM
    ActivityRunEnd    : 9/28/2017 8:28:36 PM
    DurationInMs      : 18095
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity 'Output' section:
    "dataRead": 38
    "dataWritten": 38
    "copyDuration": 7
    "throughput": 0.01
    "errors": []
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (West US)"
    "usedDataIntegrationUnits": 2
    "billedDuration": 14
    ```

[!INCLUDE [data-factory-quickstart-verify-output-cleanup.md](../../includes/data-factory-quickstart-verify-output-cleanup.md)]

## <a name="next-steps"></a>后续步骤

此示例中的管道将数据从 Azure Blob 存储中的一个位置复制到另一个位置。 完成相关[教程](tutorial-copy-data-dot-net.md)来了解如何在更多方案中使用数据工厂。
