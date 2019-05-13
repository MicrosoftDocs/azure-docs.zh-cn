---
title: Azure 状态监视器 v2 的详细说明 |Microsoft Docs
description: 有关如何开始使用状态监视器 v2 的详细的说明。 监视网站性能，无需重新部署该网站。 使用托管在本地、VM 或 Azure 上的 ASP.NET Web 应用。
services: application-insights
documentationcenter: .net
author: MS-TimothyMothra
manager: alexklim
ms.assetid: 769a5ea4-a8c6-4c18-b46c-657e864e24de
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: tilee
ms.openlocfilehash: 3aca64c7b0f1ad04967782cb3349da302db557a0
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145091"
---
# <a name="status-monitor-v2-detailed-instructions"></a>状态监视器 v2 的详细说明

此文档详细信息如何载入到 PowerShell 库和下载 ApplicationMonitor 模块。 我们记录了所需开始的最常见参数。
我们还提供了手动说明在的 internet 访问不可用。

> [!IMPORTANT]
> 状态监视器 v2 目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。
> 有关详细信息，请参阅[Supplemental Terms of Use 针对 Microsoft Azure 预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="instrumentation-key"></a>检测密钥

若要开始，你必须具有检测密钥。 有关详细信息，请阅读[创建 Application Insights 资源](create-new-resource.md#copy-the-instrumentation-key)。

## <a name="run-powershell-as-administrator-with-an-elevated-execution-policy"></a>以提升的执行策略管理员身份运行 PowerShell

**以管理员身份运行**: 
- 说明:PowerShell 将需要管理员级权限，才能对计算机进行更改。

**执行策略**:
- 说明:默认情况下，运行 PowerShell 脚本将被禁用。 我们建议为当前作用域允许 RemoteSigned 脚本。
- 参考：[有关执行策略](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6)和[的 Set-executionpolicy](
https://docs.microsoft.com/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6
)
- Cmd: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process`
- 可选参数：
    - `-Force` 这将跳过确认提示。

**示例错误：**

```
Install-Module : The 'Install-Module' command was found in the module 'PowerShellGet', but the module could not be
loaded. For more information, run 'Import-Module PowerShellGet'.
    
Import-Module : File C:\Program Files\WindowsPowerShell\Modules\PackageManagement\1.3.1\PackageManagement.psm1 cannot
be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at
https:/go.microsoft.com/fwlink/?LinkID=135170.
```


## <a name="prerequisites-for-powershell"></a>PowerShell 先决条件

审核通过运行命令的当前版本 PowerShell: `$PSVersionTable`。
该命令生成以下输出：


```
Name                           Value
----                           -----
PSVersion                      5.1.17763.316
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.17763.316
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

这些说明进行编写和测试 Windows 10 计算机上与上面列出的版本。

## <a name="prerequisites-for-powershell-gallery"></a>PowerShell 库的先决条件

这些步骤将准备你的服务器来从 PowerShell 库下载模块。

> [!NOTE] 
> 在 Windows、 Windows Server 2016 和 PowerShell 6 上包含的 PowerShell 库的支持。 对于较旧版本，查看此文档：[安装 PowerShellGet](https://docs.microsoft.com/powershell/gallery/installing-psget)


1. 使用提升权限的执行策略，以管理员身份运行 PowerShell。
2. NuGet 程序包提供程序 
    - 说明:此提供程序需要与基于 NuGet 的如 powershell 库的存储库进行交互
    - 参考：[Install-PackageProvider](https://docs.microsoft.com/powershell/module/packagemanagement/install-packageprovider?view=powershell-6)
    - Cmd: `Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201`
    - 可选参数：
        - `-Proxy` 指定请求的代理服务器。
        - `-Force` 这将跳过确认提示。 
    
    如果 NuGet 未设置，你会收到此提示：
        
        NuGet provider is required to continue
        PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet
         provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
        'C:\Users\t\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running
        'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and import
         the NuGet provider now?
        [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    
3. 受信任的存储库
    - 说明:默认情况下，powershell 库是一个不受信任的存储库。
    - 参考：[Set-PSRepository](https://docs.microsoft.com/powershell/module/powershellget/set-psrepository?view=powershell-6)
    - Cmd: `Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted`
    - 可选参数：
        - `-Proxy` 指定请求的代理服务器。

    如果 PowerShell 库不受信任，你会收到此提示：

        Untrusted repository
        You are installing the modules from an untrusted repository. If you trust this repository, change its
        InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
        'PSGallery'?
        [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):

    - 可以确认此更改，并通过运行 cmd 审核所有 PSRepositories: `Get-PSRepository`

4. PowerShellGet 版本 
    - 说明:此模块包含用于从 PowerShell 库获取其他模块的工具。 v1.0.0.1 附带于 Windows 10 和 Windows Server。 所需的最低版本是 v1.6.0。 若要审核运行该命令安装哪个版本： `Get-Command -Module PowerShellGet`
    - 参考：[安装 PowerShellGet](https://docs.microsoft.com/powershell/gallery/installing-psget)
    - Cmd: `Install-Module -Name PowerShellGet`
    - 可选参数：
        - `-Proxy` 指定请求的代理服务器。
        - `-Force` 这将忽略"已安装"警告并安装最新版本。

    如果不使用最新版本的 PowerShellGet，你会收到此错误：
    
        Install-Module : A parameter cannot be found that matches parameter name 'AllowPrerelease'.
        At line:1 char:20
        Install-Module abc -AllowPrerelease
                           ~~~~~~~~~~~~~~~~
            CategoryInfo          : InvalidArgument: (:) [Install-Module], ParameterBindingException
            FullyQualifiedErrorId : NamedParameterNotFound,Install-Module
    
5. 重新启动 PowerShell。 它不能在当前会话中加载的新版本。 任何新的 Powershell 会话将具有最新的 PowerShellGet 加载。

## <a name="download--install-via-powershell-gallery"></a>下载并安装通过 PowerShell 库

这些步骤将从 PowerShell 库下载 Az.ApplicationMonitor 模块。

1. PowerShell 库的先决条件是必需的。
2. 使用提升权限的执行策略，以管理员身份运行 PowerShell。
3. 安装 Az.ApplicationMonitor 模块
    - 参考：[Install-Module](https://docs.microsoft.com/powershell/module/powershellget/install-module?view=powershell-6)
    - Cmd: `Install-Module -Name Az.ApplicationMonitor`
    - 可选参数：
        - `-Proxy` 指定请求的代理服务器。
        - `-AllowPrerelease` 这将允许安装 alpha 和 beta 版本。
        - `-AcceptLicense` 这将跳过"接受许可证"提示
        - `-Force` 这会忽略"不受信任存储库"警告

## <a name="download--install-manually-offline-option"></a>下载并手动安装 （offline 选项）

如果出于任何原因无法连接到 PowerShell 模块，你可以手动下载并安装 Az.ApplicationMonitor 模块。

### <a name="manually-download-the-latest-nupkg"></a>手动下载最新的 nupkg

1. 导航到： https://www.powershellgallery.com/packages/Az.ApplicationMonitor
2. 从版本历史记录中选择最新版本。
3. 找到"安装选项"并选择"手动下载"。

### <a name="option-1-install-into-powershell-modules-directory"></a>选项 1： 安装 PowerShell 模块目录到
安装手动下载的 PowerShell 模块的 PowerShell 目录中，使其成为可发现 PowerShell 会话。
有关详细信息，请参阅：[安装 PowerShell 模块](https://docs.microsoft.com/powershell/developer/module/installing-a-powershell-module)


#### <a name="unzip-nupkg-as-zip-using-expand-archive-v1010"></a>以 zip 格式使用 Expand-archive (v1.0.1.0) 解压缩 nupkg

- 说明:Microsoft.PowerShell.Archive (v1.0.1.0) 的基础版本不能将 nupkg 文件解压缩。 重命名".zip"扩展名的文件。
- 参考：[Expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)
- Cmd: 

    ```
    $pathToNupkg = "C:\az.applicationmonitor.0.2.1-alpha.nupkg"
    $pathToZip = ([io.path]::ChangeExtension($pathToNupkg, "zip"))
    $pathToNupkg | rename-item -newname $pathToZip
    $pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\az.applicationmonitor"
    Expand-Archive -LiteralPath $pathToZip -DestinationPath $pathInstalledModule
    ```

#### <a name="unzip-nupkg-using-expand-archive-v1100"></a>将解压缩 nupkg 使用 Expand-archive (v1.1.0.0)。

- 说明:使用新版展开存档解压缩 nupkgs 而无需重命名该扩展。 
- 参考：[展开存档](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)和[Microsoft.PowerShell.Archive](https://www.powershellgallery.com/packages/Microsoft.PowerShell.Archive/1.1.0.0)
- Cmd:

    ```
    $pathToNupkg = "C:\az.applicationmonitor.0.2.1-alpha.nupkg"
    $pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\az.applicationmonitor"
    Expand-Archive -LiteralPath $pathToNupkg -DestinationPath $pathInstalledModule
    ```

### <a name="option-2-unzip-and-import-manually"></a>选项 2： 将解压缩并手动导入
安装手动下载的 PowerShell 模块的 PowerShell 目录中，使其成为可发现 PowerShell 会话。
有关详细信息，请参阅：[安装 PowerShell 模块](https://docs.microsoft.com/powershell/developer/module/installing-a-powershell-module)

如果安装到任何其他目录，则必须手动导入模块使用[导入模块](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/import-module?view=powershell-6)

> [!IMPORTANT] 
> 通过相对路径，安装将安装的 Dll。 将此包的内容存储到你预期的运行时的目录，并确认访问权限，允许读取但不是能写入。

1. 将扩展名更改为".zip"并将包的内容提取到你的预期的安装目录。
2. 查找"Az.ApplicationMonitor.psd1"的文件路径。
3. 使用提升权限的执行策略，以管理员身份运行 PowerShell。 
4. 加载通过命令模块： `Import-Module Az.ApplicationMonitor.psd1`。
    

## <a name="proxy"></a>代理

在监视您的专用 intranet 上的计算机时，将需要将 http 流量通过代理路由。

支持 PowerShell 命令，以下载并从 PowerShell 库安装 Az.ApplicationMonitor`-Proxy`参数。
当查看上面的说明在编写安装脚本。

Application Insights SDK 将需要将应用程序的遥测数据发送给 Microsoft。 我们建议在 web.config 中配置的应用程序代理设置。请参阅[Application Insights 常见问题解答：代理传递](https://docs.microsoft.com/azure/azure-monitor/app/troubleshoot-faq#proxy-passthrough)有关详细信息。


## <a name="enable-monitoring"></a>启用监视 

Cmd: `Enable-ApplicationInsightsMonitoring`

查看我们[API 参考](status-monitor-v2-api-enable-monitoring.md)有关如何使用此 cmdlet 的详细说明。 



## <a name="next-steps"></a>后续步骤

 查看遥测：

- [浏览指标](../../azure-monitor/app/metrics-explorer.md)，以便监视性能和使用情况
- [搜索事件和日志](../../azure-monitor/app/diagnostic-search.md)来诊断问题
- [分析](../../azure-monitor/app/analytics.md)，以便进行更高级的查询
- [创建仪表板](../../azure-monitor/app/app-insights-dashboards.md)

 添加更多遥测：

- [创建 web 测试](monitor-web-app-availability.md)以确保你的站点保持活动状态。
- [添加 web 客户端遥测](../../azure-monitor/app/javascript.md)，查看网页代码中的异常，并将其插入跟踪调用。
- [将 Application Insights SDK 添加到你的代码](../../azure-monitor/app/asp-net.md)，以便您可以插入跟踪和日志调用

用做更多状态监视器 v2:

- 使用指南，了解如何[疑难解答](status-monitor-v2-troubleshoot.md)状态监视器 v2。
