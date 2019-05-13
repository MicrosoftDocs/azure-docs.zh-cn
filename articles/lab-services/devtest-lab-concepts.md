---
title: 开发测试实验室概念 | Microsoft Docs
description: 了解开发测试实验室的基本概念及其如何轻松地创建、管理和监视 Azure 虚拟机
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 105919e8-3617-4ce3-a29f-a289fa608fb2
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 08cae51da20b6093b284618de92c61aab4bf5b55
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508387"
---
# <a name="devtest-labs-concepts"></a>开发测试实验室概念
## <a name="overview"></a>概述
下表包含关键开发测试实验室概念和定义：

## <a name="labs"></a>实验室
实验室是包含一组资源（例如，虚拟机 (VM)）的基础结构，通过指定限制和配额可以更好地管理这些资源。

## <a name="virtual-machine"></a>虚拟机
Azure VM 是 Azure 提供的多种[可缩放按需分配计算资源](/azure/architecture/guide/technology-choices/compute-decision-tree)之一。 无需购买和维护运行 Azure VM 的物理硬件，Azure VM 即可提供虚拟化灵活性，尽管你仍需要执行某些任务（如配置、修补和安装在 Azure VM 上运行的软件）来维护 VM。

[Azure 中的 Windows 虚拟机概述](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-overview)提供有关在创建 VM 之前应考虑的信息、如何创建 VM 以及如何管理 VM。

## <a name="claimable-vm"></a>可认领 VM
Azure 可认领 VM 是可供具备权限的任何实验室用户使用的虚拟机。 实验室管理员可准备具有特定基本映像和项目的 VM，并将其保存到共享池。 此后，在需要这种特定配置时，实验室用户可从池中认领一台有用的 VM。

可认领 VM 最初并未分配给任何特定用户，但会显示在“可认领虚拟机”下每个用户的列表中。 用户认领 VM 后，该 VM 会移动到“我的虚拟机”区域，并且任何其他用户都无法再对其进行认领。

## <a name="environment"></a>环境
在开发测试实验室中，环境是指实验室中 Azure 资源的集合。 [此博客文章](https://blogs.msdn.microsoft.com/devtestlab/2016/11/16/connect-2016-news-for-azure-devtest-labs-azure-resource-manager-template-based-environments-vm-auto-shutdown-and-more/)讨论如何基于 Azure 资源管理器模板创建多 VM 环境。

## <a name="base-images"></a>基础映像
基础映像为 VM 映像，包含预安装和配置为快速创建 VM 的所有工具和设置。 可通过选取现有基并添加项目来对 VM 进行预配，进而安装测试代理。 然后可将预配的 VM 保存为基，这样无需为每个 VM 的预配重新安装测试代理即可使用该基。

## <a name="artifacts"></a>项目
项目用于在预配 VM 后部署和配置应用程序。 项目可以是：

* 想要在 VM 上安装的工具，例如，代理、Fiddler 和 Visual Studio。
* 想要在 VM 上运行的操作，例如，克隆存储库。
* 想要测试的应用程序。

项目是 [Azure 资源管理器](../azure-resource-manager/resource-group-overview.md) JSON 文件，包含执行部署和应用配置的说明。

## <a name="artifact-repositories"></a>项目存储库
项目存储库是其中已签入项目的 Git 存储库。 可将项目存储库添加到组织中的多个实验室，从而允许重用和共享。

## <a name="formulas"></a>公式
除基础映像外，还可使用公式提供用于快速预配 VM 的机制。 开发测试实验室中的公式是用于创建实验室 VM 的默认属性值的列表。
利用公式，可以创建具有相同属性（例如，基础映像、VM 大小、虚拟网络和项目）组的 VM，而无需每次都指定这些属性。 使用公式创建 VM 时，可以按原样使用默认值，也可以修改默认值。

## <a name="policies"></a>策略
策略有助于控制实验室中的成本。 例如，可以创建策略来自动关闭基于定义计划的 VM。

## <a name="caps"></a>上限
上限是一种可最大程度减少实验室浪费的机制。 例如，可以将上限设置为限制每个用户或在实验室中可创建的 VM 数量。

## <a name="security-levels"></a>安全级别
安全访问权限由 Azure 基于角色的访问控制 (RBAC) 决定。 若要了解如何访问，最好先了解权限、角色和由 RBAC 定义的作用域之间的差异。

* 权限 - 权限是对特定操作的定义访问，例如，对所有虚拟机的读取访问权限。
* 角色 - 角色是一组可进行分组、可分配给用户的权限。 例如，“订阅所有者”角色对订阅中所有资源都具有访问权限。
* 作用域 - 作用域是 Azure 资源层次结构中的级别，例如，资源组、单个实验室或整个订阅。

开发测试实验室的作用域中存在两种定义用户权限的角色：实验室所有者和实验室用户。

* 实验室所有者 - 实验室所有者具有对实验室中所有资源的访问权限。 因此，实验室所有者可以修改策略、读取和写入任意 VM、更改虚拟网络等。
* 实验室用户 - 实验室用户可查看实验室中的所有资源（例如 VM、策略和虚拟网络），但不能修改其他用户创建的策略和 VM。

若要查看如何在开发测试实验室中创建自定义角色，可参阅本文[向用户授予特定实验室策略的权限](devtest-lab-grant-user-permissions-to-specific-lab-policies.md)。

由于作用域是分层的，因此用户在某个作用域具有权限时，会自动授予他所包含的每个较低级别的作用域的权限。 例如，如果用户分配到订阅所有者的角色，那么他可访问订阅中的所有资源，包括所有虚拟机、所有虚拟网络和所有实验室。 因此，订阅所有者会自动继承实验室所有者的角色。 但是反过来则不适用。 实验室所有者具有访问实验室的权限，其低于订阅级别的作用域。 因此，实验室所有者将不能查看虚拟机或虚拟网络或实验室之外的任何资源。

## <a name="azure-resource-manager-templates"></a>Azure 资源管理器模板
本文中所述的所有概念都可以使用 Azure 资源管理器模板配置，这些模板可用于定义 Azure 解决方案的基础结构/配置，并以一致的状态反复部署该解决方案。

[了解 Azure 资源管理器模板的结构和语法](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates#template-format)描述了 Azure 资源管理器模板的结构，以及模板的不同节中提供的属性。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>后续步骤
[在开发测试实验室中创建实验室](devtest-lab-create-lab.md)
