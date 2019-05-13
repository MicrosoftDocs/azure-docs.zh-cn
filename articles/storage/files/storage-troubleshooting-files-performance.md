---
title: Azure 文件性能故障排除指南
description: 已知 Azure 高级文件共享 （预览版） 和关联的解决方法的性能问题。
services: storage
author: gunjanj
ms.service: storage
ms.topic: article
ms.date: 04/25/2019
ms.author: gunjanj
ms.subservice: files
ms.openlocfilehash: 5ae0bb736a7cc0bbc38df5905abc5d8a71f60eb9
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190054"
---
# <a name="troubleshoot-azure-files-performance-issues"></a>排查 Azure 文件性能问题

本文列出了与高级版 Azure 文件共享 （预览版） 相关的一些常见问题。 它提供了可能的原因和解决方法时可能遇到这些问题。

## <a name="high-latency-low-throughput-and-general-performance-issues"></a>高延迟、 低吞吐量，以及常规的性能问题

### <a name="cause-1-share-experiencing-throttling"></a>原因 1：共享遇到限制

在共享上的默认配额为 100 GiB，后者提供 100 基线 IOPS （可能会最多 300 迸发到一小时）。 预配和 IOPS 的关系的详细信息，请参阅[预配共享](storage-files-planning.md#provisioned-shares)计划指南的部分。

若要确认你的共享被阻止，您可以在门户中利用 Azure 度量值。

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 选择**所有服务**，然后搜索**指标**。

1. 选择“指标”。

1. 选择你的存储帐户的资源。

1. 选择**文件**作为度量命名空间。

1. 选择**事务**作为该度量值。

1. 添加的筛选器**ResponseType**并进行检查以查看任何请求是否具有响应代码**SuccessWithThrottling**。

![高级文件共享的指标选项](media/storage-troubleshooting-premium-fileshares/metrics.png)

### <a name="solution"></a>解决方案

- 通过共享上指定较高的配额，增加共享预配的容量。

### <a name="cause-2-metadatanamespace-heavy-workload"></a>原因 2：元数据/命名空间繁重的工作负荷

如果你的请求的大部分都元数据为中心 （如 createfile/openfile/closefile/queryinfo/querydirectory)，则延迟将会进行比较以读/写操作时，更糟。

若要确认是否你的请求的大部分都是元数据为中心，您可以使用与上面相同的步骤。 而不是添加的筛选器除外**ResponseType**，添加的筛选器**API 名称**。

![API 名称为你的度量值中的筛选器](media/storage-troubleshooting-premium-fileshares/MetadataMetrics.png)

### <a name="workaround"></a>解决方法

- 检查是否可以修改应用程序以减少元数据操作的数量。

### <a name="cause-3-single-threaded-application"></a>原因 3：单线程应用程序

如果客户使用的应用程序是单线程，这可能导致低很多 IOPS/吞吐量比基于预配的共享大小的最大可能。

### <a name="solution"></a>解决方案

- 通过增加线程数来提高应用程序并行性。
- 切换到应用程序并行度不可能。 例如，对于复制操作，客户可以使用 AzCopy 或 RoboCopy 从 Windows 客户端或**并行**命令在 Linux 客户端上。

## <a name="very-high-latency-for-requests"></a>请求延迟很高

### <a name="cause"></a>原因

客户端 VM 可以位于不同于高级文件共享的区域。

### <a name="solution"></a>解决方案

- 从高级文件共享所在的同一区域中的 VM 中运行应用程序。

## <a name="client-unable-to-achieve-maximum-throughput-supported-by-the-network"></a>客户端无法实现由网络提供支持的最大吞吐量

一个可能原因是缺少 fo SMB 多通道的支持。 目前，Azure 文件共享只支持单个通道，因此只有一个连接到服务器的客户端 VM。 此单个连接限定于一个核心上的客户端 VM，以便从 VM 可实现的最大吞吐量受单核。

### <a name="workaround"></a>解决方法

- 获取具有更大的核心的 VM，可帮助提高吞吐量。
- 从多个 Vm 运行客户端应用程序会增加吞吐量。
- 在可能的情况，请使用 REST Api。

## <a name="throughput-on-linux-clients-is-significantly-lower-when-compared-to-windows-clients"></a>在 Linux 客户端上的吞吐量将向 Windows 客户端进行比较时大大降低。

### <a name="cause"></a>原因

这是 Linux 上的 SMB 客户端实现的已知的问题。

### <a name="workaround"></a>解决方法

- 负载分布到多个 Vm
- 在同一 VM 上使用多个具有装入点**nosharesock**选项，并传播到这些负载装入点。

## <a name="high-latencies-for-metadata-heavy-workloads-involving-extensive-openclose-operations"></a>元数据涉及大量的打开/关闭操作的大量工作负荷的高延迟。

### <a name="cause"></a>原因

缺少对目录租用的支持。

### <a name="workaround"></a>解决方法

- 如果可能，请在短时间的时间内避免过多地打开/关闭句柄上的相同目录。
- 对于 Linux Vm，通过指定增加目录条目缓存超时**actimeo =<sec>** 作为装载选项。 默认情况下，它是一秒，因此如三或五个更大的值可能会帮助。
- 对于 Linux Vm，升级到 4.20 或更高版本的内核。

## <a name="low-iops-on-centosrhel"></a>在 CentOS/RHEL 的低 IOPS

### <a name="cause"></a>原因

CentOS/RHEL 上不支持 IO 深度大于 1。

### <a name="workaround"></a>解决方法

- 升级到 CentOS 8 / RHEL 8。
- 更改到 Ubuntu。

## <a name="jitterysaw-tooth-pattern-for-iops"></a>IOPS 的抖动/锯齿模式

### <a name="cause"></a>原因

客户端应用程序持续超出基线 IOPS。 目前，没有任何服务端的请求负载进行平滑处理，因此如果客户端超出基线 IOPS，它将会受到限制的服务。 该限制可能会导致客户端遇到抖动/saw tooth IOPS 模式。 在这种情况下，平均 IOPS 来实现客户端可能会低于基线 IOPS。

### <a name="workaround"></a>解决方法

- 从客户端应用程序，减少请求负载，以便共享不会不会受到限制。
- 增加的共享的配额，以便共享不会不会受到限制。

## <a name="excessive-directoryopendirectoryclose-calls"></a>过多的 DirectoryOpen/DirectoryClose 调用

### <a name="cause"></a>原因

如果 DirectoryOpen/DirectoryClose 调用数是顶级 API 调用，并且不希望客户端对此进行的许多调用，它可能会在 Azure 的客户端 VM 上安装防病毒软件出现问题。

### <a name="workaround"></a>解决方法

- 修复此问题现已推出[年 4 月平台更新为 Windows](https://support.microsoft.com/help/4052623/update-for-windows-defender-antimalware-platform)。

## <a name="file-creation-is-slower-than-expected"></a>文件的创建过程是比预期要慢

### <a name="cause"></a>原因

依赖于创建大量的文件的工作负荷不会看到性能的高级文件共享和标准文件共享之间的重大差异。

### <a name="workaround"></a>解决方法

- 无。

## <a name="slow-performance-from-windows-81-or-server-2012-r2"></a>从 Windows 8.1 或 Server 2012 R2 性能缓慢

### <a name="cause"></a>原因

高于预期的延迟访问 Azure 文件的 IO 密集型工作负荷。

### <a name="workaround"></a>解决方法

- 安装可用[修补程序](https://support.microsoft.com/help/3114025/slow-performance-when-you-access-azure-files-storage-from-windows-8-1)。