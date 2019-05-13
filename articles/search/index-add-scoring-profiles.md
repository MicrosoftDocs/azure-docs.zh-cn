---
title: 将计分概要文件添加到搜索索引 - Azure 搜索
description: 通过添加计分概要文件，提高 Azure 搜索结果的搜索排名分数。
ms.date: 05/02/2019
services: search
ms.service: search
ms.topic: conceptual
author: Brjohnstmsft
ms.author: brjohnst
ms.manager: cgronlun
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 9ccb6944227208cee8601751cf43a53c111c09c6
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65021624"
---
# <a name="add-scoring-profiles-to-an-azure-search-index"></a>将计分概要文件添加到 Azure 搜索索引

  计分是指搜索结果中所返回每项的“搜索分数”计算。 分数指示某一项在当前搜索操作上下文中的相关性。 分数越高，项的相关度就越高。 在搜索结果中，根据为每项计算的搜索分数，按从高到低的顺序排列各项。  

 Azure 搜索使用默认计分计算初始分数，但可以通过“计分概要文件”自定义计算。 借助计分概要文件，可以更好地控制搜索结果中的项排名。 例如，建议根据创收能力提升项、提升新项或提升库存时间太长的项。  

 计分概要文件属于索引定义的一部分，由加权字段、函数和参数组成。  

 若要概览计分概要文件，请参阅下面的示例，其中展示了名为“geo”的简单概要文件。 此文件用于提升在“hotelName”字段中具有搜索词的项。 它还使用 `distance` 函数优先提升在当前位置十公里范围内的项。 如果有人搜索“inn”一词，而“inn”恰好是酒店名称的一部分，包含当前位置 10 公里范围内带有“inn”的酒店的文档会在搜索结果中的较高位置出现。  


```  
"scoringProfiles": [
  {  
    "name":"geo",
    "text": {  
      "weights": {  
        "hotelName": 5
      }                              
    },
    "functions": [
      {  
        "type": "distance",
        "boost": 5,
        "fieldName": "location",
        "interpolation": "logarithmic",
        "distance": {
          "referencePointParameter": "currentLocation",
          "boostingDistance": 10
        }                        
      }                                      
    ]                     
  }            
]
```  


 若要使用此计分概要文件，查询要表述为对查询字符串指定此概要文件。 在下面的查询中，请注意请求中的查询参数 `scoringProfile=geo`。  

```  
GET /indexes/hotels/docs?search=inn&scoringProfile=geo&scoringParameter=currentLocation--122.123,44.77233&api-version=2019-05-06 
```  

 此查询对“inn”一词进行搜索，并在当前位置中传递。 请注意，此查询包含其他参数，如`scoringParameter`。 有关查询参数的信息，请参阅[搜索文档和 Azure 搜索服务 REST API](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。  

 单击[示例](#bkmk_ex)查看计分概要文件的更详细示例。  

## <a name="what-is-default-scoring"></a>什么是默认计分？  
 计分对排序结果集中的每项计算搜索分数。 搜索结果集中的每项都分配到一个搜索分数，并从高到低排名。 分数较高的项返回应用程序。 默认返回前 50 个，但可以使用 `$top` 参数返回较少或较多的项（单个响应中最多 1000 个）。  

根据数据和查询的统计属性计算搜索分数。 Azure 搜索查找包含了查询字符串中搜索词的文档（部分或全部包含，具体取决于 `searchMode`），优先列出包含该搜索词多个实例的文档。 搜索分数仍升跨数据索引，但在文档中的常见术语很少甚至更高。 这种相关性计算方法的基础称为 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 或词频-逆文档频率。  

 假设没有自定义排序，结果在返回调用应用程序之前按搜索分数排名。 如果 $top 未指定，则返回具有最高搜索分数的 50 项。  

 搜索分数值可以在整个结果集中重复。 例如，可能有 10 个分数为 1.2 的项，20 个分数为 1.0 的项，20 个分数为 0.5 的项。 当多个命中具有相同的搜索分数时，得分相同项的顺序未定义，并且不稳定。 重新运行查询后，可能会看到项移位。 对于具有相同分数的两项，无法保证先显示哪一个。  

## <a name="when-to-use-custom-scoring"></a>何时使用自定义计分  
 当默认排名行为不足以满足业务目标时，应创建一个或更多计分概要文件。 例如，可能会决定搜索相关性应更倾向于新添加的项。 同样，可能有一个包含利润率的字段，或其他某些指示营收潜力的字段。 提升为业务带来益处的命中可能是确定使用计分概要文件的一个重要因素。  

 基于相关性的排序也通过计分概要文件实现。 请考虑过去使用的可以按价格、日期、评分或相关性排序的搜索结果页。 在 Azure 搜索中，计分概要文件驱动“相关性”选项。 相关性定义由你控制，基于业务目标和希望提供的搜索体验类型断定。  

##  <a name="bkmk_ex"></a> 示例  
 如前所述，自定义计分是通过索引模式中定义的一个或多个计分概要文件实现的。  

 此示例演示具有两个计分概要文件的索引架构（`boostGenre`、`newAndHighlyRated`）。 针对此索引的任何查询（包含任一概要文件作为查询参数）将使用此概要文件对结果集进行计分。  

```  
{  
  "name": "musicstoreindex",  
  "fields": [  
    { "name": "key", "type": "Edm.String", "key": true },  
    { "name": "albumTitle", "type": "Edm.String" },  
    { "name": "albumUrl", "type": "Edm.String", "filterable": false },  
    { "name": "genre", "type": "Edm.String" },  
    { "name": "genreDescription", "type": "Edm.String", "filterable": false },  
    { "name": "artistName", "type": "Edm.String" },  
    { "name": "orderableOnline", "type": "Edm.Boolean" },  
    { "name": "rating", "type": "Edm.Int32" },  
    { "name": "tags", "type": "Collection(Edm.String)" },  
    { "name": "price", "type": "Edm.Double", "filterable": false },  
    { "name": "margin", "type": "Edm.Int32", "retrievable": false },  
    { "name": "inventory", "type": "Edm.Int32" },  
    { "name": "lastUpdated", "type": "Edm.DateTimeOffset" }  
  ],  
  "scoringProfiles": [  
    {  
      "name": "boostGenre",  
      "text": {  
        "weights": {  
          "albumTitle": 1.5,  
          "genre": 5,  
          "artistName": 2  
        }  
      }  
    },  
    {  
      "name": "newAndHighlyRated",  
      "functions": [  
        {  
          "type": "freshness",  
          "fieldName": "lastUpdated",  
          "boost": 10,  
          "interpolation": "quadratic",  
          "freshness": {  
            "boostingDuration": "P365D"  
          }  
        },  
        {
          "type": "magnitude",  
          "fieldName": "rating",  
          "boost": 10,  
          "interpolation": "linear",  
          "magnitude": {  
            "boostingRangeStart": 1,  
            "boostingRangeEnd": 5,  
            "constantBoostBeyondRange": false  
          }  
        }  
      ]  
    }  
  ],  
  "suggesters": [  
    {  
      "name": "sg",  
      "searchMode": "analyzingInfixMatching",  
      "sourceFields": [ "albumTitle", "artistName" ]  
    }  
  ]   
}  
```  

## <a name="workflow"></a>工作流  
 要实现自定义计分行为，请将计分概要文件添加到定义该索引的架构。 在一个索引中可以具有最多 100 个计分概要文件（请参阅[服务限制](search-limits-quotas-capacity.md)），但在任何给定查询中，一次只能指定一个概要文件。  

 从本主题中提供的[模板](#bkmk_template)开始。  

 提供一个名称。 计分概要文件为可选项，但如果要添加一个概要文件，必须提供名称。 请务必遵循字段的命名约定（以字母开头，避免使用特殊字符和保留字）。 有关完整列表，请参阅[命名规则（Azure 搜索）](https://docs.microsoft.com/rest/api/searchservice/naming-rules)。  

 计分概要文件的主体由加权字段和函数构造而成。  

|||  
|-|-|  
|**权重**|指定为字段分配相对权重的名称/值对。 在[示例](#bkmk_ex)中，albumTitle、genre 和 artistName 字段各自提升 1.5、5 和 2。 genre 为何比其他字段提升更高？ 如果对在某种程度上为同类的数据执行搜索（正如 `musicstoreindex` 中的“genre”一样），可能需要在相对权重中产生较大差异。 例如，在 `musicstoreindex` 中，“rock”既作为流派显示，又显示在采用相同组句方式的流派说明中。 如果希望流派的权重在流派说明之上，genre 字段将需要更高的相对权重。|  
|**函数**|在特定上下文需要进行额外计算时使用。 有效的值为 `freshness`、`magnitude`、`distance` 和 `tag`。 每个函数具有独有的参数。<br /><br /> 当希望按项目的新旧方式进行提升时，应使用 -   `freshness`。 此函数仅可与 `datetime` 字段结合使用 (edm.DataTimeOffset)。 请注意`boostingDuration`属性仅用于`freshness`函数。<br />当希望按数值高低程度提升时，应使用 -   `magnitude`。 调用此函数的方案包括按照利润率、最高价格、最低价格或下载次数提升。 此函数仅可与 double 和 integer 字段结合使用。<br />     对于 `magnitude` 函数，如果想要反转模式（例如，将价格较低项提升至价格较高项之上），可以将范围反转为从高到低。 假设价格范围从 100 美元到 1 美元，可以将 `boostingRangeStart` 设为 100、`boostingRangeEnd` 设为 1 以提升价格较低的项。<br />当希望按距离或地理位置提升时，应使用 -   `distance`。 此函数仅可与 `Edm.GeographyPoint` 字段结合使用。<br />当希望按文档和搜索查询之间共有的标记提升时，应使用 -   `tag`。 此函数仅可与 `Edm.String` 和 `Collection(Edm.String)` 字段结合使用。<br /><br /> **使用函数的规则**<br /><br /> 函数类型（`freshness`、`magnitude`、`distance`）`tag` 必须小写。<br /><br /> 函数不能包含 null 或空值。 具体而言，如果包含字段名称，则必须将其设置为某值。<br /><br /> 函数仅可应用于可筛选字段。 有关可筛选字段的详细信息，请参阅[创建索引（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/create-index)。<br /><br /> 函数仅可应用于在索引的字段集合中定义的字段。|  

 定义索引后，请通过上传索引架构（后跟文档）建立索引。 有关这些操作的说明，请参阅[创建索引（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/create-index)和[添加、更新或删除文档（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)。 建立索引后，应获得适用于搜索数据的函数计分概要文件。  

##  <a name="bkmk_template"></a> 模板  
 本部分演示计分概要文件的语法和模板。 请参阅下一部分的[索引属性参考](#bkmk_indexref)，获取属性的说明。  

```  
. . .   
"scoringProfiles": [  
  {   
    "name": "name of scoring profile",   
    "text": (optional, only applies to searchable fields) {   
      "weights": {   
        "searchable_field_name": relative_weight_value (positive #'s),   
        ...   
      }   
    },   
    "functions": (optional) [  
      {   
        "type": "magnitude | freshness | distance | tag",   
        "boost": # (positive number used as multiplier for raw score != 1),   
        "fieldName": "...",   
        "interpolation": "constant | linear (default) | quadratic | logarithmic",   

        "magnitude": {
          "boostingRangeStart": #,   
          "boostingRangeEnd": #,   
          "constantBoostBeyondRange": true | false (default)
        }  

        // ( - or -)  

        "freshness": {
          "boostingDuration": "..." (value representing timespan over which boosting occurs)   
        }  

        // ( - or -)  

        "distance": {
          "referencePointParameter": "...", (parameter to be passed in queries to use as reference location)   
          "boostingDistance": # (the distance in kilometers from the reference location where the boosting range ends)   
        }   

        // ( - or -)  

        "tag": {
          "tagsParameter":  "..."(parameter to be passed in queries to specify a list of tags to compare against target field)   
        }
      }
    ],   
    "functionAggregation": (optional, applies only when functions are specified) "sum (default) | average | minimum | maximum | firstMatching"   
  }   
],   
"defaultScoringProfile": (optional) "...",   
. . .  
```  

##  <a name="bkmk_indexref"></a> 索引属性参考  

> [!NOTE]  
>  计分函数仅可应用于可筛选的字段。  

|属性|描述|  
|---------------|-----------------|  
|`Name`|必需。 这是计分概要文件的名称。 它遵循与字段相同的命名约定。 它必须以字母开头，不能包含点、冒号或 @ 符号，并且不能以短语“azureSearch”（区分大小写）开头。|  
|`Text`|包含 Weights 属性。|  
|`Weights`|可选。 指定字段名称和相对权重的名称-值对。 相对权重必须为正整数或浮点数。 最大值为 int32.MaxValue。<br /><br /> 可以指定不带相应权重的字段名称。 权重用于指示一个字段相对于另一个字段的重要性。|  
|`Functions`|可选。 计分函数仅可应用于可筛选的字段。|  
|`Type`|计分函数的必需项。 指示要使用的函数类型。 有效值包括 magnitude、freshness、distance 和 tag。 可以在每个计分概要文件中包含多个函数。 函数名称必须小写。|  
|`Boost`|计分函数的必需项。 用作原始分数乘数的正数。 不得等于 1。|  
|`Fieldname`|计分函数的必需项。 计分函数仅可应用于作为索引字段集合一部分且可筛选的字段。 此外，每个函数类型都引入了其他限制（freshness 与 datetime 字段结合使用、magnitude 与 integer 或 double 字段结合使用、distance 与 location 字段结合使用。）。 仅可按函数定义指定单个字段。 例如，若要在同一概要文件中使用两次 magnitude，则需要包含两个定义 magnitude，每个字段一个。|  
|`Interpolation`|计分函数的必需项。 定义从范围起始至范围结束的分数提升增量的斜率。 有效值包括 Linear（默认值）、Constant、Quadratic 和 Logarithmic。 请参阅[设置插值](#bkmk_interpolation)获取详细信息。|  
|`magnitude`|magnitude 计分函数用于改变基于数值字段的值范围的排名。 一些最常见的用法示例如下：<br /><br /> -   星级评分：根据“星级评分”字段中的值更改评分。 如果两个项相关，具有较高评分的项先显示。<br />-   **利润：** 当两个文档相关时，零售商可能希望先提升具有较高利润的文档。<br />-   **点击次数：** 对于跟踪产品或页面点击行为的应用程序，可使用 magnitude 提升容易获得最多流量的项。<br />-   **下载次数：** 对于跟踪下载的应用程序，magnitude 函数可提升下载次数最多的项。|  
|`magnitude` &#124; `boostingRangeStart`|设置对其进行量值计分的范围的起始值。 该值必须是整数或浮点数。 对于星级评分 1 到 4，这里应为 1。 对于超过 50% 的利润率，这里应为 50。|  
|`magnitude` &#124; `boostingRangeEnd`|设置对其进行量值计分的范围的结束值。 该值必须是整数或浮点数。 对于星级评分 1 到 4，这里应为 4。|  
|`magnitude` &#124; `constantBoostBeyondRange`|有效值为 true 或 false（默认）。 设置为 true 时，完整的提升将继续应用到有一个目标字段值高于范围上限的文档。 如果为 false，此函数的提升不会应用到有一个目标字段值超出范围的文档。|  
|`freshness`|freshness 计分函数用于改变基于 `DateTimeOffset` 字段值的项的排名分数。 例如，具有较新日期的项可以排在日期较旧的项之上。<br /><br /> 这样可以高于项在将来进一步排名到最新的项更接近还有可以排列日历事件等具有未来日期。<br /><br /> 在当前服务版本中，范围的一端将固定为当前时间。 另一端是基于 `boostingDuration` 的过去的时间。 要提升将来时间的范围，请使用负 `boostingDuration`。<br /><br /> 提升从最大范围和最小范围改变的比率由应用到计分概要文件的内插确定（请见下图）。 若要反转应用的提升系数，请选择不超过 1 的提升系数。|  
|`freshness` &#124; `boostingDuration`|设置一个有效期，超过这个有效期之后，针对特定文档的 Boosting 将停止。 请参阅以下语法和示例部分中的[设置 boostingDuration](#bkmk_boostdur)。|  
|`distance`|距离计分函数用于影响基于其相对的参考地理位置远近程度的文档分数。 参考位置在形参中指定为查询的一部分（使用 `scoringParameterquery` 字符串选项），作为经纬度实参。|  
|`distance` &#124; `referencePointParameter`|要在查询中传递以用作参考位置的参数。 `scoringParameter` 是一个查询参数。 有关查询参数的说明，请参阅[搜索文档（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。|  
|`distance` &#124; `boostingDistance`|以公里为单位指示与参考位置（提升范围结束）之间距离的数字。|  
|`tag`|tag 计分函数用于影响基于文档和搜索查询中标记的文档的分数。 将提升与搜索查询有共同标记的文档。 搜索查询的标记作为每个搜索请求中的计分参数提供（使用 `scoringParameterquery` 字符串选项）。|  
|`tag` &#124; `tagsParameter`|要在查询中传递以指定特定请求标记的参数。 `scoringParameter` 是一个查询参数。 有关查询参数的说明，请参阅[搜索文档（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。|  
|`functionAggregation`|可选。 仅在函数已指定时适用。 有效值包括：sum（默认值）、average、minimum、maximum和 firstMatching。 搜索分数是从多个变量（包括多个函数）中计算的单个值。 此属性指示所有函数的提升如何组合到随后应用到基本文档分数的单个聚合提升中。 基本分数基于从文档和搜索查询计算得出的 [tf-idf](http://www.tfidf.com/) 值。|  
|`defaultScoringProfile`|在执行搜索请求时，如果未指定任何计分概要文件，则使用默认计分（仅限 [tf-idf](http://www.tfidf.com/)）。<br /><br /> 可以在此处设置默认计分概要文件名称，以使 Azure 搜索在搜索请求中未给定任何特定概要文件时使用默认概要文件。|  

##  <a name="bkmk_interpolation"></a> 设置内插  
 通过内插，可设置用于计分的坡度形状。 由于评分从高到低，坡度总是在下降，但内插决定了下坡的曲线。 可以使用以下内插：  

|||  
|-|-|  
|`Linear`|对于在最大和最小范围内的项目，将按递减的方式进行提升。 Linear 是计分概要文件的默认内插。|  
|`Constant`|对于起始和结束范围内的项，将对排名结果应用恒定提升。|  
|`Quadratic`|与具有不断减小的提升的线性内插相比，二次方最初以较小的速度递减，然后在接近结束范围时，以更高的间隔递减。 tag 计分函数中不允许使用此内插选项。|  
|`Logarithmic`|与具有恒定递减提升的 Linear 内插相比，Logarithmic 最初以较大的速度递减，然后在接近结束范围时，以明显更小的间隔递减。 tag 计分函数中不允许使用此内插选项。|  

 ![关系图上的常数、线性、二次、log10 线](media/scoring-profiles/azuresearch_scorefunctioninterpolationgrapht.png "AzureSearch_ScoreFunctionInterpolationGrapht")  

##  <a name="bkmk_boostdur"></a>设置 boostingDuration  
 `boostingDuration` 是 `freshness` 函数的属性。 使用它设置一个有效期，超过这个有效期之后，针对特定文档的提升将停止。 例如，要在 10 天促销期内提升某个产品系列或品牌，应针对这些文档将 10 天期限指定为 "P10D"。  

 `boostingDuration` 必须设置为 XSD "dayTimeDuration" 值（ISO 8601 持续时间值的受限子集）的格式。 它的模式为：“P[nD][T[nH][nM][nS]]”。  

 下表提供几个示例。  

|Duration|boostingDuration|  
|--------------|----------------------|  
|1 天|"P1D"|  
|2 天 12 小时|"P2DT12H"|  
|15 分钟|"PT15M"|  
|30 天 5 小时 10 分钟 6.334 秒|"P30DT5H10M6.334S"|  

 有关更多示例，请参阅 [XML 架构：数据类型（W3.org 网站）](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration)。  

## <a name="see-also"></a>另请参阅  
 [Azure 搜索服务 REST](https://docs.microsoft.com/rest/api/searchservice/)   
 [创建索引（Azure 搜索服务 REST API）](https://docs.microsoft.com/rest/api/searchservice/create-index)   
 [Azure 搜索 .NET SDK](https://docs.microsoft.com/dotnet/api/overview/azure/search?view=azure-dotnet)  
