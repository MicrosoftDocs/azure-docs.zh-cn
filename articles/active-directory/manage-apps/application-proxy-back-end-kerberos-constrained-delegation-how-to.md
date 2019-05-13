---
title: 排查应用程序代理的 Kerberos 约束委派配置问题 | Microsoft Docs
description: 排查应用程序代理的 Kerberos 约束委派配置问题
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: celested
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8413c75a683ff135b3442da6c6a94787ed0866ca
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65441448"
---
# <a name="troubleshoot-kerberos-constrained-delegation-configurations-for-application-proxy"></a>排查应用程序代理的 Kerberos 约束委派配置问题

若要通过 SSO 登录到已发布的应用程序，可以根据不同的应用程序使用不同的方法。 Azure Active Directory (Azure AD) 应用程序代理默认提供的一个选项是 Kerberos 约束委派 (KCD)。 可以为用户将连接器配置为向后端应用程序运行受约束的 Kerberos 身份验证。

启用 KCD 的过程非常简单。 只需对支持 SSO 的各种组件和身份验证流具有一个整体的理解即可。 但有时，KCD SSO 无法按预期运行。 所以，需要良好的信息源来排查这些方案的问题。

本文提供一站式参考，帮助用户排查和自我修复一些最常见的问题。 此外，本文还介绍如何诊断更复杂的实现问题。

本文作出以下假设：

-   Azure AD 应用程序代理的部署遵循[应用程序代理入门](application-proxy-add-on-premises-application.md)，对非 KCD 应用程序的常规访问按预期方式进行。

-   发布的目标应用程序以 Kerberos 的 Internet Information Services (IIS) 和 Microsoft 实现为基础。

-   服务器和应用程序主机驻留在单个 Azure Active Directory 域中。 有关跨域和林方案的详细信息，请参阅 [KCD 白皮书](https://aka.ms/KCDPaper)。

-   主体应用程序在启用了预身份验证的 Azure 租户中发布。 用户需要通过基于窗体的身份验证进行 Azure 身份验证。 本文不介绍丰富的客户端身份验证方案。 可能会在未来某个时候添加这些方案。

## <a name="prerequisites"></a>必备组件

Azure AD 应用程序代理可以部署到许多类型的基础结构或环境中。 不同组织的体系结构有所不同。 与 KCD 相关的问题的最常见原因并非环境。 造成大多数问题的原因可能是简单的配置错误或常规错误。

因此，在开始解决问题之前，最好确保满足[借助应用程序代理使用 KCD SSO](application-proxy-configure-single-sign-on-with-kcd.md) 的所有先决条件。 请注意 2012R2 上的配置 Kerberos 约束委派部分。 此过程采用不同方法在以前版本的 Windows 上配置 KCD。 此外，请注意以下注意事项：

-   域成员服务器有时会使用特定的域控制器 (DC) 来打开安全通道对话框。 然后服务器可能在任意给定时间移动到另一个对话框。 因此，连接器主机不限制于仅与特定的本地站点 DC 进行通信。

-   跨域方案依赖于将连接器主机定向至 DC 的引荐，而这些 DC 可能不在本地网络外围内。 在这些情况下，将流量发送到表示其他各个域的 DC 同样重要。 如果未发送，则委派失败。

-   如果可能，避免在连接器主机和 DC 之间放置任何活动 IPS 或 IDS 设备。 这些设备有时会过度干扰，并干扰核心 RPC 流量。

用简单方案测试委派。 引入的变量越多，可能需要应对的变量也越多。 为节省时间，将测试限制为在单个连接器中进行。 解决问题后，添加其他连接器。

某些环境因素也可能导致问题。 若要避免这些因素，在测试期间尽可能最小化体系结构。 例如，内部防火墙 ACL 配置错误十分常见。 如果可能，将所有流量从连接器直接发送到 DC 和后端应用程序。

放置连接器的绝对最佳位置应尽可能靠近目标。 测试时内联的防火墙会增加不必要的复杂性，并且会延长调查时间。

什么显示 KCD 问题？ 有多个常见迹象指示 KCD SSO 失败。 浏览器中显示问题的第一个迹象。

   ![KCD 配置不正确的错误](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic1.png)

   ![授权因缺少权限而失败](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic2.png)

这两张图像显示了相同的故障：SSO 失败。 拒绝用户访问应用程序。

## <a name="troubleshooting"></a>故障排除

如何排除故障取决于问题和观察到的症状。 在进行更多操作之前，请浏览以下文章。 它们将提供有用的故障排除信息：

-   [应用程序代理问题和错误消息故障排除](application-proxy-troubleshoot.md)

-   [Kerberos 错误和症状](application-proxy-troubleshoot.md#kerberos-errors)

-   [在本地标识和云标识不相同的情况下使用 SSO](application-proxy-configure-single-sign-on-with-kcd.md#working-with-different-on-premises-and-cloud-identities)

如果查看这些文章，则存在主要问题。 首先，将流分为可进行故障排除的以下三个不同阶段。

### <a name="client-pre-authentication"></a>客户端预身份验证 
外部用户通过浏览器进行 Azure 身份验证。 进行 Azure 预身份验证的功能是让 KCD SSO 运行的必要条件。 测试此功能，如果存在任何问题，请解决问题。 预身份验证阶段与 KCD 或发布的应用程序无关。 可以轻松更正任何差异，只需通过对主题帐户进行健全性检查，确认该帐户在 Azure 中存在。 并且检查其未被禁用或阻止。 浏览器中的错误响应提供了足够的描述，可解释原因。 如果不确定，查看其它 Microsoft 故障排除文章进行验证。

### <a name="delegation-service"></a>委派服务 
从 Kerberos 密钥发行中心 (KCD) 为用户获取 Kerberos 服务票证的 Azure 代理连接器。

客户端和 Azure 前端之间的外部通信与 KCD 无任何关系。 这些通信仅确保 KCD 运行。 Azure 代理服务提供了用于获取 Kerberos 票证的有效用户 ID。 如果没有此 ID，KCD 无法运行且发生故障。

如前所述，浏览器错误消息会提供有关操作失败原因的有用线索。 请确保记下响应中的活动 ID 和时间戳。 此信息可帮助用户将行为与 Azure 代理事件日志中的实际事件相关联。

   ![KCD 配置不正确的错误](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic3.png)

在事件日志中看到的相应条目显示为事件 13019 或 12027。 在“应用程序和服务日志”&gt;“Microsoft”&gt;“AadApplicationProxy”&gt;“连接器”&gt;“管理员”中找到连接器事件日志。

   ![应用程序代理事件日志中的事件 13019](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic4.png)

   ![应用程序代理事件日志中的事件 12027](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic5.png)

1.   将内部 DNS 中的 A 记录用于应用程序地址，而非 CName。

2.   再次确认已向连接器主机授予对指定目标帐户 SPN 进行委托的权利。 再次确认已选择“使用任意身份验证协议”。 有关详细信息，请参阅 [SSO 配置文章](application-proxy-configure-single-sign-on-with-kcd.md)。

3.   验证 Azure AD 中是否只存在一个 SPN 实例。 在任何域成员主机上的命令提示符处发出 `setspn -x`。

4.   检查是否强制执行了限制[颁发的 Kerberos 令牌的最大大小](https://blogs.technet.microsoft.com/askds/2012/09/12/maxtokensize-and-windows-8-and-windows-server-2012/)的域策略。 如果发现令牌过多，此策略将阻止连接器获取令牌。

为了获取关于这些问题的详细信息，下一步是使用网络跟踪捕获连接器主机和域 KDC 之间的交换内容。 有关详细信息，请参阅[深入进行故障排除白皮书](https://aka.ms/proxytshootpaper)。

如果票证看起来正常，则可在日志中看到一个事件，声明身份验证因应用程序返回 401 而失败。 此事件表示目标应用程序已拒绝票证。 转到下一阶段。

### <a name="target-application"></a>目标应用程序 
连接器提供的 Kerberos 票证的使用者。 在此阶段，预期连接器已将 Kerberos 服务票证发送到后端。 此票证是第一个应用程序请求中的标头。

1.   通过使用门户中定义的应用程序内部 URL，验证该应用程序可从连接器主机上的浏览器直接访问。 然后就可以成功登录。 有关详细信息可在连接器“故障排除”页上找到。

2.   还是在连接器主机上，确认浏览器和应用程序之间的身份验证在使用 Kerberos。 请采取以下任一措施：

3.  在 Internet Explorer 中运行开发人员工具（“F12”），或从连接器主机上使用 [Fiddler](https://blogs.msdn.microsoft.com/crminthefield/2012/10/10/using-fiddler-to-check-for-kerberos-auth/)。 使用内部 URL 转到应用程序。 检查在应用程序的响应中是否返回了提供的 WWW 授权标头，以确保存在 negotiate 或 Kerberos。 

    - 在从浏览器到应用程序的响应中返回的下一个 Kerberos Blob 以“YII”开头。 这些字母表示 Kerberos 正在运行。 另一方面，Microsoft NT LAN 管理器 (NTLM) 总是以“TlRMTVNTUAAB”开头，通过 Base64 解码后读作 NTLM 安全支持提供程序 (NTLMSSP)。 如果在 Blob 的开头看到“TlRMTVNTUAAB”，则该 Kerberos 不可用。 如果未看到“TlRMTVNTUAAB”，则 Kerberos 有可能可用。
   
       > [!NOTE]
       > 如果使用 Fiddler，则此方法需要暂时禁用针对 IIS 中的应用程序配置的扩展保护。
      
      ![浏览器网络检查窗口](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic6.png)
   
    - 此图中的 Blob 不以“TIRMTVNTUAAB”开头。 因此，在此示例中，Kerberos 可用，并且 Kerberos Blob 不以“YII”开头。

4.  将 NTLM 暂时从 IIS 站点上的提供程序列表中删除。 直接从连接器主机上的 Internet Explorer 访问应用。 NTLM 不再存在于提供程序列表中。 只可通过使用 Kerberos 来访问应用程序。 如果访问失败，则应用程序的配置可能出现了问题。 Kerberos 身份验证运行不正常。

    - 如果 Kerberos 不可用，请检查 IIS 中应用程序的身份验证设置。 确保“Negotiate”列在顶部，NTLM 在其下方。 如果看到“Not Negotiate”、“Kerberos 或 Negotiate”或“PKU2U”，只要 Kerberos 正常运行，则继续操作。

       ![Windows 身份验证提供程序](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic7.png)
   
    - 既然有了 Kerberos 和 NTLM，暂时在门户中禁用应用程序的预身份验证。 尝试使用外部 URL 从 Internet 访问它。 系统将提示进行身份验证。 可以通过上一步中使用的同一帐户完成此操作。 如果不能，则后端应用程序而非 KCD 出现了问题。

    - 在门户中重新启用预身份验证。 尝试通过外部 URL 连接到应用程序，以通过 Azure 进行身份验证。 如果 SSO 失败，则将在浏览器中看到禁止的错误消息，并在日志中看到事件 13022：

       *Microsoft AAD 应用程序代理连接器无法对用户进行身份验证，因为后端服务器回应给 Kerberos 身份验证尝试的是 HTTP 401 错误。*

       ![HTTTP 401 禁止错误](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic8.png)
   
    - 检查 IIS 应用程序。 请确保已配置的应用程序池和 SPN 配置为使用 Azure AD 中的相同帐户。 在 IIS 中导航，如下图所示：
      
       ![IIS 应用程序配置窗口](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic9.png)
      
       了解标识后，确保使用上述 SPN 配置此帐户。 例如 `setspn –q http/spn.wacketywack.com`。 在命令提示符中输入以下文本：
      
       ![SetSPN 命令窗口](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic10.png)
      
    - 检查根据门户中的应用程序设置定义的 SPN。 请确保由应用程序的应用池使用根据目标 Azure AD 帐户配置的相同 SPN。

       ![Azure 门户中的 SPN 配置](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic11.png)
   
    - 转到 IIS，选择应用程序的“配置编辑器”选项。 导航到 system.webServer/security/authentication/windowsAuthentication。 请确保值 UseAppPoolCredentials 为 True。

       ![IIS 配置应用池凭据选项](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic12.png)

       将该值更改为 True。 通过运行以下命令从后端服务器删除所有缓存的 Kerberos 票证：

       ```powershell
       Get-WmiObject Win32_LogonSession | Where-Object {$_.AuthenticationPackage -ne 'NTLM'} | ForEach-Object {klist.exe purge -li ([Convert]::ToString($_.LogonId, 16))}
       ``` 

有关详细信息，请参阅 [Purge the Kerberos client ticket cache for all sessions](https://gallery.technet.microsoft.com/scriptcenter/Purge-the-Kerberos-client-b56987bf)（清除所有会话的 Kerberos 客户端票证缓存）。



如果使内核模式处于启用状态，则将提升 Kerberos 操作的性能。 但这也会导致使用计算机帐户解密请求服务的票证。 此帐户也称为本地系统。 将此值设置为 True，以在跨场中多个服务器托管应用程序时中断 KCD。

-   作为附加检查，还将禁用“扩展”保护。 在某些方案中，“扩展”保护在特定配置中启用时会中断 KCD。 在这些情况下，应用程序会作为默认网站的子文件夹发布。 仅针对匿名身份验证配置此应用程序。 所有对话框都为灰色，表明子对象不会继承任何活动设置。 建议进行测试，但若可能，请记住将此值还原为“已启用”。

    这一附加检查可使用户使用已发布的应用程序。 可启动配置为委托的其他连接器。 有关详细信息，请阅读更深入的技术演练，[故障排除 Azure AD 应用程序代理](https://aka.ms/proxytshootpaper)。

如果仍无法继续进行操作，Microsoft 支持可提供帮助。 在门户内直接创建支持票证。 工程师将与你联系。

## <a name="other-scenarios"></a>其他方案

- 在将请求发送至应用程序前，Azure 应用程序代理将请求 Kerberos 票证。 某些第三方应用程序不使用这种方法进行身份验证。 这些应用程序希望进行更多常规协商。 首个请求为匿名请求，允许应用程序响应通过 401 支持的身份验证类型。

- 多跃点身份验证常用于应用程序分层的情形，具有都需要验证身份的后端和前端，例如 SQL Server Reporting Services。 若要配置多跃点方案，请参阅支持文章 [Kerberos 约束委派可能需要在多跃点方案中进行协议转换](https://support.microsoft.com/help/2005838/kerberos-constrained-delegation-may-require-protocol-transition-in-mul)。

## <a name="next-steps"></a>后续步骤
[在托管域上配置 KCD](../../active-directory-domain-services/active-directory-ds-enable-kcd.md)。
