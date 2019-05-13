---
title: Azure Log Analytics 中的网络性能监视器解决方案 | Microsoft 文档
description: 可以使用网络性能监视器中的服务连接监视器功能，监视与 TCP 端口打开的任何终结点之间的网络连接。
services: log-analytics
documentationcenter: ''
author: abshamsft
manager: carmonm
editor: ''
ms.assetid: 5b9c9c83-3435-488c-b4f6-7653003ae18a
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/20/2018
ms.author: abshamsft
ms.openlocfilehash: 85a35207293f5afda40c78d105fc58732f06b626
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509804"
---
# <a name="service-connectivity-monitor"></a>服务连接监视器

可以使用[网络性能监视器](network-performance-monitor.md)中的服务连接监视器功能，监视与 TCP 端口打开的任何终结点之间的网络连接。 此类终结点包括网站、SaaS 应用程序、PaaS 应用程序和 SQL 数据库。 

可以使用服务连接监视器实现以下功能： 

- 监视从多个分支机构或位置到应用程序和网络服务的网络连接。 应用程序和网络服务包括 Office 365、Dynamics CRM、内部业务线应用程序和 SQL 数据库。
- 使用内置测试来监视与 Office 365 和 Dynamics 365 终结点建立的网络连接。 
- 确定在连接到终结点时经历的响应时间、网络延迟和数据包丢失情况。
- 确定应用程序性能差是由于网络问题，还是由于应用程序提供商一端出现某种问题。
- 通过查看拓扑图中每个跃点造成的延迟，来查明网络中可能导致应用程序性能差的热点。


![服务连接监视器](media/network-performance-monitor-service-endpoint/service-endpoint-intro.png)


## <a name="configuration"></a>配置 
若要打开网络性能监视器的配置，请打开[网络性能监视器解决方案](network-performance-monitor.md)并选择“配置”。

![配置网络性能监视器](media/network-performance-monitor-service-endpoint/npm-configure-button.png)


### <a name="configure-log-analytics-agents-for-monitoring"></a>配置 Log Analytics 代理的监视功能
在用于监视的节点上启用以下防火墙规则，以便解决方案能够发现从节点到服务终结点的拓扑： 

```
netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow 
```

### <a name="create-service-connectivity-monitor-tests"></a>创建服务连接监视器测试 

开始创建测试，以便监视与服务终结点建立的网络连接。

1. 选择“服务连接监视器”选项卡。
2. 选择“添加测试”并输入测试名称和说明。 可以创建每个工作区的最大 450 测试。 
3. 选择测试类型：<br>

    * 如果要监视响应 HTTP/S 请求的服务（例如 outlook.office365.com 或 bing.com）的连接，请选择“Web”。<br>
    * 如果要监视响应 TCP 请求，但不响应 HTTP/S 请求的服务（例如 SQL 服务器、FTP 服务器、SSH 端口等）的连接，请选择“网络”。 
    * 例如:若要创建 web 测试到 blob 存储帐户，请选择**Web**并输入目标，因为*yourstorageaccount*。 blob.core.windows.net。 同样，您可以创建其他表存储、 队列存储和 Azure 文件使用的测试[此链接。](https://docs.microsoft.com/azure/storage/common/storage-account-overview#storage-account-endpoints)
4. 如果你不希望执行网络度量（例如网络延迟、数据包丢失和拓扑发现），请清除“执行网络度量”复选框。 保持选中此项可以最大程度地利用此功能。 
5. 在“目标”中，输入要监视其网络连接的目标 URL/FQDN/IP 地址。
6. 在“端口号”中，输入目标服务的端口号。 
7. 在“测试频率”中，输入表示测试的运行频率的值。 
8. 选择要从中监视服务网络连接的节点。 确保添加每个测试的代理数不超过 150。 任何代理可以测试最大 150 终结点/代理。

    >[!NOTE]
    > 对于基于 Windows 服务器的节点，该功能使用基于 TCP 的请求来执行网络度量。 对于基于 Windows 客户端的节点，该功能使用基于 ICMP 的请求来执行网络度量。 在某些情况下，当节点基于 Windows 客户端时，目标应用程序会阻止传入的基于 ICMP 的请求。 解决方案无法执行网络度量。 在这种情况下，我们建议使用基于 Windows 服务器的节点。 

9. 如果不希望针对所选项生成运行状况事件，请清除“在此测试涵盖的目标中启用运行状况监视”。 
10. 选择监视条件。 可以通过输入阈值，针对运行状况事件生成设置自定义阈值。 每当条件值超出了其针对所选网络或子网对选择的阈值时，都会生成运行状况事件。 
11. 选择“保存”以保存配置。 

    ![服务连接监视器测试配置](media/network-performance-monitor-service-endpoint/service-endpoint-configuration.png)



## <a name="walkthrough"></a>演练 

转到“网络性能监视器”仪表板视图。 若要概览创建的各个测试的运行状况，请查看“服务连接监视器”页面。 

![“服务连接监视器”页](media/network-performance-monitor-service-endpoint/service-endpoint-blade.png)

可以在“测试”页上选择磁贴来查看测试详细信息。 在左侧的表中，可以查看所有测试的时间点运行状况，以及服务响应时间、网络延迟和数据包丢失值。 可以使用“网络状态记录器”控件查看过去另一时间的网络快照。 在表中选择要调查的测试。 在右侧窗格中的图表中，可以查看丢包、延迟和响应时间值的历史趋势。 可以选择“测试详细信息”链接来查看每个节点的性能。

![服务连接监视器测试](media/network-performance-monitor-service-endpoint/service-endpoint-tests.png)

在“测试节点”视图中，可以观察每个节点的网络连接。 选择性能降低的节点。 这是观察到应用程序运行速度缓慢的节点。

观察应用程序响应时间与网络延迟之间的相关性，确定应用程序性能差是由于网络问题，还是由于应用程序提供商一端出现某种问题。 

* **应用程序问题：** 如果响应时间出现峰值，但网络延迟保持一致，则表示网络正常，问题可能是由于应用程序端的问题导致的。 

    ![服务连接监视器应用程序问题](media/network-performance-monitor-service-endpoint/service-endpoint-application-issue.png)

* **网络问题：** 响应时间的峰值伴随着网络延迟的相应高峰，则表示响应时间的增加可能是由于网络延迟的增加。 

    ![服务连接监视器网络问题](media/network-performance-monitor-service-endpoint/service-endpoint-network-issue.png)

确定问题是由于网络导致的后，可以在拓扑图上选择“拓扑”视图链接来查明有问题的跃点。 下图中显示了一个示例。 节点与应用程序终结点之间的总延迟为 105 毫秒，其中，96 毫秒的延迟是带有红色标记的跃点造成的。 查明有问题的跃点后，可以采取纠正措施。 

![服务连接监视器测试](media/network-performance-monitor-service-endpoint/service-endpoint-topology.png)

## <a name="diagnostics"></a>诊断 

如果观察到异常情况，请执行以下步骤：

* 如果服务响应时间、网络断开和延迟显示为 NA，则问题可能是由下面一个或多个原因造成的：

    - 应用程序已关闭。
    - 用来检查服务的网络连接的节点已关闭。
    - 在测试配置中输入的目标不正确。
    - 节点未建立任何网络连接。

* 如果显示了有效的服务响应时间，但网络断开和延迟显示为 NA，则问题可能是由下面的一个或多个原因造成的：

    - 如果用来检查服务的网络连接的节点是 Windows 客户端计算机，则原因是目标服务正在阻止 ICMP 请求，或者网络防火墙正在阻止 ICMP 来自该节点的请求。
    - 在测试配置中，“执行网络度量”复选框为空。 

* 如果服务响应时间为 NA 但网络断开和延迟值有效，则表示目标服务可能不是 Web 应用程序。 编辑测试配置，选择“网络”而不是“Web”作为测试类型。 

* 如果应用程序运行速度缓慢，请确定应用程序性能差是由于网络问题，还是由于应用程序提供商一端出现某种问题。

## <a name="gcc-office-urls-for-us-government-customers"></a>适用于美国政府客户的 gcc 高级版 Office Url
对于美国弗吉尼亚州政府区域，仅 DOD Url 是内置 NPM。 使用 GCC Url 的客户需要创建自定义测试，并添加每个 URL 单独。

| 字段 | GCC |
|:---   |:--- |
| Office 365 门户和共享 | portal.apps.mil |
| Office 365 身份验证和标识 | * login.microsoftonline.us <br> * api.login.microsoftonline.com <br> * clientconfig.microsoftonline-p.net <br> * login.microsoftonline.com <br> * login.microsoftonline-p.com <br> * login.windows.net <br> * loginex.microsoftonline.com <br> * login-us.microsoftonline.com <br> * nexus.microsoftonline-p.com <br> * mscrl.microsoft.com <br> * secure.aadcdn.microsoftonline-p.com |
| Office Online | * adminwebservice.gov.us.microsoftonline.com <br>  * adminwebservice-s1-bn1a.microsoftonline.com <br> * adminwebservice-s1-dm2a.microsoftonline.com <br> * becws.gov.us.microsoftonline.com <br> * provisioningapi.gov.us.microsoftonline.com <br> * officehome.msocdn.us <br> * prod.msocdn.us <br> * portal.office365.us <br> * webshell.suite.office365.us <br> * www。 office365.us <br> * activation.sls.microsoft.com <br> * crl.microsoft.com <br> * go.microsoft.com <br> * insertmedia.bing.office.net <br> * ocsa.officeapps.live.com <br> * ocsredir.officeapps.live.com <br> * ocws.officeapps.live.com <br> * office15client.microsoft.com <br>* officecdn.microsoft.com <br> * officecdn.microsoft.com.edgesuite.net <br> * officepreviewredir.microsoft.com <br> * officeredir.microsoft.com <br> * ols.officeapps.live.com  <br> * r.office.microsoft.com <br> * cdn.odc.officeapps.live.com <br> * odc.officeapps.live.com <br> * officeclient.microsoft.com |
| Exchange Online | * outlook.office365.us <br> * attachments.office365-net.us <br> * autodiscover-s.office365.us <br> * manage.office365.us <br> * scc.office365.us |
| MS Teams | gov.teams.microsoft.us | 

## <a name="next-steps"></a>后续步骤
[搜索日志](../../azure-monitor/log-query/log-query-overview.md)以查看详细的网络性能数据记录。
