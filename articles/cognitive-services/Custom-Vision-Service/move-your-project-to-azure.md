---
title: 将有限的试用版项目移到 Azure
titlesuffix: Azure Cognitive Services
description: 了解如何将限时试用项目移到 Azure。
services: cognitive-services
author: anrothMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: anroth
ms.openlocfilehash: 6fac6531ea0a39796de13f95aee33b30dc91f131
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60816514"
---
# <a name="how-to-move-your-limited-trial-project-to-azure"></a>如何将限时试用项目移到 Azure

当自定义影像服务完成其迁移到 Azure 时，正在终止对 Azure 外部的限时试用项目的支持。 本文档将演示如何使用自定义视觉 Api 来将你的限时试用项目复制到 Azure 资源。

支持查看受限制的试用版项目上[自定义视觉网站](https://customvision.ai)已于 2019 年 3 月 25 日结束。 本文档现在显示如何使用自定义视觉 Api 与[迁移 python 脚本](https://github.com/Azure-Samples/custom-vision-move-project)GitHub 上) 复制到 Azure 资源项目。

有关详细信息，包括关键的截止时间在有限的试用版不推荐使用过程中，请参阅[发行说明](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/release-notes#february-25-2019)或电子邮件通信发送到的受限试用版的项目所有者。

[迁移脚本](https://github.com/Azure-Samples/custom-vision-move-project)，可通过下载，并上传的所有标记的区域，来重新创建项目和图像中当前迭代。 它将使用新的项目，然后可以训练在新订阅中。

## <a name="prerequisites"></a>必备组件

- 你将需要有效的 Azure 订阅与 Microsoft 帐户或你想要用于登录到 Azure Active Directory (AAD) 帐户相关联[自定义视觉网站](https://customvision.ai)。 
    - 如果还没有 Azure 帐户，[创建一个帐户](https://azure.microsoft.com/free/)免费。
    - 有关订阅和资源的 Azure 概念的简介，请参阅[Azure 开发人员指南。](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#manage-your-subscriptions)。
-  [Python](https://www.python.org/downloads/)
- [Pip](https://pip.pypa.io/en/stable/installing/)

## <a name="create-custom-vision-resources-in-the-azure-portal"></a>在 Azure 门户中创建自定义视觉资源

若要与 Azure 结合使用自定义影像服务，你将需要创建自定义视觉训练和预测中的资源[Azure 门户](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision)。 

多个项目可以对单个资源相关联。 有关更多详细信息[定价和限制](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/limits-and-quotas)可用。 若要继续免费使用自定义影像服务，可以在 Azure 门户中选择 F0 层。 

> [!NOTE]
> 当将自定义视觉项目移到 Azure 资源时，它将继承的基础[权限]( https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)该 Azure 资源。 如果你的组织中的其他用户是你的项目是在 Azure 资源的所有者，他们将能够访问你的项目上[自定义视觉网站](https://customvision.ai)。 同样，删除你的资源将删除你的项目。  

## <a name="find-your-limited-trial-project-information"></a>查找你的受限试用版的项目信息

若要移动你的项目，将需要_项目 ID_并_培训密钥_想要迁移的项目。 如果没有此信息，请访问[ https://limitedtrial.customvision.ai/projects ](https://limitedtrial.customvision.ai/projects)若要获取的 ID 和密钥，以每个项目。 

## <a name="use-the-python-sample-code-to-copy-your-project-to-azure"></a>使用 Python 示例代码来将你的项目复制到 Azure

请按照[示例代码说明](https://github.com/Azure-Samples/custom-vision-move-project)、 使用有限的试用版密钥和项目 ID 为"source"材料和新的 Azure 资源创建为"destination"中的密钥。

默认情况下，有限的试用版的所有项目都托管在美国中南部 Azure 区域中。

## <a name="next-steps"></a>后续步骤

你的项目现在已移至 Azure 资源。 您需要更新训练和预测中任何已编写的应用程序密钥。

若要查看你的项目上[自定义视觉网站](https://customvision.ai)，用来登录 Azure 门户的同一帐户登录。 如果没有看到你的项目，请确认你已在同一目录中[自定义视觉网站](https://customvision.ai)作为你的资源位于 Azure 门户中的目录。 在 Azure 门户和 CustomVision.ai 上，你可以从屏幕的右上角的下拉列表用户菜单选择你的目录。