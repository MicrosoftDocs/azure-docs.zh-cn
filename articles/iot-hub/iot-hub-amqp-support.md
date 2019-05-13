---
title: 了解 Azure IoT 中心的 AMQP 支持 |Microsoft Docs
description: 开发人员指南-支持设备连接到 IoT 中心使用 AMQP 协议的面向设备和面向服务的终结点。 包括的内置的 Azure IoT 设备 Sdk 中的 AMQP 支持有关的信息。
author: rezasherafat
manager: ''
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: rezas
ms.openlocfilehash: 703e2c842fb42bad8aa112d84c516a29c2327378
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473501"
---
# <a name="communicate-with-your-iot-hub-using-the-amqp-protocol"></a>与 IoT 中心使用 AMQP 协议进行通信

IoT 中心支持[AMQP 1.0 版](http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf)提供各种功能通过面向设备和面向服务的终结点。 本文档介绍如何使用 AMQP 客户端连接到 IoT 中心，以便使用 IoT 中心功能。

## <a name="service-client"></a>服务客户端

### <a name="connection-and-authenticating-to-iot-hub-service-client"></a>连接和对 IoT 中心 （服务客户端） 进行身份验证
若要连接到 IoT 中心使用 AMQP 时，客户端可以使用[基于声明的安全 (CBS)](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc)或[简单身份验证和安全层 (SASL) 身份验证](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer)。

以下信息是必需的服务客户端：

| 信息 | 值 | 
|-------------|--------------|
| IoT 中心主机名 | `<iot-hub-name>.azure-devices.net` |
| 密钥名称 | `service` |
| 访问密钥 | 与服务关联的主密钥或辅助密钥 |
| 共享访问签名 | 短期 SAS 采用以下格式： `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}` (可以找到用于生成此签名的代码[此处](./iot-hub-devguide-security.md#security-token-structure))。


使用以下代码段[uAMQP 库在 Python 中的](https://github.com/Azure/azure-uamqp-python)连接到 IoT 中心通过在发送方链接。

```python
import uamqp
import urllib
import time

# Use generate_sas_token implementation available here: https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '<operation-link-name>' # e.g., '/messages/devicebound'

username = '{policy_name}@sas.root.{iot_hub_name}'.format(iot_hub_name=iot_hub_name, policy_name=policy_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username), urllib.quote_plus(sas_token), hostname, operation)

# Create a send or receive client
send_client = uamqp.SendClient(uri, debug=True)
receive_client = uamqp.ReceiveClient(uri, debug=True)
```

### <a name="invoking-cloud-to-device-messages-service-client"></a>调用云到设备消息 （服务客户端）
服务和 IoT 中心之间以及设备和 IoT 中心之间的云到设备消息交换所述[此处](iot-hub-devguide-messages-c2d.md)。 服务客户端使用以将消息发送和接收来自设备的先前所发送消息的反馈如下所述的两个链接。

| 创建者 | 链接类型 | 链接路径 | 描述 |
|------------|-----------|-----------|-------------|
| 服务 | 在发送方链接 | `/messages/devicebound` | C2D 消息要发送到设备发送到此链接服务。 通过此链接发送的消息具有其`To`属性设置为目标设备的接收方链接路径： 即`/devices/<deviceID>/messages/devicebound`。 |
| 服务 | 接收方链接 | `/messages/serviceBound/feedback` | 完成、 拒绝和用户反馈消息来自接收此链接的服务的设备。 请参阅[此处](./iot-hub-devguide-messages-c2d.md#message-feedback)有关反馈消息的详细信息。 |

以下代码段演示如何创建 C2D 消息并将其发送到设备使用[uAMQP 库在 Python 中的](https://github.com/Azure/azure-uamqp-python)。

```python
import uuid
# Create a message and set message property 'To' to the devicebound link on device
msg_id = str(uuid.uuid4())
msg_content = b"Message content goes here!"
device_id = '<device-id>'
to = '/devices/{device_id}/messages/devicebound'.format(device_id=device_id)
ack = 'full' # Alternative values are 'positive', 'negative', and 'none'
app_props = { 'iothub-ack': ack }
msg_props = uamqp.message.MessageProperties(message_id=msg_id, to=to)
msg = uamqp.Message(msg_content, properties=msg_props, application_properties=app_props)

# Send the message using the send client created and connected IoT Hub earlier
send_client.queue_message(msg)
results = send_client.send_all_messages()

# Close the client if not needed
send_client.close()
```

若要接收反馈，服务客户端创建的接收方链接。 以下代码段演示如何执行此操作使用[uAMQP 库在 Python 中的](https://github.com/Azure/azure-uamqp-python)。

```python
import json

operation = '/messages/serviceBound/feedback'

# ...
# Recreate the URI using the feedback path above and authenticate
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username), urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
batch = receive_client.receive_message_batch(max_batch_size=10)
for msg in batch:
  print('received a message')
  # Check content_type in message property to identify feedback messages coming from device
  if msg.properties.content_type == 'application/vnd.microsoft.iothub.feedback.json':
    msg_body_raw = msg.get_data()
    msg_body_str = ''.join(msg_body_raw)
    msg_body = json.loads(msg_body_str)
    print(json.dumps(msg_body, indent=2))
    print('******************')
    for feedback in msg_body:
      print('feedback received')
      print('\tstatusCode: ' + str(feedback['statusCode']))
      print('\toriginalMessageId: ' + str(feedback['originalMessageId']))
      print('\tdeviceId: ' + str(feedback['deviceId']))
      print
  else:
    print('unknown message:', msg.properties.content_type)
```

C2D 反馈消息如上所示，具有内容类型的`application/vnd.microsoft.iothub.feedback.json`和其 JSON 正文中的属性可用于推断出原始消息的传递状态：
* 键`statusCode`反馈正文具有这些值之一： `['Success', 'Expired', 'DeliveryCountExceeded', 'Rejected', 'Purged']`。
* 密钥`deviceId`反馈正文具有目标设备的 ID。
* 密钥`originalMessageId`反馈正文具有由服务发送原始 C2D 消息的 ID。 这可以用于关联到 C2D 消息的反馈。

### <a name="receive-telemetry-messages-service-client"></a>接收遥测消息 （服务客户端）


### <a name="additional-notes"></a>其他注释
* AMQP 连接可能会突然中断由于网络故障或 （在代码中生成） 的身份验证令牌的过期日期。 服务客户端必须处理这些情况下并重新建立连接和必要的链接。 身份验证令牌过期时间的情况下，客户端还主动续订以避免连接删除其到期前的标记。
* 在某些情况下，您的客户端必须能够正确处理链接重定向。 请参阅有关如何执行此操作在 AMQP 客户端文档。

### <a name="receive-cloud-to-device-messages-device-and-module-client"></a>接收云到设备消息 （设备和模块的客户端）
在设备端上使用 AMQP 链接如下所示：

| 创建者 | 链接类型 | 链接路径 | 描述 |
|------------|-----------|-----------|-------------|
| 设备 | 接收方链接 | `/devices/<deviceID>/messages/devicebound` | 此链接每个目标设备接收 C2D 消息要发送到设备。 |
| 设备 | 在发送方链接 | `/messages/serviceBound/feedback` | C2D 消息反馈通过设备通过此链接发送到服务。 |
| 模块 | 接收方链接 | `/devices/<deviceID>/modules/<moduleID>/messages/devicebound` | C2D 消息要发送到模块由每个目标模块接收此链接。 |
| 模块 | 在发送方链接 | `/messages/serviceBound/feedback` | C2D 消息反馈模块通过此链接发送到服务。 |


## <a name="next-steps"></a>后续步骤

若要了解有关 AMQP 协议的详细信息，请参阅[AMQP 1.0 版规范](http://www.amqp.org/sites/amqp.org/files/amqp.pdf)。

若要了解有关 IoT 中心消息传送的详细信息，请参阅：

* [云到设备的消息](./iot-hub-devguide-messages-c2d.md)
* [支持其他协议](iot-hub-protocol-gateway.md)
* [对于 MQTT 协议的支持](./iot-hub-mqtt-support.md)
