---
title: Azure 的 VMware 解决方案通过 CloudSimple-CloudSimple 维护和更新
description: 介绍计划的维护和更新的 CloudSimple 服务过程
author: sharaths-cs
ms.author: dikamath
ms.date: 04/30/2019
ms.topic: article
ms.service: vmware
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4dde358f10e9ac5054297ff68a0971404c0dc135
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160240"
---
# <a name="cloudsimple-maintenance-and-updates"></a>CloudSimple 维护和更新

私有云环境可具有任何单一故障点：

* ESXi 群集具有 vSphere 高可用性配置。 群集大小有复原能力的至少一个备用节点。
* 冗余的主存储提供的 vSAN，这要求至少三个节点才能针对单个故障提供保护。 vSAN 可以配置为更大的群集为提供更高的复原能力。
* 与 RAID 10 以防止存储发生故障的存储策略配置了 vCenter、 PSC 和 NSX 管理器 Vm。 Vm 受保护节点/网络故障对 vSphere HA。
* ESXi 主机具有冗余风扇和 Nic。
* TOR 和刺交换机配置为在 HA 对，以提供复原能力。

CloudSimple 持续监视以下虚拟机的运行时间和可用性，并提供的可用性 Sla:

* ESXi 主机
* vCenter
* PSC
* NSX 管理器

CloudSimple 还监视连续失败的以下：

* 硬盘
* 物理 NIC 端口
* 服务器
* 风扇
* 幂
* 交换机
* 切换端口

如果磁盘或节点发生故障，一个新的节点是自动添加到受影响的 VMware 群集，以立即将恢复到运行状况。

CloudSimple 备份、 维护和更新这些 VMware 元素中的私有云：

* ESXi
* vCenter 平台服务
* 控制器
* vSAN
* NSX

## <a name="back-up-and-restore"></a>备份和还原

CloudSimple 备份包括：

* 夜间增量备份的 vCenter、 PSC 和 DVS 规则。
* 使用 vCenter 来备份在应用程序层组件的本机 Api。
* 之前的任何更新或升级了 VMware 管理软件的自动备份。
* 在源由 vCenter，数据通过 TLS1.2 加密通道传输到 Azure 之前的数据加密。 数据存储在 Azure blob 位置跨区域复制。

可以通过打开请求还原[支持请求](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)。

## <a name="maintenance"></a>维护

CloudSimple 执行多种类型的计划内维护。

### <a name="backendinternal-maintenance"></a>后端/内部维护

这种维护通常需要重新配置物理资产或安装的软件修补程序。 它不会影响正在提供服务的资产的正常使用。 具有冗余 Nic 转到每个物理机架，正常的网络流量和私有云操作不会受到影响。 仅当你的组织应维护间隔期间使用的完全冗余的带宽，可能会注意到的性能造成影响。

### <a name="cloudsimple-portal-maintenance"></a>CloudSimple 门户维护

更新 CloudSimple 控制平面或基础结构时需要有限的服务中断一段时间。 目前，维护的时间间隔可以是为每月一次操作一样频繁。 频率会随着时间的推移拒绝。 CloudSimple 提供门户维护的通知，并保留尽可能短的时间间隔。 门户维护期间，以下服务继续工作不会影响：

* VMware 管理平面和应用程序
* vCenter 访问权限
* 所有网络和存储
* 所有 Azure 流量

### <a name="vmware-infrastructure-maintenance"></a>VMware 基础结构维护

有时有必要对 VMware 基础结构的配置进行更改。  目前，这些间隔可能每隔 1-2 月，但频率会随着时间的推移拒绝。 这种类型的维护通常可完成而无需中断 CloudSimple 服务的正常使用。 在 VMware 维护间隔，期间继续函数不会影响以下服务：

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

[使用 Veeam 的工作负荷 Vm 备份](https://docs.azure.cloudsimple.com/backup-workloads-veeam/)。