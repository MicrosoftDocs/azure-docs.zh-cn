---
title: 使用 Windows VM 系统分配的托管标识访问 Azure 存储
description: 本教程将指导你完成使用 Windows VM 系统分配的托管标识访问 Azure 存储的过程。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/24/2019
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 147ee2450a6a67f8ca02149105533401d038a53a
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191081"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage-via-access-key"></a>教程：使用 Windows VM 系统分配的托管标识通过访问密钥访问 Azure 存储

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]


> [!IMPORTANT] 
> Azure 存储现在支持 Azure AD 身份验证。 最佳做法是使用 [Azure AD 身份验证](tutorial-vm-windows-access-storage.md)，而不是访问密钥。 


本教程介绍了如何使用 Windows 虚拟机 (VM) 的系统分配的托管标识来检索存储帐户访问密钥。 可以像平常在执行存储操作时一样使用存储访问密钥，例如使用存储 SDK 时。 本教程将使用 Azure 存储 PowerShell 上传和下载 blob。 将了解如何执行以下操作：


> [!div class="checklist"]
> * 创建存储帐户
> * 授予 VM 对资源管理器中存储帐户访问密钥的访问权限 
> * 使用 VM 的标识获取一个访问令牌，并使用它从资源管理器检索存储访问密钥 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="create-a-storage-account"></a>创建存储帐户 

如果还没有存储帐户，现在将创建存储帐户。 也可以跳过此步骤，并向 VM 的系统分配的托管标识授予对现有存储帐户密钥的访问权限。 

1. 单击 Azure 门户左上角的“+/创建新服务”按钮。
2. 依次单击“存储”、“存储帐户”，并将显示新的“创建存储帐户”面板。
3. 输入存储帐户的名称，稍后将使用该名称。  
4. **部署模型**和**帐户类型**应分别设置为“资源管理器”和“通用”。 
5. 确保“订阅”和“资源组”与上一步中创建 VM 时指定的名称匹配。
6. 单击“创建”。

    ![新建存储帐户](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>在存储帐户中创建 Blob 容器

稍后我们会将文件上传并下载到新存储帐户。 由于文件需要 blob 存储，我们需要创建用于存储文件的 blob 容器。

1. 导航回新创建的存储帐户。
2. 在“Blob 服务”下单击左侧的“容器”链接。
3. 单击页面顶部的“+ 容器”，将滑出“新建容器”面板。
4. 为容器指定名称，选择访问级别，单击“确定”。 在本教程中的后面部分将使用所指定的名称。 

    ![创建存储容器](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-storage-account-access-keys"></a>授权 VM 的系统分配的托管标识使用存储帐户访问密钥 

Azure 存储原本不支持 Azure AD 身份验证。  但是，可以使用 VM 的系统分配的托管标识从资源管理器检索存储帐户访问密钥，并使用这些密钥访问存储。  在此步骤中，将向 VM 的系统分配的托管标识授予对存储帐户密钥的访问权限。   

1. 导航回新创建的存储帐户。  
2. 单击左侧面板中的“访问控制(IAM)”链接。  
3. 单击页面顶部的“+ 添加角色分配”，为 VM 添加新的角色分配
4. 在页面右侧，将“角色”设置为“存储帐户密钥操作员服务角色”。 
5. 在下一个下拉列表中，把“将访问权限分配给”设置为资源“虚拟机”。  
6. 接下来，确保“订阅”下拉列表中列出了正确的订阅，然后将“资源组”设置为“所有资源组”。  
7. 最后，在“选择”下，从下拉列表中选择 Windows 虚拟机，然后单击“保存”。 

    ![Alt 图像文本](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role.png)

## <a name="get-an-access-token-using-the-vms-system-assigned-managed-identity-to-call-azure-resource-manager"></a>使用 VM 的系统分配的托管标识获取访问令牌以调用 Azure 资源管理器 

在本教程的剩余部分中，我们从先前创建的 VM 入手。 

在此部分中，将需要使用 Azure 资源管理器 PowerShell cmdlet。  如果尚未安装，请[下载最新版本](https://docs.microsoft.com/powershell/azure/overview)，然后再继续。

1. 在 Azure 门户中，导航到“虚拟机”，转到 Windows 虚拟机，然后在“概述”页中单击顶部的“连接”。 
2. 输入创建 Windows VM 时添加的用户名和密码。 
3. 现在，已经创建了与虚拟机的远程桌面连接，请在远程会话中打开 PowerShell。
4. 使用 Powershell 的 Invoke-WebRequest，向 Azure 资源终结点的本地托管标识发出请求以获取 Azure 资源管理器的访问令牌。

    ```powershell
       $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
    ```
    
    > [!NOTE]
    > “资源”参数的值必须完全匹配 Azure AD 预期的值。 如果使用 Azure 资源管理器资源 ID，必须在 URI 的结尾添加斜线。
    
    接下来，提取“内容”元素，该元素以 JavaScript 对象表示法 (JSON) 格式字符串的形式存储在 $response 对象中。 
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```
    接下来，从响应中提取访问令牌。
    
    ```powershell
    $ArmToken = $content.access_token
    ```
 
## <a name="get-storage-account-access-keys-from-azure-resource-manager-to-make-storage-calls"></a>从 Azure 资源管理器中获取存储帐户访问密钥，以便调用存储  

现在，我们将使用在上一部分中检索到的访问令牌通过 PowerShell 调用资源管理器，以便检索存储访问密钥。 获得存储访问密钥后，便可以调用存储上传/下载操作。

```powershell
$keysResponse = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE-ACCOUNT>/listKeys/?api-version=2016-12-01 -Method POST -Headers @{Authorization="Bearer $ARMToken"}
```
> [!NOTE] 
> URL 区分大小写。因此，请确保使用的大小写与之前在命名资源组时使用的大小写完全相同，包括“resourceGroups”使用的是大写“G”。 

```powershell
$keysContent = $keysResponse.Content | ConvertFrom-Json
$key = $keysContent.keys[0].value
```

接下来，我们将创建一个名为“test.txt”的文件。 然后，通过 `New-AzStorageContent` cmdlet 使用存储访问密钥进行身份验证，将该文件上传到 blob 容器，然后下载该文件。

```bash
echo "This is a test text file." > test.txt
```

请务必首先使用 `Install-Module Az.Storage` 安装 Azure 存储 cmdlet。 然后，可以使用 `Set-AzStorageBlobContent` PowerShell cmdlet 上传刚创建的 blob：

```powershell
$ctx = New-AzStorageContext -StorageAccountName <STORAGE-ACCOUNT> -StorageAccountKey $key
Set-AzStorageBlobContent -File test.txt -Container <CONTAINER-NAME> -Blob testblob -Context $ctx
```

响应：

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/13/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

也可以使用 `Get-AzStorageBlobContent` PowerShell cmdlet 下载刚上传的 blob：

```powershell
Get-AzStorageBlobContent -Blob testblob -Container <CONTAINER-NAME> -Destination test2.txt -Context $ctx
```

响应：

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 56
ContentType       : application/octet-stream
LastModified      : 9/13/2017 6:14:25 PM +00:00
SnapshotTime      :
ContinuationToken :
Context           : Microsoft.WindowsAzure.Commands.Storage.AzureStorageContext
Name              : testblob
```

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何创建系统分配的托管标识来使用访问密钥访问 Azure 存储。  若要详细了解 Azure 存储访问密钥，请参阅：

> [!div class="nextstepaction"]
>[管理存储访问密钥](/azure/storage/common/storage-create-storage-account)

