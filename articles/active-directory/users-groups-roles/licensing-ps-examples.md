---
title: 许可组-Azure Active Directory 的 PowerShell 和 Graph 示例 |Microsoft Docs
description: PowerShell + 关系图的示例和方案 Azure Active Directory 基于组的许可
services: active-directory
keywords: Azure AD 许可
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.subservice: users-groups-roles
ms.topic: article
ms.workload: identity
ms.date: 03/18/2019
ms.author: curtand
ms.reviewer: sumitp
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2fc6e31afbb7ced4699afef38b67b637914198e4
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65192417"
---
# <a name="powershell-examples-for-group-based-licensing-in-azure-ad"></a>Azure AD 中基于组的许可的 PowerShell 示例

基于组的许可的完整功能可通过 [Azure 门户](https://portal.azure.com)获取，PowerShell 和 Microsoft Graph 支持目前受到限制。 但是，可使用现有的 [MSOnline PowerShell cmdlet](https://docs.microsoft.com/powershell/msonline/v1/azureactivedirectory) 和 Microsoft Graph 执行一些有用的任务。 本文档提供了可执行的任务示例。

> [!NOTE]
> 在开始运行 cmdlet 之前，请确保首先，通过运行连接你的组织`Connect-MsolService`  cmdlet。

> [!WARNING]
> 此示例代码用于演示目的。 如果想要在环境中使用，请考虑先进行小规模的测试，或者在单独的测试租户中使用。 可能需要根据具体的环境需求调整该代码。

## <a name="view-product-licenses-assigned-to-a-group"></a>查看分配给组的产品许可证

[Get-msolgroup](/powershell/module/msonline/get-msolgroup?view=azureadps-1.0) cmdlet 可用于检索组对象并检查“许可证”属性：它会列出当前分配给组的所有产品许可证。

```powershell
(Get-MsolGroup -ObjectId 99c4216a-56de-42c4-a4ac-e411cd8c7c41).Licenses
| Select SkuPartNumber
```
输出：
```
SkuPartNumber
-------------
ENTERPRISEPREMIUM
EMSPREMIUM
```

> [!NOTE]
> 数据被限制为产品 (SKU) 信息。 无法列出许可证中禁用的服务计划。

使用下面的示例从 Microsoft Graph 获取相同的数据。

```
GET https://graph.microsoft.com/v1.0/groups/99c4216a-56de-42c4-a4ac-e411cd8c7c41$select=assignedLicenses
```
输出：
```
HTTP/1.1 200 OK
{
  “value”: [
{
  “assignedLicenses”: [
     {
          "accountId":"f1b45b40-57df-41f7-9596-7f2313883635",
          "skuId":"c7df2760-2c81-4ef7-b578-5b5392b571df",
      "disabledPlans":[]
     },
     {
          "accountId":"f1b45b40-57df-41f7-9596-7f2313883635",
          "skuId":" b05e124f-c7cc-45a0-a6aa-8cf78c946968",
      "disabledPlans":[]
     },
  ],
}
  ]
}
```

## <a name="get-all-groups-with-licenses"></a>获取所有具有许可证的组

可通过运行以下命令，查找所有已分配有任意许可证的组：
```powershell
Get-MsolGroup | Where {$_.Licenses}
```
可显示更多有关分配了哪些产品的详细信息：
```powershell
Get-MsolGroup | Where {$_.Licenses} | Select `
    ObjectId, `
    DisplayName, `
    @{Name="Licenses";Expression={$_.Licenses | Select -ExpandProperty SkuPartNumber}}
```

输出：
```
ObjectId                             DisplayName              Licenses
--------                             -----------              --------
7023a314-6148-4d7b-b33f-6c775572879a EMS E5 – Licensed users  EMSPREMIUM
cf41f428-3b45-490b-b69f-a349c8a4c38e PowerBi - Licensed users POWER\_BI\_STANDARD
962f7189-59d9-4a29-983f-556ae56f19a5 O365 E3 - Licensed users ENTERPRISEPACK
c2652d63-9161-439b-b74e-fcd8228a7074 EMSandOffice             {ENTERPRISEPREMIUM,EMSPREMIUM}
```

## <a name="get-statistics-for-groups-with-licenses"></a>获取具有许可证的组的统计信息
可报告具有许可证的组的基本统计信息。 在下面的示例中，脚本列出了总用户数、组已分配的具有许可证的用户数，以及组无法为其分配许可证的用户数。

```powershell
#get all groups with licenses
Get-MsolGroup -All | Where {$_.Licenses}  | Foreach {
    $groupId = $_.ObjectId;
    $groupName = $_.DisplayName;
    $groupLicenses = $_.Licenses | Select -ExpandProperty SkuPartNumber
    $totalCount = 0;
    $licenseAssignedCount = 0;
    $licenseErrorCount = 0;

    Get-MsolGroupMember -All -GroupObjectId $groupId |
    #get full info about each user in the group
    Get-MsolUser -ObjectId {$_.ObjectId} |     Foreach {
        $user = $_;
        $totalCount++

        #check if any licenses are assigned via this group
        if($user.Licenses | ? {$_.GroupsAssigningLicense -ieq $groupId })
        {
            $licenseAssignedCount++
        }
        #check if user has any licenses that failed to be assigned from this group
        if ($user.IndirectLicenseErrors | ? {$_.ReferencedObjectId -ieq $groupId })
        {
            $licenseErrorCount++
        }     
    }

    #aggregate results for this group
    New-Object Object |
                    Add-Member -NotePropertyName GroupName -NotePropertyValue $groupName -PassThru |
                    Add-Member -NotePropertyName GroupId -NotePropertyValue $groupId -PassThru |
                    Add-Member -NotePropertyName GroupLicenses -NotePropertyValue $groupLicenses -PassThru |
                    Add-Member -NotePropertyName TotalUserCount -NotePropertyValue $totalCount -PassThru |
                    Add-Member -NotePropertyName LicensedUserCount -NotePropertyValue $licenseAssignedCount -PassThru |
                    Add-Member -NotePropertyName LicenseErrorCount -NotePropertyValue $licenseErrorCount -PassThru

    } | Format-Table
```


输出：
```
GroupName         GroupId                              GroupLicenses       TotalUserCount LicensedUserCount LicenseErrorCount
---------         -------                              -------------       -------------- ----------------- -----------------
Dynamics Licen... 9160c903-9f91-4597-8f79-22b6c47eafbf AAD_PREMIUM_P2                   0                 0                 0
O365 E5 - base... 055dcca3-fb75-4398-a1b8-f8c6f4c24e65 ENTERPRISEPREMIUM                2                 2                 0
O365 E5 - extr... 6b14a1fe-c3a9-4786-9ee4-3a2bb54dcb8e ENTERPRISEPREMIUM                3                 3                 0
EMS E5 - all s... 7023a314-6148-4d7b-b33f-6c775572879a EMSPREMIUM                       2                 2                 0
PowerBi - Lice... cf41f428-3b45-490b-b69f-a349c8a4c38e POWER_BI_STANDARD                2                 2                 0
O365 E3 - all ... 962f7189-59d9-4a29-983f-556ae56f19a5 ENTERPRISEPACK                   2                 2                 0
O365 E5 - EXO     102fb8f4-bbe7-462b-83ff-2145e7cdd6ed ENTERPRISEPREMIUM                1                 1                 0
Access to Offi... 11151866-5419-4d93-9141-0603bbf78b42 STANDARDPACK                     4                 3                 1
```

## <a name="get-all-groups-with-license-errors"></a>获取含有许可证错误的所有组
查找组，这些组中包含一些无法为其分配许可证的用户：
```powershell
Get-MsolGroup -HasLicenseErrorsOnly $true
```
输出：
```
ObjectId                             DisplayName             GroupType Description
--------                             -----------             --------- -----------
11151866-5419-4d93-9141-0603bbf78b42 Access to Office 365 E1 Security  Users who should have E1 licenses
```
使用以下命令从 Microsoft Graph 获取相同的数据
```
GET https://graph.microsoft.com/v1.0/groups?$filter=hasMembersWithLicenseErrors+eq+true
```
输出：
```
HTTP/1.1 200 OK
{
  "value":[
    {
      "odata.type": "Microsoft.DirectoryServices.Group",
      "objectType": "Group",
      "id": "11151866-5419-4d93-9141-0603bbf78b42",
      ... # other group properties.
    },
    {
      "odata.type": "Microsoft.DirectoryServices.Group",
      "objectType": "Group",
      "id": "c57cdc98-0dcd-4f90-a82f-c911b288bab9",
      ...
    },
    ... # other groups with license errors.
  ]
"odata.nextLink":"https://graph.microsoft.com/v1.0/ groups?$filter=hasMembersWithLicenseErrors+eq+true&$skipToken=<encodedPageToken>"
}
```


## <a name="get-all-users-with-license-errors-in-a-group"></a>获取组中含有许可证错误的所有用户

如果某个组中包含一些许可证相关的错误，现在可以列出受这些错误影响的所有用户。 用户也可可含有其他组中的错误。 但在此示例中，通过检查用户的每个 **IndirectLicenseError** 条目的 **ReferencedObjectId** 属性，我们仅将结果限于与所涉及组相关的错误。

```powershell
#a sample group with errors
$groupId = '11151866-5419-4d93-9141-0603bbf78b42'

#get all user members of the group
Get-MsolGroupMember -All -GroupObjectId $groupId |
    #get full information about user objects
    Get-MsolUser -ObjectId {$_.ObjectId} |
    #filter out users without license errors and users with license errors from other groups
    Where {$_.IndirectLicenseErrors -and $_.IndirectLicenseErrors.ReferencedObjectId -eq $groupId} |
    #display id, name and error detail. Note: we are filtering out license errors from other groups
    Select ObjectId, `
           DisplayName, `
           @{Name="LicenseError";Expression={$_.IndirectLicenseErrors | Where {$_.ReferencedObjectId -eq $groupId} | Select -ExpandProperty Error}}
```

输出：

```powershell
ObjectId                             DisplayName      License Error
--------                             -----------      ------------
6d325baf-22b7-46fa-a2fc-a2500613ca15 Catherine Gibson MutuallyExclusiveViolation
```

使用以下命令以从 Microsoft Graph 获取相同的数据：

```powershell
GET https://graph.microsoft.com/v1.0/groups/11151866-5419-4d93-9141-0603bbf78b42/membersWithLicenseErrors
```

输出：
```powershell
HTTP/1.1 200 OK
{
  "value":[
    {
      "odata.type": "Microsoft.DirectoryServices.User",
      "objectType": "User",
      "id": "6d325baf-22b7-46fa-a2fc-a2500613ca15",
      ... # other user properties.
    },
    ... # other users.
  ],
  "odata.nextLink":"https://graph.microsoft.com/v1.0/groups/11151866-5419-4d93-9141-0603bbf78b42/membersWithLicenseErrors?$skipToken=<encodedPageToken>"
}

```

## <a name="get-all-users-with-license-errors-in-the-entire-tenant"></a>获取整个租户中含有许可证错误的所有用户

可以使用以下脚本获取一个或多个组中具有许可证错误的所有用户。 此脚本将按每个用户、每个许可证错误输出一行，以便可以清楚地确定每个错误的源。

> [!NOTE]
> 此脚本将枚举租户中的所有用户，这对于大型租户来说可能不是最佳做法。

```powershell
Get-MsolUser -All | Where {$_.IndirectLicenseErrors } | % {   
    $user = $_;
    $user.IndirectLicenseErrors | % {
            New-Object Object |
                Add-Member -NotePropertyName UserName -NotePropertyValue $user.DisplayName -PassThru |
                Add-Member -NotePropertyName UserId -NotePropertyValue $user.ObjectId -PassThru |
                Add-Member -NotePropertyName GroupId -NotePropertyValue $_.ReferencedObjectId -PassThru |
                Add-Member -NotePropertyName LicenseError -NotePropertyValue $_.Error -PassThru
        }
    }  
```

输出：

```powershell
UserName         UserId                               GroupId                              LicenseError
--------         ------                               -------                              ------------
Anna Bergman     0d0fd16d-872d-4e87-b0fb-83c610db12bc 7946137d-b00d-4336-975e-b1b81b0666d0 MutuallyExclusiveViolation
Catherine Gibson 6d325baf-22b7-46fa-a2fc-a2500613ca15 f2503e79-0edc-4253-8bed-3e158366466b CountViolation
Catherine Gibson 6d325baf-22b7-46fa-a2fc-a2500613ca15 11151866-5419-4d93-9141-0603bbf78b42 MutuallyExclusiveViolation
Drew Fogarty     f2af28fc-db0b-4909-873d-ddd2ab1fd58c 1ebd5028-6092-41d0-9668-129a3c471332 MutuallyExclusiveViolation
```

下面是该脚本的另一个版本，它只在包含许可证错误的组中搜索。 预期有问题的组较少的情况下，这可能最适用。

```powershell
$groupIds = Get-MsolGroup -HasLicenseErrorsOnly $true
    foreach ($groupId in $groupIds) {
    Get-MsolGroupMember -All -GroupObjectId $groupId.ObjectID |
        Get-MsolUser -ObjectId {$_.ObjectId} |
        Where {$_.IndirectLicenseErrors -and $_.IndirectLicenseErrors.ReferencedObjectId -eq $groupId.ObjectID} |
        Select DisplayName, `
               ObjectId, `
               @{Name="LicenseError";Expression={$_.IndirectLicenseErrors | Where {$_.ReferencedObjectId -eq $groupId.ObjectID} | Select -ExpandProperty Error}}
 
    } 
``` 

## <a name="check-if-user-license-is-assigned-directly-or-inherited-from-a-group"></a>检查用户许可证是直接从组分配还是从组继承

针对某个用户对象，可以检查某特定的产品许可证是从组分配的还是直接分配的。

以下两个示例函数可用于分析对单个用户的分配类型：

```powershell
#Returns TRUE if the user has the license assigned directly
function UserHasLicenseAssignedDirectly
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    foreach($license in $user.Licenses)
    {
        #we look for the specific license SKU in all licenses assigned to the user
        if ($license.AccountSkuId -ieq $skuId)
        {
            #GroupsAssigningLicense contains a collection of IDs of objects assigning the license
            #This could be a group object or a user object (contrary to what the name suggests)
            #If the collection is empty, this means the license is assigned directly - this is the case for users who have never been licensed via groups in the past
            if ($license.GroupsAssigningLicense.Count -eq 0)
            {
                return $true
            }

            #If the collection contains the ID of the user object, this means the license is assigned directly
            #Note: the license may also be assigned through one or more groups in addition to being assigned directly
            foreach ($assignmentSource in $license.GroupsAssigningLicense)
            {
                if ($assignmentSource -ieq $user.ObjectId)
                {
                    return $true
                }
            }
            return $false
        }
    }
    return $false
}
#Returns TRUE if the user is inheriting the license from a group
function UserHasLicenseAssignedFromGroup
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    foreach($license in $user.Licenses)
    {
        #we look for the specific license SKU in all licenses assigned to the user
        if ($license.AccountSkuId -ieq $skuId)
        {
            #GroupsAssigningLicense contains a collection of IDs of objects assigning the license
            #This could be a group object or a user object (contrary to what the name suggests)
            foreach ($assignmentSource in $license.GroupsAssigningLicense)
            {
                #If the collection contains at least one ID not matching the user ID this means that the license is inherited from a group.
                #Note: the license may also be assigned directly in addition to being inherited
                if ($assignmentSource -ine $user.ObjectId)
                {
                    return $true
                }
            }
            return $false
        }
    }
    return $false
}
```

此脚本使用 SKU ID 作为输入，对租户中的每位用户执行这些功能 - 在本示例中，用于企业移动性 + 安全性的许可证在租户中的 ID 表示为：contoso:EMS：

```powershell
#the license SKU we are interested in. use Get-MsolAccountSku to see a list of all identifiers in your tenant
$skuId = "contoso:EMS"

#find all users that have the SKU license assigned
Get-MsolUser -All | where {$_.isLicensed -eq $true -and $_.Licenses.AccountSKUID -eq $skuId} | select `
    ObjectId, `
    @{Name="SkuId";Expression={$skuId}}, `
    @{Name="AssignedDirectly";Expression={(UserHasLicenseAssignedDirectly $_ $skuId)}}, `
    @{Name="AssignedFromGroup";Expression={(UserHasLicenseAssignedFromGroup $_ $skuId)}}
```

输出：

```powershell
ObjectId                             SkuId       AssignedDirectly AssignedFromGroup
--------                             -----       ---------------- -----------------
157870f6-e050-4b3c-ad5e-0f0a377c8f4d contoso:EMS             True             False
1f3174e2-ee9d-49e9-b917-e8d84650f895 contoso:EMS            False              True
240622ac-b9b8-4d50-94e2-dad19a3bf4b5 contoso:EMS             True              True
```

图形没有了简单方法以显示结果，但此 API 可以看到它：

```powershell
GET https://graph.microsoft.com/v1.0/users/e61ff361-5baf-41f0-b2fd-380a6a5e406a?$select=licenseAssignmentStates
```

输出：

```powershell
HTTP/1.1 200 OK
{
  "value":[
    {
      "odata.type": "Microsoft.DirectoryServices.User",
      "objectType": "User",
      "id": "e61ff361-5baf-41f0-b2fd-380a6a5e406a",
      "licenseAssignmentState":[
        {
          "skuId": "157870f6-e050-4b3c-ad5e-0f0a377c8f4d”,
          "disabledPlans":[],
          "assignedByGroup": null, # assigned directly.
          "state": "Active",
          "error": "None"
        },
        {
          "skuId": "1f3174e2-ee9d-49e9-b917-e8d84650f895",
          "disabledPlans":[],
          "assignedByGroup": “e61ff361-5baf-41f0-b2fd-380a6a5e406a”, # assigned by this group.
          "state": "Active",
          "error": "None"
        },
        {
          "skuId": "240622ac-b9b8-4d50-94e2-dad19a3bf4b5", 
          "disabledPlans":[
            "e61ff361-5baf-41f0-b2fd-380a6a5e406a"
          ],
          "assignedByGroup": "e61ff361-5baf-41f0-b2fd-380a6a5e406a",
          "state": "Active",
          "error": "None"
        },
        {
          "skuId": "240622ac-b9b8-4d50-94e2-dad19a3bf4b5",
          "disabledPlans":[],
          "assignedByGroup": null, # It is the same license as the previous one. It means the license is assigned directly once and inherited from group as well.
          "state": " Active ",
          "error": " None"
        }
      ],
      ...
    }
  ],
}

```

## <a name="remove-direct-licenses-for-users-with-group-licenses"></a>删除具有组许可证的用户的直接许可证

此脚本的目的是从用户（已从组继承相同许可证）删除不必要的直接许可证；例如，作为[转换到基于组的许可](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-migration-azure-portal)的一部分。
> [!NOTE]
> 请务必先验证要删除的直接许可证没有比继承许可证启用更多的服务功能。 否则，删除直接许可证可能会禁用用户对服务和数据的访问。 当前无法借助 PowerShell 检查通过继承许可证与直接许可证启用了哪些服务。 在该脚本中，我们将指定要从组继承的服务的最低级别，并且将针对其进行检查，以确保用户不会意外丢失对服务的访问。

```powershell
#BEGIN: Helper functions used by the script

#Returns TRUE if the user has the license assigned directly
function UserHasLicenseAssignedDirectly
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    $license = GetUserLicense $user $skuId

    if ($license -ne $null)
    {
        #GroupsAssigningLicense contains a collection of IDs of objects assigning the license
        #This could be a group object or a user object (contrary to what the name suggests)
        #If the collection is empty, this means the license is assigned directly - this is the case for users who have never been licensed via groups in the past
        if ($license.GroupsAssigningLicense.Count -eq 0)
        {
            return $true
        }

        #If the collection contains the ID of the user object, this means the license is assigned directly
        #Note: the license may also be assigned through one or more groups in addition to being assigned directly
        foreach ($assignmentSource in $license.GroupsAssigningLicense)
        {
            if ($assignmentSource -ieq $user.ObjectId)
            {
                return $true
            }
        }
        return $false
    }
    return $false
}
#Returns TRUE if the user is inheriting the license from a specific group
function UserHasLicenseAssignedFromThisGroup
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId, [Guid]$groupId)

    $license = GetUserLicense $user $skuId

    if ($license -ne $null)
    {
        #GroupsAssigningLicense contains a collection of IDs of objects assigning the license
        #This could be a group object or a user object (contrary to what the name suggests)
        foreach ($assignmentSource in $license.GroupsAssigningLicense)
        {
            #If the collection contains at least one ID not matching the user ID this means that the license is inherited from a group.
            #Note: the license may also be assigned directly in addition to being inherited
            if ($assignmentSource -ieq $groupId)
            {
                return $true
            }
        }
        return $false
    }
    return $false
}

#Returns the license object corresponding to the skuId. Returns NULL if not found
function GetUserLicense
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId, [Guid]$groupId)
    #we look for the specific license SKU in all licenses assigned to the user
    foreach($license in $user.Licenses)
    {
        if ($license.AccountSkuId -ieq $skuId)
        {
            return $license
        }
    }
    return $null
}

#produces a list of disabled service plan names for a set of plans we want to leave enabled
function GetDisabledPlansForSKU
{
    Param([string]$skuId, [string[]]$enabledPlans)

    $allPlans = Get-MsolAccountSku | where {$_.AccountSkuId -ieq $skuId} | Select -ExpandProperty ServiceStatus | Where {$_.ProvisioningStatus -ine "PendingActivation" -and $_.ServicePlan.TargetClass -ieq "User"} | Select -ExpandProperty ServicePlan | Select -ExpandProperty ServiceName
    $disabledPlans = $allPlans | Where {$enabledPlans -inotcontains $_}

    return $disabledPlans
}

function GetUnexpectedEnabledPlansForUser
{
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId, [string[]]$expectedDisabledPlans)

    $license = GetUserLicense $user $skuId

    $extraPlans = @();

    if($license -ne $null)
    {
        $userDisabledPlans = $license.ServiceStatus | where {$_.ProvisioningStatus -ieq "Disabled"} | Select -ExpandProperty ServicePlan | Select -ExpandProperty ServiceName

        $extraPlans = $expectedDisabledPlans | where {$userDisabledPlans -notcontains $_}
    }
    return $extraPlans
}
#END: helper functions

#BEGIN: executing the script
#the group to be processed
$groupId = "48ca647b-7e4d-41e5-aa66-40cab1e19101"

#license to be removed - Office 365 E3
$skuId = "contoso:ENTERPRISEPACK"

#minimum set of service plans we know are inherited from groups - we want to make sure that there aren't any users who have more services enabled
#which could mean that they may lose access after we remove direct licenses
$servicePlansFromGroups = ("EXCHANGE_S_ENTERPRISE", "SHAREPOINTENTERPRISE", "OFFICESUBSCRIPTION")

$expectedDisabledPlans = GetDisabledPlansForSKU $skuId $servicePlansFromGroups

#process all members in the group and get full info about each user in the group looping through group members. 
Get-MsolGroupMember -All -GroupObjectId $groupId | Get-MsolUser -ObjectId {$_.ObjectId} | Foreach {
        $user = $_;
        $operationResult = "";

        #check if Direct license exists on the user
        if (UserHasLicenseAssignedDirectly $user $skuId)
        {
            #check if the license is assigned from this group, as expected
            if (UserHasLicenseAssignedFromThisGroup $user $skuId $groupId)
            {
                #check if there are any extra plans we didn't expect - we are being extra careful not to remove unexpected services
                $extraPlans = GetUnexpectedEnabledPlansForUser $user $skuId $expectedDisabledPlans
                if ($extraPlans.Count -gt 0)
                {
                    $operationResult = "User has extra plans that may be lost - license removal was skipped. Extra plans: $extraPlans"
                }
                else
                {
                    #remove the direct license from user
                    Set-MsolUserLicense -ObjectId $user.ObjectId -RemoveLicenses $skuId
                    $operationResult = "Removed direct license from user."   
                }

            }
            else
            {
                $operationResult = "User does not inherit this license from this group. License removal was skipped."
            }
        }
        else
        {
            $operationResult = "User has no direct license to remove. Skipping."
        }

        #format output
        New-Object Object |
                    Add-Member -NotePropertyName UserId -NotePropertyValue $user.ObjectId -PassThru |
                    Add-Member -NotePropertyName OperationResult -NotePropertyValue $operationResult -PassThru
    } | Format-Table
#END: executing the script
```

输出：

```powershell
UserId                               OperationResult
------                               ---------------
7c7f860f-700a-462a-826c-f50633931362 Removed direct license from user.
0ddacdd5-0364-477d-9e4b-07eb6cdbc8ea User has extra plans that may be lost - license removal was skipped. Extra plans: SHAREPOINTWAC
aadbe4da-c4b5-4d84-800a-9400f31d7371 User has no direct license to remove. Skipping.
```
> [!NOTE]
> 请更新变量的值`$skuId`并`$groupId` 其为目标删除直接许可证根据测试环境之前运行上述脚本。 

## <a name="next-steps"></a>后续步骤

若要详细了解用于通过组进行许可证管理的功能集，请参阅以下文章：

* [Azure Active Directory 中基于组的许可是什么？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [将许可证分配到 Azure Active Directory 中的组](licensing-groups-assign.md)
* [识别和解决 Azure Active Directory 中组的许可问题](licensing-groups-resolve-problems.md)
* [如何将单个许可用户迁移到 Azure Active Directory 中基于组的许可](licensing-groups-migrate-users.md)
* [如何在 Azure Active Directory 中使用基于组的许可在产品许可证之间迁移用户](../users-groups-roles/licensing-groups-change-licenses.md)
* [Azure Active Directory 基于组的许可的其他方案](licensing-group-advanced.md)
* [Azure Active Directory 中基于组的许可的 PowerShell 示例](../users-groups-roles/licensing-ps-examples.md)
