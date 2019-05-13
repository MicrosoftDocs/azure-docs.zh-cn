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
ms.date: 05/06/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 4f3f43ba063ed25389860183576c4dbd94e2bfcd
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466315"
---
# <a name="tutorial-use-health-check-in-azure-deployment-manager-public-preview"></a>教程：在 Azure 部署管理器（公共预览版）中使用运行状况检查

了解如何在 [Azure 部署管理器](./deployment-manager-overview.md)中集成运行状况检查。 本教程基于[将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)教程。 只有在完成该教程之后才能继续学习本教程。

在[将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)教程中使用的实施模板中，你使用了一个等待步骤。 在本教程中，你要将该等待步骤替换为运行状况检查步骤。

> [!IMPORTANT]
> 如果为 Canary 标记了你的订阅来测试新的 Azure 功能，则只能使用 Azure 部署管理器来部署到 Canary 区域。 

本教程涵盖以下任务：

> [!div class="checklist"]
> * 创建运行状况检查服务模拟器
> * 修订实施模板
> * 部署拓扑
> * 部署状态不正常的实施方案
> * 验证实施部署
> * 部署状态正常的实施方案
> * 验证实施部署
> * 清理资源

可以在[此处](https://docs.microsoft.com/rest/api/deploymentmanager/)找到 Azure 部署管理器 REST API 参考。

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 完成[将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)教程。
* 下载本教程使用的[模板和项目](https://armtutorials.blob.core.windows.net/admtutorial/ADMTutorial.zip)。 

## <a name="create-a-health-check-service-simulator"></a>创建运行状况检查服务模拟器

在生产环境中，通常会使用一个或多个监视提供程序。 为了尽量简化运行状况集成，Microsoft 一直与某些顶级服务运行状况监视公司合作，以便为客户提供简单的复制/粘贴解决方案，用于将运行状况检查与部署相集成。 有关公司列表，请参阅[运行状况监视提供商](./deployment-manager-health-check.md#health-monitoring-providers)。 在本教程中，你将创建一个 [Azure 函数](/azure/azure-functions/)来模拟运行状况监视服务。 此函数提取一个状态代码，并返回相同的代码。 Azure 部署管理器模板使用该状态代码来确定如何继续部署。 

以下两个文件用于部署 Azure 函数。 无需下载这些文件即可完成本教程。

* 资源管理器模板位于 [https://armtutorials.blob.core.windows.net/admtutorial/deploy_hc_azure_function.json](https://armtutorials.blob.core.windows.net/admtutorial/deploy_hc_azure_function.json)。 你将部署此模板以创建 Azure 函数。  
* Azure 函数源代码的 zip 文件 [https://armtutorials.blob.core.windows.net/admtutorial/ADMHCFunction0417.zip](https://armtutorials.blob.core.windows.net/admtutorial/RestHealthTest.zip)。 资源管理器模板将调用此 zip 文件。

若要部署 Azure 函数，请选择“尝试”打开 Azure Cloud shell，然后在 shell 窗口中粘贴以下脚本。  若要粘贴代码，请右键单击 shell 窗口并选择“粘贴”。 

> [!IMPORTANT]
> PowerShell 脚本中的 **projectName** 用于生成要在本教程中部署的 Azure 服务的名称。 不同 Azure 服务对名称的要求各不相同。 为确保成功部署，请选择仅包含小写字母和数字且长度不超过 12 个字符的名称。
> 保存项目名称的副本。 整篇教程都会使用相同的项目名称。

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://armtutorials.blob.core.windows.net/admtutorial/deploy_hc_azure_function.json" -projectName $projectName
```

若要验证和测试 Azure 函数：

1. 打开 [Azure 门户](https://portal.azure.com)。
1. 打开资源组。  默认名称是追加了 **rg** 的项目名称。
1. 从资源组中选择应用服务。  应用服务的默认名称是追加了 **webapp** 的项目名称。
1. 展开“函数”，然后选择“HttpTrigger1”。 

    ![Azure 部署管理器运行状况检查 - Azure 函数](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-function.png)

1. 选择“&lt;/> 获取函数 URL”。
1. 选择“复制”将 URL 复制到剪贴板。  URL 类似于：

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/{healthStatus}?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    将 URL 中的 `{healthStatus}` 替换为某个状态代码。 本教程使用 **unhealthy** 测试不正常的方案，并使用 **healthy** 或 **warning** 测试正常的方案。 创建两个 URL，其中一个包含不正常状态，另一个包含正常状态。 例如：

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/healthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    需要使用这两个 URL 才能完成本教程。

1. 若要测试运行状况监视模拟器，请打开上一步骤中创建的 URL。  不正常状态的结果应类似于：

    ```
    Status: unhealthy
    ```

## <a name="revise-the-rollout-template"></a>修订实施模板

本部分旨在介绍如何在实施模板中包含运行状况检查步骤。 无需创建自己的 CreateADMRollout.json 文件即可完成本教程。 修订的实施模板将在用于后续部分的存储帐户中共享。

1. 打开 **CreateADMRollout.json**。 此 JSON 文件是下载内容的一部分。  请参阅[先决条件](#prerequisites)。
1. 额外添加两个参数：

    ```json
    "healthCheckUrl": {
        "type": "string",
        "metadata": {
            "description": "Specifies the health check URL."
        }
    },
    "healthCheckAuthAPIKey": {
        "type": "string",
        "metadata": {
            "description": "Specifies the health check Azure Function function authorization key."
        }
    }
    ```

1. 请将等待步骤资源定义替换为运行状况检查步骤资源定义：

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

    根据定义，如果运行状态为 *healthy* 或 *warning*，则实施将会继续。 

1. 更新实施定义的 **dependsON**，以包含新定义的运行状况检查步骤：

    ```json
    "dependsOn": [
        "[resourceId('Microsoft.DeploymentManager/artifactSources', variables('rolloutArtifactSource').name)]",
        "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
    ],
    ```

1. 更新 **stepGroups**，以包含运行状况检查步骤。 **healthCheckStep** 在 **stepGroup2** 的 **postDeploymentSteps** 中调用。 仅当运行状态为 *healthy* 或 *warning* 时，才部署 **stepGroup3** 和 **stepGroup4**。 

    ```json
    "stepGroups": [
        {
            "name": "stepGroup1",
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup2",
            "dependsOnStepGroups": ["stepGroup1"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
            "postDeploymentSteps": [
                {
                    "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
                }
            ]
        },
        {
            "name": "stepGroup3",
            "dependsOnStepGroups": ["stepGroup2"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup4",
            "dependsOnStepGroups": ["stepGroup3"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
            "postDeploymentSteps": []
        }
    ]
    ```

    如果对 **stepGroup3** 节在修订之前和之后的值进行比较，会发现此节现在依赖于 **stepGroup2**。  如果 **stepGroup3** 和后续步骤组依赖于运行状况监视的结果，则这种依赖关系是必要的。

    以下屏幕截图演示了修改的区域，以及如何使用运行状况检查步骤：

    ![Azure 部署管理器运行状况检查模板](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-rollout-template.png)

## <a name="deploy-the-topology"></a>部署拓扑

为了简化本教程，拓扑模板和项目将在以下位置共享，因此你不需要准备自己的副本。 若要使用自己的副本，请遵照以下文档中的说明操作：[教程：将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)。

* 拓扑模板： https://armtutorials.blob.core.windows.net/admtutorial/ADMTemplates/CreateADMServiceTopology.json
* 项目存储： https://armtutorials.blob.core.windows.net/admtutorial/ArtifactStore

若要部署拓扑，请选择“尝试”打开 Cloud shell，然后粘贴 PowerShell 脚本。

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name used earlier in this tutorial"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$resourceGroupName = "${projectName}rg"
$artifactLocation = "https://armtutorials.blob.core.windows.net/admtutorial/ArtifactStore?st=2019-05-06T03%3A57%3A31Z&se=2020-05-07T03%3A57%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=gOh%2Bkhi693rmdxiZFQ9xbKZMU1kbLJDqXw7EP4TaGlI%3D" | ConvertTo-SecureString -AsPlainText -Force

# Create the service topology
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://armtutorials.blob.core.windows.net/admtutorial/ADMTemplatesHC/CreateADMServiceTopology.json" `
    -namePrefix $projectName `
    -azureResourceLocation $location `
    -artifactSourceSASLocation $artifactLocation
```

使用 Azure 门户验证是否已成功创建服务拓扑和带下划线的资源：

![Azure 部署管理器教程 - 已部署服务拓扑资源](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

必须选择“显示隐藏的类型”才能查看资源。

## <a name="deploy-the-rollout-with-the-unhealthy-status"></a>部署状态不正常的实施方案

为了简化本教程，修订的实施模板将在以下位置共享，因此你不需要准备自己的副本。 若要使用自己的副本，请遵照以下文档中的说明操作：[教程：将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)。

* 拓扑模板： https://armtutorials.blob.core.windows.net/admtutorial/ADMTemplatesHC/CreateADMRollout.json
* 项目存储： https://armtutorials.blob.core.windows.net/admtutorial/ArtifactStore

使用在[创建运行状况检查服务模拟器](#create-a-health-check-service-simulator)中创建的不正常状态 URL。 对于 **managedIdentityID**，请参阅[创建用户分配的托管标识](./deployment-manager-tutorial.md#create-the-user-assigned-managed-identity)。

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name used earlier in this tutorial"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$managedIdentityID = Read-Host -Prompt "Enter a user-assigned managed identity"
$healthCheckUrl = Read-Host -Prompt "Enter the health check Azure function URL"
$healthCheckAuthAPIKey = $healthCheckUrl.Substring($healthCheckUrl.IndexOf("?code=")+6, $healthCheckUrl.Length-$healthCheckUrl.IndexOf("?code=")-6)
$healthCheckUrl = $healthCheckUrl.Substring(0, $healthCheckUrl.IndexOf("?"))

$resourceGroupName = "${projectName}rg"
$artifactLocation = "https://armtutorials.blob.core.windows.net/admtutorial/ArtifactStore?st=2019-05-06T03%3A57%3A31Z&se=2020-05-07T03%3A57%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=gOh%2Bkhi693rmdxiZFQ9xbKZMU1kbLJDqXw7EP4TaGlI%3D" | ConvertTo-SecureString -AsPlainText -Force

# Create the rollout
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://armtutorials.blob.core.windows.net/admtutorial/ADMTemplatesHC/CreateADMRollout.json" `
    -namePrefix $projectName `
    -azureResourceLocation $location `
    -artifactSourceSASLocation $artifactLocation `
    -managedIdentityID $managedIdentityID `
    -healthCheckUrl $healthCheckUrl `
    -healthCheckAuthAPIKey $healthCheckAuthAPIKey
```

> [!NOTE]
> `New-AzResourceGroupDeployment` 是异步调用。 成功消息只意味着部署已成功开始。 若要验证部署，请使用 `Get-AZDeploymentManagerRollout`。  查看下一过程。

若要使用以下 PowerShell 脚本检查实施进度：

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name used earlier in this tutorial"
$resourceGroupName = "${projectName}rg"
$rolloutName = "${projectName}Rollout"

# Get the rollout status
Get-AzDeploymentManagerRollout `
    -ResourceGroupName $resourceGroupName `
    -Name $rolloutName `
    -Verbose
```

以下示例输出显示不正常的状态导致部署失败：

```output
Service: myhc0417ServiceWUSrg
    TargetLocation: WestUS
    TargetSubscriptionId: <Subscription ID>

    ServiceUnit: myhc0417ServiceWUSWeb
        TargetResourceGroup: myhc0417ServiceWUSrg

        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
Get-AzDeploymentManagerRollout :
Service: myhc0417ServiceWUSrg
    ServiceUnit: myhc0417ServiceWUSWeb
        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
At line:1 char:1
+ Get-AzDeploymentManagerRollout `
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Get-AzDeploymentManagerRollout], Exception
+ FullyQualifiedErrorId : RolloutFailed,Microsoft.Azure.Commands.DeploymentManager.Commands.GetRollout


ResourceGroupName       : myhc0417rg
BuildVersion            : 1.0.0.0
ArtifactSourceId        : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/artifactSources/myhc0417ArtifactSourceRollout
TargetServiceTopologyId : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/serviceTopologies/myhc0417ServiceTopology
Status                  : Failed
TotalRetryAttempts      : 0
Identity                : Microsoft.Azure.Commands.DeploymentManager.Models.PSIdentity
OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
Services                : {myhc0417ServiceWUS, myhc0417ServiceWUSrg}
Name                    : myhc0417Rollout
Type                    : Microsoft.DeploymentManager/rollouts
Location                : centralus
Id                      : /subscriptions/<Subscription ID>/resourcegroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/rollouts/myhc0417Rollout
Tags                    :
```

完成实施后，你会看到针对“美国西部”区域创建了一个附加的资源组。

## <a name="deploy-the-rollout-with-the-healthy-status"></a>部署状态正常的实施方案

重复本部分中的步骤，以使用正常状态 URL 重新部署实施方案。  完成实施后，你会看到针对“美国东部”区域创建了另一个资源组。

## <a name="verify-the-deployment"></a>验证部署

1. 打开 [Azure 门户](https://portal.azure.com)。
2. 在实施部署创建的新资源组下面浏览到新建的 Web 应用程序。
3. 在 Web 浏览器中打开该 Web 应用程序。 在 index.html 文件中检查位置和版本。

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

本教程已介绍如何使用 Azure 部署管理器的运行状况检查功能。 若要了解更多信息，请参阅 [Azure 资源管理器文档](/azure/azure-resource-manager/)。
