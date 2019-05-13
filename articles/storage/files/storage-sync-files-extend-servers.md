---
title: 教程 - 使用 Azure 文件同步扩展 Windows 文件服务器 | Microsoft Docs
description: 全面了解如何使用 Azure 文件同步扩展 Windows 文件服务器。
services: storage
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 10/23/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: df3850a839ac789957a9adffb7122a0b58987781
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64705061"
---
# <a name="tutorial-extend-windows-file-servers-with-azure-file-sync"></a>教程：使用 Azure 文件同步扩展 Windows 文件服务器

本文介绍使用 Azure 文件同步扩展 Windows Server 存储容量的基本步骤。虽然本教程讲述充当 Azure 虚拟机 (VM) 的 Windows Server，但通常会针对本地服务器执行此过程。 若要了解如何在自己的环境中部署 Azure 文件同步，可参阅[部署 Azure 文件同步](storage-sync-files-deployment-guide.md)一文。

> [!div class="checklist"]
> * 部署存储同步服务
> * 准备 Windows Server，用于 Azure 文件同步
> * 安装 Azure 文件同步代理
> * 将 Windows Server 注册到存储同步服务
> * 创建同步组和云终结点
> * 创建服务器终结点

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure"></a>登录 Azure

登录到 [Azure 门户](https://portal.azure.com)。

## <a name="prepare-your-environment"></a>准备环境

就本教程来说，需先执行以下操作，然后才能部署 Azure 文件同步：

- 创建 Azure 存储帐户和文件共享
- 设置 Windows Server 2016 Datacenter VM
- 准备用于 Azure 文件同步的 Windows Server VM

### <a name="create-a-folder-and-txt-file"></a>创建文件夹和 .txt 文件

请在本地计算机上创建名为 _FilesToSync_ 的新文件夹，并添加名为 _mytestdoc.txt_ 的文本文件。 在本教程后面需将该文件上传到文件共享。

### <a name="create-a-storage-account"></a>创建存储帐户

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

### <a name="create-a-file-share"></a>创建文件共享

部署 Azure 存储帐户以后，创建一个文件共享。

1. 在 Azure 门户中，选择“转到资源”。
1. 在存储帐户窗格中选择“文件”。

    ![选择文件](./media/storage-sync-files-extend-servers/click-files.png)

1. 选择“+ 文件共享”。

    ![选择”添加文件共享”按钮](./media/storage-sync-files-extend-servers/create-file-share-portal2.png)

1. 将新文件共享命名为 _afsfileshare_。 输入“1”作为**配额**，然后选择“创建”。 配额最高可以为 5 TiB，但本教程只需 1 GB。

    ![为新文件共享提供名称和配额](./media/storage-sync-files-extend-servers/create-file-share-portal3.png)

1. 选择新文件共享。 在文件共享位置选择“上传”。

    ![上传文件](./media/storage-sync-files-extend-servers/create-file-share-portal5.png)

1. 浏览到在其中创建了 .txt 文件的 _FilesToSync_ 文件夹，选择 _mytestdoc.txt_，然后选择“上传”。

    ![浏览文件共享](./media/storage-sync-files-extend-servers/create-file-share-portal6.png)

目前已创建存储帐户和包含一个文件的文件共享。 接下来需部署 Azure VM，其中的 Windows Server 2016 Datacenter 代表本教程中的本地服务器。

### <a name="deploy-a-vm-and-attach-a-data-disk"></a>部署 VM 并附加数据磁盘

1. 转到 Azure 门户，展开左侧的菜单。 在左上角选择“创建资源”。
1. 在 **Azure 市场**资源列表上方的搜索框中搜索“Windows Server 2016 Datacenter”，然后在结果中将其选中。 选择“创建”。
1. 转到“基本信息”选项卡。在“项目详细信息”下，选择为本教程创建的资源组。

   ![在门户边栏选项卡上输入 VM 的基本信息](./media/storage-sync-files-extend-servers/vm-resource-group-and-subscription.png)

1. 在“实例详细信息”下提供一个 VM 名称。 例如，使用 _myVM_ 作为名称。
1. 请勿更改“区域”、“可用性选项”、“映像”和“大小”的默认设置。
1. 在“管理员帐户”下，提供 VM 的“用户名”和“密码”。
1. 在“入站端口规则”下，选择“允许所选端口”，然后从下拉菜单中选择“RDP (3389)”和“HTTP”。

1. 在创建 VM 之前，需创建数据磁盘。

   1. 选择“下一步:磁盘”。

      ![添加数据磁盘](./media/storage-sync-files-extend-servers/vm-add-data-disk.png)

   1. 在“磁盘”选项卡的“磁盘选项”下，保留默认设置。
   1. 在“数据磁盘”下，选择“创建并附加新磁盘”。

   1. 使用默认设置，但“大小(GiB)”选项除外，在本教程中，可将该选项更改为“1 GB”。

      ![数据磁盘详细信息](./media/storage-sync-files-extend-servers/vm-create-new-disk-details.png)

   1. 选择“确定”。
1. 选择“查看 + 创建”。
1. 选择“创建”。

   可以选择“通知”图标，观察**部署进度**。 创建新的 VM 可能需要数分钟才能完成。

1. VM 部署完成以后，请选择“转到资源”。

   ![转到资源](./media/storage-sync-files-extend-servers/vm-gotoresource.png)

目前已创建一个新虚拟机并附加一个数据磁盘。 接下来，请连接到 VM。

### <a name="connect-to-your-vm"></a>连接到 VM

1. 在 Azure 门户中，选择虚拟机属性页上的“连接”。

   ![从门户连接到 Azure VM](./media/storage-sync-files-extend-servers/connect-vm.png)

1. 在“连接到虚拟机”页面上，请保留默认选项，以使用 IP 地址通过端口 3389 进行连接。 选择“下载 RDP 文件”。

   ![下载 RDP 文件](./media/storage-sync-files-extend-servers/download-rdp.png)

1. 打开下载的 RDP 文件，然后在出现提示时选择“连接”。
1. 在“Windows 安全性”窗口中，依次选择“更多选择”、“使用其他帐户”。 以 *localhost\username* 形式键入用户名，输入为虚拟机创建的密码，然后选择“确定”。

   ![更多选择](./media/storage-sync-files-extend-servers/local-host2.png)

1. 你可能会在登录过程中收到证书警告。 选择“是”或“继续”以创建连接。

### <a name="prepare-the-windows-server"></a>准备 Windows Server

对于 Windows Server 2016 Datacenter 服务器，请禁用“Internet Explorer 增强的安全配置”。 只有在进行初始的服务器注册时，此步骤才是必要步骤。 可在注册服务器后重新启用。

在 Windows Server 2016 Datacenter VM 中，服务器管理器会自动打开。  如果服务器管理器不默认打开，请在文件资源管理器中搜索它。

1. 在“服务器管理器”中，选择“本地服务器”。

   ![服务器管理器 UI 左侧的“本地服务器”](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-1.png)

1. 在“属性”窗格上，选择“IE 增强的安全性配置”的链接。  

    ![服务器管理器 UI 中的“IE 增强的安全配置”窗格](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-2.png)

1. 对于“Internet Explorer 增强的安全配置”对话框中的“管理员”和“用户”，都选择“关”。

    ![选定“关”的“Internet Explorer 增强的安全配置”弹出窗口](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-3.png)

现在可以将数据磁盘添加到 VM。

### <a name="add-the-data-disk"></a>添加数据磁盘

1. 仍在 **Windows Server 2016 Datacenter** VM 中时，请选择“文件和存储服务” > “卷” > “磁盘”。

    ![数据磁盘](media/storage-sync-files-extend-servers/your-disk.png)

1. 右键单击名为“Msft 虚拟磁盘”的 1 GB 磁盘，然后选择“新建卷”。
1. 完成该向导。 使用默认设置并记下分配的驱动器号。
1. 选择“创建”。
1. 选择“关闭”。

   目前已将磁盘联机并创建一个卷。 在 Windows Server VM 中打开文件资源管理器，确认存在最近添加的数据磁盘。

1. 在 VM 的文件资源管理器中展开“此电脑”，然后打开新驱动器。 在此示例中，它是 F: 驱动器。
1. 右键单击并选择“新建” > “文件夹”。 将文件夹命名为 _FilesToSync_。
1. 打开 **FilesToSync** 文件夹。
1. 右键单击并选择“新建” > “文本文档”。 将文本文件命名为 _MyTestFile_。

    ![添加新的文本文件](media/storage-sync-files-extend-servers/new-file.png)

1. 关闭**文件资源管理器**和**服务器管理器**。

### <a name="download-the-azure-powershell-module"></a>下载 Azure PowerShell 模块

接下来，在 Windows Server 2016 Datacenter VM 的服务器上安装 Azure PowerShell 模块。

1. 在 VM 中打开提升的 PowerShell 窗口。
1. 运行以下命令：

   ```powershell
   Install-Module -Name Az
   ```

   > [!NOTE]
   > 如果使用的 NuGet 版本低于 2.8.5.201，系统会提示下载并安装最新版本的 NuGet。

   默认情况下，PowerShell 库未配置为 PowerShellGet 的受信任存储库。 首次使用 PSGallery 时会看到以下提示：

   ```output
   Untrusted repository

   You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet.

   Are you sure you want to install the modules from 'PSGallery'?
   [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
   ```

1. 回答“是”或“全部确认”，可继续进行安装。

`Az` 模块是 Azure PowerShell cmdlet 的汇总模块。 安装它时，系统会下载所有可用的 Azure 资源管理器模块并使其 cmdlet 可供使用。

此时，你已设置本教程的环境。 可以部署存储同步服务了。

## <a name="deploy-the-service"></a>部署服务

若要部署 Azure 文件同步，请首先将一个“存储同步服务”资源置于所选订阅的资源组中。 存储同步服务从其订阅和资源组继承访问权限。

1. 在 Azure 门户中选择“创建资源”，然后搜索“Azure 文件同步”。
1. 在搜索结果中，选择“Azure 文件同步”。
1. 选择“创建”，打开“部署存储同步”选项卡。

   ![部署存储同步](media/storage-sync-files-extend-servers/afs-info.png)

   在打开的窗格中，输入以下信息：

   | 值 | 说明 |
   | ----- | ----- |
   | **名称** | 存储同步服务的唯一名称（按订阅）。<br><br>就本教程来说，请使用 _afssyncservice02_。 |
   | **订阅** | 用于本教程的 Azure 订阅。 |
   | **资源组** | 包含存储同步服务的资源组。<br><br>就本教程来说，请使用 _afsresgroup101918_。 |
   | **位置** | 美国东部 |

1. 完成后请选择“创建”，部署**存储同步服务**。
1. 选择“通知”选项卡 >“转到资源”。

## <a name="install-the-agent"></a>安装代理

Azure 文件同步代理是一个可下载包，可实现 Windows 服务器与 Azure 文件共享的同步。

1. 在 **Windows Server 2016 Datacenter** VM 中，打开 **Internet Explorer**。
1. 访问 [Microsoft 下载中心](https://go.microsoft.com/fwlink/?linkid=858257)。 向下滚动到“Azure 文件同步代理”部分并选择“下载”。

   ![同步代理下载](media/storage-sync-files-extend-servers/sync-agent-download.png)

1. 选择 **StorageSyncAgent_V3_WS2016.EXE** 对应的复选框，然后选择“下一步”。

   ![选择代理](media/storage-sync-files-extend-servers/select-agent.png)

1. 选择“允许一次” > “运行” > “打开”。
1. 如果尚未关闭 PowerShell 窗口，请将其关闭。
1. 接受**存储同步代理设置向导**中的默认设置。
1. 选择“安装”。
1. 选择“完成”。

你已在 Windows Server 2016 Datacenter VM 上部署 Azure 同步服务并安装代理。 现在需将 VM 注册到存储同步服务。

## <a name="register-windows-server"></a>注册 Windows Server

向存储同步服务注册 Windows Server 可在服务器（或群集）与存储同步服务之间建立信任关系。 一个服务器只能注册到一个存储同步服务。 它可以与存储同步服务所关联的其他服务器和 Azure 文件共享同步。

服务器注册 UI 会在 Azure 文件同步代理安装后自动打开。 如果没有，可以从其文件位置手动将其打开：`C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe.`

1. 服务器注册 UI 在 VM 中打开后，请选择“确定”。
1. 选择“登录”即可开始操作。
1. 使用 Azure 帐户凭据登录，然后选择“登录”。
1. 提供以下信息：

   ![服务器注册 UI 的屏幕快照](media/storage-sync-files-extend-servers/signin.png)

   | | |
   | ----- | ----- |
   | 值 | 说明 |
   | **Azure 订阅** | 包含本教程的存储同步服务的订阅。 |
   | **资源组** | 包含存储同步服务的资源组。 就本教程来说，请使用 _afsresgroup101918_。 |
   | **存储同步服务** | 存储同步服务的名称。 就本教程来说，请使用 _afssyncservice02_。 |

1. 选择“注册”，完成服务器注册。
1. 在注册过程中，系统会提示进行其他登录。 登录并选择“下一步”。
1. 选择“确定”。

## <a name="create-a-sync-group"></a>创建同步组

同步组定义一组文件的同步拓扑。 同步组中必须包含一个表示 Azure 文件共享的云终结点。 此外，同步组必须包含一个或多个服务器终结点。 服务器终结点表示已注册的服务器上的路径。 若要创建同步组，请执行以下操作：

1. 在 [Azure 门户](https://portal.azure.com/)的存储同步服务中选择“+ 同步组”。 就本教程来说，请使用 *afssyncservice02*。

   ![在 Azure 门户中创建新的同步组](media/storage-sync-files-extend-servers/add-sync-group.png)

1. 输入以下信息，创建具有云终结点的同步组：

   | 值 | 说明 |
   | ----- | ----- |
   | **同步组名称** | 此名称在存储同步服务内必须是唯一的，但可以是符合逻辑的任何名称。 就本教程来说，请使用 *afssyncgroup*。|
   | **订阅** | 在其中部署了本教程的存储同步服务的订阅。 |
   | **存储帐户** | 选择“选择存储帐户”。 在显示的窗格中，选择包含创建的 Azure 文件共享的存储帐户。 就本教程来说，请使用 *afsstoracct101918*。 |
   | **Azure 文件共享** | 创建的 Azure 文件共享的名称。 就本教程来说，请使用 *afsfileshare*。 |

1. 选择“创建”。

如果选择同步组，则可看到现在有一个**云终结点**。

## <a name="add-a-server-endpoint"></a>添加服务器终结点

服务器终结点表示已注册的服务器上的特定位置。 例如，服务器卷上的文件夹。 若要添加服务器终结点，请执行以下操作：

1. 选择新创建的同步组，然后选择“添加服务器终结点”。

   ![在“同步组”窗格中添加一个新的服务器终结点](media/storage-sync-files-extend-servers/add-server-endpoint.png)

1. 在“添加服务器终结点”窗格上，输入以下信息，创建服务器终结点：

   | | |
   | ----- | ----- |
   | 值 | 说明 |
   | **已注册的服务器** | 创建的服务器的名称。 就本教程来说，请使用 *afsvm101918*。 |
   | **路径** | 创建的驱动器的 Windows Server 路径。 在本教程中，请使用 *f:\filestosync*。 |
   | **云分层** | 在本教程中，保留为禁用状态。 |
   | **卷可用空间** | 在本教程中，保留为空白。 |

1. 选择“创建”。

现在，文件在 Azure 文件共享和 Windows Server 之间同步。

![Azure 存储已成功同步](media/storage-sync-files-extend-servers/files-synced-in-azurestorage.png)

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>后续步骤

本教程介绍了使用 Azure 文件同步扩展 Windows Server 存储容量的基本步骤。若要更详尽地了解如何针对 Azure 文件同步部署进行规划，请参阅：

> [!div class="nextstepaction"]
> [规划 Azure 文件同步部署](./storage-sync-files-planning.md)
