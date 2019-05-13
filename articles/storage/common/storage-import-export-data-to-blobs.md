---
title: 使用 Azure 导入/导出将数据传输到 Azure Blob | Microsoft Docs
description: 了解如何在 Azure 门户中创建导入和导出作业，以便将数据传入和传出 Azure Blob。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 04/08/2019
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 82672136d6f9af50a3d91da2044f6e0ced4b44a6
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409365"
---
# <a name="use-the-azure-importexport-service-to-import-data-to-azure-blob-storage"></a>使用 Azure 导入/导出服务将数据导入到 Azure Blob 存储

本文提供了有关如何使用 Azure 导入/导出服务安全地将大量数据导入到 Azure Blob 存储的分步说明。 若要将数据导入到 Azure Blob，此服务要求你将包含数据的已加密磁盘驱动器寄送到某个 Azure 数据中心。  

## <a name="prerequisites"></a>必备组件

在创建导入作业来将数据传输到 Azure Blob 存储之前，请仔细查看并完成此服务的以下先决条件列表。 必须：

- 拥有可用于导入/导出服务的有效 Azure 订阅。
- 拥有至少一个包含存储容器的 Azure 存储帐户。 请参阅[导入/导出服务支持的存储帐户和存储类型](storage-import-export-requirements.md)的列表。 
    - 有关创建新存储帐户的信息，请参阅[如何创建存储帐户](storage-quickstart-create-account.md)。 
    - 有关存储容器的信息，请转到[创建存储容器](../blobs/storage-quickstart-blobs-portal.md#create-a-container)。
- 拥有[受支持类型](storage-import-export-requirements.md#supported-disks)的足够数量的磁盘。 
- 拥有运行[受支持 OS 版本](storage-import-export-requirements.md#supported-operating-systems)的 Windows 系统。 
- 在 Windows 系统上启用 BitLocker。 请参阅[如何启用 BitLocker](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/)。
- 在 Windows 系统上[下载 WAImportExport 版本 1](https://aka.ms/waiev1)。 解压缩到默认文件夹 `waimportexportv1`。 例如，`C:\WaImportExportV1`。
- 具有 FedEx/DHL 帐户。 如果你想要使用非 FedEx/DHL 快递商，请联系 Azure 数据框操作团队`adbops@microsoft.com`。  
    - 该帐户必须是有余额的有效帐户，且有退货功能。
    - 生成导出作业的跟踪号。
    - 每个作业都应有一个单独的跟踪号。 不支持多个作业共享相同跟踪号。
    - 如果没有承运商帐户，请转到：
        - [创建 FedEX 帐户](https://www.fedex.com/en-us/create-account.html)，或 
        - [创建 DHL 帐户](http://www.dhl-usa.com/en/express/shipping/open_account.html)。

## <a name="step-1-prepare-the-drives"></a>步骤 1：准备驱动器

此步骤生成一个日志文件。 日志文件存储着驱动器序列号、加密密钥和存储帐户详细信息等基本信息。 

请执行以下步骤来准备驱动器。

1.  通过 SATA 连接器将磁盘驱动器连接到 Windows 系统。
1.  在每个驱动器上创建一个 NTFS 卷。 为卷分配驱动器号。 不要使用装入点。
2.  在 NTFS 卷上启用 BitLocker 加密。 如果使用某个 Windows Server 系统，请使用[如何在 Windows Server 2012 R2 上启用 BitLocker](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/) 中的说明。
3.  将数据复制到加密的卷。 可使用拖放或 Robocopy 或任何类似的复制工具。
4.  使用管理权限打开一个 PowerShell 或命令行窗口。 若要将目录切换到解压缩的文件夹，请运行以下命令：
    
    `cd C:\WaImportExportV1`
5.  若要获取驱动器的 BitLocker 密钥，请运行以下命令：
    
    `manage-bde -protectors -get <DriveLetter>:`
6.  若要准备磁盘，请运行以下命令。 **这可能要花费几小时到几天时间，具体取决于数据大小。** 

    ```
    ./WAImportExport.exe PrepImport /j:<journal file name> /id:session#<session number> /sk:<Storage account key> /t:<Drive letter> /bk:<BitLocker key> /srcdir:<Drive letter>:\ /dstdir:<Container name>/ /skipwrite 
    ```
    在运行该工具的同一文件夹中会创建一个日志文件。 还会创建两个其他文件 - 一个 *.xml* 文件（您在其中运行工具的文件夹）和一个 *drive-manifest.xml* 文件（数据所在的文件夹）。
    
    下表介绍了所使用的参数：

    |选项  |描述  |
    |---------|---------|
    |/j:     |带有 .jrn 扩展名的日志文件的名称。 会为每个驱动器生成一个日志文件。 建议使用磁盘序列号作为日志文件名。         |
    |/id:     |会话 ID。 请为该命令的每个实例使用唯一的会话编号。      |
    |/sk:     |Azure 存储帐户密钥。         |
    |/t:     |要寄送的磁盘的驱动器号。 例如，驱动器 `D`。         |
    |/bk:     |驱动器的 BitLocker 密钥。 其数字密码来自 `manage-bde -protectors -get D:` 的输出      |
    |/srcdir:     |要寄送的磁盘的驱动器号后跟 `:\`。 例如，`D:\`。         |
    |/dstdir:     |Azure 存储中的目标容器的名称。         |
    |/skipwrite:     |此选项指定没有需要复制的新数据并且要准备磁盘上的现有数据。          |
7. 为需要寄送的每个磁盘重复前面的步骤。 每次运行该命令行时，都会使用所提供的名称创建一个日志文件。
    
    > [!IMPORTANT]
    > - 与日志文件一起，还会在工具所在的同一文件夹中创建一个 `<Journal file name>_DriveInfo_<Drive serial ID>.xml` 文件。 如果日志文件太大，在创建作业时会使用该 .xml 文件而不使用日志文件。 

## <a name="step-2-create-an-import-job"></a>步骤 2：创建导入作业

在 Azure 门户中执行以下步骤来创建导入作业。

1. 登录到 https://portal.azure.com/。
2. 转到“所有服务”>“存储”>“导入/导出作业”。 
    
    ![转到导入/导出作业](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. 单击“创建导入/导出作业”。

    ![单击“创建导入/导出作业”](./media/storage-import-export-data-to-blobs/import-to-blob2.png)

4. 在“基本信息”中：

   - 选择“导入到 Azure”。
   - 输入导入作业的描述性名称。 可使用此名称来跟踪作业进度。
       - 此名称只能包含小写字母、数字和连字符。
       - 此名称必须以字母开头，并且不得包含空格。
   - 选择订阅。
   - 输入或选择一个资源组。  

     ![创建导入作业 - 步骤 1](./media/storage-import-export-data-to-blobs/import-to-blob3.png)

3. 在“作业详细信息”中：

    - 上传你在驱动器准备步骤中获取的驱动器日志文件。 如果使用了 `waimportexport.exe version1`，请为你准备的每个驱动器上传一个文件。 如果日志文件大小超过了 2 MB，则可以使用随日志文件创建的 `<Journal file name>_DriveInfo_<Drive serial ID>.xml`。 
    - 选择将用来存放数据的目标存储帐户。 
    - 放置位置会根据选定存储帐户所属的区域自动进行填充。
   
   ![创建导入作业 - 步骤 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

4. 在“回寄信息”中：

   - 从下拉列表中选择承运商。 如果你想要使用非 FedEx/DHL 快递商，请从下拉列表中选择一个现有的选项。 联系 Azure 数据框操作团队的`adbops@microsoft.com`与你打算使用与运营商相关的信息。
   - 输入你已在该承运商那里创建的有效承运商帐户编号。 导入作业完成后，Microsoft 使用此帐户寄回驱动器。 如果还没有帐户编号，请创建一个 [FedEx](https://www.fedex.com/us/oadr/) 或 [DHL](https://www.dhl.com/) 承运商帐户。
   - 提供完整、有效的联系人姓名、电话号码、电子邮件地址、街道地址、城市、邮政编码、省/自治区/直辖市和国家/地区。 
        
       > [!TIP] 
       > 请提供组电子邮件，而非为单个用户指定电子邮件地址。 这可确保即使管理员离开也会收到通知。

     ![创建导入作业 - 步骤 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)
   
5. 在“摘要”中：

   - 在摘要中复查提供的作业信息。 记下作业名称和 Azure 数据中心送货地址，以便将将磁盘寄回 Azure。 稍后将在发货标签中使用此信息。
   - 单击“确定”以创建导入作业。

     ![创建导入作业 - 步骤 4](./media/storage-import-export-data-to-blobs/import-to-blob6.png)

## <a name="step-3-ship-the-drives"></a>步骤 3：寄送驱动器 

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]


## <a name="step-4-update-the-job-with-tracking-information"></a>步骤 4：使用跟踪信息更新作业

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>步骤 5：验证 Azure 中的数据上传

跟踪作业直至完成。 作业完成后，验证数据已上传到 Azure。 仅在已确认上传成功后才删除本地数据。

## <a name="next-steps"></a>后续步骤

* [查看作业和驱动器状态](storage-import-export-view-drive-status.md)
* [查看导入/导出要求](storage-import-export-requirements.md)


