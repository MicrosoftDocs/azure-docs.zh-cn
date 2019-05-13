---
title: Azure 的 VMware 解决方案通过 CloudSimple-私有云 VMware 组件
description: 介绍如何在私有云上安装 VMware 组件
author: sharaths-cs
ms.author: dikamath
ms.date: 04/30/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 5e6548a5a04e32b374a8a9c29d2ca5f89fd65c78
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160180"
---
# <a name="private-cloud-vmware-components"></a>私有云 VMware 组件

私有云是一个独立的 VMware 堆栈 （ESXi 主机、 vCenter、 vSAN 和 NSX） 受 vCenter 服务器管理域中的环境。  CloudSimple 服务，可将 VMware 以本机模式部署 Azure 裸机裸机基础结构上的 Azure 位置中。  私有云与 Azure 云的其余部分集成。  私有云部署具有以下 VMware 堆栈组件：

* **VMware ESXi-** 在 Azure 上的虚拟机监控程序专用节点
* **VMware vCenter-** 私有云 vSphere 环境的集中式管理的设备
* **VMware 的 vSAN-** 超聚合基础结构解决方案
* **VMware NSX 数据中心-** 网络虚拟化和安全软件  

## <a name="vmware-component-versions"></a>VMware 组件版本

使用以下软件版本进行部署私有云 VMware 堆栈。

| 组件 | Version | 许可的版本 |
|-----------|---------|------------------|
| ESXi | 6.7U1 | 企业加上 |
| vCenter | 6.7U1 | vCenter 标准 |
| vSAN | 6.7 | Enterprise |
| NSX 数据中心 | 2.3 | 高级 |

## <a name="esxi"></a>ESXi

创建私有云时，已购买 CloudSimple 节点上安装 VMware ESXi。  ESXi 提供工作负荷的虚拟机 (Vm) 部署的虚拟机监控程序。  节点在私有云上提供超聚合基础结构 （计算和存储）。  节点是私有云上的 vSphere 群集的一部分。  每个节点具有四个物理网络接口连接到是网络。  使用两个物理网络接口来创建**vSphere 分布式交换机 (VDS)** vCenter 和两个用于创建**NSX 托管虚拟分布式的交换机 (N VDS)**。  在以实现高可用性的主动-主动模式下配置网络接口。

了解 VMware ESXi 上的详细信息

## <a name="vcenter-server-appliance"></a>vCenter 服务器设备

vCenter server 设备 (VCSA) 提供用于通过 CloudSimple VMware 解决方案的身份验证、 管理和业务流程的函数。 创建私有云时，会部署 VCSA 与嵌入式平台服务控制器 (PSC)。  VCSA 部署私有云时创建的 vSphere 群集上进行部署。  每个私有云中都有其自己 VCSA。  私有云的扩展将节点添加到私有云上 VCSA。

### <a name="vcenter-single-sign-on"></a>vCenter 上单一登录

与关联嵌入式的平台服务控制器上 VCSA **vCenter 上单一登录域**。  域名是**cloudsimple.local**。  默认用户**CloudOwner@cloudsimple.com**创建可访问 vCenter。  可以添加你的本地/Azure active directory [vCenter 的标识源](https://docs.azure.cloudsimple.com/set-vcenter-identity/)。

## <a name="vsan-storage"></a>vSAN 存储

完全配置的所有闪存 vSAN 存储到群集本地创建私有云。  在同一 SKU 的最小的三个节点所需的 vSAN 数据存储创建 vSphere 群集。  重复数据删除和压缩默认情况下启用 vSAN 数据存储上。  每个 vSphere 群集节点上创建两个磁盘组。 每个磁盘组包含一个缓存磁盘和三个容量的磁盘。

VSphere 群集上创建默认 vSAN 存储策略并将其应用到 vSAN 数据存储。  此策略将确定如何预配和分配的数据存储，以确保所需的服务级别中 VM 存储对象。  存储策略定义**容错 (FTT) 失败**并**失败容差方法**。  您可以创建新的存储策略，并将其应用到的 Vm。 若要维护的 SLA，必须在 vSAN 数据存储上维护 25%的备用容量。  

### <a name="default-vsan-storage-policy"></a>默认的 vSAN 存储策略

下表显示的默认 vSAN 存储策略参数。

| VSphere 群集中的节点数 | FTT | 失败容差方法 |
|------------------------------------|-----|--------------------------|
| 3 到 4 个节点 | 第 | RAID 1 （镜像）-创建 2 个副本 |
| 5 到 16 个节点 | 2 | RAID 1 （镜像）-创建 3 个副本 |

## <a name="nsx-data-center"></a>NSX 数据中心

NSX 数据中心提供了在私有云上的网络虚拟化、 微分段和网络安全功能。  你可以配置受 NSX NSX 通过在私有云上的数据中心的所有服务。  在创建私有云时，以下 NSX 组件安装和配置。

* NSXT 管理器
* 传输区域
* 主机和 Edge 上行配置文件
* 边缘传输、 Ext1，和 Ext2 逻辑交换机
* ESXi 传输节点的 IP 池
* 边缘传输节点的 IP 池
* 边缘节点
* DRS 反相关性规则适用于控制器和边缘 Vm
* 第 0 层路由器
* 启用第 0 层路由器上的 BGP

## <a name="vsphere-cluster"></a>vSphere 群集

ESXi 主机配置为群集，以便确保私有云的高可用性。  创建私有云时，第一个群集上部署 vSphere 管理组件。  为管理组件创建一个资源池和所有管理 Vm 都部署在此资源池。 无法删除第一个群集收缩私有云。  vSphere 群集提供高可用性的 Vm 使用**vSphere HA**。  可承受的失败基于群集中的可用节点数。  您可以使用公式```Number of nodes = 2N+1```其中```N```是可承受的失败数。

### <a name="vsphere-cluster-limits"></a>vSphere 群集限制

| 资源 | 限制 |
|----------|-------|
| 要创建私有云的节点的最小数量 （第一个 vSphere 群集） | 3 |
| 在私有云上群集 vSphere 中的节点的最大数目 | 16 |
| 在私有云中的节点最大数目 | 64 |
| 最大数量的 vSphere 中私有云的群集 | 21 |
| 新的 vSphere 群集上的节点的最小数目 | 3 |

## <a name="vmware-infrastructure-maintenance"></a>VMware 基础结构维护

有时有必要对 VMware 基础结构的配置进行更改。 目前，这些间隔可能每隔 1-2 月，但频率会随着时间的推移拒绝。 这种类型的维护通常可完成而无需中断 CloudSimple 服务的正常使用。 在 VMware 维护间隔，期间继续函数不会影响以下服务：

* VMware 管理平面和应用程序
* vCenter 访问权限
* 所有网络和存储
* 所有 Azure 流量

## <a name="updates-and-upgrades"></a>更新和升级

CloudSimple 负责 VMware 中的软件 （ESXi、 vCenter、 PSC 和 NSX） 私有云的生命周期管理。

软件更新包括：

* **修补程序**。 安全修补程序或由 VMware 发布的 bug 修复。
* **更新**。 VMware 堆栈组件的次要版本更改。
* **升级**。 VMware 堆栈组件的主版本更改。

CloudSimple 测试关键安全修补程序，只要它变为可从 VMware。 SLA，每 CloudSimple 推出的安全修补程序的私有云环境到一周内。

CloudSimple 提供每季度维护更新 VMware 的软件组件。 新的主版本的 VMware 软件不可用，CloudSimple 与客户协作以协调合适的维护时段，以便升级。  

## <a name="next-steps"></a>后续步骤

* [CloudSimple 维护和更新](cloudsimple-maintenance-updates.md)