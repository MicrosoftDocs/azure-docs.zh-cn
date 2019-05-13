---
title: include 文件
description: include 文件
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/21/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: dc4281c17b92e1720625764a52a34a94d6f296ab
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "64732749"
---
在创建任何项之前，让我们复习一下作用域的概念。 Azure 提供四个级别的管理：管理组、订阅、资源组和资源。 [管理组](../articles/billing/billing-enterprise-mgmt-group-overview.md)处于预览状态。 下图显示了一个这些层的示例。

![范围](./media/resource-manager-governance-scope/scope-levels.png)

将在上述任何级别的作用域中应用管理设置。 所选的级别确定应用设置的广泛程度。 较低级别继承较高级别的设置。 将设置应用到订阅时，该设置将应用于订阅中的所有资源组和资源。 将设置应用到资源组时，该设置将应用到资源组及其所有资源。 但是，其他资源组不具有该设置。

通常情况下，最好在较高级别应用关键设置，在较低级别应用特定于项目的要求。 例如，可能想要确保组织的所有资源均已部署到特定区域。 若要完成此要求，请将策略应用到指定允许位置的订阅。 当组织中的其他用户添加新资源组和资源时，会自动强制实施允许的位置。 
