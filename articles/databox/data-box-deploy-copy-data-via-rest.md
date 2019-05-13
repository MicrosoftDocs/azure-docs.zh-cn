---
title: 有关通过 REST API 将数据复制到 Azure Data Box Blob 存储的教程 | Microsoft Docs
description: 了解如何通过 REST API 将数据复制到 Azure Data Box Blob 存储
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 05/09/2019
ms.author: alkohli
ms.openlocfilehash: b3d53b7d5fceb303259823198731b5198cfce82c
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508267"
---
# <a name="tutorial-copy-data-to-azure-data-box-blob-storage-via-rest-apis"></a>教程：通过 REST API 将数据复制到 Azure Data Box Blob 存储  

本教程介绍使用 REST API 通过 *http* 或 *https* 连接到 Azure Data Box Blob 存储的过程。 此外还会介绍在建立连接后，将数据复制到 Data Box Blob 存储以及准备要寄送的 Data Box 的步骤。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 先决条件
> * 通过 *http* 或 *https* 连接到 Data Box Blob 存储
> * 将数据复制到 Data Box

## <a name="prerequisites"></a>先决条件

在开始之前，请确保：

1. 已完成[教程：设置 Azure Data Box](data-box-deploy-set-up.md)。
2. 已收到 Data Box，并且门户中的订单状态为“已送达”。
3. 已查看 [Data Box Blob 存储的系统要求](data-box-system-requirements-rest.md)，并熟悉 API、SDK 和工具的受支持版本。
4. 可以访问一台要将其中的数据复制到 Data Box 的主机。 该主机必须
    - 运行[支持的操作系统](data-box-system-requirements.md)。
    - 连接到高速网络。 强烈建议你至少建立一个 10-GbE 连接。 如果 10-GbE 连接不可用，可以使用 1-GbE 数据链路，但复制速度会受影响。
5. 在主机上[下载 AzCopy 7.1.0](https://aka.ms/azcopyforazurestack20170417)。 稍后要使用 AzCopy 将数据从主机复制到 Azure Data Box Blob 存储。


## <a name="connect-via-http-or-https"></a>通过 http 或 https 进行连接

可以通过 *http* 或 *https* 连接到 Data Box Blob 存储。

- Https 比较安全，是连接到 Data Box Blob 存储的推荐方法。
- 通过受信任的网络连接时，可以使用 *http*。

通过 http 或 https 连接到 Data Box Blob 存储时，连接步骤会有所不同。

## <a name="connect-via-http"></a>通过 http 进行连接

通过 *http* 连接到 Data Box Blob 存储 REST API 需要执行以下步骤：

- 将设备 IP 和 Blob 服务终结点添加到远程主机
- 配置第三方软件并验证连接

后续部分将介绍其中的每个步骤。

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>添加设备 IP 地址和 Blob 服务终结点

[!INCLUDE [data-box-add-device-ip](../../includes/data-box-add-device-ip.md)]

### <a name="configure-partner-software-and-verify-connection"></a>配置合作伙伴软件并验证连接

[!INCLUDE [data-box-configure-partner-software](../../includes/data-box-configure-partner-software.md)]

[!INCLUDE [data-box-verify-connection](../../includes/data-box-verify-connection.md)]

## <a name="connect-via-https"></a>通过 https 进行连接

通过 https 连接到 Azure Blob 存储 REST API 需要执行以下步骤：

- 从 Azure 门户下载证书
- 在客户端或远程主机上导入证书
- 将设备 IP 和 Blob 服务终结点添加到客户端或远程主机
- 配置第三方软件并验证连接

后续部分将介绍其中的每个步骤。

### <a name="download-certificate"></a>下载证书

使用 Azure 门户下载证书。

1. 登录到 Azure 门户。
2. 转到你的 Data Box 订单，然后导航到“常规”>“设备详细信息”。
3. 在“设备凭据”下，转到设备的“访问 API”。 单击“下载”。 此操作将下载 **\<你的订单名称>.cer** 证书文件。 **保存**此文件。 稍后将在用于连接到设备的客户端或主机上安装此证书。

    ![在 Azure 门户中下载证书](media/data-box-deploy-copy-data-via-rest/download-cert-1.png)
 
### <a name="import-certificate"></a>导入证书 

通过 HTTPS 访问 Data Box Blob 存储需要设备的 SSL 证书。 客户端应用程序可以使用此证书的方式因应用程序以及操作系统和分发而异。 一些应用程序可以在将证书导入系统的证书存储后访问该证书，而其他应用程序则不使用该机制。

本部分提到了某些应用程序的特定信息。 有关其他应用程序的详细信息，请参阅应用程序和所用操作系统的文档。

按照以下步骤将 `.cer` 文件导入 Windows 或 Linux 客户端的根存储中。 在 Windows 系统中，可以使用 Windows PowerShell 或 Windows Server UI 在系统上导入并安装该证书。

#### <a name="use-windows-powershell"></a>使用 Windows PowerShell

1. 以管理员身份启动 Windows PowerShell 会话。
2. 在命令提示符处，键入：

    ```
    Import-Certificate -FilePath C:\temp\localuihttps.cer -CertStoreLocation Cert:\LocalMachine\Root
    ```

#### <a name="use-windows-server-ui"></a>使用 Windows Server UI

1.  右键单击 `.cer` 文件并选择“安装证书”。 该操作会启动证书导入向导。
2.  对于“存储位置”，选择“本地计算机”，并单击“下一步”。

    ![使用 PowerShell 导入证书](media/data-box-deploy-copy-data-via-rest/import-cert-ws-1.png)

3.  选择“将所有证书放入下列存储”，并单击“浏览”。 导航到远程主机的根存储，并单击“下一步”。

    ![使用 PowerShell 导入证书](media/data-box-deploy-copy-data-via-rest/import-cert-ws-2.png)

4.  单击“完成”。 将显示一条提示已成功导入的消息。

    ![使用 PowerShell 导入证书](media/data-box-deploy-copy-data-via-rest/import-cert-ws-3.png)

#### <a name="use-a-linux-system"></a>使用 Linux 系统

导入证书的方法因分发而异。

例如 Ubuntu 和Debian 等使用 `update-ca-certificates` 命令。  

- 将 Base64 编码的证书文件重命名为 `.crt` 扩展并将其复制到 `/usr/local/share/ca-certificates directory`。
- 运行命令 `update-ca-certificates`。

最新版本的 RHEL、Fedora 和 CentOS 使用 `update-ca-trust` 命令。

- 将证书文件复制到 `/etc/pki/ca-trust/source/anchors` 目录中。
- 运行 `update-ca-trust`。

有关详细信息，请参阅特定于分发的文档。

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>添加设备 IP 地址和 Blob 服务终结点 

在 http 上进行连接时，请按照相同的步骤[添加设备 IP 地址和 blob 服务终结点](#add-device-ip-address-and-blob-service-endpoint)。

### <a name="configure-partner-software-and-verify-connection"></a>配置合作伙伴软件并验证连接

按照通过 http 进行连接时使用的步骤[配置合作伙伴软件](#configure-partner-software-and-verify-connection)， 唯一的差别在于，应将“使用 http 选项”保留未选中状态。

## <a name="copy-data-to-data-box"></a>将数据复制到 Data Box

连接到 Data Box Blob 存储后，下一步是复制数据。 在复制数据之前，请查看以下注意事项：

-  复制数据时，请确保数据大小符合 [Azure 存储和 Data Box 限制](data-box-limits.md)中所述的大小限制。
- 如果 Data Box 正在上传的数据同时已由 Data Box 外部的其他应用程序上传，则可能会导致上传作业失败和数据损坏。

本教程使用 AzCopy 将数据复制到 Data Box Blob 存储。 你也可以使用 Azure 存储资源管理器（如果你偏好使用基于 GUI 的工具）或合作伙伴软件来复制数据。
复制过程包括以下步骤：

- 创建容器
- 将文件夹的内容上传到 Data Box Blob 存储
- 将修改的文件上传到 Data Box Blob 存储

以下部分详细介绍了这些步骤。

### <a name="create-a-container"></a>创建容器

第一步是创建容器，因为 Blob 始终上传到容器中。 容器可以组织 Blob 组，就像组织计算机上的文件夹中的文件一样。 遵循以下步骤创建 Blob 容器。

1. 打开存储资源管理器。
2. 在左窗格中，展开需要在其中创建 Blob 容器的存储帐户。
3. 右键单击“Blob 容器”，从上下文菜单中选择“创建 Blob 容器”。

   ![“创建 Blob 容器”上下文菜单](media/data-box-deploy-copy-data-via-rest/create-blob-container-1.png)

4. 此时会在“Blob 容器”文件夹下面显示一个文本框。 输入 Blob 容器的名称。 有关 Blob 容器命名规则和限制的信息，请参阅[创建容器和设置权限](../storage/blobs/storage-quickstart-blobs-dotnet.md)。
5. 完成时按 **Enter** 可创建 Blob 容器，按 **Esc** 可取消相关操作。 成功创建 Blob 容器后，该容器会显示在所选存储帐户的“Blob 容器”文件夹下。

   ![已创建 Blob 容器](media/data-box-deploy-copy-data-via-rest/create-blob-container-2.png)

### <a name="upload-contents-of-a-folder-to-data-box-blob-storage"></a>将文件夹的内容上传到 Data Box Blob 存储

使用 AzCopy 将文件夹中的所有文件上传到 Windows 或 Linux 上的 Blob 存储中。 若要上传文件夹中的所有 Blob，请输入以下 AzCopy 命令：

#### <a name="linux"></a>Linux

    azcopy \
        --source /mnt/myfolder \
        --destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
        --dest-key <key> \
        --recursive

#### <a name="windows"></a>Windows

    AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S


将 `<key>` 替换为你的帐户密钥。 若要获取帐户密钥，请在 Azure 门户中转到自己的存储帐户。 转到“设置”>“访问密钥”，选择一个密钥，并将其粘贴到 AzCopy 命令中。

如果指定的目标容器不存在，AzCopy 将创建它并将文件上传到其中。 将源路径更新为数据目录，并将目标 URL 中的 `data-box-storage-account-name` 替换为与 Data Box 关联的存储帐户的名称。

若要将指定目录的内容以递归方式上传到 Blob 存储，请指定 `--recursive` (Linux) 或 `/S` (Windows) 选项。 当使用这些选项之一运行 AzCopy 时，会同时上传所有子文件夹及其中文件。

### <a name="upload-modified-files-to-data-box-blob-storage"></a>将修改的文件上传到 Data Box Blob 存储

根据文件的上次修改时间，使用 AzCopy 上传文件。 若要尝试此操作，可在源目录中修改文件或创建新文件，用于测试目的。 如果仅上传更新的或新文件，请将 `--exclude-older` (Linux) 或 `/XO` (Windows) 参数添加到 AzCopy 命令。

如果只想复制目标中不存在的源资源，在 AzCopy 命令中同时指定 `--exclude-older` 和 `--exclude-newer` (Linux) 或 `/XO` 和 `/XN` (Windows) 参数。 AzCopy 仅上传更新的数据（基于时间戳）。

#### <a name="linux"></a>Linux
    azcopy \
    --source /mnt/myfolder \
    --destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
    --dest-key <key> \
    --recursive \
    --exclude-older

#### <a name="windows"></a>Windows

    AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S /XO

下一步是准备好要交付的设备。

## <a name="next-steps"></a>后续步骤

本教程介绍了有关 Azure Data Box 的主题，例如：

> [!div class="checklist"]
> * 先决条件
> * 通过 *http* 或 *https* 连接到 Data Box Blob 存储
> * 将数据复制到 Data Box


请继续学习下一篇教程，了解如何将 Data Box 寄回 Microsoft。

> [!div class="nextstepaction"]
> [将 Azure Data Box 寄送到 Microsoft](./data-box-deploy-picked-up.md)
