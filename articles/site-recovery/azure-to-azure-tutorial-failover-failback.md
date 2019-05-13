---
title: 使用 Azure Site Recovery 服务对复制到 Azure 次要区域的 Azure VM 进行故障转移和重新保护，以便实现灾难恢复。
description: 了解如何使用 Azure Site Recovery 服务对复制到 Azure 次要区域的 Azure VM 进行故障转移和重新保护，以便实现灾难恢复。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 04/29/2019
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 95e4e3f3acc52c230405f0c0cc4a05b03b21a386
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153786"
---
# <a name="fail-over-and-reprotect-azure-vms-between-regions"></a>在区域之间故障转移和重新保护 Azure VM

本教程介绍如何使用 [Azure Site Recovery](site-recovery-overview.md) 服务将 Azure 虚拟机 (VM) 故障转移到次要 Azure 区域。 故障转移后，对 VM 重新进行保护。 本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 故障转移 Azure VM
> * 重新保护辅助 Azure VM，以便它复制到主要区域。

> [!NOTE]
> 本教程包含的最简单路径带有默认设置和最小自定义项。 如需更复杂的方案，请参阅 Azure VM 的“操作指南”下的文章。


## <a name="prerequisites"></a>先决条件

- 开始之前，请查看有关故障转移的[常见问题解答](site-recovery-faq.md#failover)。
- 确保已完成[灾难恢复演练](azure-to-azure-tutorial-dr-drill.md)，检查所有内容是否都按预期工作。
- 在运行测试故障转移之前，验证 VM 属性。 该 VM 必须符合 [Azure 要求](azure-to-azure-support-matrix.md#replicated-machine-operating-systems)。

## <a name="run-a-failover-to-the-secondary-region"></a>运行到次要区域的故障转移

1. 在“已复制项”中，选择要故障转移的 VM >“故障转移”

   ![故障转移](./media/azure-to-azure-tutorial-failover-failback/failover.png)

2. 在“故障转移”中，选择要故障转移到的“恢复点”。 可以使用以下选项之一：

   * **最新**（默认值）：处理 Site Recovery 服务中的所有数据，并提供最低的恢复点目标 (RPO)。
   * **最新处理**：将虚拟机还原到已由 Site Recovery 服务处理的最新恢复点。
   * **自定义**：故障转移到特定的恢复点。 此选项可用于执行测试故障转移。

3. 如果希望 Site Recovery 在触发故障转移之前尝试关闭源 VM，请选择“在开始故障转移前关闭计算机”。 即使关机失败，故障转移也仍会继续。 Site Recovery 不在故障转移之后清理源。

4. 在“作业”页上跟踪故障转移进度。

5. 故障转移后，请登录到虚拟机来对它进行验证。 如果想要转到虚拟机的另一个恢复点，可以使用“更改恢复点”选项。

6. 如果故障转移后的虚拟机符合要求，可以**提交**故障转移。
   提交会删除该服务提供的所有恢复点。 现在无法无法更改恢复点。

> [!NOTE]
> 当你将启用 VM 复制后添加磁盘的 VM 进行故障转移时，复制点会显示可用于恢复的磁盘。 例如，如果 VM 有一个磁盘，你添加了一个新的磁盘，则在添加磁盘之前创建的复制点会显示复制点包含“1 个磁盘（共 2 个）”。

![使用添加的磁盘进行故障转移](./media/azure-to-azure-tutorial-failover-failback/failover-added.png)

## <a name="reprotect-the-secondary-vm"></a>重新保护辅助 VM

故障转移 VM 之后，需要重新保护它，以便它复制回主要区域。

1. 确保 VM 处于“故障转移已提交”状态，检查主要区域是否可用，并且你是否能够在其中创建和访问新资源。
2. 在“保管库” > “已复制项”中，右键单击已故障转移的 VM，并选择“重新保护”。

   ![右键单击以重新保护](./media/azure-to-azure-tutorial-failover-failback/reprotect.png)

2. 验证是否已选择保护方向（从次要区域到主要区域）。
3. 查看“资源组、网络、存储和可用性集”信息。 任何标记为“新”的资源将在重新保护操作中创建。
4. 单击“确定”触发重新保护作业。 此作业使用最新数据植入目标站点。 然后，它将增量复制到主要区域。 该 VM 现在处于受保护状态。

## <a name="next-steps"></a>后续步骤
- 进行重新保护以后，[了解如何](azure-to-azure-tutorial-failback.md)在主要区域可用时故障回复到主要区域。
- [详细了解](azure-to-azure-how-to-reprotect.md#what-happens-during-reprotection)重新保护工作流。
