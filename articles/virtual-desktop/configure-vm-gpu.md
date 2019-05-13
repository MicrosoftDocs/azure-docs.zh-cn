---
title: 配置 GPU for Windows 虚拟桌面预览版-Azure
description: 如何启用 GPU 加速渲染和 Windows 虚拟桌面预览中的编码。
services: virtual-desktop
author: gundarev
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/06/2019
ms.author: denisgun
ms.openlocfilehash: a6a67c89253a1b16f9266d7917655d1b1104022e
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159565"
---
# <a name="configure-graphics-processing-unit-gpu-acceleration-for-windows-virtual-desktop-preview"></a>配置图形处理单元 (GPU) 加速 Windows 虚拟桌面预览

Windows 虚拟桌面预览支持 GPU 加速呈现和编码为改进后的应用性能和可伸缩性。 GPU 加速的图形密集型应用程序特别至关重要。

按照在本文中创建优化的 GPU 的 Azure 虚拟机、 将其添加到你主机的池，并将其配置为使用 GPU 加速的呈现和编码的说明。 本文假设已配置的 Windows 虚拟桌面租户。

## <a name="select-a-gpu-optimized-azure-virtual-machine-size"></a>选择 GPU 优化 Azure 虚拟机大小

Azure 提供了多种[GPU 优化虚拟机大小](/azure/virtual-machines/windows/sizes-gpu)。 主机池的正确的选择取决于多种因素，包括您的特定应用程序工作负荷、 所需的用户体验和成本的质量。 一般情况下，更大、 功能更强大的 Gpu 提供给定的用户密度更好的用户体验。

## <a name="create-a-host-pool-provision-your-virtual-machine-and-configure-an-app-group"></a>创建一个主机池、 设置虚拟机，并配置应用组

创建新的主机池使用的虚拟机选择的大小。 有关说明，请参阅[教程：使用 Azure Marketplace 创建主机池](/azure/virtual-desktop/create-host-pools-azure-marketplace)。

Windows 虚拟桌面预览支持 GPU 加速呈现及以下操作系统中的编码：

* Windows 10 版本 1511 或更高版本
* Windows Server 2016 或更高版本

此外必须配置应用组，或使用默认的桌面应用程序组 （名为"桌面应用程序组"） 时创建新的主机池自动创建的。 有关说明，请参阅[教程：管理 Windows 虚拟桌面预览的应用程序组](/azure/virtual-desktop/manage-app-groups)。

>[!NOTE]
>Windows 虚拟桌面预览版仅支持对启用了 GPU 的主机池的"桌面"应用组类型。 已启用 GPU 的主机池不支持的类型"RemoteApp"应用组。

## <a name="install-supported-graphics-drivers-in-your-virtual-machine"></a>虚拟机中安装受支持的图形驱动程序

若要充分利用 Azure N 系列 Vm 的 GPU 功能的 Windows 虚拟桌面预览中，必须安装 NVIDIA 图形驱动程序。 按照的说明[上运行 Windows 的 N 系列 Vm 安装 NVIDIA GPU 驱动程序](/azure/virtual-machines/windows/n-series-driver-setup)手动安装驱动程序，或使用[NVIDIA GPU 驱动程序扩展](/azure/virtual-machines/extensions/hpccompute-gpu-windows)。

请注意，只能[NVIDIA GRID 驱动程序](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-grid-drivers)分发的 Azure Windows 虚拟桌面预览支持。

驱动程序安装后，VM 重新启动是必需的。 使用上面说明中的验证步骤确认已成功安装图形驱动程序。

## <a name="configure-gpu-accelerated-app-rendering"></a>配置 GPU 加速应用呈现

默认情况下，应用程序和多会话配置中运行的桌面，cpu 呈现，而不使用可用 Gpu 用于呈现。 配置会话主机启用 GPU 加速的呈现的组策略：

1. 连接到使用具有本地管理员权限的帐户的 VM 的桌面。
2. 打开开始菜单，键入"gpedit.msc"，打开组策略编辑器。
3. 导航到树**计算机配置** > **管理模板** > **Windows 组件** >  **远程桌面服务** > **远程桌面会话主机** > **远程会话环境**。
4. 选择策略**的所有远程桌面服务会话使用硬件默认图形适配器**并将此策略设置为**已启用**若要启用远程会话中的 GPU 呈现。

## <a name="configure-gpu-accelerated-frame-encoding"></a>配置 GPU 加速帧编码

远程桌面将呈现的应用和桌面 （无论是使用 GPU 或 CPU 呈现） 的所有图形都编码以便传输给远程桌面客户端。 默认情况下，远程桌面不会利用此编码可用 Gpu。 配置组策略为启用 GPU 加速帧编码会话主机。 继续上面的步骤：

1. 选择策略**的远程桌面连接设置优先级 H.264/AVC 444 图形模式**并将此策略设置为**已启用**若要在远程会话中强制 H.264/AVC 444 编解码器。
2. 选择策略**的远程桌面连接的配置 H.264/AVC 硬件编码**并将此策略设置为**已启用**若要启用硬件的 AVC/H.264 编码在远程会话中。

    >[!NOTE]
    >在 Windows Server 2016 中，设置选项**喜欢 AVC 硬件编码**到**始终尝试**。

3. 现在，编辑组策略强制组策略更新。 打开命令提示符并键入：

    ```batch
    gpupdate.exe /force
    ```

4. 从远程桌面会话注销。

## <a name="verify-gpu-accelerated-app-rendering"></a>验证 GPU 加速应用呈现

若要验证，应用程序使用的 GPU 呈现，请尝试以下某项操作：

* 使用`nvidia-smi`实用程序中所述[验证驱动程序安装](/azure/virtual-machines/windows/n-series-driver-setup#verify-driver-installation)运行您的应用程序时检查 GPU 利用率。
* 在受支持的操作系统版本上，您可以使用任务管理器检查 GPU 利用率。 若要查看应用是否利用了 GPU 的"性能"选项卡中选择 GPU。

## <a name="verify-gpu-accelerated-frame-encoding"></a>验证 GPU 加速帧编码

若要验证远程桌面使用 GPU 加速编码：

1. 连接到的 VM 使用 Windows 虚拟桌面客户端桌面。
2. 启动事件查看器并导航到以下节点：**应用程序和服务日志** > **Microsoft** > **Windows** > **RemoteDesktopServices RdpCoreTS**  > **操作**
3. 若要确定是否使用 GPU 加速编码，查找事件 ID 170。 如果你看到"已启用的 AVC 硬件编码器：使用 1 – 然后 GPU 编码。
4. 若要确定是否使用 AVC 444 模式下，查找事件 ID 162。 如果你看到"AVC 可用：1 个初始配置文件：使用 2048"然后 AVC 444。

## <a name="next-steps"></a>后续步骤

这些说明应具有你启动并正在运行并在单个会话主机 VM 上的 GPU 加速。 启用 GPU 加速在更大的主机池之间的一些其他注意事项：

* 请考虑使用[NVIDIA GPU 驱动程序扩展](/azure/virtual-machines/extensions/hpccompute-gpu-windows)简化跨许多 Vm 的驱动程序安装和更新。
* 请考虑使用 Active Directory 组策略来简化在多个 Vm 间的组策略配置。 有关部署 Active Directory 域中的组策略的信息，请参阅[使用组策略对象](https://go.microsoft.com/fwlink/p/?LinkId=620889)。
