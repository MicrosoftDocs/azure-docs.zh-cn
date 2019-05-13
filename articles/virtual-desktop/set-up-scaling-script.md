---
title: 自动缩放 Windows 虚拟桌面预览会话主机-Azure
description: 介绍如何为 Windows 虚拟桌面预览会话主机设置自动缩放脚本。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 03/21/2019
ms.author: helohr
ms.openlocfilehash: 7687abf5fc4af0eea9fa6aa210cfd6734cec2b36
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410573"
---
# <a name="automatically-scale-session-hosts"></a>自动缩放会话主机

对于在 Azure 中的许多 Windows 虚拟桌面预览部署，虚拟机成本表示总体的 Windows 虚拟桌面部署成本的重要部分。 为了降低成本，最好是关闭的情况下并释放会话主机虚拟机 (Vm) 在非高峰使用时间，然后重新启动它们在高峰使用情况。

本文使用一个简单的缩放脚本来自动缩放 Windows 虚拟桌面环境中的会话主机虚拟机。 若要了解有关缩放的脚本的工作原理的详细信息，请参阅[缩放脚本的工作方式](#how-the-scaling-script-works)部分。

## <a name="prerequisites"></a>必备组件

运行脚本的环境必须具有以下操作：

- Windows 虚拟桌面租户和帐户或查询 （如 RDS 参与者） 该租户的权限的服务主体。
- 会话主机池 Vm 配置和注册到 Windows 虚拟机服务。
- 其他虚拟机，通过任务计划程序运行计划的任务，并且具有对会话主机的网络访问。 这将作为 scaler VM 是 reffered 到文档中的更高版本。
- [Microsoft Azure 资源管理器 PowerShell 模块](https://docs.microsoft.com/powershell/azure/azurerm/install-azurerm-ps)运行计划的任务在 VM 上安装。
- [Windows 虚拟桌面 PowerShell 模块](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview)运行计划的任务在 VM 上安装。

## <a name="recommendations-and-limitations"></a>建议和限制

运行时缩放的脚本，请记住以下事项：

- 此缩放脚本只能处理一个主机池正在缩放脚本的计划任务的每个实例。
- 运行缩放脚本的计划的任务必须始终在 VM 上。
- 创建每个实例缩放脚本和其配置单独的文件夹。
- 此脚本不支持在登录到 Windows 虚拟桌面管理员使用要求多重身份验证的 Azure AD 用户帐户。 我们建议使用服务主体来访问的 Windows 虚拟机服务和 Azure。 请按照[本教程](create-service-principal-role-powershell.md)如何使用 PowerShell 创建服务主体和角色分配。
- Azure 的 SLA 保证仅适用于在可用性集中的 Vm。 当前版本的文档描述了使用单个 VM 执行缩放，这可能不符合可用性需求的环境。

## <a name="deploy-the-scaling-script"></a>部署缩放脚本

以下过程将告诉您如何部署缩放脚本。

### <a name="prepare-your-environment-for-the-scaling-script"></a>准备您的环境的缩放脚本

首先，准备好的缩放脚本的环境：

1. 登录到虚拟机 (scaler VM) 将使用域管理帐户运行计划的任务。
2. Scaler 来保存缩放脚本和其配置的 VM 上创建一个文件夹 (例如， **c:\\缩放 HostPool1**)。
3. 下载**basicScale.ps1**， **Config.xml**，并**函数 PSStoredCredentials.ps1**文件，以及**PowershellModules**从文件夹[缩放脚本存储库](https://github.com/Azure/RDS-Templates/tree/master/wvd-sh/WVD%20scaling%20script)并将其复制到在步骤 2 中创建的文件夹。 有两种主要方式，然后将其复制到 scaler VM 获取的文件：
    - Git 存储库克隆到本地计算机。
    - 视图**Raw**版本的每个文件，复制并粘贴到文本编辑器中，每个文件的内容，然后使用相应的文件名和文件类型保存文件。 

### <a name="create-securely-stored-credentials"></a>创建安全存储的凭据

接下来，你将需要创建安全存储的凭据：

1. 打开 PowerShell ISE，以管理员身份。
2. 通过运行以下 cmdlet 导入的 RDS PowerShell 模块：

    ```powershell
    Install-Module Microsoft.RdInfra.RdPowershell
    ```
    
3. 打开编辑窗格和负载**函数 PSStoredCredentials.ps1**文件。
4. 运行以下 cmdlet：
    
    ```powershell
    Set-Variable -Name KeyPath -Scope Global -Value <LocalScalingScriptFolder>
    ```
    
    例如， **Set-variable-名称 KeyPath-全局作用域的值"c:\\缩放 HostPool1"**
5. 运行**新建 StoredCredential KeyPath \$KeyPath** cmdlet。 出现提示时，输入有权查询主机池虚拟桌面 Windows 凭据 (在指定主机池**config.xml**)。
    - 如果使用不同的服务主体或标准帐户，请运行**新建 StoredCredential KeyPath \$KeyPath** cmdlet 后对于每个帐户创建本地存储的凭据。
6. 运行**Get StoredCredentials-列表**以确认已成功创建凭据。

### <a name="configure-the-configxml-file"></a>配置 config.xml 文件

若要更新在 config.xml 中的缩放脚本设置以下字段中输入相关的值：

| 字段                     | 描述                    |
|-------------------------------|------------------------------------|
| AADTenantId                   | 将会话主机 Vm 的运行位置的订阅相关联的 azure AD 租户 ID     |
| AADApplicationId              | 服务主体应用程序 ID                                                       |
| AADServicePrincipalSecret     | 这可以输入在测试阶段，但要创建具有凭据后，必须保留空**函数 PSStoredCredentials.ps1**    |
| currentAzureSubscriptionId    | 会话在其中托管 Vm 的 Azure 订阅的 ID 运行                        |
| tenantName                    | Windows 虚拟桌面租户名称                                                    |
| hostPoolName                  | Windows 虚拟机主机池名称                                                 |
| RDBroker                      | WVD 服务 URL 默认值 https:\//rdbroker.wvd.microsoft.com             |
| 用户名                      | （它是可能如下所示 AADApplicationId 有相同的服务主体） 的服务主体应用程序 ID 或标准用户，而无需多重身份验证 |
| isServicePrincipal            | 接受的值是 **，则返回 true**或**false**。 指示是否正在使用的凭据的第二个集是服务主体或标准帐户。 |
| BeginPeakTime                 | 高峰期开始时                                                            |
| EndPeakTime                   | 当高峰期结束时                                                              |
| TimeDifferenceInHours         | 本地时间和 UTC，以小时为单位的时间差异                                   |
| SessionThresholdPerCPU        | 每个 CPU 阈值用于确定何时需要新的会话主机 VM 在高峰时间要启动的会话的最大数目。  |
| MinimumNumberOfRDSH           | 主机池 Vm 保持运行在非高峰使用期间的最小数目             |
| LimitSecondsToForceLogOffUser | 若要强制用户注销之前等待的秒数。如果设置为 0，用户不会被强制进行注销。  |
| LogOffMessageTitle            | 它们强制注销之前向用户发送消息的标题                  |
| LogOffMessageBody             | 正在注销之前向用户发送警告消息的正文。例如，"此计算机将关闭 X 分钟列表中的。 请保存您的工作并注销。" |

### <a name="configure-the-task-scheduler"></a>配置任务计划程序

在配置配置.xml 文件之后, 将需要配置任务计划程序在一定的时间间隔运行 basicScaler.ps1 文件。

1. 启动**任务计划程序**。
2. 在中**任务计划程序**窗口中，选择**创建任务...**
3. 在中**创建任务**对话框中，选择**常规**选项卡上，输入**名称**（例如，"动态 RDSH"），选择**运行或不用户是否登录**并**使用最高权限运行**。
4. 转到**触发器**选项卡，然后选择**新建...**
5. 在**新触发器**对话框下**高级设置**，检查**任务重复每个**，然后选择适当的时间段和持续时间 (例如， **15分钟**或**无限期地**)。
6. 选择**操作**选项卡和**新建...**
7. 在中**新的操作**对话框中，输入**powershell.exe**到**程序/脚本**字段，然后输入**c:\\缩放\\RDSScaler.ps1**成**添加参数 （可选）** 字段。
8. 转到**条件**并**设置**选项卡，然后选择**确定**接受每个的默认设置。
9. 输入你打算运行缩放脚本的管理帐户的密码。

## <a name="how-the-scaling-script-works"></a>缩放脚本的工作方式

此缩放脚本从 config.xml 文件，其中包括的开始和结束的高峰使用周期在一天内读取设置。

在高峰使用期间，该脚本检查当前会话数和每个主机池的当前正在运行 RDSH 容量。 它会计算正在运行的会话主机 Vm 是否有足够的容量来支持基于 SessionThresholdPerCPU 参数 config.xml 文件中定义的现有会话。 如果没有，则脚本会启动主机池中其他会话主机 Vm。

在非高峰使用期间，此脚本确定 Vm 应关闭基于 MinimumNumberOfRDSH 参数在 config.xml 文件中的会话主机。 该脚本将设置会话以释放模式，阻止新的会话连接到主机的主机 Vm。 如果您设置**LimitSecondsToForceLogOffUser**为非零正值的 config.xml 文件中的参数，该脚本将通知任何当前已登录用户保存工作，等待已配置的时间，并然后强制若要注销的用户。一旦所有用户会话已都签核会话主机 VM 上，该脚本将关闭服务器。

如果您设置**LimitSecondsToForceLogOffUser**为零的 config.xml 文件中的参数，该脚本将允许在主机中的会话配置设置池属性，以处理签名关闭用户会话。 如果在会话主机 VM 上有任何会话，它将保留会话主机 VM 运行。 如果不存在任何会话，该脚本会关闭会话主机 VM。

此脚本旨在使用任务计划程序在 scaler VM 服务器上定期运行。 选择适当的时间间隔基于远程桌面服务环境的大小并记住启动和关闭虚拟机可能需要一些时间。 我们建议运行缩放脚本每隔 15 分钟。

## <a name="log-files"></a>日志文件

缩放脚本将创建两个日志文件， **WVDTenantScale.log**并**WVDTenantUsage.log**。 **WVDTenantScale.log**文件将记录的事件和错误 （如果有） 在缩放脚本每次执行过程。

**WVDTenantUsage.log**文件将记录活动的核心数和虚拟机的活动数每次在执行缩放脚本。 可以使用此信息来估计 Microsoft Azure 虚拟机和成本的实际使用量。 文件格式作为以逗号分隔值与每个项包含以下信息：

>时间、 主机池、 内核，虚拟机

此外可以修改的文件的名称具有.csv 扩展名，加载到 Microsoft Excel 和分析。
