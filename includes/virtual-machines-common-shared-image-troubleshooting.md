---
title: include 文件
description: include 文件
services: virtual-machines
author: axayjo
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: akjosh; cynthn
ms.custom: include file
ms.openlocfilehash: 1476830313296615591a69a2cadd04bcc56b22bc
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65149633"
---
如果在对共享映像库、映像定义和映像版本执行任何操作时遇到问题，请在调试模式下再次运行失败的命令。 若要激活调试模式，可以通过 CLI 来传递 **-debug** 开关，通过 PowerShell 来传递 **-Debug** 开关。 确定错误的位置以后，请按本文档的说明来排查错误。


## <a name="unable-to-create-a-shared-image-gallery"></a>无法创建共享的映像库

可能的原因：

库名称无效。

允许用于库名称的字符为大写或小写字母、数字、点和句点。 库名称不能包含短划线。 请更改库名称，然后重试。 

库名称在订阅中不唯一。

请选取其他库名称，然后重试。


## <a name="unable-to-create-an-image-definition"></a>无法创建映像定义。 

可能的原因：

映像定义名称无效。

用于映像定义允许的字符是大写或小写字母、 数字、 点、 短划线和句点。 请更改映像定义名称，然后重试。

创建映像定义所需的属性未填充。

名称、发布者、套餐、sku 和 OS 类型等属性是必需的。 验证是否传递所有属性。

请确保映像定义的 **OSType**（Linux 或 Windows）与用于创建映像版本的源托管映像相同。 


## <a name="unable-to-create-an-image-version"></a>无法创建映像版本 

可能的原因：

映像版本名称无效。

允许用于映像版本的字符为数字和句点。 数字必须在 32 位整数范围内。 格式：*MajorVersion.MinorVersion.Patch*。 请更改映像版本名称，然后重试。

找不到要从其创建映像版本的源托管映像。 

查看源映像是否存在，以及是否与映像版本位于同一区域。

托管的映像尚未预配完毕。

请确保源托管映像的预配状态为“成功”。

*目标区域列表不包含源区域。*

目标区域列表必须包含映像版本的源区域。 请确保目标区域的列表中已包含源区域。目标区域是需要通过 Azure 将映像版本复制到其中的区域。

尚未完成到所有目标区域的复制。

请使用 **--expand ReplicationStatus** 标志来检查是否已完成到所有指定目标区域的复制。 如果尚未完成，请等待作业完成，最长等待时间为 6 小时。 如果失败，请再次运行用于创建并复制映像版本的命令。 如果需要将映像版本复制到许多目标区域，可考虑分阶段进行复制。

## <a name="unable-to-create-a-vm-or-a-scale-set"></a>无法创建 VM 或规模集 

可能的原因：

尝试创建 VM 或虚拟机规模集的用户没有映像版本的读取访问权限。

请联系订阅所有者，要求他们通过[基于角色的访问控制](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles) (RBAC) 授予对映像版本或父资源（例如共享映像库或映像定义）的读取访问权限。 

找不到映像版本。

验证你要尝试在其中创建 VM 或虚拟机规模集的区域是否已包括在映像版本目标区域的列表中。 如果该区域已在目标区域的列表中，则请验证复制作业是否已完成。 可以使用 **-ReplicationStatus** 标志来检查是否已完成到所有指定目标区域的复制。 

创建 VM 或虚拟机规模集需要很长的时间。

验证你要尝试从其创建 VM 或虚拟机规模集的映像版本的 **OSType** 与用于创建映像版本的源托管映像的 **OSType** 是否相同。 

## <a name="unable-to-share-resources"></a>无法共享资源

在订阅之间的共享的映像库、 映像定义和映像版本资源共享，启用[基于角色的访问控制](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles)(RBAC)。 

## <a name="replication-is-slow"></a>复制速度慢

请使用 **--expand ReplicationStatus** 标志来检查是否已完成到所有指定目标区域的复制。 如果尚未完成，请等待作业完成，最长等待时间为 6 小时。 如果失败，请再次触发用于创建并复制映像版本的命令。 如果需要将映像版本复制到许多目标区域，可考虑分阶段进行复制。

## <a name="azure-limits-and-quotas"></a>Azure 限制和配额 

[Azure 限制和配额](https://docs.microsoft.com/azure/azure-subscription-service-limits)适用于所有共享映像库、映像定义和映像版本资源。 请确保未超出订阅限制。 



