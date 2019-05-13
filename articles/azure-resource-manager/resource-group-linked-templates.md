---
title: 链接 Azure 部署的模板 | Microsoft 文档
description: 介绍如何使用 Azure Resource Manager 模板中的链接模板创建一个模块化的模板的解决方案。 演示如何传递参数值、指定参数文件和动态创建的 URL。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 27d8c4b2-1e24-45fe-88fd-8cf98a6bb2d2
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/01/2019
ms.author: tomfitz
ms.openlocfilehash: bbbaef306b9ed2bb415b29bc6d96dcfe649338f9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205941"
---
# <a name="using-linked-and-nested-templates-when-deploying-azure-resources"></a>部署 Azure 资源时使用链接模版和嵌套模版

若要部署解决方案，可以使用单个模板或包含任意相关模板的主模板。 相关模板可以是从主模板链接到的单独文件，也可以是嵌套在主模板中的模板。

对于中小型解决方案，单个模板更易于理解和维护。 可以查看单个文件中的所有资源和值。 对于高级方案，使用链接模板可将解决方案分解为目标组件，并重复使用模板。

使用链接模板时，需创建一个用于在部署期间接收参数值的主模板。 主模板包含所有链接模板，并根据需要将值传递给这些模板。

如需教程，请参阅[教程：创建链接的 Azure 资源管理器模板](./resource-manager-tutorial-create-linked-templates.md)。

> [!NOTE]
> 对于链接模板或嵌套模板，只能使用[增量](deployment-modes.md)部署模式。
>

## <a name="link-or-nest-a-template"></a>链接或嵌套模板

若要链接到另一个模板，请将一个**部署**资源添加到主模板。

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2018-05-01",
    "name": "linkedTemplate",
    "properties": {
        "mode": "Incremental",
        <nested-template-or-external-template>
    }
  }
]
```

为部署资源提供的属性将因要链接到外部模板，还是要将内联模板嵌套在主模板中而异。

### <a name="nested-template"></a>嵌套模板

若要将模板嵌套在主模板中，请使用 **template** 属性并指定模板语法。

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2018-05-01",
    "name": "nestedTemplate",
    "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[variables('storageName')]",
            "location": "West US",
            "kind": "StorageV2",
            "sku": {
                "name": "Standard_LRS"
            }
          }
        ]
      }
    }
  }
]
```

> [!NOTE]
> 对于嵌套模板，不能使用嵌套模板中定义的参数或变量。 可以使用主模板中的参数和变量。 在前面的示例中，`[variables('storageName')]` 从主模板（而不是嵌套模板）中检索值。 此限制不适用于外部模版。
>
> 如果在嵌套的模板内定义了两个资源并且一个资源依赖于另一个资源，则依赖项的值就是被依赖资源的名称：
> ```json
> "dependsOn": [
>   "[variables('storageAccountName')]"
> ],
> ```
>
> 不能在嵌套模板的 outputs 节中使用 `reference` 函数。 若要返回嵌套模板中部署的资源的值，请将嵌套模板转换为链接模板。

嵌套模板需要与标准模板[相同的属性](resource-group-authoring-templates.md)。

### <a name="external-template-and-external-parameters"></a>外部模板和外部参数

若要链接到外部模板和参数文件，请使用 **templateLink** 和 **parametersLink**。 链接到某个模板时，资源管理器服务必须能够访问该模板。 不能指定本地文件，或者只能在本地网络中使用的文件。 只能提供包含 **http** 或 **https** 的 URI 值。 一种做法是将链接模板放入存储帐户，并对该项使用 URI。

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2018-05-01",
    "name": "linkedTemplate",
    "properties": {
    "mode": "Incremental",
    "templateLink": {
        "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
        "contentVersion":"1.0.0.0"
    },
    "parametersLink": {
        "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
        "contentVersion":"1.0.0.0"
    }
    }
  }
]
```

无需为模板或参数提供 `contentVersion` 属性。 如果未提供内容版本值，将部署模板的当前版本。 如果提供内容版本值，它必须与链接的模板中的版本相匹配；否则，部署失败并产生错误。

### <a name="external-template-and-inline-parameters"></a>外部模板和内联参数

或者，可以提供内联参数。 不能同时使用内联参数和指向参数文件的链接。 同时指定 `parametersLink` 和 `parameters` 时，部署将失败，并出现错误。

若要将值从主模板传递给链接模板，请使用**参数**。

```json
"resources": [
  {
     "type": "Microsoft.Resources/deployments",
     "apiVersion": "2018-05-01",
     "name": "linkedTemplate",
     "properties": {
       "mode": "Incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parameters": {
          "StorageAccountName":{"value": "[parameters('StorageAccountName')]"}
        }
     }
  }
]
```

## <a name="using-copy"></a>使用副本

若要使用嵌套模板创建资源的多个实例，添加 copy 元素中的级别**microsoft.resources/deployments**资源。

下面的示例模板演示如何使用嵌套模板的副本。

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2018-05-01",
    "name": "[concat('nestedTemplate', copyIndex())]",
    // yes, copy works here
    "copy":{
      "name": "storagecopy",
      "count": 2
    },
    "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[concat(variables('storageName'), copyIndex())]",
            "location": "West US",
            "kind": "StorageV2",
            "sku": {
              "name": "Standard_LRS"
            }
            // no, copy doesn't work here
            //"copy":{
            //  "name": "storagecopy",
            //  "count": 2
            //}
          }
        ]
      }
    }
  }
]
```

## <a name="using-variables-to-link-templates"></a>使用变量来链接模板

前面的示例演示了用于模板链接的硬编码 URL 值。 这种方法可能适用于简单的模板，但如果使用一组大型模块化模板时，将无法正常工作。 相反，可以创建一个存储主模板的基 URL 的静态变量，并从基 URL 动态创建用于链接模板的 URL。 这种方法的好处是可以轻松地移动或派生模板，因为只需在主模板中更改静态变量。 主模板会在整个分解后的模板中传递正确的 URI。

以下示例演示如何使用基 URL 来创建两个用于链接模板的 URL（**sharedTemplateUrl** 和 **vmTemplate**）。

```json
"variables": {
    "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
    "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
    "vmTemplateUrl": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

还可以使用 [deployment()](resource-group-template-functions-deployment.md#deployment) 获取当前模板的基 URL，并使用该 URL 来获取同一位置其他模板的 URL。 如果模板位置发生变化或者想要避免对模板文件中的 URL 进行硬编码，则此方法非常有用。 仅当链接到带有 URL 的远程模板时，才会返回 templateLink 属性。 如果使用的是本地模板，该属性不可用。

```json
"variables": {
    "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

## <a name="get-values-from-linked-template"></a>从链接模板中获取值

若要从链接模板中获取输出值，请使用如下所示的语法检索属性值：`"[reference('deploymentName').outputs.propertyName.value]"`。

从链接模板获取输出属性时，属性名称不能包含短划线。

以下示例演示如何引用链接模板和检索输出值。 链接模板返回一条简单的消息。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {
        "greetingMessage": {
            "value": "Hello World",
            "type" : "string"
        }
    }
}
```

主模板部署链接模板并获取返回值。 请注意，该模板按名称引用部署资源，并使用链接模板返回的属性的名称。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "linkedTemplate",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'helloworld.json')]",
                    "contentVersion": "1.0.0.0"
                }
            }
        }
    ],
    "outputs": {
        "messageFromLinkedTemplate": {
            "type": "string",
            "value": "[reference('linkedTemplate').outputs.greetingMessage.value]"
        }
    }
}
```

链接模板与其他资源类型相似，也可在它与其他资源之间设置依赖关系。 因此，当其他资源需要链接模板的输出值时，请确保在部署这些资源之前部署链接模板。 或者，当链接模板依赖于其他资源时，请确保在部署链接模板之前部署其他资源。

以下示例演示一个部署公共 IP 地址并返回资源 ID 的模板：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2018-11-01",
            "name": "[parameters('publicIPAddresses_name')]",
            "location": "eastus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "resourceID": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
        }
    }
}
```

在部署负载均衡器时，若要使用前面所述模板中的公共 IP 地址，请链接到该模板，并添加与部署资源之间的依赖关系。 负载均衡器上的公共 IP 地址设置为链接模板的输出值。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "loadBalancers_name": {
            "defaultValue": "mylb",
            "type": "string"
        },
        "publicIPAddresses_name": {
            "defaultValue": "myip",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2018-11-01",
            "name": "[parameters('loadBalancers_name')]",
            "location": "eastus",
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "LoadBalancerFrontEnd",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[reference('linkedTemplate').outputs.resourceID.value]"
                            }
                        }
                    }
                ],
                "backendAddressPools": [],
                "loadBalancingRules": [],
                "probes": [],
                "inboundNatRules": [],
                "outboundNatRules": [],
                "inboundNatPools": []
            },
            "dependsOn": [
                "linkedTemplate"
            ]
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "linkedTemplate",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'publicip.json')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters":{
                    "publicIPAddresses_name":{"value": "[parameters('publicIPAddresses_name')]"}
                }
            }
        }
    ]
}
```

## <a name="linked-and-nested-templates-in-deployment-history"></a>部署历史记录中的链接模板和嵌套模板

资源管理器将每个模板作为部署历史记录中的单独部署进行处理。 因此，包含三个链接模板或嵌套模板的主模板在部署历史记录中显示为：

![部署历史记录](./media/resource-group-linked-templates/deployment-history.png)

部署后，可以使用历史记录中这些不同的条目来检索输出值。 以下模板创建一个公共 IP 地址并输出该 IP 地址：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2018-11-01",
            "name": "[parameters('publicIPAddresses_name')]",
            "location": "southcentralus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4,
                "dnsSettings": {
                    "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
                }
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "returnedIPAddress": {
            "type": "string",
            "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
        }
    }
}
```

以下模板链接到前面所述的模板。 它创建三个公共 IP 地址。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "[concat('linkedTemplate', copyIndex())]",
            "copy": {
                "count": 3,
                "name": "ip-loop"
            },
            "properties": {
              "mode": "Incremental",
              "templateLink": {
                "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
                "contentVersion": "1.0.0.0"
              },
              "parameters":{
                  "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
              }
            }
        }
    ]
}
```

部署后，可使用以下 PowerShell 脚本检索输出值：

```azurepowershell-interactive
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
    $name = 'linkedTemplate' + $i;
    $deployment = Get-AzResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
    Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

还可以使用 Bash shell 中的 Azure CLI 脚本：

```azurecli-interactive
#!/bin/bash

for i in 0 1 2;
do
    name="linkedTemplate$i";
    deployment=$(az group deployment show -g examplegroup -n $name);
    ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
    echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>保护外部模板

尽管链接模板必须可从外部使用，但它无需向公众正式发布。 可以将模板添加到只有存储帐户所有者可以访问的专用存储帐户。 然后，在部署期间创建共享访问签名 (SAS) 令牌来启用访问。 将该 SAS 令牌添加到链接模板的 URI。 即使令牌作为安全字符串传入，链接模板的 URI（包括 SAS 令牌）也会记录在部署操作中。 若要限制公开，请设置令牌的到期时间。

也可将参数文件限制为通过 SAS 令牌进行访问。

以下示例演示在链接到模板时如何传递 SAS 令牌：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerSasToken": { "type": "string" }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

在 PowerShell 中，使用以下命令获取容器的令牌并部署模板。 注意，**containerSasToken** 参数是在模板中定义的。 它不是 **New-AzResourceGroupDeployment** 命令中的参数。

```azurepowershell-interactive
Set-AzCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

对于 Bash Shell 中的 Azure CLI，使用以下代码获取容器的令牌并部署模板：

```azurecli-interactive
#!/bin/bash

expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name storagecontosotemplates \
    --query connectionString)
token=$(az storage container generate-sas \
    --name templates \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name parent.json \
    --output tsv \
    --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az group deployment create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>示例模板

下面的示例展示了链接的模板的常见用途。

|主模板  |链接的模板 |描述  |
|---------|---------| ---------|
|[Hello World](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json) |[链接的模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json) | 从链接的模板返回字符串。 |
|[使用公共 IP 地址的负载均衡器](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) |[链接的模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) |从链接的模板返回公共 IP 地址并在负载均衡器中设置该值。 |
|[多个 IP 地址](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json) | [链接的模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json) |在链接模板中创建多个公共 IP 地址。  |

## <a name="next-steps"></a>后续步骤

* 若要浏览教程，请参阅[教程：创建链接的 Azure 资源管理器模板](./resource-manager-tutorial-create-linked-templates.md)。
* 若要了解如何为资源定义部署顺序，请参阅[在 Azure 资源管理器模板中定义依赖关系](resource-group-define-dependencies.md)。
* 若要了解如何定义一个资源而创建多个实例，请参阅[在 Azure 资源管理器中创建多个资源实例](resource-group-create-multiple.md)。
* 有关在存储帐户中设置模板和生成 SAS 令牌的步骤，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](resource-group-template-deploy.md)或[使用 Resource Manager 模板和 Azure CLI 部署资源](resource-group-template-deploy-cli.md)。
