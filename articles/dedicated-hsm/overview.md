---
title: 什么是专用 HSM？ - Azure 专用 HSM | Microsoft Docs
description: 关于 Azure 专用 HSM 在 Azure 中提供符合 FIPS 140-2 级别 3 认证的密钥存储功能的概述
services: dedicated-hsm
author: barclayn
manager: barbkess
tags: azure-resource-manager
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc, seodec18
ms.date: 12/07/2018
ms.author: barclayn
ms.openlocfilehash: 88a49c9c124c5399749d2b60595d7f5c7ec77b20
ms.sourcegitcommit: 61c8de2e95011c094af18fdf679d5efe5069197b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62117986"
---
# <a name="what-is-azure-dedicated-hsm"></a>什么是 Azure 专用 HSM？

Azure 专用 HSM 是一项 Azure 服务，用于提供 Azure 中的加密密钥存储。 专用 HSM 符合最严格的安全要求。 对于需要 FIPS 140-2 级别 3 验证设备并需对 HSM 设备进行全权控制的客户来说，专用 HSM 是理想的解决方案。 

 HSM 设备在全球多个 Azure 区域部署。 可以很容易地将它们预配为一对设备并配置为高可用性。 HSM 设备还可跨区域预配，目的是防范区域级故障转移的情况。 Microsoft 使用 Gemalto 的 [SafeNet Luna 网络 HSM 7（型号：A790）](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/)设备提供专用 HSM 服务。 此设备提供最高级别的性能和加密集成选项。 

预配完成后，HSM 设备将直接连接到客户的虚拟网络。 在配置点到站点或站点到站点 VPN 连接时，也可以通过本地应用程序和管理工具访问它们。 客户会从 Gemalto 的支持门户获得配置和管理 HSM 设备所需的软件和文档。

## <a name="why-use-azure-dedicated-hsm"></a>为何使用 Azure 专用 HSM？

### <a name="fips-140-2-level-3-compliance"></a>FIPS 140-2 级别 3 符合性

许多组织有严格的行业规范，规定加密密钥存储必须符合 [FIPS 140-2 级别 3](https://csrc.nist.gov/publications/detail/fips/140/2/final) 要求。 Microsoft 的多租户 Azure Key Vault 服务目前只提供 FIPS 140-2 级别 2 认证。 Azure 专用 HSM 可以满足金融服务行业、政府机构和其他必须符合 FIPS 140-2 级别 3 要求的用户的实际需求。

### <a name="single-tenant-devices"></a>单租户设备

我们的许多客户要求使用单租户的加密存储设备。 使用 Azure 专用 HSM 服务，可以通过 Microsoft 的全球分布数据中心之一预配物理设备。 将设备预配到某个客户之后，只有该客户才能访问设备。

### <a name="full-administrative-control"></a>完全管理控制

许多客户要求进行完全的管理控制和对设备的单独访问，以便管理。 设备预配后，只有该客户可以对设备进行管理级别或应用程序级别的访问。

 客户第一次访问设备并在当时更改密码后，Microsoft 将不再有管理控制权。 从那时候起，客户就是一个真正的单租户，可以进行完全的管理控制并具有应用程序管理权限。 Microsoft 通过串行端口维护遥测的监视器级别访问权限（不是管理员角色）。 此访问权限涵盖硬件监视器，如温度、电源运行状况和风扇运行状况。 
 
 客户可以随意禁用这种需要的监视。 但是，如果禁用监视，客户不会收到 Microsoft 的主动运行状况警报。

### <a name="high-performance"></a>高性能

出于多种原因，Gemalto 设备被选中用于此项服务。 它提供多种加密算法支持、多种受支持的操作系统和广泛的 API 支持。 部署的特定机型提供卓越的性能，对于 RSA-2048 来说，性能可以达到每秒 10,000 次操作的程度。 它支持 10 个分区，这些分区可以用于唯一的应用程序实例。 该设备是低延迟、大容量、高吞吐量的设备。

### <a name="unique-cloud-based-offering"></a>唯一的基于云的产品/服务

Microsoft 了解特定用户组的特定需求。 这是唯一为新客户提供经 FIPS 140-2 级别 3 验证的专用 HSM 服务的云服务提供商，并提供相应程度的基于云和本地的应用程序集成。

## <a name="is-azure-dedicated-hsm-right-for-you"></a>Azure 专用 HSM 是否适合你？

Azure 专用 HSM 是一种专用服务，解决特定类型的大型组织的独特需求。 因此，正常情况下，很多 Azure 客户不适合使用此服务。 许多客户会发现 Azure Key Vault 服务更合适且更经济高效。 我们确定了以下标准，用于判断本服务是否符合你的要求。

### <a name="best-fit"></a>特别适合

Azure 专用 HSM 特别适合需要直接直接和单独访问 HSM 设备的“直接迁移”方案。 示例包括：

- 将应用程序从本地迁移到 Azure 虚拟机
- 将应用程序从 Amazon AWS EC2 迁移到使用 AWS Cloud HSM Classic 服务（Amazon 不向新客户提供此服务）的虚拟机
- 在 Azure 虚拟机中运行 Apache/Ngnix SSL Offload、Oracle TDE、ADCS 之类的现成软件 

### <a name="not-a-fit"></a>不适合

Azure 专用 HSM 不适合以下类型的方案：支持通过客户托管密钥进行加密的 Microsoft 云服务（例如 Azure 信息保护、Azure 磁盘加密、Azure Data Lake Store、Azure 存储、Azure SQL 数据库，以及适用于 Office 365 的客户密匙）不与 Azure 专用 HSM 集成。

### <a name="it-depends"></a>视情况而定

Azure 专用 HSM 是否适合取决于你能否做出需求和妥协的复杂组合。 例如 FIPS 140-2 级别 3 要求。 此要求很常见，且专用 HSM 目前是满足它的唯一选择。 如果这些强制要求不相关，那么通常可以选择 Azure Key Vault 和专用 HSM。 在做出决定之前评估要求。

需要权衡选择的情况包括： 

- 在客户的 Azure 虚拟机中运行的新代码
- Azure 虚拟机中的 SQL Server TDE
- Azure 存储客户端加密
- SQL Server 和 Azure SQL DB Always Encrypted

## <a name="next-steps"></a>后续步骤

这是一项高度专业化的服务。 因此，我们建议你完全理解本文档集中的关键概念，包括定价、支持和服务级别协议。 

[Gemalto 集成指南](https://safenet.gemalto.com/partners/microsoft/)有助于简化将 HSM 配置到现有虚拟网络环境中的过程。 还有一些操作指南可帮助你确定如何设置部署体系结构。

* [高可用性](high-availability.md)
* [物理安全性](physical-security.md)
* [网络](networking.md)
* [可支持性](supportability.md)
* [监视](monitoring.md)
