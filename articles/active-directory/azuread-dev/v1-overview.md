---
title: 适用于开发人员的 Azure Active Directory (v1.0) 概述
description: 本文概述了如何使用 Azure Active Directory v1.0 终结点和平台登录 Microsoft 工作和学校帐户。
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: overview
ms.workload: identity
ms.date: 10/24/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 8cb426239fe9f3dea207df06b6f2bafbdb54058a
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2020
ms.locfileid: "88117662"
---
# <a name="azure-active-directory-for-developers-v10-overview"></a>适用于开发人员的 Azure Active Directory (v1.0) 概述

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Azure Active Directory (Azure AD) 是一个云标识服务，开发人员可以使用它来生成应用，让用户使用 Microsoft 工作或学校帐户安全登录。 Azure AD 支持开发人员生成单租户业务线 (LOB) 应用和多租户应用。 除了基本登录以外，Azure AD 还可以让应用调用 [Microsoft Graph](/graph/overview) 等 Microsoft API，以及在 Azure AD 平台上生成的自定义 API。 本文档介绍了如何使用行业标准协议（例如 OAuth2.0 与 OpenID Connect）向应用程序添加 Azure AD 支持。

> [!NOTE]
> 本页面中的大部分内容侧重于仅支持 Microsoft 工作或学校帐户的 v1.0 终结点和平台。 若要将使用者或个人 Microsoft 帐户登录，请参阅有关 [v2.0 终结点和平台](../develop/v2-overview.md)的信息。 v2.0 终结点为想要登录所有 Microsoft 标识的应用提供统一的开发人员体验。

- [身份验证基础知识](v1-authentication-scenarios.md) 关于如何使用 Azure AD 进行身份验证的简介。
- [应用程序的类型](app-types.md) Azure AD 支持的身份验证方案的概述。

## <a name="get-started"></a>入门

v1.0 快速入门和教程将逐步讲解如何使用 Azure AD 身份验证库 (ADAL) SDK 在偏好的平台上生成应用。 若要开始使用，请参阅 [Microsoft 标识平台（面向开发人员的 Azure Active Directory）](index.yml)中的 **v1.0 快速入门**和 **v1.0 教程**。

## <a name="how-to-guides"></a>操作指南

有关 Azure AD 中最常见任务的详细信息和演练，请参阅 **v1.0 操作指南**。

## <a name="reference-topics"></a>参考主题

以下文章详细介绍了在 Azure AD 中使用的 API、协议消息和术语。

- [身份验证库 (ADAL)](active-directory-authentication-libraries.md) Azure AD 提供的库和 SDK 的概述。
- [代码示例](sample-v1-code.md) 所有 Azure AD 代码示例的列表。
- [术语表](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) 本文档通篇使用的术语和单词定义。

## <a name="videos"></a>视频

请参阅 [Azure Active Directory 开发人员平台视频](videos.md)，获取有关迁移到新的 Microsoft 标识平台的帮助。

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]