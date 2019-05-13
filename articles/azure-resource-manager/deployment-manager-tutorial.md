---
title: 将 Azure 部署管理器与资源管理器模板配合使用 | Microsoft Docs
description: 配合使用资源管理器模板与 Azure 部署管理器来部署 Azure 资源。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 04/02/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: a4f14a1e68042704ca8e8c49f1bd76b722c90d4d
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466303"
---
# <a name="tutorial-use-azure-deployment-manager-with-resource-manager-templates-public-preview"></a>教程：将 Azure 部署管理器与资源管理器模板配合使用（公共预览版）

了解如何使用 [Azure 部署管理器](./deployment-manager-overview.md)跨多个区域部署应用程序。 若要使用部署管理器，需要创建两个模板：

* **拓扑模板**：描述构成应用程序的 Azure 资源以及这些资源的部署位置。
* **实施模板**：描述部署应用程序时要执行的步骤。

> [!IMPORTANT]
> 如果为 Canary 标记了你的订阅来测试新的 Azure 功能，则只能使用 Azure 部署管理器来部署到 Canary 区域。 

本教程涵盖以下任务：

> [!div class="checklist"]
> * 了解方案
> * 下载教程文件
> * 准备项目
> * 创建用户定义的托管标识
> * 创建服务拓扑模板
> * 创建实施模板
> * 部署模板
> * 验证部署
> * 部署较新的版本
> * 清理资源

可以在[此处](https://docs.microsoft.com/rest/api/deploymentmanager/)找到 Azure 部署管理器 REST API 参考。

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 在开发 [Azure 资源管理器模板](./resource-group-overview.md)方面有一定的经验。
* Azure 部署管理器为专用预览版。 若要使用 Azure 部署管理器注册，请填写[注册表单](https://aka.ms/admsignup)。 
* Azure PowerShell。 有关详细信息，请参阅 [Azure PowerShell 入门](https://docs.microsoft.com/powershell/azure/get-started-azureps)。
* 部署管理器 cmdlet。 若要安装这些预发行版 cmdlet，需要最新版本的 PowerShellGet。 若要获取最新版本，请参阅[安装 PowerShellGet](/powershell/gallery/installing-psget)。 安装 PowerShellGet 后，关闭 PowerShell 窗口。 打开新的提升的 PowerShell 窗口并使用以下命令：

    ```powershell
    Install-Module -Name Az.DeploymentManager
    ```

* [Microsoft Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)。 Azure 存储资源管理器不是必需的，但可以简化操作。

## <a name="understand-the-scenario"></a>了解方案

服务拓扑模板描述构成服务的 Azure 资源以及这些资源的部署位置。 服务拓扑定义采用以下层次结构：

* 服务拓扑
  * 服务
    * 服务单元

下图演示了本教程中使用的服务拓扑：

![Azure 部署管理器教程方案示意图](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-scenario-diagram.png)

美国西部和美国东部位置分配了两个服务。  每个服务有两个服务单元 - Web 应用程序前端，以及后端的存储帐户。 服务单元定义包含用于创建 Web 应用程序和存储帐户的模板与参数文件的链接。

## <a name="download-the-tutorial-files"></a>下载教程文件

1. 下载本教程使用的[模板和项目](https://armtutorials.blob.core.windows.net/admtutorial/ADMTutorial.zip)。
2. 将文件解压缩到本地计算机。

根文件夹中包含两个文件夹：

* **ADMTemplates**：包含部署管理器模板，其中包括：
  * CreateADMServiceTopology.json
  * CreateADMServiceTopology.Parameters.json
  * CreateADMRollout.json
  * CreateADMRollout.Parameters.json
* **ArtifactStore**：包含模板项目和二进制项目。 请参阅[准备项目](#prepare-the-artifacts)。

请注意，有两个模板集。  一个集是用于部署服务拓扑和实施的部署管理器模板；另一个集从服务单元调用，用于创建 Web 服务和存储帐户。

## <a name="prepare-the-artifacts"></a>准备项目

下载的 ArtifactStore 文件夹中包含两个文件夹：

![Azure 部署管理器教程项目源示意图](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-artifact-source-diagram.png)

* **templates** 文件夹：包含模板项目。 **1.0.0.0** 和 **1.0.0.1** 代表二进制项目的两个版本。 在每个版本中，每个服务（美国东部服务和美国西部服务）各对应一个文件夹。 每个服务具有一对用于创建存储帐户的模板和参数文件，以及另一对用于创建 Web 应用程序的模板和参数文件。 Web 应用程序模板调用包含 Web 应用程序文件的压缩包。 压缩文件是存储在 binaries 文件夹中的二进制项目。
* **binaries** 文件夹：包含二进制项目。 **1.0.0.0** 和 **1.0.0.1** 代表二进制项目的两个版本。 在每个版本中，有一个用于在美国西部位置创建 Web 应用程序的 zip 文件，以及另一个用于在美国东部位置创建 Web 应用程序的 zip 文件。

两个版本（1.0.0.0 和 1.0.0.1）用于[修订版部署](#deploy-the-revision)。 尽管模板项目和二进制项目都有两个版本，但只有二进制项目的两个版本存在差别。 在实践中，二进制项目的更新频繁比模板项目更高。

1. 在文本编辑器中打开 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateStorageAccount.json**。 此文件是用于创建存储帐户的基本模板。  
2. 打开 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplication.json**。 

    ![Azure 部署管理器教程 - 创建 Web 应用程序模板](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-packageuri.png)

    该模板调用包含 Web 应用程序文件的部署包。 在本教程中，压缩包仅包含 index.html 文件。
3. 打开 **\ArtifactStore\templates\1.0.0.0\ServiceWUS\CreateWebApplicationParameters.json**。 

    ![Azure 部署管理器教程 - 创建 Web 应用程序模板参数 containerRoot](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-create-web-application-parameters-deploypackageuri.png)

    deployPackageUri 的值是部署包的路径。 参数包含 **$containerRoot** 变量。 $containerRoot 的值是在[实施模板](#create-the-rollout-template)中通过连接项目源 SAS 位置、项目根和 deployPackageUri 提供的。
4. 打开 **\ArtifactStore\binaries\1.0.0.0\helloWorldWebAppWUS.zip\index.html**。  

    ```html
    <html>
      <head>
        <title>Azure Deployment Manager tutorial</title>
      </head>
      <body>
        <p>Hello world from west U.S.!</p>
        <p>Version 1.0.0.0</p>
      </body>
    </html>
    ```

    html 显示位置和版本信息。 1.0.0.1 文件夹中的二进制文件显示“Version 1.0.0.1”。 部署服务后，可以浏览到这些页面。
5. 签出其他项目文件。 这有助于更好地了解方案。

模板项目由服务拓扑模板使用，二进制项目由实施模板使用。 拓扑模板和实施模板都定义一个项目源 Azure 资源，该资源用于将资源管理器指向部署中使用的模板和二进制项目。 为了简化本教程，将使用一个存储帐户来存储模板项目和二进制项目。 这两个项目源指向同一个存储帐户。

1. 创建 Azure 存储帐户。 有关说明，请参阅[快速入门：使用 Azure 门户上传、下载和列出 Blob](../storage/blobs/storage-quickstart-blobs-portal.md)。
2. 在存储帐户中创建 Blob 容器。
3. 将两个文件夹（binaries 和 templates）及其内容复制到 Blob 容器。 [Microsoft Azure 存储资源管理器](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409)支持拖放功能。
4. 遵照以下说明获取容器的 SAS 位置：

    1. 在 Azure 存储资源管理器中，导航到 Blob 容器。
    2. 在左窗格中右键单击 Blob 容器，然后选择“获取共享访问签名”。
    3. 配置“开始时间”和“过期时间”。
    4. 选择“创建”。
    5. 复制 URL。 需使用此 URL 来填充两个参数文件（[拓扑参数文件](#topology-parameters-file)和[实施参数文件](#rollout-parameters-file)）中的字段。

## <a name="create-the-user-assigned-managed-identity"></a>创建用户分配的托管标识

在本教程稍后，我们将部署一个实施资源。 需使用用户分配的托管标识来执行部署操作（例如，部署 Web 应用程序和存储帐户）。 此标识必须有权访问服务所要部署到的 Azure 订阅，并且有足够的权限来完成项目部署。

需要创建用户分配的托管标识，并为订阅配置访问控制。

> [!IMPORTANT]
> 用户分配的托管标识必须与[实施](#create-the-rollout-template)项目位于同一位置。 目前，只能在美国中部或美国东部 2 区域中创建部署管理器资源，包括实施。 但是，这仅适用于部署管理器资源（例如服务拓扑、服务、服务单元、推出和步骤）。 可以将目标资源部署到任何支持的 Azure 区域。 例如，在本教程中，部署管理器资源部署到美国中部，但服务部署到美国东部和美国西部。 此限制将来会取消。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 创建[用户分配的托管标识](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)。
3. 在门户的左侧菜单中选择“订阅”，然后选择自己的订阅。
4. 依次选择“访问控制(IAM)”、“添加角色分配”
5. 输入或选择下列值：

    ![Azure 部署管理器教程 - 用户分配的托管标识访问控制](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-access-control.png)

    * **角色**：授予足够的权限以完成项目部署（Web 应用程序和存储帐户）。 对于本教程，请选择“参与者”。 在实践中，最好是将权限限制为最低权限。
    * **已将访问权限分配到**：选择“用户分配的托管标识”。
    * 选择在本教程前面创建的用户分配的托管标识。
6. 选择“保存”。

## <a name="create-the-service-topology-template"></a>创建服务拓扑模板

打开 **\ADMTemplates\CreateADMServiceTopology.json**。

### <a name="the-parameters"></a>参数

模板包含以下参数：

![Azure 部署管理器教程 - 拓扑模板参数](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-parameters.png)

* **namePrefix**：此前缀用于创建部署管理器资源的名称。 例如，如果使用“jdoe”前缀，则服务拓扑名称为 **jdoe**ServiceTopology。  资源名称在此模板的 variables 节中定义。
* **azureResourcelocation**：为了简化教程，除非另行指定，否则所有资源共享此位置。 目前，只能在“美国中部”或“美国东部 2”区域中创建 Azure 部署管理器资源。
* **artifactSourceSASLocation**：存储要部署的服务单元模板和参数文件的 Blob 容器的 SAS URI。  请参阅[准备项目](#prepare-the-artifacts)。
* **templateArtifactRoot**：与存储模板和参数的 Blob 容器之间的偏移路径。 默认值为 **templates/1.0.0.0**。 除非你要更改[准备项目](#prepare-the-artifacts)中所述的文件夹结构，否则请不要更改此值。 本教程使用相对路径。  完整路径是通过连接 **artifactSourceSASLocation**、**templateArtifactRoot** 和 **templateArtifactSourceRelativePath**（或 **parametersArtifactSourceRelativePath**）来构造的。
* **targetSubscriptionID**：部署管理器资源要部署到的并从中计费的订阅 ID。 本教程使用你的订阅 ID。

### <a name="the-variables"></a>变量

variables 节定义资源的名称、“WUS 服务”和“EUS 服务”这两个服务的 Azure 位置，以及项目路径：

![Azure 部署管理器教程 - 拓扑模板变量](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-variables.png)

请将项目路径与上传到存储帐户的文件夹结构进行比较。 请注意，项目路径是相对路径。 完整路径是通过连接 **artifactSourceSASLocation**、**templateArtifactRoot** 和 **templateArtifactSourceRelativePath**（或 **parametersArtifactSourceRelativePath**）来构造的。

### <a name="the-resources"></a>资源

在根级别定义了两个资源：项目源和服务拓扑。

项目源定义为：

![Azure 部署管理器教程 - 拓扑模板资源项目源](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-artifact-source.png)

以下屏幕截图只显示了服务拓扑、服务和服务单元定义的某些组成部分：

![Azure 部署管理器教程 - 拓扑模板资源服务拓扑](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-topology-template-resources-service-topology.png)

* **artifactSourceId** 用于将项目源资源关联到服务拓扑资源。
* **dependsOn**：所有服务拓扑资源依赖于项目源资源。
* **项目**指向模板项目。  此处使用了相对路径。 完整路径是通过连接 artifactSourceSASLocation（在项目源中定义）、artifactRoot（在项目源中定义）和 templateArtifactSourceRelativePath（或 parametersArtifactSourceRelativePath）来构造的。

### <a name="topology-parameters-file"></a>拓扑参数文件

创建与拓扑模板配合使用的参数文件。

1. 在 Visual Studio Code 或任何文本编辑器中打开 **\ADMTemplates\CreateADMServiceTopology.Parameters**。
2. 填充参数值：

    * **namePrefix**：输入包含 4-5 个字符的字符串。 此前缀用于创建唯一的 Azure 资源名称。
    * **azureResourceLocation**：如果你不熟悉 Azure 位置，请在本教程中使用 **centralus**。
    * **artifactSourceSASLocation**：输入用于存储要部署的服务单元模板和参数文件的根目录（Blob 容器）的 SAS URI。  请参阅[准备项目](#prepare-the-artifacts)。
    * **templateArtifactRoot**：除非你要更改项目的文件夹结构，否则请在本教程中使用 **templates/1.0.0.0**。
    * **targetScriptionID**：输入 Azure 订阅 ID。

> [!IMPORTANT]
> 拓扑模板和实施模板共享一些通用参数。 这些参数的值必须相同。 这些参数是：**namePrefix**、**azureResourceLocation** 和 **artifactSourceSASLocation**（在本教程中，这两个项目源共享同一个存储帐户）。

## <a name="create-the-rollout-template"></a>创建实施模板

打开 **\ADMTemplates\CreateADMRollout.json**。

### <a name="the-parameters"></a>参数

模板包含以下参数：

![Azure 部署管理器教程 - 实施模板参数](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-parameters.png)

* **namePrefix**：此前缀用于创建部署管理器资源的名称。 例如，如果使用“jdoe”前缀，则实施名称为 **jdoe**Rollout。  名称在模板的 variables 节中定义。
* **azureResourcelocation**：为了简化教程，除非另行指定，否则所有部署管理器资源共享此位置。 目前，只能在“美国中部”或“美国东部 2”区域中创建 Azure 部署管理器资源。
* **artifactSourceSASLocation**：存储要部署的服务单元模板和参数文件的根目录（Blob 容器）的 SAS URI。  请参阅[准备项目](#prepare-the-artifacts)。
* **binaryArtifactRoot**：默认值为 **binaries/1.0.0.0**。 除非你要更改[准备项目](#prepare-the-artifacts)中所述的文件夹结构，否则请不要更改此值。 本教程使用相对路径。  完整路径是通过连接 CreateWebApplicationParameters.json 中指定的 **artifactSourceSASLocation**、**binaryArtifactRoot** 和 **deployPackageUri** 来构造的。  请参阅[准备项目](#prepare-the-artifacts)。
* **managedIdentityID**：用户分配的托管标识，用于执行部署操作。 请参阅[创建用户分配的托管标识](#create-the-user-assigned-managed-identity)。

### <a name="the-variables"></a>变量

variables 节定义资源的名称。 请确保服务拓扑名称、服务名称和服务单元名称与[拓扑模板](#create-the-service-topology-template)中定义的名称匹配。

![Azure 部署管理器教程 - 实施模板变量](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-variables.png)

### <a name="the-resources"></a>资源

在根级别定义了三个资源：项目源、步骤和实施。

项目源定义与拓扑模板中的定义相同。  有关详细信息，请参阅[创建服务拓扑模板](#create-the-service-topology-template)。

以下屏幕截图显示了等待步骤定义：

![Azure 部署管理器教程 - 实施模板资源等待步骤](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-wait-step.png)

持续时间使用 [ISO 8601 标准](https://en.wikipedia.org/wiki/ISO_8601#Durations)。 例如，**PT1M**（字母必须大写）表示等待 1 分钟。 

以下屏幕截图只显示了实施定义的某些组成部分：

![Azure 部署管理器教程 - 实施模板资源实施](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-rollout-template-resources-rollout.png)

* **dependsOn**：实施资源依赖于项目源资源，以及定义的所有步骤。
* **artifactSourceId**：用于将项目源资源关联到实施资源。
* **targetServiceTopologyId**：用于将服务拓扑资源关联到实施资源。
* **deploymentTargetId**：服务拓扑资源的服务单元资源 ID。
* **preDeploymentSteps** 和 **postDeploymentSteps**：包含实施步骤。 在模板中调用等待步骤。
* **dependsOnStepGroups**：配置步骤组之间的依赖关系。

### <a name="rollout-parameters-file"></a>实施参数文件

创建与实施模板配合使用的参数文件。

1. 在 Visual Studio Code 或任何文本编辑器中打开 **\ADMTemplates\CreateADMRollout.Parameters**。
2. 填充参数值：

    * **namePrefix**：输入包含 4-5 个字符的字符串。 此前缀用于创建唯一的 Azure 资源名称。
    * **azureResourceLocation**：目前，只能在“美国中部”或“美国东部 2”区域中创建 Azure 部署管理器资源。
    * **artifactSourceSASLocation**：输入用于存储要部署的服务单元模板和参数文件的根目录（Blob 容器）的 SAS URI。  请参阅[准备项目](#prepare-the-artifacts)。
    * **binaryArtifactRoot**：除非你要更改项目的文件夹结构，否则请在本教程中使用 **binaries/1.0.0.0**。
    * **managedIdentityID**：输入用户分配的托管标识。 请参阅[创建用户分配的托管标识](#create-the-user-assigned-managed-identity)。 语法为：

        ```
        "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userassignedidentities/<ManagedIdentityName>"
        ```

> [!IMPORTANT]
> 拓扑模板和实施模板共享一些通用参数。 这些参数的值必须相同。 这些参数是：**namePrefix**、**azureResourceLocation** 和 **artifactSourceSASLocation**（在本教程中，这两个项目源共享同一个存储帐户）。

## <a name="deploy-the-templates"></a>部署模板

可以使用 Azure PowerShell 部署模板。 

1. 运行脚本以部署服务拓扑。

    ```azurepowershell
    $resourceGroupName = "<Enter a Resource Group Name>"
    $location = "Central US"  
    $filePath = "<Enter the File Path to the Downloaded Tutorial Files>"

    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location "$location"

    # Create the service topology
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
    ```

    > [!NOTE]
    > `New-AzResourceGroupDeployment` 是异步调用。 成功消息只意味着部署已成功开始。 若要验证部署，请参阅此过程的步骤 2 和步骤 4。

2. 使用 Azure 门户验证是否已成功创建服务拓扑和带下划线的资源：

    ![Azure 部署管理器教程 - 已部署服务拓扑资源](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

    必须选择“显示隐藏的类型”才能查看资源。

3. <a id="deploy-the-rollout-template"></a>部署推出模板：

    ```azurepowershell
    # Create the rollout
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
        -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json"
    ```

4. 使用以下 PowerShell 脚本检查实施进度：

    ```azurepowershell
    # Get the rollout status
    $rolloutname = "<Enter the Rollout Name>" # "adm0925Rollout" is the rollout name used in this tutorial
    Get-AzDeploymentManagerRollout `
        -ResourceGroupName $resourceGroupName `
        -Name $rolloutName `
        -Verbose
    ```

    在运行此 cmdlet 之前，必须先安装部署管理器 PowerShell cmdlet。 请参阅“先决条件”。 -Verbose 开关可用于查看整个输出。

    以下示例显示正在运行状态：

    ```
    VERBOSE:

    Status: Succeeded
    ArtifactSourceId: /subscriptions/<AzureSubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    BuildVersion: 1.0.0.0

    Operation Info:
        Retry Attempt: 0
        Skip Succeeded: False
        Start Time: 03/05/2019 15:26:13
        End Time: 03/05/2019 15:31:26
        Total Duration: 00:05:12

    Service: adm0925ServiceEUS
        TargetLocation: EastUS
        TargetSubscriptionId: <AzureSubscriptionID>

        ServiceUnit: adm0925ServiceEUSStorage
            TargetResourceGroup: adm0925ServiceEUSrg

            Step: Deploy
                Status: Succeeded
                StepGroup: stepGroup3
                Operation Info:
                    DeploymentName: 2F535084871E43E7A7A4CE7B45BE06510adm0925ServiceEUSStorage
                    CorrelationId: 0b6f030d-7348-48ae-a578-bcd6bcafe78d
                    Start Time: 03/05/2019 15:26:32
                    End Time: 03/05/2019 15:27:41
                    Total Duration: 00:01:08
                Resource Operations:

                    Resource Operation 1:
                    Name: txq6iwnyq5xle
                    Type: Microsoft.Storage/storageAccounts
                    ProvisioningState: Succeeded
                    StatusCode: OK
                    OperationId: 64A6E6EFEF1F7755

    ...

    ResourceGroupName       : adm0925rg
    BuildVersion            : 1.0.0.0
    ArtifactSourceId        : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/artifactSources/adm0925ArtifactSourceRollout
    TargetServiceTopologyId : /subscriptions/<SubscriptionID>/resourceGroups/adm0925rg/providers/Microsoft.DeploymentManager/serviceTopologies/adm0925ServiceTopology
    Status                  : Running
    TotalRetryAttempts      : 0
    OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
    Services                : {adm0925ServiceEUS, adm0925ServiceWUS}
    Name                    : adm0925Rollout
    Type                    : Microsoft.DeploymentManager/rollouts
    Location                : centralus
    Id                      : /subscriptions/<SubscriptionID>/resourcegroups/adm0925rg/providers/Microsoft.DeploymentManager/rollouts/adm0925Rollout
    Tags                    :
    ```

    成功部署实施资源后，应会看到额外创建了两个资源组（每个服务各有一个资源组）。

## <a name="verify-the-deployment"></a>验证部署

1. 打开 [Azure 门户](https://portal.azure.com)。
2. 在实施部署创建的新资源组下面浏览到新建的 Web 应用程序。
3. 在 Web 浏览器中打开该 Web 应用程序。 在 index.html 文件中检查位置和版本。

## <a name="deploy-the-revision"></a>部署修订版

为 Web 应用程序创建新版本 (1.0.0.1) 后， 可使用以下过程重新部署该 Web 应用程序。

1. 打开 CreateADMRollout.Parameters.json。
2. 将 **binaryArtifactRoot** 更新为 **binaries/1.0.0.1**。
3. 根据[部署模板](#deploy-the-rollout-template)中所述重新部署实施资源。
4. 根据[验证部署](#verify-the-deployment)中的说明验证部署。 网页应显示 1.0.0.1 版本。

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 使用“按名称筛选”字段来缩小本教程创建的资源组的范围。 应有 3-4 个资源组：

    * **&lt;namePrefix>rg**：包含部署管理器资源。
    * **&lt;namePrefix>ServiceWUSrg**：包含 ServiceWUS 定义的资源。
    * **&lt;namePrefix>ServiceEUSrg**：包含 ServiceEUS 定义的资源。
    * 用户定义的托管标识的资源组。
3. 选择资源组名称。  
4. 在顶部菜单中选择“删除资源组”。
5. 重复最后两个步骤，以删除本教程创建的其他资源组。

## <a name="next-steps"></a>后续步骤

本教程已介绍 Azure 部署管理器的用法。 若要在 Azure 部署管理器中集成运行状况监视，请参阅[教程：在 Azure 部署管理器中使用运行状况检查](./deployment-manager-tutorial-health-check.md)。
