---
title: 使用 Azure AD 凭据来访问 blob 或队列数据运行 Azure CLI 或 PowerShell 命令 |Microsoft Docs
description: Azure CLI 和 PowerShell 支持使用 Azure AD 凭据登录 Azure 存储 blob 和队列数据上运行命令。 针对该会话提供了一个访问令牌，该访问令牌用于授权调用操作。 权限取决于分配给 Azure AD 安全主体的 RBAC 角色。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/19/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 41ca1c5f413e5e15691f336d203edb918f21dc1a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65147292"
---
# <a name="run-azure-cli-or-powershell-commands-with-azure-ad-credentials-to-access-blob-or-queue-data"></a>使用 Azure AD 凭据来访问 blob 或队列数据运行 Azure CLI 或 PowerShell 命令

Azure 存储提供的 Azure CLI 和 PowerShell，您可以登录并使用 Azure Active Directory (Azure AD) 凭据运行脚本命令扩展。 当你登录到 Azure CLI 或 PowerShell 与 Azure AD 凭据时，则返回 OAuth 2.0 访问令牌。 该令牌自动使用通过 CLI 或 PowerShell 来授权对 Blob 或队列存储执行的后续数据操作。 对于支持的操作，无需再通过命令传递帐户密钥或 SAS 令牌。

可以将权限分配给 blob 和队列数据到 Azure AD 安全主体通过基于角色的访问控制 (RBAC)。 有关 Azure 存储中 RBAC 角色的详细信息，请参阅[通过 RBAC 管理 Azure 存储数据访问权限](storage-auth-aad-rbac.md)。

## <a name="supported-operations"></a>支持的操作

扩展支持针对容器和队列的操作。 可以调用哪些操作取决于与你登录到 Azure CLI 或 PowerShell 的 Azure AD 安全主体授予的权限。 Azure 存储容器或队列的权限通过基于角色的访问控制 (RBAC) 进行分配。 例如，如果你已被分配**Blob 数据读取器**角色，则你可以运行从容器或队列读取数据的脚本命令。 如果你被分配**Blob 数据参与者**角色，则你可以运行脚本命令的读取、 写入或删除容器或队列或所包含的数据。 

若要详细了解针对容器或队列的每个 Azure 存储操作所需的权限，请参阅[用于调用 REST 操作的权限](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-azure-active-directory#permissions-for-calling-rest-operations)。  

## <a name="call-cli-commands-using-azure-ad-credentials"></a>使用 Azure AD 凭据调用 CLI 命令

Azure CLI 支持`--auth-mode`blob 和队列数据操作的参数：

- 将 `--auth-mode` 参数设置为 `login`，以使用 Azure AD 安全主体登录。
- 将 `--auth-mode` 参数设置为旧的 `key` 值，以便在未提供帐户的身份验证参数时，查询帐户密钥。 

以下示例演示如何在 Azure CLI 中使用 Azure AD 凭据，在新的存储帐户中创建一个容器。 请务必将尖括号中的占位符值替换为你自己的值： 

1. 确保已安装 Azure CLI 2.0.46 或更高版本。 运行 `az --version` 以查看已安装版本。

1. 运行 `az login` 并在浏览器窗口中进行身份验证： 

    ```azurecli
    az login
    ```
    
1. 指定所需的订阅。 使用 [az group create](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create) 创建资源组。 使用 [az storage account create](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create) 在该资源组中创建存储帐户： 

    ```azurecli
    az account set --subscription <subscription-id>

    az group create \
        --name sample-resource-group-cli \
        --location eastus

    az storage account create \
        --name <storage-account> \
        --resource-group sample-resource-group-cli \
        --location eastus \
        --sku Standard_LRS \
        --encryption-services blob
    ```
    
1. 创建容器之前，请向自己分配[存储 Blob 数据参与者](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)角色。 即使你是帐户所有者，也需要显式权限才能针对存储帐户执行数据操作。 有关分配 RBAC 角色的详细信息，请参阅[授予对 Azure blob 和队列数据使用 RBAC 在 Azure 门户中访问](storage-auth-aad-rbac.md)。

    > [!IMPORTANT]
    > 传播 RBAC 角色分配可能需要花费几分钟时间。
    
1. 在将 `--auth-mode` 参数设置为 `login` 的情况下，调用 [az storage container create](https://docs.microsoft.com/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create) 命令以使用 Azure AD 凭据创建容器：

    ```azurecli
    az storage container create \ 
        --account-name <storage-account> \ 
        --name sample-container \
        --auth-mode login
    ```

与 `--auth-mode` 参数关联的环境变量为 `AZURE_STORAGE_AUTH_MODE`。 可在环境变量中指定相应的值，以免每次调用 Azure 存储数据操作都要包含该值。

## <a name="call-powershell-commands-using-azure-ad-credentials"></a>使用 Azure AD 凭据调用 PowerShell 命令

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

若要使用 Azure PowerShell 登录并使用 Azure AD 凭据针对 Azure 存储运行后续操作，请创建一个存储上下文用于引用存储帐户，并包含 `-UseConnectedAccount` 参数。

以下示例演示如何在 Azure PowerShell 中使用 Azure AD 凭据，在新的存储帐户中创建一个容器。 请务必将尖括号中的占位符值替换为你自己的值：

1. 使用 `Connect-AzAccount` 命令登录到 Azure 订阅，然后遵照屏幕上的指示输入 Azure AD 凭据： 

    ```powershell
    Connect-AzAccount
    ```
    
1. 调用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。 

    ```powershell
    $resourceGroup = "sample-resource-group-ps"
    $location = "eastus"
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

1. 调用 [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) 创建存储帐户。

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name "<storage-account>" `
      -SkuName Standard_LRS `
      -Location $location `
    ```

1. 调用 [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext) 获取用于指定新存储帐户的存储帐户上下文。 对存储帐户执行操作时，可以引用上下文而不是重复传入凭据。 包含 `-UseConnectedAccount` 参数，以使用 Azure AD 凭据调用任何后续数据操作：

    ```powershell
    $ctx = New-AzStorageContext -StorageAccountName "<storage-account>" -UseConnectedAccount
    ```

1. 创建容器之前，请向自己分配[存储 Blob 数据参与者](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)角色。 即使你是帐户所有者，也需要显式权限才能针对存储帐户执行数据操作。 有关分配 RBAC 角色的详细信息，请参阅[授予对 Azure blob 和队列数据使用 RBAC 在 Azure 门户中访问](storage-auth-aad-rbac.md)。

    > [!IMPORTANT]
    > 传播 RBAC 角色分配可能需要花费几分钟时间。

1. 调用 [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer) 创建容器。 由于此调用使用在前面步骤中创建的上下文，因此将使用你的 Azure AD 凭据创建容器。 

    ```powershell
    $containerName = "sample-container"
    New-AzStorageContainer -Name $containerName -Context $ctx
    ```

## <a name="next-steps"></a>后续步骤

- 若要详细了解 Azure 存储中的 RBAC 角色，请参阅[使用 RBAC 管理存储数据的访问权限](storage-auth-aad-rbac.md)。
- 若要了解如何使用适用于使用 Azure 存储的 Azure 资源管理的标识，请参阅[验证对 blob 和队列与 Azure Active Directory 和管理的标识访问 Azure 资源的](storage-auth-aad-msi.md)。
- 若要了解如何从存储应用程序内授予容器和队列访问权限，请参阅[将 Azure AD 与存储应用程序配合使用](storage-auth-aad-app.md)。
