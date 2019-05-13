---
title: 关于用于将 Hyper-V VM 灾难恢复到 Azure 的 Azure Site Recovery 部署规划器 | Microsoft Docs
description: 了解用于将 Hyper-V 灾难恢复到 Azure 的 Azure Site Recovery 部署规划器。
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/18/2019
ms.author: mayg
ms.openlocfilehash: a793de302eb0833c959f2486fc2cda1f2eec4674
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149112"
---
# <a name="about-the-azure-site-recovery-deployment-planner-for-hyper-v-disaster-recovery-to-azure"></a>关于用于将 Hyper-V 灾难恢复到 Azure 的 Azure Site Recovery 部署规划器

本文为适用于 Hyper-V 到 Azure 生产部署的 Azure Site Recovery 部署规划器用户指南。

在开始使用 Site Recovery 保护任何 Hyper-V 虚拟机 (VM) 之前，请根据每日数据更改率分配足够的带宽以满足所需的恢复点目标 (RPO)，并在本地 Hyper-V 存储的每个卷上分配足够的可用存储空间。

此外还需创建适当类型和数量的目标 Azure 存储帐户。 考虑到使用量会随着时间的推移而增加，从而导致源生产服务器数目的增加，请创建标准或高级存储帐户。 选择每个 VM 的存储类型时，需要考虑到工作负荷特征，例如每秒读/写 I/O 操作数 (IOPS) 或数据变动量，以及 Azure Site Recovery 限制。 

Azure Site Recovery 部署规划器是一个命令行工具，适用于 Hyper-V 到 Azure 和 VMware 到 Azure 灾难恢复方案。 可以使用此工具远程分析多个 Hyper-V 主机上存在的 Hyper-V VM（不会给生产造成任何影响），了解成功进行复制和测试性故障转移/故障转移所需满足的带宽和 Azure 存储要求。 不需在本地安装任何 Azure Site Recovery 组件即可运行此工具。 但是，为了获取准确的已实现吞吐量结果，建议在 Windows Server 上运行此规划器，该 Server 的硬件配置应与用来启用灾难恢复保护（通过恢复到 Azure 的方式进行保护）的某个 Hyper-V 服务器相同。 

该工具提供以下详细信息：

**兼容性评估**

* 根据磁盘数量、磁盘大小、IOPS、变动量和一些 VM 特征评估 VM 的合格性。

**网络带宽需求与 RPO 评估**

* 增量复制所需的估计网络带宽
* Azure Site Recovery 可以获得的从本地到 Azure 的吞吐量
* 给定带宽可以实现的 RPO
* 在预配较低带宽的情况下，对所需 RPO 的影响。

    
**Azure 基础结构要求**

* 每个 VM 的存储类型（标准或高级存储帐户）要求
* 要为复制设置的标准和高级存储帐户总数
* 基于 Azure 存储指南的存储帐户命名建议
* 适用于所有 VM 的存储帐户归置
* 针对订阅执行测试性故障转移或故障转移之前要设置的 Azure 核心数
* 适用于每个本地 VM 的 Azure VM 建议大小

**本地基础结构要求**
* 在 Hyper-V 存储的每个卷上成功地进行初始复制和增量复制所需的可用存储空间，可确保 VM 复制不会导致任何会对生产应用程序造成负面影响的停机
* 需要为 Hyper-V 复制设置的最大复制频率

**初始复制批处理指南** 
* 要用于保护的 VM 批数
* 每个批次中的 VM 的列表
* 对每个批次进行保护的顺序
* 完成每个批次的初始复制所需的估计时间

**DR 到 Azure 的估算成本**
* DR 到 Azure 的总估算成本：计算、存储、网络和 Azure Site Recovery 许可证成本
* 每个 VM 的详细成本分析



>[!IMPORTANT]
>
>由于使用量可能会随着时间的推移而增加，所有上述工具计算在执行时都会假定在工作负荷特征中存在一个 30% 的增长系数，而且所有分析指标（读/写 IOPS、变动量等）的值都会使用 95%。 这两个元素（增长系数和百分位数计算）均可配置。 若要详细了解增长系数，请参阅“增长系数考虑因素”部分。 若要详细了解百分位值，请参阅“用于计算的百分位值”部分。
>

## <a name="support-matrix"></a>支持矩阵

| | **VMware 到 Azure** |**Hyper-V 到 Azure**|**Azure 到 Azure**|**Hyper-V 到辅助站点**|**VMware 到辅助站点**
--|--|--|--|--|--
支持的方案 |是|是|否|是*|否
支持的版本 | vCenter 6.7、 6.5、 6.0 或 5.5| Windows Server 2016、Windows Server 2012 R2 | NA |Windows Server 2016、Windows Server 2012 R2|NA
支持的配置|vCenter、ESXi| Hyper-V 群集、Hyper-V 主机|NA|Hyper-V 群集、Hyper-V 主机|NA|
可以按 Azure Site Recovery 部署规划器的运行实例进行分析的服务器数 |单个（一次只能分析属于一个 vCenter Server 或一个 ESXi 服务器的 VM）|多个（一次可以分析多个主机或主机群集的 VM）| NA |多个（一次可以分析多个主机或主机群集的 VM）| NA

*此工具主要用于 Hyper-V 到 Azure 灾难恢复方案。 对于 Hyper-V 到辅助站点灾难恢复，只能将其用于了解源端建议，例如所需网络带宽、每个源 Hyper-V 服务器上需要的可用存储空间，以及初始复制批处理数和批次定义。  忽略报表中的 Azure 建议和成本。 另外，“获取吞吐量”操作不适用于 Hyper-V 到辅助站点灾难恢复方案。

## <a name="prerequisites"></a>必备组件
此工具有三个针对 Hyper-V 的主要阶段：获取 VM 列表、分析，以及生成报表。 此外还有第死个选项，即仅计算吞吐量。 下表中显示了需要在其上执行不同阶段的服务器的要求：

| 服务器要求 | 描述 |
|---|---|
|获取 VM 列表、分析和吞吐量测量 |<ul><li>操作系统：Microsoft Windows Server 2016 或 Microsoft Windows Server 2012 R2 </li><li>计算机配置：8 个 vCPU、16 GB RAM、300 GB HDD</li><li>[Microsoft .NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[Microsoft Visual C++ Redistributable for Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>可在此服务器中通过 Internet 访问 Azure</li><li>Azure 存储帐户</li><li>服务器上的管理员访问权限</li><li>至少 100 GB 的可用磁盘空间（假定有 1000 个 VM，每个平均包含 3 个磁盘，分析时间为 30 天）</li><li>必须将从其运行 Azure Site Recovery 部署规划器工具的 VM 添加到包含所有 Hyper-V 服务器的 TrustedHosts 列表。</li><li>必须将要分析的所有 Hyper-V 服务器都添加到要运行该工具的客户端 VM 的 TrustedHosts 列表中。 [详细了解如何将服务器添加到 TrustedHosts 列表中](#steps-to-add-servers-into-trustedhosts-list)。 </li><li> 应从客户端上的 PowerShell 或命令行控制台使用管理特权运行此工具</ul></ul>|
| 报告生成 | 装有 Microsoft Excel 2013 或更高版本的 Windows 电脑或 Windows Server |
| 用户权限 | 在“获取 VM 列表”和“分析”操作过程中用于访问 Hyper-V 群集/Hyper-V 主机的管理员帐户。<br>所有需要进行分析的主机都应有一个使用相同凭据（即用户名和密码）的域管理员帐户
 |

## <a name="steps-to-add-servers-into-trustedhosts-list"></a>将服务器添加到 TrustedHosts 列表中的步骤
1.  要从其部署此工具的 VM 应该让需要分析的所有主机都位于其 TrustedHosts 列表中。 若要将客户端添加到 Trustedhosts 列表中，请在 VM 上通过提升的 PowerShell 运行以下命令。 VM 可以是 Windows Server 2012 R2 或 Windows Server 2016。 

            set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate

1.  每个需要分析的 Hyper-V 主机都应：

    a. 有一个 VM，以便其上的此工具能够在其 TrustedHosts 列表中运行。 从 Hyper-V 主机上权限提升的 PowerShell 运行以下命令。

            set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate

    b. 已启用 PowerShell 远程功能。

            Enable-PSRemoting -Force

## <a name="download-and-extract-the-deployment-planner-tool"></a>下载和提取部署规划器工具

1.  下载最新版本的 [Azure Site Recovery Deployment Planner](https://aka.ms/asr-deployment-planner)。
该工具已打包到 .zip 文件夹中。 同一工具支持 VMware 到 Azure 与 Hyper-V 到 Azure 灾难恢复方案。 也可将此工具用于 Hyper-V 到辅助站点灾难恢复方案，但请忽略报表中的 Azure 基础结构建议。

1.  将 .zip 文件夹复制到要在其上运行该工具的 Windows Server。 可以在 Windows Server 2012 R2 或 Windows Server 2016 上运行此工具。 服务器必须具有相应的网络访问权限，能够连接到 Hyper-V 群集或 Hyper-V 主机，其中有需要分析的 VM。 建议将要运行此工具的 VM 的硬件配置设置为与需要保护的 Hyper-V 服务器的硬件配置相同。 此类配置可确保该工具所报告的已实现吞吐量与 Azure Site Recovery 在复制过程中能够达到的实际吞吐量相符。 吞吐量计算取决于服务器上的可用网络带宽和服务器的硬件配置（CPU、存储等）。 计算的是从运行此工具的服务器到 Azure 的吞吐量。 如果该服务器的硬件配置不同于 Hyper-V 服务器的硬件配置，该工具所报告的已实现吞吐量将不准确。
VM 的推荐配置：8 个 vCPU、16 GB RAM、300 GB HDD。

1.  解压缩 .zip 文件夹。
该文件夹包含多个文件和子文件夹。 可执行文件是父文件夹中的 ASRDeploymentPlanner.exe。

示例：将 .zip 文件复制到 E:\ 驱动器并将它解压缩。 E:\ASR Deployment Planner_v2.3.zip

E:\ASR Deployment Planner_v2.3\ASRDeploymentPlanner.exe

### <a name="updating-to-the-latest-version-of-deployment-planner"></a>更新至最新版本的部署规划器

最新的更新汇总了部署规划器[版本历史记录](site-recovery-deployment-planner-history.md)。

如果部署规划器为旧版，请执行以下操作之一：
 * 如果最新版不包含分析修补程序且分析已在当前版本的 Planner 上进行，请继续该分析。
 * 如果最新版本不包含分析修补程序，则建议停止在当前版本上进行分析，使用新版本重新开始分析。


  >[!NOTE]
  >
  >开始使用新版本进行分析以后，请传递相同的输出目录路径，以便工具将配置文件数据追加到现有文件。 将使用完整的分析数据集来生成报告。 如果传递不同的输出目录，则会创建新文件，生成报告时不能使用旧的分析数据。
  >
  >每个新的 Deployment Planner 都是 .zip 文件的累积更新。 不需将最新文件复制到旧文件夹。 可以创建和使用新文件夹。

## <a name="version-history"></a>版本历史记录
最新的 Azure Site Recovery 部署规划器工具版本为 2.4。
请参阅[Azure 站点恢复部署规划器版本历史记录](https://social.technet.microsoft.com/wiki/contents/articles/51049.asr-deployment-planner-version-history.aspx)了解每个更新中增加的修补程序页。


## <a name="next-steps"></a>后续步骤
* [运行部署规划器](site-recovery-hyper-v-deployment-planner-run.md)。
