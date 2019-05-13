---
title: 加载：数据准备 Python SDK
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习数据准备 SDK 加载数据。 可以加载不同类型的输入数据，指定数据文件类型和参数，或使用 SDK 智能读取功能自动检测文件类型。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: jmartens
ms.date: 02/22/2019
ms.custom: seodec18
ms.openlocfilehash: e7c330846cd907f35bb23ae5e453383d7c35222e
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65471882"
---
# <a name="load-and-read-data-with-the-azure-machine-learning-data-prep-sdk"></a>加载和使用 Azure 机器学习数据准备 SDK 读取数据
在本文中，您将学习使用 Azure 机器学习数据准备 SDK 将数据加载不同的方法。  SDK 支持多个数据引入功能，包括：

* 使用分析参数推理（编码、分隔符和标头）从多种文件类型进行加载
* 在文件加载时使用推理进行类型转换
* 支持 MS SQL Server 和 Azure Data Lake Storage 连接

> [!Important]
> 如果要构建一个新的解决方案，请尝试[Azure 机器学习数据集](how-to-explore-prepare-data.md)（预览），数据探索和准备工作。 数据集是数据准备 SDK，提供可用于管理 AI 解决方案中的数据集的扩展的功能的下一个版本。
> 如果您使用`azureml-dataprep`包来创建数据流而不是使用转换与`azureml-datasets`的包装，以创建数据集，您将不能以供以后使用快照或版本控制的数据集。

下表显示了一系列函数用于从常见的文件类型加载数据。

| 文件类型 | 函数 | 参考链接 |
|-------|-------|-------|
|任意|`auto_read_file()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#auto-read-file-path--filepath--include-path--bool---false-----azureml-dataprep-api-dataflow-dataflow)|
|Text|`read_lines()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#read-lines-path--filepath--header--azureml-dataprep-api-engineapi-typedefinitions-promoteheadersmode----promoteheadersmode-none--0---encoding--azureml-dataprep-api-engineapi-typedefinitions-fileencoding----fileencoding-utf8--0---skip-rows--int---0--skip-mode--azureml-dataprep-api-engineapi-typedefinitions-skipmode----skipmode-none--0---comment--str---none--include-path--bool---false-----azureml-dataprep-api-dataflow-dataflow)|
|CSV|`read_csv()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#read-csv-path--filepath--separator--str--------header--azureml-dataprep-api-engineapi-typedefinitions-promoteheadersmode----promoteheadersmode-constantgrouped--3---encoding--azureml-dataprep-api-engineapi-typedefinitions-fileencoding----fileencoding-utf8--0---quoting--bool---false--inference-arguments--azureml-dataprep-api-builders-inferencearguments---none--skip-rows--int---0--skip-mode--azureml-dataprep-api-engineapi-typedefinitions-skipmode----skipmode-none--0---comment--str---none--include-path--bool---false--archive-options--azureml-dataprep-api--archiveoption-archiveoptions---none-----azureml-dataprep-api-dataflow-dataflow)|
|Excel|`read_excel()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#read-excel-path--filepath--sheet-name--str---none--use-column-headers--bool---false--inference-arguments--azureml-dataprep-api-builders-inferencearguments---none--skip-rows--int---0--include-path--bool---false-----azureml-dataprep-api-dataflow-dataflow)|
|固定宽度|`read_fwf()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#read-fwf-path--filepath--offsets--typing-list-int---header--azureml-dataprep-api-engineapi-typedefinitions-promoteheadersmode----promoteheadersmode-constantgrouped--3---encoding--azureml-dataprep-api-engineapi-typedefinitions-fileencoding----fileencoding-utf8--0---inference-arguments--azureml-dataprep-api-builders-inferencearguments---none--skip-rows--int---0--skip-mode--azureml-dataprep-api-engineapi-typedefinitions-skipmode----skipmode-none--0---include-path--bool---false-----azureml-dataprep-api-dataflow-dataflow)|
|JSON|`read_json()`|[reference](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#read-json-path--filepath--encoding--azureml-dataprep-api-engineapi-typedefinitions-fileencoding----fileencoding-utf8--0---flatten-nested-arrays--bool---false--include-path--bool---false-----azureml-dataprep-api-dataflow-dataflow)|

## <a name="load-data-automatically"></a>自动加载数据

若要在不指定文件类型的情况下自动加载数据，请使用 `auto_read_file()` 函数。 会自动推断文件的类型和读取它所需的参数。

```python
import azureml.dataprep as dprep

dflow = dprep.auto_read_file(path='./data/any-file.txt')
```

此函数对于从一个方便的入口点自动检测文件类型、编码和其他分析参数非常有用。 此函数还会自动执行加载分隔数据时通常执行的以下步骤：

* 推断和设置分隔符
* 跳过文件顶部的空记录
* 推断和设置标题行

或者，如果你知道文件提前键入并且想要显式控制其解析的方法，使用特定于文件的函数。

## <a name="load-text-line-data"></a>加载文本行数据

若要将简单文本数据读取到数据流中，请使用 `read_lines()`，无需指定可选参数。

```python
dflow = dprep.read_lines(path='./data/text_lines.txt')
dflow.head(5)
```

||行|
|----|-----|
|0|日期\|\|  最低温度\|\|  最高温度|
|1|2015-07-1 \|\|  -4.1 \|\|  10.0|
|2|2015-07-2 \|\|  -0.8 \|\|  10.8|


引入数据后，运行以下代码，将数据流对象转换为 Pandas 数据帧。

```python
pandas_df = dflow.to_pandas_dataframe()
```

## <a name="load-csv-data"></a>加载 CSV 数据

在读取带分隔符的文件时，可使用基础运行时推断分析参数（分隔符、编码、是否使用标头等）。 运行以下代码，尝试通过仅指定文件的位置来读取 CSV 文件。

```python
dflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv?st=2018-06-15T23%3A01%3A42Z&se=2019-06-16T23%3A01%3A00Z&sp=r&sv=2017-04-17&sr=b&sig=ugQQCmeC2eBamm6ynM7wnI%2BI3TTDTM6z9RPKj4a%2FU6g%3D')
dflow.head(5)
```

| |stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|------|-----|
|0|stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|第|阿拉巴马州|1|101710|Hale County|10171002158| |
|2|阿拉巴马州|第|101710|Hale County|10171002162| |


若要在加载过程中排除行，请定义 `skip_rows` 参数。 此参数将跳过加载在 CSV 文件中以降序排列的行（使用基于 1 的索引）。

```python
dflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv',
                          skip_rows=1)
dflow.head(5)
```

| |stnam|fipst|leaid|leanm10|ncessch|MAM_MTH00numvalid_1011|
|-----|-------|---------| -------|------|-----|------|
|0|阿拉巴马州|第|101710|Hale County|10171002158|29|
|1|阿拉巴马州|第|101710|Hale County|10171002162|40 |

运行以下代码，显示列数据类型。

```python
dflow.dtypes
```
输出：

    stnam                     object
    fipst                     object
    leaid                     object
    leanm10                   object
    ncessch                   object
    schnam10                  object
    MAM_MTH00numvalid_1011    object
    dtype: object

默认情况下，Azure 机器学习数据准备 SDK 不会更改数据类型。 正在读取的数据源是一个文本文件，因此 SDK 会将所有值读取为字符串。 在本示例中，应将数值列分析为数字。 将 `inference_arguments` 参数设置为 `InferenceArguments.current_culture()` 以在文件读取期间自动推断和转换列类型。

```
dflow = dprep.read_csv(path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv',
                          skip_rows=1,
                          inference_arguments=dprep.InferenceArguments.current_culture())
dflow.dtypes
```
输出：

    stnam                      object
    fipst                     float64
    leaid                     float64
    leanm10                    object
    ncessch                   float64
    schnam10                   object
    ALL_MTH00numvalid_1011    float64
    dtype: object


多个列已正确检测为数字且其类型设置为 `float64`。

## <a name="use-excel-data"></a>使用 Excel 数据

SDK 包括可以加载 Excel 文件的 `read_excel()` 函数。 默认情况下，该函数将加载工作簿中的第一个工作表。 若要定义要加载的特定工作表，请使用工作表名称的字符串值定义 `sheet_name` 参数。

```python
dflow = dprep.read_excel(path='./data/excel.xlsx', sheet_name='Sheet2')
dflow.head(5)
```

| |Column1|Column2|Column3|Column4|Column5|Column6|Column7|Column8| | |
|-|-------|-------|-------|-------|-------|-------|-------|-------|-|-|
|0|无|无|无|无|无|无|无|无|无| |
|第|无|无|无|无|无|无|无|无|无| |
|2|无|无|无|无|无|无|无|无|无| |
|3|设置级别|职务|工作室|全球|国内 / %|Column1|海外 / %|Column2|年份^| |
|4|第|Avatar|Fox|2788|760.5|0.273|2027.5|0.727|2009^|5|

输出显示第二个工作表中的数据在标头前有三个空行。 `read_excel()` 函数包含用于跳过行和使用标头的可选参数。 运行以下代码以跳过前三行，并将第四行用作标头。

```python
dflow = dprep.read_excel(path='./data/excel.xlsx', sheet_name='Sheet2', use_column_headers=True, skip_rows=3)
```

||设置级别|职务|工作室|全球|国内 / %|Column1|海外 / %|Column2|年份^|
|------|------|------|-----|------|-----|-------|----|-----|-----|
|0|1|Avatar|Fox|2788|760.5|0.273|2027.5|0.727|2009^|
|第|2|Titanic|Par.|2186.8|658.7|0.301|1528.1|0.699|1997^|

## <a name="load-fixed-width-data-files"></a>加载固定宽度数据文件

若要加载固定宽度文件，您指定的字符偏移量。 始终假定第一列从偏移量零处开始。

```python
dflow = dprep.read_fwf('./data/fixed_width_file.txt', offsets=[7, 13, 43, 46, 52, 58, 65, 73])
dflow.head(5)
```

||010000|99999|BOGUS NORWAY|否|NO_1|ENRS|Column7|Column8|Column9|
|------|------|------|-----|------|-----|-------|----|-----|----|
|0|010003|99999|BOGUS NORWAY|否|否|ENSO||||
|1|010010|99999|JAN MAYEN（扬马延岛）|否|JN|ENJA|+70933|-008667|+00090|


若要避免标头检测和分析正确数据，请将 `PromoteHeadersMode.NONE` 传递到 `header` 参数。

```python
dflow = dprep.read_fwf('./data/fixed_width_file.txt',
                          offsets=[7, 13, 43, 46, 52, 58, 65, 73],
                          header=dprep.PromoteHeadersMode.NONE)
```

||Column1|Column2|Column3|Column4|Column5|Column6|Column7|Column8|Column9|
|------|------|------|-----|------|-----|-------|----|-----|----|
|0|010000|99999|BOGUS NORWAY|否|NO_1|ENRS|Column7|Column8|Column9|
|第|010003|99999|BOGUS NORWAY|否|否|ENSO||||


## <a name="load-sql-data"></a>加载 SQL 数据

SDK 还可以从 SQL 源加载数据。 目前，仅支持 Microsoft SQL Server。 若要从 SQL server 读取数据，创建[ `MSSQLDataSource` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.mssqldatasource?view=azure-dataprep-py)对象，其中包含连接参数。 密码参数的`MSSQLDataSource`接受[ `Secret` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep?view=azure-dataprep-py#register-secret-value--str--id--str---none-----azureml-dataprep-api-engineapi-typedefinitions-secret)对象。 可以通过两种方式来生成机密对象：

* 使用执行引擎注册机密及其值。
* 使用 `dprep.create_secret("[SECRET-ID]")` 且仅使用 `id`（如果已在执行环境中注册机密值）创建机密。

```python
secret = dprep.register_secret(value="[SECRET-PASSWORD]", id="[SECRET-ID]")

ds = dprep.MSSQLDataSource(server_name="[SERVER-NAME]",
                           database_name="[DATABASE-NAME]",
                           user_name="[DATABASE-USERNAME]",
                           password=secret)
```

创建数据源对象后，可以继续从查询输出中读取数据。

```python
dflow = dprep.read_sql(ds, "SELECT top 100 * FROM [SalesLT].[Product]")
dflow.head(5)
```

| |ProductID|名称|ProductNumber|颜色|StandardCost|ListPrice|大小|权重|ProductCategoryID|ProductModelID|SellStartDate|SellEndDate|DiscontinuedDate|ThumbNailPhoto|ThumbnailPhotoFileName|rowguid|ModifiedDate| |
|-|---------|----|-------------|-----|------------|---------|----|------|-----------------|--------------|-------------|-----------|----------------|--------------|----------------------|-------|------------|-|
|0|680|HL Road Frame - 黑色，58|FR-R92B-58|黑色|1059.3100|1431.50|58|1016.04|18|6|2002-06-01 00:00:00+00:00|无|无|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|43dd68d6-14a4-461f-9069-55309d90ea7e|2008-03-11 |0:01:36.827000+00:00|
|第|706|HL Road Frame - 红色，58|FR-R92R-58|红色|1059.3100|1431.50|58|1016.04|18|6|2002-06-01 00:00:00+00:00|无|无|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|9540ff17-2712-4c90-a3d1-8ce5568b2462|2008-03-11 |10:01:36.827000+00:00|
|2|707|Sport-100 Helmet，红色|HL-U509-R|红色|13.0863|34.99|无|无|35|33|2005-07-01 00:00:00+00:00|无|无|b'GIF89aP\x001\x00\xf7\x00\x00\x00\x00\x00\x80...|no_image_available_small.gif|2e1ef41a-c08a-4ff6-8ada-bde58b64a712|2008-03-11 |10:01:36.827000+00:00|


## <a name="use-azure-data-lake-storage"></a>使用 Azure Data Lake Storage

SDK 可以通过两种方式获取访问 Azure Data Lake Storage 所需的 OAuth 令牌：

* 从用户的 Azure CLI 登录的最近会话中检索访问令牌
* 使用服务主体 (SP) 和证书作为机密

### <a name="use-an-access-token-from-a-recent-azure-cli-session"></a>使用最近 Azure CLI 会话中的访问令牌

在本地计算机上，运行以下命令。

```azurecli
az login
az account show --query tenantId
dflow = read_csv(path = DataLakeDataSource(path='adl://dpreptestfiles.azuredatalakestore.net/farmers-markets.csv', tenant='microsoft.onmicrosoft.com')) head = dflow.head(5) head
```

> [!NOTE]
> 如果用户帐户是多个 Azure 租户的成员，则需要在 AAD URL 主机名窗体中指定租户。

### <a name="create-a-service-principal-with-the-azure-cli"></a>使用 Azure CLI 创建服务主体

使用 Azure CLI 创建服务主体和相应的证书。 此特定服务主体配置为 `reader` 角色，其范围缩小到仅限于 Azure Data Lake Storage 帐户“dpreptestfiles”。

```azurecli
az account set --subscription "Data Wrangling development"
az ad sp create-for-rbac -n "SP-ADLS-dpreptestfiles" --create-cert --role reader --scopes /subscriptions/35f16a99-532a-4a47-9e93-00305f6c40f2/resourceGroups/dpreptestfiles/providers/Microsoft.DataLakeStore/accounts/dpreptestfiles
```

此命令发出 `appId` 和证书文件的路径（通常位于主文件夹中）。 .crt 文件包含 PEM 格式的公共证书和私钥。

```
openssl x509 -in adls-dpreptestfiles.crt -noout -fingerprint
```

若要配置 Azure Data Lake Storage 文件系统的 ACL，请使用用户的 objectId。 在本示例中，使用服务主体。

```azurecli
az ad sp show --id "8dd38f34-1fcb-4ff9-accd-7cd60b757174" --query objectId
```

若要为 Azure Data Lake Storage 文件系统配置 `Read` 和 `Execute` 访问权限，请单独为文件夹和文件配置 ACL。 这是因为底层 HDFS ACL 模型不支持继承。

```azurecli
az dls fs access set-entry --account dpreptestfiles --acl-spec "user:e37b9b1f-6a5e-4bee-9def-402b956f4e6f:r-x" --path /
az dls fs access set-entry --account dpreptestfiles --acl-spec "user:e37b9b1f-6a5e-4bee-9def-402b956f4e6f:r--" --path /farmers-markets.csv
```

```
certThumbprint = 'C2:08:9D:9E:D1:74:FC:EB:E9:7E:63:96:37:1C:13:88:5E:B9:2C:84'
certificate = ''
with open('./data/adls-dpreptestfiles.crt', 'rt', encoding='utf-8') as crtFile:
    certificate = crtFile.read()

servicePrincipalAppId = "8dd38f34-1fcb-4ff9-accd-7cd60b757174"
```

### <a name="acquire-an-oauth-access-token"></a>获取 OAuth 访问令牌

使用 `adal` 包 (`pip install adal`) 在 MSFT 租户上创建身份验证上下文，并获取 OAuth 访问令牌。 对于 ADLS，令牌请求中的资源必须为 https:\//datalake.azure.net，这是不同于大多数其他 Azure 资源。

```python
import adal
from azureml.dataprep.api.datasources import DataLakeDataSource

ctx = adal.AuthenticationContext('https://login.microsoftonline.com/microsoft.onmicrosoft.com')
token = ctx.acquire_token_with_client_certificate('https://datalake.azure.net/', servicePrincipalAppId, certificate, certThumbprint)
dflow = dprep.read_csv(path = DataLakeDataSource(path='adl://dpreptestfiles.azuredatalakestore.net/farmers-markets.csv', accessToken=token['accessToken']))
dflow.to_pandas_dataframe().head()
```

||FMID|MarketName|网站|street|city|郡/县|
|----|------|-----|----|----|----|----|
|0|1012063|喀里多尼亚农贸市场协会 - 丹维尔|https://sites.google.com/site/caledoniafarmers.. ||丹维尔|喀里多尼亚|
|第|1011871|斯特恩斯家园农贸市场|http://Stearnshomestead.com |6975 Ridge Road|帕尔马|凯霍加河|
|2|1011878|100 英里市场|https://www.pfcmarkets.com |507 哈里森街|卡拉马祖|卡拉马祖|
|3|1009364|106 S. 主要街道农贸市场|http://thetownofsixmile.wordpress.com/ |106 S. 主要街道|六英里|||
|4|1010691|第 10 街社区农贸市场|https://agrimissouri.com/.. |第十街和波普拉区|拉马尔|巴顿|

## <a name="next-steps"></a>后续步骤

* 请参阅 Azure 机器学习数据准备 SDK[教程](tutorial-data-prep.md)有关解决特定方案的示例
