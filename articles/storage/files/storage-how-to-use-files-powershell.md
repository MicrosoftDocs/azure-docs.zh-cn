---
title: 快速入门：使用 Azure PowerShell 管理 Azure 文件共享
description: 通过本快速入门了解如何使用 Azure PowerShell 管理 Azure 文件共享。
services: storage
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 439a1c60942b1540328bf9972d74d7dd4d573a65
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64700652"
---
# <a name="quickstart-create-and-manage-an-azure-file-share-with-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建和管理 Azure 文件共享 
本指南介绍通过 PowerShell 来使用 [Azure 文件共享](storage-files-introduction.md)的基本知识。 Azure 文件共享与其他文件共享一样，只不过是存储在云中并由 Azure 平台提供支持。 Azure 文件共享支持行业标准 SMB 协议，可以跨多个计算机、应用程序和实例进行文件共享。 

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

若要在本地安装并使用 PowerShell，则本指南需要 Azure PowerShell 模块 Az 0.7 或更高版本。 若要找出正在运行的 Azure PowerShell 模块的版本，请执行 `Get-Module -ListAvailable Az`。 如果需要升级，请参阅[安装 Azure PowerShell 模块](/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Login-AzAccount` 以登录到 Azure 帐户。

## <a name="create-a-resource-group"></a>创建资源组
资源组是在其中部署和管理 Azure 资源的逻辑容器。 如果没有 Azure 资源组，可以使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) cmdlet 新建一个。 

以下示例在“美国东部”区域创建名为“myResourceGroup”的资源组：

```azurepowershell-interactive
New-AzResourceGroup `
    -Name myResourceGroup `
    -Location EastUS
```

## <a name="create-a-storage-account"></a>创建存储帐户
存储帐户是一个存储共享池，可以用来部署 Azure 文件共享或其他存储资源，例如 Blob 或队列。 一个存储帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到存储帐户的容量限制为止。

此示例使用 [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) cmdlet 创建存储帐户。 存储帐户名为 *mystorageaccount\<随机数字>*，对该存储帐户的引用存储在变量 **$storageAcct** 中。 存储帐户名称必须唯一，因此请使用 `Get-Random` 将一个数字追加到名称末尾，使之变得唯一。 

```azurepowershell-interactive 
$storageAcct = New-AzStorageAccount `
                  -ResourceGroupName "myResourceGroup" `
                  -Name "mystorageacct$(Get-Random)" `
                  -Location eastus `
                  -SkuName Standard_LRS 
```

## <a name="create-an-azure-file-share"></a>创建 Azure 文件共享
现在可以创建第一个 Azure 文件共享。 可以使用 [New-AzStorageShare](/powershell/module/az.storage/New-AzStorageShare) cmdlet 创建文件共享。 此示例创建名为 `myshare` 的共享。

```azurepowershell-interactive
New-AzStorageShare `
   -Name myshare `
   -Context $storageAcct.Context
```

共享名必须全部采用小写字母、数字和单个连字符，但不能以连字符开头。 有关命名文件共享和文件的完整详细信息，请参阅 [命名和引用共享、目录、文件和元数据](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata)。

## <a name="use-your-azure-file-share"></a>使用 Azure 文件共享
Azure 文件提供两种在 Azure 文件共享中使用文件和文件夹的方法：行业标准[服务器消息块 (SMB) 协议](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx)和[文件 REST 协议](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api)。 

若要通过 SMB 装载文件共享，请参阅下述基于 OS 的文档：
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>将 Azure 文件共享与文件 REST 协议配合使用 
可以直接使用文件 REST 协议（即手动进行 REST HTTP 调用），但最常见的使用文件 REST 协议的方式是使用 Azure PowerShell 模块、[Azure CLI](storage-how-to-use-files-cli.md) 或 Azure 存储 SDK，所有这些方式都可以在所选脚本/编程语言中为文件 REST 协议提供很好的包装器。  

大多数情况下，需要通过 SMB 协议来使用 Azure 文件共享，因为这样可以使用那些预期可以使用的现有应用程序和工具，但某些情况下，使用文件 REST API 比使用 SMB 更具优势，例如：

- 需要通过 PowerShell Cloud Shell（不能通过 SMB 来装载文件共享）来浏览文件共享。
- 需在无法装载 SMB 共享的客户端（例如未将端口 445 解除阻止的本地客户端）中执行脚本或应用程序。
- 需利用无服务器资源，例如 [Azure Functions](../../azure-functions/functions-overview.md)。 

以下示例介绍如何使用 Azure PowerShell 模块通过文件 REST 协议来操作 Azure 文件共享。 

#### <a name="create-directory"></a>创建目录
若要在 Azure 文件共享的根目录中创建名为 *myDirectory* 的新目录，请使用 [New-AzStorageDirectory](/powershell/module/az.storage/New-AzStorageDirectory) cmdlet。

```azurepowershell-interactive
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName "myshare" `
   -Path "myDirectory"
```

#### <a name="upload-a-file"></a>上传文件
若要演示如何使用 [Set-AzStorageFileContent](/powershell/module/az.storage/Set-AzStorageFileContent) cmdlet 来上传文件，首先需要在 PowerShell Cloud Shell 的暂存驱动器中创建要上传的文件。 

此示例将当前的日期和时间置于暂存驱动器的新文件中，然后将文件上传到文件共享。

```azurepowershell-interactive
# this expression will put the current date and time into a new file on your scratch drive
Get-Date | Out-File -FilePath "C:\Users\ContainerAdministrator\CloudDrive\SampleUpload.txt" -Force

# this expression will upload that newly created file to your Azure file share
Set-AzStorageFileContent `
   -Context $storageAcct.Context `
   -ShareName "myshare" `
   -Source "C:\Users\ContainerAdministrator\CloudDrive\SampleUpload.txt" `
   -Path "myDirectory\SampleUpload.txt"
```   

如果在本地运行 PowerShell，则应将 `C:\Users\ContainerAdministrator\CloudDrive\` 替换为计算机上的现有路径。

上传文件后，可以使用 [Get-AzStorageFile](/powershell/module/Az.Storage/Get-AzStorageFile) cmdlet 进行检查，确保文件已上传到 Azure 文件共享。 

```azurepowershell-interactive
Get-AzStorageFile -Context $storageAcct.Context -ShareName "myshare" -Path "myDirectory" 
```

#### <a name="download-a-file"></a>下载文件
可以使用 [Get-AzStorageFileContent](/powershell/module/az.storage/Get-AzStorageFilecontent) cmdlet 下载刚上传到 Cloud Shell 的暂存驱动器的文件的副本。

```azurepowershell-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists because you've run this example before.
Remove-Item `
     -Path "C:\Users\ContainerAdministrator\CloudDrive\SampleDownload.txt" `
     -Force `
     -ErrorAction SilentlyContinue

Get-AzStorageFileContent `
    -Context $storageAcct.Context `
    -ShareName "myshare" `
    -Path "myDirectory\SampleUpload.txt" ` 
    -Destination "C:\Users\ContainerAdministrator\CloudDrive\SampleDownload.txt"
```

下载文件以后，可以使用 `Get-ChildItem` 来查看该文件是否已下载到 PowerShell Cloud Shell 的暂存驱动器。

```azurepowershell-interactive
Get-ChildItem -Path "C:\Users\ContainerAdministrator\CloudDrive"
``` 

#### <a name="copy-files"></a>复制文件
一项常见的任务是将文件从一个文件共享复制到另一个文件共享，或者将文件在文件共享和 Azure Blob 存储容器之间来回复制。 若要演示此功能，可以创建一个新的共享，然后使用 [Start-AzStorageFileCopy](/powershell/module/az.storage/Start-AzStorageFileCopy) cmdlet 将刚上传的文件复制到该新共享。 

```azurepowershell-interactive
New-AzStorageShare `
    -Name "myshare2" `
    -Context $storageAcct.Context
  
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName "myshare2" `
   -Path "myDirectory2"

Start-AzStorageFileCopy `
    -Context $storageAcct.Context `
    -SrcShareName "myshare" `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestShareName "myshare2" `
    -DestFilePath "myDirectory2\SampleCopy.txt" `
    -DestContext $storageAcct.Context
```

现在，如果列出新共享中的文件，则应看到已复制的文件。

```azurepowershell-interactive
Get-AzStorageFile -Context $storageAcct.Context -ShareName "myshare2" -Path "myDirectory2" 
```

虽然 `Start-AzStorageFileCopy` cmdlet 可以方便地用于 Azure 文件共享和 Azure Blob 存储容器之间的临时文件移动，但我们仍建议你使用 AzCopy 进行较大型的移动（就要移动的文件的数量或大小而言）。 详细了解 [Windows 版 AzCopy](../common/storage-use-azcopy.md) 和 [Linux 版 AzCopy](../common/storage-use-azcopy-linux.md)。 AzCopy 必须安装在本地，在 Cloud Shell 中不可用。 

## <a name="create-and-manage-share-snapshots"></a>创建和管理共享快照
可以通过 Azure 文件共享执行的另一项有用的任务是创建共享快照。 快照保存的是 Azure 文件共享的某个时间点。 共享快照类似于你可能已经熟悉的操作系统技术，例如：
- 适用于 Windows 文件系统（例如 NTFS 和 ReFS）的[卷影复制服务 (VSS)](https://docs.microsoft.com/windows/desktop/VSS/volume-shadow-copy-service-portal)
- 适用于 Linux 系统的[逻辑卷管理器 (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) 快照。
- 适用于 macOS 的 [Apple 文件系统 (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) 快照。 
 可以在通过 [Get-AzStorageShare](/powershell/module/az.storage/Get-AzStorageShare) cmdlet 检索的文件共享的 PowerShell 对象上使用 `Snapshot` 方法来创建某个共享的共享快照。 

```azurepowershell-interactive
$share = Get-AzStorageShare -Context $storageAcct.Context -Name "myshare"
$snapshot = $share.Snapshot()
```

### <a name="browse-share-snapshots"></a>浏览共享快照
可以浏览共享快照的内容，只需将快照引用 (`$snapshot`) 传递给 `Get-AzStorageFile` cmdlet 的 `-Share` 参数即可。

```azurepowershell-interactive
Get-AzStorageFile -Share $snapshot
```

### <a name="list-share-snapshots"></a>列出共享快照
可以使用以下命令查看为共享生成的快照的列表。

```azurepowershell-interactive
Get-AzStorageShare -Context $storageAcct.Context | Where-Object { $_.Name -eq "myshare" -and $_.IsSnapshot -eq $true }
```

### <a name="restore-from-a-share-snapshot"></a>从共享快照还原
可以使用以前用过的 `Start-AzStorageFileCopy` 命令还原某个文件。 就本快速入门来说，首先请删除以前上传的 `SampleUpload.txt` 文件，这样才能将其从快照还原。

```azurepowershell-interactive
# Delete SampleUpload.txt
Remove-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName "myshare" `
    -Path "myDirectory\SampleUpload.txt"
 # Restore SampleUpload.txt from the share snapshot
Start-AzStorageFileCopy `
    -SrcShare $snapshot `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestContext $storageAcct.Context `
    -DestShareName "myshare" `
    -DestFilePath "myDirectory\SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>删除共享快照
可以使用 [Remove-AzStorageShare](/powershell/module/az.storage/Remove-AzStorageShare) cmdlet 删除共享快照，其中的变量包含对 `-Share` 参数的 `$snapshot` 引用。

```azurepowershell-interactive
Remove-AzStorageShare -Share $snapshot
```

## <a name="clean-up-resources"></a>清理资源
完成后，可以使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) cmdlet 删除资源组和所有相关的资源。 

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

也可逐一删除资源：

- 删除为本快速入门创建的 Azure 文件共享。

    ```azurepowershell-interactive
    Get-AzStorageShare -Context $storageAcct.Context | Where-Object { $_.IsSnapshot -eq $false } | ForEach-Object { 
        Remove-AzStorageShare -Context $storageAcct.Context -Name $_.Name
    }
    ```

- 删除存储帐户本身（这会隐式删除已创建的 Azure 文件共享，以及任何其他可能已创建的存储资源，例如 Azure Blob 存储容器）。

    ```azurepowershell-interactive
    Remove-AzStorageAccount -ResourceGroupName $storageAcct.ResourceGroupName -Name $storageAcct.StorageAccountName
    ```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [什么是 Azure 文件？](storage-files-introduction.md)
