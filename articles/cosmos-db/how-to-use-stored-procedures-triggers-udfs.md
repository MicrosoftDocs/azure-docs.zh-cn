---
title: 了解如何使用 Azure Cosmos DB SDK 调用存储过程、触发器和用户定义的函数
description: 了解如何使用 Azure Cosmos DB SDK 注册和调用存储过程、触发器与用户定义的函数
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 12/08/2018
ms.author: mjbrown
ms.openlocfilehash: d271c66ecf1716e4ca264aadb9685b9c03608ddc
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506717"
---
# <a name="how-to-register-and-use-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>如何在 Azure Cosmos DB 中注册和使用存储过程、触发器与用户定义的函数

Azure Cosmos DB 中的 SQL API 支持注册和调用以 JavaScript 编写的存储过程、触发器与用户定义函数 (UDF)。 可以使用 SQL API [.NET](sql-api-sdk-dotnet.md)、[.NET Core](sql-api-sdk-dotnet-core.md)、[Java](sql-api-sdk-java.md)、[JavaScript](sql-api-sdk-node.md)、[Node.js](sql-api-sdk-node.md) 或 [Python](sql-api-sdk-python.md) SDK 注册和调用存储过程。 在定义一个或多个存储过程、触发器和用户定义的函数之后，可以使用数据资源管理器在 [Azure 门户](https://portal.azure.com/)中加载和查看它们。

## <a id="stored-procedures"></a>如何运行存储过程

存储过程是使用 JavaScript 编写的。 它们可以在 Azure Cosmos 容器中创建、更新、读取、查询和删除项。 有关如何在 Azure Cosmos DB 中编写存储过程的详细信息，请参阅[如何在 Azure Cosmos DB 中编写存储过程](how-to-write-stored-procedures-triggers-udfs.md#stored-procedures)一文。

以下示例演示如何使用 Azure Cosmos DB SDK 注册和调用存储过程。 请参阅[创建文档](how-to-write-stored-procedures-triggers-udfs.md#create-an-item)，因为此存储过程的源代码保存为 `spCreateToDoItem.js`。

> [!NOTE]
> 对于已分区的容器，在执行存储过程时，必须在请求选项中提供分区键值。 存储过程的范围始终限定为分区键。 存储过程看不到具有不同分区键值的项。 这一点也适用于触发器。

### <a name="stored-procedures---net-sdk"></a>存储过程 - .NET SDK

以下示例演示如何使用 .NET SDK 注册存储过程：

```csharp
string storedProcedureId = "spCreateToDoItem";
StoredProcedure newStoredProcedure = new StoredProcedure
   {
       Id = storedProcedureId,
       Body = File.ReadAllText($@"..\js\{storedProcedureId}.js")
   };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var response = await client.CreateStoredProcedureAsync(containerUri, newStoredProcedure);
StoredProcedure createdStoredProcedure = response.Resource;
```

以下代码演示如何使用 .NET SDK 调用存储过程：

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

Uri uri = UriFactory.CreateStoredProcedureUri("myDatabase", "myContainer", "spCreateToDoItem");
RequestOptions options = new RequestOptions { PartitionKey = new PartitionKey("Personal") };
var result = await client.ExecuteStoredProcedureAsync<string>(uri, options, newItem);
var id = result.Response;
```

### <a name="stored-procedures---java-sdk"></a>存储过程 - Java SDK

以下示例演示如何使用 Java SDK 注册存储过程：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
StoredProcedure newStoredProcedure = new StoredProcedure(
    "{" +
        "  'id':'spCreateToDoItem'," +
        "  'body':" + new String(Files.readAllBytes(Paths.get("..\\js\\spCreateToDoItem.js"))) +
    "}");
//toBlocking() blocks the thread until the operation is complete and is used only for demo.  
StoredProcedure createdStoredProcedure = asyncClient.createStoredProcedure(containerLink, newStoredProcedure, null)
    .toBlocking().single().getResource();
```

以下代码演示如何使用 Java SDK 调用存储过程：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String sprocLink = String.format("%s/sprocs/%s", containerLink, "spCreateToDoItem");
final CountDownLatch successfulCompletionLatch = new CountDownLatch(1);

class ToDoItem {
    public String category;
    public String name;
    public String description;
    public boolean isComplete;
}

ToDoItem newItem = new ToDoItem();
newItem.category = "Personal";
newItem.name = "Groceries";
newItem.description = "Pick up strawberries";
newItem.isComplete = false;

RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("Personal"));

Object[] storedProcedureArgs = new Object[] { newItem };
asyncClient.executeStoredProcedure(sprocLink, requestOptions, storedProcedureArgs)
    .subscribe(storedProcedureResponse -> {
        String storedProcResultAsString = storedProcedureResponse.getResponseAsString();
        successfulCompletionLatch.countDown();
        System.out.println(storedProcedureResponse.getActivityId());
    }, error -> {
        System.err.println("an error occurred while executing the stored procedure: actual cause: "
                + error.getMessage());
    });

successfulCompletionLatch.await();
```

### <a name="stored-procedures---javascript-sdk"></a>存储过程 - JavaScript SDK

以下示例演示如何使用 JavaScript SDK 注册存储过程

```javascript
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItem";
await container.storedProcedures.create({
    id: sprocId,
    body: require(`../js/${sprocId}`)
});
```

以下代码演示如何使用 JavaScript SDK 调用存储过程：

```javascript
const newItem = [{
    category: "Personal",
    name: "Groceries",
    description: "Pick up strawberries",
    isComplete: false
}];
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItem";
const {body: result} = await container.storedProcedure(sprocId).execute(newItem, {partitionKey: newItem[0].category});
```

### <a name="stored-procedures---python-sdk"></a>存储过程 - Python SDK

以下示例演示如何使用 Python SDK 注册存储过程

```python
with open('../js/spCreateToDoItem.js') as file:
    file_contents = file.read()
container_link = 'dbs/myDatabase/colls/myContainer'
sproc_definition = {
            'id': 'spCreateToDoItem',
            'serverScript': file_contents,
        }
sproc = client.CreateStoredProcedure(container_link, sproc_definition)
```

以下代码演示如何使用 Python SDK 调用存储过程

```python
sproc_link = 'dbs/myDatabase/colls/myContainer/sprocs/spCreateToDoItem'
new_item = [{
    'category':'Personal',
    'name':'Groceries',
    'description':'Pick up strawberries',
    'isComplete': False
}]
client.ExecuteStoredProcedure(sproc_link, new_item, {'partitionKey': 'Personal'}
```

## <a id="pre-triggers"></a>如何运行前触发器

以下示例演示如何使用 Azure Cosmos DB SDK 注册和调用前触发器。 请参阅[前触发器示例](how-to-write-stored-procedures-triggers-udfs.md#pre-triggers)，因为此前触发器的源代码保存为 `trgPreValidateToDoItemTimestamp.js`。

执行前触发器时，将会通过指定 `PreTriggerInclude` 在该触发器中传入 RequestOptions 对象，然后在 List 对象中传递该触发器的名称。

> [!NOTE]
> 即使以列表的形式传递触发器的名称，也仍只能对每个操作执行一个触发器。

### <a name="pre-triggers---net-sdk"></a>前触发器 - .NET SDK

以下代码演示如何使用 .NET SDK 注册前触发器：

```csharp
string triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger
{
    Id =  triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

以下代码演示如何使用 .NET SDK 调用前触发器：

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
RequestOptions requestOptions = new RequestOptions { PreTriggerInclude = new List<string> { "trgPreValidateToDoItemTimestamp" } };
await client.CreateDocumentAsync(containerUri, newItem, requestOptions);
```

### <a name="pre-triggers---java-sdk"></a>前触发器 - Java SDK

以下代码演示如何使用 Java SDK 注册前触发器：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Pre);
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

以下代码演示如何使用 Java SDK 调用前触发器：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
    Document item = new Document("{ "
            + "\"category\": \"Personal\", "
            + "\"name\": \"Groceries\", "
            + "\"description\": \"Pick up strawberries\", "
            + "\"isComplete\": false, "
            + "}"
            );
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPreTriggerInclude(Arrays.asList("trgPreValidateToDoItemTimestamp"));
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="pre-triggers---javascript-sdk"></a>前触发器 - JavaScript SDK

以下代码演示如何使用 JavaScript SDK 注册前触发器：

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "pre"
});
```

以下代码演示如何使用 JavaScript SDK 调用前触发器：

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.items.create({
    category: "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
}, {preTriggerInclude: [triggerId]});
```

### <a name="pre-triggers---python-sdk"></a>前触发器 - Python SDK

以下代码演示如何使用 Python SDK 注册前触发器：

```python
with open('../js/trgPreValidateToDoItemTimestamp.js') as file:
    file_contents = file.read()
container_link = 'dbs/myDatabase/colls/myContainer'
trigger_definition = {
            'id': 'trgPreValidateToDoItemTimestamp',
            'serverScript': file_contents,
            'triggerType': documents.TriggerType.Pre,
            'triggerOperation': documents.TriggerOperation.Create
        }
trigger = client.CreateTrigger(container_link, trigger_definition)
```

以下代码演示如何使用 Python SDK 调用前触发器：

```python
container_link = 'dbs/myDatabase/colls/myContainer'
item = { 'category': 'Personal', 'name': 'Groceries', 'description':'Pick up strawberries', 'isComplete': False}
client.CreateItem(container_link, item, { 'preTriggerInclude': 'trgPreValidateToDoItemTimestamp'})
```

## <a id="post-triggers"></a>如何运行后触发器

以下示例演示如何使用 Azure Cosmos DB SDK 注册后触发器。 请参阅[后触发器示例](how-to-write-stored-procedures-triggers-udfs.md#post-triggers)，因为此后触发器的源代码保存为 `trgPostUpdateMetadata.js`。

### <a name="post-triggers---net-sdk"></a>后触发器 - .NET SDK

以下代码演示如何使用 .NET SDK 注册后触发器：

```csharp
string triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger
{
    Id = triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

以下代码演示如何使用 .NET SDK 调用后触发器：

```csharp
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

var options = { postTriggerInclude: "trgPostUpdateMetadata" };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.createDocumentAsync(containerUri, newItem, options);
```

### <a name="post-triggers---java-sdk"></a>后触发器 - Java SDK

以下代码演示如何使用 Java SDK 注册后触发器：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)))));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Post);
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

以下代码演示如何使用 Java SDK 调用后触发器：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Document item = new Document(String.format("{ "
    + "\"name\": \"artist_profile_1023\", "
    + "\"artist\": \"The Band\", "
    + "\"albums\": [\"Hellujah\", \"Rotators\", \"Spinning Top\"]"
    + "}"
));
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPostTriggerInclude(Arrays.asList("trgPostUpdateMetadata"));
//toBlocking() blocks the thread until the operation is complete, and is used only for demo.
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="post-triggers---javascript-sdk"></a>后触发器 - JavaScript SDK

以下代码演示如何使用 JavaScript SDK 注册后触发器：

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "post"
});
```

以下代码演示如何使用 JavaScript SDK 调用后触发器：

```javascript
const item = {
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.items.create(item, {postTriggerInclude: [triggerId]});
```

### <a name="post-triggers---python-sdk"></a>后触发器 - Python SDK

以下代码演示如何使用 Python SDK 注册后触发器：

```python
with open('../js/trgPostUpdateMetadata.js') as file:
    file_contents = file.read()
container_link = 'dbs/myDatabase/colls/myContainer'
trigger_definition = {
            'id': 'trgPostUpdateMetadata',
            'serverScript': file_contents,
            'triggerType': documents.TriggerType.Post,
            'triggerOperation': documents.TriggerOperation.Create
        }
trigger = client.CreateTrigger(container_link, trigger_definition)
```

以下代码演示如何使用 Python SDK 调用后触发器：

```python
container_link = 'dbs/myDatabase/colls/myContainer'
item = { 'name': 'artist_profile_1023', 'artist': 'The Band', 'albums': ['Hellujah', 'Rotators', 'Spinning Top']}
client.CreateItem(container_link, item, { 'postTriggerInclude': 'trgPostUpdateMetadata'})
```

## <a id="udfs"></a>如何使用用户定义的函数

以下示例演示如何使用 Azure Cosmos DB SDK 注册用户定义的函数。 请参阅[用户定义的函数示例](how-to-write-stored-procedures-triggers-udfs.md#udfs)，因为此后触发器的源代码保存为 `udfTax.js`。

### <a name="user-defined-functions---net-sdk"></a>用户定义的函数 - .NET SDK

以下代码演示如何使用 .NET SDK 注册用户定义的函数：

```csharp
string udfId = "Tax";
var udfTax = new UserDefinedFunction
{
    Id = udfId,
    Body = File.ReadAllText($@"..\js\{udfId}.js"),
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateUserDefinedFunctionAsync(containerUri, udfTax);

```

以下代码演示如何使用 .NET SDK 调用用户定义的函数：

```csharp
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var results = client.CreateDocumentQuery<dynamic>(containerUri, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000"));

foreach (var result in results)
{
    //iterate over results
}
```

### <a name="user-defined-functions---java-sdk"></a>用户定义的函数 - Java SDK

以下代码演示如何使用 Java SDK 注册用户定义的函数：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String udfId = "Tax";
UserDefinedFunction udf = new UserDefinedFunction();
udf.setId(udfId);
udf.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", udfId)))));
//toBlocking() blocks the thread until the operation is complete and is used only for demo.
UserDefinedFunction createdUDF = client.createUserDefinedFunction(containerLink, udf, new RequestOptions()).toBlocking().single().getResource();
```

以下代码演示如何使用 Java SDK 调用用户定义的函数：

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Observable<FeedResponse<Document>> queryObservable = client.queryDocuments(containerLink, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000", new FeedOptions());
final CountDownLatch completionLatch = new CountDownLatch(1);
queryObservable.subscribe(
        queryResultPage -> {
            System.out.println("Got a page of query result with " +
                    queryResultPage.getResults().size());
        },
        // terminal error signal
        e -> {
            e.printStackTrace();
            completionLatch.countDown();
        },

        // terminal completion signal
        () -> {
            completionLatch.countDown();
        });
completionLatch.await();
```

### <a name="user-defined-functions---javascript-sdk"></a>用户定义的函数 - JavaScript SDK

以下代码演示如何使用 JavaScript SDK 注册用户定义的函数：

```javascript
const container = client.database("myDatabase").container("myContainer");
const udfId = "Tax";
await container.userDefinedFunctions.create({
    id: udfId,
    body: require(`../js/${udfId}`)
```

以下代码演示如何使用 JavaScript SDK 调用用户定义的函数：

```javascript
const container = client.database("myDatabase").container("myContainer");
const sql = "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000";
const {result} = await container.items.query(sql).toArray();
```

### <a name="user-defined-functions---python-sdk"></a>用户定义的函数 - Python SDK

以下代码演示如何使用 Python SDK 注册用户定义的函数：

```python
with open('../js/udfTax.js') as file:
    file_contents = file.read()
container_link = 'dbs/myDatabase/colls/myContainer'
udf_definition = {
            'id': 'Tax',
            'serverScript': file_contents,
        }
udf = client.CreateUserDefinedFunction(container_link, udf_definition)
```

以下代码演示如何使用 Python SDK 调用用户定义的函数：

```python
container_link = 'dbs/myDatabase/colls/myContainer'
results = list(client.QueryItems(container_link, 'SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000'))
```

## <a name="next-steps"></a>后续步骤

详细了解概念以及如何在 Azure Cosmos DB 中编写或使用存储过程、触发器和用户定义的函数：

- [在 Azure Cosmos DB 中使用 Azure Cosmos DB 存储过程、触发器与用户定义的函数](stored-procedures-triggers-udfs.md)
- [在 Azure Cosmos DB 中使用 JavaScript 语言集成式查询 API](javascript-query-api.md)
- [如何在 Azure Cosmos DB 中编写存储过程、触发器和用户定义的函数](how-to-write-stored-procedures-triggers-udfs.md)
- [如何在 Azure Cosmos DB 中使用 Javascript 查询 API 编写存储过程和触发器](how-to-write-javascript-query-api.md)
