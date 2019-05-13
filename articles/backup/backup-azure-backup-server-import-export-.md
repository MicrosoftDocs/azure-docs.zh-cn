---
title: Azure 备份 - DPM 和 Azure 备份服务器的脱机备份
description: 了解如何在 Azure 备份中使用 Azure 导入/导出服务离线发送数据。 本文介绍如何使用 Azure 导入导出服务来脱机设定初始备份数据的种子。
services: backup
author: saurabhsensharma
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 5/8/2018
ms.author: saurse
ms.openlocfilehash: 05e7dd11274bb120e40df4a63d428ac6b6d7489f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65143337"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server"></a>DPM 和 Azure 备份服务器的脱机备份工作流
Azure 备份有多个可提升效率的内置功能，能在数据初始完整备份到 Azure 期间节省网络和存储成本。 初始完整备份通常传输大量数据，且需要较多网络带宽，相比之下，后续备份只传输增量部分。 Azure 备份可压缩初始备份。 通过脱机种子设定，Azure 备份可以使用磁盘将压缩后的初始备份数据脱机上传到 Azure。

Azure 备份的脱机种子设定与 [Azure 导入/导出服务](../storage/common/storage-import-export-service.md)紧密集成，允许使用磁盘将数据传输到 Azure。 如果要通过高延迟、低带宽网络传输 TB 计的初始备份数据，可以使用脱机种子设定工作流将一个或多个硬盘驱动器中的初始备份副本传送到 Azure 数据中心。 本文对完成 System Center DPM 和 Azure 备份服务器的此工作流进行了概述，并对涉及的步骤进行了详细说明。

> [!NOTE]
> Microsoft Azure 恢复服务 (MARS) 代理的脱机备份过程不同于 System Center DPM 和 Azure 备份服务器的脱机备份过程。 有关使用 MARS 代理进行脱机备份的信息，请参阅[本文](backup-azure-backup-import-export.md)。 使用 Azure 备份代理完成的系统状态备份不支持脱机备份。 
>

## <a name="overview"></a>概述
借助 Azure 备份的脱机种子设定及 Azure 导入/导出，可以简单直接地通过磁盘将数据脱机上传到 Azure。 脱机备份过程涉及以下步骤：

> [!div class="checklist"]
> * 备份数据不会通过网络发送，而是写入到暂存位置
> * 然后使用 AzureOfflineBackupDiskPrep 实用工具将暂存位置上的数据写入到一个或多个 SATA 磁盘
> * 该实用工具会自动创建 Azure 导入作业
> * 然后将 SATA 驱动器发送到最近的 Azure 数据中心
> * 将备份数据上传到 Azure 之后，Azure 备份将备份数据复制到备份保管库，并计划增量备份。

## <a name="supported-configurations"></a>支持的配置 
在场外将数据从本地备份到 Microsoft 云的 Azure 备份的所有部署模型都支持脱机备份。 这包括：

> [!div class="checklist"]
> * 使用 Microsoft Azure 恢复服务 (MARS) 代理或 Azure 备份代理备份文件和文件夹。 
> * 使用 System Center Data Protection Manager (SC DPM) 备份所有工作负荷和文件 
> * 使用 Microsoft Azure 备份服务器备份所有工作负荷和文件 <br/>

## <a name="prerequisites"></a>必备组件
确保在启动脱机备份工作流之前已满足以下先决条件
* 已创建[恢复服务保管库](backup-azure-recovery-services-vault-overview.md)。 若要创建，请参阅[这篇文章](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault)中的步骤
* Windows Server/Windows 客户端上已安装了 Azure 备份代理、Azure 备份服务器或 SC DPM（若适用），并已向恢复服务保管库注册了计算机。 确保仅使用[最新版本的 Azure 备份](https://go.microsoft.com/fwlink/?linkid=229525)。 
* 在打算备份数据的计算机上，[下载 Azure 发布设置文件](https://portal.azure.com/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade)。 下载发布设置文件的订阅可以不同于包含恢复服务保管库的订阅。 如果你的订阅位于主权 Azure 云上，则根据需要使用以下链接下载 Azure 发布设置文件。

    | 主权云区域 | Azure 发布设置文件链接 |
    | --- | --- |
    | 美国 | [链接](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |
    | 中国 | [链接](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |

* 在下载发布设置文件的订阅中已创建了采用经典部署模型的 Azure 存储帐户，如下所示： 

  ![创建经典存储帐户](./media/backup-azure-backup-import-export/storageaccountclassiccreate.png)

* 创建了一个暂存位置，它可以是计算机上的网络共享或任何其他内部或外部驱动器，并且有足够的磁盘空间来保存初始副本。 例如，如果尝试备份 500 GB 文件服务器，请确保暂存区域至少有 500 GB 空间。 （由于压缩，实际使用量更少。）
* 对于将发送到 Azure 的磁盘，请确保仅使用 2.5 英寸 SSD 或 2.5 英寸或 3.5 英寸 SATA II/III 内部硬盘驱动器。 可以使用容量最高为 10 TB 的硬盘驱动器。 查看 [Azure 导入/导出服务文档](../storage/common/storage-import-export-requirements.md#supported-hardware)，了解服务支持的最新驱动器集。
* SATA 驱动器必须连接到一台计算机（称为“副本计算机”），在这台计算机中完成将备份数据从暂存位置复制到 SATA 驱动器。 请确保已在“副本计算机”上启用 BitLocker 

## <a name="workflow"></a>工作流
本部分中的信息可帮助完成脱机备份工作流，将数据传递到 Azure 数据中心，并上传到 Azure 存储。 如有关于导入服务或流程的任何方面的问题，请参阅上面提到的[导入服务概述](../storage/common/storage-import-export-service.md)文档。

### <a name="initiate-offline-backup"></a>启动脱机备份
1. 计划备份时，会看到以下屏幕（在 Windows Server、Windows 客户端或 System Center Data Protection Manager 中）。

    ![导入屏幕](./media/backup-azure-backup-import-export/offlineBackupscreenInputs.png)

    下面是 System Center Data Protection Manager 中的相应屏幕： <br/>
    ![SC DPM 和 Azure 备份服务器导入屏幕](./media/backup-azure-backup-import-export/dpmoffline.png)

    输入的说明如下：

   * **暂存位置**：初始备份副本写入到的临时存储位置。 暂存位置可以是在网络共享或本地计算机。 如果副本计算机与源计算机不同，建议指定暂存位置的完整网络路径。
   * **Azure 导入作业名称**：Azure 导入服务和 Azure 备份在跟踪磁盘上发送到 Azure 的数据的传输活动时使用的唯一名称。
   * **Azure 发布设置**：提供发布设置文件的本地路径。
   * **Azure 订阅 ID**：下载 Azure 发布设置文件的订阅的 Azure 订阅 ID。 
   * **Azure 存储帐户**：Azure 订阅中与 Azure 发布设置文件关联的存储帐户的名称。
   * **Azure 存储容器**：Azure 存储帐户中导入备份数据的目标存储 Blob 的名称。

     保存提供的暂存位置和 Azure 导入作业名称，因为准备磁盘需要它们。  
     
2. 完成工作流，要启动脱机备份复制，请在 Azure 备份代理管理控制台中选择“立即备份”。 初始备份写入到临时区域作为此步骤的一部分。

    ![立即备份](./media/backup-azure-backup-import-export/backupnow.png)

    要在 System Center Data Protection Manager 或 Azure 备份服务器中完成相应的工作流，请右键单击“保护组”并选择“创建恢复点”选项。 然后选择“联机保护”选项。

    ![SC DPM 和 Azure 备份服务器立即备份](./media/backup-azure-backup-import-export/dpmbackupnow.png)

    操作完成后，暂存位置已准备就绪，可用于准备磁盘。

    ![备份进度](./media/backup-azure-backup-import-export/opbackupnow.png)

### <a name="prepare-sata-drives-and-ship-to-azure"></a>准备 SATA 驱动器并寄送到 Azure
*AzureOfflineBackupDiskPrep*实用程序用于准备送到最近的 Azure 数据中心的 SATA 驱动器。 在以下路径的恢复服务代理安装目录中可获得此实用工具：

*\\Microsoft Azure 恢复服务代理\\Utils\\*

1. 转到该目录，将“AzureOfflineBackupDiskPrep”目录复制到 SATA 驱动器准备连接的副本计算机上。 确保满足以下与副本计算机相关的要求：

   * 副本计算机可使用在**启动脱机备份**工作流中提供的相同网络路径，访问脱机种子设定工作流的暂存位置。
   * 已在副本计算机上启用 BitLocker。
   * 副本计算机可以访问 Azure 门户。

     必要时，副本计算机可与源计算机相同。 
    
     > [!IMPORTANT] 
     > 如果源计算机是虚拟机，则它必须使用其他物理服务器或客户端计算机作为副本计算机。
    
    
2. 在副本计算机上打开提升的命令提示符，以 AzureOfflineBackupDiskPrep 实用工具的目录作为当前目录并运行以下命令：

    `*.\AzureOfflineBackupDiskPrep.exe*   s:<*Staging Location Path*>   [p:<*Path to AzurePublishSettingsFile*>]`

    | 参数 | 描述 |
    | --- | --- |
    | s:&lt;暂存位置路径&gt; |必需的输入，用于提供在 **启动脱机备份** 工作流中所输入的暂存位置路径。 |
    | p:&lt;发布设置文件的路径&gt; |可选的输入，用于提供在**启动脱机备份**工作流中所输入的 **Azure 发布设置**文件路径。 |

    > [!NOTE]
    > 复制计算机与源计算机不同时，&lt;AzurePublishSettingFile 的路径&gt; 值是必需的。
    >
    >

    运行该命令时，该实用工具将请求选择需要准备的驱动器对应的 Azure 导入作业。 如果只有一个与提供的暂存位置关联的导入作业，会显示如下所示的屏幕。

    ![Azure 磁盘准备工具输入](./media/backup-azure-backup-import-export/azureDiskPreparationToolDriveInput.png) <br/>
    
3. 输入想要准备传输到 Azure 的已装载磁盘的驱动器号（不要包含尾部的冒号）。 出现提示时，请确认格式化驱动器。

    工具随后便开始准备磁盘和复制备份数据。 可能需要工具的提示附加其他磁盘，以免提供的磁盘没有足够空间来容纳备份数据。 <br/>

    在工具成功执行结束时，所提供的一个或多个磁盘便已准备好可以寄送到 Azure。 此外，还会在 Azure 中创建导入作业并使用在启动脱机备份工作流期间提供的名称来命名该作业。 最后，工具上还显示磁盘所要寄送到的 Azure 数据中心的寄送地址。

    ![Azure 磁盘准备已完成](./media/backup-azure-backup-import-export/azureDiskPreparationToolSuccess.png)<br/>
    
4. 在命令执行末尾还会看到更新寄送信息的选项，如下所示：

    ![更新寄送信息选项](./media/backup-azure-backup-import-export/updateshippingutility.png)<br/>

5. 可以立即输入详细信息。 该工具将指导完成涉及一系列输入的过程。 但如果你还没有追踪号码或其他与寄送相关的详细信息，则可以结束会话。 这篇文章介绍了随后更新寄送详情的操作步骤。 

6. 将磁盘寄送到工具提供的地址，保留跟踪号码供日后参考。

   > [!IMPORTANT] 
   > 两个 Azure 导入作业不能同时拥有相同的追踪号码。 确保使用一个包裹寄送实用工具在单次 Azure 导入作业中准备的驱动器，该包裹有一个唯一的追踪号码。 请勿在一个包裹中混合不同 Azure 导入作业中准备的驱动器。 

5. 如果具有追踪号码信息，请转到正在等待导入作业完成的源计算机，然后在提升的命令提示符中运行以下命令，以 AzureOfflineBackupDiskPrep 实用工具的目录作为当前目录： 

   `*.\AzureOfflineBackupDiskPrep.exe*  u:`

   可以选择从副本计算机等其他计算机运行以下命令，以 AzureOfflineBackupDiskPrep 实用工具的目录作为当前目录：
   
   `*.\AzureOfflineBackupDiskPrep.exe*  u:  s:<*Staging Location Path*>   p:<*Path to AzurePublishSettingsFile*>`

    | 参数 | 描述 |
    | --- | --- |
    | u: | 用于更新 Azure 导入作业的寄送详情的必需输入 |
    | s:&lt;暂存位置路径&gt; | 命令不在源计算机上运行时的必需输入。 用于提供在“启动脱机备份”工作流中所输入的暂存位置路径。 |
    | p:&lt;发布设置文件的路径&gt; | 命令不在源计算机上运行时的必需输入。 用于提供在“启动脱机备份”工作流中所输入的“Azure 发布设置”文件路径。 |
    
    该实用工具会自动检测到源计算机正在等待的导入作业，或在另一台计算机上运行命令时，与暂存位置相关的导入作业。 随后它通过一系列输入提供更新寄送信息的选项，如下所示： 
    
    ![输入寄送信息](./media/backup-azure-backup-import-export/shippinginputs.png)<br/>

6. 提供所有的输入后，请仔细查看详情，并通过键入“yes”提交提供的寄送信息。 

    ![查看寄送信息](./media/backup-azure-backup-import-export/reviewshippinginformation.png)<br/>

7. 成功更新寄送信息后，该实用工具提供你输入的寄送详情存储的本地位置，如下所示 

    ![存储寄送信息](./media/backup-azure-backup-import-export/storingshippinginformation.png)<br/>

   > [!IMPORTANT] 
   > 确保驱动器在使用 AzureOfflineBackupDiskPrep 实用工具提供寄送信息后两周内送达 Azure 数据中心。 如果不能做到，可能会导致驱动器得不到处理。  

完成上述步骤后，Azure 数据中心已准备好接收驱动器并作进一步处理，将备份数据从驱动器传输到你创建的经典类型的 Azure 存储帐户。 

### <a name="time-to-process-the-drives"></a>处理驱动器的时间 
处理 Azure 导入作业的时间各不相同，具体取决于不同的因素，例如寄送时间、作业类型、要复制的数据的类型和大小，以及所提供磁盘的大小。 Azure 导入/导出服务没有 SLA，但在收到磁盘之后，该服务力求在 7 到 10 天内完成将备份数据复制到 Azure 存储帐户。 下一节会详细介绍如何监视 Azure 导入作业的状态。 

### <a name="monitoring-azure-import-job-status"></a>监视 Azure 导入作业状态
驱动器在传输过程中或已在 Azure 数据中心，等待复制到存储帐户时，Azure 备份代理或 SC DPM 或源计算机上的 Azure 备份服务器控制台将为你的计划备份显示以下作业状态。 

  `Waiting for Azure Import Job to complete. Please check on Azure Management portal for more information on job status`

遵循以下步骤，查看导入作业状态。 
1. 在源计算机上打开提升的命令提示符，并运行以下命令：
    
     `AzureOfflineBackupDiskPrep.exe u:`
    
2.  输出显示导入作业的当前状态，如下所示： 

    ![查看导入作业状态](./media/backup-azure-backup-import-export/importjobstatusreporting.png)<br/>

有关 Azure 导入作业的各种状态的详细信息，请参阅[这篇文章](../storage/common/storage-import-export-view-drive-status.md)

### <a name="complete-the-workflow"></a>完成工作流
导入作业完成后，存储帐户中的初始备份数据可供使用。 在进行到下一步计划的备份时，Azure 备份将数据的内容从存储帐户复制到恢复服务保管库，如下所示： 

   ![将数据复制到恢复服务保管库](./media/backup-azure-backup-import-export/copyingfromstorageaccounttoazurebackup.png)<br/>

在进行到下一个计划备份时，Azure 备份通过初始备份副本执行增量备份。

## <a name="next-steps"></a>后续步骤
* 如有任何关于 Azure 导入/导出工作流的问题，请参阅 [Use the Microsoft Azure Import/Export service to transfer data to Blob storage](../storage/common/storage-import-export-service.md)（使用 Microsoft Azure 导入/导出服务可将数据传输到 Blob 存储中）。
* 如有工作流方面的任何问题，请参阅 Azure 备份 [FAQ](backup-azure-backup-faq.md)（常见问题）的“脱机备份”部分。
