---
title: Windows 虚拟桌面预览环境-Azure
description: Windows 虚拟桌面预览环境的基本元素。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: helohr
ms.openlocfilehash: 6aa6c7326759e480235df5fe9d4b0878cd11024d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65142388"
---
# <a name="windows-virtual-desktop-preview-environment"></a>Windows 虚拟桌面预览环境

Windows 虚拟桌面预览版是一种服务，使用户可以对其虚拟化的桌面和 RemoteApps 轻松、 安全访问。 本主题将介绍更多 Windows 虚拟桌面环境的常规结构。

## <a name="tenants"></a>租户

Windows 虚拟桌面租户是用于管理 Windows 虚拟桌面环境的主要接口。 每个 Windows 虚拟桌面租户必须与 Azure Active Directory 包含的用户将登录到该环境相关联。 从 Windows 虚拟桌面租户，可以开始创建主机池运行用户的工作负荷。

## <a name="host-pools"></a>主机池

主机池是注册到 Windows 虚拟桌面会话主机运行 Windows 虚拟桌面代理时的 Azure 虚拟机的集合。 池中主机的所有会话主机虚拟机应都源自同一图像可查看一致的用户体验。

主机池可以是两种类型之一：

- 个人，其中每个会话主机分配给单个用户。
- 共用，会话主机接受来自主机池内的应用程序组来授予任何用户连接的位置。

若要更改其负载平衡行为，多少会话可能需要每个会话主机，主机池和用户可以对会话主机的主机池时登录到其 Windows 虚拟桌面会话中执行哪些操作，可以设置其他属性。 控制用户可通过应用组已发布的资源。

## <a name="app-groups"></a>应用组

应用组是在会话中的主机池的主机上安装的应用程序的逻辑分组。 应用组可以是两种类型之一：

- RemoteApp，用户可以访问 RemoteApps 分别选择并将发布到应用程序组
- 桌面，用户可以访问完整的桌面

默认情况下，当创建主机池，将自动创建 （名为"桌面应用程序组"） 的桌面应用程序组。 可以在任何时候删除此应用程序组。 但是，不能在主机池中创建另一个桌面应用程序组，而存在的桌面应用程序组。 若要发布 RemoteApps，必须创建 RemoteApp 应用组。 可以创建多个 RemoteApp 应用组，以适应不同的工作情况。 不同的 RemoteApp 应用程序组还可以包含重叠的 RemoteApps。

若要向用户发布资源，必须将它们分配到应用程序组中。 在将用户分配到应用组，请考虑以下操作：

- 不能为用户分配到桌面应用程序组和 RemoteApp 应用组在同一主机池中。
- 可以将用户分配到多个应用组在同一主机池中，并且其源将这两个应用程序组的累积。

## <a name="tenant-groups"></a>租户组

在 Windows 虚拟桌面，Windows 虚拟桌面租户是大部分的安装和配置发生的位置。 Windows 虚拟桌面租户包含主机池、 应用组和应用组用户分配。 但是，可能有某些情况下，您需要管理 Windows 虚拟桌面的多个租户，尤其是如果你是云服务提供商 (CSP) 或托管合作伙伴。 在这些情况下，可以使用自定义 Windows 虚拟桌面租户组放置每个客户的 Windows 虚拟桌面租户并集中管理访问权限。 但是，如果您仅管理单个 Windows 虚拟桌面租户，租户组概念不适用，并且可以继续操作和管理你的租户默认租户组中存在的。

## <a name="end-users"></a>最终用户

用户分配到其应用组后，它们可以连接到在 Windows 虚拟桌面部署中使用的 Windows 虚拟桌面客户端。

## <a name="next-steps"></a>后续步骤

了解有关委派访问权限以及如何将角色分配给用户的详细信息[中 Windows 虚拟桌面预览委派访问](delegated-access-virtual-desktop.md)。

若要了解如何设置 Windows 虚拟桌面租户，请参阅[在 Windows 虚拟桌面预览中创建一个租户](tenant-setup-azure-active-directory.md)。

若要了解如何连接到 Windows 虚拟桌面，请参阅以下文章之一：

- [从 Windows 10 或 Windows 7 连接](connect-windows-7-and-10.md)
- [从 web 浏览器连接](connect-web.md)
