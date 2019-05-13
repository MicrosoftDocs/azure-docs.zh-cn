---
title: 故障排除 - Azure IoT Edge | Microsoft Docs
description: 使用本文了解 Azure IoT Edge 的标准诊断技能，例如检索组件状态和日志，以及解决常见问题
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: 02d50b81cb91a74e2cdb039c56195e2a15858ca1
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142867"
---
# <a name="common-issues-and-resolutions-for-azure-iot-edge"></a>Azure IoT Edge 的常见问题和解决方法

如果在你的环境中运行 Azure IoT Edge 时遇到问题，请使用本文作为指南来进行疑难解答并解决问题。

## <a name="run-the-iotedge-check-command"></a>运行 iotedge 检查命令

IoT Edge 进行故障排除时的第一步应该是使用`check`命令，后者将执行一系列配置和连接测试有关的常见问题。 `check`命令将出现在[释放 1.0.7](https://github.com/Azure/azure-iotedge/releases/tag/1.0.7)及更高版本。

你可以运行`check`命令，如下所示，或包括`--help`标志可查看选项的完整列表：

* 在 Linux 上：

  ```bash
  sudo iotedge check
  ```

* 在 Windows 上：

  ```powershell
  iotedge check
  ```

检查运行该工具的类型可以分类为：

* 配置检查：检查可能阻止 Edge 设备连接到云，包括问题的详细信息*config.yaml*和容器引擎。
* 连接检查：验证 IoT Edge 运行时可以访问主机设备上的端口，所有 IoT Edge 组件均可都连接到 IoT 中心。
* 生产的准备情况检查：寻找建议的生产的最佳做法，例如设备证书颁发机构 (CA) 证书和模块的日志文件配置的状态。

诊断检查的完整列表，请参阅[内置故障排除功能](https://github.com/Azure/iotedge/blob/master/doc/troubleshoot-checks.md)。

## <a name="standard-diagnostic-steps"></a>标准诊断步骤

如果遇到问题，您可了解有关详细的 IoT Edge 设备的状态通过查看容器日志和自的消息传递到设备。 可以使用本部分中的命令和工具来收集信息。

### <a name="check-the-status-of-the-iot-edge-security-manager-and-its-logs"></a>检查 IoT Edge 安全管理器和其日志的状态

在 Linux 上：
- 若要查看 IoT Edge 安全管理器的状态，请执行以下命令：

   ```bash
   sudo systemctl status iotedge
   ```

- 若要查看 IoT Edge 安全管理器的日志，请执行以下命令：

    ```bash
    sudo journalctl -u iotedge -f
    ```

- 若要查看 IoT Edge 安全管理器的更详细日志，请执行以下操作：

   - 编辑 iotedge 守护程序设置：

      ```bash
      sudo systemctl edit iotedge.service
      ```
   
   - 更新以下行：
    
      ```
      [Service]
      Environment=IOTEDGE_LOG=edgelet=debug
      ```
    
   - 重启 IoT Edge 安全守护程序：
    
      ```bash
      sudo systemctl cat iotedge.service
      sudo systemctl daemon-reload
      sudo systemctl restart iotedge
      ```

在 Windows 上：
- 若要查看 IoT Edge 安全管理器的状态，请执行以下命令：

   ```powershell
   Get-Service iotedge
   ```

- 若要查看 IoT Edge 安全管理器的日志，请执行以下命令：

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
   ```

### <a name="if-the-iot-edge-security-manager-is-not-running-verify-your-yaml-configuration-file"></a>如果 IoT Edge 安全管理器未运行，请验证 yaml 配置文件

> [!WARNING]
> YAML 文件不能包含作为缩进选项卡。 请改用 2 个空格。

在 Linux 上：

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

在 Windows 上：

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

### <a name="check-container-logs-for-issues"></a>检查容器日志是否有问题

IoT Edge 安全守护程序运行后，请查看容器日志以检测问题。 先查看你的已部署容器，然后查看构成 IoT Edge 运行时的容器：edgeAgent 和 edgeHub。 IoT Edge 代理日志通常提供有关每个容器的生命周期的信息。 IoT Edge 中心日志提供有关消息传送和路由的信息。 

   ```cmd
   iotedge logs <container name>
   ```

### <a name="view-the-messages-going-through-the-iot-edge-hub"></a>查看通过 IoT Edge 中心的消息

查看通过 IoT Edge 中心的消息，并通过来自运行时容器的详细日志收集见解。 若要在这些容器上启用详细日志，请在 yaml 配置文件中设置 `RuntimeLogLevel`。 若要打开该文件，请执行以下操作：

在 Linux 上：

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

在 Windows 上：

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

默认情况下，`agent` 元素将类似于以下示例：

   ```yaml
   agent:
     name: edgeAgent
     type: docker
     env: {}
     config:
       image: mcr.microsoft.com/azureiotedge-agent:1.0
       auth: {}
   ```

将 `env: {}` 替换为：

   ```yaml
   env:
     RuntimeLogLevel: debug
   ```

   > [!WARNING]
   > YAML 文件不能包含制表符作为缩进。 请改用 2 个空格。

保存该文件并重启 IoT Edge 安全管理器。

还可以检查在 IoT 中心与 IoT Edge 设备之间发送的消息。 使用用于 Visual Studio Code 的 [Azure IoT 中心工具包](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)扩展（以前称为 Azure IoT 工具包扩展）来查看这些消息。 有关详细信息，请参阅 [Handy tool when you develop with Azure IoT](https://blogs.msdn.microsoft.com/iotdev/2017/09/01/handy-tool-when-you-develop-with-azure-iot/)（通过 Azure IoT 进行开发时的顺手工具）。

### <a name="restart-containers"></a>重启容器
在为了解信息而调查日志和消息后，可以尝试重启容器：

```
iotedge restart <container name>
```

重启 IoT Edge 运行时容器：

```
iotedge restart edgeAgent && iotedge restart edgeHub
```

### <a name="restart-the-iot-edge-security-manager"></a>重启 IoT Edge 安全管理器

如果问题仍然存在，可以尝试重启 IoT Edge 安全管理器。

在 Linux 上：

   ```cmd
   sudo systemctl restart iotedge
   ```

在 Windows 上：

   ```powershell
   Stop-Service iotedge -NoWait
   sleep 5
   Start-Service iotedge
   ```

## <a name="iot-edge-agent-stops-after-about-a-minute"></a>IoT Edge 代理在大约一分钟后停止

edgeAgent 模块将启动并成功运行大约一分钟，然后停止。 日志表明，IoT Edge 代理尝试通过 AMQP 连接到 IoT 中心，并且尝试使用 AMQP 通过 WebSocket 进行连接。 该操作失败时，IoT Edge 代理将会退出。 

示例 edgeAgent 日志：

```output
2017-11-28 18:46:19 [INF] - Starting module management agent. 
2017-11-28 18:46:19 [INF] - Version - 1.0.7516610 (03c94f85d0833a861a43c669842f0817924911d5) 
2017-11-28 18:46:19 [INF] - Edge agent attempting to connect to IoT Hub via AMQP... 
2017-11-28 18:46:49 [INF] - Edge agent attempting to connect to IoT Hub via AMQP over WebSocket... 
```

### <a name="root-cause"></a>根本原因
主机网络上的某个网络配置阻止 IoT Edge 代理到达该网络。 代理首先会尝试通过 AMQP（端口 5671）进行连接。 如果连接失败，它将尝试 WebSocket（端口 443）。

IoT Edge 运行时会为每个模块设置要在其中进行通信的网络。 在 Linux 上，此网络是一个桥网络。 在 Windows 上，它使用 NAT。 此问题在其中的 Windows 容器使用 NAT 网络的 Windows 设备上更为常见。 

### <a name="resolution"></a>解决方法
确保分配给此桥/NAT 网络的 IP 地址具有通向 Internet 的路由。 有时候，主机上的 VPN 配置会替代 IoT Edge 网络。 

## <a name="iot-edge-hub-fails-to-start"></a>IoT Edge 中心未能启动

edgeHub 模块未能启动，并且向日志输出了以下消息： 

```output
One or more errors occurred. 
(Docker API responded with status code=InternalServerError, response=
{\"message\":\"driver failed programming external connectivity on endpoint edgeHub (6a82e5e994bab5187939049684fb64efe07606d2bb8a4cc5655b2a9bad5f8c80): 
Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is already allocated\"}\n) 
```

### <a name="root-cause"></a>根本原因
主机上的某个其他进程已绑定了端口 443。 IoT Edge 中心映射端口 5671 和 443 以在网关方案中使用。 如果另一进程已绑定了此端口，则此端口映射会失败。 

### <a name="resolution"></a>解决方法
找到并停止正在使用端口 443 的进程。 此进程通常是 Web 服务器。

## <a name="iot-edge-agent-cant-access-a-modules-image-403"></a>IoT Edge 代理无法访问某个模块的映像 (403)
某个容器未能运行，并且 edgeAgent 日志显示了 403 错误。 

### <a name="root-cause"></a>根本原因
Iot Edge 代理无权访问某个模块的映像。 

### <a name="resolution"></a>解决方法
确保在部署清单中正确指定了注册表凭据

## <a name="iot-edge-security-daemon-fails-with-an-invalid-hostname"></a>由于主机名无效，IoT Edge 安全守护程序失败

`sudo journalctl -u iotedge` 命令失败，并输出了以下消息： 

```output
Error parsing user input data: invalid hostname. Hostname cannot be empty or greater than 64 characters
```

### <a name="root-cause"></a>根本原因
IoT Edge 运行时只支持短于 64 个字符的主机名。 物理计算机通常不具有长主机名，但此问题在虚拟机上更常见。 特别是为 Azure 中托管的 Windows 虚拟机自动生成的主机名，往往会很长。 

### <a name="resolution"></a>解决方法
看到此错误时，可以配置虚拟机的 DNS 名称，然后在设置命令中将 DNS 名称设置为主机名。

1. 在 Azure 门户中，导航到虚拟机的概述页面。 
2. 选择 DNS 名称下的“配置”。 如果你的虚拟机已配置 DNS 名称，则不需要再配置。 

   ![配置虚拟机的 DNS 名称](./media/troubleshoot/configure-dns.png)

3. 为“DNS 名称标签”提供一个值，然后选择“保存”。
4. 复制新的 DNS 名称，此名称应该为 \<DNSnamelabel\>.\<vmlocation\>.cloudapp.azure.com。
5. 在虚拟机中使用下列命令，以 DNS 名称设置 IoT Edge 运行时：

   - 在 Linux 上：

      ```bash
      sudo nano /etc/iotedge/config.yaml
      ```

   - 在 Windows 上：

      ```cmd
      notepad C:\ProgramData\iotedge\config.yaml
      ```

## <a name="stability-issues-on-resource-constrained-devices"></a>有关资源受限设备的稳定性问题 
你可能会在 Raspberry Pi 等受限设备上遇到稳定性问题，尤其是在这些设备用作网关时。 症状包括 Edge 中心模块出现“内存不足”异常、下游设备无法连接或者设备在几小时后停止发送遥测消息。

### <a name="root-cause"></a>根本原因
IoT Edge 中心是 IoT Edge 运行时的一部分，默认情况下已针对性能进行了优化，并尝试分配大块内存。 这种优化对于受限 Edge 设备并不理想，并可能会导致稳定性问题。

### <a name="resolution"></a>解决方法
对于 IoT Edge 中心，请将环境变量 **OptimizeForPerformance** 设置为 **false**。 可通过两种方式实现此目的：

在 UI 中： 

在门户中，导航到“设备详细信息” > “设置模块” > “配置高级 Edge 运行时设置”。 为 Edge 中心模块创建名为 *OptimizeForPerformance*、设置为 *false* 的环境变量。

![设为 false 的 OptimizeForPerformance](./media/troubleshoot/optimizeforperformance-false.png)

**或**

在部署清单中：

```json
  "edgeHub": {
    "type": "docker",
    "settings": {
      "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
      "createOptions": <snipped>
    },
    "env": {
      "OptimizeForPerformance": {
          "value": "false"
      }
    },
```
## <a name="cant-get-the-iot-edge-daemon-logs-on-windows"></a>无法在 Windows 上获取 IoT Edge 守护程序日志
如果在 Windows 上使用 `Get-WinEvent` 时收到 EventLogException，请检查注册表项。

### <a name="root-cause"></a>根本原因
`Get-WinEvent` PowerShell 命令依赖于存在的注册表项来按特定 `ProviderName` 查找日志。

### <a name="resolution"></a>解决方法
设置 IoT Edge 守护程序的注册表项。 创建包含以下内容的 **iotedge.reg** 文件，再双击该文件或使用 `reg import iotedge.reg` 命令将其导入到 Windows 注册表中：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
"CustomSource"=dword:00000001
"EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
"TypesSupported"=dword:00000007
```

## <a name="iot-edge-module-fails-to-send-a-message-to-the-edgehub-with-404-error"></a>IoT Edge 模块无法将消息发送到 edgeHub 并出现 404 错误

自定义 IoT Edge 模块无法将消息发送到 edgeHub 并出现 404 `Module not found` 错误。 IoT Edge 守护程序在日志中输出以下消息： 

```output
Error: Time:Thu Jun  4 19:44:58 2018 File:/usr/sdk/src/c/provisioning_client/adapters/hsm_client_http_edge.c Func:on_edge_hsm_http_recv Line:364 executing HTTP request fails, status=404, response_buffer={"message":"Module not found"}u, 04 ) 
```

### <a name="root-cause"></a>根本原因
出于安全考虑，IoT Edge 守护程序会强制对连接到 edgeHub 的所有模块执行进程识别。 它会验证某个模块发送的所有消息是否来自该模块的主进程 ID。 如果发送消息的模块的进程 ID 不同于最初建立的进程 ID，则守护程序会拒绝该消息并返回 404 错误消息。

### <a name="resolution"></a>解决方法
确保自定义 IoT Edge 模块始终使用相同的进程 ID 向 Edge 中心发送消息。 例如，请确保`ENTRYPOINT`而不是`CMD`命令在 Docker 文件中，由于`CMD`将会导致一个进程模块的 ID 和 bash 命令运行的主要程序，而另一个进程 ID`ENTRYPOINT`将导致单个进程 id。


## <a name="firewall-and-port-configuration-rules-for-iot-edge-deployment"></a>IoT Edge 部署的防火墙和端口配置规则
Azure IoT Edge 允许使用支持的 IoT 中心协议从本地服务器来与 Azure 云通信，具体请参阅[选择通信协议](../iot-hub/iot-hub-devguide-protocols.md)。 为了增强安全性，Azure IoT Edge 与 Azure IoT 中心之间的信道始终配置为出站。 此配置基于[服务辅助通信模式](https://blogs.msdn.microsoft.com/clemensv/2014/02/09/service-assisted-communication-for-connected-devices/)，可最大限度地减少恶意实体可探知的攻击面。 入站通信仅在特定情况下需要，其中 Azure IoT 中心需要将消息推送到 Azure IoT Edge 设备。 使用安全的 TLS 通道来保护云到设备的消息，并且可以使用 X.509 证书和 TPM 设备模块来增强其保护。 Azure IoT Edge 安全管理器控制这种通信的建立方式，具体请参阅 [IoT Edge 安全管理器](../iot-edge/iot-edge-security-manager.md)。

IoT Edge 提供增强的配置来保护 Azure IoT Edge 运行时和已部署的模块，但仍依赖于底层计算机和网络配置。 因此，必须确保设置适当的网络和防火墙规则来保护 Edge 与云之间的通信。 为托管 Azure IoT Edge 运行时的底层服务器配置防火墙规则时，可参考下表中的指导：

|协议|端口|传入|传出|指南|
|--|--|--|--|--|
|MQTT|8883|阻止（默认）|阻止（默认）|<ul> <li>使用 MQTT 作为通信协议时，请将传出（出站）端口配置为“打开”。<li>IoT Edge 不支持将端口 1883 用于 MQTT。 <li>应阻止传入（入站）连接。</ul>|
|AMQP|5671|阻止（默认）|打开（默认）|<ul> <li>IoT Edge 的默认通信协议。 <li> 如果未为其他支持的协议配置 Azure IoT Edge，或者 AMQP 是所需的通信协议，则必须将此端口配置为“打开”。<li>IoT Edge 不支持将端口 5672 用于 AMQP。<li>当 Azure IoT Edge 使用不同的受 IoT 中心支持的协议时，请阻止此端口。<li>应阻止传入（入站）连接。</ul></ul>|
|HTTPS|443|阻止（默认）|打开（默认）|<ul> <li>将传出（出站）配置为在 443 上打开以进行 IoT Edge 预配。 使用手动脚本或 Azure IoT 设备预配服务 (DPS) 时，此配置是必需的。 <li>只应针对特定的方案打开传入（入站）连接： <ul> <li>  如果透明网关中的叶设备可能发送方法请求。 在这种情况下，无需向外部网络打开端口 443，即可连接到 IoT 中心或通过 Azure IoT Edge 提供 IoT 中心服务。 因此，传入规则可限制为只能从内部网络打开传入（入站）连接。 <li> 适用于客户端到设备 (C2D) 的方案。</ul><li>IoT Edge 不支持将端口 80 用于 HTTP。<li>如果无法在企业中配置非 HTTP 协议（例如 AMQP 或 MQTT），消息可通过 WebSocket 发送。 将这种情况下，将使用端口 443 进行 WebSocket 通信。</ul>|

## <a name="edge-agent-module-continually-reports-empty-config-file-and-no-modules-start-on-the-device"></a>Edge 代理模块持续报告“配置文件为空”，且设备上不会启动任何模块

设备在启动部署中定义的模块时出现问题。 只有 edgeAgent 在运行，但它持续报告“配置文件为空...”。

### <a name="potential-root-cause"></a>潜在的根本原因
默认情况下，IoT Edge 在模块自身的隔离容器网络中启动模块。 在此专用网络中，设备可能会遇到 DNS 名称解析方面的问题。

### <a name="resolution"></a>解决方法

**选项 1：在容器引擎设置中设置 DNS 服务器**

在要应用到引擎启动的所有容器模块的容器引擎设置中，指定环境的 DNS 服务器。 创建名为 `daemon.json` 的文件，并在其中指定要使用的 DNS 服务器。 例如：

```
{
    "dns": ["1.1.1.1"]
}
```

上面的示例将 DNS 服务器设置为可公开访问的 DNS 服务。 如果 Edge 设备无法从其所在环境访问此 IP，请将其替换为可访问的 DNS 服务器地址。

将 `daemon.json` 放入平台上的适当位置： 

| 平台 | 位置 |
| --------- | -------- |
| Linux | `/etc/docker` |
| 包含 Windows 容器的 Windows 主机 | `C:\ProgramData\iotedge-moby-data\config` |

如果该位置已包含 `daemon.json` 文件，请在其中添加 **dns** 密钥，然后保存该文件。

*重启容器引擎，使更新生效*

| 平台 | 命令 |
| --------- | -------- |
| Linux | `sudo systemctl restart docker` |
| Windows（管理 Powershell） | `Restart-Service iotedge-moby -Force` |

**选项 2：在每个模块的 IoT Edge 部署中设置 DNS 服务器**

可以针对 IoT Edge 部署中每个模块的 *createOptions* 设置 DNS 服务器。 例如：

```
"createOptions": {
  "HostConfig": {
    "Dns": [
      "x.x.x.x"
    ]
  }
}
```

同时，请务必为 *edgeAgent* 和 *edgeHub* 模块设置此项。 

## <a name="next-steps"></a>后续步骤
认为在 IoT Edge 平台中发现了 bug？ [提交问题](https://github.com/Azure/iotedge/issues)，以便我们可以持续改进。 

如果你还有其他问题，请创建[支持请求](https://portal.azure.com/#create/Microsoft.Support)以获取帮助。 

