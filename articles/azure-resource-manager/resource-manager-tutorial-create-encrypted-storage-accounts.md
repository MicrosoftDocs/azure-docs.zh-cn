---
title: 利用 Azure 资源管理器模板参考 | Microsoft Docs
description: 利用 Azure 资源管理器模板参考创建用于部署加密存储帐户的模板。
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 03/04/2019
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: 81c02c5e39e23b49291561821b0732227feb5c05
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62103610"
---
# <a name="tutorial-utilize-the-azure-resource-manager-template-reference"></a>教程：利用 Azure 资源管理器模板参考

了解如何查找模板架构信息，以及如何使用该信息创建 Azure 资源管理器模板。

在本教程中，请使用 Azure 快速入门模板中提供的基础模板。 根据模板参考文档自定义模板，以便创建加密的存储帐户。

![资源管理器模板参考部署加密的存储帐户](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-tutorial-deploy-encrypted-storage-account.png)

本教程涵盖以下任务：

> [!div class="checklist"]
> * 打开快速入门模板
> * 了解模板
> * 查找模板参考
> * 编辑模板
> * 部署模板

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含[资源管理器工具扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)的 [Visual Studio Code](https://code.visualstudio.com/)。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

[Azure 快速入门模板](https://azure.microsoft.com/resources/templates/)是资源管理器模板的存储库。 无需从头开始创建模板，只需找到一个示例模板并对其自定义即可。 本快速入门中使用的模板称为[创建标准存储帐户](https://azure.microsoft.com/resources/templates/101-storage-account-create/)。 该模板定义 Azure 存储帐户资源。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。
2. 在“文件名”中粘贴以下 URL：

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。
4. 选择“文件”>“另存为”，将该文件作为 **azuredeploy.json** 保存到本地计算机。

## <a name="understand-the-schema"></a>了解架构

1. 在 VS Code 中将模板折叠到根级别。 你使用最简单的结构，其中包含以下元素：

    ![资源管理器模板的最简单结构](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-simplest-structure.png)

    * **$schema**：指定描述模板语言版本的 JSON 架构文件所在的位置。
    * **contentVersion**：为此元素指定任意值，以便记录模板中的重要更改。
    * **parameters**：指定执行部署以自定义资源部署时提供的值。
    * **variables**：指定在模板中用作 JSON 片段以简化模板语言表达式的值。
    * **resources**：指定已在资源组中部署或更新的资源类型。
    * **outputs**：指定部署后返回的值。

2. 展开“resources”。 已定义 `Microsoft.Storage/storageAccounts` 资源。 此模板创建非加密存储帐户。

    ![资源管理器模板存储帐户定义](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-encrypted-storage-resource.png)

## <a name="find-the-template-reference"></a>查找模板参考

1. 浏览到 [Azure 模板参考](https://docs.microsoft.com/azure/templates/)。
2. 在“按标题筛选”框中，输入“存储帐户”。
3. 选择“参考/模板参考/存储/&lt;版本>/存储帐户”，如以下屏幕截图中所示：

    ![资源管理器模板参考存储帐户](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-resources-reference-storage-accounts.png)

    如果不知道选择哪个版本，请使用最新版本。

4. 查找与加密相关的定义信息。  

    ```json
    "encryption": {
      "services": {
        "blob": {
          "enabled": boolean
        },
        "file": {
          "enabled": boolean
        }
      },
      "keySource": "string",
      "keyvaultproperties": {
        "keyname": "string",
        "keyversion": "string",
        "keyvaulturi": "string"
      }
    },
    ```

    在同一网页上，以下说明确认 `encryption` 对象用于创建加密的存储帐户。

    ![资源管理器模板参考存储帐户加密](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-resources-reference-storage-accounts-encryption.png)

    可以通过两种方式来管理加密密钥。 可以将 Microsoft 托管的加密密钥用于存储服务加密，或者使用你自己的加密密钥。 为了简化本教程，请使用 `Microsoft.Storage` 选项，这样就不需创建 Azure Key Vault。

    ![资源管理器模板参考存储帐户加密对象](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-resources-reference-storage-accounts-encryption-object.png)

    加密对象应如下所示：

    ```json
    "encryption": {
        "services": {
            "blob": {
                "enabled": true
            },
            "file": {
              "enabled": true
            }
        },
        "keySource": "Microsoft.Storage"
    }
    ```

## <a name="edit-the-template"></a>编辑模板

在 Visual Studio Code 中修改模板，使 resources 元素如下所示：

![资源管理器模板加密的存储帐户资源](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-encrypted-storage-resources.png)

## <a name="deploy-the-template"></a>部署模板

有关部署过程，请参阅 Visual Studio Code 快速入门中的[部署模板](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#deploy-the-template)部分。

以下屏幕快照显示的 CLI 命令用于列出新创建的存储帐户，该命令指示已为 Blob 存储启用加密。

![Azure 资源管理器加密的存储帐户](./media/resource-manager-tutorial-create-encrypted-storage-accounts/resource-manager-template-encrypted-storage-account.png)

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。  应会看到，该资源组中总共有六个资源。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何使用模板参考来自定义现有的模板。 若要了解如何创建多个存储帐户实例，请参阅：

> [!div class="nextstepaction"]
> [创建多个实例](./resource-manager-tutorial-create-multiple-instances.md)
