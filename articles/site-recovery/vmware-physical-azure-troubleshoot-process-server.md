---
title: 排查 Azure Site Recovery 进程服务器
description: 本文介绍如何使用 Azure Site Recovery 进程服务器的问题进行疑难解答
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: troubleshooting
ms.date: 04/29/2019
ms.author: rayne
ms.openlocfilehash: 0383226853ed86943b73d2b8740825967f3124c9
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411533"
---
# <a name="troubleshoot-the-process-server"></a>进程服务器故障排除

[Site Recovery](site-recovery-overview.md)为本地 VMware Vm 和物理服务器设置到 Azure 的灾难恢复时使用进程服务器。 本文介绍如何与进程服务器，包括复制和连接性问题的问题进行疑难解答。

[了解详细信息](vmware-physical-azure-config-process-server-overview.md)有关进程服务器。

## <a name="before-you-start"></a>开始之前

在开始之前进行故障排除：

1. 请确保你了解如何[监视的进程服务器](vmware-physical-azure-monitor-process-server.md)。
2. 查看以下最佳实践。
3. 请确保遵循[容量注意事项](site-recovery-plan-capacity-vmware.md#capacity-considerations)，并使用的大小调整指南[配置服务器](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-configuration-server-and-inbuilt-process-server)或[独立的进程服务器](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-process-server)。

## <a name="best-practices-for-process-server-deployment"></a>进程服务器部署的最佳实践

为进程服务器的最佳性能，我们已经总结数的一般最佳实践。

**最佳做法** | **详细信息**
--- |---
**使用情况** | 请确保配置服务器/独立进程服务器仅用于预期目的。 不运行任何其他计算机上。
IP 地址 | 请确保进程服务器有一个静态 IPv4 地址，并且没有 NAT 配置。
**控制内存/CPU 使用情况** |保留 CPU 和内存使用率在 70%。
**确保可用空间** | 可用空间是指在进程服务器上的缓存磁盘空间。 然后将其上载到 Azure，在缓存中存储数据。<br/><br/> 保持高于 25%的可用空间。 如果它低于 20%，复制进行限制复制的计算机与进程服务器相关联。

## <a name="check-process-server-health"></a>检查进程服务器运行状况

故障排除的第一步是要检查的运行状况和状态的进程服务器。 若要执行此操作，查看所有警报，检查所需的服务正在运行，并验证从进程服务器的检测信号。 在下图概述了这些步骤后接程序来帮助您执行步骤。

![进程服务器运行状况故障排除](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-process-server-health.png)

## <a name="step-1-troubleshoot-process-server-health-alerts"></a>步骤 1：解决进程服务器运行状况警报

进程服务器生成运行状况警报的数。 下表总结了这些警报和建议的操作。

**警报类型** | **错误** | **故障排除**
--- | --- | --- 
![状态良好][green] | 无  | 进程服务器已连接并且正常运行。
![警告][yellow] | 指定的服务不会运行。 | 1.检查服务正在运行。<br/> 2.如果服务正在按预期方式运行，按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。
![警告][yellow]  | 在过去 15 分钟的 CPU 利用率 > 80%。 | 1.不要将添加新计算机。<br/>2.使用进程服务器的 Vm 数量对应的复选[定义限制](site-recovery-plan-capacity-vmware.md#capacity-considerations)，并考虑设置[额外的进程服务器](vmware-azure-set-up-process-server-scale.md)。<br/>3.按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。
![严重][red] |  在过去 15 分钟的 CPU 利用率 > 95%。 | 1.不要将添加新计算机。<br/>2.使用进程服务器的 Vm 数量对应的复选[定义限制](site-recovery-plan-capacity-vmware.md#capacity-considerations)，并考虑设置[额外的进程服务器](vmware-azure-set-up-process-server-scale.md)。<br/>3.按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。<br/> 4.如果问题仍然存在，请运行[部署规划器](https://aka.ms/asr-v2a-deployment-planner)的 VMware/物理服务器复制。
![警告][yellow] | 过去 15 分钟内的内存使用率 > 80%。 |  1.不要将添加新计算机。<br/>2.使用进程服务器的 Vm 数量对应的复选[定义限制](site-recovery-plan-capacity-vmware.md#capacity-considerations)，并考虑设置[额外的进程服务器](vmware-azure-set-up-process-server-scale.md)。<br/>3.按照与该警告关联的任何说明进行操作。<br/> 4.如果问题仍然存在，请按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。
![严重][red] | 过去 15 分钟内的内存使用率 > 95%。 | 1.不添加新计算机，并考虑设置[额外的进程服务器](vmware-azure-set-up-process-server-scale.md)。<br/> 2.按照与该警告关联的任何说明进行操作。<br/> 3. 4. 如果问题仍然存在，按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。<br/> 4.如果问题仍然存在，请运行[部署规划器](https://aka.ms/asr-v2a-deployment-planner)VMware/物理服务器复制问题的。
![警告][yellow] | 在过去 15 分钟的缓存文件夹的可用空间 < 30%。 | 1.不添加新计算机，并请考虑设置[额外的进程服务器](vmware-azure-set-up-process-server-scale.md)。<br/>2.使用进程服务器的 Vm 数量对应的复选[指导原则](site-recovery-plan-capacity-vmware.md#capacity-considerations)。<br/> 3.按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。
![严重][red] |  过去 15 分钟内的可用空间 < 25% | 1.按照与该警告针对此问题关联的说明。<br/> 2. 3. 按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。<br/> 3.如果问题仍然存在，请运行[部署规划器](https://aka.ms/asr-v2a-deployment-planner)的 VMware/物理服务器复制。
![严重][red] | 从为 15 分钟或多个进程服务器无检测信号。 Tmansvs 服务不与配置服务器通信。 | 1） 检查进程服务器已启动并运行。<br/> 2.检查进程服务器上正在运行 tmassvc。<br/> 3.按照以下说明[对连接和复制问题进行故障排除](#check-connectivity-and-replication)。


![表键](./media/vmware-physical-azure-troubleshoot-process-server/table-key.png)


## <a name="step-2-check-process-server-services"></a>步骤 2：检查进程服务器服务

下表总结了应在进程服务器运行的服务。 有一些细微的差别在服务中，具体取决于如何部署进程服务器。 

Microsoft Azure 恢复服务代理 (obengine) 外的所有服务，都请启动类型设置为**自动**或**自动 （延迟启动）**。
 
**部署** | **正在运行的服务**
--- | ---
**在配置服务器上的进程服务器** | ProcessServer;ProcessServerMonitor;cxprocessserver;InMage PushInstall;日志上传服务 (LogUpload);InMage Scout 应用程序服务;Microsoft Azure 恢复服务代理 (obengine);InMage Scout VX 代理-Sentinel/Outpost (svagents);tmansvc;World Wide Web 发布服务 (W3SVC);MySQL;Microsoft Azure Site Recovery 服务 (dra)
**作为独立服务器运行的进程服务器** | ProcessServer;ProcessServerMonitor;cxprocessserver;InMage PushInstall;日志上传服务 (LogUpload);InMage Scout 应用程序服务;Microsoft Azure 恢复服务代理 (obengine);InMage Scout VX 代理-Sentinel/Outpost (svagents);tmansvc。
**在 Azure 中部署进行故障回复进程服务器** | ProcessServer;ProcessServerMonitor;cxprocessserver;InMage PushInstall;日志上传服务 (LogUpload)


## <a name="step-3-check-the-process-server-heartbeat"></a>步骤 3：检查进程服务器检测信号

如果没有来自进程服务器 （错误代码 806） 检测信号，请执行以下操作：

1. 验证进程服务器 VM 已启动并运行。
2. 检查这些日志中的错误。

    C:\ProgramData\ASR\home\svsystems\eventmanager *.log  C\ProgramData\ASR\home\svsystems\monitor_protection*.log

## <a name="check-connectivity-and-replication"></a>检查连接和复制

 初始和正在进行的复制故障通常是由源计算机和进程服务器之间或进程服务器与 Azure 之间的连接问题引起的。 在下图概述了这些步骤后接程序来帮助您执行步骤。

![连接和复制进行故障排除](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-connectivity-replication.png)


## <a name="step-4-verify-time-sync-on-source-machine"></a>步骤 4：验证源计算机上的时间同步

确保复制的计算机的系统日期/时间同步。[了解详细信息](https://docs.microsoft.com/windows-server/networking/windows-time-service/accurate-time)

## <a name="step-5-check-anti-virus-software-on-source-machine"></a>步骤 5：检查源计算机上的防病毒软件

检查复制的计算机上没有防病毒软件正在阻止 Site Recovery。 如果需要排除的站点恢复的防病毒程序，请查看[这篇文章](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program)。

## <a name="step-6-check-connectivity-from-source-machine"></a>步骤 6：检查源计算机之间的连接


1. 安装[Telnet 客户端](https://technet.microsoft.com/library/cc771275(v=WS.10).aspx)在需要时在源计算机上。 不要使用 Ping。
2. 源计算机执行 ping 操作上使用 Telnet 的 HTTPS 端口的进程服务器。 默认情况下 9443 是用于复制流量的 HTTPS 端口。

    `telnet <process server IP address> <port>`

3. 验证连接是否成功。


**连接** | **详细信息** | **Action**
--- | --- | ---
**Successful** | Telnet 显示空白屏幕，并且可以访问进程服务器。 | 不再需要执行操作。
**不成功** | 无法连接 | 请确保进程服务器允许入站的端口 9443。 例如，如果您具有外围网络或外围子网。 再次检查连接性。
**部分成功** | 您可以连接，但在源计算机报告无法访问进程服务器。 | 继续进行下一步的故障排除过程。

## <a name="step-7-troubleshoot-an-unreachable-process-server"></a>步骤 7：无法访问进程服务器故障排除

如果进程服务器无法访问源计算机，将显示错误 78186。 如果未解决，此问题会导致同时应用一致和崩溃一致恢复点按预期方式未生成。

通过检查源计算机可以访问进程服务器的 IP 地址，并在源计算机上，若要检查的端到端连接上运行 cxpsclient 工具进行故障排除。


### <a name="check-the-ip-connection-on-the-process-server"></a>检查进程服务器上的 IP 连接

如果 telnet 是成功但在源计算机将报告无法访问进程服务器，请检查是否可以访问进程服务器的 IP 地址。

1. 在 web 浏览器中，尝试访问 IP 地址 https://<PS_IP>:<PS_Data_Port>/。
2. 如果此检查将显示 HTTPS 证书错误，这是正常。 如果忽略该错误，应看到 400-错误请求。 这意味着服务器不能为浏览器请求提供服务，且标准的 HTTPS 连接正常。
3. 如果此检查不起作用，然后记下的浏览器错误消息。 例如，407 错误将指示代理身份验证的问题。

### <a name="check-the-connection-with-cxpsclient"></a>检查与 cxpsclient 的连接

此外，可以运行 cxpsclient 工具来检查端到端连接。

1. 运行该工具，如下所示：

    ```
    <install folder>\cxpsclient.exe -i <PS_IP> -l <PS_Data_Port> -y <timeout_in_secs:recommended 300>
    ```

2. 在进程服务器上，检查这些文件夹中生成的日志：

    C:\ProgramData\ASR\home\svsystems\transport\log\cxps.err  C:\ProgramData\ASR\home\svsystems\transport\log\cxps.xfer



### <a name="check-source-vm-logs-for-upload-failures-error-78028"></a>检查源 VM 日志上传失败 （错误 78028）

数据上传到进程服务被阻止的源计算机的问题可能会导致不生成这两种崩溃一致且应用一致恢复点。 

1. 若要对网络上传故障进行故障排除，可以查看此日志中的错误：

    C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\svagents*.log 

2. 使用本文中的过程的其余部分可以帮助你解决数据上传问题。



## <a name="step-8-check-whether-the-process-server-is-pushing-data"></a>步骤 8：检查是否在进程服务器推送的数据

检查是否在进程服务器主动将多个数据推送到 Azure。

  1. 在进程服务器上打开任务管理器（按 Ctrl+Shift+Esc）。
  2. 选择**性能**选项卡 >**打开资源监视器**。
  3. 在中**资源监视器**页上，选择**网络**选项卡。下**网络活动的进程**，检查 cbengine.exe 是否正在主动发送数据的大型 vNotolume。

       ![在网络活动的进程下的卷](./media/vmware-physical-azure-troubleshoot-process-server/cbengine.png)

  如果 cbengine.exe 未发送大量数据，请完成以下部分中的步骤。

## <a name="step-9-check-the-process-server-connection-to-azure-blob-storage"></a>步骤 9：请检查进程服务器连接到 Azure blob 存储

1. 在资源监视器中，选择**cbengine.exe**。
2. 下**TCP 连接**，检查以查看是否存在从进程服务器到 Azure 存储的连接。

  ![Cbengine.exe 和 Azure Blob 存储 URL 之间的连接](./media/vmware-physical-azure-troubleshoot-process-server/rmonitor.png)

### <a name="check-services"></a>检查服务

如果从进程服务器没有连接到 Azure blob 存储 URL，请检查服务正在运行。

1. 在控制面板中选择**Services**。
2. 验证以下服务正在运行：

    - cxprocessserver
    - InMage Scout VX Agent – Sentinel/Outpost
    - Microsoft Azure 恢复服务代理
    - Microsoft Azure Site Recovery 服务
    - tmansvc

3. 启动或重启未运行的任何服务。
4. 验证进程服务器已连接且可访问。 

## <a name="step-10-check-the-process-server-connection-to-azure-public-ip-address"></a>步骤 10： 检查进程服务器连接到 Azure 公共 IP 地址

1. 在进程服务器上，在 **%programfiles%\Microsoft Azure Recovery Services Agent\Temp**，打开最新的 CBEngineCurr.errlog 文件。
2. 在文件中，搜索**443**，或字符串**连接尝试失败，**。

  ![在 Temp 文件夹中的错误日志](./media/vmware-physical-azure-troubleshoot-process-server/logdetails1.png)

3. 如果你发现问题，可在 Azure 公用 IP 地址位于 CBEngineCurr.currLog 文件中，使用端口 443:

  `telnet <your Azure Public IP address as seen in CBEngineCurr.errlog>  443`

5. 在进程服务器上的命令行，使用 Telnet 来 ping Azure 公共 IP 地址。
6. 如果无法连接，请按照下一步的过程。

## <a name="step-11-check-process-server-firewall-settings"></a>步骤 11：检查进程服务器防火墙设置。 

检查进程服务器上的 IP 地址基于防火墙是否阻止访问。

1. 对于 IP 基于地址的防火墙规则：

    a） 下载的完整列表[Microsoft Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。

    b） 将 IP 地址范围添加到防火墙配置，以确保防火墙允许到 Azure （和默认 HTTPS 端口 443） 的通信。

    c） 允许 IP 地址范围的 Azure 区域的你的订阅和 Azure 美国西部区域 （用于访问控制和标识管理）。

2. 基于 URL 的防火墙，将添加到防火墙配置下表中列出的 Url。

    [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]  


## <a name="step-12-verify-process-server-proxy-settings"></a>步骤 12：验证进程服务器代理设置 

1. 如果使用代理服务器，请确保代理服务器名称由 DNS 服务器解析。 检查你设置注册表项中的配置服务器时提供的值**HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure Site Recovery\ProxySettings**。
2. 确保 Azure Site Recovery 代理使用相同的设置是将数据发送。

    a） 搜索**Microsoft Azure 备份**。

    b） 打开**Microsoft Azure 备份**，然后选择**操作** > **更改属性**。

    c） 在**代理配置**选项卡上，代理服务器地址应与注册表设置中所示的代理地址相同。 如果不同，请将其更改为相同的地址。

## <a name="step-13-check-bandwidth"></a>步骤 13：检查带宽

增加进程服务器与 Azure 之间的带宽，然后检查是否仍出现此问题。


## <a name="next-steps"></a>后续步骤

如需更多帮助，请在 [Azure Site Recovery 论坛](https://social.msdn.microsoft.com/Forums/azure/home?forum=hypervrecovmgr)中提问。 

[green]: ./media/vmware-physical-azure-troubleshoot-process-server/green.png
[yellow]: ./media/vmware-physical-azure-troubleshoot-process-server/yellow.png
[red]: ./media/vmware-physical-azure-troubleshoot-process-server/red.png