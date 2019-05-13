---
title: 使用 Azure 门户编辑 API | Microsoft Docs
description: 本教程演示如何使用 API 管理 (APIM) 编辑 API。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 11/08/2017
ms.author: apimpm
ms.openlocfilehash: 6f1a0cf6025cb3a398ab93320c6fcb69b1e62429
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60557996"
---
# <a name="edit-an-api"></a>编辑 API

本教程中的步骤演示如何使用 API 管理 (APIM) 编辑 API。 

+ 可以通过在 APIM 实例中添加、删除、重命名操作来执行此操作。 
+ 可以编辑 API 的 swagger。

## <a name="prerequisites"></a>先决条件

+ [创建一个 Azure API 管理实例](get-started-create-service-instance.md)
+ [导入和发布第一个 API](import-and-publish.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="edit-an-api-in-apim"></a>在 APIM 中编辑 API

![编辑 API](./media/edit-api/edit-api001.png)

1. 单击 **API** 选项卡。
2. 选择以前导入的某个 API。
3. 选择“设计”选项卡。
4. 选择要编辑的操作。
5. 若要重命名该操作，请在“前端”窗口中选择“铅笔”。

## <a name="update-the-swagger"></a>更新 swagger

可以通过执行以下步骤来从 Azure 门户更新后端 API：

1. 选择“所有操作”
2. 单击“前端”窗口中的铅笔。

    ![编辑 API](./media/edit-api/edit-api002.png)

    将显示 API 的 swagger。

    ![编辑 API](./media/edit-api/edit-api003.png)

3. 更新 swagger。
4. 按“保存”。

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [APIM 策略示例](policy-samples.md)
> [转换和保护已发布的 API](transform-api.md)