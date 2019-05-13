---
title: 使用 Azure 门户导出 Azure 资源管理器模板
description: 使用 Azure 门户从你的订阅中的资源导出 Azure 资源管理器模板。
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 05/09/2019
ms.author: tomfitz
ms.openlocfilehash: ea9499da3dac67635a48704f439f6592c6ed467e
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65515379"
---
# <a name="single-and-multi-resource-export-to-template-in-azure-portal"></a>单个和多资源导出到 Azure 门户中的模板

若要帮助创建 Azure 资源管理器模板，可以从现有资源导出模板。 导出的模板可帮助您了解的 JSON 语法和部署资源的属性。 若要自动执行以后进行部署时，导出的模板开始，并修改为你的方案。

资源管理器可以选择导出到模板的一个或多个资源。 你可以专注于完全在模板中所需的资源。

## <a name="choose-the-right-export-option"></a>选择正确的导出选项

可以通过两种方式来导出模板：

* **从资源组或资源导出**。 此选项可从现有资源生成一个新的模板。 导出的模板是资源组的当前状态的"快照"。 您可以导出整个资源组或该资源组中的特定资源。

* **在部署之前或从历史记录中导出**。 此选项检索用于部署模板的一个精确副本。

根据你选择的选项，导出的模板具有不同的质量。

| 从资源组或资源 | 在部署之前或从历史记录 |
| --------------------- | ----------------- |
| 快照模板的资源的当前状态。 它包括部署后进行任何手动更改。 | 模板在部署仅显示资源的状态。 不包括在内，部署后所做的任何手动更改。 |
| 从要导出的资源组，可以选择哪些资源。 | 中包含的特定部署的所有资源都。 无法选择这些资源的一个子集，或添加已添加在不同时间的资源。 |
| 模板包括的资源，包括通常不会在部署期间设置某些属性的所有属性。 您可能想要删除或重复使用该模板之前清理这些属性。 | 模板包括部署所需的属性。 该模板是随时可用。 |
| 模板可能不包括所有所需的重复使用的参数。 大多数属性值是在模板中硬编码。 若要重新部署在其他环境中的模板，需要添加增加配置的资源的功能的参数。 | 模板包括轻松地重新部署在不同环境中的参数。 |

从资源组或资源，导出模板时：

* 您需要捕获资源的原始部署之后所做的更改。
* 你想要选择导出的资源。

导出部署前或从历史记录，该模板时：

* 您希望轻松重用模板。
* 不需要包含在原始部署后所做的更改。

## <a name="export-template-from-resource-group"></a>从资源组导出模板

若要从资源组导出一个或多个资源：

1. 选择包含你想要导出的资源的资源组。

1. 若要导出的资源组中的所有资源，请选择所有，然后**都导出模板**。 **导出模板**选项只会启用后选择至少一个资源。

   ![导出所有资源](./media/export-template-portal/select-all-resources.png)

1. 若要选择用于导出的特定资源，请选择这些资源旁边的复选框。 然后，选择**导出模板**。

   ![选择要导出的资源](./media/export-template-portal/select-resources.png)

1. 导出的模板显示，并且可供下载。

   ![显示模板](./media/export-template-portal/show-template.png)

## <a name="export-template-from-resource"></a>从资源导出模板

若要导出一个资源：

1. 选择包含你想要导出的资源的资源组。

1. 选择要导出的资源。

   ![选择资源](./media/export-template-portal/select-link-resource.png)

1. 该资源中，选择**导出模板**的左窗格中。

   ![导出资源](./media/export-template-portal/export-single-resource.png)

1. 导出的模板显示，并且可供下载。 该模板仅包含单个资源。

## <a name="export-template-before-deployment"></a>导出模板，然后再进行部署

1. 选择你想要部署的 Azure 服务。

1. 填写新服务的值。

1. 之后通过验证，而开始部署之前，请选择**下载自动化模板**。

   ![下载模板](./media/export-template-portal/download-before-deployment.png)

1. 模板显示，并可供下载。

   ![显示模板](./media/export-template-portal/show-template-before-deployment.png)

## <a name="export-template-after-deployment"></a>在部署后导出模板

您可以导出用于部署现有资源的模板。 获取该模板是完全部署所用的一个。

1. 选择你想要导出的资源组。

1. 选择下的链接**部署**。

   ![选择部署历史记录](./media/export-template-portal/select-deployment-history.png)

1. 从部署历史记录中选择其中一个部署。

   ![选择部署](./media/export-template-portal/select-details.png)

1. 选择**模板**。 用于此部署的模板显示，并且可供下载。

   ![选择模板](./media/export-template-portal/show-template-from-history.png)

## <a name="next-steps"></a>后续步骤

- 若要了解 Azure 资源管理器，请参阅 [Azure 资源管理器概述](./resource-group-overview.md)。
- 若要了解资源管理器模板语法，请参阅[了解 Azure 资源管理器模板的结构和语法](./resource-group-authoring-templates.md)。
- 若要了解如何开发模板，请参阅[分步教程](/azure/azure-resource-manager/)。
- 若要查看 Azure 资源管理器模板架构，请参阅[模板参考](/azure/templates/)。