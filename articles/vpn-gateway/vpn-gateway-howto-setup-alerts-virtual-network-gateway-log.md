---
title: 从 Azure VPN 网关设置诊断日志事件的警报
description: 有关 VPN 网关诊断日志事件配置警报的步骤
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: conceptional
ms.date: 04/22/2019
ms.author: alzam
ms.openlocfilehash: 3880c847c54136dfd3ba1ecfe0178565091e229f
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510188"
---
# <a name="set-up-alerts-on-diagnostic-log-events-from-vpn-gateway"></a>从 VPN 网关设置诊断日志事件的警报

本文可帮助你设置基于从 Azure VPN 网关的诊断日志事件的警报。

## <a name="setup"></a>设置警报

以下示例步骤将创建一个警报，以便包括站点到站点 VPN 隧道断开连接事件：


1. 在 Azure 门户中，搜索**Log Analytics**下**的所有服务**，然后选择**Log Analytics 工作区**。

   ![选择用于跳转到 Log Analytics 工作区](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert0.png "创建")

2. 选择**创建**上**Log Analytics**页。

   ![Log Analytics 页使用创建按钮](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert1.png  "选择")

3. 选择**创建新**和填写详细信息。

   ![用于创建 Log Analytics 工作区的详细信息](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert2.png  "选择")

4. 有关 VPN 网关**监视器** > **诊断设置**边栏选项卡。

   ![选择用于在诊断设置中查找 VPN 网关](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert3.png  "选择")

5. 若要启用诊断，请双击该网关，然后选择**启用诊断**。

   ![选择启用诊断](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert4.png  "选择")

6. 填写详细信息，并确保**发送到 Log Analytics**并**TunnelDiagnosticLog**选择。 选择在步骤 3 中创建 Log Analytics 工作区。

   ![选中的复选框](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert5.png  "选择")

7. 请转到虚拟网络网关资源的概述，并选择**警报**从**监视**选项卡。然后创建新的警报规则或编辑现有警报规则。

   ![用于创建新的警报规则的选项](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "选择")

   ![点到站点](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "选择")
8. 选择 Log Analytics 工作区和资源。

   ![选择工作区和资源](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert7.png  "选择")

9. 选择**自定义日志搜索**下的信号逻辑作为**添加条件**。

   ![选择用于自定义日志搜索](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert8.png  "选择")

10. 在“搜索查询”文本框中输入以下查询。 替换为相应的 <> 中的值。

     `AzureDiagnostics |
     where Category  == "TunnelDiagnosticLog" and ResourceId == toupper("<RESOURCEID OF GATEWAY>") and TimeGenerated > ago(5m) and
     remoteIP_s == "<REMOTE IP OF TUNNEL>" and status_s == "Disconnected"`

    设置阈值的值为 0，然后选择**完成**。

    ![输入查询并选择阈值](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert9.png  "选择")

11. 上**创建规则**页上，选择**创建新**下**操作组**部分。 填写详细信息并选择**确定**。

    ![新的操作组的详细信息](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert10.png  "选择")

12. 上**创建规则**页上，填写的详细信息**自定义操作**，并确保正确名称显示在**操作组名称**部分。 选择**创建警报规则**创建规则。

    ![用于创建一条规则的选项](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert11.png  "选择")

## <a name="next-steps"></a>后续步骤

若要在隧道指标配置警报，请参阅[设置 VPN 网关指标相关警报](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md)。
