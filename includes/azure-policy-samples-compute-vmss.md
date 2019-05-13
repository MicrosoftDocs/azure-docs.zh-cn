---
title: include 文件
description: include 文件
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/12/2018
origin.date: 05/17/2018
ms.author: v-biyu
ms.custom: include file
ms.openlocfilehash: 4e2412a98b9f292398b972099daa1642534f180d
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60865749"
---
### <a name="virtual-machine-scale-sets"></a>虚拟机规模集

|  |  |
|---------|---------|
| [在 VM 未使用托管磁盘时审核](../articles/governance/policy/samples/create-vm-managed-disk.md) | 如果创建了未使用托管磁盘的虚拟机，则进行审核。|
| [使用托管磁盘创建 VM](../articles/governance/policy/samples/use-managed-disk-vm.md) | 要求虚拟机使用托管磁盘。|
| [拒绝混合使用权益](../articles/governance/policy/samples/deny-hybrid-use.md) | 禁止使用 Azure 混合使用权益 (AHUB)。 在想禁止使用本地许可证时使用。 |
| [只允许特定 VM 平台映像](../articles/governance/policy/samples/allow-certain-vm-image.md) | 要求虚拟机使用 UbuntuServer 的特定版本。 |