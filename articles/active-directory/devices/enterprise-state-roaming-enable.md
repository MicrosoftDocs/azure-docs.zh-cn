---
title: 在 Azure Active Directory 中启用企业状态漫游 | Microsoft Docs
description: 有关 Windows 设备中的企业状态漫游设置的常见问题。 企业状态漫游可跨 Windows 设备为用户提供统一体验，并减少配置新设备所需的时间。
services: active-directory
keywords: 企业状态漫游, Windows 云, 如何启用企业状态漫游
documentationcenter: ''
author: tanning
manager: daveba
editor: curtand
ms.subservice: devices
ms.assetid: f71d66fd-7f9e-45eb-9cfe-5d989870f8a4
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/25/2018
ms.author: joflore
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7e665f3124296392c8138f4f2bd3632b56e53de
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230930"
---
# <a name="enable-enterprise-state-roaming-in-azure-active-directory"></a>在 Azure Active Directory 中启用企业状态漫游
企业状态漫游可供任何拥有Azure AD Premium 或企业移动性 + 安全性 (EMS) 许可证的组织使用。 有关如何获取 Azure AD 订阅的详细信息，请参阅 [Azure AD 产品页](https://azure.microsoft.com/services/active-directory)。

启用企业状态漫游时，会自动授予组织对 Azure Rights Management 保护的免费受限使用许可证，该许可证来自 Azure 信息保护。 此免费订阅仅限用于加密和解密企业设置，以及由企业状态漫游同步的应用程序数据。 必须拥有[付费版订阅](https://azure.microsoft.com/pricing/details/information-protection/)才能使用 Azure Rights Management 服务的完整功能。

## <a name="to-enable-enterprise-state-roaming"></a>启用企业状态漫游

1. 登录到 [Azure AD 管理中心](https://aad.portal.azure.com/)。

1. 选择“Azure Active Directory”&gt;“设备”&gt;“企业状态漫游”。

1. 选择“用户可以跨设备同步设置和应用数据”。 有关详细信息，请参阅[如何配置设备设置](https://docs.microsoft.com/azure/active-directory/device-management-azure-portal)。
  
   ![标有“用户可以跨设备同步设置和应用数据”的设备设置的图像](./media/enterprise-state-roaming-enable/device-settings.png)
  
要让 Windows 10 设备使用企业状态漫游服务，设备必须使用 Azure AD 标识进行身份验证。 对于已加入到 Azure AD 的设备，用户的主登录标识为 Azure AD 标识，因此不需要其他配置。 对于使用本地 Active Directory 的设备，IT 管理员必须[配置加入混合 Azure Active Directory 的设备](https://docs.microsoft.com/azure/active-directory/devices/hybrid-azuread-join-manual-steps)。 

## <a name="data-storage"></a>数据存储
企业状态漫游数据托管在一个或多个 [Azure 区域](https://azure.microsoft.com/regions/)中，这些区域与 Azure Active Directory 实例中设置的国家/地区值最为匹配。 企业状态漫游数据基于三个主要地理区域进行分区：北美、EMEA 和 APAC。 租户的企业状态漫游数据位于本地所在的地理区域，不会跨区域复制。  例如:

国家/地区值 | 数据托管位置
---------------------|-------------------------
例如，法国或赞比亚 EMEA 国家/地区 | 欧洲的一个或多个 Azure 区域 
例如，美国或加拿大北美国家/地区 | 美国的一个或多个 Azure 区域
澳大利亚或新西兰等 APAC 国家/地区 | 亚洲的一个或多个 Azure 区域
南美国家/地区和南极洲区域 | 美国的一个或多个 Azure 区域

国家/地区值在 Azure AD 目录创建过程中设置，无法进行后续修改。 如需有关数据存储位置的更多详细信息，请向 [Azure 支持](https://azure.microsoft.com/support/options/)提交票证。

## <a name="view-per-user-device-sync-status"></a>查看每个用户的设备同步状态
遵循以下步骤查看每个用户的设备同步状态报告。

1. 登录到 [Azure AD 管理中心](https://aad.portal.azure.com/)。

1. 选择“Azure Active Directory”&gt;“用户”&gt;“所有用户”。

1. 选择用户，再选择“设备”。

1. 在“显示”下，选择“同步设置和应用数据的设备”显示同步状态。
  
   ![设备同步数据设置图像](./media/enterprise-state-roaming-enable/sync-status.png)
  
1. 如果存在同步此用户的数据的设备，此处会显示该设备。
  
   ![设备同步纵栏表数据图像](./media/enterprise-state-roaming-enable/device-status-row.png)

## <a name="data-retention"></a>数据保留
使用企业状态漫游同步到 Microsoft 云的数据会一直保留，直到手动将其删除，或判定相关的数据已过时。 

### <a name="explicit-deletion"></a>显式删除
当 Azure 管理员删除用户或目录或者显式请求删除数据时，将执行显式删除。

* **用户删除**：在 Azure AD 中删除某个用户时，会在 90 到 180 天后删除该用户帐户的漫游数据。 
* **目录删除**：在 Azure AD 中删除整个目录是一项即时操作。 会在 90 到 180 天后删除与该目录关联的所有设置数据。 
* **按请求删除**：如果 Azure AD 管理员要手动删除特定用户的数据或设置数据，可以通过 [Azure 支持](https://azure.microsoft.com/support/)提交票证。 

### <a name="stale-data-deletion"></a>删除过时的数据
一年（“保留期”）内未访问的数据会被视为过时，可能会从 Microsoft 云中删除。 保留期可能会发生变化，但不会少于 90 天。 过时数据可以是一组特定的 Windows/应用程序设置或用户的所有设置。 例如:

* 如果没有设备访问特定设置集合（例如从设备中删除了应用程序，或者对某用户的所有设备禁用了“主题”等设置组），则该集合会在保留期后变为过时，并可能被删除。 
* 如果用户已关闭其所有设备上的设置同步，则不会访问任何设置数据，同时该用户的所有设置数据会过时，并可以在保留期后被删除。 
* 如果 Azure AD 目录管理员关闭整个目录的企业状态漫游，则该目录中的所有用户将停止同步设置，所有用户的所有设置数据都将过时，并可能在保留期后被删除。 

### <a name="deleted-data-recovery"></a>恢复已删除的数据
数据保留策略不可配置。 永久性删除数据之后，无法将其恢复。 但是，设置数据只会从 Microsoft 云中删除，而不会从最终用户设备中删除。 如果任何设备以后重新连接到企业状态漫游服务，则会再次同步设置并将其存储在 Microsoft 云中。

## <a name="next-steps"></a>后续步骤

* [企业状态漫游概述](enterprise-state-roaming-overview.md)
* [设置和数据漫游的常见问题](enterprise-state-roaming-faqs.md)
* [设置同步的组策略和 MDM 设置](enterprise-state-roaming-group-policy-settings.md)
* [Windows 10 漫游设置参考](enterprise-state-roaming-windows-settings-reference.md)
* [故障排除](enterprise-state-roaming-troubleshooting.md)
