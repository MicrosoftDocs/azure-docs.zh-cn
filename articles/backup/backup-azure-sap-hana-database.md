---
title: 备份到 Azure 备份与 Azure 的 SAP HANA 数据库 |Microsoft Docs
description: 本教程介绍如何备份 SAP HANA 数据库备份到 Azure 与 Azure 备份服务。
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: raynew
ms.openlocfilehash: 5ed41013535e4591d88bff5c017c1fcf4c4053cc
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237802"
---
# <a name="back-up-an-sap-hana-database"></a>备份 SAP HANA 数据库

[Azure 备份](backup-overview.md)支持到 Azure 的 SAP HANA 数据库的备份。

> [!NOTE]
> 此功能目前处于公开预览状态。 它当前不是生产准备就绪后，但没有保证的 SLA。 


## <a name="scenario-support"></a>方案支持

**支持** | **详细信息**
--- | ---
**支持的地理区域** | 澳大利亚东南部、 澳大利亚东部 <br> 巴西南部 <br> 加拿大中部、 加拿大东部 <br> 南亚洲东部、 亚洲东部 <br> 美国东部、 美国东部 2、 美国中西部、 美国西部、 美国西部 2、 美国中北部、 美国中部、 美国中南部<br> 印度中部、 印度南部 <br> 日本东部、日本西部<br> 韩国中部、韩国南部 <br> 欧洲北部、欧洲西部 <br> 英国南部、 英国西部
**受支持的 VM 的操作系统** | SLES 12 SP2 或 SP3。
**支持的 HANA 版本** | HANA 上的 SSDC 1.x 中，在 HANA MDC 2.x < = SPS03


### <a name="current-limitations"></a>当前限制

- 仅可以备份 Azure Vm 上运行的 SAP HANA 数据库。
- 只能在 Azure 门户中配置的 SAP HANA 备份。 不能使用 PowerShell、 CLI 或 REST API 配置功能。
- 仅可以备份在纵向模式下的数据库。
- 你可以备份数据库日志每隔 15 分钟。 日志备份仅开始流动后成功的完整备份的数据库已完成。
- 可在创建完整和差异备份。 目前不支持增量备份。
- 应用的 SAP HANA 备份之后，无法修改备份策略。 如果想要备份的不同的设置，创建新的策略，或指定不同的策略。 
    - 若要创建新策略，请在保管库中单击**策略** > **备份策略** > **+ 添加** > **中的 SAP HANAAzure VM**，并指定策略设置。
    - 若要分配不同的策略，请运行数据库的虚拟机的属性在单击当前的策略名称。 然后在**备份策略**页可以选择要用于备份的不同策略。




## <a name="prerequisites"></a>必备组件

请确保您在配置备份之前执行以下操作：

1. 在 VM 上运行 SAP HANA 数据库安装正式的 Microsoft [.NET Core 运行时 2.1](https://dotnet.microsoft.com/download/linux-package-manager/sles/runtime-current)包。 请注意：
    - 只需**dotnet-运行时-2.1**包。 不需要**aspnetcore 运行时 2.1**。
    - 如果 VM 不具有 internet 访问、 镜像或适用于 dotnet-运行时-2.1 （和所有依赖 Rpm） 从 Microsoft 包源指定提供脱机缓存的页面。
    - 在包安装期间您可能需要指定一个选项。 如果是这样，指定**解决方案 2**。

        ![包安装选项](./media/backup-azure-sap-hana-database/hana-package.png)

2.  在 VM 上安装并启用从官方 SLES 包/媒体，如下所示使用 zypper，ODBC 驱动程序包：

    ``` 
    sudo zypper update
    sudo zypper install unixODBC
    ```
4.  允许连接从 VM 到 internet，以便它能够访问 Azure 中，如下面的过程中所述。


### <a name="set-up-network-connectivity"></a>设置网络连接

对于所有操作，SAP HANA VM 需要连接到 Azure 公共 IP 地址。 VM 操作 （数据库发现配置的备份、 计划备份、 还原恢复点和其他操作） 不能正常连接。 通过允许访问 Azure 数据中心 IP 范围建立连接： 

- 您可以下载[IP 地址范围](https://www.microsoft.com/download/details.aspx?id=41653)的 Azure 数据中心，然后允许访问这些 IP 地址。
- 如果使用网络安全组 (Nsg)，则可以使用 AzureCloud[服务标记](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)若要允许所有 Azure 公共 IP 地址。 可以使用[Set-azurenetworksecurityrule cmdlet](https://docs.microsoft.com/powershell/module/servicemanagement/azure/set-azurenetworksecurityrule?view=azuresmps-4.0.0)修改 NSG 规则。

## <a name="onboard-to-the-public-preview"></a>载入到公共预览版

载入到公共预览版中，如下所示：

- 在门户中，注册到恢复服务服务提供商通过你的订阅 ID[下面的文章](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-register-provider-errors#solution-3---azure-portal)。 
- 对于 PowerShell，运行此 cmdlet。 它应为"已注册"完成。

    ```
    PS C:>  Register-AzProviderFeature -FeatureName "HanaBackup" –ProviderNamespace Microsoft.RecoveryServices
    ```



[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-the-databases"></a>发现数据库


1. 在保管库中**Getting Started**，单击**备份**。 在中**工作负荷的运行位置？**，选择**在 Azure VM 中的 SAP HANA**。
2. 单击**启动发现**。 这会启动保管库区域中不受保护的 Linux Vm 的发现。

   - 发现之后，未受保护的 Vm 会显示在门户中，按列出名称和资源组。
   - 如果 VM 未列出按预期方式，请，检查是否已备份保管库中。
   - 多个 Vm 可以具有相同的名称，但它们属于不同的资源组。

3. 在中**选择虚拟机**，单击链接以下载提供了针对 Azure 备份服务来访问数据库发现 SAP HANA Vm 的权限的脚本
4. 托管你想要备份的 SAP HANA 数据库每个 VM 上运行该脚本。
5. 在 Vm 上运行该脚本后**选择虚拟机**，选择的 Vm。 然后单击**发现 Db**。
6. Azure 备份发现 VM 上的所有 SAP HANA 数据库。 在发现期间 Azure 备份 VM 注册到保管库，并会将扩展安装在 VM 上。 在数据库上不安装任何代理。

    ![发现 SAP HANA 数据库](./media/backup-azure-sap-hana-database/hana-discover.png)
    
## <a name="configure-backup"></a>配置备份  

现在启用备份。

1. 在步骤 2 中，单击**配置备份**。
2. 在中**选择要备份的项**，选择你想要保护的所有数据库 >**确定**。
3. 在中 **备份策略** > **选择备份策略**，创建新的备份策略的数据库，根据下面的说明。
4. 在创建策略后**备份**菜单上，单击**启用备份**。
5. 备份配置中跟踪进度 **通知** 门户的区域。

### <a name="create-a-backup-policy"></a>创建备份策略
备份策略定义时创建备份，并保留多长时间。

- 策略是在保管库级别创建的。
- 多个保管库可以使用相同的备份策略，但必须向每个保管库应用该备份策略。

指定的策略设置，如下所示：

1. 在“策略名称”处输入新策略的名称。
2. 在“完整备份策略”中选择“备份频率”，然后选择“每日”或“每周”。
   - **每日**：选择的小时和备份作业开始时所在的时区。
   
       - 您必须运行完整备份。 不能关闭此选项。
       - 单击“完整备份”以查看策略。
       - 对于每日完整备份，无法创建差异备份。
       
   - **每周**:选择一周、 小时和备份作业运行所在的时区的一天。
3. 在中**期**，配置完整备份的保持期设置。
    - 默认情况下会选择所有选项。 清除不想要使用，并将您这样做的那些设置任何保留期范围限制。
    - 任何类型的备份 （完整/差异/日志） 的最小保持期为 7 天。
    - 恢复点已根据其保留范围标记为保留。 例如，如果选择每日完整备份，则每天只触发一次完整备份。
    - 标记并保留特定日期基于备份的每周的保持期和设置。
    - 每月和每年保留范围的行为类似。

4. 在中**完整备份策略**菜单上，单击**确定**接受的设置。
5. 选择**差异备份**添加差异的策略。
6. 在“差异备份策略”中，选择“启用”打开频率和保留控件。
    - 每天最多可以触发一次差异备份。
    - 差异备份最多可以保留 180 天。 如果需要保留更长时间，必须使用完整备份。

    > [!NOTE]
    > 目前不支持增量备份。 

7. 单击**确定**以保存策略并返回到主**备份策略**菜单。
8. 选择**日志备份**添加事务日志备份策略
    - 在中**日志备份**，选择**启用**。
    - 设置频率和保留控制。

    > [!NOTE]
    > 日志备份仅开始流动后完成成功的完整备份。

9. 单击**确定**以保存策略并返回到主**备份策略**菜单。
10. 定义备份策略后，请单击**确定**。


## <a name="run-an-on-demand-backup"></a>运行按需备份

根据策略计划运行备份。 可以运行按需备份，如下所示：


1. 在保管库菜单中，单击“备份项”。
2. 在中**备份项**，选择运行 SAP HANA 数据库的虚拟机，然后单击**立即备份**。
3. 在“立即备份”中，使用日历控件选择恢复点的最后保留日期。 然后单击“确定”。
4. 监视门户通知。 你可以监视作业进度，在保管库仪表板 >**备份作业**> 正在进行中。 根据数据库的大小，创建初始备份可能需要一段时间。

## <a name="run-sap-hana-studio-backup-on-a-database-with-azure-backup-enabled"></a>使用 Azure 备份已启用的数据库上运行 SAP HANA Studio 备份

如果你想要进行本地备份 （使用 HANA Studio） 正在备份的数据库使用 Azure 备份，请执行以下操作：

1. 等待任何完全或要完成的数据库的日志备份。 检查 SAP HANA Studio 中的状态。
2. 禁用日志备份，并将备份的目录设置为相关数据库的文件系统。
3. 若要执行此操作，请双击**systemdb** > **配置** > **选择数据库** > **筛选器 （日志）**.
4. 设置**enable_auto_log_backup**到**否**。
5. 设置**log_backup_using_backint**到**False**。
6. 执行临时完整备份的数据库。
7. 等待完整备份和目录备份来完成。
8. 还原以前的设置还原到 azure:
    - 设置**enable_auto_log_backup**到**是**。
    - 设置**log_backup_using_backint**到**True**。



## <a name="next-steps"></a>后续步骤

[了解有关](backup-azure-arm-vms-prepare.md)备份 Azure Vm。


