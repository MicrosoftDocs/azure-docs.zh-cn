---
title: 从 Windows 10 或 Windows 7-Azure 连接到 Windows 虚拟桌面预览
description: 如何从 Windows 10 或 Windows 7 连接到 Windows 虚拟桌面预览。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 04/24/2019
ms.author: helohr
ms.openlocfilehash: b7d7b25d0355f2379b90313f17e2b595234df827
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145988"
---
# <a name="connect-from-windows-10-or-windows-7"></a>从 Windows 10 或 Windows 7 连接

> 适用对象：Windows 7 和 Windows 10。

提供可下载的客户端，提供对访问的 Windows 虚拟桌面预览资源从运行 Windows 7 和 Windows 10 的设备。

> [!IMPORTANT]
> 不要使用**RemoteApp 和桌面连接 (RADC)** 或**远程桌面连接 (MSTSC)** 访问 Windows 虚拟机的资源，因为 Windows 虚拟桌面不支持客户端。

## <a name="install-the-client"></a>安装客户端

[下载](https://go.microsoft.com/fwlink/?linkid=2068602) 和客户端安装到本地 PC。 安装需要管理员权限。

## <a name="subscribe-to-a-feed"></a>订阅源

通过订阅到源提供的管理员联系。 您获取可用的托管资源的列表订阅使资源可在本地 PC 上。

若要订阅源：

1. 启动客户端从所有应用程序列表中，查找有关**远程桌面**。
1. 选择**Subscribe**在主页上，若要连接到该服务并检索你的资源。
1. **登录**与您的用户帐户时提示。

成功身份验证后，现在应看到可用资源的列表。

可以通过两种方法之一启动的资源。

- 从客户端的主页上，双击要启动它的资源。
- 启动资源，如往常一样从开始菜单上的其他应用。
  - 您还可以搜索在搜索栏中的应用。

一旦订阅源，该源的内容定期自动更新。 资源可能会添加、 更改或删除由您的管理员所做的更改。

## <a name="view-the-details-of-a-feed"></a>查看源的详细信息

订阅之后, 可以通过访问详细信息面板中查看有关源的其他信息。

1. 从客户端的主页上，选择省略号 (**...**) 右侧的源的名称。
1. 从下拉列表菜单中选择**详细信息**。
1. 详细信息面板将显示在右侧的客户端。

详细信息面板包含有关源的有用信息：

- URL 和用于订阅的用户名
- 应用和桌面的数量
- 上次更新日期/时间
- 最后一个更新的状态

如果需要您可以通过选择上启动手动更新**立即更新**。

## <a name="unsubscribe-from-a-feed"></a>取消订阅源

本部分将介绍如何取消订阅源。 您可以随时取消订阅，以再次订阅与不同的帐户或从系统中删除你的资源。

1. 从客户端的主页上，选择省略号 (**...**) 右侧的源的名称。
1. 从下拉列表菜单中选择**Unsubscribe**。
1. 查看并选择**继续**从对话框。

## <a name="update-the-client"></a>更新客户端

可用新版本的客户端时，将客户端和 Windows 操作中心通知您。 选择通知以开始更新过程。
