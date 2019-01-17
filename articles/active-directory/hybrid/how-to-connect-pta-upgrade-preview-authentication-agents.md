---
title: Azure AD Connect - 直通身份验证 - 升级身份验证代理 | Microsoft Docs
description: 本文介绍如何升级 Azure Active Directory (Azure AD) 直通身份验证配置。
services: active-directory
keywords: Azure AD Connect 直通身份验证, 安装 Active Directory, Azure AD 所需的组件, SSO, 单一登录
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: ba5c3704d89f75617683ae886da1857d51251631
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306286"
---
# <a name="azure-active-directory-pass-through-authentication-upgrade-preview-authentication-agents"></a>Azure Active Directory 直通身份验证：升级预览身份验证代理

## <a name="overview"></a>概述

本文面向通过预览版使用 Azure AD 直通身份验证的客户。 我们最近升级了身份验证代理软件（并重新设计了品牌）。 需要 _手动_ 升级本地服务器上安装的预览身份验证代理。 此手动升级是一次性的操作。 将来对身份验证代理的所有更新都会自动应用。 升级的原因如下：

- 身份验证代理预览版不会进一步接收任何安全修复或 bug 修复。
-   为了保持高可用性，无法在其他服务器上安装身份验证代理预览版。

## <a name="check-versions-of-your-authentication-agents"></a>检查身份验证代理的版本

### <a name="step-1-check-where-your-authentication-agents-are-installed"></a>步骤 1：检查身份验证代理的安装位置

遵循以下步骤检查身份验证代理的安装位置：

1. 使用租户的全局管理员凭据登录到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 在左侧导航栏中，选择“Azure Active Directory”。
3. 选择“Azure AD Connect”。 
4. 选择“直通身份验证”。 此边栏选项卡列出了安装身份验证代理的服务器。

![Azure Active Directory 管理中心 - 传递身份验证边栏选项卡](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta8.png)

### <a name="step-2-check-the-versions-of-your-authentication-agents"></a>步骤 2：检查身份验证代理的版本

若要检查上一步骤中识别的每台服务器上的身份验证代理版本，请遵照以下说明操作：

1. 在本地服务器上转到“控制面板”->“程序”->“程序和功能”。
2. 如果“Microsoft Azure AD Connect 身份验证代理”存在对应的条目，则不需要在此服务器上执行任何操作。
3. 如果“Microsoft Azure AD 应用程序代理连接器”存在对应的条目，则需要在此服务器上手动升级。

![身份验证代理预览版](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta6.png)

## <a name="best-practices-to-follow-before-starting-the-upgrade"></a>开始升级之前要遵循的最佳做法

在升级之前，请确保已准备好以下各项：

1. **创建仅限云的全局管理员帐户**：如果没有仅限云的全局管理员帐户（在直通身份验证代理无法正常工作的紧急情况下使用），请不要升级。 了解如何[添加仅限云的全局管理员帐户](../active-directory-users-create-azure-portal.md)。 此步骤至关重要，可确保你不被锁定在租户外部。
2.  **确保高可用性**：遵照这些[说明](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability)安装另一个独立的身份验证代理，以便为登录请求提供高可用性（如果以前未这样做）。

## <a name="upgrading-the-authentication-agent-on-your-azure-ad-connect-server"></a>在 Azure AD Connect 服务器上升级身份验证代理

在同一台服务器上升级身份验证代理之前，需要升级 Azure AD Connect。 在主要和过渡 Azure AD Connect 服务器上遵循以下步骤：

1. **升级 Azure AD Connect**：遵循[此文](how-to-upgrade-previous-version.md)升级到最新版本的 Azure AD Connect。
2. **卸载身份验证代理预览版**：下载[此 PowerShell 脚本](https://aka.ms/rmpreviewagent)，并在服务器上以管理员身份运行该脚本。
3. **下载最新版身份验证代理（版本 1.5.389.0 或更高版本）**：使用租户的全局管理员凭据登录到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 选择“Azure Active Directory”->“Azure AD Connect”->“直通身份验证”->“下载代理”。 接受[服务条款](https://aka.ms/authagenteula)并下载最新版本的身份验证代理。 也可以从[此处](https://aka.ms/getauthagent)下载身份验证代理。
4. **安装最新版本的身份验证代理**：运行步骤 3 中下载的可执行文件。 出现提示时，提供租户的全局管理员凭据。
5. **验证是否已安装最新版本**：如前所述转到“控制面板”->“程序”->“程序和功能”，检查“Microsoft Azure AD Connect 身份验证代理”是否存在对应的条目。

>[!NOTE]
>如果在完成上述步骤后查看 [Azure Active Directory 管理中心](https://aad.portal.azure.com)的“传递身份验证”边栏选项卡，将看到每个服务器上有两个身份验证代理条目，一个条目显示身份验证代理处于“活动”状态，另一个显示它处于“非活动”状态。 这是 _正常情况_ 。 几天后自动删除表示“非活动”的条目。

## <a name="upgrading-the-authentication-agent-on-other-servers"></a>升级其他服务器上的身份验证代理

遵循以下步骤升级其他服务器（未安装 Azure AD Connect）上的身份验证代理：

1. **卸载身份验证代理预览版**：下载[此 PowerShell 脚本](https://aka.ms/rmpreviewagent)，并在服务器上以管理员身份运行该脚本。
2. **下载最新版身份验证代理（版本 1.5.389.0 或更高版本）**：使用租户的全局管理员凭据登录到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 选择“Azure Active Directory”->“Azure AD Connect”->“直通身份验证”->“下载代理”。 接受服务条款并下载最新版本。
3. **安装最新版本的身份验证代理**：运行步骤 2 中下载的可执行文件。 出现提示时，提供租户的全局管理员凭据。
4. **验证是否已安装最新版本**：如前所述转到“控制面板”->“程序”->“程序和功能”，检查是否存在名为“Microsoft Azure AD Connect 身份验证代理”的条目。

>[!NOTE]
>如果在完成上述步骤后查看 [Azure Active Directory 管理中心](https://aad.portal.azure.com)的“传递身份验证”边栏选项卡，将看到每个服务器上有两个身份验证代理条目，一个条目显示身份验证代理处于“活动”状态，另一个显示它处于“非活动”状态。 这是 _正常情况_ 。 几天后自动删除表示“非活动”的条目。

## <a name="next-steps"></a>后续步骤
- [故障排除](tshoot-connect-pass-through-authentication.md) - 了解如何解决使用此功能时遇到的常见问题。
