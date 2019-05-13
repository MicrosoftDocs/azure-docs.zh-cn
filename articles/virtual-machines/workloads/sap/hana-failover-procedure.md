---
title: HANA （大型实例） 上的 SAP HANA 的灾难站点故障转移过程 |Microsoft Docs
description: 如何为 Azure （大型实例） 上的 SAP HANA 执行故障转移到灾难恢复站点
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
ms.openlocfilehash: f7d4f6216b4a57796ab5c0296713316dd97c47a8
ms.sourcegitcommit: 60606c5e9a20b2906f6b6e3a3ddbcb6c826962d6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/01/2019
ms.locfileid: "64987892"
---
# <a name="disaster-recovery-failover-procedure"></a>灾难恢复故障转移过程


>[!IMPORTANT]
>本文不能取代 SAP HANA 管理文档或 SAP 说明。 我们预计有扎实的了解和 SAP HANA 管理和操作，尤其是对于备份、 还原、 高可用性和灾难恢复 (DR) 方面的专业知识。 在本文中，会显示在 SAP HANA Studio 中的屏幕快照。 SAP 管理工具屏幕的内容、结构和性质以及工具本身可能会因不同的 SAP HANA 发行版本而有所差异。

有两种情况需要时可以故障转移到 DR 站点，请考虑：

- 需要将 SAP HANA 数据库返回到最新状态的数据。 在这种情况下，没有可以执行故障转移，无需联系 Microsoft 的自助服务的脚本。 故障回复，需要与 Microsoft 合作。
- 你想要还原到不是最新的复制的快照的存储快照。 对于这种情况，需要与 Microsoft 合作。 

>[!NOTE]
>必须在表示 DR 单元的 HANA 大型实例单元上完成以下步骤。 
 
若要还原到最新复制的存储快照，按照"执行完整的灾难恢复故障转移-azure_hana_dr_failover"中的步骤中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 

如果你想要有多个故障转移的 SAP HANA 实例，运行几次 azure_hana_dr_failover 命令。 请求时，输入你想要故障转移和还原 SAP HANA SID。 


您可以还测试灾难恢复故障转移，而不会影响实际的复制关系。 若要执行测试故障转移，请按照"执行测试 DR 故障转移-azure_hana_test_dr_failover"中的步骤中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。 

>[!IMPORTANT]
>不要*不*完成的过程在 DR 站点中创建的实例上运行任何生产事务**测试故障转移**。 命令 azure_hana_test_dr_failover 创建一组的卷与主站点没有任何关系。 结果是无法同步回主站点。 

如果你想要有多个 SAP HANA 实例，若要测试，多次运行该脚本。 请求时，输入想要测试故障转移实例的 SAP HANA SID。 

>[!NOTE]
>如果需要故障转移到 DR 站点，以修复一些几小时前已删除的数据，需要将 DR 卷设置为较早的快照，此过程适用。 

1. 关闭正在运行的 HANA 大型实例的灾难恢复单元上的 HANA 非生产实例。 已预安装 HANA 休眠生产实例。
1. 确保未运行任何 SAP HANA 进程。 使用以下命令执行此检查：

      `/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`。

      输出应显示 **hdbdaemon** 进程处于停止状态且没有其他 HANA 进程处于运行或已开始状态。
1. 确定希望将灾难恢复站点还原到的快照名称或 SAP HANA 备份 ID。 在真实的灾难恢复事例中，此快照通常是最新的快照。 如果需要恢复丢失的数据，请选择更早的快照。
1. 通过高优先级支持请求联系 Azure 支持部门。 请输入的名称与该快照还原和快照发布或在 DR 站点上的 HANA 备份 ID 的日期。 默认情况下，运营团队只会还原 /hana/data 卷。 如果你想要也有/hana/logbackups 卷，需要特意。 不要还原 /hana/shared 卷。 而应选择特定文件，例如 global.ini **.snapshot**及其子目录后你重新装载 PRD 的共享卷。 

   在操作端，将执行以下步骤：

   a. 停止生产卷到灾难恢复卷的快照复制。 如果生产站点服务中断是需要执行灾难恢复过程的原因，则此中断可能已经发生。
   
   b. 在灾难恢复卷上还原所选的存储快照或具有备份 ID 的快照。
   
   c. 还原后，灾难恢复卷可装载到灾难恢复区域中的 HANA 大型实例单元。
      
1. 将灾难恢复卷装载到灾难恢复站点中的 HANA 大型实例单元。 
1. 启动处于休眠状态的 SAP HANA 生产实例。
1. 如果您选择复制事务日志备份以缩短 RPO 时间，事务日志备份合并到新装载 DR/hana//hana/logbackups 目录中。 不要覆盖现有备份。 复制不使用存储快照的最新复制复制的较新备份。
1. 此外可以还原单个文件不复制到 DR Azure 区域中 /hana/shared/PRD 卷的快照。

以下步骤说明如何恢复基于还原的存储快照和事务日志备份所具备的 SAP HANA 生产实例。

1. 使用 SAP HANA Studio 将备份位置更改为 **/hana/logbackups**。

   ![更改 DR 恢复的备份位置](./media/hana-overview-high-availability-disaster-recovery/change_backup_location_dr1.png)

1. SAP HANA 会逐个扫描备份文件位置并建议要还原到的最新事务日志备份。 扫描可能需要几分钟直到如下所示，将显示以下屏幕：

   ![DR 恢复的事务日志备份的列表](./media/hana-overview-high-availability-disaster-recovery/backup_list_dr2.PNG)

1. 调整一些默认设置：

      - 清除“使用增量备份”。
      - 选择“初始化日志区域”。

   ![设置“初始化日志区域”](./media/hana-overview-high-availability-disaster-recovery/initialize_log_dr3.PNG)

1. 选择“完成”。

   ![完成 DR 还原](./media/hana-overview-high-availability-disaster-recovery/finish_dr4.PNG)

应显示如此处所示的进度窗口。 请记住，此为 SAP HANA 三节点横向扩展配置的灾难恢复还原示例。

![还原进度](./media/hana-overview-high-availability-disaster-recovery/restore_progress_dr5.PNG)

如果还原停止在响应**完成**屏幕上，并且不显示进度屏幕，确认辅助角色节点上的所有 SAP HANA 实例正在都运行。 如有必要，请手动启动 SAP HANA 实例。


## <a name="failback-from-a-dr-to-a-production-site"></a>从 DR 站点故障回复到生产站点
可以从 DR 站点故障回复到生产站点。 让我们探讨以下场景：到灾难恢复站点的故障转移是由生产 Azure 区域中的问题引起，而不是因为需要恢复丢失的数据。 

您已运行了 SAP 生产工作负荷一段时间在灾难恢复站点中。 随着生产站点中问题的解决，需要故障回复到生产站点。 由于不能丢失数据，因此回到生产站点这一过程涉及到几个步骤且需要与 Azure 上的 SAP HANA 运营团队进行紧密协作。 问题解决后，需要联系运营团队，以便开始同步回到生产站点。

执行以下步骤:

1. Azure 上的 SAP HANA 运营团队获取从灾难恢复存储卷同步生产存储卷（现表示生产状态）的触发指令。 在此状态中，生产站点中的 HANA 大型实例单元处于关闭状态。
1. Azure 运营团队上的 SAP HANA 监视复制，并可确保，它消除之前它们将通知你。
1. 需要关闭在灾难恢复站点使用 HANA 生产实例的应用程序。 然后执行 HANA 事务日志备份。 接下来，停止灾难恢复站点中 HANA 大型实例单元运行的 HANA 实例。
1. 关闭在灾难恢复站点中的 HANA 大型实例单元中运行的 HANA 实例后，运营团队再次手动同步磁盘卷。
1. Azure 运营团队上的 SAP HANA 再次启动生产站点中的 HANA 大型实例单元。 它们将它给你。 请确保，SAP HANA 实例处于关闭状态在 HANA 大型实例单元启动时。
1. 执行相同的数据库还原步骤时以前故障转移到灾难恢复站点。

## <a name="monitor-disaster-recovery-replication"></a>监视灾难恢复复制

若要监视存储复制过程的状态，请运行该脚本`azure_hana_replication_status`。 必须从运行在灾难恢复位置，以按预期方式工作单元运行此命令。 无论复制是否处于活动状态，该命令起作用。 可以为灾难恢复位置中租户的每个 HANA 大型实例单元运行该命令。 它不能用于获取有关启动卷的详细信息。 

该命令和输出的详细信息，请参阅"获取 DR 复制状态-azure_hana_replication_status"中[Microsoft Azure 上的 SAP HANA 快照工具](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.0/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.0.pdf)。


## <a name="next-steps"></a>后续步骤
- 请参阅[监视和故障排除从 HANA 端](hana-monitor-troubleshoot.md)。
