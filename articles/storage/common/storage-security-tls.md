---
title: 为 Azure 存储客户端启用安全 TLS | Microsoft Docs
description: 了解如何在 Azure 存储的客户端中启用 TLS 1.2。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 06/25/2018
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: 218708ffc9a680150d7b6bf559a00ca87054bbe8
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65152970"
---
# <a name="enable-secure-tls-for-azure-storage-client"></a>为 Azure 存储客户端启用安全 TLS

传输层安全性 (TLS) 和安全套接字层 (SSL) 是提供计算机网络通信安全的加密协议。 我们已发现 SSL 1.0、2.0 和 3.0 易受攻击。 RFC 已将其禁止。 使用不安全的分组加密（DES CBC 和 RC2 CBC）和流加密 (RC4) 时，TLS 1.0 会变得不安全。 PCI 委员会还建议迁移到更高的 TLS 版本。 有关详细信息，可以参阅[传输层安全性 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0)。

Azure 存储自 2015 年以来已停止使用 SSL 3.0，并在公共 HTTPs 终结点上使用 TLS 1.2，但为了向后兼容性仍支持 TLS 1.0 和 TLS 1.1。

为了确保以安全且符合规定的方式连接到 Azure 存储，需要在发送请求以操作 Azure 存储服务之前在客户端中启用 TLS 1.2 或更高版本。

## <a name="enable-tls-12-in-net-client"></a>在 .NET 客户端中启用 TLS 1.2

为了使客户端可以通过 TLS 1.2 进行协商，操作系统和 .NET Framework 版本都需要支持 TLS 1.2。 请参阅[对 TLS 1.2 的支持](https://docs.microsoft.com/dotnet/framework/network-programming/tls#support-for-tls-12)中的更多详细信息。

下面的示例演示如何在 .NET 客户端中启用 TLS 1.2。

```csharp

    static void EnableTls12()
    {
        // Enable TLS 1.2 before connecting to Azure Storage
        System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;

        // Connect to Azure Storage
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName={yourstorageaccount};AccountKey={yourstorageaccountkey};EndpointSuffix=core.windows.net");
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        CloudBlobContainer container = blobClient.GetContainerReference("foo");
        container.CreateIfNotExists();
    }

```

## <a name="enable-tls-12-in-powershell-client"></a>在 PowerShell 客户端中启用 TLS 1.2

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)] 

下面的示例演示如何在 PowerShell 客户端中启用 TLS 1.2。

```powershell
# Enable TLS 1.2 before connecting to Azure Storage
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

$resourceGroup = "{YourResourceGroupName}"
$storageAccountName = "{YourStorageAccountNme}"
$prefix = "foo"

# Connect to Azure Storage
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName
$ctx = $storageAccount.Context
$listOfContainers = Get-AzStorageContainer -Context $ctx -Prefix $prefix
$listOfContainers
```

## <a name="verify-tls-12-connection"></a>验证 TLS 1.2 连接

Fiddler 可以用于验证是否实际使用了 TLS 1.2。 打开 Fiddler 以开始捕获客户端网络流量，然后执行上面的示例。 然后可以查找此示例建立的连接中的 TLS 版本。

下面的屏幕截图是一个验证示例。

![在 Fiddler 中验证 TLS 版本的屏幕截图](./media/storage-security-tls/storage-security-tls-verify-in-fiddler.png)

## <a name="see-also"></a>另请参阅

* [传输层安全性 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0)
* [TLS 上的 PCI 符合性](https://blog.pcisecuritystandards.org/migrating-from-ssl-and-early-tls)
* [在 Java 客户端中启用 TLS](https://www.java.com/en/configure_crypto.html)
