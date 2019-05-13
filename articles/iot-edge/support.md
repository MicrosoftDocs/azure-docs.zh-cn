---
title: 支持的操作系统、容器引擎 - Azure IoT Edge | Microsoft Docs
description: 了解可以运行 Azure IoT Edge 守护程序和运行时的操作系统以及生产设备支持的容器引擎
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/06/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
ms.openlocfilehash: 1db999f5d3172734311ecc7dbec72214a1e07e91
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65137808"
---
# <a name="azure-iot-edge-supported-systems"></a>Azure IoT Edge 支持的系统

有多种方法可用来寻求对 Azure IoT Edge 产品的支持。

**报告 bug** – 涉及 Azure IoT Edge 产品的大多数开发都是在 IoT Edge 开放源代码项目中进行的。 可以在项目的[问题页面](https://github.com/azure/iotedge/issues)上报告 bug。 修复很快就会从项目实施到产品更新中。

**Microsoft 客户支持团队** – 拥有[支持计划](https://azure.microsoft.com/support/plans/)的用户可以通过直接从 [Azure 门户](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac)创建支持票证来与 Microsoft 客户支持团队进行沟通。

**功能请求** – Azure IoT Edge 产品通过产品的 [User Voice 页面](https://feedback.azure.com/forums/907045-azure-iot-edge)跟踪功能请求。

## <a name="container-engines"></a>容器引擎
Azure IoT Edge 需要使用容器引擎启动模块，因为模块是以容器形式实现的。 Microsoft 提供了容器引擎 moby-engine 来满足此要求。 它基于 Moby 开放源代码项目。 Docker CE 和 Docker EE 是其他常用的容器引擎。 它们也基于 Moby 开放源代码项目并且与 Azure IoT Edge 兼容。 Microsoft 对使用那些容器引擎的系统提供尽力而为的支持；但是，Microsoft 没有能力为其中的问题提供修复。 因此，Microsoft 建议在生产系统上使用 moby-engine。

<br>
<center>

![小鲸鱼作为容器运行时](./media/support/only-moby-for-production.png)
</center>

## <a name="operating-systems"></a>操作系统
Azure IoT Edge 在可以运行容器的大多数操作系统上运行，但并非同等程度地支持所有这些系统。 操作系统分组为各个层级，这些层级表示用户可以预期的支持级别。
* 可以将第 1 层系统视为受官方支持。 对于第 1 层系统，Microsoft 会采取以下措施：
    * 将此操作系统包括在自动化测试中
    * 为它们提供安装程序包
* 第 2 层系统可视为与 Azure IoT Edge 兼容并且可以相对容易地使用。 对于第 2 层系统：
    * Microsoft 已在这些平台上进行了特别的测试，或者知道合作伙伴已成功在平台上运行 Azure IoT Edge
    * 适用于其他平台的安装程序包在这些平台上可能会正常工作
    
主机 OS 系列必须始终与模块容器内部使用的来宾 OS 系列相匹配。 换言之，在 Linux 上只能使用 Linux 容器，在 Windows 上只能使用 Windows 容器。 使用 Windows，支持隔离的容器，唯一进程时不 HYPER-V 隔离容器。  

<br>
<center>

![主机 OS 匹配来宾 OS](./media/support/edge-on-device.png)
</center>

### <a name="tier-1"></a>第 1 层
正式发布

| 操作系统 | AMD64 | ARM32v7 |
| ---------------- | ----- | ----- |
| Raspbian-stretch | 否 | 是|
| Ubuntu Server 16.04 | 是 | 否 |
| Ubuntu Server 18.04 | 是 | 否 |
| Windows 10 IoT 企业版 17763 | 是 | 否 |

公共预览版

| 操作系统 | AMD64 | ARM32v7 |
| ---------------- | ----- | ----- |
| Windows 10 IoT 核心版 17763 | 是 | 否 |
| Windows Server 2019 年，生成 17763 | 是 | 否 |
| Windows IoT 2019，生成 17763 | 是 | 否 |

在 Windows 上运行 Windows 容器的设备必须使用上面列出的 Windows 操作系统。 生产环境中仅支持此配置。 适用于 Windows 的 Azure IoT Edge 安装包允许在 Windows 上使用 Linux 容器；但是，只能将此配置用于开发和测试。 在生产环境中的 Windows 上使用 Linux 容器是不受支持的配置。 任何版本的 Windows 10 内部版本 14393 或更高版本，以及 Windows Server 2016 或更高版本都可用于此开发方案。

### <a name="tier-2"></a>第 2 层

| 操作系统 | AMD64 | ARM32v7 |
| ---------------- | ----- | ----- |
| CentOS 7.5 | 是 | 是 |
| Debian 8 | 是 | 是 |
| Debian 9 | 是 | 是 |
| RHEL 7.5 | 是 | 是 |
| Ubuntu 18.04 | 是 | 是 |
| Ubuntu 16.04 | 是 | 是 |
| Wind River 8 | 是 | 否 |
| Yocto | 是 | 否 |


## <a name="virtual-machines"></a>虚拟机
Azure IoT Edge 可在虚拟机中运行。 当客户想要利用边缘智能增强现有的基础结构时，经常会使用虚拟机作为 IoT Edge 设备。 主机 VM OS 系列必须与模块容器内部使用的来宾 OS 系列相匹配。 直接在设备上运行 Azure IoT Edge 时，同样要满足此要求。 Azure IoT Edge不区分底层虚拟化技术，将在以 Hyper-V 和 vSphere 等平台为后盾的 VM 中工作。

<br>
<center>

![在 VM 中的 azure IoT Edge](./media/support/edge-on-vm.png)
</center>

## <a name="minimum-system-requirements"></a>最低系统要求
在 Raspberry Pi3 等小型设备以及服务器级硬件上，Azure IoT Edge 都能非常顺利地运行。 哪种硬件适合方案取决于想要运行的工作负荷。 在设备方面做出最终决策可能比较复杂；但是，可以在传统的便携式计算机或台式机上轻松开始设计解决方案的原型。

在设计原型期间获得的经验可以帮助你在设备方面做出最终的选择。 应考虑的问题包括： 

* 工作负荷中有多少模块？
* 模块的容器共享多少层？
* 模块以什么语言编写？ 
* 模块将处理多少数据？
* 模块是否需要使用任何专用硬件来加速其工作负荷？
* 所需的解决方案性能特点有哪些？
* 硬件预算是多少？
