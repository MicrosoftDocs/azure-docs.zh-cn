---
title: 将 VM 迁移到 Azure 高级存储 | Microsoft Docs
description: 将现有的 VM 迁移到 Azure 高级存储。 高级存储为 Azure 虚拟机上运行的 I/O 密集型工作负载提供高性能、低延迟的磁盘支持。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 06/27/2017
ms.author: rogarana
ms.reviewer: yuemlu
ms.subservice: common
ms.openlocfilehash: 5cfb96bd3115c8f3116a28926e93df89dff54351
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153766"
---
# <a name="migrating-to-azure-premium-storage-unmanaged-disks"></a>迁移到 Azure 高级存储（非托管磁盘）

> [!NOTE]
> 本文介绍如何将使用非托管标准磁盘的 VM 迁移到使用非托管高级磁盘的 VM。 建议用户对新的 VM 使用 Azure 托管磁盘，并将以前的非托管磁盘转换为托管磁盘。 托管磁盘会处理基础存储帐户，因此无需手动操作。 有关详细信息，请参阅[托管磁盘概述](../../virtual-machines/windows/managed-disks-overview.md)。
>

Azure 高级存储为运行 I/O 密集型工作负荷的虚拟机提供高性能、低延迟的磁盘支持。 可以将应用程序的 VM 磁盘迁移到 Azure 高级存储，以充分利用这些磁盘的速度和性能。

本指南旨在帮助 Azure 高级存储的新用户更好地准备从当前系统到高级存储的平稳转换。 本指南介绍此过程中的三个主要部分：

* [计划迁移到高级存储](#plan-the-migration-to-premium-storage)
* [准备并复制虚拟硬盘 (VHD) 到高级存储](#prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage)
* [使用高级存储创建 Azure 虚拟机](#create-azure-virtual-machine-using-premium-storage)

可将其他平台中的 VM 迁移到 Azure 高级存储，或将现有 Azure VM 从标准存储迁移到高级存储。 本指南包括这两种方案的步骤。 根据具体方案，执行相关部分中指定的步骤。

> [!NOTE]
> 若要查看高级 SSD 的功能概述和定价，请参阅：[选择适用于 IaaS VM 的磁盘类型](../../virtual-machines/windows/disks-types.md#premium-ssd)。 建议将任何需要高 IOPS 的虚拟机磁盘迁移到 Azure 高级存储，以便应用程序实现最佳性能。 如果磁盘不需要高 IOPS，可以通过在标准存储（将虚拟机磁盘数据存储在硬盘驱动器 (HDD) 上而不是 SSD 上）中对其进行维护来限制成本。
>

完成整个迁移过程可能需要在执行本指南中提供的步骤前后执行其他操作。 示例包括配置虚拟网络或终结点，或在应用程序本身中进行代码更改，这可能要求应用程序一段时间内处于停用状态。 这些操作对于每个应用程序都是唯一的，应该随本指南中提供的步骤一起来完成这些操作，以便尽可能无缝地进行到高级存储的完全转换。

## <a name="plan-the-migration-to-premium-storage"></a>迁移到高级存储的计划
通过本节，可确保用户已准备好遵循本文中的迁移步骤，并帮助用户在 VM 和磁盘类型方面做出最佳决策。

### <a name="prerequisites"></a>必备组件
* 需要 Azure 订阅。 如果没有，可创建为期一个月的[免费试用](https://azure.microsoft.com/pricing/free-trial/)订阅或访问 [Azure 定价](https://azure.microsoft.com/pricing/)获取更多选择。
* 要执行 PowerShell cmdlet，将需要 Microsoft Azure PowerShell 模块。 有关安装点和安装说明，请参阅 [如何安装和配置 Azure PowerShell](/powershell/azure/overview) 。
* 计划使用在高级存储上运行的 Azure VM 时，需要使用支持高级存储的 VM。 可在支持高级存储的 VM 上使用标准和高级存储磁盘。 在将来，会有更多的 VM 类型提供高级存储磁盘。 有关所有可用 Azure VM 磁盘类型和大小的详细信息，请参阅[虚拟机大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)和[云服务大小](../../cloud-services/cloud-services-sizes-specs.md)。

### <a name="considerations"></a>注意事项
Azure VM 支持附加多个高级存储磁盘，这样应用程序的存储上限为每个 VM 256 TB。 借助高级存储，应用程序对于每个 VM 可以实现 80,000 IOPS（每秒输入/输出操作数）和每秒 2000 MB 的磁盘吞吐量，并且读取操作的延迟非常低。 可选择多种 VM 和 磁盘。 本部分可帮助用户找到最适合工作负荷的选项。

#### <a name="vm-sizes"></a>VM 大小
[虚拟机大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中列出了 Azure VM 大小规范。 查看适用于高级存储的虚拟机的性能特征并选择最适合工作负荷的 VM 大小。 确保 VM 上有足够的带宽来驱动磁盘通信。

#### <a name="disk-sizes"></a>磁盘大小
有五种类型的磁盘可用于 VM，每种磁盘都具有特定的 IOPS 和吞吐量限制。 根据应用程序在容量、性能、可伸缩性和峰值负载方面的需要为 VM 选择磁盘类型时，需要考虑这些限制。

| 高级磁盘类型  | P10   | P20   | P30            | P40            | P50            | 
|:-------------------:|:-----:|:-----:|:--------------:|:--------------:|:--------------:|
| 磁盘大小           | 128 GB| 512 GB| 1024 GB (1 TB) | 2048 GB (2 TB) | 4095 GB (4 TB) | 
| 每个磁盘的 IOPS       | 500   | 2300  | 5000           | 7500           | 7500           | 
| 每个磁盘的吞吐量 | 每秒 100 MB | 每秒 150 MB | 每秒 200 MB | 每秒 250 MB | 每秒 250 MB |

根据工作负荷，确定 VM 是否需要附加数据磁盘。 可以将多个持久性数据磁盘附加到 VM。 如有需要，可以跨磁盘条带化，以增加卷的容量与性能。 （请参阅[此处](../../virtual-machines/windows/premium-storage-performance.md#disk-striping)，了解什么是磁盘条带化。）如果使用[存储空间][4]来条带化高级存储数据磁盘，应该以使用的每个磁盘一个列的方式来配置它。 否则，条带化卷的整体性能可能会低于预期，因为磁盘之间的通信分配不平均。 对于 Linux VM，可以使用 mdadm 实用工具来实现同一目的。 有关详细信息，请参阅[在 Linux 上配置软件 RAID](../../virtual-machines/linux/configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 一文。

#### <a name="storage-account-scalability-targets"></a>存储帐户的可伸缩性目标
高级存储帐户除了 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)外还具有以下可伸缩性目标。 如果应用程序需求超过了单个存储帐户的可伸缩性目标，则在生成应用程序时请让它使用多个存储帐户，并将数据分布在这些存储帐户中。

| 总帐户容量 | 本地冗余存储帐户的总带宽 |
|:--- |:--- |
| 磁盘容量：35 TB<br />快照容量：10 TB |入站 + 出站最高每秒 50 Gbps |

有关高级存储规范的详细信息，请查看 [Azure 存储的可伸缩性和性能目标](storage-scalability-targets.md#premium-performance-storage-account-scale-limits)。

#### <a name="disk-caching-policy"></a>磁盘缓存策略
默认情况下，所有高级数据磁盘的磁盘缓存策略都是“只读”，所有附加到 VM 的高级操作系统都是“读写”。 为使应用程序的 IO 达到最佳性能，建议使用此配置设置。 对于频繁写入或只写的磁盘（例如 SQL Server 日志文件），禁用磁盘缓存可获得更佳的应用程序性能。 可以使用 [Azure 门户](https://portal.azure.com)或 *Set-AzureDataDisk* cmdlet 的 *-HostCaching* 参数更新现有数据磁盘的缓存设置。

#### <a name="location"></a>Location
选择 Azure 高级存储可用的位置。 有关可用位置的最新信息，请参阅 [Azure 服务（按区域）](https://azure.microsoft.com/regions/#services)。 与存储 VM 的磁盘的存储帐户位于同一区域的 VM 与它们在单独的区域中时相比，将提供更优异的性能。

#### <a name="other-azure-vm-configuration-settings"></a>其他 Azure VM 配置设置
在创建 Azure VM 时，需要配置某些 VM 设置。 请记住，在 VM 的生存期内有少量设置是固定不变的，但可在以后修改或添加其他设置。 请查看这些 Azure VM 配置设置，并确保这些设置都正确配置以满足工作负荷要求。

### <a name="optimization"></a>优化
[Azure 高级存储：高性能设计](../../virtual-machines/windows/premium-storage-performance.md)提供了使用 Azure 高级存储构建高性能应用程序的准则。 可结合使用这些指导和适用于应用程序所使用的技术的性能最佳做法。

## <a name="prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage"></a>准备并复制虚拟硬盘 (VHD) 到高级存储
下面的部分提供有关准备 VM 中的 VHD 和将 VHD 复制到 Azure 存储的准则。

* [场景 1：“要将现有 Azure VM 迁移到 Azure 高级存储。”](#scenario1)
* [场景 2：“要从其他平台将 VM 迁移到 Azure 高级存储。”](#scenario2)

### <a name="prerequisites"></a>必备组件
准备 VHD 迁移需具有以下条件：

* Azure 订阅、存储帐户以及该存储帐户中的一个容器（将 VHD 复制到的目标容器）。 请注意，目标存储帐户可以是标准或高级存储帐户，具体取决于具体需求。
* 用于通用化 VHD 的工具（如果计划从中创建多个 VM 实例）。 例如，sysprep for Windows 或 virt-sysprep for Ubuntu。
* 用于将 VHD 文件上传到存储帐户的工具。 请参阅[使用 AzCopy 命令行实用程序传输数据](storage-use-azcopy.md)或者使用 [Azure storage explorer](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)（Azure 存储资源管理器）。 本指南介绍使用 AzCopy 工具复制 VHD 的步骤。

> [!NOTE]
> 如果选择 AzCopy 同步复制选项，为了获得最佳性能，请从与目标存储帐户位于同一区域的 Azure VM 运行上述工具之一。 如果从其他区域中的 Azure VM 复制 VHD，性能可能会下降。
>
> 若要在带宽有限的情况下复制大量数据，可以考虑[使用 Azure 导入/导出服务将数据传输到 Blob 存储中](../storage-import-export-service.md)；这样一来，可以通过将硬盘驱动器运送到 Azure 数据中心来传输数据。 使用 Azure 导入/导出服务可以仅将数据复制到标准存储帐户。 当数据传入标准存储帐户后，可以使用[复制 Blob API](https://msdn.microsoft.com/library/azure/dd894037.aspx) 或 AzCopy 将数据传输到高级存储帐户。
>
> 请注意，Microsoft Azure 仅支持固定大小的 VHD 文件。 不支持 VHDX 文件或动态 VHD。 如果有动态 VHD，可以使用 [Convert-VHD](https://technet.microsoft.com/library/hh848454.aspx) cmdlet 将其转换为固定大小。
>
>

### <a name="scenario1"></a>场景 1：“要将现有 Azure VM 迁移到 Azure 高级存储。”
如果要迁移现有 Azure VM，请停止运行 VM，按每个所需 VHD 类型准备 VHD，并使用 AzCopy 或 PowerShell 复制 VHD。

VM 需要完全关闭，以便迁移干净状态。 在迁移完成之前会存在停机时间。

#### <a name="step-1-prepare-vhds-for-migration"></a>步骤 1。 准备 VHD 以便进行迁移
要将现有 Azure Vm 迁移到高级存储，VHD 可能为：

* 通用操作系统映像
* 唯一操作系统磁盘
* 数据磁盘

下面演练这 3 种 VHD 准备方案。

##### <a name="use-a-generalized-operating-system-vhd-to-create-multiple-vm-instances"></a>使用通用操作系统 VHD 创建多个 VM 实例
如果要上传用于创建多个泛型 Azure 虚拟机实例的 VHD，必须先使用 sysprep 实用工具通用化 VHD。 这适用于本地或云中的 VHD。 Sysprep 将从 VHD 中删除任何计算机特定的信息。

> [!IMPORTANT]
> 通用化 VM 之前，请先创建其快照或进行备份。 运行 sysprep 会停止 VM 实例和解除分配 VM 实例。 按照以下步骤对 Windows OS VHD 运行 sysprep 命令。 请注意，运行 Sysprep 命令时会要求关闭虚拟机。 有关 Sysprep 的详细信息，请参阅 [Sysprep 概述](https://technet.microsoft.com/library/hh825209.aspx)或 [Sysprep 技术参考](https://technet.microsoft.com/library/cc766049.aspx)。
>
>

1. 以管理员身份打开“命令提示符”窗口。
2. 输入以下命令打开 Sysprep：

    ```
    %windir%\system32\sysprep\sysprep.exe
    ```

3. 在系统准备工具中，选择“进入系统全新体验(OOBE)”，选中“通用化”复选框，选中“关闭”，并单击“确定”，如下图所示。 Sysprep 会通用化操作系统，并关闭系统。

    ![][1]

对于 Ubuntu VM，使用 virt-sysprep 实现同一目的。 有关更多详细信息，请参阅 [virt-sysprep](https://manpages.ubuntu.com/manpages/precise/man1/virt-sysprep.1.html)。 有关其他 Linux 操作系统，另请参阅一些开放源代码 [Linux Server Provisioning software](https://www.cyberciti.biz/tips/server-provisioning-software.html)（Linux 服务器预配软件）。

##### <a name="use-a-unique-operating-system-vhd-to-create-a-single-vm-instance"></a>使用唯一操作系统 VHD 创建单个 VM 实例
如果拥有在 VM 上运行的需要计算机特定的数据的应用程序，请不要通用化 VHD。 非通用化 VHD 可用于创建唯一的 Azure VM 实例。 例如，如果 VHD 上有域控制器，则执行 sysprep 会使它像域控制器一样没有效率。 通用化 VHD 之前，请查看 VM 上运行的应用程序，以及在这些应用程序上运行 sysprep 的影响。

##### <a name="register-data-disk-vhd"></a>注册数据磁盘 VHD
如果 Azure 中有要迁移的数据磁盘，必须确保关闭使用这些数据磁盘的 VM。

请按照下面所述的步骤将 VHD 复制到 Azure 高级存储，并将其注册为数据磁盘。

#### <a name="step-2-create-the-destination-for-your-vhd"></a>步骤 2. 创建 VHD 的目标
创建用于维护 VHD 的存储帐户。 规划 VHD 的存储位置时，应注意以下几点：

* 目标高级存储帐户。
* 存储帐户位置必须与最后阶段创建的支持高级存储的 Azure VM 相同。 可以复制到新的存储帐户，也可以根据需求计划使用同一存储帐户。
* 为下一阶段复制并保存目标存储帐户的存储帐户密钥。

对于数据磁盘，可以选择在标准存储帐户中保留一些数据磁盘（例如，具有冷却存储功能的磁盘），但我们强烈建议迁移所有数据，以便生产工作负荷使用高级存储。

#### <a name="copy-vhd-with-azcopy-or-powershell"></a>步骤 3。 使用 AzCopy 或 PowerShell 复制 VHD
处理这两个选项中的任意一个时都需找到容器路径和存储帐户密钥。 可在“Azure 门户” > “存储”中找到容器路径和存储帐户密钥。 容器 URL 将类似于"https:\//myaccount.blob.core.windows.net/mycontainer/"。

##### <a name="option-1-copy-a-vhd-with-azcopy-asynchronous-copy"></a>选项 1：使用 AzCopy 复制 VHD（异步复制）
使用 AzCopy 可通过 Internet 轻松上传 VHD。 根据 VHD 的大小，这可能需要时间。 请记住，在使用此选项时，检查存储帐户传入/传出限制。 有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)。

1. 从此处下载并安装 AzCopy：[最新版本的 AzCopy](https://aka.ms/downloadazcopy)
2. 打开 Azure PowerShell，并转到安装 AzCopy 的文件夹。
3. 使用以下命令从“Source”将 VHD 文件复制到“Destination”。

    ```azcopy
    AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
    ```

    示例：

    ```azcopy
    AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /DestKey:key2 /Pattern:abc.vhd
    ```

    下面是 AzCopy 命令中使用的参数的说明：

   * **/Source: *&lt;source&gt;:*** 包含 VHD 的文件夹或存储容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;:*** 源存储帐户的存储帐户密钥。
   * **/Dest: *&lt;destination&gt;:*** 要将 VHD 复制到的存储容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;:*** 目标存储帐户的存储帐户密钥。
   * **/Pattern: *&lt;file-name&gt;:*** 指定要复制的 VHD 文件名。

有关使用 AzCopy 工具的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](storage-use-azcopy.md)。

##### <a name="option-2-copy-a-vhd-with-powershell-synchronized-copy"></a>选项 2：使用 PowerShell 复制 VHD（同步复制）

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

还可以使用 PowerShell cmdlet Start-AzStorageBlobCopy 复制 VHD 文件。 在 Azure PowerShell 上使用以下命令复制 VHD。 将 <> 中的值替换为源和目标存储帐户中的相应值。 若要使用此命令，必须在目标存储帐户中有名为 vhds 的容器。 如果该容器不存在，则应在运行此命令之前创建一个。

```powershell
$sourceBlobUri = <source-vhd-uri>

$sourceContext = New-AzStorageContext  –StorageAccountName <source-account> -StorageAccountKey <source-account-key>

$destinationContext = New-AzStorageContext  –StorageAccountName <dest-account> -StorageAccountKey <dest-account-key>

Start-AzStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer <dest-container> -DestBlob <dest-disk-name> -DestContext $destinationContext
```

示例：

```powershell
C:\PS> $sourceBlobUri = "https://sourceaccount.blob.core.windows.net/vhds/myvhd.vhd"

C:\PS> $sourceContext = New-AzStorageContext  –StorageAccountName "sourceaccount" -StorageAccountKey "J4zUI9T5b8gvHohkiRg"

C:\PS> $destinationContext = New-AzStorageContext  –StorageAccountName "destaccount" -StorageAccountKey "XZTmqSGKUYFSh7zB5"

C:\PS> Start-AzStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer "vhds" -DestBlob "myvhd.vhd" -DestContext $destinationContext
```

### <a name="scenario2"></a>场景 2：“要从其他平台将 VM 迁移到 Azure 高级存储。”
如果要将 VHD 从非 Azure 云存储迁移到 Azure，必须首先将 VHD 导出到本地目录中。 准备好存储 VHD 的本地目录的完整源路径，并使用 AzCopy 将其上传至 Azure 存储。

#### <a name="step-1-export-vhd-to-a-local-directory"></a>步骤 1。 将 VHD 导出到本地目录
##### <a name="copy-a-vhd-from-aws"></a>从 AWS 复制 VHD
1. 如果要使用 AWS，请将 EC2 实例导出到 Amazon S3 存储桶中的 VHD。 按照 Amazon 文档导出 Amazon EC2 实例中所述的步骤安装 Amazon EC2 命令行接口 (CLI) 工具，然后运行 create-instance-export-task 命令将 EC2 实例导出到 VHD 文件。 运行 **create-instance-export-task** 命令时，请务必对 DISK&#95;IMAGE&#95;FORMAT 变量使用 **VHD**。 导出的 VHD 文件将保存在该过程中指定的 Amazon S3 存储桶中。

    ```
    aws ec2 create-instance-export-task --instance-id ID --target-environment TARGET_ENVIRONMENT \
      --export-to-s3-task DiskImageFormat=DISK_IMAGE_FORMAT,ContainerFormat=ova,S3Bucket=BUCKET,S3Prefix=PREFIX
    ```

2. 从 S3 存储桶中下载 VHD 文件。 选择 VHD 文件，并单击“操作” > “下载”。

    ![][3]

##### <a name="copy-a-vhd-from-other-non-azure-cloud"></a>从其他非 Azure 云复制 VHD
如果要将 VHD 从非 Azure 云存储迁移到 Azure，必须首先将 VHD 导出到本地目录中。 复制存储 VHD 的本地目录的完整源路径。

##### <a name="copy-a-vhd-from-on-premises"></a>从本地复制 VHD
如果要从本地环境迁移 VHD，则将需要 VHD 的存储位置的完整源路径。 源路径可以是服务器位置或文件共享。

#### <a name="step-2-create-the-destination-for-your-vhd"></a>步骤 2. 创建 VHD 的目标
创建用于维护 VHD 的存储帐户。 规划 VHD 的存储位置时，应注意以下几点：

* 目标存储帐户可以是标准或高级存储，具体取决于应用程序需求。
* 存储帐户区域必须与最后阶段创建的支持高级存储的 Azure VM 相同。 可以复制到新的存储帐户，也可以根据需求计划使用同一存储帐户。
* 为下一阶段复制并保存目标存储帐户的存储帐户密钥。

强烈建议迁移所有数据，以便生产工作负荷使用高级存储。

#### <a name="step-3-upload-the-vhd-to-azure-storage"></a>步骤 3. 将 VHD 上传到 Azure 存储
本地目录中具有 VHD 后，可使用 AzCopy 或 AzurePowerShell 将 .vhd 文件上传到 Azure 存储。 此处提供全部两个选项：

##### <a name="option-1-using-azure-powershell-add-azurevhd-to-upload-the-vhd-file"></a>选项 1：使用 Azure PowerShell Add-AzureVhd 上传 .vhd 文件

```powershell
Add-AzureVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo>
```

举例\<Uri > 可能 ***"https://storagesample.blob.core.windows.net/mycontainer/blob1.vhd"***。 举例\<FileInfo > 可能 ***"C:\path\to\upload.vhd"***。

##### <a name="option-2-using-azcopy-to-upload-the-vhd-file"></a>选项 2：使用 AzCopy 上传 .vhd 文件
使用 AzCopy 可通过 Internet 轻松上传 VHD。 根据 VHD 的大小，这可能需要时间。 请记住，在使用此选项时，检查存储帐户传入/传出限制。 有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](storage-scalability-targets.md)。

1. 从此处下载并安装 AzCopy：[最新版本的 AzCopy](https://aka.ms/downloadazcopy)
2. 打开 Azure PowerShell，并转到安装 AzCopy 的文件夹。
3. 使用以下命令从“Source”将 VHD 文件复制到“Destination”。

    ```azcopy
    AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
    ```

    示例：

    ```azcopy
    AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /DestKey:key2 /BlobType:page /Pattern:abc.vhd
    ```

    下面是 AzCopy 命令中使用的参数的说明：

   * **/Source: *&lt;source&gt;:*** 包含 VHD 的文件夹或存储容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;:*** 源存储帐户的存储帐户密钥。
   * **/Dest: *&lt;destination&gt;:*** 要将 VHD 复制到的存储容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;:*** 目标存储帐户的存储帐户密钥。
   * **/BlobType: page:** 指定目标为页 blob。
   * **/Pattern: *&lt;file-name&gt;:*** 指定要复制的 VHD 文件名。

有关使用 AzCopy 工具的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](storage-use-azcopy.md)。

##### <a name="other-options-for-uploading-a-vhd"></a>用于上传 VHD 的其他选项
也可以使用以下方法之一将 VHD 上传到存储帐户：

* [Azure 存储复制 Blob API](https://msdn.microsoft.com/library/azure/dd894037.aspx)
* [Azure 存储资源管理器上传 Blob](https://azurestorageexplorer.codeplex.com/)
* [Storage Import/Export Service REST API Reference](https://msdn.microsoft.com/library/dn529096.aspx)（存储导入/导出服务 REST API 参考）

> [!NOTE]
> 如果估计上传时间大于 7 天，则建议使用“导入/导出服务”。 可使用 [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) 根据数据大小和传输单位估计所需时间。
>
> 导入/导出可用于复制到标准存储帐户。 需要使用 AzCopy 等工具从标准存储复制到高级存储帐户。
>
>

## <a name="create-azure-virtual-machine-using-premium-storage"></a>使用高级存储创建 Azure VM
将 VHD 上传或复制到所需的存储帐户后，请按照本部分中的说明将 VHD 注册为 OS 映像或 OS 磁盘（具体取决于方案），并从中创建 VM 实例。 在创建后，可以将数据磁盘 VHD 附加到 VM。
本部分末尾提供了一个示例迁移脚本。 该示例脚本并不适用于所有方案。 可能需要更新该脚本以与特定方案相匹配。 要查看此脚本是否适用于方案，请参阅下面的[示例迁移脚本](#a-sample-migration-script)。

### <a name="checklist"></a>清单
1. 等待所有 VHD 磁盘复制完成。
2. 确保高级存储在要迁移到的区域中可用。
3. 决定要使用的新 VM 系列。 其应支持高级存储，并且其大小应具体取决于区域中的可用性和用户的需求。
4. 决定要使用的确切 VM 大小。 VM 大小需要足够大以支持所拥有的数据磁盘数。 例如 如果有 4 个数据磁盘，则 VM 必须具有 2 个或更多核心。 此外，还应考虑处理能力、内存和网络带宽需求。
5. 在目标区域中创建高级存储帐户。 这是用于新 VM 的帐户。
6. 手边具备当前 VM 详细信息，包括磁盘和对应的 VHD blob 的列表。

让应用程序做好停机准备。 为了执行净迁移，必须停止当前系统中的所有处理。 只有这样才能使其处于一致状态，可以将该状态迁移到新的平台。 停机持续时间将取决于要迁移的磁盘中的数据量。

> [!NOTE]
> 如果要从专用 VHD 磁盘创建 Azure 资源管理器 VM，请参阅 [此模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd)以使用现有磁盘部署 资源管理器 VM。
>
>

### <a name="register-your-vhd"></a>注册 VHD
要从 OS VHD 创建 VM 或将数据磁盘附加到新的 VM，必须先注册它们。 请按照以下步骤进行操作，具体取决于 VHD 方案。

#### <a name="generalized-operating-system-vhd-to-create-multiple-azure-vm-instances"></a>用于创建多个 Azure VM 实例的通用操作系统 VHD
将通用 OS 映像 VHD 上传到存储帐户后，将其注册为 **Azure VM 映像**，以便可以从中创建一个或多个 VM 实例。 使用以下 PowerShell cmdlet 将 VHD 注册为 Azure VM OS 映像。 提供 VHD 已复制到的完整容器 URL。

```powershell
Add-AzureVMImage -ImageName "OSImageName" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/osimage.vhd" -OS Windows
```

复制并保存这个新的 Azure VM 映像的名称。 在上面的示例中，名称为 *OSImageName*。

#### <a name="unique-operating-system-vhd-to-create-a-single-azure-vm-instance"></a>用于创建单个 Azure VM 实例的唯一操作系统 VHD
将唯一的 OS VHD 上传到存储帐户后，将其注册为 **Azure OS 磁盘**，以便可从中创建 VM 实例。 使用这些 PowerShell cmdlet 将 VHD 注册为 Azure OS 磁盘。 提供 VHD 已复制到的完整容器 URL。

```powershell
Add-AzureDisk -DiskName "OSDisk" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd" -Label "My OS Disk" -OS "Windows"
```

复制并保存这个新的 Azure OS 磁盘的名称。 在上面的示例中，名称为 *OSDisk*。

#### <a name="data-disk-vhd-to-be-attached-to-new-azure-vm-instances"></a>要附加到新的 Azure VM 实例的数据磁盘 VHD
将数据磁盘 VHD 上传到存储帐户后，将其注册为 Azure 数据磁盘，以便可以将它附加到新的 DS 系列、DSv2 系列或 GS 系列 Azure VM 实例。

使用这些 PowerShell cmdlet 将 VHD 注册为 Azure 数据磁盘。 提供 VHD 已复制到的完整容器 URL。

```powershell
Add-AzureDisk -DiskName "DataDisk" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/datadisk.vhd" -Label "My Data Disk"
```

复制并保存这个新的 Azure 数据磁盘的名称。 在上面的示例中，名称为 *DataDisk*。

### <a name="create-a-premium-storage-capable-vm"></a>创建支持高级存储的 VM
注册 OS 映像或 OS 磁盘后，请创建新的 DS 系列、DSv2 系列或 GS 系列 VM。 将使用注册的操作系统映像或操作系统磁盘名称。 从高级存储层选择 VM 类型。 以下示例将使用 *Standard_DS2* VM 大小。

> [!NOTE]
> 更新磁盘大小，以确保它满足容量、性能要求和可用的 Azure 磁盘大小。
>
>

逐步执行以下 PowerShell cmdlet 创建新的 VM。 首先，设置公共参数：

```powershell
$serviceName = "yourVM"
$location = "location-name" (e.g., West US)
$vmSize ="Standard_DS2"
$adminUser = "youradmin"
$adminPassword = "yourpassword"
$vmName ="yourVM"
$vmSize = "Standard_DS2"
```

首先，创建要在其中托管新 VM 的云服务。

```powershell
New-AzureService -ServiceName $serviceName -Location $location
```

接下来，根据具体方案，从注册的 OS 映像或 OS 磁盘创建 Azure VM 实例。

#### <a name="generalized-operating-system-vhd-to-create-multiple-azure-vm-instances"></a>用于创建多个 Azure VM 实例的通用操作系统 VHD
使用注册的 **Azure OS 映像**创建一个或多个新的 DS 系列 Azure VM 实例。 创建新 VM 时，在 VM 配置中指定此 OS 映像名称，如下所示。

```powershell
$OSImage = Get-AzureVMImage –ImageName "OSImageName"

$vm = New-AzureVMConfig -Name $vmName –InstanceSize $vmSize -ImageName $OSImage.ImageName

Add-AzureProvisioningConfig -Windows –AdminUserName $adminUser -Password $adminPassword –VM $vm

New-AzureVM -ServiceName $serviceName -VM $vm
```

#### <a name="unique-operating-system-vhd-to-create-a-single-azure-vm-instance"></a>用于创建单个 Azure VM 实例的唯一操作系统 VHD
使用注册的 **Azure OS 磁盘**创建新的 DS 系列 Azure VM 实例。 创建新的 VM 时，在 VM 配置中指定此 OS 磁盘名称，如下所示。

```powershell
$OSDisk = Get-AzureDisk –DiskName "OSDisk"

$vm = New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -DiskName $OSDisk.DiskName

New-AzureVM -ServiceName $serviceName –VM $vm
```

指定其他 Azure VM 信息，例如云服务、区域、存储帐户、可用性集和缓存策略。 请注意，VM 实例必须与关联的操作系统或数据磁盘同位，因此，所选的云服务、区域和存储帐户必须都与这些磁盘的基础 VHD 位于同一位置中。

### <a name="attach-data-disk"></a>附加数据磁盘
最后，如果已注册数据磁盘 VHD，请将其附加到新的支持高级存储的 Azure VM。

使用以下 PowerShell cmdlet 将数据磁盘附加到新的 VM，并指定缓存策略。 在以下示例中，缓存策略设为 ReadOnly。

```powershell
$vm = Get-AzureVM -ServiceName $serviceName -Name $vmName

Add-AzureDataDisk -ImportFrom -DiskName "DataDisk" -LUN 0 –HostCaching ReadOnly –VM $vm

Update-AzureVM  -VM $vm
```

> [!NOTE]
> 本指南可能未涵盖支持应用程序所要执行的其他步骤。
>
>

### <a name="checking-and-plan-backup"></a>检查和计划备份
启动并运行新的 VM 后，使用与原始 VM 相同的登录 ID 和密码访问它，并验证所有功能是否按预期正常工作。 所有设置（包括带区卷）都会出现在新的 VM 中。

最后一步是根据应用程序的需求规划新 VM 的备份和维护计划。

### <a name="a-sample-migration-script"></a>示例迁移脚本
如果有多个 VM 要迁移，通过 PowerShell 脚本自动执行会很有帮助。 下面是自动执行 VM 迁移的示例脚本。 请注意，下面的脚本只是一个示例并且对当前 VM 磁盘做了几点假设。 可能需要更新该脚本以与特定方案相匹配。

假设：

* 要创建经典 Azure VM。
* 源 OS 磁盘和源数据磁盘处于同一个存储帐户和同一个容器中。 如果 OS 磁盘和数据磁盘不在同一位置，可以使用 AzCopy 或 Azure PowerShell 在存储帐户和容器间复制 VHD。 请参阅上一步：[使用 AzCopy 或 PowerShell 复制 VHD](#copy-vhd-with-azcopy-or-powershell)。 另一个选择是编辑此脚本来满足方案，但建议使用 AzCopy 或 PowerShell，因为这样更快更容易。

下面提供了自动化脚本。 将文本替换为用户信息，并更新脚本以匹配特定方案。

> [!NOTE]
> 使用现有脚本不会保留源 VM 的网络配置。 需要在已迁移的 VM 上重新配置网络设置。
>
>

```powershell
    <#
    .Synopsis
    This script is provided as an EXAMPLE to show how to migrate a VM from a standard storage account to a premium storage account. You can customize it according to your specific requirements.

    .Description
    The script will copy the vhds (page blobs) of the source VM to the new storage account.
    And then it will create a new VM from these copied vhds based on the inputs that you specified for the new VM.
    You can modify the script to satisfy your specific requirement, but please be aware of the items specified
    in the Terms of Use section.

    .Terms of Use
    Copyright © 2015 Microsoft Corporation.  All rights reserved.

    THIS CODE AND ANY ASSOCIATED INFORMATION ARE PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND,
    EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY
    AND/OR FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK OF USE, INABILITY TO USE, OR
    RESULTS FROM THE USE OF THIS CODE REMAINS WITH THE USER.

    .Example (Save this script as Migrate-AzureVM.ps1)

    .\Migrate-AzureVM.ps1 -SourceServiceName CurrentServiceName -SourceVMName CurrentVMName –DestStorageAccount newpremiumstorageaccount -DestServiceName NewServiceName -DestVMName NewDSVMName -DestVMSize "Standard_DS2" –Location "Southeast Asia"

    .Link
    To find more information about how to set up Azure PowerShell, refer to the following links.
    https://azure.microsoft.com/documentation/articles/powershell-install-configure/
    https://azure.microsoft.com/documentation/articles/storage-powershell-guide-full/
    https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/

    #>

    Param(
    # the cloud service name of the VM.
    [Parameter(Mandatory = $true)]
    [string] $SourceServiceName,

    # The VM name to copy.
    [Parameter(Mandatory = $true)]
    [String] $SourceVMName,

    # The destination storage account name.
    [Parameter(Mandatory = $true)]
    [String] $DestStorageAccount,

    # The destination cloud service name
    [Parameter(Mandatory = $true)]
    [String] $DestServiceName,

    # the destination vm name
    [Parameter(Mandatory = $true)]
    [String] $DestVMName,

    # the destination vm size
    [Parameter(Mandatory = $true)]
    [String] $DestVMSize,

    # the location of destination VM.
    [Parameter(Mandatory = $true)]
    [string] $Location,

    # whether or not to copy the os disk, the default is only copy data disks
    [Parameter(Mandatory = $false)]
    [Bool] $DataDiskOnly = $true,

    # how frequently to report the copy status in seconds
    [Parameter(Mandatory = $false)]
    [Int32] $CopyStatusReportInterval = 15,

    # the name suffix to add to new created disks to avoid conflict with source disk names
    [Parameter(Mandatory = $false)]
    [String]$DiskNameSuffix = "-prem"

    ) #end param

    #######################################################################
    #  Verify Azure PowerShell module and version
    #######################################################################

    #import the Azure PowerShell module
    Write-Host "`n[WORKITEM] - Importing Azure PowerShell module" -ForegroundColor Yellow
    $azureModule = Import-Module Azure -PassThru

    if ($azureModule -ne $null)
    {
        Write-Host "`tSuccess" -ForegroundColor Green
    }
    else
    {
        #show module not found interaction and bail out
        Write-Host "[ERROR] - PowerShell module not found. Exiting." -ForegroundColor Red
        Exit
    }


    #Check the Azure PowerShell module version
    Write-Host "`n[WORKITEM] - Checking Azure PowerShell module version" -ForegroundColor Yellow
    If ($azureModule.Version -ge (New-Object System.Version -ArgumentList "0.8.14"))
    {
        Write-Host "`tSuccess" -ForegroundColor Green
    }
    Else
    {
        Write-Host "[ERROR] - Azure PowerShell module must be version 0.8.14 or higher. Exiting." -ForegroundColor Red
        Exit
    }

    #Check if there is an azure subscription set up in PowerShell
    Write-Host "`n[WORKITEM] - Checking Azure Subscription" -ForegroundColor Yellow
    $currentSubs = Get-AzureSubscription -Current
    if ($currentSubs -ne $null)
    {
        Write-Host "`tSuccess" -ForegroundColor Green
        Write-Host "`tYour current azure subscription in PowerShell is $($currentSubs.SubscriptionName)." -ForegroundColor Green
    }
    else
    {
        Write-Host "[ERROR] - There is no valid Azure subscription found in PowerShell. Please refer to this article https://azure.microsoft.com/documentation/articles/powershell-install-configure/ to connect an Azure subscription. Exiting." -ForegroundColor Red
        Exit
    }


    #######################################################################
    #  Check if the VM is shut down
    #  Stopping the VM is a required step so that the file system is consistent when you do the copy operation.
    #  Azure does not support live migration at this time..
    #######################################################################

    if (($sourceVM = Get-AzureVM –ServiceName $SourceServiceName –Name $SourceVMName) -eq $null)
    {
        Write-Host "[ERROR] - The source VM doesn't exist in the current subscription. Exiting." -ForegroundColor Red
        Exit
    }

    # check if VM is shut down
    if ( $sourceVM.Status -notmatch "Stopped" )
    {
        Write-Host "[Warning] - Stopping the VM is a required step so that the file system is consistent when you do the copy operation. Azure does not support live migration at this time. If you'd like to create a VM from a generalized image, sys-prep the Virtual Machine before stopping it." -ForegroundColor Yellow
        $ContinueAnswer = Read-Host "`n`tDo you wish to stop $SourceVMName now? Input 'N' if you want to shut down the VM manually and come back later.(Y/N)"
        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
        $sourceVM | Stop-AzureVM

        # wait until the VM is shut down
        $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
        while ($VMStatus -notmatch "Stopped")
        {
            Write-Host "`n[Status] - Waiting VM $vmName to shut down" -ForegroundColor Green
            Sleep -Seconds 5
            $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
        }
    }

    # exporting the source vm to a configuration file, you can restore the original VM by importing this config file
    # see more information for Import-AzureVM
    $workingDir = (Get-Location).Path
    $vmConfigurationPath = $env:HOMEPATH + "\VM-" + $SourceVMName + ".xml"
    Write-Host "`n[WORKITEM] - Exporting VM configuration to $vmConfigurationPath" -ForegroundColor Yellow
    $exportRe = $sourceVM | Export-AzureVM -Path $vmConfigurationPath


    #######################################################################
    #  Copy the vhds of the source vm
    #  You can choose to copy all disks including os and data disks by specifying the
    #  parameter -DataDiskOnly to be $false. The default is to copy only data disk vhds
    #  and the new VM will boot from the original os disk.
    #######################################################################

    $sourceOSDisk = $sourceVM.VM.OSVirtualHardDisk
    $sourceDataDisks = $sourceVM.VM.DataVirtualHardDisks

    # Get source storage account information, not considering the data disks and os disks are in different accounts
    $sourceStorageAccountName = $sourceOSDisk.MediaLink.Host -split "\." | select -First 1
    $sourceStorageKey = (Get-AzStorageKey -StorageAccountName $sourceStorageAccountName).Primary
    $sourceContext = New-AzStorageContext –StorageAccountName $sourceStorageAccountName -StorageAccountKey $sourceStorageKey

    # Create destination context
    $destStorageKey = (Get-AzStorageKey -StorageAccountName $DestStorageAccount).Primary
    $destContext = New-AzStorageContext –StorageAccountName $DestStorageAccount -StorageAccountKey $destStorageKey

    # Create a container of vhds if it doesn't exist
    if ((Get-AzStorageContainer -Context $destContext -Name vhds -ErrorAction SilentlyContinue) -eq $null)
    {
        Write-Host "`n[WORKITEM] - Creating a container vhds in the destination storage account." -ForegroundColor Yellow
        New-AzStorageContainer -Context $destContext -Name vhds
    }


    $allDisksToCopy = $sourceDataDisks
    # check if need to copy os disk
    $sourceOSVHD = $sourceOSDisk.MediaLink.Segments[2]
    if ($DataDiskOnly)
    {
        # copy data disks only, this option requires deleting the source VM so that dest VM can boot
        # from the same vhd blob.
        $ContinueAnswer = Read-Host "`n`t[Warning] You chose to copy data disks only. Moving VM requires removing the original VM (the disks and backing vhd files will NOT be deleted) so that the new VM can boot from the same vhd. This is an irreversible action. Do you wish to proceed right now? (Y/N)"
        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
        $destOSVHD = Get-AzStorageBlob -Blob $sourceOSVHD -Container vhds -Context $sourceContext
        Write-Host "`n[WORKITEM] - Removing the original VM (the vhd files are NOT deleted)." -ForegroundColor Yellow
        Remove-AzureVM -Name $SourceVMName -ServiceName $SourceServiceName

        Write-Host "`n[WORKITEM] - Waiting until the OS disk is released by source VM. This may take up to several minutes."
        $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
        while ($diskAttachedTo -ne $null)
        {
            Start-Sleep -Seconds 10
            $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
        }

    }
    else
    {
        # copy the os disk vhd
        Write-Host "`n[WORKITEM] - Starting copying os disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
        $allDisksToCopy += @($sourceOSDisk)
        $targetBlob = Start-AzStorageBlobCopy -SrcContainer vhds -SrcBlob $sourceOSVHD -DestContainer vhds -DestBlob $sourceOSVHD -Context $sourceContext -DestContext $destContext -Force
        $destOSVHD = $targetBlob
    }


    # Copy all data disk vhds
    # Start all async copy requests in parallel.
    foreach($disk in $sourceDataDisks)
    {
        $blobName = $disk.MediaLink.Segments[2]
        # copy all data disks
        Write-Host "`n[WORKITEM] - Starting copying data disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
        $targetBlob = Start-AzStorageBlobCopy -SrcContainer vhds -SrcBlob $blobName -DestContainer vhds -DestBlob $blobName -Context $sourceContext -DestContext $destContext -Force
        # update the media link to point to the target blob link
        $disk.MediaLink = $targetBlob.ICloudBlob.Uri.AbsoluteUri
    }

    # Wait until all vhd files are copied.
    $diskComplete = @()
    do
    {
        Write-Host "`n[WORKITEM] - Waiting for all disk copy to complete. Checking status every $CopyStatusReportInterval seconds." -ForegroundColor Yellow
        # check status every 30 seconds
        Sleep -Seconds $CopyStatusReportInterval
        foreach ( $disk in $allDisksToCopy)
        {
            if ($diskComplete -contains $disk)
            {
                Continue
            }
            $blobName = $disk.MediaLink.Segments[2]
            $copyState = Get-AzStorageBlobCopyState -Blob $blobName -Container vhds -Context $destContext
            if ($copyState.Status -eq "Success")
            {
                Write-Host "`n[Status] - Success for disk copy $($disk.DiskName) at $($copyState.CompletionTime)" -ForegroundColor Green
                $diskComplete += $disk
            }
            else
            {
                if ($copyState.TotalBytes -gt 0)
                {
                    $percent = ($copyState.BytesCopied / $copyState.TotalBytes) * 100
                    Write-Host "`n[Status] - $('{0:N2}' -f $percent)% Complete for disk copy $($disk.DiskName)" -ForegroundColor Green
                }
            }
        }
    }
    while($diskComplete.Count -lt $allDisksToCopy.Count)

    #######################################################################
    #  Create a new vm
    #  the new VM can be created from the copied disks or the original os disk.
    #  You can ddd your own logic here to satisfy your specific requirements of the vm.
    #######################################################################

    # Create a VM from the existing os disk
    if ($DataDiskOnly)
    {
        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $sourceOSDisk.DiskName
    }
    else
    {
        $newOSDisk = Add-AzureDisk -OS $sourceOSDisk.OS -DiskName ($sourceOSDisk.DiskName + $DiskNameSuffix) -MediaLocation $destOSVHD.ICloudBlob.Uri.AbsoluteUri
        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $newOSDisk.DiskName
    }
    # Attached the copied data disks to the new VM
    foreach ($dataDisk in $sourceDataDisks)
    {
        # add -DiskLabel $dataDisk.DiskLabel if there are labels for disks of the source vm
        $diskLabel = "drive" + $dataDisk.Lun
        $vm | Add-AzureDataDisk -ImportFrom -DiskLabel $diskLabel -LUN $dataDisk.Lun -MediaLocation $dataDisk.MediaLink
    }

    # Edit this if you want to add more customization to the new VM
    # $vm | Add-AzureEndpoint -Protocol tcp -LocalPort 443 -PublicPort 443 -Name 'HTTPs'
    # $vm | Set-AzureSubnet "PubSubnet","PrivSubnet"

    New-AzureVM -ServiceName $DestServiceName -VMs $vm -Location $Location
```

#### <a name="optimization"></a>优化
可以对当前 VM 配置进行专门自定义，使其很好地适用于标准磁盘。 例如，通过使用带区卷中的多个磁盘来提高性能。 例如，可通过使用单个磁盘（而不是在高级存储上单独使用 4 个磁盘）来优化成本。 此类优化需要根据具体情况进行处理，并且需要在迁移后执行自定义步骤。 另请注意，此过程可能并不适用于依赖设置中定义的磁盘布局的数据库和应用程序。

##### <a name="preparation"></a>准备工作
1. 按前面部分中所述完成简单迁移。 在迁移后会在新的 VM 上执行优化。
2. 定义优化的配置所需的新磁盘大小。
3. 确定当前磁盘/卷到新磁盘规范的映射。

##### <a name="execution-steps"></a>执行步骤
1. 在高级存储 VM 上创建具有合适大小的新磁盘。
2. 登录到该 VM 并将当前卷中的数据复制到映射到该卷的新磁盘。 对需要映射到新磁盘的所有当前卷执行此操作。
3. 接下来，更改应用程序设置以切换到新磁盘，并分离旧卷。

有关调整应用程序以实现更佳的磁盘性能的信息，请参阅[为实现高性能而设计](../../virtual-machines/windows/premium-storage-performance.md)一文中的优化应用程序性能一节。

### <a name="application-migrations"></a>应用程序迁移
数据库和其他复杂的应用程序可能需要执行应用程序提供程序定义的特殊步骤以进行迁移。 请参阅各自的应用程序文档。 例如 通常情况下，可以通过备份和还原来迁移数据库。

## <a name="next-steps"></a>后续步骤
有关虚拟机迁移的特定方案，请参阅以下资源：

* [Migrate Azure Virtual Machines between Storage Accounts](https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/)（在存储帐户之间迁移 Azure 虚拟机）
* [创建 Windows Server VHD 并将其上传到 Azure。](../../virtual-machines/windows/upload-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [创建 Linux VHD 并将其上传到 Azure](../../virtual-machines/linux/create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [将虚拟机从 Amazon AWS 迁移到 Microsoft Azure](https://channel9.msdn.com/Series/Migrating-Virtual-Machines-from-Amazon-AWS-to-Microsoft-Azure)

另请参阅以下资源，以了解有关 Azure 存储和 Azure 虚拟机的详细信息：

* [Azure 存储](https://azure.microsoft.com/documentation/services/storage/)
* [Azure 虚拟机](https://azure.microsoft.com/documentation/services/virtual-machines/)
* [选择适用于 IaaS VM 的磁盘类型](../../virtual-machines/windows/disks-types.md)

[1]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-1.png
[2]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-1.png
[3]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-3.png
[4]: https://technet.microsoft.com/library/hh831739.aspx
