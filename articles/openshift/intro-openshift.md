---
title: Azure Red Hat OpenShift 简介 |Microsoft Docs
description: 了解 Microsoft Azure Red Hat OpenShift 的功能和优势，以部署和管理基于容器的应用程序。
services: container-service
author: tylermsft
ms.author: twhitney
ms.service: container-service
manager: jeconnoc
ms.topic: overview
ms.date: 05/06/2019
ms.custom: mvc
ms.openlocfilehash: 6121c0f654a61a147e84f0697f3ddb06b7c5db92
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079945"
---
# <a name="azure-red-hat-openshift"></a>Azure Red Hat OpenShift

借助 Microsoft Azure Red Hat OpenShift 服务，可以部署完全托管的 [OpenShift](https://www.openshift.com/) 群集。

Azure Red Hat OpenShift 扩展 [Kubernetes](https://kubernetes.io/)。 在生产环境中使用 Kubernetes 运行容器需要使用其他工具和资源，例如映像注册表、存储管理、网络解决方案以及日志记录和监视工具，这些都必须在一起进行版本控制和测试。 构建基于容器的应用程序需要与中间件、框架、数据库和 CI/CD 工具的更多集成工作。 Azure Red Hat OpenShift 将这一切合并到单个平台，简化 IT 团队操作，并为应用程序团队提供了执行所需的内容。

Azure Red Hat OpenShift 由 Red Hat 和 Microsoft 共同设计、运营和支持，可以提供集成式支持体验。 无需操作虚拟机，无需修补。 Red Hat 和 Microsoft 代表你修补、更新和监视主数据库、基础结构和应用程序节点。 Azure Red Hat OpenShift 群集部署到 Azure 订阅，并包含在你的 Azure 帐单中。

可以选择自己的注册表、网络、存储和 CI/CD 解决方案，或将内置解决方案用于自动源代码管理、容器和应用程序生成、部署、缩放、运行状况管理等。 Azure Red Hat OpenShift 通过 Azure Active Directory 提供集成式登录体验。

要开始使用，请完成[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

## <a name="access-security-and-monitoring"></a>访问权限、安全性和监视

为了改进安全性和管理，Azure Red Hat OpenShift 让你与 Azure Active Directory (Azure AD) 集成，并使用基于 Kubernetes 角色的访问控制 (RBAC)。 也可监视群集和资源的运行状况。

## <a name="cluster-and-node"></a>群集和节点

Azure Red Hat OpenShift 节点在 Azure 虚拟机上运行。 可以将存储连接到节点和 Pod、升级群集配置以及使用 GPU。

## <a name="virtual-networks-and-ingress"></a>虚拟网络和入口

可以将 Azure Red Hat OpenShift 群集部署到现有的虚拟网络。 在此配置中，群集中的每个 pod 在虚拟网络中分配有一个 IP 地址，可以直接与群集中的其他 pod 以及虚拟网络中的其他节点通信。 Pod 还可以连接到对等互连虚拟网络中的其他服务，通过 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/) 或站点到站点 (S2S) VPN 连接连接到本地网络。

有关详细信息，请参阅[在 Azure 上创建 Microsoft Red Hat OpenShift 群集](tutorial-create-cluster.md)。

## <a name="kubernetes-certification"></a>Kubernetes 认证

Azure Red Hat OpenShift 服务已被 CNCF 认证为符合 Kubernetes 规定。

## <a name="next-steps"></a>后续步骤

了解 Azure Red Hat OpenShift 的先决条件：

> [!div class="nextstepaction"]
> [设置开发环境](howto-setup-environment.md)