---
title: 应用注册门户帮助主题 | Microsoft Docs
description: 介绍 Microsoft 应用注册门户中的各种功能。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: f0507c28-9464-4d3e-bd53-de9053fd5278
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/28/2018
ms.author: celested
ms.reviewer: lenalepa
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9b77f2a403bd4f410665d00bc69b3b1bcf0c3aaa
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65139179"
---
# <a name="app-registration-reference"></a>应用注册参考
本文档提供可在[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/)中找到的各种功能的上下文和说明。

> [!NOTE]
> 我们将不再支持注册和管理中的聚合与 Azure AD 应用程序[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/)启动 2019 年 5。 我们建议你管理的现有应用程序，并使用注册新的应用程序[应用注册](https://aka.ms/appregistrations)体验在 Azure 门户中。

## <a name="my-applications-or-converged-applications"></a>我的应用程序或聚合应用程序
此列表包含所有已注册且可与 Azure AD v2.0 终结点搭配使用的应用程序。 这些应用程序能够让用户使用个人 Microsoft 帐户和工作/学校帐户从 Azure Active Directory 登录。 若要了解有关 Azure AD v2.0 终结点的详细信息，请参阅 [v2.0 概述](active-directory-appmodel-v2-overview.md)。 这些应用程序也可以用来与 Microsoft 帐户身份验证终结点 `https://login.live.com` 集成。

## <a name="azure-ad-only-applications"></a>Azure AD 专用应用程序
此列表包含所有已注册且可与 Azure AD v1.0 终结点搭配使用的应用程序。 这些应用程序只能让用户使用工作/学校帐户从 Azure Active Directory 登录。 此列表包括使用 [Azure 门户](https://portal.azure.com)中的**应用注册**体验注册的应用程序。

## <a name="live-sdk-applications"></a>Live SDK 应用程序
此列表包含所有已注册且只能与 Microsoft 帐户搭配使用的应用程序。 它们不能与 Azure Active Directory 搭配使用。 可以在此处找到之前已向 MSA 开发人员门户 (`https://account.live.com/developers/applications`) 注册的所有应用程序。 之前曾在 `https://account.live.com/developers/applications` 执行的所有功能现在均可在此新门户 `https://apps.dev.microsoft.com` 中执行。

## <a name="application-secrets"></a>应用程序密码
应用程序密码是允许应用程序利用 Azure AD 执行可靠的[客户端身份验证](https://tools.ietf.org/html/rfc6749#section-2.3)的凭据。 在 OAuth 和 OpenID Connect 中，应用程序密码通常称为 `client_secret`。 在 v2.0 协议中，任何在 Web 可寻址位置（使用 `https` 方案）接收安全令牌的应用程序都必须使用应用程序密码，通过兑换该安全令牌向 Azure AD 标识自己。 此外，设备上接收令牌的任何本机客户端都将禁止使用应用程序密码来执行客户端身份验证。 这样可防止在不安全的环境中存储密码。

每个应用都能在任何指定时间包含两个有效的应用程序密码。 通过维护两个密码，就能够在应用程序的整个环境中执行定期的密钥滚动更新。 将应用程序的全部内容移迁移到新密码之后，可能会删除旧密码，并预配一个新密码。

目前，应用注册门户中只允许两种类型的应用程序密码。 如果选择“生成新密码”，将生成一个共享密码并存储在各自的数据存储中，可以在应用程序中使用该密码。 如果选择“生成新密钥对”，将创建一个新的公钥/私钥对，可下载并用来向 Azure AD 进行客户端身份验证。 如果选择“上载公钥”，则可以使用自己的公钥对/私钥对。
你需要上载包含公钥的证书。

## <a name="profile"></a>配置文件
应用注册门户的配置文件部分可以用来自定义应用程序的登录页面。 目前，可以更改登录页面的应用程序徽标、服务条款 URL 和隐私声明 URL。 徽标必须是 GIF、PNG 或 JPEG 文件中透明的 48 x 48 或 50 x 50 像素图像，且大小不超过 15 KB。 请尝试更改这些值，并查看所生成的登录页面！

## <a name="live-sdk-support"></a>Live SDK 支持
启用“Live SDK 支持”后，系统会将创建的所有应用程序密码预配到 Azure AD 和 Microsoft 帐户数据存储中。 这让应用程序能够直接与 Microsoft 帐户服务 (login.live.com) 集成。 如果要直接使用 Microsoft 帐户（而不是使用 Azure AD v2.0 终结点）生成应用，则应确保启用 Live SDK 支持。

如果禁用 Live SDK 支持，将确保系统只会将应用程序密码写入 Azure AD 数据存储。 Azure AD 数据存储包含企业级法规，使其遵守特定的标准，例如遵守 FISMA。 如果启用 Live SDK 支持，应用程序可能不会遵守其中某些标准。

如果只打算使用 Azure AD v2.0 终结点，则可以安全地禁用 Live SDK 支持。

