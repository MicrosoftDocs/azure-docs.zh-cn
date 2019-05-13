---
title: 什么是表单识别器？
titleSuffix: Azure Cognitive Services
description: 了解如何使用表单识别器来分析表单和表数据。
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: form-recognizer
ms.topic: overview
ms.date: 04/08/2019
ms.author: pafarley
ms.openlocfilehash: 2a120a59a58eb8d7a017cce0dd85c21038bdcf51
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143214"
---
# <a name="what-is-form-recognizer"></a>什么是表单识别器？

Azure 表单识别器是一个认知服务，它使用机器学习技术从表与表单中识别和提取键值对。 然后，它会输出包含原始文件中的关系的结构化数据。 可以使用简单的 REST API 调用自定义表单识别器模型，以降低复杂性，并轻松地将该模型集成到工作流或应用程序中。 只需使用五个相同类型的表单文档或空表单作为输入材料即可开始使用该服务。 可以快速准确地获取根据特定内容定制的结果，而无需进行繁琐的手动干预，也不需要具备丰富的数据科学专业知识。

## <a name="request-access"></a>请求访问权限
表单识别器目前以受限访问预览版提供。 若要访问预览版，请填写并提交[认知服务表单识别器访问请求](https://aka.ms/FormRecognizerRequestAccess)表单。 该表单将请求有关你、你的公司以及要使用表单识别器的用户方案的信息。 如果 Azure 认知服务团队已批准你的请求，则你会收到一封电子邮件，其中提供了有关如何访问该服务的说明。

## <a name="what-it-does"></a>作用

当你提交输入数据时，算法会训练这些数据，按类型聚集表单，发现现有的键和表，并学习如何将值关联到键，以及将项关联到表。 非监督式学习可让模型了解字段与项之间的布局和关系，而无需手动标记数据，或者进行繁琐的编程和维护。 相比之下，预先训练的机器学习模型需要标准化的数据，并且准确度比不同于传统格式的输入材料（例如行业特定的表单）更低。

训练模型后，可对其进行测试、重新训练，并最终使用它来根据需求提取其他表单中的数据。

## <a name="what-it-includes"></a>组成部分

表单识别器以 REST API 的形式提供。 可以通过调用 API 来创建、训练和评分模型，并选择性地在本地 Docker 容器中运行该模型。

## <a name="input-requirements"></a>输入要求

表单识别器可以处理符合以下要求的输入文档：

* JPG、PNG 或 PDF 格式（文本或扫描件）。 最好是使用文本嵌入式 PDF，因为这可以避免在提取和定位字符时出错。
* 文件大小必须小于 4 MB
* 对于图像，尺寸必须介于 50x50 与 4200x4200 像素之间
* 如果从纸质文档扫描表单，应使用优质扫描仪
* 必须使用拉丁字母（英文字符）
* 打印的数据（非手写）
* 必须包含键和值
* 键可以显示在值的上方或左侧，而不能显示在下方或右侧。

此外，表单识别器尚不支持以下类型的输入数据：

* 复杂表（嵌套表、合并的标题或单元格，等等） 
* 复选框或单选按钮
* 超过 50 页的 PDF 文档

## <a name="where-do-i-start"></a>从哪里开始？

**步骤 1：** 在 Azure 门户中创建表单识别器资源。

**步骤 2：** 尝试参考快速入门进行实际体验：
* [快速入门：使用 REST API 和 cURL 训练表单识别器模型并提取表单数据](quickstarts/curl-train-extract.md)
* [快速入门：使用 REST API 和 Python 训练表单识别器模型并提取表单数据](quickstarts/python-train-extract.md)

我们建议将免费服务用于学习目的，但请注意，免费页数限制为每月 500 页。

**步骤 3：** 查看 REST API，并使用以下 API 来训练和提取表单中的结构化数据。

| REST API | 说明 |
|-----|-------------|
| 定型 | 训练一个新模型，以使用相同类型的 5 个表单或一个空表单来分析你的表单。  |
| 分析  |使用自定义模型分析作为流传递的单个文档，以提取表单中的键值对和表。  |

浏览 [REST API 参考文档](https://aka.ms/form-recognizer/api)。 

## <a name="data-privacy-and-security"></a>数据隐私和安全性

该服务是根据[联机服务条款](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)以 Azure 服务[预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)的形式提供的。 与所有认知服务一样，使用表单识别器服务的开发人员应该了解 Microsoft 针对客户数据的政策。 请参阅 Microsoft 信任中心上的[“认知服务”页面](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices)来了解详细信息。

## <a name="next-steps"></a>后续步骤

遵循[快速入门](quickstarts/curl-train-extract.md)开始使用[表单识别器 API](https://aka.ms/form-recognizer/api)。
