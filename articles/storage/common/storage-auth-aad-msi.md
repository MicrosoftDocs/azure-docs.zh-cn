---
title: Azure 资源的 Azure 存储 blob 和队列管理的标识的访问进行身份验证 |Microsoft Docs
description: Azure Blob 和队列存储支持 Azure 资源管理的标识与 Azure Active Directory 身份验证。 可以使用 Azure 资源的托管标识在应用程序中验证对 Blob 和队列的访问权限，此类应用程序可以运行在 Azure 虚拟机、函数应用、虚拟机规模集等位置中。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 4245c44ceaf907512187d7db4a9d6f087a855f70
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507893"
---
# <a name="authenticate-access-to-blobs-and-queues-with-azure-active-directory-and-managed-identities-for-azure-resources"></a>对 blob 和队列与 Azure Active Directory 和管理的标识访问 Azure 资源进行身份验证

Azure Blob 和队列存储支持使用 [Azure 资源的托管标识](../../active-directory/managed-identities-azure-resources/overview.md)进行 Azure Active Directory (Azure AD) 身份验证。 有关 Azure 资源可以授权访问 blob 和队列数据使用 Azure AD 凭据从 Azure 虚拟机 (Vm)、 函数应用、 虚拟机规模集和其他服务中运行的应用程序托管标识。 通过使用与 Azure AD 身份验证的 Azure 资源管理的标识，可以避免存储在云中运行应用程序使用的凭据。  

本文介绍如何从 Azure VM 托管标识的 blob 或队列数据授予访问权限。 

## <a name="enable-managed-identities-on-a-vm"></a>在 VM 上启用托管标识

使用 Azure 资源的管理的标识来授予对 blob 和队列从 VM 的访问权限之前，您必须首先在 VM 上的 Azure 资源的启用管理的标识。 若要了解如何为 Azure 资源启用托管标识，请参阅下述文章之一：

- [Azure 门户](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Azure 资源管理器模板](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Azure SDK](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-an-azure-ad-managed-identity"></a>授予对托管的 Azure AD 标识的权限

若要从托管在 Azure 存储应用程序中标识授予对 Blob 或队列服务的请求，首先配置基于角色的访问控制 (RBAC) 设置为该托管标识。 Azure 存储定义 RBAC 角色包含 blob 和队列数据的权限。 如果 RBAC 角色分配给托管标识，托管的标识授予在相应的范围的 blob 或队列数据的这些权限。 

有关分配 RBAC 角色的详细信息，请参阅以下文章之一：

- [授予对 Azure blob 和队列数据使用 RBAC 在 Azure 门户中访问权限](storage-auth-aad-rbac-portal.md)
- [授予对 Azure blob 和队列数据与使用 Azure CLI 的 RBAC 访问权限](storage-auth-aad-rbac-cli.md)
- [授予对 Azure blob 和队列数据与使用 PowerShell 的 RBAC 访问权限](storage-auth-aad-rbac-powershell.md)

## <a name="authorize-with-a-managed-identity-access-token"></a>使用托管的标识访问令牌授权

若要授权对托管标识的 Blob 和队列存储的请求，应用程序或脚本必须获取 OAuth 令牌。 [Microsoft Azure 应用程序身份验证](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication).NET （预览版） 客户端库可简化获取和续订你的代码中的某个标记的过程。

应用身份验证客户端库会自动管理身份验证。 库将使用开发人员的凭据进行身份验证在本地开发过程。 在本地开发期间使用开发人员凭据更安全，因为不需创建 Azure AD 凭据，或者不需在开发人员之间共享凭据。 当解决方案更高版本部署到 Azure 时，库会自动切换到使用应用程序凭据。

若要使用 Azure 存储应用程序中的应用程序身份验证库，请安装最新的预览包从[Nuget](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication)，以及最新版本[适用于.NET 的 Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)。 添加以下**使用**语句至代码：

```csharp
using Microsoft.Azure.Services.AppAuthentication;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.Auth;
```

应用身份验证库提供了**AzureServiceTokenProvider**类。 此类的实例可以传递给回调获取令牌，然后在过期之前续订令牌。

下面的示例获取令牌并使用它来创建新的 blob，然后使用相同的令牌来读取 blob。

```csharp
const string blobName = "https://storagesamples.blob.core.windows.net/sample-container/blob1.txt";

// Get the initial access token and the interval at which to refresh it.
AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider();
var tokenAndFrequency = TokenRenewerAsync(azureServiceTokenProvider, 
                                            CancellationToken.None).GetAwaiter().GetResult();

// Create storage credentials using the initial token, and connect the callback function 
// to renew the token just before it expires
TokenCredential tokenCredential = new TokenCredential(tokenAndFrequency.Token, 
                                                        TokenRenewerAsync,
                                                        azureServiceTokenProvider, 
                                                        tokenAndFrequency.Frequency.Value);

StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

// Create a blob using the storage credentials.
CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), 
                                            storageCredentials);

// Upload text to the blob.
blob.UploadTextAsync(string.Format("This is a blob named {0}", blob.Name));

// Continue to make requests against Azure Storage. 
// The token is automatically refreshed as needed in the background.
do
{
    // Read blob contents
    Console.WriteLine("Time accessed: {0} Blob Content: {1}", 
                        DateTimeOffset.UtcNow, 
                        blob.DownloadTextAsync().Result);

    // Sleep for ten seconds, then read the contents of the blob again.
    Thread.Sleep(TimeSpan.FromSeconds(10));
} while (true);
```

回调方法检查令牌的过期时间，并根据需要续订它：

```csharp
private static async Task<NewTokenAndFrequency> TokenRenewerAsync(Object state, CancellationToken cancellationToken)
{
    // Specify the resource ID for requesting Azure AD tokens for Azure Storage.
    const string StorageResource = "https://storage.azure.com/";  

    // Use the same token provider to request a new token.
    var authResult = await ((AzureServiceTokenProvider)state).GetAuthenticationResultAsync(StorageResource);

    // Renew the token 5 minutes before it expires.
    var next = (authResult.ExpiresOn - DateTimeOffset.UtcNow) - TimeSpan.FromMinutes(5);
    if (next.Ticks < 0)
    {
        next = default(TimeSpan);
        Console.WriteLine("Renewing token...");
    }

    // Return the new token and the next refresh time.
    return new NewTokenAndFrequency(authResult.AccessToken, next);
}
```

有关应用身份验证库的详细信息，请参阅[对 Azure 密钥保管库中使用.NET 进行服务到服务身份验证](../../key-vault/service-to-service-authentication.md)。 

若要了解有关如何获取访问令牌的详细信息，请参阅[如何使用 Azure VM 上的 Azure 资源管理的标识获取访问令牌](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)。

> [!NOTE]
> 若要授权向 blob 或队列数据与 Azure AD 发出的请求，必须为这些请求使用 HTTPS。

## <a name="next-steps"></a>后续步骤

- 若要详细了解 Azure 存储中的 RBAC 角色，请参阅[使用 RBAC 管理存储数据的访问权限](storage-auth-aad-rbac.md)。
- 若要了解如何从存储应用程序内授予容器和队列访问权限，请参阅[将 Azure AD 与存储应用程序配合使用](storage-auth-aad-app.md)。
- 若要了解如何使用 Azure AD 凭据运行 Azure CLI 和 PowerShell 命令，请参阅[运行 Azure CLI 或 PowerShell 命令使用 Azure AD 凭据来访问 blob 或队列数据](storage-auth-aad-script.md)。