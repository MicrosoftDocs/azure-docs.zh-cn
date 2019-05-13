---
title: 使用 Azure Site Recovery 重新保护已故障转移回到主要 Azure 区域的 Azure VM | Microsoft Docs
description: 介绍在从主要区域故障转移后，如何使用 Azure Site Recovery 在次要区域中重新保护 Azure VM。
services: site-recovery
author: rajani-janaki-ram
manager: gauravd
ms.service: site-recovery
ms.topic: article
ms.date: 11/27/2018
ms.author: rajanaki
ms.openlocfilehash: eabb7d194a3ef65282befab1ae59e85ba56f2f5b
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472162"
---
# <a name="reprotect-failed-over-azure-vms-to-the-primary-region"></a>重新保护已故障转移到主要区域的 Azure VM


使用 [Azure Site Recovery](site-recovery-overview.md) 将 Azure VM 从一个区域[故障转移](site-recovery-failover.md)到另一个区域后，VM 将在次要区域中启动，且不受保护。 如果将 VM 故障转移回到主要区域，则需要执行以下操作：

- 在次要区域中重新保护 VM，以便这些 VM 开始复制到主要区域。
- 重新保护完成并且 VM 开始复制后，可将 VM 从次要区域故障转移到主要区域。

## <a name="prerequisites"></a>必备组件
1. 必须提交将 VM 从主要区域故障转移到次要区域的操作。
2. 主要目标站点应该可用，并且应该能够在该区域中访问或创建资源。

## <a name="reprotect-a-vm"></a>重新保护 VM

1. 在“保管库” > “复制的项”中，右键单击已故障转移的 VM，并选择“重新保护”。 重新保护方向应该显示为从次要区域到主要区域。

   ![重新保护](./media/site-recovery-how-to-reprotect-azure-to-azure/reprotect.png)

2. 查看资源组、网络、存储和可用性集。 然后单击“确定”。 如果有任何资源标记为“新”，则表示它们是在重新保护过程中创建的。
3. 重新保护作业使用最新数据植入目标站点。 完成该过程后，将发生增量复制。 然后，可以故障转移回到主要站点。 可以使用自定义选项，选择要在重新保护期间使用的存储帐户或网络。

   ![自定义选项](./media/site-recovery-how-to-reprotect-azure-to-azure/customize.png)

### <a name="customize-reprotect-settings"></a>自定义重新保护设置

可在重新保护期间自定义以下目标 VM 的属性。

![自定义](./media/site-recovery-how-to-reprotect-azure-to-azure/customizeblade.png)

|属性 |说明  |
|---------|---------|
|目标资源组     | 修改创建 VM 的目标资源组。 在重新保护的过程中，将删除目标 VM。 可以选择在故障转移后要在其下创建 VM 的新资源组。        |
|目标虚拟网络     | 执行重新保护作业期间，无法更改目标网络。 若要更改网络，请重做网络映射。         |
|目标存储（辅助 VM 不使用托管磁盘）     | 可以更改故障转移后 VM 使用的存储帐户。         |
|副本托管磁盘（辅助 VM 使用托管磁盘）    | Site Recovery 在主要区域中创建副本托管磁盘以监视辅助 VM 的托管磁盘。         |
|缓存存储     | 可以指定复制期间要使用的缓存存储帐户。 默认情况下，如果不存在缓存存储帐户，则会创建一个帐户。         |
|可用性集     |如果次要区域中的 VM 属于可用性集，则可为主要区域中的目标 VM 选择可用性集。 默认情况下，Site Recovery 会尝试在主要区域中查找并使用现有的可用性集。 在自定义期间，可以指定新的可用性集。         |


### <a name="what-happens-during-reprotection"></a>重新保护期间会发生什么情况？

默认会发生以下情况：

1. 在运行故障转移的 VM 的区域中创建一个缓存存储帐户。
2. 如果目标存储帐户（主要区域中的原始存储帐户）不存在，则会创建一个新帐户。 分配的存储帐户名称是辅助 VM 使用的存储帐户名称，并带有后缀“asr”。
3. 如果 VM 使用托管磁盘，则主要区域中会创建副本托管磁盘以存储从辅助 VM 磁盘中复制的数据。
4. 如果目标可用性集不存在，将在执行重新保护作业期间根据需要新建一个可用性集。 如果已自定义重新保护设置，则会使用选定的可用性集。

触发重新保护作业时，如果目标 VM 存在，则会发生以下情况：

1. 关闭正在运行的目标端 VM。
2. 如果 VM 使用的是托管磁盘，则会使用“-ASRReplica”后缀创建原始磁盘的副本。 原始磁盘将被删除。 将使用“-ASRReplica”副本进行复制。
3. 如果 VM 使用的是非托管磁盘，则会分离目标 VM 的数据磁盘并将其用于复制。 将会创建 OS 磁盘的一个副本并将其附加到 VM 上。 将分离原始 OS 磁盘并将其用于复制。
4. 仅同步源磁盘与目标磁盘之间的更改。 将通过比较两个磁盘来计算差异，然后传输这些差异。 若要查找下面的估计的时间检查。
5. 在同步完成后，根据复制策略开始执行增量复制，并创建恢复点。

触发重新保护作业时，如果目标 VM 和磁盘不存在，则会发生以下情况：
1. 如果 VM 使用的是托管磁盘，则会使用“-ASRReplica”后缀创建副本磁盘。 将使用“-ASRReplica”副本进行复制。
2. 如果 VM 使用的是非托管磁盘，则会在目标存储帐户中创建副本磁盘。
3. 将全部磁盘从故障转移的区域复制到新的目标区域。
4. 在同步完成后，根据复制策略开始执行增量复制，并创建恢复点。

#### <a name="estimated-time--to-do-the-reprotection"></a>若要执行重新保护操作的估计的时间 

在大多数情况下，Azure Site Recovery 不会将完整的数据复制到源区域。 以下是确定将复制数据量的条件：

1.  如果源 VM 数据是已删除、 损坏或无法访问，因为某种原因等资源组更改/删除，然后在重新保护完成 IR 期间会因为没有要使用的源区域上没有可用数据。
2.  将源 VM 是否可访问仅差异备份是计算通过比较两个磁盘，然后将其传输。 检查下表以获取估计的时间 

|* * 示例情况 * * | * * 重新保护所花费的时间 * * |
|--- | --- |
|源区域具有 1 个虚拟机使用 1 TB 的标准磁盘<br/>-使用仅 127 GB 的数据和磁盘的其余部分为空<br/>的是 60 MiB/秒吞吐量的标准端口磁盘类型<br/>的故障转移后不更改任何数据| 45 分钟 – 1.5 小时的近似时间<br/> -在重新保护期间 Site Recovery 将填充整个这需要 127 GB 的数据的校验和 / 45 MBs ~ 45 分钟<br/>-某些系统开销时间是所必需的 Site Recovery，以执行自动为 20-30 分钟的小数位数<br/>-产生流出费用 |
|源区域具有 1 个虚拟机使用 1 TB 的标准磁盘<br/>-使用仅 127 GB 的数据和磁盘的其余部分为空<br/>的是 60 MiB/秒吞吐量的标准端口磁盘类型<br/>的故障转移后 45 GB 数据更改| 大概时间 1 小时-2 小时<br/>-在重新保护期间 Site Recovery 将填充整个这需要 127 GB 的数据的校验和 / 45 MBs ~ 45 分钟<br/>-传输时间，以应用更改为 45 GB 的 45 GB / 45 MBps ~ 17 分钟<br/>-仅为 45 GB 数据的校验和不是出口费用|
 



## <a name="next-steps"></a>后续步骤

保护 VM 后，可以启动故障转移。 故障转移过程会关闭次要区域中的 VM，在主要区域中创建并启动 VM，这会出现短暂的停机。 建议相应地选择一个时间，并在启动到主站点的完全故障转移之前运行测试故障转移。 [详细了解](site-recovery-failover.md)故障转移。
