---
title: 针对 Microsoft Azure 存储使用 Python 的客户端加密 | Microsoft Docs
description: 适用于 Python 的 Azure 存储客户端库支持客户端加密，实现 Azure 存储应用程序的最高安全性。
services: storage
author: tamram
ms.service: storage
ms.devlang: python
ms.topic: article
ms.date: 05/11/2017
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: d04c1e137a190b01554106c041853aa2fd6786d7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65146901"
---
# <a name="client-side-encryption-with-python-for-microsoft-azure-storage"></a>针对 Microsoft Azure 存储使用 Python 的客户端加密
[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>概述
[用于 Python 的 Azure 存储客户端库](https://pypi.python.org/pypi/azure-storage)支持在上传到 Azure 存储之前加密客户端应用程序中的数据，并在下载到客户端时解密数据。

> [!NOTE]
> Azure 存储 Python 库目前以预览版提供。
> 
> 

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>通过信封技术加密和解密
加密和解密的过程遵循信封技术。

### <a name="encryption-via-the-envelope-technique"></a>通过信封技术加密
通过信封技术加密的工作方式如下：

1. Azure 存储客户端库生成内容加密密钥 (CEK)，这是一次性使用对称密钥。
2. 使用此 CEK 对用户数据进行加密。
3. 然后，使用密钥加密密钥 (KEK) 对此 CEK 进行包装（加密）。 KEK 由密钥标识符标识，可以是本地管理的非对称密钥对或对称密钥。
   存储客户端库本身永远无法访问 KEK。 该库调用 KEK 提供的密钥包装算法。 用户可以根据需要选择使用自定义提供程序进行密钥包装/解包。
4. 然后，将已加密的数据上传到 Azure 存储服务。 已包装的密钥以及一些附加加密元数据要么存储为元数据（在 Blob 上），要么以内插值替换已加密的数据（消息队列和表实体）。

### <a name="decryption-via-the-envelope-technique"></a>通过信封技术解密
通过信封技术解密的工作方式如下：

1. 客户端库假定用户在本地管理密钥加密密钥 (KEK)。 用户不需要知道用于加密的特定密钥。 但是，可以设置和使用一个密钥解析程序，将不同的密钥标识符解析为密钥。
2. 客户端库下载存储在服务中的已加密数据以及任何加密材料。
3. 然后，使用密钥加密密钥 (KEK) 对已包装的内容加密密钥 (CEK) 进行解包（解密）。 这里同样，客户端库无法访问 KEK。 它只是调用自定义提供程序的解包算法。
4. 然后，使用内容加密密钥 (CEK) 解密已加密的用户数据。

## <a name="encryption-mechanism"></a>加密机制
存储客户端库使用 [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 来加密用户数据。 具体而言，是使用 AES 的[加密块链接 (CBC)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) 模式。 每个服务的工作方式都稍有不同，因此我们会在此讨论其中每个服务。

### <a name="blobs"></a>Blob
目前，客户端库仅支持整个 Blob 的加密。 具体而言，用户使用 **create*** 方法时支持加密。 对于下载，支持完整下载和范围下载，并且可以并行化上传和下载。

在加密过程中，客户端库将生成 16 字节的随机初始化向量 (IV) 和 32 字节的随机内容加密密钥 (CEK) 并将使用此信息对 Blob 数据执行信封加密。 然后，已包装的 CEK 和一些附加加密元数据将与服务上的已加密 Blob 一起存储为 Blob 元数据。

> [!WARNING]
> 如果要针对 Blob 编辑或上传自己的元数据，需要确保此元数据已保留。 如果在没有此元数据的情况下上传新元数据，则已包装的 CEK、IV 和其他元数据将丢失，而 Blob 内容将永远无法再检索。
> 
> 

下载已加密的 blob 需要使用 **get*** 便捷方法检索整个 blob 的内容。 将已包装的 CEK 解包，与 IV（在本示例中存储为 Blob 元数据）一起使用将解密后的数据返回给用户。

下载已加密 blob 中的任意范围（传入了范围参数的 **get*** 方法）需要调整用户提供的范围以获取少量可用于成功解密所请求范围的附加数据。

块 Blob 和页 Blob 只能使用此方案进行加密/解密。 目前不支持加密追加 Blob。

### <a name="queues"></a>队列
由于队列消息可以采用任何格式，客户端库定义一个自定义格式，其在消息文本中包括初始化向量 (IV) 和已加密的内容加密密钥 (CEK)。

在加密过程中，客户端库将生成 16 字节的随机 IV 和 32 字节的随机 CEK，并使用此信息对队列消息文本执行信封加密。 然后，将已包装的 CEK 和一些附加加密元数据添加到已加密的队列消息中。 此修改后的消息（如下所示）将存储在服务中。

```
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

在解密过程中，将从队列消息中提取已包装的密钥并将其解包。 还将从队列消息中提取 IV，与解包的密钥一起使用来对队列消息数据进行解密。 请注意，加密元数据很少（500 字节以下），因此虽然它计入队列消息的 64KB 限制，但影响应是可管理的。

### <a name="tables"></a>表
客户端库支持对插入和替换操作的实体属性进行加密。

> [!NOTE]
> 当前不支持合并。 由于属性的子集可能以前已使用不同的密钥加密，因此只合并新属性和更新元数据将导致数据丢失。 合并需要进行额外的服务调用以从服务中读取预先存在的实体，或者需要为属性使用一个新密钥，由于性能方面的原因，这两种方案都不适用。
> 
> 

表数据加密的工作方式如下：

1. 用户指定要加密的属性。
2. 客户端库为每个实体生成 16 字节的随机初始化向量 (IV) 和 32 字节的随机内容加密密钥 (CEK)，并通过为每个属性派生新的 IV 来对要加密的单个属性执行信封加密。 加密的属性存储为二进制数据。
3. 然后，已包装的 CEK 和一些附加加密元数据将存储为两个附加保留属性。 第一个保留属性 (\_ClientEncryptionMetadata1) 是一个字符串属性，保存有关 IV、版本和已包装密钥的信息。 第二个保留属性 (\_ClientEncryptionMetadata2) 是一个二进制属性，保存有关已加密属性的信息。 第二个属性 (\_ClientEncryptionMetadata2) 中的信息本身是加密的。
4. 由于加密需要这两个附加保留属性，用户现在可能只有 250 个自定义属性，而不是 252 个。 实体的总大小必须小于 1MB。

   请注意，只有字符串属性可以加密。 如果要对其他类型的属性进行加密，必须将它们转换为字符串。 加密的字符串作为二进制属性存储在服务中，并在解密之后转换回字符串（原始字符串，不是 EdmType.STRING 类型的 EntityProperties）。

   对于表，除了加密策略以外，用户还必须指定要加密的属性。 为此，可将这些属性存储在 type 设置为 EdmType.STRING 且 encrypt 设置为 true 的 TableEntity 对象中，或者在 tableservice 对象中设置 encryption_resolver_function。 加密解析程序是一个函数，它接受分区键、行键和属性名称并返回一个布尔值以指示是否应加密该属性。 在加密过程中，客户端库将使用此信息来确定是否应在写入到网络时加密属性。 该委托还可以围绕如何加密属性来实现逻辑的可能性。 （例如，如果 X，则加密属性 A，否则加密属性 A 和 B。）请注意，在读取或查询实体时，不需要提供此信息。

### <a name="batch-operations"></a>批处理操作
一个加密策略将应用到批中的所有行。 客户端库将为批中的每行在内部生成一个新的随机 IV 和随机 CEK。 用户还可以选择通过在加密解析程序中定义此行为来加密批中的每个操作的不同属性。
如果某个批是通过 tableservice batch() 方法以上下文管理器形式创建的，则 tableservice 的加密策略会自动应用到该批。 如果某个批是通过调用构造函数显式创建的，则必须将加密策略作为参数来传递，并且在该批的生存期内都不要修改加密策略。
请注意，使用批的加密策略将实例插入批时，会将实体加密（使用 tableservice 的加密策略提交批时不会加密实体）。

### <a name="queries"></a>查询
> [!NOTE]
> 由于实体已加密，因此不能运行根据已加密属性进行筛选的查询。  如果尝试运行，结果将会不正确，因为该服务会尝试将已加密的数据与未加密的数据进行比较。
> 
> 
> 若要执行查询操作，必须指定一个能够解析结果集中的所有密钥的密钥解析程序。 如果查询结果中包含的实体不能解析为提供程序，则客户端库将引发错误。 对于执行服务器端投影的任何查询，在默认情况下，客户端库将为所选列添加特殊的加密元数据属性（\_ClientEncryptionMetadata1 和 \_ClientEncryptionMetadata2）。
> 
> [!IMPORTANT]
> 使用客户端加密时，请注意以下要点：
> 
> * 读取或写入到已加密的 Blob 时，请使用完整 Blob 上传命令和范围/完整 Blob 下载命令。 避免使用协议操作（如“放置块”、“放置块列表”、“写入页”或“清除页”）写入到已加密的 Blob，否则可能会损坏已加密的 Blob 并使其不可读。
> * 对于表，存在类似的约束。 请注意，不要在未更新加密元数据的情况下更新已加密的属性。
> * 如果在已加密的 Blob 上设置元数据，则可能会覆盖解密所需的与加密相关的元数据，因为设置元数据不是累加性的。 这也适用于快照；避免在创建已加密的 Blob 的快照时指定元数据。 如果必须设置元数据，务必调用 **get_blob_metadata** 方法首先获取当前加密元数据，并在设置元数据时避免并发写入。
> * 对于只处理加密数据的用户，请在服务对象中启用 **require_encryption** 标志。 有关详细信息，请参阅下文。

存储客户端库要求提供的 KEK 和密钥解析程序实现以下接口。 用于 Python KEK 管理的 [Azure 密钥保管库](https://azure.microsoft.com/services/key-vault/)支持正在筹备中，开发完成后会集成到此库中。

## <a name="client-api--interface"></a>客户端 API/接口
创建存储服务对象（例如 blockblobservice）后，用户可以向构成加密策略的字段赋值：key_encryption_key、key_resolver_function 和 require_encryption。 用户可仅提供 KEK 或解析程序，或同时提供两者。 key_encryption_key 是使用密钥标识符进行标识的基本密钥类型，它提供包装/解包逻辑。 key_resolver_function 用于在解密过程中解析密钥。 在指定了密钥标识符的情况下，它将返回有效的 KEK。 由此，用户能够在多个位置中托管的多个密钥之间进行选择。

KEK 必须实现以下方法才能成功加密数据：

* wrap_key(cek)：使用用户所选的算法包装指定的 CEK（字节）。 返回包装的密钥。
* get_key_wrap_algorithm()：返回用于包装密钥的算法。
* get_kid()：返回此 KEK 的字符串密钥 ID。
  KEK 必须实现以下方法才能成功解密数据：
* unwrap_key(cek, algorithm)：使用字符串指定的算法返回指定 CEK 的解包形式。
* get_kid()：返回此 KEK 的字符串密钥 ID。

密钥解析程序必须至少实现一个方法，以便在指定密钥 ID 的情况下，返回用于实现上述接口的相应 KEK。 只会将此方法分配到服务对象中的 key_resolver_function 属性。

* 对于加密，始终使用该密钥，而没有密钥将导致错误。
* 对于解密：

  * 如果指定为获取密钥，则将调用密钥解析程序。 如果指定了解析程序，但该解析程序不具有密钥标识符的映射，则将引发错误。
  * 如果未指定解析程序，但指定了密钥，则在该密钥的标识符与所需密钥标识符匹配时使用该密钥。 如果标识符不匹配，则将引发错误。

    Azure.storage.samples 中的加密示例演示了针对 blob、 队列和表的更详细的端到端方案。
      KEK 和密钥解析程序的示例实现在示例文件中分别以 KeyWrapper 和 KeyResolver 提供。

### <a name="requireencryption-mode"></a>RequireEncryption 模式
用户可以选择启用一个操作模式，让所有上传和下载都必须加密。 在此模式下，尝试在没有加密策略的情况下上传数据或下载在服务中未加密的数据，将导致在客户端上失败。 服务对象中的 **require_encryption** 标志控制此行为。

### <a name="blob-service-encryption"></a>Blob 服务加密
设置 blockblobservice 对象中的加密策略字段。 其他所有事项均由客户端库在内部处理。

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_block_blob_service.key_encryption_key = kek
my_block_blob_service.key_resolver_funcion = key_resolver.resolve_key

# Upload the encrypted contents to the blob.
my_block_blob_service.create_blob_from_stream(container_name, blob_name, stream)

# Download and decrypt the encrypted contents from the blob.
blob = my_block_blob_service.get_blob_to_bytes(container_name, blob_name)
```

### <a name="queue-service-encryption"></a>队列服务加密
设置 queueservice 对象中的加密策略字段。 其他所有事项均由客户端库在内部处理。

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_queue_service.key_encryption_key = kek
my_queue_service.key_resolver_funcion = key_resolver.resolve_key

# Add message
my_queue_service.put_message(queue_name, content)

# Retrieve message
retrieved_message_list = my_queue_service.get_messages(queue_name)
```

### <a name="table-service-encryption"></a>表服务加密
除了创建加密策略并在请求选项中设置它以外，还必须在 **tableservice** 中指定 **encryption_resolver_function**，或者在 EntityProperty 中设置 encrypt 属性。

### <a name="using-the-resolver"></a>使用解析程序

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1') # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Define the encryption resolver_function.
def my_encryption_resolver(pk, rk, property_name):
    if property_name == 'foo':
        return True
    return False

# Set the KEK and key resolver on the service object.
my_table_service.key_encryption_key = kek
my_table_service.key_resolver_funcion = key_resolver.resolve_key
my_table_service.encryption_resolver_function = my_encryption_resolver

# Insert Entity
my_table_service.insert_entity(table_name, entity)

# Retrieve Entity
# Note: No need to specify an encryption resolver for retrieve, but it is harmless to leave the property set.
my_table_service.get_entity(table_name, entity['PartitionKey'], entity['RowKey'])
```

### <a name="using-attributes"></a>使用属性
如上所述，可能通过将某个属性存储在 EntityProperty 对象中并设置 encrypt 字段，将该属性标记为进行加密。

```python
encrypted_property_1 = EntityProperty(EdmType.STRING, value, encrypt=True)
```

## <a name="encryption-and-performance"></a>加密和性能
注意，加密存储数据会导致额外的性能开销。 必须生成内容密钥和 IV，内容本身必须进行加密，并且其他元数据必须进行格式化并上传。 此开销将因所加密的数据量而有所变化。 我们建议客户在开发过程中始终测试其应用程序的性能。

## <a name="next-steps"></a>后续步骤
* 下载[适用于 Java 的 Azure 存储客户端库 PyPi 包](https://pypi.python.org/pypi/azure-storage)
* 从 GitHub 下载[适用于 Python 的 Azure 存储客户端库源代码](https://github.com/Azure/azure-storage-python)
