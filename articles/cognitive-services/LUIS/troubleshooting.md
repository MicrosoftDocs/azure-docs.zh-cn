---
title: 常见问题 (FAQ)
titleSuffix: Azure Cognitive Services
description: 本文包含有关语言理解 (LUIS) 的常见问题的解答。
author: diberry
manager: nitinme
ms.custom: seodec18
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: fc05f285b286bd0f036c583c154a40cf64e144d4
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522034"
---
# <a name="language-understanding-frequently-asked-questions-faq"></a>语言理解常见问题解答 (FAQ)

本文包含有关语言理解 (LUIS) 的常见问题的解答。

<a name="luis-authoring"></a>

## <a name="authoring"></a>创作

### <a name="what-are-the-luis-best-practices"></a>LUIS 最佳做法是什么？
从[创作周期](luis-concept-app-iteration.md)开始，然后阅读[最佳做法](luis-concept-best-practices.md)。

### <a name="what-is-the-best-way-to-start-building-my-app-in-luis"></a>若要开始在 LUIS 中生成应用，最佳方式是什么？

生成应用的最佳方式是使用[增量过程](luis-concept-app-iteration.md)。

### <a name="what-is-a-good-practice-to-model-the-intents-of-my-app-should-i-create-more-specific-or-more-generic-intents"></a>若要对应用的意向建模，什么做法较好？ 是应创建较具体的意向还是较宽泛的意向？

选择的意向不应太宽泛，免得互相重叠，也不应太具体，免得 LUIS 难以区分相似的意向。 进行 LUIS 建模时，一项最佳做法是创建差异性的具体意向。

### <a name="is-it-important-to-train-the-none-intent"></a>训练 None 意向是否很重要？

是的，在向其他意向添加更多标签时，不妨使用更多表述训练 **None** 意向。 每次向某个意向添加 10 个标签时，可以向 **None** 添加 1 到 2 个标签，这种比例比较适当。 此比例可以提高 LUIS 的区分能力。

### <a name="how-can-i-correct-spelling-mistakes-in-utterances"></a>如何更正表述中的拼写错误？

请参阅[必应拼写检查 API V7](luis-tutorial-bing-spellcheck.md) 教程。 LUIS 强制实施必应拼写检查 API V7 施加的限制。

### <a name="how-do-i-edit-my-luis-app-programmatically"></a>如何以编程方式编辑 LUIS 应用？
若要以编程方式编辑 LUIS 应用，请使用[创作 API](https://go.microsoft.com/fwlink/?linkid=2092087)。 请参阅[调用 LUIS 创作 API](./luis-quickstart-node-add-utterance.md) 和[使用 Node.js 以编程方式生成 LUIS 应用](./luis-tutorial-node-import-utterances-csv.md)，通过示例了解如何调用创作 API。 创作 API 要求使用[创作密钥](luis-concept-keys.md#authoring-key)而不是终结点密钥。 编程创作最多允许每个月进行 1,000,000 次调用，每秒处理五个事务。 若要详细了解可与 LUIS 配合使用的密钥，请参阅[管理密钥](./luis-concept-keys.md)。

### <a name="where-is-the-pattern-feature-that-provided-regular-expression-matching"></a>提供了正则表达式匹配的“模式”功能位于何处？
以前的 **“模式”功能**目前已弃用，代之以现在的**[模式](luis-concept-patterns.md)** 功能。

### <a name="how-do-i-use-an-entity-to-pull-out-the-correct-data"></a>如何使用实体来拉取出正确的数据？
请参阅[实体](luis-concept-entity-types.md)和[数据提取](luis-concept-data-extraction.md)。

### <a name="should-variations-of-an-example-utterance-include-punctuation"></a>示例表述的变体是否应包括标点？
请将不同的变体作为示例表述添加到意向，或者使用[忽略标点的语法](luis-concept-patterns.md#pattern-syntax)添加示例表述的模式。

### <a name="does-luis-currently-support-cortana"></a>LUIS 目前是否支持 Cortana？

Cortana 预构建应用已于 2017 年弃用。 它们不再受支持。

### <a name="how-do-i-transfer-ownership-of-a-luis-app"></a>如何转让 LUIS 应用的所有权？
若要将某个 LUIS 应用转让给另一 Azure 订阅，请导出该 LUIS 应用，然后使用新帐户来导入它。 在进行调用的客户端应用程序中更新 LUIS 应用 ID。 新应用返回的 LUIS 分数与原始应用返回的可能稍有不同。

### <a name="a-prebuilt-entity-is-tagged-in-an-example-utterance-instead-of-my-custom-entity-how-do-i-fix-this"></a>预生成的实体标记中的示例查询文本，而不是我的自定义实体中。 如何修复此问题? 

请参阅[故障排除预生成的实体](luis-concept-entity-types.md#troubleshooting-prebuilt-entities)。

### <a name="i-tried-to-import-an-app-or-version-file-but-i-got-an-error-what-happened"></a>我尝试导入的应用程序或版本文件，但收到错误，发生了什么情况？ 

详细了解[版本导入错误](luis-how-to-manage-versions.md#import-errors)并[应用程序导入错误](luis-how-to-start-new-app.md#import-errors)。

<a name="luis-collaborating"></a>

## <a name="collaborating"></a>协作

### <a name="how-do-i-give-collaborators-access-to-luis-with-azure-active-directory-azure-ad-or-role-based-access-control-rbac"></a>如何使用 Azure Active Directory (Azure AD) 或基于角色的访问控制 (RBAC) 来为协作者提供对 LUIS 的访问权限？

请参阅 [Azure Active Directory 资源](luis-how-to-collaborate.md#azure-active-directory-resources)和 [Azure Active Directory 租户用户](luis-how-to-collaborate.md#azure-active-directory-tenant-user)来了解如何为协作者提供访问权限。 

<a name="luis-endpoint"></a>

## <a name="endpoint"></a>终结点

### <a name="my-endpoint-query-returned-unexpected-results-what-should-i-do"></a>我的终结点查询返回了意外的结果。 我该怎么办？

意外的查询预测结果基于已发布模型的状态。 若要更正模型，可能需要更改模型、进行训练，然后重新发布。 

更正模型从[主动学习](luis-how-to-review-endpoint-utterances.md)开始。

可以通过更新[应用程序版本设置 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) 来删除非确定性训练，以便使用所有训练数据。

有关其他提示，请查看[最佳做法](luis-concept-best-practices.md)。 

### <a name="why-does-luis-add-spaces-to-the-query-around-or-in-the-middle-of-words"></a>为什么 LUIS 向查询添加空格时，会将其添加到单词的周围或中间？
LUIS 根据[区域性](luis-language-support.md#tokenization)将表述[标记化](luis-glossary.md#token)。 原始值和标记化值均可用于[数据提取](luis-concept-data-extraction.md#tokenized-entity-returned)。

### <a name="how-do-i-create-and-assign-a-luis-endpoint-key"></a>如何创建并分配 LUIS 终结点密钥？
根据[服务](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/)级别在 Azure 中[创建终结点密钥](luis-how-to-azure-subscription.md)。 在**[密钥和终结点](luis-how-to-azure-subscription.md)** 页上[分配密钥](luis-how-to-azure-subscription.md)。 此操作没有相应的 API。 然后，必须更改针对此终结点的 HTTP 请求才能[使用新终结点密钥](luis-concept-keys.md#use-endpoint-key-in-query)。

### <a name="how-do-i-interpret-luis-scores"></a>如何解释 LUIS 分数？
系统应该使用最高得分意向，不管其值如何。 例如，分数低于 0.5（不到 50%）不一定意味着 LUIS 的置信度低。 提供更多训练数据有助于提高最可能意向的[分数](luis-concept-prediction-score.md)。

### <a name="why-dont-i-see-my-endpoint-hits-in-my-apps-dashboard"></a>为何在应用的“仪表板”中看不到终结点命中数？
应用的“仪表板”中的终结点总命中数会定期更新，但 Azure 门户中与 LUIS 终结点密钥相关联的指标的更新频率更高。

如果在“仪表板”中看不到更新的终结点命中数，请登录到 Azure 门户，找到与 LUIS 终结点密钥相关联的资源，然后打开“指标”以选择“总调用数”指标。 如果将终结点密钥用于多个 LUIS 应用，则 Azure 门户中的指标会显示使用该密钥的所有 LUIS 应用进行的调用的聚合数。

### <a name="is-there-a-powershell-command-get-to-the-endpoint-quota"></a>是否有 PowerShell 命令进入终结点？

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

可以使用 PowerShell 命令查看终结点配额：

```powershell
Get-AzCognitiveServicesAccountUsage -ResourceGroupName <your-resource-group> -Name <your-resource-name>
``` 

### <a name="my-luis-app-was-working-yesterday-but-today-im-getting-403-errors-i-didnt-change-the-app-how-do-i-fix-it"></a>我的 LUIS 应用昨天还可以正常使用，但今天却出现 403 错误。 我没有更改应用。 如何解决问题？
根据下一常见问题解答中的[说明](#how-do-i-create-and-assign-a-luis-endpoint-key)创建一个 LUIS 终结点密钥，然后将其分配给应用。 然后，必须更改针对此终结点的 HTTP 请求才能[使用新终结点密钥](luis-concept-keys.md#use-endpoint-key-in-query)。

### <a name="how-do-i-secure-my-luis-endpoint"></a>如何保护 LUIS 终结点安全？
请参阅[保护终结点安全](luis-concept-security.md#securing-the-endpoint)。

## <a name="working-within-luis-limits"></a>在 LUIS 限制内操作

### <a name="what-is-the-maximum-number-of-intents-and-entities-that-a-luis-app-can-support"></a>LUIS 应用可以支持的最大意向和实体数是多少？
请查看[边界](luis-boundaries.md)参考文档。

### <a name="i-want-to-build-a-luis-app-with-more-than-the-maximum-number-of-intents-what-should-i-do"></a>我希望生成一个 LUIS 应用，其意向数超出最大意向数。 我该怎么办？

请参阅[意向最佳做法](luis-concept-intent.md#if-you-need-more-than-the-maximum-number-of-intents)。

### <a name="i-want-to-build-an-app-in-luis-with-more-than-the-maximum-number-of-entities-what-should-i-do"></a>我希望在 LUIS 中生成一个应用，其实体数超出最大实体数。 我该怎么办？

请参阅[实体最佳做法](luis-concept-entity-types.md#if-you-need-more-than-the-maximum-number-of-entities)。

### <a name="what-are-the-limits-on-the-number-and-size-of-phrase-lists"></a>短语列表的数目和大小限制是多少？
若要了解[短语列表](./luis-concept-feature.md)的最大长度，请查看[边界](luis-boundaries.md)参考文档。

### <a name="what-are-the-limits-on-example-utterances"></a>示例表述有哪些限制？
请查看[边界](luis-boundaries.md)参考文档。

## <a name="testing-and-training"></a>测试和训练

### <a name="i-see-some-errors-in-the-batch-testing-pane-for-some-of-the-models-in-my-app-how-can-i-address-this-problem"></a>我在批量测试窗格中看到应用中的某些模块出现的某些错误。 如何解决此问题？

这些错误表明标签与模型预测之间存在一些差异。 若要解决此问题，请完成以下任务中的一个或全部两个：
* 添加更多标签，以便 LUIS 提高意向之间的差异性。
* 添加短语列表功能，引入专业领域的词汇，以便 LUIS 加快学习速度。

请参阅[批量测试](luis-tutorial-batch-testing.md)教程。

### <a name="when-an-app-is-exported-then-reimported-into-a-new-app-with-a-new-app-id-the-luis-prediction-scores-are-different-why-does-this-happen"></a>将某个应用导出之后又重新导入到新应用中时（使用新的应用 ID），LUIS 预测分数会有所不同。 为何发生这种情况？

请参阅[同一应用的不同副本之间的预测差异](luis-concept-prediction-score.md#review-intents-with-similar-scores)。

### <a name="some-utterances-go-to-the-wrong-intent-after-i-made-changes-to-my-app-the-issue-seems-to-disappear-at-random-how-do-i-fix-it"></a>我对应用进行更改之后，一些表述会转到错误意图。 此问题似乎随机消失。 如何解决问题？ 

请参阅[使用所有数据进行训练](luis-how-to-train.md#train-with-all-data)。

## <a name="app-publishing"></a>应用发布

### <a name="what-is-the-tenant-id-in-the-add-a-key-to-your-app-window"></a>“向应用添加密钥”窗口中的租户 ID 是什么？
在 Azure 中，租户代表与服务关联的客户或组织。 若要在 Azure 门户的“目录 ID”框中查找租户 ID，请选择“Azure Active Directory” > “管理” > “属性”。

![Azure 门户中的租户 ID](./media/luis-manage-keys/luis-assign-key-tenant-id.png)

<a name="why-are-there-more-subscription-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>
<a name="why-are-there-more-endpoint-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>


### <a name="why-are-there-more-endpoint-keys-assigned-to-my-app-than-i-assigned"></a>为什么分配给我应用的终结点密钥多于我分配的？
为方便起见，每个 LUIS 应用在终结点列表中都有创作/初学者密钥。 此密钥仅允许命中终结点几次，因此你可以试用 LUIS。  

如果应用在 LUIS 正式发布 (GA) 之前已存在，则会自动分配订阅中的 LUIS 终结点密钥。 这样做是为了方便 GA 迁移。 Azure 门户中的任何新 LUIS 终结点密钥都_不_会自动分配给 LUIS。

## <a name="key-management"></a>密钥管理

### <a name="how-do-i-know-what-key-i-need-where-i-get-it-and-what-i-do-with-it"></a>如何了解我需要什么密钥，从哪里获取它，以及使用它来做什么？ 

若要了解[创作密钥](luis-how-to-account-settings.md)与[终结点预测密钥](luis-how-to-azure-subscription.md)之间的差异，请参阅 [LUIS 中的创作密钥和查询预测终结点密钥](luis-concept-keys.md)。 

### <a name="i-got-an-error-about-being-out-of-quota-how-do-i-fix-it"></a>我收到有关超出配额的错误。 如何解决问题？ 

请参阅[当密钥超过定价层用量时如何修复“超出配额”错误](luis-how-to-azure-subscription.md##how-to-fix-out-of-quota-errors-when-the-key-exceeds-pricing-tier-usage)来了解详细信息。

### <a name="i-need-to-handle-more-endpoint-queries-how-do-i-do-that"></a>我需要处理更多的终结点查询。 我该怎么做？ 

请参阅[当密钥超过定价层用量时如何修复“超出配额”错误](luis-how-to-azure-subscription.md##how-to-fix-out-of-quota-errors-when-the-key-exceeds-pricing-tier-usage)来了解详细信息。



## <a name="app-management"></a>应用管理

### <a name="how-do-i-download-a-log-of-user-utterances"></a>如何下载用户表述的日志？
默认情况下，LUIS 应用会记录用户提供的表述。 若要下载用户发送至 LUIS 应用的表述的日志，请转到“我的应用”，然后选择此应用。 在上下文工具栏中，选择“导出终结点日志”。 日志格式化为逗号分隔值 (CSV) 文件。

### <a name="how-can-i-disable-the-logging-of-utterances"></a>如何禁用表述的日志记录？
若要关闭用户表述的日志记录，可以在客户端应用程序用来查询 LUIS 的终结点 URL 中设置 `log=false`。 但是，关闭日志记录会禁用 LUIS 应用提供表述建议或改进基于[主动学习](luis-concept-review-endpoint-utterances.md#what-is-active-learning)的性能的功能。 如果因为数据隐私顾虑而设置 `log=false`，则不能从 LUIS 下载这些用户表述的记录，也不能使用这些表述来改进应用。

日志记录是表述的唯一存储方式。

### <a name="why-dont-i-want-all-my-endpoint-utterances-logged"></a>为什么不需要记录所有的终结点表述？
如果使用日志进行预测分析，请勿在日志中捕获测试性表述。

## <a name="data-management"></a>数据管理

### <a name="can-i-delete-data-from-luis"></a>能否从 LUIS 中删除数据？

* 可以随意删除用于训练 LUIS 的示例表述。 如果从 LUIS 应用中删除某个示例表述，则会将其从 LUIS Web 服务中删除，导致其无法导出。
* 可以从 LUIS 在“查看终结点表述”页中建议的用户表述列表中删除表述。 从此列表中删除表述可以防止系统再将其作为建议提出来，但不会将其从日志中删除。
* 如果删除某个帐户，则会删除所有应用及其示例表述和日志。 数据可以保留在服务器上 60 天，然后就会被永久删除。

### <a name="how-does-microsoft-manage-data-i-send-to-luis"></a>Microsoft 如何管理我发送给 LUIS 的数据？

[信任中心](https://www.microsoft.com/trustcenter)解释了我们的承诺以及用于在 Azure 服务中进行数据管理和访问的选项。

## <a name="language-and-translation-support"></a>语言和翻译支持

### <a name="i-have-an-app-in-one-language-and-want-to-create-a-parallel-app-in-another-language-what-is-the-easiest-way-to-do-so"></a>我已经有了某种语言的应用，希望创建一个采用另一种语言的并行应用， 最简单的方法是什么？
1. 导出应用。
2. 将已导出应用的 JSON 文件中标记的表述翻译成目标语言。
3. 可能需要更改意向和实体的名称，或者保留其不变。
4. 最后是导入应用，这样就有了一个采用目标语言的 LUIS 应用。

## <a name="app-notification"></a>应用通知

### <a name="why-did-i-get-an-email-saying-im-almost-out-of-quota"></a>为什么我收到一封电子邮件，说我的配额几乎已用光？
创作/初学者密钥每个月只允许 1000 个终结点查询。 请创建一个 LUIS 终结点密钥（免费或付费），在进行终结点查询时使用该密钥。 如果通过机器人或另一客户端应用程序进行终结点查询，则需在其中更改 LUIS 终结点密钥。

## <a name="bots"></a>机器人

### <a name="my-luis-bot-isnt-working-what-do-i-do"></a>我的 LUIS 机器人不能正常工作。 我该怎么办？

第一个问题是如果问题和 LUIS 有关或发生在 LUIS 中间件以外，则隔离。 

#### <a name="resolve-issue-in-luis"></a>在 LUIS 中解决问题
从 [LUIS 终结点](luis-get-started-create-app.md#query-the-endpoint-with-a-different-utterance)传递相同的言语到 LUIS。 如果收到错误，请在 LUIS 中解决该问题直到不再返回错误。 常见错误包括：

* `Out of call volume quota. Quota will be replenished in <time>.` - 此问题表明需要从创作密钥更改为[终结点密钥](luis-how-to-azure-subscription.md)，或者需要更改[服务层级](luis-how-to-azure-subscription.md#change-pricing-tier)。 

#### <a name="resolve-issue-in-azure-bot-service"></a>在 Azure 机器人服务中解决问题

如果在使用 Azure 机器人服务时遇到“网上聊天中的测试”返回 `Sorry, my bot code is having an issue` 问题，请查看日志：

1. 在 Azure 门户中，对于机器人，从“机器人管理”部分选择“生成”。
1. 打开联机代码编辑器。 
1. 在顶部的蓝色导航栏中，选择智能机器人名称（从右起的第二项）。
1. 在生成的下拉列表中，选择“打开 Kudu 控制台”。
1. 选择“LogFiles”，然后选择“应用程序”。 查看所有日志文件。 如果应用程序文件夹中没有错误，请查看“LogFiles”下的所有日志文件。 
1. 如果使用经过编译的语言（如 C#），请记住重新生成项目。

> [!Tip] 
> 控制台还可以安装包。 

#### <a name="resolve-issue-while-debugging-on-local-machine-with-bot-framework"></a>使用 Bot Framework 在本地计算机上进行调试时解决问题。 

若要了解有关本地调试机器人的详细信息，请参阅[调试机器人](https://docs.microsoft.com/azure/bot-service/bot-service-debug-bot?view=azure-bot-service-4.0)。

## <a name="integrating-luis"></a>集成 LUIS

### <a name="where-is-my-luis-app-created-during-the-azure-web-app-bot-subscription-process"></a>在 Azure Web 应用机器人订阅过程中创建的 LUIS 应用位于何处？
如果选择一个 LUIS 模板，然后在模板窗格中选择“选择”按钮，则左侧窗格在更改后会包括模板类型，并询问在什么区域创建 LUIS 模板。 不过，Web 应用机器人过程不会创建 LUIS 订阅。

![LUIS 模板 Web 应用机器人区域](./media/luis-faq/web-app-bot-location.png)

### <a name="what-luis-regions-support-bot-framework-speech-priming"></a>什么 LUIS 区域支持 Bot Framework 语音启动？
仅中部（美国）实例中的 LUIS 应用支持[语音启动](https://docs.microsoft.com/bot-framework/bot-service-manage-speech-priming)。

## <a name="api-programming-strategies"></a>API 编程策略

### <a name="how-do-i-programmatically-get-the-luis-region-of-a-resource"></a>如何以编程方式获取资源的 LUIS 区域？ 

通过 C# 或 Node.Js 使用 LUIS 示例以编程方式[查找区域](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/documentation-samples/find-region)。 

## <a name="luis-service"></a>LUIS 服务

### <a name="is-language-understanding-luis-available-on-premises-or-in-private-cloud"></a>是否可以在本地或私有云中使用语言理解 (LUIS)？

是，如果具有计量使用情况的必要连接，则可以为这些方案使用 LUIS [容器](luis-container-howto.md)。 

## <a name="migrating-to-the-next-version"></a>迁移到下一个版本

### <a name="how-do-i-migrate-to-preview-v3-api"></a>如何迁移预览 V3 API？ 

请参阅[API v2 到 v3 迁移指南为 LUIS 应用](luis-migration-api-v3.md)

## <a name="build-2019-conference-announcements"></a>Build 2019 大会公告

在生成 2019年大会上发布了以下功能：

* [V3 API 迁移指南的预览](luis-migration-api-v3.md)
* [改进的分析仪表板](luis-how-to-use-dashboard.md)
* [改进了预生成的域](luis-reference-prebuilt-domains.md) 

## <a name="next-steps"></a>后续步骤

若要了解有关 LUIS 的详细信息，请参阅以下资源：
* [标记了 LUIS 的 Stack Overflow 问题](https://stackoverflow.com/questions/tagged/luis)
* [MSDN 语言理解智能服务 (LUIS) 论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=LUIS)