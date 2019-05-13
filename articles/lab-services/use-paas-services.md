---
title: 在 Azure 开发测试实验室中使用平台-作为-服务 (PaaS) 服务 |Microsoft Docs
description: 了解如何在 Azure 开发测试实验室中使用平台的一种即服务 (Pass) 服务。
services: devtest-lab,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/02/2019
ms.author: spelluru
ms.openlocfilehash: 7cdd185cddbd2403b72ff0e06530913af0b031de
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233118"
---
# <a name="use-platform-as-a-service-paas-services-in-azure-devtest-labs"></a>在 Azure 开发测试实验室中使用平台-作为-服务 (PaaS) 服务
PaaS 开发测试实验室中支持通过环境功能。 通过 Git 存储库中预先配置的 Azure 资源管理器模板支持开发测试实验室中的环境。 环境可以包含 PaaS 和 IaaS 资源。 它们允许您创建可以包括虚拟机、 数据库、 虚拟网络等 Azure 资源的复杂系统和自定义以协同工作的 Web 应用程序。 这些模板允许一致的部署和使用源代码管理环境的已改进的管理。 

一个正确设置系统允许使用下列方案： 

- 开发人员能够独立于和多个环境
- 以异步方式在不同的配置上进行测试
- 集成到过渡和生产管道中，无需更改任何模板
- 使开发机与同一个实验室中的环境可以提高易用性管理和成本报告。  

开发测试实验室资源提供程序实验室用户的名义创建资源，因此任何额外权限不需要提供给实验室用户若要启用对 PaaS 资源的使用。 下图显示 Service Fabric 群集为实验室中的环境。

![为环境的 Service Fabric 群集](./media/create-environment-service-fabric-cluster/cluster-created.png)

有关详细信息上设置环境，请参阅[使用 Azure 资源管理器模板创建多 VM 环境和 PaaS 资源](devtest-lab-create-environment-from-arm.md)。 开发测试实验室有公共存储库的 Azure 资源管理器模板可用于创建环境，而无需自行连接到外部 GitHub 源。 公共环境的详细信息，请参阅[Azure 开发测试实验室中的配置和使用公共环境](devtest-lab-configure-use-public-environments.md)。

在大型组织中，开发团队通常会提供环境，如自定义/隔离测试环境。 可能有用于业务单位或部门中的所有团队的环境。 IT 小组可能想要提供其可由组织中的所有团队的环境。  

## <a name="customizations"></a>自定义项

#### <a name="sandbox"></a>沙盒 
实验室所有者可以自定义实验室环境，若要更改用户的角色**读取器**到**参与者**资源组中。 此功能目前以**实验室设置**页**配置和策略**的实验室。 此角色中的更改允许用户添加或删除该环境中的资源。 如果你想要限制访问权限进一步，使用 Azure 策略。 此功能允许你自定义资源或配置，而不在订阅级别的访问权限。

#### <a name="custom-tokens"></a>自定义令牌
还有一些自定义实验室信息，是外部的资源组和特定于在模板可以访问的环境。 下面是其中一些： 

- 实验室网络标识
- Location
- 在其中存储资源管理器模板文件的存储帐户。 
 
#### <a name="lab-virtual-network"></a>实验室的虚拟网络
[环境连接到实验室的虚拟网络](connect-environment-lab-virtual-network.md)一文介绍如何修改你的资源管理器模板以使用`$(LabSubnetId)`令牌。 创建环境时，`$(LabSubnetId)`令牌替换为第一个子网标记其中**虚拟机中的使用它来创建**选项设置为**true**。 它允许我们的环境以使用以前创建的网络。 如果你想要为过渡和生产环境中测试中使用相同的资源管理器模板，使用`$(LabSubnetId)`作为资源管理器模板参数中的默认值。 

#### <a name="environment-storage-account"></a>环境存储帐户
开发测试实验室支持使用[嵌套资源管理器模板](../azure-resource-manager/resource-group-linked-templates.md)。 [如何使用 Azure 开发测试实验室可以让嵌套资源管理器模板部署更易于测试环境](https://azure.microsoft.com/updates/azure-devtest-labs-streamlined-nested-arm-template-deployment-support-for-arm-template-based-environments)一文介绍了如何使用`_artifactsLocation`和`_artifactsLocationSasToken`令牌创建中的资源管理器模板的 URI同一个文件夹或在主模板中的嵌套文件夹中。 有关这两个标记的详细信息，请参阅**部署项目**一部分[Azure 资源管理器 – 最佳实践指南](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md)。

## <a name="user-experience"></a>用户体验

## <a name="developer"></a>开发人员
开发人员使用相同的工作流用于创建的 VM 创建特定的环境。 他们选择与计算机映像的环境，并输入所需的模板所需的信息。 每个开发人员环境允许更改和改进的内部循环调试的部署。 可以在任何时候使用最新的模板创建环境。  此功能允许环境来销毁并重新创建以帮助减少停机时间无需编写手动创建系统或恢复的错误测试。  

### <a name="testing"></a>正在测试
开发测试实验室环境允许独立测试的特定代码和配置以异步方式。 常见的做法是从单个拉取请求设置代码的环境并开始任何自动测试。 后自动测试已完成运行，任何手动测试可以对执行特定的环境。 通常作为 CI/CD 管道的一部分完成此过程。 

## <a name="management-experience"></a>管理体验

### <a name="cost-tracking"></a>成本跟踪
成本跟踪功能包括不同的环境中的 Azure 资源作为整体的一部分成本趋势。 通过资源的成本不会输出在环境中不同的资源，但显示为单个成本的环境。

### <a name="security"></a>安全
可以使用开发测试实验室正确配置的 Azure 订阅[限制对只能通过在实验室的 Azure 资源的访问](devtest-lab-add-devtest-user.md)。 使用环境，实验室所有者可以允许用户访问使用已批准配置的 PaaS 资源但不允许对任何其他 Azure 资源的访问。 在方案中，实验室用户自定义环境，实验室所有者可以允许参与者访问权限。 参与者访问权限允许实验室用户添加或删除仅在托管的资源组中的 Azure 资源。 它允许对其进行轻松跟踪和管理与允许用户对订阅的参与者访问。

### <a name="automation"></a>自动化
自动化是较大的比例，有效的生态系统的关键组件。 自动化是处理管理或跨订阅和实验室跟踪多个环境的需要。

### <a name="cicd-pipeline"></a>CI/CD 管道
开发测试实验室中的 PaaS 服务可以帮助改进具有已设定焦点的访问控制实验室部署 CI/CD 管道。

## <a name="next-steps"></a>后续步骤
请参阅以下文章，了解有关环境的详细信息： 

- 
- [使用 Azure 资源管理器模板创建多 VM 环境和 PaaS 资源](devtest-lab-create-environment-from-arm.md)
- [配置和在 Azure 开发测试实验室中使用公共环境](devtest-lab-configure-use-public-environments.md)
- [使用 Azure 开发测试实验室中的独立 Service Fabric 群集创建环境](create-environment-service-fabric-cluster.md)
- [连接到实验室的虚拟网络中 Azure 开发测试实验室环境](connect-environment-lab-virtual-network.md)
- [将环境集成到你的 Azure DevOps CI/CD 管道](integrate-environments-devops-pipeline.md)
 





