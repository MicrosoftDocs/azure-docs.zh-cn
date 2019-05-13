---
title: Azure Cosmos DB 中的 SQL 语言语法
description: 本文介绍 Azure Cosmos DB 中使用的 SQL 查询语言语法、不同的运算符以及此语言中可用的关键字。
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: mjbrown
ms.custom: seodec18
ms.openlocfilehash: 186e0365ae8aee3b7f92fcc06142e4d0496ffd08
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415461"
---
# <a name="sql-language-reference-for-azure-cosmos-db"></a>Azure Cosmos DB SQL 语言参考 

Azure Cosmos DB 支持使用分层 JSON 文档中的语法等熟悉的 SQL（结构化查询语言）来查询文档，无需显示架构或创建辅助索引。 本文提供的文档适用于在 SQL API 帐户中使用的 SQL 查询语言语法。 有关示例 SQL 查询的演练，请参阅 [Cosmos DB 中的 SQL 查询示例](how-to-sql-query.md)。  
  
请访问[查询操场](https://www.documentdb.com/sql/demo)，可在其中尝试 Cosmos DB，并针对示例数据集运行 SQL 查询。  
  
## <a name="select-query"></a>SELECT 查询  
每个查询按 ANSI-SQL 标准由 SELECT 子句和可选的 FROM 和 WHERE 子句组成。 通常情况下，对于每个查询，需枚举 FROM 子句中的源，然后将 WHERE 子句中的筛选器应用到源以检索 JSON 文档的子集。 最后，使用 SELECT 子句以投影选择列表中请求的 JSON 值。 有关示例，请参阅 [SELECT 查询示例](how-to-sql-query.md#SelectClause)
  
**语法**  
  
```sql
<select_query> ::=  
SELECT <select_specification>   
    [ FROM <from_specification>]   
    [ WHERE <filter_condition> ]  
    [ ORDER BY <sort_specification> ] 
    [ OFFSET <offset_amount> LIMIT <limit_amount>]
```  
  
 **注释**  
  
 有关每个子句的详细信息，请参阅以下部分。  
  
-   [SELECT 子句](#bk_select_query)    
-   [FROM 子句](#bk_from_clause)    
-   [WHERE 子句](#bk_where_clause)    
-   [ORDER BY 子句](#bk_orderby_clause)  
-   [偏移量的限制子句](#bk_offsetlimit_clause)

  
SELECT 语句中的子句必须按照以上所示进行排序。 可以省略任一可选子句。 但是当使用可选子句时，它们必须按正确的顺序显示。  
  
### <a name="logical-processing-order-of-the-select-statement"></a>SELECT 语句的逻辑处理顺序  
  
处理子句的顺序是：  

1.  [FROM 子句](#bk_from_clause)  
2.  [WHERE 子句](#bk_where_clause)  
3.  [ORDER BY 子句](#bk_orderby_clause)  
4.  [SELECT 子句](#bk_select_query)
5.  [偏移量的限制子句](#bk_offsetlimit_clause)

请注意，此顺序与语法中出现的顺序不同。 采用此排序是为了使已处理子句引入的所有新符号可见，并且可用于后来处理的字句中。 例如，在 WHERE 和 SELECT 子句中可访问在 FROM 子句中声明的别名。  

### <a name="whitespace-characters-and-comments"></a>空白字符和注释  

不属于带引号的字符串或带引号的标识符中的所有空格字符也不属于语言语法的一部分，在解析过程中将被忽略。  

查询语言支持 T-SQL 样式注释，如  

-   SQL 语句`-- comment text [newline]`  

虽然空格字符和注释在语法中没有任何意义，但它们必须用于分隔令牌。 例如：`-1e5` 是单个数字令牌，而 `: – 1 e5` 是减号令牌，后跟数字 1 和标识符 e5。  

##  <a name="bk_select_query"></a> SELECT 子句  
SELECT 语句中的子句必须采用上面显示的顺序。 可以省略任一可选子句。 但是当使用可选子句时，它们必须按正确的顺序显示。 有关示例，请参阅 [SELECT 查询示例](how-to-sql-query.md#SelectClause)。

**语法**  

```sql
SELECT <select_specification>  

<select_specification> ::=   
      '*'   
      | [DISTINCT] <object_property_list>   
      | [DISTINCT] VALUE <scalar_expression> [[ AS ] value_alias]  
  
<object_property_list> ::=   
{ <scalar_expression> [ [ AS ] property_alias ] } [ ,...n ]  
  
```  
  
 **参数**  
  
- `<select_specification>`  

  为结果集选择的属性或值。  
  
- `'*'`  

  指定在不进行任何更改的情况下检索该值。 具体来说，如果处理的值是一个对象，将检索所有属性。  
  
- `<object_property_list>`  
  
  指定要检索的属性列表。 每个返回值将是一个具有指定属性的对象。  
  
- `VALUE`  

  指定应检索 JSON 值而不是完整的 JSON 对象。 与 `<property_list>` 不同的是，它不会将投影值包装在对象中。  
 
- `DISTINCT`
  
  指定应删除投影属性的重复项。  

- `<scalar_expression>`  

  表示要计算的值的表达式。 请参阅[标量表达式](#bk_scalar_expressions)部分，了解详细信息。  
  
**注释**  
  
只有当 FROM 子句已声明一个别名时，`SELECT *` 语法才有效。 `SELECT *` 提供一个标识投影，在不需要投影的情况下会有所帮助。 只有当指定了 FROM 子句，并且该字句只引入单个输入源时，SELECT * 才有效。  
  
`SELECT <select_list>` 和 `SELECT *` 是“语法糖”，可另外使用简单的 SELECT 语句表示，如下所示。  
  
1. `SELECT * FROM ... AS from_alias ...`  
  
   等效于：  
  
   `SELECT from_alias FROM ... AS from_alias ...`  
  
2. `SELECT <expr1> AS p1, <expr2> AS p2,..., <exprN> AS pN [other clauses...]`  
  
   等效于：  
  
   `SELECT VALUE { p1: <expr1>, p2: <expr2>, ..., pN: <exprN> }[other clauses...]`  
  
另请参阅  
  
[标量表达式](#bk_scalar_expressions)  
[SELECT 子句](#bk_select_query)  
  
##  <a name="bk_from_clause"></a> FROM 子句  
指定源或联接源。 FROM 子句是可选的，除非稍后在查询中对源进行筛选或投影。 此子句的目的在于指定必须对其执行查询的数据源。 通常情况下，整个容器作为源，但可以改为指定容器的子集作为源。 如果未指定此子句，其他子句仍将继续执行，如同 FROM 子句提供了单个文档。 有关示例，请参阅 [FROM 子句示例](how-to-sql-query.md#FromClause)
  
**语法**  
  
```sql  
FROM <from_specification>  
  
<from_specification> ::=   
        <from_source> {[ JOIN <from_source>][,...n]}  
  
<from_source> ::=   
          <container_expression> [[AS] input_alias]  
        | input_alias IN <container_expression>  
  
<container_expression> ::=   
        ROOT   
     | container_name  
     | input_alias  
     | <container_expression> '.' property_name  
     | <container_expression> '[' "property_name" | array_index ']'  
```  
  
**参数**  
  
- `<from_source>`  
  
  指定一个具有或不具有别名的数据源。 如果未指定别名，将从 `<container_expression>` 中使用以下规则推断：  
  
  -  如果表达式为 container_name，那么 container_name 将用作别名。  
  
  -  如果表达式为 `<container_expression>`，其次是 property_name，那么 property_name 将用作别名。 如果表达式为 container_name，那么 container_name 将用作别名。  
  
- AS `input_alias`  
  
  指定 `input_alias` 为基础容器表达式返回的一组值。  
 
- `input_alias` IN  
  
  指定 `input_alias` 应表示通过遍历由基础容器表达式返回的每个数组的所有数组元素获得的一组值。 基础容器表达式返回的非数组的任何值都将被忽略。  
  
- `<container_expression>`  
  
  指定要用于检索文档的容器表达式。  
  
- `ROOT`  
  
  指定应从当前连接的默认容器中检索文档。  
  
- `container_name`  
  
  指定应从当前提供的容器中检索文档。 容器的名称必须匹配当前连接到的容器名称。  
  
- `input_alias`  
  
  指定应从提供的别名定义的其他源中检索文档。  
  
- `<container_expression> '.' property_`  
  
  指定应通过访问由指定容器表达式检索的所有文档的 `property_name` 属性或 array_index 数组元素来检索文档。  
  
- `<container_expression> '[' "property_name" | array_index ']'`  
  
  指定应通过访问由指定容器表达式检索的所有文档的 `property_name` 属性或 array_index 数组元素来检索文档。  
  
**注释**  
  
`<from_source>(`) 中提供或推断的所有别名必须唯一。 语法 `<container_expression>.`property_name 等同于 `<container_expression>' ['"property_name"']'`。 但是，如果属性名称包含非标识符字符，则可以使用后一种语法。  
  
### <a name="handling-missing-properties-missing-array-elements-and-undefined-values"></a>处理未命中属性、未命中数组元素和未定义值
  
如果某个容器表达式访问属性或数组元素，但该值不存在，将忽略该值，且不会做进一步处理。  
  
### <a name="container-expression-context-scoping"></a>容器表达式上下文范围  
  
容器表达式的范围可以是集合或文档：  
  
-   如果容器表达式的基础源是 ROOT 或 `container_name`，则表达式的范围是容器。 此类表达式表示从容器中直接检索的一组文档，且不依赖于其他容器表达式的处理。  
  
-   如果容器表达式的基础源是之前在查询中引入的 `input_alias`，则表达式的范围是文档。 此类表达式表示通过计算每个文档范围内的容器表达式获得的一组文档，该文档属于与别名容器相关联的集。  结果集为多个集的合并，通过计算基础集中每个文档的容器表达式获得。  
  
### <a name="joins"></a>联接 
  
在当前发布的版本中，Cosmos DB 支持内联。 其他联接功能即将发布。 

内联会导致加入联接的集产生完整叉积。 N 向联接的结果是获得一组 N-元素元组，其中元组中的每个值与参与联接的别名集相关联，并且可以通过引用其他子句中的这些别名来访问。 有关示例，请参阅 [JOIN 关键字示例](how-to-sql-query.md#Joins)
  
联接的计算取决于参与集的上下文范围：  
  
- 容器集 A 和容器范围的集 B 之间的联接，会导致集 A 和集 B 的所有元素产生叉积。
  
- 集 A 和文档范围的集 B 之间的联接，通过对集 A 中每个文档计算文档范围的集 B 来获得所有集的合并。  
  
  在当前发布的版本中，查询处理器支持一个容器范围的表达式的最大值。  
  
### <a name="examples-of-joins"></a>联接示例  
  
请看以下 FROM 子句：`<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`  
  
 让每个源定义 `input_alias1, input_alias2, …, input_aliasN`。 此 FROM 子句将返回一组 N 元组（带有 N 个值的元组）。 每个元组拥有通过对它们相应的集遍历所有容器别名所产生的值。  
  
示例 1 - 2 个源  
  
- 让 `<from_source1>` 的范围为容器，并表示集 {A, B, C}。  
  
- 让 `<from_source2>` 作为文档范围的引用 input_alias1，表示的集如下所示：  
  
    若 `input_alias1 = A,`，表示集 {1, 2}  
  
    若 `input_alias1 = B,`，表示集 {3}  
  
    若 `input_alias1 = C,`，表示集 {4, 5}  
  
- FROM 子句 `<from_source1> JOIN <from_source2>` 将生成以下元组：  
  
    (`input_alias1, input_alias2`):  
  
    `(A, 1), (A, 2), (B, 3), (C, 4), (C, 5)`  
  
示例 2 - 3 个源  
  
- 让 `<from_source1>` 的范围为容器，并表示集 {A, B, C}。  
  
- 让 `<from_source2>` 作为文档范围的引用 `input_alias1`，表示的集如下所示：  
  
    若 `input_alias1 = A,`，表示集 {1, 2}  
  
    若 `input_alias1 = B,`，表示集 {3}  
  
    若 `input_alias1 = C,`，表示集 {4, 5}  
  
- 让 `<from_source3>` 作为文档范围的引用 `input_alias2`，表示的集如下所示：  
  
    若 `input_alias2 = 1,`，表示集 {100, 200}  
  
    若 `input_alias2 = 3,`，表示集 {300}  
  
- FROM 子句 `<from_source1> JOIN <from_source2> JOIN <from_source3>` 将产生以下元组：  
  
    (input_alias1, input_alias2, input_alias3):  
  
    (A, 1, 100), (A, 1, 200), (B, 3, 300)  
  
  > [!NOTE]
  > `input_alias1` 和 `input_alias2` 的其他值缺少元组，`<from_source3>` 不对其返回任何值。  
  
示例 3 - 3 个源  
  
- 让 <from_source1> 的范围为容器，并表示集 {A, B, C}。  
  
- 让 `<from_source1>` 的范围为容器，并表示集 {A, B, C}。  
  
- <from_source2> 作为引用 input_alias1 的文档作用域，并表示集：  
  
    若 `input_alias1 = A,`，表示集 {1, 2}  
  
    若 `input_alias1 = B,`，表示集 {3}  
  
    若 `input_alias1 = C,`，表示集 {4, 5}  
  
- 让 `<from_source3>` 的范围限定为 `input_alias1`，表示的集如下所示：  
  
    若 `input_alias2 = A,`，表示集 {100, 200}  
  
    若 `input_alias2 = C,`，表示集 {300}  
  
- FROM 子句 `<from_source1> JOIN <from_source2> JOIN <from_source3>` 将产生以下元组：  
  
    (`input_alias1, input_alias2, input_alias3`)：  
  
    (A, 1, 100), (A, 1, 200), (A, 2, 100), (A, 2, 200),  (C, 4, 300) ,  (C, 5, 300)  
  
  > [!NOTE]
  > 这会导致 `<from_source2>` 和 `<from_source3>` 产生叉积，因为两者的范围都为 `<from_source1>`。  这会产生 4 (2x2) 个具有 A 值的元组、0 个具有 B (1x0) 值的元组、和 2 (2x1) 个具有 C 值的元组。  
  
**另请参阅**  
  
 [SELECT 子句](#bk_select_query)  
  
##  <a name="bk_where_clause"></a> WHERE 子句  
 为查询返回的文档指定搜索条件。 有关示例，请参阅 [WHERE 子句示例](how-to-sql-query.md#WhereClause)
  
 **语法**  
  
```sql  
WHERE <filter_condition>  
<filter_condition> ::= <scalar_expression>  
  
```  
  
 **参数**  
  
- `<filter_condition>`  
  
   指定返回的文档需满足的条件。  
  
- `<scalar_expression>`  
  
   表示要计算的值的表达式。 请参阅[标量表达式](#bk_scalar_expressions)部分，了解详细信息。  
  
  **注释**  
  
  为了能够返回文档，指定为筛选条件的表达式的求值结果必须为 true。 只有当布尔值为 true 才可以满足条件，任何其他值：未定义、null、false、数字、数组或对象均不满足条件。  
  
##  <a name="bk_orderby_clause"></a> ORDER BY 子句  
 为查询返回的结果指定排序顺序。 有关示例，请参阅 [ORDER BY 子句示例](how-to-sql-query.md#OrderByClause)
  
 **语法**  
  
```sql  
ORDER BY <sort_specification>  
<sort_specification> ::= <sort_expression> [, <sort_expression>]  
<sort_expression> ::= {<scalar_expression> [ASC | DESC]} [ ,...n ]  
  
```  

 **参数**  
  
- `<sort_specification>`  
  
   指定查询结合集要进行排序的属性或表达式。 列排序可指定为名称或属性的别名。  
  
   可以指定多个属性。 属性名称必须是唯一的。 ORDER BY 子句中的排序属性的序列定义的排序的结果集的组织。 即：结果集按第一个属性排序，然后该排序列表按第二个属性排序，依此类推。  
  
   在 ORDER BY 子句中引用的属性名称必须对应到选择列表中的任一属性或明确地在 FROM 子句中指定的集合中定义的属性。  
  
- `<sort_expression>`  
  
   指定一个或多个属性或对查询结果集进行排序所依据的表达式。  
  
- `<scalar_expression>`  
  
   请参阅[标量表达式](#bk_scalar_expressions)部分，了解详细信息。  
  
- `ASC | DESC`  
  
   指定在指定列中的值应按升序或降序排序。 ASC 采用从最低值到最高值的顺序排序。 DESC 采用从最高值到最低值的顺序排序。 ASC 是默认的排序顺序。 NULL 值被视为可能的最低值。  
  
  **注释**  
  
   ORDER BY 子句需要索引策略，包括正在排序的字段的索引。 Azure Cosmos DB 查询运行时支持排序与属性名和不是针对计算的属性。 Azure Cosmos DB 支持多个 ORDER BY 属性。 若要运行查询具有多个 ORDER BY 属性，则应定义[组合索引](index-policy.md#composite-indexes)上正在排序的字段。


##  <a name=bk_offsetlimit_clause></a> 偏移量的限制子句

指定跳过的项目数和返回的项数。 有关示例，请参阅[偏移量的限制子句示例](how-to-sql-query.md#OffsetLimitClause)
  
 **语法**  
  
```sql  
OFFSET <offset_amount> LIMIT <limit_amount>
```  
  
 **参数**  
 
- `<offset_amount>`

   指定查询结果应跳过的项的整数数目。


- `<limit_amount>`
  
   指定查询结果中应包含的项的整数数目

  **注释**  
  
  偏移量的限制子句中需要的偏移量计数和限制计数。 如果可选的`ORDER BY`子句使用时，结果集生成的执行跳过操作的有序的值。 否则，查询将返回值的固定的顺序。

##  <a name="bk_scalar_expressions"></a> 标量表达式  
 标量表达式是符号和运算符的组合，经计算后可获得单个值。 简单表达式可以是常数、属性引用、数组元素引用、别名引用或函数调用。 简单表达式可以使用运算符组合成复杂的表达式。 有关示例，请参阅[标量表达式示例](how-to-sql-query.md#scalar-expressions)
  
 有关标量表达式可能包含的值的详细信息，请参阅[常数](#bk_constants)部分。  
  
 **语法**  
  
```sql  
<scalar_expression> ::=  
       <constant>   
     | input_alias   
     | parameter_name  
     | <scalar_expression>.property_name  
     | <scalar_expression>'['"property_name"|array_index']'  
     | unary_operator <scalar_expression>  
     | <scalar_expression> binary_operator <scalar_expression>    
     | <scalar_expression> ? <scalar_expression> : <scalar_expression>  
     | <scalar_function_expression>  
     | <create_object_expression>   
     | <create_array_expression>  
     | (<scalar_expression>)   
  
<scalar_function_expression> ::=  
        'udf.' Udf_scalar_function([<scalar_expression>][,…n])  
        | builtin_scalar_function([<scalar_expression>][,…n])  
  
<create_object_expression> ::=  
   '{' [{property_name | "property_name"} : <scalar_expression>][,…n] '}'  
  
<create_array_expression> ::=  
   '[' [<scalar_expression>][,…n] ']'  
  
```  
  
 **参数**  
  
- `<constant>`  
  
   表示一个常数值。 有关详细信息，请参阅[常数](#bk_constants)部分。  
  
- `input_alias`  
  
   表示由 `FROM` 子句引入，由 `input_alias` 定义的值。  
  该值不得是“未定义”–输入中的“未定义”值将被跳过。  
  
- `<scalar_expression>.property_name`  
  
   表示对象的属性值。 如果属性不存在或对非对象的值引用属性，则表达式的求值结果为“未定义”值。  
  
- `<scalar_expression>'['"property_name"|array_index']'`  
  
   使用名称 `property_name` 表示属性的值，使用对象/数组索引 `array_index` 表示数组元素。 如果不存在属性/数组索引，或对非对象/数组的值引用了属性/数组索引，则表达式的求值结果为未定义值。  
  
- `unary_operator <scalar_expression>`  
  
   表示应用于单个值的运算符。 有关详细信息，请参阅[运算符](#bk_operators)部分。  
  
- `<scalar_expression> binary_operator <scalar_expression>`  
  
   表示应用于两个值的运算符。 有关详细信息，请参阅[运算符](#bk_operators)部分。  
  
- `<scalar_function_expression>`  
  
   表示由函数调用结果定义的值。  
  
- `udf_scalar_function`  
  
   用户定义的标量函数的名称。  
  
- `builtin_scalar_function`  
  
   内置标量函数的名称。  
  
- `<create_object_expression>`  
  
   表示通过使用指定属性及其值创建一个新对象来获得的值。  
  
- `<create_array_expression>`  
  
   表示通过使用指定值作为元素创建一个新数组来获得的值。  
  
- `parameter_name`  
  
   表示指定参数名称的值。 参数名称必须以单个 \@ 作为第一个字符。  
  
  **注释**  
  
  调用内置或用户定义的标量函数时，必须定义所有参数。 如果未定义任何参数，则不会调用该函数，且结果也将为未定义。  
  
  创建对象时，将跳过任何已分配未定义值的属性，并且在创建的对象中不包含该属性。  
  
  创建数组时，将跳过任何已分配“未定义”值的元素值，并且在创建的对象中不包含该值。 这将导致下一个定义的元素取代该值时，创建的数组将不含跳过的索引。  
  
##  <a name="bk_operators"></a> 运算符  
 本部分将介绍受支持的运算符。 每个运算符都恰好可以被分配给一个类别。  
  
 有关处理“未定义”值、输入值的类型要求和处理类型不匹配的值的详细信息，请参阅下方的“运算符类别”表。  
  
 运算符类别：  
  
|**类别**|**详细信息**|  
|-|-|  
|算术|运算符要求输入为数字。 输出也应为数字。 如果输入为“未定义”或键入数字外的其他值，则结果为“未定义”。|  
|位|运算符要求输入为 32 位带符号整数。 输出也应为 32 位带符号整数。<br /><br /> 任何非整数值将进行舍入。 正值将向下舍入，负值向上舍入。<br /><br /> 任何超出 32 位整数范围的值都将通过采用最后 32 位的二的补数表示法进行转换。<br /><br /> 如果输入为“未定义”或键入数字外的其他值，则结果为“未定义”。<br /><br /> **注意：** 上述行为与 JavaScript 位运算符的行为兼容。|  
|逻辑|运算符要求输入为布尔值。 输出也应为布尔值。<br />如果输入“未定义”或键入布尔值外的其他值，则结果为“未定义”。|  
|**比较**|运算符要求输入的类型应该相同并且不是未定义。 输出应为布尔值。<br /><br /> 如果输入为“未定义”或输入为其他类型，则结果为“未定义”。<br /><br /> 有关值排序的详细信息，请参阅“对要比较的值进行排序”表。|  
|**string**|运算符要求输入为字符串。 输出也应为字符串。<br />如果输入为“未定义”或键入字符串外的其他值，则结果为“未定义”。|  
  
 一元运算符：  
  
|**名称**|运算符|**详细信息**|  
|-|-|-|  
|算术|+<br /><br /> -|返回数字值。<br /><br /> 位求反。 返回求反后的数字值。|  
|位|~|一的补数。 返回数字值的补数。|  
|逻辑|**NOT**|求反。 返回求反后的布尔值。|  
  
 二进制运算符：  
  
|**名称**|运算符|**详细信息**|  
|-|-|-|  
|算术|+<br /><br /> -<br /><br /> *<br /><br /> /<br /><br /> %|加。<br /><br /> 减。<br /><br /> 乘。<br /><br /> 除。<br /><br /> 取模。|  
|位|&#124;<br /><br /> &<br /><br /> ^<br /><br /> <<<br /><br /> >><br /><br /> >>>|位或。<br /><br /> 位与。<br /><br /> 位异或。<br /><br /> 左移。<br /><br /> 右移。<br /><br /> 补零右移。|  
|逻辑|**AND**<br /><br /> **OR**|逻辑与。 如果两个参数都为 true，则返回 true，否则返回 false。<br /><br /> 逻辑或运算。 如果任何参数为 true，则返回 true，否则返回 false。|  
|比较|**=**<br /><br /> !=, <><br /><br /> **>**<br /><br /> **>=**<br /><br /> **<**<br /><br /> **<=**<br /><br /> ??|等于。 如果参数为等于，则返回 "true"，否则返回 "false"。<br /><br /> 不等于。 如果参数为不等于，则返回 "true"，否则返回 "false"。<br /><br /> 大于。 如果第一个参数大于第二个参数，则返回 "true"，否则返回 "false"。<br /><br /> 大于等于。 如果第一个参数大于或等于第二个参数，则返回 "true"，否则返回 "false"。<br /><br /> 小于。 如果第一个参数小于第二个参数，则返回 "true"，否则返回 "false"。<br /><br /> 小于等于。 如果第一个参数小于或等于第二个参数，则返回 "true"，否则返回 "false"。<br /><br /> 联合。 如果第一个参数是“未定义”值，则返回第二个参数。|  
|**字符串**|**&#124;&#124;**|串联。 返回两个参数的串联。|  
  
 三元运算符：  

|**名称**|运算符|**详细信息**| 
|-|-|-|  
|三元运算符|?|如果第一个参数求值的结果为 "true"，则返回第二个参数；否则返回第三个参数。|  

  
 对要比较的值进行排序  
  
|**类型**|值顺序|  
|-|-|  
|未定义|不可比较。|  
|Null|单个值：null|  
|**Number**|自然实数。<br /><br /> 负无穷大值小于任何其他数字值。<br /><br /> 正无穷大值大于任何其他数字值。NaN 值不可比较。 与 NaN 进行比较将产生“未定义”值。|  
|**String**|字典顺序。|  
|数组|不排序，但可以相等。|  
|**Object**|不排序，但可以相等。|  
  
 **注释**  
  
 在 Cosmos DB 中，通常在数据库中检索到值时，才知道其类型。 为了支持高效地执行查询，大多数运算符都具有严格的类型要求。 此外运算符本身也不执行隐式转换。  
  
 这意味着：SELECT * FROM ROOT r WHERE r.Age = 21 之类的查询仅返回属性 Age 等于数字 21 的文档。 属性 Age 等于字符串 "21" 或字符串 "0021" 的文档将不匹配，因为表达式 "21" = 21 的求值结果为未定义。 这样可以更好地利用索引，因为查找特定值（例如数字 21）比搜索不确定个数的可能匹配项（数字 21 或字符串 "21"、"021"、"21.0"...）更快。 这种方式与 JavaScript 计算不同类型的值的运算符不同。  
  
 **数组和对象的相等与比较**  
  
 由于没有定义对象或数组值的顺序，使用范围运算符 (>, >=, <, <=) 比较数组或对象的值将产生未定义。 但是，支持使用相等/不等运算符 (=, !=, <>)，并且在结构上比较值。  
  
 如果两个数组具有相同数量的元素，且在匹配位置的元素也相等，则数组相等。 如果比较任何一对元素产生未定义，则数组比较的结果也为未定义。  
  
 如果两个对象定义了相同的属性，且匹配属性的值也相等，则对象相等。 如果比较任何一对属性值产生未定义，则对象比较的结果也为未定义。  
  
##  <a name="bk_constants"></a> 常量  
 常数也称为文本值或标量值，是表示特定数据值的符号。 常数的格式取决于它代表的值的数据类型。  
  
 受支持的标量数据类型：  
  
|类型|值顺序|  
|-|-|  
|未定义|单个值：未定义|  
|**Null**|单个值：null|  
|布尔值|值："false"、"true"。|  
|**数字**|双精度浮点数，IEEE 754 标准。|  
|字符串|零个或多个 Unicode 字符序列。 字符串必须用单引号或双引号括起来。|  
|数组|零个或多个元素序列。 每个元素可以是任何标量数据类型的值，未定义除外。|  
|**对象**|零个或多个名称/值对的无序集。 当名称为一个Unicode字符串，值可以属于任何标量数据类型，“未定义”除外。|  
  
 **语法**  
  
```sql  
<constant> ::=  
   <undefined_constant>  
     | <null_constant>   
     | <boolean_constant>   
     | <number_constant>   
     | <string_constant>   
     | <array_constant>   
     | <object_constant>   
  
<undefined_constant> ::= undefined  
  
<null_constant> ::= null  
  
<boolean_constant> ::= false | true  
  
<number_constant> ::= decimal_literal | hexadecimal_literal  
  
<string_constant> ::= string_literal  
  
<array_constant> ::=  
    '[' [<constant>][,...n] ']'  
  
<object_constant> ::=   
   '{' [{property_name | "property_name"} : <constant>][,...n] '}'  
  
```  
  
 **参数**  
  
* `<undefined_constant>; undefined`  
  
  表示 Undefined 类型的未定义值。  
  
* `<null_constant>; null`  
  
  表示 Null 类型的 null 值。  
  
* `<boolean_constant>`  
  
  表示 Boolean 类型的常数。  
  
* `false`  
  
  表示 Boolean 类型的 false 值。  
  
* `true`  
  
  表示 Boolean 类型的 true 值。  
  
* `<number_constant>`  
  
  表示常数。  
  
* `decimal_literal`  
  
  十进制文本是使用十进制表示法或科学记数法表示的数字。  
  
* `hexadecimal_literal`  
  
  十六进制文本是使用 "0x" 前缀表示的数字，后跟一个或多个十六进制数字。  
  
* `<string_constant>`  
  
  表示字符串类型的常数。  
  
* `string _literal`  
  
  字符串文本是以零个或多个 Unicode 字符序列或转义符序列表示的 Unicode 字符串。 字符串文本括在单引号 (') 或双引号 (") 中。  
  
  允许以下转义序列：  
  
|**转义序列**|**说明**|**Unicode 字符**|  
|-|-|-|  
|\\'|撇号 (')|U+0027|  
|\\"|引号 (")|U+0022|  
|\\\ |反斜线号 (\\)|U+005C|  
|\\/|斜线号 (/)|U+002F|  
|\b|退格符|U+0008|  
|\f|换页符|U+000C|  
|\n|换行符|U+000A|  
|\r|回车符|U+000D|  
|\t|Tab|U+0009|  
|\uXXXX|由 4 个十六进制数字定义的 Unicode 字符。|U+XXXX|  
  
##  <a name="bk_query_perf_guidelines"></a> 查询性能准则  
 为了能够对大型容器高效地执行查询，应使用可通过由一个或多个索引服务的筛选器。  
  
 以下筛选器将考虑用于索引查找：  
  
- 对文档路径表达式和常数使用相等运算符 ( = )。  
  
- 对文档路径表达式和数字常数使用范围运算符 (<, \<=, >, >=)。  
  
- 文档路径表达式代表任何可从引用的数据库容器识别文档中常量路径的表达式。  
  
  文档路径表达式  
  
  文档路径表达式代表属性或数组索引器评估者在数据库容器文档中的一个文档当中的路径。 此路径可用于直接在数据库容器中的文档内识别筛选器中引用的值的位置。  
  
  若要将表达式视为一个文档路径表达式，它应：  
  
1.  直接引用容器根。  
  
2.  引用某个文档路径表达式的属性或常数数组索引器  
  
3.  引用表示某个文档路径表达式的别名。  
  
     **语法约定**  
  
     下表描述的约定用于说明以下 SQL 引用中的语法。  
  
    |约定|用途|  
    |-|-|    
    |大写|不区分大小写的关键字。|  
    |小写|区分大小写的关键字。|  
    |\<非终止符 >|单独定义的非终止符。|  
    |\<非终止符 > ::=|非终止符的语法定义。|  
    |other_terminal|详细说明的终端（令牌）。|  
    |标识符|标识符。 仅允许以下字符：a-z A-Z 0-9 _第一个字符不能为数字。|  
    |“字符串”|带引号的字符串。 允许任何有效的字符串。 请参阅 string_literal 的说明。|  
    |“符号”|属于语法的一部分的文本符号。|  
    |&#124;（垂直条）|语法项的替代项。 只可使用其中一个指定的项。|  
    |[ ] /（括号）|括号项可将一个或多个可选项括起来。|  
    |[ ,...n ]|表示前面的项可以重复 n 次。 匹配项由逗号分隔。|  
    |[ ...n ]|表示上述项可重复 n 次。 匹配项由空格分隔。|  
  
##  <a name="bk_built_in_functions"></a> 内置函数  
 Cosmos DB 提供多个内置 SQL 函数。 内置函数的类别如下所示。  
  
|函数|描述|  
|--------------|-----------------|  
|[数学函数](#bk_mathematical_functions)|每个数学函数均执行一个计算，通常基于作为参数提供的输出值，并返回数值。|  
|[类型检查函数](#bk_type_checking_functions)|类型检查函数允许检查 SQL 查询内表达式的类型。|  
|[字符串函数](#bk_string_functions)|该字符串函数对字符串输入值执行操作，并返回字符串、数值或布尔值。|  
|[数组函数](#bk_array_functions)|该数组函数对数组输入值执行操作，并返回数值、布尔值或数组值。|
|[日期和时间函数](#bk_date_and_time_functions)|日期和时间函数，您可以在两个窗体; 中获取当前的 UTC 日期和时间其值是以毫秒为单位或作为一个字符串，它符合 ISO 8601 格式的 Unix epoch 数字时间戳。|
|[空间函数](#bk_spatial_functions)|该空间函数对控件对象输入值执行操作，并返回数值或布尔值。|  
  
###  <a name="bk_mathematical_functions"></a> 数学函数  
 以下函数均执行一个计算，通常基于作为参数提供的输出值，并返回数值。  
  
||||  
|-|-|-|  
|[ABS](#bk_abs)|[ACOS](#bk_acos)|[ASIN](#bk_asin)|  
|[ATAN](#bk_atan)|[ATN2](#bk_atn2)|[CEILING](#bk_ceiling)|  
|[COS](#bk_cos)|[COT](#bk_cot)|[DEGREES](#bk_degrees)|  
|[EXP](#bk_exp)|[FLOOR](#bk_floor)|[LOG](#bk_log)|  
|[LOG10](#bk_log10)|[PI](#bk_pi)|[POWER](#bk_power)|  
|[RADIANS](#bk_radians)|[ROUND](#bk_round)|[SIN](#bk_sin)|  
|[SQRT](#bk_sqrt)|[SQUARE](#bk_square)|[SIGN](#bk_sign)|  
|[TAN](#bk_tan)|[TRUNC](#bk_trunc)||  
  
####  <a name="bk_abs"></a> ABS  
 返回指定数值表达式的绝对（正）值。  
  
 **语法**  
  
```  
ABS (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例显示了对三个不同数字使用 ABS 函数的结果。  
  
```  
SELECT ABS(-1) AS abs1, ABS(0) AS abs2, ABS(1) AS abs3 
```  
  
 结果集如下。  
  
```  
[{abs1: 1, abs2: 0, abs3: 1}]  
```  
  
####  <a name="bk_acos"></a> ACOS  
 返回角度（弧度），其余弦是指定的数值表达式；也被称为反余弦。  
  
 **语法**  
  
```  
ACOS(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回 -1 的反余弦值。  
  
```  
SELECT ACOS(-1) AS acos 
```  
  
 结果集如下。  
  
```  
[{"acos": 3.1415926535897931}]  
```  
  
####  <a name="bk_asin"></a> ASIN  
 返回角度（弧度），其正弦是指定的数值表达式。 也被称为反正弦。  
  
 **语法**  
  
```  
ASIN(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回  -1 的反正弦值。  
  
```  
SELECT ASIN(-1) AS asin  
```  
  
 结果集如下。  
  
```  
[{"asin": -1.5707963267948966}]  
```  
  
####  <a name="bk_atan"></a> ATAN  
 返回角度（弧度），其正切是指定的数值表达式。 这也被称为反正切。  
  
 **语法**  
  
```  
ATAN(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回指定值的反正切值。  
  
```  
SELECT ATAN(-45.01) AS atan  
```  
  
 结果集如下。  
  
```  
[{"atan": -1.5485826962062663}]  
```  
  
####  <a name="bk_atn2"></a> ATN2  
 返回 y/x 的反正切函数的主值（以弧度表示）。  
  
 **语法**  
  
```  
ATN2(<numeric_expression>, <numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例可计算指定的 x 分量和 y 分量的 ATN2 值。  
  
```  
SELECT ATN2(35.175643, 129.44) AS atn2  
```  
  
 结果集如下。  
  
```  
[{"atn2": 1.3054517947300646}]  
```  
  
####  <a name="bk_ceiling"></a> CEILING  
 返回大于或等于指定数值表达式的最小整数值。  
  
 **语法**  
  
```  
CEILING (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例显示了 CEILING 函数的正数、负数和零值。  
  
```  
SELECT CEILING(123.45) AS c1, CEILING(-123.45) AS c2, CEILING(0.0) AS c3  
```  
  
 结果集如下。  
  
```  
[{c1: 124, c2: -123, c3: 0}]  
```  
  
####  <a name="bk_cos"></a> COS  
 返回指定表达式中指定角度的三角余弦（弧度）。  
  
 **语法**  
  
```  
COS(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例可计算指定角的余弦值。  
  
```  
SELECT COS(14.78) AS cos  
```  
  
 结果集如下。  
  
```  
[{"cos": -0.59946542619465426}]  
```  
  
####  <a name="bk_cot"></a> COT  
 返回指定数值表达式中指定角度的三角余切。  
  
 **语法**  
  
```  
COT(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例可计算指定角的余切值。  
  
```  
SELECT COT(124.1332) AS cot  
```  
  
 结果集如下。  
  
```  
[{"cot": -0.040311998371148884}]  
```  
  
####  <a name="bk_degrees"></a> DEGREES  
 返回指定角度（弧度）的相应角度（度）。  
  
 **语法**  
  
```  
DEGREES (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回 PI/2 弧度角的度数。  
  
```  
SELECT DEGREES(PI()/2) AS degrees  
```  
  
 结果集如下。  
  
```  
[{"degrees": 90}]  
```  
  
####  <a name="bk_floor"></a> FLOOR  
 返回小于或等于指定数值表达式的最大整数。  
  
 **语法**  
  
```  
FLOOR (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例显示了 FLOOR 函数的正数、负数和零值。  
  
```  
SELECT FLOOR(123.45) AS fl1, FLOOR(-123.45) AS fl2, FLOOR(0.0) AS fl3  
```  
  
 结果集如下。  
  
```  
[{fl1: 123, fl2: -124, fl3: 0}]  
```  
  
####  <a name="bk_exp"></a> EXP  
 返回指定数值表达式的指数值。  
  
 **语法**  
  
```  
EXP (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回一个数值表达式。  
  
  **注释**  
  
  常数 e  (2.718281…) 是自然对数的底数。  
  
  某个数为指数表示常数 e 自乘的次数为该数。 例如 EXP(1.0) = e^1.0 = 2.71828182845905，EXP(10) = e^10 = 22026.4657948067。  
  
  某个数的自然对数的指数就是该数本身：EXP (LOG (n)) = n。 并且某个数的指数的自然对数也是该数本身：LOG (EXP (n)) = n。  
  
  **示例**  
  
  以下示例声明一个变量并返回到指定变量 (10) 的指数值。  
  
```  
SELECT EXP(10) AS exp  
```  
  
 结果集如下。  
  
```  
[{exp: 22026.465794806718}]  
```  
  
 以下示例将返回 20 的自然对数的指数值以及 20 的指数的自然对数。 由于这些函数互为反函数，因此在这两种情况下，用于浮点数学舍入的返回值均为 20。  
  
```  
SELECT EXP(LOG(20)) AS exp1, LOG(EXP(20)) AS exp2  
```  
  
 结果集如下。  
  
```  
[{exp1: 19.999999999999996, exp2: 20}]  
```  
  
####  <a name="bk_log"></a> LOG  
 返回指定数值表达式的自然对数。  
  
 **语法**  
  
```  
LOG (<numeric_expression> [, <base>])  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   为数值表达式。  
  
- `base`  
  
   设置对数底数的可选数值参数。  
  
  返回类型  
  
  返回一个数值表达式。  
  
  **备注**  
  
  默认情况下，LOG() 返回自然对数。 可以使用可选底数参数将对数的底数改为另一个值。  
  
  自然对数是以 e 为底数的对数，其中 e 是一个约等于2.718281828 的无理数常数。  
  
  某个数的指数的自然对数也是该数本身：LOG( EXP( n ) ) = n。 并且某个数的自然对数的指数就是该数本身：EXP( LOG( n ) ) = n。  
  
  **示例**  
  
  以下示例声明一个变量并返回指定变量 (10) 的对数值。  
  
```  
SELECT LOG(10) AS log  
```  
  
 结果集如下。  
  
```  
[{log: 2.3025850929940459}]  
```  
  
 以下示例可计算某个数的指数的对数值。  
  
```  
SELECT EXP(LOG(10)) AS expLog  
```  
  
 结果集如下。  
  
```  
[{expLog: 10.000000000000002}]  
```  
  
####  <a name="bk_log10"></a> LOG10  
 返回指定数值表达式中以 10 为底数的对数。  
  
 **语法**  
  
```  
LOG10 (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回一个数值表达式。  
  
  **注释**  
  
  LOG10 和 POWER 函数互为反函数。 例如，10 ^ LOG10(n) = n。  
  
  **示例**  
  
  以下示例声明一个变量并返回到指定变量 (100) 的 LOG10 值。  
  
```  
SELECT LOG10(100) AS log10 
```  
  
 结果集如下。  
  
```  
[{log10: 2}]  
```  
  
####  <a name="bk_pi"></a> PI  
 返回 PI 的常数值。  
  
 **语法**  
  
```  
PI ()  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回 PI 值。  
  
```  
SELECT PI() AS pi 
```  
  
 结果集如下。  
  
```  
[{"pi": 3.1415926535897931}]  
```  
  
####  <a name="bk_power"></a> POWER  
 返回指定表达式相对指定幂的值。  
  
 **语法**  
  
```  
POWER (<numeric_expression>, <y>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   为数值表达式。  
  
- `y`  
  
   为 `numeric_expression` 自乘得到的幂。  
  
  **返回类型**  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例演示了某个数自乘 3 次的幂（该数的立方）。  
  
```  
SELECT POWER(2, 3) AS pow1, POWER(2.5, 3) AS pow2  
```  
  
 结果集如下。  
  
```  
[{pow1: 8, pow2: 15.625}]  
```  
  
####  <a name="bk_radians"></a> RADIANS  
 返回输入的数值表达式（度）的弧度。  
  
 **语法**  
  
```  
RADIANS (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将几个角作为输入，然后返回其对应的弧度值。  
  
```  
SELECT RADIANS(-45.01) AS r1, RADIANS(-181.01) AS r2, RADIANS(0) AS r3, RADIANS(0.1472738) AS r4, RADIANS(197.1099392) AS r5  
```  
  
  结果集如下。  
  
```  
[{  
       "r1": -0.7855726963226477,  
       "r2": -3.1592204790349356,  
       "r3": 0,  
       "r4": 0.0025704127119236249,  
       "r5": 3.4402174274458375  
   }]  
```  
  
####  <a name="bk_round"></a> ROUND  
 返回一个数值，四舍五入到最接近的整数值。  
  
 **语法**  
  
```  
ROUND(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回一个数值表达式。  
  
  **注释**
  
  执行舍入运算遵循远离零方向舍入的中点。 如果输入是一个数值表达式，此范围恰好两个整数结果将是从零开始的最接近的整数值。  
  
  |<numeric_expression>|舍入|
  |-|-|
  |-6.5000|-7|
  |-0.5|-1|
  |0.5|第|
  |6.5000|7||
  
  **示例**  
  
  以下示例将以下的正数和负数舍入到最接近的整数。  
  
```  
SELECT ROUND(2.4) AS r1, ROUND(2.6) AS r2, ROUND(2.5) AS r3, ROUND(-2.4) AS r4, ROUND(-2.6) AS r5  
```  
  
  结果集如下。  
  
```  
[{r1: 2, r2: 3, r3: 3, r4: -2, r5: -3}]  
```  
  
####  <a name="bk_sign"></a> SIGN  
 返回指定数值表达式的正数 (+1)、零 (0) 或负数 (-1)。  
  
 **语法**  
  
```  
SIGN(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回数字 -2 到 2 的符号函数值。  
  
```  
SELECT SIGN(-2) AS s1, SIGN(-1) AS s2, SIGN(0) AS s3, SIGN(1) AS s4, SIGN(2) AS s5  
```  
  
 结果集如下。  
  
```  
[{s1: -1, s2: -1, s3: 0, s4: 1, s5: 1}]  
```  
  
####  <a name="bk_sin"></a> SIN  
 返回指定表达式中指定角度的三角正弦（弧度）。  
  
 **语法**  
  
```  
SIN(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例可计算指定角的正弦值。  
  
```  
SELECT SIN(45.175643) AS sin  
```  
  
 结果集如下。  
  
```  
[{"sin": 0.929607286611012}]  
```  
  
####  <a name="bk_sqrt"></a> SQRT  
 返回指定数值的平方根。  
  
 **语法**  
  
```  
SQRT(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回数字 1-3 的平方根。  
  
```  
SELECT SQRT(1) AS s1, SQRT(2.0) AS s2, SQRT(3) AS s3  
```  
  
 结果集如下。  
  
```  
[{s1: 1, s2: 1.4142135623730952, s3: 1.7320508075688772}]  
```  
  
####  <a name="bk_square"></a> SQUARE  
 返回指定数字值的平方。  
  
 **语法**  
  
```  
SQUARE(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回数字 1-3 的平方。  
  
```  
SELECT SQUARE(1) AS s1, SQUARE(2.0) AS s2, SQUARE(3) AS s3  
```  
  
 结果集如下。  
  
```  
[{s1: 1, s2: 4, s3: 9}]  
```  
  
####  <a name="bk_tan"></a> TAN  
 返回指定表达式中指定角度（弧度）的正切值。  
  
 **语法**  
  
```  
TAN (<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例可计算 PI()/2 的正切值。  
  
```  
SELECT TAN(PI()/2) AS tan 
```  
  
 结果集如下。  
  
```  
[{"tan": 16331239353195370 }]  
```  
  
####  <a name="bk_trunc"></a> TRUNC  
 返回一个数值，截断到最接近的整数值。  
  
 **语法**  
  
```  
TRUNC(<numeric_expression>)  
```  
  
 **参数**  
  
- `numeric_expression`  
  
   是一个数值表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将以下的正数和负数截断为最接近的整数。  
  
```  
SELECT TRUNC(2.4) AS t1, TRUNC(2.6) AS t2, TRUNC(2.5) AS t3, TRUNC(-2.4) AS t4, TRUNC(-2.6) AS t5  
```  
  
 结果集如下。  
  
```  
[{t1: 2, t2: 2, t3: 2, t4: -2, t5: -2}]  
```  
  
###  <a name="bk_type_checking_functions"></a> 类型检查函数  
 以下函数支持对输入值进行类型检查，并且每个都会返回一个布尔值。  
  
||||  
|-|-|-|  
|[IS_ARRAY](#bk_is_array)|[IS_BOOL](#bk_is_bool)|[IS_DEFINED](#bk_is_defined)|  
|[IS_NULL](#bk_is_null)|[IS_NUMBER](#bk_is_number)|[IS_OBJECT](#bk_is_object)|  
|[IS_PRIMITIVE](#bk_is_primitive)|[IS_STRING](#bk_is_string)||  
  
####  <a name="bk_is_array"></a> IS_ARRAY  
 返回一个布尔值，指示指定表达式类型是否为数组。  
  
 **语法**  
  
```  
IS_ARRAY(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_ARRAY 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
 IS_ARRAY(true) AS isArray1,   
 IS_ARRAY(1) AS isArray2,  
 IS_ARRAY("value") AS isArray3,  
 IS_ARRAY(null) AS isArray4,  
 IS_ARRAY({prop: "value"}) AS isArray5,   
 IS_ARRAY([1, 2, 3]) AS isArray6,  
 IS_ARRAY({prop: "value"}.prop2) AS isArray7  
```  
  
 结果集如下。  
  
```  
[{"isArray1":false,"isArray2":false,"isArray3":false,"isArray4":false,"isArray5":false,"isArray6":true,"isArray7":false}]
```  
  
####  <a name="bk_is_bool"></a> IS_BOOL  
 返回一个布尔值，指示指定表达式的类型是否为布尔表达式。  
  
 **语法**  
  
```  
IS_BOOL(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_BOOL 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
    IS_BOOL(true) AS isBool1,   
    IS_BOOL(1) AS isBool2,  
    IS_BOOL("value") AS isBool3,   
    IS_BOOL(null) AS isBool4,  
    IS_BOOL({prop: "value"}) AS isBool5,   
    IS_BOOL([1, 2, 3]) AS isBool6,  
    IS_BOOL({prop: "value"}.prop2) AS isBool7  
```  
  
 结果集如下。  
  
```  
[{"isBool1":true,"isBool2":false,"isBool3":false,"isBool4":false,"isBool5":false,"isBool6":false,"isBool7":false}]
```  
  
####  <a name="bk_is_defined"></a> IS_DEFINED  
 返回一个布尔，它指示属性是否已经分配了值。  
  
 **语法**  
  
```  
IS_DEFINED(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例检查了指定的 JSON 文档内是否存在属性。 由于存在 "a"，第一个会返回 true，但由于不存在 "b"，第二个会返回 false。  
  
```  
SELECT IS_DEFINED({ "a" : 5 }.a) AS isDefined1, IS_DEFINED({ "a" : 5 }.b) AS isDefined2 
```  
  
 结果集如下。  
  
```  
[{"isDefined1":true,"isDefined2":false}]  
```  
  
####  <a name="bk_is_null"></a> IS_NULL  
 返回一个布尔值，指示指定表达式的类型是否为 null。  
  
 **语法**  
  
```  
IS_NULL(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_NULL 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
    IS_NULL(true) AS isNull1,   
    IS_NULL(1) AS isNull2,  
    IS_NULL("value") AS isNull3,   
    IS_NULL(null) AS isNull4,  
    IS_NULL({prop: "value"}) AS isNull5,   
    IS_NULL([1, 2, 3]) AS isNull6,  
    IS_NULL({prop: "value"}.prop2) AS isNull7  
```  
  
 结果集如下。  
  
```  
[{"isNull1":false,"isNull2":false,"isNull3":false,"isNull4":true,"isNull5":false,"isNull6":false,"isNull7":false}]
```  
  
####  <a name="bk_is_number"></a> IS_NUMBER  
 返回一个布尔值，指示指定表达式的类型是否为数字。  
  
 **语法**  
  
```  
IS_NUMBER(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_NULL 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
    IS_NUMBER(true) AS isNum1,   
    IS_NUMBER(1) AS isNum2,  
    IS_NUMBER("value") AS isNum3,   
    IS_NUMBER(null) AS isNum4,  
    IS_NUMBER({prop: "value"}) AS isNum5,   
    IS_NUMBER([1, 2, 3]) AS isNum6,  
    IS_NUMBER({prop: "value"}.prop2) AS isNum7  
```  
  
 结果集如下。  
  
```  
[{"isNum1":false,"isNum2":true,"isNum3":false,"isNum4":false,"isNum5":false,"isNum6":false,"isNum7":false}]  
```  
  
####  <a name="bk_is_object"></a> IS_OBJECT  
 返回一个布尔值，指示指定表达式的类型是否为 JSON 对象。  
  
 **语法**  
  
```  
IS_OBJECT(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_OBJECT 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
    IS_OBJECT(true) AS isObj1,   
    IS_OBJECT(1) AS isObj2,  
    IS_OBJECT("value") AS isObj3,   
    IS_OBJECT(null) AS isObj4,  
    IS_OBJECT({prop: "value"}) AS isObj5,   
    IS_OBJECT([1, 2, 3]) AS isObj6,  
    IS_OBJECT({prop: "value"}.prop2) AS isObj7  
```  
  
 结果集如下。  
  
```  
[{"isObj1":false,"isObj2":false,"isObj3":false,"isObj4":false,"isObj5":true,"isObj6":false,"isObj7":false}]
```  
  
####  <a name="bk_is_primitive"></a> IS_PRIMITIVE  
 返回一个布尔值，指示指定表达式的类型是否为一个（字符串、布尔、数值或 null）。  
  
 **语法**  
  
```  
IS_PRIMITIVE(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_PRIMITIVE 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
           IS_PRIMITIVE(true) AS isPrim1,   
           IS_PRIMITIVE(1) AS isPrim2,  
           IS_PRIMITIVE("value") AS isPrim3,   
           IS_PRIMITIVE(null) AS isPrim4,  
           IS_PRIMITIVE({prop: "value"}) AS isPrim5,   
           IS_PRIMITIVE([1, 2, 3]) AS isPrim6,  
           IS_PRIMITIVE({prop: "value"}.prop2) AS isPrim7  
```  
  
 结果集如下。  
  
```  
[{"isPrim1": true, "isPrim2": true, "isPrim3": true, "isPrim4": true, "isPrim5": false, "isPrim6": false, "isPrim7": false}]  
```  
  
####  <a name="bk_is_string"></a> IS_STRING  
 返回一个布尔值，指示指定表达式的类型是否为字符串。  
  
 **语法**  
  
```  
IS_STRING(<expression>)  
```  
  
 **参数**  
  
- `expression`  
  
   为任何有效的表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例使用 IS_STRING 函数检查了 JSON 布尔值、数字、字符串、null、对象、数组和未定义类型的对象。  
  
```  
SELECT   
       IS_STRING(true) AS isStr1,   
       IS_STRING(1) AS isStr2,  
       IS_STRING("value") AS isStr3,   
       IS_STRING(null) AS isStr4,  
       IS_STRING({prop: "value"}) AS isStr5,   
       IS_STRING([1, 2, 3]) AS isStr6,  
       IS_STRING({prop: "value"}.prop2) AS isStr7  
```  
  
 结果集如下。  
  
```  
[{"isStr1":false,"isStr2":false,"isStr3":true,"isStr4":false,"isStr5":false,"isStr6":false,"isStr7":false}] 
```  
  
###  <a name="bk_string_functions"></a> 字符串函数  
 下面的标量函数对字符串输入值执行操作，并返回字符串、数值或布尔值。  
  
||||  
|-|-|-|  
|[CONCAT](#bk_concat)|[CONTAINS](#bk_contains)|[ENDSWITH](#bk_endswith)|  
|[INDEX_OF](#bk_index_of)|[LEFT](#bk_left)|[LENGTH](#bk_length)|  
|[LOWER](#bk_lower)|[LTRIM](#bk_ltrim)|[REPLACE](#bk_replace)|  
|[REPLICATE](#bk_replicate)|[REVERSE](#bk_reverse)|[RIGHT](#bk_right)|  
|[RTRIM](#bk_rtrim)|[STARTSWITH](#bk_startswith)|[StringToArray](#bk_stringtoarray)|
|[StringToBoolean](#bk_stringtoboolean)|[StringToNull](#bk_stringtonull)|[StringToNumber](#bk_stringtonumber)|
|[StringToObject](#bk_stringtoobject)|[SUBSTRING](#bk_substring)|[ToString](#bk_tostring)|
|[TRIM](#bk_trim)|[UPPER](#bk_upper)||
  
####  <a name="bk_concat"></a> CONCAT  
 返回一个字符串，该字符串是连接两个或多个字符串值的结果。  
  
 **语法**  
  
```  
CONCAT(<str_expr>, <str_expr> [, <str_expr>])  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例将返回指定值的串联字符串。  
  
```  
SELECT CONCAT("abc", "def") AS concat  
```  
  
 结果集如下。  
  
```  
[{"concat": "abcdef"}  
```  
  
####  <a name="bk_contains"></a> CONTAINS  
 返回一个布尔值，该值指示第一个字符串表达式是否包含第二个字符串表达式。  
  
 **语法**  
  
```  
CONTAINS(<str_expr>, <str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例检查了 "abc" 是否包含 "ab" 且包含 "d"。  
  
```  
SELECT CONTAINS("abc", "ab") AS c1, CONTAINS("abc", "d") AS c2 
```  
  
 结果集如下。  
  
```  
[{"c1": true, "c2": false}]  
```  
  
####  <a name="bk_endswith"></a> ENDSWITH  
 返回一个布尔值，指示第一个字符串表达式是否以第二个字符串表达式结尾。  
  
 **语法**  
  
```  
ENDSWITH(<str_expr>, <str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例将返回以 "b" 和 "bc" 结尾的 "abc"。  
  
```  
SELECT ENDSWITH("abc", "b") AS e1, ENDSWITH("abc", "bc") AS e2 
```  
  
 结果集如下。  
  
```  
[{"e1": false, "e2": true}]  
```  
  
####  <a name="bk_index_of"></a> INDEX_OF  
 返回第一个指定的字符串表达式中第一次出现第二个字符串表达式的起始位置，如果未找到字符串，则返回 -1。  
  
 **语法**  
  
```  
INDEX_OF(<str_expr>, <str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例将返回 "abc" 内不同子字符串的索引。  
  
```  
SELECT INDEX_OF("abc", "ab") AS i1, INDEX_OF("abc", "b") AS i2, INDEX_OF("abc", "c") AS i3 
```  
  
 结果集如下。  
  
```  
[{"i1": 0, "i2": 1, "i3": -1}]  
```  
  
####  <a name="bk_left"></a> LEFT  
 返回具有指定字符数的字符串的左侧部分。  
  
 **语法**  
  
```  
LEFT(<str_expr>, <num_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
- `num_expr`  
  
   为任何有效的数值表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例将返回不同长度值的 "abc" 的左侧部分。  
  
```  
SELECT LEFT("abc", 1) AS l1, LEFT("abc", 2) AS l2 
```  
  
 下面是结果集。  
  
```  
[{"l1": "a", "l2": "ab"}]  
```  
  
####  <a name="bk_length"></a> LENGTH  
 返回指定字符串表达式的字符数。  
  
 **语法**  
  
```  
LENGTH(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例将返回字符串的长度。  
  
```  
SELECT LENGTH("abc") AS len 
```  
  
 结果集如下。  
  
```  
[{"len": 3}]  
```  
  
####  <a name="bk_lower"></a> LOWER  
 返回在将大写字符数据转换为小写后的字符串表达式。  
  
 **语法**  
  
```  
LOWER(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 LOWER。  
  
```  
SELECT LOWER("Abc") AS lower
```  
  
 结果集如下。  
  
```  
[{"lower": "abc"}]  
  
```  
  
####  <a name="bk_ltrim"></a> LTRIM  
 返回删除前导空格后的字符串表达式。  
  
 **语法**  
  
```  
LTRIM(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 LTRIM。  
  
```  
SELECT LTRIM("  abc") AS l1, LTRIM("abc") AS l2, LTRIM("abc   ") AS l3 
```  
  
 结果集如下。  
  
```  
[{"l1": "abc", "l2": "abc", "l3": "abc   "}]  
```  
  
####  <a name="bk_replace"></a> REPLACE  
 将出现的所有指定字符串值替换为另一个字符串值。  
  
 **语法**  
  
```  
REPLACE(<str_expr>, <str_expr>, <str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 REPLACE。  
  
```  
SELECT REPLACE("This is a Test", "Test", "desk") AS replace 
```  
  
 结果集如下。  
  
```  
[{"replace": "This is a desk"}]  
```  
  
####  <a name="bk_replicate"></a> REPLICATE  
 将一个字符串值重复指定的次数。  
  
 **语法**  
  
```  
REPLICATE(<str_expr>, <num_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
- `num_expr`  
  
   为任何有效的数值表达式。 如果 num_expr 为负或非有限，则结果未定义。

  > [!NOTE]
  > 结果的最大长度为 10,000 个字符，即 (length(str_expr)  *  num_expr) <= 10,000。
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 REPLICATE。  
  
```  
SELECT REPLICATE("a", 3) AS replicate  
```  
  
 结果集如下。  
  
```  
[{"replicate": "aaa"}]  
```  
  
####  <a name="bk_reverse"></a> REVERSE  
 返回字符串值的逆序排序形式。  
  
 **语法**  
  
```  
REVERSE(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 REVERSE。  
  
```  
SELECT REVERSE("Abc") AS reverse  
```  
  
 结果集如下。  
  
```  
[{"reverse": "cbA"}]  
```  
  
####  <a name="bk_right"></a> RIGHT  
 返回具有指定字符数的字符串的右侧部分。  
  
 **语法**  
  
```  
RIGHT(<str_expr>, <num_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
- `num_expr`  
  
   为任何有效的数值表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例将返回不同长度值的 "abc" 的右侧部分。  
  
```  
SELECT RIGHT("abc", 1) AS r1, RIGHT("abc", 2) AS r2 
```  
  
 结果集如下。  
  
```  
[{"r1": "c", "r2": "bc"}]  
```  
  
####  <a name="bk_rtrim"></a> RTRIM  
 在删除尾随空格后，返回字符串表达式。  
  
 **语法**  
  
```  
RTRIM(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 RTRIM。  
  
```  
SELECT RTRIM("  abc") AS r1, RTRIM("abc") AS r2, RTRIM("abc   ") AS r3  
```  
  
 结果集如下。  
  
```  
[{"r1": "   abc", "r2": "abc", "r3": "abc"}]  
```  
  
####  <a name="bk_startswith"></a> STARTSWITH  
 返回一个布尔值，指示第一个字符串表达式是否以第二个字符串表达式开头。  
  
 **语法**  
  
```  
STARTSWITH(<str_expr>, <str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例检查了字符串 "abc" 是否以 "b" 和 "a" 开头。  
  
```  
SELECT STARTSWITH("abc", "b") AS s1, STARTSWITH("abc", "a") AS s2  
```  
  
 结果集如下。  
  
```  
[{"s1": false, "s2": true}]  
```  

  ####  <a name="bk_stringtoarray"></a> StringToArray  
 返回已转换为数组的表达式。 如果表达式无法转换，则返回未定义的表达式。  
  
 **语法**  
  
```  
StringToArray(<expr>)  
```  
  
 **参数**  
  
- `expr`  
  
   是否会将任何有效的标量表达式作为 JSON 数组表达式来计算？ 请注意，嵌套字符串值必须使用双引号编写，否则无效。 有关 JSON 格式的详细信息，请参阅 [json.org](https://json.org/)
  
  返回类型  
  
  返回一个数组表达式或未定义的表达式。  
  
  **示例**  
  
  以下示例演示 StringToArray 在不同类型中的行为方式。 
  
 下面是输入有效的示例。

```
SELECT 
    StringToArray('[]') AS a1, 
    StringToArray("[1,2,3]") AS a2,
    StringToArray("[\"str\",2,3]") AS a3,
    StringToArray('[["5","6","7"],["8"],["9"]]') AS a4,
    StringToArray('[1,2,3, "[4,5,6]",[7,8]]') AS a5
```

结果集如下。

```
[{"a1": [], "a2": [1,2,3], "a3": ["str",2,3], "a4": [["5","6","7"],["8"],["9"]], "a5": [1,2,3,"[4,5,6]",[7,8]]}]
```

下面是输入无效的示例。 
   
 在数组中使用单引号不是有效的 JSON。
即使它们在查询中有效，系统也不会将其解析为有效数组。 必须将数组字符串中的字符串转义为 "[\\"\\"]"，否则其引号必须为单个 '[""]'。

```
SELECT
    StringToArray("['5','6','7']")
```

结果集如下。

```
[{}]
```

下面是输入无效的示例。
   
 传递的表达式将会解析为 JSON 数组；下面的示例不会计算为类型数组，因此返回未定义。
   
```
SELECT
    StringToArray("["),
    StringToArray("1"),
    StringToArray(NaN),
    StringToArray(false),
    StringToArray(undefined)
```

结果集如下。

```
[{}]
```

####  <a name="bk_stringtoboolean"></a> StringToBoolean  
 返回已转换为布尔值的表达式。 如果表达式无法转换，则返回未定义的表达式。  
  
 **语法**  
  
```  
StringToBoolean(<expr>)  
```  
  
 **参数**  
  
- `expr`  
  
   是否会将任何有效的标量表达式作为布尔表达式来计算？  
  
  返回类型  
  
  返回一个布尔表达式或未定义的表达式。  
  
  **示例**  
  
  以下示例演示 StringToBoolean 在不同类型中的行为方式。 
 
 下面是输入有效的示例。

只能在 "true"/"false" 之前或之后使用空格。

```  
SELECT 
    StringToBoolean("true") AS b1, 
    StringToBoolean("    false") AS b2,
    StringToBoolean("false    ") AS b3
```  
  
 结果集如下。  
  
```  
[{"b1": true, "b2": false, "b3": false}]
```  

下面是输入无效的示例。

 布尔值区分大小写，必须全用小写字符（即 "true" 和 "false"）来表示。

```  
SELECT 
    StringToBoolean("TRUE"),
    StringToBoolean("False")
```  

结果集如下。  
  
```  
[{}]
``` 

传递的表达式将会解析为布尔表达式；以下输入不会计算为布尔类型，因此会返回未定义。

```  
SELECT 
    StringToBoolean("null"),
    StringToBoolean(undefined),
    StringToBoolean(NaN), 
    StringToBoolean(false), 
    StringToBoolean(true)
```  

结果集如下。  
  
```  
[{}]
```  

####  <a name="bk_stringtonull"></a> StringToNull  
 返回已转换为 Null 的表达式。 如果表达式无法转换，则返回未定义的表达式。  
  
 **语法**  
  
```  
StringToNull(<expr>)  
```  
  
 **参数**  
  
- `expr`  
  
   是否会将任何有效的标量表达式作为 Null 表达式来计算？
  
  返回类型  
  
  返回一个 Null 表达式或未定义的表达式。  
  
  **示例**  
  
  以下示例演示 StringToNull 在不同类型中的行为方式。 

下面是输入有效的示例。

 只能在 "null" 之前或之后使用空格。

```  
SELECT 
    StringToNull("null") AS n1, 
    StringToNull("  null ") AS n2,
    IS_NULL(StringToNull("null   ")) AS n3
```  
  
 结果集如下。  
  
```  
[{"n1": null, "n2": null, "n3": true}]
```  

下面是输入无效的示例。

Null 值区分大小写，必须全用小写字符（即 "null"）来表示。

```  
SELECT    
    StringToNull("NULL"),
    StringToNull("Null")
```  
  
 结果集如下。  
  
```  
[{}]
```  

传递的表达式将会解析为 null 表达式；以下输入不会计算为 null 类型，因此会返回未定义。

```  
SELECT    
    StringToNull("true"), 
    StringToNull(false), 
    StringToNull(undefined),
    StringToNull(NaN) 
```  
  
 结果集如下。  
  
```  
[{}]
```  

####  <a name="bk_stringtonumber"></a> StringToNumber  
 返回已转换为数字值的表达式。 如果表达式无法转换，则返回未定义的表达式。  
  
 **语法**  
  
```  
StringToNumber(<expr>)  
```  
  
 **参数**  
  
- `expr`  
  
   是否会将任何有效的标量表达式作为 JSON 数字表达式来计算？ JSON 中的数字必须是整数或浮点数。 有关 JSON 格式的详细信息，请参阅 [json.org](https://json.org/)  
  
  返回类型  
  
  返回一个数字表达式或未定义的表达式。  
  
  **示例**  
  
  以下示例演示 StringToNumber 在不同类型中的行为方式。 

只能在 Number 之前或之后使用空格。

```  
SELECT 
    StringToNumber("1.000000") AS num1, 
    StringToNumber("3.14") AS num2,
    StringToNumber("   60   ") AS num3, 
    StringToNumber("-1.79769e+308") AS num4
```  
  
 结果集如下。  
  
```  
{{"num1": 1, "num2": 3.14, "num3": 60, "num4": -1.79769e+308}}
```  

在 JSON 中，有效的 Number 必须是整数或浮点数。

```  
SELECT   
    StringToNumber("0xF")
```  
  
 结果集如下。  
  
```  
{{}}
```  

传递的表达式将会解析为 Number 表达式；以下输入不会计算为 Number 类型，因此会返回未定义。 

```  
SELECT 
    StringToNumber("99     54"),   
    StringToNumber(undefined),
    StringToNumber("false"),
    StringToNumber(false),
    StringToNumber(" "),
    StringToNumber(NaN)
```  
  
 结果集如下。  
  
```  
{{}}
```  

####  <a name="bk_stringtoobject"></a> StringToObject  
 返回已转换为对象的表达式。 如果表达式无法转换，则返回未定义的表达式。  
  
 **语法**  
  
```  
StringToObject(<expr>)  
```  
  
 **参数**  
  
- `expr`  
  
   是否会将任何有效的标量表达式作为 JSON 对象表达式来计算？ 请注意，嵌套字符串值必须使用双引号编写，否则无效。 有关 JSON 格式的详细信息，请参阅 [json.org](https://json.org/)  
  
  返回类型  
  
  返回一个对象表达式或未定义的表达式。  
  
  **示例**  
  
  以下示例演示 StringToObject 在不同类型中的行为方式。 
  
 下面是输入有效的示例。

``` 
SELECT 
    StringToObject("{}") AS obj1, 
    StringToObject('{"A":[1,2,3]}') AS obj2,
    StringToObject('{"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]}') AS obj3, 
    StringToObject("{\"C\":[{\"c1\":[5,6,7]},{\"c2\":8},{\"c3\":9}]}") AS obj4
``` 

结果集如下。

```
[{"obj1": {}, 
  "obj2": {"A": [1,2,3]}, 
  "obj3": {"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]},
  "obj4": {"C":[{"c1":[5,6,7]},{"c2":8},{"c3":9}]}}]
```

 下面是输入无效的示例。
即使它们在查询中有效，系统也不会将其解析为有效对象。 必须将对象字符串中的字符串转义为 "{\\"a\\":\\"str\\"}"，否则其引号必须为单个 '{"a": "str"}'。

属性名称的单引号不是有效的 JSON。

``` 
SELECT 
    StringToObject("{'a':[1,2,3]}")
```

结果集如下。

```  
[{}]
```  

没有引号的属性名称不是有效的 JSON。

``` 
SELECT 
    StringToObject("{a:[1,2,3]}")
```

结果集如下。

```  
[{}]
``` 

下面是输入无效的示例。

 传递的表达式将会解析为 JSON 对象；以下输入不会计算为对象类型，因此会返回未定义。

``` 
SELECT 
    StringToObject("}"),
    StringToObject("{"),
    StringToObject("1"),
    StringToObject(NaN), 
    StringToObject(false), 
    StringToObject(undefined)
``` 
 
 结果集如下。

```
[{}]
```

####  <a name="bk_substring"></a> SUBSTRING  
 返回字符串表达式的部分内容，该内容起于指定字符从零开始的位置，继续到指定长度或字符串结尾。  
  
 **语法**  
  
```  
SUBSTRING(<str_expr>, <num_expr>, <num_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
- `num_expr`  
  
   是表示开始和结束字符的任何有效数字表达式。    
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例将返回 "abc" 中从位置 1 开始且长度为 1 个字符的子字符串。  
  
```  
SELECT SUBSTRING("abc", 1, 1) AS substring  
```  
  
 结果集如下。  
  
```  
[{"substring": "b"}]  
```  
####  <a name="bk_tostring"></a> ToString  
 返回标量表达式的字符串表示形式。 
  
 **语法**  
  
```  
ToString(<expr>)
```  
  
 **参数**  
  
- `expr`  
  
   为任何有效的标量表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例演示 ToString 在不同类型中的行为方式。   
  
```  
SELECT 
    ToString(1.0000) AS str1, 
    ToString("Hello World") AS str2, 
    ToString(NaN) AS str3, 
    ToString(Infinity) AS str4,
    ToString(IS_STRING(ToString(undefined))) AS str5, 
    ToString(0.1234) AS str6, 
    ToString(false) AS str7, 
    ToString(undefined) AS str8
```  
  
 结果集如下。  
  
```  
[{"str1": "1", "str2": "Hello World", "str3": "NaN", "str4": "Infinity", "str5": "false", "str6": "0.1234", "str7": "false"}]  
```  
 给定以下输入：
```  
{"Products":[{"ProductID":1,"Weight":4,"WeightUnits":"lb"},{"ProductID":2,"Weight":32,"WeightUnits":"kg"},{"ProductID":3,"Weight":400,"WeightUnits":"g"},{"ProductID":4,"Weight":8999,"WeightUnits":"mg"}]}
```    
 以下示例演示 ToString 如何与其他字符串函数（如 CONCAT）一起使用。   
 
```  
SELECT 
CONCAT(ToString(p.Weight), p.WeightUnits) 
FROM p in c.Products 
```  

结果集如下。  
  
```  
[{"$1":"4lb" },
{"$1":"32kg"},
{"$1":"400g" },
{"$1":"8999mg" }]

```  
给定以下输入。
```
{"id":"08259","description":"Cereals ready-to-eat, KELLOGG, KELLOGG'S CRISPIX","nutrients":[{"id":"305","description":"Caffeine","units":"mg"},{"id":"306","description":"Cholesterol, HDL","nutritionValue":30,"units":"mg"},{"id":"307","description":"Sodium, NA","nutritionValue":612,"units":"mg"},{"id":"308","description":"Protein, ABP","nutritionValue":60,"units":"mg"},{"id":"309","description":"Zinc, ZN","nutritionValue":null,"units":"mg"}]}
```
以下示例演示 ToString 如何与其他字符串函数（如 REPLACE）一起使用。   
```
SELECT 
    n.id AS nutrientID,
    REPLACE(ToString(n.nutritionValue), "6", "9") AS nutritionVal
FROM food 
JOIN n IN food.nutrients
```
结果集如下。  
 ```
[{"nutrientID":"305"},
{"nutrientID":"306","nutritionVal":"30"},
{"nutrientID":"307","nutritionVal":"912"},
{"nutrientID":"308","nutritionVal":"90"},
{"nutrientID":"309","nutritionVal":"null"}]
``` 
 
####  <a name="bk_trim"></a> TRIM  
 返回删除前导空格和尾随空格后的字符串表达式。  
  
 **语法**  
  
```  
TRIM(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 TRIM。  
  
```  
SELECT TRIM("   abc") AS t1, TRIM("   abc   ") AS t2, TRIM("abc   ") AS t3, TRIM("abc") AS t4
```  
  
 结果集如下。  
  
```  
[{"t1": "abc", "t2": "abc", "t3": "abc", "t4": "abc"}]  
``` 
####  <a name="bk_upper"></a> UPPER  
 返回在将小写字符数据转换为大写后的字符串表达式。  
  
 **语法**  
  
```  
UPPER(<str_expr>)  
```  
  
 **参数**  
  
- `str_expr`  
  
   为任何有效的字符串表达式。  
  
  返回类型  
  
  返回字符串表达式。  
  
  **示例**  
  
  以下示例介绍了如何在查询中使用 UPPER。  
  
```  
SELECT UPPER("Abc") AS upper  
```  
  
 结果集如下。  
  
```  
[{"upper": "ABC"}]  
```  
  
###  <a name="bk_array_functions"></a> 数组函数  
 下面的标量函数对数组输入值执行操作，并返回数值、布尔值或数组值  
  
||||  
|-|-|-|  
|[ARRAY_CONCAT](#bk_array_concat)|[ARRAY_CONTAINS](#bk_array_contains)|[ARRAY_LENGTH](#bk_array_length)|  
|[ARRAY_SLICE](#bk_array_slice)|||  
  
####  <a name="bk_array_concat"></a> ARRAY_CONCAT  
 返回一个数组，该数组是连接两个或更多数组值的结果。  
  
 **语法**  
  
```  
ARRAY_CONCAT (<arr_expr>, <arr_expr> [, <arr_expr>])  
```  
  
 **参数**  
  
- `arr_expr`  
  
   为任何有效的数组表达式。  
  
  返回类型  
  
  返回数组表达式。  
  
  **示例**  
  
  以下示例介绍了如何连接两个数组。  
  
```  
SELECT ARRAY_CONCAT(["apples", "strawberries"], ["bananas"]) AS arrayConcat 
```  
  
 结果集如下。  
  
```  
[{"arrayConcat": ["apples", "strawberries", "bananas"]}]  
```  
  
####  <a name="bk_array_contains"></a> ARRAY_CONTAINS  
返回一个布尔，它指示数组是否包含指定的值。 可以通过在命令中使用布尔表达式来检查对象的部分匹配或完全匹配。 

**语法**  
  
```  
ARRAY_CONTAINS (<arr_expr>, <expr> [, bool_expr])  
```  
  
 **参数**  
  
- `arr_expr`  
  
   为任何有效的数组表达式。  
  
- `expr`  
  
   为任何有效的表达式。  

- `bool_expr`  
  
   为任何布尔表达式。 如果将其设置为“true”，并且指定的搜索值是对象，则该命令将检查部分匹配（搜索对象是其中一个对象的子集）。 如果将其设置为“false”，则该命令将检查数组中所有对象的完全匹配。 如果未指定，默认值为“false”。 
  
  返回类型  
  
  返回一个布尔值。  
  
  **示例**  
  
  以下示例介绍了如何使用 ARRAY_CONTAINS 检查数组内的成员资格。  
  
```  
SELECT   
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "apples") AS b1,  
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "mangoes") AS b2  
```  
  
 结果集如下。  
  
```  
[{"b1": true, "b2": false}]  
```  

以下示例介绍了如何使用 ARRAY_CONTAINS 检查数组内是否存在 JSON 字符串的部分匹配字符串。  
  
```  
SELECT  
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}, true) AS b1, 
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}) AS b2,
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "mangoes"}, true) AS b3 
```  
  
 结果集如下。  
  
```  
[{
  "b1": true,
  "b2": false,
  "b3": false
}] 
```  
  
####  <a name="bk_array_length"></a> ARRAY_LENGTH  
 返回指定数组表达式的元素数。  
  
 **语法**  
  
```  
ARRAY_LENGTH(<arr_expr>)  
```  
  
 **参数**  
  
- `arr_expr`  
  
   为任何有效的数组表达式。  
  
  返回类型  
  
  返回数值表达式。  
  
  **示例**  
  
  以下示例介绍了如何使用 ARRAY_LENGTH 获得数组的长度。  
  
```  
SELECT ARRAY_LENGTH(["apples", "strawberries", "bananas"]) AS len  
```  
  
 结果集如下。  
  
```  
[{"len": 3}]  
```  
  
####  <a name="bk_array_slice"></a> ARRAY_SLICE  
 返回部分数组表达式。
  
 **语法**  
  
```  
ARRAY_SLICE (<arr_expr>, <num_expr> [, <num_expr>])  
```  
  
 **参数**  
  
- `arr_expr`  
  
   为任何有效的数组表达式。  
  
- `num_expr`  
  
   用于开始数组的从零开始的数字索引。 负值可用于指定相对于数组最后一个元素的起始索引，即 -1 引用数组中最后一个元素。  

- `num_expr`  

   结果数组中的最大元素数。    

  返回类型  
  
  返回数组表达式。  
  
  **示例**  
  
  以下示例展示了如何使用 ARRAY_SLICE 获取数组的不同切片。  
  
```  
SELECT   
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1) AS s1,  
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1, 1) AS s2,
           ARRAY_SLICE(["apples", "strawberries", "bananas"], -2, 1) AS s3,
           ARRAY_SLICE(["apples", "strawberries", "bananas"], -2, 2) AS s4,
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1, 0) AS s5,
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1, 1000) AS s6,
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1, -100) AS s7      
  
```  
  
 结果集如下。  
  
```  
[{  
           "s1": ["strawberries", "bananas"],   
           "s2": ["strawberries"],
           "s3": ["strawberries"],  
           "s4": ["strawberries", "bananas"], 
           "s5": [],
           "s6": ["strawberries", "bananas"],
           "s7": [] 
}]  
```  

###  <a name="bk_date_and_time_functions"></a> 日期和时间函数
 以下标量函数，您可以在两个窗体; 中获取当前的 UTC 日期和时间其值是以毫秒为单位或作为一个字符串，它符合 ISO 8601 格式的 Unix epoch 数字时间戳。 

|||
|-|-|
|[GetCurrentDateTime](#bk_get_current_date_time)|[GetCurrentTimestamp](#bk_get_current_timestamp)||

####  <a name="bk_get_current_date_time"></a> GetCurrentDateTime
 返回当前 UTC 日期和时间以 ISO 8601 字符串形式。
  
 **语法**
  
```
GetCurrentDateTime ()
```
  
  返回类型
  
  返回当前 UTC 日期和时间 ISO 8601 字符串值。 

  这表示格式 YYYY MM DDThh:mm:ss.sssZ 其中：
  
  |||
  |-|-|
  |YYYY|四位数年份|
  |MM|两位数月份 (01 = 年 1 月，等等。)|
  |DD|两位数日期的月份 (01 到 31 之间)|
  |T|开始时元素为 signifier|
  |hh|两位数小时 (00 到 23)|
  |mm|两位数分钟 (00 到 59)|
  |ss|两位数秒数 (从 00 到 59)|
  |.sss|三位数字的秒的小数部分|
  |Z|UTC （协调世界时） 指示符||
  
  有关 ISO 8601 格式的更多详细信息，请参阅[ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)

  **注释**

  GetCurrentDateTime 是非确定性函数。 
  
  返回的结果是 UTC （协调世界时）。

  **示例**  
  
  下面的示例演示如何获取当前 UTC 日期时间使用 GetCurrentDateTime 内置函数。
  
```  
SELECT GetCurrentDateTime() AS currentUtcDateTime
```  
  
 下面是示例结果集。
  
```  
[{
  "currentUtcDateTime": "2019-05-03T20:36:17.784Z"
}]  
```  

####  <a name="bk_get_current_timestamp"></a> GetCurrentTimestamp
 返回 00:00:00 星期四，1 1970 年 1 月起经过的毫秒的数。 
  
 **语法**  
  
```  
GetCurrentTimestamp ()  
```  
  
  返回类型  
  
  返回数值的当前已经过自 Unix epoch 起经过即 00:00:00 星期四，1 1970 年 1 月起经过的毫秒数的毫秒数。

  **注释**

  GetCurrentTimestamp 是非确定性函数。 
  
  返回的结果是 UTC （协调世界时）。

  **示例**  
  
  下面的示例演示如何获取使用 GetCurrentTimestamp 内置函数的当前时间戳。
  
```  
SELECT GetCurrentTimestamp() AS currentUtcTimestamp
```  
  
 下面是示例结果集。
  
```  
[{
  "currentUtcTimestamp": 1556916469065
}]  
```  

###  <a name="bk_spatial_functions"></a> 空间函数  
 以下标量函数对标量对象输入值执行操作，并返回数值或布尔值。  
  
|||||
|-|-|-|-|
|[ST_DISTANCE](#bk_st_distance)|[ST_WITHIN](#bk_st_within)|[ST_INTERSECTS](#bk_st_intersects)|[ST_ISVALID](#bk_st_isvalid)|
|[ST_ISVALIDDETAILED](#bk_st_isvaliddetailed)||||
  
####  <a name="bk_st_distance"></a> ST_DISTANCE  
 返回两个 GeoJSON 点、多边形或 LineString 表达式之间的距离。  
  
 **语法**  
  
```  
ST_DISTANCE (<spatial_expr>, <spatial_expr>)  
```  
  
 **参数**  
  
- `spatial_expr`  
  
   为任何有效的 GeoJSON 点、多边形或 LineString 对象表达式。  
  
  返回类型  
  
  返回包含距离的数值表达式。 默认参考系统以米为单位表示。  
  
  **示例**  
  
  以下示例介绍了如何使用 ST_DISTANCE 内置函数返回在指定位置 30 km 内的所有家族文档。 .  
  
```  
SELECT f.id   
FROM Families f   
WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000  
```  
  
 结果集如下。  
  
```  
[{  
  "id": "WakefieldFamily"  
}]  
```  
  
####  <a name="bk_st_within"></a> ST_WITHIN  
 返回一个布尔表达式，指示第一个参数中指定的 GeoJSON 对象（点、多边形或 LineString）是否在第二个参数中的 GeoJSON（点、多边形或 LineString）内。  
  
 **语法**  
  
```  
ST_WITHIN (<spatial_expr>, <spatial_expr>)  
```  
  
 **参数**  
  
- `spatial_expr`  
  
   是任何有效的 GeoJSON 点、多边形或 LineString 对象表达式。  
 
- `spatial_expr`  
  
   为任何有效的 GeoJSON 点、多边形或 LineString 对象表达式。  
  
  返回类型  
  
  返回一个布尔值。  
  
  **示例**  
  
  以下示例介绍了如何使用 ST_WITHIN 查找多边形内的所有家族文档。  
  
```  
SELECT f.id   
FROM Families f   
WHERE ST_WITHIN(f.location, {  
    'type':'Polygon',   
    'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]  
})  
```  
  
 结果集如下。  
  
```  
[{ "id": "WakefieldFamily" }]  
```  

####  <a name="bk_st_intersects"></a> ST_INTERSECTS  
 返回一个布尔表达式，指示第一个参数中指定的 GeoJSON 对象（点、多边形或 LineString）是否与第二个参数中的 GeoJSON（点、多边形或 LineString）相交。  
  
 **语法**  
  
```  
ST_INTERSECTS (<spatial_expr>, <spatial_expr>)  
```  
  
 **参数**  
  
- `spatial_expr`  
  
   是任何有效的 GeoJSON 点、多边形或 LineString 对象表达式。  
 
- `spatial_expr`  
  
   为任何有效的 GeoJSON 点、多边形或 LineString 对象表达式。  
  
  返回类型  
  
  返回一个布尔值。  
  
  **示例**  
  
  以下示例介绍了如何查找与给定多边形相交的所有区域。  
  
```  
SELECT a.id   
FROM Areas a   
WHERE ST_INTERSECTS(a.location, {  
    'type':'Polygon',   
    'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]  
})  
```  
  
 结果集如下。  
  
```  
[{ "id": "IntersectingPolygon" }]  
```  
  
####  <a name="bk_st_isvalid"></a> ST_ISVALID  
 返回一个布尔值，指示指定的 GeoJSON 点、多边形或 LineString 表达式是否有效。  
  
 **语法**  
  
```  
ST_ISVALID(<spatial_expr>)  
```  
  
 **参数**  
  
- `spatial_expr`  
  
   为任何有效的 GeoJSON 点、多边形或 LineString 表达式。  
  
  返回类型  
  
  返回一个布尔表达式。  
  
  **示例**  
  
  以下示例介绍了如何使用 ST_VALID 检查点是否有效。  
  
  例如，由于此点具有一个不在有效值范围 [-90，90] 内的纬度值，因此查询返回 false。  
  
  对于多边形，GeoJSON 规范要求提供的最后一个坐标対应该与第一个坐标对相同，才能创建一个闭合形状。 多边形内的点必须以逆时针顺序指定。 以顺时针顺序指定的多边形表示其中的区域倒转。  
  
```  
SELECT ST_ISVALID({ "type": "Point", "coordinates": [31.9, -132.8] }) AS b 
```  
  
 结果集如下。  
  
```  
[{ "b": false }]  
```  
  
####  <a name="bk_st_isvaliddetailed"></a> ST_ISVALIDDETAILED  
 如果指定的 GeoJSON 点、多边形或 LineString 表达式有效，则返回包含布尔值的 JSON 值；如果无效，则额外加上作为字符串值的原因。  
  
 **语法**  
  
```  
ST_ISVALIDDETAILED(<spatial_expr>)  
```  
  
 **参数**  
  
- `spatial_expr`  
  
   任何有效的 GeoJSON 点或多边形表达式。  
  
  返回类型  
  
  如果指定的 GeoJSON 点或多边形表达式有效，则返回包含布尔值的 JSON 值；如果无效，则额外添加原因作为字符串值。  
  
  **示例**  
  
  以下示例介绍了如何使用 ST_ISVALIDDETAILED 检查有效性（附带详细信息）。  
  
```  
SELECT ST_ISVALIDDETAILED({   
  "type": "Polygon",   
  "coordinates": [[ [ 31.8, -5 ], [ 31.8, -4.7 ], [ 32, -4.7 ], [ 32, -5 ] ]]  
}) AS b  
```  
  
 结果集如下。  
  
```  
[{  
  "b": {   
    "valid": false,   
    "reason": "The Polygon input is not valid because the start and end points of the ring number 1 are not the same. Each ring of a polygon must have the same start and end points."   
  }  
}]  
```  
 
## <a name="next-steps"></a>后续步骤  

- [Cosmos DB 的 SQL 语法和 SQL 查询](how-to-sql-query.md)

- [Cosmos DB 文档](https://docs.microsoft.com/azure/cosmos-db/)  
