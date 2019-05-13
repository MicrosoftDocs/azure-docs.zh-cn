---
title: 映射 Data Flow 功能的 Azure 数据工厂中的表达式函数
description: 了解映射数据流中的表达式函数。
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/15/2019
ms.openlocfilehash: 35a056fbe01633dd93bd71ac2af16cba8a21344a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235734"
---
# <a name="data-transformation-expressions-in-mapping-data-flow"></a>在映射数据流中的数据转换表达式 

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

## <a name="expression-functions"></a>表达式函数

在数据工厂中使用表达式语言映射 Data Flow 功能的配置数据转换。

*********************************
### <code>abs</code>
<code><b>abs(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
将数字对正数取模。
* ``abs(-20) -> 20``
* ``abs(10) -> 10``
*********************************
### <code>acos</code>
<code><b>acos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算反余弦值 * ``acos(1) -> 0.0``
*********************************
### <code>add</code>
<code><b>add(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
添加一对字符串或数字。 将日期添加到日期数。 将类似类型的数组追加到另一个数组。 与相同 + 运算符 * ``add(10, 20) -> 30``
* ``10 + 20 -> 30``
* ``add('ice', 'cream') -> 'icecream'``
* ``'ice' + 'cream' + ' cone' -> 'icecream cone'``
* ``add(toDate('2012-12-12'), 3) -> 2012-12-15 (date value)``
* ``toDate('2012-12-12') + 3 -> 2012-12-15 (date value)``
* ``[10, 20] + [30, 40] => [10, 20, 30, 40]``
*********************************
### <code>addDays</code>
<code><b>addDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to add&gt;</i> : integral) => datetime</b></code><br/><br/>
将日期添加到日期或时间戳。 与相同 + 日期的运算符 * ``addDays(toDate('2016-08-08'), 1) -> 2016-08-09``
*********************************
### <code>addMonths</code>
<code><b>addMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to add&gt;</i> : integral) => datetime</b></code><br/><br/>
将几个月添加到日期或时间戳 * ``addMonths(toDate('2016-08-31'), 1) -> 2016-09-30``
* ``addMonths(toTimestamp('2016-09-30 10:10:10'), -1) -> 2016-08-31 10:10:10``
*********************************
### <code>and</code>
<code><b>and(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
逻辑 AND 运算符。 与相同 （& a) （& a) * ``and(true, false) -> false``
* ``true && false -> false``
*********************************
### <code>asin</code>
<code><b>asin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算反正弦值 * ``asin(0) -> 0.0``
*********************************
### <code>atan</code>
<code><b>atan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算反正切值 * ``atan(0) -> 0.0``
*********************************
### <code>atan2</code>
<code><b>atan2(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
返回表示的角度以弧度表示平面的正 x 轴与给定坐标的点之间 * ``atan2(0, 0) -> 0.0``
*********************************
### <code>avg</code>
<code><b>avg(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
获取列的值的平均值 * ``avg(sales) -> 7523420.234``
*********************************
### <code>avgIf</code>
<code><b>avgIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
基于条件获取列的值的平均值 * ``avgIf(region == 'West', sales) -> 7523420.234``
*********************************
### <code>byName</code>
<code><b>byName(<i>&lt;column name&gt;</i> : string) => any</b></code><br/><br/>
在流中按名称选择列的值。 如果有多个匹配项，则返回第一个匹配项。 如果没有匹配项则返回 NULL 值。 返回的值必须是其中一个类型转换函数 （TO_DATE，TO_STRING...） 转换的类型。在设计时已知的列名称应只需通过其名称进行寻址。 不支持计算的输入，但可以使用的参数替换 * ``toString(byName('parent')) -> appa``
* ``toLong(byName('income')) -> 9000000000009``
* ``toBoolean(byName('foster')) -> false``
* ``toLong(byName($debtCol)) -> 123456890``
* ``birthDate -> 12/31/2050``
* ``toString(byName('Bogus Column')) -> NULL``
*********************************
### <code>byPosition</code>
<code><b>byPosition(<i>&lt;position&gt;</i> : integer) => any</b></code><br/><br/>
在流中选择某列的值按其相对位置 (从 1 开始)。 如果超出界限位于它返回一个 NULL 值。 返回的值必须是其中一个类型转换函数 （TO_DATE，TO_STRING...） 转换的类型不支持计算的输入，但可以使用的参数替换 * ``toString(byPosition(1)) -> amma``
* ``toDecimal(byPosition(2), 10, 2) -> 199990.99``
* ``toBoolean(byName(4)) -> false``
* ``toString(byName($colName)) -> family``
* ``toString(byPosition(1234)) -> NULL``
*********************************
### <code>case</code>
<code><b>case(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, <i>&lt;false_expression&gt;</i> : any, ...) => any</b></code><br/><br/>
根据备用条件应用一个值或另一个值。 如果输入个数为偶数，另对于最后一个条件为 NULL * ``case(custType == 'Premium', 10, 4.5)``
* ``case(custType == 'Premium', price*0.95, custType == 'Elite',   price*0.9, price*2)``
* ``case(dayOfWeek(saleDate) == 1, 'Sunday', dayOfWeek(saleDate) == 6, 'Saturday')``
*********************************
### <code>cbrt</code>
<code><b>cbrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算一个数字的立方根 * ``cbrt(8) -> 2.0``
*********************************
### <code>ceil</code>
<code><b>ceil(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
返回不小于数的最小整数 * ``ceil(-0.1) -> 0``
*********************************
### <code>concat</code>
<code><b>concat(<i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
将字符串的可变数字连接在一起。 与相同 + 运算符与字符串 * ``concat('Awesome', 'Cool', 'Product') -> 'AwesomeCoolProduct'``
* ``'Awesome' + 'Cool' + 'Product' -> 'AwesomeCoolProduct'``
* ``concat(addrLine1, ' ', addrLine2, ' ', city, ' ', state, ' ', zip)``
* ``addrLine1 + ' ' + addrLine2 + ' ' + city + ' ' + state + ' ' + zip``
*********************************
### <code>concatWS</code>
<code><b>concatWS(<i>&lt;separator&gt;</i> : string, <i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
使用分隔符将字符串的可变数字连接在一起。 第一个参数是分隔符 * ``concatWS(' ', 'Awesome', 'Cool', 'Product') -> 'Awesome Cool Product'``
* ``concatWS(' ' , addrLine1, addrLine2, city, state, zip) -> ``
* ``concatWS(',' , toString(order_total), toString(order_discount))``
*********************************
### <code>cos</code>
<code><b>cos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算余弦值 * ``cos(10) -> -0.83907152907``
*********************************
### <code>cosh</code>
<code><b>cosh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算双曲余弦值的值 * ``cosh(0) -> 1.0``
*********************************
### <code>count</code>
<code><b>count([<i>&lt;value1&gt;</i> : any]) => long</b></code><br/><br/>
获取值的聚合计数。 如果指定的可选列，则它会忽略 NULL 值计数 * ``count(custId) -> 100``
* ``count(custId, custName) -> 50``
* ``count() -> 125``
* ``count(iif(isNull(custId), 1, NULL)) -> 5``
*********************************
### <code>countDistinct</code>
<code><b>countDistinct(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : any], ...) => long</b></code><br/><br/>
获取一组列的非重复值的聚合计数 * ``countDistinct(custId, custName) -> 60``
*********************************
### <code>countIf</code>
<code><b>countIf(<i>&lt;value1&gt;</i> : boolean, [<i>&lt;value2&gt;</i> : any]) => long</b></code><br/><br/>
根据条件获取值的聚合计数。 如果指定可选的列，则它会忽略 NULL 值计数 * ``countIf(state == 'CA' && commission < 10000, name) -> 100``
*********************************
### <code>covariancePopulation</code>
<code><b>covariancePopulation(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
获取两个列之间的总体协方差 * ``covariancePopulation(sales, profit) -> 122.12``
*********************************
### <code>covariancePopulationIf</code>
<code><b>covariancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
基于条件上，将获取两个列的总体协方差 * ``covariancePopulationIf(region == 'West', sales) -> 122.12``
*********************************
### <code>covarianceSample</code>
<code><b>covarianceSample(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
获取两个列的样本协方差 * ``covarianceSample(sales, profit) -> 122.12``
*********************************
### <code>covarianceSampleIf</code>
<code><b>covarianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
基于条件上，将获取两个列的样本协方差 * ``covarianceSampleIf(region == 'West', sales, profit) -> 122.12``
*********************************
### <code>crc32</code>
<code><b>crc32(<i>&lt;value1&gt;</i> : any, ...) => long</b></code><br/><br/>
根据位长度计算不同基元数据类型的列集的 CRC32 哈希，值只能为 0(256)、224、256、384、512。 它可用于计算行的指纹 * ``crc32(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 3630253689``
*********************************
### <code>cumeDist</code>
<code><b>cumeDist() => integer</b></code><br/><br/>
CumeDist 函数计算某个值相对于分区中所有值的位置。 结果是前面的行数，或者等于当前行在分区中的顺序除以窗口分区中的总行数。 在排序中的任何关联值的计算结果到相同的位置。
* ``cumeDist() -> 1``
*********************************
### <code>currentDate</code>
<code><b>currentDate([<i>&lt;value1&gt;</i> : string]) => date</b></code><br/><br/>
当此作业开始运行时获取当前日期。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 当地时区用作默认值。
* ``currentDate() -> 12-12-2030``
* ``currentDate('PST') -> 12-31-2050``
*********************************
### <code>currentTimestamp</code>
<code><b>currentTimestamp() => timestamp</b></code><br/><br/>
获取当前时间戳，启动作业以运行与本地时区 * ``currentTimestamp() -> 12-12-2030T12:12:12``
*********************************
### <code>currentUTC</code>
<code><b>currentUTC([<i>&lt;value1&gt;</i> : string]) => timestamp</b></code><br/><br/>
获取 UTC 格式的当前时间戳。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 默认值为当前时区 * ``currentUTC() -> 12-12-2030T19:18:12``
* ``currentUTC('Asia/Seoul') -> 12-13-2030T11:18:12``
*********************************
### <code>dayOfMonth</code>
<code><b>dayOfMonth(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取给定一个日期的月份的日期 * ``dayOfMonth(toDate('2018-06-08')) -> 08``
*********************************
### <code>dayOfWeek</code>
<code><b>dayOfWeek(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取给定日期的星期日期。 1-星期日，2-...星期一，7-星期六 * ``dayOfWeek(toDate('2018-06-08')) -> 7``
*********************************
### <code>dayOfYear</code>
<code><b>dayOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取给定一个日期的年份的日期 * ``dayOfYear(toDate('2016-04-09')) -> 100``
*********************************
### <code>degrees</code>
<code><b>degrees(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
将弧度转换为度 * ``degrees(3.141592653589793) -> 180``
*********************************
### <code>denseRank</code>
<code><b>denseRank(<i>&lt;value1&gt;</i> : any, ...) => integer</b></code><br/><br/>
计算某个值在值组中的排名。 结果是 1 加上前面的行数，或者等于当前行在分区中的顺序。 值不会在序列中生成空隙。 即使数据未排序和查找的更改中的值有效密集排名 * ``denseRank(salesQtr, salesAmt) -> 1``
*********************************
### <code>divide</code>
<code><b>divide(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
将数字对相除。 与相同 / 运算符 * ``divide(20, 10) -> 2``
* ``20 / 10 -> 2``
*********************************
### <code>endsWith</code>
<code><b>endsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
检查字符串是否结尾所提供的字符串 * ``endsWith('great', 'eat') -> true``
*********************************
### <code>equals</code>
<code><b>equals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“等于”比较运算符。 与 = = 运算符相同 * ``equals(12, 24) -> false``
* ``12==24 -> false``
* ``'bad'=='bad' -> true``
* ``'good'== NULL -> false``
* ``NULL===NULL -> false``
*********************************
### <code>equalsIgnoreCase</code>
<code><b>equalsIgnoreCase(<i>&lt;value1&gt;</i> : string, <i>&lt;value2&gt;</i> : string) => boolean</b></code><br/><br/>
忽略大小写的“等于”比较运算符。 相同 <> = 运算符 * ``'abc'<==>'abc' -> true``
* ``equalsIgnoreCase('abc', 'Abc') -> true``
*********************************
### <code>factorial</code>
<code><b>factorial(<i>&lt;value1&gt;</i> : number) => long</b></code><br/><br/>
计算一个数字的阶乘 * ``factorial(5) -> 120``
*********************************
### <code>false</code>
<code><b>false() => boolean</b></code><br/><br/>
始终返回 false 值。 如果没有名为 'false' 的列，请使用函数 syntax(false()) * ``isDiscounted == false()``
* ``isDiscounted() == false``
*********************************
### <code>first</code>
<code><b>first(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
获取列组的第一个值。 如果省略第二个参数 ignoreNulls，它被假定 false * ``first(sales) -> 12233.23``
* ``first(sales, false) -> NULL``
*********************************
### <code>floor</code>
<code><b>floor(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
返回的最大整数不大于数 * ``floor(-0.1) -> -1``
*********************************
### <code>fromUTC</code>
<code><b>fromUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
将从 UTC 转换为时间戳。 可以选择性地传递 'GMT'、 '太平洋标准时间、 'UTC'、 ' America/开曼窗体中的时区。 默认值为当前时区 * ``fromUTC(currentTimeStamp()) -> 12-12-2030T19:18:12``
* ``fromUTC(currentTimeStamp(), 'Asia/Seoul') -> 12-13-2030T11:18:12``
*********************************
### <code>greater</code>
<code><b>greater(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“大于”比较运算符。 与相同 > 运算符 * ``greater(12, 24) -> false``
* ``'abcd' > 'abc' -> true``
*********************************
### <code>greaterOrEqual</code>
<code><b>greaterOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“大于等于”比较运算符。 与相同 > = 运算符 * ``greaterOrEqual(12, 12) -> false``
* ``'abcd' >= 'abc' -> true``
*********************************
### <code>greatest</code>
<code><b>greatest(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
返回输入值列表中的最大值。 如果所有输入都均为 null，则返回 null * ``greatest(10, 30, 15, 20) -> 30``
* ``greatest(toDate('12/12/2010'), toDate('12/12/2011'), toDate('12/12/2000')) -> '12/12/2011'``
*********************************
### <code>hour</code>
<code><b>hour(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
获取时间戳的小时值。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 当地时区用作默认值。
* ``hour(toTimestamp('2009-07-30T12:58:59')) -> 12``
* ``hour(toTimestamp('2009-07-30T12:58:59'), 'PST') -> 12``
*********************************
### <code>iif</code>
<code><b>iif(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, [<i>&lt;false_expression&gt;</i> : any]) => any</b></code><br/><br/>
根据条件应用一个值或另一个值。 如果未指定另一个是它被视为 NULL。 必须是兼容的这两个值 （数字、 字符串...） * ``iif(custType == 'Premium', 10, 4.5)``
* ``iif(amount > 100, 'High')``
* ``iif(dayOfWeek(saleDate) == 6, 'Weekend', 'Weekday')``
*********************************
### <code>in</code>
<code><b>in(<i>&lt;array of items&gt;</i> : array, <i>&lt;item to find&gt;</i> : any) => boolean</b></code><br/><br/>
检查某个项是否在数组中 * ``in([10, 20, 30], 10) -> true``
* ``in(['good', 'kid'], 'bad') -> false``
*********************************
### <code>initCap</code>
<code><b>initCap(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
将每个单词的第一个字母转换为大写。 单词标识为由空格分隔 * ``initCap('cool iceCREAM') -> 'Cool IceCREAM'``
*********************************
### <code>instr</code>
<code><b>instr(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string) => integer</b></code><br/><br/>
在字符串中查找子字符串的位置（从 1 开始）。 0 是返回如果未找到 * ``instr('great', 'eat') -> 3``
* ``instr('microsoft', 'o') -> 7``
* ``instr('good', 'bad') -> 0``
*********************************
### <code>isDelete</code>
<code><b>isDelete([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查该行是否标记为删除。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isDelete() -> true``
* ``isDelete(1) -> false``
*********************************
### <code>isError</code>
<code><b>isError([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查该行是否标记为错误。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isError() -> true``
* ``isError(1) -> false``
*********************************
### <code>isIgnore</code>
<code><b>isIgnore([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查该行是否标记为忽略。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isIgnore() -> true``
* ``isIgnore(1) -> false``
*********************************
### <code>isInsert</code>
<code><b>isInsert([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查该行是否标记为插入。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isInsert() -> true``
* ``isInsert(1) -> false``
*********************************
### <code>isMatch</code>
<code><b>isMatch([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查查找时是否匹配该行。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isMatch() -> true``
* ``isMatch(1) -> false``
*********************************
### <code>isNull</code>
<code><b>isNull(<i>&lt;value1&gt;</i> : any) => boolean</b></code><br/><br/>
检查值是否为 NULL * ``isNull(NULL()) -> true``
* ``isNull('') -> false'``
*********************************
### <code>isUpdate</code>
<code><b>isUpdate([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
检查该行是否标记为更新。 采用多个输入的流转换为可以传递将索引 （从 1 开始） 的流。 流索引的默认值为 1 * ``isUpdate() -> true``
* ``isUpdate(1) -> false``
*********************************
### <code>kurtosis</code>
<code><b>kurtosis(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的峰度 * ``kurtosis(sales) -> 122.12``
*********************************
### <code>kurtosisIf</code>
<code><b>kurtosisIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的峰度 * ``kurtosisIf(region == 'West', sales) -> 122.12``
*********************************
### <code>lag</code>
<code><b>lag(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look before&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
获取当前行之前由第一个参数计算的 n 行的值。 第二个参数是要回顾的行数，默认值为 1。 如果不返回的值为 null，除非指定了默认值的行数 * ``lag(amount, 2) -> 60``
* ``lag(amount, 2000, 100) -> 100``
*********************************
### <code>last</code>
<code><b>last(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
获取列组的最后一个值。 如果省略第二个参数 ignoreNulls，它被假定 false * ``last(sales) -> 523.12``
* ``last(sales, false) -> NULL``
*********************************
### <code>lastDayOfMonth</code>
<code><b>lastDayOfMonth(<i>&lt;value1&gt;</i> : datetime) => date</b></code><br/><br/>
获取给定一个日期的月份的最后一个日期 * ``lastDayOfMonth(toDate('2009-01-12')) -> 2009-01-31``
*********************************
### <code>lead</code>
<code><b>lead(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look after&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
获取当前行之后由第一个参数计算的 n 行的值。 第二个参数是要期待的行数，默认值为 1。 如果不返回的值为 null，除非指定了默认值的行数 * ``lead(amount, 2) -> 60``
* ``lead(amount, 2000, 100) -> 100``
*********************************
### <code>least</code>
<code><b>least(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
“小于等于”比较运算符。 与相同 < = 运算符 * ``least(10, 30, 15, 20) -> 10``
* ``least(toDate('12/12/2010'), toDate('12/12/2011'), toDate('12/12/2000')) -> '12/12/2000'``
*********************************
### <code>left</code>
<code><b>left(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
从索引 1 处开始提取包含字符数的子字符串。 相同的子字符串 (str，1，n) * ``left('bojjus', 2) -> 'bo'``
* ``left('bojjus', 20) -> 'bojjus'``
*********************************
### <code>length</code>
<code><b>length(<i>&lt;value1&gt;</i> : string) => integer</b></code><br/><br/>
返回字符串的长度 * ``length('kiddo') -> 5``
*********************************
### <code>lesser</code>
<code><b>lesser(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“小于”比较运算符。 与相同 < 运算符 * ``lesser(12 < 24) -> true``
* ``'abcd' < 'abc' -> false``
*********************************
### <code>lesserOrEqual</code>
<code><b>lesserOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“小于等于”比较运算符。 与相同 < = 运算符 * ``lesserOrEqual(12, 12) -> true``
* ``'abcd' <= 'abc' -> false``
*********************************
### <code>levenshtein</code>
<code><b>levenshtein(<i>&lt;from string&gt;</i> : string, <i>&lt;to string&gt;</i> : string) => integer</b></code><br/><br/>
获取两个字符串之间的 levenshtein 距离 * ``levenshtein('boys', 'girls') -> 4``
*********************************
### <code>like</code>
<code><b>like(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
模式是将逐字匹配的字符串。 例外情况是以下特殊符号: _ 匹配 （类似于输入中任何一个字符。 posix 正则表达式中的 .）。% 匹配输入中的零个或多个字符（类似于 posix 正则表达式中的 .*）。
转义字符为 ''。 如果转义字符的之前带有特殊符号或其他转义字符，则在字面上匹配后面的字符。 转义其他任何字符的操作无效。
* ``like('icecream', 'ice%') -> true``
*********************************
### <code>locate</code>
<code><b>locate(<i>&lt;substring to find&gt;</i> : string, <i>&lt;string&gt;</i> : string, [<i>&lt;from index - 1-based&gt;</i> : integral]) => integer</b></code><br/><br/>
从特定的位置开始，在字符串中查找子字符串的位置（从 1 开始）。 如果省略位置，则视为从字符串开头查找。 0 是返回如果未找到 * ``locate('eat', 'great') -> 3``
* ``locate('o', 'microsoft', 6) -> 7``
* ``locate('bad', 'good') -> 0``
*********************************
### <code>log</code>
<code><b>log(<i>&lt;value1&gt;</i> : number, [<i>&lt;value2&gt;</i> : number]) => double</b></code><br/><br/>
计算对数值。 可选的基可以是其他提供欧拉数，如果使用 * ``log(100, 10) -> 2``
*********************************
### <code>log10</code>
<code><b>log10(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算基于 10 基日志值 * ``log10(100) -> 2``
*********************************
### <code>lower</code>
<code><b>lower(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Lowercases 字符串 * ``lower('GunChus') -> 'gunchus'``
*********************************
### <code>lpad</code>
<code><b>lpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
在左侧使用提供的填充内容填充字符串，直到达到特定的长度。 如果该字符串为等于或大于的长度，则它将被不视为执行任何操作* ``lpad('great', 10, '-') -> '-----great'`` 
* ``lpad('great', 4, '-') -> 'great'`` 
* ' lpad (卓越 8 <>)-> <><great'``
*********************************
### <code>ltrim</code>
<code><b>ltrim(<i>&lt;string to trim&gt;</i> : string, <i>&lt;trim characters&gt;</i> : string) => string</b></code><br/><br/>
在左侧裁剪掉前导和尾随字符的字符串。 如果未指定第二个参数，则裁剪掉空格。 否则剪裁第二个参数中指定的任何字符 * ``ltrim('!--!wor!ld!', '-!') -> 'wor!ld!'``
*********************************
### <code>max</code>
<code><b>max(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
获取列的最大值 * ``MAX(sales) -> 12312131.12``
*********************************
### <code>maxIf</code>
<code><b>maxIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
根据条件，则获取列的最大值 * ``maxIf(region == 'West', sales) -> 99999.56``
*********************************
### <code>md5</code>
<code><b>md5(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
计算不同基元数据类型的列集的 MD5 摘要，并返回 32 字符十六进制字符串。 它可用于计算行的指纹 * ``md5(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 'c1527622a922c83665e49835e46350fe'``
*********************************
### <code>mean</code>
<code><b>mean(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
获取列值的中间值。 AVG 相同 * ``mean(sales) -> 7523420.234``
*********************************
### <code>meanIf</code>
<code><b>meanIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根据条件获取列值的中间值。 AvgIf 相同 * ``meanIf(region == 'West', sales) -> 7523420.234``
*********************************
### <code>min</code>
<code><b>min(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
获取列的最小值 * ``min(sales) -> 00.01``
* ``min(orderDate) -> 12/12/2000``
*********************************
### <code>minIf</code>
<code><b>minIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
根据条件，则获取列的最小值 * ``minIf(region == 'West', sales) -> 00.01``
*********************************
### <code>minus</code>
<code><b>minus(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
减去数字。 从日期数中减去日期。 相同-运算符 * ``minus(20, 10) -> 10``
* ``20 - 10 -> 10``
* ``minus(toDate('2012-12-15'), 3) -> 2012-12-12 (date value)``
* ``toDate('2012-12-15') - 3 -> 2012-12-13 (date value)``
*********************************
### <code>minute</code>
<code><b>minute(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
获取时间戳的分钟值。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 当地时区用作默认值。
* ``minute(toTimestamp('2009-07-30T12:58:59')) -> 58``
* ``minute(toTimestamp('2009-07-30T12:58:59', 'PST')) -> 58``
*********************************
### <code>mod</code>
<code><b>mod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
将数字对取模。 %运算符相同 * ``mod(20, 8) -> 4``
* ``20 % 8 -> 4``
*********************************
### <code>month</code>
<code><b>month(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取一个日期或时间戳的月份值 * ``month(toDate('2012-8-8')) -> 8``
*********************************
### <code>monthsBetween</code>
<code><b>monthsBetween(<i>&lt;from date/timestamp&gt;</i> : datetime, <i>&lt;to date/timestamp&gt;</i> : datetime, [<i>&lt;time zone&gt;</i> : boolean], [<i>&lt;value4&gt;</i> : string]) => double</b></code><br/><br/>
获取两个 datesYou 之间的月数可以传递可选时区中的格林威治标准时间、 太平洋标准时间、 'UTC'、 ' America/开曼窗体。 当地时区用作默认值。
* ``monthsBetween(toDate('1997-02-28 10:30:00'), toDate('1996-10-30')) -> 3.94959677``
*********************************
### <code>multiply</code>
<code><b>multiply(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
将数字对相乘。 与相同 * 运算符 * ``multiply(20, 10) -> 200``
* ``20 * 10 -> 200``
*********************************
### <code>nTile</code>
<code><b>nTile([<i>&lt;value1&gt;</i> : integer]) => integer</b></code><br/><br/>
NTile 函数将每个窗口分区的行分割为从 1 到 `n` 的 `n` 桶。 桶值最大相差 1。 如果分区中的行数不能均匀分割成桶数，则余值将逐个分布在每个桶中，从第一个桶开始。 NTile 函数可用于计算 tertiles、 四分位数、 十分位数和其他常见的摘要统计信息。 在初始化期间，该函数将计算两个变量：正则存储桶的大小将具有一个额外的行添加到它。 这两个变量基于当前分区的大小。 在计算过程中，该函数将跟踪当前行号、当前桶号，以及发生桶更改的行号 (bucketThreshold)。 如果当前行号达到桶的阈值，则桶值将会加 1，阈值将按桶大小增加（如果当前桶已填充，则额外加 1）。
* ``nTile() -> 1``
* ``nTile(numOfBuckets) -> 1``
*********************************
### <code>negate</code>
<code><b>negate(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
对数字求反。 将正数和负数为负，反之亦然 * ``negate(13) -> -13``
*********************************
### <code>nextSequence</code>
<code><b>nextSequence() => long</b></code><br/><br/>
返回下一个唯一序列。 数是仅在一个分区内连续和 partitionId 前缀 * ``nextSequence() -> 12313112``
*********************************
### <code>normalize</code>
<code><b>normalize(<i>&lt;String to normalize&gt;</i> : string) => string</b></code><br/><br/>
规范化为单独的带重音符的 unicode 字符的字符串值 * ``normalize('boys') -> 'boys'``
*********************************
### <code>not</code>
<code><b>not(<i>&lt;value1&gt;</i> : boolean) => boolean</b></code><br/><br/>
逻辑求反运算符 * ``not(true) -> false``
* ``not(premium)``
*********************************
### <code>notEquals</code>
<code><b>notEquals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
“不等于”比较运算符。 与相同 ！ = 运算符 * ``12!=24 -> true``
* ``'abc'!='abc' -> false``
*********************************
### <code>null</code>
<code><b>null() => null</b></code><br/><br/>
返回 NULL 值。 如果存在名为“null”的列，则使用函数 syntax(null())。 使用任何操作将导致为空 * ``custId = NULL (for derived field)``
* ``custId == NULL -> NULL``
* ``'nothing' + NULL -> NULL``
* ``10 * NULL -> NULL'``
* ``NULL == '' -> NULL'``
*********************************
### <code>or</code>
<code><b>or(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
逻辑 OR 运算符。 与相同 | | * ``or(true, false) -> true``
* ``true || false -> true``
*********************************
### <code>pMod</code>
<code><b>pMod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
将数字对正数取模。
* ``pmod(-20, 8) -> 4``
*********************************
### <code>power</code>
<code><b>power(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
一个对的另一数字 * ``power(10, 2) -> 100``
*********************************
### <code>rank</code>
<code><b>rank(<i>&lt;value1&gt;</i> : any, ...) => integer</b></code><br/><br/>
计算某个值在值组中的排名。 结果是 1 加上前面的行数，或者等于当前行在分区中的顺序。 值将在序列中生成空隙。 级别工作甚至时不对数据进行排序和查找的更改中的值 * ``rank(salesQtr, salesAmt) -> 1``
*********************************
### <code>regexExtract</code>
<code><b>regexExtract(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, [<i>&lt;match group 1-based index&gt;</i> : integral]) => string</b></code><br/><br/>
提取给定正则表达式模式的匹配子字符串。 最后一个参数标识匹配组，如果省略，则默认为 1。 使用<regex>'(back quote) 来匹配而不转义的字符串 * ``regexExtract('Cost is between 600 and 800 dollars', '(\\d+) and (\\d+)', 2) -> '800'``
* ``regexExtract('Cost is between 600 and 800 dollars', `(\d+) and (\d+)`, 2) -> '800'``
*********************************
### <code>regexMatch</code>
<code><b>regexMatch(<i>&lt;string&gt;</i> : string, <i>&lt;regex to match&gt;</i> : string) => boolean</b></code><br/><br/>
检查字符串是否与给定的正则表达式模式相匹配。 使用<regex>'(back quote) 来匹配而不转义的字符串 * ``regexMatch('200.50', '(\\d+).(\\d+)') -> true``
* ``regexMatch('200.50', `(\d+).(\d+)`) -> true``
*********************************
### <code>regexReplace</code>
<code><b>regexReplace(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, <i>&lt;substring to replace&gt;</i> : string) => string</b></code><br/><br/>
使用给定的字符串中的另一个子字符串替换所有匹配的正则表达式模式 '<regex>'(back quote) 来匹配而不转义的字符串 * ``regexReplace('100 and 200', '(\\d+)', 'bojjus') -> 'bojjus and bojjus'``
* ``regexReplace('100 and 200', `(\d+)`, 'gunchus') -> 'gunchus and gunchus'``
*********************************
### <code>regexSplit</code>
<code><b>regexSplit(<i>&lt;string to split&gt;</i> : string, <i>&lt;regex expression&gt;</i> : string) => array</b></code><br/><br/>
拆分基于根据正则表达式分隔符的字符串并返回一个字符串数组 * ``regexSplit('oneAtwoBthreeC', '[CAB]') -> ['one', 'two', 'three']``
* ``regexSplit('oneAtwoBthreeC', '[CAB]')[1] -> 'one'``
* ``regexSplit('oneAtwoBthreeC', '[CAB]')[0] -> NULL``
* ``regexSplit('oneAtwoBthreeC', '[CAB]')[20] -> NULL``
*********************************
### <code>replace</code>
<code><b>replace(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string, <i>&lt;substring to replace&gt;</i> : string) => string</b></code><br/><br/>
用给定字符串中的另一个子字符串替换所有出现的子字符串 * ``replace('doggie dog', 'dog', 'cat') -> 'catgie cat'``
* ``replace('doggie dog', 'dog', '') -> 'gie'``
*********************************
### <code>reverse</code>
<code><b>reverse(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
反转字符串 * ``reverse('gunchus') -> 'suhcnug'``
*********************************
### <code>right</code>
<code><b>right(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
从右侧提取包含字符数的子字符串。 相同的子字符串 (str，LENGTH(str)-n，n) * ``right('bojjus', 2) -> 'us'``
* ``right('bojjus', 20) -> 'bojjus'``
*********************************
### <code>rlike</code>
<code><b>rlike(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
检查字符串是否与给定正则表达式模式匹配 * ``rlike('200.50', '(\d+).(\d+)') -> true``
*********************************
### <code>round</code>
<code><b>round(<i>&lt;number&gt;</i> : number, [<i>&lt;scale to round&gt;</i> : number], [<i>&lt;rounding option&gt;</i> : integral]) => double</b></code><br/><br/>
舍入给定可选的小数位数和可选的舍入模式。 如果省略小数位数，则它是默认设置为 0。  如果省略模式，则会将其默认值为 ROUND_HALF_UP(5)。 舍入的值包括 1-ROUND_UP 2-ROUND_DOWN 3-ROUND_CEILING 4-ROUND_FLOOR 5-ROUND_HALF_UP 6-ROUND_HALF_DOWN 7-ROUND_HALF_EVEN 8-ROUND_UNNECESSARY * ``round(100.123) -> 100.0``
* ``round(2.5, 0) -> 3.0``
* ``round(5.3999999999999995, 2, 7) -> 5.40``
*********************************
### <code>rowNumber</code>
<code><b>rowNumber() => integer</b></code><br/><br/>
将分配编号从 1 开始的窗口中的行的顺序行 * ``rowNumber() -> 1``
*********************************
### <code>rpad</code>
<code><b>rpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
在右侧使用提供的填充内容填充字符串，直到达到特定的长度。 如果该字符串为等于或大于的长度，则认为是一个 no-op * ``rpad('great', 10, '-') -> 'great-----'`` 
* ``rpad('great', 4, '-') -> 'great'`` 
* ``rpad('great', 8, '<>') -> 'great<><'`` 
*********************************
### <code>rtrim</code>rtrim</code>
<code><b>rtrim(<i>&lt;string to trim&gt;</i> : string, <i>&lt;trim characters&gt;</i> : string) => string</b></code><br/><br/>
在右侧裁剪掉前导和尾随字符的字符串。 如果未指定第二个参数，则裁剪掉空格。 否则剪裁第二个参数中指定的任何字符 * ``rtrim('!--!wor!ld!', '-!') -> '!--!wor!ld'``
*********************************
### <code>second</code>
<code><b>second(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
获取日期的秒值。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 当地时区用作默认值。
* ``second(toTimestamp('2009-07-30T12:58:59')) -> 59``
*********************************
### <code>sha1</code>
<code><b>sha1(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
计算不同基元数据类型的列集的 SHA-1 摘要，并返回 40 字符十六进制字符串。 它可用于计算行的指纹 * ``sha1(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> '63849fd2abb65fbc626c60b1f827bd05573f0cea'``
*********************************
### <code>sha2</code>
<code><b>sha2(<i>&lt;value1&gt;</i> : integer, <i>&lt;value2&gt;</i> : any, ...) => string</b></code><br/><br/>
计算的基元数据类型给出的位长度的值 0(256)，可以仅是不同的列集的 sha-2 摘要 224、 256、 384、 512。 它可用于计算行的指纹 * ``sha2(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 'd3b2bff62c3a00e9370b1ac85e428e661a7df73959fa1a96ae136599e9ee20fd'``
*********************************
### <code>sin</code>
<code><b>sin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算正弦值 * ``sin(2) -> 0.90929742682``
*********************************
### <code>sinh</code>
<code><b>sinh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算双曲正弦值 * ``sinh(0) -> 0.0``
*********************************
### <code>skewness</code>
<code><b>skewness(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的不对称度 * ``skewness(sales) -> 122.12``
*********************************
### <code>skewnessIf</code>
<code><b>skewnessIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的不对称度 * ``skewnessIf(region == 'West', sales) -> 122.12``
*********************************
### <code>slice</code>
<code><b>slice(<i>&lt;array to slice&gt;</i> : array, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of items&gt;</i> : integral]) => array</b></code><br/><br/>
从位置提取数组的子集。 位置从 1 开始。 如果省略长度，则默认为字符串的末尾 * ``slice([10, 20, 30, 40], 1, 2) -> [10, 20]``
* ``slice([10, 20, 30, 40], 2) -> [20, 30, 40]``
* ``slice([10, 20, 30, 40], 2)[1] -> 20``
* ``slice([10, 20, 30, 40], 2)[0] -> NULL``
* ``slice([10, 20, 30, 40], 2)[20] -> NULL``
* ``slice([10, 20, 30, 40], 8) -> []``
*********************************
### <code>soundex</code>
<code><b>soundex(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
获取字符串的 soundex 代码 * ``soundex('genius') -> 'G520'``
*********************************
### <code>split</code>
<code><b>split(<i>&lt;string to split&gt;</i> : string, <i>&lt;split characters&gt;</i> : string) => array</b></code><br/><br/>
拆分基于分隔符的字符串并返回一个字符串数组 * ``split('100,200,300', ',') -> ['100', '200', '300']``
* ``split('100,200,300', '|') -> ['100,200,300']``
* ``split('100, 200, 300', ', ') -> ['100', '200', '300']``
* ``split('100, 200, 300', ', ')[1] -> '100'``
* ``split('100, 200, 300', ', ')[0] -> NULL``
* ``split('100, 200, 300', ', ')[20] -> NULL``
* ``split('100200300', ',') -> ['100200300']``
*********************************
### <code>sqrt</code>
<code><b>sqrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算一个数字的平方根 * ``sqrt(9) -> 3``
*********************************
### <code>startsWith</code>
<code><b>startsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
检查字符串是否以与所提供的字符串 * ``startsWith('great', 'gr') -> true``
*********************************
### <code>stddev</code>
<code><b>stddev(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的标准偏差 * ``stdDev(sales) -> 122.12``
*********************************
### <code>stddevIf</code>
<code><b>stddevIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的标准偏差 * ``stddevIf(region == 'West', sales) -> 122.12``
*********************************
### <code>stddevPopulation</code>
<code><b>stddevPopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的总体标准偏差 * ``stddevPopulation(sales) -> 122.12``
*********************************
### <code>stddevPopulationIf</code>
<code><b>stddevPopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的总体标准偏差 * ``stddevPopulationIf(region == 'West', sales) -> 122.12``
*********************************
### <code>stddevSample</code>
<code><b>stddevSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的样本标准偏差 * ``stddevSample(sales) -> 122.12``
*********************************
### <code>stddevSampleIf</code>
<code><b>stddevSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的样本标准偏差 * ``stddevSampleIf(region == 'West', sales) -> 122.12``
*********************************
### <code>subDays</code>
<code><b>subDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
从日期中减去月份。 相同日期-运算符 * ``subDays(toDate('2016-08-08'), 1) -> 2016-08-09``
*********************************
### <code>subMonths</code>
<code><b>subMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
从日期或时间戳的几个月中减去 * ``subMonths(toDate('2016-09-30'), 1) -> 2016-08-31``
*********************************
### <code>substring</code>
<code><b>substring(<i>&lt;string to subset&gt;</i> : string, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of characters&gt;</i> : integral]) => string</b></code><br/><br/>
从某个位置提取特定长度的子字符串。 位置从 1 开始。 如果省略长度，则默认为字符串的末尾 * ``substring('Cat in the hat', 5, 2) -> 'in'``
* ``substring('Cat in the hat', 5, 100) -> 'in the hat'``
* ``substring('Cat in the hat', 5) -> 'in the hat'``
* ``substring('Cat in the hat', 100, 100) -> ''``
*********************************
### <code>sum</code>
<code><b>sum(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
获取数值列的聚合总数 * ``sum(col) -> value``
*********************************
### <code>sumDistinct</code>
<code><b>sumDistinct(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
获取数值列的非重复值的聚合总数 * ``sumDistinct(col) -> value``
*********************************
### <code>sumDistinctIf</code>
<code><b>sumDistinctIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根据条件获取数字列的聚合总数。 条件可以基于任何列 * ``sumDistinctIf(state == 'CA' && commission < 10000, sales) -> value``
* ``sumDistinctIf(true, sales) -> SUM(sales) ``
*********************************
### <code>sumIf</code>
<code><b>sumIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根据条件获取数字列的聚合总数。 条件可以基于任何列 * ``sumIf(state == 'CA' && commission < 10000, sales) -> value``
* ``sumIf(true, sales) -> SUM(sales) ``
*********************************
### <code>tan</code>
<code><b>tan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算正切值 * ``tan(0) -> 0.0``
*********************************
### <code>tanh</code>
<code><b>tanh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
计算双曲正切值 * ``tanh(0) -> 0.0``
*********************************
### <code>toBoolean</code>
<code><b>toBoolean(<i>&lt;value1&gt;</i> : string) => boolean</b></code><br/><br/>
可将值转换 (不，true、 y、 是，"1") 为 true 和 (f、 'false'，' n、 否，"0") 为 false 和 NULL 的任何其他值 * ``toBoolean('true') -> true``
* ``toBoolean('n') -> false``
* ``toBoolean('truthy') -> NULL``
*********************************
### <code>toDate</code>
<code><b>toDate(<i>&lt;string&gt;</i> : any, [<i>&lt;date format&gt;</i> : string]) => date</b></code><br/><br/>
将字符串转换为给定的可选日期格式的日期。 所有可能的格式，请参阅 Java SimpleDateFormat。 如果省略日期格式，则接受以下组合。 [ yyyy, yyyy-[M]M, yyyy-[M]M-[d]d, yyyy-[M]M-[d]d, yyyy-[M]M-[d]d, yyyy-[M]M-[d]dT* ] * ``toDate('2012-8-8') -> 2012-8-8``
* ``toDate('12/12/2012', 'MM/dd/yyyy') -> 2012-12-12``
*********************************
### <code>toDecimal</code>
<code><b>toDecimal(<i>&lt;value&gt;</i> : any, [<i>&lt;precision&gt;</i> : integral], [<i>&lt;scale&gt;</i> : integral], [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => decimal(10,0)</b></code><br/><br/>
将任何数字或字符串转换为小数值。 如果未指定精度和小数位数，则默认为 (10,2)。可以使用可选的 Java 十进制格式进行转换。 例如，EN-US、 de、 zh CN BCP47 语言的形式中以可选的区域设置格式 * ``toDecimal(123.45) -> 123.45``
* ``toDecimal('123.45', 8, 4) -> 123.4500``
* ``toDecimal('$123.45', 8, 4,'$###.00') -> 123.4500``
* ``toDecimal('Ç123,45', 10, 2, 'Ç###,##', 'de') -> 123.45``
*********************************
### <code>toDouble</code>
<code><b>toDouble(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => double</b></code><br/><br/>
将任何数字或字符串转换为双精度值。 可以使用可选的 Java 十进制格式进行转换。 例如，EN-US、 de、 zh CN BCP47 语言的形式中以可选的区域设置格式 * ``toDouble(123.45) -> 123.45``
* ``toDouble('123.45') -> 123.45``
* ``toDouble('$123.45', '$###.00') -> 123.45``
* ``toDouble('Ç123,45', 'Ç###,##', 'de') -> 123.45``
*********************************
### <code>toFloat</code>
<code><b>toFloat(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => float</b></code><br/><br/>
将任何数字或字符串转换为浮点值。 可以使用可选的 Java 十进制格式进行转换。 将截断任何双 * ``toFloat(123.45) -> 123.45``
* ``toFloat('123.45') -> 123.45``
* ``toFloat('$123.45', '$###.00') -> 123.45``
*********************************
### <code>toInteger</code>
<code><b>toInteger(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => integer</b></code><br/><br/>
将任何数值或字符串转换为一个整数值。 可以使用可选的 Java 十进制格式进行转换。 将截断任何 long、 float、 double * ``toInteger(123) -> 123``
* ``toInteger('123') -> 123``
* ``toInteger('$123', '$###') -> 123``
*********************************
### <code>toLong</code>
<code><b>toLong(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => long</b></code><br/><br/>
将任何数字或字符串转换为长值。 可以使用可选的 Java 十进制格式进行转换。 将截断任何 float、 double * ``toLong(123) -> 123``
* ``toLong('123') -> 123``
* ``toLong('$123', '$###') -> 123``
*********************************
### <code>toShort</code>
<code><b>toShort(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => short</b></code><br/><br/>
将任何数字或字符串转换为短值。 可以使用可选的 Java 十进制格式进行转换。 将截断任何整数，long、 float、 double * ``toShort(123) -> 123``
* ``toShort('123') -> 123``
* ``toShort('$123', '$###') -> 123``
*********************************
### <code>toString</code>
<code><b>toString(<i>&lt;value&gt;</i> : any, [<i>&lt;number format/date format&gt;</i> : string]) => string</b></code><br/><br/>
将基元数据类型转换为字符串。 对于数字和日期，可以指定格式。 如果未指定，则选择系统默认值。对数字使用 Java 十进制格式。 所有可能的日期格式; 数据格式，请参阅 Java SimpleDateFormat默认格式是年-月-日 * ``toString(10) -> '10'``
* ``toString('engineer') -> 'engineer'``
* ``toString(123456.789, '##,###.##') -> '123,456.79'``
* ``toString(123.78, '000000.000') -> '000123.780'``
* ``toString(12345, '##0.#####E0') -> '12.345E3'``
* ``toString(toDate('2018-12-31')) -> '2018-12-31'``
* ``toString(toDate('2018-12-31'), 'MM/dd/yy') -> '12/31/18'``
* ``toString(4 == 20) -> 'false'``
*********************************
### <code>toTimestamp</code>
<code><b>toTimestamp(<i>&lt;string&gt;</i> : any, [<i>&lt;timestamp format&gt;</i> : string], [<i>&lt;time zone&gt;</i> : string]) => timestamp</b></code><br/><br/>
根据可选的时间戳格式将字符串转换为日期。 所有可能的格式，请参阅 Java SimpleDateFormat。 如果省略该时间戳，则默认模式 yyyy-[M] M-[d] d hh: mm: [。.f...] 使用 * ``toTimestamp('2016-12-31 00:12:00') -> 2012-8-8T00:12:00``
* ``toTimestamp('2016/12/31T00:12:00', 'MM/dd/yyyyThh:mm:ss') -> 2012-12-12T00:12:00``
*********************************
### <code>toUTC</code>
<code><b>toUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
将时间戳转换为 UTC。 可以“GMT”、“PST”、“UTC”、“America/Cayman”格式传递可选的时区。 默认值为当前时区 * ``toUTC(currentTimeStamp()) -> 12-12-2030T19:18:12``
* ``toUTC(currentTimeStamp(), 'Asia/Seoul') -> 12-13-2030T11:18:12``
*********************************
### <code>translate</code>
<code><b>translate(<i>&lt;string to translate&gt;</i> : string, <i>&lt;lookup characters&gt;</i> : string, <i>&lt;replace characters&gt;</i> : string) => string</b></code><br/><br/>
将字符串中的一组字符替换为另一组字符。 字符具有 1 对 1 替换 * ``translate('(Hello)', '()', '[]') -> '[Hello]'``
* ``translate('(Hello)', '()', '[') -> '[Hello'``
*********************************
### <code>trim</code>
<code><b>trim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
裁剪掉前导和尾随字符的字符串。 如果未指定第二个参数，则裁剪掉空格。 否则剪裁第二个参数中指定的任何字符 * ``trim('!--!wor!ld!', '-!') -> 'wor!ld'``
*********************************
### <code>true</code>
<code><b>true() => boolean</b></code><br/><br/>
始终返回 true 值。 如果没有名为 'true' 的列，请使用函数 syntax(true()) * ``isDiscounted == true()``
* ``isDiscounted() == true``
*********************************
### <code>typeMatch</code>
<code><b>typeMatch(<i>&lt;type&gt;</i> : string, <i>&lt;base type&gt;</i> : string) => boolean</b></code><br/><br/>
匹配列的类型。 仅可在模式 expressions.number 匹配短、 整数、 长整型、 双精度，float 或 （十进制） 整数匹配简单地说，整数、 长而小数部分匹配 double、 float、 decimal 和日期时间的匹配项的日期或时间戳类型 * ``typeMatch(type, 'number') -> true``
* ``typeMatch('date', 'number') -> false``
*********************************
### <code>upper</code>
<code><b>upper(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Uppercases 字符串 * ``upper('bojjus') -> 'BOJJUS'``
*********************************
### <code>variance</code>
<code><b>variance(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的方差 * ``variance(sales) -> 122.12``
*********************************
### <code>varianceIf</code>
<code><b>varianceIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
基于条件，获取列的方差 * ``varianceIf(region == 'West', sales) -> 122.12``
*********************************
### <code>variancePopulation</code>
<code><b>variancePopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的总体方差 * ``variancePopulation(sales) -> 122.12``
*********************************
### <code>variancePopulationIf</code>
<code><b>variancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的总体方差 * ``variancePopulationIf(region == 'West', sales) -> 122.12``
*********************************
### <code>varianceSample</code>
<code><b>varianceSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
获取列的无偏差方差 * ``varianceSample(sales) -> 122.12``
*********************************
### <code>varianceSampleIf</code>
<code><b>varianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根据条件，则获取列的无偏差方差 * ``varianceSampleIf(region == 'West', sales) -> 122.12``
*********************************
### <code>weekOfYear</code>
<code><b>weekOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取给定一个日期的年份的周 * ``weekOfYear(toDate('2008-02-20')) -> 8``
*********************************
### <code>xor</code>
<code><b>xor(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
逻辑 XOR 运算符。 与相同 ^ 运算符 * ``xor(true, false) -> true``
* ``xor(true, true) -> false``
* ``true ^ false -> true``
*********************************
### <code>year</code>
<code><b>year(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
获取一个日期的年份值 * ``year(toDate('2012-8-8')) -> 2012``

## <a name="next-steps"></a>后续步骤

[了解如何使用表达式生成器](concepts-data-flow-expression-builder.md)。
