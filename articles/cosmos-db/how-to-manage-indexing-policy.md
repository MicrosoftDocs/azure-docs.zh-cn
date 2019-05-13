---
title: 管理 Azure Cosmos DB 中的索引策略
description: 了解如何管理 Azure Cosmos DB 中的索引策略
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: sample
ms.date: 05/06/2019
ms.author: thweiss
ms.openlocfilehash: 48d67c765a8a76a6058592f59eb61770e2f23df5
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65068674"
---
# <a name="manage-indexing-policies-in-azure-cosmos-db"></a>管理 Azure Cosmos DB 中的索引策略

在 Azure Cosmos DB 中，数据是按照为每个容器定义的[索引策略](index-policy.md)编制索引的。 新建容器的默认索引策略会对任何字符串或数字强制使用范围索引，对 Point 类型的任何 GeoJSON 对象强制使用空间索引。 可以采用以下方式重写此策略：

- 从 Azure 门户
- 使用 Azure CLI
- 使用某个 SDK

[索引策略更新](index-policy.md#modifying-the-indexing-policy)会触发索引转换。 还可以通过 SDK 跟踪此转换的进度。

## <a name="use-the-azure-portal"></a>使用 Azure 门户

Azure Cosmos 容器将其索引策略存储为 JSON 文档，可以在 Azure 门户中直接编辑这些文档。

1. 登录到 [Azure 门户](https://portal.azure.com/)。

1. 创建新的 Azure Cosmos 帐户或选择现有的帐户。

1. 打开“数据资源管理器”窗格，选择要使用的容器。

1. 单击“缩放设置”。

1. 修改索引策略 JSON 文档（请参阅[下文](#indexing-policy-examples)中的示例）

1. 完成后，单击“保存”。

![使用 Azure 门户管理索引编制](./media/how-to-manage-indexing-policy/indexing-policy-portal.png)

## <a name="use-the-azure-cli"></a>使用 Azure CLI

可以通过 Azure CLI 中的 [az cosmosdb collection update](/cli/azure/cosmosdb/collection#az-cosmosdb-collection-update) 命令替换容器的索引策略的 JSON 定义：

```azurecli-interactive
az cosmosdb collection update \
    --resource-group-name $resourceGroupName \
    --name $accountName \
    --db-name $databaseName \
    --collection-name $containerName \
    --indexing-policy "{\"indexingMode\": \"consistent\", \"includedPaths\": [{ \"path\": \"/*\", \"indexes\": [{ \"dataType\": \"String\", \"kind\": \"Range\" }] }], \"excludedPaths\": [{ \"path\": \"/headquarters/employees/?\" } ]}"
```

## <a name="use-the-net-sdk-v2"></a>使用 .NET SDK V2

[.NET SDK v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) 中的 `DocumentCollection` 对象（请参阅有关其用法的[此快速入门](create-sql-api-dotnet.md)）公开了一个 `IndexingPolicy` 属性，可以通过该属性更改 `IndexingMode` 以及添加或删除 `IncludedPaths` 和 `ExcludedPaths`。

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> containerResponse = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"));
// set the indexing mode to Consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/headquarters/employees/?" });
// update the container with our changes
await client.ReplaceDocumentCollectionAsync(containerResponse.Resource);
```

若要跟踪索引转换进度，请传递一个 `RequestOptions` 对象，用以将 `PopulateQuotaInfo` 属性设置为 `true`。

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> container = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"), new RequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = container.IndexTransformationProgress;
```

## <a name="use-the-java-sdk"></a>使用 Java SDK

[Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) 中的 `DocumentCollection` 对象（请参阅有关其用法的[此快速入门](create-sql-api-java.md)）公开了 `getIndexingPolicy()` 和 `setIndexingPolicy()` 方法。 通过它们操作的 `IndexingPolicy` 对象，你可以更改索引模式，以及添加或删除包括的和排除的路径。

```java
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), null);
containerResponse.subscribe(result -> {
    DocumentCollection container = result.getResource();
    IndexingPolicy indexingPolicy = container.getIndexingPolicy();
    // set the indexing mode to Consistent
    indexingPolicy.setIndexingMode(IndexingMode.Consistent);
    Collection<ExcludedPath> excludedPaths = new ArrayList<>();
    ExcludedPath excludedPath = new ExcludedPath();
    excludedPath.setPath("/*");
    // add an excluded path
    excludedPaths.add(excludedPath);
    indexingPolicy.setExcludedPaths(excludedPaths);
    // update the container with our changes
    client.replaceCollection(container, null);
});
```

若要在容器上跟踪索引转换进度，请传递一个用以请求要填充的配额信息的 `RequestOptions` 对象，然后从 `x-ms-documentdb-collection-index-transformation-progress` 响应标头中检索该值。

```java
// set the RequestOptions object
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPopulateQuotaInfo(true);
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), requestOptions);
containerResponse.subscribe(result -> {
    // retrieve the index transformation progress from the response headers
    String indexTransformationProgress = result.getResponseHeaders().get("x-ms-documentdb-collection-index-transformation-progress");
});
```

## <a name="use-the-nodejs-sdk"></a>使用 Node.js SDK

[Node.js SDK](https://www.npmjs.com/package/@azure/cosmos) 中的 `ContainerDefinition` 接口（请参阅有关其用法的[此快速入门](create-sql-api-nodejs.md)）公开了一个 `indexingPolicy` 属性，可以通过该属性更改 `indexingMode` 以及添加或删除 `includedPaths` 和 `excludedPaths`。

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read();
// set the indexing mode to Consistent
containerResponse.body.indexingPolicy.indexingMode = "consistent";
// add an excluded path
containerResponse.body.indexingPolicy.excludedPaths.push({ path: '/headquarters/employees/?' });
// update the container with our changes
const replaceResponse = await client.database('database').container('container').replace(containerResponse.body);
```

若要在容器上跟踪索引转换进度，请传递一个用以将 `populateQuotaInfo` 属性设置为 `true` 的 `RequestOptions` 对象，然后从 `x-ms-documentdb-collection-index-transformation-progress` 响应标头中检索该值。

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read({
    populateQuotaInfo: true
});
// retrieve the index transformation progress from the response headers
const indexTransformationProgress = replaceResponse.headers['x-ms-documentdb-collection-index-transformation-progress'];
```

## <a name="use-the-python-sdk"></a>使用 Python SDK

使用 [Python SDK](https://pypi.org/project/azure-cosmos/) 时（请参阅有关其用法的[此快速入门](create-sql-api-python.md)），容器配置是作为字典管理的。 从此字典中，可以访问索引策略及其所有属性。

```python
containerPath = 'dbs/database/colls/collection'
# retrieve the container's details
container = client.ReadContainer(containerPath)
# set the indexing mode to Consistent
container['indexingPolicy']['indexingMode'] = 'consistent'
# add an excluded path
container['indexingPolicy']['excludedPaths'] = [{"path" : "/headquarters/employees/?"}]
# update the container with our changes
response = client.ReplaceContainer(containerPath, container)
```

## <a name="indexing-policy-examples"></a>索引策略示例

下面是以 JSON 格式显示的一些索引策略示例，该格式是在 Azure 门户上公开索引策略的方式。 可以通过 Azure CLI 或任何 SDK 设置相同的参数。

### <a name="opt-out-policy-to-selectively-exclude-some-property-paths"></a>用以有选择地排除某些属性路径的选择退出策略
```
    {
        "indexingPolicy": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    },
                    {
                        "kind": "Spatial",
                        "dataType": "Point"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

### <a name="opt-in-policy-to-selectively-include-some-property-paths"></a>用以有选择地包括某些属性路径的选择加入策略
```
    {
        "indexingPolicy": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    }
                ]
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

注意：通常情况下，建议使用**选择退出**索引策略来让 Azure Cosmos DB 主动为可能会添加到模型的任何新属性编制索引。

### <a name="using-a-spatial-index-on-a-specific-property-path-only"></a>仅在特定属性路径上使用空间索引
```
    {
        "indexingPolicy": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            },
            {
                "path": "/path/to/geojson/property/?",
                "indexes": [
                    {
                        "kind": "Spatial",
                        "dataType": "Point"
                    }
                ]
            }
        ],
        "excludedPaths": []
    }
```

### <a name="excluding-all-property-paths-but-keeping-indexing-active"></a>排除所有属性路径，但使索引保持活动状态

当[生存时间 (TTL) 功能](time-to-live.md)处于活动状态但不需要第二索引时（使用 Azure Cosmos DB 作为纯键-值存储），可以使用此策略。
```
    {
        "indexingMode": "consistent",
        "includedPaths": [],
        "excludedPaths": [{
            "path": "/*"
        }]
    }
```

### <a name="no-indexing"></a>无索引
```
    {
        "indexingPolicy": "none"
    }
```

## <a name="composite-indexing-policy-examples"></a>组合索引策略示例

除了包含或排除各属性的路径，还可以指定一个组合索引。 如果要执行具有针对多个属性的 `ORDER BY` 子句的查询，需要使用这些属性上的[组合索引](index-policy.md#composite-indexes)。

### <a name="composite-index-defined-for-name-asc-age-desc"></a>针对（name asc、age desc）定义的组合索引：
```
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[  

        ],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

此组合索引能够支持以下两个查询：

查询 #1：
```sql
    SELECT *
    FROM c
    ORDER BY name asc, age desc    
```

查询 #2：
```sql
    SELECT *
    FROM c
    ORDER BY name desc, age asc
```

### <a name="composite-index-defined-for-name-asc-age-asc-and-name-asc-age-desc"></a>针对（name asc、age asc）和（name asc、age desc）定义的组合索引：

可以在同一个索引策略中定义多个不同的组合索引。 
```
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[  

        ],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"ascending"
                }
            ]
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

### <a name="composite-index-defined-for-name-asc-age-asc"></a>针对（name asc、age asc）定义的组合索引：

可以选择指定顺序。 如果未指定，顺序为升序。
```
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[  

        ],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                },
                {  
                    "path":"/age",
                }
            ]
        ]
}
```

## <a name="next-steps"></a>后续步骤

阅读以下文章中有关索引的详细信息：

- [索引概述](index-overview.md)
- [索引策略](index-policy.md)
