---
title: 准备和自定义主 VHD 映像-Azure
description: 了解如何准备、 自定义并将 Windows 虚拟桌面预览主映像上传到 Azure。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 04/03/2019
ms.author: helohr
ms.openlocfilehash: 58471dc539f72c49b041638e928dda751f4bf5a2
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410594"
---
# <a name="prepare-and-customize-a-master-vhd-image"></a>准备和自定义主 VHD 映像

本文将介绍如何准备要上传到 Azure，包括如何创建虚拟机 (Vm) 和安装并在其上配置软件的主虚拟硬盘 (VHD) 映像。 这些说明适用于可以与组织的现有进程使用的特定于 Windows 的虚拟桌面预览版的配置。

## <a name="create-a-vm"></a>创建 VM

Windows 10 企业版多会话是 Azure 映像库中提供的。 有两个选项用于自定义此映像。

第一个选项是设置虚拟机 (VM) 在 Azure 中的说明[从托管映像创建 VM](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-generalized-managed)，，然后跳到[软件准备和安装](set-up-customize-master-image.md#software-preparation-and-installation)。

第二个选项是通过将映像下载、 预配的 HYPER-V VM，并自定义它以满足您的需求，我们将介绍下一节中本地创建的映像。

### <a name="local-image-creation"></a>创建本地映像

下载到本地位置的映像后，打开**Hyper-v 管理器**以使用刚刚复制的 VHD 创建 VM。 以下是最简单的版本，但您可以找到更多详细的说明中[在 HYPER-V 中创建的虚拟机](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v)。

若要使用复制的 VHD 创建 VM:

1. 打开**新的虚拟机向导**。

2. 在指定代数页上，选择**第 1 代**。

    ![指定代数页的屏幕截图。 选择"第 1 代"选项。](media/a41174fd41302a181e46385e1e701975.png)

3. 在检查点类型下，通过取消选中复选框来禁用检查点。

    ![检查点页上的检查点类型部分的屏幕截图。](media/20c6dda51d7cafef33251188ae1c0c6a.png)

此外可以禁用检查点在 PowerShell 中运行以下 cmdlet。

```powershell
Set-VM -Name <VMNAME> -CheckpointType Disabled
```

### <a name="fixed-disk"></a>固定的磁盘

如果从现有 VHD 创建 VM，则默认情况下创建的动态磁盘。 它可以为固定磁盘更改，通过选择**编辑磁盘...** 下图中所示。 有关详细说明，请参阅[准备要上传到 Azure 的 Windows VHD 或 VHDX](https://docs.microsoft.com/azure/virtual-machines/windows/prepare-for-upload-vhd-image)。

![编辑磁盘选项的屏幕截图。](media/35772414b5a0f81f06f54065561d1414.png)

此外可以运行以下 PowerShell cmdlet 将磁盘更改为固定磁盘。

```powershell
Convert-VHD –Path c:\\test\\MY-VM.vhdx –DestinationPath c:\\test\\MY-NEW-VM.vhd -VHDType Fixed
```

## <a name="software-preparation-and-installation"></a>软件准备和安装

本部分介绍如何准备和安装 office 365 专业增强版、 OneDrive、 FSLogix、 Windows Defender 和其他常见的应用程序。 如果你的用户需要访问某些 LOB 应用程序，我们建议在完成本部分说明后安装。

本部分假定您拥有提升访问权限的 VM 上，无论它在 Azure 或 Hyper-v 管理器预配。

### <a name="install-office-in-shared-computer-activation-mode"></a>在共享的计算机激活模式下安装 Office

使用[Office 部署工具](https://www.microsoft.com/download/details.aspx?id=49117)来安装 Office。 Windows 10 企业版多会话仅支持 Office 365 专业增强版，不 Office 2019 永久型。

Office 部署工具需要配置 XML 文件。 若要自定义下面的示例，请参阅[Office 部署工具的配置选项](https://docs.microsoft.com/deployoffice/configuration-options-for-the-office-2016-deployment-tool)。

此示例配置 XML，我们提供了将执行以下操作：

- 从预览体验成员频道安装 Office，并将更新从预览体验成员通道传输时就会执行它们。
- 使用 x64 体系结构。
- 禁用自动更新。
- 安装 Visio 和项目。
- 删除任何现有的 Office 安装，并将其设置迁移。
- 启用共享的计算机的终端服务器环境中为操作授权。

下面是此示例配置 XML 不会执行的操作：

- 安装 Skype for Business
- 在每个用户模式下安装 OneDrive。 若要了解详细信息，请参阅[每个计算机模式下安装 OneDrive](#install-onedrive-in-per-machine-mode)。

>[!NOTE]
>共享计算机授权可以设置通过组策略对象 (Gpo) 或注册表设置。 GPO 位于**计算机配置\\策略\\管理模板\\Microsoft Office 2016 （计算机）\\授权设置**

Office 部署工具包含 setup.exe。 若要安装 Office，请在命令行中运行以下命令：

```batch
Setup.exe /configure configuration.xml
```

#### <a name="sample-configurationxml"></a>示例 configuration.xml

下面的 XML 示例将安装预览体验成员版本，也称为快速预览体验成员或预览体验成员 Main。

```xml
<Configuration>
    <Add OfficeClientEdition="64" SourcePath="https://officecdn.microsoft.com/pr/5440fd1f-7ecb-4221-8110-145efaa6372f">
        <Product ID="O365ProPlusRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Groove" />
            <ExcludeApp ID="Lync" />
            <ExcludeApp ID="OneDrive" />
            <ExcludeApp ID="Teams" />
        </Product>
        <Product ID="VisioProRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Teams" /> 
        </Product>
        <Product ID="ProjectProRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Teams" />
        </Product>
    </Add>
    <RemoveMSI All="True" />
    <Updates Enabled="FALSE" UpdatePath="https://officecdn.microsoft.com/pr/5440fd1f-7ecb-4221-8110-145efaa6372f" />
    <Display Level="None" AcceptEULA="TRUE" />
    <Logging Level="Verbose" Path="%temp%\WVDOfficeInstall" />
    <Property Value="TRUE" Name="FORCEAPPSHUTDOWN"/>
    <Property Value="1" Name="SharedComputerLicensing"/>
    <Property Value="TRUE" Name="PinIconsToTaskbar"/>
</Configuration>
```

>[!NOTE]
>Office 团队建议使用 64 位安装程序**OfficeClientEdition**参数。

安装 Office 之后, 可以更新默认 Office 行为。 单独或批处理文件，若要更新的行为，请运行以下命令。

```batch
rem Mount the default user registry hive
reg load HKU\TempDefault C:\Users\Default\NTUSER.DAT
rem Must be executed with default registry hive mounted.
reg add HKU\TempDefault\SOFTWARE\Policies\Microsoft\office\16.0\common /v InsiderSlabBehavior /t REG_DWORD /d 2 /f
rem Set Outlook's Cached Exchange Mode behavior
rem Must be executed with default registry hive mounted.
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v enable /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v syncwindowsetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSettingMonths  /t REG_DWORD /d 1 /f
rem Unmount the default user registry hive
reg unload HKU\TempDefault

rem Set the Office Update UI behavior.
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideupdatenotifications /t REG_DWORD /d 1 /f
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideenabledisableupdates /t REG_DWORD /d 1 /f
```

### <a name="disable-automatic-updates"></a>禁用自动更新

若要禁用本地组策略通过自动更新：

1. 打开**本地组策略编辑器\\管理模板\\Windows 组件\\Windows Update**。
2. 右键单击**配置自动更新**并将其设置为**禁用**。

此外可以在命令提示符下，禁用自动更新上运行以下命令。

```batch
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU /v NoAutoUpdate /t REG_DWORD /d 1 /f
```

### <a name="specify-start-layout-for-windows-10-pcs-optional"></a>Windows 10 电脑 （可选） 为指定开始布局

运行此命令适用于 Windows 10 电脑指定开始布局。

```batch
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer" /v SpecialRoamingOverrideAllowed /t REG_DWORD /d 1 /f
```

### <a name="install-onedrive-in-per-machine-mode"></a>在每台计算机模式下安装 OneDrive

OneDrive 是通常情况下已安装的每个用户。 在此环境中，它应安装每台计算机。

下面介绍了如何在每台计算机模式下安装 OneDrive:

1. 首先，创建用于暂存 OneDrive 安装程序的位置。 本地磁盘文件夹或 [\\\\unc] (file://unc) 位置很好。

2. 到暂存位置与此链接下载 OneDriveSetup.exe: <https://aka.ms/OneDriveWVD-Installer>

3. 如果使用 OneDrive 中省略安装 office  **\<ExcludeApp ID ="OneDrive"/\>**，通过运行以下命令，从提升的命令提示符卸载任何现有的 OneDrive 每用户安装命令：
    
    ```batch
    "[staged location]\OneDriveSetup.exe" /uninstall
    ```

4. 从提升的命令提示符设置运行此命令**AllUsersInstall**注册表值：

    ```batch
    REG ADD "HKLM\Software\Microsoft\OneDrive" /v "AllUsersInstall" /t REG_DWORD /d 1 /reg:64
    ```

5. 运行以下命令在每台计算机模式下安装 OneDrive:

    ```batch
    Run "[staged location]\OneDriveSetup.exe" /allusers
    ```

6. 运行以下命令来配置 OneDrive 的所有用户登录时启动：

    ```batch
    REG ADD "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDrive /t REG_SZ /d "C:\Program Files (x86)\Microsoft OneDrive\OneDrive.exe /background" /f
    ```

7. 启用**以无提示方式将用户帐户配置**通过运行以下命令。

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "SilentAccountConfig" /t REG_DWORD /d 1 /f
    ```

8. 重定向和移动 Windows 通过运行以下命令的已知文件夹复制到 OneDrive。

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "KFMSilentOptIn" /t REG_SZ /d "<your-AzureAdTenantId>" /f
    ```

### <a name="teams-and-skype"></a>团队和 Skype

Windows 虚拟桌面不正式支持适用于业务和团队 Skype。

### <a name="set-up-user-profile-container-fslogix"></a>设置用户配置文件容器 (FSLogix)

若要包含 FSLogix 容器映像的一部分，按照中的说明[主机池将用户配置文件共享设置](create-host-pools-user-profile.md#configure-the-fslogix-profile-container)。 你可以测试 FSLogix 容器使用的功能[本快速入门](https://docs.fslogix.com/display/20170529/Profile+Containers+-+Quick+Start)。

### <a name="configure-windows-defender"></a>配置 Windows Defender

如果在 VM 中配置 Windows Defender，请确保它已配置为不扫描 VHD 和 VHDX 文件的全部内容在附件的相同过程。

此配置仅删除 VHD 和 VHDX 文件的扫描期间附件，但不会影响实时扫描。

有关详细说明如何在 Windows Server 上配置 Windows Defender 的详细信息，请参阅[Windows Server 上的配置 Windows Defender 防病毒排除项](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-antivirus/configure-server-exclusions-windows-defender-antivirus)。

若要了解有关如何配置 Windows Defender，若要从扫描中排除某些文件的详细信息，请参阅[配置和验证基于文件扩展名和文件夹位置的排除项](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-antivirus/configure-extension-file-exclusions-windows-defender-antivirus)。

### <a name="configure-session-timeout-policies"></a>配置会话超时策略

由于在主机池中的所有 Vm 都是相同的安全组的成员可以在组策略级别上强制执行远程会话策略。

若要配置远程会话策略：

1. 导航到**管理模板** > **Windows 组件** > **远程桌面服务** >  **远程桌面会话主机** > **会话时间限制**。
2. 在右侧窗格中，选择**设置活动但是空闲的远程桌面服务会话的时间限制**策略。
3. 模式窗口出现后，更改的策略选项**未配置**到**已启用**要激活该策略。
4. 在策略选项下的下拉列表菜单，将设置到的时间量**4 小时**。

您可以手动配置远程会话策略，通过运行以下命令：

```batch
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v RemoteAppLogoffTimeLimit /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fResetBroken /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v MaxConnectionTime /t REG_DWORD /d 10800000 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v RemoteAppLogoffTimeLimit /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v MaxDisconnectionTime /t REG_DWORD /d 5000 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v MaxIdleTime /t REG_DWORD /d 7200000 /f
```

### <a name="set-up-time-zone-redirection"></a>设置时区重定向

由于在主机池中的所有 Vm 都是相同的安全组的成员可以在组策略级别上强制执行时区重定向。

若要重定向时区：

1. 在 Active Directory 服务器上，打开**组策略管理控制台**。
2. 展开你的域和组策略对象。
3. 右键单击**组策略对象**创建的组策略设置并选择**编辑**。
4. 在中**组策略管理编辑器**，导航到**计算机配置** > **策略** > **管理模板** > **Windows 组件** > **远程桌面服务** > **远程桌面会话主机**  > **设备和资源重定向**。
5. 启用**允许时区重定向**设置。

此外可以在主映像重定向的时区上运行此命令：

```batch
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
```

### <a name="disable-storage-sense"></a>禁用存储感知

对于使用 Windows 10 企业版或 Windows 10 企业版多会话的 Windows 虚拟桌面会话主机，我们建议禁用存储有意义。 可以下的设置菜单中禁用存储感知**存储**，如以下屏幕截图中所示：

![在设置下的存储菜单的屏幕截图。 "存储感知"选项处于关闭状态。](media/storagesense.png)

此外可以通过运行以下命令来更改注册表设置：

```batch
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy /v 01 /t REG_DWORD /d 0 /f
```

### <a name="include-additional-language-support"></a>包括其他语言支持

本文未介绍如何配置语言和区域支持。 有关详细信息，请参阅以下文章：

- [将语言添加到 Windows 映像](https://docs.microsoft.com/windows-hardware/manufacture/desktop/add-language-packs-to-windows)
- [按需功能](https://docs.microsoft.com/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities)
- [按需 (FOD) 的语言和区域的功能](https://docs.microsoft.com/windows-hardware/manufacture/desktop/features-on-demand-language-fod)

### <a name="other-applications-and-registry-configuration"></a>其他应用程序和注册表配置

本部分介绍应用程序和操作系统配置。 本部分中的所有配置都通过可以通过命令行执行的注册表项和 regedit 工具。

>[!NOTE]
>您可以使用常规策略对象 (Gpo) 或注册表导入配置中实现最佳实践。 管理员可以选择任一选项根据其组织的要求。

有关反馈中心的遥测数据收集在 Windows 10 企业版多会话上，运行以下命令：

```batch
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DataCollection "AllowTelemetry"=dword:00000003
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DataCollection /v AllowTelemetry /d 3
```

运行以下命令以修复崩溃的 Watson:

```batch
remove CorporateWerServer* from Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting
```

若要修复 5 k 解决支持在注册表编辑器中输入以下命令。 可以启用通过并行堆栈之前，必须运行命令。

```batch
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp]
"MaxMonitors"=dword:00000004
"MaxXResolution"=dword:00001400
"MaxYResolution"=dword:00000b40

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs]
"MaxMonitors"=dword:00000004
"MaxXResolution"=dword:00001400
"MaxYResolution"=dword:00000b40
```

## <a name="prepare-the-image-for-upload-to-azure"></a>准备要上传到 Azure 映像

已完成配置并安装所有应用程序后，请按照中的说明[准备要上传到 Azure 的 Windows VHD 或 VHDX](https://docs.microsoft.com/azure/virtual-machines/windows/prepare-for-upload-vhd-image)来准备映像。

在准备好后此图像可查看上传，请确保 VM 处于关闭或已解除分配状态。

## <a name="upload-master-image-to-a-storage-account-in-azure"></a>将主映像上传到 Azure 中的存储帐户

本节仅适用于主映像创建本地时。

以下说明将告诉您如何将主映像上传到 Azure 存储帐户。 如果还没有 Azure 存储帐户，请按照中的说明[这篇文章](https://code.visualstudio.com/tutorials/static-website/create-storage)创建一个。

1. 如果你尚未准备好，请将 VM 映像 (VHD) 转换为固定。 如果不将图像转换为固定，不能成功创建映像。

2. 将 VHD 上传到你的存储帐户中的 blob 容器。 可以使用快速上传[存储资源管理器工具](https://azure.microsoft.com/features/storage-explorer/)。 若要了解有关存储资源管理器工具的详细信息，请参阅[这篇文章](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows)。

    ![Microsoft Azure 存储资源管理器工具的搜索窗口的屏幕截图。 选择"上载.vhd 或 vhdx 文件作为页 blob （推荐）"复选框。](media/897aa9a9b6acc0aa775c31e7fd82df02.png)

3. 接下来，转到 Azure 门户中浏览器，然后搜索"映像"。 搜索应切换到**创建映像**页上，如以下屏幕截图中所示：

    ![使用映像的示例值填充的 Azure 门户中的创建映像页屏幕截图。](media/d3c840fe3e2430c8b9b1f44b27d2bf4f.png)

4. 创建映像后，应看到一条通知，如以下屏幕截图所示：

    !["已成功创建的映像"通知的屏幕截图。](media/1f41b7192824a2950718a2b7bb9e9d69.png)

## <a name="next-steps"></a>后续步骤

现在，你具有映像后，可以创建或更新主机池。 若要了解有关如何创建和更新主机池的详细信息，请参阅以下文章：

- [使用 Azure 资源管理器模板创建一个主机池](create-host-pools-arm-template.md)
- [教程：使用 Azure Marketplace 创建主机池](create-host-pools-azure-marketplace.md)
- [使用 PowerShell 创建主机池](create-host-pools-powershell.md)
- [设置主机池的用户配置文件共享](create-host-pools-user-profile.md)
- [配置 Windows 虚拟机负载平衡方法](configure-host-pool-load-balancing.md)
