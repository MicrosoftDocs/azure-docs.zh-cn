---
title: 如何通过 PHP-Azure 存储使用队列存储
description: 了解如何使用 Azure 队列存储服务创建和删除队列，以及插入、获取和删除消息。 示例用 PHP 编写。
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: php
ms.topic: article
ms.date: 01/11/2018
ms.author: mhopkins
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: b827894c633c425220bf94be1f51c52d464bd79b
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410452"
---
# <a name="how-to-use-queue-storage-from-php"></a>如何通过 PHP 使用队列存储
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概述
本指南演示如何使用 Azure 队列存储服务执行常见方案。 这些示例是通过[适用于 PHP 的 Azure 存储客户端库][download]中的类编写的。 介绍的方案包括插入、扫视、获取和删除队列消息以及创建和删除队列。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-php-application"></a>创建 PHP 应用程序
创建用于访问 Azure 队列存储的 PHP 应用程序的唯一要求是从代码中引用[适用于 PHP 的 Azure 存储客户端库][download]中的类。 可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，将使用队列存储服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## <a name="get-the-azure-client-libraries"></a>获取 Azure 客户端库
### <a name="install-via-composer"></a>通过 Composer 安装
1. 在项目的根目录中创建名为 **composer.json** 的文件并向其添加以下代码：
   
    ```json
    {
      "require": {
        "microsoft/azure-storage-queue": "*"
      }
    }
    ```
2. 将 [composer.phar][composer-phar] 下载到项目根目录中。
3. 打开命令提示符并在项目根目录中执行以下命令
   
    ```
    php composer.phar install
    ```

或者转到 GitHub 上的 [Azure 存储 PHP 客户端库][download]，然后克隆源代码。

## <a name="configure-your-application-to-access-queue-storage"></a>配置应用程序以访问队列存储
若要使用 Azure 队列存储 API，需执行以下操作：

1. 通过使用 [require_once] 语句引用 autoloader 文件。
2. 引用可使用的所有类。

以下示例演示如何添加 autoloader 文件和引用 QueueRestProxy 类。

```php
require_once 'vendor/autoload.php';
use MicrosoftAzure\Storage\Queue\QueueRestProxy;
```

以下示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## <a name="set-up-an-azure-storage-connection"></a>设置 Azure 存储连接
若要实例化 Azure 队列存储客户端，首先必须拥有有效的连接字符串。 队列服务连接字符串的格式如下。

对于访问实时服务：

```php
DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]
```

对于访问模拟器存储：

```php
UseDevelopmentStorage=true
```

若要创建 Azure 队列服务客户端，则需要使用 QueueRestProxy 类。 可以使用以下方法之一：

* 将连接字符串直接传递给它。
* 在 Web 应用中使用环境变量来存储连接字符串。 要配置连接字符串，请参阅 [Azure Web 应用配置设置](../../app-service/web-sites-configure.md)文档。
在此处列出的示例中，将直接传递连接字符串。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";
$queueClient = QueueRestProxy::createQueueService($connectionString);
```

## <a name="create-a-queue"></a>创建队列
**QueueRestProxy** 对象允许使用 **createQueue** 方法创建队列。 创建队列时，可以在该队列上设置选项，但此操作不是必需的。 （下面的示例演示了如何在队列上设置元数据。）

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateQueueOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set queue metadata.
$createQueueOptions = new CreateQueueOptions();
$createQueueOptions->addMetaData("key1", "value1");
$createQueueOptions->addMetaData("key2", "value2");

try    {
    // Create queue.
    $queueClient->createQueue("myqueue", $createQueueOptions);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> 元数据密钥不区分大小写。 所有密钥都是采用小写形式从服务中读取的。
> 
> 

## <a name="add-a-message-to-a-queue"></a>向队列添加消息
若要将消息添加到队列，请使用 QueueRestProxy->createMessage。 此方法接受队列名称、消息文本和消息选项（这些都是可选的）。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateMessageOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Create message.
    $queueClient->createMessage("myqueue", "Hello World!");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="peek-at-the-next-message"></a>扫视下一条消息
通过调用 QueueRestProxy->peekMessages，可以扫视队列前面的消息，而不会从队列中将其删除。 默认情况下，peekMessage 方法返回单条消息，但可以使用 PeekMessagesOptions->setNumberOfMessages 方法更改该值。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\PeekMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set peek message options.
$message_options = new PeekMessagesOptions();
$message_options->setNumberOfMessages(1); // Default value is 1.

try    {
    $peekMessagesResult = $queueClient->peekMessages("myqueue", $message_options);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$messages = $peekMessagesResult->getQueueMessages();

// View messages.
$messageCount = count($messages);
if($messageCount <= 0){
    echo "There are no messages.<br />";
}
else{
    foreach($messages as $message)    {
        echo "Peeked message:<br />";
        echo "Message Id: ".$message->getMessageId()."<br />";
        echo "Date: ".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
        echo "Message text: ".$message->getMessageText()."<br /><br />";
    }
}
```

## <a name="de-queue-the-next-message"></a>取消对下一条消息的排队
代码分两步从队列中删除消息。 首先，调用 **QueueRestProxy->listMessages**，这会使消息对从队列中读取的任何其他代码不可见。 默认情况下，此消息将持续 30 秒不可见。 （如果在此时段内未删除该消息，它会在队列上再次可见。）若要从队列中删除消息，则必须调用 **QueueRestProxy->deleteMessage**。 此删除消息的两步过程可确保当代码因硬件或软件故障而无法处理消息时，其他代码实例可以获取同一消息并重试。 代码在处理消息后会立即调用 **deleteMessage**。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

/* ---------------------
    Process message.
   --------------------- */

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Delete message.
    $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="change-the-contents-of-a-queued-message"></a>更改已排队消息的内容
可以通过调用 **QueueRestProxy->updateMessage** 来更改队列中已就位消息的内容。 如果消息表示工作任务，则可以使用此功能来更新该工作任务的状态。 以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。 这会保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。 可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。 通常同时保留重试计数，当消息重试次数超过 *n* 时再删除该消息。 这可避免每次处理某条消息时都触发应用程序错误。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

// Define new message properties.
$new_message_text = "New message text.";
$new_visibility_timeout = 5; // Measured in seconds.

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Update message.
    $queueClient->updateMessage("myqueue",
                                $messageId,
                                $popReceipt,
                                $new_message_text,
                                $new_visibility_timeout);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="additional-options-for-de-queuing-messages"></a>用于取消对消息进行排队的其他选项
可通过两种方式自定义队列中的消息检索。 首先，可获取一批消息（最多 32 条）。 其次，可以设置更长或更短的可见超时，从而允许代码使用更多或更少的时间来彻底处理每条消息。 以下代码示例使用 **getMessages** 方法在一次调用中获取 16 条消息。 然后，使用 **for** 循环处理每条消息。 它还将每条消息的不可见超时时间设置为 5 分钟。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\ListMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Set list message options.
$message_options = new ListMessagesOptions();
$message_options->setVisibilityTimeoutInSeconds(300);
$message_options->setNumberOfMessages(16);

// Get messages.
try{
    $listMessagesResult = $queueClient->listMessages("myqueue",
                                                     $message_options);
    $messages = $listMessagesResult->getQueueMessages();

    foreach($messages as $message){

        /* ---------------------
            Process message.
        --------------------- */

        // Get message Id and pop receipt.
        $messageId = $message->getMessageId();
        $popReceipt = $message->getPopReceipt();

        // Delete message.
        $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
    }
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="get-queue-length"></a>获取队列长度
可以获取队列中消息的估计数。 **QueueRestProxy->getQueueMetadata** 方法要求队列服务返回有关队列的元数据。 对返回的对象调用 **getApproximateMessageCount** 方法将提供队列中消息的计数。 此计数仅为近似值，因为只能在队列服务响应请求后添加或删除消息。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Get queue metadata.
    $queue_metadata = $queueClient->getQueueMetadata("myqueue");
    $approx_msg_count = $queue_metadata->getApproximateMessageCount();
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

echo $approx_msg_count;
```

## <a name="delete-a-queue"></a>删除队列
若要删除队列及其包含的所有消息，请调用 **QueueRestProxy->deleteQueue** 方法。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Delete queue.
    $queueClient->deleteQueue("myqueue");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="next-steps"></a>后续步骤
现在，已了解有关 Azure 队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

* 请访问 [Azure 存储 PHP 客户端库的 API 参考](https://azure.github.io/azure-storage-php/)
* 请参阅[高级队列示例](https://github.com/Azure/azure-storage-php/blob/master/samples/QueueSamples.php)。

有关详细信息，另请参阅 [PHP 开发人员中心](https://azure.microsoft.com/develop/php/)。

[download]: https://github.com/Azure/azure-storage-php
[require_once]: https://www.php.net/manual/en/function.require-once.php
[Azure Portal]: https://portal.azure.com
[composer-phar]: https://getcomposer.org/composer.phar

