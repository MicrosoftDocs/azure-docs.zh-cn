---
title: 部署 Azure AD 密码保护的 Azure Active Directory
description: 将 Azure AD 密码保护，以禁止错误密码的本地部署
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: article
ms.date: 02/01/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 256215b1976598b961ada7210e5ee92c9f72d440
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506874"
---
# <a name="deploy-azure-ad-password-protection"></a>部署 Azure AD 密码保护

现在，你已了解[如何强制执行 Windows Server Active directory 的 Azure AD 密码保护](concept-password-ban-bad-on-premises.md)下, 一步是规划和执行您的部署。

## <a name="deployment-strategy"></a>部署策略

我们建议您在审核模式下启动部署。 审核模式是默认的初始设置，可以继续设置密码。 事件日志中记录会被阻止的密码。 在审核模式下部署的代理服务器和 DC 代理后，你应监视的密码策略强制实施策略，将具有对用户和环境的影响。

在审核阶段中，许多组织找出的：

* 需要改进现有的操作过程，以使用更安全的密码。
* 用户通常使用不安全的密码。
* 他们需要通知用户有关即将发生的更改中强制实施安全机制，可能会影响它们，以及如何选择更安全的密码。

该功能在审核模式下运行的一定宽限期后，可以切换中的配置*审核*到*强制实施*需要更加安全的密码。 在此期间最好是执行有针对性的监视。

## <a name="deployment-requirements"></a>部署要求

* 获取 DC 代理服务的 Azure AD 密码保护安装必须运行 Windows Server 2012 或更高版本的所有域控制器。 此要求并不表示，Active Directory 域或林还必须在 Windows Server 2012 域或林功能级别。 如中所述[设计原则](concept-password-ban-bad-on-premises.md#design-principles)，没有最小 DFL 或 FFL 所需的 DC 代理或代理软件运行。
* 安装 DC 代理服务的所有计算机都必须安装了.NET 4.5。
* 获取代理服务的 Azure AD 密码保护安装必须运行 Windows Server 2012 R2 或更高版本的所有计算机。
   > [!NOTE]
   > 代理服务部署是部署 Azure AD 密码保护，即使域控制器可能会建立出站直接 internet 连接的强制要求。 
   >
* 将安装 Azure AD 密码保护代理服务的所有计算机都必须安装的.NET 4.7。
  应已完全更新的 Windows Server 上安装.NET 4.7。 如果这不是这种情况，下载并运行安装程序，请参阅[Windows.NET Framework 4.7 脱机安装程序](https://support.microsoft.com/en-us/help/3186497/the-net-framework-4-7-offline-installer-for-windows)。
* 所有计算机，包括获取 Azure AD 密码保护组件安装的域控制器都必须安装的通用 C 运行时。 通过确保从 Windows 更新的所有更新，可以获取运行时。 也可以特定于操作系统的更新包中获取它。 有关详细信息，请参阅[在 Windows 中的通用 C 运行时的更新](https://support.microsoft.com/help/2999226/update-for-uniersal-c-runtime-in-windows)。
* 网络连接之间必须存在的每个域中至少一个域控制器和至少一台服务器承载密码保护代理服务。 此连接必须允许域控制器访问 RPC 终结点映射器端口 135 和代理服务上的 RPC 服务器端口。 默认情况下，RPC 服务器端口是动态的 RPC 端口，但可以配置[使用静态端口](#static)。
* 承载代理服务的所有计算机必须都具有网络访问以下终结点：

    |**终结点**|**用途**|
    | --- | --- |
    |`https://login.microsoftonline.com`|身份验证请求|
    |`https://enterpriseregistration.windows.net`|Azure AD 密码保护功能|

* 承载代理服务进行密码保护的所有计算机必须都配置为允许出站 TLS 1.2 HTTP 流量。
* 用于注册 Azure AD 密码保护和林的代理服务的全局管理员帐户。
* 要与 Azure AD 中注册的 Windows Server Active Directory 林的林根域中具有 Active Directory 域管理员权限的帐户。
* 运行 DC 代理服务软件任何 Active Directory 域必须使用分布式文件系统复制 (DFSR) 进行 sysvol 复制。
* 必须在域中运行 Windows Server 2012 的所有域控制器上启用密钥分发服务。 默认情况下，此服务启用通过手动触发器启动。

## <a name="single-forest-deployment"></a>单个林部署

下图显示了 Azure AD 密码保护的基本组件如何协同工作的本地 Active Directory 环境中。

![Azure AD 密码保护组件如何配合工作](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

它是一个好办法查看之前将其部署该软件的工作原理。 请参阅[Azure AD 密码保护的概念性概述](concept-password-ban-bad-on-premises.md)。

### <a name="download-the-software"></a>下载软件

有两个 Azure AD 密码保护所需的安装程序。 它们是可从[Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=57071)。

### <a name="install-and-configure-the-proxy-service-for-password-protection"></a>安装和配置密码保护代理服务

1. 选择要承载密码保护代理服务的一个或多个服务器。
   * 每个此类服务只能为单个林提供的密码策略。 主机必须加入到该林的域中。 同时支持域根和子域。 您需要在林中的每个域中至少一个 DC 和密码保护计算机之间的网络连接。
   * 您可以用于测试的域控制器上运行的代理服务。 但此域控制器然后需要 internet 连接，可以是安全隐患。 我们建议仅用于测试此配置。
   * 我们建议在至少两台代理服务器以实现冗余。 请参阅[高可用性](howto-password-ban-bad-on-premises-deploy.md#high-availability)。

1. 安装 Azure AD 密码保护代理服务使用`AzureADPasswordProtectionProxySetup.exe`软件安装程序。
   * 安装该软件不需要重新启动。 可以使用标准的 MSI 过程将软件安装自动化，例如：

      `AzureADPasswordProtectionProxySetup.exe /quiet`

      > [!NOTE]
      > 在安装 AzureADPasswordProtectionProxySetup.msi 包，以避免出现安装错误之前，必须运行 Windows 防火墙服务。 如果 Windows 防火墙配置为不运行，解决方法是暂时启用并在安装期间运行防火墙服务。 代理软件安装后具有 Windows 防火墙上的任何特定依赖项。 如果正在使用第三方防火墙，它必须仍配置以满足部署要求。 其中包括允许到端口 135 和代理服务器的 RPC 端口的入站的访问。 请参阅[部署要求](howto-password-ban-bad-on-premises-deploy.md#deployment-requirements)。

1. 以管理员身份打开 PowerShell 窗口。
   * 该密码保护代理软件包含新的 PowerShell 模块*AzureADPasswordProtection*。 运行此 PowerShell 模块中的各种 cmdlet 的以下步骤。 按如下所示导入新的模块：

      ```powershell
      Import-Module AzureADPasswordProtection
      ```

   * 若要检查服务正在运行，请使用以下 PowerShell 命令：

      `Get-Service AzureADPasswordProtectionProxy | fl`。

     结果应显示**状态**为"Running"。

1. 注册代理。
   * 步骤 3 完成后，在计算机上运行的代理服务。 但该服务还没有与 Azure AD 通信所需的凭据。 不需要与 Azure AD 注册：

     `Register-AzureADPasswordProtectionProxy`

     此 cmdlet 需要你的 Azure 租户的全局管理员凭据。 您还需要在目录林根域中的本地 Active Directory 域管理员权限。 此命令成功一次代理服务后，它的其他调用将成功，但不是必需的。

      `Register-AzureADPasswordProtectionProxy` Cmdlet 支持以下三种身份验证模式。

     * 交互式身份验证模式：

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > 在 Server Core 操作系统，此模式下不起作用。 相反，使用以下身份验证模式之一。 此外，如果启用了 Internet Explorer 增强安全配置在此模式可能会失败。 解决方法是先禁用该配置，注册代理，然后重新启用它。

     * 设备代码身份验证模式：

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code XYZABC123 to authenticate.
        ```

        然后，按照以下不同的设备上显示的说明完成身份验证。

     * 无提示（基于密码）身份验证模式：

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionProxy -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > 如果 Azure 多重身份验证是必需的此模式下无法正常工作。 在这种情况下，使用以前的两种身份验证模式之一。

       当前不需要指定 *-ForestCredential*参数，保留供将来提供的功能。

   针对密码保护代理服务的注册是必需的一次只能在服务的生命周期中。 之后，代理服务将自动执行任何其他必要的维护。

   > [!TIP]
   > 可能在完成第一次为特定的 Azure 租户中运行此 cmdlet 之前有明显的延迟。 除非将报告失败，不必担心这种延迟。

1. 注册林。
   * 必须初始化具有所需的凭据以使用与 Azure 通信的本地 Active Directory 林`Register-AzureADPasswordProtectionForest`PowerShell cmdlet。 该 cmdlet 还要求你的 Azure 租户的全局管理员凭据。 它还需要在目录林根域中的本地 Active Directory 域管理员权限。 针对每个林运行此步骤一次。

      `Register-AzureADPasswordProtectionForest` Cmdlet 支持以下三种身份验证模式。

     * 交互式身份验证模式：

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > 此模式下无法在 Server Core 操作系统上运行。 而是使用以下两种身份验证模式之一。 此外，如果启用了 Internet Explorer 增强安全配置在此模式可能会失败。 解决方法是先禁用该配置，注册代理，然后重新启用它。  

     * 设备代码身份验证模式：

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code XYZABC123 to authenticate.
        ```

        然后，按照以下不同的设备上显示的说明完成身份验证。

     * 无提示（基于密码）身份验证模式：

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionForest -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > 如果 Azure 多重身份验证是必需的此模式下无法正常工作。 在这种情况下，使用以前的两种身份验证模式之一。

       当前登录的用户也是根级域的 Active Directory 域管理员，这些示例才成功。 如果这不是这种情况，则可以提供通过备用域凭据 *-ForestCredential*参数。

   > [!NOTE]
   > 如果在环境中安装了多个代理服务器，它并不重要用于注册在林中的代理服务器。
   >
   > [!TIP]
   > 可能在完成第一次为特定的 Azure 租户中运行此 cmdlet 之前有明显的延迟。 除非将报告失败，不必担心这种延迟。

   Active Directory 林的注册是必需的一次只能在林的生命周期。 之后，在林中域控制器代理将自动执行任何其他必要的维护。 之后`Register-AzureADPasswordProtectionForest`运行已成功为林时，该 cmdlet 的其他调用成功，但不是必需的。

   有关`Register-AzureADPasswordProtectionForest`若要成功，在至少一个域控制器运行 Windows Server 2012 或更高版本必须是可用代理服务器的域中。 但不必在此步骤之前的任何域控制器上安装 DC 代理软件。

1. 配置密码保护，通过 HTTP 代理进行通信的代理服务。

   如果你的环境需要使用特定的 HTTP 代理来与 Azure 通信，使用此方法：创建*AzureADPasswordProtectionProxy.exe.config* %ProgramFiles%\Azure AD 密码保护 Proxy\Service 文件夹中的文件。 包括以下内容：

      ```xml
      <configuration>
        <system.net>
          <defaultProxy enabled="true">
           <proxy bypassonlocal="true"
               proxyaddress="http://yourhttpproxy.com:8080" />
          </defaultProxy>
        </system.net>
      </configuration>
      ```

   如果您的 HTTP 代理服务器要求身份验证，将添加*useDefaultCredentials*标记：

      ```xml
      <configuration>
        <system.net>
          <defaultProxy enabled="true" useDefaultCredentials="true">
           <proxy bypassonlocal="true"
               proxyaddress="http://yourhttpproxy.com:8080" />
          </defaultProxy>
        </system.net>
      </configuration>
      ```

   在这两种情况下，将为`http://yourhttpproxy.com:8080`的地址和特定的 HTTP 代理服务器的端口。

   如果您的 HTTP 代理服务器配置给我们一个授权策略，您必须授予访问权限的托管代理服务进行密码保护的计算机的 Active Directory 计算机帐户。

   我们建议你停止并重新启动代理服务，在创建或更新后*AzureADPasswordProtectionProxy.exe.config*文件。

   代理服务不支持使用特定凭据用于连接到 HTTP 代理。

1. 可选：配置密码保护，以侦听特定端口的代理服务。
   * 在域控制器上的密码保护的 DC 代理软件通过 TCP 使用 RPC 与代理服务进行通信。 默认情况下，代理服务在任何可用的动态 RPC 终结点上进行侦听。 但您可以将服务配置为侦听特定 TCP 端口，如果这是必要由于网络拓扑或您的环境中的防火墙要求。
      * <a id="static" /></a>若要配置静态端口下运行的服务，请使用`Set-AzureADPasswordProtectionProxyConfiguration`cmdlet。

         ```powershell
         Set-AzureADPasswordProtectionProxyConfiguration –StaticPort <portnumber>
         ```

         > [!WARNING]
         > 必须停止并重启服务，才能让这些更改生效。

      * 若要配置动态端口下运行的服务，使用相同的过程，但设置*StaticPort*为零：

         ```powershell
         Set-AzureADPasswordProtectionProxyConfiguration –StaticPort 0
         ```

         > [!WARNING]
         > 必须停止并重启服务，才能让这些更改生效。

   > [!NOTE]
   > 端口配置中进行任何更改后，密码保护代理服务将需要手动重启。 但无需进行这些配置更改后重新启动域控制器上的 DC 代理服务软件。

   * 若要查询的服务的当前配置，请使用`Get-AzureADPasswordProtectionProxyConfiguration`cmdlet:

      ```powershell
      Get-AzureADPasswordProtectionProxyConfiguration | fl

      ServiceName : AzureADPasswordProtectionProxy
      DisplayName : Azure AD password protection Proxy
      StaticPort  : 0
      ```

### <a name="install-the-dc-agent-service"></a>安装 DC 代理服务

   使用安装的 DC 代理服务的密码保护`AzureADPasswordProtectionDCAgentSetup.msi`包。

   软件安装或卸载，则需要重新启动。 这是因为密码筛选器 Dll 是仅加载或卸载被重启。

   可以在尚不是域控制器的计算机上安装 DC 代理服务。 在这种情况下，服务将启动并运行，但保持非活动状态，直到计算机被提升为域控制器。

   您可以通过使用标准的 MSI 过程自动化软件安装。 例如:

   `msiexec.exe /i AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn`

   > [!WARNING]
   > 示例 msiexec 命令会导致立即重新启动。 若要避免这种情况，请使用`/norestart`标志。

域控制器上安装了 DC 代理软件并重新启动该计算机后，安装已完成。 无需（也无法）进行其他配置。

## <a name="multiple-forest-deployments"></a>多林部署

跨多个林部署 Azure AD 密码保护不需要满足其他要求。 "单林部署"部分中所述，单独配置每个林。 每个密码保护代理只支持联接到林中的域控制器。 密码保护软件中的任何林中不知道的密码保护软件部署到其他林，而不考虑 Active Directory 信任配置中。

## <a name="read-only-domain-controllers"></a>只读域控制器

密码更改/设置不处理和保留在只读域控制器 (Rodc) 上。 将在转发到可写域控制器。 因此，您无需在 Rodc 上安装 DC 代理软件。

## <a name="high-availability"></a>高可用性

在林中的域控制器尝试从 Azure 下载新策略或其他数据时，密码保护的主可用性问题为代理服务器的可用性。 在决定要调用的代理服务器时，每个 DC 代理将使用简单的轮循机制样式算法。 代理跳过代理服务器不响应。 对于具有正常运行复制的 directory 和 sysvol 文件夹状态的最完全连接 Active Directory 部署，两台代理服务器就足以确保可用性。 这会导致及时下载新策略和其他数据。 但是，可以将部署其他代理服务器。

DC 代理软件的设计可降低高可用性与相关联的常规问题。 DC 代理维护最近下载的密码策略的本地缓存。 即使所有已注册的代理服务器变得不可用，DC 代理会继续强制实施其缓存的密码策略。 通常是在大型部署中的密码策略的合理的更新频率*天*、 不是几小时或更少。 因此，代理服务器的短暂停机不会显著影响 Azure AD 密码保护。

## <a name="next-steps"></a>后续步骤

至此，已安装的服务所需的 Azure AD 密码保护的本地服务器上[执行安装后配置和报告信息的收集](howto-password-ban-bad-on-premises-operations.md)完成部署。

[Azure AD 密码保护的概念性概述](concept-password-ban-bad-on-premises.md)
