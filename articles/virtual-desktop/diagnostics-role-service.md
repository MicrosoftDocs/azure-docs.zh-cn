---
title: 识别的 Windows 虚拟桌面预览诊断功能的 Azure 问题
description: 介绍 Windows 虚拟桌面预览诊断功能以及如何使用它。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 03/21/2019
ms.author: helohr
ms.openlocfilehash: 747e177b0fbbfb9049959c3194ee39c3234bba50
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65234029"
---
# <a name="identify-issues-with-the-diagnostics-feature"></a>确定诊断功能的问题

Windows 虚拟桌面预览提供了诊断功能，允许管理员确定通过单个界面的问题。 每当用户与系统交互时，Windows 虚拟机角色日志诊断活动。 每个日志包含相关信息，例如事务、 错误消息、 租户信息和用户信息中涉及的 Windows 虚拟机角色。 诊断活动创建的最终用户和管理操作，并可以分为三个主要的存储桶：

* 源订阅活动： 最终用户触发这些活动，只要它们尝试连接到通过 Microsoft 远程桌面应用程序及其数据源。
* 连接活动： 每当用户尝试通过 Microsoft 远程桌面应用程序连接到桌面或 RemoteApp，最终用户触发这些活动。
* 管理活动： 管理员执行系统，如创建主机池、 将用户分配到应用组，以及创建角色分配的管理操作时将触发这些活动。
  
不会到达 Windows 虚拟桌面的连接不会显示在诊断结果因为诊断角色服务本身是 Windows 虚拟机的一部分。 Windows 虚拟桌面连接问题可能在当最终用户遇到网络连接问题。

若要开始，[下载并导入的 Windows 虚拟桌面 PowerShell 模块](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview)如果尚未在 PowerShell 会话中使用。

## <a name="diagnose-issues-with-powershell"></a>诊断问题使用 PowerShell

Windows 虚拟桌面诊断程序将使用只是一个 PowerShell cmdlet，但包含许多可选参数，以帮助缩小和隔离问题。 以下各节列出的 cmdlet 可以运行以诊断问题。 可同时应用大多数筛选器。 列出在方括号中，如值`<tenantName>`，应替换为适用于您的具体情况的值。

### <a name="retrieve-diagnostic-activities-in-your-tenant"></a>检索你的租户中的诊断活动

可以通过输入检索诊断活动**Get RdsDiagnosticActivities** cmdlet。 下面的示例 cmdlet 将返回一系列诊断活动，按从最具体到最早。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName>
```

与其他 Windows 虚拟桌面 PowerShell cmdlet，必须使用 **-TenantName**参数来指定你想要用于查询的租户的名称。 租户名称是适用于几乎所有诊断活动查询。

### <a name="retrieve-detailed-diagnostic-activities"></a>检索详细的诊断活动

**-详细**参数为返回每个诊断活动提供更多详细信息。 每个活动的格式因其活动类型而异。 **-详细**参数可以添加到任何**Get RdsDiagnosticActivities**查询，如下面的示例中所示。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -Detailed
```

### <a name="retrieve-a-specific-diagnostic-activity-by-activity-id"></a>检索特定的诊断活动通过活动 ID

**-ActivityId**参数返回特定诊断活动，如果存在，如下面的示例 cmdlet 中所示。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityId <ActivityIdGuid>
```

### <a name="filter-diagnostic-activities-by-user"></a>筛选器诊断活动用户。

**-UserName**参数返回指定的用户发出的诊断活动的列表，如下面的示例 cmdlet 中所示。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN>
```

**-UserName**参数还可以结合其他可选的筛选参数。

### <a name="filter-diagnostic-activities-by-time"></a>按时间筛选诊断活动

您可以筛选与返回的诊断活动列表 **-StartTime**并 **-EndTime**参数。 **-StartTime**参数将返回从特定日期，开始诊断活动列表，如下面的示例中所示。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -StartTime "08/01/2018"
```

**-EndTime**参数可以添加到 cmdlet 与 **-StartTime**参数来指定特定时间段内你想要接收的结果。 下面的示例 cmdlet 将返回介于 （8 月 1 日和年 8 月 10） 的诊断活动的列表。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -StartTime "08/01/2018" -EndTime "08/10/2018"
```

**-StartTime**并 **-EndTime**参数还可以结合其他可选的筛选参数。

### <a name="filter-diagnostic-activities-by-activity-type"></a>按活动类型筛选器诊断活动

你还可以筛选按与活动类型的诊断活动 **-ActivityType**参数。 以下 cmdlet 将返回一组最终用户连接：

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityType Connection
```

以下 cmdlet 将返回一组管理员管理任务：

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityType Management
```

**Get RdsDiagnosticActivities** cmdlet 当前不支持指定为 ActivityType 的源。

### <a name="filter-diagnostic-activities-by-outcome"></a>按结果筛选诊断活动

可以按与结果筛选返回的诊断活动列表 **-结果**参数。 下面的示例 cmdlet 将返回成功的诊断活动的列表。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -Outcome Success
```

下面的示例 cmdlet 将返回失败的诊断活动的列表。

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -Outcome Failure
```

**-结果**参数还可以结合其他可选的筛选参数。

## <a name="common-error-scenarios"></a>常见错误情况

错误方案被分到该服务的内部和外部到 Windows 虚拟机中。

* 内部问题： 指定的方案不能由租户管理员可以缓解并需要解决的支持问题。 提供反馈意见时[Windows 虚拟桌面技术社区](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop)、 包含的活动 ID 和近似时间范围的问题发生时。
* 外部问题： 与系统管理员可以缓解方案相关。 这些是外部的 Windows 虚拟桌面。

下表列出了你的管理员可能会遇到的常见错误。

>[!NOTE]
>此预览版不包括完整的错误分类，将定期更新。 若要确保具有最新信息，请务必检查返回的这篇文章至少一次一个月。

### <a name="external-management-error-codes"></a>外部管理错误代码

|数值代码|错误代码|建议的解决方案|
|---|---|---|
|3|UnauthorizedAccess|尝试运行管理 PowerShell cmdlet 的用户没有权限执行此操作，或键入其用户名了错误。|
|1000|TenantNotFound|您输入的租户名称不匹配任何现有租户。 查看拼写错误的租户名称，然后重试。|
|1006|TenantCannotBeRemovedHasSessionHostPools|不能删除租户，因为长时间它包含的对象。 删除会话主机池，然后再重试。|
|2000|HostPoolNotFound|您输入的主机池名称不匹配任何现有的主机池。 查看主机池名称以避免拼写错误，然后重试。|
|2005|HostPoolCannotBeRemovedHasApplicationGroups|无法删除主机池，前提是它包含的对象。 首先在主机池中删除所有应用程序组。|
|2004|HostPoolCannotBeRemovedHasSessionHosts|删除第一次然后才能删除会话主机池的所有会话主机。|
|5001|SessionHostNotFound|您已查询会话主机可能处于脱机状态。 检查主机池的状态。|
|5008|SessionHostUserSessionsExist |你必须注销会话主机上的所有用户在执行预期的管理活动之前。|
|6000|AppGroupNotFound|您输入的应用程序组名称不匹配任何现有的应用程序组。 查看以避免拼写错误的应用程序组名称，然后重试。|
|6022|RemoteAppNotFound|您输入的 RemoteApp 名称不匹配任何 RemoteApps。 查看 RemoteApp 名称以避免拼写错误，然后重试。|
|6010|PublishedItemsExist|想要发布的名称是资源的作为资源已存在相同的。 更改资源名称，然后重试。|
|7002|NameNotValidWhiteSpace|不要在名称中使用的空白区域。|
|8000|InvalidAuthorizationRoleScope|您输入的角色名称不与任何现有的角色名称匹配。 查看以避免拼写错误的角色名称，然后重试。 |
|8001|UserNotFound |您输入的用户名与任何现有的用户名称不匹配。 查看以避免拼写错误的名称，然后重试。|
|8005|UserNotFoundInAAD |您输入的用户名与任何现有的用户名称不匹配。 查看以避免拼写错误的名称，然后重试。|
|8008|TenantConsentRequired|按照说明进行操作[此处](tenant-setup-azure-active-directory.md#grant-azure-active-directory-permissions-to-the-windows-virtual-desktop-preview-service)同意你的租户。|

### <a name="external-connection-error-codes"></a>外部连接错误代码

|数值代码|错误代码|建议的解决方案|
|---|---|---|
|-2147467259|ConnectionFailedAdTrustedRelationshipFailure|会话主机未正确联接到 Active Directory。|
|-2146233088|ConnectionFailedUserHasValidSessionButRdshIsUnhealthy|连接失败，因为会话主机不可用。 检查会话主机的运行状况。|
|-2146233088|ConnectionFailedClientDisconnect|如果经常看到此错误，请确保用户的计算机连接到网络。|
|-2146233088|ConnectionFailedNoHealthyRdshAvailable|主机用户尝试连接到该会话无法正常运行。 调试虚拟机。|
|-2146233088|ConnectionFailedUserNotAuthorized|用户不会有权访问已发布的应用程序或桌面。 管理员删除已发布的资源后，可能会出现错误。 要求用户刷新远程桌面应用程序中的源。|
|2|FileNotFound|用户尝试访问该应用程序已正确安装或设置为不正确的路径。|
|3|InvalidCredentials|用户名或密码输入的用户不匹配任何现有的用户名或密码。 查看以避免拼写错误的凭据，然后重试。|
|8|ConnectionBroken|客户端和网关或服务器之间的连接断开。 不需要除非发生意外操作。|
|14|UnexpectedNetworkDisconnect|与网络的连接断开。 要求用户重新连接。|
|24|ReverseConnectFailed|主机虚拟机有没有直接连接到 RD 网关。 确保可以解析的网关 IP 地址。|

## <a name="next-steps"></a>后续步骤

若要了解有关 Windows 虚拟桌面中的角色的详细信息，请参阅[Windows 虚拟桌面预览环境](environment-setup.md)。

若要查看 Windows 虚拟机可用的 PowerShell cmdlet 的列表，请参阅[PowerShell 参考](/powershell/windows-virtual-desktop/overview)。
