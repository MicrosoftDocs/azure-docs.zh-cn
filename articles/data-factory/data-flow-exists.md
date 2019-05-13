---
title: Azure 数据工厂映射流存在数据转换
description: 如何检查现有行使用数据工厂映射数据的流使用 Exists 转换
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/30/2019
ms.openlocfilehash: b98b7afb21f2f50d44ba93ed793b6efb20f75164
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235989"
---
# <a name="mapping-data-flow-exists-transformation"></a>存在映射数据流转换

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

Exists 转换是阻止或允许数据行通过的行筛选转换。 Exists 转换类似于 ```SQL WHERE EXISTS``` 和 ```SQL WHERE NOT EXISTS```。 完成后存在转换从数据流生成的行将既包括列来自源 1 的值位于源 2 的所有行或源 2 中不存在。

![Exists 设置](media/data-flow/exists.png "exists 1")

为 Exists 选择第二个源，以便数据流可以将流 1 的值与流 2 进行比较。

从源 1 和源 2 中选择对其值进行 Exists 或 Not Exists 检查的列。

## <a name="multiple-exists-conditions"></a>多个存在条件

旁边 Exists 在列条件中的每个行，您会发现 + 符号可用时悬停在到达行。 这样，您可以添加多个行 Exists 条件。 每个其他条件是"和"。

## <a name="custom-expression"></a>自定义表达式

![存在自定义设置](media/data-flow/exists1.png "存在自定义")

可以单击"自定义表达式"改为创建自由格式表达式作为你存在或不存在的条件。 选中此复选框将允许您作为一项条件表达式中的类型。

## <a name="next-steps"></a>后续步骤

类似的转换[查找](data-flow-lookup.md)并[加入](data-flow-join.md)。
