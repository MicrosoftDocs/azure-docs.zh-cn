---
title: 使用 Azure 资源管理器模板设置设备预配 | Microsoft Docs
description: Azure 快速入门 - 使用模板设置 Azure IoT 中心设备预配服务
author: wesmc7777
ms.author: wesmc
ms.date: 06/18/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 3360bfa7eed15f72fb78f698e837d887e9c8aa85
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62126471"
---
# <a name="set-up-the-iot-hub-device-provisioning-service-with-an-azure-resource-manager-template"></a>使用 Azure 资源管理器模板设置 IoT 中心设备预配服务

可以使用 [Azure 资源管理器](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)以编程方式设置预配设备所需的 Azure 云资源。 这些步骤演示了如何创建 IoT 中心和新的 IoT 中心设备预配服务，然后使用 Azure 资源管理器模板将这两项服务链接到一起。 本快速入门使用 [Azure CLI](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli) 执行创建资源组和部署模板所需的编程步骤，但你也可以使用 [Azure 门户](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal)、[PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)、.NET、Ruby 或其他编程语言，轻松执行这些步骤和部署模板。 


## <a name="prerequisites"></a>先决条件

- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
- 本快速入门要求在本地运行 Azure CLI。 必须安装 Azure CLI 2.0 或更高版本。 运行 `az --version` 即可查找版本。 如果需要安装或升级 CLI，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。


## <a name="sign-in-to-azure-and-create-a-resource-group"></a>登录到 Azure 并创建资源组

登录到 Azure 帐户，并选择订阅。

1. 在命令提示符中，运行 [login 命令][lnk-login-command]：
    
    ```azurecli
    az login
    ```

    按照说明使用代码进行身份验证，并通过 Web 浏览器登录 Azure 帐户。

2. 如果有多个 Azure 订阅，登录 Azure 可获得与凭据关联的所有 Azure 帐户的访问权限。 使用以下[命令，列出可供使用的 Azure 帐户][lnk-az-account-command]：
    
    ```azurecli
    az account list 
    ```

    使用以下命令，选择想要用于运行命令以创建 IoT 中心的订阅。 可使用上一命令输出中的订阅名称或 ID：

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

3. 创建 Azure 云资源（例如 IoT 中心和预配服务）时，请在资源组中创建。 使用现有资源组，或运行以下[命令创建资源组][lnk-az-resource-command]：
    
    ```azurecli
     az group create --name {your resource group name} --location westus
    ```

    > [!TIP]
    > 上一示例在美国西部位置创建资源组。 可运行 `az account list-locations -o table` 命令，查看可用位置的列表。
    >
    >

## <a name="create-a-resource-manager-template"></a>创建资源管理器模板

请使用 JSON 模板在资源组中创建预配服务和链接的 IoT 中心。 还可以使用 Azure 资源管理器模板更改现有的预配服务或 IoT 中心。

1. 使用文本编辑器创建名为 **template.json** 的 Azure 资源管理器模板，其中包含以下框架内容。 

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {},
       "variables": {},
       "resources": []
   }
   ```

2. 将 **parameters** 节替换为以下内容。 parameters 节指定可以从另一文件传入的参数。 此节指定要创建的 IoT 中心和预配服务的名称， 以及 IoT 中心和预配服务的位置。 这些值限于支持 IoT 中心和预配服务的 Azure 区域。 如需设备预配服务支持的位置的列表，可以运行以下命令：`az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table`，或者转到 [Azure 状态](https://azure.microsoft.com/status/)页，然后搜索“设备预配服务”。

   ```json
    "parameters": {
        "iotHubName": {
            "type": "string"
        },
        "provisioningServiceName": {
            "type": "string"
        },
        "hubLocation": {
            "type": "string",
            "allowedValues": [
                "eastus",
                "westus",
                "westeurope",
                "northeurope",
                "southeastasia",
                "eastasia"
            ]
        }
    },

   ```

3. 将 **variables** 节替换为以下内容。 此节定义的值可以在以后用来构造 IoT 中心连接字符串，该字符串是链接预配服务和 IoT 中心所必需的。 
 
   ```json
    "variables": {        
        "iotHubResourceId": "[resourceId('Microsoft.Devices/Iothubs', parameters('iotHubName'))]",
        "iotHubKeyName": "iothubowner",
        "iotHubKeyResource": "[resourceId('Microsoft.Devices/Iothubs/Iothubkeys', parameters('iotHubName'), variables('iotHubKeyName'))]"
    },

   ```

4. 若要创建 IoT 中心，请将以下行添加到 **resources** 集合。 该 JSON 指定创建 IoT 中心所需的最少属性。 **name** 和 **location** 属性作为参数传递。 若要详细了解可以在模板中为 IoT 中心指定的属性，请参阅 [Microsoft.Devices/IotHubs template reference](https://docs.microsoft.com/azure/templates/microsoft.devices/iothubs)（Microsoft.Devices/IotHubs 模板参考）。

   ```json
        {
            "apiVersion": "2017-07-01",
            "type": "Microsoft.Devices/IotHubs",
            "name": "[parameters('iotHubName')]",
            "location": "[parameters('hubLocation')]",
            "sku": {
                "name": "S1",
                "capacity": 1
            },
            "tags": {
            },
            "properties": {
            }            
        },

   ``` 

5. 若要创建预配服务，请将以下行添加到 **resources** 集合中的 IoT 中心规格后面。 预配服务的 **name** 和 **location** 以参数方式传递。 在 **iotHubs** 集合中指定要链接到预配服务的 IoT 中心。 至少必须指定每个链接的 IoT 中心的 **connectionString** 和 **location** 属性。 也可在每个 IoT 中心设置 **allocationWeight** 和 **applyAllocationPolicy** 之类的属性，以及在预配服务中设置 **allocationPolicy** 和 **authorizationPolicies** 之类的属性。 若要进行详细了解，请参阅 [Microsoft.Devices/provisioningServices template reference](https://docs.microsoft.com/azure/templates/microsoft.devices/provisioningservices)（Microsoft.Devices/provisioningServices 模板参考）。

   使用 **dependsOn** 属性是为了确保资源管理器先创建 IoT 中心，然后再创建预配服务。 模板要求 IoT 中心的连接字符串指定其到预配服务的链接，因此必须先创建该中心及其密钥。 模板使用 **concat** 和 **listKeys** 之类的函数来创建连接字符串。 若要进行详细了解，请参阅 [Azure 资源管理器模板函数](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-functions)。

   ```json
        {
            "type": "Microsoft.Devices/provisioningServices",
            "sku": {
                "name": "S1",
                "capacity": 1
            },
            "name": "[parameters('provisioningServiceName')]",
            "apiVersion": "2017-11-15",
            "location": "[parameters('hubLocation')]",
            "tags": {},
            "properties": {
                "iotHubs": [
                    {
                        "connectionString": "[concat('HostName=', reference(variables('iotHubResourceId')).hostName, ';SharedAccessKeyName=', variables('iotHubKeyName'), ';SharedAccessKey=', listkeys(variables('iotHubKeyResource'), '2017-07-01').primaryKey)]",
                        "location": "[parameters('hubLocation')]",
                        "name": "[concat(parameters('iotHubName'),'.azure-devices.net')]"
                    }
                ]
            },
            "dependsOn": ["[parameters('iotHubName')]"]
        }

   ```

6. 保存模板文件。 完成的模板应如下所示：

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
           "iotHubName": {
               "type": "string"
           },
           "provisioningServiceName": {
               "type": "string"
           },
           "hubLocation": {
               "type": "string",
               "allowedValues": [
                   "eastus",
                   "westus",
                   "westeurope",
                   "northeurope",
                   "southeastasia",
                   "eastasia"
               ]
           }
       },
       "variables": {        
           "iotHubResourceId": "[resourceId('Microsoft.Devices/Iothubs', parameters('iotHubName'))]",
           "iotHubKeyName": "iothubowner",
           "iotHubKeyResource": "[resourceId('Microsoft.Devices/Iothubs/Iothubkeys', parameters('iotHubName'), variables('iotHubKeyName'))]"
       },
       "resources": [
           {
               "apiVersion": "2017-07-01",
               "type": "Microsoft.Devices/IotHubs",
               "name": "[parameters('iotHubName')]",
               "location": "[parameters('hubLocation')]",
               "sku": {
                   "name": "S1",
                   "capacity": 1
               },
               "tags": {
               },
               "properties": {
               }            
           },
           {
               "type": "Microsoft.Devices/provisioningServices",
               "sku": {
                   "name": "S1",
                   "capacity": 1
               },
               "name": "[parameters('provisioningServiceName')]",
               "apiVersion": "2017-11-15",
               "location": "[parameters('hubLocation')]",
               "tags": {},
               "properties": {
                   "iotHubs": [
                       {
                           "connectionString": "[concat('HostName=', reference(variables('iotHubResourceId')).hostName, ';SharedAccessKeyName=', variables('iotHubKeyName'), ';SharedAccessKey=', listkeys(variables('iotHubKeyResource'), '2017-07-01').primaryKey)]",
                           "location": "[parameters('hubLocation')]",
                           "name": "[concat(parameters('iotHubName'),'.azure-devices.net')]"
                       }
                   ]
               },
               "dependsOn": ["[parameters('iotHubName')]"]
           }
       ]
   }
   ```

## <a name="create-a-resource-manager-parameter-file"></a>创建资源管理器参数文件

在上一步定义的模板使用参数来指定 IoT 中心的名称、预配服务的名称，以及用于创建它们的位置（Azure 区域）。 在单独的文件中传递这些参数。 这样做可以重复使用同一模板进行多次部署。 若要创建参数文件，请执行以下步骤：

1. 使用文本编辑器创建名为 **parameters.json** 的 Azure 资源管理器参数文件，其中包含以下框架内容： 

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {}
       }
   }
   ```

2. 将 **iotHubName** 值添加到 parameter 节。 如果更改名称，请确保遵循适用于 IoT 中心的命名约定。 名称应该 3-50 个字符长，只能包含大写或小写字母数字字符或连字符（“-”）。 

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
    }
   
   ```

3. 将 **provisioningServiceName** 值添加到 parameter 节。 如果更改名称，请确保遵循适用于 IoT 中心设备预配服务的命名约定。 名称应该 3-64 个字符长，只能包含大写或小写字母数字字符或连字符（“-”）。

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
        "provisioningServiceName": {
            "value": "my-sample-provisioning-service"
        },
    }

   ```

4. 将 **hubLocation** 值添加到 parameter 节。 此值指定 IoT 中心和预配服务的位置。 该值必须与在模板文件的参数定义的 **allowedValues** 集合中指定的某个位置相符。 此集合将这些值限制为支持 IoT 中心和预配服务的 Azure 位置。 如需设备预配服务支持的位置的列表，可以运行以下命令：`az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table`，或者转到 [Azure 状态](https://azure.microsoft.com/status/)页，然后搜索“设备预配服务”。

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
        "provisioningServiceName": {
            "value": "my-sample-provisioning-service"
        },
        "hubLocation": {
            "value": "westus"
        }
    }

   ```

5. 保存文件。 


> [!IMPORTANT]
> IoT 中心和预配服务都将公开为 DNS 终结点，因此，对其命名时请务必避免包含任何敏感信息。
>

## <a name="deploy-the-template"></a>部署模板

请使用以下 Azure CLI 命令来部署模板和验证部署。

1. 若要部署模板，请运行下述[用于启动部署的命令](https://docs.microsoft.com/cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create)：
    
    ```azurecli
     az group deployment create -g {your resource group name} --template-file template.json --parameters @parameters.json
    ```

   在输出中查找设置为“Succeeded”的 **provisioningState** 属性。 

   ![预配输出](./media/quick-setup-auto-provision-rm/output.png) 


2. 若要验证部署，请运行下述[用于列出资源的命令](https://docs.microsoft.com/cli/azure/resource?view=azure-cli-latest#az-resource-list)，然后在输出中查找新预配服务和 IoT 中心：

    ```azurecli
     az resource list -g {your resource group name}
    ```


## <a name="clean-up-resources"></a>清理资源

本教程系列中的其他快速入门教程是在本文的基础上制作的。 如果打算继续学习后续的快速入门或相关教程，请不要清理在本快速入门中创建的资源。 如果不打算继续，可以使用 Azure CLI [删除单个资源][lnk-az-resource-command]（例如 IoT 中心或预配服务），或者删除资源组及其所有资源。

若要删除预配服务，请运行以下命令：

```azurecli
az iot hub delete --name {your provisioning service name} --resource-group {your resource group name}
```
若要删除 IoT 中心，请运行以下命令：

```azurecli
az iot hub delete --name {your iot hub name} --resource-group {your resource group name}
```

若要删除资源组及其所有资源，请运行以下命令：

```azurecli
az group delete --name {your resource group name}
```

也可以使用 Azure 门户、PowerShell 或 REST API，或者针对 Azure 资源管理器或 IoT 中心设备预配服务发布的受支持平台 SDK，来删除资源组和单个资源。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已部署 IoT 中心和设备预配服务实例，并链接了这两个资源。 若要了解如何使用此设置来预配模拟设备，请继续学习本快速入门中关于如何创建模拟设备的内容。

> [!div class="nextstepaction"]
> [创建模拟设备快速入门](./quick-create-simulated-device.md)


<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-CLI-install]: https://docs.microsoft.com/cli/azure/install-az-cli2
[lnk-login-command]: https://docs.microsoft.com/cli/azure/get-started-with-az-cli2
[lnk-az-account-command]: https://docs.microsoft.com/cli/azure/account
[lnk-az-register-command]: https://docs.microsoft.com/cli/azure/provider
[lnk-az-addcomponent-command]: https://docs.microsoft.com/cli/azure/component
[lnk-az-resource-command]: https://docs.microsoft.com/cli/azure/resource
[lnk-az-iot-command]: https://docs.microsoft.com/cli/azure/iot
[lnk-iot-pricing]: https://azure.microsoft.com/pricing/details/iot-hub/
[lnk-devguide]: iot-hub-devguide.md
[lnk-portal]: iot-hub-create-through-portal.md 
