---
title: 排查资源管理器部署问题 | Microsoft Docs
description: 了解如何监视和排查资源管理器部署问题。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 01/15/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: c889c3123160680d96889227d6964ff197dc41cc
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60388592"
---
# <a name="tutorial-troubleshoot-resource-manager-template-deployments"></a>教程：排查资源管理器模板部署问题

了解如何排查资源管理器模板部署错误。 在本教程中，请先在模板中设置两个错误，然后了解如何使用活动日志和部署历史记录来解决问题。

有两种类型的错误与模板部署相关：

- **验证错误**源于部署之前可确定的方案。 原因包括模板中的语法错误，或尝试部署超出订阅配额的资源。 
- **部署错误**源于部署过程中发生的条件。 原因包括尝试访问并行部署的资源。

两种类型的错误都会返回用于对部署进行故障排除的错误代码。 两种类型的错误都会显示在活动日志中。 但是，验证错误不会显示在部署历史记录中，因为部署从未启动。

本教程涵盖以下任务：

> [!div class="checklist"]
> * 创建有问题的模板
> * 排查验证错误
> * 排查部署错误
> * 清理资源

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

- 包含[资源管理器工具扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)的 [Visual Studio Code](https://code.visualstudio.com/)。

## <a name="create-a-problematic-template"></a>创建有问题的模板

打开 [Azure 快速入门模板](https://azure.microsoft.com/resources/templates/)中名为[创建标准存储帐户](https://azure.microsoft.com/resources/templates/101-storage-account-create/)的模板，设置两个模板问题。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。
4. 将 **apiVersion** 行更改为以下行：

    ```json
    "apiVersion1": "2018-07-02",
    ```
    - **apiVersion1** 是无效的元素名称。 它是验证错误。
    - API 版本应该是“2018-07-01”。  它是部署错误。

5. 选择“文件”>“另存为”，将该文件作为 **azuredeploy.json** 保存到本地计算机。

## <a name="troubleshoot-the-validation-error"></a>排查验证错误

请参阅[部署模板](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#deploy-the-template)部分，了解如何部署模板。

你会从 shell 获得类似于以下内容的错误：

```
New-AzResourceGroupDeployment : 4:29:24 PM - Error: Code=InvalidRequestContent; Message=The request content was invalid and could not be deserialized: 'Could not find member 'apiVersion1' on object of type 'TemplateResource'. Path 'properties.template.resources[0].apiVersion1', line 36, position 24.'.
```

错误消息指示问题出在 **apiVersion1**。

使用 Visual Studio Code 修正此问题，方法是将 **apiVersion1** 更改为 **apiVersion**，然后保存模板。

## <a name="troubleshoot-the-deployment-error"></a>排查部署错误

请参阅[部署模板](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#deploy-the-template)部分，了解如何部署模板。

你会从 shell 获得类似于以下内容的错误：

```
New-AzResourceGroupDeployment : 4:48:50 PM - Resource Microsoft.Storage/storageAccounts 'storeqii7x2rce77dc' failed with message '{
  "error": {
    "code": "NoRegisteredProviderFound",
    "message": "No registered resource provider found for location 'centralus' and API version '2018-07-02' for type 'storageAccounts'. The supported api-versions are '2018-07-01, 2018-03-01-preview, 2018-02-01, 2017-10-01, 2017-06-01, 2016-12-01, 2016-05-01, 2016-01-01, 2015-06-15, 2015-05-01-preview'. The supported locations are 'eastus, eastus2, westus, westeurope, eastasia, southeastasia, japaneast, japanwest, northcentralus, southcentralus, centralus, northeurope, brazilsouth, australiaeast, australiasoutheast, southindia, centralindia, westindia, canadaeast, canadacentral, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth, francecentral'."
  }
}'
```

可通过以下过程从 Azure 门户找出部署错误：

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 打开资源组，方法是：选择“资源组”，然后选择资源组名称。 此时会看到“部署”下显示“1 个失败”。

    ![资源管理器教程故障排除](./media/resource-manager-tutorial-troubleshoot/resource-manager-template-deployment-error.png)
3. 选择“错误详细信息”。

    ![资源管理器教程故障排除](./media/resource-manager-tutorial-troubleshoot/resource-manager-template-deployment-error-details.png)

    错误消息与此前显示的相同：

    ![资源管理器教程故障排除](./media/resource-manager-tutorial-troubleshoot/resource-manager-template-deployment-error-summary.png)

也可在活动日志中查找错误：

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 选择“Monitor” > “活动日志”。
3. 使用筛选器查找日志。

    ![资源管理器教程故障排除](./media/resource-manager-tutorial-troubleshoot/resource-manager-template-deployment-activity-log.png)

使用 Visual Studio Code 修正此问题，然后重新部署模板。

如需常见错误的列表，请参阅[排查使用 Azure 资源管理器时的常见 Azure 部署错误](./resource-manager-common-deployment-errors.md)。

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。  应会看到，该资源组中总共有六个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何排查资源管理器模板部署错误。  有关详细信息，请参阅[排查使用 Azure 资源管理器时的常见 Azure 部署错误](./resource-manager-common-deployment-errors.md)。
