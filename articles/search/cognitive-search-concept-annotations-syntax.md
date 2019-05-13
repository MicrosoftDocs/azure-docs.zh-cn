---
title: 认知搜索管道中的引用输入和输出 - Azure 搜索
description: 说明了注释语法，以及如何在 Azure 搜索的认知搜索管道中引用技能集的输入和输出中的注释。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 637edc0e45daa37a753fbaa15313b076e8af4d7c
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023878"
---
# <a name="how-to-reference-annotations-in-a-cognitive-search-skillset"></a>如何在认知搜索技能集中引用注释

本文介绍如何在技能定义中引用注释，并使用示例演示各种方案。 当文档的内容流经一组技能时，它将通过注释进行扩充。 注释可以用作进一步下游扩充的输入，也可以映射到索引中的输出字段。 
 
本文中的示例基于 *content* 字段，该字段由 [Azure Blob 索引器](search-howto-indexing-azure-blob-storage.md)在文档破解阶段中自动生成。 从 Blob 容器引用文档时，使用 `"/document/content"` 等格式，其中 *content* 字段是*文档*的一部分。 

## <a name="background-concepts"></a>背景概念

在复习语法之前，让我们回顾一些重要的概念，以便更好地理解本文后面提供的示例。

| 术语 | 描述 |
|------|-------------|
| 扩充文档 | 扩充文档是由管道创建和使用的内部结构，用来保存与文档相关的所有注释。 可以把扩充文档看作是注释树。 通常，从前一个注释创建的注释将成为前一个注释的子级。<p/>扩充文档仅在技能集执行期间存在。 内容映射到搜索索引后，就不再需要扩充文档了。 虽然不直接与扩充文档交互，但在创建技能集时，有一个文档的心理模型是很有用的。 |
| 扩充上下文 | 扩充发生的上下文，即扩充的元素。 默认情况下，扩充上下文位于 `"/document"` 级别，作用域为单个文档。 当一个技能运行时，该技能的输出将成为[定义上下文的属性](#example-2)。|

<a name="example-1"></a>
## <a name="example-1-simple-annotation-reference"></a>示例 1：简单注释引用

在 Azure Blob 存储中，假设您有很多其他的文件，其中包含对你要从中提取使用实体识别的人的名称的引用。 在下面的技能定义中，`"/document/content"` 是整个文档的文本表示，“people”是对标识为 persons 的实体的全名提取。

因为默认上下文是 `"/document"`，所以现在可以将人员列表引用为 `"/document/people"`。 在此特定示例中 `"/document/people"` 是一个注释，它现在可以映射到索引中的一个字段，或者用在同一技能集的另一个技能中。

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "categories": [ "Person"],
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      }
    ]
  }
```

<a name="example-2"></a>

## <a name="example-2-reference-an-array-within-a-document"></a>示例 2：引用文档中的数组

此示例基于前一个示例构建，演示如何在同一个文档中多次调用一个扩充步骤。 假设前面的示例生成了一个字符串数组，其中有 10 个人名来自单个文档。 合理的下一步可能是二次扩充，将姓氏从全名中提取出来。 因为有 10 个姓名，所以你希望在这个文档中调用此步骤 10 次，每个人一次。 

若要调用正确的迭代次数，请将上下文设置为 `"/document/people/*"`，其中星号 (`"*"`) 表示扩充文档中作为 `"/document/people"` 的后代的所有节点。 尽管此技能只在技能数组中定义了一次，但它会对文档中的每个成员调用，直到所有成员都被处理。

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the last name from a full name",
    "uri": "http://names.azurewebsites.net/api/GetLastName",
    "context" : "/document/people/*",
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "fullname",
        "source": "/document/people/*"
      }
    ],
    "outputs": [
      {
        "name": "lastname",
        "targetName": "last"
      }
    ]
  }
```

当注释是字符串的数组或集合时，你可能希望针对特定成员而不是整个数组。 上述示例在由上下文表示的每个节点下生成名为 `"last"` 的注释。 如果想要引用此系列的注释，可以使用语法 `"/document/people/*/last"`。 如果想要引用特定注释，可以使用显式索引：`"/document/people/1/last` 引用文档中标识的第一个人的姓氏。 请注意，在此语法中数组的“索引从 0 开始”。

<a name="example-3"></a>

## <a name="example-3-reference-members-within-an-array"></a>示例 3：引用数组中的成员

有时，需要对特定类型的所有注释进行分组，以将它们传递给特定技能。 考虑一种假设的自定义技能，它标识在示例 2 中提取的所有姓氏中最常见的姓氏。 若要只将姓氏提供自定义技能，请将上下文指定为 `"/document"`，将输入为指定为 `"/document/people/*/lastname"`。

请注意的基数`"/document/people/*/lastname"`大于的文档。 就此文档而言，可能有 10 个 lastname 节点，而只有一个 document 节点。 在这种情况下，系统将自动创建包含文档中的所有元素的 `"/document/people/*/lastname"` 数组。

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the most common string from an array of strings",
    "uri": "http://names.azurewebsites.net/api/MostCommonString",
    "context" : "/document",
    "inputs": [
      {
        "name": "strings",
        "source": "/document/people/*/lastname"
      }
    ],
    "outputs": [
      {
        "name": "mostcommon",
        "targetName": "common-lastname"
      }
    ]
  }
```



## <a name="see-also"></a>另请参阅
+ [如何将自定义技能集成到扩充管道](cognitive-search-custom-skill-interface.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
+ [创建技能集 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何将扩充字段映射到索引](cognitive-search-output-field-mapping.md)
