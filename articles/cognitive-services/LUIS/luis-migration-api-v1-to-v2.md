---
title: v1 到 v2 API 迁移
titleSuffix: Azure Cognitive Services
description: 版本 1 终结点和创作语言理解 Api 已弃用。 使用此指南了解如何迁移至第 2 版终结点和创作 API。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 04/02/2019
ms.author: diberry
ms.openlocfilehash: a4e176552fcbab9d54da696439c7e46750be3944
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522368"
---
# <a name="api-v1-to-v2-migration-guide-for-luis-apps"></a>LUIS 应用的 API v1 到 v2 迁移指南
第 1 版[终结点](https://aka.ms/v1-endpoint-api-docs)和[创作](https://aka.ms/v1-authoring-api-docs) API 已弃用。 使用此指南学习如何迁移至第 2 版[终结点](https://go.microsoft.com/fwlink/?linkid=2092356)和[创作](https://go.microsoft.com/fwlink/?linkid=2092087) API。 

## <a name="new-azure-regions"></a>新的 Azure 区域
LUIS 为 LUIS API 提供新的[区域](https://aka.ms/LUIS-regions)。 LUIS 提供区域组的另一个门户。 必须在要用于查询的区域中编写应用程序。 应用程序不会自动迁移区域。 若要在新区域中使用应用，请从一个区域中将其导出，再将其导入到另一个区域。

## <a name="authoring-route-changes"></a>创作路由的更改
创作 API 路由从使用 prog 路由改为使用 api 路由。


| version | 路由 |
|--|--|
|第|/luis/v1.0/prog/apps|
|2|/luis/api/v2.0/apps|


## <a name="endpoint-route-changes"></a>终结点路由的更改
终结点 API 有新的查询字符串参数，以及不同的响应。 如果详细标志为 true，包括 topScoringIntent 在内的所有意向（不考虑分数）都将返回到一个名为意向的数组中。

| version | GET 路由 |
|--|--|
|第|/luis/v1/application?ID={appId}&q={q}|
|2|/luis/v2.0/apps/{appId}?q={q}[&timezoneOffset][&verbose][&spellCheck][&staging][&bing-spell-check-subscription-key][&log]|


v1 终结点成功响应：
```json
{
  "odata.metadata":"https://dialogice.cloudapp.net/odata/$metadata#domain","value":[
    {
      "id":"bccb84ee-4bd6-4460-a340-0595b12db294","q":"turn on the camera","response":"[{\"intent\":\"OpenCamera\",\"score\":0.976928055},{\"intent\":\"None\",\"score\":0.0230718572}]"
    }
  ]
}
```

v2 终结点成功响应：
```json
{
  "query": "forward to frank 30 dollars through HSBC",
  "topScoringIntent": {
    "intent": "give",
    "score": 0.3964121
  },
  "entities": [
    {
      "entity": "30",
      "type": "builtin.number",
      "startIndex": 17,
      "endIndex": 18,
      "resolution": {
        "value": "30"
      }
    },
    {
      "entity": "frank",
      "type": "frank",
      "startIndex": 11,
      "endIndex": 15,
      "score": 0.935219169
    },
    {
      "entity": "30 dollars",
      "type": "builtin.currency",
      "startIndex": 17,
      "endIndex": 26,
      "resolution": {
        "unit": "Dollar",
        "value": "30"
      }
    },
    {
      "entity": "hsbc",
      "type": "Bank",
      "startIndex": 36,
      "endIndex": 39,
      "resolution": {
        "values": [
          "BankeName"
        ]
      }
    }
  ]
}
```

## <a name="key-management-no-longer-in-api"></a>不再在 API 中执行密钥管理
订阅终结点密钥 API 已弃用并返回“410 不存在”。

| version | 路由 |
|--|--|
|第|/luis/v1.0/prog/subscriptions|
|第|/luis/v1.0/prog/subscriptions/{subscriptionKey}|

在 Azure 门户中生成了 Azure [终结点密钥](luis-how-to-azure-subscription.md)。 可在[发布](luis-how-to-azure-subscription.md)页上将密钥分配至 LUIS 应用。 不需要知道实际的密钥值。 LUIS 使用订阅名称来进行分配。 

## <a name="new-versioning-route"></a>新的版本控制路由
[版本](luis-how-to-manage-versions.md)中现包含 v2 模型。 版本名称是路由中的 10 个字符。 默认版本为“0.1”。

| version | 路由 |
|--|--|
|第|/luis/v1.0/prog/apps/{appId}/entities|
|2|/luis/api/v2.0/apps/{appId}/versions/{versionId}/entities|

## <a name="metadata-renamed"></a>重命名元数据
一些返回 LUIS 元数据的 API 具有新名称。

| v1 路由名称 | v2 路由名称 |
|--|--|
|PersonalAssistantApps |assistants|
|applicationcultures|cultures|
|applicationdomains|domains|
|applicationusagescenarios|usagescenarios|


## <a name="sample-renamed-to-suggest"></a>“示例”已重命名为“建议”
LUIS 会从现有[终结点话语](luis-how-to-review-endpoint-utterances.md)中推荐能增强模型的话语。 在前一版本中，此功能名为“样本”。 在新版本中，其名称从“样本”改为“建议”。 在 LUIS 网站上名为[查看终结点话语](luis-how-to-review-endpoint-utterances.md)。

| version | 路由 |
|--|--|
|第|/luis/v1.0/prog/apps/{appId}/entities/{entityId}/sample|
|第|/luis/v1.0/prog/apps/{appId}/intents/{intentId}/sample|
|2|/luis/api/v2.0/apps/{appId}/versions/{versionId}/entities/{entityId}/suggest|
|2|/luis/api/v2.0/apps/{appId}/versions/{versionId}/intents/{intentId}/suggest|


## <a name="create-app-from-prebuilt-domains"></a>从预生成的域创建应用
[预生成的域](luis-how-to-use-prebuilt-domains.md)提供一个预定义的域模型。 借助预生成的域，可快速开发常见域的 LUIS 应用程序。 使用此 API，可基于预生成的域新建应用。 响应内容为新的应用 ID。

|v2 路由|谓词|
|--|--|
|/luis/api/v2.0/apps/customprebuiltdomains  |获取、发布|
|/luis/api/v2.0/apps/customprebuiltdomains/{culture}  |get|

## <a name="importing-1x-app-into-2x"></a>将 1.x 应用导入至 2.x
导出的 1.x 应用的 JSON 有某些区域需要在导入到 [LUIS][LUIS] 2.0 之前进行更改。 

### <a name="prebuilt-entities"></a>预生成的实体 
已更改[预生成的实体](luis-prebuilt-entities.md)。 请确保使用 V2 预生成实体。 这包括使用 [datetimeV2](luis-reference-prebuilt-datetimev2.md) 而不是 datetime。 

### <a name="actions"></a>操作
操作属性不再有效。 应该为空 

### <a name="labeled-utterances"></a>标记的话语
V1 允许标记的话语在字词或短语的开头或末尾包含空格。 删除了空格。 

## <a name="common-reasons-for-http-response-status-codes"></a>HTTP 响应状态代码的常见原因
请参阅 [LUIS API 响应代码](luis-reference-response-codes.md)。

## <a name="next-steps"></a>后续步骤

使用 v2 API 文档更新对 LUIS [终结点](https://go.microsoft.com/fwlink/?linkid=2092356)和[创作](https://go.microsoft.com/fwlink/?linkid=2092087) API 的现有 REST 调用。 

[LUIS]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions
