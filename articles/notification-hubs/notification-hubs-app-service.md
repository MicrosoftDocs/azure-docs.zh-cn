---
title: 与应用服务移动应用集成
description: 了解如何将 Azure 通知中心与 Azure 应用服务移动应用配合使用。
author: jwargo
manager: patniko
editor: spelluru
services: notification-hubs
documentationcenter: ''
ms.assetid: 83132dff-a01d-4b31-a426-b57496852b81
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: multiple
ms.devlang: multiple
ms.topic: overview
ms.custom: mvc
ms.date: 01/04/2019
ms.author: jowargo
ms.openlocfilehash: bb39949bc21ece819b7ac6e279390f8bdb0ee00a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65157445"
---
# <a name="integration-with-app-service-mobile-apps"></a>与应用服务移动应用集成

> [!NOTE]
> Microsoft 致力于为 Azure 应用服务移动应用提供完全支持，包括为最新的 OS 版本、Bug 修复、文档改进以及社区 PR 评审提供支持。 请注意，产品团队目前并未准备开发 Azure 移动应用的任何新功能。 对于社区在 Azure 移动应用的所有领域所做的贡献，我们深表感激。

为了在 Azure 服务之间促成完美且统一的体验， [应用服务移动应用](../app-service-mobile/app-service-mobile-value-prop.md) 原生支持使用通知中心推送通知。 [应用服务 Mobile Apps](../app-service-mobile/app-service-mobile-value-prop.md) 提供面向企业开发人员和系统集成商的高度可缩放、全局可用的移动应用程序平台，该平台向移动开发人员提供一组丰富的功能。

移动应用开发人员可以借助以下工作流来利用通知中心：

1. 检索设备 PNS 句柄
2. 通过便利的移动应用客户端 SDK 注册 API 将设备注册到通知中心

    > [!NOTE]
    > 请注意，出于安全方面的考虑，移动应用会在注册中去除所有标记。 直接从后端使用通知中心将标记与设备相关联。

3. 从应用后端使用通知中心发送通知

以下是这种集成为开发人员带来的便利：

- **移动应用客户端 SDK**： 这些多平台 SDK 提供简单的 API 用于注册，并自动与链接到移动应用的通知中心联系。 开发人员不需要深入了解通知中心凭据和使用其他服务。
  - 推送到用户：SDK 将使用移动应用的已经过身份验证的用户 ID 来自动标记给定设备，以实现推送到用户的方案。
  - 推送到设备：SDK 自动使用移动应用安装 ID 作为 GUID 来注册到通知中心，省去了开发人员维护多个服务 GUID 的麻烦。
- **安装模型**：移动应用使用通知中心的最新推送模型来呈现 JSON 安装中所有与设备关联的推送属性，该模型与推送通知密切合作且易于使用。
- **灵活性**： 即使是就地集成的，开发人员也始终可以选择直接使用通知中心。
- **[Azure 门户](https://portal.azure.com)中的集成体验**：移动应用以可视化方式呈现推送功能，开发人员可以通过移动应用轻松使用关联的通知中心。
