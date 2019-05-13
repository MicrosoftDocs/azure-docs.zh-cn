---
title: 在应用服务环境中创建内部负载均衡器 - Azure
description: 有关如何创建和使用与 Internet 隔离的 Azure 应用服务环境的详细信息
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 0f4c1fa4-e344-46e7-8d24-a25e247ae138
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 06/12/2018
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 8508dbecceb9984f53a133d9634882603549cdd1
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65199646"
---
# <a name="create-and-use-an-internal-load-balancer-with-an-app-service-environment"></a>在应用服务环境中创建和使用内部负载均衡器 #

 Azure 应用服务环境是指将 Azure App Service 部署到 Azure 虚拟网络 (VNet) 的子网中。 可通过两种方法部署应用服务环境 (ASE)： 

- 使用外部 IP 地址上的 VIP，通常称为外部 ASE。
- 使用内部 IP 地址上的 VIP，通常称为 ILB ASE，因为内部终结点是一个内部负载均衡器 (ILB)。 

本文介绍如何创建 ILB ASE。 有关 ASE 的概述，请参阅[应用服务环境简介][Intro]。 若要了解如何创建外部 ASE，请参阅[创建外部 ASE][MakeExternalASE]。

## <a name="overview"></a>概述 ##

可以使用可访问 Internet 的终结点或 VNet 中的 IP 地址部署 ASE。 若要将 IP 地址设置为 VNet 地址，必须为 ASE 部署 ILB。 为 ASE 部署 ILB 时，必须提供：

-   创建应用时所使用的自己的域。
-   用于 HTTPS 的证书。
-   域的 DNS 管理。

相对地，用户可以执行以下操作：

-   在可以通过站点到站点或 Azure ExpressRoute VPN 访问的云端，安全地托管 Intranet 应用程序。
-   在云端托管未在公用 DNS 服务器中列出的应用。
-   创建与 Internet 隔离且前端应用可以与之安全集成的后端应用。

### <a name="disabled-functionality"></a>已禁用的功能 ###

使用 ILB ASE 时，有一些操作无法执行：

-   使用基于 IP 的 SSL。
-   将 IP 地址分配给特定应用。
-   通过 Azure 门户购买证书并搭配应用使用。 可以直接从证书颁发机构获取证书并搭配应用使用。 无法通过 Azure 门户获取这些证书。

## <a name="create-an-ilb-ase"></a>创建 ILB ASE ##

若要创建 ILB ASE，请执行以下操作：

1. 在 Azure 门户中选择“创建资源” > “Web” > “应用服务环境”。

2. 选择订阅。

3. 选择或创建资源组。

4. 选择或创建 VNet。

5. 如果选择现有 VNet，需要创建子网来存放 ASE。 请确保设置的子网大小应该足够容纳 ASE 未来任何的增长。 建议的大小为 `/24`，其地址长度为 256 位且能够容纳最大尺寸的 ASE，满足任何缩放需求。 

6. 选择“虚拟网络/位置” > “虚拟网络配置”。 将“VIP 类型”设置为“内部”。

7. 输入域名。 该域将用于在此 ASE 中创建的应用。 存在一些限制。 不能是：

    * net   

    * azurewebsites.net

    * p.azurewebsites.net

    * &lt;asename&gt;.p.azurewebsites.net

   可以[将现有 DNS 名称映射到应用][customdomain]。 用于应用的自定义域名和 ASE 使用的域名不能重叠。 对于域名为 _contoso.com_ 的 ILB ASE，不能将如下所示的自定义域名用于应用：

   * www\.contoso.com

   * abcd.def.contoso.com

   * abcd.contoso.com

   如果知道应用的自定义域名，请为 ILB ASE 选择一个不会与这些自定义域名发生冲突的域。 在此示例中，可以为 ASE 的域使用类似于 *contoso internal.com* 的域名，因为它不会与以 *.contoso.com* 结尾的自定义域名发生冲突。

8. 选择“确定”，然后选择“创建”。

    ![ASE 创建][1]

“虚拟网络”边栏选项卡中有一个“虚拟网络配置”选项。 可通过它选择外部 VIP 或内部 VIP。 默认为“外部”。 如果选择“外部”，ASE 会使用可访问 Internet 的 VIP。 如果选择“内部”，则会在 VNet 中的 IP 地址上对 ASE 配置 ILB。

选择“内部”后，无法向 ASE 添加更多 IP 地址。 取而代之的是必须提供 ASE 的域。 在使用外部 VIP 的 ASE 中，ASE 的名称会用于在该 ASE 中创建的应用的域。

如果将“VIP 类型”设置为“内部”，ASE 名称将不会用于 ASE 的域。 显式指定域。 如果域为 *contoso.corp.net*，而用户在该 ASE 中创建的应用名为 *timereporting*，则该应用的 URL 为 timereporting.contoso.corp.net。


## <a name="create-an-app-in-an-ilb-ase"></a>在 ILB ASE 中创建应用 ##

在 ILB ASE 中创建应用通常与在 ASE 中创建应用相同。

1. 在 Azure 门户中，选择“创建资源” > “Web + 移动” > “Web 应用”。

1. 输入应用的名称。

1. 选择订阅。

1. 选择或创建资源组。

1. 选择 OS。 

    * 如果想要使用自定义 Docker 容器创建 Linux 应用，只需按照[此处][linuxapp]的说明自带容器即可。 

1. 选择或创建应用服务计划。 若想创建新的应用服务计划，请选择 ASE 作为位置。 选择希望在其中创建应用服务计划的辅助角色池。 创建应用服务计划时，选择 ASE 作为位置并选择辅助角色池。 指定应用的名称时，应用名称下面的域会替换为 ASE 的域。

1. 选择“创建”。 如果希望应用显示在仪表板上，请选中“固定到仪表板”复选框。

    ![应用服务计划创建][2]

    “应用名称”下面的域名会更新，以反映 ASE 的域。

## <a name="post-ilb-ase-creation-validation"></a>ILB ASE 创建后验证 ##

ILB ASE 与非 ILB ASE 稍有不同。 如先前所述，用户需要管理自己的 DNS。 还需提供自己的 HTTPS 连接证书。

创建 ASE 之后，域名显示指定的域。 “设置”菜单中出现一个名为“ILB 证书”的新项。 创建 ASE 的证书不指定 ILB ASE 的域。 如果 ASE 使用该证书，浏览器会告知该证书无效。 使用此证书可以更轻松地测试 HTTPS，但需要上传绑定到 ILB ASE 域的、用户自己的证书。 不管是自签名证书还是从证书颁发机构获取的证书，都需要执行此步骤。

![ILB ASE 域名][3]

ILB ASE 需要有效的 SSL 证书。 可使用内部证书颁发机构、向外部颁发者购买证书或使用自签名证书。 无论 SSL 证书的来源如何，都需要正确配置以下证书属性：

* **使用者**：此属性必须设置为 *\.your-root-domain-here。
* **使用者可选名称**：此属性必须同时包含 .your-root-domain-here 和 .scm.your-root-domain-here。 使用 *your-app-name.scm.your-root-domain-here* 形式的地址，建立与每个应用关联的 SCM/Kudu 站点的 SSL 连接。

将 SSL 证书转换/保存为 .pfx 文件。 .pfx 文件必须包括所有中间证书和根证书。 使用密码进行保护。

若想创建自签名证书，可以使用下面的 PowerShell 命令。 务必使用用户的 ILB ASE 域名而不是 *internal.contoso.com*： 

    $certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"
    
    $certThumbprint = "cert:\localMachine\my\" +$certificate.Thumbprint
    $password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText
    
    $fileName = "exportedcert.pfx" 
    Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password

这些 PowerShell 命令生成的证书仍然会被浏览器标记，因为该证书不是由浏览器信任链中的证书颁发机构创建的。 若要获取浏览器信任的证书，请向浏览器信任链中的商业证书颁发机构购买证书。 

![设置 ILB 证书][4]

上传自己的证书和测试访问：

1. 创建 ASE 后，转到 ASE UI。 选择“ASE” > “设置” > “ILB 证书”。

1. 若要设置 ILB 证书，请选择证书 .pfx 文件并输入密码。 此步骤需要一些时间来处理。 系统会显示一条消息，指示正在进行上传操作。

1. 获取 ASE 的 ILB 地址。 选择“ASE” > “属性” > “虚拟 IP 地址”。

2. 创建 ASE 后，在 ASE 中创建应用。

3. 如果在该 VNet 中还没有 VM，请创建一个。

    > [!NOTE] 
    > 请勿尝试在 ASE 的同一子网中创建此 VM，因为该操作会失败或导致问题。
    >

4. 设置 ASE 域的 DNS。 可以在 DNS 中将通配符与域结合使用。 若要执行一些简单测试，可编辑 VM 上的主机文件，将应用名称设置为 VIP IP 地址：

    a. 如果 ASE 的域名为 _.ilbase.com_，并且创建的应用名为 _mytestapp_，则它所在的地址为 _mytestapp.ilbase.com_。 然后设置 _mytestapp.ilbase.com_ 以解析为 ILB 地址。 （在 Windows 上，主机文件位于 _C:\Windows\System32\drivers\etc\\_。）

    b. 若要测试 Web 部署发布或访问高级控制台，请为 _mytestapp.scm.ilbase.com_ 创建一条记录。

5. 在该 VM 上使用浏览器并转到 https://mytestapp.ilbase.com。 （或者转到域中的任意应用名称。）

6. 在该 VM 上使用浏览器并转到 https://mytestapp.ilbase.com。 如果使用自签名证书，则需接受安全性不足的缺点。

    ILB 的 IP 地址在“IP 地址”下列出。 此列表还包含由外部 VIP 用于入站流量管理的 IP 地址。

    ![ILB IP 地址][5]

## <a name="web-jobs-functions-and-the-ilb-ase"></a>Web 作业、函数和 ILB ASE ##

ILB ASE 上同时支持函数和 Web 作业，但对于与其配合使用的门户，必须具有对 SCM 站点的网络访问权限。  这意味着浏览器必须位于在虚拟网络中或已连接虚拟网络的主机上。  

在 ILB ASE 上使用 Azure Functions 时，可能会收到一条错误消息，指示“目前无法检索你的函数。 请稍后再试。” 出现此错误的原因是，函数 UI 通过 HTTPS 利用 SCM 站点，并且根证书不在信任的浏览器链中。 Web 作业具有类似的问题。 要避免此问题，可执行下列操作之一：

- 将证书添加到受信任的证书存储。 这样可取消阻止 Microsoft Edge 和 Internet Explorer。
- 使用 Chrome 并首先转到 SCM 站点，接受不受信任的证书，然后再转到门户。
- 使用位于信任的浏览器链中的商业证书。  这是最佳选择。  

## <a name="dns-configuration"></a>DNS 配置 ##

使用外部 VIP 时，DNS 由 Azure 管理。 在 ASE 中创建的任何应用都会自动添加到 Azure DNS，这是一个公用 DNS。 在 ILB ASE 中，必须管理自己的 DNS。 针对给定的域（例如 _contoso.net_），必须在 DNS 中为以下项创建指向 ILB 地址的 DNS A 记录：

- *.contoso.net
- *.scm.contoso.net

如果 ILB ASE 的域用于此 ASE 外的多个操作，则可能需要基于应用名称逐个进行 DNS 管理。 此方法颇有难度，因为在创建每个新应用时都需要将其名称添加到 DNS。 为此，建议使用专用域。

## <a name="publish-with-an-ilb-ase"></a>使用 ILB ASE 发布 ##

创建的每个应用都有两个终结点。 在 ILB ASE 中，有 *&lt;应用名称>.&lt;ILB ASE 域>* 和 *&lt;应用名称>.scm.&lt;ILB ASE 域>*。 

SCM 站点名称能将用户带到 Kudu 控制台，在 Azure 门户中称为**高级门户**。 Kudu 控制台允许查看环境变量、浏览磁盘、使用控制台等等。 有关详细信息，请参阅[用于 Azure 应用服务的 Kudu 控制台][Kudu]。 

在多租户应用服务和外部 ASE 中，Azure 门户和 Kudu 控制台之间有单一登录。 然而，对于 ILB ASE，需要使用发布凭据登录到 Kudu 控制台。

如果生成代理可访问 Internet 并与 ILB ASE 在同一网络上，则基于 Internet 的 CI 系统（例如 GitHub 和 Azure DevOps）仍将使用 ILB ASE。 因此，如果在 ILB ASE 所在的 VNET 上（不同的子网属正常情况）创建生成代理，Azure DevOps 将能从 Azure DevOps git 中拉取代码并部署到 ILB ASE。 如果不想创建自己的生成代理，则需要使用利用拉取模型的 CI 系统，如 Dropbox。

ILB ASE 中应用的发布终结点使用创建该 ILB ASE 所用的域。 此域显示在应用的发布配置文件和应用的门户边栏选项卡中（“概述” > “软件包”以及“属性”）。 如果 ILB ASE 的子域为 *contoso.net*，应用名称为 *mytest*，则对 FTP 使用 *mytest.contoso.net*，对 Web 部署使用 *mytest.scm.contoso.net*。

## <a name="couple-an-ilb-ase-with-a-waf-device"></a>将 ILB ASE 与 WAF 设备耦合 ##

Azure App Service 提供大量安全措施，既可保护系统， 也有助于确定应用是否受到了黑客攻击。 Web 应用程序的最佳保护方法是将诸如 Azure App Service 之类的托管平台与 Web 应用程序防火墙 (WAF) 耦合。 ILB ASE 具有与网络隔离的应用程序终结点，因此很适合这样做。

若要深入了解如何在 ILB ASE 中配置 WAF 设备，请参阅[在应用服务环境中配置 Web 应用程序防火墙][ASEWAF]。 本文介绍如何在 ASE 中使用 Barracuda 虚拟设备。 另一种做法是使用 Azure 应用程序网关。 应用程序网关使用 OWASP 核心规则来保护放置在它后面的任何应用程序。 有关应用程序网关的详细信息，请参阅 [Azure Web 应用程序防火墙简介][AppGW]。

## <a name="get-started"></a>入门 ##

* 若要开始使用 ASE，请参阅[应用服务环境简介][Intro]。
 

<!--Image references-->
[1]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-network.png
[2]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-webapp.png
[3]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-certificate.png
[4]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-certificate2.png
[5]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-ipaddresses.png

<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[webapps]: ../overview.md
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
[customdomain]: ../app-service-web-tutorial-custom-domain.md
[linuxapp]: ../containers/app-service-linux-intro.md
