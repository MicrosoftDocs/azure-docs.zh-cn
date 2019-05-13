---
title: Azure Analysis Services 模型的异步刷新 | Microsoft Docs
description: 了解如何使用 REST API 编写异步刷新的代码。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/09/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 63b64df457af5b7d3d2bd5901f73d89ccd3c913a
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506977"
---
# <a name="asynchronous-refresh-with-the-rest-api"></a>使用 REST API 执行异步刷新

使用支持 REST 调用的任何编程语言，可以针对 Azure Analysis Services 表格模型执行异步数据刷新操作。 这包括同步只读副本以进行查询扩展。 

数据刷新操作可能需要一段时间，具体取决于多种因素，包括数据卷、使用分区的优化级别，等等。在传统上，这些操作是使用现有方法调用的，例如，使用 [TOM](https://docs.microsoft.com/sql/analysis-services/tabular-model-programming-compatibility-level-1200/introduction-to-the-tabular-object-model-tom-in-analysis-services-amo)（表格对象模型）、[PowerShell](https://docs.microsoft.com/sql/analysis-services/powershell/analysis-services-powershell-reference) cmdlet，或 [TMSL](https://docs.microsoft.com/sql/analysis-services/tabular-model-scripting-language-tmsl-reference)（表格模型脚本语言）。 但是，这些方法可能需要通常不可靠的且长时间运行的 HTTP 连接。

使用 Azure Analysis Services 的 REST API 能够以异步方式执行数据刷新操作。 如果使用 REST API，则不需要从客户端应用程序建立长时间运行的 HTTP 连接。 还有其他内置功能可以确保可靠性，例如自动重试和分批提交。

## <a name="base-url"></a>基 URL

基 URL 遵循以下格式：

```
https://<rollout>.asazure.windows.net/servers/<serverName>/models/<resource>/
```

例如，假设某个模型名为 AdventureWorks，位于美国西部 Azure 区域中名为 myserver 的服务器上。 此服务器名称为：

```
asazure://westus.asazure.windows.net/myserver 
```

此服务器名称的基 URL 为：

```
https://westus.asazure.windows.net/servers/myserver/models/AdventureWorks/ 
```

使用基 URL 可以根据以下参数追加资源和操作： 

![异步刷新](./media/analysis-services-async-refresh/aas-async-refresh-flow.png)

- 以 **s** 结尾的任何内容是集合。
- 以 **()** 结尾的任何内容是函数。
- 其他任何内容是资源/对象。

例如，可以在 Refreshes 集合中使用 POST 谓词来执行刷新操作：

```
https://westus.asazure.windows.net/servers/myserver/models/AdventureWorks/refreshes
```

## <a name="authentication"></a>Authentication

所有调用必须使用 Authorization 标头中的有效 Azure Active Directory (OAuth 2) 令牌进行身份验证，并且必须满足以下要求：

- 令牌必须是用户令牌或应用程序服务主体。
- 在令牌中，必须将正确的受众设置为 `https://*.asazure.windows.net`。
- 用户或应用程序必须在服务器或模型中具有足够的权限才能发出请求的调用。 权限级别由模型或者服务器上的管理员组中的角色确定。

    > [!IMPORTANT]
    > 目前，**服务器管理员**角色权限是必需的。

## <a name="post-refreshes"></a>POST /refreshes

若要执行刷新操作，可以在 /refreshes 集合中使用 POST 谓词将一个新的刷新项添加到该集合。 响应中的 Location 标头包含刷新 ID。 以后，客户端应用程序可以根据需要断开连接并检查状态，因为它是异步的。

模型每次只接受一个刷新操作。 如果当前正在运行一个刷新操作并提交了另一个刷新操作，则会返回“409 冲突”HTTP 状态代码。

正文可如下所示：

```
{
    "Type": "Full",
    "CommitMode": "transactional",
    "MaxParallelism": 2,
    "RetryCount": 2,
    "Objects": [
        {
            "table": "DimCustomer",
            "partition": "DimCustomer"
        },
        {
            "table": "DimDate"
        }
    ]
}
```

### <a name="parameters"></a>parameters

不需要指定参数。 将应用默认值。

| 名称             | 类型  | 描述  |默认  |
|------------------|-------|--------------|---------|
| `Type`           | 枚举  | 要执行的处理类型。 类型与 TMSL [refresh 命令](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/refresh-command-tmsl)类型相符：full、clearValues、calculate、dataOnly、automatic 和 defragment。 Add 类型不受支持。      |   automatic      |
| `CommitMode`     | 枚举  | 确定是要分批提交对象，还是只在完成时才提交。 模式包括：default、transactional、partialBatch。  |  transactional       |
| `MaxParallelism` | Int   | 此值确定用于并行运行处理命令的最大线程数。 此值与 MaxParallelism 属性（可以在 TMSL [Sequence 命令](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/sequence-command-tmsl)中或使用其他方法设置此属性）相符。       | 10        |
| `RetryCount`     | Int   | 指示操作在失败之前要重试的次数。      |     0    |
| `Objects`        | 阵列 | 要处理的对象数组。 每个对象包含：“table”（处理整个表时），或者“table”和“partition”（处理分区时）。 如果未指定任何对象，则会刷新整个模型。 |   处理整个模型      |

CommitMode 等于 partialBatch。 针对大型数据集执行可能需要几个小时的初始加载时，将会使用 CommitMode。 如果在成功提交一个或多个批之后刷新操作失败，则成功提交的批将保留已提交状态（不会回滚已成功提交的批）。

> [!NOTE]
> 在撰写本文时，批大小为 MaxParallelism 值，但可以更改此值。

## <a name="get-refreshesrefreshid"></a>GET /refreshes/\<refreshId>

若要检查刷新操作的状态，可以在刷新 ID 中使用 GET 谓词。 下面是响应正文的示例。 如果操作正在进行，则会在状态中返回 **inProgress**。

```
{
    "startTime": "2017-12-07T02:06:57.1838734Z",
    "endTime": "2017-12-07T02:07:00.4929675Z",
    "type": "full",
    "status": "succeeded",
    "currentRefreshType": "full",
    "objects": [
        {
            "table": "DimCustomer",
            "partition": "DimCustomer",
            "status": "succeeded"
        },
        {
            "table": "DimDate",
            "partition": "DimDate",
            "status": "succeeded"
        }
    ]
}
```

## <a name="get-refreshes"></a>GET /refreshes

若要获取模型的历史刷新操作列表，可以在 /refreshes 集合中使用 GET 谓词。 下面是响应正文的示例。 

> [!NOTE]
> 在撰写本文时，将存储并返回过去 30 天的刷新操作，但可以更改此数字。

```
[
    {
        "refreshId": "1344a272-7893-4afa-a4b3-3fb87222fdac",
        "startTime": "2017-12-09T01:58:04.76",
        "endTime": "2017-12-09T01:58:12.607",
        "status": "succeeded"
    },
    {
        "refreshId": "474fc5a0-3d69-4c5d-adb4-8a846fa5580b",
        "startTime": "2017-12-07T02:05:48.32",
        "endTime": "2017-12-07T02:05:54.913",
        "status": "succeeded"
    }
]
```

## <a name="delete-refreshesrefreshid"></a>DELETE /refreshes/\<refreshId>

若要取消正在进行的刷新操作，可以在刷新 ID 中使用 DELETE 谓词。

## <a name="post-sync"></a>POST /sync

执行刷新操作后，可能需要将新数据与副本同步，以进行查询扩展。若要对模型执行同步操作，可以在 /sync 函数中使用 POST 谓词。 响应中的 Location 标头包含同步操作 ID。

## <a name="get-sync-status"></a>GET /sync status

若要检查同步操作的状态，可以使用 GET 谓词，以参数的形式传递操作 ID。 下面是响应正文的示例：

```
{
    "operationId": "cd5e16c6-6d4e-4347-86a0-762bdf5b4875",
    "database": "AdventureWorks2",
    "UpdatedAt": "2017-12-09T02:44:26.18",
    "StartedAt": "2017-12-09T02:44:20.743",
    "syncstate": 2,
    "details": null
}
```

`syncstate` 的值：

- 0：正在复制。 正在将数据库文件复制到目标文件夹。
- 1:正在解冻。 正在只读的服务器实例上解冻数据库。
- 2:已完成。 同步操作已成功完成。
- 3：已失败。 同步操作失败。
- 4：正在完成。 同步操作已完成，但正在执行清理步骤。

## <a name="code-sample"></a>代码示例

下面是一个可以帮助你入门的 C# 代码示例：[GitHub 上的 RestApiSample](https://github.com/Microsoft/Analysis-Services/tree/master/RestApiSample)。

### <a name="to-use-the-code-sample"></a>使用代码示例

1.  克隆或下载存储库。 打开 RestApiSample 解决方案。
2.  找到 **client.BaseAddress = …** 行 并提供自己的[基 URL](#base-url)。

该代码示例使用[服务主体](#service-principal)身份验证。

### <a name="service-principal"></a>服务主体

有关如何在 Azure AS 中设置服务主体和分配必要权限的详细信息，请参阅[创建服务主体 - Azure 门户](../active-directory/develop/howto-create-service-principal-portal.md)和[将服务主体添加到服务器管理员角色](analysis-services-addservprinc-admins.md)。 完成上述步骤后，请完成以下附加步骤：

1.  在代码示例中，找到 **string authority = …**，将 **common** 替换为组织的租户 ID。
2.  注释/取消注释，以便使用 ClientCredential 类来实例化 cred 对象。 确保以安全方式访问 \<App ID> 和 \<App Key> 值，或者对服务主体使用基于证书的身份验证。
3.  运行示例。


## <a name="see-also"></a>另请参阅

[示例](analysis-services-samples.md)   
[REST API](https://docs.microsoft.com/rest/api/analysisservices/servers)   


