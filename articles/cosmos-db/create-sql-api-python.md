---
title: Azure Cosmos DB：使用 Python 和 SQL API 生成应用
description: 演示一个可以用来连接到 Azure Cosmos DB SQL API 并进行查询的 Python 代码示例
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: sngun
ms.openlocfilehash: 91a5bf71983511f58fc13a0edf4a1109c3829602
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410680"
---
# <a name="azure-cosmos-db-build-a-python-application-using-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB：使用 Azure Cosmos DB SQL API 帐户生成 Python 应用程序

> [!div class="op_single_selector"]
> * [.NET](create-sql-api-dotnet.md)
> * [.NET（预览版）](create-sql-api-dotnet-preview.md)
> * [Java](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB 是 Microsoft 提供的全球分布式多模型数据库服务。 可以快速创建和查询文档、键/值和图形数据库。 所有这些操作受益于 Azure Cosmos DB 的分布和规模。

本快速入门演示如何使用 Azure 门户创建 Azure Cosmos DB [SQL API](sql-api-introduction.md) 帐户、文档数据库和容器， 然后使用 [SQL API](sql-api-sdk-python.md) 的 Python SDK 构建并运行控制台应用。 本快速入门使用 3.0 版的 [Python SDK](https://pypi.org/project/azure-cosmos)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

## <a name="prerequisites"></a>先决条件

* [Python 3.6](https://www.python.org/downloads/)，以及在 `PATH` 中可用的 `python` 可执行文件。
* [Visual Studio Code](https://code.visualstudio.com/)
* [适用于 Visual Studio Code 的 Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview)

## <a name="create-a-database-account"></a>创建数据库帐户

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-collection"></a>添加集合

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

## <a name="add-sample-data"></a>添加示例数据

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>查询数据

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>克隆示例应用程序

现在，让我们从 GitHub 中克隆一个 SQL API 应用，设置连接字符串，然后运行该应用。

1. 打开命令提示符，新建一个名为“git-samples”的文件夹，然后关闭命令提示符。

    ```bash
    md "C:\git-samples"
    ```

2. 打开诸如 git bash 之类的 git 终端窗口，并使用 `cd` 命令更改为要安装示例应用的新文件夹。

    ```bash
    cd "C:\git-samples"
    ```

3. 运行下列命令以克隆示例存储库。 此命令在计算机上创建示例应用程序的副本。 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-python-getting-started.git
    ```  

## <a name="update-your-connection-string"></a>更新连接字符串

现在返回到 Azure 门户，获取连接字符串信息，并将其复制到应用。

1. 在 [Azure 门户](https://portal.azure.com/)中，在你的 Azure Cosmos DB 帐户中，单击左侧导航栏中的“密钥”。 在下一步骤中你将使用屏幕右侧的复制按钮将 **URI** 和**主密钥**复制到 `CosmosGetStarted.py` 文件中。

    ![在 Azure 门户的“密钥”边栏选项卡中查看并复制访问密钥](./media/create-sql-api-dotnet/keys.png)

2. 在 Visual Studio Code 中打开 C:\git-samples\azure-cosmos-db-python-getting-started 中的 `CosmosGetStarted.py` 文件。

3. 从门户中复制 **URI** 值（使用复制按钮），并在 ``CosmosGetStarted.py`` 中将其设为**终结点**密钥的值。 

    `'ENDPOINT': 'https://FILLME.documents.azure.com',`

4. 然后从门户中复制“主密钥”值，并在 ``CosmosGetStarted.py`` 中将其设为 **config.PRIMARYKEY** 的值。 现已使用与 Azure Cosmos DB 进行通信所需的所有信息更新应用。 

    `'PRIMARYKEY': 'FILLME',`

5. 保存 ``CosmosGetStarted.py`` 文件。

## <a name="review-the-code"></a>查看代码

此步骤是可选的。 了解在代码中创建的数据库资源，或者跳转到[更新连接字符串](#update-your-connection-string)。

注意，如果你熟悉旧版 Python SDK，则可能已看惯了术语“集合”和“文档”。 由于 Azure Cosmos DB 支持多 API 模型，因此 3.0+ 版的 Python SDK 使用通用术语“容器”（可能是集合、图形或表），并使用“项”来描述容器的内容。

以下代码片段全部摘自 `CosmosGetStarted.py` 文件。

* 对 CosmosClient 进行初始化。 请确保更新“Endpoint”和“masterkey”值，如[更新连接字符串](#update-your-connection-string)部分中所述。 

    ```python
    # Initialize the Cosmos client
    client = cosmos_client.CosmosClient(url_connection=config['ENDPOINT'], auth={'masterKey': config['MASTERKEY']})
    ```

* 将创建一个新数据库。

    ```python
    # Create a database
    db = client.CreateDatabase({ 'id': config['DATABASE'] })
    ```

* 将创建一个新集合。

    ```python
    # Create collection options
    options = {
        'offerThroughput': 400
    }

    # Create a container
    container = client.CreateContainer(db['_self'], container_definition, options)
    ```

* 向容器中添加一些项。

    ```python
    # Create and add some items to the container
    item1 = client.CreateItem(container['_self'], {
        'serverId': 'server1',
        'Web Site': 0,
        'Cloud Service': 0,
        'Virtual Machine': 0,
        'message': 'Hello World from Server 1!'
        }
    )

    item2 = client.CreateItem(container['_self'], {
        'serverId': 'server2',
        'Web Site': 1,
        'Cloud Service': 0,
        'Virtual Machine': 0,
        'message': 'Hello World from Server 2!'
        }
    )
    ```

* 将使用 SQL 执行查询

    ```python
    query = {'query': 'SELECT * FROM server s'}

    options = {}
    options['enableCrossPartitionQuery'] = True
    options['maxItemCount'] = 2

    result_iterable = client.QueryItems(container['_self'], query, options)
    for item in iter(result_iterable):
        print(item['message'])
    ```
   
## <a name="run-the-app"></a>运行应用

1. 在 Visual Studio Code 中，选择“视图”>“命令面板”。 

2. 在提示符处，输入 Python:Select Interpreter，然后选择要使用的 Python 的版本。

    Visual Studio Code 中的页脚将更新以指示所选的解释器。 

3. 选择“视图” > “集成终端”以打开 Visual Studio Code 集成终端。

4. 在集成的终端窗口中，确保位于 azure-cosmos-db-python-getting-started 文件夹中。 如果没有位于该文件夹中，请运行以下命令来切换到示例文件夹。 

    ```
    cd "C:\git-samples\azure-cosmos-db-python-getting-started"`
    ```

5. 运行以下命令来安装 azure-cosmos 程序包。 

    ```
    pip3 install azure-cosmos
    ```

    如果尝试安装 azure-cosmos 时收到有关访问被拒绝的错误，则需要[以管理员身份运行 VS Code](https://stackoverflow.com/questions/37700536/visual-studio-code-terminal-how-to-run-a-command-with-administrator-rights)。

6. 运行以下命令来运行示例并将新文档存储在 Azure Cosmos dB 中。

    ```
    python CosmosGetStarted.py
    ```

7. 若要确认是否已创建并保存了新项，请在 Azure 门户中选择“数据资源管理器”，展开“列”，展开“文档”，然后选择 **server1** 文档。 server1 文档内容与在集成的终端窗口中返回的内容相匹配。 

    ![在 Azure 门户中查看新文档](./media/create-sql-api-python/azure-cosmos-db-confirm-documents.png)

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 门户中查看 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门教程中，已了解如何创建 Azure Cosmos DB 帐户、使用数据资源管理器创建集合和运行应用。 现在可以将其他数据导入 Cosmos DB 帐户。 

> [!div class="nextstepaction"]
> [将 SQL API 的数据导入 Azure Cosmos DB](import-data.md)


