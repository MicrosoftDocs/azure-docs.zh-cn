---
title: Azure AD Connect：启用设备写回 | Microsoft Docs
description: 本文档详细说明如何使用 Azure AD Connect 启用设备写回功能
services: active-directory
documentationcenter: ''
author: billmath
manager: femila
editor: curtand
ms.assetid: c0ff679c-7ed5-4d6e-ac6c-b2b6392e7892
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/08/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 64e3e7928d9f39b9ed2a681eb8ec9a63f73ea54a
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507972"
---
# <a name="azure-ad-connect-enabling-device-writeback"></a>Azure AD Connect：启用设备写回
> [!NOTE]
> 设备写回需要 Azure AD Premium 订阅。
> 
> 

以下文档提供有关如何在 Azure AD Connect 中启用设备写回功能的信息。 设备写回用于以下方案：

* 启用[Windows hello 企业版使用混合证书信任部署](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-cert-trust-prereqs#device-registration)
* 对 ADFS（2012 R2 或更高版本）保护的应用程序（信赖方信任），启用基于设备的条件性访问。

这可以提供额外的安全性，确保只有受信任的设备才能访问应用程序。 有关条件性访问的详细信息，请参阅[使用条件性访问管理风险](../active-directory-conditional-access-azure-portal.md)和[使用 Azure Active Directory Device Registration 设置本地的条件性访问](../../active-directory/active-directory-device-registration-on-premises-setup.md)。

> [!IMPORTANT]
> <li>设备必须位于用户所在的同一个林中。 由于设备必须写回到单个林，此功能当前不支持具有多个用户林的部署。</li>
> <li>只能将一个设备注册配置对象添加到本地 Active Directory 林。 此功能与本地 Active Directory 同步到多个 Azure AD 目录的拓扑不兼容。</li>

## <a name="part-1-install-azure-ad-connect"></a>第 1 部分：安装 Azure AD Connect
使用自定义或快速设置安装 Azure AD Connect。 Microsoft 建议在启用设备写回之前，首先让所有用户和组成功完成同步。

## <a name="part-2-enable-device-writeback-in-azure-ad-connect"></a>第 2 部分：在 Azure AD Connect 中启用设备写回
1. 再次运行安装向导。 从“其他任务”页中选择“配置设备选项”，并单击“下一步”。 

    ![配置设备选项](./media/how-to-connect-device-writeback/deviceoptions.png)

    >[!NOTE]
    > 新的配置设备选项仅在版本 1.1.819.0 及更新版本中可用。

2. 在设备选项页上，选择“配置设备写回”。 选项“禁用设备写回”将不可用，直到启用“设备写回”。 单击“下一步”移到向导中的下一页。
    ![选择设备操作](./media/how-to-connect-device-writeback/configuredevicewriteback1.png)

3. 在写回页中，会看到提供的域是默认的设备写回林。
   ![自定义安装 - 设备写回目标林](./media/how-to-connect-device-writeback/writebackforest.png)

4. “设备容器”页提供了使用以下两个可用选项之一准备活动目录的选项：

    a. **提供企业管理员凭据**：如果为需要设备写回的林提供企业管理员凭据，Azure AD Connect 将在配置设备写回期间自动准备林。

    b. **下载 PowerShell 脚本**：Azure AD Connect 会自动生成 PowerShell 脚本，该脚本可以为设备写回准备 Active Directory。 如果无法在 Azure AD Connect 中提供企业管理员凭据，则建议下载 PowerShell 脚本。 将下载的 PowerShell 脚本 **CreateDeviceContainer.psq** 提供给设备将写回到的林的企业管理员。
    ![准备 Active Directory 林](./media/how-to-connect-device-writeback/devicecontainercreds.png)
    
    执行以下操作来准备 Active Directory 林：
    * 该命令会在 CN=Device Registration Configuration,CN=Services,CN=Configuration,[forest-dn] 下创建并配置新的容器和对象（如果不存在）。
    * 该命令会在 CN=RegisteredDevices,[domain-dn] 下创建并配置新的容器和对象（如果不存在）。 将在此容器中创建设备对象。
    * 在 Azure AD 连接器帐户中设置必要的权限，以便管理 Active Directory 上的设备。
    * 即使 Azure AD Connect 安装在多个林中，也只需要在一个林中运行。

## <a name="verify-devices-are-synchronized-to-active-directory"></a>验证设备是否已同步到 Active Directory
设备写回现在应在正常运行。 请注意，将设备对象写回到 Active Directory 最长可能需要 3 个小时。  若要验证设备是否已正确同步，请在同步规则完成之后执行以下操作：

1. 启动 Active Directory 管理中心。
2. 在要联合的域中展开 RegisteredDevices。

   ![Active Directory 管理中心 - 已注册的设备](./media/how-to-connect-device-writeback/devicewriteback5.png)

3. 其中会列出当前已注册的设备。

   ![Active Directory 管理中心 - 已注册的设备列表](./media/how-to-connect-device-writeback/devicewriteback6.png)

## <a name="enable-conditional-access"></a>启用条件性访问
[使用 Azure Active Directory Device Registration 设置本地条件性访问](../../active-directory/active-directory-device-registration-on-premises-setup.md)中提供了有关启用此方案的详细说明。

## <a name="troubleshooting"></a>故障排除
### <a name="the-writeback-checkbox-is-still-disabled"></a>写回复选框仍处于禁用状态
如果未启用设备写回复选框，即使已遵循上述步骤，以下步骤还是会在启用此框之前引导完成安装向导正在验证的程序。

首先：

* 设备所在的林必须将林架构升级到 Windows 2012 R2 级别，以便显示设备对象和相关属性。
* 如果安装向导已在运行，则不会检测到任何更改。 在此情况下，请先完成安装向导，再试一次。
* 确保在初始化脚本中提供的帐户是 Active Directory 连接器实际使用的正确用户。 若要验证，请执行以下步骤：
  * 从“开始”菜单打开“同步服务”。
  * 打开“连接器”选项卡。
  * 查找类型为 Active Directory 域服务的连接器并选择它。
  * 在“操作”下面，选择“属性”。
  * 转到“连接到 Active Directory 林”。 检查此屏幕上指定的域和用户名是否与提供给脚本的帐户匹配。
    ![同步服务管理器中的连接器帐户](./media/how-to-connect-device-writeback/connectoraccount.png)

在 Active Directory 中验证配置：

* 检查设备注册服务是否位于配置命名上下文下面的以下位置：(CN=DeviceRegistrationService,CN=Device Registration Services,CN=Device Registration Configuration,CN=Services,CN=Configuration)。

![故障排除，配置命名空间中的 DeviceRegistrationService](./media/how-to-connect-device-writeback/troubleshoot1.png)

* 搜索配置命名空间以验证是否只有一个配置对象。 如果存在多个对象，请删除重复项。

![排除故障，搜索重复的对象](./media/how-to-connect-device-writeback/troubleshoot2.png)

* 对于设备注册服务对象，请确保 msDS-DeviceLocation 属性存在且具有值。 查找此位置，并确保它存在且 objectType 为 msDS-DeviceContainer。

![故障排除，msDS-DeviceLocation](./media/how-to-connect-device-writeback/troubleshoot3.png)

![故障排除，RegisteredDevices 对象类](./media/how-to-connect-device-writeback/troubleshoot4.png)

* 验证 Active Directory 连接器使用的帐户是否具有上一步骤所找到的“注册的设备”容器的所需权限。 这是此容器上所需的权限：

![故障排除，验证对容器的权限](./media/how-to-connect-device-writeback/troubleshoot5.png)

* 验证 Active Directory 帐户是否具有对 CN=Device Registration Configuration,CN=Services,CN=Configuration 对象的权限。

![排除故障，验证对设备注册配置的权限](./media/how-to-connect-device-writeback/troubleshoot6.png)

## <a name="additional-information"></a>其他信息
* [使用条件性访问管理风险](../active-directory-conditional-access-azure-portal.md)
* [使用 Azure Active Directory Device Registration 设置本地条件性访问](../../active-directory/active-directory-device-registration-on-premises-setup.md)

## <a name="next-steps"></a>后续步骤
了解有关 [将本地标识与 Azure Active Directory 集成](whatis-hybrid-identity.md)的详细信息。

