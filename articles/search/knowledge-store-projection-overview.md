---
title: 使用知识存储-Azure 搜索中的投影
description: 保存并分析你从 AI 索引管道在搜索以外的方案中使用的丰富的数据
manager: eladz
author: vkurpad
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: vikurpad
ms.custom: seomay2019
ms.openlocfilehash: 3ab5ffafd1b20eb0e3e453d3e730840baf9233e1
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65028360"
---
# <a name="working-with-projections-in-a-knowledge-store-in-azure-search"></a>使用 Azure 搜索中的知识存储中的投影

Azure 搜索来启用通过 AI 认知技能和编制索引的一部分的自定义技能的内容扩展。 正将结构添加到你的文档，并使搜索更高效。 在许多情况下，丰富的文档可用于搜索，例如，针对知识挖掘以外的方案。

投影和组成部分[知识 store （预览版）](knowledge-store-concept-intro.md)，是可以保存到物理存储空间，知识挖掘目的的丰富文档的视图。 投影可以"投影"你的数据到符合您的需求，保持关系，这样 Power BI 等工具可读取任何额外操作数据的形状。 

使用 Azure 表存储在行和列中存储的数据或 Azure Blob 存储中存储的 JSON 对象，可以表格，投影。 它被丰富时，可以定义数据的多个数据投影。 当你想以不同的方式调整各个用例的相同数据时，这很有用。 

知识应用商店支持两种类型的投影：

+ **表**：对于最好地表示为行和列的数据，可以使用表投影在表存储中定义架构化的形状或投影。 

+ **对象**:当您需要将数据和正的 JSON 表示形式时，以 blob 形式保存对象投影。

若要查看上下文中定义的投影，单步执行[如何知识 store 入门](knowledge-store-howto.md)

## <a name="projection-groups"></a>投影组

在某些情况下，将需要项目不同的形状以满足不同的目标中丰富的数据。 知识存储让你可以定义多个组的投影。 投影组具有以下主要特征有互斥性和关联性。

### <a name="mutually-exclusivity"></a>相互独占性

投影到一个组的所有内容都是独立于数据投影到其他投影组。 这意味着，您可以调整的方式不同，但在投影中的每个组中重复相同的数据。 

强制执行投影组中的一个约束是与投影组投影类型的有互斥性。 仅可以定义表投影或单个组中的对象投影。 如果你希望表和对象，定义的表，一个投影组和对象的第二个投影组。

### <a name="relatedness"></a>关联性

预计单个投影组中的所有内容都保留在数据中的关系。 关系基于生成的密钥，每个子节点将保留对父节点的引用。 关系不跨投影组和表或一个投影组中创建的对象具有与其他投影组中生成的数据没有关系。

## <a name="input-shaping"></a>输入定型
在正确的形状或结构中获取数据到有效密钥的使用，可能是表或对象。 形状或结构，根据您计划如何访问并使用它对数据的能力是关键功能作为公开**整形程序**技能组合内的技能。  

投影是更轻松地定义投影的架构匹配的扩充树中有一个对象时。 已更新[整形程序技能](cognitive-search-skill-shaper.md)允许您编写扩充树的不同节点中的对象和其父级下一个新的节点。 **整形程序**技能，可定义具有嵌套的对象的复杂类型。

新形状定义包含需要提取出的所有元素后，你现在可以使用此形状，作为用于你的投影源或另一项技巧的输入。

## <a name="table-projections"></a>表投影

因为这样可以更轻松地导入，我们建议使用 Power BI 的数据浏览表投影。 此外，表投影允许更改更改之间表关系的基数。 

可以投影单个文档索引中为多个表，保留关系。 投影到多个表时, 完成的形状将投影到每个表中，除非子节点的同一组中的另一个表的源。

### <a name="defining-a-table-projection"></a>定义表投影

定义中的表投影时`knowledgeStore`元素的您的技能以，首先，扩充树上的节点映射到表源。 此节点通常是的输出**整形程序**添加到列表的技能以生成您需要为表项目的特定形状的技能。 选择项目的节点可以进行划分成多个表的项目。 表定义是你想要项目的表的列表。 

每个表需要三个属性：

+ 表名：在 Azure 存储中的表的名称。

+ generatedKeyName:唯一标识该行的键列名称。

+ source：从扩充树节点来源您正从。 这通常是整形程序的输出，但可以是任何技能的输出。

下面是表投影的示例。

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
      
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [
            { "tableName": "MainTable", "generatedKeyName": "SomeId", "source": "/document/EnrichedShape" },
            { "tableName": "KeyPhrases", "generatedKeyName": "KeyPhraseId", "source": "/document/EnrichedShape/*/KeyPhrases/*" },
            { "tableName": "Entities", "generatedKeyName": "EntityId", "source": "/document/EnrichedShape/*/Entities/*" }
          ]
        },
        {
          "objects": [
            
          ]
        }
      ]
    }
}
```
在此示例中所示，关键短语和实体建模到不同的表，并且将包含返回到父级 (MainTable) 为每个行的引用。 

下图是参考中的 Caselaw 练习[如何开始使用知识 store](knowledge-store-howto.md)。 在方案中，一种情况具有多个观点，而每个观点丰富通过识别它所包含的实体，可以建模投影，如下所示。

![实体和表中的关系](media/knowledge-store-projection-overview/TableRelationships.png "表投影中的关系建模")

## <a name="object-projections"></a>对象投影

对象投影是可从任何节点提供的丰富化树的 JSON 表示形式。 在许多情况下，相同**整形程序**创建表投影的技能可用于生成的对象投影。 

```json
{
 
    "name": "your-skillset",
    "skills": [
      …your skills
      
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [
            {
              "storageContainer": "Reviews", 
              "format": "json", 
              "source": "/document/Review", 
              "key": "/document/Review/Id" 
            }
          ]
        }
      ]
    }
}
```

生成的对象投影需要少量特定于对象的属性：

+ storageContainer:将在其中保存对象容器
+ source：是投影的根的扩充树节点的路径
+ 密钥：一个表示要存储的对象的唯一键的路径。 它将用于在容器中创建的 blob 的名称。

## <a name="projection-lifecycle"></a>投影生命周期

您的计划具有一个生命周期，在您的数据源绑定到源数据。 在你的数据被更新和重新创建索引时，您的计划会更新到您的计划都与您的数据源中的数据最终保持一致的结果。 投影继承已为你的索引配置的删除策略。 

## <a name="using-projections"></a>使用投影

运行索引器后，可以读取容器或通过投影指定的表中的预测的数据。 

对于分析，Power BI 中的探索是设置 Azure 表存储作为数据源一样简单。 利用中的关系对数据，可以非常轻松地创建一系列可视化效果。

或者，如果您需要在数据科学管道中使用丰富的数据，你可以[将数据从 blob 加载到 Pandas 数据帧](../machine-learning/team-data-science-process/explore-data-blob.md)。

最后，如果需要从知识存储中导出数据，Azure 数据工厂提供连接器来导出数据，然后移入所选的数据库。 

## <a name="next-steps"></a>后续步骤

下一步中，创建使用示例数据和说明在第一个知识存储。

> [!div class="nextstepaction"]
> [如何创建知识库存储](knowledge-store-howto.md)。