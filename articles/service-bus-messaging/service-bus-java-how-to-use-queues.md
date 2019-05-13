---
title: 如何通过 Java 使用 Azure 服务总线队列 | Microsoft Docs
description: 了解如何在 Azure 中使用服务总线队列。 用 Java 编写的代码示例。
services: service-bus-messaging
documentationcenter: java
author: axisc
manager: timlt
editor: spelluru
ms.assetid: f701439c-553e-402c-94a7-64400f997d59
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 04/10/2019
ms.author: aschhab
ms.openlocfilehash: 032c4af0e36626881b514573cc4a5f966e8c2077
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510311"
---
# <a name="how-to-use-service-bus-queues-with-java"></a>如何通过 Java 使用服务总线队列
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]
在本教程中，您将学习如何创建 Java 应用程序将消息发送到和从服务总线队列接收消息。 

> [!NOTE]
> 可在 [azure-service-bus 存储库](https://github.com/Azure/azure-service-bus/tree/master/samples/Java)中的 GitHub 上找到 Java 示例。

## <a name="prerequisites"></a>必备组件
1. Azure 订阅。 要完成本教程，需要一个 Azure 帐户。 可以激活您[MSDN 订户权益](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF)或注册[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF)。
2. 如果没有要处理的队列，按照步骤[使用 Azure 门户创建服务总线队列](service-bus-quickstart-portal.md)项目创建的队列。
    1. 阅读快速**概述**的服务总线**队列**。 
    2. 创建服务总线**命名空间**。 
    3. 获取**连接字符串**。
    4. 创建服务总线**队列**。
3. 安装[用于 Java 的 Azure SDK][Azure SDK for Java]。 


## <a name="configure-your-application-to-use-service-bus"></a>配置应用程序以使用服务总线
在生成本示例之前，请确保已安装 [Azure SDK for Java][Azure SDK for Java]。 如果使用 Eclipse，则可以安装包含 Azure SDK for Java 的[用于 Eclipse 的 Azure 工具包][Azure Toolkit for Eclipse]。 然后，可将 Microsoft Azure Libraries for Java 添加到项目：

![](./media/service-bus-java-how-to-use-queues/eclipselibs.png)

将以下 `import` 语句添加到 Java 文件顶部：

```java
// Include the following imports to use Service Bus APIs
import com.google.gson.reflect.TypeToken;
import com.microsoft.azure.servicebus.*;
import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;
import com.google.gson.Gson;

import static java.nio.charset.StandardCharsets.*;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.*;

import org.apache.commons.cli.*;

```

## <a name="send-messages-to-a-queue"></a>向队列发送消息
要将消息发送到服务总线队列，应用程序将实例化 QueueClient 对象并异步发送消息。 下面的代码演示如何发送通过门户创建的队列的消息。

```java
public void run() throws Exception {
    // Create a QueueClient instance and then asynchronously send messages.
    // Close the sender once the send operation is complete.
    QueueClient sendClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
    this.sendMessageAsync(sendClient).thenRunAsync(() -> sendClient.closeAsync());

    sendClient.close();
}

    CompletableFuture<Void> sendMessagesAsync(QueueClient sendClient) {
        List<HashMap<String, String>> data =
                GSON.fromJson(
                        "[" +
                                "{'name' = 'Einstein', 'firstName' = 'Albert'}," +
                                "{'name' = 'Heisenberg', 'firstName' = 'Werner'}," +
                                "{'name' = 'Curie', 'firstName' = 'Marie'}," +
                                "{'name' = 'Hawking', 'firstName' = 'Steven'}," +
                                "{'name' = 'Newton', 'firstName' = 'Isaac'}," +
                                "{'name' = 'Bohr', 'firstName' = 'Niels'}," +
                                "{'name' = 'Faraday', 'firstName' = 'Michael'}," +
                                "{'name' = 'Galilei', 'firstName' = 'Galileo'}," +
                                "{'name' = 'Kepler', 'firstName' = 'Johannes'}," +
                                "{'name' = 'Kopernikus', 'firstName' = 'Nikolaus'}" +
                                "]",
                        new TypeToken<List<HashMap<String, String>>>() {}.getType());

        List<CompletableFuture> tasks = new ArrayList<>();
        for (int i = 0; i < data.size(); i++) {
            final String messageId = Integer.toString(i);
            Message message = new Message(GSON.toJson(data.get(i), Map.class).getBytes(UTF_8));
            message.setContentType("application/json");
            message.setLabel("Scientist");
            message.setMessageId(messageId);
            message.setTimeToLive(Duration.ofMinutes(2));
            System.out.printf("\nMessage sending: Id = %s", message.getMessageId());
            tasks.add(
                    sendClient.sendAsync(message).thenRunAsync(() -> {
                        System.out.printf("\n\tMessage acknowledged: Id = %s", message.getMessageId());
                    }));
        }
        return CompletableFuture.allOf(tasks.toArray(new CompletableFuture<?>[tasks.size()]));
    }

```

发送到服务总线队列的消息，并从中接收的消息是 [Message](/java/api/com.microsoft.azure.servicebus.message?view=azure-java-stable) 类的实例。 Message 对象包含一组标准属性（如 Label 和 TimeToLive）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。 应用程序可通过将任何可序列化对象传入到 Message 的构造函数中来设置消息的正文，然后将使用适当的序列化程序来序列化对象。 或者，可提供 java.IO.InputStream 对象。


服务总线队列在[标准层](service-bus-premium-messaging.md)中支持的最大消息大小为 256 KB，在[高级层](service-bus-premium-messaging.md)中则为 1 MB。 标头最大大小为 64 KB，其中包括标准和自定义应用程序属性。 一个队列可包含的消息数不受限制，但消息的总大小受限。 此队列大小是在创建时定义的，上限为 5 GB。

## <a name="receive-messages-from-a-queue"></a>从队列接收消息
从队列接收消息的主要方法是使用 ServiceBusContract 对象。 收到的消息可在两种不同模式下工作：ReceiveAndDelete 或 PeekLock。

当使用 ReceiveAndDelete 模式时，接收是一项单次操作，即，当服务总线接收到队列中某条消息的读取请求时，它会将该消息标记为“已使用”并将其返回给应用程序。 ReceiveAndDelete 模式（默认模式）是最简单的模式，最适合应用程序可容忍出现故障时不处理消息的情景。 为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。
由于服务总线已将消息标记为“已使用”，因此当应用程序重启并重新开始使用消息时，它就漏掉了在发生故障前使用的消息。

在 PeekLock 模式下，接收变成了一个两阶段操作，从而有可能支持无法允许遗漏消息的应用程序。 当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。 应用程序完成消息处理（或可靠地存储消息以供将来处理）后，将通过对收到的消息调用 Delete 完成接收过程的第二个阶段。 发现“Delete”调用时，服务总线会将消息标记为“已使用”，并将消息从队列中删除。

以下示例演示如何使用 PeekLock 模式（非默认模式）接收和处理消息。 下面的示例将执行无限循环并在消息达到我们的 `TestQueue` 后进行处理：

```java
    public void run() throws Exception {
        // Create a QueueClient instance for receiving using the connection string builder
        // We set the receive mode to "PeekLock", meaning the message is delivered
        // under a lock and must be acknowledged ("completed") to be removed from the queue
        QueueClient receiveClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
        this.registerReceiver(receiveClient);

        // shut down receiver to close the receive loop
        receiveClient.close();
    }
    void registerReceiver(QueueClient queueClient) throws Exception {
        // register the RegisterMessageHandler callback
        queueClient.registerMessageHandler(new IMessageHandler() {
        // callback invoked when the message handler loop has obtained a message
            public CompletableFuture<Void> onMessageAsync(IMessage message) {
            // receives message is passed to callback
                if (message.getLabel() != null &&
                    message.getContentType() != null &&
                    message.getLabel().contentEquals("Scientist") &&
                    message.getContentType().contentEquals("application/json")) {

                        byte[] body = message.getBody();
                        Map scientist = GSON.fromJson(new String(body, UTF_8), Map.class);

                        System.out.printf(
                            "\n\t\t\t\tMessage received: \n\t\t\t\t\t\tMessageId = %s, \n\t\t\t\t\t\tSequenceNumber = %s, \n\t\t\t\t\t\tEnqueuedTimeUtc = %s," +
                            "\n\t\t\t\t\t\tExpiresAtUtc = %s, \n\t\t\t\t\t\tContentType = \"%s\",  \n\t\t\t\t\t\tContent: [ firstName = %s, name = %s ]\n",
                            message.getMessageId(),
                            message.getSequenceNumber(),
                            message.getEnqueuedTimeUtc(),
                            message.getExpiresAtUtc(),
                            message.getContentType(),
                            scientist != null ? scientist.get("firstName") : "",
                            scientist != null ? scientist.get("name") : "");
                    }
                    return CompletableFuture.completedFuture(null);
                }

                // callback invoked when the message handler has an exception to report
                public void notifyException(Throwable throwable, ExceptionPhase exceptionPhase) {
                    System.out.printf(exceptionPhase + "-" + throwable.getMessage());
                }
        },
        // 1 concurrent call, messages are auto-completed, auto-renew duration
        new MessageHandlerOptions(1, true, Duration.ofMinutes(1)));
    }

```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何处理应用程序崩溃和不可读消息
服务总线提供了相关功能，帮助你轻松地从应用程序错误或消息处理问题中恢复。 如果接收方应用程序出于某种原因无法处理消息，则其可以对收到的消息调用 unlockMessage 方法（而不是 deleteMessage 方法）。 这会导致服务总线解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线会自动解锁该消息并使它可再次被接收。

如果在处理消息之后，发出 deleteMessage 请求之前，应用程序发生崩溃，该消息会在应用程序重新启动时重新传送给它。 此情况通常称作至少处理一次，即每条消息至少被处理一次，但在某些情况下，同一消息可能会被重新传送。 如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。 通常可使用消息的 getMessageId 方法实现此操作，这在多个传送尝试中保持不变。

## <a name="next-steps"></a>后续步骤
了解服务总线队列的基本信息后，请参阅[队列、主题和订阅][Queues, topics, and subscriptions]以获取更多信息。

有关详细信息，请参阅 [Java 开发人员中心](https://azure.microsoft.com/develop/java/)。

[Azure SDK for Java]: https://docs.microsoft.com/java/api/overview/azure/
[Azure Toolkit for Eclipse]: https://docs.microsoft.com/java/azure/eclipse/azure-toolkit-for-eclipse
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
