---
title: Vlan 和子网中通过 CloudSimple-Azure 的 VMware 解决方案
description: 了解有关 Vlan 和子网中 CloudSimple 私有云
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e88977cc4d99df176116e6be7d8e06adb6297782
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209584"
---
# <a name="vlans-and-subnets-overview"></a>Vlan 和子网的概述

CloudSimple 提供每个区域部署 CloudSimple 服务所在的网络。  网络是默认情况下启用了路由的单一 TCP 第 3 层地址空间。  所有私有云基础架构并在此区域中创建的子网可以进行相互通信而无需任何其他配置。  可以在使用 Vlan 在 vCenter 上创建分布式的端口组。

## <a name="vlans"></a>VLAN

每个私有云创建 Vlan （第 2 层网络）。  第 2 层流量保留在私有云，使您能够隔离私有云内的本地流量的边界内。  创建私有云上的 VLAN 用于仅在私有云创建分布式的端口组。  连接到私有云的主机的所有交换机上自动配置私有云上创建的 VLAN。

## <a name="subnets"></a>子网

当您创建的 VLAN，通过定义子网的地址空间时，可以创建一个子网。 为子网网关分配的地址空间中的 IP 地址。 单个的专用第 3 层地址空间分配每个客户和区域。 网络区域中，可以使用你的本地网络或 Azure 虚拟网络配置任何 RFC 1918 非重叠地址空间。

所有子网可以与彼此通信，默认情况下，减少开销的私有云之间的路由配置。 东-西跨同一区域中的 Pc 的数据将保留在相同的第 3 层网络，而且转移区域内的本地网络基础结构。 没有出口是必需的区域中的私有云之间的通信。 此方法可以避免在部署中不同的私有云的不同工作负荷产生任何 WAN/流出量性能损失。

## <a name="vspherevsan-subnets-cidr-range"></a>vSphere/vSAN 子网 CIDR 范围

作为独立的 VMware 堆栈 （ESXi 主机、 vCenter、 vSAN 和 NSX） 创建私有云环境由 vCenter 服务器托管。  在所选的网络中部署管理组件**vSphere/vSAN 子网 CIDR**。  在部署期间将网络 CIDR 范围划分为不同的子网中。

最小 vSphere/vSAN 子网 CIDR 范围前缀： **/24**最大值 vSphere/vSAN 子网 CIDR 范围前缀：   **/21**

### <a name="vspherevsan-subnets-cidr-range-limits"></a>vSphere/vSAN 子网 CIDR 范围限制

选择 vSphere/vSAN 子网 CIDR 范围大小会影响您的私有云的大小。  下表显示了的节点可以有根据的 vSphere/vSAN 子网 CIDR 大小最大数目。

| 指定的 vSphere/vSAN 子网 CIDR 前缀长度 | 节点数上限 |
|---------------------------------------------------|-------------------------|
| /24 | 26 |
| /23 | 58 |
| /22 | 118 |
| /21 | 220 |

### <a name="management-subnets-created-on-a-private-cloud"></a>创建私有云的管理子网

创建私有云时，会创建以下管理子网。 

* **系统管理**-VLAN 和子网的 ESXi 主机管理网络，DNS 服务器，vCenter 服务器。
* **VMotion** -VLAN 和 ESXi 主机 vMotion 网络的子网。
* **VSAN** -VLAN 和 ESXi 主机的 vSAN 网络的子网。
* **NsxtEdgeUplink1** -VLAN 和 VLAN 上行链接到外部网络的子网。
* **NsxtEdgeUplink2** -VLAN 和 VLAN 上行链接到外部网络的子网。
* **NsxtEdgeTransport** -VLAN 和子网的传输区域控制 NSX T.中的第 2 层网络的覆盖范围
* **NsxtHostTransport** -VLAN 和子网的主机的传输方式区域。

### <a name="management-network-cidr-range-breakdown"></a>管理网络 CIDR 范围细分

指定的 vSphere/vSAN 子网 CIDR 范围划分为多个子网。  下表显示了以允许前缀的细目的示例。  该示例使用**192.168.0.0**为 CIDR 范围。

示例：

| 指定的 vSphere/vSAN 子网 CIDR/前缀 | 192.168.0.0/21 | 192.168.0.0/22 | 192.168.0.0/23 | 192.168.0.0/24 |
|---------------------------------|----------------|----------------|----------------|----------------|
| 系统管理 | 192.168.0.0/24 | 192.168.0.0/24 | 192.168.0.0/25 | 192.168.0.0/26 |
| vMotion | 192.168.1.0/24 | 192.168.1.0/25 | 192.168.0.128/26 | 192.168.0.64/27 |
| vSAN | 192.168.2.0/24 | 192.168.1.128/25 | 192.168.0.192/26 | 192.168.0.96/27 |
| NSX T 主机传输 | 192.168.4.0/23 | 192.168.2.0/24 | 192.168.1.0/25 | 192.168.0.128/26 |
| NSX T 边缘传输 | 192.168.7.208/28 | 192.168.3.208/28 | 192.168.1.208/28 | 192.168.0.208/28 |
| NSX T Edge Uplink1 | 192.168.7.224/28 | 192.168.3.224/28 | 192.168.1.224/28 | 192.168.0.224/28 |
| NSX T Edge uplink2 | 192.168.7.240/28 | 192.168.3.240/28 | 192.168.1.240/28 | 192.168.0.240/28 |

## <a name="next-steps"></a>后续步骤

* [创建和管理 Vlan 和子网](https://docs.azure.cloudsimple.com/create-vlan-subnet/)