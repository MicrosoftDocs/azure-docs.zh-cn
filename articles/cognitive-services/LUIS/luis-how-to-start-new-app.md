---
title: 创建新应用
titleSuffix: Language Understanding - Azure Cognitive Services
description: 在语言理解 (LUIS) 网页上创建和管理应用程序。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 9d650a17ddfac6461341e50c4693e4522d9628b3
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148202"
---
# <a name="create-a-new-luis-app-in-the-luis-portal"></a>在 LUIS 门户中创建新的 LUIS 应用
可通过多种方法创建 LUIS 应用。 可以在 [LUIS](https://www.luis.ai) 门户中创建 LUIS 应用，也可以通过 LUIS 创作 [API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f) 创建。

## <a name="using-the-luis-portal"></a>使用 LUIS 门户
可以通过以下几种方式在 LUIS 门户中创建新应用：

* 从一个空应用开始，创建意向、表述和实体。
* 从一个空应用开始，添加[预生成的域](luis-how-to-use-prebuilt-domains.md)。
* 从已包含意向、表述和实体的 JSON 文件导入 LUIS 应用。

## <a name="using-the-authoring-apis"></a>使用创作 API
可以通过以下几种方式使用创作 API 创建新应用：

* [首先](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f)创建一个空应用，然后创建意向、表述和实体。
* 从预生成域[开始](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/59104e515aca2f0b48c76be5)。  


<a name="export-app"></a>
<a name="import-new-app"></a>
<a name="delete-app"></a>
 

## <a name="create-new-app-in-luis"></a>在 LUIS 中创建新应用

1. 在“我的应用”页，选择“创建新应用”。

    ![LUIS 应用列表](./media/luis-create-new-app/apps-list.png)


2. 在对话框中，将应用程序命名为“TravelAgent”。

    ![“创建新应用”对话框](./media/luis-create-new-app/create-app.png)

3. 选择应用程序区域性（对于 TravelAgent 应用，请选择英语），然后选择“完成”。 

    > [!NOTE]
    > 创建应用程序后将无法更改区域性。 

## <a name="import-an-app-from-file"></a>从文件导入应用

1. 在“我的应用”页，选择“导入新应用”。
1. 在弹出对话框中，选择一个有效的应用 JSON 文件，并选择**完成**。

### <a name="import-errors"></a>导入错误

可能的错误为： 

* 已存在具有该名称的应用。 重新导入应用程序中，并设置**可选名称**为新名称。 

## <a name="export-app-for-backup"></a>导出应用备份

1. 上**我的应用**页上，选择**导出**。
1. 选择**导出为 JSON**。 在浏览器下载活动版本的应用。
1. 将此文件添加到您的备份系统进行存档该模型。

## <a name="export-app-for-containers"></a>导出用于容器的应用

1. 上**我的应用**页上，选择**导出**。
1. 选择**导出为容器**然后选择你想要导出的已发布的槽 （生产或阶段）。
1. 使用此文件和你[LUIS 容器](luis-container-howto.md)。 

    如果您感兴趣但不是导出一个训练的但要使用 LUIS 容器使用的已发布的模型，请转到**版本**页和从中导出。 

## <a name="delete-app"></a>删除应用

1. 在“我的应用”页，选中应用所在行末尾的三个点 (...)。
1. 从菜单中选择“删除”。
1. 在确认消息窗口中，选择“确定”。

## <a name="next-steps"></a>后续步骤

应用中的第一个任务是[添加意向](luis-how-to-add-intents.md)。
