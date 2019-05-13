---
title: 管理 Azure Log Analytics 代理 | Microsoft Docs
description: 本文介绍您通常将执行日志分析 Windows 或 Linux 代理在计算机上部署的生命周期内的不同管理任务。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: magoedte
ms.openlocfilehash: 1809cc50f3ad3c285e0b69bc6e383a2c7c398238
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139253"
---
# <a name="managing-and-maintaining-the-log-analytics-agent-for-windows-and-linux"></a>管理并维护 Windows 和 Linux 的 Log Analytics 代理

初始部署之后的日志分析 Windows 或 Azure Monitor 中的 Linux 代理，你可能需要重新配置代理、 升级它，或如果它已达到其生命周期中的停用阶段从计算机删除。 可以轻松地手动或自动管理这些日常维护任务，从而减少运行错误并降低费用。

## <a name="upgrading-agent"></a>升级代理

适用于 Windows 和 Linux 的 Log Analytics 代理可以升级到最新版本手动或自动根据部署方案和环境中运行 VM。 可以使用以下方法来升级代理。

| 环境 | 安装方法 | 升级方法 |
|--------|----------|-------------|
| Azure VM | Windows/Linux 的 log Analytics 代理 VM 扩展 | 代理自动升级默认情况下，除非您配置您的 Azure 资源管理器模板通过将属性设置中选择退出*autoUpgradeMinorVersion*到**false**。 |
| 自定义 Azure VM 映像 | 手动安装的 Windows/Linux 的 Log Analytics 代理 | 更新到最新版本的代理的 Vm 需要从命令行运行的 Windows installer 程序包或 Linux 自解压和可安装 shell 脚本捆绑包执行。|
| 非 Azure Vm | 手动安装的 Windows/Linux 的 Log Analytics 代理 | 更新到最新版本的代理的 Vm 需要从命令行运行的 Windows installer 程序包或 Linux 自解压和可安装 shell 脚本捆绑包执行。 |

### <a name="upgrade-windows-agent"></a>升级 Windows 代理 

若要更新为不安装使用 Log Analytics VM 扩展的最新版本的 Windows VM 上的代理，你可以运行从命令提示符处、 脚本或其他自动化解决方案，或通过使用 MMASetup-\<平台\>.msi 安装程序向导。  

通过执行以下步骤，可以从 Log Analytics 工作区中，下载 Windows 代理的最新版本。

1. 登录到 [Azure 门户](https://portal.azure.com)。

2. 在 Azure 门户中，单击“所有服务”。 在资源列表中，键入“Log Analytics”。 开始键入时，会根据输入筛选该列表。 选择“Log Analytics 工作区”。

3. 在 Log Analytics 工作区列表中，选择工作区。

4. 在 Log Analytics 工作区中，选择**高级设置**，然后选择**连接的源**，最后**Windows 服务器**。

5. 从**Windows 服务器**页上，选择相应**下载 Windows 代理**要下载根据 Windows 操作系统的处理器体系结构版本。

>[!NOTE]
>在升级期间的适用于 Windows 的 Log Analytics 代理，它不支持配置或重新配置一个工作区来向报告。 若要配置的代理，需要按照下面列出的受支持方法之一[添加或删除工作区](#adding-or-removing-a-workspace)。
>

#### <a name="to-upgrade-using-the-setup-wizard"></a>若要使用安装向导升级

1. 使用具有管理权限的帐户登录到计算机。

2. 执行**MMASetup-\<平台\>.exe**以启动安装向导。

3. 在安装向导的第一页上，单击**下一步**。

4. 在中**Microsoft Monitoring Agent 安装程序**对话框中，单击**我同意**接受许可协议。

5. 在中**Microsoft Monitoring Agent 安装程序**对话框中，单击**升级**。 状态页面显示升级进度。

6. 当**Microsoft Monitoring Agent 配置已成功完成。** 页出现后，单击**完成**。

#### <a name="to-upgrade-from-the-command-line"></a>若要从命令行升级

1. 使用具有管理权限的帐户登录到计算机。

2. 若要提取代理安装文件，请在提升的命令提示符处运行 `MMASetup-<platform>.exe /c`，这会提示要将文件提取到的路径。 或者，可以通过传递参数 `MMASetup-<platform>.exe /c /t:<Full Path>` 来指定路径。

3. 运行以下命令，其中 D:\是升级日志文件的位置。

    ```dos
    setup.exe /qn /l*v D:\logs\AgentUpgrade.log AcceptEndUserLicenseAgreement=1
    ```

### <a name="upgrade-linux-agent"></a>升级 Linux 代理 

从以前的版本升级 (> 1.0.0-47) 支持。 使用 `--upgrade` 命令执行安装时，会将代理的所有组件都升级到最新版本。

运行以下命令以升级代理。

`sudo sh ./omsagent-*.universal.x64.sh --upgrade`

## <a name="adding-or-removing-a-workspace"></a>添加或删除工作区

### <a name="windows-agent"></a>Windows 代理
在本部分中的步骤是必需的当你想要不只重新配置 Windows 代理的不同工作区报告或从其配置中，删除工作区但还想配置代理向多个工作区 （通常报告称为多宿主）。 代理和使用方法如下所述的初始安装后可以仅执行配置 Windows 代理向多个工作区报告。    

#### <a name="update-settings-from-control-panel"></a>从控制面板更新设置

1. 使用具有管理权限的帐户登录到计算机。

2. 打开“控制面板”

3. 选择“Microsoft Monitoring Agent”，然后单击“Azure Log Analytics”选项卡。

4. 若要删除工作区，请选中该工作区，然后单击“删除”。 如果还需代理停止向任何其他工作区报告，请重复此步骤。

5. 若要添加工作区，请单击“添加”，然后在“添加 Log Analytics 工作区”对话框中粘贴工作区 ID 和工作区密钥（主密钥）。 如果计算机应向 Azure 政府云中的 Log Analytics 工作区报告，请从“Azure 云”下拉列表中选择“Azure 美国政府”。

6. 单击“确定”  保存更改。

#### <a name="remove-a-workspace-using-powershell"></a>使用 PowerShell 删除工作区

```powershell
$workspaceId = "<Your workspace Id>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.RemoveCloudWorkspace($workspaceId)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-commercial-using-powershell"></a>使用 PowerShell 在 Azure 商业版中添加工作区

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-for-us-government-using-powershell"></a>使用 PowerShell 在 Azure 美国政府版中添加工作区

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey, 1)
$mma.ReloadConfiguration()
```

>[!NOTE]
>如果以前已使用命令行或脚本安装或配置了代理，`EnableAzureOperationalInsights` 会被 `AddCloudWorkspace` 和 `RemoveCloudWorkspace` 取代。
>

### <a name="linux-agent"></a>Linux 代理
以下步骤演示如何重新配置 Linux 代理，如果您决定将其注册到不同的工作区或从其配置中删除工作区。

1. 若要验证该代理是否已注册到工作区，请运行以下命令：

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    此命令应返回类似以下示例的状态：

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

    状态中还应显示代理正在运行，这一点非常重要，否则以下重新配置代理的步骤不会成功完成。

2. 如果代理已注册到工作区，请运行以下命令删除已注册的工作区。 如果未注册，请继续执行下一步。

    `/opt/microsoft/omsagent/bin/omsadmin.sh -X`

3. 若要注册到不同的工作区，请运行以下命令：

    `/opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <shared key> [-d <top level domain>]`
    
4. 若要验证所做的更改生效，请运行以下命令：

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    此命令应返回类似以下示例的状态：

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

无需重启代理服务即可使更改生效。

## <a name="update-proxy-settings"></a>更新代理设置
若要将代理配置为在部署后通过代理服务器或 [Log Analytics 网关](gateway.md)与服务通信，请使用以下任一方法来完成此任务。

### <a name="windows-agent"></a>Windows 代理

#### <a name="update-settings-using-control-panel"></a>使用控制面板更新设置

1. 使用具有管理权限的帐户登录到计算机。

2. 打开“控制面板”

3. 选择“Microsoft Monitoring Agent”，然后单击“代理设置”选项卡。

4. 单击“使用代理服务器”，然后提供代理服务器或网关的 URL 和端口号。 如果代理服务器或 Log Analytics 网关要求身份验证，请键入用户名和密码进行身份验证，然后单击“确定”。

#### <a name="update-settings-using-powershell"></a>使用 PowerShell 更新设置

复制以下示例 PowerShell 代码，使用特定于自己环境的信息对其进行更新，并使用 PS1 文件扩展名将其保存。 直接连接到 Azure Monitor 中的 Log Analytics 工作区的每台计算机上运行该脚本。

```powershell
param($ProxyDomainName="https://proxy.contoso.com:30443", $cred=(Get-Credential))

# First we get the Health Service configuration object. We need to determine if we
#have the right update rollup with the API we need. If not, no need to run the rest of the script.
$healthServiceSettings = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'

$proxyMethod = $healthServiceSettings | Get-Member -Name 'SetProxyInfo'

if (!$proxyMethod)
{
    Write-Output 'Health Service proxy API not present, will not update settings.'
    return
}

Write-Output "Clearing proxy settings."
$healthServiceSettings.SetProxyInfo('', '', '')

$ProxyUserName = $cred.username

Write-Output "Setting proxy to $ProxyDomainName with proxy username $ProxyUserName."
$healthServiceSettings.SetProxyInfo($ProxyDomainName, $ProxyUserName, $cred.GetNetworkCredential().password)
```

### <a name="linux-agent"></a>Linux 代理
如果 Linux 计算机需要通过代理服务器或 Log Analytics 网关进行通信，请执行以下步骤。 代理配置值具有以下语法 `[protocol://][user:password@]proxyhost[:port]`。 *proxyhost* 属性接受代理服务器的完全限定域名或 IP 地址。

1. 运行以下命令编辑文件 `/etc/opt/microsoft/omsagent/proxy.conf`，并针对特定设置更改相关的值。

    ```
    proxyconf="https://proxyuser:proxypassword@proxyserver01:30443"
    sudo echo $proxyconf >>/etc/opt/microsoft/omsagent/proxy.conf
    sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf
    ```

2. 运行以下命令重启代理：

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ```

## <a name="uninstall-agent"></a>卸载代理
使用以下过程之一卸载 Windows 或 Linux 代理使用的命令行或安装向导。

### <a name="windows-agent"></a>Windows 代理

#### <a name="uninstall-from-control-panel"></a>从控制面板卸载
1. 使用具有管理权限的帐户登录到计算机。

2. 在“控制面板”中，单击“程序和功能”。

3. 在“程序和功能”中依次单击“Microsoft Monitoring Agent”、“卸载”、“是”。

>[!NOTE]
>还可以通过双击“MMASetup-\<platform\>.exe”运行代理安装向导，该程序可从 Azure 门户中的工作区下载。

#### <a name="uninstall-from-the-command-line"></a>从命令行卸载
下载的代理文件是使用 IExpress 创建的独立安装包。 代理和支持文件的安装程序包含在该包中，需要提取才能使用以下示例中所示的命令行正确卸载。

1. 使用具有管理权限的帐户登录到计算机。

2. 若要提取代理安装文件，请在提升的命令提示符处运行 `extract MMASetup-<platform>.exe`，这会提示要将文件提取到的路径。 或者，可以通过传递参数 `extract MMASetup-<platform>.exe /c:<Path> /t:<Path>` 来指定路径。 若要深入了解 IExpress 支持的命令行开关，请参阅 [IExpress 的命令行开关](https://support.microsoft.com/help/197147/command-line-switches-for-iexpress-software-update-packages)，然后根据需要更新示例。

3. 在提示符下键入 `%WinDir%\System32\msiexec.exe /x <Path>:\MOMAgent.msi /qb`。

### <a name="linux-agent"></a>Linux 代理
若要删除代理，请在 Linux 计算机上运行以下命令。 *--purge* 参数可彻底删除代理及其配置。

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

## <a name="configure-agent-to-report-to-an-operations-manager-management-group"></a>将代理配置为向 Operations Manager 管理组报告

### <a name="windows-agent"></a>Windows 代理
执行以下步骤可将适用于 Windows 的 Log Analytics 代理配置为向 System Center Operations Manager 管理组报告。

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. 使用具有管理权限的帐户登录到计算机。

2. 打开“控制面板”

3. 单击“Microsoft Monitoring Agent”，然后单击“Operations Manager”选项卡。

4. 如果 Operations Manager 服务器与 Active Directory 集成，请单击“**自动更新 AD DS 中的管理组分配**”。

5. 单击“**添加**”以打开“**添加管理组**”对话框。

6. 在“管理组名称”字段，键入管理组的名称。

7. 在“主管理服务器”字段，键入主管理服务器的计算机名。

8. 在“管理服务器端口”字段，键入 TCP 端口号。

9. 在“**代理操作帐户**”下面，选择本地系统帐户或本地域帐户。

10. 单击“**确定**”关闭“**添加管理组**”对话框中，再单击“**确定**”关闭“**Microsoft Monitoring Agent 属性**”对话框。

### <a name="linux-agent"></a>Linux 代理
执行以下步骤可将适用于 Linux 的 Log Analytics 代理配置为向 System Center Operations Manager 管理组报告。

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. 编辑 `/etc/opt/omi/conf/omiserver.conf` 文件

2. 确保以 `httpsport=` 开头的行定义了端口 1270。 例如：`httpsport=1270`

3. 重新启动 OMI 服务器：`sudo /opt/omi/bin/service_control restart`

## <a name="next-steps"></a>后续步骤

如果在安装或管理代理时遇到问题，请查看 [Linux 代理疑难解答](agent-linux-troubleshoot.md)。
