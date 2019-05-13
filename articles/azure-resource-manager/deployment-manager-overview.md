---
title: 跨区域的安全部署实践 - Azure 部署管理器
description: 介绍如何使用 Azure 部署管理器在多个区域部署服务。 其中介绍了安全部署实践，可在部署到所有区域前验证部署的稳定性。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/09/2018
ms.author: tomfitz
ms.custom: seodec18
ms.openlocfilehash: dd7e29f8f37572565e505aade97b964254b6d72c
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466558"
---
# <a name="enable-safe-deployment-practices-with-azure-deployment-manager-public-preview"></a>启用安全部署实践与 Azure 部署管理器 （公共预览版）

要跨多个区域部署服务并确保它在每个区域中按预期运行，可以使用 Azure 部署管理器来协调服务的分阶段推出。 与任何 Azure 部署一样，需在[资源管理器模板](resource-group-authoring-templates.md)中为服务定义资源。 创建模板后，使用部署管理器描述服务的拓扑及其推出方式。

部署管理器是资源管理器的一项功能。 它将在部署过程中为你提供帮助。 如果有需要部署到多个区域的复杂服务，请使用部署管理器。 通过分阶段推出服务，你可以在服务已部署到所有区域之前发现潜在的问题。 如果不需要分阶段推出的额外预防措施，请使用资源管理器的标准[部署选项](resource-group-template-deploy-portal.md)。 部署管理器与支持资源管理器部署的所有现有第三方工具无缝集成，例如持续集成和持续交付 (CI/CD) 产品/服务。 

Azure 部署管理器现以个人预览版提供。 要使用 Azure 部署管理器，请完成[注册窗体](https://aka.ms/admsignup)。 请提供[反馈](https://aka.ms/admfeedback)，帮助改进功能。

要使用部署管理器，需要创建四个文件：

* 拓扑模板
* 推出模板
* 拓扑的参数文件
* 推出的参数文件

请在部署推出模板之前部署拓扑模板。

可以在[此处](https://docs.microsoft.com/rest/api/deploymentmanager/)找到 Azure 部署管理器 REST API 参考。

## <a name="supported-locations"></a>支持的位置

美国中部和美国东部 2 区支持预览版部署管理器资源。 定义拓扑和推出模板（如服务单位、项目源以及本文所述的推出）中的资源时，必须为该位置指定这两个区域中的一个区域。 但是，所有[标准位置](https://azure.microsoft.com/global-infrastructure/services/?products=all)都支持为创建服务而部署的资源，例如虚拟机、存储帐户和 Web 应用。  

## <a name="identity-and-access"></a>标识和访问

利用资源管理器，[用户分配的托管标识](../active-directory/managed-identities-azure-resources/overview.md)可执行部署操作。 请开始部署之前创建此标识。 它必须有权访问要将服务部署到的订阅并具有足够权限来完成部署。 有关通过角色授予的操作的详细信息，请参阅 [Azure 资源的内置角色](../role-based-access-control/built-in-roles.md)。

标识必须位于部署管理器支持的某个位置且必须与推出位于相同的位置。

## <a name="topology-template"></a>拓扑模板

拓扑模板描述构成服务的 Azure 资源以及这些资源的部署位置。 下图显示了示例服务的拓扑：

![从服务拓扑到服务再到服务单位的层次结构](./media/deployment-manager-overview/service-topology.png)

拓扑模板包括以下资源：

* 项目源 - 存储资源管理器和参数的位置
* 服务拓扑 - 指向项目源
  * 服务 - 指定位置和 Azure 订阅 ID
    * 服务单位 - 指定资源组、部署模式以及模板和参数文件的路径

要向了解每个级别发生的情况，查看你提供的值将很有帮助。

![每个级别的值](./media/deployment-manager-overview/topology-values.png)

### <a name="artifact-source-for-templates"></a>模板的项目源

在拓扑模板中，创建保存模板和参数文件的项目源。 项目源是一种拉取文件以进行部署的方法。 本文后面部分将展示针对二进制文件的另一个项目源。

以下示例显示了项目源的一般格式。

```json
{
    "type": "Microsoft.DeploymentManager/artifactSources",
    "name": "<artifact-source-name>",
    "location": "<artifact-source-location>",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "sourceType": "AzureStorage",
        "artifactRoot": "<root-folder-for-templates>",
        "authentication": {
            "type": "SAS",
            "properties": {
                "sasUri": "<SAS-URI-for-storage-container>"
            }
        }
    }
}
```

有关详细信息，请参阅 [artifactSources 模板引用](/azure/templates/Microsoft.DeploymentManager/artifactSources)。

### <a name="service-topology"></a>服务拓扑

以下示例显示了服务拓扑资源的一般格式。 需要提供保存模板和参数文件的项目源的资源 ID。 服务拓扑包括所有服务资源。 要确保项目源可用，服务拓扑需要依赖于它。

```json
{
    "type": "Microsoft.DeploymentManager/serviceTopologies",
    "name": "<topology-name>",
    "location": "<topology-location>",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "artifactSourceId": "<resource-ID-artifact-source>"
    },
    "dependsOn": [
        "<artifact-source>"
    ],
    "resources": [
        {
            "type": "services",
            ...
        }
    ]
}
```

有关详细信息，请参阅 [serviceTopologies 模板引用](/azure/templates/Microsoft.DeploymentManager/serviceTopologies)。

### <a name="services"></a>服务

以下示例显示了服务资源的一般格式。 在每个服务中，需要提供用于部署服务的位置和 Azure 订阅 ID。 要部署到多个区域，请为每个区域定义一个服务。 该服务依赖于服务拓扑。

```json
{
    "type": "services",
    "name": "<service-name>",
    "location": "<service-location>",
    "apiVersion": "2018-09-01-preview",
    "dependsOn": [
        "<service-topology>"
    ],
    "properties": {
        "targetSubscriptionId": "<subscription-ID>",
        "targetLocation": "<location-of-deployed-service>"
    },
    "resources": [
        {
            "type": "serviceUnits",
            ...
        }
    ]
}
```

有关详细信息，请参阅[服务模板引用](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services)。

### <a name="service-units"></a>服务单位

以下示例显示了服务单位资源的一般格式。 在每个服务单位中，需要指定资源组、用于部署的[部署模式](deployment-modes.md)以及模板和参数文件的路径。 如果为模板和参数指定相对路径，则完整路径将从项目源的根文件夹构造。 可以为模板和参数指定绝对路径，但将无法轻松地对版本进行版本控制。 服务单位依赖于服务。

```json
{
    "type": "serviceUnits",
    "name": "<service-unit-name>",
    "location": "<service-unit-location>",
    "apiVersion": "2018-09-01-preview",
    "dependsOn": [
        "<service>"
    ],
    "tags": {
        "serviceType": "Service West US Web App"
    },
    "properties": {
        "targetResourceGroup": "<resource-group-name>",
        "deploymentMode": "Incremental",
        "artifacts": {
            "templateArtifactSourceRelativePath": "<relative-path-to-template>",
            "parametersArtifactSourceRelativePath": "<relative-path-to-parameter-file>"
        }
    }
}
```

每个模板都应包含你想要一步部署的相关资源。 例如，服务单位可以包含为服务前端部署所有资源的模板。

有关详细信息，请参阅 [serviceUnits 模板引用](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits)。

## <a name="rollout-template"></a>推出模板

推出模板描述部署服务时要执行的步骤。 需要指定要使用的服务拓扑，并定义部署服务单位的顺序。 它包括用于存储部署二进制文件的项目源。 在推出模板中定义以下层次结构：

* 项目源
* 步骤
* 推出
  * 步骤组
    * 部署操作

下图显示了推出模板的层次结构：

![从推出到步骤的层次结构](./media/deployment-manager-overview/Rollout.png)

每个推出可能有多个步骤组。 每个步骤组具有一个指向服务拓扑中的服务单位的部署操作。

### <a name="artifact-source-for-binaries"></a>二进制文件的项目源

在推出模板中，为需要部署到服务的二进制文件创建项目源。 此项目源类似于[模板的项目源](#artifact-source-for-templates)，但它包含脚本、网页、编译代码或服务所需的其他文件。

### <a name="steps"></a>Steps

可以定义在部署操作之前或之后执行的步骤。 目前，仅`wait`步骤和运行状况检查步骤都可用。 

wait 步骤将先暂停部署，然后才能继续部署。 此步骤允许在部署下一个服务单位之前验证服务是否按预期运行。 以下示例显示了 wait 步骤的一般格式。

```json
{
    "apiVersion": "2018-09-01-preview",
    "type": "Microsoft.DeploymentManager/steps",
    "name": "waitStep",
        "location": "<step-location>",
    "properties": {
        "stepType": "wait",
        "attributes": {
          "duration": "PT1M"
        }
    }
},
```

持续时间属性使用 [ISO 8601 标准](https://en.wikipedia.org/wiki/ISO_8601#Durations)。 前面的示例指定了一分钟的等待时间。

有关运行状况检查步骤的详细信息，请参阅[]()并[]()有关详细信息，请参阅[步骤模板引用](/azure/templates/Microsoft.DeploymentManager/steps)。

### <a name="rollouts"></a>推出

要确保项目源可用，推出需要依赖于它。 推出定义已部署的每个服务单位的步骤组。 可以定义在部署之前或之后要执行的操作。 例如，可以指定部署在部署服务单位后进行等待。 可以定义步骤组的顺序。

标识对象指定执行部署操作的[用户分配的托管标识](#identity-and-access)。

以下示例显示了推出的一般格式。

```json
{
    "type": "Microsoft.DeploymentManager/rollouts",
    "name": "<rollout-name>",
    "location": "<rollout-location>",
    "apiVersion": "2018-09-01-preview",
    "Identity": {
        "type": "userAssigned",
        "identityIds": [
            "<managed-identity-ID>"
        ]
    },
    "dependsOn": [
        "<artifact-source>"
    ],
    "properties": {
        "buildVersion": "1.0.0.0",
        "artifactSourceId": "<artifact-source-ID>",
        "targetServiceTopologyId": "<service-topology-ID>",
        "stepGroups": [
            {
                "name": "stepGroup1",
                "dependsOnStepGroups": ["<step-group-name>"],
                "preDeploymentSteps": ["<step-ID>"],
                "deploymentTargetId":
                    "<service-unit-ID>",
                "postDeploymentSteps": ["<step-ID>"]
            },
            ...
        ]
    }
}
```

有关详细信息，请参阅[推出模板引用](/azure/templates/Microsoft.DeploymentManager/rollouts)。

## <a name="parameter-file"></a>参数文件

需要创建两个参数文件。 一个参数文件在部署服务拓扑时使用，另一个用于推出部署。 需要确保两个参数文件中的某些值相同。  

## <a name="containerroot-variable"></a>containerRoot 变量

对于已版本控制的部署，项目的路径随每个新版本而发生更改。 首次运行部署时，路径可能为 `https://<base-uri-blob-container>/binaries/1.0.0.0`。 第二次可能为 `https://<base-uri-blob-container>/binaries/1.0.0.1`。 部署管理器使用 `$containerRoot` 变量简化获取当前部署的正确根路径。 此值随每个版本而发生更改，并且在部署前未知。

使用模板参数文件中的 `$containerRoot` 变量部署 Azure 资源。 部署时，此变量将替换为推出的实际值。 

例如，在推出期间为二进制项目创建项目源。

```json
{
    "type": "Microsoft.DeploymentManager/artifactSources",
    "name": "[variables('rolloutArtifactSource').name]",
    "location": "[parameters('azureResourceLocation')]",
    "apiVersion": "2018-09-01-preview",
    "properties": {
        "sourceType": "AzureStorage",
        "artifactRoot": "[parameters('binaryArtifactRoot')]",
        "authentication" :
        {
            "type": "SAS",
            "properties": {
                "sasUri": "[parameters('artifactSourceSASLocation')]"
            }
        }
    }
},
```

请注意 `artifactRoot` 和 `sasUri` 属性。 项目根目录可能设置为类似 `binaries/1.0.0.0` 的值。 SAS URI 是存储容器的 URI，带有用于访问的 SAS 令牌。 部署管理器自动构造 `$containerRoot` 变量的值。 它以 `<container>/<artifactRoot>` 格式将这些值组合在一起。

模板和参数文件需要了解获取已版本控制的二进制文件的正确路径。 例如，若要为 Web 应用部署文件，请创建以下具有 $containerRoot 变量的参数文件。 路径必须使用两个反斜杠 (`\\`)，因为第一个是转义字符。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "deployPackageUri": {
            "value": "$containerRoot\\helloWorldWebAppWUS.zip"
        }
    }
}
```

然后，在模板中使用该参数：

```json
{
    "name": "MSDeploy",
    "type": "extensions",
    "location": "[parameters('location')]",
    "apiVersion": "2015-08-01",
    "dependsOn": [
        "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
    ],
    "tags": {
        "displayName": "WebAppMSDeploy"
    },
    "properties": {
        "packageUri": "[parameters('deployPackageURI')]"
    }
}
```

可通过创建新文件夹并在推出期间传入该根目录来管理已版本控制的部署。 路径将流向部署资源的模板。

## <a name="next-steps"></a>后续步骤

本文介绍了有关部署管理器的信息。 请转到下一篇文章，了解如何使用部署管理器进行部署。

> [!div class="nextstepaction"]
> [教程：将 Azure 部署管理器与资源管理器模板配合使用](./deployment-manager-tutorial.md)