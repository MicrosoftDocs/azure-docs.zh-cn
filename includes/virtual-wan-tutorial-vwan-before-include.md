---
title: include 文件
description: include 文件
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 09/12/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 66e21aefa5648262ca2fcc61501905f725ac0e4b
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60457841"
---
在开始配置之前，请验证你是否符合以下条件：

* 如果已有一个要连接的虚拟网络，请确认本地网络的任何子网都不与要连接到的虚拟网络重叠。 虚拟网络不需要网关子网，并且不能包含任何虚拟网络网关。 如果没有虚拟网络，可以使用本文中的步骤创建一个。
* 获取中心区域的 IP 地址范围。 中心是一个虚拟网络，为中心区域指定的地址范围不能与要连接到的任何现有虚拟网络重叠。 此外，它也不能与本地连接到的地址范围重叠。 如果不熟悉本地网络配置中的 IP 地址范围，则需咨询能够提供此类详细信息的人员。
* 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。