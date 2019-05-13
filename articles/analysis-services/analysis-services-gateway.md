---
title: 本地数据网关
description: 如果 Azure 中的 Analysis Services 服务器要连接到本地数据源，则本地网关是必需的。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: b35977061b7e5806d15f4b7b0087fcafa4f291ef
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141156"
---
# <a name="connecting-to-on-premises-data-sources-with-on-premises-data-gateway"></a>使用本地数据网关连接到本地数据源
本地数据网关提供本地数据源与云中的 Azure Analysis Services 服务器之间的安全数据传输。 除了在同一区域中使用多个 Azure Analysis Services 服务器，最新版本的网关也适用于 Azure 逻辑应用、Power BI、Power Apps 和 Microsoft Flow。 可以将同一订阅和同一区域中的多个服务与单个网关进行关联。 

首次设置网关的过程分为四个部分：

- 下载并运行安装程序 - 这一步会在你组织的计算机上安装网关服务。 还在[租户的](/previous-versions/azure/azure-services/jj573650(v=azure.100)#BKMK_WhatIsAnAzureADTenant) Azure AD 中使用帐户登录到 Azure。 不支持 Azure B2B（来宾）帐户。

- **注册网关** - 在这一步中，指定网关的名称和恢复密钥，然后选择区域，在网关云服务中注册你的网关。 网关资源可以注册在任何区域中，但我们建议处于与 Analysis Services 服务器位于同一区域。 

- 在 Azure 中创建网关资源 - 在这一步中，在你的 Azure 订阅中创建网关资源。

- 将你的服务器连接到网关资源 - 在订阅中拥有网关资源后，便可以着手将你的服务器连接到该网管资源了。 可以连接多个服务器和其他资源，前提是它们位于同一订阅和同一区域中。

若要立即开始，请参阅[安装和配置本地数据网关](analysis-services-gateway-install.md)。

## <a name="how-it-works"></a>工作原理
在你组织中的计算机上安装的网关作为 Windows 服务（本地数据网关）运行。 此本地服务是通过 Azure 服务总线向网关云服务注册的。 然后，为你的 Azure 订阅创建网关资源网关云服务。 然后，将你的 Azure Analysis Services 服务器连接到网关资源。 当你服务器上的模型需要连接到你的本地数据源进行查询或处理时，查询和数据的流将遍历网关资源、Azure 服务总线、本地数据网关服务，以及你的数据源。 

![工作原理](./media/analysis-services-gateway/aas-gateway-how-it-works.png)

查询和数据流：

1. 查询是通过使用本地数据源的加密凭据进行创建的。 查询随后会发送到队列让网关处理。
2. 网关云服务分析该查询，并将请求推送到 [Azure 服务总线](https://azure.microsoft.com/documentation/services/service-bus/)。
3. 本地数据网关会针对挂起的请求轮询 Azure 服务总线。
4. 网关获取查询，对凭据进行解密，并使用这些凭据连接到数据源。
5. 网关将查询发送到数据源以便执行。
6. 结果会从数据源返回到网关，并返回到云服务和你的服务器。

## <a name="windows-service-account"> </a>Windows 服务帐户
本地数据网关配置为将 NT SERVICE\PBIEgwService 用于 Windows 服务登录凭据。 默认情况下，它有权作为服务登录；这位于正在安装网关的计算机的上下文中。 此凭据与用于连接到本地数据源或 Azure 帐户的帐户不相同。  

如果由于身份验证，代理服务器遇到问题，建议将 Windows 服务帐户更改为域用户或托管服务帐户。

## <a name="ports"> </a>端口
网关会创建与 Azure 服务总线之间的出站连接。 它在出站端口上进行通信：TCP 443（默认值）、5671、5672、9350 到 9354。  网关不需要入站端口。

我们建议在防火墙中针对数据区域列出 IP 地址允许列表。 可以下载 [Microsoft Azure 数据中心 IP 列表](https://www.microsoft.com/download/details.aspx?id=41653)。 该列表每周都会进行更新。

> [!NOTE]
> Azure 数据中心 IP 列表中列出的 IP 地址使用的是 CIDR 表示法。 若要了解详细信息，请参阅 [Classless Inter-Domain Routing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)（无类别域际路由）。
>
>

以下是该网关所用的完全限定域名。

| 域名 | 出站端口 | 描述 |
| --- | --- | --- |
| *.powerbi.com |80 |用于下载该安装程序的 HTTP。 |
| *.powerbi.com |443 |HTTPS |
| *.analysis.windows.net |443 |HTTPS |
| *.login.windows.net |443 |HTTPS |
| * .servicebus.windows.net |5671-5672 |高级消息队列协议 (AMQP) |
| * .servicebus.windows.net |443, 9350-9354 |通过 TCP 的服务总线中继上的侦听器（需要 443 来获取访问控制令牌） |
| *.frontend.clouddatahub.net |443 |HTTPS |
| *.core.windows.net |443 |HTTPS |
| login.microsoftonline.com |443 |HTTPS |
| *.msftncsi.com |443 |在 Power BI 服务无法访问网关时用于测试 Internet 连接。 |
| *.microsoftonline-p.com |443 |用于根据配置进行身份验证。 |

### <a name="force-https"></a>强制与 Azure 服务总线进行 HTTPS 通信
可以强制网关使用 HTTPS 而非直接 TCP 与 Azure 服务总线进行通信，但此操作可能会显著降低性能。 若要修改 *Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config* 文件，可将值从 `AutoDetect` 更改为 `Https`。 通常情况下，此文件位于 *C:\Program Files\On-premises data gateway*。

```
<setting name="ServiceBusSystemConnectivityModeString" serializeAs="String">
    <value>Https</value>
</setting>
```

## <a name="tenant-level-administration"></a>租户级管理 

目前没有单独的位置可让租户管理员管理其他用户安装和配置的所有网关。  如果你是租户管理员，建议要求组织中的用户将你以管理员身份添加到他们安装的每个网关。 这样，即可通过“网关设置”页面或通过 [PowerShell 命令](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters#powershell-support-for-gateway-clusters)管理组织中的所有网关。 


## <a name="faq"></a>常见问题

### <a name="general"></a>常规

**问**：对于云中的数据源（如 Azure SQL 数据库），是否需要网关？ <br/>
**答**：不。 如果仅连接到本地数据源，则网关是必需的。

**问**：网关是否必须安装在与数据源相同的计算机上？ <br/>
**答**：不。 网关只需能够连接到服务器即可，通常在同一个网络上。

<a name="why-azure-work-school-account"></a>

**问**：为何需要使用工作或学校帐户登录？ <br/>
**答**：安装本地数据网关时只能使用组织的工作或学校帐户。 而且，该帐户必须与要在其中配置网关资源的订阅在同一租户中。 登录帐户存储在由 Azure Active Directory (Azure AD) 管理的租户中。 通常，Azure AD 帐户的用户主体名称 (UPN) 与电子邮件地址匹配。

**问**：凭据存储在何处？ <br/>
**答**：为数据源输入的凭据将会加密，并存储在网关云服务中。 凭据在本地数据网关中解密。

**问**：对于网络带宽是否有要求？ <br/>
**答**：建议为网络连接配置较高的吞吐量。 每个环境是不同的，所发送的数据量会影响效果。 使用 ExpressRoute 可以帮助保证本地与 Azure 数据中心之间的吞吐量级别。
可以借助第三方工具 Azure Speed Test 应用来测量吞吐量。

**问**：从网关运行对数据源的查询时的延迟是多少？ 最佳体系结构是什么？ <br/>
**答**：若要减少网络延迟，请将网关安装在尽可能靠近数据源的位置。 如果可以在实际数据源上安装网关，这种距离可最大程度降低造成的延迟。 还需考虑数据中心。 例如，如果服务使用美国西部的数据中心，而你在 Azure VM 中托管了 SQL Server，则 Azure VM 也应该位于美国西部。 这种距离可最大程度降低延迟并避免 Azure VM 产生传出费用。

**问**：如何将结果发送回云？ <br/>
**答**：可通过 Azure 服务总线发送结果。

**问**：是否存在从云到网关的入站连接？ <br/>
**答**：不。 网关使用与 Azure 服务总线之间的出站连接。

**问**：如果阻止出站连接，会发生什么情况？ 我需要打开什么？ <br/>
**答**：查看网关使用的端口和主机。

**问**：实际 Windows 服务的名称是什么？<br/>
**答**：在“服务”中，网关名为“本地数据网关服务”。

**问**：是否可以使用 Azure Active Directory 帐户运行网关 Windows 服务？ <br/>
**答**：不。 该 Windows 服务必须具有有效的 Windows 帐户。 默认情况下，服务使用服务 SID“NT SERVICE\PBIEgwService”来运行。

**问**：如何接管网关？ <br/>
**答**：要接管网关（通过在“控制面板”>“程序”中运行“设置/更改”），你需要成为 Azure 中网关资源的所有者并且需要拥有恢复密钥。 可在访问控制中配置网管资源所有者。

### <a name="high-availability"></a>高可用性和灾难恢复

**问**：如何具有更高可用性？  
**答**：可在另一台计算机上安装网关以创建群集。 若要了解详细信息，请参阅 Power BI Gateway 文档中的[本地数据网关的高可用性群集](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters)。

**问**：有哪些选项可用于灾难恢复？ <br/>
**答**：可以使用恢复密钥还原或移动网关。 安装网关时，指定恢复密钥。

**问**：恢复密钥的好处是什么？ <br/>
**答**：使用恢复密钥可在发生灾难后迁移或恢复网关设置。

## <a name="troubleshooting"></a>故障排除

**问**：尝试在 Azure 中创建网关资源时，为什么在网关实例列表中没看到我的网关？ <br/>
**答**：有两个可能的原因。 第一种可能性是已在当前订阅或某个其他订阅中为该网关创建了资源。 若要消除该可能性，请从门户枚举“本地数据网关”类型的资源。 枚举所有资源时，请确保选择所有订阅。 创建资源后，该网关将不会出现在“创建网关资源”门户体验的网关实例列表中。 第二种可能性是安装该网关的用户的 Azure AD 标识与登录到 Azure 门户的用户不同。 若要解决此问题，请使用安装该网关的用户帐户登录到门户。

**问**：如何查看正在发送到本地数据源的查询？ <br/>
**答**：可以启用查询跟踪，包括要发送的查询。 请记得在完成故障排除后将查询跟踪改回原始值。 一直保持启用查询跟踪会创建大量的日志。

还可以查看数据源用于跟踪查询的工具。 例如，可以使用 SQL Server 的扩展事件或 SQL 事件探查器以及 Analysis Services。

**问**：网关日志在何处？ <br/>
**答**：请参阅本文中后面的“日志”。

### <a name="update"></a>更新到最新版本

如果网关版本过时，可能会出现很多问题。 良好的常规做法是确保使用最新版本。 如果有一个月或更长时间未更新网关，可能要考虑安装最新版本的网关，并确定是否可以重现问题。

### <a name="error-failed-to-add-user-to-group--2147463168-pbiegwservice-performance-log-users"></a>错误：无法将用户添加到组。 (-2147463168 PBIEgwService Performance Log Users)

如果尝试在不受支持的域控制器上安装网关，可能会收到此错误。 请确保将网关部署在不是域控制器的计算机上。

## <a name="logs"></a>日志

进行故障排除时，日志文件是一项重要资源。

#### <a name="enterprise-gateway-service-logs"></a>企业网关服务日志

`C:\Users\PBIEgwService\AppData\Local\Microsoft\On-premises data gateway\<yyyyymmdd>.<Number>.log`

#### <a name="configuration-logs"></a>配置日志

`C:\Users\<username>\AppData\Local\Microsoft\On-premises data gateway\GatewayConfigurator.log`

#### <a name="event-logs"></a>事件日志

可在“应用程序和服务日志”下找到数据管理网关和 PowerBIGateway 日志。

## <a name="next-steps"></a>后续步骤
* [安装并配置本地数据网关](analysis-services-gateway-install.md)。   
* [管理 Analysis Services](analysis-services-manage.md)
* [从 Azure Analysis Services 获取数据](analysis-services-connect.md)
