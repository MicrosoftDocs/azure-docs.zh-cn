---
title: 实体类型
titleSuffix: Language Understanding - Azure Cognitive Services
description: 实体从查询文本中提取数据。 实体类型提供可预测的数据提取。 有两种类型的实体： 机器学习的内容和机器学习。 请务必知道哪种类型的查询文本中使用的实体。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: diberry
ms.openlocfilehash: 7fd9ae3ab1f50dc91118ba11bc357a0f6dc0e771
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141043"
---
# <a name="entity-types-and-their-purposes-in-luis"></a>实体类型及其在 LUIS 中的目的

实体从查询文本中提取数据。 实体类型提供可预测的数据提取。 有两种类型的实体： 机器学习的内容和机器学习。 请务必知道哪种类型的查询文本中使用的实体。 

## <a name="entity-compared-to-intent"></a>实体与意向

实体表示要提取的话语中的字词或短语。 话语可包括多个实体，也可不包含任何实体。 客户端应用程序可能需要执行其任务，或将其用作多个选项的指南来向用户显示的实体。 

实体：

* 表示包括一系列类似对象 （位置、 内容、 人员、 事件或概念） 的类。 
* 描述与意图相关的信息


例如，新闻搜索应用可能包括“主题”、“源”、“关键字”和“发布日期”等实体，这些是用于搜索新闻的关键数据。 在旅行预订应用中，“位置”、“日期”、“航空公司”、“舱位等级”和“机票”均为航班预订（与“Bookflight”意向相关）的关键信息。

相比之下，意向表示有关整个话语的预测。 

## <a name="entities-help-with-data-extraction-only"></a>实体仅有助于提取数据

只能出于提取实体的目的标记实体，实体对于意向预测没有任何帮助。

## <a name="entities-represent-data"></a>实体表示数据

实体是要从话语中提取的数据。 可以是姓名、日期、产品名称或任何词组。 

|话语|实体|数据|
|--|--|--|
|购买 3 张到纽约的机票|预生成的数字<br>Location.Destination|3<br>纽约|
|购买 3 月 5 日从纽约到伦敦的机票|Location.Origin<br>Location.Destination<br>预生成的 datetimeV2|纽约<br>伦敦<br>2018 年 3 月 5 日|

## <a name="entities-are-optional-but-highly-recommended"></a>实体是可选的（但强烈建议使用）

意向是必需的，而实体是可选的。 无需为应用中的每个概念创建实体，只需为客户端应用程序操作时所需的概念创建实体。 

如果话语不含机器人继续操作所需的详细信息，则无需添加这些信息。 随着应用的成熟，可在稍后添加信息。 

如果不确定如何使用这些信息，可添加一些常见的预生成的实体，如 [datetimeV2](luis-reference-prebuilt-datetimev2.md)、[序号](luis-reference-prebuilt-ordinal.md)、[电子邮件](luis-reference-prebuilt-email.md)和[电话号码](luis-reference-prebuilt-phonenumber.md)。

## <a name="label-for-word-meaning"></a>字词含义的标签

如果选词或字词排列方式相同，但含义并不相同，请勿将其标记为实体。 

以下话语中，`fair` 一词为同形异义词。 该词虽拼写相同但含义不同：

|话语|
|--|
|今年夏天西雅图地区会举办什么样的乡村集市？|
|西雅图评审的当前评级公平吗？|

如果希望事件实体查找所有事件数据，请标记第一个话语中的 `fair` 一词，而不是第二个话语。

## <a name="entities-are-shared-across-intents"></a>实体在意向之间共享

实体在意向之间共享。 它们不属于任一意向。 意向和实体可在语义上相关联，但这并非一种排它性的关系。

在“为我预订到巴黎的机票”这一话语中，“巴黎”是引用位置的实体。 通过识别用户话语中提到的实体，LUIS 可帮助客户端应用程序选择特定的操作来实现用户的意向。

## <a name="mark-entities-in-none-intent"></a>在 None 意向中标记实体

在可能的情况下，所有意向（包括 **None** 意向）应包含标记的实体。 这可帮助 LUIS 深入了解实体在话语中的位置，以及实体周围的字词。 

## <a name="entity-status-for-predictions"></a>预测的实体状态

当示例话语中的实体不同于标记的实体，或者与另一实体过于接近，因此不够明确时，LUIS 门户会告知这一点。 它会在示例话语中插入一条红色的下划线。 

有关详细信息，请参阅[实体状态预测](luis-how-to-add-example-utterances.md#entity-status-predictions)。 

## <a name="types-of-entities"></a>实体类型

LUIS 提供许多类型的实体。 请根据数据的提取方式以及提取后的数据表示方式，来选择实体。

可以使用机器学习提取实体，这样，LUIS 就可以持续学习该实体在话语中的显示方式。 可以通过对文本或正则表达式进行完全匹配，在不使用机器学习的情况下提取实体。 可以使用混合实现提取模式中的实体。 

提取实体后，可将实体数据表示为单个信息单元，或者将它与其他实体相结合，构成客户端应用程序可以使用的信息单元。

|机器学习|可以标记|教程|示例<br>响应|实体类型|目的|
|--|--|--|--|--|--|
|✔|✔|[✔](luis-tutorial-composite-entity.md)|[✔](luis-concept-data-extraction.md#composite-entity-data)|[**复合**](#composite-entity)|实体的分组，不考虑实体类型。|
|||[✔](luis-quickstart-intent-and-list-entity.md)|[✔](luis-concept-data-extraction.md#list-entity-data)|[**列表**](#list-entity)|使用文本完全匹配法提取的项列表及其同义词。|
|混合||[✔](luis-tutorial-pattern.md)|[✔](luis-concept-data-extraction.md#patternany-entity-data)|[**Pattern.any**](#patternany-entity)|难以确定末尾部分的实体。|
|||[✔](luis-tutorial-prebuilt-intents-entities.md)|[✔](luis-concept-data-extraction.md#prebuilt-entity-data)|[**预生成**](#prebuilt-entity)|已经过训练，可以提取各种类型的数据。|
|||[✔](luis-quickstart-intents-regex-entity.md)|[✔](luis-concept-data-extraction.md#regular-expression-entity-data)|[**正则表达式**](#regular-expression-entity)|使用正则表达式来匹配文本。|
|✔|✔|[✔](luis-quickstart-primary-and-secondary-data.md)|[✔](luis-concept-data-extraction.md#simple-entity-data)|[**简单**](#simple-entity)|包含单词或短语中的单个概念。|

只有机器学习的内容的实体需要标记的示例语音样本。 机器学习到的实体最适合通过[终结点查询](luis-concept-test.md#endpoint-testing)和[查看终结点话语](luis-how-to-review-endoint-utt.md)进行测试。 

需要在[模式](luis-how-to-model-intent-pattern.md)模板示例而不是意向用户示例中标记 Pattern.any 实体。 

混合实体使用实体检测方法的组合。

## <a name="composite-entity"></a>复合实体

复合实体由组成的其他实体，如预生成的实体，简单，正则表达式和列表实体。 各种单独的实体构成整个实体。 

如果数据具有以下特征，则非常适合使用此实体：

* 彼此相关。 
* 在使用陈述的情况下彼此相关。
* 使用各种实体类型。
* 需要由客户端应用程序作为一个信息单元进行分组和处理。
* 包含需要机器学习的各种用户话语。

![复合实体](./media/luis-concept-entities/composite-entity.png)

[教程](luis-tutorial-composite-entity.md)<br>
[实体的 JSON 响应示例](luis-concept-data-extraction.md#composite-entity-data)<br>

## <a name="list-entity"></a>列表实体

列表实体表示一组固定、封闭的相关单词及其同义词。 LUIS 不会为列表实体发现更多值。 使用“建议”功能根据当前列表查看有关新词的建议。 如果存在多个具有相同值的列表实体，则终结点查询中会返回其中每个实体。 

如果文本数据具有以下特征，则非常适合使用该实体：

* 是已知的集。
* 此集不超出此实体类型的最大 LUIS [边界](luis-boundaries.md)。
* 话语中的文本是同义项或规范名称的完全匹配。 LUIS 不会使用除文本完全匹配项之外的列表。 使用列表实体无法解析词干、复数形式和其他变体。 若要管理变体，请考虑使用带有可选文本语法的[模式](luis-concept-patterns.md#syntax-to-mark-optional-text-in-a-template-utterance)。

![列表实体](./media/luis-concept-entities/list-entity.png)

[教程](luis-quickstart-intent-and-list-entity.md)<br>
[实体的 JSON 响应示例](luis-concept-data-extraction.md#list-entity-data)

## <a name="patternany-entity"></a>Pattern.any 实体

Patterns.any 是一种长度可变的占位符，仅在模式的模板话语中使用，用于标记实体的起始和结束位置。  

在以下情况下，非常适合使用该实体：

* 实体的末尾可能与话语的其余文本相混淆。 
[教程](luis-tutorial-pattern.md)<br>
[实体的 JSON 响应示例](luis-concept-data-extraction.md#patternany-entity-data)

**示例**  
假设某个客户端应用程序需要基于标题搜索书籍，则 pattern.any 会提取完整的标题。 一个使用 pattern.any 进行这种书籍搜索的模板话语是 `Was {BookTitle} written by an American this year[?]`。 

在下表中，每行包含话语的两个版本。 最上面的话语是 LUIS 最初看到的话语，其中，书籍的开头和末尾部分并不明确。 最下面的话语表明在实施用于提取的模式后，LUIS 如何知道书籍的标题。 

|话语|
|--|
|已 Man 谁误认为。 他的妻子 Hat 和其他由美国编写本年度的临床故事？<br>《错把太太当成帽子的男人与其他医疗故事》是某位美国人在今年撰写的吗？|
|`Was Half Asleep in Frog Pajamas written by an American this year?`<br>`Was **Half Asleep in Frog Pajamas** written by an American this year?`|
|`Was The Particular Sadness of Lemon Cake: A Novel written by an American this year?`<br>`Was **The Particular Sadness of Lemon Cake: A Novel** written by an American this year?`|
|`Was There's A Wocket In My Pocket! written by an American this year?`<br>`Was **There's A Wocket In My Pocket!** written by an American this year?`|

## <a name="prebuilt-entity"></a>预生成实体

预生成实体是表示电子邮件、URL 和电话号码等常见概念的内置类型。 预生成的实体名称已保留。 添加到应用程序的[所有预生成实体](luis-prebuilt-entities.md)会返回到终结点查询中（如果在话语中发现了这些实体）。 

在以下情况下，非常适合使用该实体：

* 数据与语言区域性预生成实体支持的常见用例相匹配。 

随时可以添加和删除预生成实体。

![编号预生成实体](./media/luis-concept-entities/number-entity.png)

[教程](luis-tutorial-prebuilt-intents-entities.md)<br>
[实体的 JSON 响应示例](luis-concept-data-extraction.md#prebuilt-entity-data)

其中一些预生成实体是在开源[识别器 - 文本](https://github.com/Microsoft/Recognizers-Text)项目中定义的。 如果你的特定区域性或实体当前不受支持，请通过为项目做贡献来获得支持。 

### <a name="troubleshooting-prebuilt-entities"></a>故障排除预生成的实体

在 LUIS 门户中，如果预生成的实体标记而不是自定义实体，则将必须如何解决此问题几个的选择。

将添加到应用程序的预建的实体_始终_返回，即使查询文本应该提取自定义实体的相同的文本。 

#### <a name="change-tagged-entity-in-example-utterance"></a>示例查询文本中的标记更改实体

如果预生成的实体作为自定义实体是相同的文本或标记，选择示例查询文本中的文本，并更改标记查询文本。 

如果预生成的实体标记有更多文本或比自定义实体的令牌，则有两个选项如何修复此问题：

* [删除示例查询文本](#remove-example-utterance-to-fix-tagging)方法
* [删除预建的实体](#remove-prebuilt-entity-to-fix-tagging)方法

#### <a name="remove-example-utterance-to-fix-tagging"></a>删除示例查询文本以修复标记 

第一个选择是要删除的示例查询文本。 

1. 删除示例查询文本。
1. 重新训练的应用。 
1. 重新添加只是单词或短语，它是实体，这将被标记为预生成的实体，作为完整的示例查询文本。 词或短语仍将被标记为预生成的实体。 
1. 在选择的示例查询文本中的实体**意向**页上，并将更改为自定义实体和重新定型。 这应防止 LUIS 将此确切文本标记为任何使用该文本的示例查询文本中的预建实体。 
1. 将整个原始示例语音样本添加回意图。 自定义的实体应继续标记而不是预生成的实体。 如果未标记的自定义实体，你需要在查询文本中添加该文本的更多示例。

#### <a name="remove-prebuilt-entity-to-fix-tagging"></a>删除预建的实体，以修复标记

1. 从应用中删除预生成的实体。 
1. 上**意向**页上，将标记的示例查询文本中的自定义实体。
1. 将应用定型。
1. 返回到应用中添加预生成的实体并定型应用。 此修补程序假定预生成的实体不是复合实体的一部分。

## <a name="regular-expression-entity"></a>正则表达式实体 

正则表达式最适合用于原始话语文本。 不区分大小写，并忽略区域性变体。  完成字符级别而不是令牌级别的拼写检查更改后，会应用正则表达式匹配。 如果正则表达式过于复杂，例如使用了许多括号，则不能将表达式添加到模型。 使用部件，但不是所有[.NET 正则表达式](https://docs.microsoft.com/dotnet/standard/base-types/regular-expressions)库。 

在以下情况下，非常适合使用该实体：

* 数据的格式一致，并且其任何变体也是一致的。
* 正则表达式不需要 2 个级别以上的嵌套。 

![正则表达式实体](./media/luis-concept-entities/regex-entity.png)

[教程](luis-quickstart-intents-regex-entity.md)<br>
[实体的 JSON 响应示例](luis-concept-data-extraction.md#regular-expression-entity-data)<br>

## <a name="simple-entity"></a>简单实体 

简单实体是描述单个概念的泛型实体，通过机器学习的上下文习得。 由于简单实体采用概括性的名称，例如公司名称、产品名称或其他类别的名称，因此，在使用简单实体时，请添加一个[短语列表](luis-concept-feature.md)，以提升所用名称的信号。 

在以下情况下，非常适合使用该实体：

* 数据格式不一致，但指示相同的事物。 

![简单实体](./media/luis-concept-entities/simple-entity.png)

[教程](luis-quickstart-primary-and-secondary-data.md)<br/>
[实体响应示例](luis-concept-data-extraction.md#simple-entity-data)<br/>

## <a name="entity-limits"></a>实体限制

查看[限制](luis-boundaries.md#model-boundaries)，了解可添加到模型中的每种类型的实体的数量。

## <a name="if-you-need-more-than-the-maximum-number-of-entities"></a>如果所需实体数超过最大实体数 

您可能需要使用复合实体与实体角色结合使用。

复合实体表示整体的一部分。 例如，名为 PlaneTicketOrder 的复合实体可能具有子实体 Airline、Destination、DepartureCity、DepartureDate 和 PlaneTicketClass。

LUIS 还提供非机器学习的列表实体类型，可让 LUIS 应用指定固定的值列表。 请参阅 [LUIS 边界](luis-boundaries.md)参考内容，查看列表实体类型的限制。 

如果您已考虑过这些实体，并仍需要更多限制，请联系支持。 为此，请收集有关系统的详细信息，转到 [LUIS](luis-reference-regions.md#luis-website) 网站，然后选择“支持”。 如果所持 Azure 订阅包含支持服务，请与 [Azure 技术支持](https://azure.microsoft.com/support/options/)联系。 

## <a name="next-steps"></a>后续步骤

了解关于优良[话语](luis-concept-utterance.md)的概念。 

请参阅[添加实体](luis-how-to-add-entities.md)，了解如何将实体添加到 LUIS 应用的详细信息。
