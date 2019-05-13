---
title: 在 Azure SAP HANA（大型实例）上进行 HANA 备份和还原 | Microsoft Docs
description: 如何在 Azure SAP HANA（大型实例）上执行 HANA 备份和还原
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/22/2019
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 21232e5a678d6deed920e57cd0433a3b85ca4fdc
ms.sourcegitcommit: 60606c5e9a20b2906f6b6e3a3ddbcb6c826962d6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/01/2019
ms.locfileid: "64987902"
---
# <a name="backup-and-restore"></a>备份和还原

>[!IMPORTANT]
>本文不能取代 SAP HANA 管理文档或 SAP 说明。 我们预计有扎实的了解和 SAP HANA 管理和操作，尤其是对于备份、 还原、 高可用性和灾难恢复方面的专业知识。 在本文中，会显示在 SAP HANA Studio 中的屏幕快照。 SAP 管理工具屏幕的内容、结构和性质以及工具本身可能会因不同的 SAP HANA 发行版本而有所差异。

重要的是使用自己的 HANA 版本或发布，在相应的环境中执行步骤和过程。 在本文中介绍的某些过程进行了简化，以便更好地大致了解。 它们不是要用作最终操作手册的详细步骤。 如果你想要针对你的配置创建操作手册，测试和执行您的流程，并记录与特定配置相关的进程。 

操作数据库的最重要方面之一是防止发生灾难性事件。 这些事件可能由多种因素导致，包括自然灾难、简单的人为失误，等等。

将数据库备份，能够恢复到任何点的时间，如之前某人删除关键数据，使还原到的状态为关闭以尽可能接近中断之前所处的方法。

若要实现的功能来还原，必须执行两种类型的备份：

- 数据库备份：完整备份、增量备份或差异备份
- 事务日志备份

除了在应用程序级别执行完整数据库备份以外，还可以使用存储快照执行备份。 存储快照不会替代事务日志备份。 事务日志备份对于将数据库还原到某个时间点或从已提交的事务中清空日志而言十分重要。 存储快照可以通过快速提供数据库的前滚映像来加快恢复。 

Azure 上的 SAP HANA（大型实例）提供两个备份和还原选项：

- **执行此操作自制 (DIY)。** 请确保足够的磁盘空间后，使用以下磁盘备份方法之一执行完整数据库备份和日志备份。 可以备份直接到附加到 HANA 大型实例单元或在 Azure 虚拟机 (VM) 中设置的 NFS 共享的卷。 在后一种情况中，客户在 Azure 中设置 Linux VM、将 Azure 存储附加到 VM 并通过该 VM 中配置的 NFS 服务器共享此存储。 如果执行对直接附加到 HANA 大型实例单元的卷进行备份，将备份复制到 Azure 存储帐户。 设置完导出 NFS 共享的基于 Azure 存储的 Azure VM 后执行此操作。 此外可以使用 Azure 备份保管库或 Azure 冷存储。 

   另一个选项是使用第三方数据保护工具来存储备份后将其复制到 Azure 存储帐户。 DIY 备份选项也可能有必要为需要更长一段时间的符合性和审核目的而存储的数据。 在所有情况下，备份均复制到通过 VM 和 Azure 存储表示的 NFS 共享中。

- **基础结构备份和还原功能。** 此外可以使用备份和还原 SAP HANA on Azure （大型实例） 底层基础结构提供的功能。 此选项满足备份和快速还原的需要。 本部分的余下内容介绍 HANA 大型实例提供的备份和还原功能。 本部分还介绍了此关系的备份和还原到灾难恢复功能提供的 HANA 大型实例。

> [!NOTE]
>   使用 HANA 大型实例的底层基础结构的快照技术程序依赖于 SAP HANA 快照。 在此情况下，SAP HANA 快照不与 SAP HANA 多租户数据库容器的多个租户配合工作。 如果只部署一个租户，则 SAP HANA 快照工作，并可以使用此方法。

## <a name="use-storage-snapshots-of-sap-hana-on-azure-large-instances"></a>使用 Azure （大型实例） 上的 SAP HANA 存储快照

Azure 上的 SAP HANA（大型实例）的底层存储基础结构支持卷的存储快照。 支持备份和还原卷，不过需要注意以下事项：

- 它不是创建完整的数据库备份，而是频繁创建存储卷的快照。
- 快照触发对 /hana/data 和 /hana/shared，其中包括 /usr/sap、 卷、 快照技术启动 SAP HANA 快照之前它运行存储快照。 在恢复存储快照后，此 SAP HANA 快照是最终日志还原的设置点。 对于成功的 HANA 快照，您需要的活动的 HANA 实例。 在 HSR 方案中，存储快照不支持的 HANA 快照无法执行其中的当前辅助节点上。
- 存储快照已成功运行后，将删除 SAP HANA 快照。
- 事务日志备份需频繁创建，并存储在 /hana/logbackups 卷或 Azure 中。 可以单独触发包含事务日志备份的 /hana/logbackups 卷来创建快照。 在这种情况下，不需要运行的 HANA 快照。
- 如果您必须数据库还原到特定的时间点的时间，造成生产中断请求的 Microsoft Azure 支持部门或 SAP HANA Azure 还原到特定的存储快照。 一个例子就是按计划将沙盒系统还原到其原始状态。
- 存储快照中包含的 SAP HANA 快照是应用运行并创建存储快照后存储的事务日志备份偏移的点。
- 创建这些事务日志备份是为了将数据库还原到某个时间点。

您可以执行的卷的三个类为目标的存储快照：

- / Hana/数据和/hana/共享，通过创建组合的快照，其中包括 /usr/sap。 此快照需要创建 SAP HANA 快照，为存储快照进行准备工作。 SAP HANA 快照可确保数据库处于从存储角度处于一致状态。 对于还原过程中，这是一个点上设置。
- 基于 /hana/logbackups 创建单独的快照。
- 操作系统分区。

若要获取最新的快照脚本和文档，请参阅[GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0)。 在下载中的快照脚本包[GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0)，您会获得三个文件。 其中一个文件记录在 PDF 中，以提供的功能。 下载该工具集后，请按照的说明进行操作，在"获取快照工具。"

## <a name="storage-snapshot-considerations"></a>存储快照注意事项

>[!NOTE]
>存储快照使用分配到 HANA 大型实例单元的存储空间。 请考虑以下方面的计划存储快照和要保留的存储快照数量。 

Azure 上的 SAP HANA（大型实例）的存储快照的特定机制包括：

- 特定的存储快照在创建时的时间点会占用少的存储空间。
- 随着数据内容更改和文件更改存储卷的 SAP HANA 数据中的内容，快照需要存储原始块内容和数据更改。
- 因此，存储快照的大小会不断增长。 快照保留的时间越长，存储快照就会变得越大。
- 在存储快照的生存期内对 SAP HANA 数据库卷所做的更改越多，存储快照消耗的空间也就越大。

Azure 上的 SAP HANA（大型实例）为 SAP HANA 数据卷和日志卷使用固定的卷大小。 执行这些卷的快照会消耗卷空间。 你需要：

- 确定何时计划存储快照。
- 监视存储卷的空间消耗。 
- 管理存储的快照数。 

将大量数据导入 HANA 数据库或对其执行其他重大更改时，可以禁用存储快照。 


以下各节提供有关执行这些快照的信息，并且包括一般建议：

- 尽管硬件可以保留每个卷的 255 快照，但你想要远低于此数字保持。 建议是 250 个或更少。
- 执行存储快照之前，请监视并跟踪可用空间。
- 根据可用空间减少存储快照数量。 可以减少保留的快照数量，或者扩展卷。 可以 1 TB 为单位订购附加存储。
- 在使用 SAP 平台迁移工具 (R3load) 将数据移入 SAP HANA，或者从备份还原 SAP HANA 数据库或执行其他活动期间，可以禁用 /hana/data 卷上的存储快照。 
- 在 SAP HANA 表进行重大重组，如有可能避免存储快照。
- 存储快照是利用 Azure 上的 SAP HANA（大型实例）灾难恢复功能的先决条件。

## <a name="prerequisites-for-using-self-service-storage-snapshots"></a>使用自助服务存储快照的先决条件

若要确保快照脚本成功运行，请确保在 HANA 大型实例服务器上的 Linux 操作系统系统中安装 Perl。 Perl 随附预安装在 HANA 大型实例单元上。 若要检查 Perl 版本，请使用以下命令：

`perl -v`

![运行此命令复制公钥](./media/hana-overview-high-availability-disaster-recovery/perl_screen.png)


## <a name="set-up-storage-snapshots"></a>设置存储快照

若要设置 HANA 大型实例存储快照，请执行以下步骤。
1. 确保在 HANA 大型实例服务器上的 Linux 操作系统中安装 Perl。
1. 修改 /etc/ssh/ssh\_config，添加代码行 _MACs hmac-sha1_。
1. 如果适用，请运行，每个 SAP HANA 实例的主节点上创建一个 SAP HANA 备份用户帐户。
1. 在所有 SAP HANA 大型实例服务器上安装 SAP HANA HDB 客户端。
1. 在每个区域的第一台 SAP HANA 大型实例服务器上，创建一个公钥用于访问对快照创建操作进行控制的底层存储基础结构。
1. 将脚本和配置文件从 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0) 复制到 SAP HANA 安装中的 **hdbsql** 位置。
1. 必要时，根据相应的客户规范修改 *HANABackupDetails.txt* 文件。

从 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0) 获取最新的快照脚本和文档。 前面列出的步骤，请参阅[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

### <a name="consideration-for-mcod-scenarios"></a>MCOD 方案的注意事项
如果在运行[MCOD 方案](https://launchpad.support.sap.com/#/notes/1681092)使用一个 HANA 大型实例单元上的多个 SAP HANA 实例，可以为每个 SAP HANA 实例预配的单独的存储卷。 MDC 和其他注意事项的详细信息，请参阅"重要事项需记住"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。
 

### <a name="step-1-install-the-sap-hana-hdb-client"></a>步骤 1：安装 SAP HANA HDB 客户端

安装 Azure （大型实例） 上的 SAP HANA 上的 Linux 操作系统包含的文件夹和运行 SAP HANA 存储快照的备份和灾难恢复目的所需的脚本。 在 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0) 中查找最新版本。 脚本的最新版本为 4.0。 相同的主要版本中，不同的脚本可能具有不同的次要发行版本。

它由你负责在安装 SAP HANA 时在 HANA 大型实例单元上安装 SAP HANA HDB 客户端。

### <a name="step-2-change-the-etcsshsshconfig"></a>步骤 2：更改 /etc/ssh/ssh\_config

此步骤中所述"启用与存储的通信" [Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。


### <a name="step-3-create-a-public-key"></a>步骤 3：创建公钥

若要启用访问 HANA 大型实例租户的存储快照接口，请通过公钥建立登录过程。 

在你的租户中的 Azure （大型实例） 服务器上的第一个 SAP HANA，创建一个公钥用于访问存储基础结构。 提供一个公用密钥，无需密码即可登录到存储快照接口。 您还不需要具有公共键保留密码凭据。 

若要生成的公共密钥，请参阅在"启用与存储的通信" [Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。


### <a name="step-4-create-an-sap-hana-user-account"></a>步骤 4：创建 SAP HANA 用户帐户

若要启动 SAP HANA 快照的创建，请在存储快照脚本可以使用的 SAP HANA 中创建用户帐户。 在 SAP HANA Studio 中创建用于此目的的 SAP HANA 用户帐户。 必须在 SYSTEMDB 下创建用户并*不*MDC SID 数据库下。 在单个容器环境中，在租户数据库中创建用户。 此帐户必须具有**备份管理员**并**目录读取**特权。 

设置和使用用户帐户，请参见"启用通信的 SAP HANA"中的[GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0)。


### <a name="step-5-authorize-the-sap-hana-user-account"></a>步骤 5：为 SAP HANA 用户帐户授权

在此步骤中，您授权，以便脚本无需提交密码在运行时创建的 SAP HANA 用户帐户。 SAP HANA 命令`hdbuserstore`启用的 SAP HANA 用户密钥的创建。 密钥存储在一个或多个 SAP HANA 节点上。 用户可以使用用户密钥从脚本流程内部访问 SAP HANA，而无需管理密码。 本文稍后会介绍脚本过程。

>[!IMPORTANT]
>使用相同的用户上下文中运行的快照命令的运行这些配置命令。 否则，将无法正常工作的快照命令。


### <a name="step-6-get-the-snapshot-scripts-configure-the-snapshots-and-test-the-configuration-and-connectivity"></a>步骤 6：获取快照脚本、配置快照及测试配置和连接

从 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/tree/master/snapshot_tools_v4.0) 下载最新版本的脚本。 版本 4.0 中的脚本，脚本将安装的方式发生变化。 详细信息，请参阅"启用通信的 SAP HANA"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

具体命令顺序，请参阅"对快照工具 （默认值） 的简单安装"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 我们建议使用默认安装。 

若要从版本升级 3.x 到 4.0，请参阅"升级现有安装"中的[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 若要卸载 4.0 工具集，请参阅在"快照工具卸载" [Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

不要忘记运行"的快照工具完成安装程序"中所述的步骤，在[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

不同脚本和文件的用途有安装是中所述"什么是这些快照工具？" 在中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

配置快照工具之前，请确保您还配置 HANA 备份位置和设置正确。 详细信息，请参阅"SAP HANA 配置"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

在"配置文件-HANABackupCustomerDetails.txt"中介绍了快照工具集的配置[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

#### <a name="test-connectivity-with-sap-hana"></a>测试与 SAP HANA 的连接

将所有配置数据填入 *HANABackupCustomerDetails.txt* 文件后，需要检查 HANA 实例数据的相关配置是否正确。 使用脚本独立于 SAP HANA 纵向扩展或横向扩展配置的脚本 `testHANAConnection`。

详细信息，请参阅"使用 SAP HANA-testHANAConnection 检查连接"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

#### <a name="test-storage-connectivity"></a>测试存储连接

下一步的测试步骤是要检查其连接到基于放入的数据存储*HANABackupCustomerDetails.txt*配置文件。 然后，运行测试快照。 在运行之前`azure_hana_backup`命令时，必须运行此测试。 对于此测试的命令序列，请参阅"使用存储空间-testStorageSnapshotConnection 检查连接""中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

成功登录到存储虚拟机接口后，脚本会继续执行阶段 2 并创建测试快照。 输出显示在此处为 SAP HANA 的三节点横向扩展配置。

如果使用脚本成功运行的测试快照，您可以计划实际存储快照。 如果不成功，调查的问题之前向前移动。 测试快照应保留至第一批实际快照完成为止。


### <a name="step-7-perform-snapshots"></a>步骤 7：执行快照

完成准备步骤后，你可以开始配置和计划的实际存储快照。 要计划的脚本会使用 SAP HANA 纵向扩展和横向扩展配置。 备份脚本的定期和常规执行应使用 cron 实用工具来计划。 

具体命令语法和功能，请参阅"执行快照备份-azure_hana_backup"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 

当脚本`azure_hana_backup`运行时，它将创建存储快照中以下三个阶段：

1. 它将运行 SAP HANA 快照。
1. 它在运行存储快照。
1. 它会删除存储快照运行之前，已创建的 SAP HANA 快照。

若要运行该脚本，请从复制到的 HDB 可执行文件文件夹调用它。 

保留期会根据与运行脚本时作为参数提交的快照数管理。 存储快照消耗量是时间的期间执行，并运行脚本时作为参数提交的快照数的函数。 

如果保留的快照数超过命名为脚本的调用中的参数数目，新的快照运行前删除最旧的相同标签的存储快照。 作为最后一个调用参数指定的数字是可用于控制保留的快照数的数字。 使用此编号，您还可以控制，间接，快照使用的磁盘空间。 


## <a name="snapshot-strategies"></a>快照策略
不同类型快照的频率取决于是否使用 HANA 大型实例灾难恢复功能。 此功能依赖于存储快照，这可能需要实施某些与存储快照频率和执行时间长短相关的特殊建议。 

以下注意事项和建议假设不使用 HANA 大型实例提供的灾难恢复功能。 假设用户使用存储快照来获取备份并提供过去 30 天的时点恢复。 鉴于快照数和空间限制，请考虑以下要求：

- 时间点恢复的恢复时间。
- 使用的空间。
- 潜在灾难恢复的恢复点和恢复时间目标。
- 针对磁盘最终执行的 HANA 完整数据库备份。 每当针对磁盘执行完整数据库备份或执行 **backint** 接口时，存储快照的执行会失败。 如果你打算运行基于存储快照的完整数据库备份，请确保在此期间已禁用存储快照的执行。
- 每个卷，不能超过 250 的快照数。

<!-- backint is term for a SAP HANA interface and not a spelling error not spelling errors -->

如果不使用 HANA 大型实例的灾难恢复功能，快照会不太频繁。 在这种情况下，对 /hana/data 和 /hana/shared /usr/sap，其包括在 12 小时或 24 小时的周期执行组合的快照。 快照保留一个月。 这同样适用于日志备份卷的快照。 15 分钟的周期，在 5 分钟内发生的日志备份卷的 SAP HANA 事务日志备份执行。

最好是使用 cron 执行计划的存储快照。 所有备份和灾难恢复需求都使用同一个脚本。 修改脚本输入，使之与请求的不同备份时间匹配。 这些快照一切计划在其执行时间 cron 中做好。 它可以是每小时、 每隔 12 小时、 每天或每周。 

下面的示例演示 /etc/crontab 中的 cron 计划：
```
00 1-23 * * * ./azure_hana_backup --type=hana --prefix=hourlyhana --frequency=15min --retention=46
10 00 * * *  ./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
00,05,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
22 12 * * *  ./azure_hana_backup --type=logs --prefix=dailylogback --frequncy=3min --retention=28
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeI --prefix=dailyboot --frequncy=15min --retention=28
```
在上一示例中，每小时的组合的快照涵盖包含 /hana/data 和 /hana/shared/SID，包括 /usr/sap，位置的卷。 使用此类型的快照快速实现过去两天内的时点恢复。 此外，还有每日上这些卷的快照。 因此，您两天的情况下具有每时快照和四周的每日快照。 事务日志备份卷也是每日备份。 这些备份将保留四周。 

正如您看到 crontab 第三行中，HANA 事务日志备份计划每隔 5 分钟运行一次。 运行存储快照的不同 cron 作业的开始时间被错开。 这样一来，快照不运行一次性全部在某一时间点的时间。 

在以下示例中，执行组合的快照涵盖包含 /hana/data 和 /hana/shared/SID (包括 /usr/sap） 位置上以小时为单位的卷。 这些快照会保留两天。 事务日志备份卷的快照运行 5 分钟，并保留四个小时。 与之前一样，HANA 事务日志文件的备份计划以每 5 分钟运行一次。 

启动事务日志备份后，执行事务日志备份卷的快照时会存在 2 分钟的延迟。 正常情况下，SAP HANA 事务日志备份这 2 分钟内完成。 与之前一样，包含启动 LUN 的卷通过存储快照每天备份一次，且会保留四周。

```
10 0-23 * * * ./azure_hana_backup --type=hana ==prefix=hourlyhana --frequency=15min --retention=48
0,5,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
2,7,12,17,22,27,32,37,42,47,52,57 * * * *  ./azure_hana_backup --type=logs --prefix=logback --frequency=3min --retention=48
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeII --prefix=dailyboot --frequency=15min --retention=28
```

下图演示了上一示例的序列。 排除启动 LUN。

![备份和快照之间的关系](./media/hana-overview-high-availability-disaster-recovery/backup_snapshot_updated0921.PNG)

SAP HANA 对 /hana/log 卷执行常规写入，将提交的更改记录到数据库。 SAP HANA 将保存点定期写入 /hana/data 卷。 根据 crontab 中所指定，SAP HANA 事务日志备份每 5 分钟运行。 

你还可以看到，SAP HANA 快照在运行触发的组合的存储快照 /hana/data 和 /hana/shared/SID 卷每隔一小时。 成功执行 HANA 快照后，组合的存储快照运行。 如 crontab 中所指示，/hana/logbackup 卷上的存储快照运行每隔 5 分钟，HANA 事务日志备份后 2 分钟。

> 

>[!IMPORTANT]
> 仅当与 SAP HANA 事务日志备份配合执行存储快照时，才能针对 SAP HANA 备份使用存储快照。 这些事务日志备份需要能够涵盖执行存储快照的间隔时间段。 

如果向用户承诺时间点恢复目标为 30 天，则需要：

- 对 /hana/data、 和 /hana/shared/SID 为 30 天，在极端情况下访问组合的存储快照。 
- 创建保留时长长于任何两个组合存储快照间隔时间的连续事务日志备份。 因此事务日志备份卷快照的最长保留时长需为 30 天。 如果将事务日志备份复制到另一个位于 Azure 存储的 NFS 共享，这不是这种情况。 在这种情况下，可以从该 NFS 共享拉取旧事务日志备份。

若要从存储快照和事务日志备份的最终存储复制中获益，更改 SAP HANA 写入事务日志备份的位置。 可在 HANA Studio 中进行此更改。 

虽然 SAP HANA 会自动备份完整日志段，但指定的日志备份间隔时间是确定性的。 当使用灾难恢复选项，因为你通常想要使用确定性的周期运行日志备份时，也是如此。 在以下情况下，15 分钟设置为日志备份间隔时间。

![在 SAP HANA Studio 中计划 SAP HANA 备份日志](./media/hana-overview-high-availability-disaster-recovery/image5-schedule-backup.png)

此外可以选择高于每隔 15 分钟的备份。 较高的频率设置通常与 HANA 大型实例的灾难恢复功能结合使用。 某些客户每 5 分钟执行一次事务日志备份。

如果从未备份过数据库，则最后一步是执行基于文件的数据库备份，以创建必须在备份目录中存在的单个备份项。 否则，SAP HANA 无法启动指定的日志备份。

![创建基于文件的备份以创建单个备份项](./media/hana-overview-high-availability-disaster-recovery/image6-make-backup.png)


运行第一个成功存储快照后，删除在步骤 6 中运行的测试快照。 详细信息，请参阅"删除测试快照-removeTestStorageSnapshot"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 


### <a name="monitor-the-number-and-size-of-snapshots-on-the-disk-volume"></a>监视的数量和大小的磁盘卷上的快照

在特定的存储卷上，可以监视快照的数量以及这些快照的存储消耗量。 `ls` 命令不会显示快照目录或文件。 Linux OS 命令`du`显示有关这些存储快照的详细信息，因为它们存储在相同的卷上。 该命令使用以下选项：

- `du –sh .snapshot`：此选项提供快照目录中所有快照的总数。
- `du –sh --max-depth=1`：此选项列出 .snapshot 文件夹中保存的所有快照，以及每个快照的大小。
- `du –hc`：此选项提供所有快照占用的总大小。

使用以下命令以确保创建和存储的快照不使用卷上的所有存储。

>[!NOTE]
>启动 LUN 的快照不是使用上述命令可见。

### <a name="get-details-of-snapshots"></a>获取快照的详细信息
要获取有关快照的更多详细信息，请使用脚本`azure_hana_snapshot_details`。 如果灾难恢复位置中没有活动的服务器，可以在任一位置运行此脚本。 此脚本根据每个包含快照的卷提供以下细分输出： 
   * 一个卷中全部快照的大小
   * 该卷中每个快照包含以下详细信息： 
      - 快照名称 
      - 创建时间 
      - 快照大小
      - 快照频率
      - 与该快照关联的 HANA 备份 ID（如果相关）

命令和输出的语法，请参阅"列出快照-azure_hana_snapshot_details"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 



### <a name="reduce-the-number-of-snapshots-on-a-server"></a>减少服务器上的快照数

如前所释，可以减少带有特定标签的存储的快照的数。 用于启动快照的命令的最后两个参数是要保留的快照的标签与数量。

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
```

在上一示例中，快照标签是指**dailyhana**。 带有此标签保留的快照数是**28**。 对磁盘空间消耗量做出响应时，可能想要减少存储的快照数。 以减少到 15，例如，快照数的简单方法是运行该脚本的最后一个参数设置为**15**:

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=15
```

如果使用此设置运行该脚本，其中包括新的存储快照，快照数为 15。 会保存最新的 15 个快照，同时删除 15 个较旧的快照。

 >[!NOTE]
 > 仅当有多个一小时的快照，此脚本才能减少快照数。 该脚本不会删除不超过一个小时的快照。 这些限制与提供的可选灾难恢复功能相关。

如果不再想要保留一组包含备份的前缀的快照**dailyhana**语法示例中运行脚本时使用**0**作为保留数量。 然后删除与该标签匹配的所有快照。 删除所有快照可能会影响 HANA 大型实例灾难恢复功能的功能。

第二种删除特定快照的方法是使用脚本 `azure_hana_snapshot_delete`。 此脚本旨在使用在 HANA Studio 中找到的 HANA 备份 ID 或通过快照名称本身删除快照或一组快照。 目前，备份 ID 仅与为 **hana** 快照类型创建的快照绑定。 类型的快照备份**日志**并**启动**不执行 SAP HANA 快照，因此没有可找到这些快照的备份 ID。 如果输入快照名称，将在不同卷上查找所有与输入的快照名称匹配的快照。 

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->

该脚本的详细信息，请参阅"删除快照-azure_hana_snapshot_delete"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。

以用户身份运行该脚本**根**。

>[!IMPORTANT]
>如果仅存在于快照的数据您计划删除，请在删除快照后，数据也将永远丢失。


## <a name="file-level-restore-from-a-storage-snapshot"></a>从存储快照实现文件级还原

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->
对于 **hana** 和 **logs** 快照类型，可以直接在 **.snapshot** 目录中的卷上访问快照。 没有为每个快照的子目录。 将每个文件复制到实际目录结构该子目录中的快照瞬间的状态中。 

该脚本的当前版本中没有*没有*还原提供快照还原为自助服务脚本。 在故障转移期间，可以在灾难恢复站点的自助服务的灾难恢复脚本的一部分执行快照还原。 若要从现有的可用快照还原所需的快照，必须通过打开服务请求联系 Microsoft 运营团队。

>[!NOTE]
>单个文件还原不适用于启动与 HANA 大型实例单元的类型无关的 LUN 的快照。 **.Snapshot**目录不在启动 LUN 中公开。 
 

## <a name="recover-to-the-most-recent-hana-snapshot"></a>恢复到最近的 HANA 快照

在生产中断的情况，可以向 Microsoft Azure 支持提出客户事件启动从存储快照恢复的过程。 如果在生产系统中删除了数据并对其进行检索的唯一方法是还原生产数据库，这是造成的高度紧急问题。

在不同的情况下，时间点恢复可能不太紧急，因此可以提前几天规划。 可以在 Azure 上规划与 SAP HANA 此恢复，而不要提出高优先级标志。 例如，您可能会计划通过应用新的增强包升级 SAP 软件。 然后又需要还原到升级增强包之前的快照状态。

发送请求之前，需要做好准备。 然后，Azure 团队上的 SAP HANA 可以处理该请求，并提供已还原的卷。 接下来，需要基于快照还原 HANA 数据库。

有关用于获取快照与新的工具集，还原的可能性中，请参阅"如何还原快照"[手动恢复的 SAP HANA 指南在 Azure 上从存储快照](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/guides/Manual%20recovery%20of%20snapshot%20with%20HANA%20Studio.pdf)。

若要准备请求，请执行以下步骤。

1. 确定要还原的快照。 除非另有说明，否则只能还原 hana/data 卷。 

1. 关闭 HANA 实例。

   ![关闭 HANA 实例](./media/hana-overview-high-availability-disaster-recovery/image7-shutdown-hana.png)

1. 卸载每个 HANA 数据库节点上的数据卷。 如果仍将数据卷载入操作系统，快照还原会失败。

   ![卸载每个 HANA 数据库节点上的数据卷](./media/hana-overview-high-availability-disaster-recovery/image8-unmount-data-volumes.png)

1. 打开 Azure 支持请求，并包括有关还原特定快照的说明：

   - 还原期间：Azure 服务上的 SAP HANA 可能会要求用户参加电话会议以进行协调、 验证和确认还原正确的存储快照。 

   - 还原后：还原存储快照时，Azure 服务上的 SAP HANA 将通知你。

1. 完成还原过程后，请重新装载所有数据卷。

   ![重新装载所有数据卷](./media/hana-overview-high-availability-disaster-recovery/image9-remount-data-volumes.png)



用于获取，例如，从在存储快照恢复的 SAP HANA 数据文件的另一种可能性中的步骤 7 中所述[手动恢复的 SAP HANA 指南在 Azure 上从存储快照](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/guides/Manual%20recovery%20of%20snapshot%20with%20HANA%20Studio.pdf)。

若要从快照备份还原，请参阅[手动恢复的 SAP HANA 指南在 Azure 上从存储快照](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/guides/Manual%20recovery%20of%20snapshot%20with%20HANA%20Studio.pdf)。 

>[!Note]
>如果在快照已还原的 Microsoft 操作，不需要执行步骤 7。


### <a name="recover-to-another-point-in-time"></a>恢复到另一个时间点
若要还原到特定的时间点的时间，请参阅"将数据库恢复到以下点时间"中[手动恢复的 SAP HANA 指南在 Azure 上从存储快照](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/guides/Manual%20recovery%20of%20snapshot%20with%20HANA%20Studio.pdf)。 


## <a name="next-steps"></a>后续步骤
- 请参阅[灾难恢复原则和准备](hana-concept-preparation.md)。
