---
title: 诊断并解决问题时在 Azure Functions 中使用 Azure Cosmos DB 触发器
description: 常见的问题、 解决方法和诊断的步骤，当使用 Azure Functions 和 Azure Cosmos DB 触发器
author: ealsur
ms.service: cosmos-db
ms.date: 04/16/2019
ms.author: maquaran
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: e8f0b9c8bf1bfb846f13306f58bcb1721ed6b422
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510531"
---
# <a name="diagnose-and-troubleshoot-issues-when-using-azure-cosmos-db-trigger-in-azure-functions"></a>诊断并解决问题时在 Azure Functions 中使用 Azure Cosmos DB 触发器

这篇文章介绍常见的问题、 解决方法和诊断的步骤，使用时[Azure Cosmos DB 触发器](change-feed-functions.md)使用 Azure Functions。

## <a name="dependencies"></a>依赖项

依赖于扩展包的基本 Azure Functions 运行时通过 Azure Cosmos DB 触发器和绑定。 始终保持更新，这些包，因为它们可能包含修补程序和可能会解决任何潜在问题，可能会遇到的新功能：

* Azure Functions V2，请参阅[Microsoft.Azure.WebJobs.Extensions.CosmosDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB)。
* Azure Functions V1，请参阅[Microsoft.Azure.WebJobs.Extensions.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DocumentDB)。

本文将始终引用 Azure Functions V2 时提到运行时，除非显式指定。

## <a name="consume-the-azure-cosmos-db-sdk-independently"></a>独立地使用 Azure Cosmos DB SDK

扩展包的关键功能是 Azure Cosmos DB 触发器和绑定提供支持。 它还包括[Azure Cosmos DB.NET SDK](sql-api-sdk-dotnet-core.md)，这都很有用，如果你想要使用 Azure Cosmos DB 以编程方式而无需使用触发器和绑定交互。

如果想要使用 Azure Cosmos DB SDK，请确保不要添加到你的项目另一个 NuGet 包引用。 相反，**允许通过 Azure Functions 的扩展包解析的 SDK 引用**。 使用 Azure Cosmos DB SDK 独立于触发器和绑定

此外，如果要手动创建的实例[Azure Cosmos DB SDK 客户端](./sql-api-sdk-dotnet-core.md)，应遵循的模式的客户端的一个实例[使用单一实例模式方法](../azure-functions/manage-connections.md#documentclient-code-example-c). 此过程将避免潜在的套接字问题，在您的操作。

## <a name="common-scenarios-and-workarounds"></a>常见方案和解决方法

### <a name="azure-function-fails-with-error-message-collection-doesnt-exist"></a>Azure 函数失败，错误消息集合不存在

Azure 函数失败，出现错误消息"任一源集合集合-name （在数据库数据库名称中） 或租约集合 collection2-name （在数据库 ' database2-name'） 不存在。 侦听器开始之前，必须存在这两个集合。 若要自动创建租用集合，请设置为 'true' CreateLeaseCollectionIfNotExists'"

这意味着一个或两个触发器才能正常工作所需的 Azure Cosmos 容器不存在或无法对 Azure 函数。 **与错误本身会告诉您哪些 Azure Cosmos 数据库和容器是寻找触发器**基于你的配置。

1. 验证是否`ConnectionStringSetting`属性以及其**引用存在于 Azure Function App 的设置**。 此属性的值不应为该连接字符串本身，但配置设置的名称。
2. 确认`databaseName`和`collectionName`Azure Cosmos 帐户中存在。 如果您使用的自动值替换 (使用`%settingName%`模式)，请确保你的 Azure 函数应用中存在的设置的名称。
3. 如果未指定`LeaseCollectionName/leaseCollectionName`，默认值是"租约"。 验证存在这样的容器。 或者也可以设置`CreateLeaseCollectionIfNotExists`触发器中的属性`true`来自动创建它。
4. 验证你[Azure Cosmos 帐户的防火墙配置](how-to-configure-firewall.md)以查看它是否不会阻止 Azure 函数。

### <a name="azure-function-fails-to-start-with-shared-throughput-collection-should-have-a-partition-key"></a>Azure 函数开始失败"共享吞吐量集合应具有分区键"

早期版本的 Azure Cosmos DB 扩展插件不支持使用在中创建的租用容器[共享的吞吐量数据库](./set-throughput.md#set-throughput-on-a-database)。 若要解决此问题，更新[Microsoft.Azure.WebJobs.Extensions.CosmosDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB)扩展以获取最新版本。

### <a name="azure-function-fails-to-start-with-the-lease-collection-if-partitioned-must-have-partition-key-equal-to-id"></a>Azure 函数开始失败"租用集合中，如果分区，必须具有分区键设置为 id。"

此错误表示当前租用容器进行分区，但不是分区键路径`/id`。 若要解决此问题，需要重新创建具有租用容器`/id`作为分区键。

### <a name="you-see-a-value-cannot-be-null-parameter-name-o-in-your-azure-functions-logs-when-you-try-to-run-the-trigger"></a>您看到"值不能为 null。 参数名称： o"在 Azure Functions 日志时尝试运行触发器

如果你使用 Azure 门户并尝试进行选择，将出现此问题**运行**检查使用该触发器的 Azure 函数时在屏幕上的按钮。 触发器不需要供你选择运行以启动，它将自动启动时部署 Azure 函数。 如果你想要检查 Azure 门户上的 Azure 函数的日志流，只需转到受监视的容器，然后插入某些新项，将会自动看到触发器执行。

### <a name="my-changes-take-too-long-be-received"></a>接收我的更改时间过长

此方案中可以有多个原因，应检查所有这些：

1. 在你的 Azure Cosmos 帐户所在的同一区域中部署 Azure 函数？ 为了获得最佳的网络延迟，Azure 函数和你的 Azure Cosmos 帐户应可归置到同一 Azure 区域中。
2. 发生在 Azure Cosmos 容器连续或偶发中所做的更改？
如果是后者，可能有一定的延迟之间所做的更改存储和 Azure 函数选取它们。 这是时间的因为在内部，当触发器检查你的 Azure Cosmos 容器中的更改，并找到无挂起的要读取，它将休眠的时间以在可配置 （默认情况下 5 秒） 内检查新的更改 （以避免 RU 消耗量偏高） 之前。 可以配置通过此休眠时间`FeedPollDelay/feedPollDelay`中设置[配置](../azure-functions/functions-bindings-cosmosdb-v2.md#trigger---configuration)的触发器 （的值应为以毫秒为单位）。
3. Azure Cosmos 容器可能[速率限制](./request-units.md)。
4. 可以使用`PreferredLocations`触发器以指定的 Azure 区域来定义自定义首选的连接顺序以逗号分隔列表中的属性。

### <a name="some-changes-are-missing-in-my-trigger"></a>某些更改未在我的触发器

如果您发现，一些在 Azure Cosmos 容器中发生的更改不会被选取该 Azure 函数，则需要执行的一个初始的调查步骤。

当 Azure Function 收到所做的更改时，它通常处理它们，并可以根据需要，将结果发送到另一个目标。 当调查丢失更改时，请确保您**度量值的更改在引入点接收**（在 Azure 函数开始时），不在目标上。

如果在目标上缺少一些更改，这可能意味着这是接收到所做的更改后，在 Azure 函数执行期间出现一些错误。

在此方案中，最好的做法是添加`try/catch blocks`在代码中，可能会处理这些更改，以检测任何失败的项的特定子集并相应地处理它们的循环内 （将其发送到另一个存储以进行进一步分析或重试）。 

> [!NOTE]
> Azure Cosmos DB 触发器，默认情况下，不会重试一批更改如果代码执行期间出现未处理异常。 这意味着所做的更改未在目标位置到达的原因，是因为你无法处理它们。

如果你发现的某些更改未在所有收到了触发器，最常见情况是，则**正在运行另一个 Azure 函数**。 它可能是在 Azure 中部署的另一个 Azure 函数或开发人员的计算机上本地运行 Azure Function**完全相同的配置**（相同监视和租用容器） 和窃取该 Azure 函数你所期望 Azure 函数处理的更改的子集。

此外，可以验证方案，如果您知道多少个 Azure Function App 实例必须运行。 如果检查租约容器和租赁项目中的非重复值数目进行计数`Owner`属性中它们应等于在函数应用的实例数。 如果有多个所有者只能包含已知的 Azure Function App 实例，则表示这些额外的所有者是一个"窃取"所做的更改。

一种简单方法，解决这种情况下，是应用`LeaseCollectionPrefix/leaseCollectionPrefix`到新的/不同值的函数或，或者，使用新的租约容器进行测试。

## <a name="next-steps"></a>后续步骤

* [启用 Azure Functions 的监视](../azure-functions/functions-monitoring.md)
* [Azure Cosmos DB.NET SDK 进行故障排除](./troubleshoot-dot-net-sdk.md)