---
title: 如何通过 Node.js-Azure 存储使用队列存储
description: 了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。 相关示例是使用 Node.js 编写的。
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: nodejs
ms.topic: article
ms.date: 12/08/2016
ms.author: mhopkins
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: 01afe1ab7b9028f3f77d52f7d6f8ced27f6a79c7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142713"
---
# <a name="how-to-use-queue-storage-from-nodejs"></a>如何通过 Node.js 使用队列存储
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-check-out-samples-all](../../../includes/storage-check-out-samples-all.md)]

## <a name="overview"></a>概述
本指南演示如何使用 Microsoft Azure 队列服务执行常见任务。 相关示例是使用 Node.js API 编写的。 介绍的方案包括**插入**、**扫视**、**获取**和**删除**队列消息以及**创建和删除队列**。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-nodejs-application"></a>创建 Node.js 应用程序
创建一个空的 Node.js 应用程序。 有关创建 Node.js 应用程序的说明，请参阅[在 Azure 应用服务中创建 Node.js Web 应用](../../app-service/app-service-web-get-started-nodejs.md)、[使用 Windows PowerShell 生成 Node.js 应用程序并将其部署到 Azure 云服务](../../cloud-services/cloud-services-nodejs-develop-deploy-app.md)或 [Visual Studio Code](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial)。

## <a name="configure-your-application-to-access-storage"></a>配置应用程序以访问存储
若要使用 Azure 存储，需要 Azure Storage SDK for Node.js，其中包括一组便于与存储 REST 服务进行通信的库。

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>使用 Node 包管理器 (NPM) 可获取该程序包
1. 使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到在其中创建了示例应用程序的文件夹。
2. 在命令窗口中键入 **npm install azure-storage**。 该命令的输出类似于以下示例。
 
    ```bash
    azure-storage@0.5.0 node_modules\azure-storage
    +-- extend@1.2.1
    +-- xmlbuilder@0.4.3
    +-- mime@1.2.11
    +-- node-uuid@1.4.3
    +-- validator@3.22.2
    +-- underscore@1.4.4
    +-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
    +-- xml2js@0.2.7 (sax@0.5.2)
    +-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
    ```

3. 可以手动运行 **ls** 命令来验证是否创建了 **node\_modules** 文件夹。 在该文件夹中将找到 **azure-storage** 包，其中包含访问存储所需的库。

### <a name="import-the-package"></a>导入包
使用记事本或其他文本编辑器，将以下内容添加到要在其中使用存储的应用程序的 **server.js** 文件顶部：

```javascript
var azure = require('azure-storage');
```

## <a name="setup-an-azure-storage-connection"></a>设置 Azure 存储连接
Azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY，或 AZURE\_STORAGE\_CONNECTION\_STRING 以获取连接到 Azure 存储帐户所需的信息。 如果未设置这些环境变量，则必须在调用 **createQueueService** 时指定帐户信息。

## <a name="how-to-create-a-queue"></a>如何：创建队列
以下代码将创建一个 **QueueService** 对象，可通过该对象来操作队列。

```javascript
var queueSvc = azure.createQueueService();
```

使用 **createQueueIfNotExists** 方法，该方法返回指定的队列（如果已存在），或创建具有指定名称的新队列（如果尚不存在）。

```javascript
queueSvc.createQueueIfNotExists('myqueue', function(error, results, response){
  if(!error){
    // Queue created or exists
  }
});
```

如果创建了队列，则 `result.created` 为 true。 如果队列已存在，则 `result.created` 为 false。

### <a name="filters"></a>筛选器
可以向使用 **QueueService** 执行的操作应用可选的筛选操作。 筛选操作可包括日志记录、自动重试等。筛选器是实现具有签名的方法的对象：

```javascript
function handle (requestOptions, next)
```

在对请求选项执行预处理后，该方法需要调用“next”并且传递具有以下签名的回调：

```javascript
function (returnObject, finalCallback, next)
```

在此回调中，在处理 returnObject（来自对服务器的请求的响应）后，回调需要调用 next（如果它已存在）以继续处理其他筛选器或只调用 finalCallback 以结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 **ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。 以下代码将创建一个 **QueueService** 对象，该对象使用 **ExponentialRetryPolicyFilter**：

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var queueSvc = azure.createQueueService().withFilter(retryOperations);
```

## <a name="how-to-insert-a-message-into-a-queue"></a>如何：在队列中插入消息
要在队列中插入消息，可使用 **createMessage** 方法创建一条新消息并将其添加到队列中。

```javascript
queueSvc.createMessage('myqueue', "Hello world!", function(error, results, response){
  if(!error){
    // Message inserted
  }
});
```

## <a name="how-to-peek-at-the-next-message"></a>如何：扫视下一条消息
通过调用 **peekMessages** 方法，可查看队列前面的消息，而不必从队列中将其删除。 默认情况下，**peekMessages** 扫视单条消息。

```javascript
queueSvc.peekMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
  }
});
```

`result` 包含该消息。

> [!NOTE]
> 如果队列中没有消息，使用 **peekMessages** 不会返回错误，但也不会返回消息。
> 
> 

## <a name="how-to-dequeue-the-next-message"></a>如何：取消下一条消息的排队
处理消息是一个两阶段过程：

1. 取消消息的排队。
2. 删除消息。

若要取消消息的排队，请使用 **getMessages**。 这会使消息在队列中不可见，使其他客户端无法处理它们。 应用程序处理完某条消息后，即可调用 **deleteMessage** 将其从队列中删除。 下面的示例获取了一条消息，然后将其删除：

```javascript
queueSvc.getMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
    var message = results[0];
    queueSvc.deleteMessage('myqueue', message.messageId, message.popReceipt, function(error, response){
      if(!error){
        //message deleted
      }
    });
  }
});
```

> [!NOTE]
> 默认情况下，一条消息只会隐藏 30 秒，其他客户端就可以看见它。 可以将 `options.visibilityTimeout` 与 **getMessages** 配合使用，以指定其他值。
> 
> [!NOTE]
> 在队列中没有消息时使用 **getMessages** 不会返回错误，但也不会返回消息。
> 
> 

## <a name="how-to-change-the-contents-of-a-queued-message"></a>如何：更改已排队消息的内容
可更改队列中现有消息的内容，只需使用 **updateMessage** 即可。 以下示例将更新消息的文本：

```javascript
queueSvc.getMessages('myqueue', function(error, getResults, getResponse){
  if(!error){
    // Got the message
    var message = getResults[0];
    queueSvc.updateMessage('myqueue', message.messageId, message.popReceipt, 10, {messageText: 'new text'}, function(error, updateResults, updateResponse){
      if(!error){
        // Message updated successfully
      }
    });
  }
});
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>如何：取消消息排队的其他选项
可以通过两种方式自定义队列中的消息检索：

* `options.numOfMessages` - 获取一批消息（最多 32 条）。
* `options.visibilityTimeout` - 设置较长或较短的不可见性超时。

以下示例使用 **getMessages** 方法通过一次调用获取 15 条消息。 然后，使用 for 循环处理每条消息。 它还会将通过此方法返回的所有消息的不可见性超时设置为 5 分钟。

```javascript
queueSvc.getMessages('myqueue', {numOfMessages: 15, visibilityTimeout: 5 * 60}, function(error, results, getResponse){
  if(!error){
    // Messages retrieved
    for(var index in result){
      // text is available in result[index].messageText
      var message = results[index];
      queueSvc.deleteMessage(queueName, message.messageId, message.popReceipt, function(error, deleteResponse){
        if(!error){
          // Message deleted
        }
      });
    }
  }
});
```

## <a name="how-to-get-the-queue-length"></a>如何：获取队列长度
**getQueueMetadata** 返回有关队列的元数据，其中包括队列中等待的消息的大致数目。

```javascript
queueSvc.getQueueMetadata('myqueue', function(error, results, response){
  if(!error){
    // Queue length is available in results.approximateMessageCount
  }
});
```

## <a name="how-to-list-queues"></a>如何：列出队列
若要检索队列的列表，请使用 **listQueuesSegmented**。 若要检索按特定前缀筛选的列表，请使用 **listQueuesSegmentedWithPrefix**。

```javascript
queueSvc.listQueuesSegmented(null, function(error, results, response){
  if(!error){
    // results.entries contains the list of queues
  }
});
```

如果无法返回所有队列，可使用 `result.continuationToken` 作为 **listQueuesSegmented** 的第一个参数或 **listQueuesSegmentedWithPrefix** 的第二个参数，以检索更多结果。

## <a name="how-to-delete-a-queue"></a>如何：删除队列
若要删除队列及其中包含的所有消息，请对队列对象调用 **deleteQueue** 方法。

```javascript
queueSvc.deleteQueue(queueName, function(error, response){
  if(!error){
    // Queue has been deleted
  }
});
```

若要清除队列中的所有消息而不删除该队列，则可使用 **clearMessages**。

## <a name="how-to-work-with-shared-access-signatures"></a>如何：使用共享访问签名
共享访问签名 (SAS) 是一种安全的方法，用于对队列进行粒度访问，而无需提供存储帐户名或密钥。 通常使用 SAS 以提供对队列的有限访问权限，例如允许移动应用提交消息。

受信任的应用程序（例如基于云的服务）可使用 **QueueService** 的 **generateSharedAccessSignature** 生成 SAS，然后将其提供给不受信任的或不完全受信任的应用程序。 例如，移动应用。 可使用策略生成 SAS，该策略描述了 SAS 的生效日期和失效日期，以及授予 SAS 持有者的访问级别。

下面的示例生成了一个新的共享访问策略，该策略将允许 SAS 持有者向队列添加消息，此策略在创建后 100 分钟过期。

```javascript
var startDate = new Date();
var expiryDate = new Date(startDate);
expiryDate.setMinutes(startDate.getMinutes() + 100);
startDate.setMinutes(startDate.getMinutes() - 100);

var sharedAccessPolicy = {
  AccessPolicy: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};

var queueSAS = queueSvc.generateSharedAccessSignature('myqueue', sharedAccessPolicy);
var host = queueSvc.host;
```

请注意，还必须提供主机信息，因为当 SAS 持有者尝试访问队列时，必须提供该信息。

然后，客户端应用程序将 SAS 用于 **QueueServiceWithSAS**，以针对队列执行操作。 下面的示例连接到该队列，并创建一条消息。

```javascript
var sharedQueueService = azure.createQueueServiceWithSas(host, queueSAS);
sharedQueueService.createMessage('myqueue', 'Hello world from SAS!', function(error, result, response){
  if(!error){
    //message added
  }
});
```

由于 SAS 在生成时只具有添加访问权限，因此如果尝试读取、更新或删除消息，则会返回错误。

### <a name="access-control-lists"></a>访问控制列表
还可以使用访问控制列表 (ACL) 为 SAS 设置访问策略。 如果希望允许多个客户端访问队列，但为每个客户端提供不同的访问策略，则访问控制列表会很有用。

ACL 是使用一组访问策略实施的，每个策略都有一个关联的 ID。 以下示例定义了两个策略，一个用于“user1”，一个用于“user2”：

```javascript
var sharedAccessPolicy = {
  user1: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.PROCESS,
    Start: startDate,
    Expiry: expiryDate
  },
  user2: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};
```

下面的示例获取 **myqueue** 的当前 ACL，并使用 **setQueueAcl** 添加新策略。 此方法具有以下用途：

```javascript
var extend = require('extend');
queueSvc.getQueueAcl('myqueue', function(error, result, response) {
  if(!error){
    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
    queueSvc.setQueueAcl('myqueue', newSignedIdentifiers, function(error, result, response){
      if(!error){
        // ACL set
      }
    });
  }
});
```

设置 ACL 后，可以根据某个策略的 ID 创建 SAS。 以下示例为“user2”创建新的 SAS：

```javascript
queueSAS = queueSvc.generateSharedAccessSignature('myqueue', { Id: 'user2' });
```

## <a name="next-steps"></a>后续步骤
现在，已了解有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

* 访问 [Azure 存储团队博客][Azure Storage Team Blog]。
* 访问 GitHub 上[用于 Node 的 Azure 存储 SDK][Azure Storage SDK for Node] 存储库。



[Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node

[using the REST API]: https://msdn.microsoft.com/library/azure/hh264518.aspx

[Azure Portal]: https://portal.azure.com

[在 Azure 应用服务中创建 Node.js Web 应用](../../app-service/app-service-web-get-started-nodejs.md)

[生成 Node.js 应用程序并将其部署到 Azure 云服务](../../cloud-services/cloud-services-nodejs-develop-deploy-app.md)

[Azure Storage Team Blog]: https://blogs.msdn.com/b/windowsazurestorage/

[Build and deploy a Node.js web app to Azure using Web Matrix]: https://www.microsoft.com/web/webmatrix/
