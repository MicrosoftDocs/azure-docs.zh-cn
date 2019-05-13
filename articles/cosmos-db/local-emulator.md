---
title: 使用 Azure Cosmos 模拟器在本地开发
description: 利用 Azure Cosmos 模拟器，无需创建 Azure 订阅即可在本地免费开发和测试应用程序。
ms.service: cosmos-db
ms.topic: tutorial
author: deborahc
ms.author: dech
ms.date: 03/14/2019
ms.openlocfilehash: c83cc8dce5978798d86d2fc2e314161765a2fb2d
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205790"
---
# <a name="use-the-azure-cosmos-emulator-for-local-development-and-testing"></a>使用 Azure Cosmos 模拟器进行本地开发和测试

为方便开发，Azure Cosmos 模拟器提供了一个模拟 Azure Cosmos DB 服务的本地环境。 使用 Azure Cosmos 模拟器可在本地开发和测试应用程序，无需创建 Azure 订阅且不会产生任何费用。 如果对应用程序在 Azure Cosmos 拟器中的工作情况感到满意，可以转为在云中使用 Azure Cosmos 帐户。

可使用 Azure Cosmos 模拟器结合 [SQL](local-emulator.md#sql-api)、[Cassandra](local-emulator.md#cassandra-api)、[MongoDB](local-emulator.md#azure-cosmos-dbs-api-for-mongodb)、[Gremlin](local-emulator.md#gremlin-api) 和[表](local-emulator.md#table-api) API 帐户进行开发。 但是，该模拟器中的数据资源管理器视图目前仅完全支持 SQL API 的客户端。 

## <a name="how-the-emulator-works"></a>模拟器的工作原理

Azure Cosmos 模拟器提供对 Azure Cosmos DB 服务的高保真模拟。 它支持与 Azure Cosmos DB 相同的功能，包括创建和查询数据、支持预配和缩放容器，以及执行存储过程和触发器。 可使用 Azure Cosmos 模拟器来开发和测试应用程序，并通过对 Azure Cosmos DB 的连接终结点进行单一配置更改，将其部署到全局范围的 Azure。

虽然模拟 Azure Cosmos DB 服务很逼真，但模拟器的实现不同于服务。 例如，模拟器使用标准 OS 组件，例如用于暂留的本地文件系统和用于连接的 HTTPS 协议堆栈。 不可使用依赖于 Azure 基础结构的功能，比如全局复制、读/写的单位数毫秒延迟，以及可调整的一致性级别。

可通过 [Azure Cosmos DB 数据迁移工具](https://github.com/azure/azure-documentdb-datamigrationtool)在 Azure Cosmos 模拟器与 Azure Cosmos DB 服务之间迁移数据。

可在 Windows Docker 容器上运行 Azure Cosmos 模拟器；详见 [Docker Hub](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/) 获取 Docker 拉取命令，详见 [GitHub](https://github.com/Azure/azure-cosmos-db-emulator-docker) 获取模拟器源代码。

## <a name="differences-between-the-emulator-and-the-service"></a>模拟器和服务之间的差异

由于 Azure Cosmos 模拟器提供一个在本地开发人员工作站上运行的模拟环境，因此该模拟器在功能上与 Azure Cosmos 云端帐户之间存在一些差异：

* 目前，模拟器中的数据资源管理器支持 SQL API 的客户端。 不完全支持在数据资源管理器查看和操作 Azure Cosmos DB API，例如 MongoDB API、表 API、Graph API 和 Cassandra API。
* Azure Cosmos 模拟器只支持单一固定帐户和公开的主密钥。 不可在 Azure Cosmos 模拟器中重新生成密钥，但可使用命令行选项更改默认密钥。
* Azure Cosmos 模拟器是一项不可伸缩的服务，不支持大量容器。
* Azure Cosmos 模拟器只提供一种 [Azure Cosmos DB 一致性级别](consistency-levels.md)。
* Azure Cosmos 模拟器不提供[多区域复制](distribute-data-globally.md)。
* 由于 Azure Cosmos 模拟器副本并不总是能反映出 Azure Cosmos DB 服务中的最新更改，因此应使用 [Azure Cosmos DB 容量规划器](https://www.documentdb.com/capacityplanner)来准确估计应用程序的生产吞吐量 (RU) 需求。
* 默认使用 Azure Cosmos 模拟器时，可最多创建 25 个固定大小的容器（仅支持使用 Azure Cosmos DB SDK 创建），或使用 Azure Cosmos 模拟器创建 5 个不受限容器。 有关如何更改此值的详细信息，请参阅[设置 PartitionCount 值](#set-partitioncount)。

## <a name="system-requirements"></a>系统要求

Azure Cosmos 模拟器具有以下硬件和软件要求：

* 软件要求
  * Windows Server 2012 R2、Windows Server 2016 或 Windows 10
  * 64 位操作系统
* 最低硬件要求
  * 2-GB RAM
  * 10-GB 可用硬盘空间

## <a name="installation"></a>安装

可以从 [Microsoft 下载中心](https://aka.ms/cosmosdb-emulator)下载并安装 Azure Cosmos 模拟器，也可以在用于 Windows 的 Docker 上运行模拟器。 有关在用于 Windows 的 Docker 上使用模拟器的说明，请参阅[在 Docker 上运行](#running-on-docker)。

> [!NOTE]
> 若要安装、配置和运行 Azure Cosmos 模拟器，必须在计算机上具有管理特权。 该模拟器将创建/添加证书并设置防火墙规则，从而运行其服务；因此，模拟器必须能够执行此类操作。

## <a name="running-on-windows"></a>在 Windows 上运行

要启动 Azure Cosmos 模拟器，请选择“开始”按钮或按 Windows 键。 开始键入“Azure Cosmos 模拟器”，再从应用程序列表中选择该模拟器。

![选择“开始”按钮或按 Windows 键，开始键入**Azure Cosmos DB 模拟器**，再从应用程序列表中选择该模拟器](./media/local-emulator/database-local-emulator-start.png)

运行模拟器时，在 Windows 任务栏通知区域中会显示一个图标。 ![Azure Cosmos DB 本地模拟器任务栏通知](./media/local-emulator/database-local-emulator-taskbar.png)

默认情况下，Azure Cosmos 模拟器在本地计算机（“localhost”）上运行，侦听端口 8081。

Azure Cosmos 模拟器默认安装到 `C:\Program Files\Azure Cosmos DB Emulator`。 还可以从命令行启动和停止该模拟器。 有关详细信息，请参阅[命令行工具参考](#command-line)。

## <a name="start-data-explorer"></a>启动数据资源管理器

Azure Cosmos 模拟器启动时，会在浏览器中自动打开 Azure Cosmos 数据资源管理器。 地址显示为 `https://localhost:8081/_explorer/index.html`。 如果关闭了资源管理器且稍后想要重新打开它，可在浏览器中打开 URL，或者通过 Windows 托盘图标中的 Azure Cosmos 模拟器进行启动，如下所示。

![Azure Cosmos 本地模拟器数据资源管理器启动器](./media/local-emulator/database-local-emulator-data-explorer-launcher.png)

## <a name="checking-for-updates"></a>检查更新

数据资源管理器指示是否有新的更新可供下载。

> [!NOTE]
> 在某版本的 Azure Cosmos 模拟器中创建数据后（参见 %LOCALAPPDATA%\CosmosDBEmula 或数据路径可选设置），不保证在使用其他版本时可访问。 如果需要长期保存数据，建议将该数据存储在 Azure Cosmos 帐户中，而不是存储在 Azure Cosmos 模拟器中。

## <a name="authenticating-requests"></a>对请求进行身份验证

与云中的 Azure Cosmos DB 一样，针对 Azure Cosmos 模拟器发出的每个请求都必须进行身份验证。 Azure Cosmos 模拟器支持单一固定帐户和用于主密钥身份验证的公开的身份验证密钥。 此帐户和密钥是允许用于 Azure Cosmos 模拟器的唯一凭据。 它们是：

```bash
Account name: localhost:<port>
Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
```

> [!NOTE]
> Azure Cosmos 模拟器支持的主密钥仅可用于该模拟器。 不能在 Azure Cosmos DB 模拟器中使用生产 Azure Cosmos DB 帐户和密钥。

> [!NOTE]
> 如果是使用 /Key 选项启动的模拟器，请使用所生成的密钥而不是 `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`。 有关 /Key 选项的详细信息，请参阅[命令行工具参考](#command-line-syntax)。

与 Azure Cosmos DB 一样，Azure Cosmos 模拟器仅支持采用 SSL 的安全通信。

## <a name="running-on-a-local-network"></a>在本地网络上运行

可在本地网络中运行仿真器。 要启用网络访问，请在[命令行](#command-line-syntax)中指定 `/AllowNetworkAccess` 选项，同时还需指定 `/Key=key_string` 或 `/KeyFile=file_name`。 可使用 `/GenKeyFile=file_name` 提前生成具有随机密钥的文件。 然后，可将其传递到 `/KeyFile=file_name` 或 `/Key=contents_of_file`。

首次启用网络访问时，用户应关闭模拟器，并删除模拟器的数据目录 (%LOCALAPPDATA%\CosmosDBEmulator)。

## <a name="developing-with-the-emulator"></a>通过模拟器进行开发

### <a name="sql-api"></a>SQL API

在桌面上运行 Azure Cosmos 模拟器后，可使用任意受支持的 [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) 或 [Azure Cosmos DB REST API](/rest/api/cosmos-db/) 与模拟器进行交互。 Azure Cosmos 模拟器还包括内置数据资源管理器，它让你无需编写任何代码即可为 SQL API 创建容器（或为 Mongo DB API 创建 Cosmos DB）并查看和编辑项目。

```csharp
// Connect to the Azure Cosmos Emulator running locally
DocumentClient client = new DocumentClient(
   new Uri("https://localhost:8081"), "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

```

### <a name="azure-cosmos-dbs-api-for-mongodb"></a>Azure Cosmos DB 的用于 MongoDB 的 API

如果使用 [Azure Cosmos DB 的 API for MongoDB](mongodb-introduction.md)，请使用以下连接字符串：

```bash
mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true
```

### <a name="table-api"></a>表 API

在桌面上运行 Azure Cosmos 模拟器后，可使用任意受支持的 [Azure Cosmos DB 表 API SDK](table-storage-how-to-use-dotnet.md) 与模拟器进行交互。 在命令提示符处，以管理员的身份使用“/EnableTableEndpoint”启动模拟器。 接下来，运行下列代码以连接到表 API 帐户：

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Table;
using CloudTable = Microsoft.WindowsAzure.Storage.Table.CloudTable;
using CloudTableClient = Microsoft.WindowsAzure.Storage.Table.CloudTableClient;

string connectionString = "DefaultEndpointsProtocol=http;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=http://localhost:8902/;";

CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("testtable");
table.CreateIfNotExists();
table.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
```

### <a name="cassandra-api"></a>Cassandra API

使用“/EnableCassandraEndpoint”在管理员命令提示符处启动模拟器。 或者，也可设置环境变量 `AZURE_COSMOS_EMULATOR_CASSANDRA_ENDPOINT=true`。

* [安装 Python 2.7](https://www.python.org/downloads/release/python-2716/)

* [安装 Cassandra CLI/CQLSH](https://cassandra.apache.org/download/)

* 在常规命令提示符窗口中运行以下命令：

  ```bash
  set Path=c:\Python27;%Path%
  cd /d C:\sdk\apache-cassandra-3.11.3\bin
  set SSL_VERSION=TLSv1_2
  set SSL_VALIDATE=false
  cqlsh localhost 10350 -u localhost -p C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== --ssl
  ```

* 在 CQLSH shell 中，运行下列命令以连接到 Cassandra 终结点：

  ```bash
  CREATE KEYSPACE MyKeySpace WITH replication = {'class':'MyClass', 'replication_factor': 1};
  DESCRIBE keyspaces;
  USE mykeyspace;
  CREATE table table1(my_id int PRIMARY KEY, my_name text, my_desc text);
  INSERT into table1 (my_id, my_name, my_desc) values( 1, 'name1', 'description 1');
  SELECT * from table1;
  EXIT
  ```

### <a name="gremlin-api"></a>Gremlin API

使用“/EnableGremlinEndpoint”在管理员命令提示符处启动模拟器。 或者，也可设置环境变量 `AZURE_COSMOS_EMULATOR_GREMLIN_ENDPOINT=true`

* [安装 apache-tinkerpop-gremlin-console-3.3.4](https://tinkerpop.apache.org/downloads.html)

* 在模拟器的数据资源管理器中，创建数据库“db1”和集合“coll1”，并选择“/name”作为分区键

* 在常规命令提示符窗口中运行以下命令：

  ```bash
  cd /d C:\sdk\apache-tinkerpop-gremlin-console-3.3.4-bin\apache-tinkerpop-gremlin-console-3.3.4
  
  copy /y conf\remote.yaml conf\remote-localcompute.yaml
  notepad.exe conf\remote-localcompute.yaml
    hosts: [localhost]
    port: 8901
    username: /dbs/db1/colls/coll1
    password: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
    connectionPool: {
    enableSsl: false}
    serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
    config: { serializeResultToString: true  }}

  bin\gremlin.bat
  ```

* 在 Gremlin shell 中，运行下列命令以连接到 Gremlin 终结点：

  ```bash
  :remote connect tinkerpop.server conf/remote-localcompute.yaml
  :remote console
  :> g.V()
  :> g.addV('person1').property(id, '1').property('name', 'somename1')
  :> g.addV('person2').property(id, '2').property('name', 'somename2')
  :> g.V()
  ```

## <a name="export-the-ssl-certificate"></a>导出 SSL 证书

.NET 语言和运行时使用 Windows 证书存储来安全地连接到 Azure Cosmos DB 本地模拟器。 其他语言有自己管理和使用证书方法。 Java 使用自己的[证书存储](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html)，而 Python 使用[套接字包装器](https://docs.python.org/2/library/ssl.html)。

为了获得用于语言和运行时（未与 Windows 证书存储集成）的证书，需要通过 Windows 证书管理器将其导出。 可通过运行 certlm.msc 进行启动，也可按照[导出 Azure Cosmos 模拟器证书](./local-emulator-export-ssl-certificates.md)中的分步说明进行操作。 证书管理器开始运行后，打开个人证书（如下所示），并将友好名称为“DocumentDBEmulatorCertificate”的证书导出为 BASE-64 编码的 X.509 (.cer) 文件。

![Azure Cosmos DB 本地模拟器 SSL 证书](./media/local-emulator/database-local-emulator-ssl_certificate.png)

可按照[将证书添加到 Java CA 证书存储](https://docs.microsoft.com/azure/java-add-certificate-ca-store)中的说明，将 X.509 证书导入 Java 证书存储。 证书导入证书存储后，SQL 和 MongoDB 的 Azure Cosmos DB API 的客户端就能连接到 Azure Cosmos 模拟器。

从 Python 和 Node.js SDK 连接到模拟器时，将禁用 SSL 验证。

## <a id="command-line"></a>命令行工具参考
从安装位置中，可以使用命令行启动和停止模拟器、配置选项和执行其他操作。

### <a name="command-line-syntax"></a>命令行语法

    CosmosDB.Emulator.exe [/Shutdown] [/DataPath] [/Port] [/MongoPort] [/DirectPorts] [/Key] [/EnableRateLimiting] [/DisableRateLimiting] [/NoUI] [/NoExplorer] [/?]

若要查看选项列表，请在命令提示符下键入 `CosmosDB.Emulator.exe /?`。

|**选项** | **说明** | **命令**| **参数**|
|---|---|---|---|
|[无参数] | 使用默认设置启动 Azure Cosmos 模拟器。 |CosmosDB.Emulator.exe| |
|[帮助] |显示支持的命令行参数列表。|CosmosDB.Emulator.exe /? | |
| GetStatus |获取 Azure Cosmos 模拟器的状态。 状态由退出代码指示：1 = 正在启动，2 = 正在运行，3 = 已停止。 退出代码为负表示发生了错误。 不生成其他输出。 | CosmosDB.Emulator.exe /GetStatus| |
| 关机| 关闭 Azure Cosmos 模拟器。| CosmosDB.Emulator.exe /Shutdown | |
|DataPath | 指定要在其中存储数据文件的路径。 默认值为 %LocalAppdata%\CosmosDBEmulator。 | CosmosDB.Emulator.exe /DataPath=\<datapath\> | \<datapath\>：可访问路径 |
|端口 | 指定用于模拟器的端口号。 默认值为 8081。 |CosmosDB.Emulator.exe /Port=\<port\> | \<port\>：单个端口号 |
| MongoPort | 指定用于 MongoDB 兼容性 API 的端口号。 默认值为 10255。 |CosmosDB.Emulator.exe /MongoPort= \<mongoport\>|\<mongoport\>：单个端口号|
| CassandraPort | 指定用于 Cassandra 终结点的端口号。 默认值为 10350。 | CosmosDB.Emulator.exe /CassandraPort = \<cassandraport\> | \<cassandraport\>：单个端口号 |
| ComputePort | 指定用于计算互操作网关服务的端口号。 该网关的 HTTP 终结点探测端口计算得出 ComputePort + 79。 因此，ComputePort 和 ComputePort + 79 必须打开且可使用。 默认值为 8900, 8979。 | CosmosDB.Emulator.exe /ComputePort = \<computeport\> | \<computeport\>：单个端口号 |
| EnableCassandraEndpoint | 启用 Cassandra API | CosmosDB.Emulator.exe /EnableCassandraEndpoint | |
| EnableGremlinEndpoint | 启用 Gremlin API | CosmosDB.Emulator.exe /EnableGremlinEndpoint | |
| GremlinPort | 用于 Gremlin 终结点的端口号。 默认值为 8901。 | CosmosDB.Emulator.exe /GremlinPort=\<port\> | \<port\>：单个端口号 |
|TablePort | 用于 Azure 表终结点的端口号。 默认值为 8902。 | CosmosDB.Emulator.exe /TablePort=\<port\> | \<port\>：单个端口号|
| KeyFile | 从指定文件中读取授权密钥。 使用 /GenKeyFile 选项来生成密钥文件 | CosmosDB.Emulator.exe /KeyFile=\<file_name\> | \<file_name\>：文件的路径 |
| ResetDataPath | 以递归方式删除指定路径中的所有文件。 如果不指定路径，则默认为 %LOCALAPPDATA%\CosmosDbEmulator | CosmosDB.Emulator.exe /ResetDataPath[=\<path>] | \<path\>：文件路径  |
| StartTraces  |  开始收集调试跟踪日志。 | CosmosDB.Emulator.exe /StartTraces | |
| StopTraces     | 停止收集调试跟踪日志。 | CosmosDB.Emulator.exe /StopTraces  | |
|EnableTableEndpoint | 启用 Azure 表 API | CosmosDB.Emulator.exe /EnableTableEndpoint | |
|FailOnSslCertificateNameMismatch | 默认情况下，如果证书的 SAN 不包含模拟器主机的域名、本机 IPv4 地址、“localhost”和“127.0.0.1”，则模拟器会重新生成自签名的 SSL 证书。 启用此选项后，模拟器在启动时会失败。 之后，你应使用 /GenCert 选项来创建和安装新的自签名 SSL 证书。 | CosmosDB.Emulator.exe /FailOnSslCertificateNameMismatch  | |
| GenCert | 生成并安装新的自签名 SSL 证书。 选择性地包含用于通过网络访问模拟器的其他 DNS 名称的列表（以逗号分隔）。 | CosmosDB.Emulator.exe /GenCert[ \<comma-separated list of additional dns-names\>] | |
| DirectPorts |指定用于直接连接的端口。 默认值为 10251、10252、10253、10254。 | CosmosDB.Emulator.exe /DirectPorts:\<directports\> | \<directports\>：以逗号分隔的 4 个端口的列表 |
| 密钥 |模拟器的授权密钥。 密钥必须是 64 字节向量的 base 64 编码。 | CosmosDB.Emulator.exe /Key:\<key\> | \<key\>：密钥必须是 64 字节向量的 base 64 编码|
| EnableRateLimiting | 指定已启用请求速率限制行为。 |CosmosDB.Emulator.exe /EnableRateLimiting | |
| DisableRateLimiting |指定已禁用请求速率限制行为。 |CosmosDB.Emulator.exe /DisableRateLimiting | |
| NoUI | 不显示模拟器用户界面。 | CosmosDB.Emulator.exe /NoUI | |
| NoExplorer | 在启动时不显示数据资源管理器。 |CosmosDB.Emulator.exe /NoExplorer | | 
| PartitionCount | 指定已分区的容器的最大数。 有关详细信息，请参阅[更改容器数量](#set-partitioncount)。 | CosmosDB.Emulator.exe /PartitionCount=\<partitioncount\> | \<partitioncount\>：允许的单分区容器的最大数量。 默认值为 25。 允许的最大值为 250。|
| DefaultPartitionCount| 指定分区容器的默认分区数。 | CosmosDB.Emulator.exe /DefaultPartitionCount=\<defaultpartitioncount\> | \<defaultpartitioncount\> 默认值为 25。|
| AllowNetworkAccess | 通过网络启用对仿真器的访问。 要启用网络访问，还必须传递 /Key=\<key_string\> 或 /KeyFile=\<file_name\>。 | CosmosDB.Emulator.exe /AllowNetworkAccess /Key=\<key_string\> 或 CosmosDB.Emulator.exe /AllowNetworkAccess /KeyFile=\<file_name\>| |
| NoFirewall | 使用 /AllowNetworkAccess 选项时，不要调整防火墙规则。 |CosmosDB.Emulator.exe /NoFirewall | |
| GenKeyFile | 生成新的授权密钥并保存至指定文件。 所生成的密钥可与 /Key 或 /KeyFile 选项配合使用。 | CosmosDB.Emulator.exe /GenKeyFile=\<密钥文件路径\> | |
| 一致性 | 为帐户设置默认一致性级别。 | CosmosDB.Emulator.exe /Consistency=\<consistency\> | \<consistency\>：值必须是以下[一致性级别](consistency-levels.md)之一：Session、Strong、Eventual 或 BoundedStaleness。 默认值为“Session”。 |
| ? | 显示帮助消息。| | |

## <a id="set-partitioncount"></a>更改容器数量

默认情况下，可最多创建 25 个固定大小的容器（仅支持使用 Azure Cosmos DB SDK 创建），或使用 Azure Cosmos 模拟器创建 5 个不受限容器。 通过修改 PartitionCount 值，可最多创建 250 个固定大小的容器或 50 个不受限容器，也可创建两者的任意组合（前提是总数不超过 250 个固定大小的容器，其中 1 个不受限容器 = 5 个固定大小的容器）。 但是，建议不要设置用 200 个以上固定大小的容器进行运行的模拟器。 因为这会造成磁盘 IO 操作的开销增加，导致在运行终结点 API 时出现不可预测的超时情况。


如果在已超过当前分区计数后尝试创建容器，则模拟器将引发 ServiceUnavailable 异常，并收到以下消息。

“很抱歉，此区域当前需求较高，暂时无法满足你的请求。 我们在持续努力推出越来越多的容量，请进行重试。
随时欢迎出于任何原因发送电子邮件至 askcosmosdb@microsoft.com。 ActivityId:12345678-1234-1234-1234-123456789abc”

要更改 Azure Cosmos 模拟器中可用容器的数量，请执行以下步骤：

1. 在系统任务栏上右键单击“Azure Cosmos 模拟器”图标，并单击“重置数据...”，删除所有本地 Azure Cosmos 模拟器数据。
2. 删除文件夹 `%LOCALAPPDATA%\CosmosDBEmulator` 中的所有模拟器数据。
3. 通过在系统任务栏上右键单击“Azure Cosmos DB 模拟器”图标，并单击“退出”，退出所有打开的实例。 退出所有实例可能需要一分钟。
4. 安装最新版的 [Azure Cosmos 模拟器](https://aka.ms/cosmosdb-emulator)。
5. 通过设置一个 <= 250 的值启动具有 PartitionCount 标志的模拟器。 例如：`C:\Program Files\Azure Cosmos DB Emulator> CosmosDB.Emulator.exe /PartitionCount=100`。

## <a name="controlling-the-emulator"></a>控制模拟器

该模拟器附带一个 PowerShell 模块，它可用于启动、停止、卸载和检索服务的状态。 运行下列 cmdlet 以使用 PowerShell 模块：

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
```

或者，将 `PSModules` 目录放置在 `PSModulesPath` 上，并按如下命令中所示将其导入：

```powershell
$env:PSModulesPath += "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules"
Import-Module Microsoft.Azure.CosmosDB.Emulator
```

下面汇总的命令用于通过 PowerShell 来控制模拟器：

### `Get-CosmosDbEmulatorStatus`

**语法**

`Get-CosmosDbEmulatorStatus`

**备注**

返回以下 ServiceControllerStatus 值之一：ServiceControllerStatus.StartPending、ServiceControllerStatus.Running 或 ServiceControllerStatus.Stopped。

### `Start-CosmosDbEmulator`

**语法**

`Start-CosmosDbEmulator [-DataPath <string>] [-DefaultPartitionCount <uint16>] [-DirectPort <uint16[]>] [-MongoPort <uint16>] [-NoUI] [-NoWait] [-PartitionCount <uint16>] [-Port <uint16>] [<CommonParameters>]`

**备注**

启动模拟器。 默认情况下，此命令会一直等待，直至模拟器做好接受请求的准备。 如果希望 cmdlet 在启动模拟器后立即返回，请使用 -NoWait 选项。

### `Stop-CosmosDbEmulator`

**语法**

 `Stop-CosmosDbEmulator [-NoWait]`

**备注**

停止模拟器。 默认情况下，此命令会一直等待，直至模拟器完全关闭。 如果希望 cmdlet 在模拟器开始关闭后立即返回，请使用 -NoWait 选项。

### `Uninstall-CosmosDbEmulator`

**语法**

`Uninstall-CosmosDbEmulator [-RemoveData]`

**备注**

卸载模拟器，并可视需要删除 $env:LOCALAPPDATA\CosmosDbEmulator 的完整内容。
此 cmdlet 可确保在卸载模拟器之前，模拟器已停止。

## <a name="running-on-docker"></a>在 Docker 上运行

可在用于 Windows 的 Docker 上运行 Azure Cosmos 模拟器。 该模拟器不适合于用于 Oracle Linux 的 Docker。

安装[用于 Windows 的 Docker](https://www.docker.com/docker-windows) 后，通过右键单击工具栏上的 Docker 图标并选择“切换到 Windows 容器”切换到 Windows 容器。

接下来，通过从你喜欢使用的 shell 运行以下命令，从 Docker 中心拉取模拟器映像。

```bash
docker pull microsoft/azure-cosmosdb-emulator
```
若要启动映像，请运行以下命令。

通过命令行：
```cmd

md %LOCALAPPDATA%\CosmosDBEmulator\bind-mount

docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%LOCALAPPDATA%\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 microsoft/azure-cosmosdb-emulator
```

通过 PowerShell：
```powershell

md $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount 2>null

docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=$env:LOCALAPPDATA\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 microsoft/azure-cosmosdb-emulator

```

响应类似于以下内容：

```
Starting emulator
Emulator Endpoint: https://172.20.229.193:8081/
Master Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
Exporting SSL Certificate
You can import the SSL certificate from an administrator command prompt on the host by running:
cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
powershell .\importcert.ps1
--------------------------------------------------------------------------------------------------
Starting interactive shell
```

现在，在客户端中使用来自响应的终结点和主密钥，并将 SSL 证书导入到主机中。 若要导入 SSL 证书，请从管理员命令提示符执行以下操作：

通过命令行：

```cmd
cd  %LOCALAPPDATA%\CosmosDBEmulator\bind-mount
powershell .\importcert.ps1
```

通过 PowerShell：
```powershell
cd $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount
.\importcert.ps1
```

在模拟器启动之后便关闭交互式 shell 会关闭模拟器的容器。

若要打开数据资源管理器，请在浏览器中导航到以下 URL。 上面所示的响应消息中提供了模拟器终结点。

    https://<emulator endpoint provided in response>/_explorer/index.html


## <a name="troubleshooting"></a>故障排除

使用以下提示来帮助解决使用 Azure Cosmos 模拟器时遇到的问题：

- 如果安装了新版本的模拟器并遇到错误，请务必重置数据。 可在系统任务栏上右键单击“Azure Cosmos 模拟器”图标，然后单击“重置数据...”来重置数据。 如果仍无法消除错误，可卸载该模拟器和所有旧版模拟器（如有），删除“C:\Program files\Azure Cosmos DB Emulator”目录，并卸载模拟器。 有关说明，请参阅[卸载本地模拟器](#uninstall)。

- 如果 Azure Cosmos 模拟器崩溃，请从“%LOCALAPPDATA%\CrashDumps”文件夹收集转储文件、将其压缩，再附加到电子邮件发送到 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com)。

- 如果 `Microsoft.Azure.Cosmos.ComputeServiceStartupEntryPoint.exe` 中出现崩溃，这可能表示性能计数器处于损坏状态。 通常，从管理员命令提示符处运行以下命令即可解决此问题：

  ```cmd
  lodctr /R
   ```

- 如果遇到连接问题，请[收集跟踪文件](#trace-files)、进行压缩并将其附加到电子邮件，发送至 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com)。

- 如果收到“服务不可用”消息，模拟器可能无法初始化网络堆栈。 请查看是否安装了 Pulse 安全客户端或 Juniper 网络客户端，因为其网络筛选器驱动程序可能会导致该问题。 卸载第三方网络筛选器驱动程序通常可修复此问题。 或者，使用 /DisableRIO 启动模拟器，这会将模拟器网络通信切换到常规 Winsock。 

- 在模拟器运行时，如果计算机进入了睡眠模式或运行了任何 OS 更新，则你可能会看到“服务当前不可用”消息。 请右键单击 Windows 通知托盘中显示的图标，再选择“重置数据”来重置模拟器的数据。

### <a id="trace-files"></a>收集跟踪文件

若要收集调试跟踪，请从管理命令提示符运行以下命令：

1. `cd /d "%ProgramFiles%\Azure Cosmos DB Emulator"`
2. `CosmosDB.Emulator.exe /shutdown`。 监视系统托盘，确保该程序已关闭，这可能需要几分钟时间。 还可仅单击 Azure Cosmos 模拟器用户界面中的“退出”。
3. `CosmosDB.Emulator.exe /starttraces`
4. `CosmosDB.Emulator.exe`
5. 再现问题。 如果数据资源管理器无法运行，只需等待几秒钟，待浏览器打开以捕获错误。
5. `CosmosDB.Emulator.exe /stoptraces`
6. 导航到 `%ProgramFiles%\Azure Cosmos DB Emulator`，查找 docdbemulator_000001.etl 文件。
7. 将 .etl 文件和重现步骤一起发送至 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com) 进行调试。

### <a id="uninstall"></a>卸载本地模拟器

1. 通过在系统任务栏上右键单击“Azure Cosmos 模拟器”图标，然后单击“退出”，退出所有打开的本地模拟器实例。 退出所有实例可能需要一分钟。
2. 在 Windows 搜索框中，键入“应用和功能”，然后单击“应用和功能(系统设置)”结果。
3. 在应用列表中，滚动到“Azure Cosmos DB 模拟器”并将其选中，单击“卸载”，然后确认并再次单击“卸载”。
4. 卸载应用后，导航到 `%LOCALAPPDATA%\CosmosDBEmulator` 并删除该文件夹。

## <a name="next-steps"></a>后续步骤

在本教程中，你已了解了如何使用本地模拟器进行免费的本地开发。 现在可以继续学习下一教程，了解如何导出模拟器 SSL 证书。

> [!div class="nextstepaction"]
> [导出 Azure Cosmos 模拟器证书](local-emulator-export-ssl-certificates.md)
