---
title: 条件性的认知搜索技能 （Azure 搜索） |Microsoft Docs
description: 允许筛选、 创建默认值，以及合并值的条件性技能。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: luisca
ms.openlocfilehash: 6a203a38437ccb6a9c325e6594289744e0148c84
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028420"
---
#   <a name="conditional-skill"></a>条件的技能

**条件技能**可用于各种方案需要布尔操作以确定应分配给输出的数据。 这些方案包括： 根据条件筛选、 指定的默认值和合并数据。

下面的伪代码说明了条件技能完成：

```
if (condition) 
    { output = whenTrue } 
else 
    { output = whenFalse } 
```

> [!NOTE]
> 此技能未绑定到认知服务 API，你使用它无需付费。 但是，你仍然应该[附加认知服务资源](cognitive-search-attach-cognitive-services.md)，以覆盖**免费**资源选项，该选项限制你每天进行少量的每日扩充。

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ConditionalSkill


## <a name="evaluated-fields"></a>计算的字段

此技术很特别，因为其输入为计算的字段。

以下是表达式的有效的值：

-   批注路径 (在表达式中的路径必须分隔"$("and")") <br/>
    示例：
    ```
        "= $(/document)"
        "= $(/document/content)"
    ```

-  文本 （字符串、 数字、 true、 false、 null） <br/>
    示例：
    ```
       "= 'this is a string'"   // string, note the single quotes
       "= 34"                   // number
       "= true"                 // boolean
       "= null"                 // null value
    ```

-  使用比较运算符的表达式 (= =、 ！ =、 > =、 >、 < =、 <) <br/>
    示例：
    ```
        "= $(/document/language) == 'en'"
        "= $(/document/sentiment) >= 0.5"
    ```

-   使用布尔运算符的表达式 (& &、 | |，！，^) <br/>
    示例：
    ```
        "= $(/document/language) == 'en' && $(/document/sentiment) > 0.5"
        "= !true"
    ```

-   使用数字运算符的表达式 (+、-， \*、 /、 %) <br/>
    示例： 
    ```
        "= $(/document/sentiment) + 0.5"         // addition
        "= $(/document/totalValue) * 1.10"       // multiplication
        "= $(/document/lengthInMeters) / 0.3049" // division
    ```

由于支持评估，可以为次要转换方案使用条件的技能。 请参阅示例[技能定义 4](#transformation-examples)有关的示例。

## <a name="skill-inputs"></a>技能输入
输入区分大小写。

| 输入      | 描述 |
|-------------|-------------|
| condition   | 此输入是[计算字段](#evaluated-fields)，表示要计算的条件。 此条件计算结果应为布尔值 （true 或 false）。   <br/>  示例： <br/> "= true" <br/> "= $(/document/language) =='fr'" <br/> "= $(/document/pages/\*/language) == $(/document/expectedLanguage)" <br/> |
| whenTrue    | 此输入是[计算字段](#evaluated-fields)。 要返回对条件进行评估的值为 true。 应在返回常量字符串引号引起来。 <br/>示例值： <br/> "= 'contract'"<br/>"= $(/document/contractType)" <br/> "= $(/document/entities/\*)" <br/> |
| whenFalse   | 此输入是[计算字段](#evaluated-fields)。 要返回该条件的计算结果为 false 的值。  <br/>示例值： <br/> "= 'contract'"<br/>"= $(/document/contractType)" <br/> "= $(/document/entities/\*)" <br/>

## <a name="skill-outputs"></a>技能输出
还有一个名为 output 输出。 如果条件为 true，它将返回 whenFalse 如果条件为 false 或 whenTrue 的值。

## <a name="examples"></a>示例

### <a name="sample-skill-definition-1-filtering-documents-to-return-only-french-documents"></a>示例技能定义 1:筛选文档返回仅"法语"文档

下面的输出将返回的句子 ("/ 文档/frenchSentences") 数组，如果文档的语言为法语。 如果语言不是法语，将设置此值为 null。

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'fr'" },
        { "name": "whenTrue", "source": "/document/sentences" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "frenchSentences" } ]
}
```
如果"文档/frenchSentences"用作*上下文*的另一项技巧，该技能将仅运行如果未设置"/ 文档/frenchSentences"，则为 null


### <a name="sample-skill-definition-2-setting-a-default-value-when-it-does-not-exist"></a>示例技能定义 2:如果不存在，请设置默认值。

下面的输出将创建批注 ("/ 文档/languageWithDefault")，如果未设置语言设置为文档的语言或"es"。

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'es'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

### <a name="sample-skill-definition-3-merging-values-from-two-different-fields-into-a-single-field"></a>示例技能定义 3:将来自两个不同字段的值合并成单个字段

在此示例中，有一些句子*frenchSentiment*属性。 每当*frenchSentiment*属性为 null，则我们想要使用*englishSentiment*值。 我们将输出分配给一个名为只需成员*情绪*("/ 记录/情绪 / * / 情绪")。

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= $(/document/sentences/*/frenchSentiment) == null" },
        { "name": "whenTrue", "source": "/document/sentences/*/englishSentiment" },
        { "name": "whenFalse", "source": "/document/sentences/*/frenchSentiment" }
    ],
    "outputs": [ { "name": "output", "targetName": "sentiment" } ]
}
```

## <a name="transformation-examples"></a>转换示例
### <a name="sample-skill-definition-4-performing-data-transformations-on-a-single-field"></a>示例技能定义 4:对单个字段执行数据转换

在此示例中，我们收到的一种情绪 0 和 1 之间，以及我们想要将其转换，这样就为-1 和 1 之间。 这是我们可以执行使用条件性技能的较小的数学转换。

在此特定示例中，我们可以根据条件始终为 true 永远不会使用到的条件性方面的技能。 

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= true" },
        { "name": "whenTrue", "source": "= $(/document/sentences/*/sentiment) * 2 - 1" },
        { "name": "whenFalse", "source": "= 0" }
    ],
    "outputs": [ { "name": "output", "targetName": "normalizedSentiment" } ]
}
```


## <a name="special-considerations"></a>特殊注意事项
请注意的评估某些参数，因此您需要有案可稽的模式时务必小心。 表达式必须以等号"="开头，并且必须由分隔路径"$("and")"。 请确保将您的字符串放在 '单引号'，因为这将帮助区分字符串和实际路径和运算符，计算器。 此外，请确保将运算符周围的空格 (例如 * 路径中具有不同的含义，乘法运算符)。


## <a name="next-steps"></a>后续步骤

+ [预定义技能](cognitive-search-predefined-skills.md)
+ [如何定义技能集](cognitive-search-defining-skillset.md)
