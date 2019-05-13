---
title: 如何通过 Ruby 的 Azure 存储使用队列存储
description: 了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。 用 Ruby 编写的相关示例。
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: ruby
ms.topic: article
ms.date: 12/08/2016
ms.author: mhopkins
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: 30a090aeb2d66c732e70a9acce67d5f3374c32fa
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153150"
---
# <a name="how-to-use-queue-storage-from-ruby"></a>如何通过 Ruby 使用队列存储
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概述
本指南演示如何使用 Microsoft Azure 队列存储服务执行常见方案。 相关示例是使用 Ruby Azure API 编写的。
介绍的方案包括**插入**、**扫视**、**获取**和**删除**队列消息以及**创建和删除队列**。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-ruby-application"></a>创建 Ruby 应用程序
创建 Ruby 应用程序。 有关说明，请参阅[使用 Linux 应用服务创建 Ruby 应用](https://docs.microsoft.com/azure/app-service/containers/quickstart-ruby)。

## <a name="configure-your-application-to-access-storage"></a>配置应用程序以访问存储
要使用 Azure 存储，需下载和使用 Ruby Azure 包，其中包括与存储 REST 服务进行通信的一组方便的库。

### <a name="use-rubygems-to-obtain-the-package"></a>使用 RubyGems 获取该程序包
1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。
2. 在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### <a name="import-the-package"></a>导入包
使用常用的文本编辑器将以下内容添加到要在其中使用存储的 Ruby 文件的顶部：

```ruby
require "azure"
```

## <a name="setup-an-azure-storage-connection"></a>设置 Azure 存储连接
Azure 模块会读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS_KEY** 以获取连接到 Azure 存储帐户所需的信息。 如果未设置这些环境变量，则在使用 **Azure::QueueService** 前必须通过以下代码指定帐户信息：

```ruby
Azure.config.storage_account_name = "<your azure storage account>"
Azure.config.storage_access_key = "<your Azure storage access key>"
```

从 Azure 门户中的经典或 Resource Manager 存储帐户中获取这些值：

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 导航到要使用的存储帐户。
3. 在右侧的“设置”边栏选项卡中，单击“访问密钥”。
4. 在显示的“访问密钥”边栏选项卡中，可看到访问密钥 1 和访问密钥 2。 可以使用其中任意一个。 
5. 单击复制图标以将键复制到剪贴板。 

## <a name="how-to-create-a-queue"></a>如何：创建队列
以下代码将创建一个 **Azure::QueueService** 对象，用于处理队列。

```ruby
azure_queue_service = Azure::QueueService.new
```

使用 **create_queue()** 方法创建具有指定名称的队列。

```ruby
begin
  azure_queue_service.create_queue("test-queue")
rescue
  puts $!
end
```

## <a name="how-to-insert-a-message-into-a-queue"></a>如何：在队列中插入消息
要在队列中插入消息，可使用 **create_message()** 方法创建一条新消息并将其添加到队列中。

```ruby
azure_queue_service.create_message("test-queue", "test message")
```

## <a name="how-to-peek-at-the-next-message"></a>如何：扫视下一条消息
可调用 **peek\_messages()** 方法，查看队列前面的消息，而不必从队列中将其删除。 默认情况下，**peek\_messages()** 扫视单条消息。 也可以指定要扫视的消息数。

```ruby
result = azure_queue_service.peek_messages("test-queue",
  {:number_of_messages => 10})
```

## <a name="how-to-dequeue-the-next-message"></a>如何：取消下一条消息的排队
可通过两个步骤从队列中删除消息。

1. 在调用 **list\_messages()** 时，默认情况下会获取队列中的下一条消息。 也可以指定要获取的消息数。 从 **list\_messages()** 返回的消息变得对从此队列读取消息的任何其他代码不可见。 将传递以秒为单位的可见性超时值作为参数。
2. 还必须调用 **delete_message()**，才能完成队列消息删除操作。

此删除消息的两步过程可确保当代码因硬件或软件故障而无法处理消息时，其他代码实例可以获取同一消息并重试。 代码在处理消息后会立即调用 **delete\_message()**。

```ruby
messages = azure_queue_service.list_messages("test-queue", 30)
azure_queue_service.delete_message("test-queue", 
  messages[0].id, messages[0].pop_receipt)
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>如何：更改已排队消息的内容
可以更改队列中现有消息的内容。 以下代码使用 **update_message()** 方法来更新消息。 该方法将返回一个元组，其中包含队列消息的 pop 接收方，以及一个 UTC 日期时间值，表示消息会在队列中可见的时间。

```ruby
message = azure_queue_service.list_messages("test-queue", 30)
pop_receipt, time_next_visible = azure_queue_service.update_message(
  "test-queue", message.id, message.pop_receipt, "updated test message", 
  30)
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>如何：取消消息排队的其他选项
可以通过两种方式自定义队列中的消息检索。

1. 可以获取一批消息。
2. 可以设置更长或更短的不可见超时时间，从而允许代码使用更多或更少的时间来完全处理每个消息。

以下代码示例使用 **list\_messages()** 方法通过一次调用获取 15 条消息。 然后，它打印并删除每条消息。 它还将每条消息的不可见超时时间设置为 5 分钟。

```ruby
azure_queue_service.list_messages("test-queue", 300
  {:number_of_messages => 15}).each do |m|
  puts m.message_text
  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
end
```

## <a name="how-to-get-the-queue-length"></a>如何：获取队列长度
可以获取队列中消息数的估计值。 **get\_queue\_metadata()** 方法要求队列服务返回有关队列的大概消息数和元数据。

```ruby
message_count, metadata = azure_queue_service.get_queue_metadata(
  "test-queue")
```

## <a name="how-to-delete-a-queue"></a>如何：删除队列
若要删除队列及其中包含的所有消息，请对队列对象调用 **delete\_queue()** 方法。

```ruby
azure_queue_service.delete_queue("test-queue")
```

## <a name="next-steps"></a>后续步骤
既已了解有关队列存储的基础知识，可单击以下链接以了解更复杂的存储任务。

* 访问 [Azure Storage Team Blog](https://blogs.msdn.com/b/windowsazurestorage/)（Azure 存储团队博客）
* 访问 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

若要了解 Azure 队列服务（本文所述）和 Azure 服务总线队列（[如何使用服务总线队列](https://azure.microsoft.com/develop/ruby/how-to-guides/service-bus-queues/)文章中所述）之间的比较，请参阅 [Azure 队列和服务总线队列 - 比较与对照](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md)
