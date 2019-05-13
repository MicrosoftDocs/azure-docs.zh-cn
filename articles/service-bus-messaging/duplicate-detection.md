---
title: Azure 服务总线重复消息检测 | Microsoft Docs
description: 检测重复的服务总线消息
services: service-bus-messaging
documentationcenter: ''
author: axisc
manager: timlt
editor: spelluru
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2019
ms.author: aschhab
ms.openlocfilehash: d9f814a49924ca95078f3b3decca4f3922c74c2b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413665"
---
# <a name="duplicate-detection"></a>重复检测

如果应用程序在发送消息后立即由于致命错误而失败，并且重启的应用程序实例错误地认为先前的消息传递未成功，那么后续发送会导致同一消息在系统中出现两次。

也有可能更早在客户端或网络一级出错，导致已发送的消息被提交到队列，可以确认的是消息没有成功返回到客户端。 在这种情况下，客户端就会对发送操作结果产生怀疑。

重复检测支持发送程序重新发送相同的消息，并让队列或主题放弃任何重复的副本，从而消除了这些情况下的各种怀疑。

启用重复检测，有助于跟踪在指定时间范围内发送到队列或主题的所有消息的 MessageId（由应用程序控制）。 如果使用已在相应时间范围内记录的 MessageId 发送任何新消息，则将该消息报告为“已接受”（即发送操作成功），但将立即忽略和删除新发送的消息。 除了 MessageId 之外，不会检查消息的其他任何部分。

应用程序控制的此标识符至关重要，因为只有它才能让应用程序将 MessageId 绑定到业务流程上下文，从中可以在发生故障时预见性地重新构造消息。

如果业务流程是在处理某应用程序上下文的过程中发送多个消息，MessageId 可能会复合应用程序级上下文标识符，如采购订单号和消息主题；例如，12345.2017/payment。

虽然 MessageId 可以始终是某 GUID，但将标识符绑定到业务流程可以预测重复消息，这更有利于有效使用重复检测功能。

> [!NOTE]
> 如果已启用重复侦测，未设置会话 ID 或分区键的消息 ID 用作分区键。 如果还未设置消息 ID，.NET 和 AMQP 库自动生成的消息的消息 ID。 有关详细信息，请参阅[使用的分区密钥](service-bus-partitioning.md#use-of-partition-keys)。

## <a name="enable-duplicate-detection"></a>启用重复检测

在门户中，此功能在实体创建期间处于启用状态（即“启用重复检测”复选框已选中）。不过，默认情况下，此功能处于禁用状态。 用于新建主题的设置是等效的。

![][1]

> [!IMPORTANT]
> 创建队列后，无法启用/禁用重复检测。 只能在创建队列时执行此操作。 

通过编程方式，可以结合使用 [QueueDescription.requiresDuplicateDetection](/dotnet/api/microsoft.servicebus.messaging.queuedescription.requiresduplicatedetection#Microsoft_ServiceBus_Messaging_QueueDescription_RequiresDuplicateDetection) 属性和整个 .NET Framework API，从而设置标志。 借助 Azure 资源管理器 API，可以使用 [queueProperties.requiresDuplicateDetection](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) 属性设置此值。

队列和主题的重复检测时间历史记录默认为 30 秒，最大值不得超过 7 天。 可以在 Azure 门户的队列和主题属性窗口中更改此设置。

![][2]

通过编程方式，可以结合使用 [QueueDescription.DuplicateDetectionHistoryTimeWindow](/dotnet/api/microsoft.servicebus.messaging.queuedescription.duplicatedetectionhistorytimewindow#Microsoft_ServiceBus_Messaging_QueueDescription_DuplicateDetectionHistoryTimeWindow) 和整个 .NET Framework API，配置重复检测窗口大小，即消息 ID 的保留时长。 借助 Azure 资源管理器 API，可以使用 [queueProperties.duplicateDetectionHistoryTimeWindow](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) 属性设置此值。

启用重复检测和窗口大小直接影响队列（和主题）的吞吐量，因为必须将记录的所有消息 ID 都与新提交的消息标识符进行匹配。

一直保持较小的窗口意味着，必须保留和匹配的消息 ID 变少了，吞吐量受到的影响也会降低。 对于需要启用重复检测的高吞吐量实体，设置的窗口应尽可能小。

## <a name="next-steps"></a>后续步骤

若要了解有关服务总线消息传送的详细信息，请参阅以下主题：

* [服务总线队列、主题和订阅](service-bus-queues-topics-subscriptions.md)
* [服务总线队列入门](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服务总线主题和订阅](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/duplicate-detection/create-queue.png
[2]: ./media/duplicate-detection/queue-prop.png
