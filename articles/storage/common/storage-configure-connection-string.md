---
title: 配置 Azure 存储的连接字符串 | Microsoft Docs
description: 配置 Azure 存储帐户的连接字符串。 连接字符串包含在运行时从应用程序访问 Azure 存储帐户所需的授权信息。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/12/2017
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 7029f07b494630cc1ebe4a2dbfb297e73d85ec5e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153184"
---
# <a name="configure-azure-storage-connection-strings"></a>配置 Azure 存储连接字符串

连接字符串包含应用程序在运行时访问 Azure 存储帐户中数据所需的身份验证信息。 可以将连接字符串配置为：

* 连接到 Azure 存储模拟器。
* 在 Azure 中访问存储帐户。
* 通过共享访问签名 (SAS) 访问 Azure 中的指定资源。

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## <a name="storing-your-connection-string"></a>存储连接字符串
应用程序需要在运行时访问连接字符串，才能授权对 Azure 存储发出的请求。 可以通过几个选项来存储连接字符串：

* 在桌面或设备上运行的应用程序可以在 **app.config** 文件或 **web.config** 文件中存储连接字符串。 将连接字符串添加到这些文件的 **AppSettings** 部分。
* 在 Azure 云服务中运行的应用程序可以在 [Azure service configuration schema (.cscfg) file](https://msdn.microsoft.com/library/ee758710.aspx)（Azure 服务配置架构 (.cscfg) 文件）中存储连接字符串。 将连接字符串添加到服务配置文件的 **ConfigurationSettings** 部分。
* 可以直接在代码中使用连接字符串。 但是，在大部分情况下，建议在配置文件中存储连接字符串。

在一个配置文件中存储连接字符串可以轻松地更新连接字符串，从而在存储模拟器和云中的 Azure 存储帐户之间切换。 只需编辑连接字符串，使其指向目标环境。

可以使用 [Microsoft Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/) 在运行时访问连接字符串，而不考虑应用程序在何处运行。

## <a name="create-a-connection-string-for-the-storage-emulator"></a>创建存储模拟器的连接字符串
[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

有关存储模拟器的详细信息，请参阅[使用 Azure 存储模拟器进行开发和测试](storage-use-emulator.md)。

## <a name="create-a-connection-string-for-an-azure-storage-account"></a>创建 Azure 存储帐户的连接字符串
若要创建 Azure 存储帐户的连接字符串，请使用下面的格式。 指示要通过 HTTPS（建议）还是 HTTP 连接到存储帐户，将 `myAccountName` 替换为存储帐户的名称，将 `myAccountKey` 替换为帐户访问密钥：

`DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey`

例如，连接字符串可能如下所示：

`DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=<account-key>`

尽管 Azure 存储连接字符串同时支持 HTTP 和 HTTPS，但强烈建议使用 HTTPS。

> [!TIP]
> 可以在 [Azure 门户](https://portal.azure.com)中找到存储帐户的连接字符串。 导航到存储帐户菜单边栏选项卡中的“设置” > “访问密钥”，查看主访问密钥和辅助访问密钥的连接字符串。
>

## <a name="create-a-connection-string-using-a-shared-access-signature"></a>使用共享访问签名创建连接字符串
[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

## <a name="create-a-connection-string-for-an-explicit-storage-endpoint"></a>创建显式存储终结点的连接字符串
可以在连接字符串中显式指定服务终结点，而不使用默认终结点。 若要创建指定显式终结点的连接字符串，请使用以下格式为每个服务指定完整的服务终结点，包括协议规范（HTTPS（建议）或 HTTP）：

```
DefaultEndpointsProtocol=[http|https];
BlobEndpoint=myBlobEndpoint;
FileEndpoint=myFileEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
AccountName=myAccountName;
AccountKey=myAccountKey
```

如果已将 Blob 存储终结点映射到[自定义域](../blobs/storage-custom-domain-name.md)，则可能需要指定显式终结点。 在这种情况下，可以在连接字符串中指定 Blob 存储的自定义终结点。 可以选择指定其他服务的默认终结点，如果应用程序使用它们。

下面是用于指定 Blob 服务的显式终结点的连接字符串的示例：

```
# Blob endpoint only
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
AccountName=storagesample;
AccountKey=<account-key>
```

此示例指定所有服务的显式终结点，包括 Blob 服务的自定义域：

```
# All service endpoints
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
FileEndpoint=https://myaccount.file.core.windows.net;
QueueEndpoint=https://myaccount.queue.core.windows.net;
TableEndpoint=https://myaccount.table.core.windows.net;
AccountName=storagesample;
AccountKey=<account-key>
```

连接字符串中的终结点值用于构造存储服务的请求 URI，并决定返回到代码的所有 URI 的形式。

如果已将存储终结点映射到自定义域并从连接字符串中省略该终结点，将无法使用该连接字符串通过代码访问该服务中的数据。

> [!IMPORTANT]
> 连接字符串中的服务终结点值必须是格式正确的 URI，包括 `https://`（推荐）或 `http://`。 因为 Azure 存储尚不支持自定义域的 HTTPS，因此必须为指向自定义域的所有终结点 URI 指定 `http://`。
>

### <a name="create-a-connection-string-with-an-endpoint-suffix"></a>创建含终结点后缀的连接字符串
若要针对具有不同终结点后缀的地区或实例内的存储服务创建一个连接字符串，例如针对 Azure 中国或 Azure 政府，请使用以下连接字符串格式。 指出是通过 HTTPS（推荐）还是 HTTP 连接到存储帐户，将 `myAccountName` 替换为存储帐户的名称，将 `myAccountKey` 替换为帐户访问密钥，并将 `mySuffix` 替换为 URI 后缀：

```
DefaultEndpointsProtocol=[http|https];
AccountName=myAccountName;
AccountKey=myAccountKey;
EndpointSuffix=mySuffix;
```

下面是 Azure 中国中存储服务的连接字符串示例：

```
DefaultEndpointsProtocol=https;
AccountName=storagesample;
AccountKey=<account-key>;
EndpointSuffix=core.chinacloudapi.cn;
```

## <a name="parsing-a-connection-string"></a>分析连接字符串
[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

## <a name="next-steps"></a>后续步骤
* [使用 Azure 存储模拟器进行开发和测试](storage-use-emulator.md)
* [Azure 存储资源管理器](storage-explorers.md)
* [使用共享访问签名 (SAS)](storage-dotnet-shared-access-signature-part-1.md)

