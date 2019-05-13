---
title: include 文件
description: include 文件
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
origin.date: 05/17/2018
ms.date: 11/12/2018
ms.author: v-biyu
ms.custom: include file
ms.openlocfilehash: f189843e865863d9b4088363e691ebc42ad9f2a9
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60865738"
---
### <a name="network-interfaces"></a>网络接口

|  |  |
|---------|---------|
| [每个 NIC 上的 NSG X](../articles/governance/policy/samples/nsg-on-nic.md) | 要求将特定网络安全组用于所有虚拟网络接口。 指定要使用的网络安全组的 ID。 |
| [对 VM 网络接口使用已批准的子网](../articles/governance/policy/samples/use-approved-subnet-vm-nics.md) | 要求网络接口使用已批准的子网。 指定已批准的子网的 ID。 |
| [对 VM 网络接口使用已批准的 vNet](../articles/governance/policy/samples/use-approved-vnet-vm-nics.md) | 要求网络接口使用已批准的虚拟网络。 由你指定已批准的虚拟网络的 ID。 |