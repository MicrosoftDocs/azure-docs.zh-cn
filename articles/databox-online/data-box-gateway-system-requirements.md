---
title: Microsoft Azure Data Box Gateway 系统要求 | Microsoft Docs
description: 了解 Azure Data Box Gateway 的软件和网络要求
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 05/08/2019
ms.author: alkohli
ms.openlocfilehash: e8932097bcdef782b1a551d386c2872e02d8abfd
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442385"
---
# <a name="azure-data-box-gateway-system-requirements"></a>Azure 数据框网关系统要求

本文介绍了 Microsoft Azure Data Box Gateway 解决方案以及连接到 Azure Data Box Gateway 的客户端的重要系统要求。 建议在部署 Data Box Gateway 之前仔细查看信息，然后在部署和后续操作期间根据需要重新参考。

Data Box Gateway 虚拟设备的系统要求包括：

- **主机的软件要求** - 介绍了支持的平台、本地配置 UI 的浏览器、SMB 客户端以及连接到设备的主机的任何其他要求。
- **设备的网络要求** - 提供有关虚拟设备运转的网络要求的信息。


## <a name="specifications-for-the-virtual-device"></a>虚拟设备的规范

Data Box Gateway 的基础主机系统可以提供以下资源来预配虚拟设备：

| 规范                                          | 描述              |
|---------------------------------------------------------|--------------------------|
| 虚拟处理器（核心数）   | 最低为 4 核 |
| 内存  | 最低为 8 GB|
| 可用性|单节点|
| 磁盘| OS 磁盘：250 GB <br> 数据磁盘：最低为 2 TB、预配精简，并且必须由 SSD 提供支持|
| 网络接口|1 个或多个虚拟网络接口|


## <a name="supported-os-for-clients-connected-to-device"></a>连接到设备的客户端支持的 OS

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>访问设备的客户端支持的协议

[!INCLUDE [Supported protocols for clients accessing device](../../includes/data-box-edge-gateway-supported-client-protocols.md)]

## <a name="supported-virtualization-platforms-for-device"></a>设备支持的虚拟化平台

| **操作系统/平台**  |**版本**   |**说明**  |
|---------|---------|---------|
|Hyper-V  |  2012 R2 <br> 2016 <br> 2019 |         |
|VMware ESXi     | 6.0 <br> 6.5 <br> 6.7       |不支持 VMware 工具。         |


## <a name="supported-storage-accounts"></a>支持的存储帐户

[!INCLUDE [Supported storage accounts](../../includes/data-box-edge-gateway-supported-storage-accounts.md)]


## <a name="supported-storage-types"></a>支持的存储类型

[!INCLUDE [Supported storage types](../../includes/data-box-edge-gateway-supported-storage-types.md)]

## <a name="supported-browsers-for-local-web-ui"></a>本地 Web UI 支持的浏览器

[!INCLUDE [Supported browsers for local web UI](../../includes/data-box-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>网络端口要求

下表列出了需要在防火墙中打开以允许 SMB、云或管理流量的端口。 在此表中，*入*或*入站*表示传入客户端请求访问设备的方向。 *出*或*出站*表示 Data Box Gateway 设备从外部（超出部署范围）发送数据的方向：例如，到 Internet 的出站。

[!INCLUDE [Port configuration for device](../../includes/data-box-edge-gateway-port-config.md)]

## <a name="url-patterns-for-firewall-rules"></a>防火墙规则的 URL 模式

通常，网络管理员可以基于 URL 模式配置高级防火墙规则，以筛选入站和出站流量。 Data Box Gateway 设备和 Data Box Gateway 服务依赖于其他 Microsoft 应用程序，例如 Azure 服务总线、Azure Active Directory 访问控制、存储帐户和 Microsoft 更新服务器。 与这些应用程序相关联的 URL 模式可用于配置防火墙规则。 请务必了解可以更改与这些应用程序相关联的 URL 模式。 反之，这会要求网络管理员在需要时为 Data Box Gateway 监视和更新防火墙规则。

绝大多数情况下，建议基于 Data Box Gateway 固定 IP 地址为出站流量设置防火墙规则。 但是，下面的信息可用于设置创建安全环境所需的高级防火墙规则。

> [!NOTE]
> - 设备（源）IP 应始终设置为所有已启用云的网络接口。
> - 目标 IP 应设置为 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=41653)。

[!INCLUDE [URL patterns for firewall](../../includes/data-box-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-azure-government"></a>Azure 政府版的 URL 模式

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/data-box-edge-gateway-gov-url-patterns-firewall.md)]

## <a name="internet-bandwidth"></a>Internet 带宽

[!INCLUDE [Internet bandwidth](../../includes/data-box-edge-gateway-internet-bandwidth.md)]

## <a name="next-step"></a>后续步骤

* [部署 Azure Data Box Gateway](data-box-gateway-deploy-prep.md)

