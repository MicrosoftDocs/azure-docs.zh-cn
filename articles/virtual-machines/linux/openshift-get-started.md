---
title: Azure 中的 OpenShift 概述 | Microsoft Docs
description: Azure 中的 OpenShift 概述。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldwongms
manager: mdotson
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/7/2019
ms.author: haroldw
ms.openlocfilehash: c8ebd10544331d3b7d887124542fe4ee2753c714
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233139"
---
# <a name="openshift-in-azure"></a>Azure 中的 OpenShift

OpenShift 是一个开放可扩展的容器应用程序平台，面向企业提供 Docker 和 Kubernetes。  

OpenShift 包括用于容器业务流程和管理的 Kubernetes 容器。 它增加了以开发人员为中心的工具和以操作为中心的工具，这些工具可实现：

- 快速应用程序开发。
- 轻松部署和缩放。
- 针对团队和应用程序的长期生命期维护。

没有可用的 OpenShift 的多个版本。  这些版本中，只有两个是现在可用于在 Azure 中部署的客户：OpenShift 容器平台和 OKD (以前称为 OpenShift Origin)。

## <a name="azure-red-hat-openshift"></a>Azure Red Hat OpenShift

Microsoft Azure Red Hat OpenShift 是一种完全托管在 Azure 中运行的 OpenShift 的产品。 此服务由 Microsoft 和 Red Hat 共同管理并提供支持。 可以是完整文档[此处](https://docs.microsoft.com/azure/openshift/)。 

## <a name="openshift-container-platform"></a>OpenShift 容器平台

容器平台 是 Red Hat 支持的企业就绪[商业版本](https://www.openshift.com)。 使用此版本时，客户需购买 OpenShift 容器平台的必要权利，并负责安装和管理整个基础结构。

由于客户"拥有"整个平台，他们可以在本地数据中心，或在公有云 （例如 Azure) 中安装它。

## <a name="okd"></a>OKD

OKD 是社区支持的 OpenShift 的[开源](https://www.okd.io/)上游项目。 可在 CentOS 或 Red Hat Enterprise Linux (RHEL) 上安装 OKD。

## <a name="next-steps"></a>后续步骤

- [在 Azure 中配置 OpenShift 的常见先决条件](./openshift-prerequisites.md)
- [在 Azure 中部署 OpenShift 容器平台](./openshift-container-platform.md)
- [部署 OpenShift 容器平台自行管理的 Marketplace 产品/服务](./openshift-marketplace-self-managed.md)
- [在 Azure Stack 中部署 OpenShift](./openshift-azure-stack.md)
- [部署后任务](./openshift-post-deployment.md)
- [OpenShift 部署故障排除](./openshift-troubleshooting.md)
