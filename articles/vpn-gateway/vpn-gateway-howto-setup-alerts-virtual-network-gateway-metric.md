---
title: 设置 Azure VPN 网关指标相关警报
description: VPN 网关指标配置警报的步骤
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: conceptional
ms.date: 04/22/2019
ms.author: alzam
ms.openlocfilehash: e54dadbda0582095e8152ea30376d369177bfd86
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509910"
---
# <a name="set-up-alerts-on-vpn-gateway-metrics"></a>设置 VPN 网关指标相关警报

本文可帮助你设置 Azure VPN 网关指标相关警报。 Azure Monitor 提供的 Azure 资源设置警报的能力。 您可以设置虚拟网络网关的"VPN"类型的警报。


|**指标**   | **单位** | **粒度** | **说明** | 
|---       | ---        | ---       | ---            | ---       |
|**AverageBandwidth**| 字节/秒  | 5 分钟| 平均网关上的所有站点到站点连接的组合的带宽的使用率。     |
|**P2SBandwidth**| 字节/秒  | 1 分钟  | 平均网关上的所有点到站点连接的组合的带宽的使用率。    |
|**P2SConnectionCount**| Count  | 1 分钟  | 在网关上的点到站点连接的计数。   |
|**TunnelAverageBandwidth** | 字节/秒    | 5 分钟  | 在网关创建的隧道的平均带宽利用率。 |
|**TunnelEgressBytes** | 字节 | 5 分钟 | 在网关创建的隧道的传出流量。   |
|**TunnelEgressPackets** | Count | 5 分钟 | 在网关创建的隧道上的传出数据包计数。   |
|**TunnelEgressPacketDropTSMismatch** | Count | 5 分钟 | 在隧道引起的不匹配流量选择器上丢弃的传出数据包计数。 |
|**TunnelIngressBytes** | 字节 | 5 分钟 | 在网关创建的隧道的传入流量。   |
|**TunnelIngressPackets** | Count | 5 分钟 | 在网关创建的隧道上的传入数据包的计数。   |
|**TunnelIngressPacketDropTSMismatch** | Count | 5 分钟 | 在隧道引起的不匹配流量选择器上丢弃的传入数据包计数。 |


## <a name="setup"></a>设置 Azure Monitor 警报基于指标通过 Azure 门户

以下示例步骤将创建的网关有关的警报：

- **指标：** TunnelAverageBandwidth
- **条件：** 带宽 > 10 个字节/秒
- **窗口：** 5 分钟
- **警报操作：** 电子邮件



1. 转到虚拟网络网关资源并选择**警报**从**监视**选项卡。然后创建新的警报规则或编辑现有警报规则。

   ![用于创建警报规则的选项](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric/metric-alert1.png "创建")

2. 选择 VPN 网关的资源。

   ![选择按钮和资源的列表中的 VPN 网关](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric/metric-alert2.png "选择")

3. 选择要配置警报的指标。

   ![在度量值的列表中选择跃点数](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric/metric-alert3.png "选择")
4. 配置信号逻辑。 有三个组件：

    a. **维度**：如果该度量值维度，您可以选择特定维度值，以便警报的计算结果仅该维度的数据。 这些是可选的。

    b. **条件**：这是要评估的指标值的操作。

    c. **时间**:指定的指标数据的粒度和时间来评估警报的时间。

   ![详细信息用于配置信号逻辑](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric/metric-alert4.png "选择")

5. 若要查看配置的规则，请选择**管理警报规则**。

   ![用于管理警报规则的按钮](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric/metric-alert8.png "选择")

## <a name="next-steps"></a>后续步骤

若要配置隧道诊断日志的警报，请参阅[设置 VPN 网关诊断日志警报](vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md)。
