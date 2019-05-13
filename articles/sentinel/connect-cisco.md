---
title: 将 Cisco 数据连接到 Azure Sentinel 预览版 |Microsoft Docs
description: 了解如何将 Cisco 数据连接到 Azure Sentinel。
services: sentinel
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.assetid: 62029b5c-29d3-4336-8a22-a9db8214eb7e
ms.service: sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2019
ms.author: rkarlin
ms.openlocfilehash: 965fef42a398180475050a7273a0142a45a32305
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204427"
---
# <a name="connect-your-cisco-asa-appliance"></a>将 Cisco ASA 设备连接 

> [!IMPORTANT]
> Azure Sentinel 当前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

可以为任何 Cisco ASA 设备连接 Azure Sentinel。 Cisco ASA 本机与集成 Azure Sentinel 来引入数据，以便即使 Cisco 设备不会保存为 CEF 日志，Azure Sentinel 引入它们处理 CEF 日志的方式相同。 与 Azure Sentinel 的集成，可轻松分析和查询所有日志文件数据从运行 Cisco ASA。 

> [!NOTE]
> 数据将存储在其运行 Azure Sentinel 的工作区的地理位置。

## <a name="step-1-connect-your-cisco-asa-appliance-using-an-agent"></a>步骤 1：将 Cisco ASA 设备使用一个代理连接

若要将 Cisco ASA 设备连接到 Azure Sentinel，需要专用的计算机上部署代理 (VM 或本地) 以支持设备和 Azure Sentinel 之间的通信。 可以自动或手动部署代理。 仅当专用计算机是在 Azure 中创建的新 VM 时，才能进行自动部署。 

或者，可以在现有的 Azure VM 上、在其他云中的 VM 上或者在本地计算机上手动部署代理。

若要查看网络图的这两个选项，请参阅[数据源连接](connect-data-sources.md)。

### <a name="deploy-the-agent-in-azure"></a>部署在 Azure 中的代理

1. 在 Azure Sentinel 门户中，单击**数据连接器**，然后选择你的设备类型。 

1. 下**Linux Syslog 代理配置**:
   - 选择**自动部署**如果你想要创建预装了 Azure Sentinel 代理，并包括所有配置必要的新计算机，如上文所述。 选择**自动部署**然后单击**自动将代理部署**。 这将转到购买页自动连接到工作区，是的专用 vm。 VM 处于**标准 D2s v3 （2 个 vcpu，8 GB 内存）** 并且具有一个公共 IP 地址。
      1. 在中**自定义部署**页上，提供你的详细信息和选择用户名和密码，如果你同意条款和条件，购买的 VM。
      1. 配置你的设备，可使用连接页中列出的设置发送日志。 适用于通用的通用事件格式连接器，使用以下设置：
         - 协议 = UDP
         - 端口 = 514
         - 设施 = 本地 4
         - 格式 = CEF
   - 选择**手动部署**如果想要使用现有的 VM 作为专用 Azure Sentinel 代理应安装到其上的 Linux 计算机。 
      1. 下**下载并安装 Syslog 代理**，选择**Azure Linux 虚拟机**。 
      1. 在中**虚拟机**屏幕上，此时会打开，选择你想要使用，并单击的机**Connect**。
      1. 在连接器屏幕中下,**配置和转发的 Syslog**，将 Syslog 后台程序是否**rsyslog.d**或**syslog ng**。 
      1. 将以下命令复制并在你的设备上运行它们：
          - 如果选择了 rsyslog.d:
              
            1. 告诉 Syslog 守护程序以侦听设施 local_4 和将 Syslog 消息发送到 Azure Sentinel 代理使用端口 25226。 `sudo bash -c "printf 'local4.debug  @127.0.0.1:25226' > /etc/rsyslog.d/security-config-omsagent.conf"`
            
            2. 下载并安装[security_events 配置文件](https://aka.ms/asi-syslog-config-file-linux)配置为侦听端口 25226 上的 Syslog 代理。 `sudo wget -O /etc/opt/microsoft/omsagent/{0}/conf/omsagent.d/security_events.conf "https://aka.ms/syslog-config-file-linux"` 其中{0}应替换为你的工作区的 GUID。
            
            1. 重新启动 syslog 守护程序 `sudo service rsyslog restart`
             
          - 如果选择了 syslog ng:

              1. 告诉 Syslog 守护程序以侦听设施 local_4 和将 Syslog 消息发送到 Azure Sentinel 代理使用端口 25226。 `sudo bash -c "printf 'filter f_local4_oms { facility(local4); };\n  destination security_oms { tcp(\"127.0.0.1\" port(25226)); };\n  log { source(src); filter(f_local4_oms); destination(security_oms); };' > /etc/syslog-ng/security-config-omsagent.conf"`
              2. 下载并安装[security_events 配置文件](https://aka.ms/asi-syslog-config-file-linux)配置为侦听端口 25226 上的 Syslog 代理。 `sudo wget -O /etc/opt/microsoft/omsagent/{0}/conf/omsagent.d/security_events.conf "https://aka.ms/syslog-config-file-linux"` 其中{0}应替换为你的工作区的 GUID。

              3. 重新启动 syslog 守护程序 `sudo service syslog-ng restart`
      2. 重新启动 Syslog 代理使用以下命令： `sudo /opt/microsoft/omsagent/bin/service_control restart [{workspace GUID}]`
      1. 确认没有任何错误代理日志中通过运行以下命令： `tail /var/opt/microsoft/omsagent/log/omsagent.log`

### <a name="deploy-the-agent-on-an-on-premises-linux-server"></a>部署本地 Linux 服务器上的代理

如果不使用 Azure，手动部署 Azure Sentinel 代理专用的 Linux 服务器上运行。


1. 在 Azure Sentinel 门户中，单击**数据连接器**，然后选择你的设备类型。
1. 若要创建一个专用的 Linux VM 下,**代理配置 Linux Syslog**选择**手动部署**。
   1. 下**下载并安装 Syslog 代理**，选择**非 Azure Linux 机**。 
   1. 在中**直接代理**屏幕打开时，选择**适用于 Linux 代理**下载代理或运行以下命令来下载 Linux 计算机上：   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w {workspace GUID} -s gehIk/GvZHJmqlgewMsIcth8H6VqXLM9YXEpu0BymnZEJb6mEjZzCHhZgCx5jrMB1pVjRCMhn+XTQgDTU3DVtQ== -d opinsights.azure.com`
      1. 在连接器屏幕中下,**配置和转发的 Syslog**，将 Syslog 后台程序是否**rsyslog.d**或**syslog ng**。 
      1. 将以下命令复制并在你的设备上运行它们：
         - 如果选择了 rsyslog:
           1. 告诉 Syslog 守护程序以侦听设施 local_4 和将 Syslog 消息发送到 Azure Sentinel 代理使用端口 25226。 `sudo bash -c "printf 'local4.debug  @127.0.0.1:25226' > /etc/rsyslog.d/security-config-omsagent.conf"`
            
           2. 下载并安装[security_events 配置文件](https://aka.ms/asi-syslog-config-file-linux)配置为侦听端口 25226 上的 Syslog 代理。 `sudo wget -O /etc/opt/microsoft/omsagent/{0}/conf/omsagent.d/security_events.conf "https://aka.ms/syslog-config-file-linux"` 其中{0}应替换为你的工作区的 GUID。
           3. 重新启动 syslog 守护程序 `sudo service rsyslog restart`
         - 如果选择了 syslog ng:
            1. 告诉 Syslog 守护程序以侦听设施 local_4 和将 Syslog 消息发送到 Azure Sentinel 代理使用端口 25226。 `sudo bash -c "printf 'filter f_local4_oms { facility(local4); };\n  destination security_oms { tcp(\"127.0.0.1\" port(25226)); };\n  log { source(src); filter(f_local4_oms); destination(security_oms); };' > /etc/syslog-ng/security-config-omsagent.conf"`
            2. 下载并安装[security_events 配置文件](https://aka.ms/asi-syslog-config-file-linux)配置为侦听端口 25226 上的 Syslog 代理。 `sudo wget -O /etc/opt/microsoft/omsagent/{0}/conf/omsagent.d/security_events.conf "https://aka.ms/syslog-config-file-linux"` 其中{0}应替换为你的工作区的 GUID。
            3. 重新启动 syslog 守护程序 `sudo service syslog-ng restart`
      1. 重新启动 Syslog 代理使用以下命令： `sudo /opt/microsoft/omsagent/bin/service_control restart [{workspace GUID}]`
      1. 确认没有任何错误代理日志中通过运行以下命令： `tail /var/opt/microsoft/omsagent/log/omsagent.log`
 
## <a name="step-2-forward-cisco-asa-logs-to-the-syslog-agent"></a>步骤 2：将 Cisco ASA 日志转发到 Syslog 代理

配置 Cisco ASA 将 Syslog 消息转发到 Azure 工作区通过 Syslog 代理：

转到[到外部的 Syslog 服务器发送 Syslog 消息](https://aka.ms/asi-syslog-cisco-forwarding)，然后按照说明进行设置的连接。 使用这些参数出现提示时：
- 设置**端口**514 或代理中设置的端口。
- 设置**syslog_ip**到代理的 IP 地址。
- 设置**日志记录工具**设施中的代理设置。 默认情况下，代理将设备设置为 4。


## <a name="step-3-validate-connectivity"></a>步骤 3：验证连接

它可能需要 1-2 20 分钟，直到你的日志开始在 Log Analytics 中显示。 

1. 请确保你的日志会转到 Syslog 代理中的正确端口。 Syslog 代理计算机运行以下命令：`tcpdump -A -ni any  port 514 -vv` 此命令显示了从设备流式传输到 Syslog 计算机的日志。此命令显示了正在从设备流式传输到 Syslog 计算机的日志。 请确保源设备上的正确的端口和右设施从接收到日志。
2. 检查 Syslog 后台程序和代理之间的通信。 Syslog 代理计算机运行以下命令：`tcpdump -A -ni any  port 25226 -vv` 此命令显示了从设备流式传输到 Syslog 计算机的日志。请确保将还收到日志在代理上。
3. 如果这两个这些命令提供成功的结果，请检查 Log Analytics，请参阅正在传入到你的日志。 从这些设备流式传输的所有事件都显示在下的 Log Analytics 中的原始格式`CommonSecurityLog`类型。
1. 若要检查是否有错误或日志不会到达，查找范围 `tail /var/opt/microsoft/omsagent/<workspace id>/log/omsagent.log`
4. 请确保你 Syslog 消息的默认大小限制为 2048 个字节 (2 KB)。 如果日志太长，更新 security_events.conf 使用以下命令： `message_length_limit 4096`
6. 若要使用 Log Analytics 中的 Cisco 事件相关的架构，搜索**CommonSecurityLog**。




## <a name="next-steps"></a>后续步骤
在本文档中，您学习了如何将 Cisco ASA 设备连接到 Azure Sentinel。 要详细了解 Azure Sentinel，请参阅以下文章：
- 了解如何[来了解一下你的数据和潜在威胁](quickstart-get-visibility.md)。
- 开始[检测威胁 Azure Sentinel](tutorial-detect-threats.md)。

