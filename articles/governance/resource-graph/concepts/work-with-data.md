---
title: 处理大型数据集
description: 了解使用 Azure Resource Graph 时如何获取和控制大型数据集。
author: DCtheGeek
ms.author: dacoulte
ms.date: 04/01/2019
ms.topic: conceptual
ms.service: resource-graph
manager: carmonm
ms.openlocfilehash: ff9513418857562408c162533c48f6495b1f83c4
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65137863"
---
# <a name="working-with-large-azure-resource-data-sets"></a>处理大型 Azure 资源数据集

Azure Resource Graph 旨在处理并获取 Azure 环境中资源的相关信息。 Resource Graph 加快了获取此类数据的速度，即使在查询数千条记录，也不例外。 Resource Graph 提供了多个大型数据集处理选项。

## <a name="data-set-result-size"></a>数据集结果大小

默认情况下，Resource Graph 限制任何查询都只能返回 100 条记录。 这项控制措施可保护用户和服务不受会生成大型数据集的意外查询影响。 当客户尝试通过查询来按照能满足自己特定需求的方式查找和筛选资源时，这种情况最为常见。 这项控制措施不同于使用 [top](/azure/kusto/query/topoperator) 或 [limit](/azure/kusto/query/limitoperator) Azure 数据资源管理器语言运算符来限制结果。

> [!NOTE]
> 使用时**第一个**，建议对结果进行排序的至少一个包含的列`asc`或`desc`。 而不进行排序，返回的结果是随机的并且不重复。

通过与 Resource Graph 交互的所有方法，都可以替代默认限制。 下面的示例展示了如何将数据集大小限制更改为 200：

```azurecli-interactive
az graph query -q "project name | order by name asc" --first 200 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "project name | order by name asc" -First 200
```

在 [REST API](/rest/api/azureresourcegraph/resources/resources) 中，控制措施是 $top，它属于 QueryRequestOptions。

最具限制性的控制措施将胜出。 例如，如果查询使用 top 或 limit 运算符，并生成多于 First 的记录，那么返回的记录数上限等于 First。 同样，如果 top 或 limit 小于 First，那么返回的记录集小于 top 或 limit 配置的值。

First 当前的最大允许值是 5000。

## <a name="skipping-records"></a>跳过记录

下一个大型数据集处理选项是 Skip 控制措施。 通过这项控制措施，查询可以在返回结果之前跳过或略过定义的记录数。 Skip 适用于以一种有意义的方式对结果进行排序的查询，查询意图是在结果集中间某位置处获取记录。 如果所需的结果位于返回数据集的末尾，更高效的做法是使用不同的排序配置，并从数据集顶部检索结果。

> [!NOTE]
> 使用时**跳过**，建议对结果进行排序的至少一个包含的列`asc`或`desc`。 而不进行排序，返回的结果是随机的并且不重复。

下面的示例展示了如何跳过查询生成的前 10 条记录，改从第 11 条记录开始返回结果集：

```azurecli-interactive
az graph query -q "project name | order by name asc" --skip 10 --output table
```

```azurepowershell-interactive
Search-AzGraph -Query "project name | order by name asc" -Skip 10
```

在 [REST API](/rest/api/azureresourcegraph/resources/resources) 中，控制措施是 $skip，它属于 QueryRequestOptions。

## <a name="paging-results"></a>分页结果

需要中断结果集分成较小的集，用于处理的记录，或者因为结果集将超过允许的值的最大_1000年_返回记录，使用分页。 [REST API](/rest/api/azureresourcegraph/resources/resources) QueryResponse 提供了指明结果集已被拆分的值：resultTruncated 和 $skipToken。
resultTruncated 是布尔值，用于指示使用者返回的响应中是否还有其他记录。 如果 count 属性小于 totalRecords 属性，也可以确定此条件。 totalRecords 定义匹配查询的记录数。

如果 resultTruncated 为 true，便会在响应中设置 $skipToken 属性。 此值与相同的查询值及订阅值一起使用，以获取与查询匹配的下一个记录集。

下面的示例演示如何**跳过**第一个 3000 记录并返回**第一个**1000 条记录后这些跳过使用 Azure CLI 和 Azure PowerShell:

```azurecli-interactive
az graph query -q "project id, name | order by id asc" --first 1000 --skip 3000
```

```azurepowershell-interactive
Search-AzGraph -Query "project id, name | order by id asc" -First 1000 -Skip 3000
```

> [!IMPORTANT]
> 查询必须投射 ID 字段，这样分页才能生效。 如果查询中缺少，响应中不会包含 **$skipToken**。

有关示例，请参阅 REST API 文档中的[下一页查询](/rest/api/azureresourcegraph/resources/resources#next_page_query)。

## <a name="next-steps"></a>后续步骤

- 请参阅[初学者查询](../samples/starter.md)中使用中的语言
- 请参阅[高级查询](../samples/advanced.md)中的高级使用
- 了解如何[浏览资源](explore-resources.md)