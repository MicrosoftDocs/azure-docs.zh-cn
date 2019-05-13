---
title: 快速入门：使用 Azure 门户管理 Azure 文件共享
description: 通过本快速入门了解如何使用 Azure 门户管理 Azure 文件。
services: storage
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/18/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 69e7dba06fce14c2411a82dfade344ba1b14a6d3
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64698746"
---
# <a name="quickstart-create-and-manage-azure-file-shares-with-the-azure-portal"></a>快速入门：使用 Azure 门户创建和管理 Azure 文件共享 
[Azure 文件](storage-files-introduction.md)是 Microsoft 推出的易用云文件系统。 可以在 Windows、Linux 和 macOS 中装载 Azure 文件共享。 本指南介绍通过 [Azure 门户](https://portal.azure.com/)来使用 Azure 文件共享的基本知识。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-a-storage-account"></a>创建存储帐户
[!INCLUDE [storage-files-create-storage-account-portal](../../../includes/storage-files-create-storage-account-portal.md)]

## <a name="create-an-azure-file-share"></a>创建 Azure 文件共享
若要创建 Azure 文件共享，请执行以下操作：

1. 从仪表板中选择存储帐户。
2. 在存储帐户页的“服务”部分，选择“文件”。
    ![存储帐户的服务部分的屏幕截图；选择“文件”服务](media/storage-how-to-use-files-portal/create-file-share-1.png)

3. 在“文件服务”页面顶部的菜单中，单击“+ 文件共享”。 “新建文件共享”页此时会向下拉开。
4. 在“名称”中，键入 *myshare*。
5. 单击“确定”以创建 Azure 文件共享。

共享名必须全部采用小写字母、数字和单个连字符，但不能以连字符开头。 有关命名文件共享和文件的完整详细信息，请参阅 [命名和引用共享、目录、文件和元数据](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata)。

## <a name="use-your-azure-file-share"></a>使用 Azure 文件共享
Azure 文件提供两种在 Azure 文件共享中使用文件和文件夹的方法：行业标准[服务器消息块 (SMB) 协议](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx)和[文件 REST 协议](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api)。 

若要通过 SMB 装载文件共享，请参阅下述基于 OS 的文档：
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-from-the-azure-portal"></a>通过 Azure 门户使用 Azure 文件共享
所有通过 Azure 门户发出的请求都是通过文件 REST API 发出的，因此不需 SMB 访问权限即可在客户端创建、修改和删除文件和目录。 可以直接使用文件 REST 协议（即手动进行 REST HTTP 调用），但最常见的使用文件 REST 协议的方式（使用 Azure 门户除外）是使用 [Azure PowerShell 模块](storage-how-to-use-files-powershell.md)、[Azure CLI](storage-how-to-use-files-cli.md) 或 Azure 存储 SDK，所有这些方式都可以在所选脚本/编程语言中为文件 REST 协议提供很好的包装器。 

我们预计，在使用 Azure 文件时，大多数情况下需要通过 SMB 协议来使用 Azure 文件共享，因为这样可以使用那些预期可以使用的现有应用程序和工具，但某些情况下，使用文件 REST API 比使用 SMB 更具优势，例如：

- 需要在便携式设备（例如没有 SMB 访问权限的膝上型电脑、平板电脑或移动设备）上对 Azure 文件共享进行快速的更改。
- 需在无法装载 SMB 共享的客户端（例如未将端口 445 解除阻止的本地客户端）中执行脚本或应用程序。
- 需利用无服务器资源，例如 [Azure Functions](../../azure-functions/functions-overview.md)。 

以下示例介绍如何使用 Azure 门户通过文件 REST 协议来操作 Azure 文件共享。 

创建 Azure 文件共享以后，即可使用 SMB 在 [Windows](storage-how-to-use-files-windows.md)、[Linux](storage-how-to-use-files-linux.md) 或 [macOS](storage-how-to-use-files-mac.md) 上装载该文件共享。 也可通过 Azure 门户使用 Azure 文件共享。 

#### <a name="create-a-directory"></a>创建目录
若要在 Azure 文件共享的根目录中创建名为 *myDirectory* 的新目录，请执行以下操作：

1. 在“文件服务”页面中，选择“myshare”文件共享。 文件共享的页面此时会打开。
2. 在页面顶部的菜单中选择“+ 添加目录”。 “新建目录”页此时会向下拉开。
3. 键入 *myDirectory*，然后单击“确定”。

#### <a name="upload-a-file"></a>上传文件 
若要演示如何上传文件，首先需创建或选择一个需要上传的文件。 可以根据需要采用任何方式来这样做。 选定要上传的文件以后，请执行以下操作：

1. 单击 **myDirectory** 目录。 此时会打开 **myDirectory** 面板。
2. 在顶部菜单中，单击“上传”。 “上传文件”面板此时会打开。  
    ![“上传文件”面板的屏幕截图](media/storage-how-to-use-files-portal/upload-file-1.png)

3. 单击文件夹图标，打开一个用来浏览本地文件的窗口。 
4. 选择一个文件，然后单击“打开”。 
5. 在“上传文件”页中验证文件名，然后单击“上传”。
6. 完成后，文件会显示在 **myDirectory** 页的列表中。

#### <a name="download-a-file"></a>下载文件
若要下载已上传文件的副本，可以右键单击该文件。 单击下载按钮后的具体体验取决于所使用的操作系统和浏览器。

## <a name="clean-up-resources"></a>清理资源
[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [什么是 Azure 文件？](storage-files-introduction.md)
