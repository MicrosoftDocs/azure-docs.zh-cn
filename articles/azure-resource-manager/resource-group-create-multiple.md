---
title: 部署 Azure 资源的多个实例 | Microsoft Docs
description: 在部署资源时使用 Azure Resource Manager 模板中的复制操作和数组执行多次迭代。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
editor: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/01/2019
ms.author: tomfitz
ms.openlocfilehash: 05b68fde30587967f65ee362344eea9a258f89a7
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205973"
---
# <a name="deploy-more-than-one-instance-of-a-resource-or-property-in-azure-resource-manager-templates"></a>在 Azure 资源管理器模板中部署资源或属性的多个实例

本文介绍了如何在 Azure 资源管理器模板中进行迭代操作，以创建多个资源实例。 如需指定究竟是否部署资源，请参阅 [condition 元素](resource-group-authoring-templates.md#condition)。

有关教程，请参阅[教程：使用资源管理器模板创建多个资源实例](./resource-manager-tutorial-create-multiple-instances.md)。


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-iteration"></a>资源迭代

当必须在部署过程中决定是创建资源的一个实例还是多个实例时，请将 `copy` 元素添加到资源类型。 在 copy 元素中，为此循环指定迭代次数和名称。 计数值必须是不超过 800 的正整数。 

要多次创建的资源采用以下格式：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": 3
      }
    }
  ],
  "outputs": {}
}
```

请注意，每个资源的名称都包括 `copyIndex()` 函数，用于返回循环中的当前迭代。 `copyIndex()` 从零开始。 因此，以下示例：

```json
"name": "[concat('storage', copyIndex())]",
```

将创建以下名称：

* storage0
* storage1
* storage2。

若要偏移索引值，可以在 copyIndex() 函数中传递一个值。 要执行的迭代次数仍被指定在 copy 元素中，但 copyIndex 的值已按指定的值发生了偏移。 因此，以下示例：

```json
"name": "[concat('storage', copyIndex(1))]",
```

将创建以下名称：

* storage1
* storage2
* storage3

处理数组时可以使用复制操作，因为可对数组中的每个元素执行迭代操作。 可以对数组使用 `length` 函数来指定迭代计数，并使用 `copyIndex` 来检索数组中的当前索引。 因此，以下示例：

```json
"parameters": { 
  "org": { 
    "type": "array", 
    "defaultValue": [ 
      "contoso", 
      "fabrikam", 
      "coho" 
    ] 
  }
}, 
"resources": [ 
  { 
    "name": "[concat('storage', parameters('org')[copyIndex()])]", 
    "copy": { 
      "name": "storagecopy", 
      "count": "[length(parameters('org'))]" 
    }, 
    ...
  } 
]
```

将创建以下名称：

* storagecontoso
* storagefabrikam
* storagecoho

默认情况下，资源管理器并行创建资源。 不会保证它们的创建顺序。 但是，你可能希望将资源指定为按顺序部署。 例如，在更新生产环境时，可能需要错开更新，使任何一次仅更新一定数量。

若要按顺序部署多个资源实例，请将 `mode` 设置为“串行”，并将 `batchSize` 设置为一次要部署的实例数量。 在串行模式下，资源管理器会在循环中创建早前实例的依赖项，以便在前一个批处理完成之前它不会启动一个批处理。

例如，若要按顺序一次部署两个存储帐户，请使用：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": 4,
        "mode": "serial",
        "batchSize": 2
      }
    }
  ],
  "outputs": {}
}
```

mode 属性也接受 **parallel**（它是默认值）。

有关复制中使用嵌套模板的信息，请参阅[使用副本](resource-group-linked-templates.md#using-copy)。

## <a name="property-iteration"></a>属性迭代

若要为资源上的属性创建多个值，请在属性元素中添加一个 `copy` 数组。 此数组包含对象，每个对象具有以下属性：

* 名称 - 要创建多个值的属性的名称
* 计数 - 要创建的值的数目。 计数值必须是不超过 800 的正整数。
* input - 一个对象，其中包含要赋给该属性的值  

以下示例演示如何将 `copy` 应用到虚拟机上的 dataDisks 属性：

```json
{
  "name": "examplevm",
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2017-03-30",
  "properties": {
    "storageProfile": {
      "copy": [{
        "name": "dataDisks",
        "count": 3,
        "input": {
          "lun": "[copyIndex('dataDisks')]",
          "createOption": "Empty",
          "diskSizeGB": "1023"
        }
      }],
      ...
```

请注意，在属性迭代中使用 `copyIndex` 时，必须提供迭代的名称。 与资源迭代一起使用时，无需提供名称。

Resource Manager 在部署期间会扩展 `copy` 数组。 该数组的名称将成为属性的名称。 输入值将成为对象属性。 已部署的模板将成为：

```json
{
  "name": "examplevm",
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2017-03-30",
  "properties": {
    "storageProfile": {
      "dataDisks": [
        {
          "lun": 0,
          "createOption": "Empty",
          "diskSizeGB": "1023"
        },
        {
          "lun": 1,
          "createOption": "Empty",
          "diskSizeGB": "1023"
        },
        {
          "lun": 2,
          "createOption": "Empty",
          "diskSizeGB": "1023"
        }
      ],
      ...
```

copy 元素是一个数组，因此，可以为资源指定多个属性。 为要创建的每个属性添加一个对象。

```json
{
  "name": "string",
  "type": "Microsoft.Network/loadBalancers",
  "apiVersion": "2017-10-01",
  "properties": {
    "copy": [
      {
        "name": "loadBalancingRules",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      },
      {
        "name": "probes",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      }
    ]
  }
}
```

可将资源迭代和属性迭代结合使用。 按名称引用属性迭代。

```json
{
  "type": "Microsoft.Network/virtualNetworks",
  "name": "[concat(parameters('vnetname'), copyIndex())]",
  "apiVersion": "2018-04-01",
  "copy":{
    "count": 2,
    "name": "vnetloop"
  },
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[parameters('addressPrefix')]"
      ]
    },
    "copy": [
      {
        "name": "subnets",
        "count": 2,
        "input": {
          "name": "[concat('subnet-', copyIndex('subnets'))]",
          "properties": {
            "addressPrefix": "[variables('subnetAddressPrefix')[copyIndex('subnets')]]"
          }
        }
      }
    ]
  }
}
```

## <a name="variable-iteration"></a>变量迭代

若要创建变量的多个实例，请在变量部分中使用 `copy` 属性。 可以创建一个由 `input` 属性中的值构造的元素数组。 可以在变量中使用 `copy` 属性，或在变量部分的顶层使用该属性。 在变量迭代中使用 `copyIndex` 时，必须提供迭代的名称。

如需通过简单示例来了解如何创建字符串值的数组，请参阅[复制数组模板](https://github.com/bmoore-msft/AzureRM-Samples/blob/master/copy-array/azuredeploy.json)。

以下示例演示了使用动态构造的元素创建数组变量的多种不同方法。 它演示如何在变量中使用副本来创建对象和字符串的数组。 它还演示如何在顶层使用副本来创建对象、字符串和整数的数组。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "disk-array-on-object": {
      "copy": [
        {
          "name": "disks",
          "count": 5,
          "input": {
            "name": "[concat('myDataDisk', copyIndex('disks', 1))]",
            "diskSizeGB": "1",
            "diskIndex": "[copyIndex('disks')]"
          }
        },
        {
          "name": "diskNames",
          "count": 5,
          "input": "[concat('myDataDisk', copyIndex('diskNames', 1))]"
        }
      ]
    },
    "copy": [
      {
        "name": "top-level-object-array",
        "count": 5,
        "input": {
          "name": "[concat('myDataDisk', copyIndex('top-level-object-array', 1))]",
          "diskSizeGB": "1",
          "diskIndex": "[copyIndex('top-level-object-array')]"
        }
      },
      {
        "name": "top-level-string-array",
        "count": 5,
        "input": "[concat('myDataDisk', copyIndex('top-level-string-array', 1))]"
      },
      {
        "name": "top-level-integer-array",
        "count": 5,
        "input": "[copyIndex('top-level-integer-array')]"
      }
    ]
  },
  "resources": [],
  "outputs": {
    "exampleObject": {
      "value": "[variables('disk-array-on-object')]",
      "type": "object"
    },
    "exampleArrayOnObject": {
      "value": "[variables('disk-array-on-object').disks]",
      "type" : "array"
    },
    "exampleObjectArray": {
      "value": "[variables('top-level-object-array')]",
      "type" : "array"
    },
    "exampleStringArray": {
      "value": "[variables('top-level-string-array')]",
      "type" : "array"
    },
    "exampleIntegerArray": {
      "value": "[variables('top-level-integer-array')]",
      "type" : "array"
    }
  }
}
```

创建的变量的类型取决于输入对象。 例如，在上一示例中名为 **top-level-object-array** 的变量返回：

```json
[
  {
    "name": "myDataDisk1",
    "diskSizeGB": "1",
    "diskIndex": 0
  },
  {
    "name": "myDataDisk2",
    "diskSizeGB": "1",
    "diskIndex": 1
  },
  {
    "name": "myDataDisk3",
    "diskSizeGB": "1",
    "diskIndex": 2
  },
  {
    "name": "myDataDisk4",
    "diskSizeGB": "1",
    "diskIndex": 3
  },
  {
    "name": "myDataDisk5",
    "diskSizeGB": "1",
    "diskIndex": 4
  }
]
```

名为 **top-level-string-array** 的变量返回：

```json
[
  "myDataDisk1",
  "myDataDisk2",
  "myDataDisk3",
  "myDataDisk4",
  "myDataDisk5"
]
```

## <a name="depend-on-resources-in-a-loop"></a>依赖于循环中的资源

可以使用 `dependsOn` 元素指定一个资源在另一个资源之后部署。 若要部署的资源依赖于循环中的资源集合，请在 dependsOn 元素中提供 copy 循环的名称。 以下示例演示了如何在部署虚拟机之前部署三个存储帐户。 此处并未显示完整的虚拟机定义。 请注意，copy 元素的名称设置为 `storagecopy`，而虚拟机的 dependsOn 元素也设置为 `storagecopy`。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": 3
      }
    },
    {
      "apiVersion": "2015-06-15", 
      "type": "Microsoft.Compute/virtualMachines", 
      "name": "[concat('VM', uniqueString(resourceGroup().id))]",  
      "dependsOn": ["storagecopy"],
      ...
    }
  ],
  "outputs": {}
}
```

<a id="looping-on-a-nested-resource" />

## <a name="iteration-for-a-child-resource"></a>子资源的迭代
不能对子资源使用 copy 循环。 要创建通常定义为嵌套在另一个资源中的资源的多个实例，必须将该资源创建为顶级资源。 可以通过 type 和 name 属性定义与父资源的关系。

例如，假设用户通常会将某个数据集定义为数据工厂中的子资源。

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
  "resources": [
    {
      "type": "datasets",
      "name": "exampleDataSet",
      "dependsOn": [
        "exampleDataFactory"
      ],
      ...
    }
  ]
```

若要创建多个数据集，请将其移出数据工厂。 数据集必须与数据工厂处于同一级别，但它仍是数据工厂的子资源。 可以通过 type 和 name 属性保留数据集和数据工厂之间的关系。 由于类型不再可以从其在模板中的位置推断，因此必须按以下格式提供完全限定的类型： `{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`。

若要与数据工厂的实例建立父/子关系，提供的数据集的名称应包含父资源名称。 使用以下格式： `{parent-resource-name}/{child-resource-name}`。  

以下示例演示实现过程：

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
},
{
  "type": "Microsoft.DataFactory/datafactories/datasets",
  "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
  "dependsOn": [
    "exampleDataFactory"
  ],
  "copy": {
    "name": "datasetcopy",
    "count": "3"
  },
  ...
}]
```

## <a name="example-templates"></a>示例模板

以下示例展示了创建资源或属性的多个实例的常见方案。

|模板  |描述  |
|---------|---------|
|[复制存储](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystorage.json) |部署名称中带索引号的多个存储帐户。 |
|[串行复制存储](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/serialcopystorage.json) |一次部署多个存储帐户。 名称包含索引号。 |
|[使用数组的复制存储](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystoragewitharray.json) |部署多个存储帐户。 名称包含数组中的值。 |
|[数据磁盘数可变的 VM 部署](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-windows-copy-datadisks) |通过虚拟机部署多个数据磁盘。 |
|[复制变量](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copyvariables.json) |演示循环访问变量的不同方式。 |
|[多项安全规则](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) |将多个安全规则部署到网络安全组。 它从参数构造安全规则。 有关参数，请参阅[多个 NSG 参数文件](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json)。 |

## <a name="next-steps"></a>后续步骤

* 要查看教程，请参阅[教程：使用资源管理器模板创建多个资源实例](./resource-manager-tutorial-create-multiple-instances.md)。

* 若要了解有关模板区段的信息，请参阅[创作 Azure Resource Manager 模板](resource-group-authoring-templates.md)。
* 若要了解如何部署模板，请参阅 [使用 Azure Resource Manager 模板部署应用程序](resource-group-template-deploy.md)。

