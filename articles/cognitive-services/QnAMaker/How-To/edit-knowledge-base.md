---
title: 编辑知识库 - QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker 允许你通过提供易于使用的编辑体验来管理知识库的内容。
services: cognitive-services
author: tulasim88
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 05/10/2019
ms.author: tulasim
ms.custom: seodec18
ms.openlocfilehash: c9add80b7494ae2a8e671967a96dc5d3c7307f51
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541655"
---
# <a name="edit-a-knowledge-base-in-qna-maker"></a>在 QnA Maker 中编辑知识库

QnA Maker 允许你通过提供易于使用的编辑体验来管理知识库的内容。

<a name="add-datasource"></a>

## <a name="edit-your-knowledge-base-content"></a>编辑知识库内容

1.  在顶部导航栏中选择“我的知识库”。 

    你可以按“上次修改时间”的降序排序，查看你创建或共享的所有服务。

    ![我的知识库](../media/qnamaker-how-to-edit-kb/my-kbs.png)

1. 选择特定的知识库以对其进行编辑。
 
1. 选择“设置”。 可以在此处编辑必填字段“服务名称”。
  
    |目标|操作|
    |--|--|
    |添加 URL|可以通过单击“管理知识库”- >“+ 添加 URL”链接，添加新的 URL，以便向知识库添加新的常见问题解答内容。|
    |删除 URL|可以通过选择删除图标（垃圾桶）来删除现有 URL。|
    |刷新 URL 内容|如果希望知识库抓取现有 URL 的最新内容，请选中“刷新”复选框。 这将使用最新的 URL 内容更新知识库。|
    |添加文件|可以通过选择“管理知识库”并选择“+ 添加文件”将受支持的文件文档添加为知识库的一部分。|
    |导入|还可以通过选择“导入知识库”按钮导入任何现有知识库。 |
    |更新|知识库的更新取决于在创建与知识库关联的 QnA Maker 服务时使用的**管理定价层**。 如果需要，还可以从 Azure 门户更新管理层。

1. 完成对知识库的更改后，选择页面右上角的“保存并训练”以保留更改。    

    ![保存和训练](../media/qnamaker-how-to-edit-kb/save-and-train.png)

    >[!CAUTION]
    >如果在选择“保存并训练”前离开页面，则所有更改都将丢失。

## <a name="add-a-qna-pair"></a>添加 QnA 对

上**设置**页上，选择**添加 QnA 对**将新行添加到知识库表。

![添加 QnA 对](../media/qnamaker-how-to-edit-kb/add-qnapair.png)

## <a name="delete-a-qna-pair"></a>删除 QnA 对

要删除 QnA，请单击 QnA 行最右侧的删除图标。 这是一项永久性操作。 无法撤消。 请考虑在删除对之前从“发布”页面导出知识库。 

![删除 QnA 对](../media/qnamaker-how-to-edit-kb/delete-qnapair.png)

## <a name="add-alternate-questions"></a>添加替代问题

向现有 QnA 对添加替代问题，以提高与用户查询匹配的可能性。

![添加替代问题](../media/qnamaker-how-to-edit-kb/add-alternate-question.png)

## <a name="add-metadata"></a>添加元数据

第一个选择添加元数据对**查看选项**，然后选择**显示元数据**。 此时将显示的元数据列。 接下来，选择**+** 号来添加的元数据对。 此对组成一个键和一个值。

![添加元数据](../media/qnamaker-how-to-edit-kb/add-metadata.png)

> [!TIP]
> 确保在编辑后定期保存并定型知识库，以避免丢失更改。

## <a name="manage-large-knowledge-bases"></a>管理大型知识库

* **数据源组**：QnAs 按从中提取它们的数据源进行分组。 你可以展开或折叠数据源。

    ![使用 QnA Maker 数据源栏来折叠和展开数据源问题和解答](../media/qnamaker-how-to-edit-kb/data-source-grouping.png)

* **搜索知识库**：可以通过在知识库表顶部的文本框中键入内容来搜索知识库。 单击 Enter 以搜索问题、答案或元数据内容。 单击 X 图标以删除搜索筛选器。

    ![使用问题和解答上方的 QnA Maker 搜索框可将视图缩小为仅筛选匹配项](../media/qnamaker-how-to-edit-kb/search-paginate-group.png)

* **分页**：快速浏览数据源以管理大型知识库

    ![使用问题和解答上方的 QnA Maker 分页功能可浏览问题和解答页面](../media/qnamaker-how-to-edit-kb/pagination.png)

## <a name="delete-knowledge-bases"></a>删除知识库

删除知识库 (KB) 是一项永久性操作， 无法撤消。 删除知识库之前，应该从 QnA Maker 门户的“设置”页导出知识库。 

如果你与[协作者](collaborate-knowledge-base.md)共享知识库，然后将该知识库删除，则每个人都会失去对该知识库的访问权限。 

## <a name="delete-azure-resources"></a>删除 Azure 资源 

如果你删除了用于 QnA Maker 知识库的任何 Azure 资源，则知识库将不再工作。 在删除任何资源之前，请务必从“设置”页导出知识库。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [针对知识库进行协作](./collaborate-knowledge-base.md)
