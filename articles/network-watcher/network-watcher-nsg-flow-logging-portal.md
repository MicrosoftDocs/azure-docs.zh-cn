---
title: 记录出入 VM 的网络流量 - 教程 - Azure 门户 | Microsoft Docs
description: 了解如何使用网络观察程序的 NSG 流日志功能记录出入 VM 的网络流量。
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to log the network traffic to and from a VM so I can analyze it for anomalies.
ms.assetid: 01606cbf-d70b-40ad-bc1d-f03bb642e0af
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 3c62ad66a29943e26d1cb2f15ca71631d2feabe3
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467432"
---
# <a name="tutorial-log-network-traffic-to-and-from-a-virtual-machine-using-the-azure-portal"></a>教程：使用 Azure 门户记录出入虚拟机的网络流量

可以通过网络安全组 (NSG) 筛选虚拟机 (VM) 的入站和出站流量。 可以使用网络观察程序的 NSG 流日志功能记录流经 NSG 的网络流量。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 通过网络安全组创建 VM
> * 启用网络观察程序并注册 Microsoft.Insights 提供程序
> * 使用网络观察程序的 NSG 流日志功能启用 NSG 的流量日志
> * 下载记录的数据
> * 查看记录的数据

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-a-vm"></a>创建 VM

1. 选择 Azure 门户左上角的“+ 创建资源”。
2. 选择“计算”，然后选择“Windows Server 2016 Datacenter”或某一版本的“Ubuntu Server”。
3. 输入或选择以下信息，保留剩下的默认设置，然后选择“确定”：

    |设置|值|
    |---|---|
    |名称|myVm|
    |用户名| 输入所选用户名。|
    |密码| 输入所选密码。 密码必须至少 12 个字符长，且符合[定义的复杂性要求](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |订阅| 选择订阅。|
    |资源组| 选择“新建”，并输入 myResourceGroup|
    |位置| 选择“美国东部”|

4. 选择 VM 的大小，然后选择“选择”。
5. 保留“设置”下的所有默认设置，然后选择“确定”。
6. 在“摘要”中的“创建”下，选择“创建”以启动 VM 部署。 部署 VM 需要几分钟时间。 在继续余下的步骤之前，请等待 VM 完成部署。

创建 VM 需要几分钟时间。 在创建完 VM 之前，请勿继续执行剩余的步骤。 门户在创建 VM 的同时，还会创建名为 **myVm-nsg** 的网络安全组并将其关联到 VM 的网络接口。

## <a name="enable-network-watcher"></a>启用网络观察程序

如果已在“美国东部”区域中启用网络观察程序，请跳到[注册 Insights 提供程序](#register-insights-provider)。

1. 在门户中，选择“所有服务”。 在“筛选器”框中，输入“网络观察程序”。 结果中出现“网络观察程序”后，将其选中。
2. 选择“区域”，以便将其展开，然后选择“美国东部”右侧的“...”，如下图所示：

    ![启用网络观察程序](./media/network-watcher-nsg-flow-logging-portal/enable-network-watcher.png)

3. 选择“启用网络观察程序”。

## <a name="register-insights-provider"></a>注册 Insights 提供程序

NSG 流日志记录要求使用 **Microsoft.Insights** 提供程序。 若要注册该提供程序，请完成以下步骤：

1. 在门户左上角选择“所有服务”。 在“筛选器”框中，键入“订阅”。 当“订阅”出现在搜索结果中时，请将其选中。
2. 从订阅列表中选择要为其启用提供程序的订阅。
3. 在“设置”下，选择“资源提供程序”。
4. 确认 **microsoft.insights** 提供程序的“状态”为“已注册”，如下图所示。 如果状态为“未注册”，则请选择提供程序右侧的“注册”。

    ![注册提供程序](./media/network-watcher-nsg-flow-logging-portal/register-provider.png)

## <a name="enable-nsg-flow-log"></a>启用 NSG 流日志

1. NSG 流日志数据写入 Azure 存储帐户。 若要创建 Azure 存储帐户，请在门户左上角选择“+ 创建资源”。
2. 选择“存储”，然后选择“存储帐户 - Blob、文件、表、队列”。
3. 输入或选择以下信息，接受剩下的默认设置，然后选择“创建”。

    | 设置        | 值                                                        |
    | ---            | ---   |
    | 名称           | 长度为 3-24 个字符，只能包含小写字母和数字，且必须在所有 Azure 存储帐户中唯一。                                                               |
    | 位置       | 选择“美国东部”                                           |
    | 资源组 | 选择“使用现有资源组”，然后选择“myResourceGroup” |

    创建存储帐户可能需要大约一分钟的时间。 在创建好存储帐户之前，请勿继续执行剩余的步骤。 如果使用现有的存储帐户而不是创建一个，请确保在存储帐户的“设置”下针对“防火墙和虚拟网络”选择了“所有网络”（默认设置）。
    
    > [!NOTE]
    > 虽然 Azure 存储当前支持 Microsoft.Insight 和 Microsoft.Network 提供程序，但 NSG 流日志尚未完全上线。 在此功能完全上线之前，若要启用 NSG 流日志记录，还必须选择“所有网络”。 
4. 在门户左上角选择“所有服务”。 在“筛选器”框中，键入“网络观察程序”。 搜索结果中出现“网络观察程序”后，将其选中。
5. 在“日志”下选择“NSG 流日志”，如下图所示：

    ![NSG](./media/network-watcher-nsg-flow-logging-portal/nsgs.png)

6. 从 NSG 列表中选择名为 **myVm-nsg** 的 NSG。
7. 在“流日志设置”下选择“启用”。
8. 选择流日志记录版本。 版本 2 包含流会话统计信息（字节和数据包）

   ![选择流日志版本](./media/network-watcher-nsg-flow-logging-portal/select-flow-log-version.png)

9. 选择在步骤 3 中创建的存储帐户。
10. 将“保留期(天)”设置为 5，然后选择“保存”。

## <a name="download-flow-log"></a>下载流日志

1. 在门户的网络观察程序的“日志”下选择“NSG 流日志”。
2. 选择“可从配置的存储帐户下载流日志”，如下图所示：

   ![下载流日志](./media/network-watcher-nsg-flow-logging-portal/download-flow-logs.png)

3. 选择在[启用 NSG 流日志](#enable-nsg-flow-log)的步骤 2 中配置的存储帐户。
4. 在“Blob 服务”下选择“Blob”，然后选择“insights-logs-networksecuritygroupflowevent”容器。
5. 在容器中，导航浏览文件夹层次结构，直至找到 PT1H.json 文件，如下图所示。 日志文件写入遵循以下命名约定的文件夹层次结构： https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

   ![流日志](./media/network-watcher-nsg-flow-logging-portal/log-file.png)

6. 选择 PT1H.json 文件右侧的“...”，然后选择“下载”。

## <a name="view-flow-log"></a>查看流日志

下面的 json 是一个示例，说明了为每个流记录数据时在 PT1H.json 文件中会看到的内容：

### <a name="version-1-flow-log-event"></a>版本 1 流日志事件
```json
{
    "time": "2018-05-01T15:00:02.1713710Z",
    "systemId": "<Id>",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/<Id>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/MYVM-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 1,
        "flows": [
            {
                "rule": "UserRule_default-allow-rdp",
                "flows": [
                    {
                        "mac": "000D3A170C69",
                        "flowTuples": [
                            "1525186745,192.168.1.4,10.0.0.4,55960,3389,T,I,A"
                        ]
                    }
                ]
            }
        ]
    }
}
```
### <a name="version-2-flow-log-event"></a>版本 2 流日志事件
```json
{
    "time": "2018-11-13T12:00:35.3899262Z",
    "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 2,
        "flows": [
            {
                "rule": "DefaultRule_DenyAllInBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110402,94.102.49.190,10.5.16.4,28746,443,U,I,D,B,,,,",
                            "1542110424,176.119.4.10,10.5.16.4,56509,59336,T,I,D,B,,,,",
                            "1542110432,167.99.86.8,10.5.16.4,48495,8088,T,I,D,B,,,,"
                        ]
                    }
                ]
            },
            {
                "rule": "DefaultRule_AllowInternetOutBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110377,10.5.16.4,13.67.143.118,59831,443,T,O,A,B,,,,",
                            "1542110379,10.5.16.4,13.67.143.117,59932,443,T,O,A,E,1,66,1,66",
                            "1542110379,10.5.16.4,13.67.143.115,44931,443,T,O,A,C,30,16978,24,14008",
                            "1542110406,10.5.16.4,40.71.12.225,59929,443,T,O,A,E,15,8489,12,7054"
                        ]
                    }
                ]
            }
        ]
    }
}
```

在前面的输出中，**mac** 的值是在创建 VM 时创建的网络接口的 MAC 地址。 **flowTuples** 的逗号分隔信息如下所示：

| 示例数据 | 数据代表的内容   | 说明                                                                              |
| ---          | ---                    | ---                                                                                      |
| 1542110377   | 时间戳             | 表示流发生时间的时间戳，采用 UNIX EPOCH 格式。 在前面的示例中，转换后的日期为 2018 年 5 月 1 日下午 2:59:05 (GMT)。                                                                                    |
| 10.0.0.4  | 源 IP 地址      | 充当流源的源 IP 地址。 10.0.0.4 是在[创建 VM](#create-a-vm) 一文中创建的 VM 的专用 IP 地址。
| 13.67.143.118     | 目标 IP 地址 | 充当流目标的目标 IP 地址。                                                                                  |
| 44931        | Source Port            | 充当流源的源端口。                                           |
| 443         | Destination Port       | 充当流目标的目标端口。 由于流目标为端口 443，因此由日志文件中名为 **UserRule_default-allow-rdp** 的规则处理流。                                                |
| T            | 协议               | 流的协议是 TCP (T) 还是 UDP (U)。                                  |
| O            | 方向              | 流是入站 (I) 还是出站 (O)。                                     |
| A            | 操作                 | 是允许 (A) 流还是拒绝 (D) 流。  
| C            | 流状态**仅限版本 2** | 捕获流的状态。 可能的状态包括 **B**：创建流时开始。 未提供统计信息。 **C**：继续执行正在进行的流。 以 5 分钟的时间间隔提供统计信息。 **E**：在流终止时结束。 已提供统计信息。 |
| 30 | 发送的数据包数 - 源到目标**仅限版本 2** | 自上次更新以来，从源发送到目标的 TCP 或 UDP 数据包的总数。 |
| 16978 | 发送的字节数 - 源到目标**仅限版本 2** | 自上次更新以来，从源发送到目标的 TCP 或 UDP 数据包字节的总数。 数据包字节包括数据包标头和有效负载。 | 
| 24 | 发送的数据包数 - 目标到源**仅限版本 2** | 自上次更新以来，从目标发送到源的 TCP 或 UDP 数据包的总数。 |
| 14008| 发送的字节数 - 目标到源**仅限版本 2** | 自上次更新以来，从目标发送到源的 TCP 和 UDP 数据包字节的总数。 数据包字节包括数据包标头和有效负载。|

## <a name="next-steps"></a>后续步骤

本教程介绍了如何为 NSG 启用 NSG 流日志记录， 以及如何下载和查看文件中记录的数据。 json 文件中的原始数据可能难以解释。 若要将数据可视化，可以使用网络观察程序[流量分析](traffic-analytics.md)、Microsoft [PowerBI](network-watcher-visualize-nsg-flow-logs-power-bi.md) 以及其他工具。
